# ASP.NET Core Fundamentals & Project Structure - Complete Guide
## Practical Guide + Technical Reference

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. What is ASP.NET Core
2. Getting Started - 3 Ways to Create a Project
3. Project Structure Deep Dive
4. .NET CLI Commands Reference
5. Application Startup Lifecycle
6. Environments
7. Troubleshooting Common Issues
8. Best Practices

### Part 2: Technical Reference (Deep Dive)
9. Important Classes & Interfaces Reference
10. Configuration System Deep-Dive
11. Hosting Models Details
12. Advanced Topics

---

# PART 1: PRACTICAL GUIDE

---

## 1. What is ASP.NET Core?

**Simple Definition:** A cross-platform, high-performance framework for building modern web applications and APIs.

**Think of it like:** The engine and chassis for your web application. You bring the design and business logic, ASP.NET Core provides the infrastructure.

### Key Benefits

✅ **Cross-Platform** - Runs on Windows, Linux, macOS  
✅ **High Performance** - One of the fastest web frameworks  
✅ **Open Source** - Free and community-driven  
✅ **Cloud-Ready** - Built for Azure, AWS, Docker  
✅ **Modern** - Supports latest C# features  

---

### .NET Evolution Timeline

```
2002  ASP.NET (Framework)     Windows only, IIS only
      ↓
2016  ASP.NET Core 1.0         Cross-platform begins
      ↓
2018  ASP.NET Core 2.0/2.1    Razor Pages, SignalR
      ↓
2019  ASP.NET Core 3.0/3.1    Blazor, gRPC, LTS
      ↓
2020  .NET 5.0                Unified .NET (Core dropped)
      ↓
2021  .NET 6.0 (LTS)          Minimal APIs, Hot Reload
      ↓
2022  .NET 7.0                Performance improvements
      ↓
2023  .NET 8.0 (LTS)          Native AOT, Blazor enhancements
      ↓
2024  .NET 9.0                Latest features
      ↓
2025  .NET 10.0 (Expected)    Future release
```

**LTS = Long-Term Support (3 years)**  
**Standard = Support for 18 months**

---

### .NET Framework vs .NET Core vs .NET 5+

| Feature | .NET Framework | .NET Core | .NET 5+ |
|---------|----------------|-----------|---------|
| Cross-Platform | ❌ Windows only | ✅ Win/Linux/Mac | ✅ Win/Linux/Mac |
| Open Source | ❌ No | ✅ Yes | ✅ Yes |
| Performance | Good | Better | Best |
| Side-by-side | ❌ No | ✅ Yes | ✅ Yes |
| Future Support | ❌ Maintenance | ❌ EOL | ✅ Active |
| When to Use | Legacy apps | ❌ Use .NET 5+ | ✅ New projects |

**Recommendation:** Use .NET 8.0 (LTS) for new projects (as of 2024-2025)

---

### When to Use ASP.NET Core

**✅ Perfect For:**
- REST APIs
- Web APIs
- Microservices
- Real-time apps (SignalR)
- Cloud-native applications
- Docker containers
- Cross-platform deployment

**⚠️ Consider Alternatives:**
- Simple static websites → Consider static site generators
- WordPress-style CMS → WordPress/Umbraco
- Desktop apps → WPF, WinForms, MAUI

---

## 2. Getting Started - 3 Ways to Create a Project

### Method 1: Visual Studio (Beginner-Friendly)

**When to use:**
- ✅ New to .NET
- ✅ Windows development
- ✅ Like GUI tools
- ✅ Integrated debugging

**Step 1: Open Visual Studio**
- Launch Visual Studio 2022 (or later)

**Step 2: Create New Project**
1. File → New → Project
2. Search for "ASP.NET Core Web"
3. Choose template:
   - **ASP.NET Core Web API** (REST APIs)
   - **ASP.NET Core Web App (MVC)** (Full websites)
   - **ASP.NET Core Web App (Razor Pages)** (Page-based apps)
   - **Blazor** (SPA with C#)

**Step 3: Configure Project**
```
Project Name:     MyFirstApi
Location:         C:\Projects\
Solution Name:    MyFirstApi
Framework:        .NET 8.0 (Long-term support)
Authentication:   None (for now)
HTTPS:            ✅ Enable
Docker:           ☐ Optional
```

**Step 4: Run**
- Press F5 (or click "▶ MyFirstApi")
- Browser opens to `https://localhost:7xxx/swagger`

---

### Method 2: .NET CLI (Recommended for Professionals)

**When to use:**
- ✅ Quick project creation
- ✅ Automation/scripts
- ✅ VS Code users
- ✅ Command-line preference

**Step 1: Check .NET Installation**
```bash
dotnet --version
# Should show: 8.0.xxx or higher
```

**Step 2: Create Project**
```bash
# Web API (most common)
dotnet new webapi -n MyFirstApi

# MVC Application
dotnet new mvc -n MyMvcApp

# Razor Pages
dotnet new razor -n MyRazorApp

# Blazor Server
dotnet new blazorserver -n MyBlazorApp

# Blazor WebAssembly
dotnet new blazorwasm -n MyBlazorWasm
```

**Step 3: Navigate and Run**
```bash
cd MyFirstApi
dotnet run
```

**Output:**
```
Building...
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:7148
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
```

**Step 4: Test**
- Open browser: `https://localhost:7148/swagger`
- Or use curl: `curl https://localhost:7148/weatherforecast`

---

### Method 3: VS Code (Lightweight)

**When to use:**
- ✅ Lightweight editor
- ✅ Cross-platform development
- ✅ Don't need full Visual Studio
- ✅ Resource-constrained machine

**Step 1: Install Prerequisites**
- Install [Visual Studio Code](https://code.visualstudio.com/)
- Install C# extension (by Microsoft)
- Install .NET SDK

**Step 2: Create Project via Terminal**
```bash
# In VS Code terminal (Ctrl + `)
dotnet new webapi -n MyFirstApi
code -r MyFirstApi
```

**Step 3: Trust HTTPS Certificate** (First time only)
```bash
dotnet dev-certs https --trust
```

**Step 4: Run and Debug**
- Press F5
- VS Code asks to add debug configuration → Yes
- Creates `.vscode/launch.json` automatically
- App starts with debugger attached

---

### Template Comparison

| Template | Use Case | Complexity | Output |
|----------|----------|------------|---------|
| `webapi` | REST APIs, microservices | Simple | JSON responses |
| `mvc` | Traditional websites | Medium | HTML views |
| `razor` | Page-focused apps | Simple-Medium | HTML pages |
| `blazorserver` | Interactive UIs (server) | Medium | C# + HTML |
| `blazorwasm` | SPA (runs in browser) | Medium-Complex | WebAssembly |
| `empty` | Start from scratch | Minimal | Nothing |

**Decision Tree:**
```
What are you building?
├─ REST API / Backend only?
│  └─ Use: webapi
│
├─ Traditional website with views?
│  └─ Use: mvc
│
├─ Simple content-driven site?
│  └─ Use: razor
│
├─ Interactive SPA with C#?
│  ├─ Server-side processing?
│  │  └─ Use: blazorserver
│  └─ Client-side processing?
│     └─ Use: blazorwasm
│
└─ Need full control?
   └─ Use: empty
```

---

## 3. Project Structure Deep Dive

### Complete Folder Structure (Web API Project)

```
MyFirstApi/
│
├── Properties/
│   └── launchSettings.json        # Development settings
│
├── wwwroot/                        # Static files (CSS, JS, images)
│   ├── css/
│   ├── js/
│   └── images/
│
├── Controllers/                    # API endpoints
│   └── WeatherForecastController.cs
│
├── Models/                         # Data models
│   └── WeatherForecast.cs
│
├── Services/                       # Business logic
│   ├── IWeatherService.cs
│   └── WeatherService.cs
│
├── Data/                           # Database context
│   └── ApplicationDbContext.cs
│
├── DTOs/                           # Data Transfer Objects
│   ├── WeatherDto.cs
│   └── CreateWeatherDto.cs
│
├── Middleware/                     # Custom middleware
│   └── RequestLoggingMiddleware.cs
│
├── Program.cs                      # Application entry point ⚡
├── appsettings.json               # Configuration
├── appsettings.Development.json   # Dev-only config
└── MyFirstApi.csproj              # Project file
```

---

### Program.cs - The Entry Point

ASP.NET Core uses different hosting models depending on version:

#### ✨ .NET 6+ Minimal Hosting Model (Modern)

**File:** `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

// ═══════════════════════════════════════════════════════════
// PART 1: CONFIGURE SERVICES (Dependency Injection)
// ═══════════════════════════════════════════════════════════

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add your services
builder.Services.AddScoped<IWeatherService, WeatherService>();

var app = builder.Build();

// ═══════════════════════════════════════════════════════════
// PART 2: CONFIGURE MIDDLEWARE (HTTP Pipeline)
// ═══════════════════════════════════════════════════════════

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Key Points:**
- No `Main` method (it's implicit)
- No `Startup` class (all in one file)
- Clean and concise
- Recommended for all new projects

---

#### Pre-.NET 6 Model (Legacy)

**File:** `Program.cs`

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

**File:** `Startup.cs`

```csharp
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    // Configure services (DI)
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
        services.AddSwaggerGen();
    }

    // Configure middleware
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
            app.UseSwagger();
            app.UseSwaggerUI();
        }

        app.UseHttpsRedirection();
        app.UseRouting();
        app.UseAuthorization();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}
```

---

#### Side-by-Side Comparison

| Aspect | Minimal (.NET 6+) | Legacy (Pre-.NET 6) |
|--------|-------------------|---------------------|
| Files | 1 file (Program.cs) | 2 files (Program.cs + Startup.cs) |
| Lines of Code | ~15-20 | ~40-50 |
| Complexity | Lower | Higher |
| Learning Curve | Easier | Steeper |
| When to Use | ✅ New projects | ❌ Legacy only |

**Recommendation:** Always use Minimal Hosting Model (.NET 6+) for new projects

---

### appsettings.json - Configuration

**Purpose:** Store application configuration (connection strings, API keys, settings)

**Structure:**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;Trusted_Connection=true;"
  },
  
  "JwtSettings": {
    "SecretKey": "your-secret-key-here",
    "Issuer": "MyApp",
    "Audience": "MyApp-Users",
    "ExpiryMinutes": 60
  },
  
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "Port": 587,
    "Username": "your-email@gmail.com",
    "FromAddress": "noreply@myapp.com"
  }
}
```

---

### Environment-Specific Configuration

ASP.NET Core supports multiple configuration files:

```
appsettings.json                    ← Base settings (all environments)
appsettings.Development.json        ← Development only
appsettings.Staging.json           ← Staging only
appsettings.Production.json        ← Production only
```

**Loading Order (Priority):**
```
1. appsettings.json                     (Lowest priority)
2. appsettings.{Environment}.json       (Overrides #1)
3. User Secrets (Development only)      (Overrides #2)
4. Environment Variables                (Overrides #3)
5. Command-line arguments               (Highest priority)
```

---

### Reading Configuration (3 Methods)

#### Method 1: IConfiguration Directly (Simple)

**When to use:** Quick access, one-time reads

```csharp
public class WeatherController : ControllerBase
{
    private readonly IConfiguration _configuration;
    
    public WeatherController(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    [HttpGet]
    public IActionResult Get()
    {
        // Read single value
        var logLevel = _configuration["Logging:LogLevel:Default"];
        
        // Read connection string
        var connString = _configuration.GetConnectionString("DefaultConnection");
        
        // Read section
        var jwtIssuer = _configuration["JwtSettings:Issuer"];
        
        return Ok(new { logLevel, connString, jwtIssuer });
    }
}
```

---

#### Method 2: GetSection (Medium)

**When to use:** Group related settings

```csharp
public class EmailController : ControllerBase
{
    private readonly IConfiguration _configuration;
    
    public EmailController(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    [HttpPost("send")]
    public IActionResult SendEmail()
    {
        var emailConfig = _configuration.GetSection("EmailSettings");
        
        var smtpServer = emailConfig["SmtpServer"];
        var port = emailConfig.GetValue<int>("Port");
        var username = emailConfig["Username"];
        
        // Use settings...
        
        return Ok();
    }
}
```

---

#### Method 3: Strongly-Typed with IOptions<T> (Recommended)

**When to use:** Production apps, type safety, testability

**Step 1: Create Options Class**

```csharp
public class JwtSettings
{
    public const string SectionName = "JwtSettings";
    
    public string SecretKey { get; set; } = string.Empty;
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public int ExpiryMinutes { get; set; } = 60;
}
```

**Step 2: Register in Program.cs**

```csharp
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection(JwtSettings.SectionName));
```

**Step 3: Inject IOptions<T>**

```csharp
public class TokenService
{
    private readonly JwtSettings _jwtSettings;
    
    public TokenService(IOptions<JwtSettings> jwtSettings)
    {
        _jwtSettings = jwtSettings.Value;
    }
    
    public string GenerateToken(string userId)
    {
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_jwtSettings.SecretKey));
        
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        
        var token = new JwtSecurityToken(
            issuer: _jwtSettings.Issuer,
            audience: _jwtSettings.Audience,
            expires: DateTime.UtcNow.AddMinutes(_jwtSettings.ExpiryMinutes),
            signingCredentials: credentials
        );
        
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

**Benefits:**
- ✅ Type safety (compile-time checking)
- ✅ IntelliSense support
- ✅ Easy testing (mock IOptions)
- ✅ Validation support

---

### wwwroot/ - Static Files

**Purpose:** Serve static files (CSS, JavaScript, images, fonts)

**Default Structure:**
```
wwwroot/
├── css/
│   └── site.css
├── js/
│   └── site.js
├── lib/              # Third-party libraries
│   ├── bootstrap/
│   └── jquery/
└── images/
    └── logo.png
```

**Enable Static Files:**

```csharp
// Program.cs
app.UseStaticFiles();  // Serves files from wwwroot/
```

**Accessing Files:**
```html
<!-- In HTML/Razor -->
<link rel="stylesheet" href="/css/site.css" />
<script src="/js/site.js"></script>
<img src="/images/logo.png" alt="Logo" />
```

**Custom Static File Directory:**

```csharp
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(Directory.GetCurrentDirectory(), "CustomFiles")),
    RequestPath = "/files"
});

// Access: https://localhost:5000/files/document.pdf
```

---

### .csproj - Project File

**Purpose:** Defines project settings, dependencies, build configuration

**Example:** `MyFirstApi.csproj`

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="8.0.0" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.5.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
  </ItemGroup>

</Project>
```

**Key Elements:**

| Element | Purpose | Example |
|---------|---------|---------|
| `Sdk` | Project type | `Microsoft.NET.Sdk.Web` |
| `TargetFramework` | .NET version | `net8.0` |
| `Nullable` | Nullable reference types | `enable` |
| `ImplicitUsings` | Auto-import common namespaces | `enable` |
| `PackageReference` | NuGet packages | `Swashbuckle.AspNetCore` |

---

### Adding NuGet Packages (3 Methods)

#### Method 1: Visual Studio (GUI)

1. Right-click project → Manage NuGet Packages
2. Browse tab → Search "EntityFrameworkCore"
3. Click Install

#### Method 2: .NET CLI (Recommended)

```bash
# Add package
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

# Add specific version
dotnet add package Newtonsoft.Json --version 13.0.3

# Restore packages
dotnet restore
```

#### Method 3: Edit .csproj Manually

```xml
<ItemGroup>
  <PackageReference Include="AutoMapper" Version="12.0.1" />
</ItemGroup>
```

Then run:
```bash
dotnet restore
```

---

### Folder Conventions (Best Practices)

```
MyApi/
│
├── Controllers/          # API endpoints (HTTP handling)
│   ├── ProductsController.cs
│   └── UsersController.cs
│
├── Models/              # Domain models (database entities)
│   ├── Product.cs
│   └── User.cs
│
├── DTOs/                # Data Transfer Objects (API contracts)
│   ├── ProductDto.cs
│   ├── CreateProductDto.cs
│   └── UpdateProductDto.cs
│
├── Services/            # Business logic
│   ├── IProductService.cs
│   ├── ProductService.cs
│   ├── IEmailService.cs
│   └── EmailService.cs
│
├── Repositories/        # Data access (if not using EF Core directly)
│   ├── IProductRepository.cs
│   └── ProductRepository.cs
│
├── Data/                # Database context
│   ├── ApplicationDbContext.cs
│   └── Migrations/
│
├── Middleware/          # Custom middleware
│   ├── ErrorHandlingMiddleware.cs
│   └── RequestLoggingMiddleware.cs
│
├── Filters/             # Action filters, exception filters
│   ├── ValidationFilter.cs
│   └── CacheFilter.cs
│
├── Validators/          # FluentValidation validators
│   └── CreateProductValidator.cs
│
├── Mappers/             # AutoMapper profiles
│   └── ProductProfile.cs
│
└── Extensions/          # Extension methods
    ├── ServiceExtensions.cs
    └── MiddlewareExtensions.cs
```

**Key Principle:** Separation of Concerns

---

## 4. .NET CLI Commands Reference

### Essential Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `dotnet --version` | Check .NET version | `dotnet --version` |
| `dotnet --info` | Detailed SDK info | `dotnet --info` |
| `dotnet new` | Create new project | `dotnet new webapi -n MyApi` |
| `dotnet restore` | Restore NuGet packages | `dotnet restore` |
| `dotnet build` | Compile project | `dotnet build` |
| `dotnet run` | Build and run | `dotnet run` |
| `dotnet watch` | Run with hot reload | `dotnet watch run` |
| `dotnet test` | Run tests | `dotnet test` |
| `dotnet publish` | Publish for deployment | `dotnet publish -c Release` |
| `dotnet clean` | Clean build artifacts | `dotnet clean` |

---

### Project Management

```bash
# List available templates
dotnet new list

# Create specific project types
dotnet new webapi -n MyApi          # Web API
dotnet new mvc -n MyMvcApp          # MVC
dotnet new razor -n MyRazorApp      # Razor Pages
dotnet new classlib -n MyLibrary    # Class Library
dotnet new xunit -n MyTests         # xUnit Test Project

# Create solution
dotnet new sln -n MySolution

# Add project to solution
dotnet sln add MyApi/MyApi.csproj

# List projects in solution
dotnet sln list
```

---

### Package Management

```bash
# Add NuGet package
dotnet add package EntityFrameworkCore.SqlServer

# Add specific version
dotnet add package Newtonsoft.Json --version 13.0.3

# Remove package
dotnet remove package Newtonsoft.Json

# List packages
dotnet list package

# Update packages
dotnet list package --outdated
dotnet add package PackageName  # Updates to latest
```

---

### Build & Run

```bash
# Build (Debug by default)
dotnet build

# Build Release
dotnet build -c Release

# Run (builds first if needed)
dotnet run

# Run specific project
dotnet run --project MyApi/MyApi.csproj

# Run with hot reload (watches for file changes)
dotnet watch run

# Run without building
dotnet run --no-build
```

---

### Publishing

```bash
# Publish for deployment
dotnet publish -c Release -o ./publish

# Self-contained (includes .NET runtime)
dotnet publish -c Release -r win-x64 --self-contained

# Framework-dependent (requires .NET runtime on server)
dotnet publish -c Release -r linux-x64 --no-self-contained

# Common runtimes:
# - win-x64 (Windows 64-bit)
# - linux-x64 (Linux 64-bit)
# - osx-x64 (macOS 64-bit)
```

---

### Entity Framework Core Commands

```bash
# Install EF Core tools (one-time)
dotnet tool install --global dotnet-ef

# Add migration
dotnet ef migrations add InitialCreate

# Update database
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# Generate SQL script
dotnet ef migrations script

# Drop database
dotnet ef database drop

# List migrations
dotnet ef migrations list
```

---

### Development Helpers

```bash
# Trust HTTPS certificate (first time)
dotnet dev-certs https --trust

# Watch for changes and rebuild
dotnet watch run

# Run with specific environment
dotnet run --environment Production

# Run with specific port
dotnet run --urls "https://localhost:5001;http://localhost:5000"
```

---

## 5. Application Startup Lifecycle

### Visual Flow Diagram

```
1. Application Starts
   ↓
2. CreateBuilder(args)
   ├─ Load Configuration (appsettings.json, env vars, etc.)
   ├─ Set up Logging
   ├─ Configure Kestrel (web server)
   └─ Return WebApplicationBuilder
   ↓
3. Configure Services (builder.Services.Add...)
   ├─ Register Controllers
   ├─ Register Services (DI)
   ├─ Configure Options
   ├─ Add Authentication
   └─ Add Swagger, CORS, etc.
   ↓
4. Build Application (builder.Build())
   ├─ Create Service Provider
   ├─ Validate Services
   └─ Return WebApplication
   ↓
5. Configure Middleware (app.Use...)
   ├─ Exception Handling
   ├─ HTTPS Redirection
   ├─ Static Files
   ├─ Routing
   ├─ Authentication
   ├─ Authorization
   └─ Map Controllers/Endpoints
   ↓
6. Run Application (app.Run())
   ├─ Start Kestrel
   ├─ Listen for HTTP requests
   └─ Process requests through middleware pipeline
   ↓
7. Request Processing
   ↓
8. Shutdown (Ctrl+C or app.Lifetime.StopApplication())
   ├─ Stop accepting new requests
   ├─ Complete in-flight requests
   ├─ Dispose services
   └─ Exit
```

---

### Detailed Startup Code

```csharp
// 1. CREATE BUILDER
var builder = WebApplication.CreateBuilder(args);

// Behind the scenes:
// - Loads appsettings.json
// - Loads appsettings.{Environment}.json
// - Loads environment variables
// - Loads command-line arguments
// - Configures Kestrel (web server)
// - Sets up logging

// 2. CONFIGURE SERVICES (Dependency Injection)
// Order doesn't matter here
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add your custom services
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

// Configure options
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection("JwtSettings"));

// Add authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* config */ });

// Add CORS
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
        policy.AllowAnyOrigin().AllowAnyMethod().AllowAnyHeader());
});

// 3. BUILD APPLICATION
var app = builder.Build();

// Optional: Seed database or run startup tasks
using (var scope = app.Services.CreateScope())
{
    var dbContext = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    await dbContext.Database.MigrateAsync(); // Run migrations
    await SeedData.Initialize(dbContext);     // Seed data
}

// 4. CONFIGURE MIDDLEWARE PIPELINE
// ⚠️ ORDER MATTERS!

// Exception handling (first)
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

// HTTPS redirection
app.UseHttpsRedirection();

// Static files
app.UseStaticFiles();

// Routing
app.UseRouting();

// CORS
app.UseCors();

// Authentication (who are you?)
app.UseAuthentication();

// Authorization (what can you do?)
app.UseAuthorization();

// Custom middleware
app.UseRequestLogging();

// Endpoints (last)
app.MapControllers();

// Health checks
app.MapHealthChecks("/health");

// 5. RUN APPLICATION
app.Run();  // Blocks here until app is shut down
```

---

### Builder Phase vs App Phase

| Phase | Purpose | Examples |
|-------|---------|----------|
| **Builder Phase** | Configure services | `builder.Services.Add...` |
| | Set up DI | `AddScoped<>`, `AddSingleton<>` |
| | Configure options | `Configure<TOptions>()` |
| | Add authentication | `AddAuthentication()` |
| **App Phase** | Configure middleware | `app.Use...` |
| | Set up HTTP pipeline | `UseRouting()`, `UseAuthorization()` |
| | Map endpoints | `MapControllers()`, `MapGet()` |
| | Start server | `app.Run()` |

---

## 6. Environments

### What are Environments?

Environments allow different configurations for Development, Staging, and Production.

**Common Environments:**
- **Development** - Your local machine
- **Staging** - Pre-production testing
- **Production** - Live application

---

### Setting the Environment

#### Method 1: Environment Variable

**Windows (PowerShell):**
```powershell
$env:ASPNETCORE_ENVIRONMENT = "Development"
dotnet run
```

**Windows (Command Prompt):**
```cmd
set ASPNETCORE_ENVIRONMENT=Development
dotnet run
```

**Linux/macOS:**
```bash
export ASPNETCORE_ENVIRONMENT=Development
dotnet run
```

---

#### Method 2: launchSettings.json (Development)

**File:** `Properties/launchSettings.json`

```json
{
  "profiles": {
    "Development": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:7148;http://localhost:5148",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "Production": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": false,
      "applicationUrl": "https://localhost:7148;http://localhost:5148",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Production"
      }
    }
  }
}
```

**Usage:**
```bash
# Run with Development profile
dotnet run --launch-profile Development

# Run with Production profile
dotnet run --launch-profile Production
```

---

#### Method 3: Command Line

```bash
dotnet run --environment Production
```

---

### Environment-Specific Configuration Files

```
appsettings.json                    ← Base (all environments)
appsettings.Development.json        ← Development overrides
appsettings.Staging.json           ← Staging overrides
appsettings.Production.json        ← Production overrides
```

**Example:**

**appsettings.json:**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;Trusted_Connection=true;"
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

**appsettings.Production.json:**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft.AspNetCore": "Error"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=prod-server;Database=ProdDb;User Id=produser;Password=***;"
  }
}
```

---

### Checking Environment in Code

```csharp
var builder = WebApplication.CreateBuilder(args);

// In builder phase
if (builder.Environment.IsDevelopment())
{
    // Development-only services
    builder.Services.AddDatabaseDeveloperPageExceptionFilter();
}

var app = builder.Build();

// In app phase
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseDeveloperExceptionPage();
}
else if (app.Environment.IsStaging())
{
    app.UseExceptionHandler("/staging-error");
}
else // Production
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

// Check specific environment
if (app.Environment.IsEnvironment("QA"))
{
    // QA-specific configuration
}
```

---

### IWebHostEnvironment / IHostEnvironment

```csharp
public class MyService
{
    private readonly IWebHostEnvironment _env;
    
    public MyService(IWebHostEnvironment env)
    {
        _env = env;
    }
    
    public string GetEnvironmentInfo()
    {
        return $@"
            Environment: {_env.EnvironmentName}
            Application: {_env.ApplicationName}
            Content Root: {_env.ContentRootPath}
            Web Root: {_env.WebRootPath}
            IsDevelopment: {_env.IsDevelopment()}
            IsProduction: {_env.IsProduction()}
        ";
    }
}
```

---

### User Secrets (Development Only)

**Purpose:** Store sensitive data (API keys, passwords) outside source control

**Step 1: Initialize User Secrets**

```bash
dotnet user-secrets init
```

This adds to `.csproj`:
```xml
<PropertyGroup>
  <UserSecretsId>a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6</UserSecretsId>
</PropertyGroup>
```

**Step 2: Add Secrets**

```bash
dotnet user-secrets set "JwtSettings:SecretKey" "my-super-secret-key"
dotnet user-secrets set "ConnectionStrings:Default" "Server=localhost;..."
```

**Step 3: List Secrets**

```bash
dotnet user-secrets list
```

**Step 4: Access in Code**

```csharp
var secretKey = builder.Configuration["JwtSettings:SecretKey"];
// In Development: reads from user secrets
// In Production: reads from appsettings.json or environment variables
```

**Location:**
- Windows: `%APPDATA%\Microsoft\UserSecrets\{UserSecretsId}\secrets.json`
- Linux/macOS: `~/.microsoft/usersecrets/{UserSecretsId}/secrets.json`

---

## 7. Troubleshooting Common Issues

### Issue 1: Port Already in Use

**Symptom:**
```
Failed to bind to address https://127.0.0.1:7148: address already in use
```

**Solutions:**

**Option 1: Change Port**
```csharp
// Program.cs
app.Run("https://localhost:7200");
```

**Option 2: Kill Process Using Port**

Windows:
```cmd
netstat -ano | findstr :7148
taskkill /PID <process_id> /F
```

Linux/macOS:
```bash
lsof -i :7148
kill -9 <process_id>
```

---

### Issue 2: HTTPS Certificate Not Trusted

**Symptom:**
```
Unable to configure HTTPS endpoint. No server certificate was specified...
```

**Solution:**
```bash
dotnet dev-certs https --trust
```

If that doesn't work:
```bash
dotnet dev-certs https --clean
dotnet dev-certs https --trust
```

---

### Issue 3: Package Restore Failed

**Symptom:**
```
error NU1301: Unable to load the service index for source...
```

**Solutions:**

```bash
# Clear NuGet cache
dotnet nuget locals all --clear

# Restore packages
dotnet restore

# If still fails, check nuget.config
```

---

### Issue 4: Startup Class Not Found (Pre-.NET 6)

**Symptom:**
```
Unable to find a Startup type for 'Program'
```

**Solution:**
Ensure `Program.cs` references `Startup`:

```csharp
webBuilder.UseStartup<Startup>();
```

---

### Issue 5: Configuration Not Loading

**Symptom:**
Configuration values are null

**Checklist:**
1. ✅ File named correctly: `appsettings.json` (case-sensitive on Linux)
2. ✅ File copied to output: Right-click → Properties → Copy to Output Directory: "Copy if newer"
3. ✅ JSON syntax valid (use JSON validator)
4. ✅ Section names match exactly (case-sensitive)

---

### Issue 6: Services Not Resolving

**Symptom:**
```
System.InvalidOperationException: Unable to resolve service for type 'IMyService'
```

**Solution:**
Ensure service is registered:

```csharp
// Program.cs
builder.Services.AddScoped<IMyService, MyService>();
```

---

## 8. Best Practices

### ✅ Project Structure

- [ ] Use standard folder conventions (Controllers, Services, Models, etc.)
- [ ] Keep Program.cs clean (extract configuration to extensions)
- [ ] Separate concerns (business logic in services, not controllers)
- [ ] Use DTOs for API contracts (don't expose domain models)

---

### ✅ Configuration

- [ ] Use IOptions<T> for strongly-typed configuration
- [ ] Never hardcode sensitive data (use User Secrets or environment variables)
- [ ] Use environment-specific appsettings files
- [ ] Validate configuration on startup

---

### ✅ Dependency Injection

- [ ] Register services with appropriate lifetime (Transient/Scoped/Singleton)
- [ ] Use interfaces for testability
- [ ] Avoid service locator pattern
- [ ] Don't new up dependencies manually

---

### ✅ Security

- [ ] Always use HTTPS in production
- [ ] Enable HSTS in production
- [ ] Store secrets in User Secrets (dev) or Azure Key Vault (prod)
- [ ] Use environment variables for sensitive config in production
- [ ] Never commit appsettings.Production.json with real secrets

---

### ✅ Performance

- [ ] Enable response compression
- [ ] Use response caching where appropriate
- [ ] Optimize static file serving
- [ ] Use async/await everywhere
- [ ] Profile and monitor your app

---

### ✅ Logging

- [ ] Use ILogger<T> for logging
- [ ] Log at appropriate levels (Debug, Info, Warning, Error)
- [ ] Include correlation IDs for request tracking
- [ ] Use structured logging

---

### ✅ Error Handling

- [ ] Use exception handling middleware
- [ ] Return appropriate HTTP status codes
- [ ] Don't expose stack traces in production
- [ ] Log all errors

---

### ✅ Code Quality

- [ ] Enable nullable reference types
- [ ] Use code analysis (analyzers)
- [ ] Follow naming conventions
- [ ] Write XML documentation for public APIs
- [ ] Use consistent formatting (EditorConfig)

---

# PART 2: TECHNICAL REFERENCE

---

## 9. Important Classes & Interfaces Reference

### WebApplication Class

**Namespace:** `Microsoft.AspNetCore.Builder`

**Purpose:** Represents the configured web application (used in .NET 6+ minimal hosting)

**Full Declaration:**
```csharp
public sealed class WebApplication : IHost, IApplicationBuilder, IEndpointRouteBuilder
```

**Key Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Services` | `IServiceProvider` | Access to dependency injection container |
| `Configuration` | `IConfiguration` | Application configuration |
| `Environment` | `IWebHostEnvironment` | Hosting environment info |
| `Lifetime` | `IHostApplicationLifetime` | Application lifetime events |
| `Logger` | `ILogger` | Logger instance |
| `Urls` | `ICollection<string>` | URLs the server is listening on |

**Key Methods:**

| Method | Return Type | Description |
|--------|-------------|-------------|
| `Run()` | `void` | Runs application and blocks until shutdown |
| `RunAsync()` | `Task` | Runs application asynchronously |
| `Start()` | `void` | Starts application without blocking |
| `StopAsync()` | `Task` | Gracefully stops application |
| `CreateBuilder(args)` | `WebApplicationBuilder` | Creates builder (static) |

**Usage Example:**

```csharp
var app = WebApplication.Create(args);

// Access properties
Console.WriteLine($"Environment: {app.Environment.EnvironmentName}");
Console.WriteLine($"URLs: {string.Join(", ", app.Urls)}");

// Configure middleware
app.MapGet("/", () => "Hello World");

// Run application
app.Run();
```

---

### WebApplicationBuilder Class

**Namespace:** `Microsoft.AspNetCore.Builder`

**Purpose:** Builder for configuring services and the application pipeline

**Full Declaration:**
```csharp
public sealed class WebApplicationBuilder
```

**Key Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Services` | `IServiceCollection` | Service collection for DI |
| `Configuration` | `ConfigurationManager` | Configuration builder |
| `Environment` | `IWebHostEnvironment` | Environment information |
| `Host` | `ConfigureHostBuilder` | Host configuration |
| `WebHost` | `ConfigureWebHostBuilder` | Web host configuration |
| `Logging` | `ILoggingBuilder` | Logging configuration |

**Key Methods:**

| Method | Return Type | Description |
|--------|-------------|-------------|
| `Build()` | `WebApplication` | Builds the web application |

**Usage Example:**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Configure services
builder.Services.AddControllers();
builder.Services.AddScoped<IMyService, MyService>();

// Configure logging
builder.Logging.ClearProviders();
builder.Logging.AddConsole();

// Configure host
builder.Host.ConfigureAppConfiguration(config =>
{
    config.AddJsonFile("custom-settings.json", optional: true);
});

// Configure web host
builder.WebHost.UseUrls("https://localhost:5001");

var app = builder.Build();
```

---

### IWebHostEnvironment Interface

**Namespace:** `Microsoft.AspNetCore.Hosting`

**Purpose:** Provides information about the web hosting environment

**Full Declaration:**
```csharp
public interface IWebHostEnvironment : IHostEnvironment
{
    string WebRootPath { get; set; }
    IFileProvider WebRootFileProvider { get; set; }
}
```

**Inherited from IHostEnvironment:**

| Property | Type | Description |
|----------|------|-------------|
| `EnvironmentName` | `string` | Environment name (Development, Production, etc.) |
| `ApplicationName` | `string` | Application name |
| `ContentRootPath` | `string` | Path to application content files |
| `ContentRootFileProvider` | `IFileProvider` | File provider for content root |

**Extension Methods:**

| Method | Description |
|--------|-------------|
| `IsDevelopment()` | True if environment is "Development" |
| `IsProduction()` | True if environment is "Production" |
| `IsStaging()` | True if environment is "Staging" |
| `IsEnvironment(string env)` | True if environment matches |

**Usage Example:**

```csharp
public class FileService
{
    private readonly IWebHostEnvironment _env;
    
    public FileService(IWebHostEnvironment env)
    {
        _env = env;
    }
    
    public string GetFilePath(string fileName)
    {
        if (_env.IsDevelopment())
        {
            return Path.Combine(_env.ContentRootPath, "TestFiles", fileName);
        }
        
        return Path.Combine(_env.WebRootPath, "uploads", fileName);
    }
    
    public void LogEnvironmentInfo(ILogger logger)
    {
        logger.LogInformation("Environment: {EnvName}", _env.EnvironmentName);
        logger.LogInformation("Application: {AppName}", _env.ApplicationName);
        logger.LogInformation("Content Root: {ContentRoot}", _env.ContentRootPath);
        logger.LogInformation("Web Root: {WebRoot}", _env.WebRootPath);
    }
}
```

---

### IConfiguration Interface

**Namespace:** `Microsoft.Extensions.Configuration`

**Purpose:** Represents configuration from various sources

**Full Declaration:**
```csharp
public interface IConfiguration
{
    string this[string key] { get; set; }
    IEnumerable<IConfigurationSection> GetChildren();
    IChangeToken GetReloadToken();
    IConfigurationSection GetSection(string key);
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `this[string key]` | Get/set value by key |
| `GetSection(string key)` | Get configuration section |
| `GetChildren()` | Get child sections |
| `GetConnectionString(string name)` | Get connection string |
| `GetValue<T>(string key)` | Get strongly-typed value |
| `GetValue<T>(string key, T defaultValue)` | Get value with default |

**Extension Methods:**

| Method | Description |
|--------|-------------|
| `Bind(object instance)` | Bind configuration to object |
| `Get<T>()` | Get strongly-typed configuration |
| `Exists()` | Check if section exists |

**Usage Example:**

```csharp
public class ConfigurationService
{
    private readonly IConfiguration _configuration;
    
    public ConfigurationService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public void Examples()
    {
        // Direct access
        string value1 = _configuration["MyKey"];
        
        // Nested keys (colon separator)
        string value2 = _configuration["Parent:Child:GrandChild"];
        
        // Get section
        var section = _configuration.GetSection("EmailSettings");
        string server = section["SmtpServer"];
        int port = section.GetValue<int>("Port");
        
        // Connection string
        string connString = _configuration.GetConnectionString("DefaultConnection");
        
        // Strongly-typed
        var emailSettings = _configuration
            .GetSection("EmailSettings")
            .Get<EmailSettings>();
        
        // With default value
        int timeout = _configuration.GetValue("Timeout", 30);
        
        // Check existence
        if (_configuration.GetSection("OptionalFeature").Exists())
        {
            // Feature is configured
        }
    }
}
```

---

### ConfigurationManager Class ✨ .NET 6.0+

**Namespace:** `Microsoft.Extensions.Configuration`

**Purpose:** Combines configuration builder and configuration in one

**Full Declaration:**
```csharp
public sealed class ConfigurationManager : 
    IConfigurationBuilder, 
    IConfigurationRoot, 
    IDisposable
```

**Key Methods:**

| Method | Return Type | Description |
|--------|-------------|-------------|
| `AddJsonFile(path)` | `IConfigurationBuilder` | Add JSON configuration file |
| `AddEnvironmentVariables()` | `IConfigurationBuilder` | Add environment variables |
| `AddCommandLine(args)` | `IConfigurationBuilder` | Add command-line args |
| `AddUserSecrets<T>()` | `IConfigurationBuilder` | Add user secrets |
| `Build()` | `IConfigurationRoot` | Not needed, already built |

**Usage Example:**

```csharp
var builder = WebApplication.CreateBuilder(args);

// builder.Configuration is ConfigurationManager
builder.Configuration.AddJsonFile("custom-settings.json", optional: true);
builder.Configuration.AddEnvironmentVariables("MYAPP_");

// Read immediately
string value = builder.Configuration["MyKey"];

var app = builder.Build();
```

---

### IServiceCollection Interface

**Namespace:** `Microsoft.Extensions.DependencyInjection`

**Purpose:** Collection of service descriptors for dependency injection

**Full Declaration:**
```csharp
public interface IServiceCollection : IList<ServiceDescriptor>
{
}
```

**Extension Methods (Common):**

| Method | Description |
|--------|-------------|
| `AddTransient<TService, TImplementation>()` | Register transient service |
| `AddScoped<TService, TImplementation>()` | Register scoped service |
| `AddSingleton<TService, TImplementation>()` | Register singleton service |
| `AddTransient<TService>()` | Register concrete type as transient |
| `Configure<TOptions>(Action<TOptions>)` | Configure options |
| `AddOptions<TOptions>()` | Add options services |

**Usage Example:**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();

// Register with factory
builder.Services.AddScoped<IUserService>(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    var logger = sp.GetRequiredService<ILogger<UserService>>();
    return new UserService(config, logger);
});

// Register with instance
builder.Services.AddSingleton<AppSettings>(new AppSettings
{
    Version = "1.0.0"
});

// Configure options
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection("JwtSettings"));
```

---

### IServiceProvider Interface

**Namespace:** `System`

**Purpose:** Service locator for retrieving services

**Full Declaration:**
```csharp
public interface IServiceProvider
{
    object? GetService(Type serviceType);
}
```

**Extension Methods:**

| Method | Description |
|--------|-------------|
| `GetService<T>()` | Get service (returns null if not found) |
| `GetRequiredService<T>()` | Get service (throws if not found) |
| `GetServices<T>()` | Get all services of type |
| `CreateScope()` | Create new service scope |

**Usage Example:**

```csharp
var app = WebApplication.Create(args);

// Get service at startup (avoid this pattern in general)
using (var scope = app.Services.CreateScope())
{
    var dbContext = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    await dbContext.Database.MigrateAsync();
}

// In middleware or services, use constructor injection instead
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    
    public MyMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Get scoped service from HttpContext
        var service = context.RequestServices.GetRequiredService<IMyService>();
        
        await _next(context);
    }
}
```

---

## 10. Configuration System Deep-Dive

### Configuration Sources

ASP.NET Core loads configuration from multiple sources in order:

```
1. appsettings.json                       (Base configuration)
   ↓
2. appsettings.{Environment}.json        (Environment overrides)
   ↓
3. User Secrets                           (Development only)
   ↓
4. Environment Variables                  (System configuration)
   ↓
5. Command-line Arguments                 (Runtime overrides)
```

**Later sources override earlier sources**

---

### Adding Custom Configuration Sources

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Configuration
    .AddJsonFile("custom-settings.json", optional: true, reloadOnChange: true)
    .AddXmlFile("config.xml", optional: true)
    .AddIniFile("config.ini", optional: true)
    .AddEnvironmentVariables(prefix: "MYAPP_")
    .AddCommandLine(args);

// Azure Key Vault (production)
if (builder.Environment.IsProduction())
{
    builder.Configuration.AddAzureKeyVault(
        new Uri($"https://{keyVaultName}.vault.azure.net/"),
        new DefaultAzureCredential());
}

var app = builder.Build();
```

---

### Configuration Providers

| Provider | Package | Usage |
|----------|---------|-------|
| JSON | Built-in | `AddJsonFile()` |
| Environment Variables | Built-in | `AddEnvironmentVariables()` |
| Command Line | Built-in | `AddCommandLine()` |
| User Secrets | Built-in | `AddUserSecrets<T>()` |
| XML | `Microsoft.Extensions.Configuration.Xml` | `AddXmlFile()` |
| INI | `Microsoft.Extensions.Configuration.Ini` | `AddIniFile()` |
| Azure Key Vault | `Azure.Extensions.AspNetCore.Configuration.Secrets` | `AddAzureKeyVault()` |
| Azure App Configuration | `Microsoft.Extensions.Configuration.AzureAppConfiguration` | `AddAzureAppConfiguration()` |

---

### Strongly-Typed Configuration (IOptions<T>)

**Pattern 1: IOptions<T> (Singleton)**

```csharp
// Options class
public class EmailSettings
{
    public const string SectionName = "EmailSettings";
    
    public string SmtpServer { get; set; } = string.Empty;
    public int Port { get; set; }
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}

// Register
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection(EmailSettings.SectionName));

// Use
public class EmailService
{
    private readonly EmailSettings _settings;
    
    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;  // Evaluated once
    }
}
```

**When to use:** Configuration that doesn't change during application lifetime

---

**Pattern 2: IOptionsSnapshot<T> (Scoped)**

```csharp
// Register (same as IOptions)
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection(EmailSettings.SectionName));

// Use
public class EmailService
{
    private readonly EmailSettings _settings;
    
    public EmailService(IOptionsSnapshot<EmailSettings> options)
    {
        _settings = options.Value;  // Re-evaluated per request
    }
}
```

**When to use:** Configuration that might change between requests

---

**Pattern 3: IOptionsMonitor<T> (Singleton with Live Reload)**

```csharp
// Register (same as IOptions)
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection(EmailSettings.SectionName));

// Use
public class EmailService
{
    private readonly IOptionsMonitor<EmailSettings> _optionsMonitor;
    
    public EmailService(IOptionsMonitor<EmailSettings> optionsMonitor)
    {
        _optionsMonitor = optionsMonitor;
        
        // Subscribe to changes
        _optionsMonitor.OnChange(settings =>
        {
            Console.WriteLine("Email settings changed!");
        });
    }
    
    public void SendEmail()
    {
        var settings = _optionsMonitor.CurrentValue;  // Always current
        // Use settings...
    }
}
```

**When to use:** Configuration that changes at runtime (live reload)

---

### IOptions Comparison Table

| Feature | IOptions<T> | IOptionsSnapshot<T> | IOptionsMonitor<T> |
|---------|-------------|---------------------|-------------------|
| Lifetime | Singleton | Scoped | Singleton |
| Reload | ❌ No | ✅ Per request | ✅ Live |
| DI Support | All | Scoped/Transient only | All |
| Performance | Fastest | Medium | Medium |
| Use Case | Static config | Per-request config | Live config |

---

### Configuration Binding

**Simple Binding:**

```csharp
public class AppSettings
{
    public string ApplicationName { get; set; } = string.Empty;
    public int MaxUsers { get; set; }
    public bool EnableFeatureX { get; set; }
}

// appsettings.json
{
  "ApplicationName": "MyApp",
  "MaxUsers": 100,
  "EnableFeatureX": true
}

// Bind
var appSettings = new AppSettings();
builder.Configuration.Bind(appSettings);

// Or
var appSettings = builder.Configuration.Get<AppSettings>();
```

---

**Complex Binding (Nested Objects):**

```csharp
public class DatabaseSettings
{
    public string ConnectionString { get; set; } = string.Empty;
    public int CommandTimeout { get; set; }
    public RetryPolicy Retry { get; set; } = new();
}

public class RetryPolicy
{
    public int MaxAttempts { get; set; }
    public int DelayMilliseconds { get; set; }
}

// appsettings.json
{
  "Database": {
    "ConnectionString": "Server=localhost;...",
    "CommandTimeout": 30,
    "Retry": {
      "MaxAttempts": 3,
      "DelayMilliseconds": 1000
    }
  }
}

// Bind
var dbSettings = builder.Configuration
    .GetSection("Database")
    .Get<DatabaseSettings>();
```

---

**Array Binding:**

```csharp
// appsettings.json
{
  "AllowedHosts": ["example.com", "test.com", "localhost"]
}

// Bind
var allowedHosts = builder.Configuration
    .GetSection("AllowedHosts")
    .Get<string[]>();

// Or
var allowedHosts = builder.Configuration["AllowedHosts:0"]; // "example.com"
```

---

### Configuration Validation ✨ .NET 6.0+

```csharp
public class EmailSettings
{
    public const string SectionName = "EmailSettings";
    
    [Required]
    [EmailAddress]
    public string FromAddress { get; set; } = string.Empty;
    
    [Required]
    [Range(1, 65535)]
    public int Port { get; set; }
}

// Register with validation
builder.Services.AddOptions<EmailSettings>()
    .Bind(builder.Configuration.GetSection(EmailSettings.SectionName))
    .ValidateDataAnnotations()
    .ValidateOnStart();  // Validate at startup, not first use
```

---

## 11. Hosting Models Details

### Kestrel (Cross-Platform Web Server)

**Default web server** - Written in C#, runs on all platforms

**Basic Configuration:**

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    // Listen on specific port
    options.ListenLocalhost(5000);
    options.ListenLocalhost(5001, listenOptions =>
    {
        listenOptions.UseHttps();
    });
    
    // Configure limits
    options.Limits.MaxConcurrentConnections = 100;
    options.Limits.MaxRequestBodySize = 10 * 1024 * 1024; // 10 MB
    options.Limits.RequestHeadersTimeout = TimeSpan.FromSeconds(30);
});
```

---

### IIS Integration (Windows)

**Hosting Models:**

#### In-Process (Recommended)

```xml
<!-- .csproj -->
<PropertyGroup>
  <AspNetCoreHostingModel>InProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

- Runs inside IIS worker process (w3wp.exe)
- Better performance
- Single process

#### Out-of-Process

```xml
<!-- .csproj -->
<PropertyGroup>
  <AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

- IIS proxies to Kestrel
- Two processes
- Easier debugging

---

### Reverse Proxy Patterns

**Production Setup:**

```
Internet → Nginx/Apache → Kestrel (ASP.NET Core)
```

**Nginx Configuration Example:**

```nginx
server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**ASP.NET Core Configuration:**

```csharp
builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = 
        ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
});

var app = builder.Build();

app.UseForwardedHeaders();
```

---

## 12. Advanced Topics

### Custom Application Lifetime Events

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var lifetime = app.Services.GetRequiredService<IHostApplicationLifetime>();

lifetime.ApplicationStarted.Register(() =>
{
    Console.WriteLine("Application started!");
});

lifetime.ApplicationStopping.Register(() =>
{
    Console.WriteLine("Application is stopping...");
});

lifetime.ApplicationStopped.Register(() =>
{
    Console.WriteLine("Application stopped!");
});

app.Run();
```

---

### Generic Host vs Web Host

**.NET 6+ uses Generic Host** (recommended)

```csharp
// Generic Host (Default in .NET 6+)
var builder = WebApplication.CreateBuilder(args);

// Explicit Generic Host
var builder = Host.CreateDefaultBuilder(args)
    .ConfigureWebHostDefaults(webBuilder =>
    {
        webBuilder.UseStartup<Startup>();
    });
```

**Benefits:**
- Unified hosting model
- Works for web, console, background services
- Better dependency injection

---

### Background Services

```csharp
public class MyBackgroundService : BackgroundService
{
    private readonly ILogger<MyBackgroundService> _logger;
    
    public MyBackgroundService(ILogger<MyBackgroundService> logger)
    {
        _logger = logger;
    }
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Background service running at: {time}", DateTimeOffset.Now);
            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }
}

// Register
builder.Services.AddHostedService<MyBackgroundService>();
```

---

### Minimal APIs ✨ .NET 6.0+

**Simple REST API without controllers:**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var products = new List<Product>
{
    new Product { Id = 1, Name = "Product 1" },
    new Product { Id = 2, Name = "Product 2" }
};

// GET /products
app.MapGet("/products", () => products);

// GET /products/{id}
app.MapGet("/products/{id}", (int id) =>
{
    var product = products.FirstOrDefault(p => p.Id == id);
    return product is not null ? Results.Ok(product) : Results.NotFound();
});

// POST /products
app.MapPost("/products", (Product product) =>
{
    products.Add(product);
    return Results.Created($"/products/{product.Id}", product);
});

// PUT /products/{id}
app.MapPut("/products/{id}", (int id, Product updatedProduct) =>
{
    var product = products.FirstOrDefault(p => p.Id == id);
    if (product is null) return Results.NotFound();
    
    product.Name = updatedProduct.Name;
    return Results.Ok(product);
});

// DELETE /products/{id}
app.MapDelete("/products/{id}", (int id) =>
{
    var product = products.FirstOrDefault(p => p.Id == id);
    if (product is null) return Results.NotFound();
    
    products.Remove(product);
    return Results.NoContent();
});

app.Run();

record Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
}
```

---

### Performance Tips

**1. Use Response Compression**

```csharp
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
});

app.UseResponseCompression();
```

**2. Enable Response Caching**

```csharp
builder.Services.AddResponseCaching();

app.UseResponseCaching();
```

**3. Optimize Static Files**

```csharp
app.UseStaticFiles(new StaticFileOptions
{
    OnPrepareResponse = ctx =>
    {
        // Cache for 1 year
        ctx.Context.Response.Headers.Append(
            "Cache-Control", "public,max-age=31536000");
    }
});
```

**4. Use Output Caching** ✨ .NET 7.0+

```csharp
builder.Services.AddOutputCache();

app.UseOutputCache();

app.MapGet("/cached", () => DateTime.Now)
   .CacheOutput();
```

---

## Summary: Complete Fundamentals Checklist

### ✅ Getting Started
- [ ] Install .NET SDK
- [ ] Choose IDE (Visual Studio, VS Code, or Rider)
- [ ] Create project with appropriate template
- [ ] Trust HTTPS certificate
- [ ] Run and test application

### ✅ Project Structure
- [ ] Understand Program.cs entry point
- [ ] Configure appsettings.json
- [ ] Set up environment-specific config files
- [ ] Organize folders (Controllers, Services, Models, etc.)
- [ ] Configure static files (wwwroot)

### ✅ Configuration
- [ ] Use strongly-typed configuration (IOptions<T>)
- [ ] Set up User Secrets for development
- [ ] Use environment variables in production
- [ ] Understand configuration priority order
- [ ] Validate configuration on startup

### ✅ Dependency Injection
- [ ] Register services with appropriate lifetime
- [ ] Use constructor injection
- [ ] Create extension methods for clean registration
- [ ] Avoid service locator pattern

### ✅ Environments
- [ ] Set ASPNETCORE_ENVIRONMENT variable
- [ ] Create environment-specific appsettings files
- [ ] Use environment checks in code
- [ ] Configure different behaviors per environment

### ✅ Best Practices
- [ ] Use .NET 8.0 (LTS) for new projects
- [ ] Follow minimal hosting model (.NET 6+)
- [ ] Enable nullable reference types
- [ ] Use async/await throughout
- [ ] Implement proper error handling
- [ ] Add comprehensive logging

---

**This completes the ASP.NET Core Fundamentals & Project Structure guide combining practical hands-on content with deep technical reference!**
