If you're preparing for **5+ years SQL Server interviews**, interviewers often give a **query problem** rather than asking about SQL syntax. They expect you to write the query on the spot.

Below are **20 advanced scenario-based SQL query questions** (excluding Recursive CTEs), along with sample data, expected output, and solutions.

---

# 1. Find the Second Highest Salary

### Employee

| Id | Name  | Salary |
| -- | ----- | ------ |
| 1  | John  | 5000   |
| 2  | Mike  | 7000   |
| 3  | David | 6000   |
| 4  | Sam   | 7000   |

### Question

Find the second highest salary.

### Output

| Salary |
| ------ |
| 6000   |

### Answer

```sql
SELECT MAX(Salary) AS SecondHighestSalary
FROM Employee
WHERE Salary < (SELECT MAX(Salary) FROM Employee);
```

---

# 2. Find Employees Having the Same Salary

### Employee

| Name  | Salary |
| ----- | ------ |
| John  | 5000   |
| Mike  | 7000   |
| David | 5000   |
| Sam   | 9000   |

### Output

| Salary |
| ------ |
| 5000   |

### Answer

```sql
SELECT Salary
FROM Employee
GROUP BY Salary
HAVING COUNT(*) > 1;
```

---

# 3. Find Duplicate Records

### Employee

| Id | Email                             |
| -- | --------------------------------- |
| 1  | [a@gmail.com](mailto:a@gmail.com) |
| 2  | [b@gmail.com](mailto:b@gmail.com) |
| 3  | [a@gmail.com](mailto:a@gmail.com) |

### Output

| Email                             |
| --------------------------------- |
| [a@gmail.com](mailto:a@gmail.com) |

### Answer

```sql
SELECT Email
FROM Employee
GROUP BY Email
HAVING COUNT(*) > 1;
```

---

# 4. Delete Duplicate Records

### Employee

| Id | Email                             |
| -- | --------------------------------- |
| 1  | [a@gmail.com](mailto:a@gmail.com) |
| 2  | [b@gmail.com](mailto:b@gmail.com) |
| 3  | [a@gmail.com](mailto:a@gmail.com) |

### Answer

```sql
WITH CTE AS
(
    SELECT *,
           ROW_NUMBER() OVER(PARTITION BY Email ORDER BY Id) AS RN
    FROM Employee
)
DELETE
FROM CTE
WHERE RN > 1;
```

---

# 5. Find Employees Without Any Orders

### Employee

| Id | Name  |
| -- | ----- |
| 1  | John  |
| 2  | Mike  |
| 3  | David |

### Orders

| OrderId | EmployeeId |
| ------- | ---------- |
| 101     | 1          |
| 102     | 3          |

### Output

| Name |
| ---- |
| Mike |

### Answer

```sql
SELECT e.*
FROM Employee e
LEFT JOIN Orders o
ON e.Id = o.EmployeeId
WHERE o.EmployeeId IS NULL;
```

---

# 6. Find Customers Who Ordered Every Product

### Tables

Customer

Order

Product

### Answer

```sql
SELECT CustomerId
FROM Orders
GROUP BY CustomerId
HAVING COUNT(DISTINCT ProductId) =
(
    SELECT COUNT(*)
    FROM Product
);
```

---

# 7. Running Total

### Sales

| Date  | Amount |
| ----- | ------ |
| 1-Jan | 100    |
| 2-Jan | 200    |
| 3-Jan | 300    |

### Output

| Date  | RunningTotal |
| ----- | ------------ |
| 1-Jan | 100          |
| 2-Jan | 300          |
| 3-Jan | 600          |

### Answer

```sql
SELECT
SaleDate,
Amount,
SUM(Amount)
OVER(ORDER BY SaleDate)
AS RunningTotal
FROM Sales;
```

---

# 8. Top 3 Salaries Department Wise

### Output

Return top three salaries from each department.

### Answer

```sql
SELECT *
FROM
(
SELECT *,
DENSE_RANK()
OVER
(
PARTITION BY DepartmentId
ORDER BY Salary DESC
) DR
FROM Employee
)t
WHERE DR <= 3;
```

---

# 9. Find Missing Numbers

Table

```text
1
2
3
5
6
8
```

### Output

```text
4
7
```

### Answer

```sql
WITH Numbers AS
(
SELECT 1 Number

UNION ALL

SELECT Number+1
FROM Numbers
WHERE Number<8
)

SELECT Number
FROM Numbers

EXCEPT

SELECT Id
FROM Test

OPTION(MAXRECURSION 100);
```

---

# 10. Swap Values

Before

| Id | Gender |
| -- | ------ |
| 1  | M      |
| 2  | F      |
| 3  | M      |

After

| Id | Gender |
| -- | ------ |
| 1  | F      |
| 2  | M      |
| 3  | F      |

### Answer

```sql
UPDATE Employee
SET Gender=
CASE
WHEN Gender='M' THEN 'F'
ELSE 'M'
END;
```

---

# 11. Find Consecutive Login Days

Table

| User | Date |
| ---- | ---- |
| A    | 1    |
| A    | 2    |
| A    | 3    |
| A    | 5    |

Question

Find users logged in for 3 consecutive days.

### Answer

```sql
WITH CTE AS
(
SELECT *,
ROW_NUMBER()
OVER
(
PARTITION BY UserId
ORDER BY LoginDate
) RN
FROM LoginHistory
)

SELECT *
FROM CTE;
```

(Usually solved using `ROW_NUMBER()` with date difference logic.)

---

# 12. Pivot Rows into Columns

Before

| Employee | Month | Sales |
| -------- | ----- | ----- |
| John     | Jan   | 100   |
| John     | Feb   | 200   |

Output

| Employee | Jan | Feb |
| -------- | --- | --- |
| John     | 100 | 200 |

### Answer

```sql
SELECT *
FROM Sales
PIVOT
(
SUM(Sales)
FOR Month
IN
(
Jan,
Feb,
Mar
)
)p;
```

---

# 13. Unpivot Columns into Rows

Before

| Employee | Jan | Feb |
| -------- | --- | --- |
| John     | 100 | 200 |

Output

| Employee | Month | Sales |
| -------- | ----- | ----- |
| John     | Jan   | 100   |
| John     | Feb   | 200   |

### Answer

```sql
SELECT *
FROM Sales
UNPIVOT
(
Sales
FOR Month
IN
(
Jan,
Feb
)
)u;
```

---

# 14. Find the Latest Order per Customer

### Answer

```sql
SELECT *
FROM
(
SELECT *,
ROW_NUMBER()
OVER
(
PARTITION BY CustomerId
ORDER BY OrderDate DESC
) RN
FROM Orders
)t
WHERE RN=1;
```

---

# 15. Compare Current Salary with Previous Salary

### Output

|Employee|Current|Previous|

### Answer

```sql
SELECT
EmployeeId,
Salary,
LAG(Salary)
OVER
(
PARTITION BY EmployeeId
ORDER BY EffectiveDate
)
AS PreviousSalary
FROM SalaryHistory;
```

---

# 16. Find Employees Earning More Than Their Manager

### Tables

Employee

ManagerId

Salary

### Answer

```sql
SELECT
e.Name
FROM Employee e
JOIN Employee m
ON e.ManagerId=m.Id
WHERE e.Salary>m.Salary;
```

---

# 17. Highest Sale Per Month

### Answer

```sql
SELECT *
FROM
(
SELECT *,
ROW_NUMBER()
OVER
(
PARTITION BY MONTH(OrderDate)
ORDER BY Amount DESC
) RN
FROM Sales
)t
WHERE RN=1;
```

---

# 18. Find Customers Who Never Ordered

### Answer

```sql
SELECT c.*
FROM Customer c
WHERE NOT EXISTS
(
SELECT 1
FROM Orders o
WHERE o.CustomerId=c.Id
);
```

---

# 19. Find Gaps Between Dates

### Output

```text
Missing Dates

2025-01-05

2025-01-08
```

### Answer

```sql
SELECT
OrderDate,
LEAD(OrderDate)
OVER
(
ORDER BY OrderDate
)
NextDate
FROM Orders;
```

Use `LEAD()` to identify gaps between consecutive dates.

---

# 20. Return the Last Record for Each Group

### Answer

```sql
SELECT *
FROM
(
SELECT *,
ROW_NUMBER()
OVER
(
PARTITION BY Department
ORDER BY CreatedDate DESC
) RN
FROM Employee
)t
WHERE RN=1;
```

---

# Common SQL Coding Questions Asked in Interviews (5–8 Years)

| Question                            | Concepts Tested              |
| ----------------------------------- | ---------------------------- |
| Find Nth Highest Salary             | `DENSE_RANK()`, subqueries   |
| Delete Duplicate Rows               | `ROW_NUMBER()`, CTE          |
| Running Total                       | Window functions             |
| Top N Per Group                     | `PARTITION BY`               |
| Latest Record Per Group             | `ROW_NUMBER()`               |
| Compare Current vs Previous         | `LAG()`                      |
| Compare Current vs Next             | `LEAD()`                     |
| Employees Without Orders            | `LEFT JOIN`, `NOT EXISTS`    |
| Customers With All Products         | `GROUP BY`, `HAVING`         |
| Pivot/Unpivot Data                  | `PIVOT`, `UNPIVOT`           |
| Find Missing Numbers                | CTE, `EXCEPT`                |
| Find Date Gaps                      | `LEAD()`                     |
| Employees Earning More Than Manager | Self Join                    |
| Running Balance                     | Window functions             |
| Monthly Sales Report                | Aggregation + Date functions |
| Remove Duplicates                   | Window functions             |
| Merge Data from Staging             | `MERGE`                      |
| Dynamic Pivot                       | Dynamic SQL                  |
| Split Comma-Separated Values        | `STRING_SPLIT()`             |
| Generate Calendar Table             | Recursive CTE / Tally Table  |

These are the kinds of practical SQL coding problems that frequently appear in technical interviews because they evaluate both SQL syntax and problem-solving skills.
