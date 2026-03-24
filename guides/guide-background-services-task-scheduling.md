# Background Services & Task Scheduling Quick Reference

---

## What are Background Services?

**Background Service** = Long-running task that executes independently of HTTP request/response cycle

**Common Use Cases:**
- ✅ **Scheduled jobs** - Send emails at midnight, generate daily reports
- ✅ **Queue processing** - Process messages from a message queue
- ✅ **Health monitoring** - Periodic health checks, cleanup tasks
- ✅ **Cache warming** - Pre-load data on startup
- ✅ **Event processing** - Handle domain events asynchronously
- ✅ **File processing** - Watch directories, process uploads

### How It Fits in .NET

```
ASP.NET Core Host
├── Web Server (Kestrel)         ← Handles HTTP requests
├── Dependency Injection
├── Configuration
└── Hosted Services              ← Your background work lives here
    ├── IHostedService
    ├── BackgroundService
    └── Worker Services
```

---

## IHostedService

**IHostedService** = Base interface for all hosted services in .NET

```csharp
public interface IHostedService
{
    Task StartAsync(CancellationToken cancellationToken);
    Task StopAsync(CancellationToken cancellationToken);
}
```

### Basic Implementation

```csharp
public class MyHostedService : IHostedService, IDisposable
{
    private readonly ILogger<MyHostedService> _logger;
    private Timer? _timer;

    public MyHostedService(ILogger<MyHostedService> logger)
    {
        _logger = logger;
    }

    public Task StartAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Service starting...");

        // Create timer - fires every 5 seconds
        _timer = new Timer(DoWork, null, TimeSpan.Zero, TimeSpan.FromSeconds(5));

        return Task.CompletedTask;
    }

    private void DoWork(object? state)
    {
        _logger.LogInformation("Work executing at: {time}", DateTimeOffset.Now);
        // Your work here
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Service stopping...");
        _timer?.Change(Timeout.Infinite, 0); // Stop the timer
        return Task.CompletedTask;
    }

    public void Dispose()
    {
        _timer?.Dispose();
    }
}
```

### Register in Program.cs

```csharp
// Program.cs
builder.Services.AddHostedService<MyHostedService>();
```

### When to Use IHostedService

```
✅ Use IHostedService when:
   - You need full control over start/stop lifecycle
   - Running simple periodic tasks with Timer
   - You need to run something at startup AND shutdown

❌ Avoid IHostedService when:
   - You need async work loops (use BackgroundService instead)
   - Complex background processing (use Hangfire)
```

---

## BackgroundService

**BackgroundService** = Abstract base class that implements `IHostedService` — the recommended approach for async long-running tasks

```csharp
public abstract class BackgroundService : IHostedService, IDisposable
{
    protected abstract Task ExecuteAsync(CancellationToken stoppingToken);

    public virtual Task StartAsync(CancellationToken cancellationToken) { ... }
    public virtual Task StopAsync(CancellationToken cancellationToken) { ... }
}
```

### Basic BackgroundService

```csharp
public class DataProcessingService : BackgroundService
{
    private readonly ILogger<DataProcessingService> _logger;

    public DataProcessingService(ILogger<DataProcessingService> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("DataProcessingService starting");

        // Loop until application is stopping
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                await ProcessDataAsync(stoppingToken);
                await Task.Delay(TimeSpan.FromSeconds(30), stoppingToken);
            }
            catch (OperationCanceledException)
            {
                // Expected when cancellation is requested
                break;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error in DataProcessingService");
                await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken); // Wait before retry
            }
        }

        _logger.LogInformation("DataProcessingService stopping");
    }

    private async Task ProcessDataAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Processing data at: {time}", DateTimeOffset.Now);
        await Task.Delay(1000, cancellationToken); // Simulate work
    }
}
```

### Periodic Task with PeriodicTimer (.NET 6+)

```csharp
// ✅ Preferred approach in .NET 6+ - PeriodicTimer doesn't drift
public class CleanupService : BackgroundService
{
    private readonly ILogger<CleanupService> _logger;
    private readonly PeriodicTimer _timer = new(TimeSpan.FromHours(1));

    public CleanupService(ILogger<CleanupService> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (await _timer.WaitForNextTickAsync(stoppingToken))
        {
            try
            {
                await CleanupOldFilesAsync(stoppingToken);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error during cleanup");
            }
        }
    }

    private async Task CleanupOldFilesAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Starting file cleanup at: {time}", DateTimeOffset.UtcNow);
        // Cleanup logic here
        await Task.CompletedTask;
    }

    public override void Dispose()
    {
        _timer.Dispose();
        base.Dispose();
    }
}
```

### Queue-Based BackgroundService

```csharp
// Interface for the queue
public interface IBackgroundTaskQueue
{
    ValueTask QueueBackgroundWorkItemAsync(Func<CancellationToken, ValueTask> workItem);
    ValueTask<Func<CancellationToken, ValueTask>> DequeueAsync(CancellationToken cancellationToken);
}

// Queue implementation
public class BackgroundTaskQueue : IBackgroundTaskQueue
{
    private readonly Channel<Func<CancellationToken, ValueTask>> _queue;

    public BackgroundTaskQueue(int capacity = 100)
    {
        var options = new BoundedChannelOptions(capacity)
        {
            FullMode = BoundedChannelFullMode.Wait
        };
        _queue = Channel.CreateBounded<Func<CancellationToken, ValueTask>>(options);
    }

    public async ValueTask QueueBackgroundWorkItemAsync(
        Func<CancellationToken, ValueTask> workItem)
    {
        ArgumentNullException.ThrowIfNull(workItem);
        await _queue.Writer.WriteAsync(workItem);
    }

    public async ValueTask<Func<CancellationToken, ValueTask>> DequeueAsync(
        CancellationToken cancellationToken)
    {
        return await _queue.Reader.ReadAsync(cancellationToken);
    }
}

// Worker that processes the queue
public class QueuedHostedService : BackgroundService
{
    private readonly IBackgroundTaskQueue _taskQueue;
    private readonly ILogger<QueuedHostedService> _logger;

    public QueuedHostedService(
        IBackgroundTaskQueue taskQueue,
        ILogger<QueuedHostedService> logger)
    {
        _taskQueue = taskQueue;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var workItem = await _taskQueue.DequeueAsync(stoppingToken);

            try
            {
                await workItem(stoppingToken);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error occurred executing work item");
            }
        }
    }
}

// Register services
builder.Services.AddSingleton<IBackgroundTaskQueue, BackgroundTaskQueue>();
builder.Services.AddHostedService<QueuedHostedService>();

// Enqueue work from a controller or service
public class OrdersController : ControllerBase
{
    private readonly IBackgroundTaskQueue _queue;

    public OrdersController(IBackgroundTaskQueue queue)
    {
        _queue = queue;
    }

    [HttpPost]
    public async Task<IActionResult> CreateOrder([FromBody] CreateOrderDto dto)
    {
        // Save order immediately...
        var orderId = Guid.NewGuid();

        // Process email in background
        await _queue.QueueBackgroundWorkItemAsync(async (ct) =>
        {
            await SendOrderConfirmationEmailAsync(orderId, ct);
        });

        return Ok(orderId);
    }
}
```

---

## Scoped Services Inside Hosted Services

**⚠️ Common Pitfall:** Hosted services are singletons. You cannot inject scoped services (like `DbContext`) directly.

```csharp
// ❌ WRONG - DbContext is scoped, this will throw at runtime
public class BadService : BackgroundService
{
    private readonly AppDbContext _dbContext; // ❌ Scoped injected into singleton

    public BadService(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }
}

// ✅ CORRECT - Use IServiceScopeFactory to create a scope
public class GoodService : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<GoodService> _logger;

    public GoodService(IServiceScopeFactory scopeFactory, ILogger<GoodService> logger)
    {
        _scopeFactory = scopeFactory;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            // Create a new scope for each iteration
            using (var scope = _scopeFactory.CreateScope())
            {
                var dbContext = scope.ServiceProvider.GetRequiredService<AppDbContext>();
                var emailService = scope.ServiceProvider.GetRequiredService<IEmailService>();

                var pendingOrders = await dbContext.Orders
                    .Where(o => o.Status == OrderStatus.Pending)
                    .ToListAsync(stoppingToken);

                foreach (var order in pendingOrders)
                {
                    await emailService.SendConfirmationAsync(order, stoppingToken);
                }
            } // Scope disposed here - DbContext cleaned up

            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }
}
```

---

## Worker Services

**Worker Service** = Standalone .NET application for background processing — no HTTP server, just a host with background services

### Create Worker Service Project

```bash
# Create new Worker Service project
dotnet new worker -n MyWorkerService

# Project structure
MyWorkerService/
├── Program.cs
├── Worker.cs               ← Your BackgroundService
├── appsettings.json
└── MyWorkerService.csproj
```

### Worker.cs (Generated)

```csharp
public class Worker : BackgroundService
{
    private readonly ILogger<Worker> _logger;

    public Worker(ILogger<Worker> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

### Program.cs for Worker Service

```csharp
// Program.cs
using MyWorkerService;

var builder = Host.CreateApplicationBuilder(args);

// Register services
builder.Services.AddHostedService<Worker>();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddScoped<IEmailService, EmailService>();

// Logging
builder.Services.AddLogging(logging =>
{
    logging.AddConsole();
    logging.AddEventLog(); // Windows Event Log (Windows only)
});

var host = builder.Build();
host.Run();
```

### Run Worker Service as Windows Service

```csharp
// Install NuGet: Microsoft.Extensions.Hosting.WindowsServices
var builder = Host.CreateApplicationBuilder(args);

builder.Services.AddWindowsService(options =>
{
    options.ServiceName = "My Background Worker";
});

builder.Services.AddHostedService<Worker>();

var host = builder.Build();
host.Run();
```

```powershell
# Publish as self-contained
dotnet publish -c Release -r win-x64 --self-contained

# Install as Windows Service
sc create "MyWorkerService" binPath="C:\Services\MyWorkerService.exe"
sc start "MyWorkerService"

# Or use New-Service in PowerShell
New-Service -Name "MyWorkerService" `
    -BinaryPathName "C:\Services\MyWorkerService.exe" `
    -DisplayName "My Background Worker" `
    -StartupType Automatic
```

### Run Worker Service as Linux systemd Service

```bash
# Install NuGet: Microsoft.Extensions.Hosting.Systemd
```

```csharp
builder.Services.AddSystemd();
builder.Services.AddHostedService<Worker>();
```

```ini
# /etc/systemd/system/myworker.service
[Unit]
Description=My Background Worker

[Service]
Type=notify
ExecStart=/opt/myworker/MyWorkerService
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
sudo systemctl enable myworker
sudo systemctl start myworker
sudo systemctl status myworker
```

---

## Long-Running Tasks & Cancellation

**CancellationToken** is your lifeline for graceful shutdown. Always pass it through.

### Graceful Shutdown Pattern

```csharp
public class LongRunningService : BackgroundService
{
    private readonly ILogger<LongRunningService> _logger;

    public LongRunningService(ILogger<LongRunningService> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        // Register cleanup on cancellation
        stoppingToken.Register(() =>
            _logger.LogInformation("LongRunningService is stopping"));

        while (!stoppingToken.IsCancellationRequested)
        {
            await DoWorkAsync(stoppingToken);
        }
    }

    private async Task DoWorkAsync(CancellationToken cancellationToken)
    {
        try
        {
            // ✅ Pass cancellationToken to every async call
            var data = await FetchDataFromApiAsync(cancellationToken);
            await ProcessDataAsync(data, cancellationToken);
            await SaveResultsAsync(data, cancellationToken);
        }
        catch (OperationCanceledException)
        {
            // ✅ Gracefully handle cancellation
            _logger.LogInformation("Work cancelled, shutting down cleanly");
            throw; // Re-throw to exit the loop
        }
    }

    private async Task<string> FetchDataFromApiAsync(CancellationToken ct)
    {
        using var httpClient = new HttpClient();
        // ✅ HttpClient respects cancellation
        return await httpClient.GetStringAsync("https://api.example.com/data", ct);
    }
}
```

### Timeout + Cancellation

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        // Combine app shutdown token with a timeout
        using var cts = CancellationTokenSource.CreateLinkedTokenSource(stoppingToken);
        cts.CancelAfter(TimeSpan.FromSeconds(30)); // 30-second timeout per iteration

        try
        {
            await DoWorkAsync(cts.Token);
        }
        catch (OperationCanceledException) when (!stoppingToken.IsCancellationRequested)
        {
            // Timeout hit (not app shutdown)
            _logger.LogWarning("Work timed out after 30 seconds, retrying");
        }

        await Task.Delay(TimeSpan.FromSeconds(10), stoppingToken);
    }
}
```

### Parallel Background Work

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        var items = await GetWorkItemsAsync(stoppingToken);

        // Process up to 5 items in parallel
        var semaphore = new SemaphoreSlim(5);
        var tasks = items.Select(async item =>
        {
            await semaphore.WaitAsync(stoppingToken);
            try
            {
                await ProcessItemAsync(item, stoppingToken);
            }
            finally
            {
                semaphore.Release();
            }
        });

        await Task.WhenAll(tasks);
        await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
    }
}
```

---

## Hangfire

**Hangfire** = Robust background job processing with persistence, retries, and a built-in dashboard

**Key Features:**
- ✅ **Persistent jobs** - Jobs survive app restarts
- ✅ **Automatic retries** - Configurable retry policies
- ✅ **Dashboard** - Visual job monitoring UI
- ✅ **Scheduling** - Cron-based recurring jobs
- ✅ **Multiple job types** - Fire-and-forget, delayed, recurring, continuations, batch

### Installation

```bash
# Core package
dotnet add package Hangfire.AspNetCore

# SQL Server storage (recommended for production)
dotnet add package Hangfire.SqlServer

# Optional: Redis storage (for high-throughput)
dotnet add package Hangfire.Redis.StackExchange
```

### Basic Setup

```csharp
// Program.cs
builder.Services.AddHangfire(config =>
{
    config
        .SetDataCompatibilityLevel(CompatibilityLevel.Version_180)
        .UseSimpleAssemblyNameTypeSerializer()
        .UseRecommendedSerializerSettings()
        .UseSqlServerStorage(
            builder.Configuration.GetConnectionString("DefaultConnection"),
            new SqlServerStorageOptions
            {
                CommandBatchMaxTimeout = TimeSpan.FromMinutes(5),
                SlidingInvisibilityTimeout = TimeSpan.FromMinutes(5),
                QueuePollInterval = TimeSpan.Zero,
                UseRecommendedIsolationLevel = true,
                DisableGlobalLocks = true
            });
});

// Add Hangfire server (processes background jobs)
builder.Services.AddHangfireServer(options =>
{
    options.WorkerCount = Environment.ProcessorCount * 2; // Default: min(processor*5, 20)
    options.Queues = new[] { "critical", "default", "low" }; // Queue priority order
});

var app = builder.Build();

// Hangfire Dashboard UI
app.UseHangfireDashboard("/hangfire", new DashboardOptions
{
    // ⚠️ Secure the dashboard in production!
    Authorization = new[] { new HangfireAuthorizationFilter() }
});
```

### Securing the Dashboard

```csharp
// Custom authorization filter
public class HangfireAuthorizationFilter : IDashboardAuthorizationFilter
{
    public bool Authorize(DashboardContext context)
    {
        var httpContext = context.GetHttpContext();

        // Only allow authenticated admins
        return httpContext.User.Identity?.IsAuthenticated == true
            && httpContext.User.IsInRole("Admin");
    }
}
```

### Job Types

#### 1. Fire-and-Forget

```csharp
// Executes once, immediately (in background)
var jobId = BackgroundJob.Enqueue<IEmailService>(
    x => x.SendWelcomeEmailAsync("user@example.com", CancellationToken.None));

// Or via lambda
BackgroundJob.Enqueue(() => Console.WriteLine("Hello from background!"));
```

#### 2. Delayed Jobs

```csharp
// Execute once, after a delay
var jobId = BackgroundJob.Schedule<IEmailService>(
    x => x.SendReminderEmailAsync("user@example.com", CancellationToken.None),
    delay: TimeSpan.FromHours(24));

// Or with specific time
BackgroundJob.Schedule(
    () => CleanupTempFiles(),
    enqueueAt: DateTimeOffset.UtcNow.AddDays(1));
```

#### 3. Recurring Jobs

```csharp
// Run on a cron schedule
RecurringJob.AddOrUpdate<IReportService>(
    recurringJobId: "daily-sales-report",
    methodCall: x => x.GenerateDailySalesReportAsync(CancellationToken.None),
    cronExpression: Cron.Daily(8, 0),      // Every day at 08:00
    options: new RecurringJobOptions
    {
        TimeZone = TimeZoneInfo.FindSystemTimeZoneById("India Standard Time")
    });

// Using cron expressions
RecurringJob.AddOrUpdate<ICleanupService>(
    "cleanup-logs",
    x => x.CleanupOldLogsAsync(CancellationToken.None),
    "0 2 * * 0");  // Every Sunday at 2:00 AM

// Remove recurring job
RecurringJob.RemoveIfExists("daily-sales-report");

// Trigger immediately (outside schedule)
RecurringJob.TriggerJob("daily-sales-report");
```

#### 4. Continuations

```csharp
// Job B runs after Job A completes
var jobAId = BackgroundJob.Enqueue<IOrderService>(
    x => x.ProcessOrderAsync(orderId, CancellationToken.None));

BackgroundJob.ContinueJobWith<IEmailService>(
    jobAId,
    x => x.SendOrderConfirmationAsync(orderId, CancellationToken.None));
```

#### 5. Batch Jobs (Hangfire Pro)

```csharp
// All jobs in batch run in parallel, callback when all complete
var batchId = BatchJob.StartNew(batch =>
{
    batch.Enqueue<IReportService>(x => x.GenerateRegionReport("North"));
    batch.Enqueue<IReportService>(x => x.GenerateRegionReport("South"));
    batch.Enqueue<IReportService>(x => x.GenerateRegionReport("East"));
    batch.Enqueue<IReportService>(x => x.GenerateRegionReport("West"));
});

BatchJob.ContinueBatchWith(batchId, batch =>
{
    batch.Enqueue<IReportService>(x => x.MergeAllRegionReports());
});
```

### Job with Queues (Priority)

```csharp
// Enqueue to a specific queue
BackgroundJob.Enqueue<IEmailService>(
    x => x.SendCriticalAlertAsync("admin@company.com"),
    new EnqueuedState("critical")); // Goes to "critical" queue first

// Or use attribute on method
[Queue("critical")]
public async Task SendCriticalAlertAsync(string email)
{
    // ...
}
```

### Configuring Retry Policy

```csharp
// Global retry policy
GlobalJobFilters.Filters.Add(new AutomaticRetryAttribute
{
    Attempts = 3,
    DelaysInSeconds = new[] { 60, 300, 3600 } // 1min, 5min, 1hr
});

// Per-job retry policy (attribute on method)
[AutomaticRetry(Attempts = 5, OnAttemptsExceeded = AttemptsExceededAction.Delete)]
public async Task SendEmailAsync(string to, string subject)
{
    // ...
}

// Disable retries for a specific job
[AutomaticRetry(Attempts = 0)]
public async Task OneTimeMigrationAsync()
{
    // ...
}
```

### Job Interface (Clean Architecture)

```csharp
// ✅ Best Practice: Define jobs as services, not static methods
public interface IEmailService
{
    Task SendWelcomeEmailAsync(string email, CancellationToken cancellationToken);
    Task SendOrderConfirmationAsync(Guid orderId, CancellationToken cancellationToken);
}

public class EmailService : IEmailService
{
    private readonly ILogger<EmailService> _logger;
    private readonly AppDbContext _dbContext;

    public EmailService(ILogger<EmailService> logger, AppDbContext dbContext)
    {
        _logger = logger;
        _dbContext = dbContext;
    }

    public async Task SendWelcomeEmailAsync(string email, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Sending welcome email to {Email}", email);
        // Send email logic...
    }
}

// Register with DI (Hangfire resolves via DI automatically)
builder.Services.AddScoped<IEmailService, EmailService>();

// Enqueue using interface
BackgroundJob.Enqueue<IEmailService>(
    x => x.SendWelcomeEmailAsync("user@example.com", CancellationToken.None));
```

---

## Cron Expression Quick Reference

```
Cron Format: * * * * *
             │ │ │ │ └── Day of week (0-7, 0=Sunday)
             │ │ │ └──── Month (1-12)
             │ │ └────── Day of month (1-31)
             │ └──────── Hour (0-23)
             └────────── Minute (0-59)
```

### Common Cron Expressions

```bash
# Every minute
* * * * *

# Every 5 minutes
*/5 * * * *

# Every hour (at minute 0)
0 * * * *

# Every day at midnight
0 0 * * *

# Every day at 8:00 AM
0 8 * * *

# Every weekday (Mon-Fri) at 9:00 AM
0 9 * * 1-5

# Every Monday at 7:00 AM
0 7 * * 1

# First day of every month at midnight
0 0 1 * *

# Every Sunday at 2:00 AM
0 2 * * 0

# Twice a day: midnight and noon
0 0,12 * * *

# Every 15 minutes during business hours (9-5, Mon-Fri)
*/15 9-17 * * 1-5
```

### Hangfire Cron Helpers

```csharp
// ✅ Use Hangfire's Cron helpers for readability
Cron.Minutely()          // Every minute
Cron.Hourly()            // Every hour
Cron.Hourly(30)          // Every hour at :30
Cron.Daily()             // Every day at midnight
Cron.Daily(8)            // Every day at 8:00 AM
Cron.Daily(8, 30)        // Every day at 8:30 AM
Cron.Weekly()            // Every Monday at midnight
Cron.Weekly(DayOfWeek.Friday, 17, 0)  // Every Friday at 5:00 PM
Cron.Monthly()           // 1st of month at midnight
Cron.Monthly(15, 9, 0)   // 15th of month at 9:00 AM
Cron.Yearly()            // Jan 1st at midnight
Cron.Never()             // Never (disable without removing)
```

---

## IHostedService vs BackgroundService vs Hangfire

| Feature | IHostedService | BackgroundService | Hangfire |
|---|---|---|---|
| **Complexity** | Low | Low | Medium |
| **Persistence** | ❌ In-memory | ❌ In-memory | ✅ Database |
| **Scheduling** | Manual | Manual | ✅ Cron built-in |
| **Dashboard** | ❌ None | ❌ None | ✅ Built-in |
| **Retries** | Manual | Manual | ✅ Automatic |
| **Survives restart** | ❌ No | ❌ No | ✅ Yes |
| **DI support** | ✅ Full | ✅ Full | ✅ Full |
| **Async support** | Manual | ✅ Native | ✅ Native |
| **Best for** | Startup tasks, simple timers | Queue processing, polling | Scheduled/reliable jobs |

### Decision Guide

```
Need a job to run once at startup?
    → IHostedService.StartAsync()

Need to poll a queue or process data in a loop?
    → BackgroundService

Need cron scheduling + persistence + retries + dashboard?
    → Hangfire

Building a standalone background-only app?
    → Worker Service (any of the above inside it)
```

---

## Best Practices

### 1. Always Use CancellationToken

```csharp
// ✅ Pass token to every async operation
await dbContext.SaveChangesAsync(cancellationToken);
await httpClient.GetAsync(url, cancellationToken);
await Task.Delay(1000, cancellationToken);
```

### 2. Never Block in BackgroundService

```csharp
// ❌ Bad - Blocks the thread pool
protected override Task ExecuteAsync(CancellationToken stoppingToken)
{
    Thread.Sleep(1000);
    return Task.CompletedTask;
}

// ✅ Good - Async all the way
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    await Task.Delay(1000, stoppingToken);
}
```

### 3. Handle Exceptions — Don't Let Services Die Silently

```csharp
// ✅ Wrap work in try/catch with logging
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        try
        {
            await DoWorkAsync(stoppingToken);
        }
        catch (OperationCanceledException)
        {
            break; // Shutting down, exit cleanly
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unexpected error in background service");
            // ✅ Wait before retrying to avoid tight error loops
            await Task.Delay(TimeSpan.FromSeconds(30), stoppingToken);
        }
    }
}
```

### 4. Use IServiceScopeFactory for Scoped Dependencies

```csharp
// ✅ Create a scope per unit of work
using var scope = _scopeFactory.CreateScope();
var dbContext = scope.ServiceProvider.GetRequiredService<AppDbContext>();
```

### 5. Log Start/Stop and Each Execution

```csharp
_logger.LogInformation("{ServiceName} starting at {Time}", nameof(MyService), DateTimeOffset.UtcNow);
// ...work...
_logger.LogInformation("{ServiceName} completed in {Elapsed}ms", nameof(MyService), stopwatch.ElapsedMilliseconds);
```

### 6. Set Appropriate Worker Count for Hangfire

```csharp
// ✅ Balance between throughput and resource usage
builder.Services.AddHangfireServer(options =>
{
    // CPU-bound work: match processor count
    options.WorkerCount = Environment.ProcessorCount;

    // I/O-bound work (DB, HTTP): can go higher
    options.WorkerCount = Environment.ProcessorCount * 5;
});
```

---

## Quick Reference

```csharp
// ─── IHostedService ─────────────────────────────────────────────────
public class MyService : IHostedService
{
    public Task StartAsync(CancellationToken ct) { /* start */ return Task.CompletedTask; }
    public Task StopAsync(CancellationToken ct) { /* cleanup */ return Task.CompletedTask; }
}

// ─── BackgroundService ───────────────────────────────────────────────
public class MyService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await DoWorkAsync(stoppingToken);
            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }
}

// ─── PeriodicTimer (.NET 6+) ─────────────────────────────────────────
using var timer = new PeriodicTimer(TimeSpan.FromHours(1));
while (await timer.WaitForNextTickAsync(stoppingToken))
    await DoWorkAsync(stoppingToken);

// ─── Hangfire Jobs ───────────────────────────────────────────────────
BackgroundJob.Enqueue<IService>(x => x.DoWork());                      // Fire-and-forget
BackgroundJob.Schedule<IService>(x => x.DoWork(), TimeSpan.FromHours(1)); // Delayed
RecurringJob.AddOrUpdate<IService>("job-id", x => x.DoWork(), Cron.Daily(8)); // Recurring
BackgroundJob.ContinueJobWith<IService>(parentId, x => x.DoWork());   // Continuation

// ─── Scoped Services ─────────────────────────────────────────────────
using var scope = _scopeFactory.CreateScope();
var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();

// ─── Register ────────────────────────────────────────────────────────
builder.Services.AddHostedService<MyService>();
```

---

**Guide Complete!** This comprehensive guide covers `IHostedService`, `BackgroundService`, Worker Services, long-running tasks with graceful cancellation, Hangfire job scheduling, cron expressions, and all the patterns you need for reliable background processing in .NET! ⚙️
