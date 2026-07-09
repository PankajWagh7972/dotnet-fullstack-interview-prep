
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


# What is `IHttpClientFactory` in ASP.NET Core?

## Definition

`IHttpClientFactory` is a built-in factory introduced in ASP.NET Core to **create and manage `HttpClient` instances efficiently**.

It helps prevent **socket exhaustion**, manages the lifetime of `HttpClient`, and provides centralized configuration for outgoing HTTP requests.

> **Interview Definition:**
> **`IHttpClientFactory` is used to create and manage `HttpClient` instances efficiently, avoiding socket exhaustion and enabling centralized configuration, logging, and resiliency for HTTP calls.**

---

# Why not create `HttpClient` using `new`?

❌ Bad Practice

```csharp
public async Task GetData()
{
    using var client = new HttpClient();

    var response = await client.GetAsync("https://api.example.com");
}
```

### Problem

Every `HttpClient` creates a new underlying connection.

If many requests are made:

```text
Request 1 → New Connection
Request 2 → New Connection
Request 3 → New Connection
...
Thousands of connections
        ↓
Socket Exhaustion
```

This can degrade performance and cause connection failures.

---

# Solution: `IHttpClientFactory`

The factory:

* Reuses underlying `HttpMessageHandler` instances.
* Manages connection pooling.
* Creates lightweight `HttpClient` objects.
* Improves performance and reliability.

---

# Register in `Program.cs`

```csharp
builder.Services.AddHttpClient();
```

---

# Basic Example

### Service

```csharp
public class WeatherService
{
    private readonly IHttpClientFactory _httpClientFactory;

    public WeatherService(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }

    public async Task<string> GetWeather()
    {
        var client = _httpClientFactory.CreateClient();

        return await client.GetStringAsync("https://api.example.com/weather");
    }
}
```

---

# Named Client

Useful when calling multiple external APIs.

### Register

```csharp
builder.Services.AddHttpClient("WeatherApi", client =>
{
    client.BaseAddress = new Uri("https://api.weather.com/");
    client.DefaultRequestHeaders.Add("Accept", "application/json");
});
```

### Use

```csharp
var client = _httpClientFactory.CreateClient("WeatherApi");

var response = await client.GetAsync("forecast");
```

---

# Typed Client (Recommended)

### Service

```csharp
public class WeatherApiService
{
    private readonly HttpClient _httpClient;

    public WeatherApiService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<string> GetWeather()
    {
        return await _httpClient.GetStringAsync("forecast");
    }
}
```

### Register

```csharp
builder.Services.AddHttpClient<WeatherApiService>(client =>
{
    client.BaseAddress = new Uri("https://api.weather.com/");
});
```

Now `WeatherApiService` receives a configured `HttpClient` automatically.

---

# Benefits of `IHttpClientFactory`

* ✅ Prevents socket exhaustion.
* ✅ Reuses underlying connections.
* ✅ Centralized configuration (Base URL, Headers, Timeout).
* ✅ Supports Dependency Injection.
* ✅ Easy to add logging and retry policies (e.g., Polly).

---

# Real-World Use Cases

* Calling payment gateways.
* Consuming third-party REST APIs.
* Calling microservices.
* Integrating with external systems (CRM, ERP, Azure services).

---

# Interview Questions

### **Q: Why is `HttpClient` intended to be reused?**

Creating and disposing many `HttpClient` instances can exhaust available sockets because the underlying TCP connections are not released immediately. Reusing handlers through `IHttpClientFactory` avoids this problem.

---

### **Q: What are the types of `HttpClient` registration?**

1. **Basic Client**

   ```csharp
   builder.Services.AddHttpClient();
   ```

2. **Named Client**

   ```csharp
   builder.Services.AddHttpClient("GitHub");
   ```

3. **Typed Client** (Most commonly used)

   ```csharp
   builder.Services.AddHttpClient<MyService>();
   ```

---

# Interview One-Liner

> **`IHttpClientFactory` is used to create and manage `HttpClient` instances efficiently. It prevents socket exhaustion, enables centralized configuration, integrates with dependency injection, and supports resilient HTTP communication with external services.**

---

# Quick Revision

| Feature      | `IHttpClientFactory`                               |
| ------------ | -------------------------------------------------- |
| Purpose      | Manage `HttpClient` instances                      |
| Registration | `builder.Services.AddHttpClient()`                 |
| Main Benefit | Prevents socket exhaustion                         |
| Supports DI  | ✅ Yes                                              |
| Types        | Basic, Named, Typed                                |
| Common Uses  | REST APIs, microservices, third-party integrations |

### **Memory Trick**

Think:

**External API Call → `IHttpClientFactory` → `HttpClient` → Response**

Instead of:

**`new HttpClient()` every request ❌**

Use:

**`IHttpClientFactory.CreateClient()` ✅**


# How does .NET handle Multithreading and Concurrency?

## Definition

* **Multithreading** is the ability to execute multiple threads simultaneously within a process.
* **Concurrency** is the ability to manage multiple tasks that make progress independently, even if they don't all run at the exact same time.

.NET provides built-in libraries like **Thread**, **ThreadPool**, **Task Parallel Library (TPL)**, and **async/await** to efficiently handle multithreading and concurrency.

> **Interview Definition:**
> **.NET handles multithreading and concurrency using Threads, the ThreadPool, the Task Parallel Library (TPL), and async/await, enabling efficient execution of CPU-bound and I/O-bound operations while managing thread synchronization and resource sharing.**

---

# Thread vs Task

| Thread              | Task                       |
| ------------------- | -------------------------- |
| Actual OS thread    | Higher-level abstraction   |
| Expensive to create | Lightweight                |
| Manual management   | Managed by .NET ThreadPool |
| Less preferred      | Recommended                |

---

# Ways to Handle Multithreading in .NET

## 1. Thread

Creates a dedicated thread.

```csharp
Thread thread = new Thread(() =>
{
    Console.WriteLine("Running on another thread");
});

thread.Start();
```

### Use Case

* Rarely used today.
* Full control over thread lifecycle.

---

## 2. ThreadPool

Reuses threads instead of creating new ones.

```csharp
ThreadPool.QueueUserWorkItem(_ =>
{
    Console.WriteLine("Running from ThreadPool");
});
```

### Benefits

* Faster
* Better performance
* Reduces thread creation overhead

---

## 3. Task (TPL) ⭐ Recommended

Represents an asynchronous operation.

```csharp
Task.Run(() =>
{
    Console.WriteLine("Task Running");
});
```

### Benefits

* Easier to use
* Uses ThreadPool internally
* Supports cancellation and continuations

---

## 4. async / await (For I/O-bound Work)

```csharp
public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);

    return "Completed";
}
```

### Use Case

* API calls
* Database queries
* File operations
* Network requests

**Note:** `async/await` doesn't create a new thread by itself. It allows the current thread to be released while waiting for I/O.

---

# CPU-bound vs I/O-bound

| CPU-bound          | I/O-bound         |
| ------------------ | ----------------- |
| Heavy calculations | API Calls         |
| Image processing   | Database          |
| Encryption         | File Read/Write   |
| Use `Task.Run()`   | Use `async/await` |

---

# Running Multiple Tasks Concurrently

```csharp
var task1 = Task.Delay(1000);
var task2 = Task.Delay(2000);

await Task.WhenAll(task1, task2);
```

Both tasks run concurrently, and execution continues after **both** complete.

---

# Synchronization (Avoiding Race Conditions)

When multiple threads access shared data, synchronization is required.

## `lock`

```csharp
private readonly object _lock = new();

lock (_lock)
{
    counter++;
}
```

Only one thread can execute the locked block at a time.

---

## `SemaphoreSlim`

Limits the number of concurrent operations.

```csharp
SemaphoreSlim semaphore = new SemaphoreSlim(2);

await semaphore.WaitAsync();

try
{
    // Critical section
}
finally
{
    semaphore.Release();
}
```

---

## `Monitor`

```csharp
Monitor.Enter(_lock);

try
{
    counter++;
}
finally
{
    Monitor.Exit(_lock);
}
```

`lock` is a simpler syntax over `Monitor`.

---

## Concurrent Collections

Instead of `List<T>`:

```csharp
ConcurrentDictionary<int, string>
ConcurrentQueue<T>
ConcurrentBag<T>
ConcurrentStack<T>
```

These are thread-safe.

---

# Parallel Programming

## `Parallel.For`

```csharp
Parallel.For(1, 5, i =>
{
    Console.WriteLine(i);
});
```

Runs loop iterations in parallel.

---

## `Parallel.ForEach`

```csharp
Parallel.ForEach(numbers, number =>
{
    Console.WriteLine(number);
});
```

Useful for CPU-intensive operations.

---

# Cancellation

```csharp
CancellationTokenSource cts = new();

await Task.Run(() =>
{
    while (!cts.Token.IsCancellationRequested)
    {
        // Work
    }
});
```

Allows graceful cancellation of long-running tasks.

---

# Real-World Examples

### API Calls (I/O-bound)

```csharp
public async Task<Product> GetProduct()
{
    return await _httpClient.GetFromJsonAsync<Product>("products/1");
}
```

---

### Image Processing (CPU-bound)

```csharp
await Task.Run(() =>
{
    ProcessLargeImage();
});
```

---

### Parallel Processing

```csharp
await Task.WhenAll(
    SendEmailAsync(),
    GenerateInvoiceAsync(),
    UpdateInventoryAsync()
);
```

All three operations execute concurrently.

---

# Common Interview Questions

### **Q: What is the difference between Multithreading and Concurrency?**

* **Multithreading** → Multiple threads execute within a process.
* **Concurrency** → Multiple tasks make progress independently (they may or may not run simultaneously).

---

### **Q: Should we use `Thread` or `Task`?**

Use **`Task`** in most modern applications because it is easier to use, integrates with the ThreadPool, and supports async programming.

---

### **Q: Does `async/await` create a new thread?**

**No.** It doesn't create a new thread by itself. It enables non-blocking asynchronous execution, primarily for I/O-bound operations.

---

### **Q: When do you use `Task.Run()`?**

Use `Task.Run()` to offload **CPU-bound** work from the current thread (e.g., heavy calculations). Don't use it just to wrap naturally asynchronous I/O operations.

---

# Interview One-Liner

> **.NET handles multithreading and concurrency using Threads, the ThreadPool, the Task Parallel Library (TPL), and async/await. `Task` is preferred over `Thread`, `async/await` is best for I/O-bound operations, `Task.Run()` is used for CPU-bound work, and synchronization mechanisms like `lock` and `SemaphoreSlim` help ensure thread safety.**

---

# Quick Revision

| Feature                | Purpose                                     |
| ---------------------- | ------------------------------------------- |
| `Thread`               | Manual thread creation                      |
| `ThreadPool`           | Reuse threads efficiently                   |
| `Task`                 | Preferred abstraction for asynchronous work |
| `async/await`          | Non-blocking I/O operations                 |
| `Task.WhenAll()`       | Run multiple tasks concurrently             |
| `lock`                 | Protect shared resources                    |
| `SemaphoreSlim`        | Limit concurrent access                     |
| `Parallel.For/ForEach` | Parallelize CPU-bound loops                 |
| `CancellationToken`    | Cancel long-running operations              |

### **Memory Trick**

**TTPA-LSC**

* **T**hread
* **T**hreadPool
* **P**L (`Task` / TPL)
* **A**sync/Await
* **L**ock
* **S**emaphoreSlim
* **C**ancellationToken


This topic is confusing for many developers because several concepts are related but solve different problems. Let's simplify it with **real-world analogies**.

---

# Imagine a Restaurant 🍽️

* **Customer** = Request
* **Waiter** = Thread
* **Kitchen** = CPU
* **Cooking** = Work

Now let's understand each concept.

---

# 1. What is a Thread?

A **Thread** is a worker that executes your code.

Imagine a restaurant has **1 waiter**.

```
Customer 1 → Waiter serves
Customer 2 → Waiter waits
Customer 3 → Waiter waits
```

Only one customer is served at a time.

Now hire **3 waiters**.

```
Customer 1 → Waiter 1
Customer 2 → Waiter 2
Customer 3 → Waiter 3
```

Now multiple customers are served simultaneously.

In .NET:

```
Main Thread
     │
     ├── Thread 1
     ├── Thread 2
     └── Thread 3
```

Example

```csharp
Thread t = new Thread(() =>
{
    Console.WriteLine("Running...");
});

t.Start();
```

A brand new OS thread is created.

---

# Problem with Threads

Creating threads is expensive.

Imagine hiring a new waiter for every customer.

```
Customer 1 → New Waiter
Customer 2 → New Waiter
Customer 3 → New Waiter
Customer 1000 → New Waiter
```

Very costly.

---

# 2. ThreadPool

Instead of hiring new waiters every time...

Restaurant already has **10 waiters**.

```
Customer arrives

↓

Available waiter serves

↓

Waiter becomes free

↓

Serves next customer
```

Waiters are reused.

.NET does the same.

```
Task

↓

ThreadPool

↓

Existing Thread

↓

Execute
```

Example

```csharp
ThreadPool.QueueUserWorkItem(_ =>
{
    Console.WriteLine("Hello");
});
```

No new thread created.

---

# 3. Task

A **Task** is **NOT a thread**.

A Task is simply **a unit of work**.

Think of it as:

```
Order Ticket

↓

Kitchen decides

↓

Which chef prepares it
```

You don't care who cooks.

You only care that food is prepared.

Same in .NET.

```
Task

↓

ThreadPool chooses thread

↓

Runs work
```

Example

```csharp
Task.Run(() =>
{
    Console.WriteLine("Processing...");
});
```

You never create a thread.

.NET decides.

---

# Thread vs Task

Imagine Uber.

Thread:

```
You buy your own car.

Drive yourself.
```

Task:

```
Book Uber.

Uber decides

which driver comes.
```

Task is much easier.

That's why Microsoft recommends Task.

---

# 4. async/await

This confuses almost everyone.

Imagine you're waiting for food.

Wrong way:

```
Order food

↓

Stand near kitchen

↓

Wait 20 minutes

↓

Take food
```

You're wasting time.

---

Better way:

```
Order food

↓

Sit at table

↓

Talk to friends

↓

Waiter calls you

↓

Collect food
```

That's async/await.

The thread doesn't sit idle waiting.

Example

```csharp
await httpClient.GetAsync(url);
```

What happens?

```
Thread starts request

↓

Network is slow

↓

Thread becomes FREE

↓

Another request uses thread

↓

Response comes

↓

Thread continues execution
```

This is why async is efficient for I/O operations.

---

# Does async create a new thread?

**No.**

```csharp
await File.ReadAllTextAsync("data.txt");
```

No new thread is created just because of `await`.

It simply **doesn't block** the current thread while waiting.

---

# 5. Task.Run()

Suppose you have heavy work.

```
Calculate 5 million records

↓

Compress files

↓

Generate PDF
```

This keeps the current thread busy.

Move it to another thread.

```csharp
await Task.Run(() =>
{
    ProcessLargeFile();
});
```

Here, `Task.Run()` schedules the work on a ThreadPool thread.

**Use it for CPU-bound work.**

---

# CPU-bound vs I/O-bound

## CPU-bound

Computer is busy thinking.

```
Calculate

Encrypt

Compress

Resize Image
```

Use

```csharp
Task.Run(...)
```

---

## I/O-bound

Computer is waiting.

```
Database

API

File

Network
```

Use

```csharp
await
```

No `Task.Run()` needed.

---

# Real ASP.NET Core Example

Suppose your API calls SQL Server.

```csharp
public async Task<IActionResult> Get()
{
    var users = await _db.Users.ToListAsync();

    return Ok(users);
}
```

What happens?

```
Request

↓

SQL Query Sent

↓

Database Working

↓

.NET Thread is FREE

↓

Other requests use that thread

↓

Database responds

↓

Thread continues

↓

Return Response
```

If 100 users call your API, threads are not blocked waiting on the database.

---

# Simple Comparison

| Concept     | Simple Meaning                  |
| ----------- | ------------------------------- |
| Thread      | Worker                          |
| ThreadPool  | Reusable workers                |
| Task        | Work to be done                 |
| async/await | Don't block while waiting       |
| Task.Run    | Give CPU work to another worker |
| lock        | Only one worker at a time       |

---

# Easy Interview Flow

If the interviewer asks:

**"How does .NET handle multithreading?"**

You can answer:

> ".NET provides several mechanisms for concurrency. At the lowest level, there are **Threads**, but creating them is expensive. Therefore, .NET uses the **ThreadPool** to reuse threads efficiently. Modern applications typically use **Task**, which schedules work on the ThreadPool. For I/O-bound operations like database or API calls, **async/await** allows threads to remain free while waiting, improving scalability. For CPU-bound operations, **Task.Run()** is used to execute heavy work on a ThreadPool thread. Synchronization primitives such as `lock` and `SemaphoreSlim` are used to protect shared resources when multiple threads access them."

This explanation is suitable for **5+ years of .NET interviews**.

If you're preparing for senior .NET interviews, the next topic to understand is **how `async/await` works internally (state machine, SynchronizationContext, ConfigureAwait, and why deadlocks happen)**, which is one of the most frequently asked advanced interview topics.

This is an excellent question. This is where many developers misuse `Task.Run()`.

## Scenario

Suppose you need to call two independent APIs:

* Customer API
* Order API

Both should run **at the same time**.

### ❌ Should you use `Task.Run()`?

**No.**

Because calling an API is an **I/O-bound** operation, not a CPU-bound operation.

Using `Task.Run()` here only wastes a ThreadPool thread.

---

## ✅ Correct Way: Start Both Tasks and Await Them Together

```csharp
public async Task<IActionResult> GetDashboard()
{
    Task<Customer> customerTask = _customerService.GetCustomerAsync();
    Task<List<Order>> orderTask = _orderService.GetOrdersAsync();

    await Task.WhenAll(customerTask, orderTask);

    Customer customer = await customerTask;
    List<Order> orders = await orderTask;

    return Ok(new
    {
        customer,
        orders
    });
}
```

### What happens internally?

```
Request comes
      │
      ▼
Start Customer API Call
      │
Start Order API Call
      │
      ▼
Both APIs are running concurrently
      │
      ▼
Current thread is FREE while waiting
      │
      ▼
Both responses arrive
      │
      ▼
Continue execution
```

Notice that **no extra thread is created** just because two API calls are in progress.

---

## ❌ Wrong Way

```csharp
var customer = await _customerService.GetCustomerAsync();
var orders = await _orderService.GetOrdersAsync();
```

Execution:

```
Call Customer API
      │
Wait
      │
Response received
      │
Call Order API
      │
Wait
      │
Response received
```

Total time:

```
Customer API = 2 sec
Order API = 3 sec

Total = 5 sec
```

---

## ✅ Concurrent Way

```csharp
var customerTask = _customerService.GetCustomerAsync();
var orderTask = _orderService.GetOrdersAsync();

await Task.WhenAll(customerTask, orderTask);
```

Execution:

```
Customer API  ----2 sec----
Order API     -------3 sec-------

Total ≈ 3 sec
```

Because both requests started immediately.

---

# When should I use `Task.Run()`?

Suppose after getting the API response, you need to generate a large PDF.

```csharp
public async Task<IActionResult> DownloadReport()
{
    var data = await _api.GetDataAsync();

    var pdf = await Task.Run(() =>
    {
        return GenerateLargePdf(data);
    });

    return File(pdf, "application/pdf");
}
```

Here:

* API call → **I/O-bound** → `await`
* PDF generation → **CPU-bound** → `Task.Run()`

This is an appropriate use of `Task.Run()`.

---

# Interview Question

### **Can we use `Task.WhenAll()` for API calls?**

**Yes.** It's the recommended approach when the operations are independent.

```csharp
var task1 = _service1.GetAsync();
var task2 = _service2.GetAsync();

await Task.WhenAll(task1, task2);
```

---

### **Should we wrap API calls inside `Task.Run()`?**

**No.**

```csharp
// ❌ Don't do this
await Task.Run(() => _httpClient.GetAsync(url));
```

Why?

* `HttpClient.GetAsync()` is already asynchronous.
* `Task.Run()` only adds unnecessary scheduling overhead.
* It doesn't make the API call faster.

---

# Rule to Remember

| Scenario           | Use              |
| ------------------ | ---------------- |
| Database call      | `await`          |
| API call           | `await`          |
| File I/O           | `await`          |
| HTTP call          | `await`          |
| Multiple API calls | `Task.WhenAll()` |
| Image processing   | `Task.Run()`     |
| PDF generation     | `Task.Run()`     |
| Encryption         | `Task.Run()`     |
| Heavy calculations | `Task.Run()`     |

---

## Easy Memory Trick

Ask yourself:

> **"Is the computer waiting, or is the computer working?"**

* **Waiting** (API, DB, File, Network) → **`async/await` + `Task.WhenAll()` if multiple**
* **Working** (calculation, compression, image processing, PDF generation) → **`Task.Run()`**

### Interview One-Liner

> **For multiple independent API calls, I start both asynchronous operations and use `Task.WhenAll()` to await them concurrently. I do not use `Task.Run()` because HTTP calls are I/O-bound and are already asynchronous. `Task.Run()` is intended for CPU-bound work, not for wrapping asynchronous I/O operations.**
