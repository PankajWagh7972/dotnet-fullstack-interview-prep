Based on the **MS Dynamics 365 Plugin Developer** job description you uploaded, these are the most likely interview questions you'll face. The role focuses on **Dynamics 365 CRM, C#, .NET Framework, Plugins, Azure Functions, Logic Apps, Dataverse, PowerApps, Power Automate, Web API, and FetchXML**. 

Since you already have around **5+ years** of experience, expect scenario-based and architecture-level questions.

---

# 1. Explain Dynamics 365 Architecture.

### Answer

Dynamics 365 consists of:

* Client Layer (Model Driven App/Portal)
* Business Logic

  * Plugins
  * Workflows
  * Custom Actions
  * Power Automate
* Dataverse
* Integration Layer

  * Web API
  * Azure Functions
  * Logic Apps
  * Service Bus

Flow:

```
User
   ↓
Model Driven App
   ↓
Dataverse
   ↓
Plugin
   ↓
Azure Function
   ↓
External API
```

---

# 2. What is Plugin in Dynamics 365?

### Answer

A Plugin is custom C# code executed when an event occurs in Dynamics.

Example:

* Account Created
* Contact Updated
* Opportunity Won

Plugins implement

```csharp
IPlugin
```

Example

```csharp
public class AccountPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {

    }
}
```

---

# 3. Difference between Plugin and Workflow?

| Plugin               | Workflow              |
| -------------------- | --------------------- |
| C# Code              | No code / Low code    |
| Faster               | Slower                |
| Complex Logic        | Simple Business Logic |
| Can execute Pre/Post | Mostly Post           |
| Full Control         | Limited               |

---

# 4. What are Plugin Execution Stages?

There are 4 stages.

### Pre Validation

Before security checks.

Use:

Validation

Duplicate Check

---

### Pre Operation

Inside Transaction

Use

Modify values before saving

```
Target["name"]="Updated";
```

---

### Post Operation

After database commit

Use

Send Email

Call API

Create Related Records

---

### Async

Background execution

Good for

Notifications

External APIs

Heavy Processing

---

# 5. Difference between Synchronous and Asynchronous Plugin?

### Synchronous

Runs immediately

User waits

Transaction rollback possible

Example

```
Validate Data
```

---

### Asynchronous

Runs in background

User doesn't wait

Cannot block save

Example

```
Send Email
```

---

# 6. Explain Plugin Execution Pipeline.

```
User Saves Record
      ↓
Pre Validation
      ↓
Pre Operation
      ↓
Database Save
      ↓
Post Operation
      ↓
Async Plugin
```

---

# 7. How do you access Target Entity?

```csharp
Entity entity =
(Entity)context.InputParameters["Target"];
```

---

# 8. What is Execution Context?

Contains

* Message Name

* User

* Primary Entity

* Input Parameters

* Depth

* Images

Example

```csharp
context.MessageName
```

```
Create

Update

Delete
```

---

# 9. What are Images?

Images store entity values.

Types

### Pre Image

Old values

### Post Image

New values

Example

```
Status changed?
```

Compare

```
PreImage

PostImage
```

---

# 10. Why use Images instead of Retrieve?

Because

* Faster

* Less Database Calls

* Better Performance

---

# 11. What is Depth?

Depth prevents infinite recursion.

Example

Plugin updates same record.

Plugin triggers again.

Depth

```
1

2

3

```

Check

```csharp
if(context.Depth >1)
return;
```

---

# 12. How do you avoid Infinite Loop?

```csharp
if(context.Depth>1)
return;
```

or

Update only required fields.

---

# 13. Difference between Organization Service and Web API?

Organization Service

SOAP

Older

Strongly typed

Plugin usage

---

Web API

REST

JSON

Modern Integration

Used by

React

Angular

PowerApps

---

# 14. What is FetchXML?

FetchXML is XML query language for Dynamics.

Example

```xml
<fetch>
   <entity name='account'>
      <attribute name='name'/>
   </entity>
</fetch>
```

---

# 15. Difference between FetchXML and QueryExpression?

FetchXML

Supports Aggregation

Reports

Advanced Find

Complex Joins

---

QueryExpression

C#

Compile Time

Easy Coding

---

# 16. Explain Dataverse.

Dataverse is cloud database for Dynamics.

Stores

Accounts

Contacts

Opportunity

Lead

Custom Tables

Security

Business Rules

Auditing

---

# 17. Difference between Entity and Table?

Old Name

Entity

New Name

Table

Same thing.

---

# 18. How do you retrieve records?

```csharp
QueryExpression query =
new QueryExpression("account");

query.ColumnSet =
new ColumnSet("name");

EntityCollection records =
service.RetrieveMultiple(query);
```

---

# 19. Difference between Retrieve and RetrieveMultiple?

Retrieve

Single Record

---

RetrieveMultiple

Many Records

---

# 20. Explain OptionSet.

Choice field

Example

Status

```
Open

Won

Lost
```

---

# 21. What are Alternate Keys?

Unique Key

Example

Customer Number

Instead of GUID

---

# 22. How do you call External API from Plugin?

Don't call directly in synchronous plugin.

Better

Plugin

↓

Azure Service Bus

↓

Azure Function

↓

External API

Improves performance.

---

# 23. Why Azure Function?

Serverless

Scalable

Cost Effective

Easy Integration

Retry

Queue Trigger

HTTP Trigger

---

# 24. Logic App vs Azure Function

Logic App

Low Code

Connectors

Business Integration

---

Azure Function

C#

Coding

Complex Logic

---

# 25. What are Secure and Unsecure Configurations?

**Secure Configuration**

* Visible only to admins
* Stores secrets like API keys

**Unsecure Configuration**

* Visible to all users who can access plugin registration
* Stores non-sensitive values

---

# 26. What is Impersonation?

Running plugin using another user's security context.

```csharp
serviceFactory.CreateOrganizationService(userId);
```

---

# 27. What is Shared Variables?

Pass data between plugins in same pipeline.

```csharp
context.SharedVariables["Key"] = "Value";
```

---

# 28. Scenario: Update Account when Contact is Created

**Solution:**

1. Register plugin on Contact Create
2. Get Parent Account
3. Update Account
4. Check `Depth`
5. Use Pre/Post Images where appropriate

---

# 29. Scenario: Send Data to External ERP

**Recommended Architecture:**

```
Dynamics Plugin
      ↓
Azure Service Bus
      ↓
Azure Function
      ↓
ERP API
```

Benefits:

* Reliable retries
* Loose coupling
* Better scalability
* Prevents CRM performance issues

---

# 30. Scenario: Duplicate Record Validation

Register plugin on **PreValidation**.

```csharp
RetrieveMultiple()

if(found)
{
   throw new InvalidPluginExecutionException(
      "Duplicate Record");
}
```

---

# 31. What is the difference between Power Automate and Plugin?

| Plugin                      | Power Automate           |
| --------------------------- | ------------------------ |
| C#                          | Low Code                 |
| Fast                        | Slower                   |
| Runs inside CRM transaction | Runs outside transaction |
| Complex logic               | Business automation      |
| Supports rollback           | No transaction rollback  |

---

# 32. How do you debug a Plugin?

* Use **Plugin Registration Tool Profiler**
* Capture execution profile
* Replay in Visual Studio
* Use tracing with `ITracingService`
* Review Plugin Trace Logs in Dynamics

Example:

```csharp
tracing.Trace("Plugin execution started");
```

---

# 33. Explain Plugin Registration.

You register:

* Message (Create, Update, Delete, Assign)
* Primary Entity
* Stage (PreValidation, PreOperation, PostOperation)
* Mode (Sync/Async)
* Filtering Attributes (for Update)

---

# 34. Why use Filtering Attributes?

For Update plugins, execute only when specific fields change.

Example:

```
name
emailaddress1
telephone1
```

This avoids unnecessary executions and improves performance.

---

# 35. What are common Plugin best practices?

* Check `context.Depth`
* Validate `Target` entity
* Use Images instead of extra retrieves
* Minimize database calls
* Register filtering attributes
* Avoid long-running synchronous operations
* Use Azure Functions/Service Bus for integrations
* Log using `ITracingService`
* Handle exceptions with meaningful messages
* Keep plugins focused on a single responsibility

---

These questions cover almost all the primary skills listed in the job description and are highly relevant for a **6+ years MS Dynamics 365 Plugin Developer** interview. 


For a **6+ years MS Dynamics 365 CRM Developer**, interviewers usually don't ask basic plugin questions. They focus on **architecture, performance, security, scalability, Azure integration, Dataverse internals, and real-world scenarios**.

Below are advanced interview questions with detailed answers that are commonly asked by companies like Microsoft, Deloitte, Hitachi, Capgemini, EY, Cognizant, and Cloudangles.

---

# 1. Explain the complete Plugin Execution Pipeline in Detail.

### Answer

Plugin execution occurs in four stages:

```
User Request
      │
      ▼
Pre Validation
      │
      ▼
Security Validation
      │
      ▼
Pre Operation
      │
      ▼
SQL Transaction Starts
      │
      ▼
Database Operation
      │
      ▼
Post Operation
      │
      ▼
Transaction Commit
      │
      ▼
Async Service
```

### PreValidation

* Executes before security validation.
* Outside SQL transaction.
* Best place for validation.
* Throwing exception here avoids transaction cost.

Example:

```
Duplicate Customer Check

Mandatory Business Rules

Prevent Delete
```

---

### PreOperation

Inside SQL Transaction.

Use cases:

* Update Target entity
* Set calculated fields
* Modify values before save

```
Target["fullname"] =
firstname + lastname;
```

No extra Update() required.

---

### PostOperation

Runs after database save.

Good for

* Create child records
* Trigger integrations
* Send notifications

---

### Async Plugin

Runs through Async Service.

Used for

* Email
* External API
* ERP Integration

---

## Interview Follow-up

**Q:** Why should duplicate validation happen in PreValidation instead of PostOperation?

### Answer

Because:

* No SQL Transaction
* Better Performance
* Saves DB rollback cost

---

# 2. Explain Plugin Transaction Behavior.

Every synchronous plugin shares same SQL transaction.

```
Create Account

↓

Plugin

↓

Create Contact

↓

Exception

↓

Everything Rollback
```

Even Account won't be created.

---

Async Plugin

Different Transaction

No rollback.

---

# 3. How does Dataverse Transaction work?

Dataverse uses SQL transaction internally.

```
Plugin
    ↓
Create Account
    ↓
Create Contact
    ↓
Update Opportunity
```

If

```
Update Opportunity

fails
```

Entire transaction rolls back.

---

# 4. Explain Optimistic Concurrency.

Suppose

```
User A
```

opens Account.

At same time

```
User B
```

updates Account.

Now

User A saves.

Without concurrency

User B changes lost.

With Row Version

CRM detects conflict.

Returns

```
412 Precondition Failed
```

---

# 5. How do you improve Plugin Performance?

Interview favorite.

Answer:

Avoid

```
RetrieveMultiple()

inside loop
```

Instead

```
Retrieve once

Store Dictionary
```

Avoid

```
Update()

inside loop
```

Use

```
ExecuteMultiple
```

Avoid

```
All Columns
```

Retrieve

```
Specific Columns
```

Avoid

```
External API
```

inside Sync Plugin.

Use

```
Azure Service Bus

↓

Azure Function
```

---

# 6. Difference between ExecuteMultiple and ExecuteTransaction?

### ExecuteMultiple

```
100 Requests

↓

Executed independently
```

One fails

Others continue.

---

### ExecuteTransaction

```
100 Requests

↓

Single Transaction
```

One fails

Everything rollback.

---

# 7. When would you use ExecuteTransaction?

Example

Insurance Policy Creation

Need

```
Account

Policy

Premium

Invoice
```

If Invoice fails

Everything rollback.

---

# 8. What is ExecuteMultiple best practice?

Wrong

```
for each customer

Create()
```

500 SQL Calls

Correct

```
ExecuteMultipleRequest
```

One Batch

Much Faster

---

# 9. Explain Service Protection Limits.

Dynamics prevents excessive API usage.

Limits include

* API calls
* Concurrent requests
* Execution time

If exceeded

```
429

Too Many Requests
```

Use

Retry Policy

Exponential Backoff

---

# 10. What is Throttling?

If application continuously hits Web API

```
1000+

requests
```

CRM throttles.

Returns

```
429
```

Retry after delay.

---

# 11. Explain Sandbox Isolation.

Plugins run in

```
Sandbox
```

Restricted environment.

Cannot

* Access registry
* Write disk
* Execute EXE
* Access local files

Reason

Security

---

# 12. Difference between Sandbox and Full Trust?

| Sandbox                   | Full Trust             |
| ------------------------- | ---------------------- |
| Online                    | On-Premise             |
| Secure                    | Less Restricted        |
| Limited Resources         | Full Server Access     |
| Cannot access File System | Can access File System |

---

# 13. Explain Plugin Isolation Mode.

Two Modes

```
Sandbox

None
```

Online

Always Sandbox.

OnPrem

Can use None.

---

# 14. Explain Secure vs Unsecure Configuration.

Secure

```
Connection String

API Key

Secret
```

Stored separately.

Only Admin.

---

Unsecure

```
Default URL

Timeout

Country
```

Visible.

---

# 15. Explain Images internally.

Suppose

```
Account

Name = ABC
```

Update

```
ABC

↓

XYZ
```

Pre Image

```
ABC
```

Post Image

```
XYZ
```

No Retrieve required.

---

# 16. Explain Shared Variables.

Scenario

Plugin A

Calculates Discount

Plugin B

Needs Discount.

Instead of Retrieve

```
SharedVariables

["Discount"]
```

Much Faster.

---

# 17. Difference between QueryExpression, FetchXML and LINQ?

| QueryExpression | FetchXML            | LINQ           |
| --------------- | ------------------- | -------------- |
| C#              | XML                 | Strongly Typed |
| Compile Time    | Runtime             | Readable       |
| No Aggregate    | Aggregate Supported | Limited        |

---

# 18. Explain Early Bound vs Late Bound.

Late Bound

```
Entity
```

Dynamic

```
entity["name"]
```

Flexible

---

Early Bound

```
Account.Name
```

Strong typing

Compile checking

Better Performance

---

# 19. Why Early Bound is faster?

No Dictionary lookup.

Compiler optimized.

Intellisense.

Compile validation.

---

# 20. Explain Metadata Cache.

CRM loads

Tables

Relationships

Fields

OptionSets

into Metadata Cache.

Plugin doesn't hit DB every time.

Improves performance.

---

# 21. Explain Alternate Keys.

Instead of GUID

```
CustomerNumber
```

can identify customer.

Useful in Integration.

---

# 22. Explain Upsert Request.

Instead of

```
Retrieve

↓

Create

↓

Update
```

Use

```
Upsert
```

Automatically

Create

or

Update.

---

# 23. Explain Azure Service Bus Integration.

Architecture

```
CRM Plugin

↓

Service Bus Queue

↓

Azure Function

↓

SAP

↓

Response
```

Benefits

* Retry
* Dead Letter Queue
* Decoupled
* High Availability

---

# 24. Why not call ERP directly from Plugin?

Bad

```
Plugin

↓

ERP

↓

20 seconds
```

User waits.

Timeout.

Good

```
Plugin

↓

Queue

↓

Azure Function

↓

ERP
```

---

# 25. Explain Idempotency.

Suppose

Queue retries.

Message

```
Create Invoice
```

received twice.

Need only

One Invoice.

Store

```
InvoiceNumber
```

Check

Already Exists

Return.

---

# 26. Explain Retry Strategy.

```
Attempt 1

↓

Fail

↓

5 sec

↓

Attempt 2

↓

Fail

↓

10 sec

↓

Attempt 3
```

Exponential Backoff.

---

# 27. Explain Dead Letter Queue.

After

```
5 Retries
```

Move Message

to

```
Dead Letter Queue
```

Developer investigates later.

---

# 28. Explain Azure Function Retry.

```
Queue Trigger

↓

Exception

↓

Automatic Retry

↓

Dead Letter
```

---

# 29. Explain Plugin Trace Log.

```
ITracingService

↓

Plugin Trace Log

↓

Dynamics
```

Used for Production Debugging.

---

# 30. Scenario: Million Accounts Import

Interview Question

How will you import?

Answer

❌ Not

```
for()
{
Create()
}
```

Use

```
ExecuteMultiple

Batch Size 500

Parallel

Retry

Logging
```

---

# 31. How would you migrate 5 million records without impacting CRM users?

### Answer

* Use Azure Data Factory or SSIS/KingswaySoft.
* Process records in batches (500–1000).
* Run during off-peak hours.
* Disable non-essential synchronous plugins/workflows during migration.
* Enable detailed logging and retry failed batches.
* Validate data after import.

---

# 32. How do you secure a Web API exposed by Azure Functions?

### Answer

* Microsoft Entra ID (Azure AD) authentication.
* OAuth 2.0 with bearer tokens.
* Managed Identity when Azure resources communicate.
* Store secrets in Azure Key Vault.
* Never hardcode client secrets in plugins.

---

# 33. Explain how to prevent plugin recursion beyond checking `Depth`.

### Answer

Besides checking `Depth`:

* Update only changed attributes.
* Use filtering attributes.
* Separate business logic across messages.
* Use `SharedVariables` to identify internally triggered operations.
* Avoid unnecessary `Update()` calls on the same record.

---

# 34. What are common causes of slow Dynamics 365 performance?

### Answer

* Retrieving all columns (`ColumnSet(true)`).
* Multiple database calls in loops.
* Long-running synchronous plugins.
* Complex FetchXML with many linked entities.
* Poor indexing on alternate keys.
* Excessive JavaScript on forms.
* Too many business rules and synchronous flows.

---

# 35. Design Question: Integrate Dynamics 365 with SAP

**Expected Architecture:**

```
Dynamics 365
      │
      ▼
Post Operation Plugin
      │
      ▼
Azure Service Bus Queue
      │
      ▼
Azure Function
      │
      ▼
SAP API
      │
      ▼
Application Insights + Retry + DLQ
```

**Why this design?**

* Non-blocking user experience.
* Reliable retries.
* Dead Letter Queue for failures.
* Scalable and loosely coupled.
* Easier monitoring with Application Insights.

---

## Interview Tips for 6+ Years

At this level, interviewers expect you to explain:

* **Why** you chose a particular design.
* **Trade-offs** (Plugin vs Power Automate vs Azure Function).
* **Performance optimizations**.
* **Scalability and resiliency**.
* **Security (OAuth, Managed Identity, Key Vault)**.
* **Real production scenarios** you've handled.

If you're targeting companies like **Microsoft, Deloitte, Capgemini, Hitachi, EY, or Cognizant**, I can also provide **50 architecture-based Dynamics 365 interview questions** and **30 coding-based plugin interview questions with complete C# implementations**.


Plugins in **Microsoft Dynamics 365 CRM (Dataverse)** are custom .NET classes that execute automatically when specific events occur in the platform. They are used to implement **server-side business logic** that runs before or after operations like Create, Update, Delete, Assign, SetState, etc.

---

# What is a Plugin?

A plugin is a **C# class library (.NET)** that implements the **IPlugin** interface.

```csharp
public class AccountPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        // Business Logic
    }
}
```

It is compiled into a DLL and registered using the **Plugin Registration Tool (PRT)**.

---

# Why do we use Plugins?

Common scenarios:

* Validate business rules
* Auto-populate fields
* Prevent invalid data
* Integrate with external systems
* Create related records
* Send notifications
* Audit changes
* Perform calculations

Example:

Whenever an Opportunity is created

```
Create Opportunity
      ↓
Plugin Executes
      ↓
Create Follow-up Task
      ↓
Send Email
```

---

# Plugin Execution Pipeline

```
Request

↓

Pre Validation

↓

Pre Operation

↓

Core Platform Operation

↓

Post Operation

↓

Response
```

---

# Plugin Stages

## 1. Pre Validation (Stage 10)

Runs **before security checks**.

Used for

* Input validation
* Business validation
* Cancel operation early

Example

```
User creates Account

↓

Check duplicate PAN Number

↓

If Exists

↓

Throw Exception
```

Code

```csharp
throw new InvalidPluginExecutionException("Duplicate PAN Number.");
```

---

## 2. Pre Operation (Stage 20)

Runs inside transaction before database save.

Best for

* Modify input data
* Set default values
* Update fields

Example

```
Create Contact

↓

Plugin

↓

Set Full Name

↓

Save
```

```csharp
Entity target = (Entity)context.InputParameters["Target"];

target["fullname"] =
target["firstname"] + " " + target["lastname"];
```

No Update() call required.

---

## 3. Main Operation (Stage 30)

Microsoft internal operation.

Cannot register plugins here.

---

## 4. Post Operation (Stage 40)

Runs after database commit.

Used for

* Create related records
* Call external APIs
* Azure Service Bus
* Azure Functions
* Email
* Logging

Example

```
Account Created

↓

Plugin

↓

Create Welcome Task

↓

Send Email
```

---

# Plugin Execution Mode

## Synchronous

```
User Click Save

↓

Plugin Executes

↓

Save Completes
```

User waits.

Used when

* Validation
* Mandatory logic
* Data manipulation

---

## Asynchronous

```
User Click Save

↓

Record Saved

↓

Background Plugin Executes
```

User does not wait.

Used for

* Email
* API Calls
* Integration
* Heavy Processing

---

# Plugin Images

Images capture entity values before or after an operation.

## Pre Image

Old values

Example

```
Status

Open

↓

Update

↓

Closed
```

Pre Image

```
Status = Open
```

---

## Post Image

New values

```
Status = Closed
```

---

Example

```csharp
Entity preImage =
context.PreEntityImages["PreImage"];

string oldName =
preImage.GetAttributeValue<string>("name");
```

---

# Input Parameters

Contains incoming request data.

```csharp
Entity entity =
(Entity)context.InputParameters["Target"];
```

---

# Output Parameters

Contains response.

Example

Retrieve

```
Retrieved Entity
```

---

# Shared Variables

Pass data between plugins.

Plugin 1

```csharp
context.SharedVariables["Discount"] = 10;
```

Plugin 2

```csharp
int discount =
(int)context.SharedVariables["Discount"];
```

---

# Execution Context

```csharp
IPluginExecutionContext context
```

Contains

* MessageName
* PrimaryEntityName
* UserId
* InitiatingUserId
* CorrelationId
* Depth
* Stage
* Mode
* ParentContext
* SharedVariables
* Images

---

# Important Context Properties

## Message Name

```csharp
context.MessageName
```

Examples

```
Create
Update
Delete
Assign
Merge
SetState
Associate
Disassociate
```

---

## Primary Entity

```csharp
context.PrimaryEntityName
```

Output

```
account
contact
lead
opportunity
```

---

## Depth

Very important.

Used to prevent infinite recursion.

Example

Plugin updates Account

↓

Update triggers Plugin again

↓

Infinite Loop

Prevent

```csharp
if (context.Depth > 1)
    return;
```

---

# Organization Service

Used for CRUD operations.

```csharp
IOrganizationServiceFactory factory =
(IOrganizationServiceFactory)
serviceProvider.GetService(typeof(IOrganizationServiceFactory));

IOrganizationService service =
factory.CreateOrganizationService(context.UserId);
```

---

Create

```csharp
service.Create(entity);
```

Retrieve

```csharp
service.Retrieve("account", id, new ColumnSet(true));
```

Update

```csharp
service.Update(entity);
```

Delete

```csharp
service.Delete("account", id);
```

---

# Tracing Service

Useful for debugging.

```csharp
ITracingService tracing =
(ITracingService)
serviceProvider.GetService(typeof(ITracingService));

tracing.Trace("Plugin Started");
```

Logs visible in Plugin Trace Logs.

---

# Exception Handling

```csharp
try
{
    // logic
}
catch(Exception ex)
{
    tracing.Trace(ex.ToString());

    throw new InvalidPluginExecutionException(
        "Unexpected Error", ex);
}
```

---

# Complete Plugin Example

```csharp
public class AccountPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        ITracingService tracing =
            (ITracingService)serviceProvider.GetService(typeof(ITracingService));

        IPluginExecutionContext context =
            (IPluginExecutionContext)serviceProvider.GetService(typeof(IPluginExecutionContext));

        if (context.Depth > 1)
            return;

        if (!context.InputParameters.Contains("Target"))
            return;

        Entity target =
            (Entity)context.InputParameters["Target"];

        if (target.LogicalName != "account")
            return;

        if (target.Contains("name"))
        {
            target["description"] =
                "Created from Plugin";
        }

        tracing.Trace("Plugin Completed.");
    }
}
```

---

# Registration Steps

1. Build DLL
2. Open Plugin Registration Tool
3. Connect to Dataverse
4. Register Assembly
5. Register Step
6. Select Message (Create/Update/Delete)
7. Select Entity
8. Select Stage
9. Select Mode
10. Register Image (if required)

---

# Real-Time Scenario

### Requirement

Whenever an Opportunity is won:

* Update Customer Status
* Create Invoice
* Send Email
* Notify ERP
* Log Activity

Pipeline

```
Opportunity Won

↓

Post Operation Plugin

↓

Update Account

↓

Create Invoice

↓

Call ERP API

↓

Queue Email

↓

Log Audit
```

For long-running integrations (ERP, emails, third-party APIs), a common pattern is:

```
Dynamics Plugin (Post Operation)

↓

Azure Service Bus

↓

Azure Function

↓

ERP / SAP / External APIs

↓

Dead Letter Queue (if retries fail)
```

This keeps the plugin fast and avoids blocking the CRM transaction.

---

# Plugin Best Practices

* Use **Pre Operation** when modifying the target entity before save.
* Use **Post Operation** for creating related records or publishing events.
* Avoid long-running operations in synchronous plugins.
* Check `context.Depth` to prevent recursion.
* Use filtering attributes on Update steps so the plugin runs only when relevant fields change.
* Use `ITracingService` for diagnostics.
* Keep plugins focused on a single responsibility.
* Use secure/unsecure configuration for configurable values instead of hardcoding.
* For external integrations, publish a message to **Azure Service Bus** instead of making lengthy HTTP calls directly from the plugin.
* Register only the necessary images and columns to reduce overhead.
* Always validate that `Target` exists and is the expected entity type.

---

# Frequently Asked Interview Questions

| Question                                            | Expected Answer                                                                                                                       |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Why use a plugin instead of JavaScript?             | Plugins run server-side, are more secure, cannot be bypassed by users, and can execute regardless of the client (web, API, import).   |
| Pre Validation vs Pre Operation?                    | Pre Validation runs before security checks and transaction; Pre Operation runs inside the transaction before the record is committed. |
| Why use `Depth`?                                    | To prevent infinite recursion when plugin logic updates records that trigger the same plugin again.                                   |
| What are Plugin Images?                             | Snapshots of entity data before (Pre Image) or after (Post Image) the operation.                                                      |
| Synchronous vs Asynchronous?                        | Synchronous blocks the user's request until complete; asynchronous runs in the background after the operation.                        |
| When should you use Azure Service Bus with plugins? | For long-running, resilient, or decoupled integrations with external systems.                                                         |
| Why use filtering attributes?                       | They prevent unnecessary plugin execution when unrelated fields are updated, improving performance.                                   |

These concepts and scenarios are commonly discussed in Dynamics 365 CRM interviews for developers with 5–10 years of experience.


These are very common Dynamics 365 interview questions. Let's cover them in detail.

---

# 1. Time Limit for Synchronous and Asynchronous Plugins

## Synchronous Plugin

* Executes immediately during the user request.
* The user waits until the plugin finishes.
* **Maximum execution time: 2 minutes (120 seconds).**

If it exceeds 2 minutes:

* Plugin is terminated.
* Transaction is rolled back.
* User receives a timeout/error.

Example:

```text
User Clicks Save

↓

Plugin Runs (Validation)

↓

Record Saved
```

Good use cases

* Validation
* Calculations
* Setting default values
* Preventing invalid data

Avoid

* External API calls
* File processing
* Long SQL queries
* Heavy loops

---

## Asynchronous Plugin

Runs in the background through the **Asynchronous Service**.

The user does not wait.

Also has a **2-minute execution limit per execution**, but because it is decoupled from the user transaction, it is suitable for longer-running background work than synchronous plugins. If the work is expected to take even longer or involve unreliable external systems, move it outside the plugin (for example, to Azure Service Bus + Azure Functions).

Example

```text
Save Account

↓

Record Saved

↓

Async Plugin Starts

↓

Call SAP API

↓

Send Email

↓

Complete
```

Good use cases

* Email
* API Calls
* Azure Service Bus
* ERP Integration
* Notifications

---

# What if processing takes more than 2 minutes?

Never do this:

```text
CRM Plugin

↓

Call SAP

↓

Wait 5 Minutes

↓

Return
```

Instead

```text
CRM Plugin

↓

Send Message

↓

Azure Service Bus

↓

Azure Function

↓

SAP

↓

Update CRM
```

This is Microsoft's recommended architecture.

---

# 2. Plugin Parameters

Plugin parameters are available through

```csharp
context.InputParameters

context.OutputParameters
```

Both are

```csharp
ParameterCollection
```

which behaves like

```csharp
Dictionary<string, object>
```

---

# Input Parameters

Contain information coming into the request.

Example

```text
User Creates Account

↓

Target Entity

↓

Plugin
```

Code

```csharp
Entity entity =
(Entity)context.InputParameters["Target"];
```

---

## Common Input Parameters

| Parameter       | Type                                                 | Used For               |
| --------------- | ---------------------------------------------------- | ---------------------- |
| Target          | Entity / EntityReference                             | Create, Update, Delete |
| Id              | Guid                                                 | Record ID              |
| Relationship    | Relationship                                         | Associate/Disassociate |
| RelatedEntities | EntityReferenceCollection                            | Associate              |
| Assignee        | EntityReference                                      | Assign Request         |
| State           | OptionSetValue                                       | SetState               |
| Status          | OptionSetValue                                       | SetState               |
| Query           | QueryExpression / FetchExpression / QueryByAttribute | RetrieveMultiple       |
| ColumnSet       | ColumnSet                                            | Retrieve               |
| BusinessEntity  | Entity                                               | Older SDK (legacy)     |

---

## Example 1

Create Account

```text
Target

↓

Entity
```

```csharp
Entity account =
(Entity)context.InputParameters["Target"];
```

---

## Example 2

Delete

```text
Target

↓

EntityReference
```

```csharp
EntityReference account =
(EntityReference)context.InputParameters["Target"];
```

Notice

Delete doesn't provide the full entity.

Only

```text
Logical Name

Id
```

---

## Example 3

Assign

```text
Assignee

↓

EntityReference(User)
```

---

# Output Parameters

Contains the response returned by CRM.

Example

Create

```text
User

↓

Create Account

↓

Plugin

↓

Account Id Returned
```

Code

```csharp
Guid id =
(Guid)context.OutputParameters["id"];
```

---

## Common Output Parameters

| Parameter                | Type               | Message          |
| ------------------------ | ------------------ | ---------------- |
| id                       | Guid               | Create           |
| BusinessEntity           | Entity             | Retrieve         |
| BusinessEntityCollection | EntityCollection   | RetrieveMultiple |
| ResponseName             | Depends on request | Custom Actions   |

---

# Shared Variables

Used for passing data between plugins in the same execution pipeline.

```csharp
context.SharedVariables["OrderTotal"] = 500;
```

Later

```csharp
decimal total =
(decimal)context.SharedVariables["OrderTotal"];
```

Type

```text
Dictionary<String,Object>
```

---

# Plugin Images

## Pre Image

Old values

Type

```text
Entity
```

```csharp
Entity pre =
context.PreEntityImages["PreImage"];
```

---

## Post Image

Updated values

Type

```text
Entity
```

```csharp
Entity post =
context.PostEntityImages["PostImage"];
```

---

# Entity Types Used in Plugins

## Entity

Contains complete record.

```csharp
Entity account = new Entity("account");
```

Example

```text
Name

Phone

Email

Revenue
```

---

## EntityReference

Contains only

```text
Logical Name

GUID

Name (optional)
```

Example

```csharp
EntityReference customer =
new EntityReference("account", accountId);
```

---

## EntityCollection

Multiple records

```csharp
EntityCollection contacts;
```

---

## OptionSetValue

Choice Field

```csharp
OptionSetValue status =
new OptionSetValue(1);
```

---

## Money

Currency field

```csharp
Money amount =
new Money(5000);
```

---

## AliasedValue

Joined query result

```csharp
AliasedValue city =
(AliasedValue)entity["contact.city"];
```

---

## ColumnSet

Columns to retrieve

```csharp
new ColumnSet("name","email");
```

---

## Relationship

Associate

```csharp
Relationship rel =
new Relationship("account_contacts");
```

---

## EntityReferenceCollection

Many records

```csharp
EntityReferenceCollection contacts;
```

---

# Most Frequently Used Parameters

| Message          | Target Type                              | Output           |
| ---------------- | ---------------------------------------- | ---------------- |
| Create           | Entity                                   | Guid (id)        |
| Update           | Entity                                   | None             |
| Delete           | EntityReference                          | None             |
| Assign           | EntityReference + Assignee               | None             |
| SetState         | EntityReference + State + Status         | None             |
| Associate        | Relationship + EntityReferenceCollection | None             |
| Disassociate     | Relationship + EntityReferenceCollection | None             |
| Retrieve         | EntityReference + ColumnSet              | Entity           |
| RetrieveMultiple | QueryExpression / FetchExpression        | EntityCollection |

---

# Interview Questions

### Q1. Why doesn't Delete have an Entity Target?

Because the record is being deleted. Dynamics only needs the record's **logical name** and **GUID**, so it passes an `EntityReference` instead of the full `Entity`.

---

### Q2. Why use Pre Image?

To access values **before** an update or delete.

Example:

```text
Name

Old = ABC Pvt Ltd

↓

Updated

↓

XYZ Pvt Ltd
```

Target only contains the changed fields, whereas the Pre Image contains the original values.

---

### Q3. Why use Post Image?

To access the record **after** the operation completes.

Example:

```text
Status

Open

↓

Won
```

The Post Image contains the updated status.

---

### Q4. What is the difference between Target and Pre Image?

| Target                                 | Pre Image                                     |
| -------------------------------------- | --------------------------------------------- |
| Contains only submitted/changed fields | Contains original values before the operation |
| Available in Create/Update             | Available only if registered and applicable   |
| Used to modify incoming data           | Used to compare old vs. new values            |

---

### Q5. What is the difference between `Entity` and `EntityReference`?

| Entity                      | EntityReference                                            |
| --------------------------- | ---------------------------------------------------------- |
| Full record with attributes | Only logical name, GUID, and optional name                 |
| Used for Create/Update      | Commonly used for Delete, lookup fields, and relationships |

These questions are frequently asked in Dynamics 365 CRM interviews because they test your understanding of the plugin execution pipeline and how data flows through it.

