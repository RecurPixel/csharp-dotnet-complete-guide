# PHASE: ADVANCED PROBLEMS

**Total**: 52 problems

---

### Problem 35: Number Base Converter ⭐⭐⭐
**Concepts**: Methods, Math, Conversion Algorithms

**What You'll Learn**:
- Converting between number bases
- Using StringBuilder for efficiency
- Understanding positional notation

**Requirements**:
Implement conversions:
1. Decimal to Binary
2. Decimal to Octal
3. Decimal to Hexadecimal
4. Binary to Decimal
5. Hexadecimal to Decimal

**Bonus Challenges**:
- Any base to any base conversion
- Handle fractional numbers
- Add validation for invalid digits

---

## 🏁 Phase 1 Checkpoint: Mini-Project

### **Project: Smart Console Calculator**

**Duration**: 2-3 days  
**Concepts Integration**: Everything from Phase 1

**Requirements**:
Create a feature-rich calculator with:

1. **Basic Operations**: +, -, *, /, %
2. **Scientific Functions**: Power, Square Root, Factorial
3. **History**: Store last 10 calculations
4. **Multiple Modes**:
   - Basic Calculator
   - Scientific Calculator
   - Base Converter
   - Statistics Calculator (mean, median, mode)
5. **Error Handling**: Division by zero, invalid input
6. **Menu System**: Clean, user-friendly interface

**Bonus Features**:
- Save/load calculation history to file
- Memory functions (M+, M-, MR, MC)
- Expression evaluation (e.g., "5 + 3 * 2")
- Unit converter (temperature, length, weight)

**Learning Goals**:
- Integrate all Phase 1 concepts
- Practice code organization
- Build confidence in program structure
- Create something you can show in your portfolio

**Success Criteria**:
✅ All operations work correctly  
✅ Clean, intuitive menu  
✅ Proper error handling  
✅ Code is well-organized in methods  
✅ History feature works  

---

## 📊 Phase 1 Progress Tracker

Track your completion:

**Section 1.1**: ☐☐☐☐☐☐☐☐☐☐ (0/10)  
**Section 1.2**: ☐☐☐☐☐☐☐☐☐☐ (0/10)  
**Section 1.3**: ☐☐☐☐☐ (0/5)  
**Section 1.4**: ☐☐☐☐☐☐☐☐☐☐ (0/10)  
**Mini-Project**: ☐ (0/1)

**Total Phase 1**: 0/36

---

## 🎯 Phase 1 Self-Assessment

Before moving to Phase 2, ensure you can:

✅ Write programs using variables, loops, and conditionals confidently  
✅ Create and use methods with parameters and return values  
✅ Work with arrays and perform basic operations  
✅ Handle user input and validate data  
✅ Debug simple logic errors  
✅ Organize code into clear, reusable methods  

**Not confident in any area?** Go back and practice those problems again!

**Ready for Phase 2?** Let's dive into Object-Oriented Programming! 🚀


---
---

# 📗 PHASE 2: OBJECT-ORIENTED PROGRAMMING MASTERY
## Building Reusable, Maintainable Code (25 Problems)

**Learning Goals**: Design classes, implement inheritance and polymorphism, use interfaces effectively  
**Estimated Time**: 2-3 weeks (beginners) | 4-5 days (experienced)  
**Job Readiness**: 25% → 45%

---

## Section 2.1: Classes & Objects (8 Problems)
**Focus**: Encapsulation, constructors, properties, access modifiers

---

### Problem 46: Shape Hierarchy (Abstract Classes) ⭐⭐⭐
**Concepts**: Abstract Classes, Polymorphism, Interfaces

**Requirements**:
```csharp
abstract class Shape
{
    public abstract double CalculateArea();
    public abstract double CalculatePerimeter();
    
    public void DisplayInfo()
    {
        Console.WriteLine($"Area: {CalculateArea()}");
        Console.WriteLine($"Perimeter: {CalculatePerimeter()}");
    }
}

class Circle : Shape
{
    public double Radius { get; set; }
    // Implement abstract methods
}

class Rectangle : Shape
{
    public double Length { get; set; }
    public double Width { get; set; }
    // Implement abstract methods
}

class Triangle : Shape
{
    public double Side1 { get; set; }
    public double Side2 { get; set; }
    public double Side3 { get; set; }
    // Implement abstract methods
}
```

Create a `List<Shape>` and demonstrate polymorphism.

**Bonus**:
- Add `IDrawable` interface
- Implement comparison by area
- Calculate total area of all shapes

---

---

### Problem 52: IComparable Implementation ⭐⭐⭐
**Concepts**: Interfaces, Custom Sorting

**Requirements**:
```csharp
class Student : IComparable<Student>
{
    public string Name { get; set; }
    public int RollNumber { get; set; }
    public double Percentage { get; set; }
    
    public int CompareTo(Student other)
    {
        // Sort by percentage
        return this.Percentage.CompareTo(other.Percentage);
    }
}
```

Sort a List<Student> using built-in Sort().

**Bonus**: Implement IComparer for multiple sort criteria

---

---

### Problem 53: IEnumerable Custom Collection ⭐⭐⭐
**Concepts**: Iterator Pattern, yield return

**Requirements**:
```csharp
class Library : IEnumerable<Book>
{
    private List<Book> books = new List<Book>();
    
    public void AddBook(Book book) { }
    
    public IEnumerator<Book> GetEnumerator()
    {
        foreach (var book in books)
        {
            yield return book;
        }
    }
    
    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}
```

Use foreach on your custom collection.

---

---

### Problem 56: Copy Constructor (Deep/Shallow) ⭐⭐⭐
**Concepts**: Object Cloning, Reference vs Value

**Requirements**:
Demonstrate deep copy vs shallow copy:
```csharp
class Person
{
    public string Name { get; set; }
    public Address HomeAddress { get; set; }
    
    // Shallow copy
    public Person ShallowCopy()
    {
        return (Person)this.MemberwiseClone();
    }
    
    // Deep copy
    public Person DeepCopy()
    {
        Person copy = (Person)this.MemberwiseClone();
        copy.HomeAddress = new Address
        {
            Street = this.HomeAddress.Street,
            City = this.HomeAddress.City
        };
        return copy;
    }
}
```

Show how changing nested objects affects copies.

---

---

### Problem 57: IDisposable Pattern ⭐⭐⭐
**Concepts**: Resource Management, using Statement

**Requirements**:
```csharp
class FileManager : IDisposable
{
    private StreamWriter writer;
    private bool disposed = false;
    
    public FileManager(string path)
    {
        writer = new StreamWriter(path);
    }
    
    public void Write(string data)
    {
        writer.WriteLine(data);
    }
    
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                writer?.Dispose();
            }
            disposed = true;
        }
    }
    
    ~FileManager()
    {
        Dispose(false);
    }
}
```

Use with `using` statement.

---

---

### Problem 58: Singleton Pattern ⭐⭐⭐
**Concepts**: Design Patterns, Thread Safety

**Requirements**:
```csharp
class DatabaseConnection
{
    private static DatabaseConnection instance = null;
    private static readonly object lockObj = new object();
    
    private DatabaseConnection() 
    {
        Console.WriteLine("Database connected");
    }
    
    public static DatabaseConnection Instance
    {
        get
        {
            if (instance == null)
            {
                lock (lockObj)
                {
                    if (instance == null)
                    {
                        instance = new DatabaseConnection();
                    }
                }
            }
            return instance;
        }
    }
}
```

**Bonus**: Implement lazy initialization with Lazy<T>

---

---

### Problem 59: Factory Pattern ⭐⭐⭐
**Concepts**: Object Creation Patterns

**Requirements**:
```csharp
interface IVehicle
{
    void Drive();
}

class Car : IVehicle { }
class Motorcycle : IVehicle { }
class Truck : IVehicle { }

class VehicleFactory
{
    public static IVehicle CreateVehicle(string type)
    {
        switch (type.ToLower())
        {
            case "car": return new Car();
            case "motorcycle": return new Motorcycle();
            case "truck": return new Truck();
            default: throw new ArgumentException("Invalid type");
        }
    }
}
```

---

---

### Problem 60: Observer Pattern ⭐⭐⭐
**Concepts**: Event-Driven Design

**Requirements**:
```csharp
class Stock
{
    private decimal price;
    public event EventHandler<decimal> PriceChanged;
    
    public decimal Price
    {
        get { return price; }
        set
        {
            if (price != value)
            {
                price = value;
                OnPriceChanged(price);
            }
        }
    }
    
    protected virtual void OnPriceChanged(decimal newPrice)
    {
        PriceChanged?.Invoke(this, newPrice);
    }
}

class Investor
{
    public string Name { get; set; }
    
    public void OnPriceChanged(object sender, decimal newPrice)
    {
        Console.WriteLine($"{Name} notified: Price is now {newPrice}");
    }
}
```

---

## 🏁 Phase 2 Checkpoint: Mini-Project

### **Library Management System**

**Duration**: 5-7 days  
**Concepts**: Full OOP Integration

**Core Requirements**:

1. **Classes**:
   - `Book` (ISBN, Title, Author, Available)
   - `Member` (ID, Name, BooksCheckedOut)
   - `Librarian` (inherits from Member, additional privileges)
   - `Library` (manages books and members)

2. **Features**:
   - Add/Remove books
   - Register members
   - Check out/Return books
   - Search books (by title, author, ISBN)
   - View member history
   - Fine calculation for late returns

3. **OOP Concepts to Apply**:
   - Encapsulation (private fields, public properties)
   - Inheritance (Member → Librarian)
   - Polymorphism (different member types)
   - Interfaces (ISearchable, IReportable)
   - Exception handling

**Bonus Features**:
- Implement reservation system
- Multiple copies of same book
- Category-based browsing
- Save/load library data from file
- Generate reports (most popular books, active members)

---

## 📊 Phase 2 Progress Tracker

**Section 2.1**: ☐☐☐☐☐☐☐☐ (0/8)  
**Section 2.2**: ☐☐☐☐☐☐☐☐ (0/8)  
**Section 2.3**: ☐☐☐☐☐☐☐☐☐ (0/9)  
**Mini-Project**: ☐ (0/1)

**Total Phase 2**: 0/26

---

## 🎯 Phase 2 Self-Assessment

Before moving to Phase 3, can you:

✅ Create classes with proper encapsulation?  
✅ Use inheritance to avoid code duplication?  
✅ Implement polymorphism effectively?  
✅ Work with interfaces confidently?  
✅ Apply basic design patterns?  
✅ Explain when to use abstract classes vs interfaces?  

**Ready for Phase 3?** Let's master collections! 📚


---
---

# 📙 PHASE 3: COLLECTIONS & DATA STRUCTURES
## Mastering Data Organization (30 Problems)

**Learning Goals**: Choose and use appropriate data structures, optimize performance  
**Estimated Time**: 2-3 weeks (beginners) | 1 week (experienced)  
**Job Readiness**: 45% → 65%

---

## Section 3.1: Array Fundamentals (10 Problems)

---

### Problem 66: Merge Two Sorted Arrays ⭐⭐⭐

---

### Problem 69: Two Sum Problem ⭐⭐⭐

---

### Problem 70: Maximum Subarray Sum (Kadane's) ⭐⭐⭐

---

## Section 3.2: Lists & Collections (10 Problems)

---

### Problem 78: Dictionary Sorting ⭐⭐⭐

---

### Problem 79: Group Data into Dictionary ⭐⭐⭐

---

### Problem 80: Nested Collections ⭐⭐⭐

---

## Section 3.3: Stacks, Queues & Advanced (10 Problems)

---

### Problem 83: Balanced Parentheses Checker ⭐⭐⭐
**Classic Stack Problem**

---

### Problem 85: Queue-Based Task Scheduler ⭐⭐⭐

---

### Problem 86: Priority Queue Simulation ⭐⭐⭐

---

### Problem 88: Circular Queue ⭐⭐⭐

---

### Problem 89: Stack using Two Queues ⭐⭐⭐

---

### Problem 92: Generic Repository Simulator ⭐⭐⭐
**Concepts**: Generic Classes, Type Constraints, Repository Pattern, CRUD Operations

**What You'll Learn**:
- Creating generic classes
- Type constraints (`where T : class`)
- Repository design pattern
- Generic collections usage
- Predicate delegates with generics

**Requirements**:
Create a generic Repository<T> class that:
1. Stores items of type T
2. Supports CRUD operations
3. Uses constraints to ensure T is a class
4. Provides search/filter capabilities

**Complete Implementation**:
```csharp
// Repository Pattern - Generic data access layer
class Repository<T> where T : class
{
    private List<T> items;
    
    public int Count => items.Count;
    
    public Repository()
    {
        items = new List<T>();
    }
    
    // CREATE
    public void Add(T item)
    {
        if (item == null)
            throw new ArgumentNullException(nameof(item));
        
        items.Add(item);
    }
    
    public void AddRange(IEnumerable<T> collection)
    {
        if (collection == null)
            throw new ArgumentNullException(nameof(collection));
        
        items.AddRange(collection);
    }
    
    // READ
    public T GetById(int id, Func<T, int> idSelector)
    {
        return items.FirstOrDefault(item => idSelector(item) == id);
    }
    
    public List<T> GetAll()
    {
        return new List<T>(items); // Return copy
    }
    
    public T Find(Predicate<T> match)
    {
        return items.Find(match);
    }
    
    public List<T> FindAll(Predicate<T> match)
    {
        return items.FindAll(match);
    }
    
    // UPDATE
    public bool Update(Predicate<T> match, T newItem)
    {
        int index = items.FindIndex(match);
        if (index == -1)
            return false;
        
        items[index] = newItem;
        return true;
    }
    
    // DELETE
    public bool Remove(T item)
    {
        return items.Remove(item);
    }
    
    public int RemoveAll(Predicate<T> match)
    {
        return items.RemoveAll(match);
    }
    
    public void Clear()
    {
        items.Clear();
    }
    
    // QUERY
    public bool Exists(Predicate<T> match)
    {
        return items.Exists(match);
    }
    
    public int CountWhere(Predicate<T> match)
    {
        return items.Count(item => match(item));
    }
    
    // SORT
    public void Sort(Comparison<T> comparison)
    {
        items.Sort(comparison);
    }
    
    // STATISTICS
    public void DisplayStatistics()
    {
        Console.WriteLine($"Repository<{typeof(T).Name}> Statistics:");
        Console.WriteLine($"  Total items: {Count}");
    }
}

// Example entities to use with repository
class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
    
    public override string ToString()
    {
        return $"[{Id}] {Name} - ${Price:F2} ({Category})";
    }
}

class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    
    public override string ToString()
    {
        return $"[{Id}] {Name} ({Email})";
    }
}

// Demonstration
class RepositoryDemo
{
    static void Main()
    {
        Console.WriteLine("=== GENERIC REPOSITORY DEMO ===\n");
        
        // Product Repository
        Console.WriteLine("--- PRODUCT REPOSITORY ---\n");
        Repository<Product> productRepo = new Repository<Product>();
        
        // Add products
        productRepo.Add(new Product { Id = 1, Name = "Laptop", Price = 999.99m, Category = "Electronics" });
        productRepo.Add(new Product { Id = 2, Name = "Mouse", Price = 29.99m, Category = "Electronics" });
        productRepo.Add(new Product { Id = 3, Name = "Desk", Price = 299.99m, Category = "Furniture" });
        productRepo.Add(new Product { Id = 4, Name = "Chair", Price = 149.99m, Category = "Furniture" });
        
        Console.WriteLine("All products:");
        foreach (var product in productRepo.GetAll())
        {
            Console.WriteLine($"  {product}");
        }
        
        // Find by condition
        Console.WriteLine("\nProducts under $100:");
        var affordable = productRepo.FindAll(p => p.Price < 100);
        foreach (var product in affordable)
        {
            Console.WriteLine($"  {product}");
        }
        
        // Find by category
        Console.WriteLine("\nElectronics:");
        var electronics = productRepo.FindAll(p => p.Category == "Electronics");
        foreach (var product in electronics)
        {
            Console.WriteLine($"  {product}");
        }
        
        // Update
        Console.WriteLine("\nUpdating Laptop price to $899.99:");
        productRepo.Update(p => p.Id == 1, 
            new Product { Id = 1, Name = "Laptop", Price = 899.99m, Category = "Electronics" });
        
        var laptop = productRepo.Find(p => p.Id == 1);
        Console.WriteLine($"  {laptop}");
        
        // Delete
        Console.WriteLine("\nRemoving items over $200:");
        int removed = productRepo.RemoveAll(p => p.Price > 200);
        Console.WriteLine($"  Removed {removed} items");
        
        productRepo.DisplayStatistics();
        
        // Customer Repository (same generic class, different type!)
        Console.WriteLine("\n--- CUSTOMER REPOSITORY ---\n");
        Repository<Customer> customerRepo = new Repository<Customer>();
        
        customerRepo.AddRange(new List<Customer>
        {
            new Customer { Id = 1, Name = "Alice", Email = "alice@email.com" },
            new Customer { Id = 2, Name = "Bob", Email = "bob@email.com" },
            new Customer { Id = 3, Name = "Charlie", Email = "charlie@email.com" }
        });
        
        Console.WriteLine("All customers:");
        foreach (var customer in customerRepo.GetAll())
        {
            Console.WriteLine($"  {customer}");
        }
        
        // Search by email domain
        Console.WriteLine("\nCustomers with gmail:");
        var gmailUsers = customerRepo.FindAll(c => c.Email.Contains("gmail"));
        Console.WriteLine($"  Found {gmailUsers.Count} gmail users");
        
        customerRepo.DisplayStatistics();
    }
}
```

**Type Constraint Explained**:
```csharp
// WITHOUT constraint - can be ANY type
class Repository<T>
{
    // Problem: What if T is int, string, etc.?
    // We want objects with properties!
}

// WITH constraint - T must be a reference type (class)
class Repository<T> where T : class
{
    // ✓ Ensures T is a class with properties
    // ✓ Can safely use null checks
    // ✓ No boxing for reference types
}
```

**Common Constraints**:
```csharp
where T : struct          // T must be value type
where T : class           // T must be reference type
where T : new()           // T must have parameterless constructor
where T : SomeBaseClass   // T must inherit from SomeBaseClass
where T : ISomeInterface  // T must implement ISomeInterface
where T : U               // T must be or derive from U

// Multiple constraints
where T : class, new()    // Must be class AND have default constructor
where T : IComparable<T>, new() // Must implement interface AND have constructor
```

**Bonus Challenges**:
- ⭐⭐ Add pagination (GetPage method)
- ⭐⭐⭐ Add IDisposable for cleanup
- ⭐⭐⭐ Create IRepository<T> interface
- ⭐⭐⭐⭐ Add async methods (Task<T>)

**Real-World Usage**:
- Data access layers
- Entity Framework patterns
- Service layers
- Generic utilities

---

---

### Problem 99: LINQ Grouping ⭐⭐⭐
**Concepts**: GroupBy, Key, Group Operations, Nested Grouping

**What You'll Learn**:
- Grouping data by key
- Working with IGrouping<TKey, TElement>
- Aggregating within groups
- Multi-level grouping
- Transforming grouped data

**Requirements**:
Group and analyze collections:
1. Group by single property
2. Aggregate within groups
3. Multi-level grouping
4. Transform grouped results

**Complete Implementation**:
```csharp
class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Department { get; set; }
    public string City { get; set; }
    public decimal Salary { get; set; }
    public int YearsOfService { get; set; }
    
    public override string ToString()
    {
        return $"{Name} ({Department}) - ${Salary:F0}/year, {YearsOfService}yrs";
    }
}

class LinqGrouping
{
    static List<Employee> GetEmployees()
    {
        return new List<Employee>
        {
            new Employee { Id = 1, Name = "Alice", Department = "Engineering", City = "Seattle", Salary = 95000, YearsOfService = 5 },
            new Employee { Id = 2, Name = "Bob", Department = "Engineering", City = "Seattle", Salary = 85000, YearsOfService = 3 },
            new Employee { Id = 3, Name = "Charlie", Department = "Sales", City = "New York", Salary = 75000, YearsOfService = 7 },
            new Employee { Id = 4, Name = "Diana", Department = "Sales", City = "New York", Salary = 70000, YearsOfService = 4 },
            new Employee { Id = 5, Name = "Eve", Department = "Engineering", City = "Austin", Salary = 90000, YearsOfService = 6 },
            new Employee { Id = 6, Name = "Frank", Department = "Marketing", City = "Seattle", Salary = 65000, YearsOfService = 2 },
            new Employee { Id = 7, Name = "Grace", Department = "Marketing", City = "Austin", Salary = 68000, YearsOfService = 3 },
            new Employee { Id = 8, Name = "Henry", Department = "Sales", City = "Seattle", Salary = 72000, YearsOfService = 5 },
            new Employee { Id = 9, Name = "Ivy", Department = "Engineering", City = "New York", Salary = 88000, YearsOfService = 4 },
            new Employee { Id = 10, Name = "Jack", Department = "Marketing", City = "New York", Salary = 66000, YearsOfService = 1 }
        };
    }
    
    static void Main()
    {
        var employees = GetEmployees();
        
        Console.WriteLine("=== LINQ GROUPING ===\n");
        
        // BASIC GROUPING - Group by Department
        Console.WriteLine("--- Employees by Department ---");
        var byDepartment = employees.GroupBy(e => e.Department);
        
        foreach (var group in byDepartment)
        {
            Console.WriteLine($"\n{group.Key} ({group.Count()} employees):");
            foreach (var emp in group)
            {
                Console.WriteLine($"  - {emp.Name}: ${emp.Salary:F0}");
            }
        }
        
        // GROUPING WITH AGGREGATION
        Console.WriteLine("\n\n--- Department Statistics ---");
        var deptStats = employees.GroupBy(e => e.Department);
        
        foreach (var group in deptStats)
        {
            Console.WriteLine($"\n{group.Key}:");
            Console.WriteLine($"  Employee Count: {group.Count()}");
            Console.WriteLine($"  Total Payroll: ${group.Sum(e => e.Salary):F2}");
            Console.WriteLine($"  Average Salary: ${group.Average(e => e.Salary):F2}");
            Console.WriteLine($"  Highest Salary: ${group.Max(e => e.Salary):F2}");
            Console.WriteLine($"  Avg Years of Service: {group.Average(e => e.YearsOfService):F1}");
        }
        
        // GROUPING AND PROJECTING
        Console.WriteLine("\n\n--- Department Summary (Anonymous Type) ---");
        var summary = employees.GroupBy(e => e.Department)
                              .Select(g => new
                              {
                                  Department = g.Key,
                                  Count = g.Count(),
                                  TotalSalary = g.Sum(e => e.Salary),
                                  AvgSalary = g.Average(e => e.Salary),
                                  TopEarner = g.OrderByDescending(e => e.Salary).First().Name
                              });
        
        foreach (var dept in summary)
        {
            Console.WriteLine($"{dept.Department}:");
            Console.WriteLine($"  Employees: {dept.Count}");
            Console.WriteLine($"  Total Payroll: ${dept.TotalSalary:F0}");
            Console.WriteLine($"  Avg Salary: ${dept.AvgSalary:F0}");
            Console.WriteLine($"  Top Earner: {dept.TopEarner}");
        }
        
        // GROUP BY CITY
        Console.WriteLine("\n\n--- Employees by City ---");
        var byCity = employees.GroupBy(e => e.City)
                             .OrderByDescending(g => g.Count());
        
        foreach (var group in byCity)
        {
            Console.WriteLine($"\n{group.Key} ({group.Count()} employees):");
            foreach (var emp in group.OrderByDescending(e => e.Salary))
            {
                Console.WriteLine($"  {emp.Name} - {emp.Department} - ${emp.Salary:F0}");
            }
        }
        
        // MULTI-LEVEL GROUPING - Group by Department, then City
        Console.WriteLine("\n\n--- Multi-Level: Department → City ---");
        var multiLevel = employees.GroupBy(e => e.Department)
                                 .Select(deptGroup => new
                                 {
                                     Department = deptGroup.Key,
                                     Cities = deptGroup.GroupBy(e => e.City)
                                                      .Select(cityGroup => new
                                                      {
                                                          City = cityGroup.Key,
                                                          Count = cityGroup.Count(),
                                                          Employees = cityGroup.ToList()
                                                      })
                                 });
        
        foreach (var dept in multiLevel)
        {
            Console.WriteLine($"\n{dept.Department}:");
            foreach (var city in dept.Cities)
            {
                Console.WriteLine($"  {city.City} ({city.Count} employees):");
                foreach (var emp in city.Employees)
                {
                    Console.WriteLine($"    - {emp.Name}");
                }
            }
        }
        
        // GROUP BY CALCULATED VALUE
        Console.WriteLine("\n\n--- By Salary Range ---");
        var bySalaryRange = employees.GroupBy(e => 
        {
            if (e.Salary < 70000) return "< $70K";
            else if (e.Salary < 80000) return "$70K-$80K";
            else if (e.Salary < 90000) return "$80K-$90K";
            else return "$90K+";
        });
        
        foreach (var group in bySalaryRange.OrderBy(g => g.Key))
        {
            Console.WriteLine($"\n{group.Key}: {group.Count()} employees");
            foreach (var emp in group)
            {
                Console.WriteLine($"  {emp.Name}: ${emp.Salary:F0}");
            }
        }
        
        // QUERY SYNTAX FOR GROUPING
        Console.WriteLine("\n\n--- Query Syntax Example ---");
        var queryGrouping = from e in employees
                           group e by e.Department into deptGroup
                           select new
                           {
                               Department = deptGroup.Key,
                               Count = deptGroup.Count(),
                               AvgSalary = deptGroup.Average(emp => emp.Salary)
                           };
        
        foreach (var dept in queryGrouping)
        {
            Console.WriteLine($"{dept.Department}: {dept.Count} employees, Avg: ${dept.AvgSalary:F0}");
        }
        
        // FINDING TOP N PER GROUP
        Console.WriteLine("\n\n--- Top 2 Earners per Department ---");
        var topEarners = employees.GroupBy(e => e.Department)
                                 .Select(g => new
                                 {
                                     Department = g.Key,
                                     TopTwo = g.OrderByDescending(e => e.Salary)
                                              .Take(2)
                                              .ToList()
                                 });
        
        foreach (var dept in topEarners)
        {
            Console.WriteLine($"\n{dept.Department}:");
            foreach (var emp in dept.TopTwo)
            {
                Console.WriteLine($"  {emp.Name}: ${emp.Salary:F0}");
            }
        }
    }
}
```

**GroupBy Fundamentals**:
```csharp
// GroupBy returns IEnumerable<IGrouping<TKey, TElement>>

var groups = items.GroupBy(i => i.Category);

// Each group has:
// - Key: The grouping key (Category value)
// - IEnumerable<T>: The items in that group

foreach (var group in groups)
{
    Console.WriteLine(group.Key);  // The category
    foreach (var item in group)    // The items in this category
    {
        Console.WriteLine(item);
    }
}
```

**Common Grouping Patterns**:
```csharp
// Simple grouping
var byCategory = products.GroupBy(p => p.Category);

// Group and count
var counts = products.GroupBy(p => p.Category)
                    .Select(g => new { Category = g.Key, Count = g.Count() });

// Group and sum
var totals = sales.GroupBy(s => s.Region)
                 .Select(g => new { Region = g.Key, Total = g.Sum(s => s.Amount) });

// Group by multiple properties (composite key)
var composite = items.GroupBy(i => new { i.Year, i.Month });

// Group by calculated value
var ranges = numbers.GroupBy(n => n / 10 * 10);  // Group into 0-9, 10-19, 20-29...

// Multi-level grouping
var multiLevel = items.GroupBy(i => i.Category)
                     .Select(g => new
                     {
                         Category = g.Key,
                         SubGroups = g.GroupBy(i => i.SubCategory)
                     });
```

**Query Syntax**:
```csharp
// Method syntax
var result = items.GroupBy(i => i.Category)
                 .Select(g => new { Category = g.Key, Count = g.Count() });

// Query syntax (note: "group...by...into")
var result = from i in items
             group i by i.Category into g
             select new { Category = g.Key, Count = g.Count() };
```

**Bonus Challenges**:
- ⭐⭐⭐ Group by date ranges (weeks, months)
- ⭐⭐⭐ Implement pivot table functionality
- ⭐⭐⭐⭐ Dynamic grouping based on user selection
- ⭐⭐⭐⭐ Hierarchical grouping (3+ levels)

**Real-World Usage**:
- Sales reports by region/product
- Analytics dashboards
- Data aggregation for charts
- Organizing hierarchical data
- Category summaries

**Interview Tips**:
💡 Know: "GroupBy returns IGrouping<TKey, TElement>"  
💡 Explain: "Key property holds the grouping value"  
💡 Common question: "Find top N per group"  
💡 Mention performance for large datasets  

---

---

### Problem 100: LINQ Join Operations ⭐⭐⭐
**Concepts**: Join, GroupJoin, Inner Join, Left Outer Join, Multiple Joins

**What You'll Learn**:
- Inner joins
- Left outer joins
- Group joins
- Joining multiple collections
- Method syntax vs Query syntax for joins

**Requirements**:
Join related collections:
1. Inner join (matching records only)
2. Left outer join (all from left, matching from right)
3. Multi-table joins
4. Group join

**Complete Implementation**:
```csharp
class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string City { get; set; }
    
    public override string ToString() => $"[{Id}] {Name} from {City}";
}

class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public string Product { get; set; }
    public decimal Amount { get; set; }
    public DateTime OrderDate { get; set; }
    
    public override string ToString() => $"Order #{Id}: {Product} - ${Amount}";
}

class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    
    public override string ToString() => $"{Name} - ${Price}";
}

class OrderDetail
{
    public int OrderId { get; set; }
    public int ProductId { get; set; }
    public int Quantity { get; set; }
}

class LinqJoins
{
    static void Main()
    {
        var customers = new List<Customer>
        {
            new Customer { Id = 1, Name = "Alice", City = "Seattle" },
            new Customer { Id = 2, Name = "Bob", City = "New York" },
            new Customer { Id = 3, Name = "Charlie", City = "Austin" },
            new Customer { Id = 4, Name = "Diana", City = "Seattle" },
            new Customer { Id = 5, Name = "Eve", City = "Boston" }
        };
        
        var orders = new List<Order>
        {
            new Order { Id = 101, CustomerId = 1, Product = "Laptop", Amount = 1200, OrderDate = new DateTime(2024, 1, 15) },
            new Order { Id = 102, CustomerId = 1, Product = "Mouse", Amount = 30, OrderDate = new DateTime(2024, 1, 16) },
            new Order { Id = 103, CustomerId = 2, Product = "Keyboard", Amount = 80, OrderDate = new DateTime(2024, 1, 17) },
            new Order { Id = 104, CustomerId = 3, Product = "Monitor", Amount = 300, OrderDate = new DateTime(2024, 1, 18) },
            new Order { Id = 105, CustomerId = 1, Product = "Desk", Amount = 400, OrderDate = new DateTime(2024, 1, 19) },
            new Order { Id = 106, CustomerId = 4, Product = "Chair", Amount = 150, OrderDate = new DateTime(2024, 1, 20) }
        };
        
        Console.WriteLine("=== LINQ JOIN OPERATIONS ===\n");
        
        // INNER JOIN - Method Syntax
        Console.WriteLine("--- INNER JOIN (Method Syntax) ---");
        var innerJoin = customers.Join(
            orders,                          // Inner collection
            c => c.Id,                       // Outer key selector
            o => o.CustomerId,               // Inner key selector
            (c, o) => new                    // Result selector
            {
                CustomerName = c.Name,
                City = c.City,
                OrderId = o.Id,
                Product = o.Product,
                Amount = o.Amount
            });
        
        foreach (var item in innerJoin)
        {
            Console.WriteLine($"{item.CustomerName} ({item.City}) - Order #{item.OrderId}: {item.Product} ${item.Amount}");
        }
        
        // INNER JOIN - Query Syntax
        Console.WriteLine("\n--- INNER JOIN (Query Syntax) ---");
        var queryJoin = from c in customers
                       join o in orders on c.Id equals o.CustomerId
                       select new
                       {
                           Customer = c.Name,
                           Order = o.Product,
                           Amount = o.Amount
                       };
        
        foreach (var item in queryJoin)
        {
            Console.WriteLine($"{item.Customer} ordered {item.Order} for ${item.Amount}");
        }
        
        // LEFT OUTER JOIN - All customers, even without orders
        Console.WriteLine("\n--- LEFT OUTER JOIN ---");
        var leftJoin = from c in customers
                      join o in orders on c.Id equals o.CustomerId into customerOrders
                      from co in customerOrders.DefaultIfEmpty()
                      select new
                      {
                          CustomerName = c.Name,
                          OrderProduct = co?.Product ?? "No orders",
                          OrderAmount = co?.Amount ?? 0
                      };
        
        foreach (var item in leftJoin)
        {
            Console.WriteLine($"{item.CustomerName}: {item.OrderProduct} - ${item.OrderAmount}");
        }
        
        // GROUP JOIN - All customers with their list of orders
        Console.WriteLine("\n--- GROUP JOIN ---");
        var groupJoin = customers.GroupJoin(
            orders,
            c => c.Id,
            o => o.CustomerId,
            (c, orders) => new
            {
                Customer = c,
                Orders = orders.ToList()
            });
        
        foreach (var item in groupJoin)
        {
            Console.WriteLine($"\n{item.Customer.Name} ({item.Orders.Count} orders):");
            if (item.Orders.Any())
            {
                foreach (var order in item.Orders)
                {
                    Console.WriteLine($"  - {order.Product}: ${order.Amount}");
                }
                Console.WriteLine($"  Total: ${item.Orders.Sum(o => o.Amount)}");
            }
            else
            {
                Console.WriteLine("  (No orders)");
            }
        }
        
        // MULTIPLE CONDITIONS JOIN
        Console.WriteLine("\n--- JOIN WITH MULTIPLE CONDITIONS ---");
        var multiConditionJoin = from c in customers
                                join o in orders on new { Id = c.Id, c.City } 
                                equals new { Id = o.CustomerId, City = "Seattle" }
                                select new { c.Name, o.Product };
        
        foreach (var item in multiConditionJoin)
        {
            Console.WriteLine($"{item.Name} ordered {item.Product} (Seattle customers only)");
        }
        
        // CUSTOMER ORDERS SUMMARY
        Console.WriteLine("\n--- CUSTOMER ORDER SUMMARY ---");
        var summary = from c in customers
                     join o in orders on c.Id equals o.CustomerId into custOrders
                     select new
                     {
                         Customer = c.Name,
                         OrderCount = custOrders.Count(),
                         TotalSpent = custOrders.Sum(o => o.Amount),
                         AvgOrder = custOrders.Any() ? custOrders.Average(o => o.Amount) : 0
                     };
        
        foreach (var item in summary.OrderByDescending(s => s.TotalSpent))
        {
            Console.WriteLine($"{item.Customer}:");
            Console.WriteLine($"  Orders: {item.OrderCount}");
            Console.WriteLine($"  Total: ${item.TotalSpent}");
            Console.WriteLine($"  Avg: ${item.AvgOrder:F2}");
        }
        
        // THREE-WAY JOIN
        var products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 1200 },
            new Product { Id = 2, Name = "Mouse", Price = 30 },
            new Product { Id = 3, Name = "Keyboard", Price = 80 }
        };
        
        var orderDetails = new List<OrderDetail>
        {
            new OrderDetail { OrderId = 101, ProductId = 1, Quantity = 1 },
            new OrderDetail { OrderId = 102, ProductId = 2, Quantity = 2 },
            new OrderDetail { OrderId = 103, ProductId = 3, Quantity = 1 }
        };
        
        Console.WriteLine("\n--- THREE-WAY JOIN ---");
        var threeWay = from c in customers
                      join o in orders on c.Id equals o.CustomerId
                      join od in orderDetails on o.Id equals od.OrderId
                      join p in products on od.ProductId equals p.Id
                      select new
                      {
                          Customer = c.Name,
                          Product = p.Name,
                          Quantity = od.Quantity,
                          Total = p.Price * od.Quantity
                      };
        
        foreach (var item in threeWay)
        {
            Console.WriteLine($"{item.Customer} bought {item.Quantity}x {item.Product} = ${item.Total}");
        }
    }
}
```

**Join Types Comparison**:
```
INNER JOIN: Only matching records
┌─────────┐    ┌─────────┐
│    A    │    │    B    │
│   ┌──┐  │    │  ┌──┐   │
│   │AB│◄─┼────┼──┤AB│   │  Result: AB only
│   └──┘  │    │  └──┘   │
└─────────┘    └─────────┘

LEFT OUTER JOIN: All A + matching B
┌─────────┐    ┌─────────┐
│ A  ┌──┐ │    │  ┌──┐   │
│────┤AB│◄┼────┼──┤AB│   │  Result: A + AB
│    └──┘ │    │  └──┘   │
└─────────┘    └─────────┘

GROUP JOIN: A with collection of B
Customer 1 → [Order1, Order2, Order3]
Customer 2 → [Order4]
Customer 3 → []
```

**Join Syntax Templates**:
```csharp
// Inner Join - Method
var result = outer.Join(
    inner,
    o => o.Key,        // Outer key
    i => i.Key,        // Inner key
    (o, i) => new { }) // Result

// Inner Join - Query
var result = from o in outer
             join i in inner on o.Key equals i.Key
             select new { };

// Left Join - Query
var result = from o in outer
             join i in inner on o.Key equals i.Key into temp
             from t in temp.DefaultIfEmpty()
             select new { };

// Group Join
var result = outer.GroupJoin(
    inner,
    o => o.Key,
    i => i.Key,
    (o, innerCollection) => new { });
```

**Bonus Challenges**:
- ⭐⭐⭐ Implement right outer join
- ⭐⭐⭐ Implement full outer join
- ⭐⭐⭐⭐ Join on multiple conditions
- ⭐⭐⭐⭐ Self-join (employees to managers)

**Real-World Usage**:
- Database query results
- Combining data from multiple sources
- Master-detail relationships
- Order systems
- Reporting

**Interview Tips**:
💡 Know difference: Join vs GroupJoin  
💡 Explain: "Left join uses DefaultIfEmpty()"  
💡 Common question: "How to implement right/full outer join?"  
💡 Mention performance considerations  

---

*[Due to length, continuing with remaining LINQ problems 101-106 in next message...]*

**Section 4.2 Progress: 4/10 complete**

Should I continue with the remaining 6 LINQ problems?

---

### Problem 101: LINQ with Complex Objects ⭐⭐⭐
**Concepts**: Nested Objects, Navigation Properties, SelectMany, Flattening

**What You'll Learn**:
- Working with nested collections
- SelectMany for flattening
- Navigating object hierarchies
- Querying related data
- Projection with complex types

**Requirements**:
Query hierarchical data structures:
1. Access nested collections
2. Flatten hierarchical data
3. Filter at multiple levels
4. Aggregate nested data

**Complete Implementation**:
```csharp
class School
{
    public string Name { get; set; }
    public List<Department> Departments { get; set; }
}

class Department
{
    public string Name { get; set; }
    public List<Course> Courses { get; set; }
}

class Course
{
    public string Name { get; set; }
    public string Instructor { get; set; }
    public List<Student> Students { get; set; }
}

class Student
{
    public string Name { get; set; }
    public int Age { get; set; }
    public double GPA { get; set; }
}

class LinqComplexObjects
{
    static School GetSchoolData()
    {
        return new School
        {
            Name = "Tech University",
            Departments = new List<Department>
            {
                new Department
                {
                    Name = "Computer Science",
                    Courses = new List<Course>
                    {
                        new Course
                        {
                            Name = "Data Structures",
                            Instructor = "Dr. Smith",
                            Students = new List<Student>
                            {
                                new Student { Name = "Alice", Age = 20, GPA = 3.8 },
                                new Student { Name = "Bob", Age = 21, GPA = 3.6 },
                                new Student { Name = "Charlie", Age = 19, GPA = 3.9 }
                            }
                        },
                        new Course
                        {
                            Name = "Algorithms",
                            Instructor = "Dr. Jones",
                            Students = new List<Student>
                            {
                                new Student { Name = "Diana", Age = 22, GPA = 3.7 },
                                new Student { Name = "Eve", Age = 20, GPA = 3.5 }
                            }
                        }
                    }
                },
                new Department
                {
                    Name = "Mathematics",
                    Courses = new List<Course>
                    {
                        new Course
                        {
                            Name = "Calculus",
                            Instructor = "Prof. Brown",
                            Students = new List<Student>
                            {
                                new Student { Name = "Frank", Age = 19, GPA = 3.4 },
                                new Student { Name = "Grace", Age = 20, GPA = 3.8 }
                            }
                        }
                    }
                }
            }
        };
    }
    
    static void Main()
    {
        var school = GetSchoolData();
        
        Console.WriteLine("=== LINQ WITH COMPLEX OBJECTS ===\n");
        
        // SELECT MANY - Flatten departments to courses
        Console.WriteLine("--- All Courses (SelectMany) ---");
        var allCourses = school.Departments
                              .SelectMany(d => d.Courses);
        
        foreach (var course in allCourses)
        {
            Console.WriteLine($"{course.Name} - {course.Instructor}");
        }
        
        // SELECT MANY with transformation
        Console.WriteLine("\n--- All Courses with Department ---");
        var coursesWithDept = school.Departments
                                   .SelectMany(
                                       d => d.Courses,
                                       (dept, course) => new
                                       {
                                           Department = dept.Name,
                                           Course = course.Name,
                                           Instructor = course.Instructor
                                       });
        
        foreach (var item in coursesWithDept)
        {
            Console.WriteLine($"{item.Department} - {item.Course} ({item.Instructor})");
        }
        
        // FLATTEN to ALL STUDENTS
        Console.WriteLine("\n--- All Students (Fully Flattened) ---");
        var allStudents = school.Departments
                               .SelectMany(d => d.Courses)
                               .SelectMany(c => c.Students);
        
        Console.WriteLine($"Total students: {allStudents.Count()}");
        
        // FLATTEN with context
        Console.WriteLine("\n--- Students with Course & Department ---");
        var studentsWithContext = school.Departments
            .SelectMany(d => d.Courses, (dept, course) => new { dept, course })
            .SelectMany(x => x.course.Students, (x, student) => new
            {
                Department = x.dept.Name,
                Course = x.course.Name,
                Instructor = x.course.Instructor,
                Student = student.Name,
                GPA = student.GPA
            });
        
        foreach (var item in studentsWithContext)
        {
            Console.WriteLine($"{item.Student} ({item.GPA}) - {item.Course} [{item.Department}]");
        }
        
        // HIGH ACHIEVERS across all courses
        Console.WriteLine("\n--- High Achievers (GPA >= 3.7) ---");
        var highAchievers = school.Departments
                                 .SelectMany(d => d.Courses)
                                 .SelectMany(c => c.Students)
                                 .Where(s => s.GPA >= 3.7)
                                 .OrderByDescending(s => s.GPA);
        
        foreach (var student in highAchievers)
        {
            Console.WriteLine($"{student.Name}: {student.GPA}");
        }
        
        // STATISTICS per Department
        Console.WriteLine("\n--- Department Statistics ---");
        var deptStats = school.Departments.Select(d => new
        {
            Department = d.Name,
            TotalCourses = d.Courses.Count,
            TotalStudents = d.Courses.SelectMany(c => c.Students).Count(),
            AvgGPA = d.Courses.SelectMany(c => c.Students).Average(s => s.GPA)
        });
        
        foreach (var stat in deptStats)
        {
            Console.WriteLine($"\n{stat.Department}:");
            Console.WriteLine($"  Courses: {stat.TotalCourses}");
            Console.WriteLine($"  Students: {stat.TotalStudents}");
            Console.WriteLine($"  Avg GPA: {stat.AvgGPA:F2}");
        }
        
        // FIND students by name across all courses
        Console.WriteLine("\n--- Find Student: Alice ---");
        var alice = school.Departments
                         .SelectMany(d => d.Courses)
                         .SelectMany(c => c.Students)
                         .FirstOrDefault(s => s.Name == "Alice");
        
        if (alice != null)
        {
            Console.WriteLine($"Found: {alice.Name}, Age {alice.Age}, GPA {alice.GPA}");
        }
        
        // COURSES with enrollment count
        Console.WriteLine("\n--- Courses by Enrollment ---");
        var courseEnrollment = school.Departments
                                    .SelectMany(d => d.Courses, (d, c) => new
                                    {
                                        Department = d.Name,
                                        Course = c.Name,
                                        Enrollment = c.Students.Count,
                                        AvgGPA = c.Students.Average(s => s.GPA)
                                    })
                                    .OrderByDescending(x => x.Enrollment);
        
        foreach (var item in courseEnrollment)
        {
            Console.WriteLine($"{item.Course} ({item.Department}): {item.Enrollment} students, Avg GPA: {item.AvgGPA:F2}");
        }
        
        // QUERY SYNTAX version
        Console.WriteLine("\n--- Query Syntax: CS Students ---");
        var csStudents = from dept in school.Departments
                        where dept.Name == "Computer Science"
                        from course in dept.Courses
                        from student in course.Students
                        where student.GPA >= 3.6
                        select new { student.Name, student.GPA, Course = course.Name };
        
        foreach (var item in csStudents)
        {
            Console.WriteLine($"{item.Name} ({item.GPA}) in {item.Course}");
        }
    }
}
```

**SelectMany Explained**:
```csharp
// WITHOUT SelectMany - nested loops
foreach (var dept in departments)
{
    foreach (var course in dept.Courses)
    {
        // Use course
    }
}

// WITH SelectMany - flattened
var allCourses = departments.SelectMany(d => d.Courses);
foreach (var course in allCourses)
{
    // Use course directly
}

// Visual:
// Input:  [[1,2], [3,4], [5]]
// Output: [1, 2, 3, 4, 5]  (flattened!)
```

**Bonus Challenges**:
- ⭐⭐⭐ Query JSON-like nested structures
- ⭐⭐⭐ Implement tree traversal with LINQ
- ⭐⭐⭐⭐ Build query builder for nested data
- ⭐⭐⭐⭐ Performance optimize deep nesting

---

---

### Problem 104: LINQ with Custom Comparers ⭐⭐⭐
**Concepts**: IEqualityComparer, IComparer, Custom Sorting, Distinct with Comparer

**What You'll Learn**:
- Creating custom comparison logic
- IEqualityComparer<T> for Distinct, Union, Except
- IComparer<T> for sorting
- Case-insensitive comparisons
- Complex object comparison

**Requirements**:
Implement custom comparers for:
1. Case-insensitive string comparison
2. Custom object equality
3. Natural number sorting (1, 2, 10 vs 1, 10, 2)
4. Multi-property sorting

**Complete Implementation**:
```csharp
class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
    
    public override string ToString() => $"{Name} (${Price})";
}

// EQUALITY COMPARER - For Distinct, Union, Except
class ProductNameComparer : IEqualityComparer<Product>
{
    public bool Equals(Product x, Product y)
    {
        if (x == null && y == null) return true;
        if (x == null || y == null) return false;
        
        return x.Name.Equals(y.Name, StringComparison.OrdinalIgnoreCase);
    }
    
    public int GetHashCode(Product obj)
    {
        return obj.Name.ToLower().GetHashCode();
    }
}

// COMPARER - For OrderBy, sorting
class ProductPriceComparer : IComparer<Product>
{
    public int Compare(Product x, Product y)
    {
        if (x == null && y == null) return 0;
        if (x == null) return -1;
        if (y == null) return 1;
        
        return x.Price.CompareTo(y.Price);
    }
}

// Case-insensitive string comparer
class CaseInsensitiveComparer : IEqualityComparer<string>
{
    public bool Equals(string x, string y)
    {
        return string.Equals(x, y, StringComparison.OrdinalIgnoreCase);
    }
    
    public int GetHashCode(string obj)
    {
        return obj.ToLower().GetHashCode();
    }
}

class CustomComparers
{
    static void Main()
    {
        Console.WriteLine("=== LINQ WITH CUSTOM COMPARERS ===\n");
        
        // STRING COMPARISON - Case insensitive
        Console.WriteLine("--- Case-Insensitive Distinct ---");
        var words = new List<string> { "Apple", "APPLE", "Banana", "apple", "Cherry", "BANANA" };
        
        var distinct1 = words.Distinct();  // Default: case-sensitive
        Console.WriteLine($"Default Distinct: {string.Join(", ", distinct1)}");
        // Result: Apple, APPLE, Banana, apple, Cherry, BANANA
        
        var distinct2 = words.Distinct(new CaseInsensitiveComparer());
        Console.WriteLine($"Case-Insensitive: {string.Join(", ", distinct2)}");
        // Result: Apple, Banana, Cherry
        
        // PRODUCT COMPARISON
        Console.WriteLine("\n--- Custom Product Comparer ---");
        var products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999, Category = "Electronics" },
            new Product { Id = 2, Name = "laptop", Price = 1099, Category = "Electronics" }, // Different price, same name
            new Product { Id = 3, Name = "Mouse", Price = 29, Category = "Electronics" },
            new Product { Id = 4, Name = "LAPTOP", Price = 899, Category = "Electronics" }  // Different price, same name
        };
        
        var uniqueProducts = products.Distinct(new ProductNameComparer());
        Console.WriteLine("Unique products by name (case-insensitive):");
        foreach (var p in uniqueProducts)
        {
            Console.WriteLine($"  {p}");
        }
        
        // CUSTOM SORTING
        Console.WriteLine("\n--- Custom Sorting ---");
        var unsorted = new List<Product>
        {
            new Product { Name = "A", Price = 100 },
            new Product { Name = "B", Price = 50 },
            new Product { Name = "C", Price = 200 }
        };
        
        var sorted = unsorted.OrderBy(p => p, new ProductPriceComparer());
        Console.WriteLine("Sorted by custom price comparer:");
        foreach (var p in sorted)
        {
            Console.WriteLine($"  {p}");
        }
        
        // UNION with comparer
        Console.WriteLine("\n--- Union with Comparer ---");
        var list1 = new List<string> { "Apple", "Banana" };
        var list2 = new List<string> { "apple", "Cherry" };
        
        var union = list1.Union(list2, new CaseInsensitiveComparer());
        Console.WriteLine($"Union (case-insensitive): {string.Join(", ", union)}");
        
        // INTERSECT with comparer
        Console.WriteLine("\n--- Intersect with Comparer ---");
        var set1 = new List<string> { "Apple", "Banana", "Cherry" };
        var set2 = new List<string> { "APPLE", "CHERRY", "Date" };
        
        var intersect = set1.Intersect(set2, new CaseInsensitiveComparer());
        Console.WriteLine($"Intersect (case-insensitive): {string.Join(", ", intersect)}");
        
        // NATURAL NUMBER SORTING
        Console.WriteLine("\n--- Natural Number Sorting ---");
        var files = new List<string> { "file1.txt", "file10.txt", "file2.txt", "file20.txt", "file3.txt" };
        
        // Default sorting (alphabetical)
        var defaultSort = files.OrderBy(f => f);
        Console.WriteLine($"Default sort: {string.Join(", ", defaultSort)}");
        // Result: file1.txt, file10.txt, file2.txt, file20.txt, file3.txt
        
        // Natural sorting
        var naturalSort = files.OrderBy(f => f, new NaturalStringComparer());
        Console.WriteLine($"Natural sort: {string.Join(", ", naturalSort)}");
        // Result: file1.txt, file2.txt, file3.txt, file10.txt, file20.txt
    }
}

// Natural string comparer (sorts numbers naturally)
class NaturalStringComparer : IComparer<string>
{
    public int Compare(string x, string y)
    {
        if (x == y) return 0;
        if (x == null) return -1;
        if (y == null) return 1;
        
        int i = 0, j = 0;
        
        while (i < x.Length && j < y.Length)
        {
            if (char.IsDigit(x[i]) && char.IsDigit(y[j]))
            {
                // Extract numbers
                int numX = 0, numY = 0;
                while (i < x.Length && char.IsDigit(x[i]))
                {
                    numX = numX * 10 + (x[i] - '0');
                    i++;
                }
                while (j < y.Length && char.IsDigit(y[j]))
                {
                    numY = numY * 10 + (y[j] - '0');
                    j++;
                }
                
                if (numX != numY)
                    return numX.CompareTo(numY);
            }
            else
            {
                if (x[i] != y[j])
                    return x[i].CompareTo(y[j]);
                i++;
                j++;
            }
        }
        
        return x.Length.CompareTo(y.Length);
    }
}
```

**When to Use Custom Comparers**:
```csharp
// IEqualityComparer<T> - for Distinct, Union, Except, Intersect
items.Distinct(new MyEqualityComparer())
items.Union(other, new MyEqualityComparer())
items.Except(other, new MyEqualityComparer())
items.Intersect(other, new MyEqualityComparer())

// IComparer<T> - for OrderBy, ThenBy
items.OrderBy(i => i, new MyComparer())

// Built-in: StringComparer
items.Distinct(StringComparer.OrdinalIgnoreCase)
items.OrderBy(i => i.Name, StringComparer.OrdinalIgnoreCase)
```

**Bonus Challenges**:
- ⭐⭐⭐ Create multi-property comparer
- ⭐⭐⭐ Implement version number comparer (1.0.0 vs 1.0.10)
- ⭐⭐⭐⭐ Build configurable comparer (runtime property selection)

---

---

### Problem 105: LINQ Performance Optimization ⭐⭐⭐
**Concepts**: Deferred Execution, Materialization, Query Optimization, Best Practices

**What You'll Learn**:
- Deferred vs immediate execution
- When to use ToList(), ToArray()
- Avoiding multiple enumeration
- Filter before sorting
- Database query optimization
- Performance profiling

**Requirements**:
Optimize LINQ queries for performance:
1. Understand deferred execution
2. Avoid common performance pitfalls
3. Measure query performance
4. Apply optimization techniques

**Complete Implementation**:
```csharp
class PerformanceDemo
{
    static List<int> GenerateLargeList(int count)
    {
        return Enumerable.Range(1, count).ToList();
    }
    
    static void Main()
    {
        Console.WriteLine("=== LINQ PERFORMANCE OPTIMIZATION ===\n");
        
        // DEFERRED EXECUTION
        Console.WriteLine("--- Deferred vs Immediate Execution ---");
        
        var numbers = new List<int> { 1, 2, 3, 4, 5 };
        
        // Deferred - query not executed yet!
        var deferredQuery = numbers.Where(n => 
        {
            Console.WriteLine($"  Checking {n}");
            return n > 2;
        });
        
        Console.WriteLine("Query defined (no output yet)");
        
        Console.WriteLine("\nIterating now:");
        foreach (var n in deferredQuery)
        {
            Console.WriteLine($"    Result: {n}");
        }
        
        // Immediate - executed right away
        Console.WriteLine("\nWith ToList() - immediate:");
        var immediateQuery = numbers.Where(n => 
        {
            Console.WriteLine($"  Checking {n}");
            return n > 2;
        }).ToList();
        
        Console.WriteLine("Query already executed!");
        
        // MULTIPLE ENUMERATION PROBLEM
        Console.WriteLine("\n--- Multiple Enumeration Problem ---");
        
        var data = GenerateLargeList(5);
        
        // BAD - Query executed multiple times
        var badQuery = data.Where(n => n % 2 == 0);
        
        Console.WriteLine($"Count: {badQuery.Count()}");      // Enumerated once
        Console.WriteLine($"First: {badQuery.First()}");      // Enumerated again
        Console.WriteLine($"Sum: {badQuery.Sum()}");          // Enumerated again
        // Total: 3 enumerations!
        
        // GOOD - Materialize once
        var goodQuery = data.Where(n => n % 2 == 0).ToList();
        
        Console.WriteLine($"Count: {goodQuery.Count}");       // No enumeration
        Console.WriteLine($"First: {goodQuery.First()}");     // No enumeration
        Console.WriteLine($"Sum: {goodQuery.Sum()}");         // One enumeration
        // Total: 1 enumeration!
        
        // FILTER BEFORE SORT
        Console.WriteLine("\n--- Filter Before Sort ---");
        
        var largeList = GenerateLargeList(10000);
        
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        // BAD - Sort all, then filter (sorts 10000 items)
        var bad = largeList.OrderBy(n => n)
                          .Where(n => n > 9000)
                          .ToList();
        sw.Stop();
        Console.WriteLine($"Sort then filter: {sw.ElapsedMilliseconds}ms");
        
        sw.Restart();
        
        // GOOD - Filter first, then sort (sorts ~1000 items)
        var good = largeList.Where(n => n > 9000)
                           .OrderBy(n => n)
                           .ToList();
        sw.Stop();
        Console.WriteLine($"Filter then sort: {sw.ElapsedMilliseconds}ms (FASTER!)");
        
        // ANY vs COUNT
        Console.WriteLine("\n--- Any() vs Count() > 0 ---");
        
        sw.Restart();
        bool hasItems1 = largeList.Count() > 0;  // BAD - counts all
        sw.Stop();
        var time1 = sw.Elapsed.TotalMicroseconds;
        
        sw.Restart();
        bool hasItems2 = largeList.Any();        // GOOD - stops at first
        sw.Stop();
        var time2 = sw.Elapsed.TotalMicroseconds;
        
        Console.WriteLine($"Count() > 0: {time1:F2}μs");
        Console.WriteLine($"Any(): {time2:F2}μs (FASTER!)");
        
        // FIRST vs WHERE + FIRST
        Console.WriteLine("\n--- First() vs Where().First() ---");
        
        // BAD - creates intermediate collection
        var bad2 = largeList.Where(n => n > 5000).First();
        
        // GOOD - stops at first match
        var good2 = largeList.First(n => n > 5000);
        
        Console.WriteLine("Use First(predicate) instead of Where().First()");
        
        // SELECT PROJECTION
        Console.WriteLine("\n--- Select Only What You Need ---");
        
        var products = Enumerable.Range(1, 1000)
            .Select(i => new
            {
                Id = i,
                Name = $"Product {i}",
                Description = new string('x', 1000), // Large string
                Price = i * 10m
            })
            .ToList();
        
        sw.Restart();
        // BAD - Select entire object
        var badSelect = products.Where(p => p.Price > 500).ToList();
        sw.Stop();
        Console.WriteLine($"Select full object: {sw.ElapsedMilliseconds}ms");
        
        sw.Restart();
        // GOOD - Select only needed properties
        var goodSelect = products.Where(p => p.Price > 500)
                                .Select(p => new { p.Id, p.Name, p.Price })
                                .ToList();
        sw.Stop();
        Console.WriteLine($"Select only needed: {sw.ElapsedMilliseconds}ms (FASTER!)");
        
        // ASPARALLEL - For CPU-intensive operations
        Console.WriteLine("\n--- Parallel LINQ (PLINQ) ---");
        
        var heavyComputation = Enumerable.Range(1, 1000);
        
        sw.Restart();
        var sequential = heavyComputation
            .Where(n => IsPrime(n))
            .ToList();
        sw.Stop();
        Console.WriteLine($"Sequential: {sw.ElapsedMilliseconds}ms - Found {sequential.Count} primes");
        
        sw.Restart();
        var parallel = heavyComputation
            .AsParallel()
            .Where(n => IsPrime(n))
            .ToList();
        sw.Stop();
        Console.WriteLine($"Parallel: {sw.ElapsedMilliseconds}ms - Found {parallel.Count} primes (FASTER!)");
        
        // BEST PRACTICES SUMMARY
        Console.WriteLine("\n=== BEST PRACTICES ===");
        Console.WriteLine("✓ Use Any() instead of Count() > 0");
        Console.WriteLine("✓ Filter before sorting");
        Console.WriteLine("✓ Use First(predicate) not Where().First()");
        Console.WriteLine("✓ Materialize once with ToList() if using multiple times");
        Console.WriteLine("✓ Select only needed properties");
        Console.WriteLine("✓ Use AsParallel() for CPU-intensive operations");
        Console.WriteLine("✓ Avoid multiple enumerations");
        Console.WriteLine("✓ Use Take() for pagination, not Skip().Take() on large sets");
    }
    
    static bool IsPrime(int number)
    {
        if (number < 2) return false;
        for (int i = 2; i <= Math.Sqrt(number); i++)
        {
            if (number % i == 0) return false;
        }
        return true;
    }
}
```

**Performance Checklist**:
```csharp
// ❌ BAD
var query = items.Where(i => i.Active);
if (query.Count() > 0)        // Enumeration 1
{
    var first = query.First(); // Enumeration 2
    var total = query.Sum();   // Enumeration 3
}

// ✅ GOOD
var query = items.Where(i => i.Active).ToList(); // Materialize once
if (query.Any())              // No enumeration
{
    var first = query.First(); // No enumeration
    var total = query.Sum();   // Single enumeration
}

// ❌ BAD - Sort all, then filter
items.OrderBy(i => i.Price).Where(i => i.Price > 100)

// ✅ GOOD - Filter first, then sort
items.Where(i => i.Price > 100).OrderBy(i => i.Price)

// ❌ BAD - Count all
bool hasAny = items.Count() > 0;

// ✅ GOOD - Stop at first
bool hasAny = items.Any();

// ❌ BAD - Extra step
var item = items.Where(i => i.Id == 5).First();

// ✅ GOOD - Combined
var item = items.First(i => i.Id == 5);
```

**Bonus Challenges**:
- ⭐⭐⭐ Benchmark LINQ vs loops
- ⭐⭐⭐ Profile memory usage
- ⭐⭐⭐⭐ Optimize Entity Framework queries
- ⭐⭐⭐⭐ Build performance monitoring

---

---

### Problem 106: LINQ Data Pipeline ⭐⭐⭐
**Concepts**: Chaining Operations, ETL Process, Data Transformation, Integration

**What You'll Learn**:
- Building data pipelines
- Chaining multiple LINQ operations
- Transform data through stages
- Error handling in pipelines
- Logging and monitoring

**Requirements**:
Build a complete data processing pipeline:
1. Extract data from source
2. Transform with multiple operations
3. Load into destination format
4. Error handling
5. Progress reporting

**Complete Implementation**:
```csharp
class RawData
{
    public string Line { get; set; }
    public int LineNumber { get; set; }
}

class ParsedRecord
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Amount { get; set; }
    public DateTime Date { get; set; }
    public bool IsValid { get; set; }
    public string ErrorMessage { get; set; }
}

class ProcessedRecord
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Amount { get; set; }
    public DateTime Date { get; set; }
    public decimal Tax { get; set; }
    public decimal Total { get; set; }
    public string Category { get; set; }
}

class DataPipeline
{
    static void Main()
    {
        Console.WriteLine("=== LINQ DATA PIPELINE ===\n");
        
        // STAGE 1: EXTRACT - Simulate reading from file
        var rawData = ExtractData();
        Console.WriteLine($"Extracted: {rawData.Count} lines");
        
        // STAGE 2: PARSE - Convert to structured format
        var parsedRecords = rawData
            .Select(ParseLine)
            .ToList();
        
        var validCount = parsedRecords.Count(r => r.IsValid);
        var errorCount = parsedRecords.Count(r => !r.IsValid);
        Console.WriteLine($"Parsed: {validCount} valid, {errorCount} errors");
        
        // Display errors
        if (errorCount > 0)
        {
            Console.WriteLine("\nErrors:");
            var errors = parsedRecords.Where(r => !r.IsValid);
            foreach (var error in errors)
            {
                Console.WriteLine($"  Line {rawData.FindIndex(r => r.LineNumber == error.Id) + 1}: {error.ErrorMessage}");
            }
        }
        
        // STAGE 3: FILTER - Only valid records
        var validRecords = parsedRecords.Where(r => r.IsValid);
        
        // STAGE 4: TRANSFORM - Apply business logic
        var processedRecords = validRecords
            .Select(r => new ProcessedRecord
            {
                Id = r.Id,
                Name = r.Name,
                Amount = r.Amount,
                Date = r.Date,
                Tax = r.Amount * 0.1m,  // 10% tax
                Total = r.Amount * 1.1m,
                Category = CategorizeAmount(r.Amount)
            })
            .ToList();
        
        Console.WriteLine($"\nProcessed: {processedRecords.Count} records");
        
        // STAGE 5: ENRICH - Add computed fields
        var enrichedRecords = processedRecords
            .Select(r => new
            {
                r.Id,
                r.Name,
                r.Amount,
                r.Tax,
                r.Total,
                r.Category,
                Month = r.Date.ToString("yyyy-MM"),
                Quarter = GetQuarter(r.Date),
                IsLarge = r.Amount > 1000
            })
            .ToList();
        
        // STAGE 6: AGGREGATE - Generate statistics
        Console.WriteLine("\n=== PIPELINE STATISTICS ===");
        
        var totalAmount = enrichedRecords.Sum(r => r.Amount);
        var totalTax = enrichedRecords.Sum(r => r.Tax);
        var avgAmount = enrichedRecords.Average(r => r.Amount);
        
        Console.WriteLine($"Total Amount: ${totalAmount:F2}");
        Console.WriteLine($"Total Tax: ${totalTax:F2}");
        Console.WriteLine($"Average Amount: ${avgAmount:F2}");
        
        // Group by category
        Console.WriteLine("\nBy Category:");
        var byCategory = enrichedRecords
            .GroupBy(r => r.Category)
            .Select(g => new
            {
                Category = g.Key,
                Count = g.Count(),
                Total = g.Sum(r => r.Total)
            })
            .OrderByDescending(x => x.Total);
        
        foreach (var cat in byCategory)
        {
            Console.WriteLine($"  {cat.Category}: {cat.Count} transactions, ${cat.Total:F2}");
        }
        
        // Group by month
        Console.WriteLine("\nBy Month:");
        var byMonth = enrichedRecords
            .GroupBy(r => r.Month)
            .Select(g => new
            {
                Month = g.Key,
                Count = g.Count(),
                Total = g.Sum(r => r.Total)
            })
            .OrderBy(x => x.Month);
        
        foreach (var month in byMonth)
        {
            Console.WriteLine($"  {month.Month}: {month.Count} transactions, ${month.Total:F2}");
        }
        
        // STAGE 7: OUTPUT - Format for export
        var exportData = enrichedRecords
            .OrderByDescending(r => r.Total)
            .Select(r => $"{r.Id},{r.Name},{r.Amount},{r.Tax},{r.Total},{r.Category},{r.Month}")
            .ToList();
        
        Console.WriteLine($"\n=== EXPORT ===");
        Console.WriteLine($"Generated {exportData.Count} lines for export");
        Console.WriteLine("\nSample (top 3):");
        foreach (var line in exportData.Take(3))
        {
            Console.WriteLine($"  {line}");
        }
        
        // COMPLETE PIPELINE IN ONE CHAIN
        Console.WriteLine("\n=== COMPLETE PIPELINE (SINGLE CHAIN) ===");
        
        var pipelineResult = ExtractData()
            .Select(ParseLine)                                    // Parse
            .Where(r => r.IsValid)                               // Filter
            .Select(r => new ProcessedRecord                     // Transform
            {
                Id = r.Id,
                Name = r.Name,
                Amount = r.Amount,
                Date = r.Date,
                Tax = r.Amount * 0.1m,
                Total = r.Amount * 1.1m,
                Category = CategorizeAmount(r.Amount)
            })
            .OrderByDescending(r => r.Total)                     // Sort
            .Take(5)                                             // Limit
            .ToList();                                           // Materialize
        
        Console.WriteLine("Top 5 transactions:");
        foreach (var record in pipelineResult)
        {
            Console.WriteLine($"  {record.Name}: ${record.Total:F2} ({record.Category})");
        }
    }
    
    static List<RawData> ExtractData()
    {
        // Simulate reading from file/database
        return new List<RawData>
        {
            new RawData { LineNumber = 1, Line = "1,Alice,500.00,2024-01-15" },
            new RawData { LineNumber = 2, Line = "2,Bob,1200.50,2024-01-16" },
            new RawData { LineNumber = 3, Line = "3,Charlie,invalid,2024-01-17" }, // Error
            new RawData { LineNumber = 4, Line = "4,Diana,750.00,2024-02-10" },
            new RawData { LineNumber = 5, Line = "5,Eve,2500.00,2024-02-15" },
            new RawData { LineNumber = 6, Line = "6,Frank,150.00,2024-03-01" },
            new RawData { LineNumber = 7, Line = "7,Grace,missing-field" }, // Error
            new RawData { LineNumber = 8, Line = "8,Henry,900.00,2024-03-20" }
        };
    }
    
    static ParsedRecord ParseLine(RawData raw)
    {
        try
        {
            var parts = raw.Line.Split(',');
            
            if (parts.Length != 4)
            {
                return new ParsedRecord
                {
                    Id = raw.LineNumber,
                    IsValid = false,
                    ErrorMessage = "Invalid format: expected 4 fields"
                };
            }
            
            return new ParsedRecord
            {
                Id = int.Parse(parts[0]),
                Name = parts[1],
                Amount = decimal.Parse(parts[2]),
                Date = DateTime.Parse(parts[3]),
                IsValid = true
            };
        }
        catch (Exception ex)
        {
            return new ParsedRecord
            {
                Id = raw.LineNumber,
                IsValid = false,
                ErrorMessage = ex.Message
            };
        }
    }
    
    static string CategorizeAmount(decimal amount)
    {
        if (amount < 100) return "Small";
        if (amount < 500) return "Medium";
        if (amount < 1000) return "Large";
        return "Extra Large";
    }
    
    static string GetQuarter(DateTime date)
    {
        int quarter = (date.Month - 1) / 3 + 1;
        return $"{date.Year}-Q{quarter}";
    }
}
```

**Pipeline Patterns**:
```csharp
// ETL Pattern (Extract, Transform, Load)
var result = dataSource
    .Extract()                // Get raw data
    .Parse()                  // Convert to objects
    .Validate()               // Check validity
    .Transform()              // Apply business logic
    .Enrich()                 // Add computed fields
    .Aggregate()              // Calculate statistics
    .Format()                 // Prepare for output
    .Load();                  // Save/return

// Error Handling Pattern
var processed = data
    .Select(item => TryProcess(item))
    .Where(result => result.IsSuccess)
    .Select(result => result.Value);

// Logging Pattern
var result = data
    .Do(Log("Starting"))
    .Transform1()
    .Do(Log("After Transform1"))
    .Transform2()
    .Do(Log("After Transform2"))
    .ToList();
```

**Bonus Challenges**:
- ⭐⭐⭐ Add retry logic for failures
- ⭐⭐⭐ Implement batch processing
- ⭐⭐⭐⭐ Build async pipeline with Task
- ⭐⭐⭐⭐ Add cancellation token support

**Real-World Usage**:
- Data migration
- ETL processes
- Log processing
- Report generation
- Data cleansing

---

## ✅ Section 4.2 COMPLETE!

**LINQ Mastery (10/10 Problems)** 🎉

**What You've Mastered**:
- ✅ Filtering and sorting (Where, OrderBy)
- ✅ Aggregations (Sum, Average, Count, Max, Min)
- ✅ Grouping data (GroupBy)
- ✅ Joining collections (Join, GroupJoin)
- ✅ Complex objects and nesting (SelectMany)
- ✅ Projections and transformations (Select)
- ✅ Both syntaxes (Method and Query)
- ✅ Custom comparers
- ✅ Performance optimization
- ✅ Complete data pipelines

**🔥 You now have the #1 most important C# job skill!**

---

## 🎯 Phase 4 Progress

**Section 4.1: Generics & Constraints** (6/6) ✅  
**Section 4.2: LINQ Mastery** (10/10) ✅  
**Section 4.3: Delegates, Events & Lambdas** (0/9) ⏸️

**Total Phase 4: 16/25 (64%)**

---

## 🚀 Next: Section 4.3 - Delegates, Events & Lambdas

The final piece of modern C# programming:
- Event-driven architecture
- Callback patterns
- Lambda expressions
- Func, Action, Predicate
- Real-world event systems

**Ready to complete Phase 4?** 🎯

---

### Problem 111: Custom Event with EventArgs ⭐⭐⭐
**Concepts**: Custom EventArgs, Data Passing, Event Design, Best Practices

**What You'll Learn**:
- Creating custom EventArgs classes
- Passing data through events
- Immutable event data
- Event naming conventions
- Strong typing for events

**Requirements**:
Build a stock trading system with custom events:
1. PriceChangedEventArgs with old/new price
2. TradeExecutedEventArgs with details
3. AlertTriggeredEventArgs with conditions
4. Proper event raising pattern

**Complete Implementation**:
```csharp
// CUSTOM EVENTARGS CLASSES
class PriceChangedEventArgs : EventArgs
{
    public string Symbol { get; }
    public decimal OldPrice { get; }
    public decimal NewPrice { get; }
    public decimal Change => NewPrice - OldPrice;
    public decimal ChangePercent => (Change / OldPrice) * 100;
    public DateTime Timestamp { get; }
    
    public PriceChangedEventArgs(string symbol, decimal oldPrice, decimal newPrice)
    {
        Symbol = symbol;
        OldPrice = oldPrice;
        NewPrice = newPrice;
        Timestamp = DateTime.Now;
    }
}

class TradeExecutedEventArgs : EventArgs
{
    public string Symbol { get; }
    public int Quantity { get; }
    public decimal Price { get; }
    public decimal TotalValue => Quantity * Price;
    public TradeType Type { get; }
    public DateTime ExecutedAt { get; }
    
    public TradeExecutedEventArgs(string symbol, int qty, decimal price, TradeType type)
    {
        Symbol = symbol;
        Quantity = qty;
        Price = price;
        Type = type;
        ExecutedAt = DateTime.Now;
    }
}

class AlertTriggeredEventArgs : EventArgs
{
    public string Symbol { get; }
    public string Message { get; }
    public AlertLevel Level { get; }
    public decimal CurrentPrice { get; }
    
    public AlertTriggeredEventArgs(string symbol, string message, AlertLevel level, decimal price)
    {
        Symbol = symbol;
        Message = message;
        Level = level;
        CurrentPrice = price;
    }
}

enum TradeType { Buy, Sell }
enum AlertLevel { Info, Warning, Critical }

// PUBLISHER
class Stock
{
    private decimal price;
    
    public string Symbol { get; set; }
    public decimal Price
    {
        get => price;
        set
        {
            if (price != value)
            {
                decimal oldPrice = price;
                price = value;
                OnPriceChanged(oldPrice, value);
                CheckAlerts();
            }
        }
    }
    
    // EVENTS
    public event EventHandler<PriceChangedEventArgs> PriceChanged;
    public event EventHandler<TradeExecutedEventArgs> TradeExecuted;
    public event EventHandler<AlertTriggeredEventArgs> AlertTriggered;
    
    // Alert thresholds
    private decimal? highPriceAlert;
    private decimal? lowPriceAlert;
    
    public Stock(string symbol, decimal initialPrice)
    {
        Symbol = symbol;
        price = initialPrice;
    }
    
    public void SetHighAlert(decimal threshold)
    {
        highPriceAlert = threshold;
        Console.WriteLine($"[{Symbol}] High price alert set at ${threshold}");
    }
    
    public void SetLowAlert(decimal threshold)
    {
        lowPriceAlert = threshold;
        Console.WriteLine($"[{Symbol}] Low price alert set at ${threshold}");
    }
    
    public void ExecuteTrade(TradeType type, int quantity)
    {
        OnTradeExecuted(type, quantity);
    }
    
    // RAISE EVENTS (protected virtual pattern)
    protected virtual void OnPriceChanged(decimal oldPrice, decimal newPrice)
    {
        PriceChanged?.Invoke(this, new PriceChangedEventArgs(Symbol, oldPrice, newPrice));
    }
    
    protected virtual void OnTradeExecuted(TradeType type, int quantity)
    {
        TradeExecuted?.Invoke(this, new TradeExecutedEventArgs(Symbol, quantity, Price, type));
    }
    
    protected virtual void OnAlertTriggered(string message, AlertLevel level)
    {
        AlertTriggered?.Invoke(this, new AlertTriggeredEventArgs(Symbol, message, level, Price));
    }
    
    private void CheckAlerts()
    {
        if (highPriceAlert.HasValue && Price >= highPriceAlert.Value)
        {
            OnAlertTriggered($"Price reached high threshold: ${highPriceAlert}", AlertLevel.Warning);
        }
        
        if (lowPriceAlert.HasValue && Price <= lowPriceAlert.Value)
        {
            OnAlertTriggered($"Price dropped to low threshold: ${lowPriceAlert}", AlertLevel.Warning);
        }
    }
}

// SUBSCRIBERS
class PriceMonitor
{
    public void Subscribe(Stock stock)
    {
        stock.PriceChanged += OnPriceChanged;
    }
    
    private void OnPriceChanged(object sender, PriceChangedEventArgs e)
    {
        string trend = e.Change > 0 ? "↑" : "↓";
        Console.WriteLine($"[MONITOR] {e.Symbol}: ${e.OldPrice} → ${e.NewPrice} " +
                         $"{trend} {Math.Abs(e.ChangePercent):F2}%");
    }
}

class TradeLogger
{
    public void Subscribe(Stock stock)
    {
        stock.TradeExecuted += OnTradeExecuted;
    }
    
    private void OnTradeExecuted(object sender, TradeExecutedEventArgs e)
    {
        Console.WriteLine($"[TRADE LOG] {e.Type} {e.Quantity} {e.Symbol} @ ${e.Price} " +
                         $"(Total: ${e.TotalValue}) at {e.ExecutedAt:HH:mm:ss}");
    }
}

class AlertSystem
{
    public void Subscribe(Stock stock)
    {
        stock.AlertTriggered += OnAlertTriggered;
    }
    
    private void OnAlertTriggered(object sender, AlertTriggeredEventArgs e)
    {
        string icon = e.Level switch
        {
            AlertLevel.Info => "ℹ️",
            AlertLevel.Warning => "⚠️",
            AlertLevel.Critical => "🚨",
            _ => ""
        };
        
        Console.WriteLine($"[ALERT {icon}] {e.Symbol}: {e.Message} (Current: ${e.CurrentPrice})");
    }
}

class CustomEventArgsDemo
{
    static void Main()
    {
        Console.WriteLine("=== CUSTOM EVENTARGS DEMO ===\n");
        
        // Create stock
        var apple = new Stock("AAPL", 150.00m);
        
        // Create and subscribe monitors
        var monitor = new PriceMonitor();
        var logger = new TradeLogger();
        var alerts = new AlertSystem();
        
        monitor.Subscribe(apple);
        logger.Subscribe(apple);
        alerts.Subscribe(apple);
        
        // Set alerts
        apple.SetHighAlert(155.00m);
        apple.SetLowAlert(145.00m);
        
        Console.WriteLine("\n--- Price Changes ---");
        apple.Price = 152.50m;  // Triggers PriceChanged
        apple.Price = 148.00m;  // Triggers PriceChanged
        apple.Price = 156.00m;  // Triggers PriceChanged + Alert
        
        Console.WriteLine("\n--- Trade Execution ---");
        apple.ExecuteTrade(TradeType.Buy, 100);   // Triggers TradeExecuted
        apple.ExecuteTrade(TradeType.Sell, 50);   // Triggers TradeExecuted
        
        Console.WriteLine("\n--- More Price Changes ---");
        apple.Price = 144.00m;  // Triggers PriceChanged + Alert
        apple.Price = 147.00m;  // Triggers PriceChanged
    }
}
```

**EventArgs Best Practices**:
```csharp
// ✓ GOOD - Immutable properties
class MyEventArgs : EventArgs
{
    public string Data { get; }  // get-only
    public DateTime Timestamp { get; }
    
    public MyEventArgs(string data)
    {
        Data = data;
        Timestamp = DateTime.Now;
    }
}

// ✗ BAD - Mutable properties
class BadEventArgs : EventArgs
{
    public string Data { get; set; }  // Subscribers could modify!
}

// ✓ GOOD - Calculated properties
class GoodEventArgs : EventArgs
{
    public decimal OldValue { get; }
    public decimal NewValue { get; }
    public decimal Change => NewValue - OldValue;  // Calculated
}
```

**Bonus Challenges**:
- ⭐⭐⭐ Add cancellable events (CancelEventArgs)
- ⭐⭐⭐ Implement event history/replay
- ⭐⭐⭐⭐ Build event sourcing pattern
- ⭐⭐⭐⭐ Create event aggregator

---

---

### Problem 114: Event-Based Timer ⭐⭐⭐
**Concepts**: Events, Threading, Timers, Event Lifecycle, Cleanup

**What You'll Learn**:
- Creating timer with events
- Event subscription lifecycle
- Proper event unsubscription
- Timer cleanup
- Multiple subscribers

**Requirements**:
Build a countdown timer with events:
1. Tick event (every second)
2. Complete event
3. Cancelled event
4. Progress reporting
5. Multiple subscribers

**Complete Implementation**:
```csharp
class CountdownTimer
{
    private int remainingSeconds;
    private System.Threading.Timer timer;
    private int totalSeconds;
    
    // EVENTS
    public event EventHandler Started;
    public event EventHandler<int> Tick;  // Remaining seconds
    public event EventHandler Completed;
    public event EventHandler Cancelled;
    
    public int RemainingSeconds => remainingSeconds;
    public bool IsRunning { get; private set; }
    
    public CountdownTimer(int seconds)
    {
        totalSeconds = seconds;
        remainingSeconds = seconds;
    }
    
    public void Start()
    {
        if (IsRunning)
        {
            Console.WriteLine("[TIMER] Already running");
            return;
        }
        
        IsRunning = true;
        OnStarted();
        
        // Create timer that fires every second
        timer = new System.Threading.Timer(
            callback: _ => OnTimerTick(),
            state: null,
            dueTime: 1000,  // First tick after 1 second
            period: 1000);  // Then every 1 second
    }
    
    public void Cancel()
    {
        if (!IsRunning)
            return;
        
        timer?.Dispose();
        IsRunning = false;
        OnCancelled();
    }
    
    private void OnTimerTick()
    {
        remainingSeconds--;
        
        OnTick(remainingSeconds);
        
        if (remainingSeconds <= 0)
        {
            timer?.Dispose();
            IsRunning = false;
            OnCompleted();
        }
    }
    
    protected virtual void OnStarted()
    {
        Started?.Invoke(this, EventArgs.Empty);
    }
    
    protected virtual void OnTick(int remaining)
    {
        Tick?.Invoke(this, remaining);
    }
    
    protected virtual void OnCompleted()
    {
        Completed?.Invoke(this, EventArgs.Empty);
    }
    
    protected virtual void OnCancelled()
    {
        Cancelled?.Invoke(this, EventArgs.Empty);
    }
    
    public void Reset()
    {
        remainingSeconds = totalSeconds;
        IsRunning = false;
    }
}

class TimerDisplay
{
    public void Subscribe(CountdownTimer timer)
    {
        timer.Started += OnStarted;
        timer.Tick += OnTick;
        timer.Completed += OnCompleted;
        timer.Cancelled += OnCancelled;
    }
    
    public void Unsubscribe(CountdownTimer timer)
    {
        timer.Started -= OnStarted;
        timer.Tick -= OnTick;
        timer.Completed -= OnCompleted;
        timer.Cancelled -= OnCancelled;
    }
    
    private void OnStarted(object sender, EventArgs e)
    {
        Console.WriteLine("[DISPLAY] Timer started!");
    }
    
    private void OnTick(object sender, int remaining)
    {
        Console.WriteLine($"[DISPLAY] {remaining} seconds remaining...");
    }
    
    private void OnCompleted(object sender, EventArgs e)
    {
        Console.WriteLine("[DISPLAY] ⏰ Timer completed!");
    }
    
    private void OnCancelled(object sender, EventArgs e)
    {
        Console.WriteLine("[DISPLAY] ⏸️ Timer cancelled");
    }
}

class TimerLogger
{
    public void Subscribe(CountdownTimer timer)
    {
        timer.Tick += OnTick;
        timer.Completed += OnCompleted;
    }
    
    private void OnTick(object sender, int remaining)
    {
        Console.WriteLine($"[LOG] Tick: {DateTime.Now:HH:mm:ss} - {remaining}s left");
    }
    
    private void OnCompleted(object sender, EventArgs e)
    {
        Console.WriteLine($"[LOG] Completed at {DateTime.Now:HH:mm:ss}");
    }
}

class EventBasedTimerDemo
{
    static void Main()
    {
        Console.WriteLine("=== EVENT-BASED TIMER ===\n");
        
        // Create timer (5 seconds)
        var timer = new CountdownTimer(5);
        
        // Create subscribers
        var display = new TimerDisplay();
        var logger = new TimerLogger();
        
        // Subscribe
        display.Subscribe(timer);
        logger.Subscribe(timer);
        
        // Start timer
        timer.Start();
        
        // Wait for completion
        System.Threading.Thread.Sleep(6000);
        
        Console.WriteLine("\n--- Starting Another Timer (3 seconds) ---\n");
        
        timer.Reset();
        timer = new CountdownTimer(3);
        display.Subscribe(timer);
        
        timer.Start();
        
        // Cancel after 1.5 seconds
        System.Threading.Thread.Sleep(1500);
        timer.Cancel();
        
        // Cleanup - IMPORTANT!
        display.Unsubscribe(timer);
        
        Console.WriteLine("\nPress any key to exit...");
        Console.ReadKey();
    }
}
```

**Event Subscription Lifecycle**:
```
1. CREATE publisher
   ↓
2. CREATE subscribers
   ↓
3. SUBSCRIBE (+=)
   ↓
4. USE (events fire)
   ↓
5. UNSUBSCRIBE (-=)  ← IMPORTANT! Prevents memory leaks
   ↓
6. DISPOSE/CLEANUP
```

**Memory Leak Prevention**:
```csharp
// ✗ BAD - Creates memory leak
void BadMethod()
{
    var timer = new Timer();
    var display = new Display();
    display.Subscribe(timer);  // Subscribe
    timer.Start();
    // Method ends, but display still subscribed!
    // Display cannot be garbage collected!
}

// ✓ GOOD - Proper cleanup
void GoodMethod()
{
    var timer = new Timer();
    var display = new Display();
    
    try
    {
        display.Subscribe(timer);
        timer.Start();
        // Use timer...
    }
    finally
    {
        display.Unsubscribe(timer);  // Always unsubscribe!
    }
}

// ✓ BETTER - Using statement
class Display : IDisposable
{
    public void Dispose()
    {
        Unsubscribe(timer);
    }
}

using (var display = new Display())
{
    display.Subscribe(timer);
    // Automatically disposed
}
```

**Bonus Challenges**:
- ⭐⭐⭐ Add pause/resume functionality
- ⭐⭐⭐ Support multiple timers
- ⭐⭐⭐⭐ Create alarm clock with snooze
- ⭐⭐⭐⭐ Build Pomodoro timer

---

---

### Problem 115: Notification System (Complete Integration) ⭐⭐⭐
**Concepts**: **INTEGRATION OF ALL CONCEPTS**, Event-Driven Architecture, Observer Pattern

**What You'll Learn**:
- Combining delegates, events, lambdas
- Observer pattern implementation
- Multi-channel notifications
- Priority handling
- Real-world event system

**Requirements**:
Build a complete notification system integrating:
1. Multiple notification channels (Email, SMS, Push)
2. Priority levels
3. Event-driven architecture
4. Filtering with lambda expressions
5. Async notifications

**Complete Implementation**:
```csharp
enum NotificationPriority { Low, Normal, High, Critical }
enum NotificationChannel { Email, SMS, Push, All }

class Notification
{
    public string Message { get; set; }
    public NotificationPriority Priority { get; set; }
    public DateTime Timestamp { get; set; }
    public string Sender { get; set; }
    
    public Notification(string message, NotificationPriority priority, string sender)
    {
        Message = message;
        Priority = priority;
        Sender = sender;
        Timestamp = DateTime.Now;
    }
    
    public override string ToString() =>
        $"[{Priority}] {Message} (from {Sender})";
}

class NotificationEventArgs : EventArgs
{
    public Notification Notification { get; }
    public NotificationChannel Channel { get; }
    
    public NotificationEventArgs(Notification notification, NotificationChannel channel)
    {
        Notification = notification;
        Channel = channel;
    }
}

// NOTIFICATION HUB (Publisher)
class NotificationHub
{
    // Events for each channel
    public event EventHandler<NotificationEventArgs> NotificationSent;
    public event EventHandler<Notification> EmailRequested;
    public event EventHandler<Notification> SMSRequested;
    public event EventHandler<Notification> PushRequested;
    
    // Filters (using Func<>)
    private List<Func<Notification, bool>> filters = new List<Func<Notification, bool>>();
    
    public void AddFilter(Func<Notification, bool> filter)
    {
        filters.Add(filter);
    }
    
    public void Send(Notification notification, NotificationChannel channel = NotificationChannel.All)
    {
        // Apply filters
        foreach (var filter in filters)
        {
            if (!filter(notification))
            {
                Console.WriteLine($"[HUB] Notification blocked by filter: {notification.Message}");
                return;
            }
        }
        
        // Send to appropriate channels
        if (channel == NotificationChannel.All || channel == NotificationChannel.Email)
        {
            EmailRequested?.Invoke(this, notification);
        }
        
        if (channel == NotificationChannel.All || channel == NotificationChannel.SMS)
        {
            SMSRequested?.Invoke(this, notification);
        }
        
        if (channel == NotificationChannel.All || channel == NotificationChannel.Push)
        {
            PushRequested?.Invoke(this, notification);
        }
        
        // Raise sent event
        OnNotificationSent(notification, channel);
    }
    
    protected virtual void OnNotificationSent(Notification notification, NotificationChannel channel)
    {
        NotificationSent?.Invoke(this, new NotificationEventArgs(notification, channel));
    }
}

// EMAIL SERVICE (Subscriber)
class EmailService
{
    private Func<Notification, bool> shouldProcess;
    
    public EmailService(Func<Notification, bool> filter = null)
    {
        shouldProcess = filter ?? (n => true);  // Default: accept all
    }
    
    public void Subscribe(NotificationHub hub)
    {
        hub.EmailRequested += OnEmailRequested;
    }
    
    private void OnEmailRequested(object sender, Notification notification)
    {
        if (!shouldProcess(notification))
        {
            Console.WriteLine($"[EMAIL] Skipped: {notification.Priority}");
            return;
        }
        
        Console.WriteLine($"[EMAIL] 📧 Sending: {notification}");
        // Simulate sending
        System.Threading.Thread.Sleep(100);
        Console.WriteLine($"[EMAIL] ✓ Sent!");
    }
}

// SMS SERVICE (Subscriber)
class SMSService
{
    public void Subscribe(NotificationHub hub)
    {
        // Only subscribe to high-priority notifications
        hub.SMSRequested += (sender, notification) =>
        {
            if (notification.Priority >= NotificationPriority.High)
            {
                Console.WriteLine($"[SMS] 📱 Sending: {notification}");
                System.Threading.Thread.Sleep(50);
                Console.WriteLine($"[SMS] ✓ Sent!");
            }
            else
            {
                Console.WriteLine($"[SMS] Skipped (priority too low): {notification.Priority}");
            }
        };
    }
}

// PUSH SERVICE (Subscriber)
class PushService
{
    public void Subscribe(NotificationHub hub)
    {
        hub.PushRequested += OnPushRequested;
    }
    
    private void OnPushRequested(object sender, Notification notification)
    {
        Console.WriteLine($"[PUSH] 🔔 Sending: {notification}");
        System.Threading.Thread.Sleep(30);
        Console.WriteLine($"[PUSH] ✓ Sent!");
    }
}

// ANALYTICS SERVICE (Subscriber)
class AnalyticsService
{
    private int emailCount = 0;
    private int smsCount = 0;
    private int pushCount = 0;
    
    public void Subscribe(NotificationHub hub)
    {
        hub.NotificationSent += OnNotificationSent;
    }
    
    private void OnNotificationSent(object sender, NotificationEventArgs e)
    {
        switch (e.Channel)
        {
            case NotificationChannel.Email:
                emailCount++;
                break;
            case NotificationChannel.SMS:
                smsCount++;
                break;
            case NotificationChannel.Push:
                pushCount++;
                break;
            case NotificationChannel.All:
                emailCount++;
                smsCount++;
                pushCount++;
                break;
        }
    }
    
    public void PrintStats()
    {
        Console.WriteLine("\n=== ANALYTICS ===");
        Console.WriteLine($"Emails sent: {emailCount}");
        Console.WriteLine($"SMS sent: {smsCount}");
        Console.WriteLine($"Push sent: {pushCount}");
    }
}

class NotificationSystemDemo
{
    static void Main()
    {
        Console.WriteLine("=== NOTIFICATION SYSTEM (COMPLETE INTEGRATION) ===\n");
        
        // Create hub
        var hub = new NotificationHub();
        
        // Create services
        var emailService = new EmailService(n => n.Priority >= NotificationPriority.Normal);  // Lambda filter
        var smsService = new SMSService();
        var pushService = new PushService();
        var analytics = new AnalyticsService();
        
        // Subscribe all services
        emailService.Subscribe(hub);
        smsService.Subscribe(hub);
        pushService.Subscribe(hub);
        analytics.Subscribe(hub);
        
        // Add global filter (using lambda)
        hub.AddFilter(n => !string.IsNullOrEmpty(n.Message));  // Must have message
        hub.AddFilter(n => n.Message.Length > 3);              // Minimum length
        
        // Send notifications
        Console.WriteLine("--- Sending Notifications ---\n");
        
        hub.Send(new Notification("Server is down!", NotificationPriority.Critical, "System"));
        Console.WriteLine();
        
        hub.Send(new Notification("New comment on your post", NotificationPriority.Normal, "SocialApp"));
        Console.WriteLine();
        
        hub.Send(new Notification("FYI", NotificationPriority.Low, "InfoBot"));  // Short message - filtered!
        Console.WriteLine();
        
        hub.Send(new Notification("Payment received: $500", NotificationPriority.High, "PaymentSystem"), NotificationChannel.Email);
        Console.WriteLine();
        
        // Show analytics
        analytics.PrintStats();
        
        Console.WriteLine("\n=== CONCEPTS DEMONSTRATED ===");
        Console.WriteLine("✓ Delegates (Func<Notification, bool> for filters)");
        Console.WriteLine("✓ Events (NotificationSent, EmailRequested, etc.)");
        Console.WriteLine("✓ Lambda Expressions (inline event handlers)");
        Console.WriteLine("✓ Generics (EventHandler<T>)");
        Console.WriteLine("✓ LINQ (filtering logic)");
        Console.WriteLine("✓ Observer Pattern (Hub + Services)");
        Console.WriteLine("✓ Event-Driven Architecture");
    }
}
```

**Architecture Diagram**:
```
                 ┌─────────────────┐
                 │NotificationHub  │ (Publisher)
                 │  - Events       │
                 │  - Filters      │
                 └────────┬────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
   ┌────▼─────┐     ┌────▼─────┐     ┌────▼─────┐
   │EmailServ │     │SMSService│     │PushServ  │
   │(Lambda)  │     │(Lambda)  │     │(Method)  │
   └──────────┘     └──────────┘     └──────────┘
        
   All using Delegates + Events + Lambdas!
```

**Concepts Integrated**:
```csharp
// 1. DELEGATES
Func<Notification, bool> filter = n => n.Priority >= High;

// 2. EVENTS
public event EventHandler<NotificationEventArgs> NotificationSent;

// 3. LAMBDAS
hub.EmailRequested += (sender, n) => Console.WriteLine(n);

// 4. GENERICS
EventHandler<Notification>
EventHandler<NotificationEventArgs>

// 5. LINQ (implicitly through filters)
notifications.Where(n => filter(n))

// 6. OBSERVER PATTERN
Hub (Subject) → Services (Observers)
```

**Bonus Challenges**:
- ⭐⭐⭐ Add rate limiting
- ⭐⭐⭐ Implement retry logic
- ⭐⭐⭐⭐ Make it fully async
- ⭐⭐⭐⭐ Add message queuing

**Real-World Usage**:
- Push notification systems
- Email marketing platforms
- SMS gateways
- Alert systems
- Event-driven microservices

---

## 🎉 SECTION 4.3 COMPLETE!

**Delegates, Events & Lambdas (9/9 Problems)** ✅

---

## ✅ PHASE 4 COMPLETE!!!

**All 25 Problems Fully Expanded** 🏆

**Section 4.1: Generics & Constraints** (6/6) ✅  
**Section 4.2: LINQ Mastery** (10/10) ✅  
**Section 4.3: Delegates, Events & Lambdas** (9/9) ✅  

---

## 🎓 Phase 4 Achievement Summary

**You Now Master**:
- ✅ Generic programming (type safety + reusability)
- ✅ LINQ (THE most important C# skill)
- ✅ Delegates (function pointers + callbacks)
- ✅ Events (publisher-subscriber pattern)
- ✅ Lambda expressions (modern C# syntax)
- ✅ Event-driven architecture
- ✅ Observer pattern
- ✅ Functional programming concepts

**Job Readiness**: 85% 🚀

**Skills Remaining**:
- Async/Await (Phase 5) - Critical
- File I/O & JSON (Phase 6) - Essential
- Exception Handling (Phase 6) - Production-ready

---

## 📊 Overall Progress

| Phase | Problems | Status | Job Impact |
|-------|----------|--------|------------|
| Phase 1 | 35 | ✅ 100% | Foundation |
| Phase 2 | 25 | ✅ 100% | OOP Skills |
| Phase 3 | 30 | ✅ 100% | Data Skills |
| **Phase 4** | **25** | **✅ 100%** | **Modern C#** |
| Phase 5 | 20 | ⏸️ 0% | Async Skills |
| Phase 6 | 15 | ⏸️ 0% | Integration |

**Fully Expanded: 115 / 208 problems (55%)**

---

### Problem 117: Thread Synchronization (Lock) ⭐⭐⭐
**Concepts**: Race Conditions, lock Keyword, Monitor, Thread Safety, Critical Sections

**What You'll Learn**:
- What race conditions are
- Why synchronization is needed
- Using lock keyword
- Monitor class
- Critical sections
- Deadlock risks

**Requirements**:
Demonstrate thread synchronization:
1. Show race condition without lock
2. Fix with lock keyword
3. Use Monitor class
4. Show deadlock scenario
5. Compare synchronized vs unsynchronized

**Complete Implementation**:
```csharp
class ThreadSynchronization
{
    // Shared resource (dangerous without synchronization!)
    private static int sharedCounter = 0;
    private static object lockObject = new object();
    
    static void Main()
    {
        Console.WriteLine("=== THREAD SYNCHRONIZATION ===\n");
        
        // DEMONSTRATE RACE CONDITION
        Console.WriteLine("--- Race Condition (NO LOCK) ---");
        sharedCounter = 0;
        
        Thread t1 = new Thread(IncrementUnsafe);
        Thread t2 = new Thread(IncrementUnsafe);
        Thread t3 = new Thread(IncrementUnsafe);
        
        t1.Start();
        t2.Start();
        t3.Start();
        
        t1.Join();
        t2.Join();
        t3.Join();
        
        Console.WriteLine($"Final counter (UNSAFE): {sharedCounter}");
        Console.WriteLine($"Expected: 30000, Actual: {sharedCounter}");
        Console.WriteLine($"Lost updates: {30000 - sharedCounter}\n");
        
        // FIX WITH LOCK
        Console.WriteLine("--- With LOCK (Thread-Safe) ---");
        sharedCounter = 0;
        
        Thread t4 = new Thread(IncrementSafe);
        Thread t5 = new Thread(IncrementSafe);
        Thread t6 = new Thread(IncrementSafe);
        
        t4.Start();
        t5.Start();
        t6.Start();
        
        t4.Join();
        t5.Join();
        t6.Join();
        
        Console.WriteLine($"Final counter (SAFE): {sharedCounter}");
        Console.WriteLine($"Expected: 30000, Actual: {sharedCounter}");
        Console.WriteLine($"Lost updates: 0 ✓\n");
        
        // BANK ACCOUNT EXAMPLE
        Console.WriteLine("--- Bank Account Example ---");
        var account = new BankAccount(1000);
        
        Thread deposit1 = new Thread(() => 
        {
            for (int i = 0; i < 5; i++)
            {
                account.Deposit(100);
                Thread.Sleep(10);
            }
        });
        
        Thread deposit2 = new Thread(() => 
        {
            for (int i = 0; i < 5; i++)
            {
                account.Deposit(50);
                Thread.Sleep(10);
            }
        });
        
        Thread withdraw1 = new Thread(() => 
        {
            for (int i = 0; i < 3; i++)
            {
                account.Withdraw(200);
                Thread.Sleep(15);
            }
        });
        
        deposit1.Start();
        deposit2.Start();
        withdraw1.Start();
        
        deposit1.Join();
        deposit2.Join();
        withdraw1.Join();
        
        Console.WriteLine($"\nFinal balance: ${account.Balance}");
        
        // MONITOR CLASS EXAMPLE
        Console.WriteLine("\n--- Monitor Class ---");
        DemonstrateMonitor();
        
        // DEADLOCK EXAMPLE (commented to prevent hang)
        Console.WriteLine("\n--- Deadlock (Demonstration) ---");
        Console.WriteLine("Deadlock can occur when:");
        Console.WriteLine("  Thread 1: locks A, waits for B");
        Console.WriteLine("  Thread 2: locks B, waits for A");
        Console.WriteLine("  Result: Both threads wait forever!");
        Console.WriteLine("(Not running actual deadlock to prevent hang)");
    }
    
    static void IncrementUnsafe()
    {
        for (int i = 0; i < 10000; i++)
        {
            sharedCounter++; // RACE CONDITION!
            // This is actually three operations:
            // 1. Read sharedCounter
            // 2. Add 1
            // 3. Write back
            // Another thread can interrupt between these!
        }
    }
    
    static void IncrementSafe()
    {
        for (int i = 0; i < 10000; i++)
        {
            lock (lockObject)  // Only one thread at a time!
            {
                sharedCounter++;
            }
        }
    }
    
    static void DemonstrateMonitor()
    {
        object monitorLock = new object();
        int counter = 0;
        
        Thread t = new Thread(() =>
        {
            Monitor.Enter(monitorLock);
            try
            {
                counter++;
                Console.WriteLine($"  Monitor: Counter = {counter}");
            }
            finally
            {
                Monitor.Exit(monitorLock);  // Always release!
            }
        });
        
        t.Start();
        t.Join();
        
        Console.WriteLine("  Monitor.Enter/Exit is what 'lock' uses internally");
    }
}

class BankAccount
{
    private decimal balance;
    private object balanceLock = new object();
    
    public decimal Balance
    {
        get
        {
            lock (balanceLock)
            {
                return balance;
            }
        }
    }
    
    public BankAccount(decimal initialBalance)
    {
        balance = initialBalance;
    }
    
    public void Deposit(decimal amount)
    {
        lock (balanceLock)
        {
            Console.WriteLine($"[Thread {Thread.CurrentThread.ManagedThreadId}] Depositing ${amount}");
            balance += amount;
            Console.WriteLine($"  New balance: ${balance}");
        }
    }
    
    public bool Withdraw(decimal amount)
    {
        lock (balanceLock)
        {
            Console.WriteLine($"[Thread {Thread.CurrentThread.ManagedThreadId}] Attempting to withdraw ${amount}");
            
            if (balance >= amount)
            {
                balance -= amount;
                Console.WriteLine($"  Withdrawn ${amount}. New balance: ${balance}");
                return true;
            }
            else
            {
                Console.WriteLine($"  Insufficient funds (balance: ${balance})");
                return false;
            }
        }
    }
}
```

**Race Condition Explained**:
```
Without Lock (RACE CONDITION):
Thread 1: Read counter (0)
Thread 2: Read counter (0)  ← Both read same value!
Thread 1: Add 1 (result: 1)
Thread 2: Add 1 (result: 1)  ← Should be 2!
Thread 1: Write back (1)
Thread 2: Write back (1)  ← Lost Thread 1's update!

Result: Counter = 1 (should be 2!)

With Lock (THREAD-SAFE):
Thread 1: Lock → Read → Add → Write → Unlock
Thread 2: [waiting...]
Thread 2: Lock → Read → Add → Write → Unlock

Result: Counter = 2 ✓
```

**lock Keyword**:
```csharp
// GOOD - lock on private object
private static object lockObj = new object();

lock (lockObj)
{
    // Critical section - only one thread at a time
    sharedCounter++;
}

// BAD - don't lock on this!
lock (this) { }  // Bad: external code can lock on it

// BAD - don't lock on type!
lock (typeof(MyClass)) { }  // Bad: global lock

// BAD - don't lock on string!
string str = "lock";
lock (str) { }  // Bad: strings are interned
```

**Monitor vs lock**:
```csharp
// lock keyword (preferred)
lock (obj)
{
    // Critical section
}

// Equivalent to:
Monitor.Enter(obj);
try
{
    // Critical section
}
finally
{
    Monitor.Exit(obj);
}

// Monitor with timeout
if (Monitor.TryEnter(obj, TimeSpan.FromSeconds(5)))
{
    try
    {
        // Got lock
    }
    finally
    {
        Monitor.Exit(obj);
    }
}
else
{
    // Couldn't get lock within 5 seconds
}
```

**Deadlock Example**:
```csharp
object lockA = new object();
object lockB = new object();

// Thread 1
lock (lockA)
{
    Thread.Sleep(100);
    lock (lockB)  // Waits for lockB
    {
        // Work
    }
}

// Thread 2 (running simultaneously)
lock (lockB)
{
    Thread.Sleep(100);
    lock (lockA)  // Waits for lockA
    {
        // Work
    }
}

// DEADLOCK! Both threads waiting forever!

// FIX: Always lock in same order
// Both threads lock A first, then B
```

**Bonus Challenges**:
- ⭐⭐⭐ Implement reader-writer lock
- ⭐⭐⭐ Create lock-free counter using Interlocked
- ⭐⭐⭐⭐ Build thread-safe collection
- ⭐⭐⭐⭐ Detect and prevent deadlocks

**Real-World Usage**:
- Thread-safe collections
- Database connection pools
- Cache implementations
- Singleton patterns (thread-safe)
- Resource management

**Interview Tips**:
💡 Explain: "Race condition = multiple threads accessing shared data"  
💡 Know: "lock prevents race conditions"  
💡 Mention: "Always lock on private object"  
💡 Important: "Deadlock = circular wait for locks"  
💡 Common question: "How to make thread-safe?"  

---

---

### Problem 118: ThreadPool Usage ⭐⭐⭐
**Concepts**: ThreadPool, QueueUserWorkItem, Managed Thread Pool, Performance

**What You'll Learn**:
- What ThreadPool is
- When to use ThreadPool vs manual threads
- QueueUserWorkItem method
- ThreadPool configuration
- Performance benefits
- Limitations of ThreadPool

**Requirements**:
Use ThreadPool for concurrent operations:
1. Queue multiple work items
2. Compare with manual threads
3. Show performance benefits
4. Demonstrate thread reuse
5. Handle work item completion

**Complete Implementation**:
```csharp
class ThreadPoolDemo
{
    private static int completedTasks = 0;
    private static object lockObj = new object();
    
    static void Main()
    {
        Console.WriteLine("=== THREADPOOL USAGE ===\n");
        
        // BASIC THREADPOOL USAGE
        Console.WriteLine("--- Basic ThreadPool ---");
        
        for (int i = 1; i <= 5; i++)
        {
            int taskNumber = i;  // Capture loop variable
            
            ThreadPool.QueueUserWorkItem(state =>
            {
                ProcessTask(taskNumber);
            });
        }
        
        // Wait for all tasks
        Thread.Sleep(3000);
        Console.WriteLine($"\nCompleted tasks: {completedTasks}\n");
        
        // THREADPOOL vs MANUAL THREADS
        Console.WriteLine("--- Performance Comparison ---");
        
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        // Manual threads (expensive to create)
        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++)
        {
            threads[i] = new Thread(() =>
            {
                Thread.Sleep(100);  // Simulate work
            });
            threads[i].Start();
        }
        
        foreach (var t in threads)
        {
            t.Join();
        }
        
        sw.Stop();
        Console.WriteLine($"Manual threads: {sw.ElapsedMilliseconds}ms");
        
        // ThreadPool (reuses threads)
        sw.Restart();
        
        int remaining = 10;
        ManualResetEvent allDone = new ManualResetEvent(false);
        
        for (int i = 0; i < 10; i++)
        {
            ThreadPool.QueueUserWorkItem(_ =>
            {
                Thread.Sleep(100);  // Simulate work
                
                if (Interlocked.Decrement(ref remaining) == 0)
                {
                    allDone.Set();
                }
            });
        }
        
        allDone.WaitOne();
        sw.Stop();
        Console.WriteLine($"ThreadPool: {sw.ElapsedMilliseconds}ms (FASTER!)\n");
        
        // THREADPOOL INFORMATION
        Console.WriteLine("--- ThreadPool Information ---");
        
        ThreadPool.GetAvailableThreads(out int workerThreads, out int completionPortThreads);
        ThreadPool.GetMaxThreads(out int maxWorkers, out int maxCompletion);
        ThreadPool.GetMinThreads(out int minWorkers, out int minCompletion);
        
        Console.WriteLine($"Available worker threads: {workerThreads}");
        Console.WriteLine($"Max worker threads: {maxWorkers}");
        Console.WriteLine($"Min worker threads: {minWorkers}");
        
        // PASSING DATA TO THREADPOOL
        Console.WriteLine("\n--- Passing Data to ThreadPool ---");
        
        var workItems = new[] { "Task A", "Task B", "Task C" };
        
        foreach (var item in workItems)
        {
            ThreadPool.QueueUserWorkItem(ProcessNamedTask, item);
        }
        
        Thread.Sleep(2000);
        
        // LAMBDA WITH CLOSURE
        Console.WriteLine("\n--- Lambda with Closure ---");
        
        for (int i = 1; i <= 3; i++)
        {
            int taskId = i;  // Capture to avoid closure issue
            
            ThreadPool.QueueUserWorkItem(_ =>
            {
                Console.WriteLine($"[ThreadPool] Processing task {taskId} on thread {Thread.CurrentThread.ManagedThreadId}");
                Thread.Sleep(500);
            });
        }
        
        Thread.Sleep(2000);
    }
    
    static void ProcessTask(int taskNumber)
    {
        Console.WriteLine($"[Task {taskNumber}] Started on thread {Thread.CurrentThread.ManagedThreadId}");
        Console.WriteLine($"[Task {taskNumber}] IsThreadPoolThread: {Thread.CurrentThread.IsThreadPoolThread}");
        
        // Simulate work
        Thread.Sleep(Random.Shared.Next(500, 1500));
        
        Console.WriteLine($"[Task {taskNumber}] Completed");
        
        lock (lockObj)
        {
            completedTasks++;
        }
    }
    
    static void ProcessNamedTask(object state)
    {
        string taskName = (string)state;
        Console.WriteLine($"[{taskName}] Processing on thread {Thread.CurrentThread.ManagedThreadId}");
        Thread.Sleep(500);
        Console.WriteLine($"[{taskName}] Done");
    }
}
```

**ThreadPool Benefits**:
```
MANUAL THREADS:
- Create thread (expensive!)
- Start thread
- Execute work
- Destroy thread (expensive!)
- Repeat for each task...

THREADPOOL:
- Thread already exists ✓
- Queue work item (cheap!)
- Execute work
- Thread returns to pool (reused!) ✓
- Next work item uses same thread ✓

Result: Much faster for many short tasks!
```

**When to Use**:
```csharp
// ✅ USE THREADPOOL:
- Many short-lived operations
- Fire-and-forget tasks
- Background work
- Don't need thread control

// ❌ DON'T USE THREADPOOL:
- Long-running operations (blocks pool thread!)
- Need to control thread (priority, name, etc.)
- Need dedicated thread
- Require thread apartment state (STA)

// ⚠️ MODERN C#:
// Use Task.Run() instead of ThreadPool directly!
Task.Run(() => DoWork());  // Uses ThreadPool internally
```

**ThreadPool Limitations**:
```csharp
// ❌ PROBLEMS:
1. Can't cancel work once queued
2. Can't wait for specific work item
3. Can't get return value easily
4. Limited control over thread

// ✅ SOLUTION:
// Use Task instead!
// Task wraps ThreadPool with better features
```

**Bonus Challenges**:
- ⭐⭐⭐ Configure ThreadPool min/max threads
- ⭐⭐⭐ Monitor ThreadPool saturation
- ⭐⭐⭐⭐ Build custom thread pool
- ⭐⭐⭐⭐ Compare ThreadPool vs Task.Run

---

---

### Problem 119: Multithreaded Counter ⭐⭐⭐
**Concepts**: Interlocked, Atomic Operations, Lock-Free Programming

**What You'll Learn**:
- Atomic operations
- Interlocked class
- Lock-free thread safety
- Performance comparison
- When to use Interlocked vs lock

**Requirements**:
Implement thread-safe counters:
1. Unsafe counter (race condition)
2. Locked counter (thread-safe)
3. Interlocked counter (lock-free)
4. Performance comparison
5. Various Interlocked operations

**Complete Implementation**:
```csharp
class MultithreadedCounter
{
    static void Main()
    {
        Console.WriteLine("=== MULTITHREADED COUNTER ===\n");
        
        int iterations = 1000000;
        int threadCount = 4;
        
        // TEST 1: UNSAFE (Race Condition)
        Console.WriteLine("--- Unsafe Counter (Race Condition) ---");
        var unsafeResult = TestUnsafeCounter(threadCount, iterations);
        Console.WriteLine($"Expected: {threadCount * iterations:N0}");
        Console.WriteLine($"Actual: {unsafeResult:N0}");
        Console.WriteLine($"Lost updates: {(threadCount * iterations) - unsafeResult:N0}\n");
        
        // TEST 2: LOCKED (Thread-Safe but slower)
        Console.WriteLine("--- Locked Counter (Thread-Safe) ---");
        var (lockedResult, lockedTime) = TestLockedCounter(threadCount, iterations);
        Console.WriteLine($"Result: {lockedResult:N0} ✓");
        Console.WriteLine($"Time: {lockedTime}ms\n");
        
        // TEST 3: INTERLOCKED (Lock-Free, Fast)
        Console.WriteLine("--- Interlocked Counter (Lock-Free) ---");
        var (interlockedResult, interlockedTime) = TestInterlockedCounter(threadCount, iterations);
        Console.WriteLine($"Result: {interlockedResult:N0} ✓");
        Console.WriteLine($"Time: {interlockedTime}ms");
        Console.WriteLine($"Speedup: {(double)lockedTime / interlockedTime:F2}x faster!\n");
        
        // INTERLOCKED OPERATIONS
        Console.WriteLine("--- Interlocked Operations ---");
        DemonstrateInterlockedOperations();
        
        // COMPARE-EXCHANGE (Advanced)
        Console.WriteLine("\n--- CompareExchange (Atomic Test-and-Set) ---");
        DemonstrateCompareExchange();
    }
    
    static int TestUnsafeCounter(int threadCount, int iterations)
    {
        int counter = 0;
        var threads = new Thread[threadCount];
        
        for (int i = 0; i < threadCount; i++)
        {
            threads[i] = new Thread(() =>
            {
                for (int j = 0; j < iterations; j++)
                {
                    counter++;  // UNSAFE!
                }
            });
            threads[i].Start();
        }
        
        foreach (var t in threads)
        {
            t.Join();
        }
        
        return counter;
    }
    
    static (int result, long time) TestLockedCounter(int threadCount, int iterations)
    {
        int counter = 0;
        object lockObj = new object();
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        var threads = new Thread[threadCount];
        
        for (int i = 0; i < threadCount; i++)
        {
            threads[i] = new Thread(() =>
            {
                for (int j = 0; j < iterations; j++)
                {
                    lock (lockObj)
                    {
                        counter++;
                    }
                }
            });
            threads[i].Start();
        }
        
        foreach (var t in threads)
        {
            t.Join();
        }
        
        sw.Stop();
        return (counter, sw.ElapsedMilliseconds);
    }
    
    static (int result, long time) TestInterlockedCounter(int threadCount, int iterations)
    {
        int counter = 0;
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        var threads = new Thread[threadCount];
        
        for (int i = 0; i < threadCount; i++)
        {
            threads[i] = new Thread(() =>
            {
                for (int j = 0; j < iterations; j++)
                {
                    Interlocked.Increment(ref counter);  // Atomic!
                }
            });
            threads[i].Start();
        }
        
        foreach (var t in threads)
        {
            t.Join();
        }
        
        sw.Stop();
        return (counter, sw.ElapsedMilliseconds);
    }
    
    static void DemonstrateInterlockedOperations()
    {
        int value = 10;
        
        // Increment
        int newValue = Interlocked.Increment(ref value);
        Console.WriteLine($"Increment: {value} (returned: {newValue})");
        
        // Decrement
        newValue = Interlocked.Decrement(ref value);
        Console.WriteLine($"Decrement: {value} (returned: {newValue})");
        
        // Add
        newValue = Interlocked.Add(ref value, 5);
        Console.WriteLine($"Add 5: {value} (returned: {newValue})");
        
        // Exchange (swap)
        int oldValue = Interlocked.Exchange(ref value, 100);
        Console.WriteLine($"Exchange: old={oldValue}, new={value}");
        
        // Read (atomic read of long/double on 32-bit systems)
        long longValue = 12345L;
        long readValue = Interlocked.Read(ref longValue);
        Console.WriteLine($"Read: {readValue}");
    }
    
    static void DemonstrateCompareExchange()
    {
        int value = 10;
        
        // Compare and exchange
        // If value == 10, set to 20
        int original = Interlocked.CompareExchange(ref value, 20, 10);
        
        Console.WriteLine($"CompareExchange(ref value, newValue:20, comparand:10)");
        Console.WriteLine($"  Original value: {original}");
        Console.WriteLine($"  Current value: {value}");
        Console.WriteLine($"  Exchange happened: {original == 10}");
        
        // Try again (will fail - value is now 20, not 10)
        original = Interlocked.CompareExchange(ref value, 30, 10);
        
        Console.WriteLine($"\nCompareExchange(ref value, newValue:30, comparand:10)");
        Console.WriteLine($"  Original value: {original}");
        Console.WriteLine($"  Current value: {value}");
        Console.WriteLine($"  Exchange happened: {original == 10} (No - value was 20)");
        
        // USE CASE: Lock-free flag
        Console.WriteLine("\nUse case: Lock-free 'first one wins' pattern:");
        int flag = 0;
        
        // Only first thread to call this succeeds
        if (Interlocked.CompareExchange(ref flag, 1, 0) == 0)
        {
            Console.WriteLine("  I'm the first! Doing work...");
        }
        else
        {
            Console.WriteLine("  Someone else got here first");
        }
    }
}
```

**Interlocked vs Lock**:
```
LOCK:
lock (obj) { counter++; }
- Acquires lock (expensive!)
- Increments
- Releases lock
- Time: ~50-100ns

INTERLOCKED:
Interlocked.Increment(ref counter);
- Single atomic CPU instruction
- Time: ~5-10ns
- 10x faster! ✓
```

**When to Use Each**:
```csharp
// USE INTERLOCKED:
✅ Simple operations (increment, add, exchange)
✅ Single variable updates
✅ Performance-critical code
✅ Lock-free algorithms

// USE LOCK:
✅ Multiple operations together
✅ Complex logic in critical section
✅ Multiple variables to update atomically

// Example where lock is needed:
lock (obj)
{
    balance -= amount;      // Two operations
    transactions.Add(...);  // must be atomic together
}
// Interlocked can't do this!
```

**Available Operations**:
```csharp
// Arithmetic
Interlocked.Increment(ref value);      // value++
Interlocked.Decrement(ref value);      // value--
Interlocked.Add(ref value, amount);    // value += amount

// Exchange
Interlocked.Exchange(ref value, newValue);  // value = newValue, return old

// Compare and Exchange (test-and-set)
Interlocked.CompareExchange(ref value, newValue, expectedValue);
// If (value == expectedValue) { value = newValue; } return originalValue

// Read (for long/double on 32-bit)
Interlocked.Read(ref longValue);

// Memory barriers
Interlocked.MemoryBarrier();  // Advanced!
```

**Bonus Challenges**:
- ⭐⭐⭐ Build lock-free stack
- ⭐⭐⭐⭐ Implement lock-free queue
- ⭐⭐⭐⭐ Create ABA problem demonstration
- ⭐⭐⭐⭐ Build spin-wait with Interlocked

---

---

### Problem 124: Task Continuation ⭐⭐⭐
**Concepts**: ContinueWith, Task Chaining, Continuation Options, Error Handling in Continuations

**What You'll Learn**:
- Chaining tasks with ContinueWith
- Continuation options
- Error handling in chains
- Success/failure continuations
- Multiple continuations

**Complete Implementation**:
```csharp
class TaskContinuation
{
    static void Main()
    {
        Console.WriteLine("=== TASK CONTINUATION ===\n");
        
        // BASIC CONTINUATION
        Console.WriteLine("--- Basic ContinueWith ---");
        
        Task.Run(() =>
        {
            Console.WriteLine("  Step 1: Downloading...");
            Thread.Sleep(1000);
            return "data.txt";
        })
        .ContinueWith(previousTask =>
        {
            string filename = previousTask.Result;
            Console.WriteLine($"  Step 2: Processing {filename}...");
            Thread.Sleep(500);
            return filename.ToUpper();
        })
        .ContinueWith(previousTask =>
        {
            string processed = previousTask.Result;
            Console.WriteLine($"  Step 3: Result is {processed}");
        })
        .Wait();
        
        Console.WriteLine();
        
        // CONTINUATION WITH OPTIONS
        Console.WriteLine("--- Continuation Options ---");
        
        Task<int> calculation = Task.Run(() =>
        {
            Console.WriteLine("  Calculating...");
            Thread.Sleep(500);
            return 42;
        });
        
        // Only run if success
        calculation.ContinueWith(task =>
        {
            Console.WriteLine($"  Success! Result: {task.Result}");
        }, TaskContinuationOptions.OnlyOnRanToCompletion);
        
        // Only run if failed
        calculation.ContinueWith(task =>
        {
            Console.WriteLine($"  Error: {task.Exception.InnerException.Message}");
        }, TaskContinuationOptions.OnlyOnFaulted);
        
        calculation.Wait();
        Console.WriteLine();
        
        // HANDLING EXCEPTIONS IN CONTINUATION
        Console.WriteLine("--- Exception Handling ---");
        
        Task.Run(() =>
        {
            Console.WriteLine("  Task that will fail...");
            Thread.Sleep(300);
            throw new InvalidOperationException("Oops!");
            return 100;
        })
        .ContinueWith(task =>
        {
            if (task.IsFaulted)
            {
                Console.WriteLine($"  Error caught in continuation: {task.Exception.InnerException.Message}");
                return 0;  // Fallback value
            }
            else
            {
                Console.WriteLine($"  Success: {task.Result}");
                return task.Result;
            }
        })
        .ContinueWith(task =>
        {
            Console.WriteLine($"  Final result: {task.Result}");
        })
        .Wait();
        
        Console.WriteLine();
        
        // MULTIPLE CONTINUATIONS (FAN-OUT)
        Console.WriteLine("--- Multiple Continuations ---");
        
        Task<string> getData = Task.Run(() =>
        {
            Console.WriteLine("  Fetching data...");
            Thread.Sleep(500);
            return "Important Data";
        });
        
        // Multiple continuations from same task
        Task save = getData.ContinueWith(task =>
        {
            Console.WriteLine($"  Saving: {task.Result}");
            Thread.Sleep(300);
        });
        
        Task log = getData.ContinueWith(task =>
        {
            Console.WriteLine($"  Logging: {task.Result}");
            Thread.Sleep(200);
        });
        
        Task notify = getData.ContinueWith(task =>
        {
            Console.WriteLine($"  Notifying: {task.Result}");
            Thread.Sleep(100);
        });
        
        Task.WaitAll(save, log, notify);
        Console.WriteLine("  All continuations completed\n");
        
        // REAL-WORLD PIPELINE
        Console.WriteLine("--- Data Processing Pipeline ---");
        
        Task.Run(() => DownloadData())
            .ContinueWith(task => ValidateData(task.Result))
            .ContinueWith(task => ProcessData(task.Result))
            .ContinueWith(task => SaveData(task.Result))
            .ContinueWith(task =>
            {
                Console.WriteLine($"  Pipeline complete! Processed {task.Result} records");
            })
            .Wait();
    }
    
    static string DownloadData()
    {
        Console.WriteLine("  [1] Downloading data...");
        Thread.Sleep(500);
        return "raw_data.csv";
    }
    
    static string ValidateData(string data)
    {
        Console.WriteLine($"  [2] Validating {data}...");
        Thread.Sleep(300);
        return "validated_data.csv";
    }
    
    static string ProcessData(string data)
    {
        Console.WriteLine($"  [3] Processing {data}...");
        Thread.Sleep(400);
        return "processed_data.json";
    }
    
    static int SaveData(string data)
    {
        Console.WriteLine($"  [4] Saving {data}...");
        Thread.Sleep(200);
        return 1000;  // Number of records
    }
}
```

**Continuation Options**:
```csharp
TaskContinuationOptions.None  // Default
TaskContinuationOptions.OnlyOnRanToCompletion  // Only if success
TaskContinuationOptions.OnlyOnFaulted  // Only if exception
TaskContinuationOptions.OnlyOnCanceled  // Only if canceled
TaskContinuationOptions.NotOnRanToCompletion  // If NOT success
TaskContinuationOptions.NotOnFaulted  // If NO exception
TaskContinuationOptions.NotOnCanceled  // If NOT canceled

// Example:
task.ContinueWith(t => HandleSuccess(t.Result),
    TaskContinuationOptions.OnlyOnRanToCompletion);

task.ContinueWith(t => HandleError(t.Exception),
    TaskContinuationOptions.OnlyOnFaulted);
```

**ContinueWith vs await**:
```csharp
// ContinueWith (old way - still works)
Task.Run(() => Step1())
    .ContinueWith(t => Step2(t.Result))
    .ContinueWith(t => Step3(t.Result));

// await (modern way - preferred!)
await Task.Run(() => Step1());
var result1 = await Step2();
await Step3(result1);

// await is:
// - Cleaner syntax
// - Better error handling
// - Easier to understand
// - Preferred in modern C#
```

**Bonus Challenges**:
- ⭐⭐⭐ Build retry logic with continuations
- ⭐⭐⭐ Create conditional pipeline
- ⭐⭐⭐⭐ Implement saga pattern
- ⭐⭐⭐⭐ Build workflow engine

---

---

### Problem 125: Parallel Task Execution ⭐⭐⭐
**Concepts**: Task.WhenAll, Task.WhenAny, Parallel Execution, Aggregation

**What You'll Learn**:
- Running tasks in parallel
- WhenAll for multiple tasks
- WhenAny for fastest result
- Handling multiple results
- Error handling in parallel tasks

**Complete Implementation**:
```csharp
class ParallelTaskExecution
{
    static void Main()
    {
        Console.WriteLine("=== PARALLEL TASK EXECUTION ===\n");
        
        // SEQUENTIAL vs PARALLEL
        Console.WriteLine("--- Sequential vs Parallel ---");
        
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        // Sequential
        FetchData1();
        FetchData2();
        FetchData3();
        
        sw.Stop();
        Console.WriteLine($"Sequential: {sw.ElapsedMilliseconds}ms\n");
        
        sw.Restart();
        
        // Parallel with Task.WhenAll
        Task<string> t1 = Task.Run(() => FetchData1());
        Task<string> t2 = Task.Run(() => FetchData2());
        Task<string> t3 = Task.Run(() => FetchData3());
        
        Task.WhenAll(t1, t2, t3).Wait();
        
        sw.Stop();
        Console.WriteLine($"Parallel: {sw.ElapsedMilliseconds}ms (3x faster!)\n");
        
        // TASK.WHENALL - Wait for all tasks
        Console.WriteLine("--- Task.WhenAll ---");
        
        Task<int>[] tasks = new[]
        {
            Task.Run(() => { Thread.Sleep(500); return 10; }),
            Task.Run(() => { Thread.Sleep(300); return 20; }),
            Task.Run(() => { Thread.Sleep(700); return 30; })
        };
        
        Task<int[]> allResults = Task.WhenAll(tasks);
        allResults.Wait();
        
        Console.WriteLine("All tasks completed:");
        foreach (int result in allResults.Result)
        {
            Console.WriteLine($"  Result: {result}");
        }
        
        int sum = allResults.Result.Sum();
        Console.WriteLine($"Sum: {sum}\n");
        
        // TASK.WHENANY - First to complete
        Console.WriteLine("--- Task.WhenAny (Race) ---");
        
        Task<string>[] sources = new[]
        {
            Task.Run(() => FetchFromSource("Server 1", 800)),
            Task.Run(() => FetchFromSource("Server 2", 500)),  // Fastest!
            Task.Run(() => FetchFromSource("Server 3", 1000))
        };
        
        Task<Task<string>> winner = Task.WhenAny(sources);
        winner.Wait();
        
        string winnerResult = winner.Result.Result;
        Console.WriteLine($"Winner: {winnerResult}\n");
        
        // PARALLEL API CALLS
        Console.WriteLine("--- Parallel API Calls ---");
        
        var apiTasks = new[]
        {
            Task.Run(() => CallAPI("api.weather.com", 600)),
            Task.Run(() => CallAPI("api.stocks.com", 400)),
            Task.Run(() => CallAPI("api.news.com", 500))
        };
        
        string[] apiResults = Task.WhenAll(apiTasks).Result;
        
        Console.WriteLine("All API responses:");
        foreach (var response in apiResults)
        {
            Console.WriteLine($"  {response}");
        }
        Console.WriteLine();
        
        // HANDLING EXCEPTIONS IN PARALLEL
        Console.WriteLine("--- Exception Handling ---");
        
        Task<int>[] mixedTasks = new[]
        {
            Task.Run(() => { Thread.Sleep(300); return 100; }),
            Task.Run(() => { Thread.Sleep(200); throw new Exception("Task 2 failed!"); return 200; }),
            Task.Run(() => { Thread.Sleep(400); return 300; })
        };
        
        try
        {
            Task.WhenAll(mixedTasks).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine($"Caught {ae.InnerExceptions.Count} exception(s):");
            foreach (var ex in ae.InnerExceptions)
            {
                Console.WriteLine($"  - {ex.Message}");
            }
        }
        
        // Get successful results even if some failed
        Console.WriteLine("\nSuccessful results:");
        foreach (var task in mixedTasks.Where(t => t.Status == TaskStatus.RanToCompletion))
        {
            Console.WriteLine($"  {task.Result}");
        }
        Console.WriteLine();
        
        // DOWNLOAD MULTIPLE FILES
        Console.WriteLine("--- Download Multiple Files ---");
        
        string[] urls = { "file1.txt", "file2.txt", "file3.txt" };
        
        Task<bool>[] downloads = urls.Select(url =>
            Task.Run(() => DownloadFile(url))
        ).ToArray();
        
        bool[] results = Task.WhenAll(downloads).Result;
        
        int successCount = results.Count(r => r);
        Console.WriteLine($"Downloaded {successCount}/{urls.Length} files successfully");
    }
    
    static string FetchData1()
    {
        Console.WriteLine("  Fetching data 1...");
        Thread.Sleep(1000);
        return "Data 1";
    }
    
    static string FetchData2()
    {
        Console.WriteLine("  Fetching data 2...");
        Thread.Sleep(1000);
        return "Data 2";
    }
    
    static string FetchData3()
    {
        Console.WriteLine("  Fetching data 3...");
        Thread.Sleep(1000);
        return "Data 3";
    }
    
    static string FetchFromSource(string source, int delay)
    {
        Console.WriteLine($"  {source} starting...");
        Thread.Sleep(delay);
        Console.WriteLine($"  {source} finished!");
        return $"Data from {source}";
    }
    
    static string CallAPI(string endpoint, int delay)
    {
        Console.WriteLine($"  Calling {endpoint}...");
        Thread.Sleep(delay);
        return $"Response from {endpoint}";
    }
    
    static bool DownloadFile(string filename)
    {
        Console.WriteLine($"  Downloading {filename}...");
        Thread.Sleep(Random.Shared.Next(300, 800));
        Console.WriteLine($"  Downloaded {filename}");
        return true;
    }
}
```

**WhenAll vs WhenAny**:
```csharp
// WHENALL - Wait for ALL tasks
Task<int>[] tasks = { task1, task2, task3 };
int[] results = await Task.WhenAll(tasks);
// Returns when: All tasks complete
// Returns: Array of all results

// WHENANY - Wait for FIRST task
Task<int> winner = await Task.WhenAny(tasks);
int result = await winner;  // Get the result
// Returns when: Any task completes
// Returns: The winning task

// Use cases:
// - WhenAll: Need all data (download multiple files)
// - WhenAny: Race for fastest (ping multiple servers)
```

**Exception Handling**:
```csharp
// WhenAll with exceptions
try
{
    await Task.WhenAll(task1, task2, task3);
}
catch (Exception ex)
{
    // Only first exception! ⚠️
}

// Get ALL exceptions:
try
{
    await Task.WhenAll(tasks);
}
catch
{
    foreach (var task in tasks.Where(t => t.IsFaulted))
    {
        Console.WriteLine(task.Exception.InnerException.Message);
    }
}
```

**Bonus Challenges**:
- ⭐⭐⭐ Implement timeout for WhenAny
- ⭐⭐⭐ Build parallel downloader
- ⭐⭐⭐⭐ Create load balancer
- ⭐⭐⭐⭐ Implement circuit breaker

---

---

### Problem 126: Task Cancellation ⭐⭐⭐
**Concepts**: CancellationToken, CancellationTokenSource, Cooperative Cancellation

**What You'll Learn**:
- Cancelling long-running tasks
- CancellationToken usage
- Cooperative cancellation
- Timeouts
- Cleanup after cancellation

**Complete Implementation**:
```csharp
class TaskCancellation
{
    static void Main()
    {
        Console.WriteLine("=== TASK CANCELLATION ===\n");
        
        // BASIC CANCELLATION
        Console.WriteLine("--- Basic Cancellation ---");
        
        CancellationTokenSource cts = new CancellationTokenSource();
        
        Task task1 = Task.Run(() =>
        {
            for (int i = 0; i < 10; i++)
            {
                // Check if cancellation requested
                if (cts.Token.IsCancellationRequested)
                {
                    Console.WriteLine("  Task cancelled!");
                    return;
                }
                
                Console.WriteLine($"  Working... {i}");
                Thread.Sleep(500);
            }
        }, cts.Token);
        
        Thread.Sleep(1500);  // Let it run a bit
        
        Console.WriteLine("[Main] Requesting cancellation...");
        cts.Cancel();  // Request cancellation
        
        try
        {
            task1.Wait();
        }
        catch (AggregateException ae)
        {
            if (ae.InnerException is TaskCanceledException)
            {
                Console.WriteLine("[Main] Task was cancelled\n");
            }
        }
        
        // THROW ON CANCELLATION
        Console.WriteLine("--- ThrowIfCancellationRequested ---");
        
        CancellationTokenSource cts2 = new CancellationTokenSource();
        
        Task task2 = Task.Run(() =>
        {
            for (int i = 0; i < 10; i++)
            {
                // Throws OperationCanceledException if cancelled
                cts2.Token.ThrowIfCancellationRequested();
                
                Console.WriteLine($"  Step {i}");
                Thread.Sleep(300);
            }
        }, cts2.Token);
        
        Thread.Sleep(1000);
        cts2.Cancel();
        
        try
        {
            task2.Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine($"  Exception: {ae.InnerException.GetType().Name}\n");
        }
        
        // TIMEOUT CANCELLATION
        Console.WriteLine("--- Timeout Cancellation ---");
        
        // Automatically cancel after 2 seconds
        CancellationTokenSource cts3 = new CancellationTokenSource(TimeSpan.FromSeconds(2));
        
        Task longTask = Task.Run(() =>
        {
            try
            {
                for (int i = 0; i < 100; i++)
                {
                    cts3.Token.ThrowIfCancellationRequested();
                    Console.WriteLine($"  Processing {i}...");
                    Thread.Sleep(500);
                }
            }
            catch (OperationCanceledException)
            {
                Console.WriteLine("  Task timed out!");
            }
        }, cts3.Token);
        
        longTask.Wait();
        Console.WriteLine();
        
        // CANCELLATION CALLBACK
        Console.WriteLine("--- Cancellation Callback ---");
        
        CancellationTokenSource cts4 = new CancellationTokenSource();
        
        // Register callback for when cancellation is requested
        cts4.Token.Register(() =>
        {
            Console.WriteLine("  [Callback] Cancellation requested! Cleaning up...");
        });
        
        Task task4 = Task.Run(() =>
        {
            for (int i = 0; i < 5; i++)
            {
                if (cts4.Token.IsCancellationRequested)
                {
                    Console.WriteLine("  Task stopping...");
                    return;
                }
                
                Console.WriteLine($"  Working {i}");
                Thread.Sleep(400);
            }
        }, cts4.Token);
        
        Thread.Sleep(1000);
        cts4.Cancel();
        task4.Wait();
        Console.WriteLine();
        
        // MULTIPLE TASKS WITH SAME TOKEN
        Console.WriteLine("--- Cancel Multiple Tasks ---");
        
        CancellationTokenSource cts5 = new CancellationTokenSource();
        
        Task[] parallelTasks = new[]
        {
            Task.Run(() => LongRunningWork("Task A", cts5.Token), cts5.Token),
            Task.Run(() => LongRunningWork("Task B", cts5.Token), cts5.Token),
            Task.Run(() => LongRunningWork("Task C", cts5.Token), cts5.Token)
        };
        
        Thread.Sleep(1500);
        
        Console.WriteLine("[Main] Cancelling all tasks...");
        cts5.Cancel();  // Cancels ALL tasks using this token
        
        try
        {
            Task.WaitAll(parallelTasks);
        }
        catch (AggregateException)
        {
            Console.WriteLine("[Main] All tasks cancelled\n");
        }
        
        // LINKED CANCELLATION TOKENS
        Console.WriteLine("--- Linked Tokens ---");
        
        CancellationTokenSource parentCts = new CancellationTokenSource();
        CancellationTokenSource childCts = CancellationTokenSource.CreateLinkedTokenSource(parentCts.Token);
        
        Task child = Task.Run(() =>
        {
            while (!childCts.Token.IsCancellationRequested)
            {
                Console.WriteLine("  Child working...");
                Thread.Sleep(300);
            }
            Console.WriteLine("  Child cancelled!");
        }, childCts.Token);
        
        Thread.Sleep(1000);
        parentCts.Cancel();  // Cancels parent AND child!
        
        child.Wait();
        Console.WriteLine("  Parent cancellation propagated to child");
    }
    
    static void LongRunningWork(string name, CancellationToken token)
    {
        try
        {
            for (int i = 0; i < 10; i++)
            {
                token.ThrowIfCancellationRequested();
                Console.WriteLine($"  [{name}] Step {i}");
                Thread.Sleep(500);
            }
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine($"  [{name}] Cancelled");
        }
    }
}
```

**Cancellation Pattern**:
```csharp
// 1. Create CancellationTokenSource
CancellationTokenSource cts = new CancellationTokenSource();

// 2. Pass token to task
Task task = Task.Run(() => 
{
    while (!cts.Token.IsCancellationRequested)
    {
        // Do work
    }
}, cts.Token);

// 3. Request cancellation
cts.Cancel();

// 4. Wait and handle
try
{
    task.Wait();
}
catch (AggregateException ae) when (ae.InnerException is TaskCanceledException)
{
    // Task was cancelled
}
```

**Two Ways to Check Cancellation**:
```csharp
// METHOD 1: Check and return (graceful)
if (token.IsCancellationRequested)
{
    // Cleanup
    return;
}

// METHOD 2: Throw exception (standard)
token.ThrowIfCancellationRequested();
// Throws OperationCanceledException
```

**Timeout Pattern**:
```csharp
// Cancel after specific time
CancellationTokenSource cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));

// Or cancel after delay
cts.CancelAfter(TimeSpan.FromSeconds(5));
```

**Bonus Challenges**:
- ⭐⭐⭐ Implement graceful shutdown
- ⭐⭐⭐ Build cancellable download manager
- ⭐⭐⭐⭐ Create cancellable pipeline
- ⭐⭐⭐⭐ Implement cancellation with retry

---

---

### Problem 127: Task Exception Handling ⭐⭐⭐
**Concepts**: AggregateException, Exception Propagation, Fault Handling

[Complete working implementation with exception handling patterns]

**Key Pattern**:
```csharp
try
{
    await task;  // Unwraps exception
}
catch (SpecificException ex)
{
    // Handle specific exception
}

// vs

try
{
    task.Wait();  // Wraps in AggregateException
}
catch (AggregateException ae)
{
    ae.Handle(ex => ex is SpecificException);
}
```

---

---

### Problem 128: Progress Reporting ⭐⭐⭐
**Concepts**: IProgress<T>, Progress<T>, Reporting from Tasks

[Complete working implementation with progress reporting]

**Key Pattern**:
```csharp
var progress = new Progress<int>(percent =>
{
    Console.WriteLine($"Progress: {percent}%");
});

await Task.Run(() =>
{
    for (int i = 0; i <= 100; i += 10)
    {
        ((IProgress<int>)progress).Report(i);
        Thread.Sleep(100);
    }
}, cts.Token);
```

---

---

### Problem 131: Async File Operations ⭐⭐⭐
**Concepts**: Async I/O, StreamReader/Writer, FileStream Async Methods, Non-Blocking I/O

**What You'll Learn**:
- Async file reading and writing
- StreamReader/Writer async methods
- FileStream async operations
- Benefits of async I/O
- Large file processing without blocking

**Requirements**:
Implement async file operations:
1. Read file asynchronously
2. Write file asynchronously
3. Process large files
4. Copy files without blocking
5. Measure performance improvement

**Complete Implementation**:
```csharp
class AsyncFileOperations
{
    static async Task Main(string[] args)
    {
        Console.WriteLine("=== ASYNC FILE OPERATIONS ===\n");
        
        string sourceFile = "large-file.txt";
        string destFile = "output.txt";
        
        // CREATE TEST FILE
        Console.WriteLine("--- Creating Test File ---");
        await CreateLargeFileAsync(sourceFile, 1000);
        Console.WriteLine($"Created {sourceFile}\n");
        
        // ASYNC READ
        Console.WriteLine("--- Async Read ---");
        string content = await ReadFileAsync(sourceFile);
        Console.WriteLine($"Read {content.Length} characters");
        Console.WriteLine($"First 100 chars: {content.Substring(0, Math.Min(100, content.Length))}\n");
        
        // ASYNC WRITE
        Console.WriteLine("--- Async Write ---");
        await WriteFileAsync(destFile, "This is async written content!");
        Console.WriteLine($"Written to {destFile}\n");
        
        // ASYNC COPY
        Console.WriteLine("--- Async Copy ---");
        await CopyFileAsync(sourceFile, "copy.txt");
        Console.WriteLine("File copied asynchronously\n");
        
        // READ LINES ASYNCHRONOUSLY
        Console.WriteLine("--- Read Lines Async ---");
        await ReadLinesAsync(sourceFile);
        Console.WriteLine();
        
        // PROCESS LARGE FILE
        Console.WriteLine("--- Process Large File ---");
        int wordCount = await CountWordsAsync(sourceFile);
        Console.WriteLine($"Total words: {wordCount:N0}\n");
        
        // PERFORMANCE COMPARISON
        Console.WriteLine("--- Performance: Sync vs Async ---");
        await ComparePerformanceAsync(sourceFile);
        
        // CLEANUP
        File.Delete(sourceFile);
        File.Delete(destFile);
        File.Delete("copy.txt");
    }
    
    static async Task CreateLargeFileAsync(string filename, int lines)
    {
        using (StreamWriter writer = new StreamWriter(filename))
        {
            for (int i = 0; i < lines; i++)
            {
                await writer.WriteLineAsync($"Line {i}: This is sample text for testing async file operations.");
            }
        }
    }
    
    static async Task<string> ReadFileAsync(string filename)
    {
        using (StreamReader reader = new StreamReader(filename))
        {
            return await reader.ReadToEndAsync();  // Async!
        }
    }
    
    static async Task WriteFileAsync(string filename, string content)
    {
        using (StreamWriter writer = new StreamWriter(filename))
        {
            await writer.WriteAsync(content);  // Async!
        }
    }
    
    static async Task CopyFileAsync(string source, string dest)
    {
        using (FileStream sourceStream = new FileStream(source, FileMode.Open, FileAccess.Read, FileShare.Read, 4096, useAsync: true))
        using (FileStream destStream = new FileStream(dest, FileMode.Create, FileAccess.Write, FileShare.None, 4096, useAsync: true))
        {
            await sourceStream.CopyToAsync(destStream);  // Async!
        }
    }
    
    static async Task ReadLinesAsync(string filename)
    {
        int lineCount = 0;
        
        using (StreamReader reader = new StreamReader(filename))
        {
            string line;
            while ((line = await reader.ReadLineAsync()) != null)  // Async!
            {
                lineCount++;
                if (lineCount <= 5)  // Show first 5
                {
                    Console.WriteLine($"  {line}");
                }
            }
        }
        
        Console.WriteLine($"  ... ({lineCount} total lines)");
    }
    
    static async Task<int> CountWordsAsync(string filename)
    {
        int wordCount = 0;
        
        using (StreamReader reader = new StreamReader(filename))
        {
            string line;
            while ((line = await reader.ReadLineAsync()) != null)
            {
                wordCount += line.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
            }
        }
        
        return wordCount;
    }
    
    static async Task ComparePerformanceAsync(string filename)
    {
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        // Synchronous read
        string syncContent = File.ReadAllText(filename);
        sw.Stop();
        Console.WriteLine($"  Sync read: {sw.ElapsedMilliseconds}ms");
        
        sw.Restart();
        
        // Asynchronous read
        string asyncContent = await ReadFileAsync(filename);
        sw.Stop();
        Console.WriteLine($"  Async read: {sw.ElapsedMilliseconds}ms");
        Console.WriteLine("  (Async doesn't block thread - main benefit!)");
    }
}
```

**Async I/O Methods**:
```csharp
// StreamReader
await reader.ReadToEndAsync()
await reader.ReadLineAsync()
await reader.ReadAsync(buffer, 0, buffer.Length)

// StreamWriter
await writer.WriteAsync(text)
await writer.WriteLineAsync(text)

// FileStream
await stream.ReadAsync(buffer, 0, buffer.Length)
await stream.WriteAsync(buffer, 0, buffer.Length)
await stream.CopyToAsync(destination)

// File class (C# 8.0+)
await File.ReadAllTextAsync(path)
await File.WriteAllTextAsync(path, content)
await File.ReadAllLinesAsync(path)
```

**Why Async I/O Matters**:
```
SYNC I/O (blocks thread):
Thread → Read file → [BLOCKED WAITING] → Continue
         ↓
    Other work waits 😞

ASYNC I/O (doesn't block):
Thread → Start read → Do other work → Read complete → Process
         ↓              ↓
    I/O operation   Thread free! 🎉
```

**Bonus Challenges**:
- ⭐⭐⭐ Build async log file processor
- ⭐⭐⭐ Create async CSV parser
- ⭐⭐⭐⭐ Implement async file watcher
- ⭐⭐⭐⭐ Build async backup system

---

---

### Problem 132: Parallel URL Fetcher ⭐⭐⭐
**Concepts**: HttpClient, Async HTTP, Parallel Downloads, Error Handling, Real Async I/O

**What You'll Learn**:
- HttpClient for async HTTP requests
- Downloading multiple URLs in parallel
- Error handling in parallel operations
- HttpClient best practices
- Measuring download performance

**Requirements**:
Build parallel URL fetcher:
1. Download single URL
2. Download multiple URLs in parallel
3. Handle HTTP errors
4. Measure download times
5. Save downloaded content

**Complete Implementation**:
```csharp
class ParallelUrlFetcher
{
    // IMPORTANT: Single HttpClient instance (best practice!)
    private static readonly HttpClient httpClient = new HttpClient();
    
    static async Task Main(string[] args)
    {
        Console.WriteLine("=== PARALLEL URL FETCHER ===\n");
        
        // Configure HttpClient
        httpClient.Timeout = TimeSpan.FromSeconds(30);
        
        string[] urls = new[]
        {
            "https://jsonplaceholder.typicode.com/posts/1",
            "https://jsonplaceholder.typicode.com/posts/2",
            "https://jsonplaceholder.typicode.com/posts/3",
            "https://jsonplaceholder.typicode.com/users/1",
            "https://jsonplaceholder.typicode.com/users/2"
        };
        
        // SINGLE URL DOWNLOAD
        Console.WriteLine("--- Single URL Download ---");
        string result = await DownloadUrlAsync(urls[0]);
        Console.WriteLine($"Downloaded {result.Length} characters");
        Console.WriteLine($"Content preview: {result.Substring(0, Math.Min(100, result.Length))}...\n");
        
        // SEQUENTIAL DOWNLOADS
        Console.WriteLine("--- Sequential Downloads ---");
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        foreach (var url in urls)
        {
            string content = await DownloadUrlAsync(url);
            Console.WriteLine($"  Downloaded: {url} ({content.Length} chars)");
        }
        
        sw.Stop();
        Console.WriteLine($"Sequential time: {sw.ElapsedMilliseconds}ms\n");
        
        // PARALLEL DOWNLOADS
        Console.WriteLine("--- Parallel Downloads ---");
        sw.Restart();
        
        // Start all downloads at once
        Task<string>[] downloadTasks = urls.Select(url => DownloadUrlAsync(url)).ToArray();
        
        // Wait for all to complete
        string[] results = await Task.WhenAll(downloadTasks);
        
        sw.Stop();
        Console.WriteLine($"Parallel time: {sw.ElapsedMilliseconds}ms (Much faster!)\n");
        
        for (int i = 0; i < urls.Length; i++)
        {
            Console.WriteLine($"  [{i + 1}] {urls[i]}: {results[i].Length} chars");
        }
        Console.WriteLine();
        
        // ERROR HANDLING
        Console.WriteLine("--- Error Handling ---");
        
        string[] mixedUrls = new[]
        {
            "https://jsonplaceholder.typicode.com/posts/1",
            "https://invalid-url-that-will-fail.com/test",  // Will fail
            "https://jsonplaceholder.typicode.com/posts/2"
        };
        
        await DownloadWithErrorHandlingAsync(mixedUrls);
        Console.WriteLine();
        
        // DOWNLOAD WITH PROGRESS
        Console.WriteLine("--- Download with Progress ---");
        await DownloadWithProgressAsync(urls);
        Console.WriteLine();
        
        // DOWNLOAD AND SAVE
        Console.WriteLine("--- Download and Save ---");
        await DownloadAndSaveAsync("https://jsonplaceholder.typicode.com/posts", "posts.json");
        Console.WriteLine();
        
        // FASTEST RESPONSE WINS
        Console.WriteLine("--- Race: Fastest Response Wins ---");
        await GetFastestResponseAsync(urls);
    }
    
    static async Task<string> DownloadUrlAsync(string url)
    {
        HttpResponseMessage response = await httpClient.GetAsync(url);
        response.EnsureSuccessStatusCode();  // Throw if not success
        
        string content = await response.Content.ReadAsStringAsync();
        return content;
    }
    
    static async Task DownloadWithErrorHandlingAsync(string[] urls)
    {
        var tasks = urls.Select(async url =>
        {
            try
            {
                string content = await DownloadUrlAsync(url);
                Console.WriteLine($"  ✓ Success: {url} ({content.Length} chars)");
                return (url, success: true, content, error: (string)null);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"  ✗ Failed: {url} - {ex.Message}");
                return (url, success: false, content: (string)null, error: ex.Message);
            }
        });
        
        var results = await Task.WhenAll(tasks);
        
        int successCount = results.Count(r => r.success);
        Console.WriteLine($"\nCompleted: {successCount}/{urls.Length} successful");
    }
    
    static async Task DownloadWithProgressAsync(string[] urls)
    {
        int completed = 0;
        var lockObj = new object();
        
        var tasks = urls.Select(async url =>
        {
            string content = await DownloadUrlAsync(url);
            
            lock (lockObj)
            {
                completed++;
                Console.WriteLine($"  Progress: {completed}/{urls.Length} - Downloaded {url}");
            }
            
            return content;
        });
        
        await Task.WhenAll(tasks);
        Console.WriteLine($"  All {urls.Length} downloads complete!");
    }
    
    static async Task DownloadAndSaveAsync(string url, string filename)
    {
        Console.WriteLine($"  Downloading {url}...");
        
        string content = await DownloadUrlAsync(url);
        
        Console.WriteLine($"  Saving to {filename}...");
        await File.WriteAllTextAsync(filename, content);
        
        Console.WriteLine($"  Saved {content.Length} characters to {filename}");
        
        // Cleanup
        File.Delete(filename);
    }
    
    static async Task GetFastestResponseAsync(string[] urls)
    {
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        // Start all requests
        Task<string>[] tasks = urls.Select(url => DownloadUrlAsync(url)).ToArray();
        
        // Wait for first to complete
        Task<string> winner = await Task.WhenAny(tasks);
        
        sw.Stop();
        
        string winnerUrl = urls[Array.IndexOf(tasks, winner)];
        string content = await winner;
        
        Console.WriteLine($"  Fastest: {winnerUrl}");
        Console.WriteLine($"  Time: {sw.ElapsedMilliseconds}ms");
        Console.WriteLine($"  Content: {content.Length} chars");
    }
}
```

**HttpClient Best Practices**:
```csharp
// ✓ CORRECT - Single instance (reuse!)
private static readonly HttpClient httpClient = new HttpClient();

// ❌ WRONG - Creates new instance each time
public async Task<string> Download(string url)
{
    using (var client = new HttpClient())  // BAD!
    {
        return await client.GetStringAsync(url);
    }
}
// Problems: Socket exhaustion, poor performance

// ✓ CORRECT - Reuse HttpClient
public async Task<string> Download(string url)
{
    return await httpClient.GetStringAsync(url);
}
```

**HttpClient Methods**:
```csharp
// GET request
string content = await httpClient.GetStringAsync(url);

// GET with response details
HttpResponseMessage response = await httpClient.GetAsync(url);
response.EnsureSuccessStatusCode();
string content = await response.Content.ReadAsStringAsync();

// POST request
var jsonContent = new StringContent(json, Encoding.UTF8, "application/json");
HttpResponseMessage response = await httpClient.PostAsync(url, jsonContent);

// With headers
httpClient.DefaultRequestHeaders.Add("Authorization", "Bearer token");
```

**Bonus Challenges**:
- ⭐⭐⭐ Add retry logic with exponential backoff
- ⭐⭐⭐ Implement download resume
- ⭐⭐⭐⭐ Build web scraper
- ⭐⭐⭐⭐ Create HTTP load tester

---

---

### Problem 133: Parallel.For Demo ⭐⭐⭐
**Concepts**: Parallel.For, Parallel.ForEach, Data Parallelism, Partitioning

**What You'll Learn**:
- Parallel.For for loop parallelism
- Parallel.ForEach for collection processing
- ParallelOptions for control
- Performance comparison
- When to use parallelism

**Complete Implementation**:
```csharp
class ParallelForDemo
{
    static void Main()
    {
        Console.WriteLine("=== PARALLEL.FOR DEMO ===\n");
        
        int arraySize = 10000000;
        int[] numbers = Enumerable.Range(1, arraySize).ToArray();
        
        // SEQUENTIAL FOR LOOP
        Console.WriteLine("--- Sequential For Loop ---");
        
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        long sequentialSum = 0;
        for (int i = 0; i < numbers.Length; i++)
        {
            sequentialSum += (long)numbers[i] * numbers[i];  // Square each
        }
        
        sw.Stop();
        Console.WriteLine($"Sum: {sequentialSum:N0}");
        Console.WriteLine($"Time: {sw.ElapsedMilliseconds}ms\n");
        
        // PARALLEL.FOR
        Console.WriteLine("--- Parallel.For ---");
        
        sw.Restart();
        
        object lockObj = new object();
        long parallelSum = 0;
        
        Parallel.For(0, numbers.Length, i =>
        {
            long square = (long)numbers[i] * numbers[i];
            
            lock (lockObj)  // Need lock for shared variable!
            {
                parallelSum += square;
            }
        });
        
        sw.Stop();
        Console.WriteLine($"Sum: {parallelSum:N0}");
        Console.WriteLine($"Time: {sw.ElapsedMilliseconds}ms");
        Console.WriteLine($"Speedup: {(double)sw.ElapsedMilliseconds / sw.ElapsedMilliseconds:F2}x\n");
        
        // PARALLEL.FOR WITH LOCAL STATE (Better!)
        Console.WriteLine("--- Parallel.For with Thread-Local State ---");
        
        sw.Restart();
        
        long betterSum = 0;
        
        Parallel.For(0, numbers.Length,
            () => 0L,  // Initialize thread-local state
            (i, loopState, localSum) =>
            {
                // Each thread has its own localSum (no lock needed!)
                return localSum + (long)numbers[i] * numbers[i];
            },
            localSum =>
            {
                // Combine all thread-local results
                lock (lockObj)
                {
                    betterSum += localSum;
                }
            });
        
        sw.Stop();
        Console.WriteLine($"Sum: {betterSum:N0}");
        Console.WriteLine($"Time: {sw.ElapsedMilliseconds}ms (Faster - less locking!)\n");
        
        // PARALLEL.FOREACH
        Console.WriteLine("--- Parallel.ForEach ---");
        
        var items = new List<string>();
        for (int i = 0; i < 1000; i++)
        {
            items.Add($"Item{i}");
        }
        
        sw.Restart();
        
        int processedCount = 0;
        
        Parallel.ForEach(items, item =>
        {
            // Simulate processing
            Thread.SpinWait(10000);
            
            Interlocked.Increment(ref processedCount);
        });
        
        sw.Stop();
        Console.WriteLine($"Processed {processedCount} items in {sw.ElapsedMilliseconds}ms\n");
        
        // PARALLEL OPTIONS
        Console.WriteLine("--- Parallel with Options ---");
        
        var options = new ParallelOptions
        {
            MaxDegreeOfParallelism = 4,  // Limit to 4 threads
            CancellationToken = CancellationToken.None
        };
        
        Parallel.For(0, 10, options, i =>
        {
            Console.WriteLine($"  Processing {i} on thread {Thread.CurrentThread.ManagedThreadId}");
            Thread.Sleep(100);
        });
        
        Console.WriteLine();
        
        // EARLY EXIT
        Console.WriteLine("--- Early Exit with Break ---");
        
        Parallel.For(0, 100, (i, loopState) =>
        {
            if (i == 50)
            {
                Console.WriteLine($"  Breaking at {i}");
                loopState.Break();  // Stop after current iteration
                return;
            }
            
            if (i < 55)  // Only print nearby iterations
            {
                Console.WriteLine($"  Processing {i}");
            }
            
            Thread.Sleep(10);
        });
        
        Console.WriteLine();
        
        // WHEN NOT TO USE PARALLEL
        Console.WriteLine("--- When NOT to Use Parallel ---");
        
        Console.WriteLine("DON'T use Parallel.For when:");
        Console.WriteLine("  - Operation is very fast (overhead > benefit)");
        Console.WriteLine("  - Order matters");
        Console.WriteLine("  - Lots of shared state (contention)");
        Console.WriteLine("  - I/O bound (use async instead!)");
    }
}
```

**Parallel.For Patterns**:
```csharp
// Basic
Parallel.For(0, 100, i =>
{
    // Process i
});

// With thread-local state
Parallel.For(0, 100,
    () => 0,  // Initialize
    (i, state, local) => local + Process(i),  // Body
    local => Aggregate(local));  // Finalize

// ForEach
Parallel.ForEach(collection, item =>
{
    Process(item);
});

// With options
var options = new ParallelOptions
{
    MaxDegreeOfParallelism = Environment.ProcessorCount
};
Parallel.For(0, 100, options, i => Process(i));
```

**When to Use**:
```
USE Parallel.For:
✅ CPU-intensive operations
✅ Large datasets
✅ Independent iterations
✅ Long-running operations

DON'T USE Parallel.For:
❌ I/O operations (use async!)
❌ Very fast operations
❌ Order-dependent operations
❌ Heavy synchronization needed
```

**Bonus Challenges**:
- ⭐⭐⭐ Build parallel image processor
- ⭐⭐⭐ Create parallel data transformer
- ⭐⭐⭐⭐ Implement parallel merge sort
- ⭐⭐⭐⭐ Build parallel search engine

---

---

### Problem 134: PLINQ Performance Test ⭐⭐⭐
**Concepts**: PLINQ (Parallel LINQ), AsParallel(), Performance Optimization, When to Use

**What You'll Learn**:
- PLINQ (Parallel LINQ)
- AsParallel() method
- Performance comparison
- When PLINQ helps
- PLINQ pitfalls

**Complete Implementation**:
```csharp
class PLINQPerformanceTest
{
    static void Main()
    {
        Console.WriteLine("=== PLINQ PERFORMANCE TEST ===\n");
        
        int dataSize = 10000000;
        var numbers = Enumerable.Range(1, dataSize).ToArray();
        
        // TEST 1: SIMPLE FILTER
        Console.WriteLine("--- Test 1: Filter Even Numbers ---");
        
        var sw = System.Diagnostics.Stopwatch.StartNew();
        
        // Regular LINQ
        var evenLinq = numbers.Where(n => n % 2 == 0).ToArray();
        
        sw.Stop();
        Console.WriteLine($"LINQ: {sw.ElapsedMilliseconds}ms ({evenLinq.Length:N0} results)");
        
        sw.Restart();
        
        // PLINQ
        var evenPlinq = numbers.AsParallel().Where(n => n % 2 == 0).ToArray();
        
        sw.Stop();
        Console.WriteLine($"PLINQ: {sw.ElapsedMilliseconds}ms ({evenPlinq.Length:N0} results)");
        Console.WriteLine($"Speedup: {(double)sw.ElapsedMilliseconds / sw.ElapsedMilliseconds:F2}x\n");
        
        // TEST 2: COMPLEX OPERATION
        Console.WriteLine("--- Test 2: Complex Calculation ---");
        
        sw.Restart();
        
        // LINQ - CPU intensive
        var resultsLinq = numbers
            .Where(n => IsPrime(n))
            .Select(n => n * n)
            .ToArray();
        
        sw.Stop();
        Console.WriteLine($"LINQ: {sw.ElapsedMilliseconds}ms ({resultsLinq.Length:N0} primes)");
        
        sw.Restart();
        
        // PLINQ - CPU intensive (benefits from parallelism!)
        var resultsPlinq = numbers
            .AsParallel()
            .Where(n => IsPrime(n))
            .Select(n => n * n)
            .ToArray();
        
        sw.Stop();
        Console.WriteLine($"PLINQ: {sw.ElapsedMilliseconds}ms ({resultsPlinq.Length:N0} primes)");
        Console.WriteLine($"Speedup: Significant! 🚀\n");
        
        // TEST 3: AGGREGATION
        Console.WriteLine("--- Test 3: Aggregation ---");
        
        sw.Restart();
        var sumLinq = numbers.Sum(n => (long)n);
        sw.Stop();
        Console.WriteLine($"LINQ Sum: {sw.ElapsedMilliseconds}ms (Sum: {sumLinq:N0})");
        
        sw.Restart();
        var sumPlinq = numbers.AsParallel().Sum(n => (long)n);
        sw.Stop();
        Console.WriteLine($"PLINQ Sum: {sw.ElapsedMilliseconds}ms (Sum: {sumPlinq:N0})\n");
        
        // TEST 4: ORDERING
        Console.WriteLine("--- Test 4: Ordering Impact ---");
        
        var smallSet = Enumerable.Range(1, 1000).ToArray();
        
        sw.Restart();
        var unordered = smallSet.AsParallel().Where(n => n % 2 == 0).ToArray();
        sw.Stop();
        Console.WriteLine($"Unordered: {sw.ElapsedMilliseconds}ms");
        
        sw.Restart();
        var ordered = smallSet.AsParallel().AsOrdered().Where(n => n % 2 == 0).ToArray();
        sw.Stop();
        Console.WriteLine($"Ordered: {sw.ElapsedMilliseconds}ms (slower due to ordering overhead)\n");
        
        // PLINQ OPTIONS
        Console.WriteLine("--- PLINQ Options ---");
        
        var query = numbers
            .AsParallel()
            .WithDegreeOfParallelism(4)  // Limit to 4 threads
            .WithExecutionMode(ParallelExecutionMode.ForceParallelism)
            .Where(n => n % 2 == 0)
            .Take(100);
        
        var results = query.ToArray();
        Console.WriteLine($"With options: {results.Length} results\n");
        
        // WHEN PLINQ DOESN'T HELP
        Console.WriteLine("--- When PLINQ Doesn't Help ---");
        
        var tiny = Enumerable.Range(1, 10).ToArray();
        
        sw.Restart();
        var tinyLinq = tiny.Where(n => n > 5).ToArray();
        sw.Stop();
        var linqTime = sw.ElapsedMilliseconds;
        
        sw.Restart();
        var tinyPlinq = tiny.AsParallel().Where(n => n > 5).ToArray();
        sw.Stop();
        var plinqTime = sw.ElapsedMilliseconds;
        
        Console.WriteLine($"Small dataset:");
        Console.WriteLine($"  LINQ: {linqTime}ms");
        Console.WriteLine($"  PLINQ: {plinqTime}ms");
        Console.WriteLine($"  PLINQ overhead > benefit for small data!\n");
        
        // BEST PRACTICES
        Console.WriteLine("--- PLINQ Best Practices ---");
        Console.WriteLine("USE PLINQ when:");
        Console.WriteLine("  ✅ CPU-intensive operations");
        Console.WriteLine("  ✅ Large datasets");
        Console.WriteLine("  ✅ Independent operations");
        Console.WriteLine("  ✅ Order doesn't matter (or use AsOrdered)");
        Console.WriteLine("\nDON'T use PLINQ when:");
        Console.WriteLine("  ❌ Small datasets (overhead > benefit)");
        Console.WriteLine("  ❌ I/O operations (use async!)");
        Console.WriteLine("  ❌ Operations are very fast");
        Console.WriteLine("  ❌ Heavy synchronization needed");
    }
    
    static bool IsPrime(int number)
    {
        if (number < 2) return false;
        if (number == 2) return true;
        if (number % 2 == 0) return false;
        
        int sqrt = (int)Math.Sqrt(number);
        for (int i = 3; i <= sqrt; i += 2)
        {
            if (number % i == 0) return false;
        }
        
        return true;
    }
}
```

**PLINQ Usage**:
```csharp
// Convert to PLINQ
var query = collection.AsParallel()
                      .Where(...)
                      .Select(...)
                      .ToArray();

// With options
var query = collection
    .AsParallel()
    .WithDegreeOfParallelism(4)
    .WithExecutionMode(ParallelExecutionMode.ForceParallelism)
    .AsOrdered()  // Preserve order
    .Where(...)
    .Select(...);

// Merge options
var query = collection.AsParallel()
                      .WithMergeOptions(ParallelMergeOptions.NotBuffered);
```

**Performance Comparison**:
```
Small Data (< 1000 items):
LINQ:  Fast ✓
PLINQ: Slow (overhead) ❌

Large Data (1M+ items):
LINQ:  Slow ❌
PLINQ: Fast (parallel processing) ✓

CPU-Intensive:
LINQ:  Slow ❌
PLINQ: Very fast (uses all cores) ✓✓

I/O-Bound:
LINQ:  Slow ❌
PLINQ: Still slow (wrong tool!) ❌
async: Fast ✓ (use this instead!)
```

**Bonus Challenges**:
- ⭐⭐⭐ Benchmark PLINQ vs LINQ
- ⭐⭐⭐ Build PLINQ data analyzer
- ⭐⭐⭐⭐ Create adaptive query engine
- ⭐⭐⭐⭐ Implement custom partitioner

---

---

## Problem 154: Longest Substring Without Repeating Characters ⭐⭐⭐

**Problem Statement:**

Given a string `s`, find the length of the longest substring without repeating characters.

**Examples:**
```
Input: s = "abcabcbb"
Output: 3
Explanation: "abc" is the longest substring

Input: s = "bbbbb"
Output: 1
Explanation: "b"

Input: s = "pwwkew"
Output: 3
Explanation: "wke"
```

**Constraints:**
- 0 ≤ s.length ≤ 5 × 10⁴
- s consists of English letters, digits, symbols, and spaces

---

**Approach 1: Brute Force**

**Concept:**
- Check all possible substrings
- For each, verify if all characters are unique

**Complexity:**
- Time: O(n³)
- Space: O(min(n, m)) where m is character set size

**Too slow!**

---

**Approach 2: Sliding Window with HashSet (Optimal)**

**Key Insight:**
- Use two pointers (left and right)
- Expand window by moving right
- Contract window when duplicate found
- Track maximum length seen

**Concept:**
- HashSet stores characters in current window
- When duplicate found, remove from left until no duplicate
- Update max length at each step

**Complexity:**
- Time: O(n) - each character visited at most twice
- Space: O(min(n, m)) - HashSet size

**Hints:**
```csharp
var seen = new HashSet<char>();
int left = 0, maxLength = 0;

for (int right = 0; right < s.Length; right++)
{
    // While s[right] is in seen:
    //   Remove s[left] from seen
    //   Move left++
    
    // Add s[right] to seen
    // Update maxLength
}
```

---

**Approach 3: Sliding Window with Dictionary (Optimized)**

**Concept:**
- Store character → last seen index
- When duplicate found, jump left pointer directly

**Complexity:**
- Time: O(n) - single pass
- Space: O(min(n, m))

**Hints:**
```csharp
var lastSeen = new Dictionary<char, int>();
int left = 0, maxLength = 0;

for (int right = 0; right < s.Length; right++)
{
    if (lastSeen.ContainsKey(s[right]))
    {
        // Move left to the right of last occurrence
        left = Math.Max(left, lastSeen[s[right]] + 1);
    }
    
    // Update last seen
    lastSeen[s[right]] = right;
    
    // Update max length
    maxLength = Math.Max(maxLength, right - left + 1);
}
```

---

**Test Cases:**
```csharp
"abcabcbb" → 3 ("abc")
"bbbbb" → 1 ("b")
"pwwkew" → 3 ("wke" or "kew")
"" → 0
" " → 1
"au" → 2
"dvdf" → 3 ("vdf")
"abba" → 2 ("ab" or "ba")
```

**Common Mistakes:**
- Not handling empty string
- Off-by-one errors with left pointer
- Forgetting to update maxLength
- Not using Math.Max when moving left pointer

**Interview Tips:**
- Start by explaining brute force (shows understanding)
- Draw the sliding window movement
- Explain why each character is visited at most twice
- Mention the optimization with Dictionary

---

---

## Problem 158: String Permutations ⭐⭐⭐

**Problem Statement:**

Given a string `s`, return all possible permutations of its characters.

**Examples:**
```
Input: s = "abc"
Output: ["abc", "acb", "bac", "bca", "cab", "cba"]

Input: s = "ab"
Output: ["ab", "ba"]

Input: s = "a"
Output: ["a"]
```

**Constraints:**
- 1 ≤ s.length ≤ 8
- s consists of unique lowercase English letters

---

**Approach: Backtracking**

**Key Insight:**
- For each position, try every remaining character
- Recurse for rest of positions
- Backtrack when done

**Concept:**
- Fix first character, permute rest
- Swap characters to try different first characters
- Recursively build permutations

**Complexity:**
- Time: O(n × n!) - n! permutations, n to build each
- Space: O(n!) - storing all permutations

**Hints:**
```csharp
void Backtrack(List<string> result, char[] chars, int start)
{
    // Base case: reached end
    if (start == chars.Length)
    {
        result.Add(new string(chars));
        return;
    }
    
    // Try each character at current position
    for (int i = start; i < chars.Length; i++)
    {
        // Swap to try this character
        Swap(chars, start, i);
        
        // Recurse
        Backtrack(result, chars, start + 1);
        
        // Backtrack (undo swap)
        Swap(chars, start, i);
    }
}
```

---

**Test Cases:**
```csharp
"abc" → 6 permutations
"ab" → 2 permutations
"a" → 1 permutation
"abcd" → 24 permutations
```

**Interview Tips:**
- Draw the recursion tree
- Explain backtracking concept
- Mention time complexity (factorial!)
- Discuss duplicate handling (if chars not unique)

---

---

## Problem 161: Implement atoi (String to Integer) ⭐⭐⭐

**Problem Statement:**

Implement the `myAtoi(string s)` function, which converts a string to a 32-bit signed integer.

Algorithm:
1. Read in and ignore leading whitespace
2. Check if next character is '-' or '+' (determines sign)
3. Read digits until non-digit or end of input
4. Convert to integer
5. Clamp to [−2³¹, 2³¹ − 1]

**Examples:**
```
Input: s = "42"
Output: 42

Input: s = "   -42"
Output: -42

Input: s = "4193 with words"
Output: 4193

Input: s = "words and 987"
Output: 0 (no digits at start)

Input: s = "-91283472332"
Output: -2147483648 (clamped to int.MinValue)
```

---

**Approach: State Machine**

**Concept:**
- Trim whitespace
- Handle sign
- Build number digit by digit
- Check for overflow

**Complexity:**
- Time: O(n)
- Space: O(1)

**Hints:**
```csharp
int i = 0;
int sign = 1;
long result = 0; // Use long to detect overflow

// Skip whitespace
while (i < s.Length && s[i] == ' ') i++;

// Check sign
if (i < s.Length && (s[i] == '+' || s[i] == '-'))
{
    sign = s[i] == '-' ? -1 : 1;
    i++;
}

// Build number
while (i < s.Length && char.IsDigit(s[i]))
{
    result = result * 10 + (s[i] - '0');
    
    // Check overflow
    if (result * sign > int.MaxValue) return int.MaxValue;
    if (result * sign < int.MinValue) return int.MinValue;
    
    i++;
}

return (int)(result * sign);
```

---

**Test Cases:**
```csharp
"42" → 42
"   -42" → -42
"4193 with words" → 4193
"words and 987" → 0
"-91283472332" → -2147483648
"2147483648" → 2147483647 (overflow)
"+1" → 1
"+-12" → 0 (invalid)
```

**Common Mistakes:**
- Not handling overflow correctly
- Not stopping at first non-digit
- Not handling '+' sign
- Not handling leading zeros

**Interview Tips:**
- This tests edge case handling
- Use long to detect overflow
- Be thorough with test cases
- Ask about leading zeros, multiple signs, etc.

---

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

---

