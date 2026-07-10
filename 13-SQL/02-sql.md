Table-Valued Parameters (TVPs) are one of the **most frequently asked SQL Server interview topics** for experienced .NET developers. They are commonly used when you need to send **multiple rows from your .NET application to a stored procedure in a single database call**.

Typical interview questions include:

* "How do you insert 10,000 records efficiently from .NET?"
* "How do you pass a list of IDs to a stored procedure?"
* "What is a Table-Valued Parameter?"
* "When would you use TVPs instead of JSON or XML?"

Let's go through it step by step.

---

# What is a Table-Valued Parameter (TVP)?

A TVP is a **user-defined table type** that allows you to pass an entire table (multiple rows) as a parameter to a stored procedure or function.

Instead of calling the database 1000 times:

```text
.NET

↓

Insert Employee 1

↓

Insert Employee 2

↓

Insert Employee 3

↓

1000 Database Calls
```

You do

```text
.NET

↓

Create DataTable

↓

Pass DataTable

↓

Stored Procedure

↓

1 Database Call
```

This is much faster.

---

# Real-Time Scenario

Suppose HR uploads an Excel file containing **500 employees**.

Without TVP

```text
foreach(employee)
{
    Execute Stored Procedure
}
```

Database Calls

```text
500
```

---

With TVP

```text
Create DataTable

↓

Pass DataTable

↓

Stored Procedure

↓

Insert All Records
```

Database Calls

```text
1
```

---

# Step 1 Create Employee Table

```sql
CREATE TABLE Employee
(
    EmployeeId INT IDENTITY PRIMARY KEY,
    Name NVARCHAR(100),
    Salary DECIMAL(18,2),
    DepartmentId INT
);
```

---

# Step 2 Create User Defined Table Type

```sql
CREATE TYPE EmployeeType AS TABLE
(
    Name NVARCHAR(100),
    Salary DECIMAL(18,2),
    DepartmentId INT
);
```

Think of this as

```text
Custom DataTable inside SQL Server
```

---

# Step 3 Create Stored Procedure

Notice

```sql
READONLY
```

TVPs are always READONLY.

```sql
CREATE PROCEDURE usp_InsertEmployees
(
    @Employees EmployeeType READONLY
)
AS
BEGIN

INSERT INTO Employee
(
    Name,
    Salary,
    DepartmentId
)

SELECT
Name,
Salary,
DepartmentId

FROM @Employees;

END
```

---

# SQL Test

```sql
DECLARE @Emp EmployeeType;

INSERT INTO @Emp
VALUES
('John',50000,1),
('Mike',60000,2),
('David',55000,3);

EXEC usp_InsertEmployees @Emp;
```

Employee table

| Id | Name  | Salary |
| -- | ----- | ------ |
| 1  | John  | 50000  |
| 2  | Mike  | 60000  |
| 3  | David | 55000  |

---

# EF Core Implementation

Employee Model

```csharp
public class Employee
{
    public string Name { get; set; }

    public decimal Salary { get; set; }

    public int DepartmentId { get; set; }
}
```

---

## Convert List into DataTable

```csharp
using System.Data;

public static DataTable ToDataTable(List<Employee> employees)
{
    var table = new DataTable();

    table.Columns.Add("Name", typeof(string));
    table.Columns.Add("Salary", typeof(decimal));
    table.Columns.Add("DepartmentId", typeof(int));

    foreach (var employee in employees)
    {
        table.Rows.Add(
            employee.Name,
            employee.Salary,
            employee.DepartmentId);
    }

    return table;
}
```

---

# Create SqlParameter

```csharp
var employees = new List<Employee>
{
    new Employee
    {
        Name="John",
        Salary=50000,
        DepartmentId=1
    },

    new Employee
    {
        Name="Mike",
        Salary=60000,
        DepartmentId=2
    }
};

var table = ToDataTable(employees);

var parameter = new SqlParameter("@Employees", table)
{
    SqlDbType = SqlDbType.Structured,
    TypeName = "dbo.EmployeeType"
};
```

The important properties are:

```csharp
SqlDbType.Structured
```

and

```csharp
TypeName = "dbo.EmployeeType"
```

---

# Execute Stored Procedure (EF Core)

```csharp
await _context.Database.ExecuteSqlRawAsync(
    "EXEC usp_InsertEmployees @Employees",
    parameter);
```

That's it.

EF Core sends the DataTable as a TVP.

---

# Flow Diagram

```text
React

↓

API

↓

List<Employee>

↓

Convert To DataTable

↓

SqlParameter

↓

EmployeeType (SQL Type)

↓

Stored Procedure

↓

Insert

↓

Employee Table
```

---

# Passing List of IDs (Very Common)

Instead of passing employees

Pass IDs.

Create Type

```sql
CREATE TYPE IdListType AS TABLE
(
    Id INT
);
```

Procedure

```sql
CREATE PROCEDURE usp_DeleteEmployees

@Ids IdListType READONLY

AS

DELETE

FROM Employee

WHERE EmployeeId IN

(
SELECT Id
FROM @Ids
)
```

.NET

```csharp
var table = new DataTable();
table.Columns.Add("Id", typeof(int));

table.Rows.Add(1);
table.Rows.Add(5);
table.Rows.Add(10);
```

Very common in

* Bulk Delete
* Bulk Update
* Bulk Approval

---

# Advantages

* Only one database call.
* Excellent performance for bulk operations.
* Strongly typed.
* Easier to work with than XML.
* Lower parsing overhead than JSON/XML for SQL Server.

---

# Limitations

* TVPs are **READONLY**. You cannot update or delete rows inside the parameter itself.
* Supported only by SQL Server.
* Requires creating a user-defined table type in the database.
* Large TVPs consume memory, so batching may still be appropriate for extremely large datasets.

---

# TVP vs JSON vs XML

| Feature              | TVP       | JSON | XML     |
| -------------------- | --------- | ---- | ------- |
| Performance          | ⭐⭐⭐⭐⭐     | ⭐⭐⭐  | ⭐⭐      |
| Strongly Typed       | ✅         | ❌    | ❌       |
| SQL Parsing Required | ❌         | ✅    | ✅       |
| Bulk Insert          | Excellent | Good | Average |
| SQL Server Specific  | ✅         | ❌    | ❌       |

---

# Interview Questions

### 1. Why use a TVP instead of a loop?

A TVP sends all rows in a single round trip to SQL Server, greatly reducing network overhead and improving performance.

---

### 2. Why must a TVP be `READONLY`?

SQL Server does not allow modifications to the rows of a table-valued parameter inside the stored procedure. If you need to modify the data, copy it into a temporary table first.

---

### 3. When would you use a TVP?

* Bulk inserts
* Bulk updates
* Bulk deletes
* Passing a list of IDs
* Importing Excel or CSV data
* Batch approval workflows

---

### 4. Can EF Core map a TVP automatically?

No. EF Core does not have built-in mapping for TVPs. The common approach is to create a `DataTable` (or use `IEnumerable<SqlDataRecord>` for advanced scenarios), wrap it in a `SqlParameter` with `SqlDbType.Structured`, and execute the stored procedure using `ExecuteSqlRaw` or ADO.NET.

---

### 5. TVP vs `SqlBulkCopy`—which should you use?

| TVP                                                                                   | `SqlBulkCopy`                                                                         |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Best for passing data to stored procedures that include business logic or validation. | Best for inserting very large datasets directly into a table with maximum throughput. |
| Supports insert, update, delete, and merge logic.                                     | Insert-only into a destination table.                                                 |
| Good for thousands to tens of thousands of rows.                                      | Ideal for hundreds of thousands or millions of rows.                                  |

**Rule of thumb:** Use **TVPs** when you need stored procedure logic; use **`SqlBulkCopy`** when raw insert performance is the primary goal.
