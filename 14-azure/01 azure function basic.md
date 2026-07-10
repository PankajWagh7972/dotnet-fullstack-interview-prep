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


Since you're preparing for **5–7 years .NET Full Stack interviews**, I'll explain Azure Functions the way interviewers expect—not just definitions, but **how they work internally, when to use them, architecture decisions, storage, scaling, and real-world scenarios.**

---

# Azure Functions Complete Interview Guide

---

# 1. What is Azure Function?

An Azure Function is a **serverless event-driven compute service**.

Serverless does **NOT** mean there are no servers.

It means:

* Microsoft manages the servers.
* You only deploy your code.
* Azure handles:

  * Scaling
  * OS updates
  * Runtime
  * Availability
  * Infrastructure

Your responsibility is only writing business logic.

---

## Traditional API

```text
User

↓

API

↓

Virtual Machine

↓

Windows/Linux

↓

IIS/Kestrel

↓

Application
```

You manage

* Server
* Scaling
* Patching
* CPU
* Memory

---

## Azure Function

```text
User

↓

Trigger

↓

Azure Function

↓

Azure Runtime

↓

Azure Infrastructure
```

Azure manages everything.

---

# 2. When should you use Azure Functions?

Good for

* Event Processing
* Background Jobs
* Scheduled Jobs
* Queue Processing
* Image Processing
* Notifications
* File Uploads
* IoT
* Microservices

Don't use Azure Functions for

❌ Large Monolithic APIs

❌ Long-running user requests

❌ Complex Web Applications

Use

* ASP.NET Core API
* App Service
* AKS

instead.

---

# 3. Azure Function Execution Model

Every Azure Function has

```text
Trigger

↓

Input Binding

↓

Function

↓

Output Binding
```

Example

```text
Blob Uploaded

↓

Blob Trigger

↓

Resize Image

↓

Save to Blob Storage
```

No code required for reading blob.

Binding handles it.

---

# 4. Azure Function Lifecycle

Suppose

Customer uploads image.

Execution

```text
Upload Image

↓

Blob Storage

↓

Blob Trigger Fires

↓

Runtime Creates Function Instance

↓

Execute Code

↓

Save Thumbnail

↓

Destroy Instance
```

If idle

Function instance is destroyed.

This is why Cold Start happens.

---

# 5. What is Cold Start?

Suppose

Nobody uses function for

```text
30 minutes
```

Azure unloads it.

Next request

```text
Request

↓

Start VM

↓

Load Runtime

↓

Load DLL

↓

Run Function
```

Delay

```text
2-10 seconds
```

This is Cold Start.

Premium Plan removes cold start.

---

# 6. Hosting Plans

## Consumption Plan

Most popular.

Pros

* Pay per execution
* Auto Scale
* Cheap

Cons

* Cold Start
* Max execution timeout (configurable, but not unlimited)

Best for

* Notifications
* Queue Processing
* APIs with low or burst traffic

---

## Premium Plan

Pros

* No Cold Start
* VNET Support
* More CPU
* Always Running

Best for

Enterprise applications.

---

## Dedicated Plan (App Service Plan)

Runs on dedicated VMs.

Pros

* Always Running
* Predictable Performance

Cons

You pay even when idle.

---

## Flex Consumption Plan (Modern)

Microsoft's latest serverless hosting option.

Features

* Better scaling
* Improved networking
* Reduced cold start compared to classic Consumption
* More flexible instance management

---

# 7. Azure Function Triggers

There are many triggers.

Let's understand each.

---

# HTTP Trigger

Most common.

Architecture

```text
Browser

↓

HTTP Request

↓

Azure Function

↓

Response
```

Example

```text
GET /employees
```

Use Cases

* APIs
* Webhooks
* Mobile Backend

Example

```csharp
[Function("GetEmployee")]
public HttpResponseData Run(...)
```

---

# Timer Trigger

Runs automatically.

Architecture

```text
Timer

↓

Azure Function

↓

Execute
```

Cron

```text
Every day

Every hour

Every minute
```

Example

Every midnight

```text
Generate Reports

Backup Database

Archive Logs

Delete Temp Files
```

---

# Blob Trigger

Triggered whenever

Blob Storage changes.

```text
Upload PDF

↓

Blob Storage

↓

Function

↓

OCR

↓

Save Metadata
```

Use Cases

* Image Resize
* OCR
* Virus Scan
* Document Processing

---

# Queue Trigger (Azure Storage Queue)

```text
Application

↓

Storage Queue

↓

Azure Function
```

Simple messaging.

Use

* Email
* Logging
* Notifications

---

# Service Bus Trigger

Enterprise Queue.

```text
API

↓

Service Bus

↓

Function
```

Supports

* Retry
* Dead Letter Queue
* Sessions
* FIFO
* Duplicate Detection

Use

Enterprise Microservices.

---

# Cosmos DB Trigger

Whenever data changes

```text
Insert

↓

Cosmos DB

↓

Function
```

Use

* Audit
* Notifications
* Search Index
* Analytics

---

# Event Grid Trigger

Azure resource events.

Example

```text
Blob Created

↓

Event Grid

↓

Function
```

Used for

Azure Event Routing.

---

# Event Hub Trigger

Streaming.

Millions of events.

```text
IoT Device

↓

Event Hub

↓

Function

↓

Analytics
```

Use

* Telemetry
* Sensors
* GPS
* Logs

---

# Kafka Trigger

Streaming from Kafka.

Mostly

Large Enterprises.

---

# Durable Function Trigger

Most advanced.

Supports

* Long-running workflows
* Checkpointing
* State

Example

Loan Approval

```text
Submit

↓

Manager Approval

↓

Finance

↓

Legal

↓

Complete
```

Normal Function cannot remember state.

Durable Function can.

---

# Storage Used by Azure Functions

Interview Question

Why does Azure Function need Storage Account?

Answer

Azure Functions use Azure Storage internally for runtime operations.

Storage stores

* Logs
* Trigger metadata
* Scale Controller information
* Checkpoints
* Durable Function state

Without storage

Function App cannot operate (for most hosting models).

---

# Storage Types

## Blob Storage

Stores

```text
Images

Videos

PDF

CSV

Backup Files
```

Use Cases

* Upload
* Download
* Image Processing

---

## Queue Storage

Simple Queue.

```text
Message

↓

Queue

↓

Function
```

Use

Background jobs.

---

## Table Storage

NoSQL Key-Value Database.

Stores

```text
Logs

Configuration

Metadata
```

Cheap.

---

## File Share

SMB Share.

Mostly

Enterprise applications.

---

# Difference

| Storage | Purpose            |
| ------- | ------------------ |
| Blob    | Files              |
| Queue   | Messages           |
| Table   | NoSQL              |
| Files   | Shared File System |

---

# Azure Function Bindings

Instead of writing SDK code

Binding injects data automatically.

Without Binding

```csharp
BlobClient

↓

Download

↓

Open Stream

↓

Read
```

With Binding

```csharp
public void Run(
[BlobTrigger(...)] Stream stream)
```

Done.

---

# Input Binding

Reads data.

```text
SQL

↓

Function
```

---

# Output Binding

Writes data.

```text
Function

↓

Blob

↓

Queue

↓

Cosmos
```

---

# Retry Policy

Suppose SQL Server is down.

Function

↓

Exception

↓

Retry

↓

Retry

↓

Retry

↓

DLQ

---

# Scaling

1000 messages

Azure creates

```text
Instance1

Instance2

Instance3

Instance4
```

Each processes messages independently.

---

# Azure Function vs WebJob

| Azure Function    | WebJob                      |
| ----------------- | --------------------------- |
| Serverless        | Runs inside App Service     |
| Auto Scale        | Limited                     |
| Multiple Triggers | Mostly scheduled/background |
| Modern            | Legacy                      |

---

# Azure Function vs Logic Apps

Function

* Code

Logic App

* No Code

Logic Apps connect

* SAP
* Salesforce
* Outlook
* Dynamics

Business workflow.

---

# Azure Function vs ASP.NET Core API

| Azure Function | ASP.NET Core       |
| -------------- | ------------------ |
| Event Driven   | Request Driven     |
| Auto Scale     | Manual/Configured  |
| Small Tasks    | Complete API       |
| Serverless     | Hosted Application |

---

# Azure Function Interview Questions (5–8 Years)

## Q1. What happens internally when an HTTP request reaches an Azure Function?

**Answer:**

1. Request reaches Azure Front Door/App Service infrastructure.
2. Azure Functions runtime receives the request.
3. If no instance exists, Azure creates one (cold start if necessary).
4. Dependency Injection container is initialized (once per instance).
5. Function executes.
6. Response is returned.
7. Instance is reused for subsequent requests until Azure scales down.

---

## Q2. Why is a Storage Account mandatory?

Because Azure Functions use it internally for:

* Trigger state
* Scale controller
* Checkpointing
* Host metadata
* Durable Functions state
* Logs (depending on configuration)

---

## Q3. What is the difference between Azure Queue Storage and Azure Service Bus?

| Azure Queue Storage | Azure Service Bus       |
| ------------------- | ----------------------- |
| Simple messaging    | Enterprise messaging    |
| Basic retry         | Advanced retry policies |
| No Topics           | Topics & Subscriptions  |
| No Sessions         | Sessions supported      |
| No Transactions     | Transactions supported  |
| Lower cost          | More features           |

---

## Q4. What are Durable Functions?

Durable Functions extend Azure Functions by allowing you to build **stateful workflows**. They use orchestration functions to coordinate activities and persist execution state, making them suitable for long-running business processes.

Common patterns:

* Function chaining
* Fan-out/Fan-in
* Async HTTP APIs
* Human approval workflows
* Monitoring loops

---

## Q5. What are the different types of bindings?

* **Trigger Binding** – Starts the function (HTTP, Timer, Blob, Service Bus, etc.).
* **Input Binding** – Reads data from external services without manual SDK code.
* **Output Binding** – Writes data to external services such as Blob Storage, Queue Storage, or Cosmos DB.

---

## Q6. Can Azure Functions call other Azure Functions?

Yes. Common approaches include:

* HTTP Trigger → HTTP call
* Service Bus → Event-driven communication (preferred for microservices)
* Event Grid
* Durable Function orchestration
* Azure SDKs (depending on the service)

For loosely coupled architectures, asynchronous messaging with **Azure Service Bus** is generally preferred over direct HTTP calls.

---

## Q7. What is the difference between In-Process and Isolated Worker models?

| In-Process                                   | Isolated Worker                                |
| -------------------------------------------- | ---------------------------------------------- |
| Runs inside the Azure Functions host process | Runs in a separate .NET process                |
| Shares the host runtime                      | Independent worker process                     |
| Tighter coupling                             | Better isolation and flexibility               |
| Older model                                  | Recommended for .NET 8+ and future development |

Microsoft recommends the **Isolated Worker** model for new .NET applications because it provides better version independence, middleware support, and long-term compatibility.
