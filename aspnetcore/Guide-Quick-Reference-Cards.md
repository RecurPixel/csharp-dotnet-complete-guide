# ASP.NET Core Quick Reference Cards

---

## Card 1: Request Pipeline & Middleware Order

### Standard Middleware Pipeline Order

```
┌─────────────────────────────────────────────────┐
│ HTTP Request                                     │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 1. Exception Handler (Global Error Handling)    │
│    app.UseExceptionHandler()                     │
│    OR app.UseDeveloperExceptionPage() (Dev)     │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 2. HSTS (HTTP Strict Transport Security)        │
│    app.UseHsts()                                 │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 3. HTTPS Redirection                             │
│    app.UseHttpsRedirection()                     │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 4. Static Files (wwwroot)                        │
│    app.UseStaticFiles()                          │
│    ⚡ Short-circuits if file found              │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 5. Routing                                       │
│    app.UseRouting()                              │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 6. CORS (Cross-Origin Resource Sharing)         │
│    app.UseCors()                                 │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 7. Authentication (Who are you?)                 │
│    app.UseAuthentication()                       │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 8. Authorization (What can you do?)              │
│    app.UseAuthorization()                        │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 9. Session (if needed)                           │
│    app.UseSession()                              │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 10. Custom Middleware                            │
│     app.UseMyCustomMiddleware()                  │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 11. Response Caching                             │
│     app.UseResponseCaching()                     │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ 12. Endpoints (Controllers/Minimal APIs)         │
│     app.MapControllers()                         │
│     app.MapGet("/api/users", ...)                │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│ HTTP Response                                    │
└──────────────────────────────────────────────────┘
```

### Built-in Middleware Quick Reference

| Middleware | Method | When to Use | Position |
|------------|--------|-------------|----------|
| **Developer Exception Page** | `UseDeveloperExceptionPage()` | Development only | First |
| **Exception Handler** | `UseExceptionHandler("/error")` | Production | First |
| **HSTS** | `UseHsts()` | Production HTTPS | After exception |
| **HTTPS Redirection** | `UseHttpsRedirection()` | HTTP → HTTPS | Early |
| **Static Files** | `UseStaticFiles()` | Serve wwwroot files | Before routing |
| **Routing** | `UseRouting()` | Enable routing | Before auth |
| **CORS** | `UseCors()` | Cross-origin requests | After routing |
| **Authentication** | `UseAuthentication()` | Identify user | Before authorization |
| **Authorization** | `UseAuthorization()` | Check permissions | After authentication |
| **Session** | `UseSession()` | Enable sessions | After routing |
| **Response Compression** | `UseResponseCompression()` | Compress responses | Early |
| **Request Localization** | `UseRequestLocalization()` | i18n/l10n | After routing |
| **Response Caching** | `UseResponseCaching()` | Cache responses | Before endpoints |

### Common Middleware Mistakes

| ❌ Wrong | ✅ Correct | Issue |
|---------|-----------|-------|
| `UseAuthorization()` before `UseAuthentication()` | `UseAuthentication()` then `UseAuthorization()` | Can't authorize without identity |
| `UseRouting()` after `UseAuthorization()` | `UseRouting()` before `UseAuthorization()` | Routing must happen first |
| `UseCors()` after `UseAuthorization()` | `UseCors()` before `UseAuthorization()` | CORS needs to be early |
| `UseStaticFiles()` after `UseRouting()` | `UseStaticFiles()` before `UseRouting()` | Skip routing for static files |
| Custom middleware after endpoints | Custom middleware before endpoints | Can't intercept endpoint execution |

### Minimal Program.cs Template

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddDbContext<AppDbContext>();
// ... other services

var app = builder.Build();

// Configure pipeline (ORDER MATTERS!)
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Request/Response Flow

```
REQUEST PHASE (Top → Bottom)
┌─────────────────────────────┐
│ Exception Handler           │ ← Registers error handling
│ HTTPS Redirect              │ ← Redirects if needed
│ Static Files                │ ← Returns file if found
│ Routing                     │ ← Matches route
│ Authentication              │ ← Identifies user
│ Authorization               │ ← Checks permissions
│ Custom Middleware           │ ← Your logic
│ Endpoint                    │ ← Executes action
└─────────────────────────────┘

RESPONSE PHASE (Bottom → Top)
┌─────────────────────────────┐
│ Endpoint                    │ ← Generates response
│ Custom Middleware           │ ← Modifies response
│ Authorization               │ ←
│ Authentication              │ ←
│ Routing                     │ ←
│ Static Files                │ ←
│ HTTPS Redirect              │ ←
│ Exception Handler           │ ← Handles errors
└─────────────────────────────┘
```

### Key Principles

1. **Exception handling first** - Catch all errors
2. **Security early** - HTTPS, CORS, Auth
3. **Static files before routing** - Performance
4. **Auth before authorization** - Identity first
5. **Custom middleware before endpoints** - Intercept requests
6. **Endpoints last** - Process the request

---

## Card 2: Service Lifetimes Quick Reference

### Service Lifetime Types

```
┌─────────────────────────────────────────────────┐
│ TRANSIENT                                       │
│ • New instance EVERY TIME requested             │
│ • Lightweight, stateless services               │
│ • No shared state                               │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ SCOPED                                          │
│ • One instance PER HTTP REQUEST                 │
│ • Shared within same request                    │
│ • Disposed at end of request                    │
│ • Most common for DbContext                     │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ SINGLETON                                       │
│ • One instance FOR ENTIRE APPLICATION           │
│ • Created once, never disposed                  │
│ • Must be thread-safe                           │
│ • Memory efficient                              │
└─────────────────────────────────────────────────┘
```

### Registration Methods

```csharp
// TRANSIENT - New instance every time
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddTransient<EmailService>();

// SCOPED - One per request
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<AppDbContext>();

// SINGLETON - One for application lifetime
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
builder.Services.AddSingleton<IConfiguration>(Configuration);

// With implementation factory
builder.Services.AddTransient<IService>(sp => 
    new Service(sp.GetRequiredService<IDependency>()));

// With instance
var settings = new Settings();
builder.Services.AddSingleton(settings);
```

### Visual Lifetime Comparison

```
REQUEST 1:
┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│ Transient #1   │  │ Scoped #1      │  │ Singleton #1   │
└────────────────┘  └────────────────┘  └────────────────┘
┌────────────────┐        ↑                     ↑
│ Transient #2   │        │                     │
└────────────────┘        │                     │
                          │                     │
REQUEST 2:                │                     │
┌────────────────┐  ┌────────────────┐          │
│ Transient #3   │  │ Scoped #2      │          │
└────────────────┘  └────────────────┘          │
┌────────────────┐        ↑                     │
│ Transient #4   │        │                     │
└────────────────┘        │                     │
                          │                     │
REQUEST 3:                │                     │
┌────────────────┐  ┌────────────────┐          │
│ Transient #5   │  │ Scoped #3      │          │
└────────────────┘  └────────────────┘  Same instance
                                        used across
                                        all requests
```

### When to Use Which

| Service Type | Use Transient | Use Scoped | Use Singleton |
|--------------|---------------|------------|---------------|
| **DbContext** | ❌ Never | ✅ Always | ❌ Never |
| **Repository** | ❌ Rarely | ✅ Usually | ❌ Rarely |
| **Email Service** | ✅ Yes | ⚠️ Maybe | ❌ No |
| **Cache Service** | ❌ No | ❌ No | ✅ Yes |
| **Configuration** | ❌ No | ❌ No | ✅ Yes |
| **Logger** | ❌ No | ❌ No | ✅ Yes |
| **HTTP Client** | ❌ No | ❌ No | ✅ Yes* |
| **Validators** | ✅ Yes | ⚠️ Maybe | ❌ No |

*Use `IHttpClientFactory` instead

### Common Service Patterns

```csharp
// Stateless service - TRANSIENT
public class EmailService : IEmailService
{
    public Task SendAsync(string to, string subject, string body)
    {
        // No state, create new each time
    }
}
builder.Services.AddTransient<IEmailService, EmailService>();

// Database access - SCOPED
public class UserRepository : IUserRepository
{
    private readonly AppDbContext _context;
    
    public UserRepository(AppDbContext context)
    {
        _context = context; // Scoped DbContext
    }
}
builder.Services.AddScoped<IUserRepository, UserRepository>();

// Application-wide cache - SINGLETON
public class CacheService : ICacheService
{
    private readonly ConcurrentDictionary<string, object> _cache = new();
    
    public void Set(string key, object value)
    {
        _cache[key] = value; // Thread-safe
    }
}
builder.Services.AddSingleton<ICacheService, CacheService>();
```

### Dependency Rules (Captive Dependencies)

```
✅ ALLOWED:
Singleton    → Singleton
Scoped       → Singleton
Scoped       → Scoped
Transient    → Singleton
Transient    → Scoped
Transient    → Transient

❌ FORBIDDEN:
Singleton    → Scoped      (CAPTIVE DEPENDENCY!)
Singleton    → Transient   (CAPTIVE DEPENDENCY!)
Scoped       → Transient   (Usually OK, but wasteful)
```

**Captive Dependency Example:**

```csharp
// ❌ BAD - Singleton capturing Scoped
public class MySingleton
{
    private readonly AppDbContext _context; // SCOPED!
    
    public MySingleton(AppDbContext context) // ❌ WRONG!
    {
        _context = context;
    }
}

// ✅ GOOD - Use IServiceProvider
public class MySingleton
{
    private readonly IServiceProvider _serviceProvider;
    
    public MySingleton(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public void DoWork()
    {
        using var scope = _serviceProvider.CreateScope();
        var context = scope.ServiceProvider
            .GetRequiredService<AppDbContext>();
        // Use context...
    }
}
```

### Decision Tree

```
Is service stateless?
├─ YES → Does it use DbContext?
│        ├─ YES → SCOPED
│        └─ NO  → TRANSIENT
│
└─ NO  → Does it need to be shared?
         ├─ YES → SINGLETON (must be thread-safe)
         └─ NO  → SCOPED
```

### Common Registrations

```csharp
// Entity Framework
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
// ↑ Registers as SCOPED

// Identity
builder.Services.AddIdentity<User, Role>()
    .AddEntityFrameworkStores<AppDbContext>();
// ↑ Registers UserManager, SignInManager as SCOPED

// HTTP Client
builder.Services.AddHttpClient<IApiClient, ApiClient>();
// ↑ Registers IHttpClientFactory as SINGLETON
// ↑ Registers ApiClient as TRANSIENT

// AutoMapper
builder.Services.AddAutoMapper(typeof(Program));
// ↑ Registers IMapper as SINGLETON

// Memory Cache
builder.Services.AddMemoryCache();
// ↑ Registers IMemoryCache as SINGLETON

// Distributed Cache
builder.Services.AddDistributedMemoryCache();
// ↑ Registers IDistributedCache as SINGLETON
```

### Tips

1. **Default to Scoped** for most services
2. **Use Transient** for lightweight, stateless services
3. **Use Singleton** only if thread-safe and stateless
4. **Never inject Scoped into Singleton** (captive dependency)
5. **DbContext is always Scoped**
6. **Configuration is always Singleton**

---

## Card 3: HTTP Methods & Status Codes

### HTTP Methods (Verbs)

| Method | Purpose | Body | Idempotent | Safe | Cacheable |
|--------|---------|------|------------|------|-----------|
| **GET** | Retrieve resource | No | ✅ Yes | ✅ Yes | ✅ Yes |
| **POST** | Create resource | ✅ Yes | ❌ No | ❌ No | ⚠️ Rarely |
| **PUT** | Replace resource | ✅ Yes | ✅ Yes | ❌ No | ❌ No |
| **PATCH** | Update partial | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **DELETE** | Remove resource | No* | ✅ Yes | ❌ No | ❌ No |
| **HEAD** | Get headers only | No | ✅ Yes | ✅ Yes | ✅ Yes |
| **OPTIONS** | Get allowed methods | No | ✅ Yes | ✅ Yes | ❌ No |

*Can have body but rarely used

### HTTP Status Codes

#### 2xx Success

| Code | Name | When to Use | Action Result |
|------|------|-------------|---------------|
| **200** | OK | Successful GET, PUT, PATCH | `Ok(data)` |
| **201** | Created | Resource created (POST) | `Created(uri, data)` |
| **202** | Accepted | Async processing started | `Accepted()` |
| **204** | No Content | Successful DELETE, PUT | `NoContent()` |

```csharp
// 200 OK
[HttpGet("{id}")]
public ActionResult<User> GetUser(int id)
    => Ok(user);

// 201 Created
[HttpPost]
public ActionResult<User> CreateUser(User user)
    => CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);

// 204 No Content
[HttpDelete("{id}")]
public IActionResult DeleteUser(int id)
    => NoContent();
```

#### 3xx Redirection

| Code | Name | When to Use | Action Result |
|------|------|-------------|---------------|
| **301** | Moved Permanently | Resource moved forever | `RedirectPermanent(url)` |
| **302** | Found | Temporary redirect | `Redirect(url)` |
| **304** | Not Modified | Cached version valid | - |

#### 4xx Client Errors

| Code | Name | When to Use | Action Result |
|------|------|-------------|---------------|
| **400** | Bad Request | Invalid input/validation failed | `BadRequest(error)` |
| **401** | Unauthorized | Authentication required | `Unauthorized()` |
| **403** | Forbidden | Authenticated but not authorized | `Forbid()` |
| **404** | Not Found | Resource doesn't exist | `NotFound(message)` |
| **405** | Method Not Allowed | Wrong HTTP method | - |
| **409** | Conflict | Resource conflict (duplicate) | `Conflict(error)` |
| **415** | Unsupported Media Type | Wrong Content-Type | - |
| **422** | Unprocessable Entity | Validation failed (semantic) | - |
| **429** | Too Many Requests | Rate limit exceeded | `StatusCode(429)` |

```csharp
// 400 Bad Request
if (!ModelState.IsValid)
    return BadRequest(ModelState);

// 401 Unauthorized
if (!User.Identity.IsAuthenticated)
    return Unauthorized();

// 403 Forbidden
if (!User.IsInRole("Admin"))
    return Forbid();

// 404 Not Found
if (user == null)
    return NotFound(new { message = "User not found" });

// 409 Conflict
if (EmailExists(user.Email))
    return Conflict(new { message = "Email already exists" });
```

#### 5xx Server Errors

| Code | Name | When to Use | Action Result |
|------|------|-------------|---------------|
| **500** | Internal Server Error | Unhandled exception | `StatusCode(500, error)` |
| **501** | Not Implemented | Feature not implemented | `StatusCode(501)` |
| **502** | Bad Gateway | Invalid proxy response | - |
| **503** | Service Unavailable | Service temporarily down | `StatusCode(503)` |
| **504** | Gateway Timeout | Proxy timeout | - |

### RESTful API Patterns

```csharp
// Collection resource
GET    /api/users           → 200 (list)
POST   /api/users           → 201 (created)

// Single resource
GET    /api/users/5         → 200 (user) or 404
PUT    /api/users/5         → 200 or 204 or 404
PATCH  /api/users/5         → 200 or 204 or 404
DELETE /api/users/5         → 204 or 404

// Sub-resource
GET    /api/users/5/orders  → 200 (user's orders)
POST   /api/users/5/orders  → 201 (new order)

// Filtering, sorting, pagination
GET    /api/users?role=admin&sort=name&page=2&pageSize=10
```

### Complete CRUD Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    // GET /api/users → 200
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public ActionResult<IEnumerable<User>> GetUsers()
        => Ok(users);
    
    // GET /api/users/5 → 200 or 404
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public ActionResult<User> GetUser(int id)
        => user == null ? NotFound() : Ok(user);
    
    // POST /api/users → 201 or 400
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public ActionResult<User> CreateUser(User user)
        => CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    
    // PUT /api/users/5 → 204 or 404 or 400
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public IActionResult UpdateUser(int id, User user)
    {
        if (id != user.Id) return BadRequest();
        if (!Exists(id)) return NotFound();
        // Update...
        return NoContent();
    }
    
    // DELETE /api/users/5 → 204 or 404
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public IActionResult DeleteUser(int id)
    {
        if (!Exists(id)) return NotFound();
        // Delete...
        return NoContent();
    }
}
```

### Status Code Decision Tree

```
Was the request successful?
├─ YES → Did it create a resource?
│        ├─ YES → 201 Created
│        └─ NO  → Is there response data?
│                ├─ YES → 200 OK
│                └─ NO  → 204 No Content
│
└─ NO  → Who caused the problem?
         ├─ CLIENT → What's wrong?
         │          ├─ Invalid data → 400 Bad Request
         │          ├─ Not authenticated → 401 Unauthorized
         │          ├─ Not authorized → 403 Forbidden
         │          ├─ Not found → 404 Not Found
         │          └─ Duplicate → 409 Conflict
         │
         └─ SERVER → 500 Internal Server Error
```

### Common Mistakes

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| `return Ok("Not found")` | `return NotFound()` |
| `return Ok(error)` for errors | `return BadRequest(error)` |
| POST returns 200 | POST returns 201 |
| DELETE returns 200 with data | DELETE returns 204 |
| 404 with body describing error | 404 with simple message |
| Always returning 200 | Use appropriate status codes |

### Quick Tips

- **GET** = Retrieve (200 or 404)
- **POST** = Create (201 + Location header)
- **PUT** = Replace entire resource (204 or 200)
- **PATCH** = Update partial (204 or 200)
- **DELETE** = Remove (204)
- **4xx** = Client's fault
- **5xx** = Server's fault

---

## Card 4: EF Core Quick Reference

### DbContext Essentials

```csharp
// Basic DbContext
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options) { }
    
    // DbSets (tables)
    public DbSet<User> Users => Set<User>();
    public DbSet<Order> Orders => Set<Order>();
    
    // Configure relationships
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Fluent API configuration
        modelBuilder.Entity<User>(entity =>
        {
            entity.HasKey(u => u.Id);
            entity.Property(u => u.Name).IsRequired().HasMaxLength(100);
            entity.HasIndex(u => u.Email).IsUnique();
        });
        
        // Relationships
        modelBuilder.Entity<Order>()
            .HasOne(o => o.User)
            .WithMany(u => u.Orders)
            .HasForeignKey(o => o.UserId);
    }
}
```

### Registration & Connection String

```csharp
// In Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=MyDb;Trusted_Connection=True;"
  }
}
```

### CRUD Operations

```csharp
public class UserService
{
    private readonly AppDbContext _context;
    
    public UserService(AppDbContext context)
    {
        _context = context;
    }
    
    // CREATE
    public async Task<User> CreateAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return user;
    }
    
    // READ - Single
    public async Task<User?> GetByIdAsync(int id)
    {
        return await _context.Users.FindAsync(id);
        // or
        return await _context.Users
            .FirstOrDefaultAsync(u => u.Id == id);
    }
    
    // READ - Multiple
    public async Task<List<User>> GetAllAsync()
    {
        return await _context.Users.ToListAsync();
    }
    
    // READ - With filter
    public async Task<List<User>> GetActiveUsersAsync()
    {
        return await _context.Users
            .Where(u => u.IsActive)
            .ToListAsync();
    }
    
    // READ - With related data
    public async Task<User?> GetWithOrdersAsync(int id)
    {
        return await _context.Users
            .Include(u => u.Orders)
            .FirstOrDefaultAsync(u => u.Id == id);
    }
    
    // UPDATE
    public async Task UpdateAsync(User user)
    {
        _context.Users.Update(user);
        await _context.SaveChangesAsync();
    }
    
    // UPDATE - Partial
    public async Task UpdateEmailAsync(int id, string newEmail)
    {
        var user = await _context.Users.FindAsync(id);
        if (user != null)
        {
            user.Email = newEmail;
            await _context.SaveChangesAsync();
        }
    }
    
    // DELETE
    public async Task DeleteAsync(int id)
    {
        var user = await _context.Users.FindAsync(id);
        if (user != null)
        {
            _context.Users.Remove(user);
            await _context.SaveChangesAsync();
        }
    }
}
```

### Common Query Patterns

```csharp
// Filtering
var adults = await _context.Users
    .Where(u => u.Age >= 18)
    .ToListAsync();

// Sorting
var sorted = await _context.Users
    .OrderBy(u => u.Name)
    .ToListAsync();

// Pagination
var page = await _context.Users
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();

// Projection (Select specific columns)
var names = await _context.Users
    .Select(u => new { u.Id, u.Name })
    .ToListAsync();

// Count
var count = await _context.Users.CountAsync();
var activeCount = await _context.Users
    .CountAsync(u => u.IsActive);

// Any/All
var hasUsers = await _context.Users.AnyAsync();
var allActive = await _context.Users.AllAsync(u => u.IsActive);

// First/Single
var first = await _context.Users.FirstAsync();
var firstOrNull = await _context.Users.FirstOrDefaultAsync();
var single = await _context.Users.SingleAsync(u => u.Id == 1);

// GroupBy
var grouped = await _context.Orders
    .GroupBy(o => o.UserId)
    .Select(g => new { UserId = g.Key, Count = g.Count() })
    .ToListAsync();

// Include (Eager loading)
var usersWithOrders = await _context.Users
    .Include(u => u.Orders)
    .ToListAsync();

// Multiple includes
var data = await _context.Users
    .Include(u => u.Orders)
    .Include(u => u.Address)
    .ToListAsync();

// Nested include
var full = await _context.Users
    .Include(u => u.Orders)
        .ThenInclude(o => o.Items)
    .ToListAsync();

// AsNoTracking (read-only, faster)
var users = await _context.Users
    .AsNoTracking()
    .ToListAsync();
```

### Migrations Commands

```bash
# .NET CLI
dotnet ef migrations add InitialCreate
dotnet ef database update
dotnet ef migrations remove
dotnet ef database drop
dotnet ef migrations list

# Package Manager Console (Visual Studio)
Add-Migration InitialCreate
Update-Database
Remove-Migration
Drop-Database
Get-Migration
```

### Relationships

```csharp
// One-to-Many
public class User
{
    public int Id { get; set; }
    public List<Order> Orders { get; set; } = new();
}

public class Order
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public User User { get; set; } = null!;
}

// Configuration
modelBuilder.Entity<Order>()
    .HasOne(o => o.User)
    .WithMany(u => u.Orders)
    .HasForeignKey(o => o.UserId);// One-to-One
    
public class User
{
    public int Id { get; set; }
    public Profile Profile { get; set; } = null!;
}

public class Profile
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public User User { get; set; } = null!;
}

// Configuration
modelBuilder.Entity<Profile>()
    .HasOne(p => p.User)
    .WithOne(u => u.Profile)
    .HasForeignKey<Profile>(p => p.UserId);

// Many-to-Many (.NET 5+)
public class Student
{
    public int Id { get; set; }
    public List<Course> Courses { get; set; } = new();
}

public class Course
{
    public int Id { get; set; }
    public List<Student> Students { get; set; } = new();
}

// No explicit configuration needed!
// EF Core creates join table automatically
```

### Performance Tips

```csharp
// ✅ Use AsNoTracking for read-only queries
var users = await _context.Users
    .AsNoTracking()
    .ToListAsync();

// ✅ Select only needed columns
var names = await _context.Users
    .Select(u => new { u.Id, u.Name })
    .ToListAsync();

// ✅ Use pagination
var page = await _context.Users
    .Skip(skip)
    .Take(take)
    .ToListAsync();

// ✅ Filter early (in SQL, not memory)
var adults = await _context.Users
    .Where(u => u.Age >= 18) // SQL WHERE clause
    .ToListAsync();

// ❌ Don't filter after ToList
var all = await _context.Users.ToListAsync();
var adults = all.Where(u => u.Age >= 18); // In-memory filter

// ✅ Use Include for related data
var users = await _context.Users
    .Include(u => u.Orders)
    .ToListAsync();

// ❌ Don't use lazy loading in loops (N+1 problem)
var users = await _context.Users.ToListAsync();
foreach (var user in users)
{
    var orders = user.Orders; // N+1 queries!
}
```

### Common Patterns

```csharp
// Repository Pattern
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;
    private readonly DbSet<T> _dbSet;
    
    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T?> GetByIdAsync(int id)
        => await _dbSet.FindAsync(id);
    
    public async Task<IEnumerable<T>> GetAllAsync()
        => await _dbSet.ToListAsync();
    
    public async Task<T> AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
        return entity;
    }
    
    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }
    
    public async Task DeleteAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}
```

---

## Card 5: Validation Attributes

### Built-in Validation Attributes

```csharp
using System.ComponentModel.DataAnnotations;

public class CreateUserRequest
{
    // Required
    [Required(ErrorMessage = "Name is required")]
    public string Name { get; set; }
    
    // String length
    [StringLength(100, MinimumLength = 2)]
    public string Username { get; set; }
    
    // Email format
    [EmailAddress]
    public string Email { get; set; }
    
    // Regular expression
    [RegularExpression(@"^[a-zA-Z\s]+$", 
        ErrorMessage = "Only letters allowed")]
    public string FullName { get; set; }
    
    // Range
    [Range(18, 120)]
    public int Age { get; set; }
    
    // Comparison
    [Compare("Password")]
    public string ConfirmPassword { get; set; }
    
    // URL format
    [Url]
    public string Website { get; set; }
    
    // Phone format
    [Phone]
    public string PhoneNumber { get; set; }
    
    // Credit card
    [CreditCard]
    public string CreditCardNumber { get; set; }
    
    // Min/Max length
    [MinLength(10)]
    [MaxLength(500)]
    public string Description { get; set; }
}
```

### Complete Validation Attributes Reference

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[Required]` | Field must have value | `[Required]` |
| `[StringLength]` | String length limits | `[StringLength(100, MinimumLength = 2)]` |
| `[MinLength]` | Minimum length | `[MinLength(5)]` |
| `[MaxLength]` | Maximum length | `[MaxLength(100)]` |
| `[Range]` | Numeric range | `[Range(1, 100)]` |
| `[EmailAddress]` | Valid email | `[EmailAddress]` |
| `[Phone]` | Valid phone | `[Phone]` |
| `[Url]` | Valid URL | `[Url]` |
| `[CreditCard]` | Valid credit card | `[CreditCard]` |
| `[Compare]` | Match another property | `[Compare("Password")]` |
| `[RegularExpression]` | Match regex pattern | `[RegularExpression(@"^\d{5}$")]` |
| `[DataType]` | Specify data type | `[DataType(DataType.Date)]` |
| `[Display]` | Display name | `[Display(Name = "Full Name")]` |

### Common Regex Patterns

```csharp
// Password (min 8, upper, lower, number, special)
[RegularExpression(@"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$")]
public string Password { get; set; }

// Username (alphanumeric, underscore, 3-20 chars)
[RegularExpression(@"^[a-zA-Z0-9_]{3,20}$")]
public string Username { get; set; }

// ZIP code (US 5 digits)
[RegularExpression(@"^\d{5}$")]
public string ZipCode { get; set; }

// Phone (US format)
[RegularExpression(@"^\(\d{3}\)\s\d{3}-\d{4}$")]
public string Phone { get; set; } // (123) 456-7890

// Only letters and spaces
[RegularExpression(@"^[a-zA-Z\s]+$")]
public string Name { get; set; }

// Alphanumeric only
[RegularExpression(@"^[a-zA-Z0-9]+$")]
public string Code { get; set; }
```

### Custom Validation Attribute

```csharp
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

// Usage
public class CreateEventRequest
{
    [FutureDate(ErrorMessage = "Event date must be in the future")]
    public DateTime EventDate { get; set; }
}
```

### FluentValidation Quick Start

```bash
# Install
dotnet add package FluentValidation.AspNetCore
```

```csharp
// Validator
public class CreateUserRequestValidator 
    : AbstractValidator<CreateUserRequest>
{
    public CreateUserRequestValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty()
            .Length(2, 100);
        
        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress();
        
        RuleFor(x => x.Age)
            .InclusiveBetween(18, 120);
        
        RuleFor(x => x.Password)
            .NotEmpty()
            .MinimumLength(8)
            .Matches(@"[A-Z]").WithMessage("Must contain uppercase")
            .Matches(@"[a-z]").WithMessage("Must contain lowercase")
            .Matches(@"\d").WithMessage("Must contain number");
    }
}

// Register
builder.Services.AddValidatorsFromAssemblyContaining<Program>();
builder.Services.AddFluentValidationAutoValidation();

// Use (automatic validation with [ApiController])
[HttpPost]
public IActionResult CreateUser(CreateUserRequest request)
{
    // Validation already done
    return Ok();
}
```

### Model State Handling

```csharp
[HttpPost]
public IActionResult CreateUser(User user)
{
    // Check if valid
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    
    // Manual validation
    if (EmailExists(user.Email))
    {
        ModelState.AddModelError(nameof(user.Email), 
            "Email already exists");
        return BadRequest(ModelState);
    }
    
    // Get errors
    var errors = ModelState
        .Where(x => x.Value?.Errors.Count > 0)
        .ToDictionary(
            kvp => kvp.Key,
            kvp => kvp.Value?.Errors
                .Select(e => e.ErrorMessage)
                .ToArray()
        );
    
    return Ok();
}
```

### Validation Error Response

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "00-abc123...",
  "errors": {
    "Name": [
      "Name is required",
      "Name must be between 2 and 100 characters"
    ],
    "Email": [
      "Invalid email format"
    ],
    "Age": [
      "Age must be between 18 and 120"
    ]
  }
}
```

---

## Card 6: Authentication & Authorization

### Authentication vs Authorization

```
┌─────────────────────────────────────────────┐
│ AUTHENTICATION                              │
│ "Who are you?"                              │
│ • Verify identity                           │
│ • Login process                             │
│ • Credentials check                         │
│ • Returns: User identity (ClaimsPrincipal)  │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ AUTHORIZATION                               │
│ "What can you do?"                          │
│ • Check permissions                         │
│ • Role-based access                         │
│ • Policy-based rules                        │
│ • Returns: Allow or Deny                    │
└─────────────────────────────────────────────┘
```

### JWT Authentication Setup

```csharp
// 1. Install package
// dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer

// 2. appsettings.json
{
  "Jwt": {
    "Key": "YourSuperSecretKeyThatIsAtLeast32CharactersLong",
    "Issuer": "YourApp",
    "Audience": "YourAppUsers",
    "ExpireMinutes": 60
  }
}

// 3. Configure services (Program.cs)
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });

// 4. Add to pipeline
app.UseAuthentication(); // ⚠️ Before UseAuthorization
app.UseAuthorization();

// 5. Generate token
public string GenerateJwtToken(User user)
{
    var securityKey = new SymmetricSecurityKey(
        Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]!));
    var credentials = new SigningCredentials(
        securityKey, SecurityAlgorithms.HmacSha256);
    
    var claims = new[]
    {
        new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new Claim(ClaimTypes.Name, user.Username),
        new Claim(ClaimTypes.Email, user.Email),
        new Claim(ClaimTypes.Role, user.Role)
    };
    
    var token = new JwtSecurityToken(
        issuer: _configuration["Jwt:Issuer"],
        audience: _configuration["Jwt:Audience"],
        claims: claims,
        expires: DateTime.Now.AddMinutes(
            int.Parse(_configuration["Jwt:ExpireMinutes"]!)),
        signingCredentials: credentials);
    
    return new JwtSecurityTokenHandler().WriteToken(token);
}

// 6. Login endpoint
[HttpPost("login")]
public IActionResult Login(LoginRequest request)
{
    var user = ValidateCredentials(request.Username, request.Password);
    
    if (user == null)
        return Unauthorized();
    
    var token = GenerateJwtToken(user);
    
    return Ok(new { token });
}

// 7. Protect endpoints
[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile()
{
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    // Return user data
}
```

### Authorization Attributes

```csharp
// Require authentication
[Authorize]
public class UsersController : ControllerBase { }

// Allow anonymous access
[AllowAnonymous]
[HttpGet("public")]
public IActionResult GetPublicData() { }

// Require specific role
[Authorize(Roles = "Admin")]
[HttpDelete("{id}")]
public IActionResult DeleteUser(int id) { }

// Require multiple roles (OR)
[Authorize(Roles = "Admin,Manager")]
public IActionResult ManageUsers() { }

// Require multiple roles (AND) - use multiple attributes
[Authorize(Roles = "Admin")]
[Authorize(Roles = "Manager")]
public IActionResult SpecialAction() { }

// Require policy
[Authorize(Policy = "MustBeOver18")]
public IActionResult RestrictedContent() { }
```

### Role-Based Authorization

```csharp
// Configure roles
var user = User.FindFirst(ClaimTypes.Role)?.Value;

// Check in code
if (User.IsInRole("Admin"))
{
    // Admin only
}

// Multiple roles
if (User.IsInRole("Admin") || User.IsInRole("Manager"))
{
    // Admin or Manager
}
```

### Claims-Based Authorization

```csharp
// Add claims when creating token
var claims = new[]
{
    new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
    new Claim(ClaimTypes.Name, user.Username),
    new Claim(ClaimTypes.Email, user.Email),
    new Claim(ClaimTypes.Role, user.Role),
    new Claim("EmployeeNumber", user.EmployeeNumber),
    new Claim("Department", user.Department)
};

// Check claims in controller
var employeeNumber = User.FindFirst("EmployeeNumber")?.Value;
var department = User.FindFirst("Department")?.Value;

// Check if claim exists
if (User.HasClaim(c => c.Type == "Department" && c.Value == "IT"))
{
    // IT department only
}
```

### Policy-Based Authorization ⭐

```csharp
// 1. Define policies (Program.cs)
builder.Services.AddAuthorization(options =>
{
    // Simple policy
    options.AddPolicy("MustBeOver18", policy =>
        policy.RequireClaim("Age", "18", "19", "20")); // etc.
    
    // Role-based policy
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
    
    // Multiple requirements (AND)
    options.AddPolicy("SeniorAdmin", policy =>
    {
        policy.RequireRole("Admin");
        policy.RequireClaim("YearsOfService", "5");
    });
    
    // Custom requirement
    options.AddPolicy("CanEditUser", policy =>
        policy.Requirements.Add(new CanEditUserRequirement()));
});

// 2. Use policy
[Authorize(Policy = "MustBeOver18")]
public IActionResult ViewContent() { }

[Authorize(Policy = "AdminOnly")]
public IActionResult AdminPanel() { }
```

### Custom Authorization Requirement

```csharp
// Requirement
public class CanEditUserRequirement : IAuthorizationRequirement { }

// Handler
public class CanEditUserHandler 
    : AuthorizationHandler<CanEditUserRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        CanEditUserRequirement requirement)
    {
        var userId = context.User
            .FindFirst(ClaimTypes.NameIdentifier)?.Value;
        
        // Check if user is admin or editing their own profile
        if (context.User.IsInRole("Admin") || 
            context.Resource is int targetUserId && 
            userId == targetUserId.ToString())
        {
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}

// Register
builder.Services.AddSingleton<IAuthorizationHandler, CanEditUserHandler>();
```

### Common Auth Patterns

```csharp
// Get current user ID
var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

// Get username
var username = User.Identity?.Name;

// Check if authenticated
if (User?.Identity?.IsAuthenticated == true)
{
    // User is logged in
}

// Get all claims
var claims = User.Claims.Select(c => new
{
    Type = c.Type,
    Value = c.Value
});

// Get specific claim
var email = User.FindFirst(ClaimTypes.Email)?.Value;
var role = User.FindFirst(ClaimTypes.Role)?.Value;
```

### Cookie Authentication (Alternative to JWT)

```csharp
// Configure
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/login";
        options.LogoutPath = "/logout";
        options.ExpireTimeSpan = TimeSpan.FromDays(14);
        options.SlidingExpiration = true;
    });

// Sign in
var claims = new List<Claim>
{
    new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
    new Claim(ClaimTypes.Name, user.Username),
    new Claim(ClaimTypes.Role, user.Role)
};

var claimsIdentity = new ClaimsIdentity(claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity));

// Sign out
await HttpContext.SignOutAsync(
    CookieAuthenticationDefaults.AuthenticationScheme);
```

### Quick Decision Tree

```
What type of app?
├─ Web API (SPA/Mobile) → JWT
└─ Traditional Web App → Cookies

What authorization model?
├─ Simple roles → Role-based
├─ Complex permissions → Policy-based
└─ Fine-grained → Claims + Policies
```

---

**These 6 quick reference cards cover the most essential ASP.NET Core concepts for quick desk-side reference!**