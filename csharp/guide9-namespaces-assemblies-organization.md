# C# Namespaces, Assemblies & Project Organization Quick Reference

---

## Part 1: Namespaces

### 1. What are Namespaces?

**Purpose:**

- Organize code into logical groups
- Avoid naming conflicts
- Provide clear hierarchy and structure

**Namespace vs Assembly:**

- **Namespace** - Logical organization (in code)
- **Assembly** - Physical packaging (.dll or .exe file)
- One assembly can contain multiple namespaces
- One namespace can span multiple assemblies

```csharp
// Different classes with same name in different namespaces
namespace MyCompany.UI
{
    public class Button { }  // UI button
}

namespace MyCompany.Hardware
{
    public class Button { }  // Physical button sensor
}
```

---

### 2. Declaring Namespaces

#### Block-scoped (Traditional)

```csharp
namespace MyCompany.MyApp
{
    public class MyClass
    {
        public void Method() { }
    }
}
```

#### File-scoped (C# 10.0+) - Modern ✅

```csharp
namespace MyCompany.MyApp;  // Applies to entire file

public class MyClass
{
    public void Method() { }
}
// No extra indentation needed!
```

#### Nested Namespaces

```csharp
// Option 1: Nested blocks (old way)
namespace Outer
{
    namespace Inner
    {
        public class MyClass { }
    }
}

// Option 2: Dot notation (preferred)
namespace Outer.Inner
{
    public class MyClass { }
}

// Option 3: File-scoped (C# 10.0+)
namespace Outer.Inner;

public class MyClass { }
```

---

### 3. Using Directives

#### Basic using Statement

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// Now can use types without fully qualified names
List<int> numbers = new List<int>();  // Instead of System.Collections.Generic.List<int>
```

#### using static (C# 6.0+)

```csharp
using static System.Console;
using static System.Math;

// Can use static members directly
WriteLine("Hello");       // Instead of Console.WriteLine
double result = Sqrt(16); // Instead of Math.Sqrt
```

#### using Alias

```csharp
// Alias for namespace
using MyUtils = MyCompany.Utilities;
MyUtils.Helper.DoSomething();

// Alias for type
using StringBuilder = System.Text.StringBuilder;
StringBuilder sb = new StringBuilder();

// Resolve naming conflicts
using WinForms = System.Windows.Forms;
using WPF = System.Windows;

WinForms.Button button1 = new WinForms.Button();
WPF.Controls.Button button2 = new WPF.Controls.Button();
```

#### Alias any Type (C# 12.0+)

```csharp
// Alias tuples
using Point = (int X, int Y);
Point p = (10, 20);
Console.WriteLine(p.X);

// Alias complex types
using StringList = System.Collections.Generic.List<string>;
StringList names = new StringList();

// Alias delegates
using IntOperation = System.Func<int, int, int>;
IntOperation add = (a, b) => a + b;
```

#### Global using (C# 10.0+)

```csharp
// In any .cs file (typically GlobalUsings.cs)
global using System;
global using System.Collections.Generic;
global using System.Linq;

// Available in ALL files in the project (no need to repeat)
```

---

### 4. Implicit Usings (.NET 6.0+)

**What are Implicit Usings?**
Common namespaces automatically included based on project type.

#### Enable in .csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

#### Default Implicit Usings by Project Type

**Console App:**
```csharp
// Automatically included:
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net.Http;
using System.Threading;
using System.Threading.Tasks;
```

**Web API / ASP.NET Core:**
```csharp
// Automatically included:
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net.Http;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Routing;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
```

#### Add Custom Implicit Usings

```xml
<ItemGroup>
  <Using Include="MyCompany.Common" />
  <Using Include="Newtonsoft.Json" />
</ItemGroup>
```

#### Remove Implicit Usings

```xml
<ItemGroup>
  <Using Remove="System.Net.Http" />
</ItemGroup>
```

---

### 5. Namespace Best Practices

✅ **Naming Conventions:**
```csharp
// Format: <CompanyName>.<ProductName>.<Feature>.<SubFeature>
namespace Microsoft.EntityFrameworkCore.Metadata;
namespace MyCompany.ECommerce.Orders;
namespace Acme.Inventory.Products.Categories;
```

✅ **Match Folder Structure:**
```
Project/
├── Services/
│   └── OrderService.cs      // namespace MyApp.Services
├── Models/
│   └── Order.cs             // namespace MyApp.Models
└── Repositories/
    └── OrderRepository.cs   // namespace MyApp.Repositories
```

✅ **Avoid System.***
```csharp
// ❌ Bad
namespace System.MyCompany;

// ✅ Good
namespace MyCompany.System;  // If you need "System" in name
```

✅ **One Namespace Per File** (guideline)
```csharp
// Preferred
namespace MyCompany.Orders;

public class Order { }
public class OrderItem { }
```

❌ **Don't:**

- Use too many nested levels (3-4 max)
- Create namespace for single class
- Mix unrelated types in one namespace

---

## Part 2: Essential Namespaces Reference

### Core Namespaces ⭐⭐⭐ (Must Know)

#### System

**Purpose:** Fundamental types and base classes

**Key Types:**
```csharp
using System;

// Fundamental
Object, String, Array, Console
ValueType, Enum, Delegate, Exception
Type, Attribute

// Math & Numbers
Math, Random, Convert
Int32, Int64, Double, Decimal, Boolean

// Date & Time
DateTime, DateTimeOffset, TimeSpan
DateOnly, TimeOnly (.NET 6+)

// Other
Guid, Tuple, ValueTuple
Nullable<T>, Lazy<T>
Environment, GC

// Common operations
Console.WriteLine("Hello");
Math.Sqrt(16);
DateTime.Now;
Guid.NewGuid();
Convert.ToInt32("123");
```

#### System.Collections.Generic

**Purpose:** Generic collection types

**Key Types:**
```csharp
using System.Collections.Generic;

// Most Common
List<T>                    // Dynamic array
Dictionary<TKey, TValue>   // Hash table
HashSet<T>                 // Unique items
Queue<T>                   // FIFO
Stack<T>                   // LIFO

// Less Common
SortedSet<T>              // Sorted unique items
SortedDictionary<TKey, TValue>  // Sorted key-value
LinkedList<T>             // Doubly-linked list

// Interfaces
IEnumerable<T>            // Iteration
ICollection<T>            // Size + add/remove
IList<T>                  // Indexed access
IDictionary<TKey, TValue> // Key-value access

// Usage
List<int> numbers = new List<int>();
Dictionary<string, int> ages = new Dictionary<string, int>();
HashSet<string> uniqueNames = new HashSet<string>();
```

#### System.Linq

**Purpose:** LINQ query operators

**Key Types:**
```csharp
using System.Linq;

// Main class (all extension methods)
Enumerable

// Key types
IGrouping<TKey, TElement>
ILookup<TKey, TElement>

// Usage
var evens = numbers.Where(n => n % 2 == 0);
var sorted = names.OrderBy(n => n);
var first = items.FirstOrDefault();
var grouped = people.GroupBy(p => p.Age);
int sum = numbers.Sum();
double avg = numbers.Average();
```

#### System.Threading.Tasks

**Purpose:** Async/await and task-based programming

**Key Types:**
```csharp
using System.Threading.Tasks;

// Core
Task                      // Async operation (no result)
Task<T>                   // Async operation (with result)
ValueTask<T>              // Performance optimization

// Advanced
TaskCompletionSource<T>   // Manual task control
TaskFactory               // Task creation options
Parallel                  // Parallel operations (CPU-bound)

// Usage
await Task.Delay(1000);
string result = await DownloadAsync();
await Task.WhenAll(task1, task2);
Parallel.ForEach(items, item => Process(item));
```

#### System.IO

**Purpose:** File and stream I/O

**Key Types:**
```csharp
using System.IO;

// File operations (static)
File                      // File.ReadAllText, WriteAllText, etc.
Directory                 // Directory.GetFiles, Exists, etc.
Path                      // Path.Combine, GetFileName, etc.

// File operations (instance)
FileInfo                  // File metadata + operations
DirectoryInfo             // Directory metadata + operations

// Streams
Stream                    // Base class
FileStream                // File I/O
MemoryStream              // In-memory buffer
BufferedStream            // Buffered I/O

// Readers/Writers
StreamReader              // Text reading
StreamWriter              // Text writing
BinaryReader              // Binary reading
BinaryWriter              // Binary writing

// Usage
string content = File.ReadAllText("file.txt");
using var reader = new StreamReader("file.txt");
string[] files = Directory.GetFiles("folder");
string path = Path.Combine("folder", "file.txt");
```

#### System.Text

**Purpose:** Text encoding and manipulation

**Key Types:**
```csharp
using System.Text;

// Main classes
StringBuilder             // Efficient string building
Encoding                  // Character encoding

// Common encodings
Encoding.UTF8             // UTF-8 (most common)
Encoding.ASCII            // ASCII
Encoding.Unicode          // UTF-16

// Usage
StringBuilder sb = new StringBuilder();
sb.Append("Hello").Append(" ").Append("World");
byte[] bytes = Encoding.UTF8.GetBytes("Hello");
string text = Encoding.UTF8.GetString(bytes);
```

---

### Common Namespaces ⭐⭐ (Frequently Used)

#### System.Text.RegularExpressions

```csharp
using System.Text.RegularExpressions;

// Pattern matching
Regex.IsMatch("test@email.com", @"^[\w.]+@[\w.]+$");
Match match = Regex.Match("Hello 123", @"\d+");
string replaced = Regex.Replace("Hello 123", @"\d+", "###");
```

#### System.Threading

```csharp
using System.Threading;

// Threading primitives
Thread, ThreadPool
Monitor, Mutex, Semaphore, SemaphoreSlim
CancellationToken, CancellationTokenSource
Interlocked
Lock (C# 13.0+)

// Usage
CancellationTokenSource cts = new();
await DoWorkAsync(cts.Token);
Interlocked.Increment(ref counter);
```

#### System.Diagnostics

```csharp
using System.Diagnostics;

// Debugging
Debug.WriteLine("Debug message");
Trace.WriteLine("Trace message");

// Performance
Stopwatch sw = Stopwatch.StartNew();
// ... code ...
sw.Stop();
Console.WriteLine(sw.ElapsedMilliseconds);

// Process info
Process current = Process.GetCurrentProcess();
```

#### System.Net.Http

```csharp
using System.Net.Http;

// HTTP client
HttpClient client = new HttpClient();
string response = await client.GetStringAsync("https://api.example.com");
HttpResponseMessage result = await client.PostAsync(url, content);
```

#### System.Text.Json

```csharp
using System.Text.Json;

// JSON serialization (.NET Core 3.0+)
string json = JsonSerializer.Serialize(obj);
MyObject obj = JsonSerializer.Deserialize<MyObject>(json);

JsonDocument doc = JsonDocument.Parse(json);
JsonElement root = doc.RootElement;
```

#### System.Collections.Concurrent

```csharp
using System.Collections.Concurrent;

// Thread-safe collections
ConcurrentDictionary<TKey, TValue>
ConcurrentQueue<T>
ConcurrentStack<T>
ConcurrentBag<T>
BlockingCollection<T>

// Usage
ConcurrentDictionary<int, string> dict = new();
dict.TryAdd(1, "one");
```

---

### Specialized Namespaces ⭐ (As Needed)

#### System.Reflection

```csharp
using System.Reflection;

// Runtime type inspection
Type type = typeof(MyClass);
MethodInfo method = type.GetMethod("MyMethod");
PropertyInfo[] props = type.GetProperties();
Assembly assembly = Assembly.GetExecutingAssembly();
```

#### System.Numerics

```csharp
using System.Numerics;

// Large numbers
BigInteger huge = BigInteger.Pow(2, 1000);

// Complex numbers
Complex c = new Complex(3, 4);

// SIMD vectors
Vector<int> v = new Vector<int>(new int[] { 1, 2, 3, 4 });
```

#### System.Security.Cryptography

```csharp
using System.Security.Cryptography;

// Hashing
using SHA256 sha = SHA256.Create();
byte[] hash = sha.ComputeHash(data);

// Random (cryptographically secure)
byte[] randomBytes = RandomNumberGenerator.GetBytes(32);
```

---

### Common Using Combinations by Scenario

#### Console Application
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
```

#### File I/O
```csharp
using System;
using System.IO;
using System.Text;
```

#### Async Programming
```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
```

#### Data Processing
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text.Json;
```

#### Web API Client
```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;
using System.Text.Json;
```

#### Database Operations
```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Threading.Tasks;
```

---

## Part 3: Assemblies

### 10. What are Assemblies?

**Definition:** Compiled code library (.dll or .exe)

**Assembly Components:**
```
Assembly (.dll or .exe)
├── Manifest
│   ├── Assembly metadata (name, version, culture)
│   ├── List of files in assembly
│   ├── Referenced assemblies
│   └── Security permissions
├── Type Metadata
│   ├── Type definitions
│   ├── Method signatures
│   └── Custom attributes
├── IL Code (MSIL/CIL)
│   └── Intermediate Language bytecode
└── Resources (optional)
    ├── Images
    ├── Strings
    └── Other embedded files
```

---

### 11. Assembly Types

**Private Assemblies**

- Application-specific
- Located in application folder
- No versioning conflicts
- Most common type

**Shared Assemblies**

- Stored in GAC (Global Assembly Cache)
- Requires strong name
- Shared across applications
- Rarely used in modern .NET

**Static Assemblies**

- Stored on disk (.dll or .exe files)
- Standard assemblies

**Dynamic Assemblies**

- Created in memory at runtime
- Using Reflection.Emit
- Advanced scenarios only

---

### 12. DLL vs EXE

#### DLL (Dynamic Link Library)

```csharp
// Class library project
// .csproj: <OutputType>Library</OutputType>

namespace MyLibrary
{
    public class Helper
    {
        public static void DoSomething() { }
    }
}
// Compiles to MyLibrary.dll
```

#### EXE (Executable)

```csharp
// Console/desktop app project
// .csproj: <OutputType>Exe</OutputType>

namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            // Entry point
        }
    }
}
// Compiles to MyApp.exe
```

### DLL vs EXE Comparison

| Feature | DLL | EXE |
|---------|-----|-----|
| **Execution** | Cannot run independently | Can run independently |
| **Entry Point** | No Main() method | Has Main() method |
| **Purpose** | Library/component | Application |
| **Extension** | .dll | .exe |
| **Sharing** | Can be shared by multiple apps | Standalone |
| **Referenced by** | Other projects | N/A (top-level) |

---

### 13. Strong Naming

**What is Strong Naming?**
Signing assembly with public/private key pair for uniqueness and security.

**Components:**

- Simple name (assembly name: "MyLibrary")
- Version number (1.0.0.0)
- Culture information (en-US, neutral, etc.)
- Public key token (from key pair)

**Creating Strong-Named Assembly:**

1. Generate key pair:
```bash
sn.exe -k MyKey.snk
```

2. Add to .csproj:
```xml
<PropertyGroup>
  <AssemblyOriginatorKeyFile>MyKey.snk</AssemblyOriginatorKeyFile>
  <SignAssembly>true</SignAssembly>
</PropertyGroup>
```

**Benefits:**

- ✅ Uniqueness guarantee (no name collisions)
- ✅ Version enforcement
- ✅ Tamper protection
- ✅ Can be placed in GAC
- ✅ Trust level for security

**When to Use:**

- Shared libraries (GAC deployment)
- Versioning critical
- Security requirements
- Large organizations

---

### 14. Assembly Versioning

#### Version Format

```
Major.Minor.Build.Revision
  │     │     │       │
  1  .  2  .  3  .  4

Example: 1.0.0.0, 2.1.3.4
```

#### Version Types

```csharp
// AssemblyVersion - Used by CLR for binding
[assembly: AssemblyVersion("1.0.0.0")]

// AssemblyFileVersion - File system version
[assembly: AssemblyFileVersion("1.0.0.1")]

// AssemblyInformationalVersion - Display version
[assembly: AssemblyInformationalVersion("1.0.0-beta")]
```

#### Semantic Versioning (SemVer)

```
MAJOR.MINOR.PATCH

Example: 2.1.3

MAJOR (2): Breaking changes
MINOR (1): New features (backward compatible)
PATCH (3): Bug fixes
```

**SemVer Rules:**

- Increment MAJOR: Breaking changes
- Increment MINOR: New features, backward compatible
- Increment PATCH: Bug fixes only

---

### 15. Assembly Loading

#### Loading Methods

```csharp
// Load by name (searches GAC and probing paths)
Assembly asm1 = Assembly.Load("MyAssembly");

// Load from specific path
Assembly asm2 = Assembly.LoadFrom(@"C:\Path\MyAssembly.dll");

// Load file without dependencies
Assembly asm3 = Assembly.LoadFile(@"C:\Path\MyAssembly.dll");
```

#### Probing Paths

CLR searches for assemblies in:

1. Application base directory
2. Private bin path (if specified)
3. GAC (for strong-named assemblies)

---

### 16. Assembly Scope & Visibility

#### internal Access Modifier

```csharp
// Only visible within same assembly
internal class InternalClass { }

public class PublicClass
{
    internal void InternalMethod() { }
}
```

#### InternalsVisibleTo Attribute

```csharp
// In MyLibrary.csproj or AssemblyInfo.cs
[assembly: InternalsVisibleTo("MyLibrary.Tests")]

// Now MyLibrary.Tests can access internal members

// For strong-named assemblies
[assembly: InternalsVisibleTo("MyLibrary.Tests, PublicKey=...")]
```

**Common Use Case: Unit Testing**
```csharp
// In library
internal class InternalHelper { }

// In test project (can access because of InternalsVisibleTo)
var helper = new InternalHelper();
```

---

## Part 4: Project Structure

### 17. .NET Project File (.csproj)

#### SDK-Style Project (Modern)

```xml
<Project Sdk="Microsoft.NET.Sdk">
  
  <PropertyGroup>
    <!-- Target Framework -->
    <TargetFramework>net8.0</TargetFramework>
    
    <!-- Output Type -->
    <OutputType>Exe</OutputType>  <!-- Exe, Library, WinExe -->
    
    <!-- Language Version -->
    <LangVersion>12.0</LangVersion>  <!-- Latest, 12.0, 13.0, etc. -->
    
    <!-- Nullable Reference Types -->
    <Nullable>enable</Nullable>  <!-- enable, disable, warnings, annotations -->
    
    <!-- Implicit Usings -->
    <ImplicitUsings>enable</ImplicitUsings>
    
    <!-- Root Namespace -->
    <RootNamespace>MyCompany.MyApp</RootNamespace>
    
    <!-- Assembly Name -->
    <AssemblyName>MyApp</AssemblyName>
    
    <!-- XML Documentation -->
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
  </PropertyGroup>
  
  <!-- NuGet Package References -->
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
    <PackageReference Include="Serilog" Version="3.1.1" />
  </ItemGroup>
  
  <!-- Project References -->
  <ItemGroup>
    <ProjectReference Include="..\MyLibrary\MyLibrary.csproj" />
  </ItemGroup>
  
</Project>
```

---

### 18. Solution Files (.sln)

**What is a Solution?**
Container for one or more projects.

**Benefits:**
- Build multiple projects together
- Manage dependencies
- Share configurations
- Organize related projects

**Solution Structure:**
```
MySolution.sln
├── MyApp (Console App)
├── MyLibrary (Class Library)
├── MyApp.Tests (Test Project)
└── Solution Items
    ├── README.md
    └── .editorconfig
```

**Build Configurations:**

- Debug (development)
- Release (production)
- Custom configurations

---

### 19. NuGet Packages

**Package Sources:**

- nuget.org (official public repository)
- Private feeds (Azure Artifacts, MyGet)
- Local folders

**Package Versioning:**
```xml
<!-- Exact version -->
<PackageReference Include="MyPackage" Version="[1.2.3]" />

<!-- Minimum version -->
<PackageReference Include="MyPackage" Version="1.2.3" />

<!-- Version range -->
<PackageReference Include="MyPackage" Version="[1.0, 2.0)" />

<!-- Floating version -->
<PackageReference Include="MyPackage" Version="1.*" />
```

**Common NuGet Packages:**
```xml
<ItemGroup>
  <!-- JSON -->
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  <PackageReference Include="System.Text.Json" Version="8.0.0" />
  
  <!-- Logging -->
  <PackageReference Include="Serilog" Version="3.1.1" />
  <PackageReference Include="NLog" Version="5.2.8" />
  
  <!-- Testing -->
  <PackageReference Include="xunit" Version="2.6.6" />
  <PackageReference Include="NUnit" Version="4.1.0" />
  <PackageReference Include="Moq" Version="4.20.70" />
  
  <!-- ORM -->
  <PackageReference Include="Dapper" Version="2.1.28" />
  <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
  
  <!-- HTTP -->
  <PackageReference Include="RestSharp" Version="110.2.0" />
  
  <!-- Object Mapping -->
  <PackageReference Include="AutoMapper" Version="12.0.1" />
</ItemGroup>
```

---

### 20. Target Frameworks

#### Available Frameworks

**.NET Framework (Windows only)** - Legacy
```xml
<TargetFramework>net48</TargetFramework>
<!-- net48, net472, net471, net47, net462, net461, net46, net452, net451, net45 -->
```

**.NET Core (cross-platform)** - Superseded
```xml
<TargetFramework>netcoreapp3.1</TargetFramework>
<!-- netcoreapp3.1, netcoreapp3.0, netcoreapp2.2, netcoreapp2.1 -->
```

**.NET 5+ (unified platform)** - Modern ✅
```xml
<TargetFramework>net8.0</TargetFramework>
<!-- net9.0, net8.0, net7.0, net6.0, net5.0 -->
```

**.NET Standard (API specification)** - For libraries
```xml
<TargetFramework>netstandard2.0</TargetFramework>
<!-- netstandard2.1, netstandard2.0, netstandard1.6 -->
```

#### Multi-Targeting

```xml
<!-- Support multiple frameworks -->
<TargetFrameworks>net8.0;net6.0;net48</TargetFrameworks>
```

#### Framework Compatibility Matrix

| Target | Can Reference |
|--------|--------------|
| **.NET 8.0** | .NET 8.0, .NET Standard 2.1, .NET Standard 2.0 |
| **.NET 6.0** | .NET 6.0, .NET Standard 2.1, .NET Standard 2.0 |
| **.NET Framework 4.8** | .NET Framework 4.8, .NET Standard 2.0 |
| **.NET Standard 2.1** | .NET Standard 2.1, .NET Standard 2.0, .NET Standard 1.x |
| **.NET Standard 2.0** | .NET Standard 2.0, .NET Standard 1.x |

---

### 21. Project Organization Best Practices

#### Recommended Folder Structure

```
MySolution/
├── src/                              # Source code
│   ├── MyApp/                        # Main application
│   │   ├── Program.cs
│   │   └── MyApp.csproj
│   ├── MyApp.Core/                   # Business logic
│   │   ├── Models/
│   │   ├── Services/
│   │   ├── Interfaces/
│   │   └── MyApp.Core.csproj
│   ├── MyApp.Infrastructure/         # Data access, external services
│   │   ├── Data/
│   │   ├── Repositories/
│   │   └── MyApp.Infrastructure.csproj
│   └── MyApp.Api/                    # Web API (if applicable)
│       ├── Controllers/
│       ├── DTOs/
│       └── MyApp.Api.csproj
├── tests/                            # Test projects
│   ├── MyApp.UnitTests/
│   │   └── MyApp.UnitTests.csproj
│   └── MyApp.IntegrationTests/
│       └── MyApp.IntegrationTests.csproj
├── docs/                             # Documentation
│   ├── README.md
│   └── architecture.md
├── tools/                            # Build scripts, utilities
│   └── build.ps1
├── .gitignore
├── .editorconfig
├── global.json
├── Directory.Build.props              # Shared MSBuild properties
└── MySolution.sln
```

#### Namespace = Folder Path

```
MyApp.Core/
├── Services/
│   └── OrderService.cs          // namespace MyApp.Core.Services
├── Models/
│   └── Order.cs                 // namespace MyApp.Core.Models
└── Repositories/
    └── OrderRepository.cs       // namespace MyApp.Core.Repositories
```

#### Best Practices

✅ **Do:**

- One type per file (easier navigation, better source control)
- Match namespace to folder structure
- Use partial classes for generated code
- Separate concerns (UI, Business Logic, Data Access)
- Group related projects in solution folders

❌ **Don't:**

- Mix unrelated code in same project
- Create circular dependencies
- Put everything in root folder
- Use "Utilities" or "Helpers" folders (be specific)

#### Separation of Concerns

```
Presentation Layer (UI)
    ↓
Business Logic Layer (Core)
    ↓
Data Access Layer (Infrastructure)
    ↓
Database
```

---

## Quick Reference Summary

### Namespaces

- **Purpose:** Organize code, avoid conflicts
- **Modern syntax:** File-scoped (C# 10+)
- **Global usings:** Available across all files (C# 10+)
- **Implicit usings:** Auto-included by project type (.NET 6+)

### Essential Namespaces

- **System** - Fundamentals (Object, String, Console, Math, DateTime)
- **System.Collections.Generic** - Collections (List, Dictionary, HashSet)
- **System.Linq** - LINQ operators
- **System.Threading.Tasks** - Async/await (Task, Task\<T\>)
- **System.IO** - File operations
- **System.Text** - StringBuilder, Encoding

### Assemblies

- **Definition:** Compiled code (.dll or .exe)
- **DLL:** Library (no Main method)
- **EXE:** Application (has Main method)
- **Strong naming:** Sign with key for uniqueness/security
- **Versioning:** Major.Minor.Build.Revision

### Project Organization

- **.csproj:** Project file (SDK-style for modern .NET)
- **.sln:** Solution file (container for projects)
- **NuGet:** Package management
- **Target frameworks:** net8.0, net6.0, net48, netstandard2.0
- **Folder structure:** src/, tests/, docs/, tools/

---

**Guide Complete!** This reference will help you organize your C# projects professionally! 📘