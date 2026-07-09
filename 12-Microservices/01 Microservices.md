For **5–8 years .NET Core interviews**, Microservices is one of the most heavily tested topics. Interviewers usually don't ask "What is a microservice?" They ask **scenario-based questions** to evaluate whether you've actually built or maintained microservices.

Below are **50+ frequently asked Microservices interview questions with answers**.

---

# 1. What is Microservice Architecture?

### Answer

Microservices is an architectural style where an application is divided into **small, independent services**, each responsible for a single business capability.

Each service:

* Has its own database
* Can be deployed independently
* Can be developed by different teams
* Communicates using HTTP/gRPC/Message Broker

Example:

```
E-Commerce

Order Service

Product Service

Payment Service

Inventory Service

Notification Service
```

Instead of one large application, every module is its own API.

---

# 2. Monolithic vs Microservices

| Monolithic              | Microservices                   |
| ----------------------- | ------------------------------- |
| Single application      | Multiple services               |
| Single deployment       | Independent deployment          |
| Shared database         | Database per service            |
| Scale whole application | Scale individual service        |
| Technology locked       | Different technologies possible |
| Hard to maintain        | Easier to maintain              |

---

# 3. When should you use Microservices?

Use Microservices when

* Large application
* Multiple teams
* Independent deployment required
* Different scalability requirements
* High availability

Don't use Microservices for

* Small CRUD application
* Small startup
* Internal tool

---

# 4. Benefits of Microservices

* Independent deployment
* Independent scaling
* Fault isolation
* Smaller codebase
* Faster development
* Better maintainability

---

# 5. Challenges of Microservices

* Distributed transactions
* Network latency
* Monitoring
* Logging
* Authentication
* Service discovery
* Versioning
* Deployment complexity

---

# 6. How do Microservices communicate?

### Synchronous

```
REST API

HTTP

gRPC
```

Example

```
Order Service

↓

Payment Service
```

Order waits for Payment.

---

### Asynchronous

```
RabbitMQ

Azure Service Bus

Kafka
```

Order publishes event

```
Order Created
```

Notification consumes later.

---

# 7. REST vs gRPC

| REST           | gRPC                   |
| -------------- | ---------------------- |
| HTTP           | HTTP/2                 |
| JSON           | Protocol Buffers       |
| Slower         | Faster                 |
| Human readable | Binary                 |
| Public APIs    | Internal communication |

---

# 8. What is API Gateway?

Instead of calling

```
Order

Payment

Inventory
```

Client calls

```
API Gateway
```

Gateway routes request.

Benefits

* Authentication
* Rate limiting
* Caching
* Load balancing
* Routing

Common Gateway

* YARP (.NET)
* Ocelot
* Azure API Management

---

# 9. What is Service Discovery?

Suppose

```
Payment Service

runs on

10.0.1.5
```

Tomorrow

```
10.0.2.8
```

IP changed.

Instead of hardcoding

Use

* Kubernetes DNS
* Consul
* Eureka

---

# 10. Why Database Per Service?

Bad

```
Order

Payment

Inventory

↓

One Database
```

Everything becomes tightly coupled.

Good

```
Order DB

Payment DB

Inventory DB
```

Each service owns its own data.

---

# 11. Can one Microservice directly access another service's database?

No.

Always communicate through

```
API

or

Events
```

Otherwise coupling increases.

---

# 12. What is Event-Driven Architecture?

Instead of

```
Order

↓

Notification
```

Order publishes

```
OrderCreated Event
```

Notification listens.

Advantages

* Loose coupling
* Better scalability
* Retry possible

---

# 13. RabbitMQ vs Kafka

| RabbitMQ                          | Kafka           |
| --------------------------------- | --------------- |
| Queue                             | Log             |
| Removes message after consumption | Stores messages |
| Banking                           | Analytics       |
| Small-medium                      | Huge traffic    |

---

# 14. What is Saga Pattern?

Distributed transaction.

Example

```
Order

↓

Payment

↓

Inventory

↓

Shipping
```

Inventory failed.

Need rollback.

Saga handles rollback.

---

# 15. Choreography Saga

Every service publishes events.

No central coordinator.

```
Order Created

↓

Payment

↓

Inventory

↓

Shipping
```

---

# 16. Orchestration Saga

One service controls all.

```
Saga Coordinator

↓

Payment

↓

Inventory

↓

Shipping
```

---

# 17. What is Eventual Consistency?

Unlike SQL transaction

Data may take

```
2-3 seconds
```

to synchronize.

All services eventually become consistent.

---

# 18. Why Distributed Transactions are difficult?

Traditional

```
SQL Transaction
```

doesn't work across

```
Order DB

Payment DB

Inventory DB
```

Need Saga.

---

# 19. What is Circuit Breaker?

Payment Service down.

Instead of retrying forever

Circuit opens.

No requests sent.

Later tries again.

Libraries

```
Polly
```

---

# 20. Retry Pattern

Temporary failure

↓

Retry

Example

```csharp
builder.Services.AddHttpClient()
.AddTransientHttpErrorPolicy(policy =>
    policy.WaitAndRetryAsync(3,
        retry => TimeSpan.FromSeconds(2)));
```

---

# 21. Bulkhead Pattern

One failing service shouldn't consume all threads.

Separate resources.

---

# 22. Timeout Pattern

Don't wait forever.

```csharp
client.Timeout =
TimeSpan.FromSeconds(5);
```

---

# 23. Health Checks

```csharp
builder.Services.AddHealthChecks();
```

Endpoint

```
/health
```

Used by

* Kubernetes
* Azure
* Load Balancer

---

# 24. Liveness vs Readiness

Liveness

```
Is application alive?
```

Readiness

```
Can application accept requests?
```

---

# 25. How do you secure Microservices?

* JWT
* OAuth2
* Azure AD / Microsoft Entra ID
* HTTPS
* API Gateway
* Role-based Authorization

---

# 26. How do Microservices authenticate each other?

Common options:

* OAuth 2.0 Client Credentials flow
* Managed Identity (Azure)
* Mutual TLS (mTLS)
* API Keys (less preferred for internal services)

For Azure-hosted services, **Managed Identity** is often the recommended approach.

---

# 27. How do you share code?

Avoid

```
Huge Common Library
```

Use

Small NuGet packages

Example

```
Shared Contracts

Shared Logging

Shared DTOs
```

---

# 28. Logging

Use

```
Serilog

Application Insights

Elastic

Seq
```

Correlation ID is important.

---

# 29. What is Correlation ID?

Request

```
12345
```

travels

```
Gateway

↓

Order

↓

Payment

↓

Inventory
```

All logs

```
12345
```

Easy debugging.

---

# 30. What is Idempotency?

Calling API twice

```
POST Payment
```

Should not charge twice.

Usually implemented using an **Idempotency-Key** stored server-side.

---

# 31. How do you version Microservices?

Options include:

* URL versioning (`/api/v1/orders`)
* Header versioning
* Query string versioning (less common)

Avoid breaking existing clients.

---

# 32. How do you deploy Microservices?

Typical deployment platforms:

* Docker
* Kubernetes
* Azure Kubernetes Service (AKS)
* Azure App Service
* Azure Container Apps

---

# 33. Docker Interview Question

Why Docker?

Every service gets

```
Same Environment

Dev

QA

Production
```

---

# 34. Kubernetes Role

Automatically

* Restarts container
* Scaling
* Load balancing
* Rolling updates

---

# 35. What is Sidecar Pattern?

Additional container

Example

```
App

+

Logging Container
```

---

# 36. CQRS

Separate

```
Read

Write
```

Models.

Improves performance.

---

# 37. Event Sourcing

Instead of storing

```
Balance=1000
```

Store

```
Deposit

Withdraw

Deposit
```

Rebuild state anytime.

---

# 38. Caching

Use

```
Redis
```

Reduce database calls.

---

# 39. Message Broker

Examples

* RabbitMQ
* Kafka
* Azure Service Bus

Used for asynchronous communication.

---

# 40. Why not use synchronous calls everywhere?

If Payment API is down,

Order API also waits.

Eventually,

Entire system slows.

Use events when possible.

---

# 41. How do you monitor Microservices?

Typical stack:

* Application Insights
* Prometheus
* Grafana
* OpenTelemetry
* Azure Monitor

---

# 42. What is Distributed Tracing?

One request

↓

Gateway

↓

Order

↓

Payment

↓

Inventory

Shows complete execution path and latency.

---

# 43. What is OpenTelemetry?

Vendor-neutral standard for collecting:

* Traces
* Metrics
* Logs

---

# 44. How do you handle failures in asynchronous processing?

* Retry with backoff
* Dead Letter Queue (DLQ)
* Idempotent consumers
* Monitoring and alerts

---

# 45. What is a Dead Letter Queue?

Messages that repeatedly fail processing are moved to a separate queue for later investigation instead of being retried forever.

---

# 46. How do you prevent duplicate event processing?

* Idempotent consumers
* Store processed message IDs
* Ignore duplicates

---

# 47. Why shouldn't services share DTOs directly?

Sharing a single DTO library tightly couples services. Prefer:

* Versioned contracts
* Separate API contracts
* Event contracts for messaging

---

# 48. What database is best for Microservices?

There isn't one answer. Choose based on the service:

* SQL Server/PostgreSQL for transactional data
* MongoDB for document data
* Redis for caching
* Cosmos DB for globally distributed scenarios

---

# 49. How do you test Microservices?

* Unit Tests
* Integration Tests
* Contract Tests
* End-to-End Tests
* Load Tests

---

# 50. Real Interview Scenario

### Question

**Order Service needs to:**

1. Save the order
2. Process payment
3. Reserve inventory
4. Send email

What approach would you use?

### Good Answer

1. Order Service saves the order in its own database.
2. It publishes an `OrderCreated` event.
3. Payment Service consumes the event and processes payment.
4. On success, Payment Service publishes `PaymentSucceeded`.
5. Inventory Service consumes that event and reserves stock.
6. Inventory Service publishes `InventoryReserved`.
7. Notification Service consumes the event and sends the email.
8. If payment or inventory fails, use the **Saga Pattern** with compensating actions (for example, cancel the order or refund the payment).

This design keeps services loosely coupled, scalable, and resilient.

---

# Questions Asked Frequently in Senior .NET Interviews (5–8 Years)

| Topic                               | Importance |
| ----------------------------------- | ---------- |
| API Gateway                         | ⭐⭐⭐⭐⭐      |
| Service Discovery                   | ⭐⭐⭐⭐       |
| Docker                              | ⭐⭐⭐⭐⭐      |
| Kubernetes                          | ⭐⭐⭐⭐⭐      |
| RabbitMQ / Azure Service Bus        | ⭐⭐⭐⭐⭐      |
| Saga Pattern                        | ⭐⭐⭐⭐⭐      |
| Eventual Consistency                | ⭐⭐⭐⭐⭐      |
| Polly (Retry, Circuit Breaker)      | ⭐⭐⭐⭐⭐      |
| Distributed Transactions            | ⭐⭐⭐⭐⭐      |
| Health Checks                       | ⭐⭐⭐⭐       |
| OpenTelemetry & Distributed Tracing | ⭐⭐⭐⭐       |
| Correlation IDs                     | ⭐⭐⭐⭐       |
| Idempotency                         | ⭐⭐⭐⭐⭐      |
| CQRS                                | ⭐⭐⭐⭐       |
| Event Sourcing                      | ⭐⭐⭐        |
| Redis Caching                       | ⭐⭐⭐⭐       |
| OAuth2 / JWT / Entra ID             | ⭐⭐⭐⭐⭐      |

These are the topics that appear most consistently in senior .NET Core microservices interviews because they focus on designing, operating, and troubleshooting distributed systems—not just building REST APIs.
