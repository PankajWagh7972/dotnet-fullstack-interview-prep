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
---

This is an advanced **Dynamics 365 / Dataverse plugin interview** question.

The interviewer wants to know whether you know how to retrieve **entity, attribute, option set, and relationship metadata** using the Dataverse SDK.

---

# Short Answer

> Metadata in a plugin is queried using the **Metadata API** through `IOrganizationService.Execute()`. The SDK provides requests such as `RetrieveEntityRequest`, `RetrieveAttributeRequest`, `RetrieveOptionSetRequest`, and `RetrieveMetadataChangesRequest` to retrieve entity definitions, attributes, option sets, relationships, and other metadata.

---

# Metadata Examples

Metadata includes:

* Entity Logical Name
* Display Name
* Attributes
* Primary Key
* Primary Name Field
* Option Sets (Choices)
* Relationships (1:N, N:N)
* Alternate Keys

Example:

```text
Account

Logical Name : account

Display Name : Account

Primary Id : accountid

Primary Name : name
```

---

# Architecture

```text
Plugin

↓

IOrganizationService

↓

RetrieveEntityRequest

↓

Metadata

↓

EntityMetadata
```

---

# Example 1: Retrieve Entity Metadata

```csharp
using Microsoft.Xrm.Sdk.Messages;
using Microsoft.Xrm.Sdk.Metadata;

RetrieveEntityRequest request = new RetrieveEntityRequest
{
    LogicalName = "account",
    EntityFilters = EntityFilters.Entity
};

RetrieveEntityResponse response =
    (RetrieveEntityResponse)service.Execute(request);

EntityMetadata metadata = response.EntityMetadata;

Console.WriteLine(metadata.DisplayName.UserLocalizedLabel.Label);
Console.WriteLine(metadata.PrimaryNameAttribute);
Console.WriteLine(metadata.PrimaryIdAttribute);
```

Output

```
Account

name

accountid
```

---

# Example 2: Retrieve All Attributes

```csharp
RetrieveEntityRequest request =
    new RetrieveEntityRequest
{
    LogicalName = "account",
    EntityFilters = EntityFilters.Attributes
};

RetrieveEntityResponse response =
    (RetrieveEntityResponse)service.Execute(request);

foreach(AttributeMetadata attribute
    in response.EntityMetadata.Attributes)
{
    Console.WriteLine(attribute.LogicalName);
}
```

Output

```
name

telephone1

emailaddress1

websiteurl
```

---

# Example 3: Retrieve One Attribute

```csharp
RetrieveAttributeRequest request =
    new RetrieveAttributeRequest
{
    EntityLogicalName = "account",
    LogicalName = "telephone1"
};

RetrieveAttributeResponse response =
    (RetrieveAttributeResponse)service.Execute(request);

AttributeMetadata attribute =
    response.AttributeMetadata;

Console.WriteLine(attribute.DisplayName.UserLocalizedLabel.Label);
```

Output

```
Main Phone
```

---

# Example 4: Retrieve Choice (Option Set) Metadata

Suppose

```
Status

1 = Active

2 = Inactive
```

```csharp
RetrieveAttributeRequest request =
    new RetrieveAttributeRequest
{
    EntityLogicalName = "account",
    LogicalName = "statuscode"
};

RetrieveAttributeResponse response =
    (RetrieveAttributeResponse)service.Execute(request);

StatusAttributeMetadata status =
    (StatusAttributeMetadata)response.AttributeMetadata;

foreach(var option in status.OptionSet.Options)
{
    Console.WriteLine(
        $"{option.Value} - {option.Label.UserLocalizedLabel.Label}");
}
```

Output

```
1 - Active

2 - Inactive
```

---

# Example 5: Retrieve Relationships

```csharp
RetrieveEntityRequest request =
    new RetrieveEntityRequest
{
    LogicalName = "account",
    EntityFilters = EntityFilters.Relationships
};

RetrieveEntityResponse response =
    (RetrieveEntityResponse)service.Execute(request);

foreach(var relationship
    in response.EntityMetadata.ManyToOneRelationships)
{
    Console.WriteLine(relationship.SchemaName);
}
```

Output

```
account_primary_contact

account_parent_account
```

---

# Example 6: Global Choice (Global Option Set)

```csharp
RetrieveOptionSetRequest request =
    new RetrieveOptionSetRequest
{
    Name = "new_Country"
};

RetrieveOptionSetResponse response =
    (RetrieveOptionSetResponse)service.Execute(request);

OptionSetMetadata optionSet =
    (OptionSetMetadata)response.OptionSetMetadata;
```

---

# Real Project Example

Suppose you're building a generic audit plugin.

Instead of hardcoding labels,

```
statuscode = 1
```

Retrieve metadata.

```
1

↓

Active
```

Now the audit log stores

```
Status changed to Active
```

instead of

```
Status changed to 1
```

---

# Performance Considerations

Metadata changes very rarely.

❌ Don't query metadata on every plugin execution.

Instead:

* Cache metadata in memory (for long-running external applications).
* Cache in Azure Cache/Redis (for external services).
* Retrieve only the metadata you need.

> **Note:** In sandbox plugins, in-memory caching is limited because plugin instances can be recycled. Avoid expensive metadata lookups on every execution if possible.

---

# Common Metadata Requests

| Request                          | Purpose                                                  |
| -------------------------------- | -------------------------------------------------------- |
| `RetrieveEntityRequest`          | Entity metadata                                          |
| `RetrieveAttributeRequest`       | Single attribute metadata                                |
| `RetrieveOptionSetRequest`       | Global choice metadata                                   |
| `RetrieveMetadataChangesRequest` | Incremental metadata changes                             |
| `RetrieveAllEntitiesRequest`     | All entity metadata (rarely used in plugins due to cost) |

---

# Interview Answer (2 Minutes)

> Metadata in a plugin is queried using the Dataverse Metadata API through `IOrganizationService.Execute()`. For example, I use `RetrieveEntityRequest` to retrieve entity definitions, `RetrieveAttributeRequest` for attribute details, `RetrieveOptionSetRequest` for global choice metadata, and `RetrieveMetadataChangesRequest` when tracking metadata updates. This allows me to access information such as display names, logical names, option set labels, relationships, and primary attributes. Since metadata changes infrequently, I avoid querying it on every plugin execution and instead retrieve only the required metadata or cache it where appropriate.


---
This is an **advanced Dynamics 365 interview question** related to the **early-bound API (`OrganizationServiceContext`)** and **optimistic concurrency**.

## Short Answer

> `OrganizationServiceContext.SaveChanges()` submits all tracked changes to Dataverse. By default, it **does not automatically protect against concurrent updates** using RowVersion checks. If another user updates the same record before `SaveChanges()` is called, the last update generally wins unless you explicitly implement optimistic concurrency using `RowVersion` and `UpdateRequest` with `ConcurrencyBehavior.IfRowVersionMatches`.

---

# What is `OrganizationServiceContext`?

`OrganizationServiceContext` is the early-bound equivalent of Entity Framework's `DbContext`.

```text
OrganizationServiceContext

↓

Tracks Entity Changes

↓

SaveChanges()

↓

Dataverse
```

It keeps track of entities you retrieve and modifies them when you call `SaveChanges()`.

---

# Example

```csharp
using (var context = new OrganizationServiceContext(service))
{
    Account account = context.CreateQuery<Account>()
        .First(a => a.AccountId == accountId);

    account.Name = "ABC Technologies";

    context.UpdateObject(account);

    context.SaveChanges();
}
```

`SaveChanges()` sends the pending changes to Dataverse.

---

# What Happens During Concurrency?

Suppose:

```text
10:00

User A retrieves Account

Name = ABC
```

At the same time:

```text
10:01

User B updates

Name = XYZ
```

Then:

```text
10:02

User A calls SaveChanges()

Name = ABC Technologies
```

Without optimistic concurrency:

```text
Final Value

ABC Technologies
```

User B's update is overwritten (**last writer wins**).

---

# Does `SaveChanges()` Check RowVersion?

**No, not by default.**

It doesn't automatically include the record's `RowVersion` or perform an `IfRowVersionMatches` check.

To enforce optimistic concurrency, you should use `UpdateRequest` (or another request that supports concurrency behavior) and set:

```csharp
ConcurrencyBehavior = ConcurrencyBehavior.IfRowVersionMatches;
```

---

# SaveChanges Options

You can control how `SaveChanges()` behaves using `SaveChangesOptions`.

```csharp
context.SaveChanges(SaveChangesOptions.None);
```

Other options include:

```csharp
SaveChangesOptions.ContinueOnError

SaveChangesOptions.BatchWithSingleChangeset
```

These options affect batching and error handling—not concurrency checking.

---

# Example: Batch Save

```csharp
context.UpdateObject(account);

context.UpdateObject(contact);

context.SaveChanges(
    SaveChangesOptions.BatchWithSingleChangeset);
```

Both updates are sent together in a single batch request.

---

# How to Handle Concurrency Properly

Instead of relying on `SaveChanges()` alone:

```csharp
Entity account = service.Retrieve(
    "account",
    accountId,
    new ColumnSet("name"));

string rowVersion = account.RowVersion;

account["name"] = "ABC";

UpdateRequest request = new UpdateRequest
{
    Target = account,
    ConcurrencyBehavior =
        ConcurrencyBehavior.IfRowVersionMatches
};

service.Execute(request);
```

If another user has modified the record, Dataverse throws a concurrency exception instead of overwriting the data.

---

# Real Project Example

### Banking Application

Suppose two users edit the same loan.

```text
Loan

↓

User A

↓

Change Interest Rate

------------------------

Loan

↓

User B

↓

Change Loan Amount
```

If both call `SaveChanges()` without optimistic concurrency:

```text
Last Save Wins
```

One user's changes may be lost.

Using `RowVersion` prevents this by detecting that the record changed between retrieval and update.

---

# Interview Answer (2 Minutes)

> `OrganizationServiceContext.SaveChanges()` persists all tracked entity changes to Dataverse, similar to Entity Framework. By default, it does **not** perform optimistic concurrency checks using `RowVersion`, so if multiple users update the same record, the last update can overwrite previous changes. `SaveChangesOptions` control batching and error handling but do not enable concurrency protection. When concurrency control is required, I use `UpdateRequest` with `ConcurrencyBehavior.IfRowVersionMatches` and the record's `RowVersion` so Dataverse detects conflicting updates and throws an exception instead of silently overwriting another user's changes.
---
This is one of the **most important Dynamics 365 interview questions** for experienced developers.

The interviewer expects you to know how plugins respect **Dataverse security**, **impersonation**, and **business-level authorization**.

---

# Short Answer

> Plugins execute under a security context and respect Dataverse security roles and privileges. Security can be controlled by running the plugin as the **calling user** or a **specific user (impersonation)**. Within the plugin, authorization can be enforced using security roles, privileges, team membership, business units, or record ownership. Sensitive data should never be hardcoded, and external resources should be secured using Azure AD, Managed Identity, or Azure Key Vault where applicable.

---

# Security Architecture

```text
                User

                  │

          Security Roles

                  │

                  ▼

            Plugin Executes

                  │

      IOrganizationService

                  │

        Dataverse Security

                  │

                  ▼

            Create / Read /
            Update / Delete
```

---

# 1. Execute as Calling User (Default)

By default, plugins use the security context of the user who triggered them.

```csharp
IOrganizationService service =
    factory.CreateOrganizationService(context.UserId);
```

Example

```text
Sales User

↓

Plugin

↓

Create Invoice

↓

❌ Access Denied
```

If the Sales user lacks permission, the operation fails.

---

# 2. Impersonation

Run operations as another user.

```csharp
Guid serviceAccountId = new Guid("USER_GUID");

IOrganizationService service =
    factory.CreateOrganizationService(serviceAccountId);
```

Example

```text
Sales User

↓

Plugin

↓

Run as Service Account

↓

Create Invoice

↓

Success
```

Use impersonation sparingly and grant the service account only the permissions it needs.

---

# 3. Check Security Roles

Retrieve the user's assigned roles.

```csharp
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
```

Example

```csharp
bool isManager =
    roles.Entities.Any(r =>
        r.GetAttributeValue<string>("name") ==
        "Sales Manager");

if (!isManager)
{
    throw new InvalidPluginExecutionException(
        "Only Sales Managers can approve.");
}
```

> Prefer checking privileges or using Dataverse security where possible rather than hardcoding role names.

---

# 4. Validate Record Ownership

Sometimes only the record owner should update it.

```csharp
Entity account = service.Retrieve(
    "account",
    accountId,
    new ColumnSet("ownerid"));

EntityReference owner =
    account.GetAttributeValue<EntityReference>("ownerid");

if (owner.Id != context.UserId)
{
    throw new InvalidPluginExecutionException(
        "Only the owner can modify this record.");
}
```

---

# 5. Team-Based Access

Many organizations assign permissions through teams.

```text
User

↓

Sales Team

↓

Security Role

↓

Plugin
```

The plugin can query team membership before allowing certain operations.

---

# 6. Business Unit Validation

Example:

```text
India Sales

↓

Can Update

---------------------

US Sales

↓

Cannot Update
```

The plugin can compare the user's Business Unit with the record's Business Unit before proceeding.

---

# 7. Field-Level Security

Some fields are protected by Field Security Profiles.

Example:

```text
Salary

↓

Field Security

↓

Only HR
```

If the executing user doesn't have permission, Dataverse prevents access.

The plugin should not attempt to bypass field-level security unless intentionally running under an authorized service account.

---

# 8. Secure Configuration

Don't hardcode

```text
API Keys

Connection Strings

Passwords
```

Instead use:

* Environment Variables (non-sensitive configuration)
* Secure Plugin Configuration
* Azure Key Vault (preferred for secrets)

---

# 9. Secure External APIs

Instead of

```text
Plugin

↓

Password

↓

API
```

Use

```text
Plugin

↓

Azure AD Token

↓

API
```

or Managed Identity (when supported by the hosting environment).

---

# 10. Audit Sensitive Operations

```text
Plugin

↓

Update Credit Limit

↓

Create Audit Record

↓

Log User
```

This provides traceability for important business actions.

---

# Real Project Example

### Requirement

Only Sales Managers can approve discounts greater than **20%**.

```text
Update Opportunity

↓

Plugin

↓

Discount > 20%

↓

Check User Privilege/Role

↓

Yes

↓

Approve

↓

No

↓

Throw Exception
```

---

# Security Best Practices

✅ Respect Dataverse security roles and privileges.

✅ Keep plugins running as the calling user unless elevation is required.

✅ Use impersonation only when necessary.

✅ Follow the principle of least privilege for service accounts.

✅ Avoid hardcoding secrets.

✅ Use Environment Variables, Secure Configuration, or Azure Key Vault.

✅ Log and audit sensitive operations.

---

# Common Interview Follow-Up

**Q: Can a plugin bypass Dataverse security?**

**Answer:**

Normally **no**. A plugin runs under its configured execution context (calling user or a configured user). It cannot arbitrarily bypass Dataverse security. If it executes under a highly privileged service account (through step configuration or impersonation), then the operations performed through that context use **that account's** permissions.

---

# Interview Answer (2 Minutes)

> Security in Dynamics 365 plugins is primarily enforced by Dataverse. Plugins usually execute under the calling user's security context, so all CRUD operations respect that user's roles and privileges. If business requirements require elevated permissions, I use impersonation or configure the plugin step to run as a dedicated service account, following the principle of least privilege. For business authorization, I may validate security roles, team membership, record ownership, or business unit membership before allowing an operation. I also avoid hardcoding secrets by using Environment Variables, Secure Plugin Configuration, or Azure Key Vault, and I audit sensitive operations to provide traceability. This approach keeps plugins secure while aligning with Dataverse's built-in security model.


This is one of the **most commonly asked Dynamics 365 plugin interview questions**.

## Short Answer

> In an **Update** plugin, the **previous (old) value** is obtained from the **Pre Entity Image**, while the **new value** comes from the **Target** entity (in Pre-Operation) or the **Post Entity Image** (in Post-Operation). Since the `Target` only contains changed attributes, it's a best practice to compare the `Target` with the `Pre Image`.

---

# Execution Flow

```text
User Updates Account

Old Name : ABC Ltd

        │

        ▼

Pre Image
Name = ABC Ltd

        │

        ▼

Target
Name = ABC Technologies

        │

        ▼

Database Update

        │

        ▼

Post Image
Name = ABC Technologies
```

---

# Step 1: Register Images

In the **Plugin Registration Tool**, register:

### Pre Image

```text
Alias:
PreImage

Attributes:
name
telephone1
emailaddress1
creditlimit
```

### Post Image

```text
Alias:
PostImage
```

---

# Step 2: Access Previous Value (Pre Image)

```csharp
Entity preImage = context.PreEntityImages["PreImage"];

string oldName = preImage.GetAttributeValue<string>("name");

tracing.Trace($"Old Name: {oldName}");
```

Output:

```text
Old Name: ABC Ltd
```

---

# Step 3: Access New Value (Target)

```csharp
Entity target = (Entity)context.InputParameters["Target"];

if (target.Contains("name"))
{
    string newName = target.GetAttributeValue<string>("name");

    tracing.Trace($"New Name: {newName}");
}
```

Output:

```text
New Name: ABC Technologies
```

---

# Complete Plugin Example

```csharp
using Microsoft.Xrm.Sdk;

public class CompareFieldPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context = (IPluginExecutionContext)
            serviceProvider.GetService(typeof(IPluginExecutionContext));

        var tracing = (ITracingService)
            serviceProvider.GetService(typeof(ITracingService));

        if (!(context.InputParameters.Contains("Target") &&
              context.InputParameters["Target"] is Entity target))
        {
            return;
        }

        if (!context.PreEntityImages.Contains("PreImage"))
            return;

        Entity preImage = context.PreEntityImages["PreImage"];

        string oldName = preImage.GetAttributeValue<string>("name");

        string newName = target.Contains("name")
            ? target.GetAttributeValue<string>("name")
            : oldName;

        if (oldName != newName)
        {
            tracing.Trace($"Name changed from '{oldName}' to '{newName}'");
        }
    }
}
```

---

# Access Final Value (Post Image)

If you need the value **after Dataverse has saved the record**, use the Post Image.

```csharp
Entity postImage = context.PostEntityImages["PostImage"];

string updatedName =
    postImage.GetAttributeValue<string>("name");
```

---

# Real-World Example 1: Credit Limit Change

Customer credit limit changes.

```text
Before

Credit Limit = 10000

↓

User Updates

↓

Credit Limit = 15000

↓

Plugin

↓

Notify Finance
```

```csharp
Money oldLimit =
    preImage.GetAttributeValue<Money>("creditlimit");

Money newLimit = target.Contains("creditlimit")
    ? target.GetAttributeValue<Money>("creditlimit")
    : oldLimit;

if (oldLimit.Value != newLimit.Value)
{
    tracing.Trace(
        $"Credit Limit changed from {oldLimit.Value} to {newLimit.Value}");

    // Send Email
    // Create Audit Record
}
```

---

# Real-World Example 2: Status Change

```text
Status

Active

↓

Inactive

↓

Plugin

↓

Create Audit Log
```

```csharp
OptionSetValue oldStatus =
    preImage.GetAttributeValue<OptionSetValue>("statuscode");

OptionSetValue newStatus = target.Contains("statuscode")
    ? target.GetAttributeValue<OptionSetValue>("statuscode")
    : oldStatus;

if (oldStatus.Value != newStatus.Value)
{
    tracing.Trace("Status Changed");
}
```

---

# Target vs Pre Image vs Post Image

| Source         | Contains             | Use Case               |
| -------------- | -------------------- | ---------------------- |
| **Target**     | Only changed fields  | New values before save |
| **Pre Image**  | Record before update | Previous values        |
| **Post Image** | Record after update  | Final persisted values |

---

# Best Practices

* ✅ Register **only the required attributes** in Pre/Post Images.
* ✅ Always check `target.Contains("fieldname")` before accessing a field.
* ✅ Use **Pre Images** instead of `service.Retrieve()` to avoid extra database calls.
* ✅ Use **Post Images** when you need the final saved values after all processing.
* ✅ Compare old and new values before executing business logic to avoid unnecessary processing.

---
This is an important interview question because many developers misunderstand **Pre Images**.

## Short Answer

> A **Pre Image** contains the **values of the record before the operation starts**, but **only for the attributes you selected during plugin registration**. It does **not** automatically contain all columns.

---

# Example

Suppose the Account record in Dataverse is:

| Field        | Value                                     |
| ------------ | ----------------------------------------- |
| Name         | ABC Ltd                                   |
| Telephone    | 9876543210                                |
| Email        | [abc@company.com](mailto:abc@company.com) |
| Credit Limit | 10000                                     |
| Address      | Pune                                      |

The user changes:

```text
Name = ABC Technologies
```

---

## Plugin Registration

You configure the Pre Image as:

```text
Alias : PreImage

Attributes:
name
telephone1
creditlimit
```

---

## Pre Image Contains

```csharp
Entity preImage = context.PreEntityImages["PreImage"];
```

Available values:

```text
name         = ABC Ltd
telephone1   = 9876543210
creditlimit  = 10000
```

It **does not contain**:

```text
emailaddress1
address1_city
```

because they were **not registered** in the image.

---

# Target Contains

The `Target` entity contains **only changed attributes**.

```text
Target

name = ABC Technologies
```

It does **not** contain:

```text
telephone1
creditlimit
```

unless those fields were also changed.

---

# Post Image

After the update:

```text
Post Image

name         = ABC Technologies
telephone1   = 9876543210
creditlimit  = 10000
```

Again, **only the attributes configured in the Post Image** are available.

---

# Example Code

```csharp
Entity preImage = context.PreEntityImages["PreImage"];

string oldName =
    preImage.GetAttributeValue<string>("name");

string oldPhone =
    preImage.GetAttributeValue<string>("telephone1");

Money oldCredit =
    preImage.GetAttributeValue<Money>("creditlimit");
```

---

# If You Didn't Register a Field

Suppose you try:

```csharp
string email =
    preImage.GetAttributeValue<string>("emailaddress1");
```

It returns:

* `null` (if using `GetAttributeValue<T>()` and the attribute is missing), or
* `false` from `preImage.Contains("emailaddress1")`.

That's why you should always check:

```csharp
if (preImage.Contains("emailaddress1"))
{
    string email =
        preImage.GetAttributeValue<string>("emailaddress1");
}
```

---

# Can We Register All Attributes?

Yes.

Plugin Registration Tool:

```text
Pre Image

Attributes

All Attributes
```

or

```text
*
```

depending on the tool.

But this is **not recommended**.

### Why?

Because it:

* Increases memory usage
* Makes the execution context larger
* Slightly impacts performance

It's better to register only the attributes your plugin actually needs.

---

# Best Practice

Suppose your plugin compares:

* Name
* Credit Limit
* Status

Register only:

```text
name
creditlimit
statuscode
```

instead of:

```text
All Attributes
```

---

# Interview Answer

> A **Pre Image** contains the values of the record **before** the operation executes, but **only for the attributes configured in the plugin registration**. It does not automatically include every field from the entity. If I need old values for `name`, `creditlimit`, and `statuscode`, I register only those attributes in the Pre Image. This improves performance and avoids unnecessary data being loaded into the plugin execution context.

# Interview Answer (2 Minutes)

> In an Update plugin, I use the **Pre Entity Image** to get the previous value of a field and the **Target** entity to get the new value being submitted. Since the Target only contains modified attributes, I always check `target.Contains("fieldname")` before reading it. If I need the final value after the update has been committed, I use the **Post Entity Image**. Using entity images is more efficient than calling `Retrieve()` because it avoids additional database queries and improves plugin performance.

Yes. **Plugins can create, update, delete, or associate child records** using `IOrganizationService`. This is one of the most common use cases for Dynamics 365 plugins.

---

# Short Answer

> Yes. A plugin can create, update, delete, or associate related (child) records by using `IOrganizationService.Create()`, `Update()`, `Delete()`, or `Associate()`. If the plugin is synchronous, these operations participate in the same Dataverse transaction, so if the plugin fails, all changes are rolled back.

---

# Example 1: Create Child Record

### Scenario

When an **Account** is created, automatically create a **Contact**.

```
Account
   │
   └── Contact
```

### Plugin

```csharp
public class AccountCreatePlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context = (IPluginExecutionContext)
            serviceProvider.GetService(typeof(IPluginExecutionContext));

        var factory = (IOrganizationServiceFactory)
            serviceProvider.GetService(typeof(IOrganizationServiceFactory));

        var service = factory.CreateOrganizationService(context.UserId);

        Entity account = (Entity)context.InputParameters["Target"];

        // Create child contact
        Entity contact = new Entity("contact");

        contact["firstname"] = "John";
        contact["lastname"] = "Doe";

        // Associate with Account
        contact["parentcustomerid"] = account.ToEntityReference();

        service.Create(contact);
    }
}
```

---

# Example 2: Update Child Records

### Scenario

If an Account's phone number changes, update all related Contacts.

```
Account

Phone Changed

      │

      ▼

Update All Contacts
```

```csharp
QueryExpression query = new QueryExpression("contact");

query.ColumnSet = new ColumnSet("telephone1");

query.Criteria.AddCondition(
    "parentcustomerid",
    ConditionOperator.Equal,
    accountId);

EntityCollection contacts =
    service.RetrieveMultiple(query);

foreach (Entity contact in contacts.Entities)
{
    contact["telephone1"] = newPhone;

    service.Update(contact);
}
```

---

# Example 3: Delete Child Records

```csharp
foreach (Entity contact in contacts.Entities)
{
    service.Delete("contact", contact.Id);
}
```

---

# Example 4: Create Multiple Child Records

Suppose:

```
Opportunity Won

↓

Create

Invoice

Invoice Lines

Tasks

Activities
```

```csharp
Entity invoice = new Entity("invoice");
Guid invoiceId = service.Create(invoice);

for(int i=1;i<=5;i++)
{
    Entity line = new Entity("invoicedetail");

    line["invoiceid"] =
        new EntityReference("invoice", invoiceId);

    line["quantity"] = i;

    service.Create(line);
}
```

---

# Transaction Behavior

For a **synchronous** plugin:

```
Create Account

↓

Plugin

↓

Create Contact

↓

Create Task

↓

Update Opportunity

↓

Success
```

All operations succeed together.

If one fails:

```
Create Account

↓

Plugin

↓

Create Contact

↓

❌ Exception

↓

Rollback

↓

Account Not Created

↓

Contact Not Created
```

Everything is rolled back.

---

# Avoid Infinite Loops

Suppose:

```
Account Plugin

↓

Updates Contact

↓

Contact Plugin

↓

Updates Account

↓

Account Plugin

↓

...
```

This creates an infinite loop.

Prevent it:

```csharp
if (context.Depth > 1)
{
    return;
}
```

---

# Performance Considerations

### Avoid

```text
For each Contact

↓

Retrieve()

↓

Update()
```

for thousands of records in a synchronous plugin.

Instead:

* Keep synchronous plugins lightweight.
* Use **ExecuteMultipleRequest** (for external applications; note it's not recommended inside plugins due to transaction and performance considerations).
* Offload heavy processing to **Azure Service Bus**, **Azure Functions**, or **Power Automate**.

---

# Real Project Example

### Requirement

When an Opportunity is Won:

```
Opportunity

↓

Plugin

↓

Create Invoice

↓

Create Invoice Lines

↓

Create Follow-up Task

↓

Send Email
```

Implementation:

```csharp
// Create Invoice
Guid invoiceId = service.Create(invoice);

// Create Invoice Line
service.Create(invoiceLine);

// Create Task
service.Create(task);
```

All of these are child or related records created from a single plugin execution.

---

# Best Practices

* ✅ Use `EntityReference` to link child records to the parent.
* ✅ Handle exceptions so the transaction behaves predictably.
* ✅ Prevent recursion using `context.Depth`.
* ✅ Avoid processing thousands of child records synchronously.
* ✅ Retrieve only the columns you need with `ColumnSet`.

---

# Interview Answer (2 Minutes)

> Yes, plugins can create, update, delete, or associate child records using `IOrganizationService`. A common example is creating a Contact when an Account is created or updating related Contacts when an Account changes. In a synchronous plugin, these operations participate in the same Dataverse transaction, so if any operation fails, the entire transaction is rolled back. While implementing this, I ensure I prevent recursive execution using `context.Depth`, retrieve only the required columns, and avoid large-scale child record processing inside synchronous plugins. For heavy workloads, I prefer asynchronous processing or Azure-based integration services.

This is a **very important Dynamics 365 interview question** for experienced developers.

The interviewer wants to know how Dataverse transactions work and what best practices you follow.

---

# Short Answer

> The recommended approach is to **let Dataverse manage the transaction**. Synchronous **Pre-Operation** and **Post-Operation** plugins automatically participate in the same database transaction. If an exception is thrown, Dataverse rolls back the entire transaction. Developers should **not** manually manage SQL transactions or use `TransactionScope` inside plugins.

---

# Transaction Flow

```text
User Creates Account
        │
        ▼
Pre-Operation Plugin
        │
        ▼
Core Dataverse Operation
        │
        ▼
Post-Operation Plugin
        │
        ▼
Commit Transaction
```

If any step fails:

```text
User Creates Account
        │
        ▼
Pre-Operation Plugin
        │
        ▼
Create Contact
        │
        ▼
❌ Exception
        │
        ▼
Rollback
        │
        ▼
Account NOT Created
Contact NOT Created
```

Everything is rolled back.

---

# Example

Suppose your plugin creates:

* Account
* Contact
* Task

```text
Account

↓

Plugin

↓

Create Contact

↓

Create Task

↓

Commit
```

Code

```csharp
Entity contact = new Entity("contact");
service.Create(contact);

Entity task = new Entity("task");
service.Create(task);
```

If

```csharp
throw new InvalidPluginExecutionException("Error");
```

Then

```text
Account ❌

Contact ❌

Task ❌
```

Nothing is saved.

---

# Don't Use TransactionScope

❌ Avoid this:

```csharp
using (TransactionScope scope = new TransactionScope())
{
    service.Create(entity);

    scope.Complete();
}
```

Why?

* Dataverse already manages the transaction.
* Nested or distributed transactions are **not supported** in plugins.
* It can lead to runtime errors or unexpected behavior.

---

# Synchronous vs Asynchronous

## Synchronous Plugin

```text
User

↓

Plugin

↓

Dataverse Transaction

↓

Commit
```

* Runs inside the Dataverse transaction.
* Exception = rollback.
* User waits for completion.

---

## Asynchronous Plugin

```text
User

↓

Record Saved

↓

Transaction Completed

↓

Async Plugin

↓

Background Processing
```

* Runs **after** the main transaction has committed.
* Cannot roll back the original Create/Update/Delete.
* Best for notifications, integrations, emails, etc.

---

# When to Use Synchronous

Use when you need:

* Validation
* Business rules
* Calculations
* Default values
* Related record creation that must succeed with the parent
* Atomic operations

Example:

```text
Create Order

↓

Create Order Items

↓

Commit Together
```

---

# When to Use Asynchronous

Use when you need:

* Send email
* Call REST API
* Azure Service Bus
* Power Automate
* Long-running operations
* Integrations

Example:

```text
Opportunity Won

↓

Save Opportunity

↓

Async Plugin

↓

Send SAP

↓

Send Email

↓

Generate PDF
```

---

# Real Project Example

### Requirement

When an Order is created:

* Create Invoice
* Create Invoice Lines
* Update Inventory

```text
Order

↓

Plugin

↓

Invoice

↓

Invoice Line

↓

Inventory

↓

Commit
```

If inventory update fails:

```text
Order ❌

Invoice ❌

Invoice Line ❌

Inventory ❌
```

Everything is rolled back automatically.

---

# Best Practices

### ✅ Let Dataverse manage the transaction

Don't implement your own transaction logic.

---

### ✅ Keep synchronous plugins fast

Avoid:

* Thousands of updates
* Long loops
* Slow HTTP calls
* Large file processing

---

### ✅ Throw exceptions for business validation

```csharp
if (creditLimit < amount)
{
    throw new InvalidPluginExecutionException(
        "Credit limit exceeded.");
}
```

Dataverse rolls back automatically.

---

### ✅ Use asynchronous plugins for external systems

Instead of:

```text
Plugin

↓

SAP

↓

Oracle

↓

Blob

↓

Email
```

Use:

```text
Plugin

↓

Azure Service Bus

↓

Azure Function

↓

SAP
```

This improves performance and reliability.

---

### ✅ Avoid recursion

```csharp
if (context.Depth > 1)
{
    return;
}
```

---

### ✅ Minimize database operations

Retrieve only the columns you need:

```csharp
ColumnSet columns = new ColumnSet("name", "telephone1");
```

instead of:

```csharp
new ColumnSet(true);
```

---

# Common Interview Follow-Up

### Q: Can I roll back an asynchronous plugin?

**Answer:**

No.

The main transaction has already been committed.

Only synchronous plugins participate in the Dataverse transaction.

---

### Q: Can I commit half the records?

**Answer:**

No.

A synchronous plugin participates in a single transaction.

Either:

```text
Everything Commits
```

or

```text
Everything Rolls Back
```

---

# Interview Answer (2 Minutes)

> The recommended approach is to rely on Dataverse's built-in transaction management. Synchronous Pre-Operation and Post-Operation plugins automatically execute within the same transaction as the core operation. If the plugin throws an `InvalidPluginExecutionException` or another unhandled exception, Dataverse rolls back all changes made during that transaction. I avoid using `TransactionScope` or manual transaction management because Dataverse already handles transaction boundaries. I also keep synchronous plugins lightweight and move long-running tasks such as external integrations, email notifications, or document generation to asynchronous plugins, Azure Functions, or Service Bus to avoid blocking users and reduce the risk of timeouts.
