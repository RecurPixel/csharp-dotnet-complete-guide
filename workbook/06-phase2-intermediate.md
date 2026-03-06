# PHASE: INTERMEDIATE PROBLEMS

**Total**: 66 problems

---

### Problem 12: Number Guessing Game ⭐⭐
**Concepts**: While Loop, Random Numbers, User Interaction

**What You'll Learn**:
- Using `Random` class
- Implementing game logic with loops
- Providing user feedback

**Requirements**:
1. Generate random number (1-100)
2. Let user guess
3. Give hints: "Too high" / "Too low"
4. Count attempts
5. Congratulate on correct guess

**Bonus Challenges**:
- Add difficulty levels (range size)
- Limit number of attempts
- High score tracking
- Play again option

---

---

### Problem 13: Sum and Average Calculator ⭐⭐
**Concepts**: Arrays, Loops, Aggregation

**What You'll Learn**:
- Using arrays to store multiple values
- Iterating through arrays
- Calculating aggregates

**Requirements**:
1. Ask how many numbers to enter
2. Store in array
3. Calculate and display:
   - Sum
   - Average
   - Maximum
   - Minimum

**Bonus Challenges**:
- Find median value
- Calculate standard deviation
- Display as a bar chart (ASCII)

---

---

### Problem 14: Prime Number Checker ⭐⭐
**Concepts**: Loops, Optimization, Boolean Logic

**What You'll Learn**:
- Implementing efficient algorithms
- Optimizing loop conditions
- Understanding mathematical properties

**Requirements**:
1. Check if a number is prime
2. Only check divisors up to √n
3. Handle edge cases (1, 2, negative numbers)

**Bonus Challenges**:
- Find all primes in a range
- Implement Sieve of Eratosthenes
- Count primes up to N

---

---

### Problem 16: Fibonacci Series Generator ⭐⭐
**Concepts**: Loop Patterns, Sequence Logic

**What You'll Learn**:
- Generating sequences
- Using multiple variables to track state
- Understanding Fibonacci properties

**Requirements**:
1. Generate first N Fibonacci numbers
2. Start with 0, 1
3. Display series: 0, 1, 1, 2, 3, 5, 8...

**Bonus Challenges**:
- Find Nth Fibonacci number only
- Use recursion with memoization
- Find Fibonacci numbers up to a limit

---

---

### Problem 19: Armstrong Number Checker ⭐⭐
**Concepts**: Loops, Power Function, Digit Processing

**What You'll Learn**:
- Using Math.Pow()
- Counting digits
- Complex number properties

**Requirements**:
1. Check if number is Armstrong
2. Armstrong: Sum of cubes of digits equals number
3. Example: 153 = 1³ + 5³ + 3³ = 1 + 125 + 27 = 153

**Bonus Challenges**:
- Find all Armstrong numbers in range
- Generalize for n-digit numbers
- Find next Armstrong number

---

---

### Problem 20: Perfect Number Validator ⭐⭐
**Concepts**: Nested Loops, Divisor Logic

**What You'll Learn**:
- Finding divisors efficiently
- Understanding number theory
- Optimizing with early termination

**Requirements**:
1. Check if number is perfect
2. Perfect: sum of proper divisors equals number
3. Example: 6 = 1 + 2 + 3

**Bonus Challenges**:
- Find all perfect numbers up to N
- Classify as abundant or deficient
- Find all friendly pairs

---

## Section 1.3: Pattern Printing
**Focus**: Nested loops, spacing logic, visual output

---

### Problem 22: Pyramid Pattern ⭐⭐
**Concepts**: Loop Control, Spacing, Centering

**What You'll Learn**:
- Calculating spaces for centering
- Symmetric pattern building
- Mathematical relationships in patterns

**Requirements**:
Print centered pyramid:
```
    *
   ***
  *****
 *******
*********
```

**Bonus Challenges**:
- Inverted pyramid
- Diamond pattern
- Number pyramid

---

---

### Problem 23: Diamond Pattern ⭐⭐
**Concepts**: Complex Nesting, Symmetry

**What You'll Learn**:
- Combining ascending and descending patterns
- Managing multiple loop variables
- Creating symmetric designs

**Requirements**:
Print diamond:
```
    *
   ***
  *****
 *******
  *****
   ***
    *
```

**Bonus Challenges**:
- Hollow diamond
- Number diamond
- Variable size diamond

---

---

### Problem 24: Number Pyramid ⭐⭐
**Concepts**: Variable Patterns, Formatting

**What You'll Learn**:
- Printing variable content
- Maintaining alignment with varying widths
- Number sequences in patterns

**Requirements**:
```
1
12
123
1234
12345
```

**Bonus Challenges**:
- Reverse number pyramid
- Floyd's triangle
- Pascal's triangle

---

---

### Problem 25: Floyd's Triangle ⭐⭐
**Concepts**: Sequence Generation, Display

**What You'll Learn**:
- Maintaining state across loops
- Generating sequences
- Formatting numerical output

**Requirements**:
```
1
2 3
4 5 6
7 8 9 10
11 12 13 14 15
```

**Bonus Challenges**:
- Floyd's triangle with custom starting number
- Binary Floyd's triangle
- Alphabetic Floyd's triangle

---

## Section 1.4: Methods & Code Organization
**Focus**: Breaking code into reusable methods, parameters, return values

---

### Problem 26: Palindrome Checker ⭐⭐
**Concepts**: Methods, String Manipulation

**What You'll Learn**:
- Creating methods with return values
- String reversal techniques
- Case-insensitive comparison

**Requirements**:
Create method `bool IsPalindrome(string input)`:
1. Remove spaces and convert to lowercase
2. Check if string reads same backwards
3. Return true/false

**Bonus Challenges**:
- Ignore punctuation
- Check numeric palindromes
- Find longest palindrome in text

---

---

### Problem 28: String Analyzer ⭐⭐
**Concepts**: Multiple Methods, Analysis

**What You'll Learn**:
- Breaking complex tasks into methods
- Character analysis
- Organizing related methods

**Requirements**:
Create methods to analyze a string:
1. `CountVowels(string s)`
2. `CountConsonants(string s)`
3. `CountDigits(string s)`
4. `CountSpecialChars(string s)`

**Bonus Challenges**:
- Create a full report method
- Find character frequency
- Identify most common character

---

---

### Problem 29: Simple Menu System ⭐⭐
**Concepts**: Methods, Switch, Loop Control

**What You'll Learn**:
- Organizing code into menu-driven interface
- Using methods for each operation
- Controlling program flow with loops

**Requirements**:
Create menu with options:
1. Add two numbers
2. Subtract
3. Multiply
4. Divide
5. Exit
Each operation should be a separate method.

**Bonus Challenges**:
- Add scientific calculator functions
- Save calculation history
- Support chained operations

---

---

### Problem 30: Student Grades Summary ⭐⭐
**Concepts**: Arrays, Multiple Methods, Analysis

**What You'll Learn**:
- Processing arrays with methods
- Statistical calculations
- Organizing data analysis code

**Requirements**:
Create methods:
1. `GetHighest(int[] marks)`
2. `GetLowest(int[] marks)`
3. `GetAverage(int[] marks)`
4. `CountPassing(int[] marks, int passMark)`
5. `AssignGrade(double average)`

**Bonus Challenges**:
- Calculate standard deviation
- Rank students
- Generate report card

---

---

### Problem 31: GCD and LCM Calculator ⭐⭐
**Concepts**: Algorithm Implementation, Recursion

**What You'll Learn**:
- Implementing Euclidean algorithm
- Recursive vs iterative approaches
- Mathematical relationships (GCD × LCM = a × b)

**Requirements**:
1. `int GCD(int a, int b)` using Euclidean algorithm
2. `int LCM(int a, int b)` using GCD
3. Handle edge cases (0, negative numbers)

**Bonus Challenges**:
- GCD/LCM for multiple numbers
- Implement extended Euclidean algorithm
- Optimize for large numbers

---

---

### Problem 32: Power Calculator (Custom) ⭐⭐
**Concepts**: Recursion Introduction, Base Cases

**What You'll Learn**:
- Understanding recursion
- Base case and recursive case
- Comparing recursion with loops

**Requirements**:
Implement `double Power(int base, int exponent)`:
1. Handle positive exponents
2. Handle zero exponent
3. Handle negative exponents

**Bonus Challenges**:
- Optimize with fast exponentiation (O(log n))
- Compare recursive vs iterative performance
- Handle very large exponents

---

---

### Problem 33: Array Statistics Module ⭐⭐
**Concepts**: Parameters, Return Values, Multiple Calculations

**What You'll Learn**:
- Passing arrays to methods
- Returning multiple values (using out parameters or tuples)
- Organizing statistical functions

**Requirements**:
Create comprehensive stats module:
1. Mean, Median, Mode
2. Range, Variance, Standard Deviation
3. Quartiles

**Bonus Challenges**:
- Implement using tuples for multiple returns
- Add percentile calculation
- Create histogram visualization

---

---

### Problem 34: Text Formatter Tool ⭐⭐
**Concepts**: String Methods, Validation, Formatting

**What You'll Learn**:
- String manipulation techniques
- Text formatting standards
- Input validation

**Requirements**:
Create methods:
1. `ToTitleCase(string s)` - Capitalize Each Word
2. `ToCamelCase(string s)` - camelCase
3. `ToSnakeCase(string s)` - snake_case
4. `ToKebabCase(string s)` - kebab-case

**Bonus Challenges**:
- Add PascalCase
- Validate naming conventions
- Convert between all formats

---

---

### Problem 38: Bank Account Manager ⭐⭐
**Concepts**: Encapsulation, Properties, Validation

**What You'll Learn**:
- Private fields with public properties
- Data validation in setters
- Business logic in methods

**Requirements**:
Create `BankAccount` class:
```csharp
class BankAccount
{
    private string accountNumber;
    private string accountHolder;
    private decimal balance;
    
    // Properties with validation
    public decimal Balance 
    { 
        get { return balance; }
        private set 
        { 
            if (value < 0)
                throw new Exception("Balance cannot be negative");
            balance = value;
        }
    }
    
    // Methods
    public void Deposit(decimal amount) { }
    public bool Withdraw(decimal amount) { }
    public void DisplayStatement() { }
}
```

**Bonus Challenges**:
- Add transaction history (List of transactions)
- Implement minimum balance requirement
- Add interest calculation
- Create multiple account types

---

---

### Problem 39: Employee Management ⭐⭐
**Concepts**: Private Fields, Validation, Business Logic

**Requirements**:
Create `Employee` class with validation:
- Properties: `Name`, `Age`, `Salary`, `Department`
- Age must be 18-65
- Salary must be > 0
- Name cannot be empty
- Methods: `GiveRaise(decimal percentage)`, `DisplayInfo()`

**Bonus**:
- Add employee ID auto-generation
- Track employment date
- Calculate years of service
- Implement performance rating system

---

---

### Problem 41: Static Members & Counters ⭐⭐
**Concepts**: Static Fields, Static Methods, Static Constructors

**What You'll Learn**:
- Difference between instance and static members
- Using static for shared data
- Static constructors

**Requirements**:
Create `Product` class:
```csharp
class Product
{
    private static int productCount = 0;
    private static decimal totalValue = 0;
    
    private int productId;
    private string name;
    private decimal price;
    
    public Product(string name, decimal price)
    {
        productCount++;
        productId = productCount;
        this.name = name;
        this.price = price;
        totalValue += price;
    }
    
    public static void DisplayStatistics() { }
}
```

**Bonus**:
- Add static method to find product by ID
- Implement singleton pattern
- Track most expensive product

---

---

### Problem 42: Product Catalog System ⭐⭐
**Concepts**: Multiple Objects, Collections Integration

**Requirements**:
Build a product catalog:
1. Create `Product` class (id, name, price, category)
2. Store products in a List
3. Methods: Add, Remove, Search, Display
4. Calculate total inventory value

**Bonus**:
- Group by category
- Sort by price
- Apply discount to category
- Low stock alerts

---

---

### Problem 43: Student Report Card ⭐⭐
**Concepts**: Class Design, Calculations, Formatting

**Requirements**:
Create `Student` class:
- Properties: Name, Roll Number, Subjects (Dictionary<string, int>)
- Methods:
  - `AddMarks(string subject, int marks)`
  - `CalculateTotal()`
  - `CalculatePercentage()`
  - `GetGrade()`
  - `GenerateReportCard()`

**Bonus**:
- Rank students by percentage
- Subject-wise topper
- Pass/Fail status
- Graphical representation (ASCII art)

---

## Section 2.2: Inheritance & Polymorphism (8 Problems)

---

### Problem 45: Vehicle with Constructor Chain ⭐⭐
**Concepts**: base() keyword, Constructor Chaining

**Requirements**:
```csharp
class Vehicle
{
    protected string licensePlate;
    protected int wheels;
    
    public Vehicle(string plate, int wheels)
    {
        this.licensePlate = plate;
        this.wheels = wheels;
        Console.WriteLine("Vehicle created");
    }
}

class Car : Vehicle
{
    private bool hasTrunk;
    
    public Car(string plate, int wheels, bool trunk) 
        : base(plate, wheels)
    {
        hasTrunk = trunk;
        Console.WriteLine("Car created");
    }
}
```

Observe constructor execution order.

**Bonus**: Create Motorcycle, Truck subclasses

---

---

### Problem 48: Method Overriding ⭐⭐
**Concepts**: virtual, override, base keyword

**Requirements**:
```csharp
class Employee
{
    public string Name { get; set; }
    public decimal BaseSalary { get; set; }
    
    public virtual decimal CalculateSalary()
    {
        return BaseSalary;
    }
}

class Manager : Employee
{
    public decimal Bonus { get; set; }
    
    public override decimal CalculateSalary()
    {
        return base.CalculateSalary() + Bonus;
    }
}

class Developer : Employee
{
    public int Projects { get; set; }
    
    public override decimal CalculateSalary()
    {
        return base.CalculateSalary() + (Projects * 1000);
    }
}
```

---

---

### Problem 49: Interface Implementation ⭐⭐
**Concepts**: Interfaces, Multiple Inheritance

**Requirements**:
```csharp
interface IPlayable
{
    void Play();
    void Pause();
    void Stop();
}

interface IRecordable
{
    void Record();
    void StopRecording();
}

class MediaPlayer : IPlayable, IRecordable
{
    // Implement all interface methods
}
```

**Bonus**: Add `IDownloadable` interface

---

---

### Problem 50: Multiple Interface Demo ⭐⭐
**Concepts**: Polymorphic Behavior with Interfaces

**Requirements**:
Create interfaces:
- `IMovable` (Move(), Speed property)
- `IAttackable` (Attack(), Damage property)

Create classes that implement various combinations:
- `Player` (IMovable, IAttackable)
- `Enemy` (IMovable, IAttackable)
- `NPC` (IMovable)

Demonstrate polymorphic collections.

---

---

### Problem 54: Operator Overloading ⭐⭐
**Concepts**: Operator Overloading, Custom Operators

**Requirements**:
```csharp
class ComplexNumber
{
    public double Real { get; set; }
    public double Imaginary { get; set; }
    
    public static ComplexNumber operator +(ComplexNumber a, ComplexNumber b)
    {
        return new ComplexNumber 
        { 
            Real = a.Real + b.Real,
            Imaginary = a.Imaginary + b.Imaginary
        };
    }
    
    // Implement -, *, ==, !=
}
```

**Bonus**: Implement ToString override

---

---

### Problem 55: Indexer Implementation ⭐⭐
**Concepts**: Custom [] Operator

**Requirements**:
```csharp
class Roster
{
    private List<Student> students = new List<Student>();
    
    public Student this[int index]
    {
        get { return students[index]; }
        set { students[index] = value; }
    }
    
    public Student this[string name]
    {
        get { return students.Find(s => s.Name == name); }
    }
}
```

---

---

### Problem 63: Second Largest Element ⭐⭐

---

### Problem 64: Remove Duplicates ⭐⭐

---

### Problem 65: Rotate Array (Left/Right) ⭐⭐

---

### Problem 67: Find Missing Number ⭐⭐

---

### Problem 68: Move Zeros to End ⭐⭐

---

### Problem 73: List of Objects (CRUD) ⭐⭐

---

### Problem 74: HashSet Duplicate Removal ⭐⭐

---

### Problem 75: HashSet Union/Intersection ⭐⭐

---

### Problem 76: Dictionary Basics ⭐⭐

---

### Problem 77: Character Frequency Counter ⭐⭐

---

### Problem 81: Stack Implementation (Array) ⭐⭐
**Requirements**:
```csharp
class MyStack<T>
{
    private T[] items;
    private int top;
    
    public void Push(T item) { }
    public T Pop() { }
    public T Peek() { }
    public bool IsEmpty() { }
}
```

---

### Problem 82: Queue Implementation (Array) ⭐⭐

---

### Problem 84: Reverse String Using Stack ⭐⭐

---

### Problem 91: Generic Value Swapper ⭐⭐
**Concepts**: Generic Methods, Type Parameters, ref Keyword, Type Inference

**What You'll Learn**:
- Creating generic methods
- Using type parameters (T, TKey, TValue)
- Type inference (compiler figures out T)
- Generic constraints basics
- When generics are better than object

**Requirements**:
Create a generic swap method that:
1. Works with any data type
2. Uses ref parameters to modify originals
3. Demonstrates type safety
4. Show type inference in action

**Complete Implementation**:
```csharp
class GenericSwapper
{
    // Generic method - works with ANY type
    public static void Swap<T>(ref T a, ref T b)
    {
        T temp = a;
        a = b;
        b = temp;
    }
    
    // Compare with non-generic (object-based) approach
    public static void SwapObject(ref object a, ref object b)
    {
        object temp = a;
        a = b;
        b = temp;
    }
}

// Demonstration
class SwapperDemo
{
    static void Main()
    {
        Console.WriteLine("=== GENERIC SWAP DEMO ===\n");
        
        // Swap integers
        int x = 10, y = 20;
        Console.WriteLine($"Before swap: x={x}, y={y}");
        GenericSwapper.Swap(ref x, ref y); // Type inference: T = int
        Console.WriteLine($"After swap:  x={x}, y={y}");
        
        // Swap strings
        string s1 = "Hello", s2 = "World";
        Console.WriteLine($"\nBefore swap: s1=\"{s1}\", s2=\"{s2}\"");
        GenericSwapper.Swap(ref s1, ref s2); // Type inference: T = string
        Console.WriteLine($"After swap:  s1=\"{s1}\", s2=\"{s2}\"");
        
        // Swap doubles
        double d1 = 3.14, d2 = 2.71;
        Console.WriteLine($"\nBefore swap: d1={d1}, d2={d2}");
        GenericSwapper.Swap(ref d1, ref d2);
        Console.WriteLine($"After swap:  d1={d1}, d2={d2}");
        
        // Swap custom objects
        Person p1 = new Person { Name = "Alice", Age = 25 };
        Person p2 = new Person { Name = "Bob", Age = 30 };
        Console.WriteLine($"\nBefore swap: p1={p1}, p2={p2}");
        GenericSwapper.Swap(ref p1, ref p2);
        Console.WriteLine($"After swap:  p1={p1}, p2={p2}");
        
        // Demonstrate explicit type parameter
        GenericSwapper.Swap<int>(ref x, ref y); // Explicit: T = int
        
        Console.WriteLine("\n=== WHY GENERICS ARE BETTER ===\n");
        
        // Problem with object-based approach
        int num1 = 100, num2 = 200;
        object obj1 = num1; // Boxing
        object obj2 = num2; // Boxing
        
        GenericSwapper.SwapObject(ref obj1, ref obj2);
        
        num1 = (int)obj1; // Unboxing - requires cast
        num2 = (int)obj2; // Unboxing - requires cast
        
        Console.WriteLine("Object approach problems:");
        Console.WriteLine("  1. Boxing/Unboxing (performance hit)");
        Console.WriteLine("  2. No compile-time type safety");
        Console.WriteLine("  3. Requires explicit casting");
        Console.WriteLine("\nGeneric approach benefits:");
        Console.WriteLine("  ✓ No boxing/unboxing");
        Console.WriteLine("  ✓ Compile-time type safety");
        Console.WriteLine("  ✓ No casting needed");
        Console.WriteLine("  ✓ Type inference (cleaner code)");
    }
}

class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public override string ToString() => $"{Name} ({Age})";
}
```

**Why Generics?**
```csharp
// WITHOUT GENERICS (old way):
public static void SwapInts(ref int a, ref int b) { } // For ints
public static void SwapStrings(ref string a, ref string b) { } // For strings
public static void SwapDoubles(ref double a, ref double b) { } // For doubles
// ... need separate method for EVERY type!

// WITH GENERICS (modern way):
public static void Swap<T>(ref T a, ref T b) { } // Works for ALL types!
```

**Type Inference Example**:
```csharp
int a = 5, b = 10;

// Explicit type parameter
Swap<int>(ref a, ref b);

// Type inference - compiler knows T = int from arguments
Swap(ref a, ref b); // Cleaner! ✓
```

**Test Cases**:
```csharp
// Value types
int x = 1, y = 2;
Swap(ref x, ref y);
Assert(x == 2 && y == 1);

// Reference types
string s1 = "A", s2 = "B";
Swap(ref s1, ref s2);
Assert(s1 == "B" && s2 == "A");

// Custom types
Person p1 = new Person(), p2 = new Person();
Swap(ref p1, ref p2);
// Objects swapped ✓
```

**Bonus Challenges**:
- ⭐⭐ Create generic Min/Max methods
- ⭐⭐ Create generic array rotation method
- ⭐⭐⭐ Add constraint: `where T : IComparable<T>`
- ⭐⭐⭐ Create three-way swap (a→b, b→c, c→a)

**Real-World Usage**:
- Collection libraries (List<T>, Dictionary<K,V>)
- LINQ methods (Where<T>, Select<T,R>)
- Utility methods
- Framework code

**Interview Tips**:
💡 Always explain: "Generics provide type safety without boxing"  
💡 Know the difference: Generic vs Object vs Overloading  
💡 Understand type inference  

---

---

### Problem 93: Nullable Product Pricing ⭐⭐
**Concepts**: Nullable Types, Null-Coalescing Operators, Value Types, Nullable<T>

**What You'll Learn**:
- Nullable value types (int?, double?)
- Null-coalescing operator (??)
- Null-conditional operator (?.)
- Null-coalescing assignment (??=)
- HasValue and Value properties
- Difference between reference and value type nullability

**Requirements**:
Create a product pricing system that handles:
1. Optional discount (nullable decimal)
2. Optional tax rate (nullable decimal)
3. Use ?? operator for defaults
4. Use ?. for safe navigation
5. Demonstrate all nullable operators

**Complete Implementation**:
```csharp
class Product
{
    public string Name { get; set; }
    public decimal BasePrice { get; set; }
    public decimal? Discount { get; set; }  // Nullable decimal
    public decimal? TaxRate { get; set; }   // Nullable decimal
    
    // Calculate final price with null handling
    public decimal GetFinalPrice()
    {
        // Apply discount if present, otherwise 0
        decimal discountAmount = BasePrice * (Discount ?? 0);
        
        // Price after discount
        decimal priceAfterDiscount = BasePrice - discountAmount;
        
        // Apply tax if present, otherwise 0
        decimal taxAmount = priceAfterDiscount * (TaxRate ?? 0);
        
        return priceAfterDiscount + taxAmount;
    }
    
    public void DisplayPricing()
    {
        Console.WriteLine($"\n{Name}:");
        Console.WriteLine($"  Base Price: ${BasePrice:F2}");
        
        // Use ?. operator for safe access
        if (Discount.HasValue)
        {
            Console.WriteLine($"  Discount: {Discount.Value * 100}%");
            Console.WriteLine($"  Discount Amount: ${BasePrice * Discount.Value:F2}");
        }
        else
        {
            Console.WriteLine($"  Discount: None");
        }
        
        if (TaxRate.HasValue)
        {
            decimal taxAmount = (BasePrice - (BasePrice * (Discount ?? 0))) * TaxRate.Value;
            Console.WriteLine($"  Tax Rate: {TaxRate.Value * 100}%");
            Console.WriteLine($"  Tax Amount: ${taxAmount:F2}");
        }
        else
        {
            Console.WriteLine($"  Tax: None");
        }
        
        Console.WriteLine($"  FINAL PRICE: ${GetFinalPrice():F2}");
    }
}

class NullableDemo
{
    static void Main()
    {
        Console.WriteLine("=== NULLABLE TYPES DEMO ===\n");
        
        // Product with both discount and tax
        Product p1 = new Product
        {
            Name = "Laptop",
            BasePrice = 1000m,
            Discount = 0.10m,    // 10% discount
            TaxRate = 0.08m      // 8% tax
        };
        p1.DisplayPricing();
        
        // Product with discount, no tax
        Product p2 = new Product
        {
            Name = "Mouse",
            BasePrice = 50m,
            Discount = 0.20m,    // 20% discount
            TaxRate = null       // No tax
        };
        p2.DisplayPricing();
        
        // Product with no discount, with tax
        Product p3 = new Product
        {
            Name = "Keyboard",
            BasePrice = 80m,
            Discount = null,     // No discount
            TaxRate = 0.08m      // 8% tax
        };
        p3.DisplayPricing();
        
        // Product with neither
        Product p4 = new Product
        {
            Name = "Cable",
            BasePrice = 10m,
            Discount = null,
            TaxRate = null
        };
        p4.DisplayPricing();
        
        Console.WriteLine("\n=== NULLABLE OPERATORS ===\n");
        
        // Null-coalescing operator (??)
        decimal? discount = null;
        decimal effectiveDiscount = discount ?? 0.05m; // Use 0.05 if null
        Console.WriteLine($"Discount (with ??): {effectiveDiscount * 100}%");
        
        // Null-coalescing assignment (??=)
        decimal? price = null;
        price ??= 100m; // Assign 100 if null
        Console.WriteLine($"Price (with ??=): ${price}");
        
        // Null-conditional operator (?.)
        Product product = null;
        decimal? finalPrice = product?.GetFinalPrice(); // null if product is null
        Console.WriteLine($"Final price of null product: {finalPrice?.ToString() ?? "N/A"}");
        
        product = new Product { Name = "Test", BasePrice = 50m };
        finalPrice = product?.GetFinalPrice(); // Calls method since product is not null
        Console.WriteLine($"Final price of real product: ${finalPrice}");
        
        DemonstrateNullableTypes();
    }
    
    static void DemonstrateNullableTypes()
    {
        Console.WriteLine("\n=== NULLABLE VALUE TYPES ===\n");
        
        // Regular int - cannot be null
        int regularInt = 10;
        // int cannotBeNull = null; // Compile error!
        
        // Nullable int - can be null
        int? nullableInt = null;
        Console.WriteLine($"Nullable int: {nullableInt?.ToString() ?? "null"}");
        
        nullableInt = 20;
        Console.WriteLine($"Nullable int: {nullableInt}");
        
        // HasValue and Value properties
        if (nullableInt.HasValue)
        {
            Console.WriteLine($"  Has value: {nullableInt.Value}");
        }
        
        // GetValueOrDefault
        int? maybeNull = null;
        int value = maybeNull.GetValueOrDefault(100); // Returns 100 if null
        Console.WriteLine($"GetValueOrDefault: {value}");
        
        // Nullable<T> is syntactic sugar
        Nullable<int> explicitNullable = 42;
        int? syntacticSugar = 42;
        // Both are the same!
        
        Console.WriteLine("\n=== NULLABLE REFERENCE TYPES (C# 8.0+) ===\n");
        Console.WriteLine("Reference types are nullable by default:");
        string? nullableString = null; // Explicitly nullable
        // string nonNullableString = null; // Warning in C# 8.0+ with nullable enabled
    }
}
```

**Visual Representation**:
```
Laptop Pricing:
  Base: $1000
  - Discount (10%): -$100
  ─────────────────────
  Subtotal: $900
  + Tax (8%): +$72
  ─────────────────────
  FINAL: $972

Calculation with nullable:
  Discount ?? 0     → Use discount if exists, else 0
  TaxRate ?? 0      → Use tax rate if exists, else 0
  product?.Method() → Call method if product not null
```

**Nullable Operators Cheat Sheet**:
```csharp
// ?? (Null-coalescing)
int? a = null;
int b = a ?? 10;  // b = 10 (use 10 if a is null)

// ??= (Null-coalescing assignment)
int? c = null;
c ??= 20;  // c = 20 (assign 20 if c is null)
c ??= 30;  // c = 20 (already has value, don't assign)

// ?. (Null-conditional)
Product p = null;
decimal? price = p?.BasePrice;  // price = null (p is null)

// ?[] (Null-conditional indexer)
int[]? array = null;
int? first = array?[0];  // first = null (array is null)

// Combining operators
decimal final = product?.GetPrice() ?? 0;
// If product is null OR GetPrice returns null, use 0
```

**Test Cases**:
```csharp
// Both values present
Product p1 = new Product { BasePrice = 100, Discount = 0.1m, TaxRate = 0.08m };
Assert(p1.GetFinalPrice() == 97.2m); // 100 - 10 + 7.2

// No discount
Product p2 = new Product { BasePrice = 100, Discount = null, TaxRate = 0.08m };
Assert(p2.GetFinalPrice() == 108m); // 100 + 8

// No tax
Product p3 = new Product { BasePrice = 100, Discount = 0.1m, TaxRate = null };
Assert(p3.GetFinalPrice() == 90m); // 100 - 10

// Neither
Product p4 = new Product { BasePrice = 100, Discount = null, TaxRate = null };
Assert(p4.GetFinalPrice() == 100m);
```

**Bonus Challenges**:
- ⭐⭐ Add nullable shipping cost
- ⭐⭐ Handle nullable quantity with default 1
- ⭐⭐⭐ Chain multiple nullable calculations
- ⭐⭐⭐ Implement custom nullable type wrapper

**Real-World Usage**:
- Optional configuration values
- Database fields that allow NULL
- User input that may be missing
- API responses with optional fields

**Interview Tips**:
💡 Know the difference: `T?` for value types, reference types nullable by default  
💡 Explain: "?? provides default for null values"  
💡 Mention C# 8.0 nullable reference types  

---

---

### Problem 96: Extension Method Playground ⭐⭐
**Concepts**: Extension Methods, this Keyword, Static Classes, Method Chaining

**What You'll Learn**:
- Creating extension methods
- Extending built-in types (string, int, IEnumerable)
- Method chaining
- LINQ-style extensions
- Limitations of extension methods

**Requirements**:
Create useful extension methods for:
1. String manipulation
2. Integer operations
3. Collection processing
4. Method chaining examples

**Complete Implementation**:
```csharp
// Extension methods MUST be in static class
static class StringExtensions
{
    // Extension for string - note 'this' keyword
    public static string ToTitleCase(this string str)
    {
        if (string.IsNullOrEmpty(str))
            return str;
        
        var words = str.ToLower().Split(' ');
        for (int i = 0; i < words.Length; i++)
        {
            if (words[i].Length > 0)
            {
                words[i] = char.ToUpper(words[i][0]) + words[i].Substring(1);
            }
        }
        
        return string.Join(" ", words);
    }
    
    public static string RemoveSpaces(this string str)
    {
        return str?.Replace(" ", "") ?? string.Empty;
    }
    
    public static bool IsPalindrome(this string str)
    {
        if (string.IsNullOrEmpty(str))
            return false;
        
        str = str.ToLower().Replace(" ", "");
        int left = 0, right = str.Length - 1;
        
        while (left < right)
        {
            if (str[left] != str[right])
                return false;
            left++;
            right--;
        }
        
        return true;
    }
    
    public static string Reverse(this string str)
    {
        if (string.IsNullOrEmpty(str))
            return str;
        
        char[] chars = str.ToCharArray();
        Array.Reverse(chars);
        return new string(chars);
    }
    
    public static int WordCount(this string str)
    {
        if (string.IsNullOrWhiteSpace(str))
            return 0;
        
        return str.Split(new[] { ' ' }, StringSplitOptions.RemoveEmptyEntries).Length;
    }
    
    // Chainable extension
    public static string TrimAll(this string str)
    {
        return str?.Trim();
    }
}

static class IntegerExtensions
{
    public static bool IsPrime(this int number)
    {
        if (number < 2) return false;
        if (number == 2) return true;
        if (number % 2 == 0) return false;
        
        for (int i = 3; i <= Math.Sqrt(number); i += 2)
        {
            if (number % i == 0)
                return false;
        }
        
        return true;
    }
    
    public static bool IsEven(this int number)
    {
        return number % 2 == 0;
    }
    
    public static bool IsOdd(this int number)
    {
        return number % 2 != 0;
    }
    
    public static int Factorial(this int number)
    {
        if (number < 0)
            throw new ArgumentException("Cannot calculate factorial of negative number");
        
        if (number == 0 || number == 1)
            return 1;
        
        int result = 1;
        for (int i = 2; i <= number; i++)
        {
            result *= i;
        }
        
        return result;
    }
    
    public static int DigitSum(this int number)
    {
        int sum = 0;
        number = Math.Abs(number);
        
        while (number > 0)
        {
            sum += number % 10;
            number /= 10;
        }
        
        return sum;
    }
}

static class CollectionExtensions
{
    // Extension for any IEnumerable<T>
    public static void ForEach<T>(this IEnumerable<T> source, Action<T> action)
    {
        foreach (T item in source)
        {
            action(item);
        }
    }
    
    public static IEnumerable<T> Shuffle<T>(this IEnumerable<T> source)
    {
        Random rng = new Random();
        return source.OrderBy(x => rng.Next());
    }
    
    public static T Median<T>(this IEnumerable<T> source) where T : struct, IComparable<T>
    {
        var sorted = source.OrderBy(x => x).ToList();
        int count = sorted.Count;
        
        if (count == 0)
            throw new InvalidOperationException("Sequence contains no elements");
        
        if (count % 2 == 0)
        {
            // For even count, average of middle two
            dynamic a = sorted[count / 2 - 1];
            dynamic b = sorted[count / 2];
            return (T)((a + b) / 2);
        }
        else
        {
            return sorted[count / 2];
        }
    }
    
    public static string ToDelimitedString<T>(this IEnumerable<T> source, string delimiter = ", ")
    {
        return string.Join(delimiter, source);
    }
}

// Demonstration
class ExtensionMethodDemo
{
    static void Main()
    {
        Console.WriteLine("=== EXTENSION METHODS DEMO ===\n");
        
        // STRING EXTENSIONS
        Console.WriteLine("--- String Extensions ---");
        string text = "hello world";
        Console.WriteLine($"Original: \"{text}\"");
        Console.WriteLine($"ToTitleCase: \"{text.ToTitleCase()}\"");
        Console.WriteLine($"RemoveSpaces: \"{text.RemoveSpaces()}\"");
        Console.WriteLine($"Reverse: \"{text.Reverse()}\"");
        Console.WriteLine($"WordCount: {text.WordCount()}");
        
        string palindrome = "racecar";
        Console.WriteLine($"\nIs \"{palindrome}\" a palindrome? {palindrome.IsPalindrome()}");
        
        // METHOD CHAINING
        Console.WriteLine("\n--- Method Chaining ---");
        string messy = "  HELLO WORLD  ";
        string clean = messy.TrimAll().ToTitleCase();
        Console.WriteLine($"Chained: \"{messy}\" → \"{clean}\"");
        
        // INTEGER EXTENSIONS
        Console.WriteLine("\n--- Integer Extensions ---");
        int number = 17;
        Console.WriteLine($"{number} is prime: {number.IsPrime()}");
        Console.WriteLine($"{number} is even: {number.IsEven()}");
        Console.WriteLine($"{number} is odd: {number.IsOdd()}");
        
        int num = 5;
        Console.WriteLine($"{num}! = {num.Factorial()}");
        
        int digits = 12345;
        Console.WriteLine($"Digit sum of {digits}: {digits.DigitSum()}");
        
        // COLLECTION EXTENSIONS
        Console.WriteLine("\n--- Collection Extensions ---");
        
        var numbers = new List<int> { 1, 2, 3, 4, 5 };
        
        Console.Write("ForEach: ");
        numbers.ForEach(n => Console.Write($"{n} "));
        Console.WriteLine();
        
        Console.WriteLine($"Shuffled: {numbers.Shuffle().ToDelimitedString()}");
        Console.WriteLine($"Median: {numbers.Median()}");
        
        var words = new[] { "apple", "banana", "cherry" };
        Console.WriteLine($"Delimited: {words.ToDelimitedString(" | ")}");
    }
}
```

**Extension Method Syntax Explained**:
```csharp
// REGULAR STATIC METHOD
public static class StringHelper
{
    public static string Reverse(string str)
    {
        // ...
    }
}

// Usage:
string reversed = StringHelper.Reverse("hello");

// EXTENSION METHOD (note 'this' keyword)
public static class StringExtensions
{
    public static string Reverse(this string str)
    {
        // ...
    }
}

// Usage (looks like instance method!):
string reversed = "hello".Reverse(); // ✨ Magic!
```

**Method Chaining**:
```csharp
// All return string, so we can chain
string result = "  hello world  "
    .TrimAll()           // Remove whitespace
    .ToTitleCase()       // Capitalize
    .Reverse()           // Reverse
    .RemoveSpaces();     // Remove spaces

// Much cleaner than:
string temp1 = input.TrimAll();
string temp2 = temp1.ToTitleCase();
string temp3 = temp2.Reverse();
string result = temp3.RemoveSpaces();
```

**Extension Method Rules**:
1. ✅ Must be in static class
2. ✅ Must be static method
3. ✅ First parameter has `this` keyword
4. ✅ Can extend any type (even sealed!)
5. ❌ Cannot access private members
6. ❌ Instance methods have priority over extensions
7. ❌ Cannot override existing methods

**Real-World Examples**:
```csharp
// LINQ is built on extension methods!
var result = numbers
    .Where(n => n > 5)      // Extension method
    .Select(n => n * 2)     // Extension method
    .OrderBy(n => n)        // Extension method
    .ToList();              // Extension method

// ASP.NET Core uses them heavily
services.AddMvc()          // Extension method
        .AddJsonOptions()  // Extension method
        .Configure();      // Extension method
```

**Test Cases**:
```csharp
// String extensions
Assert("hello world".ToTitleCase() == "Hello World");
Assert("racecar".IsPalindrome() == true);
Assert("hello".Reverse() == "olleh");

// Integer extensions
Assert(5.Factorial() == 120);
Assert(17.IsPrime() == true);
Assert(123.DigitSum() == 6);

// Collection extensions
var nums = new[] { 1, 3, 5 };
Assert(nums.Median() == 3);
```

**Bonus Challenges**:
- ⭐⭐ Create DateTime extensions (AddWeeks, IsWeekend)
- ⭐⭐ Create List<T> extensions (AddRange with condition)
- ⭐⭐⭐ Create custom LINQ-style operators
- ⭐⭐⭐⭐ Build fluent API using extensions

**Real-World Usage**:
- LINQ (Where, Select, OrderBy, etc.)
- ASP.NET Core (AddMvc, UseRouting, etc.)
- Entity Framework (Include, ThenInclude, etc.)
- Custom utility libraries

**Interview Tips**:
💡 Mention: "LINQ is built entirely on extension methods"  
💡 Explain: "First parameter with 'this' makes it an extension"  
💡 Know: "Can extend sealed classes and interfaces"  
💡 Important: "Cannot access private members"  

---

## ✅ Section 4.1 Complete!

**Generics & Constraints (6/6 Problems)** ✅

**Concepts Mastered**:
- ✅ Generic methods and classes
- ✅ Type parameters (T, TKey, TValue)
- ✅ Type constraints (where clauses)
- ✅ Nullable types and operators
- ✅ Static classes and methods
- ✅ Partial classes
- ✅ Extension methods

---

## 🎯 Next Up: Section 4.2 - LINQ Mastery

**THIS IS THE MOST IMPORTANT SECTION FOR JOBS!**

LINQ appears in:
- 95% of C# job descriptions
- Every real-world C# application
- Most common interview questions

**10 Problems covering**:
- Query syntax vs Method syntax
- Filtering, Sorting, Projecting
- Grouping, Joining, Aggregating
- Performance optimization
- Custom operators

Ready to master LINQ? 🚀

---

### Problem 97: LINQ Filtering & Sorting ⭐⭐
**Concepts**: Where, OrderBy, OrderByDescending, ThenBy, Method Syntax vs Query Syntax

**What You'll Learn**:
- Filtering with Where()
- Single and multi-level sorting
- Method syntax (fluent API)
- Query syntax (SQL-like)
- When to use each syntax
- Chaining LINQ operations

**Requirements**:
Work with a collection of products:
1. Filter by price range
2. Filter by category
3. Sort by price
4. Multi-level sorting (category, then price)
5. Show both method and query syntax

**Complete Implementation**:
```csharp
class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
    public int Stock { get; set; }
    
    public override string ToString()
    {
        return $"[{Id}] {Name} - ${Price:F2} ({Category}) Stock: {Stock}";
    }
}

class LinqFilteringSorting
{
    static List<Product> GetProducts()
    {
        return new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999.99m, Category = "Electronics", Stock = 15 },
            new Product { Id = 2, Name = "Mouse", Price = 29.99m, Category = "Electronics", Stock = 50 },
            new Product { Id = 3, Name = "Keyboard", Price = 79.99m, Category = "Electronics", Stock = 30 },
            new Product { Id = 4, Name = "Monitor", Price = 299.99m, Category = "Electronics", Stock = 20 },
            new Product { Id = 5, Name = "Desk", Price = 399.99m, Category = "Furniture", Stock = 10 },
            new Product { Id = 6, Name = "Chair", Price = 149.99m, Category = "Furniture", Stock = 25 },
            new Product { Id = 7, Name = "Lamp", Price = 49.99m, Category = "Furniture", Stock = 40 },
            new Product { Id = 8, Name = "Notebook", Price = 4.99m, Category = "Stationery", Stock = 100 },
            new Product { Id = 9, Name = "Pen", Price = 1.99m, Category = "Stationery", Stock = 200 },
            new Product { Id = 10, Name = "Headphones", Price = 89.99m, Category = "Electronics", Stock = 35 }
        };
    }
    
    static void Main()
    {
        var products = GetProducts();
        
        Console.WriteLine("=== LINQ FILTERING & SORTING ===\n");
        
        // FILTERING - Method Syntax
        Console.WriteLine("--- Products under $100 (Method Syntax) ---");
        var affordable = products.Where(p => p.Price < 100);
        
        foreach (var product in affordable)
        {
            Console.WriteLine($"  {product}");
        }
        
        // FILTERING - Query Syntax
        Console.WriteLine("\n--- Products under $100 (Query Syntax) ---");
        var affordableQuery = from p in products
                             where p.Price < 100
                             select p;
        
        foreach (var product in affordableQuery)
        {
            Console.WriteLine($"  {product}");
        }
        
        // MULTIPLE CONDITIONS
        Console.WriteLine("\n--- Electronics under $100 ---");
        var cheapElectronics = products.Where(p => p.Category == "Electronics" && p.Price < 100);
        
        foreach (var product in cheapElectronics)
        {
            Console.WriteLine($"  {product}");
        }
        
        // SORTING - Single Level
        Console.WriteLine("\n--- All products sorted by price (ascending) ---");
        var sortedByPrice = products.OrderBy(p => p.Price);
        
        foreach (var product in sortedByPrice)
        {
            Console.WriteLine($"  {product}");
        }
        
        // SORTING - Descending
        Console.WriteLine("\n--- All products sorted by price (descending) ---");
        var sortedByPriceDesc = products.OrderByDescending(p => p.Price);
        
        foreach (var product in sortedByPriceDesc.Take(3))
        {
            Console.WriteLine($"  {product}");
        }
        
        // MULTI-LEVEL SORTING
        Console.WriteLine("\n--- Sorted by Category, then Price ---");
        var multiSort = products.OrderBy(p => p.Category)
                               .ThenBy(p => p.Price);
        
        foreach (var product in multiSort)
        {
            Console.WriteLine($"  {product}");
        }
        
        // COMPLEX SORTING
        Console.WriteLine("\n--- Category (asc), then Price (desc) ---");
        var complexSort = products.OrderBy(p => p.Category)
                                 .ThenByDescending(p => p.Price);
        
        foreach (var product in complexSort)
        {
            Console.WriteLine($"  {product}");
        }
        
        // FILTERING + SORTING COMBINED
        Console.WriteLine("\n--- In-stock items sorted by stock level ---");
        var inStock = products.Where(p => p.Stock > 0)
                             .OrderByDescending(p => p.Stock);
        
        foreach (var product in inStock.Take(5))
        {
            Console.WriteLine($"  {product}");
        }
        
        // QUERY SYNTAX - Complex
        Console.WriteLine("\n--- Query Syntax: Electronics sorted by price ---");
        var queryComplex = from p in products
                          where p.Category == "Electronics"
                          orderby p.Price descending
                          select p;
        
        foreach (var product in queryComplex)
        {
            Console.WriteLine($"  {product}");
        }
    }
}
```

**Method Syntax vs Query Syntax Comparison**:
```csharp
// METHOD SYNTAX (Fluent API) - More common in real code
var result = products
    .Where(p => p.Price < 100)
    .OrderBy(p => p.Name)
    .Select(p => p.Name);

// QUERY SYNTAX (SQL-like) - More readable for complex queries
var result = from p in products
             where p.Price < 100
             orderby p.Name
             select p.Name;

// Both produce the SAME result!
```

**Common LINQ Filtering Patterns**:
```csharp
// Single condition
products.Where(p => p.Price > 100)

// Multiple AND conditions
products.Where(p => p.Price > 100 && p.Stock > 0)

// Multiple OR conditions
products.Where(p => p.Category == "Electronics" || p.Category == "Furniture")

// Negation
products.Where(p => p.Price <= 100)  // NOT greater than 100
products.Where(p => !p.Name.Contains("Pro"))  // NOT contains

// String operations
products.Where(p => p.Name.StartsWith("L"))
products.Where(p => p.Name.Contains("book"))
products.Where(p => p.Category.Equals("Electronics", StringComparison.OrdinalIgnoreCase))

// Null checks
products.Where(p => p.Description != null)
products.Where(p => !string.IsNullOrEmpty(p.Name))

// Range checks
products.Where(p => p.Price >= 50 && p.Price <= 150)

// Collection membership
var categories = new[] { "Electronics", "Furniture" };
products.Where(p => categories.Contains(p.Category))
```

**Sorting Patterns**:
```csharp
// Ascending (default)
products.OrderBy(p => p.Price)

// Descending
products.OrderByDescending(p => p.Price)

// Multi-level
products.OrderBy(p => p.Category)
        .ThenBy(p => p.Price)
        .ThenBy(p => p.Name)

// Mixed ascending/descending
products.OrderBy(p => p.Category)
        .ThenByDescending(p => p.Price)

// Custom comparison
products.OrderBy(p => p.Name.Length)
        .ThenBy(p => p.Name)
```

**Performance Tips**:
```csharp
// ❌ BAD - Multiple iterations
var result1 = products.Where(p => p.Price > 100).ToList();
var result2 = result1.OrderBy(p => p.Name).ToList();

// ✅ GOOD - Single chain
var result = products
    .Where(p => p.Price > 100)
    .OrderBy(p => p.Name)
    .ToList();  // Only materialize once at the end

// ❌ BAD - Filter after sorting (wastes time)
var bad = products.OrderBy(p => p.Price).Where(p => p.Price > 100);

// ✅ GOOD - Filter first, then sort (fewer items to sort)
var good = products.Where(p => p.Price > 100).OrderBy(p => p.Price);
```

**Bonus Challenges**:
- ⭐⭐ Add pagination (Skip, Take)
- ⭐⭐ Implement search across multiple fields
- ⭐⭐⭐ Create dynamic sorting based on user input
- ⭐⭐⭐ Optimize for very large collections

**Real-World Usage**:
- E-commerce product filtering
- Search results sorting
- Admin dashboards
- Report generation
- Data grids and tables

**Interview Tips**:
💡 Know both syntaxes - method syntax is more common  
💡 Explain: "Filter before sorting for better performance"  
💡 Common question: "How would you add pagination?"  
💡 Mention deferred execution  

---

---

### Problem 98: LINQ Aggregations ⭐⭐
**Concepts**: Sum, Average, Count, Max, Min, Aggregate, Any, All

**What You'll Learn**:
- Calculating totals and statistics
- Counting with conditions
- Finding extremes
- Checking existence
- Custom aggregations
- Handling empty sequences

**Requirements**:
Perform statistical analysis on collections:
1. Calculate sums and averages
2. Find max/min values
3. Count with conditions
4. Check for existence
5. Custom aggregations

**Complete Implementation**:
```csharp
class SalesRecord
{
    public int Id { get; set; }
    public string ProductName { get; set; }
    public string Region { get; set; }
    public decimal Amount { get; set; }
    public int Quantity { get; set; }
    public DateTime Date { get; set; }
    
    public override string ToString()
    {
        return $"{ProductName} - ${Amount:F2} ({Quantity} units) [{Region}] on {Date:yyyy-MM-dd}";
    }
}

class LinqAggregations
{
    static List<SalesRecord> GetSales()
    {
        return new List<SalesRecord>
        {
            new SalesRecord { Id = 1, ProductName = "Laptop", Region = "North", Amount = 1200m, Quantity = 2, Date = new DateTime(2024, 1, 15) },
            new SalesRecord { Id = 2, ProductName = "Mouse", Region = "North", Amount = 30m, Quantity = 10, Date = new DateTime(2024, 1, 16) },
            new SalesRecord { Id = 3, ProductName = "Keyboard", Region = "South", Amount = 80m, Quantity = 5, Date = new DateTime(2024, 1, 17) },
            new SalesRecord { Id = 4, ProductName = "Monitor", Region = "East", Amount = 600m, Quantity = 3, Date = new DateTime(2024, 1, 18) },
            new SalesRecord { Id = 5, ProductName = "Laptop", Region = "West", Amount = 2400m, Quantity = 4, Date = new DateTime(2024, 1, 19) },
            new SalesRecord { Id = 6, ProductName = "Mouse", Region = "South", Amount = 45m, Quantity = 15, Date = new DateTime(2024, 1, 20) },
            new SalesRecord { Id = 7, ProductName = "Desk", Region = "North", Amount = 800m, Quantity = 2, Date = new DateTime(2024, 1, 21) },
            new SalesRecord { Id = 8, ProductName = "Chair", Region = "East", Amount = 300m, Quantity = 6, Date = new DateTime(2024, 1, 22) }
        };
    }
    
    static void Main()
    {
        var sales = GetSales();
        
        Console.WriteLine("=== LINQ AGGREGATIONS ===\n");
        
        // SUM
        Console.WriteLine("--- Sum Aggregations ---");
        decimal totalRevenue = sales.Sum(s => s.Amount);
        Console.WriteLine($"Total Revenue: ${totalRevenue:F2}");
        
        int totalQuantity = sales.Sum(s => s.Quantity);
        Console.WriteLine($"Total Quantity Sold: {totalQuantity} units");
        
        // Sum with condition
        decimal northRevenue = sales.Where(s => s.Region == "North")
                                   .Sum(s => s.Amount);
        Console.WriteLine($"North Region Revenue: ${northRevenue:F2}");
        
        // AVERAGE
        Console.WriteLine("\n--- Average Calculations ---");
        decimal avgSale = sales.Average(s => s.Amount);
        Console.WriteLine($"Average Sale Amount: ${avgSale:F2}");
        
        double avgQuantity = sales.Average(s => s.Quantity);
        Console.WriteLine($"Average Quantity per Sale: {avgQuantity:F2} units");
        
        // COUNT
        Console.WriteLine("\n--- Count Operations ---");
        int totalSales = sales.Count();
        Console.WriteLine($"Total Sales: {totalSales}");
        
        // Count with condition
        int largeSales = sales.Count(s => s.Amount > 500);
        Console.WriteLine($"Sales over $500: {largeSales}");
        
        int laptopSales = sales.Count(s => s.ProductName == "Laptop");
        Console.WriteLine($"Laptop sales: {laptopSales}");
        
        // MAX / MIN
        Console.WriteLine("\n--- Max/Min Values ---");
        decimal maxSale = sales.Max(s => s.Amount);
        Console.WriteLine($"Largest Sale: ${maxSale:F2}");
        
        decimal minSale = sales.Min(s => s.Amount);
        Console.WriteLine($"Smallest Sale: ${minSale:F2}");
        
        // Finding the item with max value (not just the max value)
        var largestSale = sales.OrderByDescending(s => s.Amount).First();
        Console.WriteLine($"Largest Sale Details: {largestSale}");
        
        // Or using MaxBy (C# 6.0+)
        var largestSaleMaxBy = sales.MaxBy(s => s.Amount);
        Console.WriteLine($"Largest Sale (MaxBy): {largestSaleMaxBy}");
        
        // ANY - Check existence
        Console.WriteLine("\n--- Existence Checks (Any) ---");
        bool hasNorthSales = sales.Any(s => s.Region == "North");
        Console.WriteLine($"Has North region sales? {hasNorthSales}");
        
        bool hasLargeSales = sales.Any(s => s.Amount > 1000);
        Console.WriteLine($"Has sales over $1000? {hasLargeSales}");
        
        bool hasWestSales = sales.Any(s => s.Region == "West");
        Console.WriteLine($"Has West region sales? {hasWestSales}");
        
        // ALL - Check if all match
        Console.WriteLine("\n--- Universal Checks (All) ---");
        bool allPositive = sales.All(s => s.Amount > 0);
        Console.WriteLine($"All sales positive? {allPositive}");
        
        bool allLarge = sales.All(s => s.Amount > 100);
        Console.WriteLine($"All sales over $100? {allLarge}");
        
        // AGGREGATE - Custom aggregation
        Console.WriteLine("\n--- Custom Aggregation ---");
        
        // Calculate total with custom logic
        decimal total = sales.Aggregate(0m, (acc, s) => acc + s.Amount);
        Console.WriteLine($"Total (using Aggregate): ${total:F2}");
        
        // Build a summary string
        string summary = sales.Aggregate("Sales: ", 
            (acc, s) => acc + $"{s.ProductName}, ");
        Console.WriteLine(summary.TrimEnd(',', ' '));
        
        // Complex aggregation - running total with max tracking
        var runningTotal = sales.Aggregate(
            new { Total = 0m, Max = 0m },
            (acc, s) => new { 
                Total = acc.Total + s.Amount, 
                Max = Math.Max(acc.Max, s.Amount) 
            });
        Console.WriteLine($"Running Total: ${runningTotal.Total:F2}, Max: ${runningTotal.Max:F2}");
        
        // STATISTICS DASHBOARD
        Console.WriteLine("\n=== SALES DASHBOARD ===");
        DisplayStatistics(sales);
        
        // REGIONAL BREAKDOWN
        Console.WriteLine("\n=== REGIONAL BREAKDOWN ===");
        var regions = sales.Select(s => s.Region).Distinct();
        foreach (var region in regions)
        {
            var regionSales = sales.Where(s => s.Region == region);
            Console.WriteLine($"\n{region} Region:");
            Console.WriteLine($"  Sales Count: {regionSales.Count()}");
            Console.WriteLine($"  Total Revenue: ${regionSales.Sum(s => s.Amount):F2}");
            Console.WriteLine($"  Average Sale: ${regionSales.Average(s => s.Amount):F2}");
            Console.WriteLine($"  Largest Sale: ${regionSales.Max(s => s.Amount):F2}");
        }
    }
    
    static void DisplayStatistics(List<SalesRecord> sales)
    {
        Console.WriteLine("┌────────────────────────────────────┐");
        Console.WriteLine("│        SALES STATISTICS            │");
        Console.WriteLine("├────────────────────────────────────┤");
        Console.WriteLine($"│ Total Sales:        {sales.Count(),15} │");
        Console.WriteLine($"│ Total Revenue:      ${sales.Sum(s => s.Amount),14:F2} │");
        Console.WriteLine($"│ Average Sale:       ${sales.Average(s => s.Amount),14:F2} │");
        Console.WriteLine($"│ Largest Sale:       ${sales.Max(s => s.Amount),14:F2} │");
        Console.WriteLine($"│ Smallest Sale:      ${sales.Min(s => s.Amount),14:F2} │");
        Console.WriteLine($"│ Total Units:        {sales.Sum(s => s.Quantity),15} │");
        Console.WriteLine($"│ Avg Units/Sale:     {sales.Average(s => s.Quantity),15:F2} │");
        Console.WriteLine("└────────────────────────────────────┘");
    }
}
```

**Common Aggregation Patterns**:
```csharp
// Sum
decimal total = items.Sum(i => i.Price);

// Sum with filtering
decimal filteredSum = items.Where(i => i.Active).Sum(i => i.Price);

// Average
double avg = items.Average(i => i.Score);

// Count total
int count = items.Count();

// Count with condition
int activeCount = items.Count(i => i.Active);

// Max/Min
int max = items.Max(i => i.Value);
int min = items.Min(i => i.Value);

// Find item with max value
var maxItem = items.OrderByDescending(i => i.Value).First();
// Or (C# 6.0+)
var maxItem = items.MaxBy(i => i.Value);

// Check if any match
bool exists = items.Any(i => i.Price > 100);

// Check if all match
bool allValid = items.All(i => i.Price > 0);

// First/Last with default
var first = items.FirstOrDefault(i => i.Active);
var last = items.LastOrDefault(i => i.Active);
```

**Handling Empty Sequences**:
```csharp
List<int> empty = new List<int>();

// These throw exception on empty:
// empty.Max()  ❌
// empty.Min()  ❌
// empty.Average()  ❌
// empty.First()  ❌

// Safe alternatives:
int max = empty.DefaultIfEmpty(0).Max();  ✅
double avg = empty.DefaultIfEmpty(0).Average();  ✅
int first = empty.FirstOrDefault();  ✅
bool any = empty.Any();  ✅ (returns false)
```

**Bonus Challenges**:
- ⭐⭐ Calculate standard deviation
- ⭐⭐ Find percentile values
- ⭐⭐⭐ Create custom aggregation extension method
- ⭐⭐⭐⭐ Build complete analytics dashboard

**Real-World Usage**:
- Sales reports and dashboards
- Financial calculations
- Analytics and BI
- Performance metrics
- Data validation

**Interview Tips**:
💡 Know difference between Count() and Count  
💡 Mention: "Any() is more efficient than Count() > 0"  
💡 Always handle empty sequences  
💡 Common question: "Calculate running total"  

---

---

### Problem 102: LINQ Projection (Anonymous Types) ⭐⭐
**Concepts**: Select, Anonymous Types, Named Tuples, Custom Projections

**What You'll Learn**:
- Projecting to anonymous types
- Transforming data shape
- Selecting specific properties
- Creating computed properties
- Using ValueTuple

**Complete Implementation**:
```csharp
class Employee
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Department { get; set; }
    public decimal Salary { get; set; }
    public DateTime HireDate { get; set; }
}

class LinqProjection
{
    static void Main()
    {
        var employees = new List<Employee>
        {
            new Employee { Id = 1, FirstName = "Alice", LastName = "Johnson", Department = "Engineering", Salary = 95000, HireDate = new DateTime(2019, 3, 15) },
            new Employee { Id = 2, FirstName = "Bob", LastName = "Smith", Department = "Sales", Salary = 75000, HireDate = new DateTime(2020, 6, 1) },
            new Employee { Id = 3, FirstName = "Charlie", LastName = "Brown", Department = "Engineering", Salary = 88000, HireDate = new DateTime(2018, 9, 20) },
            new Employee { Id = 4, FirstName = "Diana", LastName = "Prince", Department = "Marketing", Salary = 70000, HireDate = new DateTime(2021, 1, 10) }
        };
        
        Console.WriteLine("=== LINQ PROJECTION ===\n");
        
        // SELECT - Specific properties
        Console.WriteLine("--- Names Only ---");
        var names = employees.Select(e => e.FirstName);
        foreach (var name in names)
        {
            Console.WriteLine(name);
        }
        
        // ANONYMOUS TYPE - Multiple properties
        Console.WriteLine("\n--- Anonymous Type Projection ---");
        var simplified = employees.Select(e => new
        {
            Name = $"{e.FirstName} {e.LastName}",
            e.Department,
            e.Salary
        });
        
        foreach (var emp in simplified)
        {
            Console.WriteLine($"{emp.Name} - {emp.Department} - ${emp.Salary}");
        }
        
        // COMPUTED PROPERTIES
        Console.WriteLine("\n--- With Computed Properties ---");
        var withComputed = employees.Select(e => new
        {
            FullName = $"{e.FirstName} {e.LastName}",
            YearsOfService = DateTime.Now.Year - e.HireDate.Year,
            AnnualSalary = e.Salary * 12,
            Seniority = DateTime.Now.Year - e.HireDate.Year >= 3 ? "Senior" : "Junior"
        });
        
        foreach (var emp in withComputed)
        {
            Console.WriteLine($"{emp.FullName}: {emp.YearsOfService} years, ${emp.AnnualSalary}/year ({emp.Seniority})");
        }
        
        // VALUE TUPLES (C# 7.0+)
        Console.WriteLine("\n--- Value Tuples ---");
        var tuples = employees.Select(e => (
            Name: $"{e.FirstName} {e.LastName}",
            Dept: e.Department,
            Pay: e.Salary
        ));
        
        foreach (var (name, dept, pay) in tuples)  // Deconstruction
        {
            Console.WriteLine($"{name} in {dept} earns ${pay}");
        }
        
        // INDEX-BASED PROJECTION
        Console.WriteLine("\n--- With Index ---");
        var indexed = employees.Select((e, index) => new
        {
            Position = index + 1,
            Name = e.FirstName,
            Department = e.Department
        });
        
        foreach (var item in indexed)
        {
            Console.WriteLine($"{item.Position}. {item.Name} ({item.Department})");
        }
        
        // NESTED ANONYMOUS TYPES
        Console.WriteLine("\n--- Nested Anonymous Types ---");
        var nested = employees.Select(e => new
        {
            Personal = new { e.FirstName, e.LastName },
            Work = new { e.Department, e.Salary },
            Tenure = DateTime.Now.Year - e.HireDate.Year
        });
        
        foreach (var emp in nested)
        {
            Console.WriteLine($"{emp.Personal.FirstName} {emp.Personal.LastName}:");
            Console.WriteLine($"  Works in {emp.Work.Department}");
            Console.WriteLine($"  Earns ${emp.Work.Salary}");
            Console.WriteLine($"  {emp.Tenure} years of service");
        }
        
        // CONDITIONAL PROJECTION
        Console.WriteLine("\n--- Conditional Projection ---");
        var conditional = employees.Select(e => new
        {
            Name = $"{e.FirstName} {e.LastName}",
            SalaryLevel = e.Salary >= 80000 ? "High" : "Standard",
            Bonus = e.Salary >= 80000 ? e.Salary * 0.1m : e.Salary * 0.05m
        });
        
        foreach (var emp in conditional)
        {
            Console.WriteLine($"{emp.Name}: {emp.SalaryLevel} (Bonus: ${emp.Bonus})");
        }
    }
}
```

**Projection Patterns**:
```csharp
// Simple property selection
var names = items.Select(i => i.Name);

// Anonymous type with multiple properties
var result = items.Select(i => new { i.Name, i.Price });

// Computed properties
var computed = items.Select(i => new { 
    i.Name, 
    Total = i.Price * i.Quantity 
});

// String formatting
var formatted = items.Select(i => $"{i.Name}: ${i.Price}");

// Conditional logic
var conditional = items.Select(i => new {
    i.Name,
    Status = i.Stock > 0 ? "Available" : "Out of Stock"
});

// With index
var indexed = items.Select((item, index) => new {
    Position = index,
    item.Name
});
```

**Bonus Challenges**:
- ⭐⭐ Project to custom class instead of anonymous type
- ⭐⭐ Create DTO (Data Transfer Object) projections
- ⭐⭐⭐ Implement AutoMapper-like projection
- ⭐⭐⭐ Dynamic projection based on runtime config

---

---

### Problem 103: LINQ Method vs Query Syntax ⭐⭐
**Concepts**: Syntax Comparison, Readability, Complex Queries, Conversion

**What You'll Learn**:
- When to use each syntax
- Converting between syntaxes
- Pros and cons of each
- Combining both syntaxes

**Requirements**:
Compare the two LINQ syntaxes:
1. Same query in both syntaxes
2. Complex queries comparison
3. Performance considerations
4. Best practices

**Implementation**:
```csharp
class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
}

class MethodVsQuerySyntax
{
    static void Main()
    {
        var products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Category = "Electronics", Price = 999, Stock = 5 },
            new Product { Id = 2, Name = "Mouse", Category = "Electronics", Price = 29, Stock = 50 },
            new Product { Id = 3, Name = "Desk", Category = "Furniture", Price = 299, Stock = 10 },
            new Product { Id = 4, Name = "Chair", Category = "Furniture", Price = 149, Stock = 20 },
            new Product { Id = 5, Name = "Monitor", Category = "Electronics", Price = 399, Stock = 15 }
        };
        
        Console.WriteLine("=== METHOD SYNTAX vs QUERY SYNTAX ===\n");
        
        // EXAMPLE 1: Simple Filter
        Console.WriteLine("--- Example 1: Filter Products Under $300 ---\n");
        
        // Method Syntax
        var method1 = products.Where(p => p.Price < 300);
        
        // Query Syntax
        var query1 = from p in products
                    where p.Price < 300
                    select p;
        
        Console.WriteLine("Method Syntax:");
        Console.WriteLine("var result = products.Where(p => p.Price < 300);");
        
        Console.WriteLine("\nQuery Syntax:");
        Console.WriteLine("var result = from p in products");
        Console.WriteLine("             where p.Price < 300");
        Console.WriteLine("             select p;");
        
        // EXAMPLE 2: Filter + Sort
        Console.WriteLine("\n\n--- Example 2: Filter and Sort ---\n");
        
        // Method Syntax (chaining)
        var method2 = products
            .Where(p => p.Stock > 0)
            .OrderBy(p => p.Price);
        
        // Query Syntax
        var query2 = from p in products
                    where p.Stock > 0
                    orderby p.Price
                    select p;
        
        Console.WriteLine("Method Syntax:");
        Console.WriteLine("var result = products");
        Console.WriteLine("    .Where(p => p.Stock > 0)");
        Console.WriteLine("    .OrderBy(p => p.Price);");
        
        Console.WriteLine("\nQuery Syntax:");
        Console.WriteLine("var result = from p in products");
        Console.WriteLine("             where p.Stock > 0");
        Console.WriteLine("             orderby p.Price");
        Console.WriteLine("             select p;");
        
        // EXAMPLE 3: Projection
        Console.WriteLine("\n\n--- Example 3: Projection ---\n");
        
        // Method Syntax
        var method3 = products.Select(p => new { p.Name, p.Price });
        
        // Query Syntax
        var query3 = from p in products
                    select new { p.Name, p.Price };
        
        Console.WriteLine("Both produce same result:");
        foreach (var item in method3.Take(2))
        {
            Console.WriteLine($"  {item.Name}: ${item.Price}");
        }
        
        // EXAMPLE 4: Grouping
        Console.WriteLine("\n\n--- Example 4: Grouping ---\n");
        
        // Method Syntax
        var method4 = products
            .GroupBy(p => p.Category)
            .Select(g => new { Category = g.Key, Count = g.Count() });
        
        // Query Syntax
        var query4 = from p in products
                    group p by p.Category into g
                    select new { Category = g.Key, Count = g.Count() };
        
        Console.WriteLine("Method Syntax:");
        Console.WriteLine("products.GroupBy(p => p.Category)");
        Console.WriteLine("        .Select(g => new { Category = g.Key, Count = g.Count() })");
        
        Console.WriteLine("\nQuery Syntax:");
        Console.WriteLine("from p in products");
        Console.WriteLine("group p by p.Category into g");
        Console.WriteLine("select new { Category = g.Key, Count = g.Count() }");
        
        // EXAMPLE 5: Join (Query syntax more readable!)
        Console.WriteLine("\n\n--- Example 5: Join ---\n");
        
        var categories = new List<(string Name, string Description)>
        {
            ("Electronics", "Tech gadgets"),
            ("Furniture", "Office furniture")
        };
        
        // Method Syntax (harder to read)
        var method5 = products.Join(
            categories,
            p => p.Category,
            c => c.Name,
            (p, c) => new { p.Name, Category = c.Description });
        
        // Query Syntax (more readable!)
        var query5 = from p in products
                    join c in categories on p.Category equals c.Name
                    select new { p.Name, Category = c.Description };
        
        Console.WriteLine("Query syntax is MORE READABLE for joins!");
        
        // EXAMPLE 6: Only Method Syntax (no query equivalent)
        Console.WriteLine("\n\n--- Example 6: Method-Only Operations ---\n");
        
        Console.WriteLine("These DON'T have query syntax equivalents:");
        Console.WriteLine("- Take(5)");
        Console.WriteLine("- Skip(10)");
        Console.WriteLine("- Distinct()");
        Console.WriteLine("- Reverse()");
        Console.WriteLine("- SelectMany() (partially)");
        Console.WriteLine("- Any(), All(), Contains()");
        
        var methodOnly = products
            .OrderBy(p => p.Price)
            .Take(3)
            .Select(p => p.Name);
        
        Console.WriteLine("\nExample:");
        foreach (var name in methodOnly)
        {
            Console.WriteLine($"  {name}");
        }
        
        // COMBINING BOTH SYNTAXES
        Console.WriteLine("\n\n--- Combining Both Syntaxes ---\n");
        
        var combined = (from p in products
                       where p.Category == "Electronics"
                       orderby p.Price descending
                       select p)
                       .Take(2)  // Method syntax added!
                       .Select(p => p.Name);
        
        Console.WriteLine("You can mix them:");
        Console.WriteLine("(from p in products");
        Console.WriteLine(" where p.Category == \"Electronics\"");
        Console.WriteLine(" select p).Take(2)");
    }
}
```

**When to Use Each**:
```
METHOD SYNTAX:
✅ Simple operations (Where, Select, OrderBy)
✅ Chaining multiple operations
✅ Most common in production code
✅ Required for: Take, Skip, Distinct, Any, All
✅ Better IntelliSense support

QUERY SYNTAX:
✅ Complex joins (more readable)
✅ Multiple from clauses (SelectMany)
✅ SQL-like readability
✅ Grouping operations
✅ When team prefers SQL-style
```

**Conversion Guide**:
```csharp
// WHERE
Method: items.Where(i => i.Price > 100)
Query:  from i in items where i.Price > 100 select i

// SELECT
Method: items.Select(i => i.Name)
Query:  from i in items select i.Name

// ORDERBY
Method: items.OrderBy(i => i.Price)
Query:  from i in items orderby i.Price select i

// GROUPBY
Method: items.GroupBy(i => i.Category)
Query:  from i in items group i by i.Category
```

**Bonus Challenges**:
- ⭐⭐ Convert complex query between syntaxes
- ⭐⭐ Benchmark performance differences
- ⭐⭐⭐ Build LINQ query builder UI

---

---

### Problem 112: Lambda Expressions (Func, Action, Predicate) ⭐⭐
**Concepts**: Lambda Syntax, Built-in Delegates, Anonymous Functions, Closures

**What You'll Learn**:
- Lambda expression syntax (=>)
- Func<T, TResult> for returning values
- Action<T> for void methods
- Predicate<T> for boolean returns
- Expression-bodied members
- Closures and captured variables

**Requirements**:
Master lambda expressions:
1. Basic lambda syntax
2. Func, Action, Predicate usage
3. Closures demonstration
4. LINQ with lambdas
5. Expression-bodied members

**Complete Implementation**:
```csharp
class LambdaExpressions
{
    static void Main()
    {
        Console.WriteLine("=== LAMBDA EXPRESSIONS ===\n");
        
        // BASIC LAMBDA SYNTAX
        Console.WriteLine("--- Basic Lambda Syntax ---");
        
        // No parameters
        Action sayHello = () => Console.WriteLine("Hello!");
        sayHello();
        
        // One parameter (parentheses optional)
        Action<string> greet = name => Console.WriteLine($"Hello, {name}!");
        greet("Alice");
        
        // Multiple parameters (parentheses required)
        Action<string, int> greetAge = (name, age) => 
            Console.WriteLine($"Hello, {name}! You are {age} years old.");
        greetAge("Bob", 25);
        
        // FUNC<T, TRESULT> - Returns a value
        Console.WriteLine("\n--- Func Delegates ---");
        
        // Func<int, int> = takes int, returns int
        Func<int, int> square = x => x * x;
        Console.WriteLine($"Square of 5: {square(5)}");
        
        // Func<int, int, int> = takes two ints, returns int
        Func<int, int, int> add = (a, b) => a + b;
        Console.WriteLine($"5 + 3 = {add(5, 3)}");
        
        // Func<string, int> = takes string, returns int
        Func<string, int> getLength = s => s.Length;
        Console.WriteLine($"Length of 'Hello': {getLength("Hello")}");
        
        // Multi-line lambda
        Func<int, int, string> describe = (a, b) =>
        {
            int sum = a + b;
            int product = a * b;
            return $"Sum: {sum}, Product: {product}";
        };
        Console.WriteLine(describe(4, 5));
        
        // ACTION<T> - Void methods
        Console.WriteLine("\n--- Action Delegates ---");
        
        // Action = void, no parameters
        Action simpleAction = () => Console.WriteLine("  Simple action executed");
        simpleAction();
        
        // Action<int> = void, takes int
        Action<int> printSquare = n => Console.WriteLine($"  Square of {n}: {n * n}");
        printSquare(7);
        
        // Action<string, string> = void, takes two strings
        Action<string, string> printFullName = (first, last) =>
            Console.WriteLine($"  Full name: {first} {last}");
        printFullName("John", "Doe");
        
        // PREDICATE<T> - Returns bool
        Console.WriteLine("\n--- Predicate Delegates ---");
        
        // Predicate<int> = takes int, returns bool
        Predicate<int> isEven = x => x % 2 == 0;
        Console.WriteLine($"Is 4 even? {isEven(4)}");
        Console.WriteLine($"Is 7 even? {isEven(7)}");
        
        Predicate<string> isPalindrome = s =>
        {
            string reversed = new string(s.Reverse().ToArray());
            return s.Equals(reversed, StringComparison.OrdinalIgnoreCase);
        };
        Console.WriteLine($"Is 'racecar' palindrome? {isPalindrome("racecar")}");
        
        // WORKING WITH COLLECTIONS
        Console.WriteLine("\n--- Lambdas with Collections ---");
        
        var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // Using Func
        var squares = numbers.Select(n => n * n).ToList();
        Console.WriteLine($"Squares: {string.Join(", ", squares)}");
        
        // Using Predicate (via FindAll)
        var evens = numbers.FindAll(n => n % 2 == 0);
        Console.WriteLine($"Evens: {string.Join(", ", evens)}");
        
        // Using Action (via ForEach)
        Console.Write("Printing each: ");
        numbers.ForEach(n => Console.Write($"{n} "));
        Console.WriteLine();
        
        // CLOSURES - Capturing variables
        Console.WriteLine("\n--- Closures (Variable Capture) ---");
        
        int multiplier = 10;
        Func<int, int> multiply = x => x * multiplier;  // Captures 'multiplier'
        
        Console.WriteLine($"5 * {multiplier} = {multiply(5)}");
        
        multiplier = 20;  // Change captured variable
        Console.WriteLine($"5 * {multiplier} = {multiply(5)}");  // Uses new value!
        
        // CLOSURE IN LOOP (common pitfall)
        Console.WriteLine("\n--- Closure in Loop ---");
        
        var actions = new List<Action>();
        
        // WRONG - All lambdas capture same variable
        for (int i = 0; i < 3; i++)
        {
            actions.Add(() => Console.WriteLine($"  Wrong: {i}"));
        }
        
        Console.WriteLine("Wrong way (all print 3):");
        actions.ForEach(a => a());
        
        // CORRECT - Capture loop variable
        actions.Clear();
        for (int i = 0; i < 3; i++)
        {
            int captured = i;  // Local copy
            actions.Add(() => Console.WriteLine($"  Correct: {captured}"));
        }
        
        Console.WriteLine("Correct way:");
        actions.ForEach(a => a());
        
        // EXPRESSION-BODIED MEMBERS
        Console.WriteLine("\n--- Expression-Bodied Members ---");
        
        var person = new Person { FirstName = "Jane", LastName = "Doe", Age = 30 };
        Console.WriteLine($"Full name: {person.FullName}");
        Console.WriteLine($"Is adult: {person.IsAdult}");
        person.Celebrate();
        
        // COMPARISON: Lambda vs Method
        Console.WriteLine("\n--- Lambda vs Traditional Method ---");
        
        // Traditional method
        int TraditionalSquare(int x)
        {
            return x * x;
        }
        
        // Lambda
        Func<int, int> lambdaSquare = x => x * x;
        
        Console.WriteLine($"Traditional: {TraditionalSquare(6)}");
        Console.WriteLine($"Lambda: {lambdaSquare(6)}");
        
        // PRACTICAL EXAMPLE: Sorting
        Console.WriteLine("\n--- Practical: Custom Sorting ---");
        
        var people = new List<Person>
        {
            new Person { FirstName = "Alice", LastName = "Smith", Age = 30 },
            new Person { FirstName = "Bob", LastName = "Jones", Age = 25 },
            new Person { FirstName = "Charlie", LastName = "Brown", Age = 35 }
        };
        
        // Sort by age (ascending)
        people.Sort((p1, p2) => p1.Age.CompareTo(p2.Age));
        Console.WriteLine("Sorted by age:");
        people.ForEach(p => Console.WriteLine($"  {p.FullName}: {p.Age}"));
        
        // Sort by last name (descending)
        people.Sort((p1, p2) => p2.LastName.CompareTo(p1.LastName));
        Console.WriteLine("\nSorted by last name (desc):");
        people.ForEach(p => Console.WriteLine($"  {p.FullName}"));
    }
}

class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
    
    // Expression-bodied property
    public string FullName => $"{FirstName} {LastName}";
    
    // Expression-bodied property with logic
    public bool IsAdult => Age >= 18;
    
    // Expression-bodied method
    public void Celebrate() => Console.WriteLine($"  {FullName} is celebrating!");
}
```

**Lambda Syntax Guide**:
```csharp
// NO PARAMETERS
() => expression
() => { statements; }

// ONE PARAMETER (parentheses optional)
x => x * 2
x => { return x * 2; }
(x) => x * 2  // Also valid

// MULTIPLE PARAMETERS (parentheses required)
(x, y) => x + y
(x, y) => { return x + y; }

// TYPE INFERENCE (usually not needed)
(int x, int y) => x + y  // Explicit types
```

**Built-in Delegates**:
```csharp
// Action - void, 0-16 parameters
Action              // void()
Action<T>           // void(T)
Action<T1, T2>      // void(T1, T2)

// Func - returns value, 0-16 parameters + return type
Func<TResult>       // TResult()
Func<T, TResult>    // TResult(T)
Func<T1, T2, TResult>  // TResult(T1, T2)

// Predicate - returns bool, 1 parameter
Predicate<T>        // bool(T)
// (equivalent to Func<T, bool>)
```

**Bonus Challenges**:
- ⭐⭐ Build calculator with lambda operations
- ⭐⭐ Implement strategy pattern with lambdas
- ⭐⭐⭐ Create fluent API using lambdas
- ⭐⭐⭐⭐ Expression trees and compiled lambdas

**Real-World Usage**:
- LINQ queries
- Event handlers
- Async callbacks
- Sorting and filtering
- Dependency injection configuration

---

---

### Problem 113: LINQ with Lambdas ⭐⭐
**Concepts**: LINQ + Lambda Integration, Query Composition, Functional Programming

**What You'll Learn**:
- Combining LINQ and lambda expressions
- Method chaining with lambdas
- Complex filtering with lambdas
- Query composition
- Functional programming style

**Requirements**:
Build a complete LINQ + Lambda query system:
1. Filter using lambda predicates
2. Transform using lambda selectors
3. Sort using lambda comparisons
4. Group and aggregate with lambdas

**Complete Implementation**:
```csharp
class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Department { get; set; }
    public decimal Salary { get; set; }
    public int YearsOfService { get; set; }
    public List<string> Skills { get; set; }
    
    public override string ToString() => 
        $"{Name} ({Department}) - ${Salary:N0}, {YearsOfService}yrs";
}

class LinqWithLambdas
{
    static List<Employee> GetEmployees()
    {
        return new List<Employee>
        {
            new Employee { Id = 1, Name = "Alice", Department = "Engineering", Salary = 95000, YearsOfService = 5, Skills = new List<string> { "C#", "SQL", "Azure" } },
            new Employee { Id = 2, Name = "Bob", Department = "Engineering", Salary = 85000, YearsOfService = 3, Skills = new List<string> { "C#", "JavaScript" } },
            new Employee { Id = 3, Name = "Charlie", Department = "Sales", Salary = 75000, YearsOfService = 7, Skills = new List<string> { "Negotiation", "CRM" } },
            new Employee { Id = 4, Name = "Diana", Department = "Sales", Salary = 70000, YearsOfService = 4, Skills = new List<string> { "Presentation", "Excel" } },
            new Employee { Id = 5, Name = "Eve", Department = "Engineering", Salary = 90000, YearsOfService = 6, Skills = new List<string> { "C#", "Python", "Docker" } },
            new Employee { Id = 6, Name = "Frank", Department = "Marketing", Salary = 65000, YearsOfService = 2, Skills = new List<string> { "SEO", "Analytics" } },
            new Employee { Id = 7, Name = "Grace", Department = "Engineering", Salary = 88000, YearsOfService = 4, Skills = new List<string> { "C#", "React" } }
        };
    }
    
    static void Main()
    {
        var employees = GetEmployees();
        
        Console.WriteLine("=== LINQ WITH LAMBDAS ===\n");
        
        // FILTERING WITH LAMBDA
        Console.WriteLine("--- Filter: High Earners (Salary > $80K) ---");
        var highEarners = employees.Where(e => e.Salary > 80000);
        
        foreach (var emp in highEarners)
        {
            Console.WriteLine($"  {emp}");
        }
        
        // COMPLEX FILTER
        Console.WriteLine("\n--- Complex Filter: Senior Engineers ---");
        var seniorEngineers = employees.Where(e => 
            e.Department == "Engineering" && 
            e.YearsOfService >= 4 &&
            e.Salary > 85000);
        
        foreach (var emp in seniorEngineers)
        {
            Console.WriteLine($"  {emp}");
        }
        
        // TRANSFORM WITH SELECT
        Console.WriteLine("\n--- Transform: Names Only ---");
        var names = employees.Select(e => e.Name);
        Console.WriteLine($"  {string.Join(", ", names)}");
        
        // PROJECTION TO ANONYMOUS TYPE
        Console.WriteLine("\n--- Project to Anonymous Type ---");
        var summary = employees
            .Where(e => e.Salary > 70000)
            .Select(e => new {
                e.Name,
                e.Department,
                AnnualSalary = e.Salary,
                TenureLevel = e.YearsOfService >= 5 ? "Senior" : "Junior"
            });
        
        foreach (var item in summary)
        {
            Console.WriteLine($"  {item.Name} ({item.TenureLevel}): ${item.AnnualSalary:N0}/year");
        }
        
        // SORTING WITH LAMBDA
        Console.WriteLine("\n--- Sort: By Salary (Descending) ---");
        var sorted = employees
            .OrderByDescending(e => e.Salary)
            .ThenBy(e => e.Name);
        
        foreach (var emp in sorted.Take(3))
        {
            Console.WriteLine($"  {emp}");
        }
        
        // GROUPING WITH LAMBDA
        Console.WriteLine("\n--- Group By Department ---");
        var byDepartment = employees
            .GroupBy(e => e.Department)
            .Select(g => new {
                Department = g.Key,
                Count = g.Count(),
                AvgSalary = g.Average(e => e.Salary),
                TotalPayroll = g.Sum(e => e.Salary)
            });
        
        foreach (var dept in byDepartment)
        {
            Console.WriteLine($"  {dept.Department}:");
            Console.WriteLine($"    Employees: {dept.Count}");
            Console.WriteLine($"    Avg Salary: ${dept.AvgSalary:N0}");
            Console.WriteLine($"    Total Payroll: ${dept.TotalPayroll:N0}");
        }
        
        // AGGREGATION WITH LAMBDA
        Console.WriteLine("\n--- Aggregations ---");
        
        var totalSalaries = employees.Sum(e => e.Salary);
        var avgSalary = employees.Average(e => e.Salary);
        var maxSalary = employees.Max(e => e.Salary);
        var seniorCount = employees.Count(e => e.YearsOfService >= 5);
        
        Console.WriteLine($"  Total Salaries: ${totalSalaries:N0}");
        Console.WriteLine($"  Average Salary: ${avgSalary:N0}");
        Console.WriteLine($"  Highest Salary: ${maxSalary:N0}");
        Console.WriteLine($"  Senior Employees: {seniorCount}");
        
        // ANY / ALL WITH LAMBDA
        Console.WriteLine("\n--- Any / All ---");
        
        bool hasHighEarners = employees.Any(e => e.Salary > 90000);
        Console.WriteLine($"  Has anyone earning > $90K? {hasHighEarners}");
        
        bool allPositiveSalary = employees.All(e => e.Salary > 0);
        Console.WriteLine($"  Do all have positive salary? {allPositiveSalary}");
        
        bool allSenior = employees.All(e => e.YearsOfService >= 5);
        Console.WriteLine($"  Are all employees senior (5+ years)? {allSenior}");
        
        // FIRST / SINGLE WITH LAMBDA
        Console.WriteLine("\n--- First / FirstOrDefault ---");
        
        var firstEngineer = employees.First(e => e.Department == "Engineering");
        Console.WriteLine($"  First engineer: {firstEngineer.Name}");
        
        var marketingPerson = employees.FirstOrDefault(e => e.Department == "Marketing");
        Console.WriteLine($"  Marketing person: {marketingPerson?.Name ?? "None"}");
        
        // SELECTMANY WITH LAMBDA (Flatten)
        Console.WriteLine("\n--- SelectMany: All Skills ---");
        
        var allSkills = employees
            .SelectMany(e => e.Skills)
            .Distinct()
            .OrderBy(s => s);
        
        Console.WriteLine($"  All skills: {string.Join(", ", allSkills)}");
        
        // COMPLEX QUERY CHAIN
        Console.WriteLine("\n--- Complex Query: Top Skilled Engineers ---");
        
        var topEngineers = employees
            .Where(e => e.Department == "Engineering")        // Filter department
            .Where(e => e.Skills.Contains("C#"))              // Filter by skill
            .OrderByDescending(e => e.Salary)                 // Sort by salary
            .ThenByDescending(e => e.YearsOfService)          // Then by tenure
            .Take(3)                                          // Top 3
            .Select(e => new {                                // Project
                e.Name,
                e.Salary,
                e.YearsOfService,
                SkillCount = e.Skills.Count
            });
        
        Console.WriteLine("  Top 3 C# Engineers:");
        foreach (var eng in topEngineers)
        {
            Console.WriteLine($"    {eng.Name}: ${eng.Salary:N0}, {eng.YearsOfService}yrs, {eng.SkillCount} skills");
        }
        
        // FUNCTIONAL COMPOSITION
        Console.WriteLine("\n--- Functional Composition ---");
        
        Func<Employee, bool> isEngineer = e => e.Department == "Engineering";
        Func<Employee, bool> isHighPaid = e => e.Salary > 85000;
        Func<Employee, bool> isSenior = e => e.YearsOfService >= 5;
        
        var seniorHighPaidEngineers = employees
            .Where(e => isEngineer(e) && isHighPaid(e) && isSenior(e));
        
        Console.WriteLine("  Senior, high-paid engineers:");
        foreach (var emp in seniorHighPaidEngineers)
        {
            Console.WriteLine($"    {emp.Name}");
        }
    }
}
```

**Lambda + LINQ Patterns**:
```csharp
// Filtering
items.Where(i => i.Active)
items.Where(i => i.Price > 100 && i.Stock > 0)

// Sorting
items.OrderBy(i => i.Price)
items.OrderByDescending(i => i.Date).ThenBy(i => i.Name)

// Projection
items.Select(i => i.Name)
items.Select(i => new { i.Name, i.Price })

// Grouping
items.GroupBy(i => i.Category)
     .Select(g => new { Category = g.Key, Count = g.Count() })

// Aggregation
items.Sum(i => i.Price)
items.Average(i => i.Score)
items.Max(i => i.Value)

// Existence
items.Any(i => i.IsNew)
items.All(i => i.Active)

// First/Single
items.First(i => i.Id == 5)
items.FirstOrDefault(i => i.Name == "Test")
```

**Bonus Challenges**:
- ⭐⭐ Build query builder with lambdas
- ⭐⭐⭐ Create custom LINQ operators
- ⭐⭐⭐ Implement expression tree visitor
- ⭐⭐⭐⭐ Build LINQ provider

---

---

### Problem 116: Basic Thread Creation ⭐⭐
**Concepts**: Thread, ThreadStart, Thread Lifecycle, Foreground vs Background Threads

**What You'll Learn**:
- Creating and starting threads
- Thread lifecycle (unstarted, running, stopped)
- Foreground vs background threads
- Thread.Sleep() and delays
- Thread naming and identification
- When to use threads vs tasks

**Requirements**:
Create programs demonstrating threading:
1. Create and start threads
2. Pass parameters to threads
3. Use foreground and background threads
4. Show thread interleaving
5. Demonstrate thread naming

**Complete Implementation**:
```csharp
class ThreadBasics
{
    static void Main()
    {
        Console.WriteLine("=== BASIC THREAD CREATION ===\n");
        Console.WriteLine($"Main thread ID: {Thread.CurrentThread.ManagedThreadId}\n");
        
        // METHOD 1: ThreadStart delegate (no parameters)
        Console.WriteLine("--- Method 1: ThreadStart (No Parameters) ---");
        Thread thread1 = new Thread(PrintNumbers);
        thread1.Name = "NumberPrinter";
        thread1.Start();
        
        // Main thread continues
        Console.WriteLine($"[Main] Main thread continues...");
        
        thread1.Join(); // Wait for thread1 to complete
        Console.WriteLine("[Main] Thread1 completed\n");
        
        // METHOD 2: ParameterizedThreadStart (with parameter)
        Console.WriteLine("--- Method 2: ParameterizedThreadStart ---");
        Thread thread2 = new Thread(PrintNumbersWithParam);
        thread2.Name = "ParameterizedPrinter";
        thread2.Start(5); // Pass parameter
        
        thread2.Join();
        Console.WriteLine();
        
        // METHOD 3: Lambda expression
        Console.WriteLine("--- Method 3: Lambda Expression ---");
        Thread thread3 = new Thread(() =>
        {
            for (int i = 0; i < 3; i++)
            {
                Console.WriteLine($"[Lambda Thread {Thread.CurrentThread.ManagedThreadId}] Count: {i}");
                Thread.Sleep(200);
            }
        });
        thread3.Start();
        thread3.Join();
        Console.WriteLine();
        
        // DEMONSTRATING CONCURRENT EXECUTION
        Console.WriteLine("--- Concurrent Execution ---");
        Thread t1 = new Thread(() => PrintPattern("A", 5));
        Thread t2 = new Thread(() => PrintPattern("B", 5));
        
        t1.Start();
        t2.Start();
        
        t1.Join();
        t2.Join();
        
        Console.WriteLine("\n--- Foreground vs Background Threads ---");
        
        // Foreground thread (default)
        Thread foreground = new Thread(LongRunningTask);
        foreground.Name = "Foreground";
        foreground.IsBackground = false; // Default
        
        // Background thread
        Thread background = new Thread(LongRunningTask);
        background.Name = "Background";
        background.IsBackground = true;
        
        foreground.Start();
        background.Start();
        
        Thread.Sleep(1000); // Let them run briefly
        
        Console.WriteLine("[Main] Main thread ending...");
        Console.WriteLine("[Main] Foreground thread will keep app alive");
        Console.WriteLine("[Main] Background thread will terminate when app ends\n");
        
        // Wait only for foreground (background will be killed when app ends)
        foreground.Join();
        
        // THREAD INFORMATION
        Console.WriteLine("\n--- Thread Information ---");
        Thread current = Thread.CurrentThread;
        Console.WriteLine($"Name: {current.Name ?? "(unnamed)"}");
        Console.WriteLine($"ID: {current.ManagedThreadId}");
        Console.WriteLine($"IsBackground: {current.IsBackground}");
        Console.WriteLine($"IsThreadPoolThread: {current.IsThreadPoolThread}");
        Console.WriteLine($"Priority: {current.Priority}");
        Console.WriteLine($"ThreadState: {current.ThreadState}");
    }
    
    static void PrintNumbers()
    {
        Thread current = Thread.CurrentThread;
        Console.WriteLine($"[{current.Name}] Thread ID: {current.ManagedThreadId}");
        
        for (int i = 1; i <= 5; i++)
        {
            Console.WriteLine($"[{current.Name}] {i}");
            Thread.Sleep(300); // Simulate work
        }
    }
    
    static void PrintNumbersWithParam(object maxValue)
    {
        int max = (int)maxValue;
        Thread current = Thread.CurrentThread;
        
        Console.WriteLine($"[{current.Name}] Printing up to {max}");
        
        for (int i = 1; i <= max; i++)
        {
            Console.WriteLine($"[{current.Name}] {i}");
            Thread.Sleep(200);
        }
    }
    
    static void PrintPattern(string pattern, int count)
    {
        for (int i = 0; i < count; i++)
        {
            Console.Write(pattern);
            Thread.Sleep(100);
        }
        Console.WriteLine();
    }
    
    static void LongRunningTask()
    {
        Thread current = Thread.CurrentThread;
        for (int i = 0; i < 3; i++)
        {
            Console.WriteLine($"[{current.Name}] Working... {i + 1}/3");
            Thread.Sleep(500);
        }
        Console.WriteLine($"[{current.Name}] Completed!");
    }
}
```

**Thread Lifecycle**:
```
┌─────────────┐
│ Unstarted   │  (Thread created but not started)
└──────┬──────┘
       │ .Start()
       ▼
┌─────────────┐
│  Running    │  (Thread executing)
└──────┬──────┘
       │ Completes or aborts
       ▼
┌─────────────┐
│  Stopped    │  (Thread finished)
└─────────────┘
```

**Foreground vs Background**:
```csharp
// FOREGROUND (default)
Thread fg = new Thread(Work);
fg.IsBackground = false;  // App waits for this to finish
fg.Start();

// BACKGROUND
Thread bg = new Thread(Work);
bg.IsBackground = true;   // App can exit while this runs
bg.Start();

// If main thread ends:
// - Foreground threads: App waits
// - Background threads: Terminated immediately
```

**Thread vs Task**:
```
USE THREAD WHEN:
❌ Almost never in modern C#!
✓ Only for very specific low-level scenarios
✓ When you need absolute control over thread

USE TASK WHEN:
✅ Any asynchronous operation
✅ Working with async/await
✅ Parallel processing
✅ Modern C# (preferred approach)

Rule of thumb: Use Task, not Thread!
```

**Test Cases**:
```csharp
// Create thread
Thread t = new Thread(() => Console.WriteLine("Test"));
Assert(t.ThreadState == ThreadState.Unstarted);

// Start thread
t.Start();
Assert(t.IsAlive == true);

// Wait for completion
t.Join();
Assert(t.ThreadState == ThreadState.Stopped);
```

**Bonus Challenges**:
- ⭐⭐ Create thread pool manually
- ⭐⭐ Implement thread-safe counter without lock
- ⭐⭐⭐ Build thread scheduling system
- ⭐⭐⭐ Compare thread vs task performance

**Real-World Usage**:
- Legacy code maintenance
- Low-level system programming
- Custom thread pools (rare)
- ⚠️ Modern C# uses Task instead!

**Interview Tips**:
💡 Mention: "Threads are legacy - use Task in modern C#"  
💡 Know: "Foreground threads keep app alive"  
💡 Explain: "Thread.Join() waits for completion"  
💡 Important: "Background threads terminate with app"  

---

---

### Problem 122: Task Creation & Execution ⭐⭐
**Concepts**: Task, Task.Run, Task.Factory.StartNew, Task Status, Wait

**What You'll Learn**:
- Creating tasks
- Task vs Thread
- Starting and waiting for tasks
- Task status and lifecycle
- Task.Run vs Task.Factory.StartNew
- When to use each

**Requirements**:
Create and execute tasks:
1. Basic task creation
2. Task with lambda
3. Wait for task completion
4. Check task status
5. Multiple task patterns

**Complete Implementation**:
```csharp
class TaskBasics
{
    static void Main()
    {
        Console.WriteLine("=== TASK BASICS ===\n");
        Console.WriteLine($"Main thread: {Thread.CurrentThread.ManagedThreadId}\n");
        
        // METHOD 1: Task.Run (PREFERRED)
        Console.WriteLine("--- Method 1: Task.Run ---");
        Task task1 = Task.Run(() =>
        {
            Console.WriteLine($"  Task running on thread {Thread.CurrentThread.ManagedThreadId}");
            Thread.Sleep(1000);
            Console.WriteLine("  Task completed!");
        });
        
        Console.WriteLine($"Task created. Status: {task1.Status}");
        task1.Wait();  // Wait for completion
        Console.WriteLine($"Task finished. Status: {task1.Status}\n");
        
        // METHOD 2: Task.Factory.StartNew (Advanced scenarios)
        Console.WriteLine("--- Method 2: Task.Factory.StartNew ---");
        Task task2 = Task.Factory.StartNew(() =>
        {
            Console.WriteLine($"  Factory task on thread {Thread.CurrentThread.ManagedThreadId}");
            Thread.Sleep(500);
        }, TaskCreationOptions.LongRunning); // Hint: might need dedicated thread
        
        task2.Wait();
        Console.WriteLine("Factory task completed\n");
        
        // METHOD 3: Task with named method
        Console.WriteLine("--- Method 3: Named Method ---");
        Task task3 = Task.Run(DoWork);
        task3.Wait();
        Console.WriteLine();
        
        // TASK STATUS MONITORING
        Console.WriteLine("--- Task Status Lifecycle ---");
        Task statusTask = new Task(() =>
        {
            Thread.Sleep(1000);
        });
        
        Console.WriteLine($"1. Created: {statusTask.Status}");
        
        statusTask.Start();
        Console.WriteLine($"2. Started: {statusTask.Status}");
        
        Thread.Sleep(100);
        Console.WriteLine($"3. Running: {statusTask.Status}");
        
        statusTask.Wait();
        Console.WriteLine($"4. Completed: {statusTask.Status}\n");
        
        // MULTIPLE TASKS
        Console.WriteLine("--- Multiple Tasks ---");
        
        Task[] tasks = new Task[3];
        for (int i = 0; i < 3; i++)
        {
            int taskId = i;  // Capture loop variable
            tasks[i] = Task.Run(() =>
            {
                Console.WriteLine($"  Task {taskId} started on thread {Thread.CurrentThread.ManagedThreadId}");
                Thread.Sleep(Random.Shared.Next(500, 1500));
                Console.WriteLine($"  Task {taskId} completed");
            });
        }
        
        // Wait for all tasks
        Task.WaitAll(tasks);
        Console.WriteLine("All tasks completed!\n");
        
        // TASK vs THREAD COMPARISON
        Console.WriteLine("--- Task vs Thread Comparison ---");
        
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        // Using Thread (old way)
        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++)
        {
            threads[i] = new Thread(() => Thread.Sleep(100));
            threads[i].Start();
        }
        foreach (var t in threads) t.Join();
        
        sw.Stop();
        Console.WriteLine($"Thread approach: {sw.ElapsedMilliseconds}ms");
        
        sw.Restart();
        
        // Using Task (new way)
        Task[] taskArray = new Task[10];
        for (int i = 0; i < 10; i++)
        {
            taskArray[i] = Task.Run(() => Thread.Sleep(100));
        }
        Task.WaitAll(taskArray);
        
        sw.Stop();
        Console.WriteLine($"Task approach: {sw.ElapsedMilliseconds}ms");
        Console.WriteLine("(Similar performance, but Task has many more features!)\n");
        
        // TASK PROPERTIES
        Console.WriteLine("--- Task Properties ---");
        Task infoTask = Task.Run(() =>
        {
            Thread.Sleep(500);
            return "Data";
        });
        
        Console.WriteLine($"Id: {infoTask.Id}");
        Console.WriteLine($"IsCompleted: {infoTask.IsCompleted}");
        Console.WriteLine($"IsCanceled: {infoTask.IsCanceled}");
        Console.WriteLine($"IsFaulted: {infoTask.IsFaulted}");
        Console.WriteLine($"Status: {infoTask.Status}");
        
        infoTask.Wait();
        
        Console.WriteLine($"\nAfter completion:");
        Console.WriteLine($"IsCompleted: {infoTask.IsCompleted}");
        Console.WriteLine($"Status: {infoTask.Status}");
    }
    
    static void DoWork()
    {
        Console.WriteLine($"  DoWork() on thread {Thread.CurrentThread.ManagedThreadId}");
        Thread.Sleep(800);
        Console.WriteLine("  DoWork() completed");
    }
}
```

**Task Lifecycle**:
```
┌─────────────┐
│   Created   │  (Task object created)
└──────┬──────┘
       │ .Start() or Task.Run()
       ▼
┌─────────────┐
│WaitingToRun │  (Scheduled, not yet running)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Running   │  (Executing on thread pool)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│RanToCompletion│ (Finished successfully)
└─────────────┘
```

**Task.Run vs Task.Factory.StartNew**:
```csharp
// TASK.RUN (Use this 99% of the time!)
Task.Run(() => DoWork());
// - Simple, clean syntax
// - Good defaults (uses ThreadPool)
// - Returns unwrapped Task
// - Preferred for most scenarios

// TASK.FACTORY.STARTNEW (Advanced scenarios only)
Task.Factory.StartNew(() => DoWork(),
    CancellationToken.None,
    TaskCreationOptions.LongRunning,  // Dedicated thread
    TaskScheduler.Default);
// - More control
// - Can specify scheduler
// - Can use TaskCreationOptions
// - Returns Task that might wrap another Task

// WHEN TO USE FACTORY:
// - Need TaskCreationOptions.LongRunning
// - Need custom TaskScheduler
// - Need to pass state object
// Otherwise: Use Task.Run!
```

**Waiting for Tasks**:
```csharp
// Wait for single task
task.Wait();

// Wait with timeout
bool completed = task.Wait(TimeSpan.FromSeconds(5));

// Wait for any task to complete
int index = Task.WaitAny(task1, task2, task3);

// Wait for all tasks to complete
Task.WaitAll(task1, task2, task3);

// Better: Use await (next section!)
await task;  // ✓ Preferred!
```

**Bonus Challenges**:
- ⭐⭐ Create task with custom scheduler
- ⭐⭐ Monitor task performance
- ⭐⭐⭐ Build task pool manager
- ⭐⭐⭐ Compare Task vs Thread performance

**Interview Tips**:
💡 Say: "Task is preferred over Thread in modern C#"  
💡 Know: "Task.Run uses ThreadPool internally"  
💡 Mention: "Task provides cancellation, progress, continuation"  
💡 Important: "Use Task.Run, not Task.Factory.StartNew"  

---

---

### Problem 123: Task with Return Values ⭐⭐
**Concepts**: Task<T>, Result Property, Generic Tasks, Return Values

**What You'll Learn**:
- Task<T> for returning values
- Accessing Result property
- Blocking vs non-blocking
- Multiple tasks with results
- Combining results

**Requirements**:
Work with tasks that return values:
1. Create Task<T>
2. Get result with .Result
3. Multiple tasks with different return types
4. Aggregate results
5. Handle exceptions in results

**Complete Implementation**:
```csharp
class TaskWithResults
{
    static void Main()
    {
        Console.WriteLine("=== TASK WITH RETURN VALUES ===\n");
        
        // BASIC Task<T>
        Console.WriteLine("--- Basic Task<int> ---");
        
        Task<int> task1 = Task.Run(() =>
        {
            Console.WriteLine("  Calculating sum...");
            Thread.Sleep(1000);
            return 1 + 2 + 3 + 4 + 5;
        });
        
        Console.WriteLine("Task started...");
        
        // Get result (BLOCKS until task completes!)
        int result = task1.Result;
        Console.WriteLine($"Result: {result}\n");
        
        // Task<string>
        Console.WriteLine("--- Task<string> ---");
        
        Task<string> task2 = Task.Run(() =>
        {
            Thread.Sleep(500);
            return "Hello from Task!";
        });
        
        string message = task2.Result;
        Console.WriteLine($"Message: {message}\n");
        
        // MULTIPLE TASKS WITH RESULTS
        Console.WriteLine("--- Multiple Tasks with Results ---");
        
        Task<int> add = Task.Run(() => 10 + 20);
        Task<int> multiply = Task.Run(() => 10 * 20);
        Task<int> subtract = Task.Run(() => 20 - 10);
        
        // Wait for all
        Task.WaitAll(add, multiply, subtract);
        
        Console.WriteLine($"Add: {add.Result}");
        Console.WriteLine($"Multiply: {multiply.Result}");
        Console.WriteLine($"Subtract: {subtract.Result}\n");
        
        // COMBINING RESULTS
        Console.WriteLine("--- Combining Results ---");
        
        Task<int>[] tasks = new Task<int>[]
        {
            Task.Run(() => { Thread.Sleep(500); return 10; }),
            Task.Run(() => { Thread.Sleep(300); return 20; }),
            Task.Run(() => { Thread.Sleep(700); return 30; })
        };
        
        Task.WaitAll(tasks);
        
        int sum = tasks.Sum(t => t.Result);
        Console.WriteLine($"Sum of all results: {sum}\n");
        
        // COMPLEX RETURN TYPES
        Console.WriteLine("--- Complex Return Types ---");
        
        Task<Person> personTask = Task.Run(() =>
        {
            Thread.Sleep(500);
            return new Person { Name = "Alice", Age = 30 };
        });
        
        Person person = personTask.Result;
        Console.WriteLine($"Person: {person.Name}, Age {person.Age}\n");
        
        // NAMED METHOD RETURNING VALUE
        Console.WriteLine("--- Named Method ---");
        
        Task<double> calcTask = Task.Run(CalculateAverage);
        double average = calcTask.Result;
        Console.WriteLine($"Average: {average:F2}\n");
        
        // REAL-WORLD EXAMPLE: Parallel Calculations
        Console.WriteLine("--- Parallel Calculations ---");
        
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        var factorial5 = Task.Run(() => Factorial(10));
        var factorial10 = Task.Run(() => Factorial(15));
        var factorial15 = Task.Run(() => Factorial(20));
        
        Task.WaitAll(factorial5, factorial10, factorial15);
        
        sw.Stop();
        
        Console.WriteLine($"10! = {factorial5.Result:N0}");
        Console.WriteLine($"15! = {factorial10.Result:N0}");
        Console.WriteLine($"20! = {factorial15.Result:N0}");
        Console.WriteLine($"Calculated in parallel in {sw.ElapsedMilliseconds}ms\n");
        
        // HANDLING EXCEPTIONS
        Console.WriteLine("--- Exception Handling ---");
        
        Task<int> faultyTask = Task.Run(() =>
        {
            Thread.Sleep(500);
            throw new InvalidOperationException("Something went wrong!");
            return 42;  // Never reached
        });
        
        try
        {
            int badResult = faultyTask.Result;  // Throws AggregateException!
        }
        catch (AggregateException ae)
        {
            Console.WriteLine($"Caught exception: {ae.InnerException.Message}");
        }
    }
    
    static double CalculateAverage()
    {
        int[] numbers = { 10, 20, 30, 40, 50 };
        Thread.Sleep(500);
        return numbers.Average();
    }
    
    static long Factorial(int n)
    {
        long result = 1;
        for (int i = 2; i <= n; i++)
        {
            result *= i;
        }
        Thread.Sleep(500);  // Simulate work
        return result;
    }
}

class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

**Task<T> vs Task**:
```csharp
// Task - no return value (void)
Task voidTask = Task.Run(() => 
{
    Console.WriteLine("Done");
});

// Task<T> - returns value of type T
Task<int> intTask = Task.Run(() => 
{
    return 42;
});

int result = intTask.Result;  // Get the value
```

**.Result Property** (⚠️ Warning):
```csharp
Task<int> task = Task.Run(() => SlowCalculation());

// BLOCKS current thread until task completes!
int result = task.Result;  // ⚠️ Blocks!

// Better: Use await (async method)
int result = await task;  // ✓ Doesn't block thread

// DEADLOCK WARNING:
// In UI or ASP.NET, using .Result can cause DEADLOCK!
// Always use await in async methods!
```

**Common Patterns**:
```csharp
// Parallel calculations
var task1 = Task.Run(() => Calculate1());
var task2 = Task.Run(() => Calculate2());
int total = task1.Result + task2.Result;

// Fastest result wins
var tasks = new[] {
    Task.Run(() => Method1()),
    Task.Run(() => Method2())
};
int winner = await Task.WhenAny(tasks);

// Combine multiple results
var results = await Task.WhenAll(tasks);
int sum = results.Sum();
```

**Bonus Challenges**:
- ⭐⭐ Build parallel aggregator
- ⭐⭐ Create task result cache
- ⭐⭐⭐ Implement map-reduce pattern
- ⭐⭐⭐ Build result combiner

---

---

### Problem 130: Async Method Basics ⭐⭐
**Concepts**: async Keyword, await Keyword, Task<T>, Async Method Signature

**What You'll Learn**:
- async and await keywords
- Async method naming convention
- Return types (Task, Task<T>, void)
- Async all the way down
- Common mistakes

**Requirements**:
Master async/await fundamentals:
1. Create async methods
2. Use await keyword
3. Return Task and Task<T>
4. Understand async void (don't use!)
5. Async method chaining

**Complete Implementation**:
```csharp
class AsyncBasics
{
    static async Task Main(string[] args)  // Main can be async!
    {
        Console.WriteLine("=== ASYNC/AWAIT BASICS ===\n");
        
        // BASIC ASYNC CALL
        Console.WriteLine("--- Basic Async Method ---");
        
        Console.WriteLine("Calling async method...");
        string result = await GetDataAsync();  // await = wait WITHOUT blocking thread!
        Console.WriteLine($"Result: {result}\n");
        
        // MULTIPLE ASYNC CALLS (Sequential)
        Console.WriteLine("--- Sequential Async Calls ---");
        
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        string data1 = await FetchData1Async();  // Wait for 1st
        string data2 = await FetchData2Async();  // Then 2nd
        string data3 = await FetchData3Async();  // Then 3rd
        
        sw.Stop();
        Console.WriteLine($"Sequential time: {sw.ElapsedMilliseconds}ms\n");
        
        // PARALLEL ASYNC CALLS
        Console.WriteLine("--- Parallel Async Calls ---");
        
        sw.Restart();
        
        // Start all at once (don't await yet!)
        Task<string> task1 = FetchData1Async();
        Task<string> task2 = FetchData2Async();
        Task<string> task3 = FetchData3Async();
        
        // Now await all
        await Task.WhenAll(task1, task2, task3);
        
        sw.Stop();
        Console.WriteLine($"Parallel time: {sw.ElapsedMilliseconds}ms (3x faster!)\n");
        
        // ASYNC METHOD WITH RETURN VALUE
        Console.WriteLine("--- Async with Return Value ---");
        
        int sum = await CalculateSumAsync(new[] { 1, 2, 3, 4, 5 });
        Console.WriteLine($"Sum: {sum}\n");
        
        // ASYNC METHOD CHAINING
        Console.WriteLine("--- Async Method Chaining ---");
        
        var finalResult = await DownloadAsync()
            .ContinueWith(t => ProcessAsync(t.Result))
            .Unwrap()  // Unwrap Task<Task<string>> to Task<string>
            .ContinueWith(t => SaveAsync(t.Result))
            .Unwrap();
        
        Console.WriteLine($"Chained result: {finalResult}\n");
        
        // BETTER: Just use await (cleaner!)
        Console.WriteLine("--- Cleaner with await ---");
        
        string downloaded = await DownloadAsync();
        string processed = await ProcessAsync(downloaded);
        string saved = await SaveAsync(processed);
        Console.WriteLine($"Final: {saved}\n");
        
        // EXCEPTION HANDLING
        Console.WriteLine("--- Exception Handling ---");
        
        try
        {
            await MethodThatThrowsAsync();
        }
        catch (InvalidOperationException ex)
        {
            Console.WriteLine($"Caught: {ex.Message}\n");
        }
        
        // ASYNC VOID (⚠️ DON'T USE - shown for education only)
        Console.WriteLine("--- Async Void (DON'T USE!) ---");
        Console.WriteLine("async void should ONLY be used for event handlers");
        Console.WriteLine("Problem: Can't await, can't catch exceptions properly\n");
        
        // CONFIGUREAWAIT
        Console.WriteLine("--- ConfigureAwait ---");
        
        // In library code:
        await DoWorkAsync().ConfigureAwait(false);  // Don't capture context
        Console.WriteLine("ConfigureAwait(false) used in libraries");
        Console.WriteLine("ConfigureAwait(true) or omit in UI/web apps\n");
        
        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    
    // ASYNC METHOD TEMPLATE
    static async Task<string> GetDataAsync()
    {
        Console.WriteLine("  GetDataAsync started...");
        
        // await = yield control until complete
        await Task.Delay(1000);  // Simulate async work (like HttpClient call)
        
        Console.WriteLine("  GetDataAsync completed");
        return "Sample Data";
    }
    
    static async Task<string> FetchData1Async()
    {
        Console.WriteLine("  Fetching 1...");
        await Task.Delay(1000);
        Console.WriteLine("  Fetched 1");
        return "Data 1";
    }
    
    static async Task<string> FetchData2Async()
    {
        Console.WriteLine("  Fetching 2...");
        await Task.Delay(1000);
        Console.WriteLine("  Fetched 2");
        return "Data 2";
    }
    
    static async Task<string> FetchData3Async()
    {
        Console.WriteLine("  Fetching 3...");
        await Task.Delay(1000);
        Console.WriteLine("  Fetched 3");
        return "Data 3";
    }
    
    static async Task<int> CalculateSumAsync(int[] numbers)
    {
        // Even if not truly async, return Task to be async-compatible
        await Task.Delay(100);
        return numbers.Sum();
    }
    
    static async Task<string> DownloadAsync()
    {
        Console.WriteLine("  Downloading...");
        await Task.Delay(500);
        return "downloaded.txt";
    }
    
    static async Task<string> ProcessAsync(string filename)
    {
        Console.WriteLine($"  Processing {filename}...");
        await Task.Delay(300);
        return "processed.json";
    }
    
    static async Task<string> SaveAsync(string filename)
    {
        Console.WriteLine($"  Saving {filename}...");
        await Task.Delay(200);
        return $"Saved: {filename}";
    }
    
    static async Task MethodThatThrowsAsync()
    {
        await Task.Delay(100);
        throw new InvalidOperationException("Something went wrong!");
    }
    
    static async Task DoWorkAsync()
    {
        await Task.Delay(100);
    }
}
```

**async/await Rules**:
```csharp
// ✓ CORRECT - async Task
public async Task DoWorkAsync()
{
    await Task.Delay(100);
}

// ✓ CORRECT - async Task<T>
public async Task<int> GetNumberAsync()
{
    await Task.Delay(100);
    return 42;
}

// ⚠️ AVOID - async void (only for event handlers!)
public async void ButtonClick(object sender, EventArgs e)
{
    await DoWorkAsync();  // Event handler - OK here
}

// ❌ WRONG - async without await
public async Task WrongAsync()
{
    Thread.Sleep(100);  // ❌ Should use await Task.Delay
    // Warning: Method runs synchronously!
}
```

**Naming Convention**:
```csharp
// Always suffix with "Async"
DownloadAsync()
ProcessAsync()
SaveAsync()
GetDataAsync()

// Easier to identify async methods
```

**Sequential vs Parallel**:
```csharp
// SEQUENTIAL (slow - waits for each)
var a = await Task1();  // 1 second
var b = await Task2();  // 1 second  
var c = await Task3();  // 1 second
// Total: 3 seconds

// PARALLEL (fast - all at once)
var t1 = Task1();  // Start
var t2 = Task2();  // Start
var t3 = Task3();  // Start
await Task.WhenAll(t1, t2, t3);  // Wait for all
// Total: 1 second (3x faster!)
```

**Common Mistakes**:
```csharp
// ❌ MISTAKE 1: Blocking on async
var result = GetDataAsync().Result;  // BLOCKS thread! (deadlock risk)
GetDataAsync().Wait();  // BLOCKS thread! (deadlock risk)

// ✓ CORRECT: await
var result = await GetDataAsync();  // Doesn't block ✓

// ❌ MISTAKE 2: async void
public async void ProcessAsync()  // Can't await! Can't catch exceptions!

// ✓ CORRECT: async Task
public async Task ProcessAsync()  // Can await, can catch ✓

// ❌ MISTAKE 3: Forgetting await
Task<int> task = GetNumberAsync();  // Compiles but doesn't wait!
int num = task.Result;  // Blocking!

// ✓ CORRECT:
int num = await GetNumberAsync();  // ✓
```

**Bonus Challenges**:
- ⭐⭐ Convert all Task examples to async/await
- ⭐⭐ Build async retry mechanism
- ⭐⭐⭐ Create async caching layer
- ⭐⭐⭐⭐ Implement async circuit breaker

**Real-World Usage**:
- Every web API endpoint (ASP.NET Core)
- Database calls (Entity Framework)
- HTTP requests (HttpClient)
- File I/O
- UI event handlers

**Interview Tips**:
💡 Explain: "await doesn't block thread - it yields control"  
💡 Know: "async Task<T> returns T when awaited"  
💡 Mention: "ConfigureAwait(false) in libraries"  
💡 Critical: "Never use async void except event handlers"  
💡 Common question: "Difference between Task.Result and await?"  

---

---

## Problem 151: Reverse String (3 Methods) ⭐⭐

**Problem Statement:**

Write a function that reverses a string. Implement THREE different approaches:
1. Using iteration
2. Using recursion
3. Using built-in methods

**Examples:**
```
Input: "hello"
Output: "olleh"

Input: "C#"
Output: "#C"

Input: "a"
Output: "a"

Input: ""
Output: ""
```

**Constraints:**
- 0 ≤ string.length ≤ 10⁴
- String contains ASCII characters

---

**Approach 1: Two Pointers (Iteration)**

**Concept:**
- Use two pointers: one at start, one at end
- Swap characters and move pointers toward center
- Convert string to char array first (strings are immutable)

**Complexity:**
- Time: O(n)
- Space: O(n) - for char array

**Hints:**
```csharp
char[] chars = str.ToCharArray();
int left = 0, right = chars.Length - 1;
// Swap chars[left] and chars[right]
// Move pointers
return new string(chars);
```

---

**Approach 2: Recursion**

**Concept:**
- Base case: empty or single character
- Recursive case: first char + reverse(rest of string)

**Complexity:**
- Time: O(n)
- Space: O(n) - recursion stack + string concatenation

**Hints:**
```csharp
if (str.Length <= 1) return str;
return str[str.Length - 1] + Reverse(str.Substring(0, str.Length - 1));
```

---

**Approach 3: Built-in Methods**

**Concept:**
- Use Array.Reverse() or LINQ

**Hints:**
```csharp
// Method 1: Array.Reverse
char[] arr = str.ToCharArray();
Array.Reverse(arr);

// Method 2: LINQ
new string(str.Reverse().ToArray());
```

---

**Test Cases:**
```csharp
"hello" → "olleh"
"C#" → "#C"
"a" → "a"
"" → ""
"racecar" → "racecar" (palindrome)
"12345" → "54321"
```

**Interview Tips:**
- Show you know multiple approaches
- Mention string immutability
- Discuss which method is most efficient
- Production code: use built-in methods
- Interview: show you can implement from scratch

---

---

## Problem 152: Check Anagrams ⭐⭐

**Problem Statement:**

Given two strings `s` and `t`, return true if `t` is an anagram of `s`, and false otherwise.

An anagram is a word formed by rearranging the letters of another word, using all original letters exactly once.

**Examples:**
```
Input: s = "anagram", t = "nagaram"
Output: true

Input: s = "rat", t = "car"
Output: false

Input: s = "listen", t = "silent"
Output: true
```

**Constraints:**
- 1 ≤ s.length, t.length ≤ 5 × 10⁴
- s and t consist of lowercase English letters

---

**Approach 1: Sorting**

**Concept:**
- Sort both strings
- Compare if sorted versions are equal

**Complexity:**
- Time: O(n log n)
- Space: O(1) or O(n) depending on sorting

**Hints:**
```csharp
// Convert to char arrays, sort, compare
char[] sChars = s.ToCharArray();
char[] tChars = t.ToCharArray();
Array.Sort(sChars);
Array.Sort(tChars);
// Compare arrays
```

---

**Approach 2: Frequency Map (Optimal)**

**Concept:**
- Count frequency of each character
- Both strings should have same character frequencies

**Complexity:**
- Time: O(n)
- Space: O(1) - at most 26 letters

**Hints:**
```csharp
// Use Dictionary<char, int> or int[26]
var freq = new Dictionary<char, int>();
// Count chars in s (add)
// Count chars in t (subtract)
// Check if all frequencies are 0
```

---

**Approach 3: Single Pass (Optimized)**

**Concept:**
- Use array of size 26 for lowercase letters
- Increment for first string, decrement for second
- Check if all values are zero

**Hints:**
```csharp
int[] counts = new int[26];
// For each char in s: counts[c - 'a']++
// For each char in t: counts[c - 'a']--
// Check if all counts[i] == 0
```

---

**Test Cases:**
```csharp
("anagram", "nagaram") → true
("rat", "car") → false
("listen", "silent") → true
("abc", "abcd") → false (different lengths)
("a", "a") → true
("ab", "ba") → true
```

**Common Mistakes:**
- Not checking lengths first (quick optimization)
- Case sensitivity (problem says lowercase, but ask!)
- Unicode characters (problem says English letters)

**Interview Tips:**
- Always check lengths first (O(1) optimization)
- Mention both approaches
- Sorting is simpler code but slower
- Hash map is optimal

---

---

## Problem 153: First Non-Repeating Character ⭐⭐

**Problem Statement:**

Given a string `s`, find the first non-repeating character and return its index. If it doesn't exist, return -1.

**Examples:**
```
Input: s = "leetcode"
Output: 0 (character 'l')

Input: s = "loveleetcode"
Output: 2 (character 'v')

Input: s = "aabb"
Output: -1
```

**Constraints:**
- 1 ≤ s.length ≤ 10⁵
- s consists of lowercase English letters

---

**Approach 1: Brute Force**

**Concept:**
- For each character, check if it appears elsewhere
- Return first character that doesn't repeat

**Complexity:**
- Time: O(n²)
- Space: O(1)

**Not recommended for interview!**

---

**Approach 2: Two Pass with Dictionary (Optimal)**

**Concept:**
- First pass: count frequency of each character
- Second pass: find first character with frequency 1

**Complexity:**
- Time: O(n)
- Space: O(1) - at most 26 letters

**Hints:**
```csharp
// First pass: build frequency map
var freq = new Dictionary<char, int>();
foreach (char c in s)
{
    // Count frequencies
}

// Second pass: find first with freq == 1
for (int i = 0; i < s.Length; i++)
{
    if (freq[s[i]] == 1)
        return i;
}
return -1;
```

---

**Approach 3: Array Instead of Dictionary**

**Concept:**
- Use int[26] for lowercase letters
- Faster than Dictionary for this use case

**Hints:**
```csharp
int[] counts = new int[26];
// First pass: count
// Second pass: find first with count == 1
```

---

**Test Cases:**
```csharp
"leetcode" → 0
"loveleetcode" → 2
"aabb" → -1
"z" → 0
"aabbccddeeffgghhiiz" → 18
```

**Interview Tips:**
- Explain why two passes are needed
- Mention array vs Dictionary trade-off
- Array is faster for known small character set
- Dictionary is more flexible for Unicode

---

---

## Problem 155: String Compression (aaabb → a3b2) ⭐⭐

**Problem Statement:**

Implement basic string compression using the counts of repeated characters.

For example: "aabcccccaaa" becomes "a2b1c5a3"

If the compressed string is not smaller than the original, return the original string.

**Examples:**
```
Input: "aabcccccaaa"
Output: "a2b1c5a3"

Input: "abcdef"
Output: "abcdef" (no compression)

Input: "aabbcc"
Output: "aabbcc" (compressed: "a2b2c2" is longer)

Input: "aaaa"
Output: "a4"
```

**Constraints:**
- 1 ≤ s.length ≤ 10⁴
- s consists of lowercase English letters

---

**Approach: Single Pass with StringBuilder**

**Concept:**
- Iterate through string counting consecutive characters
- When character changes, append count to result
- Compare final length with original

**Complexity:**
- Time: O(n)
- Space: O(n) - StringBuilder

**Hints:**
```csharp
var compressed = new StringBuilder();
int count = 1;

for (int i = 0; i < s.Length; i++)
{
    // If next char is same: count++
    // Else: append current char and count
    //       reset count to 1
}

// Don't forget the last character group!

// Return shorter of original or compressed
return compressed.Length < s.Length ? 
    compressed.ToString() : s;
```

---

**Test Cases:**
```csharp
"aabcccccaaa" → "a2b1c5a3"
"abcdef" → "abcdef"
"aabbcc" → "aabbcc"
"aaaa" → "a4"
"a" → "a"
"aabbccddee" → "aabbccddee"
```

**Common Mistakes:**
- Forgetting last character group
- Not comparing lengths before returning
- Using string concatenation instead of StringBuilder (slow!)

**Interview Tips:**
- Mention StringBuilder for performance
- Handle edge case: single character
- Ask: "Should single occurrences show '1'?" (Yes in this version)

---

---

## Problem 157: Valid Parentheses ⭐⭐

**Problem Statement:**

Given a string `s` containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

Valid means:
- Open brackets must be closed by the same type
- Open brackets must be closed in the correct order
- Every close bracket has a corresponding open bracket

**Examples:**
```
Input: s = "()"
Output: true

Input: s = "()[]{}"
Output: true

Input: s = "(]"
Output: false

Input: s = "([)]"
Output: false

Input: s = "{[]}"
Output: true
```

**Constraints:**
- 1 ≤ s.length ≤ 10⁴
- s consists of parentheses only '()[]{}'

---

**Approach: Stack**

**Key Insight:**
- Most recent open bracket must match next close bracket
- This is LIFO (Last In, First Out) → Stack!

**Concept:**
- Push open brackets onto stack
- For close bracket: check if matches top of stack
- At end, stack should be empty

**Complexity:**
- Time: O(n)
- Space: O(n) - stack size

**Hints:**
```csharp
var stack = new Stack<char>();

foreach (char c in s)
{
    // If opening bracket: push
    if (c == '(' || c == '{' || c == '[')
    {
        stack.Push(c);
    }
    // If closing bracket: check match
    else
    {
        if (stack.Count == 0) return false;
        
        char top = stack.Pop();
        // Check if top matches c
        if ((c == ')' && top != '(') ||
            (c == '}' && top != '{') ||
            (c == ']' && top != '['))
        {
            return false;
        }
    }
}

return stack.Count == 0;
```

---

**Optimization: Use Dictionary for Matching**

```csharp
var pairs = new Dictionary<char, char>
{
    {')', '('},
    {'}', '{'},
    {']', '['}
};
```

---

**Test Cases:**
```csharp
"()" → true
"()[]{}" → true
"(]" → false
"([)]" → false
"{[]}" → true
"(((" → false
")))" → false
"" → true (edge case)
"{[()]}" → true
```

**Common Mistakes:**
- Not checking if stack is empty before popping
- Not checking if stack is empty at the end
- Forgetting edge case: empty string

**Interview Tips:**
- Stack is THE data structure for matching problems
- Explain LIFO property
- Walk through example visually
- Mention variations: min stack, next greater element, etc.

---

---

## Problem 159: String Rotation Check ⭐⭐

**Problem Statement:**

Check if one string is a rotation of another.

Example: "waterbottle" is a rotation of "erbottlewat"

**Examples:**
```
Input: s1 = "waterbottle", s2 = "erbottlewat"
Output: true

Input: s1 = "hello", s2 = "llohe"
Output: true

Input: s1 = "hello", s2 = "world"
Output: false
```

---

**Approach 1: Brute Force**

**Concept:**
- Try all possible rotations of s1
- Check if any matches s2

**Complexity:**
- Time: O(n²)
- Space: O(n)

---

**Approach 2: Clever Trick (Optimal)**

**Key Insight:**
- If s2 is rotation of s1, then s2 is substring of s1+s1!
- Example: "waterbottle" + "waterbottle" = "waterbottlewaterbottle"
- "erbottlewat" is substring of above!

**Concept:**
- Check if lengths are equal (must be for rotation)
- Check if s2 is substring of s1 + s1

**Complexity:**
- Time: O(n)
- Space: O(n)

**Hints:**
```csharp
if (s1.Length != s2.Length) return false;
string doubled = s1 + s1;
return doubled.Contains(s2);
```

---

**Test Cases:**
```csharp
("waterbottle", "erbottlewat") → true
("hello", "llohe") → true
("hello", "world") → false
("abc", "bca") → true
("abc", "acb") → false (not rotation, different order)
```

**Interview Tips:**
- Start with brute force to show understanding
- Then present the clever trick
- Explain why it works with example
- Mention this is a common interview trick

---

---

## Problem 160: Longest Common Prefix ⭐⭐

**Problem Statement:**

Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string "".

**Examples:**
```
Input: strs = ["flower", "flow", "flight"]
Output: "fl"

Input: strs = ["dog", "racecar", "car"]
Output: ""

Input: strs = ["interspecies", "interstellar", "interstate"]
Output: "inters"
```

**Constraints:**
- 1 ≤ strs.length ≤ 200
- 0 ≤ strs[i].length ≤ 200
- strs[i] consists of lowercase English letters

---

**Approach 1: Vertical Scanning**

**Concept:**
- Compare characters at same position across all strings
- Stop when mismatch found or any string ends

**Complexity:**
- Time: O(S) where S is sum of all characters
- Space: O(1)

**Hints:**
```csharp
if (strs.Length == 0) return "";

for (int i = 0; i < strs[0].Length; i++)
{
    char c = strs[0][i];
    
    // Check this character in all strings
    for (int j = 1; j < strs.Length; j++)
    {
        // If reached end or mismatch
        if (i >= strs[j].Length || strs[j][i] != c)
        {
            return strs[0].Substring(0, i);
        }
    }
}

return strs[0]; // First string is the prefix
```

---

**Approach 2: Horizontal Scanning**

**Concept:**
- Start with first string as prefix
- For each subsequent string, reduce prefix until it matches

**Hints:**
```csharp
string prefix = strs[0];

for (int i = 1; i < strs.Length; i++)
{
    while (!strs[i].StartsWith(prefix))
    {
        prefix = prefix.Substring(0, prefix.Length - 1);
        if (prefix == "") return "";
    }
}

return prefix;
```

---

**Test Cases:**
```csharp
["flower", "flow", "flight"] → "fl"
["dog", "racecar", "car"] → ""
["interspecies", "interstellar", "interstate"] → "inters"
["a"] → "a"
["", "b"] → ""
["abc", "abc", "abc"] → "abc"
```

**Interview Tips:**
- Both approaches are valid
- Vertical scanning is more intuitive
- Handle edge cases: empty array, empty strings
- Mention early termination optimization

---

---

## Problem 162: Regex Email Validator ⭐⭐

**Problem Statement:**

Validate if a string is a valid email address using basic rules:
- Local part (before @): letters, digits, dots, hyphens, underscores
- Domain part (after @): letters, digits, dots
- At least one dot in domain
- No consecutive dots

**Examples:**
```
Input: "user@example.com"
Output: true

Input: "user.name@example.co.uk"
Output: true

Input: "user@"
Output: false

Input: "@example.com"
Output: false

Input: "user..name@example.com"
Output: false
```

---

**Approach 1: Manual Validation**

**Concept:**
- Check for single @
- Validate local part
- Validate domain part

**Hints:**
```csharp
// Split by @
string[] parts = email.Split('@');
if (parts.Length != 2) return false;

string local = parts[0];
string domain = parts[1];

// Validate local: not empty, valid chars
// Validate domain: contains dot, valid format
```

---

**Approach 2: Regex Pattern**

**Concept:**
- Use regular expression pattern

**Hints:**
```csharp
var pattern = @"^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$";
return Regex.IsMatch(email, pattern);
```

---

**Test Cases:**
```csharp
"user@example.com" → true
"user.name@example.co.uk" → true
"user@" → false
"@example.com" → false
"user..name@example.com" → false
"user@example" → false (no dot in domain)
"user name@example.com" → false (space)
```

**Interview Tips:**
- Start with manual approach
- Then show regex (if you know it)
- Mention this is simplified version
- Real email validation is VERY complex
- In production: use libraries

---

## ✅ Track A Complete!

You've covered 12 essential string algorithm problems. Practice these until you can solve them confidently without hints!

**Next**: Track B - Array Algorithms (15 problems)

---

## Problem 163: Two Sum ⭐⭐
[See previous example file - already completed]

---

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

---

