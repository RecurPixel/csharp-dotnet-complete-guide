# BenchmarkDotNet & Performance Optimization Quick Reference

---

## What is BenchmarkDotNet?

**BenchmarkDotNet** = Powerful .NET library for benchmarking code performance

**Key Features:**
- ✅ **Accurate** - Statistical analysis of results
- ✅ **Reliable** - Warm-up, multiple iterations
- ✅ **Detailed** - Execution time, memory allocation
- ✅ **Easy to use** - Attribute-based API
- ✅ **Cross-platform** - Works on .NET Framework, .NET Core, .NET 5+

**Why Benchmark?**
- 📊 Measure actual performance
- 🔍 Find bottlenecks
- ⚡ Compare different approaches
- 📈 Track performance over time
- 🎯 Validate optimizations

---

## Installation

```bash
# Install BenchmarkDotNet
dotnet add package BenchmarkDotNet

# Create console application for benchmarks
dotnet new console -n MyApp.Benchmarks
cd MyApp.Benchmarks
dotnet add package BenchmarkDotNet
```

```xml
<!-- MyApp.Benchmarks.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="BenchmarkDotNet" Version="0.13.12" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\MyApp\MyApp.csproj" />
  </ItemGroup>
</Project>
```

---

## Basic Benchmarking

### Simple Benchmark

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

public class StringConcatenationBenchmark
{
    private const int Iterations = 1000;

    [Benchmark]
    public string UsingStringConcat()
    {
        string result = "";
        for (int i = 0; i < Iterations; i++)
        {
            result += i.ToString();
        }
        return result;
    }

    [Benchmark]
    public string UsingStringBuilder()
    {
        var sb = new StringBuilder();
        for (int i = 0; i < Iterations; i++)
        {
            sb.Append(i);
        }
        return sb.ToString();
    }

    [Benchmark]
    public string UsingStringCreate()
    {
        return string.Create(Iterations * 4, Iterations, (span, count) =>
        {
            for (int i = 0; i < count; i++)
            {
                i.ToString().AsSpan().CopyTo(span);
                span = span.Slice(i.ToString().Length);
            }
        });
    }
}

// Program.cs
class Program
{
    static void Main(string[] args)
    {
        var summary = BenchmarkRunner.Run<StringConcatenationBenchmark>();
    }
}
```

### Running Benchmarks

```bash
# Build in Release mode (IMPORTANT!)
dotnet build -c Release

# Run benchmarks
dotnet run -c Release

# Results will be saved to BenchmarkDotNet.Artifacts/results/
```

**Sample Output:**
```
|              Method |      Mean |    Error |   StdDev |    Median |
|-------------------- |----------:|---------:|---------:|----------:|
| UsingStringConcat   | 245.12 μs | 4.89 μs  | 4.58 μs  | 244.50 μs |
| UsingStringBuilder  |  12.45 μs | 0.24 μs  | 0.23 μs  |  12.40 μs |
| UsingStringCreate   |   8.32 μs | 0.16 μs  | 0.15 μs  |   8.30 μs |
```

---

## Benchmark Attributes

### [Benchmark]

```csharp
public class MyBenchmarks
{
    [Benchmark]
    public void Method1()
    {
        // Code to benchmark
    }

    [Benchmark(Baseline = true)]  // Mark as baseline for comparison
    public void Method2()
    {
        // Baseline method
    }

    [Benchmark(Description = "Custom description")]
    public void Method3()
    {
        // Method with custom description
    }
}
```

### [Params] - Test with Different Parameters

```csharp
public class CollectionBenchmark
{
    [Params(10, 100, 1000, 10000)]
    public int Size { get; set; }

    private List<int> _list;
    private HashSet<int> _hashSet;

    [GlobalSetup]
    public void Setup()
    {
        _list = Enumerable.Range(0, Size).ToList();
        _hashSet = _list.ToHashSet();
    }

    [Benchmark]
    public bool ListContains()
    {
        return _list.Contains(Size / 2);
    }

    [Benchmark]
    public bool HashSetContains()
    {
        return _hashSet.Contains(Size / 2);
    }
}

// Results will show performance for each Size value
```

### Setup and Cleanup

```csharp
public class DatabaseBenchmark
{
    private ApplicationDbContext _context;

    [GlobalSetup]  // Run once before all benchmarks
    public void GlobalSetup()
    {
        // Setup database connection
        _context = new ApplicationDbContext();
    }

    [GlobalCleanup]  // Run once after all benchmarks
    public void GlobalCleanup()
    {
        _context.Dispose();
    }

    [IterationSetup]  // Run before each benchmark iteration
    public void IterationSetup()
    {
        // Reset state
    }

    [IterationCleanup]  // Run after each benchmark iteration
    public void IterationCleanup()
    {
        // Cleanup after iteration
    }

    [Benchmark]
    public async Task<List<Customer>> GetCustomers()
    {
        return await _context.Customers.ToListAsync();
    }
}
```

---

## Memory Diagnostics

### [MemoryDiagnoser]

```csharp
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
public class MemoryBenchmark
{
    [Benchmark]
    public List<int> CreateList()
    {
        var list = new List<int>();
        for (int i = 0; i < 1000; i++)
        {
            list.Add(i);
        }
        return list;
    }

    [Benchmark]
    public List<int> CreateListWithCapacity()
    {
        var list = new List<int>(1000);  // Pre-allocate capacity
        for (int i = 0; i < 1000; i++)
        {
            list.Add(i);
        }
        return list;
    }

    [Benchmark]
    public int[] CreateArray()
    {
        var array = new int[1000];
        for (int i = 0; i < 1000; i++)
        {
            array[i] = i;
        }
        return array;
    }
}

// Output includes:
// Gen0, Gen1, Gen2 - Garbage collection counts
// Allocated - Total memory allocated
```

**Sample Output:**
```
|                  Method |      Mean | Allocated |
|------------------------ |----------:|----------:|
|              CreateList | 18.50 μs  |   13.2 KB |
| CreateListWithCapacity  | 15.20 μs  |    8.1 KB |
|             CreateArray | 12.30 μs  |    4.0 KB |
```

---

## Advanced Configuration

### Custom Configuration

```csharp
using BenchmarkDotNet.Configs;
using BenchmarkDotNet.Jobs;
using BenchmarkDotNet.Toolchains.InProcess.Emit;

[Config(typeof(Config))]
public class MyBenchmarks
{
    private class Config : ManualConfig
    {
        public Config()
        {
            // Add multiple jobs to compare
            AddJob(Job.Default.WithRuntime(CoreRuntime.Core60));
            AddJob(Job.Default.WithRuntime(CoreRuntime.Core70));
            AddJob(Job.Default.WithRuntime(CoreRuntime.Core80));

            // Add diagnosers
            AddDiagnoser(MemoryDiagnoser.Default);
            
            // Set validation
            AddValidator(JitOptimizationsValidator.FailOnError);
        }
    }

    [Benchmark]
    public int Sum()
    {
        return Enumerable.Range(0, 100).Sum();
    }
}
```

### Iteration Control

```csharp
[SimpleJob(warmupCount: 3, iterationCount: 10)]
[MemoryDiagnoser]
public class IterationControlBenchmark
{
    [Benchmark]
    public void MyMethod()
    {
        Thread.Sleep(10);
    }
}

// warmupCount: Number of warm-up iterations
// iterationCount: Number of actual measurement iterations
```

---

## Real-World Benchmarks

### LINQ vs For Loop

```csharp
[MemoryDiagnoser]
public class LinqVsForLoopBenchmark
{
    private List<int> _numbers;

    [Params(100, 1000, 10000)]
    public int Size { get; set; }

    [GlobalSetup]
    public void Setup()
    {
        _numbers = Enumerable.Range(0, Size).ToList();
    }

    [Benchmark(Baseline = true)]
    public int SumWithForLoop()
    {
        int sum = 0;
        for (int i = 0; i < _numbers.Count; i++)
        {
            sum += _numbers[i];
        }
        return sum;
    }

    [Benchmark]
    public int SumWithForeach()
    {
        int sum = 0;
        foreach (var number in _numbers)
        {
            sum += number;
        }
        return sum;
    }

    [Benchmark]
    public int SumWithLinq()
    {
        return _numbers.Sum();
    }

    [Benchmark]
    public int SumWithLinqAggregate()
    {
        return _numbers.Aggregate(0, (acc, n) => acc + n);
    }
}
```

### String Operations

```csharp
[MemoryDiagnoser]
public class StringOperationsBenchmark
{
    private const string TestString = "Hello, World! This is a test string.";

    [Benchmark(Baseline = true)]
    public bool ContainsWithIndexOf()
    {
        return TestString.IndexOf("test") >= 0;
    }

    [Benchmark]
    public bool ContainsWithContains()
    {
        return TestString.Contains("test");
    }

    [Benchmark]
    public bool ContainsWithSpan()
    {
        return TestString.AsSpan().Contains("test", StringComparison.Ordinal);
    }

    [Benchmark]
    public string ToUpperWithString()
    {
        return TestString.ToUpper();
    }

    [Benchmark]
    public string ToUpperWithSpan()
    {
        Span<char> buffer = stackalloc char[TestString.Length];
        TestString.AsSpan().ToUpper(buffer, CultureInfo.InvariantCulture);
        return new string(buffer);
    }
}
```

### Collection Performance

```csharp
[MemoryDiagnoser]
public class CollectionBenchmark
{
    private const int ItemCount = 1000;
    private List<int> _list;
    private HashSet<int> _hashSet;
    private Dictionary<int, int> _dictionary;

    [GlobalSetup]
    public void Setup()
    {
        _list = Enumerable.Range(0, ItemCount).ToList();
        _hashSet = _list.ToHashSet();
        _dictionary = _list.ToDictionary(x => x, x => x);
    }

    [Benchmark]
    public bool ListContains()
    {
        return _list.Contains(500);
    }

    [Benchmark]
    public bool HashSetContains()
    {
        return _hashSet.Contains(500);
    }

    [Benchmark]
    public bool DictionaryContainsKey()
    {
        return _dictionary.ContainsKey(500);
    }

    [Benchmark]
    public int ListAdd()
    {
        var list = new List<int>();
        for (int i = 0; i < 100; i++)
        {
            list.Add(i);
        }
        return list.Count;
    }

    [Benchmark]
    public int ListAddWithCapacity()
    {
        var list = new List<int>(100);
        for (int i = 0; i < 100; i++)
        {
            list.Add(i);
        }
        return list.Count;
    }
}
```

### Async vs Sync

```csharp
[MemoryDiagnoser]
public class AsyncVsSyncBenchmark
{
    [Benchmark(Baseline = true)]
    public int Synchronous()
    {
        return ComputeSync();
    }

    [Benchmark]
    public async Task<int> Asynchronous()
    {
        return await ComputeAsync();
    }

    [Benchmark]
    public int ConfigureAwaitFalse()
    {
        return ComputeAsync().ConfigureAwait(false).GetAwaiter().GetResult();
    }

    private int ComputeSync()
    {
        Thread.Sleep(10);
        return 42;
    }

    private async Task<int> ComputeAsync()
    {
        await Task.Delay(10);
        return 42;
    }
}
```

---

## Performance Optimization Techniques

### 1. String Operations

```csharp
// ❌ Bad - String concatenation in loop
public string BuildString(int count)
{
    string result = "";
    for (int i = 0; i < count; i++)
    {
        result += i.ToString();
    }
    return result;
}

// ✅ Good - StringBuilder
public string BuildStringOptimized(int count)
{
    var sb = new StringBuilder(count * 4); // Pre-allocate capacity
    for (int i = 0; i < count; i++)
    {
        sb.Append(i);
    }
    return sb.ToString();
}

// ✅ Better - Span<char>
public string BuildStringWithSpan(int count)
{
    return string.Create(count * 4, count, (span, length) =>
    {
        for (int i = 0; i < length; i++)
        {
            // Use span operations
        }
    });
}
```

### 2. Collection Initialization

```csharp
// ❌ Bad - No capacity specified
public List<int> CreateList()
{
    var list = new List<int>();
    for (int i = 0; i < 1000; i++)
    {
        list.Add(i);
    }
    return list;
}

// ✅ Good - Pre-allocate capacity
public List<int> CreateListOptimized()
{
    var list = new List<int>(1000);
    for (int i = 0; i < 1000; i++)
    {
        list.Add(i);
    }
    return list;
}

// ✅ Best - Use array if size is known
public int[] CreateArray()
{
    var array = new int[1000];
    for (int i = 0; i < 1000; i++)
    {
        array[i] = i;
    }
    return array;
}
```

### 3. LINQ Optimization

```csharp
// ❌ Bad - Multiple enumerations
public int ProcessData(List<int> numbers)
{
    if (numbers.Any())  // First enumeration
    {
        var sum = numbers.Sum();  // Second enumeration
        var max = numbers.Max();  // Third enumeration
        return sum + max;
    }
    return 0;
}

// ✅ Good - Single enumeration
public int ProcessDataOptimized(List<int> numbers)
{
    if (numbers.Count == 0)  // Use Count property
        return 0;

    int sum = 0;
    int max = int.MinValue;
    
    foreach (var number in numbers)
    {
        sum += number;
        if (number > max)
            max = number;
    }
    
    return sum + max;
}

// ❌ Bad - Creating intermediate collections
public List<int> ProcessNumbers(List<int> numbers)
{
    return numbers
        .Where(n => n > 0)
        .Select(n => n * 2)
        .OrderBy(n => n)
        .ToList();
}

// ✅ Good - Use for loop when possible
public List<int> ProcessNumbersOptimized(List<int> numbers)
{
    var result = new List<int>(numbers.Count);
    
    for (int i = 0; i < numbers.Count; i++)
    {
        if (numbers[i] > 0)
        {
            result.Add(numbers[i] * 2);
        }
    }
    
    result.Sort();
    return result;
}
```

### 4. Avoid Boxing

```csharp
// ❌ Bad - Boxing
public void LogValue(int value)
{
    Console.WriteLine("Value: " + value);  // Boxing
    object obj = value;  // Boxing
}

// ✅ Good - No boxing
public void LogValueOptimized(int value)
{
    Console.WriteLine($"Value: {value}");  // No boxing with interpolation
}

// ❌ Bad - Boxing in collections
public void StoreValue(int value)
{
    var list = new ArrayList();
    list.Add(value);  // Boxing
}

// ✅ Good - Generic collections
public void StoreValueOptimized(int value)
{
    var list = new List<int>();
    list.Add(value);  // No boxing
}
```

### 5. Struct vs Class

```csharp
// Use struct for small, immutable data
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

// Benchmark
[MemoryDiagnoser]
public class StructVsClassBenchmark
{
    [Benchmark]
    public Point CreateStruct()
    {
        return new Point(10, 20);  // Stack allocation
    }

    [Benchmark]
    public PointClass CreateClass()
    {
        return new PointClass(10, 20);  // Heap allocation
    }
}

public class PointClass
{
    public int X { get; }
    public int Y { get; }
    
    public PointClass(int x, int y)
    {
        X = x;
        Y = y;
    }
}
```

### 6. ValueTask vs Task

```csharp
[MemoryDiagnoser]
public class ValueTaskBenchmark
{
    private readonly List<int> _cache = new() { 1, 2, 3 };

    [Benchmark(Baseline = true)]
    public async Task<int> UsingTask()
    {
        return await GetValueWithTask(1);
    }

    [Benchmark]
    public async ValueTask<int> UsingValueTask()
    {
        return await GetValueWithValueTask(1);
    }

    // Task always allocates
    private Task<int> GetValueWithTask(int key)
    {
        if (_cache.Contains(key))
            return Task.FromResult(key);  // Allocation
        
        return Task.FromResult(0);
    }

    // ValueTask - no allocation if synchronous path
    private ValueTask<int> GetValueWithValueTask(int key)
    {
        if (_cache.Contains(key))
            return new ValueTask<int>(key);  // No allocation
        
        return new ValueTask<int>(0);
    }
}
```

### 7. Span<T> and Memory<T>

```csharp
[MemoryDiagnoser]
public class SpanBenchmark
{
    private readonly string _text = "Hello, World!";

    [Benchmark(Baseline = true)]
    public string SubstringWithString()
    {
        return _text.Substring(0, 5);  // Allocates new string
    }

    [Benchmark]
    public ReadOnlySpan<char> SubstringWithSpan()
    {
        return _text.AsSpan(0, 5);  // No allocation
    }

    [Benchmark]
    public int ParseWithString()
    {
        return int.Parse("12345");
    }

    [Benchmark]
    public int ParseWithSpan()
    {
        return int.Parse("12345".AsSpan());  // No allocation
    }
}
```

### 8. ArrayPool<T>

```csharp
using System.Buffers;

[MemoryDiagnoser]
public class ArrayPoolBenchmark
{
    [Benchmark(Baseline = true)]
    public void WithoutArrayPool()
    {
        var array = new byte[1024];
        ProcessArray(array);
        // Array will be garbage collected
    }

    [Benchmark]
    public void WithArrayPool()
    {
        var array = ArrayPool<byte>.Shared.Rent(1024);
        try
        {
            ProcessArray(array);
        }
        finally
        {
            ArrayPool<byte>.Shared.Return(array);
        }
    }

    private void ProcessArray(byte[] array)
    {
        // Process array
    }
}
```

---

## Database Query Optimization

### EF Core vs Dapper

```csharp
[MemoryDiagnoser]
public class DatabaseBenchmark
{
    private ApplicationDbContext _context;
    private string _connectionString;

    [GlobalSetup]
    public void Setup()
    {
        _connectionString = "connection string";
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlServer(_connectionString)
            .Options;
        _context = new ApplicationDbContext(options);
    }

    [Benchmark(Baseline = true)]
    public async Task<List<Customer>> EFCore()
    {
        return await _context.Customers
            .Where(c => c.IsActive)
            .ToListAsync();
    }

    [Benchmark]
    public async Task<List<Customer>> EFCoreAsNoTracking()
    {
        return await _context.Customers
            .AsNoTracking()
            .Where(c => c.IsActive)
            .ToListAsync();
    }

    [Benchmark]
    public async Task<List<Customer>> Dapper()
    {
        using var connection = new SqlConnection(_connectionString);
        return (await connection.QueryAsync<Customer>(
            "SELECT * FROM Customers WHERE IsActive = 1"
        )).AsList();
    }
}
```

### Query Optimization

```csharp
// ❌ Bad - N+1 query problem
public async Task<List<OrderDto>> GetOrdersWithCustomers()
{
    var orders = await _context.Orders.ToListAsync();
    
    var result = new List<OrderDto>();
    foreach (var order in orders)
    {
        var customer = await _context.Customers
            .FirstAsync(c => c.Id == order.CustomerId);  // N queries!
        
        result.Add(new OrderDto
        {
            OrderId = order.Id,
            CustomerName = customer.Name
        });
    }
    
    return result;
}

// ✅ Good - Single query with Include
public async Task<List<OrderDto>> GetOrdersWithCustomersOptimized()
{
    return await _context.Orders
        .Include(o => o.Customer)
        .Select(o => new OrderDto
        {
            OrderId = o.Id,
            CustomerName = o.Customer.Name
        })
        .ToListAsync();
}

// ✅ Better - Projection (even faster)
public async Task<List<OrderDto>> GetOrdersWithCustomersProjection()
{
    return await _context.Orders
        .Select(o => new OrderDto
        {
            OrderId = o.Id,
            CustomerName = o.Customer.Name
        })
        .ToListAsync();
}
```

---

## Caching Optimization

```csharp
[MemoryDiagnoser]
public class CachingBenchmark
{
    private readonly Dictionary<int, string> _cache = new();
    private readonly MemoryCache _memoryCache;

    public CachingBenchmark()
    {
        _memoryCache = new MemoryCache(new MemoryCacheOptions());
    }

    [Benchmark(Baseline = true)]
    public string NoCaching()
    {
        return ExpensiveOperation(1);
    }

    [Benchmark]
    public string WithDictionary()
    {
        if (!_cache.TryGetValue(1, out var value))
        {
            value = ExpensiveOperation(1);
            _cache[1] = value;
        }
        return value;
    }

    [Benchmark]
    public string WithMemoryCache()
    {
        return _memoryCache.GetOrCreate(1, entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
            return ExpensiveOperation(1);
        });
    }

    private string ExpensiveOperation(int key)
    {
        Thread.Sleep(10);
        return $"Result for {key}";
    }
}
```

---

## Interpreting Results

### Understanding Metrics

```
|      Method |     Mean |    Error |   StdDev | Allocated |
|------------ |---------:|---------:|---------:|----------:|
|     Method1 | 100.5 μs | 2.01 μs  | 1.88 μs  |   1.2 KB  |
|     Method2 |  50.2 μs | 0.98 μs  | 0.92 μs  |   0.6 KB  |
```

**Mean:** Average execution time
**Error:** Half of 99.9% confidence interval
**StdDev:** Standard deviation of all measurements
**Allocated:** Total memory allocated per operation

### Performance Units

```
ns  = nanoseconds  (1,000,000,000 ns = 1 second)
μs  = microseconds (1,000,000 μs = 1 second)
ms  = milliseconds (1,000 ms = 1 second)
s   = seconds

KB  = kilobytes
MB  = megabytes
GB  = gigabytes
```

### Garbage Collection

```
Gen0: Number of Gen 0 collections per 1000 operations
Gen1: Number of Gen 1 collections per 1000 operations
Gen2: Number of Gen 2 collections per 1000 operations

Lower is better for Gen collections
```

---

## Best Practices

### 1. Always Run in Release Mode

```bash
# ❌ Bad
dotnet run

# ✅ Good
dotnet run -c Release
```

### 2. Use Baseline Comparisons

```csharp
[Benchmark(Baseline = true)]
public void CurrentImplementation() { }

[Benchmark]
public void NewImplementation() { }
```

### 3. Warm-up and Iterations

```csharp
[SimpleJob(warmupCount: 5, iterationCount: 10)]
public class MyBenchmark { }
```

### 4. Use Appropriate Scope

```csharp
// ✅ Good - Benchmark specific operation
[Benchmark]
public int SumArray()
{
    return _array.Sum();
}

// ❌ Bad - Benchmarking too much
[Benchmark]
public int SumArrayWithSetup()
{
    var array = new int[1000];  // Don't benchmark setup!
    for (int i = 0; i < 1000; i++)
        array[i] = i;
    return array.Sum();
}
```

### 5. Avoid Dead Code Elimination

```csharp
// ❌ Bad - Result not used, may be optimized away
[Benchmark]
public void Calculate()
{
    int result = ExpensiveCalculation();
}

// ✅ Good - Return or consume result
[Benchmark]
public int Calculate()
{
    return ExpensiveCalculation();
}
```

---

## Common Performance Anti-Patterns

### 1. Premature Optimization

```csharp
// ❌ Bad - Optimizing before measuring
public string GetUserName(int userId)
{
    // Complex caching logic
    // When simple database call is fast enough
}

// ✅ Good - Measure first, then optimize
[Benchmark]
public string GetUserName() { }

[Benchmark]
public string GetUserNameCached() { }
// Run benchmark, then decide if caching is worth it
```

### 2. Over-Optimization

```csharp
// ❌ Bad - Unreadable for minimal gain
public int Sum(int[] array)
{
    int sum = 0;
    int i = 0;
    int length = array.Length;
    int remainder = length & 3;
    
    for (; i < length - remainder; i += 4)
    {
        sum += array[i];
        sum += array[i + 1];
        sum += array[i + 2];
        sum += array[i + 3];
    }
    
    for (; i < length; i++)
        sum += array[i];
    
    return sum;
}

// ✅ Good - Readable and fast enough
public int Sum(int[] array)
{
    int sum = 0;
    for (int i = 0; i < array.Length; i++)
    {
        sum += array[i];
    }
    return sum;
}

// Or even better - use LINQ if performance is acceptable
public int Sum(int[] array) => array.Sum();
```

---

## Performance Checklist

### Code Level
- [ ] Avoid string concatenation in loops
- [ ] Pre-allocate collection capacity when size is known
- [ ] Use StringBuilder for string building
- [ ] Avoid boxing/unboxing
- [ ] Use struct for small, immutable data
- [ ] Consider Span<T> and Memory<T> for performance-critical code
- [ ] Use ArrayPool<T> for temporary buffers
- [ ] Prefer ValueTask over Task for hot paths
- [ ] Minimize allocations in hot paths
- [ ] Use AsNoTracking() for read-only EF queries

### LINQ
- [ ] Avoid multiple enumerations
- [ ] Use Any() instead of Count() > 0
- [ ] Use for/foreach instead of LINQ for performance-critical code
- [ ] Materialize queries appropriately
- [ ] Avoid creating intermediate collections

### Database
- [ ] Use AsNoTracking() for read-only queries
- [ ] Avoid N+1 queries (use Include or projection)
- [ ] Use pagination for large result sets
- [ ] Consider Dapper for read-heavy scenarios
- [ ] Add appropriate indexes
- [ ] Use compiled queries for frequently executed queries

### Async/Await
- [ ] Use ConfigureAwait(false) in libraries
- [ ] Don't use async void (except event handlers)
- [ ] Avoid async over sync (when operation is synchronous)
- [ ] Use ValueTask for hot paths with synchronous results

---

## Quick Reference: Common Benchmarks

```csharp
// String operations
[Benchmark] public string StringConcat() => "a" + "b" + "c";
[Benchmark] public string StringBuilder() => new StringBuilder().Append("a").Append("b").Append("c").ToString();
[Benchmark] public string StringInterpolation() => $"{"a"}{"b"}{"c"}";

// Collections
[Benchmark] public List<int> List() => new List<int> { 1, 2, 3 };
[Benchmark] public int[] Array() => new[] { 1, 2, 3 };
[Benchmark] public ImmutableList<int> ImmutableList() => ImmutableList.Create(1, 2, 3);

// LINQ
[Benchmark] public int LinqSum() => Enumerable.Range(0, 100).Sum();
[Benchmark] public int ForLoopSum() { int sum = 0; for (int i = 0; i < 100; i++) sum += i; return sum; }

// Dictionary lookup
[Benchmark] public bool DictionaryContainsKey() => _dict.ContainsKey(50);
[Benchmark] public bool DictionaryTryGetValue() => _dict.TryGetValue(50, out _);
```

---

**Guide Complete!** This comprehensive BenchmarkDotNet & Performance Optimization guide covers benchmarking setup, memory diagnostics, real-world examples, optimization techniques, and best practices for writing high-performance .NET code! ⚡
