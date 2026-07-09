
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
