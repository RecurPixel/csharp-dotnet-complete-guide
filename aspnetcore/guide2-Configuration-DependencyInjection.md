# ASP.NET Core Configuration & Dependency Injection - Complete Guide
## Practical Guide + Technical Reference

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. Configuration Basics
2. Configuration Sources & Priority
3. Dependency Injection Fundamentals
4. Service Lifetimes (Transient, Scoped, Singleton)
5. Registering Services (3 Methods)
6. Injecting and Using Services
7. Common DI Patterns
8. Troubleshooting Common Issues
9. Best Practices

### Part 2: Technical Reference (Deep Dive)
10. Important Interfaces & Classes Reference
11. Configuration Deep-Dive
12. Advanced DI Topics
13. Performance Considerations

---

# PART 1: PRACTICAL GUIDE

---

## 1. Configuration Basics

**Simple Definition:** Configuration is how you store settings outside your code (database connections, API keys, feature flags).

**Think of it like:** A settings panel for your application that can change without recompiling code.

### Why Configuration?

```
❌ WRONG - Hardcoded
var connectionString = "Server=localhost;Database=MyDb;...";

✅ RIGHT - Configured
var connectionString = configuration["ConnectionStrings:Default"];
```

**Key Benefits:**
- 🔧 Change settings without recompiling
- 🌍 Different settings per environment (Dev, Staging, Prod)
- 🔐 Keep secrets secure (API keys, passwords)
- 📊 Easy to manage and audit

---

## 2. Configuration Sources & Priority

### Configuration Sources (Loaded in Order)

ASP.NET Core loads configuration from multiple sources. **Later sources override earlier ones.**

```
1. appsettings.json                    (Base settings)
   ↓
2. appsettings.{Environment}.json      (Environment-specific)
   ↓
3. User Secrets                        (Development only)
   ↓
4. Environment Variables               (Server/Container)
   ↓
5. Command-line Arguments              (Runtime)
```

**Example: Same Key Different Sources**

```json
// appsettings.json
{
  "ApiUrl": "https://api.example.com"
}

// appsettings.Development.json
{
  "ApiUrl": "https://localhost:5001"  // ✅ This wins in Development!
}
```

**Environment Variable (highest priority):**
```bash
# In terminal
export ApiUrl="https://staging-api.example.com"  # ✅ This wins over everything!
```

---

### Reading Configuration - 3 Methods

### Method 1: IConfiguration Directly (Quick & Simple)

**When to use:**
- ✅ Quick prototyping
- ✅ Simple string values
- ❌ Complex objects
- ❌ Strong typing

**Step 1: Inject IConfiguration**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/config", (IConfiguration config) =>
{
    var apiUrl = config["ApiUrl"];
    var dbConnection = config["ConnectionStrings:Default"];
    
    return new { apiUrl, dbConnection };
});

app.Run();
```

**Complete Example:**

```csharp
// appsettings.json
{
  "AppName": "MyApp",
  "ApiUrl": "https://api.example.com",
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MyDb"
  }
}

// Program.cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/settings", (IConfiguration config) =>
{
    return new
    {
        Name = config["AppName"],
        ApiUrl = config["ApiUrl"],
        DbConnection = config["ConnectionStrings:Default"],
        
        // GetSection alternative
        DbConnection2 = config.GetSection("ConnectionStrings")["Default"]
    };
});

app.Run();
```

---

### Method 2: GetSection & Bind (Medium Complexity)

**When to use:**
- ✅ Nested configurations
- ✅ Arrays/lists
- ✅ Some type safety
- ⚠️ Manual binding

**Step 1: Create Settings Class**

```csharp
public class EmailSettings
{
    public string SmtpServer { get; set; }
    public int Port { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
    public bool EnableSsl { get; set; }
}
```

**Step 2: Configure appsettings.json**

```json
{
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "Port": 587,
    "Username": "myapp@example.com",
    "Password": "secret",
    "EnableSsl": true
  }
}
```

**Step 3: Bind Configuration**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Bind to object
var emailSettings = new EmailSettings();
builder.Configuration.GetSection("EmailSettings").Bind(emailSettings);

// Use it
Console.WriteLine($"SMTP: {emailSettings.SmtpServer}:{emailSettings.Port}");

var app = builder.Build();
app.Run();
```

---

### Method 3: IOptions Pattern (Production) ⭐ RECOMMENDED

**When to use:**
- ✅ Production applications
- ✅ Strong typing
- ✅ Validation support
- ✅ Multiple consumers
- ✅ Dependency injection

**Step 1: Create Options Class**

```csharp
public class EmailSettings
{
    public const string SectionName = "EmailSettings"; // Best practice
    
    public string SmtpServer { get; set; }
    public int Port { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
    public bool EnableSsl { get; set; }
}
```

**Step 2: Configure appsettings.json**

```json
{
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "Port": 587,
    "Username": "myapp@example.com",
    "Password": "secret",
    "EnableSsl": true
  }
}
```

**Step 3: Register in DI**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register options
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection(EmailSettings.SectionName));

var app = builder.Build();
```

**Step 4: Inject and Use**

```csharp
public class EmailService
{
    private readonly EmailSettings _settings;
    
    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value; // Get the actual value
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        // Use _settings.SmtpServer, _settings.Port, etc.
        using var client = new SmtpClient(_settings.SmtpServer, _settings.Port)
        {
            EnableSsl = _settings.EnableSsl,
            Credentials = new NetworkCredential(_settings.Username, _settings.Password)
        };
        
        await client.SendMailAsync("from@example.com", to, subject, body);
    }
}
```

---

### Configuration Methods Comparison

| Method | Complexity | Type Safety | DI Support | Reload | Production |
|--------|-----------|-------------|-----------|--------|-----------|
| `config["Key"]` | Simple | ❌ None | ✅ Yes | N/A | ⚠️ Testing only |
| `GetSection().Bind()` | Medium | ⚠️ Partial | ⚠️ Manual | ❌ No | ⚠️ Maybe |
| `IOptions<T>` | Medium | ✅ Full | ✅ Yes | ❌ No | ✅ Yes |
| `IOptionsSnapshot<T>` | Medium | ✅ Full | ✅ Yes | ✅ Per request | ✅ Yes |
| `IOptionsMonitor<T>` | Complex | ✅ Full | ✅ Yes | ✅ Live | ✅ Advanced |

---

## 3. Dependency Injection Fundamentals

**Simple Definition:** A design pattern where objects don't create their dependencies - they receive them instead.

**Think of it like:** Instead of building your own tools (hard), someone hands you the tools you need (easy).

### Without DI (Tightly Coupled ❌)

```csharp
public class OrderService
{
    private readonly EmailService _emailService;
    private readonly PaymentService _paymentService;
    
    public OrderService()
    {
        _emailService = new EmailService(); // ❌ Creates dependencies
        _paymentService = new PaymentService(); // ❌ Hard to test
    }
}
```

**Problems:**
- ❌ Hard to test (can't mock dependencies)
- ❌ Tight coupling (changes ripple through code)
- ❌ Hard to change implementations
- ❌ Can't use interfaces

---

### With DI (Loosely Coupled ✅)

```csharp
public class OrderService
{
    private readonly IEmailService _emailService;
    private readonly IPaymentService _paymentService;
    
    public OrderService(
        IEmailService emailService,      // ✅ Injected
        IPaymentService paymentService)  // ✅ Injected
    {
        _emailService = emailService;
        _paymentService = paymentService;
    }
}
```

**Benefits:**
- ✅ Easy to test (inject mocks)
- ✅ Loose coupling
- ✅ Easy to swap implementations
- ✅ Interface-based design

---

## 4. Service Lifetimes (Transient, Scoped, Singleton)

### Visual Comparison

```
┌──────────────────────────────────────────────────────────┐
│ APPLICATION LIFETIME                                      │
│                                                           │
│  Singleton ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  (One instance for entire app)                           │
│                                                           │
│  ┌─────────────────┐  ┌─────────────────┐               │
│  │  REQUEST 1      │  │  REQUEST 2      │               │
│  │                 │  │                 │               │
│  │  Scoped ────────┤  │  Scoped ────────┤               │
│  │  (One per req)  │  │  (New instance) │               │
│  │                 │  │                 │               │
│  │  ┌──┐  ┌──┐    │  │  ┌──┐  ┌──┐    │               │
│  │  │T │  │T │    │  │  │T │  │T │    │               │
│  │  └──┘  └──┘    │  │  └──┘  └──┘    │               │
│  │  Transient      │  │  Transient      │               │
│  │  (New each time)│  │  (New each time)│               │
│  └─────────────────┘  └─────────────────┘               │
└──────────────────────────────────────────────────────────┘
```

---

### Transient Lifetime

**Definition:** New instance every time requested.

**When to use:**
- ✅ Lightweight, stateless services
- ✅ No shared state
- ✅ Short-lived operations
- ❌ Database contexts (use Scoped)
- ❌ Expensive to create (use Singleton)

**Example:**

```csharp
public interface IEmailService
{
    Task SendEmailAsync(string to, string subject, string body);
}

public class EmailService : IEmailService
{
    private readonly ILogger<EmailService> _logger;
    
    public EmailService(ILogger<EmailService> logger)
    {
        _logger = logger;
        _logger.LogInformation("EmailService created"); // ⚠️ Called EVERY time
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        // Send email logic
    }
}

// Registration
builder.Services.AddTransient<IEmailService, EmailService>();

// Usage in controller
public class OrderController : ControllerBase
{
    private readonly IEmailService _email1;
    private readonly IEmailService _email2;
    
    public OrderController(IEmailService email1, IEmailService email2)
    {
        _email1 = email1; // New instance
        _email2 = email2; // Different new instance!
    }
}
```

---

### Scoped Lifetime ⭐ MOST COMMON

**Definition:** One instance per HTTP request (or scope).

**When to use:**
- ✅ Database contexts (Entity Framework)
- ✅ Unit of Work pattern
- ✅ Per-request state
- ✅ Most business logic services

**Example:**

```csharp
public interface IUserService
{
    Task<User> GetUserAsync(int id);
}

public class UserService : IUserService
{
    private readonly ApplicationDbContext _context; // Scoped!
    private readonly ILogger<UserService> _logger;
    
    public UserService(
        ApplicationDbContext context,
        ILogger<UserService> logger)
    {
        _context = context;
        _logger = logger;
        _logger.LogInformation("UserService created"); // ⚠️ Once per request
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }
}

// Registration
builder.Services.AddScoped<IUserService, UserService>();

// Usage
public class OrderController : ControllerBase
{
    private readonly IUserService _user1;
    private readonly IUserService _user2;
    
    public OrderController(IUserService user1, IUserService user2)
    {
        _user1 = user1; // Same instance within this request
        _user2 = user2; // Same instance! (points to same object)
    }
}
```

---

### Singleton Lifetime

**Definition:** One instance for entire application lifetime.

**When to use:**
- ✅ Configuration/Settings
- ✅ Caching services
- ✅ Logging
- ✅ Expensive to create objects
- ❌ Database contexts (NOT thread-safe)
- ⚠️ Must be thread-safe!

**Example:**

```csharp
public interface ICacheService
{
    void Set(string key, object value);
    object Get(string key);
}

public class CacheService : ICacheService
{
    private readonly ConcurrentDictionary<string, object> _cache;
    private readonly ILogger<CacheService> _logger;
    
    public CacheService(ILogger<CacheService> logger)
    {
        _cache = new ConcurrentDictionary<string, object>();
        _logger = logger;
        _logger.LogInformation("CacheService created"); // ⚠️ Only ONCE for entire app
    }
    
    public void Set(string key, object value)
    {
        _cache[key] = value; // ⚠️ Must be thread-safe!
    }
    
    public object Get(string key)
    {
        return _cache.TryGetValue(key, out var value) ? value : null;
    }
}

// Registration
builder.Services.AddSingleton<ICacheService, CacheService>();
```

---

### Lifetime Decision Tree

```
Need to share state across requests?
│
├─ YES → Singleton (⚠️ Must be thread-safe!)
│
└─ NO
   │
   └─ Need to share state within a request?
      │
      ├─ YES → Scoped (e.g., DbContext)
      │
      └─ NO → Transient (stateless)
```

---

### Service Lifetime Comparison Table

| Lifetime | Instance Created | Shared Across | Use For | Thread Safety Required |
|----------|------------------|--------------|---------|----------------------|
| **Transient** | Every injection | Nothing | Stateless services | No |
| **Scoped** | Once per request | Same request | DbContext, per-request state | No |
| **Singleton** | Once for app | All requests | Cache, config, logging | ✅ YES |

---

## 5. Registering Services (3 Methods)

### Method 1: Built-in DI (Direct Registration)

**When to use:**
- ✅ Simple projects
- ✅ Few services
- ✅ Quick prototypes
- ❌ Large projects (gets messy)

**Step 1: Register Services in Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services directly
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddSingleton<ICacheService, CacheService>();

// DbContext (scoped by default)
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();
app.Run();
```

**Complete Example:**

```csharp
// Services
public interface IEmailService
{
    Task SendEmailAsync(string to, string body);
}

public class EmailService : IEmailService
{
    public async Task SendEmailAsync(string to, string body)
    {
        // Send email logic
        await Task.Delay(100); // Simulate sending
    }
}

// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Register
builder.Services.AddTransient<IEmailService, EmailService>();

var app = builder.Build();

app.MapGet("/send-email", async (IEmailService emailService) =>
{
    await emailService.SendEmailAsync("user@example.com", "Hello!");
    return "Email sent!";
});

app.Run();
```

---

### Method 2: Extension Methods (Production) ⭐ RECOMMENDED

**When to use:**
- ✅ Clean, organized code
- ✅ Reusable service groups
- ✅ Production applications
- ✅ Library/package development

**Step 1: Create Extension Method**

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddApplicationServices(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        // Business services
        services.AddScoped<IUserService, UserService>();
        services.AddScoped<IOrderService, OrderService>();
        services.AddScoped<IProductService, ProductService>();
        
        // Infrastructure services
        services.AddTransient<IEmailService, EmailService>();
        services.AddSingleton<ICacheService, CacheService>();
        
        // Database
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(configuration.GetConnectionString("Default")));
        
        // Options
        services.Configure<EmailSettings>(
            configuration.GetSection(EmailSettings.SectionName));
        
        return services;
    }
}
```

**Step 2: Use in Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Clean! One line instead of 10+
builder.Services.AddApplicationServices(builder.Configuration);

var app = builder.Build();
app.Run();
```

**Multiple Extension Methods (Best Practice):**

```csharp
// Infrastructure/Extensions/DatabaseExtensions.cs
public static class DatabaseExtensions
{
    public static IServiceCollection AddDatabase(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(configuration.GetConnectionString("Default")));
        
        return services;
    }
}

// Infrastructure/Extensions/EmailExtensions.cs
public static class EmailExtensions
{
    public static IServiceCollection AddEmail(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.Configure<EmailSettings>(
            configuration.GetSection(EmailSettings.SectionName));
        
        services.AddTransient<IEmailService, EmailService>();
        
        return services;
    }
}

// Program.cs - Beautiful!
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddDatabase(builder.Configuration)
    .AddEmail(builder.Configuration)
    .AddBusinessServices();

var app = builder.Build();
```

---

### Method 3: Third-Party Containers (Advanced)

**When to use:**
- ✅ Need advanced features (property injection, decorators, interceptors)
- ✅ Legacy code integration
- ⚠️ Adds complexity
- ⚠️ Another dependency

**Popular Containers:**
- Autofac (most popular)
- StructureMap
- Castle Windsor
- Ninject

**Example with Autofac:**

**Step 1: Install Package**
```bash
dotnet add package Autofac.Extensions.DependencyInjection
```

**Step 2: Configure Autofac**

```csharp
using Autofac;
using Autofac.Extensions.DependencyInjection;

var builder = WebApplication.CreateBuilder(args);

// Use Autofac
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());

// Register services with built-in DI
builder.Services.AddControllers();

// Configure Autofac container
builder.Host.ConfigureContainer<ContainerBuilder>(containerBuilder =>
{
    // Property injection
    containerBuilder.RegisterType<OrderService>()
        .As<IOrderService>()
        .PropertiesAutowired();
    
    // Assembly scanning
    containerBuilder.RegisterAssemblyTypes(typeof(Program).Assembly)
        .Where(t => t.Name.EndsWith("Service"))
        .AsImplementedInterfaces()
        .InstancePerLifetimeScope(); // Scoped
});

var app = builder.Build();
app.Run();
```

---

## 6. Injecting and Using Services

### Constructor Injection (Primary Method) ⭐

**When to use:** 99% of the time

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly IEmailService _emailService;
    private readonly ILogger<OrderController> _logger;
    
    // Dependencies injected via constructor
    public OrderController(
        IOrderService orderService,
        IEmailService emailService,
        ILogger<OrderController> logger)
    {
        _orderService = orderService;
        _emailService = emailService;
        _logger = logger;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
    {
        var order = await _orderService.CreateAsync(request);
        await _emailService.SendOrderConfirmationAsync(order);
        _logger.LogInformation($"Order {order.Id} created");
        
        return Ok(order);
    }
}
```

---

### FromServices Attribute (Action Injection)

**When to use:** Only needed in one action

```csharp
public class OrderController : ControllerBase
{
    // No constructor needed!
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(
        CreateOrderRequest request,
        [FromServices] IOrderService orderService,     // ✅ Injected per action
        [FromServices] IEmailService emailService)
    {
        var order = await orderService.CreateAsync(request);
        await emailService.SendOrderConfirmationAsync(order);
        
        return Ok(order);
    }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetOrder(
        int id,
        [FromServices] IOrderService orderService)  // ✅ Different instance per action
    {
        var order = await orderService.GetByIdAsync(id);
        return Ok(order);
    }
}
```

---

### Service Locator Pattern (AVOID! ❌)

**Anti-pattern - Don't use unless absolutely necessary**

```csharp
public class OrderService
{
    private readonly IServiceProvider _serviceProvider;
    
    public OrderService(IServiceProvider serviceProvider) // ❌ Don't inject IServiceProvider
    {
        _serviceProvider = serviceProvider;
    }
    
    public async Task ProcessOrder()
    {
        // ❌ Anti-pattern - hides dependencies
        var emailService = _serviceProvider.GetRequiredService<IEmailService>();
        
        // Use emailService...
    }
}
```

**Why it's bad:**
- ❌ Hides dependencies (hard to see what's needed)
- ❌ Runtime errors instead of compile-time
- ❌ Harder to test
- ❌ Breaks dependency injection principles

---

### Minimal API Injection

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IUserService, UserService>();

var app = builder.Build();

// Inject into route handlers
app.MapGet("/orders/{id}", async (
    int id,
    IOrderService orderService,      // ✅ Automatically injected
    IUserService userService) =>
{
    var order = await orderService.GetByIdAsync(id);
    var user = await userService.GetByIdAsync(order.UserId);
    
    return new { order, user };
});

app.Run();
```

---

## 7. Common DI Patterns

### Repository Pattern

**Purpose:** Abstract data access logic

```csharp
// Interface
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

// Implementation
public class Repository<T> : IRepository<T> where T : class
{
    private readonly ApplicationDbContext _context;
    private readonly DbSet<T> _dbSet;
    
    public Repository(ApplicationDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
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

// Specific repository
public interface IUserRepository : IRepository<User>
{
    Task<User> GetByEmailAsync(string email);
}

public class UserRepository : Repository<User>, IUserRepository
{
    private readonly ApplicationDbContext _context;
    
    public UserRepository(ApplicationDbContext context) : base(context)
    {
        _context = context;
    }
    
    public async Task<User> GetByEmailAsync(string email)
    {
        return await _context.Users
            .FirstOrDefaultAsync(u => u.Email == email);
    }
}

// Registration
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
builder.Services.AddScoped<IUserRepository, UserRepository>();

// Usage
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public async Task<User> GetUserByEmailAsync(string email)
    {
        return await _userRepository.GetByEmailAsync(email);
    }
}
```

---

### Unit of Work Pattern

**Purpose:** Coordinate multiple repository operations in a transaction

```csharp
public interface IUnitOfWork : IDisposable
{
    IUserRepository Users { get; }
    IOrderRepository Orders { get; }
    IProductRepository Products { get; }
    
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    private IDbContextTransaction _transaction;
    
    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
        Users = new UserRepository(_context);
        Orders = new OrderRepository(_context);
        Products = new ProductRepository(_context);
    }
    
    public IUserRepository Users { get; }
    public IOrderRepository Orders { get; }
    public IProductRepository Products { get; }
    
    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }
    
    public async Task BeginTransactionAsync()
    {
        _transaction = await _context.Database.BeginTransactionAsync();
    }
    
    public async Task CommitTransactionAsync()
    {
        await _transaction.CommitAsync();
    }
    
    public async Task RollbackTransactionAsync()
    {
        await _transaction.RollbackAsync();
    }
    
    public void Dispose()
    {
        _transaction?.Dispose();
        _context.Dispose();
    }
}

// Registration
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

// Usage
public class OrderService
{
    private readonly IUnitOfWork _unitOfWork;
    
    public OrderService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        await _unitOfWork.BeginTransactionAsync();
        
        try
        {
            // Create order
            var order = new Order { UserId = request.UserId, Total = request.Total };
            await _unitOfWork.Orders.AddAsync(order);
            
            // Update product inventory
            var product = await _unitOfWork.Products.GetByIdAsync(request.ProductId);
            product.Stock -= request.Quantity;
            await _unitOfWork.Products.UpdateAsync(product);
            
            // Save all changes in one transaction
            await _unitOfWork.SaveChangesAsync();
            await _unitOfWork.CommitTransactionAsync();
            
            return order;
        }
        catch
        {
            await _unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }
}
```

---

### Factory Pattern

**Purpose:** Create objects with complex initialization

```csharp
public interface IEmailServiceFactory
{
    IEmailService Create(EmailProvider provider);
}

public class EmailServiceFactory : IEmailServiceFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public EmailServiceFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public IEmailService Create(EmailProvider provider)
    {
        return provider switch
        {
            EmailProvider.SendGrid => _serviceProvider.GetRequiredService<SendGridEmailService>(),
            EmailProvider.Smtp => _serviceProvider.GetRequiredService<SmtpEmailService>(),
            EmailProvider.AmazonSes => _serviceProvider.GetRequiredService<AmazonSesEmailService>(),
            _ => throw new ArgumentException($"Unknown provider: {provider}")
        };
    }
}

// Registration
builder.Services.AddTransient<SendGridEmailService>();
builder.Services.AddTransient<SmtpEmailService>();
builder.Services.AddTransient<AmazonSesEmailService>();
builder.Services.AddSingleton<IEmailServiceFactory, EmailServiceFactory>();

// Usage
public class NotificationService
{
    private readonly IEmailServiceFactory _emailFactory;
    
    public NotificationService(IEmailServiceFactory emailFactory)
    {
        _emailFactory = emailFactory;
    }
    
    public async Task SendNotificationAsync(User user, string message)
    {
        var emailService = _emailFactory.Create(user.PreferredEmailProvider);
        await emailService.SendEmailAsync(user.Email, "Notification", message);
    }
}
```

---

## 8. Troubleshooting Common Issues

### Problem 1: Service Not Registered

**Error:**
```
InvalidOperationException: Unable to resolve service for type 'IUserService'
```

**Solution:**
```csharp
// Make sure service is registered
builder.Services.AddScoped<IUserService, UserService>();
```

---

### Problem 2: Captive Dependency

**Error:** Singleton depends on Scoped service

```csharp
// ❌ WRONG - Singleton capturing Scoped dependency
public class CacheService  // Registered as Singleton
{
    private readonly ApplicationDbContext _context;  // Scoped!
    
    public CacheService(ApplicationDbContext context)
    {
        _context = context;  // ❌ DbContext lifetime shorter than CacheService!
    }
}
```

**Solution:**
```csharp
// ✅ RIGHT - Use IServiceProvider to get scoped service when needed
public class CacheService
{
    private readonly IServiceProvider _serviceProvider;
    
    public CacheService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        using var scope = _serviceProvider.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
        return await context.Users.FindAsync(id);
    }
}
```

---

### Problem 3: Circular Dependencies

**Error:**
```
System.InvalidOperationException: A circular dependency was detected
```

**Example:**
```csharp
// ❌ Circular dependency
public class ServiceA
{
    private readonly ServiceB _serviceB;
    public ServiceA(ServiceB serviceB) => _serviceB = serviceB;
}

public class ServiceB
{
    private readonly ServiceA _serviceA;
    public ServiceB(ServiceA serviceA) => _serviceA = serviceA;  // ❌ Circular!
}
```

**Solution 1: Refactor design**
```csharp
// Extract common logic into third service
public class SharedService
{
    public void CommonLogic() { }
}

public class ServiceA
{
    private readonly SharedService _shared;
    public ServiceA(SharedService shared) => _shared = shared;
}

public class ServiceB
{
    private readonly SharedService _shared;
    public ServiceB(SharedService shared) => _shared = shared;
}
```

**Solution 2: Use Lazy<T>**
```csharp
public class ServiceA
{
    private readonly Lazy<ServiceB> _serviceB;
    
    public ServiceA(Lazy<ServiceB> serviceB)
    {
        _serviceB = serviceB;
    }
    
    public void DoWork()
    {
        _serviceB.Value.DoSomething();  // Resolved only when accessed
    }
}
```

---

### Problem 4: Missing Configuration Section

**Error:**
```
System.ArgumentNullException: Value cannot be null. (Parameter 'source')
```

**Solution:**
```csharp
// ✅ Check if section exists
var emailSection = builder.Configuration.GetSection("EmailSettings");
if (emailSection.Exists())
{
    builder.Services.Configure<EmailSettings>(emailSection);
}
else
{
    throw new InvalidOperationException("EmailSettings section not found in appsettings.json");
}
```

---

## 9. Best Practices

### ✅ DO's

1. **Use constructor injection**
   ```csharp
   public class UserService
   {
       private readonly IUserRepository _repository;
       
       public UserService(IUserRepository repository)
       {
           _repository = repository;
       }
   }
   ```

2. **Program to interfaces**
   ```csharp
   builder.Services.AddScoped<IUserService, UserService>(); // ✅ Interface
   ```

3. **Use appropriate lifetime**
   - Transient: Stateless services
   - Scoped: DbContext, per-request state
   - Singleton: Cache, configuration

4. **Use extension methods for registration**
   ```csharp
   builder.Services.AddApplicationServices(builder.Configuration);
   ```

5. **Use IOptions for configuration**
   ```csharp
   builder.Services.Configure<EmailSettings>(
       builder.Configuration.GetSection("EmailSettings"));
   ```

---

### ❌ DON'Ts

1. **Don't inject IServiceProvider**
   ```csharp
   // ❌ Don't do this
   public UserService(IServiceProvider provider)
   ```

2. **Don't create dependencies manually**
   ```csharp
   // ❌ Don't do this
   public UserService()
   {
       _repository = new UserRepository();
   }
   ```

3. **Don't inject Scoped into Singleton**
   ```csharp
   // ❌ Captive dependency
   builder.Services.AddSingleton<CacheService>();  // Singleton
   // But CacheService depends on DbContext (Scoped) ❌
   ```

4. **Don't hardcode configuration**
   ```csharp
   // ❌ Don't do this
   var apiKey = "hardcoded-key-12345";
   
   // ✅ Do this
   var apiKey = configuration["ApiKey"];
   ```

5. **Don't forget to dispose**
   ```csharp
   // ✅ DI container handles disposal automatically
   // But if you manually create:
   using var context = new ApplicationDbContext();
   ```

---

### Best Practices Checklist

**Configuration:**
- [ ] Use appsettings.json for configuration
- [ ] Use User Secrets for local development
- [ ] Use Environment Variables for production
- [ ] Use IOptions<T> for strongly-typed configuration
- [ ] Validate configuration on startup

**Dependency Injection:**
- [ ] Register services with appropriate lifetime
- [ ] Use constructor injection (not service locator)
- [ ] Program to interfaces, not implementations
- [ ] Use extension methods for clean registration
- [ ] Avoid captive dependencies
- [ ] Handle circular dependencies properly

**Code Organization:**
- [ ] Group service registrations by feature
- [ ] Create extension methods for related services
- [ ] Keep Program.cs clean and readable
- [ ] Document complex dependencies

---

# PART 2: TECHNICAL REFERENCE

---

## 10. Important Interfaces & Classes Reference

### IConfiguration Interface

**Namespace:** `Microsoft.Extensions.Configuration`

**Purpose:** Read configuration from multiple sources

**Declaration:**
```csharp
public interface IConfiguration
{
    string this[string key] { get; set; }
    IEnumerable<IConfigurationSection> GetChildren();
    IChangeToken GetReloadToken();
    IConfigurationSection GetSection(string key);
}
```

**Common Members:**

| Member | Return Type | Description |
|--------|------------|-------------|
| `this[string key]` | `string` | Get/set value by key (e.g., "ConnectionStrings:Default") |
| `GetSection(key)` | `IConfigurationSection` | Get configuration section |
| `GetChildren()` | `IEnumerable<IConfigurationSection>` | Get child sections |
| `GetReloadToken()` | `IChangeToken` | Token that triggers when configuration changes |

**Usage Examples:**

```csharp
// Get simple value
var apiUrl = configuration["ApiUrl"];

// Get nested value with colon separator
var dbConnection = configuration["ConnectionStrings:Default"];

// Get section
var emailSection = configuration.GetSection("EmailSettings");
var smtpServer = emailSection["SmtpServer"];

// Bind to object
var emailSettings = new EmailSettings();
configuration.GetSection("EmailSettings").Bind(emailSettings);

// Get strongly typed value
var port = configuration.GetValue<int>("EmailSettings:Port");
var enabled = configuration.GetValue<bool>("Features:EmailEnabled", defaultValue: true);
```

---

### IConfigurationSection Interface

**Namespace:** `Microsoft.Extensions.Configuration`

**Purpose:** Represents a section of configuration

**Declaration:**
```csharp
public interface IConfigurationSection : IConfiguration
{
    string Key { get; }
    string Path { get; }
    string Value { get; set; }
}
```

**Additional Members:**

| Member | Return Type | Description |
|--------|------------|-------------|
| `Key` | `string` | Section key name |
| `Path` | `string` | Full path to this section |
| `Value` | `string` | Section value (if it's a value, not object) |

**Usage:**

```csharp
var emailSection = configuration.GetSection("EmailSettings");
Console.WriteLine($"Key: {emailSection.Key}");        // "EmailSettings"
Console.WriteLine($"Path: {emailSection.Path}");      // "EmailSettings"

var smtpServer = emailSection["SmtpServer"];
```

---

### ConfigurationBuilder Class

**Namespace:** `Microsoft.Extensions.Configuration`

**Purpose:** Build configuration from multiple sources

**Common Methods:**

| Method | Description |
|--------|-------------|
| `AddJsonFile(path, optional, reloadOnChange)` | Add JSON file |
| `AddXmlFile(path, optional, reloadOnChange)` | Add XML file |
| `AddIniFile(path, optional, reloadOnChange)` | Add INI file |
| `AddEnvironmentVariables()` | Add environment variables |
| `AddCommandLine(args)` | Add command-line arguments |
| `AddUserSecrets<T>()` | Add user secrets (dev only) |
| `AddInMemoryCollection()` | Add in-memory collection |
| `Build()` | Build and return IConfiguration |

**Usage:**

```csharp
var configuration = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{env}.json", optional: true, reloadOnChange: true)
    .AddEnvironmentVariables()
    .AddCommandLine(args)
    .Build();
```

---

### IServiceCollection Interface

**Namespace:** `Microsoft.Extensions.DependencyInjection`

**Purpose:** Register services for dependency injection

**Common Extension Methods:**

| Method | Lifetime | Description |
|--------|----------|-------------|
| `AddTransient<TService, TImplementation>()` | Transient | New instance every time |
| `AddTransient<TService>()` | Transient | Same type as service and implementation |
| `AddScoped<TService, TImplementation>()` | Scoped | One instance per request |
| `AddScoped<TService>()` | Scoped | Same type as service and implementation |
| `AddSingleton<TService, TImplementation>()` | Singleton | One instance for app lifetime |
| `AddSingleton<TService>()` | Singleton | Same type as service and implementation |
| `AddSingleton<TService>(instance)` | Singleton | Use provided instance |

**Advanced Methods:**

```csharp
// Factory-based registration
builder.Services.AddTransient<IEmailService>(sp =>
{
    var logger = sp.GetRequiredService<ILogger<EmailService>>();
    var settings = sp.GetRequiredService<IOptions<EmailSettings>>();
    return new EmailService(logger, settings);
});

// Try add (only if not already registered)
builder.Services.TryAddScoped<IUserService, UserService>();

// Replace existing registration
builder.Services.Replace(ServiceDescriptor.Scoped<IUserService, NewUserService>());

// Remove service
builder.Services.RemoveAll<IUserService>();
```

---

### IServiceProvider Interface

**Namespace:** `Microsoft.Extensions.DependencyInjection`

**Purpose:** Resolve registered services

**Common Methods:**

| Method | Description |
|--------|-------------|
| `GetService<T>()` | Get service (returns null if not found) |
| `GetRequiredService<T>()` | Get service (throws if not found) |
| `GetServices<T>()` | Get all registered implementations |

**Usage:**

```csharp
// In middleware or where IServiceProvider is available
var userService = serviceProvider.GetRequiredService<IUserService>();

// Get optional service
var cacheService = serviceProvider.GetService<ICacheService>();
if (cacheService != null)
{
    // Use cache
}

// Get all implementations
var validators = serviceProvider.GetServices<IValidator>();
```

**Create Scope:**

```csharp
// Create a new scope for scoped services
using (var scope = serviceProvider.CreateScope())
{
    var scopedProvider = scope.ServiceProvider;
    var dbContext = scopedProvider.GetRequiredService<ApplicationDbContext>();
    
    // Use dbContext
}
```

---

### IOptions<T>, IOptionsSnapshot<T>, IOptionsMonitor<T>

**Namespace:** `Microsoft.Extensions.Options`

**Purpose:** Access strongly-typed configuration

**Comparison:**

| Interface | Lifetime | Reload | Use Case |
|-----------|----------|--------|----------|
| `IOptions<T>` | Singleton | ❌ No | Read once at startup |
| `IOptionsSnapshot<T>` | Scoped | ✅ Per request | Per-request reload |
| `IOptionsMonitor<T>` | Singleton | ✅ Live | Real-time configuration changes |

**IOptions<T> - Most Common:**

```csharp
public class EmailService
{
    private readonly EmailSettings _settings;
    
    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value; // Get current value
    }
    
    public void SendEmail()
    {
        // Use _settings.SmtpServer, etc.
    }
}
```

**IOptionsSnapshot<T> - Per Request:**

```csharp
public class OrderService
{
    private readonly IOptionsSnapshot<OrderSettings> _options;
    
    public OrderService(IOptionsSnapshot<OrderSettings> options)
    {
        _options = options;
    }
    
    public async Task ProcessOrder()
    {
        var settings = _options.Value; // Fresh value per request
        // Use settings
    }
}
```

**IOptionsMonitor<T> - Live Reload:**

```csharp
public class CacheService
{
    private readonly IOptionsMonitor<CacheSettings> _options;
    
    public CacheService(IOptionsMonitor<CacheSettings> options)
    {
        _options = options;
        
        // React to changes
        _options.OnChange(newSettings =>
        {
            Console.WriteLine("Cache settings changed!");
            // Reconfigure cache
        });
    }
    
    public void ClearCache()
    {
        var settings = _options.CurrentValue; // Always current
        // Use settings
    }
}
```

---

### ILogger<T> and ILoggerFactory

**Namespace:** `Microsoft.Extensions.Logging`

**Purpose:** Logging throughout application

**ILogger<T> Members:**

| Method | Description |
|--------|-------------|
| `LogTrace(message)` | Detailed diagnostic information |
| `LogDebug(message)` | Debugging information |
| `LogInformation(message)` | General information |
| `LogWarning(message)` | Warning but application continues |
| `LogError(exception, message)` | Error occurred |
| `LogCritical(exception, message)` | Critical failure |

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
        _logger.LogInformation($"Getting user {id}");
        
        try
        {
            var user = await _repository.GetByIdAsync(id);
            
            if (user == null)
            {
                _logger.LogWarning($"User {id} not found");
            }
            
            return user;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, $"Error getting user {id}");
            throw;
        }
    }
}
```

**Structured Logging:**

```csharp
_logger.LogInformation("User {UserId} logged in at {LoginTime}", userId, DateTime.UtcNow);
```

---

### WebApplication and WebApplicationBuilder

**Namespace:** `Microsoft.AspNetCore.Builder`

**WebApplicationBuilder Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Services` | `IServiceCollection` | Register services |
| `Configuration` | `ConfigurationManager` | Access configuration |
| `Environment` | `IWebHostEnvironment` | Environment info |
| `Host` | `IHostBuilder` | Configure host |
| `WebHost` | `IWebHostBuilder` | Configure web host |
| `Logging` | `ILoggingBuilder` | Configure logging |

**WebApplication Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Services` | `IServiceProvider` | Resolve services |
| `Configuration` | `IConfiguration` | Access configuration |
| `Environment` | `IWebHostEnvironment` | Environment info |
| `Lifetime` | `IHostApplicationLifetime` | Application lifetime events |
| `Logger` | `ILogger` | Logger for WebApplication |
| `Urls` | `ICollection<string>` | URLs to listen on |

**Usage:**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Use builder properties
builder.Services.AddControllers();
builder.Logging.AddConsole();

var connectionString = builder.Configuration.GetConnectionString("Default");

if (builder.Environment.IsDevelopment())
{
    builder.Services.AddDatabaseDeveloperPageExceptionFilter();
}

var app = builder.Build();

// Use app properties
var logger = app.Logger;
logger.LogInformation($"Environment: {app.Environment.EnvironmentName}");
logger.LogInformation($"Listening on: {string.Join(", ", app.Urls)}");

app.Run();
```

---

## 11. Configuration Deep-Dive

### Configuration Providers & Order

ASP.NET Core loads configuration from providers in this order (last wins):

1. **appsettings.json** - Base configuration
2. **appsettings.{Environment}.json** - Environment-specific
3. **User Secrets** - Development secrets (never commit)
4. **Environment Variables** - Server/container configuration
5. **Command-line Arguments** - Runtime overrides

**Example:**

```json
// appsettings.json
{
  "ApiUrl": "https://api.example.com",
  "LogLevel": "Warning"
}

// appsettings.Development.json
{
  "ApiUrl": "https://localhost:5001",  // ✅ Overrides base
  "LogLevel": "Debug"                   // ✅ Overrides base
}
```

**Environment Variable (highest priority):**
```bash
export ApiUrl="https://staging-api.example.com"  # ✅ Overrides everything
```

**Command-line (highest priority):**
```bash
dotnet run --ApiUrl="https://prod-api.example.com"  # ✅ Overrides all
```

---

### User Secrets (Development Only)

**Purpose:** Store secrets locally without committing to source control

**Step 1: Initialize User Secrets**
```bash
dotnet user-secrets init
```

This adds to .csproj:
```xml
<PropertyGroup>
  <UserSecretsId>a-unique-guid-here</UserSecretsId>
</PropertyGroup>
```

**Step 2: Set Secrets**
```bash
dotnet user-secrets set "ConnectionStrings:Default" "Server=localhost;Database=Dev"
dotnet user-secrets set "ApiKey" "secret-key-12345"
```

**Step 3: List Secrets**
```bash
dotnet user-secrets list
```

**Step 4: Use in Code**
```csharp
// Automatically loaded in Development environment
var connectionString = builder.Configuration["ConnectionStrings:Default"];
```

**Location:**
- Windows: `%APPDATA%\Microsoft\UserSecrets\<UserSecretsId>\secrets.json`
- Linux/macOS: `~/.microsoft/usersecrets/<UserSecretsId>/secrets.json`

---

### Environment Variables

**Naming Convention:** Use double underscore `__` for nesting

```bash
# appsettings.json structure:
# {
#   "ConnectionStrings": {
#     "Default": "..."
#   }
# }

# Environment variable equivalent:
export ConnectionStrings__Default="Server=prod;Database=ProdDb"

# Another example:
export EmailSettings__SmtpServer="smtp.production.com"
export EmailSettings__Port="587"
```

**Usage:**
```csharp
var connectionString = builder.Configuration["ConnectionStrings:Default"];
// Will use environment variable if set, otherwise falls back to appsettings.json
```

---

### Azure Key Vault Integration

**Step 1: Install Package**
```bash
dotnet add package Azure.Extensions.AspNetCore.Configuration.Secrets
dotnet add package Azure.Identity
```

**Step 2: Configure**
```csharp
var builder = WebApplication.CreateBuilder(args);

if (!builder.Environment.IsDevelopment())
{
    var keyVaultUrl = builder.Configuration["KeyVaultUrl"];
    
    builder.Configuration.AddAzureKeyVault(
        new Uri(keyVaultUrl),
        new DefaultAzureCredential());
}

var app = builder.Build();
```

**Step 3: Use Secrets**
```csharp
// Key Vault secret names use hyphens: "ConnectionStrings--Default"
var connectionString = builder.Configuration["ConnectionStrings:Default"];
```

---

### Configuration Validation

**Step 1: Add Validation Attributes**
```csharp
using System.ComponentModel.DataAnnotations;

public class EmailSettings
{
    public const string SectionName = "EmailSettings";
    
    [Required]
    [EmailAddress]
    public string SmtpServer { get; set; }
    
    [Range(1, 65535)]
    public int Port { get; set; }
    
    [Required]
    public string Username { get; set; }
    
    [Required]
    [MinLength(8)]
    public string Password { get; set; }
}
```

**Step 2: Enable Validation**
```csharp
builder.Services
    .AddOptions<EmailSettings>()
    .Bind(builder.Configuration.GetSection(EmailSettings.SectionName))
    .ValidateDataAnnotations()
    .ValidateOnStart(); // ✅ Validate on startup (fail fast)
```

**Step 3: Custom Validation**
```csharp
builder.Services
    .AddOptions<EmailSettings>()
    .Bind(builder.Configuration.GetSection(EmailSettings.SectionName))
    .Validate(settings =>
    {
        if (settings.Port == 25 && settings.EnableSsl)
        {
            return false; // Port 25 doesn't support SSL
        }
        return true;
    }, "Invalid SMTP configuration")
    .ValidateOnStart();
```

---

### Complex Configuration Binding

**Arrays:**
```json
{
  "AllowedHosts": ["example.com", "api.example.com", "app.example.com"]
}
```

```csharp
var allowedHosts = builder.Configuration.GetSection("AllowedHosts").Get<string[]>();
```

**Nested Objects:**
```json
{
  "Database": {
    "ConnectionString": "Server=localhost",
    "Timeout": 30,
    "Retry": {
      "MaxAttempts": 3,
      "DelaySeconds": 5
    }
  }
}
```

```csharp
public class DatabaseSettings
{
    public string ConnectionString { get; set; }
    public int Timeout { get; set; }
    public RetrySettings Retry { get; set; }
}

public class RetrySettings
{
    public int MaxAttempts { get; set; }
    public int DelaySeconds { get; set; }
}

// Bind
var dbSettings = builder.Configuration.GetSection("Database").Get<DatabaseSettings>();
```

**Dictionaries:**
```json
{
  "CacheSettings": {
    "Timeouts": {
      "User": 300,
      "Product": 600,
      "Order": 180
    }
  }
}
```

```csharp
var timeouts = builder.Configuration
    .GetSection("CacheSettings:Timeouts")
    .Get<Dictionary<string, int>>();

var userTimeout = timeouts["User"]; // 300
```

---

## 12. Advanced DI Topics

### Multiple Implementations

**Register:**
```csharp
public interface INotificationService
{
    Task NotifyAsync(string message);
}

public class EmailNotificationService : INotificationService
{
    public async Task NotifyAsync(string message) { /* Send email */ }
}

public class SmsNotificationService : INotificationService
{
    public async Task NotifyAsync(string message) { /* Send SMS */ }
}

// Register both
builder.Services.AddTransient<INotificationService, EmailNotificationService>();
builder.Services.AddTransient<INotificationService, SmsNotificationService>();
```

**Use:**
```csharp
public class NotificationManager
{
    private readonly IEnumerable<INotificationService> _notificationServices;
    
    public NotificationManager(IEnumerable<INotificationService> notificationServices)
    {
        _notificationServices = notificationServices; // Gets all implementations
    }
    
    public async Task NotifyAllAsync(string message)
    {
        foreach (var service in _notificationServices)
        {
            await service.NotifyAsync(message);
        }
    }
}
```

---

### Keyed Services ✨ .NET 8.0+

**Register with Keys:**
```csharp
builder.Services.AddKeyedTransient<INotificationService, EmailNotificationService>("email");
builder.Services.AddKeyedTransient<INotificationService, SmsNotificationService>("sms");
builder.Services.AddKeyedTransient<INotificationService, PushNotificationService>("push");
```

**Inject by Key:**
```csharp
public class OrderService
{
    private readonly INotificationService _emailNotification;
    private readonly INotificationService _smsNotification;
    
    public OrderService(
        [FromKeyedServices("email")] INotificationService emailNotification,
        [FromKeyedServices("sms")] INotificationService smsNotification)
    {
        _emailNotification = emailNotification;
        _smsNotification = smsNotification;
    }
    
    public async Task CompleteOrder(Order order)
    {
        await _emailNotification.NotifyAsync($"Order {order.Id} confirmed");
        await _smsNotification.NotifyAsync($"Order {order.Id} shipped");
    }
}
```

**Get Keyed Service from IServiceProvider:**
```csharp
var emailService = serviceProvider.GetRequiredKeyedService<INotificationService>("email");
```

---

### Decorator Pattern

**Scenario:** Add logging/caching to existing service

```csharp
public interface IUserService
{
    Task<User> GetUserAsync(int id);
}

// Original implementation
public class UserService : IUserService
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
}

// Decorator with caching
public class CachedUserService : IUserService
{
    private readonly IUserService _inner;
    private readonly ICacheService _cache;
    
    public CachedUserService(IUserService inner, ICacheService cache)
    {
        _inner = inner;
        _cache = cache;
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        var cacheKey = $"user_{id}";
        
        if (_cache.TryGet(cacheKey, out User cachedUser))
        {
            return cachedUser;
        }
        
        var user = await _inner.GetUserAsync(id);
        _cache.Set(cacheKey, user);
        
        return user;
    }
}

// Register
builder.Services.AddScoped<UserService>(); // Register concrete type
builder.Services.AddScoped<IUserService>(sp =>
{
    var innerService = sp.GetRequiredService<UserService>();
    var cacheService = sp.GetRequiredService<ICacheService>();
    return new CachedUserService(innerService, cacheService);
});
```

---

### Generic Type Registration

**Register Generic Repository:**
```csharp
// Generic interface
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
}

// Generic implementation
public class Repository<T> : IRepository<T> where T : class
{
    private readonly DbContext _context;
    
    public Repository(DbContext context)
    {
        _context = context;
    }
    
    public async Task<T> GetByIdAsync(int id)
    {
        return await _context.Set<T>().FindAsync(id);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _context.Set<T>().ToListAsync();
    }
}

// Register generic type
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));

// Use
public class UserService
{
    private readonly IRepository<User> _userRepository;
    private readonly IRepository<Order> _orderRepository;
    
    public UserService(
        IRepository<User> userRepository,
        IRepository<Order> orderRepository)
    {
        _userRepository = userRepository;
        _orderRepository = orderRepository;
    }
}
```

---

### Conditional Registration

**Register Based on Environment:**
```csharp
if (builder.Environment.IsDevelopment())
{
    builder.Services.AddScoped<IEmailService, FakeEmailService>();
}
else
{
    builder.Services.AddScoped<IEmailService, SmtpEmailService>();
}
```

**Register Based on Configuration:**
```csharp
var useRedisCache = builder.Configuration.GetValue<bool>("UseRedisCache");

if (useRedisCache)
{
    builder.Services.AddSingleton<ICacheService, RedisCacheService>();
}
else
{
    builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
}
```

**TryAdd Methods:**
```csharp
// Only register if not already registered
builder.Services.TryAddScoped<IUserService, UserService>();

// Won't replace existing registration
builder.Services.TryAddScoped<IUserService, NewUserService>();
```

---

### Replacing Built-in Services

**Replace Existing Service:**
```csharp
using Microsoft.Extensions.DependencyInjection.Extensions;

// Replace
builder.Services.Replace(
    ServiceDescriptor.Scoped<IUserService, NewUserService>());

// Or remove and add
builder.Services.RemoveAll<IUserService>();
builder.Services.AddScoped<IUserService, NewUserService>();
```

---

## 13. Performance Considerations

### Service Lifetime Performance Impact

**Transient:**
- ✅ No memory overhead between requests
- ❌ More allocations (GC pressure)
- ❌ Slower resolution

**Scoped:**
- ✅ Balanced performance
- ✅ Shared within request (less allocations)
- ✅ Automatically disposed per request

**Singleton:**
- ✅ Fastest resolution
- ✅ No allocations after first creation
- ⚠️ Memory stays for app lifetime
- ⚠️ Must be thread-safe

---

### Avoid Service Locator Anti-Pattern

**Slow (Service Locator):**
```csharp
public class SlowService
{
    private readonly IServiceProvider _provider;
    
    public SlowService(IServiceProvider provider)
    {
        _provider = provider;
    }
    
    public void DoWork()
    {
        // ❌ Resolves service every time (slow)
        var service = _provider.GetRequiredService<IEmailService>();
        service.SendEmail();
    }
}
```

**Fast (Constructor Injection):**
```csharp
public class FastService
{
    private readonly IEmailService _emailService;
    
    public FastService(IEmailService emailService)
    {
        _emailService = emailService; // ✅ Resolved once
    }
    
    public void DoWork()
    {
        _emailService.SendEmail(); // ✅ Fast!
    }
}
```

---

### Configuration Performance

**Slow (Repeated Reads):**
```csharp
public class SlowService
{
    private readonly IConfiguration _configuration;
    
    public SlowService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public void DoWork()
    {
        // ❌ Re-reads config every time
        var apiUrl = _configuration["ApiUrl"];
        // Use apiUrl
    }
}
```

**Fast (IOptions):**
```csharp
public class FastService
{
    private readonly AppSettings _settings;
    
    public FastService(IOptions<AppSettings> options)
    {
        _settings = options.Value; // ✅ Cached
    }
    
    public void DoWork()
    {
        var apiUrl = _settings.ApiUrl; // ✅ Fast property access
    }
}
```

---

## Summary: Complete Checklist

### Configuration Checklist

**Setup:**
- [ ] Create appsettings.json with base configuration
- [ ] Create appsettings.Development.json for dev overrides
- [ ] Use User Secrets for local development secrets
- [ ] Use environment variables for production
- [ ] Set up Azure Key Vault for production secrets

**Structure:**
- [ ] Group related settings in sections
- [ ] Use const string for section names
- [ ] Create strongly-typed settings classes
- [ ] Add validation attributes where needed
- [ ] Enable ValidateOnStart for fail-fast

**Usage:**
- [ ] Use IOptions<T> for production code
- [ ] Use IOptionsSnapshot<T> for per-request reload
- [ ] Use IOptionsMonitor<T> for live reload scenarios
- [ ] Avoid reading IConfiguration directly in services

---

### Dependency Injection Checklist

**Registration:**
- [ ] Choose appropriate lifetime (Transient/Scoped/Singleton)
- [ ] Program to interfaces, not implementations
- [ ] Use extension methods for clean registration
- [ ] Group related services together
- [ ] Validate no captive dependencies

**Usage:**
- [ ] Use constructor injection (not service locator)
- [ ] Inject ILogger<T> for logging
- [ ] Keep constructors simple (no logic)
- [ ] Don't inject IServiceProvider unless necessary
- [ ] Use FromServices for action-specific dependencies

**Organization:**
- [ ] Create extension methods per feature/layer
- [ ] Keep Program.cs clean and readable
- [ ] Document complex dependency chains
- [ ] Handle circular dependencies properly
- [ ] Test service resolution in integration tests

**Best Practices:**
- [ ] Singleton services are thread-safe
- [ ] Scoped services don't reference Singleton
- [ ] DbContext is Scoped (default)
- [ ] Dispose pattern handled by DI container
- [ ] Use IOptions for configuration injection

---

**This completes the Configuration & Dependency Injection guide combining practical hands-on content with deep technical reference!**
