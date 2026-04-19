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
9. [LINQ Fundamentals](#linq-fundamentals)
10. [Authentication and Authorisation in ASP.NET Core](#authentication-and-authorisation-in-aspnet-core)
11. [Working with Files and Streams](#working-with-files-and-streams)
12. [Caching Strategies](#caching-strategies)
13. [Middleware in Depth](#middleware-in-depth)
14. [Configuration Management](#configuration-management)
15. [Unit Testing Basics](#unit-testing-basics)
16. [Version Control with Git (Essentials)](#version-control-with-git-essentials)
17. [Error Handling and Logging Best Practices](#error-handling-and-logging-best-practices)
18. [Background Tasks with `IHostedService`](#background-tasks-with-ihostedservice)
19. [Working with JSON in .NET](#working-with-json-in-net)

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
## Options Pattern in ASP.NET Core

The **Options Pattern** provides a strongly‑typed way to access configuration settings. It decouples configuration from consuming classes, enables validation, and supports reloading changes.

---

### 1. Why Use the Options Pattern?

| Without Options Pattern                     | With Options Pattern                                   |
|---------------------------------------------|--------------------------------------------------------|
| Magic strings (`_config["Key"]`)            | Strongly‑typed properties (`_options.Key`)             |
| No compile‑time checking                    | Full IntelliSense and refactoring support              |
| Scattered configuration access              | Centralised settings class                             |
| Manual validation code                      | Built‑in validation with `DataAnnotations`             |
| Hard to unit test                           | Easy to mock `IOptions<T>`                             |

---

### 2. Basic Setup

**Step 1: Define a Settings Class**
```csharp
public class EmailSettings
{
    public string SmtpServer { get; set; } = string.Empty;
    public int Port { get; set; }
    public bool EnableSsl { get; set; }
    public string FromAddress { get; set; } = string.Empty;
}
```

**Step 2: Add Configuration in `appsettings.json`**
```json
{
  "Email": {
    "SmtpServer": "smtp.example.com",
    "Port": 587,
    "EnableSsl": true,
    "FromAddress": "noreply@example.com"
  }
}
```

**Step 3: Register in `Program.cs`**
```csharp
builder.Services.Configure<EmailSettings>(builder.Configuration.GetSection("Email"));
```

**Step 4: Inject and Use**
```csharp
public class EmailService
{
    private readonly EmailSettings _settings;

    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }

    public void SendEmail(string to, string subject, string body)
    {
        using var client = new SmtpClient(_settings.SmtpServer, _settings.Port);
        client.EnableSsl = _settings.EnableSsl;
        // ...
    }
}
```

---

### 3. Three Options Interfaces

| Interface              | Lifetime | Reloads on Change | Use Case                                                      |
|-----------------------|----------|-------------------|---------------------------------------------------------------|
| **`IOptions<T>`**     | Singleton | No                | Settings that **never change** after app startup.             |
| **`IOptionsSnapshot<T>`** | Scoped   | Yes (per request) | Settings that may change **between requests**.                |
| **`IOptionsMonitor<T>`** | Singleton | Yes (real‑time)   | Settings that must react **immediately** to changes.          |

---

#### `IOptions<T>` – Static, Read‑Once

```csharp
public class SingletonService
{
    private readonly EmailSettings _settings;
    public SingletonService(IOptions<EmailSettings> options) => _settings = options.Value;
}
```
- `Value` is **cached** and **never updates** after the first access.
- Suitable for settings from files that require app restart to change.

---

#### `IOptionsSnapshot<T>` – Scoped, Per‑Request Refresh

```csharp
public class ScopedService
{
    private readonly EmailSettings _settings;
    public ScopedService(IOptionsSnapshot<EmailSettings> options) => _settings = options.Value;
}
```
- `Value` is computed **once per scope** (in ASP.NET Core, per HTTP request).
- If configuration source changes (e.g., Azure App Configuration, reloaded JSON file), the next request gets the **new values**.
- **Best choice for most web applications** where config changes should apply gracefully.

---

#### `IOptionsMonitor<T>` – Real‑Time Notifications

```csharp
public class MonitorService : IDisposable
{
    private readonly IOptionsMonitor<EmailSettings> _monitor;
    private readonly IDisposable _changeListener;
    private EmailSettings _currentSettings;

    public MonitorService(IOptionsMonitor<EmailSettings> monitor)
    {
        _monitor = monitor;
        _currentSettings = monitor.CurrentValue;
        _changeListener = monitor.OnChange(settings =>
        {
            _currentSettings = settings;
            Console.WriteLine("Email settings updated!");
        });
    }

    public void SendEmail()
    {
        // Always use _currentSettings or monitor.CurrentValue for latest
        var settings = _monitor.CurrentValue;
        // ...
    }

    public void Dispose() => _changeListener?.Dispose();
}
```
- `CurrentValue` always returns the **latest** settings.
- `OnChange` registers a callback that fires whenever configuration changes.
- Useful for long‑lived singleton services that need to adapt immediately (e.g., background workers).

---

### 4. Named Options

When you need **multiple instances** of the same settings class (e.g., different email providers).

**Registration:**
```csharp
services.Configure<EmailSettings>("Smtp", builder.Configuration.GetSection("Email:Smtp"));
services.Configure<EmailSettings>("SendGrid", builder.Configuration.GetSection("Email:SendGrid"));
```

**appsettings.json:**
```json
{
  "Email": {
    "Smtp": { "SmtpServer": "smtp.office365.com", "Port": 587 },
    "SendGrid": { "SmtpServer": "smtp.sendgrid.net", "Port": 465 }
  }
}
```

**Consumption:**
```csharp
public class MultiEmailService
{
    private readonly EmailSettings _smtpSettings;
    private readonly EmailSettings _sendGridSettings;

    public MultiEmailService(IOptionsSnapshot<EmailSettings> optionsAccessor)
    {
        _smtpSettings = optionsAccessor.Get("Smtp");
        _sendGridSettings = optionsAccessor.Get("SendGrid");
    }
}
```

**Alternative using `IOptionsMonitor<T>`:**
```csharp
var smtp = _monitor.Get("Smtp");
```

---

### 5. Options Validation

#### Using Data Annotations

```csharp
public class EmailSettings
{
    [Required(ErrorMessage = "SMTP server is required")]
    public string SmtpServer { get; set; } = string.Empty;

    [Range(1, 65535, ErrorMessage = "Port must be between 1 and 65535")]
    public int Port { get; set; }

    [EmailAddress]
    public string FromAddress { get; set; } = string.Empty;
}
```

**Enable validation at startup:**
```csharp
builder.Services.AddOptions<EmailSettings>()
    .Bind(builder.Configuration.GetSection("Email"))
    .ValidateDataAnnotations()
    .ValidateOnStart(); // Throws exception at app start if invalid
```

#### Custom Validation Logic

```csharp
builder.Services.AddOptions<EmailSettings>()
    .Bind(builder.Configuration.GetSection("Email"))
    .Validate(settings =>
    {
        if (settings.EnableSsl && settings.Port == 25)
            return false; // SSL on port 25 is unusual
        return true;
    }, "SSL should not be used with port 25.");
```

#### Complex Validation Across Multiple Settings

```csharp
builder.Services.AddOptions<EmailSettings>()
    .Bind(builder.Configuration.GetSection("Email"))
    .Validate<ILogger<EmailSettings>>((settings, logger) =>
    {
        if (string.IsNullOrEmpty(settings.FromAddress))
        {
            logger.LogWarning("FromAddress is empty, using default.");
            settings.FromAddress = "default@example.com";
        }
        return true;
    });
```

---

### 6. Post‑Configuration

Modify or compute settings after binding.

```csharp
builder.Services.PostConfigure<EmailSettings>(settings =>
{
    // Ensure FromAddress is never null
    settings.FromAddress ??= "noreply@default.com";
    
    // Compute derived value
    settings.ConnectionString = $"{settings.SmtpServer}:{settings.Port}";
});
```

**For named options:**
```csharp
builder.Services.PostConfigure<EmailSettings>("Smtp", settings =>
{
    settings.FromAddress = "smtp@example.com";
});
```

---

### 7. Real‑World Example: Environment‑Aware Settings

```csharp
public class ApiSettings
{
    public string BaseUrl { get; set; } = string.Empty;
    public int TimeoutSeconds { get; set; }
    public RetryPolicy Retry { get; set; } = new();
}

public class RetryPolicy
{
    public int MaxAttempts { get; set; } = 3;
    public int BackoffMilliseconds { get; set; } = 1000;
}
```

**appsettings.json:**
```json
{
  "ApiSettings": {
    "BaseUrl": "https://api.example.com",
    "TimeoutSeconds": 30,
    "Retry": {
      "MaxAttempts": 3,
      "BackoffMilliseconds": 1000
    }
  }
}
```

**appsettings.Development.json:**
```json
{
  "ApiSettings": {
    "BaseUrl": "https://localhost:5001",
    "TimeoutSeconds": 60
  }
}
```

**Registration with validation:**
```csharp
builder.Services.AddOptions<ApiSettings>()
    .Bind(builder.Configuration.GetSection("ApiSettings"))
    .ValidateDataAnnotations()
    .ValidateOnStart();

// Usage in typed HttpClient
builder.Services.AddHttpClient<IApiClient, ApiClient>((sp, client) =>
{
    var settings = sp.GetRequiredService<IOptionsSnapshot<ApiSettings>>().Value;
    client.BaseAddress = new Uri(settings.BaseUrl);
    client.Timeout = TimeSpan.FromSeconds(settings.TimeoutSeconds);
});
```

---

### 8. Unit Testing with Options

```csharp
[Fact]
public void EmailService_UsesConfiguredSettings()
{
    // Arrange
    var settings = new EmailSettings { SmtpServer = "test.smtp.com", Port = 25 };
    var options = Options.Create(settings); // Simple wrapper
    var service = new EmailService(options);

    // Act & Assert
    // ...
}

// Using IOptionsSnapshot
var mockSnapshot = new Mock<IOptionsSnapshot<EmailSettings>>();
mockSnapshot.Setup(s => s.Value).Returns(settings);
```

---

### 9. When to Use Each Interface – Decision Tree

```
┌─────────────────────────────────────────┐
│ Does the setting change without app     │
│ restart? (e.g., Azure App Config)       │
└─────────────────────────────────────────┘
        │                    │
       YES                  NO
        │                    │
        ▼                    ▼
┌───────────────────┐   ┌───────────────┐
│ Does the consumer │   │ Use           │
│ need immediate    │   │ IOptions<T>   │
│ updates?          │   └───────────────┘
└───────────────────┘
        │                    │
       YES                  NO
        │                    │
        ▼                    ▼
┌───────────────────┐   ┌───────────────────┐
│ Use               │   │ Use               │
│ IOptionsMonitor<T>│   │ IOptionsSnapshot<T>│
└───────────────────┘   └───────────────────┘
```

---

### 10. Common Pitfalls

| Pitfall                                      | Solution                                                     |
|----------------------------------------------|--------------------------------------------------------------|
| Injecting `IOptionsSnapshot<T>` into a Singleton | Use `IOptionsMonitor<T>` instead.                        |
| Assuming `IOptions<T>.Value` updates          | It **never** updates; use Snapshot or Monitor.               |
| Forgetting to call `ValidateOnStart()`        | Validation only runs when `Value` is accessed, or at startup if configured. |
| Complex nested objects not binding            | Ensure property names match JSON keys **case‑insensitively**. |

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
Here's the controller code with added comments showing the actual endpoint URLs and HTTP methods:

```csharp
[ApiController]
[Route("api/[controller]")] // Base route: /api/products
public class ProductsController : ControllerBase
{
    private static List<Product> _products = new();

    // GET /api/products
    [HttpGet]
    public IActionResult GetAll() => Ok(_products);

    // GET /api/products/{id}
    // Example: GET /api/products/5
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null) return NotFound();
        return Ok(product);
    }

    // POST /api/products
    // Request Body: { "name": "Laptop", "price": 999.99 }
    [HttpPost]
    public IActionResult Create(Product product)
    {
        product.Id = _products.Count + 1;
        _products.Add(product);
        // Returns 201 Created with Location header pointing to the new resource
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }

    // PUT /api/products/{id}
    // Example: PUT /api/products/3
    // Request Body: { "name": "Updated Laptop", "price": 899.99 }
    [HttpPut("{id}")]
    public IActionResult Update(int id, Product updatedProduct)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null) return NotFound();
        product.Name = updatedProduct.Name;
        product.Price = updatedProduct.Price;
        return NoContent(); // 204 No Content
    }

    // DELETE /api/products/{id}
    // Example: DELETE /api/products/2
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null) return NotFound();
        _products.Remove(product);
        return NoContent(); // 204 No Content
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

Here's the model validation explanation with detailed inline comments for clarity:

```csharp
// ============================================
// MODEL VALIDATION IN ASP.NET CORE
// ============================================

// Step 1: Define the model with validation attributes
public class Product
{
    public int Id { get; set; }

    // Required attribute: field cannot be null or empty
    [Required(ErrorMessage = "Name is required")]
    // StringLength: maximum character limit
    [StringLength(50)]
    public string Name { get; set; }

    // Range attribute: value must be between 0.01 and 9999.99
    [Range(0.01, 9999.99)]
    public decimal Price { get; set; }
}

// Step 2: Controller action with manual ModelState check
[ApiController]  // This attribute enables automatic 400 responses for invalid models
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // POST /api/products
    // Request body: { "name": "Laptop", "price": 999.99 }
    [HttpPost]
    public IActionResult Create(Product product)
    {
        // Check if validation passed (true if all attributes are satisfied)
        if (!ModelState.IsValid)
        {
            // Returns 400 Bad Request with detailed error information
            return BadRequest(ModelState);
        }

        // Proceed with business logic if validation succeeds
        // ... save to database, etc.
        return Ok(product);
    }
}
```

---

### Customising the Validation Response Format

The default error response includes technical details. To return a cleaner, client‑friendly format, override `ApiBehaviorOptions` in `Program.cs`:

```csharp
// Program.cs - Configure custom validation error response
builder.Services.Configure<ApiBehaviorOptions>(options =>
{
    // This factory runs when ModelState is invalid (automatic validation triggered by [ApiController])
    options.InvalidModelStateResponseFactory = context =>
    {
        // Transform ModelState dictionary into a simplified errors object
        var errors = context.ModelState
            // Keep only entries that have validation errors
            .Where(entry => entry.Value.Errors.Count > 0)
            .ToDictionary(
                keySelector: kvp => kvp.Key,                                    // Field name (e.g., "Name")
                elementSelector: kvp => kvp.Value.Errors                         // Array of error objects
                                          .Select(e => e.ErrorMessage)           // Extract just the message string
                                          .ToArray()                             // Convert to array
            );

        // Return a 400 Bad Request with custom JSON structure
        return new BadRequestObjectResult(new { Errors = errors });
    };
});
```

---

### Example Request and Response

**Invalid Request (POST `/api/products`):**
```json
{
    "name": "",           // Empty string fails [Required]
    "price": 0            // 0 fails [Range(0.01, 9999.99)]
}
```

**Custom Response (with the above configuration):**
```json
{
    "errors": {
        "Name": ["Name is required"],
        "Price": ["The field Price must be between 0.01 and 9999.99."]
    }
}
```

**Default Response (without customisation):**
```json
{
    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "traceId": "00-abc123...",
    "errors": {
        "Name": ["Name is required"],
        "Price": ["The field Price must be between 0.01 and 9999.99."]
    }
}
```

---

### Key Points to Remember

| Concept                              | Explanation                                                                           |
|--------------------------------------|---------------------------------------------------------------------------------------|
| `[ApiController]`                    | Automatically triggers validation and returns `400 Bad Request` for invalid models.   |
| `ModelState.IsValid`                 | Boolean property that reflects whether **all** validation attributes passed.           |
| `BadRequest(ModelState)`             | Returns a `400` response containing the raw validation errors.                         |
| `InvalidModelStateResponseFactory`   | Custom delegate that formats the error response exactly as you need.                   |
| Data Annotations                    | Attributes like `[Required]`, `[StringLength]`, `[Range]` that define validation rules.|

> 💡 **Best Practice:** Always keep validation logic **in the model** using attributes. Use `InvalidModelStateResponseFactory` to provide a consistent error shape across your entire API.
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

# Basic & Intermediate .NET Full‑Stack Developer Notes (Extended)

*Additional topics for developers with 1–4 years of experience*

This document extends the previous notes with more practical concepts you'll encounter in everyday .NET development.

---



---

## LINQ Fundamentals

### 31. What is LINQ and what are the two main syntaxes?

**Answer:**  
LINQ (Language Integrated Query) allows you to write queries against data sources (collections, databases, XML) directly in C#.

**Two syntaxes:**
- **Query syntax** (SQL‑like)
- **Method syntax** (fluent extension methods)

**Example:**
```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6 };

// Query syntax
var evenNumbersQuery = from n in numbers
                       where n % 2 == 0
                       select n;

// Method syntax
var evenNumbersMethod = numbers.Where(n => n % 2 == 0);
```

---

### 32. Explain commonly used LINQ methods: `Where`, `Select`, `OrderBy`, `GroupBy`, `FirstOrDefault`, `Any`.

**Answer:**

| Method           | Purpose                                                         | Example                                                       |
|------------------|-----------------------------------------------------------------|---------------------------------------------------------------|
| `Where`          | Filters a sequence based on a predicate.                        | `numbers.Where(n => n > 10)`                                  |
| `Select`         | Projects each element into a new form.                          | `products.Select(p => p.Name)`                                |
| `OrderBy`        | Sorts elements in ascending order.                              | `products.OrderBy(p => p.Price)`                              |
| `GroupBy`        | Groups elements by a key.                                       | `products.GroupBy(p => p.CategoryId)`                         |
| `FirstOrDefault` | Returns the first element or default if none.                   | `products.FirstOrDefault(p => p.Id == 1)`                     |
| `Any`            | Checks if any element satisfies a condition (returns `bool`).   | `products.Any(p => p.Price > 1000)`                           |

**Examples:**
```csharp
var productNames = products.Select(p => p.Name).ToList();
var cheapProducts = products.Where(p => p.Price < 50).ToList();
var sortedByName = products.OrderBy(p => p.Name).ToList();
var categoryGroups = products.GroupBy(p => p.CategoryId);
bool hasExpensive = products.Any(p => p.Price > 1000);
Product first = products.FirstOrDefault(p => p.Id == 5);
```

---

### 33. What is deferred execution in LINQ?

**Answer:**  
LINQ queries are not executed when they are defined, but only when the results are actually enumerated (e.g., with `foreach`, `ToList()`, `Count()`). This allows queries to be composed efficiently.

**Example:**
```csharp
var query = products.Where(p => p.Price > 100);  // Query defined, not executed yet
var list = query.ToList();                       // Execution happens here
```

**Important:** If the underlying data changes before enumeration, the query will reflect those changes.

---

## Authentication and Authorisation in ASP.NET Core

### 34. How do you add JWT Bearer authentication to an ASP.NET Core API?

**Answer:**  
JWT (JSON Web Token) is commonly used to secure APIs. Clients include the token in the `Authorization` header.

**Step 1: Install package:**
```
Microsoft.AspNetCore.Authentication.JwtBearer
```

**Step 2: Configure in `Program.cs`:**
```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });

builder.Services.AddAuthorization();
```

**Step 3: Protect endpoints with `[Authorize]`:**
```csharp
[Authorize]
[HttpGet]
public IActionResult GetSecureData() => Ok("This is protected.");
```

**Step 4: Generate a token (login endpoint):**
```csharp
[HttpPost("login")]
public IActionResult Login([FromBody] LoginModel model)
{
    // Validate credentials...
    var tokenHandler = new JwtSecurityTokenHandler();
    var key = Encoding.UTF8.GetBytes(_config["Jwt:Key"]);
    var tokenDescriptor = new SecurityTokenDescriptor
    {
        Subject = new ClaimsIdentity(new[] { new Claim("id", user.Id.ToString()) }),
        Expires = DateTime.UtcNow.AddHours(1),
        Issuer = _config["Jwt:Issuer"],
        Audience = _config["Jwt:Audience"],
        SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key),
                                                    SecurityAlgorithms.HmacSha256Signature)
    };
    var token = tokenHandler.CreateToken(tokenDescriptor);
    return Ok(new { token = tokenHandler.WriteToken(token) });
}
```

---

### 35. What is the difference between Authentication and Authorisation?

| Concept         | Description                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| **Authentication** | Verifies **who** the user is (e.g., username/password, token).            |
| **Authorisation**   | Determines **what** the user is allowed to do (e.g., roles, policies).    |

**Example:**
```csharp
[Authorize]                        // Authentication required
[Authorize(Roles = "Admin")]       // Authentication + role authorisation
[Authorize(Policy = "AtLeast21")]  // Custom policy authorisation
```

---

## Working with Files and Streams

### 36. How do you read and write text files in C#?

**Answer:**  
Use `File` static methods for simple operations, or `StreamReader`/`StreamWriter` for more control.

**Read all text:**
```csharp
string content = await File.ReadAllTextAsync("data.txt");
string[] lines = await File.ReadAllLinesAsync("data.txt");
```

**Write all text:**
```csharp
await File.WriteAllTextAsync("output.txt", "Hello World");
await File.WriteAllLinesAsync("output.txt", new[] { "Line1", "Line2" });
```

**Using streams (better for large files):**
```csharp
using var reader = new StreamReader("largefile.txt");
string line;
while ((line = await reader.ReadLineAsync()) != null)
{
    Console.WriteLine(line);
}

using var writer = new StreamWriter("output.txt");
await writer.WriteLineAsync("Some content");
```

---

### 37. How do you upload a file in ASP.NET Core?

**Answer:**  
Use `IFormFile` in the action method.

**Controller:**
```csharp
[HttpPost("upload")]
public async Task<IActionResult> UploadFile(IFormFile file)
{
    if (file == null || file.Length == 0)
        return BadRequest("No file uploaded.");

    var filePath = Path.Combine("uploads", file.FileName);
    using var stream = new FileStream(filePath, FileMode.Create);
    await file.CopyToAsync(stream);

    return Ok(new { filePath });
}
```

**Multiple files:**
```csharp
[HttpPost("upload-multiple")]
public async Task<IActionResult> UploadMultiple(List<IFormFile> files)
{
    foreach (var file in files)
    {
        // Save each file...
    }
    return Ok();
}
```

---

## Caching Strategies

### 38. What is `IMemoryCache` and how do you use it?

**Answer:**  
`IMemoryCache` stores data in the memory of the web server. It's fast but not shared across multiple servers.

**Registration:**
```csharp
builder.Services.AddMemoryCache();
```

**Usage:**
```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;
    public ProductService(IMemoryCache cache) => _cache = cache;

    public async Task<Product> GetProductAsync(int id)
    {
        string cacheKey = $"product-{id}";
        if (!_cache.TryGetValue(cacheKey, out Product product))
        {
            product = await FetchFromDatabaseAsync(id);
            var cacheOptions = new MemoryCacheEntryOptions()
                .SetSlidingExpiration(TimeSpan.FromMinutes(5));
            _cache.Set(cacheKey, product, cacheOptions);
        }
        return product;
    }
}
```

**Shorter with `GetOrCreateAsync`:**
```csharp
public async Task<Product> GetProductAsync(int id)
{
    return await _cache.GetOrCreateAsync($"product-{id}", async entry =>
    {
        entry.SlidingExpiration = TimeSpan.FromMinutes(5);
        return await FetchFromDatabaseAsync(id);
    });
}
```

---

## Middleware in Depth

### 39. How do you create custom middleware for request/response logging?

**Answer:**  
Middleware is a class with an `InvokeAsync` method.

**Example:**
```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation($"Request: {context.Request.Method} {context.Request.Path}");
        await _next(context);
        _logger.LogInformation($"Response: {context.Response.StatusCode}");
    }
}

// Register in Program.cs
app.UseMiddleware<RequestLoggingMiddleware>();
```

**Simpler inline middleware:**
```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine($"Request: {context.Request.Path}");
    await next();
    Console.WriteLine($"Response: {context.Response.StatusCode}");
});
```

---

## Configuration Management

### 40. How do you use different configuration files for different environments?

**Answer:**  
ASP.NET Core loads `appsettings.json` plus `appsettings.{Environment}.json` based on the `ASPNETCORE_ENVIRONMENT` variable.

**Files:**
- `appsettings.json` – base settings
- `appsettings.Development.json` – overrides for development
- `appsettings.Production.json` – overrides for production

**Setting environment:**
```
set ASPNETCORE_ENVIRONMENT=Development
```

**Access in code:** Use `IConfiguration` – it automatically merges files with the environment‑specific file overriding base values.

---

### 41. What are User Secrets and when should you use them?

**Answer:**  
User Secrets store sensitive data (connection strings, API keys) during development **outside** the project folder, preventing accidental commits to source control.

**Enable User Secrets:**
```
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:Default" "your-connection-string"
```

**Access:** Same as any configuration.
```csharp
var connStr = _config.GetConnectionString("Default");
```

**Important:** User Secrets are for **development only**. Use Azure Key Vault or environment variables in production.

---

## Unit Testing Basics

### 42. How do you write a unit test using xUnit?

**Answer:**  
xUnit is a popular testing framework for .NET.

**Example test class:**
```csharp
using Xunit;

public class CalculatorTests
{
    [Fact]
    public void Add_TwoPositiveNumbers_ReturnsSum()
    {
        // Arrange
        var calculator = new Calculator();
        // Act
        int result = calculator.Add(2, 3);
        // Assert
        Assert.Equal(5, result);
    }

    [Theory]
    [InlineData(2, 3, 5)]
    [InlineData(-1, 1, 0)]
    [InlineData(0, 0, 0)]
    public void Add_VariousInputs_ReturnsExpected(int a, int b, int expected)
    {
        var calculator = new Calculator();
        int result = calculator.Add(a, b);
        Assert.Equal(expected, result);
    }
}
```

---

### 43. What is mocking and how do you use Moq?

**Answer:**  
Mocking creates fake objects that simulate dependencies, allowing you to test a class in isolation.

**Example using Moq:**
```csharp
using Moq;
using Xunit;

public class ProductServiceTests
{
    [Fact]
    public async Task GetProduct_ReturnsProductFromRepository()
    {
        // Arrange
        var mockRepo = new Mock<IProductRepository>();
        mockRepo.Setup(repo => repo.GetByIdAsync(1))
                .ReturnsAsync(new Product { Id = 1, Name = "Test" });

        var service = new ProductService(mockRepo.Object);

        // Act
        var product = await service.GetProductAsync(1);

        // Assert
        Assert.Equal("Test", product.Name);
        mockRepo.Verify(repo => repo.GetByIdAsync(1), Times.Once);
    }
}
```

---

## Version Control with Git (Essentials)

### 44. What are the basic Git commands every developer should know?

| Command                              | Purpose                                                   |
|--------------------------------------|-----------------------------------------------------------|
| `git clone <url>`                    | Copy a remote repository to your local machine.           |
| `git status`                         | Show changed files.                                       |
| `git add <file>` / `git add .`       | Stage changes for commit.                                 |
| `git commit -m "message"`            | Commit staged changes with a message.                     |
| `git push`                           | Upload local commits to remote repository.                |
| `git pull`                           | Fetch and merge changes from remote.                      |
| `git branch`                         | List branches.                                            |
| `git checkout -b <branch-name>`      | Create and switch to a new branch.                        |
| `git merge <branch>`                 | Merge another branch into current branch.                 |
| `git log --oneline`                  | Show commit history.                                      |

---

### 45. What is the purpose of a `.gitignore` file?

**Answer:**  
`.gitignore` tells Git which files or folders to **ignore** (not track). Common entries include build outputs, user secrets, and IDE‑specific files.

**Example `.gitignore` for .NET:**
```
bin/
obj/
*.user
appsettings.Development.json
.vs/
```

---

## Error Handling and Logging Best Practices

### 46. How do you implement global exception handling in ASP.NET Core?

**Answer:**  
Use `UseExceptionHandler` middleware.

```csharp
app.UseExceptionHandler("/error");
```

**Custom error endpoint:**
```csharp
[ApiController]
[Route("error")]
public class ErrorController : ControllerBase
{
    [HttpGet]
    public IActionResult HandleError()
    {
        var exception = HttpContext.Features.Get<IExceptionHandlerFeature>()?.Error;
        // Log exception...
        return Problem(title: "An error occurred", statusCode: 500);
    }
}
```

**For APIs, return ProblemDetails:**
```csharp
app.UseExceptionHandler(exceptionHandlerApp =>
{
    exceptionHandlerApp.Run(async context =>
    {
        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/problem+json";
        var error = context.Features.Get<IExceptionHandlerFeature>()?.Error;
        await context.Response.WriteAsJsonAsync(new { error = error?.Message });
    });
});
```

---

### 47. How do you use `ILogger` effectively?

**Answer:**  
Inject `ILogger<T>` into your classes and use log levels appropriately.

**Log levels (increasing severity):** `Trace`, `Debug`, `Information`, `Warning`, `Error`, `Critical`.

**Example:**
```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    public OrderService(ILogger<OrderService> logger) => _logger = logger;

    public void ProcessOrder(Order order)
    {
        _logger.LogInformation("Processing order {OrderId}", order.Id);
        try
        {
            // ...
            _logger.LogDebug("Order processed successfully");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to process order {OrderId}", order.Id);
            throw;
        }
    }
}
```

**Structured logging:** Use placeholders (`{OrderId}`) instead of string concatenation for better performance and searchability.

---

## Background Tasks with `IHostedService`

### 48. How do you run a background task that executes periodically?

**Answer:**  
Implement `BackgroundService` (a base class for `IHostedService`).

**Example – a timed service that runs every 30 seconds:**
```csharp
public class TimedBackgroundService : BackgroundService
{
    private readonly ILogger<TimedBackgroundService> _logger;

    public TimedBackgroundService(ILogger<TimedBackgroundService> logger) => _logger = logger;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Background task running at: {time}", DateTimeOffset.Now);
            // Do work...

            await Task.Delay(TimeSpan.FromSeconds(30), stoppingToken);
        }
    }
}

// Register in Program.cs
builder.Services.AddHostedService<TimedBackgroundService>();
```

**Important:** The `stoppingToken` signals when the application is shutting down. Always respect it to allow graceful shutdown.

---

## Working with JSON in .NET

### 49. How do you serialise and deserialise JSON in .NET?

**Answer:**  
Use `System.Text.Json` (built‑in, fast) or `Newtonsoft.Json` (popular third‑party).

**Serialisation (object to JSON):**
```csharp
var product = new Product { Id = 1, Name = "Laptop", Price = 999.99m };
string json = JsonSerializer.Serialize(product);
```

**Deserialisation (JSON to object):**
```csharp
string json = @"{ ""Id"": 1, ""Name"": ""Laptop"", ""Price"": 999.99 }";
Product product = JsonSerializer.Deserialize<Product>(json);
```

**Customising options:**
```csharp
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    WriteIndented = true
};
string json = JsonSerializer.Serialize(product, options);
```

---

### 50. How do you handle JSON with different property names using `JsonPropertyName`?

**Answer:**  
Use `[JsonPropertyName]` attribute to map C# properties to JSON fields with different names.

**Example:**
```csharp
public class Product
{
    [JsonPropertyName("productId")]
    public int Id { get; set; }

    [JsonPropertyName("productName")]
    public string Name { get; set; }

    [JsonPropertyName("priceInUsd")]
    public decimal Price { get; set; }
}
```

Now the JSON will use `productId`, `productName`, and `priceInUsd` instead of the C# property names.

---

*This extended set of notes covers additional essential topics for intermediate .NET full‑stack developers. Practice these concepts by building small projects to solidify your understanding.*
