> Q.01 **Find all customers from India who, during the last 365 days, have placed more than 5 orders and whose average order value exceeds $1,000.**

If you also need the corresponding SQL query, here's one way to write it:

```sql
SELECT c.customerid, c.name, c.region,
       COUNT(o.customerid) AS order_count,
       AVG(o.totalamount) AS avg_order_value
FROM Customer c
JOIN "Order" o ON c.customerid = o.customerid
WHERE c.region = 'India'
  AND o.orderdate >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY c.customerid, c.name, c.region
HAVING COUNT(o.customerid) > 5
   AND AVG(o.totalamount) > 1000;
```

*Note: The table name "Order" is a reserved keyword in SQL; using double quotes or brackets may be required depending on your database.*

Here's the LINQ (C#) equivalent of the query, using Entity Framework Core syntax:

```csharp
using Microsoft.EntityFrameworkCore;
using System;
using System.Linq;

// Assuming you have a DbContext with DbSet<Customer> and DbSet<Order>
var oneYearAgo = DateTime.Now.AddYears(-1);

var result = await context.Customers
    .Where(c => c.Region == "India")
    .Join(context.Orders,
          customer => customer.CustomerId,
          order => order.CustomerId,
          (customer, order) => new { customer, order })
    .Where(joined => joined.order.OrderDate >= oneYearAgo)
    .GroupBy(joined => new { joined.customer.CustomerId, joined.customer.Name, joined.customer.Region })
    .Select(group => new
    {
        group.Key.CustomerId,
        group.Key.Name,
        group.Key.Region,
        OrderCount = group.Count(),
        AvgOrderValue = group.Average(g => g.order.TotalAmount)
    })
    .Where(g => g.OrderCount > 5 && g.AvgOrderValue > 1000)
    .ToListAsync();
```

### Alternative with Navigation Property (if configured)

If your `Customer` entity has a navigation collection `Orders`, you can write a cleaner version:

```csharp
var oneYearAgo = DateTime.Now.AddYears(-1);

var result = await context.Customers
    .Where(c => c.Region == "India")
    .Select(c => new
    {
        Customer = c,
        RecentOrders = c.Orders.Where(o => o.OrderDate >= oneYearAgo)
    })
    .Where(x => x.RecentOrders.Count() > 5)
    .Select(x => new
    {
        x.Customer.CustomerId,
        x.Customer.Name,
        x.Customer.Region,
        OrderCount = x.RecentOrders.Count(),
        AvgOrderValue = x.RecentOrders.Average(o => o.TotalAmount)
    })
    .Where(x => x.AvgOrderValue > 1000)
    .ToListAsync();
```

### Query Syntax Version (for completeness)

```csharp
var oneYearAgo = DateTime.Now.AddYears(-1);

var query = from c in context.Customers
            join o in context.Orders on c.CustomerId equals o.CustomerId
            where c.Region == "India" && o.OrderDate >= oneYearAgo
            group o by new { c.CustomerId, c.Name, c.Region } into g
            let orderCount = g.Count()
            let avgAmount = g.Average(o => o.TotalAmount)
            where orderCount > 5 && avgAmount > 1000
            select new
            {
                g.Key.CustomerId,
                g.Key.Name,
                g.Key.Region,
                OrderCount = orderCount,
                AvgOrderValue = avgAmount
            };

var result = await query.ToListAsync();
```

**Note:**  
- Replace `context` with your actual `DbContext` instance.  
- Use `DateTime.UtcNow` instead of `DateTime.Now` if you prefer UTC time for consistency.  
- Ensure `TotalAmount` and `OrderDate` property names match your actual `Order` entity.

For SQL Server specifically, here are the most reliable approaches to synchronize data from a `Source` table into a `Target` table—inserting new rows and updating existing ones based on a key column.

---
> Q.02 how to perform an upsert (update existing rows, insert new rows) from a source table to a target table in SQL, with examples for the most common database systems.
### Recommended: `MERGE` Statement (Most Common)

```sql
MERGE INTO Target AS T
USING Source AS S
ON T.Id = S.Id   -- Match on primary/unique key

WHEN MATCHED THEN
    UPDATE SET 
        T.Column1 = S.Column1,
        T.Column2 = S.Column2,
        T.ModifiedDate = GETDATE()

WHEN NOT MATCHED BY TARGET THEN
    INSERT (Id, Column1, Column2, CreatedDate, ModifiedDate)
    VALUES (S.Id, S.Column1, S.Column2, GETDATE(), GETDATE());
```

**Optional:** If you also need to delete rows in `Target` that no longer exist in `Source`, add:

```sql
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

---

### ⚠️ Important `MERGE` Considerations in SQL Server

- **Concurrency:** Under high concurrency, `MERGE` can encounter race conditions. Use `WITH (HOLDLOCK)` on the target table to prevent issues:

```sql
MERGE INTO Target WITH (HOLDLOCK) AS T
...
```

- **Performance:** For very large source tables, consider batching or using a temporary table with indexes.

---

### Alternative 1: Two‑Step `UPDATE` + `INSERT` (Avoids `MERGE` Nuances)

This method is straightforward and often just as performant:

```sql
BEGIN TRANSACTION;

-- 1. Update rows that exist in both tables
UPDATE T
SET 
    T.Column1 = S.Column1,
    T.Column2 = S.Column2,
    T.ModifiedDate = GETDATE()
FROM Target T
INNER JOIN Source S ON T.Id = S.Id;

-- 2. Insert rows that only exist in Source
INSERT INTO Target (Id, Column1, Column2, CreatedDate, ModifiedDate)
SELECT S.Id, S.Column1, S.Column2, GETDATE(), GETDATE()
FROM Source S
WHERE NOT EXISTS (SELECT 1 FROM Target T WHERE T.Id = S.Id);

COMMIT TRANSACTION;
```

---

### Alternative 2: Using `OUTPUT` to Track Changes (If You Need an Audit Log)

```sql
DECLARE @Changes TABLE (Action NVARCHAR(10), Id INT);

MERGE INTO Target AS T
USING Source AS S
ON T.Id = S.Id

WHEN MATCHED THEN
    UPDATE SET 
        T.Column1 = S.Column1,
        T.Column2 = S.Column2,
        T.ModifiedDate = GETDATE()

WHEN NOT MATCHED THEN
    INSERT (Id, Column1, Column2, CreatedDate, ModifiedDate)
    VALUES (S.Id, S.Column1, S.Column2, GETDATE(), GETDATE())

OUTPUT $action, INSERTED.Id INTO @Changes;

-- View what was done
SELECT Action, Id FROM @Changes;
```

---

### Full Working Example with Sample Data

```sql
-- Sample tables
CREATE TABLE Target (
    Id INT PRIMARY KEY,
    Name NVARCHAR(50),
    Amount DECIMAL(10,2),
    LastUpdated DATETIME
);

CREATE TABLE Source (
    Id INT PRIMARY KEY,
    Name NVARCHAR(50),
    Amount DECIMAL(10,2)
);

-- Populate with test data
INSERT INTO Target VALUES (1, 'Old A', 100.00, GETDATE());
INSERT INTO Source VALUES (1, 'New A', 150.00), (2, 'New B', 200.00);

-- Perform upsert
MERGE INTO Target AS T
USING Source AS S
ON T.Id = S.Id

WHEN MATCHED THEN
    UPDATE SET 
        T.Name = S.Name,
        T.Amount = S.Amount,
        T.LastUpdated = GETDATE()

WHEN NOT MATCHED THEN
    INSERT (Id, Name, Amount, LastUpdated)
    VALUES (S.Id, S.Name, S.Amount, GETDATE());

-- Verify results
SELECT * FROM Target;
```

**Result:**  
- Row `Id=1` is updated to `'New A'` and `150.00`.  
- Row `Id=2` is inserted with `'New B'` and `200.00`.

---

> Q.03 Here's how to find the missing number in a sequence of consecutive integers where exactly one number is missing, using C#.

---

### Problem Understanding
Given an array like `[1, 3, 4]` → missing **2**  
Given `[4, 5, 6, 8]` → missing **7**

Assumption: The array contains **n** distinct integers that should form a consecutive range from `Min` to `Max`, with exactly **one** number missing.

---

### C# Solution 1: Using Sum Formula (O(n) time, O(1) space)

```csharp
public static int FindMissingNumber(int[] arr)
{
    if (arr == null || arr.Length == 0)
        throw new ArgumentException("Array is empty or null");

    int min = arr.Min();
    int max = arr.Max();
    
    // Expected sum of numbers from min to max inclusive
    int n = max - min + 1;
    int expectedSum = n * (min + max) / 2;
    
    int actualSum = arr.Sum();
    
    return expectedSum - actualSum;
}
```

**Usage:**
```csharp
int[] arr1 = { 1, 3, 4 };
int missing1 = FindMissingNumber(arr1);  // 2

int[] arr2 = { 4, 5, 6, 8 };
int missing2 = FindMissingNumber(arr2);  // 7
```

---

### C# Solution 2: Using XOR (Bit Manipulation)

This method avoids potential integer overflow with large sums.

```csharp
public static int FindMissingNumberXOR(int[] arr)
{
    if (arr == null || arr.Length == 0)
        throw new ArgumentException("Array is empty or null");

    int min = arr.Min();
    int max = arr.Max();

    int xorFullRange = 0;
    for (int i = min; i <= max; i++)
        xorFullRange ^= i;

    int xorArray = 0;
    foreach (int num in arr)
        xorArray ^= num;

    return xorFullRange ^ xorArray;
}
```

---

### C# Solution 3: Sequential Scan (If Array is Sorted)

If the array is already sorted ascending, you can simply scan for the first gap:

```csharp
public static int FindMissingNumberSorted(int[] sortedArr)
{
    for (int i = 0; i < sortedArr.Length - 1; i++)
    {
        if (sortedArr[i + 1] != sortedArr[i] + 1)
            return sortedArr[i] + 1;
    }
    throw new Exception("No missing number found");
}
```

---

### Important Notes

- If the array has **more than one** missing number, the sum/XOR methods will not work correctly (they return the difference, not individual missing values).
- For multiple missing numbers, use a `HashSet` approach or sort and iterate.
- All methods assume the input contains only integers and the missing number is within the min-max range.

Let me know if you'd like a version that finds **all missing numbers** in an unsorted array!



> Q.04
> Here's how to design both the **SQL Server** side and the **C#** application layer to gracefully handle new column additions without breaking existing functionality or requiring immediate code changes.

---

## 1. SQL Server Side – Resilient Schema Design

### Option A: Use Views with `SELECT *` (Simple, but with Caution)
```sql
CREATE VIEW dbo.vw_DynamicTable
AS
SELECT * FROM dbo.MainTable;
```
**Caveats:**
- Adding columns to `MainTable` automatically includes them in the view.
- **Downside:** `SELECT *` is brittle if column order changes. Always pair with a `sp_refreshview` after DDL changes.
- For production, prefer explicit column lists in queries, but for **read‑only reporting** this works.

### Option B: Stored Procedures that Recompile on Each Execution
```sql
CREATE PROCEDURE dbo.GetDynamicData
WITH RECOMPILE   -- Ensures latest schema is used
AS
BEGIN
    SELECT * FROM dbo.MainTable;
END;
```
- `WITH RECOMPILE` forces a fresh compilation, picking up new columns immediately.

### Option C: Use a JSON Column for Extensible Attributes
This is the **most flexible** approach for dynamic fields.
```sql
CREATE TABLE dbo.Orders
(
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    OrderDate DATETIME,
    -- Fixed core columns
    DynamicAttributes NVARCHAR(MAX)   -- Stores JSON
    CONSTRAINT CK_Orders_Json CHECK (ISJSON(DynamicAttributes) = 1)
);
```
**Insert/Update Example:**
```sql
INSERT INTO dbo.Orders (OrderId, CustomerId, OrderDate, DynamicAttributes)
VALUES (1, 100, GETDATE(), '{"Discount":10.5, "PromoCode":"SAVE20", "NewField":"some value"}');
```
**Querying JSON:**
```sql
SELECT OrderId,
       JSON_VALUE(DynamicAttributes, '$.Discount') AS Discount,
       JSON_VALUE(DynamicAttributes, '$.PromoCode') AS PromoCode
FROM dbo.Orders;
```
- Adding a new attribute simply means including it in the JSON payload; no DDL change needed.

---

## 2. C# Application Side – Dynamic Model Handling

### Approach 1: Using `dynamic` and `ExpandoObject` with Dapper

Dapper can map query results directly to `dynamic` or `ExpandoObject`, allowing you to access columns by name.

```csharp
using Dapper;
using System.Data.SqlClient;

string connectionString = "...";
using var connection = new SqlConnection(connectionString);

// Query returns IEnumerable<dynamic>
var results = connection.Query("SELECT * FROM dbo.MainTable");

foreach (dynamic row in results)
{
    Console.WriteLine($"Id: {row.Id}");
    // Access a column that might have been added later
    // If the column doesn't exist, an exception is thrown, so use with care.
    if (HasProperty(row, "NewColumn"))
        Console.WriteLine($"NewColumn: {row.NewColumn}");
}

// Helper to check for property existence on dynamic object
static bool HasProperty(dynamic obj, string name)
{
    var dictionary = (IDictionary<string, object>)obj;
    return dictionary.ContainsKey(name);
}
```

### Approach 2: Using `JObject` (Newtonsoft.Json) for Maximum Flexibility

```csharp
using Newtonsoft.Json;
using Dapper;

var rows = connection.Query("SELECT * FROM dbo.MainTable FOR JSON PATH").FirstOrDefault();
if (rows != null)
{
    JArray array = JArray.Parse(rows);
    foreach (JObject item in array)
    {
        Console.WriteLine(item["Id"]);
        // Safely access any column, new or old
        if (item.ContainsKey("NewColumn"))
            Console.WriteLine(item["NewColumn"]);
    }
}
```

### Approach 3: Using `DataTable` – Old‑School but Still Effective

```csharp
using var adapter = new SqlDataAdapter("SELECT * FROM dbo.MainTable", connectionString);
DataTable dt = new DataTable();
adapter.Fill(dt);

foreach (DataRow row in dt.Rows)
{
    Console.WriteLine(row["Id"]);
    if (dt.Columns.Contains("NewColumn"))
        Console.WriteLine(row["NewColumn"]);
}
```

### Approach 4: Entity Framework Core with **Property Bag** (EF Core 5+)

Define an entity type as a dictionary (`Dictionary<string, object>`).

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.SharedTypeEntity<Dictionary<string, object>>("MainTable", b =>
    {
        b.Property<int>("Id");
        b.Property<string>("Name");
        // Add only core properties; others are automatically tracked
        // New columns will be available in the dictionary at runtime
    });
}

// Usage
var context = new MyDbContext();
var items = context.Set<Dictionary<string, object>>("MainTable").ToList();
foreach (var item in items)
{
    Console.WriteLine(item["Id"]);
    if (item.TryGetValue("NewColumn", out var newColValue))
        Console.WriteLine(newColValue);
}
```

---

## 3. Putting It All Together – Recommended Stack

| Layer           | Recommendation                                                                 |
|-----------------|---------------------------------------------------------------------------------|
| **SQL Storage** | Use a **JSON column** for true dynamic fields; keep core columns static.         |
| **C# Data Access** | Use **Dapper with `dynamic`** for quick development, or **EF Core Property Bags** for a more structured approach. |
| **API Output**  | Return **JSON** (e.g., `IActionResult` with anonymous objects) to naturally handle varying fields. |

### Example: Dynamic API Endpoint
```csharp
[HttpGet("orders")]
public IActionResult GetOrders()
{
    var data = connection.Query("SELECT * FROM dbo.Orders");
    return Ok(data); // Serializes to JSON with all columns present
}
```
- The JSON serializer will automatically include any new columns returned by the query.

---

## Summary

- **SQL Server:** `WITH RECOMPILE` stored procs or JSON columns for extensibility.
- **C#:** Use `dynamic`, `JObject`, or `DataTable` to consume variable schemas without recompilation.

Would you like a deeper dive into any specific approach, such as full EF Core property bag implementation with migrations?
