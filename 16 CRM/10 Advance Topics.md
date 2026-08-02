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


---
This is one of the **most important Dynamics 365 CRM interview questions** for 5–10 years of experience. Interviewers often ask you to explain the complete security model from the basics to advanced concepts.

---

# Dynamics 365 Security Model Overview

The Dynamics 365 security model controls:

* Who can access data
* What operations they can perform
* Which records they can access
* Which fields they can view or edit
* Which apps/features they can use

The security model consists of:

```text
                User
                  │
                  ▼
          Business Unit
                  │
                  ▼
          Security Roles
                  │
                  ▼
            Privileges
                  │
                  ▼
        Record-Level Access
                  │
                  ▼
      Field-Level Security
                  │
                  ▼
        Teams & Sharing
```

---

# 1. Business Units (BU)

Business Units divide the organization into logical groups.

Example

```text
Company

├── India BU

│     ├── Sales

│     └── Support

├── USA BU

│     ├── Sales

│     └── Finance

└── UK BU
```

Each user belongs to **one Business Unit**.

Business Units help define data boundaries.

Example

```
Pankaj → India BU

John → USA BU
```

---

# 2. Users

Every employee is created as a **System User**.

Example

```
Pankaj

↓

Sales Executive

↓

India BU
```

Users receive permissions through:

* Security Roles
* Teams
* Sharing

---

# 3. Security Roles

Security Roles define what a user can do.

Examples:

* Salesperson
* Sales Manager
* CSR
* Customer Service Manager
* System Administrator

Example

```
Pankaj

↓

Salesperson Role
```

A user can have multiple roles.

```
Pankaj

↓

Salesperson

Sales Manager

Custom Role
```

Permissions are cumulative (union of all assigned roles).

---

# 4. Privileges

A Security Role contains **Privileges**.

Common privileges:

```
Create

Read

Write

Delete

Append

Append To

Assign

Share
```

Example

| Entity  | Create | Read | Write | Delete |
| ------- | ------ | ---- | ----- | ------ |
| Account | Yes    | Yes  | Yes   | No     |
| Contact | Yes    | Yes  | No    | No     |

---

# 5. Access Levels

Each privilege has an **Access Level**.

There are five access levels.

## User

Only records owned by the current user.

```
Pankaj

↓

Own Accounts Only
```

---

## Business Unit

Records owned by anyone in the same Business Unit.

```
India BU

↓

Pankaj

Rahul

Sneha
```

All users can access each other's records (subject to privileges).

---

## Parent-Child Business Unit

Access records in the current BU and child BUs.

```
India

│

├── Mumbai

├── Pune

└── Hyderabad
```

Manager can access all child BUs.

---

## Organization

Access all records.

```
Entire CRM
```

Used for:

* Administrators
* Executives

---

## None

No access.

---

# Access Level Diagram

```text
None

↓

User

↓

Business Unit

↓

Parent:Child BU

↓

Organization
```

---

# 6. Record Ownership

Every record has an owner.

Example

```
Account

↓

Owner

Pankaj
```

Ownership determines whether a user can access the record based on their security role.

---

# 7. Teams

Instead of assigning permissions individually:

```
Sales Team

↓

Security Role

↓

Members
```

Benefits:

* Easier administration
* Shared access
* Better maintainability

Types:

* Owner Team
* Access Team

---

# Owner Team

Can own records.

```
Sales Team

↓

Owns Account
```

---

# Access Team

Cannot own records.

Used only for sharing.

---

# 8. Record Sharing

Suppose Rahul owns an Account.

```
Rahul

↓

Account
```

Rahul shares it with Pankaj.

```
Rahul

↓

Share

↓

Pankaj
```

Pankaj can now:

* Read
* Write
* Append

depending on the permissions granted.

---

# 9. Field-Level Security

Some fields are highly sensitive.

Example

```
Salary

Credit Card

SSN

Bank Account
```

Create a **Field Security Profile**.

```
HR Team

↓

Salary

Read

Write
```

Sales users

```
Salary

↓

No Access
```

Even if they can open the record, the field remains hidden or read-only.

---

# Example

Employee Record

| Name | Salary   |
| ---- | -------- |
| John | 1,00,000 |

Sales User sees:

| Name | Salary |
| ---- | ------ |
| John | *****  |

HR sees:

| Name | Salary   |
| ---- | -------- |
| John | 1,00,000 |

---

# 10. Hierarchy Security

Managers can access subordinate records.

```
Manager

├── Employee A

├── Employee B

└── Employee C
```

Useful for sales organizations.

---

# 11. Position Hierarchy

Based on company positions.

```
CEO

↓

Director

↓

Manager

↓

Executive
```

Higher positions inherit access to lower positions (if enabled).

---

# 12. Plugin Security

Plugins respect Dataverse security.

```csharp
IOrganizationService service =
    factory.CreateOrganizationService(context.UserId);
```

Uses the current user's permissions.

Impersonation:

```csharp
factory.CreateOrganizationService(otherUserId);
```

Runs using another user's permissions.

---

# Real Project Example

Requirement:

Only Sales Managers can approve discounts above 20%.

```
Opportunity

↓

Discount = 25%

↓

Plugin

↓

Check User Role

↓

Sales Manager?

↓

Yes

Approve

↓

No

Throw Exception
```

---

# Example Security Design

```
Company

↓

Business Unit

↓

User

↓

Security Role

↓

Privileges

↓

Field Security

↓

Teams

↓

Sharing
```

---

# Best Practices

* Grant the **least privilege** required.
* Use Teams instead of assigning permissions individually.
* Use Field-Level Security for sensitive columns.
* Avoid assigning System Administrator unless necessary.
* Prefer Security Roles and privileges over hardcoded authorization in plugins.
* Use record sharing only when required, not as the primary security model.

---

# Interview Answer (3–5 Minutes)

> The Dynamics 365 security model is based on **Business Units, Security Roles, Privileges, Access Levels, Record Ownership, Teams, Sharing, and Field-Level Security**. Every user belongs to a Business Unit and is assigned one or more Security Roles. A Security Role contains privileges such as Create, Read, Write, Delete, Append, Assign, and Share, each with an access level like User, Business Unit, Parent-Child Business Unit, or Organization. Records are typically secured through ownership, but access can also be granted using Owner Teams, Access Teams, or record sharing. For protecting sensitive data within a record, Dynamics provides **Field-Level Security**, where Field Security Profiles control who can read, update, or create values for specific fields. Plugins respect these security settings by default and execute under the configured user context, unless impersonation is explicitly used. This layered model provides secure, scalable, and flexible access control across the organization.
