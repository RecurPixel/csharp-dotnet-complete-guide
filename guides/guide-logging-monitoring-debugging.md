# Logging, Monitoring & Debugging Quick Reference

---

## Why Logging, Monitoring & Debugging?

**Logging** = Recording application events for troubleshooting and analysis
**Monitoring** = Observing application health and performance in real-time
**Debugging** = Finding and fixing bugs in code

**Benefits:**
- 🔍 **Troubleshooting** - Diagnose issues quickly
- 📊 **Performance** - Track metrics and bottlenecks
- 🔒 **Security** - Detect suspicious activity
- 📈 **Analytics** - Understand user behavior
- ⚠️ **Alerting** - Get notified of problems
- 📝 **Audit Trail** - Track changes and actions

---

## Logging Levels

### Standard Log Levels

```csharp
LogLevel.Trace       // Very detailed (e.g., method entry/exit)
LogLevel.Debug       // Debugging information
LogLevel.Information // General informational messages
LogLevel.Warning     // Unexpected but recoverable events
LogLevel.Error       // Errors that don't stop the application
LogLevel.Critical    // Fatal errors that crash the application
```

### When to Use Each Level

```csharp
// Trace - Very detailed for debugging
_logger.LogTrace("Entering GetCustomer method with id: {CustomerId}", customerId);

// Debug - Debugging information
_logger.LogDebug("Cache hit for customer {CustomerId}", customerId);

// Information - General flow of the application
_logger.LogInformation("Customer {CustomerId} created successfully", customerId);

// Warning - Unexpected but handled
_logger.LogWarning("Customer {CustomerId} not found in cache, fetching from database", customerId);

// Error - Errors that are handled
_logger.LogError(ex, "Failed to update customer {CustomerId}", customerId);

// Critical - Fatal errors
_logger.LogCritical(ex, "Database connection failed, application cannot continue");
```

---

## Built-in Logging (ILogger)

### Basic Setup

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Configure logging
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();
builder.Logging.AddEventSourceLogger();

// Set minimum level
builder.Logging.SetMinimumLevel(LogLevel.Information);

var app = builder.Build();
```

### Configuration (appsettings.json)

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.EntityFrameworkCore": "Warning"
    }
  }
}
```

### Basic Usage

```csharp
public class CustomerService
{
    private readonly ILogger<CustomerService> _logger;
    private readonly ICustomerRepository _repository;
    
    public CustomerService(
        ILogger<CustomerService> logger,
        ICustomerRepository repository)
    {
        _logger = logger;
        _repository = repository;
    }
    
    public async Task<Customer?> GetCustomerAsync(int id)
    {
        _logger.LogInformation("Getting customer with id {CustomerId}", id);
        
        try
        {
            var customer = await _repository.GetByIdAsync(id);
            
            if (customer == null)
            {
                _logger.LogWarning("Customer {CustomerId} not found", id);
                return null;
            }
            
            _logger.LogInformation("Successfully retrieved customer {CustomerId}", id);
            return customer;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error getting customer {CustomerId}", id);
            throw;
        }
    }
}
```

---

## Structured Logging with Serilog

### Installation

```bash
# Install Serilog packages
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Sinks.Seq
```

### Setup

```csharp
// Program.cs
using Serilog;

// Create logger first
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithEnvironmentName()
    .WriteTo.Console()
    .WriteTo.File(
        path: "logs/app-.log",
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 7)
    .CreateLogger();

try
{
    Log.Information("Starting application");
    
    var builder = WebApplication.CreateBuilder(args);
    
    // Use Serilog
    builder.Host.UseSerilog();
    
    // Rest of configuration...
    
    var app = builder.Build();
    
    app.UseSerilogRequestLogging(); // Log HTTP requests
    
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application failed to start");
}
finally
{
    Log.CloseAndFlush();
}
```

### Configuration (appsettings.json)

```json
{
  "Serilog": {
    "Using": ["Serilog.Sinks.Console", "Serilog.Sinks.File"],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      { "Name": "Console" },
      {
        "Name": "File",
        "Args": {
          "path": "logs/app-.log",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 7
        }
      }
    ],
    "Enrich": ["FromLogContext", "WithMachineName", "WithThreadId"]
  }
}
```

### Structured Logging Examples

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        // Structured logging with properties
        _logger.LogInformation(
            "Creating order for customer {CustomerId} with {ItemCount} items totaling {TotalAmount:C}",
            request.CustomerId,
            request.Items.Count,
            request.TotalAmount);
        
        try
        {
            var order = await ProcessOrderAsync(request);
            
            // Log with multiple properties
            _logger.LogInformation(
                "Order {OrderId} created successfully. Customer: {CustomerId}, Total: {TotalAmount:C}, Items: {@Items}",
                order.Id,
                order.CustomerId,
                order.TotalAmount,
                order.Items); // @ prefix for object serialization
            
            return order;
        }
        catch (Exception ex)
        {
            // Log exception with context
            _logger.LogError(ex,
                "Failed to create order for customer {CustomerId}. Request: {@Request}",
                request.CustomerId,
                request);
            
            throw;
        }
    }
}
```

### Log Enrichers

```csharp
// Custom enricher
public class UserEnricher : ILogEventEnricher
{
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public UserEnricher(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }
    
    public void Enrich(LogEvent logEvent, ILogEventPropertyFactory propertyFactory)
    {
        var context = _httpContextAccessor.HttpContext;
        if (context?.User?.Identity?.IsAuthenticated == true)
        {
            var userIdProperty = propertyFactory.CreateProperty("UserId", context.User.FindFirst("sub")?.Value);
            logEvent.AddPropertyIfAbsent(userIdProperty);
        }
    }
}

// Register enricher
Log.Logger = new LoggerConfiguration()
    .Enrich.With<UserEnricher>()
    .CreateLogger();
```

### Serilog Sinks

```csharp
// Multiple sinks with different configurations
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    
    // Console with colored output
    .WriteTo.Console(
        outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}")
    
    // File with JSON format
    .WriteTo.File(
        new JsonFormatter(),
        path: "logs/app-.json",
        rollingInterval: RollingInterval.Day)
    
    // File for errors only
    .WriteTo.File(
        path: "logs/errors-.log",
        rollingInterval: RollingInterval.Day,
        restrictedToMinimumLevel: LogEventLevel.Error)
    
    // Seq (centralized logging)
    .WriteTo.Seq("http://localhost:5341")
    
    // Email for critical errors
    .WriteTo.Email(
        fromEmail: "alerts@myapp.com",
        toEmail: "team@myapp.com",
        mailServer: "smtp.gmail.com",
        restrictedToMinimumLevel: LogEventLevel.Error)
    
    .CreateLogger();
```

---

## Application Insights

### Setup

```bash
# Install Application Insights
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add Application Insights
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
});

var app = builder.Build();
```

### Configuration

```json
// appsettings.json
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey=your-key;IngestionEndpoint=https://..."
  }
}
```

### Custom Telemetry

```csharp
public class OrderService
{
    private readonly TelemetryClient _telemetry;
    private readonly ILogger<OrderService> _logger;
    
    public OrderService(TelemetryClient telemetry, ILogger<OrderService> logger)
    {
        _telemetry = telemetry;
        _logger = logger;
    }
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var order = await ProcessOrderAsync(request);
            
            // Track custom event
            _telemetry.TrackEvent("OrderCreated", new Dictionary<string, string>
            {
                ["OrderId"] = order.Id.ToString(),
                ["CustomerId"] = order.CustomerId.ToString(),
                ["TotalAmount"] = order.TotalAmount.ToString()
            });
            
            // Track custom metric
            _telemetry.TrackMetric("OrderValue", order.TotalAmount);
            
            // Track dependency
            stopwatch.Stop();
            _telemetry.TrackDependency(
                "Database",
                "CreateOrder",
                startTime: DateTimeOffset.UtcNow - stopwatch.Elapsed,
                duration: stopwatch.Elapsed,
                success: true);
            
            return order;
        }
        catch (Exception ex)
        {
            // Track exception
            _telemetry.TrackException(ex, new Dictionary<string, string>
            {
                ["CustomerId"] = request.CustomerId.ToString(),
                ["Operation"] = "CreateOrder"
            });
            
            throw;
        }
    }
    
    public void TrackUserAction(string action, Dictionary<string, string> properties)
    {
        _telemetry.TrackEvent(action, properties);
    }
    
    public void TrackPageView(string pageName)
    {
        _telemetry.TrackPageView(pageName);
    }
}
```

### Custom Telemetry Initializer

```csharp
public class CustomTelemetryInitializer : ITelemetryInitializer
{
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public CustomTelemetryInitializer(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }
    
    public void Initialize(ITelemetry telemetry)
    {
        var context = _httpContextAccessor.HttpContext;
        
        if (context?.User?.Identity?.IsAuthenticated == true)
        {
            telemetry.Context.User.AuthenticatedUserId = 
                context.User.FindFirst("sub")?.Value;
        }
        
        // Add custom properties
        if (telemetry is ISupportProperties properties)
        {
            properties.Properties["Environment"] = 
                Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Unknown";
        }
    }
}

// Register
builder.Services.AddSingleton<ITelemetryInitializer, CustomTelemetryInitializer>();
```

---

## Health Checks

### Setup

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy())
    .AddSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        name: "database",
        timeout: TimeSpan.FromSeconds(3))
    .AddRedis(
        builder.Configuration.GetConnectionString("Redis"),
        name: "redis")
    .AddUrlGroup(
        new Uri("https://api.external.com/health"),
        name: "external-api",
        timeout: TimeSpan.FromSeconds(3));

var app = builder.Build();

// Endpoints
app.MapHealthChecks("/health");
app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
});
app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false
});

// Detailed health check response
app.MapHealthChecks("/health/detailed", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        var result = JsonSerializer.Serialize(new
        {
            status = report.Status.ToString(),
            checks = report.Entries.Select(e => new
            {
                name = e.Key,
                status = e.Value.Status.ToString(),
                description = e.Value.Description,
                duration = e.Value.Duration.TotalMilliseconds
            }),
            totalDuration = report.TotalDuration.TotalMilliseconds
        });
        
        context.Response.ContentType = "application/json";
        await context.Response.WriteAsync(result);
    }
});
```

### Custom Health Check

```csharp
public class DatabaseHealthCheck : IHealthCheck
{
    private readonly IDbConnection _connection;
    
    public DatabaseHealthCheck(IDbConnection connection)
    {
        _connection = connection;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            await _connection.ExecuteScalarAsync<int>("SELECT 1");
            return HealthCheckResult.Healthy("Database is responsive");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy(
                "Database is not responsive",
                ex);
        }
    }
}

// Register
builder.Services.AddHealthChecks()
    .AddCheck<DatabaseHealthCheck>("database");
```

---

## Debugging Techniques

### 1. Visual Studio Debugging

```csharp
public async Task<Order> ProcessOrderAsync(CreateOrderRequest request)
{
    // Breakpoint here - F9
    var customer = await _customerRepository.GetByIdAsync(request.CustomerId);
    
    // Conditional breakpoint: customer == null
    if (customer == null)
    {
        throw new NotFoundException($"Customer {request.CustomerId} not found");
    }
    
    // Watch window: request.TotalAmount
    var order = new Order
    {
        CustomerId = customer.Id,
        TotalAmount = request.TotalAmount
    };
    
    // Step Into - F11
    // Step Over - F10
    // Step Out - Shift+F11
    
    await _orderRepository.AddAsync(order);
    
    return order;
}

// Debug attributes
[DebuggerDisplay("Order {Id}: ${TotalAmount}")]
public class Order
{
    public int Id { get; set; }
    public decimal TotalAmount { get; set; }
    
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    public string InternalField { get; set; }
}
```

### 2. Diagnostic Logging

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    
    public async Task<Order> ProcessOrderAsync(CreateOrderRequest request)
    {
        using (_logger.BeginScope("OrderProcessing-{OrderId}", Guid.NewGuid()))
        {
            _logger.LogDebug("Starting order processing. Request: {@Request}", request);
            
            var stopwatch = Stopwatch.StartNew();
            
            try
            {
                // Step 1
                _logger.LogDebug("Validating customer {CustomerId}", request.CustomerId);
                var customer = await ValidateCustomerAsync(request.CustomerId);
                _logger.LogDebug("Customer validation completed in {Ms}ms", stopwatch.ElapsedMilliseconds);
                
                // Step 2
                stopwatch.Restart();
                _logger.LogDebug("Calculating order total");
                var total = CalculateTotal(request.Items);
                _logger.LogDebug("Order total calculated: {Total:C} in {Ms}ms", total, stopwatch.ElapsedMilliseconds);
                
                // Step 3
                stopwatch.Restart();
                _logger.LogDebug("Creating order in database");
                var order = await CreateOrderInDatabaseAsync(customer, request.Items, total);
                _logger.LogDebug("Order created with id {OrderId} in {Ms}ms", order.Id, stopwatch.ElapsedMilliseconds);
                
                _logger.LogInformation("Order processing completed successfully for {OrderId}", order.Id);
                
                return order;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Order processing failed at step. Elapsed: {Ms}ms", stopwatch.ElapsedMilliseconds);
                throw;
            }
        }
    }
}
```

### 3. Exception Handling & Logging

```csharp
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;
    
    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }
    
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        var (statusCode, message) = exception switch
        {
            NotFoundException => (StatusCodes.Status404NotFound, exception.Message),
            ValidationException => (StatusCodes.Status400BadRequest, exception.Message),
            UnauthorizedException => (StatusCodes.Status401Unauthorized, "Unauthorized"),
            _ => (StatusCodes.Status500InternalServerError, "An error occurred")
        };
        
        // Log with context
        _logger.LogError(exception,
            "Unhandled exception: {ExceptionType}. Path: {Path}, Method: {Method}, User: {User}",
            exception.GetType().Name,
            httpContext.Request.Path,
            httpContext.Request.Method,
            httpContext.User?.Identity?.Name ?? "Anonymous");
        
        httpContext.Response.StatusCode = statusCode;
        await httpContext.Response.WriteAsJsonAsync(new
        {
            error = message,
            traceId = Activity.Current?.Id ?? httpContext.TraceIdentifier
        }, cancellationToken);
        
        return true;
    }
}

// Register
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
```

### 4. Request/Response Logging Middleware

```csharp
public class RequestResponseLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestResponseLoggingMiddleware> _logger;
    
    public RequestResponseLoggingMiddleware(
        RequestDelegate next,
        ILogger<RequestResponseLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Log request
        await LogRequestAsync(context.Request);
        
        // Capture original response body stream
        var originalBodyStream = context.Response.Body;
        
        using var responseBody = new MemoryStream();
        context.Response.Body = responseBody;
        
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            await _next(context);
            
            stopwatch.Stop();
            
            // Log response
            await LogResponseAsync(context.Response, stopwatch.ElapsedMilliseconds);
        }
        finally
        {
            await responseBody.CopyToAsync(originalBodyStream);
        }
    }
    
    private async Task LogRequestAsync(HttpRequest request)
    {
        request.EnableBuffering();
        
        var body = await new StreamReader(request.Body).ReadToEndAsync();
        request.Body.Position = 0;
        
        _logger.LogInformation(
            "HTTP Request: {Method} {Path} {QueryString} {Body}",
            request.Method,
            request.Path,
            request.QueryString,
            body);
    }
    
    private async Task LogResponseAsync(HttpResponse response, long elapsedMs)
    {
        response.Body.Seek(0, SeekOrigin.Begin);
        var body = await new StreamReader(response.Body).ReadToEndAsync();
        response.Body.Seek(0, SeekOrigin.Begin);
        
        _logger.LogInformation(
            "HTTP Response: {StatusCode} in {ElapsedMs}ms {Body}",
            response.StatusCode,
            elapsedMs,
            body);
    }
}
```

---

## Performance Monitoring

### 1. Custom Metrics

```csharp
public class MetricsService
{
    private readonly ILogger<MetricsService> _logger;
    
    private static readonly Dictionary<string, long> _counters = new();
    private static readonly Dictionary<string, List<double>> _timings = new();
    
    public void IncrementCounter(string name)
    {
        lock (_counters)
        {
            _counters.TryGetValue(name, out var count);
            _counters[name] = count + 1;
        }
    }
    
    public void RecordTiming(string name, double milliseconds)
    {
        lock (_timings)
        {
            if (!_timings.ContainsKey(name))
                _timings[name] = new List<double>();
            
            _timings[name].Add(milliseconds);
        }
    }
    
    public void LogMetrics()
    {
        _logger.LogInformation("=== Metrics Summary ===");
        
        lock (_counters)
        {
            foreach (var counter in _counters)
            {
                _logger.LogInformation("{Metric}: {Count}", counter.Key, counter.Value);
            }
        }
        
        lock (_timings)
        {
            foreach (var timing in _timings)
            {
                var avg = timing.Value.Average();
                var min = timing.Value.Min();
                var max = timing.Value.Max();
                
                _logger.LogInformation(
                    "{Metric} - Avg: {Avg:F2}ms, Min: {Min:F2}ms, Max: {Max:F2}ms, Count: {Count}",
                    timing.Key, avg, min, max, timing.Value.Count);
            }
        }
    }
}

// Usage
public class OrderService
{
    private readonly MetricsService _metrics;
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var order = await ProcessOrderAsync(request);
            
            stopwatch.Stop();
            
            _metrics.IncrementCounter("orders.created");
            _metrics.RecordTiming("orders.create.duration", stopwatch.ElapsedMilliseconds);
            
            return order;
        }
        catch
        {
            _metrics.IncrementCounter("orders.errors");
            throw;
        }
    }
}
```

### 2. Action Filters for Timing

```csharp
public class TimingFilter : IAsyncActionFilter
{
    private readonly ILogger<TimingFilter> _logger;
    
    public TimingFilter(ILogger<TimingFilter> logger)
    {
        _logger = logger;
    }
    
    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        var stopwatch = Stopwatch.StartNew();
        
        var actionName = $"{context.Controller.GetType().Name}.{context.ActionDescriptor.DisplayName}";
        
        _logger.LogDebug("Starting {Action}", actionName);
        
        var resultContext = await next();
        
        stopwatch.Stop();
        
        if (resultContext.Exception == null)
        {
            _logger.LogInformation(
                "Completed {Action} in {Ms}ms",
                actionName,
                stopwatch.ElapsedMilliseconds);
        }
        else
        {
            _logger.LogError(
                resultContext.Exception,
                "Failed {Action} after {Ms}ms",
                actionName,
                stopwatch.ElapsedMilliseconds);
        }
    }
}

// Register globally
builder.Services.AddControllers(options =>
{
    options.Filters.Add<TimingFilter>();
});
```

---

## Best Practices

### 1. Use Structured Logging

```csharp
// ❌ Bad - String interpolation
_logger.LogInformation($"User {userId} logged in");

// ✅ Good - Structured logging
_logger.LogInformation("User {UserId} logged in", userId);

// ❌ Bad - ToString() on objects
_logger.LogInformation("Order created: {Order}", order.ToString());

// ✅ Good - Serialize object
_logger.LogInformation("Order created: {@Order}", order);
```

### 2. Don't Log Sensitive Data

```csharp
// ❌ Bad - Logging passwords, credit cards, etc.
_logger.LogInformation("User login: {Email} {Password}", email, password);

// ✅ Good - Mask sensitive data
_logger.LogInformation("User login: {Email} {PasswordHash}", email, HashPassword(password));

// Custom object with sensitive data
public class User
{
    public string Email { get; set; }
    
    [JsonIgnore] // Don't serialize in logs
    public string Password { get; set; }
    
    public string PasswordHash { get; set; }
}
```

### 3. Use Log Scopes

```csharp
public async Task<Order> ProcessOrderAsync(int orderId)
{
    using (_logger.BeginScope(new Dictionary<string, object>
    {
        ["OrderId"] = orderId,
        ["CorrelationId"] = Guid.NewGuid()
    }))
    {
        _logger.LogInformation("Starting order processing");
        
        // All logs in this scope will have OrderId and CorrelationId
        
        var order = await _repository.GetByIdAsync(orderId);
        _logger.LogInformation("Order loaded");
        
        await ProcessPaymentAsync(order);
        _logger.LogInformation("Payment processed");
        
        return order;
    }
}
```

### 4. Appropriate Log Levels

```csharp
public class CustomerService
{
    public async Task<Customer?> GetCustomerAsync(int id)
    {
        // Trace - Very detailed
        _logger.LogTrace("Entering GetCustomerAsync with {CustomerId}", id);
        
        // Debug - Useful for debugging
        _logger.LogDebug("Checking cache for customer {CustomerId}", id);
        
        var customer = await _cache.GetAsync<Customer>($"customer_{id}");
        
        if (customer != null)
        {
            // Debug - Cache hit
            _logger.LogDebug("Cache hit for customer {CustomerId}", id);
            return customer;
        }
        
        // Debug - Cache miss
        _logger.LogDebug("Cache miss for customer {CustomerId}, fetching from database", id);
        
        customer = await _repository.GetByIdAsync(id);
        
        if (customer == null)
        {
            // Warning - Expected but shouldn't happen often
            _logger.LogWarning("Customer {CustomerId} not found", id);
            return null;
        }
        
        // Information - Normal flow
        _logger.LogInformation("Customer {CustomerId} retrieved successfully", id);
        
        return customer;
    }
    
    public async Task UpdateCustomerAsync(Customer customer)
    {
        try
        {
            await _repository.UpdateAsync(customer);
            _logger.LogInformation("Customer {CustomerId} updated successfully", customer.Id);
        }
        catch (DbUpdateException ex)
        {
            // Error - Recoverable error
            _logger.LogError(ex, "Failed to update customer {CustomerId}", customer.Id);
            throw;
        }
        catch (Exception ex)
        {
            // Critical - Unexpected error
            _logger.LogCritical(ex, "Unexpected error updating customer {CustomerId}", customer.Id);
            throw;
        }
    }
}
```

### 5. Performance Considerations

```csharp
// ❌ Bad - Always evaluates even if not logged
_logger.LogDebug("Data: " + ExpensiveOperation());

// ✅ Good - Check if enabled first
if (_logger.IsEnabled(LogLevel.Debug))
{
    _logger.LogDebug("Data: {Data}", ExpensiveOperation());
}

// ✅ Better - Use LoggerMessage (compiled)
private static readonly Action<ILogger, int, Exception?> _logCustomerRetrieved =
    LoggerMessage.Define<int>(
        LogLevel.Information,
        new EventId(1, nameof(GetCustomerAsync)),
        "Customer {CustomerId} retrieved successfully");

public async Task<Customer?> GetCustomerAsync(int id)
{
    var customer = await _repository.GetByIdAsync(id);
    
    if (customer != null)
    {
        _logCustomerRetrieved(_logger, id, null);
    }
    
    return customer;
}
```

---

## Monitoring Dashboard Example

### Custom Health Check Dashboard

```csharp
public class HealthCheckService
{
    private readonly IHealthCheckService _healthCheck;
    
    public async Task<HealthReport> GetHealthAsync()
    {
        return await _healthCheck.CheckHealthAsync();
    }
    
    public async Task<Dictionary<string, object>> GetMetricsAsync()
    {
        return new Dictionary<string, object>
        {
            ["cpu_usage"] = GetCpuUsage(),
            ["memory_usage"] = GetMemoryUsage(),
            ["active_connections"] = GetActiveConnections(),
            ["requests_per_second"] = GetRequestsPerSecond(),
            ["average_response_time"] = GetAverageResponseTime()
        };
    }
    
    private double GetCpuUsage()
    {
        // Implementation
        return 45.2;
    }
    
    private long GetMemoryUsage()
    {
        return GC.GetTotalMemory(false);
    }
    
    private int GetActiveConnections()
    {
        // Implementation
        return 150;
    }
    
    private double GetRequestsPerSecond()
    {
        // Implementation
        return 250.5;
    }
    
    private double GetAverageResponseTime()
    {
        // Implementation
        return 45.3;
    }
}
```

---

## Common Debugging Scenarios

### 1. Tracking Down Null Reference Exception

```csharp
public class DebuggingExample
{
    private readonly ILogger<DebuggingExample> _logger;
    
    public async Task ProcessDataAsync(DataRequest request)
    {
        try
        {
            _logger.LogDebug("Processing data. Request: {@Request}", request);
            
            // Add null checks with logging
            if (request == null)
            {
                _logger.LogError("Request is null");
                throw new ArgumentNullException(nameof(request));
            }
            
            _logger.LogDebug("Request validated. Items count: {Count}", request.Items?.Count ?? 0);
            
            if (request.Items == null)
            {
                _logger.LogError("Request.Items is null");
                throw new ArgumentNullException(nameof(request.Items));
            }
            
            foreach (var item in request.Items)
            {
                _logger.LogDebug("Processing item: {@Item}", item);
                
                if (item == null)
                {
                    _logger.LogWarning("Null item found in collection, skipping");
                    continue;
                }
                
                await ProcessItemAsync(item);
            }
        }
        catch (NullReferenceException ex)
        {
            // Log with full context
            _logger.LogError(ex,
                "NullReferenceException during processing. Request: {@Request}",
                request);
            throw;
        }
    }
}
```

### 2. Performance Bottleneck Investigation

```csharp
public class PerformanceDebugging
{
    private readonly ILogger<PerformanceDebugging> _logger;
    
    public async Task<List<Order>> GetOrdersAsync(int customerId)
    {
        var totalStopwatch = Stopwatch.StartNew();
        
        _logger.LogDebug("Starting GetOrders for customer {CustomerId}", customerId);
        
        // Step 1
        var sw1 = Stopwatch.StartNew();
        var customer = await _customerRepository.GetByIdAsync(customerId);
        sw1.Stop();
        _logger.LogDebug("Customer fetch took {Ms}ms", sw1.ElapsedMilliseconds);
        
        // Step 2
        var sw2 = Stopwatch.StartNew();
        var orders = await _orderRepository.GetByCustomerIdAsync(customerId);
        sw2.Stop();
        _logger.LogDebug("Orders fetch took {Ms}ms, Count: {Count}", 
            sw2.ElapsedMilliseconds, orders.Count);
        
        // Step 3 - Suspected bottleneck
        var sw3 = Stopwatch.StartNew();
        foreach (var order in orders)
        {
            var itemSw = Stopwatch.StartNew();
            order.Items = await _orderItemRepository.GetByOrderIdAsync(order.Id);
            itemSw.Stop();
            
            if (itemSw.ElapsedMilliseconds > 100)
            {
                _logger.LogWarning(
                    "Slow item fetch for order {OrderId}: {Ms}ms",
                    order.Id, itemSw.ElapsedMilliseconds);
            }
        }
        sw3.Stop();
        _logger.LogDebug("All items fetch took {Ms}ms", sw3.ElapsedMilliseconds);
        
        totalStopwatch.Stop();
        _logger.LogInformation(
            "GetOrders completed in {Ms}ms (Customer: {S1}ms, Orders: {S2}ms, Items: {S3}ms)",
            totalStopwatch.ElapsedMilliseconds,
            sw1.ElapsedMilliseconds,
            sw2.ElapsedMilliseconds,
            sw3.ElapsedMilliseconds);
        
        return orders;
    }
}
```

---

## Quick Reference Checklist

### Logging Checklist
- [ ] Use structured logging (not string interpolation)
- [ ] Appropriate log levels (Trace/Debug/Info/Warning/Error/Critical)
- [ ] Include relevant context (IDs, usernames, etc.)
- [ ] Don't log sensitive data (passwords, credit cards, etc.)
- [ ] Use log scopes for related operations
- [ ] Configure retention policies
- [ ] Set up centralized logging (Seq, Application Insights)

### Monitoring Checklist
- [ ] Health checks configured
- [ ] Application Insights or similar APM tool
- [ ] Custom metrics for business operations
- [ ] Alerting for critical errors
- [ ] Performance monitoring
- [ ] Resource usage tracking
- [ ] Uptime monitoring

### Debugging Checklist
- [ ] Enable detailed logging in development
- [ ] Use breakpoints effectively
- [ ] Log method entry/exit in complex flows
- [ ] Time expensive operations
- [ ] Check for null references
- [ ] Validate inputs
- [ ] Handle exceptions with context
- [ ] Use diagnostic tools (profilers, memory analyzers)

---

**Guide Complete!** This comprehensive Logging, Monitoring & Debugging guide covers structured logging with Serilog, Application Insights, health checks, debugging techniques, performance monitoring, and best practices for production-ready .NET applications! 🔍
