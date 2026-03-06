# PHASE: BEGINNER PROBLEMS

**Total**: 28 problems

---

### Problem 1: Simple Calculator ⭐
**Concepts**: Variables, Arithmetic Operators, Switch Statement, User Input

**What You'll Learn**:
- How to declare and use variables
- Reading user input with `Console.ReadLine()`
- Using switch statements for menu logic
- Basic error handling with try-catch

**Requirements**:
1. Ask user to enter two numbers
2. Display a menu: `+`, `-`, `*`, `/`
3. Perform the selected operation
4. Display the result
5. Handle division by zero

**Bonus Challenges**:
- Add modulo `%` operation
- Validate numeric input using `int.TryParse()`
- Allow continuous calculations (loop until user exits)
- Keep a history of last 5 calculations

**Sample Output**:
```
Enter first number: 10
Enter second number: 5
Choose operation:
1. Add (+)
2. Subtract (-)
3. Multiply (*)
4. Divide (/)
Your choice: 1
Result: 10 + 5 = 15
```

---

---

### Problem 2: Grade Calculator ⭐
**Concepts**: If-Else Chains, Comparison Operators, Input Validation

**What You'll Learn**:
- Using if-else-if ladder for range checking
- Validating input boundaries
- Converting numbers to letter grades

**Requirements**:
1. Accept a numeric score (0-100)
2. Convert to letter grade:
   - 90-100: A
   - 80-89: B
   - 70-79: C
   - 60-69: D
   - Below 60: F
3. Reject invalid scores (negative or >100)

**Bonus Challenges**:
- Add plus/minus grades (A+, B-, etc.)
- Calculate GPA (A=4.0, B=3.0, etc.)
- Handle multiple students
- Calculate class average

---

---

### Problem 3: Even/Odd Checker ⭐
**Concepts**: Modulo Operator, Loops, Boolean Logic

**What You'll Learn**:
- Using `%` operator to check divisibility
- Creating a loop for repeated checks
- Breaking out of loops

**Requirements**:
1. Ask user for a number
2. Determine if even or odd
3. Display result
4. Allow checking multiple numbers

**Bonus Challenges**:
- Check for prime numbers
- Find next even/odd number
- Count even and odd numbers in a range

---

---

### Problem 4: Swap Numbers (3 Ways) ⭐
**Concepts**: Variables, Arithmetic Operations, XOR Operator

**What You'll Learn**:
- Using a temporary variable
- Arithmetic swap without temp
- Bitwise XOR swap

**Requirements**:
Implement three different methods to swap two numbers:
1. Using temporary variable
2. Using arithmetic operations (a = a + b; b = a - b; a = a - b)
3. Using XOR bitwise operation

**Bonus Challenges**:
- Compare performance of each method
- Swap three variables in one line
- Implement swap as a generic method

---

---

### Problem 5: Temperature Converter ⭐
**Concepts**: Methods, Parameters, Return Values, Formatting

**What You'll Learn**:
- Creating methods with parameters
- Returning calculated values
- String formatting with `$` interpolation

**Requirements**:
Create methods for:
1. `CelsiusToFahrenheit(double c)`
2. `FahrenheitToCelsius(double f)`
3. `CelsiusToKelvin(double c)`
4. Display results with 2 decimal places

**Bonus Challenges**:
- Create a conversion table (0-100°C)
- Add Kelvin to all other conversions
- Validate temperature ranges (Kelvin can't be negative)

---

---

### Problem 6: Number Properties Analyzer ⭐
**Concepts**: Math Operations, Multiple Conditions, Boolean Methods

**What You'll Learn**:
- Checking multiple properties of a number
- Using boolean methods
- Combining logical operators

**Requirements**:
For a given number, determine:
1. Is it even or odd?
2. Is it positive, negative, or zero?
3. Is it divisible by 3?
4. Is it divisible by 5?
5. Is it a perfect square?

**Bonus Challenges**:
- Check if it's a perfect cube
- Find all divisors
- Classify as abundant, perfect, or deficient number

---

---

### Problem 7: Simple Interest Calculator ⭐
**Concepts**: Arithmetic, Formula Application, Validation

**What You'll Learn**:
- Applying mathematical formulas
- Using decimal for financial calculations
- Validating realistic inputs

**Requirements**:
Calculate Simple Interest: `SI = (P × R × T) / 100`
1. Accept Principal (P), Rate (R), Time (T)
2. Calculate and display SI
3. Display total amount = P + SI

**Bonus Challenges**:
- Add compound interest calculation
- Compare simple vs compound interest
- Calculate monthly EMI

---

---

### Problem 8: Time Format Converter ⭐
**Concepts**: String Parsing, Validation, Formatting

**What You'll Learn**:
- Parsing time strings
- Converting between 12-hour and 24-hour formats
- Handling AM/PM

**Requirements**:
1. Accept time in 12-hour format (e.g., "2:30 PM")
2. Convert to 24-hour format (e.g., "14:30")
3. Handle edge cases (12:00 AM, 12:00 PM)

**Bonus Challenges**:
- Reverse conversion (24-hour to 12-hour)
- Add time zone conversion
- Validate time format

---

---

### Problem 9: Bill Calculator with Discount ⭐
**Concepts**: Arithmetic, Conditional Logic, Multiple Operations

**What You'll Learn**:
- Calculating percentages
- Applying conditional discounts
- Chaining calculations

**Requirements**:
1. Accept item price and quantity
2. Calculate subtotal
3. Apply discount rules:
   - 5% if subtotal > $100
   - 10% if subtotal > $500
4. Add tax (8%)
5. Display itemized bill

**Bonus Challenges**:
- Handle multiple items
- Different tax rates by category
- Apply loyalty points

---

---

### Problem 10: BMI Calculator ⭐
**Concepts**: Formula Application, Classification, Formatting

**What You'll Learn**:
- Using formulas with real-world data
- Classifying results into categories
- Displaying health-related output

**Requirements**:
1. Accept weight (kg) and height (m)
2. Calculate BMI = weight / (height²)
3. Classify:
   - < 18.5: Underweight
   - 18.5-24.9: Normal
   - 25-29.9: Overweight
   - ≥ 30: Obese

**Bonus Challenges**:
- Accept imperial units (lbs, inches)
- Calculate ideal weight range
- Track BMI over time

---

## Section 1.2: Loops & Iterations
**Focus**: Mastering for, while, do-while loops, and iteration patterns

---

### Problem 11: Multiplication Table Generator ⭐
**Concepts**: For Loop, Formatting, String Interpolation

**What You'll Learn**:
- Using for loops effectively
- String formatting for aligned output
- Nesting loops for complex patterns

**Requirements**:
1. Accept a number from user
2. Print multiplication table from 1 to 12
3. Format output nicely

**Sample Output**:
```
5 × 1 = 5
5 × 2 = 10
...
5 × 12 = 60
```

**Bonus Challenges**:
- Generate tables for multiple numbers
- Create a full 12×12 multiplication grid
- Allow custom range (e.g., 5 to 20)

---

---

### Problem 15: Factorial Calculator ⭐
**Concepts**: Loops, Accumulation, Large Numbers

**What You'll Learn**:
- Using loops for accumulation
- Handling large numbers with long
- Understanding factorial growth

**Requirements**:
1. Calculate factorial using a loop
2. Handle 0! = 1
3. Display result

**Bonus Challenges**:
- Use recursion instead
- Compare loop vs recursion performance
- Calculate using BigInteger for very large numbers

---

---

### Problem 17: Digit Sum Calculator ⭐
**Concepts**: While Loop, Integer Math

**What You'll Learn**:
- Extracting individual digits
- Using modulo and integer division
- Processing digits one by one

**Requirements**:
1. Accept a number
2. Calculate sum of its digits
3. Example: 1234 → 1+2+3+4 = 10

**Bonus Challenges**:
- Calculate product of digits
- Count number of digits
- Find largest and smallest digit

---

---

### Problem 18: Reverse a Number ⭐
**Concepts**: Loop, Math Operations

**What You'll Learn**:
- Building numbers from digits
- Reversing without strings
- Handling negative numbers

**Requirements**:
1. Reverse digits of a number
2. Example: 1234 → 4321
3. Handle negative numbers

**Bonus Challenges**:
- Check if number equals its reverse (palindrome)
- Reverse and add until palindrome
- Handle trailing zeros

---

---

### Problem 21: Right-Angled Triangle ⭐
**Concepts**: Nested Loops, Pattern Logic

**What You'll Learn**:
- Using nested loops for 2D patterns
- Controlling row and column iteration
- Building patterns incrementally

**Requirements**:
Print right-angled triangle of stars:
```
*
**
***
****
*****
```

**Bonus Challenges**:
- Inverted triangle
- Right-aligned triangle
- Hollow triangle

---

---

### Problem 27: Word Counter ⭐
**Concepts**: String Methods, Split, Loops

**What You'll Learn**:
- Using String.Split()
- Handling multiple spaces
- String array manipulation

**Requirements**:
Create method `int CountWords(string sentence)`:
1. Handle multiple spaces between words
2. Ignore leading/trailing spaces
3. Return word count

**Bonus Challenges**:
- Count unique words
- Find longest word
- Calculate average word length

---

---

### Problem 36: Car Class with Methods ⭐
**Concepts**: Class Definition, Objects, Methods

**What You'll Learn**:
- Creating a class with fields and methods
- Instantiating objects
- Calling methods on objects

**Requirements**:
Create a `Car` class with:
- Fields: `brand`, `model`, `year`, `mileage`
- Methods: `DisplayInfo()`, `Drive(int miles)`
- Create 3 car objects with different data

---

### Problem 37: Constructors Demo ⭐
**Concepts**: Default Constructor, Parameterized Constructor

**Requirements**:
Enhance the `Car` class with:
1. Default constructor (sets default values)
2. Parameterized constructor (accepts all fields)
3. Constructor overloading (partial parameters)
4. Demonstrate all three constructors

**Bonus**: Add a copy constructor

---

---

### Problem 40: Access Modifiers Demo ⭐
**Concepts**: public, private, protected, internal

**Requirements**:
Create a class demonstrating all access modifiers:
```csharp
class AccessDemo
{
    public int publicField;
    private int privateField;
    protected int protectedField;
    internal int internalField;
    
    // Methods to show accessibility
}
```

Create another class to test which fields are accessible.

---

---

### Problem 44: Animal Hierarchy ⭐
**Concepts**: Base Class, Derived Classes, Method Overriding

**Requirements**:
```csharp
class Animal
{
    public string Name { get; set; }
    public virtual void Speak() 
    { 
        Console.WriteLine("Animal makes a sound"); 
    }
}

class Dog : Animal
{
    public override void Speak() 
    { 
        Console.WriteLine($"{Name} says: Woof!"); 
    }
}

class Cat : Animal
{
    public override void Speak() 
    { 
        Console.WriteLine($"{Name} says: Meow!"); 
    }
}
```

Test polymorphic behavior with `Animal` references.

---

---

### Problem 47: Method Overloading Demo ⭐
**Concepts**: Method Overloading, Signature Differences

**Requirements**:
Create `Calculator` class with overloaded `Add` methods:
```csharp
public int Add(int a, int b)
public double Add(double a, double b)
public int Add(int a, int b, int c)
public double Add(params double[] numbers)
```

Demonstrate all variants.

---

---

### Problem 51: Sealed Class Usage ⭐
**Concepts**: sealed keyword, Preventing Inheritance

**Requirements**:
```csharp
sealed class FinalImplementation
{
    public void DoSomething() { }
}

// This should cause error:
// class Extended : FinalImplementation { }
```

Demonstrate when and why to use sealed.

---

## Section 2.3: Advanced OOP (9 Problems)

---

### Problem 61: Reverse Array In-Place ⭐
**What You'll Learn**: Array manipulation, in-place algorithms

**Requirements**:
Reverse array without creating new array:
```csharp
int[] arr = {1, 2, 3, 4, 5};
ReverseArray(arr);
// arr is now {5, 4, 3, 2, 1}
```

**Bonus**: Reverse only a portion of array

---

### Problem 62: Find Min/Max in Array ⭐

---

### Problem 71: List vs Array Comparison ⭐
**What You'll Learn**: When to use List<T> vs arrays

**Requirements**:
Create side-by-side comparison showing:
1. Dynamic sizing (List wins)
2. Performance (Array wins for known size)
3. Methods available (List wins)
4. Memory efficiency (Array wins)

---

### Problem 72: Dynamic List Operations ⭐

---

### Problem 94: Static Utility Library ⭐
**Concepts**: Static Classes, Static Methods, Math Utilities, Encapsulation

**What You'll Learn**:
- Creating static classes
- When to use static vs instance
- Static utility methods
- Math helper libraries
- Static constructors

**Requirements**:
Create a static MathHelper class with:
1. Common math operations
2. Validation methods
3. Conversion utilities
4. Cannot be instantiated

**Complete Implementation**:
```csharp
// Static class - cannot be instantiated
static class MathHelper
{
    // Static readonly field - shared across all usage
    private static readonly Random random = new Random();
    
    // Static constructor - runs once before first use
    static MathHelper()
    {
        Console.WriteLine("[MathHelper initialized]");
    }
    
    // BASIC OPERATIONS
    public static int Square(int number)
    {
        return number * number;
    }
    
    public static int Cube(int number)
    {
        return number * number * number;
    }
    
    public static double Power(double baseNumber, int exponent)
    {
        if (exponent == 0) return 1;
        
        double result = 1;
        for (int i = 0; i < Math.Abs(exponent); i++)
        {
            result *= baseNumber;
        }
        
        return exponent > 0 ? result : 1 / result;
    }
    
    // VALIDATION
    public static bool IsPrime(int number)
    {
        if (number < 2) return false;
        if (number == 2) return true;
        if (number % 2 == 0) return false;
        
        int sqrt = (int)Math.Sqrt(number);
        for (int i = 3; i <= sqrt; i += 2)
        {
            if (number % i == 0)
                return false;
        }
        
        return true;
    }
    
    public static bool IsEven(int number)
    {
        return number % 2 == 0;
    }
    
    public static bool IsOdd(int number)
    {
        return number % 2 != 0;
    }
    
    public static bool IsPerfectSquare(int number)
    {
        if (number < 0) return false;
        int sqrt = (int)Math.Sqrt(number);
        return sqrt * sqrt == number;
    }
    
    // RANGE OPERATIONS
    public static int Clamp(int value, int min, int max)
    {
        if (value < min) return min;
        if (value > max) return max;
        return value;
    }
    
    public static bool InRange(int value, int min, int max)
    {
        return value >= min && value <= max;
    }
    
    // RANDOM UTILITIES
    public static int RandomInt(int min, int max)
    {
        return random.Next(min, max + 1);
    }
    
    public static double RandomDouble()
    {
        return random.NextDouble();
    }
    
    // CONVERSION
    public static double CelsiusToFahrenheit(double celsius)
    {
        return (celsius * 9 / 5) + 32;
    }
    
    public static double FahrenheitToCelsius(double fahrenheit)
    {
        return (fahrenheit - 32) * 5 / 9;
    }
    
    // STATISTICS
    public static double Average(params int[] numbers)
    {
        if (numbers.Length == 0)
            throw new ArgumentException("Need at least one number");
        
        return numbers.Average();
    }
    
    public static int Max(params int[] numbers)
    {
        if (numbers.Length == 0)
            throw new ArgumentException("Need at least one number");
        
        return numbers.Max();
    }
    
    public static int Min(params int[] numbers)
    {
        if (numbers.Length == 0)
            throw new ArgumentException("Need at least one number");
        
        return numbers.Min();
    }
    
    // FACTORIAL
    public static long Factorial(int n)
    {
        if (n < 0)
            throw new ArgumentException("Factorial undefined for negative numbers");
        if (n == 0 || n == 1)
            return 1;
        
        long result = 1;
        for (int i = 2; i <= n; i++)
        {
            result *= i;
        }
        
        return result;
    }
    
    // GCD & LCM
    public static int GCD(int a, int b)
    {
        while (b != 0)
        {
            int temp = b;
            b = a % b;
            a = temp;
        }
        return Math.Abs(a);
    }
    
    public static int LCM(int a, int b)
    {
        return Math.Abs(a * b) / GCD(a, b);
    }
}

// Demonstration
class MathHelperDemo
{
    static void Main()
    {
        Console.WriteLine("=== STATIC UTILITY LIBRARY DEMO ===\n");
        
        // Cannot instantiate static class
        // MathHelper helper = new MathHelper(); // COMPILE ERROR!
        
        // Use static methods directly
        Console.WriteLine("Basic Operations:");
        Console.WriteLine($"  Square(5) = {MathHelper.Square(5)}");
        Console.WriteLine($"  Cube(3) = {MathHelper.Cube(3)}");
        Console.WriteLine($"  Power(2, 10) = {MathHelper.Power(2, 10)}");
        
        Console.WriteLine("\nValidation:");
        int[] testNumbers = { 2, 7, 15, 16, 17, 25 };
        foreach (int num in testNumbers)
        {
            Console.WriteLine($"  {num}: Prime={MathHelper.IsPrime(num)}, " +
                            $"Even={MathHelper.IsEven(num)}, " +
                            $"Perfect Square={MathHelper.IsPerfectSquare(num)}");
        }
        
        Console.WriteLine("\nRange Operations:");
        Console.WriteLine($"  Clamp(15, 0, 10) = {MathHelper.Clamp(15, 0, 10)}");
        Console.WriteLine($"  InRange(5, 0, 10) = {MathHelper.InRange(5, 0, 10)}");
        
        Console.WriteLine("\nRandom:");
        Console.WriteLine($"  Random int (1-100): {MathHelper.RandomInt(1, 100)}");
        Console.WriteLine($"  Random double: {MathHelper.RandomDouble():F4}");
        
        Console.WriteLine("\nConversion:");
        Console.WriteLine($"  100°F = {MathHelper.FahrenheitToCelsius(100):F1}°C");
        Console.WriteLine($"  0°C = {MathHelper.CelsiusToFahrenheit(0):F1}°F");
        
        Console.WriteLine("\nStatistics (params):");
        Console.WriteLine($"  Average(1,2,3,4,5) = {MathHelper.Average(1, 2, 3, 4, 5)}");
        Console.WriteLine($"  Max(5,2,9,1,7) = {MathHelper.Max(5, 2, 9, 1, 7)}");
        
        Console.WriteLine("\nAdvanced:");
        Console.WriteLine($"  Factorial(5) = {MathHelper.Factorial(5)}");
        Console.WriteLine($"  GCD(48, 18) = {MathHelper.GCD(48, 18)}");
        Console.WriteLine($"  LCM(12, 18) = {MathHelper.LCM(12, 18)}");
    }
}
```

**Static vs Instance Comparison**:
```csharp
// INSTANCE CLASS (needs object)
class Calculator
{
    private int memory; // Instance field
    
    public int Add(int a, int b) // Instance method
    {
        return a + b;
    }
}

// Usage:
Calculator calc = new Calculator(); // Must create instance
int result = calc.Add(5, 3);

// STATIC CLASS (no object needed)
static class MathHelper
{
    public static int Add(int a, int b) // Static method
    {
        return a + b;
    }
}

// Usage:
int result = MathHelper.Add(5, 3); // Direct access ✓
```

**When to Use Static**:
✅ Utility methods (no state needed)  
✅ Math libraries  
✅ Extension methods  
✅ Factory methods  
✅ Configuration helpers  

**When NOT to Use Static**:
❌ Need to maintain state  
❌ Need polymorphism  
❌ Need inheritance  
❌ Testing is harder (can't mock easily)  

**Bonus Challenges**:
- ⭐⭐ Add string utility methods (static class StringHelper)
- ⭐⭐ Add validation utilities (static class Validator)
- ⭐⭐⭐ Create extension methods using static class
- ⭐⭐⭐ Add thread-safe singleton using static

---

---

### Problem 95: Partial Class Implementation ⭐
**Concepts**: Partial Classes, Code Organization, File Splitting, Auto-Generated Code

**What You'll Learn**:
- Splitting class definition across files
- Organizing large classes
- Working with auto-generated code
- Partial methods
- When partial classes are useful

**Requirements**:
Create an Employee class split across multiple files:
1. File 1: Properties and basic info
2. File 2: Business logic and calculations
3. Demonstrate they compile into single class

**Implementation - File 1 (Employee.Properties.cs)**:
```csharp
// Employee.Properties.cs - Properties and data
partial class Employee
{
    // Properties
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Department { get; set; }
    public decimal BaseSalary { get; set; }
    public DateTime HireDate { get; set; }
    
    // Computed property
    public string FullName => $"{FirstName} {LastName}";
    
    // Constructor
    public Employee(int id, string firstName, string lastName)
    {
        Id = id;
        FirstName = firstName;
        LastName = lastName;
        HireDate = DateTime.Now;
    }
    
    // Partial method declaration (no implementation here)
    partial void OnSalaryChanged(decimal oldSalary, decimal newSalary);
}
```

**Implementation - File 2 (Employee.BusinessLogic.cs)**:
```csharp
// Employee.BusinessLogic.cs - Business logic and calculations
partial class Employee
{
    // Business logic methods
    public decimal CalculateAnnualSalary()
    {
        return BaseSalary * 12;
    }
    
    public decimal CalculateBonus(double percentage)
    {
        return BaseSalary * (decimal)percentage;
    }
    
    public int GetYearsOfService()
    {
        return DateTime.Now.Year - HireDate.Year;
    }
    
    public void GiveRaise(decimal percentage)
    {
        decimal oldSalary = BaseSalary;
        BaseSalary += BaseSalary * percentage;
        
        // Call partial method (if implemented)
        OnSalaryChanged(oldSalary, BaseSalary);
    }
    
    // Partial method implementation (optional)
    partial void OnSalaryChanged(decimal oldSalary, decimal newSalary)
    {
        Console.WriteLine($"Salary changed: ${oldSalary:F2} → ${newSalary:F2}");
    }
    
    // Display method
    public void DisplayInfo()
    {
        Console.WriteLine($"Employee: {FullName} (ID: {Id})");
        Console.WriteLine($"  Department: {Department}");
        Console.WriteLine($"  Base Salary: ${BaseSalary:F2}");
        Console.WriteLine($"  Annual Salary: ${CalculateAnnualSalary():F2}");
        Console.WriteLine($"  Years of Service: {GetYearsOfService()}");
    }
}
```

**Demonstration**:
```csharp
class PartialClassDemo
{
    static void Main()
    {
        Console.WriteLine("=== PARTIAL CLASS DEMO ===\n");
        
        // Create employee - both partial parts work together!
        Employee emp = new Employee(1, "Alice", "Johnson")
        {
            Department = "Engineering",
            BaseSalary = 5000m
        };
        
        emp.DisplayInfo();
        
        // Call method from File 2
        Console.WriteLine($"\n10% Bonus: ${emp.CalculateBonus(0.10):F2}");
        
        // Trigger partial method
        Console.WriteLine("\nGiving 15% raise:");
        emp.GiveRaise(0.15m);
        
        emp.DisplayInfo();
    }
}
```

**When to Use Partial Classes**:

✅ **Large Classes**: Split into logical files
```
Employee.Properties.cs  - Properties
Employee.Business.cs    - Business logic
Employee.Validation.cs  - Validation methods
```

✅ **Auto-Generated Code**: Keep generated code separate
```
Form.Designer.cs  - Auto-generated by designer
Form.cs           - Your custom code
```

✅ **Team Collaboration**: Different files for different developers
```
Customer.Data.cs      - Developer A
Customer.Analytics.cs - Developer B
```

✅ **Separation of Concerns**:
```
Database.Connection.cs  - Connection logic
Database.Queries.cs     - Query methods
Database.Transactions.cs - Transaction handling
```

**Partial Methods**:
```csharp
// File 1: Declaration (no body)
partial void OnValidate();

// File 2: Implementation (optional)
partial void OnValidate()
{
    // Validation logic
}

// If not implemented, compiler removes all calls to it!
```

**Real-World Example - WinForms**:
```csharp
// Form1.Designer.cs (auto-generated - DON'T EDIT!)
partial class Form1
{
    private Button button1;
    private Label label1;
    
    private void InitializeComponent()
    {
        // Auto-generated designer code
    }
}

// Form1.cs (your code)
partial class Form1
{
    public Form1()
    {
        InitializeComponent();
    }
    
    private void button1_Click(object sender, EventArgs e)
    {
        // Your event handling code
    }
}
```

**Test Cases**:
```csharp
// Both parts work together
Employee e = new Employee(1, "Test", "User");
Assert(e.FullName == "Test User"); // Property from File 1
Assert(e.GetYearsOfService() >= 0); // Method from File 2
```

**Bonus Challenges**:
- ⭐ Split into 3 files (add Validation.cs)
- ⭐⭐ Use partial methods for logging
- ⭐⭐ Create partial interface (advanced C# 13)

**Interview Tips**:
💡 Mention: "Partial classes are used in auto-generated code (WinForms, WPF)"  
💡 Explain: "All parts must have 'partial' keyword"  
💡 Know limitation: "All parts must be in same assembly and namespace"  

---

---

