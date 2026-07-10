# Recursive CTE Interview Questions (Scenario-Based with Output)

These are the types of Recursive CTE questions commonly asked in interviews for **5+ years of SQL Server/.NET developers**. Each scenario includes:

* Business Problem
* Sample Data
* SQL Solution
* Expected Output
* Interview Explanation

---

# Scenario 1: Employee-Manager Hierarchy (Most Frequently Asked)

## Employee Table

| EmployeeId | EmployeeName | ManagerId |
| ---------- | ------------ | --------- |
| 1          | CEO          | NULL      |
| 2          | Manager A    | 1         |
| 3          | Manager B    | 1         |
| 4          | Developer 1  | 2         |
| 5          | Developer 2  | 2         |
| 6          | Tester       | 3         |
| 7          | Intern       | 4         |

Hierarchy

```text
CEO
├── Manager A
│   ├── Developer 1
│   │   └── Intern
│   └── Developer 2
└── Manager B
    └── Tester
```

---

## Question

Display the complete employee hierarchy with levels.

---

## SQL

```sql
WITH EmployeeHierarchy AS
(
    -- Anchor Member
    SELECT
        EmployeeId,
        EmployeeName,
        ManagerId,
        0 AS Level
    FROM Employee
    WHERE ManagerId IS NULL

    UNION ALL

    -- Recursive Member
    SELECT
        e.EmployeeId,
        e.EmployeeName,
        e.ManagerId,
        h.Level + 1
    FROM Employee e
    INNER JOIN EmployeeHierarchy h
        ON e.ManagerId = h.EmployeeId
)

SELECT *
FROM EmployeeHierarchy
ORDER BY Level, EmployeeId;
```

---

## Output

| EmployeeId | EmployeeName | ManagerId | Level |
| ---------- | ------------ | --------- | ----- |
| 1          | CEO          | NULL      | 0     |
| 2          | Manager A    | 1         | 1     |
| 3          | Manager B    | 1         | 1     |
| 4          | Developer 1  | 2         | 2     |
| 5          | Developer 2  | 2         | 2     |
| 6          | Tester       | 3         | 2     |
| 7          | Intern       | 4         | 3     |

---

## Interview Explanation

**Anchor Member**

```sql
SELECT ...
WHERE ManagerId IS NULL
```

Starts from the CEO.

---

**Recursive Member**

```sql
JOIN EmployeeHierarchy
ON e.ManagerId = h.EmployeeId
```

Keeps finding employees reporting to the previous level.

---

Execution

```text
Iteration 1

CEO

↓

Iteration 2

Manager A
Manager B

↓

Iteration 3

Developer1
Developer2
Tester

↓

Iteration 4

Intern

↓

No more rows

Stop
```

---

# Scenario 2: Find All Managers of an Employee

Suppose employee **Intern (7)** is selected.

---

## Question

Find everyone above Intern.

---

## SQL

```sql
WITH ManagerHierarchy AS
(
    SELECT
        EmployeeId,
        EmployeeName,
        ManagerId
    FROM Employee
    WHERE EmployeeId = 7

    UNION ALL

    SELECT
        e.EmployeeId,
        e.EmployeeName,
        e.ManagerId
    FROM Employee e
    INNER JOIN ManagerHierarchy mh
        ON e.EmployeeId = mh.ManagerId
)

SELECT *
FROM ManagerHierarchy;
```

---

## Output

| EmployeeId | EmployeeName |
| ---------- | ------------ |
| 7          | Intern       |
| 4          | Developer 1  |
| 2          | Manager A    |
| 1          | CEO          |

---

Hierarchy

```text
Intern

↑

Developer1

↑

Manager A

↑

CEO
```

---

# Scenario 3: Folder Structure

Table

| FolderId | FolderName | ParentFolderId |
| -------- | ---------- | -------------- |
| 1        | Root       | NULL           |
| 2        | HR         | 1              |
| 3        | Finance    | 1              |
| 4        | Salary     | 2              |
| 5        | Leaves     | 2              |
| 6        | Payroll    | 3              |
| 7        | GST        | 3              |

---

Question

Display folder hierarchy.

---

SQL

```sql
WITH FolderTree AS
(
SELECT
FolderId,
FolderName,
ParentFolderId,
0 Level
FROM Folder
WHERE ParentFolderId IS NULL

UNION ALL

SELECT
f.FolderId,
f.FolderName,
f.ParentFolderId,
ft.Level+1
FROM Folder f
JOIN FolderTree ft
ON f.ParentFolderId=ft.FolderId
)

SELECT *
FROM FolderTree;
```

---

Output

| Folder  | Level |
| ------- | ----- |
| Root    | 0     |
| HR      | 1     |
| Finance | 1     |
| Salary  | 2     |
| Leaves  | 2     |
| Payroll | 2     |
| GST     | 2     |

Visual

```text
Root

├── HR

│   ├── Salary

│   └── Leaves

└── Finance

    ├── Payroll

    └── GST
```

---

# Scenario 4: Product Categories

Table

| Id | Category    | ParentId |
| -- | ----------- | -------- |
| 1  | Electronics | NULL     |
| 2  | Mobile      | 1        |
| 3  | Laptop      | 1        |
| 4  | Android     | 2        |
| 5  | iPhone      | 2        |
| 6  | Dell        | 3        |
| 7  | Lenovo      | 3        |

---

Question

Display all child categories.

---

SQL

```sql
WITH CategoryTree AS
(
SELECT
Id,
Category,
ParentId
FROM Category
WHERE ParentId IS NULL

UNION ALL

SELECT
c.Id,
c.Category,
c.ParentId
FROM Category c
JOIN CategoryTree ct
ON c.ParentId=ct.Id
)

SELECT *
FROM CategoryTree;
```

---

Output

| Category    | Level |
| ----------- | ----- |
| Electronics | 0     |
| Mobile      | 1     |
| Laptop      | 1     |
| Android     | 2     |
| iPhone      | 2     |
| Dell        | 2     |
| Lenovo      | 2     |

Visual

```text
Electronics

├── Mobile

│   ├── Android

│   └── iPhone

└── Laptop

    ├── Dell

    └── Lenovo
```

---

# Scenario 5: Display Complete Path

Question

Instead of showing level, show

```text
CEO

CEO → Manager A

CEO → Manager A → Developer1

CEO → Manager A → Developer1 → Intern
```

---

SQL

```sql
WITH Org AS
(
SELECT
EmployeeId,
EmployeeName,
ManagerId,
CAST(EmployeeName AS VARCHAR(MAX)) AS Path
FROM Employee
WHERE ManagerId IS NULL

UNION ALL

SELECT
e.EmployeeId,
e.EmployeeName,
e.ManagerId,
o.Path+' -> '+e.EmployeeName
FROM Employee e
JOIN Org o
ON e.ManagerId=o.EmployeeId
)

SELECT EmployeeName,Path
FROM Org;
```

---

Output

| Employee   | Path                                  |
| ---------- | ------------------------------------- |
| CEO        | CEO                                   |
| Manager A  | CEO → Manager A                       |
| Manager B  | CEO → Manager B                       |
| Developer1 | CEO → Manager A → Developer1          |
| Developer2 | CEO → Manager A → Developer2          |
| Tester     | CEO → Manager B → Tester              |
| Intern     | CEO → Manager A → Developer1 → Intern |

---

# Scenario 6: Indented Hierarchy

Question

Print hierarchy like Windows Explorer.

---

SQL

```sql
WITH Org AS
(
SELECT
EmployeeId,
EmployeeName,
ManagerId,
0 Level
FROM Employee
WHERE ManagerId IS NULL

UNION ALL

SELECT
e.EmployeeId,
e.EmployeeName,
e.ManagerId,
o.Level+1
FROM Employee e
JOIN Org o
ON e.ManagerId=o.EmployeeId
)

SELECT
REPLICATE('    ',Level)+EmployeeName AS Hierarchy
FROM Org;
```

---

Output

```text
CEO

    Manager A

        Developer1

            Intern

        Developer2

    Manager B

        Tester
```

---

# Scenario 7: Detect Circular Reference

Table

```text
A → B

B → C

C → A
```

Question

What happens?

---

SQL

```sql
WITH Test AS
(
SELECT
EmployeeId,
ManagerId
FROM Employee
WHERE EmployeeId=1

UNION ALL

SELECT
e.EmployeeId,
e.ManagerId
FROM Employee e
JOIN Test t
ON e.ManagerId=t.EmployeeId
)

SELECT *
FROM Test;
```

---

Output

```text
Msg 530

The statement terminated.

The maximum recursion 100 has been exhausted before statement completion.
```

---

Solution

```sql
OPTION(MAXRECURSION 500)
```

or

```sql
OPTION(MAXRECURSION 0)
```

Use `MAXRECURSION 0` carefully because it removes the recursion limit and can lead to an infinite loop if the data contains cycles.

---

# Scenario 8: Calculate Employee Depth

Question

Find how deep each employee is in the hierarchy.

Output

| Employee   | Depth |
| ---------- | ----- |
| CEO        | 0     |
| Manager A  | 1     |
| Manager B  | 1     |
| Developer1 | 2     |
| Developer2 | 2     |
| Tester     | 2     |
| Intern     | 3     |

---

SQL

```sql
WITH EmployeeDepth AS
(
SELECT
EmployeeId,
EmployeeName,
ManagerId,
0 AS Depth
FROM Employee
WHERE ManagerId IS NULL

UNION ALL

SELECT
e.EmployeeId,
e.EmployeeName,
e.ManagerId,
ed.Depth + 1
FROM Employee e
JOIN EmployeeDepth ed
ON e.ManagerId = ed.EmployeeId
)

SELECT EmployeeName, Depth
FROM EmployeeDepth;
```

---

# Common Interview Follow-Up Questions

### Q1. Why do we use `UNION ALL` instead of `UNION`?

**Answer:** `UNION ALL` avoids duplicate elimination and sorting, making recursion faster. `UNION` introduces unnecessary overhead.

---

### Q2. What are the two mandatory parts of a Recursive CTE?

1. **Anchor Member** – starting rows.
2. **Recursive Member** – references the CTE itself to fetch the next level.

---

### Q3. How does SQL Server know when to stop recursion?

Recursion stops automatically when the recursive member returns **no new rows**. If the data contains a cycle, SQL Server stops after reaching the default recursion limit of **100** and raises an error.

---

### Q4. Where are Recursive CTEs used in real projects?

* Employee reporting hierarchies
* Organization charts
* Product category trees
* Folder and file systems
* Menu hierarchies
* Bill of Materials (BOM)
* Comment/reply threads
* Multi-level approval workflows

---

### Q5. What are the limitations of Recursive CTEs?

* Default maximum recursion is **100** levels.
* Performance may degrade on very deep or wide hierarchies.
* Cyclic relationships must be handled carefully.
* Alternative models (such as hierarchy paths or closure tables) may perform better for frequently queried, very large hierarchies.

These are the kinds of scenario-based Recursive CTE questions and explanations commonly seen in interviews for experienced SQL Server and .NET developers.
