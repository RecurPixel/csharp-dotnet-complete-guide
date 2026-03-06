# C# Attributes & Reflection Guide

---

## Part 1: Understanding the Metadata System

### The Big Picture

```

┌────────────────────────────────────────────────────┐
│ METADATA SYSTEM                                    │
│                                                    │
│ ┌──────────────┐         ┌──────────────┐          │
│ │ ATTRIBUTES   │────────>│ REFLECTION   │          │
│ │ (Add data)   │         │ (Read data)  │          │
│ └──────────────┘         └──────────────┘          │
│        │                        │                  │
│        │                        │                  │
│        v                        v                  │
│  [Validation]         GetCustomAttribute()         │
│  [Route("api")]       GetProperties()              │
│  [Obsolete]           GetMethods()                 │
│  [Serializable]       Activator.CreateInstance()   │
│                                                    │
└────────────────────────────────────────────────────┘

```

### What Are Attributes?

**Simple Definition:** Tags you put on code (classes, methods, properties) to add metadata

**Think of it like:** Sticky notes on code that frameworks can read later

```csharp
[Obsolete("Use NewMethod instead")] // ← Attribute (sticky note)
public void OldMethod() { }         // ← Your code

[Required]                          // ← Attribute
public string Name { get; set; }    // ← Your property
```

**Key Point:** Attributes do NOTHING by themselves. Something else (framework or your code) must READ them using Reflection.

### What Is Reflection?

**Definition:** Examining and manipulating code at runtime

**Common Uses:**

- Read attributes
- Get type information
- Create instances dynamically
- Invoke methods dynamically
- Access private members (testing)

**Namespace:** System.Reflection

---

## Part 2: Built-in Attributes

### 1. [Obsolete] - Mark Old Code

```csharp
// Warning
[Obsolete]
public void OldMethod() { }

// Warning with message
[Obsolete("Use NewMethod instead")]
public void OldMethod() { }

// Compiler ERROR
[Obsolete("Don't use this!", error: true)]
public void OldMethod() { }
```

### 2. [Serializable] - Allow Serialization

```csharp
[Serializable]
public class User
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    [NonSerialized]
    private string _temporaryData;  // Won't be serialized
}
```

### 3. [DllImport] - Call Native Code

```csharp
using System.Runtime.InteropServices;

[DllImport("user32.dll")]
public static extern int MessageBox(
    IntPtr hWnd, 
    string text, 
    string caption, 
    uint type);

// Usage
MessageBox(IntPtr.Zero, "Hello", "Title", 0);
```

### 4. [CallerMemberName] - Get Caller Info

```csharp
using System.Runtime.CompilerServices;

public void Log(
    string message,
    [CallerMemberName] string caller = "",
    [CallerFilePath] string file = "",
    [CallerLineNumber] int line = 0)
{
    Console.WriteLine($"{caller} ({file}:{line}): {message}");
}

// Usage
void MyMethod()
{
    Log("Something happened");
    // Output: MyMethod (Program.cs:42): Something happened
}
```

### 5. [Flags] - Enum as Bit Flags

```csharp
[Flags]
public enum FileAccess
{
    None = 0,       // 000
    Read = 1,       // 001
    Write = 2,      // 010
    Execute = 4     // 100
}

// Usage
var access = FileAccess.Read | FileAccess.Write;  // 011 (3)
bool canRead = (access & FileAccess.Read) != 0;   // true
```

### 6. [Conditional] - Conditional Compilation

```csharp
using System.Diagnostics;

[Conditional("DEBUG")]
public void DebugLog(string message)
{
    Console.WriteLine(message); // Only runs in DEBUG mode
}

// Usage
DebugLog("Debug info"); // Removed in Release build
```

---

## Part 3: Creating Custom Attributes

### Step 1: Define Attribute Class

**Rules:**

1. Must inherit from `System.Attribute`
2. Name should end with "Attribute" (convention)
3. Add `[AttributeUsage]` to specify where it can be used

```csharp
// Simple attribute (no parameters)
public class MyAttribute : Attribute
{
}

// Attribute with parameters
public class DescriptionAttribute : Attribute
{
    public string Text { get; }
    
    public DescriptionAttribute(string text)
    {
        Text = text;
    }
}

// Attribute with named parameters
public class ValidationAttribute : Attribute
{
    public string ErrorMessage { get; set; }
    public int MinLength { get; set; }
    public int MaxLength { get; set; }
}
```

### Step 2: Control Where Attribute Can Be Used

```csharp
// Only on classes
[AttributeUsage(AttributeTargets.Class)]
public class TableAttribute : Attribute
{
    public string Name { get; }
    public TableAttribute(string name) => Name = name;
}

// Only on properties
[AttributeUsage(AttributeTargets.Property)]
public class ColumnAttribute : Attribute
{
    public string Name { get; }
    public ColumnAttribute(string name) => Name = name;
}

// Multiple targets
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class LogAttribute : Attribute { }

// Allow multiple instances
[AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
public class RouteAttribute : Attribute
{
    public string Path { get; }
    public RouteAttribute(string path) => Path = path;
}
```

### AttributeTargets Options

```csharp
AttributeTargets.Assembly          // [assembly: AssemblyVersion("1.0")]
AttributeTargets.Module            // Rarely used
AttributeTargets.Class             // [Table("Users")]
AttributeTargets.Struct            // [Serializable]
AttributeTargets.Enum              // [Flags]
AttributeTargets.Constructor       // [Obsolete]
AttributeTargets.Method            // [Route("api/users")]
AttributeTargets.Property          // [Required]
AttributeTargets.Field             // [NonSerialized]
AttributeTargets.Event             // [Description("Fired when...")]
AttributeTargets.Interface         // [ServiceContract]
AttributeTargets.Parameter         // void Method([FromBody] string data)
AttributeTargets.Delegate          // Rarely used
AttributeTargets.ReturnValue       // [return: MarshalAs(...)]
AttributeTargets.GenericParameter  // class MyClass<[SomeAttribute] T>
AttributeTargets.All               // Any of the above
```

### Step 3: Using Custom Attributes

```csharp
// Usage (Note: "Attribute" suffix is optional)
[Description("This is a user")]
public class User { }

// Same as:
[DescriptionAttribute("This is a user")]
public class User { }

// With named parameters
[Validation(ErrorMessage = "Invalid", MinLength = 5, MaxLength = 50)]
public string Username { get; set; }

// Multiple attributes
[Table("Users")]
[Description("User entity")]
[Serializable]
public class User { }

// Multiple instances (if AllowMultiple = true)
[Route("api/users")]
[Route("api/v1/users")]
public class UserController { }
```

---

## Part 4: Reflection Basics

### Getting Type Information

```csharp
// Three ways to get Type
Type type1 = typeof(User);              // Compile-time
Type type2 = user.GetType();            // Runtime (from instance)
Type type3 = Type.GetType("MyNamespace.User"); // From string

// Basic type info
string name = type.Name;                // "User"
string fullName = type.FullName;        // "MyNamespace.User"
string namespaceName = type.Namespace;  // "MyNamespace"
bool isClass = type.IsClass;            // true
bool isInterface = type.IsInterface;    // false
bool isAbstract = type.IsAbstract;      // false
bool isSealed = type.IsSealed;          // false
bool isPublic = type.IsPublic;          // true
Type baseType = type.BaseType;          // Base class type
Type[] interfaces = type.GetInterfaces(); // All interfaces
```

### Reflection Classes

```csharp
Type              // Represents a type (class, interface, etc.)
Assembly          // Represents a .dll or .exe
MemberInfo        // Base for all members
PropertyInfo      // Property information
MethodInfo        // Method information
FieldInfo         // Field information
ConstructorInfo   // Constructor information
EventInfo         // Event information
ParameterInfo     // Parameter information
```

---

## Part 5: Reading Attributes

### Understanding Attribute Retrieval Methods

|Method|Returns|Use When|
|---|---|---|
|`GetCustomAttribute<T>()`|Single T or null|Expect ONE attribute|
|`GetCustomAttributes<T>()`|IEnumerable<T>|Expect MULTIPLE attributes|
|`GetCustomAttributes(type, inherit)`|object[]|Legacy, needs casting|
|`IsDefined(type, inherit)`|bool|Just checking existence|

### Single Attribute on Class

```csharp
// Define
[Table("Users")]
public class User { }

// Read - Modern way (Generic)
Type type = typeof(User);
TableAttribute attr = type.GetCustomAttribute<TableAttribute>();
if (attr != null)
{
    Console.WriteLine(attr.Name); // "Users"
}

// Read - Legacy way (Non-generic, requires cast)
object[] attrs = type.GetCustomAttributes(typeof(TableAttribute), false);
if (attrs.Length > 0)
{
    TableAttribute attr = (TableAttribute)attrs[0]; // Must cast!
    Console.WriteLine(attr.Name);
}

// Just check if exists
bool hasTable = type.IsDefined(typeof(TableAttribute), false);
```

### Multiple Attributes on Class

```csharp
// Define
[Route("api/users")]
[Route("api/v1/users")]
public class UserController { }

// ✅ Method 1: Modern Generic (PREFERRED)
Type type = typeof(UserController);
IEnumerable<RouteAttribute> routes = type.GetCustomAttributes<RouteAttribute>();
foreach (var route in routes)
{
    Console.WriteLine(route.Path);
}

// ✅ Method 2: Legacy Non-Generic (Requires casting)
object[] routesArray = type.GetCustomAttributes(typeof(RouteAttribute), false);
foreach (object attr in routesArray)
{
    RouteAttribute route = (RouteAttribute)attr; // Must cast!
    Console.WriteLine(route.Path);
}

// ❌ WRONG: Using GetCustomAttribute (singular) for multiple
var route = type.GetCustomAttribute<RouteAttribute>(); // Only gets first!
```

### Attributes on Properties

```csharp
// Define
public class User
{
    [Required]
    [StringLength(50)]
    public string Name { get; set; }
    
    [Column("email_address")]
    public string Email { get; set; }
}

// Read from single property
PropertyInfo prop = typeof(User).GetProperty("Name");
var required = prop.GetCustomAttribute<RequiredAttribute>();
var stringLength = prop.GetCustomAttribute<StringLengthAttribute>();

// Read from all properties
Type type = typeof(User);
foreach (PropertyInfo prop in type.GetProperties())
{
    var columnAttr = prop.GetCustomAttribute<ColumnAttribute>();
    if (columnAttr != null)
    {
        Console.WriteLine($"{prop.Name} -> {columnAttr.Name}");
    }
}
```

### Attributes on Methods

```csharp
// Define
public class UserService
{
    [Obsolete("Use GetUserByIdAsync instead")]
    public User GetUser(int id) => null;
    
    [HttpGet("api/users/{id}")]
    public async Task<User> GetUserByIdAsync(int id) => null;
}

// Read
MethodInfo method = typeof(UserService).GetMethod("GetUser");
var obsolete = method.GetCustomAttribute<ObsoleteAttribute>();
if (obsolete != null)
{
    Console.WriteLine(obsolete.Message);
}
```

### Attributes on Parameters

```csharp
// Define
public void UpdateUser([FromBody] User user, [FromQuery] int id) { }

// Read
MethodInfo method = typeof(MyController).GetMethod("UpdateUser");
ParameterInfo[] parameters = method.GetParameters();
foreach (var param in parameters)
{
    var fromBody = param.GetCustomAttribute<FromBodyAttribute>();
    if (fromBody != null)
    {
        Console.WriteLine($"{param.Name} is from body");
    }
}
```

---

## Part 6: Attribute Filtering Patterns

### Pattern 1: Find All Classes with Specific Attribute

```csharp
// Find all classes decorated with [Developer] attribute
Assembly assembly = Assembly.GetExecutingAssembly();
var classesWithAttr = assembly.GetTypes()
    .Where(t => t.IsClass && !t.IsAbstract)
    .Where(t => t.IsDefined(typeof(DeveloperAttribute), false));

foreach (var type in classesWithAttr)
{
    Console.WriteLine(type.Name);
}
```

### Pattern 2: Find Classes Where Attribute Matches Condition

```csharp
// Find classes developed by specific person
string targetDeveloper = "Arthor";
var matchingClasses = assembly.GetTypes()
    .Where(t => t.IsClass && !t.IsAbstract)
    .Where(t => t.GetCustomAttributes<DeveloperAttribute>()
        .Any(d => d.Name == targetDeveloper));
```

### Pattern 3: Get All Attribute Values Across All Classes

```csharp
// Get all unique developer names
var allDevelopers = assembly.GetTypes()
    .Where(t => t.IsClass && !t.IsAbstract)
    .SelectMany(t => t.GetCustomAttributes<DeveloperAttribute>())
    .Select(d => d.Name)
    .Distinct();

foreach (var name in allDevelopers)
{
    Console.WriteLine(name);
}
```

### Pattern 4: Group Classes by Attribute Value

```csharp
// Group classes by developer
var grouped = assembly.GetTypes()
    .Where(t => t.IsClass && !t.IsAbstract)
    .SelectMany(t => t.GetCustomAttributes<DeveloperAttribute>()
        .Select(d => new { Type = t, Developer = d }))
    .GroupBy(x => x.Developer.Name);

foreach (var group in grouped)
{
    Console.WriteLine($"\nDeveloper: {group.Key}");
    foreach (var item in group)
    {
        Console.WriteLine($"  - {item.Type.Name} v{item.Developer.Version}");
    }
}
```

### Pattern 5: Complex Filtering (Version Range)

```csharp
// Find classes with version between 2.0 and 4.0
var versionRange = assembly.GetTypes()
    .Where(t => t.IsClass && !t.IsAbstract)
    .Where(t => t.GetCustomAttributes<DeveloperAttribute>()
        .Any(d => d.Version >= 2.0 && d.Version <= 4.0));
```

---

## Part 7: Common Mistakes & Best Practices

### ❌ Mistake 1: Forgetting to Cast (Legacy Method)

```csharp
// WRONG
object[] attrs = type.GetCustomAttributes(typeof(DeveloperAttribute), false);
Console.WriteLine(attrs[0].Name); // ❌ Error! object doesn't have Name

// RIGHT
DeveloperAttribute dev = (DeveloperAttribute)attrs[0]; // ✅ Cast first
Console.WriteLine(dev.Name);
```

### ❌ Mistake 2: Using Singular Method for Multiple Attributes

```csharp
// WRONG (only gets first attribute)
var attr = type.GetCustomAttribute<DeveloperAttribute>(); // ❌ Loses others

// RIGHT
var attrs = type.GetCustomAttributes<DeveloperAttribute>(); // ✅ Gets all
```

### ❌ Mistake 3: Not Checking for Null/Empty

```csharp
// WRONG
var attr = type.GetCustomAttribute<DeveloperAttribute>();
Console.WriteLine(attr.Name); // ❌ NullReferenceException if not found

// RIGHT
var attr = type.GetCustomAttribute<DeveloperAttribute>();
if (attr != null)
{
    Console.WriteLine(attr.Name);
}

// Or use null-conditional operator
Console.WriteLine(attr?.Name ?? "No developer");
```

### ✅ Best Practice: Use Generic Methods

```csharp
// ❌ Old way (verbose, requires casting)
object[] attrs = type.GetCustomAttributes(typeof(DeveloperAttribute), false);
foreach (object attr in attrs)
{
    DeveloperAttribute dev = (DeveloperAttribute)attr;
    Console.WriteLine(dev.Name);
}

// ✅ Modern way (clean, type-safe)
foreach (DeveloperAttribute dev in type.GetCustomAttributes<DeveloperAttribute>())
{
    Console.WriteLine(dev.Name);
}
```

---

## Part 8: Real-World Patterns

### Pattern 1: Validation Framework

```csharp
// 1. Define validation attributes
public class RequiredAttribute : Attribute { }

public class RangeAttribute : Attribute
{
    public int Min { get; }
    public int Max { get; }
    
    public RangeAttribute(int min, int max)
    {
        Min = min;
        Max = max;
    }
}

public class EmailAttribute : Attribute { }

// 2. Use on model
public class User
{
    [Required]
    public string Name { get; set; }
    
    [Required]
    [Email]
    public string Email { get; set; }
    
    [Range(18, 100)]
    public int Age { get; set; }
}

// 3. Create validator
public static class Validator
{
    public static List<string> Validate(object obj)
    {
        var errors = new List<string>();
        Type type = obj.GetType();
        
        foreach (PropertyInfo prop in type.GetProperties())
        {
            object value = prop.GetValue(obj);
            
            // Check Required
            if (prop.IsDefined(typeof(RequiredAttribute)))
            {
                if (value == null || string.IsNullOrWhiteSpace(value.ToString()))
                {
                    errors.Add($"{prop.Name} is required");
                }
            }
            
            // Check Range
            var rangeAttr = prop.GetCustomAttribute<RangeAttribute>();
            if (rangeAttr != null && value is int intValue)
            {
                if (intValue < rangeAttr.Min || intValue > rangeAttr.Max)
                {
                    errors.Add($"{prop.Name} must be between {rangeAttr.Min} and {rangeAttr.Max}");
                }
            }
            
            // Check Email
            if (prop.IsDefined(typeof(EmailAttribute)))
            {
                if (value != null && !value.ToString().Contains("@"))
                {
                    errors.Add($"{prop.Name} must be a valid email");
                }
            }
        }
        
        return errors;
    }
}

// 4. Usage
var user = new User { Name = "", Email = "invalid", Age = 15 };
var errors = Validator.Validate(user);
foreach (var error in errors)
{
    Console.WriteLine(error);
}
```

### Pattern 2: Simple ORM (Object-Relational Mapping)

```csharp
// 1. Define attributes
[AttributeUsage(AttributeTargets.Class)]
public class TableAttribute : Attribute
{
    public string Name { get; }
    public TableAttribute(string name) => Name = name;
}

[AttributeUsage(AttributeTargets.Property)]
public class ColumnAttribute : Attribute
{
    public string Name { get; }
    public ColumnAttribute(string name) => Name = name;
}

[AttributeUsage(AttributeTargets.Property)]
public class PrimaryKeyAttribute : Attribute { }

// 2. Use on model
[Table("users")]
public class User
{
    [PrimaryKey]
    [Column("user_id")]
    public int Id { get; set; }
    
    [Column("user_name")]
    public string Name { get; set; }
    
    [Column("email_address")]
    public string Email { get; set; }
    
    public int Age { get; set; } // No attribute = ignored
}

// 3. Create SQL generator
public static class SqlGenerator
{
    public static string GenerateInsert<T>(T obj)
    {
        Type type = typeof(T);
        
        // Get table name
        var tableAttr = type.GetCustomAttribute<TableAttribute>();
        string tableName = tableAttr?.Name ?? type.Name;
        
        var columns = new List<string>();
        var values = new List<string>();
        
        foreach (PropertyInfo prop in type.GetProperties())
        {
            // Skip if no Column attribute
            var columnAttr = prop.GetCustomAttribute<ColumnAttribute>();
            if (columnAttr == null) continue;
            
            // Skip primary key (usually auto-increment)
            if (prop.IsDefined(typeof(PrimaryKeyAttribute))) continue;
            
            columns.Add(columnAttr.Name);
            object value = prop.GetValue(obj);
            string valueStr = value is string ? $"'{value}'" : value.ToString();
            values.Add(valueStr);
        }
        
        return $"INSERT INTO {tableName} ({string.Join(", ", columns)}) " +
               $"VALUES ({string.Join(", ", values)})";
    }
    
    public static string GenerateSelect<T>()
    {
        Type type = typeof(T);
        var tableAttr = type.GetCustomAttribute<TableAttribute>();
        string tableName = tableAttr?.Name ?? type.Name;
        
        var columns = new List<string>();
        foreach (PropertyInfo prop in type.GetProperties())
        {
            var columnAttr = prop.GetCustomAttribute<ColumnAttribute>();
            if (columnAttr != null)
            {
                columns.Add(columnAttr.Name);
            }
        }
        
        return $"SELECT {string.Join(", ", columns)} FROM {tableName}";
    }
}

// 4. Usage
var user = new User
{
    Name = "John",
    Email = "john@example.com",
    Age = 30
};

string insertSql = SqlGenerator.GenerateInsert(user);
Console.WriteLine(insertSql);
// INSERT INTO users (user_name, email_address) VALUES ('John', 'john@example.com')

string selectSql = SqlGenerator.GenerateSelect<User>();
Console.WriteLine(selectSql);
// SELECT user_id, user_name, email_address FROM users
```

### Pattern 3: API Route Registration

```csharp
// 1. Define route attribute
[AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
public class RouteAttribute : Attribute
{
    public string Method { get; }  // GET, POST, etc.
    public string Path { get; }
    
    public RouteAttribute(string method, string path)
    {
        Method = method;
        Path = path;
    }
}

// 2. Use on controller
public class UserController
{
    [Route("GET", "/api/users")]
    public List<User> GetAllUsers()
    {
        return new List<User>();
    }
    
    [Route("GET", "/api/users/{id}")]
    public User GetUser(int id)
    {
        return null;
    }
    
    [Route("POST", "/api/users")]
    public void CreateUser(User user)
    {
    }
}

// 3. Create route scanner
public class RouteScanner
{
    public static void RegisterRoutes(Type controllerType)
    {
        foreach (MethodInfo method in controllerType.GetMethods())
        {
            var routes = method.GetCustomAttributes<RouteAttribute>();
            foreach (var route in routes)
            {
                Console.WriteLine($"{route.Method} {route.Path} -> {method.Name}");
            }
        }
    }
}

// 4. Usage
RouteScanner.RegisterRoutes(typeof(UserController));
// Output:
// GET /api/users -> GetAllUsers
// GET /api/users/{id} -> GetUser
// POST /api/users -> CreateUser
```

---

## Part 9: Working with Types

### Creating Instances Dynamically

```csharp
// Method 1: Activator.CreateInstance (simple)
User user1 = (User)Activator.CreateInstance(typeof(User));

// Method 2: Activator.CreateInstance with parameters
var user2 = (User)Activator.CreateInstance(
    typeof(User),
    new object[] { "John", 25 } // Constructor params
);

// Method 3: Using ConstructorInfo
Type type = typeof(User);
ConstructorInfo ctor = type.GetConstructor(new[] { typeof(string), typeof(int) });
var user3 = (User)ctor.Invoke(new object[] { "John", 25 });

// Method 4: Generic constraint
T CreateInstance<T>() where T : new()
{
    return new T(); // Requires parameterless constructor
}
```

### Invoking Methods Dynamically

```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
    public int Multiply(int a, int b) => a * b;
}

// Get method
Type type = typeof(Calculator);
MethodInfo method = type.GetMethod("Add");

// Invoke
Calculator calc = new Calculator();
object result = method.Invoke(calc, new object[] { 5, 3 });
Console.WriteLine(result); // 8

// Invoke static method
// MethodInfo staticMethod = type.GetMethod("StaticMethod");
// object result = staticMethod.Invoke(null, parameters);
```

### Getting/Setting Properties Dynamically

```csharp
public class User
{
    public string Name { get; set; }
    public int Age { get; set; }
}

User user = new User();
PropertyInfo nameProp = typeof(User).GetProperty("Name");

// Set value
nameProp.SetValue(user, "John");

// Get value
object value = nameProp.GetValue(user);
Console.WriteLine(value); // John
```

### Working with Generic Types

```csharp
// Define generic class
public class Repository<T> where T : class
{
    public void Save(T entity) { }
}

// Get generic type info
Type genericType = typeof(Repository<>);     // Open generic
Type closedType = typeof(Repository<User>);  // Closed generic

Console.WriteLine(genericType.IsGenericType);           // true
Console.WriteLine(genericType.IsGenericTypeDefinition); // true
Console.WriteLine(closedType.IsGenericType);            // true
Console.WriteLine(closedType.IsGenericTypeDefinition);  // false

// Get generic arguments
Type[] typeArgs = closedType.GetGenericArguments();
Console.WriteLine(typeArgs[0].Name); // "User"

// Create instance of generic type
Type repoType = typeof(Repository<>).MakeGenericType(typeof(User));
object repo = Activator.CreateInstance(repoType);
```

---

## Part 10: Performance Considerations

### Reflection is SLOW

```csharp
// ❌ Slow: Reflection in loop
for (int i = 0; i < 1000000; i++)
{
    var method = typeof(MyClass).GetMethod("MyMethod");
    method.Invoke(instance, null);
}

// ✅ Fast: Cache reflection results
var method = typeof(MyClass).GetMethod("MyMethod");
for (int i = 0; i < 1000000; i++)
{
    method.Invoke(instance, null);
}

// ✅ Faster: Compile to delegate
var method = typeof(MyClass).GetMethod("MyMethod");
var action = (Action)Delegate.CreateDelegate(typeof(Action), instance, method);
for (int i = 0; i < 1000000; i++)
{
    action(); // Much faster
}
```

### Caching Pattern

```csharp
using System.Collections.Concurrent;

public static class AttributeCache
{
    private static readonly ConcurrentDictionary<Type, object> _cache = new();
    
    public static T GetAttribute<T>(Type type) where T : Attribute
    {
        return (T)_cache.GetOrAdd(type, t => t.GetCustomAttribute<T>());
    }
}
```

---

## Part 11: Common Reflection Methods Reference

### Type Information

```csharp
Type type = typeof(User);

type.Name                    // "User"
type.FullName                // "MyNamespace.User"
type.IsClass                 // true
type.IsInterface             // false
type.IsAbstract              // false
type.IsSealed                // false
type.BaseType                // typeof(object)
type.GetInterfaces()         // All interfaces
```

### Attributes

```csharp
type.GetCustomAttribute<T>()        // Single attribute
type.GetCustomAttributes<T>()       // Multiple attributes
type.IsDefined(typeof(T))           // Check if has attribute
```

### Constructors

```csharp
type.GetConstructor(Type[])         // Specific constructor
type.GetConstructors()              // All public constructors
```

### Properties

```csharp
type.GetProperty("Name")            // Specific property
type.GetProperties()                // All public properties
prop.GetValue(obj)                  // Get property value
prop.SetValue(obj, value)           // Set property value
prop.GetCustomAttribute<T>()        // Property attribute
```

### Methods

```csharp
type.GetMethod("MethodName")        // Specific method
type.GetMethods()                   // All public methods
method.Invoke(instance, params)     // Invoke method
method.GetCustomAttribute<T>()      // Method attribute
```

### Fields

```csharp
type.GetField("fieldName")          // Specific field
type.GetFields()                    // All public fields
```

### Assemblies

```csharp
Assembly.GetExecutingAssembly()     // Current assembly
Assembly.GetTypes()                 // All types in assembly
```

---

## Quick Reference Summary

### When to Use Attributes

✅ **Use for:**

- Validation rules
- API routing
- Database mapping
- Serialization control
- Testing metadata
- Configuration

❌ **Don't use for:**

- Business logic (use methods)
- Performance-critical paths

### When to Use Reflection

✅ **Use for:**

- Framework/library code
- Plugin systems
- ORM implementation
- Dependency injection
- Test frameworks
- Code generation tools

❌ **Don't use for:**

- Application code (usually)
- Tight loops (cache results)

### How Frameworks Use This

**ASP.NET Core:**

- Reads `[HttpGet]` to register routes
- Reads `[Required]` to validate

**Entity Framework:**

- Reads `[Table]`, `[Column]` to generate SQL

**Dependency Injection:**

- Uses reflection to create instances

---

**Guide Complete!** You now have a comprehensive Attributes & Reflection reference! 🏷️