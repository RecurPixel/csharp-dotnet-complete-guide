# Quick Reference & Cheat Sheets

**Purpose:** Fast lookup for syntax and common patterns while coding

**Tip:** Bookmark this page! You'll reference it constantly.

---

## C# Syntax Quick Reference

### **Variables & Data Types**

```csharp
// Integer types
byte age = 25;              // 0 to 255
short year = 2024;          // -32,768 to 32,767
int count = 1000;           // -2 billion to 2 billion
long bigNum = 1000000L;     // Very large numbers

// Floating point
float price = 19.99f;       // 7 digits precision
double pi = 3.14159;        // 15-16 digits precision
decimal money = 99.99m;     // 28-29 digits (for money!)

// Other types
bool isActive = true;       // true or false
char letter = 'A';          // Single character
string name = "John";       // Text

// Nullable types
int? nullableInt = null;    // Can be null
string? nullableString = null; // C# 8.0+

// Type inference
var number = 42;            // Compiler figures out type
var text = "Hello";         // Still strongly typed!
```

### **String Operations**

```csharp
// Declaration
string name = "John Doe";

// Concatenation
string fullName = firstName + " " + lastName;

// Interpolation (preferred)
string message = $"Hello, {name}!";
string calc = $"Sum: {5 + 3}";  // Expressions allowed

// Common methods
name.Length                  // 8
name.ToUpper()              // "JOHN DOE"
name.ToLower()              // "john doe"
name.Trim()                 // Remove whitespace
name.Substring(0, 4)        // "John"
name.Replace("Doe", "Smith") // "John Smith"
name.Contains("John")       // true
name.StartsWith("John")     // true
name.Split(' ')             // ["John", "Doe"]

// Comparison
name == "John Doe"          // true (value equality)
name.Equals("john doe", StringComparison.OrdinalIgnoreCase) // true

// Multi-line (C# 11+)
string multiline = """
    This is
    multi-line text
    """;
```

### **Operators**

```csharp
// Arithmetic
+  -  *  /  %              // Add, subtract, multiply, divide, modulo
++  --                      // Increment, decrement

// Comparison
==  !=  >  <  >=  <=       // Equals, not equals, etc.

// Logical
&&  ||  !                   // AND, OR, NOT

// Assignment
=  +=  -=  *=  /=  %=      // Assign and combine

// Null coalescing
??                          // If null, use default
string name = input ?? "Unknown";

??=                         // Assign if null
name ??= "Default";

// Null conditional
?.                          // Safe navigation
int? length = name?.Length;

// Ternary
condition ? trueValue : falseValue
int age = isAdult ? 18 : 0;
```

### **Control Flow**

```csharp
// If-Else
if (condition)
{
    // code
}
else if (otherCondition)
{
    // code
}
else
{
    // code
}

// Switch (traditional)
switch (value)
{
    case 1:
        Console.WriteLine("One");
        break;
    case 2:
        Console.WriteLine("Two");
        break;
    default:
        Console.WriteLine("Other");
        break;
}

// Switch expression (C# 8.0+)
string result = value switch
{
    1 => "One",
    2 => "Two",
    _ => "Other"
};

// For loop
for (int i = 0; i < 10; i++)
{
    Console.WriteLine(i);
}

// While loop
while (condition)
{
    // code
}

// Do-While loop
do
{
    // code
} while (condition);

// Foreach loop
foreach (var item in collection)
{
    Console.WriteLine(item);
}
```

### **Arrays**

```csharp
// Declaration
int[] numbers = new int[5];          // Fixed size
int[] nums = { 1, 2, 3, 4, 5 };     // Initialize with values
int[] scores = new int[] { 90, 85, 92 };

// Access
int first = numbers[0];              // Get first
numbers[0] = 100;                    // Set first

// Properties
numbers.Length                       // 5

// Common operations
Array.Sort(numbers);                 // Sort in place
Array.Reverse(numbers);              // Reverse
int index = Array.IndexOf(numbers, 3); // Find index
```

### **Collections**

```csharp
// List<T> (dynamic array)
List<int> numbers = new List<int>();
numbers.Add(5);                      // Add element
numbers.Remove(5);                   // Remove element
numbers.Count                        // Number of elements
numbers[0]                          // Access by index

// Dictionary<TKey, TValue> (key-value pairs)
Dictionary<string, int> ages = new Dictionary<string, int>();
ages["John"] = 25;                   // Add/Update
ages.Add("Jane", 30);               // Add
int age = ages["John"];             // Get
ages.Remove("John");                // Remove
ages.ContainsKey("John")            // Check if exists

// HashSet<T> (unique values)
HashSet<int> unique = new HashSet<int>();
unique.Add(5);                      // Add
unique.Contains(5)                  // Check membership
unique.Count                        // Size

// Queue<T> (FIFO)
Queue<string> queue = new Queue<string>();
queue.Enqueue("first");             // Add to end
string item = queue.Dequeue();      // Remove from front
string peek = queue.Peek();         // Look without removing

// Stack<T> (LIFO)
Stack<int> stack = new Stack<int>();
stack.Push(5);                      // Add to top
int top = stack.Pop();              // Remove from top
int peekTop = stack.Peek();         // Look without removing
```

### **Methods**

```csharp
// Basic method
void PrintMessage()
{
    Console.WriteLine("Hello!");
}

// Method with parameters
void Greet(string name)
{
    Console.WriteLine($"Hello, {name}!");
}

// Method with return value
int Add(int a, int b)
{
    return a + b;
}

// Multiple return values (tuple)
(int sum, int product) Calculate(int a, int b)
{
    return (a + b, a * b);
}
// Usage: var (sum, prod) = Calculate(5, 3);

// Optional parameters
void Print(string text, bool uppercase = false)
{
    Console.WriteLine(uppercase ? text.ToUpper() : text);
}

// params keyword (variable arguments)
int Sum(params int[] numbers)
{
    return numbers.Sum();
}
// Usage: Sum(1, 2, 3, 4, 5);

// Expression-bodied method
int Square(int x) => x * x;
void Log(string msg) => Console.WriteLine(msg);
```

### **Classes**

```csharp
// Basic class
public class Person
{
    // Fields (private)
    private string name;
    
    // Properties (public)
    public string Name { get; set; }
    public int Age { get; set; }
    
    // Auto-implemented property
    public string Email { get; set; }
    
    // Read-only property
    public string FullName => $"{FirstName} {LastName}";
    
    // Constructor
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    // Method
    public void Introduce()
    {
        Console.WriteLine($"I'm {Name}, age {Age}");
    }
}

// Usage
Person person = new Person("John", 25);
person.Introduce();
```

### **Inheritance**

```csharp
// Base class
public class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Some sound");
    }
}

// Derived class
public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}

// Usage
Animal dog = new Dog();
dog.MakeSound();  // "Woof!"
```

### **Interfaces**

```csharp
// Define interface
public interface IDrawable
{
    void Draw();
}

// Implement interface
public class Circle : IDrawable
{
    public void Draw()
    {
        Console.WriteLine("Drawing circle");
    }
}
```

### **Exception Handling**

```csharp
// Basic try-catch
try
{
    int result = 10 / 0;
}
catch (DivideByZeroException ex)
{
    Console.WriteLine("Cannot divide by zero!");
}

// Multiple catch blocks
try
{
    // risky code
}
catch (FileNotFoundException ex)
{
    // handle file not found
}
catch (Exception ex)
{
    // handle any other exception
}
finally
{
    // always runs (cleanup code)
}

// Throw exception
throw new ArgumentException("Invalid argument");
```

---

## LINQ Quick Reference

### **Basic LINQ Syntax**

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Where (filter)
var evens = numbers.Where(n => n % 2 == 0);
// Result: [2, 4, 6, 8, 10]

// Select (map/transform)
var squared = numbers.Select(n => n * n);
// Result: [1, 4, 9, 16, 25, ...]

// OrderBy (sort ascending)
var sorted = numbers.OrderBy(n => n);

// OrderByDescending (sort descending)
var sortedDesc = numbers.OrderByDescending(n => n);

// First (get first element)
int first = numbers.First();               // Throws if empty
int firstOrDefault = numbers.FirstOrDefault(); // Returns 0 if empty

// Take (get first n elements)
var firstThree = numbers.Take(3);          // [1, 2, 3]

// Skip (skip first n elements)
var skipTwo = numbers.Skip(2);             // [3, 4, 5, ...]

// Any (check if any match)
bool hasEvens = numbers.Any(n => n % 2 == 0);  // true

// All (check if all match)
bool allPositive = numbers.All(n => n > 0);    // true

// Count (with condition)
int evenCount = numbers.Count(n => n % 2 == 0); // 5

// Sum, Average, Min, Max
int sum = numbers.Sum();
double avg = numbers.Average();
int min = numbers.Min();
int max = numbers.Max();

// Distinct (unique values)
var unique = numbers.Distinct();

// GroupBy (group by key)
var grouped = numbers.GroupBy(n => n % 2 == 0 ? "Even" : "Odd");

// ToList, ToArray (convert to collection)
List<int> list = numbers.Where(n => n > 5).ToList();
int[] array = numbers.Where(n => n > 5).ToArray();
```

### **Chaining LINQ Operations**

```csharp
var result = numbers
    .Where(n => n > 5)           // Filter
    .Select(n => n * 2)          // Transform
    .OrderByDescending(n => n)   // Sort
    .Take(3)                     // Get top 3
    .ToList();                   // Convert to list
```

---

## Async/Await Patterns

```csharp
// Async method
async Task<string> FetchDataAsync()
{
    // Simulate async work
    await Task.Delay(1000);
    return "Data";
}

// Calling async method
async Task MainAsync()
{
    string data = await FetchDataAsync();
    Console.WriteLine(data);
}

// Multiple async operations (parallel)
async Task ParallelOperationsAsync()
{
    Task<string> task1 = FetchDataAsync();
    Task<string> task2 = FetchDataAsync();
    
    // Wait for both
    await Task.WhenAll(task1, task2);
    
    string result1 = task1.Result;
    string result2 = task2.Result;
}

// Error handling in async
async Task SafeOperationAsync()
{
    try
    {
        await FetchDataAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
    }
}
```

---

## File I/O Quick Reference

```csharp
// Read entire file
string content = File.ReadAllText("file.txt");
string[] lines = File.ReadAllLines("file.txt");

// Write entire file
File.WriteAllText("file.txt", "content");
File.WriteAllLines("file.txt", new[] { "line1", "line2" });

// Append to file
File.AppendAllText("file.txt", "more content");

// Check if exists
bool exists = File.Exists("file.txt");

// Delete file
File.Delete("file.txt");

// StreamReader (line by line)
using (StreamReader reader = new StreamReader("file.txt"))
{
    string line;
    while ((line = reader.ReadLine()) != null)
    {
        Console.WriteLine(line);
    }
}

// StreamWriter
using (StreamWriter writer = new StreamWriter("file.txt"))
{
    writer.WriteLine("Line 1");
    writer.WriteLine("Line 2");
}
```

---

## JSON Serialization (System.Text.Json)

```csharp
using System.Text.Json;

// Object to JSON
Person person = new Person { Name = "John", Age = 25 };
string json = JsonSerializer.Serialize(person);
// Result: {"Name":"John","Age":25}

// JSON to Object
string json = """{"Name":"John","Age":25}""";
Person person = JsonSerializer.Deserialize<Person>(json);

// Pretty print
var options = new JsonSerializerOptions { WriteIndented = true };
string json = JsonSerializer.Serialize(person, options);

// To/From file
string json = JsonSerializer.Serialize(people);
File.WriteAllText("data.json", json);

string json = File.ReadAllText("data.json");
List<Person> people = JsonSerializer.Deserialize<List<Person>>(json);
```

---

## Common Patterns Cheat Sheet

### **Null Checking**

```csharp
// Traditional
if (name != null)
{
    Console.WriteLine(name);
}

// Null coalescing
string displayName = name ?? "Unknown";

// Null conditional
int? length = name?.Length;

// Throw if null (C# 11+)
ArgumentNullException.ThrowIfNull(name);
```

### **Collection Initialization**

```csharp
// List
var numbers = new List<int> { 1, 2, 3, 4, 5 };

// Dictionary
var ages = new Dictionary<string, int>
{
    ["John"] = 25,
    ["Jane"] = 30
};

// Array
int[] nums = { 1, 2, 3, 4, 5 };
```

### **String Formatting**

```csharp
// String interpolation
string msg = $"Hello, {name}!";

// Format specifiers
string price = $"Price: {value:C}";      // Currency: $19.99
string percent = $"Rate: {rate:P}";      // Percent: 75.00%
string decimal = $"Pi: {pi:F2}";         // Fixed: 3.14
string padded = $"ID: {id:D5}";          // Zero-pad: 00042
```

### **Loops vs LINQ**

```csharp
// Traditional loop
List<int> evens = new List<int>();
foreach (int n in numbers)
{
    if (n % 2 == 0)
        evens.Add(n);
}

// LINQ (preferred)
var evens = numbers.Where(n => n % 2 == 0).ToList();
```

### **Value vs Reference Types**

```csharp
// Value types (stack)
int x = 5;
struct Point { public int X, Y; }

// Reference types (heap)
string name = "John";
class Person { }
List<int> numbers = new List<int>();
```

---

## Time Complexity Reference

| Operation | List | Dictionary | HashSet | Array | LinkedList |
|-----------|------|------------|---------|-------|------------|
| Add | O(1)* | O(1)* | O(1)* | N/A | O(1) |
| Insert | O(n) | N/A | N/A | N/A | O(1)** |
| Remove | O(n) | O(1)* | O(1)* | N/A | O(1)** |
| Search | O(n) | O(1)* | O(1)* | O(n) | O(n) |
| Access by index | O(1) | N/A | N/A | O(1) | O(n) |

\* Amortized, ** If you have the node

---

## Error Messages Decoder

### **Common Compiler Errors:**

**"CS0103: The name '...' does not exist"**
- Variable/method not declared
- Check spelling and scope

**"CS1002: ; expected"**
- Missing semicolon
- Check previous line

**"CS0029: Cannot implicitly convert type"**
- Type mismatch
- Use explicit cast or conversion

**"CS0161: Not all code paths return a value"**
- Method missing return statement
- Add return or throw exception

**"CS1061: Type does not contain a definition for '...'"**
- Method/property doesn't exist
- Check object type and spelling

### **Common Runtime Errors:**

**NullReferenceException**
- Accessing null object
- Use null checks (?. or ??)

**IndexOutOfRangeException**
- Array/list index too large
- Check bounds

**DivideByZeroException**
- Dividing by zero
- Check divisor before dividing

**InvalidOperationException**
- Operation not valid for current state
- Check preconditions

**ArgumentException**
- Invalid argument passed
- Validate inputs

---

## Keyboard Shortcuts Reference

### **Visual Studio:**

| Action | Shortcut |
|--------|----------|
| Run | F5 |
| Run without debugging | Ctrl+F5 |
| Build | Ctrl+Shift+B |
| Format document | Ctrl+K, Ctrl+D |
| Comment/Uncomment | Ctrl+K, Ctrl+C / Ctrl+K, Ctrl+U |
| Find | Ctrl+F |
| Replace | Ctrl+H |
| Go to definition | F12 |
| IntelliSense | Ctrl+Space |
| Quick actions | Ctrl+. |

### **VS Code:**

| Action | Shortcut |
|--------|----------|
| Run | F5 |
| Open terminal | Ctrl+` |
| Format document | Shift+Alt+F |
| Comment/Uncomment | Ctrl+/ |
| Find | Ctrl+F |
| Replace | Ctrl+H |
| Go to definition | F12 |
| IntelliSense | Ctrl+Space |
| Command palette | Ctrl+Shift+P |

---

## When to Use What?

### **Collections:**

- **List<T>**: Default choice, need order and index access
- **Dictionary<TKey, TValue>**: Fast lookups by key
- **HashSet<T>**: Unique values, fast membership check
- **Queue<T>**: FIFO processing
- **Stack<T>**: LIFO processing, undo functionality
- **LinkedList<T>**: Frequent insertions/deletions in middle

### **Loops:**

- **for**: Known number of iterations, need index
- **foreach**: Iterate over collection
- **while**: Unknown iterations, condition-based
- **do-while**: Must execute at least once

### **String Operations:**

- **Concatenation (+)**: Few strings, simple cases
- **String interpolation ($"")**: Embed variables (preferred)
- **StringBuilder**: Many concatenations in loop
- **string.Format()**: Legacy code (use interpolation instead)

---

## Naming Conventions

```csharp
// Classes, Methods, Properties: PascalCase
public class UserAccount { }
public void CalculateTotal() { }
public string FirstName { get; set; }

// Local variables, parameters: camelCase
int itemCount = 0;
void Process(string userName) { }

// Constants: PascalCase
public const int MaxRetries = 3;

// Private fields: _camelCase (with underscore)
private string _userName;

// Interfaces: IPascalCase (I prefix)
public interface IRepository { }
```

---

## Print This Page!

This reference contains everything you'll use daily. Keep it handy while coding!

**Pro Tip:** As you learn new concepts, add your own notes to this reference.

**Next:** Start coding and refer back here whenever you forget syntax!

