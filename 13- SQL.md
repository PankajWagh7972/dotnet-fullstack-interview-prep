For **5+ years of .NET Full Stack Developer interviews**, companies like TCS, Accenture, Deloitte, JP Morgan, Microsoft, Oracle, Hitachi, and product companies usually don't ask basic SQL questions (SELECT, JOIN, GROUP BY). Instead, they focus on **query optimization, indexing, execution plans, transactions, locking, window functions, CTEs, and real-world SQL problems.**

Here are **advanced SQL interview questions with detailed answers.**

---

# 1. What is the difference between ROW_NUMBER(), RANK(), and DENSE_RANK()?

Suppose we have:

| Employee | Salary |
| -------- | ------ |
| A        | 100    |
| B        | 100    |
| C        | 90     |
| D        | 80     |

### ROW_NUMBER()

```sql
SELECT *,
ROW_NUMBER() OVER(ORDER BY Salary DESC) AS RN
FROM Employee;
```

Result

```
A 100 1
B 100 2
C 90 3
D 80 4
```

Every row gets a unique number.

---

### RANK()

```
A 100 1
B 100 1
C 90 3
D 80 4
```

Notice Rank 2 is skipped.

---

### DENSE_RANK()

```
A 100 1
B 100 1
C 90 2
D 80 3
```

No gaps.

**Interview Question**

Find the 3rd highest salary.

```sql
SELECT *
FROM
(
SELECT *,
DENSE_RANK() OVER(ORDER BY Salary DESC) AS DR
FROM Employee
)t
WHERE DR=3;
```

---

# 2. Difference between CTE and Temporary Table

### CTE

```sql
WITH EmployeeCTE AS
(
SELECT *
FROM Employee
WHERE Salary>50000
)
SELECT *
FROM EmployeeCTE;
```

Advantages

* Readability
* Recursive queries
* No physical storage

Disadvantages

* Exists only during query execution

---

### Temp Table

```sql
CREATE TABLE #Employee
(
Id INT,
Name VARCHAR(100)
)

INSERT INTO #Employee
SELECT *
FROM Employee
```

Advantages

* Can create indexes
* Better for large datasets
* Reusable in multiple queries

---

Interview answer:

Use CTE for readability and recursion.

Use Temp Tables for complex processing involving millions of records.

---

# 3. Explain Execution Plan

SQL Server converts SQL into an Execution Plan.

Example

```sql
SELECT *
FROM Employee
WHERE EmployeeId=10;
```

Execution Plan may show

```
Clustered Index Seek
```

or

```
Table Scan
```

Seek is fast.

Scan is slow.

---

Interview Question

Why is Table Scan slow?

Because SQL Server reads every row.

Complexity

```
O(n)
```

Whereas Index Seek is

```
O(log n)
```

---

# 4. What is Parameter Sniffing?

Suppose procedure

```sql
CREATE PROCEDURE GetEmployee
@DepartmentId INT
AS
SELECT *
FROM Employee
WHERE DepartmentId=@DepartmentId
```

First call

```sql
EXEC GetEmployee 1
```

Department 1 has 5 rows.

Execution plan gets cached.

Next

```sql
EXEC GetEmployee 100
```

Department 100 has

```
500000 rows
```

SQL still uses previous execution plan.

Performance becomes terrible.

Solutions

```
OPTION(RECOMPILE)

OPTIMIZE FOR UNKNOWN

Local variables

Query hints
```

---

# 5. What is SARGable Query?

Bad

```sql
SELECT *
FROM Employee
WHERE YEAR(HireDate)=2025;
```

Index cannot be used.

Good

```sql
SELECT *
FROM Employee
WHERE HireDate
BETWEEN '2025-01-01'
AND '2025-12-31';
```

Uses Index Seek.

---

# 6. Explain Covering Index

Table

```
Employee

Id

Name

Salary

Department
```

Query

```sql
SELECT Name,Salary
FROM Employee
WHERE Department='IT'
```

Index

```sql
CREATE INDEX IX_Employee_Department

ON Employee(Department)

INCLUDE(Name,Salary)
```

Now SQL Server doesn't touch table.

Everything comes from index.

Much faster.

---

# 7. Clustered vs Non-Clustered Index

Clustered

```
Actual data sorted
```

Only one.

Non Clustered

Separate B-tree.

Multiple allowed.

Example

Clustered

```
EmployeeId
```

Non Clustered

```
Department

Salary

Email
```

---

# 8. What are Window Functions?

Example

Running Total

```sql
SELECT

Name,

Salary,

SUM(Salary)
OVER
(
ORDER BY EmployeeId
) RunningTotal

FROM Employee;
```

Result

```
10000

25000

45000

70000
```

---

Moving Average

```sql
AVG(Salary)
OVER
(
ORDER BY EmployeeId
ROWS BETWEEN 2 PRECEDING
AND CURRENT ROW
)
```

---

# 9. Delete Duplicate Records

Table

```
1 A

2 A

3 B

4 B

5 C
```

```sql
WITH CTE AS
(
SELECT *,
ROW_NUMBER()
OVER
(
PARTITION BY Name
ORDER BY Id
) RN
FROM Employee
)

DELETE
FROM CTE
WHERE RN>1;
```

---

# 10. Find Nth Highest Salary

```sql
DECLARE @N INT=5;

SELECT Salary
FROM
(
SELECT Salary,
DENSE_RANK()
OVER
(
ORDER BY Salary DESC
) DR
FROM Employee
)t
WHERE DR=@N;
```

---

# 11. Explain Isolation Levels

### Read Uncommitted

Dirty Reads allowed.

Fastest.

---

### Read Committed

Default.

No dirty reads.

---

### Repeatable Read

Prevents updates while reading.

---

### Serializable

Highest isolation.

No phantom reads.

Slowest.

---

### Snapshot

Uses Row Versioning.

No blocking.

Excellent for reporting.

---

# 12. Deadlock Example

Transaction 1

```sql
UPDATE Employee
SET Salary=1000
WHERE Id=1;
```

Waiting...

Transaction 2

```sql
UPDATE Employee
SET Salary=2000
WHERE Id=2;
```

Then

Transaction1

```
Needs Id=2
```

Transaction2

```
Needs Id=1
```

Deadlock.

SQL Server kills one transaction.

Avoid

* Same table access order
* Short transactions
* Proper indexes

---

# 13. CROSS APPLY vs OUTER APPLY

Suppose function

```sql
GetOrders(CustomerId)
```

Cross Apply

Returns only matching rows.

Outer Apply

Returns all customers.

Equivalent

```
CROSS APPLY = INNER JOIN

OUTER APPLY = LEFT JOIN
```

---

# 14. EXISTS vs IN

```sql
SELECT *
FROM Employee e
WHERE EXISTS
(
SELECT 1
FROM Orders o
WHERE o.EmployeeId=e.Id
)
```

Better for large datasets.

Stops after first match.

---

IN

```sql
WHERE Id IN
(
SELECT EmployeeId
FROM Orders
)
```

Can be slower.

---

# 15. MERGE Statement

```sql
MERGE Employee AS Target

USING EmployeeTemp AS Source

ON Target.Id=Source.Id

WHEN MATCHED

THEN

UPDATE SET

Target.Name=Source.Name

WHEN NOT MATCHED

THEN

INSERT(Id,Name)

VALUES(Source.Id,Source.Name);
```

Useful for synchronization.

---

# 16. How do you identify slow queries?

Use:

* SQL Server Profiler (legacy)
* Extended Events
* Query Store
* Dynamic Management Views (DMVs)

```sql
SELECT TOP 10
    qs.execution_count,
    qs.total_elapsed_time / qs.execution_count AS AvgElapsedTime,
    st.text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY AvgElapsedTime DESC;
```

---

# 17. What is Index Fragmentation?

As inserts, updates, and deletes happen, index pages become fragmented, causing slower reads.

Check fragmentation:

```sql
SELECT
    OBJECT_NAME(object_id) AS TableName,
    avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED');
```

Fix:

* `ALTER INDEX ... REORGANIZE` (5–30% fragmentation)
* `ALTER INDEX ... REBUILD` (>30% fragmentation)

---

# 18. What are Common Table Expressions (CTEs) used for besides readability?

* Recursive queries (e.g., organization hierarchy)
* Breaking complex queries into logical steps
* Replacing derived tables
* Deleting duplicates
* Pagination (older SQL Server versions)

Recursive example:

```sql
WITH EmployeeHierarchy AS
(
    SELECT Id, ManagerId, Name
    FROM Employee
    WHERE ManagerId IS NULL

    UNION ALL

    SELECT e.Id, e.ManagerId, e.Name
    FROM Employee e
    INNER JOIN EmployeeHierarchy h
        ON e.ManagerId = h.Id
)
SELECT *
FROM EmployeeHierarchy;
```

---

# 19. Explain Query Optimization Techniques

Common ways to improve performance:

* Create appropriate indexes.
* Avoid `SELECT *`.
* Filter data as early as possible.
* Write SARGable predicates.
* Avoid functions on indexed columns.
* Keep statistics updated.
* Examine execution plans.
* Minimize unnecessary joins.
* Use set-based operations instead of cursors where possible.

---

# 20. SQL Scenario Question

**Question:** You have an `Orders` table with 100 million records. A report filtering by `OrderDate` and `CustomerId` takes 30 seconds. How would you optimize it?

**Answer:**

1. Check the execution plan.
2. Ensure indexes exist on `OrderDate` and `CustomerId` (possibly a composite index based on query pattern).
3. Rewrite non-SARGable predicates.
4. Avoid `SELECT *`.
5. Update statistics.
6. Consider partitioning if the table is very large and queries are date-based.
7. Use Query Store/DMVs to compare execution plans.
8. Evaluate whether parameter sniffing is affecting performance.
9. Archive or partition old data if appropriate.
10. Test changes and measure execution time before and after.

---

## Frequently Asked Advanced SQL Interview Topics (5–8 Years)

* Window Functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`)
* Recursive CTEs
* Index Internals (Clustered/Non-Clustered/Covering/Filtered)
* Execution Plans
* Query Optimization
* Parameter Sniffing
* Transactions and Isolation Levels
* Locking, Blocking, and Deadlocks
* Partitioning
* Query Store and DMVs
* Temp Tables vs Table Variables vs CTEs
* MERGE statement
* Dynamic SQL
* APPLY operators
* XML/JSON support in SQL Server
* SQL Server Agent and job scheduling
* Backup and Restore basics
* High Availability concepts (Always On Availability Groups, Log Shipping, Replication)
* Advanced string and date manipulation
* Real-world SQL performance troubleshooting scenarios

These are among the most commonly discussed advanced SQL topics in interviews for experienced .NET developers and SQL Server professionals.
