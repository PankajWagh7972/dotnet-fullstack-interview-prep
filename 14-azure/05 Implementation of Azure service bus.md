A production-grade implementation for this scenario uses an **Azure Service Bus Topic**. Each service (Payment, Inventory, Notification) gets its own **Subscription**, so they all receive a copy of the same message. Each subscription maintains its own retry count, dead-letter queue (DLQ), and processing state.

## Architecture

```text
                           Order API
                               |
                               |
                     Publish OrderCreated
                               |
                               v
                  +-------------------------+
                  | Service Bus Topic       |
                  | order-created-topic     |
                  +-------------------------+
                     /         |          \
                    /          |           \
                   /           |            \
                  v            v             v
        Payment Subscription  Inventory   Notification
             payment-sub         inventory-sub   notification-sub
                  |                  |                |
                  v                  v                v
            Payment API       Inventory API     Notification API
```

Each subscription receives **its own copy** of the message.

---

# Step 1: Create Azure Resources

Create

```
Topic
------
order-created-topic

Subscriptions
-------------
payment-sub
inventory-sub
notification-sub
```

Each subscription has its own

* Active Queue
* Dead Letter Queue
* Retry Count
* Lock

---

# Step 2: Message Contract

```csharp
public class OrderCreatedEvent
{
    public Guid OrderId { get; set; }

    public string CustomerId { get; set; }

    public decimal Amount { get; set; }

    public DateTime CreatedOn { get; set; }
}
```

---

# Step 3: Publish Message

Install

```bash
dotnet add package Azure.Messaging.ServiceBus
```

Publisher

```csharp
using Azure.Messaging.ServiceBus;
using System.Text.Json;

public class OrderPublisher
{
    private readonly ServiceBusClient _client;

    public OrderPublisher(ServiceBusClient client)
    {
        _client = client;
    }

    public async Task Publish(OrderCreatedEvent order)
    {
        ServiceBusSender sender =
            _client.CreateSender("order-created-topic");

        var message = new ServiceBusMessage(
            JsonSerializer.Serialize(order));

        message.MessageId = order.OrderId.ToString();

        message.ContentType = "application/json";

        await sender.SendMessageAsync(message);
    }
}
```

---

# Step 4: Payment Service

```csharp
using Azure.Messaging.ServiceBus;

public class PaymentProcessor : IHostedService
{
    private readonly ServiceBusProcessor _processor;

    public PaymentProcessor(ServiceBusClient client)
    {
        _processor = client.CreateProcessor(
            "order-created-topic",
            "payment-sub",
            new ServiceBusProcessorOptions
            {
                AutoCompleteMessages = false,

                MaxConcurrentCalls = 5,

                MaxAutoLockRenewalDuration =
                    TimeSpan.FromMinutes(10)
            });

        _processor.ProcessMessageAsync += ProcessMessage;

        _processor.ProcessErrorAsync += ErrorHandler;
    }

    private async Task ProcessMessage(ProcessMessageEventArgs args)
    {
        try
        {
            var order =
                JsonSerializer.Deserialize<OrderCreatedEvent>(
                    args.Message.Body);

            Console.WriteLine($"Processing payment {order.OrderId}");

            await ProcessPayment(order);

            await args.CompleteMessageAsync(args.Message);
        }
        catch (PaymentGatewayException ex)
        {
            Console.WriteLine(ex.Message);

            await args.AbandonMessageAsync(args.Message);
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex);

            await args.DeadLetterMessageAsync(
                args.Message,
                "Payment Failure",
                ex.Message);
        }
    }

    private Task ErrorHandler(ProcessErrorEventArgs args)
    {
        Console.WriteLine(args.Exception);

        return Task.CompletedTask;
    }

    public async Task StartAsync(CancellationToken token)
        => await _processor.StartProcessingAsync();

    public async Task StopAsync(CancellationToken token)
        => await _processor.StopProcessingAsync();

    private async Task ProcessPayment(OrderCreatedEvent order)
    {
        await Task.Delay(1000);

        // Payment Logic
    }
}
```

Inventory and Notification services follow the same pattern but subscribe to `inventory-sub` and `notification-sub`.

---

# Retry Flow

Assume payment gateway is temporarily unavailable.

```
Receive Message

↓

Payment API

↓

Throws Exception

↓

Abandon()

↓

Message Returns To Queue

↓

Retry
```

Retry continues until `MaxDeliveryCount` is reached.

---

# Configure Retry

In Azure Portal

```
Subscription

↓

Properties

↓

Max Delivery Count = 10
```

or using infrastructure as code.

```
Delivery Attempt

1

↓

2

↓

3

↓

...

↓

10

↓

Dead Letter Queue
```

---

# Dead Letter Queue

After 10 failures

```
Payment Subscription

↓

Dead Letter Queue
```

Inventory and Notification continue normally because each subscription is independent.

---

# Reading DLQ

```csharp
var processor = client.CreateProcessor(
    "order-created-topic",
    "payment-sub",
    new ServiceBusProcessorOptions
    {
        SubQueue = SubQueue.DeadLetter
    });
```

---

# Error Handling Strategy

| Error             | Action            |
| ----------------- | ----------------- |
| SQL timeout       | Abandon           |
| HTTP 503          | Abandon           |
| Network issue     | Abandon           |
| Invalid JSON      | Dead Letter       |
| Validation failed | Dead Letter       |
| Duplicate Order   | Complete (ignore) |

---

# Idempotency

Service Bus provides **at-least-once delivery**, so consumers must handle duplicates.

Example using a database table:

```sql
ProcessedMessages
-----------------
MessageId
ProcessedDate
```

```csharp
if(await repository.Exists(message.MessageId))
{
    await args.CompleteMessageAsync(args.Message);
    return;
}

await repository.Insert(message.MessageId);

// Business Logic

await args.CompleteMessageAsync(args.Message);
```

This ensures duplicate deliveries don't cause duplicate business actions (for example, charging a customer twice).

---

# Retry Policy for External APIs

Use a resilience library such as Polly.

```csharp
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .Or<TimeoutException>()
    .WaitAndRetryAsync(
        3,
        retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));

await retryPolicy.ExecuteAsync(async () =>
{
    await paymentGateway.Charge(order);
});
```

This performs exponential backoff before allowing the message to be abandoned back to Service Bus.

---

# Production Best Practices

* **Topics + Subscriptions** for fan-out to multiple services.
* **`AutoCompleteMessages = false`** so your code explicitly completes messages after successful processing.
* Configure **`MaxConcurrentCalls`** based on service capacity.
* Enable **lock renewal** (`MaxAutoLockRenewalDuration`) for long-running processing.
* Set **`MaxDeliveryCount`** (for example, 10) to move poison messages to the DLQ.
* Implement **idempotency** using `MessageId` or a business key.
* Use **Polly** (or similar) for transient failures when calling downstream APIs.
* Monitor **dead-letter queues** and create alerts for messages accumulating in the DLQ.
* Log and correlate processing using the message's **CorrelationId** and **MessageId** for end-to-end tracing.

This is the pattern commonly used in production systems for scenarios such as **Order Created**, where Payment, Inventory, and Notification services all need to react independently to the same event while maintaining reliable delivery, retries, and fault isolation.
