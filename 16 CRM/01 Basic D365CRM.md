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
