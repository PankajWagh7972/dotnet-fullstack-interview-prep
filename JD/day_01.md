Perfect. Since this is a **5+ years backend engineering interview**, I'll teach you as if I'm preparing you for the actual interview, not just explaining concepts.

We'll cover **Day 1** in this order:

1. OOP (Interview Depth)
2. SOLID Principles
3. Dependency Injection
4. Interfaces vs Abstract Classes
5. Delegates & Events
6. Generics
7. Extension Methods
8. LINQ
9. Exception Handling
10. async/await (Most Important)
11. Task Parallel Library
12. Memory Management & Garbage Collection
13. Value Types vs Reference Types
14. String vs StringBuilder
15. Records vs Classes
16. Common Interview Questions

---

# Part 1 — Object-Oriented Programming (OOP)

This is almost always the first topic in a .NET interview.

## What is OOP?

OOP (Object-Oriented Programming) is a programming paradigm where software is designed using **objects** that contain **data (properties/fields)** and **behavior (methods)**.

Instead of writing everything in one place, we model real-world entities.

Example:

```
Car
--------
Color
Speed

Start()
Stop()
Accelerate()
```

In C#

```csharp
public class Car
{
    public string Color { get; set; }

    public int Speed { get; set; }

    public void Start()
    {
    }

    public void Stop()
    {
    }
}
```

---

# Four Pillars of OOP

These are interview favorites.

1. Encapsulation
2. Abstraction
3. Inheritance
4. Polymorphism

Let's understand each in depth.

---

# 1. Encapsulation

### Definition

Encapsulation means **wrapping data and methods together into a single unit** and **controlling access** to that data.

Think of a bank account.

You cannot directly change the balance.

Instead,

```
Deposit()

Withdraw()
```

are provided.

Example

```csharp
public class BankAccount
{
    private decimal _balance;

    public void Deposit(decimal amount)
    {
        _balance += amount;
    }

    public decimal GetBalance()
    {
        return _balance;
    }
}
```

Notice

```
_balance
```

is private.

No one outside the class can modify it directly.

Instead,

```
Deposit()
```

controls the modification.

### Benefits

* Protects data
* Prevents invalid updates
* Easier maintenance
* Better security

---

Interview Question

Why not make everything public?

Because anyone can modify the object into an invalid state.

Example

```csharp
account.Balance = -100000;
```

Now the object is invalid.

Encapsulation prevents this.

---

# 2. Abstraction

This is commonly confused with encapsulation.

### Definition

Abstraction means

> Hide implementation details and expose only necessary functionality.

Example

When driving a car,

You press

```
Start
Brake
Accelerator
```

You don't know how the engine works internally.

That's abstraction.

Example

```csharp
public interface IPaymentService
{
    void ProcessPayment(decimal amount);
}
```

Implementation

```csharp
public class StripePaymentService : IPaymentService
{
    public void ProcessPayment(decimal amount)
    {
        // Stripe API logic
    }
}
```

Consumer

```csharp
IPaymentService payment = new StripePaymentService();

payment.ProcessPayment(100);
```

The caller doesn't know

* HTTP requests
* Authentication
* Serialization

Only

```
ProcessPayment()
```

---

Encapsulation vs Abstraction

This is asked very frequently.

| Encapsulation         | Abstraction                      |
| --------------------- | -------------------------------- |
| Hides data            | Hides implementation             |
| Uses access modifiers | Uses interfaces/abstract classes |
| Focuses on security   | Focuses on simplicity            |

---

# 3. Inheritance

Definition

Inheritance allows one class to reuse properties and methods from another class.

Example

```csharp
public class Employee
{
    public string Name { get; set; }

    public void Login()
    {
    }
}
```

Derived class

```csharp
public class Manager : Employee
{
    public void ApproveLeave()
    {
    }
}
```

Now

```
Manager
```

has

```
Name

Login()

ApproveLeave()
```

Advantages

* Code reuse
* Less duplication
* Easier maintenance

---

Interview Question

Can C# support multiple inheritance?

No.

Example (Not Allowed)

```csharp
class A { }

class B { }

class C : A, B
{
}
```

Reason

Diamond Problem.

Instead,

C# supports

Multiple Interface Inheritance

```csharp
public interface IA
{
}

public interface IB
{
}

public class C : IA, IB
{
}
```

---

# 4. Polymorphism

Meaning

One interface

Multiple implementations.

Types

* Compile-time Polymorphism
* Runtime Polymorphism

---

Compile-Time

Method Overloading

```csharp
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }

    public double Add(double a, double b)
    {
        return a + b;
    }
}
```

Same method

Different parameters

Compiler decides.

---

Runtime Polymorphism

Method Overriding

```csharp
public class Animal
{
    public virtual void Speak()
    {
        Console.WriteLine("Animal");
    }
}

public class Dog : Animal
{
    public override void Speak()
    {
        Console.WriteLine("Bark");
    }
}
```

Usage

```csharp
Animal animal = new Dog();

animal.Speak();
```

Output

```
Bark
```

At runtime,

CLR decides

Which method should execute.

---

# Interview Scenario

Suppose

```csharp
Employee emp = new Manager();
```

Which methods can you access?

Only

```
Employee
```

members.

Runtime object

```
Manager
```

Reference

```
Employee
```

This demonstrates **runtime polymorphism**.

---

# Frequently Asked Interview Questions

### Q1. What is the difference between Overloading and Overriding?

| Overloading          | Overriding                        |
| -------------------- | --------------------------------- |
| Same class           | Parent-child classes              |
| Compile time         | Runtime                           |
| Different parameters | Same signature                    |
| No `virtual` needed  | Requires `virtual` and `override` |

---

### Q2. Why use OOP?

* Code reusability
* Scalability
* Maintainability
* Security through encapsulation
* Extensibility
* Better testing

---

### Q3. Can we override a private method?

No.

Private methods are not inherited, so they cannot be overridden.

---

### Q4. Can constructors be inherited?

No.

Constructors are not inherited, but a derived constructor can call a base constructor using `base(...)`.

```csharp
public class Employee
{
    public Employee(string name)
    {
        Console.WriteLine(name);
    }
}

public class Manager : Employee
{
    public Manager() : base("Pankaj")
    {
    }
}
```

---

### Q5. Can we create an object of an abstract class?

No.

Abstract classes are meant to be inherited.

```csharp
Animal a = new Animal(); // ❌ Not allowed
```

---

### Q6. Can static methods be overridden?

No.

Static methods belong to the class, not an instance, so they cannot participate in runtime polymorphism.

---

## Interview Tip

For a **5–6 years experience** candidate, interviewers won't stop at "What is encapsulation?" They often follow up with practical questions like:

* Why did you choose inheritance instead of composition in your project?
* Where did you apply polymorphism in your APIs?
* How does dependency injection rely on abstraction and polymorphism?
* Can inheritance create tight coupling? How would you avoid it?

They expect you to connect OOP principles to real application design, not just definitions.

---

These are exactly the kind of **follow-up questions** interviewers ask to determine whether you understand design principles beyond theory. Let's answer them in the way a **5.5+ years .NET Backend Developer** should.

---

# 1. Why did you choose inheritance instead of composition in your project?

### What the interviewer is checking

They want to know if you understand **"Inheritance vs Composition"** and can justify your design decisions.

### Good Answer

> In general, I prefer **composition over inheritance** because it creates loosely coupled and more flexible systems. However, I use inheritance when there is a clear **"is-a" relationship** and multiple classes genuinely share common behavior.
>
> For example, if I have a base `BaseEntity` containing common properties like `Id`, `CreatedDate`, `ModifiedDate`, and `CreatedBy`, then entities like `Customer`, `Order`, and `Product` can inherit from it because they all represent entities with shared characteristics.
>
> On the other hand, for behaviors like sending notifications, logging, or payment processing, I use composition through dependency injection instead of inheritance. This allows me to swap implementations without modifying existing classes.

Example

```csharp
public abstract class BaseEntity
{
    public int Id { get; set; }

    public DateTime CreatedDate { get; set; }
}

public class Customer : BaseEntity
{
    public string Name { get; set; }
}
```

Composition Example

```csharp
public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

### Interview Tip

If they ask:

> "Would you still choose inheritance today?"

Say:

> "Only when there is a strong 'is-a' relationship. Otherwise, I prefer composition because it provides better flexibility, testability, and maintainability."

---

# 2. Where did you apply polymorphism in your APIs?

### What the interviewer wants

They want to know whether you've actually used interfaces and dependency injection.

### Example 1 (Best Example)

```csharp
public interface IPaymentService
{
    Task ProcessPaymentAsync();
}
```

Implementation 1

```csharp
public class StripePaymentService : IPaymentService
{
}
```

Implementation 2

```csharp
public class RazorpayPaymentService : IPaymentService
{
}
```

Usage

```csharp
public class OrderController
{
    private readonly IPaymentService _paymentService;

    public OrderController(IPaymentService paymentService)
    {
        _paymentService = paymentService;
    }
}
```

The controller doesn't know which payment provider is being used.

At runtime,

DI injects

```
StripePaymentService
```

or

```
RazorpayPaymentService
```

This is **runtime polymorphism**.

---

### Example from your experience

Based on your background, you can answer like this:

> In my .NET Core APIs, I frequently used interfaces with dependency injection. For example, I had interfaces like `IStudentService`, `IEmailService`, and `IFileStorageService`, each with one or more implementations. Controllers depended on the interfaces rather than concrete classes, allowing us to replace implementations or mock them during testing without changing controller code.

That is a very practical answer.

---

# 3. How does Dependency Injection rely on abstraction and polymorphism?

This is one of the most common senior-level questions.

### Step 1: Abstraction

Dependency Injection first requires **abstraction**.

Example

```csharp
public interface ILogger
{
    void Log(string message);
}
```

Notice

The class doesn't depend on

```csharp
ConsoleLogger
```

It depends on

```csharp
ILogger
```

That is abstraction.

---

### Step 2: Polymorphism

Now we have multiple implementations.

```csharp
public class ConsoleLogger : ILogger
{
}
```

```csharp
public class FileLogger : ILogger
{
}
```

Now

```csharp
ILogger logger = new ConsoleLogger();
```

or

```csharp
ILogger logger = new FileLogger();
```

The variable type remains

```csharp
ILogger
```

but behavior changes.

That is **runtime polymorphism**.

---

### Step 3: Dependency Injection

Now the container injects the required implementation.

```csharp
services.AddScoped<ILogger, FileLogger>();
```

Controller

```csharp
public class ProductService
{
    public ProductService(ILogger logger)
    {
    }
}
```

ProductService never creates

```csharp
new FileLogger()
```

Instead,

the DI container injects it.

---

### Complete Interview Answer

> Dependency Injection relies on abstraction because classes depend on interfaces rather than concrete implementations. It relies on polymorphism because multiple classes can implement the same interface, and the DI container decides which implementation to inject at runtime. This makes the application loosely coupled, easier to test, and easier to extend.

That is exactly what interviewers want.

---

# 4. Can inheritance create tight coupling? How would you avoid it?

### Answer

Yes.

Inheritance creates **compile-time coupling** between the parent and child classes.

Suppose

```csharp
public class Employee
{
    public void Login()
    {
    }

    public void Logout()
    {
    }
}
```

Now

```csharp
public class Manager : Employee
{
}
```

If someone changes

```csharp
Employee
```

all child classes are affected.

Sometimes,

changing the base class can unintentionally break multiple derived classes.

---

### Another Example

Suppose

```csharp
Vehicle
```

contains

```csharp
StartEngine()
```

Now

```
Car
Bike
Truck
ElectricScooter
```

inherit from it.

Later,

ElectricScooter doesn't even have an engine.

Now inheritance no longer models the real world correctly.

This is known as a bad inheritance hierarchy.

---

### How to Avoid Tight Coupling

Prefer **composition**.

Instead of

```csharp
class Car : Vehicle
```

do

```csharp
class Car
{
    private readonly IEngine _engine;
}
```

Now

```
PetrolEngine

DieselEngine

ElectricMotor
```

can all be injected.

No inheritance required.

This is much more flexible.

---

### Real Example

Instead of

```csharp
NotificationService
        ↑
EmailNotification

SMSNotification

PushNotification
```

Use

```csharp
INotificationSender
```

```
EmailSender

SmsSender

PushSender
```

Then inject

```csharp
INotificationSender
```

This allows adding new notification types without modifying existing code.

---

### Interview Answer

> Yes, inheritance can introduce tight coupling because child classes become dependent on the implementation of the base class. Changes in the base class can impact all derived classes, making the system harder to maintain. To avoid this, I prefer composition over inheritance wherever possible by injecting dependencies through interfaces. I reserve inheritance for scenarios with a genuine "is-a" relationship and use composition for reusable behaviors.

---

# Interview Follow-up Question (Very Common)

**Interviewer:** *"What do you prefer—Inheritance or Composition?"*

A strong answer is:

> I generally prefer **composition over inheritance** because it promotes loose coupling, better testability, and greater flexibility. Inheritance is useful when there's a clear and stable "is-a" relationship, such as a shared `BaseEntity` for common entity properties. For reusable behaviors like logging, notifications, payment processing, or file storage, I use composition with dependency injection, as it allows implementations to be changed or extended without affecting existing classes.

These are exactly the kind of **follow-up questions** interviewers ask to determine whether you understand design principles beyond theory. Let's answer them in the way a **5.5+ years .NET Backend Developer** should.

---

# 1. Why did you choose inheritance instead of composition in your project?

### What the interviewer is checking

They want to know if you understand **"Inheritance vs Composition"** and can justify your design decisions.

### Good Answer

> In general, I prefer **composition over inheritance** because it creates loosely coupled and more flexible systems. However, I use inheritance when there is a clear **"is-a" relationship** and multiple classes genuinely share common behavior.
>
> For example, if I have a base `BaseEntity` containing common properties like `Id`, `CreatedDate`, `ModifiedDate`, and `CreatedBy`, then entities like `Customer`, `Order`, and `Product` can inherit from it because they all represent entities with shared characteristics.
>
> On the other hand, for behaviors like sending notifications, logging, or payment processing, I use composition through dependency injection instead of inheritance. This allows me to swap implementations without modifying existing classes.

Example

```csharp
public abstract class BaseEntity
{
    public int Id { get; set; }

    public DateTime CreatedDate { get; set; }
}

public class Customer : BaseEntity
{
    public string Name { get; set; }
}
```

Composition Example

```csharp
public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

### Interview Tip

If they ask:

> "Would you still choose inheritance today?"

Say:

> "Only when there is a strong 'is-a' relationship. Otherwise, I prefer composition because it provides better flexibility, testability, and maintainability."

---

# 2. Where did you apply polymorphism in your APIs?

### What the interviewer wants

They want to know whether you've actually used interfaces and dependency injection.

### Example 1 (Best Example)

```csharp
public interface IPaymentService
{
    Task ProcessPaymentAsync();
}
```

Implementation 1

```csharp
public class StripePaymentService : IPaymentService
{
}
```

Implementation 2

```csharp
public class RazorpayPaymentService : IPaymentService
{
}
```

Usage

```csharp
public class OrderController
{
    private readonly IPaymentService _paymentService;

    public OrderController(IPaymentService paymentService)
    {
        _paymentService = paymentService;
    }
}
```

The controller doesn't know which payment provider is being used.

At runtime,

DI injects

```
StripePaymentService
```

or

```
RazorpayPaymentService
```

This is **runtime polymorphism**.

---

### Example from your experience

Based on your background, you can answer like this:

> In my .NET Core APIs, I frequently used interfaces with dependency injection. For example, I had interfaces like `IStudentService`, `IEmailService`, and `IFileStorageService`, each with one or more implementations. Controllers depended on the interfaces rather than concrete classes, allowing us to replace implementations or mock them during testing without changing controller code.

That is a very practical answer.

---

# 3. How does Dependency Injection rely on abstraction and polymorphism?

This is one of the most common senior-level questions.

### Step 1: Abstraction

Dependency Injection first requires **abstraction**.

Example

```csharp
public interface ILogger
{
    void Log(string message);
}
```

Notice

The class doesn't depend on

```csharp
ConsoleLogger
```

It depends on

```csharp
ILogger
```

That is abstraction.

---

### Step 2: Polymorphism

Now we have multiple implementations.

```csharp
public class ConsoleLogger : ILogger
{
}
```

```csharp
public class FileLogger : ILogger
{
}
```

Now

```csharp
ILogger logger = new ConsoleLogger();
```

or

```csharp
ILogger logger = new FileLogger();
```

The variable type remains

```csharp
ILogger
```

but behavior changes.

That is **runtime polymorphism**.

---

### Step 3: Dependency Injection

Now the container injects the required implementation.

```csharp
services.AddScoped<ILogger, FileLogger>();
```

Controller

```csharp
public class ProductService
{
    public ProductService(ILogger logger)
    {
    }
}
```

ProductService never creates

```csharp
new FileLogger()
```

Instead,

the DI container injects it.

---

### Complete Interview Answer

> Dependency Injection relies on abstraction because classes depend on interfaces rather than concrete implementations. It relies on polymorphism because multiple classes can implement the same interface, and the DI container decides which implementation to inject at runtime. This makes the application loosely coupled, easier to test, and easier to extend.

That is exactly what interviewers want.

---

# 4. Can inheritance create tight coupling? How would you avoid it?

### Answer

Yes.

Inheritance creates **compile-time coupling** between the parent and child classes.

Suppose

```csharp
public class Employee
{
    public void Login()
    {
    }

    public void Logout()
    {
    }
}
```

Now

```csharp
public class Manager : Employee
{
}
```

If someone changes

```csharp
Employee
```

all child classes are affected.

Sometimes,

changing the base class can unintentionally break multiple derived classes.

---

### Another Example

Suppose

```csharp
Vehicle
```

contains

```csharp
StartEngine()
```

Now

```
Car
Bike
Truck
ElectricScooter
```

inherit from it.

Later,

ElectricScooter doesn't even have an engine.

Now inheritance no longer models the real world correctly.

This is known as a bad inheritance hierarchy.

---

### How to Avoid Tight Coupling

Prefer **composition**.

Instead of

```csharp
class Car : Vehicle
```

do

```csharp
class Car
{
    private readonly IEngine _engine;
}
```

Now

```
PetrolEngine

DieselEngine

ElectricMotor
```

can all be injected.

No inheritance required.

This is much more flexible.

---

### Real Example

Instead of

```csharp
NotificationService
        ↑
EmailNotification

SMSNotification

PushNotification
```

Use

```csharp
INotificationSender
```

```
EmailSender

SmsSender

PushSender
```

Then inject

```csharp
INotificationSender
```

This allows adding new notification types without modifying existing code.

---

### Interview Answer

> Yes, inheritance can introduce tight coupling because child classes become dependent on the implementation of the base class. Changes in the base class can impact all derived classes, making the system harder to maintain. To avoid this, I prefer composition over inheritance wherever possible by injecting dependencies through interfaces. I reserve inheritance for scenarios with a genuine "is-a" relationship and use composition for reusable behaviors.

---

# Interview Follow-up Question (Very Common)

**Interviewer:** *"What do you prefer—Inheritance or Composition?"*

A strong answer is:

> I generally prefer **composition over inheritance** because it promotes loose coupling, better testability, and greater flexibility. Inheritance is useful when there's a clear and stable "is-a" relationship, such as a shared `BaseEntity` for common entity properties. For reusable behaviors like logging, notifications, payment processing, or file storage, I use composition with dependency injection, as it allows implementations to be changed or extended without affecting existing classes.

This answer demonstrates that you understand not just the concepts, but also how they influence maintainable application architecture.

Great. Now we'll move to the **second most important topic after OOP**.

# Day 1 – Part 2: SOLID Principles

> **Interview Importance: ⭐⭐⭐⭐⭐ (Very High)**

For **5+ years of experience**, almost every .NET interview includes SOLID principles. Don't just memorize definitions—understand the problems each principle solves.

---

# What is SOLID?

SOLID is a set of **five object-oriented design principles** that help build software that is:

* Easy to maintain
* Easy to extend
* Easy to test
* Loosely coupled
* Less prone to bugs

SOLID stands for:

| Letter | Principle                       |
| ------ | ------------------------------- |
| S      | Single Responsibility Principle |
| O      | Open/Closed Principle           |
| L      | Liskov Substitution Principle   |
| I      | Interface Segregation Principle |
| D      | Dependency Inversion Principle  |

---

# 1. Single Responsibility Principle (SRP)

## Definition

> **A class should have only one reason to change.**

This doesn't mean a class should have only one method.

It means a class should have **one responsibility**.

---

## Bad Example

Suppose we have an OrderService.

```csharp
public class OrderService
{
    public void SaveOrder()
    {
        // Save to database
    }

    public void SendEmail()
    {
        // Send confirmation email
    }

    public void GenerateInvoice()
    {
        // Create PDF invoice
    }
}
```

### What's wrong?

This class has **three responsibilities**:

* Database operations
* Email sending
* Invoice generation

If tomorrow:

* Email logic changes
* Invoice format changes
* Database changes

All require modifying the same class.

This violates SRP.

---

## Good Design

Separate responsibilities.

```csharp
public class OrderService
{
    public void SaveOrder()
    {
    }
}
```

```csharp
public class EmailService
{
    public void SendEmail()
    {
    }
}
```

```csharp
public class InvoiceService
{
    public void GenerateInvoice()
    {
    }
}
```

Each class now has **one responsibility**.

---

## Real Project Example

Imagine your Student Management System.

Instead of:

```text
StudentService

Save Student

Delete Student

Generate Excel

Send Email

Upload Photo

Create PDF
```

Split it into:

```text
StudentService

StudentReportService

StudentEmailService

StudentPhotoService
```

Much cleaner.

---

## Benefits

* Easier testing
* Easier maintenance
* Easier debugging
* Less merge conflicts
* Better code organization

---

## Interview Question

### Q: Is having one responsibility the same as having one method?

**Answer:**

No.

A class can have many methods.

Those methods should all contribute to **one responsibility**.

Example

```csharp
public class StudentService
{
    public void AddStudent()
    {
    }

    public void UpdateStudent()
    {
    }

    public void DeleteStudent()
    {
    }
}
```

All methods relate to **student management**, so SRP is not violated.

---

# 2. Open/Closed Principle (OCP)

## Definition

> **Software entities should be open for extension but closed for modification.**

Meaning:

You should be able to add new functionality **without modifying existing code**.

---

## Bad Example

```csharp
public class DiscountService
{
    public decimal Calculate(string customerType)
    {
        if(customerType=="Regular")
            return 10;

        if(customerType=="Premium")
            return 20;

        if(customerType=="VIP")
            return 30;

        return 0;
    }
}
```

Tomorrow a new customer type:

```
Gold
```

Now you modify this class again.

Next month

```
Diamond
```

Modify again.

This violates OCP.

---

## Good Example

Create an abstraction.

```csharp
public interface IDiscount
{
    decimal Calculate();
}
```

Regular

```csharp
public class RegularDiscount : IDiscount
{
    public decimal Calculate()
    {
        return 10;
    }
}
```

Premium

```csharp
public class PremiumDiscount : IDiscount
{
    public decimal Calculate()
    {
        return 20;
    }
}
```

Now,

adding

```
GoldDiscount
```

requires **creating a new class**, not changing existing ones.

---

## Real Project Example

Notification System

Instead of

```text
if(email)

if(sms)

if(push)

if(whatsapp)
```

Create

```
INotificationSender
```

Implement

```
EmailSender

SmsSender

PushSender

WhatsappSender
```

No existing code changes.

---

## Benefits

* Safer changes
* Easier to extend
* Lower risk of introducing bugs
* Supports plugin-like architectures

---

# 3. Liskov Substitution Principle (LSP)

This is one of the most misunderstood principles.

## Definition

> **Derived classes should be replaceable with their base class without changing the correctness of the program.**

Simply:

If `Dog` inherits `Animal`, then anywhere an `Animal` is expected, a `Dog` should work correctly.

---

## Bad Example

```csharp
public class Bird
{
    public virtual void Fly()
    {
    }
}
```

```csharp
public class Penguin : Bird
{
    public override void Fly()
    {
        throw new Exception();
    }
}
```

Problem:

Penguins can't fly.

So `Penguin` is not a proper substitute for `Bird`.

This violates LSP.

---

## Better Design

Separate behaviors.

```text
Animal

FlyingBird

NonFlyingBird
```

or

```
IFlyable
```

Only birds that fly implement `IFlyable`.

---

## Another Example

```csharp
Rectangle

Square
```

This classic example often breaks because changing width and height independently doesn't make sense for a square.

---

## Interview Tip

LSP is about **behavior**, not inheritance syntax.

---

# 4. Interface Segregation Principle (ISP)

## Definition

> **Clients should not be forced to depend on interfaces they do not use.**

---

## Bad Example

```csharp
public interface IWorker
{
    void Work();

    void Eat();

    void Sleep();
}
```

Robot

```csharp
public class Robot : IWorker
{
    public void Work()
    {
    }

    public void Eat()
    {
        throw new NotImplementedException();
    }

    public void Sleep()
    {
        throw new NotImplementedException();
    }
}
```

Robot doesn't eat or sleep.

Bad design.

---

## Good Example

Split interfaces.

```csharp
public interface IWork
{
    void Work();
}
```

```csharp
public interface IEat
{
    void Eat();
}
```

```csharp
public interface ISleep
{
    void Sleep();
}
```

Robot only implements

```text
IWork
```

Perfect.

---

## Real Project Example

Don't create

```text
IUserService

CreateUser()

DeleteUser()

SendEmail()

ExportExcel()

UploadImage()

GenerateReport()
```

Instead

```
IUserService

IReportService

IEmailService

IFileService
```

Much cleaner.

---

# 5. Dependency Inversion Principle (DIP)

This is the backbone of Dependency Injection.

## Definition

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

---

## Bad Example

```csharp
public class OrderService
{
    private readonly EmailService _emailService =
        new EmailService();
}
```

Problem:

OrderService depends directly on EmailService.

Hard to test.

Hard to replace.

---

## Good Example

```csharp
public interface IEmailService
{
    void Send();
}
```

Implementation

```csharp
public class EmailService : IEmailService
{
    public void Send()
    {
    }
}
```

Consumer

```csharp
public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

Now,

DI injects

```text
EmailService
```

or

```text
MockEmailService
```

or

```text
SendGridEmailService
```

No code changes.

---

# Interview Questions

### Q1. Which SOLID principle is used the most in .NET Core?

**Answer:**

Dependency Inversion Principle (DIP).

ASP.NET Core's built-in Dependency Injection container is based on DIP.

---

### Q2. Which SOLID principle improves testability the most?

**Answer:**

Dependency Inversion Principle.

Because dependencies can be mocked.

---

### Q3. Which principle prevents "God Classes"?

**Answer:**

Single Responsibility Principle.

---

### Q4. Which principle says we shouldn't modify existing code?

**Answer:**

Open/Closed Principle.

---

### Q5. What is the difference between DIP and DI?

This is asked **very frequently**.

| DIP                           | DI                                          |
| ----------------------------- | ------------------------------------------- |
| A design principle            | A design pattern / implementation technique |
| Says "depend on abstractions" | Injects those abstractions into classes     |
| Concept                       | Mechanism                                   |

**Example Answer:**

> Dependency Inversion Principle is a design principle that says high-level modules should depend on abstractions instead of concrete implementations. Dependency Injection is the technique used to provide those abstractions to a class, typically through constructor injection in ASP.NET Core.

---

# How to Answer in an Interview

If asked:

**"Which SOLID principle have you used in your projects?"**

A strong answer would be:

> "I've applied all five principles to varying degrees. The ones I use most frequently are SRP by separating business logic into focused services, OCP by extending functionality through interfaces instead of modifying existing classes, and DIP by using constructor-based dependency injection throughout ASP.NET Core applications. These practices have made the applications easier to test, maintain, and extend."

---

## Next Topic

The next topic is **Dependency Injection (DI)**. We'll go much deeper than just `AddScoped`, `AddSingleton`, and `AddTransient`. We'll cover:

* What happens internally when ASP.NET Core resolves dependencies.
* Constructor vs Property vs Method Injection.
* Service lifetimes (`Transient`, `Scoped`, `Singleton`) with real-world scenarios.
* Common mistakes (like injecting a Scoped service into a Singleton).
* How the built-in DI container works behind the scenes.
* The most frequently asked DI interview questions for 5+ years of experience.


Excellent. This is the **most important .NET interview topic**. Almost every 5+ years interview spends **20-30 minutes** on Dependency Injection.

I'll explain it exactly the way interviewers expect.

---

# 1. What happens internally when ASP.NET Core resolves dependencies?

This is a favorite senior-level question.

Suppose we have:

```csharp
builder.Services.AddScoped<IEmployeeService, EmployeeService>();
```

And

```csharp
public class EmployeeController : ControllerBase
{
    private readonly IEmployeeService _service;

    public EmployeeController(IEmployeeService service)
    {
        _service = service;
    }
}
```

---

## Step 1: Application Starts

When your application starts,

```csharp
builder.Services.AddScoped<IEmployeeService, EmployeeService>();
```

does **NOT** create an object.

It simply stores a mapping inside the **Service Collection**.

Think of it as a dictionary.

```
IEmployeeService
        ↓
EmployeeService
Lifetime = Scoped
```

No object exists yet.

---

## Step 2: Build()

When

```csharp
var app = builder.Build();
```

runs,

ASP.NET Core creates the **Service Provider**.

```
ServiceCollection
        ↓
Build()
        ↓
ServiceProvider
```

The Service Provider is the **DI Container** responsible for creating objects.

---

## Step 3: HTTP Request Comes

Suppose

```
GET /api/employees
```

comes in.

ASP.NET Core determines that it needs

```
EmployeeController
```

---

## Step 4: Constructor Inspection

Using Reflection,

ASP.NET Core checks

```csharp
public EmployeeController(IEmployeeService service)
```

It sees

```
Needs IEmployeeService
```

---

## Step 5: Resolve Dependency

ServiceProvider looks inside its registrations.

```
IEmployeeService

↓

EmployeeService
```

It creates

```csharp
new EmployeeService();
```

---

## Step 6: Recursive Resolution

Suppose EmployeeService itself needs

```csharp
public EmployeeService(
    IEmployeeRepository repository,
    ILogger<EmployeeService> logger)
{
}
```

Now DI resolves

```
Repository

Logger
```

first.

Then creates

```
EmployeeService
```

Finally

```
EmployeeController
```

This process is called the **Dependency Graph**.

Example

```
Controller
      ↓

Service

     ↓

Repository

     ↓

DbContext
```

DI recursively resolves the graph from the bottom up.

---

## Step 7: Constructor Injection

Finally

```csharp
new EmployeeController(service)
```

is created.

Controller executes.

---

### Interview Answer

> When a request arrives, ASP.NET Core uses the ServiceProvider to inspect the constructor of the requested class using reflection. It recursively resolves all registered dependencies according to their configured lifetimes, builds the dependency graph, creates the required objects, and injects them through the constructor before executing the request.

---

# 2. Constructor vs Property vs Method Injection

There are three types.

---

## Constructor Injection ⭐⭐⭐⭐⭐

Most common.

```csharp
public class EmployeeService
{
    private readonly ILogger<EmployeeService> _logger;

    public EmployeeService(ILogger<EmployeeService> logger)
    {
        _logger = logger;
    }
}
```

Advantages

* Mandatory dependency
* Immutable
* Easy testing
* Preferred by Microsoft

Use this **95% of the time**.

---

## Property Injection ⭐⭐

```csharp
public class EmployeeService
{
    public ILogger Logger { get; set; }
}
```

Problem

Someone can forget to set

```
Logger
```

Result

```
NullReferenceException
```

Not recommended.

Used only by some third-party containers.

---

## Method Injection ⭐⭐

```csharp
public void Process(IEmailService email)
{
}
```

Dependency is passed only to one method.

Useful when

* Dependency is needed temporarily.
* Optional.

---

### Which one should you use?

Always answer:

> Constructor Injection because it makes dependencies explicit, guarantees object validity, supports immutability, and is the recommended approach in ASP.NET Core.

---

# 3. Service Lifetimes

This is almost guaranteed in interviews.

---

## Transient

```csharp
services.AddTransient<IEmailService, EmailService>();
```

New instance every time.

```
Controller

↓

EmailService (1)

↓

EmailService (2)

↓

EmailService (3)
```

Every injection

↓

New object.

Good for

* Lightweight services
* Stateless helpers
* Email services
* PDF generators

---

## Scoped

```csharp
services.AddScoped<IEmployeeService, EmployeeService>();
```

One instance

Per HTTP Request.

Example

Request 1

```
Controller A

↓

EmployeeService

↓

Repository
```

Controller B

Same request

↓

Same EmployeeService instance.

---

Request 2

New instance.

Perfect for

* DbContext
* Business Services
* Repository

---

## Singleton

```csharp
services.AddSingleton<ICacheService, CacheService>();
```

Created once.

Lives until application stops.

```
Application Starts

↓

CacheService

↓

Every Request

↓

Same Object
```

Good for

* Cache
* Configuration
* Logging
* Memory Cache

---

## Real World Example

### Transient

```
PdfGenerator

EmailFormatter

OTPGenerator
```

Each request can have a new instance.

---

### Scoped

```
OrderService

CustomerService

EF DbContext

Repository
```

Each request gets one instance.

---

### Singleton

```
MemoryCache

Configuration

Application Settings

Redis Connection
```

One instance for the application.

---

# 4. Common Mistakes

## Mistake 1

Inject Scoped into Singleton

Example

```csharp
public class CacheService
{
    public CacheService(AppDbContext db)
    {
    }
}
```

Registered as

```csharp
AddSingleton<CacheService>();
```

Problem

DbContext is Scoped.

Singleton lives forever.

Scoped lives per request.

Impossible.

Runtime error

```
Cannot consume scoped service
from singleton.
```

---

## Why?

```
Singleton

↓

DbContext(Request 1)

↓

Request Ends

↓

DbContext Disposed

↓

Singleton Still Holds Reference

❌
```

---

## Solution

Use

```
IServiceScopeFactory
```

or redesign the dependency.

---

## Mistake 2

Using Singleton for DbContext

Never.

DbContext is **not thread-safe**.

---

## Mistake 3

State inside Singleton

```csharp
public class Counter
{
    public int Count;
}
```

Every request

Same Count.

Users affect each other.

---

## Mistake 4

Creating services manually

```csharp
new EmailService()
```

Now DI can't

* Manage lifetime
* Mock it
* Dispose it

---

# 5. How does the Built-in DI Container work?

Internally

```
ServiceCollection

↓

Stores Registration

↓

Build()

↓

ServiceProvider

↓

ObjectFactory

↓

Reflection

↓

Constructor Selection

↓

Create Object

↓

Inject Dependencies
```

Important concepts

### ServiceCollection

Stores registrations.

Like

```
Interface

Implementation

Lifetime
```

---

### ServiceProvider

Actually creates objects.

---

### ActivatorUtilities

Uses reflection

to invoke constructors.

---

### Constructor Selection

If multiple constructors exist,

DI chooses

The constructor with the **most resolvable parameters**.

Example

```csharp
public EmployeeService()
{
}
```

```csharp
public EmployeeService(
ILogger logger)
{
}
```

Second constructor is selected if `ILogger` is registered.

---

### Disposal

If a service implements

```csharp
IDisposable
```

DI disposes it automatically when its lifetime ends.

Example

Scoped

↓

Disposed after request.

Singleton

↓

Disposed when application shuts down.

---

# 6. Frequently Asked Interview Questions

---

## Q1 Why do we use Dependency Injection?

Answer

> Dependency Injection promotes loose coupling by allowing classes to depend on abstractions instead of concrete implementations. It improves maintainability, testability, and flexibility, making it easy to replace implementations or use mocks during testing.

---

## Q2 Why Constructor Injection over Property Injection?

Answer

* Mandatory dependency
* No NullReference
* Immutable
* Easier testing
* Microsoft recommended

---

## Q3 Difference between Scoped and Singleton?

| Scoped                            | Singleton           |
| --------------------------------- | ------------------- |
| Per Request                       | Entire Application  |
| Multiple Instances                | One Instance        |
| Thread-safe by design per request | Must be thread-safe |
| Good for DbContext                | Good for Cache      |

---

## Q4 Why is DbContext Scoped?

Because

* It represents a **Unit of Work** for a single request.
* It is **not thread-safe**.
* Sharing it across requests (Singleton) could lead to concurrency issues and data corruption.

---

## Q5 Can Singleton depend on Scoped?

No.

Because the Scoped object may be disposed while the Singleton is still alive.

---

## Q6 Can Scoped depend on Singleton?

Yes.

Singleton outlives Scoped.

Safe.

---

## Q7 Can Transient depend on Scoped?

Yes.

As long as both are resolved within the same request scope.

---

## Q8 Can Singleton depend on Transient?

Technically yes, but be careful.

If the Transient is injected into the Singleton constructor, it is created **only once** during Singleton creation and effectively behaves like part of the Singleton. If you need a fresh Transient instance each time, resolve it through a factory (for example, `IServiceProvider` or a dedicated factory abstraction) instead of constructor injection.

---

## Q9 What happens if two implementations are registered?

Example

```csharp
services.AddScoped<IPaymentService, StripePaymentService>();

services.AddScoped<IPaymentService, RazorpayPaymentService>();
```

If you inject

```csharp
IPaymentService
```

the **last registration wins** (`RazorpayPaymentService` in this example).

If you inject

```csharp
IEnumerable<IPaymentService>
```

you'll receive **both implementations** in the order they were registered.

---

## Q10 What is Service Locator? Why is it discouraged?

Example

```csharp
public class OrderService
{
    private readonly IServiceProvider _provider;

    public OrderService(IServiceProvider provider)
    {
        _provider = provider;
    }

    public void Process()
    {
        var email = _provider.GetRequiredService<IEmailService>();
    }
}
```

This is known as the **Service Locator pattern**.

### Why is it discouraged?

* Hides dependencies (you can't tell from the constructor what the class needs).
* Makes testing harder because dependencies are resolved internally.
* Violates the Dependency Inversion Principle by relying on the container instead of explicit abstractions.

Prefer **constructor injection**, where dependencies are clearly declared.

---

# Interview Tip (5+ Years)

A common senior-level question is:

> **"Explain what happens when ASP.NET Core creates a controller using Dependency Injection."**

A strong answer is:

> "At application startup, services are registered in the `IServiceCollection`. When `Build()` is called, ASP.NET Core creates a `ServiceProvider`, which acts as the DI container. When a request reaches a controller, the framework inspects the controller's constructor using reflection, builds the dependency graph by recursively resolving registered services according to their lifetimes, creates the required objects, injects them through the constructor, and automatically disposes disposable services when their lifetime ends."

--------
Excellent. Now let's move to what is arguably the **#1 technical topic** for senior .NET interviews.

# Day 1 – Part 4: async/await, Task & Multithreading

> **Interview Importance: ⭐⭐⭐⭐⭐**

Almost every interviewer asks at least one of these:

* What happens internally when you use `async/await`?
* How does `await` work?
* Difference between `Task`, `Thread`, and `async`?
* Can `async` improve performance?
* Why should we use async in Web APIs?
* What is `ConfigureAwait(false)`?
* What happens if you don't `await` a Task?

Let's understand it from the basics.

---

# Why do we need async?

Suppose your API calls SQL Server.

```csharp
public IActionResult GetEmployees()
{
    var employees = _context.Employees.ToList();

    return Ok(employees);
}
```

Timeline

```
Request

↓

Thread Starts

↓

SQL Query (3 seconds)

↓

Thread Waits

↓

Response
```

During those **3 seconds**, the thread is **doing nothing**.

It is simply waiting for SQL Server.

That is a waste of a valuable ThreadPool thread.

---

## What happens in a synchronous call?

```
Request

↓

Thread #25

↓

Call SQL

↓

Wait...

↓

Wait...

↓

Wait...

↓

Receive Data

↓

Return Response
```

Thread #25 is blocked.

If 1000 requests come simultaneously,

ASP.NET needs many more threads.

Eventually, the ThreadPool becomes exhausted.

---

# What does async do?

Instead of blocking the thread,

it releases it back to the ThreadPool while waiting.

```
Request

↓

Thread #25

↓

Start SQL Query

↓

Release Thread

↓

ThreadPool can use Thread #25 for another request

↓

SQL Completes

↓

Resume execution

↓

Return Response
```

This is why ASP.NET Core scales better with asynchronous I/O.

---

# Important Interview Point

### Does async make SQL faster?

**No.**

This is a common misconception.

Suppose SQL takes

```
3 seconds
```

Sync

```
3 seconds
```

Async

```
Still 3 seconds
```

The SQL execution time hasn't changed.

The benefit is that the server thread is free to process other requests.

### Interview Answer

> Async does not reduce execution time for I/O operations. It improves scalability by preventing request threads from being blocked while waiting for external resources such as databases, file systems, or HTTP services.

---

# What is Task?

A `Task` represents an operation that **will complete in the future**.

Example

```csharp
Task<int> task = GetEmployeesAsync();
```

You're not getting the result immediately.

You're getting a promise that the result will be available later.

---

# What is async?

`async` tells the compiler:

> "This method contains one or more await operations."

Example

```csharp
public async Task<Employee> GetEmployeeAsync()
{
}
```

Notice

`async` does **not** create a new thread.

This is a very common interview question.

---

# What does await do?

Suppose

```csharp
public async Task GetData()
{
    var employees = await GetEmployeesAsync();

    Console.WriteLine(employees.Count);
}
```

When execution reaches

```csharp
await GetEmployeesAsync();
```

the method pauses.

The current thread is returned to the ThreadPool.

When the database call finishes,

execution resumes **after** the `await`.

---

# What happens internally?

Consider:

```csharp
public async Task<int> GetValueAsync()
{
    await Task.Delay(5000);

    return 10;
}
```

The compiler transforms it into a **state machine**.

Conceptually:

```
State 0

↓

Call Task.Delay()

↓

Return to caller

↓

Delay completes

↓

State 1

↓

Return 10
```

The compiler generates this state machine automatically using `AsyncTaskMethodBuilder`.

You don't write it yourself.

---

# Does async create a new thread?

**No.**

This is one of the most frequently asked questions.

### Wrong Answer

> Yes, async creates another thread.

### Correct Answer

> No. Async itself does not create a new thread. It enables non-blocking asynchronous operations. New threads are only created if you explicitly use APIs such as `Task.Run()`.

---

# Task vs Thread

| Task                 | Thread                              |
| -------------------- | ----------------------------------- |
| Logical unit of work | Physical OS thread                  |
| Lightweight          | Expensive                           |
| Uses ThreadPool      | Own OS thread                       |
| Preferred in .NET    | Used less often in application code |

Think of it like this:

```
Task

↓

ThreadPool

↓

Available Thread
```

Tasks are scheduled onto ThreadPool threads.

---

# Thread vs ThreadPool vs Task

## Thread

```csharp
var thread = new Thread(() =>
{
    Console.WriteLine("Hello");
});

thread.Start();
```

Creates a dedicated OS thread.

Expensive.

---

## ThreadPool

Managed by .NET.

```
Pool

↓

Thread 1

Thread 2

Thread 3
```

Threads are reused.

More efficient.

---

## Task

```csharp
Task.Run(() =>
{
});
```

Uses the ThreadPool internally.

Preferred.

---

# Task.Run()

Example

```csharp
await Task.Run(() =>
{
    CalculateLargeReport();
});
```

This executes CPU-intensive work on a ThreadPool thread.

---

## Should you use Task.Run() in ASP.NET Core APIs?

Usually **No**.

Why?

ASP.NET Core already uses ThreadPool threads for request processing.

Wrapping synchronous code in `Task.Run()` usually just moves work from one ThreadPool thread to another.

Only use it when you have genuine CPU-bound work that you intentionally want to offload.

---

# CPU-bound vs I/O-bound

## CPU-bound

Examples:

* Image processing
* PDF generation
* Compression
* Encryption
* Complex calculations

Use:

```csharp
Task.Run(...)
```

if you need to offload that work.

---

## I/O-bound

Examples:

* SQL queries
* HTTP calls
* Reading files
* Azure Blob Storage
* Redis

Use the asynchronous APIs directly.

```csharp
await _context.Employees.ToListAsync();
```

Do **not** wrap them in `Task.Run()`.

---

# Return Types

### Task

```csharp
public async Task SaveAsync()
{
}
```

No return value.

---

### Task<T>

```csharp
public async Task<Employee> GetEmployeeAsync()
{
}
```

Returns a value.

---

### ValueTask

Introduced to reduce allocations when an operation often completes synchronously.

```csharp
public ValueTask<int> GetValueAsync()
{
}
```

Use only when profiling shows it provides a benefit. For most application code, `Task` is the better default.

---

### async void

Avoid it.

```csharp
public async void Save()
{
}
```

Problems:

* Caller can't await it.
* Exceptions are difficult to handle.
* Poor testability.

Use only for event handlers.

---

# ConfigureAwait(false)

This is a classic interview topic.

In older ASP.NET and desktop UI frameworks, `await` captured the current synchronization context by default.

`ConfigureAwait(false)` tells the runtime:

> "Don't resume on the original synchronization context."

Example

```csharp
await SomeOperationAsync().ConfigureAwait(false);
```

Benefits in applications with a synchronization context:

* Avoids unnecessary context switching.
* Helps prevent certain deadlocks.

### What about ASP.NET Core?

ASP.NET Core **does not have a `SynchronizationContext`** like classic ASP.NET or WinForms/WPF.

So in ASP.NET Core applications, `ConfigureAwait(false)` generally provides **little or no practical benefit**.

It is still commonly used in reusable libraries because those libraries might run in environments that do have a synchronization context.

---

# Common Interview Questions

## Q1. What happens if you don't await a Task?

Example

```csharp
SaveAsync();

return Ok();
```

The API returns immediately.

The background operation continues.

If it throws an exception, you may never observe it unless you handle it.

---

## Q2. Can an async method have no await?

Yes.

```csharp
public async Task Test()
{
}
```

It compiles (with a warning).

The compiler warns because the method runs synchronously.

In most cases, remove `async` or add an actual `await`.

---

## Q3. Can Main() be async?

Yes.

```csharp
public static async Task Main()
{
}
```

Supported in modern C#.

---

## Q4. Can constructors be async?

No.

Constructors cannot be declared `async`.

Alternatives:

* Static async factory methods.
* Explicit initialization methods.

---

## Q5. Is async faster than synchronous code?

No.

Async improves **throughput and scalability**, not the speed of the underlying operation.

---

## Q6. Why should EF Core use ToListAsync() instead of ToList()?

Because database access is I/O-bound.

`ToListAsync()` frees the request thread while waiting for SQL Server, allowing the server to handle more concurrent requests.

---

# Questions You Will Almost Certainly Be Asked

1. Explain `async` and `await` internally.
2. Does `async` create a new thread?
3. What is the difference between `Task` and `Thread`?
4. When would you use `Task.Run()`?
5. Explain CPU-bound vs I/O-bound operations.
6. Why should Web APIs be asynchronous?
7. Why is `async void` discouraged?
8. What happens if you forget to `await` a Task?
9. Explain `ConfigureAwait(false)` and when it matters.
10. How does the compiler implement `async`/`await`?

If you can answer those confidently, you'll be well prepared for the async portion of a senior .NET interview.

---

## Next Topic

The next topic in Day 1 is **Memory Management & Garbage Collection**, where we'll cover:

* Stack vs Heap
* Value Types vs Reference Types (deep dive)
* Garbage Collection generations (Gen 0, 1, 2)
* Large Object Heap (LOH)
* Boxing & Unboxing
* `IDisposable` vs Finalizers
* `using` statement and `using` declaration
* Memory leaks in .NET
* Weak references
* Common GC interview questions for experienced developers
Excellent. This is one of the **hardest and most frequently asked** topics for experienced .NET developers. Interviewers ask it because it reveals whether you understand what happens **inside the CLR**, not just how to write C#.

# Day 1 – Part 5: Memory Management & Garbage Collection (Deep Dive)

> **Interview Importance: ⭐⭐⭐⭐⭐**

---

# Memory Management Architecture

When a .NET application starts, the CLR (Common Language Runtime) manages memory for your application.

A simplified view looks like this:

```text
+--------------------------------------+
|          Process Memory              |
+--------------------------------------+
|   Code Segment                       |
|--------------------------------------|
|   Stack (per thread)                 |
|--------------------------------------|
|   Managed Heap                       |
|--------------------------------------|
|   Large Object Heap (LOH)            |
|--------------------------------------|
|   Native Heap                        |
+--------------------------------------+
```

Today we'll mainly discuss:

* Stack
* Managed Heap
* Garbage Collector

---

# What is the Stack?

The **stack** stores:

* Local variables
* Method parameters
* Value types (when they are local variables)
* References (addresses) to objects on the heap
* Method call information

Think of it as a stack of plates.

```
Push
Push
Push

↓

Pop
Pop
Pop
```

The **Last In, First Out (LIFO)** principle applies.

---

## Example

```csharp
public void Display()
{
    int age = 25;
    double salary = 1000;

    Print(age);
}

public void Print(int age)
{
}
```

Memory

```
STACK

-------------------------
Print()

age = 25
-------------------------

Display()

age = 25

salary = 1000
-------------------------
```

When `Print()` finishes,

its stack frame disappears instantly.

No Garbage Collector is involved.

---

## Characteristics of Stack

* Very fast
* Automatically managed
* One stack per thread
* Limited size
* LIFO

---

# What is the Heap?

The heap stores:

* Objects
* Arrays
* Strings
* Classes
* Delegates

Example

```csharp
Employee emp = new Employee();
```

Memory

```
STACK

emp
↓

0x1020

-------------------

HEAP

0x1020

Employee

Name

Age
```

Notice

The stack only stores

```
Reference

↓

0x1020
```

The actual object lives in the heap.

---

# Stack vs Heap

| Stack                  | Heap                        |
| ---------------------- | --------------------------- |
| Stores local variables | Stores objects              |
| Fast                   | Slower than stack           |
| Automatic cleanup      | Garbage Collector cleans it |
| Per thread             | Shared across threads       |
| Fixed size             | Grows dynamically           |

---

# Why Objects Go to Heap

Suppose

```csharp
Employee emp = new Employee();
```

Imagine the object is 50 KB.

If stored on the stack,

every method call would require copying 50 KB.

Very expensive.

Instead,

the stack stores only an **8-byte reference** (on a 64-bit process).

Efficient.

---

# Value Types vs Reference Types

One of the most common interview questions.

## Value Types

Examples

```csharp
int

double

bool

DateTime

decimal

struct

enum
```

Usually stored directly in the stack when used as local variables.

Example

```csharp
int x = 10;
int y = x;
```

Memory

```
STACK

x = 10

y = 10
```

Two independent values.

Changing one doesn't affect the other.

---

## Reference Types

Examples

```csharp
class

array

delegate

interface

string
```

Example

```csharp
Employee e1 = new Employee();

Employee e2 = e1;
```

Memory

```
STACK

e1

↓

0x500

e2

↓

0x500

----------------

HEAP

Employee
```

Both variables point to the **same object**.

Changing through one reference is visible through the other.

---

## Interview Question

```csharp
Employee a = new Employee();

Employee b = a;

b.Name = "John";
```

What is `a.Name`?

Answer:

```
John
```

Because both references point to the same object.

---

# Boxing and Unboxing

A favorite interview question.

## Boxing

Converting a value type into a reference type.

Example

```csharp
int number = 10;

object obj = number;
```

What happens?

```
STACK

number = 10

↓

Heap

Object(10)
```

A new object is created on the heap.

This allocation has a performance cost.

---

## Unboxing

```csharp
object obj = 10;

int x = (int)obj;
```

The value is copied back to a value type.

---

## Why Avoid Boxing?

Imagine

```csharp
for(int i=0;i<1000000;i++)
{
    object obj=i;
}
```

One million heap allocations.

Extra GC pressure.

Use generics instead.

---

# What is Garbage Collection?

Garbage Collection automatically removes objects that are no longer reachable.

Example

```csharp
Employee emp = new Employee();
```

Later

```csharp
emp = null;
```

Now no reference points to the object.

```
Heap

Employee

×

Unreachable
```

Eventually,

the Garbage Collector removes it.

---

# How does the Garbage Collector know an object is unused?

The GC starts from **GC Roots**.

GC Roots include:

* Local variables on thread stacks
* Static fields
* CPU registers
* Objects referenced by the runtime

Example

```
GC Root

↓

Employee

↓

Department

↓

Manager
```

Everything reachable is **alive**.

Anything not reachable becomes garbage.

This process is called **reachability analysis**.

---

# GC Generations

This is asked very frequently.

.NET divides objects into generations.

```
Gen 0

↓

Gen 1

↓

Gen 2
```

---

## Generation 0

New objects.

```csharp
new Employee();
```

Every new object starts in Gen 0.

Most objects die here.

Examples:

* Request DTOs
* Temporary lists
* LINQ results

---

## Generation 1

Objects that survive one GC.

```
Gen 0

↓

Survived

↓

Gen 1
```

Acts as a buffer.

---

## Generation 2

Long-lived objects.

Examples:

* Cache
* Singleton services
* Static collections

```
Gen 2

↓

Application Lifetime
```

GC runs here less frequently because collecting Gen 2 is expensive.

---

# Large Object Heap (LOH)

Objects larger than **85,000 bytes** go to the Large Object Heap.

Example

```csharp
byte[] file = new byte[100000];
```

This is approximately 100 KB, so it is allocated on the LOH.

Characteristics:

* Not allocated in Gen 0.
* Collected with Gen 2 collections.
* More expensive to manage.
* Can become fragmented.

Avoid repeatedly allocating large arrays when possible. Reuse buffers with techniques such as `ArrayPool<T>` if appropriate.

---

# Garbage Collection Process

Suppose

```csharp
Employee e1 = new Employee();

Employee e2 = new Employee();
```

Later

```csharp
e1 = null;
```

Heap

```
Employee

×

Garbage

Employee

Alive
```

GC performs:

### 1. Mark

Marks reachable objects.

### 2. Sweep

Removes unreachable objects.

### 3. Compact

Moves remaining objects together to reduce fragmentation and updates references.

---

# IDisposable vs Garbage Collection

Many developers misunderstand this.

Suppose

```csharp
SqlConnection connection = new SqlConnection();
```

GC manages **memory**, but it does **not** immediately release unmanaged resources such as:

* Database connections
* File handles
* Network sockets

That's why these types implement `IDisposable`.

Example

```csharp
using var connection = new SqlConnection(connectionString);
```

When execution leaves the scope,

`Dispose()` is called automatically.

---

# Finalizer

Example

```csharp
class FileManager
{
    ~FileManager()
    {
    }
}
```

Finalizers are executed by the runtime before memory is reclaimed, but **you cannot predict when** they will run.

They are slower than `Dispose()` and should only be used when your class directly owns unmanaged resources.

---

# using Statement

```csharp
using(var file = new StreamReader(path))
{
}
```

Equivalent to

```csharp
StreamReader file = new StreamReader(path);

try
{
}
finally
{
    file.Dispose();
}
```

---

# Common Memory Leaks in .NET

Developers often think .NET cannot have memory leaks.

It can.

Example:

```csharp
public static List<Employee> Employees = new();
```

If objects are continually added and never removed,

the list keeps references alive.

The GC cannot collect those objects.

---

Another example:

Event subscriptions.

```csharp
publisher.SomeEvent += subscriber.Handler;
```

If you forget to unsubscribe when appropriate,

the publisher still references the subscriber,

keeping it alive longer than intended.

---

# WeakReference

A `WeakReference` allows an object to be collected even though you still have a weak reference to it.

Useful in specialized scenarios such as certain cache implementations.

Not commonly used in everyday business applications.

---

# Common Interview Questions

## Q1. Why is the stack faster than the heap?

Answer:

* Contiguous memory
* LIFO allocation/deallocation
* No GC required
* No fragmentation management

---

## Q2. Can a value type live on the heap?

Yes.

Examples include:

* When it's boxed.
* When it's a field inside a reference type object.
* When it's captured by certain compiler-generated structures (such as closures).

So it's not accurate to say value types are **always** on the stack.

---

## Q3. Does `Dispose()` free memory?

No.

`Dispose()` releases unmanaged resources.

Memory is still reclaimed by the Garbage Collector when the object becomes unreachable.

---

## Q4. Can we force Garbage Collection?

Yes.

```csharp
GC.Collect();
```

But avoid calling it in production unless you have a very specific reason.

The runtime usually knows better when to collect.

---

## Q5. Why is Gen 2 collection expensive?

Because it scans long-lived objects and may include the Large Object Heap, making it significantly more work than collecting Gen 0.

---

## Q6. What causes memory leaks in .NET?

Common causes include:

* Static collections that keep growing.
* Event subscriptions that aren't removed.
* Long-lived caches without eviction.
* Holding references longer than necessary.
* Not disposing unmanaged resources promptly (resource leak rather than managed memory leak).

---

# Interview Tip

A strong answer to:

**"Explain Garbage Collection."**

would be:

> ".NET uses a generational Garbage Collector that automatically manages managed memory. New objects are allocated on the managed heap in Generation 0. During a collection, the GC starts from the GC Roots, marks all reachable objects, removes unreachable ones, and compacts memory to reduce fragmentation. Objects that survive collections are promoted to higher generations, with Gen 2 holding long-lived objects. The GC manages managed memory only; unmanaged resources such as database connections and file handles should be released using `IDisposable` and the `using` pattern."

That answer demonstrates both conceptual understanding and practical knowledge expected from a senior .NET developer.

