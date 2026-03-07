# Clean Architecture Quick Reference

---

## What is Clean Architecture?

**Clean Architecture** = Architecture pattern that separates concerns into layers with clear dependency rules

**Created by:** Robert C. Martin (Uncle Bob)

**Key Principles:**
- ✅ **Independence** - Framework, UI, database independent
- ✅ **Testability** - Business logic easily testable
- ✅ **Maintainability** - Easy to change and extend
- ✅ **Separation of Concerns** - Each layer has specific responsibility
- ✅ **Dependency Rule** - Dependencies point inward only

### The Dependency Rule

**CRITICAL:** Source code dependencies can only point INWARD

```
┌─────────────────────────────────────┐
│     External (UI, Database, etc)    │  ← Outer Layer
├─────────────────────────────────────┤
│     Infrastructure & Frameworks     │
├─────────────────────────────────────┤
│     Application (Use Cases)         │
├─────────────────────────────────────┤
│     Domain (Entities, Business)     │  ← Inner Layer (Core)
└─────────────────────────────────────┘

Dependencies flow: Outer → Inner (never Inner → Outer)
```

---

## The Four Layers

### 1. Domain Layer (Core / Entities)

**Location:** Center of architecture
**Dependencies:** None (completely independent)
**Contains:**
- Entities (business objects)
- Value Objects
- Domain Events
- Enums
- Exceptions
- Interfaces (repository contracts)

**Rules:**
- ❌ No dependencies on other layers
- ❌ No framework dependencies
- ❌ No database concerns
- ✅ Pure business logic only

```csharp
// Domain/Entities/Customer.cs
namespace Domain.Entities
{
    public class Customer
    {
        public int Id { get; private set; }
        public string FirstName { get; private set; }
        public string LastName { get; private set; }
        public string Email { get; private set; }
        public CustomerStatus Status { get; private set; }
        public DateTime CreatedAt { get; private set; }
        
        // Business logic in domain
        public void Activate()
        {
            if (Status == CustomerStatus.Active)
                throw new InvalidOperationException("Customer is already active");
            
            Status = CustomerStatus.Active;
        }
        
        public void Deactivate()
        {
            if (Status == CustomerStatus.Inactive)
                throw new InvalidOperationException("Customer is already inactive");
            
            Status = CustomerStatus.Inactive;
        }
        
        public void UpdateEmail(string newEmail)
        {
            if (string.IsNullOrWhiteSpace(newEmail))
                throw new ArgumentException("Email cannot be empty");
            
            if (!newEmail.Contains("@"))
                throw new ArgumentException("Invalid email format");
            
            Email = newEmail;
        }
    }
    
    public enum CustomerStatus
    {
        Active,
        Inactive,
        Suspended
    }
}

// Domain/Interfaces/ICustomerRepository.cs
namespace Domain.Interfaces
{
    public interface ICustomerRepository
    {
        Task<Customer?> GetByIdAsync(int id);
        Task<IEnumerable<Customer>> GetAllAsync();
        Task<int> AddAsync(Customer customer);
        Task UpdateAsync(Customer customer);
        Task DeleteAsync(int id);
    }
}
```

### 2. Application Layer (Use Cases)

**Location:** Second layer from center
**Dependencies:** Domain layer only
**Contains:**
- Use Cases / Application Services
- DTOs (Data Transfer Objects)
- Mapping Profiles
- Validators
- Interfaces for infrastructure

**Rules:**
- ✅ Can depend on Domain
- ❌ Cannot depend on Infrastructure or Presentation
- ✅ Orchestrates business logic
- ✅ Defines interfaces for external services

```csharp
// Application/DTOs/CustomerDto.cs
namespace Application.DTOs
{
    public record CustomerDto(
        int Id,
        string FirstName,
        string LastName,
        string Email,
        string Status);
    
    public record CreateCustomerDto(
        string FirstName,
        string LastName,
        string Email);
    
    public record UpdateCustomerDto(
        int Id,
        string FirstName,
        string LastName,
        string Email);
}

// Application/Interfaces/IEmailService.cs
namespace Application.Interfaces
{
    public interface IEmailService
    {
        Task SendWelcomeEmailAsync(string email, string name);
        Task SendNotificationAsync(string email, string message);
    }
}

// Application/Services/CustomerService.cs
namespace Application.Services
{
    public class CustomerService
    {
        private readonly ICustomerRepository _customerRepository;
        private readonly IEmailService _emailService;
        
        public CustomerService(
            ICustomerRepository customerRepository,
            IEmailService emailService)
        {
            _customerRepository = customerRepository;
            _emailService = emailService;
        }
        
        public async Task<CustomerDto?> GetCustomerAsync(int id)
        {
            var customer = await _customerRepository.GetByIdAsync(id);
            
            if (customer == null)
                return null;
            
            return new CustomerDto(
                customer.Id,
                customer.FirstName,
                customer.LastName,
                customer.Email,
                customer.Status.ToString());
        }
        
        public async Task<int> CreateCustomerAsync(CreateCustomerDto dto)
        {
            // Domain entity creation
            var customer = new Customer
            {
                FirstName = dto.FirstName,
                LastName = dto.LastName,
                Email = dto.Email
            };
            
            // Persist
            var id = await _customerRepository.AddAsync(customer);
            
            // Send welcome email (infrastructure concern)
            await _emailService.SendWelcomeEmailAsync(
                customer.Email,
                customer.FirstName);
            
            return id;
        }
        
        public async Task UpdateCustomerAsync(UpdateCustomerDto dto)
        {
            var customer = await _customerRepository.GetByIdAsync(dto.Id);
            
            if (customer == null)
                throw new NotFoundException($"Customer {dto.Id} not found");
            
            // Use domain logic
            customer.UpdateEmail(dto.Email);
            
            await _customerRepository.UpdateAsync(customer);
        }
    }
}
```

### 3. Infrastructure Layer

**Location:** Third layer
**Dependencies:** Domain and Application layers
**Contains:**
- Repository implementations
- Database context (EF Core)
- External service implementations
- File system access
- API clients

**Rules:**
- ✅ Can depend on Domain and Application
- ✅ Implements interfaces from Domain/Application
- ❌ Should not contain business logic

```csharp
// Infrastructure/Data/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;
using Domain.Entities;

namespace Infrastructure.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }
        
        public DbSet<Customer> Customers => Set<Customer>();
        
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Customer>(entity =>
            {
                entity.HasKey(e => e.Id);
                entity.Property(e => e.FirstName).IsRequired().HasMaxLength(50);
                entity.Property(e => e.LastName).IsRequired().HasMaxLength(50);
                entity.Property(e => e.Email).IsRequired().HasMaxLength(100);
                entity.HasIndex(e => e.Email).IsUnique();
            });
        }
    }
}

// Infrastructure/Repositories/CustomerRepository.cs
using Dapper;
using Domain.Entities;
using Domain.Interfaces;
using Microsoft.Data.SqlClient;

namespace Infrastructure.Repositories
{
    public class CustomerRepository : ICustomerRepository
    {
        private readonly string _connectionString;
        
        public CustomerRepository(string connectionString)
        {
            _connectionString = connectionString;
        }
        
        public async Task<Customer?> GetByIdAsync(int id)
        {
            using var connection = new SqlConnection(_connectionString);
            return await connection.QueryFirstOrDefaultAsync<Customer>(
                "SELECT * FROM Customers WHERE Id = @Id",
                new { Id = id });
        }
        
        public async Task<IEnumerable<Customer>> GetAllAsync()
        {
            using var connection = new SqlConnection(_connectionString);
            return await connection.QueryAsync<Customer>(
                "SELECT * FROM Customers");
        }
        
        public async Task<int> AddAsync(Customer customer)
        {
            using var connection = new SqlConnection(_connectionString);
            return await connection.ExecuteScalarAsync<int>(@"
                INSERT INTO Customers (FirstName, LastName, Email, Status, CreatedAt)
                VALUES (@FirstName, @LastName, @Email, @Status, @CreatedAt);
                SELECT CAST(SCOPE_IDENTITY() AS INT);",
                customer);
        }
        
        public async Task UpdateAsync(Customer customer)
        {
            using var connection = new SqlConnection(_connectionString);
            await connection.ExecuteAsync(@"
                UPDATE Customers
                SET FirstName = @FirstName,
                    LastName = @LastName,
                    Email = @Email,
                    Status = @Status
                WHERE Id = @Id",
                customer);
        }
        
        public async Task DeleteAsync(int id)
        {
            using var connection = new SqlConnection(_connectionString);
            await connection.ExecuteAsync(
                "DELETE FROM Customers WHERE Id = @Id",
                new { Id = id });
        }
    }
}

// Infrastructure/Services/EmailService.cs
using Application.Interfaces;

namespace Infrastructure.Services
{
    public class EmailService : IEmailService
    {
        public async Task SendWelcomeEmailAsync(string email, string name)
        {
            // Implementation using SMTP, SendGrid, etc.
            Console.WriteLine($"Sending welcome email to {email}");
            await Task.CompletedTask;
        }
        
        public async Task SendNotificationAsync(string email, string message)
        {
            Console.WriteLine($"Sending notification to {email}: {message}");
            await Task.CompletedTask;
        }
    }
}
```

### 4. Presentation Layer (UI / API)

**Location:** Outermost layer
**Dependencies:** Application and Infrastructure (for DI setup)
**Contains:**
- Controllers (API)
- Views (MVC)
- ViewModels
- API models
- Startup/Program.cs

**Rules:**
- ✅ Can depend on Application
- ✅ Configures DI container
- ❌ Should not contain business logic
- ❌ Should not directly access Domain

```csharp
// API/Controllers/CustomersController.cs
using Application.DTOs;
using Application.Services;
using Microsoft.AspNetCore.Mvc;

namespace API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class CustomersController : ControllerBase
    {
        private readonly CustomerService _customerService;
        
        public CustomersController(CustomerService customerService)
        {
            _customerService = customerService;
        }
        
        [HttpGet("{id}")]
        public async Task<ActionResult<CustomerDto>> GetCustomer(int id)
        {
            var customer = await _customerService.GetCustomerAsync(id);
            
            if (customer == null)
                return NotFound();
            
            return Ok(customer);
        }
        
        [HttpPost]
        public async Task<ActionResult<int>> CreateCustomer(CreateCustomerDto dto)
        {
            var id = await _customerService.CreateCustomerAsync(dto);
            return CreatedAtAction(nameof(GetCustomer), new { id }, id);
        }
        
        [HttpPut("{id}")]
        public async Task<IActionResult> UpdateCustomer(int id, UpdateCustomerDto dto)
        {
            if (id != dto.Id)
                return BadRequest();
            
            await _customerService.UpdateCustomerAsync(dto);
            return NoContent();
        }
    }
}

// API/Program.cs
using Application.Interfaces;
using Application.Services;
using Domain.Interfaces;
using Infrastructure.Data;
using Infrastructure.Repositories;
using Infrastructure.Services;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Database
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Dependency Injection
// Domain/Application → Infrastructure mapping
builder.Services.AddScoped<ICustomerRepository, CustomerRepository>();
builder.Services.AddScoped<IEmailService, EmailService>();

// Application Services
builder.Services.AddScoped<CustomerService>();

var app = builder.Build();

// Configure pipeline
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

---

## Project Structure

### Recommended Folder Structure

```
Solution/
├── src/
│   ├── Domain/                          # Core business logic
│   │   ├── Entities/
│   │   │   ├── Customer.cs
│   │   │   ├── Order.cs
│   │   │   └── Product.cs
│   │   ├── ValueObjects/
│   │   │   ├── Address.cs
│   │   │   └── Money.cs
│   │   ├── Enums/
│   │   │   └── OrderStatus.cs
│   │   ├── Interfaces/
│   │   │   ├── ICustomerRepository.cs
│   │   │   └── IOrderRepository.cs
│   │   ├── Exceptions/
│   │   │   ├── DomainException.cs
│   │   │   └── NotFoundException.cs
│   │   └── Events/
│   │       └── CustomerCreatedEvent.cs
│   │
│   ├── Application/                     # Use cases
│   │   ├── DTOs/
│   │   │   ├── CustomerDto.cs
│   │   │   └── OrderDto.cs
│   │   ├── Interfaces/
│   │   │   ├── IEmailService.cs
│   │   │   └── INotificationService.cs
│   │   ├── Services/
│   │   │   ├── CustomerService.cs
│   │   │   └── OrderService.cs
│   │   ├── Validators/
│   │   │   └── CreateCustomerValidator.cs
│   │   └── Mappings/
│   │       └── MappingProfile.cs
│   │
│   ├── Infrastructure/                  # External concerns
│   │   ├── Data/
│   │   │   ├── ApplicationDbContext.cs
│   │   │   └── Migrations/
│   │   ├── Repositories/
│   │   │   ├── CustomerRepository.cs
│   │   │   └── OrderRepository.cs
│   │   ├── Services/
│   │   │   ├── EmailService.cs
│   │   │   └── BlobStorageService.cs
│   │   └── Configuration/
│   │       └── DependencyInjection.cs
│   │
│   └── API/                             # Presentation
│       ├── Controllers/
│       │   ├── CustomersController.cs
│       │   └── OrdersController.cs
│       ├── Middleware/
│       │   └── ExceptionHandlingMiddleware.cs
│       ├── Filters/
│       │   └── ValidationFilter.cs
│       └── Program.cs
│
└── tests/
    ├── Domain.Tests/
    ├── Application.Tests/
    ├── Infrastructure.Tests/
    └── API.Tests/
```

---

## CQRS Pattern in Clean Architecture

**CQRS** = Command Query Responsibility Segregation

**Concept:** Separate read operations from write operations

### Commands (Write Operations)

```csharp
// Application/Commands/CreateCustomerCommand.cs
namespace Application.Commands
{
    public record CreateCustomerCommand(
        string FirstName,
        string LastName,
        string Email);
    
    public class CreateCustomerCommandHandler
    {
        private readonly ICustomerRepository _repository;
        private readonly IEmailService _emailService;
        
        public CreateCustomerCommandHandler(
            ICustomerRepository repository,
            IEmailService emailService)
        {
            _repository = repository;
            _emailService = emailService;
        }
        
        public async Task<int> HandleAsync(CreateCustomerCommand command)
        {
            // Create domain entity
            var customer = new Customer
            {
                FirstName = command.FirstName,
                LastName = command.LastName,
                Email = command.Email,
                Status = CustomerStatus.Active,
                CreatedAt = DateTime.UtcNow
            };
            
            // Persist
            var id = await _repository.AddAsync(customer);
            
            // Send email
            await _emailService.SendWelcomeEmailAsync(
                customer.Email,
                customer.FirstName);
            
            return id;
        }
    }
}

// Application/Commands/UpdateCustomerCommand.cs
namespace Application.Commands
{
    public record UpdateCustomerCommand(
        int Id,
        string FirstName,
        string LastName,
        string Email);
    
    public class UpdateCustomerCommandHandler
    {
        private readonly ICustomerRepository _repository;
        
        public UpdateCustomerCommandHandler(ICustomerRepository repository)
        {
            _repository = repository;
        }
        
        public async Task HandleAsync(UpdateCustomerCommand command)
        {
            var customer = await _repository.GetByIdAsync(command.Id);
            
            if (customer == null)
                throw new NotFoundException($"Customer {command.Id} not found");
            
            // Update using domain logic
            customer.FirstName = command.FirstName;
            customer.LastName = command.LastName;
            customer.UpdateEmail(command.Email);
            
            await _repository.UpdateAsync(customer);
        }
    }
}
```

### Queries (Read Operations)

```csharp
// Application/Queries/GetCustomerQuery.cs
namespace Application.Queries
{
    public record GetCustomerQuery(int Id);
    
    public class GetCustomerQueryHandler
    {
        private readonly ICustomerRepository _repository;
        
        public GetCustomerQueryHandler(ICustomerRepository repository)
        {
            _repository = repository;
        }
        
        public async Task<CustomerDto?> HandleAsync(GetCustomerQuery query)
        {
            var customer = await _repository.GetByIdAsync(query.Id);
            
            if (customer == null)
                return null;
            
            return new CustomerDto(
                customer.Id,
                customer.FirstName,
                customer.LastName,
                customer.Email,
                customer.Status.ToString());
        }
    }
}

// Application/Queries/GetCustomersQuery.cs
namespace Application.Queries
{
    public record GetCustomersQuery(
        string? SearchTerm = null,
        int PageNumber = 1,
        int PageSize = 10);
    
    public class GetCustomersQueryHandler
    {
        private readonly ICustomerRepository _repository;
        
        public GetCustomersQueryHandler(ICustomerRepository repository)
        {
            _repository = repository;
        }
        
        public async Task<IEnumerable<CustomerDto>> HandleAsync(GetCustomersQuery query)
        {
            var customers = await _repository.GetAllAsync();
            
            // Apply filters
            if (!string.IsNullOrEmpty(query.SearchTerm))
            {
                customers = customers.Where(c =>
                    c.FirstName.Contains(query.SearchTerm, StringComparison.OrdinalIgnoreCase) ||
                    c.LastName.Contains(query.SearchTerm, StringComparison.OrdinalIgnoreCase) ||
                    c.Email.Contains(query.SearchTerm, StringComparison.OrdinalIgnoreCase));
            }
            
            // Pagination
            customers = customers
                .Skip((query.PageNumber - 1) * query.PageSize)
                .Take(query.PageSize);
            
            return customers.Select(c => new CustomerDto(
                c.Id,
                c.FirstName,
                c.LastName,
                c.Email,
                c.Status.ToString()));
        }
    }
}
```

### Controller with CQRS

```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    private readonly CreateCustomerCommandHandler _createHandler;
    private readonly UpdateCustomerCommandHandler _updateHandler;
    private readonly GetCustomerQueryHandler _getHandler;
    private readonly GetCustomersQueryHandler _getAllHandler;
    
    public CustomersController(
        CreateCustomerCommandHandler createHandler,
        UpdateCustomerCommandHandler updateHandler,
        GetCustomerQueryHandler getHandler,
        GetCustomersQueryHandler getAllHandler)
    {
        _createHandler = createHandler;
        _updateHandler = updateHandler;
        _getHandler = getHandler;
        _getAllHandler = getAllHandler;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<CustomerDto>> GetCustomer(int id)
    {
        var query = new GetCustomerQuery(id);
        var customer = await _getHandler.HandleAsync(query);
        
        if (customer == null)
            return NotFound();
        
        return Ok(customer);
    }
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<CustomerDto>>> GetCustomers(
        [FromQuery] string? searchTerm,
        [FromQuery] int pageNumber = 1,
        [FromQuery] int pageSize = 10)
    {
        var query = new GetCustomersQuery(searchTerm, pageNumber, pageSize);
        var customers = await _getAllHandler.HandleAsync(query);
        return Ok(customers);
    }
    
    [HttpPost]
    public async Task<ActionResult<int>> CreateCustomer(CreateCustomerCommand command)
    {
        var id = await _createHandler.HandleAsync(command);
        return CreatedAtAction(nameof(GetCustomer), new { id }, id);
    }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateCustomer(int id, UpdateCustomerCommand command)
    {
        if (id != command.Id)
            return BadRequest();
        
        await _updateHandler.HandleAsync(command);
        return NoContent();
    }
}
```

---

## MediatR Integration

**MediatR** = Library for implementing CQRS and Mediator pattern

```bash
dotnet add package MediatR
```

### Commands with MediatR

```csharp
// Application/Commands/CreateCustomerCommand.cs
using MediatR;

namespace Application.Commands
{
    public record CreateCustomerCommand(
        string FirstName,
        string LastName,
        string Email) : IRequest<int>;
    
    public class CreateCustomerCommandHandler : IRequestHandler<CreateCustomerCommand, int>
    {
        private readonly ICustomerRepository _repository;
        private readonly IEmailService _emailService;
        
        public CreateCustomerCommandHandler(
            ICustomerRepository repository,
            IEmailService emailService)
        {
            _repository = repository;
            _emailService = emailService;
        }
        
        public async Task<int> Handle(CreateCustomerCommand request, CancellationToken cancellationToken)
        {
            var customer = new Customer
            {
                FirstName = request.FirstName,
                LastName = request.LastName,
                Email = request.Email,
                Status = CustomerStatus.Active,
                CreatedAt = DateTime.UtcNow
            };
            
            var id = await _repository.AddAsync(customer);
            
            await _emailService.SendWelcomeEmailAsync(
                customer.Email,
                customer.FirstName);
            
            return id;
        }
    }
}
```

### Queries with MediatR

```csharp
// Application/Queries/GetCustomerQuery.cs
using MediatR;

namespace Application.Queries
{
    public record GetCustomerQuery(int Id) : IRequest<CustomerDto?>;
    
    public class GetCustomerQueryHandler : IRequestHandler<GetCustomerQuery, CustomerDto?>
    {
        private readonly ICustomerRepository _repository;
        
        public GetCustomerQueryHandler(ICustomerRepository repository)
        {
            _repository = repository;
        }
        
        public async Task<CustomerDto?> Handle(GetCustomerQuery request, CancellationToken cancellationToken)
        {
            var customer = await _repository.GetByIdAsync(request.Id);
            
            if (customer == null)
                return null;
            
            return new CustomerDto(
                customer.Id,
                customer.FirstName,
                customer.LastName,
                customer.Email,
                customer.Status.ToString());
        }
    }
}
```

### Controller with MediatR

```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public CustomersController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<CustomerDto>> GetCustomer(int id)
    {
        var customer = await _mediator.Send(new GetCustomerQuery(id));
        
        if (customer == null)
            return NotFound();
        
        return Ok(customer);
    }
    
    [HttpPost]
    public async Task<ActionResult<int>> CreateCustomer(CreateCustomerCommand command)
    {
        var id = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetCustomer), new { id }, id);
    }
}

// Program.cs - Register MediatR
builder.Services.AddMediatR(cfg => 
    cfg.RegisterServicesFromAssembly(typeof(CreateCustomerCommand).Assembly));
```

---

## Validation with FluentValidation

```bash
dotnet add package FluentValidation
dotnet add package FluentValidation.DependencyInjectionExtensions
```

### Validator

```csharp
// Application/Validators/CreateCustomerCommandValidator.cs
using FluentValidation;

namespace Application.Validators
{
    public class CreateCustomerCommandValidator : AbstractValidator<CreateCustomerCommand>
    {
        public CreateCustomerCommandValidator()
        {
            RuleFor(x => x.FirstName)
                .NotEmpty().WithMessage("First name is required")
                .MaximumLength(50).WithMessage("First name must not exceed 50 characters");
            
            RuleFor(x => x.LastName)
                .NotEmpty().WithMessage("Last name is required")
                .MaximumLength(50).WithMessage("Last name must not exceed 50 characters");
            
            RuleFor(x => x.Email)
                .NotEmpty().WithMessage("Email is required")
                .EmailAddress().WithMessage("Invalid email format")
                .MaximumLength(100).WithMessage("Email must not exceed 100 characters");
        }
    }
}
```

### MediatR Pipeline Behavior for Validation

```csharp
// Application/Behaviors/ValidationBehavior.cs
using FluentValidation;
using MediatR;

namespace Application.Behaviors
{
    public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
        where TRequest : IRequest<TResponse>
    {
        private readonly IEnumerable<IValidator<TRequest>> _validators;
        
        public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
        {
            _validators = validators;
        }
        
        public async Task<TResponse> Handle(
            TRequest request,
            RequestHandlerDelegate<TResponse> next,
            CancellationToken cancellationToken)
        {
            if (_validators.Any())
            {
                var context = new ValidationContext<TRequest>(request);
                
                var validationResults = await Task.WhenAll(
                    _validators.Select(v => v.ValidateAsync(context, cancellationToken)));
                
                var failures = validationResults
                    .SelectMany(r => r.Errors)
                    .Where(f => f != null)
                    .ToList();
                
                if (failures.Count != 0)
                    throw new ValidationException(failures);
            }
            
            return await next();
        }
    }
}

// Program.cs - Register validation
builder.Services.AddValidatorsFromAssembly(typeof(CreateCustomerCommandValidator).Assembly);
builder.Services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
```

---

## Repository Pattern

### Generic Repository

```csharp
// Domain/Interfaces/IRepository.cs
namespace Domain.Interfaces
{
    public interface IRepository<T> where T : class
    {
        Task<T?> GetByIdAsync(int id);
        Task<IEnumerable<T>> GetAllAsync();
        Task<int> AddAsync(T entity);
        Task UpdateAsync(T entity);
        Task DeleteAsync(int id);
    }
}

// Infrastructure/Repositories/Repository.cs
using Dapper;
using Domain.Interfaces;
using Microsoft.Data.SqlClient;

namespace Infrastructure.Repositories
{
    public class Repository<T> : IRepository<T> where T : class
    {
        protected readonly string _connectionString;
        protected readonly string _tableName;
        
        public Repository(string connectionString, string tableName)
        {
            _connectionString = connectionString;
            _tableName = tableName;
        }
        
        public virtual async Task<T?> GetByIdAsync(int id)
        {
            using var connection = new SqlConnection(_connectionString);
            return await connection.QueryFirstOrDefaultAsync<T>(
                $"SELECT * FROM {_tableName} WHERE Id = @Id",
                new { Id = id });
        }
        
        public virtual async Task<IEnumerable<T>> GetAllAsync()
        {
            using var connection = new SqlConnection(_connectionString);
            return await connection.QueryAsync<T>($"SELECT * FROM {_tableName}");
        }
        
        public virtual async Task<int> AddAsync(T entity)
        {
            using var connection = new SqlConnection(_connectionString);
            // Simplified - in real app, generate INSERT dynamically
            return await connection.ExecuteScalarAsync<int>(
                $"INSERT INTO {_tableName} VALUES (...); SELECT SCOPE_IDENTITY();");
        }
        
        public virtual async Task UpdateAsync(T entity)
        {
            using var connection = new SqlConnection(_connectionString);
            // Simplified - in real app, generate UPDATE dynamically
            await connection.ExecuteAsync($"UPDATE {_tableName} SET ...");
        }
        
        public virtual async Task DeleteAsync(int id)
        {
            using var connection = new SqlConnection(_connectionString);
            await connection.ExecuteAsync(
                $"DELETE FROM {_tableName} WHERE Id = @Id",
                new { Id = id });
        }
    }
}

// Specific repository
public class CustomerRepository : Repository<Customer>, ICustomerRepository
{
    public CustomerRepository(string connectionString)
        : base(connectionString, "Customers")
    {
    }
    
    // Add custom methods specific to Customer
    public async Task<IEnumerable<Customer>> GetActiveCustomersAsync()
    {
        using var connection = new SqlConnection(_connectionString);
        return await connection.QueryAsync<Customer>(
            "SELECT * FROM Customers WHERE Status = @Status",
            new { Status = CustomerStatus.Active });
    }
}
```

---

## Unit of Work Pattern

```csharp
// Application/Interfaces/IUnitOfWork.cs
namespace Application.Interfaces
{
    public interface IUnitOfWork : IDisposable
    {
        ICustomerRepository Customers { get; }
        IOrderRepository Orders { get; }
        IProductRepository Products { get; }
        
        Task<int> SaveChangesAsync();
        Task BeginTransactionAsync();
        Task CommitTransactionAsync();
        Task RollbackTransactionAsync();
    }
}

// Infrastructure/Data/UnitOfWork.cs
using Application.Interfaces;
using Domain.Interfaces;
using Infrastructure.Repositories;
using Microsoft.EntityFrameworkCore;

namespace Infrastructure.Data
{
    public class UnitOfWork : IUnitOfWork
    {
        private readonly ApplicationDbContext _context;
        private ICustomerRepository? _customers;
        private IOrderRepository? _orders;
        private IProductRepository? _products;
        
        public UnitOfWork(ApplicationDbContext context)
        {
            _context = context;
        }
        
        public ICustomerRepository Customers =>
            _customers ??= new CustomerRepository(_context);
        
        public IOrderRepository Orders =>
            _orders ??= new OrderRepository(_context);
        
        public IProductRepository Products =>
            _products ??= new ProductRepository(_context);
        
        public async Task<int> SaveChangesAsync()
        {
            return await _context.SaveChangesAsync();
        }
        
        public async Task BeginTransactionAsync()
        {
            await _context.Database.BeginTransactionAsync();
        }
        
        public async Task CommitTransactionAsync()
        {
            await _context.Database.CommitTransactionAsync();
        }
        
        public async Task RollbackTransactionAsync()
        {
            await _context.Database.RollbackTransactionAsync();
        }
        
        public void Dispose()
        {
            _context.Dispose();
        }
    }
}

// Usage in command handler
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, int>
{
    private readonly IUnitOfWork _unitOfWork;
    
    public CreateOrderCommandHandler(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }
    
    public async Task<int> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        await _unitOfWork.BeginTransactionAsync();
        
        try
        {
            // Create order
            var order = new Order { CustomerId = request.CustomerId };
            var orderId = await _unitOfWork.Orders.AddAsync(order);
            
            // Update customer
            var customer = await _unitOfWork.Customers.GetByIdAsync(request.CustomerId);
            customer.TotalOrders++;
            await _unitOfWork.Customers.UpdateAsync(customer);
            
            await _unitOfWork.SaveChangesAsync();
            await _unitOfWork.CommitTransactionAsync();
            
            return orderId;
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

## Exception Handling

### Domain Exceptions

```csharp
// Domain/Exceptions/DomainException.cs
namespace Domain.Exceptions
{
    public abstract class DomainException : Exception
    {
        protected DomainException(string message) : base(message)
        {
        }
    }
    
    public class NotFoundException : DomainException
    {
        public NotFoundException(string message) : base(message)
        {
        }
    }
    
    public class ValidationException : DomainException
    {
        public ValidationException(string message) : base(message)
        {
        }
    }
    
    public class BusinessRuleException : DomainException
    {
        public BusinessRuleException(string message) : base(message)
        {
        }
    }
}
```

### Global Exception Middleware

```csharp
// API/Middleware/ExceptionHandlingMiddleware.cs
using Domain.Exceptions;
using System.Net;
using System.Text.Json;

namespace API.Middleware
{
    public class ExceptionHandlingMiddleware
    {
        private readonly RequestDelegate _next;
        private readonly ILogger<ExceptionHandlingMiddleware> _logger;
        
        public ExceptionHandlingMiddleware(
            RequestDelegate next,
            ILogger<ExceptionHandlingMiddleware> logger)
        {
            _next = next;
            _logger = logger;
        }
        
        public async Task InvokeAsync(HttpContext context)
        {
            try
            {
                await _next(context);
            }
            catch (Exception ex)
            {
                await HandleExceptionAsync(context, ex);
            }
        }
        
        private async Task HandleExceptionAsync(HttpContext context, Exception exception)
        {
            _logger.LogError(exception, "An error occurred");
            
            var (statusCode, message) = exception switch
            {
                NotFoundException => (HttpStatusCode.NotFound, exception.Message),
                ValidationException => (HttpStatusCode.BadRequest, exception.Message),
                BusinessRuleException => (HttpStatusCode.BadRequest, exception.Message),
                _ => (HttpStatusCode.InternalServerError, "An error occurred")
            };
            
            context.Response.ContentType = "application/json";
            context.Response.StatusCode = (int)statusCode;
            
            var response = new
            {
                error = message,
                statusCode = (int)statusCode
            };
            
            await context.Response.WriteAsync(JsonSerializer.Serialize(response));
        }
    }
}

// Program.cs
app.UseMiddleware<ExceptionHandlingMiddleware>();
```

---

## Testing Strategy

### Domain Tests

```csharp
// Domain.Tests/CustomerTests.cs
using Domain.Entities;
using Xunit;

namespace Domain.Tests
{
    public class CustomerTests
    {
        [Fact]
        public void Activate_ShouldChangeStatusToActive()
        {
            // Arrange
            var customer = new Customer
            {
                Status = CustomerStatus.Inactive
            };
            
            // Act
            customer.Activate();
            
            // Assert
            Assert.Equal(CustomerStatus.Active, customer.Status);
        }
        
        [Fact]
        public void Activate_WhenAlreadyActive_ShouldThrowException()
        {
            // Arrange
            var customer = new Customer
            {
                Status = CustomerStatus.Active
            };
            
            // Act & Assert
            Assert.Throws<InvalidOperationException>(() => customer.Activate());
        }
        
        [Fact]
        public void UpdateEmail_WithInvalidEmail_ShouldThrowException()
        {
            // Arrange
            var customer = new Customer();
            
            // Act & Assert
            Assert.Throws<ArgumentException>(() => customer.UpdateEmail("invalid-email"));
        }
    }
}
```

### Application Tests

```csharp
// Application.Tests/CreateCustomerCommandHandlerTests.cs
using Application.Commands;
using Application.Interfaces;
using Domain.Entities;
using Domain.Interfaces;
using Moq;
using Xunit;

namespace Application.Tests
{
    public class CreateCustomerCommandHandlerTests
    {
        [Fact]
        public async Task Handle_ShouldCreateCustomerAndSendEmail()
        {
            // Arrange
            var mockRepository = new Mock<ICustomerRepository>();
            var mockEmailService = new Mock<IEmailService>();
            
            mockRepository
                .Setup(r => r.AddAsync(It.IsAny<Customer>()))
                .ReturnsAsync(1);
            
            var handler = new CreateCustomerCommandHandler(
                mockRepository.Object,
                mockEmailService.Object);
            
            var command = new CreateCustomerCommand(
                "John",
                "Doe",
                "john@example.com");
            
            // Act
            var result = await handler.Handle(command, CancellationToken.None);
            
            // Assert
            Assert.Equal(1, result);
            mockRepository.Verify(r => r.AddAsync(It.IsAny<Customer>()), Times.Once);
            mockEmailService.Verify(
                e => e.SendWelcomeEmailAsync("john@example.com", "John"),
                Times.Once);
        }
    }
}
```

---

## Best Practices

### 1. Keep Domain Logic Pure

```csharp
// ✅ Good - Pure domain logic
public class Order
{
    public decimal CalculateTotal()
    {
        return OrderItems.Sum(i => i.Quantity * i.UnitPrice);
    }
    
    public void AddItem(Product product, int quantity)
    {
        if (quantity <= 0)
            throw new ArgumentException("Quantity must be positive");
        
        OrderItems.Add(new OrderItem
        {
            ProductId = product.Id,
            Quantity = quantity,
            UnitPrice = product.Price
        });
    }
}

// ❌ Bad - Database concerns in domain
public class Order
{
    public decimal CalculateTotal()
    {
        using var connection = new SqlConnection("...");
        return connection.QuerySingle<decimal>("SELECT SUM...");
    }
}
```

### 2. Use Value Objects

```csharp
// Value object for email
public record Email
{
    public string Value { get; }
    
    public Email(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new ArgumentException("Email cannot be empty");
        
        if (!value.Contains("@"))
            throw new ArgumentException("Invalid email format");
        
        Value = value;
    }
    
    public static implicit operator string(Email email) => email.Value;
    public static explicit operator Email(string value) => new Email(value);
}

// Usage in entity
public class Customer
{
    public Email Email { get; private set; }
    
    public void UpdateEmail(Email newEmail)
    {
        Email = newEmail; // Validation already done in Email constructor
    }
}
```

### 3. Separate Read and Write Models

```csharp
// Write model (full entity)
public class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public Email Email { get; set; }
    public List<Order> Orders { get; set; }
}

// Read model (optimized for queries)
public record CustomerListDto(
    int Id,
    string FullName,
    string Email,
    int OrderCount,
    decimal TotalSpent);
```

### 4. Use DTOs for API Boundaries

```csharp
// ✅ Good - DTO at API boundary
[HttpGet("{id}")]
public async Task<ActionResult<CustomerDto>> GetCustomer(int id)
{
    var customer = await _mediator.Send(new GetCustomerQuery(id));
    return Ok(customer); // CustomerDto
}

// ❌ Bad - Exposing domain entity
[HttpGet("{id}")]
public async Task<ActionResult<Customer>> GetCustomer(int id)
{
    var customer = await _repository.GetByIdAsync(id);
    return Ok(customer); // Domain entity exposed!
}
```

### 5. Dependency Injection Configuration

```csharp
// Infrastructure/Configuration/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        // Database
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(configuration.GetConnectionString("DefaultConnection")));
        
        // Repositories
        services.AddScoped<ICustomerRepository, CustomerRepository>();
        services.AddScoped<IOrderRepository, OrderRepository>();
        
        // Services
        services.AddScoped<IEmailService, EmailService>();
        services.AddScoped<INotificationService, NotificationService>();
        
        // Unit of Work
        services.AddScoped<IUnitOfWork, UnitOfWork>();
        
        return services;
    }
}

// Application/Configuration/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        // MediatR
        services.AddMediatR(cfg =>
            cfg.RegisterServicesFromAssembly(typeof(CreateCustomerCommand).Assembly));
        
        // FluentValidation
        services.AddValidatorsFromAssembly(typeof(CreateCustomerCommandValidator).Assembly);
        services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
        
        // AutoMapper
        services.AddAutoMapper(typeof(MappingProfile).Assembly);
        
        return services;
    }
}

// Program.cs
builder.Services.AddApplication();
builder.Services.AddInfrastructure(builder.Configuration);
```

---

## Quick Reference: Clean Architecture Rules

### ✅ DO

1. **Domain Layer:**
   - Keep pure business logic
   - No external dependencies
   - Define repository interfaces
   - Throw domain exceptions

2. **Application Layer:**
   - Orchestrate use cases
   - Define DTOs
   - Define service interfaces
   - Validate input

3. **Infrastructure Layer:**
   - Implement repositories
   - Implement external services
   - Database concerns
   - File system access

4. **Presentation Layer:**
   - Handle HTTP concerns
   - Map DTOs to API models
   - Configure DI container
   - Minimal logic

### ❌ DON'T

1. **Domain Layer:**
   - Don't reference other layers
   - Don't use EF Core DbContext
   - Don't access database directly
   - Don't use framework types

2. **Application Layer:**
   - Don't access database directly
   - Don't contain presentation logic
   - Don't implement infrastructure

3. **Infrastructure Layer:**
   - Don't contain business logic
   - Don't expose EF entities to application

4. **Presentation Layer:**
   - Don't contain business logic
   - Don't access database directly
   - Don't depend on infrastructure (except DI)

---

## Dependency Flow Diagram

```
┌────────────────────────────────────────────────┐
│              PRESENTATION                      │
│  (API, MVC, Blazor, Console)                  │
│  • Controllers                                 │
│  • ViewModels                                  │
│  • Startup/Program.cs                          │
└───────────────┬────────────────────────────────┘
                │ depends on
                ↓
┌───────────────────────────────────────────────┐
│           APPLICATION                          │
│  • Use Cases / Commands / Queries              │
│  • DTOs                                        │
│  • Service Interfaces                          │
│  • Validators                                  │
└───────────────┬────────────────────────────────┘
                │ depends on
                ↓
┌───────────────────────────────────────────────┐
│              DOMAIN                            │
│  • Entities                                    │
│  • Value Objects                               │
│  • Repository Interfaces                       │
│  • Domain Events                               │
│  • Business Rules                              │
└────────────────────────────────────────────────┘
                ↑
                │ implements
┌───────────────┴────────────────────────────────┐
│          INFRASTRUCTURE                        │
│  • Repository Implementations                  │
│  • DbContext                                   │
│  • External Service Implementations            │
│  • File System                                 │
└────────────────────────────────────────────────┘
```

---

**Guide Complete!** This comprehensive Clean Architecture guide covers all layers, dependency rules, CQRS, MediatR integration, validation, repository patterns, and best practices for building maintainable .NET applications! 🏗️
