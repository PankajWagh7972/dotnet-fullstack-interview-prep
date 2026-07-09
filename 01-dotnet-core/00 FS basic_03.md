
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

