If you're preparing for a **5+ years .NET Full Stack interview**, the C# coding questions are usually based on:

1. Collections
2. LINQ
3. String manipulation
4. Arrays
5. Dictionary & HashSet
6. Grouping
7. Sorting
8. Objects & Classes
9. Recursion
10. Async/Parallel
11. Real-world LINQ scenarios

The easiest way to remember them is to think in terms of **LINQ operations**:

```
Where()      -> Filter
Select()     -> Transform
OrderBy()    -> Sort
GroupBy()    -> Group
Distinct()   -> Remove duplicates
Any()        -> Exists?
All()        -> All satisfy?
First()      -> First item
Single()     -> Exactly one
Count()      -> Count
Sum()        -> Total
Max()/Min()  -> Highest/Lowest
Join()       -> SQL Join
ToDictionary()-> Dictionary
```

---

# 1. Find Even Numbers

```csharp
var numbers = new List<int> {1,2,3,4,5,6,7,8};

var result = numbers.Where(x => x % 2 == 0);

Console.WriteLine(string.Join(",", result));
```

Output

```
2,4,6,8
```

Remember

```
Where = Filter
```

---

# 2. Find Odd Numbers

```csharp
var result = numbers.Where(x => x % 2 != 0);
```

---

# 3. Find Numbers Greater Than 50

```csharp
var numbers = new List<int>{10,20,55,65,70};

var result = numbers.Where(x=>x>50);
```

---

# 4. Square Every Number

```csharp
var result = numbers.Select(x => x * x);
```

Output

```
100
400
3025
```

Remember

```
Select = Convert
```

---

# 5. Cube Every Number

```csharp
var result = numbers.Select(x => x * x * x);
```

---

# 6. Remove Duplicate Numbers

```csharp
var numbers = new List<int>{1,2,2,3,4,4,5};

var result = numbers.Distinct();
```

Output

```
1 2 3 4 5
```

---

# 7. Count Total Numbers

```csharp
var total = numbers.Count();
```

---

# 8. Count Even Numbers

```csharp
var total = numbers.Count(x=>x%2==0);
```

---

# 9. Sum of Numbers

```csharp
var sum = numbers.Sum();
```

---

# 10. Maximum Number

```csharp
var max = numbers.Max();
```

---

# 11. Minimum Number

```csharp
var min = numbers.Min();
```

---

# 12. Average

```csharp
var avg = numbers.Average();
```

---

# 13. Find First Even Number

```csharp
var first = numbers.First(x=>x%2==0);
```

---

# 14. FirstOrDefault

```csharp
var first = numbers.FirstOrDefault(x=>x>100);
```

Output

```
0
```

---

# 15. Any()

Is there any employee with salary >50000?

```csharp
bool exist = employees.Any(x=>x.Salary>50000);
```

---

# 16. All()

Are all employees adults?

```csharp
bool result = employees.All(x=>x.Age>=18);
```

---

# 17. Sort Ascending

```csharp
numbers.OrderBy(x=>x);
```

---

# 18. Sort Descending

```csharp
numbers.OrderByDescending(x=>x);
```

---

# 19. ThenBy

```csharp
employees
.OrderBy(x=>x.Department)
.ThenBy(x=>x.Name);
```

---

# 20. Reverse

```csharp
numbers.Reverse();
```

---

# 21. Skip First 5

```csharp
numbers.Skip(5);
```

---

# 22. Take First 5

```csharp
numbers.Take(5);
```

---

# 23. Pagination

```csharp
var page=2;
var pageSize=10;

var result = employees
.Skip((page-1)*pageSize)
.Take(pageSize);
```

Very common interview question.

---

# 24. Group By Department

```csharp
var result = employees
.GroupBy(x=>x.Department);
```

Output

```
IT
HR
Finance
```

---

# 25. Count Employees in Each Department

```csharp
var result = employees
.GroupBy(x=>x.Department)
.Select(x=>new
{
    Department=x.Key,
    Count=x.Count()
});
```

Output

```
IT      15
HR       8
Finance 12
```

---

# 26. Highest Salary in Every Department

```csharp
var result =
employees
.GroupBy(x=>x.Department)
.Select(g=>new
{
    Department=g.Key,
    MaxSalary=g.Max(x=>x.Salary)
});
```

---

# 27. Average Salary Department Wise

```csharp
.GroupBy(x=>x.Department)
.Select(g=>new
{
    g.Key,
    Average=g.Average(x=>x.Salary)
});
```

---

# 28. Employee With Highest Salary

```csharp
var emp =
employees
.OrderByDescending(x=>x.Salary)
.First();
```

---

# 29. Second Highest Salary

```csharp
var salary =
employees
.Select(x=>x.Salary)
.Distinct()
.OrderByDescending(x=>x)
.Skip(1)
.First();
```

Interview Favorite

Remember

```
Distinct

↓

Descending

↓

Skip

↓

First
```

---

# 30. Nth Highest Salary

```csharp
int n=3;

var salary =
employees
.Select(x=>x.Salary)
.Distinct()
.OrderByDescending(x=>x)
.Skip(n-1)
.First();
```

---

# 31. Duplicate Numbers

```csharp
var duplicate =
numbers
.GroupBy(x=>x)
.Where(g=>g.Count()>1)
.Select(g=>g.Key);
```

---

# 32. Duplicate Strings

```csharp
var duplicate =
names
.GroupBy(x=>x)
.Where(g=>g.Count()>1)
.Select(g=>g.Key);
```

---

# 33. Frequency of Every Number

```csharp
numbers
.GroupBy(x=>x)
.Select(g=>new
{
    Number=g.Key,
    Count=g.Count()
});
```

Output

```
1  3
2  5
3  2
```

---

# 34. Remove Null

```csharp
names.Where(x=>x!=null);
```

---

# 35. Remove Empty String

```csharp
names.Where(x=>!string.IsNullOrEmpty(x));
```

---

# 36. Remove Null or Empty

```csharp
names.Where(x=>!string.IsNullOrWhiteSpace(x));
```

---

# 37. StartsWith

```csharp
employees.Where(x=>x.Name.StartsWith("A"));
```

---

# 38. EndsWith

```csharp
employees.Where(x=>x.Name.EndsWith("n"));
```

---

# 39. Contains

```csharp
employees.Where(x=>x.Name.Contains("an"));
```

---

# 40. Join Two Lists

```csharp
var result =
employees.Join(departments,
e=>e.DepartmentId,
d=>d.Id,
(e,d)=>new
{
    e.Name,
    d.DepartmentName
});
```

Equivalent SQL

```sql
SELECT *
FROM Employee e
JOIN Department d
ON e.DepartmentId=d.Id
```

---

# 41. Left Join

```csharp
var result =
from e in employees
join d in departments
on e.DepartmentId equals d.Id
into grp
from d in grp.DefaultIfEmpty()
select new
{
    e.Name,
    Department = d?.DepartmentName
};
```

---

# 42. Dictionary

```csharp
var dict =
employees.ToDictionary(x=>x.Id);
```

---

# 43. HashSet

```csharp
HashSet<int> set = new(numbers);
```

Automatically removes duplicates.

---

# 44. Find Missing Numbers

```csharp
var expected = Enumerable.Range(1,10);

var missing =
expected.Except(numbers);
```

---

# 45. Common Numbers

```csharp
list1.Intersect(list2);
```

---

# 46. Numbers Present Only in First List

```csharp
list1.Except(list2);
```

---

# 47. Union

```csharp
list1.Union(list2);
```

---

# 48. Flatten Nested Collection

```csharp
var allOrders =
customers.SelectMany(x=>x.Orders);
```

Remember

```
Select = One

SelectMany = Many
```

---

# 49. Find Top 3 Salaries

```csharp
employees
.OrderByDescending(x=>x.Salary)
.Take(3);
```

---

# 50. Real Interview Question

Employees

```
Name

Department

Salary
```

Find highest paid employee in every department.

```csharp
var result =
employees
.GroupBy(e=>e.Department)
.Select(g=>g
.OrderByDescending(e=>e.Salary)
.First());
```

---

# Quick LINQ Cheat Sheet (Memory Trick)

| Requirement           | LINQ Method                        |
| --------------------- | ---------------------------------- |
| Filter                | `Where()`                          |
| Convert               | `Select()`                         |
| Flatten               | `SelectMany()`                     |
| Sort ↑                | `OrderBy()`                        |
| Sort ↓                | `OrderByDescending()`              |
| Secondary Sort        | `ThenBy()`                         |
| Remove Duplicates     | `Distinct()`                       |
| Group                 | `GroupBy()`                        |
| Count                 | `Count()`                          |
| Sum                   | `Sum()`                            |
| Average               | `Average()`                        |
| Maximum               | `Max()`                            |
| Minimum               | `Min()`                            |
| Exists?               | `Any()`                            |
| All Match?            | `All()`                            |
| First                 | `First()`                          |
| Safe First            | `FirstOrDefault()`                 |
| Single Record         | `Single()`                         |
| Pagination            | `Skip()` + `Take()`                |
| SQL Inner Join        | `Join()`                           |
| SQL Left Join         | `GroupJoin()` + `DefaultIfEmpty()` |
| Difference            | `Except()`                         |
| Common                | `Intersect()`                      |
| Merge Unique          | `Union()`                          |
| Convert to Dictionary | `ToDictionary()`                   |

These cover the vast majority of C# coding questions asked in .NET interviews. Once you're comfortable with these patterns, you'll be able to solve many array, string, collection, and object-manipulation problems by combining a small set of LINQ operators.
