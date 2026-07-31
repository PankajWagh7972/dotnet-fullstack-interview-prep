## Short Answer

**Yes, but only for the `Retrieve` and `RetrieveMultiple` messages.**

A plugin **can** execute when a user or application retrieves records from Dataverse.

Typical read operations are:

* `Retrieve` → Read a single record.
* `RetrieveMultiple` → Read multiple records (Views, Advanced Find, Subgrids, Web API queries).

---

# Supported Read Messages

| Message            | Description               | Example                             |
| ------------------ | ------------------------- | ----------------------------------- |
| `Retrieve`         | Retrieve one record       | Opening an Account form             |
| `RetrieveMultiple` | Retrieve multiple records | Opening a view, grid, Advanced Find |

---

# Example 1: Retrieve Plugin

When a user opens an Account record.

```text
User Opens Account

↓

Retrieve

↓

Plugin Executes

↓

Account Returned
```

Plugin Registration:

```
Message:
Retrieve

Primary Entity:
account

Stage:
PreOperation or PostOperation
```

Example:

```csharp
public class RetrieveAccountPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context = (IPluginExecutionContext)
            serviceProvider.GetService(typeof(IPluginExecutionContext));

        var tracing = (ITracingService)
            serviceProvider.GetService(typeof(ITracingService));

        tracing.Trace("Account retrieved.");
    }
}
```

---

# Example 2: RetrieveMultiple Plugin

When a user opens Active Accounts.

```text
User Opens View

↓

RetrieveMultiple

↓

Plugin Executes

↓

Records Returned
```

Example:

```csharp
using Microsoft.Xrm.Sdk.Query;

public class RetrieveMultiplePlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context = (IPluginExecutionContext)
            serviceProvider.GetService(typeof(IPluginExecutionContext));

        if (context.InputParameters["Query"] is QueryExpression query)
        {
            // Add filter
            query.Criteria.AddCondition(
                "statecode",
                ConditionOperator.Equal,
                0);
        }
    }
}
```

---

# Real World Example

Suppose HR users should only see employees from their own department.

Instead of changing every view,

```text
RetrieveMultiple

↓

Plugin

↓

Add Department Filter

↓

Return Records
```

This enforces the filter centrally.

---

# Common Use Cases

### 1. Add Dynamic Filters

```text
RetrieveMultiple

↓

Add Region Filter

↓

Return Records
```

---

### 2. Mask Sensitive Data

Example:

```
Salary

100000

↓

*****

```

(PostOperation)

---

### 3. Audit Reads

Log who viewed confidential records.

```
Retrieve

↓

Plugin

↓

Audit Entity
```

---

### 4. Modify Query

Example:

```csharp
query.Criteria.AddCondition(
    "ownerid",
    ConditionOperator.Equal,
    context.UserId);
```

Now users only retrieve their own records.

---

# Can We Modify the Returned Data?

**Yes**, in **PostOperation**.

Example:

```csharp
Entity entity =
    (Entity)context.OutputParameters["BusinessEntity"];

entity["new_displayname"] = "Modified by Plugin";
```

Or for RetrieveMultiple:

```csharp
EntityCollection collection =
    (EntityCollection)context.OutputParameters["BusinessEntityCollection"];

foreach (var entity in collection.Entities)
{
    entity["new_status"] = "Visible";
}
```

---

# Performance Considerations

Be careful!

`RetrieveMultiple` runs **very frequently**:

* Opening Views
* Subgrids
* Lookups
* Dashboards
* Web API Queries
* Advanced Find

Poorly written plugins here can significantly slow down the entire application.

**Best practices:**

* Avoid extra Dataverse queries inside `RetrieveMultiple`.
* Keep logic lightweight.
* Avoid external HTTP/API calls.
* Filter only when necessary.

---

# Interview Answer (2 Minutes)

> Yes, plugins can run on read operations by registering them on the `Retrieve` and `RetrieveMultiple` messages. `Retrieve` is triggered when a single record is read, while `RetrieveMultiple` is triggered when multiple records are queried, such as opening a view or subgrid. These plugins are commonly used to add dynamic query filters, enforce custom security, audit record access, or modify returned data. Because `RetrieveMultiple` executes very frequently, it's important to keep the logic lightweight and avoid expensive database queries or external service calls to prevent performance issues.
