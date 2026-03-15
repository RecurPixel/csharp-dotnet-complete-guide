# C# Performance and Diagnostics Playbook

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│              PERFORMANCE WORKFLOW — MEASURE FIRST                        │
│                                                                          │
│  1. Profile → find the actual bottleneck                                 │
│  2. Measure → establish a baseline (BenchmarkDotNet)                     │
│  3. Optimize → change one thing at a time                                │
│  4. Measure again → confirm improvement                                  │
│  5. Review → don't break correctness for micro-gains                     │
│                                                                          │
│  ⚠️ 80% of performance issues are: allocations, LINQ on hot paths,       │
│     string concatenation, boxing, synchronous I/O, or N+1 queries.      │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 1. BenchmarkDotNet — Measure First

### Setup

```xml
<PackageReference Include="BenchmarkDotNet" Version="0.14.*" />
```

### Writing a Benchmark

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]          // shows allocations and GC pressure
[RankColumn]               // rank results
public class StringBenchmarks
{
    private const int N = 1000;
    private readonly string[] _parts = Enumerable.Repeat("hello", N).ToArray();

    [Benchmark(Baseline = true)]
    public string Concatenation()
    {
        string result = "";
        foreach (var s in _parts) result += s;
        return result;
    }

    [Benchmark]
    public string StringBuilderJoin()
    {
        var sb = new StringBuilder();
        foreach (var s in _parts) sb.Append(s);
        return sb.ToString();
    }

    [Benchmark]
    public string StringJoin() => string.Join("", _parts);
}

// Run in Main
BenchmarkRunner.Run<StringBenchmarks>();
// Always run in Release mode: dotnet run -c Release
```

### Benchmark Output Reading

```
| Method          | Mean        | Allocated |
|-----------------|-------------|-----------|
| Concatenation   | 5,432.0 μs  | 2,048 KB  |   ← baseline — slow + heavy
| StringBuilder   |   18.3 μs   |     4 KB  |   ← 300x faster, 500x less alloc
| StringJoin      |   17.1 μs   |     4 KB  |   ← similar, more readable
```

---

## 2. Allocation Hotspots and GC Pressure

### What Causes Allocations

```csharp
// ❌ Boxing — value type wrapped in object
int n = 42;
object boxed = n;      // heap allocation!
Console.WriteLine(n);  // boxing in old Console.WriteLine(object)

// ✅ Use generic overloads
Console.WriteLine(n.ToString());

// ❌ LINQ allocates enumerators, intermediate collections
var hot = items.Where(x => x.Active).Select(x => x.Id).ToList();

// ✅ For hot paths, use a manual loop
var result = new List<int>(items.Count);
foreach (var item in items)
    if (item.Active) result.Add(item.Id);

// ❌ Closures capture variables — heap allocation per lambda
int threshold = 10;
items.Where(x => x.Value > threshold);  // captures threshold

// ✅ Avoid closure in very hot paths — use static lambda + state
items.Where(static x => x.Value > 10);
```

### GC Generation Basics

```
Gen 0 → short-lived objects (request DTOs, temps) — cheap collection
Gen 1 → survived one Gen 0 — moderate cost
Gen 2 → long-lived (static lists, caches) — expensive collection
LOH   → objects > 85,000 bytes — never compacted by default

Goal: Keep Gen 2 and LOH collections rare.
      Allocate small, short-lived objects — let Gen 0 collect them fast.
```

---

## 3. String / Collection Allocation Pitfalls

```csharp
// ❌ String.Format or $"..." in hot loops — allocates per call
for (int i = 0; i < 1000; i++)
    Console.WriteLine($"Item {i}: {items[i].Name}");

// ✅ Use StringBuilder or stackalloc for hot paths
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append("Item ").Append(i).Append(": ").AppendLine(items[i].Name);

// ❌ .ToList() / .ToArray() mid-pipeline when not needed
var filtered = items.Where(x => x.Active).ToList().Select(x => x.Id);

// ✅ Defer materialization to the last step
var ids = items.Where(x => x.Active).Select(x => x.Id).ToList();

// ❌ LINQ .Count() on IEnumerable — iterates entire sequence
if (items.Count() > 0)    // O(n) for non-list types

// ✅ Use .Any() — stops at first element
if (items.Any())           // O(1) stop-early

// ❌ Dictionary pre-check + add (two lookups)
if (!dict.ContainsKey(key)) dict[key] = new List<int>();
dict[key].Add(item);

// ✅ GetOrAdd / TryGetValue pattern (one lookup)
if (!dict.TryGetValue(key, out var list))
    dict[key] = list = new List<int>();
list.Add(item);
```

---

## 4. Span<T> and Pooling Guidelines

### Span<T> — Zero-Copy Slicing

```csharp
// Parse without creating substrings
static int ParseFirstSegment(ReadOnlySpan<char> input)
{
    int idx = input.IndexOf(',');
    var segment = idx >= 0 ? input[..idx] : input;
    return int.Parse(segment);  // TryParse with span overload
}

// Stack-allocated buffer for small temporary byte work
Span<byte> buffer = stackalloc byte[256];   // no heap allocation
int n = Encoding.UTF8.GetBytes("hello".AsSpan(), buffer);
ProcessBytes(buffer[..n]);

// Span rules:
// ✅ Use for: temporary slicing, parsing, small buffers
// ❌ Cannot be stored in a class field (stack-only type)
// ❌ Cannot cross async boundaries — use Memory<T> instead

// Memory<T> — heap-safe version of Span for async contexts
static async Task ProcessAsync(Memory<byte> buffer, CancellationToken ct)
{
    // Memory<T> can be stored, awaited, and passed across async calls
    await stream.ReadAsync(buffer, ct);
    ProcessBytes(buffer.Span);  // access as Span when needed
}
```

### ArrayPool — Reuse Large Buffers

```csharp
using System.Buffers;

// ✅ Rent and return — avoids large heap allocation
byte[] buffer = ArrayPool<byte>.Shared.Rent(minimumLength: 81920);
try
{
    int read = await stream.ReadAsync(buffer.AsMemory(0, 81920), ct);
    await ProcessAsync(buffer.AsMemory(0, read), ct);
}
finally
{
    // Return with clear: true if buffer contained sensitive data
    ArrayPool<byte>.Shared.Return(buffer, clearArray: false);
}

// ✅ When to use ArrayPool:
// Large transient byte arrays > ~1KB (avoids LOH pressure)
// High-frequency allocations of the same size
// ❌ Don't use for small arrays — overhead exceeds benefit
```

---

## 5. ValueTask — When and Pitfalls

```csharp
// ValueTask<T>: avoids Task allocation when result is frequently synchronous
public ValueTask<int> GetCachedAsync(string key)
{
    if (_cache.TryGetValue(key, out int val))
        return ValueTask.FromResult(val);   // no Task allocation

    return new ValueTask<int>(FetchFromDbAsync(key));  // allocates Task only when needed
}

// ✅ Use ValueTask when:
// The method completes synchronously most of the time
// Called in very hot paths (millions of times/sec)
// Implementing interfaces like IAsyncEnumerable

// ❌ ValueTask pitfalls:
// Awaiting a ValueTask more than once — undefined behavior!
// Storing in a field and awaiting later — may complete inline
// Converting to Task with .AsTask() — defeats the purpose

// Safe pattern: consume immediately
var result = await myObj.GetCachedAsync("key");  // ✅
var vt = myObj.GetCachedAsync("key");
await vt;  // ✅ single await
await vt;  // ❌ second await of same ValueTask — undefined
```

---

## 6. Common Optimization Targets — Decision Table

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| High GC % | Too many allocations | Span, ArrayPool, struct, avoid LINQ |
| String concat slow | `+=` in loop | StringBuilder |
| Dictionary miss slow | Double lookup | `TryGetValue` pattern |
| LINQ slower than expected | Repeated `.ToList()` mid-chain | Remove intermediate materializations |
| Async overhead on sync path | Task allocation | `ValueTask` / synchronous short-circuit |
| LOH collection spikes | Large arrays (>85KB) | `ArrayPool<byte>` |
| Hot path boxing | Value types in non-generic APIs | Use generic collections and APIs |
| Thread pool starvation | Sync over async | `await` everywhere, no `.Result` / `.Wait()` |

---

## 7. Profiling Workflow — What to Inspect First

### Step 1: CPU Profiling

```
Tools:
- dotnet trace — CLI: dotnet-trace collect --process-id <pid>
- Visual Studio Diagnostics Tools (Windows)
- Rider's profiler (built-in)
- PerfView (Windows, deep CLR insight)

What to look for:
→ Which methods consume the most CPU?
→ Are there unexpected .NET runtime calls (GC, JIT, reflection)?
→ Is work distributed well across threads?
```

### Step 2: Memory Profiling

```
Tools:
- dotnet-counters: dotnet-counters monitor --process-id <pid>
- dotnet-gcdump: dotnet-gcdump collect --process-id <pid>
- Visual Studio Memory Usage
- JetBrains dotMemory

Key metrics:
→ Gen 0 / Gen 1 / Gen 2 collection frequency
→ Allocations per request
→ LOH size growth
→ Object retention (what's keeping things alive?)
```

### Step 3: Quick Counters

```bash
# Install dotnet-counters
dotnet tool install -g dotnet-counters

# Monitor live
dotnet-counters monitor --process-id 12345 --counters \
  System.Runtime[gc-heap-size,gen-0-gc-count,gen-2-gc-count,alloc-rate,threadpool-queue-length]
```

---

## 8. Practical Optimization Decision Tree

```
Is it actually slow?
  → Measure first with BenchmarkDotNet. Gut feeling is wrong 50% of the time.

CPU-bound or memory-bound?
  → CPU: reduce work, use better algorithm, parallelize carefully
  → Memory: reduce allocations, use Span, pool buffers

Is the hot path allocating?
  → Replace string concat with StringBuilder
  → Replace LINQ with manual loops on critical paths
  → Use struct instead of class for small, short-lived data
  → Use ArrayPool for large byte buffers

Is there GC pressure?
  → Look for boxing (object, IComparable without generic)
  → Look for closures in hot lambdas
  → Look for excessive string creation

Is I/O blocking threads?
  → Ensure all I/O is truly async (no .Result, no .Wait())
  → Increase HttpClient parallelism appropriately
  → Use streaming instead of loading full payloads

Is a specific method the bottleneck?
  → Try Span-based overloads
  → Try precomputation / caching
  → Try unsafe only as a last resort with proven gain
```

---

## 9. Source Generators and NativeAOT Readiness Notes

```csharp
// Source generators: code generated at compile time
// Benefit: eliminates reflection, startup cost, improves AOT compatibility
//
// Key scenarios already using source gen:
// - System.Text.Json: [JsonSerializable] + JsonSerializerContext
// - Regex: [GeneratedRegex]
// - LoggerMessage: [LoggerMessage] (structured logging)
// - IIncrementalGenerator for custom generators

// NativeAOT readiness checklist:
// ✅ Use JsonSerializerContext (no reflection-based JSON)
// ✅ Use [GeneratedRegex] (no Regex.Compile at runtime)
// ✅ Avoid Assembly.Load, Type.GetType(string), Activator.CreateInstance
// ✅ Avoid dynamic and ExpandoObject
// ✅ Avoid non-AOT-safe reflection patterns
// ✅ Use PublishAot=true and fix trim warnings early
//
// In .csproj:
// <PublishAot>true</PublishAot>
// <TrimmerRootDescriptor>TrimmerRoots.xml</TrimmerRootDescriptor>
```

---

## 10. Practical Recipes

### Recipe 1: Object Pool for Expensive Objects

```csharp
using Microsoft.Extensions.ObjectPool;

// DI registration
services.AddSingleton<ObjectPoolProvider, DefaultObjectPoolProvider>();
services.AddSingleton(sp =>
    sp.GetRequiredService<ObjectPoolProvider>()
      .Create<StringBuilder>());  // built-in StringBuilder policy

// Usage
public class TextProcessor(ObjectPool<StringBuilder> pool)
{
    public string Build(IEnumerable<string> parts)
    {
        var sb = pool.Get();
        try
        {
            foreach (var p in parts) sb.Append(p);
            return sb.ToString();
        }
        finally { pool.Return(sb); }  // clears and returns
    }
}
```

### Recipe 2: Struct for Hot Allocation Avoidance

```csharp
// Replace class with readonly struct for small, short-lived data
// Only if: < 16 bytes, immutable, copied often

public readonly struct PriceRange(decimal min, decimal max)
{
    public decimal Min { get; } = min;
    public decimal Max { get; } = max;
    public bool Contains(decimal price) => price >= Min && price <= Max;
}
// No heap allocation when created in a loop
```

### Recipe 3: Cached Delegate to Avoid Closure Allocation

```csharp
// ❌ Captures external variable — new delegate instance per call
public IEnumerable<Product> FilterByMin(IEnumerable<Product> products, decimal min)
    => products.Where(p => p.Price >= min);

// ✅ No closure: use struct + static lambda pattern or pass state directly
public IEnumerable<Product> FilterByMin(List<Product> products, decimal min)
{
    var result = new List<Product>();
    foreach (var p in products)
        if (p.Price >= min) result.Add(p);
    return result;
}
```

---

## Anti-Patterns

```
❌ Optimizing without measuring — you'll optimize the wrong thing
❌ String + string in a loop — O(n²) allocations
❌ .ToList() / .ToArray() in the middle of a LINQ chain unnecessarily
❌ .Result or .Wait() on async code — thread pool starvation
❌ Awaiting a ValueTask more than once
❌ Not returning ArrayPool buffers — silent memory growth
❌ Using Span in a class field — compile error (rightfully)
❌ Large object (>85KB) allocations in hot loops — LOH pressure
❌ Reflection in hot paths — expensive per-call cost
```

---

## Quick Reference Summary

| Topic | Tool / API |
|-------|-----------|
| Benchmark | `BenchmarkDotNet` + `[MemoryDiagnoser]` |
| Live counters | `dotnet-counters monitor` |
| GC dump | `dotnet-gcdump collect` |
| CPU trace | `dotnet-trace collect` |
| Zero-copy slice | `ReadOnlySpan<char>` / `Memory<T>` |
| Buffer pooling | `ArrayPool<T>.Shared` |
| Avoid Task alloc | `ValueTask<T>` |
| Object pooling | `ObjectPool<T>` (Microsoft.Extensions) |
| Source gen JSON | `JsonSerializerContext` |
| Source gen Regex | `[GeneratedRegex]` |
| AOT readiness | `PublishAot=true` + trim warnings |

---

**Guide Complete!** The single most important rule in performance engineering: measure before and after every change. A fast wrong answer is still wrong. 📘
