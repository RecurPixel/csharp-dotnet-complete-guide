# ASP.NET Core Authentication & Authorization - Complete Guide
## Practical Guide + Technical Reference

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. Authentication vs Authorization Explained
2. 3 Primary Authentication Methods (Cookie, JWT, Identity)
3. Cookie Authentication Deep Dive
4. JWT Authentication Deep Dive
5. ASP.NET Core Identity Setup
6. Authorization Patterns (Role, Claims, Policy)
7. External Authentication Providers (OAuth/OpenID)
8. Additional Authentication Methods (API Key, Certificate)
9. Common Security Patterns
10. Troubleshooting Common Issues
11. Best Practices & Security Checklist

### Part 2: Technical Reference (Deep Dive)
12. Important Interfaces & Classes Reference
13. Configuration Deep-Dive
14. Advanced Topics
15. Version Timeline

---

# PART 1: PRACTICAL GUIDE

---

## 1. Authentication vs Authorization Explained

### Simple Definitions

**Authentication:** "Who are you?"
- Verifying user identity
- Think: Showing your ID at airport security

**Authorization:** "What can you do?"
- Checking permissions/access rights
- Think: Having a boarding pass for first class

### Visual Flow

```
1. User sends credentials
   ↓
2. Authentication validates identity → Creates Principal/Claims
   ↓
3. User makes request to protected resource
   ↓
4. Authorization checks permissions
   ↓
5. Allow or Deny access
```

### Key Differences

| Aspect | Authentication | Authorization |
|--------|---------------|---------------|
| **Purpose** | Verify identity | Check permissions |
| **When** | Happens first | Happens after authentication |
| **Result** | Identity (ClaimsPrincipal) | Allow/Deny access |
| **Example** | Login with username/password | [Authorize(Roles = "Admin")] |
| **Middleware** | `UseAuthentication()` | `UseAuthorization()` |

---

## 2. 3 Primary Authentication Methods

### Quick Comparison

| Method | Best For | Complexity | Stateless | Mobile-Friendly |
|--------|----------|------------|-----------|-----------------|
| **Cookie** | Traditional web apps | Simple | ❌ No | ❌ No |
| **JWT** | Modern APIs, SPAs | Medium | ✅ Yes | ✅ Yes |
| **Identity** | Full-featured apps | High | Depends | ✅ Yes |

### When to Use Which?

**Use Cookie Authentication when:**
- ✅ Building traditional MVC/Razor Pages app
- ✅ Users log in through web browser
- ✅ Session-based state is acceptable
- ✅ Simple authentication needs

**Use JWT Authentication when:**
- ✅ Building RESTful APIs
- ✅ Need stateless authentication
- ✅ Mobile app or SPA frontend
- ✅ Microservices architecture
- ✅ Cross-domain authentication

**Use Identity Framework when:**
- ✅ Need full user management (register, login, roles, etc.)
- ✅ Want built-in features (email confirmation, password reset, 2FA)
- ✅ Enterprise applications
- ✅ Don't want to build auth from scratch

---

## 3. Cookie Authentication Deep Dive

### Method 1: Cookie Authentication (Traditional Web Apps)

**When to use:**
- ✅ Server-rendered views (MVC/Razor Pages)
- ✅ Browser-based authentication
- ✅ Session management needed
- ❌ Not ideal for APIs
- ❌ Not mobile-friendly

### Step-by-Step Setup

**Step 1: Configure Cookie Authentication**

```csharp
using Microsoft.AspNetCore.Authentication.Cookies;

var builder = WebApplication.CreateBuilder(args);

// Add authentication services
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.LogoutPath = "/Account/Logout";
        options.AccessDeniedPath = "/Account/AccessDenied";
        options.ExpireTimeSpan = TimeSpan.FromHours(1);
        options.SlidingExpiration = true;
        options.Cookie.HttpOnly = true; // Security: Prevent JavaScript access
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always; // HTTPS only
        options.Cookie.SameSite = SameSiteMode.Strict; // CSRF protection
    });

builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

// ⚠️ Order matters!
app.UseAuthentication(); // Must be before Authorization
app.UseAuthorization();

app.MapControllers();

app.Run();
```

**Step 2: Create Login Action**

```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using System.Security.Claims;

public class AccountController : Controller
{
    // GET: /Account/Login
    public IActionResult Login(string returnUrl = "/")
    {
        ViewData["ReturnUrl"] = returnUrl;
        return View();
    }

    // POST: /Account/Login
    [HttpPost]
    public async Task<IActionResult> Login(LoginModel model, string returnUrl = "/")
    {
        // Step 1: Validate credentials (check database, etc.)
        if (!ValidateUser(model.Username, model.Password))
        {
            ModelState.AddModelError("", "Invalid username or password");
            return View(model);
        }

        // Step 2: Create claims (user information)
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, model.Username),
            new Claim(ClaimTypes.Email, "user@example.com"),
            new Claim(ClaimTypes.Role, "User"),
            new Claim("EmployeeId", "12345") // Custom claim
        };

        // Step 3: Create claims identity
        var claimsIdentity = new ClaimsIdentity(
            claims,
            CookieAuthenticationDefaults.AuthenticationScheme);

        // Step 4: Create authentication properties
        var authProperties = new AuthenticationProperties
        {
            IsPersistent = model.RememberMe, // "Remember me" checkbox
            ExpiresUtc = DateTimeOffset.UtcNow.AddHours(1)
        };

        // Step 5: Sign in (creates encrypted cookie)
        await HttpContext.SignInAsync(
            CookieAuthenticationDefaults.AuthenticationScheme,
            new ClaimsPrincipal(claimsIdentity),
            authProperties);

        return LocalRedirect(returnUrl);
    }

    private bool ValidateUser(string username, string password)
    {
        // TODO: Check against database
        // NEVER store plain text passwords!
        // Use password hashing (see Identity section)
        return username == "demo" && password == "password";
    }
}

public class LoginModel
{
    public string Username { get; set; }
    public string Password { get; set; }
    public bool RememberMe { get; set; }
}
```

**Step 3: Create Logout Action**

```csharp
[HttpPost]
public async Task<IActionResult> Logout()
{
    await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
    return RedirectToAction("Index", "Home");
}
```

**Step 4: Protect Actions**

```csharp
using Microsoft.AspNetCore.Authorization;

[Authorize] // Requires authentication
public class DashboardController : Controller
{
    public IActionResult Index()
    {
        // Access user information
        var username = User.Identity.Name;
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        
        return View();
    }
}

[Authorize(Roles = "Admin")] // Requires Admin role
public IActionResult AdminPanel()
{
    return View();
}

[AllowAnonymous] // Allow anonymous access (overrides [Authorize] on controller)
public IActionResult PublicPage()
{
    return View();
}
```

### Complete Login View Example

```html
@model LoginModel
@{
    var returnUrl = ViewData["ReturnUrl"] as string;
}

<h2>Login</h2>

<form asp-action="Login" asp-route-returnUrl="@returnUrl" method="post">
    <div>
        <label asp-for="Username"></label>
        <input asp-for="Username" />
        <span asp-validation-for="Username"></span>
    </div>
    
    <div>
        <label asp-for="Password"></label>
        <input asp-for="Password" type="password" />
        <span asp-validation-for="Password"></span>
    </div>
    
    <div>
        <input asp-for="RememberMe" type="checkbox" />
        <label asp-for="RememberMe">Remember me</label>
    </div>
    
    <div asp-validation-summary="All"></div>
    
    <button type="submit">Login</button>
</form>
```

---

## 4. JWT Authentication Deep Dive

### Method 2: JWT (JSON Web Token) - Modern API Authentication

**When to use:**
- ✅ RESTful APIs
- ✅ Single Page Applications (SPAs)
- ✅ Mobile applications
- ✅ Microservices
- ✅ Stateless authentication
- ❌ Traditional web apps (use cookies instead)

### What is JWT?

**JWT Structure:** `header.payload.signature`

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Decoded:**
```json
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload (Claims)
{
  "sub": "1234567890",
  "name": "John Doe",
  "email": "john@example.com",
  "role": "Admin",
  "exp": 1516239022
}

// Signature (verifies token hasn't been tampered with)
```

### Step-by-Step JWT Setup

**Step 1: Install Required Package**

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

**Step 2: Add JWT Settings to appsettings.json**

```json
{
  "Jwt": {
    "Issuer": "https://yourapp.com",
    "Audience": "https://yourapp.com",
    "Key": "YourSuperSecretKeyThatIsAtLeast32CharactersLong!!",
    "ExpiryMinutes": 60
  }
}
```

⚠️ **NEVER** commit the secret key to source control! Use User Secrets (development) or Azure Key Vault (production).

**Step 3: Configure JWT Authentication**

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Read JWT settings
var jwtSettings = builder.Configuration.GetSection("Jwt");
var secretKey = jwtSettings["Key"];

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwtSettings["Issuer"],
            ValidAudience = jwtSettings["Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(secretKey)),
            ClockSkew = TimeSpan.Zero // Remove default 5 min tolerance
        };

        // Optional: Handle JWT events
        options.Events = new JwtBearerEvents
        {
            OnAuthenticationFailed = context =>
            {
                Console.WriteLine($"Authentication failed: {context.Exception.Message}");
                return Task.CompletedTask;
            },
            OnTokenValidated = context =>
            {
                Console.WriteLine($"Token validated for: {context.Principal.Identity.Name}");
                return Task.CompletedTask;
            }
        };
    });

builder.Services.AddAuthorization();
builder.Services.AddControllers();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthentication(); // Before Authorization!
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Step 4: Create JWT Token Service**

```csharp
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

public interface ITokenService
{
    string GenerateToken(string userId, string username, List<string> roles);
}

public class TokenService : ITokenService
{
    private readonly IConfiguration _configuration;

    public TokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateToken(string userId, string username, List<string> roles)
    {
        var jwtSettings = _configuration.GetSection("Jwt");
        var secretKey = jwtSettings["Key"];
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secretKey));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        // Create claims
        var claims = new List<Claim>
        {
            new Claim(JwtRegisteredClaimNames.Sub, userId),
            new Claim(JwtRegisteredClaimNames.Name, username),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()), // Unique token ID
            new Claim(JwtRegisteredClaimNames.Iat, 
                DateTimeOffset.UtcNow.ToUnixTimeSeconds().ToString(), 
                ClaimValueTypes.Integer64)
        };

        // Add roles as claims
        foreach (var role in roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }

        // Create token descriptor
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.UtcNow.AddMinutes(
                int.Parse(jwtSettings["ExpiryMinutes"])),
            Issuer = jwtSettings["Issuer"],
            Audience = jwtSettings["Audience"],
            SigningCredentials = credentials
        };

        var tokenHandler = new JwtSecurityTokenHandler();
        var token = tokenHandler.CreateToken(tokenDescriptor);

        return tokenHandler.WriteToken(token);
    }
}

// Register in Program.cs
builder.Services.AddScoped<ITokenService, TokenService>();
```

**Step 5: Create Login Endpoint (API)**

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly ITokenService _tokenService;
    // In real app, inject UserManager or your user service

    public AuthController(ITokenService tokenService)
    {
        _tokenService = tokenService;
    }

    [HttpPost("login")]
    public IActionResult Login([FromBody] LoginRequest request)
    {
        // Step 1: Validate credentials
        // TODO: Check against database with hashed password
        if (request.Username != "demo" || request.Password != "password")
        {
            return Unauthorized(new { message = "Invalid credentials" });
        }

        // Step 2: Get user details (from database)
        var userId = "user-123";
        var username = request.Username;
        var roles = new List<string> { "User", "Admin" };

        // Step 3: Generate JWT token
        var token = _tokenService.GenerateToken(userId, username, roles);

        // Step 4: Return token to client
        return Ok(new LoginResponse
        {
            Token = token,
            Username = username,
            ExpiresIn = 3600 // seconds
        });
    }

    [Authorize] // Requires valid JWT token
    [HttpGet("profile")]
    public IActionResult GetProfile()
    {
        // Access claims from token
        var userId = User.FindFirst(JwtRegisteredClaimNames.Sub)?.Value;
        var username = User.Identity.Name;
        var roles = User.FindAll(ClaimTypes.Role).Select(c => c.Value).ToList();

        return Ok(new
        {
            UserId = userId,
            Username = username,
            Roles = roles
        });
    }

    [Authorize(Roles = "Admin")] // Requires Admin role
    [HttpGet("admin")]
    public IActionResult AdminOnly()
    {
        return Ok(new { message = "Admin access granted" });
    }
}

public class LoginRequest
{
    public string Username { get; set; }
    public string Password { get; set; }
}

public class LoginResponse
{
    public string Token { get; set; }
    public string Username { get; set; }
    public int ExpiresIn { get; set; }
}
```

### Client-Side JWT Usage

**JavaScript Example (Fetch API):**

```javascript
// Login and get token
async function login() {
    const response = await fetch('https://api.example.com/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            username: 'demo',
            password: 'password'
        })
    });
    
    const data = await response.json();
    localStorage.setItem('token', data.token); // Store token
}

// Use token in subsequent requests
async function getProfile() {
    const token = localStorage.getItem('token');
    
    const response = await fetch('https://api.example.com/api/auth/profile', {
        headers: {
            'Authorization': `Bearer ${token}` // Add token to header
        }
    });
    
    const profile = await response.json();
    console.log(profile);
}
```

### JWT Best Practices

1. ✅ **Use HTTPS** - Always encrypt traffic
2. ✅ **Short expiration** - 15-60 minutes recommended
3. ✅ **Implement refresh tokens** - For long-lived sessions
4. ✅ **Validate all parameters** - issuer, audience, expiry
5. ✅ **Use strong secret key** - At least 256 bits
6. ✅ **Store tokens securely** - HttpOnly cookies or secure storage
7. ❌ **Don't store sensitive data** - Tokens are base64 encoded, not encrypted
8. ❌ **Don't trust token contents** - Always validate signature

---

## 5. ASP.NET Core Identity Setup

### Method 3: ASP.NET Core Identity - Full-Featured Authentication

**When to use:**
- ✅ Need complete user management system
- ✅ Want built-in features (register, login, roles, email confirmation, 2FA)
- ✅ Enterprise applications
- ✅ Don't want to implement auth from scratch
- ❌ Overkill for simple authentication needs

**Features included:**
- User registration & login
- Password hashing & validation
- Role management
- Claims management
- Email confirmation
- Password reset
- Two-factor authentication (2FA)
- Account lockout
- External login providers (Google, Facebook, etc.)

### Step-by-Step Identity Setup

**Step 1: Install Packages**

```bash
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

**Step 2: Create ApplicationUser (Custom User)**

```csharp
using Microsoft.AspNetCore.Identity;

public class ApplicationUser : IdentityUser
{
    // Add custom properties
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime DateOfBirth { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}
```

**Step 3: Create ApplicationDbContext**

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    // Add your DbSets here
    public DbSet<Product> Products { get; set; }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder); // IMPORTANT: Call base method!

        // Customize Identity tables if needed
        builder.Entity<ApplicationUser>(entity =>
        {
            entity.Property(e => e.FirstName).HasMaxLength(100);
            entity.Property(e => e.LastName).HasMaxLength(100);
        });
    }
}
```

**Step 4: Configure Services in Program.cs**

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add DbContext
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

// Add Identity
builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    // Password settings
    options.Password.RequireDigit = true;
    options.Password.RequireLowercase = true;
    options.Password.RequireUppercase = true;
    options.Password.RequireNonAlphanumeric = true;
    options.Password.RequiredLength = 8;
    options.Password.RequiredUniqueChars = 1;

    // Lockout settings
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5);
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.AllowedForNewUsers = true;

    // User settings
    options.User.RequireUniqueEmail = true;
    
    // Sign-in settings
    options.SignIn.RequireConfirmedEmail = false; // Set to true in production!
    options.SignIn.RequireConfirmedPhoneNumber = false;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders(); // For password reset, email confirmation, etc.

// Configure cookie settings
builder.Services.ConfigureApplicationCookie(options =>
{
    options.LoginPath = "/Account/Login";
    options.LogoutPath = "/Account/Logout";
    options.AccessDeniedPath = "/Account/AccessDenied";
    options.ExpireTimeSpan = TimeSpan.FromHours(2);
    options.SlidingExpiration = true;
    options.Cookie.HttpOnly = true;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});

builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseAuthentication(); // Must be before Authorization
app.UseAuthorization();

app.MapControllers();

app.Run();
```

**Step 5: Add Connection String to appsettings.json**

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MyAppDb;Trusted_Connection=true;MultipleActiveResultSets=true"
  }
}
```

**Step 6: Create and Apply Migrations**

```bash
dotnet ef migrations add InitialIdentity
dotnet ef database update
```

This creates tables:
- AspNetUsers
- AspNetRoles
- AspNetUserRoles
- AspNetUserClaims
- AspNetUserLogins
- AspNetUserTokens
- AspNetRoleClaims

**Step 7: Create AccountController (Registration & Login)**

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;

public class AccountController : Controller
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;

    public AccountController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager)
    {
        _userManager = userManager;
        _signInManager = signInManager;
    }

    // GET: /Account/Register
    [HttpGet]
    public IActionResult Register()
    {
        return View();
    }

    // POST: /Account/Register
    [HttpPost]
    public async Task<IActionResult> Register(RegisterModel model)
    {
        if (!ModelState.IsValid)
            return View(model);

        // Create user
        var user = new ApplicationUser
        {
            UserName = model.Email,
            Email = model.Email,
            FirstName = model.FirstName,
            LastName = model.LastName,
            DateOfBirth = model.DateOfBirth
        };

        var result = await _userManager.CreateAsync(user, model.Password);

        if (result.Succeeded)
        {
            // Assign default role
            await _userManager.AddToRoleAsync(user, "User");

            // Sign in automatically
            await _signInManager.SignInAsync(user, isPersistent: false);

            // Optional: Send confirmation email
            // var token = await _userManager.GenerateEmailConfirmationTokenAsync(user);
            // Send email with token...

            return RedirectToAction("Index", "Home");
        }

        // Add errors to ModelState
        foreach (var error in result.Errors)
        {
            ModelState.AddModelError(string.Empty, error.Description);
        }

        return View(model);
    }

    // GET: /Account/Login
    [HttpGet]
    public IActionResult Login(string returnUrl = null)
    {
        ViewData["ReturnUrl"] = returnUrl;
        return View();
    }

    // POST: /Account/Login
    [HttpPost]
    public async Task<IActionResult> Login(LoginModel model, string returnUrl = null)
    {
        if (!ModelState.IsValid)
            return View(model);

        var result = await _signInManager.PasswordSignInAsync(
            model.Email,
            model.Password,
            model.RememberMe,
            lockoutOnFailure: true); // Lock account after failed attempts

        if (result.Succeeded)
        {
            return LocalRedirect(returnUrl ?? "/");
        }

        if (result.IsLockedOut)
        {
            ModelState.AddModelError("", "Account locked. Try again in 5 minutes.");
            return View(model);
        }

        if (result.RequiresTwoFactor)
        {
            return RedirectToAction("LoginWith2fa", new { returnUrl, model.RememberMe });
        }

        ModelState.AddModelError("", "Invalid login attempt");
        return View(model);
    }

    // POST: /Account/Logout
    [HttpPost]
    public async Task<IActionResult> Logout()
    {
        await _signInManager.SignOutAsync();
        return RedirectToAction("Index", "Home");
    }
}

public class RegisterModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    public string FirstName { get; set; }

    [Required]
    public string LastName { get; set; }

    [Required]
    [DataType(DataType.Date)]
    public DateTime DateOfBirth { get; set; }

    [Required]
    [StringLength(100, MinimumLength = 8)]
    [DataType(DataType.Password)]
    public string Password { get; set; }

    [Required]
    [Compare("Password")]
    [DataType(DataType.Password)]
    public string ConfirmPassword { get; set; }
}

public class LoginModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [DataType(DataType.Password)]
    public string Password { get; set; }

    public bool RememberMe { get; set; }
}
```

**Step 8: Seed Roles (Optional but Recommended)**

```csharp
public static class SeedData
{
    public static async Task Initialize(IServiceProvider serviceProvider)
    {
        var roleManager = serviceProvider.GetRequiredService<RoleManager<IdentityRole>>();
        var userManager = serviceProvider.GetRequiredService<UserManager<ApplicationUser>>();

        // Create roles
        string[] roleNames = { "Admin", "Manager", "User" };
        foreach (var roleName in roleNames)
        {
            if (!await roleManager.RoleExistsAsync(roleName))
            {
                await roleManager.CreateAsync(new IdentityRole(roleName));
            }
        }

        // Create admin user
        var adminEmail = "admin@example.com";
        var adminUser = await userManager.FindByEmailAsync(adminEmail);

        if (adminUser == null)
        {
            adminUser = new ApplicationUser
            {
                UserName = adminEmail,
                Email = adminEmail,
                FirstName = "Admin",
                LastName = "User",
                EmailConfirmed = true
            };

            await userManager.CreateAsync(adminUser, "Admin@123!");
            await userManager.AddToRoleAsync(adminUser, "Admin");
        }
    }
}

// Call in Program.cs after app.Build()
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    await SeedData.Initialize(services);
}
```

### Identity API Endpoints (For SPAs/Mobile)

```csharp
[ApiController]
[Route("api/[controller]")]
public class IdentityAuthController : ControllerBase
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly ITokenService _tokenService; // Your JWT service

    public IdentityAuthController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager,
        ITokenService tokenService)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _tokenService = tokenService;
    }

    [HttpPost("register")]
    public async Task<IActionResult> Register([FromBody] RegisterRequest request)
    {
        var user = new ApplicationUser
        {
            UserName = request.Email,
            Email = request.Email,
            FirstName = request.FirstName,
            LastName = request.LastName
        };

        var result = await _userManager.CreateAsync(user, request.Password);

        if (!result.Succeeded)
        {
            return BadRequest(result.Errors);
        }

        await _userManager.AddToRoleAsync(user, "User");

        return Ok(new { message = "Registration successful" });
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        var user = await _userManager.FindByEmailAsync(request.Email);
        if (user == null)
        {
            return Unauthorized(new { message = "Invalid credentials" });
        }

        var result = await _signInManager.CheckPasswordSignInAsync(
            user, request.Password, lockoutOnFailure: true);

        if (!result.Succeeded)
        {
            return Unauthorized(new { message = "Invalid credentials" });
        }

        // Get user roles
        var roles = await _userManager.GetRolesAsync(user);

        // Generate JWT token
        var token = _tokenService.GenerateToken(user.Id, user.UserName, roles.ToList());

        return Ok(new
        {
            token,
            username = user.UserName,
            roles
        });
    }

    [Authorize]
    [HttpPost("change-password")]
    public async Task<IActionResult> ChangePassword([FromBody] ChangePasswordRequest request)
    {
        var user = await _userManager.GetUserAsync(User);
        if (user == null)
        {
            return Unauthorized();
        }

        var result = await _userManager.ChangePasswordAsync(
            user, request.OldPassword, request.NewPassword);

        if (!result.Succeeded)
        {
            return BadRequest(result.Errors);
        }

        return Ok(new { message = "Password changed successfully" });
    }
}
```

---

## 6. Authorization Patterns (Role, Claims, Policy)

### Pattern 1: Role-Based Authorization (Simple)

**When to use:**
- ✅ Simple permission model
- ✅ Users belong to predefined roles
- ❌ Complex permission requirements

```csharp
// Single role
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly()
{
    return View();
}

// Multiple roles (OR logic - user needs ANY of these roles)
[Authorize(Roles = "Admin,Manager")]
public IActionResult ManagerOrAdmin()
{
    return View();
}

// Check role in code
if (User.IsInRole("Admin"))
{
    // Admin-specific logic
}

// Assign roles with Identity
await _userManager.AddToRoleAsync(user, "Admin");
await _userManager.AddToRolesAsync(user, new[] { "Admin", "Manager" });

// Remove role
await _userManager.RemoveFromRoleAsync(user, "Manager");
```

### Pattern 2: Claims-Based Authorization (Flexible)

**When to use:**
- ✅ Fine-grained permissions
- ✅ User attributes beyond roles
- ✅ Modern applications

**What are Claims?**
- Key-value pairs describing the user
- Examples: Name, Email, Age, Department, Permissions

```csharp
// Add claims to user (with Identity)
var claims = new List<Claim>
{
    new Claim("Department", "IT"),
    new Claim("EmployeeId", "12345"),
    new Claim("SecurityClearance", "Level3"),
    new Claim("CanApproveExpenses", "true")
};

await _userManager.AddClaimsAsync(user, claims);

// Read claims
var department = User.FindFirst("Department")?.Value;
var canApprove = User.HasClaim("CanApproveExpenses", "true");

// Protect with claims
[Authorize(Policy = "RequireIT")]
public IActionResult ITOnly()
{
    return View();
}

// Configure policy in Program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireIT", policy =>
        policy.RequireClaim("Department", "IT"));
    
    options.AddPolicy("HighSecurityClearance", policy =>
        policy.RequireClaim("SecurityClearance", "Level3", "Level4"));
});
```

### Pattern 3: Policy-Based Authorization (Recommended - Most Powerful)

**When to use:**
- ✅ Complex authorization logic
- ✅ Reusable authorization rules
- ✅ Business logic in authorization
- ✅ Modern best practice

**Step 1: Create Authorization Requirement**

```csharp
using Microsoft.AspNetCore.Authorization;

// Requirement: User must be over 18
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }

    public MinimumAgeRequirement(int minimumAge)
    {
        MinimumAge = minimumAge;
    }
}

// Requirement: User must be in specific department
public class DepartmentRequirement : IAuthorizationRequirement
{
    public string Department { get; }

    public DepartmentRequirement(string department)
    {
        Department = department;
    }
}
```

**Step 2: Create Authorization Handler**

```csharp
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        // Get date of birth claim
        var dobClaim = context.User.FindFirst(c => c.Type == ClaimTypes.DateOfBirth);

        if (dobClaim == null)
        {
            return Task.CompletedTask; // Requirement not met
        }

        var dateOfBirth = DateTime.Parse(dobClaim.Value);
        var age = DateTime.Today.Year - dateOfBirth.Year;

        if (dateOfBirth.Date > DateTime.Today.AddYears(-age))
        {
            age--; // Adjust for birthday not yet occurred this year
        }

        if (age >= requirement.MinimumAge)
        {
            context.Succeed(requirement); // Requirement met!
        }

        return Task.CompletedTask;
    }
}

public class DepartmentHandler : AuthorizationHandler<DepartmentRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        DepartmentRequirement requirement)
    {
        var departmentClaim = context.User.FindFirst("Department");

        if (departmentClaim != null && 
            departmentClaim.Value.Equals(requirement.Department, 
                StringComparison.OrdinalIgnoreCase))
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

**Step 3: Register Handlers and Policies**

```csharp
builder.Services.AddAuthorization(options =>
{
    // Simple policies
    options.AddPolicy("MustBeOver18", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(18)));

    options.AddPolicy("MustBeOver21", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(21)));

    options.AddPolicy("RequireITDepartment", policy =>
        policy.Requirements.Add(new DepartmentRequirement("IT")));

    // Combined requirements (AND logic)
    options.AddPolicy("AdminOver21", policy =>
    {
        policy.RequireRole("Admin");
        policy.Requirements.Add(new MinimumAgeRequirement(21));
    });

    // OR logic (multiple policies)
    options.AddPolicy("ManagerOrAdmin", policy =>
        policy.RequireRole("Manager", "Admin"));
});

// Register handlers
builder.Services.AddScoped<IAuthorizationHandler, MinimumAgeHandler>();
builder.Services.AddScoped<IAuthorizationHandler, DepartmentHandler>();
```

**Step 4: Use Policies**

```csharp
[Authorize(Policy = "MustBeOver18")]
public IActionResult AdultContent()
{
    return View();
}

[Authorize(Policy = "RequireITDepartment")]
public IActionResult ITDashboard()
{
    return View();
}

[Authorize(Policy = "AdminOver21")]
public IActionResult SensitiveAdminArea()
{
    return View();
}

// Check policy in code
var authService = HttpContext.RequestServices
    .GetRequiredService<IAuthorizationService>();

var result = await authService.AuthorizeAsync(User, "MustBeOver18");

if (result.Succeeded)
{
    // User meets policy requirements
}
```

### Resource-Based Authorization

**When to use:**
- ✅ Authorization depends on the resource being accessed
- ✅ Example: Users can only edit their own posts

```csharp
// Requirement
public class SameAuthorRequirement : IAuthorizationRequirement { }

// Handler
public class SameAuthorHandler : AuthorizationHandler<SameAuthorRequirement, BlogPost>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        SameAuthorRequirement requirement,
        BlogPost resource)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

        if (resource.AuthorId == userId)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}

// Usage in controller
[HttpPut("{id}")]
public async Task<IActionResult> UpdatePost(int id, BlogPost updatedPost)
{
    var post = await _context.Posts.FindAsync(id);
    
    var authResult = await _authorizationService.AuthorizeAsync(
        User, post, "SameAuthor");
    
    if (!authResult.Succeeded)
    {
        return Forbid(); // 403 Forbidden
    }
    
    // Update post...
    return Ok(post);
}

// Register
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("SameAuthor", policy =>
        policy.Requirements.Add(new SameAuthorRequirement()));
});
builder.Services.AddScoped<IAuthorizationHandler, SameAuthorHandler>();
```

---

## 7. External Authentication Providers (OAuth/OpenID Connect)

### Method 4: External Providers (Google, Facebook, Microsoft)

**When to use:**
- ✅ Social login ("Sign in with Google")
- ✅ Reduce password management burden
- ✅ Improve user experience
- ✅ Enterprise SSO (Single Sign-On)

### Google Authentication Setup

**Step 1: Get Google OAuth Credentials**

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create project
3. Enable Google+ API
4. Create OAuth 2.0 Client ID
5. Add authorized redirect URI: `https://localhost:5001/signin-google`
6. Copy Client ID and Client Secret

**Step 2: Install Package**

```bash
dotnet add package Microsoft.AspNetCore.Authentication.Google
```

**Step 3: Add to appsettings.json**

```json
{
  "Authentication": {
    "Google": {
      "ClientId": "your-client-id.apps.googleusercontent.com",
      "ClientSecret": "your-client-secret"
    }
  }
}
```

⚠️ Use User Secrets for development:
```bash
dotnet user-secrets set "Authentication:Google:ClientId" "your-client-id"
dotnet user-secrets set "Authentication:Google:ClientSecret" "your-secret"
```

**Step 4: Configure Google Authentication**

```csharp
builder.Services.AddAuthentication()
    .AddGoogle(options =>
    {
        options.ClientId = builder.Configuration["Authentication:Google:ClientId"];
        options.ClientSecret = builder.Configuration["Authentication:Google:ClientSecret"];
        
        // Optional: Request additional scopes
        options.Scope.Add("profile");
        options.Scope.Add("email");
        
        // Optional: Save tokens for API calls
        options.SaveTokens = true;
        
        // Optional: Custom events
        options.Events.OnCreatingTicket = context =>
        {
            // Access user info from Google
            var email = context.Principal.FindFirst(ClaimTypes.Email)?.Value;
            // Save to database, etc.
            return Task.CompletedTask;
        };
    });
```

**Step 5: Add External Login UI**

```csharp
public class AccountController : Controller
{
    private readonly SignInManager<ApplicationUser> _signInManager;

    [HttpPost]
    public IActionResult ExternalLogin(string provider, string returnUrl = null)
    {
        var redirectUrl = Url.Action("ExternalLoginCallback", "Account", 
            new { returnUrl });
        
        var properties = _signInManager.ConfigureExternalAuthenticationProperties(
            provider, redirectUrl);
        
        return Challenge(properties, provider);
    }

    [HttpGet]
    public async Task<IActionResult> ExternalLoginCallback(string returnUrl = null)
    {
        var info = await _signInManager.GetExternalLoginInfoAsync();
        if (info == null)
        {
            return RedirectToAction("Login");
        }

        // Sign in with external provider
        var result = await _signInManager.ExternalLoginSignInAsync(
            info.LoginProvider, info.ProviderKey, isPersistent: false);

        if (result.Succeeded)
        {
            return LocalRedirect(returnUrl ?? "/");
        }

        // User doesn't exist - create account
        var email = info.Principal.FindFirstValue(ClaimTypes.Email);
        var user = new ApplicationUser
        {
            UserName = email,
            Email = email
        };

        var createResult = await _userManager.CreateAsync(user);
        if (createResult.Succeeded)
        {
            await _userManager.AddLoginAsync(user, info);
            await _signInManager.SignInAsync(user, isPersistent: false);
            return LocalRedirect(returnUrl ?? "/");
        }

        return RedirectToAction("Login");
    }
}
```

**Login View (Razor):**

```html
<h2>Login</h2>

<!-- Regular login form -->
<form asp-action="Login" method="post">
    <!-- Email, Password fields... -->
    <button type="submit">Login</button>
</form>

<!-- External logins -->
<h3>Or sign in with:</h3>
<form asp-action="ExternalLogin" method="post">
    <button type="submit" name="provider" value="Google">
        Google
    </button>
</form>
```

### Multiple External Providers

```csharp
builder.Services.AddAuthentication()
    .AddGoogle(options =>
    {
        options.ClientId = builder.Configuration["Authentication:Google:ClientId"];
        options.ClientSecret = builder.Configuration["Authentication:Google:ClientSecret"];
    })
    .AddFacebook(options =>
    {
        options.AppId = builder.Configuration["Authentication:Facebook:AppId"];
        options.AppSecret = builder.Configuration["Authentication:Facebook:AppSecret"];
    })
    .AddMicrosoftAccount(options =>
    {
        options.ClientId = builder.Configuration["Authentication:Microsoft:ClientId"];
        options.ClientSecret = builder.Configuration["Authentication:Microsoft:ClientSecret"];
    })
    .AddTwitter(options =>
    {
        options.ConsumerKey = builder.Configuration["Authentication:Twitter:ConsumerKey"];
        options.ConsumerSecret = builder.Configuration["Authentication:Twitter:ConsumerSecret"];
    });
```

---

## 8. Additional Authentication Methods

### Method 5: API Key Authentication

**When to use:**
- ✅ Machine-to-machine communication
- ✅ Simple API access control
- ✅ Third-party integrations
- ❌ User-facing applications (use JWT instead)

**Implementation:**

```csharp
// API Key authentication handler
using Microsoft.AspNetCore.Authentication;
using Microsoft.Extensions.Options;
using System.Security.Claims;
using System.Text.Encodings.Web;

public class ApiKeyAuthenticationOptions : AuthenticationSchemeOptions
{
    public const string DefaultScheme = "ApiKey";
    public string HeaderName { get; set; } = "X-API-Key";
}

public class ApiKeyAuthenticationHandler : AuthenticationHandler<ApiKeyAuthenticationOptions>
{
    private readonly IApiKeyValidator _apiKeyValidator;

    public ApiKeyAuthenticationHandler(
        IOptionsMonitor<ApiKeyAuthenticationOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock,
        IApiKeyValidator apiKeyValidator)
        : base(options, logger, encoder, clock)
    {
        _apiKeyValidator = apiKeyValidator;
    }

    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        // Check if header exists
        if (!Request.Headers.TryGetValue(Options.HeaderName, out var apiKeyValues))
        {
            return AuthenticateResult.Fail("API Key not found");
        }

        var apiKey = apiKeyValues.FirstOrDefault();
        if (string.IsNullOrWhiteSpace(apiKey))
        {
            return AuthenticateResult.Fail("API Key is empty");
        }

        // Validate API key
        var validationResult = await _apiKeyValidator.ValidateAsync(apiKey);
        if (!validationResult.IsValid)
        {
            return AuthenticateResult.Fail("Invalid API Key");
        }

        // Create claims
        var claims = new[]
        {
            new Claim(ClaimTypes.Name, validationResult.ClientName),
            new Claim("ApiKeyId", validationResult.ApiKeyId)
        };

        var identity = new ClaimsIdentity(claims, Scheme.Name);
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, Scheme.Name);

        return AuthenticateResult.Success(ticket);
    }
}

// API Key validator interface
public interface IApiKeyValidator
{
    Task<ApiKeyValidationResult> ValidateAsync(string apiKey);
}

public class ApiKeyValidationResult
{
    public bool IsValid { get; set; }
    public string ClientName { get; set; }
    public string ApiKeyId { get; set; }
}

// Example implementation
public class ApiKeyValidator : IApiKeyValidator
{
    private readonly ApplicationDbContext _context;

    public ApiKeyValidator(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<ApiKeyValidationResult> ValidateAsync(string apiKey)
    {
        // Check against database
        var key = await _context.ApiKeys
            .Where(k => k.Key == apiKey && k.IsActive)
            .FirstOrDefaultAsync();

        if (key == null)
        {
            return new ApiKeyValidationResult { IsValid = false };
        }

        // Update last used
        key.LastUsed = DateTime.UtcNow;
        await _context.SaveChangesAsync();

        return new ApiKeyValidationResult
        {
            IsValid = true,
            ClientName = key.ClientName,
            ApiKeyId = key.Id.ToString()
        };
    }
}

// Registration
builder.Services.AddScoped<IApiKeyValidator, ApiKeyValidator>();
builder.Services.AddAuthentication(ApiKeyAuthenticationOptions.DefaultScheme)
    .AddScheme<ApiKeyAuthenticationOptions, ApiKeyAuthenticationHandler>(
        ApiKeyAuthenticationOptions.DefaultScheme, null);

// Usage
[Authorize(AuthenticationSchemes = ApiKeyAuthenticationOptions.DefaultScheme)]
[ApiController]
[Route("api/[controller]")]
public class ExternalApiController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        var clientName = User.Identity.Name;
        return Ok(new { message = $"Hello {clientName}" });
    }
}
```

### Method 6: Certificate Authentication

**When to use:**
- ✅ High-security requirements
- ✅ Mutual TLS (mTLS)
- ✅ Enterprise B2B scenarios
- ❌ Public-facing applications

```csharp
builder.Services.AddAuthentication(
    CertificateAuthenticationDefaults.AuthenticationScheme)
    .AddCertificate(options =>
    {
        options.AllowedCertificateTypes = CertificateTypes.All;
        
        options.Events = new CertificateAuthenticationEvents
        {
            OnCertificateValidated = context =>
            {
                var claims = new[]
                {
                    new Claim(ClaimTypes.Name, 
                        context.ClientCertificate.Subject,
                        ClaimValueTypes.String)
                };

                context.Principal = new ClaimsPrincipal(
                    new ClaimsIdentity(claims, context.Scheme.Name));
                
                context.Success();
                return Task.CompletedTask;
            },
            OnAuthenticationFailed = context =>
            {
                context.Fail("Invalid certificate");
                return Task.CompletedTask;
            }
        };
    });

// Configure Kestrel to require certificate
builder.WebHost.ConfigureKestrel(options =>
{
    options.ConfigureHttpsDefaults(httpsOptions =>
    {
        httpsOptions.ClientCertificateMode = ClientCertificateMode.RequireCertificate;
    });
});
```

---

## 9. Common Security Patterns

### Pattern 1: Refresh Tokens (For Long-Lived Sessions)

**Why needed:** JWT access tokens should be short-lived (15-60 min). Refresh tokens allow getting new access tokens without re-login.

```csharp
public class RefreshToken
{
    public string Token { get; set; }
    public string UserId { get; set; }
    public DateTime ExpiresAt { get; set; }
    public DateTime CreatedAt { get; set; }
    public bool IsRevoked { get; set; }
    public string CreatedByIp { get; set; }
}

public interface ITokenService
{
    string GenerateAccessToken(string userId, string username, List<string> roles);
    RefreshToken GenerateRefreshToken(string userId, string ipAddress);
    Task<(string accessToken, RefreshToken refreshToken)?> RefreshTokensAsync(
        string refreshToken, string ipAddress);
}

public class TokenService : ITokenService
{
    private readonly ApplicationDbContext _context;
    private readonly IConfiguration _configuration;

    public string GenerateAccessToken(string userId, string username, List<string> roles)
    {
        // ... JWT generation (short-lived: 15-60 min)
    }

    public RefreshToken GenerateRefreshToken(string userId, string ipAddress)
    {
        return new RefreshToken
        {
            Token = Convert.ToBase64String(RandomNumberGenerator.GetBytes(64)),
            UserId = userId,
            ExpiresAt = DateTime.UtcNow.AddDays(7), // 7 days
            CreatedAt = DateTime.UtcNow,
            CreatedByIp = ipAddress
        };
    }

    public async Task<(string accessToken, RefreshToken refreshToken)?> 
        RefreshTokensAsync(string refreshToken, string ipAddress)
    {
        var token = await _context.RefreshTokens
            .Include(t => t.User)
            .FirstOrDefaultAsync(t => t.Token == refreshToken);

        if (token == null || token.IsRevoked || token.ExpiresAt < DateTime.UtcNow)
        {
            return null; // Invalid or expired
        }

        // Revoke old token
        token.IsRevoked = true;
        
        // Generate new tokens
        var user = token.User;
        var roles = await _userManager.GetRolesAsync(user);
        
        var newAccessToken = GenerateAccessToken(user.Id, user.UserName, roles.ToList());
        var newRefreshToken = GenerateRefreshToken(user.Id, ipAddress);
        
        _context.RefreshTokens.Add(newRefreshToken);
        await _context.SaveChangesAsync();

        return (newAccessToken, newRefreshToken);
    }
}

// API endpoints
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        // Validate credentials...
        
        var accessToken = _tokenService.GenerateAccessToken(userId, username, roles);
        var refreshToken = _tokenService.GenerateRefreshToken(userId, GetIpAddress());
        
        _context.RefreshTokens.Add(refreshToken);
        await _context.SaveChangesAsync();

        return Ok(new
        {
            accessToken,
            refreshToken = refreshToken.Token,
            expiresIn = 900 // 15 minutes in seconds
        });
    }

    [HttpPost("refresh")]
    public async Task<IActionResult> Refresh([FromBody] RefreshRequest request)
    {
        var result = await _tokenService.RefreshTokensAsync(
            request.RefreshToken, GetIpAddress());

        if (result == null)
        {
            return Unauthorized(new { message = "Invalid refresh token" });
        }

        return Ok(new
        {
            accessToken = result.Value.accessToken,
            refreshToken = result.Value.refreshToken.Token,
            expiresIn = 900
        });
    }

    private string GetIpAddress()
    {
        return HttpContext.Connection.RemoteIpAddress?.ToString() ?? "unknown";
    }
}
```

### Pattern 2: Password Reset with Tokens

```csharp
public class AccountController : Controller
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly IEmailSender _emailSender;

    [HttpPost("forgot-password")]
    public async Task<IActionResult> ForgotPassword([FromBody] ForgotPasswordRequest request)
    {
        var user = await _userManager.FindByEmailAsync(request.Email);
        if (user == null)
        {
            // Don't reveal user doesn't exist (security)
            return Ok(new { message = "If email exists, reset link sent" });
        }

        // Generate reset token
        var token = await _userManager.GeneratePasswordResetTokenAsync(user);
        
        // Create reset link
        var resetLink = Url.Action("ResetPassword", "Account",
            new { token, email = user.Email }, Request.Scheme);

        // Send email
        await _emailSender.SendEmailAsync(user.Email, "Password Reset",
            $"Click here to reset your password: {resetLink}");

        return Ok(new { message = "If email exists, reset link sent" });
    }

    [HttpPost("reset-password")]
    public async Task<IActionResult> ResetPassword([FromBody] ResetPasswordRequest request)
    {
        var user = await _userManager.FindByEmailAsync(request.Email);
        if (user == null)
        {
            return BadRequest(new { message = "Invalid request" });
        }

        var result = await _userManager.ResetPasswordAsync(
            user, request.Token, request.NewPassword);

        if (!result.Succeeded)
        {
            return BadRequest(result.Errors);
        }

        return Ok(new { message = "Password reset successful" });
    }
}
```

### Pattern 3: Email Confirmation

```csharp
[HttpPost("register")]
public async Task<IActionResult> Register([FromBody] RegisterRequest request)
{
    var user = new ApplicationUser
    {
        UserName = request.Email,
        Email = request.Email
    };

    var result = await _userManager.CreateAsync(user, request.Password);
    if (!result.Succeeded)
    {
        return BadRequest(result.Errors);
    }

    // Generate email confirmation token
    var token = await _userManager.GenerateEmailConfirmationTokenAsync(user);
    
    // Create confirmation link
    var confirmationLink = Url.Action("ConfirmEmail", "Account",
        new { userId = user.Id, token }, Request.Scheme);

    // Send confirmation email
    await _emailSender.SendEmailAsync(user.Email, "Confirm your email",
        $"Please confirm your email: {confirmationLink}");

    return Ok(new { message = "Registration successful. Please confirm your email." });
}

[HttpGet("confirm-email")]
public async Task<IActionResult> ConfirmEmail(string userId, string token)
{
    var user = await _userManager.FindByIdAsync(userId);
    if (user == null)
    {
        return BadRequest("Invalid user");
    }

    var result = await _userManager.ConfirmEmailAsync(user, token);
    if (!result.Succeeded)
    {
        return BadRequest("Email confirmation failed");
    }

    return Ok("Email confirmed successfully");
}
```

### Pattern 4: Two-Factor Authentication (2FA)

```csharp
// Enable 2FA for user
[Authorize]
[HttpPost("enable-2fa")]
public async Task<IActionResult> EnableTwoFactor()
{
    var user = await _userManager.GetUserAsync(User);
    
    // Generate authenticator key
    var key = await _userManager.GetAuthenticatorKeyAsync(user);
    if (string.IsNullOrEmpty(key))
    {
        await _userManager.ResetAuthenticatorKeyAsync(user);
        key = await _userManager.GetAuthenticatorKeyAsync(user);
    }

    // Generate QR code URL (for Google Authenticator, etc.)
    var qrCodeUrl = $"otpauth://totp/YourApp:{user.Email}?secret={key}&issuer=YourApp";

    return Ok(new
    {
        key,
        qrCodeUrl
    });
}

// Verify and enable 2FA
[Authorize]
[HttpPost("verify-2fa")]
public async Task<IActionResult> VerifyTwoFactor([FromBody] Verify2FARequest request)
{
    var user = await _userManager.GetUserAsync(User);
    
    var isValid = await _userManager.VerifyTwoFactorTokenAsync(
        user, _userManager.Options.Tokens.AuthenticatorTokenProvider, request.Code);

    if (!isValid)
    {
        return BadRequest("Invalid code");
    }

    await _userManager.SetTwoFactorEnabledAsync(user, true);
    
    // Generate recovery codes
    var recoveryCodes = await _userManager.GenerateNewTwoFactorRecoveryCodesAsync(user, 10);

    return Ok(new
    {
        message = "2FA enabled",
        recoveryCodes
    });
}

// Login with 2FA
[HttpPost("login-2fa")]
public async Task<IActionResult> LoginWith2FA([FromBody] Login2FARequest request)
{
    var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();
    if (user == null)
    {
        return BadRequest("Invalid request");
    }

    var result = await _signInManager.TwoFactorAuthenticatorSignInAsync(
        request.Code, request.RememberMe, request.RememberMachine);

    if (result.Succeeded)
    {
        // Generate JWT or redirect...
        return Ok(new { message = "Login successful" });
    }

    return Unauthorized("Invalid code");
}
```

---

## 10. Troubleshooting Common Issues

### Issue 1: 401 Unauthorized Despite Valid Token

**Problem:**
```
GET /api/protected → 401 Unauthorized
(Token is valid but still getting 401)
```

**Common Causes:**

1. **Authentication middleware not added or in wrong order**
```csharp
// ❌ Wrong
app.UseAuthorization();
app.UseAuthentication(); // Too late!

// ✅ Correct
app.UseAuthentication(); // Must be before Authorization
app.UseAuthorization();
```

2. **Token not sent correctly**
```javascript
// ❌ Wrong
headers: { 'Authorization': token }

// ✅ Correct
headers: { 'Authorization': `Bearer ${token}` }
```

3. **Token expired**
- Check token expiry time
- Implement refresh token pattern

4. **Wrong authentication scheme**
```csharp
// If using multiple schemes
[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

### Issue 2: CORS Errors with Authentication

**Problem:**
```
Access to fetch at 'https://api.example.com' from origin 'http://localhost:3000' 
has been blocked by CORS policy
```

**Solution:**
```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", builder =>
    {
        builder.WithOrigins("http://localhost:3000", "https://myapp.com")
               .AllowAnyMethod()
               .AllowAnyHeader()
               .AllowCredentials(); // Important for cookies/auth!
    });
});

var app = builder.Build();

app.UseCors("AllowFrontend"); // Before Authentication!
app.UseAuthentication();
app.UseAuthorization();
```

### Issue 3: Claims Not Available After Login

**Problem:**
```csharp
var email = User.FindFirst(ClaimTypes.Email)?.Value; // null
```

**Solutions:**

1. **Ensure claims are added during sign-in**
```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Email, user.Email), // Make sure this is added!
    new Claim(ClaimTypes.Name, user.UserName)
};
```

2. **For Identity, add claims to user**
```csharp
await _userManager.AddClaimAsync(user, new Claim("Department", "IT"));
```

3. **For JWT, include in token**
```csharp
claims.Add(new Claim("Department", "IT"));
// Then create JWT with these claims
```

### Issue 4: Identity Scaffolding Issues

**Problem:** Can't scaffold Identity pages

**Solution:**
```bash
# Install tool
dotnet tool install -g dotnet-aspnet-codegenerator

# Scaffold specific pages
dotnet aspnet-codegenerator identity -dc ApplicationDbContext --files "Account.Register;Account.Login"

# Scaffold all pages
dotnet aspnet-codegenerator identity -dc ApplicationDbContext
```

### Issue 5: "Unable to resolve service" for UserManager

**Problem:**
```
Unable to resolve service for type 'Microsoft.AspNetCore.Identity.UserManager`1'
```

**Solution:** Ensure Identity is registered before building app
```csharp
builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

var app = builder.Build(); // Identity must be registered before this
```

---

## 11. Best Practices & Security Checklist

### Authentication Best Practices

- ✅ **Always use HTTPS** - Never send credentials over HTTP
- ✅ **Hash passwords** - Use Identity or BCrypt, never plain text
- ✅ **Implement account lockout** - Prevent brute force attacks
- ✅ **Use strong password requirements** - Min 8 chars, upper, lower, digit, special
- ✅ **Implement 2FA** - For sensitive applications
- ✅ **Short JWT expiry** - 15-60 minutes recommended
- ✅ **Use refresh tokens** - For long-lived sessions
- ✅ **Validate all token parameters** - Issuer, audience, expiry, signature
- ✅ **Store secrets securely** - User Secrets (dev), Key Vault (prod)
- ✅ **Implement rate limiting** - Prevent abuse
- ✅ **Log authentication events** - For security monitoring
- ❌ **Never log passwords** - Even in error messages
- ❌ **Don't trust client data** - Always validate server-side
- ❌ **Don't store tokens in localStorage** - XSS vulnerable (use httpOnly cookies)

### Authorization Best Practices

- ✅ **Principle of least privilege** - Grant minimum necessary permissions
- ✅ **Use policy-based authorization** - More flexible than role-based
- ✅ **Implement resource-based authorization** - For user-owned resources
- ✅ **Check authorization at API level** - Not just UI
- ✅ **Use [Authorize] by default** - Opt-out with [AllowAnonymous]
- ✅ **Centralize authorization logic** - Reusable policies and handlers
- ✅ **Test authorization** - Unit test policies and handlers
- ❌ **Don't rely on client-side authorization** - Always validate server-side
- ❌ **Don't expose sensitive data in errors** - "Access denied" vs "User not found"

### Security Checklist

**Before Production:**

- [ ] HTTPS enforced everywhere
- [ ] Strong password requirements configured
- [ ] Account lockout enabled
- [ ] Email confirmation required
- [ ] JWT tokens short-lived (< 60 min)
- [ ] Refresh tokens implemented
- [ ] Secrets in Key Vault/environment variables
- [ ] CORS properly configured
- [ ] Rate limiting implemented
- [ ] Logging and monitoring in place
- [ ] 2FA available for sensitive accounts
- [ ] Password reset flow tested
- [ ] Authorization tested (unit tests)
- [ ] Security headers configured (HSTS, etc.)
- [ ] SQL injection prevention (use parameters)
- [ ] XSS prevention (validate/encode output)
- [ ] CSRF protection (for cookies)

**Regular Security Maintenance:**

- [ ] Review and rotate secrets/keys
- [ ] Monitor authentication logs
- [ ] Update security packages
- [ ] Review user permissions
- [ ] Test security scenarios
- [ ] Revoke old/compromised tokens
- [ ] Audit external login providers

---

# PART 2: TECHNICAL REFERENCE

---

## 12. Important Interfaces & Classes Reference

### ClaimsPrincipal Class ⭐⭐⭐

**Purpose:** Represents the authenticated user with all their identities and claims

**Namespace:** `System.Security.Claims`

**Declaration:**
```csharp
public class ClaimsPrincipal : IPrincipal
{
    public ClaimsPrincipal();
    public ClaimsPrincipal(IEnumerable<ClaimsIdentity> identities);
    public ClaimsPrincipal(IIdentity identity);
    
    public virtual IEnumerable<ClaimsIdentity> Identities { get; }
    public virtual IIdentity Identity { get; }
    public virtual IEnumerable<Claim> Claims { get; }
    
    // Find claims
    public virtual Claim FindFirst(string type);
    public virtual Claim FindFirst(Predicate<Claim> match);
    public virtual IEnumerable<Claim> FindAll(string type);
    public virtual IEnumerable<Claim> FindAll(Predicate<Claim> match);
    
    // Check claims
    public virtual bool HasClaim(string type, string value);
    public virtual bool HasClaim(Predicate<Claim> match);
    
    // Check roles
    public virtual bool IsInRole(string role);
}
```

**Key Properties & Methods:**

| Member | Type | Description |
|--------|------|-------------|
| `Identity` | Property | Primary identity (first in Identities) |
| `Identities` | Property | All identities for this principal |
| `Claims` | Property | All claims from all identities |
| `FindFirst(type)` | Method | Find first claim of type |
| `FindAll(type)` | Method | Find all claims of type |
| `HasClaim(type, value)` | Method | Check if claim exists |
| `IsInRole(role)` | Method | Check if user has role |

**Usage Examples:**

```csharp
// Access current user in controller
public class MyController : ControllerBase
{
    public IActionResult GetUserInfo()
    {
        // User is ClaimsPrincipal
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        var username = User.Identity.Name;
        var email = User.FindFirst(ClaimTypes.Email)?.Value;
        var roles = User.FindAll(ClaimTypes.Role).Select(c => c.Value);
        
        if (User.IsInRole("Admin"))
        {
            // Admin-specific logic
        }
        
        return Ok(new { userId, username, email, roles });
    }
}

// Access in middleware
public class MyMiddleware
{
    public async Task InvokeAsync(HttpContext context)
    {
        if (context.User.Identity.IsAuthenticated)
        {
            var name = context.User.Identity.Name;
            // ...
        }
        
        await _next(context);
    }
}

// Access in Razor view
@if (User.IsInRole("Admin"))
{
    <a href="/admin">Admin Panel</a>
}

// Create ClaimsPrincipal manually
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "john"),
    new Claim(ClaimTypes.Email, "john@example.com"),
    new Claim(ClaimTypes.Role, "User")
};

var identity = new ClaimsIdentity(claims, "Custom");
var principal = new ClaimsPrincipal(identity);
```

---

### ClaimsIdentity Class

**Purpose:** Represents a single identity with claims

**Namespace:** `System.Security.Claims`

**Declaration:**
```csharp
public class ClaimsIdentity : IIdentity
{
    public ClaimsIdentity();
    public ClaimsIdentity(IEnumerable<Claim> claims);
    public ClaimsIdentity(IEnumerable<Claim> claims, string authenticationType);
    
    public virtual string AuthenticationType { get; }
    public virtual bool IsAuthenticated { get; }
    public virtual string Name { get; }
    public virtual IEnumerable<Claim> Claims { get; }
    
    public virtual void AddClaim(Claim claim);
    public virtual void AddClaims(IEnumerable<Claim> claims);
    public virtual void RemoveClaim(Claim claim);
    public virtual Claim FindFirst(string type);
}
```

**Usage:**
```csharp
// Create identity with claims
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "john"),
    new Claim(ClaimTypes.Email, "john@example.com")
};

var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);

// Add more claims
identity.AddClaim(new Claim("Department", "IT"));

// Check authentication
if (identity.IsAuthenticated)
{
    Console.WriteLine($"Authenticated as: {identity.Name}");
}
```

---

### Claim Class

**Purpose:** Represents a single piece of information about the user

**Declaration:**
```csharp
public class Claim
{
    public Claim(string type, string value);
    public Claim(string type, string value, string valueType);
    public Claim(string type, string value, string valueType, string issuer);
    
    public string Type { get; }
    public string Value { get; }
    public string ValueType { get; }
    public string Issuer { get; }
}
```

**Common Claim Types:**

| ClaimTypes Constant | Value | Description |
|---------------------|-------|-------------|
| `ClaimTypes.Name` | name | Username |
| `ClaimTypes.Email` | email | Email address |
| `ClaimTypes.NameIdentifier` | sub | Unique user ID |
| `ClaimTypes.Role` | role | User role |
| `ClaimTypes.DateOfBirth` | birthdate | Date of birth |
| `ClaimTypes.GivenName` | given_name | First name |
| `ClaimTypes.Surname` | family_name | Last name |
| `ClaimTypes.MobilePhone` | phone_number | Phone number |

**JWT Registered Claims:**

| JwtRegisteredClaimNames | Description |
|-------------------------|-------------|
| `Sub` | Subject (user ID) |
| `Name` | Name |
| `Email` | Email |
| `Jti` | JWT ID (unique token ID) |
| `Iat` | Issued at |
| `Exp` | Expiration time |
| `Nbf` | Not before |
| `Aud` | Audience |
| `Iss` | Issuer |

**Usage:**
```csharp
// Standard claims
var nameClaim = new Claim(ClaimTypes.Name, "john");
var emailClaim = new Claim(ClaimTypes.Email, "john@example.com");
var roleClaim = new Claim(ClaimTypes.Role, "Admin");

// Custom claims
var departmentClaim = new Claim("Department", "IT");
var employeeIdClaim = new Claim("EmployeeId", "12345");

// JWT claims
var claims = new List<Claim>
{
    new Claim(JwtRegisteredClaimNames.Sub, "user-123"),
    new Claim(JwtRegisteredClaimNames.Email, "user@example.com"),
    new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
};
```

---

### UserManager<TUser> Class ✨ Identity

**Purpose:** Manages users in the backing store

**Namespace:** `Microsoft.AspNetCore.Identity`

**Key Methods:**

| Method | Description |
|--------|-------------|
| `CreateAsync(user, password)` | Create user with password |
| `UpdateAsync(user)` | Update user |
| `DeleteAsync(user)` | Delete user |
| `FindByIdAsync(userId)` | Find user by ID |
| `FindByNameAsync(username)` | Find user by username |
| `FindByEmailAsync(email)` | Find user by email |
| `CheckPasswordAsync(user, password)` | Verify password |
| `ChangePasswordAsync(user, oldPwd, newPwd)` | Change password |
| `AddToRoleAsync(user, role)` | Add user to role |
| `RemoveFromRoleAsync(user, role)` | Remove user from role |
| `GetRolesAsync(user)` | Get user's roles |
| `IsInRoleAsync(user, role)` | Check if user in role |
| `AddClaimAsync(user, claim)` | Add claim to user |
| `RemoveClaimAsync(user, claim)` | Remove claim from user |
| `GetClaimsAsync(user)` | Get user's claims |
| `GeneratePasswordResetTokenAsync(user)` | Generate reset token |
| `ResetPasswordAsync(user, token, newPwd)` | Reset password with token |
| `GenerateEmailConfirmationTokenAsync(user)` | Generate email token |
| `ConfirmEmailAsync(user, token)` | Confirm email with token |
| `SetTwoFactorEnabledAsync(user, enabled)` | Enable/disable 2FA |
| `GetTwoFactorEnabledAsync(user)` | Check if 2FA enabled |

**Usage:**
```csharp
public class UserService
{
    private readonly UserManager<ApplicationUser> _userManager;

    public UserService(UserManager<ApplicationUser> userManager)
    {
        _userManager = userManager;
    }

    public async Task<ApplicationUser> CreateUserAsync(string email, string password)
    {
        var user = new ApplicationUser
        {
            UserName = email,
            Email = email
        };

        var result = await _userManager.CreateAsync(user, password);
        if (result.Succeeded)
        {
            await _userManager.AddToRoleAsync(user, "User");
            return user;
        }

        throw new Exception(string.Join(", ", result.Errors.Select(e => e.Description)));
    }

    public async Task<bool> ValidatePasswordAsync(string email, string password)
    {
        var user = await _userManager.FindByEmailAsync(email);
        if (user == null) return false;

        return await _userManager.CheckPasswordAsync(user, password);
    }
}
```

---

### SignInManager<TUser> Class ✨ Identity

**Purpose:** Manages user sign-in operations

**Key Methods:**

| Method | Description |
|--------|-------------|
| `PasswordSignInAsync(username, password, isPersistent, lockout)` | Sign in with password |
| `SignInAsync(user, isPersistent)` | Sign in user directly |
| `SignOutAsync()` | Sign out user |
| `CheckPasswordSignInAsync(user, password, lockout)` | Check password without signing in |
| `ExternalLoginSignInAsync(loginProvider, providerKey, isPersistent)` | Sign in with external provider |
| `GetExternalLoginInfoAsync()` | Get external login info |
| `ConfigureExternalAuthenticationProperties(provider, redirectUrl)` | Configure external auth |
| `TwoFactorAuthenticatorSignInAsync(code, isPersistent, rememberClient)` | Sign in with 2FA code |

**Usage:**
```csharp
public class AuthService
{
    private readonly SignInManager<ApplicationUser> _signInManager;

    public async Task<bool> SignInAsync(string email, string password, bool rememberMe)
    {
        var result = await _signInManager.PasswordSignInAsync(
            email, password, rememberMe, lockoutOnFailure: true);

        return result.Succeeded;
    }

    public async Task SignOutAsync()
    {
        await _signInManager.SignOutAsync();
    }
}
```

---

### RoleManager<TRole> Class ✨ Identity

**Purpose:** Manages roles in the backing store

**Key Methods:**

| Method | Description |
|--------|-------------|
| `CreateAsync(role)` | Create role |
| `UpdateAsync(role)` | Update role |
| `DeleteAsync(role)` | Delete role |
| `RoleExistsAsync(roleName)` | Check if role exists |
| `FindByNameAsync(roleName)` | Find role by name |
| `AddClaimAsync(role, claim)` | Add claim to role |

**Usage:**
```csharp
public async Task EnsureRolesExistAsync()
{
    string[] roles = { "Admin", "Manager", "User" };

    foreach (var roleName in roles)
    {
        if (!await _roleManager.RoleExistsAsync(roleName))
        {
            await _roleManager.CreateAsync(new IdentityRole(roleName));
        }
    }
}
```

---

### IAuthorizationService Interface

**Purpose:** Evaluate authorization policies programmatically

**Methods:**
```csharp
public interface IAuthorizationService
{
    Task<AuthorizationResult> AuthorizeAsync(
        ClaimsPrincipal user, 
        object resource, 
        string policyName);
    
    Task<AuthorizationResult> AuthorizeAsync(
        ClaimsPrincipal user, 
        object resource, 
        IEnumerable<IAuthorizationRequirement> requirements);
}
```

**Usage:**
```csharp
public class DocumentController : ControllerBase
{
    private readonly IAuthorizationService _authorizationService;

    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var document = await _context.Documents.FindAsync(id);
        
        // Check authorization programmatically
        var authResult = await _authorizationService.AuthorizeAsync(
            User, document, "DocumentOwner");

        if (!authResult.Succeeded)
        {
            return Forbid();
        }

        _context.Documents.Remove(document);
        await _context.SaveChangesAsync();

        return NoContent();
    }
}
```

---

### AuthenticationProperties Class

**Purpose:** Stores state values related to authentication session

**Key Properties:**

| Property | Description |
|----------|-------------|
| `IsPersistent` | Whether authentication is persistent across sessions |
| `ExpiresUtc` | When authentication expires |
| `IssuedUtc` | When authentication was issued |
| `AllowRefresh` | Whether session can be refreshed |
| `RedirectUri` | Where to redirect after authentication |

**Usage:**
```csharp
var properties = new AuthenticationProperties
{
    IsPersistent = true, // "Remember me"
    ExpiresUtc = DateTimeOffset.UtcNow.AddDays(7),
    AllowRefresh = true
};

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal,
    properties);
```

---

## 13. Configuration Deep-Dive

### Pattern 1: Inline Configuration (Quick)

**When to use:** Development, prototyping, simple scenarios

```csharp
var builder = WebApplication.CreateBuilder(args);

// Inline configuration
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = "https://myapp.com",
            ValidAudience = "https://myapp.com",
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes("MySuperSecretKeyThatIsAtLeast32Characters"))
        };
    });
```

**Pros:** ✅ Quick, easy to understand  
**Cons:** ❌ Hardcoded values, not configurable, secrets in code

---

### Pattern 2: Configuration Object (Better)

**When to use:** Production applications, need flexibility

**Step 1: Create configuration class**

```csharp
public class JwtSettings
{
    public string Issuer { get; set; }
    public string Audience { get; set; }
    public string SecretKey { get; set; }
    public int ExpiryMinutes { get; set; }
}

public class CookieSettings
{
    public int ExpiryHours { get; set; }
    public bool SlidingExpiration { get; set; }
    public string LoginPath { get; set; }
    public string LogoutPath { get; set; }
}

public class IdentitySettings
{
    public PasswordOptions Password { get; set; }
    public LockoutOptions Lockout { get; set; }
    public UserOptions User { get; set; }
}
```

**Step 2: Add to appsettings.json**

```json
{
  "JwtSettings": {
    "Issuer": "https://myapp.com",
    "Audience": "https://myapp.com",
    "SecretKey": "YourSecretKeyGoesHere",
    "ExpiryMinutes": 60
  },
  "CookieSettings": {
    "ExpiryHours": 2,
    "SlidingExpiration": true,
    "LoginPath": "/Account/Login",
    "LogoutPath": "/Account/Logout"
  },
  "IdentitySettings": {
    "Password": {
      "RequireDigit": true,
      "RequireLowercase": true,
      "RequireUppercase": true,
      "RequireNonAlphanumeric": true,
      "RequiredLength": 8
    },
    "Lockout": {
      "DefaultLockoutTimeSpanMinutes": 5,
      "MaxFailedAccessAttempts": 5,
      "AllowedForNewUsers": true
    },
    "User": {
      "RequireUniqueEmail": true
    }
  }
}
```

**Step 3: Use in Program.cs**

```csharp
// Read configuration
var jwtSettings = builder.Configuration.GetSection("JwtSettings").Get<JwtSettings>();
var cookieSettings = builder.Configuration.GetSection("CookieSettings").Get<CookieSettings>();

// Configure JWT
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwtSettings.Issuer,
            ValidAudience = jwtSettings.Audience,
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(jwtSettings.SecretKey))
        };
    });

// Configure cookies
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.ExpireTimeSpan = TimeSpan.FromHours(cookieSettings.ExpiryHours);
        options.SlidingExpiration = cookieSettings.SlidingExpiration;
        options.LoginPath = cookieSettings.LoginPath;
        options.LogoutPath = cookieSettings.LogoutPath;
    });
```

**Pros:** ✅ Configurable, environment-specific  
**Cons:** ❌ Still need to manually read configuration

---

### Pattern 3: IOptions Pattern (Recommended)

**When to use:** Production applications, dependency injection, testability

**Step 1: Create settings classes (same as Pattern 2)**

**Step 2: Register settings**

```csharp
// Register settings
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection("JwtSettings"));

builder.Services.Configure<CookieSettings>(
    builder.Configuration.GetSection("CookieSettings"));

builder.Services.Configure<IdentitySettings>(
    builder.Configuration.GetSection("IdentitySettings"));
```

**Step 3: Inject and use in services**

```csharp
using Microsoft.Extensions.Options;

public class TokenService : ITokenService
{
    private readonly JwtSettings _jwtSettings;

    public TokenService(IOptions<JwtSettings> jwtSettings)
    {
        _jwtSettings = jwtSettings.Value;
    }

    public string GenerateToken(string userId, string username, List<string> roles)
    {
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_jwtSettings.SecretKey));
        
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var claims = new List<Claim>
        {
            new Claim(JwtRegisteredClaimNames.Sub, userId),
            new Claim(JwtRegisteredClaimNames.Name, username)
        };

        foreach (var role in roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }

        var token = new JwtSecurityToken(
            issuer: _jwtSettings.Issuer,
            audience: _jwtSettings.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwtSettings.ExpiryMinutes),
            signingCredentials: credentials
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

**Pros:** ✅ Dependency injection, testable, reusable, change monitoring  
**Cons:** ❌ Slightly more complex setup

---

### IOptions vs IOptionsSnapshot vs IOptionsMonitor

| Feature | IOptions<T> | IOptionsSnapshot<T> | IOptionsMonitor<T> |
|---------|------------|---------------------|-------------------|
| **Lifetime** | Singleton | Scoped | Singleton |
| **Reloads config** | ❌ No | ✅ Yes (per request) | ✅ Yes (immediately) |
| **Use case** | Static config | Per-request config | Real-time config changes |
| **Performance** | ✅ Fastest | Good | Good |
| **Named options** | ✅ Yes | ✅ Yes | ✅ Yes |

**When to use which:**

```csharp
// IOptions<T> - Static configuration (doesn't change)
public class TokenService
{
    public TokenService(IOptions<JwtSettings> settings)
    {
        // Settings won't change during app lifetime
    }
}

// IOptionsSnapshot<T> - Scoped, reloads per request
public class MyController : ControllerBase
{
    public MyController(IOptionsSnapshot<AppSettings> settings)
    {
        // Settings reload for each HTTP request
    }
}

// IOptionsMonitor<T> - Real-time monitoring
public class ConfigMonitorService
{
    public ConfigMonitorService(IOptionsMonitor<AppSettings> monitor)
    {
        // React to configuration changes immediately
        monitor.OnChange(settings =>
        {
            Console.WriteLine("Settings changed!");
        });
    }
}
```

---

### Environment-Specific Configuration

**appsettings.Development.json:**
```json
{
  "JwtSettings": {
    "Issuer": "http://localhost:5000",
    "Audience": "http://localhost:5000",
    "SecretKey": "DevSecretKey123!",
    "ExpiryMinutes": 1440
  }
}
```

**appsettings.Production.json:**
```json
{
  "JwtSettings": {
    "Issuer": "https://myapp.com",
    "Audience": "https://myapp.com",
    "ExpiryMinutes": 60
  }
}
```

**Note:** Don't put secrets in appsettings.Production.json! Use environment variables or Azure Key Vault.

---

### Secret Management

**Development - User Secrets:**

```bash
# Initialize user secrets
dotnet user-secrets init

# Set secrets
dotnet user-secrets set "JwtSettings:SecretKey" "MyDevSecret123!"
dotnet user-secrets set "Authentication:Google:ClientSecret" "dev-secret"

# List secrets
dotnet user-secrets list
```

**Production - Environment Variables:**

```csharp
// Set in hosting environment (Azure, AWS, Docker, etc.)
JwtSettings__SecretKey=ProductionSecret123!
Authentication__Google__ClientSecret=prod-secret
```

**Production - Azure Key Vault:**

```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential());
```

---

## 14. Advanced Topics

### Multiple Authentication Schemes

**Scenario:** Support both Cookie (for web) and JWT (for API) authentication

```csharp
builder.Services.AddAuthentication(options =>
{
    // Default scheme for [Authorize] with no parameters
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = CookieAuthenticationDefaults.AuthenticationScheme;
})
.AddCookie(options =>
{
    options.LoginPath = "/Account/Login";
    options.ExpireTimeSpan = TimeSpan.FromHours(2);
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidIssuer = jwtSettings.Issuer,
        ValidateAudience = true,
        ValidAudience = jwtSettings.Audience,
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(jwtSettings.SecretKey))
    };
});

// Use specific scheme
[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
public class ApiController : ControllerBase { }

[Authorize(AuthenticationSchemes = CookieAuthenticationDefaults.AuthenticationScheme)]
public class WebController : Controller { }

// Allow both schemes
[Authorize(AuthenticationSchemes = 
    JwtBearerDefaults.AuthenticationScheme + "," + 
    CookieAuthenticationDefaults.AuthenticationScheme)]
public class HybridController : ControllerBase { }
```

---

### Custom Authentication Handler

**When to use:** Custom authentication logic not covered by existing schemes

```csharp
public class CustomAuthenticationOptions : AuthenticationSchemeOptions
{
    public string TokenHeaderName { get; set; } = "X-Custom-Token";
}

public class CustomAuthenticationHandler : 
    AuthenticationHandler<CustomAuthenticationOptions>
{
    public CustomAuthenticationHandler(
        IOptionsMonitor<CustomAuthenticationOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock)
        : base(options, logger, encoder, clock)
    {
    }

    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        // Get token from header
        if (!Request.Headers.TryGetValue(Options.TokenHeaderName, out var token))
        {
            return AuthenticateResult.NoResult();
        }

        // Validate token (your custom logic)
        var isValid = await ValidateTokenAsync(token);
        if (!isValid)
        {
            return AuthenticateResult.Fail("Invalid token");
        }

        // Create claims
        var claims = new[]
        {
            new Claim(ClaimTypes.Name, "CustomUser"),
            new Claim(ClaimTypes.NameIdentifier, "user-123")
        };

        var identity = new ClaimsIdentity(claims, Scheme.Name);
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, Scheme.Name);

        return AuthenticateResult.Success(ticket);
    }

    private Task<bool> ValidateTokenAsync(string token)
    {
        // Your validation logic
        return Task.FromResult(token == "valid-token");
    }
}

// Registration
builder.Services.AddAuthentication("CustomScheme")
    .AddScheme<CustomAuthenticationOptions, CustomAuthenticationHandler>(
        "CustomScheme", options =>
        {
            options.TokenHeaderName = "X-My-Token";
        });
```

---

### Custom Authorization Requirement with Dependency Injection

```csharp
// Requirement that uses database
public class ValidSubscriptionRequirement : IAuthorizationRequirement { }

public class ValidSubscriptionHandler : 
    AuthorizationHandler<ValidSubscriptionRequirement>
{
    private readonly ISubscriptionService _subscriptionService;

    public ValidSubscriptionHandler(ISubscriptionService subscriptionService)
    {
        _subscriptionService = subscriptionService;
    }

    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        ValidSubscriptionRequirement requirement)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (userId == null)
        {
            return;
        }

        // Check database for active subscription
        var hasValidSubscription = await _subscriptionService
            .HasActiveSubscriptionAsync(userId);

        if (hasValidSubscription)
        {
            context.Succeed(requirement);
        }
    }
}

// Registration
builder.Services.AddScoped<IAuthorizationHandler, ValidSubscriptionHandler>();
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireSubscription", policy =>
        policy.Requirements.Add(new ValidSubscriptionRequirement()));
});
```

---

### Token Blacklisting (Revocation)

**Problem:** JWT tokens can't be revoked once issued

**Solution:** Maintain blacklist/revocation list

```csharp
public interface ITokenBlacklistService
{
    Task BlacklistTokenAsync(string jti, DateTime expiry);
    Task<bool> IsBlacklistedAsync(string jti);
}

public class TokenBlacklistService : ITokenBlacklistService
{
    private readonly IDistributedCache _cache;

    public TokenBlacklistService(IDistributedCache cache)
    {
        _cache = cache;
    }

    public async Task BlacklistTokenAsync(string jti, DateTime expiry)
    {
        var timeToLive = expiry - DateTime.UtcNow;
        await _cache.SetStringAsync(
            $"blacklist:{jti}",
            "revoked",
            new DistributedCacheEntryOptions
            {
                AbsoluteExpiration = expiry
            });
    }

    public async Task<bool> IsBlacklistedAsync(string jti)
    {
        var value = await _cache.GetStringAsync($"blacklist:{jti}");
        return value != null;
    }
}

// Use in JWT validation
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Events = new JwtBearerEvents
        {
            OnTokenValidated = async context =>
            {
                var jti = context.Principal.FindFirst(JwtRegisteredClaimNames.Jti)?.Value;
                var blacklistService = context.HttpContext.RequestServices
                    .GetRequiredService<ITokenBlacklistService>();

                if (await blacklistService.IsBlacklistedAsync(jti))
                {
                    context.Fail("Token has been revoked");
                }
            }
        };
    });
```

---

### Sliding Expiration for JWT

**Problem:** Fixed JWT expiry means user gets logged out mid-session

**Solution:** Issue new token when current token is close to expiry

```csharp
public class TokenRefreshMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ITokenService _tokenService;

    public TokenRefreshMiddleware(RequestDelegate next, ITokenService tokenService)
    {
        _next = next;
        _tokenService = tokenService;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        if (context.User.Identity?.IsAuthenticated == true)
        {
            var exp = context.User.FindFirst(JwtRegisteredClaimNames.Exp)?.Value;
            if (exp != null)
            {
                var expiryDate = DateTimeOffset.FromUnixTimeSeconds(long.Parse(exp));
                var minutesUntilExpiry = (expiryDate - DateTimeOffset.UtcNow).TotalMinutes;

                // If token expires in less than 5 minutes, issue new token
                if (minutesUntilExpiry < 5 && minutesUntilExpiry > 0)
                {
                    var userId = context.User.FindFirst(JwtRegisteredClaimNames.Sub)?.Value;
                    var username = context.User.Identity.Name;
                    var roles = context.User.FindAll(ClaimTypes.Role)
                        .Select(c => c.Value).ToList();

                    var newToken = _tokenService.GenerateToken(userId, username, roles);
                    
                    // Add new token to response header
                    context.Response.Headers.Add("X-New-Token", newToken);
                }
            }
        }

        await _next(context);
    }
}

// Register middleware
app.UseMiddleware<TokenRefreshMiddleware>();
```

---

## 15. Version Timeline

### ASP.NET Core Authentication & Authorization Evolution

| Version | Year | New Features |
|---------|------|--------------|
| **Core 1.0** | 2016 | • Basic Cookie & Bearer authentication<br>• Role-based authorization<br>• Claims-based authorization |
| **Core 1.1** | 2016 | • External authentication (Google, Facebook)<br>• Policy-based authorization |
| **Core 2.0** | 2017 | • ASP.NET Core Identity<br>• IMiddleware interface<br>• Default authentication schemes<br>• Resource-based authorization |
| **Core 2.1** | 2018 | • Identity UI (scaffolding)<br>• GDPR features<br>• Personal data protection APIs |
| **Core 2.2** | 2018 | • Health checks<br>• Improved Identity scaffolding |
| **Core 3.0** | 2019 | • Endpoint routing integration<br>• Authorization middleware improvements<br>• Bearer token improvements |
| **Core 3.1** | 2019 | • Long-term support (LTS)<br>• Performance improvements |
| **.NET 5** | 2020 | • Certificate authentication improvements<br>• API versioning enhancements |
| **.NET 6** | 2021 | • Minimal APIs with authentication<br>• JWT Bearer improvements<br>• Microsoft.Identity.Web integration |
| **.NET 7** | 2022 | • Rate limiting (built-in)<br>• Output caching<br>• Auth improvements for minimal APIs |
| **.NET 8** | 2023 | • Identity API endpoints<br>• Keyed DI services<br>• Enhanced authentication options<br>• Improved OpenAPI support |

### Key Milestones

**✨ ASP.NET Core 2.0** - Identity framework introduced  
**✨ ASP.NET Core 3.0** - Endpoint routing (major architecture change)  
**✨ .NET 6** - Minimal APIs with authentication support  
**✨ .NET 8** - Identity API endpoints (no UI needed)

---

## Summary & Quick Reference

### Decision Tree: Which Authentication Method?

```
START
  ↓
  Building API? ──Yes──→ JWT Authentication
  ↓ No
  ↓
  Traditional web app (MVC/Razor)? ──Yes──→ Cookie Authentication
  ↓ No
  ↓
  Need full user management? ──Yes──→ ASP.NET Core Identity
  ↓ No
  ↓
  Machine-to-machine? ──Yes──→ API Key or Certificate
  ↓ No
  ↓
  Social login? ──Yes──→ External Providers (OAuth)
  ↓
  Consider your requirements...
```

### Common Patterns Quick Reference

```csharp
// 1. Protect action with authentication
[Authorize]
public IActionResult SecureAction() { }

// 2. Protect with role
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly() { }

// 3. Protect with policy
[Authorize(Policy = "MustBeOver18")]
public IActionResult AdultContent() { }

// 4. Allow anonymous (override [Authorize])
[AllowAnonymous]
public IActionResult PublicAction() { }

// 5. Check authentication in code
if (User.Identity.IsAuthenticated) { }

// 6. Get user claims
var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
var email = User.FindFirst(ClaimTypes.Email)?.Value;

// 7. Check role
if (User.IsInRole("Admin")) { }

// 8. Programmatic authorization
var result = await _authorizationService.AuthorizeAsync(User, resource, "Policy");
if (result.Succeeded) { }
```

---

**END OF GUIDE 7: AUTHENTICATION & AUTHORIZATION**

---

**Page Count:** ~40-45 pages (when printed)  
**Last Updated:** December 2024  
**ASP.NET Core Version:** .NET 8.0  
**Status:** ✅ Complete

---
