# C# DSA and Coding Interview Cheatsheet

---

## Pattern Recognition Map

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Problem Signal               → Pattern / Data Structure                 │
│  ─────────────────────────────────────────────────────────────────────  │
│  "Find pair that sums to X"   → Two Pointers / Hash Set                 │
│  "Subarray with max sum/len"  → Sliding Window                           │
│  "Frequency / seen before"    → HashMap / HashSet                        │
│  "Balanced parens / nesting"  → Stack                                    │
│  "BFS / shortest path"        → Queue                                    │
│  "Top K / Kth largest"        → Heap (PriorityQueue)                     │
│  "Sorted array search"        → Binary Search                            │
│  "Range sum queries"          → Prefix Sum                               │
│  "Optimal substructure"       → Dynamic Programming                      │
│  "Connected components"       → Union-Find / BFS/DFS                     │
│  "All paths / combinations"   → DFS + Backtracking                       │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Complexity Quick Table

| Algorithm | Time (avg) | Time (worst) | Space |
|-----------|-----------|--------------|-------|
| Hash map lookup | O(1) | O(n) | O(n) |
| Binary search | O(log n) | O(log n) | O(1) |
| Sorting (general) | O(n log n) | O(n log n) | O(log n) |
| BFS / DFS | O(V + E) | O(V + E) | O(V) |
| Heap push/pop | O(log n) | O(log n) | O(n) |
| DP 1D | O(n) | O(n) | O(n) or O(1) |
| DP 2D | O(n×m) | O(n×m) | O(n×m) |
| Union-Find | O(α(n)) ≈ O(1) | O(α(n)) | O(n) |
| Sliding window | O(n) | O(n) | O(1)–O(k) |
| Prefix sum | O(n) build, O(1) query | same | O(n) |

---

## 1. Two Pointers Template

```csharp
// Pattern: sorted array, find pair with target sum
static (int, int)? TwoSum(int[] nums, int target)
{
    int lo = 0, hi = nums.Length - 1;
    while (lo < hi)
    {
        int sum = nums[lo] + nums[hi];
        if (sum == target) return (lo, hi);
        if (sum < target) lo++;
        else              hi--;
    }
    return null;
}

// Pattern: remove duplicates in-place from sorted array
static int RemoveDuplicates(int[] nums)
{
    if (nums.Length == 0) return 0;
    int write = 1;
    for (int read = 1; read < nums.Length; read++)
        if (nums[read] != nums[read - 1])
            nums[write++] = nums[read];
    return write;
}
```

---

## 2. Sliding Window Template

```csharp
// Fixed window — maximum sum subarray of size k
static int MaxSumFixed(int[] nums, int k)
{
    int windowSum = nums[..k].Sum();
    int best      = windowSum;
    for (int i = k; i < nums.Length; i++)
    {
        windowSum += nums[i] - nums[i - k];
        best = Math.Max(best, windowSum);
    }
    return best;
}

// Variable window — longest subarray with at most k distinct values
static int LongestKDistinct(int[] nums, int k)
{
    var freq = new Dictionary<int, int>();
    int left = 0, best = 0;
    for (int right = 0; right < nums.Length; right++)
    {
        freq[nums[right]] = freq.GetValueOrDefault(nums[right]) + 1;
        while (freq.Count > k)
        {
            if (--freq[nums[left]] == 0) freq.Remove(nums[left]);
            left++;
        }
        best = Math.Max(best, right - left + 1);
    }
    return best;
}
```

---

## 3. HashMap / HashSet Template

```csharp
// Two-sum with unsorted array
static int[] TwoSumMap(int[] nums, int target)
{
    var seen = new Dictionary<int, int>();  // value → index
    for (int i = 0; i < nums.Length; i++)
    {
        int complement = target - nums[i];
        if (seen.TryGetValue(complement, out int j))
            return [j, i];
        seen[nums[i]] = i;
    }
    return [];
}

// Frequency map pattern
static char FirstUniqueChar(string s)
{
    var freq = new int[26];
    foreach (char c in s) freq[c - 'a']++;
    foreach (char c in s) if (freq[c - 'a'] == 1) return c;
    return '\0';
}
```

---

## 4. Stack Template

```csharp
// Valid parentheses
static bool IsValid(string s)
{
    var stack = new Stack<char>();
    foreach (char c in s)
    {
        if (c is '(' or '[' or '{') { stack.Push(c); continue; }
        if (stack.Count == 0) return false;
        char top = stack.Pop();
        if ((c == ')' && top != '(') ||
            (c == ']' && top != '[') ||
            (c == '}' && top != '{')) return false;
    }
    return stack.Count == 0;
}

// Monotonic stack — next greater element
static int[] NextGreater(int[] nums)
{
    var result = new int[nums.Length];
    var stack  = new Stack<int>();  // stores indices
    Array.Fill(result, -1);
    for (int i = 0; i < nums.Length; i++)
    {
        while (stack.Count > 0 && nums[stack.Peek()] < nums[i])
            result[stack.Pop()] = nums[i];
        stack.Push(i);
    }
    return result;
}
```

---

## 5. Queue / BFS Template

```csharp
// BFS on a grid (shortest path)
static int ShortestPath(int[][] grid, (int r, int c) start, (int r, int c) end)
{
    int rows = grid.Length, cols = grid[0].Length;
    var queue   = new Queue<(int r, int c, int dist)>();
    var visited = new bool[rows, cols];

    queue.Enqueue((start.r, start.c, 0));
    visited[start.r, start.c] = true;

    int[][] dirs = [[-1,0],[1,0],[0,-1],[0,1]];
    while (queue.Count > 0)
    {
        var (r, c, dist) = queue.Dequeue();
        if (r == end.r && c == end.c) return dist;
        foreach (var d in dirs)
        {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
            if (grid[nr][nc] == 1 || visited[nr, nc]) continue;
            visited[nr, nc] = true;
            queue.Enqueue((nr, nc, dist + 1));
        }
    }
    return -1;
}
```

---

## 6. Heap / PriorityQueue Template

```csharp
// Top K largest elements — O(n log k)
static int[] TopKLargest(int[] nums, int k)
{
    // Min-heap of size k
    var pq = new PriorityQueue<int, int>();
    foreach (int n in nums)
    {
        pq.Enqueue(n, n);           // priority = value
        if (pq.Count > k) pq.Dequeue();  // remove smallest
    }
    var result = new int[k];
    for (int i = k - 1; i >= 0; i--)
        result[i] = pq.Dequeue();
    return result;
}

// Kth smallest — O(n log k) with max-heap trick
static int KthSmallest(int[] nums, int k)
{
    // Max-heap: negate priority to simulate max
    var pq = new PriorityQueue<int, int>();
    foreach (int n in nums)
    {
        pq.Enqueue(n, -n);          // negate = max-heap
        if (pq.Count > k) pq.Dequeue();
    }
    return pq.Peek();
}
```

---

## 7. Binary Search Template

```csharp
// Standard — find exact target
static int BinarySearch(int[] nums, int target)
{
    int lo = 0, hi = nums.Length - 1;
    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2;   // avoids overflow
        if      (nums[mid] == target) return mid;
        else if (nums[mid] < target)  lo = mid + 1;
        else                          hi = mid - 1;
    }
    return -1;
}

// Left boundary — first position where nums[i] >= target
static int LowerBound(int[] nums, int target)
{
    int lo = 0, hi = nums.Length;
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] < target) lo = mid + 1;
        else                    hi = mid;
    }
    return lo;  // insert position
}

// Search on answer space — "minimum valid X"
static int MinimumValid(int lo, int hi, Func<int, bool> feasible)
{
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (feasible(mid)) hi = mid;
        else               lo = mid + 1;
    }
    return lo;
}
```

---

## 8. Prefix Sum Template

```csharp
// Build prefix sums — O(n), then range queries in O(1)
static int[] BuildPrefix(int[] nums)
{
    var prefix = new int[nums.Length + 1];
    for (int i = 0; i < nums.Length; i++)
        prefix[i + 1] = prefix[i] + nums[i];
    return prefix;
}

// Range sum [l, r] inclusive
static int RangeSum(int[] prefix, int l, int r)
    => prefix[r + 1] - prefix[l];

// Count subarrays with sum = k (prefix + hashmap)
static int SubarraySumEqualsK(int[] nums, int k)
{
    var counts = new Dictionary<int, int> { [0] = 1 };
    int runSum = 0, result = 0;
    foreach (int n in nums)
    {
        runSum += n;
        result += counts.GetValueOrDefault(runSum - k);
        counts[runSum] = counts.GetValueOrDefault(runSum) + 1;
    }
    return result;
}
```

---

## 9. Dynamic Programming Templates

### 1D DP

```csharp
// Climbing stairs — n steps, take 1 or 2 at a time
static int ClimbStairs(int n)
{
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;
    for (int i = 3; i <= n; i++)
        (prev2, prev1) = (prev1, prev2 + prev1);
    return prev1;
}

// Coin change — minimum coins for amount
static int CoinChange(int[] coins, int amount)
{
    var dp = new int[amount + 1];
    Array.Fill(dp, int.MaxValue / 2);
    dp[0] = 0;
    for (int a = 1; a <= amount; a++)
        foreach (int c in coins)
            if (c <= a) dp[a] = Math.Min(dp[a], dp[a - c] + 1);
    return dp[amount] >= int.MaxValue / 2 ? -1 : dp[amount];
}
```

### 2D DP

```csharp
// Longest common subsequence
static int LCS(string s, string t)
{
    int m = s.Length, n = t.Length;
    var dp = new int[m + 1, n + 1];
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            dp[i, j] = s[i-1] == t[j-1]
                ? dp[i-1, j-1] + 1
                : Math.Max(dp[i-1, j], dp[i, j-1]);
    return dp[m, n];
}
```

---

## Interview Edge-Case Checklist

```
Input validation:
  □ Empty array / string / null input?
  □ Single element?
  □ All elements the same?
  □ Negative numbers? Zero?
  □ Integer overflow? (use long, checked, or Math.Clamp)

Array / string:
  □ Off-by-one in indices?
  □ lo + (hi - lo) / 2 to avoid overflow in binary search
  □ Last element not processed in sliding window?

Graph / Tree:
  □ Disconnected graph?
  □ Cycles — need visited set?
  □ Empty tree (null root)?

DP:
  □ Base cases initialized?
  □ Iteration order correct (forward / backward)?
  □ Space optimization possible (rolling array)?
```

---

## Common C# Mistakes in Interviews

```csharp
// ❌ Array sort without specifying comparator (strings sort lexicographically)
Array.Sort(strArr);              // "10" < "9" in string sort!
Array.Sort(intArr, (a, b) => a.CompareTo(b));  // ✅ explicit for clarity

// ❌ Modifying a collection while iterating
foreach (var item in list)
    if (condition) list.Remove(item);  // InvalidOperationException

// ✅ Iterate backwards or build a remove list
for (int i = list.Count - 1; i >= 0; i--)
    if (condition) list.RemoveAt(i);

// ❌ Dictionary not checking key exists
dict[key]++;                     // KeyNotFoundException

// ✅ Use GetValueOrDefault or TryGetValue
dict[key] = dict.GetValueOrDefault(key) + 1;
dict.TryGetValue(key, out int v); dict[key] = v + 1;

// ❌ int overflow in binary search midpoint
int mid = (lo + hi) / 2;        // overflows for large lo + hi

// ✅
int mid = lo + (hi - lo) / 2;

// ❌ Assuming List<int>.Sort() is stable — it isn't in all cases
// ✅ Use LINQ .OrderBy() which is stable (Merge sort based)

// C# PriorityQueue is a MIN-heap by default
// For max-heap: negate the priority value
```

---

## C# Collections Quick Reference

| Need | Type | Key Operations |
|------|------|---------------|
| Stack | `Stack<T>` | `Push`, `Pop`, `Peek` |
| Queue | `Queue<T>` | `Enqueue`, `Dequeue`, `Peek` |
| Min-heap | `PriorityQueue<T,P>` | `Enqueue(val, priority)`, `Dequeue`, `Peek` |
| Hash map | `Dictionary<K,V>` | `[]`, `TryGetValue`, `GetValueOrDefault` |
| Hash set | `HashSet<T>` | `Add`, `Contains`, `Remove` |
| Sorted map | `SortedDictionary<K,V>` | same as Dictionary, ordered |
| Sorted set | `SortedSet<T>` | `Add`, `Min`, `Max`, `GetViewBetween` |
| Deque | `LinkedList<T>` | `AddFirst`, `AddLast`, `RemoveFirst`, `RemoveLast` |

---

**Guide Complete!** Pattern recognition + language-specific gotcha awareness is what separates good interview performance from great. Practice these templates until they're automatic. 📘
