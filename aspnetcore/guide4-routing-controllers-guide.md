# ASP.NET Core Routing & Controllers - Complete Guide
## Practical Guide + Technical Reference

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. What is Routing
2. Routing System Overview
3. Creating Routes (3 Methods)
4. Controllers Deep Dive
5. Action Results & Return Types
6. Model Binding & Parameters
7. Common Routing Patterns
8. Troubleshooting Common Issues
9. Best Practices

### Part 2: Technical Reference (Deep Dive)
10. Important Interfaces & Classes Reference
11. Configuration Deep-Dive
12. Advanced Routing Topics
13. Performance Tips

---

# PART 1: PRACTICAL GUIDE

---

## 1. What is Routing?

**Simple Definition:** The process of matching incoming HTTP requests to endpoints (controllers/handlers).

**Think of it like:** A postal service routing letters to addresses. The URL is the address, routing finds the right mailbox (endpoint).

```
Request: GET /api/users/5
    ↓
Routing System matches pattern: /api/users/{id}
    ↓
Routes to: UsersController.GetUser(id: 5)
    ↓
Response: User data
```

**Key Concepts:**
- **Route Template:** Pattern like `/api/users/{id}`
- **Route Parameters:** Values extracted from URL (`{id}`)
- **Endpoint:** The code that handles the request

---

## 2. Routing System Overview

### Endpoint Routing (ASP.NET Core 3.0+) ⭐ Current Standard

**Two-Phase System:**

**Phase 1: Routing** - Match URL to endpoint
```csharp
app.UseRouting(); // Enable routing
```

**Phase 2: Execution** - Run the matched endpoint
```csharp
app.MapControllers(); // Map controller endpoints
```

### Visual Flow

```
Request: GET /api/users/5
    ↓
UseRouting() → Matches route template
    ↓
[Middleware can run here - knows which endpoint]
    ↓
UseAuthentication() → Check user
    ↓
UseAuthorization() → Check permissions
    ↓
MapControllers() → Execute endpoint
    ↓
Response: User data
```

**Why Two Phases?**
- Middleware can see which endpoint will run
- Authentication/authorization know what's being accessed
- Better performance and flexibility

---

## 3. Creating Routes (3 Methods)

### Method 1: Minimal APIs (Quick & Modern) ✨ .NET 6+

**When to use:**
- ✅ Simple APIs (microservices)
- ✅ Prototyping
- ✅ Small projects
- ❌ Large applications (use controllers)

**Step 1: Basic Routes**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// GET /hello
app.MapGet("/hello", () => "Hello World!");

// GET /users
app.MapGet("/users", () => new[] { "Alice", "Bob", "Charlie" });

// GET /users/5
app.MapGet("/users/{id}", (int id) => $"User {id}");

// POST /users
app.MapPost("/users", (User user) => Results.Created($"/users/{user.Id}", user));

// PUT /users/5
app.MapPut("/users/{id}", (int id, User user) => Results.Ok(user));

// DELETE /users/5
app.MapDelete("/users/{id}", (int id) => Results.NoContent());

app.Run();

record User(int Id, string Name, string Email);
```

**Step 2: Complete CRUD API**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// In-memory storage (for demo)
var users = new List<User>
{
    new(1, "Alice", "alice@example.com"),
    new(2, "Bob", "bob@example.com")
};

// GET /api/users - List all users
app.MapGet("/api/users", () => Results.Ok(users));

// GET /api/users/5 - Get user by ID
app.MapGet("/api/users/{id:int}", (int id) =>
{
    var user = users.FirstOrDefault(u => u.Id == id);
    return user is not null ? Results.Ok(user) : Results.NotFound();
});

// POST /api/users - Create user
app.MapPost("/api/users", (User user) =>
{
    users.Add(user);
    return Results.Created($"/api/users/{user.Id}", user);
});

// PUT /api/users/5 - Update user
app.MapPut("/api/users/{id:int}", (int id, User user) =>
{
    var index = users.FindIndex(u => u.Id == id);
    if (index == -1) return Results.NotFound();
    
    users[index] = user with { Id = id };
    return Results.Ok(users[index]);
});

// DELETE /api/users/5 - Delete user
app.MapDelete("/api/users/{id:int}", (int id) =>
{
    var removed = users.RemoveAll(u => u.Id == id);
    return removed > 0 ? Results.NoContent() : Results.NotFound();
});

app.Run();

record User(int Id, string Name, string Email);
```

**Step 3: Route Groups** ✨ .NET 7+

```csharp
var app = builder.Build();

// Group related routes
var usersApi = app.MapGroup("/api/users");

usersApi.MapGet("", () => users);
usersApi.MapGet("{id:int}", (int id) => { /* ... */ });
usersApi.MapPost("", (User user) => { /* ... */ });
usersApi.MapPut("{id:int}", (int id, User user) => { /* ... */ });
usersApi.MapDelete("{id:int}", (int id) => { /* ... */ });

// Add filters to entire group
usersApi.RequireAuthorization();

app.Run();
```

---

### Method 2: Attribute Routing (Production Standard) ⭐ RECOMMENDED

**When to use:**
- ✅ Production applications
- ✅ Large projects
- ✅ Clear route organization
- ✅ Team development

**Step 1: Create Controller**

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    // GET api/users
    [HttpGet]
    public IActionResult GetUsers()
    {
        var users = new[] { "Alice", "Bob", "Charlie" };
        return Ok(users);
    }
    
    // GET api/users/5
    [HttpGet("{id}")]
    public IActionResult GetUser(int id)
    {
        return Ok(new { id, name = "Alice" });
    }
    
    // POST api/users
    [HttpPost]
    public IActionResult CreateUser([FromBody] User user)
    {
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
    
    // PUT api/users/5
    [HttpPut("{id}")]
    public IActionResult UpdateUser(int id, [FromBody] User user)
    {
        return Ok(user);
    }
    
    // DELETE api/users/5
    [HttpDelete("{id}")]
    public IActionResult DeleteUser(int id)
    {
        return NoContent();
    }
}

public record User(int Id, string Name, string Email);
```

**Step 2: Register Controllers in Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add controllers
builder.Services.AddControllers();

var app = builder.Build();

// Map controllers
app.MapControllers();

app.Run();
```

**Step 3: Complete CRUD Controller**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private static List<User> _users = new()
    {
        new(1, "Alice", "alice@example.com"),
        new(2, "Bob", "bob@example.com")
    };
    
    private readonly ILogger<UsersController> _logger;
    
    public UsersController(ILogger<UsersController> logger)
    {
        _logger = logger;
    }
    
    /// <summary>
    /// Get all users
    /// </summary>
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public ActionResult<IEnumerable<User>> GetUsers()
    {
        _logger.LogInformation("Getting all users");
        return Ok(_users);
    }
    
    /// <summary>
    /// Get user by ID
    /// </summary>
    [HttpGet("{id:int}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public ActionResult<User> GetUser(int id)
    {
        var user = _users.FirstOrDefault(u => u.Id == id);
        
        if (user is null)
        {
            _logger.LogWarning("User {Id} not found", id);
            return NotFound();
        }
        
        return Ok(user);
    }
    
    /// <summary>
    /// Create new user
    /// </summary>
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public ActionResult<User> CreateUser([FromBody] User user)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        _users.Add(user);
        _logger.LogInformation("Created user {Id}", user.Id);
        
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
    
    /// <summary>
    /// Update existing user
    /// </summary>
    [HttpPut("{id:int}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public ActionResult<User> UpdateUser(int id, [FromBody] User user)
    {
        var index = _users.FindIndex(u => u.Id == id);
        
        if (index == -1)
            return NotFound();
        
        _users[index] = user with { Id = id };
        _logger.LogInformation("Updated user {Id}", id);
        
        return Ok(_users[index]);
    }
    
    /// <summary>
    /// Delete user
    /// </summary>
    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public IActionResult DeleteUser(int id)
    {
        var removed = _users.RemoveAll(u => u.Id == id);
        
        if (removed == 0)
            return NotFound();
        
        _logger.LogInformation("Deleted user {Id}", id);
        return NoContent();
    }
    
    /// <summary>
    /// Search users by name
    /// </summary>
    [HttpGet("search")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public ActionResult<IEnumerable<User>> SearchUsers([FromQuery] string name)
    {
        var results = _users.Where(u => 
            u.Name.Contains(name, StringComparison.OrdinalIgnoreCase));
        
        return Ok(results);
    }
}
```

---

### Method 3: Convention Routing (Legacy MVC)

**When to use:**
- ⚠️ MVC applications with views
- ⚠️ Legacy applications
- ❌ New APIs (use attribute routing)

**Setup:**

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();

var app = builder.Build();

// Convention-based routing
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

**Example Routes:**
```
/                          → HomeController.Index()
/Home/Index                → HomeController.Index()
/Products/Details/5        → ProductsController.Details(id: 5)
/Admin/Users/Edit/10       → Admin.UsersController.Edit(id: 10)
```

---

### Comparison: Which Method?

| Feature | Minimal APIs | Attribute Routing | Convention Routing |
|---------|-------------|-------------------|-------------------|
| Complexity | Simple | Medium | Medium |
| Best For | Small APIs, Prototypes | Production APIs | MVC with Views |
| Scalability | Low | High | Medium |
| Discoverability | Low | High | Medium |
| Type Safety | Medium | High | High |
| Performance | Fastest | Fast | Fast |
| .NET Version | 6.0+ | 2.0+ | 1.0+ |

**Decision Tree:**
```
Building an API?
├─ YES → Small API (<10 endpoints)?
│        ├─ YES → Minimal APIs
│        └─ NO  → Attribute Routing ⭐
│
└─ NO  → Building MVC app with views?
         └─ YES → Convention Routing
```

---

## 4. Controllers Deep Dive

### ControllerBase vs Controller

| Class | Inherit From | Use For | Has View Support |
|-------|-------------|---------|------------------|
| `ControllerBase` | - | APIs (JSON) | No |
| `Controller` | ControllerBase | MVC (Views) | Yes |

**ControllerBase** (API Controllers) ⭐

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // API methods - return data (JSON)
    [HttpGet]
    public IActionResult Get() => Ok(new { message = "Hello" });
}
```

**Controller** (MVC with Views)

```csharp
public class HomeController : Controller
{
    // Returns views (HTML)
    public IActionResult Index() => View();
    
    public IActionResult About() => View();
}
```

---

### Route Attributes

**[Route] Attribute** - Define route template

```csharp
// Class-level route
[Route("api/products")]
public class ProductsController : ControllerBase
{
    // Combines: api/products
    [HttpGet]
    public IActionResult GetAll() { }
    
    // Combines: api/products/5
    [HttpGet("{id}")]
    public IActionResult Get(int id) { }
}
```

**Token Replacement:**

```csharp
// [controller] → class name without "Controller"
[Route("api/[controller]")]
public class UsersController { } // Route: api/users

// [action] → method name
[Route("[action]")]
public IActionResult GetUser() { } // Route: GetUser
```

**HTTP Method Attributes:**

| Attribute | HTTP Method | Common Use |
|-----------|-------------|------------|
| `[HttpGet]` | GET | Read data |
| `[HttpPost]` | POST | Create data |
| `[HttpPut]` | PUT | Update (full) |
| `[HttpPatch]` | PATCH | Update (partial) |
| `[HttpDelete]` | DELETE | Delete data |
| `[HttpHead]` | HEAD | Headers only |
| `[HttpOptions]` | OPTIONS | Supported methods |

**Combined Routes:**

```csharp
// Multiple routes to same action
[HttpGet("")]           // GET api/users
[HttpGet("all")]        // GET api/users/all
public IActionResult GetUsers() { }

// Template override
[HttpGet("/special")]   // GET /special (ignores class route)
public IActionResult Special() { }
```

---

### [ApiController] Attribute ✨ ASP.NET Core 2.1+

**What it does:**
1. ✅ Automatic model validation
2. ✅ Infers parameter binding sources
3. ✅ Returns 400 for invalid models
4. ✅ Problem details for errors

**Without [ApiController]:**

```csharp
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create([FromBody] User user)
    {
        // ❌ Must manually check ModelState
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        // ❌ Must specify [FromBody]
        return Ok(user);
    }
}
```

**With [ApiController]:**

```csharp
[ApiController]  // ⭐ Add this!
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(User user)  // ✅ [FromBody] inferred
    {
        // ✅ ModelState automatically validated
        // ✅ Returns 400 if invalid
        return Ok(user);
    }
}
```

**Configure Behavior:**

```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        // Customize automatic 400 response
        options.InvalidModelStateResponseFactory = context =>
        {
            var errors = context.ModelState
                .Where(e => e.Value.Errors.Count > 0)
                .Select(e => new
                {
                    Field = e.Key,
                    Errors = e.Value.Errors.Select(x => x.ErrorMessage)
                });
            
            return new BadRequestObjectResult(new
            {
                Message = "Validation failed",
                Errors = errors
            });
        };
    });
```

---

## 5. Action Results & Return Types

### IActionResult - Flexible Return Type ⭐

**Common Action Results:**

| Method | Status Code | Use Case |
|--------|-------------|----------|
| `Ok(value)` | 200 | Success with data |
| `Created(uri, value)` | 201 | Resource created |
| `CreatedAtAction(...)` | 201 | Created (route to resource) |
| `NoContent()` | 204 | Success, no data |
| `BadRequest(error)` | 400 | Invalid request |
| `Unauthorized()` | 401 | Not authenticated |
| `Forbid()` | 403 | Not authorized |
| `NotFound()` | 404 | Resource not found |
| `Conflict(error)` | 409 | Conflict (duplicate) |
| `UnprocessableEntity(error)` | 422 | Validation failed |
| `StatusCode(code)` | Custom | Any status code |

**Examples:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // 200 OK
    [HttpGet]
    public IActionResult GetAll()
    {
        var products = new[] { "Product1", "Product2" };
        return Ok(products);
    }
    
    // 200 OK or 404 Not Found
    [HttpGet("{id}")]
    public IActionResult Get(int id)
    {
        var product = FindProduct(id);
        
        if (product is null)
            return NotFound();  // 404
        
        return Ok(product);     // 200
    }
    
    // 201 Created
    [HttpPost]
    public IActionResult Create(Product product)
    {
        // Save product...
        
        // Returns: 201 with Location header
        return CreatedAtAction(
            nameof(Get),
            new { id = product.Id },
            product);
    }
    
    // 204 No Content
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        // Delete product...
        return NoContent();
    }
    
    // 400 Bad Request
    [HttpPost("validate")]
    public IActionResult Validate(Product product)
    {
        if (product.Price < 0)
            return BadRequest("Price cannot be negative");
        
        return Ok();
    }
    
    // 409 Conflict
    [HttpPost("register")]
    public IActionResult Register(User user)
    {
        if (UserExists(user.Email))
            return Conflict("Email already registered");
        
        return Ok();
    }
}
```

---

### ActionResult<T> - Type-Safe Returns ✨ ASP.NET Core 2.1+

**Benefits:**
- ✅ Compile-time type safety
- ✅ Better documentation
- ✅ Cleaner code
- ✅ OpenAPI/Swagger support

**Examples:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    // Returns User or IActionResult
    [HttpGet("{id}")]
    public ActionResult<User> Get(int id)
    {
        var user = FindUser(id);
        
        if (user is null)
            return NotFound();  // IActionResult
        
        return user;  // Implicit conversion to Ok(user)
    }
    
    // Returns list or IActionResult
    [HttpGet]
    public ActionResult<List<User>> GetAll()
    {
        var users = GetUsers();
        return users;  // Implicit Ok(users)
    }
    
    // Created response
    [HttpPost]
    public ActionResult<User> Create(User user)
    {
        // Save user...
        
        return CreatedAtAction(nameof(Get), new { id = user.Id }, user);
    }
}
```

---

### Comparison: IActionResult vs ActionResult<T>

```csharp
// ❌ IActionResult - No type safety
[HttpGet("{id}")]
public IActionResult Get(int id)
{
    var user = FindUser(id);
    if (user is null) return NotFound();
    return Ok(user);  // Type of user not known
}

// ✅ ActionResult<T> - Type safe
[HttpGet("{id}")]
public ActionResult<User> Get(int id)
{
    var user = FindUser(id);
    if (user is null) return NotFound();
    return user;  // Compiler knows it's User
}
```

**When to Use:**

| Return Type | Use When |
|-------------|----------|
| `IActionResult` | Multiple different return types |
| `ActionResult<T>` | Primary return type with error cases ⭐ |
| `T` | Always returns data (no errors) |

---

### Results Class ✨ .NET 6+

**For Minimal APIs:**

```csharp
app.MapGet("/users/{id}", (int id) =>
{
    var user = FindUser(id);
    
    if (user is null)
        return Results.NotFound();
    
    return Results.Ok(user);
});

app.MapPost("/users", (User user) =>
{
    return Results.Created($"/users/{user.Id}", user);
});

app.MapDelete("/users/{id}", (int id) =>
{
    return Results.NoContent();
});
```

---

## 6. Model Binding & Parameters

### Parameter Binding Sources

**Automatic Binding (with [ApiController]):**

| Source | Attribute | Use For |
|--------|-----------|---------|
| URL route | `[FromRoute]` | `/users/{id}` |
| Query string | `[FromQuery]` | `?page=1&size=10` |
| Request body | `[FromBody]` | JSON payload |
| Form data | `[FromForm]` | Form submissions |
| Header | `[FromHeader]` | HTTP headers |
| Services | `[FromServices]` | DI services |

**Examples:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    // From route: /api/users/5
    [HttpGet("{id}")]
    public IActionResult Get(int id)  // [FromRoute] inferred
    {
        return Ok(new { id });
    }
    
    // From query: /api/users/search?name=Alice&page=1
    [HttpGet("search")]
    public IActionResult Search(string name, int page = 1)  // [FromQuery] inferred
    {
        return Ok(new { name, page });
    }
    
    // From body: POST with JSON
    [HttpPost]
    public IActionResult Create(User user)  // [FromBody] inferred
    {
        return Ok(user);
    }
    
    // From header: X-API-Key
    [HttpGet("secure")]
    public IActionResult Secure([FromHeader(Name = "X-API-Key")] string apiKey)
    {
        return Ok(new { apiKey });
    }
    
    // From services: DI
    [HttpGet("config")]
    public IActionResult GetConfig([FromServices] IConfiguration config)
    {
        var value = config["AppSettings:Key"];
        return Ok(new { value });
    }
    
    // Multiple sources
    [HttpPost("{id}")]
    public IActionResult Update(
        int id,                    // From route
        User user,                 // From body
        [FromQuery] bool notify,   // From query
        [FromHeader] string token) // From header
    {
        return Ok(new { id, user, notify, token });
    }
}
```

---

### Complex Binding Examples

**Binding Lists:**

```csharp
// Query: ?ids=1&ids=2&ids=3
[HttpGet("multiple")]
public IActionResult GetMultiple([FromQuery] List<int> ids)
{
    return Ok(ids);  // [1, 2, 3]
}

// Query: ?filters[0].name=Alice&filters[1].name=Bob
[HttpGet("filters")]
public IActionResult Filter([FromQuery] List<Filter> filters)
{
    return Ok(filters);
}

public class Filter
{
    public string Name { get; set; }
    public string Value { get; set; }
}
```

**Binding Complex Objects:**

```csharp
[HttpPost("search")]
public IActionResult Search([FromBody] SearchRequest request)
{
    // POST body:
    // {
    //   "query": "Alice",
    //   "filters": { "age": 25, "city": "NYC" },
    //   "pagination": { "page": 1, "size": 10 }
    // }
    
    return Ok(request);
}

public class SearchRequest
{
    public string Query { get; set; }
    public Dictionary<string, object> Filters { get; set; }
    public Pagination Pagination { get; set; }
}

public class Pagination
{
    public int Page { get; set; }
    public int Size { get; set; }
}
```

---

### Model Validation

**Data Annotations:**

```csharp
using System.ComponentModel.DataAnnotations;

public class CreateUserRequest
{
    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; }
    
    [Required]
    [EmailAddress]
    public string Email { get; set; }
    
    [Range(18, 100)]
    public int Age { get; set; }
    
    [Phone]
    public string PhoneNumber { get; set; }
    
    [Url]
    public string Website { get; set; }
    
    [RegularExpression(@"^[A-Z]{2}\d{6}$")]
    public string Code { get; set; }
    
    [Compare(nameof(Password))]
    public string ConfirmPassword { get; set; }
    
    public string Password { get; set; }
}
```

**Using Validation:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    // With [ApiController] - automatic validation
    [HttpPost]
    public IActionResult Create(CreateUserRequest request)
    {
        // If validation fails, returns 400 automatically
        // If we get here, model is valid
        return Ok(request);
    }
    
    // Manual validation (without [ApiController])
    [HttpPost("manual")]
    public IActionResult CreateManual(CreateUserRequest request)
    {
        if (!ModelState.IsValid)
        {
            var errors = ModelState.Values
                .SelectMany(v => v.Errors)
                .Select(e => e.ErrorMessage);
            
            return BadRequest(new { errors });
        }
        
        return Ok(request);
    }
}
```

---

## 7. Common Routing Patterns

### RESTful API Design

**Resource-Based URLs:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    // Collection endpoints
    [HttpGet]                              // GET    /api/orders
    public IActionResult GetOrders() { }
    
    [HttpPost]                             // POST   /api/orders
    public IActionResult CreateOrder() { }
    
    // Single resource endpoints
    [HttpGet("{id}")]                      // GET    /api/orders/5
    public IActionResult GetOrder(int id) { }
    
    [HttpPut("{id}")]                      // PUT    /api/orders/5
    public IActionResult UpdateOrder(int id) { }
    
    [HttpDelete("{id}")]                   // DELETE /api/orders/5
    public IActionResult DeleteOrder(int id) { }
    
    // Sub-resources
    [HttpGet("{id}/items")]                // GET    /api/orders/5/items
    public IActionResult GetOrderItems(int id) { }
    
    [HttpPost("{id}/items")]               // POST   /api/orders/5/items
    public IActionResult AddOrderItem(int id) { }
    
    // Actions on resource
    [HttpPost("{id}/cancel")]              // POST   /api/orders/5/cancel
    public IActionResult CancelOrder(int id) { }
    
    [HttpPost("{id}/ship")]                // POST   /api/orders/5/ship
    public IActionResult ShipOrder(int id) { }
}
```

---

### API Versioning

**Method 1: URL Versioning** (Most Common)

```csharp
// V1 Controller
[ApiController]
[Route("api/v1/[controller]")]
public class UsersV1Controller : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new { version = "1.0", data = "..." });
    }
}

// V2 Controller
[ApiController]
[Route("api/v2/[controller]")]
public class UsersV2Controller : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new { version = "2.0", data = "...", newField = "..." });
    }
}

// URLs:
// GET /api/v1/users
// GET /api/v2/users
```

**Method 2: Query String Versioning**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet]
    public IActionResult Get([FromQuery] string version = "1.0")
    {
        return version switch
        {
            "1.0" => Ok(new { version = "1.0", data = "..." }),
            "2.0" => Ok(new { version = "2.0", data = "...", newField = "..." }),
            _ => BadRequest("Unsupported version")
        };
    }
}

// URL: GET /api/users?version=2.0
```

**Method 3: Header Versioning**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet]
    public IActionResult Get([FromHeader(Name = "X-API-Version")] string version = "1.0")
    {
        return version switch
        {
            "1.0" => Ok(new { version = "1.0" }),
            "2.0" => Ok(new { version = "2.0" }),
            _ => BadRequest("Unsupported version")
        };
    }
}

// Request:
// GET /api/users
// X-API-Version: 2.0
```

---

### Route Constraints

**Built-in Constraints:**

| Constraint | Example | Matches |
|-----------|---------|---------|
| `int` | `{id:int}` | 1, 123, -5 |
| `long` | `{id:long}` | 123456789 |
| `decimal` | `{price:decimal}` | 10.5, 99.99 |
| `bool` | `{active:bool}` | true, false |
| `datetime` | `{date:datetime}` | 2024-01-15 |
| `guid` | `{id:guid}` | 32-char GUID |
| `alpha` | `{name:alpha}` | ABC, xyz |
| `min(n)` | `{age:min(18)}` | 18, 25, 100 |
| `max(n)` | `{age:max(100)}` | 1, 50, 100 |
| `range(min,max)` | `{age:range(18,100)}` | 18-100 |
| `length(n)` | `{code:length(6)}` | ABCDEF |
| `minlength(n)` | `{name:minlength(2)}` | AB, ABC |
| `maxlength(n)` | `{name:maxlength(50)}` | Up to 50 chars |
| `regex(expr)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | Pattern match |

**Examples:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Integer only: /api/products/5
    [HttpGet("{id:int}")]
    public IActionResult Get(int id) => Ok(new { id });
    
    // Minimum value: /api/products/18 (not /api/products/5)
    [HttpGet("age/{age:min(18)}")]
    public IActionResult GetByAge(int age) => Ok(new { age });
    
    // Range: /api/products/price/10.5 (10-1000)
    [HttpGet("price/{price:range(10,1000)}")]
    public IActionResult GetByPrice(decimal price) => Ok(new { price });
    
    // Alpha only: /api/products/category/electronics
    [HttpGet("category/{category:alpha}")]
    public IActionResult GetByCategory(string category) => Ok(new { category });
    
    // GUID: /api/products/123e4567-e89b-12d3-a456-426614174000
    [HttpGet("guid/{id:guid}")]
    public IActionResult GetByGuid(Guid id) => Ok(new { id });
    
    // Regex: /api/products/code/ABC123
    [HttpGet("code/{code:regex(^[A-Z]{{3}}\\d{{3}}$)}")]
    public IActionResult GetByCode(string code) => Ok(new { code });
    
    // Multiple constraints: /api/products/5/ABC
    [HttpGet("{id:int:min(1)}/{name:alpha:minlength(2)}")]
    public IActionResult GetComplex(int id, string name) => Ok(new { id, name });
}
```

---

### Catch-All Routes

```csharp
[ApiController]
[Route("api/[controller]")]
public class FilesController : ControllerBase
{
    // Catch-all: /api/files/documents/2024/report.pdf
    [HttpGet("{**path}")]
    public IActionResult GetFile(string path)
    {
        // path = "documents/2024/report.pdf"
        return Ok(new { path });
    }
}
```

---

### Area Routing (Large Applications)

```csharp
// Areas/Admin/Controllers/UsersController.cs
[Area("Admin")]
[Route("[area]/[controller]")]
public class UsersController : Controller
{
    [HttpGet]
    public IActionResult Index() => View();
}

// Program.cs
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

// URL: /Admin/Users
```

---

## 8. Troubleshooting Common Issues

### Issue 1: Route Conflicts

**Problem:**
```csharp
[HttpGet("{id}")]           // /api/users/5
public IActionResult Get(int id) { }

[HttpGet("active")]         // ❌ /api/users/active conflicts!
public IActionResult GetActive() { }
```

**Solution: Order matters!**
```csharp
[HttpGet("active")]         // ✅ More specific first
public IActionResult GetActive() { }

[HttpGet("{id:int}")]       // ✅ Add constraint
public IActionResult Get(int id) { }
```

---

### Issue 2: Missing MapControllers()

**Problem:**
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();
// ❌ Forgot app.MapControllers()
app.Run();
```

**Solution:**
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();  // ✅ Add this!
app.Run();
```

---

### Issue 3: Wrong HTTP Method

**Problem:**
```csharp
// POST to GET endpoint
[HttpGet]
public IActionResult Create(User user) { }  // ❌ Should be POST
```

**Solution:**
```csharp
[HttpPost]  // ✅ Correct method
public IActionResult Create(User user) { }
```

---

### Issue 4: Model Binding Fails

**Problem:**
```csharp
// POST with JSON body
[HttpPost]
public IActionResult Create(int id, string name)  // ❌ Not bound
{
    // id and name are 0 and null
}
```

**Solution:**
```csharp
// Create model
public class CreateRequest
{
    public int Id { get; set; }
    public string Name { get; set; }
}

[HttpPost]
public IActionResult Create(CreateRequest request)  // ✅ Binds from body
{
    return Ok(request);
}

// Or explicit binding
[HttpPost]
public IActionResult Create([FromBody] CreateRequest request)
{
    return Ok(request);
}
```

---

## 9. Best Practices

### API Design

- ✅ Use attribute routing for APIs
- ✅ Use `[ApiController]` attribute
- ✅ Return `ActionResult<T>` for type safety
- ✅ Use proper HTTP methods (GET, POST, PUT, DELETE)
- ✅ Return correct status codes (200, 201, 404, etc.)
- ✅ Version your APIs (URL versioning recommended)
- ✅ Use route constraints for validation

### Route Templates

- ✅ Use lowercase URLs: `api/users` not `API/Users`
- ✅ Use plural nouns: `users` not `user`
- ✅ Use hyphens for readability: `order-items` not `orderitems`
- ✅ Keep routes simple and predictable
- ✅ Order specific routes before generic routes
- ✅ Use constraints to avoid conflicts

### Controllers

- ✅ One controller per resource (Users, Products, Orders)
- ✅ Keep controllers thin (use services)
- ✅ Use async methods for I/O operations
- ✅ Add XML comments for Swagger/OpenAPI
- ✅ Use `[ProducesResponseType]` for documentation
- ✅ Inject dependencies via constructor

---

# PART 2: TECHNICAL REFERENCE

---

## 10. Important Interfaces & Classes Reference

### ControllerBase Class ⭐⭐⭐

**Purpose:** Base class for API controllers (no view support)

**Namespace:** `Microsoft.AspNetCore.Mvc`

**Key Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Request` | HttpRequest | Current HTTP request |
| `Response` | HttpResponse | Current HTTP response |
| `User` | ClaimsPrincipal | Current authenticated user |
| `ModelState` | ModelStateDictionary | Model validation state |
| `HttpContext` | HttpContext | Full HTTP context |
| `RouteData` | RouteData | Route values |

**Key Methods (Action Results):**

| Method | Status | Returns |
|--------|--------|---------|
| `Ok()` | 200 | OkResult |
| `Ok(value)` | 200 | OkObjectResult |
| `Created(uri, value)` | 201 | CreatedResult |
| `CreatedAtAction(...)` | 201 | CreatedAtActionResult |
| `CreatedAtRoute(...)` | 201 | CreatedAtRouteResult |
| `NoContent()` | 204 | NoContentResult |
| `BadRequest()` | 400 | BadRequestResult |
| `BadRequest(error)` | 400 | BadRequestObjectResult |
| `Unauthorized()` | 401 | UnauthorizedResult |
| `Forbid()` | 403 | ForbidResult |
| `NotFound()` | 404 | NotFoundResult |
| `NotFound(value)` | 404 | NotFoundObjectResult |
| `Conflict()` | 409 | ConflictResult |
| `Conflict(error)` | 409 | ConflictObjectResult |
| `UnprocessableEntity()` | 422 | UnprocessableEntityResult |
| `StatusCode(code)` | Custom | StatusCodeResult |

**Helper Methods:**

| Method | Purpose |
|--------|---------|
| `TryValidateModel(model)` | Manual validation |
| `Problem(...)` | RFC 7807 problem details |
| `File(...)` | Return file download |
| `Redirect(url)` | Redirect to URL |
| `RedirectToAction(...)` | Redirect to action |
| `LocalRedirect(url)` | Redirect (local only) |

**Complete Example:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    
    public ProductsController(IProductService service)
    {
        _service = service;
    }
    
    // Using properties
    [HttpGet("info")]
    public IActionResult GetInfo()
    {
        var info = new
        {
            User = User?.Identity?.Name,
            Path = Request.Path,
            Method = Request.Method,
            IsAuthenticated = User?.Identity?.IsAuthenticated
        };
        
        return Ok(info);
    }
    
    // Using helper methods
    [HttpPost]
    public async Task<IActionResult> Create(Product product)
    {
        // Manual validation
        if (!TryValidateModel(product))
            return BadRequest(ModelState);
        
        var created = await _service.CreateAsync(product);
        
        return CreatedAtAction(nameof(Get), new { id = created.Id }, created);
    }
    
    // Problem details
    [HttpGet("{id}")]
    public IActionResult Get(int id)
    {
        try
        {
            var product = _service.Get(id);
            return Ok(product);
        }
        catch (NotFoundException ex)
        {
            return Problem(
                title: "Product not found",
                detail: ex.Message,
                statusCode: 404);
        }
    }
}
```

---

### Controller Class ⭐⭐

**Purpose:** Base class for MVC controllers (with view support)

**Inherits:** ControllerBase

**Additional Methods:**

| Method | Purpose |
|--------|---------|
| `View()` | Return view result |
| `View(model)` | Return view with model |
| `View(viewName, model)` | Return named view |
| `PartialView(...)` | Return partial view |
| `Json(data)` | Return JSON result |
| `Content(text)` | Return text content |

**Example:**

```csharp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();  // Returns Index.cshtml
    }
    
    public IActionResult About()
    {
        var model = new AboutViewModel { Title = "About Us" };
        return View(model);
    }
    
    public IActionResult GetJson()
    {
        return Json(new { message = "Hello" });
    }
}
```

---

### IActionResult Interface ⭐⭐⭐

**Purpose:** Represents the result of an action method

**Implementations:**

| Class | Status | Usage |
|-------|--------|-------|
| `OkResult` | 200 | Success, no data |
| `OkObjectResult` | 200 | Success with data |
| `CreatedResult` | 201 | Resource created |
| `CreatedAtActionResult` | 201 | Created with route |
| `CreatedAtRouteResult` | 201 | Created with route name |
| `NoContentResult` | 204 | Success, no content |
| `BadRequestResult` | 400 | Bad request, no data |
| `BadRequestObjectResult` | 400 | Bad request with errors |
| `UnauthorizedResult` | 401 | Not authenticated |
| `ForbidResult` | 403 | Not authorized |
| `NotFoundResult` | 404 | Not found, no data |
| `NotFoundObjectResult` | 404 | Not found with data |
| `ConflictResult` | 409 | Conflict |
| `UnprocessableEntityResult` | 422 | Validation failed |
| `StatusCodeResult` | Custom | Any status code |
| `ObjectResult` | Custom | Any status + data |

---

### RouteData Class ⭐⭐

**Purpose:** Contains route values from the current request

**Key Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `Values` | RouteValueDictionary | Route parameter values |
| `DataTokens` | RouteValueDictionary | Additional route data |
| `Routers` | IList<IRouter> | Route handlers |

**Example:**

```csharp
[HttpGet("{id}")]
public IActionResult Get(int id)
{
    // Access route values
    var routeId = RouteData.Values["id"];
    var controller = RouteData.Values["controller"];
    var action = RouteData.Values["action"];
    
    return Ok(new { routeId, controller, action });
}
```

---

### ModelStateDictionary Class ⭐⭐

**Purpose:** Contains validation state and errors

**Key Methods:**

| Method | Purpose |
|--------|---------|
| `IsValid` | Check if model is valid |
| `AddModelError(key, message)` | Add error |
| `Clear()` | Clear all errors |
| `Remove(key)` | Remove specific error |

**Example:**

```csharp
[HttpPost]
public IActionResult Create(User user)
{
    // Custom validation
    if (UserExists(user.Email))
    {
        ModelState.AddModelError(nameof(user.Email), "Email already exists");
    }
    
    if (!ModelState.IsValid)
    {
        var errors = ModelState
            .Where(x => x.Value.Errors.Any())
            .Select(x => new
            {
                Field = x.Key,
                Errors = x.Value.Errors.Select(e => e.ErrorMessage)
            });
        
        return BadRequest(errors);
    }
    
    return Ok(user);
}
```

---

## 11. Configuration Deep-Dive

### Pattern 1: No Configuration (Defaults)

**Simple setup:**

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();  // Default configuration

var app = builder.Build();
app.MapControllers();
app.Run();
```

---

### Pattern 2: Code-Based Configuration

**Configure controllers in code:**

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers(options =>
{
    // Add filters
    options.Filters.Add<MyGlobalFilter>();
    
    // Suppress model state invalid filter
    options.SuppressModelStateInvalidFilter = false;
    
    // Return HTTP 406 for unsupported media types
    options.ReturnHttpNotAcceptable = true;
})
.ConfigureApiBehaviorOptions(options =>
{
    // Customize automatic 400 response
    options.InvalidModelStateResponseFactory = context =>
    {
        var errors = context.ModelState
            .Where(e => e.Value.Errors.Count > 0)
            .ToDictionary(
                e => e.Key,
                e => e.Value.Errors.Select(x => x.ErrorMessage).ToArray());
        
        return new BadRequestObjectResult(new
        {
            Message = "Validation failed",
            Errors = errors
        });
    };
})
.AddJsonOptions(options =>
{
    // JSON serialization
    options.JsonSerializerOptions.PropertyNamingPolicy = null; // Keep PascalCase
    options.JsonSerializerOptions.WriteIndented = true;
});

var app = builder.Build();
app.MapControllers();
app.Run();
```

---

### Pattern 3: Configuration from appsettings.json

**appsettings.json:**

```json
{
  "ControllerOptions": {
    "ReturnHttpNotAcceptable": true,
    "SuppressModelStateInvalidFilter": false
  },
  "JsonOptions": {
    "WriteIndented": true,
    "PropertyNamingPolicy": "CamelCase"
  }
}
```

**Program.cs:**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Bind from configuration
var controllerConfig = builder.Configuration
    .GetSection("ControllerOptions");

builder.Services.AddControllers(options =>
{
    options.ReturnHttpNotAcceptable = 
        controllerConfig.GetValue<bool>("ReturnHttpNotAcceptable");
});

var app = builder.Build();
app.MapControllers();
app.Run();
```

---

## 12. Advanced Routing Topics

### Custom Route Constraints

**Create custom constraint:**

```csharp
public class SlugConstraint : IRouteConstraint
{
    public bool Match(
        HttpContext httpContext,
        IRouter route,
        string routeKey,
        RouteValueDictionary values,
        RouteDirection routeDirection)
    {
        if (!values.TryGetValue(routeKey, out var value))
            return false;
        
        var slug = value?.ToString();
        
        // Slug: lowercase, letters, numbers, hyphens
        return !string.IsNullOrEmpty(slug) && 
               Regex.IsMatch(slug, @"^[a-z0-9-]+$");
    }
}

// Register
builder.Services.Configure<RouteOptions>(options =>
{
    options.ConstraintMap.Add("slug", typeof(SlugConstraint));
});

// Use
[HttpGet("posts/{slug:slug}")]
public IActionResult GetPost(string slug) => Ok(new { slug });

// Matches: /posts/my-first-post
// Rejects: /posts/My_Post!, /posts/123ABC
```

---

### Route Value Transformers ✨ ASP.NET Core 3.0+

**Transform route values:**

```csharp
public class SlugifyParameterTransformer : IOutboundParameterTransformer
{
    public string TransformOutbound(object value)
    {
        if (value == null) return null;
        
        // Convert "MyAction" to "my-action"
        return Regex.Replace(value.ToString(), "([a-z])([A-Z])", "$1-$2").ToLower();
    }
}

// Register
builder.Services.AddControllers(options =>
{
    options.Conventions.Add(new RouteTokenTransformerConvention(
        new SlugifyParameterTransformer()));
});

// Now URLs are automatically slugified:
// GetUserDetails() → /get-user-details
// CreateOrder() → /create-order
```

---

### Link Generation

**Generate URLs to actions:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(User user)
    {
        // Generate URL to Get action
        var url = Url.Action(nameof(Get), new { id = user.Id });
        // Result: /api/users/5
        
        return Created(url, user);
    }
    
    [HttpGet("{id}", Name = "GetUser")]
    public IActionResult Get(int id) => Ok(new { id });
    
    [HttpPost("link")]
    public IActionResult GetLink()
    {
        // Generate URL by route name
        var url = Url.RouteUrl("GetUser", new { id = 10 });
        // Result: /api/users/10
        
        // Generate absolute URL
        var absoluteUrl = Url.Action(nameof(Get), null, new { id = 5 }, Request.Scheme);
        // Result: https://example.com/api/users/5
        
        return Ok(new { url, absoluteUrl });
    }
}
```

---

### Route Debugging

**View matched endpoint:**

```csharp
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();
    
    if (endpoint != null)
    {
        Console.WriteLine($"Matched: {endpoint.DisplayName}");
        Console.WriteLine($"Route: {endpoint.Metadata.GetMetadata<RouteNameMetadata>()?.RouteName}");
    }
    
    await next();
});
```

---

## 13. Performance Tips

### 1. Use Route Constraints

**Improves matching performance:**

```csharp
// ✅ Good - constraint prevents string matching
[HttpGet("{id:int}")]
public IActionResult Get(int id) { }

// ❌ Slower - must try string conversion
[HttpGet("{id}")]
public IActionResult Get(int id) { }
```

---

### 2. Order Routes Efficiently

**Most specific routes first:**

```csharp
// ✅ Good order
[HttpGet("special")]         // Try this first
[HttpGet("{id:int}")]        // Then this
[HttpGet("{slug:alpha}")]    // Then this

// ❌ Bad order
[HttpGet("{slug}")]          // Matches everything!
[HttpGet("special")]         // Never reached
```

---

### 3. Use ActionResult<T>

**Reduces allocations:**

```csharp
// ✅ Better performance
[HttpGet("{id}")]
public ActionResult<User> Get(int id)
{
    var user = FindUser(id);
    if (user is null) return NotFound();
    return user;  // Direct return, no boxing
}

// ❌ Slower
[HttpGet("{id}")]
public IActionResult Get(int id)
{
    var user = FindUser(id);
    if (user is null) return NotFound();
    return Ok(user);  // Boxing overhead
}
```

---

### 4. Async All The Way

**Use async for I/O:**

```csharp
// ✅ Async
[HttpGet]
public async Task<ActionResult<List<User>>> GetUsers()
{
    var users = await _service.GetUsersAsync();
    return users;
}

// ❌ Blocking
[HttpGet]
public ActionResult<List<User>> GetUsers()
{
    var users = _service.GetUsers();  // Blocks thread
    return users;
}
```

---

## Summary: Complete Routing Checklist

**Project Setup:**
- [ ] Add `builder.Services.AddControllers()`
- [ ] Add `app.MapControllers()`
- [ ] Enable routing with `app.UseRouting()` (if needed)

**Controller Design:**
- [ ] Use `[ApiController]` attribute
- [ ] Inherit from `ControllerBase` (APIs) or `Controller` (MVC)
- [ ] Use `[Route("api/[controller]")]` for consistent URLs
- [ ] Use HTTP method attributes (`[HttpGet]`, `[HttpPost]`, etc.)

**Actions:**
- [ ] Return `ActionResult<T>` for type safety
- [ ] Use proper status codes (200, 201, 404, etc.)
- [ ] Add `[ProducesResponseType]` for documentation
- [ ] Use async/await for I/O operations

**Routing:**
- [ ] Use attribute routing for APIs
- [ ] Add route constraints for validation
- [ ] Order specific routes before generic routes
- [ ] Use lowercase, plural nouns for URLs

**Validation:**
- [ ] Use data annotations on models
- [ ] Let `[ApiController]` handle automatic validation
- [ ] Return proper error responses

---

**This completes the Routing & Controllers guide with practical examples and deep technical reference!**