# C# Algorithm Templates Quick Reference

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                   ALGORITHM SELECTION GUIDE                              │
│                                                                          │
│  Problem Type            Algorithm          Complexity                   │
│  ────────────────────────────────────────────────────────────────────── │
│  Find in sorted array    Binary Search      O(log n)                     │
│  Order elements          Sort               O(n log n)                   │
│  Shortest unweighted     BFS                O(V + E)                     │
│  Explore all paths       DFS                O(V + E)                     │
│  Shortest weighted       Dijkstra           O((V+E) log V)               │
│  Top K elements          Heap               O(n log k)                   │
│  Dynamic sets            Union-Find         O(α(n)) ≈ O(1)              │
│  Repeated subproblems    DP                 O(n) to O(n×m)              │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Binary Search Variants

### Standard — Find Target

```csharp
// Returns index of target, or -1
static int BinarySearch(int[] arr, int target)
{
    int lo = 0, hi = arr.Length - 1;
    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2;
        if      (arr[mid] == target) return mid;
        else if (arr[mid] < target)  lo = mid + 1;
        else                         hi = mid - 1;
    }
    return -1;
}
```

### Lower Bound — First Position ≥ Target

```csharp
// Returns index where target should be inserted to keep sorted order
// Same as C++ lower_bound
static int LowerBound(int[] arr, int target)
{
    int lo = 0, hi = arr.Length;
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < target) lo = mid + 1;
        else                   hi = mid;
    }
    return lo;
}
```

### Upper Bound — First Position > Target

```csharp
static int UpperBound(int[] arr, int target)
{
    int lo = 0, hi = arr.Length;
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] <= target) lo = mid + 1;
        else                    hi = mid;
    }
    return lo;
}

// Count occurrences of target
int count = UpperBound(arr, target) - LowerBound(arr, target);
```

### Binary Search on Answer

```csharp
// "What is the minimum/maximum X that satisfies condition?"
// Feasibility function drives the search

// Example: minimum speed to eat bananas in h hours
static int MinEatingSpeed(int[] piles, int h)
{
    static bool CanFinish(int[] piles, int speed, int h)
    {
        long total = 0;
        foreach (int p in piles)
            total += (p + speed - 1) / speed;  // ceiling division
        return total <= h;
    }

    int lo = 1, hi = piles.Max();
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (CanFinish(piles, mid, h)) hi = mid;
        else                          lo = mid + 1;
    }
    return lo;
}
```

---

## 2. Sorting Templates

```csharp
// Built-in IntroSort — fastest general purpose
Array.Sort(arr);
list.Sort();

// Custom comparison — sort by multiple fields
Array.Sort(intervals, (a, b) => a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]);

// Stable sort (LINQ — preserves equal element order)
var sorted = items.OrderBy(x => x.Priority)
                  .ThenBy(x => x.Name)
                  .ToList();

// Sort with Comparer<T>.Create
var comparer = Comparer<string>.Create((a, b) =>
    a.Length != b.Length ? a.Length.CompareTo(b.Length) : string.Compare(a, b, StringComparison.Ordinal));
Array.Sort(strings, comparer);
```

### When to Use Which Sort

| Situation | Use |
|-----------|-----|
| General purpose | `Array.Sort()` / `List.Sort()` — O(n log n) IntroSort |
| Stable sort | LINQ `OrderBy()` — O(n log n) merge-sort based |
| Nearly sorted | `Array.Sort()` still fine |
| Small range integers | Counting sort — O(n + k) |
| Large objects by key | `OrderBy(x => x.Key)` with selector |

---

## 3. DFS Templates

### DFS on Graph (Iterative — Avoids Stack Overflow)

```csharp
static void DfsIterative(Dictionary<int, List<int>> graph, int start)
{
    var visited = new HashSet<int>();
    var stack   = new Stack<int>();
    stack.Push(start);

    while (stack.Count > 0)
    {
        int node = stack.Pop();
        if (!visited.Add(node)) continue;  // Add returns false if already present

        Console.WriteLine(node);
        foreach (int neighbor in graph.GetValueOrDefault(node, []))
            if (!visited.Contains(neighbor))
                stack.Push(neighbor);
    }
}
```

### DFS on Graph (Recursive)

```csharp
static void DfsRecursive(
    Dictionary<int, List<int>> graph,
    int node,
    HashSet<int> visited)
{
    if (!visited.Add(node)) return;
    Console.WriteLine(node);
    foreach (int neighbor in graph.GetValueOrDefault(node, []))
        DfsRecursive(graph, neighbor, visited);
}
```

### DFS on Grid

```csharp
static void DfsGrid(char[][] grid, int r, int c, bool[][] visited)
{
    int rows = grid.Length, cols = grid[0].Length;
    if (r < 0 || r >= rows || c < 0 || c >= cols) return;
    if (visited[r][c] || grid[r][c] == '0') return;

    visited[r][c] = true;
    int[][] dirs = [[-1,0],[1,0],[0,-1],[0,1]];
    foreach (var d in dirs)
        DfsGrid(grid, r + d[0], c + d[1], visited);
}
```

### DFS with Backtracking (Combinations / Permutations)

```csharp
// Generate all subsets
static List<List<int>> Subsets(int[] nums)
{
    var result = new List<List<int>>();
    Backtrack(nums, 0, [], result);
    return result;
}

static void Backtrack(int[] nums, int start, List<int> current, List<List<int>> result)
{
    result.Add([..current]);
    for (int i = start; i < nums.Length; i++)
    {
        current.Add(nums[i]);
        Backtrack(nums, i + 1, current, result);
        current.RemoveAt(current.Count - 1);  // undo
    }
}
```

---

## 4. BFS Template

```csharp
// Generic BFS — returns shortest distance to each reachable node
static Dictionary<int, int> Bfs(Dictionary<int, List<int>> graph, int start)
{
    var distances = new Dictionary<int, int> { [start] = 0 };
    var queue     = new Queue<int>();
    queue.Enqueue(start);

    while (queue.Count > 0)
    {
        int node = queue.Dequeue();
        foreach (int neighbor in graph.GetValueOrDefault(node, []))
        {
            if (!distances.ContainsKey(neighbor))
            {
                distances[neighbor] = distances[node] + 1;
                queue.Enqueue(neighbor);
            }
        }
    }
    return distances;
}

// Multi-source BFS (start from multiple nodes simultaneously)
static int[,] MultiSourceBfs(int[,] grid, List<(int r, int c)> sources)
{
    int rows = grid.GetLength(0), cols = grid.GetLength(1);
    var dist  = new int[rows, cols];
    Array.Fill2D(dist, int.MaxValue);  // custom helper or manual init

    var queue = new Queue<(int r, int c)>();
    foreach (var (r, c) in sources)
    {
        dist[r, c] = 0;
        queue.Enqueue((r, c));
    }

    int[][] dirs = [[-1,0],[1,0],[0,-1],[0,1]];
    while (queue.Count > 0)
    {
        var (r, c) = queue.Dequeue();
        foreach (var d in dirs)
        {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
            if (dist[nr, nc] != int.MaxValue) continue;
            dist[nr, nc] = dist[r, c] + 1;
            queue.Enqueue((nr, nc));
        }
    }
    return dist;
}
```

---

## 5. Heap Top-K Template

```csharp
// Top K frequent elements
static int[] TopKFrequent(int[] nums, int k)
{
    var freq = new Dictionary<int, int>();
    foreach (int n in nums)
        freq[n] = freq.GetValueOrDefault(n) + 1;

    // Min-heap by frequency — keep only k largest
    var pq = new PriorityQueue<int, int>();
    foreach (var (num, count) in freq)
    {
        pq.Enqueue(num, count);
        if (pq.Count > k) pq.Dequeue();
    }

    var result = new int[k];
    for (int i = k - 1; i >= 0; i--)
        result[i] = pq.Dequeue();
    return result;
}

// Merge K sorted lists — heap by current head value
static List<int> MergeKSorted(List<List<int>> lists)
{
    // (value, listIndex, elementIndex)
    var pq     = new PriorityQueue<(int val, int li, int ei), int>();
    var result = new List<int>();

    for (int i = 0; i < lists.Count; i++)
        if (lists[i].Count > 0)
            pq.Enqueue((lists[i][0], i, 0), lists[i][0]);

    while (pq.Count > 0)
    {
        var (val, li, ei) = pq.Dequeue();
        result.Add(val);
        if (ei + 1 < lists[li].Count)
            pq.Enqueue((lists[li][ei + 1], li, ei + 1), lists[li][ei + 1]);
    }
    return result;
}
```

---

## 6. Union-Find (Disjoint Set Union)

```csharp
// Path compression + union by rank — near O(1) per operation
class UnionFind
{
    private int[] _parent, _rank;
    public int Components { get; private set; }

    public UnionFind(int n)
    {
        _parent    = Enumerable.Range(0, n).ToArray();
        _rank      = new int[n];
        Components = n;
    }

    public int Find(int x)
    {
        if (_parent[x] != x)
            _parent[x] = Find(_parent[x]);  // path compression
        return _parent[x];
    }

    public bool Union(int x, int y)
    {
        int px = Find(x), py = Find(y);
        if (px == py) return false;  // already connected
        if (_rank[px] < _rank[py]) (px, py) = (py, px);
        _parent[py] = px;
        if (_rank[px] == _rank[py]) _rank[px]++;
        Components--;
        return true;
    }

    public bool Connected(int x, int y) => Find(x) == Find(y);
}

// Usage — number of islands
static int NumIslands(char[][] grid)
{
    int r = grid.Length, c = grid[0].Length;
    var uf = new UnionFind(r * c);
    int water = 0;

    int[][] dirs = [[-1,0],[1,0],[0,-1],[0,1]];
    for (int i = 0; i < r; i++)
    for (int j = 0; j < c; j++)
    {
        if (grid[i][j] == '0') { water++; continue; }
        foreach (var d in dirs)
        {
            int ni = i + d[0], nj = j + d[1];
            if (ni >= 0 && ni < r && nj >= 0 && nj < c && grid[ni][nj] == '1')
                uf.Union(i * c + j, ni * c + nj);
        }
    }
    return uf.Components - water;
}
```

---

## 7. Dijkstra's Shortest Path

```csharp
// Returns shortest distances from start to all reachable nodes
// Graph: adjacency list — List<(neighbor, weight)>
static int[] Dijkstra(List<(int to, int w)>[] graph, int start, int n)
{
    var dist = new int[n];
    Array.Fill(dist, int.MaxValue);
    dist[start] = 0;

    // (distance, node) min-heap
    var pq = new PriorityQueue<int, int>();
    pq.Enqueue(start, 0);

    while (pq.Count > 0)
    {
        int node = pq.Dequeue();
        foreach (var (to, weight) in graph[node])
        {
            int newDist = dist[node] + weight;
            if (newDist < dist[to])
            {
                dist[to] = newDist;
                pq.Enqueue(to, newDist);
            }
        }
    }
    return dist;
}
```

---

## 8. DP Starter Templates

### Knapsack (0/1)

```csharp
// Can we reach exactly target weight with subset of weights?
static bool CanPartition(int[] nums, int target)
{
    var dp = new bool[target + 1];
    dp[0] = true;
    foreach (int num in nums)
        for (int j = target; j >= num; j--)  // iterate backwards for 0/1
            dp[j] |= dp[j - num];
    return dp[target];
}
```

### Longest Increasing Subsequence

```csharp
// O(n log n) — binary search patience sort
static int LIS(int[] nums)
{
    var tails = new List<int>();
    foreach (int n in nums)
    {
        int pos = LowerBound([..tails], n);
        if (pos == tails.Count) tails.Add(n);
        else                    tails[pos] = n;
    }
    return tails.Count;
}

static int LowerBound(int[] arr, int target)
{
    int lo = 0, hi = arr.Length;
    while (lo < hi) { int mid = lo + (hi - lo)/2; if (arr[mid] < target) lo = mid+1; else hi = mid; }
    return lo;
}
```

### 2D Grid DP — Unique Paths

```csharp
static int UniquePaths(int m, int n)
{
    var dp = new int[m, n];
    for (int i = 0; i < m; i++) dp[i, 0] = 1;
    for (int j = 0; j < n; j++) dp[0, j] = 1;
    for (int i = 1; i < m; i++)
    for (int j = 1; j < n; j++)
        dp[i, j] = dp[i-1, j] + dp[i, j-1];
    return dp[m-1, n-1];
}
```

---

## Best Practices

```
✅ Prefer iterative DFS over recursive when depth can be > 1000 (stack overflow risk)
✅ Always initialize dist[] to int.MaxValue in Dijkstra — not -1
✅ Use lo + (hi - lo) / 2 to prevent binary search overflow
✅ In backtracking, undo state immediately after recursive call
✅ Use HashSet.Add() return value for O(1) visited check
✅ PriorityQueue<T,P> is min-heap — negate priority for max-heap
✅ Chunk large DP tables: rolling array reduces O(n×m) space to O(m)
```

---

## Quick Reference Summary

| Algorithm | Template Location | Key Class/API |
|-----------|------------------|--------------|
| Binary search | Section 1 | Manual `lo/hi` loop |
| Sorting | Section 2 | `Array.Sort`, `OrderBy` |
| DFS iterative | Section 3 | `Stack<T>` |
| DFS recursive | Section 3 | Recursive + HashSet |
| Backtracking | Section 3 | Recursive + undo |
| BFS | Section 4 | `Queue<T>` |
| Top-K heap | Section 5 | `PriorityQueue<T,P>` |
| Union-Find | Section 6 | Custom class |
| Dijkstra | Section 7 | `PriorityQueue<T,P>` |
| DP 1D | Section 8 | `int[]` rolling array |
| DP 2D | Section 8 | `int[,]` |

---

**Guide Complete!** These templates are the 80% toolkit that covers the vast majority of algorithmic problems. Learn the templates, then adapt — not memorize individual problems. 📘
