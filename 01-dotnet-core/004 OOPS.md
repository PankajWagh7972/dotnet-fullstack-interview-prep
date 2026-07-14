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
