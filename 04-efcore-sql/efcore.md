# Explicit Loading in EF Core (`.Collection()` / `.Reference()`)

**Explicit Loading** means you **manually load related data after the main entity has already been loaded**.

Instead of loading everything upfront with `Include()`, you decide **when** to load navigation properties.

---

# When to Use

* Load related data **only if needed**.
* Improve performance by avoiding unnecessary data retrieval.
* Useful when the decision to load related data depends on business logic.

---

## Example Entity Relationship

```text
Customer
   │
   └── Orders
```

---

# 1. Loading a Collection (`.Collection()`)

### Step 1: Load Customer

```csharp
var customer = await context.Customers
    .FirstOrDefaultAsync(c => c.Id == 1);
```

At this point:

```text
Customer   ✅ Loaded
Orders     ❌ Not Loaded
```

---

### Step 2: Load Orders Manually

```csharp
await context.Entry(customer)
    .Collection(c => c.Orders)
    .LoadAsync();
```

Now:

```text
Customer   ✅ Loaded
Orders     ✅ Loaded
```

Access:

```csharp
foreach (var order in customer.Orders)
{
    Console.WriteLine(order.Id);
}
```

---

# 2. Loading a Reference (`.Reference()`)

Suppose:

```text
Order
   │
   └── Customer
```

Load Order:

```csharp
var order = await context.Orders
    .FirstAsync(o => o.Id == 10);
```

Customer is not loaded.

Load it explicitly:

```csharp
await context.Entry(order)
    .Reference(o => o.Customer)
    .LoadAsync();
```

Now:

```text
Order      ✅ Loaded
Customer   ✅ Loaded
```

---

# Real-Time Example

Suppose you're displaying customer details.

Initially:

```csharp
var customer = await context.Customers
    .FirstAsync(c => c.Id == id);
```

Only customer information is shown.

If the user clicks **"View Orders"**, then load orders:

```csharp
await context.Entry(customer)
    .Collection(c => c.Orders)
    .LoadAsync();
```

This avoids fetching orders unless they are actually needed.

---

# `Collection()` vs `Reference()`

| Method          | Used For                                         |
| --------------- | ------------------------------------------------ |
| `.Collection()` | One-to-Many / Many-to-Many navigation properties |
| `.Reference()`  | One-to-One / Many-to-One navigation properties   |

### Example

```csharp
// Collection
await context.Entry(customer)
    .Collection(c => c.Orders)
    .LoadAsync();

// Reference
await context.Entry(order)
    .Reference(o => o.Customer)
    .LoadAsync();
```

---

# Loading with Query

You can also filter the related data before loading.

```csharp
await context.Entry(customer)
    .Collection(c => c.Orders)
    .Query()
    .Where(o => o.TotalAmount > 1000)
    .LoadAsync();
```

Only orders with `TotalAmount > 1000` are loaded.

---

# Eager vs Lazy vs Explicit Loading

| Loading Type         | When Loaded                                        | Method                           |
| -------------------- | -------------------------------------------------- | -------------------------------- |
| **Eager Loading**    | Immediately with the main query                    | `Include()` / `ThenInclude()`    |
| **Lazy Loading**     | Automatically when navigation property is accessed | Lazy Loading Proxies             |
| **Explicit Loading** | Manually after the entity is loaded                | `.Collection()` / `.Reference()` |

---

# Interview Answer (30 Seconds)

> Explicit loading in EF Core is used to manually load related entities after the main entity has already been retrieved. It uses `context.Entry(entity).Collection(...).LoadAsync()` for collection navigation properties and `context.Entry(entity).Reference(...).LoadAsync()` for reference navigation properties. It is useful when related data is needed only under certain conditions, helping improve performance by avoiding unnecessary database queries.
