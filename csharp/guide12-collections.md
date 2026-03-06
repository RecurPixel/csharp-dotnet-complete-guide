# C# Collections Quick Reference Guide

---

## Part 1: Collection Types Overview

### What are Collections?

**Definition:** Data structures that store and manage groups of objects

**Why Use Collections?**

- Store multiple values efficiently
- Dynamic sizing (grow/shrink as needed)
- Rich functionality (search, sort, filter)
- Type-safe with generics

**Categories:**

- **Lists** - Ordered, indexed access (Array, List\<T\>, ArrayList)
- **Dictionaries** - Key-value pairs (Dictionary, SortedDictionary, Hashtable)
- **Sets** - Unique elements (HashSet, SortedSet)
- **Queues/Stacks** - Specialized ordering (Queue, Stack)
- **Linked** - Node-based traversal (LinkedList)
- **Bit Collections** - Boolean flags (BitArray)

---

## Part 2: Arrays & Lists

### 1. Array

**Type:** Fixed-size, indexed, immutable length

**When to Use:**

- Know exact size in advance
- Performance critical (fastest access)
- Multi-dimensional data

**Namespace:** System

#### Creation

```csharp
// Empty array with default values
int[] arr1 = new int[5];              // [0, 0, 0, 0, 0]

// Array initializer
int[] arr2 = {1, 2, 3, 4, 5};

// Explicit array creation
int[] arr3 = new int[] {1, 2, 3};

// Multi-dimensional arrays
int[,] matrix = new int[3, 3];        // Rectangular array
int[][] jagged = new int[3][];        // Jagged array (array of arrays)
```

#### Access

```csharp
// Index access
int first = arr[0];
int last = arr[^1];                   // C# 8.0+ (Index from end)

// Range access (C# 8.0+)
int[] slice = arr[1..4];              // Elements 1, 2, 3

// Iteration
for (int i = 0; i < arr.Length; i++)
{
    Console.WriteLine(arr[i]);
}

foreach (int x in arr)
{
    Console.WriteLine(x);
}
```

#### Add/Remove

```csharp
// ❌ Cannot add/remove (fixed size)

// ✅ Resize creates NEW array (expensive)
Array.Resize(ref arr, 10);
```

#### Key Properties & Methods

```csharp
// Properties
arr.Length                    // Number of elements
arr.Rank                      // Number of dimensions (1 for single-dimensional)
arr.GetLength(0)              // Length of specific dimension
arr.IsFixedSize               // Always true
arr.IsReadOnly                // Usually false
arr.IsSynchronized            // false (not thread-safe)
arr.SyncRoot                  // Object for thread synchronization

// Methods
Array.Sort(arr);              // Sort in place
Array.Reverse(arr);           // Reverse order
Array.IndexOf(arr, value);    // Find first index of value
Array.LastIndexOf(arr, value);// Find last index of value
Array.Clear(arr, 0, arr.Length); // Set elements to default
Array.Copy(source, dest, length); // Copy elements
arr.Clone();                  // Shallow copy (returns object)
Array.Find(arr, predicate);   // Find first match
Array.FindAll(arr, predicate);// Find all matches
Array.Exists(arr, predicate); // Check if any match
```

---

### 2. List\<T\>

**Type:** Dynamic array, indexed, mutable, generic

**When to Use:**

- Unknown size (default choice for collections)
- Need indexed access
- Frequent additions/removals

**Namespace:** System.Collections.Generic

#### Creation

```csharp
// Empty list
List<int> list1 = new List<int>();

// With initial capacity
List<int> list2 = new List<int>(100);

// Collection initializer
List<int> list3 = new List<int> {1, 2, 3, 4, 5};

// From existing collection
List<int> list4 = new List<int>(array);

// Modern syntax (C# 12.0+)
List<int> list5 = [1, 2, 3, 4, 5];    // Collection expression
```

#### Access

```csharp
// Index access
int value = list[0];
list[0] = 10;

// Iteration
for (int i = 0; i < list.Count; i++)
{
    Console.WriteLine(list[i]);
}

foreach (int x in list)
{
    Console.WriteLine(x);
}
```

#### Add/Remove

```csharp
// Add
list.Add(5);                          // Add single item
list.AddRange(new[] {6, 7, 8});       // Add multiple items
list.Insert(0, 99);                   // Insert at index

// Remove
list.Remove(5);                       // Remove first occurrence
list.RemoveAt(0);                     // Remove at index
list.RemoveAll(x => x > 10);          // Remove all matching
list.RemoveRange(0, 3);               // Remove range
list.Clear();                         // Remove all
```

#### Key Properties & Methods

```csharp
// Properties
list.Count                    // Number of elements
list.Capacity                 // Allocated size
list.IsReadOnly               // Always false

// Search
list.Contains(5);             // Check if exists
list.IndexOf(5);              // Find first index
list.LastIndexOf(5);          // Find last index
list.Find(x => x > 10);       // Find first match
list.FindAll(x => x > 10);    // Find all matches
list.FindIndex(x => x > 10);  // Find index of first match
list.Exists(x => x > 10);     // Check if any match
list.TrueForAll(x => x > 0);  // Check if all match

// Modify
list.Sort();                  // Sort in place
list.Reverse();               // Reverse order

// Convert
list.ToArray();               // Convert to array
```

#### Performance Notes

- **Capacity doubling:** When full, capacity doubles (expensive)
- **Pre-allocate:** Use `new List<int>(expectedSize)` if size known
- **Insertion:** O(n) for Insert, O(1) for Add (at end)
- **Access:** O(1) by index

---

### 3. ArrayList (Legacy)

**Type:** Dynamic array, indexed, mutable, **non-generic** (stores objects)

**When to Use:** ❌ **Don't use in new code** - use `List<T> instead

**Why Avoid:**

- No type safety
- Boxing/unboxing overhead for value types
- Slower than generic collections

**Namespace:** System.Collections

#### Creation

```csharp
ArrayList arr = new ArrayList();
ArrayList arr2 = new ArrayList(10);           // Capacity
ArrayList arr3 = new ArrayList {1, "two", 3.0}; // Mixed types (bad!)
```

#### Access

```csharp
// Returns object - needs casting!
object obj = arr[0];
int value = (int)arr[0];                     // Must cast

foreach (object x in arr)
{
    // Process object
}
```

#### Add/Remove

```csharp
arr.Add(5);                                   // Boxing if value type!
arr.AddRange(new[] {1, 2, 3});
arr.Insert(0, 99);
arr.InsertRange(0, new[] {1, 2});

arr.Remove(5);
arr.RemoveAt(0);
arr.RemoveRange(0, 3);
arr.Clear();
```

#### Key Properties & Methods

```csharp
// Properties
arr.Count
arr.Capacity
arr.IsFixedSize               // false (dynamic)
arr.IsReadOnly                // false
arr.IsSynchronized            // false (not thread-safe)
arr.SyncRoot                  // Object for synchronization

// Methods (same as List\<T\> but with objects)
arr.Contains(5);              // Boxing!
arr.IndexOf(5);
arr.Sort();
arr.Reverse();
```

#### Migration Example

```csharp
// ❌ Old way (ArrayList)
ArrayList list = new ArrayList();
list.Add(5);                  // Boxing
int value = (int)list[0];     // Unboxing + casting

// ✅ New way (List\<T\>)
List<int> list = new List<int>();
list.Add(5);                  // No boxing
int value = list[0];          // No casting
```

---

## Part 3: Dictionaries (Key-Value Collections)

### 4. Dictionary<TKey, TValue>

**Type:** Key-value pairs, unordered, unique keys, fast lookups (hash table)

**When to Use:**

- Need fast lookup by key (O(1))
- Keys must be unique
- Order doesn't matter

**Namespace:** System.Collections.Generic

#### Creation

```csharp
// Empty dictionary
Dictionary<int, string> dict = new Dictionary<int, string>();

// Collection initializer (old syntax)
Dictionary<int, string> dict2 = new Dictionary<int, string>
{
    {1, "one"},
    {2, "two"},
    {3, "three"}
};

// Modern syntax (C# 6.0+)
Dictionary<int, string> dict3 = new Dictionary<int, string>
{
    [1] = "one",
    [2] = "two",
    [3] = "three"
};

// Collection expression (C# 12.0+) - Not supported yet for dictionaries
```

#### Access

```csharp
// By key (throws if not found)
string value = dict[1];

// Safe access
if (dict.TryGetValue(1, out string result))
{
    Console.WriteLine(result);
}

// Iterate
foreach (KeyValuePair<int, string> kvp in dict)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}

// Iterate keys only
foreach (int key in dict.Keys)
{
    Console.WriteLine(key);
}

// Iterate values only
foreach (string value in dict.Values)
{
    Console.WriteLine(value);
}
```

#### Add/Remove

```csharp
// Add (throws if key exists)
dict.Add(4, "four");

// Add or update (upsert)
dict[4] = "four";

// Safe add (C# 10.0+)
bool added = dict.TryAdd(4, "four");  // Returns false if exists

// Remove
dict.Remove(1);                       // Returns true if removed
dict.Clear();                         // Remove all
```

#### Key Properties & Methods

```csharp
// Properties
dict.Count                    // Number of key-value pairs
dict.Keys                     // Collection of keys
dict.Values                   // Collection of values

// Search
dict.ContainsKey(1);          // O(1) check for key
dict.ContainsValue("one");    // O(n) check for value (slow!)
dict.TryGetValue(1, out string value); // Safe retrieval
```

#### Performance

- **Lookup:** O(1) average, O(n) worst case
- **Insert:** O(1) average
- **Remove:** O(1) average
- **Best for:** Fast lookups by key

---

### 5. SortedDictionary<TKey, TValue>

**Type:** Key-value pairs, **sorted by key**, slower than Dictionary

**When to Use:**

- Need keys in sorted order
- Willing to sacrifice speed for ordering

**Namespace:** System.Collections.Generic

#### Creation

```csharp
SortedDictionary<int, string> sd = new SortedDictionary<int, string>();

SortedDictionary<int, string> sd2 = new SortedDictionary<int, string>
{
    {3, "three"},
    {1, "one"},     // Automatically sorted by key
    {2, "two"}
};
```

#### Access

```csharp
// Same as Dictionary
string value = sd[1];

// Iteration is in sorted order
foreach (KeyValuePair<int, string> kvp in sd)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}
// Output: 1: one, 2: two, 3: three (sorted!)
```

#### Add/Remove

```csharp
// Same as Dictionary
sd.Add(4, "four");
sd[4] = "four";
sd.Remove(1);
sd.Clear();
```

#### Key Properties & Methods

```csharp
// Properties
sd.Count
sd.Keys                       // Sorted keys collection
sd.Values                     // Values in key order

// Methods (same as Dictionary)
sd.ContainsKey(1);
sd.ContainsValue("one");
sd.TryGetValue(1, out string value);
```

#### Performance

- **Lookup:** O(log n)
- **Insert:** O(log n)
- **Remove:** O(log n)
- **Ordered iteration:** Yes (by key)

#### Dictionary vs SortedDictionary

|Feature|Dictionary|SortedDictionary|
|---|---|---|
|**Order**|Unordered|Sorted by key|
|**Lookup**|O(1)|O(log n)|
|**Insert**|O(1)|O(log n)|
|**Use when**|Speed critical|Order matters|

---

### 6. SortedList<TKey, TValue>

**Type:** Key-value pairs, **sorted by key**, **indexed access**

**When to Use:**

- Need both key-value lookup AND indexed access
- Small collections (< 1000 items)
- Memory efficient

**Namespace:** System.Collections.Generic

#### Creation

```csharp
SortedList<int, string> sl = new SortedList<int, string>();

SortedList<int, string> sl2 = new SortedList<int, string>
{
    {1, "one"},
    {2, "two"}
};
```

#### Access

```csharp
// By key
string value = sl[1];

// By index (unique feature!)
int key = sl.Keys[0];         // First key
string val = sl.Values[0];    // First value

// Iterate
foreach (KeyValuePair<int, string> kvp in sl)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}
```

#### Add/Remove

```csharp
// Add (throws if key exists)
sl.Add(3, "three");

// Remove
sl.Remove(1);                 // By key
sl.RemoveAt(0);               // By index (unique!)
sl.Clear();
```

#### Key Properties & Methods

```csharp
// Properties
sl.Count
sl.Keys                       // Sorted, indexed keys
sl.Values                     // Sorted, indexed values
sl.Capacity

// Methods
sl.ContainsKey(1);
sl.ContainsValue("one");
sl.TryGetValue(1, out string value);
sl.IndexOfKey(1);             // Get index of key
sl.IndexOfValue("one");       // Get index of value
```

#### SortedList vs SortedDictionary

|Feature|SortedList|SortedDictionary|
|---|---|---|
|**Memory**|Less|More|
|**Indexed access**|Yes|No|
|**Insert/Remove**|O(n)|O(log n)|
|**Lookup**|O(log n)|O(log n)|
|**Best for**|Small, memory-critical|Large, frequent updates|

---

### 7. Hashtable (Legacy)

**Type:** Key-value pairs, **non-generic**, stores objects, **legacy**

**When to Use:** ❌ **Don't use in new code** - use `Dictionary<TKey, TValue>` instead

**Namespace:** System.Collections

#### Creation

```csharp
Hashtable ht = new Hashtable();

// Can mix types (bad!)
Hashtable ht2 = new Hashtable
{
    {1, "one"},
    {"key", 123}
};
```

#### Access

```csharp
// Returns object - needs casting
object obj = ht[1];
string value = (string)ht[1];

// Iterate
foreach (DictionaryEntry de in ht)
{
    Console.WriteLine($"{de.Key}: {de.Value}");
}
```

#### Add/Remove

```csharp
ht.Add(1, "one");                     // Boxing!
ht[1] = "one";                        // Upsert
ht.Remove(1);
ht.Clear();
```

#### Key Properties & Methods

```csharp
// Properties
ht.Count
ht.Keys
ht.Values
ht.IsFixedSize                // false
ht.IsReadOnly                 // false
ht.IsSynchronized             // false
ht.SyncRoot                   // Object for synchronization

// Methods
ht.Contains(1);               // Check key (legacy)
ht.ContainsKey(1);            // Check key
ht.ContainsValue("one");      // Check value
```

---

## Part 4: Sets (Unique Elements)

### 8. HashSet\<T\>

**Type:** Unordered, unique elements, no indexing, fast lookups

**When to Use:**

- Need unique elements only
- Fast membership tests (O(1))
- Set operations (union, intersection, etc.)

**Namespace:** System.Collections.Generic

#### Creation

```csharp
HashSet<int> hs = new HashSet<int>();

HashSet<int> hs2 = new HashSet<int> {1, 2, 3, 2, 1}; // {1, 2, 3} only

HashSet<int> hs3 = new HashSet<int>(array);

// Collection expression (C# 12.0+)
HashSet<int> hs4 = [1, 2, 3];
```

#### Access

```csharp
// ❌ No index access (unordered)

// Check membership (O(1))
bool exists = hs.Contains(5);

// Iterate (no guaranteed order)
foreach (int x in hs)
{
    Console.WriteLine(x);
}
```

#### Add/Remove

```csharp
// Add (returns false if already exists)
bool added = hs.Add(5);               // true if added, false if existed

// Remove
bool removed = hs.Remove(5);          // true if removed, false if not found
hs.RemoveWhere(x => x > 10);          // Remove all matching
hs.Clear();
```

#### Set Operations

```csharp
HashSet<int> a = new HashSet<int> {1, 2, 3};
HashSet<int> b = new HashSet<int> {3, 4, 5};

// Union (a ∪ b)
a.UnionWith(b);                       // a = {1, 2, 3, 4, 5}

// Intersection (a ∩ b)
a.IntersectWith(b);                   // a = {3}

// Difference (a - b)
a.ExceptWith(b);                      // a = {1, 2}

// Symmetric difference (a ⊕ b)
a.SymmetricExceptWith(b);             // a = {1, 2, 4, 5}

// Subset check
bool isSubset = a.IsSubsetOf(b);
bool isSuperset = a.IsSupersetOf(b);
bool isProperSubset = a.IsProperSubsetOf(b);

// Overlap check
bool overlaps = a.Overlaps(b);
bool setEquals = a.SetEquals(b);
```

#### Key Properties & Methods

```csharp
// Properties
hs.Count

// Methods
hs.Contains(5);               // O(1) lookup
hs.Add(5);
hs.Remove(5);
hs.Clear();

// Set operations (all modify the set)
hs.UnionWith(other);
hs.IntersectWith(other);
hs.ExceptWith(other);
hs.SymmetricExceptWith(other);

// Comparison (non-modifying)
hs.IsSubsetOf(other);
hs.IsSupersetOf(other);
hs.Overlaps(other);
hs.SetEquals(other);
```

---

### 9. SortedSet\<T\>

**Type:** Sorted order, unique elements, no indexing

**When to Use:**

- Need unique elements in sorted order
- Need Min/Max quickly

**Namespace:** System.Collections.Generic

#### Creation

```csharp
SortedSet<int> ss = new SortedSet<int>();

SortedSet<int> ss2 = new SortedSet<int> {3, 1, 2}; // {1, 2, 3} sorted!
```

#### Access

```csharp
// ❌ No index access (but ordered)

// Get Min/Max (O(1))
int min = ss.Min;
int max = ss.Max;

// Iterate (in sorted order)
foreach (int x in ss)
{
    Console.WriteLine(x);                 // 1, 2, 3 (sorted)
}

// Reverse iteration
foreach (int x in ss.Reverse())
{
    Console.WriteLine(x);                 // 3, 2, 1
}
```

#### Add/Remove

```csharp
// Add (returns false if exists)
bool added = ss.Add(5);

// Remove
bool removed = ss.Remove(5);
ss.RemoveWhere(x => x > 10);
ss.Clear();
```

#### Set Operations

```csharp
// Same as HashSet
ss.UnionWith(other);
ss.IntersectWith(other);
ss.ExceptWith(other);
ss.SymmetricExceptWith(other);

// Comparison
ss.IsSubsetOf(other);
ss.IsSupersetOf(other);
ss.Overlaps(other);
ss.SetEquals(other);
```

#### Unique Features

```csharp
// Get subset in range
SortedSet<int> subset = ss.GetViewBetween(10, 20); // 10 <= x <= 20

// Reverse iteration
foreach (int x in ss.Reverse()) { }
```

#### HashSet vs SortedSet

|Feature|HashSet|SortedSet|
|---|---|---|
|**Order**|Unordered|Sorted|
|**Lookup**|O(1)|O(log n)|
|**Insert**|O(1)|O(log n)|
|**Min/Max**|O(n)|O(1)|
|**Use when**|Speed only|Need ordering|

---

## Part 5: Queues & Stacks

### 10. Stack\<T\>

**Type:** LIFO (Last In, First Out), no indexing

**When to Use:**

- Undo/redo functionality
- Recursive algorithms (call stack)
- Depth-first traversal
- Expression evaluation

**Namespace:** System.Collections.Generic

#### Creation

```csharp
Stack<int> stack = new Stack<int>();

Stack<int> stack2 = new Stack<int>(array);
```

#### Operations

```csharp
// Push (add to top)
stack.Push(1);
stack.Push(2);
stack.Push(3);
// Stack: [3, 2, 1] (3 is on top)

// Peek (view top without removing)
int top = stack.Peek();               // 3 (doesn't remove)

// Pop (remove and return top)
int value = stack.Pop();              // 3 (removes it)
// Stack: [2, 1]

// Iterate (top to bottom, doesn't remove)
foreach (int x in stack)
{
    Console.WriteLine(x);             // 2, then 1
}
```

#### Key Properties & Methods

```csharp
// Properties
stack.Count

// Methods
stack.Push(item);                     // Add to top
stack.Pop();                          // Remove from top (throws if empty)
stack.Peek();                         // View top (throws if empty)
stack.Contains(item);                 // Check if exists
stack.Clear();                        // Remove all
stack.ToArray();                      // Convert to array (top to bottom)
```

#### Safe Access

```csharp
// Check before Pop/Peek
if (stack.Count > 0)
{
    int value = stack.Pop();
}

// Or use TryPeek/TryPop (.NET 5.0+)
if (stack.TryPop(out int value))
{
    Console.WriteLine(value);
}
```

---

### 11. Queue\<T\>

**Type:** FIFO (First In, First Out), no indexing

**When to Use:**

- Process items in order received
- Breadth-first traversal
- Message queues
- Print spooling

**Namespace:** System.Collections.Generic

#### Creation

```csharp
Queue<int> queue = new Queue<int>();

Queue<int> queue2 = new Queue<int>(array);
```

#### Operations

```csharp
// Enqueue (add to back)
queue.Enqueue(1);
queue.Enqueue(2);
queue.Enqueue(3);
// Queue: [1, 2, 3] (1 is front, 3 is back)

// Peek (view front without removing)
int front = queue.Peek();             // 1 (doesn't remove)

// Dequeue (remove and return front)
int value = queue.Dequeue();          // 1 (removes it)
// Queue: [2, 3]

// Iterate (front to back, doesn't remove)
foreach (int x in queue)
{
    Console.WriteLine(x);             // 2, then 3
}
```

#### Key Properties & Methods

```csharp
// Properties
queue.Count

// Methods
queue.Enqueue(item);                  // Add to back
queue.Dequeue();                      // Remove from front (throws if empty)
queue.Peek();                         // View front (throws if empty)
queue.Contains(item);                 // Check if exists
queue.Clear();                        // Remove all
queue.ToArray();                      // Convert to array (front to back)
```

#### Safe Access

```csharp
// Check before Dequeue/Peek
if (queue.Count > 0)
{
    int value = queue.Dequeue();
}

// Or use TryDequeue/TryPeek (.NET 5.0+)
if (queue.TryDequeue(out int value))
{
    Console.WriteLine(value);
}
```

---

## Part 6: Specialized Collections

### 12. LinkedList\<T\>

**Type:** Doubly-linked list, no indexing, efficient insertion/removal

**When to Use:**

- Frequent insertions/removals in middle
- Don't need indexed access
- Implement custom data structures (deque, etc.)

**Namespace:** System.Collections.Generic

#### Creation

```csharp
LinkedList<int> ll = new LinkedList<int>();

LinkedList<int> ll2 = new LinkedList<int>(array);
```

#### Node-Based Operations

```csharp
// Add at ends
LinkedListNode<int> node1 = ll.AddFirst(1);   // Add to front
LinkedListNode<int> node2 = ll.AddLast(3);    // Add to back

// Add relative to node
ll.AddAfter(node1, 2);                        // Add after node1
ll.AddBefore(node2, 2);                       // Add before node2

// Remove
ll.RemoveFirst();                             // Remove from front
ll.RemoveLast();                              // Remove from back
ll.Remove(node1);                             // Remove specific node
ll.Remove(5);                                 // Remove first occurrence of value
ll.Clear();
```

#### Navigation

```csharp
// Get first/last nodes
LinkedListNode<int> first = ll.First;
LinkedListNode<int> last = ll.Last;

// Navigate nodes
LinkedListNode<int> current = ll.First;
while (current != null)
{
    Console.WriteLine(current.Value);
    current = current.Next;               // Move to next
}

// Backwards navigation
current = ll.Last;
while (current != null)
{
    Console.WriteLine(current.Value);
    current = current.Previous;           // Move to previous
}

// Iterate
foreach (int x in ll)
{
    Console.WriteLine(x);
}
```

#### Key Properties & Methods

```csharp
// Properties
ll.Count
ll.First                      // First node (or null)
ll.Last                       // Last node (or null)

// Node properties
node.Value                    // The value
node.Next                     // Next node (or null)
node.Previous                 // Previous node (or null)
node.List                     // Parent LinkedList

// Methods
ll.AddFirst(value);           // Returns LinkedListNode\<T\>
ll.AddLast(value);
ll.AddAfter(node, value);
ll.AddBefore(node, value);
ll.RemoveFirst();
ll.RemoveLast();
ll.Remove(node);
ll.Remove(value);
ll.Find(value);               // Find first node with value
ll.FindLast(value);           // Find last node with value
ll.Contains(value);
ll.Clear();
```

#### Performance

- **Access by index:** ❌ Not supported (O(n) traversal)
- **Insert/Remove at known position:** O(1)
- **Insert/Remove at unknown position:** O(n) to find + O(1) to modify
- **Best for:** Frequent mid-list insertions/removals

---

### 13. BitArray

**Type:** Array of boolean values stored as bits, mutable

**When to Use:**

- Store boolean flags efficiently (8x memory saving)
- Bitwise operations on large boolean sets
- Bit manipulation

**Namespace:** System.Collections

#### Creation

```csharp
// Create with size (all false)
BitArray ba1 = new BitArray(8);              // [F, F, F, F, F, F, F, F]

// Create with size and default value
BitArray ba2 = new BitArray(8, true);        // [T, T, T, T, T, T, T, T]

// From boolean array
BitArray ba3 = new BitArray(new bool[] {true, false, true});

// From byte array
BitArray ba4 = new BitArray(new byte[] {0xFF, 0x00});
```

#### Access

```csharp
// Index access
bool value = ba[0];
ba[0] = true;

// Iteration
for (int i = 0; i < ba.Length; i++)
{
    Console.WriteLine(ba[i]);
}

foreach (bool bit in ba)
{
    Console.WriteLine(bit);
}
```

#### Modify

```csharp
// ❌ Cannot add/remove (fixed size)

// Set individual bits
ba.Set(0, true);              // Set bit 0 to true
ba.Set(1, false);             // Set bit 1 to false

// Set all bits
ba.SetAll(true);              // Set all to true
ba.SetAll(false);             // Set all to false
```

#### Bitwise Operations

```csharp
BitArray a = new BitArray(new bool[] {true, false, true});
BitArray b = new BitArray(new bool[] {true, true, false});

// AND
a.And(b);                     // a = {true, false, false}

// OR
a.Or(b);                      // a = {true, true, true}

// XOR
a.Xor(b);                     // a = {false, true, true}

// NOT
a.Not();                      // a = {true, false, false}
```

#### Key Properties & Methods

```csharp
// Properties
ba.Length                     // Number of bits
ba.Count                      // Same as Length

// Methods
ba.Get(index);                // Get bit value
ba.Set(index, value);         // Set bit value
ba.SetAll(value);             // Set all bits

// Bitwise operations (modify in place)
ba.And(other);
ba.Or(other);
ba.Xor(other);
ba.Not();
```

---

## Part 7: Concurrent Collections (Thread-Safe)

**Namespace:** System.Collections.Concurrent

**When to Use:** Multi-threaded scenarios

### Thread-Safe Collections

```csharp
using System.Collections.Concurrent;

// Thread-safe dictionary
ConcurrentDictionary<int, string> dict = new();
dict.TryAdd(1, "one");
dict.TryGetValue(1, out string value);
dict.AddOrUpdate(1, "one", (key, old) => "ONE");

// Thread-safe queue (FIFO)
ConcurrentQueue<int> queue = new();
queue.Enqueue(1);
queue.TryDequeue(out int value);

// Thread-safe stack (LIFO)
ConcurrentStack<int> stack = new();
stack.Push(1);
stack.TryPop(out int value);

// Thread-safe bag (unordered)
ConcurrentBag<int> bag = new();
bag.Add(1);
bag.TryTake(out int value);

// Blocking collection (producer-consumer)
BlockingCollection<int> bc = new();
bc.Add(1);                                // Producer
int item = bc.Take();                     // Consumer (blocks if empty)
```

---

## Part 8: Collection Comparison Tables

### Quick Comparison Table

|Collection|Ordered|Unique|Indexed|Add/Remove|Time Complexity|Use Case|
|---|---|---|---|---|---|---|
|**Array**|Yes|No|Yes|❌ Fixed|O(1) access|Fixed-size data|
|**List\<T\>**|Yes|No|Yes|Add, Remove|O(1) access, O(n) insert|Dynamic arrays|
|**ArrayList**|Yes|No|Yes|Add, Remove|Same as List|❌ Legacy (avoid)|
|**Dictionary**|No|Keys|By key|Add, Remove|O(1) lookup|Fast key-value|
|**SortedDictionary**|Sorted|Keys|By key|Add, Remove|O(log n)|Sorted key-value|
|**SortedList**|Sorted|Keys|Yes|Add, Remove|O(log n) lookup|Small sorted + indexed|
|**Hashtable**|No|Keys|By key|Add, Remove|O(1) lookup|❌ Legacy (avoid)|
|**HashSet\<T\>**|No|Yes|No|Add, Remove|O(1) lookup|Unique items|
|**SortedSet\<T\>**|Sorted|Yes|No|Add, Remove|O(log n)|Unique + sorted|
|**Stack\<T\>**|LIFO|No|No|Push, Pop|O(1)|LIFO operations|
|**Queue\<T\>**|FIFO|No|No|Enqueue, Dequeue|O(1)|FIFO operations|
|**LinkedList\<T\>**|Yes|No|No|AddFirst/Last|O(1) at known node|Frequent mid-insert|
|**BitArray**|Yes|No|Yes|❌ Fixed|O(1) access|Boolean flags|

### Performance Comparison

|Operation|Array|List\<T\>|Dictionary|HashSet|LinkedList|
|---|---|---|---|---|---|
|**Access by index**|O(1)|O(1)|N/A|N/A|O(n)|
|**Access by key**|N/A|N/A|O(1)|N/A|N/A|
|**Insert at end**|❌|O(1)*|O(1)|O(1)|O(1)|
|**Insert at start**|❌|O(n)|N/A|N/A|O(1)|
|**Insert in middle**|❌|O(n)|N/A|N/A|O(1)**|
|**Remove**|❌|O(n)|O(1)|O(1)|O(1)**|
|**Search**|O(n)|O(n)|O(1)|O(1)|O(n)|
|**Memory**|Low|Low|Medium|Medium|High|

* Amortized (may resize)  
** If node is already known

---

## Part 9: Special Properties Explained

### Which Collections Have Special Properties?

**Legacy/Non-Generic Collections ONLY:**

- **Array** - Has `Rank`, `IsFixedSize`, `IsReadOnly`, `IsSynchronized`, `SyncRoot`
- **ArrayList** - Has `IsFixedSize`, `IsReadOnly`, `IsSynchronized`, `SyncRoot`
- **Hashtable** - Has `IsFixedSize`, `IsReadOnly`, `IsSynchronized`, `SyncRoot`
- **BitArray** - Has `IsSynchronized`, `SyncRoot`
- **Queue** (non-generic) - Has `IsSynchronized`, `SyncRoot`
- **Stack** (non-generic) - Has `IsSynchronized`, `SyncRoot`

**Modern/Generic Collections DO NOT Have These:**

- `List<T>`, `Dictionary<TKey,TValue>`, `HashSet<T>`, `SortedSet<T>`, `Queue<T>`, `Stack<T>`, `LinkedList<T>`
- **None of these have special properties**

### Why the Difference?

**Legacy Collections (pre-.NET 2.0):**

- Built when multithreading was handled differently
- Implement `ICollection` interface (non-generic) which requires:
    - `IsSynchronized` - Is collection thread-safe?
    - `SyncRoot` - Object to lock on for thread safety
    - `IsFixedSize` - Can size change?
    - `IsReadOnly` - Can items be modified?

**Modern Generic Collections (.NET 2.0+):**

- Built with better design principles
- Thread safety handled externally (use `Concurrent*` collections)
- Don't clutter API with rarely-used properties
- Simpler, cleaner interfaces

### Property Meanings

**`Rank` (Array only)**

```csharp
int[] arr1 = new int[5];          // Rank = 1 (single-dimensional)
int[,] arr2 = new int[3, 3];      // Rank = 2 (2D array)
int[][] arr3 = new int[3][];      // Rank = 1 (jagged array)

arr2.GetLength(0);                // 3 (rows)
arr2.GetLength(1);                // 3 (columns)
```

**`IsFixedSize`**

- `true` = Cannot add/remove elements (Array)
- `false` = Can grow/shrink (ArrayList, Hashtable)

**`IsReadOnly`**

- `true` = Cannot modify elements
- Usually `false` for standard collections

**`IsSynchronized`**

- `true` = Thread-safe (rarely true by default)
- `false` = NOT thread-safe (most collections)
- ❌ Legacy property - use `Concurrent*` collections instead

**`SyncRoot`**

- Object to use with `lock()` for thread safety
- ❌ Legacy pattern - use `Concurrent*` collections instead

```csharp
// ❌ Legacy pattern (avoid)
lock (myArrayList.SyncRoot)
{
    // Thread-safe operations
}

// ✅ Modern approach
ConcurrentDictionary<int, string> dict = new();
```

### Quick Rule

- **Using generics (`<T>)? No special properties.**
- **Using old non-generic? Has special properties.**
- **Need thread safety? Use `System.Collections.Concurrent`**

---

## Part 10: Best Practices & Guidelines

### Choosing the Right Collection

**Decision Tree:**

```
Need key-value pairs?
├─ Yes
│  ├─ Need ordering?
│  │  ├─ Yes → SortedDictionary<K,V> or SortedList<K,V>
│  │  └─ No → Dictionary<K,V> ✅ (default choice)
│  └─ Need indexed access? → SortedList<K,V>
│
└─ No
   ├─ Need unique elements only?
   │  ├─ Yes
   │  │  ├─ Need ordering? → SortedSet\<T\>
   │  │  └─ No → HashSet\<T\> ✅
   │  │
   │  └─ No (allow duplicates)
   │     ├─ Need indexed access? → List\<T\> ✅ (default choice)
   │     ├─ LIFO (stack)? → Stack\<T\>
   │     ├─ FIFO (queue)? → Queue\<T\>
   │     ├─ Frequent mid-insertions? → LinkedList\<T\>
   │     ├─ Boolean flags? → BitArray
   │     └─ Fixed size? → Array
```

### General Guidelines

✅ **Do:**

- Use generic collections (`List<T>, not `ArrayList`)
- Use `Dictionary<K,V>` for fast lookups
- Use `HashSet<T> for unique elements
- Pre-allocate size if known: `new List<int>(1000)`
- Use `Concurrent*` collections for thread safety
- Use collection expressions (C# 12.0+): `List<int> list = [1, 2, 3];`

❌ **Don't:**

- Use non-generic collections (ArrayList, Hashtable) in new code
- Use `Add()` in loops without pre-allocating capacity
- Box value types (use generics)
- Use `lock(collection)` for thread safety (use `Concurrent*`)
- Access List/Array by index in tight loops (use foreach if possible)

### Performance Tips

**List\<T\> Capacity:**

```csharp
// ❌ Bad (resizes multiple times)
List<int> list = new List<int>();
for (int i = 0; i < 1000; i++)
    list.Add(i);

// ✅ Good (no resizing)
List<int> list = new List<int>(1000);
for (int i = 0; i < 1000; i++)
    list.Add(i);
```

**Dictionary vs List for Lookups:**

```csharp
// ❌ Slow (O(n) search)
List<int> list = new List<int>();
bool exists = list.Contains(5000);

// ✅ Fast (O(1) search)
HashSet<int> set = new HashSet<int>();
bool exists = set.Contains(5000);
```

**Avoid Boxing:**

```csharp
// ❌ Boxing
ArrayList list = new ArrayList();
list.Add(5);                      // Boxing!

// ✅ No boxing
List<int> list = new List<int>();
list.Add(5);                      // No boxing
```

---

## Quick Reference Summary

### Most Common Collections

**For general use:**

- `List<T> - Dynamic array (default choice)
- `Dictionary<K,V>` - Fast key-value lookup
- `HashSet<T> - Unique elements

**For specialized needs:**

- `SortedDictionary<K,V>` - Sorted key-value
- `SortedSet<T> - Sorted unique elements
- `Queue<T>` - FIFO operations
- `Stack**<T>** - LIFO operations
- `LinkedList<T> - Frequent insertions/removals

**For thread safety:**

- `ConcurrentDictionary<K,V>`
- `ConcurrentQueue<T>
- `ConcurrentStack<T>
- `ConcurrentBag<T>
- `BlockingCollection<T>

### Key Terminology

- **Immutable Length** - Size cannot change (Array)
- **Mutable** - Can add/remove elements
- **Generic** - Type-safe (`List<T>, `Dictionary<K,V>`)
- **Non-Generic** - Stores objects, requires casting (ArrayList, Hashtable) - ❌ Avoid
- **LIFO** - Last In, First Out (Stack)
- **FIFO** - First In, First Out (Queue)
- **Ordered** - Maintains insertion order
- **Sorted** - Automatically sorted
- **Indexed** - Can access by numeric index
- **O(1)** - Constant time (very fast)
- **O(log n)** - Logarithmic time (fast)
- **O(n)** - Linear time (slower with more items)

---

**Guide Complete!** You now have a complete collections reference! 📚