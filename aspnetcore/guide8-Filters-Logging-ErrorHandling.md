# ASP.NET Core Filters, Logging & Error Handling - Complete Guide
## Practical Guide + Technical Reference
**Version: .NET 9.0 | C# 13 | December 2024**

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. Filters Overview
2. The 5 Filter Types
3. Creating Filters (3 Methods)
4. Logging Fundamentals
5. 3 Logging Approaches
6. Error Handling Patterns
7. Common Patterns & Use Cases
8. Troubleshooting Common Issues
9. Best Practices

### Part 2: Technical Reference (Deep Dive)
10. Important Interfaces & Classes Reference
11. Configuration Deep-Dive
12. Advanced Topics
13. Performance Tips

---

# PART 1: PRACTICAL GUIDE

---

## 1. Filters Overview

### What are Filters?

**Simple Definition:** Components that run code before or after specific stages in the request processing pipeline.

**Think of it like:** Quality control checkpoints in a factory where you can inspect, modify, or reject items at specific stages.

### Filter Pipeline Visual

```
HTTP Request
    ↓
[Authorization Filters] ───→ Check permissions FIRST
    ↓
[Resource Filters] ────────→ Before model binding
    ↓
Model Binding
    ↓
[Action Filters] ──────────→ Before/After action execution
    ↓
Action Method Executes
    ↓
[Result Filters] ──────────→ Before/After result execution
    ↓
[Exception Filters] ───────→ If ANY exception occurs
    ↓
HTTP Response
```

### Filters vs Middleware

| Feature | Middleware | Filters |
|---------|-----------|---------|
| **Scope** | App-wide | Controller/Action specific |
| **MVC Context** | ❌ No | ✅ Yes (ActionContext, ModelState) |
| **Order** | Sequential | 5 types with specific order |
| **Use Cases** | CORS, Auth, Static Files | Validation, Logging actions, Exception handling |
| **When to Use** | Request/Response processing | MVC-specific logic |

**Rule of Thumb:**
- Use **Middleware** for: Authentication, CORS, Request/Response modification
- Use **Filters** for: Action logging, validation, caching, exception handling

---

## 2. The 5 Filter Types

### ✨ .NET 9.0 Filter Execution Order

```
1. Authorization Filters
   ↓
2. Resource Filters (OnResourceExecuting)
   ↓
3. Model Binding
   ↓
4. Action Filters (OnActionExecuting)
   ↓
5. Action Method
   ↓
6. Action Filters (OnActionExecuted)
   ↓
7. Result Filters (OnResultExecuting)
   ↓
8. Result Execution
   ↓
9. Result Filters (OnResultExecuted)
   ↓
10. Resource Filters (OnResourceExecuted)

EXCEPTION at ANY stage:
   ↓
Exception Filters
```

---

### Type 1: Authorization Filters (IAuthorizationFilter)

**Purpose:** Run FIRST - determine if request is authorized

**When to use:**
- ✅ Custom authorization logic
- ✅ API key validation
- ✅ Custom security checks
- ❌ Authentication (use middleware instead)

**Complete Example:**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;

public class ApiKeyAuthorizationFilter : IAuthorizationFilter
{
    private readonly IConfiguration _configuration;

    public ApiKeyAuthorizationFilter(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public void OnAuthorization(AuthorizationFilterContext context)
    {
        if (!context.HttpContext.Request.Headers.TryGetValue("X-API-Key", out var apiKey))
        {
            context.Result = new UnauthorizedObjectResult(new
            {
                error = "API Key is required",
                header = "X-API-Key"
            });
            return;
        }

        var validApiKey = _configuration["ApiKey"];
        if (apiKey != validApiKey)
        {
            context.Result = new UnauthorizedObjectResult(new
            {
                error = "Invalid API Key"
            });
        }
    }
}

// Usage
[ServiceFilter(typeof(ApiKeyAuthorizationFilter))]
public class SecureController : ControllerBase
{
    [HttpGet("data")]
    public IActionResult GetData() => Ok("Secure data");
}

// Register in Program.cs
builder.Services.AddScoped<ApiKeyAuthorizationFilter>();
```

---

### Type 2: Resource Filters (IResourceFilter)

**Purpose:** Run before/after model binding - good for caching

**When to use:**
- ✅ Caching entire action results
- ✅ Short-circuit expensive operations
- ✅ Logging before model binding
- ❌ Accessing model data (not bound yet)

**Complete Example:**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;
using Microsoft.Extensions.Caching.Memory;

public class CacheResourceFilter : IResourceFilter
{
    private readonly IMemoryCache _cache;
    private readonly int _cacheDurationSeconds;

    public CacheResourceFilter(IMemoryCache cache, int cacheDurationSeconds = 60)
    {
        _cache = cache;
        _cacheDurationSeconds = cacheDurationSeconds;
    }

    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        var cacheKey = context.HttpContext.Request.Path.ToString();
        
        if (_cache.TryGetValue(cacheKey, out ObjectResult cachedResult))
        {
            context.Result = cachedResult; // Short-circuit!
            return;
        }
        
        // Store the cache key in HttpContext for use in OnResourceExecuted
        context.HttpContext.Items["CacheKey"] = cacheKey;
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        if (context.Result is ObjectResult result && context.HttpContext.Items["CacheKey"] is string cacheKey)
        {
            _cache.Set(cacheKey, result, TimeSpan.FromSeconds(_cacheDurationSeconds));
        }
    }
}

// Usage with attribute
public class CacheAttribute : TypeFilterAttribute
{
    public CacheAttribute(int durationSeconds = 60) 
        : base(typeof(CacheResourceFilter))
    {
        Arguments = new object[] { durationSeconds };
    }
}

[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [Cache(120)] // Cache for 2 minutes
    public IActionResult GetAll()
    {
        // Expensive operation
        return Ok(GetProductsFromDatabase());
    }
}

// Register in Program.cs
builder.Services.AddMemoryCache();
```

---

### Type 3: Action Filters (IActionFilter)

**Purpose:** Run before/after action method execution

**When to use:**
- ✅ Logging action execution
- ✅ Modifying action arguments
- ✅ Action-specific validation
- ✅ Performance measurement

**Complete Example:**

```csharp
using Microsoft.AspNetCore.Mvc.Filters;
using System.Diagnostics;

public class LogActionFilter : IActionFilter
{
    private readonly ILogger<LogActionFilter> _logger;

    public LogActionFilter(ILogger<LogActionFilter> logger)
    {
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        var actionName = context.ActionDescriptor.DisplayName;
        var arguments = string.Join(", ", context.ActionArguments.Select(x => $"{x.Key}={x.Value}"));
        
        _logger.LogInformation("Executing: {ActionName} with arguments: {Arguments}", 
            actionName, arguments);
        
        // Store start time for performance measurement
        context.HttpContext.Items["ActionStartTime"] = Stopwatch.GetTimestamp();
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        var actionName = context.ActionDescriptor.DisplayName;
        var startTime = (long)context.HttpContext.Items["ActionStartTime"]!;
        var elapsedMs = (Stopwatch.GetTimestamp() - startTime) * 1000.0 / Stopwatch.Frequency;
        
        if (context.Exception != null)
        {
            _logger.LogError(context.Exception, 
                "Action {ActionName} failed after {ElapsedMs}ms", 
                actionName, elapsedMs);
        }
        else
        {
            _logger.LogInformation("Action {ActionName} completed in {ElapsedMs}ms", 
                actionName, elapsedMs);
        }
    }
}

// Usage
[ApiController]
[Route("api/[controller]")]
[ServiceFilter(typeof(LogActionFilter))] // Apply to all actions
public class OrdersController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult Get(int id)
    {
        return Ok(new { id, name = "Order #" + id });
    }
    
    [HttpPost]
    public IActionResult Create([FromBody] Order order)
    {
        return CreatedAtAction(nameof(Get), new { id = order.Id }, order);
    }
}

// Register in Program.cs
builder.Services.AddScoped<LogActionFilter>();
```

---

### Type 4: Exception Filters (IExceptionFilter)

**Purpose:** Handle exceptions that occur during action execution

**When to use:**
- ✅ Global exception handling
- ✅ Custom error responses
- ✅ Exception logging
- ❌ Exceptions in middleware (use middleware instead)

**Complete Example:**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;

public class GlobalExceptionFilter : IExceptionFilter
{
    private readonly ILogger<GlobalExceptionFilter> _logger;
    private readonly IHostEnvironment _env;

    public GlobalExceptionFilter(
        ILogger<GlobalExceptionFilter> logger,
        IHostEnvironment env)
    {
        _logger = logger;
        _env = env;
    }

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, 
            "Unhandled exception in {ActionName}", 
            context.ActionDescriptor.DisplayName);

        var problemDetails = new ProblemDetails
        {
            Type = "https://tools.ietf.org/html/rfc7231#section-6.6.1",
            Title = "An error occurred",
            Status = StatusCodes.Status500InternalServerError,
            Instance = context.HttpContext.Request.Path
        };

        // Different responses for different exception types
        switch (context.Exception)
        {
            case ArgumentException argEx:
                problemDetails.Status = StatusCodes.Status400BadRequest;
                problemDetails.Title = "Invalid argument";
                problemDetails.Detail = argEx.Message;
                break;
                
            case KeyNotFoundException:
                problemDetails.Status = StatusCodes.Status404NotFound;
                problemDetails.Title = "Resource not found";
                break;
                
            case UnauthorizedAccessException:
                problemDetails.Status = StatusCodes.Status403Forbidden;
                problemDetails.Title = "Access denied";
                break;
                
            default:
                problemDetails.Detail = _env.IsDevelopment() 
                    ? context.Exception.Message 
                    : "An unexpected error occurred";
                break;
        }

        // Include stack trace in development
        if (_env.IsDevelopment())
        {
            problemDetails.Extensions["stackTrace"] = context.Exception.StackTrace;
        }

        context.Result = new ObjectResult(problemDetails)
        {
            StatusCode = problemDetails.Status
        };

        context.ExceptionHandled = true; // Mark as handled
    }
}

// Register globally in Program.cs
builder.Services.AddControllers(options =>
{
    options.Filters.Add<GlobalExceptionFilter>();
});
```

---

### Type 5: Result Filters (IResultFilter)

**Purpose:** Run before/after result execution (e.g., JSON serialization)

**When to use:**
- ✅ Modifying response before serialization
- ✅ Adding response headers
- ✅ Response formatting
- ❌ Exception handling (use Exception Filters)

**Complete Example:**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;

public class AddResponseHeaderFilter : IResultFilter
{
    private readonly string _headerName;
    private readonly string _headerValue;

    public AddResponseHeaderFilter(string headerName, string headerValue)
    {
        _headerName = headerName;
        _headerValue = headerValue;
    }

    public void OnResultExecuting(ResultExecutingContext context)
    {
        // Modify result before execution
        if (context.Result is ObjectResult objectResult)
        {
            // Wrap response
            objectResult.Value = new
            {
                data = objectResult.Value,
                timestamp = DateTime.UtcNow,
                requestId = context.HttpContext.TraceIdentifier
            };
        }
        
        context.HttpContext.Response.Headers.Append(_headerName, _headerValue);
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        // Log after result execution
        // Response is already written to client
    }
}

// Usage with attribute
public class AddHeaderAttribute : TypeFilterAttribute
{
    public AddHeaderAttribute(string headerName, string headerValue) 
        : base(typeof(AddResponseHeaderFilter))
    {
        Arguments = new object[] { headerName, headerValue };
    }
}

[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet]
    [AddHeader("X-Custom-Header", "CustomValue")]
    public IActionResult GetAll()
    {
        return Ok(new[] { "User1", "User2" });
    }
}
```

---

## 3. Creating Filters (3 Methods)

### Method 1: Attribute Filters (Simple)

**When to use:**
- ✅ Simple logic
- ✅ No dependency injection needed
- ✅ Apply to specific actions/controllers
- ❌ Need services (use TypeFilter instead)

**Step 1: Create Filter Attribute**

```csharp
using Microsoft.AspNetCore.Mvc.Filters;

public class ValidateModelAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
        {
            context.Result = new BadRequestObjectResult(new
            {
                errors = context.ModelState
                    .Where(x => x.Value?.Errors.Count > 0)
                    .ToDictionary(
                        x => x.Key,
                        x => x.Value?.Errors.Select(e => e.ErrorMessage).ToArray()
                    )
            });
        }
    }
}
```

**Step 2: Apply to Actions/Controllers**

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    [ValidateModel] // Apply filter
    public IActionResult Create([FromBody] Product product)
    {
        // Model is already validated by filter
        return Ok(product);
    }
}
```

---

### Method 2: TypeFilter/ServiceFilter (With Dependency Injection) 🎯 RECOMMENDED

**When to use:**
- ✅ Need dependency injection
- ✅ Production code
- ✅ Inject ILogger, DbContext, etc.

**Difference:**
- **ServiceFilter** - Filter must be registered in DI
- **TypeFilter** - ASP.NET Core creates instance (doesn't need registration)

**Step 1: Create Filter with Dependencies**

```csharp
public class AuditActionFilter : IActionFilter
{
    private readonly ILogger<AuditActionFilter> _logger;
    private readonly ApplicationDbContext _context;

    // Constructor injection works!
    public AuditActionFilter(
        ILogger<AuditActionFilter> logger,
        ApplicationDbContext context)
    {
        _logger = logger;
        _context = context;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        // Can use injected services
        _logger.LogInformation("Action executing");
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        // Log to database
        _context.AuditLogs.Add(new AuditLog
        {
            Action = context.ActionDescriptor.DisplayName,
            Timestamp = DateTime.UtcNow,
            UserId = context.HttpContext.User.Identity?.Name
        });
        _context.SaveChanges();
    }
}
```

**Step 2A: Use with ServiceFilter (Must Register)**

```csharp
// Register in Program.cs
builder.Services.AddScoped<AuditActionFilter>();

// Use in controller
[ServiceFilter(typeof(AuditActionFilter))]
public class OrdersController : ControllerBase { }
```

**Step 2B: Use with TypeFilter (No Registration) ⭐ EASIER**

```csharp
// No registration needed!

// Use directly in controller
[TypeFilter(typeof(AuditActionFilter))]
public class OrdersController : ControllerBase { }
```

**Step 3: Create Custom Attribute (Best Practice)**

```csharp
public class AuditAttribute : TypeFilterAttribute
{
    public AuditAttribute() : base(typeof(AuditActionFilter)) { }
}

// Clean usage!
[Audit]
public class OrdersController : ControllerBase { }
```

---

### Method 3: Global Filters (Application-Wide) 🎯 RECOMMENDED for Cross-Cutting

**When to use:**
- ✅ Apply to ALL controllers/actions
- ✅ Exception handling
- ✅ Logging
- ✅ Model validation

**Step 1: Register in Program.cs**

```csharp
builder.Services.AddControllers(options =>
{
    // Add global filters here
    options.Filters.Add<GlobalExceptionFilter>();
    options.Filters.Add<LogActionFilter>();
    options.Filters.Add(new ValidateModelAttribute());
});
```

**Complete Example: Global Exception Handler**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers(options =>
{
    // Global exception filter
    options.Filters.Add<GlobalExceptionFilter>();
    
    // Global model validation
    options.Filters.Add(new ValidateModelAttribute());
    
    // Global action logging
    options.Filters.Add<LogActionFilter>();
});

builder.Services.AddScoped<GlobalExceptionFilter>();
builder.Services.AddScoped<LogActionFilter>();

var app = builder.Build();
app.MapControllers();
app.Run();
```

---

### Filter Scope & Order

**Scope Priority (Most specific wins):**
```
1. Action-level filters      [ValidateModel] on method
2. Controller-level filters  [ValidateModel] on class
3. Global filters            options.Filters.Add<>()
```

**Execution Order:**
```
Global (OnExecuting)
  ↓
Controller (OnExecuting)
  ↓
Action (OnExecuting)
  ↓
** Action Method **
  ↓
Action (OnExecuted)
  ↓
Controller (OnExecuted)
  ↓
Global (OnExecuted)
```

**Custom Order (IOrderedFilter):**

```csharp
public class OrderedFilter : IActionFilter, IOrderedFilter
{
    public int Order { get; set; } = 0; // Lower runs first
    
    public void OnActionExecuting(ActionExecutingContext context) { }
    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage
[TypeFilter(typeof(OrderedFilter), Order = 1)]
public class MyController : ControllerBase { }
```

---

## 4. Logging Fundamentals

### What is Logging?

**Simple Definition:** Recording application events for monitoring, debugging, and auditing.

### Built-in Logging in ASP.NET Core

ASP.NET Core includes a powerful logging framework out of the box!

**6 Log Levels:**

| Level | Value | When to Use | Example |
|-------|-------|-------------|---------|
| **Trace** | 0 | Extremely detailed debugging | "Variable x = 5" |
| **Debug** | 1 | Development debugging | "User query executed" |
| **Information** | 2 | General app flow | "Request started" |
| **Warning** | 3 | Unexpected but not errors | "Slow query (2s)" |
| **Error** | 4 | Errors that can be recovered | "Failed to save, retrying" |
| **Critical** | 5 | App-breaking errors | "Database unavailable" |

### ILogger<T> Injection

**Step 1: Inject ILogger in Any Class**

```csharp
public class ProductsController : ControllerBase
{
    private readonly ILogger<ProductsController> _logger;
    
    public ProductsController(ILogger<ProductsController> logger)
    {
        _logger = logger;
    }
    
    [HttpGet("{id}")]
    public IActionResult Get(int id)
    {
        _logger.LogInformation("Getting product {ProductId}", id);
        
        try
        {
            var product = GetProduct(id);
            return Ok(product);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to get product {ProductId}", id);
            return StatusCode(500);
        }
    }
}
```

**Best Practices:**
- ✅ Use structured logging: `_logger.LogInfo("User {UserId} logged in", userId)`
- ❌ Avoid: `_logger.LogInfo($"User {userId} logged in")` (string interpolation)
- ✅ Include context: `_logger.LogError(ex, "Error message", contextData)`

---

### Default Logging Configuration

**appsettings.json:**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Warning"
    }
  }
}
```

**appsettings.Development.json:**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information"
    }
  }
}
```

---

## 5. 3 Logging Approaches

### Method 1: Built-in Console Logging (Quick Start)

**When to use:**
- ✅ Development/Learning
- ✅ Quick debugging
- ❌ Production (logs disappear on restart)

**Already Configured by Default!**

```csharp
// Program.cs - Default template includes this
var builder = WebApplication.CreateBuilder(args);

// Console logging is already configured!
// Logs to console automatically

builder.Services.AddControllers();

var app = builder.Build();
app.MapControllers();
app.Run();
```

**Usage Example:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly ILogger<UsersController> _logger;

    public UsersController(ILogger<UsersController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IActionResult GetAll()
    {
        _logger.LogInformation("Getting all users");
        _logger.LogDebug("This will show in Development only");
        
        return Ok(new[] { "User1", "User2" });
    }
}
```

**Output:**
```
info: MyApp.Controllers.UsersController[0]
      Getting all users
debug: MyApp.Controllers.UsersController[0]
      This will show in Development only
```

---

### Method 2: File Logging with Serilog 🎯 RECOMMENDED for Production

**When to use:**
- ✅ Production applications
- ✅ Structured logging
- ✅ Persistent logs
- ✅ Log rotation

**Step 1: Install Serilog**

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Sinks.Console
```

**Step 2: Configure in Program.cs**

```csharp
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft.AspNetCore", Serilog.Events.LogEventLevel.Warning)
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File(
        path: "logs/app-.log",
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 30,
        outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss} [{Level:u3}] {Message:lj}{NewLine}{Exception}"
    )
    .CreateLogger();

builder.Host.UseSerilog(); // Replace built-in logging with Serilog

builder.Services.AddControllers();

var app = builder.Build();

try
{
    Log.Information("Starting web application");
    app.MapControllers();
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

**Step 3: Configure from appsettings.json**

```json
{
  "Serilog": {
    "Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.File" ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.AspNetCore": "Warning"
      }
    },
    "WriteTo": [
      { "Name": "Console" },
      {
        "Name": "File",
        "Args": {
          "path": "logs/app-.log",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 30,
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss} [{Level:u3}] {Message:lj}{NewLine}{Exception}"
        }
      }
    ],
    "Enrich": [ "FromLogContext" ]
  }
}
```

**Step 4: Use Configuration in Program.cs**

```csharp
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Read Serilog config from appsettings.json
builder.Host.UseSerilog((context, configuration) =>
{
    configuration.ReadFrom.Configuration(context.Configuration);
});

var app = builder.Build();
app.MapControllers();
app.Run();
```

**Complete Example with Structured Logging:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly ILogger<OrdersController> _logger;

    public OrdersController(ILogger<OrdersController> logger)
    {
        _logger = logger;
    }

    [HttpPost]
    public IActionResult CreateOrder([FromBody] Order order)
    {
        // Structured logging - properties can be queried!
        _logger.LogInformation(
            "Creating order for customer {CustomerId} with total {Total:C}",
            order.CustomerId,
            order.Total);

        try
        {
            // Save order
            _logger.LogInformation(
                "Order {OrderId} created successfully",
                order.Id);
            
            return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex,
                "Failed to create order for customer {CustomerId}",
                order.CustomerId);
            
            return StatusCode(500);
        }
    }
}
```

---

### Method 3: Centralized Logging (Production at Scale)

**When to use:**
- ✅ Production environment
- ✅ Multiple servers/containers
- ✅ Log analysis and searching
- ✅ Alerting and monitoring

**Option 3A: Seq (Free for local development)**

**Step 1: Install Seq Sink**

```bash
dotnet add package Serilog.Sinks.Seq
```

**Step 2: Configure Seq**

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Seq("http://localhost:5341") // Seq server URL
    .CreateLogger();
```

**appsettings.json:**

```json
{
  "Serilog": {
    "WriteTo": [
      {
        "Name": "Seq",
        "Args": {
          "serverUrl": "http://localhost:5341",
          "apiKey": "your-api-key" // Optional
        }
      }
    ]
  }
}
```

**Option 3B: Application Insights (Azure)**

**Step 1: Install Package**

```bash
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

**Step 2: Configure**

```csharp
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
});

builder.Host.UseSerilog((context, configuration) =>
{
    configuration
        .ReadFrom.Configuration(context.Configuration)
        .WriteTo.ApplicationInsights(
            telemetryConfiguration: context.Configuration["ApplicationInsights:ConnectionString"],
            telemetryConverter: TelemetryConverter.Traces);
});
```

**appsettings.json:**

```json
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey=your-key-here"
  }
}
```

**Option 3C: ELK Stack (Elasticsearch, Logstash, Kibana)**

```bash
dotnet add package Serilog.Sinks.Elasticsearch
```

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(new Uri("http://localhost:9200"))
    {
        AutoRegisterTemplate = true,
        IndexFormat = "myapp-logs-{0:yyyy.MM.dd}"
    })
    .CreateLogger();
```

---

### Logging Comparison Table

| Approach | Setup | Cost | Search | Alerts | Production |
|----------|-------|------|--------|--------|-----------|
| **Console** | None | Free | ❌ No | ❌ No | ❌ No |
| **File (Serilog)** | Easy | Free | ⚠️ Manual | ❌ No | ⚠️ Single server |
| **Seq** | Easy | Free/Paid | ✅ Yes | ✅ Yes | ✅ Yes |
| **App Insights** | Medium | Azure cost | ✅ Yes | ✅ Yes | ✅ Yes |
| **ELK Stack** | Complex | Free/Self-host | ✅ Yes | ✅ Yes | ✅ Yes |

---

## 6. Error Handling Patterns

### Method 1: Exception Filter (Global) 🎯 RECOMMENDED

**When to use:**
- ✅ API applications
- ✅ Consistent error responses
- ✅ MVC context available

**Complete Implementation:**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;

public class GlobalExceptionFilter : IExceptionFilter
{
    private readonly ILogger<GlobalExceptionFilter> _logger;
    private readonly IHostEnvironment _env;

    public GlobalExceptionFilter(
        ILogger<GlobalExceptionFilter> logger,
        IHostEnvironment env)
    {
        _logger = logger;
        _env = env;
    }

    public void OnException(ExceptionContext context)
    {
        // Log the exception
        _logger.LogError(context.Exception,
            "Unhandled exception in {ControllerName}.{ActionName}",
            context.RouteData.Values["controller"],
            context.RouteData.Values["action"]);

        // Create ProblemDetails response
        var problemDetails = CreateProblemDetails(context);

        // Set the result
        context.Result = new ObjectResult(problemDetails)
        {
            StatusCode = problemDetails.Status
        };

        // Mark exception as handled
        context.ExceptionHandled = true;
    }

    private ProblemDetails CreateProblemDetails(ExceptionContext context)
    {
        var (statusCode, title) = context.Exception switch
        {
            ArgumentException => (StatusCodes.Status400BadRequest, "Bad Request"),
            ArgumentNullException => (StatusCodes.Status400BadRequest, "Bad Request"),
            KeyNotFoundException => (StatusCodes.Status404NotFound, "Not Found"),
            UnauthorizedAccessException => (StatusCodes.Status403Forbidden, "Forbidden"),
            InvalidOperationException => (StatusCodes.Status409Conflict, "Conflict"),
            _ => (StatusCodes.Status500InternalServerError, "Internal Server Error")
        };

        var problemDetails = new ProblemDetails
        {
            Type = $"https://httpstatuses.com/{statusCode}",
            Title = title,
            Status = statusCode,
            Detail = _env.IsDevelopment() 
                ? context.Exception.Message 
                : "An error occurred processing your request",
            Instance = context.HttpContext.Request.Path
        };

        // Add additional info in development
        if (_env.IsDevelopment())
        {
            problemDetails.Extensions["exception"] = context.Exception.GetType().Name;
            problemDetails.Extensions["stackTrace"] = context.Exception.StackTrace;
        }

        // Add trace ID for correlation
        problemDetails.Extensions["traceId"] = context.HttpContext.TraceIdentifier;

        return problemDetails;
    }
}

// Register in Program.cs
builder.Services.AddControllers(options =>
{
    options.Filters.Add<GlobalExceptionFilter>();
});
```

**Custom Exceptions:**

```csharp
// Define domain-specific exceptions
public class ResourceNotFoundException : Exception
{
    public string ResourceName { get; }
    public object ResourceId { get; }

    public ResourceNotFoundException(string resourceName, object resourceId)
        : base($"{resourceName} with ID {resourceId} was not found")
    {
        ResourceName = resourceName;
        ResourceId = resourceId;
    }
}

public class ValidationException : Exception
{
    public IDictionary<string, string[]> Errors { get; }

    public ValidationException(IDictionary<string, string[]> errors)
        : base("One or more validation errors occurred")
    {
        Errors = errors;
    }
}

// Enhanced filter
public void OnException(ExceptionContext context)
{
    var problemDetails = context.Exception switch
    {
        ResourceNotFoundException ex => new ProblemDetails
        {
            Status = StatusCodes.Status404NotFound,
            Title = "Resource Not Found",
            Detail = ex.Message,
            Extensions =
            {
                ["resourceName"] = ex.ResourceName,
                ["resourceId"] = ex.ResourceId
            }
        },
        ValidationException ex => new ValidationProblemDetails(ex.Errors)
        {
            Status = StatusCodes.Status400BadRequest,
            Title = "Validation Failed"
        },
        _ => CreateProblemDetails(context)
    };

    // ... rest of implementation
}
```

---

### Method 2: Exception Middleware

**When to use:**
- ✅ Catch ALL exceptions (including middleware)
- ✅ Non-MVC applications
- ✅ More control over response

**Complete Implementation:**

```csharp
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;
    private readonly IHostEnvironment _env;

    public GlobalExceptionMiddleware(
        RequestDelegate next,
        ILogger<GlobalExceptionMiddleware> logger,
        IHostEnvironment env)
    {
        _next = next;
        _logger = logger;
        _env = env;
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
            await HandleExceptionAsync(context, ex);
        }
    }

    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";

        var (statusCode, message) = exception switch
        {
            ArgumentException => (StatusCodes.Status400BadRequest, exception.Message),
            KeyNotFoundException => (StatusCodes.Status404NotFound, "Resource not found"),
            UnauthorizedAccessException => (StatusCodes.Status403Forbidden, "Access denied"),
            _ => (StatusCodes.Status500InternalServerError, 
                _env.IsDevelopment() ? exception.Message : "An error occurred")
        };

        context.Response.StatusCode = statusCode;

        var problemDetails = new ProblemDetails
        {
            Type = $"https://httpstatuses.com/{statusCode}",
            Title = GetTitle(statusCode),
            Status = statusCode,
            Detail = message,
            Instance = context.Request.Path
        };

        if (_env.IsDevelopment())
        {
            problemDetails.Extensions["exception"] = exception.GetType().Name;
            problemDetails.Extensions["stackTrace"] = exception.StackTrace;
        }

        await context.Response.WriteAsJsonAsync(problemDetails);
    }

    private static string GetTitle(int statusCode) => statusCode switch
    {
        StatusCodes.Status400BadRequest => "Bad Request",
        StatusCodes.Status404NotFound => "Not Found",
        StatusCodes.Status403Forbidden => "Forbidden",
        StatusCodes.Status500InternalServerError => "Internal Server Error",
        _ => "Error"
    };
}

// Extension method
public static class GlobalExceptionMiddlewareExtensions
{
    public static IApplicationBuilder UseGlobalExceptionHandler(
        this IApplicationBuilder app)
    {
        return app.UseMiddleware<GlobalExceptionMiddleware>();
    }
}

// Register in Program.cs
var app = builder.Build();

app.UseGlobalExceptionHandler(); // First middleware!

app.UseRouting();
app.MapControllers();
app.Run();
```

---

### Method 3: Built-in Exception Handler Middleware

**When to use:**
- ✅ Simple setup
- ✅ Standard error page
- ✅ Works with Razor Pages/MVC views

**Step 1: Create Error Controller**

```csharp
[ApiController]
public class ErrorController : ControllerBase
{
    private readonly ILogger<ErrorController> _logger;

    public ErrorController(ILogger<ErrorController> logger)
    {
        _logger = logger;
    }

    [Route("/error")]
    [ApiExplorerSettings(IgnoreApi = true)]
    public IActionResult HandleError()
    {
        var exceptionFeature = HttpContext.Features.Get<IExceptionHandlerFeature>();
        var exception = exceptionFeature?.Error;

        _logger.LogError(exception, "Unhandled exception");

        var (statusCode, message) = exception switch
        {
            ArgumentException => (StatusCodes.Status400BadRequest, exception.Message),
            KeyNotFoundException => (StatusCodes.Status404NotFound, "Resource not found"),
            _ => (StatusCodes.Status500InternalServerError, "An error occurred")
        };

        return Problem(
            title: "An error occurred",
            statusCode: statusCode,
            detail: message);
    }

    [Route("/error-development")]
    [ApiExplorerSettings(IgnoreApi = true)]
    public IActionResult HandleErrorDevelopment()
    {
        var exceptionFeature = HttpContext.Features.Get<IExceptionHandlerFeature>();
        var exception = exceptionFeature?.Error;

        return Problem(
            title: exception?.GetType().Name,
            detail: exception?.StackTrace);
    }
}
```

**Step 2: Configure in Program.cs**

```csharp
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error-development");
}
else
{
    app.UseExceptionHandler("/error");
}

app.MapControllers();
app.Run();
```

---

### Error Handling Comparison

| Approach | Scope | MVC Context | Middleware Errors | Best For |
|----------|-------|-------------|-------------------|----------|
| **Exception Filter** | Controllers only | ✅ Yes | ❌ No | APIs, MVC apps |
| **Exception Middleware** | Entire pipeline | ❌ No | ✅ Yes | Catch-all |
| **Built-in Handler** | Entire pipeline | ⚠️ Limited | ✅ Yes | Simple apps |

**Recommendation:** Use **Exception Filter** for APIs + **Exception Middleware** as fallback

```csharp
var app = builder.Build();

// Catch middleware exceptions
app.UseGlobalExceptionHandler();

// ... other middleware ...

// Exception filter catches controller exceptions
builder.Services.AddControllers(options =>
{
    options.Filters.Add<GlobalExceptionFilter>();
});
```

---

## 7. Common Patterns & Use Cases

### Pattern 1: Request/Response Logging

```csharp
public class RequestResponseLoggingFilter : IActionFilter
{
    private readonly ILogger<RequestResponseLoggingFilter> _logger;

    public RequestResponseLoggingFilter(ILogger<RequestResponseLoggingFilter> logger)
    {
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        _logger.LogInformation(
            "Request: {Method} {Path} | Controller: {Controller} | Action: {Action}",
            context.HttpContext.Request.Method,
            context.HttpContext.Request.Path,
            context.RouteData.Values["controller"],
            context.RouteData.Values["action"]);

        if (context.ActionArguments.Any())
        {
            _logger.LogDebug(
                "Arguments: {Arguments}",
                System.Text.Json.JsonSerializer.Serialize(context.ActionArguments));
        }
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        if (context.Result is ObjectResult objectResult)
        {
            _logger.LogInformation(
                "Response: Status {StatusCode}",
                objectResult.StatusCode ?? 200);

            _logger.LogDebug(
                "Response Data: {Data}",
                System.Text.Json.JsonSerializer.Serialize(objectResult.Value));
        }
    }
}
```

---

### Pattern 2: Performance Monitoring

```csharp
public class PerformanceMonitoringFilter : IActionFilter
{
    private readonly ILogger<PerformanceMonitoringFilter> _logger;
    private const string StopwatchKey = "StopwatchKey";

    public PerformanceMonitoringFilter(ILogger<PerformanceMonitoringFilter> logger)
    {
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        context.HttpContext.Items[StopwatchKey] = stopwatch;
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        if (context.HttpContext.Items[StopwatchKey] is Stopwatch stopwatch)
        {
            stopwatch.Stop();
            var elapsedMs = stopwatch.ElapsedMilliseconds;

            var actionName = $"{context.RouteData.Values["controller"]}.{context.RouteData.Values["action"]}";

            if (elapsedMs > 1000) // Slow request threshold
            {
                _logger.LogWarning(
                    "SLOW REQUEST: {Action} took {ElapsedMs}ms",
                    actionName,
                    elapsedMs);
            }
            else
            {
                _logger.LogInformation(
                    "{Action} completed in {ElapsedMs}ms",
                    actionName,
                    elapsedMs);
            }
        }
    }
}
```

---

### Pattern 3: API Versioning Filter

```csharp
public class ApiVersionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        context.HttpContext.Response.Headers.Append("X-API-Version", "1.0");
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}
```

---

### Pattern 4: Correlation ID

```csharp
public class CorrelationIdMiddleware
{
    private readonly RequestDelegate _next;
    private const string CorrelationIdHeader = "X-Correlation-ID";

    public CorrelationIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = context.Request.Headers[CorrelationIdHeader].FirstOrDefault()
            ?? Guid.NewGuid().ToString();

        context.Items["CorrelationId"] = correlationId;
        context.Response.Headers.Append(CorrelationIdHeader, correlationId);

        using (LogContext.PushProperty("CorrelationId", correlationId))
        {
            await _next(context);
        }
    }
}

// Usage in controllers
public class OrdersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create([FromBody] Order order)
    {
        var correlationId = HttpContext.Items["CorrelationId"];
        // All logs will include correlationId automatically with Serilog
        return Ok();
    }
}
```

---

### Pattern 5: Model Validation with Custom Messages

```csharp
public class ValidateModelFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
        {
            var errors = context.ModelState
                .Where(x => x.Value?.Errors.Count > 0)
                .ToDictionary(
                    kvp => ToCamelCase(kvp.Key),
                    kvp => kvp.Value!.Errors.Select(e => e.ErrorMessage).ToArray()
                );

            var validationProblem = new ValidationProblemDetails(errors)
            {
                Type = "https://tools.ietf.org/html/rfc7231#section-6.5.1",
                Title = "One or more validation errors occurred",
                Status = StatusCodes.Status400BadRequest,
                Instance = context.HttpContext.Request.Path
            };

            context.Result = new BadRequestObjectResult(validationProblem);
        }
    }

    public void OnActionExecuted(ActionExecutedContext context) { }

    private static string ToCamelCase(string str)
    {
        if (string.IsNullOrEmpty(str) || !char.IsUpper(str[0]))
            return str;

        return char.ToLowerInvariant(str[0]) + str[1..];
    }
}
```

---

## 8. Troubleshooting Common Issues

### Issue 1: Filter Not Executing

**Problem:** Filter registered but not running

**Solutions:**

```csharp
// ❌ WRONG - Filter not registered
[TypeFilter(typeof(MyFilter))] // TypeFilter doesn't need registration

// ✅ CORRECT
[ServiceFilter(typeof(MyFilter))] // Need to register
builder.Services.AddScoped<MyFilter>();

// OR
[TypeFilter(typeof(MyFilter))] // Works without registration
```

---

### Issue 2: Dependency Injection Fails in Filter

**Problem:** `NullReferenceException` when accessing injected service

**Solution:**

```csharp
// ❌ WRONG - Using Attribute directly
public class MyFilterAttribute : Attribute, IActionFilter
{
    private readonly IMyService _service; // This WON'T work!
    
    // Attributes can't use constructor injection
}

// ✅ CORRECT - Use TypeFilter or ServiceFilter
public class MyFilter : IActionFilter
{
    private readonly IMyService _service; // This WILL work!
    
    public MyFilter(IMyService service)
    {
        _service = service;
    }
}

// Usage
[TypeFilter(typeof(MyFilter))]
public IActionResult MyAction() { }
```

---

### Issue 3: Exception Filter Not Catching Exceptions

**Problem:** Exceptions not being caught by exception filter

**Cause:** Exception thrown in middleware (before MVC)

**Solution:**

```csharp
// Exception in middleware - NOT caught by exception filter
app.Use(async (context, next) =>
{
    throw new Exception("Not caught by filter!"); // ❌
    await next();
});

// Use exception middleware instead
app.UseGlobalExceptionHandler(); // ✅ Catches all
```

---

### Issue 4: Logs Not Appearing

**Problem:** Logs not showing in console/file

**Solutions:**

```csharp
// 1. Check log level in appsettings.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information", // Make sure not set to Warning or higher
      "Microsoft.AspNetCore": "Warning"
    }
  }
}

// 2. Make sure using correct ILogger<T>
public class MyController : ControllerBase
{
    private readonly ILogger<MyController> _logger; // ✅ CORRECT type
    
    // NOT ILogger - too generic
}

// 3. Check Serilog configuration
builder.Host.UseSerilog(); // ✅ Must call this!
```

---

### Issue 5: ProblemDetails Not Returned

**Problem:** Custom error format instead of ProblemDetails

**Solution:**

```csharp
// Make sure to set the result correctly
context.Result = new ObjectResult(problemDetails) // ✅
{
    StatusCode = problemDetails.Status
};

// NOT
context.Result = new JsonResult(problemDetails); // ❌ May not work
```

---

## 9. Best Practices

### ✅ DO's

1. **Use Global Exception Filter for APIs**
   ```csharp
   builder.Services.AddControllers(options =>
   {
       options.Filters.Add<GlobalExceptionFilter>();
   });
   ```

2. **Use Structured Logging**
   ```csharp
   // ✅ GOOD - Structured
   _logger.LogInformation("User {UserId} ordered {ProductId}", userId, productId);
   
   // ❌ BAD - String interpolation
   _logger.LogInformation($"User {userId} ordered {productId}");
   ```

3. **Create Custom Attributes for TypeFilters**
   ```csharp
   public class AuditAttribute : TypeFilterAttribute
   {
       public AuditAttribute() : base(typeof(AuditFilter)) { }
   }
   
   [Audit] // Clean!
   ```

4. **Use ILogger<T> Not ILogger**
   ```csharp
   // ✅ GOOD
   private readonly ILogger<MyController> _logger;
   
   // ❌ BAD
   private readonly ILogger _logger;
   ```

5. **Handle Exceptions at the Right Level**
   ```csharp
   // Middleware exceptions → Exception Middleware
   // Controller exceptions → Exception Filter
   ```

6. **Use ProblemDetails for Errors**
   ```csharp
   return Problem(
       title: "Resource not found",
       statusCode: 404,
       detail: "Product with ID 123 was not found");
   ```

---

### ❌ DON'Ts

1. **Don't Log Sensitive Data**
   ```csharp
   // ❌ BAD
   _logger.LogInformation("User password: {Password}", password);
   
   // ✅ GOOD
   _logger.LogInformation("User logged in successfully");
   ```

2. **Don't Use Filters for Cross-Cutting Concerns Across All Requests**
   ```csharp
   // ❌ Use middleware instead for:
   // - CORS
   // - Authentication
   // - Request logging (for ALL requests)
   ```

3. **Don't Swallow Exceptions**
   ```csharp
   // ❌ BAD
   catch (Exception ex)
   {
       // Silent failure
   }
   
   // ✅ GOOD
   catch (Exception ex)
   {
       _logger.LogError(ex, "Error occurred");
       throw; // or handle appropriately
   }
   ```

4. **Don't Modify Response After Calling next()**
   ```csharp
   // ❌ BAD
   await _next(context);
   context.Response.StatusCode = 200; // Too late!
   
   // ✅ GOOD
   context.Response.StatusCode = 200;
   await _next(context);
   ```

5. **Don't Use Console.WriteLine for Logging**
   ```csharp
   // ❌ BAD
   Console.WriteLine("User logged in");
   
   // ✅ GOOD
   _logger.LogInformation("User logged in");
   ```

---

### Performance Best Practices

1. **Order Filters by Cost**
   ```csharp
   // Cheap filters first
   [ValidateModel]        // Fast
   [Cache]                // Medium
   [DatabaseAudit]        // Expensive
   public IActionResult MyAction() { }
   ```

2. **Use Async Filters for I/O Operations**
   ```csharp
   public class AsyncFilter : IAsyncActionFilter
   {
       public async Task OnActionExecutionAsync(
           ActionExecutingContext context,
           ActionExecutionDelegate next)
       {
           await DoSomethingAsync(); // ✅ Async I/O
           await next();
       }
   }
   ```

3. **Avoid Expensive Operations in Filters**
   ```csharp
   // ❌ BAD - Database call in every request
   public void OnActionExecuting(ActionExecutingContext context)
   {
       var user = _context.Users.FirstOrDefault(); // Expensive!
   }
   
   // ✅ GOOD - Cache or use middleware
   ```

---

# PART 2: TECHNICAL REFERENCE

---

## 10. Important Interfaces & Classes Reference

### IActionFilter Interface

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Purpose:** Execute code before/after action method execution

**Declaration:**
```csharp
public interface IActionFilter : IFilterMetadata
{
    void OnActionExecuting(ActionExecutingContext context);
    void OnActionExecuted(ActionExecutedContext context);
}
```

**Members:**

| Member | Parameters | Return Type | Description |
|--------|-----------|-------------|-------------|
| `OnActionExecuting` | `ActionExecutingContext` | `void` | Called before action execution |
| `OnActionExecuted` | `ActionExecutedContext` | `void` | Called after action execution |

**Async Version:**
```csharp
public interface IAsyncActionFilter : IFilterMetadata
{
    Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next);
}
```

**Usage Example:**
```csharp
public class LogActionFilter : IActionFilter
{
    private readonly ILogger _logger;
    
    public LogActionFilter(ILogger<LogActionFilter> logger)
    {
        _logger = logger;
    }
    
    public void OnActionExecuting(ActionExecutingContext context)
    {
        _logger.LogInformation("Executing {Action}", 
            context.ActionDescriptor.DisplayName);
    }
    
    public void OnActionExecuted(ActionExecutedContext context)
    {
        _logger.LogInformation("Executed {Action}", 
            context.ActionDescriptor.DisplayName);
    }
}
```

---

### IExceptionFilter Interface

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Purpose:** Handle exceptions during action execution

**Declaration:**
```csharp
public interface IExceptionFilter : IFilterMetadata
{
    void OnException(ExceptionContext context);
}
```

**Members:**

| Member | Parameters | Return Type | Description |
|--------|-----------|-------------|-------------|
| `OnException` | `ExceptionContext` | `void` | Called when exception occurs |

**ExceptionContext Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Exception` | `Exception` | The exception that occurred |
| `ExceptionHandled` | `bool` | Set to true to mark as handled |
| `Result` | `IActionResult` | Set the response result |
| `ActionDescriptor` | `ActionDescriptor` | Action metadata |
| `HttpContext` | `HttpContext` | HTTP context |

**Async Version:**
```csharp
public interface IAsyncExceptionFilter : IFilterMetadata
{
    Task OnExceptionAsync(ExceptionContext context);
}
```

---

### IAuthorizationFilter Interface

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Purpose:** Perform authorization checks

**Declaration:**
```csharp
public interface IAuthorizationFilter : IFilterMetadata
{
    void OnAuthorization(AuthorizationFilterContext context);
}
```

**AuthorizationFilterContext Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Result` | `IActionResult` | Set to short-circuit pipeline |
| `HttpContext` | `HttpContext` | HTTP context |
| `Filters` | `IList<IFilterMetadata>` | All filters |
| `ActionDescriptor` | `ActionDescriptor` | Action metadata |

---

### IResourceFilter Interface

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Purpose:** Execute before/after model binding

**Declaration:**
```csharp
public interface IResourceFilter : IFilterMetadata
{
    void OnResourceExecuting(ResourceExecutingContext context);
    void OnResourceExecuted(ResourceExecutedContext context);
}
```

**Async Version:**
```csharp
public interface IAsyncResourceFilter : IFilterMetadata
{
    Task OnResourceExecutionAsync(
        ResourceExecutingContext context,
        ResourceExecutionDelegate next);
}
```

---

### IResultFilter Interface

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Purpose:** Execute before/after result execution

**Declaration:**
```csharp
public interface IResultFilter : IFilterMetadata
{
    void OnResultExecuting(ResultExecutingContext context);
    void OnResultExecuted(ResultExecutedContext context);
}
```

---

### ILogger<T> Interface

**Namespace:** `Microsoft.Extensions.Logging`

**Purpose:** Logging abstraction

**Declaration:**
```csharp
public interface ILogger<out TCategoryName> : ILogger
{
}

public interface ILogger
{
    void Log<TState>(
        LogLevel logLevel,
        EventId eventId,
        TState state,
        Exception? exception,
        Func<TState, Exception?, string> formatter);
    
    bool IsEnabled(LogLevel logLevel);
    
    IDisposable BeginScope<TState>(TState state);
}
```

**Extension Methods:**

| Method | Description | Example |
|--------|-------------|---------|
| `LogTrace(message, args)` | Trace level logging | `_logger.LogTrace("Details: {Details}", details)` |
| `LogDebug(message, args)` | Debug level logging | `_logger.LogDebug("Debug info")` |
| `LogInformation(message, args)` | Info level logging | `_logger.LogInformation("User {Id}", id)` |
| `LogWarning(message, args)` | Warning level logging | `_logger.LogWarning("Slow query")` |
| `LogError(exception, message, args)` | Error level logging | `_logger.LogError(ex, "Failed")` |
| `LogCritical(exception, message, args)` | Critical level logging | `_logger.LogCritical(ex, "DB down")` |

---

### ActionExecutingContext Class

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Key Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `ActionArguments` | `IDictionary<string, object>` | Action method arguments |
| `ActionDescriptor` | `ActionDescriptor` | Action metadata |
| `Controller` | `object` | Controller instance |
| `HttpContext` | `HttpContext` | HTTP context |
| `ModelState` | `ModelStateDictionary` | Model validation state |
| `Result` | `IActionResult` | Set to short-circuit |
| `RouteData` | `RouteData` | Route values |

---

### ActionExecutedContext Class

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Key Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Result` | `IActionResult` | Action result |
| `Canceled` | `bool` | If action was canceled |
| `Exception` | `Exception` | Exception if occurred |
| `ExceptionHandled` | `bool` | If exception was handled |
| `ExceptionDispatchInfo` | `ExceptionDispatchInfo` | Exception details |

---

### FilterDescriptor Class

**Namespace:** `Microsoft.AspNetCore.Mvc.Filters`

**Key Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Filter` | `IFilterMetadata` | The filter instance |
| `Scope` | `FilterScope` | Filter scope (Global, Controller, Action) |
| `Order` | `int` | Execution order |

---

### LogLevel Enum

**Namespace:** `Microsoft.Extensions.Logging`

**Values:**

| Value | Int | Description | When to Use |
|-------|-----|-------------|-------------|
| `Trace` | 0 | Very detailed logs | Variable values, detailed flow |
| `Debug` | 1 | Debug information | Development debugging |
| `Information` | 2 | General flow | Application flow, state changes |
| `Warning` | 3 | Abnormal events | Deprecated API usage, slow queries |
| `Error` | 4 | Error events | Handled exceptions, failures |
| `Critical` | 5 | Critical failures | Unhandled exceptions, app crashes |
| `None` | 6 | Disable logging | N/A |

---

## 11. Configuration Deep-Dive

### Pattern 1: Inline Configuration (Hardcoded)

**When to use:** Quick testing, simple scenarios

```csharp
public class SimpleFilter : IActionFilter
{
    private const int MaxRetries = 3; // Hardcoded
    
    public void OnActionExecuting(ActionExecutingContext context)
    {
        // Use hardcoded value
    }
    
    public void OnActionExecuted(ActionExecutedContext context) { }
}
```

**Pros:**
- ✅ Simple
- ✅ No setup needed

**Cons:**
- ❌ Can't change without recompiling
- ❌ Not suitable for production

---

### Pattern 2: Constructor Parameters (Code-based)

**When to use:** Need flexibility, configured in code

```csharp
public class ConfigurableFilter : IActionFilter
{
    private readonly int _maxRetries;
    private readonly bool _enabled;
    
    public ConfigurableFilter(int maxRetries, bool enabled)
    {
        _maxRetries = maxRetries;
        _enabled = enabled;
    }
    
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!_enabled) return;
        
        // Use _maxRetries
    }
    
    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Custom attribute
public class ConfigurableAttribute : TypeFilterAttribute
{
    public ConfigurableAttribute(int maxRetries = 3, bool enabled = true)
        : base(typeof(ConfigurableFilter))
    {
        Arguments = new object[] { maxRetries, enabled };
    }
}

// Usage
[Configurable(maxRetries: 5, enabled: true)]
public IActionResult MyAction() { }
```

---

### Pattern 3: IOptions Pattern (appsettings.json) 🎯 RECOMMENDED

**When to use:** Production applications, configuration files

**Step 1: Create Options Class**

```csharp
public class FilterOptions
{
    public const string SectionName = "FilterSettings";
    
    public int MaxRetries { get; set; } = 3;
    public bool Enabled { get; set; } = true;
    public TimeSpan Timeout { get; set; } = TimeSpan.FromSeconds(30);
    public string[] ExcludedPaths { get; set; } = Array.Empty<string>();
}
```

**Step 2: Configure in appsettings.json**

```json
{
  "FilterSettings": {
    "MaxRetries": 5,
    "Enabled": true,
    "Timeout": "00:01:00",
    "ExcludedPaths": ["/health", "/metrics"]
  }
}
```

**Step 3: Register Options**

```csharp
builder.Services.Configure<FilterOptions>(
    builder.Configuration.GetSection(FilterOptions.SectionName));
```

**Step 4: Inject in Filter**

```csharp
public class ConfiguredFilter : IActionFilter
{
    private readonly FilterOptions _options;
    private readonly ILogger _logger;
    
    public ConfiguredFilter(
        IOptions<FilterOptions> options,
        ILogger<ConfiguredFilter> logger)
    {
        _options = options.Value;
        _logger = logger;
    }
    
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!_options.Enabled)
        {
            _logger.LogDebug("Filter disabled in configuration");
            return;
        }
        
        var path = context.HttpContext.Request.Path.Value;
        if (_options.ExcludedPaths.Contains(path))
        {
            _logger.LogDebug("Path {Path} excluded", path);
            return;
        }
        
        // Use _options.MaxRetries, _options.Timeout, etc.
    }
    
    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Register as global filter
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ConfiguredFilter>();
});
```

---

### IOptions vs IOptionsSnapshot vs IOptionsMonitor

**Complete Comparison:**

```csharp
public class FilterWithAllOptions : IActionFilter
{
    private readonly FilterOptions _options;           // IOptions
    private readonly IOptionsSnapshot<FilterOptions> _snapshot;
    private readonly IOptionsMonitor<FilterOptions> _monitor;
    
    public FilterWithAllOptions(
        IOptions<FilterOptions> options,           // Singleton
        IOptionsSnapshot<FilterOptions> snapshot,  // Scoped
        IOptionsMonitor<FilterOptions> monitor)    // Singleton + Reload
    {
        _options = options.Value;
        _snapshot = snapshot;
        _monitor = monitor;
    }
    
    public void OnActionExecuting(ActionExecutingContext context)
    {
        // IOptions - never reloads, same value always
        var opt1 = _options;
        
        // IOptionsSnapshot - reloads per request
        var opt2 = _snapshot.Value;
        
        // IOptionsMonitor - reloads immediately when config changes
        var opt3 = _monitor.CurrentValue;
    }
    
    public void OnActionExecuted(ActionExecutedContext context) { }
}
```

**Comparison Table:**

| Feature | IOptions<T> | IOptionsSnapshot<T> | IOptionsMonitor<T> |
|---------|-------------|---------------------|-------------------|
| **Lifetime** | Singleton | Scoped | Singleton |
| **Reload** | Never | Per request | Immediate |
| **Performance** | Fastest | Medium | Slow |
| **Use Case** | Static config | Per-request reload | Live reload |
| **Registration** | .Value property | .Value property | .CurrentValue property |

---

### Logging Configuration

**appsettings.json:**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Warning",
      "MyApp.Controllers": "Debug",
      "MyApp.Filters": "Trace"
    }
  }
}
```

**appsettings.Development.json:**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  }
}
```

**appsettings.Production.json:**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "MyApp": "Information"
    }
  }
}
```

---

### Serilog Configuration (Complete)

**appsettings.json:**

```json
{
  "Serilog": {
    "Using": [
      "Serilog.Sinks.Console",
      "Serilog.Sinks.File",
      "Serilog.Sinks.Seq"
    ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.AspNetCore": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"
        }
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/app-.log",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 30,
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] [{SourceContext}] {Message:lj}{NewLine}{Exception}",
          "fileSizeLimitBytes": 10485760,
          "rollOnFileSizeLimit": true
        }
      },
      {
        "Name": "Seq",
        "Args": {
          "serverUrl": "http://localhost:5341",
          "apiKey": ""
        }
      }
    ],
    "Enrich": [
      "FromLogContext",
      "WithMachineName",
      "WithThreadId"
    ],
    "Properties": {
      "Application": "MyApp",
      "Environment": "Production"
    }
  }
}
```

**Program.cs:**

```csharp
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog from appsettings.json
builder.Host.UseSerilog((context, configuration) =>
{
    configuration.ReadFrom.Configuration(context.Configuration);
});

var app = builder.Build();

try
{
    Log.Information("Starting application");
    app.MapControllers();
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

---

## 12. Advanced Topics

### Async Filters

**When to use:**
- ✅ I/O operations (database, HTTP calls)
- ✅ Better performance
- ✅ Recommended for production

**IAsyncActionFilter:**

```csharp
public class AsyncActionFilter : IAsyncActionFilter
{
    private readonly ILogger _logger;
    private readonly IHttpClientFactory _httpClientFactory;
    
    public AsyncActionFilter(
        ILogger<AsyncActionFilter> logger,
        IHttpClientFactory httpClientFactory)
    {
        _logger = logger;
        _httpClientFactory = httpClientFactory;
    }
    
    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        // Before action
        _logger.LogInformation("Before action");
        
        // Can do async I/O
        var client = _httpClientFactory.CreateClient();
        var response = await client.GetAsync("https://api.example.com/validate");
        
        if (!response.IsSuccessStatusCode)
        {
            context.Result = new UnauthorizedResult();
            return; // Short-circuit
        }
        
        // Execute action
        var resultContext = await next();
        
        // After action
        _logger.LogInformation("After action");
        
        // Can modify result
        if (resultContext.Result is ObjectResult objectResult)
        {
            // Process result asynchronously
            await ProcessResultAsync(objectResult.Value);
        }
    }
    
    private async Task ProcessResultAsync(object result)
    {
        // Async processing
        await Task.Delay(100);
    }
}
```

---

### Filter Factory

**When to use:**
- ✅ Create filter instances dynamically
- ✅ Complex initialization logic

**IFilterFactory:**

```csharp
public class FilterFactoryAttribute : Attribute, IFilterFactory
{
    public bool IsReusable => false; // Create new instance each time
    
    public IFilterMetadata CreateInstance(IServiceProvider serviceProvider)
    {
        var logger = serviceProvider.GetRequiredService<ILogger<MyFilter>>();
        var config = serviceProvider.GetRequiredService<IConfiguration>();
        
        // Complex initialization
        var apiKey = config["ApiKey"];
        
        return new MyFilter(logger, apiKey);
    }
}

public class MyFilter : IActionFilter
{
    private readonly ILogger _logger;
    private readonly string _apiKey;
    
    public MyFilter(ILogger logger, string apiKey)
    {
        _logger = logger;
        _apiKey = apiKey;
    }
    
    public void OnActionExecuting(ActionExecutingContext context) { }
    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage
[FilterFactory]
public class MyController : ControllerBase { }
```

---

### Short-Circuit Pipeline

**Authorization Filter Short-Circuit:**

```csharp
public class QuotaAuthorizationFilter : IAuthorizationFilter
{
    public void OnAuthorization(AuthorizationFilterContext context)
    {
        var user = context.HttpContext.User;
        
        if (!HasQuota(user))
        {
            // Short-circuit! No other filters or action will run
            context.Result = new StatusCodeResult(429); // Too Many Requests
        }
        
        // If context.Result is null, pipeline continues
    }
}
```

**Resource Filter Short-Circuit (Caching):**

```csharp
public class CacheResourceFilter : IResourceFilter
{
    private readonly IMemoryCache _cache;
    
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        var cacheKey = context.HttpContext.Request.Path;
        
        if (_cache.TryGetValue(cacheKey, out ObjectResult cached))
        {
            context.Result = cached; // Short-circuit!
            return;
        }
    }
    
    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        // Cache the result
    }
}
```

---

### Custom Log Providers

**Create Custom Provider:**

```csharp
public class DatabaseLoggerProvider : ILoggerProvider
{
    private readonly string _connectionString;
    
    public DatabaseLoggerProvider(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public ILogger CreateLogger(string categoryName)
    {
        return new DatabaseLogger(categoryName, _connectionString);
    }
    
    public void Dispose() { }
}

public class DatabaseLogger : ILogger
{
    private readonly string _categoryName;
    private readonly string _connectionString;
    
    public DatabaseLogger(string categoryName, string connectionString)
    {
        _categoryName = categoryName;
        _connectionString = connectionString;
    }
    
    public IDisposable BeginScope<TState>(TState state) => null;
    
    public bool IsEnabled(LogLevel logLevel) => logLevel >= LogLevel.Information;
    
    public void Log<TState>(
        LogLevel logLevel,
        EventId eventId,
        TState state,
        Exception exception,
        Func<TState, Exception, string> formatter)
    {
        if (!IsEnabled(logLevel)) return;
        
        var message = formatter(state, exception);
        
        // Write to database
        using var connection = new SqlConnection(_connectionString);
        connection.Open();
        
        var command = new SqlCommand(
            "INSERT INTO Logs (Timestamp, Level, Category, Message, Exception) " +
            "VALUES (@ts, @level, @category, @message, @exception)",
            connection);
        
        command.Parameters.AddWithValue("@ts", DateTime.UtcNow);
        command.Parameters.AddWithValue("@level", logLevel.ToString());
        command.Parameters.AddWithValue("@category", _categoryName);
        command.Parameters.AddWithValue("@message", message);
        command.Parameters.AddWithValue("@exception", exception?.ToString() ?? "");
        
        command.ExecuteNonQuery();
    }
}

// Register in Program.cs
builder.Logging.AddProvider(new DatabaseLoggerProvider(connectionString));
```

---

### Multiple Exception Handlers

**Combine Middleware + Filter:**

```csharp
var app = builder.Build();

// 1. Exception Middleware (catches everything)
app.UseGlobalExceptionHandler();

// ... other middleware ...

// 2. Exception Filter (catches controller exceptions)
builder.Services.AddControllers(options =>
{
    options.Filters.Add<GlobalExceptionFilter>();
});

// Priority:
// - Filter handles controller exceptions first
// - Middleware catches anything the filter missed
```

---

## 13. Performance Tips

### 1. Use Async Filters for I/O

```csharp
// ❌ BAD - Blocking I/O
public class SyncFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        var data = _httpClient.GetAsync("url").Result; // Blocks thread!
    }
}

// ✅ GOOD - Async I/O
public class AsyncFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        var data = await _httpClient.GetAsync("url"); // Non-blocking!
        await next();
    }
}
```

---

### 2. Minimize Filter Overhead

```csharp
// ❌ BAD - Heavy operation in every request
public class HeavyFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        var data = LoadLargeDataset(); // Expensive!
    }
}

// ✅ GOOD - Cache expensive data
public class OptimizedFilter : IActionFilter
{
    private static readonly Lazy<Data> _cachedData = 
        new Lazy<Data>(() => LoadLargeDataset());
    
    public void OnActionExecuting(ActionExecutingContext context)
    {
        var data = _cachedData.Value; // Fast!
    }
}
```

---

### 3. Use IOptionsMonitor Carefully

```csharp
// ⚠️ CAREFUL - Reloads on every config change
public class MonitorFilter : IActionFilter
{
    private readonly IOptionsMonitor<MyOptions> _monitor;
    
    public void OnActionExecuting(ActionExecutingContext context)
    {
        var options = _monitor.CurrentValue; // May reload from file!
    }
}

// ✅ BETTER - Use IOptions for static config
public class OptionsFilter : IActionFilter
{
    private readonly MyOptions _options;
    
    public OptionsFilter(IOptions<MyOptions> options)
    {
        _options = options.Value; // Cached!
    }
}
```

---

### 4. Short-Circuit Early

```csharp
public class EarlyShortCircuitFilter : IAuthorizationFilter
{
    public void OnAuthorization(AuthorizationFilterContext context)
    {
        // Check cheapest conditions first
        if (!context.HttpContext.Request.Headers.ContainsKey("X-API-Key"))
        {
            context.Result = new UnauthorizedResult(); // Fast exit!
            return;
        }
        
        // More expensive checks only if needed
        var isValid = ValidateApiKey();
        if (!isValid)
        {
            context.Result = new UnauthorizedResult();
        }
    }
}
```

---

### 5. Use Structured Logging Efficiently

```csharp
// ❌ BAD - String interpolation before log level check
_logger.LogDebug($"Processing {complexObject.ToString()}"); 

// ✅ GOOD - Only evaluates if Debug is enabled
_logger.LogDebug("Processing {Object}", complexObject);

// ✅ EVEN BETTER - Manual check for expensive operations
if (_logger.IsEnabled(LogLevel.Debug))
{
    var expensiveString = BuildExpensiveDebugString();
    _logger.LogDebug("Debug: {Info}", expensiveString);
}
```

---

## Summary: Complete Checklist

### Creating Filters:
- [ ] Choose correct filter type (Authorization, Resource, Action, Exception, Result)
- [ ] Decide on implementation (Attribute, TypeFilter, Global)
- [ ] Use async versions for I/O operations
- [ ] Add proper error handling
- [ ] Create extension methods for clean API

### Logging:
- [ ] Inject ILogger<T> not ILogger
- [ ] Use structured logging (not string interpolation)
- [ ] Set appropriate log levels
- [ ] Configure Serilog for production
- [ ] Never log sensitive data

### Error Handling:
- [ ] Implement global exception filter
- [ ] Use ProblemDetails for errors
- [ ] Add correlation IDs
- [ ] Log exceptions with context
- [ ] Return appropriate HTTP status codes

### Configuration:
- [ ] Use IOptions pattern for production
- [ ] Configure from appsettings.json
- [ ] Use environment-specific settings
- [ ] Validate options on startup

### Performance:
- [ ] Use async filters for I/O
- [ ] Short-circuit early when possible
- [ ] Cache expensive operations
- [ ] Use correct IOptions variant

---

**Version Information:**
- ✨ ASP.NET Core 9.0
- 📦 C# 13
- 🎯 .NET 9 SDK

**This completes Guide 8: Filters, Logging & Error Handling!**
