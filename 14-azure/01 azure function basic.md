Azure Functions and Azure Service Bus are among the **most frequently asked Azure interview topics** for .NET developers with 5+ years of experience. Interviewers usually ask about real-world event-driven architectures rather than basic definitions.

---

# What is Azure Function?

An **Azure Function** is a **serverless compute service** that lets you execute code in response to events without managing servers.

Instead of:

* Creating an ASP.NET Core API
* Deploying it
* Managing IIS/Kestrel
* Scaling manually

You simply write a function, and Azure handles the infrastructure.

## Real-Time Example

Imagine an e-commerce application.

```text
Customer Places Order
        │
        ▼
SQL Database
        │
        ▼
Azure Function Triggered
        │
        ├── Send Email
        ├── Send SMS
        ├── Update Inventory
        └── Generate Invoice
```

You only pay when the function executes (Consumption Plan).

---

# What is Azure Service Bus?

Azure Service Bus is a **message broker** that enables reliable communication between applications or microservices.

Instead of calling another service directly:

```text
Order API
    │
HTTP
    ▼
Inventory API
```

Use Service Bus:

```text
Order API
    │
Send Message
    ▼
Azure Service Bus Queue
    │
    ▼
Inventory Service
```

Advantages:

* Loose coupling
* Reliable messaging
* Retry support
* Dead-letter queue
* Better scalability

---

# Real-Time Scenario

Suppose an order is placed.

Without Service Bus

```text
React

↓

Order API

↓

Payment API

↓

Inventory API

↓

Email API

↓

SMS API
```

Problem:

If Email API is down, the Order API also fails.

---

With Service Bus

```text
React

↓

Order API

↓

Service Bus Queue

↓

Inventory Function

↓

Email Function

↓

Notification Function

↓

Analytics Function
```

Now each service works independently.

---

# Queue vs Topic

## Queue (One-to-One)

```text
Producer

↓

Queue

↓

Consumer
```

Only one consumer processes each message.

Example:

* Order Processing
* Invoice Generation
* Payment Processing

---

## Topic (One-to-Many)

```text
Producer

↓

Topic

├── Email Subscription

├── SMS Subscription

├── Analytics Subscription

└── Audit Subscription
```

One message is delivered to multiple subscribers.

Example:

Order Placed Event.

Need to:

* Send Email
* Update CRM
* Analytics
* Notification

---

# Azure Function Triggers

| Trigger             | Use Case                     |
| ------------------- | ---------------------------- |
| HTTP Trigger        | API                          |
| Timer Trigger       | Scheduled Job                |
| Blob Trigger        | File Upload                  |
| Queue Trigger       | Storage Queue                |
| Service Bus Trigger | Process Service Bus Messages |
| Event Grid Trigger  | Azure Events                 |
| Event Hub Trigger   | Streaming Data               |
| Cosmos DB Trigger   | Database Changes             |

---

# Service Bus Trigger Example

Suppose a queue named

```text
orders
```

A message arrives

```json
{
    "OrderId":1001,
    "Customer":"John",
    "Amount":500
}
```

Azure Function executes automatically.

---

# Create Azure Function (.NET 8 Isolated Worker)

```csharp
using Azure.Messaging.ServiceBus;
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;

public class OrderProcessor
{
    private readonly ILogger<OrderProcessor> _logger;

    public OrderProcessor(ILogger<OrderProcessor> logger)
    {
        _logger = logger;
    }

    [Function("OrderProcessor")]
    public async Task Run(
        [ServiceBusTrigger("orders",
        Connection = "ServiceBusConnection")]
        ServiceBusReceivedMessage message)
    {
        var body = message.Body.ToString();

        _logger.LogInformation(body);

        await Task.CompletedTask;
    }
}
```

---

# Send Message from .NET API

Install

```text
Azure.Messaging.ServiceBus
```

```csharp
var client = new ServiceBusClient(connectionString);

var sender = client.CreateSender("orders");

var message = new ServiceBusMessage(
@"{
""OrderId"":1001,
""Amount"":1000
}");

await sender.SendMessageAsync(message);
```

Flow

```text
ASP.NET Core API

↓

SendMessageAsync()

↓

Service Bus Queue

↓

Azure Function

↓

Process Order
```

---

# Deserialize JSON

Suppose

```json
{
    "OrderId":100,
    "Amount":500
}
```

Model

```csharp
public class Order
{
    public int OrderId { get; set; }

    public decimal Amount { get; set; }
}
```

Function

```csharp
var order =
JsonSerializer.Deserialize<Order>(
message.Body.ToString());
```

Now

```csharp
Console.WriteLine(order.OrderId);
```

---

# Complete Flow

```text
React

↓

Order API

↓

Save Order

↓

Send Service Bus Message

↓

Queue

↓

Azure Function

↓

Generate Invoice

↓

Email Customer

↓

Update Inventory

↓

Notify CRM
```

---

# Retry Mechanism

Suppose Email Server is down.

```text
Queue

↓

Function

↓

Exception

↓

Retry

↓

Retry

↓

Retry
```

Azure retries automatically based on the configured retry policy.

---

# Dead Letter Queue (DLQ)

Suppose message processing fails repeatedly.

```text
Queue

↓

Retry

↓

Retry

↓

Retry

↓

Dead Letter Queue
```

The message is moved to the DLQ instead of being lost.

Common reasons:

* Invalid JSON
* Database unavailable
* Validation failure
* Poison message

Administrators can inspect, fix, and resubmit the message.

---

# Peek Lock vs Receive and Delete

## Peek Lock (Default)

```text
Read Message

↓

Lock Message

↓

Process

↓

Success

↓

Complete

↓

Remove
```

If processing fails, the lock expires and the message becomes available again.

Use this in production.

---

## Receive and Delete

```text
Read

↓

Delete Immediately

↓

Process
```

If processing fails, the message is already gone.

Use only when occasional message loss is acceptable.

---

# Scaling

Suppose:

```text
100000 Orders
```

Azure automatically creates multiple Function instances.

```text
Queue

↓

Function Instance 1

Function Instance 2

Function Instance 3

Function Instance 4
```

Each instance processes different messages concurrently.

---

# Queue vs Topic

| Queue                       | Topic                                        |
| --------------------------- | -------------------------------------------- |
| One producer → One consumer | One producer → Multiple subscribers          |
| One message processed once  | One message copied to multiple subscriptions |
| Work distribution           | Event broadcasting                           |

---

# Interview Scenario

### Question

A customer places an order.

You need to:

* Save Order
* Send Email
* Generate Invoice
* Notify CRM
* Update Inventory

Would you call each service using HTTP?

### Expected Answer

No.

Save the order first.

Publish an `OrderPlaced` event to **Azure Service Bus**.

Each downstream service (Email, Inventory, CRM, Invoice) subscribes to that event and processes it independently using **Azure Functions** or other consumers.

Benefits:

* Loose coupling
* Independent scaling
* Automatic retries
* Dead-letter handling
* Better fault tolerance
* Easier to add new consumers without changing the Order API

---

# Common Interview Questions

### 1. Why use Service Bus instead of HTTP?

HTTP is synchronous and tightly coupled. If the downstream service is unavailable, the request may fail. Service Bus provides asynchronous, reliable messaging with retries and buffering.

---

### 2. Why use Azure Functions with Service Bus?

Azure Functions automatically execute when a message arrives, making them ideal for event-driven processing without managing infrastructure.

---

### 3. What happens if the Function crashes?

If using **Peek Lock**, the message remains in the queue until the lock expires, then it is retried. After exceeding the maximum delivery count, it is moved to the Dead Letter Queue.

---

### 4. Queue or Topic—which should you choose?

* **Queue**: One consumer should process the message (e.g., payment processing).
* **Topic**: Multiple independent consumers should react to the same event (e.g., email, inventory, analytics).

---

### 5. Service Bus vs Azure Storage Queue

| Azure Service Bus                 | Azure Storage Queue             |
| --------------------------------- | ------------------------------- |
| Rich enterprise messaging         | Simple queue service            |
| FIFO support (Sessions)           | No guaranteed FIFO              |
| Topics & Subscriptions            | Queues only                     |
| Dead Letter Queue                 | Basic poison message handling   |
| Duplicate detection               | No                              |
| Transactions                      | Supported                       |
| Scheduled messages                | Supported                       |
| Best for enterprise microservices | Best for simple background jobs |

For enterprise .NET microservices, **Azure Service Bus** is typically the preferred choice because it offers advanced messaging capabilities, reliability, and scalability.
