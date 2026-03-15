# C# Unit Testing for Core Libraries Quick Reference

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                  TESTING PYRAMID FOR C# LIBRARIES                        │
│                                                                          │
│                      ▲  Integration / E2E                                │
│                     ▲▲▲  (Fewer, slower, more realistic)                │
│                   ▲▲▲▲▲▲▲  Unit Tests                                   │
│                 (Many, fast, isolated, deterministic)                     │
│                                                                          │
│  Tools: xUnit (test runner) + FluentAssertions + Moq                    │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 1. xUnit Basics

### Project Setup

```xml
<!-- In YourLib.Tests.csproj -->
<ItemGroup>
  <PackageReference Include="xunit"                        Version="2.9.*" />
  <PackageReference Include="xunit.runner.visualstudio"   Version="2.8.*" />
  <PackageReference Include="Microsoft.NET.Test.Sdk"      Version="17.*"  />
  <PackageReference Include="FluentAssertions"            Version="6.*"   />
  <PackageReference Include="Moq"                         Version="4.*"   />
</ItemGroup>
```

### Simplest Test

```csharp
using Xunit;
using FluentAssertions;

public class CalculatorTests
{
    [Fact]
    public void Add_TwoPositiveIntegers_ReturnsSum()
    {
        // Arrange
        var calculator = new Calculator();

        // Act
        int result = calculator.Add(2, 3);

        // Assert
        result.Should().Be(5);
    }
}
```

---

## 2. Arrange-Act-Assert (AAA) Pattern

```csharp
// Every test follows: Arrange → Act → Assert
[Fact]
public void Discount_AppliedToOrderOver100_Returns10Percent()
{
    // Arrange — set up the scenario
    var order = new Order { TotalBeforeDiscount = 150m };
    var pricer = new OrderPricer();

    // Act — execute the single behavior under test
    decimal finalPrice = pricer.ApplyDiscount(order);

    // Assert — verify the outcome
    finalPrice.Should().Be(135m);    // 150 - 10%
}

// One test → one behavior. If the test name has "and", split it.
```

---

## 3. Theory — Parameterized Tests

### InlineData

```csharp
[Theory]
[InlineData("hello",  "HELLO")]
[InlineData("World",  "WORLD")]
[InlineData("",       "")]
[InlineData(null,     null)]
public void ToUpper_Input_ReturnsExpected(string? input, string? expected)
{
    string? result = input?.ToUpper();
    result.Should().Be(expected);
}
```

### MemberData — Complex / Shared Data

```csharp
public static IEnumerable<object[]> InvalidEmailData =>
[
    [""],
    ["notanemail"],
    ["@nodomain.com"],
    ["user@"],
];

[Theory]
[MemberData(nameof(InvalidEmailData))]
public void Validate_InvalidEmail_ReturnsFalse(string email)
{
    bool valid = EmailValidator.IsValid(email);
    valid.Should().BeFalse();
}
```

### ClassData — Strongly Typed Data Source

```csharp
public class PriceTestData : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return [100m, 10m,  90m];   // price, discount, expected
        yield return [200m, 25m, 150m];
        yield return [0m,   10m,  0m];
    }
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

[Theory]
[ClassData(typeof(PriceTestData))]
public void ApplyDiscount_ReturnsCorrectPrice(decimal price, decimal pct, decimal expected)
{
    decimal result = PricingEngine.ApplyDiscount(price, pct);
    result.Should().Be(expected);
}
```

---

## 4. Testing Async Methods and Cancellation

```csharp
// Async tests — just mark as async Task (not async void!)
[Fact]
public async Task FetchUser_ValidId_ReturnsUser()
{
    // Arrange
    var repo = new InMemoryUserRepo();
    await repo.AddAsync(new User(Id: 1, Name: "Alice"));

    // Act
    User? user = await repo.FindByIdAsync(1);

    // Assert
    user.Should().NotBeNull();
    user!.Name.Should().Be("Alice");
}

// Testing cancellation
[Fact]
public async Task FetchUser_CancelledToken_ThrowsOperationCancelled()
{
    using var cts = new CancellationTokenSource();
    cts.Cancel();

    var repo = new UserRepository(/* dependencies */);
    Func<Task> act = () => repo.FindByIdAsync(1, cts.Token);

    await act.Should().ThrowAsync<OperationCanceledException>();
}

// Testing timeout
[Fact]
public async Task SlowOperation_ExceedsTimeout_Cancels()
{
    using var cts = new CancellationTokenSource(TimeSpan.FromMilliseconds(50));
    var service   = new SlowService();

    Func<Task> act = () => service.LongRunningAsync(cts.Token);
    await act.Should().ThrowAsync<OperationCanceledException>();
}
```

---

## 5. Mocking with Moq

```csharp
using Moq;

public class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _repoMock  = new();
    private readonly Mock<IEmailService>    _emailMock = new();
    private readonly OrderService           _sut;

    public OrderServiceTests()
        => _sut = new OrderService(_repoMock.Object, _emailMock.Object);

    [Fact]
    public async Task PlaceOrder_ValidOrder_SavesAndSendsEmail()
    {
        // Arrange
        var order = new Order { Id = 1, CustomerId = "C1" };
        _repoMock
            .Setup(r => r.SaveAsync(It.IsAny<Order>()))
            .Returns(Task.CompletedTask);

        // Act
        await _sut.PlaceOrderAsync(order);

        // Assert
        _repoMock.Verify(r => r.SaveAsync(It.Is<Order>(o => o.Id == 1)), Times.Once);
        _emailMock.Verify(e => e.SendAsync("C1", It.IsAny<string>()), Times.Once);
    }

    [Fact]
    public async Task PlaceOrder_RepositoryThrows_PropagatesException()
    {
        // Arrange
        _repoMock
            .Setup(r => r.SaveAsync(It.IsAny<Order>()))
            .ThrowsAsync(new DbException("Connection lost"));

        // Act & Assert
        await _sut.Invoking(s => s.PlaceOrderAsync(new Order()))
                  .Should().ThrowAsync<DbException>();
    }
}
```

### Setup Return Values

```csharp
// Return a value
_repoMock
    .Setup(r => r.FindByIdAsync(1))
    .ReturnsAsync(new User { Id = 1 });

// Return based on argument
_repoMock
    .Setup(r => r.FindByIdAsync(It.IsAny<int>()))
    .ReturnsAsync((int id) => new User { Id = id });

// Return different values on successive calls
_serviceMock
    .SetupSequence(s => s.GetNext())
    .Returns(1)
    .Returns(2)
    .Returns(3);

// Throw on call
_repoMock
    .Setup(r => r.DeleteAsync(It.IsAny<int>()))
    .ThrowsAsync(new Exception("fail"));

// Verify a method was never called
_emailMock.Verify(e => e.SendAsync(It.IsAny<string>(), It.IsAny<string>()), Times.Never);

// Verify exact call count
_repoMock.Verify(r => r.SaveAsync(It.IsAny<Order>()), Times.Exactly(2));
```

---

## 6. Fake Objects vs Mocks

```csharp
// Prefer Fakes for complex collaborators — simpler, faster, more readable

// Fake repository (in-memory)
public class FakeUserRepository : IUserRepository
{
    private readonly Dictionary<int, User> _store = new();

    public Task<User?> FindByIdAsync(int id, CancellationToken ct = default)
        => Task.FromResult(_store.GetValueOrDefault(id));

    public Task AddAsync(User user, CancellationToken ct = default)
    {
        _store[user.Id] = user;
        return Task.CompletedTask;
    }
}

// Use mocks for: verifying interactions (was method called?)
// Use fakes for: state-based testing (what did the repository end up containing?)
```

---

## 7. FluentAssertions Common Patterns

```csharp
// Primitives
result.Should().Be(42);
result.Should().NotBe(0);
result.Should().BeGreaterThan(0);
result.Should().BeInRange(1, 100);

// Strings
str.Should().Be("hello");
str.Should().StartWith("he");
str.Should().Contain("ello");
str.Should().MatchRegex(@"^\d{4}$");
str.Should().BeNullOrEmpty();

// Collections
list.Should().HaveCount(3);
list.Should().Contain(42);
list.Should().ContainInOrder(1, 2, 3);
list.Should().BeEquivalentTo(expected);         // order-insensitive deep equality
list.Should().AllSatisfy(x => x.IsActive.Should().BeTrue());

// Objects
obj.Should().NotBeNull();
obj.Should().BeEquivalentTo(expected, opts => opts.Excluding(x => x.CreatedAt));
obj.Should().BeOfType<OrderDto>();

// Exceptions
Func<int> act = () => parser.Parse("bad");
act.Should().Throw<FormatException>()
   .WithMessage("*invalid*");

// Async exceptions
Func<Task> asyncAct = () => service.DoAsync();
await asyncAct.Should().ThrowAsync<InvalidOperationException>();

// Numeric with tolerance
result.Should().BeApproximately(3.14, precision: 0.01);
```

---

## 8. Test Organization and Naming

```csharp
// File naming: [SubjectClass]Tests.cs
// Class naming: public class OrderServiceTests
// Method naming: Method_Scenario_ExpectedOutcome

// ✅ Good names
[Fact] public void Add_NegativeNumbers_ReturnsNegativeSum() { }
[Fact] public void FindById_NonExistentId_ReturnsNull() { }
[Fact] public void ApplyDiscount_ZeroPercent_ReturnOriginalPrice() { }

// ❌ Bad names
[Fact] public void Test1() { }
[Fact] public void AddTest() { }
[Fact] public void Works() { }

// Group related tests in nested classes
public class OrderServiceTests
{
    public class PlaceOrderTests
    {
        [Fact] public void PlaceOrder_Valid_Succeeds() { }
        [Fact] public void PlaceOrder_NullOrder_Throws() { }
    }

    public class CancelOrderTests
    {
        [Fact] public void CancelOrder_Pending_Succeeds() { }
    }
}
```

---

## 9. Property-Based Testing

```csharp
// Install: FsCheck.Xunit (F# Quickcheck port) or CsCheck
using FsCheck;
using FsCheck.Xunit;

// Property: reverse of reverse equals original
[Property]
public bool Reverse_Twice_IsOriginal(int[] arr)
{
    var reversed = arr.Reverse().Reverse().ToArray();
    return arr.SequenceEqual(reversed);
}

// Property with custom generator
[Property(Arbitrary = new[] { typeof(PositiveIntArb) })]
public bool Sort_IsIdempotent(int[] arr)
{
    var sorted1 = arr.OrderBy(x => x).ToArray();
    var sorted2 = sorted1.OrderBy(x => x).ToArray();
    return sorted1.SequenceEqual(sorted2);
}

public static class PositiveIntArb
{
    public static Arbitrary<int[]> PositiveInts()
        => Arb.Default.Array<int>()
              .Filter(arr => arr.All(x => x > 0));
}
```

---

## 10. Test Maintainability Checklist

```
✅ One assertion concept per test (multiple Should() for the same thing is fine)
✅ Test name tells the story: Method_Scenario_ExpectedResult
✅ No logic in tests (no if/else, loops, try/catch in Arrange/Act)
✅ Use builder / factory methods for complex test data setup
✅ Use [Fact(Skip = "reason")] instead of commenting out tests
✅ Test edge cases: null, empty, min, max, boundary values
✅ Tests run in isolation — no shared mutable state between tests
✅ Fast tests: avoid Thread.Sleep, use fake time providers

❌ Don't test framework code or third-party libraries
❌ Don't assert on multiple unrelated behaviors in one test
❌ Don't write fragile tests that check internal implementation details
❌ Don't use async void in tests — use async Task
```

### Test Data Builder Pattern

```csharp
// Builder for complex test objects — keeps tests readable
class OrderBuilder
{
    private int     _id         = 1;
    private string  _customerId = "CUST-01";
    private decimal _total      = 100m;

    public OrderBuilder WithId(int id)               { _id = id; return this; }
    public OrderBuilder WithCustomer(string id)      { _customerId = id; return this; }
    public OrderBuilder WithTotal(decimal total)     { _total = total; return this; }
    public Order        Build() => new Order { Id = _id, CustomerId = _customerId, Total = _total };
}

// Usage in test
var order = new OrderBuilder()
    .WithId(42)
    .WithTotal(250m)
    .Build();
```

---

## Anti-Patterns

```
❌ Testing private methods — test through the public API
❌ Mocking concrete classes — mock interfaces / abstractions
❌ Tests that depend on execution order — xUnit doesn't guarantee order
❌ One giant test class with thousands of lines
❌ System.DateTime.Now in production code — inject IDateTimeProvider
❌ File system or database calls in unit tests — use fakes/mocks
❌ Catching exceptions in tests with try/catch — use .Should().Throw()
❌ Asserting on ToString() output — fragile
```

---

## Quick Reference Summary

| Task               | API                                          |
| ------------------ | -------------------------------------------- |
| Simple test        | `[Fact]`                                     |
| Parameterized test | `[Theory]` + `[InlineData]` / `[MemberData]` |
| Async test         | `async Task` return type                     |
| Mock               | `new Mock<IInterface>()` + `.Object` (Moq)   |
| Verify call        | `.Verify(x => x.Method(...), Times.Once)`    |
| Assert value       | `.Should().Be(x)`                            |
| Assert collection  | `.Should().BeEquivalentTo(x)`                |
| Assert exception   | `.Should().Throw<T>()` / `.ThrowAsync<T>()`  |
| Assert approximate | `.Should().BeApproximately(x, delta)`        |
| Property test      | `[Property]` (FsCheck.Xunit)                 |

---

**Guide Complete!** Write tests that describe behavior, not implementation. Keep them fast, isolated, and named like documentation — because they are. 📘
