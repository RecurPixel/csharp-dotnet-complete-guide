# 💼 INTERVIEW TRACK - TRACK B: ARRAY ALGORITHMS
## 15 Problems with Guidance Only (NO Solutions)

**Purpose**: Master array manipulation - most common interview topic  
**Difficulty**: ⭐⭐ Easy → ⭐⭐⭐⭐ Hard  
**Time per problem**: 20-45 minutes  

---

## Problem 163: Two Sum ⭐⭐
[See previous example file - already completed]

---

## Problem 164: Three Sum ⭐⭐⭐

**Problem Statement:**

Given an integer array `nums`, return all triplets `[nums[i], nums[j], nums[k]]` such that:
- i != j, i != k, and j != k
- nums[i] + nums[j] + nums[k] = 0

The solution set must not contain duplicate triplets.

**Examples:**
```
Input: nums = [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]

Input: nums = [0,1,1]
Output: []

Input: nums = [0,0,0]
Output: [[0,0,0]]
```

**Constraints:**
- 3 ≤ nums.length ≤ 3000
- -10⁵ ≤ nums[i] ≤ 10⁵

---

**Approach: Sort + Two Pointers**

**Key Insight:**
- Fix one number, find two others that sum to its negative
- This becomes Two Sum problem for each fixed number!

**Concept:**
1. Sort the array
2. For each number (as first number):
   - Use two pointers for remaining array
   - Find pairs that sum to -(first number)
3. Skip duplicates to avoid duplicate triplets

**Complexity:**
- Time: O(n²) - O(n log n) sort + O(n²) for nested loops
- Space: O(1) excluding output

**Hints:**
```csharp
Array.Sort(nums);

for (int i = 0; i < nums.Length - 2; i++)
{
    // Skip duplicates for first number
    if (i > 0 && nums[i] == nums[i-1]) continue;
    
    int left = i + 1;
    int right = nums.Length - 1;
    int target = -nums[i];
    
    while (left < right)
    {
        int sum = nums[left] + nums[right];
        
        if (sum == target)
        {
            // Found triplet!
            // Add to result
            // Skip duplicates for left
            // Skip duplicates for right
            left++;
            right--;
        }
        else if (sum < target)
        {
            left++;
        }
        else
        {
            right--;
        }
    }
}
```

---

**Test Cases:**
```csharp
[-1,0,1,2,-1,-4] → [[-1,-1,2],[-1,0,1]]
[0,1,1] → []
[0,0,0] → [[0,0,0]]
[-2,0,1,1,2] → [[-2,0,2],[-2,1,1]]
```

**Common Mistakes:**
- Not sorting first
- Not skipping duplicates (causes duplicate triplets)
- Off-by-one errors with pointers
- Forgetting edge cases (all zeros, all same numbers)

**Interview Tips:**
- Explain it builds on Two Sum
- Draw the pointer movement
- Emphasize duplicate handling
- Mention Four Sum as follow-up

---

## Problem 165: Subarray with Given Sum ⭐⭐⭐

**Problem Statement:**

Given an array of positive integers and a target sum, find a continuous subarray that sums to the target. Return the start and end indices.

**Examples:**
```
Input: arr = [1,2,3,7,5], sum = 12
Output: [1, 3] (subarray [2,3,7])

Input: arr = [1,2,3,4,5,6,7,8,9,10], sum = 15
Output: [0, 4] (subarray [1,2,3,4,5])

Input: arr = [1,2,3], sum = 10
Output: [] (no subarray found)
```

**Constraints:**
- Array contains only positive integers
- 1 ≤ arr.length ≤ 10⁵

---

**Approach: Sliding Window**

**Key Insight:**
- Since all numbers are positive, we can use sliding window
- If sum too small → expand window (add right)
- If sum too large → shrink window (remove left)

**Concept:**
- Two pointers: left and right
- Maintain current sum
- Adjust window based on sum comparison

**Complexity:**
- Time: O(n) - each element added and removed at most once
- Space: O(1)

**Hints:**
```csharp
int left = 0, currentSum = 0;

for (int right = 0; right < arr.Length; right++)
{
    currentSum += arr[right];
    
    // Shrink window if sum too large
    while (currentSum > targetSum && left <= right)
    {
        currentSum -= arr[left];
        left++;
    }
    
    // Check if found
    if (currentSum == targetSum)
    {
        return new int[] { left, right };
    }
}

return new int[] {}; // Not found
```

---

**What if array has negative numbers?**
- Sliding window doesn't work!
- Need different approach: prefix sum + hash map
- Time: O(n), Space: O(n)

---

**Test Cases:**
```csharp
([1,2,3,7,5], 12) → [1,3]
([1,2,3,4,5,6,7,8,9,10], 15) → [0,4]
([1,2,3], 10) → []
([5], 5) → [0,0]
([1,1,1,1,1], 3) → [0,2]
```

**Interview Tips:**
- Clarify if array has negatives (changes approach!)
- Explain sliding window technique
- Mention it works for positive numbers only
- Discuss prefix sum approach for negatives

---

## Problem 166: Equilibrium Index ⭐⭐⭐

**Problem Statement:**

Find an index where the sum of elements on the left equals the sum of elements on the right.

**Examples:**
```
Input: arr = [-7, 1, 5, 2, -4, 3, 0]
Output: 3 (index 3: left sum = -7+1+5 = -1, right sum = -4+3+0 = -1)

Input: arr = [1, 2, 3]
Output: -1 (no equilibrium)

Input: arr = [1]
Output: 0 (only element)
```

---

**Approach: Prefix Sum**

**Key Insight:**
- leftSum + arr[i] + rightSum = totalSum
- If leftSum = rightSum, then:
  - leftSum = (totalSum - arr[i]) / 2

**Concept:**
1. Calculate total sum
2. Iterate, maintaining left sum
3. Right sum = total - left - current
4. Check if left == right

**Complexity:**
- Time: O(n)
- Space: O(1)

**Hints:**
```csharp
int totalSum = arr.Sum();
int leftSum = 0;

for (int i = 0; i < arr.Length; i++)
{
    // Right sum = total - left - current
    int rightSum = totalSum - leftSum - arr[i];
    
    if (leftSum == rightSum)
        return i;
    
    leftSum += arr[i];
}

return -1;
```

---

**Test Cases:**
```csharp
[-7,1,5,2,-4,3,0] → 3
[1,2,3] → -1
[1] → 0
[0,0,0] → 0 (or 1 or 2, return any)
[-1,-1,2,1,-1] → 2
```

**Interview Tips:**
- Explain prefix sum concept
- Mention two-pass vs one-pass
- Discuss edge case: single element

---

## Problem 167: Leaders in Array ⭐⭐

**Problem Statement:**

An element is a leader if it's greater than all elements to its right. Find all leaders.

**Examples:**
```
Input: arr = [16, 17, 4, 3, 5, 2]
Output: [17, 5, 2]
Explanation: 17 > all right, 5 > 2, 2 is rightmost

Input: arr = [1, 2, 3, 4, 5]
Output: [5] (increasing array)

Input: arr = [5, 4, 3, 2, 1]
Output: [5, 4, 3, 2, 1] (all are leaders)
```

---

**Approach: Right to Left Scan**

**Key Insight:**
- Rightmost element is always a leader
- Scan from right to left, tracking max seen so far
- Any element > max is a leader

**Concept:**
- Start from right
- Track maximum element seen
- If current > max, it's a leader

**Complexity:**
- Time: O(n)
- Space: O(1) excluding output

**Hints:**
```csharp
var leaders = new List<int>();
int maxRight = int.MinValue;

// Scan right to left
for (int i = arr.Length - 1; i >= 0; i--)
{
    if (arr[i] > maxRight)
    {
        leaders.Add(arr[i]);
        maxRight = arr[i];
    }
}

// Reverse to get left-to-right order
leaders.Reverse();
return leaders;
```

---

**Test Cases:**
```csharp
[16,17,4,3,5,2] → [17,5,2]
[1,2,3,4,5] → [5]
[5,4,3,2,1] → [5,4,3,2,1]
[5] → [5]
[1,1,1,1] → [1]
```

**Interview Tips:**
- Explain right-to-left approach
- Mention why left-to-right is harder (O(n²))
- Discuss whether order matters in output

---

## Problem 168: Trapping Rainwater ⭐⭐⭐⭐

**Problem Statement:**

Given n non-negative integers representing elevation map where width of each bar is 1, compute how much water can be trapped after raining.

**Examples:**
```
Input: height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6

Input: height = [4,2,0,3,2,5]
Output: 9
```

---

**Approach 1: Dynamic Programming**

**Key Insight:**
- Water trapped at index i = min(maxLeft[i], maxRight[i]) - height[i]
- Precompute max heights to left and right of each position

**Complexity:**
- Time: O(n)
- Space: O(n) - two arrays

**Hints:**
```csharp
int n = height.Length;
int[] leftMax = new int[n];
int[] rightMax = new int[n];

// Fill leftMax
leftMax[0] = height[0];
for (int i = 1; i < n; i++)
    leftMax[i] = Math.Max(leftMax[i-1], height[i]);

// Fill rightMax
rightMax[n-1] = height[n-1];
for (int i = n-2; i >= 0; i--)
    rightMax[i] = Math.Max(rightMax[i+1], height[i]);

// Calculate water
int water = 0;
for (int i = 0; i < n; i++)
{
    water += Math.Min(leftMax[i], rightMax[i]) - height[i];
}
```

---

**Approach 2: Two Pointers (Optimal)**

**Key Insight:**
- Don't need full arrays if we use two pointers
- Process from both ends moving toward center

**Complexity:**
- Time: O(n)
- Space: O(1)

**Hints:**
```csharp
int left = 0, right = height.Length - 1;
int leftMax = 0, rightMax = 0;
int water = 0;

while (left < right)
{
    if (height[left] < height[right])
    {
        if (height[left] >= leftMax)
            leftMax = height[left];
        else
            water += leftMax - height[left];
        left++;
    }
    else
    {
        if (height[right] >= rightMax)
            rightMax = height[right];
        else
            water += rightMax - height[right];
        right--;
    }
}
```

---

**Test Cases:**
```csharp
[0,1,0,2,1,0,1,3,2,1,2,1] → 6
[4,2,0,3,2,5] → 9
[3,0,2,0,4] → 7
[] → 0
[3] → 0
[3,2,1] → 0
```

**Interview Tips:**
- Draw visual representation!
- Start with DP approach (easier to explain)
- Then optimize to two pointers
- This is a hard problem - don't worry if stuck

---

## Problem 169: Sort 0s, 1s, 2s (Dutch Flag) ⭐⭐⭐

**Problem Statement:**

Given an array containing only 0s, 1s, and 2s, sort it in-place without using sort function.

**Examples:**
```
Input: arr = [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]

Input: arr = [2,0,1]
Output: [0,1,2]

Input: arr = [0]
Output: [0]
```

---

**Approach 1: Counting**

**Concept:**
- Count occurrences of 0, 1, 2
- Overwrite array with counts

**Complexity:**
- Time: O(n)
- Space: O(1)

**Hints:**
```csharp
int count0 = 0, count1 = 0, count2 = 0;

// Count
foreach (int num in arr)
{
    if (num == 0) count0++;
    else if (num == 1) count1++;
    else count2++;
}

// Overwrite
int i = 0;
while (count0-- > 0) arr[i++] = 0;
while (count1-- > 0) arr[i++] = 1;
while (count2-- > 0) arr[i++] = 2;
```

---

**Approach 2: Dutch National Flag (Single Pass)**

**Key Insight:**
- Three pointers: low (next position for 0), mid (current), high (next position for 2)
- Partition array into three sections

**Concept:**
- 0s go to low region
- 2s go to high region
- 1s stay in middle

**Complexity:**
- Time: O(n) - single pass
- Space: O(1)

**Hints:**
```csharp
int low = 0, mid = 0, high = arr.Length - 1;

while (mid <= high)
{
    if (arr[mid] == 0)
    {
        Swap(arr, low, mid);
        low++;
        mid++;
    }
    else if (arr[mid] == 1)
    {
        mid++;
    }
    else // arr[mid] == 2
    {
        Swap(arr, mid, high);
        high--;
        // Don't increment mid! Need to check swapped element
    }
}
```

---

**Test Cases:**
```csharp
[2,0,2,1,1,0] → [0,0,1,1,2,2]
[2,0,1] → [0,1,2]
[0] → [0]
[2,2,2,2] → [2,2,2,2]
[0,1,2,0,1,2] → [0,0,1,1,2,2]
```

**Interview Tips:**
- Show both approaches
- Dutch flag is more elegant (single pass)
- Explain why we don't increment mid when swapping with high
- Mention this generalizes to k colors

---

## Problem 170: Find Majority Element ⭐⭐⭐

**Problem Statement:**

Given an array, find the element that appears more than ⌊n/2⌋ times. You may assume such element always exists.

**Examples:**
```
Input: nums = [3,2,3]
Output: 3

Input: nums = [2,2,1,1,1,2,2]
Output: 2
```

---

**Approach 1: Hash Map**

**Concept:**
- Count frequencies
- Find element with count > n/2

**Complexity:**
- Time: O(n)
- Space: O(n)

---

**Approach 2: Boyer-Moore Voting Algorithm (Optimal)**

**Key Insight:**
- Majority element appears more than all others combined
- Cancel out different elements
- Remaining is majority

**Concept:**
- Candidate and count
- If same as candidate, count++
- If different, count--
- If count = 0, new candidate

**Complexity:**
- Time: O(n)
- Space: O(1)

**Hints:**
```csharp
int candidate = nums[0];
int count = 1;

for (int i = 1; i < nums.Length; i++)
{
    if (count == 0)
    {
        candidate = nums[i];
        count = 1;
    }
    else if (nums[i] == candidate)
    {
        count++;
    }
    else
    {
        count--;
    }
}

return candidate;
```

---

**Why this works:**
- Majority element survives cancellation
- Even if all others team up, majority wins!

---

**Test Cases:**
```csharp
[3,2,3] → 3
[2,2,1,1,1,2,2] → 2
[1] → 1
[1,1,1,1,2,2,2] → 1
```

**Interview Tips:**
- Explain both approaches
- Boyer-Moore is brilliant but non-obvious
- Walk through example showing cancellation
- This is a classic algorithm!

---

## Problem 171: Kth Largest Element ⭐⭐⭐

**Problem Statement:**

Find the kth largest element in an unsorted array. Note that it is the kth largest element in sorted order, not the kth distinct element.

**Examples:**
```
Input: nums = [3,2,1,5,6,4], k = 2
Output: 5

Input: nums = [3,2,3,1,2,4,5,5,6], k = 4
Output: 4
```

---

**Approach 1: Sort**

**Concept:**
- Sort array
- Return element at index (length - k)

**Complexity:**
- Time: O(n log n)
- Space: O(1)

---

**Approach 2: Min Heap of Size K**

**Concept:**
- Maintain heap of k largest elements
- Top of heap is kth largest

**Complexity:**
- Time: O(n log k)
- Space: O(k)

**Hints:**
```csharp
var minHeap = new PriorityQueue<int, int>();

foreach (int num in nums)
{
    minHeap.Enqueue(num, num);
    
    if (minHeap.Count > k)
        minHeap.Dequeue();
}

return minHeap.Peek();
```

---

**Approach 3: QuickSelect (Optimal)**

**Concept:**
- Like QuickSort but only recurse on one side
- Partition array, check if pivot is kth largest

**Complexity:**
- Time: O(n) average, O(n²) worst
- Space: O(1)

**This is advanced - mention but implementation is tricky**

---

**Test Cases:**
```csharp
([3,2,1,5,6,4], 2) → 5
([3,2,3,1,2,4,5,5,6], 4) → 4
([1], 1) → 1
([1,2,3,4,5], 1) → 5
```

**Interview Tips:**
- Start with sort approach (simple)
- Then min heap (better for small k)
- Mention QuickSelect (optimal but complex)
- Ask: "Do you want me to implement QuickSelect?"

---

## Problem 172: Next Permutation ⭐⭐⭐⭐

**Problem Statement:**

Implement next permutation, which rearranges numbers into the lexicographically next greater permutation.

If not possible (highest permutation), rearrange to lowest (sorted ascending).

Must be in-place with O(1) extra space.

**Examples:**
```
Input: nums = [1,2,3]
Output: [1,3,2]

Input: nums = [3,2,1]
Output: [1,2,3]

Input: nums = [1,1,5]
Output: [1,5,1]
```

---

**Approach: Single Pass Algorithm**

**Key Insight:**
1. Find rightmost pair where nums[i] < nums[i+1]
2. Swap nums[i] with smallest element to its right that's larger
3. Reverse everything after i

**Concept:**
```
Example: [1,3,5,4,2]
Step 1: Find i where nums[i] < nums[i+1]
        i=1 (3 < 5)
Step 2: Find smallest element > 3 to the right
        That's 4
Step 3: Swap 3 and 4: [1,4,5,3,2]
Step 4: Reverse after i: [1,4,2,3,5]
```

**Complexity:**
- Time: O(n)
- Space: O(1)

**Hints:**
```csharp
// Step 1: Find pivot (rightmost i where nums[i] < nums[i+1])
int i = nums.Length - 2;
while (i >= 0 && nums[i] >= nums[i+1])
    i--;

if (i >= 0)
{
    // Step 2: Find smallest element > nums[i] to the right
    int j = nums.Length - 1;
    while (nums[j] <= nums[i])
        j--;
    
    // Step 3: Swap
    Swap(nums, i, j);
}

// Step 4: Reverse elements after i
Reverse(nums, i + 1, nums.Length - 1);
```

---

**Test Cases:**
```csharp
[1,2,3] → [1,3,2]
[3,2,1] → [1,2,3]
[1,1,5] → [1,5,1]
[1,3,2] → [2,1,3]
[1] → [1]
```

**Interview Tips:**
- This is a hard problem
- Draw out the steps
- Explain lexicographic order
- The algorithm is non-obvious - OK to struggle

---

## Problem 173: Merge Intervals ⭐⭐⭐

**Problem Statement:**

Given a collection of intervals, merge all overlapping intervals.

**Examples:**
```
Input: intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]

Input: intervals = [[1,4],[4,5]]
Output: [[1,5]]
```

---

**Approach: Sort + Merge**

**Key Insight:**
- Sort intervals by start time
- Merge consecutive overlapping intervals

**Concept:**
1. Sort by start time
2. Iterate, checking if current overlaps with previous
3. If overlaps, merge (extend end)
4. If not, add previous and start new interval

**Complexity:**
- Time: O(n log n) - sorting
- Space: O(n) - output

**Hints:**
```csharp
// Sort by start time
Array.Sort(intervals, (a, b) => a[0].CompareTo(b[0]));

var merged = new List<int[]>();
int[] current = intervals[0];

foreach (var interval in intervals)
{
    if (interval[0] <= current[1])
    {
        // Overlaps - merge
        current[1] = Math.Max(current[1], interval[1]);
    }
    else
    {
        // No overlap - add current, start new
        merged.Add(current);
        current = interval;
    }
}

merged.Add(current); // Don't forget last interval!
return merged.ToArray();
```

---

**Test Cases:**
```csharp
[[1,3],[2,6],[8,10],[15,18]] → [[1,6],[8,10],[15,18]]
[[1,4],[4,5]] → [[1,5]]
[[1,4],[0,4]] → [[0,4]]
[[1,4],[2,3]] → [[1,4]] (one inside other)
```

**Interview Tips:**
- Always sort first!
- Draw timeline visualization
- Handle edge case: one interval inside another
- Mention insert interval as follow-up

---

## Problem 174: Longest Increasing Subsequence ⭐⭐⭐⭐

**Problem Statement:**

Given an integer array nums, return the length of the longest strictly increasing subsequence.

Note: Subsequence != subarray (elements don't need to be contiguous)

**Examples:**
```
Input: nums = [10,9,2,5,3,7,101,18]
Output: 4
Explanation: [2,3,7,101] or [2,3,7,18]

Input: nums = [0,1,0,3,2,3]
Output: 4

Input: nums = [7,7,7,7,7,7,7]
Output: 1
```

---

**Approach 1: Dynamic Programming**

**Concept:**
- dp[i] = length of LIS ending at index i
- For each i, check all previous elements
- If nums[j] < nums[i], can extend LIS at j

**Complexity:**
- Time: O(n²)
- Space: O(n)

**Hints:**
```csharp
int[] dp = new int[nums.Length];
Array.Fill(dp, 1); // Each element is LIS of length 1

for (int i = 1; i < nums.Length; i++)
{
    for (int j = 0; j < i; j++)
    {
        if (nums[j] < nums[i])
        {
            dp[i] = Math.Max(dp[i], dp[j] + 1);
        }
    }
}

return dp.Max();
```

---

**Approach 2: Binary Search (Optimal)**

**Concept:**
- Maintain array of smallest tail values for each length
- Use binary search to find position

**Complexity:**
- Time: O(n log n)
- Space: O(n)

**This is advanced - mention but don't worry if can't implement**

---

**Test Cases:**
```csharp
[10,9,2,5,3,7,101,18] → 4
[0,1,0,3,2,3] → 4
[7,7,7,7,7,7,7] → 1
[1,3,6,7,9,4,10,5,6] → 6
```

**Interview Tips:**
- DP approach is expected
- Draw the dp array
- Binary search optimization is bonus
- This is a classic hard problem

---

## Problem 175: Stock Buy/Sell (One Transaction) ⭐⭐

**Problem Statement:**

You are given an array prices where prices[i] is the price of a stock on day i.

Find maximum profit from one buy and one sell. If no profit possible, return 0.

**Examples:**
```
Input: prices = [7,1,5,3,6,4]
Output: 5 (buy at 1, sell at 6)

Input: prices = [7,6,4,3,1]
Output: 0 (no profit possible)
```

---

**Approach: Single Pass**

**Key Insight:**
- Track minimum price seen so far
- Calculate profit if sold today
- Update maximum profit

**Concept:**
- Keep track of lowest price
- For each day, calculate: price today - lowest price
- Update max profit

**Complexity:**
- Time: O(n)
- Space: O(1)

**Hints:**
```csharp
int minPrice = int.MaxValue;
int maxProfit = 0;

foreach (int price in prices)
{
    minPrice = Math.Min(minPrice, price);
    
    int profit = price - minPrice;
    maxProfit = Math.Max(maxProfit, profit);
}

return maxProfit;
```

---

**Test Cases:**
```csharp
[7,1,5,3,6,4] → 5
[7,6,4,3,1] → 0
[2,4,1] → 2
[3,3,5,0,0,3,1,4] → 4
```

**Follow-up Questions:**
- Multiple transactions? (Different problem - DP)
- At most k transactions? (Hard DP problem)
- With transaction fee? (Modified DP)

**Interview Tips:**
- This is easier than it looks
- One pass is sufficient
- Explain why greedy works here
- Mention follow-ups to show broader knowledge

---

## ✅ Track B Complete!

You've covered 15 essential array algorithm problems. These are the MOST common in interviews!

**Next**: Track C - Searching & Sorting (10 problems)

