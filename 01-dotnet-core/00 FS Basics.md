# Basic & Intermediate .NET Full‑Stack Developer Notes

*For developers with 1–4 years of experience*

This document covers foundational and intermediate concepts in C#, ASP.NET Core, Entity Framework Core, SQL, and front‑end integration. Each topic includes clear explanations and practical code examples.

---

## Table of Contents

1. [C# Fundamentals](#c-fundamentals)
2. [Object‑Oriented Programming in C#](#object-oriented-programming-in-c)
3. [ASP.NET Core Basics](#aspnet-core-basics)
4. [Building REST APIs with ASP.NET Core](#building-rest-apis-with-aspnet-core)
5. [Entity Framework Core Essentials](#entity-framework-core-essentials)
6. [SQL for .NET Developers](#sql-for-net-developers)
7. [Front‑End Integration (Angular/React Basics)](#front-end-integration-angularreact-basics)
8. [Debugging and Troubleshooting](#debugging-and-troubleshooting)

---

## C# Fundamentals

### 1. What are value types and reference types in C#?

**Answer:**  
- **Value types** store the actual data. They are allocated on the stack (or inline within an object). Examples: `int`, `bool`, `struct`, `enum`.  
- **Reference types** store a reference (memory address) to the data on the heap. Examples: `string`, `class`, `array`, `delegate`.

**Example:**
```csharp
int a = 5;
int b = a;    // b gets a copy of 5
b = 10;       // a remains 5

var person1 = new Person { Name = "Alice" };
var person2 = person1;     // both point to the same object
person2.Name = "Bob";      // person1.Name is now "Bob"
```

---

### 2. Explain the difference between `string` and `StringBuilder`.

**Answer:**  
- `string` is **immutable** – every modification creates a new string object in memory.  
- `StringBuilder` is **mutable** and designed for efficient string manipulation in loops.

**Example:**
```csharp
// Inefficient – creates multiple strings
string result = "";
for (int i = 0; i < 100; i++)
{
    result += i + ",";
}

// Efficient – single buffer
var sb = new StringBuilder();
for (int i = 0; i < 100; i++)
{
    sb.Append(i).Append(',');
}
string result = sb.ToString();
```

---

### 3. What are nullable types and how do you use them?

**Answer:**  
Nullable types allow value types to represent `null`. Use `?` syntax.

**Example:**
```csharp
int? age = null;
if (age.HasValue)
    Console.WriteLine(age.Value);
else
    Console.WriteLine("Age not provided");

// Null-coalescing operator
int actualAge = age ?? 18;

// Null-conditional operator
string name = person?.Name ?? "Unknown";
```

---

### 4. What is the difference between `const` and `readonly`?

| Feature       | `const`                                | `readonly`                                   |
|---------------|----------------------------------------|----------------------------------------------|
| Value set at  | Compile‑time                           | Runtime (constructor)                        |
| Can be static | Implicitly static                      | Instance or static                           |
| Type          | Only primitive, string, enum           | Any type                                     |
| Modifiable    | Never                                  | Only in constructor/field initializer        |

**Example:**
```csharp
public class MathConstants
{
    public const double Pi = 3.14159;          // compile-time constant
    public readonly DateTime CreatedAt;        // set in constructor

    public MathConstants()
    {
        CreatedAt = DateTime.Now;
    }
}
```

---

### 5. What are extension methods and how do you create one?

**Answer:**  
Extension methods allow you to add new methods to existing types without modifying them. They must be defined in a **static class** and the first parameter uses the `this` keyword.

**Example:**
```csharp
public static class StringExtensions
{
    public static bool IsValidEmail(this string input)
    {
        return input.Contains("@") && input.Contains(".");
    }
}

// Usage:
string email = "test@example.com";
bool isValid = email.IsValidEmail();
```

---

### 6. Explain `async` and `await` with a simple example.

**Answer:**  
`async` and `await` make asynchronous programming easier. The `await` keyword pauses the method until the asynchronous operation completes, without blocking the thread.

**Example:**
```csharp
public async Task<string> GetWebPageContentAsync(string url)
{
    using var client = new HttpClient();
    string content = await client.GetStringAsync(url);
    return content;
}

// Calling method:
string result = await GetWebPageContentAsync("https://example.com");
```

**Important:**  
- `async` methods should return `Task`, `Task<T>`, or `ValueTask<T>`.  
- Avoid `async void` except for event handlers.

---

## Object‑Oriented Programming in C#

### 7. What are the four pillars of OOP? Provide brief examples.

**Answer:**

1. **Encapsulation** – Hiding internal state and requiring interaction through methods/properties.
```csharp
public class BankAccount
{
    private decimal _balance;
    public void Deposit(decimal amount) { _balance += amount; }
    public decimal GetBalance() => _balance;
}
```

2. **Inheritance** – Deriving a class from a base class.
```csharp
public class Animal { public void Eat() { } }
public class Dog : Animal { public void Bark() { } }
```

3. **Polymorphism** – Treating derived classes as base types and overriding behaviour.
```csharp
public virtual void MakeSound() => Console.WriteLine("Some sound");
public override void MakeSound() => Console.WriteLine("Woof");
```

4. **Abstraction** – Exposing only essential features and hiding complexity.
```csharp
public interface IEmailService { void Send(string to, string subject); }
```

---

### 8. What is the difference between an abstract class and an interface?

| Feature                | Abstract Class                               | Interface                                       |
|------------------------|----------------------------------------------|-------------------------------------------------|
| Multiple inheritance   | No                                           | Yes (a class can implement many interfaces)     |
| Implementation         | Can contain both abstract and concrete members | Only method/property signatures (prior to C# 8) |
| Access modifiers       | Allowed                                      | All members are public by default               |
| Constructor            | Can have                                     | Cannot have                                     |
| When to use            | Share common code among related classes      | Define a contract for unrelated classes         |

**Example:**
```csharp
public abstract class Shape
{
    public abstract double Area();
    public virtual void Display() => Console.WriteLine("Shape");
}

public interface IPrintable
{
    void Print();
}
```

---

### 9. What is method overloading vs overriding?

- **Overloading**: Same method name, different parameters (compile‑time polymorphism).
- **Overriding**: Redefining a base class method in a derived class (run‑time polymorphism).

**Example:**
```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;          // Overload 1
    public double Add(double a, double b) => a + b; // Overload 2
}

public class Animal
{
    public virtual void Speak() => Console.WriteLine("Animal sound");
}
public class Cat : Animal
{
    public override void Speak() => Console.WriteLine("Meow");
}
```

---

### 10. Explain the `using` statement and `IDisposable`.

**Answer:**  
The `using` statement ensures that an object that implements `IDisposable` is properly disposed, even if an exception occurs. This is critical for resources like file handles, database connections, etc.

**Example:**
```csharp
using (var reader = new StreamReader("file.txt"))
{
    string content = reader.ReadToEnd();
} // reader.Dispose() automatically called

// C# 8+ simplified syntax:
using var reader2 = new StreamReader("file.txt");
string content2 = reader2.ReadToEnd();
```

---

## ASP.NET Core Basics

### 11. What is the request pipeline in ASP.NET Core?

**Answer:**  
The request pipeline is a series of middleware components that process an HTTP request. Each middleware can:
- Perform actions (e.g., logging, authentication)
- Call the next middleware
- Perform actions after the next middleware returns

**Example `Program.cs`:**
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

---

### 12. How do you read configuration settings (e.g., `appsettings.json`) in ASP.NET Core?

**Answer:**  
Use the `IConfiguration` interface, which is automatically registered.

**Example `appsettings.json`:**
```json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=MyDb;Trusted_Connection=true"
  },
  "AppSettings": {
    "PageSize": 10
  }
}
```

**Access in a controller/service:**
```csharp
public class HomeController : Controller
{
    private readonly IConfiguration _config;
    public HomeController(IConfiguration config) => _config = config;

    public IActionResult Index()
    {
        string connStr = _config.GetConnectionString("Default");
        int pageSize = _config.GetValue<int>("AppSettings:PageSize");
        return Ok();
    }
}
```

**Using Options Pattern (recommended for complex settings):**
```csharp
public class AppSettings { public int PageSize { get; set; } }
services.Configure<AppSettings>(Configuration.GetSection("AppSettings"));

// Inject IOptions<AppSettings> to use
```

---

### 13. What is Dependency Injection (DI) and how is it used in ASP.NET Core?

**Answer:**  
DI is a design pattern where objects receive their dependencies from an external source rather than creating them internally. ASP.NET Core has built‑in DI container.

**Register services in `Program.cs`:**
```csharp
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddTransient<IEmailSender, EmailSender>();
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
```

**Lifetimes:**
- **Transient**: Created each time requested.
- **Scoped**: Created once per HTTP request.
- **Singleton**: Created once and shared.

**Inject into controller:**
```csharp
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    public ProductsController(IProductService productService) => _productService = productService;
}
```

---

## Building REST APIs with ASP.NET Core

### 14. Create a simple CRUD API controller.

**Example – ProductsController:**
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private static List<Product> _products = new();

    [HttpGet]
    public IActionResult GetAll() => Ok(_products);

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null) return NotFound();
        return Ok(product);
    }

    [HttpPost]
    public IActionResult Create(Product product)
    {
        product.Id = _products.Count + 1;
        _products.Add(product);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }

    [HttpPut("{id}")]
    public IActionResult Update(int id, Product updatedProduct)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null) return NotFound();
        product.Name = updatedProduct.Name;
        product.Price = updatedProduct.Price;
        return NoContent();
    }

    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null) return NotFound();
        _products.Remove(product);
        return NoContent();
    }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

---

### 15. How do you handle model validation in ASP.NET Core?

**Answer:**  
Use data annotations and let the framework validate automatically. The `[ApiController]` attribute enables automatic validation error responses.

**Example:**
```csharp
public class Product
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(50)]
    public string Name { get; set; }

    [Range(0.01, 9999.99)]
    public decimal Price { get; set; }
}

[HttpPost]
public IActionResult Create(Product product)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);
    // Proceed...
}
```

**Returning custom validation response:**
```csharp
services.Configure<ApiBehaviorOptions>(options =>
{
    options.InvalidModelStateResponseFactory = context =>
    {
        var errors = context.ModelState
            .Where(e => e.Value.Errors.Count > 0)
            .ToDictionary(kvp => kvp.Key, kvp => kvp.Value.Errors.Select(e => e.ErrorMessage).ToArray());
        return new BadRequestObjectResult(new { Errors = errors });
    };
});
```

---

### 16. What are action filters and when would you use them?

**Answer:**  
Action filters run before and after an action method executes. They are useful for cross‑cutting concerns like logging, caching, or validation.

**Example – logging execution time:**
```csharp
public class LogActionFilter : IActionFilter
{
    private Stopwatch _stopwatch;
    private readonly ILogger<LogActionFilter> _logger;

    public LogActionFilter(ILogger<LogActionFilter> logger) => _logger = logger;

    public void OnActionExecuting(ActionExecutingContext context)
    {
        _stopwatch = Stopwatch.StartNew();
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        _stopwatch.Stop();
        _logger.LogInformation($"Action {context.ActionDescriptor.DisplayName} took {_stopwatch.ElapsedMilliseconds}ms");
    }
}

// Register globally:
builder.Services.AddControllers(options => options.Filters.Add<LogActionFilter>());
```

---

## Entity Framework Core Essentials

### 17. How do you set up Entity Framework Core with SQL Server?

**Steps:**

1. Install NuGet packages:  
   `Microsoft.EntityFrameworkCore.SqlServer`  
   `Microsoft.EntityFrameworkCore.Tools` (for migrations)

2. Define a `DbContext`:
```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Product> Products { get; set; }
    public DbSet<Category> Categories { get; set; }
}
```

3. Register in `Program.cs`:
```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

4. Create a migration:
```
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

### 18. Explain the difference between eager loading, lazy loading, and explicit loading.

| Loading Type     | Description                                                                           | When to Use                                          |
|------------------|---------------------------------------------------------------------------------------|------------------------------------------------------|
| **Eager Loading**| Load related data as part of the initial query using `.Include()` / `.ThenInclude()`.  | When you know you'll need related data immediately.  |
| **Lazy Loading** | Related data is loaded automatically when the navigation property is accessed.         | When related data is optional (requires virtual nav props and proxy package). |
| **Explicit Loading** | Manually load related data after the main entity is loaded using `.Collection()` / `.Reference()`. | Fine‑grained control over which relations to load.   |

**Eager Loading Example:**
```csharp
var orders = await dbContext.Orders
    .Include(o => o.Customer)
    .Include(o => o.Items)
        .ThenInclude(i => i.Product)
    .ToListAsync();
```

**Lazy Loading Setup:**
```csharp
public class Order
{
    public int Id { get; set; }
    public virtual Customer Customer { get; set; }  // virtual keyword required
}
// Install Microsoft.EntityFrameworkCore.Proxies
options.UseLazyLoadingProxies();
```

---

### 19. How do you perform basic CRUD operations with EF Core?

```csharp
// CREATE
var product = new Product { Name = "Laptop", Price = 999.99m };
dbContext.Products.Add(product);
await dbContext.SaveChangesAsync();

// READ
var product = await dbContext.Products.FindAsync(id);
var allProducts = await dbContext.Products.ToListAsync();
var filtered = await dbContext.Products.Where(p => p.Price > 500).ToListAsync();

// UPDATE
var product = await dbContext.Products.FindAsync(id);
if (product != null)
{
    product.Price = 899.99m;
    await dbContext.SaveChangesAsync();
}

// DELETE
var product = await dbContext.Products.FindAsync(id);
if (product != null)
{
    dbContext.Products.Remove(product);
    await dbContext.SaveChangesAsync();
}
```

---

### 20. What are database migrations and how do you manage them?

**Answer:**  
Migrations keep your database schema in sync with your C# model classes. Each migration is a set of instructions to create/alter tables.

**Common commands:**
```
dotnet ef migrations add AddNewColumnToProduct   # create migration
dotnet ef database update                         # apply to database
dotnet ef migrations remove                       # remove last migration (if not applied)
dotnet ef database update LastGoodMigration       # rollback to specific migration
dotnet ef migrations script                       # generate SQL script
```

**Applying migrations on startup (development only):**
```csharp
using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    db.Database.Migrate();
}
```

---

## SQL for .NET Developers

### 21. Write a SQL query to find the second highest salary from an `Employees` table.

```sql
SELECT MAX(Salary) AS SecondHighestSalary
FROM Employees
WHERE Salary < (SELECT MAX(Salary) FROM Employees);
```

**Using `OFFSET/FETCH` (SQL Server 2012+):**
```sql
SELECT DISTINCT Salary
FROM Employees
ORDER BY Salary DESC
OFFSET 1 ROW FETCH NEXT 1 ROW ONLY;
```

---

### 22. What is the difference between `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, and `FULL OUTER JOIN`?

**Answer:**  

| Join Type          | Returns                                                                          |
|--------------------|----------------------------------------------------------------------------------|
| `INNER JOIN`       | Only rows where there is a match in **both** tables.                              |
| `LEFT JOIN`        | All rows from the **left** table, and matched rows from the right (NULL if none).|
| `RIGHT JOIN`       | All rows from the **right** table, and matched rows from the left.                |
| `FULL OUTER JOIN`  | All rows from both tables, with NULLs where no match exists.                      |

**Example:**
```sql
-- Get all customers and their orders (including customers with no orders)
SELECT c.Name, o.OrderDate
FROM Customers c
LEFT JOIN Orders o ON c.Id = o.CustomerId;
```

---

### 23. How do you use `GROUP BY` with aggregate functions?

**Answer:**  
`GROUP BY` groups rows that have the same values in specified columns. Aggregate functions like `COUNT()`, `SUM()`, `AVG()`, `MAX()`, `MIN()` are applied to each group.

**Example:**
```sql
SELECT CategoryId, COUNT(*) AS ProductCount, AVG(Price) AS AvgPrice
FROM Products
GROUP BY CategoryId
HAVING COUNT(*) > 5;   -- filter groups
```

---

### 24. What are indexes and why are they important?

**Answer:**  
Indexes are database structures that improve the speed of data retrieval operations. They work like a book's index – allowing the database to find rows without scanning the entire table.

**Creating an index:**
```sql
CREATE INDEX IX_Products_Name ON Products(Name);
CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId) INCLUDE (OrderDate, Total);
```

**Trade‑offs:**  
- **Pros**: Faster `SELECT` queries, especially with `WHERE` and `JOIN`.  
- **Cons**: Slower `INSERT`, `UPDATE`, `DELETE` (index must be updated), consumes disk space.

---

## Front‑End Integration (Angular/React Basics)

### 25. How do you call an ASP.NET Core API from an Angular service?

**Answer:**  
Angular's `HttpClient` is used to make HTTP requests.

**Example Angular service (`product.service.ts`):**
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface Product {
  id: number;
  name: string;
  price: number;
}

@Injectable({ providedIn: 'root' })
export class ProductService {
  private apiUrl = 'https://localhost:5001/api/products';

  constructor(private http: HttpClient) { }

  getAll(): Observable<Product[]> {
    return this.http.get<Product[]>(this.apiUrl);
  }

  getById(id: number): Observable<Product> {
    return this.http.get<Product>(`${this.apiUrl}/${id}`);
  }

  create(product: Product): Observable<Product> {
    return this.http.post<Product>(this.apiUrl, product);
  }

  update(id: number, product: Product): Observable<void> {
    return this.http.put<void>(`${this.apiUrl}/${id}`, product);
  }

  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

**Enable CORS in ASP.NET Core:**
```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAngularApp",
        policy => policy.WithOrigins("http://localhost:4200")
                        .AllowAnyMethod()
                        .AllowAnyHeader());
});
app.UseCors("AllowAngularApp");
```

---

### 26. How do you fetch data from an API in a React component using `useEffect`?

**Answer:**

```jsx
import React, { useState, useEffect } from 'react';

function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('https://localhost:5001/api/products')
      .then(response => response.json())
      .then(data => {
        setProducts(data);
        setLoading(false);
      })
      .catch(error => console.error('Error:', error));
  }, []); // empty dependency array = run once on mount

  if (loading) return <div>Loading...</div>;

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>{product.name} - ${product.price}</li>
      ))}
    </ul>
  );
}

export default ProductList;
```

---

## Debugging and Troubleshooting

### 27. How do you debug an ASP.NET Core application?

**Answer:**  
- Set breakpoints in Visual Studio / VS Code.  
- Use `Debug.WriteLine()` or `Console.WriteLine()` for quick output.  
- Attach the debugger to a running process.  
- Use the **Diagnostics Tools** window to monitor CPU/memory.  
- Log with `ILogger<T>` and view output in console or log files.

**Example logging:**
```csharp
public class ProductsController : ControllerBase
{
    private readonly ILogger<ProductsController> _logger;
    public ProductsController(ILogger<ProductsController> logger) => _logger = logger;

    [HttpGet]
    public IActionResult GetAll()
    {
        _logger.LogInformation("Getting all products");
        // ...
    }
}
```

---

### 28. What are common HTTP status codes and what do they mean?

| Code  | Name              | Meaning                                                      |
|-------|-------------------|--------------------------------------------------------------|
| 200   | OK                | Request succeeded.                                           |
| 201   | Created           | Resource created successfully (often with `Location` header).|
| 204   | No Content        | Success, but no response body (common for PUT/DELETE).       |
| 400   | Bad Request       | Client error (invalid data, malformed request).              |
| 401   | Unauthorized      | Authentication required.                                     |
| 403   | Forbidden         | Authenticated but not allowed.                               |
| 404   | Not Found         | Resource doesn't exist.                                      |
| 500   | Internal Server Error | Server encountered an unexpected condition.               |

---

### 29. How do you handle errors globally in an ASP.NET Core API?

**Answer:**  
Use exception handling middleware.

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        context.Response.StatusCode = StatusCodes.Status500InternalServerError;
        context.Response.ContentType = "application/json";

        var exception = context.Features.Get<IExceptionHandlerFeature>()?.Error;
        var response = new { error = "An unexpected error occurred.", detail = exception?.Message };
        await context.Response.WriteAsJsonAsync(response);
    });
});
```

**For development environment**, `app.UseDeveloperExceptionPage()` shows detailed error pages.

---

### 30. What tools can you use to profile and optimise a .NET application?

**Answer:**  
- **Visual Studio Diagnostic Tools** – CPU usage, memory, GC events.  
- **dotnet-trace** / **dotnet-counters** – command‑line performance monitoring.  
- **PerfView** – advanced ETW trace analysis.  
- **Application Insights** / **Azure Monitor** – cloud‑hosted application monitoring.  
- **BenchmarkDotNet** – micro‑benchmarking library.

**Example using `dotnet-counters`:**
```
dotnet-counters monitor --process-id <pid> System.Runtime Microsoft.AspNetCore.Hosting
```

---

*These notes provide a solid foundation for basic to intermediate .NET full‑stack development. Continue to build projects and explore official Microsoft documentation to deepen your understanding.*
