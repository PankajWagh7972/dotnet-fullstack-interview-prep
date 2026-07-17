Excellent. The **second most important topic** after caching is **Scaling**.

This is one of the favorite questions for **5–8 years .NET Full Stack** interviews because it naturally leads into discussions about Azure, load balancing, SQL Server, caching, microservices, and system design.

---

# Topic 2 - Scaling in ASP.NET Core Applications

---

# Q1. What is Scaling?

### Interview Answer (2 Minutes)

> Scaling is the process of increasing an application's capacity to handle more users, more requests, or larger amounts of data while maintaining performance and reliability.
>
> There are two main types of scaling:
>
> * Vertical Scaling (Scale Up)
> * Horizontal Scaling (Scale Out)

---

# Real Example

Suppose your e-commerce website receives

```text
100 Users

↓

100 Requests/Minute
```

Everything works fine.

During Diwali Sale

```text
100 Users

↓

100,000 Users
```

Now

* CPU reaches 100%
* Memory is full
* SQL becomes slow
* APIs start timing out

Now we need **Scaling**.

---

# Q2. What is Vertical Scaling (Scale Up)?

### Interview Answer

Vertical Scaling means increasing the resources of a **single server**.

Example

Before

```text
CPU : 4 Core

RAM : 8 GB
```

After

```text
CPU : 16 Core

RAM : 64 GB
```

Same server.

More power.

---

## Diagram

```text
Application

↓

Server

↓

4 CPU

↓

Upgrade

↓

16 CPU
```

---

## Advantages

* Easy
* No code changes
* No architecture changes

---

## Disadvantages

* Hardware limit
* Expensive
* Single Point of Failure

---

# Follow-up

**Interviewer**

Can Vertical Scaling solve every problem?

Answer

No.

Eventually, a server reaches its maximum capacity.

After that, Horizontal Scaling is required.

---

# Q3. What is Horizontal Scaling (Scale Out)?

### Interview Answer

Instead of upgrading one server,

we add multiple application servers.

Example

Instead of

```text
1 Server
```

We deploy

```text
5 Servers
```

---

## Architecture

```text
                Users

                   │

          Load Balancer

      ┌────────┼────────┐

Server1     Server2    Server3

      │         │          │

      └────────┼──────────┘

             SQL Server
```

---

Advantages

* High Availability
* Better Performance
* Fault Tolerance

---

Disadvantages

Need

* Load Balancer
* Distributed Cache
* Session Management

---

# Follow-up

Which scaling is preferred in Cloud?

Answer

Horizontal Scaling

Azure App Service

Azure Kubernetes Service

Azure VM Scale Sets

All support Scale-Out.

---

# Q4. Why can't we use IMemoryCache in Horizontal Scaling?

This is one of the most asked questions.

Suppose

Server 1

```text
Product

Cached
```

Server 2

```text
Empty Cache
```

User A

↓

Server1

↓

Fast

User B

↓

Server2

↓

SQL Query

Different results.

---

## Correct Architecture

```text
Server1

↓

Redis

↑

Server2

↑

Server3
```

Everyone shares the same cache.

---

### Interview Answer

IMemoryCache stores data inside one server.

In multiple servers,

every server has different memory.

Therefore,

Redis Distributed Cache should be used.

---

# Follow-up

Which cache do you recommend in production?

Answer

Redis.

Always.

---

# Q5. Why Load Balancer?

Imagine

One API Server

```text
1000 Requests

↓

One Server

↓

Crash
```

Now users cannot access the application.

---

Instead

```text
1000 Requests

↓

Load Balancer

↓

Server1

↓

Server2

↓

Server3
```

Load is divided.

---

Interview Answer

A Load Balancer distributes incoming traffic across multiple servers.

Benefits

* Better Performance
* High Availability
* Fault Tolerance
* Zero Downtime Deployment

---

# Follow-up

Can users know which server handled their request?

Answer

No.

Load Balancer decides automatically.

---

# Q6. How does Load Balancer choose a server?

Common Algorithms

### Round Robin

```text
User1 → Server1

User2 → Server2

User3 → Server3

User4 → Server1
```

---

### Least Connections

Choose server having minimum active requests.

Used in production.

---

### IP Hash

Same client

↓

Same Server

Useful for session affinity.

---

# Interview Question

Which algorithm is best?

Answer

Depends.

API

↓

Least Connection

Static Website

↓

Round Robin

---

# Q7. Session Problem in Multiple Servers

Suppose

User Login

↓

Server1

Stores Session

User Next Request

↓

Server2

Session Missing.

User Logged Out.

---

Solution

Store Session in

Redis

instead of server memory.

---

Architecture

```text
Server1

↓

Redis Session

↑

Server2

↑

Server3
```

---

# Q8. Sticky Session

What is it?

Load Balancer always sends

same user

↓

same server.

Advantages

Simple.

Disadvantages

Poor scalability.

If server crashes

↓

Session lost.

Modern applications usually avoid relying on sticky sessions.

---

# Q9. Auto Scaling

Azure supports automatic scaling.

Example

If CPU

>

80%

↓

Create New Server

If CPU

<

20%

↓

Remove Server

---

Interview Answer

Azure automatically increases or decreases instances based on CPU, memory, request count, or custom metrics.

This helps optimize both performance and cost.

---

# Follow-up

Can scaling happen automatically?

Answer

Yes.

Azure Autoscale handles it.

---

# Q10. Database Bottleneck

Suppose

Application

↓

10 Servers

↓

One SQL Server

Still slow.

Why?

Database becomes bottleneck.

---

Solutions

* Redis
* Read Replica
* Indexing
* Query Optimization
* Partitioning
* Sharding

---

# Q11. Read Replica

Interview Answer

One database

handles Writes.

Multiple databases

handle Reads.

Architecture

```text
Application

↓

Primary Database

↓

Writes

↓

Replica1

↓

Reads

↓

Replica2

↓

Reads
```

Useful for

E-commerce

Banking

Reporting

---

# Follow-up

Can we write to Read Replica?

Answer

No.

Writes always go to Primary.

---

# Q12. How would you scale an ASP.NET Core API?

Interview Answer

If my API receives heavy traffic, I would:

1. Deploy multiple API instances.
2. Place them behind Azure Load Balancer or Azure Application Gateway.
3. Store cache in Redis.
4. Move sessions to Redis or use JWT to make the API stateless.
5. Optimize SQL queries and indexes.
6. Use asynchronous programming.
7. Enable response compression.
8. Use Azure Autoscale.
9. Monitor with Application Insights.
10. Add read replicas if the database becomes the bottleneck.

This answer demonstrates knowledge across the application, infrastructure, and database layers.

---

# Q13. Real Project Answer (You Can Use)

> During peak business hours, some APIs experienced increased response times because they were frequently reading the same reference data. We optimized performance by implementing Redis distributed caching for lookup data, tuning SQL indexes, and ensuring our APIs remained stateless using JWT authentication. The application was deployed on Azure App Service, which allowed us to scale out multiple instances during high traffic. Application Insights helped us monitor latency and identify bottlenecks proactively.

---

# Q14. Scenario Question

**Interviewer**

Our API response increased from

```text
300ms

↓

5 Seconds
```

What would you check?

---

### Senior-Level Answer

I would investigate layer by layer:

**Application Layer**

* Check Application Insights for slow endpoints.
* Review recent deployments.
* Verify thread pool usage and async code.

**Cache Layer**

* Check Redis availability.
* Review cache hit ratio.
* Look for cache expiration or stampede issues.

**Database Layer**

* Analyze SQL execution plans.
* Check missing indexes.
* Monitor blocking or deadlocks.
* Verify connection pool usage.

**Infrastructure Layer**

* CPU and memory utilization.
* Network latency.
* Load balancer health.
* Number of active application instances.

This systematic approach shows you troubleshoot based on evidence rather than guesswork.

---

# ⭐ Interview Gold Tip

If asked:

> **"How do you improve scalability?"**

Don't answer only:

> "We add more servers."

Instead say:

> "Scaling is not just adding servers. First, I identify the bottleneck. If the issue is at the application layer, I scale out the API and use stateless services with Redis caching. If the bottleneck is the database, I optimize queries, add indexes, introduce read replicas, or partition data if necessary. I also use asynchronous processing, Azure Autoscale, and continuous monitoring with Application Insights to ensure the system scales efficiently."

That answer is what interviewers expect from someone with **5+ years of .NET experience**.

---

## Next Topic Recommendation (Highest Priority)

Tomorrow's interview is very likely to include this:

> **SQL Indexing + Execution Plans + Query Optimization**

This is one of the most common areas where interviewers probe deeply because every scalable .NET application ultimately depends on SQL Server performance. It also naturally leads to discussions about Entity Framework Core, API performance, and overall system design.


Excellent questions. These are exactly the kinds of **follow-up questions** that distinguish a 5+ year developer from someone who has only memorized definitions. Let's go one by one.

---

# 1. How do Read Replicas stay in sync with the Primary Database?

## Interview Question

**Interviewer:**

> If writes happen only on the Primary database, how does the Read Replica get updated?

### Answer

Read Replicas use **database replication**.

Whenever data changes on the Primary database, SQL Server captures the transaction and sends it to the replica(s).

```text
          INSERT Product

                │

                ▼

        Primary Database

                │

      Transaction Log (.ldf)

                │

      Replication Mechanism

      ┌─────────┴─────────┐

      ▼                   ▼

 Read Replica 1      Read Replica 2
```

The replica continuously applies those changes.

---

## Real Example

Suppose:

```sql
INSERT INTO Product(Name,Price)
VALUES('iPhone',1000)
```

This is written to:

```text
Primary Database
```

Immediately after the transaction commits:

```text
Transaction Log

↓

Replica receives log

↓

Replica executes same transaction
```

Now both databases contain:

```text
Product

iPhone

1000
```

---

## Is it always instant?

No.

Usually there is a **Replication Lag**.

Example:

```text
Primary

10:00:00

↓

Replica

10:00:02
```

2 seconds later.

---

## Follow-up

**Interviewer**

Can users read stale data?

Answer

Yes.

Because replication is asynchronous in many systems.

That's why:

* Order Placement → Read Primary
* Reports → Read Replica

---

# 2. Which SQL Server Replication Types exist?

### Snapshot Replication

Copies entire database.

Good for

Reference data.

---

### Transactional Replication ⭐

Most common.

Copies transaction log continuously.

Near real-time.

---

### Merge Replication

Both databases update independently.

Later merged.

Used in offline systems.

---

# 3. How do applications decide where to read?

Never let the UI decide.

Application Layer handles it.

Example

```text
Client

↓

.NET API

↓

Read?

↓

Read Replica

Write?

↓

Primary Database
```

Repository example

```csharp
public async Task<Product> GetProduct(int id)
{
    return await _readDb.Products
        .FirstOrDefaultAsync(x=>x.Id==id);
}

public async Task Save(Product product)
{
    _writeDb.Products.Update(product);
    await _writeDb.SaveChangesAsync();
}
```

---

# 4. What if user updates and immediately refreshes?

This is an excellent senior-level question.

User:

```text
Update Salary
```

Immediately

```text
GET Salary
```

Replica hasn't synchronized yet.

User sees old value.

---

Solutions

### Option 1 (Most Common)

Read from Primary after write.

Example

```text
POST

↓

Primary

↓

Next GET

↓

Primary
```

---

### Option 2

Use Cache

Update cache immediately.

Users read cache.

---

### Option 3

Accept eventual consistency.

Good for

* Reports
* Analytics

---

# 5. How do we add a Load Balancer for an ASP.NET Core API?

The **application code does not change**. The load balancer sits **in front of** your API instances.

```text
                  Internet

                     │

                     ▼

        Azure Application Gateway

                     │

        ┌────────────┼────────────┐

        ▼            ▼            ▼

   API Instance1 API Instance2 API Instance3

             │          │          │

             └──────────┼──────────┘

                    SQL Server
```

---

## Azure App Service

Suppose

You deployed

```text
API

↓

Azure App Service
```

Open

```text
Scale Out
```

Increase

```text
Instances

1

↓

5
```

Azure automatically puts them behind Microsoft's load balancer.

No code changes.

---

## Azure Kubernetes Service

Pods

```text
Pod1

Pod2

Pod3
```

Service

↓

Load Balancer

↓

Traffic distributed.

---

# 6. Does .NET Core know which server handled the request?

No.

ASP.NET Core is unaware.

The load balancer forwards the request.

---

# 7. What changes are required in the API?

Ideally:

**Nothing.**

BUT

Application should be

## Stateless

Don't do

```csharp
public static string CurrentUser;
```

Wrong.

Because

Server1

stores

```text
Pankaj
```

Server2

stores

```text
NULL
```

Always use:

* JWT
* Redis
* SQL

Never store user state in server memory.

---

# 8. What is Rate Limiting?

One of the hottest .NET 8 interview topics.

Suppose

One user sends

```text
100000 Requests
```

Server crashes.

Rate Limiting stops abuse.

---

Example

Allow

```text
100 Requests

Per Minute
```

Request

101

↓

Blocked

HTTP

```text
429

Too Many Requests
```

---

# .NET 8 Rate Limiting

Program.cs

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("fixed", opt =>
    {
        opt.Window = TimeSpan.FromMinutes(1);
        opt.PermitLimit = 100;
        opt.QueueLimit = 0;
    });
});

app.UseRateLimiter();

app.MapControllers().RequireRateLimiting("fixed");
```

---

# 9. Types of Rate Limiting

## Fixed Window

```text
1 Minute

↓

100 Requests

↓

Reset
```

Simple.

---

## Sliding Window

Instead of fixed intervals,

continuously moves.

More accurate.

---

## Token Bucket ⭐

Imagine bucket

```text
100 Tokens
```

Each request

↓

Consumes one token.

Tokens refill gradually.

Most cloud providers use this.

---

## Leaky Bucket

Requests enter bucket.

Released

Constant speed.

Smooth traffic.

---

# Which one is best?

Interview Answer

For most APIs

Sliding Window

or

Token Bucket.

---

# 10. Other Techniques to Improve Scalability

## Response Compression

Without

```text
2 MB JSON
```

With Gzip

```text
300 KB
```

ASP.NET Core

```csharp
builder.Services.AddResponseCompression();

app.UseResponseCompression();
```

---

## Pagination

Bad

```sql
SELECT *
FROM Orders
```

Good

```sql
SELECT *
FROM Orders
ORDER BY Id
OFFSET 0 ROWS
FETCH NEXT 50 ROWS ONLY
```

---

## Projection

Wrong

```csharp
_context.Products.ToList();
```

Returns every column.

Better

```csharp
_context.Products
.Select(x=>new
{
    x.Id,
    x.Name
});
```

Only required columns are returned.

---

## AsNoTracking()

Wrong

```csharp
_context.Products.ToList();
```

Better

```csharp
_context.Products
.AsNoTracking()
.ToList();
```

Useful for read-only queries because EF Core skips change tracking, reducing memory usage and improving performance.

---

## Async Programming

Wrong

```csharp
_context.SaveChanges();
```

Better

```csharp
await _context.SaveChangesAsync();
```

This frees the request thread while waiting for I/O, allowing the server to handle more concurrent requests.

---

## Connection Pooling

Opening SQL connections repeatedly is expensive.

ADO.NET maintains a pool of reusable connections.

```text
Request

↓

Pool

↓

Existing Connection

↓

SQL
```

Connection pooling improves throughput and reduces latency.

---

## Background Processing

Don't send emails during the request.

Instead

```text
API

↓

Queue

↓

Background Worker

↓

Email
```

This keeps API response times low.

---

# ⭐ Senior-Level Interview Question

**Interviewer:**

> Our application has 10 API servers, Redis, and a Load Balancer, but it's still slow. What could be the bottleneck?

### Strong Answer

I would investigate each layer:

1. **Database** – Slow queries, missing indexes, blocking, deadlocks.
2. **External Services** – Payment gateway, third-party APIs, network latency.
3. **Redis** – Cache hit ratio, latency, memory pressure.
4. **Application** – Synchronous I/O, thread pool starvation, memory leaks.
5. **Infrastructure** – CPU, memory, disk I/O, network bandwidth.
6. **Load Balancer** – Health probes, uneven traffic distribution.
7. **Monitoring** – Application Insights, SQL execution plans, Azure Monitor.

> **Key point:** Scaling the API tier doesn't help if the real bottleneck is the database or an external dependency.

---

## 💡 One Interview Correction

Many candidates say:

> "We added more servers, so the application scaled."

A better answer is:

> "Adding servers is only one aspect of scaling. A scalable application must be stateless, use distributed caching (Redis), optimize database access with proper indexing and query tuning, support asynchronous processing where appropriate, and be monitored continuously. Otherwise, adding servers simply shifts the bottleneck to another layer."

That's the kind of answer that interviewers expect from someone with **5+ years of .NET and Azure experience**.
