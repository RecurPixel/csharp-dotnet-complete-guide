# C# Delegates, Events & Functional Programming Quick Reference

---

## Evolution Timeline

```
C# 1.0 (2002)          C# 2.0 (2005)              C# 3.0 (2007)
    ↓                       ↓                          ↓
Delegates    →    Anonymous Methods    →    Lambda Expressions
(Verbose)         (Better)                   (Best - Modern)
```

---

## 1. Delegates (C# 1.0)

### What are Delegates?

**Delegates** = Type-safe function pointers

**Definition:** A delegate is a type that represents references to methods with a particular parameter list and return type.

**Think of it as:** A variable that holds a method.

```csharp
// Without delegate (direct call)
int result = Add(5, 3);

// With delegate (indirect call)
Operation operation = Add;
int result = operation(5, 3);
```

### Declaring Delegates

```csharp
// Syntax: delegate ReturnType DelegateName(Parameters);

// No parameters, no return
delegate void SimpleDelegate();

// With parameters, no return
delegate void PrintDelegate(string message);

// With parameters and return value
delegate int MathOperation(int x, int y);

// Generic delegate (not common - use Action/Func instead)
delegate T GenericDelegate<T>(T value);
```

### Instantiating Delegates

```csharp
// Method to reference
int Add(int x, int y)
{
    return x + y;
}

// Method 1: Constructor syntax
MathOperation operation = new MathOperation(Add);

// Method 2: Simplified syntax (C# 2.0+)
MathOperation operation = Add;

// Method 3: Lambda expression (C# 3.0+) - Most common
MathOperation operation = (x, y) => x + y;
```

### Invoking Delegates

```csharp
delegate int MathOperation(int x, int y);

int Add(int x, int y) => x + y;

MathOperation operation = Add;

// Method 1: Direct invocation
int result = operation(5, 3);  // 8

// Method 2: Invoke() method
int result = operation.Invoke(5, 3);  // 8

// Method 3: Safe invocation (null check)
int? result = operation?.Invoke(5, 3);
```

### Multicast Delegates

**Multicast delegates** can reference multiple methods.

```csharp
delegate void Logger(string message);

void LogToConsole(string message)
{
    Console.WriteLine($"Console: {message}");
}

void LogToFile(string message)
{
    Console.WriteLine($"File: {message}");
}

// Combine delegates with + or +=
Logger log = LogToConsole;
log += LogToFile;  // Now references both methods

log("Hello");
// Output:
// Console: Hello
// File: Hello

// Remove with - or -=
log -= LogToFile;

log("World");
// Output:
// Console: World
```

**Important Notes:**

- Methods are invoked in the order they were added
- If delegate returns a value, only the last method's return value is returned
- If any method throws an exception, remaining methods are not called

```csharp
delegate int Calculator(int x, int y);

int Add(int x, int y) => x + y;
int Multiply(int x, int y) => x * y;

Calculator calc = Add;
calc += Multiply;

int result = calc(5, 3);  // Returns 15 (only Multiply's result)
```

### Delegate Variance (Covariance & Contravariance)

**Covariance** - Return type can be more derived

```csharp
class Animal { }
class Dog : Animal { }

// Delegate returns Animal
delegate Animal AnimalFactory();

// But we can assign a method that returns Dog (more derived)
Dog CreateDog() => new Dog();

AnimalFactory factory = CreateDog;  // ✅ Valid (covariant)
Animal animal = factory();
```

**Contravariance** - Parameter type can be less derived

```csharp
delegate void AnimalHandler(Dog dog);

void HandleAnimal(Animal animal)
{
    Console.WriteLine("Handling animal");
}

// Can assign method that accepts Animal (less derived)
AnimalHandler handler = HandleAnimal;  // ✅ Valid (contravariant)
handler(new Dog());
```

---

## 2. Built-in Generic Delegates

### Action\<T\> - No Return Value (C# 3.0+)

**Purpose:** Encapsulates a method that takes 0-16 parameters and returns void.

```csharp
// No parameters
Action action = () => Console.WriteLine("Hello");
action();  // Hello

// One parameter
Action<string> print = message => Console.WriteLine(message);
print("World");  // World

// Two parameters
Action<string, int> printWithNumber = (text, num) => 
    Console.WriteLine($"{text}: {num}");
printWithNumber("Count", 42);  // Count: 42

// Up to 16 parameters
Action<int, int, int> action3 = (a, b, c) => 
    Console.WriteLine(a + b + c);
```

**Common Use Cases:**
```csharp
// Callbacks
void ProcessData(string data, Action<string> onComplete)
{
    // Process data...
    onComplete("Processing complete");
}

ProcessData("mydata", result => Console.WriteLine(result));

// Event handlers (simplified)
button.Click += () => Console.WriteLine("Clicked!");

// Iteration
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
numbers.ForEach(n => Console.WriteLine(n));
```

### Func<T, TResult> - With Return Value (C# 3.0+)

**Purpose:** Encapsulates a method that takes 0-16 parameters and returns a value.

```csharp
// No parameters, returns int
Func<int> getNumber = () => 42;
int result = getNumber();  // 42

// One parameter, returns result
Func<int, int> square = x => x * x;
int result = square(5);  // 25

// Two parameters, returns result
Func<int, int, int> add = (x, y) => x + y;
int result = add(5, 3);  // 8

// Three parameters (string, int, bool) returns string
Func<string, int, bool, string> format = 
    (text, num, flag) => $"{text}: {num} ({flag})";
string result = format("Value", 42, true);  // "Value: 42 (True)"

// Last type parameter is ALWAYS the return type
Func<int, string> intToString = x => x.ToString();
```

**Common Use Cases:**
```csharp
// Transformations
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
List<string> strings = numbers.Select(n => n.ToString()).ToList();

// Calculations
Func<int, int, int> calculate = (a, b) => a * b + 10;
int result = calculate(5, 3);  // 25

// Factory methods
Func<string, Person> createPerson = name => new Person { Name = name };
Person person = createPerson("John");

// LINQ (heavily uses Func)
var evens = numbers.Where(n => n % 2 == 0);  // Func<int, bool>
var doubled = numbers.Select(n => n * 2);    // Func<int, int>
```

### Predicate\<T\> - Returns bool (C# 2.0+)

**Purpose:** Represents a method that takes one parameter and returns bool.

```csharp
// Predicate<T> is equivalent to Func<T, bool>
Predicate<int> isEven = x => x % 2 == 0;
bool result = isEven(4);  // true

// Used with List<T> methods
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6 };

int first = numbers.Find(n => n > 3);           // 4
List<int> all = numbers.FindAll(n => n > 3);   // [4, 5, 6]
bool exists = numbers.Exists(n => n > 10);      // false
bool allPositive = numbers.TrueForAll(n => n > 0);  // true
```

### Comparison: Action vs Func vs Predicate

| Type | Parameters | Return Type | Example |
|------|------------|-------------|---------|
| **Action** | 0-16 | void | `Action<string> print = s => Console.WriteLine(s);` |
| **Func** | 0-16 | Any type (last param) | `Func<int, int> square = x => x * x;` |
| **Predicate** | 1 | bool | `Predicate<int> isEven = x => x % 2 == 0;` |

**When to use:**

- **Action** - Side effects, no return value needed
- **Func** - Transformations, calculations, return value needed
- **Predicate** - Testing/filtering (bool result) - mostly legacy, use `Func<T, bool>`

---

## 3. Events (C# 1.0)

### What are Events?

**Events** = Special delegates that implement the publisher-subscriber pattern.

**Purpose:** Allow a class to notify other classes when something happens.

### Event vs Delegate

```csharp
// Delegate (public, can be assigned anywhere)
public delegate void MyDelegate();
public MyDelegate myDelegate;

// Anyone can do this:
myDelegate = SomeMethod;  // ❌ Replaces all subscribers!
myDelegate();             // ❌ Can invoke from outside

// Event (restricted delegate)
public event MyDelegate MyEvent;

// Can only subscribe/unsubscribe:
MyEvent += SomeMethod;    // ✅ Subscribe
MyEvent -= SomeMethod;    // ✅ Unsubscribe
// MyEvent = SomeMethod;  // ❌ Compiler error!
// MyEvent();             // ❌ Can only invoke from declaring class
```

### EventHandler and EventHandler\<T\>

```csharp
// Built-in delegate for events
public delegate void EventHandler(object sender, EventArgs e);
public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e);

// EventArgs - base class for event data
public class EventArgs
{
    public static readonly EventArgs Empty;
}
```

### Basic Event Pattern

```csharp
// Publisher class
class Button
{
    // 1. Declare event
    public event EventHandler Click;
    
    // 2. Method to raise event
    protected virtual void OnClick(EventArgs e)
    {
        // Invoke event (null-safe)
        Click?.Invoke(this, e);
    }
    
    // 3. Method that triggers event
    public void PerformClick()
    {
        Console.WriteLine("Button clicked!");
        OnClick(EventArgs.Empty);
    }
}

// Subscriber
class Program
{
    static void Main()
    {
        Button button = new Button();
        
        // Subscribe to event
        button.Click += Button_Click;
        
        // Trigger event
        button.PerformClick();
        
        // Unsubscribe
        button.Click -= Button_Click;
    }
    
    static void Button_Click(object sender, EventArgs e)
    {
        Console.WriteLine("Event handler executed!");
    }
}
```

### Custom EventArgs

```csharp
// Custom event data
public class PriceChangedEventArgs : EventArgs
{
    public decimal OldPrice { get; }
    public decimal NewPrice { get; }
    
    public PriceChangedEventArgs(decimal oldPrice, decimal newPrice)
    {
        OldPrice = oldPrice;
        NewPrice = newPrice;
    }
}

// Publisher
class Product
{
    private decimal _price;
    
    public event EventHandler<PriceChangedEventArgs> PriceChanged;
    
    public decimal Price
    {
        get => _price;
        set
        {
            if (_price != value)
            {
                decimal oldPrice = _price;
                _price = value;
                OnPriceChanged(new PriceChangedEventArgs(oldPrice, value));
            }
        }
    }
    
    protected virtual void OnPriceChanged(PriceChangedEventArgs e)
    {
        PriceChanged?.Invoke(this, e);
    }
}

// Subscriber
Product product = new Product();
product.PriceChanged += (sender, e) => 
{
    Console.WriteLine($"Price changed from ${e.OldPrice} to ${e.NewPrice}");
};

product.Price = 100;  // Price changed from $0 to $100
product.Price = 150;  // Price changed from $100 to $150
```

### Multiple Subscribers

```csharp
class Alarm
{
    public event EventHandler AlarmTriggered;
    
    public void Trigger()
    {
        AlarmTriggered?.Invoke(this, EventArgs.Empty);
    }
}

Alarm alarm = new Alarm();

// Multiple subscribers
alarm.AlarmTriggered += (s, e) => Console.WriteLine("Security notified");
alarm.AlarmTriggered += (s, e) => Console.WriteLine("Police called");
alarm.AlarmTriggered += (s, e) => Console.WriteLine("Owner alerted");

alarm.Trigger();
// Output:
// Security notified
// Police called
// Owner alerted
```

### Event Accessors (add/remove)

```csharp
// Custom event implementation with backing delegate
class MyClass
{
    private EventHandler _myEvent;
    
    public event EventHandler MyEvent
    {
        add
        {
            Console.WriteLine("Subscriber added");
            _myEvent += value;
        }
        remove
        {
            Console.WriteLine("Subscriber removed");
            _myEvent -= value;
        }
    }
    
    public void RaiseEvent()
    {
        _myEvent?.Invoke(this, EventArgs.Empty);
    }
}
```

---

## 4. Anonymous Methods (C# 2.0)

**Anonymous methods** allow you to define methods inline without a name.

### Syntax

```csharp
delegate void PrintDelegate(string message);

// Named method (old way)
void PrintMessage(string message)
{
    Console.WriteLine(message);
}
PrintDelegate print = PrintMessage;

// Anonymous method (C# 2.0)
PrintDelegate print = delegate(string message)
{
    Console.WriteLine(message);
};

print("Hello");  // Hello

// With Action
Action<string> print = delegate(string message)
{
    Console.WriteLine(message);
};
```

### Omitting Parameters

```csharp
// If you don't use parameters, you can omit them
Action<string, int> action = delegate
{
    Console.WriteLine("No parameters used");
};
```

### Captured Variables (Closures)

```csharp
int multiplier = 10;

Func<int, int> multiply = delegate(int x)
{
    return x * multiplier;  // Captures 'multiplier'
};

Console.WriteLine(multiply(5));  // 50

multiplier = 20;
Console.WriteLine(multiply(5));  // 100 (uses updated value)
```

### Limitations

- More verbose than lambda expressions
- Cannot omit parameter types
- Cannot be converted to expression trees
- **Replaced by lambdas in C# 3.0+**

---

## 5. Lambda Expressions (C# 3.0+)

**Lambda expressions** are a concise way to write anonymous methods.

### Expression Lambda

**Single expression, implicit return**

```csharp
// Syntax: (parameters) => expression

// No parameters
Func<int> getNumber = () => 42;

// One parameter (parentheses optional)
Func<int, int> square = x => x * x;
Func<int, int> square = (x) => x * x;  // Same

// Multiple parameters (parentheses required)
Func<int, int, int> add = (x, y) => x + y;

// Examples
Func<string, int> getLength = s => s.Length;
Func<int, bool> isEven = n => n % 2 == 0;
Func<int, int, int> max = (a, b) => a > b ? a : b;
```

### Statement Lambda

**Multiple statements, explicit return**

```csharp
// Syntax: (parameters) => { statements }

Func<int, int> square = x =>
{
    int result = x * x;
    Console.WriteLine($"Squaring {x}");
    return result;
};

// With Action (no return)
Action<string> print = message =>
{
    Console.WriteLine("=====");
    Console.WriteLine(message);
    Console.WriteLine("=====");
};

// Complex logic
Func<int, int, string> compare = (a, b) =>
{
    if (a > b)
        return "First is greater";
    else if (a < b)
        return "Second is greater";
    else
        return "Equal";
};
```

### Type Inference

```csharp
// Compiler infers types
Func<int, int> square = x => x * x;  // x is inferred as int

// Explicit types (when needed)
Func<int, int> square = (int x) => x * x;

// Multiple parameters with explicit types
Func<int, double, string> format = (int num, double dec) => 
    $"Integer: {num}, Decimal: {dec}";
```

### No Parameters

```csharp
// Empty parentheses required
Func<int> getRandom = () => new Random().Next();

Action greet = () => Console.WriteLine("Hello!");
```

### Discards (C# 9.0+)

```csharp
// Unused parameters
Func<int, int, int> useFirst = (x, _) => x * 2;

// Multiple discards
Action<int, int, int> action = (_, _, _) => Console.WriteLine("Ignore all");
```

### Lambda Improvements (C# 10.0+)

**Natural Types**

```csharp
// Compiler can infer delegate type
var parse = (string s) => int.Parse(s);  // Inferred as Func<string, int>
var print = (string s) => Console.WriteLine(s);  // Inferred as Action<string>
```

**Attributes on Lambdas**

```csharp
// Apply attributes to lambda parameters
Func<string, int> parse = ([NotNull] string s) => int.Parse(s);
```

### Lambda Optional Parameters (C# 12.0+)

```csharp
// Default parameter values in lambdas
var greet = (string name = "World") => $"Hello, {name}!";

Console.WriteLine(greet());        // Hello, World!
Console.WriteLine(greet("John"));  // Hello, John!
```

### Evolution: Delegate → Anonymous Method → Lambda

```csharp
// C# 1.0: Named method + delegate
int Add(int x, int y) { return x + y; }
Func<int, int, int> add1 = Add;

// C# 2.0: Anonymous method
Func<int, int, int> add2 = delegate(int x, int y)
{
    return x + y;
};

// C# 3.0: Lambda (expression)
Func<int, int, int> add3 = (x, y) => x + y;

// C# 3.0: Lambda (statement)
Func<int, int, int> add4 = (x, y) =>
{
    return x + y;
};
```

**Always prefer lambdas in modern C#!**

---

## 6. Expression Trees (C# 3.0+)

### What are Expression Trees?

**Expression trees** represent code as data structures.

```csharp
// Regular lambda - compiled to IL
Func<int, int> square1 = x => x * x;

// Expression tree - stored as data structure
Expression<Func<int, int>> square2 = x => x * x;
```

### Why Use Expression Trees?

**LINQ to SQL/Entities** uses them to translate C# to SQL:

```csharp
// LINQ to Objects (uses Func)
var local = people.Where(p => p.Age > 18);  // Executes in C#

// LINQ to Entities (uses Expression)
var db = context.People.Where(p => p.Age > 18);  // Translates to SQL
// SELECT * FROM People WHERE Age > 18
```

### Building Expression Trees

```csharp
// Manually build expression tree
// Expression: x => x * x

ParameterExpression param = Expression.Parameter(typeof(int), "x");
BinaryExpression multiply = Expression.Multiply(param, param);
Expression<Func<int, int>> lambda = Expression.Lambda<Func<int, int>>(multiply, param);

// Compile and invoke
Func<int, int> compiled = lambda.Compile();
int result = compiled(5);  // 25
```

### Examining Expression Trees

```csharp
Expression<Func<int, bool>> expr = x => x > 5;

Console.WriteLine($"Body: {expr.Body}");              // (x > 5)
Console.WriteLine($"Parameters: {expr.Parameters[0]}"); // x
Console.WriteLine($"Type: {expr.Body.NodeType}");     // GreaterThan
```

---

## 7. Closures

### What are Closures?

**Closure** = Lambda/delegate that captures variables from outer scope.

```csharp
int multiplier = 10;

Func<int, int> multiply = x => x * multiplier;  // Captures 'multiplier'

Console.WriteLine(multiply(5));  // 50

multiplier = 20;
Console.WriteLine(multiply(5));  // 100 (uses current value!)
```

### How Closures Work

```csharp
// Compiler generates a class to hold captured variables
public Func<int, int> CreateMultiplier(int factor)
{
    // 'factor' is captured
    return x => x * factor;
}

var times2 = CreateMultiplier(2);
var times3 = CreateMultiplier(3);

Console.WriteLine(times2(5));  // 10
Console.WriteLine(times3(5));  // 15
```

### Lifetime Extension

```csharp
public Func<int> CreateCounter()
{
    int count = 0;  // Local variable
    
    return () =>
    {
        count++;  // Captured - extends lifetime
        return count;
    };
}

var counter = CreateCounter();
Console.WriteLine(counter());  // 1
Console.WriteLine(counter());  // 2
Console.WriteLine(counter());  // 3
// 'count' still alive!
```

### Common Pitfall: Loop Variable Capture

```csharp
// ❌ WRONG: All lambdas capture same variable
List<Func<int>> functions = new List<Func<int>>();

for (int i = 0; i < 5; i++)
{
    functions.Add(() => i);  // Captures 'i' (not its value)
}

foreach (var func in functions)
{
    Console.WriteLine(func());  // 5, 5, 5, 5, 5 (all print 5!)
}

// ✅ CORRECT: Capture local copy
List<Func<int>> functions = new List<Func<int>>();

for (int i = 0; i < 5; i++)
{
    int local = i;  // Create local copy
    functions.Add(() => local);  // Capture local
}

foreach (var func in functions)
{
    Console.WriteLine(func());  // 0, 1, 2, 3, 4 (correct!)
}

// ✅ ALSO CORRECT: foreach is safe (C# 5.0+)
foreach (var i in Enumerable.Range(0, 5))
{
    functions.Add(() => i);  // Safe - each iteration has new 'i'
}
```

---

## 8. Func and Action in Practice

### Higher-Order Functions

**Functions that take functions as parameters**

```csharp
// Higher-order function
void Repeat(int times, Action action)
{
    for (int i = 0; i < times; i++)
    {
        action();
    }
}

// Usage
Repeat(3, () => Console.WriteLine("Hello"));
// Hello
// Hello
// Hello

// With parameters
void Transform(int[] array, Func<int, int> transformer)
{
    for (int i = 0; i < array.Length; i++)
    {
        array[i] = transformer(array[i]);
    }
}

int[] numbers = { 1, 2, 3, 4, 5 };
Transform(numbers, x => x * 2);
// numbers is now { 2, 4, 6, 8, 10 }
```

### Callbacks

```csharp
// Asynchronous operation with callback
void DownloadData(string url, Action<string> onComplete)
{
    // Simulate download
    Thread.Sleep(1000);
    string data = "Downloaded data from " + url;
    
    onComplete(data);
}

// Usage
DownloadData("http://example.com", result =>
{
    Console.WriteLine(result);
});
```

### Method Chaining

```csharp
class Calculator
{
    private int _value;
    
    public Calculator(int initial)
    {
        _value = initial;
    }
    
    public Calculator Apply(Func<int, int> operation)
    {
        _value = operation(_value);
        return this;
    }
    
    public int Result() => _value;
}

// Usage
int result = new Calculator(10)
    .Apply(x => x + 5)
    .Apply(x => x * 2)
    .Apply(x => x - 3)
    .Result();

Console.WriteLine(result);  // 27
```

### LINQ Integration

```csharp
// LINQ heavily uses Func delegates
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Where: Func<int, bool>
var evens = numbers.Where(n => n % 2 == 0);

// Select: Func<int, string>
var strings = numbers.Select(n => n.ToString());

// OrderBy: Func<int, int>
var sorted = numbers.OrderBy(n => n);

// Aggregate: Func<int, int, int>
int sum = numbers.Aggregate((total, next) => total + next);

// All using Func/Action delegates!
```

---

## Common Patterns

### Observer Pattern (Events)

```csharp
// Subject (Observable)
class Stock
{
    private decimal _price;
    
    public event EventHandler<PriceChangedEventArgs> PriceChanged;
    
    public decimal Price
    {
        get => _price;
        set
        {
            if (_price != value)
            {
                var oldPrice = _price;
                _price = value;
                OnPriceChanged(new PriceChangedEventArgs(oldPrice, value));
            }
        }
    }
    
    protected virtual void OnPriceChanged(PriceChangedEventArgs e)
    {
        PriceChanged?.Invoke(this, e);
    }
}

// Observers
class StockMonitor
{
    public void Subscribe(Stock stock)
    {
        stock.PriceChanged += OnPriceChanged;
    }
    
    private void OnPriceChanged(object sender, PriceChangedEventArgs e)
    {
        Console.WriteLine($"Price changed: {e.OldPrice} → {e.NewPrice}");
    }
}
```

### Command Pattern (Action)

```csharp
class Button
{
    private Action _command;
    
    public void SetCommand(Action command)
    {
        _command = command;
    }
    
    public void Click()
    {
        _command?.Invoke();
    }
}

// Usage
Button saveButton = new Button();
saveButton.SetCommand(() => Console.WriteLine("Saving..."));
saveButton.Click();  // Saving...

Button cancelButton = new Button();
cancelButton.SetCommand(() => Console.WriteLine("Cancelled"));
cancelButton.Click();  // Cancelled
```

### Strategy Pattern (Func)

```csharp
class PriceCalculator
{
    public decimal Calculate(decimal price, Func<decimal, decimal> strategy)
    {
        return strategy(price);
    }
}

// Strategies
Func<decimal, decimal> noDiscount = price => price;
Func<decimal, decimal> tenPercent = price => price * 0.9m;
Func<decimal, decimal> twentyPercent = price => price * 0.8m;

PriceCalculator calc = new PriceCalculator();
decimal original = 100;

Console.WriteLine(calc.Calculate(original, noDiscount));      // 100
Console.WriteLine(calc.Calculate(original, tenPercent));      // 90
Console.WriteLine(calc.Calculate(original, twentyPercent));   // 80
```

### Factory Pattern (Func)

```csharp
class ObjectFactory<T>
{
    private Func<T> _creator;
    
    public ObjectFactory(Func<T> creator)
    {
        _creator = creator;
    }
    
    public T Create()
    {
        return _creator();
    }
}

// Usage
var personFactory = new ObjectFactory<Person>(() => new Person { Name = "Default" });
Person p1 = personFactory.Create();
Person p2 = personFactory.Create();

var randomFactory = new ObjectFactory<int>(() => new Random().Next());
int r1 = randomFactory.Create();
int r2 = randomFactory.Create();
```

---

## Comparison Tables

### Delegate Evolution

| Feature | Delegate (C# 1.0) | Anonymous Method (C# 2.0) | Lambda (C# 3.0+) |
|---------|-------------------|---------------------------|------------------|
| **Syntax** | Named method required | `delegate(params) { }` | `(params) => expr` |
| **Verbosity** | Most verbose | Verbose | Concise |
| **Type inference** | No | No | Yes |
| **Expression trees** | No | No | Yes |
| **Modern usage** | Legacy | Rare | ✅ Preferred |

### Built-in Delegates

| Delegate | Parameters | Return | Example |
|----------|------------|--------|---------|
| **Action** | 0-16 | void | `Action<string> print = s => Console.WriteLine(s);` |
| **Func** | 0-16 | Last type param | `Func<int, int> square = x => x * x;` |
| **Predicate** | 1 | bool | `Predicate<int> isEven = x => x % 2 == 0;` |

### Event vs Delegate

| Feature | Delegate | Event |
|---------|----------|-------|
| **Assignment** | Can reassign (`=`) | Can only add/remove (`+=`, `-=`) |
| **Invocation** | Can invoke from anywhere | Can only invoke from declaring class |
| **Encapsulation** | Poor | Excellent |
| **Use case** | Callbacks | Notifications |

---

## Lambda Syntax Reference

```csharp
// No parameters
() => expression
() => { statements }

// One parameter (parentheses optional for expression lambda)
x => expression
x => { statements }
(x) => expression

// Multiple parameters
(x, y) => expression
(x, y) => { statements }

// Explicit types
(int x, int y) => x + y
(string s) => s.Length

// Discard unused parameters (C# 9.0+)
(x, _) => x * 2
(_, _) => Console.WriteLine("Ignore all")

// Attributes (C# 10.0+)
([NotNull] string s) => s.Length

// Default parameters (C# 12.0+)
(int x = 10) => x * 2
```

---

## Common Pitfalls

### 1. Event Memory Leaks

```csharp
// ❌ Problem: Event holds reference, prevents garbage collection
class Publisher
{
    public event EventHandler MyEvent;
}

class Subscriber
{
    public Subscriber(Publisher publisher)
    {
        publisher.MyEvent += OnEvent;  // Creates reference
        // If Subscriber is disposed but event not unsubscribed,
        // Publisher keeps Subscriber alive!
    }
    
    private void OnEvent(object sender, EventArgs e) { }
}

// ✅ Solution: Always unsubscribe
class Subscriber : IDisposable
{
    private Publisher _publisher;
    
    public Subscriber(Publisher publisher)
    {
        _publisher = publisher;
        _publisher.MyEvent += OnEvent;
    }
    
    public void Dispose()
    {
        _publisher.MyEvent -= OnEvent;  // Unsubscribe
    }
    
    private void OnEvent(object sender, EventArgs e) { }
}
```

### 2. Loop Variable Capture (Before C# 5.0)

```csharp
// ❌ Wrong (C# 4.0 and earlier)
for (int i = 0; i < 5; i++)
{
    Task.Run(() => Console.WriteLine(i));  // All print 5!
}

// ✅ Correct
for (int i = 0; i < 5; i++)
{
    int local = i;
    Task.Run(() => Console.WriteLine(local));  // 0, 1, 2, 3, 4
}

// ✅ foreach is safe (C# 5.0+)
foreach (var i in Enumerable.Range(0, 5))
{
    Task.Run(() => Console.WriteLine(i));  // Safe!
}
```

### 3. Multicast Delegate Return Values

```csharp
// ❌ Problem: Only last return value is returned
Func<int> getNumber = () => 1;
getNumber += () => 2;
getNumber += () => 3;

int result = getNumber();  // Returns 3 (only last)

// ✅ Solution: Use Action if no return needed, or invoke separately
```

### 4. Invoking Null Delegates

```csharp
// ❌ Risk
public event EventHandler MyEvent;

public void RaiseEvent()
{
    MyEvent(this, EventArgs.Empty);  // NullReferenceException if no subscribers!
}

// ✅ Solution 1: Null check
if (MyEvent != null)
{
    MyEvent(this, EventArgs.Empty);
}

// ✅ Solution 2: Null-conditional operator (preferred)
MyEvent?.Invoke(this, EventArgs.Empty);
```

---

## Best Practices

### 1. Use Lambda Expressions

```csharp
// ❌ Old
Func<int, int> square = delegate(int x) { return x * x; };

// ✅ Modern
Func<int, int> square = x => x * x;
```

### 2. Use Built-in Delegates

```csharp
// ❌ Don't define custom delegates for simple cases
public delegate void MyCallback(string message);

// ✅ Use Action/Func
public void DoWork(Action<string> callback)
{
    callback("Done");
}
```

### 3. Use Events for Notifications

```csharp
// ❌ Exposing delegate directly
public Action<string> OnCompleted;

// ✅ Use event
public event Action<string> Completed;
```

### 4. Always Unsubscribe from Events

```csharp
// ✅ Prevent memory leaks
public class MyClass : IDisposable
{
    private Button _button;
    
    public MyClass(Button button)
    {
        _button = button;
        _button.Click += OnClick;
    }
    
    public void Dispose()
    {
        _button.Click -= OnClick;
    }
    
    private void OnClick(object sender, EventArgs e) { }
}
```

### 5. Use Expression Lambda for Simple Operations

```csharp
// ✅ Expression lambda (concise)
Func<int, int> square = x => x * x;

// Use statement lambda only when needed
Func<int, int> complex = x =>
{
    Console.WriteLine($"Processing {x}");
    int result = x * x;
    return result;
};
```

### 6. Be Careful with Closures

```csharp
// ⚠️ Be aware of captured variables
int multiplier = 10;
Func<int, int> multiply = x => x * multiplier;

Console.WriteLine(multiply(5));  // 50

multiplier = 20;
Console.WriteLine(multiply(5));  // 100 (changed!)
```

### 7. Use Null-Conditional for Events

```csharp
// ✅ Always use null-conditional
MyEvent?.Invoke(this, EventArgs.Empty);

// Not this
if (MyEvent != null)
    MyEvent(this, EventArgs.Empty);  // Race condition possible!
```

---

## Quick Reference Summary

### Delegates

- Type-safe function pointers
- Can reference multiple methods (multicast)
- Use Action/Func instead of custom delegates

### Events

- Special delegates for publisher-subscriber pattern
- Encapsulated (can't reassign or invoke externally)
- Always unsubscribe to prevent memory leaks

### Lambda Expressions

- Concise way to write anonymous methods
- Syntax: `(params) => expression` or `(params) => { statements }`
- Preferred over anonymous methods and named methods (for simple cases)

### Closures

- Lambdas/delegates that capture outer variables
- Be careful with loop variables
- Variables are captured by reference, not value

### Common Delegates

- **Action** - No return value
- **Func** - Returns value
- **Predicate** - Returns bool (legacy, use Func<T, bool>)

---

**Guide Complete!** This covers all aspects of delegates, events, and functional programming in C#. Master these concepts and you'll understand how LINQ works under the hood! 📘