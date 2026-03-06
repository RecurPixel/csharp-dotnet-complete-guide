# ASP.NET Core Testing - Complete Guide
## Practical Guide + Technical Reference

---

## 📋 Table of Contents

### Part 1: Practical Guide (Hands-On)
1. Testing Fundamentals
2. Unit Testing (3 Approaches)
3. Integration Testing (3 Approaches)
4. Testing Controllers & APIs
5. Testing with Database (DbContext)
6. Mocking and Test Doubles
7. Common Testing Patterns
8. Troubleshooting Common Issues
9. Best Practices

### Part 2: Technical Reference (Deep Dive)
10. Important Interfaces & Classes Reference
11. Configuration Deep-Dive
12. Advanced Testing Topics
13. Performance Testing Basics

---

# PART 1: PRACTICAL GUIDE

---

## 1. Testing Fundamentals

**Simple Definition:** Testing is writing code to verify your application works correctly.

**Think of it like:** Quality control inspectors checking products before they ship. Each test checks one thing works as expected.

### The Testing Pyramid

```
                  /\
                 /  \
                / UI \           Few, slow, expensive
               /------\
              /        \
             / Integration\     Some, medium speed
            /------------\
           /              \
          /   Unit Tests   \   Many, fast, cheap
         /------------------\
```

**Key Point:** Most tests should be unit tests (fast, isolated), fewer integration tests (test components together), and minimal UI/E2E tests.

---

### Test Types Overview

| Test Type | What It Tests | Speed | Complexity | Quantity |
|-----------|--------------|-------|------------|----------|
| **Unit** | Single method/class | ⚡ Very Fast | Simple | Many (70%) |
| **Integration** | Multiple components | ⏱️ Medium | Medium | Some (20%) |
| **E2E** | Complete user flows | 🐌 Slow | Complex | Few (10%) |

---

### Popular Testing Frameworks

**xUnit** ⭐ RECOMMENDED (Official .NET recommendation)
- Modern, clean API
- Parallel test execution
- No [TestClass] attributes needed
- Used by ASP.NET Core team

**NUnit**
- Similar to JUnit/Jest
- Rich assertion library
- Good for complex scenarios

**MSTest**
- Built into Visual Studio
- Good for simple scenarios
- Less features than xUnit/NUnit

**This guide uses xUnit for examples**

---

## 2. Unit Testing (3 Approaches)

### Method 1: Simple Tests (No Framework Setup)

**When to use:**
- ✅ Testing pure functions
- ✅ Testing business logic classes
- ✅ Quick prototyping
- ❌ Testing controllers/services with dependencies

**Step 1: Install xUnit**

```bash
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
dotnet add package Microsoft.NET.Test.Sdk
```

**Step 2: Create Test Class**

```csharp
using Xunit;

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
    [InlineData(-1, 1, 0)]
    [InlineData(100, 200, 300)]
    public void Add_VariousInputs_ReturnsCorrectSum(int a, int b, int expected)
    {
        // Arrange
        var calculator = new Calculator();
        
        // Act
        var result = calculator.Add(a, b);
        
        // Assert
        Assert.Equal(expected, result);
    }
}
```

**Step 3: Run Tests**

```bash
dotnet test
```

---

### Method 2: Testing with Dependencies (Mocking)

**When to use:**
- ✅ Testing services with dependencies
- ✅ Need to control external behavior
- ✅ Production code

**Step 1: Install Moq**

```bash
dotnet add package Moq
```

**Step 2: Create Service to Test**

```csharp
public interface IUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task<bool> ExistsAsync(int id);
}

public class UserService
{
    private readonly IUserRepository _repository;
    private readonly ILogger<UserService> _logger;
    
    public UserService(IUserRepository repository, ILogger<UserService> logger)
    {
        _repository = repository;
        _logger = logger;
    }
    
    public async Task<User> GetUserAsync(int id)
    {
        if (id <= 0)
            throw new ArgumentException("ID must be positive", nameof(id));
        
        var exists = await _repository.ExistsAsync(id);
        if (!exists)
        {
            _logger.LogWarning("User {UserId} not found", id);
            return null;
        }
        
        return await _repository.GetByIdAsync(id);
    }
}
```

**Step 3: Write Tests with Mocks**

```csharp
using Moq;
using Xunit;
using Microsoft.Extensions.Logging;

public class UserServiceTests
{
    private readonly Mock<IUserRepository> _mockRepository;
    private readonly Mock<ILogger<UserService>> _mockLogger;
    private readonly UserService _service;
    
    public UserServiceTests()
    {
        // Create mocks
        _mockRepository = new Mock<IUserRepository>();
        _mockLogger = new Mock<ILogger<UserService>>();
        
        // Create service with mocks
        _service = new UserService(_mockRepository.Object, _mockLogger.Object);
    }
    
    [Fact]
    public async Task GetUserAsync_UserExists_ReturnsUser()
    {
        // Arrange
        var userId = 1;
        var expectedUser = new User { Id = userId, Name = "John" };
        
        _mockRepository.Setup(r => r.ExistsAsync(userId))
            .ReturnsAsync(true);
        _mockRepository.Setup(r => r.GetByIdAsync(userId))
            .ReturnsAsync(expectedUser);
        
        // Act
        var result = await _service.GetUserAsync(userId);
        
        // Assert
        Assert.NotNull(result);
        Assert.Equal(expectedUser.Id, result.Id);
        Assert.Equal(expectedUser.Name, result.Name);
        
        // Verify methods were called
        _mockRepository.Verify(r => r.ExistsAsync(userId), Times.Once);
        _mockRepository.Verify(r => r.GetByIdAsync(userId), Times.Once);
    }
    
    [Fact]
    public async Task GetUserAsync_UserDoesNotExist_ReturnsNull()
    {
        // Arrange
        var userId = 999;
        
        _mockRepository.Setup(r => r.ExistsAsync(userId))
            .ReturnsAsync(false);
        
        // Act
        var result = await _service.GetUserAsync(userId);
        
        // Assert
        Assert.Null(result);
        
        // Verify GetByIdAsync was NOT called
        _mockRepository.Verify(r => r.GetByIdAsync(It.IsAny<int>()), Times.Never);
        
        // Verify warning was logged
        _mockLogger.Verify(
            x => x.Log(
                LogLevel.Warning,
                It.IsAny<EventId>(),
                It.Is<It.IsAnyType>((v, t) => v.ToString().Contains("not found")),
                It.IsAny<Exception>(),
                It.IsAny<Func<It.IsAnyType, Exception, string>>()),
            Times.Once);
    }
    
    [Theory]
    [InlineData(0)]
    [InlineData(-1)]
    [InlineData(-100)]
    public async Task GetUserAsync_InvalidId_ThrowsArgumentException(int invalidId)
    {
        // Act & Assert
        await Assert.ThrowsAsync<ArgumentException>(
            () => _service.GetUserAsync(invalidId));
    }
}
```

---

### Method 3: Testing with Test Fixtures (Shared Setup)

**When to use:**
- ✅ Expensive setup (database, files)
- ✅ Shared resources across tests
- ✅ Need cleanup after tests

**Step 1: Create Fixture**

```csharp
public class DatabaseFixture : IDisposable
{
    public DbContextOptions<ApplicationDbContext> Options { get; }
    
    public DatabaseFixture()
    {
        // Setup in-memory database
        Options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;
        
        // Seed test data
        using (var context = new ApplicationDbContext(Options))
        {
            context.Users.AddRange(
                new User { Id = 1, Name = "Alice", Email = "alice@test.com" },
                new User { Id = 2, Name = "Bob", Email = "bob@test.com" }
            );
            context.SaveChanges();
        }
    }
    
    public void Dispose()
    {
        // Cleanup if needed
    }
}
```

**Step 2: Use Fixture in Tests**

```csharp
public class UserRepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;
    
    public UserRepositoryTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }
    
    [Fact]
    public async Task GetByIdAsync_ExistingUser_ReturnsUser()
    {
        // Arrange
        using var context = new ApplicationDbContext(_fixture.Options);
        var repository = new UserRepository(context);
        
        // Act
        var user = await repository.GetByIdAsync(1);
        
        // Assert
        Assert.NotNull(user);
        Assert.Equal("Alice", user.Name);
    }
    
    [Fact]
    public async Task GetByIdAsync_NonExistentUser_ReturnsNull()
    {
        // Arrange
        using var context = new ApplicationDbContext(_fixture.Options);
        var repository = new UserRepository(context);
        
        // Act
        var user = await repository.GetByIdAsync(999);
        
        // Assert
        Assert.Null(user);
    }
}
```

---

## 3. Integration Testing (3 Approaches)

### Method 1: WebApplicationFactory (Standard Approach)

**When to use:**
- ✅ Testing HTTP endpoints
- ✅ Full request/response cycle
- ✅ Middleware testing

**Step 1: Install Required Packages**

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing
```

**Step 2: Create Test Project Setup**

```csharp
using Microsoft.AspNetCore.Mvc.Testing;

public class BasicIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;
    
    public BasicIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task Get_EndpointReturnsSuccess()
    {
        // Act
        var response = await _client.GetAsync("/api/users");
        
        // Assert
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json; charset=utf-8", 
            response.Content.Headers.ContentType.ToString());
    }
    
    [Fact]
    public async Task Get_ReturnsExpectedJson()
    {
        // Act
        var response = await _client.GetAsync("/api/users/1");
        var content = await response.Content.ReadAsStringAsync();
        var user = JsonSerializer.Deserialize<User>(content, 
            new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
        
        // Assert
        Assert.NotNull(user);
        Assert.Equal(1, user.Id);
    }
}
```

**Note:** Make Program class accessible in your API project:

```csharp
// Program.cs - Add at bottom
public partial class Program { } // Make Program accessible for tests
```

---

### Method 2: Custom WebApplicationFactory (Production Approach) ⭐ RECOMMENDED

**When to use:**
- ✅ Need custom configuration
- ✅ Replace real services with test doubles
- ✅ Use test database
- ✅ Production testing scenarios

**Step 1: Create Custom Factory**

```csharp
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
            
            // Replace real services with test versions
            services.Remove(services.SingleOrDefault(
                d => d.ServiceType == typeof(IEmailService)));
            services.AddSingleton<IEmailService, FakeEmailService>();
            
            // Build service provider and seed database
            var sp = services.BuildServiceProvider();
            using (var scope = sp.CreateScope())
            {
                var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
                db.Database.EnsureCreated();
                
                // Seed test data
                SeedTestData(db);
            }
        });
        
        builder.ConfigureAppConfiguration((context, config) =>
        {
            // Add test configuration
            config.AddInMemoryCollection(new Dictionary<string, string>
            {
                ["TestSetting"] = "TestValue",
                ["ConnectionStrings:Default"] = "test-connection"
            });
        });
    }
    
    private void SeedTestData(ApplicationDbContext context)
    {
        context.Users.AddRange(
            new User { Id = 1, Name = "Test User 1", Email = "test1@test.com" },
            new User { Id = 2, Name = "Test User 2", Email = "test2@test.com" }
        );
        context.SaveChanges();
    }
}
```

**Step 2: Create Fake Services**

```csharp
public class FakeEmailService : IEmailService
{
    public List<string> SentEmails { get; } = new();
    
    public Task SendAsync(string to, string subject, string body)
    {
        SentEmails.Add($"To: {to}, Subject: {subject}");
        return Task.CompletedTask;
    }
}
```

**Step 3: Use Custom Factory**

```csharp
public class UserIntegrationTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;
    private readonly CustomWebApplicationFactory _factory;
    
    public UserIntegrationTests(CustomWebApplicationFactory factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task GetUsers_ReturnsSeededData()
    {
        // Act
        var response = await _client.GetAsync("/api/users");
        var content = await response.Content.ReadAsStringAsync();
        var users = JsonSerializer.Deserialize<List<User>>(content,
            new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
        
        // Assert
        Assert.NotNull(users);
        Assert.Equal(2, users.Count);
        Assert.Contains(users, u => u.Name == "Test User 1");
    }
    
    [Fact]
    public async Task CreateUser_SendsEmail()
    {
        // Arrange
        var newUser = new { Name = "New User", Email = "new@test.com" };
        var content = new StringContent(
            JsonSerializer.Serialize(newUser),
            Encoding.UTF8,
            "application/json");
        
        // Act
        var response = await _client.PostAsync("/api/users", content);
        
        // Assert
        response.EnsureSuccessStatusCode();
        
        // Verify email was sent (using fake service)
        var emailService = _factory.Services
            .GetRequiredService<IEmailService>() as FakeEmailService;
        Assert.NotNull(emailService);
        Assert.Single(emailService.SentEmails);
        Assert.Contains("new@test.com", emailService.SentEmails[0]);
    }
}
```

---

### Method 3: Integration Tests with Authentication

**When to use:**
- ✅ Testing protected endpoints
- ✅ Testing authorization
- ✅ Role-based access

**Step 1: Create Test Authentication Handler**

```csharp
public class TestAuthHandler : AuthenticationHandler<AuthenticationSchemeOptions>
{
    public const string SchemeName = "TestScheme";
    
    public TestAuthHandler(
        IOptionsMonitor<AuthenticationSchemeOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock)
        : base(options, logger, encoder, clock)
    {
    }
    
    protected override Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, "test-user-id"),
            new Claim(ClaimTypes.Name, "Test User"),
            new Claim(ClaimTypes.Email, "test@test.com"),
            new Claim(ClaimTypes.Role, "Admin")
        };
        
        var identity = new ClaimsIdentity(claims, SchemeName);
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, SchemeName);
        
        return Task.FromResult(AuthenticateResult.Success(ticket));
    }
}
```

**Step 2: Create Factory with Test Auth**

```csharp
public class AuthenticatedWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Add test authentication
            services.AddAuthentication(TestAuthHandler.SchemeName)
                .AddScheme<AuthenticationSchemeOptions, TestAuthHandler>(
                    TestAuthHandler.SchemeName, options => { });
            
            // Configure other test services...
        });
    }
}
```

**Step 3: Test Protected Endpoints**

```csharp
public class ProtectedEndpointTests : IClassFixture<AuthenticatedWebApplicationFactory>
{
    private readonly HttpClient _client;
    
    public ProtectedEndpointTests(AuthenticatedWebApplicationFactory factory)
    {
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task GetProtectedResource_WithAuth_ReturnsSuccess()
    {
        // Act
        var response = await _client.GetAsync("/api/admin/users");
        
        // Assert
        response.EnsureSuccessStatusCode();
    }
    
    [Fact]
    public async Task GetProtectedResource_WithoutAuth_ReturnsUnauthorized()
    {
        // Arrange
        var factory = new WebApplicationFactory<Program>(); // No auth
        var client = factory.CreateClient();
        
        // Act
        var response = await client.GetAsync("/api/admin/users");
        
        // Assert
        Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
    }
}
```

---

## 4. Testing Controllers & APIs

### Testing Controller Actions Directly

**Simple Controller Test:**

```csharp
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetUser(int id)
    {
        var user = await _userService.GetUserAsync(id);
        
        if (user == null)
            return NotFound();
        
        return Ok(user);
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserDto dto)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        var user = await _userService.CreateUserAsync(dto);
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
}
```

**Controller Unit Tests:**

```csharp
public class UsersControllerTests
{
    private readonly Mock<IUserService> _mockUserService;
    private readonly UsersController _controller;
    
    public UsersControllerTests()
    {
        _mockUserService = new Mock<IUserService>();
        _controller = new UsersController(_mockUserService.Object);
    }
    
    [Fact]
    public async Task GetUser_ExistingUser_ReturnsOkResult()
    {
        // Arrange
        var userId = 1;
        var user = new User { Id = userId, Name = "Test" };
        _mockUserService.Setup(s => s.GetUserAsync(userId))
            .ReturnsAsync(user);
        
        // Act
        var result = await _controller.GetUser(userId);
        
        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var returnedUser = Assert.IsType<User>(okResult.Value);
        Assert.Equal(userId, returnedUser.Id);
    }
    
    [Fact]
    public async Task GetUser_NonExistentUser_ReturnsNotFound()
    {
        // Arrange
        _mockUserService.Setup(s => s.GetUserAsync(It.IsAny<int>()))
            .ReturnsAsync((User)null);
        
        // Act
        var result = await _controller.GetUser(999);
        
        // Assert
        Assert.IsType<NotFoundResult>(result);
    }
    
    [Fact]
    public async Task CreateUser_ValidData_ReturnsCreatedResult()
    {
        // Arrange
        var dto = new CreateUserDto { Name = "New User", Email = "new@test.com" };
        var createdUser = new User { Id = 1, Name = dto.Name, Email = dto.Email };
        
        _mockUserService.Setup(s => s.CreateUserAsync(dto))
            .ReturnsAsync(createdUser);
        
        // Act
        var result = await _controller.CreateUser(dto);
        
        // Assert
        var createdResult = Assert.IsType<CreatedAtActionResult>(result);
        Assert.Equal(nameof(UsersController.GetUser), createdResult.ActionName);
        Assert.Equal(createdUser.Id, ((User)createdResult.Value).Id);
    }
    
    [Fact]
    public async Task CreateUser_InvalidModel_ReturnsBadRequest()
    {
        // Arrange
        _controller.ModelState.AddModelError("Name", "Required");
        var dto = new CreateUserDto();
        
        // Act
        var result = await _controller.CreateUser(dto);
        
        // Assert
        Assert.IsType<BadRequestObjectResult>(result);
    }
}
```

---

## 5. Testing with Database (DbContext)

### Method 1: In-Memory Database (Quick Tests)

**When to use:**
- ✅ Fast unit tests
- ✅ Simple queries
- ❌ Testing raw SQL
- ❌ Testing database-specific features

```csharp
public class UserRepositoryTests
{
    private DbContextOptions<ApplicationDbContext> CreateNewContextOptions()
    {
        // Create unique database for each test
        return new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;
    }
    
    [Fact]
    public async Task GetActiveUsers_ReturnsOnlyActiveUsers()
    {
        // Arrange
        var options = CreateNewContextOptions();
        
        // Seed data
        using (var context = new ApplicationDbContext(options))
        {
            context.Users.AddRange(
                new User { Id = 1, Name = "Active", IsActive = true },
                new User { Id = 2, Name = "Inactive", IsActive = false },
                new User { Id = 3, Name = "Active2", IsActive = true }
            );
            await context.SaveChangesAsync();
        }
        
        // Act & Assert
        using (var context = new ApplicationDbContext(options))
        {
            var repository = new UserRepository(context);
            var activeUsers = await repository.GetActiveUsersAsync();
            
            Assert.Equal(2, activeUsers.Count());
            Assert.All(activeUsers, u => Assert.True(u.IsActive));
        }
    }
}
```

---

### Method 2: SQLite In-Memory (Better for Testing)

**When to use:**
- ✅ More realistic database behavior
- ✅ Testing complex queries
- ✅ Testing migrations
- ✅ Production-like tests

```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
```

```csharp
public class DatabaseTestBase : IDisposable
{
    protected ApplicationDbContext Context { get; }
    private readonly SqliteConnection _connection;
    
    public DatabaseTestBase()
    {
        // Create and open connection (keeps database alive)
        _connection = new SqliteConnection("DataSource=:memory:");
        _connection.Open();
        
        // Create options using the connection
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlite(_connection)
            .Options;
        
        // Create context
        Context = new ApplicationDbContext(options);
        
        // Create schema
        Context.Database.EnsureCreated();
    }
    
    public void Dispose()
    {
        Context.Dispose();
        _connection.Close();
    }
}

public class UserRepositoryTests : DatabaseTestBase
{
    [Fact]
    public async Task Add_NewUser_SavesSuccessfully()
    {
        // Arrange
        var repository = new UserRepository(Context);
        var user = new User 
        { 
            Name = "Test User", 
            Email = "test@test.com",
            CreatedAt = DateTime.UtcNow
        };
        
        // Act
        await repository.AddAsync(user);
        await Context.SaveChangesAsync();
        
        // Assert
        var savedUser = await Context.Users.FindAsync(user.Id);
        Assert.NotNull(savedUser);
        Assert.Equal("Test User", savedUser.Name);
    }
    
    [Fact]
    public async Task GetByEmail_ExistingEmail_ReturnsUser()
    {
        // Arrange
        Context.Users.Add(new User 
        { 
            Name = "John", 
            Email = "john@test.com" 
        });
        await Context.SaveChangesAsync();
        
        var repository = new UserRepository(Context);
        
        // Act
        var user = await repository.GetByEmailAsync("john@test.com");
        
        // Assert
        Assert.NotNull(user);
        Assert.Equal("John", user.Name);
    }
}
```

---

### Method 3: Real Database with Docker (Integration Tests)

**When to use:**
- ✅ Full integration testing
- ✅ Testing database-specific features
- ✅ Testing migrations on real DB
- ✅ CI/CD pipelines

**Step 1: docker-compose.test.yml**

```yaml
version: '3.8'
services:
  test-db:
    image: postgres:15
    environment:
      POSTGRES_DB: testdb
      POSTGRES_USER: testuser
      POSTGRES_PASSWORD: testpass
    ports:
      - "5433:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U testuser"]
      interval: 5s
      timeout: 5s
      retries: 5
```

**Step 2: Test Base with Real DB**

```csharp
public class DatabaseIntegrationTestBase : IDisposable
{
    protected ApplicationDbContext Context { get; }
    private readonly string _connectionString;
    
    public DatabaseIntegrationTestBase()
    {
        _connectionString = "Host=localhost;Port=5433;Database=testdb;Username=testuser;Password=testpass";
        
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseNpgsql(_connectionString)
            .Options;
        
        Context = new ApplicationDbContext(options);
        
        // Apply migrations
        Context.Database.Migrate();
        
        // Clean database before each test
        CleanDatabase();
    }
    
    private void CleanDatabase()
    {
        Context.Users.RemoveRange(Context.Users);
        Context.SaveChanges();
    }
    
    public void Dispose()
    {
        CleanDatabase();
        Context.Dispose();
    }
}
```

**Step 3: Run Tests**

```bash
# Start test database
docker-compose -f docker-compose.test.yml up -d

# Run tests
dotnet test

# Stop test database
docker-compose -f docker-compose.test.yml down
```

---

## 6. Mocking and Test Doubles

### Types of Test Doubles

```
Fake    → Working implementation (in-memory database)
Mock    → Records calls, verifies behavior
Stub    → Returns predefined responses
Spy     → Records information about calls
Dummy   → Passed but never used
```

### Using Moq - Complete Guide

**Basic Setup:**

```csharp
// Create mock
var mock = new Mock<IUserRepository>();

// Setup return value
mock.Setup(m => m.GetByIdAsync(1))
    .ReturnsAsync(new User { Id = 1, Name = "John" });

// Use mock
var repository = mock.Object;
var user = await repository.GetByIdAsync(1);

// Verify method was called
mock.Verify(m => m.GetByIdAsync(1), Times.Once);
```

**Common Moq Patterns:**

```csharp
public class MoqExamples
{
    [Fact]
    public void MockingExamples()
    {
        var mock = new Mock<IUserRepository>();
        
        // 1. Return specific value
        mock.Setup(m => m.GetByIdAsync(1))
            .ReturnsAsync(new User { Id = 1 });
        
        // 2. Return different values based on input
        mock.Setup(m => m.GetByIdAsync(It.IsAny<int>()))
            .ReturnsAsync((int id) => new User { Id = id });
        
        // 3. Throw exception
        mock.Setup(m => m.GetByIdAsync(-1))
            .ThrowsAsync(new ArgumentException());
        
        // 4. Match specific conditions
        mock.Setup(m => m.GetByIdAsync(It.Is<int>(id => id > 0)))
            .ReturnsAsync(new User());
        
        // 5. Setup property
        mock.SetupProperty(m => m.Count, 10);
        
        // 6. Setup multiple return values (sequence)
        mock.SetupSequence(m => m.GetNextAsync())
            .ReturnsAsync(new User { Id = 1 })
            .ReturnsAsync(new User { Id = 2 })
            .ReturnsAsync((User)null);
        
        // 7. Callback (execute code when called)
        var calledWith = 0;
        mock.Setup(m => m.GetByIdAsync(It.IsAny<int>()))
            .Callback<int>(id => calledWith = id)
            .ReturnsAsync(new User());
        
        // 8. Verify specific calls
        mock.Verify(m => m.GetByIdAsync(1), Times.Once);
        mock.Verify(m => m.GetByIdAsync(It.IsAny<int>()), Times.AtLeastOnce);
        mock.Verify(m => m.DeleteAsync(It.IsAny<int>()), Times.Never);
        
        // 9. Verify all setups were called
        mock.VerifyAll();
        
        // 10. Verify no other calls made
        mock.VerifyNoOtherCalls();
    }
}
```

**Advanced Mocking Scenarios:**

```csharp
public class AdvancedMockingTests
{
    [Fact]
    public async Task MockingAsyncMethods()
    {
        // Arrange
        var mock = new Mock<IUserService>();
        
        // Setup async method with delay
        mock.Setup(m => m.ProcessAsync(It.IsAny<User>()))
            .Returns(async () =>
            {
                await Task.Delay(100); // Simulate work
                return true;
            });
        
        // Act
        var result = await mock.Object.ProcessAsync(new User());
        
        // Assert
        Assert.True(result);
    }
    
    [Fact]
    public void MockingOutParameters()
    {
        // Arrange
        var mock = new Mock<IValidator>();
        var errorMessage = "Validation failed";
        
        // Setup method with out parameter
        mock.Setup(m => m.TryValidate(It.IsAny<User>(), out errorMessage))
            .Returns(false);
        
        // Act
        var result = mock.Object.TryValidate(new User(), out var message);
        
        // Assert
        Assert.False(result);
        Assert.Equal("Validation failed", message);
    }
    
    [Fact]
    public void MockingEvents()
    {
        // Arrange
        var mock = new Mock<IUserService>();
        var eventRaised = false;
        
        // Subscribe to event
        mock.Object.UserCreated += (sender, args) => eventRaised = true;
        
        // Raise event
        mock.Raise(m => m.UserCreated += null, EventArgs.Empty);
        
        // Assert
        Assert.True(eventRaised);
    }
}
```

---

## 7. Common Testing Patterns

### Arrange-Act-Assert (AAA) Pattern

```csharp
[Fact]
public void ExampleTest()
{
    // Arrange - Set up test data and mocks
    var service = new UserService();
    var userId = 1;
    
    // Act - Execute the operation
    var result = service.GetUser(userId);
    
    // Assert - Verify the outcome
    Assert.NotNull(result);
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
    Assert.Throws<ArgumentNullException>(() => 
        service.CreateUser(null));
}

[Fact]
public async Task MethodAsync_InvalidInput_ThrowsException()
{
    // Arrange
    var service = new UserService();
    
    // Act & Assert
    var exception = await Assert.ThrowsAsync<ArgumentException>(
        () => service.GetUserAsync(-1));
    
    Assert.Equal("ID must be positive", exception.Message);
}
```

---

### Testing Collections

```csharp
[Fact]
public void GetUsers_ReturnsAllUsers()
{
    // Arrange
    var service = new UserService();
    
    // Act
    var users = service.GetAllUsers();
    
    // Assert
    Assert.NotNull(users);
    Assert.NotEmpty(users);
    Assert.Equal(3, users.Count());
    Assert.All(users, u => Assert.NotNull(u.Name));
    Assert.Contains(users, u => u.Id == 1);
    Assert.DoesNotContain(users, u => u.Id == 999);
}
```

---

### Data-Driven Tests with Theory

```csharp
public class TheoryExamples
{
    [Theory]
    [InlineData(1, 1, 2)]
    [InlineData(2, 2, 4)]
    [InlineData(-1, 1, 0)]
    public void Add_VariousInputs_ReturnsExpectedResult(int a, int b, int expected)
    {
        // Arrange
        var calculator = new Calculator();
        
        // Act
        var result = calculator.Add(a, b);
        
        // Assert
        Assert.Equal(expected, result);
    }
    
    [Theory]
    [MemberData(nameof(GetTestUsers))]
    public void ValidateUser_VariousUsers_ValidatesCorrectly(User user, bool expected)
    {
        // Arrange
        var validator = new UserValidator();
        
        // Act
        var result = validator.IsValid(user);
        
        // Assert
        Assert.Equal(expected, result);
    }
    
    public static IEnumerable<object[]> GetTestUsers()
    {
        yield return new object[] { new User { Name = "Valid", Email = "test@test.com" }, true };
        yield return new object[] { new User { Name = "", Email = "test@test.com" }, false };
        yield return new object[] { new User { Name = "Name", Email = "" }, false };
    }
    
    [Theory]
    [ClassData(typeof(UserTestData))]
    public void ProcessUser_VariousScenarios(User user, ProcessResult expected)
    {
        var processor = new UserProcessor();
        var result = processor.Process(user);
        Assert.Equal(expected, result);
    }
}

public class UserTestData : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { new User { Age = 25 }, ProcessResult.Success };
        yield return new object[] { new User { Age = 17 }, ProcessResult.TooYoung };
    }
    
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

---

### Test Setup and Cleanup

```csharp
public class TestLifecycleExample : IDisposable
{
    private readonly UserService _service;
    
    // Constructor - runs before each test
    public TestLifecycleExample()
    {
        _service = new UserService();
    }
    
    [Fact]
    public void Test1()
    {
        // Test logic
    }
    
    [Fact]
    public void Test2()
    {
        // Test logic
    }
    
    // Dispose - runs after each test
    public void Dispose()
    {
        _service?.Dispose();
    }
}

// Async setup/cleanup
public class AsyncLifecycleExample : IAsyncLifetime
{
    private ApplicationDbContext _context;
    
    public async Task InitializeAsync()
    {
        // Async setup
        _context = CreateContext();
        await _context.Database.MigrateAsync();
    }
    
    public async Task DisposeAsync()
    {
        // Async cleanup
        if (_context != null)
        {
            await _context.Database.EnsureDeletedAsync();
            await _context.DisposeAsync();
        }
    }
    
    [Fact]
    public async Task TestExample()
    {
        // Test logic
    }
}
```

---

## 8. Troubleshooting Common Issues

### Problem: Tests Pass Individually but Fail Together

**Cause:** Shared state between tests

**Solution:**
```csharp
// ❌ BAD - Shared static state
public class BadTests
{
    private static int _counter = 0; // Shared!
    
    [Fact]
    public void Test1() => Assert.Equal(0, _counter++);
    
    [Fact]
    public void Test2() => Assert.Equal(0, _counter++); // Will fail!
}

// ✅ GOOD - Isolated state
public class GoodTests
{
    [Fact]
    public void Test1()
    {
        var counter = 0; // Isolated
        Assert.Equal(0, counter++);
    }
    
    [Fact]
    public void Test2()
    {
        var counter = 0; // Isolated
        Assert.Equal(0, counter++);
    }
}
```

---

### Problem: DbContext Tracking Issues

**Cause:** EF Core caches entities

**Solution:**
```csharp
// ❌ BAD - Same context used
[Fact]
public async Task UpdateUser_BadTest()
{
    var user = await _context.Users.FindAsync(1);
    user.Name = "Updated";
    await _context.SaveChangesAsync();
    
    var updated = await _context.Users.FindAsync(1);
    // _context still has old copy cached!
}

// ✅ GOOD - New context or detach
[Fact]
public async Task UpdateUser_GoodTest()
{
    // Option 1: Use new context
    using (var context1 = CreateContext())
    {
        var user = await context1.Users.FindAsync(1);
        user.Name = "Updated";
        await context1.SaveChangesAsync();
    }
    
    using (var context2 = CreateContext())
    {
        var updated = await context2.Users.FindAsync(1);
        Assert.Equal("Updated", updated.Name);
    }
    
    // Option 2: Detach entity
    // _context.Entry(user).State = EntityState.Detached;
}
```

---

### Problem: Async Tests Don't Wait

**Cause:** Forgot await or Task.Run

**Solution:**
```csharp
// ❌ BAD - Doesn't wait
[Fact]
public void BadAsyncTest()
{
    var task = _service.GetUserAsync(1); // Not awaited!
    Assert.NotNull(task.Result); // Blocks, can deadlock
}

// ✅ GOOD - Proper async
[Fact]
public async Task GoodAsyncTest()
{
    var user = await _service.GetUserAsync(1);
    Assert.NotNull(user);
}
```

---

### Problem: Mock Not Working as Expected

**Cause:** Setup doesn't match actual call

**Solution:**
```csharp
// ❌ BAD - Specific value in setup
[Fact]
public void BadMockTest()
{
    var mock = new Mock<IUserRepository>();
    mock.Setup(m => m.GetByIdAsync(1))  // Only works for ID 1
        .ReturnsAsync(new User());
    
    var user = await mock.Object.GetByIdAsync(2); // null!
}

// ✅ GOOD - Use It.IsAny
[Fact]
public async Task GoodMockTest()
{
    var mock = new Mock<IUserRepository>();
    mock.Setup(m => m.GetByIdAsync(It.IsAny<int>())) // Works for any ID
        .ReturnsAsync((int id) => new User { Id = id });
    
    var user = await mock.Object.GetByIdAsync(2);
    Assert.Equal(2, user.Id);
}
```

---

### Problem: Integration Tests are Slow

**Solutions:**

```csharp
// 1. Use IClassFixture to share setup
public class OptimizedTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    
    public OptimizedTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient(); // Reused across tests
    }
}

// 2. Run tests in parallel (xUnit default)
[assembly: CollectionBehavior(DisableTestParallelization = false)]

// 3. Use in-memory database for most tests
// 4. Mark slow tests
[Trait("Category", "Integration")]
[Trait("Speed", "Slow")]
public class SlowIntegrationTests { }

// Run only fast tests:
// dotnet test --filter "Category!=Integration"
```

---

## 9. Best Practices

### ✅ DO:

1. **Follow AAA Pattern**
   ```csharp
   [Fact]
   public void TestName()
   {
       // Arrange
       var sut = new SystemUnderTest();
       
       // Act
       var result = sut.DoSomething();
       
       // Assert
       Assert.Equal(expected, result);
   }
   ```

2. **Use Descriptive Test Names**
   ```csharp
   // ✅ Good
   [Fact]
   public void GetUser_WhenUserExists_ReturnsUser()
   
   // ❌ Bad
   [Fact]
   public void Test1()
   ```

3. **Test One Thing Per Test**
   ```csharp
   // ✅ Good
   [Fact]
   public void GetUser_ReturnsCorrectName() { }
   
   [Fact]
   public void GetUser_ReturnsCorrectEmail() { }
   
   // ❌ Bad
   [Fact]
   public void GetUser_ReturnsEverything()
   {
       Assert.Equal(name, user.Name);
       Assert.Equal(email, user.Email);
       Assert.Equal(age, user.Age);
       // Testing too much
   }
   ```

4. **Keep Tests Fast**
   - Unit tests < 100ms
   - Integration tests < 1s
   - Use in-memory databases
   - Mock external dependencies

5. **Make Tests Independent**
   - No shared state
   - Can run in any order
   - Can run in parallel

---

### ❌ DON'T:

1. **Don't Test Implementation Details**
   ```csharp
   // ❌ Bad - Testing private method
   [Fact]
   public void TestPrivateMethod()
   {
       var method = typeof(UserService).GetMethod("PrivateHelper", 
           BindingFlags.NonPublic | BindingFlags.Instance);
       // Don't do this!
   }
   
   // ✅ Good - Test public behavior
   [Fact]
   public void CreateUser_CallsValidation()
   {
       // Test the public API that uses the private method
   }
   ```

2. **Don't Use Real External Services**
   ```csharp
   // ❌ Bad
   var emailService = new SmtpEmailService(); // Real SMTP!
   
   // ✅ Good
   var mockEmail = new Mock<IEmailService>();
   ```

3. **Don't Ignore Test Failures**
   - Fix failing tests immediately
   - Don't comment out failing tests
   - Don't skip tests without reason

4. **Don't Copy-Paste Tests**
   - Use Theory for similar tests
   - Extract common setup to helper methods
   - Use test fixtures for shared resources

---

### Test Organization

```
YourProject.Tests/
├── Unit/
│   ├── Services/
│   │   ├── UserServiceTests.cs
│   │   └── OrderServiceTests.cs
│   ├── Repositories/
│   │   └── UserRepositoryTests.cs
│   └── Validators/
│       └── UserValidatorTests.cs
├── Integration/
│   ├── Controllers/
│   │   └── UsersControllerTests.cs
│   ├── WebApplicationFactoryFixture.cs
│   └── DatabaseFixture.cs
├── TestHelpers/
│   ├── TestDataBuilder.cs
│   └── MockHelper.cs
└── XUnit.config.json
```

---

### Code Coverage Guidelines

**Aim for:**
- Critical paths: 100%
- Business logic: 90-100%
- Controllers: 80%+
- Overall: 70-80%+

**Check coverage:**
```bash
dotnet test /p:CollectCoverage=true
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
```

**Generate reports:**
```bash
dotnet add package coverlet.collector
dotnet test --collect:"XPlat Code Coverage"

# Generate HTML report
reportgenerator -reports:**/coverage.cobertura.xml -targetdir:coverage
```

---

# PART 2: TECHNICAL REFERENCE

---

## 10. Important Interfaces & Classes Reference

### xUnit Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[Fact]` | Simple test | `[Fact] public void Test()` |
| `[Theory]` | Data-driven test | `[Theory] [InlineData(1,2)]` |
| `[InlineData]` | Provide test data | `[InlineData(1, 2, 3)]` |
| `[MemberData]` | External test data | `[MemberData(nameof(Data))]` |
| `[ClassData]` | Class-based data | `[ClassData(typeof(TestData))]` |
| `[Trait]` | Categorize tests | `[Trait("Category", "Unit")]` |
| `[Skip]` | Skip test | `[Fact(Skip = "Reason")]` |

---

### Assert Class (xUnit.Assert)

**Namespace:** `Xunit`

```csharp
public static class Assert
{
    // Equality
    public static void Equal<T>(T expected, T actual);
    public static void NotEqual<T>(T expected, T actual);
    public static void Same(object expected, object actual);
    public static void NotSame(object expected, object actual);
    
    // Null checks
    public static void Null(object @object);
    public static void NotNull(object @object);
    
    // Boolean
    public static void True(bool condition);
    public static void False(bool condition);
    
    // Collections
    public static void Empty(IEnumerable collection);
    public static void NotEmpty(IEnumerable collection);
    public static void Contains<T>(T expected, IEnumerable<T> collection);
    public static void DoesNotContain<T>(T expected, IEnumerable<T> collection);
    public static void Single(IEnumerable collection);
    public static void All<T>(IEnumerable<T> collection, Action<T> action);
    
    // Exceptions
    public static T Throws<T>(Action testCode) where T : Exception;
    public static Task<T> ThrowsAsync<T>(Func<Task> testCode) where T : Exception;
    public static void ThrowsAny<T>(Action testCode) where T : Exception;
    
    // Types
    public static void IsType<T>(object @object);
    public static void IsNotType<T>(object @object);
    public static void IsAssignableFrom<T>(object @object);
    
    // Ranges
    public static void InRange<T>(T actual, T low, T high);
    public static void NotInRange<T>(T actual, T low, T high);
    
    // Strings
    public static void StartsWith(string expectedStartString, string actualString);
    public static void EndsWith(string expectedEndString, string actualString);
    public static void Matches(string expectedRegexPattern, string actualString);
}
```

**Usage Examples:**

```csharp
// Equality
Assert.Equal(5, result);
Assert.NotEqual(0, result);
Assert.Same(obj1, obj2); // Reference equality
Assert.NotSame(obj1, obj2);

// Null
Assert.Null(user);
Assert.NotNull(user);

// Boolean
Assert.True(isValid);
Assert.False(hasErrors);

// Collections
Assert.Empty(list);
Assert.NotEmpty(list);
Assert.Contains(item, list);
Assert.DoesNotContain(item, list);
Assert.Single(list); // Exactly one element
Assert.All(users, u => Assert.NotNull(u.Name));

// Collection count
Assert.Equal(3, list.Count);
Assert.Equal(5, list.Count());

// Exceptions
var ex = Assert.Throws<ArgumentException>(() => service.DoWork(null));
Assert.Equal("Parameter cannot be null", ex.Message);

var ex = await Assert.ThrowsAsync<InvalidOperationException>(
    () => service.DoWorkAsync());

// Types
Assert.IsType<User>(result);
Assert.IsNotType<Admin>(result);
Assert.IsAssignableFrom<IUser>(result);

// Ranges
Assert.InRange(age, 18, 100);
Assert.NotInRange(value, -10, 10);

// Strings
Assert.StartsWith("Hello", greeting);
Assert.EndsWith("World", greeting);
Assert.Matches(@"\d{3}-\d{4}", phoneNumber);
```

---

### WebApplicationFactory<TEntryPoint>

**Namespace:** `Microsoft.AspNetCore.Mvc.Testing`

**Purpose:** Create test server for integration tests

```csharp
public class WebApplicationFactory<TEntryPoint> 
    : IAsyncLifetime where TEntryPoint : class
{
    // Create HTTP client
    public HttpClient CreateClient();
    public HttpClient CreateClient(WebApplicationFactoryClientOptions options);
    
    // Get services
    public IServiceProvider Services { get; }
    
    // Override configuration
    protected virtual void ConfigureWebHost(IWebHostBuilder builder);
    
    // Server
    public TestServer Server { get; }
    
    // Lifecycle
    public Task InitializeAsync();
    public Task DisposeAsync();
}
```

**Members Table:**

| Member | Type | Purpose |
|--------|------|---------|
| `CreateClient()` | Method | Create HttpClient for tests |
| `Services` | Property | Access service provider |
| `ConfigureWebHost()` | Method | Override configuration |
| `Server` | Property | Access test server |
| `WithWebHostBuilder()` | Method | Configure builder |

**Complete Example:**

```csharp
public class CustomFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Replace real services
            services.RemoveAll<ApplicationDbContext>();
            services.AddDbContext<ApplicationDbContext>(options =>
                options.UseInMemoryDatabase("Test"));
        });
        
        builder.ConfigureAppConfiguration((context, config) =>
        {
            // Add test configuration
            config.AddInMemoryCollection(new Dictionary<string, string>
            {
                ["TestKey"] = "TestValue"
            });
        });
    }
}
```

---

### Mock<T> Class (Moq)

**Namespace:** `Moq`

**Purpose:** Create mock objects for testing

```csharp
public class Mock<T> where T : class
{
    // Create instance
    public Mock();
    public Mock(MockBehavior behavior);
    
    // Get mocked object
    public T Object { get; }
    
    // Setup methods
    public ISetup<T, TResult> Setup<TResult>(Expression<Func<T, TResult>> expression);
    public ISetup<T> Setup(Expression<Action<T>> expression);
    
    // Setup properties
    public Mock<T> SetupProperty<TProperty>(
        Expression<Func<T, TProperty>> property);
    public Mock<T> SetupAllProperties();
    
    // Verification
    public void Verify<TResult>(
        Expression<Func<T, TResult>> expression,
        Times times);
    public void VerifyAll();
    public void VerifyNoOtherCalls();
    
    // Behavior
    public CallBase { get; set; }
    public DefaultValue DefaultValue { get; set; }
}
```

**Methods Table:**

| Method | Purpose | Example |
|--------|---------|---------|
| `Setup()` | Configure return value | `Setup(m => m.Get()).Returns(value)` |
| `Returns()` | Set return value | `.Returns(value)` |
| `ReturnsAsync()` | Set async return | `.ReturnsAsync(value)` |
| `Throws()` | Throw exception | `.Throws<Exception>()` |
| `Callback()` | Execute code | `.Callback(() => count++)` |
| `Verify()` | Check if called | `Verify(m => m.Get(), Times.Once)` |
| `VerifyAll()` | Verify all setups | Mock calls all configured setups |
| `VerifyNoOtherCalls()` | No unexpected calls | Ensures only verified calls made |

**Verification Times:**

```csharp
Times.Never          // 0 times
Times.Once           // Exactly 1 time
Times.Exactly(n)     // Exactly n times
Times.AtLeastOnce    // 1 or more times
Times.AtLeast(n)     // n or more times
Times.AtMostOnce     // 0 or 1 time
Times.AtMost(n)      // n or fewer times
Times.Between(m, n, Range.Inclusive)  // Between m and n
```

---

### IClassFixture<T> Interface

**Namespace:** `Xunit`

**Purpose:** Share setup/cleanup across tests in a class

```csharp
public interface IClassFixture<TFixture> where TFixture : class
{
    // No members - marker interface
}
```

**Usage Pattern:**

```csharp
// Step 1: Create fixture
public class DatabaseFixture : IDisposable
{
    public ApplicationDbContext Context { get; }
    
    public DatabaseFixture()
    {
        // Setup
        Context = CreateContext();
    }
    
    public void Dispose()
    {
        // Cleanup
        Context?.Dispose();
    }
}

// Step 2: Use fixture
public class MyTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;
    
    public MyTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }
    
    [Fact]
    public void Test1()
    {
        // Use _fixture.Context
    }
}
```

---

### IAsyncLifetime Interface

**Namespace:** `Xunit`

**Purpose:** Async setup/cleanup for test classes

```csharp
public interface IAsyncLifetime
{
    Task InitializeAsync();
    Task DisposeAsync();
}
```

**Usage:**

```csharp
public class AsyncTests : IAsyncLifetime
{
    private ApplicationDbContext _context;
    
    public async Task InitializeAsync()
    {
        _context = CreateContext();
        await _context.Database.MigrateAsync();
    }
    
    public async Task DisposeAsync()
    {
        if (_context != null)
        {
            await _context.Database.EnsureDeletedAsync();
            await _context.DisposeAsync();
        }
    }
    
    [Fact]
    public async Task TestExample()
    {
        // Use _context
    }
}
```

---

## 11. Configuration Deep-Dive

### Pattern 1: Inline Configuration (Simple Tests)

**When to use:** Quick tests, prototypes

```csharp
[Fact]
public void InlineTest()
{
    // Arrange - Everything inline
    var options = new DbContextOptionsBuilder<ApplicationDbContext>()
        .UseInMemoryDatabase("TestDb")
        .Options;
    
    using var context = new ApplicationDbContext(options);
    var repository = new UserRepository(context);
    
    // Act & Assert
    var user = repository.GetById(1);
    Assert.Null(user);
}
```

**Pros:**
- ✅ Simple and direct
- ✅ Good for single tests
- ✅ No configuration files

**Cons:**
- ❌ Repeated code
- ❌ Hard to maintain
- ❌ Can't reuse configuration

---

### Pattern 2: Base Class Setup (Reusable)

**When to use:** Multiple test classes need same setup

```csharp
public class TestBase : IDisposable
{
    protected ApplicationDbContext Context { get; }
    protected IMapper Mapper { get; }
    
    protected TestBase()
    {
        // Configure DbContext
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(Guid.NewGuid().ToString())
            .Options;
        Context = new ApplicationDbContext(options);
        
        // Configure AutoMapper
        var config = new MapperConfiguration(cfg =>
        {
            cfg.AddProfile<MappingProfile>();
        });
        Mapper = config.CreateMapper();
    }
    
    public void Dispose()
    {
        Context?.Dispose();
    }
}

// Use in tests
public class UserServiceTests : TestBase
{
    [Fact]
    public async Task GetUser_ReturnsUser()
    {
        // Context and Mapper available from base class
        var service = new UserService(Context, Mapper);
        var user = await service.GetUserAsync(1);
        Assert.NotNull(user);
    }
}
```

---

### Pattern 3: Test Configuration Files (Production-Like)

**When to use:** Complex configuration, environment-specific settings

**Step 1: appsettings.test.json**

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=TestDb;User=test;Password=test"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft": "Warning"
    }
  },
  "Email": {
    "Provider": "Fake",
    "From": "test@test.com"
  }
}
```

**Step 2: Load Configuration in Tests**

```csharp
public class ConfigurationTestBase
{
    protected IConfiguration Configuration { get; }
    
    protected ConfigurationTestBase()
    {
        Configuration = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.test.json", optional: false)
            .AddEnvironmentVariables()
            .Build();
    }
}

public class MyIntegrationTests : ConfigurationTestBase
{
    [Fact]
    public void TestWithConfiguration()
    {
        var connectionString = Configuration.GetConnectionString("Default");
        // Use configuration...
    }
}
```

---

### Configuration Comparison

| Pattern | Complexity | Reusability | Maintenance | Production-Like |
|---------|-----------|-------------|-------------|-----------------|
| Inline | Simple | Low | Hard | ❌ No |
| Base Class | Medium | High | Easy | ⚠️ Partial |
| Config Files | Complex | High | Easy | ✅ Yes |

---

## 12. Advanced Testing Topics

### Testing Middleware

```csharp
public class MiddlewareTests
{
    [Fact]
    public async Task Middleware_ModifiesRequest()
    {
        // Arrange
        var middleware = new RequestLoggingMiddleware(
            next: (innerHttpContext) => Task.CompletedTask,
            logger: Mock.Of<ILogger<RequestLoggingMiddleware>>());
        
        var context = new DefaultHttpContext();
        context.Request.Path = "/test";
        
        // Act
        await middleware.InvokeAsync(context);
        
        // Assert
        Assert.Contains("X-Request-Id", context.Response.Headers.Keys);
    }
    
    [Fact]
    public async Task Middleware_CallsNext()
    {
        // Arrange
        var nextCalled = false;
        var middleware = new MyMiddleware(
            next: (innerHttpContext) => 
            {
                nextCalled = true;
                return Task.CompletedTask;
            });
        
        var context = new DefaultHttpContext();
        
        // Act
        await middleware.InvokeAsync(context);
        
        // Assert
        Assert.True(nextCalled);
    }
}
```

---

### Testing Background Services

```csharp
public class BackgroundServiceTests
{
    [Fact]
    public async Task BackgroundService_ProcessesItems()
    {
        // Arrange
        var mockService = new Mock<IDataService>();
        var service = new MyBackgroundService(mockService.Object);
        var cts = new CancellationTokenSource();
        
        // Act
        var task = service.StartAsync(cts.Token);
        await Task.Delay(1000); // Let it run
        cts.Cancel();
        await task;
        
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
        await hub.SendMessage("Test User", "Hello");
        
        // Assert
        mockClientProxy.Verify(
            c => c.SendCoreAsync(
                "ReceiveMessage",
                It.Is<object[]>(o => o.Length == 2),
                default),
            Times.Once);
    }
}
```

---

### Testing with Time/DateTime

```csharp
// Create abstraction
public interface IDateTimeProvider
{
    DateTime UtcNow { get; }
}

public class SystemDateTimeProvider : IDateTimeProvider
{
    public DateTime UtcNow => DateTime.UtcNow;
}

// Use in service
public class OrderService
{
    private readonly IDateTimeProvider _dateTime;
    
    public OrderService(IDateTimeProvider dateTime)
    {
        _dateTime = dateTime;
    }
    
    public Order CreateOrder()
    {
        return new Order
        {
            CreatedAt = _dateTime.UtcNow
        };
    }
}

// Test with fixed time
[Fact]
public void CreateOrder_SetsCorrectCreatedDate()
{
    // Arrange
    var fixedTime = new DateTime(2024, 1, 1, 12, 0, 0);
    var mockDateTime = new Mock<IDateTimeProvider>();
    mockDateTime.Setup(d => d.UtcNow).Returns(fixedTime);
    
    var service = new OrderService(mockDateTime.Object);
    
    // Act
    var order = service.CreateOrder();
    
    // Assert
    Assert.Equal(fixedTime, order.CreatedAt);
}
```

---

### Snapshot Testing

**Install:**
```bash
dotnet add package Verify.Xunit
```

**Usage:**

```csharp
using VerifyXunit;

public class SnapshotTests
{
    [Fact]
    public Task GeneratedCode_MatchesSnapshot()
    {
        var generator = new CodeGenerator();
        var code = generator.Generate();
        
        return Verifier.Verify(code);
    }
    
    [Fact]
    public Task ApiResponse_MatchesSnapshot()
    {
        var response = new UserResponse
        {
            Id = 1,
            Name = "John",
            Email = "john@test.com"
        };
        
        return Verifier.Verify(response);
    }
}
```

---

## 13. Performance Testing Basics

### BenchmarkDotNet Setup

```bash
dotnet add package BenchmarkDotNet
```

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
public class PerformanceBenchmarks
{
    private List<User> _users;
    
    [GlobalSetup]
    public void Setup()
    {
        _users = Enumerable.Range(1, 1000)
            .Select(i => new User { Id = i, Name = $"User {i}" })
            .ToList();
    }
    
    [Benchmark]
    public List<User> GetActiveUsers_Linq()
    {
        return _users.Where(u => u.IsActive).ToList();
    }
    
    [Benchmark]
    public List<User> GetActiveUsers_ForLoop()
    {
        var result = new List<User>();
        for (int i = 0; i < _users.Count; i++)
        {
            if (_users[i].IsActive)
                result.Add(_users[i]);
        }
        return result;
    }
}

// Run benchmarks
public class Program
{
    public static void Main(string[] args)
    {
        BenchmarkRunner.Run<PerformanceBenchmarks>();
    }
}
```

---

### Load Testing with NBomber

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
        .WithWarmUpDuration(TimeSpan.FromSeconds(5))
        .WithLoadSimulations(
            Simulation.InjectPerSec(rate: 100, during: TimeSpan.FromSeconds(30))
        );
        
        var stats = NBomberRunner
            .RegisterScenarios(scenario)
            .Run();
        
        // Assert performance requirements
        var okCount = stats.ScenarioStats[0].Ok.Request.Count;
        Assert.True(okCount > 2500, $"Expected > 2500 successful requests, got {okCount}");
    }
}
```

---

## Summary: Complete Testing Checklist

**Setup:**
- [ ] Install xUnit, Moq, WebApplicationFactory
- [ ] Create test project structure
- [ ] Configure test settings

**Unit Tests:**
- [ ] Test business logic
- [ ] Use AAA pattern
- [ ] Mock dependencies
- [ ] Test edge cases
- [ ] Test exceptions

**Integration Tests:**
- [ ] Create WebApplicationFactory
- [ ] Replace services with test doubles
- [ ] Use test database
- [ ] Test complete workflows
- [ ] Test authentication/authorization

**Test Quality:**
- [ ] Tests are fast (< 1s)
- [ ] Tests are independent
- [ ] Tests have clear names
- [ ] One assertion per test (when possible)
- [ ] Good code coverage (70%+)

**Best Practices:**
- [ ] Follow naming conventions
- [ ] Keep tests maintainable
- [ ] Use Theory for data-driven tests
- [ ] Clean up resources (IDisposable)
- [ ] Run tests in CI/CD

---

**This completes the Testing guide combining practical hands-on content with deep technical reference!**
