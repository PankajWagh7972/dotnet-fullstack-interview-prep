This is a **very common 5–8 years .NET interview question**, especially for Azure-based projects.

Interviewers may ask:

* **How do microservices communicate asynchronously?**
* **How do you implement async processing in Azure?**
* **How do you avoid tight coupling between microservices?**
* **How would you process millions of messages?**

The most common answer in the **.NET + Azure ecosystem** is:

> **Azure Service Bus + Azure Functions (or Background Service) + Event-driven architecture**

---

# Typical Architecture

```text
                User

                 │
                 ▼

         Order API (.NET Core)

                 │
         Save Order (SQL)

                 │
       Publish Event
   OrderCreatedEvent

                 │
                 ▼

         Azure Service Bus
      (Queue / Topic)

      ┌─────────┼─────────┐
      │         │         │
      ▼         ▼         ▼

 Inventory   Payment   Notification
 Service     Service     Service

      │         │         │
      ▼         ▼         ▼

 Database   Database   Send Email/SMS
```

Notice:

The Order service **doesn't call** Inventory or Payment directly.

Instead, it **publishes an event**.

---

# Why Async?

Imagine this synchronous flow:

```text
Order API

↓

Inventory API

↓

Payment API

↓

Email API

↓

Return Response
```

Problems:

* Slow
* One service failure affects others
* Timeout risk
* Tight coupling

---

Async flow:

```text
Order API

↓

Save Order

↓

Publish Event

↓

Return 200 OK
```

Everything else happens in the background.

---

# Azure Service Bus

Azure Service Bus is Microsoft's enterprise message broker.

Think of it as a post office.

```text
Order Service

↓

Service Bus

↓

Payment Service
```

Order Service doesn't know:

* where Payment is
* whether Payment is online
* whether Notification is slow

It simply drops a message into Service Bus.

---

# Queue vs Topic

## Queue

One producer

One consumer

```text
Producer

↓

Queue

↓

Consumer
```

Only one service processes the message.

Example:

```text
Generate Invoice
```

Only Invoice Service should process it.

---

## Topic

One producer

Multiple subscribers

```text
Order Created

↓

Topic

↓

Inventory

↓

Email

↓

Payment

↓

Analytics
```

Every subscriber receives a copy.

Very common in microservices.

---

# Example

## Order API

```csharp
public async Task CreateOrder(CreateOrderRequest request)
{
    var order = new Order();

    _db.Orders.Add(order);

    await _db.SaveChangesAsync();

    await _serviceBus.PublishAsync(
        new OrderCreatedEvent
        {
            OrderId = order.Id
        });
}
```

Notice:

The API never calls Payment Service.

---

# Service Bus Message

```json
{
  "OrderId":1001,
  "CustomerId":45,
  "Amount":2500
}
```

---

# Payment Service

It listens continuously.

```text
Azure Service Bus

↓

Payment Service

↓

Receive Message

↓

Process Payment

↓

Update Database
```

---

# Azure Function Example

```csharp
[Function("ProcessPayment")]
public async Task Run(
    [ServiceBusTrigger("orders")]
    string message)
{
    // Deserialize

    // Process payment

    // Save database
}
```

Whenever a message arrives:

Azure automatically executes this function.

No polling required.

---

# BackgroundService Example

Instead of Azure Function:

```csharp
public class PaymentWorker : BackgroundService
{
    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            // Read Service Bus message

            // Process

            // Complete message
        }
    }
}
```

Useful when hosted inside ASP.NET Core or Worker Service.

---

# Event Example

```csharp
public class OrderCreatedEvent
{
    public int OrderId { get; set; }

    public decimal Amount { get; set; }

    public int CustomerId { get; set; }
}
```

This is what gets published.

---

# Event Flow

```text
Create Order

↓

Save SQL

↓

Publish OrderCreated

↓

Azure Service Bus

↓

Inventory Service

↓

Reserve Stock
```

---

```text
Same Event

↓

Payment Service

↓

Process Payment
```

---

```text
Same Event

↓

Notification Service

↓

Send Email
```

One event.

Three independent services.

---

# Reliability Features

Azure Service Bus provides:

### Retry

If processing fails:

```text
Receive Message

↓

Exception

↓

Retry
```

---

### Dead Letter Queue (DLQ)

If retries exceed the limit:

```text
Service Bus

↓

Dead Letter Queue
```

Messages are preserved for investigation instead of being lost.

---

### Duplicate Detection

Avoids processing the same message twice.

---

### Sessions

Keeps related messages in order.

Example:

```text
Order-100

↓

Payment Started

↓

Payment Completed

↓

Invoice Generated
```

These can be processed sequentially.

---

# Common Azure Components

| Component           | Purpose                                                |
| ------------------- | ------------------------------------------------------ |
| Azure Service Bus   | Reliable messaging between services                    |
| Azure Functions     | Event/message processing                               |
| Azure Storage Queue | Simpler, lower-cost queueing                           |
| Event Grid          | Reactive event routing (e.g., Blob Created)            |
| Event Hubs          | High-throughput event ingestion (telemetry, logs, IoT) |
| Azure Logic Apps    | Low-code workflow automation                           |

---

# Common Interview Scenario

**Interviewer:**

> User places an order.
>
> Inventory, Payment, Email and Analytics all need to know.

How will you implement?

Answer:

```text
Order API

↓

Save Order

↓

Publish OrderCreated Event

↓

Azure Service Bus Topic

↓

Inventory

↓

Payment

↓

Email

↓

Analytics
```

Every service is independent.

---

# Another Scenario

**Interviewer:**

> Payment service is down.

Answer:

The message stays in Azure Service Bus until Payment Service becomes available. Other services, such as Email and Inventory, can continue processing independently. This improves resilience and prevents request failures caused by one unavailable service.

---

# Idempotency (Very Important)

**Interviewer:**

> What if the same message is delivered twice?

Answer:

Consumers should be **idempotent**, meaning processing the same message multiple times produces the same result. Common approaches include:

* Store and check a unique `MessageId` before processing.
* Use a business key (e.g., `OrderId`) to detect duplicate work.
* Enforce database unique constraints where appropriate.

This prevents duplicate payments, duplicate emails, or duplicate inventory reservations.

---

# Best Practices

* ✅ Prefer **events** over direct service-to-service API calls for asynchronous workflows.
* ✅ Use **Azure Service Bus Topics** for publish/subscribe scenarios.
* ✅ Keep messages small; send identifiers rather than large object graphs.
* ✅ Design consumers to be **idempotent**.
* ✅ Configure retries and monitor the **Dead Letter Queue**.
* ✅ Add distributed tracing (e.g., Application Insights/OpenTelemetry) to follow requests across services.

---

## Interview Answer (2–3 minutes)

> "In Azure-based .NET microservices, I typically use an event-driven architecture with Azure Service Bus. The API performs its core business transaction, commits the database changes, and then publishes an event such as `OrderCreated`. Services like Inventory, Payment, Notification, and Analytics subscribe to that event and process it independently using Azure Functions or Worker Services. This keeps services loosely coupled, improves scalability, and allows failures to be isolated. I also implement retries, dead-letter queues, and idempotent consumers to handle duplicate deliveries and transient failures reliably."

This is the architecture most interviewers expect when discussing **asynchronous microservices on Azure**.


Idempotency is one of the **most frequently asked concepts** in microservices interviews because **Azure Service Bus, RabbitMQ, Kafka, and SQS all provide *at least once* delivery**. That means **the same message may be delivered more than once**, so your consumer must safely handle duplicates.

Let's implement it step by step using **.NET 8 + Azure Service Bus + EF Core + SQL Server**.

---

# Scenario

Suppose your Order Service publishes:

```json
{
  "MessageId": "7f4f51b5-2f90-4d27-a317-c6d7d3d21a90",
  "OrderId": 1001,
  "CustomerId": 5,
  "Amount": 2500
}
```

Payment Service receives it.

Now imagine this happens:

```
Receive Message

↓

Payment Successful

↓

Save Payment

↓

Application crashes ❌

↓

Azure Service Bus never receives Complete()

↓

Service Bus delivers message again
```

Without idempotency:

```
Payment #1

↓

Payment #2

↓

Customer charged twice ❌
```

---

# Solution

Maintain a table that tracks processed messages.

```
ProcessedMessages

----------------------------------------

MessageId

ProcessedDate
```

If a message already exists:

```
Ignore it.
```

---

# Step 1 - Entity

```csharp
public class ProcessedMessage
{
    public Guid MessageId { get; set; }

    public DateTime ProcessedOnUtc { get; set; }
}
```

---

# Step 2 - DbContext

```csharp
public class PaymentDbContext : DbContext
{
    public DbSet<ProcessedMessage> ProcessedMessages => Set<ProcessedMessage>();

    public DbSet<Payment> Payments => Set<Payment>();
}
```

---

# Step 3 - Create Unique Index

```csharp
protected override void OnModelCreating(ModelBuilder builder)
{
    builder.Entity<ProcessedMessage>()
        .HasKey(x => x.MessageId);
}
```

or SQL

```sql
CREATE TABLE ProcessedMessages
(
    MessageId UNIQUEIDENTIFIER PRIMARY KEY,
    ProcessedOnUtc DATETIME2 NOT NULL
)
```

The primary key guarantees uniqueness.

---

# Step 4 - Event

```csharp
public class OrderCreatedEvent
{
    public Guid MessageId { get; set; }

    public int OrderId { get; set; }

    public decimal Amount { get; set; }

    public int CustomerId { get; set; }
}
```

---

# Step 5 - Azure Function Consumer

```csharp
public class PaymentConsumer
{
    private readonly PaymentDbContext _db;

    public PaymentConsumer(PaymentDbContext db)
    {
        _db = db;
    }

    [Function("ProcessPayment")]
    public async Task Run(
        [ServiceBusTrigger("orders")]
        OrderCreatedEvent message)
    {
        // Check duplicate

        bool alreadyProcessed =
            await _db.ProcessedMessages
                .AnyAsync(x => x.MessageId == message.MessageId);

        if (alreadyProcessed)
        {
            return;
        }

        var payment = new Payment
        {
            OrderId = message.OrderId,
            Amount = message.Amount
        };

        _db.Payments.Add(payment);

        _db.ProcessedMessages.Add(
            new ProcessedMessage
            {
                MessageId = message.MessageId,
                ProcessedOnUtc = DateTime.UtcNow
            });

        await _db.SaveChangesAsync();
    }
}
```

Works...

But there is still a problem.

---

# Race Condition

Imagine two consumers receive the same message almost simultaneously.

```
Consumer A

↓

Checks DB

↓

Not Found
```

At exactly the same time

```
Consumer B

↓

Checks DB

↓

Not Found
```

Both continue.

Now both insert payment.

Duplicate payment.

---

# Better Solution

Everything should be inside one transaction.

```csharp
await using var transaction =
    await _db.Database.BeginTransactionAsync();
```

Now:

```csharp
try
{
    ...

    await _db.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
}
```

Still not perfect.

Why?

Both consumers may still pass the `AnyAsync()` check before either commits.

---

# Production Solution

Instead of:

```
Check

↓

Insert
```

Simply:

```
Insert

↓

If duplicate key

↓

Ignore
```

The database becomes the source of truth.

---

## Code

```csharp
try
{
    _db.ProcessedMessages.Add(
        new ProcessedMessage
        {
            MessageId = message.MessageId,
            ProcessedOnUtc = DateTime.UtcNow
        });

    await _db.SaveChangesAsync();
}
catch (DbUpdateException)
{
    // Duplicate message

    return;
}
```

Now only one consumer succeeds.

The second gets

```
Violation of PRIMARY KEY
```

and exits safely.

---

# Even Better

Wrap payment + processed message in one transaction.

```csharp
await using var transaction =
    await _db.Database.BeginTransactionAsync();

try
{
    _db.ProcessedMessages.Add(
        new ProcessedMessage
        {
            MessageId = message.MessageId,
            ProcessedOnUtc = DateTime.UtcNow
        });

    _db.Payments.Add(
        new Payment
        {
            OrderId = message.OrderId,
            Amount = message.Amount
        });

    await _db.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch (DbUpdateException)
{
    await transaction.RollbackAsync();

    // duplicate

    return;
}
```

Now either:

```
Payment

+

ProcessedMessage

```

are both committed

OR

nothing is.

---

# Better Architecture

Instead of putting idempotency logic in every consumer, create a reusable service.

```csharp
public interface IIdempotencyService
{
    Task<bool> HasProcessedAsync(Guid messageId);

    Task MarkProcessedAsync(Guid messageId);
}
```

Implementation:

```csharp
public class IdempotencyService : IIdempotencyService
{
    private readonly PaymentDbContext _db;

    public IdempotencyService(PaymentDbContext db)
    {
        _db = db;
    }

    public async Task<bool> HasProcessedAsync(Guid messageId)
    {
        return await _db.ProcessedMessages
            .AnyAsync(x => x.MessageId == messageId);
    }

    public async Task MarkProcessedAsync(Guid messageId)
    {
        _db.ProcessedMessages.Add(
            new ProcessedMessage
            {
                MessageId = messageId,
                ProcessedOnUtc = DateTime.UtcNow
            });

        await _db.SaveChangesAsync();
    }
}
```

Every consumer can use it.

---

# Enterprise Pattern (Inbox Pattern)

Many enterprise systems implement the **Inbox Pattern**.

```
Azure Service Bus

↓

Consumer

↓

Inbox Table

↓

Business Logic

↓

Commit

↓

Complete Message
```

The Inbox table stores every received message. If the message already exists, it's skipped. This provides reliable idempotent processing and is widely used with EF Core and SQL Server.

---

# Interview Question

**Interviewer:** *Why not just use `AnyAsync()` before processing?*

**Good Answer:**

> "`AnyAsync()` is not sufficient because two consumers may read the database before either inserts the record, creating a race condition. The correct approach is to enforce uniqueness with a database primary key or unique index and treat the insert as the idempotency check. This guarantees correctness even with concurrent consumers. Ideally, the idempotency record and the business update are committed in the same transaction."

---

# Production-Grade Flow

```text
Azure Service Bus
        │
        ▼
Receive Message
        │
        ▼
Begin SQL Transaction
        │
        ▼
Insert ProcessedMessage (PK = MessageId)
        │
        ├── Duplicate Key?
        │      │
        │      └── Yes → Rollback → Acknowledge/Exit
        │
        ▼
Execute Business Logic
(Create Payment / Update Order)
        │
        ▼
SaveChanges()
        │
        ▼
Commit Transaction
        │
        ▼
Complete Service Bus Message
```

### Additional Interview Tip

If the interviewer asks, **"Is storing `MessageId` enough?"**, you can answer:

> "It's sufficient if the producer generates a globally unique `MessageId` and reuses it on retries. In some business scenarios, it's even better to use a business key such as `OrderId` plus an operation type, because that prevents duplicate business actions even if a new message is accidentally published with a different `MessageId`."
