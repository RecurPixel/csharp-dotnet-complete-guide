# ASP.NET Core Entity Framework Core - Complete Guide
## Practical Guide + Technical Reference

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. EF Core Basics
2. 3 Ways to Set Up EF Core
3. DbContext Essentials
4. Defining Models
5. CRUD Operations
6. Migrations
7. Relationships
8. Querying Patterns
9. Troubleshooting Common Issues
10. Best Practices

### Part 2: Technical Reference (Deep Dive)
11. Important Interfaces & Classes Reference
12. Configuration Deep-Dive
13. Advanced Topics
14. Performance Optimization

---

# PART 1: PRACTICAL GUIDE

---

## 1. EF Core Basics

**Simple Definition:** Entity Framework Core is an ORM (Object-Relational Mapper) that lets you work with databases using C# objects instead of SQL.

**Think of it like:** A translator between your C# code and the database. You work with regular C# classes, and EF Core handles all the SQL behind the scenes.

```
C# Code (Objects) ←→ [EF Core] ←→ Database (Tables)
    User user;              SQL: SELECT * FROM Users
    user.Name = "John";     SQL: UPDATE Users SET Name = 'John'
```

### What is an ORM?

**Without ORM (Raw SQL):**
```csharp
// Complex, error-prone
var command = "SELECT * FROM Users WHERE Id = @id";
var reader = connection.ExecuteReader(command, new { id = 5 });
// Manual mapping...
```

**With ORM (EF Core):**
```csharp
// Simple, type-safe
var user = context.Users.Find(5);
```

### Code-First vs Database-First

| Approach | When to Use | Starting Point |
|----------|-------------|----------------|
| **Code-First** 🎯 | New projects, full control | Write C# classes → Generate database |
| **Database-First** | Legacy databases, existing DB | Database exists → Generate C# classes |

**This guide focuses on Code-First** (most common for new ASP.NET Core projects)

### When to Use EF Core

✅ **Use EF Core when:**
- Building CRUD applications
- Working with relational databases (SQL Server, PostgreSQL, MySQL)
- Need type-safe queries
- Want automatic change tracking
- Prefer C# over SQL

❌ **Don't use EF Core when:**
- Need maximum performance (use Dapper)
- Complex stored procedures (better in SQL)
- Very simple read-only queries (use ADO.NET)
- NoSQL databases (use native drivers)

### EF Core Evolution Timeline

```
EF Core 1.0 (2016) → Basic functionality
EF Core 2.0 (2017) → Table splitting, owned entities
EF Core 3.0 (2019) → Async streams, nullable reference types
EF Core 5.0 (2020) → Many-to-many, table-per-type
EF Core 6.0 (2021) → Temporal tables, compiled models
EF Core 7.0 (2022) → Bulk operations, JSON columns
EF Core 8.0 (2023) → Complex types, primitive collections ✨ Latest
```

---

## 2. 3 Ways to Set Up EF Core

### Method 1: SQL Server (Most Common) 🎯 RECOMMENDED

**When to use:**
- ✅ Production applications
- ✅ Team development
- ✅ Enterprise projects
- ✅ Windows or Azure hosting

**Step 1: Install NuGet Package**

```bash
# .NET CLI
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

```xml
<!-- Or in .csproj -->
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.0" />
```

**Step 2: Create DbContext**

```csharp
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    // Define your tables
    public DbSet<User> Users { get; set; }
    public DbSet<Product> Products { get; set; }
}

// Entity classes
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

**Step 3: Configure Connection String**

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyAppDb;Trusted_Connection=true;TrustServerCertificate=true;"
  }
}
```

**Step 4: Register DbContext in Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add DbContext
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();
```

**Step 5: Create Initial Migration**

```bash
# Creates Migrations folder with migration files
dotnet ef migrations add InitialCreate
```

**Step 6: Update Database**

```bash
# Creates/updates the database
dotnet ef database update
```

**Complete Working Example:**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllers();

var app = builder.Build();
app.MapControllers();
app.Run();

// ApplicationDbContext.cs
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options) { }
    
    public DbSet<User> Users { get; set; }
}

// User.cs
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

// UsersController.cs
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    
    public UsersController(ApplicationDbContext context)
    {
        _context = context;
    }
    
    [HttpGet]
    public async Task<ActionResult<List<User>>> GetUsers()
    {
        return await _context.Users.ToListAsync();
    }
    
    [HttpPost]
    public async Task<ActionResult<User>> CreateUser(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(GetUsers), new { id = user.Id }, user);
    }
}
```

---

### Method 2: SQLite (Development & Learning) 📚 GREAT FOR LEARNING

**When to use:**
- ✅ Local development
- ✅ Learning EF Core
- ✅ Testing/prototyping
- ✅ Lightweight applications
- ✅ No database server needed

**Step 1: Install Package**

```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

**Step 2: Connection String**

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=app.db"
  }
}
```

**Step 3: Register DbContext**

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
```

**Step 4: Create & Run Migrations**

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

**Complete SQLite Example:**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// SQLite - creates app.db file in project folder
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite("Data Source=app.db"));

builder.Services.AddControllers();

var app = builder.Build();

// Auto-apply migrations on startup (dev only!)
using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    db.Database.Migrate();
}

app.MapControllers();
app.Run();
```

**Advantages:**
- Zero configuration
- File-based (app.db)
- Perfect for demos
- Works everywhere

---

### Method 3: In-Memory Database (Testing) 🧪

**When to use:**
- ✅ Unit testing
- ✅ Integration testing
- ✅ Temporary data
- ❌ NOT for production

**Step 1: Install Package**

```bash
dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

**Step 2: Configure In-Memory Database**

```csharp
// For testing
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseInMemoryDatabase("TestDb"));
```

**Complete Testing Example:**

```csharp
// UserServiceTests.cs
public class UserServiceTests
{
    private ApplicationDbContext GetInMemoryDbContext()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString()) // Unique per test
            .Options;
        
        return new ApplicationDbContext(options);
    }
    
    [Fact]
    public async Task CreateUser_ShouldAddUser()
    {
        // Arrange
        using var context = GetInMemoryDbContext();
        var service = new UserService(context);
        var user = new User { Name = "John", Email = "john@example.com" };
        
        // Act
        await service.CreateUserAsync(user);
        
        // Assert
        Assert.Equal(1, await context.Users.CountAsync());
    }
}
```

---

### Database Provider Comparison

| Provider | NuGet Package | Connection String Example | Best For |
|----------|---------------|---------------------------|----------|
| **SQL Server** | EntityFrameworkCore.SqlServer | `Server=.;Database=MyDb;...` | Production |
| **PostgreSQL** | Npgsql.EntityFrameworkCore.PostgreSQL | `Host=localhost;Database=mydb;...` | Open source |
| **MySQL** | Pomelo.EntityFrameworkCore.MySql | `server=localhost;database=mydb;...` | LAMP stack |
| **SQLite** | EntityFrameworkCore.Sqlite | `Data Source=app.db` | Development |
| **In-Memory** | EntityFrameworkCore.InMemory | `InMemoryDatabase("TestDb")` | Testing |

---

## 3. DbContext Essentials

**What is DbContext?**

DbContext is your gateway to the database. It:
- Represents a session with the database
- Tracks changes to entities
- Manages database connections
- Executes queries and commands

**Think of it like:** A shopping cart that tracks what you want to buy (add/modify/delete) and commits all changes at once when you checkout (SaveChanges).

### Basic DbContext Structure

```csharp
public class ApplicationDbContext : DbContext
{
    // 1. Constructor (required for DI)
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    // 2. DbSet properties (your tables)
    public DbSet<User> Users { get; set; }
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }
    
    // 3. OnModelCreating (optional configuration)
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Configure entities here
        modelBuilder.Entity<User>(entity =>
        {
            entity.HasIndex(e => e.Email).IsUnique();
            entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
        });
    }
}
```

### DbSet<T> Properties

```csharp
public DbSet<User> Users { get; set; }
```

This one line creates:
- A table named "Users"
- Queryable collection of User entities
- Add/Remove/Update capabilities

### OnConfiguring vs OnModelCreating

| Method | Purpose | When to Use |
|--------|---------|-------------|
| `OnConfiguring` | Configure DbContext options | Rarely (use DI instead) |
| `OnModelCreating` | Configure entity mappings | Always (Fluent API) |

**OnConfiguring (Avoid in ASP.NET Core):**
```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    // ❌ Don't hardcode connection strings
    optionsBuilder.UseSqlServer("Server=...;Database=...;");
}
```

**OnModelCreating (Use This):**
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    
    // ✅ Configure entities using Fluent API
    modelBuilder.Entity<User>(entity =>
    {
        entity.HasKey(e => e.Id);
        entity.Property(e => e.Email).IsRequired();
        entity.HasMany(e => e.Orders)
              .WithOne(e => e.User)
              .HasForeignKey(e => e.UserId);
    });
}
```

### 3 Ways to Configure DbContext

#### Configuration Method 1: Inline/Hardcoded (Quick Testing)

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer("Server=localhost;Database=TestDb;Trusted_Connection=true;"));
```

**Pros:** Quick, simple
**Cons:** Hardcoded, insecure, can't change environments

---

#### Configuration Method 2: From Configuration (Standard) 🎯

```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;..."
  }
}

// Program.cs
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));
```

**Pros:** Environment-specific (appsettings.Development.json, appsettings.Production.json)
**Cons:** Connection string visible in config

---

#### Configuration Method 3: With Options Pattern (Advanced)

```csharp
// DatabaseSettings.cs
public class DatabaseSettings
{
    public string ConnectionString { get; set; }
    public int CommandTimeout { get; set; }
    public bool EnableSensitiveDataLogging { get; set; }
}

// appsettings.json
{
  "Database": {
    "ConnectionString": "Server=...;",
    "CommandTimeout": 30,
    "EnableSensitiveDataLogging": false
  }
}

// Program.cs
builder.Services.Configure<DatabaseSettings>(
    builder.Configuration.GetSection("Database"));

builder.Services.AddDbContext<ApplicationDbContext>((serviceProvider, options) =>
{
    var dbSettings = serviceProvider.GetRequiredService<IOptions<DatabaseSettings>>().Value;
    
    options.UseSqlServer(dbSettings.ConnectionString, sqlOptions =>
    {
        sqlOptions.CommandTimeout(dbSettings.CommandTimeout);
    });
    
    if (dbSettings.EnableSensitiveDataLogging)
    {
        options.EnableSensitiveDataLogging();
    }
});
```

**Pros:** Type-safe, testable, flexible
**Cons:** More complex

---

### Complete DbContext Example

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    // Tables
    public DbSet<User> Users { get; set; }
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // User configuration
        modelBuilder.Entity<User>(entity =>
        {
            entity.ToTable("Users");
            entity.HasKey(e => e.Id);
            entity.HasIndex(e => e.Email).IsUnique();
            entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
            entity.Property(e => e.Email).IsRequired().HasMaxLength(200);
            entity.Property(e => e.CreatedAt).HasDefaultValueSql("GETUTCDATE()");
        });
        
        // Product configuration
        modelBuilder.Entity<Product>(entity =>
        {
            entity.ToTable("Products");
            entity.Property(e => e.Price).HasPrecision(18, 2);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(200);
        });
        
        // Order configuration
        modelBuilder.Entity<Order>(entity =>
        {
            entity.ToTable("Orders");
            entity.HasOne(e => e.User)
                  .WithMany(e => e.Orders)
                  .HasForeignKey(e => e.UserId)
                  .OnDelete(DeleteBehavior.Cascade);
        });
        
        // Seed data
        modelBuilder.Entity<Product>().HasData(
            new Product { Id = 1, Name = "Laptop", Price = 999.99m },
            new Product { Id = 2, Name = "Mouse", Price = 29.99m }
        );
    }
}
```

---

## 4. Defining Models (Entity Classes)

### Simple Entity

```csharp
public class User
{
    public int Id { get; set; }          // Primary key (by convention)
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

### Primary Key Conventions

EF Core automatically recognizes these as primary keys:
- Property named `Id`
- Property named `{ClassName}Id` (e.g., `UserId` for User class)

**3 Ways to Define Primary Key:**

#### Method 1: By Convention (Simple)

```csharp
public class User
{
    public int Id { get; set; }  // ✅ Automatically recognized
}
```

#### Method 2: Data Annotations

```csharp
using System.ComponentModel.DataAnnotations;

public class User
{
    [Key]
    public int UserId { get; set; }  // Explicitly marked as key
}
```

#### Method 3: Fluent API (Flexible) 🎯

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>()
        .HasKey(u => u.UserId);
    
    // Composite key
    modelBuilder.Entity<OrderItem>()
        .HasKey(oi => new { oi.OrderId, oi.ProductId });
}
```

---

### Data Annotations

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class User
{
    [Key]
    public int Id { get; set; }
    
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Required]
    [EmailAddress]
    [StringLength(200)]
    public string Email { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal Balance { get; set; }
    
    [NotMapped]  // Not stored in database
    public string FullName => $"{FirstName} {LastName}";
    
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

**Common Data Annotations:**

| Annotation | Purpose |
|------------|---------|
| `[Key]` | Primary key |
| `[Required]` | NOT NULL constraint |
| `[StringLength(100)]` | VARCHAR(100) |
| `[MaxLength(500)]` | Maximum length |
| `[MinLength(10)]` | Minimum length |
| `[EmailAddress]` | Email format validation |
| `[Phone]` | Phone format |
| `[Range(1, 100)]` | Value range |
| `[Column(TypeName = "decimal(18,2)")]` | SQL type |
| `[Table("Users")]` | Table name |
| `[NotMapped]` | Exclude from DB |
| `[ForeignKey("UserId")]` | Foreign key |

---

### Navigation Properties

**One-to-Many Example:**

```csharp
// Principal (One)
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property - collection
    public List<Order> Orders { get; set; }
}

// Dependent (Many)
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    
    // Foreign key
    public int UserId { get; set; }
    
    // Navigation property - single
    public User User { get; set; }
}
```

**How to Use:**

```csharp
// Load user with orders
var user = await context.Users
    .Include(u => u.Orders)  // Eager loading
    .FirstOrDefaultAsync(u => u.Id == 1);

Console.WriteLine($"{user.Name} has {user.Orders.Count} orders");
```

---

### Complete Model Examples

```csharp
// User.cs
public class User
{
    public int Id { get; set; }
    
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Required]
    [EmailAddress]
    public string Email { get; set; }
    
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    
    // Navigation property
    public List<Order> Orders { get; set; } = new();
}

// Product.cs
public class Product
{
    public int Id { get; set; }
    
    [Required]
    [StringLength(200)]
    public string Name { get; set; }
    
    [StringLength(500)]
    public string Description { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal Price { get; set; }
    
    public int StockQuantity { get; set; }
    
    // Navigation property
    public List<OrderItem> OrderItems { get; set; } = new();
}

// Order.cs
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; } = DateTime.UtcNow;
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal TotalAmount { get; set; }
    
    // Foreign key
    public int UserId { get; set; }
    
    // Navigation properties
    public User User { get; set; }
    public List<OrderItem> OrderItems { get; set; } = new();
}

// OrderItem.cs (Join table)
public class OrderItem
{
    public int Id { get; set; }
    public int Quantity { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal UnitPrice { get; set; }
    
    // Foreign keys
    public int OrderId { get; set; }
    public int ProductId { get; set; }
    
    // Navigation properties
    public Order Order { get; set; }
    public Product Product { get; set; }
}
```

---

## 5. CRUD Operations

### Create (C)

#### Add Single Entity

```csharp
[HttpPost]
public async Task<ActionResult<User>> CreateUser(User user)
{
    // Method 1: Add
    _context.Users.Add(user);
    await _context.SaveChangesAsync();
    
    return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
}
```

#### Add Multiple Entities

```csharp
public async Task AddMultipleUsers(List<User> users)
{
    // Method 1: AddRange (recommended)
    _context.Users.AddRange(users);
    await _context.SaveChangesAsync();
    
    // Method 2: Loop (slower)
    foreach (var user in users)
    {
        _context.Users.Add(user);
    }
    await _context.SaveChangesAsync();
}
```

#### Complete Create Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    
    public UsersController(ApplicationDbContext context)
    {
        _context = context;
    }
    
    [HttpPost]
    public async Task<ActionResult<User>> CreateUser(CreateUserDto dto)
    {
        var user = new User
        {
            Name = dto.Name,
            Email = dto.Email,
            CreatedAt = DateTime.UtcNow
        };
        
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
    
    [HttpPost("bulk")]
    public async Task<ActionResult> CreateMultipleUsers(List<CreateUserDto> dtos)
    {
        var users = dtos.Select(dto => new User
        {
            Name = dto.Name,
            Email = dto.Email,
            CreatedAt = DateTime.UtcNow
        }).ToList();
        
        _context.Users.AddRange(users);
        await _context.SaveChangesAsync();
        
        return Ok(new { created = users.Count });
    }
}
```

---

### Read (R)

#### Find by ID

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<User>> GetUser(int id)
{
    // Method 1: Find (searches by primary key)
    var user = await _context.Users.FindAsync(id);
    
    if (user == null)
        return NotFound();
    
    return user;
}
```

#### Query with LINQ

```csharp
[HttpGet]
public async Task<ActionResult<List<User>>> GetUsers()
{
    // Simple query
    var users = await _context.Users.ToListAsync();
    
    // With filtering
    var activeUsers = await _context.Users
        .Where(u => u.IsActive)
        .ToListAsync();
    
    // With ordering
    var sortedUsers = await _context.Users
        .OrderBy(u => u.Name)
        .ToListAsync();
    
    // With pagination
    var page = await _context.Users
        .Skip(20)
        .Take(10)
        .ToListAsync();
    
    return users;
}
```

#### Eager Loading (Include)

```csharp
[HttpGet("{id}/with-orders")]
public async Task<ActionResult<User>> GetUserWithOrders(int id)
{
    // Include related data
    var user = await _context.Users
        .Include(u => u.Orders)  // Load orders
        .FirstOrDefaultAsync(u => u.Id == id);
    
    if (user == null)
        return NotFound();
    
    return user;
}

// Multiple levels
public async Task<User> GetUserWithOrdersAndItems(int id)
{
    return await _context.Users
        .Include(u => u.Orders)
            .ThenInclude(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
        .FirstOrDefaultAsync(u => u.Id == id);
}
```

#### Lazy Loading ✨ EF Core 2.1+

```csharp
// Step 1: Install package
// dotnet add package Microsoft.EntityFrameworkCore.Proxies

// Step 2: Enable in DbContext configuration
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString)
           .UseLazyLoadingProxies());  // Enable lazy loading

// Step 3: Mark navigation properties as virtual
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    public virtual List<Order> Orders { get; set; }  // virtual = lazy loaded
}

// Step 4: Use normally (loads automatically when accessed)
var user = await _context.Users.FindAsync(1);
var orderCount = user.Orders.Count;  // Automatically queries Orders table
```

**⚠️ Lazy Loading Warning:** Can cause N+1 query problems. Use with caution.

#### Complete Read Examples

```csharp
public class UsersController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    
    public UsersController(ApplicationDbContext context)
    {
        _context = context;
    }
    
    // Get all users
    [HttpGet]
    public async Task<ActionResult<List<User>>> GetUsers(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10)
    {
        var users = await _context.Users
            .OrderBy(u => u.Name)
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
        
        return users;
    }
    
    // Get user by ID
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        var user = await _context.Users.FindAsync(id);
        
        if (user == null)
            return NotFound();
        
        return user;
    }
    
    // Get user with orders
    [HttpGet("{id}/orders")]
    public async Task<ActionResult<User>> GetUserWithOrders(int id)
    {
        var user = await _context.Users
            .Include(u => u.Orders)
            .FirstOrDefaultAsync(u => u.Id == id);
        
        if (user == null)
            return NotFound();
        
        return user;
    }
    
    // Search users
    [HttpGet("search")]
    public async Task<ActionResult<List<User>>> SearchUsers([FromQuery] string query)
    {
        var users = await _context.Users
            .Where(u => u.Name.Contains(query) || u.Email.Contains(query))
            .ToListAsync();
        
        return users;
    }
    
    // Get single user with specific condition
    [HttpGet("by-email/{email}")]
    public async Task<ActionResult<User>> GetUserByEmail(string email)
    {
        var user = await _context.Users
            .FirstOrDefaultAsync(u => u.Email == email);
        
        if (user == null)
            return NotFound();
        
        return user;
    }
}
```

---

### Update (U)

#### Modify Existing Entity

```csharp
[HttpPut("{id}")]
public async Task<IActionResult> UpdateUser(int id, UpdateUserDto dto)
{
    // Method 1: Find, modify, save (recommended)
    var user = await _context.Users.FindAsync(id);
    
    if (user == null)
        return NotFound();
    
    user.Name = dto.Name;
    user.Email = dto.Email;
    
    await _context.SaveChangesAsync();
    
    return NoContent();
}
```

#### Attach and Modify (Disconnected Scenario)

```csharp
[HttpPut("{id}/attach")]
public async Task<IActionResult> UpdateUserAttach(int id, User user)
{
    // Method 2: Attach (when entity isn't tracked)
    if (id != user.Id)
        return BadRequest();
    
    _context.Users.Attach(user);
    _context.Entry(user).State = EntityState.Modified;
    
    await _context.SaveChangesAsync();
    
    return NoContent();
}
```

#### Update Specific Properties

```csharp
public async Task UpdateUserEmail(int id, string newEmail)
{
    var user = await _context.Users.FindAsync(id);
    
    if (user != null)
    {
        user.Email = newEmail;
        await _context.SaveChangesAsync();
    }
}

// Or update only specific property (no tracking)
public async Task UpdateUserEmailNoTracking(int id, string newEmail)
{
    var user = new User { Id = id };
    _context.Users.Attach(user);
    _context.Entry(user).Property(u => u.Email).IsModified = true;
    user.Email = newEmail;
    
    await _context.SaveChangesAsync();
}
```

#### Complete Update Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    
    public UsersController(ApplicationDbContext context)
    {
        _context = context;
    }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateUser(int id, UpdateUserDto dto)
    {
        var user = await _context.Users.FindAsync(id);
        
        if (user == null)
            return NotFound();
        
        // Update properties
        user.Name = dto.Name;
        user.Email = dto.Email;
        user.UpdatedAt = DateTime.UtcNow;
        
        try
        {
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException)
        {
            if (!await UserExists(id))
                return NotFound();
            else
                throw;
        }
        
        return NoContent();
    }
    
    [HttpPatch("{id}/email")]
    public async Task<IActionResult> UpdateEmail(int id, [FromBody] string email)
    {
        var user = await _context.Users.FindAsync(id);
        
        if (user == null)
            return NotFound();
        
        user.Email = email;
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
    
    private async Task<bool> UserExists(int id)
    {
        return await _context.Users.AnyAsync(e => e.Id == id);
    }
}
```

---

### Delete (D)

#### Remove Entity

```csharp
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteUser(int id)
{
    // Method 1: Find then remove (recommended)
    var user = await _context.Users.FindAsync(id);
    
    if (user == null)
        return NotFound();
    
    _context.Users.Remove(user);
    await _context.SaveChangesAsync();
    
    return NoContent();
}
```

#### Remove Without Loading

```csharp
public async Task<IActionResult> DeleteUserWithoutLoading(int id)
{
    // Method 2: Create entity and remove (faster, no SELECT query)
    var user = new User { Id = id };
    _context.Users.Attach(user);
    _context.Users.Remove(user);
    
    try
    {
        await _context.SaveChangesAsync();
        return NoContent();
    }
    catch (DbUpdateConcurrencyException)
    {
        return NotFound();
    }
}
```

#### Soft Delete

```csharp
// Add IsDeleted property to model
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public bool IsDeleted { get; set; }
    public DateTime? DeletedAt { get; set; }
}

// Soft delete implementation
[HttpDelete("{id}/soft")]
public async Task<IActionResult> SoftDeleteUser(int id)
{
    var user = await _context.Users.FindAsync(id);
    
    if (user == null)
        return NotFound();
    
    user.IsDeleted = true;
    user.DeletedAt = DateTime.UtcNow;
    
    await _context.SaveChangesAsync();
    
    return NoContent();
}

// Configure global query filter to exclude deleted
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>()
        .HasQueryFilter(u => !u.IsDeleted);
}
```

#### Complete Delete Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    
    public UsersController(ApplicationDbContext context)
    {
        _context = context;
    }
    
    // Hard delete
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteUser(int id)
    {
        var user = await _context.Users.FindAsync(id);
        
        if (user == null)
            return NotFound();
        
        _context.Users.Remove(user);
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
    
    // Soft delete
    [HttpDelete("{id}/soft")]
    public async Task<IActionResult> SoftDeleteUser(int id)
    {
        var user = await _context.Users.FindAsync(id);
        
        if (user == null || user.IsDeleted)
            return NotFound();
        
        user.IsDeleted = true;
        user.DeletedAt = DateTime.UtcNow;
        
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
    
    // Restore soft-deleted user
    [HttpPost("{id}/restore")]
    public async Task<IActionResult> RestoreUser(int id)
    {
        var user = await _context.Users
            .IgnoreQueryFilters()  // Include soft-deleted
            .FirstOrDefaultAsync(u => u.Id == id);
        
        if (user == null)
            return NotFound();
        
        user.IsDeleted = false;
        user.DeletedAt = null;
        
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
    
    // Permanent delete
    [HttpDelete("{id}/permanent")]
    public async Task<IActionResult> PermanentDeleteUser(int id)
    {
        var user = await _context.Users
            .IgnoreQueryFilters()
            .FirstOrDefaultAsync(u => u.Id == id);
        
        if (user == null)
            return NotFound();
        
        _context.Users.Remove(user);
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
}
```

---

## 6. Migrations

**What are Migrations?**

Migrations are version control for your database schema. They track changes to your models and update the database to match.

**Think of it like:** Git for your database. Each migration is like a commit that describes what changed.

### Migration Workflow

```
1. Change models (add property, new entity)
   ↓
2. Create migration (dotnet ef migrations add)
   ↓
3. Review migration code
   ↓
4. Apply migration (dotnet ef database update)
   ↓
5. Database updated ✅
```

### 3 Methods to Create Migrations

#### Method 1: .NET CLI (Recommended) 🎯

```bash
# Create migration
dotnet ef migrations add AddPhoneToUser

# Update database
dotnet ef database update

# List migrations
dotnet ef migrations list

# Remove last migration (not applied)
dotnet ef migrations remove

# Rollback to specific migration
dotnet ef database update PreviousMigration

# Generate SQL script (don't apply)
dotnet ef migrations script
```

---

#### Method 2: Package Manager Console (Visual Studio)

```powershell
# Create migration
Add-Migration AddPhoneToUser

# Update database
Update-Database

# List migrations
Get-Migration

# Remove last migration
Remove-Migration

# Rollback
Update-Database -Migration PreviousMigration

# Generate SQL
Script-Migration
```

---

#### Method 3: Code-Based Migration (Advanced)

```csharp
// Manually create migration class
public partial class AddPhoneToUser : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "PhoneNumber",
            table: "Users",
            type: "nvarchar(20)",
            maxLength: 20,
            nullable: true);
    }
    
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "PhoneNumber",
            table: "Users");
    }
}
```

---

### Creating Migrations Step-by-Step

**Step 1: Modify Your Model**

```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string PhoneNumber { get; set; }  // ✨ New property
}
```

**Step 2: Create Migration**

```bash
dotnet ef migrations add AddPhoneToUser
```

This creates:
- `Migrations/20241220_AddPhoneToUser.cs` - Migration code
- `Migrations/ApplicationDbContextModelSnapshot.cs` - Current model state

**Step 3: Review Generated Code**

```csharp
public partial class AddPhoneToUser : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "PhoneNumber",
            table: "Users",
            type: "nvarchar(max)",
            nullable: true);
    }
    
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "PhoneNumber",
            table: "Users");
    }
}
```

**Step 4: Apply Migration**

```bash
dotnet ef database update
```

---

### Applying Migrations

#### Development Environment

```bash
# Apply all pending migrations
dotnet ef database update

# Apply to specific migration
dotnet ef database update InitialCreate
```

#### Production Environment (Recommended)

```bash
# Generate SQL script
dotnet ef migrations script --output migration.sql

# Review SQL
# Apply manually or through deployment pipeline
```

#### Auto-Apply on Startup (Development Only)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// ⚠️ Development only!
if (app.Environment.IsDevelopment())
{
    using (var scope = app.Services.CreateScope())
    {
        var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
        db.Database.Migrate();  // Auto-apply migrations
    }
}

app.Run();
```

---

### Rolling Back Migrations

```bash
# Remove last migration (if not applied)
dotnet ef migrations remove

# Rollback database to previous migration
dotnet ef database update PreviousMigrationName

# Rollback to initial state
dotnet ef database update 0
```

---

### Seed Data

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    
    // Method 1: HasData (for small datasets)
    modelBuilder.Entity<Product>().HasData(
        new Product { Id = 1, Name = "Laptop", Price = 999.99m },
        new Product { Id = 2, Name = "Mouse", Price = 29.99m },
        new Product { Id = 3, Name = "Keyboard", Price = 79.99m }
    );
    
    // Will generate INSERT statements in migrations
}

// Method 2: Custom seeding (for large datasets)
public static class DbInitializer
{
    public static void Initialize(ApplicationDbContext context)
    {
        context.Database.Migrate();  // Apply migrations
        
        if (context.Products.Any())
            return;  // Already seeded
        
        var products = new List<Product>
        {
            new Product { Name = "Laptop", Price = 999.99m },
            new Product { Name = "Mouse", Price = 29.99m }
        };
        
        context.Products.AddRange(products);
        context.SaveChanges();
    }
}

// Use in Program.cs
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    DbInitializer.Initialize(context);
}
```

---

### Production Migrations Best Practices

```csharp
// appsettings.Production.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=prod-server;Database=ProdDb;..."
  }
}

// Generate script for production
```

```bash
# Generate SQL script for all migrations
dotnet ef migrations script --output deploy.sql

# Generate script from specific migration to latest
dotnet ef migrations script FromMigration --output update.sql

# Generate idempotent script (can run multiple times)
dotnet ef migrations script --idempotent --output deploy.sql
```

**Deployment Process:**
1. Review SQL script
2. Test on staging environment
3. Apply during maintenance window
4. Never auto-migrate in production!

---

## 7. Relationships

### One-to-Many Relationship

**Example:** One User has many Orders

```csharp
// Principal (One)
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Collection navigation property
    public List<Order> Orders { get; set; } = new();
}

// Dependent (Many)
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    
    // Foreign key
    public int UserId { get; set; }
    
    // Reference navigation property
    public User User { get; set; }
}
```

**3 Ways to Configure:**

#### Method 1: By Convention (Simplest)

EF Core automatically detects:
- Navigation properties (Orders, User)
- Foreign key (UserId)
- Creates relationship

```csharp
// No configuration needed!
// EF Core figures it out
```

---

#### Method 2: Data Annotations

```csharp
public class Order
{
    public int Id { get; set; }
    
    [ForeignKey("User")]
    public int UserId { get; set; }
    
    public User User { get; set; }
}
```

---

#### Method 3: Fluent API (Most Control) 🎯

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>()
        .HasMany(u => u.Orders)
        .WithOne(o => o.User)
        .HasForeignKey(o => o.UserId)
        .OnDelete(DeleteBehavior.Cascade);  // Delete orders when user deleted
}
```

**Complete One-to-Many Example:**

```csharp
// Models
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    public List<Product> Products { get; set; } = new();
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int CategoryId { get; set; }
    public Category Category { get; set; }
}

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Category>()
        .HasMany(c => c.Products)
        .WithOne(p => p.Category)
        .HasForeignKey(p => p.CategoryId)
        .OnDelete(DeleteBehavior.Restrict);  // Don't delete category if it has products
}

// Usage
var category = new Category { Name = "Electronics" };
category.Products.Add(new Product { Name = "Laptop" });
category.Products.Add(new Product { Name = "Phone" });

_context.Categories.Add(category);
await _context.SaveChangesAsync();  // Saves category and products
```

---

### One-to-One Relationship

**Example:** User has one Profile

```csharp
// Principal
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Reference navigation
    public UserProfile Profile { get; set; }
}

// Dependent
public class UserProfile
{
    public int Id { get; set; }  // Also the foreign key
    public string Bio { get; set; }
    public string Website { get; set; }
    
    // Reference navigation
    public User User { get; set; }
}
```

**Configuration (Fluent API):**

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>()
        .HasOne(u => u.Profile)
        .WithOne(p => p.User)
        .HasForeignKey<UserProfile>(p => p.Id);  // UserProfile.Id is FK
}
```

**Complete One-to-One Example:**

```csharp
// Models
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Address Address { get; set; }
}

public class Address
{
    public int Id { get; set; }
    public string Street { get; set; }
    public string City { get; set; }
    public string ZipCode { get; set; }
    public Employee Employee { get; set; }
}

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>()
        .HasOne(e => e.Address)
        .WithOne(a => a.Employee)
        .HasForeignKey<Address>(a => a.Id)
        .OnDelete(DeleteBehavior.Cascade);
}

// Usage
var employee = new Employee
{
    Name = "John Doe",
    Address = new Address
    {
        Street = "123 Main St",
        City = "New York",
        ZipCode = "10001"
    }
};

_context.Employees.Add(employee);
await _context.SaveChangesAsync();
```

---

### Many-to-Many Relationship ✨ EF Core 5.0+

**Example:** Students enroll in many Courses, Courses have many Students

#### Method 1: Skip Navigation (EF Core 5.0+) 🎯 RECOMMENDED

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Collection navigation
    public List<Course> Courses { get; set; } = new();
}

public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }
    
    // Collection navigation
    public List<Student> Students { get; set; } = new();
}

// Configuration (optional - EF Core auto-creates join table)
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .HasMany(s => s.Courses)
        .WithMany(c => c.Students);
        // Auto-creates CourseStudent join table
}

// Usage
var student = new Student { Name = "John" };
student.Courses.Add(new Course { Title = "Math" });
student.Courses.Add(new Course { Title = "Science" });

_context.Students.Add(student);
await _context.SaveChangesAsync();

// Querying
var studentWithCourses = await _context.Students
    .Include(s => s.Courses)
    .FirstOrDefaultAsync(s => s.Id == 1);
```

---

#### Method 2: Explicit Join Entity (More Control)

```csharp
// Entities
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public List<Enrollment> Enrollments { get; set; } = new();
}

public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }
    public List<Enrollment> Enrollments { get; set; } = new();
}

// Join entity (with extra properties)
public class Enrollment
{
    public int StudentId { get; set; }
    public Student Student { get; set; }
    
    public int CourseId { get; set; }
    public Course Course { get; set; }
    
    // Extra properties
    public DateTime EnrolledDate { get; set; }
    public int Grade { get; set; }
}

// Configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Enrollment>()
        .HasKey(e => new { e.StudentId, e.CourseId });  // Composite key
    
    modelBuilder.Entity<Enrollment>()
        .HasOne(e => e.Student)
        .WithMany(s => s.Enrollments)
        .HasForeignKey(e => e.StudentId);
    
    modelBuilder.Entity<Enrollment>()
        .HasOne(e => e.Course)
        .WithMany(c => c.Enrollments)
        .HasForeignKey(e => e.CourseId);
}

// Usage
var enrollment = new Enrollment
{
    StudentId = 1,
    CourseId = 2,
    EnrolledDate = DateTime.UtcNow,
    Grade = 95
};

_context.Set<Enrollment>().Add(enrollment);
await _context.SaveChangesAsync();

// Querying
var student = await _context.Students
    .Include(s => s.Enrollments)
        .ThenInclude(e => e.Course)
    .FirstOrDefaultAsync(s => s.Id == 1);
```

---

### Relationship Delete Behaviors

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>()
        .HasMany(u => u.Orders)
        .WithOne(o => o.User)
        .OnDelete(DeleteBehavior.Cascade);  // Options below
}
```

| Behavior | What Happens |
|----------|-------------|
| `Cascade` | Delete dependent (Orders) when principal (User) deleted |
| `Restrict` | Prevent delete if dependent exists |
| `SetNull` | Set foreign key to NULL |
| `NoAction` | Do nothing (database constraint handles it) |

---

## 8. Querying Patterns

### AsNoTracking (Read-Only Queries)

```csharp
// Tracked (default) - EF Core monitors changes
var users = await _context.Users.ToListAsync();
users[0].Name = "Changed";  // This change is tracked
await _context.SaveChangesAsync();  // Updates database

// No tracking - Better performance for read-only
var users = await _context.Users
    .AsNoTracking()  // ✅ 30-40% faster
    .ToListAsync();
users[0].Name = "Changed";  // This change is NOT tracked
await _context.SaveChangesAsync();  // Does nothing
```

**When to use:**
- ✅ Read-only queries (GET endpoints)
- ✅ Reports, dashboards
- ✅ Any time you won't modify entities
- ❌ If you need to update data

---

### Projection (Select)

```csharp
// ❌ Bad - Loads all columns
var users = await _context.Users.ToListAsync();

// ✅ Good - Loads only needed columns
var userDtos = await _context.Users
    .Select(u => new UserDto
    {
        Id = u.Id,
        Name = u.Name,
        Email = u.Email
    })
    .ToListAsync();

// Complex projection
var userSummary = await _context.Users
    .Select(u => new
    {
        u.Id,
        u.Name,
        OrderCount = u.Orders.Count,
        TotalSpent = u.Orders.Sum(o => o.TotalAmount),
        LastOrderDate = u.Orders.Max(o => o.OrderDate)
    })
    .ToListAsync();
```

---

### Filtering (Where)

```csharp
// Simple filter
var activeUsers = await _context.Users
    .Where(u => u.IsActive)
    .ToListAsync();

// Multiple conditions
var results = await _context.Users
    .Where(u => u.IsActive && u.CreatedAt > DateTime.Now.AddDays(-30))
    .ToListAsync();

// String operations
var searchResults = await _context.Users
    .Where(u => u.Name.Contains("John") || u.Email.StartsWith("john"))
    .ToListAsync();

// In list
var ids = new[] { 1, 2, 3, 4, 5 };
var users = await _context.Users
    .Where(u => ids.Contains(u.Id))
    .ToListAsync();
```

---

### Sorting (OrderBy)

```csharp
// Ascending
var users = await _context.Users
    .OrderBy(u => u.Name)
    .ToListAsync();

// Descending
var users = await _context.Users
    .OrderByDescending(u => u.CreatedAt)
    .ToListAsync();

// Multiple sorting
var users = await _context.Users
    .OrderBy(u => u.Name)
    .ThenByDescending(u => u.CreatedAt)
    .ToListAsync();
```

---

### Pagination (Skip/Take)

```csharp
[HttpGet]
public async Task<ActionResult<PagedResult<User>>> GetUsers(
    [FromQuery] int page = 1,
    [FromQuery] int pageSize = 10)
{
    var query = _context.Users.AsQueryable();
    
    var totalCount = await query.CountAsync();
    
    var users = await query
        .OrderBy(u => u.Name)
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();
    
    return new PagedResult<User>
    {
        Data = users,
        Page = page,
        PageSize = pageSize,
        TotalCount = totalCount,
        TotalPages = (int)Math.Ceiling(totalCount / (double)pageSize)
    };
}

public class PagedResult<T>
{
    public List<T> Data { get; set; }
    public int Page { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages { get; set; }
}
```

---

### GroupBy

```csharp
// Group users by creation month
var usersByMonth = await _context.Users
    .GroupBy(u => new { u.CreatedAt.Year, u.CreatedAt.Month })
    .Select(g => new
    {
        Year = g.Key.Year,
        Month = g.Key.Month,
        Count = g.Count(),
        Users = g.ToList()
    })
    .ToListAsync();

// Order statistics by product
var productStats = await _context.OrderItems
    .GroupBy(oi => oi.ProductId)
    .Select(g => new
    {
        ProductId = g.Key,
        TotalQuantity = g.Sum(oi => oi.Quantity),
        TotalRevenue = g.Sum(oi => oi.Quantity * oi.UnitPrice),
        OrderCount = g.Count()
    })
    .ToListAsync();
```

---

### Complete Querying Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    
    public ProductsController(ApplicationDbContext context)
    {
        _context = context;
    }
    
    [HttpGet]
    public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
        [FromQuery] string search = null,
        [FromQuery] decimal? minPrice = null,
        [FromQuery] decimal? maxPrice = null,
        [FromQuery] string sortBy = "name",
        [FromQuery] bool descending = false,
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10)
    {
        var query = _context.Products.AsQueryable();
        
        // Filtering
        if (!string.IsNullOrEmpty(search))
        {
            query = query.Where(p => 
                p.Name.Contains(search) || 
                p.Description.Contains(search));
        }
        
        if (minPrice.HasValue)
        {
            query = query.Where(p => p.Price >= minPrice.Value);
        }
        
        if (maxPrice.HasValue)
        {
            query = query.Where(p => p.Price <= maxPrice.Value);
        }
        
        // Sorting
        query = sortBy.ToLower() switch
        {
            "price" => descending 
                ? query.OrderByDescending(p => p.Price)
                : query.OrderBy(p => p.Price),
            "name" => descending
                ? query.OrderByDescending(p => p.Name)
                : query.OrderBy(p => p.Name),
            _ => query.OrderBy(p => p.Name)
        };
        
        // Count before pagination
        var totalCount = await query.CountAsync();
        
        // Projection and pagination
        var products = await query
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .Select(p => new ProductDto
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price,
                StockQuantity = p.StockQuantity
            })
            .AsNoTracking()
            .ToListAsync();
        
        return new PagedResult<ProductDto>
        {
            Data = products,
            Page = page,
            PageSize = pageSize,
            TotalCount = totalCount,
            TotalPages = (int)Math.Ceiling(totalCount / (double)pageSize)
        };
    }
}
```

---

## 9. Troubleshooting Common Issues

### Issue 1: No Database Provider Configured

**Problem:**
```
InvalidOperationException: No database provider has been configured for this DbContext
```

**Solution:**
```csharp
// ❌ Missing
public class ApplicationDbContext : DbContext
{
}

// ✅ Add configuration in Program.cs
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

---

### Issue 2: Entity Not Tracked

**Problem:**
```csharp
var user = new User { Id = 1, Name = "John" };
_context.SaveChangesAsync();  // ❌ Nothing happens - entity not tracked
```

**Solution:**
```csharp
// ✅ Attach entity
_context.Users.Attach(user);
_context.Entry(user).State = EntityState.Modified;
await _context.SaveChangesAsync();

// ✅ Or query first
var user = await _context.Users.FindAsync(1);
user.Name = "John";
await _context.SaveChangesAsync();
```

---

### Issue 3: N+1 Query Problem

**Problem:**
```csharp
// ❌ Loads users, then 1 query per user for orders (1 + N queries)
var users = await _context.Users.ToListAsync();
foreach (var user in users)
{
    var orderCount = user.Orders.Count;  // Separate query!
}
```

**Solution:**
```csharp
// ✅ Single query with Include
var users = await _context.Users
    .Include(u => u.Orders)
    .ToListAsync();
```

---

### Issue 4: Lazy Loading Not Working

**Problem:**
```csharp
var user = await _context.Users.FindAsync(1);
var orders = user.Orders;  // ❌ null or empty
```

**Solution:**
```csharp
// Option 1: Explicit loading
var user = await _context.Users.FindAsync(1);
await _context.Entry(user).Collection(u => u.Orders).LoadAsync();

// Option 2: Eager loading
var user = await _context.Users
    .Include(u => u.Orders)
    .FirstOrDefaultAsync(u => u.Id == 1);

// Option 3: Enable lazy loading
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString)
           .UseLazyLoadingProxies());
```

---

### Issue 5: Migrations Not Applying

**Problem:**
```bash
dotnet ef database update
# No error but database unchanged
```

**Solution:**
```bash
# Check pending migrations
dotnet ef migrations list

# Drop database and recreate (dev only!)
dotnet ef database drop
dotnet ef database update

# Or apply specific migration
dotnet ef database update MigrationName
```

---

### Issue 6: Connection String Not Found

**Problem:**
```
ArgumentNullException: Connection string 'DefaultConnection' not found
```

**Solution:**
```json
// ✅ Check appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;..."
  }
}

// ✅ Check appsettings.{Environment}.json exists
// ✅ Verify ASPNETCORE_ENVIRONMENT variable
```

---

### Issue 7: Tracking Multiple Instances

**Problem:**
```
InvalidOperationException: The instance of entity type 'User' cannot be tracked because another instance with the same key value is already being tracked
```

**Solution:**
```csharp
// ❌ Problem
var user1 = await _context.Users.FindAsync(1);
var user2 = await _context.Users.FindAsync(1);  // Error!

// ✅ Solution 1: Use AsNoTracking
var user1 = await _context.Users.AsNoTracking().FirstAsync(u => u.Id == 1);
var user2 = await _context.Users.FirstAsync(u => u.Id == 1);

// ✅ Solution 2: Detach first instance
_context.Entry(user1).State = EntityState.Detached;
```

---

## 10. Best Practices

### ✅ DO

1. **Use AsNoTracking for read-only queries**
   ```csharp
   var users = await _context.Users.AsNoTracking().ToListAsync();
   ```

2. **Always use async/await**
   ```csharp
   await _context.SaveChangesAsync();
   ```

3. **Use Include for eager loading**
   ```csharp
   var users = await _context.Users.Include(u => u.Orders).ToListAsync();
   ```

4. **Project to DTOs**
   ```csharp
   var dtos = await _context.Users
       .Select(u => new UserDto { Id = u.Id, Name = u.Name })
       .ToListAsync();
   ```

5. **Use migrations for schema changes**
   ```bash
   dotnet ef migrations add AddNewColumn
   ```

6. **Configure relationships with Fluent API**
   ```csharp
   modelBuilder.Entity<Order>()
       .HasOne(o => o.User)
       .WithMany(u => u.Orders);
   ```

7. **Use scoped lifetime for DbContext**
   ```csharp
   builder.Services.AddDbContext<ApplicationDbContext>(options => ...);
   ```

---

### ❌ DON'T

1. **Don't use DbContext as singleton**
   ```csharp
   // ❌ Never do this
   builder.Services.AddSingleton<ApplicationDbContext>();
   ```

2. **Don't call SaveChanges in loops**
   ```csharp
   // ❌ Slow
   foreach (var user in users)
   {
       _context.Users.Add(user);
       await _context.SaveChangesAsync();
   }
   
   // ✅ Fast
   _context.Users.AddRange(users);
   await _context.SaveChangesAsync();
   ```

3. **Don't load entire tables**
   ```csharp
   // ❌ Bad
   var users = await _context.Users.ToListAsync();
   
   // ✅ Good - filter first
   var activeUsers = await _context.Users.Where(u => u.IsActive).ToListAsync();
   ```

4. **Don't forget using statements (if not using DI)**
   ```csharp
   // ❌ Memory leak
   var context = new ApplicationDbContext();
   
   // ✅ Dispose properly
   using var context = new ApplicationDbContext();
   ```

5. **Don't expose DbContext directly in APIs**
   ```csharp
   // ❌ Bad
   [HttpGet]
   public ApplicationDbContext GetContext() => _context;
   
   // ✅ Good - use repository or service
   public class UserService { ... }
   ```

---

### Performance Tips

1. **Use compiled queries for frequently used queries** ✨ EF Core 2.0+
   ```csharp
   private static readonly Func<ApplicationDbContext, int, Task<User>> _getUserById =
       EF.CompileAsyncQuery((ApplicationDbContext context, int id) =>
           context.Users.FirstOrDefault(u => u.Id == id));
   
   var user = await _getUserById(_context, 5);
   ```

2. **Batch operations** ✨ EF Core 7.0+
   ```csharp
   // Bulk delete (EF Core 7+)
   await _context.Users
       .Where(u => u.IsActive == false)
       .ExecuteDeleteAsync();
   
   // Bulk update (EF Core 7+)
   await _context.Users
       .Where(u => u.CreatedAt < DateTime.Now.AddYears(-1))
       .ExecuteUpdateAsync(u => u.SetProperty(x => x.IsActive, false));
   ```

3. **Use DbContext pooling**
   ```csharp
   builder.Services.AddDbContextPool<ApplicationDbContext>(options =>
       options.UseSqlServer(connectionString));
   ```

4. **Split queries for large includes** ✨ EF Core 5.0+
   ```csharp
   var users = await _context.Users
       .Include(u => u.Orders)
       .AsSplitQuery()  // Separate queries instead of JOIN
       .ToListAsync();
   ```

---

# PART 2: TECHNICAL REFERENCE

---

## 11. Important Interfaces & Classes Reference

### DbContext Class ⭐⭐⭐

**Purpose:** Represents a session with the database and provides APIs for CRUD operations

**Namespace:** `Microsoft.EntityFrameworkCore`

**Key Members:**

| Member | Type | Description |
|--------|------|-------------|
| `Database` | DatabaseFacade | Database operations (Migrate, BeginTransaction) |
| `ChangeTracker` | ChangeTracker | Tracks entity changes |
| `Model` | IModel | Database model metadata |
| `SaveChanges()` | int | Persist changes to database (returns affected rows) |
| `SaveChangesAsync()` | Task<int> | Async version |
| `Add<T>()` | EntityEntry<T> | Start tracking entity in Added state |
| `Update<T>()` | EntityEntry<T> | Start tracking entity in Modified state |
| `Remove<T>()` | EntityEntry<T> | Start tracking entity in Deleted state |
| `Attach<T>()` | EntityEntry<T> | Start tracking entity in Unchanged state |
| `Entry<T>()` | EntityEntry<T> | Access entity state and metadata |
| `Find<T>()` | T | Find entity by primary key |
| `FindAsync<T>()` | Task<T> | Async version |

**Important Methods:**

```csharp
// Database operations
await context.Database.MigrateAsync();  // Apply migrations
await context.Database.EnsureCreatedAsync();  // Create database if not exists
await context.Database.BeginTransactionAsync();  // Start transaction
await context.Database.ExecuteSqlRawAsync("UPDATE Users SET ...");

// Change tracking
context.ChangeTracker.Clear();  // Clear all tracked entities
context.ChangeTracker.DetectChanges();  // Detect changes manually

// Entity state
context.Entry(user).State = EntityState.Modified;
var isModified = context.Entry(user).State == EntityState.Modified;

// Save changes
var affected = await context.SaveChangesAsync();
```

**Configuration:**

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(connectionString)
        .EnableSensitiveDataLogging()  // Log parameter values (dev only)
        .EnableDetailedErrors()  // Detailed error messages
        .LogTo(Console.WriteLine);  // Log to console
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Configure entities
    modelBuilder.Entity<User>().ToTable("Users");
}
```

---

### DbSet<TEntity> Class ⭐⭐⭐

**Purpose:** Represents a collection of entities (table)

**Namespace:** `Microsoft.EntityFrameworkCore`

**Key Members:**

| Member | Return Type | Description |
|--------|-------------|-------------|
| `Add(entity)` | EntityEntry<T> | Add entity |
| `AddAsync(entity)` | Task<EntityEntry<T>> | Async add |
| `AddRange(entities)` | void | Add multiple entities |
| `Remove(entity)` | EntityEntry<T> | Remove entity |
| `RemoveRange(entities)` | void | Remove multiple |
| `Update(entity)` | EntityEntry<T> | Update entity |
| `UpdateRange(entities)` | void | Update multiple |
| `Find(key)` | T | Find by primary key |
| `FindAsync(key)` | Task<T> | Async find |

**LINQ Methods (from IQueryable<T>):**

| Method | Description |
|--------|-------------|
| `Where()` | Filter |
| `Select()` | Projection |
| `OrderBy()` | Sort ascending |
| `OrderByDescending()` | Sort descending |
| `Skip()` | Skip records (pagination) |
| `Take()` | Take records (pagination) |
| `Include()` | Eager load related data |
| `ThenInclude()` | Nested eager loading |
| `AsNoTracking()` | Disable change tracking |
| `FirstOrDefaultAsync()` | Get first or null |
| `SingleOrDefaultAsync()` | Get single or null |
| `ToListAsync()` | Execute and return list |
| `CountAsync()` | Count records |
| `AnyAsync()` | Check if any exist |

**Usage Examples:**

```csharp
// Adding
_context.Users.Add(new User { Name = "John" });
await _context.Users.AddRangeAsync(userList);

// Finding
var user = await _context.Users.FindAsync(5);

// Querying
var users = await _context.Users
    .Where(u => u.IsActive)
    .OrderBy(u => u.Name)
    .ToListAsync();

// Removing
_context.Users.Remove(user);
```

---

### ModelBuilder Class ⭐⭐

**Purpose:** Fluent API for configuring entity models

**Namespace:** `Microsoft.EntityFrameworkCore`

**Key Methods:**

| Method | Purpose |
|--------|---------|
| `Entity<T>()` | Configure entity |
| `HasKey()` | Define primary key |
| `HasOne()` | Configure one side of relationship |
| `HasMany()` | Configure many side of relationship |
| `Property()` | Configure property |
| `HasIndex()` | Create index |
| `HasData()` | Seed data |
| `HasQueryFilter()` | Global query filter |

**Complete Example:**

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>(entity =>
    {
        // Table name
        entity.ToTable("Users");
        
        // Primary key
        entity.HasKey(e => e.Id);
        
        // Properties
        entity.Property(e => e.Name)
              .IsRequired()
              .HasMaxLength(100);
        
        entity.Property(e => e.Email)
              .IsRequired()
              .HasMaxLength(200);
        
        entity.Property(e => e.CreatedAt)
              .HasDefaultValueSql("GETUTCDATE()");
        
        // Index
        entity.HasIndex(e => e.Email)
              .IsUnique()
              .HasDatabaseName("IX_Users_Email");
        
        // Relationships
        entity.HasMany(e => e.Orders)
              .WithOne(e => e.User)
              .HasForeignKey(e => e.UserId)
              .OnDelete(DeleteBehavior.Cascade);
        
        // Query filter (soft delete)
        entity.HasQueryFilter(e => !e.IsDeleted);
        
        // Seed data
        entity.HasData(
            new User { Id = 1, Name = "Admin", Email = "admin@example.com" }
        );
    });
}
```

---

### ChangeTracker Class

**Purpose:** Tracks entity changes

**Namespace:** `Microsoft.EntityFrameworkCore.ChangeTracking`

**Key Members:**

| Member | Description |
|--------|-------------|
| `Entries()` | Get all tracked entities |
| `Entries<T>()` | Get tracked entities of type T |
| `Clear()` | Stop tracking all entities |
| `HasChanges()` | Check if any changes |
| `DetectChanges()` | Manually detect changes |

**Usage:**

```csharp
// Get all modified entities
var modifiedEntities = _context.ChangeTracker
    .Entries()
    .Where(e => e.State == EntityState.Modified);

// Clear tracking
_context.ChangeTracker.Clear();

// Check for changes
if (_context.ChangeTracker.HasChanges())
{
    await _context.SaveChangesAsync();
}
```

---

### EntityEntry<TEntity> Class

**Purpose:** Access entity state and metadata

**Key Properties:**

| Property | Description |
|----------|-------------|
| `State` | EntityState (Added, Modified, Deleted, Unchanged) |
| `Entity` | The tracked entity |
| `OriginalValues` | Original property values |
| `CurrentValues` | Current property values |

**Usage:**

```csharp
var entry = _context.Entry(user);

// Change state
entry.State = EntityState.Modified;

// Check if modified
bool isModified = entry.State == EntityState.Modified;

// Get original value
var originalName = entry.OriginalValues["Name"];

// Mark property as modified
entry.Property(u => u.Email).IsModified = true;

// Reload from database
await entry.ReloadAsync();
```

---

### Entity States

| State | Description | When |
|-------|-------------|------|
| `Detached` | Not tracked | Entity created without context |
| `Unchanged` | Tracked, no changes | After query or SaveChanges |
| `Added` | Tracked, will be inserted | After Add() |
| `Modified` | Tracked, will be updated | After modifying tracked entity |
| `Deleted` | Tracked, will be deleted | After Remove() |

**State Diagram:**

```
New Entity
   ↓
[Detached] → Add() → [Added] → SaveChanges() → [Unchanged]
                                                      ↓
                                        Modify property
                                                      ↓
                                                 [Modified]
                                                      ↓
                                                Remove()
                                                      ↓
                                                 [Deleted]
```

---

## 12. Configuration Deep-Dive

### OnModelCreating Patterns

#### Pattern 1: Inline Configuration

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>(entity =>
    {
        entity.ToTable("Users");
        entity.HasKey(e => e.Id);
        entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
    });
}
```

**Pros:** Simple, all in one place
**Cons:** Gets messy with many entities

---

#### Pattern 2: Configuration Classes (IEntityTypeConfiguration) 🎯 RECOMMENDED

```csharp
// UserConfiguration.cs
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("Users");
        
        builder.HasKey(e => e.Id);
        
        builder.Property(e => e.Name)
               .IsRequired()
               .HasMaxLength(100);
        
        builder.Property(e => e.Email)
               .IsRequired()
               .HasMaxLength(200);
        
        builder.HasIndex(e => e.Email)
               .IsUnique();
        
        builder.HasMany(e => e.Orders)
               .WithOne(e => e.User)
               .HasForeignKey(e => e.UserId);
    }
}

// ApplicationDbContext.cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Apply all configurations in assembly
    modelBuilder.ApplyConfigurationsFromAssembly(typeof(ApplicationDbContext).Assembly);
    
    // Or apply specific configuration
    modelBuilder.ApplyConfiguration(new UserConfiguration());
}
```

**Pros:** Clean, organized, one file per entity
**Cons:** More files

---

#### Pattern 3: Extension Methods

```csharp
public static class ModelBuilderExtensions
{
    public static void ConfigureUserEntity(this ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>(entity =>
        {
            // Configuration
        });
    }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ConfigureUserEntity();
    modelBuilder.ConfigureProductEntity();
}
```

---

### Fluent API Complete Reference

#### Table Configuration

```csharp
entity.ToTable("Users");  // Table name
entity.ToTable("Users", "dbo");  // Table name and schema
```

#### Primary Key

```csharp
entity.HasKey(e => e.Id);  // Single key
entity.HasKey(e => new { e.OrderId, e.ProductId });  // Composite key
```

#### Properties

```csharp
builder.Property(e => e.Name)
       .IsRequired()  // NOT NULL
       .HasMaxLength(100)  // VARCHAR(100)
       .HasColumnName("UserName")  // Column name
       .HasColumnType("nvarchar(100)")  // Explicit type
       .HasDefaultValue("Unknown")  // Default value
       .HasDefaultValueSql("GETUTCDATE()")  // SQL default
       .IsUnicode(true)  // NVARCHAR vs VARCHAR
       .HasPrecision(18, 2)  // DECIMAL(18,2)
       .HasComment("User's full name");  // Column comment
```

#### Indexes

```csharp
builder.HasIndex(e => e.Email)
       .IsUnique()
       .HasDatabaseName("IX_Users_Email")
       .HasFilter("[IsDeleted] = 0");  // Filtered index

// Composite index
builder.HasIndex(e => new { e.LastName, e.FirstName });
```

#### Relationships

```csharp
// One-to-Many
builder.HasMany(u => u.Orders)
       .WithOne(o => o.User)
       .HasForeignKey(o => o.UserId)
       .OnDelete(DeleteBehavior.Cascade)
       .HasConstraintName("FK_Orders_Users");

// One-to-One
builder.HasOne(u => u.Profile)
       .WithOne(p => p.User)
       .HasForeignKey<UserProfile>(p => p.UserId);

// Many-to-Many (EF Core 5+)
builder.HasMany(s => s.Courses)
       .WithMany(c => c.Students)
       .UsingEntity(j => j.ToTable("Enrollments"));
```

---

### Global Query Filters

```csharp
// Soft delete filter
modelBuilder.Entity<User>()
    .HasQueryFilter(u => !u.IsDeleted);

// Tenant filter
modelBuilder.Entity<Order>()
    .HasQueryFilter(o => o.TenantId == _tenantId);

// Bypass filter when needed
var allUsers = await _context.Users
    .IgnoreQueryFilters()
    .ToListAsync();
```

---

### Value Conversions ✨ EF Core 2.1+

```csharp
// Enum to string
builder.Property(e => e.Status)
       .HasConversion<string>();

// Custom conversion
builder.Property(e => e.Price)
       .HasConversion(
           v => v.ToString(),  // To database
           v => decimal.Parse(v));  // From database

// JSON conversion (EF Core 7+)
builder.Property(e => e.Address)
       .HasConversion(
           v => JsonSerializer.Serialize(v, null),
           v => JsonSerializer.Deserialize<Address>(v, null))
       .HasColumnType("nvarchar(max)");
```

---

### Owned Entities ✨ EF Core 2.0+

```csharp
public class Order
{
    public int Id { get; set; }
    public Address ShippingAddress { get; set; }  // Owned entity
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string ZipCode { get; set; }
}

// Configuration
modelBuilder.Entity<Order>()
    .OwnsOne(o => o.ShippingAddress, address =>
    {
        address.Property(a => a.Street).HasColumnName("ShippingStreet");
        address.Property(a => a.City).HasColumnName("ShippingCity");
    });

// Stores in same table:
// Orders table: Id, ShippingStreet, ShippingCity, ShippingZipCode
```

---

### Table Splitting

```csharp
// Split entity across multiple entities in same table
modelBuilder.Entity<User>()
    .ToTable("Users")
    .HasOne(u => u.Profile)
    .WithOne()
    .HasForeignKey<UserProfile>(p => p.UserId);

modelBuilder.Entity<UserProfile>()
    .ToTable("Users");  // Same table!
```

---

## 13. Advanced Topics

### Raw SQL Queries

```csharp
// Execute raw query
var users = await _context.Users
    .FromSqlRaw("SELECT * FROM Users WHERE IsActive = 1")
    .ToListAsync();

// With parameters
var email = "john@example.com";
var user = await _context.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Email = {email}")
    .FirstOrDefaultAsync();

// Execute non-query
await _context.Database.ExecuteSqlRawAsync(
    "UPDATE Users SET IsActive = 0 WHERE LastLogin < @date",
    new SqlParameter("date", DateTime.Now.AddYears(-1)));
```

---

### Stored Procedures

```csharp
// Call stored procedure
var users = await _context.Users
    .FromSqlRaw("EXEC GetActiveUsers")
    .ToListAsync();

// With parameters
var result = await _context.Database
    .ExecuteSqlRawAsync("EXEC UpdateUserStatus @userId, @status",
        new SqlParameter("userId", 5),
        new SqlParameter("status", true));
```

---

### Concurrency Handling

```csharp
// Add concurrency token
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    [Timestamp]  // Concurrency token
    public byte[] RowVersion { get; set; }
}

// Or Fluent API
modelBuilder.Entity<User>()
    .Property(u => u.RowVersion)
    .IsRowVersion();

// Handle concurrency exception
try
{
    await _context.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{
    foreach (var entry in ex.Entries)
    {
        var databaseValues = await entry.GetDatabaseValuesAsync();
        
        if (databaseValues == null)
        {
            // Entity was deleted
        }
        else
        {
            // Update conflict - handle appropriately
            entry.OriginalValues.SetValues(databaseValues);
        }
    }
    
    await _context.SaveChangesAsync();
}
```

---

### Shadow Properties

```csharp
// Configure shadow property (not in C# class)
modelBuilder.Entity<User>()
    .Property<DateTime>("LastModified");

// Set value
_context.Entry(user).Property("LastModified").CurrentValue = DateTime.UtcNow;

// Query
var recentUsers = await _context.Users
    .Where(u => EF.Property<DateTime>(u, "LastModified") > DateTime.Now.AddDays(-7))
    .ToListAsync();
```

---

### Interceptors ✨ EF Core 3.0+

```csharp
public class AuditInterceptor : SaveChangesInterceptor
{
    public override InterceptionResult<int> SavingChanges(
        DbContextEventData eventData,
        InterceptionResult<int> result)
    {
        UpdateAuditFields(eventData.Context);
        return base.SavingChanges(eventData, result);
    }
    
    private void UpdateAuditFields(DbContext context)
    {
        foreach (var entry in context.ChangeTracker.Entries())
        {
            if (entry.Entity is IAuditable auditable)
            {
                if (entry.State == EntityState.Added)
                {
                    auditable.CreatedAt = DateTime.UtcNow;
                }
                else if (entry.State == EntityState.Modified)
                {
                    auditable.UpdatedAt = DateTime.UtcNow;
                }
            }
        }
    }
}

// Register
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString)
           .AddInterceptors(new AuditInterceptor()));
```

---

### DbContext Pooling ✨ EF Core 2.0+

```csharp
// Instead of AddDbContext
builder.Services.AddDbContextPool<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString),
    poolSize: 128);  // Default is 1024
```

**Benefits:**
- Reuses DbContext instances
- Reduces overhead of creating new contexts
- Improves performance in high-traffic apps

**Limitations:**
- DbContext state must be reset between requests
- Can't use constructor DI for dependencies

---

## 14. Performance Optimization

### Query Performance Tips

```csharp
// ✅ Use AsNoTracking for read-only
var users = await _context.Users.AsNoTracking().ToListAsync();

// ✅ Project only needed columns
var dtos = await _context.Users
    .Select(u => new { u.Id, u.Name })
    .ToListAsync();

// ✅ Use compiled queries
private static readonly Func<ApplicationDbContext, int, Task<User>> _getUserById =
    EF.CompileAsyncQuery((ApplicationDbContext context, int id) =>
        context.Users.FirstOrDefault(u => u.Id == id));

// ✅ Split large includes
var users = await _context.Users
    .Include(u => u.Orders)
    .AsSplitQuery()
    .ToListAsync();

// ✅ Use Where before Include
var users = await _context.Users
    .Where(u => u.IsActive)  // Filter first
    .Include(u => u.Orders)
    .ToListAsync();
```

---

### Batch Operations ✨ EF Core 7.0+

```csharp
// Bulk delete
await _context.Users
    .Where(u => !u.IsActive)
    .ExecuteDeleteAsync();

// Bulk update
await _context.Users
    .Where(u => u.CreatedAt < DateTime.Now.AddYears(-1))
    .ExecuteUpdateAsync(u => u.SetProperty(x => x.IsActive, false));
```

---

### Connection Resiliency

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString, sqlOptions =>
    {
        sqlOptions.EnableRetryOnFailure(
            maxRetryCount: 5,
            maxRetryDelay: TimeSpan.FromSeconds(30),
            errorNumbersToAdd: null);
    }));
```

---

### Monitoring and Logging

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString)
           .LogTo(Console.WriteLine, LogLevel.Information)  // Log queries
           .EnableSensitiveDataLogging()  // Log parameter values (dev only)
           .EnableDetailedErrors());  // Detailed error messages
```

---

## Summary

This guide covered:
- ✅ EF Core setup (SQL Server, SQLite, In-Memory)
- ✅ DbContext configuration (3 methods)
- ✅ Model definition and data annotations
- ✅ Complete CRUD operations
- ✅ Migrations workflow
- ✅ Relationships (one-to-many, one-to-one, many-to-many)
- ✅ Advanced querying patterns
- ✅ Troubleshooting common issues
- ✅ Best practices
- ✅ Performance optimization

**Next Steps:**
- Practice with a real project
- Explore EF Core advanced features
- Learn about repository pattern
- Study performance optimization

---

**Version:** ASP.NET Core 8.0  
**Created:** December 2024  
**Target:** Developers learning Entity Framework Core
