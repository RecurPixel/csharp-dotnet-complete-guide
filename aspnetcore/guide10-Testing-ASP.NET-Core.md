# ASP.NET Core Testing - Complete Guide
## Practical Guide + Technical Reference

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. Testing Fundamentals
2. Unit Testing Setup (3 Methods)
3. Integration Testing Setup (3 Methods)
4. Testing Controllers & APIs
5. Testing Middleware
6. Testing with Databases
7. Common Testing Patterns
8. Mocking & Test Doubles
9. Troubleshooting Common Issues
10. Best Practices

### Part 2: Technical Reference (Deep Dive)
11. Important Testing Interfaces & Classes Reference
12. Configuration Deep-Dive
13. Advanced Testing Topics
14. Performance Testing

---

# PART 1: PRACTICAL GUIDE

---

## 1. Testing Fundamentals

**Simple Definition:** Writing code that automatically verifies your application works correctly.

**Think of it like:** A quality control inspector checking every part of a car before it leaves the factory.

### Testing Pyramid

```
        /\
       /  \       E2E Tests (Few, Slow, Expensive)
      /----\      - Test entire app
     /      \     - UI interactions
    /--------\    - Real browser
   /          \   
  /------------\  Integration Tests (Some, Medium Speed)
 /              \ - Test components together
/----------------\ - Real database, HTTP calls
|  Unit Tests    | - Many, Fast, Cheap
|  (Foundation)  | - Test single method/class
------------------  - No dependencies
```

**Key Principles:**
1. **Unit Tests** - Test one thing in isolation
2. **Integration Tests** - Test components working together
3. **End-to-End Tests** - Test entire workflows

---

### Types of Tests in ASP.NET Core

| Type | What It Tests | Speed | Cost | Quantity |
|------|---------------|-------|------|----------|
| **Unit** | Single class/method | ⚡ Fast (ms) | 💰 Cheap | 🔢 Many (70%) |
| **Integration** | API endpoints + DB | 🐢 Medium (100-500ms) | 💰💰 Medium | 🔢 Some (20%) |
| **E2E** | Full user flow | 🐌 Slow (seconds) | 💰💰💰 Expensive | 🔢 Few (10%) |

---

## 2. Unit Testing Setup (3 Methods)

### Method 1: xUnit Only (Simple & Quick)

**When to use:**
- ✅ Simple testing needs
- ✅ Learning
- ✅ Small projects
- ❌ Need advanced mocking
- ❌ Complex assertions

**Step 1: Create Test Project**

```bash
dotnet new xunit -n MyApp.Tests
cd MyApp.Tests
dotnet add reference ../MyApp/MyApp.csproj
```

**Step 2: Write Your First Test**

```csharp
using Xunit;

namespace MyApp.Tests;

public class CalculatorTests
{
    [Fact]
    public void Add_TwoNumbers_ReturnsSum()
    {
        // Arrange
        var calculator = new Calculator();
        
        // Act
        var result = calculator.Add(2, 3);
        
        // Assert
        Assert.Equal(5, result);
    }
    
    [Theory]
    [InlineData(2, 3, 5)]
    [InlineData(0, 0, 0)]
    [InlineData(-2, 3, 1)]
    public void Add_VariousInputs_ReturnsCorrectSum(int a, int b, int expected)
    {
        var calculator = new Calculator();
        var result = calculator.Add(a, b);
        Assert.Equal(expected, result);
    }
}
```

**Step 3: Run Tests**

```bash
dotnet test
```

---

### Method 2: xUnit + Moq + FluentAssertions (Production) ⭐ RECOMMENDED

**When to use:**
- ✅ Production code
- ✅ Need to mock dependencies
- ✅ Readable assertions
- ✅ Testing services with DI

**Step 1: Create Test Project with Packages**

```bash
dotnet new xunit -n MyApp.Tests
cd MyApp.Tests
dotnet add reference ../MyApp/MyApp.csproj
dotnet add package Moq
dotnet add package FluentAssertions
```

**Step 2: Test Service with Dependencies**

```csharp
using Moq;
using FluentAssertions;
using Xunit;

namespace MyApp.Tests;

public class UserServiceTests
{
    [Fact]
    public async Task GetUserById_ExistingUser_ReturnsUser()
    {
        // Arrange
        var mockRepo = new Mock<IUserRepository>();
        var expectedUser = new User { Id = 1, Name = "John" };
        
        mockRepo.Setup(r => r.GetByIdAsync(1))
                .ReturnsAsync(expectedUser);
        
        var service = new UserService(mockRepo.Object);
        
        // Act
        var result = await service.GetUserByIdAsync(1);
        
        // Assert
        result.Should().NotBeNull();
        result.Id.Should().Be(1);
        result.Name.Should().Be("John");
        mockRepo.Verify(r => r.GetByIdAsync(1), Times.Once);
    }
    
    [Fact]
    public async Task CreateUser_ValidUser_CallsRepository()
    {
        // Arrange
        var mockRepo = new Mock<IUserRepository>();
        var newUser = new User { Name = "Jane" };
        
        mockRepo.Setup(r => r.AddAsync(It.IsAny<User>()))
                .ReturnsAsync(newUser);
        
        var service = new UserService(mockRepo.Object);
        
        // Act
        var result = await service.CreateUserAsync(newUser);
        
        // Assert
        result.Should().NotBeNull();
        mockRepo.Verify(r => r.AddAsync(It.Is<User>(u => u.Name == "Jane")), Times.Once);
    }
}
```

---

### Method 3: xUnit + NSubstitute + Shouldly (Alternative)

**When to use:**
- ✅ Prefer fluent mocking syntax
- ✅ Alternative to Moq
- ✅ Different assertion style

**Step 1: Install Packages**

```bash
dotnet add package NSubstitute
dotnet add package Shouldly
```

**Step 2: Write Tests**

```csharp
using NSubstitute;
using Shouldly;
using Xunit;

namespace MyApp.Tests;

public class UserServiceTests
{
    [Fact]
    public async Task GetUserById_ExistingUser_ReturnsUser()
    {
        // Arrange
        var mockRepo = Substitute.For<IUserRepository>();
        var expectedUser = new User { Id = 1, Name = "John" };
        
        mockRepo.GetByIdAsync(1).Returns(expectedUser);
        
        var service = new UserService(mockRepo);
        
        // Act
        var result = await service.GetUserByIdAsync(1);
        
        // Assert
        result.ShouldNotBeNull();
        result.Id.ShouldBe(1);
        result.Name.ShouldBe("John");
        await mockRepo.Received(1).GetByIdAsync(1);
    }
}
```

---

## 3. Integration Testing Setup (3 Methods)

### Method 1: WebApplicationFactory (Standard)

**When to use:**
- ✅ Testing API endpoints
- ✅ In-memory database
- ✅ Full HTTP request/response
- ✅ Production-like testing

**Step 1: Install Package**

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing
```

**Step 2: Create Test Class**

```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using System.Net.Http.Json;
using Xunit;

namespace MyApp.Tests.Integration;

public class UsersControllerTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    
    public UsersControllerTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task GetUsers_ReturnsSuccessStatusCode()
    {
        // Act
        var response = await _client.GetAsync("/api/users");
        
        // Assert
        response.EnsureSuccessStatusCode();
    }
    
    [Fact]
    public async Task GetUsers_ReturnsListOfUsers()
    {
        // Act
        var users = await _client.GetFromJsonAsync<List<User>>("/api/users");
        
        // Assert
        users.Should().NotBeNull();
        users.Should().HaveCountGreaterThan(0);
    }
}
```

**Step 3: Make Program.cs Accessible**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// ... your code ...

app.Run();

// Add this at the end for testing
public partial class Program { }
```

---

### Method 2: Custom WebApplicationFactory (Production) ⭐ RECOMMENDED

**When to use:**
- ✅ Need custom configuration
- ✅ Test database instead of real one
- ✅ Replace services for testing
- ✅ Integration testing in CI/CD

**Step 1: Create Custom Factory**

```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;

namespace MyApp.Tests.Integration;

public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Remove real database
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));
            
            if (descriptor != null)
                services.Remove(descriptor);
            
            // Add in-memory database
            services.AddDbContext<ApplicationDbContext>(options =>
            {
                options.UseInMemoryDatabase("TestDb");
            });
            
            // Build service provider
            var sp = services.BuildServiceProvider();
            
            // Seed test data
            using var scope = sp.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
            db.Database.EnsureCreated();
            SeedTestData(db);
        });
    }
    
    private void SeedTestData(ApplicationDbContext db)
    {
        db.Users.AddRange(
            new User { Id = 1, Name = "Test User 1", Email = "test1@example.com" },
            new User { Id = 2, Name = "Test User 2", Email = "test2@example.com" }
        );
        db.SaveChanges();
    }
}
```

**Step 2: Use Custom Factory**

```csharp
public class UsersControllerTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;
    private readonly CustomWebApplicationFactory _factory;
    
    public UsersControllerTests(CustomWebApplicationFactory factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task GetUser_ExistingId_ReturnsUser()
    {
        // Act
        var user = await _client.GetFromJsonAsync<User>("/api/users/1");
        
        // Assert
        user.Should().NotBeNull();
        user.Id.Should().Be(1);
        user.Name.Should().Be("Test User 1");
    }
    
    [Fact]
    public async Task CreateUser_ValidData_ReturnsCreatedUser()
    {
        // Arrange
        var newUser = new { Name = "New User", Email = "new@example.com" };
        
        // Act
        var response = await _client.PostAsJsonAsync("/api/users", newUser);
        var createdUser = await response.Content.ReadFromJsonAsync<User>();
        
        // Assert
        response.StatusCode.Should().Be(System.Net.HttpStatusCode.Created);
        createdUser.Name.Should().Be("New User");
    }
}
```

---

### Method 3: TestServer (Lightweight)

**When to use:**
- ✅ Unit testing middleware
- ✅ Don't need full HTTP client
- ✅ Faster than WebApplicationFactory

**Step 1: Create TestServer**

```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.TestHost;
using Microsoft.Extensions.Hosting;

namespace MyApp.Tests.Integration;

public class MiddlewareTests
{
    [Fact]
    public async Task CustomMiddleware_AddsCustomHeader()
    {
        // Arrange
        using var host = await new HostBuilder()
            .ConfigureWebHost(webBuilder =>
            {
                webBuilder
                    .UseTestServer()
                    .Configure(app =>
                    {
                        app.UseCustomMiddleware();
                        app.Run(async context =>
                        {
                            await context.Response.WriteAsync("Hello");
                        });
                    });
            })
            .StartAsync();
        
        var client = host.GetTestClient();
        
        // Act
        var response = await client.GetAsync("/");
        
        // Assert
        response.Headers.Should().Contain(h => h.Key == "X-Custom-Header");
    }
}
```

---

## 4. Testing Controllers & APIs

### Testing GET Endpoints

```csharp
public class ProductsControllerTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;
    
    public ProductsControllerTests(CustomWebApplicationFactory factory)
    {
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task GetProducts_ReturnsAllProducts()
    {
        // Act
        var products = await _client.GetFromJsonAsync<List<Product>>("/api/products");
        
        // Assert
        products.Should().NotBeNull();
        products.Should().HaveCount(2); // Assuming 2 seeded products
    }
    
    [Fact]
    public async Task GetProduct_ExistingId_ReturnsProduct()
    {
        // Act
        var response = await _client.GetAsync("/api/products/1");
        var product = await response.Content.ReadFromJsonAsync<Product>();
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        product.Id.Should().Be(1);
    }
    
    [Fact]
    public async Task GetProduct_NonExistingId_ReturnsNotFound()
    {
        // Act
        var response = await _client.GetAsync("/api/products/999");
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.NotFound);
    }
}
```

---

### Testing POST Endpoints

```csharp
[Fact]
public async Task CreateProduct_ValidData_ReturnsCreated()
{
    // Arrange
    var newProduct = new ProductCreateDto
    {
        Name = "New Product",
        Price = 99.99m,
        Description = "Test product"
    };
    
    // Act
    var response = await _client.PostAsJsonAsync("/api/products", newProduct);
    var createdProduct = await response.Content.ReadFromJsonAsync<Product>();
    
    // Assert
    response.StatusCode.Should().Be(HttpStatusCode.Created);
    response.Headers.Location.Should().NotBeNull();
    createdProduct.Name.Should().Be("New Product");
    createdProduct.Id.Should().BeGreaterThan(0);
}

[Fact]
public async Task CreateProduct_InvalidData_ReturnsBadRequest()
{
    // Arrange
    var invalidProduct = new ProductCreateDto
    {
        Name = "", // Invalid: empty name
        Price = -10 // Invalid: negative price
    };
    
    // Act
    var response = await _client.PostAsJsonAsync("/api/products", invalidProduct);
    
    // Assert
    response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
}
```

---

### Testing PUT Endpoints

```csharp
[Fact]
public async Task UpdateProduct_ValidData_ReturnsNoContent()
{
    // Arrange
    var updateDto = new ProductUpdateDto
    {
        Name = "Updated Product",
        Price = 149.99m
    };
    
    // Act
    var response = await _client.PutAsJsonAsync("/api/products/1", updateDto);
    
    // Assert
    response.StatusCode.Should().Be(HttpStatusCode.NoContent);
    
    // Verify update
    var updatedProduct = await _client.GetFromJsonAsync<Product>("/api/products/1");
    updatedProduct.Name.Should().Be("Updated Product");
}
```

---

### Testing DELETE Endpoints

```csharp
[Fact]
public async Task DeleteProduct_ExistingId_ReturnsNoContent()
{
    // Act
    var response = await _client.DeleteAsync("/api/products/1");
    
    // Assert
    response.StatusCode.Should().Be(HttpStatusCode.NoContent);
    
    // Verify deletion
    var getResponse = await _client.GetAsync("/api/products/1");
    getResponse.StatusCode.Should().Be(HttpStatusCode.NotFound);
}
```

---

### Testing Controller Actions Directly (Unit Testing)

```csharp
using Microsoft.AspNetCore.Mvc;

public class ProductsControllerUnitTests
{
    [Fact]
    public async Task GetProducts_ReturnsOkResult()
    {
        // Arrange
        var mockRepo = new Mock<IProductRepository>();
        mockRepo.Setup(r => r.GetAllAsync())
                .ReturnsAsync(new List<Product>
                {
                    new Product { Id = 1, Name = "Product 1" }
                });
        
        var controller = new ProductsController(mockRepo.Object);
        
        // Act
        var result = await controller.GetProducts();
        
        // Assert
        var okResult = result.Result.Should().BeOfType<OkObjectResult>().Subject;
        var products = okResult.Value.Should().BeAssignableTo<List<Product>>().Subject;
        products.Should().HaveCount(1);
    }
    
    [Fact]
    public async Task GetProduct_NonExistingId_ReturnsNotFound()
    {
        // Arrange
        var mockRepo = new Mock<IProductRepository>();
        mockRepo.Setup(r => r.GetByIdAsync(999))
                .ReturnsAsync((Product)null);
        
        var controller = new ProductsController(mockRepo.Object);
        
        // Act
        var result = await controller.GetProduct(999);
        
        // Assert
        result.Result.Should().BeOfType<NotFoundResult>();
    }
}
```

---

## 5. Testing Middleware

### Unit Testing Middleware

```csharp
public class RequestLoggingMiddlewareTests
{
    [Fact]
    public async Task InvokeAsync_LogsRequestAndResponse()
    {
        // Arrange
        var mockLogger = new Mock<ILogger<RequestLoggingMiddleware>>();
        var context = new DefaultHttpContext();
        context.Request.Method = "GET";
        context.Request.Path = "/test";
        
        var requestDelegateCalled = false;
        RequestDelegate next = (HttpContext ctx) =>
        {
            requestDelegateCalled = true;
            return Task.CompletedTask;
        };
        
        var middleware = new RequestLoggingMiddleware(next, mockLogger.Object);
        
        // Act
        await middleware.InvokeAsync(context);
        
        // Assert
        requestDelegateCalled.Should().BeTrue();
        mockLogger.Verify(
            x => x.Log(
                LogLevel.Information,
                It.IsAny<EventId>(),
                It.Is<It.IsAnyType>((o, t) => o.ToString().Contains("GET")),
                It.IsAny<Exception>(),
                It.IsAny<Func<It.IsAnyType, Exception, string>>()),
            Times.AtLeastOnce);
    }
}
```

---

### Integration Testing Middleware

```csharp
public class MiddlewareIntegrationTests
{
    [Fact]
    public async Task ApiKeyMiddleware_ValidKey_AllowsRequest()
    {
        // Arrange
        using var factory = new WebApplicationFactory<Program>()
            .WithWebHostBuilder(builder =>
            {
                builder.ConfigureServices(services =>
                {
                    // Configure test services
                });
            });
        
        var client = factory.CreateClient();
        client.DefaultRequestHeaders.Add("X-API-Key", "test-api-key");
        
        // Act
        var response = await client.GetAsync("/api/protected");
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
    }
    
    [Fact]
    public async Task ApiKeyMiddleware_MissingKey_ReturnsUnauthorized()
    {
        // Arrange
        using var factory = new WebApplicationFactory<Program>();
        var client = factory.CreateClient();
        
        // Act
        var response = await client.GetAsync("/api/protected");
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Unauthorized);
    }
}
```

---

## 6. Testing with Databases

### Method 1: In-Memory Database (Fast, Simple)

**When to use:**
- ✅ Fast tests
- ✅ Don't need specific DB features
- ❌ Testing SQL queries
- ❌ Testing transactions

```csharp
public class UserRepositoryTests
{
    private ApplicationDbContext CreateContext()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;
        
        var context = new ApplicationDbContext(options);
        
        // Seed data
        context.Users.AddRange(
            new User { Id = 1, Name = "User 1" },
            new User { Id = 2, Name = "User 2" }
        );
        context.SaveChanges();
        
        return context;
    }
    
    [Fact]
    public async Task GetByIdAsync_ExistingUser_ReturnsUser()
    {
        // Arrange
        using var context = CreateContext();
        var repository = new UserRepository(context);
        
        // Act
        var user = await repository.GetByIdAsync(1);
        
        // Assert
        user.Should().NotBeNull();
        user.Name.Should().Be("User 1");
    }
    
    [Fact]
    public async Task AddAsync_NewUser_AddsToDatabase()
    {
        // Arrange
        using var context = CreateContext();
        var repository = new UserRepository(context);
        var newUser = new User { Name = "New User" };
        
        // Act
        var result = await repository.AddAsync(newUser);
        await context.SaveChangesAsync();
        
        // Assert
        var users = await context.Users.ToListAsync();
        users.Should().HaveCount(3);
    }
}
```

---

### Method 2: SQLite In-Memory (Better for Real SQL)

**When to use:**
- ✅ Need real SQL behavior
- ✅ Test migrations
- ✅ Fast tests
- ✅ Production-like queries

```csharp
using Microsoft.Data.Sqlite;

public class UserRepositoryTests : IDisposable
{
    private readonly SqliteConnection _connection;
    private readonly ApplicationDbContext _context;
    
    public UserRepositoryTests()
    {
        // Create in-memory SQLite database
        _connection = new SqliteConnection("DataSource=:memory:");
        _connection.Open();
        
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlite(_connection)
            .Options;
        
        _context = new ApplicationDbContext(options);
        _context.Database.EnsureCreated();
        
        // Seed data
        SeedData();
    }
    
    private void SeedData()
    {
        _context.Users.AddRange(
            new User { Id = 1, Name = "User 1", Email = "user1@test.com" },
            new User { Id = 2, Name = "User 2", Email = "user2@test.com" }
        );
        _context.SaveChanges();
    }
    
    [Fact]
    public async Task FindByEmailAsync_ExistingEmail_ReturnsUser()
    {
        // Arrange
        var repository = new UserRepository(_context);
        
        // Act
        var user = await repository.FindByEmailAsync("user1@test.com");
        
        // Assert
        user.Should().NotBeNull();
        user.Name.Should().Be("User 1");
    }
    
    public void Dispose()
    {
        _context.Dispose();
        _connection.Dispose();
    }
}
```

---

### Method 3: Test Containers (Real Database) ⭐ Most Realistic

**When to use:**
- ✅ CI/CD pipelines
- ✅ Need exact production DB
- ✅ Test complex queries
- ⚠️ Slower than in-memory

```bash
dotnet add package Testcontainers.PostgreSQL
# or
dotnet add package Testcontainers.MsSql
```

```csharp
using Testcontainers.PostgreSql;

public class UserRepositoryIntegrationTests : IAsyncLifetime
{
    private PostgreSqlContainer _container;
    private ApplicationDbContext _context;
    
    public async Task InitializeAsync()
    {
        _container = new PostgreSqlBuilder()
            .WithImage("postgres:15")
            .WithDatabase("testdb")
            .WithUsername("testuser")
            .WithPassword("testpass")
            .Build();
        
        await _container.StartAsync();
        
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseNpgsql(_container.GetConnectionString())
            .Options;
        
        _context = new ApplicationDbContext(options);
        await _context.Database.MigrateAsync();
        
        SeedData();
    }
    
    private void SeedData()
    {
        _context.Users.AddRange(
            new User { Id = 1, Name = "User 1" },
            new User { Id = 2, Name = "User 2" }
        );
        _context.SaveChanges();
    }
    
    [Fact]
    public async Task ComplexQuery_ReturnsCorrectResults()
    {
        // Arrange
        var repository = new UserRepository(_context);
        
        // Act - Test actual PostgreSQL query
        var users = await repository.GetActiveUsersWithOrders();
        
        // Assert
        users.Should().NotBeEmpty();
    }
    
    public async Task DisposeAsync()
    {
        await _context.DisposeAsync();
        await _container.DisposeAsync();
    }
}
```

---

## 7. Common Testing Patterns

### AAA Pattern (Arrange-Act-Assert)

```csharp
[Fact]
public async Task MethodName_Scenario_ExpectedBehavior()
{
    // Arrange - Set up test data and dependencies
    var mockRepo = new Mock<IUserRepository>();
    var expectedUser = new User { Id = 1, Name = "Test" };
    mockRepo.Setup(r => r.GetByIdAsync(1)).ReturnsAsync(expectedUser);
    var service = new UserService(mockRepo.Object);
    
    // Act - Execute the method being tested
    var result = await service.GetUserByIdAsync(1);
    
    // Assert - Verify the results
    result.Should().NotBeNull();
    result.Id.Should().Be(1);
}
```

---

### Test Fixtures (Shared Setup)

```csharp
public class DatabaseFixture : IDisposable
{
    public ApplicationDbContext Context { get; private set; }
    
    public DatabaseFixture()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase("TestDb")
            .Options;
        
        Context = new ApplicationDbContext(options);
        
        // Seed common test data
        Context.Users.AddRange(
            new User { Id = 1, Name = "User 1" },
            new User { Id = 2, Name = "User 2" }
        );
        Context.SaveChanges();
    }
    
    public void Dispose()
    {
        Context.Dispose();
    }
}

// Use in tests
public class UserRepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly ApplicationDbContext _context;
    
    public UserRepositoryTests(DatabaseFixture fixture)
    {
        _context = fixture.Context;
    }
    
    [Fact]
    public async Task GetAllAsync_ReturnsAllUsers()
    {
        // Arrange
        var repository = new UserRepository(_context);
        
        // Act
        var users = await repository.GetAllAsync();
        
        // Assert
        users.Should().HaveCount(2);
    }
}
```

---

### Testing Async Methods

```csharp
[Fact]
public async Task AsyncMethod_ReturnsExpectedResult()
{
    // Arrange
    var mockRepo = new Mock<IUserRepository>();
    mockRepo.Setup(r => r.GetByIdAsync(It.IsAny<int>()))
            .ReturnsAsync(new User { Id = 1 });
    
    var service = new UserService(mockRepo.Object);
    
    // Act
    var result = await service.GetUserByIdAsync(1);
    
    // Assert
    result.Should().NotBeNull();
}

[Fact]
public async Task AsyncMethod_ThrowsException()
{
    // Arrange
    var mockRepo = new Mock<IUserRepository>();
    mockRepo.Setup(r => r.GetByIdAsync(999))
            .ThrowsAsync(new NotFoundException("User not found"));
    
    var service = new UserService(mockRepo.Object);
    
    // Act & Assert
    await service.Invoking(s => s.GetUserByIdAsync(999))
                 .Should().ThrowAsync<NotFoundException>()
                 .WithMessage("User not found");
}
```

---

### Testing Exceptions

```csharp
[Fact]
public void Method_InvalidInput_ThrowsException()
{
    // Arrange
    var service = new UserService();
    
    // Act & Assert
    service.Invoking(s => s.CreateUser(null))
           .Should().Throw<ArgumentNullException>()
           .WithParameterName("user");
}

[Fact]
public async Task Method_NotFound_ThrowsNotFoundException()
{
    // Arrange
    var mockRepo = new Mock<IUserRepository>();
    mockRepo.Setup(r => r.GetByIdAsync(999))
            .ReturnsAsync((User)null);
    
    var service = new UserService(mockRepo.Object);
    
    // Act & Assert
    await service.Invoking(s => s.GetUserByIdAsync(999))
                 .Should().ThrowAsync<NotFoundException>();
}
```

---

## 8. Mocking & Test Doubles

### Mock Setup Patterns

```csharp
// Return a value
mockRepo.Setup(r => r.GetByIdAsync(1))
        .ReturnsAsync(new User { Id = 1 });

// Return different values based on input
mockRepo.Setup(r => r.GetByIdAsync(It.IsAny<int>()))
        .ReturnsAsync((int id) => new User { Id = id });

// Throw exception
mockRepo.Setup(r => r.DeleteAsync(999))
        .ThrowsAsync(new NotFoundException());

// Callback for custom logic
mockRepo.Setup(r => r.AddAsync(It.IsAny<User>()))
        .Callback<User>(u => Console.WriteLine($"Adding {u.Name}"))
        .ReturnsAsync((User u) => u);

// Verify method was called
mockRepo.Verify(r => r.GetByIdAsync(1), Times.Once);
mockRepo.Verify(r => r.AddAsync(It.IsAny<User>()), Times.Never);

// Verify with specific arguments
mockRepo.Verify(r => r.UpdateAsync(
    It.Is<User>(u => u.Id == 1 && u.Name == "Updated")), 
    Times.Once);
```

---

### Argument Matchers

```csharp
// Any value
mockRepo.Setup(r => r.GetByIdAsync(It.IsAny<int>()))
        .ReturnsAsync(new User());

// Specific condition
mockRepo.Setup(r => r.GetByIdAsync(It.Is<int>(id => id > 0)))
        .ReturnsAsync(new User());

// Range
mockRepo.Setup(r => r.GetByIdAsync(It.IsInRange(1, 100, Range.Inclusive)))
        .ReturnsAsync(new User());

// Regex
mockRepo.Setup(r => r.FindByEmailAsync(It.IsRegex(@".*@example\.com")))
        .ReturnsAsync(new User());
```

---

### Mock vs Stub vs Fake

```csharp
// Mock - Verify behavior (interactions)
var mockRepo = new Mock<IUserRepository>();
mockRepo.Setup(r => r.AddAsync(It.IsAny<User>()));
// ... call code
mockRepo.Verify(r => r.AddAsync(It.IsAny<User>()), Times.Once);

// Stub - Provide data
var stubRepo = new Mock<IUserRepository>();
stubRepo.Setup(r => r.GetAllAsync())
        .ReturnsAsync(new List<User> { new User { Id = 1 } });
var users = await stubRepo.Object.GetAllAsync(); // Just return data

// Fake - Simple working implementation
public class FakeUserRepository : IUserRepository
{
    private readonly List<User> _users = new();
    
    public Task<User> GetByIdAsync(int id)
    {
        return Task.FromResult(_users.FirstOrDefault(u => u.Id == id));
    }
    
    public Task AddAsync(User user)
    {
        _users.Add(user);
        return Task.CompletedTask;
    }
}
```

---

## 9. Troubleshooting Common Issues

### Issue 1: Tests Interfering with Each Other

**Problem:** Tests pass individually but fail when run together

**Solution 1: Use unique database names**
```csharp
private ApplicationDbContext CreateContext()
{
    var options = new DbContextOptionsBuilder<ApplicationDbContext>()
        .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString()) // ✅ Unique per test
        .Options;
    return new ApplicationDbContext(options);
}
```

**Solution 2: Cleanup between tests**
```csharp
public class UserRepositoryTests : IDisposable
{
    private readonly ApplicationDbContext _context;
    
    public UserRepositoryTests()
    {
        _context = CreateContext();
    }
    
    public void Dispose()
    {
        _context.Database.EnsureDeleted(); // Clean up
        _context.Dispose();
    }
}
```

---

### Issue 2: Async Tests Hanging

**Problem:** Test never completes

```csharp
// ❌ WRONG - Deadlock
[Fact]
public void BadAsyncTest()
{
    var result = service.GetUserAsync(1).Result; // Blocks!
}

// ✅ CORRECT
[Fact]
public async Task GoodAsyncTest()
{
    var result = await service.GetUserAsync(1);
}
```

---

### Issue 3: Mock Not Being Called

**Problem:** `mockRepo.Verify()` fails

```csharp
// Make sure setup matches actual call
mockRepo.Setup(r => r.GetByIdAsync(1)) // Setup for ID 1
        .ReturnsAsync(new User());

// But code calls with different ID
var result = await service.GetUserByIdAsync(2); // ❌ Doesn't match!

mockRepo.Verify(r => r.GetByIdAsync(1), Times.Once); // ❌ Fails!

// Solution: Use It.IsAny<>()
mockRepo.Setup(r => r.GetByIdAsync(It.IsAny<int>())) // ✅ Matches any ID
        .ReturnsAsync(new User());
```

---

### Issue 4: Integration Tests Not Finding Program

**Problem:** `WebApplicationFactory<Program>` fails

**Solution: Make Program.cs accessible**
```csharp
// Add to end of Program.cs
public partial class Program { }
```

---

### Issue 5: Database Context Disposed

**Problem:** "Cannot access a disposed object"

```csharp
// ❌ WRONG
[Fact]
public async Task BadTest()
{
    ApplicationDbContext context;
    using (context = CreateContext())
    {
        var repo = new UserRepository(context);
    } // Context disposed here
    
    var user = await repo.GetByIdAsync(1); // ❌ Context is disposed!
}

// ✅ CORRECT
[Fact]
public async Task GoodTest()
{
    using var context = CreateContext();
    var repo = new UserRepository(context);
    var user = await repo.GetByIdAsync(1); // ✅ Context still alive
}
```

---

## 10. Best Practices

### Naming Conventions

```csharp
// Pattern: MethodName_Scenario_ExpectedBehavior
[Fact]
public void Add_TwoPositiveNumbers_ReturnsSum() { }

[Fact]
public void GetUser_NonExistingId_ThrowsNotFoundException() { }

[Fact]
public async Task CreateUser_ValidData_ReturnsCreatedUser() { }

[Fact]
public void Divide_ByZero_ThrowsDivideByZeroException() { }
```

---

### Test Organization

```
MyApp.Tests/
├── Unit/
│   ├── Services/
│   │   ├── UserServiceTests.cs
│   │   └── ProductServiceTests.cs
│   ├── Repositories/
│   │   └── UserRepositoryTests.cs
│   └── Validators/
│       └── UserValidatorTests.cs
├── Integration/
│   ├── Controllers/
│   │   ├── UsersControllerTests.cs
│   │   └── ProductsControllerTests.cs
│   ├── Middleware/
│   │   └── ApiKeyMiddlewareTests.cs
│   └── CustomWebApplicationFactory.cs
└── Helpers/
    ├── TestData.cs
    └── MockHelpers.cs
```

---

### Test Data Builders

```csharp
public class UserBuilder
{
    private int _id = 1;
    private string _name = "Test User";
    private string _email = "test@example.com";
    private bool _isActive = true;
    
    public UserBuilder WithId(int id)
    {
        _id = id;
        return this;
    }
    
    public UserBuilder WithName(string name)
    {
        _name = name;
        return this;
    }
    
    public UserBuilder WithEmail(string email)
    {
        _email = email;
        return this;
    }
    
    public UserBuilder Inactive()
    {
        _isActive = false;
        return this;
    }
    
    public User Build()
    {
        return new User
        {
            Id = _id,
            Name = _name,
            Email = _email,
            IsActive = _isActive
        };
    }
}

// Usage
[Fact]
public async Task Test_WithCustomUser()
{
    var user = new UserBuilder()
        .WithId(5)
        .WithName("Custom User")
        .Inactive()
        .Build();
    
    // ... use user in test
}
```

---

### DRY Principle (Don't Repeat Yourself)

```csharp
public class UserServiceTestsBase
{
    protected Mock<IUserRepository> CreateMockRepository()
    {
        var mock = new Mock<IUserRepository>();
        mock.Setup(r => r.GetAllAsync())
            .ReturnsAsync(GetTestUsers());
        return mock;
    }
    
    protected List<User> GetTestUsers()
    {
        return new List<User>
        {
            new User { Id = 1, Name = "User 1" },
            new User { Id = 2, Name = "User 2" }
        };
    }
}

public class UserServiceTests : UserServiceTestsBase
{
    [Fact]
    public async Task Test1()
    {
        var mockRepo = CreateMockRepository(); // Reuse!
        // ...
    }
}
```

---

### Test Categories (Tags)

```csharp
public class UserServiceTests
{
    [Fact]
    [Trait("Category", "Unit")]
    public void UnitTest1() { }
    
    [Fact]
    [Trait("Category", "Integration")]
    public async Task IntegrationTest1() { }
    
    [Fact]
    [Trait("Category", "Slow")]
    public async Task SlowTest1() { }
}

// Run specific category
// dotnet test --filter Category=Unit
// dotnet test --filter "Category!=Slow"
```

---

### Code Coverage

```bash
# Install coverage tool
dotnet tool install --global dotnet-coverage

# Run tests with coverage
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover

# Generate report
dotnet tool install --global dotnet-reportgenerator-globaltool
reportgenerator -reports:coverage.opencover.xml -targetdir:coverage
```

---

### Best Practices Checklist

- [ ] **Follow AAA pattern** (Arrange, Act, Assert)
- [ ] **One assertion per test** (or closely related assertions)
- [ ] **Clear test names** (MethodName_Scenario_ExpectedBehavior)
- [ ] **Independent tests** (no shared state)
- [ ] **Fast tests** (unit tests < 100ms)
- [ ] **Deterministic** (same input = same output)
- [ ] **Test edge cases** (null, empty, negative, max values)
- [ ] **Mock external dependencies** (database, HTTP, file system)
- [ ] **Use test builders** for complex objects
- [ ] **Organize tests** by feature/layer
- [ ] **Clean up resources** (Dispose, IAsyncLifetime)
- [ ] **Verify behavior** not implementation
- [ ] **Write tests first** (TDD when possible)
- [ ] **Keep tests simple** (easier to read than production code)
- [ ] **Aim for 70-80% coverage** (not 100%)

---

# PART 2: TECHNICAL REFERENCE

---

## 11. Important Testing Interfaces & Classes Reference

### IClassFixture<TFixture>

**Namespace:** `Xunit`

**Purpose:** Share setup/cleanup code across tests in a class

**Declaration:**
```csharp
public interface IClassFixture<TFixture> where TFixture : class
```

**Usage:**
```csharp
public class DatabaseFixture : IDisposable
{
    public ApplicationDbContext Context { get; }
    
    public DatabaseFixture()
    {
        // Setup - runs once per test class
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase("TestDb")
            .Options;
        Context = new ApplicationDbContext(options);
    }
    
    public void Dispose()
    {
        // Cleanup - runs once after all tests
        Context.Dispose();
    }
}

public class MyTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;
    
    public MyTests(DatabaseFixture fixture)
    {
        _fixture = fixture; // Injected by xUnit
    }
}
```

---

### IAsyncLifetime

**Namespace:** `Xunit`

**Purpose:** Async initialization and cleanup

**Declaration:**
```csharp
public interface IAsyncLifetime
{
    Task InitializeAsync();
    Task DisposeAsync();
}
```

**Members:**

| Member | Return Type | Description |
|--------|-------------|-------------|
| `InitializeAsync()` | `Task` | Called before each test |
| `DisposeAsync()` | `Task` | Called after each test |

**Usage:**
```csharp
public class DatabaseTests : IAsyncLifetime
{
    private ApplicationDbContext _context;
    
    public async Task InitializeAsync()
    {
        // Setup before each test
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(Guid.NewGuid().ToString())
            .Options;
        
        _context = new ApplicationDbContext(options);
        await _context.Database.EnsureCreatedAsync();
        await SeedDataAsync();
    }
    
    public async Task DisposeAsync()
    {
        // Cleanup after each test
        await _context.Database.EnsureDeletedAsync();
        await _context.DisposeAsync();
    }
    
    private async Task SeedDataAsync()
    {
        _context.Users.Add(new User { Id = 1, Name = "Test" });
        await _context.SaveChangesAsync();
    }
}
```

---

### WebApplicationFactory<TEntryPoint>

**Namespace:** `Microsoft.AspNetCore.Mvc.Testing`

**Purpose:** Create a test server for integration testing

**Declaration:**
```csharp
public class WebApplicationFactory<TEntryPoint> : IDisposable 
    where TEntryPoint : class
```

**Important Members:**

| Member | Type | Description |
|--------|------|-------------|
| `CreateClient()` | `HttpClient` | Create HTTP client for testing |
| `CreateDefaultClient()` | `HttpClient` | Create client with default settings |
| `Server` | `TestServer` | Access underlying test server |
| `Services` | `IServiceProvider` | Access DI container |
| `WithWebHostBuilder()` | `WebApplicationFactory<T>` | Customize configuration |

**Usage:**
```csharp
public class ApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    
    public ApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient(new WebApplicationFactoryClientOptions
        {
            AllowAutoRedirect = false,
            BaseAddress = new Uri("https://localhost")
        });
    }
    
    [Fact]
    public async Task GetUsers_ReturnsSuccess()
    {
        var response = await _client.GetAsync("/api/users");
        response.EnsureSuccessStatusCode();
    }
}
```

**Customization:**
```csharp
var factory = new WebApplicationFactory<Program>()
    .WithWebHostBuilder(builder =>
    {
        builder.ConfigureServices(services =>
        {
            // Replace services for testing
            services.RemoveAll<IUserRepository>();
            services.AddScoped<IUserRepository, FakeUserRepository>();
        });
        
        builder.ConfigureTestServices(services =>
        {
            // Add test-specific services
            services.AddScoped<ITestDataService, TestDataService>();
        });
        
        builder.UseEnvironment("Testing");
    });
```

---

### Mock<T> (Moq Library)

**Namespace:** `Moq`

**Purpose:** Create mock objects for testing

**Important Members:**

| Member | Description |
|--------|-------------|
| `Setup()` | Configure method behavior |
| `Returns()` | Specify return value |
| `ReturnsAsync()` | Specify async return value |
| `Throws()` | Specify exception to throw |
| `Callback()` | Execute custom logic |
| `Verify()` | Verify method was called |
| `VerifyNoOtherCalls()` | Ensure no other methods called |
| `Object` | Get mocked instance |

**Setup Methods:**
```csharp
var mock = new Mock<IUserRepository>();

// Return value
mock.Setup(r => r.GetByIdAsync(1))
    .ReturnsAsync(new User { Id = 1 });

// Throw exception
mock.Setup(r => r.DeleteAsync(999))
    .ThrowsAsync(new NotFoundException());

// Callback
mock.Setup(r => r.AddAsync(It.IsAny<User>()))
    .Callback<User>(u => Console.WriteLine($"Adding {u.Name}"))
    .ReturnsAsync((User u) => u);

// Sequential returns
mock.SetupSequence(r => r.GetNextIdAsync())
    .ReturnsAsync(1)
    .ReturnsAsync(2)
    .ReturnsAsync(3);
```

**Verify Methods:**
```csharp
// Verify called once
mock.Verify(r => r.GetByIdAsync(1), Times.Once);

// Verify never called
mock.Verify(r => r.DeleteAsync(It.IsAny<int>()), Times.Never);

// Verify called at least once
mock.Verify(r => r.AddAsync(It.IsAny<User>()), Times.AtLeastOnce);

// Verify with specific arguments
mock.Verify(r => r.UpdateAsync(It.Is<User>(u => u.Id == 1)), Times.Once);

// Verify all setups were called
mock.VerifyAll();
```

---

### It Class (Moq Argument Matching)

**Namespace:** `Moq`

**Purpose:** Match method arguments in mock setups

**Static Methods:**

| Method | Description | Example |
|--------|-------------|---------|
| `It.IsAny<T>()` | Any value of type T | `It.IsAny<int>()` |
| `It.Is<T>(predicate)` | Value matching predicate | `It.Is<int>(x => x > 0)` |
| `It.IsIn<T>(values)` | Value in collection | `It.IsIn(1, 2, 3)` |
| `It.IsInRange<T>(from, to, range)` | Value in range | `It.IsInRange(1, 100, Range.Inclusive)` |
| `It.IsRegex(pattern)` | String matching regex | `It.IsRegex(@"^\d+$")` |

**Usage Examples:**
```csharp
// Any integer
mock.Setup(r => r.GetByIdAsync(It.IsAny<int>()))
    .ReturnsAsync(new User());

// Positive integers only
mock.Setup(r => r.GetByIdAsync(It.Is<int>(id => id > 0)))
    .ReturnsAsync(new User());

// Specific values
mock.Setup(r => r.GetByIdAsync(It.IsIn(1, 2, 3)))
    .ReturnsAsync(new User());

// Range
mock.Setup(r => r.GetByIdAsync(It.IsInRange(1, 100, Range.Inclusive)))
    .ReturnsAsync(new User());

// Complex object matching
mock.Setup(r => r.UpdateAsync(It.Is<User>(u => 
    u.Id > 0 && 
    !string.IsNullOrEmpty(u.Name) &&
    u.Email.Contains("@")
))).ReturnsAsync(true);
```

---

### FluentAssertions Extensions

**Namespace:** `FluentAssertions`

**Purpose:** Readable assertions

**Common Assertions:**

```csharp
// Equality
result.Should().Be(expected);
result.Should().NotBe(unexpected);

// Null checks
result.Should().BeNull();
result.Should().NotBeNull();

// Type checks
result.Should().BeOfType<User>();
result.Should().BeAssignableTo<IUser>();

// Strings
result.Should().StartWith("Hello");
result.Should().EndWith("World");
result.Should().Contain("middle");
result.Should().Match("*pattern*");

// Collections
list.Should().HaveCount(5);
list.Should().Contain(user);
list.Should().NotBeEmpty();
list.Should().BeInAscendingOrder(u => u.Name);
list.Should().AllSatisfy(u => u.IsActive.Should().BeTrue());

// Exceptions
action.Should().Throw<InvalidOperationException>();
action.Should().ThrowAsync<NotFoundException>()
      .WithMessage("User not found");

// Async
(await func()).Should().Be(expected);
await action.Should().ThrowAsync<Exception>();
```

---

### Times (Moq Verification)

**Namespace:** `Moq`

**Purpose:** Specify expected call count

**Static Members:**

| Member | Description |
|--------|-------------|
| `Times.Never` | Should not be called |
| `Times.Once` | Should be called exactly once |
| `Times.Exactly(n)` | Should be called exactly n times |
| `Times.AtLeastOnce` | Should be called at least once |
| `Times.AtLeast(n)` | Should be called at least n times |
| `Times.AtMostOnce` | Should be called at most once |
| `Times.AtMost(n)` | Should be called at most n times |
| `Times.Between(min, max, range)` | Should be called between min and max |

**Usage:**
```csharp
mock.Verify(r => r.GetByIdAsync(1), Times.Once);
mock.Verify(r => r.AddAsync(It.IsAny<User>()), Times.Never);
mock.Verify(r => r.UpdateAsync(It.IsAny<User>()), Times.AtLeastOnce);
mock.Verify(r => r.DeleteAsync(It.IsAny<int>()), Times.Exactly(3));
mock.Verify(r => r.GetAllAsync(), Times.AtMost(2));
mock.Verify(r => r.SaveAsync(), Times.Between(1, 5, Range.Inclusive));
```

---

### TestServer

**Namespace:** `Microsoft.AspNetCore.TestHost`

**Purpose:** Lightweight in-memory test server

**Important Members:**

| Member | Type | Description |
|--------|------|-------------|
| `CreateClient()` | `HttpClient` | Create HTTP client |
| `CreateRequest(path)` | `RequestBuilder` | Build custom request |
| `CreateWebSocketClient()` | `WebSocketClient` | Create WebSocket client |
| `Services` | `IServiceProvider` | Access DI container |
| `Host` | `IHost` | Access host |

**Usage:**
```csharp
using var host = await new HostBuilder()
    .ConfigureWebHost(webBuilder =>
    {
        webBuilder
            .UseTestServer()
            .ConfigureServices(services =>
            {
                services.AddRouting();
                services.AddControllers();
            })
            .Configure(app =>
            {
                app.UseRouting();
                app.UseEndpoints(endpoints =>
                {
                    endpoints.MapControllers();
                });
            });
    })
    .StartAsync();

var client = host.GetTestClient();
var response = await client.GetAsync("/api/users");
```

---

## 12. Configuration Deep-Dive

### Pattern 1: In-Memory Database (Hardcoded)

**When to use:** Simple unit tests, no real DB needed

```csharp
public class UserRepositoryTests
{
    private ApplicationDbContext CreateContext()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName: "TestDb")
            .Options;
        
        return new ApplicationDbContext(options);
    }
    
    [Fact]
    public async Task Test()
    {
        using var context = CreateContext();
        var repository = new UserRepository(context);
        // ...
    }
}
```

**Pros:**
- ✅ Fast
- ✅ Simple
- ✅ No external dependencies

**Cons:**
- ❌ Not real SQL
- ❌ Can't test database-specific features
- ❌ Potential false positives

---

### Pattern 2: SQLite In-Memory (Code-based)

**When to use:** Need real SQL behavior

```csharp
public class UserRepositoryTests : IDisposable
{
    private readonly SqliteConnection _connection;
    private readonly ApplicationDbContext _context;
    
    public UserRepositoryTests()
    {
        _connection = new SqliteConnection("DataSource=:memory:");
        _connection.Open();
        
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlite(_connection)
            .Options;
        
        _context = new ApplicationDbContext(options);
        _context.Database.EnsureCreated();
    }
    
    public void Dispose()
    {
        _context.Dispose();
        _connection.Dispose();
    }
}
```

**Pros:**
- ✅ Real SQL behavior
- ✅ Fast
- ✅ Test migrations
- ✅ Better than EF In-Memory

**Cons:**
- ⚠️ SQLite != SQL Server/PostgreSQL
- ⚠️ Some features different

---

### Pattern 3: Test Containers (Production-like)

**When to use:** Integration tests, CI/CD

```csharp
public class UserRepositoryIntegrationTests : IAsyncLifetime
{
    private PostgreSqlContainer _container;
    private ApplicationDbContext _context;
    
    public async Task InitializeAsync()
    {
        _container = new PostgreSqlBuilder()
            .WithImage("postgres:15")
            .Build();
        
        await _container.StartAsync();
        
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseNpgsql(_container.GetConnectionString())
            .Options;
        
        _context = new ApplicationDbContext(options);
        await _context.Database.MigrateAsync();
    }
    
    public async Task DisposeAsync()
    {
        await _context.DisposeAsync();
        await _container.DisposeAsync();
    }
}
```

**Pros:**
- ✅ Exact production database
- ✅ Test real features
- ✅ Test migrations
- ✅ Repeatable

**Cons:**
- ⚠️ Slower
- ⚠️ Requires Docker
- ⚠️ More complex setup

---

### Configuration Comparison

| Approach | Speed | Accuracy | Complexity | Use Case |
|----------|-------|----------|------------|----------|
| **In-Memory** | ⚡⚡⚡ | ⚠️ Low | ⭐ Simple | Unit tests |
| **SQLite** | ⚡⚡ | ⚠️ Medium | ⭐⭐ Medium | Unit/Integration |
| **Test Containers** | ⚡ | ✅ High | ⭐⭐⭐ Complex | Integration/CI |
| **Real DB** | 🐌 | ✅ Highest | ⭐⭐⭐ Complex | Manual testing |

---

## 13. Advanced Testing Topics

### Testing Background Services

```csharp
public class BackgroundServiceTests
{
    [Fact]
    public async Task BackgroundService_Executes_PeriodicallyAsync()
    {
        // Arrange
        var mockService = new Mock<IDataProcessor>();
        var service = new DataProcessingBackgroundService(mockService.Object);
        var cts = new CancellationTokenSource();
        
        // Act
        var task = service.StartAsync(cts.Token);
        await Task.Delay(TimeSpan.FromSeconds(5));
        cts.Cancel();
        
        // Assert
        mockService.Verify(s => s.ProcessAsync(), Times.AtLeastOnce);
    }
}
```

---

### Testing SignalR Hubs

```csharp
public class ChatHubTests
{
    [Fact]
    public async Task SendMessage_BroadcastsToAll()
    {
        // Arrange
        var mockClients = new Mock<IHubCallerClients>();
        var mockClientProxy = new Mock<IClientProxy>();
        
        mockClients.Setup(c => c.All).Returns(mockClientProxy.Object);
        
        var hub = new ChatHub
        {
            Clients = mockClients.Object
        };
        
        // Act
        await hub.SendMessage("TestUser", "Hello");
        
        // Assert
        mockClientProxy.Verify(
            p => p.SendCoreAsync(
                "ReceiveMessage",
                It.Is<object[]>(o => 
                    o.Length == 2 && 
                    o[0].ToString() == "TestUser" && 
                    o[1].ToString() == "Hello"),
                default),
            Times.Once);
    }
}
```

---

### Testing gRPC Services

```csharp
public class GreeterServiceTests
{
    [Fact]
    public async Task SayHello_ReturnsGreeting()
    {
        // Arrange
        var service = new GreeterService();
        var context = TestServerCallContext.Create();
        var request = new HelloRequest { Name = "World" };
        
        // Act
        var response = await service.SayHello(request, context);
        
        // Assert
        response.Message.Should().Be("Hello World");
    }
}

// Helper class
public class TestServerCallContext : ServerCallContext
{
    public static ServerCallContext Create()
    {
        return new TestServerCallContext();
    }
    
    protected override Task WriteResponseHeadersAsyncCore(Metadata responseHeaders)
        => Task.CompletedTask;
    
    protected override ContextPropagationToken CreatePropagationTokenCore(
        ContextPropagationOptions options)
        => null;
    
    // ... implement other abstract members
}
```

---

### Testing with Time (Fake Clock)

```csharp
public interface ISystemClock
{
    DateTime UtcNow { get; }
}

public class SystemClock : ISystemClock
{
    public DateTime UtcNow => DateTime.UtcNow;
}

public class FakeSystemClock : ISystemClock
{
    public DateTime UtcNow { get; set; } = DateTime.UtcNow;
}

// In tests
[Fact]
public void Service_ChecksExpiration_UsingClock()
{
    // Arrange
    var fakeClock = new FakeSystemClock();
    fakeClock.UtcNow = new DateTime(2024, 1, 1);
    
    var service = new SubscriptionService(fakeClock);
    var subscription = new Subscription 
    { 
        ExpiresAt = new DateTime(2023, 12, 31) 
    };
    
    // Act
    var isExpired = service.IsExpired(subscription);
    
    // Assert
    isExpired.Should().BeTrue();
}
```

---

### Snapshot Testing

```bash
dotnet add package Verify.Xunit
```

```csharp
public class SnapshotTests
{
    [Fact]
    public Task UserDto_MatchesSnapshot()
    {
        var user = new UserDto
        {
            Id = 1,
            Name = "John Doe",
            Email = "john@example.com",
            CreatedAt = new DateTime(2024, 1, 1)
        };
        
        return Verifier.Verify(user);
    }
}
```

---

### Property-Based Testing

```bash
dotnet add package FsCheck.Xunit
```

```csharp
using FsCheck;
using FsCheck.Xunit;

public class PropertyTests
{
    [Property]
    public bool Reverse_Reverse_IsOriginal(int[] array)
    {
        var reversed = array.Reverse().Reverse();
        return reversed.SequenceEqual(array);
    }
    
    [Property]
    public Property Add_IsCommutative(int a, int b)
    {
        return (a + b == b + a).ToProperty();
    }
}
```

---

### Mutation Testing

```bash
dotnet tool install --global dotnet-stryker
stryker
```

**What it does:** Changes your code slightly (mutates) to verify tests catch the changes

---

## 14. Performance Testing

### Benchmark Testing with BenchmarkDotNet

```bash
dotnet add package BenchmarkDotNet
```

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
public class HashingBenchmarks
{
    private const string TestData = "Hello World";
    
    [Benchmark]
    public string MD5Hash()
    {
        using var md5 = MD5.Create();
        var hash = md5.ComputeHash(Encoding.UTF8.GetBytes(TestData));
        return Convert.ToBase64String(hash);
    }
    
    [Benchmark]
    public string SHA256Hash()
    {
        using var sha = SHA256.Create();
        var hash = sha.ComputeHash(Encoding.UTF8.GetBytes(TestData));
        return Convert.ToBase64String(hash);
    }
}

// Run benchmarks
public class Program
{
    public static void Main(string[] args)
    {
        BenchmarkRunner.Run<HashingBenchmarks>();
    }
}
```

---

### Load Testing

```bash
dotnet add package NBomber
```

```csharp
using NBomber.CSharp;
using NBomber.Http.CSharp;

public class LoadTests
{
    [Fact]
    public void LoadTest_ApiEndpoint()
    {
        var scenario = Scenario.Create("api_load_test", async context =>
        {
            var request = Http.CreateRequest("GET", "https://localhost:5001/api/users")
                .WithHeader("Accept", "application/json");
            
            var response = await Http.Send(request, context);
            return response;
        })
        .WithLoadSimulations(
            Simulation.InjectPerSec(rate: 100, during: TimeSpan.FromSeconds(30))
        );
        
        var stats = NBomberRunner
            .RegisterScenarios(scenario)
            .Run();
        
        // Assert performance requirements
        var okCount = stats.ScenarioStats[0].Ok.Request.Count;
        okCount.Should().BeGreaterThan(2500); // At least 2500 successful requests
    }
}
```

---

### Response Time Testing

```csharp
[Fact]
public async Task GetUsers_ResponseTime_UnderThreshold()
{
    // Arrange
    var stopwatch = Stopwatch.StartNew();
    
    // Act
    var response = await _client.GetAsync("/api/users");
    stopwatch.Stop();
    
    // Assert
    response.EnsureSuccessStatusCode();
    stopwatch.ElapsedMilliseconds.Should().BeLessThan(100); // Must respond under 100ms
}
```

---

## Summary: Complete Testing Checklist

**Unit Testing:**
- [ ] Test one thing per test
- [ ] Use AAA pattern (Arrange, Act, Assert)
- [ ] Mock external dependencies
- [ ] Fast tests (< 100ms)
- [ ] Test edge cases
- [ ] Achieve 70-80% code coverage

**Integration Testing:**
- [ ] Use WebApplicationFactory
- [ ] Test with realistic database
- [ ] Test complete workflows
- [ ] Verify HTTP responses
- [ ] Test authentication/authorization
- [ ] Clean up test data

**Best Practices:**
- [ ] Clear, descriptive test names
- [ ] Independent tests
- [ ] Deterministic (no random data)
- [ ] Use test builders for complex data
- [ ] Organize by feature/layer
- [ ] Run tests in CI/CD
- [ ] Monitor code coverage
- [ ] Regular test maintenance

**Common Patterns:**
- [ ] Use IClassFixture for shared setup
- [ ] Use IAsyncLifetime for async setup
- [ ] Create custom WebApplicationFactory
- [ ] Use FluentAssertions for readability
- [ ] Mock with Moq or NSubstitute
- [ ] Test async methods properly
- [ ] Handle exceptions in tests

---

**This completes the Testing guide combining practical hands-on content with deep technical reference!**
