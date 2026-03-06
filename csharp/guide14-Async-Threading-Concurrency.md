# C# Async, Threading & Concurrency Guide

---

## Part 1: Core Concepts

### The Big Picture

```
┌─────────────────────────────────────────────────────┐
│ CONCURRENCY                                         │
│ (Doing multiple things in overlapping time periods)│
│                                                     │
│ ┌──────────────────────┐ ┌──────────────────────┐  │
│ │ PARALLELISM          │ │ ASYNCHRONOUS         │  │
│ │ (True simultaneous   │ │ (Non-blocking,       │  │
│ │ execution on         │ │ waiting without      │  │
│ │ multiple cores)      │ │ blocking threads)    │  │
│ │                      │ │                      │  │
│ │ • Parallel class     │ │ • async/await        │  │
│ │ • PLINQ              │ │ • Task (I/O bound)   │  │
│ │ • Task (CPU bound)   │ │ • TAP pattern        │  │
│ └──────────────────────┘ └──────────────────────┘  │
│                                                     │
│ BOTH Built on: Thread & Task                       │
└─────────────────────────────────────────────────────┘
```

### Key Relationships

- **Thread** - OS-level execution unit (expensive, limited)
- **Task** - High-level abstraction over threads (cheap, efficient)
- **async/await** - Language feature to write asynchronous code that looks synchronous
- **Parallel** - For CPU-intensive work across multiple cores
- **Concurrency** - Umbrella term for all of the above

---

## Part 2: Synchronous vs Asynchronous

### Synchronous Execution

**What it is:** Code executes line-by-line, blocking until each operation completes

```csharp
// Synchronous (blocks thread)
public string GetData()
{
    Thread.Sleep(1000);      // Thread sits idle, wasting resources
    return "data";
}

public void ProcessData()
{
    string data = GetData(); // Waits here (blocks)
    Console.WriteLine(data);
}
```

### Asynchronous Execution

**What it is:** Code can continue without waiting for operation to complete

```csharp
// Asynchronous (releases thread)
public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);  // Thread does other work
    return "data";
}

public async Task ProcessDataAsync()
{
    Task<string> dataTask = GetDataAsync(); // Starts immediately
    // Do other work here
    string data = await dataTask;            // Wait only when needed
    Console.WriteLine(data);
}
```

### Blocking vs Non-Blocking

**Blocking:**

- Thread sits idle, waiting
- Wasteful (threads are expensive resources)
- Examples: `Thread.Sleep()`, `Task.Wait()`, `Task.Result`

**Non-Blocking:**

- Thread is released to do other work
- Efficient resource usage
- Examples: `await`, `ContinueWith()`

---

## Part 3: CPU-Bound vs I/O-Bound

### CPU-Bound Work

**What it is:** Work that requires computation

**Examples:**

- Mathematical calculations
- Data processing
- Image/video processing
- Encryption/decryption
- Parsing large files

**Use:**

- `Task.Run()`
- `Parallel.For()` / `Parallel.ForEach()`
- `PLINQ`

```csharp
// CPU-bound example
public async Task<int> CalculatePrimesAsync(int max)
{
    return await Task.Run(() => 
    {
        // Heavy computation on thread pool
        int count = 0;
        for (int i = 2; i <= max; i++)
        {
            if (IsPrime(i)) count++;
        }
        return count;
    });
}
```

### I/O-Bound Work

**What it is:** Work that waits for external resources

**Examples:**

- File reading/writing
- Network requests (HTTP, database)
- User input
- Hardware operations

**Use:**

- `async/await`
- Asynchronous methods (`ReadAsync()`, `WriteAsync()`, etc.)

```csharp
// I/O-bound example
public async Task<string> DownloadDataAsync(string url)
{
    using HttpClient client = new HttpClient();
    return await client.GetStringAsync(url); // Non-blocking wait
}
```

---

## Part 4: Thread (Legacy Approach)

**What it is:** Low-level OS thread for executing code concurrently

**When to use:** ❌ Rarely in modern C# - use `Task` instead

**Namespace:** System.Threading

### Thread Basics

```csharp
// Creating a thread
Thread thread = new Thread(() => 
{
    Console.WriteLine($"Running on thread {Thread.CurrentThread.ManagedThreadId}");
});

// Configure
thread.IsBackground = true;  // Won't prevent app from terminating
thread.Name = "WorkerThread";
thread.Priority = ThreadPriority.Normal;

// Start
thread.Start();

// Wait for completion
thread.Join();               // Blocks until thread finishes
thread.Join(1000);           // Wait with timeout
```

### Thread Properties

```csharp
Thread.CurrentThread         // Get current thread
thread.IsAlive               // Boolean if running
thread.IsBackground          // Background vs foreground
thread.Name                  // Thread name (for debugging)
thread.Priority              // ThreadPriority enum
thread.ThreadState           // Current state
thread.ManagedThreadId       // Unique ID
```

### Thread Methods

```csharp
thread.Start();              // Begin execution
thread.Join();               // Wait for completion
thread.Abort();              // ❌ DEPRECATED: Force stop (dangerous!)
thread.Interrupt();          // Interrupt waiting thread
Thread.Sleep(ms);            // Pause current thread
Thread.Yield();              // Give up CPU to other threads
```

### Problems with Threads

❌ **Expensive** - 1MB stack per thread  
❌ **Limited** - OS has thread limit  
❌ **Hard to manage** - Manual synchronization  
❌ **No return values** - Can't easily get result  
❌ **No error handling** - Exceptions crash thread

✅ **Modern Alternative:** Use `Task` instead!

---

## Part 5: Task & Task\<T\> (Modern Approach)

**What it is:** High-level abstraction representing an asynchronous operation

**When to use:** ✅ Default choice for all async/concurrent work

**Namespace:** System.Threading.Tasks

### Task vs Task\<T\>

```csharp
// Task - No return value (like void)
public async Task DoWorkAsync()
{
    await Task.Delay(1000);
    Console.WriteLine("Work done");
}

// Task\<T\> - Returns value of type T
public async Task<int> CalculateAsync()
{
    await Task.Delay(1000);
    return 42;
}
```

### Creating Tasks

```csharp
// Run on thread pool (CPU-bound)
Task task1 = Task.Run(() => DoWork());
Task<int> task2 = Task.Run(() => Calculate());

// Delay (async version of Thread.Sleep)
Task delay = Task.Delay(1000);

// Completed task
Task completed = Task.CompletedTask;
Task<int> result = Task.FromResult(42);

// Faulted task
Task faulted = Task.FromException(new Exception("Error"));

// Canceled task
Task canceled = Task.FromCanceled(cancellationToken);
```

### Task Properties

```csharp
task.Status                  // TaskStatus enum
task.IsCompleted             // true if finished (any state)
task.IsCompletedSuccessfully // true if finished successfully
task.IsFaulted               // true if threw exception
task.IsCanceled              // true if was canceled
task.Exception               // AggregateException if faulted
task.Result                  // ⚠️ Blocks until complete! Use await instead
task.Id                      // Unique identifier
```

### Waiting for Tasks

```csharp
// ❌ Blocking methods (avoid!)
task.Wait();                 // Block until complete
task.Wait(timeout);          // Block with timeout
int result = task.Result;    // Block and get result

// ✅ Async methods (prefer!)
await task;                  // Non-blocking wait
int result = await taskOfInt;
```

### Task Continuation

```csharp
// Chain tasks
Task task = Task.Run(() => Step1())
    .ContinueWith(t => Step2())
    .ContinueWith(t => Step3());

// Or use async/await (cleaner)
await Task.Run(() => Step1());
await Task.Run(() => Step2());
await Task.Run(() => Step3());
```

### Multiple Tasks

```csharp
Task task1 = Task.Run(() => Work1());
Task task2 = Task.Run(() => Work2());
Task task3 = Task.Run(() => Work3());

// Wait for all
await Task.WhenAll(task1, task2, task3);

// Wait for first
Task first = await Task.WhenAny(task1, task2, task3);

// Get all results
Task<int>[] tasks = {task1, task2, task3};
int[] results = await Task.WhenAll(tasks);
```

### ConfigureAwait

```csharp
// Capture context (default) - for UI apps
await SomeMethodAsync();

// Don't capture context - for libraries, better performance
await SomeMethodAsync().ConfigureAwait(false);
```

**When to use ConfigureAwait(false):**

- ✅ Library code
- ✅ ASP.NET Core
- ✅ Background services
- ❌ UI code (WPF, WinForms)

---

## Part 6: async / await (The Game Changer)

**What it is:** C# keywords that make asynchronous code look synchronous

**When to use:** Any I/O-bound operation

**Namespace:** Built into C# language

### Syntax Rules

```csharp
// Method signature
public async Task<string> DownloadDataAsync()
{
    string data = await httpClient.GetStringAsync(url);
    return data; // Returns Task<string> automatically
}

// Calling async method
string result = await DownloadDataAsync();
```

### Key Rules

1. Methods with `await` must be marked `async`
2. Async methods should return `Task` or `Task<T>`
3. By convention, name async methods with "Async" suffix
4. Cannot use `await` in:
    - Synchronous methods
    - `lock` statements
    - `unsafe` code

### Return Types

```csharp
// Task - Async method with no return value
public async Task ProcessAsync() { }

// Task\<T\> - Async method returning T
public async Task<int> CalculateAsync() { return 42; }

// ValueTask\<T\> - Performance optimization (avoid allocations)
public async ValueTask<int> GetCachedValueAsync() { }

// void - ❌ Only for event handlers! (no error handling)
private async void Button_Click(object sender, EventArgs e) { }

// IAsyncEnumerable\<T\> - Async streams (C# 8.0+)
public async IAsyncEnumerable<int> GetNumbersAsync() { }
```

### Async vs Sync Example

```csharp
// ❌ Synchronous (blocks thread)
public string GetData()
{
    Thread.Sleep(1000);      // Thread sits idle
    return "data";
}

// ✅ Asynchronous (releases thread)
public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);  // Thread does other work
    return "data";
}
```

### Async All The Way

```csharp
// ✅ Good: Async all the way
public async Task<string> ControllerActionAsync()
{
    var data = await _service.GetDataAsync();
    return await _service.ProcessDataAsync(data);
}

// ❌ Bad: Mixing sync and async
public string ControllerAction()
{
    var data = _service.GetDataAsync().Result; // DEADLOCK RISK!
    return _service.ProcessDataAsync(data).Result;
}
```

---

## Part 7: TAP (Task-based Asynchronous Pattern)

**What it is:** The recommended pattern for async programming in .NET

**When to use:** All new asynchronous APIs

### TAP Rules

1. Method returns `Task` or `Task<T>`
2. Method name ends with "Async"
3. Should have an overload accepting `CancellationToken`
4. Should be truly asynchronous (not just wrapping sync code)

### TAP Example

```csharp
public async Task<string> ReadFileAsync(
    string path,
    CancellationToken cancellationToken = default)
{
    using StreamReader reader = new StreamReader(path);
    return await reader.ReadToEndAsync(cancellationToken);
}
```

### TAP vs Other Patterns

|Pattern|Era|Example|
|---|---|---|
|**TAP** ⭐|Modern (.NET 4.5+)|`Task\<T\> MethodAsync()`|
|**APM**|Legacy (.NET 1.0)|`BeginMethod()` + `EndMethod()`|
|**EAP**|Legacy (.NET 2.0)|`MethodAsync()` + `MethodCompleted` event|

✅ **Always use TAP for new code**

---

## Part 8: Parallel Programming (CPU-Bound)

**What it is:** Execute CPU-intensive work across multiple cores simultaneously

**When to use:** CPU-bound work that can be divided

**Don't use for:** I/O-bound work (use async/await instead)

**Namespace:** System.Threading.Tasks

### Parallel Class

#### Parallel.For

```csharp
// Sequential (slow)
for (int i = 0; i < 1000; i++)
{
    ProcessItem(i);
}

// Parallel (fast on multi-core)
Parallel.For(0, 1000, i => 
{
    ProcessItem(i);
});
```

#### Parallel.ForEach

```csharp
// Sequential
foreach (var item in items)
{
    ProcessItem(item);
}

// Parallel
Parallel.ForEach(items, item => 
{
    ProcessItem(item);
});
```

#### Parallel.Invoke

```csharp
// Execute multiple methods in parallel
Parallel.Invoke(
    () => Method1(),
    () => Method2(),
    () => Method3()
);
```

### ParallelOptions

```csharp
var options = new ParallelOptions
{
    MaxDegreeOfParallelism = 4,         // Limit to 4 threads
    CancellationToken = cancellationToken
};

Parallel.ForEach(items, options, item => 
{
    ProcessItem(item);
});
```

---

## Part 9: PLINQ (Parallel LINQ)

**What it is:** Parallel version of LINQ for query processing

**When to use:** Parallelize LINQ queries on large datasets

**Namespace:** System.Linq

### Basic Usage

```csharp
// Sequential LINQ
var results = items
    .Where(x => x.IsValid)
    .Select(x => ProcessItem(x))
    .ToList();

// Parallel LINQ
var results = items
    .AsParallel()
    .Where(x => x.IsValid)
    .Select(x => ProcessItem(x))
    .ToList();
```

### PLINQ Options

```csharp
var results = items
    .AsParallel()
    .WithDegreeOfParallelism(4)           // Limit parallelism
    .WithCancellation(cancellationToken)   // Enable cancellation
    .AsOrdered()                           // Maintain order (slower)
    .Where(x => x.IsValid)
    .Select(x => ProcessItem(x))
    .ToList();

// ForAll (parallel action on results)
items
    .AsParallel()
    .Where(x => x.IsValid)
    .ForAll(item => ProcessItem(item));
```

---

## Part 10: CancellationToken & CancellationTokenSource

**What it is:** Cooperative cancellation mechanism for async operations

**When to use:** Allow users to cancel long-running operations

**Namespace:** System.Threading

### CancellationTokenSource

```csharp
// Create cancellation source
CancellationTokenSource cts = new CancellationTokenSource();

// Get token
CancellationToken token = cts.Token;

// Cancel
cts.Cancel();

// Cancel after timeout
cts.CancelAfter(5000); // 5 seconds

// Dispose when done
cts.Dispose();
```

### Using CancellationToken

```csharp
public async Task LongRunningOperationAsync(CancellationToken token)
{
    for (int i = 0; i < 1000; i++)
    {
        // Check for cancellation
        token.ThrowIfCancellationRequested();
        
        // Or check manually
        if (token.IsCancellationRequested)
        {
            // Clean up
            return;
        }
        
        await Task.Delay(100, token);
    }
}

// Calling code
CancellationTokenSource cts = new CancellationTokenSource();

Task task = LongRunningOperationAsync(cts.Token);

// Cancel after 5 seconds
cts.CancelAfter(5000);

try
{
    await task;
}
catch (OperationCanceledException)
{
    Console.WriteLine("Operation canceled");
}
```

### Cancellation Callback

```csharp
token.Register(() => 
{
    Console.WriteLine("Cancellation requested");
});
```

---

## Part 11: Thread Safety & Synchronization

### The Problem: Race Conditions

```csharp
int counter = 0;

// ❌ NOT thread-safe
Parallel.For(0, 1000, i => 
{
    counter++;  // Race condition!
});

Console.WriteLine(counter); // Wrong result!
```

### Solution 1: lock (Monitor)

```csharp
object lockObj = new object();
int counter = 0;

Parallel.For(0, 1000, i => 
{
    lock (lockObj)
    {
        counter++;  // Safe
    }
});
```

**Key Points:**

- Lock on private object, never public
- Keep locked sections small
- Never lock on `this`, `typeof(MyClass)`, or strings
- Can cause deadlocks if not careful

### Solution 2: Interlocked

```csharp
int counter = 0;

Parallel.For(0, 1000, i => 
{
    Interlocked.Increment(ref counter); // Safe and fast
});
```

**Interlocked Methods:**

```csharp
Interlocked.Increment(ref value);
Interlocked.Decrement(ref value);
Interlocked.Add(ref location, value);
Interlocked.Exchange(ref location, value);
Interlocked.CompareExchange(ref location, value, comparand);
```

### Solution 3: Concurrent Collections

```csharp
using System.Collections.Concurrent;

// Thread-safe bag
ConcurrentBag<int> bag = new ConcurrentBag<int>();
Parallel.For(0, 1000, i => 
{
    bag.Add(i); // Safe
});

// Thread-safe dictionary
ConcurrentDictionary<int, string> dict = new ConcurrentDictionary<int, string>();
dict.TryAdd(1, "one");
dict.TryGetValue(1, out string value);
dict.AddOrUpdate(1, "one", (key, old) => "ONE");

// Thread-safe queue
ConcurrentQueue<int> queue = new ConcurrentQueue<int>();
queue.Enqueue(1);
queue.TryDequeue(out int item);

// Thread-safe stack
ConcurrentStack<int> stack = new ConcurrentStack<int>();
stack.Push(1);
stack.TryPop(out int item);
```

### Solution 4: SemaphoreSlim

**What it is:** Limits number of threads accessing a resource

**When to use:** Rate limiting, connection pooling

```csharp
SemaphoreSlim semaphore = new SemaphoreSlim(3); // Max 3 concurrent

await semaphore.WaitAsync(); // Acquire permit
try
{
    // Do work (max 3 threads here at once)
}
finally
{
    semaphore.Release(); // Always release
}
```

### Solution 5: ReaderWriterLockSlim

**What it is:** Allows multiple readers or one writer

**When to use:** Reads common, writes rare

```csharp
ReaderWriterLockSlim rwLock = new ReaderWriterLockSlim();

// Reading (multiple readers allowed)
rwLock.EnterReadLock();
try
{
    // Read data
}
finally
{
    rwLock.ExitReadLock();
}

// Writing (exclusive access)
rwLock.EnterWriteLock();
try
{
    // Write data
}
finally
{
    rwLock.ExitWriteLock();
}
```

---

## Part 12: ThreadPool

**What it is:** Managed pool of worker threads reused for tasks

**When to use:** Understanding what's under the hood

**Direct use:** Rarely needed (Task uses this internally)

**Namespace:** System.Threading

```csharp
// Legacy approach
ThreadPool.QueueUserWorkItem(state => 
{
    Console.WriteLine("Work item");
});

// Modern approach (same thing)
Task.Run(() => 
{
    Console.WriteLine("Work item");
});

// Configuration
ThreadPool.GetMaxThreads(out int worker, out int io);
ThreadPool.SetMaxThreads(worker, io);
ThreadPool.GetAvailableThreads(out worker, out io);
```

---

## Part 13: Async Coordination Primitives

### TaskCompletionSource\<T\>

**What it is:** Manually control Task completion

**When to use:** Wrap callbacks or events into Task

```csharp
TaskCompletionSource<int> tcs = new TaskCompletionSource<int>();

// Simulate async operation
Task.Run(async () => 
{
    await Task.Delay(1000);
    tcs.SetResult(42); // Complete the task
});

// Wait for result
int result = await tcs.Task;

// Other completion methods
tcs.SetException(new Exception("Error"));
tcs.SetCanceled();
tcs.TrySetResult(42);
```

### ManualResetEventSlim / AutoResetEvent

**What it is:** Signal between threads

**When to use:** Rarely needed with async/await

```csharp
ManualResetEventSlim mre = new ManualResetEventSlim(false);

// Thread 1: Wait for signal
Task.Run(() => 
{
    mre.Wait(); // Blocks until signaled
    Console.WriteLine("Signaled!");
});

// Thread 2: Send signal
Task.Run(() => 
{
    Thread.Sleep(1000);
    mre.Set(); // Signal waiting threads
});
```

---

## Part 14: Common Patterns

### Pattern 1: Progress Reporting

```csharp
public async Task ProcessDataAsync(IProgress<int> progress)
{
    for (int i = 0; i < 100; i++)
    {
        await Task.Delay(10);
        progress?.Report(i + 1); // Report progress
    }
}

// Usage
var progress = new Progress<int>(percent => 
{
    Console.WriteLine($"Progress: {percent}%");
});

await ProcessDataAsync(progress);
```

### Pattern 2: Timeout Pattern

```csharp
public async Task<string> GetDataWithTimeoutAsync(int timeoutMs)
{
    using CancellationTokenSource cts = new CancellationTokenSource(timeoutMs);
    
    try
    {
        return await GetDataAsync(cts.Token);
    }
    catch (OperationCanceledException)
    {
        throw new TimeoutException();
    }
}

// Or using Task.WhenAny
public async Task<string> GetDataWithTimeoutAsync2(int timeoutMs)
{
    var dataTask = GetDataAsync();
    var timeoutTask = Task.Delay(timeoutMs);
    
    var completedTask = await Task.WhenAny(dataTask, timeoutTask);
    
    if (completedTask == timeoutTask)
        throw new TimeoutException();
    
    return await dataTask;
}
```

### Pattern 3: Retry Logic

```csharp
public async Task\<T\> RetryAsync\<T\>(
    Func<Task\<T\>> operation,
    int maxAttempts = 3,
    int delayMs = 1000)
{
    for (int i = 0; i < maxAttempts; i++)
    {
        try
        {
            return await operation();
        }
        catch (Exception) when (i < maxAttempts - 1)
        {
            await Task.Delay(delayMs);
        }
    }
    
    throw new Exception("Max retry attempts reached");
}

// Usage
var result = await RetryAsync(() => DownloadDataAsync());
```

### Pattern 4: Lazy Initialization (Thread-Safe)

```csharp
private readonly Lazy<ExpensiveObject> _lazyObj = 
    new Lazy<ExpensiveObject>(() => new ExpensiveObject());

public ExpensiveObject Instance => _lazyObj.Value;
```

### Pattern 5: Async Lazy

```csharp
public class AsyncLazy\<T\>
{
    private readonly Lazy<Task\<T\>> _lazy;
    
    public AsyncLazy(Func<Task\<T\>> factory)
    {
        _lazy = new Lazy<Task\<T\>>(() => Task.Run(factory));
    }
    
    public Task\<T\> Value => _lazy.Value;
}

// Usage
private readonly AsyncLazy<ExpensiveObject> _asyncLazy = 
    new AsyncLazy<ExpensiveObject>(async () =>
    {
        await Task.Delay(100);
        return new ExpensiveObject();
    });

public Task<ExpensiveObject> GetInstanceAsync() => _asyncLazy.Value;
```

---

## Part 15: Common Pitfalls

### Pitfall 1: Deadlock with .Result or .Wait()

```csharp
// ❌ DEADLOCK in UI or ASP.NET (legacy)
public void Method()
{
    var result = GetDataAsync().Result; // BLOCKS!
}

// ✅ Solution: Use async all the way
public async Task MethodAsync()
{
    var result = await GetDataAsync();
}
```

### Pitfall 2: Async Void

```csharp
// ❌ Bad: Can't catch exceptions
public async void ProcessDataAsync()
{
    await Task.Delay(1000);
    throw new Exception(); // Lost!
}

// ✅ Good: Return Task
public async Task ProcessDataAsync()
{
    await Task.Delay(1000);
    throw new Exception(); // Can be caught
}

// ⚠️ Exception: Event handlers only
private async void Button_Click(object sender, EventArgs e)
{
    await ProcessDataAsync();
}
```

### Pitfall 3: Capturing Loop Variables

```csharp
// ❌ Bad: All tasks capture same variable
for (int i = 0; i < 10; i++)
{
    Task.Run(() => Console.WriteLine(i)); // Prints 10 ten times!
}

// ✅ Good: Capture local copy
for (int i = 0; i < 10; i++)
{
    int local = i;
    Task.Run(() => Console.WriteLine(local)); // Prints 0-9
}

// ✅ C# 5.0+: foreach is safe
foreach (var item in items)
{
    Task.Run(() => Console.WriteLine(item)); // Safe
}
```

### Pitfall 4: Starting Too Many Tasks

```csharp
// ❌ Bad: Creates 1 million tasks!
var tasks = Enumerable.Range(0, 1000000)
    .Select(i => Task.Run(() => DoWork(i)))
    .ToArray();
await Task.WhenAll(tasks);

// ✅ Good: Use Parallel.ForEach (manages thread pool)
Parallel.ForEach(Enumerable.Range(0, 1000000), i => 
{
    DoWork(i);
});

// ✅ Good: Batch with SemaphoreSlim
var semaphore = new SemaphoreSlim(10); // Max 10 concurrent
var tasks = Enumerable.Range(0, 1000000)
    .Select(async i => 
    {
        await semaphore.WaitAsync();
        try
        {
            await DoWorkAsync(i);
        }
        finally
        {
            semaphore.Release();
        }
    })
    .ToArray();
await Task.WhenAll(tasks);
```

---

## Part 16: Async Streams (C# 8.0+)

**What it is:** Asynchronously iterate over data as it arrives

**When to use:** Large datasets, real-time data, pagination

**Namespace:** System.Collections.Generic

```csharp
// Producer
public async IAsyncEnumerable<int> GetNumbersAsync(
    [EnumeratorCancellation] CancellationToken token = default)
{
    for (int i = 0; i < 10; i++)
    {
        token.ThrowIfCancellationRequested();
        await Task.Delay(100, token);
        yield return i;
    }
}

// Consumer
await foreach (var number in GetNumbersAsync())
{
    Console.WriteLine(number); // Process as they arrive
}

// Real-world: Paginated API
public async IAsyncEnumerable<User> GetAllUsersAsync()
{
    int page = 1;
    while (true)
    {
        var users = await GetUserPageAsync(page);
        if (users.Count == 0) break;
        
        foreach (var user in users)
            yield return user;
        
        page++;
    }
}

// Usage
await foreach (var user in GetAllUsersAsync())
{
    Console.WriteLine(user.Name);
}
```

---

## Part 17: ValueTask\<T\> (Performance)

**What it is:** Struct-based Task alternative to avoid allocations

**When to use:** High-performance scenarios where result is often available synchronously

**Caution:** More restrictions than Task

```csharp
// ❌ Use Task\<T\>: Usually async
public async Task<string> DownloadAsync(string url)
{
    return await httpClient.GetStringAsync(url);
}

// ✅ Use ValueTask\<T\>: Often cached/synchronous
public ValueTask<User> GetUserAsync(int id)
{
    if (_cache.TryGetValue(id, out var user))
        return new ValueTask<User>(user); // No allocation
    
    return new ValueTask<User>(LoadUserAsync(id));
}
```

**Restrictions:**

- Can only await once
- Cannot use `.Result` or `.Wait()`
- Cannot await simultaneously from multiple threads

**If in doubt, use Task\<T\>**

---

## Decision Tree: What to Use When?

```
Is the work I/O-bound (network, file, database)?
│
├─ YES: Use async/await
│  ├─ Single operation: await SomeMethodAsync()
│  ├─ Multiple (sequential): await one, then await another
│  ├─ Multiple (parallel): await Task.WhenAll(tasks)
│  └─ Need cancellation: Pass CancellationToken
│
└─ NO: Is it CPU-bound (computation, processing)?
   │
   ├─ YES: Can the work be parallelized?
   │  ├─ YES: Loop over collection?
   │  │  ├─ YES: Use Parallel.ForEach or Parallel.For
   │  │  └─ NO: Use Parallel.Invoke
   │  │
   │  └─ NO: Use Task.Run() to offload to thread pool
   │
   └─ NO: Use synchronous code (normal method)

Special cases:
- Event handling: Event-driven programming
- Rate limiting: SemaphoreSlim
- Shared state: lock, Interlocked, or Concurrent collections
- Background work: Timer + async Task
- Producer-Consumer: BlockingCollection or Channels
```

---

## Quick Reference Summary

### Most Common Scenarios

**I/O-Bound (async/await):**

```csharp
// Single operation
string data = await httpClient.GetStringAsync(url);

// Multiple parallel
Task<string> t1 = httpClient.GetStringAsync(url1);
Task<string> t2 = httpClient.GetStringAsync(url2);
string[] results = await Task.WhenAll(t1, t2);
```

**CPU-Bound (Parallel):**

```csharp
// Parallel loop
Parallel.ForEach(items, item => ProcessItem(item));

// PLINQ
var results = items.AsParallel().Select(x => Process(x)).ToList();

// Single heavy operation
int result = await Task.Run(() => HeavyCalculation());
```

**Thread Safety:**

```csharp
// Simple counter
Interlocked.Increment(ref counter);

// Complex object
lock (lockObj) { /* modify object */ }

// Collections
ConcurrentDictionary<K,V> dict = new();
```

**Cancellation:**

```csharp
CancellationTokenSource cts = new();
await LongOperationAsync(cts.Token);
cts.Cancel();
```

---

**Guide Complete!** You now have a comprehensive Async/Threading reference! ⚡