
## 1. Difference between `var`, `dynamic`, and `object` (Interview Notes)

| Feature          | `var`                | `dynamic`             | `object`              |
| ---------------- | -------------------- | --------------------- | --------------------- |
| Type Checking    | Compile Time         | Runtime               | Compile Time          |
| Actual Type      | Inferred by compiler | Determined at runtime | Always `object`       |
| Type Safety      | ✅ Strongly Typed     | ❌ Not Type Safe       | ✅ Strongly Typed      |
| Casting Required | ❌ No                 | ❌ No                  | ✅ Yes                 |
| Performance      | Fast                 | Slower                | Fast                  |
| IntelliSense     | ✅ Full               | ⚠️ Limited            | Only `object` members |

---

## 1. `var`

* Compiler automatically infers the type from the assigned value.
* Type is fixed after initialization.
* Checked at **compile time**.

```csharp
var name = "Pankaj";   // string
var age = 25;          // int

name = "John";         // ✅
name = 100;            // ❌ Compile-time error
```

**Use when:** The type is obvious (LINQ, long generic types).

---

## 2. `dynamic`

* Type is resolved at **runtime**.
* No compile-time type checking.
* Can change its type anytime.

```csharp
dynamic data = "Hello";
Console.WriteLine(data.Length);   // ✅

data = 100;
Console.WriteLine(data.Length);   // ❌ Runtime Exception
```

**Use when:** Reflection, COM Interop, dynamic JSON, scripting.

---

## 3. `object`

* Base class of all C# types.
* Can store any type.
* Requires **casting** to access specific members.

```csharp
object name = "Pankaj";

// Console.WriteLine(name.Length); // ❌ Error

Console.WriteLine(((string)name).Length); // ✅
```

**Use when:** You need a generic container for different types.

---

## Interview Difference

```csharp
var a = "Hello";
dynamic b = "Hello";
object c = "Hello";

Console.WriteLine(a.Length);          // ✅
Console.WriteLine(b.Length);          // ✅
Console.WriteLine(((string)c).Length); // ✅
```

---

## Memory Trick

* **`var`** → **Compiler knows the type** (Compile Time)
* **`dynamic`** → **Runtime knows the type** (Runtime)
* **`object`** → **Stores any type, but requires casting**

---

## Interview One-Liner

* **`var`** → *Compile-time type inference, strongly typed.*
* **`dynamic`** → *Runtime type resolution, bypasses compile-time type checking.*
* **`object`** → *Base class of all C# types; requires casting to access specific members.*

# 2. Difference Between Early Binding and Late Binding (C# Interview Notes)

## Definition

### Early Binding

* Method call is resolved at **compile time**.
* Compiler knows the object's type.
* Faster and type-safe.

### Late Binding

* Method call is resolved at **runtime**.
* Compiler doesn't know the object's type.
* Slower and less type-safe.

---

## Comparison

| Feature             | Early Binding | Late Binding    |
| ------------------- | ------------- | --------------- |
| Binding Time        | Compile Time  | Runtime         |
| Type Checking       | Compile Time  | Runtime         |
| Performance         | Faster        | Slower          |
| IntelliSense        | ✅ Available   | ❌ Not Available |
| Compile-Time Errors | ✅ Yes         | ❌ No            |
| Runtime Errors      | Rare          | Possible        |

---

## Early Binding Example

```csharp
class Employee
{
    public void Display()
    {
        Console.WriteLine("Employee");
    }
}

Employee emp = new Employee();
emp.Display();
```

**Output**

```
Employee
```

✔ Compiler knows `Display()` exists.

---

## Late Binding Example (`dynamic`)

```csharp
dynamic emp = new Employee();
emp.Display();
```

The compiler doesn't verify whether `Display()` exists.

If you call a non-existing method:

```csharp
dynamic emp = new Employee();
emp.Show();
```

✔ Compiles successfully.

❌ At runtime:

```
RuntimeBinderException
```

---

## Another Late Binding Example (Reflection)

```csharp
Type type = typeof(Employee);

object obj = Activator.CreateInstance(type);

type.GetMethod("Display").Invoke(obj, null);
```

The method is located and invoked at **runtime**.

---

## Real-World Uses

### Early Binding

* Normal C# applications
* ASP.NET Core
* Business logic
* Most day-to-day development

### Late Binding

* Reflection
* Plugin architectures
* COM Interop (Excel, Word)
* Dynamic JSON handling
* Loading assemblies at runtime

---

## Memory Trick

* **Early Binding** → **Compiler knows everything** (Fast & Safe)
* **Late Binding** → **Runtime decides everything** (Flexible but Slower)

---

## Interview One-Liner

> **Early Binding** resolves method calls at **compile time**, providing **better performance and compile-time type safety**.
> **Late Binding** resolves method calls at **runtime**, offering **more flexibility** but with **runtime overhead and the possibility of runtime errors**.

### 3. What are Filters in ASP.NET Core?

## Definition

**Filters** are components in ASP.NET Core that allow you to execute code **before or after** specific stages of the request processing pipeline, such as before an action executes, after an action completes, before a result is returned, or when an exception occurs.

They are commonly used for **cross-cutting concerns** like logging, authentication, authorization, exception handling, and validation.

---

## Request Flow

```text
HTTP Request
      │
      ▼
Authorization Filter
      │
      ▼
Resource Filter
      │
      ▼
Action Filter (Before)
      │
      ▼
Controller Action
      │
      ▼
Action Filter (After)
      │
      ▼
Result Filter (Before)
      │
      ▼
Result Execution
      │
      ▼
Result Filter (After)
      │
      ▼
HTTP Response

(If an exception occurs → Exception Filter)
```

---

# Types of Filters

| Filter Type              | Runs When                    | Common Use                     |
| ------------------------ | ---------------------------- | ------------------------------ |
| **Authorization Filter** | Before anything else         | Authentication & Authorization |
| **Resource Filter**      | Before/After model binding   | Caching, resource management   |
| **Action Filter**        | Before & after action method | Logging, validation, timing    |
| **Exception Filter**     | When an exception occurs     | Error handling, logging        |
| **Result Filter**        | Before & after action result | Modify response, headers       |

---

# 1. Authorization Filter

### Purpose

Checks whether the user is authorized before executing the request.

### Runs

✔ First filter in the pipeline.

### Example

```csharp
[Authorize]
public IActionResult Dashboard()
{
    return View();
}
```

### Real-world Uses

* Login authentication
* Role-based authorization
* JWT token validation

---

# 2. Resource Filter

### Purpose

Runs before model binding and after the response is generated.

### Use Cases

* Response caching
* Performance monitoring
* Resource allocation/cleanup

---

# 3. Action Filter

### Purpose

Executes before and after the controller action.

### Example

```csharp
public class LogActionFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        Console.WriteLine("Before Action");
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        Console.WriteLine("After Action");
    }
}
```

Apply it:

```csharp
[LogActionFilter]
public IActionResult Index()
{
    return View();
}
```

### Real-world Uses

* Logging
* Input validation
* Measuring execution time
* Auditing

---

# 4. Exception Filter

### Purpose

Handles unhandled exceptions thrown by the action.

### Example

```csharp
public class CustomExceptionFilter : ExceptionFilterAttribute
{
    public override void OnException(ExceptionContext context)
    {
        Console.WriteLine(context.Exception.Message);

        context.Result = new ContentResult
        {
            Content = "Something went wrong."
        };

        context.ExceptionHandled = true;
    }
}
```

### Real-world Uses

* Global exception handling
* Logging errors
* Returning custom error responses

---

# 5. Result Filter

### Purpose

Runs before and after the action result is executed.

### Example

```csharp
public class HeaderFilter : ResultFilterAttribute
{
    public override void OnResultExecuting(ResultExecutingContext context)
    {
        context.HttpContext.Response.Headers.Add("App-Version", "1.0");
    }
}
```

### Real-world Uses

* Add response headers
* Modify response
* Compress or transform output

---

# Filter Execution Order

```text
Request
   │
   ▼
Authorization Filter
   │
   ▼
Resource Filter
   │
   ▼
Action Filter (Before)
   │
   ▼
Controller Action
   │
   ▼
Action Filter (After)
   │
   ▼
Result Filter (Before)
   │
   ▼
Result Execution
   │
   ▼
Result Filter (After)
   │
   ▼
Response

Exception Filter executes only if an unhandled exception occurs.
```

---

# Interview One-Liner

> **Filters in ASP.NET Core are reusable components that execute code before or after different stages of request processing. They are used to implement cross-cutting concerns such as authentication, authorization, logging, validation, caching, and exception handling.**

---

# Quick Revision Table

| Filter            | Executes                   | Purpose                        |
| ----------------- | -------------------------- | ------------------------------ |
| **Authorization** | Before everything          | Authentication & Authorization |
| **Resource**      | Before/After model binding | Caching, resource management   |
| **Action**        | Before/After action        | Logging, validation, auditing  |
| **Exception**     | On exception               | Error handling                 |
| **Result**        | Before/After result        | Modify response, headers       |

### **Memory Trick**

* **Authorization** → **Who are you?**
* **Resource** → **Prepare resources**
* **Action** → **Before & after business logic**
* **Exception** → **Handle errors**
* **Result** → **Modify the response before sending it**


# 4. What is Model Binding in ASP.NET Core?

## Definition

**Model Binding** is the process by which **ASP.NET Core automatically maps data from an HTTP request to action method parameters or model objects.**

Instead of manually reading values from the request, ASP.NET Core binds them automatically.

---

## Example

### Request

```http
GET /api/users/10?name=Pankaj
```

### Controller

```csharp
[HttpGet("{id}")]
public IActionResult GetUser(int id, string name)
{
    return Ok($"{id} - {name}");
}
```

ASP.NET Core automatically binds:

* `id = 10` (Route)
* `name = "Pankaj"` (Query String)

No manual parsing is required.

---

# How Model Binding Works

```text
HTTP Request
      │
      ▼
Route Values
Query String
Form Data
Request Body
Headers
      │
      ▼
Model Binder
      │
      ▼
C# Parameters / Model Object
      │
      ▼
Controller Action
```

---

# Sources of Model Binding

| Attribute          | Data Source             | Example                |
| ------------------ | ----------------------- | ---------------------- |
| **[FromRoute]**    | Route values            | `/users/10`            |
| **[FromQuery]**    | Query string            | `?name=Pankaj`         |
| **[FromBody]**     | Request body (JSON/XML) | POST request           |
| **[FromForm]**     | Form data               | HTML form              |
| **[FromHeader]**   | HTTP headers            | Authorization, API Key |
| **[FromServices]** | Dependency Injection    | Inject a service       |

---

# Examples

## 1. From Route

```csharp
[HttpGet("{id}")]
public IActionResult Get([FromRoute] int id)
{
    return Ok(id);
}
```

Request

```http
GET /users/10
```

Output

```text
10
```

---

## 2. From Query

```csharp
[HttpGet]
public IActionResult Search([FromQuery] string name)
{
    return Ok(name);
}
```

Request

```http
GET /users?name=Pankaj
```

Output

```text
Pankaj
```

---

## 3. From Body

Request Body

```json
{
    "name":"Pankaj",
    "age":28
}
```

Controller

```csharp
public class User
{
    public string Name { get; set; }
    public int Age { get; set; }
}

[HttpPost]
public IActionResult Create([FromBody] User user)
{
    return Ok(user);
}
```

The JSON is automatically converted into a `User` object.

---

## 4. From Form

```csharp
[HttpPost]
public IActionResult Save([FromForm] User user)
{
    return Ok(user);
}
```

Used for HTML form submissions and file uploads.

---

## 5. From Header

```csharp
[HttpGet]
public IActionResult Get([FromHeader] string apiKey)
{
    return Ok(apiKey);
}
```

Header

```http
apiKey: 12345
```

---

# Complex Model Binding

Instead of multiple parameters:

```csharp
public IActionResult Save(string name, int age)
```

Use a model:

```csharp
public class User
{
    public string Name { get; set; }
    public int Age { get; set; }
}

public IActionResult Save(User user)
```

ASP.NET Core automatically fills the object from the request.

---

# Model Validation with Model Binding

```csharp
public class User
{
    [Required]
    public string Name { get; set; }

    [Range(18,60)]
    public int Age { get; set; }
}
```

Controller

```csharp
[HttpPost]
public IActionResult Create(User user)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    return Ok(user);
}
```

The model is **bound first**, then **validated**.

---

# Real-World Example

Request

```http
POST /api/employees
```

Body

```json
{
    "name":"John",
    "salary":50000
}
```

Controller

```csharp
[HttpPost]
public IActionResult AddEmployee(Employee employee)
{
    // employee.Name = John
    // employee.Salary = 50000

    return Ok(employee);
}
```

The framework automatically converts the JSON into an `Employee` object.

---

# Interview One-Liner

> **Model Binding is the ASP.NET Core feature that automatically maps data from an HTTP request (route values, query string, form data, headers, or request body) to action method parameters or model objects, reducing manual parsing and simplifying controller code.**

---

# Quick Revision

* **Purpose:** Convert HTTP request data into C# objects.
* **Data Sources:** Route, Query String, Body, Form, Headers, Services.
* **Works for:** Primitive types and complex models.
* **Benefit:** Less boilerplate code, automatic conversion, and integration with model validation.

### **Memory Trick**

**RQBFHS** → **R**oute, **Q**uery, **B**ody, **F**orm, **H**eader, **S**ervices.


# What is the Host in ASP.NET Core?

## Definition

The **Host** is the **main object responsible for starting and managing the ASP.NET Core application**.

It is responsible for:

* Creating the Dependency Injection (DI) container.
* Loading configuration (`appsettings.json`, environment variables, etc.).
* Configuring logging.
* Starting the web server (Kestrel).
* Managing the application lifetime (startup and shutdown).

> **Simple Definition (Interview):**
> **Host is the runtime environment that creates, configures, starts, and manages an ASP.NET Core application.**

---

# Program.cs (.NET 6/7/8 Minimal Hosting Model)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register Services
builder.Services.AddControllers();

var app = builder.Build();

// Configure Middleware
app.UseHttpsRedirection();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

Let's understand each line.

---

# 1. WebApplication.CreateBuilder(args)

```csharp
var builder = WebApplication.CreateBuilder(args);
```

## What happens internally?

This is the **Host Builder**.

It automatically creates:

* Host
* Web Server (Kestrel)
* Configuration
* Dependency Injection Container
* Logging
* Environment
* Default Services

Internally it is similar to:

```csharp
Host.CreateDefaultBuilder(args)
```

---

## It automatically loads

```
appsettings.json

appsettings.Development.json

Environment Variables

Command Line Arguments

User Secrets (Development)

Logging

Dependency Injection

Kestrel Server
```

---

# Builder contains

```
builder
│
├── Services
├── Configuration
├── Environment
├── Logging
├── Host
└── WebHost
```

---

# 2. builder.Services

```csharp
builder.Services.AddControllers();
```

This is the **Dependency Injection Container**.

All services are registered here.

Example

```csharp
builder.Services.AddControllers();

builder.Services.AddDbContext<AppDbContext>();

builder.Services.AddScoped<IEmployeeRepository, EmployeeRepository>();

builder.Services.AddAuthentication();

builder.Services.AddAuthorization();

builder.Services.AddSwaggerGen();
```

Everything your application needs is registered here.

---

# 3. builder.Configuration

Used to read configuration.

```csharp
string cs =
builder.Configuration.GetConnectionString("DefaultConnection");
```

Reads from

```
appsettings.json

Environment Variables

Azure Key Vault

Secrets

etc.
```

---

# 4. builder.Environment

Gives current environment.

```csharp
if(builder.Environment.IsDevelopment())
{
    // Development code
}
```

Possible values

```
Development

Staging

Production
```

---

# 5. builder.Logging

Configure logging.

```csharp
builder.Logging.ClearProviders();

builder.Logging.AddConsole();

builder.Logging.AddDebug();
```

---

# 6. builder.Host

Configure Host.

Example

```csharp
builder.Host.ConfigureServices(...);

builder.Host.ConfigureLogging(...);
```

Rarely modified in normal projects.

---

# 7. builder.WebHost

Configure the web server.

Example

```csharp
builder.WebHost.UseKestrel();
```

or

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(5000);
});
```

---

# 8. builder.Build()

```csharp
var app = builder.Build();
```

Creates the application object.

Now all services are ready.

After Build()

```
Host Created

DI Ready

Configuration Ready

Logging Ready

Middleware Pipeline Ready
```

---

# app object

```
app
│
├── Middleware
├── Routing
├── Endpoints
├── Lifetime
└── Run()
```

---

# 9. Middleware Configuration

Example

```csharp
app.UseHttpsRedirection();

app.UseAuthentication();

app.UseAuthorization();
```

Middleware executes one after another.

Pipeline

```
Request

↓

Middleware1

↓

Middleware2

↓

Middleware3

↓

Controller

↓

Response
```

---

# 10. app.MapControllers()

```csharp
app.MapControllers();
```

Maps all controller endpoints.

Without this

```
/api/employees
```

will never reach the controller.

---

# 11. app.Run()

```csharp
app.Run();
```

Starts the application.

Internally

```
Starts Kestrel

Starts listening on Port

Waits for HTTP Requests
```

Application keeps running until stopped.

---

# Overall Flow

```
Program.cs Starts

↓

Create Host

↓

Load Configuration

↓

Create DI Container

↓

Register Services

↓

Build Application

↓

Configure Middleware

↓

Map Endpoints

↓

Run Kestrel Server

↓

Wait for Requests
```

---

# Common Objects in Program.cs

| Object                  | Purpose                                                    |
| ----------------------- | ---------------------------------------------------------- |
| `builder`               | Creates and configures the Host                            |
| `builder.Services`      | Register services for Dependency Injection                 |
| `builder.Configuration` | Read configuration values                                  |
| `builder.Environment`   | Get current environment (Development, Staging, Production) |
| `builder.Logging`       | Configure logging providers                                |
| `builder.Host`          | Configure the generic host                                 |
| `builder.WebHost`       | Configure the web server (Kestrel, IIS integration)        |
| `app`                   | Built application used to configure middleware             |
| `app.Use...()`          | Add middleware to the request pipeline                     |
| `app.MapControllers()`  | Map controller routes/endpoints                            |
| `app.Run()`             | Start the web server and begin listening for requests      |

---

# Interview One-Liner

> **`WebApplication.CreateBuilder(args)` creates the ASP.NET Core Host, which sets up configuration, dependency injection, logging, the web server (Kestrel), and the application environment. `builder.Services` registers services, `builder.Build()` creates the application, `app.Use...()` configures the middleware pipeline, `app.MapControllers()` maps endpoints, and `app.Run()` starts the server to handle incoming requests.**

---

# Memory Trick

Remember the startup flow as:

**Create → Register → Build → Configure → Map → Run**

* **Create** → `WebApplication.CreateBuilder()`
* **Register** → `builder.Services`
* **Build** → `builder.Build()`
* **Configure** → `app.Use...()`
* **Map** → `app.MapControllers()`
* **Run** → `app.Run()`

This sequence is the lifecycle of every modern ASP.NET Core application startup.


What is IHttpContextAccessor used for?
# What is `IHttpContextAccessor` in ASP.NET Core?

## Definition

`IHttpContextAccessor` is a service that provides access to the **current `HttpContext`** from classes where `HttpContext` is not directly available (such as services, repositories, or helper classes).

> **Interview Definition:**
> **`IHttpContextAccessor` is used to access the current HTTP request and response context outside of controllers or middleware.**

---

# Why do we need it?

Inside a **Controller**, you can directly access:

```csharp
HttpContext
```

Example:

```csharp
public IActionResult Index()
{
    var user = HttpContext.User.Identity?.Name;
    return Ok(user);
}
```

But inside a **Service**, `HttpContext` is **not available**.

```csharp
public class UserService
{
    // ❌ HttpContext is not accessible here
}
```

To access it, inject `IHttpContextAccessor`.

---

# Register the Service

In `Program.cs`:

```csharp
builder.Services.AddHttpContextAccessor();
```

---

# Example

### Service

```csharp
public class UserService
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public UserService(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    public string GetUserName()
    {
        return _httpContextAccessor.HttpContext?.User?.Identity?.Name;
    }
}
```

---

### Controller

```csharp
public class UserController : Controller
{
    private readonly UserService _userService;

    public UserController(UserService userService)
    {
        _userService = userService;
    }

    public IActionResult Profile()
    {
        return Ok(_userService.GetUserName());
    }
}
```

---

# What can you access using `HttpContext`?

```csharp
var context = _httpContextAccessor.HttpContext;
```

| Property          | Description                    |
| ----------------- | ------------------------------ |
| `User`            | Logged-in user information     |
| `Request`         | HTTP request details           |
| `Response`        | HTTP response                  |
| `Session`         | Session data                   |
| `Request.Headers` | Request headers                |
| `Request.Cookies` | Cookies                        |
| `Connection`      | Client IP, Port                |
| `Items`           | Data shared during the request |

---

# Real-World Use Cases

### 1. Get Logged-in User

```csharp
var userName = _httpContextAccessor.HttpContext?.User?.Identity?.Name;
```

---

### 2. Get User ID from JWT Claims

```csharp
var userId = _httpContextAccessor.HttpContext?
    .User?
    .FindFirst("sub")?
    .Value;
```

---

### 3. Get Client IP Address

```csharp
var ip = _httpContextAccessor.HttpContext?
    .Connection
    .RemoteIpAddress?
    .ToString();
```

---

### 4. Read Request Header

```csharp
var apiKey = _httpContextAccessor.HttpContext?
    .Request
    .Headers["ApiKey"];
```

---

### 5. Read Session Value

```csharp
var user = _httpContextAccessor.HttpContext?
    .Session
    .GetString("UserName");
```

---

# When should you use it?

✅ Good use cases:

* Audit logging (Current User ID)
* Multi-tenant applications (Tenant ID from headers/claims)
* Reading JWT claims in services
* Accessing request headers or client IP in business services

❌ Avoid:

* Passing `HttpContext` deep into repositories or domain models.
* Overusing it when the required data (e.g., User ID) can be passed as a method parameter.

---

# Interview Question

### **Q: Why not use `HttpContext` directly in a service?**

**Answer:**
Because `HttpContext` is only directly available in controllers, Razor Pages, and middleware. Services are independent of the HTTP pipeline, so `IHttpContextAccessor` is used to access the current `HttpContext` through dependency injection.

---

# Interview One-Liner

> **`IHttpContextAccessor` is a service that provides access to the current `HttpContext` in classes where `HttpContext` isn't directly available, such as services or helper classes. It is commonly used to access the current user, request headers, session, cookies, or client IP.**

---

# Quick Revision

| Feature       | `IHttpContextAccessor`                                                 |
| ------------- | ---------------------------------------------------------------------- |
| Purpose       | Access current `HttpContext` outside controllers                       |
| Registration  | `builder.Services.AddHttpContextAccessor();`                           |
| Common Uses   | User claims, headers, session, cookies, IP address                     |
| Access Method | `_httpContextAccessor.HttpContext`                                     |
| Interview Tip | Use it sparingly; prefer passing only the required data when possible. |



# What is a Hosted Service in ASP.NET Core?

## Definition

A **Hosted Service** is a **background service** that runs independently of HTTP requests. It starts when the application starts and stops when the application shuts down.

It is used for **long-running or scheduled background tasks**.

> **Interview Definition:**
> **A Hosted Service is a background process managed by the ASP.NET Core Host that starts with the application and runs independently of incoming HTTP requests.**

---

# Why do we need Hosted Services?

Normally, controller actions execute **only when an HTTP request arrives**.

Some tasks need to run **even when no user is making a request**, such as:

* Sending scheduled emails
* Processing background jobs
* Generating reports
* Cleaning temporary files
* Polling external APIs
* Processing message queues (RabbitMQ, Azure Service Bus)
* Refreshing cache periodically

These are ideal use cases for Hosted Services.

---

# How It Works

```text
Application Starts
        │
        ▼
ASP.NET Core Host
        │
        ▼
Hosted Service Starts
        │
        ▼
Runs in Background
        │
        ▼
Application Stops
        │
        ▼
Hosted Service Stops
```

---

# Types of Hosted Services

### 1. `IHostedService`

* Basic interface for background tasks.
* You implement `StartAsync()` and `StopAsync()`.

### 2. `BackgroundService` (Recommended)

* Abstract class that implements `IHostedService`.
* You only override `ExecuteAsync()`.

Most ASP.NET Core applications use **`BackgroundService`**.

---

# Example Using `BackgroundService`

```csharp
public class EmailBackgroundService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            Console.WriteLine("Sending scheduled emails...");

            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }
}
```

---

# Register in `Program.cs`

```csharp
builder.Services.AddHostedService<EmailBackgroundService>();
```

The service starts automatically when the application starts.

---

# Lifecycle

```text
Application Starts
        │
        ▼
ExecuteAsync() Starts
        │
        ▼
Runs Continuously
        │
        ▼
Cancellation Token Triggered
        │
        ▼
Graceful Shutdown
```

---

# Using `IHostedService`

```csharp
public class MyHostedService : IHostedService
{
    public Task StartAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("Service Started");
        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("Service Stopped");
        return Task.CompletedTask;
    }
}
```

Register:

```csharp
builder.Services.AddHostedService<MyHostedService>();
```

---

# Real-World Examples

### 1. Email Scheduler

```text
Every 5 minutes
        ↓
Check Pending Emails
        ↓
Send Emails
```

---

### 2. Cache Refresh

```text
Every 30 minutes
        ↓
Read Database
        ↓
Update Memory Cache
```

---

### 3. Queue Processing

```text
RabbitMQ Queue
        ↓
Hosted Service
        ↓
Read Message
        ↓
Process Order
```

---

### 4. File Cleanup

```text
Every Night
        ↓
Delete Old Files
```

---

# Difference Between Controller and Hosted Service

| Feature  | Controller             | Hosted Service              |
| -------- | ---------------------- | --------------------------- |
| Trigger  | HTTP Request           | Application Start           |
| Runs     | On demand              | Background                  |
| Lifetime | Per request            | Entire application lifetime |
| Purpose  | Handle client requests | Background processing       |

---

# Interview Questions

### **Q: Can a Hosted Service access Dependency Injection?**

**Yes.**

Example:

```csharp
public class ReportService : BackgroundService
{
    private readonly ILogger<ReportService> _logger;

    public ReportService(ILogger<ReportService> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Running...");
    }
}
```

---

### **Q: Can we use `DbContext` directly in a Hosted Service?**

**Not directly.**

`DbContext` is **Scoped**, while Hosted Services are **Singleton** by default.

Use `IServiceScopeFactory` to create a scope:

```csharp
using var scope = _serviceScopeFactory.CreateScope();
var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
```

---

# Interview One-Liner

> **A Hosted Service is a background service managed by the ASP.NET Core Host that starts with the application and runs independently of HTTP requests. It is commonly used for scheduled jobs, queue processing, cache refresh, and other long-running background tasks.**

---

# Quick Revision

| Feature                | Hosted Service                                |
| ---------------------- | --------------------------------------------- |
| Purpose                | Run background tasks                          |
| Starts                 | When application starts                       |
| Stops                  | When application stops                        |
| Interface              | `IHostedService`                              |
| Recommended Base Class | `BackgroundService`                           |
| Registration           | `builder.Services.AddHostedService<T>()`      |
| Common Uses            | Emails, queues, cache refresh, scheduled jobs |

### **Memory Trick**

**Hosted Service = Background Worker**

Think:

**Application Starts → Background Task Runs → Application Stops → Task Stops**
