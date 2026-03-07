# SOLID Principles & Design Patterns Quick Reference

---

## SOLID Principles Overview

**SOLID** = Five principles for object-oriented design

**Purpose:**
- ✅ **Maintainable** code
- ✅ **Testable** code
- ✅ **Flexible** and extensible
- ✅ **Reduce coupling**
- ✅ **Easier to understand**

| Principle | Description |
|-----------|-------------|
| **S** - Single Responsibility | A class should have one reason to change |
| **O** - Open/Closed | Open for extension, closed for modification |
| **L** - Liskov Substitution | Subtypes must be substitutable for base types |
| **I** - Interface Segregation | Many specific interfaces > one general interface |
| **D** - Dependency Inversion | Depend on abstractions, not concretions |

---

## Single Responsibility Principle (SRP)

**Definition:** A class should have only one reason to change (one responsibility)

### ❌ Violation

```csharp
// Bad: UserService has multiple responsibilities
public class UserService
{
    public void CreateUser(User user)
    {
        // 1. Validate user
        if (string.IsNullOrEmpty(user.Email))
            throw new Exception("Email required");
        
        // 2. Save to database
        var connection = new SqlConnection("...");
        connection.Execute("INSERT INTO Users...", user);
        
        // 3. Send welcome email
        var smtpClient = new SmtpClient();
        smtpClient.Send(new MailMessage
        {
            To = { user.Email },
            Subject = "Welcome!",
            Body = "Welcome to our platform"
        });
        
        // 4. Log activity
        File.AppendAllText("log.txt", $"User {user.Email} created");
    }
}

// Problems:
// - Changes to validation affect this class
// - Changes to database affect this class
// - Changes to email affect this class
// - Changes to logging affect this class
// - Hard to test
// - Hard to reuse parts
```

### ✅ Following SRP

```csharp
// Good: Each class has single responsibility

// Responsibility 1: Validation
public class UserValidator
{
    public void Validate(User user)
    {
        if (string.IsNullOrEmpty(user.Email))
            throw new ValidationException("Email required");
        
        if (!user.Email.Contains("@"))
            throw new ValidationException("Invalid email format");
    }
}

// Responsibility 2: Data Access
public class UserRepository
{
    private readonly string _connectionString;
    
    public UserRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public void Add(User user)
    {
        using var connection = new SqlConnection(_connectionString);
        connection.Execute("INSERT INTO Users (Email, Name) VALUES (@Email, @Name)", user);
    }
}

// Responsibility 3: Email Notification
public class EmailService
{
    private readonly SmtpClient _smtpClient;
    
    public EmailService(SmtpClient smtpClient)
    {
        _smtpClient = smtpClient;
    }
    
    public void SendWelcomeEmail(User user)
    {
        _smtpClient.Send(new MailMessage
        {
            To = { user.Email },
            Subject = "Welcome!",
            Body = "Welcome to our platform"
        });
    }
}

// Responsibility 4: Logging
public class Logger
{
    private readonly string _logFilePath;
    
    public Logger(string logFilePath)
    {
        _logFilePath = logFilePath;
    }
    
    public void Log(string message)
    {
        File.AppendAllText(_logFilePath, $"{DateTime.Now}: {message}\n");
    }
}

// Orchestration
public class UserService
{
    private readonly UserValidator _validator;
    private readonly UserRepository _repository;
    private readonly EmailService _emailService;
    private readonly Logger _logger;
    
    public UserService(
        UserValidator validator,
        UserRepository repository,
        EmailService emailService,
        Logger logger)
    {
        _validator = validator;
        _repository = repository;
        _emailService = emailService;
        _logger = logger;
    }
    
    public void CreateUser(User user)
    {
        _validator.Validate(user);
        _repository.Add(user);
        _emailService.SendWelcomeEmail(user);
        _logger.Log($"User {user.Email} created");
    }
}

// Benefits:
// - Each class is easy to understand
// - Easy to test in isolation
// - Easy to modify without affecting others
// - Easy to reuse components
```

---

## Open/Closed Principle (OCP)

**Definition:** Software entities should be open for extension but closed for modification

### ❌ Violation

```csharp
// Bad: Need to modify class to add new payment types
public class PaymentProcessor
{
    public void ProcessPayment(string paymentType, decimal amount)
    {
        if (paymentType == "CreditCard")
        {
            // Process credit card
            Console.WriteLine($"Processing ${amount} via Credit Card");
        }
        else if (paymentType == "PayPal")
        {
            // Process PayPal
            Console.WriteLine($"Processing ${amount} via PayPal");
        }
        else if (paymentType == "BankTransfer")
        {
            // Process bank transfer
            Console.WriteLine($"Processing ${amount} via Bank Transfer");
        }
        // Adding Crypto payment requires modifying this class!
    }
}
```

### ✅ Following OCP

```csharp
// Good: Use abstraction for extension

public interface IPaymentMethod
{
    void ProcessPayment(decimal amount);
}

public class CreditCardPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing ${amount} via Credit Card");
        // Credit card specific logic
    }
}

public class PayPalPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing ${amount} via PayPal");
        // PayPal specific logic
    }
}

public class BankTransferPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing ${amount} via Bank Transfer");
        // Bank transfer specific logic
    }
}

// New payment method - no modification to existing code!
public class CryptoPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Processing ${amount} via Cryptocurrency");
        // Crypto specific logic
    }
}

public class PaymentProcessor
{
    public void ProcessPayment(IPaymentMethod paymentMethod, decimal amount)
    {
        paymentMethod.ProcessPayment(amount);
    }
}

// Usage
var processor = new PaymentProcessor();
processor.ProcessPayment(new CreditCardPayment(), 100);
processor.ProcessPayment(new CryptoPayment(), 200); // New payment type!
```

### Real-World Example: Discount Calculator

```csharp
// ❌ Bad: Violates OCP
public class DiscountCalculator
{
    public decimal CalculateDiscount(string customerType, decimal amount)
    {
        if (customerType == "Regular")
            return amount * 0.05m;
        else if (customerType == "Premium")
            return amount * 0.10m;
        else if (customerType == "VIP")
            return amount * 0.20m;
        return 0;
    }
}

// ✅ Good: Follows OCP
public interface IDiscountStrategy
{
    decimal CalculateDiscount(decimal amount);
}

public class RegularCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount) => amount * 0.05m;
}

public class PremiumCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount) => amount * 0.10m;
}

public class VIPCustomerDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(decimal amount) => amount * 0.20m;
}

public class DiscountCalculator
{
    public decimal CalculateDiscount(IDiscountStrategy strategy, decimal amount)
    {
        return strategy.CalculateDiscount(amount);
    }
}
```

---

## Liskov Substitution Principle (LSP)

**Definition:** Objects of a superclass should be replaceable with objects of subclasses without breaking the application

### ❌ Violation

```csharp
// Bad: Square is-a Rectangle, but breaks LSP
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    
    public int GetArea() => Width * Height;
}

public class Square : Rectangle
{
    private int _side;
    
    public override int Width
    {
        get => _side;
        set => _side = value;
    }
    
    public override int Height
    {
        get => _side;
        set => _side = value;
    }
}

// Problem:
public void TestRectangle(Rectangle rectangle)
{
    rectangle.Width = 5;
    rectangle.Height = 10;
    Console.WriteLine(rectangle.GetArea()); // Expects 50
}

var rect = new Rectangle();
TestRectangle(rect); // Output: 50 ✅

var square = new Square();
TestRectangle(square); // Output: 100 ❌ (Expected 50, but got 100!)
// Square breaks the expected behavior of Rectangle
```

### ✅ Following LSP

```csharp
// Good: Proper abstraction
public abstract class Shape
{
    public abstract int GetArea();
}

public class Rectangle : Shape
{
    public int Width { get; set; }
    public int Height { get; set; }
    
    public override int GetArea() => Width * Height;
}

public class Square : Shape
{
    public int Side { get; set; }
    
    public override int GetArea() => Side * Side;
}

// Now both can be used interchangeably as Shape
public int CalculateTotalArea(List<Shape> shapes)
{
    return shapes.Sum(s => s.GetArea());
}
```

### Real-World Example: Bird Hierarchy

```csharp
// ❌ Bad: Violates LSP
public class Bird
{
    public virtual void Fly()
    {
        Console.WriteLine("Flying...");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotImplementedException("Penguins can't fly!");
    }
}

// Problem:
public void MakeBirdFly(Bird bird)
{
    bird.Fly(); // Crashes if bird is a Penguin!
}

// ✅ Good: Follows LSP
public abstract class Bird
{
    public abstract void Move();
}

public interface IFlyable
{
    void Fly();
}

public class Sparrow : Bird, IFlyable
{
    public override void Move() => Fly();
    
    public void Fly()
    {
        Console.WriteLine("Sparrow flying...");
    }
}

public class Penguin : Bird
{
    public override void Move() => Swim();
    
    public void Swim()
    {
        Console.WriteLine("Penguin swimming...");
    }
}

// Usage:
public void MakeMove(Bird bird)
{
    bird.Move(); // Works for all birds ✅
}

public void MakeFly(IFlyable flyable)
{
    flyable.Fly(); // Only for flying birds ✅
}
```

---

## Interface Segregation Principle (ISP)

**Definition:** Clients should not be forced to depend on interfaces they don't use

### ❌ Violation

```csharp
// Bad: Fat interface
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

public class HumanWorker : IWorker
{
    public void Work() => Console.WriteLine("Working...");
    public void Eat() => Console.WriteLine("Eating...");
    public void Sleep() => Console.WriteLine("Sleeping...");
}

public class RobotWorker : IWorker
{
    public void Work() => Console.WriteLine("Working...");
    
    // Robots don't eat or sleep!
    public void Eat() => throw new NotImplementedException();
    public void Sleep() => throw new NotImplementedException();
}
```

### ✅ Following ISP

```csharp
// Good: Segregated interfaces
public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public class HumanWorker : IWorkable, IEatable, ISleepable
{
    public void Work() => Console.WriteLine("Working...");
    public void Eat() => Console.WriteLine("Eating...");
    public void Sleep() => Console.WriteLine("Sleeping...");
}

public class RobotWorker : IWorkable
{
    public void Work() => Console.WriteLine("Working...");
}

// Now clients can depend on only what they need
public class WorkManager
{
    public void ManageWork(IWorkable worker)
    {
        worker.Work(); // Only needs Work method
    }
}
```

### Real-World Example: Printer Interfaces

```csharp
// ❌ Bad: All-in-one interface
public interface IMultiFunctionDevice
{
    void Print(Document document);
    void Scan(Document document);
    void Fax(Document document);
    void Copy(Document document);
}

public class SimplePrinter : IMultiFunctionDevice
{
    public void Print(Document document) => Console.WriteLine("Printing...");
    
    // Simple printer doesn't have these features!
    public void Scan(Document document) => throw new NotImplementedException();
    public void Fax(Document document) => throw new NotImplementedException();
    public void Copy(Document document) => throw new NotImplementedException();
}

// ✅ Good: Segregated interfaces
public interface IPrinter
{
    void Print(Document document);
}

public interface IScanner
{
    void Scan(Document document);
}

public interface IFax
{
    void Fax(Document document);
}

public class SimplePrinter : IPrinter
{
    public void Print(Document document) => Console.WriteLine("Printing...");
}

public class MultiFunctionPrinter : IPrinter, IScanner, IFax
{
    public void Print(Document document) => Console.WriteLine("Printing...");
    public void Scan(Document document) => Console.WriteLine("Scanning...");
    public void Fax(Document document) => Console.WriteLine("Faxing...");
}
```

---

## Dependency Inversion Principle (DIP)

**Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions.

### ❌ Violation

```csharp
// Bad: High-level depends on low-level concrete classes
public class EmailNotification
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}

public class OrderService
{
    private readonly EmailNotification _notification;
    
    public OrderService()
    {
        _notification = new EmailNotification(); // Tight coupling!
    }
    
    public void PlaceOrder(Order order)
    {
        // Process order
        _notification.Send("Order placed successfully");
    }
}

// Problems:
// - Can't easily switch to SMS notification
// - Hard to test (can't mock EmailNotification)
// - OrderService is tightly coupled to EmailNotification
```

### ✅ Following DIP

```csharp
// Good: Depend on abstractions
public interface INotificationService
{
    void Send(string message);
}

public class EmailNotification : INotificationService
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}

public class SmsNotification : INotificationService
{
    public void Send(string message)
    {
        Console.WriteLine($"SMS: {message}");
    }
}

public class PushNotification : INotificationService
{
    public void Send(string message)
    {
        Console.WriteLine($"Push: {message}");
    }
}

public class OrderService
{
    private readonly INotificationService _notification;
    
    // Dependency injection
    public OrderService(INotificationService notification)
    {
        _notification = notification;
    }
    
    public void PlaceOrder(Order order)
    {
        // Process order
        _notification.Send("Order placed successfully");
    }
}

// Usage with DI container:
// services.AddScoped<INotificationService, EmailNotification>();
// Or switch to: services.AddScoped<INotificationService, SmsNotification>();

// Benefits:
// - Easy to switch implementations
// - Easy to test (can mock INotificationService)
// - Loose coupling
```

### Real-World Example: Data Access

```csharp
// ❌ Bad: Direct database dependency
public class CustomerService
{
    private readonly SqlConnection _connection;
    
    public CustomerService()
    {
        _connection = new SqlConnection("connection string");
    }
    
    public Customer GetCustomer(int id)
    {
        // Direct SQL query
        _connection.Open();
        var command = new SqlCommand("SELECT * FROM Customers WHERE Id = @Id", _connection);
        command.Parameters.AddWithValue("@Id", id);
        // ... execute and map
        return null; // simplified
    }
}

// ✅ Good: Abstraction with repository pattern
public interface ICustomerRepository
{
    Customer GetById(int id);
    IEnumerable<Customer> GetAll();
    void Add(Customer customer);
    void Update(Customer customer);
    void Delete(int id);
}

public class SqlCustomerRepository : ICustomerRepository
{
    private readonly string _connectionString;
    
    public SqlCustomerRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public Customer GetById(int id)
    {
        using var connection = new SqlConnection(_connectionString);
        return connection.QueryFirstOrDefault<Customer>(
            "SELECT * FROM Customers WHERE Id = @Id",
            new { Id = id });
    }
    
    // ... implement other methods
    public IEnumerable<Customer> GetAll() => throw new NotImplementedException();
    public void Add(Customer customer) => throw new NotImplementedException();
    public void Update(Customer customer) => throw new NotImplementedException();
    public void Delete(int id) => throw new NotImplementedException();
}

public class CustomerService
{
    private readonly ICustomerRepository _repository;
    
    public CustomerService(ICustomerRepository repository)
    {
        _repository = repository;
    }
    
    public Customer GetCustomer(int id)
    {
        return _repository.GetById(id);
    }
}

// Can easily switch to different database or mock for testing
public class MongoCustomerRepository : ICustomerRepository
{
    public Customer GetById(int id)
    {
        // MongoDB implementation
        return null;
    }
    
    // ... other implementations
    public IEnumerable<Customer> GetAll() => throw new NotImplementedException();
    public void Add(Customer customer) => throw new NotImplementedException();
    public void Update(Customer customer) => throw new NotImplementedException();
    public void Delete(int id) => throw new NotImplementedException();
}
```

---

# Design Patterns

## Creational Patterns

Design patterns for object creation mechanisms

---

## Singleton Pattern

**Purpose:** Ensure a class has only one instance and provide global access to it

**When to use:**
- ✅ Need exactly one instance (configuration, logging, cache)
- ✅ Global access point required
- ✅ Lazy initialization needed

**When NOT to use:**
- ❌ Testing is difficult (global state)
- ❌ Can cause tight coupling
- ❌ Not thread-safe by default

### Implementation

```csharp
// Thread-safe Singleton (Lazy<T>)
public sealed class Singleton
{
    private static readonly Lazy<Singleton> _instance = 
        new Lazy<Singleton>(() => new Singleton());
    
    private Singleton()
    {
        // Private constructor prevents instantiation
    }
    
    public static Singleton Instance => _instance.Value;
    
    public void DoSomething()
    {
        Console.WriteLine("Singleton method called");
    }
}

// Usage
var instance1 = Singleton.Instance;
var instance2 = Singleton.Instance;
Console.WriteLine(instance1 == instance2); // True

// Alternative: Double-check locking
public sealed class SingletonDoubleCheck
{
    private static SingletonDoubleCheck? _instance;
    private static readonly object _lock = new object();
    
    private SingletonDoubleCheck() { }
    
    public static SingletonDoubleCheck Instance
    {
        get
        {
            if (_instance == null)
            {
                lock (_lock)
                {
                    if (_instance == null)
                    {
                        _instance = new SingletonDoubleCheck();
                    }
                }
            }
            return _instance;
        }
    }
}
```

### Real-World Example: Logger

```csharp
public sealed class Logger
{
    private static readonly Lazy<Logger> _instance = new Lazy<Logger>(() => new Logger());
    private readonly string _logFilePath;
    
    private Logger()
    {
        _logFilePath = "app.log";
    }
    
    public static Logger Instance => _instance.Value;
    
    public void Log(string message)
    {
        File.AppendAllText(_logFilePath, $"{DateTime.Now}: {message}\n");
    }
}

// Usage
Logger.Instance.Log("Application started");
Logger.Instance.Log("Processing order");
```

---

## Factory Pattern

**Purpose:** Create objects without specifying exact class

**When to use:**
- ✅ Object creation logic is complex
- ✅ Need to create different types based on input
- ✅ Want to hide creation logic from client

### Simple Factory

```csharp
public interface IPayment
{
    void Process(decimal amount);
}

public class CreditCardPayment : IPayment
{
    public void Process(decimal amount)
    {
        Console.WriteLine($"Processing ${amount} via Credit Card");
    }
}

public class PayPalPayment : IPayment
{
    public void Process(decimal amount)
    {
        Console.WriteLine($"Processing ${amount} via PayPal");
    }
}

public class BankTransferPayment : IPayment
{
    public void Process(decimal amount)
    {
        Console.WriteLine($"Processing ${amount} via Bank Transfer");
    }
}

// Factory
public class PaymentFactory
{
    public static IPayment CreatePayment(string type)
    {
        return type.ToLower() switch
        {
            "creditcard" => new CreditCardPayment(),
            "paypal" => new PayPalPayment(),
            "banktransfer" => new BankTransferPayment(),
            _ => throw new ArgumentException($"Unknown payment type: {type}")
        };
    }
}

// Usage
var payment = PaymentFactory.CreatePayment("creditcard");
payment.Process(100);
```

### Factory Method

```csharp
// Abstract creator
public abstract class NotificationCreator
{
    public abstract INotification CreateNotification();
    
    public void SendNotification(string message)
    {
        var notification = CreateNotification();
        notification.Send(message);
    }
}

public interface INotification
{
    void Send(string message);
}

public class EmailNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}

public class SmsNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine($"SMS: {message}");
    }
}

// Concrete creators
public class EmailNotificationCreator : NotificationCreator
{
    public override INotification CreateNotification()
    {
        return new EmailNotification();
    }
}

public class SmsNotificationCreator : NotificationCreator
{
    public override INotification CreateNotification()
    {
        return new SmsNotification();
    }
}

// Usage
NotificationCreator creator = new EmailNotificationCreator();
creator.SendNotification("Hello!");
```

---

## Abstract Factory Pattern

**Purpose:** Create families of related objects without specifying concrete classes

**When to use:**
- ✅ Need to create families of related objects
- ✅ Want to ensure objects from the same family are used together

### Implementation

```csharp
// Abstract products
public interface IButton
{
    void Render();
}

public interface ICheckbox
{
    void Render();
}

// Concrete products - Windows
public class WindowsButton : IButton
{
    public void Render() => Console.WriteLine("Rendering Windows button");
}

public class WindowsCheckbox : ICheckbox
{
    public void Render() => Console.WriteLine("Rendering Windows checkbox");
}

// Concrete products - Mac
public class MacButton : IButton
{
    public void Render() => Console.WriteLine("Rendering Mac button");
}

public class MacCheckbox : ICheckbox
{
    public void Render() => Console.WriteLine("Rendering Mac checkbox");
}

// Abstract factory
public interface IGUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

// Concrete factories
public class WindowsFactory : IGUIFactory
{
    public IButton CreateButton() => new WindowsButton();
    public ICheckbox CreateCheckbox() => new WindowsCheckbox();
}

public class MacFactory : IGUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ICheckbox CreateCheckbox() => new MacCheckbox();
}

// Client code
public class Application
{
    private readonly IButton _button;
    private readonly ICheckbox _checkbox;
    
    public Application(IGUIFactory factory)
    {
        _button = factory.CreateButton();
        _checkbox = factory.CreateCheckbox();
    }
    
    public void Render()
    {
        _button.Render();
        _checkbox.Render();
    }
}

// Usage
IGUIFactory factory = new WindowsFactory();
var app = new Application(factory);
app.Render();
```

---

## Builder Pattern

**Purpose:** Construct complex objects step by step

**When to use:**
- ✅ Object has many optional parameters
- ✅ Construction process is complex
- ✅ Want to create different representations

### Implementation

```csharp
// Product
public class Pizza
{
    public string? Size { get; set; }
    public string? Crust { get; set; }
    public List<string> Toppings { get; set; } = new();
    public bool ExtraCheese { get; set; }
    
    public override string ToString()
    {
        return $"{Size} pizza with {Crust} crust, " +
               $"Toppings: {string.Join(", ", Toppings)}, " +
               $"Extra cheese: {ExtraCheese}";
    }
}

// Builder
public class PizzaBuilder
{
    private readonly Pizza _pizza = new();
    
    public PizzaBuilder SetSize(string size)
    {
        _pizza.Size = size;
        return this;
    }
    
    public PizzaBuilder SetCrust(string crust)
    {
        _pizza.Crust = crust;
        return this;
    }
    
    public PizzaBuilder AddTopping(string topping)
    {
        _pizza.Toppings.Add(topping);
        return this;
    }
    
    public PizzaBuilder AddExtraCheese()
    {
        _pizza.ExtraCheese = true;
        return this;
    }
    
    public Pizza Build()
    {
        return _pizza;
    }
}

// Usage
var pizza = new PizzaBuilder()
    .SetSize("Large")
    .SetCrust("Thin")
    .AddTopping("Pepperoni")
    .AddTopping("Mushrooms")
    .AddTopping("Olives")
    .AddExtraCheese()
    .Build();

Console.WriteLine(pizza);
```

### Fluent Builder with Director

```csharp
// Director for common configurations
public class PizzaDirector
{
    public static Pizza CreateMargherita()
    {
        return new PizzaBuilder()
            .SetSize("Medium")
            .SetCrust("Regular")
            .AddTopping("Tomato sauce")
            .AddTopping("Mozzarella")
            .AddTopping("Basil")
            .Build();
    }
    
    public static Pizza CreatePepperoni()
    {
        return new PizzaBuilder()
            .SetSize("Large")
            .SetCrust("Thin")
            .AddTopping("Tomato sauce")
            .AddTopping("Mozzarella")
            .AddTopping("Pepperoni")
            .AddExtraCheese()
            .Build();
    }
}

// Usage
var margherita = PizzaDirector.CreateMargherita();
var pepperoni = PizzaDirector.CreatePepperoni();
```

---

## Prototype Pattern

**Purpose:** Create new objects by copying existing ones

**When to use:**
- ✅ Object creation is expensive
- ✅ Need to create many similar objects
- ✅ Want to avoid subclasses

### Implementation

```csharp
public interface IPrototype<T>
{
    T Clone();
}

public class Product : IPrototype<Product>
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public List<string> Features { get; set; } = new();
    
    public Product Clone()
    {
        // Shallow copy
        return (Product)MemberwiseClone();
        
        // Or deep copy if needed:
        // return new Product
        // {
        //     Name = this.Name,
        //     Price = this.Price,
        //     Features = new List<string>(this.Features)
        // };
    }
    
    public override string ToString()
    {
        return $"{Name} - ${Price} - Features: {string.Join(", ", Features)}";
    }
}

// Usage
var originalProduct = new Product
{
    Name = "Laptop",
    Price = 999,
    Features = new List<string> { "16GB RAM", "512GB SSD" }
};

var clonedProduct = originalProduct.Clone();
clonedProduct.Name = "Gaming Laptop";
clonedProduct.Price = 1499;
clonedProduct.Features.Add("RTX 4060");

Console.WriteLine(originalProduct);
Console.WriteLine(clonedProduct);
```

---

## Structural Patterns

Design patterns for composing classes and objects

---

## Adapter Pattern

**Purpose:** Convert interface of a class into another interface clients expect

**When to use:**
- ✅ Want to use existing class with incompatible interface
- ✅ Need to integrate third-party libraries
- ✅ Legacy code integration

### Implementation

```csharp
// Target interface (what client expects)
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount, string currency);
}

// Adaptee (existing incompatible class)
public class ThirdPartyPaymentGateway
{
    public void MakePayment(double amountInCents, string currencyCode)
    {
        Console.WriteLine($"Processing {amountInCents} cents in {currencyCode}");
    }
}

// Adapter
public class PaymentAdapter : IPaymentProcessor
{
    private readonly ThirdPartyPaymentGateway _gateway;
    
    public PaymentAdapter(ThirdPartyPaymentGateway gateway)
    {
        _gateway = gateway;
    }
    
    public void ProcessPayment(decimal amount, string currency)
    {
        // Convert dollars to cents
        double amountInCents = (double)(amount * 100);
        _gateway.MakePayment(amountInCents, currency);
    }
}

// Usage
var thirdPartyGateway = new ThirdPartyPaymentGateway();
IPaymentProcessor processor = new PaymentAdapter(thirdPartyGateway);
processor.ProcessPayment(99.99m, "USD");
```

---

## Decorator Pattern

**Purpose:** Add new functionality to existing objects dynamically

**When to use:**
- ✅ Need to add responsibilities dynamically
- ✅ Want to avoid subclass explosion
- ✅ Extension by composition rather than inheritance

### Implementation

```csharp
// Component
public interface ICoffee
{
    string GetDescription();
    decimal GetCost();
}

// Concrete component
public class SimpleCoffee : ICoffee
{
    public string GetDescription() => "Simple coffee";
    public decimal GetCost() => 2.00m;
}

// Base decorator
public abstract class CoffeeDecorator : ICoffee
{
    protected readonly ICoffee _coffee;
    
    protected CoffeeDecorator(ICoffee coffee)
    {
        _coffee = coffee;
    }
    
    public virtual string GetDescription() => _coffee.GetDescription();
    public virtual decimal GetCost() => _coffee.GetCost();
}

// Concrete decorators
public class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(ICoffee coffee) : base(coffee) { }
    
    public override string GetDescription() => _coffee.GetDescription() + ", Milk";
    public override decimal GetCost() => _coffee.GetCost() + 0.50m;
}

public class SugarDecorator : CoffeeDecorator
{
    public SugarDecorator(ICoffee coffee) : base(coffee) { }
    
    public override string GetDescription() => _coffee.GetDescription() + ", Sugar";
    public override decimal GetCost() => _coffee.GetCost() + 0.25m;
}

public class WhippedCreamDecorator : CoffeeDecorator
{
    public WhippedCreamDecorator(ICoffee coffee) : base(coffee) { }
    
    public override string GetDescription() => _coffee.GetDescription() + ", Whipped Cream";
    public override decimal GetCost() => _coffee.GetCost() + 0.75m;
}

// Usage
ICoffee coffee = new SimpleCoffee();
Console.WriteLine($"{coffee.GetDescription()} - ${coffee.GetCost()}");

coffee = new MilkDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} - ${coffee.GetCost()}");

coffee = new SugarDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} - ${coffee.GetCost()}");

coffee = new WhippedCreamDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} - ${coffee.GetCost()}");

// Output:
// Simple coffee - $2.00
// Simple coffee, Milk - $2.50
// Simple coffee, Milk, Sugar - $2.75
// Simple coffee, Milk, Sugar, Whipped Cream - $3.50
```

---

## Facade Pattern

**Purpose:** Provide simplified interface to complex subsystem

**When to use:**
- ✅ Complex subsystem with many classes
- ✅ Want to hide complexity from clients
- ✅ Need layer between client and subsystem

### Implementation

```csharp
// Complex subsystem classes
public class CPU
{
    public void Freeze() => Console.WriteLine("CPU: Freezing processor");
    public void Jump(long position) => Console.WriteLine($"CPU: Jumping to {position}");
    public void Execute() => Console.WriteLine("CPU: Executing instructions");
}

public class Memory
{
    public void Load(long position, byte[] data) => 
        Console.WriteLine($"Memory: Loading {data.Length} bytes at {position}");
}

public class HardDrive
{
    public byte[] Read(long lba, int size)
    {
        Console.WriteLine($"HardDrive: Reading {size} bytes from sector {lba}");
        return new byte[size];
    }
}

// Facade
public class ComputerFacade
{
    private readonly CPU _cpu;
    private readonly Memory _memory;
    private readonly HardDrive _hardDrive;
    
    public ComputerFacade()
    {
        _cpu = new CPU();
        _memory = new Memory();
        _hardDrive = new HardDrive();
    }
    
    public void Start()
    {
        Console.WriteLine("Starting computer...");
        _cpu.Freeze();
        _memory.Load(0, _hardDrive.Read(100, 1024));
        _cpu.Jump(0);
        _cpu.Execute();
        Console.WriteLine("Computer started successfully!");
    }
}

// Usage (simple interface)
var computer = new ComputerFacade();
computer.Start();
```

---

## Proxy Pattern

**Purpose:** Provide placeholder or surrogate for another object

**When to use:**
- ✅ Control access to object (protection proxy)
- ✅ Lazy initialization (virtual proxy)
- ✅ Remote objects (remote proxy)
- ✅ Caching (cache proxy)

### Virtual Proxy (Lazy Loading)

```csharp
public interface IImage
{
    void Display();
}

// Real object (expensive to create)
public class RealImage : IImage
{
    private readonly string _fileName;
    
    public RealImage(string fileName)
    {
        _fileName = fileName;
        LoadFromDisk();
    }
    
    private void LoadFromDisk()
    {
        Console.WriteLine($"Loading image: {_fileName}");
        Thread.Sleep(2000); // Simulate expensive operation
    }
    
    public void Display()
    {
        Console.WriteLine($"Displaying image: {_fileName}");
    }
}

// Proxy (lazy initialization)
public class ProxyImage : IImage
{
    private readonly string _fileName;
    private RealImage? _realImage;
    
    public ProxyImage(string fileName)
    {
        _fileName = fileName;
    }
    
    public void Display()
    {
        if (_realImage == null)
        {
            _realImage = new RealImage(_fileName);
        }
        _realImage.Display();
    }
}

// Usage
IImage image1 = new ProxyImage("photo1.jpg");
IImage image2 = new ProxyImage("photo2.jpg");

// Image not loaded yet
Console.WriteLine("Images created");

// Load and display (first time - slow)
image1.Display();

// Display again (already loaded - fast)
image1.Display();

// Output:
// Images created
// Loading image: photo1.jpg (2 second delay)
// Displaying image: photo1.jpg
// Displaying image: photo1.jpg (instant)
```

### Protection Proxy

```csharp
public interface IDocument
{
    void Display();
    void Edit(string content);
}

public class Document : IDocument
{
    private string _content;
    
    public Document(string content)
    {
        _content = content;
    }
    
    public void Display()
    {
        Console.WriteLine($"Document content: {_content}");
    }
    
    public void Edit(string content)
    {
        _content = content;
        Console.WriteLine("Document updated");
    }
}

public class ProtectedDocument : IDocument
{
    private readonly Document _document;
    private readonly string _userRole;
    
    public ProtectedDocument(Document document, string userRole)
    {
        _document = document;
        _userRole = userRole;
    }
    
    public void Display()
    {
        _document.Display();
    }
    
    public void Edit(string content)
    {
        if (_userRole == "Admin" || _userRole == "Editor")
        {
            _document.Edit(content);
        }
        else
        {
            Console.WriteLine("Access denied: You don't have permission to edit");
        }
    }
}

// Usage
var document = new Document("Original content");
IDocument protectedDoc = new ProtectedDocument(document, "Viewer");

protectedDoc.Display(); // ✅ Allowed
protectedDoc.Edit("New content"); // ❌ Access denied
```

---

## Behavioral Patterns

Design patterns for communication between objects

---

## Strategy Pattern

**Purpose:** Define family of algorithms and make them interchangeable

**When to use:**
- ✅ Multiple algorithms for specific task
- ✅ Want to avoid conditional statements
- ✅ Need to switch algorithms at runtime

### Implementation

```csharp
// Strategy interface
public interface ISortStrategy
{
    void Sort(List<int> list);
}

// Concrete strategies
public class QuickSort : ISortStrategy
{
    public void Sort(List<int> list)
    {
        Console.WriteLine("Sorting using Quick Sort");
        list.Sort(); // Simplified
    }
}

public class MergeSort : ISortStrategy
{
    public void Sort(List<int> list)
    {
        Console.WriteLine("Sorting using Merge Sort");
        list.Sort(); // Simplified
    }
}

public class BubbleSort : ISortStrategy
{
    public void Sort(List<int> list)
    {
        Console.WriteLine("Sorting using Bubble Sort");
        list.Sort(); // Simplified
    }
}

// Context
public class DataProcessor
{
    private ISortStrategy _sortStrategy;
    
    public DataProcessor(ISortStrategy sortStrategy)
    {
        _sortStrategy = sortStrategy;
    }
    
    public void SetStrategy(ISortStrategy sortStrategy)
    {
        _sortStrategy = sortStrategy;
    }
    
    public void ProcessData(List<int> data)
    {
        _sortStrategy.Sort(data);
        Console.WriteLine($"Sorted: {string.Join(", ", data)}");
    }
}

// Usage
var data = new List<int> { 5, 2, 8, 1, 9 };

var processor = new DataProcessor(new QuickSort());
processor.ProcessData(new List<int>(data));

processor.SetStrategy(new MergeSort());
processor.ProcessData(new List<int>(data));
```

### Real-World: Payment Processing

```csharp
public interface IPaymentStrategy
{
    void Pay(decimal amount);
}

public class CreditCardStrategy : IPaymentStrategy
{
    private readonly string _cardNumber;
    
    public CreditCardStrategy(string cardNumber)
    {
        _cardNumber = cardNumber;
    }
    
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} using Credit Card ending in {_cardNumber[^4..]}");
    }
}

public class PayPalStrategy : IPaymentStrategy
{
    private readonly string _email;
    
    public PayPalStrategy(string email)
    {
        _email = email;
    }
    
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} using PayPal account {_email}");
    }
}

public class ShoppingCart
{
    private IPaymentStrategy? _paymentStrategy;
    
    public void SetPaymentStrategy(IPaymentStrategy strategy)
    {
        _paymentStrategy = strategy;
    }
    
    public void Checkout(decimal amount)
    {
        if (_paymentStrategy == null)
            throw new InvalidOperationException("Payment strategy not set");
        
        _paymentStrategy.Pay(amount);
    }
}

// Usage
var cart = new ShoppingCart();

cart.SetPaymentStrategy(new CreditCardStrategy("1234567890123456"));
cart.Checkout(100.00m);

cart.SetPaymentStrategy(new PayPalStrategy("user@example.com"));
cart.Checkout(50.00m);
```

---

## Observer Pattern

**Purpose:** Define one-to-many dependency so when one object changes state, all dependents are notified

**When to use:**
- ✅ Object state change affects other objects
- ✅ Don't know how many objects need notification
- ✅ Event-driven systems

### Implementation

```csharp
// Subject (Observable)
public interface ISubject
{
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify();
}

// Observer
public interface IObserver
{
    void Update(ISubject subject);
}

// Concrete Subject
public class Stock : ISubject
{
    private readonly List<IObserver> _observers = new();
    private decimal _price;
    
    public string Symbol { get; }
    public decimal Price
    {
        get => _price;
        set
        {
            _price = value;
            Notify();
        }
    }
    
    public Stock(string symbol, decimal initialPrice)
    {
        Symbol = symbol;
        _price = initialPrice;
    }
    
    public void Attach(IObserver observer)
    {
        _observers.Add(observer);
    }
    
    public void Detach(IObserver observer)
    {
        _observers.Remove(observer);
    }
    
    public void Notify()
    {
        foreach (var observer in _observers)
        {
            observer.Update(this);
        }
    }
}

// Concrete Observers
public class StockDisplay : IObserver
{
    private readonly string _name;
    
    public StockDisplay(string name)
    {
        _name = name;
    }
    
    public void Update(ISubject subject)
    {
        if (subject is Stock stock)
        {
            Console.WriteLine($"{_name}: {stock.Symbol} is now ${stock.Price}");
        }
    }
}

public class StockAlert : IObserver
{
    private readonly decimal _threshold;
    
    public StockAlert(decimal threshold)
    {
        _threshold = threshold;
    }
    
    public void Update(ISubject subject)
    {
        if (subject is Stock stock && stock.Price > _threshold)
        {
            Console.WriteLine($"ALERT: {stock.Symbol} exceeded ${_threshold}!");
        }
    }
}

// Usage
var appleStock = new Stock("AAPL", 150.00m);

var display = new StockDisplay("Main Display");
var alert = new StockAlert(175.00m);

appleStock.Attach(display);
appleStock.Attach(alert);

appleStock.Price = 160.00m;
appleStock.Price = 180.00m;

// Output:
// Main Display: AAPL is now $160.00
// Main Display: AAPL is now $180.00
// ALERT: AAPL exceeded $175.00!
```

### Using C# Events

```csharp
public class Stock
{
    private decimal _price;
    
    public string Symbol { get; }
    public decimal Price
    {
        get => _price;
        set
        {
            _price = value;
            OnPriceChanged();
        }
    }
    
    // Event
    public event EventHandler<PriceChangedEventArgs>? PriceChanged;
    
    public Stock(string symbol, decimal initialPrice)
    {
        Symbol = symbol;
        _price = initialPrice;
    }
    
    protected virtual void OnPriceChanged()
    {
        PriceChanged?.Invoke(this, new PriceChangedEventArgs(Symbol, Price));
    }
}

public class PriceChangedEventArgs : EventArgs
{
    public string Symbol { get; }
    public decimal NewPrice { get; }
    
    public PriceChangedEventArgs(string symbol, decimal newPrice)
    {
        Symbol = symbol;
        NewPrice = newPrice;
    }
}

// Usage
var stock = new Stock("AAPL", 150.00m);

stock.PriceChanged += (sender, e) =>
{
    Console.WriteLine($"Display: {e.Symbol} is now ${e.NewPrice}");
};

stock.PriceChanged += (sender, e) =>
{
    if (e.NewPrice > 175)
        Console.WriteLine($"ALERT: {e.Symbol} exceeded $175!");
};

stock.Price = 160.00m;
stock.Price = 180.00m;
```

---

## Command Pattern

**Purpose:** Encapsulate request as object, allowing parameterization and queuing

**When to use:**
- ✅ Need to queue operations
- ✅ Support undo/redo
- ✅ Log operations
- ✅ Decouple sender and receiver

### Implementation

```csharp
// Command interface
public interface ICommand
{
    void Execute();
    void Undo();
}

// Receiver
public class Light
{
    public void TurnOn()
    {
        Console.WriteLine("Light is ON");
    }
    
    public void TurnOff()
    {
        Console.WriteLine("Light is OFF");
    }
}

// Concrete Commands
public class LightOnCommand : ICommand
{
    private readonly Light _light;
    
    public LightOnCommand(Light light)
    {
        _light = light;
    }
    
    public void Execute() => _light.TurnOn();
    public void Undo() => _light.TurnOff();
}

public class LightOffCommand : ICommand
{
    private readonly Light _light;
    
    public LightOffCommand(Light light)
    {
        _light = light;
    }
    
    public void Execute() => _light.TurnOff();
    public void Undo() => _light.TurnOn();
}

// Invoker
public class RemoteControl
{
    private readonly Stack<ICommand> _history = new();
    
    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        _history.Push(command);
    }
    
    public void Undo()
    {
        if (_history.Count > 0)
        {
            var command = _history.Pop();
            command.Undo();
        }
    }
}

// Usage
var light = new Light();
var remote = new RemoteControl();

remote.ExecuteCommand(new LightOnCommand(light));   // Light is ON
remote.ExecuteCommand(new LightOffCommand(light));  // Light is OFF
remote.Undo();                                      // Light is ON
remote.Undo();                                      // Light is OFF
```

---

## Template Method Pattern

**Purpose:** Define skeleton of algorithm, letting subclasses override specific steps

**When to use:**
- ✅ Common algorithm structure with varying steps
- ✅ Want to avoid code duplication
- ✅ Control extension points

### Implementation

```csharp
public abstract class DataProcessor
{
    // Template method
    public void Process()
    {
        ReadData();
        ProcessData();
        SaveData();
    }
    
    protected abstract void ReadData();
    protected abstract void ProcessData();
    protected abstract void SaveData();
}

public class CSVDataProcessor : DataProcessor
{
    protected override void ReadData()
    {
        Console.WriteLine("Reading data from CSV file");
    }
    
    protected override void ProcessData()
    {
        Console.WriteLine("Processing CSV data");
    }
    
    protected override void SaveData()
    {
        Console.WriteLine("Saving processed CSV data");
    }
}

public class JSONDataProcessor : DataProcessor
{
    protected override void ReadData()
    {
        Console.WriteLine("Reading data from JSON file");
    }
    
    protected override void ProcessData()
    {
        Console.WriteLine("Processing JSON data");
    }
    
    protected override void SaveData()
    {
        Console.WriteLine("Saving processed JSON data");
    }
}

// Usage
DataProcessor csvProcessor = new CSVDataProcessor();
csvProcessor.Process();

Console.WriteLine();

DataProcessor jsonProcessor = new JSONDataProcessor();
jsonProcessor.Process();
```

---

## Quick Reference: When to Use Which Pattern

### Creational Patterns
| Pattern | Use When |
|---------|----------|
| **Singleton** | Need exactly one instance (logger, config) |
| **Factory** | Create objects based on runtime conditions |
| **Abstract Factory** | Need families of related objects |
| **Builder** | Object has many optional parameters |
| **Prototype** | Object creation is expensive, need copies |

### Structural Patterns
| Pattern | Use When |
|---------|----------|
| **Adapter** | Integrate incompatible interfaces |
| **Decorator** | Add responsibilities dynamically |
| **Facade** | Simplify complex subsystem |
| **Proxy** | Control access or add lazy loading |

### Behavioral Patterns
| Pattern | Use When |
|---------|----------|
| **Strategy** | Multiple algorithms, choose at runtime |
| **Observer** | Object changes affect multiple objects |
| **Command** | Queue operations, support undo |
| **Template Method** | Algorithm structure with varying steps |

---

**Guide Complete!** This comprehensive guide covers all SOLID principles with C# examples, and the most important design patterns (Creational, Structural, and Behavioral). Master these to write maintainable, flexible, and testable code! 🎯
