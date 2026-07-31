This is a very common Dynamics 365 interview question.

## Short Answer

> In a plugin, related records are accessed using `IOrganizationService`. Typically, you first obtain the lookup (`EntityReference`) from the current entity, then use `Retrieve`, `RetrieveMultiple`, or `QueryExpression` to fetch the related record(s). For performance, use Pre/Post Entity Images when the related data is already available instead of making unnecessary database calls.

---

# Scenario 1: Parent Lookup (Most Common)

Suppose **Contact** has a lookup to **Account** (`parentcustomerid`).

```text id="b8ktv8"
Account

↓

ABC Technologies

↓

Contact

↓

John Smith
```

When a Contact is updated, retrieve its parent Account.

### Step 1: Get Lookup

```csharp
Entity contact = (Entity)context.InputParameters["Target"];

EntityReference accountRef =
    contact.GetAttributeValue<EntityReference>("parentcustomerid");
```

### Step 2: Retrieve Parent

```csharp
Entity account = service.Retrieve(
    "account",
    accountRef.Id,
    new ColumnSet("name", "telephone1"));
```

Now

```csharp
string accountName =
    account.GetAttributeValue<string>("name");
```

---

# Complete Example

```csharp
public class ContactPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context =
            (IPluginExecutionContext)
            serviceProvider.GetService(typeof(IPluginExecutionContext));

        var factory =
            (IOrganizationServiceFactory)
            serviceProvider.GetService(typeof(IOrganizationServiceFactory));

        var service =
            factory.CreateOrganizationService(context.UserId);

        Entity contact =
            (Entity)context.InputParameters["Target"];

        if (!contact.Contains("parentcustomerid"))
            return;

        EntityReference accountRef =
            contact.GetAttributeValue<EntityReference>("parentcustomerid");

        Entity account =
            service.Retrieve(
                "account",
                accountRef.Id,
                new ColumnSet("name"));

        string accountName =
            account.GetAttributeValue<string>("name");
    }
}
```

---

# Scenario 2: Retrieve Child Records

Suppose:

```text
Account

↓

Many Contacts
```

Need all contacts of an Account.

```csharp
QueryExpression query =
    new QueryExpression("contact");

query.ColumnSet =
    new ColumnSet("fullname", "emailaddress1");

query.Criteria.AddCondition(
    "parentcustomerid",
    ConditionOperator.Equal,
    accountId);

EntityCollection contacts =
    service.RetrieveMultiple(query);
```

Loop

```csharp
foreach(Entity contact in contacts.Entities)
{
    string name =
        contact.GetAttributeValue<string>("fullname");
}
```

---

# Scenario 3: Using FetchXML

```xml
<fetch>
  <entity name="contact">
    <attribute name="fullname"/>
    <filter>
      <condition
        attribute="parentcustomerid"
        operator="eq"
        value="{ACCOUNTID}" />
    </filter>
  </entity>
</fetch>
```

Plugin

```csharp
FetchExpression fetch =
    new FetchExpression(fetchXml);

EntityCollection contacts =
    service.RetrieveMultiple(fetch);
```

---

# Scenario 4: Using LinkEntity (SQL JOIN Equivalent)

Need Account with Primary Contact.

```text
Account

↓

Primary Contact
```

```csharp
QueryExpression query =
    new QueryExpression("account");

query.ColumnSet =
    new ColumnSet("name");

LinkEntity link =
    query.AddLink(
        "contact",
        "primarycontactid",
        "contactid");

link.Columns =
    new ColumnSet("fullname");

link.EntityAlias = "contact";
```

Read

```csharp
AliasedValue value =
    (AliasedValue)account["contact.fullname"];

string name =
    value.Value.ToString();
```

---

# Scenario 5: Many-to-Many Relationship

Example

```text
User

↓

N:N

↓

Role
```

Use `LinkEntity`.

```text
systemuser

↓

systemuserroles

↓

role
```

---

# Scenario 6: Using Entity Images (Best Performance)

Suppose you need Account Name and it already exists in the image.

Instead of

```text
Retrieve()

↓

Database
```

Use

```csharp
Entity preImage =
    context.PreEntityImages["PreImage"];

EntityReference account =
    preImage.GetAttributeValue<EntityReference>(
        "parentcustomerid");
```

No extra database call.

---

# Performance Best Practices

✅ Use **Entity Images** whenever possible.

✅ Retrieve only required columns.

```csharp
new ColumnSet("name")
```

❌ Avoid

```csharp
new ColumnSet(true)
```

unless you truly need all columns.

❌ Avoid repeated `Retrieve()` calls inside loops (N+1 query problem).

---

# Real Project Example

**Requirement**

When an Opportunity is won:

* Retrieve Customer
* Retrieve Customer Address
* Retrieve Active Contracts
* Generate Invoice

```text
Opportunity

↓

Customer

↓

Account

↓

Contracts

↓

Invoice
```

Implementation:

1. Read `customerid` lookup.
2. Retrieve Account.
3. Query active Contracts using `RetrieveMultiple`.
4. Create Invoice.

---

# Interview Answer (2 Minutes)

> In a plugin, related records are accessed using `IOrganizationService`. For parent records, I read the lookup (`EntityReference`) from the current entity and use `Retrieve()` to fetch only the required columns. For child records, I use `RetrieveMultiple()` with a `QueryExpression` or `FetchXML`. When joining related tables in a single query, I use `LinkEntity`, which works similarly to a SQL JOIN. If the required data is already available in a Pre or Post Entity Image, I use the image instead of making another database call, as it improves performance and reduces unnecessary round trips to Dataverse.
