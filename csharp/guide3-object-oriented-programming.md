# C# Object-Oriented Programming Quick Reference

---

## The Four Pillars of OOP

```

┌─────────────────────────────────────────────────────────────────────────────┐
│                    OBJECT-ORIENTED PROGRAMMING                              │
│                                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │ ENCAPSULATION  │  │  ABSTRACTION   │  │  INHERITANCE │  │ POLYMORPHISM │ │
│  │────────────────│  │────────────────│  │──────────────│  │──────────────│ │
│  │ Hide details   │  │ Hide complex   │  │  Reuse code  │  │ One interface│ │
│  │ Use properties │  │ Show essential │  │  Extend types│  │ Many forms   │ │
│  │ Access control │  │ Interfaces     │  │  Base/derived│  │ Overriding   │ │
│  └────────────────┘  └────────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘

```

---

## 1. Classes and Objects

### Class Definition Anatomy

```csharp
public class Person  // ← Class declaration
{
    // FIELDS (private data)
    private string _name;
    private int _age;
    
    // PROPERTIES (public interface to data)
    public string Name
    {
        get { return _name; }
        set { _name = value; }
    }
    
    public int Age { get; set; }  // Auto-property
    
    // CONSTRUCTOR (initialize object)
    public Person(string name, int age)
    {
        _name = name;
        _age = age;
    }
    
    // METHODS (behavior)
    public void Introduce()
    {
        Console.WriteLine($"Hi, I'm {_name}, {_age} years old.");
    }
    
    // EVENTS
    public event EventHandler AgeChanged;
}
```

**Class Components:**

- **Fields** - Private data storage
- **Properties** - Controlled access to data
- **Methods** - Actions/behavior
- **Constructors** - Object initialization
- **Events** - Notifications

### Object Creation (new keyword)

```csharp
// Create object
Person person = new Person("John", 25);

// C# 9.0+ Target-typed new
Person person = new("John", 25);

// Call methods
person.Introduce();  // Hi, I'm John, 25 years old.

// Access properties
string name = person.Name;
person.Age = 26;
```

### Reference Semantics

```csharp
// Classes are reference types
Person p1 = new Person("John", 25);
Person p2 = p1;  // Both point to SAME object

p2.Name = "Jane";
Console.WriteLine(p1.Name);  // "Jane" (same object!)

// Compare this to value types
int x = 10;
int y = x;  // y gets COPY of value
y = 20;
Console.WriteLine(x);  // 10 (different values)
```

**Memory Diagram:**
```
STACK              HEAP
┌──────┐          ┌──────────────┐
│ p1 ──┼────────> │ Person       │
│ p2 ──┼────────> │ { Name: "Jo" }│
└──────┘          │   Age: 25    │
                  └──────────────┘
```

### this Keyword

```csharp
public class Person
{
    private string name;
    
    public Person(string name)
    {
        this.name = name;  // 'this' refers to current instance
    }
    
    public void SetName(string name)
    {
        this.name = name;  // Disambiguate parameter vs field
    }
    
    public Person GetCopy()
    {
        return this;  // Return current instance
    }
}
```

### Object Initializers (C# 3.0+)

```csharp
// Traditional
Person p = new Person();
p.Name = "John";
p.Age = 25;

// Object initializer (cleaner)
Person p = new Person
{
    Name = "John",
    Age = 25
};

// With constructor
Person p = new Person("John", 25)
{
    Email = "john@example.com"  // Set additional properties
};
```

---

## 2. Structs

### Struct vs Class

| Feature | Struct | Class |
|---------|--------|-------|
| **Type** | Value type | Reference type |
| **Storage** | Stack (usually) | Heap |
| **Default** | Cannot be null | Can be null |
| **Inheritance** | Cannot inherit from struct | Can inherit from class |
| **Constructor** | Must initialize all fields | Can have parameterless constructor |
| **Performance** | Faster for small data | Slower (heap allocation) |
| **Copying** | Copies all data | Copies reference |
| **When to use** | Small, immutable data (< 16 bytes) | Complex objects, inheritance needed |

### Struct Declaration

```csharp
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
    
    // Constructor required to set all fields
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    public double DistanceFromOrigin()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
}

// Usage
Point p1 = new Point(10, 20);
Point p2 = p1;  // COPIES all data (not reference)
p2.X = 30;
Console.WriteLine(p1.X);  // 10 (different copies)
```

### Struct Limitations

```csharp
// ❌ Cannot inherit from struct
public struct Point { }
public struct Point3D : Point { }  // Error!

// ❌ Cannot have parameterless constructor (C# < 10)
public struct Point
{
    public Point() { }  // Error in C# < 10
}

// ❌ Cannot initialize fields at declaration (C# < 10)
public struct Point
{
    public int X = 0;  // Error in C# < 10
}

// ✅ But CAN implement interfaces
public struct Point : IComparable<Point>
{
    // Implementation
}
```

### readonly struct (C# 7.2+)

```csharp
// Immutable struct - all fields must be readonly
public readonly struct Point
{
    public int X { get; }
    public int Y { get; }
    
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
}
```

### ref struct (C# 7.2+)

```csharp
// Stack-only struct (cannot be boxed, stored on heap)
public ref struct Span<T>
{
    // Implementation
}

// Use case: High-performance scenarios, avoid heap allocation
```

### When to Use Struct vs Class

**Use Struct When:**

- ✅ Small data (< 16 bytes)
- ✅ Immutable (readonly)
- ✅ Value semantics needed
- ✅ Short-lived
- Examples: Point, Color, DateTime

**Use Class When:**

- ✅ Large objects
- ✅ Inheritance needed
- ✅ Reference semantics needed
- ✅ Long-lived
- Examples: Person, Order, Repository

---

## 3. Constructors

### Default Constructor

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    // Compiler generates default constructor if none defined
    // public Person() { }
}

// Usage
Person p = new Person();  // Name = null, Age = 0
```

### Parameterized Constructor

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}

// Usage
Person p = new Person("John", 25);
```

### Constructor Overloading

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Email { get; set; }
    
    // Default constructor
    public Person()
    {
        Name = "Unknown";
        Age = 0;
    }
    
    // Constructor with name
    public Person(string name)
    {
        Name = name;
        Age = 0;
    }
    
    // Constructor with name and age
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    // Constructor with all parameters
    public Person(string name, int age, string email)
    {
        Name = name;
        Age = age;
        Email = email;
    }
}
```

### Constructor Chaining (this)

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Email { get; set; }
    
    // Main constructor
    public Person(string name, int age, string email)
    {
        Name = name;
        Age = age;
        Email = email;
    }
    
    // Chains to main constructor
    public Person(string name, int age) : this(name, age, "unknown@example.com")
    {
    }
    
    // Chains to two-parameter constructor
    public Person(string name) : this(name, 0)
    {
    }
}
```

### Copy Constructor Pattern

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    // Regular constructor
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    // Copy constructor
    public Person(Person other)
    {
        Name = other.Name;
        Age = other.Age;
    }
}

// Usage
Person p1 = new Person("John", 25);
Person p2 = new Person(p1);  // Copy
```

### Static Constructor

```csharp
public class Configuration
{
    public static string ConnectionString { get; private set; }
    
    // Static constructor - runs ONCE when type is first used
    static Configuration()
    {
        // Load configuration
        ConnectionString = LoadFromFile();
    }
    
    private static string LoadFromFile()
    {
        return "Server=localhost;Database=mydb";
    }
}

// First access triggers static constructor
string conn = Configuration.ConnectionString;
```

**Static Constructor Rules:**

- No access modifiers
- No parameters
- Called automatically before any static members accessed
- Called only once
- Cannot be called directly

### Private Constructor (Singleton Pattern)

```csharp
public class Singleton
{
    // Private static instance
    private static Singleton _instance;
    
    // Private constructor prevents external instantiation
    private Singleton()
    {
    }
    
    // Public static method to get instance
    public static Singleton Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = new Singleton();
            }
            return _instance;
        }
    }
}

// Usage
Singleton s = Singleton.Instance;  // OK
// Singleton s = new Singleton();  // Error: Constructor is private
```

### Primary Constructors (C# 12.0+)

```csharp
// Traditional
public class Person
{
    public string Name { get; }
    public int Age { get; }
    
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}

// Primary constructor (shorter)
public class Person(string name, int age)
{
    public string Name { get; } = name;
    public int Age { get; } = age;
    
    public void Introduce()
    {
        Console.WriteLine($"I'm {name}, {age} years old.");
        // Can use constructor parameters directly
    }
}

// Usage (same)
Person p = new Person("John", 25);
```

---

## 4. Destructors (Finalizers)

### Syntax

```csharp
public class ResourceHolder
{
    // Constructor
    public ResourceHolder()
    {
        Console.WriteLine("Resource acquired");
    }
    
    // Destructor (finalizer)
    ~ResourceHolder()
    {
        Console.WriteLine("Resource released");
    }
}
```

### When Destructors Run

- Called by Garbage Collector (GC) when object is about to be collected
- Timing is **non-deterministic** (you don't know when)
- Object survives at least one more GC cycle

### Destructor vs Dispose

| Feature | Destructor (~) | Dispose() |
|---------|----------------|-----------|
| When runs | GC time (unpredictable) | Immediately when called |
| Guaranteed | No (may never run) | Yes (if called) |
| Use for | Unmanaged resources cleanup | Unmanaged resources cleanup |
| Performance | Slower (two GC cycles) | Faster |
| Recommended | Only if needed | Yes, implement IDisposable |

### Why to Avoid Destructors

```csharp
// ❌ Bad: Destructor delays collection
public class Bad
{
    ~Bad()
    {
        // Cleanup
    }
}

// ✅ Good: Use IDisposable instead
public class Good : IDisposable
{
    public void Dispose()
    {
        // Cleanup - deterministic
    }
}

using (var resource = new Good())
{
    // Use resource
}  // Dispose() called automatically
```

**Best Practice:** Use IDisposable pattern, not destructors. Only add destructor if you have unmanaged resources AND user might forget to call Dispose().

---

## 5. Properties

### Full Property Syntax

```csharp
public class Person
{
    private string _name;  // Backing field
    
    public string Name
    {
        get
        {
            return _name;
        }
        set
        {
            if (string.IsNullOrWhiteSpace(value))
                throw new ArgumentException("Name cannot be empty");
            _name = value;
        }
    }
}
```

### Auto-Implemented Properties (C# 3.0+)

```csharp
public class Person
{
    // Compiler creates backing field automatically
    public string Name { get; set; }
    public int Age { get; set; }
}
```

### Property Access Levels

```csharp
public class Person
{
    // Public get, private set
    public string Name { get; private set; }
    
    // Public get, protected set
    public int Age { get; protected set; }
    
    // Public get, internal set
    public string Email { get; internal set; }
}
```

### Read-Only Properties

```csharp
public class Person
{
    // Read-only: get only
    public string Name { get; }
    
    // Can only be set in constructor
    public Person(string name)
    {
        Name = name;
    }
}

// Or with backing field
public class Person
{
    private string _name = "John";
    
    public string Name
    {
        get { return _name; }
        // No setter = read-only
    }
}
```

### Computed Properties

```csharp
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    
    // Computed property (no backing field)
    public string FullName
    {
        get { return $"{FirstName} {LastName}"; }
    }
    
    // Or expression-bodied (C# 6.0+)
    public string FullName => $"{FirstName} {LastName}";
}
```

### Init-Only Setters (C# 9.0+)

```csharp
public class Person
{
    // Can only be set during initialization
    public string Name { get; init; }
    public int Age { get; init; }
}

// Usage
Person p = new Person
{
    Name = "John",   // OK during initialization
    Age = 25
};

p.Name = "Jane";  // ❌ Error: init-only property
```

### required Properties (C# 11.0+)

```csharp
public class Person
{
    // Must be set during initialization
    public required string Name { get; set; }
    public required int Age { get; set; }
    public string? Email { get; set; }  // Optional
}

// Usage
Person p1 = new Person
{
    Name = "John",  // ✅ Required
    Age = 25        // ✅ Required
};

Person p2 = new Person();  // ❌ Error: Name and Age required
```

### Property vs Field Comparison

| Feature | Field | Property |
|---------|-------|----------|
| Syntax | `public int age;` | `public int Age { get; set; }` |
| Encapsulation | Direct access | Controlled access |
| Validation | No | Yes (in setter) |
| Debugging | Can't set breakpoint | Can set breakpoint |
| Interface | Cannot be in interface | Can be in interface |
| Virtual | Cannot override | Can be virtual/override |
| Recommended | Private only | Public API |

**Best Practice:** Always use properties for public API, not fields.

### Expression-Bodied Properties (C# 6.0+)

```csharp
public class Person
{
    private string _name;
    
    // Get-only property
    public string Name => _name;
    
    // With setter (C# 7.0+)
    public string Name
    {
        get => _name;
        set => _name = value ?? "Unknown";
    }
}
```

---

## 6. Access Modifiers

### Access Levels

| Modifier | Accessible From |
|----------|----------------|
| `public` | Anywhere |
| `private` | Same class only |
| `protected` | Same class + derived classes |
| `internal` | Same assembly |
| `protected internal` | Same assembly OR derived classes |
| `private protected` | Same assembly AND derived classes (C# 7.2+) |

### Visual Accessibility Diagram

```
                  Same    Same      Derived    Derived
                  Class   Assembly  (Same Asm) (Other Asm)
public            ✓       ✓         ✓          ✓
private           ✓       ✗         ✗          ✗
protected         ✓       ✗         ✓          ✓
internal          ✓       ✓         ✓          ✗
protected internal✓       ✓         ✓          ✓
private protected ✓       ✓         ✓          ✗
```

### Examples

```csharp
public class Person
{
    public string Name;           // Accessible everywhere
    private int _age;             // Only in Person class
    protected string _email;      // Person and derived classes
    internal string _phone;       // Same assembly
    protected internal string _address;  // Same assembly OR derived
    private protected string _ssn;       // Same assembly AND derived
}

// Derived class in SAME assembly
public class Employee : Person
{
    public void Test()
    {
        Name = "John";      // ✓ public
        // _age = 25;       // ✗ private
        _email = "...";     // ✓ protected
        _phone = "...";     // ✓ internal
        _address = "...";   // ✓ protected internal
        _ssn = "...";       // ✓ private protected
    }
}

// Derived class in OTHER assembly
public class Manager : Person
{
    public void Test()
    {
        Name = "Jane";      // ✓ public
        // _age = 30;       // ✗ private
        _email = "...";     // ✓ protected
        // _phone = "...";  // ✗ internal
        _address = "...";   // ✓ protected internal (derived)
        // _ssn = "...";    // ✗ private protected (other assembly)
    }
}
```

### Default Access Levels

```csharp
// Class default: internal
class Person { }  // internal

// Member default: private
public class Person
{
    string name;  // private
    void Method() { }  // private
}

// Interface members: public (implicit)
interface IPerson
{
    void Method();  // public (no modifier needed)
}
```

---

## 7. Methods

### Method Signature

```csharp
public int Calculate(int x, int y)
{
    return x + y;
}
//│   │    │        │
//│   │    │        └─ Parameters
//│   │    └────────── Method name
//│   └─────────────── Return type
//└─────────────────── Access modifier
```

### Method Overloading

**Same name, different parameters**

```csharp
public class Calculator
{
    // Different number of parameters
    public int Add(int a, int b)
    {
        return a + b;
    }
    
    public int Add(int a, int b, int c)
    {
        return a + b + c;
    }
    
    // Different parameter types
    public double Add(double a, double b)
    {
        return a + b;
    }
    
    // Different parameter order
    public void Print(int value, string format)
    {
    }
    
    public void Print(string format, int value)
    {
    }
}

// Usage
Calculator calc = new Calculator();
calc.Add(1, 2);        // Calls Add(int, int)
calc.Add(1, 2, 3);     // Calls Add(int, int, int)
calc.Add(1.5, 2.5);    // Calls Add(double, double)
```

**Overloading Rules:**

- Must differ in: number of parameters, types of parameters, or parameter order
- Return type alone is NOT enough
- Parameter names don't matter

```csharp
// ❌ Invalid overload (only return type differs)
public int GetValue() { }
public string GetValue() { }  // Error!

// ❌ Invalid overload (only parameter names differ)
public void Method(int x) { }
public void Method(int y) { }  // Error!

// ✅ Valid overload (parameter types differ)
public void Method(int x) { }
public void Method(string x) { }  // OK
```

### Method Parameters

#### Value Parameters (Default)
```csharp
public void Increment(int value)
{
    value++;  // Only affects local copy
}

int x = 10;
Increment(x);
Console.WriteLine(x);  // 10 (unchanged)
```

#### ref Parameters
```csharp
public void Increment(ref int value)
{
    value++;  // Modifies original variable
}

int x = 10;
Increment(ref x);  // Must use 'ref' keyword
Console.WriteLine(x);  // 11 (changed)
```

**ref Rules:**

- Must initialize before passing
- Caller must use `ref` keyword
- Method can read and write

#### out Parameters
```csharp
public bool TryParse(string input, out int result)
{
    if (int.TryParse(input, out result))
    {
        return true;
    }
    result = 0;
    return false;
}

// Usage
if (TryParse("123", out int value))
{
    Console.WriteLine(value);  // 123
}

// C# 7.0+: Inline declaration
if (TryParse("123", out var value))
{
    Console.WriteLine(value);
}
```

**out Rules:**

- No need to initialize before passing
- Must be assigned in method before returning
- Caller must use `out` keyword

#### in Parameters (C# 7.2+)
```csharp
public void Process(in LargeStruct data)
{
    // data is read-only reference (no copy)
    // data.Field = 10;  // Error: cannot modify
}

LargeStruct s = new LargeStruct();
Process(in s);  // Passes by reference, no copy
```

**in Rules:**

- Read-only reference
- Avoids copying large structs
- Caller can optionally use `in` keyword

#### params Parameters
```csharp
public int Sum(params int[] numbers)
{
    int total = 0;
    foreach (int n in numbers)
    {
        total += n;
    }
    return total;
}

// Usage
Sum(1, 2, 3);           // Calls Sum(new int[] {1,2,3})
Sum(1, 2, 3, 4, 5);     // Variable number of arguments
int[] arr = {1, 2, 3};
Sum(arr);               // Or pass array directly
```

### Optional Parameters (C# 4.0+)

```csharp
public void Log(string message, string level = "INFO", bool timestamp = true)
{
    if (timestamp)
    {
        Console.WriteLine($"[{DateTime.Now}] {level}: {message}");
    }
    else
    {
        Console.WriteLine($"{level}: {message}");
    }
}

// Usage
Log("Error occurred");                      // Uses defaults
Log("Warning", "WARN");                     // Override level
Log("Debug info", timestamp: false);        // Named argument
Log("Critical", "ERROR", false);            // All parameters
```

**Rules:**

- Optional parameters must be last
- Must have default value (compile-time constant)

### Named Arguments (C# 4.0+)

```csharp
public void CreateUser(string name, int age, string email, bool isActive)
{
}

// Traditional (positional)
CreateUser("John", 25, "john@example.com", true);

// Named arguments (any order)
CreateUser(
    email: "john@example.com",
    name: "John",
    isActive: true,
    age: 25
);

// Mix positional and named
CreateUser("John", 25, email: "john@example.com", isActive: true);
```

### Expression-Bodied Methods (C# 6.0+)

```csharp
// Traditional
public int Add(int a, int b)
{
    return a + b;
}

// Expression-bodied (single expression)
public int Add(int a, int b) => a + b;

// void method
public void Print(string message) => Console.WriteLine(message);

// Multiple statements (C# 7.0+)
public int Calculate(int x) => x > 0 ? x * 2 : x;
```

### Local Functions (C# 7.0+)

```csharp
public int Factorial(int n)
{
    // Local function (only accessible within Factorial)
    int Calculate(int x)
    {
        if (x <= 1) return 1;
        return x * Calculate(x - 1);
    }
    
    return Calculate(n);
}

// Can access outer variables
public void Process(List<int> numbers)
{
    int threshold = 10;
    
    // Local function can access 'threshold'
    void ProcessItem(int value)
    {
        if (value > threshold)
        {
            Console.WriteLine(value);
        }
    }
    
    foreach (int num in numbers)
    {
        ProcessItem(num);
    }
}
```

### Static Local Functions (C# 8.0+)

```csharp
public int Calculate(int x, int y)
{
    // Static local function - cannot access outer variables
    static int Add(int a, int b)
    {
        // Cannot access x or y from outer method
        return a + b;
    }
    
    return Add(x, y);
}
```

**When to Use:**

- Prevent accidental capture of outer variables
- Better performance (no closure allocation)

---

## 8. The Four Pillars of OOP

### Pillar 1: Encapsulation

**Definition:** Hide implementation details, expose only what's necessary.

```csharp
// ❌ Bad: Public field (no control)
public class BankAccount
{
    public decimal balance;  // Anyone can modify
}

// ✅ Good: Private field with property (controlled access)
public class BankAccount
{
    private decimal _balance;
    
    public decimal Balance
    {
        get { return _balance; }
        private set  // Only class can modify
        {
            if (value < 0)
                throw new ArgumentException("Balance cannot be negative");
            _balance = value;
        }
    }
    
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");
        Balance += amount;
    }
    
    public void Withdraw(decimal amount)
    {
        if (amount > Balance)
            throw new InvalidOperationException("Insufficient funds");
        Balance -= amount;
    }
}
```

**Benefits:**

- Control access to data
- Validate inputs
- Change implementation without breaking external code
- Hide complexity

### Pillar 2: Abstraction

**Definition:** Show only essential features, hide complex implementation.

#### Abstract Classes

```csharp
public abstract class Animal  // Cannot be instantiated
{
    // Abstract method - must be implemented by derived class
    public abstract void MakeSound();
    
    // Abstract property
    public abstract string Name { get; set; }
    
    // Concrete method (has implementation)
    public void Sleep()
    {
        Console.WriteLine("Sleeping...");
    }
}

public class Dog : Animal
{
    // Must implement abstract members
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
    
    public override string Name { get; set; }
}

// Usage
// Animal a = new Animal();  // ❌ Error: Cannot instantiate abstract class
Animal dog = new Dog();  // ✅ OK
dog.MakeSound();  // Woof!
```

#### Interfaces

```csharp
public interface IDrawable
{
    // Interface members are implicitly public and abstract
    void Draw();
    void Resize(int width, int height);
    string Color { get; set; }
}

public class Circle : IDrawable
{
    public string Color { get; set; }
    
    public void Draw()
    {
        Console.WriteLine($"Drawing {Color} circle");
    }
    
    public void Resize(int width, int height)
    {
        Console.WriteLine($"Resizing to {width}x{height}");
    }
}
```

#### Multiple Interface Implementation

```csharp
public interface IDrawable
{
    void Draw();
}

public interface IMovable
{
    void Move(int x, int y);
}

// Class can implement multiple interfaces
public class Sprite : IDrawable, IMovable
{
    public void Draw()
    {
        Console.WriteLine("Drawing sprite");
    }
    
    public void Move(int x, int y)
    {
        Console.WriteLine($"Moving to ({x}, {y})");
    }
}
```

#### Explicit Interface Implementation

```csharp
public interface IPrintable
{
    void Print();
}

public interface ILoggable
{
    void Print();  // Name conflict!
}

public class Document : IPrintable, ILoggable
{
    // Explicit implementation
    void IPrintable.Print()
    {
        Console.WriteLine("Printing document");
    }
    
    void ILoggable.Print()
    {
        Console.WriteLine("Logging document");
    }
}

// Usage
Document doc = new Document();
// doc.Print();  // ❌ Error: Ambiguous

IPrintable printable = doc;
printable.Print();  // Printing document

ILoggable loggable = doc;
loggable.Print();  // Logging document
```

#### Default Interface Methods (C# 8.0+)

```csharp
public interface ILogger
{
    void Log(string message);
    
    // Default implementation
    void LogError(string message)
    {
        Log($"ERROR: {message}");
    }
}

public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
    
    // LogError is inherited (optional to override)
}
```

#### Static Abstract Members (C# 11.0+)

```csharp
public interface INumber<T>
{
    static abstract T Zero { get; }
    static abstract T operator +(T left, T right);
}

public struct Integer : INumber<Integer>
{
    public int Value { get; set; }
    
    public static Integer Zero => new Integer { Value = 0 };
    
    public static Integer operator +(Integer left, Integer right)
    {
        return new Integer { Value = left.Value + right.Value };
    }
}
```

#### Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| **Can have** | Abstract + concrete members | Abstract members (+ default C# 8.0+) |
| **Fields** | Yes | No |
| **Constructors** | Yes | No |
| **Access modifiers** | Any | Public only (implicitly) |
| **Inheritance** | Single (one base class) | Multiple (many interfaces) |
| **When to use** | Shared code, common base | Contract, multiple inheritance |

**Decision Guide:**

- Use **abstract class** when: Classes share common code, need fields/constructors
- Use **interface** when: Define contract, need multiple inheritance, no shared code

### Pillar 3: Inheritance

**Definition:** Create new classes from existing ones, reusing code.

```csharp
// Base class
public class Animal
{
    public string Name { get; set; }
    
    public void Eat()
    {
        Console.WriteLine($"{Name} is eating");
    }
}

// Derived class inherits from Animal
public class Dog : Animal
{
    public void Bark()
    {
        Console.WriteLine($"{Name} says Woof!");
    }
}

// Usage
Dog dog = new Dog();
dog.Name = "Buddy";
dog.Eat();   // Inherited from Animal
dog.Bark();  // Defined in Dog
```

#### Single Inheritance

```csharp
// C# supports single inheritance for classes
public class Animal { }
public class Mammal : Animal { }
public class Dog : Mammal { }  // ✅ OK

// ❌ Cannot inherit from multiple classes
public class Dog : Mammal, Reptile { }  // Error!
```

#### Multiple Inheritance (Interfaces)

```csharp
// But can implement multiple interfaces
public interface ISwimmable { void Swim(); }
public interface IFlyable { void Fly(); }

public class Duck : Animal, ISwimmable, IFlyable
{
    public void Swim()
    {
        Console.WriteLine("Duck is swimming");
    }
    
    public void Fly()
    {
        Console.WriteLine("Duck is flying");
    }
}
```

#### base Keyword

```csharp
public class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Some sound");
    }
}

public class Dog : Animal
{
    public override void MakeSound()
    {
        base.MakeSound();  // Call base class method
        Console.WriteLine("Woof!");
    }
}

// Output:
// Some sound
// Woof!
```

#### Calling Base Constructors

```csharp
public class Animal
{
    public string Name { get; set; }
    
    public Animal(string name)
    {
        Name = name;
    }
}

public class Dog : Animal
{
    public string Breed { get; set; }
    
    // Must call base constructor
    public Dog(string name, string breed) : base(name)
    {
        Breed = breed;
    }
}

// Usage
Dog dog = new Dog("Buddy", "Golden Retriever");
```

#### sealed Classes (Prevent Inheritance)

```csharp
// Cannot be inherited
public sealed class FinalClass
{
    public void Method()
    {
    }
}

// ❌ Error: Cannot inherit from sealed class
public class Derived : FinalClass { }
```

**When to use sealed:**

- Prevent further inheritance
- Security reasons
- Performance (compiler optimizations)

#### Inheritance Hierarchy Example

```
        Animal
        ├─ Mammal
        │  ├─ Dog
        │  ├─ Cat
        │  └─ Elephant
        ├─ Bird
        │  ├─ Eagle
        │  └─ Penguin
        └─ Reptile
           ├─ Snake
           └─ Lizard
```

### Pillar 4: Polymorphism

**Definition:** One interface, many implementations.

#### Compile-Time Polymorphism (Method Overloading)

```csharp
public class Calculator
{
    // Same method name, different parameters
    public int Add(int a, int b)
    {
        return a + b;
    }
    
    public double Add(double a, double b)
    {
        return a + b;
    }
    
    public int Add(int a, int b, int c)
    {
        return a + b + c;
    }
}

// Compiler selects correct method based on arguments
Calculator calc = new Calculator();
int result1 = calc.Add(1, 2);        // Calls Add(int, int)
double result2 = calc.Add(1.5, 2.5); // Calls Add(double, double)
```

#### Runtime Polymorphism (Method Overriding)

```csharp
public class Animal
{
    // virtual = can be overridden
    public virtual void MakeSound()
    {
        Console.WriteLine("Some sound");
    }
}

public class Dog : Animal
{
    // override = replaces base implementation
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}

public class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Meow!");
    }
}

// Polymorphism in action
Animal[] animals = new Animal[]
{
    new Dog(),
    new Cat(),
    new Animal()
};

foreach (Animal animal in animals)
{
    animal.MakeSound();  // Calls correct implementation at runtime
}
// Output:
// Woof!
// Meow!
// Some sound
```

#### virtual, override, sealed

```csharp
public class Base
{
    // virtual = can be overridden
    public virtual void Method1()
    {
        Console.WriteLine("Base.Method1");
    }
    
    // sealed = cannot be overridden (must also be override)
    public sealed override void Method2()
    {
        Console.WriteLine("Base.Method2");
    }
}

public class Derived : Base
{
    // override = replaces base implementation
    public override void Method1()
    {
        Console.WriteLine("Derived.Method1");
    }
    
    // ❌ Error: Cannot override sealed method
    // public override void Method2() { }
}
```

#### Method Hiding with new

```csharp
public class Base
{
    public virtual void Method()
    {
        Console.WriteLine("Base.Method");
    }
}

public class Derived : Base
{
    // 'new' hides base method (not polymorphic!)
    public new void Method()
    {
        Console.WriteLine("Derived.Method");
    }
}

// Behavior
Derived d = new Derived();
d.Method();  // Derived.Method

Base b = new Derived();
b.Method();  // Base.Method (not polymorphic!)

// vs override
public class Derived2 : Base
{
    public override void Method()
    {
        Console.WriteLine("Derived2.Method");
    }
}

Base b2 = new Derived2();
b2.Method();  // Derived2.Method (polymorphic!)
```

**virtual / override / new Summary:**

| Keyword | Purpose | Polymorphic? |
|---------|---------|--------------|
| `virtual` | Mark method as overridable | Yes |
| `override` | Replace base implementation | Yes |
| `new` | Hide base method | No |
| `sealed override` | Override but prevent further overrides | Yes |

---

## 9. Static Members

### Static Fields

```csharp
public class Counter
{
    // Instance field (each object has its own)
    private int _instanceCount;
    
    // Static field (shared across all instances)
    private static int _totalCount = 0;
    
    public Counter()
    {
        _instanceCount++;
        _totalCount++;
    }
    
    public static int TotalCount => _totalCount;
}

// Usage
Counter c1 = new Counter();
Counter c2 = new Counter();
Counter c3 = new Counter();

Console.WriteLine(Counter.TotalCount);  // 3 (shared)
```

### Static Methods

```csharp
public class MathHelper
{
    // Static method - no instance needed
    public static int Add(int a, int b)
    {
        return a + b;
    }
    
    // Cannot access instance members
    public static void Invalid()
    {
        // Error: Cannot access instance member
        // this.SomeMethod();
    }
}

// Usage - no object needed
int result = MathHelper.Add(5, 3);
```

### Static Constructors

```csharp
public class Configuration
{
    public static string ConnectionString { get; private set; }
    public static int MaxConnections { get; private set; }
    
    // Static constructor - runs once when type first used
    static Configuration()
    {
        ConnectionString = LoadConnectionString();
        MaxConnections = 100;
    }
    
    private static string LoadConnectionString()
    {
        // Load from file
        return "Server=localhost;Database=mydb";
    }
}

// First access triggers static constructor
string conn = Configuration.ConnectionString;
```

### Static Classes

```csharp
// Static class - cannot be instantiated
public static class MathHelper
{
    // All members must be static
    public static int Add(int a, int b)
    {
        return a + b;
    }
    
    public static double PI = 3.14159;
    
    // ❌ Cannot have instance members
    // public void InstanceMethod() { }
}

// Usage
int sum = MathHelper.Add(5, 3);
double pi = MathHelper.PI;

// ❌ Cannot create instance
// var helper = new MathHelper();  // Error!
```

**When to Use Static Class:**

- Utility/helper methods (Math, Console, Convert)
- Extension methods (must be in static class)
- Constants and global state
- No state needed

**Static vs Instance:**

| Feature | Instance | Static |
|---------|----------|--------|
| Creation | `new ClassName()` | N/A |
| Access | Through instance | Through class name |
| this | Can use | Cannot use |
| State | Per object | Shared |
| Inheritance | Yes | No (static class) |

---

## 10. Partial Classes (C# 2.0+)

### Split Class Definition

```csharp
// File1.cs
public partial class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

// File2.cs
public partial class Person
{
    public string FullName => $"{FirstName} {LastName}";
    
    public void Introduce()
    {
        Console.WriteLine($"Hi, I'm {FullName}");
    }
}

// Compiler combines both into single class
Person p = new Person
{
    FirstName = "John",
    LastName = "Doe"
};
p.Introduce();  // Hi, I'm John Doe
```

### Use Cases

1. **Code Generation**
```csharp
// Generated by tool (DO NOT EDIT)
public partial class CustomerData
{
    // Auto-generated properties
}

// Your custom code
public partial class CustomerData
{
    // Your custom methods
}
```

2. **Large Classes**
```csharp
// Person.Properties.cs
public partial class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    // ... many properties
}

// Person.Methods.cs
public partial class Person
{
    public void Method1() { }
    public void Method2() { }
    // ... many methods
}
```

### Partial Methods (C# 3.0+)

```csharp
public partial class Person
{
    // Declaration (can be in generated code)
    partial void OnNameChanged();
    
    private string _name;
    public string Name
    {
        get => _name;
        set
        {
            _name = value;
            OnNameChanged();  // Calls if implemented
        }
    }
}

public partial class Person
{
    // Implementation (optional)
    partial void OnNameChanged()
    {
        Console.WriteLine($"Name changed to {Name}");
    }
}
```

**Rules:**

- Must return void
- Cannot have out parameters
- Implicitly private
- Implementation is optional (if not implemented, call is removed by compiler)

### Partial Properties (C# 13.0+)

```csharp
public partial class Person
{
    // Declaration
    public partial string Name { get; set; }
}

public partial class Person
{
    // Implementation
    private string _name;
    public partial string Name
    {
        get => _name;
        set => _name = value;
    }
}
```

---

## 11. Extension Methods (C# 3.0+)

### Syntax

```csharp
// Must be in static class
public static class StringExtensions
{
    // 'this' on first parameter makes it extension method
    public static string Reverse(this string str)
    {
        char[] chars = str.ToCharArray();
        Array.Reverse(chars);
        return new string(chars);
    }
    
    public static int WordCount(this string str)
    {
        return str.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
    }
}

// Usage - appears as instance method
string text = "Hello World";
string reversed = text.Reverse();      // "dlroW olleH"
int count = text.WordCount();          // 2
```

### Rules and Limitations

```csharp
// ✅ Must be in static class
public static class Extensions
{
    // ✅ Must be static method
    public static void Method(this Type obj)
    {
    }
}

// ❌ Cannot access private members
public static class PersonExtensions
{
    public static void Print(this Person person)
    {
        // Cannot access person's private fields
        // Console.WriteLine(person._name);  // Error!
        Console.WriteLine(person.Name);  // OK (public)
    }
}

// ❌ Instance methods take priority
public class MyClass
{
    public void Method()
    {
        Console.WriteLine("Instance");
    }
}

public static class Extensions
{
    public static void Method(this MyClass obj)
    {
        Console.WriteLine("Extension");
    }
}

MyClass m = new MyClass();
m.Method();  // "Instance" (extension ignored)
```

### Common Use Cases

```csharp
// LINQ extension methods
public static class Enumerable
{
    public static IEnumerable<T> Where<T>(
        this IEnumerable<T> source,
        Func<T, bool> predicate)
    {
        // Implementation
    }
}

// Usage
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
var evens = numbers.Where(n => n % 2 == 0);  // Extension method

// Custom extensions
public static class IntExtensions
{
    public static bool IsEven(this int number)
    {
        return number % 2 == 0;
    }
    
    public static bool IsOdd(this int number)
    {
        return number % 2 != 0;
    }
}

// Usage
int x = 5;
if (x.IsOdd())  // Reads naturally
{
    Console.WriteLine("Odd number");
}
```

---

## 12. Nested Classes

### Inner Classes

```csharp
public class Outer
{
    private int _outerField = 10;
    
    // Nested class
    public class Inner
    {
        public void PrintOuter()
        {
            // ❌ Cannot directly access outer instance members
            // Console.WriteLine(_outerField);  // Error!
            
            // Must have reference to outer instance
            Outer outer = new Outer();
            Console.WriteLine(outer._outerField);  // OK if made accessible
        }
    }
}

// Usage
Outer.Inner inner = new Outer.Inner();
inner.PrintOuter();
```

### Access to Outer Class Members

```csharp
public class LinkedList
{
    private Node _head;
    
    // Private nested class
    private class Node
    {
        public int Value { get; set; }
        public Node Next { get; set; }
    }
    
    public void Add(int value)
    {
        Node newNode = new Node { Value = value };  // Can access private nested class
        
        if (_head == null)
        {
            _head = newNode;
        }
        else
        {
            Node current = _head;
            while (current.Next != null)
            {
                current = current.Next;
            }
            current.Next = newNode;
        }
    }
}
```

### When to Use Nested Classes

**Use When:**

- ✅ Class is only used by outer class
- ✅ Want to hide implementation details
- ✅ Logically belongs to outer class
- Examples: Node in LinkedList, Enumerator in collection

**Don't Use When:**

- ❌ Class might be useful elsewhere
- ❌ Makes code harder to read
- ❌ Just for organization (use namespace instead)

---

## 13. Anonymous Types (C# 3.0+)

### Syntax

```csharp
// Create anonymous type with 'new' and object initializer
var person = new
{
    Name = "John",
    Age = 25,
    Email = "john@example.com"
};

// Properties are read-only
Console.WriteLine(person.Name);  // John
// person.Name = "Jane";  // ❌ Error: read-only

// Type is inferred by compiler
// Type: <>f__AnonymousType0<string, int, string>
```

### Use with LINQ

```csharp
var people = new[]
{
    new { Name = "John", Age = 25 },
    new { Name = "Jane", Age = 30 },
    new { Name = "Bob", Age = 35 }
};

// Project to anonymous type
var query = from p in people
            where p.Age > 25
            select new
            {
                p.Name,
                IsAdult = p.Age >= 18,
                YearsUntilRetirement = 65 - p.Age
            };

foreach (var item in query)
{
    Console.WriteLine($"{item.Name}: {item.YearsUntilRetirement} years until retirement");
}
```

### Limitations

```csharp
// ❌ Cannot be used as method return type (no explicit type)
public var GetPerson()  // Error!
{
    return new { Name = "John", Age = 25 };
}

// ✅ Must use object or dynamic
public object GetPerson()
{
    return new { Name = "John", Age = 25 };
}

// ❌ Cannot be used as field type
public class MyClass
{
    private var _person;  // Error!
}

// ✅ Limited to method scope
public void Method()
{
    var person = new { Name = "John" };  // OK
}
```

**When to Use:**

- ✅ LINQ projections
- ✅ Temporary data structures within a method
- ✅ Grouping related data for short-term use

**Don't Use:**

- ❌ Public API (return types, parameters)
- ❌ Long-term data storage
- ❌ When you need methods or constructors

---

## 14. Records (C# 9.0+)

### Record Classes (Reference Types)

```csharp
// Record declaration
public record Person
{
    public string Name { get; init; }
    public int Age { get; init; }
}

// Usage
Person person = new Person
{
    Name = "John",
    Age = 25
};

// Immutable (init-only properties)
// person.Age = 26;  // ❌ Error
```

### Positional Records

```csharp
// Shorter syntax (primary constructor)
public record Person(string Name, int Age);

// Equivalent to:
public record Person
{
    public string Name { get; init; }
    public int Age { get; init; }
    
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    // Auto-generated: Equals, GetHashCode, ToString, Deconstruct
}

// Usage
Person person = new Person("John", 25);
Console.WriteLine(person);  // Person { Name = John, Age = 25 }

// Deconstruction
var (name, age) = person;
```

### with Expressions (Non-Destructive Mutation)

```csharp
Person person1 = new Person("John", 25);

// Create new record with changed property
Person person2 = person1 with { Age = 26 };

Console.WriteLine(person1);  // Person { Name = John, Age = 25 }
Console.WriteLine(person2);  // Person { Name = John, Age = 26 }
```

### Value-Based Equality

```csharp
// Records use value-based equality
Person p1 = new Person("John", 25);
Person p2 = new Person("John", 25);

Console.WriteLine(p1 == p2);  // true (same values)
Console.WriteLine(p1.Equals(p2));  // true

// Compare to classes (reference equality)
public class PersonClass
{
    public string Name { get; set; }
    public int Age { get; set; }
}

PersonClass c1 = new PersonClass { Name = "John", Age = 25 };
PersonClass c2 = new PersonClass { Name = "John", Age = 25 };

Console.WriteLine(c1 == c2);  // false (different references)
```

### Record Structs (C# 10.0+)

```csharp
// Record struct (value type)
public record struct Point(int X, int Y);

// Usage
Point p1 = new Point(10, 20);
Point p2 = new Point(10, 20);

Console.WriteLine(p1 == p2);  // true (value equality)

// Mutable record struct
public record struct MutablePoint(int X, int Y)
{
    public int X { get; set; } = X;
    public int Y { get; set; } = Y;
}
```

### Record vs Class vs Struct

| Feature | Class | Struct | Record (class) | Record struct |
|---------|-------|--------|----------------|---------------|
| **Type** | Reference | Value | Reference | Value |
| **Storage** | Heap | Stack | Heap | Stack |
| **Equality** | Reference | Value | Value | Value |
| **Immutability** | No (default) | No | Yes (default) | Yes (default) |
| **with** | No | No | Yes | Yes |
| **Inheritance** | Yes | No | Yes (records only) | No |
| **ToString()** | Type name | Type name | All properties | All properties |
| **When to use** | Mutable objects | Small, value semantics | Immutable data | Small, immutable value data |

### When to Use Records

**Use Records When:**

- ✅ Immutable data (DTOs, data transfer objects)
- ✅ Value-based equality needed
- ✅ POCO (Plain Old CLR Objects)
- ✅ API models, configuration

**Examples:**
```csharp
// DTO for API
public record UserDto(int Id, string Name, string Email);

// Configuration
public record AppSettings(string ConnectionString, int MaxRetries);

// Domain model (immutable)
public record Order(int Id, decimal Total, OrderStatus Status);
```

---

## 15. Operator Overloading

### Overloadable Operators

**Can Overload:**

- Unary: `+`, `-`, `!`, `~`, `++`, `--`, `true`, `false`
- Binary: `+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `<<`, `>>`, `==`, `!=`, `<`, `>`, `<=`, `>=`

**Cannot Overload:**

- `=`, `.`, `?:`, `??`, `->`, `=>`, `is`, `as`, `new`, `typeof`, `sizeof`, `checked`, `unchecked`

### Syntax

```csharp
public struct Complex
{
    public double Real { get; set; }
    public double Imaginary { get; set; }
    
    // operator keyword
    public static Complex operator +(Complex a, Complex b)
    {
        return new Complex
        {
            Real = a.Real + b.Real,
            Imaginary = a.Imaginary + b.Imaginary
        };
    }
    
    public static Complex operator -(Complex a, Complex b)
    {
        return new Complex
        {
            Real = a.Real - b.Real,
            Imaginary = a.Imaginary - b.Imaginary
        };
    }
    
    public static Complex operator *(Complex a, Complex b)
    {
        return new Complex
        {
            Real = a.Real * b.Real - a.Imaginary * b.Imaginary,
            Imaginary = a.Real * b.Imaginary + a.Imaginary * b.Real
        };
    }
}

// Usage
Complex c1 = new Complex { Real = 1, Imaginary = 2 };
Complex c2 = new Complex { Real = 3, Imaginary = 4 };
Complex sum = c1 + c2;  // Calls operator +
```

### Comparison Operators (Must Override in Pairs)

```csharp
public struct Vector
{
    public double X { get; set; }
    public double Y { get; set; }
    
    // == and != must be overloaded together
    public static bool operator ==(Vector a, Vector b)
    {
        return a.X == b.X && a.Y == b.Y;
    }
    
    public static bool operator !=(Vector a, Vector b)
    {
        return !(a == b);
    }
    
    // < and > must be overloaded together
    public static bool operator <(Vector a, Vector b)
    {
        return a.Magnitude() < b.Magnitude();
    }
    
    public static bool operator >(Vector a, Vector b)
    {
        return a.Magnitude() > b.Magnitude();
    }
    
    // <= and >= must be overloaded together
    public static bool operator <=(Vector a, Vector b)
    {
        return a.Magnitude() <= b.Magnitude();
    }
    
    public static bool operator >=(Vector a, Vector b)
    {
        return a.Magnitude() >= b.Magnitude();
    }
    
    private double Magnitude()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
    
    // Should also override Equals and GetHashCode
    public override bool Equals(object obj)
    {
        if (obj is Vector v)
            return this == v;
        return false;
    }
    
    public override int GetHashCode()
    {
        return HashCode.Combine(X, Y);
    }
}
```

### Best Practices

```csharp
// ✅ Good: Intuitive behavior
public static Money operator +(Money a, Money b)
{
    return new Money(a.Amount + b.Amount);
}

// ❌ Bad: Surprising behavior
public static Money operator -(Money a, Money b)
{
    return new Money(a.Amount * b.Amount);  // Confusing!
}

// ✅ Good: Consistent with related operations
// If you overload +, consider overloading +=

// ✅ Good: Override Equals and GetHashCode when overloading == and !=
```

---

## 16. Conversion Operators

### implicit operator (Automatic Conversion)

```csharp
public struct Celsius
{
    public double Degrees { get; set; }
    
    // Implicit conversion from double to Celsius
    public static implicit operator Celsius(double degrees)
    {
        return new Celsius { Degrees = degrees };
    }
    
    // Implicit conversion from Celsius to double
    public static implicit operator double(Celsius celsius)
    {
        return celsius.Degrees;
    }
}

// Usage - no cast needed
Celsius temp = 25.5;  // Automatically converts double to Celsius
double degrees = temp;  // Automatically converts Celsius to double
```

### explicit operator (Manual Conversion)

```csharp
public struct Fahrenheit
{
    public double Degrees { get; set; }
    
    // Explicit conversion from Celsius to Fahrenheit
    public static explicit operator Fahrenheit(Celsius celsius)
    {
        return new Fahrenheit
        {
            Degrees = celsius.Degrees * 9 / 5 + 32
        };
    }
}

// Usage - requires cast
Celsius c = new Celsius { Degrees = 25 };
Fahrenheit f = (Fahrenheit)c;  // Must cast explicitly
```

### When to Use Each

**Use `implicit` When:**

- ✅ Conversion is always safe (no data loss)
- ✅ Conversion is intuitive
- ✅ No exceptions will be thrown
- Examples: int → long, float → double

**Use `explicit` When:**

- ✅ Conversion might lose data
- ✅ Conversion might throw exception
- ✅ Conversion is not obvious
- Examples: double → int, Celsius → Fahrenheit

```csharp
// ❌ Bad: implicit for lossy conversion
public static implicit operator int(double value)
{
    return (int)value;  // Loses decimal part!
}

// ✅ Good: explicit for lossy conversion
public static explicit operator int(double value)
{
    return (int)value;
}
```

---

## 17. Indexers

### Syntax

```csharp
public class StringCollection
{
    private List<string> _items = new List<string>();
    
    // Indexer - allows collection[index] syntax
    public string this[int index]
    {
        get
        {
            if (index < 0 || index >= _items.Count)
                throw new IndexOutOfRangeException();
            return _items[index];
        }
        set
        {
            if (index < 0 || index >= _items.Count)
                throw new IndexOutOfRangeException();
            _items[index] = value;
        }
    }
    
    public void Add(string item)
    {
        _items.Add(item);
    }
}

// Usage - looks like array access
StringCollection collection = new StringCollection();
collection.Add("Hello");
collection.Add("World");

string first = collection[0];  // "Hello"
collection[1] = "C#";          // Modify
```

### Multiple Indexers

```csharp
public class Grid
{
    private int[,] _data = new int[10, 10];
    
    // Indexer with two parameters
    public int this[int row, int col]
    {
        get { return _data[row, col]; }
        set { _data[row, col] = value; }
    }
}

// Usage
Grid grid = new Grid();
grid[5, 3] = 42;
int value = grid[5, 3];
```

### Overloading Indexers

```csharp
public class SmartDictionary
{
    private Dictionary<string, string> _data = new Dictionary<string, string>();
    
    // Indexer by string key
    public string this[string key]
    {
        get { return _data[key]; }
        set { _data[key] = value; }
    }
    
    // Indexer by int index
    public string this[int index]
    {
        get
        {
            return _data.ElementAt(index).Value;
        }
    }
}

// Usage
SmartDictionary dict = new SmartDictionary();
dict["name"] = "John";           // String indexer
string first = dict[0];          // Int indexer
```

### Read-Only Indexer

```csharp
public class ReadOnlyList
{
    private List<int> _items = new List<int> { 1, 2, 3, 4, 5 };
    
    // Read-only indexer (no setter)
    public int this[int index]
    {
        get { return _items[index]; }
    }
}

// Usage
ReadOnlyList list = new ReadOnlyList();
int value = list[0];  // OK
// list[0] = 10;      // ❌ Error: no setter
```

---

## Common Pitfalls

### 1. Forgetting virtual/override

```csharp
// ❌ Bad: Method not virtual, cannot override
public class Base
{
    public void Method()  // Not virtual
    {
    }
}

public class Derived : Base
{
    public override void Method()  // ❌ Error: nothing to override
    {
    }
}

// ✅ Good: Mark as virtual
public class Base
{
    public virtual void Method()
    {
    }
}

public class Derived : Base
{
    public override void Method()  // ✅ OK
    {
    }
}
```

### 2. Using new Instead of override

```csharp
public class Base
{
    public virtual void Method()
    {
        Console.WriteLine("Base");
    }
}

public class Derived : Base
{
    // ⚠️ Hides base method, not polymorphic
    public new void Method()
    {
        Console.WriteLine("Derived");
    }
}

Base b = new Derived();
b.Method();  // "Base" (not polymorphic!)

// ✅ Use override for polymorphism
public class Derived : Base
{
    public override void Method()
    {
        Console.WriteLine("Derived");
    }
}

Base b = new Derived();
b.Method();  // "Derived" (polymorphic!)
```

### 3. Not Calling base Constructor

```csharp
public class Base
{
    public Base(string name)
    {
        // Initialize
    }
}

// ❌ Error: No parameterless constructor in base
public class Derived : Base
{
    public Derived()
    {
    }
}

// ✅ Good: Call base constructor
public class Derived : Base
{
    public Derived() : base("default")
    {
    }
}
```

### 4. Struct Mutability Issues

```csharp
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
}

Point[] points = new Point[2];
points[0] = new Point { X = 10, Y = 20 };

// ❌ Doesn't work as expected
points[0].X = 30;  // Modifies temporary copy!

// ✅ Better: Use readonly struct or full replacement
readonly struct Point
{
    public int X { get; }
    public int Y { get; }
}

// Or replace entire struct
points[0] = new Point { X = 30, Y = points[0].Y };
```

### 5. Property vs Field in Interfaces

```csharp
// ❌ Error: Cannot have fields in interface
public interface IPerson
{
    string name;  // Error!
}

// ✅ Good: Use properties
public interface IPerson
{
    string Name { get; set; }
}
```

### 6. Forgetting to Override Equals and GetHashCode

```csharp
// ❌ Bad: Overload == but not Equals
public class Person
{
    public string Name { get; set; }
    
    public static bool operator ==(Person a, Person b)
    {
        return a.Name == b.Name;
    }
    
    public static bool operator !=(Person a, Person b)
    {
        return !(a == b);
    }
}

// ✅ Good: Override Equals and GetHashCode too
public class Person
{
    public string Name { get; set; }
    
    public static bool operator ==(Person a, Person b)
    {
        if (ReferenceEquals(a, b)) return true;
        if (a is null || b is null) return false;
        return a.Name == b.Name;
    }
    
    public static bool operator !=(Person a, Person b)
    {
        return !(a == b);
    }
    
    public override bool Equals(object obj)
    {
        return obj is Person person && this == person;
    }
    
    public override int GetHashCode()
    {
        return Name?.GetHashCode() ?? 0;
    }
}
```

---

## Best Practices

### 1. Encapsulation

```csharp
// ❌ Bad: Public fields
public class Person
{
    public string name;
    public int age;
}

// ✅ Good: Properties with validation
public class Person
{
    private string _name;
    private int _age;
    
    public string Name
    {
        get => _name;
        set => _name = string.IsNullOrWhiteSpace(value) 
            ? throw new ArgumentException("Name required") 
            : value;
    }
    
    public int Age
    {
        get => _age;
        set => _age = value >= 0 
            ? value 
            : throw new ArgumentException("Age must be positive");
    }
}
```

### 2. Use Auto-Properties When Possible

```csharp
// ❌ Verbose
public class Person
{
    private string _name;
    
    public string Name
    {
        get { return _name; }
        set { _name = value; }
    }
}

// ✅ Concise
public class Person
{
    public string Name { get; set; }
}
```

### 3. Prefer Composition Over Inheritance

```csharp
// ❌ Deep inheritance hierarchy
public class Vehicle { }
public class Car : Vehicle { }
public class ElectricCar : Car { }
public class TeslaModelS : ElectricCar { }  // Too deep!

// ✅ Use composition
public class Car
{
    public Engine Engine { get; set; }
    public Battery Battery { get; set; }
}
```

### 4. Use Interfaces for Contracts

```csharp
// ✅ Good: Depend on interface, not implementation
public class OrderProcessor
{
    private readonly IPaymentService _paymentService;
    
    public OrderProcessor(IPaymentService paymentService)
    {
        _paymentService = paymentService;
    }
}

// Can swap implementations
IPaymentService payment = new StripePaymentService();
// or
IPaymentService payment = new PayPalPaymentService();
```

### 5. Mark Classes sealed When Not Designed for Inheritance

```csharp
// ✅ Prevent unintended inheritance
public sealed class ConfigurationManager
{
    // Implementation
}
```

### 6. Use Records for Immutable Data

```csharp
// ✅ Good: Immutable DTO
public record UserDto(int Id, string Name, string Email);

// ❌ Avoid: Mutable DTO
public class UserDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}
```

### 7. Initialize Collections in Constructor or Inline

```csharp
// ✅ Good: Prevent null reference
public class Order
{
    public List<OrderItem> Items { get; set; } = new List<OrderItem>();
    
    // Or in constructor
    public Order()
    {
        Items = new List<OrderItem>();
    }
}
```

### 8. Use Expression-Bodied Members for Simple Properties

```csharp
// ✅ Concise
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    
    public string FullName => $"{FirstName} {LastName}";
}
```

### 9. Validate in Constructors

```csharp
// ✅ Good: Fail fast
public class Person
{
    public string Name { get; }
    public int Age { get; }
    
    public Person(string name, int age)
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException("Name required", nameof(name));
        if (age < 0)
            throw new ArgumentException("Age must be positive", nameof(age));
        
        Name = name;
        Age = age;
    }
}
```

### 10. Use readonly for Immutable Fields

```csharp
// ✅ Communicate intent
public class Configuration
{
    private readonly string _connectionString;
    
    public Configuration(string connectionString)
    {
        _connectionString = connectionString;
    }
}
```

---

## Quick Reference: Decision Trees

### Should I Use Class, Struct, or Record?

```
Need inheritance? 
    YES → Class or Record (class)
    NO → Continue

Immutable data (DTOs, data transfer)?
    YES → Record (class or struct)
    NO → Continue

Small data (< 16 bytes) with value semantics?
    YES → Struct (or record struct)
    NO → Class

Need reference semantics?
    YES → Class
    NO → Struct
```

### Should I Use Abstract Class or Interface?

```
Need to share code (implementation)?
    YES → Abstract class
    NO → Continue

Need multiple inheritance?
    YES → Interface
    NO → Either (prefer interface for flexibility)

Need fields or constructors?
    YES → Abstract class
    NO → Interface
```

### Should Method Be virtual?

```
Will derived classes need to override?
    YES → Make virtual
    NO → Don't make virtual (keep sealed)

Already overriding?
    YES → Use override (automatically virtual)
    NO → Consider if extension point needed
```

---

**Guide Complete!** This comprehensive OOP guide covers all essential concepts from basic classes to advanced features like records and operator overloading. Ready for printing and study! 📘
