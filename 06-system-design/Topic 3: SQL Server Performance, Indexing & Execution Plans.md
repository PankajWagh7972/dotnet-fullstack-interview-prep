Perfect. The **next topic is the one that interviewers spend the most time on**, especially for 5+ years.

# ⭐ Topic 3: SQL Server Performance, Indexing & Execution Plans

If you answer this topic well, the interviewer usually assumes you have good backend experience.

---

# Q1. My API is slow. Where do you start?

This is one of the most common interview questions.

## ❌ Wrong Answer

> I'll add an index.

Don't say this.

---

## ✅ Correct Answer

> Before making changes, I identify where the bottleneck is. I first determine whether the issue is in the API, SQL Server, network, or an external dependency.

My investigation order is:

```text
Client

↓

API

↓

Business Logic

↓

Entity Framework

↓

SQL Server

↓

Disk / Network
```

I use:

* Application Insights
* SQL Execution Plan
* SQL Profiler
* Query Store
* Performance Monitor

---

## Follow-up

**Interviewer**

How do you know SQL is slow?

### Answer

Indicators include:

* High SQL CPU
* High Duration
* Long waits
* Slow execution plan
* Many logical reads

---

# Q2. What is an Index?

## Interview Answer

> An index is a data structure that allows SQL Server to locate rows quickly without scanning the entire table.

Think of it like the index at the back of a book.

Without the index, you'd read every page.

With the index, you jump directly to the topic.

---

## Example

Employee table

```text
Id

Name

Department
```

Without Index

```text
Find Employee

↓

Read Row 1

↓

Read Row 2

↓

Read Row 3

↓

...

↓

Read 10 Million Rows
```

---

With Index

```text
Employee Index

↓

Jump Directly

↓

Required Row
```

Huge performance improvement.

---

# Q3. What happens without an Index?

Suppose

```sql
SELECT *
FROM Employee
WHERE EmployeeId=100
```

No Index

```text
SQL

↓

Table Scan

↓

10 Million Rows
```

Execution

8 Seconds

---

With Index

```text
SQL

↓

Index Seek

↓

One Row
```

Execution

20 milliseconds

---

# Follow-up

How much improvement?

Sometimes

100x

Sometimes

1000x

Depends on data.

---

# Q4. Clustered Index

Interview Answer

A Clustered Index determines how data is physically stored on disk.

Only ONE Clustered Index is allowed because data can only be stored in one order.

---

Example

```text
Employee

1

2

3

4

5
```

Stored physically.

---

Usually

Primary Key

↓

Clustered Index.

---

# Follow-up

Why only one?

Because a table can only have one physical order.

---

# Q5. Non-Clustered Index

Interview Answer

A Non-Clustered Index stores keys separately and points to the actual table rows.

Think of it like a book index.

The index tells you which page to open.

---

Example

```text
Name

↓

Pointer

↓

Employee Table
```

---

You can create

100+

Non-clustered indexes.

---

# Follow-up

Which is faster?

Depends.

If query needs only indexed columns

↓

Non-clustered can be faster.

---

# Q6. What is an Execution Plan?

Interview Answer

SQL Server generates an Execution Plan before running a query.

It decides

* Which index to use
* Join order
* Scan vs Seek
* Estimated cost

Think of it as SQL Server's roadmap.

---

# Q7. Table Scan vs Index Seek

This is almost always asked.

## Table Scan

```text
100 Million Rows

↓

Read All Rows

↓

Find One
```

Slow.

---

## Index Seek

```text
100 Million Rows

↓

Jump

↓

One Row
```

Fast.

---

Interview Tip

Always mention

> "We should aim for Index Seek instead of Table Scan whenever appropriate."

---

# Q8. Why doesn't SQL use my Index?

Excellent senior question.

Possible reasons:

* Small table (scan may be cheaper)
* Function used on indexed column
* Data type mismatch
* Outdated statistics
* Query optimizer estimates a scan is more efficient

---

Example

Wrong

```sql
WHERE YEAR(OrderDate)=2025
```

SQL cannot efficiently use an index on `OrderDate`.

Better

```sql
WHERE OrderDate >= '2025-01-01'
AND OrderDate < '2026-01-01'
```

This is **SARGable** (Search ARGument Able).

---

# Follow-up

What is SARGable?

A query that allows SQL Server to use indexes efficiently.

---

# Q9. Why avoid SELECT *?

Wrong

```sql
SELECT *
FROM Employee
```

Reads

* Name
* Salary
* Address
* Photo
* Notes

Everything.

---

Better

```sql
SELECT Name,
Department
FROM Employee
```

Only required columns.

Less memory.

Less network.

Faster.

---

# Q10. Covering Index

Suppose

```sql
SELECT Name
FROM Employee
WHERE EmployeeId=10
```

Index

```text
EmployeeId

Name
```

SQL gets everything from the index.

No table lookup.

Very fast.

---

# Q11. Included Columns

Example

```sql
CREATE INDEX IX_EMP
ON Employee(EmployeeId)
INCLUDE(Name,Department)
```

Why?

Filter

↓

EmployeeId

Return

↓

Name

Department

Without touching table.

---

# Follow-up

Difference between Key Column and Include Column?

| Key Column                  | Included Column            |
| --------------------------- | -------------------------- |
| Used for searching          | Stored only for retrieval  |
| Affects index order         | Doesn't affect order       |
| Used in WHERE/JOIN/ORDER BY | Helps cover SELECT columns |

---

# Q12. Composite Index

Example

```sql
WHERE City='Pune'

AND Department='IT'
```

Index

```sql
(City,Department)
```

SQL uses both efficiently.

---

Interview Tip

Order matters.

```sql
(City,Department)
```

works for

```sql
WHERE City='Pune'
```

But

```sql
WHERE Department='IT'
```

may not use the index efficiently.

This is called the **Leftmost Prefix Rule**.

---

# Q13. SQL Suddenly Becomes Slow

Interviewer

Yesterday

100 ms

Today

5 sec

Why?

---

Possible reasons

* Missing index
* Statistics outdated
* Blocking
* Deadlock
* Parameter Sniffing
* Large data growth
* Execution plan changed

---

# Q14. Parameter Sniffing

One of the favorite senior questions.

Suppose

First execution

```sql
CustomerId=1
```

1 row

SQL creates plan.

Later

```sql
CustomerId=1000
```

5 million rows

Same plan reused.

Performance poor.

---

Solutions

* `OPTION (RECOMPILE)`
* Optimize query
* Local variables (use judiciously)
* Query Store plan forcing (when appropriate)

---

# Q15. SQL Blocking

Transaction A

```sql
UPDATE Employee
```

Transaction B

```sql
UPDATE Employee
```

Second transaction waits.

This is **Blocking**.

---

# Q16. Deadlock

Transaction A

Locks Employee

Needs Salary

Transaction B

Locks Salary

Needs Employee

```text
Employee

↓

Salary

↑

Employee
```

Neither can continue.

SQL chooses one as the deadlock victim and rolls it back.

---

# Follow-up

How to avoid deadlocks?

* Keep transactions short.
* Access tables in the same order.
* Use proper indexes.
* Avoid unnecessary locks.
* Retry deadlock victims in application code.

---

# Q17. Pagination

Wrong

```sql
SELECT *
FROM Orders
```

Returns

10 Million Rows.

---

Better

```sql
SELECT *
FROM Orders
ORDER BY Id
OFFSET 0 ROWS
FETCH NEXT 50 ROWS ONLY;
```

---

Even Better (Large Tables)

**Keyset Pagination**

```sql
SELECT TOP (50) *
FROM Orders
WHERE Id > @LastId
ORDER BY Id;
```

This avoids scanning skipped rows and performs better for very large datasets.

---

# Q18. EF Core Performance

Wrong

```csharp
_context.Products.ToList();
```

Better

```csharp
_context.Products
.AsNoTracking()
.Select(x=>new ProductDto
{
    Id=x.Id,
    Name=x.Name
})
.ToList();
```

Benefits:

* No tracking
* Less memory
* Less SQL data
* Faster serialization

---

# ⭐ Senior Interview Question

Interviewer

Our Product table has

50 Million Rows.

API is slow.

What will you do?

---

Perfect Answer

1. Analyze Execution Plan.
2. Check for Table Scans.
3. Add or improve indexes.
4. Return only required columns.
5. Implement pagination.
6. Use `AsNoTracking()` for read-only queries.
7. Add Redis caching for frequently accessed data.
8. Consider read replicas for heavy read workloads.
9. Review statistics and index fragmentation.
10. Monitor using Query Store and Application Insights.

This answer covers the application, ORM, database, and infrastructure layers.

---

# ⭐ One Important Topic You Should Know

Interviewers at companies like **Grant Thornton**, **Deloitte**, **Microsoft**, **JP Morgan**, and **Accenture** often ask:

> **"Suppose your application has 10 million users. Design it."**

To answer well, you need to connect everything you've learned:

* Load Balancer
* Horizontal Scaling
* Redis
* SQL Indexing
* Read Replicas
* Async Processing
* Background Jobs
* Azure Service Bus
* Rate Limiting
* CDN
* Monitoring
* Circuit Breaker
* Application Insights

This is called **System Design**, and it's often the deciding factor for senior .NET interviews.

## ⭐ My Recommendation

The **next topic** should be:

> **Design a Scalable E-Commerce System in ASP.NET Core + SQL Server + Azure**

We'll cover:

* API design
* Authentication
* Load balancer
* Redis
* SQL optimization
* Read replicas
* Service Bus
* Background workers
* File storage
* Payment integration
* Monitoring
* Azure architecture

That single example ties together nearly all of the concepts interviewers expect from a 5+ year .NET Full Stack developer.
