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


---
This is one of the **most frequently asked Dynamics 365 plugin interview questions**.

The key difference is **when** they execute and **whether they are inside the database transaction**.

---

# Execution Pipeline

```text
                Request Received
                       │
                       ▼
               Pre-Validation (Stage 10)
                       │
             Security Checks (mostly)
                       │
                       ▼
               Pre-Operation (Stage 20)
                       │
                Database Transaction
                       │
                       ▼
                 Core Operation
               (Create/Update/Delete)
                       │
                       ▼
              Post-Operation (Stage 40)
```

---

# Comparison

| Feature                          | Pre-Validation                      | Pre-Operation                                    |
| -------------------------------- | ----------------------------------- | ------------------------------------------------ |
| Stage                            | 10                                  | 20                                               |
| Runs before security checks      | Yes (for the initial request)       | No                                               |
| Runs inside database transaction | ❌ No                                | ✅ Yes                                            |
| Can modify `Target` entity       | ✅ Yes                               | ✅ Yes                                            |
| Can cancel the operation         | ✅ Yes                               | ✅ Yes                                            |
| Best for                         | Early validation, business rules    | Data modification, calculations, related updates |
| Performance                      | Better (no transaction started yet) | Slightly slower (inside transaction)             |

---

# Pre-Validation Plugin

### Purpose

Use it to **validate** whether the operation should continue before the transaction starts.

Examples:

* Duplicate checks
* Mandatory business rules
* Validate business hours
* Block deletion
* Check external conditions

Example:

> Don't allow deleting an Account if it has active Opportunities.

```text
Delete Account

↓

Pre-Validation

↓

Check Active Opportunities

↓

Found

↓

Throw Exception

↓

Delete Never Starts
```

Example code:

```csharp
if (activeOpportunityCount > 0)
{
    throw new InvalidPluginExecutionException(
        "Cannot delete account with active opportunities.");
}
```

Because the transaction hasn't started, you avoid unnecessary transaction overhead.

---

# Pre-Operation Plugin

### Purpose

Use it when you want to **modify the data** before it is saved.

Examples:

* Auto-generate values
* Calculate totals
* Set default fields
* Update lookup values
* Normalize data

Example

User enters

```text
Name

Pankaj
```

Plugin changes

```text
Name

PANKAJ
```

before saving.

Code

```csharp
Entity target =
    (Entity)context.InputParameters["Target"];

target["name"] =
    target.GetAttributeValue<string>("name").ToUpper();
```

No extra Update call is required because you're modifying the `Target` before it is persisted.

---

# Real Project Example 1

**Loan Approval**

Requirement

```text
Loan Amount

↓

Validate Credit Score

↓

Reject if score < 650
```

Use:

✅ **Pre-Validation**

Reason:

Don't start a transaction if the request will be rejected.

---

# Real Project Example 2

Calculate Invoice Total

```text
Invoice

↓

Calculate GST

↓

Calculate Discount

↓

Save Total
```

Use:

✅ **Pre-Operation**

Reason:

You're modifying the record before it is saved.

---

# Real Project Example 3

Prevent Duplicate Email

```text
Create Contact

↓

Pre-Validation

↓

Email Exists?

↓

Yes

↓

Throw Exception
```

No transaction is opened.

---

# Real Project Example 4

Auto Number

```text
Create Account

↓

Pre-Operation

↓

Generate Account Number

↓

Save Record
```

---

# When to Choose Which?

### Use Pre-Validation

* Validate business rules.
* Cancel invalid operations.
* Prevent unnecessary transactions.
* Check external conditions before processing.

### Use Pre-Operation

* Modify entity attributes.
* Set default values.
* Calculate fields.
* Update related values before saving.
* Ensure all changes are committed in the same transaction.

---

# Common Interview Question

**Q:** Where should I calculate a Total Amount?

**Answer:**

**Pre-Operation**, because you want the calculated value to be saved as part of the same database transaction.

---

**Q:** Where should I stop duplicate record creation?

**Answer:**

**Pre-Validation**, because it's better to reject the request before the transaction begins.

---

# Interview Answer (2 Minutes)

> **Pre-Validation** plugins execute first, before the main database transaction starts. They are ideal for validating requests, enforcing business rules, preventing duplicate records, or cancelling operations early, which avoids unnecessary transaction overhead. **Pre-Operation** plugins execute inside the database transaction, just before Dataverse performs the core operation. They are best suited for modifying the `Target` entity, calculating values, setting default fields, or making related updates that should be committed atomically with the main operation. In general, I use **Pre-Validation** for validation and **Pre-Operation** for data modification.
---
This is a common interview question for **Dynamics 365 CE Plugins**.

## Short Answer

> To access the current user's roles in a plugin, retrieve the current user's `systemuserid` from `IPluginExecutionContext.UserId`, then query the **systemuserroles** intersect table and join it with the **role** table using `QueryExpression` (or FetchXML). This returns all security roles assigned to the user.

---

# Dataverse Relationship

```text
SystemUser
     │
     │ N:N
     │
SystemUserRoles (Intersect Table)
     │
     │
Role
```

A user can have multiple roles.

Example:

```text
Pankaj

↓

Salesperson

System Administrator

Custom Manager
```

---

# Step 1: Get Current User

```csharp
Guid userId = context.UserId;
```

or

```csharp
Guid initiatingUser = context.InitiatingUserId;
```

### Difference

| Property           | Meaning                                                                          |
| ------------------ | -------------------------------------------------------------------------------- |
| `UserId`           | User under whose security context the plugin is executing (may be impersonated). |
| `InitiatingUserId` | The original user who initiated the request.                                     |

Most of the time you'll use:

```csharp
context.UserId;
```

---

# Step 2: Query User Roles

```csharp
QueryExpression query = new QueryExpression("role");

query.ColumnSet = new ColumnSet("name");

LinkEntity userRoleLink = query.AddLink(
    "systemuserroles",
    "roleid",
    "roleid");

userRoleLink.LinkCriteria.AddCondition(
    "systemuserid",
    ConditionOperator.Equal,
    context.UserId);

EntityCollection roles = service.RetrieveMultiple(query);
```

---

# Step 3: Read Role Names

```csharp
foreach (Entity role in roles.Entities)
{
    string roleName =
        role.GetAttributeValue<string>("name");

    tracing.Trace(roleName);
}
```

Output

```text
Salesperson

Sales Manager

CSR Manager
```

---

# Complete Plugin Example

```csharp
using Microsoft.Xrm.Sdk;
using Microsoft.Xrm.Sdk.Query;

public class CheckUserRolePlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context = (IPluginExecutionContext)
            serviceProvider.GetService(typeof(IPluginExecutionContext));

        var factory = (IOrganizationServiceFactory)
            serviceProvider.GetService(typeof(IOrganizationServiceFactory));

        var service =
            factory.CreateOrganizationService(context.UserId);

        var tracing =
            (ITracingService)
            serviceProvider.GetService(typeof(ITracingService));

        QueryExpression query = new QueryExpression("role");

        query.ColumnSet = new ColumnSet("name");

        LinkEntity link = query.AddLink(
            "systemuserroles",
            "roleid",
            "roleid");

        link.LinkCriteria.AddCondition(
            "systemuserid",
            ConditionOperator.Equal,
            context.UserId);

        EntityCollection roles =
            service.RetrieveMultiple(query);

        foreach (Entity role in roles.Entities)
        {
            tracing.Trace(
                $"Role : {role.GetAttributeValue<string>("name")}");
        }
    }
}
```

---

# Check if User Has a Specific Role

```csharp
bool isAdmin = roles.Entities.Any(r =>
    r.GetAttributeValue<string>("name") ==
    "System Administrator");

if (!isAdmin)
{
    throw new InvalidPluginExecutionException(
        "Only System Administrators can perform this action.");
}
```

---

# Alternative Using FetchXML

```xml
<fetch>
  <entity name="role">
    <attribute name="name"/>
    <link-entity name="systemuserroles"
                 from="roleid"
                 to="roleid">
      <filter>
        <condition attribute="systemuserid"
                   operator="eq"
                   value="{USERID}" />
      </filter>
    </link-entity>
  </entity>
</fetch>
```

```csharp
EntityCollection roles =
    service.RetrieveMultiple(
        new FetchExpression(fetchXml));
```

---

# Real Project Example

### Requirement

Only users with the **Sales Manager** role can approve discounts greater than **20%**.

```text
Update Opportunity

↓

Plugin

↓

Get Current User Roles

↓

Sales Manager?

↓

Yes

↓

Approve

↓

No

↓

Throw Exception
```

```csharp
bool isSalesManager = roles.Entities.Any(r =>
    r.GetAttributeValue<string>("name") == "Sales Manager");

if (!isSalesManager && discount > 20)
{
    throw new InvalidPluginExecutionException(
        "Only Sales Managers can approve discounts above 20%.");
}
```

---

# Best Practice

If you're checking **privileges** rather than just role names, it's generally better to rely on **Dataverse security roles and privileges** or built-in access checks instead of hardcoding role names. Role names can vary between environments or be renamed, whereas privileges remain consistent.

---

# Interview Answer (2 Minutes)

> In a plugin, I obtain the current user's ID from `IPluginExecutionContext.UserId` (or `InitiatingUserId` if I need the original caller). I then query the `role` table and join it with the `systemuserroles` intersect table using `QueryExpression` or `FetchXML` to retrieve all roles assigned to that user. I can iterate through the returned roles or check for a specific role before allowing a business operation. For authorization decisions, I prefer relying on Dataverse security privileges where possible instead of hardcoding role names, since role names can differ across environments.
---
This is a common **Dynamics 365/Power Platform ALM** interview question.

The interviewer wants to know **how you avoid hardcoding environment-specific values** such as:

* API URLs
* Azure Function URLs
* Service Bus connection strings
* Key Vault URLs
* Storage Account URLs
* Feature flags

---

# Short Answer

> Environment-specific values should **never be hardcoded** in a plug-in. The recommended approach is to use **Environment Variables** in Dataverse. Other options include **Secure/Unsecure Plugin Configuration**, **Azure Key Vault**, or **Custom Configuration Entities**, depending on the type of configuration and security requirements.

---

# Option 1 (Recommended): Environment Variables ⭐⭐⭐⭐⭐

Microsoft's recommended approach.

### Example

Development

```text
API URL

https://dev-api.company.com
```

UAT

```text
https://uat-api.company.com
```

Production

```text
https://api.company.com
```

No code changes.

---

## Architecture

```text
Solution

↓

Environment Variable

↓

Current Environment Value

↓

Plugin

↓

Call API
```

---

## Create Environment Variable

```text
Solution

↓

New

↓

Environment Variable

↓

Name

API_URL

↓

Default Value

https://dev-api.company.com
```

Each environment overrides only the value.

---

## Read Environment Variable

```csharp
QueryExpression query = new QueryExpression("environmentvariabledefinition")
{
    ColumnSet = new ColumnSet("schemaname")
};

query.Criteria.AddCondition(
    "schemaname",
    ConditionOperator.Equal,
    "API_URL");

LinkEntity valueLink = query.AddLink(
    "environmentvariablevalue",
    "environmentvariabledefinitionid",
    "environmentvariabledefinitionid",
    JoinOperator.LeftOuter);

valueLink.Columns = new ColumnSet("value");
valueLink.EntityAlias = "Value";

Entity result = service.RetrieveMultiple(query).Entities.FirstOrDefault();

string apiUrl =
    ((AliasedValue)result["Value.value"]).Value.ToString();
```

Now

```text
https://api.company.com
```

is automatically read from the current environment.

---

# Option 2: Secure/Unsecure Plugin Configuration ⭐⭐⭐⭐

Good for plugin-specific configuration.

Secure

```text
API Key

OAuth Secret

Connection String
```

Unsecure

```text
API URL

Timeout

Retry Count
```

Plugin

```csharp
public AccountPlugin(
    string unsecure,
    string secure)
{
    _apiUrl = unsecure;

    _apiKey = secure;
}
```

---

# Option 3: Azure Key Vault ⭐⭐⭐⭐⭐

Best for secrets.

```text
Plugin

↓

Key Vault

↓

Secret

↓

API Key
```

Never store

```text
Client Secret

OAuth Secret

Certificate Password
```

inside Dataverse.

---

# Option 4: Custom Configuration Table ⭐⭐⭐

Create

```text
Configuration

Name

Value
```

Example

| Name       | Value                                              |
| ---------- | -------------------------------------------------- |
| ApiUrl     | [https://api.company.com](https://api.company.com) |
| RetryCount | 5                                                  |
| Timeout    | 60                                                 |

Plugin

```csharp
QueryExpression query =
    new QueryExpression("new_configuration");

query.Criteria.AddCondition(
    "new_name",
    ConditionOperator.Equal,
    "ApiUrl");

Entity config =
    service.RetrieveMultiple(query)
    .Entities
    .FirstOrDefault();
```

Useful when business users need to update configuration without redeploying.

---

# Option 5: Azure App Configuration

Common in Azure-hosted integrations.

```text
Plugin

↓

Azure App Configuration

↓

Settings
```

Mostly used by Azure Functions and APIs rather than plugins.

---

# Which Should You Use?

| Configuration      | Best Choice                                      |
| ------------------ | ------------------------------------------------ |
| API URL            | ✅ Environment Variable                           |
| Azure Function URL | ✅ Environment Variable                           |
| Retry Count        | ✅ Environment Variable                           |
| Feature Flag       | ✅ Environment Variable                           |
| API Secret         | ✅ Azure Key Vault or Secure Plugin Configuration |
| OAuth Secret       | ✅ Azure Key Vault                                |
| Connection String  | ✅ Azure Key Vault                                |

---

# Real Project Example

Suppose your company has three environments.

```
Development

↓

https://dev-api.company.com

----------------------------

UAT

↓

https://uat-api.company.com

----------------------------

Production

↓

https://api.company.com
```

Your plugin reads the `API_URL` environment variable. During deployment, only the environment variable value changes—**the plugin assembly remains the same**, making deployments safer and easier.

---

# Best Practices

* ✅ Use **Environment Variables** for environment-specific settings like URLs and feature flags.
* ✅ Use **Secure Plugin Configuration** or **Azure Key Vault** for secrets.
* ✅ Avoid hardcoding URLs or credentials in code.
* ✅ Cache configuration values when appropriate to reduce repeated Dataverse queries.

---

# Interview Answer (2 Minutes)

> I avoid hardcoding environment-specific values in plugins. For values such as API URLs, Azure Function endpoints, or feature flags, I use **Dataverse Environment Variables**, which allow different values in Development, UAT, and Production without changing the plugin code. For sensitive information like API keys, client secrets, or connection strings, I use **Secure Plugin Configuration** or, preferably, **Azure Key Vault**. This approach follows ALM best practices, keeps deployments consistent across environments, and improves security and maintainability.


---
This is one of the **most frequently asked Dynamics 365 plugin interview questions**.

The correct answer is:

* **Previous (Old) Value** → **Pre Image**
* **New Value** → **Target Entity** (Pre-Operation) or **Post Image** (Post-Operation)

---

# Short Answer

> In an Update plugin, the previous value is obtained from the **Pre Entity Image**, while the new value comes from the **Target** entity in Pre-Operation or the **Post Entity Image** in Post-Operation. Since the `Target` only contains changed attributes, it's common to combine `Target` with the Pre Image to determine old and new values.

---

# Update Pipeline

```text
User Updates Account

Name
ABC Ltd

↓

Changes Name

ABC Technologies

↓

Pre Image
Name = ABC Ltd

↓

Target
Name = ABC Technologies

↓

Database Update

↓

Post Image
Name = ABC Technologies
```

---

# Plugin Registration

When registering the plugin, configure:

```
Pre Image

Alias:
PreImage

Attributes:
name,emailaddress1,telephone1
```

Optionally configure

```
Post Image

Alias:
PostImage
```

---

# Access Previous Value (Pre Image)

```csharp
Entity preImage =
    context.PreEntityImages["PreImage"];

string oldName =
    preImage.GetAttributeValue<string>("name");
```

Output

```
ABC Ltd
```

---

# Access New Value (Target)

```csharp
Entity target =
    (Entity)context.InputParameters["Target"];

string newName =
    target.GetAttributeValue<string>("name");
```

Output

```
ABC Technologies
```

> **Important:** The `Target` contains **only the fields that were changed**. If `name` wasn't modified, it won't be present.

---

# Complete Example

```csharp
public class AccountPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context =
            (IPluginExecutionContext)
            serviceProvider.GetService(typeof(IPluginExecutionContext));

        Entity target =
            (Entity)context.InputParameters["Target"];

        Entity preImage =
            context.PreEntityImages["PreImage"];

        string oldName =
            preImage.GetAttributeValue<string>("name");

        string newName =
            target.GetAttributeValue<string>("name");

        if(oldName != newName)
        {
            // Log changes

            // Send Email

            // Audit
        }
    }
}
```

---

# Safer Implementation

Since the `Target` may not contain every attribute:

```csharp
string oldName =
    preImage.GetAttributeValue<string>("name");

string newName = target.Contains("name")
    ? target.GetAttributeValue<string>("name")
    : oldName;
```

This avoids exceptions and correctly handles updates where `name` wasn't changed.

---

# Using Post Image

If you need the final saved values after the database update:

```csharp
Entity postImage =
    context.PostEntityImages["PostImage"];

string updatedName =
    postImage.GetAttributeValue<string>("name");
```

---

# Compare Old vs New

```csharp
if(oldName != newName)
{
    tracing.Trace(
        $"Name changed from {oldName} to {newName}");
}
```

---

# Real Project Example

### Requirement

Whenever a customer's credit limit changes:

* Store old limit
* Store new limit
* Notify Finance

```
Pre Image

Credit Limit = 10000

↓

Target

Credit Limit = 15000

↓

Plugin

↓

Audit Log

Old : 10000

New : 15000
```

Implementation:

```csharp
Money oldLimit =
    preImage.GetAttributeValue<Money>("creditlimit");

Money newLimit =
    target.GetAttributeValue<Money>("creditlimit");

if(oldLimit.Value != newLimit.Value)
{
    // Notify Finance
}
```

---

# Pre Image vs Target vs Post Image

| Source         | Contains                    | Use For                |
| -------------- | --------------------------- | ---------------------- |
| **Pre Image**  | Record values before update | Old values             |
| **Target**     | Only changed attributes     | New values being saved |
| **Post Image** | Record values after update  | Final persisted values |

---

# Best Practices

* Configure only the attributes you need in **Pre/Post Images** to reduce overhead.
* Always check `target.Contains("fieldname")` before reading from `Target`.
* Prefer **Pre Images** over calling `Retrieve()` for old values—they're faster and avoid extra database calls.
* Use **Post Images** when you need calculated values or the final saved state after the operation.

---

# Interview Answer (2 Minutes)

> In an Update plugin, I use the **Pre Entity Image** to access the previous values of fields and the **Target** entity to access the new values being submitted. Since the Target only contains attributes that have changed, I first check `target.Contains("fieldname")` before reading them. If I need the final values after the record is saved, I use the **Post Entity Image**. Using Pre and Post Images is preferable to calling `Retrieve()` because it avoids additional database calls and improves plugin performance.
