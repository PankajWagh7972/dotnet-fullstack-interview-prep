For **5+ years .NET interviews**, many LINQ coding questions require combining LINQ with **Dictionary** or **HashSet** for optimal time complexity. Interviewers usually expect you to avoid nested loops (`O(n²)`) and instead use hashing (`O(n)`).

Here are some of the most common questions.

---

# 1. Find Duplicate Elements (HashSet)

### Question

Given an integer array, print all duplicate numbers.

```csharp
int[] numbers = { 1, 2, 3, 4, 2, 5, 1, 6, 4 };
```

### Expected Output

```
2
1
4
```

---

### Using HashSet

```csharp
int[] numbers = { 1, 2, 3, 4, 2, 5, 1, 6, 4 };

HashSet<int> seen = new();
HashSet<int> duplicates = new();

foreach (var num in numbers)
{
    if (!seen.Add(num))
        duplicates.Add(num);
}

Console.WriteLine(string.Join(", ", duplicates));
```

Time Complexity

```
O(n)
```

---

### Using LINQ

```csharp
var duplicates = numbers
                .GroupBy(x => x)
                .Where(g => g.Count() > 1)
                .Select(g => g.Key);
```

---

# 2. First Non-Repeating Character (Dictionary)

### Question

```
"swiss"
```

Expected Output

```
w
```

---

### Solution

```csharp
string word = "swiss";

Dictionary<char, int> frequency = new();

foreach (char c in word)
{
    frequency[c] = frequency.GetValueOrDefault(c) + 1;
}

char result = word.First(c => frequency[c] == 1);

Console.WriteLine(result);
```

Output

```
w
```
```
string word = "swiss";

char firstNonRepeating = word
    .GroupBy(c => c)
    .Where(g => g.Count() == 1)
    .Select(g => g.Key)
    .First();

Console.WriteLine(firstNonRepeating);
```
---

# 3. Two Sum (Dictionary)

### Question

```
nums = [2,7,11,15]
target = 9
```

Expected Output

```
0,1
```

---

### Solution

```csharp
int[] nums = {2,7,11,15};
int target = 9;

Dictionary<int,int> map = new();

for(int i=0;i<nums.Length;i++)
{
    int diff = target - nums[i];

    if(map.ContainsKey(diff))
    {
        Console.WriteLine($"{map[diff]}, {i}");
        break;
    }

    map[nums[i]] = i;
}
```

Complexity

```
O(n)
```

---

# 4. Intersection of Two Arrays (HashSet)

### Question

```
A = [1,2,3,4]
B = [3,4,5,6]
```

Output

```
3 4
```

---

### Solution

```csharp
int[] A = {1,2,3,4};
int[] B = {3,4,5,6};

HashSet<int> set = new(A);

var result = B.Where(x => set.Contains(x));

Console.WriteLine(string.Join(",", result));
```

---

# 5. Remove Duplicates

### Question

```
[1,2,2,3,4,4,5]
```

Output

```
1,2,3,4,5
```

---

### HashSet

```csharp
var unique = new HashSet<int>(numbers);
```

---

### LINQ

```csharp
var unique = numbers.Distinct();
```

---

# 6. Count Frequency (Dictionary)

### Question

```
[1,1,2,3,2,2]
```

Output

```
1 -> 2
2 -> 3
3 -> 1
```

---

### Solution

```csharp
Dictionary<int,int> dict = new();

foreach(var num in numbers)
{
    dict[num] = dict.GetValueOrDefault(num) + 1;
}

foreach(var item in dict)
{
    Console.WriteLine($"{item.Key} -> {item.Value}");
}
```

---

# 7. Find Missing Numbers

### Question

```
Array = [1,2,4,6]
Range = 1-6
```

Output

```
3
5
```

---

### Solution

```csharp
int[] nums = {1,2,4,6};

HashSet<int> set = nums.ToHashSet();

var missing = Enumerable.Range(1,6)
                        .Where(x => !set.Contains(x));

Console.WriteLine(string.Join(",", missing));
```

---

# 8. Employees Working in Both Projects

### Question

```csharp
ProjectA = {1,2,3,4}
ProjectB = {3,4,5,6}
```

Output

```
3
4
```

---

### Solution

```csharp
HashSet<int> projectA = new(){1,2,3,4};

var common = projectB.Where(projectA.Contains);
```

---

# 9. Group Employees by Department (Dictionary)

```csharp
class Employee
{
    public string Name { get; set; }
    public string Department { get; set; }
}
```

---

### Solution

```csharp
var grouped = employees
                .GroupBy(e => e.Department)
                .ToDictionary(g => g.Key,
                              g => g.ToList());
```

Now

```csharp
grouped["IT"];
```

returns all IT employees in **O(1)** lookup.

---

# 10. Find Common Strings (HashSet)

```csharp
string[] list1 = {"A","B","C"};
string[] list2 = {"B","C","D"};
```

---

### Solution

```csharp
HashSet<string> set = list1.ToHashSet();

var common = list2.Where(set.Contains);
```

---

# 11. Validate Unique Email Addresses

### Question

Check if any duplicate email exists.

---

### Solution

```csharp
HashSet<string> emails = new();

foreach(var email in emailList)
{
    if(!emails.Add(email))
    {
        Console.WriteLine("Duplicate Found");
        break;
    }
}
```

---

# 12. Count Occurrences of Words

```csharp
string sentence = "apple banana apple orange banana apple";
```

---

### Solution

```csharp
var result = sentence
            .Split(' ')
            .GroupBy(x => x)
            .ToDictionary(g => g.Key,
                          g => g.Count());

foreach(var item in result)
{
    Console.WriteLine($"{item.Key} : {item.Value}");
}
```

---

# 13. Join Two Lists Using Dictionary (Optimization)

### Question

```csharp
Employees

Id Name

Departments

EmployeeId Department
```

Avoid nested loops.

---

### Solution

```csharp
var deptLookup = departments.ToDictionary(x => x.EmployeeId);

foreach(var emp in employees)
{
    if(deptLookup.TryGetValue(emp.Id, out var dept))
    {
        Console.WriteLine($"{emp.Name} - {dept.Department}");
    }
}
```

Complexity

```
Nested Loop
O(n²)

Dictionary
O(n)
```

---

# 14. Find Anagrams (Dictionary)

```text
listen
silent
```

---

### Solution

```csharp
bool IsAnagram(string a, string b)
{
    if(a.Length != b.Length)
        return false;

    Dictionary<char,int> dict = new();

    foreach(char c in a)
        dict[c] = dict.GetValueOrDefault(c) + 1;

    foreach(char c in b)
    {
        if(!dict.ContainsKey(c))
            return false;

        dict[c]--;

        if(dict[c] == 0)
            dict.Remove(c);
    }

    return dict.Count == 0;
}
```

---

# 15. Longest Consecutive Sequence (HashSet)

### Question

```text
[100,4,200,1,3,2]
```

Output

```
4
```

Because

```
1,2,3,4
```

---

### Solution

```csharp
HashSet<int> set = nums.ToHashSet();

int longest = 0;

foreach(var num in set)
{
    if(!set.Contains(num-1))
    {
        int current = num;
        int length = 1;

        while(set.Contains(current+1))
        {
            current++;
            length++;
        }

        longest = Math.Max(longest,length);
    }
}

Console.WriteLine(longest);
```

Time Complexity

```
O(n)
```

---

# Frequently Asked Interview Scenarios

| Problem                           | Best Data Structure | Complexity |
| --------------------------------- | ------------------- | ---------- |
| Find duplicates                   | HashSet             | O(n)       |
| Remove duplicates                 | HashSet             | O(n)       |
| Frequency count                   | Dictionary          | O(n)       |
| Two Sum                           | Dictionary          | O(n)       |
| Join two collections by ID        | Dictionary          | O(n)       |
| Find common elements              | HashSet             | O(n)       |
| Missing numbers                   | HashSet             | O(n)       |
| First non-repeating character     | Dictionary          | O(n)       |
| Longest consecutive sequence      | HashSet             | O(n)       |
| Detect duplicate emails/usernames | HashSet             | O(n)       |
| Group by key with fast lookup     | Dictionary          | O(n)       |
| Word frequency                    | Dictionary          | O(n)       |

These are among the most commonly asked hashing-based coding problems in C# interviews, especially for mid-to-senior .NET developer roles, because they test both your LINQ knowledge and your ability to choose efficient data structures.

