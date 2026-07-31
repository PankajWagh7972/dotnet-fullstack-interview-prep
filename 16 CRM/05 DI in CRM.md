This is a good interview question because **Dynamics 365 plug-ins do not support built-in Dependency Injection (DI)** like ASP.NET Core.

The interviewer wants to know whether you understand the limitations and how to implement DI patterns.

---

# Short Interview Answer

> Dynamics 365 plug-ins do not have a built-in dependency injection container because the platform creates the plug-in instance. However, we can implement dependency injection manually using constructor injection, a simple factory, or a lightweight IoC container. We usually inject services such as repositories, business services, logging abstractions, or HTTP clients to improve testability and maintainability.

---

# Why DI Doesn't Work Like ASP.NET Core

In ASP.NET Core:

```csharp
builder.Services.AddScoped<IAccountRepository, AccountRepository>();
```

The framework creates the object.

In a plugin:

```text
Dataverse

↓

Plugin Runtime

↓

new AccountPlugin()

↓

Execute()
```

**Dataverse creates the plugin instance**, so you don't control object creation and can't register services in a built-in DI container.

---

# Option 1 (Recommended): Constructor Injection (Manual)

### Service Interface

```csharp
public interface IEmailService
{
    void Send(string email);
}
```

---

### Implementation

```csharp
public class EmailService : IEmailService
{
    public void Send(string email)
    {
        Console.WriteLine($"Sending email to {email}");
    }
}
```

---

### Plugin

```csharp
using Microsoft.Xrm.Sdk;

public class AccountPlugin : IPlugin
{
    private readonly IEmailService _emailService;

    // Default constructor used by Dataverse
    public AccountPlugin()
        : this(new EmailService())
    {
    }

    // Constructor used in unit tests
    public AccountPlugin(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public void Execute(IServiceProvider serviceProvider)
    {
        _emailService.Send("test@company.com");
    }
}
```

### Why this is useful

* Easy to mock during unit testing.
* Separates business logic from plugin logic.
* No dependency on a DI framework.

---

# Option 2: Factory Pattern

Instead of creating dependencies directly:

```text
Plugin

↓

Factory

↓

Business Service

↓

Repository
```

Factory

```csharp
public static class ServiceFactory
{
    public static IEmailService CreateEmailService()
    {
        return new EmailService();
    }
}
```

Plugin

```csharp
public class AccountPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var emailService = ServiceFactory.CreateEmailService();

        emailService.Send("abc@company.com");
    }
}
```

This centralizes object creation.

---

# Option 3: Repository + Service Pattern (Enterprise Projects)

```text
Plugin

↓

AccountService

↓

AccountRepository

↓

Dataverse
```

Repository

```csharp
public class AccountRepository
{
    private readonly IOrganizationService _service;

    public AccountRepository(IOrganizationService service)
    {
        _service = service;
    }

    public Entity Get(Guid id)
    {
        return _service.Retrieve(
            "account",
            id,
            new Microsoft.Xrm.Sdk.Query.ColumnSet(true));
    }
}
```

Business Service

```csharp
public class AccountService
{
    private readonly AccountRepository _repository;

    public AccountService(AccountRepository repository)
    {
        _repository = repository;
    }

    public Entity GetAccount(Guid id)
    {
        return _repository.Get(id);
    }
}
```

Plugin

```csharp
public class AccountPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context = (IPluginExecutionContext)
            serviceProvider.GetService(typeof(IPluginExecutionContext));

        var factory = (IOrganizationServiceFactory)
            serviceProvider.GetService(typeof(IOrganizationServiceFactory));

        var service = factory.CreateOrganizationService(context.UserId);

        var repository = new AccountRepository(service);
        var accountService = new AccountService(repository);

        var account = accountService.GetAccount(Guid.NewGuid());
    }
}
```

This keeps the plugin thin and moves business logic into reusable services.

---

# Option 4: Using a DI Container (Possible but Less Common)

You can use containers like **Autofac** or **Microsoft.Extensions.DependencyInjection**:

```csharp
var services = new ServiceCollection();

services.AddSingleton<IEmailService, EmailService>();

var provider = services.BuildServiceProvider();

var emailService = provider.GetRequiredService<IEmailService>();
```

However:

* The container is created inside the plugin.
* There is no application startup to register services once.
* Building a container on every execution adds overhead.

Because of that, this approach is generally **not recommended** unless you have a strong architectural reason.

---

# Real Project Example

Suppose a plugin creates an order.

```text
Order Plugin

↓

OrderService

↓

PricingService

↓

TaxService

↓

OrderRepository

↓

Dataverse
```

The plugin only coordinates execution, while business logic is encapsulated in services, making the code easier to test and maintain.

---

# Best Practices

* Keep plugins thin; move business logic into service classes.
* Use interfaces to make services mockable for unit tests.
* Pass `IOrganizationService`, `ITracingService`, and other platform services into your business layer rather than accessing `IServiceProvider` everywhere.
* Avoid creating heavyweight DI containers inside every plugin execution unless there's a compelling reason.

---

# Interview Answer (2 Minutes)

> Dynamics 365 plug-ins don't support built-in dependency injection because the Dataverse runtime creates the plugin instance. The most common approach is to use **manual constructor injection** with a default constructor for production and an overloaded constructor for unit testing. In enterprise projects, I keep the plugin thin and delegate work to service and repository classes, passing dependencies such as `IOrganizationService` into those classes. While it's technically possible to use a DI container like `Microsoft.Extensions.DependencyInjection`, it's usually not recommended because the container would need to be created during plugin execution, adding unnecessary overhead. The constructor injection plus service/repository pattern is the approach I typically prefer.
