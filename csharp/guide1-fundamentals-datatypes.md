# C# Fundamentals & Data Types Quick Reference

---

## 1. C# Introduction & Setup

### What is C#?
**C#** (pronounced "C-Sharp") is a modern, object-oriented programming language developed by Microsoft in 2000.

**Key Characteristics:**

- Type-safe and memory-managed
- Object-oriented (classes, inheritance, polymorphism)
- Component-oriented (properties, events, attributes)
- Compiled to Intermediate Language (IL), then JIT-compiled
- Cross-platform (.NET Core/.NET 5+)

### .NET Ecosystem Overview
```
.NET Framework (2002-present)  → Windows only, legacy
.NET Core (2016-2020)          → Cross-platform, modern
.NET 5/6/7/8/9 (2020-present)  → Unified platform, recommended
```

**Relationship:**

- **C#** = Programming language
- **.NET** = Runtime and libraries (like JRE for Java)
- **CLR** = Common Language Runtime (executes IL code)

### Installing Development Tools

**Option 1: Visual Studio** (Windows, macOS)

- Full-featured IDE
- Best for Windows development
- Community Edition is free
- Download: visualstudio.microsoft.com

**Option 2: Visual Studio Code** (Windows, macOS, Linux)

- Lightweight editor
- Install C# extension
- Cross-platform
- Download: code.visualstudio.com

**Option 3: JetBrains Rider** (Commercial)

- Powerful cross-platform IDE

### First Program Structure

```csharp
// Traditional structure (C# 1.0 - 8.0)
using System;

namespace HelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

```csharp
// Modern structure with top-level statements (C# 9.0+)
Console.WriteLine("Hello, World!");
```

**Program Components:**

- `using` - Import namespaces
- `namespace` - Organize code
- `class` - Define types
- `Main()` - Entry point
- `Console.WriteLine()` - Output to console

### Comments

```csharp
// Single-line comment

/* Multi-line comment
   Can span multiple lines
   Useful for longer explanations */

/// <summary>
/// XML documentation comment (triple slash)
/// Used for generating documentation
/// </summary>
/// <param name="name">Parameter description</param>
/// <returns>Return value description</returns>
public string GetGreeting(string name)
{
    return $"Hello, {name}!";
}
```

**Best Practice:** Use comments to explain "why", not "what" (code should be self-explanatory).

---

## 2. Identifiers & Variables

### Naming Rules (Enforced by Compiler)

✅ **Valid:**

- Must start with letter or underscore: `name`, `_count`
- Can contain letters, digits, underscores: `user1`, `first_name`
- Case-sensitive: `Name` ≠ `name`

❌ **Invalid:**

- Cannot start with digit: `1user` ❌
- Cannot use keywords: `int` ❌ (unless prefixed with `@`: `@int` ✅)
- Cannot contain spaces or special characters: `first name` ❌, `user#1` ❌

### Naming Conventions (Best Practices)

| Type | Convention | Example |
|------|------------|---------|
| Local variables | camelCase | `firstName`, `totalCount` |
| Parameters | camelCase | `userName`, `itemId` |
| Private fields | _camelCase (underscore prefix) | `_userCount`, `_isActive` |
| Public fields | PascalCase (avoid, use properties) | `UserCount` |
| Constants | PascalCase or UPPER_CASE | `MaxValue`, `MAX_SIZE` |
| Properties | PascalCase | `FirstName`, `IsActive` |
| Methods | PascalCase | `GetUserName()`, `Calculate()` |
| Classes | PascalCase | `UserAccount`, `OrderProcessor` |
| Interfaces | IPascalCase (I prefix) | `IComparable`, `IDisposable` |

### Variable Declaration Patterns

```csharp
// Explicit type
int age = 25;
string name = "John";

// Multiple variables (same type)
int x = 10, y = 20, z = 30;

// Declaration without initialization (default value)
int count;          // 0 (default for int)
string text;        // null (default for reference types)
bool isActive;      // false (default for bool)

// Initialization later
int value;
value = 100;
```

### var Keyword (C# 3.0+)
**Implicitly typed local variables** - compiler infers type from initializer.

```csharp
var age = 25;               // Inferred as int
var name = "John";          // Inferred as string
var price = 19.99;          // Inferred as double
var isValid = true;         // Inferred as bool
var numbers = new int[5];   // Inferred as int[]

// Must have initializer
var x;  // ❌ Error: Cannot infer type
```

**When to Use var:**

- ✅ When type is obvious: `var user = new User();`
- ✅ With LINQ queries: `var results = from x in list select x;`
- ✅ With long generic types: `var dict = new Dictionary<string, List<int>>();`
- ❌ When type is not clear: `var result = GetData();` (what is result?)

### dynamic Keyword (C# 4.0+)
**Bypasses compile-time type checking** - type resolved at runtime.

```csharp
dynamic value = 10;
value = "Hello";      // OK - can change type
value = true;         // OK

int result = value + 5;  // Resolved at runtime
                         // Throws exception if value is not numeric
```

**When to Use dynamic:**

- Interop with COM objects
- Working with dynamic languages (Python, JavaScript)
- JSON deserialization (avoid if possible, use strong types)

**⚠️ Warning:** Loses IntelliSense, compile-time safety, and performance. Use sparingly!

### Constants: const vs readonly

#### const - Compile-Time Constant
```csharp
public const int MaxUsers = 100;
public const string AppName = "MyApp";
public const double Pi = 3.14159;

// Value must be known at compile time
// Implicitly static
// Cannot be changed
```

**Restrictions:**

- Value must be compile-time constant
- Can only be applied to primitive types and strings
- Implicitly static (no `static` keyword needed)

#### readonly - Runtime Constant
```csharp
public readonly int MaxUsers;
public readonly DateTime StartDate;

public MyClass()
{
    MaxUsers = 100;              // Can set in constructor
    StartDate = DateTime.Now;    // Can use runtime values
}

// Can be instance or static
public static readonly string DefaultName = "Unknown";
```

**Comparison:**

| Feature | const | readonly |
|---------|-------|----------|
| When set | Compile time | Constructor or declaration |
| Can use runtime values | ❌ No | ✅ Yes |
| Scope | Implicitly static | Instance or static |
| Types allowed | Primitives, strings | Any type |
| Can be changed | Never | In constructor only |

### Literals

```csharp
// Integer literals
int decimal = 42;
int hex = 0x2A;           // Hexadecimal (42)
int binary = 0b101010;    // Binary (42) - C# 7.0+
int large = 1_000_000;    // Digit separator (1000000) - C# 7.0+

// Floating-point literals
float f = 3.14F;          // F or f suffix
double d = 3.14;          // Default (no suffix) or D/d
decimal m = 3.14M;        // M or m suffix

// Character literals
char letter = 'A';
char newline = '\n';
char tab = '\t';
char unicode = '\u0041';  // 'A'

// String literals
string text = "Hello";
string empty = "";

// Boolean literals
bool isTrue = true;
bool isFalse = false;

// Null literal
string nothing = null;
int? nullableInt = null;
```

---

## 3. Data Types Deep Dive

### Value Types vs Reference Types

**Value Types:**

- Stored directly in memory (usually stack)
- Contain the actual data
- Copied by value
- Examples: int, double, bool, struct, enum

**Reference Types:**

- Stored on heap, variable holds reference (pointer)
- Copied by reference (both variables point to same object)
- Examples: string, array, class, interface, delegate

**Memory Diagram:**
```
STACK                          HEAP
┌──────────────┐              ┌──────────────┐
│ int x = 10   │              │              │
│ [10]         │              │  Person obj  │
│              │              │  {           │
│ Person p ────┼─────────────>│    Name="Jo" │
│ [reference]  │              │    Age=25    │
│              │              │  }           │
└──────────────┘              └──────────────┘
```

**Copy Behavior:**
```csharp
// Value type - copy by value
int x = 10;
int y = x;      // y gets a COPY of 10
y = 20;         // x is still 10

// Reference type - copy by reference
Person p1 = new Person { Name = "John" };
Person p2 = p1;      // p2 points to SAME object
p2.Name = "Jane";    // p1.Name is also "Jane"!
```

### Built-in Value Types Table

#### Integral Types

| Type | Size (bytes) | Range | Default | Suffix | .NET Type |
|------|--------------|-------|---------|--------|-----------|
| `sbyte` | 1 | -128 to 127 | 0 | - | `System.SByte` |
| `byte` | 1 | 0 to 255 | 0 | - | `System.Byte` |
| `short` | 2 | -32,768 to 32,767 | 0 | - | `System.Int16` |
| `ushort` | 2 | 0 to 65,535 | 0 | - | `System.UInt16` |
| `int` | 4 | -2.1B to 2.1B | 0 | - | `System.Int32` |
| `uint` | 4 | 0 to 4.2B | 0 | U or u | `System.UInt32` |
| `long` | 8 | -9.2E+18 to 9.2E+18 | 0 | L or l | `System.Int64` |
| `ulong` | 8 | 0 to 1.8E+19 | 0 | UL or ul | `System.UInt64` |

**Usage:**
```csharp
byte age = 25;              // Small positive numbers
short temperature = -40;     // Small signed numbers
int count = 1000000;        // Default integer type
uint positiveOnly = 4000000000U;  // U suffix
long bigNumber = 9000000000L;     // L suffix
ulong hugeNumber = 18000000000UL; // UL suffix
```

#### Floating-Point Types

| Type | Size (bytes) | Precision | Range | Default | Suffix | .NET Type |
|------|--------------|-----------|-------|---------|--------|-----------|
| `float` | 4 | 7 digits | ±1.5E-45 to ±3.4E+38 | 0.0F | F or f | `System.Single` |
| `double` | 8 | 15-16 digits | ±5.0E-324 to ±1.7E+308 | 0.0D | D or d (optional) | `System.Double` |
| `decimal` | 16 | 28-29 digits | ±1.0E-28 to ±7.9E+28 | 0.0M | M or m | `System.Decimal` |

**When to Use:**

- `float` - Graphics, scientific calculations (lower precision OK)
- `double` - General-purpose (default for decimal literals)
- `decimal` - Financial calculations, currency (no rounding errors)

```csharp
float pi = 3.14F;
double gravity = 9.81;         // Or 9.81D
decimal price = 19.99M;

// Precision comparison
float f = 0.1F + 0.1F + 0.1F;
double d = 0.1 + 0.1 + 0.1;
decimal m = 0.1M + 0.1M + 0.1M;

Console.WriteLine(f == 0.3F);   // false (precision loss)
Console.WriteLine(d == 0.3);    // false (precision loss)
Console.WriteLine(m == 0.3M);   // true (exact)
```

#### Other Value Types

| Type | Size (bytes) | Values | Default | .NET Type |
|------|--------------|--------|---------|-----------|
| `bool` | 1 | `true`, `false` | `false` | `System.Boolean` |
| `char` | 2 | Unicode character | `'\0'` | `System.Char` |

```csharp
bool isActive = true;
char grade = 'A';
char newline = '\n';
char unicode = '\u0041';  // 'A'
```

### String Type (Reference Type!)

**Key Characteristics:**

- Immutable (cannot be changed after creation)
- Reference type (stored on heap)
- Sequence of characters

```csharp
string name = "John";
string empty = "";
string nullString = null;
```

#### String vs string
```csharp
// These are IDENTICAL (lowercase is alias)
string s1 = "Hello";    // Alias (recommended)
String s2 = "Hello";    // .NET type

// Both compile to System.String
```
**Best Practice:** Use lowercase `string` (C# convention).

#### String Immutability
```csharp
string original = "Hello";
string modified = original + " World";

// original is STILL "Hello" (new string created for modified)
// "Hello" object unchanged in memory
```

**Why Immutable?**

- Thread-safe
- Can be shared/cached safely
- String literals reused

**Performance Impact:**
```csharp
// ❌ Bad: Creates many string objects
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString();  // Creates new string each iteration!
}

// ✅ Good: Use StringBuilder for concatenation
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);
}
string result = sb.ToString();
```

#### String Interpolation (C# 6.0+)
```csharp
string name = "John";
int age = 25;

// Old way
string message = "Name: " + name + ", Age: " + age;

// String interpolation (recommended)
string message = $"Name: {name}, Age: {age}";

// With expressions
string message = $"Next year: {age + 1}";

// With formatting
decimal price = 19.99M;
string formatted = $"Price: {price:C}";  // $19.99 (currency)
```

#### Verbatim Strings (C# 1.0+)
```csharp
// Regular string (need escaping)
string path1 = "C:\\Users\\John\\Documents";

// Verbatim string (@ prefix - no escaping needed)
string path2 = @"C:\Users\John\Documents";

// Multi-line
string multiLine = @"Line 1
Line 2
Line 3";

// Quote character (double it)
string quote = @"He said ""Hello""";  // He said "Hello"
```

#### Raw String Literals (C# 11.0+)
```csharp
// Triple quotes (no escaping needed at all!)
string json = """
{
    "name": "John",
    "age": 25,
    "email": "john@example.com"
}
""";

// Useful for: JSON, XML, regex patterns, code snippets
string regex = """^\d{3}-\d{2}-\d{4}$""";  // No need to escape \
```

### Object Type (The Universal Base)

**Every type inherits from `object` (System.Object).**

```csharp
object obj = 10;           // Boxing (int → object)
object obj2 = "Hello";     // String → object
object obj3 = new Person(); // Custom type → object

// Can store anything, but lose type information
// Need casting to get back
int value = (int)obj;      // Unboxing
```

**When to Use:**

- Collections before generics (ArrayList - legacy)
- Truly polymorphic code (rare)
- Usually: Use generics (`List<T>`) instead

### Type Hierarchy Diagram

```
                    System.Object (object)
                           │
        ┌──────────────────┴──────────────────┐
        │                                     │
   Value Types                         Reference Types
   (System.ValueType)                         │
        │                    ┌─────────┬──────┴────┬─────────┐
   ┌────┴────┐               │         │           │         │
   │         │            string    array      delegate   class
Struct      Enum             │         │           │         │
   │         │            String    Array     Delegate   (user)
   │         │
┌──┴──┬──────┴────┬────┐
│     │           │    │
int  bool       char  (user)
double
decimal
```

**Key Points:**

- Everything inherits from `object`
- Value types inherit from `System.ValueType` (which inherits from `object`)
- Reference types directly inherit from `object` (or other classes)

---

## 4. Nullable Types

### Problem: Value Types Can't Be Null

```csharp
int age = null;  // ❌ Error: Cannot assign null to int
```

**But sometimes you need "no value":**

- Database columns (NULL)
- Optional parameters
- Uninitialized state

### Nullable Value Types (C# 2.0+)

```csharp
// Syntax: Type? is shorthand for Nullable<Type>
int? age = null;              // OK
int? count = 10;              // OK

// Longhand (equivalent)
Nullable<int> age = null;
```

**All value types can be nullable:**
```csharp
int? nullableInt = null;
double? nullableDouble = null;
bool? nullableBool = null;
DateTime? nullableDate = null;
```

### Working with Nullable Types

```csharp
int? count = null;

// Check if has value
if (count.HasValue)
{
    int value = count.Value;  // Get value (throws if null!)
    Console.WriteLine(value);
}

// Safe access
int actual = count ?? 0;  // Use 0 if null
```

**Properties:**

- `HasValue` - bool, true if has value
- `Value` - Get value (throws `InvalidOperationException` if null)

### Null-Coalescing Operator (??) - C# 2.0+

```csharp
int? count = null;

// If count is null, use 0
int actual = count ?? 0;

// Chain multiple
int? a = null;
int? b = null;
int? c = 10;
int result = a ?? b ?? c ?? 0;  // 10
```

**Works with reference types too:**
```csharp
string name = null;
string displayName = name ?? "Unknown";  // "Unknown"
```

### Null-Coalescing Assignment (??=) - C# 8.0+

```csharp
int? count = null;

// Assign only if null
count ??= 10;  // count becomes 10

// Equivalent to:
// if (count == null) count = 10;

// If not null, keeps value
int? age = 25;
age ??= 30;  // Still 25 (not null)
```

### Null-Conditional Operator (?.) - C# 6.0+

```csharp
Person person = null;

// Without null-conditional (need if check)
if (person != null)
{
    string name = person.Name;
}

// With null-conditional (returns null if person is null)
string name = person?.Name;  // null (no exception)

// Chain multiple
string city = person?.Address?.City;

// With method calls
int? length = person?.Name?.Length;

// Array/indexer access
int? first = array?[0];
```

### Nullable Reference Types (C# 8.0+)

**Before C# 8.0:** Reference types could always be null (by default).

**C# 8.0+:** Can enable nullable reference type checking.

```csharp
// Enable nullable reference types
#nullable enable

// Non-nullable (should never be null)
string name = "John";     // OK
name = null;              // ⚠️ Warning

// Nullable (can be null)
string? optionalName = null;  // OK
optionalName = "Jane";        // OK

// Compiler warns on potential null access
Console.WriteLine(optionalName.Length);  // ⚠️ Warning: possible null

// Safe access
if (optionalName != null)
{
    Console.WriteLine(optionalName.Length);  // ✅ No warning
}

// Or use null-conditional
int? length = optionalName?.Length;

#nullable disable  // Turn off (optional)
```

**Benefits:**

- Catch null reference bugs at compile time
- Document intent (nullable vs non-nullable)
- Safer code

**Enable in project:**
```xml
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>
```

---

## 5. Enums

### Enum Declaration and Usage

**Enum** = Named constants (enumeration type)

```csharp
// Define enum
public enum DayOfWeek
{
    Sunday,      // 0 (default)
    Monday,      // 1
    Tuesday,     // 2
    Wednesday,   // 3
    Thursday,    // 4
    Friday,      // 5
    Saturday     // 6
}

// Use enum
DayOfWeek today = DayOfWeek.Monday;

if (today == DayOfWeek.Friday)
{
    Console.WriteLine("Weekend soon!");
}
```

### Explicit Values

```csharp
public enum Status
{
    Pending = 1,
    Processing = 2,
    Completed = 3,
    Failed = 4
}

// Can skip numbers
public enum Priority
{
    Low = 1,
    Medium = 5,
    High = 10
}
```

### Underlying Types

**Default:** `int` (can be any integral type)

```csharp
// Default (int)
public enum Color { Red, Green, Blue }

// Specify underlying type
public enum Color : byte
{
    Red = 1,
    Green = 2,
    Blue = 3
}

// Can use: byte, sbyte, short, ushort, int, uint, long, ulong
```

**When to use non-int:**

- Save memory (byte for small enums)
- Interop with other systems
- Match database column type

### Flags Attribute [Flags]

**Use for bitmasks** - combine multiple enum values.

```csharp
[Flags]
public enum FileAccess
{
    None = 0,       // 0000
    Read = 1,       // 0001
    Write = 2,      // 0010
    Execute = 4,    // 0100
    Delete = 8      // 1000
}

// Combine with bitwise OR
FileAccess access = FileAccess.Read | FileAccess.Write;

// Check with bitwise AND
if ((access & FileAccess.Read) == FileAccess.Read)
{
    Console.WriteLine("Can read");
}

// HasFlag method (easier but slower)
if (access.HasFlag(FileAccess.Write))
{
    Console.WriteLine("Can write");
}
```

**Best Practice for [Flags]:**
```csharp
[Flags]
public enum Permissions
{
    None = 0,           // Always include 0
    Read = 1,           // 1 << 0 = 1
    Write = 2,          // 1 << 1 = 2
    Execute = 4,        // 1 << 2 = 4
    Delete = 8,         // 1 << 3 = 8
    All = Read | Write | Execute | Delete  // Combine all
}
```

### Enum Methods

```csharp
// Parse string to enum
Status status = (Status)Enum.Parse(typeof(Status), "Pending");

// TryParse (safe, returns bool)
if (Enum.TryParse("Completed", out Status result))
{
    Console.WriteLine(result);  // Status.Completed
}

// Get all values
foreach (DayOfWeek day in Enum.GetValues(typeof(DayOfWeek)))
{
    Console.WriteLine(day);
}

// Get all names
string[] names = Enum.GetNames(typeof(DayOfWeek));

// Check if defined
bool exists = Enum.IsDefined(typeof(Status), 3);  // true

// Convert to int
int value = (int)Status.Completed;  // 3

// Convert from int
Status status = (Status)3;  // Status.Completed

// Get name as string
string name = Status.Completed.ToString();  // "Completed"
```

### Bitwise Operations with Enums

```csharp
[Flags]
public enum Options
{
    None = 0,
    Option1 = 1,
    Option2 = 2,
    Option3 = 4,
    Option4 = 8
}

Options opts = Options.Option1 | Options.Option3;  // Combine (OR)

opts |= Options.Option2;   // Add flag (OR)
opts &= ~Options.Option1;  // Remove flag (AND NOT)

bool hasOption = (opts & Options.Option3) != 0;  // Check flag
```

---

## 6. Type Casting & Conversion

### Implicit vs Explicit Casting

**Implicit Casting** - Automatic, no data loss
```csharp
int i = 100;
long l = i;        // int → long (OK, no data loss)

float f = 3.14F;
double d = f;      // float → double (OK)
```

**Implicit Conversion Hierarchy:**
```
byte → short → int → long → float → double
                            ↓
                         decimal
```

**Explicit Casting** - Manual, potential data loss
```csharp
double d = 3.14;
int i = (int)d;           // 3 (decimal part lost)

long l = 1000;
int i = (int)l;           // OK if value fits

// Risk: Data loss or overflow
long big = 9000000000;
int i = (int)big;         // ⚠️ Overflow! (wrong value)
```

### Cast Operator: (type)

```csharp
double d = 9.7;
int i = (int)d;           // 9 (truncates, not rounds)

// With overflow
byte b = (byte)300;       // Overflow: 44 (300 % 256)

// Reference types
object obj = "Hello";
string s = (string)obj;   // OK

// Wrong type
object obj = 123;
string s = (string)obj;   // ⚠️ InvalidCastException at runtime!
```

### as Operator

**Safe casting** - Returns `null` if cast fails (reference types only).

```csharp
object obj = "Hello";
string s = obj as string;    // "Hello"

object obj2 = 123;
string s2 = obj2 as string;  // null (no exception)

// Check result
if (s2 != null)
{
    Console.WriteLine(s2.Length);
}
```

**Comparison:**
```csharp
// (type) - throws exception if fails
string s = (string)obj;

// as - returns null if fails
string s = obj as string;
if (s != null) { /* use s */ }

// Modern pattern
if (obj is string s)  // Pattern matching (C# 7.0+)
{
    // Use s here
}
```

### is Operator

**Type checking**

```csharp
// C# 1.0 - Basic type check
object obj = "Hello";
if (obj is string)
{
    string s = (string)obj;
    Console.WriteLine(s);
}

// C# 7.0+ - Pattern matching (check and cast)
if (obj is string s)
{
    Console.WriteLine(s);  // s is available here
}

// Check for null
if (obj is null)
{
    Console.WriteLine("obj is null");
}

// Check for not null (C# 9.0+)
if (obj is not null)
{
    Console.WriteLine("obj has value");
}
```

### typeof Operator

**Get `Type` object at compile time**

```csharp
Type intType = typeof(int);
Type stringType = typeof(string);
Type listType = typeof(List<>);        // Generic type definition

Console.WriteLine(intType.Name);       // "Int32"
Console.WriteLine(intType.FullName);   // "System.Int32"

// Compare types
object obj = 123;
if (obj.GetType() == typeof(int))
{
    Console.WriteLine("obj is int");
}
```

### Parse vs TryParse

**Parse** - Throws exception if fails
```csharp
string input = "123";
int number = int.Parse(input);  // 123

string bad = "abc";
int x = int.Parse(bad);         // ⚠️ FormatException!
```

**TryParse** - Returns bool, safe
```csharp
string input = "123";
if (int.TryParse(input, out int number))
{
    Console.WriteLine(number);  // 123
}

string bad = "abc";
if (int.TryParse(bad, out int x))
{
    // Won't execute
}
else
{
    Console.WriteLine("Invalid number");
}
```

**All numeric types have Parse/TryParse:**
```csharp
int.Parse("123")
long.Parse("1000000")
double.Parse("3.14")
decimal.Parse("19.99")
bool.Parse("true")
DateTime.Parse("2024-01-01")
```

### Convert Class Methods

**Universal conversion methods**

```csharp
// To integer
int i = Convert.ToInt32("123");
int i2 = Convert.ToInt32(3.14);      // 3 (rounds)
int i3 = Convert.ToInt32(true);      // 1

// To double
double d = Convert.ToDouble("3.14");
double d2 = Convert.ToDouble(123);

// To string
string s = Convert.ToString(123);
string s2 = Convert.ToString(true);

// To boolean
bool b = Convert.ToBoolean(1);       // true
bool b2 = Convert.ToBoolean(0);      // false

// Base conversions
string binary = Convert.ToString(42, 2);   // "101010"
string hex = Convert.ToString(42, 16);     // "2A"
int fromHex = Convert.ToInt32("2A", 16);   // 42
```

**Convert vs Cast:**
```csharp
// Cast - type must be compatible
double d = 3.14;
int i = (int)d;           // 3 (truncates)

// Convert - tries to convert value
int i2 = Convert.ToInt32(d);  // 3 (rounds)

Convert.ToInt32(3.5);     // 4 (rounds to nearest even)
Convert.ToInt32(3.14);    // 3
(int)3.5;                 // 3 (truncates)
```

### Boxing and Unboxing

**Boxing** - Value type → Object (reference type)
```csharp
int value = 123;
object obj = value;  // Boxing (allocates on heap)
```

**Unboxing** - Object → Value type
```csharp
object obj = 123;
int value = (int)obj;  // Unboxing (must cast to exact type)
```

**Memory Impact:**
```
BEFORE BOXING:           AFTER BOXING:
Stack: [123]            Stack: [reference] ──> Heap: [123]
```

**Performance Cost:**
```csharp
// ❌ Bad: Boxing in loop
for (int i = 0; i < 1000000; i++)
{
    object obj = i;  // Boxes 1 million times!
}

// ✅ Good: Avoid boxing
List<int> list = new List<int>();  // Generic, no boxing
for (int i = 0; i < 1000000; i++)
{
    list.Add(i);  // No boxing
}
```

**Common Boxing Scenarios:**
```csharp
// ArrayList (legacy) - boxes value types
ArrayList list = new ArrayList();
list.Add(123);       // Boxing

// String concatenation with value types
Console.WriteLine("Value: " + 123);  // Boxes 123

// Generic collections - NO boxing
List<int> numbers = new List<int>();
numbers.Add(123);    // No boxing ✅
```

---

## 7. Operators

### Arithmetic Operators

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Addition | `5 + 3` | `8` |
| `-` | Subtraction | `5 - 3` | `2` |
| `*` | Multiplication | `5 * 3` | `15` |
| `/` | Division | `5 / 2` | `2` (integer division) |
|     |          | `5.0 / 2` | `2.5` (floating-point) |
| `%` | Modulus (remainder) | `5 % 2` | `1` |
| `++` | Increment | `x++` or `++x` | Adds 1 |
| `--` | Decrement | `x--` or `--x` | Subtracts 1 |

```csharp
// Integer division vs floating-point
int a = 5 / 2;        // 2 (truncates)
double b = 5.0 / 2;   // 2.5
double c = 5 / 2.0;   // 2.5

// Modulus
int remainder = 17 % 5;  // 2

// Increment/Decrement
int x = 5;
int y = x++;  // y = 5, x = 6 (post-increment: use then increment)
int z = ++x;  // z = 7, x = 7 (pre-increment: increment then use)
```

### Comparison Operators

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `==` | Equal | `5 == 5` | `true` |
| `!=` | Not equal | `5 != 3` | `true` |
| `<` | Less than | `3 < 5` | `true` |
| `>` | Greater than | `5 > 3` | `true` |
| `<=` | Less than or equal | `5 <= 5` | `true` |
| `>=` | Greater than or equal | `5 >= 3` | `true` |

```csharp
int a = 10, b = 20;

if (a < b)          // true
if (a == 10)        // true
if (a != b)         // true

// String comparison
string s1 = "hello";
string s2 = "hello";
if (s1 == s2)       // true (value comparison for strings)
```

### Logical Operators

| Operator | Name        | Example  | Description       |     |     |     |                     |
| -------- | ----------- | -------- | ----------------- | --- | --- | --- | ------------------- |
| `&&`     | Logical AND | `a && b` | True if both true |     |     |     |                     |
| `        |             | `        | Logical OR        | `a  |     | b`  | True if either true |
| `!`      | Logical NOT | `!a`     | Inverts boolean   |     |     |     |                     |

```csharp
bool a = true, b = false;

bool result1 = a && b;   // false (both must be true)
bool result2 = a || b;   // true (at least one true)
bool result3 = !a;       // false (inverts true)

// Short-circuit evaluation
if (x > 0 && y / x > 2)  // Safe: if x <= 0, y/x not evaluated
{
}
```

### Bitwise Operators

| Operator | Name | Example | Description |
|----------|------|---------|-------------|
| `&` | Bitwise AND | `a & b` | AND each bit |
| `|` | Bitwise OR | `a | b` | OR each bit |
| `^` | Bitwise XOR | `a ^ b` | XOR each bit |
| `~` | Bitwise NOT | `~a` | Inverts all bits |
| `<<` | Left shift | `a << 2` | Shift bits left |
| `>>` | Right shift | `a >> 2` | Shift bits right |

```csharp
int a = 5;   // 0101
int b = 3;   // 0011

int and = a & b;   // 0001 = 1
int or = a | b;    // 0111 = 7
int xor = a ^ b;   // 0110 = 6
int not = ~a;      // 1010 (in 32-bit: ...11111010)

int left = a << 1;   // 1010 = 10 (multiply by 2)
int right = a >> 1;  // 0010 = 2 (divide by 2)
```

### Assignment Operators

| Operator | Example   | Equivalent to |      |        |     |
| -------- | --------- | ------------- | ---- | ------ | --- |
| `=`      | `x = 5`   | Assign 5 to x |      |        |     |
| `+=`     | `x += 3`  | `x = x + 3`   |      |        |     |
| `-=`     | `x -= 3`  | `x = x - 3`   |      |        |     |
| `*=`     | `x *= 3`  | `x = x * 3`   |      |        |     |
| `/=`     | `x /= 3`  | `x = x / 3`   |      |        |     |
| `%=`     | `x %= 3`  | `x = x % 3`   |      |        |     |
| `&=`     | `x &= 3`  | `x = x & 3`   |      |        |     |
| `        | =`        | `x            | = 3` | `x = x | 3`  |
| `^=`     | `x ^= 3`  | `x = x ^ 3`   |      |        |     |
| `<<=`    | `x <<= 2` | `x = x << 2`  |      |        |     |
| `>>=`    | `x >>= 2` | `x = x >> 2`  |      |        |     |

```csharp
int x = 10;
x += 5;   // x = 15
x *= 2;   // x = 30
x /= 3;   // x = 10
```

### Null Operators

| Operator | Name | Example | Description |
|----------|------|---------|-------------|
| `??` | Null-coalescing | `a ?? b` | If a is null, use b |
| `??=` | Null-coalescing assignment | `a ??= b` | If a is null, assign b (C# 8.0+) |
| `?.` | Null-conditional | `a?.Method()` | Call if not null (C# 6.0+) |
| `?[]` | Null-conditional index | `a?[0]` | Access if not null (C# 6.0+) |

```csharp
string name = null;
string display = name ?? "Unknown";  // "Unknown"

int? count = null;
count ??= 10;  // count becomes 10

Person person = null;
string city = person?.Address?.City;  // null (no exception)

int[] array = null;
int? first = array?[0];  // null (no exception)
```

### Other Operators

| Operator | Name | Example | Description |
|----------|------|---------|-------------|
| `?:` | Ternary | `a ? b : c` | If a, then b, else c |
| `is` | Type test | `obj is string` | Check if type |
| `as` | Type cast | `obj as string` | Safe cast (null if fails) |
| `typeof` | Type | `typeof(int)` | Get Type object |
| `sizeof` | Size | `sizeof(int)` | Size in bytes |
| `nameof` | Name of | `nameof(variable)` | Get name as string (C# 6.0+) |

```csharp
// Ternary
int max = (a > b) ? a : b;

// is
if (obj is string)
{
}

// as
string s = obj as string;

// typeof
Type t = typeof(int);

// sizeof (unsafe context or with primitives)
int size = sizeof(int);  // 4

// nameof (C# 6.0+)
string name = nameof(myVariable);  // "myVariable"
```

### Operator Precedence (High to Low)

| Level | Operators | Associativity |
|-------|-----------|---------------|
| 1 | `x.y`, `x?.y`, `x?[y]`, `f(x)`, `a[i]`, `x++`, `x--`, `new`, `typeof`, `sizeof`, `checked`, `unchecked` | Left |
| 2 | `+x`, `-x`, `!x`, `~x`, `++x`, `--x`, `(T)x` | Right |
| 3 | `*`, `/`, `%` | Left |
| 4 | `+`, `-` | Left |
| 5 | `<<`, `>>` | Left |
| 6 | `<`, `>`, `<=`, `>=`, `is`, `as` | Left |
| 7 | `==`, `!=` | Left |
| 8 | `&` | Left |
| 9 | `^` | Left |
| 10 | `|` | Left |
| 11 | `&&` | Left |
| 12 | `||` | Left |
| 13 | `??` | Right |
| 14 | `?:` | Right |
| 15 | `=`, `+=`, `-=`, `*=`, `/=`, etc. | Right |

**Example:**
```csharp
int result = 2 + 3 * 4;      // 14 (not 20) - * before +
int result = (2 + 3) * 4;    // 20 - parentheses override

bool check = x > 0 && y < 10 || z == 5;
// Evaluated as: ((x > 0) && (y < 10)) || (z == 5)

// Use parentheses for clarity!
bool check = ((x > 0) && (y < 10)) || (z == 5);
```

---

## 8. Scope of Variables

### Local Scope (Method)
```csharp
public void MyMethod()
{
    int x = 10;  // Local variable
    // x is only accessible within MyMethod
}
// x does not exist here
```

### Class Scope (Fields)
```csharp
public class Person
{
    private int age;        // Field - accessible in all methods
    private string name;
    
    public void SetAge(int value)
    {
        age = value;  // Can access field
    }
    
    public int GetAge()
    {
        return age;   // Can access field
    }
}
```

### Block Scope
```csharp
public void Example()
{
    if (true)
    {
        int x = 10;  // x only exists in this block
    }
    // x does not exist here
    
    for (int i = 0; i < 10; i++)
    {
        // i only exists in for loop
    }
    // i does not exist here
}
```

### Parameter Scope
```csharp
public void MyMethod(int parameter)
{
    // parameter is accessible throughout method
    int local = parameter * 2;
}
// parameter does not exist here
```

### Static Scope
```csharp
public class Counter
{
    private static int count = 0;  // Shared across all instances
    
    public void Increment()
    {
        count++;  // Same count for all instances
    }
}
```

### Variable Shadowing (Hiding)
```csharp
public class Example
{
    private int x = 10;  // Field
    
    public void Method()
    {
        int x = 20;  // Local variable (shadows field)
        Console.WriteLine(x);       // 20 (local)
        Console.WriteLine(this.x);  // 10 (field)
    }
}
```

**Scope Summary:**

| Scope Type | Declared | Lifetime | Access |
|------------|----------|----------|--------|
| Local | Inside method/block | Until end of block | Only in block |
| Parameter | Method signature | Method execution | Throughout method |
| Field | Class level | Object lifetime | All instance methods |
| Static | Class level (static) | Program lifetime | All code in class |

---

## 9. params Keyword

### Variable-Length Arguments

**params** allows passing variable number of arguments to a method.

### Traditional params with Arrays (C# 1.0+)

```csharp
// Method with params
public void PrintNumbers(params int[] numbers)
{
    foreach (int num in numbers)
    {
        Console.WriteLine(num);
    }
}

// Call with any number of arguments
PrintNumbers(1);                    // 1 argument
PrintNumbers(1, 2, 3);              // 3 arguments
PrintNumbers(1, 2, 3, 4, 5);        // 5 arguments

// Or pass array directly
int[] array = { 1, 2, 3 };
PrintNumbers(array);

// Call with no arguments
PrintNumbers();  // Empty array
```

### Rules for params

- Must be last parameter in method signature
- Only one params parameter allowed
- Can be combined with other parameters

```csharp
// ✅ Valid: params last
public void Log(string message, params object[] args)
{
}

// ❌ Invalid: params not last
public void Invalid(params int[] numbers, string text)
{
}

// ❌ Invalid: multiple params
public void Invalid(params int[] a, params string[] b)
{
}
```

### params with Span\<T\> (C# 12.0+)

```csharp
// More efficient (stack allocation)
public void ProcessNumbers(params Span<int> numbers)
{
    foreach (int num in numbers)
    {
        Console.Write(num + " ");
    }
}

// Usage (same as array)
ProcessNumbers(1, 2, 3, 4);
```

### params with Collections (C# 13.0+)

```csharp
// Can use any collection type
public void ProcessItems(params List<string> items)
{
}

public void ProcessSet(params HashSet<int> numbers)
{
}

// Usage
ProcessItems("a", "b", "c");
ProcessSet(1, 2, 3);
```

### Real-World Examples

```csharp
// String.Format uses params
string formatted = string.Format("Name: {0}, Age: {1}", name, age);

// Console.WriteLine uses params
Console.WriteLine("Values: {0}, {1}, {2}", a, b, c);

// Sum method
public int Sum(params int[] numbers)
{
    int total = 0;
    foreach (int num in numbers)
    {
        total += num;
    }
    return total;
}

int result = Sum(1, 2, 3, 4, 5);  // 15

// String concatenation
public string Concatenate(string separator, params string[] words)
{
    return string.Join(separator, words);
}

string result = Concatenate(" ", "Hello", "World", "!");
// "Hello World !"
```

---

## Quick Reference Tables

### Data Types Quick Reference

| Category | Type | Size | Range | Default | Use Case |
|----------|------|------|-------|---------|----------|
| **Integer** | `byte` | 1 | 0 to 255 | 0 | Small positive numbers |
| | `sbyte` | 1 | -128 to 127 | 0 | Small signed numbers |
| | `short` | 2 | -32K to 32K | 0 | Small integers |
| | `ushort` | 2 | 0 to 65K | 0 | Small positive integers |
| | `int` | 4 | -2.1B to 2.1B | 0 | **Default integer** |
| | `uint` | 4 | 0 to 4.2B | 0 | Large positive integers |
| | `long` | 8 | -9.2E+18 to 9.2E+18 | 0 | Very large integers |
| | `ulong` | 8 | 0 to 1.8E+19 | 0 | Very large positive |
| **Float** | `float` | 4 | ±1.5E-45 to ±3.4E+38 | 0.0F | Graphics, low precision |
| | `double` | 8 | ±5E-324 to ±1.7E+308 | 0.0D | **Default decimal** |
| | `decimal` | 16 | ±1.0E-28 to ±7.9E+28 | 0.0M | **Financial/currency** |
| **Other** | `bool` | 1 | true, false | false | Boolean values |
| | `char` | 2 | Unicode characters | '\0' | Single character |
| **Reference** | `string` | - | Text | null | Text/strings |
| | `object` | - | Any type | null | Universal base |

### Type Conversion Methods

| Scenario | Method | Example |
|----------|--------|---------|
| String → Number | `Parse()` | `int.Parse("123")` |
| String → Number (safe) | `TryParse()` | `int.TryParse("123", out int x)` |
| Any → Any | `Convert.ToXxx()` | `Convert.ToInt32("123")` |
| Number → String | `.ToString()` | `123.ToString()` |
| Upcast (safe) | Implicit | `long l = 10;` |
| Downcast (risky) | `(type)` | `int i = (int)longValue;` |
| Safe reference cast | `as` | `string s = obj as string;` |
| Type check | `is` | `if (obj is string)` |
| Type check + cast | `is` pattern | `if (obj is string s)` |

### Operator Quick Reference

| Category | Operators | Example |
|----------|-----------|---------|
| **Arithmetic** | `+  -  *  /  %  ++  --` | `x + y`, `x++` |
| **Comparison** | `==  !=  <  >  <=  >=` | `x == y` |
| **Logical** | `&&  ||  !` | `a && b` |
| **Bitwise** | `&  |  ^  ~  <<  >>` | `a & b` |
| **Assignment** | `=  +=  -=  *=  /=  %=` | `x += 5` |
| **Null** | `??  ??=  ?.  ?[]` | `x ?? 0`, `obj?.Method()` |
| **Other** | `?:  is  as  typeof  sizeof  nameof` | `x ? y : z` |

### Variable Declaration Patterns

```csharp
// Explicit type
int age = 25;

// Implicit type (var)
var name = "John";

// Multiple variables
int x = 1, y = 2, z = 3;

// Constant
const double PI = 3.14159;

// Readonly (runtime constant)
readonly DateTime created = DateTime.Now;

// Nullable value type
int? optionalAge = null;

// Nullable reference type (C# 8.0+)
string? optionalName = null;

// Dynamic type
dynamic value = 10;
```

---

## Common Pitfalls

### 1. Integer Division
```csharp
// ❌ Wrong: Integer division truncates
int result = 5 / 2;         // 2 (not 2.5)

// ✅ Correct: Use floating-point
double result = 5.0 / 2;    // 2.5
double result = (double)5 / 2;  // 2.5
```

### 2. String Immutability
```csharp
// ❌ Inefficient: Creates many string objects
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString();  // New string each time!
}

// ✅ Efficient: Use StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);
}
string result = sb.ToString();
```

### 3. Boxing Overhead
```csharp
// ❌ Bad: Boxing in legacy collections
ArrayList list = new ArrayList();
list.Add(123);  // Boxes int to object

// ✅ Good: Use generic collections
List<int> list = new List<int>();
list.Add(123);  // No boxing
```

### 4. Nullable Value Access
```csharp
int? count = null;

// ❌ Dangerous: Throws if null
int value = count.Value;  // InvalidOperationException!

// ✅ Safe: Check first
if (count.HasValue)
{
    int value = count.Value;
}

// ✅ Safe: Use null-coalescing
int value = count ?? 0;
```

### 5. == vs Equals for Strings
```csharp
string a = "hello";
string b = "hello";

// Both work for strings (value comparison)
if (a == b)         // true
if (a.Equals(b))    // true

// For other reference types, == compares references!
Person p1 = new Person { Name = "John" };
Person p2 = new Person { Name = "John" };
if (p1 == p2)  // false (different objects)
```

### 6. Forgetting Suffix for Literals
```csharp
// ❌ Wrong: float needs F suffix
float pi = 3.14;  // Error: Cannot convert double to float

// ✅ Correct
float pi = 3.14F;

// ❌ Wrong: decimal needs M suffix
decimal price = 19.99;  // Error: Cannot convert double to decimal

// ✅ Correct
decimal price = 19.99M;
```

### 7. Overflow Without Checking
```csharp
// ❌ Silent overflow
int max = int.MaxValue;
int overflow = max + 1;  // Wraps to int.MinValue

// ✅ Check with checked
checked
{
    int max = int.MaxValue;
    int overflow = max + 1;  // OverflowException
}
```

### 8. Confusing = and ==
```csharp
// ❌ Wrong: Assignment instead of comparison
if (x = 5)  // Error (C# prevents this)
{
}

// ✅ Correct
if (x == 5)
{
}
```

---

## Best Practices

### 1. Use Meaningful Names
```csharp
// ❌ Bad
int d;  // days? distance? data?

// ✅ Good
int daysUntilDeadline;
```

### 2. Use var Judiciously
```csharp
// ✅ Good: Type is obvious
var user = new User();
var numbers = new List<int>();

// ❌ Bad: Type is unclear
var data = GetData();  // What type is this?

// ✅ Good: Explicit when unclear
DataResult data = GetData();
```

### 3. Prefer Explicit Types for Primitives
```csharp
// ❌ OK but less clear
var count = 10;
var price = 19.99M;

// ✅ Better: Explicit
int count = 10;
decimal price = 19.99M;
```

### 4. Use const for True Constants
```csharp
// ✅ Good: Compile-time constant
public const int MaxRetries = 3;
public const string AppName = "MyApp";

// ✅ Good: Runtime value needs readonly
public readonly DateTime Created = DateTime.Now;
```

### 5. Use Nullable Reference Types (C# 8.0+)
```csharp
#nullable enable

// ✅ Clear: This should never be null
string name = GetName();

// ✅ Clear: This might be null
string? optionalName = GetOptionalName();
```

### 6. Use TryParse Instead of Parse
```csharp
// ❌ Risky: Throws exception
int value = int.Parse(userInput);

// ✅ Safe: Returns bool
if (int.TryParse(userInput, out int value))
{
    // Use value
}
```

### 7. Use Proper Numeric Types
```csharp
// ✅ Financial calculations: Use decimal
decimal totalPrice = subtotal + tax;

// ✅ Graphics/scientific: Use float/double
double distance = Math.Sqrt(x * x + y * y);

// ✅ Default integers: Use int
int count = items.Count;
```

### 8. Be Careful with Floating-Point Comparison
```csharp
// ❌ Wrong: Floating-point precision issues
if (0.1 + 0.2 == 0.3)  // false!
{
}

// ✅ Correct: Use epsilon for comparison
const double Epsilon = 0.0001;
if (Math.Abs((0.1 + 0.2) - 0.3) < Epsilon)
{
}

// ✅ Best: Use decimal for exact arithmetic
decimal result = 0.1M + 0.2M;
if (result == 0.3M)  // true
{
}
```

### 9. Initialize Variables
```csharp
// ❌ Bad: Uninitialized
int count;
count = count + 1;  // Error: Use of unassigned variable

// ✅ Good: Initialize
int count = 0;
count = count + 1;
```

### 10. Use String Interpolation (C# 6.0+)
```csharp
// ❌ Old way
string message = "Name: " + name + ", Age: " + age;

// ✅ Modern way
string message = $"Name: {name}, Age: {age}";
```

---

