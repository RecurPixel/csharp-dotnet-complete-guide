# C# Data Processing Patterns Quick Reference

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│               DATA PROCESSING PATTERN SELECTION                          │
│                                                                          │
│  Volume     │ Latency    │ Source      │ Pattern                        │
│  ───────────┼────────────┼─────────────┼────────────────────────────── │
│  Small      │ Low        │ In-memory   │ LINQ pipeline                  │
│  Medium     │ Medium     │ File / DB   │ Batch loop + IEnumerable       │
│  Large      │ Acceptable │ Stream / IO │ IAsyncEnumerable + chunking    │
│  Huge       │ Any        │ External    │ Channel<T> / Dataflow          │
│  Continuous │ Real-time  │ Queue / Bus │ IAsyncEnumerable + cancellation│
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Batch Processing Loop Pattern

```csharp
// Process a large dataset in fixed-size batches
static async Task ProcessInBatchesAsync<T>(
    IEnumerable<T> source,
    int batchSize,
    Func<IReadOnlyList<T>, CancellationToken, Task> process,
    CancellationToken ct = default)
{
    var batch = new List<T>(batchSize);

    foreach (var item in source)
    {
        ct.ThrowIfCancellationRequested();
        batch.Add(item);

        if (batch.Count == batchSize)
        {
            await process(batch, ct);
            batch.Clear();
        }
    }

    if (batch.Count > 0)
        await process(batch, ct);   // flush remainder
}

// Usage
await ProcessInBatchesAsync(records, batchSize: 500, async (batch, token) =>
{
    await db.BulkInsertAsync(batch, token);
}, cancellationToken);
```

### Chunk with .NET 6+ LINQ

```csharp
// LINQ Chunk — built-in batching
foreach (int[] chunk in Enumerable.Range(1, 10_000).Chunk(200))
{
    await ProcessChunkAsync(chunk);
}

// IAsyncEnumerable source with Chunk
await foreach (Order[] batch in GetOrdersAsync().Chunk(100))
{
    await SaveBatchAsync(batch);
}
```

---

## 2. Streaming with IEnumerable / IAsyncEnumerable

### IEnumerable — Lazy Pull (Synchronous)

```csharp
// ✅ yield return — memory-efficient, processes one at a time
static IEnumerable<Record> ReadCsvLazy(string path)
{
    using var reader = new StreamReader(path);
    string? line;
    while ((line = reader.ReadLine()) is not null)
    {
        var parts = line.Split(',');
        yield return new Record(parts[0], parts[1]);
    }
}

// Consumer controls pace — no full file in memory
foreach (var record in ReadCsvLazy("data.csv"))
    ProcessRecord(record);
```

### IAsyncEnumerable — Async Pull (I/O Streaming)

```csharp
// Async lazy stream — ideal for DB cursors, API pagination, file streams
static async IAsyncEnumerable<Order> GetOrdersAsync(
    DbConnection conn,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    using var cmd = conn.CreateCommand();
    cmd.CommandText = "SELECT * FROM Orders ORDER BY CreatedAt";
    using var reader = await cmd.ExecuteReaderAsync(ct);

    while (await reader.ReadAsync(ct))
        yield return MapOrder(reader);
}

// Consume
await foreach (var order in GetOrdersAsync(conn, cancellationToken))
{
    await ProcessOrderAsync(order, cancellationToken);
}
```

---

## 3. Transform / Filter / Aggregate Pipeline

```csharp
// Functional pipeline: compose transforms cleanly
var results = rawData
    .Where(r => r.IsActive)                    // filter
    .Select(r => MapToDto(r))                  // transform
    .GroupBy(dto => dto.Region)                // aggregate
    .Select(g => new RegionSummary
    {
        Region = g.Key,
        Total  = g.Sum(d => d.Amount),
        Count  = g.Count()
    })
    .OrderByDescending(s => s.Total)
    .ToList();

// Async pipeline with IAsyncEnumerable
static async IAsyncEnumerable<OrderDto> TransformOrders(
    IAsyncEnumerable<RawOrder> source,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    await foreach (var raw in source.WithCancellation(ct))
    {
        if (!raw.IsValid) continue;              // filter
        yield return Enrich(raw);               // transform
    }
}
```

### Pipelining with Extension Methods

```csharp
// Chainable pipeline stages
static IAsyncEnumerable<T> WhereAsync<T>(
    this IAsyncEnumerable<T> source,
    Func<T, bool> predicate,
    CancellationToken ct = default)
{
    return source.Where(predicate);  // System.Linq.Async (NuGet)
}

// System.Linq.Async NuGet package provides LINQ for IAsyncEnumerable:
// .WhereAwait(), .SelectAwait(), .ToListAsync(), .CountAsync(), etc.
```

---

## 4. Memory-Aware Chunking and Backpressure

### Channel<T> — Producer / Consumer with Backpressure

```csharp
using System.Threading.Channels;

// Bounded channel — blocks producer when full (backpressure)
var channel = Channel.CreateBounded<WorkItem>(new BoundedChannelOptions(capacity: 200)
{
    FullMode      = BoundedChannelFullMode.Wait,
    SingleWriter  = false,
    SingleReader  = false
});

// Producer
async Task ProduceAsync(CancellationToken ct)
{
    await foreach (var item in GetItemsAsync(ct))
    {
        await channel.Writer.WriteAsync(item, ct);
    }
    channel.Writer.Complete();
}

// Consumer
async Task ConsumeAsync(CancellationToken ct)
{
    await foreach (var item in channel.Reader.ReadAllAsync(ct))
    {
        await ProcessAsync(item, ct);
    }
}

// Run both
await Task.WhenAll(ProduceAsync(ct), ConsumeAsync(ct));
```

### Memory-Aware Streaming with ArrayPool

```csharp
using System.Buffers;

// Rent a buffer instead of allocating — return when done
byte[] buffer = ArrayPool<byte>.Shared.Rent(minimumLength: 4096);
try
{
    int read = await stream.ReadAsync(buffer.AsMemory(0, 4096), ct);
    ProcessChunk(buffer.AsSpan(0, read));
}
finally
{
    ArrayPool<byte>.Shared.Return(buffer, clearArray: true);
}
```

---

## 5. Retry / Idempotency / Error Buckets

### Retry with Exponential Backoff and Jitter

```csharp
static async Task<T> RetryWithBackoffAsync<T>(
    Func<CancellationToken, Task<T>> operation,
    int maxAttempts = 3,
    CancellationToken ct = default)
{
    var rng = new Random();
    for (int attempt = 1; attempt <= maxAttempts; attempt++)
    {
        try
        {
            return await operation(ct);
        }
        catch (Exception ex) when (attempt < maxAttempts && IsTransient(ex))
        {
            int baseDelay = (int)Math.Pow(2, attempt) * 100;   // 200, 400, 800ms
            int jitter    = rng.Next(0, 100);
            await Task.Delay(baseDelay + jitter, ct);
        }
    }
    return await operation(ct);  // final attempt — let it throw
}

static bool IsTransient(Exception ex) =>
    ex is TimeoutException or HttpRequestException or SocketException;
```

### Error Buckets — Separate Successes from Failures

```csharp
record ProcessResult<T>(T? Value, Exception? Error, bool IsSuccess);

static async IAsyncEnumerable<ProcessResult<TOut>> ProcessWithErrors<TIn, TOut>(
    IAsyncEnumerable<TIn> source,
    Func<TIn, Task<TOut>> transform,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    await foreach (var item in source.WithCancellation(ct))
    {
        ProcessResult<TOut> result;
        try
        {
            var value = await transform(item);
            result = new(value, null, true);
        }
        catch (Exception ex)
        {
            result = new(default, ex, false);
        }
        yield return result;
    }
}

// Separate success / failure after processing
var allResults  = await ProcessWithErrors(orders, ProcessOrderAsync).ToListAsync();
var succeeded   = allResults.Where(r => r.IsSuccess).ToList();
var failed      = allResults.Where(r => !r.IsSuccess).ToList();
// Retry or DLQ 'failed'
```

### Idempotency Guard

```csharp
// Track processed IDs to prevent re-processing (e.g., message deduplication)
var processed = new HashSet<string>();

await foreach (var message in messageStream)
{
    if (!processed.Add(message.Id))
        continue;  // Already handled — skip

    await HandleAsync(message);
}
```

---

## 6. Cancellation and Progress Reporting

### CancellationToken Propagation

```csharp
// Always pass CancellationToken through the entire call chain
static async Task RunPipelineAsync(CancellationToken ct = default)
{
    var source = ReadSourceAsync(ct);
    var transformed = TransformAsync(source, ct);
    await WriteOutputAsync(transformed, ct);
}

// CancellationTokenSource with timeout
using var cts     = new CancellationTokenSource(TimeSpan.FromMinutes(5));
using var linked  = CancellationTokenSource.CreateLinkedTokenSource(
                        cts.Token, externalCancellationToken);

await RunPipelineAsync(linked.Token);
```

### Progress Reporting

```csharp
// IProgress<T> decouples progress reporting from processing logic
static async Task ImportAsync(
    IEnumerable<Row> rows,
    IProgress<int>? progress = null,
    CancellationToken ct = default)
{
    int processed = 0;
    foreach (var batch in rows.Chunk(100))
    {
        ct.ThrowIfCancellationRequested();
        await SaveBatchAsync(batch, ct);
        processed += batch.Length;
        progress?.Report(processed);       // thread-safe via Progress<T>
    }
}

// Caller
var progress = new Progress<int>(count =>
    Console.WriteLine($"Processed {count} rows"));

await ImportAsync(rows, progress, cancellationToken);
```

---

## 7. Parallel Processing with Controlled Concurrency

```csharp
// Parallel.ForEachAsync — .NET 6+ controlled concurrency
await Parallel.ForEachAsync(
    orders,
    new ParallelOptions
    {
        MaxDegreeOfParallelism = Environment.ProcessorCount,
        CancellationToken      = cancellationToken
    },
    async (order, ct) =>
    {
        await ProcessOrderAsync(order, ct);
    });

// SemaphoreSlim for custom concurrency on IAsyncEnumerable
var semaphore = new SemaphoreSlim(initialCount: 8);

var tasks = orders.Select(async order =>
{
    await semaphore.WaitAsync(ct);
    try   { await ProcessOrderAsync(order, ct); }
    finally { semaphore.Release(); }
});

await Task.WhenAll(tasks);
```

---

## 8. Lazy Initialization Patterns

```csharp
// Lazy<T> — thread-safe lazy singleton initialization
private static readonly Lazy<ExpensiveService> _service =
    new(() => new ExpensiveService(), LazyThreadSafetyMode.ExecutionAndPublication);

ExpensiveService svc = _service.Value;  // initialized once

// Async lazy — requires wrapper (Lazy<Task<T>>)
private static readonly Lazy<Task<Config>> _config =
    new(() => LoadConfigAsync());

Config cfg = await _config.Value;  // one load, shared result

// AsyncLazy pattern (thread-safe, one initialization)
public class AsyncLazy<T>
{
    private readonly Lazy<Task<T>> _inner;
    public AsyncLazy(Func<Task<T>> factory)
        => _inner = new Lazy<Task<T>>(factory, LazyThreadSafetyMode.ExecutionAndPublication);
    public Task<T> Value => _inner.Value;
}
```

---

## Anti-Patterns

```
❌ Loading entire large datasets into memory before processing
❌ Forgetting to flush the last partial batch
❌ Ignoring CancellationToken in long-running loops
❌ Unbounded Task.WhenAll on thousands of items — saturates thread pool
❌ Using Parallel.ForEach for I/O-bound work — use async instead
❌ Re-processing items without idempotency checks
❌ Silently swallowing errors in a batch — use error buckets
❌ Not disposing IAsyncEnumerable producers — resource leaks
```

---

## Best Practices

```
✅ Prefer IAsyncEnumerable for I/O-sourced streams
✅ Use Chunk() for batch boundaries — clean and readable
✅ Always propagate CancellationToken through every async call
✅ Use Channel<T> when you need producer/consumer decoupling
✅ Report progress with IProgress<T> — decoupled from UI thread
✅ Separate error handling into result buckets — don't abort the pipeline
✅ Use ArrayPool for buffer reuse in high-throughput byte processing
✅ Bound concurrency with SemaphoreSlim or Parallel.ForEachAsync
```

---

## Quick Reference Summary

| Pattern | API / Type |
|---------|-----------|
| Batch loop | `Chunk()` + `foreach` |
| Lazy stream | `IEnumerable<T>` + `yield return` |
| Async stream | `IAsyncEnumerable<T>` + `yield return` |
| Producer/consumer | `Channel<T>` |
| Buffer reuse | `ArrayPool<T>.Shared` |
| Parallel async | `Parallel.ForEachAsync` |
| Concurrency limit | `SemaphoreSlim` |
| Retry | custom `RetryWithBackoffAsync` |
| Progress | `IProgress<T>` + `Progress<T>` |
| Lazy init | `Lazy<T>` / `Lazy<Task<T>>` |

---

**Guide Complete!** These patterns form the backbone of reliable data pipelines in C#. Combine IAsyncEnumerable, Channel<T>, and CancellationToken for production-grade streaming. 📘
