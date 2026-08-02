This is a very common interview question.

The method is:

```csharp
IOrganizationService service =
    factory.CreateOrganizationService(userId);
```

The behavior depends on **what you pass**.

---

# Case 1: Pass `context.UserId` (Most Common)

```csharp
IOrganizationService service =
    factory.CreateOrganizationService(context.UserId);
```

The plugin executes using the **current execution user**.

If the user has permission:

```
Create Contact
```

✅ Success

If not:

```
Access Denied
```

Example:

```text
Sales User
    │
Plugin
    │
Create Invoice
    │
❌ Sales User has no Create privilege
```

---

# Case 2: Pass `null`

```csharp
IOrganizationService service =
    factory.CreateOrganizationService(null);
```

**This is where many developers get confused.**

In a **plugin**, passing `null` **does NOT magically bypass security**.

It creates the service using the **same user context that the plugin itself is configured to run under**.

That means:

* If the plugin step is registered as **Calling User**, `null` behaves like `context.UserId`.
* If the plugin step is registered to run as a **Specific User**, `null` uses that configured user.

Example:

```
Plugin Registration

Run in User's Context

Calling User
```

Then:

```csharp
factory.CreateOrganizationService(null);
```

is effectively the same as

```csharp
factory.CreateOrganizationService(context.UserId);
```

---

Example 2:

```
Plugin Registration

Run in User's Context

CRM Service Account
```

Then

```csharp
factory.CreateOrganizationService(null);
```

runs as

```
CRM Service Account
```

---

# Case 3: Pass Another User's GUID

```csharp
Guid managerId = new Guid("...");

IOrganizationService service =
    factory.CreateOrganizationService(managerId);
```

Now every CRUD operation executes as that user.

```
Sales User

↓

Plugin

↓

Run as Manager

↓

Update Account
```

This is called **impersonation**.

---

# Summary

| Code                                                  | Executes As                                                                   |
| ----------------------------------------------------- | ----------------------------------------------------------------------------- |
| `CreateOrganizationService(context.UserId)`           | Current execution user                                                        |
| `CreateOrganizationService(context.InitiatingUserId)` | Original user who initiated the operation (if different due to impersonation) |
| `CreateOrganizationService(null)`                     | The plugin's configured execution user (calling user or configured step user) |
| `CreateOrganizationService(otherUserGuid)`            | The specified user (impersonation)                                            |

---

# Interview Answer

> If I don't pass `context.UserId` and instead pass `null`, the service does **not** ignore security or automatically become a system administrator. In a plugin, `null` creates the service using the **plugin's configured execution context**. If the plugin step runs as the calling user, it behaves the same as `context.UserId`. If the step is configured to run as a specific user, it uses that user's security context. If I need to explicitly impersonate another user, I pass that user's GUID to `CreateOrganizationService()`.
