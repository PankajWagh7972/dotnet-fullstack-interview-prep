**CQRS (Command Query Responsibility Segregation)** is an architectural pattern that separates **read operations (Queries)** from **write operations (Commands)**. Instead of using the same model for both reading and updating data, CQRS uses separate models and handlers.

This pattern is commonly used in **ASP.NET Core Web APIs**, especially with **MediatR**, **Entity Framework Core**, and **Clean Architecture**.

---

# Why use CQRS?

Without CQRS:

```
Controller
      |
      V
Service
      |
      V
Repository
      |
      V
Database
```

The same service handles:

* Create
* Update
* Delete
* Get By Id
* Get All

As applications grow, services become very large.

Example:

```csharp
public class EmployeeService
{
    public Task<Employee> GetEmployee(int id) { }

    public Task<List<Employee>> GetEmployees() { }

    public Task AddEmployee(Employee employee) { }

    public Task UpdateEmployee(Employee employee) { }

    public Task DeleteEmployee(int id) { }
}
```

Eventually this service may contain dozens of methods.

---

# CQRS Solution

Split into:

* Commands → Change data
* Queries → Read data

```
                Request

                   |
             MediatR (Mediator)

          /                    \
Command Handler          Query Handler

     |                        |
 Write DB                 Read DB
```

---

# Command

A command changes data.

Examples:

* Create Employee
* Update Employee
* Delete Employee

Commands should **not return data**, except perhaps the new ID or a success flag.

Example:

```csharp
public record CreateEmployeeCommand(
    string Name,
    string Email
) : IRequest<int>;
```

Returns the created Employee ID.

---

## Command Handler

```csharp
public class CreateEmployeeCommandHandler
    : IRequestHandler<CreateEmployeeCommand, int>
{
    private readonly AppDbContext _context;

    public CreateEmployeeCommandHandler(AppDbContext context)
    {
        _context = context;
    }

    public async Task<int> Handle(
        CreateEmployeeCommand request,
        CancellationToken cancellationToken)
    {
        var employee = new Employee
        {
            Name = request.Name,
            Email = request.Email
        };

        _context.Employees.Add(employee);

        await _context.SaveChangesAsync(cancellationToken);

        return employee.Id;
    }
}
```

---

# Query

A query only retrieves data.

It should **never modify the database**.

Example:

```csharp
public record GetEmployeeByIdQuery(int Id)
    : IRequest<EmployeeDto>;
```

---

## Query Handler

```csharp
public class GetEmployeeByIdQueryHandler
    : IRequestHandler<GetEmployeeByIdQuery, EmployeeDto>
{
    private readonly AppDbContext _context;

    public GetEmployeeByIdQueryHandler(AppDbContext context)
    {
        _context = context;
    }

    public async Task<EmployeeDto> Handle(
        GetEmployeeByIdQuery request,
        CancellationToken cancellationToken)
    {
        return await _context.Employees
            .Where(x => x.Id == request.Id)
            .Select(x => new EmployeeDto
            {
                Id = x.Id,
                Name = x.Name,
                Email = x.Email
            })
            .FirstOrDefaultAsync(cancellationToken);
    }
}
```

Notice:

* Query returns a DTO.
* It does not expose the EF entity.
* It does not call `SaveChanges()`.

---

# Controller

Instead of calling services directly:

```csharp
public class EmployeeController : ControllerBase
{
    private readonly IMediator _mediator;

    public EmployeeController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<IActionResult> Create(CreateEmployeeCommand command)
    {
        var id = await _mediator.Send(command);

        return Ok(id);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> Get(int id)
    {
        var employee = await _mediator.Send(
            new GetEmployeeByIdQuery(id));

        return Ok(employee);
    }
}
```

Controller doesn't know:

* EF Core
* Repository
* Business logic

It only sends requests to the mediator.

---

# Folder Structure

```
Application
│
├── Commands
│     ├── CreateEmployee
│     │      ├── CreateEmployeeCommand.cs
│     │      └── CreateEmployeeCommandHandler.cs
│
├── Queries
│     ├── GetEmployeeById
│     │      ├── GetEmployeeByIdQuery.cs
│     │      └── GetEmployeeByIdQueryHandler.cs
│
├── DTOs
│
├── Interfaces
│
Infrastructure
│
├── Persistence
│
Domain
│
API
```

This organization scales well for large projects.

---

# MediatR Registration

Install:

```bash
dotnet add package MediatR
```

Register in `Program.cs`:

```csharp
builder.Services.AddMediatR(cfg =>
{
    cfg.RegisterServicesFromAssembly(typeof(CreateEmployeeCommand).Assembly);
});
```

---

# Request Flow

```
HTTP POST /employees

        |
        V

EmployeeController

        |
        V

Mediator.Send(command)

        |
        V

CreateEmployeeCommandHandler

        |
        V

EF Core

        |
        V

SQL Server
```

For GET:

```
HTTP GET /employees/10

       |
       V

Controller

       |
       V

Mediator.Send(query)

       |
       V

GetEmployeeQueryHandler

       |
       V

Database

       |
       V

Employee DTO
```

---

# Benefits

* **Single Responsibility:** Each handler has one job.
* **Better maintainability:** Commands and queries are isolated.
* **Easy testing:** Handlers can be unit tested independently.
* **Scalability:** Read and write logic can evolve separately.
* **Supports Clean Architecture:** Business logic stays in the Application layer.
* **Extensibility:** Easy to add validation, logging, caching, or authorization via MediatR pipeline behaviors.

---

# When to Use CQRS

Use CQRS when:

* Large enterprise applications
* Complex business rules
* Microservices
* Applications with many read/write operations
* Systems requiring different optimization for reads and writes

Avoid CQRS for:

* Small CRUD applications
* Simple APIs where the extra abstraction adds unnecessary complexity

---

# CQRS vs Traditional CRUD

| Traditional CRUD                         | CQRS                                             |
| ---------------------------------------- | ------------------------------------------------ |
| One service handles everything           | Separate command and query handlers              |
| Same model for reads and writes          | Different models for reads and writes            |
| Services grow large over time            | Small, focused handlers                          |
| Harder to scale read/write independently | Read and write paths can be optimized separately |
| Simpler for small apps                   | Better suited for complex enterprise systems     |

### Interview Answer (2–3 minutes)

> "CQRS stands for Command Query Responsibility Segregation. It separates write operations (Commands) from read operations (Queries). Commands modify data and are handled by command handlers, while queries retrieve data and are handled by query handlers. In ASP.NET Core, CQRS is commonly implemented with MediatR, where controllers send commands or queries through `IMediator`, and the appropriate handler executes the business logic. This improves maintainability, testability, and scalability by keeping each handler focused on a single responsibility. For simple CRUD applications, CQRS can be overkill, but for enterprise applications with complex business logic, it's a widely adopted architectural pattern."


The **Mediator pattern** is the core idea behind **MediatR**. It acts as a **middleman** between your Controller and your business logic (handlers). The controller doesn't know who handles the request—it simply sends it to the mediator.

Let's walk through the complete flow.

---

# Without MediatR

```text
HTTP Request
      |
      V
EmployeeController
      |
      V
EmployeeService
      |
      V
EmployeeRepository
      |
      V
Database
```

Example:

```csharp
[HttpPost]
public async Task<IActionResult> Create(EmployeeDto dto)
{
    await _employeeService.CreateEmployee(dto);

    return Ok();
}
```

Here the controller knows about `EmployeeService`.

If tomorrow you rename or replace the service, the controller changes too.

---

# With MediatR

```text
HTTP Request
      |
      V
EmployeeController
      |
      V
IMediator.Send(Command)
      |
      V
MediatR Library
      |
      V
Find Correct Handler
      |
      V
Command Handler
      |
      V
Database
```

The controller only knows one thing:

> "I have a request. Mediator, please send it to whoever handles it."

---

# Step 1: Client Sends Request

```http
POST /api/employees
```

Body

```json
{
   "name":"Pankaj",
   "email":"pankaj@gmail.com"
}
```

---

# Step 2: Controller Receives Request

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmployeeController : ControllerBase
{
    private readonly IMediator _mediator;

    public EmployeeController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<IActionResult> Create(CreateEmployeeCommand command)
    {
        var id = await _mediator.Send(command);

        return Ok(id);
    }
}
```

Notice:

The controller **does not know**:

* EF Core
* Repository
* Service
* SQL

It only knows `IMediator`.

---

# Step 3: What is CreateEmployeeCommand?

```csharp
public record CreateEmployeeCommand(
    string Name,
    string Email
) : IRequest<int>;
```

This is just a **message**.

Think of it like a parcel.

```text
Parcel

Name = Pankaj
Email = pankaj@gmail.com
```

Nothing happens here.

It only carries data.

---

# Step 4: What happens inside `Send()`?

```csharp
await _mediator.Send(command);
```

This is where MediatR starts working.

Internally it does something conceptually like:

```text
Receive Request

↓

What is request type?

↓

CreateEmployeeCommand

↓

Who handles CreateEmployeeCommand?

↓

CreateEmployeeCommandHandler

↓

Call Handle()
```

You don't write this lookup yourself—the MediatR library does it using **Dependency Injection (DI)**.

---

# Step 5: How does MediatR know the handler?

Your handler is:

```csharp
public class CreateEmployeeCommandHandler
    : IRequestHandler<CreateEmployeeCommand, int>
{
}
```

Notice:

```csharp
IRequestHandler<CreateEmployeeCommand,int>
```

This tells MediatR:

> "Whenever someone sends a `CreateEmployeeCommand`, call this handler."

It's essentially a mapping:

```text
CreateEmployeeCommand
        ↓
CreateEmployeeCommandHandler
```

---

# Step 6: Handler Executes

```csharp
public class CreateEmployeeCommandHandler
    : IRequestHandler<CreateEmployeeCommand,int>
{
    private readonly AppDbContext _context;

    public CreateEmployeeCommandHandler(AppDbContext context)
    {
        _context = context;
    }

    public async Task<int> Handle(
        CreateEmployeeCommand request,
        CancellationToken cancellationToken)
    {
        var employee = new Employee
        {
            Name = request.Name,
            Email = request.Email
        };

        _context.Employees.Add(employee);

        await _context.SaveChangesAsync();

        return employee.Id;
    }
}
```

The `request` parameter is the same object the controller sent.

```text
Controller

CreateEmployeeCommand

↓

Mediator

↓

Handler

↓

Handle(request)
```

---

# Step 7: Response Returns

The handler returns:

```csharp
return employee.Id;
```

The value flows back:

```text
Handler

↓

Mediator

↓

Controller

↓

HTTP Response
```

The client receives:

```json
25
```

---

# How is the handler found?

In `Program.cs`:

```csharp
builder.Services.AddMediatR(cfg =>
{
    cfg.RegisterServicesFromAssembly(
        typeof(CreateEmployeeCommand).Assembly);
});
```

During application startup:

```text
Application Starts

↓

Scan Assembly

↓

Find all IRequestHandler<,>

↓

Register them in DI Container
```

It discovers handlers like:

```text
CreateEmployeeCommandHandler

UpdateEmployeeCommandHandler

DeleteEmployeeCommandHandler

GetEmployeeByIdQueryHandler

GetEmployeesQueryHandler
```

These are all registered automatically.

---

# What happens inside `Send()` (Simplified)

Think of MediatR doing something like this internally:

```csharp
public async Task<TResponse> Send<TResponse>(IRequest<TResponse> request)
{
    var handler = GetHandler(request.GetType());

    return await handler.Handle(request);
}
```

In reality, the implementation is more sophisticated (it uses generic wrappers, caching, and DI), but conceptually this is what happens.

---

# Query Example

Controller

```csharp
[HttpGet("{id}")]
public async Task<IActionResult> Get(int id)
{
    var employee = await _mediator.Send(
        new GetEmployeeByIdQuery(id));

    return Ok(employee);
}
```

Query

```csharp
public record GetEmployeeByIdQuery(int Id)
    : IRequest<EmployeeDto>;
```

Handler

```csharp
public class GetEmployeeByIdQueryHandler
    : IRequestHandler<GetEmployeeByIdQuery, EmployeeDto>
{
    public async Task<EmployeeDto> Handle(
        GetEmployeeByIdQuery request,
        CancellationToken cancellationToken)
    {
        // Read from database
    }
}
```

Flow:

```text
Controller

↓

Mediator.Send(Query)

↓

Query Handler

↓

Database

↓

Employee DTO

↓

Mediator

↓

Controller
```

---

# How Dependency Injection Helps

The handler can have its own dependencies:

```csharp
public class CreateEmployeeCommandHandler
{
    private readonly AppDbContext _db;
    private readonly ILogger<CreateEmployeeCommandHandler> _logger;
    private readonly IEmailService _emailService;

    public CreateEmployeeCommandHandler(
        AppDbContext db,
        ILogger<CreateEmployeeCommandHandler> logger,
        IEmailService emailService)
    {
        _db = db;
        _logger = logger;
        _emailService = emailService;
    }
}
```

MediatR asks the DI container to create the handler, and DI automatically injects these dependencies. The controller remains completely unaware of them.

---

# Pipeline Behaviors (One of MediatR's Biggest Advantages)

Before the handler runs, you can execute common logic.

```text
Controller

↓

Mediator

↓

Validation

↓

Logging

↓

Authorization

↓

Performance Tracking

↓

Caching (for queries)

↓

Handler
```

Instead of writing logging in every handler:

```csharp
_logger.LogInformation("Creating Employee...");
```

you can create one pipeline behavior that logs every request automatically.

---

# Real-World Analogy

Imagine you're in a restaurant:

```text
Customer
    |
    V
Waiter (Mediator)
    |
    +--> Chef (Cook food)
    |
    +--> Cashier (Process payment)
    |
    +--> Barista (Make coffee)
```

You never go directly to the chef or cashier. You tell the **waiter** what you need, and the waiter knows exactly who should handle it.

Similarly:

* **Controller** = Customer
* **IMediator** = Waiter
* **Command/Query** = Order slip
* **Handler** = Chef
* **Database** = Kitchen

The controller doesn't need to know *who* performs the work—it simply hands the request to the mediator.

### Interview Explanation

If asked, **"How does MediatR work internally?"**, you can answer:

> "MediatR implements the Mediator pattern. Controllers send commands or queries using `IMediator.Send()`. At application startup, MediatR scans the registered assemblies and registers all `IRequestHandler<TRequest, TResponse>` implementations in the dependency injection container. When `Send()` is called, MediatR resolves the appropriate handler from DI based on the request type, executes any configured pipeline behaviors (such as validation or logging), invokes the handler's `Handle()` method, and returns the handler's response back to the controller. This keeps controllers decoupled from business logic and makes the application easier to maintain and test."


Good question. **No, CQRS does *not* mean you cannot reuse the same model.** It depends on what you mean by "model."

There are several types of models in a .NET application:

1. **Domain Entity** (EF Core entity)
2. **Command model**
3. **Query model (DTO)**
4. **View model/API model**

Let's look at each one.

---

## 1. Domain Entity (Can be reused)

Suppose you have an `Employee` entity:

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public decimal Salary { get; set; }
}
```

This same entity can be used by multiple command handlers.

### CreateEmployeeCommandHandler

```csharp
var employee = new Employee
{
    Name = request.Name,
    Email = request.Email,
    Salary = request.Salary
};

_context.Employees.Add(employee);
```

### UpdateEmployeeCommandHandler

```csharp
var employee = await _context.Employees.FindAsync(request.Id);

employee.Name = request.Name;
employee.Email = request.Email;

await _context.SaveChangesAsync();
```

Both handlers use the **same `Employee` entity**.

---

## 2. Commands (Usually not reused)

Commands represent a **specific business action**.

For example:

```csharp
public record CreateEmployeeCommand(
    string Name,
    string Email,
    decimal Salary
);
```

Another command:

```csharp
public record UpdateEmployeeCommand(
    int Id,
    string Name,
    string Email
);
```

Even though both contain `Name` and `Email`, they represent different operations.

**Why separate them?**

* Create doesn't need `Id`.
* Update requires `Id`.
* Validation rules may differ.

For example:

### Create

```text
Name ✔
Email ✔
Salary ✔
```

### Update

```text
Id ✔
Name ✔
Email ✔
Salary optional
```

So it's better to keep them separate.

---

## 3. Query Models (DTOs)

Queries often return only the data the client needs.

Database entity:

```csharp
Employee
{
    Id
    Name
    Email
    Salary
    PasswordHash
    CreatedDate
}
```

API response:

```csharp
EmployeeDto
{
    Id
    Name
    Email
}
```

You don't expose `PasswordHash` or other internal fields.

---

## Can one command be used by multiple handlers?

**No.**

Each request type should have **one handler**.

For example:

```csharp
public record CreateEmployeeCommand(...)
```

Handled by:

```csharp
CreateEmployeeCommandHandler
```

Not:

```text
CreateEmployeeCommand
       |
       +--> Handler A ❌
       |
       +--> Handler B ❌
```

Instead:

```text
CreateEmployeeCommand
        |
        V
CreateEmployeeCommandHandler ✅
```

This one-to-one relationship is how `IMediator.Send()` works.

---

## Can multiple commands use the same Entity?

**Yes.** This is very common.

```text
                Employee Entity
                     ▲
                     │
      ┌──────────────┼──────────────┐
      │              │              │
      │              │              │
CreateEmployee   UpdateEmployee   DeleteEmployee
   Command          Command          Command
      │              │              │
      ▼              ▼              ▼
CreateHandler   UpdateHandler   DeleteHandler
```

All three handlers work with the same `Employee` entity.

---

## Can multiple commands share common code?

Yes. If multiple handlers need the same logic, move that logic into a service.

Example:

```csharp
public class EmployeeValidator
{
    public bool IsEmailUnique(string email)
    {
        // validation logic
    }
}
```

Inject it into multiple handlers:

```csharp
public class CreateEmployeeCommandHandler
{
    private readonly EmployeeValidator _validator;
}

public class UpdateEmployeeCommandHandler
{
    private readonly EmployeeValidator _validator;
}
```

This avoids duplicating business logic while keeping each handler focused on a single use case.

---

## Summary

| Component                        | Can it be reused? | Example                                                                |
| -------------------------------- | ----------------- | ---------------------------------------------------------------------- |
| **Entity (Employee)**            | ✅ Yes             | Used by Create, Update, Delete handlers                                |
| **Command**                      | ❌ Usually no      | `CreateEmployeeCommand` is only for creating                           |
| **Query**                        | ❌ Usually no      | `GetEmployeeByIdQuery` is only for fetching by ID                      |
| **DTO**                          | ✅ Sometimes       | Same response DTO can be used by multiple queries if the shape matches |
| **Business Services/Validators** | ✅ Yes             | Shared across multiple handlers                                        |

### Interview Tip

If an interviewer asks, **"Can we reuse the same model in CQRS?"**, a strong answer is:

> "Yes, the domain entity can absolutely be reused across multiple command and query handlers. What CQRS separates are the request models—commands and queries—because each represents a specific use case with its own validation and intent. The `Employee` entity may be shared by Create, Update, and Delete handlers, while each operation has its own command object such as `CreateEmployeeCommand` or `UpdateEmployeeCommand`."

Since you have **5.6+ years of .NET Full Stack experience**, interviewers usually don't stop at "What is CQRS?". They often ask scenario-based and implementation questions.

Here are some common questions with concise answers.

---

# 1. What is CQRS?

**Answer:**

CQRS (Command Query Responsibility Segregation) is a pattern that separates read operations (Queries) from write operations (Commands). Commands modify data, while queries only retrieve data. This separation improves maintainability, scalability, and testability.

---

# 2. What is the difference between Command and Query?

| Command                  | Query                       |
| ------------------------ | --------------------------- |
| Changes data             | Reads data                  |
| Can call `SaveChanges()` | Never calls `SaveChanges()` |
| Returns success/ID       | Returns DTO/ViewModel       |
| POST, PUT, DELETE        | GET                         |

---

# 3. Why use CQRS instead of Repository + Service?

**Answer:**

In traditional architecture, services become very large with many methods. CQRS keeps each operation in its own handler, making code easier to maintain, test, and extend.

---

# 4. What is MediatR?

**Answer:**

MediatR is a .NET library that implements the Mediator pattern. Controllers send commands or queries to `IMediator`, which locates the appropriate handler and executes it. This decouples controllers from business logic.

---

# 5. How does `Mediator.Send()` work internally?

**Answer:**

1. Controller calls `Send()`.
2. MediatR identifies the request type.
3. DI resolves the matching `IRequestHandler`.
4. Pipeline behaviors execute (if configured).
5. Handler's `Handle()` method runs.
6. Response is returned to the controller.

---

# 6. What is `IRequest<T>`?

```csharp
public record CreateEmployeeCommand : IRequest<int>;
```

**Answer:**

It represents a request expecting a response of type `int`. MediatR uses this type to determine which handler to invoke.

---

# 7. What is `IRequestHandler<TRequest, TResponse>`?

```csharp
public class CreateEmployeeCommandHandler
    : IRequestHandler<CreateEmployeeCommand, int>
{
}
```

**Answer:**

It defines the class responsible for handling a specific request type.

---

# 8. Can one Command have multiple Handlers?

**Answer:**

No.

Each request should have exactly one handler when using `Send()`.

```
CreateEmployeeCommand

        ↓

CreateEmployeeCommandHandler
```

Having multiple handlers would make it ambiguous which one should execute.

---

# 9. Can multiple Commands use the same Entity?

**Answer:**

Yes.

```
Employee

↑

Create
Update
Delete
```

The entity is shared; the command models are separate.

---

# 10. Can multiple Queries use the same DTO?

Yes.

Example:

```
GetEmployeeById

↓

EmployeeDto

GetEmployeeByEmail

↓

EmployeeDto
```

If the response shape is identical, reusing the DTO is perfectly fine.

---

# 11. Why shouldn't Queries return Entities?

Example:

```
Employee
{
    Id
    Name
    Email
    Salary
    PasswordHash
}
```

API only needs

```
EmployeeDto
{
    Id
    Name
}
```

Returning entities can expose internal fields and tightly couple the API to the database model.

---

# 12. Why use DTOs?

* Hide sensitive fields
* Reduce payload size
* Avoid exposing EF entities
* Version APIs more easily

---

# 13. Can Commands return data?

Yes, but only what's necessary.

Example:

```
CreateEmployeeCommand

↓

returns EmployeeId
```

Avoid returning the full entity after a command.

---

# 14. Can Queries modify data?

No.

Queries should be side-effect free.

Calling `SaveChanges()` inside a query handler violates CQRS principles.

---

# 15. Why is CQRS more testable?

Each handler has a single responsibility.

Example:

```
Test

↓

CreateEmployeeCommandHandler

↓

Mock DbContext
```

No need to instantiate large service classes.

---

# 16. Where should validation happen?

Common approaches:

* FluentValidation
* MediatR Pipeline Behaviors
* Handler (for business validation)

Avoid putting business validation in controllers.

---

# 17. What are Pipeline Behaviors?

They are middleware for MediatR.

```
Controller

↓

Validation

↓

Logging

↓

Authorization

↓

Performance

↓

Handler
```

Cross-cutting concerns are implemented once instead of repeating code in every handler.

---

# 18. Difference between Middleware and Pipeline Behavior?

Middleware

```
HTTP Request

↓

Authentication

↓

Routing

↓

Controller
```

Pipeline Behavior

```
Controller

↓

Mediator

↓

Validation

↓

Handler
```

Middleware works at the HTTP request level, while pipeline behaviors work around MediatR requests.

---

# 19. What is the advantage of CQRS?

* Better separation of concerns
* Easier maintenance
* Easier testing
* Independent read/write optimization
* Cleaner business logic
* Fits well with Clean Architecture

---

# 20. What are the disadvantages?

* More classes
* More files
* More complexity
* Overkill for simple CRUD applications

---

# 21. When should CQRS NOT be used?

Don't use it for:

* Small CRUD apps
* Admin panels
* Simple internal tools
* Applications with very little business logic

---

# 22. Does CQRS require two databases?

No.

You can use:

```
Commands

↓

SQL Server

Queries

↓

SQL Server
```

The same database is perfectly valid.

For very large systems, separate read and write databases may be introduced.

---

# 23. What is Eventual Consistency?

If writes go to one database and reads come from another (read replica), there may be a short delay before the latest changes appear in query results. This temporary lag is called eventual consistency.

---

# 24. How is CQRS used with Clean Architecture?

```
API

↓

Application
    Commands
    Queries
    Handlers

↓

Domain

↓

Infrastructure
```

Commands and Queries belong in the **Application** layer, while Infrastructure contains EF Core and database implementations.

---

# 25. Difference between CQRS and Mediator?

Many candidates confuse these.

| CQRS                       | MediatR                              |
| -------------------------- | ------------------------------------ |
| Architectural pattern      | .NET library                         |
| Separates reads and writes | Implements the Mediator pattern      |
| Can exist without MediatR  | Helps implement CQRS but is optional |

**Example:** You can implement CQRS without MediatR by calling handlers directly, but MediatR simplifies request routing and decoupling.

---

# 26. Can you use CQRS without MediatR?

Yes.

You could manually instantiate or inject handlers and invoke them directly:

```csharp
var handler = new CreateEmployeeCommandHandler(context);
await handler.Handle(command, CancellationToken.None);
```

MediatR simply automates the routing and keeps the controller unaware of the concrete handler.

---

# 27. Scenario-Based Question

**Interviewer:** *You have a `CreateOrderCommand`. After saving the order, you need to send an email and write an audit log. How would you implement it?*

**Good answer:**

* Keep the command handler responsible for creating the order.
* Use injected services (e.g., `IEmailService`, `IAuditService`) if these actions are part of the same transaction/business process.
* If these are independent side effects, publish a notification/event after the order is created, and let separate notification handlers send the email and write the audit log.

This keeps the command handler focused and avoids mixing unrelated responsibilities.

---

## Common Mistakes to Avoid in Interviews

❌ "CQRS means two databases."

✅ No. Two databases are optional.

❌ "Queries can update data."

✅ Queries should not modify state.

❌ "MediatR is CQRS."

✅ MediatR is a library that helps implement CQRS.

❌ "Each entity should have only one command."

✅ An entity can have many commands (`Create`, `Update`, `Delete`, `Activate`, etc.).

❌ "CQRS is required for every project."

✅ It's most beneficial for applications with complex business logic; for simple CRUD applications, it often adds unnecessary complexity.

These are the kinds of questions commonly asked for **5–8 years .NET Developer** interviews at companies such as Deloitte, Accenture, Cognizant, Capgemini, EPAM, LTIMindtree, BNY, and product-based organizations.
