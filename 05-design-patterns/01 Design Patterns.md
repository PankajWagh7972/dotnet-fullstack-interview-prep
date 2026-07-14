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
