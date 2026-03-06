# ASP.NET Core Middleware & HTTP Pipeline - Complete Guide
## Practical Guide + Technical Reference

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. What is Middleware
2. HTTP Request Pipeline Flow
3. Creating Custom Middleware (3 Methods)
4. Middleware Branching (Use, Run, Map)
5. Common Middleware Patterns
6. Built-in Middleware Quick Reference
7. Troubleshooting Common Issues
8. Best Practices

### Part 2: Technical Reference (Deep Dive)
9. Important Interfaces & Classes Reference
10. Configuration Deep-Dive
11. Built-in Middleware Configuration Details
12. Advanced Topics

---

# PART 1: PRACTICAL GUIDE

---

## 1. What is Middleware?

**Simple Definition:** Software components that handle HTTP requests and responses.

**Think of it like:** An assembly line where each worker (middleware) inspects or modifies a product (HTTP request) before passing it to the next worker.

```
Request → [Middleware 1] → [Middleware 2] → [Middleware 3] → Endpoint
             ↓                ↓                ↓                ↓
Response ← [Middleware 1] ← [Middleware 2] ← [Middleware 3] ← Handler
```

**Key Point:** Order matters! Request flows top to bottom, response flows bottom to top.

---

## 2. HTTP Request Pipeline Flow

### Visual Pipeline

```
1. HTTP Request arrives
   ↓
2. Exception Handler (catches all errors)
   ↓
3. HTTPS Redirection (HTTP → HTTPS)
   ↓
4. Static Files (CSS, JS, images)
   ↓
5. Routing (match URL to endpoint)
   ↓
6. CORS (cross-origin)
   ↓
7. Authentication (who are you?)
   ↓
8. Authorization (what can you do?)
   ↓
9. Custom Middleware (your logic)
   ↓
10. Endpoint (controller/handler)
    ↓
11. Response flows back through middleware
    ↓
12. HTTP Response sent to client
```

### Example: Standard Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

// ⚠️ ORDER MATTERS!

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
app.UseMyCustomMiddleware();
app.MapControllers();

app.Run();
```

---

## 3. Creating Custom Middleware (3 Methods)

### Method 1: Inline Lambda (Quick & Simple)

**When to use:**
- ✅ Simple logic (1-5 lines)
- ✅ Prototyping
- ❌ Complex logic
- ❌ Needs dependency injection

**Step 1: Use app.Use() with Lambda**

```csharp
app.Use(async (context, next) =>
{
    // Before next middleware
    Console.WriteLine($"Request: {context.Request.Path}");
    
    await next(); // Call next middleware
    
    // After next middleware (response phase)
    Console.WriteLine($"Response: {context.Response.StatusCode}");
});
```

**Complete Example:**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Request/Response logging
app.Use(async (context, next) =>
{
    var startTime = DateTime.UtcNow;
    
    Console.WriteLine($"[{startTime:HH:mm:ss}] {context.Request.Method} {context.Request.Path}");
    
    await next();
    
    var duration = (DateTime.UtcNow - startTime).TotalMilliseconds;
    Console.WriteLine($"Completed in {duration}ms");
});

app.MapGet("/test", () => "Hello World");

app.Run();
```

---

### Method 2: Middleware Class (Production)

**When to use:**
- ✅ Complex logic (10+ lines)
- ✅ Reusable
- ✅ Production code
- ✅ Needs logging/DI

**Step 1: Create Middleware Class**

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;
    
    public RequestLoggingMiddleware(
        RequestDelegate next,
        ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var startTime = DateTime.UtcNow;
        
        _logger.LogInformation($"{context.Request.Method} {context.Request.Path}");
        
        await _next(context);
        
        var duration = (DateTime.UtcNow - startTime).TotalMilliseconds;
        _logger.LogInformation($"Completed in {duration}ms");
    }
}
```

**Step 2: Create Extension Method**

```csharp
public static class RequestLoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestLoggingMiddleware>();
    }
}
```

**Step 3: Use in Program.cs**

```csharp
app.UseRequestLogging(); // Clean syntax!
```

---

### Method 3: IMiddleware Interface (Scoped Dependencies)

**When to use:**
- ✅ Need DbContext (scoped)
- ✅ Middleware per request
- ✅ Advanced DI scenarios

**Step 1: Implement IMiddleware**

```csharp
public class ApiKeyValidationMiddleware : IMiddleware
{
    private readonly ApplicationDbContext _context; // Scoped!
    private readonly ILogger<ApiKeyValidationMiddleware> _logger;
    
    public ApiKeyValidationMiddleware(
        ApplicationDbContext context,
        ILogger<ApiKeyValidationMiddleware> logger)
    {
        _context = context;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        if (!context.Request.Headers.TryGetValue("X-API-Key", out var apiKey))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("API Key required");
            return;
        }
        
        var isValid = await _context.ApiKeys
            .AnyAsync(k => k.Key == apiKey.ToString() && k.IsActive);
        
        if (!isValid)
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Invalid API Key");
            return;
        }
        
        await next(context);
    }
}
```

**Step 2: Register in DI** ⚠️ REQUIRED

```csharp
builder.Services.AddScoped<ApiKeyValidationMiddleware>();
```

**Step 3: Use in Pipeline**

```csharp
app.UseMiddleware<ApiKeyValidationMiddleware>();
```

---

### Method 3B: IMiddleware with Add/Use Pattern (Best Practice) ⭐ RECOMMENDED

**When to use:**
- ✅ Production code
- ✅ Reusable middleware packages
- ✅ Follows ASP.NET Core conventions
- ✅ Clean, discoverable API

**The Pattern:**
ASP.NET Core uses a consistent pattern for all features:
- `AddXxx()` - Register services and configure (on IServiceCollection)
- `UseXxx()` - Add to pipeline (on IApplicationBuilder)

**Examples from ASP.NET Core:**
```csharp
// Authentication
builder.Services.AddAuthentication();  // Register
app.UseAuthentication();                // Use

// CORS
builder.Services.AddCors();            // Register
app.UseCors();                          // Use

// Controllers
builder.Services.AddControllers();     // Register
app.MapControllers();                   // Use
```

**Step 1: Create Your Middleware (Same as Method 3)**

```csharp
public class ApiKeyValidationMiddleware : IMiddleware
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<ApiKeyValidationMiddleware> _logger;
    
    public ApiKeyValidationMiddleware(
        ApplicationDbContext context,
        ILogger<ApiKeyValidationMiddleware> logger)
    {
        _context = context;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        if (!context.Request.Headers.TryGetValue("X-API-Key", out var apiKey))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("API Key required");
            return;
        }
        
        var isValid = await _context.ApiKeys
            .AnyAsync(k => k.Key == apiKey.ToString() && k.IsActive);
        
        if (!isValid)
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Invalid API Key");
            return;
        }
        
        await next(context);
    }
}
```

**Step 2: Create Extension Methods Class**

```csharp
public static class ApiKeyValidationExtensions
{
    // Register services in DI container
    public static IServiceCollection AddApiKeyValidation(
        this IServiceCollection services)
    {
        services.AddScoped<ApiKeyValidationMiddleware>();
        return services;
    }
    
    // Add middleware to pipeline
    public static IApplicationBuilder UseApiKeyValidation(
        this IApplicationBuilder app)
    {
        return app.UseMiddleware<ApiKeyValidationMiddleware>();
    }
}
```

**Step 3: Use in Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register
builder.Services.AddApiKeyValidation();  // ✅ Clean!

var app = builder.Build();

// Use
app.UseApiKeyValidation();  // ✅ Discoverable!

app.Run();
```

---

### Advanced: Add/Use Pattern with Configuration

**When you need configuration options:**

**Step 1: Create Options Class**

```csharp
public class ApiKeyValidationOptions
{
    public string HeaderName { get; set; } = "X-API-Key";
    public bool RequireHttps { get; set; } = true;
    public string[] ExcludedPaths { get; set; } = Array.Empty<string>();
}
```

**Step 2: Update Middleware to Accept Options**

```csharp
public class ApiKeyValidationMiddleware : IMiddleware
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<ApiKeyValidationMiddleware> _logger;
    private readonly ApiKeyValidationOptions _options;
    
    public ApiKeyValidationMiddleware(
        ApplicationDbContext context,
        ILogger<ApiKeyValidationMiddleware> logger,
        IOptions<ApiKeyValidationOptions> options)
    {
        _context = context;
        _logger = logger;
        _options = options.Value;
    }
    
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        // Check excluded paths
        if (_options.ExcludedPaths.Any(p => context.Request.Path.StartsWithSegments(p)))
        {
            await next(context);
            return;
        }
        
        // Require HTTPS if configured
        if (_options.RequireHttps && !context.Request.IsHttps)
        {
            context.Response.StatusCode = 400;
            await context.Response.WriteAsync("HTTPS required");
            return;
        }
        
        // Get header (use configured name)
        if (!context.Request.Headers.TryGetValue(_options.HeaderName, out var apiKey))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("API Key required");
            return;
        }
        
        var isValid = await _context.ApiKeys
            .AnyAsync(k => k.Key == apiKey.ToString() && k.IsActive);
        
        if (!isValid)
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Invalid API Key");
            return;
        }
        
        await next(context);
    }
}
```

**Step 3: Update Extension Methods with Configuration**

```csharp
public static class ApiKeyValidationExtensions
{
    // Method 1: Default configuration
    public static IServiceCollection AddApiKeyValidation(
        this IServiceCollection services)
    {
        services.AddScoped<ApiKeyValidationMiddleware>();
        services.Configure<ApiKeyValidationOptions>(options => { });
        return services;
    }
    
    // Method 2: Configuration delegate
    public static IServiceCollection AddApiKeyValidation(
        this IServiceCollection services,
        Action<ApiKeyValidationOptions> configure)
    {
        services.AddScoped<ApiKeyValidationMiddleware>();
        services.Configure(configure);
        return services;
    }
    
    // Method 3: From IConfiguration (appsettings.json)
    public static IServiceCollection AddApiKeyValidation(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddScoped<ApiKeyValidationMiddleware>();
        services.Configure<ApiKeyValidationOptions>(
            configuration.GetSection("ApiKeyValidation"));
        return services;
    }
    
    // Add to pipeline (same for all)
    public static IApplicationBuilder UseApiKeyValidation(
        this IApplicationBuilder app)
    {
        return app.UseMiddleware<ApiKeyValidationMiddleware>();
    }
}
```

**Step 4: Usage - Three Configuration Methods**

**Method 1: Default Configuration**
```csharp
builder.Services.AddApiKeyValidation();
app.UseApiKeyValidation();
```

**Method 2: Code-based Configuration**
```csharp
builder.Services.AddApiKeyValidation(options =>
{
    options.HeaderName = "X-Custom-API-Key";
    options.RequireHttps = false; // Dev only!
    options.ExcludedPaths = new[] { "/health", "/metrics" };
});

app.UseApiKeyValidation();
```

**Method 3: appsettings.json Configuration**

```json
{
  "ApiKeyValidation": {
    "HeaderName": "X-API-Key",
    "RequireHttps": true,
    "ExcludedPaths": ["/health", "/swagger"]
  }
}
```

```csharp
builder.Services.AddApiKeyValidation(builder.Configuration);
app.UseApiKeyValidation();
```

---

### Complete Example: Request Logging with Add/Use Pattern

**Full Implementation:**

```csharp
// 1. Options Class
public class RequestLoggingOptions
{
    public bool LogHeaders { get; set; } = false;
    public bool LogBody { get; set; } = false;
    public int MaxBodyLength { get; set; } = 1000;
    public string[] ExcludedPaths { get; set; } = Array.Empty<string>();
}

// 2. Middleware
public class RequestLoggingMiddleware : IMiddleware
{
    private readonly ILogger<RequestLoggingMiddleware> _logger;
    private readonly RequestLoggingOptions _options;
    
    public RequestLoggingMiddleware(
        ILogger<RequestLoggingMiddleware> logger,
        IOptions<RequestLoggingOptions> options)
    {
        _logger = logger;
        _options = options.Value;
    }
    
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        // Check excluded paths
        if (_options.ExcludedPaths.Any(p => context.Request.Path.StartsWithSegments(p)))
        {
            await next(context);
            return;
        }
        
        var startTime = DateTime.UtcNow;
        
        // Log request
        _logger.LogInformation(
            "Request: {Method} {Path}",
            context.Request.Method,
            context.Request.Path);
        
        // Log headers if enabled
        if (_options.LogHeaders)
        {
            foreach (var header in context.Request.Headers)
            {
                _logger.LogDebug("Header: {Key} = {Value}", header.Key, header.Value);
            }
        }
        
        // Log body if enabled (careful with this!)
        if (_options.LogBody && context.Request.ContentLength > 0)
        {
            context.Request.EnableBuffering();
            using var reader = new StreamReader(context.Request.Body, leaveOpen: true);
            var body = await reader.ReadToEndAsync();
            body = body.Length > _options.MaxBodyLength 
                ? body.Substring(0, _options.MaxBodyLength) + "..."
                : body;
            _logger.LogDebug("Body: {Body}", body);
            context.Request.Body.Position = 0;
        }
        
        await next(context);
        
        var duration = (DateTime.UtcNow - startTime).TotalMilliseconds;
        
        // Log response
        _logger.LogInformation(
            "Response: {StatusCode} in {Duration}ms",
            context.Response.StatusCode,
            duration);
    }
}

// 3. Extension Methods
public static class RequestLoggingExtensions
{
    // Default configuration
    public static IServiceCollection AddRequestLogging(
        this IServiceCollection services)
    {
        services.AddScoped<RequestLoggingMiddleware>();
        services.Configure<RequestLoggingOptions>(options => { });
        return services;
    }
    
    // With configuration delegate
    public static IServiceCollection AddRequestLogging(
        this IServiceCollection services,
        Action<RequestLoggingOptions> configure)
    {
        services.AddScoped<RequestLoggingMiddleware>();
        services.Configure(configure);
        return services;
    }
    
    // From IConfiguration
    public static IServiceCollection AddRequestLogging(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddScoped<RequestLoggingMiddleware>();
        services.Configure<RequestLoggingOptions>(
            configuration.GetSection("RequestLogging"));
        return services;
    }
    
    // Add to pipeline
    public static IApplicationBuilder UseRequestLogging(
        this IApplicationBuilder app)
    {
        return app.UseMiddleware<RequestLoggingMiddleware>();
    }
}

// 4. Usage in Program.cs

// Option A: Default
builder.Services.AddRequestLogging();
app.UseRequestLogging();

// Option B: Configure in code
builder.Services.AddRequestLogging(options =>
{
    options.LogHeaders = true;
    options.ExcludedPaths = new[] { "/health", "/metrics" };
});
app.UseRequestLogging();

// Option C: Configure from appsettings.json
// appsettings.json:
// {
//   "RequestLogging": {
//     "LogHeaders": false,
//     "ExcludedPaths": ["/health", "/swagger"]
//   }
// }
builder.Services.AddRequestLogging(builder.Configuration);
app.UseRequestLogging();
```

---

### Comparison: Direct Registration vs Add/Use Pattern

| Approach | Code | Pros | Cons |
|----------|------|------|------|
| **Direct** | `services.AddScoped<MyMiddleware>();`<br/>`app.UseMiddleware<MyMiddleware>();` | Simple, less code | Not discoverable, no configuration, verbose |
| **Add/Use** ⭐ | `services.AddMyMiddleware();`<br/>`app.UseMyMiddleware();` | Clean, discoverable, configurable, follows conventions | Requires extension methods |

**When to Use Add/Use Pattern:**
- ✅ Production middleware
- ✅ Reusable libraries
- ✅ Middleware needs configuration
- ✅ Following ASP.NET Core conventions
- ✅ IntelliSense discoverability important

**When Direct Registration is OK:**
- ⚠️ Quick prototyping
- ⚠️ Internal middleware (not shared)
- ⚠️ No configuration needed
- ⚠️ Simple middleware

---

### Why This Pattern Matters

**1. Discoverability**
```csharp
builder.Services.Add[TAB] // IntelliSense shows all available services
app.Use[TAB]              // IntelliSense shows all middleware
```

**2. Consistency**
All ASP.NET Core features follow this pattern:
- Authentication, Authorization, CORS, Controllers, MVC, etc.

**3. Configuration**
Clean place to handle options and dependencies

**4. Separation of Concerns**
- `AddXxx()` - DI registration and configuration
- `UseXxx()` - Pipeline registration

**5. Testability**
Easy to mock and test services separately from pipeline

---

### Best Practice Summary

**For Production Middleware:**

1. ✅ Use IMiddleware interface
2. ✅ Create extension methods (Add + Use)
3. ✅ Support configuration options
4. ✅ Register as Scoped (if using DbContext)
5. ✅ Follow naming conventions (AddXxx, UseXxx)
6. ✅ Return IServiceCollection and IApplicationBuilder for chaining
7. ✅ Provide overloads for different configuration methods

**Template to Follow:**
```csharp
// Options (if needed)
public class XxxOptions { }

// Middleware
public class XxxMiddleware : IMiddleware { }

// Extensions
public static class XxxExtensions
{
    public static IServiceCollection AddXxx(this IServiceCollection services) { }
    public static IApplicationBuilder UseXxx(this IApplicationBuilder app) { }
}

// Usage
builder.Services.AddXxx();
app.UseXxx();
```

---

### Comparison: Which Method?

| Feature | Lambda | Class | IMiddleware |
|---------|--------|-------|-------------|
| Complexity | Simple | Medium-Complex | Complex |
| Reusability | No | Yes | Yes |
| DI Dependencies | Singleton only | Singleton only | Scoped/Transient ✅ |
| Testing | Hard | Easy | Easy |
| Performance | Fast | Fast | Slightly slower |

**Decision Tree:**
```
Need DbContext/Scoped service?
├─ YES → Method 3 (IMiddleware)
└─ NO → Complex logic?
         ├─ YES → Method 2 (Class)
         └─ NO → Method 1 (Lambda)
```

---

## 4. Middleware Branching

### Use() - Chain Middleware

```csharp
app.Use(async (context, next) =>
{
    // Do something
    await next(); // Continue pipeline
    // Do something after
});
```

### Run() - Terminal Middleware

```csharp
app.Run(async context =>
{
    await context.Response.WriteAsync("Pipeline ends");
    // No next() - stops here
});
```

### Map() - Branch by Path

```csharp
app.Map("/api", apiApp =>
{
    apiApp.MapGet("/users", () => new[] { "Alice", "Bob" });
});

// GET /api/users → returns users
// GET /other → not matched
```

### MapWhen() - Conditional Branch

```csharp
app.MapWhen(
    context => context.Request.Query.ContainsKey("preview"),
    previewApp =>
    {
        previewApp.Run(async context =>
        {
            await context.Response.WriteAsync("Preview mode");
        });
    }
);
```

**Complete Branching Example:**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Main pipeline
app.Use(async (context, next) =>
{
    Console.WriteLine("Main pipeline");
    await next();
});

// Branch 1: /api
app.Map("/api", apiApp =>
{
    apiApp.MapGet("/users", () => new[] { "Alice", "Bob" });
});

// Branch 2: /admin
app.Map("/admin", adminApp =>
{
    adminApp.MapGet("/dashboard", () => "Admin Dashboard");
});

// Default
app.MapGet("/", () => "Home");

app.Run();
```

---

## 5. Common Middleware Patterns

### Pattern 1: Correlation ID

```csharp
app.Use(async (context, next) =>
{
    var correlationId = Guid.NewGuid().ToString();
    context.Items["CorrelationId"] = correlationId;
    context.Response.Headers.Add("X-Correlation-ID", correlationId);
    
    await next();
});
```

### Pattern 2: Request Timing

```csharp
app.Use(async (context, next) =>
{
    var sw = Stopwatch.StartNew();
    
    await next();
    
    sw.Stop();
    context.Response.Headers.Add("X-Response-Time", $"{sw.ElapsedMilliseconds}ms");
});
```

### Pattern 3: Error Handling

```csharp
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;
    
    public GlobalExceptionMiddleware(RequestDelegate next, ILogger<GlobalExceptionMiddleware> logger)
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
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception");
            
            context.Response.StatusCode = 500;
            context.Response.ContentType = "application/json";
            
            await context.Response.WriteAsJsonAsync(new
            {
                error = "An error occurred",
                message = ex.Message
            });
        }
    }
}
```

### Pattern 4: Simple Rate Limiting

```csharp
public class SimpleRateLimitMiddleware
{
    private readonly RequestDelegate _next;
    private static readonly Dictionary<string, (int count, DateTime reset)> _requests = new();
    private readonly int _maxRequests = 10;
    private readonly TimeSpan _window = TimeSpan.FromMinutes(1);
    
    public SimpleRateLimitMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var ip = context.Connection.RemoteIpAddress?.ToString() ?? "unknown";
        
        lock (_requests)
        {
            if (_requests.TryGetValue(ip, out var info))
            {
                if (DateTime.UtcNow < info.reset)
                {
                    if (info.count >= _maxRequests)
                    {
                        context.Response.StatusCode = 429;
                        await context.Response.WriteAsync("Rate limit exceeded");
                        return;
                    }
                    _requests[ip] = (info.count + 1, info.reset);
                }
                else
                {
                    _requests[ip] = (1, DateTime.UtcNow.Add(_window));
                }
            }
            else
            {
                _requests[ip] = (1, DateTime.UtcNow.Add(_window));
            }
        }
        
        await _next(context);
    }
}
```

---

## 6. Built-in Middleware Quick Reference

| Middleware | Position | Purpose |
|-----------|----------|---------|
| `UseDeveloperExceptionPage()` | First | Detailed errors (dev only) |
| `UseExceptionHandler()` | First | Global error handler (production) |
| `UseHsts()` | After exception | HTTP Strict Transport Security |
| `UseHttpsRedirection()` | Early | Redirect HTTP → HTTPS |
| `UseStaticFiles()` | Before routing | Serve wwwroot files |
| `UseRouting()` | Before auth | Enable endpoint routing |
| `UseCors()` | After routing | Cross-origin requests |
| `UseAuthentication()` | Before authorization | Authenticate user |
| `UseAuthorization()` | After authentication | Check permissions |
| `UseSession()` | After routing | Enable sessions |
| `MapControllers()` | Last | Map controller endpoints |

---

## 7. Troubleshooting Common Issues

### Issue 1: Wrong Middleware Order

**Problem:**
```csharp
app.UseAuthorization(); // ❌ Before authentication
app.UseAuthentication();
```

**Solution:**
```csharp
app.UseAuthentication(); // ✅ Auth first
app.UseAuthorization();  // Then authorization
```

### Issue 2: Response Already Started

**Problem:**
```csharp
app.Use(async (context, next) =>
{
    await next();
    context.Response.Headers.Add("X-Custom", "Value"); // ❌ Too late!
});
```

**Solution:**
```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("X-Custom", "Value"); // ✅ Before next()
    await next();
});
```

### Issue 3: Scoped Service in Singleton Middleware

**Problem:**
```csharp
public class MyMiddleware
{
    private readonly ApplicationDbContext _context; // ❌ Captive dependency!
    
    public MyMiddleware(RequestDelegate next, ApplicationDbContext context)
    {
        _context = context;
    }
}
```

**Solution: Use IMiddleware**
```csharp
public class MyMiddleware : IMiddleware
{
    private readonly ApplicationDbContext _context; // ✅ Now scoped!
    
    public MyMiddleware(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        await next(context);
    }
}

builder.Services.AddScoped<MyMiddleware>();
```

---

## 8. Best Practices

- ✅ Order middleware correctly (Exception → HTTPS → Static → Routing → Auth)
- ✅ Use extension methods for custom middleware
- ✅ Keep middleware focused (single responsibility)
- ✅ Use IMiddleware for scoped dependencies (DbContext)
- ✅ Don't modify response after next() (use OnStarting)
- ✅ Short-circuit when appropriate (don't call next if handled)
- ✅ Use ILogger, not Console.WriteLine
- ✅ Handle errors gracefully (try-catch)

---

# PART 2: TECHNICAL REFERENCE

---

## 9. Important Interfaces & Classes Reference

### IMiddleware Interface ✨ ASP.NET Core 2.0+

**Purpose:** Factory-based middleware creation (supports scoped/transient lifetimes)

**Namespace:** `Microsoft.AspNetCore.Http`

**Declaration:**
```csharp
public interface IMiddleware
{
    Task InvokeAsync(HttpContext context, RequestDelegate next);
}
```

**Key Points:**
- Allows scoped/transient lifetime (unlike traditional middleware which is singleton)
- Must be registered in DI container
- Slightly slower than traditional middleware
- Use when you need DbContext or other scoped services

**Full Example:**
```csharp
public class DatabaseHealthCheckMiddleware : IMiddleware
{
    private readonly ApplicationDbContext _context; // Scoped!
    
    public DatabaseHealthCheckMiddleware(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        var canConnect = await _context.Database.CanConnectAsync();
        
        if (!canConnect)
        {
            context.Response.StatusCode = 503;
            await context.Response.WriteAsync("Database unavailable");
            return;
        }
        
        await next(context);
    }
}

// Registration (REQUIRED)
builder.Services.AddScoped<DatabaseHealthCheckMiddleware>();
app.UseMiddleware<DatabaseHealthCheckMiddleware>();
```

---

### RequestDelegate Delegate

**Purpose:** Represents the next middleware in the pipeline

**Declaration:**
```csharp
public delegate Task RequestDelegate(HttpContext context);
```

**Usage:**
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
        // Before
        await _next(context); // Call next middleware
        // After
    }
}
```

**Key Points:**
- Always injected by framework in constructor
- Calling `await _next(context)` continues pipeline
- Not calling `_next` short-circuits the pipeline

---

### HttpContext Class ⭐⭐⭐

**Purpose:** Encapsulates all HTTP request/response information

**Namespace:** `Microsoft.AspNetCore.Http`

**Key Members:**

| Member | Type | Purpose |
|--------|------|---------|
| `Request` | HttpRequest | Incoming HTTP request |
| `Response` | HttpResponse | Outgoing HTTP response |
| `User` | ClaimsPrincipal | Authenticated user |
| `Items` | IDictionary | Request-scoped data storage |
| `RequestServices` | IServiceProvider | Access DI services |
| `Connection` | ConnectionInfo | Connection details |
| `RequestAborted` | CancellationToken | Request cancellation |
| `Session` | ISession | Session data |
| `TraceIdentifier` | string | Unique request ID |

**Common Usage Patterns:**

**Access Request Information:**
```csharp
var path = context.Request.Path;              // /api/users
var method = context.Request.Method;          // GET
var query = context.Request.Query["id"];      // Query string param
var header = context.Request.Headers["X-API-Key"]; // Header
var isHttps = context.Request.IsHttps;        // true/false
```

**Modify Response:**
```csharp
context.Response.StatusCode = 200;
context.Response.ContentType = "application/json";
context.Response.Headers.Add("X-Custom", "Value");
await context.Response.WriteAsync("Hello");
await context.Response.WriteAsJsonAsync(new { status = "ok" });
```

**Access User Information:**
```csharp
if (context.User?.Identity?.IsAuthenticated == true)
{
    var userName = context.User.Identity.Name;
    var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var isAdmin = context.User.IsInRole("Admin");
}
```

**Store Request-Scoped Data:**
```csharp
// Middleware 1
context.Items["StartTime"] = DateTime.UtcNow;
context.Items["CorrelationId"] = Guid.NewGuid().ToString();

// Middleware 2
var startTime = (DateTime)context.Items["StartTime"];
var duration = DateTime.UtcNow - startTime;
```

**Resolve Services:**
```csharp
var logger = context.RequestServices.GetRequiredService<ILogger<MyClass>>();
var dbContext = context.RequestServices.GetRequiredService<ApplicationDbContext>();
var config = context.RequestServices.GetService<IConfiguration>();
```

---

### HttpRequest Class ⭐⭐⭐

**Purpose:** Represents incoming HTTP request

**Key Members:**

| Member | Type | Description |
|--------|------|-------------|
| `Method` | string | HTTP verb (GET, POST, etc.) |
| `Scheme` | string | http or https |
| `Host` | HostString | example.com:443 |
| `Path` | PathString | /api/users |
| `QueryString` | QueryString | ?id=5&page=2 |
| `Query` | IQueryCollection | Parsed query parameters |
| `Headers` | IHeaderDictionary | HTTP headers |
| `Cookies` | IRequestCookieCollection | Request cookies |
| `ContentType` | string | Content-Type header |
| `ContentLength` | long? | Request body size |
| `Body` | Stream | Request body stream |
| `Form` | IFormCollection | Form data (POST) |
| `IsHttps` | bool | Is HTTPS request |

**Reading URL Components:**
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
```

**Reading Headers:**
```csharp
var userAgent = context.Request.Headers["User-Agent"].ToString();
var apiKey = context.Request.Headers["X-API-Key"].ToString();

// Check if header exists
if (context.Request.Headers.TryGetValue("Authorization", out var auth))
{
    var token = auth.ToString();
}
```

**Reading Request Body:**
```csharp
// Enable multiple reads
context.Request.EnableBuffering();

// Read as text
using var reader = new StreamReader(context.Request.Body);
var body = await reader.ReadToEndAsync();
context.Request.Body.Position = 0; // Reset for next middleware

// Read as JSON
var user = await context.Request.ReadFromJsonAsync<User>();
```

---

### HttpResponse Class ⭐⭐⭐

**Purpose:** Represents outgoing HTTP response

**Key Members:**

| Member | Type | Description |
|--------|------|-------------|
| `StatusCode` | int | HTTP status code (200, 404, etc.) |
| `Headers` | IHeaderDictionary | Response headers |
| `ContentType` | string | Content-Type header |
| `ContentLength` | long? | Response body size |
| `Body` | Stream | Response body stream |
| `Cookies` | IResponseCookies | Response cookies |
| `HasStarted` | bool | Has response started sending |

**Methods:**

| Method | Purpose |
|--------|---------|
| `WriteAsync(string)` | Write text to response |
| `WriteAsJsonAsync<T>(T)` | Write JSON to response |
| `Redirect(string)` | Redirect to URL |
| `OnStarting(Func<Task>)` | Execute before response starts |
| `OnCompleted(Func<Task>)` | Execute after response completes |

**Setting Status and Headers:**
```csharp
context.Response.StatusCode = 200;
context.Response.ContentType = "application/json";
context.Response.Headers.Add("X-Correlation-ID", correlationId);
context.Response.Headers.Add("Cache-Control", "no-cache");
```

**Writing Response:**
```csharp
// Text
await context.Response.WriteAsync("Hello World");

// JSON
var data = new { message = "Success", timestamp = DateTime.UtcNow };
await context.Response.WriteAsJsonAsync(data);

// Binary
var bytes = await File.ReadAllBytesAsync("file.pdf");
context.Response.ContentType = "application/pdf";
await context.Response.Body.WriteAsync(bytes);
```

**Setting Cookies:**
```csharp
context.Response.Cookies.Append("SessionId", sessionId, new CookieOptions
{
    HttpOnly = true,
    Secure = true,
    SameSite = SameSiteMode.Strict,
    Expires = DateTimeOffset.UtcNow.AddHours(1)
});
```

**Response Callbacks:**
```csharp
// Before response starts
context.Response.OnStarting(() =>
{
    context.Response.Headers.Add("X-Server", "MyServer");
    return Task.CompletedTask;
});

// After response completed
context.Response.OnCompleted(() =>
{
    Console.WriteLine("Response sent");
    return Task.CompletedTask;
});
```

---

### IApplicationBuilder Interface ⭐⭐⭐

**Purpose:** Defines the middleware pipeline

**Namespace:** `Microsoft.AspNetCore.Builder`

**Key Methods:**

| Method | Purpose |
|--------|---------|
| `Use(Func<RequestDelegate, RequestDelegate>)` | Add middleware |
| `Run(RequestDelegate)` | Terminal middleware |
| `Map(PathString, Action<IApplicationBuilder>)` | Branch pipeline |
| `MapWhen(Func<HttpContext, bool>, Action<IApplicationBuilder>)` | Conditional branch |
| `UseMiddleware<T>()` | Add typed middleware |

**Extension Method Pattern:**
```csharp
public static class MyMiddlewareExtensions
{
    public static IApplicationBuilder UseMyMiddleware(
        this IApplicationBuilder app)
    {
        return app.UseMiddleware<MyMiddleware>();
    }
    
    // With configuration
    public static IApplicationBuilder UseMyMiddleware(
        this IApplicationBuilder app,
        Action<MyOptions> configure)
    {
        var options = new MyOptions();
        configure?.Invoke(options);
        return app.UseMiddleware<MyMiddleware>(options);
    }
}

// Usage
app.UseMyMiddleware();
app.UseMyMiddleware(options => options.Timeout = 30);
```

---

## 10. Configuration Deep-Dive

### Pattern 1: Inline Configuration (No Options)

**When to use:** Simple, hardcoded configuration

```csharp
app.Use(async (context, next) =>
{
    // Configuration is hardcoded
    var timeout = 30;
    var enabled = true;
    
    if (enabled)
    {
        await next();
    }
});
```

**Pros:** Simple, quick
**Cons:** Not configurable, not reusable

---

### Pattern 2: Constructor Options (Code Configuration)

**When to use:** Reusable middleware with code-based configuration

**Step 1: Create Options Class**
```csharp
public class RequestLoggingOptions
{
    public bool LogHeaders { get; set; } = false;
    public bool LogBody { get; set; } = false;
    public int MaxBodyLength { get; set; } = 1000;
    public string[] ExcludedPaths { get; set; } = Array.Empty<string>();
}
```

**Step 2: Middleware Accepts Options**
```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;
    private readonly RequestLoggingOptions _options;
    
    public RequestLoggingMiddleware(
        RequestDelegate next,
        ILogger<RequestLoggingMiddleware> logger,
        RequestLoggingOptions options)
    {
        _next = next;
        _logger = logger;
        _options = options;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        if (_options.ExcludedPaths.Any(p => context.Request.Path.StartsWithSegments(p)))
        {
            await _next(context);
            return;
        }
        
        _logger.LogInformation($"Request: {context.Request.Path}");
        
        if (_options.LogHeaders)
        {
            foreach (var header in context.Request.Headers)
            {
                _logger.LogDebug($"Header: {header.Key} = {header.Value}");
            }
        }
        
        await _next(context);
    }
}
```

**Step 3: Extension Method with Configuration**
```csharp
public static class RequestLoggingExtensions
{
    public static IApplicationBuilder UseRequestLogging(
        this IApplicationBuilder app,
        Action<RequestLoggingOptions> configure = null)
    {
        var options = new RequestLoggingOptions();
        configure?.Invoke(options);
        
        return app.UseMiddleware<RequestLoggingMiddleware>(options);
    }
}
```

**Step 4: Usage**
```csharp
// Default options
app.UseRequestLogging();

// Custom options
app.UseRequestLogging(options =>
{
    options.LogHeaders = true;
    options.LogBody = true;
    options.ExcludedPaths = new[] { "/health", "/metrics" };
});
```

---

### Pattern 3: IOptions Pattern (appsettings.json)

**When to use:** Production apps, configuration from files

**Step 1: Options Class with Section Name**
```csharp
public class RequestLoggingOptions
{
    public const string SectionName = "RequestLogging";
    
    public bool Enabled { get; set; } = true;
    public bool LogHeaders { get; set; } = false;
    public string[] ExcludedPaths { get; set; } = Array.Empty<string>();
}
```

**Step 2: appsettings.json**
```json
{
  "RequestLogging": {
    "Enabled": true,
    "LogHeaders": false,
    "ExcludedPaths": ["/health", "/metrics"]
  }
}
```

**Step 3: Register in DI**
```csharp
builder.Services.Configure<RequestLoggingOptions>(
    builder.Configuration.GetSection(RequestLoggingOptions.SectionName));
```

**Step 4: Inject IOptions**
```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly RequestLoggingOptions _options;
    
    public RequestLoggingMiddleware(
        RequestDelegate next,
        IOptions<RequestLoggingOptions> options)
    {
        _next = next;
        _options = options.Value; // Get value from IOptions
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        if (!_options.Enabled)
        {
            await _next(context);
            return;
        }
        
        // Use options...
    }
}
```

**Step 5: Use Middleware**
```csharp
app.UseMiddleware<RequestLoggingMiddleware>();
// Configuration loaded from appsettings.json automatically
```

---

### Configuration Comparison Table

| Approach | Source | Reload | Complexity | Production |
|----------|--------|--------|------------|-----------|
| Inline | Hardcoded | No | Simple | ❌ No |
| Constructor Options | Code | No | Medium | ⚠️ Maybe |
| IOptions | appsettings.json | No | Medium | ✅ Yes |
| IOptionsSnapshot | appsettings.json | Per request | Medium | ✅ Yes |
| IOptionsMonitor | appsettings.json | Live | Complex | ✅ Yes |

---

## 11. Built-in Middleware Configuration Details

### CORS Configuration

**Important Classes:**
- `CorsOptions` - Configure policies
- `CorsPolicyBuilder` - Build CORS policy

**Method 1: Default Policy**
```csharp
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins("https://example.com")
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

app.UseCors(); // Uses default policy
```

**Method 2: Named Policies**
```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("StrictPolicy", policy =>
    {
        policy.WithOrigins("https://example.com")
              .WithMethods("GET", "POST")
              .WithHeaders("Content-Type");
    });
    
    options.AddPolicy("RelaxedPolicy", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

app.UseCors("StrictPolicy");
```

**Method 3: From Configuration**
```json
{
  "Cors": {
    "AllowedOrigins": ["https://example.com", "https://app.example.com"]
  }
}
```

```csharp
var origins = builder.Configuration.GetSection("Cors:AllowedOrigins").Get<string[]>();

builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins(origins)
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});
```

---

### Static Files Configuration

**Important Class:** `StaticFileOptions`

**Method 1: Default (wwwroot)**
```csharp
app.UseStaticFiles();
```

**Method 2: Custom Directory**
```csharp
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(Directory.GetCurrentDirectory(), "StaticFiles")),
    RequestPath = "/files"
});
```

**Method 3: Custom Headers**
```csharp
app.UseStaticFiles(new StaticFileOptions
{
    OnPrepareResponse = ctx =>
    {
        ctx.Context.Response.Headers.Append("Cache-Control", "public,max-age=31536000");
    }
});
```

---

### Authentication Configuration

**Important Classes:**
- `AuthenticationOptions` - Global settings
- `JwtBearerOptions` - JWT configuration

**JWT Bearer Setup:**
```csharp
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
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });

app.UseAuthentication();
app.UseAuthorization();
```

---

## 12. Advanced Topics

### Short-Circuit Middleware ✨ ASP.NET Core 8.0+

**Purpose:** Skip remaining middleware

```csharp
app.Use(async (context, next) =>
{
    if (context.Request.Path == "/health")
    {
        context.Response.StatusCode = 200;
        await context.Response.WriteAsync("OK");
        return; // Short-circuit - don't call next()
    }
    
    await next();
});
```

### Middleware Order Visualization

```
Exception Handler ─┐
                   │
HTTPS Redirect ────┤
                   │
Static Files ──────┤
                   │
Routing ───────────┤
                   │
CORS ──────────────┤
                   ├─→ [Request Processing]
Authentication ───┤
                   │
Authorization ─────┤
                   │
Custom ────────────┤
                   │
Endpoints ─────────┘
```

### Performance Tips

1. **Order matters for performance:**
   - Put cheap middleware first (static files)
   - Put expensive middleware last (database checks)

2. **Short-circuit when possible:**
   - Health checks don't need authentication
   - Static files don't need routing

3. **Use IMiddleware sparingly:**
   - Slight performance overhead
   - Only when you need scoped dependencies

---

## Summary: Complete Middleware Checklist

**Creating Middleware:**
- [ ] Choose method: Lambda / Class / IMiddleware
- [ ] Implement InvokeAsync with HttpContext and RequestDelegate
- [ ] Handle errors with try-catch
- [ ] Create extension method for clean usage

**Configuration:**
- [ ] Decide: Inline / Constructor Options / IOptions
- [ ] Create options class if needed
- [ ] Add to appsettings.json for production
- [ ] Register in DI if using IOptions

**Usage:**
- [ ] Order middleware correctly
- [ ] Add to Program.cs
- [ ] Test with different scenarios

**Best Practices:**
- [ ] Keep focused (single responsibility)
- [ ] Use ILogger for logging
- [ ] Don't modify response after next()
- [ ] Handle short-circuiting properly

---

**This completes the Middleware guide combining practical hands-on content with deep technical reference!**
