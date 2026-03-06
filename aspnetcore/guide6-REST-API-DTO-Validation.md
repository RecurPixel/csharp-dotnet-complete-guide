# ASP.NET Core REST APIs, DTOs & Validation - Complete Guide
## Practical Guide + Technical Reference

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. REST API Fundamentals
2. Building REST APIs (3 Approaches)
3. DTOs (Data Transfer Objects)
4. Model Validation (3 Methods)
5. Content Negotiation
6. Error Handling & Problem Details
7. API Versioning
8. Swagger/OpenAPI Documentation
9. Troubleshooting Common Issues
10. Best Practices

### Part 2: Technical Reference (Deep Dive)
11. Important Interfaces & Classes Reference
12. Configuration Deep-Dive
13. Advanced Topics

---

# PART 1: PRACTICAL GUIDE

---

## 1. REST API Fundamentals

**Simple Definition:** REST (Representational State Transfer) is an architectural style for building web APIs using HTTP methods.

**Think of it like:** A restaurant menu where:
- URLs are dishes (resources)
- HTTP methods are actions (order, modify, cancel)
- Status codes are feedback (success, error, not found)

### HTTP Methods (Verbs)

| Method | Purpose | Example | Idempotent* |
|--------|---------|---------|-------------|
| **GET** | Retrieve data | Get user by ID | ✅ Yes |
| **POST** | Create new resource | Create new user | ❌ No |
| **PUT** | Update/Replace entire resource | Update all user fields | ✅ Yes |
| **PATCH** | Update partial resource | Update user email only | ❌ No |
| **DELETE** | Remove resource | Delete user | ✅ Yes |

*Idempotent = Multiple identical requests have same effect as single request

### HTTP Status Codes

**Success (2xx):**
- `200 OK` - Request succeeded (GET, PUT, PATCH)
- `201 Created` - Resource created (POST)
- `204 No Content` - Success with no response body (DELETE)

**Client Errors (4xx):**
- `400 Bad Request` - Invalid input/validation failed
- `401 Unauthorized` - Authentication required
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - Resource conflict (duplicate)

**Server Errors (5xx):**
- `500 Internal Server Error` - Unhandled exception
- `503 Service Unavailable` - Service down

### RESTful URL Design Principles

✅ **Good:**
```
GET    /api/users          - Get all users
GET    /api/users/5        - Get user 5
POST   /api/users          - Create user
PUT    /api/users/5        - Update user 5
DELETE /api/users/5        - Delete user 5
GET    /api/users/5/orders - Get orders for user 5
```

❌ **Bad:**
```
GET    /api/getUsers
POST   /api/createUser
GET    /api/user?action=delete&id=5
```

**Key Principles:**
1. Use nouns, not verbs (users, not getUsers)
2. Use plural nouns (users, not user)
3. Use HTTP methods for actions
4. Keep URLs hierarchical (users/5/orders)
5. Use lowercase and hyphens (user-profiles, not UserProfiles)

---

## 2. Building REST APIs (3 Approaches)

### Method 1: Minimal APIs ✨ (.NET 6+) - Quick

**When to use:**
- ✅ Small APIs (5-10 endpoints)
- ✅ Microservices
- ✅ Prototyping/Learning
- ❌ Complex business logic
- ❌ Large applications (20+ endpoints)

**Step 1: Basic Setup**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Define endpoints here...

app.Run();
```

**Step 2: Define Endpoints**

```csharp
// GET /api/users
app.MapGet("/api/users", () => 
{
    return Results.Ok(new[] 
    { 
        new { Id = 1, Name = "Alice" },
        new { Id = 2, Name = "Bob" }
    });
});

// GET /api/users/{id}
app.MapGet("/api/users/{id}", (int id) =>
{
    if (id <= 0)
        return Results.BadRequest("Invalid ID");
    
    var user = new { Id = id, Name = "Alice" };
    return Results.Ok(user);
});

// POST /api/users
app.MapPost("/api/users", (User user) =>
{
    // Validation happens automatically
    return Results.Created($"/api/users/{user.Id}", user);
});

// PUT /api/users/{id}
app.MapPut("/api/users/{id}", (int id, User user) =>
{
    if (id != user.Id)
        return Results.BadRequest("ID mismatch");
    
    return Results.Ok(user);
});

// DELETE /api/users/{id}
app.MapDelete("/api/users/{id}", (int id) =>
{
    return Results.NoContent();
});

record User(int Id, string Name, string Email);
```

**Step 3: Complete CRUD Example with Database**

```csharp
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// GET all users
app.MapGet("/api/users", async (AppDbContext db) =>
{
    var users = await db.Users.ToListAsync();
    return Results.Ok(users);
});

// GET user by ID
app.MapGet("/api/users/{id}", async (int id, AppDbContext db) =>
{
    var user = await db.Users.FindAsync(id);
    return user is null ? Results.NotFound() : Results.Ok(user);
});

// POST create user
app.MapPost("/api/users", async (User user, AppDbContext db) =>
{
    db.Users.Add(user);
    await db.SaveChangesAsync();
    return Results.Created($"/api/users/{user.Id}", user);
});

// PUT update user
app.MapPut("/api/users/{id}", async (int id, User user, AppDbContext db) =>
{
    if (id != user.Id)
        return Results.BadRequest("ID mismatch");
    
    var existing = await db.Users.FindAsync(id);
    if (existing is null)
        return Results.NotFound();
    
    existing.Name = user.Name;
    existing.Email = user.Email;
    await db.SaveChangesAsync();
    
    return Results.Ok(existing);
});

// DELETE user
app.MapDelete("/api/users/{id}", async (int id, AppDbContext db) =>
{
    var user = await db.Users.FindAsync(id);
    if (user is null)
        return Results.NotFound();
    
    db.Users.Remove(user);
    await db.SaveChangesAsync();
    return Results.NoContent();
});

app.Run();

// Models
class User
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}

class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<User> Users => Set<User>();
}
```

---

### Method 2: API Controllers ⭐ - Standard (RECOMMENDED)

**When to use:**
- ✅ Most production APIs
- ✅ Complex business logic
- ✅ Large applications
- ✅ Need filters, authorization
- ✅ Team development

**Step 1: Create Controller**

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly AppDbContext _context;
    private readonly ILogger<UsersController> _logger;
    
    public UsersController(
        AppDbContext context, 
        ILogger<UsersController> logger)
    {
        _context = context;
        _logger = logger;
    }
    
    // GET: api/users
    [HttpGet]
    public async Task<ActionResult<IEnumerable<User>>> GetUsers()
    {
        var users = await _context.Users.ToListAsync();
        return Ok(users);
    }
    
    // GET: api/users/5
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        var user = await _context.Users.FindAsync(id);
        
        if (user == null)
            return NotFound();
        
        return Ok(user);
    }
    
    // POST: api/users
    [HttpPost]
    public async Task<ActionResult<User>> CreateUser(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        
        return CreatedAtAction(
            nameof(GetUser), 
            new { id = user.Id }, 
            user);
    }
    
    // PUT: api/users/5
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateUser(int id, User user)
    {
        if (id != user.Id)
            return BadRequest("ID mismatch");
        
        var existing = await _context.Users.FindAsync(id);
        if (existing == null)
            return NotFound();
        
        existing.Name = user.Name;
        existing.Email = user.Email;
        
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
    
    // DELETE: api/users/5
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
}
```

**Step 2: Register in Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

builder.Services.AddControllers(); // ⚠️ Required for controllers

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();

app.MapControllers(); // ⚠️ Required for controllers

app.Run();
```

**Step 3: Complete CRUD Controller with Best Practices**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class UsersController : ControllerBase
{
    private readonly AppDbContext _context;
    private readonly ILogger<UsersController> _logger;
    
    public UsersController(
        AppDbContext context,
        ILogger<UsersController> logger)
    {
        _context = context;
        _logger = logger;
    }
    
    /// <summary>
    /// Get all users with pagination
    /// </summary>
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<ActionResult<PagedResult<User>>> GetUsers(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10)
    {
        if (page < 1 || pageSize < 1 || pageSize > 100)
            return BadRequest("Invalid pagination parameters");
        
        var totalCount = await _context.Users.CountAsync();
        var users = await _context.Users
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
        
        var result = new PagedResult<User>
        {
            Items = users,
            Page = page,
            PageSize = pageSize,
            TotalCount = totalCount
        };
        
        return Ok(result);
    }
    
    /// <summary>
    /// Get user by ID
    /// </summary>
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        var user = await _context.Users.FindAsync(id);
        
        if (user == null)
        {
            _logger.LogWarning("User {UserId} not found", id);
            return NotFound(new { message = $"User {id} not found" });
        }
        
        return Ok(user);
    }
    
    /// <summary>
    /// Create new user
    /// </summary>
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<User>> CreateUser(User user)
    {
        // Check for duplicate email
        if (await _context.Users.AnyAsync(u => u.Email == user.Email))
            return Conflict(new { message = "Email already exists" });
        
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Created user {UserId}", user.Id);
        
        return CreatedAtAction(
            nameof(GetUser),
            new { id = user.Id },
            user);
    }
    
    /// <summary>
    /// Update existing user
    /// </summary>
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> UpdateUser(int id, User user)
    {
        if (id != user.Id)
            return BadRequest(new { message = "ID mismatch" });
        
        var existing = await _context.Users.FindAsync(id);
        if (existing == null)
            return NotFound(new { message = $"User {id} not found" });
        
        // Check email uniqueness
        if (await _context.Users.AnyAsync(u => 
            u.Email == user.Email && u.Id != id))
            return Conflict(new { message = "Email already exists" });
        
        existing.Name = user.Name;
        existing.Email = user.Email;
        
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Updated user {UserId}", id);
        
        return NoContent();
    }
    
    /// <summary>
    /// Delete user
    /// </summary>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteUser(int id)
    {
        var user = await _context.Users.FindAsync(id);
        if (user == null)
            return NotFound(new { message = $"User {id} not found" });
        
        _context.Users.Remove(user);
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Deleted user {UserId}", id);
        
        return NoContent();
    }
}

// Helper class for pagination
public class PagedResult<T>
{
    public IEnumerable<T> Items { get; set; } = Enumerable.Empty<T>();
    public int Page { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
}
```

---

### Method 3: MVC Controllers - Full MVC

**When to use:**
- ✅ Mixed API + web pages
- ✅ Server-side rendering
- ❌ Pure APIs (use ApiController instead)

```csharp
using Microsoft.AspNetCore.Mvc;

public class UsersController : Controller // Inherit from Controller, not ControllerBase
{
    // GET: /users
    public IActionResult Index()
    {
        return View(); // Returns HTML view
    }
    
    // GET: /users/details/5
    public IActionResult Details(int id)
    {
        var user = GetUser(id);
        return View(user); // Returns HTML
    }
    
    // API endpoint (returns JSON)
    [HttpGet("api/users")]
    public IActionResult GetUsersApi()
    {
        var users = GetAllUsers();
        return Json(users); // Returns JSON
    }
}
```

---

### Comparison: Which Approach?

| Feature | Minimal APIs | API Controllers ⭐ | MVC Controllers |
|---------|--------------|-------------------|-----------------|
| Setup | Simplest | Medium | Complex |
| Code Lines | Fewest | Moderate | Most |
| Filters/Attributes | Limited | Full support ✅ | Full support ✅ |
| Model Binding | Basic | Full ✅ | Full ✅ |
| Validation | Basic | Full ✅ | Full ✅ |
| Testing | Harder | Easy ✅ | Easy ✅ |
| Organization | Inline | Files/Folders ✅ | Files/Folders ✅ |
| Performance | Fastest | Fast | Fast |
| Best For | Small APIs | Production APIs ✅ | Web + API |

**Decision Tree:**
```
How many endpoints?
├─ < 10 → Minimal APIs
└─ 10+ → Need views/pages?
         ├─ YES → MVC Controllers
         └─ NO → API Controllers ⭐ RECOMMENDED
```

---

## 3. DTOs (Data Transfer Objects)

**What are DTOs?**
Objects designed specifically for transferring data between layers or over the network.

**Why use DTOs?**
1. **Security** - Don't expose internal models (hide sensitive fields)
2. **Versioning** - API changes don't break database models
3. **Performance** - Send only what's needed
4. **Clarity** - Different shapes for different operations

### The Problem Without DTOs

```csharp
// ❌ Bad: Exposing database model directly
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string PasswordHash { get; set; } // ⚠️ SECURITY ISSUE!
    public DateTime CreatedAt { get; set; }
    public DateTime? LastLoginAt { get; set; }
    public bool IsDeleted { get; set; } // ⚠️ Internal flag exposed
}

[HttpGet("{id}")]
public async Task<ActionResult<User>> GetUser(int id)
{
    var user = await _context.Users.FindAsync(id);
    return Ok(user); // ❌ Returns password hash!
}
```

### The Solution With DTOs

**Step 1: Create DTOs for Different Operations**

```csharp
// Request DTOs (what client sends)
public class CreateUserRequest
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}

public class UpdateUserRequest
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}

// Response DTOs (what client receives)
public class UserResponse
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
}

public class UserDetailResponse : UserResponse
{
    public DateTime? LastLoginAt { get; set; }
    public int OrderCount { get; set; }
}

// Database model (internal)
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string PasswordHash { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
    public DateTime? LastLoginAt { get; set; }
    public bool IsDeleted { get; set; }
}
```

**Step 2: Use DTOs in Controller**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly AppDbContext _context;
    
    public UsersController(AppDbContext context)
    {
        _context = context;
    }
    
    // GET: api/users
    [HttpGet]
    public async Task<ActionResult<IEnumerable<UserResponse>>> GetUsers()
    {
        var users = await _context.Users
            .Where(u => !u.IsDeleted)
            .Select(u => new UserResponse
            {
                Id = u.Id,
                Name = u.Name,
                Email = u.Email,
                CreatedAt = u.CreatedAt
            })
            .ToListAsync();
        
        return Ok(users); // ✅ No sensitive data
    }
    
    // GET: api/users/5
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDetailResponse>> GetUser(int id)
    {
        var user = await _context.Users
            .Where(u => u.Id == id && !u.IsDeleted)
            .Select(u => new UserDetailResponse
            {
                Id = u.Id,
                Name = u.Name,
                Email = u.Email,
                CreatedAt = u.CreatedAt,
                LastLoginAt = u.LastLoginAt,
                OrderCount = u.Orders.Count
            })
            .FirstOrDefaultAsync();
        
        if (user == null)
            return NotFound();
        
        return Ok(user);
    }
    
    // POST: api/users
    [HttpPost]
    public async Task<ActionResult<UserResponse>> CreateUser(CreateUserRequest request)
    {
        // Map DTO → Entity
        var user = new User
        {
            Name = request.Name,
            Email = request.Email,
            PasswordHash = HashPassword(request.Password), // ✅ Hash password
            CreatedAt = DateTime.UtcNow
        };
        
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        
        // Map Entity → DTO
        var response = new UserResponse
        {
            Id = user.Id,
            Name = user.Name,
            Email = user.Email,
            CreatedAt = user.CreatedAt
        };
        
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, response);
    }
    
    // PUT: api/users/5
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateUser(int id, UpdateUserRequest request)
    {
        var user = await _context.Users.FindAsync(id);
        if (user == null || user.IsDeleted)
            return NotFound();
        
        // Map DTO → Entity (only allowed fields)
        user.Name = request.Name;
        user.Email = request.Email;
        
        await _context.SaveChangesAsync();
        
        return NoContent();
    }
    
    private string HashPassword(string password)
    {
        // Use BCrypt, PBKDF2, or ASP.NET Core Identity
        return BCrypt.Net.BCrypt.HashPassword(password);
    }
}
```

---

### Mapping DTOs (3 Methods)

### Method 1: Manual Mapping - Simple

**When to use:**
- ✅ Small projects
- ✅ Simple mappings
- ✅ Learning
- ❌ Large projects
- ❌ Complex mappings

```csharp
// Manual mapping in controller
var response = new UserResponse
{
    Id = user.Id,
    Name = user.Name,
    Email = user.Email,
    CreatedAt = user.CreatedAt
};
```

**Pros:** No dependencies, simple, clear
**Cons:** Repetitive, error-prone, tedious

---

### Method 2: AutoMapper ⭐ - Powerful (RECOMMENDED)

**When to use:**
- ✅ Production applications
- ✅ Many DTOs
- ✅ Complex mappings
- ✅ Consistent mapping logic

**Step 1: Install Package**
```bash
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

**Step 2: Create Mapping Profile**

```csharp
using AutoMapper;

public class MappingProfile : Profile
{
    public MappingProfile()
    {
        // Simple mapping (properties match by name)
        CreateMap<User, UserResponse>();
        
        // Reverse mapping
        CreateMap<CreateUserRequest, User>()
            .ForMember(dest => dest.Id, opt => opt.Ignore())
            .ForMember(dest => dest.CreatedAt, opt => opt.MapFrom(_ => DateTime.UtcNow))
            .ForMember(dest => dest.PasswordHash, opt => opt.Ignore()); // Handle separately
        
        // Custom mapping
        CreateMap<User, UserDetailResponse>()
            .ForMember(dest => dest.OrderCount, 
                opt => opt.MapFrom(src => src.Orders.Count));
        
        // Flatten nested objects
        CreateMap<Order, OrderResponse>()
            .ForMember(dest => dest.UserName, 
                opt => opt.MapFrom(src => src.User.Name))
            .ForMember(dest => dest.UserEmail, 
                opt => opt.MapFrom(src => src.User.Email));
    }
}
```

**Step 3: Register in Program.cs**

```csharp
builder.Services.AddAutoMapper(typeof(Program));
// or
builder.Services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());
```

**Step 4: Use in Controller**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly AppDbContext _context;
    private readonly IMapper _mapper;
    
    public UsersController(AppDbContext context, IMapper mapper)
    {
        _context = context;
        _mapper = mapper;
    }
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<UserResponse>>> GetUsers()
    {
        var users = await _context.Users.ToListAsync();
        var response = _mapper.Map<IEnumerable<UserResponse>>(users);
        return Ok(response);
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDetailResponse>> GetUser(int id)
    {
        var user = await _context.Users
            .Include(u => u.Orders)
            .FirstOrDefaultAsync(u => u.Id == id);
        
        if (user == null)
            return NotFound();
        
        var response = _mapper.Map<UserDetailResponse>(user);
        return Ok(response);
    }
    
    [HttpPost]
    public async Task<ActionResult<UserResponse>> CreateUser(CreateUserRequest request)
    {
        var user = _mapper.Map<User>(request);
        user.PasswordHash = HashPassword(request.Password);
        
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        
        var response = _mapper.Map<UserResponse>(user);
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, response);
    }
}
```

---

### Method 3: Mapster - Fast & Simple

**When to use:**
- ✅ Performance critical
- ✅ Simpler than AutoMapper
- ✅ Less configuration

**Step 1: Install Package**
```bash
dotnet add package Mapster
dotnet add package Mapster.DependencyInjection
```

**Step 2: Configure Mappings**

```csharp
using Mapster;

public class MappingConfig
{
    public static void RegisterMappings()
    {
        // Convention-based (auto-maps matching properties)
        TypeAdapterConfig<User, UserResponse>.NewConfig();
        
        // Custom mapping
        TypeAdapterConfig<User, UserDetailResponse>
            .NewConfig()
            .Map(dest => dest.OrderCount, src => src.Orders.Count);
        
        TypeAdapterConfig<CreateUserRequest, User>
            .NewConfig()
            .Ignore(dest => dest.Id)
            .Map(dest => dest.CreatedAt, src => DateTime.UtcNow);
    }
}

// In Program.cs
MappingConfig.RegisterMappings();
```

**Step 3: Use in Controller**

```csharp
[HttpGet]
public async Task<ActionResult<IEnumerable<UserResponse>>> GetUsers()
{
    var users = await _context.Users.ToListAsync();
    var response = users.Adapt<IEnumerable<UserResponse>>();
    return Ok(response);
}

[HttpPost]
public async Task<ActionResult<UserResponse>> CreateUser(CreateUserRequest request)
{
    var user = request.Adapt<User>();
    user.PasswordHash = HashPassword(request.Password);
    
    _context.Users.Add(user);
    await _context.SaveChangesAsync();
    
    var response = user.Adapt<UserResponse>();
    return CreatedAtAction(nameof(GetUser), new { id = user.Id }, response);
}
```

---

### Mapping Comparison

| Feature | Manual | AutoMapper ⭐ | Mapster |
|---------|--------|--------------|---------|
| Setup | None | Medium | Simple |
| Performance | Fastest | Good | Faster |
| Flexibility | Limited | High ✅ | Medium |
| Learning Curve | Easiest | Medium | Easy |
| Configuration | None | Required | Optional |
| Community | N/A | Large ✅ | Growing |
| Best For | Small projects | Production ✅ | Performance |

---

## 4. Model Validation (3 Methods)

**What is validation?**
Ensuring data meets business rules before processing.

### Method 1: Data Annotations ⭐ - Built-in (RECOMMENDED for simple cases)

**When to use:**
- ✅ Simple validation rules
- ✅ Quick setup
- ✅ Standard validation
- ❌ Complex business logic
- ❌ Cross-property validation

**Step 1: Add Attributes to DTO**

```csharp
using System.ComponentModel.DataAnnotations;

public class CreateUserRequest
{
    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2, 
        ErrorMessage = "Name must be between 2 and 100 characters")]
    public string Name { get; set; } = string.Empty;
    
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    public string Email { get; set; } = string.Empty;
    
    [Required(ErrorMessage = "Password is required")]
    [StringLength(100, MinimumLength = 8, 
        ErrorMessage = "Password must be at least 8 characters")]
    [RegularExpression(@"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).+$",
        ErrorMessage = "Password must contain uppercase, lowercase, and number")]
    public string Password { get; set; } = string.Empty;
    
    [Range(18, 120, ErrorMessage = "Age must be between 18 and 120")]
    public int Age { get; set; }
    
    [Url(ErrorMessage = "Invalid URL format")]
    public string? Website { get; set; }
    
    [Phone(ErrorMessage = "Invalid phone number")]
    public string? PhoneNumber { get; set; }
}
```

**Step 2: Validation Happens Automatically with [ApiController]**

```csharp
[ApiController] // ⚠️ This enables automatic validation
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public async Task<ActionResult<UserResponse>> CreateUser(
        CreateUserRequest request) // ✅ Validated automatically
    {
        // If validation fails, returns 400 Bad Request automatically
        // If validation passes, code continues here
        
        var user = new User
        {
            Name = request.Name,
            Email = request.Email,
            // ... create user
        };
        
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
}
```

**Step 3: Custom Error Response (Optional)**

```csharp
// In Program.cs - Customize validation error format
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.InvalidModelStateResponseFactory = context =>
        {
            var errors = context.ModelState
                .Where(e => e.Value?.Errors.Count > 0)
                .Select(e => new
                {
                    Field = e.Key,
                    Errors = e.Value?.Errors.Select(x => x.ErrorMessage).ToArray()
                })
                .ToArray();
            
            return new BadRequestObjectResult(new
            {
                Message = "Validation failed",
                Errors = errors
            });
        };
    });
```

**Built-in Validation Attributes Reference:**

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[Required]` | Field must have value | `[Required]` |
| `[StringLength]` | String length limits | `[StringLength(100, MinimumLength = 2)]` |
| `[Range]` | Numeric range | `[Range(1, 100)]` |
| `[EmailAddress]` | Valid email format | `[EmailAddress]` |
| `[Phone]` | Valid phone number | `[Phone]` |
| `[Url]` | Valid URL | `[Url]` |
| `[RegularExpression]` | Matches regex pattern | `[RegularExpression(@"^\d{5}$")]` |
| `[Compare]` | Matches another property | `[Compare("Password")]` |
| `[CreditCard]` | Valid credit card | `[CreditCard]` |
| `[MinLength]` | Minimum collection size | `[MinLength(1)]` |
| `[MaxLength]` | Maximum collection size | `[MaxLength(10)]` |

---

### Method 2: FluentValidation ⭐ - Powerful (RECOMMENDED for complex validation)

**When to use:**
- ✅ Complex business rules
- ✅ Cross-property validation
- ✅ Reusable validation logic
- ✅ Testable validators
- ✅ Production applications

**Step 1: Install Package**
```bash
dotnet add package FluentValidation.AspNetCore
```

**Step 2: Create Validator Class**

```csharp
using FluentValidation;

public class CreateUserRequestValidator : AbstractValidator<CreateUserRequest>
{
    private readonly AppDbContext _context;
    
    public CreateUserRequestValidator(AppDbContext context)
    {
        _context = context;
        
        // Name validation
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required")
            .Length(2, 100).WithMessage("Name must be between 2 and 100 characters")
            .Must(BeValidName).WithMessage("Name must contain only letters and spaces");
        
        // Email validation
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MustAsync(BeUniqueEmail).WithMessage("Email already exists");
        
        // Password validation
        RuleFor(x => x.Password)
            .NotEmpty().WithMessage("Password is required")
            .MinimumLength(8).WithMessage("Password must be at least 8 characters")
            .Matches(@"[A-Z]").WithMessage("Password must contain uppercase letter")
            .Matches(@"[a-z]").WithMessage("Password must contain lowercase letter")
            .Matches(@"\d").WithMessage("Password must contain number")
            .Matches(@"[!@#$%^&*]").WithMessage("Password must contain special character");
        
        // Age validation
        RuleFor(x => x.Age)
            .InclusiveBetween(18, 120).WithMessage("Age must be between 18 and 120");
        
        // Website validation (optional field)
        RuleFor(x => x.Website)
            .Must(BeValidUrl).When(x => !string.IsNullOrEmpty(x.Website))
            .WithMessage("Invalid website URL");
    }
    
    private bool BeValidName(string name)
    {
        return name.All(c => char.IsLetter(c) || char.IsWhiteSpace(c));
    }
    
    private async Task<bool> BeUniqueEmail(string email, CancellationToken cancellationToken)
    {
        return !await _context.Users.AnyAsync(u => u.Email == email, cancellationToken);
    }
    
    private bool BeValidUrl(string? url)
    {
        return Uri.TryCreate(url, UriKind.Absolute, out _);
    }
}
```

**Step 3: Register in Program.cs**

```csharp
using FluentValidation;

builder.Services.AddControllers();

// Register FluentValidation
builder.Services.AddValidatorsFromAssemblyContaining<CreateUserRequestValidator>();
builder.Services.AddFluentValidationAutoValidation(); // ✨ .NET 6+ method
```

**Step 4: Use in Controller (Automatic)**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public async Task<ActionResult<UserResponse>> CreateUser(
        CreateUserRequest request) // ✅ Validated automatically by FluentValidation
    {
        // Validation already happened
        // If failed, 400 Bad Request returned automatically
        
        var user = new User
        {
            Name = request.Name,
            Email = request.Email,
            // ... create user
        };
        
        return Ok(user);
    }
}
```

**Step 5: Manual Validation (Optional)**

```csharp
[HttpPost]
public async Task<ActionResult<UserResponse>> CreateUser(
    CreateUserRequest request,
    IValidator<CreateUserRequest> validator) // Inject validator
{
    // Manual validation
    var validationResult = await validator.ValidateAsync(request);
    
    if (!validationResult.IsValid)
    {
        return BadRequest(new
        {
            Message = "Validation failed",
            Errors = validationResult.Errors.Select(e => new
            {
                Field = e.PropertyName,
                Error = e.ErrorMessage
            })
        });
    }
    
    // Continue with valid data...
}
```

**Advanced FluentValidation Examples:**

```csharp
public class UpdateUserRequestValidator : AbstractValidator<UpdateUserRequest>
{
    public UpdateUserRequestValidator()
    {
        // Conditional validation
        When(x => x.Age < 18, () =>
        {
            RuleFor(x => x.ParentEmail)
                .NotEmpty()
                .EmailAddress();
        });
        
        // Cross-property validation
        RuleFor(x => x.EndDate)
            .GreaterThan(x => x.StartDate)
            .WithMessage("End date must be after start date");
        
        // Collection validation
        RuleFor(x => x.Tags)
            .NotEmpty().WithMessage("At least one tag required")
            .Must(tags => tags.Count <= 5).WithMessage("Maximum 5 tags allowed");
        
        // Custom validation with dependency injection
        RuleFor(x => x.Username)
            .MustAsync(async (username, cancellation) =>
            {
                // Complex async validation
                return await CheckUsernameAvailable(username);
            })
            .WithMessage("Username not available");
        
        // Nested object validation
        RuleFor(x => x.Address)
            .SetValidator(new AddressValidator());
    }
}

public class AddressValidator : AbstractValidator<Address>
{
    public AddressValidator()
    {
        RuleFor(x => x.Street).NotEmpty();
        RuleFor(x => x.City).NotEmpty();
        RuleFor(x => x.ZipCode).Matches(@"^\d{5}$");
    }
}
```

---

### Method 3: Custom Validation Attributes - Reusable

**When to use:**
- ✅ Reusable validation logic
- ✅ Attribute-based approach preferred
- ✅ Simple to medium complexity

**Step 1: Create Custom Attribute**

```csharp
using System.ComponentModel.DataAnnotations;

public class FutureDateAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(
        object? value, 
        ValidationContext validationContext)
    {
        if (value is DateTime date)
        {
            if (date <= DateTime.UtcNow)
            {
                return new ValidationResult(
                    ErrorMessage ?? "Date must be in the future");
            }
        }
        
        return ValidationResult.Success;
    }
}

public class UniqueEmailAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(
        object? value,
        ValidationContext validationContext)
    {
        if (value is string email)
        {
            var context = (AppDbContext)validationContext
                .GetService(typeof(AppDbContext))!;
            
            if (context.Users.Any(u => u.Email == email))
            {
                return new ValidationResult(
                    ErrorMessage ?? "Email already exists");
            }
        }
        
        return ValidationResult.Success;
    }
}

// With parameters
public class MinimumAgeAttribute : ValidationAttribute
{
    private readonly int _minimumAge;
    
    public MinimumAgeAttribute(int minimumAge)
    {
        _minimumAge = minimumAge;
    }
    
    protected override ValidationResult? IsValid(
        object? value,
        ValidationContext validationContext)
    {
        if (value is DateTime birthDate)
        {
            var age = DateTime.Today.Year - birthDate.Year;
            
            if (birthDate.Date > DateTime.Today.AddYears(-age))
                age--;
            
            if (age < _minimumAge)
            {
                return new ValidationResult(
                    ErrorMessage ?? $"Must be at least {_minimumAge} years old");
            }
        }
        
        return ValidationResult.Success;
    }
}
```

**Step 2: Use Custom Attributes**

```csharp
public class CreateEventRequest
{
    [Required]
    public string Name { get; set; } = string.Empty;
    
    [FutureDate(ErrorMessage = "Event must be scheduled in the future")]
    public DateTime EventDate { get; set; }
    
    [UniqueEmail]
    [EmailAddress]
    public string OrganizerEmail { get; set; } = string.Empty;
    
    [MinimumAge(18, ErrorMessage = "Organizer must be at least 18")]
    public DateTime OrganizerBirthDate { get; set; }
}
```

---

### Validation Comparison

| Feature | Data Annotations | FluentValidation ⭐ | Custom Attributes |
|---------|------------------|---------------------|-------------------|
| Setup | None | Package install | Create classes |
| Complexity | Simple | Complex ✅ | Medium |
| Async Validation | Limited | Full support ✅ | Supported |
| Testability | Hard | Easy ✅ | Medium |
| Separation of Concerns | Mixed | Clean ✅ | Mixed |
| Cross-property | Limited | Easy ✅ | Medium |
| Dependency Injection | Limited | Full ✅ | Limited |
| Best For | Simple validation | Production apps ✅ | Reusable rules |

**Decision Tree:**
```
Need async validation or DI?
├─ YES → FluentValidation ⭐
└─ NO → Complex business rules?
         ├─ YES → FluentValidation ⭐
         └─ NO → Need reusable attributes?
                  ├─ YES → Custom Attributes
                  └─ NO → Data Annotations ✅
```

---

## 5. Content Negotiation

**What is content negotiation?**
The process of selecting the best representation of a resource based on client preferences (JSON, XML, etc.)

### Default: JSON

```csharp
// Client sends: Accept: application/json
// Server responds with JSON (default)

[HttpGet("{id}")]
public ActionResult<User> GetUser(int id)
{
    var user = GetUserById(id);
    return Ok(user); // Returns JSON by default
}
```

### Supporting XML

**Step 1: Add XML Support**

```csharp
// In Program.cs
builder.Services.AddControllers()
    .AddXmlSerializerFormatters(); // Add XML support
```

**Step 2: Client Requests XML**

```
GET /api/users/5
Accept: application/xml
```

**Response:**
```xml
<User>
    <Id>5</Id>
    <Name>Alice</Name>
    <Email>alice@example.com</Email>
</User>
```

### Force Specific Format

```csharp
[HttpGet("{id}")]
[Produces("application/json")] // Only returns JSON
public ActionResult<User> GetUser(int id)
{
    return Ok(GetUserById(id));
}

[HttpGet("{id}/xml")]
[Produces("application/xml")] // Only returns XML
public ActionResult<User> GetUserXml(int id)
{
    return Ok(GetUserById(id));
}
```

### Custom Formatters

```csharp
// CSV formatter example
public class CsvOutputFormatter : TextOutputFormatter
{
    public CsvOutputFormatter()
    {
        SupportedMediaTypes.Add("text/csv");
        SupportedEncodings.Add(Encoding.UTF8);
    }
    
    public override async Task WriteResponseBodyAsync(
        OutputFormatterWriteContext context,
        Encoding selectedEncoding)
    {
        var response = context.HttpContext.Response;
        
        if (context.Object is IEnumerable<User> users)
        {
            var csv = new StringBuilder();
            csv.AppendLine("Id,Name,Email");
            
            foreach (var user in users)
            {
                csv.AppendLine($"{user.Id},{user.Name},{user.Email}");
            }
            
            await response.WriteAsync(csv.ToString(), selectedEncoding);
        }
    }
}

// Register in Program.cs
builder.Services.AddControllers(options =>
{
    options.OutputFormatters.Add(new CsvOutputFormatter());
});
```

---

## 6. Error Handling & Problem Details

### The Problem Without Error Handling

```csharp
// ❌ Bad: Unhandled exceptions
[HttpGet("{id}")]
public async Task<ActionResult<User>> GetUser(int id)
{
    var user = await _context.Users.FindAsync(id); // Could throw exception
    return Ok(user); // What if user is null?
}
```

### Method 1: Try-Catch in Controller

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<User>> GetUser(int id)
{
    try
    {
        var user = await _context.Users.FindAsync(id);
        
        if (user == null)
            return NotFound(new { message = $"User {id} not found" });
        
        return Ok(user);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error retrieving user {UserId}", id);
        return StatusCode(500, new { message = "An error occurred" });
    }
}
```

### Method 2: Global Exception Middleware ⭐ (RECOMMENDED)

**Step 1: Create Exception Middleware**

```csharp
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;
    
    public GlobalExceptionMiddleware(
        RequestDelegate next,
        ILogger<GlobalExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (NotFoundException ex)
        {
            _logger.LogWarning(ex, "Resource not found");
            context.Response.StatusCode = 404;
            await context.Response.WriteAsJsonAsync(new
            {
                message = ex.Message,
                type = "NotFound"
            });
        }
        catch (ValidationException ex)
        {
            _logger.LogWarning(ex, "Validation failed");
            context.Response.StatusCode = 400;
            await context.Response.WriteAsJsonAsync(new
            {
                message = "Validation failed",
                errors = ex.Errors
            });
        }
        catch (UnauthorizedAccessException ex)
        {
            _logger.LogWarning(ex, "Unauthorized access");
            context.Response.StatusCode = 401;
            await context.Response.WriteAsJsonAsync(new
            {
                message = "Unauthorized"
            });
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception");
            context.Response.StatusCode = 500;
            await context.Response.WriteAsJsonAsync(new
            {
                message = "An unexpected error occurred",
                // Only include details in development
                details = context.RequestServices
                    .GetRequiredService<IHostEnvironment>()
                    .IsDevelopment() ? ex.Message : null
            });
        }
    }
}

// Custom exceptions
public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) { }
}

public class ValidationException : Exception
{
    public Dictionary<string, string[]> Errors { get; }
    
    public ValidationException(Dictionary<string, string[]> errors)
        : base("Validation failed")
    {
        Errors = errors;
    }
}
```

**Step 2: Register in Program.cs**

```csharp
var app = builder.Build();

app.UseMiddleware<GlobalExceptionMiddleware>(); // ⚠️ First in pipeline

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Step 3: Use in Controllers**

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<User>> GetUser(int id)
{
    var user = await _context.Users.FindAsync(id);
    
    if (user == null)
        throw new NotFoundException($"User {id} not found"); // ✅ Middleware handles it
    
    return Ok(user);
}
```

---

### Method 3: Problem Details (RFC 7807) ⭐ - Standard

**What is Problem Details?**
A standard format for HTTP API error responses defined in RFC 7807.

**Step 1: Enable Problem Details**

```csharp
// In Program.cs
builder.Services.AddProblemDetails();

var app = builder.Build();

// Use built-in exception handler
app.UseExceptionHandler();
app.UseStatusCodePages(); // For 404, 401, etc.

// Or use developer exception page in development
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler();
}
```

**Step 2: Return Problem Details**

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<User>> GetUser(int id)
{
    var user = await _context.Users.FindAsync(id);
    
    if (user == null)
    {
        return Problem(
            statusCode: StatusCodes.Status404NotFound,
            title: "User not found",
            detail: $"User with ID {id} was not found",
            instance: $"/api/users/{id}");
    }
    
    return Ok(user);
}
```

**Response:**
```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  "title": "User not found",
  "status": 404,
  "detail": "User with ID 5 was not found",
  "instance": "/api/users/5",
  "traceId": "00-abc123..."
}
```

**Step 3: Custom Problem Details**

```csharp
// In Program.cs
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = context =>
    {
        context.ProblemDetails.Extensions["traceId"] = 
            Activity.Current?.Id ?? context.HttpContext.TraceIdentifier;
        
        context.ProblemDetails.Extensions["timestamp"] = 
            DateTime.UtcNow;
        
        // Add custom data
        if (context.HttpContext.Items.TryGetValue("errors", out var errors))
        {
            context.ProblemDetails.Extensions["errors"] = errors;
        }
    };
});
```

---

### Error Response Comparison

| Approach | Consistency | Complexity | Standard | Best For |
|----------|-------------|------------|----------|----------|
| Try-Catch | Low | Simple | No | Small projects |
| Global Middleware | High ✅ | Medium | No | Custom needs |
| Problem Details | High ✅ | Medium | Yes ✅ | Production ⭐ |

---

## 7. API Versioning

**Why version APIs?**
- Breaking changes without affecting existing clients
- Gradual migration path
- Support multiple versions simultaneously

### Method 1: URL Versioning ⭐ (RECOMMENDED)

**Step 1: Install Package**
```bash
dotnet add package Asp.Versioning.Mvc
```

**Step 2: Configure Versioning**

```csharp
// In Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
}).AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});
```

**Step 3: Create Versioned Controllers**

```csharp
// Version 1.0
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class UsersController : ControllerBase
{
    [HttpGet("{id}")]
    public ActionResult<UserV1Response> GetUser(int id)
    {
        return Ok(new UserV1Response
        {
            Id = id,
            Name = "Alice",
            Email = "alice@example.com"
        });
    }
}

// Version 2.0 (breaking changes)
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("2.0")]
public class UsersV2Controller : ControllerBase
{
    [HttpGet("{id}")]
    public ActionResult<UserV2Response> GetUser(int id)
    {
        return Ok(new UserV2Response
        {
            Id = id,
            FirstName = "Alice", // ✨ Changed: Split name
            LastName = "Smith",
            Email = "alice@example.com",
            CreatedAt = DateTime.UtcNow // ✨ New field
        });
    }
}

public class UserV1Response
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}

public class UserV2Response
{
    public int Id { get; set; }
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
}
```

**Usage:**
```
GET /api/v1/users/5  → Returns UserV1Response
GET /api/v2/users/5  → Returns UserV2Response
```

---

### Method 2: Query String Versioning

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new QueryStringApiVersionReader("api-version");
});

[ApiController]
[Route("api/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class UsersController : ControllerBase
{
    [HttpGet("{id}"), MapToApiVersion("1.0")]
    public ActionResult<UserV1Response> GetUserV1(int id)
    {
        // Version 1 logic
    }
    
    [HttpGet("{id}"), MapToApiVersion("2.0")]
    public ActionResult<UserV2Response> GetUserV2(int id)
    {
        // Version 2 logic
    }
}
```

**Usage:**
```
GET /api/users/5?api-version=1.0
GET /api/users/5?api-version=2.0
```

---

### Method 3: Header Versioning

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new HeaderApiVersionReader("X-API-Version");
});
```

**Usage:**
```
GET /api/users/5
Headers: X-API-Version: 1.0
```

---

### Versioning Comparison

| Method | Pros | Cons | Best For |
|--------|------|------|----------|
| **URL** ⭐ | Clear, cacheable, browser-friendly | URLs change | Public APIs |
| **Query String** | Clean URLs, flexible | Not RESTful, harder to cache | Internal APIs |
| **Header** | Clean URLs, RESTful | Not browser-friendly | Advanced clients |

---

## 8. Swagger/OpenAPI Documentation

**What is Swagger?**
Interactive API documentation that lets developers test endpoints in browser.

### Step 1: Add Swagger (Included in Web API template)

```csharp
// In Program.cs
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1",
        Description = "A sample ASP.NET Core Web API",
        Contact = new OpenApiContact
        {
            Name = "Your Name",
            Email = "your.email@example.com",
            Url = new Uri("https://example.com")
        },
        License = new OpenApiLicense
        {
            Name = "MIT License",
            Url = new Uri("https://opensource.org/licenses/MIT")
        }
    });
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
        options.RoutePrefix = string.Empty; // Swagger at root (/)
    });
}
```

### Step 2: Add XML Documentation Comments

**Enable XML documentation:**

```xml
<!-- In .csproj -->
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup>
```

**Add XML comments to controllers:**

```csharp
/// <summary>
/// Get user by ID
/// </summary>
/// <param name="id">User ID</param>
/// <returns>User details</returns>
/// <response code="200">Returns the user</response>
/// <response code="404">User not found</response>
[HttpGet("{id}")]
[ProducesResponseType(typeof(UserResponse), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<ActionResult<UserResponse>> GetUser(int id)
{
    var user = await _context.Users.FindAsync(id);
    
    if (user == null)
        return NotFound();
    
    return Ok(user);
}
```

**Configure Swagger to use XML comments:**

```csharp
builder.Services.AddSwaggerGen(options =>
{
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});
```

### Step 3: Add Response Examples

```csharp
/// <summary>
/// Create new user
/// </summary>
/// <remarks>
/// Sample request:
/// 
///     POST /api/users
///     {
///        "name": "Alice Smith",
///        "email": "alice@example.com",
///        "age": 25
///     }
/// 
/// </remarks>
[HttpPost]
[ProducesResponseType(typeof(UserResponse), StatusCodes.Status201Created)]
[ProducesResponseType(typeof(ValidationProblemDetails), StatusCodes.Status400BadRequest)]
public async Task<ActionResult<UserResponse>> CreateUser(CreateUserRequest request)
{
    // Implementation...
}
```

### Access Swagger UI

Navigate to: `https://localhost:5001/` (if RoutePrefix set to empty)
Or: `https://localhost:5001/swagger`

---

## 9. Troubleshooting Common Issues

### Issue 1: Validation Not Working

**Problem:**
```csharp
[HttpPost]
public IActionResult CreateUser(User user) // ❌ No validation
{
    // Validation attributes ignored
}
```

**Solution:**
```csharp
[ApiController] // ⚠️ Required for automatic validation
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult CreateUser(User user) // ✅ Validation works
    {
        // [ApiController] enables automatic model validation
    }
}
```

---

### Issue 2: 415 Unsupported Media Type

**Problem:**
Client sends JSON but server rejects it.

**Solution:**
```csharp
// Ensure Content-Type header is set
// Client request:
POST /api/users
Content-Type: application/json  // ⚠️ Must be set

{
  "name": "Alice"
}

// Or explicitly accept JSON in controller
[HttpPost]
[Consumes("application/json")]
public IActionResult CreateUser([FromBody] User user)
{
    // ...
}
```

---

### Issue 3: Null Values in Request

**Problem:**
```csharp
[HttpPost]
public IActionResult CreateUser(User user) // user is null
{
    // ...
}
```

**Solutions:**
```csharp
// Solution 1: Add [FromBody]
[HttpPost]
public IActionResult CreateUser([FromBody] User user) // ✅ Works
{
    // ...
}

// Solution 2: Use [ApiController] (does this automatically)
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult CreateUser(User user) // ✅ [FromBody] inferred
    {
        // ...
    }
}
```

---

### Issue 4: Circular Reference in JSON

**Problem:**
```csharp
public class User
{
    public int Id { get; set; }
    public List<Order> Orders { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public User User { get; set; } // ❌ Circular reference
}
```

**Solutions:**
```csharp
// Solution 1: Use DTOs (RECOMMENDED)
public class UserResponse
{
    public int Id { get; set; }
    // No Orders property
}

// Solution 2: Configure JSON options
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.ReferenceHandler = 
            ReferenceHandler.IgnoreCycles;
    });

// Solution 3: Use [JsonIgnore]
public class Order
{
    public int Id { get; set; }
    
    [JsonIgnore]
    public User User { get; set; }
}
```

---

### Issue 5: CORS Errors

**Problem:**
```
Access to fetch at 'https://api.example.com' from origin 'https://app.example.com'
has been blocked by CORS policy
```

**Solution:**
```csharp
// In Program.cs
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins("https://app.example.com")
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

app.UseCors(); // ⚠️ Must be before UseAuthorization()
app.UseAuthorization();
app.MapControllers();
```

---

## 10. Best Practices

### ✅ Do's

1. **Use DTOs for API contracts**
   ```csharp
   // ✅ Good
   [HttpPost]
   public ActionResult<UserResponse> CreateUser(CreateUserRequest request)
   ```

2. **Use appropriate HTTP status codes**
   ```csharp
   // ✅ Good
   return Created($"/api/users/{user.Id}", userResponse); // 201
   return NotFound(); // 404
   return BadRequest(); // 400
   ```

3. **Version your APIs**
   ```csharp
   // ✅ Good
   [Route("api/v1/[controller]")]
   ```

4. **Document with Swagger**
   ```csharp
   // ✅ Good
   /// <summary>Get user by ID</summary>
   [HttpGet("{id}")]
   ```

5. **Validate input**
   ```csharp
   // ✅ Good
   public class CreateUserRequest
   {
       [Required]
       [EmailAddress]
       public string Email { get; set; }
   }
   ```

6. **Use async/await**
   ```csharp
   // ✅ Good
   [HttpGet]
   public async Task<ActionResult<IEnumerable<User>>> GetUsers()
   {
       return await _context.Users.ToListAsync();
   }
   ```

7. **Log appropriately**
   ```csharp
   // ✅ Good
   _logger.LogInformation("User {UserId} created", user.Id);
   _logger.LogError(ex, "Failed to create user");
   ```

8. **Use pagination for lists**
   ```csharp
   // ✅ Good
   [HttpGet]
   public ActionResult<PagedResult<User>> GetUsers(int page = 1, int pageSize = 10)
   ```

### ❌ Don'ts

1. **Don't expose database models directly**
   ```csharp
   // ❌ Bad
   [HttpGet]
   public ActionResult<User> GetUser(int id) // Database model
   ```

2. **Don't ignore validation**
   ```csharp
   // ❌ Bad
   [HttpPost]
   public IActionResult CreateUser(User user)
   {
       _context.Add(user); // No validation check
   }
   ```

3. **Don't use verbs in URLs**
   ```csharp
   // ❌ Bad
   [HttpGet("getUsers")]
   [HttpPost("createUser")]
   ```

4. **Don't return 200 for errors**
   ```csharp
   // ❌ Bad
   return Ok(new { error = "User not found" });
   
   // ✅ Good
   return NotFound(new { message = "User not found" });
   ```

5. **Don't forget authorization**
   ```csharp
   // ❌ Bad - Anyone can delete
   [HttpDelete("{id}")]
   public IActionResult DeleteUser(int id)
   
   // ✅ Good
   [HttpDelete("{id}")]
   [Authorize(Roles = "Admin")]
   public IActionResult DeleteUser(int id)
   ```

---

# PART 2: TECHNICAL REFERENCE

---

## 11. Important Interfaces & Classes Reference

### ControllerBase Class ⭐⭐⭐

**Purpose:** Base class for API controllers (no view support)

**Namespace:** `Microsoft.AspNetCore.Mvc`

**Key Members:**

| Member | Type | Purpose |
|--------|------|---------|
| `Request` | HttpRequest | Current HTTP request |
| `Response` | HttpResponse | Current HTTP response |
| `User` | ClaimsPrincipal | Current authenticated user |
| `ModelState` | ModelStateDictionary | Model validation state |
| `HttpContext` | HttpContext | Current HTTP context |

**Action Result Methods:**

| Method | Status Code | Purpose |
|--------|-------------|---------|
| `Ok(object)` | 200 | Success with data |
| `Created(string, object)` | 201 | Resource created |
| `CreatedAtAction(string, object, object)` | 201 | Created with location |
| `NoContent()` | 204 | Success, no content |
| `BadRequest(object)` | 400 | Invalid request |
| `Unauthorized()` | 401 | Authentication required |
| `Forbid()` | 403 | Not authorized |
| `NotFound(object)` | 404 | Resource not found |
| `Conflict(object)` | 409 | Resource conflict |
| `StatusCode(int, object)` | Custom | Custom status code |
| `Problem(ProblemDetails)` | Varies | Problem details response |

**Complete Example:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Access request information
    var path = Request.Path;
    var method = Request.Method;
    var query = Request.Query["search"];
    
    // Access user information
    var isAuthenticated = User?.Identity?.IsAuthenticated ?? false;
    var userName = User?.Identity?.Name;
    var userId = User?.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    
    // Check model validation
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    
    // Return different action results
    return Ok(data);              // 200
    return Created("/api/products/1", product); // 201
    return CreatedAtAction(nameof(GetProduct), new { id = 1 }, product); // 201
    return NoContent();           // 204
    return BadRequest();          // 400
    return NotFound();            // 404
    return Conflict();            // 409
    return StatusCode(500);       // 500
}
```

---

### IActionResult Interface ⭐⭐⭐

**Purpose:** Represents the result of an action method

**Namespace:** `Microsoft.AspNetCore.Mvc`

**Common Implementations:**

| Class | Status | Usage |
|-------|--------|-------|
| `OkResult` | 200 | `return Ok();` |
| `OkObjectResult` | 200 | `return Ok(data);` |
| `CreatedResult` | 201 | `return Created(uri, data);` |
| `CreatedAtActionResult` | 201 | `return CreatedAtAction(...);` |
| `NoContentResult` | 204 | `return NoContent();` |
| `BadRequestResult` | 400 | `return BadRequest();` |
| `BadRequestObjectResult` | 400 | `return BadRequest(error);` |
| `UnauthorizedResult` | 401 | `return Unauthorized();` |
| `ForbidResult` | 403 | `return Forbid();` |
| `NotFoundResult` | 404 | `return NotFound();` |
| `NotFoundObjectResult` | 404 | `return NotFound(message);` |
| `ConflictResult` | 409 | `return Conflict();` |
| `ConflictObjectResult` | 409 | `return Conflict(error);` |
| `StatusCodeResult` | Custom | `return StatusCode(500);` |
| `ObjectResult` | Custom | `return StatusCode(500, error);` |
| `FileResult` | 200 | `return File(bytes, contentType);` |
| `JsonResult` | 200 | `return Json(data);` |
| `RedirectResult` | 302 | `return Redirect(url);` |

**Usage Examples:**

```csharp
// Return data
[HttpGet]
public IActionResult GetUsers()
{
    var users = GetAllUsers();
    return Ok(users); // OkObjectResult
}

// Return without data
[HttpDelete("{id}")]
public IActionResult DeleteUser(int id)
{
    DeleteUserById(id);
    return NoContent(); // NoContentResult
}

// Return with location
[HttpPost]
public IActionResult CreateUser(User user)
{
    _context.Users.Add(user);
    _context.SaveChanges();
    
    return CreatedAtAction(
        nameof(GetUser),
        new { id = user.Id },
        user); // CreatedAtActionResult
}

// Return error
[HttpGet("{id}")]
public IActionResult GetUser(int id)
{
    var user = _context.Users.Find(id);
    
    if (user == null)
        return NotFound(new { message = $"User {id} not found" }); // NotFoundObjectResult
    
    return Ok(user);
}

// Return file
[HttpGet("{id}/photo")]
public IActionResult GetUserPhoto(int id)
{
    var bytes = GetPhotoBytes(id);
    return File(bytes, "image/jpeg"); // FileResult
}
```

---

### ActionResult<T> Class ⭐⭐⭐

**Purpose:** Generic action result that can return either T or IActionResult

**Namespace:** `Microsoft.AspNetCore.Mvc`

**Why use it?**
- Better OpenAPI/Swagger documentation
- Type-safe return values
- Implicit conversion from T

**Usage:**

```csharp
// ✅ Recommended
[HttpGet("{id}")]
public ActionResult<User> GetUser(int id)
{
    var user = _context.Users.Find(id);
    
    if (user == null)
        return NotFound(); // IActionResult
    
    return user; // Implicit conversion from User
    // or
    return Ok(user); // Explicit ActionResult
}

// ❌ Old way
[HttpGet("{id}")]
public IActionResult GetUser(int id)
{
    var user = _context.Users.Find(id);
    
    if (user == null)
        return NotFound();
    
    return Ok(user); // Must wrap in Ok()
}
```

**With async:**

```csharp
[HttpGet]
public async Task<ActionResult<IEnumerable<User>>> GetUsers()
{
    var users = await _context.Users.ToListAsync();
    return users; // Implicit conversion
}
```

---

### ModelStateDictionary Class ⭐⭐

**Purpose:** Contains validation errors

**Namespace:** `Microsoft.AspNetCore.Mvc.ModelBinding`

**Key Members:**

| Member | Type | Purpose |
|--------|------|---------|
| `IsValid` | bool | True if no errors |
| `ErrorCount` | int | Number of errors |
| `Keys` | IEnumerable<string> | Property names with errors |
| `Values` | IEnumerable<ModelStateEntry> | Validation entries |

**Usage:**

```csharp
[HttpPost]
public IActionResult CreateUser(User user)
{
    // Manual validation
    if (string.IsNullOrEmpty(user.Name))
    {
        ModelState.AddModelError(nameof(user.Name), "Name is required");
    }
    
    if (!ModelState.IsValid)
    {
        // Return validation errors
        return BadRequest(ModelState);
    }
    
    // Or get specific errors
    if (ModelState.ContainsKey(nameof(user.Email)))
    {
        var emailErrors = ModelState[nameof(user.Email)]
            .Errors
            .Select(e => e.ErrorMessage);
    }
    
    // Or format errors
    var errors = ModelState
        .Where(x => x.Value?.Errors.Count > 0)
        .ToDictionary(
            kvp => kvp.Key,
            kvp => kvp.Value?.Errors.Select(e => e.ErrorMessage).ToArray()
        );
    
    return BadRequest(new { errors });
}
```

---

### ProblemDetails Class ⭐⭐

**Purpose:** Standard error response format (RFC 7807)

**Namespace:** `Microsoft.AspNetCore.Mvc`

**Properties:**

| Property | Type | Purpose |
|----------|------|---------|
| `Type` | string | URI identifying problem type |
| `Title` | string | Short summary |
| `Status` | int? | HTTP status code |
| `Detail` | string | Detailed explanation |
| `Instance` | string | URI of specific occurrence |
| `Extensions` | IDictionary | Additional data |

**Usage:**

```csharp
[HttpGet("{id}")]
public IActionResult GetUser(int id)
{
    var user = _context.Users.Find(id);
    
    if (user == null)
    {
        return Problem(
            statusCode: StatusCodes.Status404NotFound,
            title: "User not found",
            detail: $"No user exists with ID {id}",
            instance: $"/api/users/{id}",
            type: "https://myapi.com/errors/not-found"
        );
    }
    
    return Ok(user);
}

// Custom problem details
[HttpPost]
public IActionResult CreateUser(User user)
{
    if (EmailExists(user.Email))
    {
        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status409Conflict,
            Title = "Email conflict",
            Detail = "Email already registered",
            Instance = HttpContext.Request.Path,
            Extensions =
            {
                ["email"] = user.Email,
                ["suggestedAction"] = "Use forgot password"
            }
        };
        
        return Conflict(problemDetails);
    }
    
    return Ok();
}
```

---

### ValidationProblemDetails Class ⭐⭐

**Purpose:** Problem details for validation errors

**Namespace:** `Microsoft.AspNetCore.Mvc`

**Extends:** `ProblemDetails`

**Additional Property:**

| Property | Type | Purpose |
|----------|------|---------|
| `Errors` | IDictionary<string, string[]> | Validation errors by field |

**Usage:**

```csharp
[HttpPost]
public IActionResult CreateUser(User user)
{
    if (!ModelState.IsValid)
    {
        var validationErrors = new ValidationProblemDetails(ModelState)
        {
            Status = StatusCodes.Status400BadRequest,
            Title = "Validation failed",
            Detail = "One or more validation errors occurred",
            Instance = HttpContext.Request.Path
        };
        
        return BadRequest(validationErrors);
    }
    
    return Ok();
}

// Response:
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "Validation failed",
  "status": 400,
  "detail": "One or more validation errors occurred",
  "instance": "/api/users",
  "errors": {
    "Name": ["Name is required"],
    "Email": ["Invalid email format"]
  }
}
```

---

## 12. Configuration Deep-Dive

### Pattern 1: Inline Configuration

**When to use:** Simple, hardcoded settings

```csharp
[HttpGet]
public IActionResult GetUsers([FromQuery] int page = 1, [FromQuery] int pageSize = 10)
{
    // Hardcoded limits
    if (pageSize > 100)
        pageSize = 100;
    
    if (page < 1)
        page = 1;
    
    var users = _context.Users
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToList();
    
    return Ok(users);
}
```

---

### Pattern 2: Constructor Options

**When to use:** Code-based configuration

**Step 1: Create Options Class**

```csharp
public class PaginationOptions
{
    public int DefaultPageSize { get; set; } = 10;
    public int MaxPageSize { get; set; } = 100;
}
```

**Step 2: Register**

```csharp
builder.Services.Configure<PaginationOptions>(options =>
{
    options.DefaultPageSize = 20;
    options.MaxPageSize = 200;
});
```

**Step 3: Inject**

```csharp
public class UsersController : ControllerBase
{
    private readonly PaginationOptions _options;
    
    public UsersController(IOptions<PaginationOptions> options)
    {
        _options = options.Value;
    }
    
    [HttpGet]
    public IActionResult GetUsers(int page = 1, int? pageSize = null)
    {
        var size = Math.Min(
            pageSize ?? _options.DefaultPageSize,
            _options.MaxPageSize);
        
        // Use size...
    }
}
```

---

### Pattern 3: IOptions from appsettings.json ⭐

**When to use:** Production configuration

**Step 1: appsettings.json**

```json
{
  "Pagination": {
    "DefaultPageSize": 10,
    "MaxPageSize": 100
  },
  "Api": {
    "Version": "1.0",
    "BaseUrl": "https://api.example.com"
  }
}
```

**Step 2: Options Classes**

```csharp
public class PaginationOptions
{
    public const string SectionName = "Pagination";
    
    public int DefaultPageSize { get; set; }
    public int MaxPageSize { get; set; }
}

public class ApiOptions
{
    public const string SectionName = "Api";
    
    public string Version { get; set; } = string.Empty;
    public string BaseUrl { get; set; } = string.Empty;
}
```

**Step 3: Register**

```csharp
builder.Services.Configure<PaginationOptions>(
    builder.Configuration.GetSection(PaginationOptions.SectionName));

builder.Services.Configure<ApiOptions>(
    builder.Configuration.GetSection(ApiOptions.SectionName));
```

**Step 4: Use**

```csharp
public class UsersController : ControllerBase
{
    private readonly PaginationOptions _paginationOptions;
    private readonly ApiOptions _apiOptions;
    
    public UsersController(
        IOptions<PaginationOptions> paginationOptions,
        IOptions<ApiOptions> apiOptions)
    {
        _paginationOptions = paginationOptions.Value;
        _apiOptions = apiOptions.Value;
    }
    
    [HttpGet]
    public IActionResult GetUsers()
    {
        var pageSize = _paginationOptions.DefaultPageSize;
        var baseUrl = _apiOptions.BaseUrl;
        // Use options...
    }
}
```

---

### IOptions vs IOptionsSnapshot vs IOptionsMonitor

| Feature | IOptions<T> | IOptionsSnapshot<T> | IOptionsMonitor<T> |
|---------|-------------|---------------------|---------------------|
| Lifetime | Singleton | Scoped | Singleton |
| Reload | No | Per request | Live |
| When to use | Static config | Per-request reload | Background services |

**Usage Example:**

```csharp
// IOptions - Singleton, no reload
public class Service1
{
    public Service1(IOptions<MyOptions> options)
    {
        var value = options.Value; // Loaded once
    }
}

// IOptionsSnapshot - Scoped, per-request reload
public class UsersController : ControllerBase
{
    public UsersController(IOptionsSnapshot<MyOptions> options)
    {
        var value = options.Value; // Reloaded per request
    }
}

// IOptionsMonitor - Singleton, live reload
public class BackgroundService
{
    private readonly IOptionsMonitor<MyOptions> _options;
    
    public BackgroundService(IOptionsMonitor<MyOptions> options)
    {
        _options = options;
        
        // React to changes
        _options.OnChange(newOptions =>
        {
            Console.WriteLine("Options changed!");
        });
    }
    
    public void DoWork()
    {
        var current = _options.CurrentValue; // Always current
    }
}
```

---

## 13. Advanced Topics

### Pagination Helper

```csharp
public class PagedResult<T>
{
    public IEnumerable<T> Items { get; set; } = Enumerable.Empty<T>();
    public int Page { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
    public bool HasPrevious => Page > 1;
    public bool HasNext => Page < TotalPages;
}

public static class QueryableExtensions
{
    public static async Task<PagedResult<T>> ToPagedResultAsync<T>(
        this IQueryable<T> query,
        int page,
        int pageSize)
    {
        var totalCount = await query.CountAsync();
        var items = await query
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
        
        return new PagedResult<T>
        {
            Items = items,
            Page = page,
            PageSize = pageSize,
            TotalCount = totalCount
        };
    }
}

// Usage
[HttpGet]
public async Task<ActionResult<PagedResult<UserResponse>>> GetUsers(
    int page = 1, int pageSize = 10)
{
    var result = await _context.Users
        .Select(u => new UserResponse { ... })
        .ToPagedResultAsync(page, pageSize);
    
    return Ok(result);
}
```

---

### HATEOAS (Hypermedia)

```csharp
public class UserHateoasResponse
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public List<Link> Links { get; set; } = new();
}

public class Link
{
    public string Href { get; set; } = string.Empty;
    public string Rel { get; set; } = string.Empty;
    public string Method { get; set; } = string.Empty;
}

[HttpGet("{id}")]
public ActionResult<UserHateoasResponse> GetUser(int id)
{
    var user = _context.Users.Find(id);
    if (user == null)
        return NotFound();
    
    var response = new UserHateoasResponse
    {
        Id = user.Id,
        Name = user.Name,
        Links = new List<Link>
        {
            new Link
            {
                Href = Url.Action(nameof(GetUser), new { id })!,
                Rel = "self",
                Method = "GET"
            },
            new Link
            {
                Href = Url.Action(nameof(UpdateUser), new { id })!,
                Rel = "update",
                Method = "PUT"
            },
            new Link
            {
                Href = Url.Action(nameof(DeleteUser), new { id })!,
                Rel = "delete",
                Method = "DELETE"
            }
        }
    };
    
    return Ok(response);
}
```

---

### Response Caching

```csharp
// In Program.cs
builder.Services.AddResponseCaching();

var app = builder.Build();
app.UseResponseCaching();

// In controller
[HttpGet]
[ResponseCache(Duration = 60)] // Cache for 60 seconds
public ActionResult<IEnumerable<User>> GetUsers()
{
    return Ok(_context.Users.ToList());
}

// No caching
[HttpGet("{id}")]
[ResponseCache(Location = ResponseCacheLocation.None, NoStore = true)]
public ActionResult<User> GetUser(int id)
{
    return Ok(_context.Users.Find(id));
}
```

---

### Rate Limiting (.NET 7+)

```csharp
using System.Threading.RateLimiting;

// In Program.cs
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("fixed", options =>
    {
        options.PermitLimit = 10;
        options.Window = TimeSpan.FromMinutes(1);
        options.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
        options.QueueLimit = 2;
    });
});

var app = builder.Build();
app.UseRateLimiter();

// In controller
[HttpGet]
[EnableRateLimiting("fixed")]
public ActionResult<IEnumerable<User>> GetUsers()
{
    return Ok(_context.Users.ToList());
}
```

---

## Summary: Complete API Development Checklist

**Planning:**
- [ ] Design RESTful URLs (nouns, plural)
- [ ] Choose approach (Minimal API vs Controllers)
- [ ] Plan versioning strategy
- [ ] Design DTOs (Request/Response)

**Implementation:**
- [ ] Create controllers with [ApiController]
- [ ] Implement CRUD operations
- [ ] Add validation (FluentValidation or Data Annotations)
- [ ] Create DTOs and mapping (AutoMapper)
- [ ] Add error handling (Global middleware + Problem Details)
- [ ] Add logging
- [ ] Add pagination for lists

**Documentation:**
- [ ] Add Swagger/OpenAPI
- [ ] Add XML documentation comments
- [ ] Add [ProducesResponseType] attributes

**Security:**
- [ ] Add authentication
- [ ] Add authorization ([Authorize])
- [ ] Enable CORS
- [ ] Add rate limiting

**Testing:**
- [ ] Test all endpoints
- [ ] Test validation
- [ ] Test error handling
- [ ] Load testing

**Production:**
- [ ] Configure for production
- [ ] Enable HTTPS
- [ ] Add response caching (where appropriate)
- [ ] Add health checks
- [ ] Monitor and log

---

**This completes the REST APIs, DTOs & Validation guide with practical examples and comprehensive technical reference!**