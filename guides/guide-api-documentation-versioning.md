# API Documentation & Versioning Quick Reference

---

## Why API Documentation Matters

**Good API docs** = Developers can use your API without asking questions

**Benefits:**
- ✅ **Faster onboarding** - New developers self-serve
- ✅ **Reduced support** - Less "how do I call this endpoint?"
- ✅ **Client generation** - Auto-generate SDKs from OpenAPI spec
- ✅ **Contract testing** - Validate API behavior against spec
- ✅ **Discoverability** - Explore all endpoints in one place

---

## OpenAPI / Swagger

**OpenAPI** = Industry standard specification for describing REST APIs (formerly Swagger Spec)

**Swagger** = Tooling built around OpenAPI (Swagger UI, Swagger Editor, etc.)

```
Your .NET App
    ↓
OpenAPI Spec (JSON/YAML)   ← Machine-readable description
    ↓
Swagger UI                 ← Interactive browser UI
    ↓
Client SDKs                ← Auto-generated (C#, JS, Python...)
```

### Built-in OpenAPI (.NET 9)

```bash
# .NET 9+ has built-in OpenAPI support (no extra NuGet needed)
# For .NET 8 and below, use Swashbuckle
```

### Swashbuckle Setup (.NET 8 and below)

```bash
dotnet add package Swashbuckle.AspNetCore
```

```csharp
// Program.cs
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1",
        Description = "A comprehensive API for managing resources",
        Contact = new OpenApiContact
        {
            Name = "API Support",
            Email = "api@mycompany.com",
            Url = new Uri("https://mycompany.com/support")
        },
        License = new OpenApiLicense
        {
            Name = "MIT",
            Url = new Uri("https://opensource.org/licenses/MIT")
        }
    });

    // ✅ Include XML documentation comments
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});

var app = builder.Build();

// Serve Swagger UI in development (or with auth in production)
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/swagger/v1/swagger.json", "My API v1");
        options.RoutePrefix = string.Empty;       // Serve at root URL
        options.DocumentTitle = "My API Docs";
        options.DefaultModelsExpandDepth(-1);     // Hide schemas section by default
    });
}
```

### Swagger with JWT Authentication

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });

    // ✅ Add JWT auth button to Swagger UI
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Enter JWT token (without 'Bearer ' prefix)"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });

    // Include XML comments
    var xmlPath = Path.Combine(AppContext.BaseDirectory, $"{Assembly.GetExecutingAssembly().GetName().Name}.xml");
    options.IncludeXmlComments(xmlPath);
});
```

---

## XML Documentation Comments

Enable XML docs to auto-populate Swagger descriptions.

### Enable XML Documentation

```xml
<!-- MyApp.API.csproj -->
<PropertyGroup>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <NoWarn>$(NoWarn);1591</NoWarn>  <!-- Suppress warnings for undocumented members -->
</PropertyGroup>
```

### Controller Documentation

```csharp
/// <summary>
/// Manages product resources in the catalog.
/// </summary>
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    /// <summary>
    /// Retrieves a paginated list of all active products.
    /// </summary>
    /// <param name="page">Page number (1-based). Default: 1</param>
    /// <param name="pageSize">Number of items per page (max 100). Default: 20</param>
    /// <returns>A paginated list of products</returns>
    /// <response code="200">Successfully retrieved products</response>
    /// <response code="400">Invalid pagination parameters</response>
    /// <response code="401">Authentication required</response>
    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<ProductDto>), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status401Unauthorized)]
    public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20)
    {
        // ...
    }

    /// <summary>
    /// Retrieves a specific product by its unique identifier.
    /// </summary>
    /// <param name="id">The unique identifier of the product</param>
    /// <returns>The requested product</returns>
    /// <response code="200">Product found and returned</response>
    /// <response code="404">Product not found</response>
    [HttpGet("{id:int}")]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        // ...
    }

    /// <summary>
    /// Creates a new product in the catalog.
    /// </summary>
    /// <param name="dto">The product creation details</param>
    /// <returns>The newly created product</returns>
    /// <remarks>
    /// Sample request:
    ///
    ///     POST /api/products
    ///     {
    ///         "name": "Wireless Headphones",
    ///         "price": 99.99,
    ///         "categoryId": 5
    ///     }
    ///
    /// </remarks>
    /// <response code="201">Product created successfully</response>
    /// <response code="400">Invalid product data</response>
    /// <response code="409">Product with same SKU already exists</response>
    [HttpPost]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ValidationProblemDetails), StatusCodes.Status400BadRequest)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status409Conflict)]
    public async Task<ActionResult<ProductDto>> CreateProduct([FromBody] CreateProductDto dto)
    {
        // ...
    }

    /// <summary>
    /// Deletes a product from the catalog.
    /// </summary>
    /// <param name="id">The unique identifier of the product to delete</param>
    /// <response code="204">Product deleted successfully</response>
    /// <response code="404">Product not found</response>
    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        // ...
    }
}
```

### DTO Documentation

```csharp
/// <summary>
/// Data transfer object for creating a new product.
/// </summary>
public class CreateProductDto
{
    /// <summary>
    /// The display name of the product. Must be unique within the category.
    /// </summary>
    /// <example>Wireless Noise-Cancelling Headphones</example>
    [Required]
    [StringLength(200, MinimumLength = 3)]
    public string Name { get; set; } = string.Empty;

    /// <summary>
    /// Detailed description of the product.
    /// </summary>
    /// <example>Premium over-ear headphones with 30-hour battery life</example>
    [StringLength(2000)]
    public string? Description { get; set; }

    /// <summary>
    /// The retail price in USD. Must be greater than 0.
    /// </summary>
    /// <example>99.99</example>
    [Required]
    [Range(0.01, 999999.99)]
    public decimal Price { get; set; }

    /// <summary>
    /// The category this product belongs to.
    /// </summary>
    /// <example>5</example>
    [Required]
    public int CategoryId { get; set; }

    /// <summary>
    /// Stock keeping unit — must be unique across all products.
    /// </summary>
    /// <example>WH-NC-001</example>
    [Required]
    [StringLength(50)]
    public string Sku { get; set; } = string.Empty;
}

/// <summary>
/// Paginated result wrapper for list endpoints.
/// </summary>
/// <typeparam name="T">The type of items in the result</typeparam>
public class PagedResult<T>
{
    /// <summary>List of items for the current page</summary>
    public IEnumerable<T> Items { get; set; } = [];

    /// <summary>Total number of items across all pages</summary>
    /// <example>150</example>
    public int TotalCount { get; set; }

    /// <summary>Current page number (1-based)</summary>
    /// <example>1</example>
    public int Page { get; set; }

    /// <summary>Number of items per page</summary>
    /// <example>20</example>
    public int PageSize { get; set; }

    /// <summary>Total number of pages</summary>
    /// <example>8</example>
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
}
```

### Useful Swagger Attributes

```csharp
// ✅ Group endpoints in Swagger UI
[ApiExplorerSettings(GroupName = "Products")]
public class ProductsController : ControllerBase { }

// ✅ Ignore internal endpoints from Swagger
[ApiExplorerSettings(IgnoreApi = true)]
[HttpGet("internal/health-detailed")]
public IActionResult InternalHealth() => Ok();

// ✅ Declare content types
[Produces("application/json")]
[Consumes("application/json")]

// ✅ Mark endpoint as obsolete (shows warning in Swagger UI)
[Obsolete]
[HttpGet("old-endpoint")]

// ✅ Swagger UI operation ID (for SDK generation)
[HttpGet("{id}")]
[SwaggerOperation(
    Summary = "Get product by ID",
    Description = "Retrieves full product details",
    OperationId = "GetProductById",
    Tags = new[] { "Products" })]
```

---

## API Versioning Strategies

When your API evolves, you need versioning to avoid breaking existing clients.

### Strategies Comparison

```
Strategy          Example                          Best For
──────────────    ─────────────────────────────    ──────────────────────────
URL Path          /api/v1/products                 Public APIs (most common)
Query String      /api/products?api-version=1.0    Simple internal APIs
Header            api-version: 1.0                 Clean URLs, enterprise APIs
Media Type        Accept: application/json;ver=1   Strict REST purists
```

### Install Packages

```bash
dotnet add package Asp.Versioning.Mvc
dotnet add package Asp.Versioning.Mvc.ApiExplorer   # For Swagger integration
```

### Setup — URL Path Versioning (Most Common)

```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);    // Default to v1.0
    options.AssumeDefaultVersionWhenUnspecified = true;  // Use default if unspecified
    options.ReportApiVersions = true;                    // Add api-supported-versions header
    options.ApiVersionReader = new UrlSegmentApiVersionReader(); // Read from URL
})
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";           // v1, v1.1, v2
    options.SubstituteApiVersionInUrl = true;     // Replace {version} in route
});
```

### Multiple Readers (Support All Strategies)

```csharp
options.ApiVersionReader = ApiVersionReader.Combine(
    new UrlSegmentApiVersionReader(),
    new HeaderApiVersionReader("api-version"),
    new QueryStringApiVersionReader("api-version"));
// Client can use any of the three methods
```

### Versioned Controllers

```csharp
// ─── v1 Controller ─────────────────────────────────────────────────
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class ProductsController : ControllerBase
{
    /// <summary>Gets all products (v1 - basic fields)</summary>
    [HttpGet]
    public async Task<ActionResult<IEnumerable<ProductDtoV1>>> GetProducts()
    {
        return Ok(new[] { new ProductDtoV1 { Id = 1, Name = "Widget" } });
    }

    /// <summary>Gets a product by ID</summary>
    [HttpGet("{id:int}")]
    public async Task<ActionResult<ProductDtoV1>> GetProduct(int id)
    {
        // v1 implementation
    }
}

// ─── v2 Controller ─────────────────────────────────────────────────
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("2.0")]
public class ProductsV2Controller : ControllerBase
{
    /// <summary>Gets all products (v2 - enriched with images, inventory)</summary>
    [HttpGet]
    public async Task<ActionResult<IEnumerable<ProductDtoV2>>> GetProducts()
    {
        return Ok(new[] { new ProductDtoV2 { Id = 1, Name = "Widget", ImageUrl = "..." } });
    }
}
```

### Multiple Versions on Same Controller

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class CategoriesController : ControllerBase
{
    // Available in both v1 and v2 (unchanged)
    [HttpGet]
    public async Task<IActionResult> GetCategories() => Ok();

    // Only available in v2
    [HttpGet("tree")]
    [MapToApiVersion("2.0")]
    public async Task<IActionResult> GetCategoryTree() => Ok();

    // Only available in v1 (removed in v2)
    [HttpGet("old-format")]
    [MapToApiVersion("1.0")]
    public async Task<IActionResult> GetCategoriesOldFormat() => Ok();
}
```

### Deprecating Old Versions

```csharp
// Mark v1 as deprecated - shows warning in Swagger, returns deprecation header
[ApiVersion("1.0", Deprecated = true)]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetV1()
    {
        // Optionally add sunset header for explicit deprecation date
        Response.Headers.Add("Sunset", "Sat, 31 Dec 2025 23:59:59 GMT");
        Response.Headers.Add("Deprecation", "true");
        return Ok();
    }
}
```

---

## Versioned Swagger Docs

Show separate Swagger UI pages for each API version.

```csharp
// Program.cs
builder.Services.AddSwaggerGen(options =>
{
    // ✅ Create a doc for each version
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
    options.SwaggerDoc("v2", new OpenApiInfo { Title = "My API", Version = "v2" });

    var xmlPath = Path.Combine(AppContext.BaseDirectory,
        $"{Assembly.GetExecutingAssembly().GetName().Name}.xml");
    options.IncludeXmlComments(xmlPath);
});

// ✅ Use Asp.Versioning.Mvc.ApiExplorer to auto-configure swagger docs
builder.Services.AddSwaggerGen();
builder.Services.ConfigureOptions<ConfigureSwaggerOptions>(); // Custom options class

// Swagger middleware
app.UseSwagger();
app.UseSwaggerUI(options =>
{
    var descriptions = app.DescribeApiVersions();
    foreach (var description in descriptions)
    {
        var url = $"/swagger/{description.GroupName}/swagger.json";
        var name = description.GroupName.ToUpperInvariant();
        options.SwaggerEndpoint(url, name);
    }
});
```

```csharp
// ConfigureSwaggerOptions.cs (auto-add doc per version)
public class ConfigureSwaggerOptions : IConfigureOptions<SwaggerGenOptions>
{
    private readonly IApiVersionDescriptionProvider _provider;

    public ConfigureSwaggerOptions(IApiVersionDescriptionProvider provider)
    {
        _provider = provider;
    }

    public void Configure(SwaggerGenOptions options)
    {
        foreach (var description in _provider.ApiVersionDescriptions)
        {
            options.SwaggerDoc(description.GroupName, new OpenApiInfo
            {
                Title = "My API",
                Version = description.ApiVersion.ToString(),
                Description = description.IsDeprecated
                    ? "⚠️ This API version is deprecated. Please upgrade to the latest version."
                    : "The current stable API version."
            });
        }
    }
}
```

---

## Header Versioning

```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new HeaderApiVersionReader("api-version");
});

// ─── Controller uses same [ApiVersion] attributes ───────────────────
[ApiController]
[Route("api/[controller]")]  // No version in URL
[ApiVersion("1.0")]
public class ProductsController : ControllerBase { }
```

```bash
# Client calls with header
curl -H "api-version: 1.0" https://api.example.com/api/products
curl -H "api-version: 2.0" https://api.example.com/api/products
```

---

## Minimal API Versioning

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.ApiVersionReader = new UrlSegmentApiVersionReader();
})
.AddApiExplorer(options => options.GroupNameFormat = "'v'VVV");

var app = builder.Build();

// Version group
var v1 = app.NewApiVersionSet()
    .HasApiVersion(new ApiVersion(1, 0))
    .HasDeprecatedApiVersion(new ApiVersion(0, 9))
    .ReportApiVersions()
    .Build();

var v2 = app.NewApiVersionSet()
    .HasApiVersion(new ApiVersion(2, 0))
    .ReportApiVersions()
    .Build();

// Versioned minimal endpoints
app.MapGet("/api/v{version:apiVersion}/products", () =>
{
    return Results.Ok(new[] { "Product A", "Product B" });
})
.WithApiVersionSet(v1)
.MapToApiVersion(new ApiVersion(1, 0));

app.MapGet("/api/v{version:apiVersion}/products", () =>
{
    return Results.Ok(new[] { new { Name = "Product A", ImageUrl = "..." } });
})
.WithApiVersionSet(v2)
.MapToApiVersion(new ApiVersion(2, 0));
```

---

## Best Practices

### Documentation

```
✅ Document every public endpoint with XML comments
✅ Include example values on all DTO properties (<example> tags)
✅ Use [ProducesResponseType] for every possible response code
✅ Add sample request bodies using <remarks> in XML comments
✅ Group related endpoints with [ApiExplorerSettings(GroupName = "...")]
✅ Secure Swagger UI in production (behind auth or disable entirely)
✅ Keep descriptions clear and concise — write for consumers, not implementors
```

### Versioning

```
✅ Version from day one — retrofitting versioning is painful
✅ Use URL path versioning for public APIs (most discoverable)
✅ Default to latest stable version when unspecified
✅ Mark deprecated versions explicitly with Deprecated = true
✅ Add Sunset header with deprecation date
✅ Maintain deprecated versions for at least 6-12 months
✅ Communicate breaking changes in release notes
❌ Never make breaking changes to an existing version
❌ Don't version every endpoint change — only breaking changes
```

---

## Quick Reference

```csharp
// ─── Swagger Setup ───────────────────────────────────────────────────
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(o => {
    o.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
    o.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, "MyApp.xml"));
});
app.UseSwagger();
app.UseSwaggerUI();

// ─── XML Doc Comment ─────────────────────────────────────────────────
/// <summary>Brief description</summary>
/// <param name="id">Parameter description</param>
/// <returns>What it returns</returns>
/// <response code="200">Success description</response>

// ─── Response Type Attributes ────────────────────────────────────────
[ProducesResponseType(typeof(ProductDto), 200)]
[ProducesResponseType(typeof(ProblemDetails), 404)]
[ProducesResponseType(401)]

// ─── Versioning ──────────────────────────────────────────────────────
builder.Services.AddApiVersioning(o => {
    o.DefaultApiVersion = new ApiVersion(1, 0);
    o.AssumeDefaultVersionWhenUnspecified = true;
    o.ApiVersionReader = new UrlSegmentApiVersionReader();
});

// On controller:
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
// On action:
[MapToApiVersion("2.0")]
// Deprecate:
[ApiVersion("1.0", Deprecated = true)]
```

---

**Guide Complete!** This comprehensive guide covers Swagger/OpenAPI setup, JWT auth in Swagger, XML documentation comments, all API versioning strategies, versioned Swagger UIs, and best practices for documenting and versioning .NET APIs! 📖
