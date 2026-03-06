# C# LINQ (Language Integrated Query) Quick Reference

---

## What is LINQ? (C# 3.0+)

**LINQ** = Language Integrated Query

**Definition:** Query collections using SQL-like syntax directly in C#.

**Benefits:**

- ✅ **Type-safe queries** - Compile-time checking
- ✅ **IntelliSense support** - Auto-completion
- ✅ **Consistent syntax** - Same queries for arrays, lists, databases, XML
- ✅ **Readable code** - Declarative instead of imperative
- ✅ **Powerful operations** - Filter, sort, group, join with minimal code

### LINQ Providers

**LINQ to Objects** - Query in-memory collections (arrays, lists, etc.)
```csharp
int[] numbers = { 1, 2, 3, 4, 5 };
var evens = numbers.Where(n => n % 2 == 0);
```

**LINQ to SQL** - Query SQL Server databases
```csharp
var customers = from c in db.Customers
                where c.City == "London"
                select c;
```

**LINQ to Entities (Entity Framework)** - Query databases via EF
```csharp
var products = context.Products.Where(p => p.Price > 100);
```

**LINQ to XML** - Query XML documents
```csharp
var names = from e in doc.Descendants("Customer")
            select e.Element("Name").Value;
```

---

## Query Syntax vs Method Syntax

### Side-by-Side Comparison

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// QUERY SYNTAX (SQL-like)
var queryResult = from n in numbers
                  where n % 2 == 0
                  orderby n descending
                  select n * 2;

// METHOD SYNTAX (Extension methods)
var methodResult = numbers
    .Where(n => n % 2 == 0)
    .OrderByDescending(n => n)
    .Select(n => n * 2);

// Both produce: [20, 16, 12, 8, 4]
```

### When to Use Each

**Query Syntax:**

- ✅ More readable for complex queries with joins/grouping
- ✅ Familiar to SQL developers
- ✅ Better for multiple operations
- ❌ Doesn't support all LINQ operators

**Method Syntax:**

- ✅ Supports all LINQ operators
- ✅ Can chain easily
- ✅ Better for single operations
- ✅ More flexible

**Recommendation:** Use method syntax (it's more common in modern C#)

### Mixing Both Syntaxes

```csharp
// Query syntax with method chaining
var result = (from n in numbers
              where n > 5
              select n)
             .Take(3)
             .ToList();

// Parentheses required when chaining after query syntax
```

---

## LINQ Operators by Category

### 1. Filtering Operators

#### Where - Filter elements

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Method syntax
var evens = numbers.Where(n => n % 2 == 0);
// [2, 4, 6, 8, 10]

// Query syntax
var evens = from n in numbers
            where n % 2 == 0
            select n;

// Multiple conditions
var result = numbers.Where(n => n > 3 && n < 8);
// [4, 5, 6, 7]

// With index
var result = numbers.Where((n, index) => index % 2 == 0);
// [1, 3, 5, 7, 9] (elements at even indices)
```

#### OfType\<T\> - Filter by type

```csharp
object[] mixed = { 1, "two", 3, "four", 5 };

var integers = mixed.OfType<int>();
// [1, 3, 5]

var strings = mixed.OfType<string>();
// ["two", "four"]
```

---

### 2. Projection Operators

#### Select - Transform elements

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// Simple transformation
var doubled = numbers.Select(n => n * 2);
// [2, 4, 6, 8, 10]

// Project to different type
var strings = numbers.Select(n => n.ToString());
// ["1", "2", "3", "4", "5"]

// Project to anonymous type
var result = numbers.Select(n => new { Number = n, Square = n * n });
// [{ Number: 1, Square: 1 }, { Number: 2, Square: 4 }, ...]

// With index
var result = numbers.Select((n, index) => $"[{index}] = {n}");
// ["[0] = 1", "[1] = 2", ...]

// Query syntax
var doubled = from n in numbers
              select n * 2;
```

#### SelectMany - Flatten nested collections

```csharp
List<List<int>> nested = new List<List<int>>
{
    new List<int> { 1, 2, 3 },
    new List<int> { 4, 5 },
    new List<int> { 6, 7, 8, 9 }
};

// Flatten
var flat = nested.SelectMany(list => list);
// [1, 2, 3, 4, 5, 6, 7, 8, 9]

// Real-world example
class Order
{
    public List<string> Items { get; set; }
}

List<Order> orders = new List<Order>
{
    new Order { Items = new List<string> { "Apple", "Banana" } },
    new Order { Items = new List<string> { "Orange" } }
};

var allItems = orders.SelectMany(o => o.Items);
// ["Apple", "Banana", "Orange"]

// With transformation
var result = orders.SelectMany(o => o.Items, 
                               (order, item) => new { Order = order, Item = item });
```

---

### 3. Sorting Operators

#### OrderBy / OrderByDescending

```csharp
int[] numbers = { 5, 2, 8, 1, 9, 3 };

// Ascending
var ascending = numbers.OrderBy(n => n);
// [1, 2, 3, 5, 8, 9]

// Descending
var descending = numbers.OrderByDescending(n => n);
// [9, 8, 5, 3, 2, 1]

// Query syntax
var ascending = from n in numbers
                orderby n
                select n;

var descending = from n in numbers
                 orderby n descending
                 select n;
```

#### ThenBy / ThenByDescending - Secondary sort

```csharp
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

List<Person> people = new List<Person>
{
    new Person { Name = "John", Age = 25 },
    new Person { Name = "Jane", Age = 30 },
    new Person { Name = "Bob", Age = 25 }
};

// Sort by Age, then by Name
var sorted = people.OrderBy(p => p.Age)
                   .ThenBy(p => p.Name);
// Bob(25), John(25), Jane(30)

// Query syntax
var sorted = from p in people
             orderby p.Age, p.Name
             select p;

// Multiple levels
var sorted = people.OrderBy(p => p.Age)
                   .ThenByDescending(p => p.Name);
```

#### Reverse - Reverse order

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

var reversed = numbers.Reverse();
// [5, 4, 3, 2, 1]
```

---

### 4. Grouping Operators

#### GroupBy - Group by key

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Group by even/odd
var grouped = numbers.GroupBy(n => n % 2 == 0 ? "Even" : "Odd");

foreach (var group in grouped)
{
    Console.WriteLine($"{group.Key}:");
    foreach (var num in group)
    {
        Console.WriteLine($"  {num}");
    }
}
// Odd: 1, 3, 5, 7, 9
// Even: 2, 4, 6, 8, 10

// Real-world example
class Product
{
    public string Category { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

var products = new List<Product>
{
    new Product { Category = "Electronics", Name = "Laptop", Price = 999 },
    new Product { Category = "Electronics", Name = "Phone", Price = 699 },
    new Product { Category = "Books", Name = "C# Guide", Price = 39 }
};

var byCategory = products.GroupBy(p => p.Category);

foreach (var group in byCategory)
{
    Console.WriteLine($"\n{group.Key}:");
    foreach (var product in group)
    {
        Console.WriteLine($"  {product.Name}: ${product.Price}");
    }
}

// Query syntax
var grouped = from n in numbers
              group n by n % 2 == 0 ? "Even" : "Odd";

// With projection
var result = numbers.GroupBy(n => n % 2, n => n * 10);
// Groups: 0 (even) => [20, 40, 60, ...], 1 (odd) => [10, 30, 50, ...]
```

#### ToLookup - Create lookup dictionary

```csharp
// Like GroupBy but executes immediately (not deferred)
var lookup = products.ToLookup(p => p.Category);

// Access groups
var electronics = lookup["Electronics"];
foreach (var product in electronics)
{
    Console.WriteLine(product.Name);
}
```

---

### 5. Join Operators

#### Join - Inner join

```csharp
class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
}

class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public decimal Total { get; set; }
}

List<Customer> customers = new List<Customer>
{
    new Customer { Id = 1, Name = "John" },
    new Customer { Id = 2, Name = "Jane" }
};

List<Order> orders = new List<Order>
{
    new Order { Id = 101, CustomerId = 1, Total = 100 },
    new Order { Id = 102, CustomerId = 2, Total = 200 },
    new Order { Id = 103, CustomerId = 1, Total = 150 }
};

// Method syntax
var result = customers.Join(
    orders,
    customer => customer.Id,      // outer key
    order => order.CustomerId,     // inner key
    (customer, order) => new       // result selector
    {
        CustomerName = customer.Name,
        OrderId = order.Id,
        Total = order.Total
    }
);

// Query syntax (more readable)
var result = from customer in customers
             join order in orders on customer.Id equals order.CustomerId
             select new
             {
                 CustomerName = customer.Name,
                 OrderId = order.Id,
                 Total = order.Total
             };

foreach (var item in result)
{
    Console.WriteLine($"{item.CustomerName}: Order {item.OrderId}, ${item.Total}");
}
// John: Order 101, $100
// Jane: Order 102, $200
// John: Order 103, $150
```

#### GroupJoin - Left outer join

```csharp
// All customers, with their orders (if any)
var result = customers.GroupJoin(
    orders,
    customer => customer.Id,
    order => order.CustomerId,
    (customer, customerOrders) => new
    {
        CustomerName = customer.Name,
        Orders = customerOrders
    }
);

foreach (var item in result)
{
    Console.WriteLine($"{item.CustomerName}:");
    foreach (var order in item.Orders)
    {
        Console.WriteLine($"  Order {order.Id}: ${order.Total}");
    }
}

// Query syntax
var result = from customer in customers
             join order in orders on customer.Id equals order.CustomerId into customerOrders
             select new
             {
                 CustomerName = customer.Name,
                 Orders = customerOrders
             };
```

#### Zip - Combine two sequences

```csharp
int[] numbers = { 1, 2, 3, 4 };
string[] words = { "one", "two", "three", "four" };

// Combine element by element
var result = numbers.Zip(words, (num, word) => $"{num} = {word}");
// ["1 = one", "2 = two", "3 = three", "4 = four"]

// Tuple result (C# 7.0+)
var result = numbers.Zip(words);
// [(1, "one"), (2, "two"), ...]

// Stops at shortest sequence
int[] numbers = { 1, 2, 3 };
string[] words = { "one", "two", "three", "four", "five" };
var result = numbers.Zip(words);
// Only 3 pairs (stops when numbers ends)
```

---

### 6. Set Operators

#### Distinct - Unique elements

```csharp
int[] numbers = { 1, 2, 2, 3, 3, 3, 4, 4, 4, 4 };

var unique = numbers.Distinct();
// [1, 2, 3, 4]

// With custom comparer
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

List<Person> people = new List<Person>
{
    new Person { Name = "John", Age = 25 },
    new Person { Name = "John", Age = 30 }  // Same name
};

var uniqueByName = people.DistinctBy(p => p.Name);  // C# 6.0+
// Only first John
```

#### Union - Combine sets (distinct)

```csharp
int[] first = { 1, 2, 3, 4 };
int[] second = { 3, 4, 5, 6 };

var combined = first.Union(second);
// [1, 2, 3, 4, 5, 6] (no duplicates)
```

#### Intersect - Common elements

```csharp
int[] first = { 1, 2, 3, 4 };
int[] second = { 3, 4, 5, 6 };

var common = first.Intersect(second);
// [3, 4]
```

#### Except - Difference

```csharp
int[] first = { 1, 2, 3, 4 };
int[] second = { 3, 4, 5, 6 };

var difference = first.Except(second);
// [1, 2] (elements in first but not in second)
```

---

### 7. Quantifier Operators

#### Any - At least one matches

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// Any elements?
bool hasElements = numbers.Any();  // true

// Any even?
bool hasEvens = numbers.Any(n => n % 2 == 0);  // true

// Empty collection
int[] empty = { };
bool hasElements = empty.Any();  // false
```

#### All - All match

```csharp
int[] numbers = { 2, 4, 6, 8 };

// All even?
bool allEven = numbers.All(n => n % 2 == 0);  // true

// All positive?
bool allPositive = numbers.All(n => n > 0);  // true
```

#### Contains - Specific element exists

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

bool hasThree = numbers.Contains(3);  // true
bool hasTen = numbers.Contains(10);   // false
```

---

### 8. Aggregation Operators

#### Count / LongCount

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

int count = numbers.Count();  // 5

// With condition
int evenCount = numbers.Count(n => n % 2 == 0);  // 2

// LongCount for very large collections
long longCount = numbers.LongCount();
```

#### Sum

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

int sum = numbers.Sum();  // 15

// With selector
class Product
{
    public decimal Price { get; set; }
}

List<Product> products = new List<Product>
{
    new Product { Price = 10 },
    new Product { Price = 20 },
    new Product { Price = 30 }
};

decimal total = products.Sum(p => p.Price);  // 60
```

#### Average

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

double average = numbers.Average();  // 3.0

// With selector
double avgPrice = products.Average(p => p.Price);
```

#### Min / Max

```csharp
int[] numbers = { 5, 2, 8, 1, 9 };

int min = numbers.Min();  // 1
int max = numbers.Max();  // 9

// With selector
decimal minPrice = products.Min(p => p.Price);
decimal maxPrice = products.Max(p => p.Price);

// MinBy / MaxBy (C# 6.0+) - returns whole object
var cheapest = products.MinBy(p => p.Price);
var mostExpensive = products.MaxBy(p => p.Price);
```

#### Aggregate - Custom aggregation

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// Sum using Aggregate
int sum = numbers.Aggregate((total, next) => total + next);  // 15

// Product (multiply all)
int product = numbers.Aggregate((total, next) => total * next);  // 120

// With seed value
int sum = numbers.Aggregate(0, (total, next) => total + next);

// With result selector
string result = numbers.Aggregate(
    "",                                    // seed
    (str, num) => str + num,              // accumulate
    str => "Numbers: " + str              // result selector
);
// "Numbers: 12345"
```

---

### 9. Element Operators

#### First / FirstOrDefault

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// First element
int first = numbers.First();  // 1

// First matching condition
int firstEven = numbers.First(n => n % 2 == 0);  // 2

// FirstOrDefault - returns default if empty
int[] empty = { };
int result = empty.FirstOrDefault();  // 0 (default for int)

// With condition
int result = numbers.FirstOrDefault(n => n > 10);  // 0 (not found)
```

#### Last / LastOrDefault

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

int last = numbers.Last();  // 5
int lastEven = numbers.Last(n => n % 2 == 0);  // 4
int lastOrDefault = numbers.LastOrDefault(n => n > 10);  // 0
```

#### Single / SingleOrDefault

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// Single - expects exactly one element
int single = new[] { 42 }.Single();  // 42

// Throws if zero or multiple elements
// int single = numbers.Single();  // Exception! (multiple elements)

// With condition
int singleEven = new[] { 1, 2, 3 }.Single(n => n % 2 == 0);  // 2

// SingleOrDefault
int result = numbers.SingleOrDefault(n => n == 3);  // 3
int notFound = numbers.SingleOrDefault(n => n > 10);  // 0
```

#### ElementAt / ElementAtOrDefault

```csharp
int[] numbers = { 10, 20, 30, 40, 50 };

// Get element at index
int third = numbers.ElementAt(2);  // 30

// Out of bounds
// int invalid = numbers.ElementAt(10);  // Exception!

// ElementAtOrDefault - returns default if out of bounds
int result = numbers.ElementAtOrDefault(10);  // 0
```

#### DefaultIfEmpty

```csharp
int[] numbers = { 1, 2, 3 };
int[] empty = { };

// Return sequence or default if empty
var result1 = numbers.DefaultIfEmpty();  // [1, 2, 3]
var result2 = empty.DefaultIfEmpty();    // [0]

// With custom default
var result3 = empty.DefaultIfEmpty(999); // [999]
```

---

### 10. Generation Operators

#### Range - Sequence of integers

```csharp
// Enumerable.Range(start, count)
var numbers = Enumerable.Range(1, 5);
// [1, 2, 3, 4, 5]

var numbers = Enumerable.Range(10, 3);
// [10, 11, 12]

// Use in LINQ
var squares = Enumerable.Range(1, 10).Select(n => n * n);
// [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

#### Repeat - Repeat value

```csharp
// Enumerable.Repeat(value, count)
var repeated = Enumerable.Repeat("Hello", 3);
// ["Hello", "Hello", "Hello"]

var zeros = Enumerable.Repeat(0, 5);
// [0, 0, 0, 0, 0]
```

#### Empty - Empty sequence

```csharp
var empty = Enumerable.Empty<int>();
// []

// Useful for default return values
IEnumerable<int> GetNumbers(bool condition)
{
    if (condition)
        return new[] { 1, 2, 3 };
    else
        return Enumerable.Empty<int>();
}
```

---

### 11. Partitioning Operators

#### Take / Skip

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Take first n
var first5 = numbers.Take(5);
// [1, 2, 3, 4, 5]

// Skip first n
var after5 = numbers.Skip(5);
// [6, 7, 8, 9, 10]

// Pagination
int pageSize = 3;
int pageNumber = 2;  // 0-based

var page = numbers.Skip(pageNumber * pageSize)
                  .Take(pageSize);
// [4, 5, 6] (page 2)
```

#### TakeWhile / SkipWhile

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 1, 2, 3 };

// Take while condition is true
var result = numbers.TakeWhile(n => n < 4);
// [1, 2, 3] (stops at first 4)

// Skip while condition is true
var result = numbers.SkipWhile(n => n < 4);
// [4, 5, 1, 2, 3] (starts from first 4)
```

#### TakeLast / SkipLast (C# 6.0+)

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

var last3 = numbers.TakeLast(3);
// [8, 9, 10]

var withoutLast3 = numbers.SkipLast(3);
// [1, 2, 3, 4, 5, 6, 7]
```

#### Chunk (C# 6.0+)

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Split into chunks
var chunks = numbers.Chunk(3);

foreach (var chunk in chunks)
{
    Console.WriteLine(string.Join(", ", chunk));
}
// 1, 2, 3
// 4, 5, 6
// 7, 8, 9
// 10
```

---

### 12. Conversion Operators

#### ToArray / ToList (Immediate Execution!)

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// To array (creates new array)
int[] array = numbers.Where(n => n > 2).ToArray();
// [3, 4, 5]

// To list (creates new List<T>)
List<int> list = numbers.Where(n => n > 2).ToList();
// [3, 4, 5]

// ⚠️ These execute immediately (not deferred)
```

#### ToDictionary

```csharp
class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
}

List<Product> products = new List<Product>
{
    new Product { Id = 1, Name = "Laptop" },
    new Product { Id = 2, Name = "Phone" }
};

// To dictionary
var dict = products.ToDictionary(p => p.Id);
// { 1: Product(Laptop), 2: Product(Phone) }

// With value selector
var dict = products.ToDictionary(p => p.Id, p => p.Name);
// { 1: "Laptop", 2: "Phone" }
```

#### ToLookup

```csharp
// Like ToDictionary but allows duplicate keys
var lookup = products.ToLookup(p => p.Category);

// Access groups
var electronics = lookup["Electronics"];
```

#### ToHashSet (C# 7.3+)

```csharp
int[] numbers = { 1, 2, 2, 3, 3, 3 };

var set = numbers.ToHashSet();
// HashSet { 1, 2, 3 }
```

#### AsEnumerable / AsQueryable

```csharp
// AsEnumerable - force LINQ to Objects
IEnumerable<int> enumerable = numbers.AsEnumerable();

// AsQueryable - enable IQueryable features (for databases)
IQueryable<int> queryable = numbers.AsQueryable();
```

#### Cast\<T\>

```csharp
System.Collections.ArrayList list = new System.Collections.ArrayList();
list.Add(1);
list.Add(2);
list.Add(3);

// Cast non-generic collection to typed
var typed = list.Cast<int>();
// IEnumerable<int>
```

---

### 13. Concatenation Operators

#### Concat

```csharp
int[] first = { 1, 2, 3 };
int[] second = { 4, 5, 6 };

var combined = first.Concat(second);
// [1, 2, 3, 4, 5, 6]

// Multiple sequences
var all = first.Concat(second).Concat(new[] { 7, 8, 9 });
```

#### Append / Prepend (C# 7.3+)

```csharp
int[] numbers = { 2, 3, 4 };

// Append single element
var withLast = numbers.Append(5);
// [2, 3, 4, 5]

// Prepend single element
var withFirst = numbers.Prepend(1);
// [1, 2, 3, 4]

// Chain
var result = numbers.Prepend(1).Append(5);
// [1, 2, 3, 4, 5]
```

---

### 14. Equality Operator

#### SequenceEqual

```csharp
int[] first = { 1, 2, 3 };
int[] second = { 1, 2, 3 };
int[] third = { 1, 2, 3, 4 };

bool equal1 = first.SequenceEqual(second);  // true
bool equal2 = first.SequenceEqual(third);   // false

// Order matters
int[] fourth = { 3, 2, 1 };
bool equal3 = first.SequenceEqual(fourth);  // false
```

---

## Deferred vs Immediate Execution

### Deferred Execution (Lazy Evaluation)

**Most LINQ operators use deferred execution** - query is not executed until you iterate.

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// Query is defined but NOT executed yet
var query = numbers.Where(n =>
{
    Console.WriteLine($"Checking {n}");
    return n > 2;
});

Console.WriteLine("Query created");

// NOW it executes (when enumerated)
foreach (var num in query)
{
    Console.WriteLine($"Result: {num}");
}

// Output:
// Query created
// Checking 1
// Checking 2
// Checking 3
// Result: 3
// Checking 4
// Result: 4
// Checking 5
// Result: 5
```

**Deferred Operators:** Where, Select, OrderBy, GroupBy, Join, Take, Skip, etc.

### Immediate Execution (Eager Evaluation)

**Some operators execute immediately** - query runs when called.

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// Executes immediately
var array = numbers.Where(n => n > 2).ToArray();
var list = numbers.Where(n => n > 2).ToList();
int count = numbers.Count(n => n > 2);
int sum = numbers.Sum();

Console.WriteLine("Already executed!");
```

**Immediate Operators:** ToArray, ToList, ToDictionary, ToHashSet, Count, Sum, Average, Min, Max, First, Last, Single, Any, All

### Execution Behavior Table

| Category | Deferred | Immediate |
|----------|----------|-----------|
| **Filtering** | Where, OfType | - |
| **Projection** | Select, SelectMany | - |
| **Sorting** | OrderBy, ThenBy | - |
| **Grouping** | GroupBy | ToLookup |
| **Join** | Join, GroupJoin | - |
| **Set** | Distinct, Union, Intersect | - |
| **Partitioning** | Take, Skip, TakeWhile | - |
| **Conversion** | Cast, AsEnumerable | ToArray, ToList, ToDictionary |
| **Element** | DefaultIfEmpty | First, Last, Single, ElementAt |
| **Quantifier** | - | Any, All, Contains |
| **Aggregation** | - | Count, Sum, Average, Min, Max |

### Multiple Enumeration Warning

```csharp
// ⚠️ Problem: Query executed multiple times
var query = numbers.Where(n => n > 2);  // Deferred

int count = query.Count();          // Executes query
int sum = query.Sum();              // Executes query again!
var list = query.ToList();          // Executes query again!

// ✅ Solution: Materialize once
var list = numbers.Where(n => n > 2).ToList();  // Execute once

int count = list.Count;             // No query
int sum = list.Sum();               // No query
```

---

## Let Clause (Query Syntax)

Creates intermediate variables in query.

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// Without let (calculate twice)
var result = from n in numbers
             where n * n > 10
             select n * n;

// With let (calculate once)
var result = from n in numbers
             let square = n * n
             where square > 10
             select square;

// More readable
var result = from n in numbers
             let square = n * n
             let cube = n * n * n
             where square > 10
             select new { Number = n, Square = square, Cube = cube };
```

---

## Into Clause (Query Syntax)

Continues query after grouping or joining.

```csharp
// Group then filter groups
var result = from n in numbers
             group n by n % 2 into g
             where g.Count() > 2
             select new { Key = g.Key, Count = g.Count() };

// Join then continue
var result = from customer in customers
             join order in orders on customer.Id equals order.CustomerId into customerOrders
             select new { Customer = customer, OrderCount = customerOrders.Count() };
```

---

## Performance Considerations

### 1. Multiple Enumeration

```csharp
// ❌ Bad: Enumerated multiple times
var query = numbers.Where(n => ExpensiveOperation(n));

int count = query.Count();          // Enumerates
var first = query.First();          // Enumerates again
var list = query.ToList();          // Enumerates again

// ✅ Good: Enumerate once
var list = numbers.Where(n => ExpensiveOperation(n)).ToList();

int count = list.Count;
var first = list.First();
```

### 2. ToList() Timing

```csharp
// ❌ Bad: Materializes too early
var all = numbers.ToList();
var filtered = all.Where(n => n > 5).ToList();
var sorted = filtered.OrderBy(n => n).ToList();

// ✅ Good: Chain queries, materialize at end
var result = numbers
    .Where(n => n > 5)
    .OrderBy(n => n)
    .ToList();
```

### 3. LINQ vs foreach

```csharp
// For simple operations, similar performance
var result = numbers.Where(n => n > 5).ToList();

List<int> result = new List<int>();
foreach (var n in numbers)
{
    if (n > 5)
        result.Add(n);
}

// LINQ is cleaner, foreach is slightly faster
// Use LINQ for readability, foreach for hot paths
```

### 4. Avoid Unnecessary Operations

```csharp
// ❌ Bad: Unnecessary conversions
var result = numbers.ToList().ToArray().ToList();

// ✅ Good: Direct conversion
var result = numbers.ToList();

// ❌ Bad: Multiple sorts
var result = numbers.OrderBy(n => n).OrderBy(n => n * 2);

// ✅ Good: Single sort
var result = numbers.OrderBy(n => n * 2);
```

---

## Common LINQ Patterns

### Pattern 1: Filtering and Projecting

```csharp
// Get names of adults
var adultNames = people
    .Where(p => p.Age >= 18)
    .Select(p => p.Name);

// Get discounted prices
var prices = products
    .Where(p => p.InStock)
    .Select(p => p.Price * 0.9m);
```

### Pattern 2: Grouping and Aggregating

```csharp
// Total sales by category
var salesByCategory = products
    .GroupBy(p => p.Category)
    .Select(g => new
    {
        Category = g.Key,
        TotalSales = g.Sum(p => p.Price),
        Count = g.Count()
    });

// Average age by city
var avgAgeByCity = people
    .GroupBy(p => p.City)
    .Select(g => new
    {
        City = g.Key,
        AverageAge = g.Average(p => p.Age)
    });
```

### Pattern 3: Joining Collections

```csharp
// Customers with their orders
var customersWithOrders = customers
    .Join(orders,
          c => c.Id,
          o => o.CustomerId,
          (c, o) => new { Customer = c.Name, OrderTotal = o.Total });

// Or with query syntax (more readable)
var customersWithOrders = 
    from c in customers
    join o in orders on c.Id equals o.CustomerId
    select new { Customer = c.Name, OrderTotal = o.Total };
```

### Pattern 4: Pagination

```csharp
// Get page of results
int pageSize = 10;
int pageNumber = 2;  // 0-based

var page = items
    .Skip(pageNumber * pageSize)
    .Take(pageSize)
    .ToList();

// With total count
int totalCount = items.Count();
int totalPages = (totalCount + pageSize - 1) / pageSize;
```

### Pattern 5: Distinct Values from Property

```csharp
// Get unique categories
var categories = products
    .Select(p => p.Category)
    .Distinct();

// Or (C# 6.0+)
var categories = products
    .DistinctBy(p => p.Category)
    .Select(p => p.Category);
```

### Pattern 6: Top N Items

```csharp
// Top 10 most expensive
var topProducts = products
    .OrderByDescending(p => p.Price)
    .Take(10);

// Top 5 by rating, then by price
var topProducts = products
    .OrderByDescending(p => p.Rating)
    .ThenBy(p => p.Price)
    .Take(5);
```

### Pattern 7: Conditional Filtering

```csharp
// Build query conditionally
var query = products.AsQueryable();

if (!string.IsNullOrEmpty(category))
    query = query.Where(p => p.Category == category);

if (minPrice.HasValue)
    query = query.Where(p => p.Price >= minPrice.Value);

if (maxPrice.HasValue)
    query = query.Where(p => p.Price <= maxPrice.Value);

var results = query.ToList();
```

### Pattern 8: Flattening Nested Data

```csharp
// All items from all orders
var allItems = orders
    .SelectMany(o => o.Items);

// With parent context
var allItems = orders
    .SelectMany(o => o.Items, 
                (order, item) => new 
                { 
                    OrderId = order.Id, 
                    ItemName = item.Name 
                });
```

---

## LINQ Quick Reference Table

| Category | Operator | Purpose | Example |
|----------|----------|---------|---------|
| **Filter** | Where | Filter by condition | `Where(n => n > 5)` |
| | OfType | Filter by type | `OfType<int>()` |
| **Project** | Select | Transform | `Select(n => n * 2)` |
| | SelectMany | Flatten | `SelectMany(o => o.Items)` |
| **Sort** | OrderBy | Sort ascending | `OrderBy(p => p.Name)` |
| | OrderByDescending | Sort descending | `OrderByDescending(p => p.Price)` |
| | ThenBy | Secondary sort | `ThenBy(p => p.Age)` |
| | Reverse | Reverse order | `Reverse()` |
| **Group** | GroupBy | Group by key | `GroupBy(p => p.Category)` |
| | ToLookup | Group immediately | `ToLookup(p => p.City)` |
| **Join** | Join | Inner join | `Join(orders, c=>c.Id, ...)` |
| | GroupJoin | Left outer join | `GroupJoin(...)` |
| | Zip | Combine pairs | `Zip(other)` |
| **Set** | Distinct | Unique | `Distinct()` |
| | Union | Combine (unique) | `Union(other)` |
| | Intersect | Common | `Intersect(other)` |
| | Except | Difference | `Except(other)` |
| **Quantify** | Any | Has matching | `Any(n => n > 5)` |
| | All | All match | `All(n => n > 0)` |
| | Contains | Has element | `Contains(5)` |
| **Aggregate** | Count | Count | `Count()` |
| | Sum | Total | `Sum()` |
| | Average | Average | `Average()` |
| | Min/Max | Min/Max | `Min()`, `Max()` |
| | Aggregate | Custom | `Aggregate((a,b) => a+b)` |
| **Element** | First | First element | `First()` |
| | FirstOrDefault | First or default | `FirstOrDefault()` |
| | Last | Last element | `Last()` |
| | Single | Exactly one | `Single()` |
| | ElementAt | At index | `ElementAt(3)` |
| **Generate** | Range | Int sequence | `Range(1, 10)` |
| | Repeat | Repeat value | `Repeat("x", 5)` |
| | Empty | Empty sequence | `Empty<int>()` |
| **Partition** | Take | First n | `Take(5)` |
| | Skip | Skip n | `Skip(10)` |
| | TakeWhile | Take while true | `TakeWhile(n => n < 100)` |
| | Chunk | Split chunks | `Chunk(10)` |
| **Convert** | ToArray | To array | `ToArray()` ⚡ |
| | ToList | To list | `ToList()` ⚡ |
| | ToDictionary | To dict | `ToDictionary(k => k.Id)` ⚡ |
| | ToHashSet | To set | `ToHashSet()` ⚡ |
| **Concat** | Concat | Concatenate | `Concat(other)` |
| | Append | Add to end | `Append(item)` |
| | Prepend | Add to start | `Prepend(item)` |
| **Equal** | SequenceEqual | Compare | `SequenceEqual(other)` |

⚡ = Immediate execution (all others are deferred)

---

## Top 20 Most-Used LINQ Operators

1. **Where** - Filter elements
2. **Select** - Transform elements
3. **FirstOrDefault** - Get first or default
4. **ToList** - Materialize to list
5. **OrderBy** - Sort ascending
6. **Any** - Check if any match
7. **Count** - Count elements
8. **Sum** - Total values
9. **GroupBy** - Group by key
10. **Distinct** - Unique elements
11. **Take** - First n elements
12. **Skip** - Skip n elements
13. **Join** - Inner join
14. **SelectMany** - Flatten
15. **OrderByDescending** - Sort descending
16. **Max** - Maximum value
17. **Min** - Minimum value
18. **Average** - Average value
19. **All** - Check all match
20. **Contains** - Check contains

---

## Common Pitfalls

### 1. Modifying Collection During Iteration

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

// ❌ Error: Collection modified during enumeration
foreach (var n in numbers)
{
    if (n % 2 == 0)
        numbers.Remove(n);  // Exception!
}

// ✅ Solution: ToList()
foreach (var n in numbers.ToList())
{
    if (n % 2 == 0)
        numbers.Remove(n);
}
```

### 2. Forgetting ToList/ToArray

```csharp
// ❌ Query executed multiple times
var query = numbers.Where(n => ExpensiveOperation(n));
var count = query.Count();  // Execute
var first = query.First();  // Execute again

// ✅ Materialize once
var list = numbers.Where(n => ExpensiveOperation(n)).ToList();
var count = list.Count;
var first = list.First();
```

### 3. Using Single() Incorrectly

```csharp
int[] numbers = { 1, 2, 3 };

// ❌ Exception: Multiple elements
int single = numbers.Single();

// ✅ Use First if expecting multiple
int first = numbers.First();

// ✅ Use Single only when expecting exactly one
int single = new[] { 42 }.Single();
```

### 4. Unnecessary ToList Chains

```csharp
// ❌ Multiple materializations
var result = numbers
    .Where(n => n > 5).ToList()
    .Select(n => n * 2).ToList()
    .OrderBy(n => n).ToList();

// ✅ Chain, then materialize once
var result = numbers
    .Where(n => n > 5)
    .Select(n => n * 2)
    .OrderBy(n => n)
    .ToList();
```

---

## Best Practices

### 1. Use Method Syntax (Modern Approach)

```csharp
// ✅ Preferred (more common)
var result = people
    .Where(p => p.Age > 18)
    .OrderBy(p => p.Name)
    .Select(p => p.Name);

// Less common (but valid)
var result = from p in people
             where p.Age > 18
             orderby p.Name
             select p.Name;
```

### 2. Materialize at the End

```csharp
// ✅ Good
var result = collection
    .Where(...)
    .OrderBy(...)
    .Select(...)
    .ToList();  // Materialize once at end
```

### 3. Use FirstOrDefault Over First

```csharp
// ❌ Risky
var item = items.First();  // Exception if empty

// ✅ Safer
var item = items.FirstOrDefault();  // null if empty
if (item != null)
{
    // Use item
}
```

### 4. Prefer Any() Over Count() > 0

```csharp
// ❌ Inefficient
if (items.Count() > 0)  // Enumerates entire collection

// ✅ Efficient
if (items.Any())  // Stops at first element
```

### 5. Use Specific Operators

```csharp
// ❌ Verbose
var max = numbers.OrderByDescending(n => n).First();

// ✅ Concise
var max = numbers.Max();
```

### 6. Avoid Multiple Enumerations

```csharp
// ❌ Bad
var query = items.Where(x => x.IsValid);
if (query.Any())
{
    foreach (var item in query)  // Enumerates again
    {
        Process(item);
    }
}

// ✅ Good
var list = items.Where(x => x.IsValid).ToList();
if (list.Any())
{
    foreach (var item in list)
    {
        Process(item);
    }
}
```

### 7. Use LINQ for Readability

```csharp
// ❌ Imperative (harder to read)
List<int> evens = new List<int>();
foreach (var n in numbers)
{
    if (n % 2 == 0)
    {
        evens.Add(n * 2);
    }
}

// ✅ Declarative (easier to read)
var evens = numbers
    .Where(n => n % 2 == 0)
    .Select(n => n * 2)
    .ToList();
```

---

## Decision Tree: Which LINQ Operator?

```
What do you want to do?

Filter elements?
  └─> Where

Transform elements?
  └─> Select

Flatten nested collections?
  └─> SelectMany

Sort elements?
  ├─> OrderBy (ascending)
  └─> OrderByDescending (descending)

Group elements?
  └─> GroupBy

Join two collections?
  └─> Join (or query syntax)

Get unique elements?
  └─> Distinct

Check if any match?
  ├─> Any (at least one)
  ├─> All (all match)
  └─> Contains (specific element)

Count/sum/average?
  ├─> Count
  ├─> Sum
  ├─> Average
  ├─> Min / Max
  └─> Aggregate (custom)

Get first/last element?
  ├─> First / FirstOrDefault
  ├─> Last / LastOrDefault
  └─> Single / SingleOrDefault (exactly one)

Take/skip elements?
  ├─> Take (first n)
  ├─> Skip (skip n)
  ├─> TakeWhile (while condition)
  └─> SkipWhile (while condition)

Convert to collection?
  ├─> ToList
  ├─> ToArray
  ├─> ToDictionary
  └─> ToHashSet
```

---

**Guide Complete!** This comprehensive LINQ guide covers all operators, patterns, and best practices. LINQ is one of the most powerful features in C# - master it and you'll write cleaner, more expressive code! 📘