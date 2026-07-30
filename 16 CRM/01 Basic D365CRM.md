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
