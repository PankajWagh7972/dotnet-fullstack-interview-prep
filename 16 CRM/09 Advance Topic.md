This question is about **optimistic concurrency** in Dataverse.

## Short Answer

> To prevent concurrent updates from overwriting each other, use **optimistic concurrency** with the record's **RowVersion (ETag)**. When updating a record, include the RowVersion and set the request's concurrency behavior to `IfRowVersionMatches`. If another user has modified the record in the meantime, Dataverse throws a concurrency exception instead of silently overwriting the changes.

---

# Problem

Imagine two users open the same Account.

```text
10:00 AM

User A opens Account

Balance = 1000

-----------------------------

User B opens Account

Balance = 1000
```

User A updates:

```text
Balance = 1500
```

Then User B updates:

```text
Balance = 1200
```

Final value:

```text
1200
```

User A's change is lost.

This is called the **Lost Update Problem**.

---

# Solution: Optimistic Concurrency

Dataverse stores a **RowVersion** for each record.

```text
Account

Balance = 1000

RowVersion = 345678
```

When updating, you tell Dataverse:

> "Update this record **only if** the RowVersion is still `345678`."

If another update has already changed the record, the RowVersion is different and the update fails.

---

# Step 1: Retrieve the Record

```csharp
Entity account = service.Retrieve(
    "account",
    accountId,
    new ColumnSet("name"));

string rowVersion = account.RowVersion;
```

---

# Step 2: Update Using `UpdateRequest`

```csharp
account["name"] = "ABC Technologies";

UpdateRequest request = new UpdateRequest
{
    Target = account,
    ConcurrencyBehavior = ConcurrencyBehavior.IfRowVersionMatches
};

service.Execute(request);
```

If another user updated the record first, Dataverse throws an exception.

---

# Exception Handling

```csharp
try
{
    service.Execute(request);
}
catch (FaultException<OrganizationServiceFault> ex)
{
    throw new InvalidPluginExecutionException(
        "The record has been modified by another user. Please refresh and try again.",
        ex);
}
```

---

# Flow

```text
User A                    User B

Retrieve
RowVersion = 100

                        Retrieve
                        RowVersion = 100

Update

RowVersion = 101

                        Update

                        RowVersion = 100

                        ❌ Concurrency Exception
```

---

# Real Project Example

Suppose an **Order** has an `Amount`.

Two finance users edit it simultaneously.

Without concurrency:

```text
User A → 5000

User B → 3000

Final = 3000
```

User A's update is lost.

With optimistic concurrency:

```text
User A → Success

User B → RowVersion mismatch

Show error:
"This record has been modified. Refresh and try again."
```

---

# Alternative Approaches

### 1. Optimistic Concurrency (Recommended)

* No record locking.
* Best for most Dataverse scenarios.
* Prevents accidental overwrites.

### 2. Business Logic Checks

For example, compare a timestamp or status before updating and reject the change if it has changed unexpectedly.

### 3. Transactions

Within a plugin, multiple operations can be executed in the same Dataverse transaction, but transactions **do not** prevent another user from modifying the same record between separate requests. Transactions provide atomicity, not concurrency control.

---

# Interview Answer (2 Minutes)

> The recommended approach is to use **optimistic concurrency**. Dataverse maintains a `RowVersion` for records. I retrieve the record, preserve its `RowVersion`, and perform the update using an `UpdateRequest` with `ConcurrencyBehavior.IfRowVersionMatches`. If another user has modified the record in the meantime, the RowVersion no longer matches and Dataverse throws a concurrency exception. This prevents one user's changes from silently overwriting another's. I then handle the exception by informing the user to refresh the record and retry. This approach is preferable to locking because it scales well and avoids blocking other users.


This is a **very common Dynamics 365 interview question**.

## Short Answer

> **Impersonation** means executing plugin operations as a **different Dataverse user** instead of the user who triggered the plugin. This is done by creating an `IOrganizationService` for a specific user, allowing the plugin to respect that user's security roles and permissions.

---

# Why Do We Need Impersonation?

Suppose:

* **Sales User** creates an Opportunity.
* The plugin needs to create an **Invoice**.
* Only the **Finance Team** has permission to create Invoices.

Without impersonation:

```text
Sales User

↓

Plugin

↓

Create Invoice

↓

❌ Access Denied
```

The Sales User doesn't have permission.

---

With impersonation:

```text
Sales User

↓

Plugin

↓

Run as Finance Service Account

↓

Create Invoice

↓

✅ Success
```

---

# How to Impersonate

Use `IOrganizationServiceFactory`.

```csharp
IOrganizationServiceFactory factory =
    (IOrganizationServiceFactory)
    serviceProvider.GetService(typeof(IOrganizationServiceFactory));

IOrganizationService service =
    factory.CreateOrganizationService(userId);
```

The important part is **which `userId` you pass**.

---

# Execute as Triggering User

```csharp
IOrganizationService service =
    factory.CreateOrganizationService(context.UserId);
```

Here the plugin uses the permissions of the user who triggered it.

Example:

```
Pankaj

↓

Plugin

↓

Runs as Pankaj
```

---

# Execute as SYSTEM User

```csharp
IOrganizationService service =
    factory.CreateOrganizationService(null);
```

Passing `null` creates the service using the **SYSTEM** context (for sandboxed plugins, this effectively runs under the plugin's configured execution context rather than the triggering user). This is commonly used for platform-level operations where appropriate.

---

# Execute as Another User

Suppose Finance User ID is known.

```csharp
Guid financeUserId =
    new Guid("A1234567-B123-C123-D123-123456789ABC");

IOrganizationService financeService =
    factory.CreateOrganizationService(financeUserId);
```

Now:

```csharp
Entity invoice = new Entity("invoice");
invoice["name"] = "INV-1001";

financeService.Create(invoice);
```

The Invoice is created using the Finance user's permissions.

---

# Complete Example

```csharp
public class OpportunityPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context =
            (IPluginExecutionContext)
            serviceProvider.GetService(typeof(IPluginExecutionContext));

        var factory =
            (IOrganizationServiceFactory)
            serviceProvider.GetService(typeof(IOrganizationServiceFactory));

        Guid financeUserId =
            new Guid("YOUR-FINANCE-USER-ID");

        IOrganizationService financeService =
            factory.CreateOrganizationService(financeUserId);

        Entity invoice = new Entity("invoice");

        invoice["name"] = "Invoice 1001";

        financeService.Create(invoice);
    }
}
```

---

# Plugin Registration vs Code

There are **two ways** to control execution identity.

### 1. Plugin Registration Tool (Most Common)

When registering the plugin step:

```
Run in User's Context

↓

Calling User

or

Specific User
```

If you choose **Specific User**, the entire plugin runs under that user's security context.

---

### 2. In Code

```csharp
factory.CreateOrganizationService(userId);
```

Only the operations performed through that service instance are impersonated.

---

# Real Project Example

### Requirement

Sales representatives cannot create **Contracts**.

When an Opportunity is Won:

```
Sales User

↓

Plugin

↓

Impersonate Service Account

↓

Create Contract

↓

Notify Customer
```

The Sales User doesn't need Contract permissions, but the plugin can still complete the business process.

---

# Best Practices

* Use impersonation only when there's a genuine business need.
* Prefer configuring the **Run in User's Context** setting on the plugin step when all operations should run under a single service account.
* Avoid hardcoding user GUIDs. Store them in configuration or retrieve them dynamically if needed.
* Ensure the impersonated account has only the permissions required (principle of least privilege).

---

# Interview Answer (2 Minutes)

> **Impersonation** means executing Dataverse operations under the security context of a different user than the one who triggered the plugin. It's implemented by creating an `IOrganizationService` with `IOrganizationServiceFactory.CreateOrganizationService(userId)`. The impersonated user's security roles determine what operations are allowed. This is useful when a business process requires permissions that the initiating user doesn't have, such as creating finance records or integrating with restricted entities. Another option is to configure the plugin step to run as a specific user in the Plugin Registration Tool, which applies that security context to the entire plugin execution.
