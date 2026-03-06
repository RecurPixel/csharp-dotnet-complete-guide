# C# .NET Runtime & Core Concepts Quick Reference

---

## Part 1: .NET Ecosystem

### 1. .NET Evolution

#### .NET Framework (2002-present)

- **Platform:** Windows only
- **Architecture:** Monolithic, full framework
- **Status:** Maintenance mode (no new features after 4.8)
- **Use when:** Legacy Windows applications, WCF server, Windows Workflow
- **Latest:** 4.8.1

#### .NET Core (2016-2020)

- **Platform:** Cross-platform (Windows, Linux, macOS)
- **Architecture:** Modular, lightweight
- **Open Source:** Yes
- **Status:** Superseded by .NET 5+
- **Versions:** 1.0, 1.1, 2.0, 2.1 (LTS), 2.2, 3.0, 3.1 (LTS)

#### .NET 5+ (2020-present) - Modern ✅

- **Platform:** Unified cross-platform
- **Evolution:** Continuation of .NET Core (skipped version 4 to avoid confusion)
- **Versioning:** Annual releases
- **LTS Releases:** Every other version (6, 8, etc.)
- **Versions:**
  - .NET 5.0 (2020)
  - .NET 6.0 (2021) - LTS
  - .NET 7.0 (2022)
  - .NET 8.0 (2023) - LTS
  - .NET 9.0 (2024)

#### Evolution Timeline

```
2002: .NET Framework 1.0
      │
      ├── Windows-only
      ├── Full framework
      └── Mature ecosystem
      
2016: .NET Core 1.0
      │
      ├── Cross-platform
      ├── Open source
      ├── Modular
      └── High performance
      
2020: .NET 5.0
      │
      ├── Unified platform
      ├── Single codebase
      ├── Best of both worlds
      └── Annual releases
      
TODAY: .NET 9.0
```

---

### 2. .NET Framework vs .NET Core vs .NET 5+

| Feature | .NET Framework | .NET Core | .NET 5+ |
|---------|---------------|-----------|---------|
| **Platform** | Windows only | Cross-platform | Cross-platform |
| **Open Source** | No | Yes | Yes |
| **Performance** | Good | Better | Best |
| **Deployment** | System-wide | Side-by-side | Side-by-side |
| **App Models** | WinForms, WPF, ASP.NET | Console, ASP.NET Core, Worker | All modern apps |
| **Size** | Large (~500MB) | Small (~30MB) | Small (~30MB) |
| **Innovation** | None (maintenance) | None (superseded) | Active |
| **Support** | Until OS EOL | .NET Core 3.1 until 2022 | .NET 6/8 LTS |
| **Latest** | 4.8.1 | 3.1 | 9.0 |

**When to use:**

- **.NET Framework** ❌ Legacy apps only, Windows-specific (WCF server, Windows Workflow)
- **.NET Core** ❌ Use .NET 5+ instead
- **.NET 5+** ✅ All new development

---

### 3. .NET Standard

**What is it?** API specification (not implementation) for code sharing across .NET platforms.

**Purpose:** Create libraries that work on .NET Framework, .NET Core, and .NET 5+.

**Versions:** 1.0 through 2.1

**Key Versions:**

- **.NET Standard 2.0** - Sweet spot (broadest compatibility)
- **.NET Standard 2.1** - More APIs, but no .NET Framework support

#### .NET Standard Compatibility

| .NET Standard | .NET Framework | .NET Core | .NET 5+ |
|--------------|----------------|-----------|---------|
| **2.0** | 4.6.1+ | 2.0+ | All |
| **2.1** | ❌ Not supported | 3.0+ | All |

**Modern Approach:** Target specific .NET versions (net8.0) instead of .NET Standard for new projects.

---

### 4. .NET Architecture

```
┌─────────────────────────────────────────────┐
│       APPLICATION LAYER                     │
│   (Your Code: Console, Web, Desktop)        │
├─────────────────────────────────────────────┤
│   FRAMEWORK LIBRARIES (BCL)                 │
│   System.*, Collections, LINQ, IO, etc.     │
├─────────────────────────────────────────────┤
│   RUNTIME (CLR / CoreCLR)                   │
│   - Just-In-Time (JIT) Compiler             │
│   - Garbage Collector (GC)                  │
│   - Type System                             │
│   - Exception Handling                      │
│   - Thread Management                       │
├─────────────────────────────────────────────┤
│   OPERATING SYSTEM                          │
│   Windows / Linux / macOS                   │
└─────────────────────────────────────────────┘
```

**BCL (Base Class Library):** Standard library of types (System.*)

**CLR (Common Language Runtime):** Execution engine

---

## Part 2: Common Language Runtime (CLR)

### 5. What is CLR?

**Definition:** Execution engine for .NET applications

**CLR vs CoreCLR:**

- **CLR** - .NET Framework (Windows only, closed source)
- **CoreCLR** - .NET Core/5+ (cross-platform, open source)

**CLR Responsibilities:**

1. **Memory Management** - Automatic garbage collection
2. **Type Safety** - Enforce type rules at runtime
3. **Exception Handling** - Structured exception handling
4. **Security** - Code Access Security (Framework only, deprecated)
5. **Thread Management** - Thread pool, synchronization
6. **JIT Compilation** - Convert IL to native code
7. **Interoperability** - COM interop, P/Invoke

**Managed Code:** Code running under CLR control (C#, VB.NET, F#)

---

### 6. Compilation Process

#### Two-Stage Compilation

**Stage 1: Compile Time (C# → IL)**
```
C# Source Code (.cs)
        ↓
[C# Compiler (Roslyn)]
        ↓
Intermediate Language (IL/MSIL)
Stored in .dll or .exe assembly
```

**Stage 2: Runtime (IL → Native Code)**
```
IL Code in Assembly
        ↓
[CLR - JIT Compiler]
        ↓
Native Machine Code (x86, x64, ARM, etc.)
        ↓
[CPU Execution]
```

#### Why Two Stages?

✅ **Platform Independence**

- IL is platform-agnostic
- Same assembly runs on Windows, Linux, macOS

✅ **Language Interoperability**

- C#, VB.NET, F# all compile to IL
- Can reference each other's assemblies

✅ **Optimization Opportunities**

- JIT knows actual CPU features (AVX, SSE)
- Can optimize for runtime conditions
- Profile-guided optimization

---

### 7. CIL / MSIL (Intermediate Language)

**Names:**

- **CIL** - Common Intermediate Language (official)
- **MSIL** - Microsoft Intermediate Language (legacy name)
- **IL** - Short form

**Characteristics:**

- Object-oriented assembly language
- Stack-based instruction set
- Type-safe
- Platform-independent

**Example:**

```csharp
// C# code
int Add(int a, int b)
{
    return a + b;
}

// Equivalent IL (simplified)
.method private hidebysig instance int32 Add(int32 a, int32 b)
{
    ldarg.1      // Load argument 1 (a) onto stack
    ldarg.2      // Load argument 2 (b) onto stack
    add          // Pop two values, add, push result
    ret          // Return value on stack
}
```

**Viewing IL:**

- **ildasm.exe** - IL Disassembler (comes with .NET SDK)
- **ILSpy** - Open source .NET decompiler
- **dnSpy** - Debugger and assembly editor

---

### 8. JIT Compilation

**Just-In-Time Compilation:** Convert IL to native code at runtime

#### When JIT Happens

1. Method called for first time
2. IL → Native code conversion
3. Native code cached
4. Subsequent calls use cached native code

#### Types of JIT

**Normal JIT** (Default)

- Compiles methods as needed
- First call: slower (compilation overhead)
- Subsequent calls: fast (cached)

**Tiered Compilation** (.NET Core 3.0+)

- **Tier 0:** Quick JIT, minimal optimization → fast startup
- **Tier 1:** After method runs frequently, re-JIT with full optimization
- **Result:** Fast startup + optimized steady-state

```
Method First Called
    ↓
Tier 0 JIT (quick, minimal optimization)
    ↓
Native Code (faster startup)
    ↓
Method runs frequently
    ↓
Tier 1 JIT (full optimization)
    ↓
Optimized Native Code (best performance)
```

#### JIT Benefits

- Platform-specific optimization (use CPU-specific instructions)
- Runtime type information (devirtualization)
- Profile-guided optimization
- Dead code elimination
- Inlining

---

### 9. AOT Compilation (.NET 7+)

**Ahead-Of-Time Compilation:** Compile IL to native code before deployment

#### Native AOT

**Characteristics:**

- ✅ Entire app compiled to native code
- ✅ No JIT at runtime
- ✅ Faster startup (~100x)
- ✅ Smaller memory footprint
- ✅ Single-file executable
- ❌ Larger deployment size
- ❌ Limited reflection support
- ❌ No runtime code generation

**Enable in .csproj:**
```xml
<PublishAot>true</PublishAot>
```

#### ReadyToRun (R2R)

**Hybrid Approach:**

- Pre-compile frequently used code
- Keep IL for fallback
- Faster startup than pure JIT
- All features still available

**Enable:**
```xml
<PublishReadyToRun>true</PublishReadyToRun>
```

#### AOT Use Cases

- Microservices (fast startup critical)
- Serverless functions (cold start optimization)
- CLI tools
- Games (reduce startup time)
- Mobile apps

#### JIT vs AOT Comparison

| Feature | JIT | Native AOT | ReadyToRun |
|---------|-----|-----------|------------|
| **Startup** | Slow | Very fast | Fast |
| **Steady-state** | Fast | Fast | Fast |
| **Size** | Small | Large | Medium |
| **Reflection** | Full | Limited | Full |
| **Trimming** | Partial | Aggressive | Partial |

---

## Part 3: Memory Management

### 10. Managed vs Unmanaged Code

#### Managed Code (C#, VB.NET, F#)

**Features:**

- Runs under CLR control
- Automatic memory management (GC)
- Type safety guaranteed
- Exception handling
- Cross-language interoperability

**Advantages:**

- ✅ Easy to develop
- ✅ Memory safe (no leaks, no corruption)
- ✅ Platform independent (via IL)

#### Unmanaged Code (C, C++, native APIs)

**Features:**

- Runs directly on OS
- Manual memory management (malloc/free)
- No CLR overhead
- Direct hardware access

**Advantages:**

- ✅ Maximum performance
- ✅ Full control
- ✅ Access to OS features

#### Comparison

| Feature | Managed | Unmanaged |
|---------|---------|-----------|
| **Memory** | Automatic (GC) | Manual (malloc/free) |
| **Type Safety** | Yes | No |
| **Performance** | Good (JIT overhead) | Excellent (direct) |
| **Development** | Easy | Complex |
| **Platform** | Independent (IL) | Specific (native) |
| **Errors** | Exceptions | Crashes |

**Interoperability:** P/Invoke allows calling unmanaged code from managed code

---

### 11. Memory Structure

#### Stack (Per Thread)

**Characteristics:**

- Fast allocation/deallocation (LIFO - Last In, First Out)
- Limited size (~1MB per thread)
- Automatic cleanup (scope-based)

**Stores:**

- Value types (local variables)
- Method parameters
- Return addresses
- References to heap objects

#### Heap (Shared)

**Characteristics:**

- Slower allocation
- Large size (can be GBs)
- Garbage collected

**Stores:**

- Reference type objects (classes)
- Boxed value types
- Large arrays

#### Memory Layout

```
STACK (per thread)          HEAP (shared across threads)
┌──────────────────┐        ┌─────────────────────────┐
│ int x = 5;       │        │                         │
│ [5]              │        │                         │
│                  │        │                         │
│ Person p;        │        │  ┌──────────────────┐   │
│ [ref: 0x1234]────┼────────┼─→│ Person object    │   │
│                  │        │  │ Name: "John"     │   │
│ string s;        │        │  │ Age: 30          │   │
│ [ref: 0x5678]────┼────────┼─→└──────────────────┘   │
│                  │        │                         │
└──────────────────┘        │  ┌──────────────────┐   │
                            │  │ String "Hello"   │   │
                            │  └──────────────────┘   │
                            │                         │
                            │  [Other objects...]     │
                            └─────────────────────────┘
```

---

### 12. Value Types vs Reference Types (Memory)

#### Value Types

**Characteristics:**

- Stored on stack (local variables)
- Stored inline in containing type (if class field)
- Copy by value

**Types:**

- Primitives: int, double, bool, char, decimal, etc.
- Structs
- Enums

**Example:**
```csharp
int x = 10;           // Stack: [10]
int y = x;            // Stack: [10] (copy)
y = 20;               // x is still 10 (independent copy)
```

#### Reference Types

**Characteristics:**

- Object stored on heap
- Reference (pointer) stored on stack
- Copy by reference

**Types:**

- Classes
- Strings
- Arrays
- Delegates

**Example:**
```csharp
Person p1 = new Person { Age = 30 };  // Heap: [Person object]
Person p2 = p1;                       // p2 references same object
p2.Age = 40;                          // p1.Age is also 40 (same object)
```

#### Boxing and Unboxing

**Boxing:** Value type → Reference type (heap allocation)

```csharp
int x = 5;               // Stack: [5]
object obj = x;          // Boxing: heap allocation
                         // Stack: [ref] → Heap: [boxed int: 5]
```

**Unboxing:** Reference type → Value type (extract value)

```csharp
int y = (int)obj;        // Unboxing: copy from heap to stack
                         // Stack: [5]
```

**Performance Impact:**

- Heap allocation (GC pressure)
- Type checking
- Memory copying

**Avoid Boxing:**
```csharp
// ❌ Boxing
ArrayList list = new ArrayList();
list.Add(5);  // Boxing: int → object

// ✅ No boxing (use generics)
List<int> list = new List<int>();
list.Add(5);  // No boxing
```

---

### 13. Garbage Collection (GC)

**What is GC?** Automatic memory reclamation for heap objects

#### When GC Runs

- Automatically when Gen 0 fills up
- Memory pressure from OS
- Manually triggered (GC.Collect() - avoid)

#### GC Generations

**Generation 0** (Gen 0)

- Newly created objects
- Collected frequently
- Fast collection (~1ms)
- Short-lived objects (90% die here)

**Generation 1** (Gen 1)

- Objects that survived 1 GC
- Medium-lived objects
- Buffer between Gen 0 and Gen 2

**Generation 2** (Gen 2)

- Long-lived objects
- Collected infrequently
- Expensive collection (can pause app)
- Persistent objects (static fields, caches)

**Large Object Heap (LOH)**

- Objects ≥ 85,000 bytes
- Treated as Gen 2
- Not compacted (causes fragmentation)
- Use for large arrays, big strings

```
Generation Promotion:

new Object() → Gen 0
                │
         Survives GC
                ↓
              Gen 1
                │
         Survives GC
                ↓
              Gen 2
           (Long-lived)
```

#### GC Process

1. **Mark Phase**
   
   - Identify live objects (reachable from GC roots)
   - GC roots: static fields, local variables, CPU registers, active threads

2. **Sweep Phase**
   
   - Reclaim memory from dead (unreachable) objects
   - Add to free list

3. **Compact Phase**
   
   - Move live objects together (reduce fragmentation)
   - Not done for LOH (too expensive)

#### GC Modes

**Workstation GC** (Default)

- Lower latency
- Suitable for desktop apps
- Single GC thread

**Server GC**

- Higher throughput
- Suitable for server apps
- Multiple GC threads (one per CPU core)
- More aggressive

```xml
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
</PropertyGroup>
```

#### GC Methods

```csharp
// Check generation
int gen = GC.GetGeneration(obj);  // 0, 1, or 2

// Total memory
long bytes = GC.GetTotalMemory(false);

// Collect (avoid!)
GC.Collect();           // All generations
GC.Collect(0);          // Gen 0 only
GC.WaitForPendingFinalizers();

// Suppress finalization
GC.SuppressFinalize(this);
```

**Note:** Avoid calling GC.Collect() - GC is smarter than manual intervention!

---

### 14. Object Lifecycle

```
1. Allocation
   new Object()
        ↓
   Heap Memory (Gen 0)

2. Usage
   Object referenced and used
        ↓
   Still in Gen 0

3. Survives GC
   Promoted to Gen 1
        ↓
   Survives again → Gen 2

4. Becomes Unreachable
   No more references
        ↓
   Eligible for collection

5. Finalization (if has finalizer)
   ~Object() runs
        ↓
   One more GC cycle

6. Collection
   Memory reclaimed
        ↓
   Back to free list
```

**GC Roots** (what keeps objects alive):

- Static fields
- Local variables (on stack)
- Method parameters
- CPU registers
- Active threads
- Pinned objects (GCHandle)

---

### 15. Dispose Pattern (Brief Review)

**Purpose:** Deterministic cleanup of unmanaged resources

**IDisposable Interface:**
```csharp
public interface IDisposable
{
    void Dispose();
}
```

**Full Pattern:**
```csharp
public class MyResource : IDisposable
{
    private bool disposed = false;
    
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
                // Dispose managed resources
            }
            // Free unmanaged resources
            disposed = true;
        }
    }
    
    ~MyResource()  // Finalizer (only if unmanaged resources)
    {
        Dispose(false);
    }
}
```

**Using Statement:**
```csharp
using (var resource = new MyResource())
{
    // Use resource
}  // Dispose() called automatically
```

**Dispose vs Finalizer:**

| Feature | Dispose() | Finalizer (~) |
|---------|-----------|---------------|
| **When** | Deterministic | Non-deterministic (GC time) |
| **Performance** | Fast | Slow (2 GC cycles) |
| **Call** | Explicit | Automatic (GC) |
| **Recommended** | Yes | Only if unmanaged resources |

---

## Part 4: Type System

### 16. Common Type System (CTS)

**What is CTS?** Specification for how types are defined in .NET

**Purpose:** Enable cross-language interoperability

#### Type Categories

- **Value Types** - struct, enum
- **Reference Types** - class, interface, delegate, array
- **Pointer Types** - unsafe code only

#### Type Hierarchy

```
System.Object (root of all types)
├── Value Types (System.ValueType)
│   ├── Primitives
│   │   ├── Numeric: int, long, double, decimal
│   │   ├── Boolean: bool
│   │   └── Character: char
│   ├── Structs (user-defined value types)
│   └── Enums (System.Enum)
└── Reference Types
    ├── Classes (user-defined reference types)
    ├── Interfaces (contracts)
    ├── Delegates (System.Delegate)
    ├── Arrays (System.Array)
    └── Strings (System.String)
```

**Type Safety:** CLR enforces type rules at runtime (no invalid casts)

---

### 17. Common Language Specification (CLS)

**What is CLS?** Subset of CTS for language interoperability

**Purpose:** Define minimum requirements for .NET languages

**CLS-Compliant Code:**

- Can be consumed by all .NET languages
- Avoids language-specific features

**Non-CLS-Compliant Examples:**
```csharp
// ❌ Unsigned types not CLS-compliant
public uint GetValue() { return 42; }

// ✅ Use signed types in public APIs
public int GetValue() { return 42; }
```

**CLSCompliant Attribute:**
```csharp
[assembly: CLSCompliant(true)]  // Mark assembly as CLS-compliant
```

**Language Interoperability:**

- C# library → VB.NET ✅
- C# library → F# ✅
- If CLS-compliant!

---

### 18. Type Metadata

**What is Metadata?** Information about types stored in assembly

**Includes:**

- Type names and namespaces
- Member signatures (methods, properties, fields)
- Base types and interfaces
- Custom attributes
- Assembly references

**Used By:**

- CLR (type loading, method invocation)
- Reflection (runtime inspection)
- Serialization
- ORM frameworks (Entity Framework)
- Dependency injection containers

**Stored In:** Assembly manifest

---

## Part 5: Advanced Runtime Features

### 19. AppDomain (.NET Framework Only)

**What:** Isolated execution environment within process

**Features:**

- Application isolation
- Unload assemblies without restarting process
- Security boundaries

```csharp
// .NET Framework only
AppDomain domain = AppDomain.CreateDomain("MyDomain");
// Load and execute code
AppDomain.Unload(domain);
```

**Status:** Not available in .NET Core/.NET 5+ → Use AssemblyLoadContext

---

### 20. AssemblyLoadContext (.NET Core/.NET 5+)

**What:** Modern replacement for AppDomain

**Features:**

- Load multiple versions of same assembly
- Unload assemblies
- Plugin architectures
- Isolated dependency graphs

```csharp
var context = new AssemblyLoadContext("PluginContext", isCollectible: true);
var assembly = context.LoadFromAssemblyPath(pluginPath);

// Use assembly types
var type = assembly.GetType("MyPlugin.PluginClass");
dynamic instance = Activator.CreateInstance(type);
instance.Execute();

// Unload when done
context.Unload();
```

**Use Cases:**

- Plugin systems
- Hot reload
- Isolated testing
- Loading multiple versions of same dependency

---

### 21. unsafe Code and Pointers

**Enable in .csproj:**
```xml
<AllowUnsafeBlocks>true</AllowUnsafeBlocks>
```

**Pointer Operations:**
```csharp
unsafe
{
    int x = 10;
    int* ptr = &x;       // Address-of operator
    *ptr = 20;           // Dereference operator
    Console.WriteLine(x); // 20
}
```

**fixed Statement** (pin memory):
```csharp
unsafe
{
    int[] arr = {1, 2, 3};
    fixed (int* ptr = arr)
    {
        // arr won't move in memory during this block
        *ptr = 100;
    }
}
```

**stackalloc** (stack allocation):
```csharp
// Unsafe version
unsafe
{
    int* arr = stackalloc int[100];
}

// Safe version (C# 7.2+)
Span<int> arr = stackalloc int[100];  // No unsafe needed!
```

**When to Use:**

- Performance-critical code
- Native interop
- Low-level memory manipulation
- Graphics programming

**Warning:** Bypasses CLR safety checks (memory corruption possible)

---

### 22. Platform Invoke (P/Invoke)

**Purpose:** Call unmanaged code (C/C++ DLLs) from C#

**DllImport Attribute:**
```csharp
using System.Runtime.InteropServices;

[DllImport("user32.dll", CharSet = CharSet.Unicode)]
public static extern int MessageBox(
    IntPtr hWnd, 
    string text, 
    string caption, 
    uint type);

// Usage
MessageBox(IntPtr.Zero, "Hello from C#!", "P/Invoke", 0);
```

**Windows API Example:**
```csharp
[DllImport("kernel32.dll")]
static extern bool Beep(uint frequency, uint duration);

Beep(800, 200);  // Beep at 800Hz for 200ms
```

**Marshalling:** Converting between managed and unmanaged types

- Automatic for simple types (int, bool, float)
- Manual for complex types (structs, strings, arrays)

**Use Cases:**

- Windows API calls
- Legacy native libraries
- Performance-critical native code
- Hardware access

---

## Part 6: Preprocessor Directives

**Definition:** Compiler instructions (not C# code), start with #

### Conditional Compilation

```csharp
#define DEBUG
#define FEATURE_X

#if DEBUG
    Console.WriteLine("Debug mode");
#elif FEATURE_X
    Console.WriteLine("Feature X enabled");
#else
    Console.WriteLine("Release mode");
#endif
```

### Compiler Messages

```csharp
#warning This code is deprecated
#error This feature is not yet implemented
```

### Code Organization

```csharp
#region Database Access
public void SaveToDatabase() { }
public void LoadFromDatabase() { }
#endregion
```

### Warning Control

```csharp
#pragma warning disable CS0618  // Disable obsolete warning
// Use obsolete member
#pragma warning restore CS0618   // Re-enable warning
```

### Nullable Context (C# 8.0+)

```csharp
#nullable enable
string? nullableString = null;  // OK
string nonNullable = null;      // Warning

#nullable disable
string anyString = null;        // No warning

#nullable restore  // Restore project setting
```

### Common Symbols

```csharp
#if DEBUG       // Debug configuration
#if RELEASE     // Release configuration
#if WINDOWS     // Windows platform
#if LINUX       // Linux platform
#if NET8_0      // .NET 8.0 target
```

---

## Part 7: Environment & Console

### Environment Class

```csharp
using System;

// System information
string machine = Environment.MachineName;
string user = Environment.UserName;
string os = Environment.OSVersion.ToString();
int cpus = Environment.ProcessorCount;
bool is64Bit = Environment.Is64BitOperatingSystem;

// Directories
string currentDir = Environment.CurrentDirectory;
string systemDir = Environment.SystemDirectory;

// Version
Version version = Environment.Version;  // CLR version

// Environment variables
string path = Environment.GetEnvironmentVariable("PATH");
Environment.SetEnvironmentVariable("MY_VAR", "value");

// Special folders
string desktop = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
string docs = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);

// Exit application
Environment.Exit(0);  // Exit code 0 = success
```

### Console Class

```csharp
// Input
string line = Console.ReadLine();
int ch = Console.Read();
ConsoleKeyInfo key = Console.ReadKey();

// Output
Console.Write("No newline");
Console.WriteLine("With newline");
Console.WriteLine("Value: {0}", value);

// Colors
Console.ForegroundColor = ConsoleColor.Red;
Console.BackgroundColor = ConsoleColor.White;
Console.ResetColor();

// Cursor
Console.Clear();
Console.SetCursorPosition(10, 5);
Console.CursorVisible = false;

// Sound
Console.Beep();
Console.Beep(800, 200);  // Frequency, duration
```

---

## Quick Reference Summary

### .NET Ecosystem

- **.NET Framework** - Windows only, legacy
- **.NET Core** - Cross-platform, superseded
- **.NET 5+** - Modern, unified platform ✅
- **Annual releases** - LTS every other version (6, 8, etc.)

### CLR (Common Language Runtime)

- **Execution engine** for .NET
- **Two-stage compilation** - C# → IL → Native code
- **JIT compilation** - IL to native at runtime
- **Tiered compilation** - Fast startup + optimized steady-state
- **AOT compilation** - Compile before deployment (faster startup)

### Memory Management

- **Stack** - Fast, limited, value types
- **Heap** - Slower, large, reference types, garbage collected
- **GC Generations** - Gen 0 (young), Gen 1 (medium), Gen 2 (old), LOH (≥85KB)
- **GC Process** - Mark → Sweep → Compact
- **Dispose Pattern** - Deterministic cleanup (IDisposable)

### Type System

- **CTS** - Common Type System (type specification)
- **CLS** - Common Language Specification (interoperability)
- **Value types** - Stack, copy by value (int, struct, enum)
- **Reference types** - Heap, copy by reference (class, string, array)
- **Boxing/Unboxing** - Convert between value and reference (avoid!)

### Advanced Features

- **AssemblyLoadContext** - Load/unload assemblies, plugins
- **unsafe code** - Pointers, unmanaged memory
- **P/Invoke** - Call native DLLs
- **Preprocessor** - #if, #define, #region, #nullable

---

**Guide Complete!** You now understand what happens under the hood when C# code runs! 📘