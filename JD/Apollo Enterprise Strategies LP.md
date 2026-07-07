# 1. C# / .NET (Highest Priority ⭐⭐⭐⭐⭐)

Since C#/.NET is one of their primary languages, expect deep questions.

### Core C#

* OOP concepts
* SOLID Principles
* Dependency Injection
* Interfaces vs Abstract Classes
* Delegates
* Events
* Generics
* Reflection
* Extension Methods
* LINQ
* async/await
* Task Parallel Library
* Exception Handling
* IDisposable
* Garbage Collection
* Memory Management
* Value Types vs Reference Types
* string vs StringBuilder
* Records vs Classes (.NET 8+)

Typical interview questions

* Explain async/await internally.
* What happens if you forget await?
* Difference between Task.Run() and Task.Factory.StartNew()
* ConfigureAwait(false)?
* How does Dependency Injection work?
* Singleton vs Scoped vs Transient?
* How is memory allocated?

---

# 2. ASP.NET Core Web API ⭐⭐⭐⭐⭐

This is almost guaranteed.

Prepare

* Middleware Pipeline
* Routing
* Model Binding
* Model Validation
* Filters
* Authentication
* Authorization
* JWT
* Dependency Injection
* Logging
* Configuration
* Exception Middleware
* API Versioning
* Swagger
* Health Checks

Questions

* Explain request lifecycle.
* How does middleware execute?
* Global Exception Handling?
* Authentication vs Authorization?
* Secure APIs?
* How would you version APIs?

---

# 3. REST APIs ⭐⭐⭐⭐⭐

Very important.

Know

* GET
* POST
* PUT
* PATCH
* DELETE

Status Codes

* 200
* 201
* 204
* 400
* 401
* 403
* 404
* 409
* 500

Topics

* Idempotency
* Pagination
* Filtering
* Sorting
* Rate Limiting
* Versioning
* Caching
* HATEOAS (basic understanding)

Questions

Design

```
GET /customers

GET /customers/{id}

POST /customers

PATCH /customers/{id}
```

---

# 4. Microservices ⭐⭐⭐⭐⭐

The JD explicitly mentions Microservices.

Prepare

* Monolith vs Microservices

* Service Discovery

* API Gateway

* Circuit Breaker

* Retry

* Distributed Transactions

* Saga Pattern

* Event Driven Architecture

* Communication

* REST

* Messaging

* Kafka

Questions

Why Microservices?

When not to use them?

Challenges?

How to communicate between services?

---

# 5. Entity Framework Core ⭐⭐⭐⭐⭐

Since you've been studying EF Core recently, this is a big advantage.

Know

* DbContext
* Change Tracking
* Lazy Loading
* Explicit Loading
* Eager Loading
* Include
* ThenInclude
* AsNoTracking
* AsSplitQuery
* Transactions
* Concurrency
* Optimistic Concurrency
* RowVersion
* Migrations
* Global Query Filters
* Cascade Delete
* Restrict
* SetNull
* IQueryable vs IEnumerable

Performance

* Projection
* Select()
* Pagination
* Bulk Updates
* Compiled Queries
* Indexes

---

# 6. SQL ⭐⭐⭐⭐⭐

Expect SQL questions.

Prepare

Joins

* Inner
* Left
* Right
* Full

Window Functions

```
ROW_NUMBER()

RANK()

DENSE_RANK()

```

Indexes

Clustered

Non-clustered

Composite

Execution Plans

Normalization

Transactions

Isolation Levels

Deadlocks

Stored Procedures

Views

CTEs

Pagination

OFFSET FETCH

Cursor Pagination

---

# 7. System Design ⭐⭐⭐⭐

Very likely.

Prepare

Design

* Order Management
* Loan Processing
* Customer Management
* Notification System
* File Upload System

Know

Load Balancer

Caching

Redis

CDN

Message Queue

Database Sharding

Read Replicas

Scaling

Horizontal

Vertical

Consistency

Availability

CAP Theorem

---

# 8. Data Structures & Algorithms ⭐⭐⭐⭐

JD explicitly mentions this.

Prepare

Arrays

Strings

HashMap

Queue

Stack

Linked List

Tree

Binary Search

Sorting

Searching

Time Complexity

Big O

Practice

20-30 medium problems.

---

# 9. React ⭐⭐⭐⭐

Not very deep.

Know

Components

Hooks

useState

useEffect

Props

State

Context

API Calls

Performance

Memoization

---

# 10. Azure ⭐⭐⭐⭐

Because you have Azure experience.

Know

* App Service
* Azure Functions
* Storage Account
* Key Vault
* Service Bus
* Event Grid
* Azure SQL
* Azure DevOps
* Application Insights

Questions

How did you deploy?

How did you secure secrets?

---

# 11. CI/CD ⭐⭐⭐⭐

Prepare

Azure DevOps

GitHub Actions

Pipeline

Build

Release

Environment

Rollback

---

# 12. Cloud Architecture ⭐⭐⭐⭐

Understand

Docker

Containers

Kubernetes (basic)

Load Balancer

Scaling

Secrets

Monitoring

---

# 13. Design Patterns ⭐⭐⭐⭐

Must know

Repository

Factory

Singleton

Strategy

Builder

Mediator

CQRS (basic)

Unit Of Work

---

# 14. Performance Optimization ⭐⭐⭐⭐

The JD repeatedly mentions performance.

Know

Caching

Redis

Memory Cache

Indexes

Connection Pooling

Async Programming

Database Optimization

Profiling

Logging

---

# 15. Event Driven Architecture ⭐⭐⭐

Good to know

Kafka

Producer

Consumer

Topics

Partitions

Consumer Groups

Exactly Once

At Least Once

Retry

Dead Letter Queue

---

# 16. Security ⭐⭐⭐⭐

JWT

OAuth

Authentication

Authorization

HTTPS

Encryption

Hashing

SQL Injection

XSS

CSRF

Secrets Management

---

# 17. Engineering Scenarios ⭐⭐⭐⭐⭐

Expect questions like:

* API is slow. How do you troubleshoot?
* Database CPU is 100%.
* API is timing out.
* Production is down.
* Duplicate transactions are happening.
* Multiple users update the same record.
* Loan application is processed twice.
* Millions of records need processing.
* How do you make a system scalable?
* How do you improve reliability?

These evaluate practical engineering judgment more than memorized facts.

---

# 18. Behavioral Questions ⭐⭐⭐⭐

Prepare examples using the STAR method for:

* A challenging production issue you resolved.
* Leading your React Native POC team.
* Optimizing a slow application.
* Resolving conflicts within a team.
* Learning a new technology quickly.
* A project you're particularly proud of.
* A mistake you made and what you learned from it.

---

# Based on your experience, your 7-day preparation plan

| Day | Focus                                                                                     |
| --- | ----------------------------------------------------------------------------------------- |
| 1   | C#, OOP, SOLID, async/await, DI                                                           |
| 2   | ASP.NET Core, REST APIs, Authentication, Middleware                                       |
| 3   | EF Core + SQL (joins, indexes, transactions, pagination, performance)                     |
| 4   | Microservices, System Design, Event-Driven Architecture                                   |
| 5   | Azure, CI/CD, Docker, performance optimization                                            |
| 6   | DSA (arrays, hash maps, trees, complexity) + React fundamentals                           |
| 7   | Mock interview covering coding, system design, behavioral questions, and resume deep dive |

## Topics I'd expect to carry the most weight

Given this specific JD, I'd allocate your effort roughly as follows:

* **30%** C#, .NET, ASP.NET Core
* **20%** REST APIs, Microservices, System Design
* **20%** SQL + EF Core + performance tuning
* **10%** Data Structures & Algorithms
* **10%** Azure, CI/CD, cloud concepts
* **5%** React fundamentals
* **5%** Behavioral and project discussions
