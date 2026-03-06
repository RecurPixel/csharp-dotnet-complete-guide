# C# Exception Handling & Resource Management Quick Reference

---

## 1. What are Exceptions?

### Error vs Exception

**Error** - Programming mistakes caught at compile-time
```csharp
int x = "hello";  // Compile-time error
```

**Exception** - Runtime problems that occur during program execution
```csharp
int[] numbers = { 1, 2, 3 };
int x = numbers[10];  // Runtime exception (IndexOutOfRangeException)
```

### Exception Hierarchy

```
System.Object
    │
    └─ System.Exception (base for all exceptions)
        │
        ├─ System.SystemException (runtime errors)
        │   ├─ NullReferenceException
        │   ├─ IndexOutOfRangeException
        │   ├─ InvalidOperationException
        │   ├─ ArgumentException
        │   │   ├─ ArgumentNullException
        │   │   └─ ArgumentOutOfRangeException
        │   ├─ DivideByZeroException
        │   ├─ OverflowException
        │   ├─ StackOverflowException
        │   ├─ OutOfMemoryException
        │   ├─ FormatException
        │   └─ InvalidCastException
        │
        ├─ System.IO.IOException
        │   ├─ FileNotFoundException
        │   ├─ DirectoryNotFoundException
        │   └─ EndOfStreamException
        │
        └─ Custom exceptions (derive from Exception)
            └─ MyCustomException
```

---

## 2. Common Exception Types

| Exception | When Thrown | Example |
|-----------|-------------|---------|
| **NullReferenceException** | Accessing null reference | `string s = null; s.Length;` |
| **IndexOutOfRangeException** | Invalid array/list index | `arr[100]` when arr.Length < 100 |
| **ArgumentNullException** | Null argument passed | Method receives null when not allowed |
| **ArgumentException** | Invalid argument | Method receives invalid value |
| **ArgumentOutOfRangeException** | Argument out of range | `list.RemoveAt(-1)` |
| **InvalidOperationException** | Invalid state for operation | Call method when object in wrong state |
| **DivideByZeroException** | Division by zero | `x / 0` (integer division) |
| **FormatException** | Invalid format in conversion | `int.Parse("abc")` |
| **InvalidCastException** | Invalid type cast | `(int)"hello"` |
| **OverflowException** | Arithmetic overflow | `checked { int x = int.MaxValue + 1; }` |
| **FileNotFoundException** | File not found | `File.Open("missing.txt")` |
| **IOException** | I/O operation failed | File access error |
| **NotImplementedException** | Method not implemented | Placeholder methods |
| **NotSupportedException** | Operation not supported | Feature not available |

### System-Level vs Application-Level Exceptions

**System Exceptions (SystemException)** - Thrown by .NET runtime
```csharp
NullReferenceException  // Null reference access
IndexOutOfRangeException  // Array bounds violation
InvalidOperationException  // Invalid state
```

**Application Exceptions** - ❌ **DEPRECATED approach**
```csharp
// ❌ OLD WAY (don't use)
public class MyException : ApplicationException { }
```

**Modern Approach** - ✅ Derive directly from Exception
```csharp
// ✅ MODERN WAY
public class MyException : Exception { }
```

---

## 3. Try-Catch-Finally

### Basic Structure

```csharp
try
{
    // Code that might throw an exception
    int result = 10 / 0;
}
catch (DivideByZeroException ex)
{
    // Handle the exception
    Console.WriteLine($"Error: {ex.Message}");
}
finally
{
    // Always executes (even if exception occurs)
    Console.WriteLine("Cleanup code");
}
```

### Try-Catch-Finally Flow

```
┌─────────────────────────────────────────┐
│ try block                               │
│ ├─ Execute code                         │
│ └─ Exception thrown? ──┐                │
└─────────────────────────┼────────────────┘
                          │
        No ◄──────────────┼──────────────► Yes
         │                │                 │
         │                │                 ▼
         │                │        ┌────────────────────┐
         │                │        │ catch block        │
         │                │        │ Handle exception   │
         │                │        └────────────────────┘
         │                │                 │
         │                ▼                 │
         │        ┌────────────────────────┐│
         └───────►│ finally block          ││
                  │ Always executes        ││
                  └────────────────────────┘│
                           │◄───────────────┘
                           ▼
                     Continue or Exit
```

### Catching Specific Exceptions

```csharp
try
{
    string input = Console.ReadLine();
    int number = int.Parse(input);
    int result = 100 / number;
    Console.WriteLine(result);
}
catch (FormatException ex)
{
    Console.WriteLine("Invalid format. Please enter a number.");
}
catch (DivideByZeroException ex)
{
    Console.WriteLine("Cannot divide by zero.");
}
catch (OverflowException ex)
{
    Console.WriteLine("Number too large.");
}
```

### Exception Filters (C# 6.0+)

```csharp
try
{
    // Code
}
catch (HttpRequestException ex) when (ex.StatusCode == 404)
{
    Console.WriteLine("Page not found");
}
catch (HttpRequestException ex) when (ex.StatusCode == 500)
{
    Console.WriteLine("Server error");
}
catch (HttpRequestException ex)
{
    Console.WriteLine($"Other HTTP error: {ex.StatusCode}");
}

// Another example
catch (Exception ex) when (ex.Message.Contains("timeout"))
{
    Console.WriteLine("Operation timed out");
}
```

---

## 4. Multiple Catch Clauses

### Order Matters! (Most Specific First)

```csharp
// ✅ CORRECT: Most specific to most general
try
{
    // Code
}
catch (FileNotFoundException ex)      // Most specific
{
    Console.WriteLine("File not found");
}
catch (IOException ex)                // More general
{
    Console.WriteLine("I/O error");
}
catch (Exception ex)                  // Most general
{
    Console.WriteLine("General error");
}

// ❌ WRONG: Will never catch specific exceptions
try
{
    // Code
}
catch (Exception ex)                  // Too general first!
{
    Console.WriteLine("Error");       // Catches everything
}
catch (FileNotFoundException ex)      // Never reached!
{
    Console.WriteLine("File not found");
}
```

### Multiple Exceptions in One Catch (C# 6.0+)

```csharp
// Option 1: Separate catch blocks
catch (ArgumentNullException ex) { /* handle */ }
catch (ArgumentOutOfRangeException ex) { /* handle */ }

// Option 2: Exception filters (if same handling)
catch (ArgumentException ex) when 
    (ex is ArgumentNullException || ex is ArgumentOutOfRangeException)
{
    // Handle both
}

// Option 3: Base class catch
catch (ArgumentException ex)  // Catches all derived types
{
    // Handle
}
```

---

## 5. Nesting Try-Catch Blocks

```csharp
try
{
    // Outer try
    Console.WriteLine("Opening file...");
    
    try
    {
        // Inner try
        int result = 10 / 0;
    }
    catch (DivideByZeroException ex)
    {
        // Handle inner exception
        Console.WriteLine("Division error");
    }
    
    // Continues here after inner catch
    Console.WriteLine("Continuing...");
}
catch (Exception ex)
{
    // Handle outer exceptions
    Console.WriteLine("Outer error");
}

// When to nest?
// - Different error handling strategies at different levels
// - Try something, if it fails, try alternative
try
{
    TryPrimaryMethod();
}
catch (Exception)
{
    try
    {
        TryFallbackMethod();
    }
    catch (Exception ex)
    {
        LogError(ex);
    }
}
```

---

## 6. Throwing Exceptions

### throw Keyword

```csharp
public void SetAge(int age)
{
    if (age < 0)
        throw new ArgumentException("Age cannot be negative", nameof(age));
    
    if (age > 150)
        throw new ArgumentOutOfRangeException(nameof(age), "Age must be less than 150");
    
    _age = age;
}
```

### throw vs throw ex (CRITICAL!)

```csharp
// ✅ CORRECT: Preserves stack trace
try
{
    SomeMethod();
}
catch (Exception ex)
{
    LogError(ex);
    throw;  // Re-throw original exception with full stack trace
}

// ❌ WRONG: Loses original stack trace!
try
{
    SomeMethod();
}
catch (Exception ex)
{
    LogError(ex);
    throw ex;  // Resets stack trace to this line!
}

// Wrapping in new exception (preserves original)
try
{
    SomeMethod();
}
catch (Exception ex)
{
    throw new MyException("Operation failed", ex);  // ex becomes InnerException
}
```

### Throw Expressions (C# 7.0+)

```csharp
// In conditional expressions
string name = input ?? throw new ArgumentNullException(nameof(input));

// In expression-bodied members
public string Name
{
    get => _name ?? throw new InvalidOperationException("Name not set");
}

// In lambda expressions
Func<string, int> parse = s => int.TryParse(s, out int result) 
    ? result 
    : throw new FormatException();
```

---

## 7. Custom Exceptions

### Creating Custom Exceptions

```csharp
// Minimal custom exception
public class MyException : Exception
{
    public MyException() { }
    
    public MyException(string message) : base(message) { }
    
    public MyException(string message, Exception innerException) 
        : base(message, innerException) { }
}

// Custom exception with additional properties
public class ValidationException : Exception
{
    public string PropertyName { get; }
    public object AttemptedValue { get; }
    
    public ValidationException(string propertyName, object attemptedValue, string message)
        : base(message)
    {
        PropertyName = propertyName;
        AttemptedValue = attemptedValue;
    }
}

// Usage
throw new ValidationException("Age", -5, "Age cannot be negative");
```

### Best Practices for Custom Exceptions

✅ **Do:**

- End name with "Exception" (e.g., `ValidationException`)
- Derive from `Exception` (not `ApplicationException`)
- Provide at least 3 constructors (default, message, message + inner)
- Include relevant context in properties
- Document when your exception is thrown

❌ **Don't:**

- Derive from `SystemException` or `ApplicationException`
- Throw generic `Exception` type
- Use exceptions for control flow

```csharp
// ✅ GOOD
public class InsufficientFundsException : Exception
{
    public decimal Balance { get; }
    public decimal RequestedAmount { get; }
    
    public InsufficientFundsException(decimal balance, decimal requested)
        : base($"Insufficient funds. Balance: {balance}, Requested: {requested}")
    {
        Balance = balance;
        RequestedAmount = requested;
    }
}

// Usage
if (balance < amount)
    throw new InsufficientFundsException(balance, amount);
```

---

## 8. Exception Properties

```csharp
try
{
    MethodThatThrows();
}
catch (Exception ex)
{
    // Message - description of the error
    Console.WriteLine($"Message: {ex.Message}");
    
    // StackTrace - call stack when exception occurred
    Console.WriteLine($"Stack Trace:\n{ex.StackTrace}");
    
    // InnerException - original exception (if wrapped)
    if (ex.InnerException != null)
    {
        Console.WriteLine($"Inner: {ex.InnerException.Message}");
    }
    
    // Source - assembly name where exception occurred
    Console.WriteLine($"Source: {ex.Source}");
    
    // TargetSite - method that threw the exception
    Console.WriteLine($"Method: {ex.TargetSite}");
    
    // HelpLink - URL for help (rarely used)
    ex.HelpLink = "https://docs.example.com/error123";
    
    // Data - custom key-value pairs
    ex.Data["UserId"] = userId;
    ex.Data["Timestamp"] = DateTime.Now;
    
    foreach (DictionaryEntry entry in ex.Data)
    {
        Console.WriteLine($"{entry.Key}: {entry.Value}");
    }
}
```

### Walking the Exception Chain

```csharp
void PrintExceptionDetails(Exception ex)
{
    Exception current = ex;
    int level = 0;
    
    while (current != null)
    {
        Console.WriteLine($"Level {level}: {current.GetType().Name}");
        Console.WriteLine($"  Message: {current.Message}");
        
        current = current.InnerException;
        level++;
    }
}

try
{
    // Code
}
catch (Exception ex)
{
    PrintExceptionDetails(ex);
}
```

---

## 9. Best Practices

### When to Catch Exceptions

✅ **DO catch when:**

- You can handle the error and recover
- You need to add context before re-throwing
- You need to clean up resources
- At application boundaries (UI, API endpoints)
- You need to log the error

❌ **DON'T catch when:**

- You can't do anything about it
- It would hide bugs (like `NullReferenceException` in your code)
- It's a programming error that should be fixed
- You're just going to re-throw it without adding value

```csharp
// ✅ GOOD: Can recover
try
{
    LoadConfigFromFile();
}
catch (FileNotFoundException)
{
    LoadDefaultConfig();  // Recover with defaults
}

// ✅ GOOD: Add context
try
{
    ProcessOrder(orderId);
}
catch (Exception ex)
{
    throw new OrderProcessingException($"Failed to process order {orderId}", ex);
}

// ❌ BAD: Catching but doing nothing
try
{
    RiskyOperation();
}
catch (Exception)
{
    // Silent failure - very bad!
}

// ❌ BAD: Hiding programming errors
try
{
    user.Name.ToUpper();  // user or Name is null
}
catch (NullReferenceException)
{
    // Should fix the null, not catch it!
}
```

### Specific vs General Exceptions

```csharp
// ✅ GOOD: Catch specific exceptions
try
{
    int number = int.Parse(input);
}
catch (FormatException)
{
    Console.WriteLine("Please enter a valid number");
}
catch (OverflowException)
{
    Console.WriteLine("Number is too large");
}

// ❌ BAD: Catch-all
try
{
    int number = int.Parse(input);
}
catch (Exception ex)  // Too general!
{
    Console.WriteLine("Something went wrong");
}
```

### Don't Use Exceptions for Control Flow

```csharp
// ❌ BAD: Using exceptions for logic
try
{
    int result = int.Parse(input);
    return result;
}
catch (FormatException)
{
    return 0;  // Default value
}

// ✅ GOOD: Use TryParse
if (int.TryParse(input, out int result))
{
    return result;
}
else
{
    return 0;
}

// ❌ BAD: Exception to check existence
try
{
    return File.ReadAllText(path);
}
catch (FileNotFoundException)
{
    return string.Empty;
}

// ✅ GOOD: Check first
if (File.Exists(path))
{
    return File.ReadAllText(path);
}
else
{
    return string.Empty;
}
```

### Log Exceptions

```csharp
try
{
    // Code
}
catch (Exception ex)
{
    // Log full exception details
    Logger.Error(ex, "Failed to process order {OrderId}", orderId);
    
    // Then decide: re-throw, handle, or return error
    throw;
}
```

### Fail Fast Principle

```csharp
// ✅ GOOD: Fail immediately with clear error
public void SetEmail(string email)
{
    if (string.IsNullOrWhiteSpace(email))
        throw new ArgumentException("Email is required", nameof(email));
    
    if (!email.Contains("@"))
        throw new ArgumentException("Invalid email format", nameof(email));
    
    _email = email;
}

// ❌ BAD: Silent failure or delayed error
public void SetEmail(string email)
{
    _email = email ?? "";  // Hides the problem
}
```

---

## 10. Resource Management

### IDisposable Pattern

#### IDisposable Interface

```csharp
public interface IDisposable
{
    void Dispose();
}
```

**Purpose:** Clean up unmanaged resources (file handles, database connections, network sockets)

#### Basic Implementation

```csharp
public class MyResource : IDisposable
{
    private bool _disposed = false;
    
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);  // Don't call finalizer
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Dispose managed resources
                // (objects that implement IDisposable)
            }
            
            // Dispose unmanaged resources
            // (file handles, etc.)
            
            _disposed = true;
        }
    }
}
```

#### Full Dispose Pattern (with Finalizer)

```csharp
public class UnmanagedResource : IDisposable
{
    private IntPtr _unmanagedHandle;
    private ManagedResource _managedResource;
    private bool _disposed = false;
    
    public UnmanagedResource()
    {
        _unmanagedHandle = AllocateUnmanagedResource();
        _managedResource = new ManagedResource();
    }
    
    // Public Dispose method
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);  // Prevent finalizer from running
    }
    
    // Protected Dispose method
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Dispose managed resources
                _managedResource?.Dispose();
            }
            
            // Always dispose unmanaged resources
            if (_unmanagedHandle != IntPtr.Zero)
            {
                FreeUnmanagedResource(_unmanagedHandle);
                _unmanagedHandle = IntPtr.Zero;
            }
            
            _disposed = true;
        }
    }
    
    // Finalizer (only if you have unmanaged resources)
    ~UnmanagedResource()
    {
        Dispose(false);
    }
    
    // Throw if already disposed
    private void ThrowIfDisposed()
    {
        if (_disposed)
            throw new ObjectDisposedException(GetType().Name);
    }
}
```

### using Statement (C# 1.0)

**Purpose:** Automatically call `Dispose()` when scope exits

```csharp
// Traditional using statement
using (var file = new FileStream("data.txt", FileMode.Open))
{
    // Use file
    byte[] buffer = new byte[1024];
    file.Read(buffer, 0, buffer.Length);
}  // Dispose() called automatically here

// Equivalent to:
FileStream file = null;
try
{
    file = new FileStream("data.txt", FileMode.Open);
    byte[] buffer = new byte[1024];
    file.Read(buffer, 0, buffer.Length);
}
finally
{
    file?.Dispose();
}
```

#### Multiple Resources

```csharp
// Option 1: Nested using
using (var reader = new StreamReader("input.txt"))
using (var writer = new StreamWriter("output.txt"))
{
    string line;
    while ((line = reader.ReadLine()) != null)
    {
        writer.WriteLine(line.ToUpper());
    }
}

// Option 2: Sequential using (same scope)
using (var reader = new StreamReader("input.txt"))
{
    using (var writer = new StreamWriter("output.txt"))
    {
        // Use both
    }
}
```

### using Declaration (C# 8.0+)

**Simplified syntax** - disposes at end of containing scope

```csharp
// Modern using declaration
void ProcessFile()
{
    using var file = new FileStream("data.txt", FileMode.Open);
    // Use file...
    // No braces needed
    // Disposed at end of method
}

// Multiple resources
void CopyFile()
{
    using var source = new FileStream("input.txt", FileMode.Open);
    using var dest = new FileStream("output.txt", FileMode.Create);
    
    source.CopyTo(dest);
    // Both disposed at end of method
}

// With exception handling
void SafeProcessFile()
{
    try
    {
        using var file = new FileStream("data.txt", FileMode.Open);
        // Process file
    }
    catch (IOException ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
    }
    // file disposed before catch block
}
```

### IAsyncDisposable (C# 8.0+)

**Purpose:** Asynchronous resource cleanup

```csharp
public class AsyncResource : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        // Async cleanup
        await CloseConnectionAsync();
        await FlushBuffersAsync();
    }
}

// Usage with await using
async Task ProcessDataAsync()
{
    await using var resource = new AsyncResource();
    // Use resource
    // DisposeAsync() called automatically
}

// Multiple resources
async Task CopyDataAsync()
{
    await using var source = new AsyncReader();
    await using var dest = new AsyncWriter();
    
    await source.CopyToAsync(dest);
}
```

### using Statement Evolution

```csharp
// C# 1.0: Traditional using
using (var file = File.OpenRead("data.txt"))
{
    // Use file
}

// C# 8.0: using declaration
void Method()
{
    using var file = File.OpenRead("data.txt");
    // Use file
}  // Disposed here

// C# 8.0: await using
async Task MethodAsync()
{
    await using var connection = new SqlConnection(connectionString);
    // Use connection
}  // DisposeAsync() called here
```

---

## 11. Exception Handling with Async

### try-catch in Async Methods

```csharp
public async Task<string> DownloadDataAsync(string url)
{
    try
    {
        using HttpClient client = new HttpClient();
        return await client.GetStringAsync(url);
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"Request failed: {ex.Message}");
        return string.Empty;
    }
    catch (TaskCanceledException ex)
    {
        Console.WriteLine("Request timed out");
        return string.Empty;
    }
}
```

### AggregateException

**When multiple tasks fail:**

```csharp
// When using Task.WaitAll or Task.Wait (blocking)
try
{
    Task.WaitAll(task1, task2, task3);
}
catch (AggregateException ex)
{
    foreach (var inner in ex.InnerExceptions)
    {
        Console.WriteLine($"Task failed: {inner.Message}");
    }
}

// ✅ BETTER: Use await (unwraps AggregateException)
try
{
    await Task.WhenAll(task1, task2, task3);
}
catch (Exception ex)  // First exception only
{
    Console.WriteLine($"Task failed: {ex.Message}");
}

// Handle all exceptions from multiple tasks
var tasks = new[] { task1, task2, task3 };
try
{
    await Task.WhenAll(tasks);
}
catch
{
    // Check each task individually
    foreach (var task in tasks)
    {
        if (task.IsFaulted)
        {
            Console.WriteLine($"Task failed: {task.Exception.Message}");
        }
    }
}
```

### Handling Task Exceptions

```csharp
// Task status
if (task.IsFaulted)
{
    Exception ex = task.Exception;  // AggregateException
    Console.WriteLine(ex.InnerException.Message);  // Actual exception
}

if (task.IsCanceled)
{
    Console.WriteLine("Task was canceled");
}

if (task.IsCompletedSuccessfully)
{
    Console.WriteLine("Task completed successfully");
}

// Observing exceptions (prevent unobserved task exceptions)
task.ContinueWith(t =>
{
    if (t.IsFaulted)
    {
        Log(t.Exception);
    }
}, TaskContinuationOptions.OnlyOnFaulted);
```

---

## Common Patterns

### Pattern 1: Try-Parse

```csharp
// ❌ BAD: Exception for control flow
try
{
    int number = int.Parse(input);
    ProcessNumber(number);
}
catch (FormatException)
{
    Console.WriteLine("Invalid number");
}

// ✅ GOOD: TryParse
if (int.TryParse(input, out int number))
{
    ProcessNumber(number);
}
else
{
    Console.WriteLine("Invalid number");
}
```

### Pattern 2: Retry Logic

```csharp
public async Task<T> RetryAsync<T>(
    Func<Task<T>> operation, 
    int maxAttempts = 3,
    int delayMs = 1000)
{
    for (int attempt = 1; attempt <= maxAttempts; attempt++)
    {
        try
        {
            return await operation();
        }
        catch (Exception ex) when (attempt < maxAttempts)
        {
            Console.WriteLine($"Attempt {attempt} failed: {ex.Message}");
            await Task.Delay(delayMs);
        }
    }
    
    // Last attempt without catch
    return await operation();
}

// Usage
var data = await RetryAsync(() => DownloadDataAsync(url));
```

### Pattern 3: Exception Translation

```csharp
// Translate low-level exceptions to domain exceptions
public class UserRepository
{
    public User GetById(int id)
    {
        try
        {
            return database.Query<User>(id);
        }
        catch (SqlException ex)
        {
            throw new RepositoryException($"Failed to retrieve user {id}", ex);
        }
    }
}
```

### Pattern 4: Safe Disposal

```csharp
public class SafeDisposal : IDisposable
{
    private readonly List<IDisposable> _resources = new();
    
    public void AddResource(IDisposable resource)
    {
        _resources.Add(resource);
    }
    
    public void Dispose()
    {
        var exceptions = new List<Exception>();
        
        foreach (var resource in _resources)
        {
            try
            {
                resource?.Dispose();
            }
            catch (Exception ex)
            {
                exceptions.Add(ex);
            }
        }
        
        if (exceptions.Any())
        {
            throw new AggregateException("Disposal failed", exceptions);
        }
    }
}
```

---

## Anti-Patterns (What NOT to Do)

### ❌ Empty Catch Block

```csharp
// ❌ NEVER DO THIS
try
{
    RiskyOperation();
}
catch (Exception)
{
    // Silent failure - debugging nightmare!
}

// ✅ At minimum, log it
catch (Exception ex)
{
    Logger.Error(ex, "Operation failed");
}
```

### ❌ Catching Exception Without Re-throwing

```csharp
// ❌ BAD: Swallowing exceptions
catch (Exception ex)
{
    Console.WriteLine(ex.Message);  // Just logging, not re-throwing
}

// ✅ GOOD: Log and re-throw
catch (Exception ex)
{
    Logger.Error(ex);
    throw;
}
```

### ❌ throw ex Instead of throw

```csharp
// ❌ BAD: Loses stack trace
catch (Exception ex)
{
    Logger.Error(ex);
    throw ex;  // Resets stack trace!
}

// ✅ GOOD: Preserves stack trace
catch (Exception ex)
{
    Logger.Error(ex);
    throw;  // Original stack trace preserved
}
```

### ❌ Exception for Control Flow

```csharp
// ❌ BAD: Expensive!
try
{
    var user = users.First(u => u.Id == id);
}
catch (InvalidOperationException)
{
    return null;
}

// ✅ GOOD: Use FirstOrDefault
var user = users.FirstOrDefault(u => u.Id == id);
return user;
```

### ❌ Catching and Doing Nothing

```csharp
// ❌ BAD
try
{
    SaveToDatabase(data);
}
catch (Exception)
{
    // Oops, forgot to handle
}
```

---

## Decision Tree: Should I Catch This Exception?

```
┌─────────────────────────────────────┐
│ Exception thrown                    │
└────────────────┬────────────────────┘
                 │
                 ▼
         Can I handle it?
         /              \
       YES               NO
        │                 │
        ▼                 ▼
  Can I recover?    Is it a boundary?
   /        \        (UI/API/Main)
 YES        NO           /      \
  │          │         YES      NO
  │          │          │        │
  │          ▼          │        │
  │    Log and         │        │
  │    Re-throw        │        │
  │          │          │        │
  └──────────┴──────────┘        │
             │                    │
             ▼                    ▼
      CATCH IT              DON'T CATCH
                           (let it bubble up)
```

---

## Quick Reference Summary

### Exception Hierarchy

- All exceptions derive from `System.Exception`
- Catch specific exceptions, not `Exception` (unless at boundary)
- Order catches from most specific to most general

### Try-Catch-Finally

- `try` - Code that might throw
- `catch` - Handle exception
- `finally` - Always executes (cleanup)

### Throwing

- `throw` - Preserves stack trace ✅
- `throw ex` - Loses stack trace ❌
- `throw new Exception("message", innerException)` - Wrap exception

### Resource Management

- Implement `IDisposable` for cleanup
- Use `using` statement/declaration
- Use `await using` for async resources
- Always dispose resources in `finally` or `using`

### Best Practices

- ✅ Catch specific exceptions
- ✅ Log exceptions
- ✅ Clean up resources
- ✅ Fail fast
- ✅ Use try-parse instead of try-catch
- ❌ Don't use exceptions for control flow
- ❌ Don't catch and do nothing
- ❌ Don't use `throw ex`

---

**Guide Complete!** Master these exception handling patterns and you'll write robust, production-ready C# code! 📘