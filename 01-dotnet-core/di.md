# 📘 01-dotnet-core/di.md

## 🔥 Advanced DI (Deep Dive with Real-World Examples)

---

# 🧠 1. Real Problem DI Solves (Production Scenario)

## ❓ Problem

You are building a **Notification System**:

* Email
* SMS
* Push Notifications

Without DI:

```csharp
public class NotificationService
{
    public void Send()
    {
        var email = new EmailService();
        email.Send();
    }
}
```

❌ Problems:

* Hardcoded dependency
* Cannot switch to SMS
* Impossible to test

---

## ✅ With DI (Production Approach)

```csharp
public interface INotificationService
{
    Task SendAsync(string message);
}
```

### Multiple Implementations

```csharp
public class EmailService : INotificationService
{
    public Task SendAsync(string message)
    {
        Console.WriteLine($"Email: {message}");
        return Task.CompletedTask;
    }
}

public class SmsService : INotificationService
{
    public Task SendAsync(string message)
    {
        Console.WriteLine($"SMS: {message}");
        return Task.CompletedTask;
    }
}
```

---

### Register in DI

```csharp
builder.Services.AddScoped<EmailService>();
builder.Services.AddScoped<SmsService>();
```

---

### Dynamic Resolution (Real Interview Question)

```csharp
public class NotificationFactory
{
    private readonly IServiceProvider _provider;

    public NotificationFactory(IServiceProvider provider)
    {
        _provider = provider;
    }

    public INotificationService GetService(string type)
    {
        return type switch
        {
            "email" => _provider.GetRequiredService<EmailService>(),
            "sms" => _provider.GetRequiredService<SmsService>(),
            _ => throw new Exception("Invalid type")
        };
    }
}
```

---

### Use in Controller

```csharp
public class NotificationController : ControllerBase
{
    private readonly NotificationFactory _factory;

    public NotificationController(NotificationFactory factory)
    {
        _factory = factory;
    }

    [HttpPost]
    public async Task<IActionResult> Send(string type)
    {
        var service = _factory.GetService(type);
        await service.SendAsync("Hello");

        return Ok();
    }
}
```

---

## 🎯 Interview Angle

👉 They may ask:

> Why not inject `IEnumerable<INotificationService>`?

Better answer:

* Use factory when **runtime decision is needed**
* Use IEnumerable when **processing all implementations**

---

# ⚙️ 2. Service Lifetime Bug (REAL JPM-Level Question)

## ❓ Scenario

```csharp
public class MySingletonService
{
    private readonly AppDbContext _context;

    public MySingletonService(AppDbContext context)
    {
        _context = context;
    }
}
```

❌ Problem:

* `DbContext` is **Scoped**
* Singleton holds it forever

---

## 💥 What Happens?

* First request → OK
* Second request → uses same DbContext
* Data corruption / threading issues

---

## ✅ Correct Fix

```csharp
public class MySingletonService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public MySingletonService(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    public void Execute()
    {
        using var scope = _scopeFactory.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();

        // safe usage
    }
}
```

---

## 🎯 Interview Insight

👉 This question checks:

* Lifetime understanding
* Real-world debugging ability

---

# 🔄 3. Scoped Services in Background Jobs

## ❓ Problem

BackgroundService runs outside HTTP pipeline.

```csharp
public class Worker : BackgroundService
{
    private readonly AppDbContext _context; // ❌ WRONG
}
```

---

## ✅ Correct Implementation

```csharp
public class Worker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public Worker(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = _scopeFactory.CreateScope();

            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();

            // Work with DB safely
            await Task.Delay(5000);
        }
    }
}
```

---

## 🎯 Interview Trap

👉 If you inject DbContext directly → ❌ reject-level mistake

---

# 🧩 4. Multiple Implementations (Enterprise Use Case)

## ❓ Scenario

Payment system:

* Credit Card
* UPI
* Net Banking

---

## ✅ Implementation

```csharp
public interface IPaymentService
{
    string Pay();
}
```

```csharp
public class UpiPayment : IPaymentService
{
    public string Pay() => "Paid via UPI";
}

public class CardPayment : IPaymentService
{
    public string Pay() => "Paid via Card";
}
```

---

## Register

```csharp
services.AddScoped<IPaymentService, UpiPayment>();
services.AddScoped<IPaymentService, CardPayment>();
```

---

## Inject All

```csharp
public class PaymentProcessor
{
    private readonly IEnumerable<IPaymentService> _services;

    public PaymentProcessor(IEnumerable<IPaymentService> services)
    {
        _services = services;
    }

    public void Process()
    {
        foreach (var service in _services)
        {
            Console.WriteLine(service.Pay());
        }
    }
}
```

---

## 🎯 Interview Insight

👉 This shows:

* Strategy pattern + DI
* Extensibility without modifying code

---

# 🧪 5. Unit Testing with DI (REAL WORLD)

## ❓ Scenario

Controller depends on service

```csharp
public class OrderController
{
    private readonly IOrderService _service;

    public OrderController(IOrderService service)
    {
        _service = service;
    }
}
```

---

## ✅ Test

```csharp
[Fact]
public void Test_Order()
{
    var mock = new Mock<IOrderService>();

    mock.Setup(x => x.PlaceOrder()).Returns(true);

    var controller = new OrderController(mock.Object);

    var result = controller.PlaceOrder();

    Assert.True(result);
}
```

---

## 🎯 Interview Insight

👉 They want:

* Mocking understanding
* Isolation testing

---

# ⚡ 6. Performance & Internal Mechanics

## ❓ How DI is fast?

👉 .NET does NOT use reflection every time.

Instead:

* Builds **expression tree**
* Compiles into delegate
* Reuses it

---

## ❓ What is Call Site Factory?

👉 Internal component that:

* Builds dependency graph
* Caches resolution plan

---

## 🎯 Interview Level Answer

👉 Say:

> “.NET DI container compiles service resolution into delegates, avoiding reflection overhead during runtime.”

---

# 🧠 7. Design-Level Question

## ❓ When NOT to use DI?

👉 Avoid when:

* Simple utility classes
* Performance-critical loops
* Static helpers

---

## ❓ Overuse Problem

```csharp
public class A(B b, C c, D d, E e, F f) {}
```

👉 This is:

* Hard to maintain
* Indicates bad design

---

## ✅ Fix

* Break into smaller services
* Use Facade pattern

---
