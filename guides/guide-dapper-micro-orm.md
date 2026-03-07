# Dapper - Micro ORM Quick Reference

---

## What is Dapper?

**Dapper** = Lightweight, fast micro-ORM for .NET by Stack Overflow team

**Key Features:**
- ✅ **Blazing fast** - Nearly as fast as raw ADO.NET
- ✅ **Simple API** - Easy to learn and use
- ✅ **Lightweight** - ~1500 lines of code
- ✅ **No configuration** - No mappings, no context
- ✅ **Full control** - Write your own SQL
- ✅ **Multi-database** - Works with any ADO.NET provider

### Dapper vs Entity Framework Core

| Feature | Dapper | EF Core |
|---------|--------|---------|
| **Speed** | ⚡⚡⚡ Near ADO.NET | ⚡⚡ Slower |
| **Learning Curve** | ✅ Easy | ⚠️ Moderate |
| **SQL Control** | ✅ Full control | ⚠️ Generated SQL |
| **Migrations** | ❌ Manual | ✅ Built-in |
| **Change Tracking** | ❌ No | ✅ Yes |
| **Lazy Loading** | ❌ No | ✅ Yes |
| **Complex Queries** | ✅ Write SQL | ⚠️ LINQ translation |
| **Best For** | Performance-critical, simple CRUD | Complex domains, rapid development |

### When to Use Dapper

**✅ Use Dapper when:**
- Performance is critical
- Simple data access patterns
- You prefer writing SQL
- Legacy database with complex schema
- Microservices with focused data access
- Reports and analytics
- Hybrid approach (Dapper + EF Core)

**❌ Use EF Core when:**
- Complex domain models
- Need change tracking
- Want migrations
- Rapid prototyping
- Learning curve is acceptable

---

## Installation

```bash
# Install Dapper
dotnet add package Dapper

# For SQL Server
dotnet add package Microsoft.Data.SqlClient

# For PostgreSQL
dotnet add package Npgsql

# For MySQL
dotnet add package MySql.Data

# For SQLite
dotnet add package Microsoft.Data.Sqlite
```

```xml
<!-- In .csproj -->
<ItemGroup>
    <PackageReference Include="Dapper" Version="2.1.28" />
    <PackageReference Include="Microsoft.Data.SqlClient" Version="5.1.5" />
</ItemGroup>
```

---

## Basic Setup

### Connection String Configuration

```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDatabase;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}

// Program.cs (Dependency Injection)
using Microsoft.Data.SqlClient;

var builder = WebApplication.CreateBuilder(args);

// Register connection string
builder.Services.AddScoped(_ => 
    new SqlConnection(builder.Configuration.GetConnectionString("DefaultConnection")));

// Or create a service
builder.Services.AddScoped<ICustomerRepository, CustomerRepository>();
```

### Creating a Connection

```csharp
using Microsoft.Data.SqlClient;
using Dapper;

// Simple connection
string connectionString = "Server=localhost;Database=MyDb;Trusted_Connection=True;";
using var connection = new SqlConnection(connectionString);

// Connection is opened automatically by Dapper
var customers = connection.Query<Customer>("SELECT * FROM Customers");
```

---

## Basic CRUD Operations

### Query - Return Multiple Rows

```csharp
using var connection = new SqlConnection(connectionString);

// Simple query
var customers = connection.Query<Customer>("SELECT * FROM Customers");

// With WHERE clause
var activeCustomers = connection.Query<Customer>(
    "SELECT * FROM Customers WHERE IsActive = 1");

// With parameters (anonymous object)
var customer = connection.Query<Customer>(
    "SELECT * FROM Customers WHERE CustomerId = @Id",
    new { Id = 1 });

// Strongly typed result
public class Customer
{
    public int CustomerId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public bool IsActive { get; set; }
}
```

### QueryFirst / QueryFirstOrDefault

```csharp
// QueryFirst - Throws if no results
var customer = connection.QueryFirst<Customer>(
    "SELECT * FROM Customers WHERE CustomerId = @Id",
    new { Id = 1 });

// QueryFirstOrDefault - Returns null if no results
var customer = connection.QueryFirstOrDefault<Customer>(
    "SELECT * FROM Customers WHERE CustomerId = @Id",
    new { Id = 1 });

// Usage
if (customer != null)
{
    Console.WriteLine($"{customer.FirstName} {customer.LastName}");
}
```

### QuerySingle / QuerySingleOrDefault

```csharp
// QuerySingle - Throws if 0 or more than 1 result
var customer = connection.QuerySingle<Customer>(
    "SELECT * FROM Customers WHERE Email = @Email",
    new { Email = "john@example.com" });

// QuerySingleOrDefault - Returns null if no results, throws if more than 1
var customer = connection.QuerySingleOrDefault<Customer>(
    "SELECT * FROM Customers WHERE Email = @Email",
    new { Email = "john@example.com" });
```

### Execute - INSERT, UPDATE, DELETE

```csharp
// INSERT
var sql = @"
    INSERT INTO Customers (FirstName, LastName, Email, IsActive)
    VALUES (@FirstName, @LastName, @Email, @IsActive)";

var rowsAffected = connection.Execute(sql, new
{
    FirstName = "John",
    LastName = "Doe",
    Email = "john@example.com",
    IsActive = true
});

// UPDATE
sql = @"
    UPDATE Customers 
    SET FirstName = @FirstName, LastName = @LastName 
    WHERE CustomerId = @Id";

rowsAffected = connection.Execute(sql, new
{
    Id = 1,
    FirstName = "Jane",
    LastName = "Smith"
});

// DELETE
sql = "DELETE FROM Customers WHERE CustomerId = @Id";
rowsAffected = connection.Execute(sql, new { Id = 1 });
```

### ExecuteScalar - Return Single Value

```csharp
// Count
var count = connection.ExecuteScalar<int>("SELECT COUNT(*) FROM Customers");

// Max value
var maxId = connection.ExecuteScalar<int>("SELECT MAX(CustomerId) FROM Customers");

// Check existence
var exists = connection.ExecuteScalar<bool>(
    "SELECT CAST(CASE WHEN EXISTS(SELECT 1 FROM Customers WHERE Email = @Email) THEN 1 ELSE 0 END AS BIT)",
    new { Email = "john@example.com" });

// Get single value
var email = connection.ExecuteScalar<string>(
    "SELECT Email FROM Customers WHERE CustomerId = @Id",
    new { Id = 1 });
```

---

## Working with Parameters

### Anonymous Objects

```csharp
// Most common approach
var customer = connection.QueryFirstOrDefault<Customer>(
    "SELECT * FROM Customers WHERE CustomerId = @Id AND IsActive = @IsActive",
    new { Id = 1, IsActive = true });

// Multiple parameters
var orders = connection.Query<Order>(@"
    SELECT * FROM Orders 
    WHERE CustomerId = @CustomerId 
    AND OrderDate >= @StartDate 
    AND OrderDate <= @EndDate",
    new 
    { 
        CustomerId = 1, 
        StartDate = new DateTime(2024, 1, 1),
        EndDate = new DateTime(2024, 12, 31)
    });
```

### DynamicParameters

```csharp
using Dapper;

// When you need more control
var parameters = new DynamicParameters();
parameters.Add("@FirstName", "John");
parameters.Add("@LastName", "Doe");
parameters.Add("@Email", "john@example.com");

var customer = connection.QueryFirstOrDefault<Customer>(
    "SELECT * FROM Customers WHERE FirstName = @FirstName AND LastName = @LastName",
    parameters);

// With output parameter
parameters = new DynamicParameters();
parameters.Add("@FirstName", "John");
parameters.Add("@LastName", "Doe");
parameters.Add("@Email", "john@example.com");
parameters.Add("@CustomerId", dbType: DbType.Int32, direction: ParameterDirection.Output);

connection.Execute(@"
    INSERT INTO Customers (FirstName, LastName, Email)
    VALUES (@FirstName, @LastName, @Email);
    SET @CustomerId = SCOPE_IDENTITY();",
    parameters);

var newCustomerId = parameters.Get<int>("@CustomerId");
Console.WriteLine($"New Customer ID: {newCustomerId}");
```

### IN Clause with Lists

```csharp
// Dapper automatically expands lists
var customerIds = new[] { 1, 2, 3, 4, 5 };

var customers = connection.Query<Customer>(
    "SELECT * FROM Customers WHERE CustomerId IN @Ids",
    new { Ids = customerIds });

// Generates: WHERE CustomerId IN (@Ids1, @Ids2, @Ids3, @Ids4, @Ids5)

// With strings
var categories = new[] { "Electronics", "Books", "Clothing" };
var products = connection.Query<Product>(
    "SELECT * FROM Products WHERE Category IN @Categories",
    new { Categories = categories });
```

---

## Advanced Queries

### Multi-Mapping (JOIN queries)

```csharp
// One-to-One mapping
var sql = @"
    SELECT c.*, o.*
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerId = o.CustomerId
    WHERE c.CustomerId = @CustomerId";

var result = connection.Query<Customer, Order, Customer>(
    sql,
    (customer, order) =>
    {
        customer.Orders = customer.Orders ?? new List<Order>();
        customer.Orders.Add(order);
        return customer;
    },
    new { CustomerId = 1 },
    splitOn: "OrderId"  // Column that marks start of Order data
).GroupBy(c => c.CustomerId).Select(g => 
{
    var customer = g.First();
    customer.Orders = g.Select(c => c.Orders.First()).ToList();
    return customer;
}).First();

// Models
public class Customer
{
    public int CustomerId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public List<Order> Orders { get; set; }
}

public class Order
{
    public int OrderId { get; set; }
    public int CustomerId { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal TotalAmount { get; set; }
}
```

### Multi-Mapping (Better Approach)

```csharp
// Using Dictionary for better performance
var sql = @"
    SELECT 
        c.CustomerId, c.FirstName, c.LastName, c.Email,
        o.OrderId, o.OrderDate, o.TotalAmount
    FROM Customers c
    LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
    WHERE c.Country = @Country";

var customerDictionary = new Dictionary<int, Customer>();

var customers = connection.Query<Customer, Order, Customer>(
    sql,
    (customer, order) =>
    {
        if (!customerDictionary.TryGetValue(customer.CustomerId, out var existingCustomer))
        {
            existingCustomer = customer;
            existingCustomer.Orders = new List<Order>();
            customerDictionary.Add(existingCustomer.CustomerId, existingCustomer);
        }

        if (order != null)
        {
            existingCustomer.Orders.Add(order);
        }

        return existingCustomer;
    },
    new { Country = "USA" },
    splitOn: "OrderId"
).Distinct().ToList();
```

### Three-Way Mapping

```csharp
var sql = @"
    SELECT 
        c.CustomerId, c.FirstName, c.LastName,
        o.OrderId, o.OrderDate, o.TotalAmount,
        oi.OrderItemId, oi.ProductId, oi.Quantity, oi.UnitPrice
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerId = o.CustomerId
    INNER JOIN OrderItems oi ON o.OrderId = oi.OrderId
    WHERE c.CustomerId = @CustomerId";

var customerDictionary = new Dictionary<int, Customer>();
var orderDictionary = new Dictionary<int, Order>();

var result = connection.Query<Customer, Order, OrderItem, Customer>(
    sql,
    (customer, order, orderItem) =>
    {
        // Get or create customer
        if (!customerDictionary.TryGetValue(customer.CustomerId, out var existingCustomer))
        {
            existingCustomer = customer;
            existingCustomer.Orders = new List<Order>();
            customerDictionary.Add(existingCustomer.CustomerId, existingCustomer);
        }

        // Get or create order
        if (!orderDictionary.TryGetValue(order.OrderId, out var existingOrder))
        {
            existingOrder = order;
            existingOrder.Items = new List<OrderItem>();
            existingCustomer.Orders.Add(existingOrder);
            orderDictionary.Add(existingOrder.OrderId, existingOrder);
        }

        // Add order item
        existingOrder.Items.Add(orderItem);

        return existingCustomer;
    },
    new { CustomerId = 1 },
    splitOn: "OrderId,OrderItemId"
).FirstOrDefault();
```

### Multiple Result Sets

```csharp
var sql = @"
    SELECT * FROM Customers WHERE CustomerId = @CustomerId;
    SELECT * FROM Orders WHERE CustomerId = @CustomerId;
    SELECT * FROM Addresses WHERE CustomerId = @CustomerId;";

using var multi = connection.QueryMultiple(sql, new { CustomerId = 1 });

var customer = multi.ReadFirst<Customer>();
var orders = multi.Read<Order>().ToList();
var addresses = multi.Read<Address>().ToList();

customer.Orders = orders;
customer.Addresses = addresses;

// Or with async
using var multi = await connection.QueryMultipleAsync(sql, new { CustomerId = 1 });

var customer = await multi.ReadFirstAsync<Customer>();
var orders = (await multi.ReadAsync<Order>()).ToList();
var addresses = (await multi.ReadAsync<Address>()).ToList();
```

---

## Stored Procedures

### Calling Stored Procedures

```csharp
// Simple stored procedure
var customers = connection.Query<Customer>(
    "GetCustomersByCountry",
    new { Country = "USA" },
    commandType: CommandType.StoredProcedure
);

// With output parameter
var parameters = new DynamicParameters();
parameters.Add("@CustomerId", 1);
parameters.Add("@TotalOrders", dbType: DbType.Int32, direction: ParameterDirection.Output);
parameters.Add("@TotalSpent", dbType: DbType.Decimal, direction: ParameterDirection.Output);

connection.Execute(
    "GetCustomerStats",
    parameters,
    commandType: CommandType.StoredProcedure
);

var totalOrders = parameters.Get<int>("@TotalOrders");
var totalSpent = parameters.Get<decimal>("@TotalSpent");

// With return value
parameters = new DynamicParameters();
parameters.Add("@FirstName", "John");
parameters.Add("@LastName", "Doe");
parameters.Add("@Email", "john@example.com");
parameters.Add("@ReturnValue", dbType: DbType.Int32, direction: ParameterDirection.ReturnValue);

connection.Execute(
    "CreateCustomer",
    parameters,
    commandType: CommandType.StoredProcedure
);

var returnValue = parameters.Get<int>("@ReturnValue");
```

### Example Stored Procedures

```sql
-- Simple stored procedure
CREATE PROCEDURE GetCustomersByCountry
    @Country NVARCHAR(50)
AS
BEGIN
    SELECT * FROM Customers WHERE Country = @Country;
END;

-- With output parameters
CREATE PROCEDURE GetCustomerStats
    @CustomerId INT,
    @TotalOrders INT OUTPUT,
    @TotalSpent DECIMAL(10,2) OUTPUT
AS
BEGIN
    SELECT 
        @TotalOrders = COUNT(OrderId),
        @TotalSpent = SUM(TotalAmount)
    FROM Orders
    WHERE CustomerId = @CustomerId;
END;

-- With return value
CREATE PROCEDURE CreateCustomer
    @FirstName NVARCHAR(50),
    @LastName NVARCHAR(50),
    @Email NVARCHAR(100)
AS
BEGIN
    INSERT INTO Customers (FirstName, LastName, Email)
    VALUES (@FirstName, @LastName, @Email);
    
    RETURN SCOPE_IDENTITY();
END;
```

---

## Transactions

### Basic Transaction

```csharp
using var connection = new SqlConnection(connectionString);
connection.Open();

using var transaction = connection.BeginTransaction();

try
{
    // Multiple operations in same transaction
    connection.Execute(@"
        INSERT INTO Customers (FirstName, LastName, Email)
        VALUES (@FirstName, @LastName, @Email)",
        new { FirstName = "John", LastName = "Doe", Email = "john@example.com" },
        transaction: transaction);

    connection.Execute(@"
        UPDATE Accounts SET Balance = Balance - @Amount
        WHERE AccountId = @FromAccountId",
        new { FromAccountId = 1, Amount = 100 },
        transaction: transaction);

    connection.Execute(@"
        UPDATE Accounts SET Balance = Balance + @Amount
        WHERE AccountId = @ToAccountId",
        new { ToAccountId = 2, Amount = 100 },
        transaction: transaction);

    transaction.Commit();
}
catch
{
    transaction.Rollback();
    throw;
}
```

### Transaction with Using Statement

```csharp
using var connection = new SqlConnection(connectionString);
connection.Open();

using (var transaction = connection.BeginTransaction())
{
    try
    {
        var customerId = connection.ExecuteScalar<int>(@"
            INSERT INTO Customers (FirstName, LastName, Email)
            VALUES (@FirstName, @LastName, @Email);
            SELECT CAST(SCOPE_IDENTITY() AS INT);",
            new { FirstName = "John", LastName = "Doe", Email = "john@example.com" },
            transaction: transaction);

        connection.Execute(@"
            INSERT INTO Orders (CustomerId, OrderDate, TotalAmount)
            VALUES (@CustomerId, @OrderDate, @TotalAmount)",
            new { CustomerId = customerId, OrderDate = DateTime.Now, TotalAmount = 100.00m },
            transaction: transaction);

        transaction.Commit();
    }
    catch
    {
        transaction.Rollback();
        throw;
    }
}
```

### TransactionScope (Distributed Transactions)

```csharp
using System.Transactions;

using (var scope = new TransactionScope())
{
    using var connection1 = new SqlConnection(connectionString1);
    using var connection2 = new SqlConnection(connectionString2);

    connection1.Execute(@"
        UPDATE Accounts SET Balance = Balance - @Amount
        WHERE AccountId = @AccountId",
        new { AccountId = 1, Amount = 100 });

    connection2.Execute(@"
        INSERT INTO AuditLog (Action, Timestamp)
        VALUES (@Action, @Timestamp)",
        new { Action = "Transfer", Timestamp = DateTime.Now });

    scope.Complete(); // Commit both transactions
}
```

---

## Async Operations

### Async CRUD

```csharp
// Query async
var customers = await connection.QueryAsync<Customer>(
    "SELECT * FROM Customers WHERE Country = @Country",
    new { Country = "USA" });

// QueryFirst async
var customer = await connection.QueryFirstAsync<Customer>(
    "SELECT * FROM Customers WHERE CustomerId = @Id",
    new { Id = 1 });

// QueryFirstOrDefault async
var customer = await connection.QueryFirstOrDefaultAsync<Customer>(
    "SELECT * FROM Customers WHERE Email = @Email",
    new { Email = "john@example.com" });

// Execute async
var rowsAffected = await connection.ExecuteAsync(@"
    INSERT INTO Customers (FirstName, LastName, Email)
    VALUES (@FirstName, @LastName, @Email)",
    new { FirstName = "John", LastName = "Doe", Email = "john@example.com" });

// ExecuteScalar async
var count = await connection.ExecuteScalarAsync<int>(
    "SELECT COUNT(*) FROM Customers");
```

### Async with Multiple Results

```csharp
var sql = @"
    SELECT * FROM Customers WHERE CustomerId = @CustomerId;
    SELECT * FROM Orders WHERE CustomerId = @CustomerId;";

using var multi = await connection.QueryMultipleAsync(sql, new { CustomerId = 1 });

var customer = await multi.ReadFirstAsync<Customer>();
var orders = (await multi.ReadAsync<Order>()).ToList();

customer.Orders = orders;
```

---

## Bulk Operations

### Bulk Insert

```csharp
// Insert multiple rows
var customers = new[]
{
    new { FirstName = "John", LastName = "Doe", Email = "john@example.com" },
    new { FirstName = "Jane", LastName = "Smith", Email = "jane@example.com" },
    new { FirstName = "Bob", LastName = "Johnson", Email = "bob@example.com" }
};

var sql = @"
    INSERT INTO Customers (FirstName, LastName, Email)
    VALUES (@FirstName, @LastName, @Email)";

var rowsAffected = connection.Execute(sql, customers);

// Async bulk insert
var rowsAffected = await connection.ExecuteAsync(sql, customers);
```

### Bulk Update

```csharp
var updates = new[]
{
    new { CustomerId = 1, IsActive = true },
    new { CustomerId = 2, IsActive = false },
    new { CustomerId = 3, IsActive = true }
};

var sql = "UPDATE Customers SET IsActive = @IsActive WHERE CustomerId = @CustomerId";
var rowsAffected = connection.Execute(sql, updates);
```

### Bulk Delete

```csharp
var customerIds = new[] { 1, 2, 3, 4, 5 };

var sql = "DELETE FROM Customers WHERE CustomerId IN @Ids";
var rowsAffected = connection.Execute(sql, new { Ids = customerIds });
```

---

## Dynamic Queries

### Dynamic Results

```csharp
// Query without model (dynamic)
var customers = connection.Query(
    "SELECT CustomerId, FirstName, LastName FROM Customers");

foreach (var customer in customers)
{
    Console.WriteLine($"{customer.CustomerId}: {customer.FirstName} {customer.LastName}");
}

// Single dynamic result
var customer = connection.QueryFirstOrDefault(
    "SELECT * FROM Customers WHERE CustomerId = @Id",
    new { Id = 1 });

Console.WriteLine($"{customer.FirstName} {customer.LastName}");
```

### Building Dynamic SQL

```csharp
public async Task<IEnumerable<Customer>> SearchCustomersAsync(
    string? firstName = null,
    string? lastName = null,
    string? country = null,
    bool? isActive = null)
{
    var sql = "SELECT * FROM Customers WHERE 1=1";
    var parameters = new DynamicParameters();

    if (!string.IsNullOrEmpty(firstName))
    {
        sql += " AND FirstName LIKE @FirstName";
        parameters.Add("FirstName", $"%{firstName}%");
    }

    if (!string.IsNullOrEmpty(lastName))
    {
        sql += " AND LastName LIKE @LastName";
        parameters.Add("LastName", $"%{lastName}%");
    }

    if (!string.IsNullOrEmpty(country))
    {
        sql += " AND Country = @Country";
        parameters.Add("Country", country);
    }

    if (isActive.HasValue)
    {
        sql += " AND IsActive = @IsActive";
        parameters.Add("IsActive", isActive.Value);
    }

    using var connection = new SqlConnection(_connectionString);
    return await connection.QueryAsync<Customer>(sql, parameters);
}
```

---

## Repository Pattern with Dapper

### Interface

```csharp
public interface ICustomerRepository
{
    Task<Customer?> GetByIdAsync(int customerId);
    Task<IEnumerable<Customer>> GetAllAsync();
    Task<IEnumerable<Customer>> GetByCountryAsync(string country);
    Task<int> CreateAsync(Customer customer);
    Task<bool> UpdateAsync(Customer customer);
    Task<bool> DeleteAsync(int customerId);
}
```

### Implementation

```csharp
using Microsoft.Data.SqlClient;
using Dapper;

public class CustomerRepository : ICustomerRepository
{
    private readonly string _connectionString;

    public CustomerRepository(IConfiguration configuration)
    {
        _connectionString = configuration.GetConnectionString("DefaultConnection")
            ?? throw new ArgumentNullException(nameof(configuration));
    }

    public async Task<Customer?> GetByIdAsync(int customerId)
    {
        const string sql = @"
            SELECT CustomerId, FirstName, LastName, Email, Country, IsActive
            FROM Customers
            WHERE CustomerId = @CustomerId";

        using var connection = new SqlConnection(_connectionString);
        return await connection.QueryFirstOrDefaultAsync<Customer>(
            sql, 
            new { CustomerId = customerId });
    }

    public async Task<IEnumerable<Customer>> GetAllAsync()
    {
        const string sql = "SELECT * FROM Customers ORDER BY LastName, FirstName";

        using var connection = new SqlConnection(_connectionString);
        return await connection.QueryAsync<Customer>(sql);
    }

    public async Task<IEnumerable<Customer>> GetByCountryAsync(string country)
    {
        const string sql = @"
            SELECT * FROM Customers 
            WHERE Country = @Country 
            ORDER BY LastName, FirstName";

        using var connection = new SqlConnection(_connectionString);
        return await connection.QueryAsync<Customer>(sql, new { Country = country });
    }

    public async Task<int> CreateAsync(Customer customer)
    {
        const string sql = @"
            INSERT INTO Customers (FirstName, LastName, Email, Country, IsActive)
            VALUES (@FirstName, @LastName, @Email, @Country, @IsActive);
            SELECT CAST(SCOPE_IDENTITY() AS INT);";

        using var connection = new SqlConnection(_connectionString);
        return await connection.ExecuteScalarAsync<int>(sql, customer);
    }

    public async Task<bool> UpdateAsync(Customer customer)
    {
        const string sql = @"
            UPDATE Customers
            SET FirstName = @FirstName,
                LastName = @LastName,
                Email = @Email,
                Country = @Country,
                IsActive = @IsActive
            WHERE CustomerId = @CustomerId";

        using var connection = new SqlConnection(_connectionString);
        var rowsAffected = await connection.ExecuteAsync(sql, customer);
        return rowsAffected > 0;
    }

    public async Task<bool> DeleteAsync(int customerId)
    {
        const string sql = "DELETE FROM Customers WHERE CustomerId = @CustomerId";

        using var connection = new SqlConnection(_connectionString);
        var rowsAffected = await connection.ExecuteAsync(sql, new { CustomerId = customerId });
        return rowsAffected > 0;
    }
}
```

### Registration and Usage

```csharp
// Program.cs
builder.Services.AddScoped<ICustomerRepository, CustomerRepository>();

// Controller or Service
public class CustomerService
{
    private readonly ICustomerRepository _customerRepository;

    public CustomerService(ICustomerRepository customerRepository)
    {
        _customerRepository = customerRepository;
    }

    public async Task<Customer?> GetCustomerAsync(int customerId)
    {
        return await _customerRepository.GetByIdAsync(customerId);
    }

    public async Task<int> CreateCustomerAsync(Customer customer)
    {
        return await _customerRepository.CreateAsync(customer);
    }
}
```

---

## Unit of Work Pattern

### Interface

```csharp
public interface IUnitOfWork : IDisposable
{
    ICustomerRepository Customers { get; }
    IOrderRepository Orders { get; }
    IProductRepository Products { get; }
    
    void BeginTransaction();
    void Commit();
    void Rollback();
}
```

### Implementation

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly SqlConnection _connection;
    private SqlTransaction? _transaction;
    private bool _disposed;

    private ICustomerRepository? _customers;
    private IOrderRepository? _orders;
    private IProductRepository? _products;

    public UnitOfWork(string connectionString)
    {
        _connection = new SqlConnection(connectionString);
        _connection.Open();
    }

    public ICustomerRepository Customers
    {
        get { return _customers ??= new CustomerRepository(_connection, _transaction); }
    }

    public IOrderRepository Orders
    {
        get { return _orders ??= new OrderRepository(_connection, _transaction); }
    }

    public IProductRepository Products
    {
        get { return _products ??= new ProductRepository(_connection, _transaction); }
    }

    public void BeginTransaction()
    {
        _transaction = _connection.BeginTransaction();
    }

    public void Commit()
    {
        if (_transaction == null)
            throw new InvalidOperationException("Transaction not started");

        try
        {
            _transaction.Commit();
        }
        catch
        {
            _transaction.Rollback();
            throw;
        }
        finally
        {
            _transaction.Dispose();
            _transaction = null;
        }
    }

    public void Rollback()
    {
        if (_transaction == null)
            throw new InvalidOperationException("Transaction not started");

        _transaction.Rollback();
        _transaction.Dispose();
        _transaction = null;
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                _transaction?.Dispose();
                _connection?.Dispose();
            }
            _disposed = true;
        }
    }
}

// Usage
using var unitOfWork = new UnitOfWork(connectionString);

try
{
    unitOfWork.BeginTransaction();

    var customerId = await unitOfWork.Customers.CreateAsync(customer);
    await unitOfWork.Orders.CreateAsync(new Order { CustomerId = customerId });

    unitOfWork.Commit();
}
catch
{
    unitOfWork.Rollback();
    throw;
}
```

---

## Performance Optimization

### Query Performance

```csharp
// ❌ Bad - N+1 query problem
var customers = connection.Query<Customer>("SELECT * FROM Customers");
foreach (var customer in customers)
{
    customer.Orders = connection.Query<Order>(
        "SELECT * FROM Orders WHERE CustomerId = @CustomerId",
        new { CustomerId = customer.CustomerId }).ToList();
}

// ✅ Good - Single query with multi-mapping
var sql = @"
    SELECT c.*, o.*
    FROM Customers c
    LEFT JOIN Orders o ON c.CustomerId = o.CustomerId";

var customerDict = new Dictionary<int, Customer>();
var customers = connection.Query<Customer, Order, Customer>(
    sql,
    (customer, order) =>
    {
        if (!customerDict.TryGetValue(customer.CustomerId, out var existingCustomer))
        {
            existingCustomer = customer;
            existingCustomer.Orders = new List<Order>();
            customerDict.Add(customer.CustomerId, existingCustomer);
        }
        if (order != null)
        {
            existingCustomer.Orders.Add(order);
        }
        return existingCustomer;
    },
    splitOn: "OrderId"
).Distinct();
```

### Buffered vs Unbuffered Queries

```csharp
// Buffered (default) - Loads all results into memory immediately
var customers = connection.Query<Customer>("SELECT * FROM Customers");
// Fast enumeration, uses more memory

// Unbuffered - Streams results (forward-only)
var customers = connection.Query<Customer>(
    "SELECT * FROM Customers",
    buffered: false);
// Memory efficient, connection stays open during enumeration

// Use unbuffered for:
// - Large result sets
// - When you only need to enumerate once
// - Memory-constrained scenarios

// Use buffered for:
// - Multiple enumerations
// - Small to medium result sets
// - When you can close connection immediately
```

### Command Timeout

```csharp
// Set command timeout (default is 30 seconds)
var customers = connection.Query<Customer>(
    "SELECT * FROM Customers",
    commandTimeout: 60); // 60 seconds

// For long-running queries
var report = connection.Query<ReportData>(
    "EXEC GenerateComplexReport @Year",
    new { Year = 2024 },
    commandTimeout: 300); // 5 minutes
```

---

## Best Practices

### 1. Always Use Parameters (Prevent SQL Injection)

```csharp
// ❌ DANGEROUS - SQL Injection vulnerability
var email = "user@example.com";
var customer = connection.QueryFirstOrDefault<Customer>(
    $"SELECT * FROM Customers WHERE Email = '{email}'");

// ✅ SAFE - Parameterized query
var customer = connection.QueryFirstOrDefault<Customer>(
    "SELECT * FROM Customers WHERE Email = @Email",
    new { Email = email });
```

### 2. Use Async Methods

```csharp
// ✅ Async for scalability
public async Task<Customer?> GetCustomerAsync(int customerId)
{
    using var connection = new SqlConnection(_connectionString);
    return await connection.QueryFirstOrDefaultAsync<Customer>(
        "SELECT * FROM Customers WHERE CustomerId = @CustomerId",
        new { CustomerId = customerId });
}
```

### 3. Dispose Connections Properly

```csharp
// ✅ Using statement (auto-dispose)
using var connection = new SqlConnection(connectionString);
var customers = await connection.QueryAsync<Customer>("SELECT * FROM Customers");

// Or with explicit using block
using (var connection = new SqlConnection(connectionString))
{
    var customers = await connection.QueryAsync<Customer>("SELECT * FROM Customers");
}
```

### 4. Use Const for SQL Strings

```csharp
public class CustomerRepository
{
    private const string GetByIdSql = @"
        SELECT CustomerId, FirstName, LastName, Email
        FROM Customers
        WHERE CustomerId = @CustomerId";

    private const string GetAllSql = "SELECT * FROM Customers";

    public async Task<Customer?> GetByIdAsync(int customerId)
    {
        using var connection = new SqlConnection(_connectionString);
        return await connection.QueryFirstOrDefaultAsync<Customer>(
            GetByIdSql,
            new { CustomerId = customerId });
    }
}
```

### 5. Handle NULLs Properly

```csharp
// Model with nullable types
public class Customer
{
    public int CustomerId { get; set; }
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string? Email { get; set; }  // Nullable
    public string? Phone { get; set; }  // Nullable
    public DateTime? LastLoginDate { get; set; }  // Nullable
}

// Dapper automatically maps NULL to null for nullable types
```

### 6. Use Transactions for Multiple Operations

```csharp
// ✅ Transaction for data consistency
using var connection = new SqlConnection(connectionString);
connection.Open();

using var transaction = connection.BeginTransaction();
try
{
    await connection.ExecuteAsync(insertCustomerSql, customer, transaction);
    await connection.ExecuteAsync(insertOrderSql, order, transaction);
    transaction.Commit();
}
catch
{
    transaction.Rollback();
    throw;
}
```

### 7. Avoid Over-Fetching Data

```csharp
// ❌ Fetching unnecessary data
var customers = connection.Query<Customer>("SELECT * FROM Customers");

// ✅ Select only needed columns
var customers = connection.Query<Customer>(
    "SELECT CustomerId, FirstName, LastName, Email FROM Customers");
```

### 8. Use Stored Procedures for Complex Logic

```csharp
// ✅ Stored procedure for complex business logic
var result = await connection.QueryAsync<ReportData>(
    "GetSalesReport",
    new { StartDate = startDate, EndDate = endDate },
    commandType: CommandType.StoredProcedure);
```

---

## Common Patterns

### Pagination

```csharp
public async Task<(IEnumerable<Customer> Items, int TotalCount)> GetPagedCustomersAsync(
    int pageNumber, 
    int pageSize)
{
    const string sql = @"
        SELECT CustomerId, FirstName, LastName, Email
        FROM Customers
        ORDER BY CustomerId
        OFFSET @Offset ROWS
        FETCH NEXT @PageSize ROWS ONLY;

        SELECT COUNT(*) FROM Customers;";

    using var connection = new SqlConnection(_connectionString);
    using var multi = await connection.QueryMultipleAsync(sql, new
    {
        Offset = (pageNumber - 1) * pageSize,
        PageSize = pageSize
    });

    var items = await multi.ReadAsync<Customer>();
    var totalCount = await multi.ReadSingleAsync<int>();

    return (items, totalCount);
}
```

### Soft Delete

```csharp
public async Task<bool> SoftDeleteAsync(int customerId)
{
    const string sql = @"
        UPDATE Customers
        SET IsDeleted = 1, DeletedAt = @DeletedAt
        WHERE CustomerId = @CustomerId AND IsDeleted = 0";

    using var connection = new SqlConnection(_connectionString);
    var rowsAffected = await connection.ExecuteAsync(sql, new
    {
        CustomerId = customerId,
        DeletedAt = DateTime.UtcNow
    });

    return rowsAffected > 0;
}

public async Task<IEnumerable<Customer>> GetActiveCustomersAsync()
{
    const string sql = @"
        SELECT * FROM Customers
        WHERE IsDeleted = 0
        ORDER BY LastName, FirstName";

    using var connection = new SqlConnection(_connectionString);
    return await connection.QueryAsync<Customer>(sql);
}
```

### Audit Trail

```csharp
public async Task<int> CreateCustomerWithAuditAsync(Customer customer, int userId)
{
    const string insertCustomerSql = @"
        INSERT INTO Customers (FirstName, LastName, Email, CreatedBy, CreatedAt)
        VALUES (@FirstName, @LastName, @Email, @CreatedBy, @CreatedAt);
        SELECT CAST(SCOPE_IDENTITY() AS INT);";

    const string insertAuditSql = @"
        INSERT INTO AuditLog (EntityType, EntityId, Action, UserId, Timestamp)
        VALUES (@EntityType, @EntityId, @Action, @UserId, @Timestamp);";

    using var connection = new SqlConnection(_connectionString);
    connection.Open();

    using var transaction = connection.BeginTransaction();
    try
    {
        var customerId = await connection.ExecuteScalarAsync<int>(
            insertCustomerSql,
            new
            {
                customer.FirstName,
                customer.LastName,
                customer.Email,
                CreatedBy = userId,
                CreatedAt = DateTime.UtcNow
            },
            transaction);

        await connection.ExecuteAsync(
            insertAuditSql,
            new
            {
                EntityType = "Customer",
                EntityId = customerId,
                Action = "Create",
                UserId = userId,
                Timestamp = DateTime.UtcNow
            },
            transaction);

        transaction.Commit();
        return customerId;
    }
    catch
    {
        transaction.Rollback();
        throw;
    }
}
```

---

## Performance Comparison

### Dapper vs EF Core Benchmark

```csharp
// Benchmark results (approximate):

// Single row query:
// Dapper:      ~25 μs
// EF Core:     ~75 μs (3x slower)
// ADO.NET:     ~20 μs

// Query 1000 rows:
// Dapper:      ~500 μs
// EF Core:     ~1500 μs (3x slower)
// ADO.NET:     ~450 μs

// Insert 1000 rows:
// Dapper:      ~50 ms
// EF Core:     ~150 ms (3x slower)
// ADO.NET:     ~45 ms

// Conclusion: Dapper is nearly as fast as raw ADO.NET
```

---

## Dapper Extensions

### Dapper.Contrib

```bash
dotnet add package Dapper.Contrib
```

```csharp
using Dapper.Contrib.Extensions;

[Table("Customers")]
public class Customer
{
    [Key]
    public int CustomerId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    
    [Computed]
    public string FullName => $"{FirstName} {LastName}";
}

// Simple CRUD
var customer = connection.Get<Customer>(1);
var allCustomers = connection.GetAll<Customer>();
var id = connection.Insert(new Customer { FirstName = "John", LastName = "Doe" });
connection.Update(customer);
connection.Delete(customer);
```

### Dapper.SimpleCRUD

```bash
dotnet add package Dapper.SimpleCRUD
```

```csharp
using Dapper.SimpleCRUD;

// Get by ID
var customer = connection.Get<Customer>(1);

// Get list with WHERE
var customers = connection.GetList<Customer>("WHERE Country = @Country", new { Country = "USA" });

// Get all
var allCustomers = connection.GetList<Customer>();

// Insert
var id = connection.Insert(customer);

// Update
connection.Update(customer);

// Delete
connection.Delete(customer);

// Delete by ID
connection.Delete<Customer>(1);
```

---

## Quick Reference

### Common Dapper Methods

| Method | Returns | Use Case |
|--------|---------|----------|
| `Query<T>` | `IEnumerable<T>` | Multiple rows |
| `QueryFirst<T>` | `T` | First row (throws if empty) |
| `QueryFirstOrDefault<T>` | `T?` | First row or null |
| `QuerySingle<T>` | `T` | Exactly one row |
| `QuerySingleOrDefault<T>` | `T?` | 0 or 1 row |
| `Execute` | `int` | INSERT/UPDATE/DELETE |
| `ExecuteScalar<T>` | `T` | Single value (COUNT, MAX, etc.) |
| `QueryMultiple` | `GridReader` | Multiple result sets |

### Async Versions

All methods have async versions with `Async` suffix:
- `QueryAsync<T>`
- `QueryFirstOrDefaultAsync<T>`
- `ExecuteAsync`
- `ExecuteScalarAsync<T>`
- `QueryMultipleAsync`

---

## Decision Tree: Dapper vs EF Core

```
Consider your requirements:

Performance critical? (< 100ms response time)
└─> ✅ Dapper

Complex domain model with relationships?
└─> ✅ EF Core

Need change tracking?
└─> ✅ EF Core

Legacy database with complex schema?
└─> ✅ Dapper (full SQL control)

Rapid prototyping?
└─> ✅ EF Core (migrations, scaffolding)

Team comfortable with SQL?
└─> ✅ Dapper

Team prefers LINQ?
└─> ✅ EF Core

Reports and analytics?
└─> ✅ Dapper (optimized queries)

Microservices with simple data access?
└─> ✅ Dapper (lightweight)

Existing app with EF Core?
└─> ✅ Hybrid (EF Core + Dapper for performance-critical queries)
```

---

**Guide Complete!** This comprehensive Dapper guide covers setup, CRUD operations, stored procedures, transactions, async patterns, bulk operations, repository pattern, performance optimization, and when to use Dapper vs EF Core. Use Dapper when performance matters and you want full control over your SQL! ⚡
