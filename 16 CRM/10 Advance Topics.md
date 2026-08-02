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

---
This is an **advanced Dynamics 365 / Dataverse plugin interview question**.

The interviewer expects you to know **how to retrieve entity definitions, attribute metadata, option sets (Choices), relationships, and other schema information** at runtime.

---

# Short Answer

> Metadata is queried in a plugin using the **Dataverse Metadata API** through `IOrganizationService.Execute()`. Common requests include `RetrieveEntityRequest`, `RetrieveAttributeRequest`, `RetrieveOptionSetRequest`, and `RetrieveMetadataChangesRequest`.

---

# What is Metadata?

Metadata describes the structure of Dataverse tables.

Examples:

* Entity (Table) Name
* Display Name
* Columns (Attributes)
* Data Types
* Choice (Option Set) Values
* Relationships
* Primary Name Column
* Primary Key
* Alternate Keys

Example

```text
Entity : account

Display Name : Account

Primary Name : name

Primary Key : accountid
```

---

# Metadata API Flow

```text
Plugin

↓

IOrganizationService.Execute()

↓

Metadata Request

↓

Metadata Response
```

---

# 1. Retrieve Entity Metadata

Suppose you want information about the **Account** table.

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

EntityMetadata entityMetadata = response.EntityMetadata;

tracing.Trace(entityMetadata.DisplayName.UserLocalizedLabel.Label);
tracing.Trace(entityMetadata.PrimaryIdAttribute);
tracing.Trace(entityMetadata.PrimaryNameAttribute);
```

Output

```text
Account

accountid

name
```

---

# 2. Retrieve All Attributes

```csharp
RetrieveEntityRequest request = new RetrieveEntityRequest
{
    LogicalName = "account",
    EntityFilters = EntityFilters.Attributes
};

RetrieveEntityResponse response =
    (RetrieveEntityResponse)service.Execute(request);

foreach (AttributeMetadata attribute in response.EntityMetadata.Attributes)
{
    tracing.Trace(attribute.LogicalName);
}
```

Output

```text
name

telephone1

emailaddress1

websiteurl
```

---

# 3. Retrieve One Attribute

Example:

Retrieve metadata for **telephone1**

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

tracing.Trace(attribute.DisplayName.UserLocalizedLabel.Label);
```

Output

```text
Main Phone
```

---

# 4. Retrieve Choice (Option Set) Metadata

Suppose

```text
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

StatusAttributeMetadata metadata =
    (StatusAttributeMetadata)response.AttributeMetadata;

foreach (OptionMetadata option in metadata.OptionSet.Options)
{
    tracing.Trace(
        $"{option.Value} - {option.Label.UserLocalizedLabel.Label}");
}
```

Output

```text
1 - Active

2 - Inactive
```

---

# 5. Retrieve Global Choice (Global Option Set)

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

# 6. Retrieve Relationships

```csharp
RetrieveEntityRequest request =
    new RetrieveEntityRequest
{
    LogicalName = "account",
    EntityFilters = EntityFilters.Relationships
};

RetrieveEntityResponse response =
    (RetrieveEntityResponse)service.Execute(request);

foreach (var relation in response.EntityMetadata.OneToManyRelationships)
{
    tracing.Trace(relation.SchemaName);
}
```

Example Output

```text
account_primary_contact

account_contacts
```

---

# 7. Retrieve Metadata Changes

Useful for applications that cache metadata and want only incremental updates.

```csharp
RetrieveMetadataChangesRequest request =
    new RetrieveMetadataChangesRequest();

// Configure query as needed

RetrieveMetadataChangesResponse response =
    (RetrieveMetadataChangesResponse)service.Execute(request);
```

---

# Real Project Example

### Requirement

Create a generic audit plugin.

Instead of storing

```text
statuscode = 1
```

Retrieve metadata.

```text
1

↓

Active
```

Audit

```text
Status changed to Active
```

instead of

```text
Status changed to 1
```

---

# Performance Best Practices

❌ Don't retrieve metadata on every plugin execution.

Metadata rarely changes.

Instead:

* Retrieve only what you need.
* Cache metadata where appropriate (keeping in mind sandbox plugin instances may be recycled).
* Avoid `RetrieveAllEntitiesRequest` inside plugins because it is expensive.

---

# Common Metadata Requests

| Request                          | Purpose                                             |
| -------------------------------- | --------------------------------------------------- |
| `RetrieveEntityRequest`          | Entity metadata                                     |
| `RetrieveAttributeRequest`       | Single column metadata                              |
| `RetrieveOptionSetRequest`       | Global Choice metadata                              |
| `RetrieveMetadataChangesRequest` | Incremental metadata updates                        |
| `RetrieveAllEntitiesRequest`     | All entity metadata (rarely recommended in plugins) |

---

# Interview Answer (2 Minutes)

> To query metadata in a plugin, I use the Dataverse Metadata API through `IOrganizationService.Execute()`. Depending on the requirement, I use `RetrieveEntityRequest` to get entity information, `RetrieveAttributeRequest` for column metadata, `RetrieveOptionSetRequest` for global choice metadata, and `RetrieveMetadataChangesRequest` for tracking schema changes. This allows me to retrieve logical names, display names, data types, relationships, and option labels dynamically. Since metadata changes infrequently, I avoid querying it on every execution and instead retrieve only the required metadata or cache it where appropriate to improve performance.
