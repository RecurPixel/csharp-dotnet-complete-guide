# Security Best Practices Quick Reference

---

## Why Security Matters

**Security** = Protecting your application, data, and users from unauthorized access, data breaches, and attacks

**Cost of a Breach:**
- 💸 Average cost: $4.45 million (IBM 2023)
- 📉 Reputational damage
- ⚖️ Legal liability (GDPR fines up to 4% of annual revenue)
- 😔 User trust lost

**Security is not optional — it's a core feature.**

---

## OWASP Top 10 (2021)

**OWASP** = Open Web Application Security Project

```
A01 - Broken Access Control          ← #1 most common
A02 - Cryptographic Failures         ← Weak encryption, plain-text secrets
A03 - Injection                      ← SQL, command, LDAP injection
A04 - Insecure Design                ← Missing threat modeling
A05 - Security Misconfiguration      ← Default credentials, open ports
A06 - Vulnerable Components          ← Outdated NuGet/npm packages
A07 - Identity/Auth Failures         ← Weak passwords, broken sessions
A08 - Software/Data Integrity        ← Insecure deserialization
A09 - Security Logging Failures      ← No audit trail
A10 - SSRF                           ← Server-Side Request Forgery
```

| OWASP Risk | .NET Mitigation |
|---|---|
| Broken Access Control | `[Authorize]`, `[Authorize(Roles="Admin")]`, resource-based auth |
| Cryptographic Failures | ASP.NET Data Protection, HTTPS enforcement |
| Injection | Parameterized queries, EF Core, input validation |
| Security Misconfiguration | Security headers, disable dev features in prod |
| Vulnerable Components | `dotnet list package --vulnerable`, Dependabot |
| Auth Failures | ASP.NET Core Identity, strong password policies |

---

## SQL Injection Prevention

**SQL Injection** = Attacker inserts malicious SQL into a query to access or destroy data

### How It Happens

```sql
-- User input: ' OR '1'='1
-- Query becomes:
SELECT * FROM Users WHERE Username = '' OR '1'='1' AND Password = ''
-- ☠️ Returns ALL users - authentication bypassed
```

### Prevention

```csharp
// ❌ NEVER - String concatenation
var query = $"SELECT * FROM Users WHERE Username = '{username}'";
// If username = "'; DROP TABLE Users;--"  ← Your table is gone

// ✅ ALWAYS - Parameterized queries with ADO.NET
using var command = new SqlCommand(
    "SELECT * FROM Users WHERE Username = @Username AND Password = @Password",
    connection);

command.Parameters.AddWithValue("@Username", username);
command.Parameters.AddWithValue("@Password", hashedPassword);

// ✅ ALWAYS - Dapper (still parameterized)
var user = await connection.QueryFirstOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Username = @Username",
    new { Username = username });

// ✅ ALWAYS - Entity Framework Core (parameterized automatically)
var user = await dbContext.Users
    .Where(u => u.Username == username)
    .FirstOrDefaultAsync();

// ✅ EF Core Raw SQL - use FromSqlInterpolated (NOT FromSqlRaw with string concat)
var users = await dbContext.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Username = {username}")
    .ToListAsync();

// ❌ NEVER - FromSqlRaw with string interpolation
var users = await dbContext.Users
    .FromSqlRaw($"SELECT * FROM Users WHERE Username = '{username}'") // ← Vulnerable!
    .ToListAsync();
```

### Stored Procedures (Safe)

```sql
-- Create parameterized stored procedure
CREATE PROCEDURE GetUserByUsername
    @Username NVARCHAR(100)
AS
BEGIN
    SELECT UserId, Username, Email
    FROM Users
    WHERE Username = @Username  -- Parameter, not concatenation
END
```

```csharp
// Call safely from .NET
var user = await connection.QueryFirstOrDefaultAsync<User>(
    "GetUserByUsername",
    new { Username = username },
    commandType: CommandType.StoredProcedure);
```

---

## XSS Prevention (Cross-Site Scripting)

**XSS** = Attacker injects malicious JavaScript that executes in victims' browsers

### Types of XSS

```
Reflected XSS  → Malicious script in URL parameter, reflected back immediately
Stored XSS     → Malicious script stored in database, served to all visitors
DOM-based XSS  → Client-side JavaScript manipulates DOM unsafely
```

### Prevention in ASP.NET Core

```csharp
// ✅ Razor automatically HTML-encodes output
<p>@Model.UserInput</p>
// If UserInput = "<script>alert('xss')</script>"
// Rendered as: &lt;script&gt;alert('xss')&lt;/script&gt;  ← Safe

// ❌ Raw HTML output - only use with trusted content
<p>@Html.Raw(Model.UserInput)</p>  // Dangerous with user input!

// ✅ Explicit HTML encoding
using Microsoft.AspNetCore.Html;
var encoded = HtmlEncoder.Default.Encode(userInput);

// ✅ URL encoding for URLs
var encodedUrl = UrlEncoder.Default.Encode(userInput);

// ✅ JavaScript encoding for inline JS
var encodedJs = JavaScriptEncoder.Default.Encode(userInput);
```

### Content Security Policy (CSP)

```csharp
// Program.cs - Add CSP header
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("Content-Security-Policy",
        "default-src 'self'; " +
        "script-src 'self' https://cdn.jsdelivr.net; " +
        "style-src 'self' https://fonts.googleapis.com; " +
        "img-src 'self' data: https:; " +
        "font-src 'self' https://fonts.gstatic.com; " +
        "connect-src 'self'; " +
        "frame-ancestors 'none'");

    await next();
});
```

### Sanitizing HTML Input (Rich Text Editors)

```bash
# Install: dotnet add package HtmlSanitizer
```

```csharp
// When users need to submit HTML (e.g., blog posts, comments)
using Ganss.Xss;

public class ContentService
{
    private readonly HtmlSanitizer _sanitizer = new HtmlSanitizer();

    public string SanitizeUserContent(string htmlInput)
    {
        // Allows safe HTML tags, strips scripts and event handlers
        return _sanitizer.Sanitize(htmlInput);
    }
}
```

---

## CSRF Protection (Cross-Site Request Forgery)

**CSRF** = Attacker tricks authenticated users into submitting malicious requests from another site

### How It Works

```
1. User logs into yourbank.com (gets auth cookie)
2. User visits evil.com
3. evil.com sends POST to yourbank.com/transfer?amount=1000&to=attacker
4. Browser auto-attaches the auth cookie → Transfer succeeds! ☠️
```

### Antiforgery Tokens (MVC/Razor Pages)

```csharp
// Program.cs - enabled by default in Razor Pages / MVC
builder.Services.AddAntiforgery(options =>
{
    options.HeaderName = "X-CSRF-TOKEN";    // For AJAX requests
    options.Cookie.Name = "CSRF-TOKEN";
    options.Cookie.SameSite = SameSiteMode.Strict;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});

// ✅ Razor form - @Html.AntiForgeryToken() or tag helper
<form method="post">
    @Html.AntiForgeryToken()   <!-- Embeds hidden token -->
    <!-- or use tag helper (auto-added with method="post") -->
    <input type="submit" value="Submit" />
</form>

// ✅ Controller - validate with [ValidateAntiForgeryToken]
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Transfer(TransferDto dto)
{
    // CSRF-protected
}

// ✅ Apply globally via filter
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});
```

### CSRF for API Endpoints + SameSite Cookies

```csharp
// For APIs with cookie-based auth, SameSite=Strict is your CSRF defense
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.SameSite = SameSiteMode.Strict;  // ✅ Blocks cross-site requests
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.HttpOnly = true;
});

// For APIs using JWT in Authorization header - CSRF doesn't apply
// Browsers never auto-send Authorization headers cross-origin
```

### AJAX with Antiforgery

```javascript
// JavaScript - attach token to AJAX requests
const token = document.querySelector('input[name="__RequestVerificationToken"]').value;

fetch('/api/transfer', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'RequestVerificationToken': token   // Send in header
    },
    body: JSON.stringify({ amount: 100, to: 'friend@email.com' })
});
```

---

## Secure Password Storage

**NEVER store passwords in plain text or with reversible encryption.**

### What NOT to Do

```csharp
// ❌ NEVER - Plain text
user.Password = password;

// ❌ NEVER - MD5 (broken, no salt)
user.Password = MD5.HashData(Encoding.UTF8.GetBytes(password));

// ❌ NEVER - SHA256 without salt (vulnerable to rainbow tables)
user.Password = SHA256.HashData(Encoding.UTF8.GetBytes(password));

// ❌ NEVER - Reversible encryption
user.Password = Encrypt(password, secretKey); // Can be decrypted!
```

### BCrypt (Recommended for Most Apps)

```bash
dotnet add package BCrypt.Net-Next
```

```csharp
using BCrypt.Net;

public class PasswordService
{
    // ✅ Hash password (includes automatic random salt)
    public string HashPassword(string plainTextPassword)
    {
        // Work factor 12 = ~250ms to hash (increases brute-force cost)
        return BCrypt.HashPassword(plainTextPassword, workFactor: 12);
    }

    // ✅ Verify password (timing-safe comparison)
    public bool VerifyPassword(string plainTextPassword, string hashedPassword)
    {
        return BCrypt.Verify(plainTextPassword, hashedPassword);
    }
}

// Usage
var hash = BCrypt.HashPassword("MyPassword123!");
// "$2a$12$R9h/cIPz0gi.URNNX3kh2OPST9/PgBkqquzi.Ss7KIUgO2t0jWMUW"
// └── Algorithm + work factor + salt + hash (all in one string)

var isValid = BCrypt.Verify("MyPassword123!", hash); // true
var isValid = BCrypt.Verify("WrongPassword", hash);  // false
```

### ASP.NET Core Identity (Built-in Hashing)

```csharp
// ASP.NET Core Identity uses PBKDF2 with HMAC-SHA256 by default
// It handles everything automatically

// Register
var result = await _userManager.CreateAsync(user, password);

// Sign in (verifies internally)
var result = await _signInManager.PasswordSignInAsync(username, password, isPersistent: false, lockoutOnFailure: true);

// Custom password hasher if needed
builder.Services.Configure<PasswordHasherOptions>(options =>
{
    options.IterationCount = 300_000; // NIST recommended minimum
});
```

### Argon2 (Highest Security)

```bash
dotnet add package Isopoh.Cryptography.Argon2
```

```csharp
using Isopoh.Cryptography.Argon2;

public string HashPassword(string password)
{
    // Argon2id - winner of the Password Hashing Competition
    var config = new Argon2Config
    {
        Type = Argon2Type.DataIndependentAddressing,
        Version = Argon2Version.Nineteen,
        TimeCost = 3,           // Iterations
        MemoryCost = 65536,     // 64 MB
        Lanes = 4,              // Parallelism
        Threads = Environment.ProcessorCount,
        Password = Encoding.UTF8.GetBytes(password),
        Salt = GenerateSalt(),  // Random 16+ bytes
        HashLength = 32
    };
    using var argon2 = new Argon2(config);
    return argon2.Hash().ToString();
}
```

### Password Rules

```csharp
// ASP.NET Core Identity password configuration
builder.Services.Configure<IdentityOptions>(options =>
{
    options.Password.RequiredLength = 12;          // ✅ Min 12 chars
    options.Password.RequireDigit = true;
    options.Password.RequireLowercase = true;
    options.Password.RequireUppercase = true;
    options.Password.RequireNonAlphanumeric = true;
    options.Password.RequiredUniqueChars = 4;      // ✅ Prevents "aaaaaaaaaaa1!"

    // Account lockout
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
    options.Lockout.AllowedForNewUsers = true;
});
```

---

## HTTPS / TLS

**HTTPS** = HTTP + TLS encryption. Protects data in transit from eavesdropping and tampering.

### Enforce HTTPS in ASP.NET Core

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Redirect HTTP to HTTPS
builder.Services.AddHttpsRedirection(options =>
{
    options.RedirectStatusCode = StatusCodes.Status308PermanentRedirect;
    options.HttpsPort = 443;
});

// HSTS (HTTP Strict Transport Security)
// Tells browsers to ONLY use HTTPS for this domain
builder.Services.AddHsts(options =>
{
    options.Preload = true;
    options.IncludeSubDomains = true;
    options.MaxAge = TimeSpan.FromDays(365);
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();      // ✅ Add HSTS header
}

app.UseHttpsRedirection(); // ✅ Redirect HTTP → HTTPS
```

### Kestrel TLS Configuration

```csharp
// Program.cs - configure HTTPS in Kestrel
builder.WebHost.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(httpsOptions =>
    {
        // Minimum TLS version
        httpsOptions.SslProtocols = SslProtocols.Tls12 | SslProtocols.Tls13;
    });
});
```

### appsettings.json for Certificates

```json
{
  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://*:443",
        "Certificate": {
          "Path": "/certs/app.pfx",
          "Password": "CertificatePassword"
        }
      }
    }
  }
}
```

---

## Security Headers

Security headers instruct browsers on security policies. Add them once, protect everywhere.

```csharp
// Program.cs - add all security headers
app.Use(async (context, next) =>
{
    var headers = context.Response.Headers;

    // Prevent MIME type sniffing
    headers.Add("X-Content-Type-Options", "nosniff");

    // Prevent clickjacking
    headers.Add("X-Frame-Options", "DENY");

    // Enable browser XSS filter (legacy browsers)
    headers.Add("X-XSS-Protection", "1; mode=block");

    // Control referrer information
    headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");

    // Permissions policy (restrict browser features)
    headers.Add("Permissions-Policy",
        "camera=(), microphone=(), geolocation=(), payment=()");

    // HSTS (also set via AddHsts)
    if (context.Request.IsHttps)
        headers.Add("Strict-Transport-Security", "max-age=31536000; includeSubDomains; preload");

    // CSP (adjust to your app's needs)
    headers.Add("Content-Security-Policy",
        "default-src 'self'; " +
        "script-src 'self'; " +
        "style-src 'self' 'unsafe-inline'; " +
        "img-src 'self' data:; " +
        "frame-ancestors 'none'");

    await next();
});
```

### Using NWebSec (Recommended)

```bash
dotnet add package NWebsec.AspNetCore.Middleware
```

```csharp
app.UseXContentTypeOptions();
app.UseXfo(options => options.Deny());
app.UseXXssProtection(options => options.EnabledWithBlockMode());
app.UseReferrerPolicy(options => options.StrictOriginWhenCrossOrigin());
app.UseHsts(options => options.MaxAge(days: 365).IncludeSubdomains().Preload());

app.UseCsp(options =>
{
    options.DefaultSources(s => s.Self())
           .ScriptSources(s => s.Self())
           .StyleSources(s => s.Self().UnsafeInline())
           .ImageSources(s => s.Self().CustomSources("data:"))
           .FontSources(s => s.Self());
});
```

---

## JWT Security

### Secure JWT Configuration

```csharp
// Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            // ✅ Validate everything
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,

            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:SecretKey"]!)),

            // ✅ Clock skew tolerance (default is 5 min - reduce it)
            ClockSkew = TimeSpan.FromSeconds(30)
        };
    });
```

### Generating Secure Tokens

```csharp
public class TokenService
{
    private readonly IConfiguration _config;

    public TokenService(IConfiguration config)
    {
        _config = config;
    }

    public string GenerateToken(User user)
    {
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_config["Jwt:SecretKey"]!));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()), // Unique ID
            new Claim(JwtRegisteredClaimNames.Iat,
                DateTimeOffset.UtcNow.ToUnixTimeSeconds().ToString()) // Issued at
        };

        var token = new JwtSecurityToken(
            issuer: _config["Jwt:Issuer"],
            audience: _config["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(15),  // ✅ Short expiry (15 min)
            signingCredentials: credentials
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### JWT Best Practices

```
✅ Use short expiry (15-60 minutes) + refresh tokens
✅ Store JWT in memory (JS) or HttpOnly cookie — NOT localStorage
✅ Use at least 256-bit signing key
✅ Validate all claims (issuer, audience, expiry)
✅ Implement token refresh rotation
✅ Maintain a revocation list for logout (Redis or DB)
❌ Never store sensitive data in JWT payload (it's base64, not encrypted)
❌ Never use algorithm=none
```

---

## Input Validation & Sanitization

```csharp
// ✅ Use Data Annotations
public class RegisterDto
{
    [Required]
    [StringLength(50, MinimumLength = 3)]
    [RegularExpression(@"^[a-zA-Z0-9_]+$", ErrorMessage = "Alphanumeric only")]
    public string Username { get; set; } = string.Empty;

    [Required]
    [EmailAddress]
    [StringLength(200)]
    public string Email { get; set; } = string.Empty;

    [Required]
    [StringLength(100, MinimumLength = 12)]
    public string Password { get; set; } = string.Empty;

    [Range(18, 120)]
    public int Age { get; set; }
}

// ✅ FluentValidation (more expressive)
public class RegisterDtoValidator : AbstractValidator<RegisterDto>
{
    public RegisterDtoValidator()
    {
        RuleFor(x => x.Username)
            .NotEmpty()
            .Length(3, 50)
            .Matches(@"^[a-zA-Z0-9_]+$").WithMessage("Alphanumeric and underscore only");

        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress()
            .MaximumLength(200);

        RuleFor(x => x.Password)
            .NotEmpty()
            .MinimumLength(12)
            .Matches("[A-Z]").WithMessage("Must contain uppercase")
            .Matches("[0-9]").WithMessage("Must contain digit")
            .Matches("[^a-zA-Z0-9]").WithMessage("Must contain special character");
    }
}

// ✅ Validate in controller (automatic with [ApiController])
[HttpPost]
public async Task<IActionResult> Register([FromBody] RegisterDto dto)
{
    // ModelState.IsValid is checked automatically by [ApiController]
    // ...
}
```

---

## Secrets Management

**NEVER hardcode secrets in source code or appsettings.json committed to source control.**

```csharp
// ❌ NEVER - Hardcoded secrets
var connectionString = "Server=prod-db;Password=SuperSecret123!";
var apiKey = "sk-live-abcdefghijk1234567890";

// ✅ Development - User Secrets (never committed to git)
// dotnet user-secrets init
// dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=..."
// dotnet user-secrets set "Jwt:SecretKey" "my-super-secret-key-min-256-bits!"

// ✅ Production - Environment Variables
var connectionString = Environment.GetEnvironmentVariable("DB_CONNECTION_STRING");

// ✅ Production - Azure Key Vault
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{builder.Configuration["KeyVaultName"]}.vault.azure.net/"),
    new DefaultAzureCredential());

// Read secret transparently
var secret = builder.Configuration["MySecret"];

// ✅ Ensure secrets are set at startup
var jwtSecret = builder.Configuration["Jwt:SecretKey"]
    ?? throw new InvalidOperationException("JWT secret key is not configured");
```

### .gitignore for Secrets

```gitignore
# NEVER commit these
appsettings.Production.json
appsettings.Secrets.json
*.pfx
*.p12
.env
secrets.json
```

---

## Security Checklist

### Authentication & Authorization

```
✅ Use ASP.NET Core Identity or proven auth library
✅ Implement MFA for sensitive operations
✅ Use [Authorize] on all endpoints that require auth
✅ Implement role/policy-based authorization
✅ Short JWT expiry + refresh token rotation
✅ Account lockout after failed attempts
✅ Validate JWT issuer, audience, expiry
```

### Data Protection

```
✅ Never store plain-text passwords — use BCrypt/Argon2/PBKDF2
✅ Parameterized queries / EF Core for all DB access
✅ Encrypt sensitive data at rest (Azure Storage encryption, TDE)
✅ Use HTTPS everywhere, enforce with HSTS
✅ Never log passwords, tokens, or sensitive PII
```

### Headers & Transport

```
✅ Add all security headers (X-Frame-Options, CSP, HSTS, etc.)
✅ SameSite=Strict on auth cookies
✅ HttpOnly + Secure flags on all cookies
✅ Disable TLS 1.0/1.1 — use TLS 1.2+
✅ Validate and sanitize all user input
```

### Dependencies & Config

```
✅ Keep NuGet packages up to date
✅ Run: dotnet list package --vulnerable
✅ Store secrets in Key Vault / environment variables
✅ Disable swagger in production (or protect it)
✅ Remove default/sample data in production
✅ Disable detailed error pages in production
```

---

## Quick Reference

```csharp
// ─── SQL Injection Prevention ────────────────────────────────────────
// EF Core (safe by default)
var user = await db.Users.Where(u => u.Email == email).FirstOrDefaultAsync();
// Dapper (parameterized)
var user = await conn.QueryFirstOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Email = @Email", new { Email = email });

// ─── Password Hashing ────────────────────────────────────────────────
var hash = BCrypt.HashPassword(password, workFactor: 12);
var valid = BCrypt.Verify(password, hash);

// ─── Enforce HTTPS ───────────────────────────────────────────────────
app.UseHsts();
app.UseHttpsRedirection();

// ─── Antiforgery ─────────────────────────────────────────────────────
builder.Services.AddControllersWithViews(o =>
    o.Filters.Add(new AutoValidateAntiforgeryTokenAttribute()));

// ─── Secure Cookie ───────────────────────────────────────────────────
options.Cookie.HttpOnly = true;
options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
options.Cookie.SameSite = SameSiteMode.Strict;

// ─── Input Validation ────────────────────────────────────────────────
[Required][StringLength(100)][EmailAddress] public string Email { get; set; }

// ─── Secrets ─────────────────────────────────────────────────────────
var secret = config["Jwt:SecretKey"]
    ?? throw new InvalidOperationException("Secret not configured");
```

---

**Guide Complete!** This comprehensive security guide covers OWASP Top 10, SQL injection, XSS, CSRF, password storage, HTTPS, security headers, JWT hardening, input validation, and secrets management — everything you need to build secure .NET applications! 🔐
