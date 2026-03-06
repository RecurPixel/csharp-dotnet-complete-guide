# C# Advanced Features & Version Timeline Quick Reference

---

## Part 1: C# Version Timeline

### C# Evolution Overview

```
2002 ────> 2007 ────> 2012 ────> 2017 ────> 2020 ────> 2024
  │          │          │          │          │          │
C# 1.0     C# 3.0    C# 5.0    C# 7.0    C# 9.0    C# 13.0
  │          │          │          │          │          │
Classes    LINQ      async/    Pattern   Records   params
Delegates           await     Matching            collections
```

### Major Themes by Version

| Era | Versions | Major Theme |
|-----|----------|-------------|
| **Foundation** | C# 1.0-2.0 (2002-2005) | Object-oriented basics, Generics |
| **LINQ Era** | C# 3.0-4.0 (2007-2010) | Query syntax, Functional programming |
| **Async Era** | C# 5.0-6.0 (2012-2015) | Asynchronous programming |
| **Modern Era** | C# 7.0-8.0 (2017-2019) | Pattern matching, Nullable types |
| **Unified .NET** | C# 9.0-13.0 (2020-2024) | Records, Simplification, Performance |

---

## Version-by-Version Features

### C# 1.0 (.NET Framework 1.0, 2002) 🏛️ Foundation

**The Beginning**

```csharp
// Classes and structs
class Person
{
    public string Name;
    public int Age;
}

// Properties
public string Name { get; set; }

// Events and delegates
public event EventHandler Click;
public delegate void MyDelegate();

// All control structures
if, switch, for, while, foreach
```

**Key Features:**

- Classes, structs, interfaces, enums
- Properties, events, delegates
- Value types vs reference types
- Operators and expressions
- Attributes

---

### C# 2.0 (.NET Framework 2.0, 2005) 🔥 Generics!

**Game Changer: Type Safety**

```csharp
// Generics - HUGE!
List<int> numbers = new List<int>();
Dictionary<string, int> ages = new Dictionary<string, int>();

// Nullable types
int? nullableInt = null;
if (nullableInt.HasValue)
{
    int value = nullableInt.Value;
}

// Anonymous methods
button.Click += delegate(object sender, EventArgs e)
{
    Console.WriteLine("Clicked!");
};

// Iterators (yield)
IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}

// Partial types
partial class MyClass { }

// Static classes
static class Utils { }

// Null coalescing operator
string name = username ?? "Guest";
```

**Key Features:**

- ⭐ **Generics** (`List<T>`, `Dictionary<TKey,TValue>`)
- Nullable value types (`int?`, `Nullable<T>`)
- Anonymous methods
- Iterators (`yield return`, `yield break`)
- Partial types (`partial class`)
- Static classes
- Null-coalescing operator (`??`)
- Covariance and contravariance (delegates)

---

### C# 3.0 (.NET Framework 3.5, 2007) 🚀 LINQ Revolution

**Game Changer: Query Syntax**

```csharp
// LINQ - Revolutionary!
var adults = people.Where(p => p.Age >= 18)
                   .OrderBy(p => p.Name)
                   .Select(p => p.Name);

// Lambda expressions
Func<int, int> square = x => x * x;

// Extension methods
public static class StringExtensions
{
    public static bool IsEmail(this string s) => s.Contains("@");
}

// Auto-implemented properties
public string Name { get; set; }

// Object initializers
var person = new Person { Name = "John", Age = 25 };

// Collection initializers
var numbers = new List<int> { 1, 2, 3, 4, 5 };

// Anonymous types
var person = new { Name = "John", Age = 25 };

// Implicit typing (var)
var message = "Hello";  // string inferred

// Expression trees
Expression<Func<int, int>> expr = x => x * x;
```

**Key Features:**

- ⭐ **LINQ** (Language Integrated Query)
- ⭐ **Lambda expressions** (`x => x * 2`)
- Extension methods (`this` parameter)
- Anonymous types
- Auto-implemented properties
- Object and collection initializers
- Expression trees (`Expression<T>`)
- Partial methods
- Implicit typing (`var`)

---

### C# 4.0 (.NET Framework 4.0, 2010) 🔀 Dynamic

```csharp
// Dynamic binding
dynamic obj = GetDynamicObject();
obj.SomeMethod();  // Resolved at runtime

// Named arguments
PrintMessage(message: "Hello", times: 3);

// Optional parameters
void PrintMessage(string message, int times = 1) { }

// Generic variance
IEnumerable<string> strings = new List<string>();
IEnumerable<object> objects = strings;  // Covariant
```

**Key Features:**

- **dynamic** type (late binding)
- Named arguments
- Optional parameters
- Generic covariance and contravariance
- Embedded interop types (COM)

---

### C# 5.0 (.NET Framework 4.5, 2012) ⏳ Async Revolution

**Game Changer: Asynchronous Programming**

```csharp
// async/await - Revolutionary!
public async Task<string> DownloadDataAsync(string url)
{
    using (HttpClient client = new HttpClient())
    {
        string data = await client.GetStringAsync(url);
        return data;
    }
}

// Caller information attributes
void Log(string message,
    [CallerMemberName] string memberName = "",
    [CallerFilePath] string filePath = "",
    [CallerLineNumber] int lineNumber = 0)
{
    Console.WriteLine($"{memberName} at {filePath}:{lineNumber} - {message}");
}
```

**Key Features:**

- ⭐ **async/await** (asynchronous programming)
- Caller information attributes (`CallerMemberName`, `CallerFilePath`, `CallerLineNumber`)

---

### C# 6.0 (.NET Framework 4.6, 2015) ✨ Syntax Sugar

```csharp
// String interpolation
string message = $"Hello, {name}! You are {age} years old.";

// Expression-bodied members
public int Square(int x) => x * x;
public string FullName => $"{FirstName} {LastName}";

// Null-conditional operator
int? length = name?.Length;
string first = names?[0];

// Auto-property initializers
public string Name { get; set; } = "Default";
public List<int> Numbers { get; } = new List<int>();

// nameof operator
if (age < 0)
    throw new ArgumentException(nameof(age));

// Index initializers
var dict = new Dictionary<int, string>
{
    [1] = "one",
    [2] = "two"
};

// Exception filters
try { }
catch (Exception ex) when (ex.Message.Contains("specific"))
{
    // Only catch if condition is true
}
```

**Key Features:**

- ⭐ **String interpolation** (`$"{name}"`)
- ⭐ **Null-conditional operators** (`?.`, `?[]`)
- Expression-bodied members
- Auto-property initializers
- **nameof** operator
- Index initializers
- Exception filters (`when`)
- `using static`

---

### C# 7.0 (.NET Framework 4.7, 2017) 🎯 Pattern Matching

**Game Changer: Pattern Matching & Tuples**

```csharp
// out variables (inline)
if (int.TryParse(input, out int result))
{
    Console.WriteLine(result);  // result in scope
}

// Tuples (ValueTuple)
(string name, int age) GetPerson() => ("John", 25);
var person = GetPerson();
Console.WriteLine(person.name);

// Tuple deconstruction
var (name, age) = GetPerson();

// Pattern matching with is
if (obj is string s)
{
    Console.WriteLine(s.Length);
}

// Pattern matching in switch
switch (obj)
{
    case string s:
        return s.Length;
    case int i:
        return i;
    default:
        return 0;
}

// Local functions
int Add(int x, int y)
{
    return x + y;
    
    int Square(int n) => n * n;  // Local function
}

// Ref returns and locals
ref int GetFirstElement(int[] array) => ref array[0];

// Throw expressions
string name = input ?? throw new ArgumentNullException();

// Binary literals and digit separators
int binary = 0b1010_1111;
int million = 1_000_000;

// Expression-bodied everything
public Person(string name) => Name = name;  // Constructor
~Person() => Console.WriteLine("Destructor");  // Finalizer
public int this[int index] => data[index];     // Indexer
```

**Key Features:**

- ⭐ **out variables** (inline declaration)
- ⭐ **Tuples** (`(int, string)` syntax)
- ⭐ **Pattern matching** (`is`, `switch`)
- **Deconstruction**
- Local functions
- Ref returns and ref locals
- Generalized async return types (`ValueTask<T>`)
- Expression-bodied constructors/finalizers
- Throw expressions
- Binary literals (`0b`) and digit separators (`_`)

---

### C# 7.1-7.3 (2017-2018) 🔧 Refinements

```csharp
// C# 7.1: Async Main
static async Task Main(string[] args)
{
    await DoWorkAsync();
}

// C# 7.1: Default literal
int x = default;  // Instead of default(int)

// C# 7.1: Inferred tuple names
var person = (name, age);  // Names inferred

// C# 7.2: readonly struct
readonly struct Point
{
    public int X { get; }
    public int Y { get; }
}

// C# 7.2: ref struct (stack-only)
ref struct StackOnlyStruct { }

// C# 7.2: in parameters (read-only ref)
void Process(in LargeStruct data) { }

// C# 7.2: private protected access modifier
private protected void Method() { }

// C# 7.3: Tuple equality
var t1 = (1, 2);
var t2 = (1, 2);
bool equal = t1 == t2;  // true
```

**Key Features:**

- Async Main
- Default literal expressions
- Inferred tuple names
- **readonly struct** (C# 7.2)
- **ref struct** (C# 7.2)
- **in parameters** (C# 7.2)
- **private protected** modifier (C# 7.2)
- Tuple equality (C# 7.3)

---

### C# 8.0 (.NET Core 3.0, 2019) ✅ Nullable References

**Game Changer: Nullable Reference Types**

```csharp
// Nullable reference types
#nullable enable
string? nullableString = null;
string nonNullableString = "Hello";  // Cannot be null

// Switch expressions
string result = dayOfWeek switch
{
    DayOfWeek.Monday => "Start of week",
    DayOfWeek.Friday => "End of week",
    _ => "Middle of week"
};

// Property patterns
string result = person switch
{
    { Age: >= 18 } => "Adult",
    { Age: < 18 } => "Minor"
};

// Tuple patterns
string result = (x, y) switch
{
    (0, 0) => "Origin",
    (var a, 0) => "On X-axis",
    (0, var b) => "On Y-axis",
    _ => "Somewhere else"
};

// Positional patterns
Point point = new Point(0, 0);
string result = point switch
{
    (0, 0) => "Origin",
    (var x, var y) => $"({x}, {y})"
};

// Indices and ranges
int[] numbers = { 1, 2, 3, 4, 5 };
int last = numbers[^1];           // 5 (last element)
int secondLast = numbers[^2];     // 4
int[] slice = numbers[1..4];      // {2, 3, 4}
int[] fromStart = numbers[..3];   // {1, 2, 3}
int[] toEnd = numbers[3..];       // {4, 5}

// using declarations
using var file = new FileStream("file.txt", FileMode.Open);
// Disposed at end of scope

// Static local functions
void Method()
{
    static int Add(int x, int y) => x + y;  // Cannot capture variables
}

// Null-coalescing assignment
int? x = null;
x ??= 10;  // Assign only if null

// Async streams
async IAsyncEnumerable<int> GetNumbersAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);
        yield return i;
    }
}

await foreach (var number in GetNumbersAsync())
{
    Console.WriteLine(number);
}

// Default interface methods
interface ILogger
{
    void Log(string message);
    
    void LogError(string message)  // Default implementation
    {
        Log($"ERROR: {message}");
    }
}
```

**Key Features:**

- ⭐ **Nullable reference types** (`string?`, `#nullable enable`)
- ⭐ **Switch expressions** (concise syntax)
- **Property patterns** in pattern matching
- **Tuple patterns**
- **Positional patterns**
- ⭐ **Indices and ranges** (`^`, `..`)
- **using declarations** (simplified)
- Static local functions
- **Null-coalescing assignment** (`??=`)
- ⭐ **Async streams** (`IAsyncEnumerable<T>`, `await foreach`)
- Default interface methods

---

### C# 9.0 (.NET 5.0, 2020) 📝 Records

**Game Changer: Records & Init**

```csharp
// Records (immutable by default)
public record Person(string Name, int Age);

var person = new Person("John", 25);
Console.WriteLine(person.Name);  // John

// with expressions (non-destructive mutation)
var older = person with { Age = 26 };

// Value-based equality
var p1 = new Person("John", 25);
var p2 = new Person("John", 25);
Console.WriteLine(p1 == p2);  // true (value equality)

// Init-only setters
public class Person
{
    public string Name { get; init; }
    public int Age { get; init; }
}

var person = new Person { Name = "John", Age = 25 };
// person.Name = "Jane";  // Error! Cannot modify after initialization

// Top-level statements
using System;
Console.WriteLine("Hello, World!");
// No class, no Main method needed!

// Target-typed new
Person person = new("John", 25);  // Type inferred

// Pattern matching enhancements
// Relational patterns
int score = 85;
string grade = score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    _ => "F"
};

// Logical patterns (and, or, not)
bool IsLetter(char c) => c is (>= 'a' and <= 'z') or (>= 'A' and <= 'Z');

// Covariant return types
class Animal
{
    public virtual Animal Clone() => new Animal();
}

class Dog : Animal
{
    public override Dog Clone() => new Dog();  // Can return Dog instead of Animal
}
```

**Key Features:**

- ⭐ **Records** (immutable data classes)
- **Init-only setters** (`init` accessor)
- **Top-level statements** (no Main method)
- **with expressions** (non-destructive mutation)
- Target-typed `new`
- **Relational patterns** (`>=`, `<`, etc.)
- **Logical patterns** (`and`, `or`, `not`)
- Covariant return types
- Function pointers
- Native integers (`nint`, `nuint`)
- Module initializers

---

### C# 10.0 (.NET 6.0, 2021) 🌍 Global Usings

```csharp
// Global using directives (at top of any file)
global using System;
global using System.Collections.Generic;
global using System.Linq;

// File-scoped namespaces (reduced indentation)
namespace MyApp;  // Applies to whole file

class MyClass
{
    // No extra indentation needed!
}

// Record structs
public record struct Point(int X, int Y);

// with on structs
Point p1 = new(1, 2);
Point p2 = p1 with { X = 10 };

// Lambda improvements
var parse = (string s) => int.Parse(s);  // Type inferred

// Attributes on lambdas
var handler = [HttpGet] (HttpContext context) => context.Response.WriteAsync("Hello");

// CallerArgumentExpression
void Assert(bool condition, [CallerArgumentExpression("condition")] string? message = null)
{
    if (!condition)
        throw new Exception($"Assertion failed: {message}");
}

Assert(x > 0);  // message will be "x > 0"

// Constant interpolated strings
const string Name = "World";
const string Message = $"Hello, {Name}!";  // Was not allowed before C# 10
```

**Key Features:**

- ⭐ **Global using directives** (project-wide usings)
- ⭐ **File-scoped namespaces** (less indentation)
- **Record structs**
- **with expressions** on structs
- Lambda improvements (natural types, attributes)
- **CallerArgumentExpression**
- Constant interpolated strings
- Extended property patterns
- Enhanced `#line` pragma
- Implicit usings (.NET 6 SDK feature)

---

### C# 11.0 (.NET 7.0, 2022) 🔤 Raw Strings

```csharp
// Raw string literals (no escape sequences)
string json = """
{
    "name": "John",
    "age": 25
}
""";

// With interpolation
string name = "John";
string json = $$"""
{
    "name": "{{name}}",
    "age": 25
}
""";

// List patterns
int[] numbers = { 1, 2, 3, 4, 5 };

string result = numbers switch
{
    [] => "Empty",
    [1] => "Just one",
    [1, 2] => "One and two",
    [1, .., 5] => "Starts with 1, ends with 5",
    [var first, .., var last] => $"First: {first}, Last: {last}"
};

// required members
public class Person
{
    public required string Name { get; init; }
    public required int Age { get; init; }
}

var person = new Person { Name = "John", Age = 25 };  // Must set both
// var invalid = new Person { Name = "John" };  // Error! Age is required

// UTF-8 string literals
ReadOnlySpan<byte> utf8 = "Hello"u8;

// Generic math
T AddNumbers<T>(T a, T b) where T : INumber<T>
{
    return a + b;  // Works with int, double, decimal, etc.
}

// Static abstract members in interfaces
interface IFactory<T>
{
    static abstract T Create();
}

// File-local types
file class InternalHelper { }  // Only visible in this file

// Newlines in string interpolations
string message = $"Hello {
    GetLongExpression()
}!";
```

**Key Features:**

- ⭐ **Raw string literals** (`"""..."""`)
- ⭐ **List patterns** (`[1, 2, ..]`)
- ⭐ **required members** (force initialization)
- **Generic math support** (`INumber<T>`)
- **Static abstract members** in interfaces
- UTF-8 string literals (`"text"u8`)
- Pattern match `Span<char>`
- Newlines in interpolations
- File-local types (`file` modifier)
- Auto-default structs
- `nameof` with parameters

---

### C# 12.0 (.NET 8.0, 2023) 🏗️ Primary Constructors

```csharp
// Primary constructors for classes
public class Person(string name, int age)  // Parameters declared here
{
    public string Name => name;  // Can use parameters
    public int Age => age;
    
    public void PrintInfo()
    {
        Console.WriteLine($"{name} is {age} years old");  // Can use anywhere
    }
}

var person = new Person("John", 25);

// Primary constructors for structs
public struct Point(int x, int y)
{
    public int X => x;
    public int Y => y;
}

// Collection expressions
int[] numbers = [1, 2, 3, 4, 5];  // Instead of new int[] { }
List<int> list = [1, 2, 3];
Span<int> span = [1, 2, 3];

// Spread operator in collections
int[] first = [1, 2, 3];
int[] second = [4, 5, 6];
int[] combined = [..first, ..second];  // [1, 2, 3, 4, 5, 6]
int[] extended = [0, ..first, 10, ..second, 20];

// Lambda optional parameters
var greet = (string name = "World") => $"Hello, {name}!";
greet();        // "Hello, World!"
greet("John");  // "Hello, John!"

// ref readonly parameters
void ProcessData(ref readonly LargeStruct data)
{
    // Can read but not modify data
    // Passed by reference for performance
}

// Alias any type
using Point = (int x, int y);
Point p = (10, 20);

// Inline arrays
[System.Runtime.CompilerServices.InlineArray(10)]
public struct Buffer
{
    private int _element0;
}

// Experimental attributes
[Experimental("MYLIB001")]
public class ExperimentalFeature { }
```

**Key Features:**

- ⭐ **Primary constructors** (for classes and structs)
- ⭐ **Collection expressions** (`[1, 2, 3]`)
- **Spread operator** (`..` in collections)
- Lambda optional parameters
- **ref readonly parameters**
- Alias any type
- Inline arrays
- Experimental attributes
- Interceptors

---

### C# 13.0 (.NET 9.0, 2024) 📦 Latest

```csharp
// params collections (not just arrays)
void Print(params IEnumerable<int> numbers)  // Can be List, Span, etc.
{
    foreach (var n in numbers)
        Console.WriteLine(n);
}

Print(1, 2, 3);  // Works with any collection type

// New Lock type (System.Threading.Lock)
Lock myLock = new();
lock (myLock)
{
    // Critical section (better performance than object lock)
}

// \e escape sequence (ANSI escape codes)
string redText = "\e[31mRed Text\e[0m";

// Implicit indexer access in object initializers
var timer = new System.Timers.Timer
{
    Interval = 1000,
    [0] = "value"  // Can use indexers in initializers
};

// Partial properties and indexers
public partial class MyClass
{
    public partial string Name { get; set; }
    public partial int this[int index] { get; set; }
}

// ref and unsafe in iterators and async
async IAsyncEnumerable<int> GetNumbers(ref int start)  // ref now allowed
{
    for (int i = start; i < 10; i++)
    {
        await Task.Delay(100);
        yield return i;
    }
}

// Overload resolution priority
[OverloadResolutionPriority(1)]
public void Method(string s) { }  // Preferred

[OverloadResolutionPriority(0)]
public void Method(object o) { }  // Fallback
```

**Key Features:**

- **params collections** (not just arrays)
- New **Lock** type (`System.Threading.Lock`)
- **\e escape sequence** (ANSI codes)
- Implicit indexer access in object initializers
- **Partial properties and indexers**
- **ref and unsafe** in iterators and async
- **Overload resolution priority** attribute

---

## Part 2: Advanced Features Deep Dive

### Tuples and Deconstruction

#### System.Tuple (C# 4.0) - Legacy ❌

```csharp
// Old way (don't use)
Tuple<string, int> person = Tuple.Create("John", 25);
string name = person.Item1;
int age = person.Item2;
```

#### ValueTuple (C# 7.0+) - Modern ✅

```csharp
// Unnamed tuple
(string, int) person = ("John", 25);
Console.WriteLine(person.Item1);  // John

// Named tuple
(string Name, int Age) person = ("John", 25);
Console.WriteLine(person.Name);  // John
Console.WriteLine(person.Age);   // 25

// Return from method
(string Name, int Age) GetPerson()
{
    return ("John", 25);
}

// Deconstruction
var (name, age) = GetPerson();
Console.WriteLine($"{name} is {age}");

// Discard unwanted values
var (name, _) = GetPerson();  // Ignore age

// Tuple equality
var t1 = (1, "test");
var t2 = (1, "test");
Console.WriteLine(t1 == t2);  // true (value equality)

// Swap without temp variable
(a, b) = (b, a);
```

**When to Use Tuples:**

- ✅ Returning multiple values from methods
- ✅ Grouping related data temporarily
- ✅ LINQ projections
- ❌ Don't use for complex types (use classes/records)
- ❌ Don't use for public APIs (use proper types)

**Performance:**

- ValueTuple is a struct (stack-allocated)
- Much faster than System.Tuple (class/heap)
- Zero allocation for local tuples

---

### Pattern Matching Complete Guide

#### Type Patterns (C# 7.0)

```csharp
if (obj is string s)
{
    Console.WriteLine($"String length: {s.Length}");
}

// Switch
switch (obj)
{
    case string s:
        return s.Length;
    case int i:
        return i;
}
```

#### Constant Patterns (C# 7.0)

```csharp
if (value is null)
    return;

if (value is 42)
    Console.WriteLine("The answer!");
```

#### Property Patterns (C# 8.0)

```csharp
if (person is { Age: >= 18, Name: "John" })
{
    Console.WriteLine("Adult John");
}

string result = person switch
{
    { Age: < 13 } => "Child",
    { Age: >= 13, Age: < 20 } => "Teen",
    { Age: >= 20 } => "Adult"
};
```

#### Tuple Patterns (C# 8.0)

```csharp
string GetQuadrant(int x, int y) => (x, y) switch
{
    (0, 0) => "Origin",
    (>0, >0) => "Quadrant I",
    (<0, >0) => "Quadrant II",
    (<0, <0) => "Quadrant III",
    (>0, <0) => "Quadrant IV"
};
```

#### Positional Patterns (C# 8.0)

```csharp
public record Point(int X, int Y);

string Classify(Point p) => p switch
{
    (0, 0) => "Origin",
    (var x, 0) => "On X-axis",
    (0, var y) => "On Y-axis",
    _ => "Somewhere"
};
```

#### Relational Patterns (C# 9.0)

```csharp
string GetGrade(int score) => score switch
{
    < 0 => "Invalid",
    < 60 => "F",
    < 70 => "D",
    < 80 => "C",
    < 90 => "B",
    <= 100 => "A",
    _ => "Invalid"
};
```

#### Logical Patterns (C# 9.0)

```csharp
// and, or, not
bool IsLetter(char c) => c is (>= 'a' and <= 'z') or (>= 'A' and <= 'Z');

bool IsValidAge(int age) => age is > 0 and < 120;

bool IsNotNull(object obj) => obj is not null;
```

#### List Patterns (C# 11.0)

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

string result = numbers switch
{
    [] => "Empty",
    [1] => "Just one",
    [1, 2, 3, 4, 5] => "Exact match",
    [1, ..] => "Starts with 1",
    [.., 5] => "Ends with 5",
    [1, .., 5] => "Starts with 1, ends with 5",
    [var first, .., var last] => $"First: {first}, Last: {last}"
};
```

---

### Nullable Reference Types (C# 8.0+)

```csharp
// Enable nullable reference types
#nullable enable

// Nullable reference type
string? nullableString = null;  // OK
string nonNullableString = null;  // Warning!

// Null-forgiving operator (!)
string text = nullableString!;  // "I know it's not null"

// Checking for null
if (nullableString != null)
{
    Console.WriteLine(nullableString.Length);  // No warning
}

// Null-conditional
int? length = nullableString?.Length;

// Nullable context options
#nullable enable     // Enable
#nullable disable    // Disable
#nullable restore    // Restore to project setting

// In .csproj
<Nullable>enable</Nullable>
```

**Migration Strategy:**

1. Enable nullable in new projects
2. For existing projects, enable file-by-file
3. Fix warnings gradually
4. Use `!` operator sparingly

---

### Indices and Ranges (C# 8.0+)

```csharp
int[] numbers = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// Index operator (^)
int last = numbers[^1];      // 9 (last element)
int secondLast = numbers[^2]; // 8
int third = numbers[^3];      // 7

// Range operator (..)
int[] slice = numbers[2..5];    // {2, 3, 4}
int[] fromStart = numbers[..3]; // {0, 1, 2}
int[] toEnd = numbers[7..];     // {7, 8, 9}
int[] all = numbers[..];        // {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

// With variables
Index last = ^1;
Range middle = 2..^2;
int lastNum = numbers[last];
int[] middleNums = numbers[middle];

// Implicit index access (C# 13.0+)
var timer = new System.Timers.Timer
{
    [0] = value  // Can use in initializers
};
```

---

### Span\<T\> and Memory\<T\> (C# 7.2+)

**Purpose:** Zero-allocation slicing of arrays, strings, or stack memory.

```csharp
// Span<T> - stack-only type
int[] array = { 1, 2, 3, 4, 5 };
Span<int> span = array;
Span<int> slice = span[1..4];  // {2, 3, 4} - no allocation!

// ReadOnlySpan<T>
ReadOnlySpan<char> text = "Hello, World!";
ReadOnlySpan<char> hello = text[..5];  // "Hello"

// stackalloc with Span (safe)
Span<int> numbers = stackalloc int[100];  // Stack allocation

// Memory<T> - heap-safe alternative
Memory<int> memory = array.AsMemory();
Memory<int> memorySlice = memory[1..4];

// Use cases
void ProcessData(ReadOnlySpan<byte> data)
{
    // Efficient - no allocation
}
```

**Key Differences:**

- **Span\<T\>**: Stack-only, cannot be field, fastest
- **Memory\<T\>**: Can be stored in fields, slightly slower
- **ReadOnlySpan\<T\>**: Immutable span

---

### ref struct and ref Safety (C# 7.2+)

```csharp
// ref struct (stack-only type)
ref struct StackOnlyStruct
{
    public int Value;
}

// Restrictions:
// - Cannot be boxed
// - Cannot be field in regular class
// - Cannot implement interfaces
// - Cannot be used in async methods

// ref returns
ref int GetFirst(int[] array)
{
    return ref array[0];
}

int[] numbers = { 1, 2, 3 };
ref int first = ref GetFirst(numbers);
first = 100;  // Modifies array[0]

// ref locals
ref int localRef = ref numbers[0];

// ref readonly (C# 12.0+)
void Process(ref readonly LargeStruct data)
{
    // Can read but not modify
}

// ref struct interfaces (C# 13.0+)
// Now ref structs can implement interfaces
```

---

### Iterators and yield (C# 2.0+)

```csharp
// Iterator with yield return
IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}

foreach (var num in GetNumbers())
{
    Console.WriteLine(num);  // 1, 2, 3
}

// Lazy evaluation
IEnumerable<int> Generate(int count)
{
    for (int i = 0; i < count; i++)
    {
        Console.WriteLine($"Generating {i}");
        yield return i;
    }
}

var numbers = Generate(1000000);  // No execution yet!
var first = numbers.First();      // Only generates first

// yield break (early exit)
IEnumerable<int> GetUntilNegative(int[] numbers)
{
    foreach (var num in numbers)
    {
        if (num < 0)
            yield break;  // Stop iteration
        yield return num;
    }
}
```

**Behind the Scenes:**

- Compiler generates state machine
- Maintains state between calls
- Efficient memory usage

---

### Async Streams (C# 8.0+)

```csharp
// IAsyncEnumerable<T>
async IAsyncEnumerable<int> GetNumbersAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);  // Async operation
        yield return i;
    }
}

// Consume with await foreach
await foreach (var number in GetNumbersAsync())
{
    Console.WriteLine(number);  // Processes as they arrive
}

// With cancellation
async IAsyncEnumerable<int> GetNumbersAsync(
    [EnumeratorCancellation] CancellationToken token = default)
{
    for (int i = 0; i < 10; i++)
    {
        token.ThrowIfCancellationRequested();
        await Task.Delay(100, token);
        yield return i;
    }
}
```

**Use Cases:**

- Streaming API responses
- Database cursors
- Real-time data feeds
- Paginated results

---

### Modern C# Simplifications

#### Top-level Statements (C# 9.0+)

```csharp
// Old way
using System;

namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}

// New way (C# 9.0+)
using System;
Console.WriteLine("Hello, World!");

// With async
await Task.Delay(1000);
Console.WriteLine("Done!");

// Access args
Console.WriteLine(args[0]);
```

#### Global Usings (C# 10.0+)

```csharp
// In any .cs file (typically GlobalUsings.cs)
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;

// Or in .csproj (implicit usings)
<ImplicitUsings>enable</ImplicitUsings>
```

#### File-scoped Namespaces (C# 10.0+)

```csharp
// Old way
namespace MyCompany.MyApp
{
    public class MyClass
    {
        public void Method() { }
    }
}

// New way (C# 10.0+)
namespace MyCompany.MyApp;  // Applies to whole file

public class MyClass
{
    public void Method() { }
}
```

#### Raw String Literals (C# 11.0+)

```csharp
// No escape sequences needed
string json = """
{
    "name": "John",
    "age": 25,
    "address": "C:\Users\John"
}
""";

// With interpolation
string name = "John";
string json = $$"""
{
    "name": "{{name}}",
    "age": 25
}
""";
```

#### Collection Expressions (C# 12.0+)

```csharp
// Arrays
int[] numbers = [1, 2, 3, 4, 5];

// Lists
List<int> list = [1, 2, 3];

// Spread operator
int[] first = [1, 2, 3];
int[] second = [4, 5, 6];
int[] combined = [..first, ..second];  // [1, 2, 3, 4, 5, 6]

// With additions
int[] extended = [0, ..first, 10, ..second, 20];
// [0, 1, 2, 3, 10, 4, 5, 6, 20]
```

---

## Part 3: Quick Reference Cards

### Version Feature Matrix

| Feature | Version | .NET Version |
|---------|---------|--------------|
| Generics | C# 2.0 | .NET Framework 2.0 |
| LINQ | C# 3.0 | .NET Framework 3.5 |
| dynamic | C# 4.0 | .NET Framework 4.0 |
| async/await | C# 5.0 | .NET Framework 4.5 |
| String interpolation | C# 6.0 | .NET Framework 4.6 |
| Tuples | C# 7.0 | .NET Framework 4.7 |
| Nullable reference types | C# 8.0 | .NET Core 3.0 |
| Records | C# 9.0 | .NET 5.0 |
| Global usings | C# 10.0 | .NET 6.0 |
| Raw strings | C# 11.0 | .NET 7.0 |
| Primary constructors | C# 12.0 | .NET 8.0 |
| params collections | C# 13.0 | .NET 9.0 |

### Modern C# Checklist (What to Use Now)

✅ **Essential (Use Always):**

- Lambda expressions `x => x * 2`
- LINQ `Where`, `Select`, `OrderBy`
- String interpolation `$"{name}"`
- Null-conditional `?.`, `?[]`
- async/await
- Collection initializers `new List<int> { 1, 2, 3 }`

✅ **Modern (C# 8.0+):**

- Nullable reference types `#nullable enable`
- Switch expressions `value switch { ... }`
- Indices and ranges `[^1]`, `[..]`
- using declarations `using var file = ...`

✅ **Latest (C# 9.0-13.0):**

- Records `public record Person(string Name, int Age)`
- Init-only setters `{ get; init; }`
- Top-level statements (simple programs)
- Global usings `global using System;`
- File-scoped namespaces `namespace MyApp;`
- Raw string literals `""" ... """`
- Collection expressions `[1, 2, 3]`
- Primary constructors `class Person(string name)`

❌ **Legacy (Avoid):**

- System.Tuple (use ValueTuple)
- Anonymous methods (use lambdas)
- Old string formatting (use interpolation)

---

## Migration Guide

### From Old C# to Modern

#### Tuples
```csharp
// ❌ Old (C# 4.0)
Tuple<string, int> person = Tuple.Create("John", 25);
string name = person.Item1;

// ✅ New (C# 7.0+)
(string Name, int Age) person = ("John", 25);
string name = person.Name;
```

#### Anonymous Methods to Lambdas
```csharp
// ❌ Old (C# 2.0)
Func<int, int> square = delegate(int x) { return x * x; };

// ✅ New (C# 3.0+)
Func<int, int> square = x => x * x;
```

#### String Formatting
```csharp
// ❌ Old
string message = string.Format("Hello, {0}! You are {1} years old.", name, age);

// ✅ New (C# 6.0+)
string message = $"Hello, {name}! You are {age} years old.";
```

#### Null Checking
```csharp
// ❌ Old
if (obj != null)
{
    int length = obj.ToString().Length;
}

// ✅ New (C# 6.0+)
int? length = obj?.ToString()?.Length;
```

#### Switch
```csharp
// ❌ Old
string result;
switch (value)
{
    case 1:
        result = "One";
        break;
    case 2:
        result = "Two";
        break;
    default:
        result = "Other";
        break;
}

// ✅ New (C# 8.0+)
string result = value switch
{
    1 => "One",
    2 => "Two",
    _ => "Other"
};
```

---

## When to Use What?

### Choose the Right Feature

**Need to return multiple values?**

- Use **tuples** `(string, int)` for simple cases
- Use **records** for complex types

**Need immutable data?**

- Use **records** with `init` properties
- Use **readonly struct** for value types

**Need to check for null?**

- Use **null-conditional** `?.`
- Use **null-coalescing** `??`, `??=`
- Enable **nullable reference types** `#nullable enable`

**Need pattern matching?**

- Use **switch expressions** for simple cases
- Use **property patterns** for complex conditions
- Use **list patterns** for collection matching

**Need efficient slicing?**

- Use **Span\<T\>** for performance-critical code
- Use **Memory\<T\>** when need to store in fields

**Need async sequences?**
- Use **IAsyncEnumerable\<T\>** for streaming data
- Use **await foreach** to consume

---

**Guide Complete!** This comprehensive timeline and feature reference will help you understand C# evolution and use modern features effectively! 📘