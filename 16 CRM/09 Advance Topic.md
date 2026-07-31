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
