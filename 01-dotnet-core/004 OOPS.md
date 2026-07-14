## 1. Implicit Interface Implementation

**Definition:**
A class implements an interface by creating **public members** with the same signature as the interface members.

### Example

```csharp
public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

### Usage

```csharp
ConsoleLogger logger = new ConsoleLogger();
logger.Log("Hello");      // ✅

ILogger iLogger = logger;
iLogger.Log("Hello");     // ✅
```

**Key Points**

* Members are **public**.
* Accessible through both the **class** and the **interface**.
* Most commonly used (e.g., ASP.NET Core services, repositories).

---

## 2. Explicit Interface Implementation

**Definition:**
A class implements an interface by prefixing the member with the **interface name**. The member is **not publicly accessible** and can only be called through the interface.

### Example

```csharp
public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    void ILogger.Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

### Usage

```csharp
ConsoleLogger logger = new ConsoleLogger();

// logger.Log("Hello");   // ❌ Compile-time error

ILogger iLogger = logger;
iLogger.Log("Hello");     // ✅
```

**Key Points**

* Members are **not public**.
* Accessible **only through the interface**.
* Used to **hide methods** or **resolve conflicts** when multiple interfaces have the same member.

---

## Interview Comparison

| Feature              | Implicit              | Explicit                                   |
| -------------------- | --------------------- | ------------------------------------------ |
| Method Declaration   | `public void Log()`   | `void ILogger.Log()`                       |
| Public               | ✅ Yes                 | ❌ No                                       |
| Access via Class     | ✅ Yes                 | ❌ No                                       |
| Access via Interface | ✅ Yes                 | ✅ Yes                                      |
| Main Use             | Normal implementation | Hide members / Resolve interface conflicts |

### One-line Interview Answer

* **Implicit Implementation:** Interface members are implemented as **public methods**, so they can be accessed through both the class and the interface.
* **Explicit Implementation:** Interface members are implemented with the **interface name**, so they are accessible **only through the interface** and not through the class instance.


# 1. What is `readonly` in C#?

`readonly` is used for an **instance field** whose value **can only be assigned once**—either at declaration or inside the constructor. After that, it cannot be changed.

### Example

```csharp
public class Employee
{
    public readonly int EmployeeId;

    public Employee(int id)
    {
        EmployeeId = id;
    }
}
```

### Usage

```csharp
Employee emp = new Employee(101);

Console.WriteLine(emp.EmployeeId);   // 101

// emp.EmployeeId = 102;   // ❌ Compile-time error
```

### Use Cases

* Employee ID
* Date of Birth
* Order ID
* Values that should remain constant for each object after creation

### Interview Answer (30 seconds)

> `readonly` is used for instance fields whose value is assigned only once, either during declaration or in the constructor. After initialization, the value cannot be modified. It is commonly used for immutable object data like IDs or creation dates.

---

# 2. What is `const` (Constant) in C#?

A `const` is a **compile-time constant** whose value **must be known at compile time** and **can never change**.

### Example

```csharp
public class MathConstants
{
    public const double PI = 3.14159;
}
```

### Usage

```csharp
Console.WriteLine(MathConstants.PI);

// MathConstants.PI = 3.14;   // ❌ Compile-time error
```

### Use Cases

* Mathematical constants (`PI`)
* Tax rates (if fixed)
* Error messages
* Fixed application values

### Interview Answer (30 seconds)

> A `const` is a compile-time constant whose value must be assigned when declared and cannot be changed. It is implicitly static, so it's accessed using the class name. It is commonly used for values that never change, such as `PI` or fixed application constants.

---

# `readonly` vs `const`

| Feature                         | `readonly`                           | `const`                     |
| ------------------------------- | ------------------------------------ | --------------------------- |
| Value assigned                  | Runtime (constructor or declaration) | Compile time only           |
| Can change after initialization | ❌ No                                 | ❌ No                        |
| Belongs to                      | Object (instance) by default         | Class (implicitly `static`) |
| Can use `DateTime.Now`          | ✅ Yes                                | ❌ No                        |
| Example                         | Employee ID                          | PI, MaxValue                |

### Quick Rule to Remember

* **`const`** → Fixed **compile-time** value that will **never** change.
* **`readonly`** → Fixed **runtime** value that is set **once per object**.

# What is `static` in C#?

A **`static`** member belongs to the **class itself**, not to any object (instance). You can access it **without creating an object**.

### Example

```csharp
public class MathHelper
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}

// Usage
int result = MathHelper.Add(10, 20);
```

---

## Key Points

* Belongs to the **class**, not an instance.
* Only **one copy** exists in memory.
* Access using the **class name**.
* No need to create an object.

---

## Common Use Cases

### 1. Utility/Helper Methods

```csharp
public static class StringHelper
{
    public static bool IsNullOrEmpty(string text)
    {
        return string.IsNullOrEmpty(text);
    }
}
```

---

### 2. Shared Variables

```csharp
public class Employee
{
    public static int TotalEmployees = 0;
}
```

Used to keep a count shared by all instances.

---

### 3. Singleton Support

```csharp
public static readonly Logger Instance = new Logger();
```

Used in Singleton implementations.

---

### 4. Constants and Configuration

```csharp
public static string ApplicationName = "CRM System";
```

Shared application-wide values.

---

## Interview Answer (30 seconds)

> `static` means a member belongs to the class rather than an object. Only one copy exists for the entire application, and it can be accessed using the class name without creating an instance. It's commonly used for utility methods, shared variables, application-wide configuration, and Singleton implementations.

### Example

```csharp
public class Calculator
{
    public static int Add(int a, int b) => a + b;
}

int result = Calculator.Add(5, 10);
```

**Output:**

```
15
```


  # What is `static readonly` in C#?

`static readonly` is used for a **class-level variable** whose value is **shared by all objects** and **can only be assigned once** (either at declaration or in a static constructor).

### Syntax

```csharp
public static readonly string CompanyName = "OpenAI";
```

or

```csharp
public class Config
{
    public static readonly string Environment;

    static Config()
    {
        Environment = "Production";
    }
}
```

---

## Key Points

* **`static`** → One shared copy for the entire application.
* **`readonly`** → Value can only be assigned during declaration or in the static constructor.
* Cannot be modified afterward.

---

## Use Cases

### 1. Application Constants (known at runtime)

```csharp
public static readonly string ApiBaseUrl = "https://api.example.com";
```

---

### 2. Configuration Values

```csharp
public static readonly string ConnectionString = GetConnectionString();
```

---

### 3. Shared Objects

```csharp
public static readonly HttpClient HttpClient = new HttpClient();
```

This avoids creating multiple `HttpClient` instances.

---

### 4. Default Values

```csharp
public static readonly DateTime AppStartTime = DateTime.Now;
```

Stores the application's startup time.

---

## `const` vs `readonly` vs `static readonly`

| Feature                      | `const`                   | `readonly`                  | `static readonly`     |
| ---------------------------- | ------------------------- | --------------------------- | --------------------- |
| Shared across all objects    | ✅                         | ❌                           | ✅                     |
| Assigned at runtime          | ❌                         | ✅                           | ✅                     |
| Changed after initialization | ❌                         | ❌                           | ❌                     |
| Use for                      | Fixed compile-time values | Per-instance runtime values | Shared runtime values |

---

## Interview Answer (30 seconds)

> `static readonly` is used to create a class-level field that is shared across all instances and can only be assigned once, either at declaration or in a static constructor. It's commonly used for configuration values, shared objects like `HttpClient`, or values determined at application startup that should remain immutable.

# How to Catch Multiple Exceptions at Once in C#?

There are **three common ways** to handle multiple exceptions in C#.

---

# 1. Multiple `catch` Blocks (Most Common)

Handle each exception differently.

```csharp
try
{
    int result = 10 / int.Parse("0");
}
catch (DivideByZeroException ex)
{
    Console.WriteLine("Cannot divide by zero.");
}
catch (FormatException ex)
{
    Console.WriteLine("Invalid number format.");
}
catch (Exception ex)
{
    Console.WriteLine("General exception.");
}
```

**Use Case:** Different exceptions require different handling.

---

# 2. Single `catch` with Exception Filters (`when`)

Catch multiple exception types in one block.

```csharp
try
{
    // Code
}
catch (Exception ex) when (
    ex is DivideByZeroException ||
    ex is FormatException)
{
    Console.WriteLine("Handled DivideByZero or FormatException.");
}
```

**Use Case:** Multiple exceptions need the **same handling**.

---

# 3. Catch the Base `Exception`

Catch all exceptions.

```csharp
try
{
    // Code
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message);
}
```

**Use Case:** Global logging or a final fallback handler.

---

# Interview Answer (30 seconds)

> In C#, multiple exceptions can be handled using multiple `catch` blocks, a single `catch` block with an exception filter (`when`) for common handling, or a general `catch (Exception)` block as a fallback. The preferred approach is to catch specific exceptions first and keep `Exception` as the last catch block.

### Best Practice

* ✅ Catch **specific exceptions first**.
* ✅ Keep `catch (Exception)` **last**.
* ✅ Use a **single `catch` with `when`** if multiple exception types share the same handling logic.

# What is `IDisposable` in C#?

`IDisposable` is an interface used to **release unmanaged resources** (or other expensive resources) when they are no longer needed.

It contains a single method:

```csharp
void Dispose();
```

Calling `Dispose()` frees resources immediately instead of waiting for the Garbage Collector.

---

# Why is it needed?

The **Garbage Collector (GC)** only cleans up **managed memory**. It does **not** automatically release unmanaged resources such as:

* File handles
* Database connections
* Network sockets
* Streams
* OS handles

`IDisposable` allows you to release these resources explicitly.

---

# Example

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        using (StreamReader reader = new StreamReader("test.txt"))
        {
            Console.WriteLine(reader.ReadToEnd());
        }   // Dispose() is called automatically here
    }
}
```

Without `using`, you would need:

```csharp
StreamReader reader = new StreamReader("test.txt");

try
{
    Console.WriteLine(reader.ReadToEnd());
}
finally
{
    reader.Dispose();
}
```

---

# Creating Your Own `IDisposable`

```csharp
public class FileManager : IDisposable
{
    public void Dispose()
    {
        Console.WriteLine("Resources released.");
    }
}
```

Usage:

```csharp
using var manager = new FileManager();

// Work with manager
```

When execution leaves the scope, `Dispose()` is called automatically.

---

# Common Classes That Implement `IDisposable`

* `SqlConnection`
* `SqlCommand`
* `FileStream`
* `StreamReader`
* `MemoryStream`
* `HttpClient` (in some usage patterns)
* `DbContext` (EF Core)

---

# `using` Statement

The `using` statement ensures `Dispose()` is called automatically, even if an exception occurs.

```csharp
using var connection = new SqlConnection(connectionString);

// Use the connection
```

Equivalent to:

```csharp
var connection = new SqlConnection(connectionString);

try
{
    // Use connection
}
finally
{
    connection.Dispose();
}
```

---

# Interview Answer (30 Seconds)

> `IDisposable` is an interface used to release unmanaged or expensive resources such as file handles, database connections, streams, and sockets. It defines a single method, `Dispose()`, which is called to clean up resources immediately instead of waiting for the Garbage Collector. In C#, we typically use the `using` statement to ensure `Dispose()` is called automatically, even if an exception occurs.

---

# Quick Interview Points

* ✅ `IDisposable` has **one method**: `Dispose()`.
* ✅ Used to release unmanaged or expensive resources.
* ✅ `using` automatically calls `Dispose()`.
* ✅ Garbage Collector frees **memory**, while `Dispose()` frees **external resources**.
* ✅ Commonly used with `DbContext`, streams, and database connections.

