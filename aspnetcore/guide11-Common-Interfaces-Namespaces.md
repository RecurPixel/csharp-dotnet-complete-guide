# ASP.NET Core Common Interfaces, Classes & Namespaces - Complete Reference
## Quick Reference Guide + Technical Documentation

---

## 📋 Table of Contents

### Part 1: Core Types Quick Reference (Organized by Category)
1. Core HTTP Types (HttpContext, HttpRequest, HttpResponse)
2. Middleware Types (IMiddleware, RequestDelegate, IApplicationBuilder)
3. Dependency Injection (IServiceCollection, IServiceProvider, Lifetimes)
4. Configuration (IConfiguration, IOptions, IOptionsSnapshot, IOptionsMonitor)
5. Database (DbContext, DbSet, ModelBuilder)
6. Controllers & Routing (ControllerBase, IActionResult, RouteData)
7. Authentication & Authorization (ClaimsPrincipal, UserManager, SignInManager)
8. Logging (ILogger, LogLevel, ILoggerFactory)
9. Filters (IActionFilter, IExceptionFilter, IAuthorizationFilter)
10. Hosting & Application (WebApplication, WebApplicationBuilder, IWebHostEnvironment)

### Part 2: Namespace Reference
11. Essential Namespaces by Category

### Part 3: Common Attributes Reference
12. Attributes by Category (Routing, Validation, API, Authorization, Filters)

### Part 4: Quick Lookup Tables
13. HTTP Status Codes - Complete Reference
14. Content Types - Common MIME Types
15. Extension Methods - Commonly Used Extensions
16. Common Patterns - Code Snippets for Frequent Tasks

---

# PART 1: CORE TYPES QUICK REFERENCE

---

## 1. Core HTTP Types

### HttpContext ⭐⭐⭐

**Purpose:** Encapsulates all HTTP request/response information for a single HTTP request

**Namespace:** `Microsoft.AspNetCore.Http`

**Declaration:**
```csharp
public abstract class HttpContext
```

**Key Members:**

| Member | Type | Purpose | Example |
|--------|------|---------|---------|
| `Request` | HttpRequest | Incoming HTTP request | `context.Request.Path` |
| `Response` | HttpResponse | Outgoing HTTP response | `context.Response.StatusCode = 200` |
| `User` | ClaimsPrincipal | Authenticated user | `context.User.Identity.Name` |
| `Items` | IDictionary<object, object> | Request-scoped data storage | `context.Items["Key"] = value` |
| `RequestServices` | IServiceProvider | Access DI services | `context.RequestServices.GetService<T>()` |
| `Connection` | ConnectionInfo | Connection details | `context.Connection.RemoteIpAddress` |
| `RequestAborted` | CancellationToken | Request cancellation | `await Task.Delay(1000, context.RequestAborted)` |
| `Session` | ISession | Session data | `context.Session.SetString("key", "value")` |
| `TraceIdentifier` | string | Unique request ID | `context.TraceIdentifier` |
| `Features` | IFeatureCollection | Request features | `context.Features.Get<IHttpRequestFeature>()` |

**Common Usage Patterns:**

**1. Access Request Information:**
```csharp
var path = context.Request.Path;              // /api/users
var method = context.Request.Method;          // GET
var query = context.Request.Query["id"];      // Query string param
var header = context.Request.Headers["X-API-Key"]; // Header
var isHttps = context.Request.IsHttps;        // true/false
```

**2. Modify Response:**
```csharp
context.Response.StatusCode = 200;
context.Response.ContentType = "application/json";
context.Response.Headers.Add("X-Custom", "Value");
await context.Response.WriteAsync("Hello");
await context.Response.WriteAsJsonAsync(new { status = "ok" });
```

**3. Access User Information:**
```csharp
if (context.User?.Identity?.IsAuthenticated == true)
{
    var userName = context.User.Identity.Name;
    var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var isAdmin = context.User.IsInRole("Admin");
}
```

**4. Store Request-Scoped Data:**
```csharp
// Middleware 1
context.Items["StartTime"] = DateTime.UtcNow;
context.Items["CorrelationId"] = Guid.NewGuid().ToString();

// Middleware 2
var startTime = (DateTime)context.Items["StartTime"];
var duration = DateTime.UtcNow - startTime;
```

**5. Resolve Services:**
```csharp
var logger = context.RequestServices.GetRequiredService<ILogger<MyClass>>();
var dbContext = context.RequestServices.GetRequiredService<ApplicationDbContext>();
var config = context.RequestServices.GetService<IConfiguration>();
```

---

### HttpRequest ⭐⭐⭐

**Purpose:** Represents incoming HTTP request

**Namespace:** `Microsoft.AspNetCore.Http`

**Key Members:**

| Member | Type | Description | Example |
|--------|------|-------------|---------|
| `Method` | string | HTTP verb (GET, POST, etc.) | `request.Method == "GET"` |
| `Scheme` | string | http or https | `request.Scheme` |
| `Host` | HostString | Domain and port | `request.Host.Value` |
| `Path` | PathString | URL path | `request.Path.Value` |
| `PathBase` | PathString | Application base path | `request.PathBase` |
| `QueryString` | QueryString | Raw query string | `request.QueryString.Value` |
| `Query` | IQueryCollection | Parsed query parameters | `request.Query["page"]` |
| `Headers` | IHeaderDictionary | HTTP headers | `request.Headers["Authorization"]` |
| `Cookies` | IRequestCookieCollection | Request cookies | `request.Cookies["session"]` |
| `ContentType` | string | Content-Type header | `request.ContentType` |
| `ContentLength` | long? | Request body size | `request.ContentLength` |
| `Body` | Stream | Request body stream | `await request.Body.ReadAsync(buffer)` |
| `Form` | IFormCollection | Form data (POST) | `request.Form["username"]` |
| `HasFormContentType` | bool | Is form data | `request.HasFormContentType` |
| `IsHttps` | bool | Is HTTPS request | `request.IsHttps` |
| `Protocol` | string | HTTP protocol version | `request.Protocol` |
| `RouteValues` | RouteValueDictionary | Route parameters | `request.RouteValues["id"]` |

**Complete Example:**
```csharp
// URL: https://example.com:443/api/users/5?page=2&sort=name

var method = context.Request.Method;         // GET
var scheme = context.Request.Scheme;         // https
var host = context.Request.Host.Value;       // example.com:443
var path = context.Request.Path.Value;       // /api/users/5
var query = context.Request.QueryString.Value; // ?page=2&sort=name

// Query parameters
var page = context.Request.Query["page"];    // "2"
var sort = context.Request.Query["sort"];    // "name"

// Headers
var auth = context.Request.Headers["Authorization"].ToString();
var userAgent = context.Request.Headers["User-Agent"].ToString();

// Cookies
var sessionId = context.Request.Cookies["SessionId"];

// Check method
if (context.Request.Method == HttpMethods.Post)
{
    // Handle POST
}
```

**Reading Request Body:**
```csharp
// JSON body
var user = await context.Request.ReadFromJsonAsync<User>();

// Raw text
using var reader = new StreamReader(context.Request.Body);
var body = await reader.ReadToEndAsync();

// Form data
if (context.Request.HasFormContentType)
{
    var form = await context.Request.ReadFormAsync();
    var username = form["username"];
}
```

---

### HttpResponse ⭐⭐⭐

**Purpose:** Represents outgoing HTTP response

**Namespace:** `Microsoft.AspNetCore.Http`

**Key Members:**

| Member | Type | Description | Example |
|--------|------|-------------|---------|
| `StatusCode` | int | HTTP status code | `response.StatusCode = 200` |
| `ContentType` | string | Content-Type header | `response.ContentType = "application/json"` |
| `ContentLength` | long? | Response body size | `response.ContentLength = 1024` |
| `Headers` | IHeaderDictionary | HTTP headers | `response.Headers.Add("X-Custom", "Value")` |
| `Cookies` | IResponseCookies | Response cookies | `response.Cookies.Append("session", "value")` |
| `Body` | Stream | Response body stream | `await response.Body.WriteAsync(bytes)` |
| `BodyWriter` | PipeWriter | High-performance writing | `response.BodyWriter` |
| `HasStarted` | bool | Response started | `if (!response.HasStarted)` |
| `OnStarting` | event | Called before response | `response.OnStarting(callback)` |
| `OnCompleted` | event | Called after response | `response.OnCompleted(callback)` |

**Common Patterns:**

**1. Setting Status and Content Type:**
```csharp
context.Response.StatusCode = 200;
context.Response.ContentType = "application/json";
```

**2. Writing JSON:**
```csharp
await context.Response.WriteAsJsonAsync(new 
{ 
    message = "Success",
    data = users 
});
```

**3. Writing Text:**
```csharp
await context.Response.WriteAsync("Hello World");
```

**4. Setting Headers:**
```csharp
context.Response.Headers.Add("X-Custom-Header", "Value");
context.Response.Headers.Add("Cache-Control", "no-cache");
context.Response.Headers["X-Response-Time"] = "50ms";
```

**5. Setting Cookies:**
```csharp
context.Response.Cookies.Append("SessionId", sessionId, new CookieOptions
{
    HttpOnly = true,
    Secure = true,
    SameSite = SameSiteMode.Strict,
    Expires = DateTime.UtcNow.AddHours(1)
});
```

**6. Redirecting:**
```csharp
context.Response.Redirect("/login");
context.Response.StatusCode = 302;
context.Response.Headers["Location"] = "/login";
```

**7. Streaming Large Response:**
```csharp
context.Response.ContentType = "application/octet-stream";
context.Response.Headers.Add("Content-Disposition", "attachment; filename=file.zip");

await using var fileStream = File.OpenRead("large-file.zip");
await fileStream.CopyToAsync(context.Response.Body);
```

---

### IHeaderDictionary ⭐⭐

**Purpose:** Collection of HTTP headers

**Namespace:** `Microsoft.AspNetCore.Http`

**Declaration:**
```csharp
public interface IHeaderDictionary : IDictionary<string, StringValues>
```

**Common Operations:**

```csharp
// Reading headers
var auth = context.Request.Headers["Authorization"];
var contentType = context.Request.Headers["Content-Type"];

// Setting headers
context.Response.Headers["X-Custom"] = "Value";
context.Response.Headers.Add("Cache-Control", "no-cache");

// Check if exists
if (context.Request.Headers.ContainsKey("X-API-Key"))
{
    var apiKey = context.Request.Headers["X-API-Key"];
}

// Remove header
context.Response.Headers.Remove("Server");

// Iterate headers
foreach (var header in context.Request.Headers)
{
    Console.WriteLine($"{header.Key}: {header.Value}");
}
```

---

### IQueryCollection ⭐⭐

**Purpose:** Collection of query string parameters

**Namespace:** `Microsoft.AspNetCore.Http`

**Usage:**

```csharp
// URL: /api/users?page=2&pageSize=10&sort=name&filter=active&filter=verified

// Get single value
var page = context.Request.Query["page"];           // "2"
var pageSize = context.Request.Query["pageSize"];   // "10"

// Get multiple values (array)
var filters = context.Request.Query["filter"];      // ["active", "verified"]

// Check if exists
if (context.Request.Query.ContainsKey("sort"))
{
    var sort = context.Request.Query["sort"];
}

// Iterate all
foreach (var param in context.Request.Query)
{
    Console.WriteLine($"{param.Key}: {param.Value}");
}

// Convert to specific types
int pageNumber = int.TryParse(context.Request.Query["page"], out var p) ? p : 1;
```

---

### IFormCollection ⭐⭐

**Purpose:** Collection of form data

**Namespace:** `Microsoft.AspNetCore.Http`

**Usage:**

```csharp
// Check if request has form data
if (context.Request.HasFormContentType)
{
    var form = await context.Request.ReadFormAsync();
    
    // Get form values
    var username = form["username"];
    var password = form["password"];
    
    // Get files
    var files = form.Files;
    foreach (var file in files)
    {
        Console.WriteLine($"File: {file.FileName}, Size: {file.Length}");
        
        // Save file
        await using var stream = File.Create($"uploads/{file.FileName}");
        await file.CopyToAsync(stream);
    }
}
```

---

## 2. Middleware Types

### IMiddleware ⭐⭐

**Purpose:** Factory-based middleware creation (supports scoped/transient lifetimes)

**Namespace:** `Microsoft.AspNetCore.Http`

**Declaration:**
```csharp
public interface IMiddleware
{
    Task InvokeAsync(HttpContext context, RequestDelegate next);
}
```

**When to Use:**
- ✅ Need scoped services (DbContext, etc.)
- ✅ Need transient lifetime
- ✅ Factory pattern for middleware
- ❌ Performance is critical (slight overhead)

**Complete Example:**
```csharp
public class DatabaseHealthCheckMiddleware : IMiddleware
{
    private readonly ApplicationDbContext _context; // Scoped!
    private readonly ILogger<DatabaseHealthCheckMiddleware> _logger;
    
    public DatabaseHealthCheckMiddleware(
        ApplicationDbContext context,
        ILogger<DatabaseHealthCheckMiddleware> logger)
    {
        _context = context;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        var canConnect = await _context.Database.CanConnectAsync();
        
        if (!canConnect)
        {
            _logger.LogError("Database connection failed");
            context.Response.StatusCode = 503;
            await context.Response.WriteAsync("Database unavailable");
            return;
        }
        
        await next(context);
    }
}

// Registration (REQUIRED for IMiddleware)
builder.Services.AddScoped<DatabaseHealthCheckMiddleware>();
app.UseMiddleware<DatabaseHealthCheckMiddleware>();
```

---

### RequestDelegate

**Purpose:** Represents the next middleware in the pipeline

**Namespace:** `Microsoft.AspNetCore.Http`

**Declaration:**
```csharp
public delegate Task RequestDelegate(HttpContext context);
```

**Usage in Middleware:**
```csharp
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    
    public MyMiddleware(RequestDelegate next)
    {
        _next = next; // Framework injects this
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Before next middleware
        Console.WriteLine("Before");
        
        await _next(context); // Call next middleware
        
        // After next middleware
        Console.WriteLine("After");
    }
}
```

**Key Points:**
- Always injected by framework in constructor
- Calling `await _next(context)` continues pipeline
- Not calling `_next` short-circuits the pipeline

---

### IApplicationBuilder ⭐⭐⭐

**Purpose:** Configure the HTTP request pipeline

**Namespace:** `Microsoft.AspNetCore.Builder`

**Key Members:**

| Member | Purpose | Example |
|--------|---------|---------|
| `Use()` | Add middleware (can call next) | `app.Use(async (ctx, next) => await next())` |
| `Run()` | Terminal middleware (no next) | `app.Run(async ctx => await ctx.Response.WriteAsync("Hi"))` |
| `Map()` | Branch pipeline by path | `app.Map("/api", apiApp => {...})` |
| `MapWhen()` | Branch by condition | `app.MapWhen(ctx => ctx.Request.IsHttps, ...)` |
| `UseWhen()` | Conditional middleware (rejoins) | `app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api"), ...)` |
| `ApplicationServices` | Service provider | `app.ApplicationServices` |
| `ServerFeatures` | Server capabilities | `app.ServerFeatures` |

**Common Patterns:**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build(); // app is IApplicationBuilder

// Use middleware
app.Use(async (context, next) =>
{
    Console.WriteLine("Before");
    await next();
    Console.WriteLine("After");
});

// Run (terminal)
app.Run(async context =>
{
    await context.Response.WriteAsync("End of pipeline");
});

// Map (branching)
app.Map("/api", apiApp =>
{
    apiApp.Run(async ctx => await ctx.Response.WriteAsync("API"));
});

// UseWhen (conditional, rejoins pipeline)
app.UseWhen(
    context => context.Request.Path.StartsWithSegments("/admin"),
    appBuilder =>
    {
        appBuilder.Use(async (context, next) =>
        {
            // Admin-only middleware
            await next();
        });
    });
```

**Extension Method Pattern:**
```csharp
public static class MiddlewareExtensions
{
    public static IApplicationBuilder UseMyMiddleware(this IApplicationBuilder app)
    {
        return app.UseMiddleware<MyMiddleware>();
    }
}

// Usage
app.UseMyMiddleware();
```

---

## 3. Dependency Injection

### IServiceCollection ⭐⭐⭐

**Purpose:** Register services for dependency injection

**Namespace:** `Microsoft.Extensions.DependencyInjection`

**Declaration:**
```csharp
public interface IServiceCollection : IList<ServiceDescriptor>
```

**Key Methods:**

| Method | Lifetime | Purpose | Example |
|--------|----------|---------|---------|
| `AddTransient<TService, TImplementation>()` | Transient | New instance every time | `services.AddTransient<IEmailService, EmailService>()` |
| `AddScoped<TService, TImplementation>()` | Scoped | One per request | `services.AddScoped<IUserService, UserService>()` |
| `AddSingleton<TService, TImplementation>()` | Singleton | One for app lifetime | `services.AddSingleton<ICacheService, CacheService>()` |
| `AddTransient<TService>(factory)` | Transient | Factory method | `services.AddTransient<IService>(sp => new Service())` |
| `AddScoped<TService>(factory)` | Scoped | Factory method | `services.AddScoped<IService>(sp => new Service())` |
| `AddSingleton<TService>(factory)` | Singleton | Factory method | `services.AddSingleton<IService>(sp => new Service())` |
| `AddSingleton<TService>(instance)` | Singleton | Specific instance | `services.AddSingleton<IConfig>(config)` |

**Service Lifetime Comparison:**

| Lifetime | Instance Created | Use When | Example |
|----------|------------------|----------|---------|
| **Transient** | Every request | Lightweight, stateless services | Email sender, HTTP client wrapper |
| **Scoped** | Once per HTTP request | Database contexts, unit of work | DbContext, request-specific services |
| **Singleton** | Once for app lifetime | Shared state, caching, configuration | Configuration, cache, logging factory |

**Visual Diagram:**
```
┌─────────────────────────────────────────────┐
│         Service Lifetime                    │
├─────────────────────────────────────────────┤
│                                             │
│  TRANSIENT: New ──> New ──> New ──> New    │
│             [A]     [B]     [C]     [D]     │
│                                             │
│  SCOPED:    ┌────────────┐  ┌────────────┐ │
│             │ Request 1  │  │ Request 2  │ │
│             │   [Same]   │  │   [Same]   │ │
│             └────────────┘  └────────────┘ │
│                                             │
│  SINGLETON: ┌──────────────────────────┐   │
│             │    [Same Instance]       │   │
│             └──────────────────────────┘   │
└─────────────────────────────────────────────┘
```

**Registration Examples:**

**1. Basic Registration:**
```csharp
// Interface → Implementation
services.AddScoped<IUserService, UserService>();
services.AddTransient<IEmailService, EmailService>();
services.AddSingleton<ICacheService, MemoryCacheService>();

// Concrete types (no interface)
services.AddScoped<UserRepository>();
services.AddTransient<EmailSender>();
```

**2. Factory Method:**
```csharp
services.AddScoped<IUserService>(sp =>
{
    var logger = sp.GetRequiredService<ILogger<UserService>>();
    var config = sp.GetRequiredService<IConfiguration>();
    return new UserService(logger, config);
});
```

**3. With Options:**
```csharp
services.Configure<EmailSettings>(builder.Configuration.GetSection("Email"));
services.AddScoped<IEmailService, EmailService>();
```

**4. Multiple Implementations:**
```csharp
services.AddScoped<INotificationService, EmailNotificationService>();
services.AddScoped<INotificationService, SmsNotificationService>();

// Resolve all
var notifiers = serviceProvider.GetServices<INotificationService>();
```

**5. Keyed Services (.NET 8.0+):**
```csharp
services.AddKeyedScoped<INotificationService, EmailNotificationService>("email");
services.AddKeyedScoped<INotificationService, SmsNotificationService>("sms");

// Resolve
var emailService = serviceProvider.GetRequiredKeyedService<INotificationService>("email");
```

---

### IServiceProvider ⭐⭐

**Purpose:** Resolve registered services

**Namespace:** `System`

**Key Methods:**

| Method | Purpose | Exception if not found |
|--------|---------|------------------------|
| `GetService<T>()` | Get service | Returns null |
| `GetRequiredService<T>()` | Get service | Throws exception |
| `GetServices<T>()` | Get all implementations | Returns empty |
| `GetKeyedService<T>(key)` | Get keyed service (.NET 8+) | Returns null |
| `GetRequiredKeyedService<T>(key)` | Get keyed service (.NET 8+) | Throws exception |

**Usage Examples:**

```csharp
// In controllers (injected automatically)
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService; // Injected
    }
}

// Manual resolution (service locator - avoid when possible)
var userService = serviceProvider.GetRequiredService<IUserService>();

// Get optional service
var cacheService = serviceProvider.GetService<ICacheService>();
if (cacheService != null)
{
    // Use cache
}

// Get all implementations
var notifiers = serviceProvider.GetServices<INotificationService>();
foreach (var notifier in notifiers)
{
    await notifier.SendAsync(message);
}

// From HttpContext
var logger = context.RequestServices.GetRequiredService<ILogger<MyClass>>();
```

---

### ServiceLifetime Enum

**Purpose:** Defines service lifetime in DI container

**Namespace:** `Microsoft.Extensions.DependencyInjection`

**Values:**

```csharp
public enum ServiceLifetime
{
    Singleton = 0,   // One instance for app lifetime
    Scoped = 1,      // One instance per request
    Transient = 2    // New instance every time
}
```

**When to Use Which:**

| Scenario | Lifetime | Reason |
|----------|----------|--------|
| Database context (EF Core) | Scoped | Per-request, manages transaction scope |
| HTTP client | Singleton/Transient | Reuse connections (use IHttpClientFactory) |
| Logging | Singleton | Shared logger factory |
| Caching service | Singleton | Shared cache across all requests |
| Email service | Transient | Stateless, lightweight |
| Unit of Work | Scoped | Per-request transaction |
| Configuration | Singleton | Same for entire app |
| User service (with DbContext) | Scoped | Depends on scoped DbContext |

**Decision Tree:**
```
Does it have state?
├─ NO → Transient (stateless services)
└─ YES → Does state need to be shared?
    ├─ Across entire app → Singleton
    └─ Only within a request → Scoped
```

---

## 4. Configuration

### IConfiguration ⭐⭐⭐

**Purpose:** Read application configuration

**Namespace:** `Microsoft.Extensions.Configuration`

**Key Members:**

| Member | Purpose | Example |
|--------|---------|---------|
| `[key]` | Get value by key | `config["AppName"]` |
| `GetSection()` | Get configuration section | `config.GetSection("ConnectionStrings")` |
| `GetChildren()` | Get child sections | `config.GetChildren()` |
| `Exists()` | Check if key exists | `config.GetSection("Key").Exists()` |
| `GetValue<T>()` | Get typed value | `config.GetValue<int>("Port")` |
| `Get<T>()` | Bind to object | `config.Get<AppSettings>()` |
| `Bind()` | Bind to existing object | `config.Bind(settings)` |

**Reading Configuration:**

**appsettings.json:**
```json
{
  "AppName": "MyApp",
  "Port": 5000,
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MyDb"
  },
  "Email": {
    "SmtpServer": "smtp.example.com",
    "Port": 587,
    "FromAddress": "noreply@example.com"
  },
  "AllowedHosts": ["example.com", "api.example.com"]
}
```

**Method 1: Direct Access (Simple Values)**
```csharp
var appName = configuration["AppName"];                    // "MyApp"
var connString = configuration["ConnectionStrings:Default"]; // Colon notation
var smtpServer = configuration["Email:SmtpServer"];
```

**Method 2: GetSection (Nested Values)**
```csharp
var emailSection = configuration.GetSection("Email");
var smtpServer = emailSection["SmtpServer"];
var port = emailSection.GetValue<int>("Port");
```

**Method 3: Strongly-Typed (Best Practice)**
```csharp
// Define class
public class EmailSettings
{
    public string SmtpServer { get; set; }
    public int Port { get; set; }
    public string FromAddress { get; set; }
}

// Bind
var emailSettings = configuration.GetSection("Email").Get<EmailSettings>();

// Or in DI
builder.Services.Configure<EmailSettings>(builder.Configuration.GetSection("Email"));
```

**Method 4: Arrays**
```csharp
var hosts = configuration.GetSection("AllowedHosts").Get<string[]>();
```

---

### IOptions<T> ⭐⭐

**Purpose:** Strongly-typed configuration (singleton pattern)

**Namespace:** `Microsoft.Extensions.Options`

**Declaration:**
```csharp
public interface IOptions<out TOptions> where TOptions : class
{
    TOptions Value { get; }
}
```

**When to Use:**
- ✅ Configuration doesn't change during runtime
- ✅ Singleton services
- ❌ Need to reload configuration

**Setup:**
```csharp
// 1. Define settings class
public class EmailSettings
{
    public string SmtpServer { get; set; }
    public int Port { get; set; }
    public string FromAddress { get; set; }
}

// 2. Register in DI
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("Email"));

// 3. Inject and use
public class EmailService
{
    private readonly EmailSettings _settings;
    
    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value; // Get settings once
    }
}
```

---

### IOptionsSnapshot<T> ⭐⭐

**Purpose:** Scoped configuration reload (per-request)

**Namespace:** `Microsoft.Extensions.Options`

**When to Use:**
- ✅ Need configuration reload per request
- ✅ Scoped services
- ❌ Singleton services (can't inject scoped into singleton)

**Usage:**
```csharp
public class UserService
{
    private readonly EmailSettings _settings;
    
    public UserService(IOptionsSnapshot<EmailSettings> options)
    {
        _settings = options.Value; // Reloaded each request
    }
}
```

---

### IOptionsMonitor<T> ⭐⭐

**Purpose:** Live configuration reload (singleton with change notifications)

**Namespace:** `Microsoft.Extensions.Options`

**Declaration:**
```csharp
public interface IOptionsMonitor<out TOptions>
{
    TOptions CurrentValue { get; }
    TOptions Get(string name);
    IDisposable OnChange(Action<TOptions, string> listener);
}
```

**When to Use:**
- ✅ Need live reload in singleton services
- ✅ React to configuration changes
- ✅ Named options

**Usage:**
```csharp
public class CacheService
{
    private readonly IOptionsMonitor<CacheSettings> _options;
    
    public CacheService(IOptionsMonitor<CacheSettings> options)
    {
        _options = options;
        
        // React to changes
        _options.OnChange(settings =>
        {
            Console.WriteLine("Cache settings changed!");
            // Reconfigure cache
        });
    }
    
    public void DoWork()
    {
        var settings = _options.CurrentValue; // Always current
    }
}
```

---

### IOptions Comparison Table

| Feature | IOptions<T> | IOptionsSnapshot<T> | IOptionsMonitor<T> |
|---------|-------------|---------------------|-------------------|
| **Lifetime** | Singleton | Scoped | Singleton |
| **Reload** | No | Per request | Live |
| **Change notification** | No | No | Yes |
| **Named options** | No | Yes | Yes |
| **Performance** | Fastest | Medium | Slower |
| **Use in singleton** | Yes | ❌ No | Yes |
| **Use in scoped** | Yes | Yes | Yes |
| **Best for** | Static config | Per-request reload | Live config changes |

**When to Use:**
```
Do you need configuration reload?
├─ NO → IOptions<T>
└─ YES → Is your service singleton?
    ├─ YES → IOptionsMonitor<T>
    └─ NO (Scoped) → IOptionsSnapshot<T>
```

---

## 5. Database (Entity Framework Core)

### DbContext ⭐⭐⭐

**Purpose:** Database session and unit of work

**Namespace:** `Microsoft.EntityFrameworkCore`

**Key Members:**

| Member | Purpose | Example |
|--------|---------|---------|
| `Database` | Database operations | `context.Database.EnsureCreated()` |
| `SaveChanges()` | Save changes synchronously | `context.SaveChanges()` |
| `SaveChangesAsync()` | Save changes asynchronously | `await context.SaveChangesAsync()` |
| `Entry()` | Get entity entry | `context.Entry(user).State` |
| `Set<T>()` | Get DbSet<T> | `context.Set<User>()` |
| `Add()` | Track entity for insert | `context.Add(user)` |
| `Update()` | Track entity for update | `context.Update(user)` |
| `Remove()` | Track entity for delete | `context.Remove(user)` |
| `ChangeTracker` | Track entity states | `context.ChangeTracker.Entries()` |

**Complete Example:**

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    // DbSet properties
    public DbSet<User> Users { get; set; }
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }
    
    // Configure model
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure entities
        modelBuilder.Entity<User>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Email).IsRequired().HasMaxLength(256);
            entity.HasIndex(e => e.Email).IsUnique();
        });
        
        modelBuilder.Entity<Order>(entity =>
        {
            entity.HasOne(o => o.User)
                  .WithMany(u => u.Orders)
                  .HasForeignKey(o => o.UserId);
        });
    }
}

// Registration
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

// Usage in service
public class UserService
{
    private readonly ApplicationDbContext _context;
    
    public UserService(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }
    
    public async Task CreateUserAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
    }
    
    public async Task UpdateUserAsync(User user)
    {
        _context.Users.Update(user);
        await _context.SaveChangesAsync();
    }
    
    public async Task DeleteUserAsync(int id)
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

---

### DbSet<T> ⭐⭐⭐

**Purpose:** Collection of entities for a type

**Namespace:** `Microsoft.EntityFrameworkCore`

**Key Methods:**

| Method | Purpose | Example |
|--------|---------|---------|
| `Add()` | Add new entity | `context.Users.Add(user)` |
| `AddRange()` | Add multiple entities | `context.Users.AddRange(users)` |
| `Update()` | Update entity | `context.Users.Update(user)` |
| `UpdateRange()` | Update multiple | `context.Users.UpdateRange(users)` |
| `Remove()` | Delete entity | `context.Users.Remove(user)` |
| `RemoveRange()` | Delete multiple | `context.Users.RemoveRange(users)` |
| `Find()` | Find by primary key | `context.Users.Find(5)` |
| `FindAsync()` | Find by PK async | `await context.Users.FindAsync(5)` |
| `AsNoTracking()` | Query without tracking | `context.Users.AsNoTracking()` |

**CRUD Operations:**

```csharp
// CREATE
var user = new User { Name = "John", Email = "john@example.com" };
context.Users.Add(user);
await context.SaveChangesAsync();

// READ (single)
var user = await context.Users.FindAsync(5);
var user = await context.Users.FirstOrDefaultAsync(u => u.Email == "john@example.com");

// READ (multiple)
var users = await context.Users.ToListAsync();
var activeUsers = await context.Users.Where(u => u.IsActive).ToListAsync();

// READ (with relationships)
var user = await context.Users
    .Include(u => u.Orders)
    .FirstOrDefaultAsync(u => u.Id == 5);

// UPDATE
var user = await context.Users.FindAsync(5);
user.Name = "Jane";
await context.SaveChangesAsync();

// DELETE
var user = await context.Users.FindAsync(5);
context.Users.Remove(user);
await context.SaveChangesAsync();

// BULK DELETE
var inactiveUsers = context.Users.Where(u => !u.IsActive);
context.Users.RemoveRange(inactiveUsers);
await context.SaveChangesAsync();
```

**Querying Patterns:**

```csharp
// Filtering
var adults = await context.Users.Where(u => u.Age >= 18).ToListAsync();

// Ordering
var sorted = await context.Users.OrderBy(u => u.Name).ToListAsync();

// Paging
var page = await context.Users
    .OrderBy(u => u.Id)
    .Skip(20)
    .Take(10)
    .ToListAsync();

// Projection
var names = await context.Users
    .Select(u => u.Name)
    .ToListAsync();

// Grouping
var grouped = await context.Orders
    .GroupBy(o => o.UserId)
    .Select(g => new { UserId = g.Key, Count = g.Count() })
    .ToListAsync();

// Joining
var result = await context.Orders
    .Join(context.Users, o => o.UserId, u => u.Id, (o, u) => new { o, u })
    .ToListAsync();
```

---

### ModelBuilder ⭐⭐

**Purpose:** Configure entity models using Fluent API

**Namespace:** `Microsoft.EntityFrameworkCore`

**Common Configurations:**

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Primary Key
    modelBuilder.Entity<User>()
        .HasKey(u => u.Id);
    
    // Composite Key
    modelBuilder.Entity<OrderItem>()
        .HasKey(oi => new { oi.OrderId, oi.ProductId });
    
    // Property configuration
    modelBuilder.Entity<User>()
        .Property(u => u.Email)
        .IsRequired()
        .HasMaxLength(256);
    
    // Index
    modelBuilder.Entity<User>()
        .HasIndex(u => u.Email)
        .IsUnique();
    
    // One-to-Many relationship
    modelBuilder.Entity<Order>()
        .HasOne(o => o.User)
        .WithMany(u => u.Orders)
        .HasForeignKey(o => o.UserId);
    
    // Many-to-Many relationship
    modelBuilder.Entity<Student>()
        .HasMany(s => s.Courses)
        .WithMany(c => c.Students)
        .UsingEntity(j => j.ToTable("StudentCourses"));
    
    // Table name
    modelBuilder.Entity<User>()
        .ToTable("AspNetUsers");
    
    // Ignore property
    modelBuilder.Entity<User>()
        .Ignore(u => u.FullName);
    
    // Default value
    modelBuilder.Entity<User>()
        .Property(u => u.CreatedAt)
        .HasDefaultValueSql("GETUTCDATE()");
    
    // Computed column
    modelBuilder.Entity<Product>()
        .Property(p => p.TotalPrice)
        .HasComputedColumnSql("[Price] * [Quantity]");
}
```

---

## 6. Controllers & Routing

### ControllerBase ⭐⭐⭐

**Purpose:** Base class for API controllers (no view support)

**Namespace:** `Microsoft.AspNetCore.Mvc`

**Key Properties:**

| Property | Type | Purpose |
|----------|------|---------|
| `HttpContext` | HttpContext | Current HTTP context |
| `Request` | HttpRequest | Current HTTP request |
| `Response` | HttpResponse | Current HTTP response |
| `User` | ClaimsPrincipal | Current user |
| `ModelState` | ModelStateDictionary | Model validation state |
| `Url` | IUrlHelper | Generate URLs |
| `RouteData` | RouteData | Current route data |

**Result Helper Methods:**

| Method | Status Code | Purpose | Example |
|--------|-------------|---------|---------|
| `Ok()` | 200 | Success | `Ok(data)` |
| `Created()` | 201 | Resource created | `Created(uri, data)` |
| `CreatedAtAction()` | 201 | Created with action route | `CreatedAtAction(nameof(Get), new { id }, data)` |
| `NoContent()` | 204 | Success, no content | `NoContent()` |
| `BadRequest()` | 400 | Client error | `BadRequest(error)` |
| `Unauthorized()` | 401 | Not authenticated | `Unauthorized()` |
| `Forbid()` | 403 | Not authorized | `Forbid()` |
| `NotFound()` | 404 | Resource not found | `NotFound()` |
| `Conflict()` | 409 | Conflict | `Conflict(error)` |
| `UnprocessableEntity()` | 422 | Validation error | `UnprocessableEntity(ModelState)` |
| `StatusCode()` | Custom | Custom status | `StatusCode(500, error)` |

**Complete Controller Example:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<UsersController> _logger;
    
    public UsersController(IUserService userService, ILogger<UsersController> logger)
    {
        _userService = userService;
        _logger = logger;
    }
    
    // GET: api/users
    [HttpGet]
    public async Task<ActionResult<IEnumerable<UserDto>>> GetAll()
    {
        var users = await _userService.GetAllAsync();
        return Ok(users);
    }
    
    // GET: api/users/5
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDto>> GetById(int id)
    {
        var user = await _userService.GetByIdAsync(id);
        if (user == null)
            return NotFound();
        
        return Ok(user);
    }
    
    // POST: api/users
    [HttpPost]
    public async Task<ActionResult<UserDto>> Create([FromBody] CreateUserDto dto)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        var user = await _userService.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById), new { id = user.Id }, user);
    }
    
    // PUT: api/users/5
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, [FromBody] UpdateUserDto dto)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        var exists = await _userService.ExistsAsync(id);
        if (!exists)
            return NotFound();
        
        await _userService.UpdateAsync(id, dto);
        return NoContent();
    }
    
    // DELETE: api/users/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var exists = await _userService.ExistsAsync(id);
        if (!exists)
            return NotFound();
        
        await _userService.DeleteAsync(id);
        return NoContent();
    }
}
```

---

### IActionResult ⭐⭐⭐

**Purpose:** Represents the result of an action method

**Namespace:** `Microsoft.AspNetCore.Mvc`

**Common Implementations:**

| Type | Status Code | Use Case |
|------|-------------|----------|
| `OkResult` | 200 | Success, no content |
| `OkObjectResult` | 200 | Success with content |
| `CreatedResult` | 201 | Resource created |
| `CreatedAtActionResult` | 201 | Created with route |
| `NoContentResult` | 204 | Success, no content |
| `BadRequestResult` | 400 | Client error |
| `BadRequestObjectResult` | 400 | Client error with details |
| `UnauthorizedResult` | 401 | Not authenticated |
| `ForbidResult` | 403 | Not authorized |
| `NotFoundResult` | 404 | Resource not found |
| `NotFoundObjectResult` | 404 | Not found with message |
| `ConflictResult` | 409 | Conflict |
| `UnprocessableEntityResult` | 422 | Validation failed |
| `StatusCodeResult` | Custom | Custom status code |
| `JsonResult` | 200 | JSON response |
| `FileResult` | 200 | File download |
| `RedirectResult` | 302 | Redirect |

**Usage Patterns:**

```csharp
// Simple returns
return Ok();                                    // 200
return Ok(data);                                // 200 with data
return Created("/api/users/5", user);           // 201
return CreatedAtAction(nameof(Get), new { id }, user); // 201
return NoContent();                             // 204
return BadRequest();                            // 400
return BadRequest("Invalid input");             // 400 with message
return Unauthorized();                          // 401
return NotFound();                              // 404
return NotFound($"User {id} not found");        // 404 with message

// ActionResult<T> (recommended for APIs)
[HttpGet("{id}")]
public async Task<ActionResult<UserDto>> Get(int id)
{
    var user = await _userService.GetAsync(id);
    
    // Can return T directly
    if (user != null)
        return user; // Implicit conversion to OkObjectResult
    
    // Or IActionResult
    return NotFound();
}

// File results
return File(bytes, "application/pdf", "report.pdf");
return PhysicalFile(path, "image/jpeg");
return File(stream, "text/csv", "export.csv");

// JSON with custom settings
return Json(data, new JsonSerializerOptions 
{ 
    WriteIndented = true 
});
```

---

### ActionResult<T> (.NET Core 2.1+)

**Purpose:** Strongly-typed action result

**Benefits:**
- ✅ IntelliSense support
- ✅ Return T directly or IActionResult
- ✅ Automatic serialization
- ✅ API documentation generation

**Example:**
```csharp
[HttpGet("{id}")]
public async Task<ActionResult<User>> Get(int id)
{
    var user = await _db.Users.FindAsync(id);
    if (user == null)
        return NotFound();
    
    return user; // Implicit Ok(user)
}
```

---

### RouteData ⭐

**Purpose:** Contains route values for current request

**Namespace:** `Microsoft.AspNetCore.Routing`

**Usage:**
```csharp
// In controller
var id = RouteData.Values["id"];
var controller = RouteData.Values["controller"];
var action = RouteData.Values["action"];

// Route values dictionary
var userId = Request.RouteValues["userId"];
```

---

## 7. Authentication & Authorization

### ClaimsPrincipal ⭐⭐

**Purpose:** Represents the current user

**Namespace:** `System.Security.Claims`

**Key Members:**

| Member | Purpose | Example |
|--------|---------|---------|
| `Identity` | Primary identity | `User.Identity.Name` |
| `Identities` | All identities | `User.Identities` |
| `Claims` | All claims | `User.Claims` |
| `FindFirst()` | Get first claim | `User.FindFirst(ClaimTypes.NameIdentifier)` |
| `FindAll()` | Get all claims of type | `User.FindAll("role")` |
| `HasClaim()` | Check claim exists | `User.HasClaim("admin", "true")` |
| `IsInRole()` | Check role | `User.IsInRole("Admin")` |

**Common Usage:**

```csharp
// In controller - User property is ClaimsPrincipal
if (User?.Identity?.IsAuthenticated == true)
{
    var userName = User.Identity.Name;
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var email = User.FindFirst(ClaimTypes.Email)?.Value;
    
    var isAdmin = User.IsInRole("Admin");
    var hasPermission = User.HasClaim("permission", "delete");
    
    // Get all roles
    var roles = User.FindAll(ClaimTypes.Role).Select(c => c.Value);
}

// In middleware
var user = context.User;
if (user.Identity?.IsAuthenticated == true)
{
    // User is authenticated
}
```

---

### ClaimsIdentity ⭐

**Purpose:** Represents a single identity

**Namespace:** `System.Security.Claims`

**Usage:**
```csharp
var identity = new ClaimsIdentity(new[]
{
    new Claim(ClaimTypes.NameIdentifier, "123"),
    new Claim(ClaimTypes.Name, "john@example.com"),
    new Claim(ClaimTypes.Email, "john@example.com"),
    new Claim(ClaimTypes.Role, "Admin"),
}, "Bearer");

var principal = new ClaimsPrincipal(identity);
```

---

### Claim ⭐

**Purpose:** Represents a single claim (key-value pair)

**Namespace:** `System.Security.Claims`

**Common Claim Types:**
```csharp
ClaimTypes.NameIdentifier   // User ID
ClaimTypes.Name             // Username
ClaimTypes.Email            // Email
ClaimTypes.Role             // Role
ClaimTypes.GivenName        // First name
ClaimTypes.Surname          // Last name
```

**Creating Claims:**
```csharp
var claims = new[]
{
    new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
    new Claim(ClaimTypes.Email, user.Email),
    new Claim(ClaimTypes.Role, "Admin"),
    new Claim("CustomClaim", "CustomValue")
};
```

---

### UserManager<TUser> ⭐⭐

**Purpose:** Manages users in ASP.NET Core Identity

**Namespace:** `Microsoft.AspNetCore.Identity`

**Key Methods:**

| Method | Purpose |
|--------|---------|
| `CreateAsync()` | Create new user |
| `UpdateAsync()` | Update user |
| `DeleteAsync()` | Delete user |
| `FindByIdAsync()` | Find by ID |
| `FindByEmailAsync()` | Find by email |
| `FindByNameAsync()` | Find by username |
| `AddToRoleAsync()` | Add user to role |
| `RemoveFromRoleAsync()` | Remove from role |
| `GetRolesAsync()` | Get user roles |
| `IsInRoleAsync()` | Check if in role |
| `AddClaimAsync()` | Add claim to user |
| `RemoveClaimAsync()` | Remove claim |
| `GetClaimsAsync()` | Get user claims |

**Example:**
```csharp
public class AccountService
{
    private readonly UserManager<ApplicationUser> _userManager;
    
    public AccountService(UserManager<ApplicationUser> userManager)
    {
        _userManager = userManager;
    }
    
    public async Task<IdentityResult> RegisterAsync(RegisterDto dto)
    {
        var user = new ApplicationUser
        {
            UserName = dto.Email,
            Email = dto.Email
        };
        
        var result = await _userManager.CreateAsync(user, dto.Password);
        
        if (result.Succeeded)
        {
            await _userManager.AddToRoleAsync(user, "User");
        }
        
        return result;
    }
    
    public async Task<ApplicationUser> GetUserByEmailAsync(string email)
    {
        return await _userManager.FindByEmailAsync(email);
    }
    
    public async Task<bool> CheckPasswordAsync(ApplicationUser user, string password)
    {
        return await _userManager.CheckPasswordAsync(user, password);
    }
}
```

---

### SignInManager<TUser> ⭐⭐

**Purpose:** Manages user sign-in

**Namespace:** `Microsoft.AspNetCore.Identity`

**Key Methods:**

| Method | Purpose |
|--------|---------|
| `SignInAsync()` | Sign in user |
| `SignOutAsync()` | Sign out user |
| `PasswordSignInAsync()` | Sign in with password |
| `IsSignedIn()` | Check if signed in |
| `GetExternalAuthenticationSchemesAsync()` | Get external providers |

**Example:**
```csharp
public class AuthController : ControllerBase
{
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly UserManager<ApplicationUser> _userManager;
    
    public AuthController(
        SignInManager<ApplicationUser> signInManager,
        UserManager<ApplicationUser> userManager)
    {
        _signInManager = signInManager;
        _userManager = userManager;
    }
    
    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginDto dto)
    {
        var result = await _signInManager.PasswordSignInAsync(
            dto.Email,
            dto.Password,
            isPersistent: dto.RememberMe,
            lockoutOnFailure: false);
        
        if (result.Succeeded)
            return Ok();
        
        if (result.IsLockedOut)
            return BadRequest("Account locked");
        
        return Unauthorized();
    }
    
    [HttpPost("logout")]
    public async Task<IActionResult> Logout()
    {
        await _signInManager.SignOutAsync();
        return NoContent();
    }
}
```

---

## 8. Logging

### ILogger<T> ⭐⭐⭐

**Purpose:** Log messages with structured logging

**Namespace:** `Microsoft.Extensions.Logging`

**Key Methods:**

| Method | Log Level | Use When |
|--------|-----------|----------|
| `LogTrace()` | Trace | Detailed diagnostic info |
| `LogDebug()` | Debug | Debugging during development |
| `LogInformation()` | Information | General flow of application |
| `LogWarning()` | Warning | Unexpected but handled events |
| `LogError()` | Error | Error that stopped operation |
| `LogCritical()` | Critical | Critical failure |
| `Log()` | Custom | Custom log level |

**Usage:**

```csharp
public class UserService
{
    private readonly ILogger<UserService> _logger;
    
    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        _logger.LogInformation("Getting user with ID {UserId}", id);
        
        try
        {
            var user = await _db.Users.FindAsync(id);
            
            if (user == null)
            {
                _logger.LogWarning("User with ID {UserId} not found", id);
                return null;
            }
            
            _logger.LogDebug("Found user: {@User}", user);
            return user;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error getting user {UserId}", id);
            throw;
        }
    }
}
```

**Structured Logging:**
```csharp
// ✅ Good - Structured (searchable, analyzable)
_logger.LogInformation("User {UserId} logged in from {IpAddress}", userId, ip);

// ❌ Bad - String interpolation (not structured)
_logger.LogInformation($"User {userId} logged in from {ip}");
```

**Log Scopes:**
```csharp
using (_logger.BeginScope("Processing order {OrderId}", orderId))
{
    _logger.LogInformation("Validating order");
    _logger.LogInformation("Processing payment");
    _logger.LogInformation("Sending confirmation");
}
```

---

### LogLevel Enum

**Purpose:** Defines log severity

**Namespace:** `Microsoft.Extensions.Logging`

```csharp
public enum LogLevel
{
    Trace = 0,       // Most detailed
    Debug = 1,       // Debugging
    Information = 2, // General flow
    Warning = 3,     // Unexpected
    Error = 4,       // Error
    Critical = 5,    // Critical failure
    None = 6         // No logging
}
```

**When to Use:**

| Level | Use For | Example |
|-------|---------|---------|
| Trace | Very detailed diagnostic | Variable values, method entry/exit |
| Debug | Debugging information | Query details, cache hits/misses |
| Information | General application flow | User logged in, order placed |
| Warning | Unexpected but handled | Missing optional config, deprecated API |
| Error | Error that stopped current operation | Database error, file not found |
| Critical | System-wide failure | Database down, out of memory |

**Configuration (appsettings.json):**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "MyApp.Services": "Debug"
    }
  }
}
```

---

### ILoggerFactory ⭐

**Purpose:** Create logger instances

**Namespace:** `Microsoft.Extensions.Logging`

**Usage:**
```csharp
public class MyClass
{
    private readonly ILogger _logger;
    
    public MyClass(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<MyClass>();
        // or
        _logger = loggerFactory.CreateLogger("MyCategory");
    }
}
```

---

## 9. Filters

### IActionFilter ⭐⭐

**Purpose:** Execute code before/after action execution

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Declaration:**
```csharp
public interface IActionFilter
{
    void OnActionExecuting(ActionExecutingContext context);
    void OnActionExecuted(ActionExecutedContext context);
}
```

**Async Version:**
```csharp
public interface IAsyncActionFilter
{
    Task OnActionExecutionAsync(
        ActionExecutingContext context, 
        ActionExecutionDelegate next);
}
```

**Example:**
```csharp
public class LogActionFilter : IAsyncActionFilter
{
    private readonly ILogger<LogActionFilter> _logger;
    
    public LogActionFilter(ILogger<LogActionFilter> logger)
    {
        _logger = logger;
    }
    
    public async Task OnActionExecutionAsync(
        ActionExecutingContext context, 
        ActionExecutionDelegate next)
    {
        // Before action
        _logger.LogInformation("Executing {Action}", context.ActionDescriptor.DisplayName);
        
        var result = await next(); // Execute action
        
        // After action
        _logger.LogInformation("Executed {Action}", context.ActionDescriptor.DisplayName);
    }
}

// Register globally
builder.Services.AddControllers(options =>
{
    options.Filters.Add<LogActionFilter>();
});

// Or use as attribute
[ServiceFilter(typeof(LogActionFilter))]
public class UsersController : ControllerBase { }
```

---

### IExceptionFilter ⭐⭐

**Purpose:** Handle exceptions from actions

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Declaration:**
```csharp
public interface IExceptionFilter
{
    void OnException(ExceptionContext context);
}
```

**Example:**
```csharp
public class GlobalExceptionFilter : IExceptionFilter
{
    private readonly ILogger<GlobalExceptionFilter> _logger;
    
    public GlobalExceptionFilter(ILogger<GlobalExceptionFilter> logger)
    {
        _logger = logger;
    }
    
    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "Unhandled exception");
        
        var response = new
        {
            error = "An error occurred",
            message = context.Exception.Message
        };
        
        context.Result = new ObjectResult(response)
        {
            StatusCode = 500
        };
        
        context.ExceptionHandled = true;
    }
}

// Register
builder.Services.AddControllers(options =>
{
    options.Filters.Add<GlobalExceptionFilter>();
});
```

---

### IAuthorizationFilter ⭐⭐

**Purpose:** Perform authorization

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Declaration:**
```csharp
public interface IAuthorizationFilter
{
    void OnAuthorization(AuthorizationFilterContext context);
}
```

**Example:**
```csharp
public class ApiKeyAuthorizationFilter : IAuthorizationFilter
{
    private readonly IConfiguration _configuration;
    
    public ApiKeyAuthorizationFilter(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public void OnAuthorization(AuthorizationFilterContext context)
    {
        var apiKey = context.HttpContext.Request.Headers["X-API-Key"].ToString();
        var validKey = _configuration["ApiKey"];
        
        if (string.IsNullOrEmpty(apiKey) || apiKey != validKey)
        {
            context.Result = new UnauthorizedResult();
        }
    }
}
```

---

### Filter Pipeline Order

```
1. Authorization Filters
   ↓
2. Resource Filters (before)
   ↓
3. Action Filters (before)
   ↓
4. Exception Filters (if exception)
   ↓
5. Action Execution
   ↓
6. Action Filters (after)
   ↓
7. Result Filters (before)
   ↓
8. Result Execution
   ↓
9. Result Filters (after)
   ↓
10. Resource Filters (after)
```

---

## 10. Hosting & Application

### WebApplication ⭐⭐⭐

**Purpose:** Represents configured application in minimal hosting model (.NET 6+)

**Namespace:** `Microsoft.AspNetCore.Builder`

**Key Members:**

| Member | Purpose | Example |
|--------|---------|---------|
| `Services` | Service provider | `app.Services.GetService<T>()` |
| `Configuration` | Configuration | `app.Configuration["Key"]` |
| `Environment` | Host environment | `app.Environment.IsDevelopment()` |
| `Lifetime` | Application lifetime | `app.Lifetime.ApplicationStopping` |
| `Urls` | Server URLs | `app.Urls` |
| `Logger` | Logger factory | `app.Logger` |
| `Run()` | Start application | `app.Run()` |
| `RunAsync()` | Start async | `await app.RunAsync()` |
| `StartAsync()` | Start without blocking | `await app.StartAsync()` |
| `StopAsync()` | Stop application | `await app.StopAsync()` |

**Both IApplicationBuilder and IEndpointRouteBuilder:**
```csharp
// Configure pipeline (IApplicationBuilder)
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

// Map endpoints (IEndpointRouteBuilder)
app.MapGet("/", () => "Hello");
app.MapControllers();
```

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

// Check environment
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

// Configure pipeline
app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();

// Map endpoints
app.MapControllers();
app.MapGet("/health", () => "Healthy");

// Run
app.Run();
```

---

### WebApplicationBuilder ⭐⭐⭐

**Purpose:** Builder for WebApplication

**Namespace:** `Microsoft.AspNetCore.Builder`

**Key Members:**

| Member | Purpose | Example |
|--------|---------|---------|
| `Services` | Service collection | `builder.Services.AddScoped<T>()` |
| `Configuration` | Configuration builder | `builder.Configuration.GetSection("Key")` |
| `Environment` | Host environment | `builder.Environment.EnvironmentName` |
| `Host` | Host builder | `builder.Host.ConfigureServices()` |
| `WebHost` | Web host builder | `builder.WebHost.UseUrls()` |
| `Logging` | Logging builder | `builder.Logging.AddConsole()` |
| `Build()` | Build application | `var app = builder.Build()` |

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Configure services
builder.Services.AddControllers();
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
builder.Services.AddScoped<IUserService, UserService>();

// Configure logging
builder.Logging.AddConsole();
builder.Logging.SetMinimumLevel(LogLevel.Information);

// Configure web host
builder.WebHost.UseUrls("http://localhost:5000");

// Build app
var app = builder.Build();
```

---

### IWebHostEnvironment ⭐⭐

**Purpose:** Provides information about web hosting environment

**Namespace:** `Microsoft.AspNetCore.Hosting`

**Key Members:**

| Member | Type | Purpose |
|--------|------|---------|
| `EnvironmentName` | string | Environment name (Development, Staging, Production) |
| `ApplicationName` | string | Application name |
| `ContentRootPath` | string | Content root directory path |
| `WebRootPath` | string | Web root directory path (wwwroot) |
| `IsDevelopment()` | bool | Is Development environment |
| `IsStaging()` | bool | Is Staging environment |
| `IsProduction()` | bool | Is Production environment |
| `IsEnvironment(name)` | bool | Check specific environment |

**Usage:**
```csharp
public class MyService
{
    private readonly IWebHostEnvironment _env;
    
    public MyService(IWebHostEnvironment env)
    {
        _env = env;
    }
    
    public void DoWork()
    {
        if (_env.IsDevelopment())
        {
            // Development-specific logic
        }
        
        var contentPath = _env.ContentRootPath;  // /app
        var webPath = _env.WebRootPath;          // /app/wwwroot
        var envName = _env.EnvironmentName;      // "Development"
    }
}

// In Program.cs
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}
```

---

# PART 2: NAMESPACE REFERENCE

---

## 11. Essential Namespaces by Category

### Core HTTP
```csharp
using Microsoft.AspNetCore.Http;
```
**Key Types:**
- HttpContext
- HttpRequest
- HttpResponse
- IHeaderDictionary
- IQueryCollection
- IFormCollection
- ISession
- CookieOptions
- StatusCodes

**When to Use:** Working with HTTP requests/responses, middleware

---

### HTTP Features
```csharp
using Microsoft.AspNetCore.Http.Features;
```
**Key Types:**
- IHttpRequestFeature
- IHttpResponseFeature
- IHttpConnectionFeature
- IHttpRequestLifetimeFeature

**When to Use:** Low-level HTTP features, advanced scenarios

---

### Middleware & Pipeline
```csharp
using Microsoft.AspNetCore.Builder;
```
**Key Types:**
- IApplicationBuilder
- WebApplication
- WebApplicationBuilder
- EndpointRouteBuilderExtensions

**When to Use:** Configuring middleware pipeline, application setup

---

### MVC / Controllers
```csharp
using Microsoft.AspNetCore.Mvc;
```
**Key Types:**
- ControllerBase
- Controller (with view support)
- IActionResult
- ActionResult<T>
- JsonResult
- ObjectResult
- StatusCodeResult
- FileResult
- RedirectResult

**When to Use:** Creating API controllers, MVC controllers

---

### MVC Filters
```csharp
using Microsoft.AspNetCore.Mvc.Filters;
```
**Key Types:**
- IActionFilter
- IAsyncActionFilter
- IExceptionFilter
- IAuthorizationFilter
- IResourceFilter
- IResultFilter
- FilterAttribute

**When to Use:** Creating filters for cross-cutting concerns

---

### Model Binding
```csharp
using Microsoft.AspNetCore.Mvc.ModelBinding;
```
**Key Types:**
- ModelStateDictionary
- IModelBinder
- BindingSource
- ModelBinderAttribute

**When to Use:** Custom model binding, validation

---

### Routing
```csharp
using Microsoft.AspNetCore.Routing;
```
**Key Types:**
- IRouter
- RouteData
- RouteValueDictionary
- LinkGenerator
- EndpointDataSource

**When to Use:** Custom routing, URL generation

---

### Dependency Injection
```csharp
using Microsoft.Extensions.DependencyInjection;
```
**Key Types:**
- IServiceCollection
- IServiceProvider
- ServiceDescriptor
- ServiceLifetime
- ActivatorUtilities

**When to Use:** Registering services, service resolution

---

### Configuration
```csharp
using Microsoft.Extensions.Configuration;
```
**Key Types:**
- IConfiguration
- IConfigurationBuilder
- IConfigurationSection
- ConfigurationManager

**When to Use:** Reading configuration

---

### Options Pattern
```csharp
using Microsoft.Extensions.Options;
```
**Key Types:**
- IOptions<T>
- IOptionsSnapshot<T>
- IOptionsMonitor<T>
- IConfigureOptions<T>
- OptionsBuilder<T>

**When to Use:** Strongly-typed configuration

---

### Logging
```csharp
using Microsoft.Extensions.Logging;
```
**Key Types:**
- ILogger<T>
- ILoggerFactory
- LogLevel
- ILoggingBuilder

**When to Use:** Logging throughout application

---

### Hosting
```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;
```
**Key Types:**
- IWebHostEnvironment
- IHostEnvironment
- IHostBuilder
- IHost
- IHostApplicationLifetime

**When to Use:** Application startup, environment checks

---

### Entity Framework Core
```csharp
using Microsoft.EntityFrameworkCore;
```
**Key Types:**
- DbContext
- DbSet<T>
- ModelBuilder
- DbContextOptions
- ChangeTracker
- Database

**When to Use:** Database access with EF Core

---

### Authentication
```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Authentication.Cookies;
```
**Key Types:**
- AuthenticationOptions
- JwtBearerOptions
- CookieAuthenticationOptions
- AuthenticationScheme

**When to Use:** Configuring authentication

---

### Authorization
```csharp
using Microsoft.AspNetCore.Authorization;
```
**Key Types:**
- AuthorizeAttribute
- AllowAnonymousAttribute
- IAuthorizationService
- AuthorizationPolicy
- IAuthorizationRequirement

**When to Use:** Authorization policies, requirements

---

### Identity
```csharp
using Microsoft.AspNetCore.Identity;
```
**Key Types:**
- UserManager<TUser>
- SignInManager<TUser>
- RoleManager<TRole>
- IdentityUser
- IdentityRole
- IdentityResult

**When to Use:** User management with Identity

---

### Claims
```csharp
using System.Security.Claims;
```
**Key Types:**
- ClaimsPrincipal
- ClaimsIdentity
- Claim
- ClaimTypes

**When to Use:** Working with user claims

---

# PART 3: COMMON ATTRIBUTES REFERENCE

---

## 12. Attributes by Category

### Routing Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[Route]` | Define route template | `[Route("api/[controller]")]` |
| `[HttpGet]` | HTTP GET method | `[HttpGet("{id}")]` |
| `[HttpPost]` | HTTP POST method | `[HttpPost]` |
| `[HttpPut]` | HTTP PUT method | `[HttpPut("{id}")]` |
| `[HttpDelete]` | HTTP DELETE method | `[HttpDelete("{id}")]` |
| `[HttpPatch]` | HTTP PATCH method | `[HttpPatch("{id}")]` |
| `[HttpHead]` | HTTP HEAD method | `[HttpHead]` |
| `[HttpOptions]` | HTTP OPTIONS method | `[HttpOptions]` |

**Examples:**
```csharp
[Route("api/[controller]")]
[ApiController]
public class UsersController : ControllerBase
{
    [HttpGet]                    // GET api/users
    public IActionResult GetAll() { }
    
    [HttpGet("{id}")]            // GET api/users/5
    public IActionResult Get(int id) { }
    
    [HttpPost]                   // POST api/users
    public IActionResult Create([FromBody] User user) { }
    
    [HttpPut("{id}")]            // PUT api/users/5
    public IActionResult Update(int id, [FromBody] User user) { }
    
    [HttpDelete("{id}")]         // DELETE api/users/5
    public IActionResult Delete(int id) { }
}
```

---

### Model Binding Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[FromRoute]` | Bind from route data | `public IActionResult Get([FromRoute] int id)` |
| `[FromQuery]` | Bind from query string | `public IActionResult Get([FromQuery] string name)` |
| `[FromBody]` | Bind from request body | `public IActionResult Create([FromBody] User user)` |
| `[FromForm]` | Bind from form data | `public IActionResult Upload([FromForm] IFormFile file)` |
| `[FromHeader]` | Bind from header | `public IActionResult Get([FromHeader] string apiKey)` |
| `[FromServices]` | Inject from DI | `public IActionResult Get([FromServices] IUserService service)` |

**Examples:**
```csharp
// Route parameter
[HttpGet("{id}")]
public IActionResult Get([FromRoute] int id) { }

// Query string
[HttpGet]
public IActionResult Search([FromQuery] string q, [FromQuery] int page = 1) { }
// GET /api/users?q=john&page=2

// Request body (JSON)
[HttpPost]
public IActionResult Create([FromBody] CreateUserDto dto) { }

// Form data
[HttpPost("upload")]
public IActionResult Upload([FromForm] IFormFile file, [FromForm] string description) { }

// Header
[HttpGet]
public IActionResult Get([FromHeader(Name = "X-API-Key")] string apiKey) { }

// Service injection
[HttpGet]
public IActionResult Get([FromServices] IUserService userService) { }
```

---

### Validation Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[Required]` | Field is required | `[Required] public string Name { get; set; }` |
| `[StringLength]` | Max/min string length | `[StringLength(100, MinimumLength = 3)]` |
| `[MinLength]` | Minimum length | `[MinLength(3)]` |
| `[MaxLength]` | Maximum length | `[MaxLength(100)]` |
| `[Range]` | Numeric range | `[Range(1, 100)]` |
| `[EmailAddress]` | Valid email format | `[EmailAddress]` |
| `[Phone]` | Valid phone format | `[Phone]` |
| `[Url]` | Valid URL format | `[Url]` |
| `[CreditCard]` | Valid credit card | `[CreditCard]` |
| `[RegularExpression]` | Regex pattern | `[RegularExpression(@"^\d{3}-\d{2}-\d{4}$")]` |
| `[Compare]` | Compare two properties | `[Compare("Password")]` |

**Example DTO:**
```csharp
public class CreateUserDto
{
    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; }
    
    [Required]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    public string Email { get; set; }
    
    [Required]
    [StringLength(100, MinimumLength = 8)]
    [RegularExpression(@"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).+$", 
        ErrorMessage = "Password must contain uppercase, lowercase and number")]
    public string Password { get; set; }
    
    [Compare("Password", ErrorMessage = "Passwords don't match")]
    public string ConfirmPassword { get; set; }
    
    [Range(18, 120, ErrorMessage = "Age must be between 18 and 120")]
    public int Age { get; set; }
    
    [Phone]
    public string PhoneNumber { get; set; }
    
    [Url]
    public string Website { get; set; }
}
```

---

### API Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[ApiController]` | Enable API behaviors | `[ApiController]` |
| `[Produces]` | Response content type | `[Produces("application/json")]` |
| `[Consumes]` | Request content type | `[Consumes("application/json")]` |
| `[ProducesResponseType]` | Document response types | `[ProducesResponseType(200, Type = typeof(User))]` |

**Examples:**
```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class UsersController : ControllerBase
{
    [HttpGet("{id}")]
    [ProducesResponseType(200, Type = typeof(UserDto))]
    [ProducesResponseType(404)]
    public async Task<ActionResult<UserDto>> Get(int id)
    {
        var user = await _userService.GetAsync(id);
        return user != null ? Ok(user) : NotFound();
    }
    
    [HttpPost]
    [Consumes("application/json")]
    [ProducesResponseType(201, Type = typeof(UserDto))]
    [ProducesResponseType(400)]
    public async Task<ActionResult<UserDto>> Create([FromBody] CreateUserDto dto)
    {
        var user = await _userService.CreateAsync(dto);
        return CreatedAtAction(nameof(Get), new { id = user.Id }, user);
    }
}
```

---

### Authorization Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[Authorize]` | Require authentication | `[Authorize]` |
| `[Authorize(Roles)]` | Require specific role | `[Authorize(Roles = "Admin")]` |
| `[Authorize(Policy)]` | Require policy | `[Authorize(Policy = "MinAge18")]` |
| `[AllowAnonymous]` | Allow anonymous access | `[AllowAnonymous]` |

**Examples:**
```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize] // All actions require authentication
public class UsersController : ControllerBase
{
    [HttpGet]
    [AllowAnonymous] // Override - allow anonymous
    public IActionResult GetAll() { }
    
    [HttpGet("{id}")]
    // Requires authentication (from controller)
    public IActionResult Get(int id) { }
    
    [HttpPost]
    [Authorize(Roles = "Admin")] // Requires Admin role
    public IActionResult Create([FromBody] User user) { }
    
    [HttpDelete("{id}")]
    [Authorize(Policy = "CanDeleteUsers")] // Requires policy
    public IActionResult Delete(int id) { }
}
```

---

### Filter Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[ServiceFilter]` | Use filter from DI | `[ServiceFilter(typeof(MyFilter))]` |
| `[TypeFilter]` | Use filter (creates instance) | `[TypeFilter(typeof(MyFilter))]` |
| `[ValidateAntiForgeryToken]` | Validate anti-forgery token | `[ValidateAntiForgeryToken]` |
| `[IgnoreAntiforgeryToken]` | Ignore anti-forgery | `[IgnoreAntiforgeryToken]` |

**Examples:**
```csharp
// Service filter (filter registered in DI)
builder.Services.AddScoped<LogActionFilter>();

[ServiceFilter(typeof(LogActionFilter))]
public class UsersController : ControllerBase { }

// Type filter (creates instance, can pass arguments)
[TypeFilter(typeof(MyFilter), Arguments = new object[] { "param" })]
public IActionResult MyAction() { }
```

---

# PART 4: QUICK LOOKUP TABLES

---

## 13. HTTP Status Codes - Complete Reference

### Success (2xx)

| Code | Name | Usage |
|------|------|-------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST (resource created) |
| 202 | Accepted | Request accepted (async processing) |
| 204 | No Content | Successful DELETE, PUT (no body) |
| 206 | Partial Content | Partial GET (range request) |

### Redirection (3xx)

| Code | Name | Usage |
|------|------|-------|
| 301 | Moved Permanently | Resource permanently moved |
| 302 | Found | Temporary redirect |
| 304 | Not Modified | Cached resource still valid |
| 307 | Temporary Redirect | Temporary redirect (same method) |
| 308 | Permanent Redirect | Permanent redirect (same method) |

### Client Errors (4xx)

| Code | Name | Usage | Example |
|------|------|-------|---------|
| 400 | Bad Request | Invalid request | Malformed JSON |
| 401 | Unauthorized | Not authenticated | Missing/invalid token |
| 403 | Forbidden | Not authorized | Insufficient permissions |
| 404 | Not Found | Resource not found | User doesn't exist |
| 405 | Method Not Allowed | HTTP method not supported | POST on read-only endpoint |
| 409 | Conflict | Conflict with current state | Duplicate email |
| 415 | Unsupported Media Type | Wrong Content-Type | Sent XML, expects JSON |
| 422 | Unprocessable Entity | Validation failed | Email format invalid |
| 429 | Too Many Requests | Rate limit exceeded | Too many API calls |

### Server Errors (5xx)

| Code | Name | Usage | Example |
|------|------|-------|---------|
| 500 | Internal Server Error | Unexpected server error | Unhandled exception |
| 501 | Not Implemented | Not implemented | Feature not ready |
| 502 | Bad Gateway | Invalid response from upstream | Proxy error |
| 503 | Service Unavailable | Service temporarily unavailable | Database down |
| 504 | Gateway Timeout | Upstream timeout | Long-running operation |

**Common Patterns:**
```csharp
// Success
return Ok(data);                    // 200
return Created(uri, data);          // 201
return NoContent();                 // 204

// Client errors
return BadRequest("Invalid data");  // 400
return Unauthorized();              // 401
return Forbid();                    // 403
return NotFound();                  // 404
return Conflict("Email exists");    // 409

// Server errors
return StatusCode(500, error);      // 500
return StatusCode(503, "DB down");  // 503
```

---

## 14. Content Types - Common MIME Types

### Application

| MIME Type | Extension | Usage |
|-----------|-----------|-------|
| `application/json` | .json | JSON data (APIs) |
| `application/xml` | .xml | XML data |
| `application/pdf` | .pdf | PDF documents |
| `application/zip` | .zip | ZIP archives |
| `application/x-www-form-urlencoded` | - | Form data (POST) |
| `application/octet-stream` | - | Binary data |

### Text

| MIME Type | Extension | Usage |
|-----------|-----------|-------|
| `text/plain` | .txt | Plain text |
| `text/html` | .html | HTML documents |
| `text/css` | .css | CSS files |
| `text/javascript` | .js | JavaScript files |
| `text/csv` | .csv | CSV data |

### Image

| MIME Type | Extension | Usage |
|-----------|-----------|-------|
| `image/jpeg` | .jpg, .jpeg | JPEG images |
| `image/png` | .png | PNG images |
| `image/gif` | .gif | GIF images |
| `image/svg+xml` | .svg | SVG images |
| `image/webp` | .webp | WebP images |

### Audio/Video

| MIME Type | Extension | Usage |
|-----------|-----------|-------|
| `audio/mpeg` | .mp3 | MP3 audio |
| `audio/wav` | .wav | WAV audio |
| `video/mp4` | .mp4 | MP4 video |
| `video/webm` | .webm | WebM video |

**Usage:**
```csharp
// Set Content-Type
context.Response.ContentType = "application/json";
context.Response.ContentType = "text/html; charset=utf-8";

// File results
return File(bytes, "application/pdf", "report.pdf");
return File(stream, "image/jpeg", "photo.jpg");
return File(path, "application/zip", "archive.zip");
```

---

## 15. Extension Methods - Commonly Used

### IServiceCollection Extensions

```csharp
// Add services
services.AddControllers();
services.AddRazorPages();
services.AddEndpointsApiExplorer();
services.AddSwaggerGen();

// Database
services.AddDbContext<ApplicationDbContext>(options => ...);

// Authentication
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => ...);

// Authorization
services.AddAuthorization(options => ...);

// CORS
services.AddCors(options => ...);

// Health checks
services.AddHealthChecks();

// HTTP Client
services.AddHttpClient();

// Memory cache
services.AddMemoryCache();

// Distributed cache
services.AddDistributedMemoryCache();
services.AddStackExchangeRedisCache(options => ...);

// Identity
services.AddIdentity<ApplicationUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

// API versioning
services.AddApiVersioning();
```

### IApplicationBuilder Extensions

```csharp
// Exception handling
app.UseDeveloperExceptionPage();
app.UseExceptionHandler("/error");

// HTTPS
app.UseHttpsRedirection();
app.UseHsts();

// Static files
app.UseStaticFiles();

// Routing
app.UseRouting();

// CORS
app.UseCors();

// Authentication & Authorization
app.UseAuthentication();
app.UseAuthorization();

// Session
app.UseSession();

// Rate limiting
app.UseRateLimiter();

// Response compression
app.UseResponseCompression();

// Response caching
app.UseResponseCaching();
```

### IEndpointRouteBuilder Extensions

```csharp
// Map endpoints
app.MapControllers();
app.MapRazorPages();
app.MapBlazorHub();
app.MapFallbackToPage("/_Host");

// Map HTTP methods
app.MapGet("/", () => "Hello");
app.MapPost("/api/users", async (User user) => ...);
app.MapPut("/api/users/{id}", async (int id, User user) => ...);
app.MapDelete("/api/users/{id}", async (int id) => ...);

// Map groups
var api = app.MapGroup("/api");
api.MapGet("/users", () => ...);
api.MapPost("/users", () => ...);

// Health checks
app.MapHealthChecks("/health");
```

---

## 16. Common Patterns - Code Snippets

### Pattern 1: Async CRUD Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _service;
    
    public UsersController(IUserService service) => _service = service;
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<UserDto>>> GetAll() =>
        Ok(await _service.GetAllAsync());
    
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDto>> Get(int id)
    {
        var user = await _service.GetAsync(id);
        return user != null ? Ok(user) : NotFound();
    }
    
    [HttpPost]
    public async Task<ActionResult<UserDto>> Create(CreateUserDto dto)
    {
        var user = await _service.CreateAsync(dto);
        return CreatedAtAction(nameof(Get), new { id = user.Id }, user);
    }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, UpdateUserDto dto)
    {
        var exists = await _service.ExistsAsync(id);
        if (!exists) return NotFound();
        
        await _service.UpdateAsync(id, dto);
        return NoContent();
    }
    
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var exists = await _service.ExistsAsync(id);
        if (!exists) return NotFound();
        
        await _service.DeleteAsync(id);
        return NoContent();
    }
}
```

### Pattern 2: Repository Pattern

```csharp
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly ApplicationDbContext _context;
    private readonly DbSet<T> _dbSet;
    
    public Repository(ApplicationDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T> GetByIdAsync(int id) =>
        await _dbSet.FindAsync(id);
    
    public async Task<IEnumerable<T>> GetAllAsync() =>
        await _dbSet.ToListAsync();
    
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
        var entity = await GetByIdAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}
```

### Pattern 3: Result Pattern

```csharp
public class Result<T>
{
    public bool Success { get; set; }
    public T Data { get; set; }
    public string Error { get; set; }
    
    public static Result<T> Ok(T data) =>
        new Result<T> { Success = true, Data = data };
    
    public static Result<T> Fail(string error) =>
        new Result<T> { Success = false, Error = error };
}

// Usage
public async Task<Result<User>> CreateUserAsync(CreateUserDto dto)
{
    try
    {
        var user = await _userRepository.AddAsync(dto);
        return Result<User>.Ok(user);
    }
    catch (Exception ex)
    {
        return Result<User>.Fail(ex.Message);
    }
}

// In controller
var result = await _service.CreateUserAsync(dto);
return result.Success 
    ? CreatedAtAction(nameof(Get), new { id = result.Data.Id }, result.Data)
    : BadRequest(result.Error);
```

### Pattern 4: API Response Wrapper

```csharp
public class ApiResponse<T>
{
    public bool Success { get; set; }
    public T Data { get; set; }
    public string Message { get; set; }
    public List<string> Errors { get; set; } = new();
}

// Usage in middleware
public class ApiResponseMiddleware
{
    private readonly RequestDelegate _next;
    
    public ApiResponseMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var originalBody = context.Response.Body;
        
        using var newBody = new MemoryStream();
        context.Response.Body = newBody;
        
        await _next(context);
        
        newBody.Seek(0, SeekOrigin.Begin);
        var responseText = await new StreamReader(newBody).ReadToEndAsync();
        
        var response = new ApiResponse<object>
        {
            Success = context.Response.StatusCode < 400,
            Data = JsonSerializer.Deserialize<object>(responseText)
        };
        
        context.Response.Body = originalBody;
        await context.Response.WriteAsJsonAsync(response);
    }
}
```

### Pattern 5: Unit of Work

```csharp
public interface IUnitOfWork : IDisposable
{
    IRepository<User> Users { get; }
    IRepository<Product> Products { get; }
    Task<int> SaveChangesAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    
    public IRepository<User> Users { get; }
    public IRepository<Product> Products { get; }
    
    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
        Users = new Repository<User>(context);
        Products = new Repository<Product>(context);
    }
    
    public async Task<int> SaveChangesAsync() =>
        await _context.SaveChangesAsync();
    
    public void Dispose() => _context.Dispose();
}

// Usage
public class OrderService
{
    private readonly IUnitOfWork _uow;
    
    public OrderService(IUnitOfWork uow) => _uow = uow;
    
    public async Task CreateOrderAsync(CreateOrderDto dto)
    {
        var user = await _uow.Users.GetByIdAsync(dto.UserId);
        var product = await _uow.Products.GetByIdAsync(dto.ProductId);
        
        // Create order...
        
        await _uow.SaveChangesAsync(); // Single transaction
    }
}
```

---

## Summary: Quick Reference Checklist

**When building ASP.NET Core applications:**

### Core Types to Remember:
- [ ] **HttpContext** - Access request/response
- [ ] **IServiceCollection** - Register services
- [ ] **IConfiguration** - Read settings
- [ ] **DbContext** - Database access
- [ ] **ControllerBase** - API controllers
- [ ] **ILogger<T>** - Logging
- [ ] **ClaimsPrincipal** - User identity

### Common Patterns:
- [ ] Dependency Injection (Transient, Scoped, Singleton)
- [ ] Options Pattern (IOptions<T>)
- [ ] Repository Pattern
- [ ] Unit of Work
- [ ] Result Pattern
- [ ] CRUD Controllers

### Essential Namespaces:
- [ ] Microsoft.AspNetCore.Http
- [ ] Microsoft.AspNetCore.Mvc
- [ ] Microsoft.Extensions.DependencyInjection
- [ ] Microsoft.Extensions.Configuration
- [ ] Microsoft.EntityFrameworkCore
- [ ] Microsoft.Extensions.Logging

### Key Attributes:
- [ ] [Route], [HttpGet], [HttpPost], [HttpPut], [HttpDelete]
- [ ] [FromBody], [FromRoute], [FromQuery]
- [ ] [Required], [StringLength], [Range], [EmailAddress]
- [ ] [Authorize], [AllowAnonymous]
- [ ] [ApiController]

---

**This completes the Common Interfaces, Classes & Namespaces Reference guide - your comprehensive desk reference for ASP.NET Core development!**
