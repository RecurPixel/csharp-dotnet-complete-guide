# C# Control Flow & Program Structure Quick Reference

---

## 1. Decision Making Statements

### if Statement

```csharp
int age = 18;

if (age >= 18)
{
    Console.WriteLine("Adult");
}
```

**Syntax:**
```csharp
if (condition)
{
    // Code executes if condition is true
}
```

### if-else Statement

```csharp
int age = 15;

if (age >= 18)
{
    Console.WriteLine("Adult");
}
else
{
    Console.WriteLine("Minor");
}
```

### if-else-if Ladder

```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("A");
}
else if (score >= 80)
{
    Console.WriteLine("B");
}
else if (score >= 70)
{
    Console.WriteLine("C");
}
else if (score >= 60)
{
    Console.WriteLine("D");
}
else
{
    Console.WriteLine("F");
}
```

### Nested if Statements

```csharp
int age = 25;
bool hasLicense = true;

if (age >= 18)
{
    if (hasLicense)
    {
        Console.WriteLine("Can drive");
    }
    else
    {
        Console.WriteLine("Need license");
    }
}
else
{
    Console.WriteLine("Too young to drive");
}
```

### Ternary Operator (?:)

```csharp
// Syntax: condition ? valueIfTrue : valueIfFalse

int age = 20;
string status = age >= 18 ? "Adult" : "Minor";

// Equivalent to:
string status;
if (age >= 18)
    status = "Adult";
else
    status = "Minor";

// Nested ternary (avoid if complex)
int num = 10;
string result = num > 0 ? "Positive" : num < 0 ? "Negative" : "Zero";
```

### switch Statement (Traditional) - C# 1.0

```csharp
int dayOfWeek = 3;

switch (dayOfWeek)
{
    case 1:
        Console.WriteLine("Monday");
        break;
    case 2:
        Console.WriteLine("Tuesday");
        break;
    case 3:
        Console.WriteLine("Wednesday");
        break;
    case 4:
        Console.WriteLine("Thursday");
        break;
    case 5:
        Console.WriteLine("Friday");
        break;
    case 6:
    case 7:
        Console.WriteLine("Weekend");
        break;
    default:
        Console.WriteLine("Invalid day");
        break;
}
```

**Key Points:**

- Each case must end with `break`, `return`, or `goto`
- C# does NOT have fall-through (unlike C/C++)
- Multiple cases can share same code block
- `default` is optional but recommended

**No Fall-Through in C#:**
```csharp
switch (value)
{
    case 1:
        Console.WriteLine("One");
        // break required - no automatic fall-through
        break;
    case 2:
        Console.WriteLine("Two");
        break;
}
```

### switch Expressions - C# 8.0+

**Much more concise and returns a value**

```csharp
// Traditional switch
string GetGrade(int score)
{
    switch (score)
    {
        case >= 90:
            return "A";
        case >= 80:
            return "B";
        case >= 70:
            return "C";
        default:
            return "F";
    }
}

// Switch expression (C# 8.0+)
string GetGrade(int score) => score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    _     => "F"  // _ is discard pattern (default)
};
```

**Syntax:**
```csharp
result = expression switch
{
    pattern1 => value1,
    pattern2 => value2,
    _        => defaultValue
};
```

**More Examples:**
```csharp
// Day of week
string GetDayType(DayOfWeek day) => day switch
{
    DayOfWeek.Saturday or DayOfWeek.Sunday => "Weekend",
    _ => "Weekday"
};

// Multiple conditions
int value = 5;
string result = value switch
{
    < 0 => "Negative",
    0 => "Zero",
    > 0 and < 10 => "Single digit",
    >= 10 => "Multiple digits"
};
```

### Pattern Matching in switch

#### Type Patterns (C# 7.0+)

```csharp
object obj = "Hello";

switch (obj)
{
    case string s:
        Console.WriteLine($"String: {s}");
        break;
    case int i:
        Console.WriteLine($"Integer: {i}");
        break;
    case null:
        Console.WriteLine("null");
        break;
    default:
        Console.WriteLine("Unknown type");
        break;
}

// Or with switch expression
string result = obj switch
{
    string s => $"String: {s}",
    int i => $"Integer: {i}",
    null => "null",
    _ => "Unknown"
};
```

#### Property Patterns (C# 8.0+)

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

Person person = new Person { Name = "John", Age = 25 };

string category = person switch
{
    { Age: < 13 } => "Child",
    { Age: >= 13, Age: < 20 } => "Teenager",
    { Age: >= 20, Age: < 60 } => "Adult",
    { Age: >= 60 } => "Senior",
    _ => "Unknown"
};

// More complex
string description = person switch
{
    { Name: "John", Age: > 18 } => "Adult John",
    { Name: "John" } => "Young John",
    { Age: < 18 } => "Minor",
    _ => "Unknown person"
};
```

#### Tuple Patterns (C# 8.0+)

```csharp
string GetQuadrant(int x, int y) => (x, y) switch
{
    (0, 0) => "Origin",
    (> 0, > 0) => "Quadrant I",
    (< 0, > 0) => "Quadrant II",
    (< 0, < 0) => "Quadrant III",
    (> 0, < 0) => "Quadrant IV",
    (_, 0) => "On X-axis",
    (0, _) => "On Y-axis"
};

// Rock, Paper, Scissors
string GetWinner(string player1, string player2) => (player1, player2) switch
{
    ("rock", "scissors") => "Player 1 wins",
    ("scissors", "paper") => "Player 1 wins",
    ("paper", "rock") => "Player 1 wins",
    ("scissors", "rock") => "Player 2 wins",
    ("paper", "scissors") => "Player 2 wins",
    ("rock", "paper") => "Player 2 wins",
    _ => "Draw"
};
```

#### Positional Patterns (C# 8.0+)

```csharp
public record Point(int X, int Y);

Point point = new Point(10, 20);

string location = point switch
{
    (0, 0) => "Origin",
    (var x, 0) => $"On X-axis at {x}",
    (0, var y) => $"On Y-axis at {y}",
    (var x, var y) => $"Point at ({x}, {y})"
};
```

#### Relational Patterns (C# 9.0+)

```csharp
string GetWaterState(int temperature) => temperature switch
{
    < 0 => "Ice",
    >= 0 and < 100 => "Water",
    >= 100 => "Steam"
};

// Multiple ranges
string GetAgeGroup(int age) => age switch
{
    < 0 => "Invalid",
    < 13 => "Child",
    >= 13 and < 20 => "Teenager",
    >= 20 and < 60 => "Adult",
    >= 60 => "Senior"
};
```

#### Logical Patterns (C# 9.0+)

```csharp
// and, or, not
bool IsLetter(char c) => c is (>= 'a' and <= 'z') or (>= 'A' and <= 'Z');

bool IsWeekend(DayOfWeek day) => day is DayOfWeek.Saturday or DayOfWeek.Sunday;

bool IsNotNull(object obj) => obj is not null;

// Complex
string Classify(int number) => number switch
{
    < 0 => "Negative",
    0 => "Zero",
    > 0 and < 10 => "Single digit",
    >= 10 and < 100 => "Double digit",
    >= 100 => "Triple digit or more"
};
```

#### List Patterns (C# 11.0+)

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

string result = numbers switch
{
    [] => "Empty",
    [1] => "Single element: 1",
    [1, 2] => "Two elements: 1, 2",
    [1, 2, 3] => "Three elements: 1, 2, 3",
    [1, ..] => "Starts with 1",
    [.., 5] => "Ends with 5",
    [var first, .., var last] => $"First: {first}, Last: {last}",
    _ => "Other"
};

// Pattern with length
string Describe(int[] arr) => arr switch
{
    { Length: 0 } => "Empty array",
    { Length: 1 } => "Single element",
    { Length: > 10 } => "Large array",
    _ => "Normal array"
};
```

### Pattern Matching Evolution Table

| Version | Feature | Example |
|---------|---------|---------|
| C# 7.0 | Type patterns | `case string s:` |
| C# 7.0 | var pattern | `case var x:` |
| C# 8.0 | Property patterns | `{ Age: > 18 }` |
| C# 8.0 | Tuple patterns | `(1, 2) => ...` |
| C# 8.0 | Positional patterns | `Point(0, 0) => ...` |
| C# 8.0 | Switch expressions | `value switch { ... }` |
| C# 9.0 | Relational patterns | `>= 0 and < 100` |
| C# 9.0 | Logical patterns | `and`, `or`, `not` |
| C# 11.0 | List patterns | `[1, 2, ..]` |

---

## 2. Loops

### for Loop

**Syntax:**
```csharp
for (initialization; condition; increment)
{
    // Code to repeat
}
```

**Basic Example:**
```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);  // 0, 1, 2, 3, 4
}
```

**Multiple Variables:**
```csharp
for (int i = 0, j = 10; i < 10; i++, j--)
{
    Console.WriteLine($"i={i}, j={j}");
}
```

**Infinite Loop:**
```csharp
for (;;)  // or for (; true; )
{
    Console.WriteLine("Forever");
    // Use break to exit
}
```

**Variations:**
```csharp
// Count down
for (int i = 10; i >= 0; i--)
{
    Console.WriteLine(i);
}

// Step by 2
for (int i = 0; i < 10; i += 2)
{
    Console.WriteLine(i);  // 0, 2, 4, 6, 8
}

// Empty body
int sum = 0;
for (int i = 1; i <= 10; sum += i, i++) ;
Console.WriteLine(sum);  // 55
```

### while Loop

**Syntax:**
```csharp
while (condition)
{
    // Code to repeat
}
```

**Example:**
```csharp
int i = 0;
while (i < 5)
{
    Console.WriteLine(i);
    i++;
}

// Read until sentinel value
string input;
while ((input = Console.ReadLine()) != "quit")
{
    Console.WriteLine($"You entered: {input}");
}
```

**Infinite Loop:**
```csharp
while (true)
{
    Console.WriteLine("Forever");
    // Use break to exit
}
```

### do-while Loop

**Syntax:**
```csharp
do
{
    // Code to repeat
} while (condition);
```

**Key Difference:** Executes at least once (condition checked after)

```csharp
int i = 0;
do
{
    Console.WriteLine(i);
    i++;
} while (i < 5);

// Menu example (executes at least once)
string choice;
do
{
    Console.WriteLine("1. Option 1");
    Console.WriteLine("2. Option 2");
    Console.WriteLine("3. Exit");
    choice = Console.ReadLine();
    
    // Process choice...
    
} while (choice != "3");
```

### foreach Loop

**Syntax:**
```csharp
foreach (type variable in collection)
{
    // Code using variable
}
```

**With Arrays:**
```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

foreach (int num in numbers)
{
    Console.WriteLine(num);
}
```

**With Collections:**
```csharp
List<string> names = new List<string> { "John", "Jane", "Bob" };

foreach (string name in names)
{
    Console.WriteLine(name);
}

// Dictionary
Dictionary<string, int> ages = new Dictionary<string, int>
{
    ["John"] = 25,
    ["Jane"] = 30
};

foreach (KeyValuePair<string, int> pair in ages)
{
    Console.WriteLine($"{pair.Key}: {pair.Value}");
}

// Or with deconstruction
foreach (var (name, age) in ages)
{
    Console.WriteLine($"{name}: {age}");
}
```

**await foreach (C# 8.0+)**

```csharp
// For IAsyncEnumerable<T>
await foreach (var item in GetItemsAsync())
{
    Console.WriteLine(item);
}

async IAsyncEnumerable<int> GetItemsAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);
        yield return i;
    }
}
```

### Nested Loops

```csharp
// Multiplication table
for (int i = 1; i <= 10; i++)
{
    for (int j = 1; j <= 10; j++)
    {
        Console.Write($"{i * j,4}");
    }
    Console.WriteLine();
}

// 2D array traversal
int[,] matrix = { { 1, 2, 3 }, { 4, 5, 6 } };

for (int i = 0; i < matrix.GetLength(0); i++)
{
    for (int j = 0; j < matrix.GetLength(1); j++)
    {
        Console.Write(matrix[i, j] + " ");
    }
    Console.WriteLine();
}
```

**Loop Comparison:**

| Loop Type | When to Use |
|-----------|-------------|
| `for` | Know iteration count, need index |
| `while` | Don't know iteration count, check before |
| `do-while` | Must execute at least once |
| `foreach` | Iterate over collection, don't need index |

---

## 3. Jump Statements

### break

**Exits loop or switch immediately**

```csharp
// Exit loop
for (int i = 0; i < 10; i++)
{
    if (i == 5)
        break;  // Exit loop when i is 5
    Console.WriteLine(i);  // 0, 1, 2, 3, 4
}

// Exit switch
switch (value)
{
    case 1:
        Console.WriteLine("One");
        break;  // Required in switch
    case 2:
        Console.WriteLine("Two");
        break;
}

// Break outer loop (use flag or goto)
bool found = false;
for (int i = 0; i < 10 && !found; i++)
{
    for (int j = 0; j < 10; j++)
    {
        if (i * j == 20)
        {
            found = true;
            break;  // Only breaks inner loop
        }
    }
}
```

### continue

**Skip rest of current iteration, continue with next**

```csharp
// Skip even numbers
for (int i = 0; i < 10; i++)
{
    if (i % 2 == 0)
        continue;  // Skip even, go to next iteration
    Console.WriteLine(i);  // 1, 3, 5, 7, 9
}

// Skip empty strings
foreach (string s in strings)
{
    if (string.IsNullOrWhiteSpace(s))
        continue;
    
    Console.WriteLine(s.ToUpper());
}
```

### return

**Exit method and optionally return value**

```csharp
int FindIndex(int[] array, int target)
{
    for (int i = 0; i < array.Length; i++)
    {
        if (array[i] == target)
            return i;  // Exit method immediately
    }
    return -1;  // Not found
}

// void method
void ProcessData(string data)
{
    if (string.IsNullOrEmpty(data))
        return;  // Exit early
    
    // Process data...
}

// Multiple returns
string GetGrade(int score)
{
    if (score >= 90) return "A";
    if (score >= 80) return "B";
    if (score >= 70) return "C";
    return "F";
}
```

### goto (Discouraged)

**Jump to labeled statement**

```csharp
// ⚠️ Avoid goto - makes code hard to follow
for (int i = 0; i < 10; i++)
{
    for (int j = 0; j < 10; j++)
    {
        if (i * j == 20)
            goto Found;
    }
}
Console.WriteLine("Not found");
return;

Found:
Console.WriteLine("Found!");

// Rare acceptable use: break out of nested loops
// Better: use method with return or flag
```

### throw

**Throw exception (exits to exception handler)**

```csharp
void Withdraw(decimal amount)
{
    if (amount < 0)
        throw new ArgumentException("Amount cannot be negative");
    
    if (amount > Balance)
        throw new InvalidOperationException("Insufficient funds");
    
    Balance -= amount;
}
```

---

## 4. Arrays Deep Dive

### Single-Dimensional Arrays

**Declaration and Initialization:**
```csharp
// Declaration (not initialized)
int[] numbers;

// Declaration with size (initialized to default values)
int[] numbers = new int[5];  // [0, 0, 0, 0, 0]

// Declaration with initializer
int[] numbers = new int[] { 1, 2, 3, 4, 5 };

// Shorthand (type inference)
int[] numbers = { 1, 2, 3, 4, 5 };

// Empty array
int[] empty = new int[0];
// or
int[] empty = { };
// or (C# 12.0+)
int[] empty = [];
```

**Accessing Elements:**
```csharp
int[] numbers = { 10, 20, 30, 40, 50 };

// Get element
int first = numbers[0];      // 10
int last = numbers[4];       // 50

// Set element
numbers[2] = 35;

// Index from end (C# 8.0+)
int last = numbers[^1];      // 50 (last element)
int secondLast = numbers[^2]; // 40

// Out of bounds
int invalid = numbers[10];    // ❌ IndexOutOfRangeException
```

**Length Property:**
```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

int length = numbers.Length;  // 5

// Iterate using Length
for (int i = 0; i < numbers.Length; i++)
{
    Console.WriteLine(numbers[i]);
}
```

### Multi-Dimensional Arrays (Rectangular)

```csharp
// Declaration
int[,] matrix = new int[3, 4];  // 3 rows, 4 columns

// Initialization
int[,] matrix = new int[,]
{
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 }
};

// Shorthand
int[,] matrix = {
    { 1, 2, 3 },
    { 4, 5, 6 }
};

// Access element
int value = matrix[0, 1];  // 2 (row 0, column 1)
matrix[1, 2] = 99;

// GetLength() - get size of dimension
int rows = matrix.GetLength(0);     // 2
int cols = matrix.GetLength(1);     // 3
int total = matrix.Length;          // 6 (total elements)

// Iterate
for (int i = 0; i < matrix.GetLength(0); i++)
{
    for (int j = 0; j < matrix.GetLength(1); j++)
    {
        Console.Write(matrix[i, j] + " ");
    }
    Console.WriteLine();
}
```

**3D Arrays:**
```csharp
int[,,] cube = new int[2, 3, 4];  // 2x3x4 cube

cube[0, 1, 2] = 5;

int depth = cube.GetLength(0);   // 2
int rows = cube.GetLength(1);    // 3
int cols = cube.GetLength(2);    // 4
```

### Jagged Arrays (Array of Arrays)

```csharp
// Declaration
int[][] jagged = new int[3][];

// Initialize each sub-array separately
jagged[0] = new int[] { 1, 2 };
jagged[1] = new int[] { 3, 4, 5 };
jagged[2] = new int[] { 6, 7, 8, 9 };

// Or all at once
int[][] jagged = new int[][]
{
    new int[] { 1, 2 },
    new int[] { 3, 4, 5 },
    new int[] { 6, 7, 8, 9 }
};

// Access element
int value = jagged[1][2];  // 5

// Iterate
for (int i = 0; i < jagged.Length; i++)
{
    for (int j = 0; j < jagged[i].Length; j++)
    {
        Console.Write(jagged[i][j] + " ");
    }
    Console.WriteLine();
}

// Or with foreach
foreach (int[] row in jagged)
{
    foreach (int value in row)
    {
        Console.Write(value + " ");
    }
    Console.WriteLine();
}
```

**Rectangular vs Jagged:**

| Feature | Rectangular `[,]` | Jagged `[][]` |
|---------|------------------|---------------|
| Memory | Single contiguous block | Multiple arrays |
| Size | All rows same length | Rows can vary |
| Access | `arr[i, j]` | `arr[i][j]` |
| Performance | Faster access | Slower access |
| Flexibility | Fixed structure | Variable row lengths |
| Use when | Matrix, grid | Irregular data |

### Array Class Methods

```csharp
int[] numbers = { 5, 2, 8, 1, 9, 3 };

// Sort() - sort in place
Array.Sort(numbers);  // [1, 2, 3, 5, 8, 9]

// Reverse() - reverse in place
Array.Reverse(numbers);  // [9, 8, 5, 3, 2, 1]

// IndexOf() - find index of element
int index = Array.IndexOf(numbers, 5);  // 2

// LastIndexOf() - find last occurrence
int lastIndex = Array.LastIndexOf(numbers, 2);

// Clear() - set elements to default
Array.Clear(numbers, 0, numbers.Length);  // [0, 0, 0, 0, 0, 0]

// Copy() - copy elements
int[] source = { 1, 2, 3, 4, 5 };
int[] destination = new int[5];
Array.Copy(source, destination, source.Length);

// Clone() - shallow copy
int[] original = { 1, 2, 3 };
int[] clone = (int[])original.Clone();

// Find() - find first element matching predicate
int[] numbers = { 1, 2, 3, 4, 5 };
int first = Array.Find(numbers, n => n > 3);  // 4

// FindAll() - find all matching
int[] all = Array.FindAll(numbers, n => n > 3);  // [4, 5]

// FindIndex() - find index of first match
int index = Array.FindIndex(numbers, n => n > 3);  // 3

// Exists() - check if any element matches
bool exists = Array.Exists(numbers, n => n > 10);  // false

// TrueForAll() - check if all elements match
bool allPositive = Array.TrueForAll(numbers, n => n > 0);  // true

// ConvertAll() - convert all elements
string[] strings = Array.ConvertAll(numbers, n => n.ToString());
```

### Collection Expressions (C# 12.0+)

```csharp
// New syntax: [1, 2, 3]
int[] numbers = [1, 2, 3, 4, 5];

// Works with any collection type
List<int> list = [1, 2, 3];
Span<int> span = [1, 2, 3];

// Spread operator (..)
int[] first = [1, 2, 3];
int[] second = [4, 5, 6];
int[] combined = [..first, ..second];  // [1, 2, 3, 4, 5, 6]

// Mix values and spreads
int[] numbers = [0, ..first, 10, ..second, 20];
// [0, 1, 2, 3, 10, 4, 5, 6, 20]

// Empty collection
int[] empty = [];
```

---

## 5. Strings Deep Dive

### String Properties

```csharp
string text = "Hello World";

// Length - number of characters
int length = text.Length;  // 11

// Indexer - access character by index
char first = text[0];      // 'H'
char last = text[^1];      // 'd' (C# 8.0+)
char fifth = text[4];      // 'o'

// ❌ Cannot modify via indexer (strings are immutable)
// text[0] = 'h';  // Error!
```

### String Methods Reference Table

| Method | Description | Example | Result |
|--------|-------------|---------|--------|
| `Substring(start)` | Get substring from start | `"Hello".Substring(1)` | `"ello"` |
| `Substring(start, length)` | Get substring with length | `"Hello".Substring(1, 3)` | `"ell"` |
| `Replace(old, new)` | Replace all occurrences | `"Hello".Replace("l", "L")` | `"HeLLo"` |
| `Trim()` | Remove whitespace from both ends | `" Hi ".Trim()` | `"Hi"` |
| `TrimStart()` | Remove from start | `" Hi ".TrimStart()` | `"Hi "` |
| `TrimEnd()` | Remove from end | `" Hi ".TrimEnd()` | `" Hi"` |
| `ToUpper()` | Convert to uppercase | `"hello".ToUpper()` | `"HELLO"` |
| `ToLower()` | Convert to lowercase | `"HELLO".ToLower()` | `"hello"` |
| `ToUpperInvariant()` | Culture-independent upper | `"hello".ToUpperInvariant()` | `"HELLO"` |
| `ToLowerInvariant()` | Culture-independent lower | `"HELLO".ToLowerInvariant()` | `"hello"` |
| `StartsWith(value)` | Check if starts with | `"Hello".StartsWith("He")` | `true` |
| `EndsWith(value)` | Check if ends with | `"Hello".EndsWith("lo")` | `true` |
| `Contains(value)` | Check if contains | `"Hello".Contains("ll")` | `true` |
| `Split(separator)` | Split into array | `"a,b,c".Split(',')` | `["a","b","c"]` |
| `Join(separator, array)` | Join array elements | `String.Join("-", arr)` | `"a-b-c"` |
| `IndexOf(value)` | Find first occurrence | `"Hello".IndexOf("l")` | `2` |
| `LastIndexOf(value)` | Find last occurrence | `"Hello".LastIndexOf("l")` | `3` |
| `Insert(index, value)` | Insert at position | `"Hlo".Insert(1, "el")` | `"Hello"` |
| `Remove(start)` | Remove from start | `"Hello".Remove(3)` | `"Hel"` |
| `Remove(start, count)` | Remove count chars | `"Hello".Remove(1, 3)` | `"Ho"` |
| `PadLeft(width)` | Pad left with spaces | `"Hi".PadLeft(5)` | `"   Hi"` |
| `PadLeft(width, char)` | Pad with character | `"Hi".PadLeft(5, '0')` | `"000Hi"` |
| `PadRight(width)` | Pad right | `"Hi".PadRight(5)` | `"Hi   "` |
| `Compare(s1, s2)` | Compare (static) | `String.Compare("a","b")` | `-1` |
| `CompareTo(other)` | Compare instance | `"a".CompareTo("b")` | `-1` |
| `Equals(other)` | Check equality | `"Hi".Equals("Hi")` | `true` |
| `Format(format, args)` | Format string (static) | `String.Format("{0}", 5)` | `"5"` |
| `Concat(strings)` | Concatenate (static) | `String.Concat("a","b")` | `"ab"` |

### String Manipulation

#### Concatenation

```csharp
string first = "Hello";
string second = "World";

// + operator
string result = first + " " + second;  // "Hello World"

// String.Concat()
string result = String.Concat(first, " ", second);

// String.Join()
string[] words = { "Hello", "World" };
string result = String.Join(" ", words);  // "Hello World"

// StringBuilder (for many concatenations)
StringBuilder sb = new StringBuilder();
sb.Append("Hello");
sb.Append(" ");
sb.Append("World");
string result = sb.ToString();  // "Hello World"
```

#### String Interpolation ($"")

```csharp
string name = "John";
int age = 25;

// String interpolation (C# 6.0+)
string message = $"Name: {name}, Age: {age}";
// "Name: John, Age: 25"

// With expressions
string message = $"Next year: {age + 1}";

// With formatting
decimal price = 19.99M;
string formatted = $"Price: {price:C}";  // "Price: $19.99"

// Multiline (C# 10.0+)
string multiline = $"""
    Name: {name}
    Age: {age}
    """;
```

#### Composite Formatting (String.Format)

```csharp
// String.Format()
string message = String.Format("Name: {0}, Age: {1}", name, age);

// Console.WriteLine uses same format
Console.WriteLine("Name: {0}, Age: {1}", name, age);

// With formatting
string formatted = String.Format("Price: {0:C}", 19.99);  // "Price: $19.99"

// Alignment
string aligned = String.Format("{0,10} | {1,-10}", "Right", "Left");
//                                "     Right | Left      "
```

### String vs StringBuilder

**String (Immutable):**
```csharp
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString();  // ❌ Creates 1000 new string objects!
}
```

**StringBuilder (Mutable):**
```csharp
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);  // ✅ Efficient - modifies same object
}
string result = sb.ToString();
```

**StringBuilder Methods:**
```csharp
StringBuilder sb = new StringBuilder();

sb.Append("Hello");           // Add at end
sb.AppendLine("World");       // Add with newline
sb.Insert(5, " Beautiful");   // Insert at position
sb.Remove(5, 10);             // Remove characters
sb.Replace("Hello", "Hi");    // Replace text
sb.Clear();                   // Remove all

string result = sb.ToString();
```

**When to Use:**

| String | StringBuilder |
|--------|---------------|
| Few concatenations (< 5) | Many concatenations |
| Immutability needed | Building string in loop |
| Thread-safe (immutable) | Single-threaded building |
| Small strings | Large strings |

**Performance Comparison:**
```csharp
// ❌ Slow (1000 iterations)
string s = "";
for (int i = 0; i < 1000; i++)
    s += i;  // ~500ms

// ✅ Fast
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append(i);  // ~1ms
```

### Escape Sequences

```csharp
// Common escape sequences
string newline = "Line 1\nLine 2";         // Newline
string tab = "Column1\tColumn2";           // Tab
string backslash = "C:\\Users\\File";      // Backslash
string quote = "He said \"Hello\"";        // Quote
string apostrophe = "It\'s fine";          // Apostrophe (optional in string)

// Unicode
string unicode = "\u0041";                 // 'A'
string emoji = "\U0001F600";               // 😀

// Escape character (C# 13.0+)
string escape = "\e[31mRed Text\e[0m";     // ANSI color codes

// Verbatim string (no escaping)
string path = @"C:\Users\Documents";       // No need to escape \
string multiline = @"Line 1
Line 2
Line 3";

// Raw string literal (C# 11.0+)
string json = """
{
    "name": "John",
    "age": 25
}
""";
```

---

## Common Loop Patterns

### Sum of Numbers
```csharp
int[] numbers = { 1, 2, 3, 4, 5 };
int sum = 0;

for (int i = 0; i < numbers.Length; i++)
{
    sum += numbers[i];
}
// sum = 15
```

### Find Maximum
```csharp
int[] numbers = { 5, 2, 9, 1, 7 };
int max = numbers[0];

for (int i = 1; i < numbers.Length; i++)
{
    if (numbers[i] > max)
    {
        max = numbers[i];
    }
}
// max = 9
```

### Count Occurrences
```csharp
string text = "Hello World";
int count = 0;

foreach (char c in text)
{
    if (c == 'o')
    {
        count++;
    }
}
// count = 2
```

### Reverse Array
```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

for (int i = 0; i < numbers.Length / 2; i++)
{
    int temp = numbers[i];
    numbers[i] = numbers[numbers.Length - 1 - i];
    numbers[numbers.Length - 1 - i] = temp;
}
// numbers = [5, 4, 3, 2, 1]

// Or use Array.Reverse()
Array.Reverse(numbers);
```

### Filter Array
```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6 };
List<int> evens = new List<int>();

foreach (int n in numbers)
{
    if (n % 2 == 0)
    {
        evens.Add(n);
    }
}
// evens = [2, 4, 6]
```

---

## String Manipulation Recipes

### Split and Join
```csharp
// Split CSV
string csv = "John,25,Engineer";
string[] parts = csv.Split(',');  // ["John", "25", "Engineer"]

// Join with delimiter
string[] words = { "Hello", "World" };
string sentence = String.Join(" ", words);  // "Hello World"
```

### Remove Whitespace
```csharp
string text = "  Hello   World  ";

string trimmed = text.Trim();              // "Hello   World"
string noSpaces = text.Replace(" ", "");   // "HelloWorld"
```

### Extract Substring
```csharp
string email = "john@example.com";

int atIndex = email.IndexOf('@');
string username = email.Substring(0, atIndex);        // "john"
string domain = email.Substring(atIndex + 1);         // "example.com"

// Or with Range (C# 8.0+)
string username = email[..atIndex];
string domain = email[(atIndex + 1)..];
```

### Check if String is Numeric
```csharp
string input = "12345";
bool isNumeric = int.TryParse(input, out int result);

// Or with LINQ
bool isNumeric = input.All(char.IsDigit);
```

### Capitalize First Letter
```csharp
string text = "hello";

// Method 1
string capitalized = char.ToUpper(text[0]) + text.Substring(1);  // "Hello"

// Method 2
string capitalized = text[0].ToString().ToUpper() + text[1..];
```

### Reverse String
```csharp
string text = "Hello";

// Method 1: Array.Reverse
char[] chars = text.ToCharArray();
Array.Reverse(chars);
string reversed = new string(chars);  // "olleH"

// Method 2: LINQ
string reversed = new string(text.Reverse().ToArray());
```

---

## Common Pitfalls

### 1. Off-by-One Errors
```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// ❌ Wrong: Includes Length (index out of range)
for (int i = 0; i <= numbers.Length; i++)  // Error!
{
    Console.WriteLine(numbers[i]);
}

// ✅ Correct: Stop before Length
for (int i = 0; i < numbers.Length; i++)
{
    Console.WriteLine(numbers[i]);
}
```

### 2. Modifying Collection During foreach
```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

// ❌ Wrong: Cannot modify collection during foreach
foreach (int n in numbers)
{
    if (n % 2 == 0)
        numbers.Remove(n);  // Exception!
}

// ✅ Correct: Use for loop or ToList()
for (int i = numbers.Count - 1; i >= 0; i--)
{
    if (numbers[i] % 2 == 0)
        numbers.RemoveAt(i);
}

// Or
foreach (int n in numbers.ToList())
{
    if (n % 2 == 0)
        numbers.Remove(n);
}
```

### 3. String Concatenation in Loop
```csharp
// ❌ Inefficient: Creates many objects
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString();
}

// ✅ Efficient: Use StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);
}
string result = sb.ToString();
```

### 4. Forgetting break in switch
```csharp
// ❌ Error: Missing break
switch (value)
{
    case 1:
        Console.WriteLine("One");
        // Error: Control cannot fall through
    case 2:
        Console.WriteLine("Two");
        break;
}

// ✅ Correct: Include break
switch (value)
{
    case 1:
        Console.WriteLine("One");
        break;
    case 2:
        Console.WriteLine("Two");
        break;
}
```

### 5. Infinite Loops
```csharp
// ❌ Infinite: Condition never false
int i = 0;
while (i < 10)
{
    Console.WriteLine(i);
    // Forgot to increment i!
}

// ✅ Correct: Update loop variable
int i = 0;
while (i < 10)
{
    Console.WriteLine(i);
    i++;
}
```

---

## Best Practices

### 1. Use foreach When You Don't Need Index
```csharp
// ❌ Verbose
for (int i = 0; i < names.Length; i++)
{
    Console.WriteLine(names[i]);
}

// ✅ Cleaner
foreach (string name in names)
{
    Console.WriteLine(name);
}
```

### 2. Use Switch Expressions for Simple Mappings
```csharp
// ❌ Verbose
string GetGrade(int score)
{
    if (score >= 90)
        return "A";
    else if (score >= 80)
        return "B";
    else if (score >= 70)
        return "C";
    else
        return "F";
}

// ✅ Concise
string GetGrade(int score) => score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    _ => "F"
};
```

### 3. Use Pattern Matching for Type Checks
```csharp
// ❌ Old way
if (obj is string)
{
    string s = (string)obj;
    Console.WriteLine(s.Length);
}

// ✅ Modern
if (obj is string s)
{
    Console.WriteLine(s.Length);
}
```

### 4. Use String Interpolation Over Concatenation
```csharp
// ❌ Hard to read
string message = "Hello " + name + "! You have " + count + " messages.";

// ✅ Clear
string message = $"Hello {name}! You have {count} messages.";
```

### 5. Validate Loop Bounds
```csharp
// ❌ Risky
for (int i = start; i < end; i++)
{
    Process(data[i]);
}

// ✅ Safe
if (start >= 0 && end <= data.Length)
{
    for (int i = start; i < end; i++)
    {
        Process(data[i]);
    }
}
```

### 6. Use Collection Expressions (C# 12.0+)
```csharp
// ❌ Verbose
int[] numbers = new int[] { 1, 2, 3, 4, 5 };

// ✅ Concise
int[] numbers = [1, 2, 3, 4, 5];
```

### 7. Use Early Returns to Reduce Nesting
```csharp
// ❌ Nested
void ProcessData(string data)
{
    if (!string.IsNullOrEmpty(data))
    {
        if (IsValid(data))
        {
            // Process...
        }
    }
}

// ✅ Early returns
void ProcessData(string data)
{
    if (string.IsNullOrEmpty(data))
        return;
    
    if (!IsValid(data))
        return;
    
    // Process...
}
```

### 8. Use StringBuilder for Repeated Concatenation
```csharp
// ❌ Inefficient in loops
string result = "";
foreach (var item in items)
{
    result += item.ToString();
}

// ✅ Efficient
StringBuilder sb = new StringBuilder();
foreach (var item in items)
{
    sb.Append(item);
}
string result = sb.ToString();
```

---

## Quick Reference Summary

### Control Flow

- **if/else** - Basic branching
- **switch** - Multiple cases
- **switch expression** - Modern, returns value
- **Ternary (?:)** - Inline condition

### Loops

- **for** - Known iterations, need index
- **while** - Condition checked first
- **do-while** - Executes at least once
- **foreach** - Iterate collections

### Jump Statements

- **break** - Exit loop/switch
- **continue** - Skip iteration
- **return** - Exit method
- **goto** - Jump to label (avoid)

### Arrays

- **Single-dimensional** - `int[]`
- **Multi-dimensional** - `int[,]`
- **Jagged** - `int[][]`

### Strings

- **Immutable** - Cannot change after creation
- **Use StringBuilder** - For many concatenations
- **String interpolation** - Modern formatting

---
