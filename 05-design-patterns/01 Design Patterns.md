There are **three main types of Design Patterns** in software development (Gang of Four - GoF Design Patterns).

---

# 1. Creational Design Patterns

**Purpose:** Deal with **object creation**. They make object creation flexible and reusable.

### Types

| Pattern          | Use Case                                        |
| ---------------- | ----------------------------------------------- |
| Singleton        | Only one instance (Logger, Configuration)       |
| Factory Method   | Creates objects without exposing creation logic |
| Abstract Factory | Creates families of related objects             |
| Builder          | Builds complex objects step by step             |
| Prototype        | Creates objects by cloning existing objects     |

### Real-Time Example

* `ILogger`
* `DbContextFactory`
* `IHttpClientFactory`
* `ConfigurationManager`

---

# 2. Structural Design Patterns

**Purpose:** Deal with **how classes and objects are organized** to form larger structures.

### Types

| Pattern   | Use Case                                            |
| --------- | --------------------------------------------------- |
| Adapter   | Makes incompatible interfaces work together         |
| Bridge    | Separates abstraction from implementation           |
| Composite | Tree-like structures (Folders, Menus)               |
| Decorator | Adds functionality without modifying the class      |
| Facade    | Provides a simplified interface to a complex system |
| Flyweight | Shares objects to reduce memory usage               |
| Proxy     | Controls access to another object                   |

### Real-Time Example

* Logging decorators
* Repository wrappers
* API gateway (Facade)
* Caching proxy

---

# 3. Behavioral Design Patterns

**Purpose:** Deal with **communication and responsibilities between objects**.

### Types

| Pattern                 | Use Case                                       |
| ----------------------- | ---------------------------------------------- |
| Observer                | Event notifications (EventHandler, SignalR)    |
| Strategy                | Switch algorithms at runtime (Payment methods) |
| Command                 | Encapsulates requests (Undo/Redo)              |
| State                   | Object behavior changes based on state         |
| Chain of Responsibility | Middleware pipeline                            |
| Mediator                | Central communication (MediatR)                |
| Iterator                | Traverse collections (`foreach`)               |
| Memento                 | Undo functionality                             |
| Template Method         | Common algorithm with customizable steps       |
| Visitor                 | Add operations without changing classes        |
| Interpreter             | Evaluate expressions                           |

### Real-Time Example

* ASP.NET Core Middleware → Chain of Responsibility
* MediatR → Mediator
* `foreach` → Iterator
* Payment Gateway → Strategy

---

# Summary

| Category       | Purpose              | Examples                     |
| -------------- | -------------------- | ---------------------------- |
| **Creational** | Object creation      | Singleton, Factory, Builder  |
| **Structural** | Object composition   | Adapter, Decorator, Facade   |
| **Behavioral** | Object communication | Strategy, Observer, Mediator |

---

## Interview Answer (1 Minute)

> Design patterns are reusable solutions to common software design problems. They are categorized into three types:
>
> 1. **Creational patterns**, which focus on object creation (e.g., Singleton, Factory, Builder).
> 2. **Structural patterns**, which organize classes and objects into larger structures (e.g., Adapter, Decorator, Facade).
> 3. **Behavioral patterns**, which define how objects communicate and interact (e.g., Strategy, Observer, Chain of Responsibility).
>
> In ASP.NET Core, examples include **Singleton** for shared services, **Factory** for object creation, **Decorator** for adding functionality, **Chain of Responsibility** in the middleware pipeline, and **Strategy** for selecting algorithms like payment processing.


# Static Class vs Singleton Instance

| Feature              | Static Class                  | Singleton                          |
| -------------------- | ----------------------------- | ---------------------------------- |
| Object Creation      | ❌ Cannot create objects       | ✅ One object is created            |
| Instance             | No instance                   | Single instance                    |
| Inheritance          | ❌ Cannot inherit              | ✅ Can inherit/implement interfaces |
| Dependency Injection | ❌ Not supported               | ✅ Supported                        |
| Interface Support    | ❌ Cannot implement interfaces | ✅ Can implement interfaces         |
| Constructor          | Only static constructor       | Private constructor                |
| Testing/Mocking      | ❌ Difficult                   | ✅ Easy (using interfaces)          |
| State                | Shared globally               | Shared through one instance        |
| Best For             | Utility/helper methods        | Shared services/resources          |

---

# Static Class

A **static class** contains only static members and **cannot be instantiated**.

### Example

```csharp
public static class MathHelper
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}

// Usage
int result = MathHelper.Add(10, 20);
```

### Use Cases

* Helper methods
* Utility classes
* Extension methods
* Mathematical calculations

Examples:

* `Math`
* `Convert`
* `File`
* `Path`

---

# Singleton

A **Singleton** ensures that **only one instance** of a class exists throughout the application's lifetime.

### Example

```csharp
public sealed class Logger
{
    private static readonly Logger _instance = new Logger();

    private Logger() { }

    public static Logger Instance => _instance;

    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}

// Usage
Logger.Instance.Log("Application Started");
```

### Use Cases

* Logger
* Cache service
* Configuration service
* Database connection manager (where appropriate)

---

# Key Difference

### Static Class

* No object is created.
* Members are accessed directly using the class name.
* Cannot implement interfaces or participate in dependency injection.

### Singleton

* Exactly one object is created.
* Accessed through a single instance (e.g., `Instance` property or DI container).
* Can implement interfaces, making it testable and suitable for dependency injection.

---

# ASP.NET Core Example

```csharp
builder.Services.AddSingleton<IEmailService, EmailService>();
```

`EmailService` is a **Singleton service**:

* One instance is created.
* Reused across the application.
* Can be injected into controllers and other services.

A static class **cannot** be registered with the DI container.

---

# When to Use

### Use a Static Class when:

* You only need stateless utility methods.
* No dependency injection is required.

Examples:

* `Math`
* `StringHelper`
* `DateTimeHelper`

### Use a Singleton when:

* You need one shared object with state or dependencies.
* The service should participate in dependency injection.
* You want to mock or replace it during testing.

Examples:

* Logging service
* Configuration service
* In-memory cache service

---

# Interview Answer (30 seconds)

> A static class cannot be instantiated and contains only static members. It is mainly used for stateless utility methods and cannot implement interfaces or be injected through dependency injection. A Singleton, on the other hand, is a design pattern that ensures only one instance of a class exists. It can implement interfaces, maintain state, work with dependency injection, and is easier to test. In modern ASP.NET Core applications, Singleton services are generally preferred over static classes for shared application services.

