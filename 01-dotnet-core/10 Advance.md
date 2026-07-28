This JD is targeted at a **Senior Full Stack .NET + React Developer (5–8+ years)** with strong focus on **high-performance backend systems, React optimization, Azure deployment, async processing, authentication, and production troubleshooting**.

Below are **advanced interview questions with detailed answers** similar to what Microsoft, Uber, Amazon, JPMorgan, or product companies ask.

---

# 1. Explain Authentication vs Authorization in ASP.NET Core.

### Answer

Authentication determines **who the user is**.

Authorization determines **what the user can access.**

Flow:

```
Client
   |
JWT Token
   |
Authentication Middleware
   |
User Principal Created
   |
Authorization Middleware
   |
Controller
```

Authentication

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true
    };
});
```

Authorization

```csharp
[Authorize(Roles="Admin")]
public IActionResult GetUsers()
{
}
```

Policy-based

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ManagerOnly",
        policy => policy.RequireClaim("Department","IT"));
});
```

```
[Authorize(Policy="ManagerOnly")]
```

---

# 2. How would you secure APIs in production?

### Answer

Multiple layers.

✔ JWT Authentication

✔ HTTPS only

✔ Refresh Tokens

✔ Role-based authorization

✔ Claims-based authorization

✔ API Rate Limiting

✔ CORS

✔ Request Validation

✔ SQL Injection prevention

✔ XSS prevention

✔ CSRF protection (if cookies)

✔ Secrets stored in Azure Key Vault

✔ Logging

✔ Audit Trail

---

# 3. Explain Refresh Token implementation.

Flow

```
Login

↓

Access Token (15 min)

↓

Refresh Token (30 days)

↓

Access Token expires

↓

Client sends Refresh Token

↓

Validate Refresh Token

↓

Generate New Access Token
```

Database

```
UserId

RefreshToken

Expiry

CreatedDate

Revoked
```

Advantages

* User stays logged in
* Short-lived access token
* Better security

---

# 4. Explain IHostedService.

Answer

Background service running independently.

Used for

* polling
* email
* notifications
* cache refresh
* scheduled jobs

Example

```csharp
public class Worker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while(!stoppingToken.IsCancellationRequested)
        {
            Console.WriteLine("Running...");
            await Task.Delay(10000);
        }
    }
}
```

Register

```csharp
builder.Services.AddHostedService<Worker>();
```

---

# 5. Difference between BackgroundService and IHostedService

| IHostedService            | BackgroundService |
| ------------------------- | ----------------- |
| Interface                 | Abstract class    |
| Must implement StartAsync | Only ExecuteAsync |
| More boilerplate          | Easier            |
| Flexible                  | Recommended       |

---

# 6. How would you execute a task every 10 seconds?

Option 1

```csharp
PeriodicTimer
```

```csharp
protected override async Task ExecuteAsync(CancellationToken token)
{
    var timer = new PeriodicTimer(TimeSpan.FromSeconds(10));

    while(await timer.WaitForNextTickAsync(token))
    {
        await Process();
    }
}
```

Better than

```
Task.Delay()
```

because

* precise timing
* cancellation support

---

# 7. Quartz.NET vs Hangfire vs BackgroundService

| Feature         | BackgroundService | Quartz  | Hangfire |
| --------------- | ----------------- | ------- | -------- |
| In-memory       | Yes               | Yes     | No       |
| Persistent Jobs | No                | Yes     | Yes      |
| Dashboard       | No                | Limited | Yes      |
| Retry           | Manual            | Yes     | Yes      |
| Cluster         | No                | Yes     | Yes      |

---

# 8. How do you avoid overlapping scheduled jobs?

Problem

```
10 sec timer

↓

Job takes 15 sec

↓

Second Job starts

↓

Duplicate execution
```

Solution

SemaphoreSlim

```csharp
private SemaphoreSlim semaphore = new(1,1);

await semaphore.WaitAsync();

try
{
    await Process();
}
finally
{
    semaphore.Release();
}
```

---

# 9. Explain optimistic concurrency.

Entity Framework

```csharp
[Timestamp]
public byte[] RowVersion { get; set; }
```

EF checks

```
Old Version == Current Version

Yes

↓

Update

No

↓

DbUpdateConcurrencyException
```

---

# 10. Explain async/await internals.

When compiler sees

```csharp
await service.GetData();
```

Compiler generates

State Machine

```
State 0

↓

API Call

↓

Thread Released

↓

IO Complete

↓

Resume State 1
```

Thread is **not blocked**.

---

# 11. Difference between Task.Run and async.

```
Task.Run
```

Creates ThreadPool thread.

```
async
```

Does NOT create thread.

It simply waits without blocking.

---

# 12. Explain ConfigureAwait(false).

Normally

```
await

↓

Resume Original Context
```

With

```csharp
await service.GetAsync().ConfigureAwait(false);
```

Continuation can execute anywhere.

Useful

* libraries
* performance

Not usually needed in ASP.NET Core because there is no synchronization context.

---

# 13. Explain Thread Pool starvation.

Occurs when

```
Thread

↓

Blocked

↓

More Requests

↓

No Free Threads

↓

Slow Application
```

Avoid

❌ .Result

❌ Wait()

Use async all the way.

---

# 14. Explain CancellationToken.

```csharp
public async Task Process(CancellationToken token)
{
    while(!token.IsCancellationRequested)
    {
        await Task.Delay(1000);
    }
}
```

Benefits

* graceful shutdown
* avoid memory leaks
* stop unnecessary work

---

# 15. Explain React Virtual DOM.

```
State Changed

↓

Virtual DOM

↓

Diff Algorithm

↓

Compare Previous Tree

↓

Update Changed Nodes Only

↓

Browser DOM
```

Advantages

* Faster rendering
* Less DOM manipulation

---

# 16. Why does React re-render?

Reasons

* state changes
* props change
* parent renders
* context changes

---

# 17. Why does console.log execute twice?

Usually because

```
React StrictMode
```

Development only.

React intentionally renders twice to detect side effects.

Production renders once.

---

# 18. How to prevent unnecessary re-renders?

Use

```
React.memo

useMemo

useCallback

memoized selectors

lazy loading

virtualization
```

Example

```jsx
const Child = React.memo(({name})=>{
   return <div>{name}</div>;
});
```

---

# 19. Difference between useMemo and useCallback.

```
useMemo

returns VALUE
```

```jsx
const total = useMemo(()=>calculate(),[]);
```

```
useCallback

returns FUNCTION
```

```jsx
const save = useCallback(()=>{},[]);
```

---

# 20. Explain React.memo.

Without memo

Parent renders

↓

Child renders

↓

Grandchild renders

With memo

If props unchanged

↓

Skip render

---

# 21. Explain React reconciliation.

Steps

```
Old Virtual DOM

↓

New Virtual DOM

↓

Diff

↓

Update Changed Elements
```

---

# 22. How does React key improve performance?

Wrong

```jsx
index
```

Correct

```jsx
product.id
```

Keys allow React to correctly identify elements and minimize DOM operations.

---

# 23. Large list optimization?

Options

* react-window
* react-virtualized
* Pagination
* Infinite Scroll
* Memoization

---

# 24. Explain Redux flow.

```
Component

↓

Dispatch Action

↓

Reducer

↓

Store Updated

↓

React Re-render
```

---

# 25. Explain Normalization in Redux.

Instead of

```
Users

Posts

Comments
```

inside each other,

store separately.

```
Users

Posts

Comments
```

Lookup by IDs.

Much faster.

---

# 26. Explain Azure App Service deployment pipeline.

```
Developer

↓

Git Push

↓

Azure DevOps

↓

Build

↓

Run Tests

↓

Publish

↓

Deploy to Azure App Service

↓

Health Check

↓

Swap Slot
```

---

# 27. Explain Deployment Slots.

```
Production

↓

Staging Slot

↓

Deploy

↓

Test

↓

Swap
```

Zero downtime deployment.

---

# 28. How do you store secrets?

Never

```
appsettings.json
```

Use

Azure Key Vault

Managed Identity

---

# 29. Explain Azure Static Web Apps.

Ideal for

React

Angular

Vue

Benefits

* Global CDN
* SSL
* GitHub integration
* Serverless APIs
* Authentication

---

# 30. How would you optimize SQL queries?

* Proper indexes
* Covering indexes
* Avoid `SELECT *`
* Parameterized queries
* Batch updates
* Query execution plans
* Connection pooling
* Read replicas (where applicable)

---

# 31. What causes deadlocks in SQL Server, and how do you prevent them?

**Answer:**
Deadlocks occur when two transactions each hold a lock that the other needs.

Example:

* Transaction A updates `Orders`, then `Customers`.
* Transaction B updates `Customers`, then `Orders`.

**Prevention:**

* Access tables in the same order.
* Keep transactions short.
* Create proper indexes to reduce lock duration.
* Use appropriate isolation levels (e.g., `READ COMMITTED SNAPSHOT` when suitable).
* Implement retry logic for deadlock victims.

---

# 32. Explain EF Core tracking vs AsNoTracking.

**Tracking**

```csharp
var users = await context.Users.ToListAsync();
```

* Change tracking enabled.
* Suitable for updates.

**No Tracking**

```csharp
var users = await context.Users
    .AsNoTracking()
    .ToListAsync();
```

* Faster for read-only queries.
* Lower memory usage.

---

# 33. How would you handle 10,000 concurrent API requests?

**Answer:**

* Use async I/O throughout.
* Avoid blocking calls (`.Result`, `Wait()`).
* Optimize database queries and indexing.
* Cache frequently accessed data with Redis.
* Implement rate limiting.
* Use load balancing and horizontal scaling.
* Queue long-running work using Azure Service Bus or similar.
* Monitor with Application Insights and tune bottlenecks.

---

# 34. What is idempotency, and why is it important?

**Answer:**

An idempotent operation produces the same result even if it is executed multiple times.

Example:

* A payment API receives the same request twice due to a network retry.

**Solution:**

* Generate an idempotency key on the client.
* Store processed keys in the database/cache.
* Return the previous response if the key already exists, avoiding duplicate processing.

---

# 35. How do you troubleshoot a slow production application?

Typical approach:

1. Check application metrics (CPU, memory, response times).
2. Review logs and exceptions.
3. Identify slow endpoints with Application Insights.
4. Analyze SQL execution plans.
5. Check external dependencies.
6. Capture memory dumps if needed.
7. Profile the application.
8. Verify infrastructure scaling and network latency.

---

These questions cover the core competencies in the JD: **ASP.NET Core, C#, JWT, EF Core, React performance, asynchronous programming, scheduling, Azure deployment, SQL optimization, and production-grade troubleshooting.** They are at the level typically expected for a senior full-stack engineer with 5–8+ years of experience.
