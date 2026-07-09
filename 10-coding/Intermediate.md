For a **5–8 years .NET Developer** interview, you're right—questions like `Where()`, `Select()`, and `GroupBy()` are considered basic. Most product companies (Microsoft, JPMorgan, Oracle, Adobe, Walmart, Amazon, etc.) will ask **DSA-based coding questions**. While you won't always use LINQ to solve them optimally, I recommend first solving them with algorithms and then discussing whether LINQ is appropriate.

Here are advanced questions grouped by topic.

---

# 1. Two Sum ⭐⭐⭐ (Very Frequently Asked)

### Problem

```text
Input:

[2,7,11,15]

Target = 9

Output

[0,1]
```

### Best Solution (O(n))

```csharp
public static int[] TwoSum(int[] nums, int target)
{
    Dictionary<int, int> map = new();

    for (int i = 0; i < nums.Length; i++)
    {
        int diff = target - nums[i];

        if (map.ContainsKey(diff))
            return new[] { map[diff], i };

        map[nums[i]] = i;
    }

    return Array.Empty<int>();
}
```

### Interview Concept

```
Dictionary

Value -> Index

Lookup = O(1)

Overall = O(n)
```

---

# 2. Longest Substring Without Repeating Characters ⭐⭐⭐⭐⭐

Example

```
abcabcbb

Output

3

abc
```

### Sliding Window

```csharp
public static int Length(string s)
{
    HashSet<char> set = new();

    int left = 0;
    int max = 0;

    for (int right = 0; right < s.Length; right++)
    {
        while (set.Contains(s[right]))
        {
            set.Remove(s[left]);
            left++;
        }

        set.Add(s[right]);

        max = Math.Max(max, right - left + 1);
    }

    return max;
}
```

Time

```
O(n)
```

Memory Trick

```
Duplicate found

↓

Move Left Pointer

↓

Continue
```

---

# 3. Maximum Subarray Sum (Kadane Algorithm)

```
-2 1 -3 4 -1 2 1 -5 4

Output

6

4 -1 2 1
```

```csharp
public static int MaxSubArray(int[] nums)
{
    int current = nums[0];
    int max = nums[0];

    for(int i=1;i<nums.Length;i++)
    {
        current = Math.Max(nums[i], current + nums[i]);
        max = Math.Max(max, current);
    }

    return max;
}
```

Complexity

```
O(n)
```

---

# 4. Product of Array Except Self

```
1 2 3 4

Output

24 12 8 6
```

Without division.

Uses

```
Prefix

Suffix
```

---

# 5. Merge Overlapping Intervals ⭐⭐⭐⭐⭐

```
Input

[1,3]

[2,6]

[8,10]

[15,18]

Output

[1,6]

[8,10]

[15,18]
```

Logic

```
Sort

↓

Compare

↓

Merge
```

---

# 6. Rotate Array

```
1 2 3 4 5 6 7

k=3

Output

5 6 7 1 2 3 4
```

Optimal

Reverse Algorithm

```
Reverse Entire

↓

Reverse First k

↓

Reverse Remaining
```

---

# 7. Move Zeroes

```
1 0 2 0 4 5

↓

1 2 4 5 0 0
```

Two Pointer

```
public static void MoveZeroes(int[] nums)
{
    int insert = 0;

    foreach(var n in nums)
    {
        if(n!=0)
            nums[insert++] = n;
    }

    while(insert<nums.Length)
        nums[insert++] = 0;
}
```

---

# 8. Dutch National Flag

```
0 1 2 1 0 2

↓

0 0 1 1 2 2
```

Pointers

```
Low

Mid

High
```

---

# 9. Trapping Rain Water ⭐⭐⭐⭐⭐

```
0 1 0 2 1 0 1 3 2 1 2 1

Output

6
```

Uses

```
Two Pointer

LeftMax

RightMax
```

---

# 10. Container With Most Water

```
Height

1 8 6 2 5 4 8 3 7

Output

49
```

Uses

```
Two Pointer
```

---

# 11. Best Time to Buy and Sell Stock

```
7 1 5 3 6 4

Profit

5
```

```csharp
int min = int.MaxValue;
int profit = 0;

foreach(var price in prices)
{
    min = Math.Min(min, price);

    profit = Math.Max(profit, price-min);
}
```

---

# 12. Find Missing Number

```
0 1 3

Output

2
```

Using XOR

```
O(n)

No Extra Space
```

---

# 13. First Missing Positive

```
3 4 -1 1

Output

2
```

Hard Interview Question

---

# 14. Majority Element

```
2 2 1 1 2 2

Output

2
```

Boyer Moore Voting Algorithm

```
O(n)

O(1)
```

---

# 15. Next Greater Element

Uses

```
Stack
```

Example

```
2 1 2 4 3

Output

4 2 4 -1 -1
```

---

# 16. Largest Rectangle Histogram

Uses

```
Monotonic Stack
```

---

# 17. Valid Parentheses

```
()

[]

{}

Output

True
```

Uses

```
Stack
```

---

# 18. Reverse Words

```
I Love India

↓

India Love I
```

---

# 19. Longest Palindrome

```
babad

Output

bab
```

---

# 20. Group Anagrams

```
eat

tea

tan

ate

nat

bat

↓

eat tea ate

tan nat

bat
```

Dictionary + Sorted String

---

# 21. Minimum Window Substring

```
ADOBECODEBANC

ABC

↓

BANC
```

Sliding Window

Very Popular

---

# 22. String Compression

```
aaabbcccc

↓

a3b2c4
```

---

# 23. Check Rotation

```
abcd

cdab

True
```

```
(s+s).Contains(target)
```

---

# 24. Longest Common Prefix

```
flower

flow

flight

↓

fl
```

---

# 25. Edit Distance

```
Horse

Ros

↓

3
```

Dynamic Programming

---

# 26. Decode String

```
3[a2[c]]

↓

accaccacc
```

Uses

```
Stack
```

---

# 27. Spiral Matrix

```
1 2 3

4 5 6

7 8 9

↓

1 2 3 6 9 8 7 4 5
```

---

# 28. Search in Rotated Sorted Array

```
4 5 6 7 0 1 2

Find 0
```

Binary Search

```
O(log n)
```

---

# 29. Kth Largest Element

```
3 2 1 5 6 4

k=2

↓

5
```

Heap

QuickSelect

---

# 30. Median of Two Sorted Arrays

Very Famous

Google

Amazon

Microsoft

Hard

Uses

```
Binary Search
```

---

# Must-Know Patterns (Don't Memorize Every Problem)

| Pattern               | Common Problems                                                               |
| --------------------- | ----------------------------------------------------------------------------- |
| Two Pointers          | Two Sum (sorted), Move Zeroes, Container With Most Water, Trapping Rain Water |
| Sliding Window        | Longest Substring, Minimum Window Substring, Max Consecutive Ones             |
| HashMap/Dictionary    | Two Sum, Group Anagrams, Frequency Count                                      |
| HashSet               | Longest Consecutive Sequence, Duplicate Detection                             |
| Prefix Sum            | Subarray Sum Equals K, Product Except Self                                    |
| Binary Search         | Rotated Array, First/Last Position, Median Problems                           |
| Stack                 | Valid Parentheses, Next Greater Element, Histogram, Decode String             |
| Queue/Deque           | Sliding Window Maximum                                                        |
| Heap (Priority Queue) | Kth Largest, Merge K Sorted Lists, Top K Frequent                             |
| Dynamic Programming   | Edit Distance, Coin Change, Longest Increasing Subsequence                    |
| Greedy                | Jump Game, Gas Station, Stock Buy/Sell                                        |
| Backtracking          | N-Queens, Subsets, Permutations, Sudoku Solver                                |
| Graph                 | Number of Islands, Clone Graph, Course Schedule                               |
| Tree                  | Lowest Common Ancestor, Diameter, Level Order Traversal                       |

## For a 5–8 Years .NET Developer Interview

If your goal is to crack interviews at companies like JPMorgan, Microsoft, Goldman Sachs, or Walmart, I recommend mastering around **75–100 carefully selected problems** instead of solving hundreds randomly. Those problems should cover every major pattern above, because interviewers often test your ability to recognize the pattern more than your ability to memorize a specific solution.

I can also provide a **"Top 100 C# DSA Interview Questions"** guide with:

* Problems ordered from Easy → Medium → Hard.
* Full C# solutions.
* Time and space complexity analysis.
* Dry runs with diagrams.
* Common interview follow-up questions.
* Alternative solutions and optimizations.
* Memory tricks to recognize each pattern quickly.

That collection is designed specifically for experienced **.NET developers (5+ years)** preparing for coding rounds.
