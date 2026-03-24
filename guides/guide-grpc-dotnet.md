# gRPC Quick Reference

---

## What is gRPC?

**gRPC** = Google Remote Procedure Call — a high-performance, contract-first RPC framework

**gRPC** lets you call methods on a remote server as if they were local function calls — with type safety, streaming, and automatic code generation.

**Key Features:**
- ✅ **Contract-first** - Define API in `.proto` files
- ✅ **High performance** - Binary serialization (Protobuf), HTTP/2
- ✅ **Code generation** - Client and server stubs auto-generated
- ✅ **Streaming** - Server, client, and bidirectional streaming
- ✅ **Strongly typed** - Compile-time API contract
- ✅ **Language agnostic** - .NET, Java, Go, Python, JS clients from same `.proto`

### gRPC vs REST vs SignalR

| Feature | REST | gRPC | SignalR |
|---|---|---|---|
| **Protocol** | HTTP/1.1 + JSON | HTTP/2 + Protobuf | HTTP/1.1 or 2 + JSON |
| **Contract** | OpenAPI (optional) | `.proto` (required) | Hub interface |
| **Performance** | Good | Excellent | Good |
| **Streaming** | Limited (SSE) | ✅ Full | ✅ Full |
| **Browser support** | ✅ Native | ⚠️ Needs gRPC-Web | ✅ Native |
| **Code gen** | Optional | ✅ Built-in | ❌ Manual |
| **Human readable** | ✅ JSON | ❌ Binary | ✅ JSON |
| **Best for** | Public APIs, mobile | Microservices, internal | Real-time UIs, chat |

### When to Use gRPC

```
✅ Service-to-service communication (microservices)
✅ High-throughput internal APIs (low latency required)
✅ Streaming large datasets
✅ Polyglot environments (different languages calling same service)
✅ You want a strict API contract

❌ Public-facing browser APIs (use REST instead)
❌ Simple CRUD APIs with no perf requirements
❌ When teams prefer human-readable payloads for debugging
```

---

## Protocol Buffers (Protobuf)

**Protobuf** = gRPC's interface definition language and binary serialization format

### Basic .proto File

```protobuf
// products.proto
syntax = "proto3";                    // Always proto3 for new projects

option csharp_namespace = "MyApp.Grpc";   // Generated C# namespace

// Import for well-known types
import "google/protobuf/timestamp.proto";
import "google/protobuf/empty.proto";

package products;                     // Package name (avoids name conflicts)

// ─── Messages ─────────────────────────────────────────────────────────

message Product {
    int32 id = 1;                     // Field number (1-15 = 1 byte, prefer for common fields)
    string name = 2;
    string description = 3;
    double price = 4;
    int32 category_id = 5;
    bool is_active = 6;
    google.protobuf.Timestamp created_at = 7;
    repeated string tags = 8;         // repeated = array/list
    ProductStatus status = 9;         // Enum
}

message CreateProductRequest {
    string name = 1;
    string description = 2;
    double price = 3;
    int32 category_id = 4;
}

message GetProductRequest {
    int32 id = 1;
}

message ListProductsRequest {
    int32 page = 1;
    int32 page_size = 2;
    string category = 3;             // optional in proto3 (default: empty string)
}

message ListProductsResponse {
    repeated Product products = 1;
    int32 total_count = 2;
}

// ─── Enums ─────────────────────────────────────────────────────────────

enum ProductStatus {
    PRODUCT_STATUS_UNSPECIFIED = 0;  // ✅ Always define 0 as UNSPECIFIED
    PRODUCT_STATUS_ACTIVE = 1;
    PRODUCT_STATUS_INACTIVE = 2;
    PRODUCT_STATUS_DISCONTINUED = 3;
}

// ─── Service ───────────────────────────────────────────────────────────

service ProductService {
    // Unary: one request → one response
    rpc GetProduct (GetProductRequest) returns (Product);

    // Unary with empty response
    rpc DeleteProduct (GetProductRequest) returns (google.protobuf.Empty);

    // Server streaming: one request → stream of responses
    rpc ListProducts (ListProductsRequest) returns (stream Product);

    // Client streaming: stream of requests → one response
    rpc BulkCreateProducts (stream CreateProductRequest) returns (ListProductsResponse);

    // Bidirectional streaming: stream of requests → stream of responses
    rpc SyncProducts (stream Product) returns (stream Product);
}
```

### Protobuf Scalar Types

```protobuf
// Protobuf type → C# type
double      → double
float       → float
int32       → int
int64       → long
uint32      → uint
uint64      → ulong
sint32      → int    (for negative numbers, more efficient than int32)
sint64      → long
bool        → bool
string      → string
bytes       → ByteString

// ✅ Use sint32/sint64 when values are likely negative (more efficient)
// ✅ Use string for text, bytes for binary data
// ✅ Field numbers 1-15 use 1 byte (use for frequently sent fields)
// ✅ Field numbers 16-2047 use 2 bytes
```

---

## .NET gRPC Project Setup

### Create gRPC Server

```bash
# Create new gRPC project
dotnet new grpc -n MyApp.GrpcService

# Or add to existing project
dotnet add package Grpc.AspNetCore
```

### .csproj Configuration

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <!-- ✅ Server-side: GrpcServices="Server" generates service base class -->
    <Protobuf Include="Protos\products.proto" GrpcServices="Server" />
    <Protobuf Include="Protos\orders.proto" GrpcServices="Server" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Grpc.AspNetCore" Version="2.62.0" />
    <PackageReference Include="Google.Protobuf" Version="3.27.0" />
    <PackageReference Include="Grpc.Tools" Version="2.62.0" PrivateAssets="All" />
  </ItemGroup>

</Project>
```

### Program.cs

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add gRPC
builder.Services.AddGrpc(options =>
{
    options.EnableDetailedErrors = builder.Environment.IsDevelopment();
    options.MaxReceiveMessageSize = 4 * 1024 * 1024; // 4MB
    options.MaxSendMessageSize = 4 * 1024 * 1024;    // 4MB
});

// Add gRPC reflection (allows tools like grpcurl to explore your API)
builder.Services.AddGrpcReflection();

// Register your DI services
builder.Services.AddScoped<IProductRepository, ProductRepository>();

var app = builder.Build();

// Map gRPC services
app.MapGrpcService<ProductGrpcService>();

if (app.Environment.IsDevelopment())
    app.MapGrpcReflectionService(); // Enable gRPC reflection in dev

// Health check endpoint
app.MapGet("/", () => "gRPC server running. Use a gRPC client to connect.");

app.Run();
```

---

## Implementing gRPC Services

### Unary Service (One Request → One Response)

```csharp
// ProductGrpcService.cs
using Grpc.Core;
using MyApp.Grpc;          // Generated namespace from .proto

public class ProductGrpcService : ProductService.ProductServiceBase
{
    private readonly IProductRepository _repository;
    private readonly ILogger<ProductGrpcService> _logger;

    public ProductGrpcService(
        IProductRepository repository,
        ILogger<ProductGrpcService> logger)
    {
        _repository = repository;
        _logger = logger;
    }

    // ─── Unary RPC ────────────────────────────────────────────────────
    public override async Task<Product> GetProduct(
        GetProductRequest request,
        ServerCallContext context)
    {
        _logger.LogInformation("Getting product {Id}", request.Id);

        var product = await _repository.GetByIdAsync(request.Id);

        if (product is null)
        {
            // ✅ Use gRPC status codes (not HTTP status codes)
            throw new RpcException(
                new Status(StatusCode.NotFound, $"Product {request.Id} not found"));
        }

        return MapToProto(product);
    }

    public override async Task<Google.Protobuf.WellKnownTypes.Empty> DeleteProduct(
        GetProductRequest request,
        ServerCallContext context)
    {
        await _repository.DeleteAsync(request.Id);
        return new Google.Protobuf.WellKnownTypes.Empty();
    }

    // ─── Mapping helper ───────────────────────────────────────────────
    private static Product MapToProto(ProductEntity entity) => new Product
    {
        Id = entity.Id,
        Name = entity.Name,
        Description = entity.Description ?? string.Empty,
        Price = (double)entity.Price,
        CategoryId = entity.CategoryId,
        IsActive = entity.IsActive,
        CreatedAt = Google.Protobuf.WellKnownTypes.Timestamp.FromDateTime(
            entity.CreatedAt.ToUniversalTime())
    };
}
```

### Server Streaming (One Request → Stream of Responses)

```csharp
// ─── Server Streaming ─────────────────────────────────────────────────
public override async Task ListProducts(
    ListProductsRequest request,
    IServerStreamWriter<Product> responseStream,
    ServerCallContext context)
{
    _logger.LogInformation("Streaming products, page {Page}", request.Page);

    var products = _repository.GetProductsAsAsyncEnumerable(request.PageSize);

    await foreach (var product in products)
    {
        // Check if client cancelled
        if (context.CancellationToken.IsCancellationRequested)
            break;

        await responseStream.WriteAsync(MapToProto(product));

        // Optional: small delay to control throughput
        await Task.Delay(10, context.CancellationToken);
    }
}
```

### Client Streaming (Stream of Requests → One Response)

```csharp
// ─── Client Streaming ─────────────────────────────────────────────────
public override async Task<ListProductsResponse> BulkCreateProducts(
    IAsyncStreamReader<CreateProductRequest> requestStream,
    ServerCallContext context)
{
    var createdProducts = new List<Product>();

    await foreach (var request in requestStream.ReadAllAsync())
    {
        var entity = await _repository.CreateAsync(new ProductEntity
        {
            Name = request.Name,
            Description = request.Description,
            Price = (decimal)request.Price,
            CategoryId = request.CategoryId
        });

        createdProducts.Add(MapToProto(entity));
    }

    return new ListProductsResponse
    {
        Products = { createdProducts },
        TotalCount = createdProducts.Count
    };
}
```

### Bidirectional Streaming

```csharp
// ─── Bidirectional Streaming ──────────────────────────────────────────
public override async Task SyncProducts(
    IAsyncStreamReader<Product> requestStream,
    IServerStreamWriter<Product> responseStream,
    ServerCallContext context)
{
    await foreach (var incomingProduct in requestStream.ReadAllAsync())
    {
        if (context.CancellationToken.IsCancellationRequested)
            break;

        // Process incoming product
        var result = await _repository.UpsertAsync(MapFromProto(incomingProduct));

        // Stream back the processed result
        await responseStream.WriteAsync(MapToProto(result));
    }
}
```

---

## gRPC Client

### Client Project Setup

```xml
<!-- Client.csproj -->
<ItemGroup>
    <!-- Client-side: GrpcServices="Client" generates client stub -->
    <Protobuf Include="Protos\products.proto" GrpcServices="Client" />
</ItemGroup>

<ItemGroup>
    <PackageReference Include="Google.Protobuf" Version="3.27.0" />
    <PackageReference Include="Grpc.Net.Client" Version="2.62.0" />
    <PackageReference Include="Grpc.Tools" Version="2.62.0" PrivateAssets="All" />
</ItemGroup>
```

### Creating and Using a Client

```csharp
// ─── Simple client (console/test) ────────────────────────────────────
using var channel = GrpcChannel.ForAddress("https://localhost:7001");
var client = new ProductService.ProductServiceClient(channel);

// Unary call
var product = await client.GetProductAsync(new GetProductRequest { Id = 1 });
Console.WriteLine($"Product: {product.Name} - ${product.Price}");

// ─── ASP.NET Core with typed HttpClient pattern ───────────────────────
// Program.cs
builder.Services.AddGrpcClient<ProductService.ProductServiceClient>(options =>
{
    options.Address = new Uri(builder.Configuration["GrpcEndpoints:ProductService"]!);
})
.ConfigureChannel(options =>
{
    // Configure credentials
    options.Credentials = ChannelCredentials.SecureSsl;
})
.AddCallCredentials(async (context, metadata) =>
{
    // Attach JWT token to every call
    var token = await tokenService.GetTokenAsync();
    metadata.Add("Authorization", $"Bearer {token}");
});

// Inject and use
public class ProductApiService
{
    private readonly ProductService.ProductServiceClient _client;

    public ProductApiService(ProductService.ProductServiceClient client)
    {
        _client = client;
    }

    public async Task<ProductDto?> GetProductAsync(int id)
    {
        try
        {
            var response = await _client.GetProductAsync(
                new GetProductRequest { Id = id });
            return MapToDto(response);
        }
        catch (RpcException ex) when (ex.StatusCode == StatusCode.NotFound)
        {
            return null;
        }
    }
}
```

### Client Streaming

```csharp
// ─── Server streaming client ──────────────────────────────────────────
using var call = client.ListProducts(new ListProductsRequest { PageSize = 100 });

await foreach (var product in call.ResponseStream.ReadAllAsync())
{
    Console.WriteLine($"Received: {product.Name}");
}

// ─── Client streaming ─────────────────────────────────────────────────
using var call = client.BulkCreateProducts();

foreach (var product in productsToCreate)
{
    await call.RequestStream.WriteAsync(new CreateProductRequest
    {
        Name = product.Name,
        Price = (double)product.Price
    });
}

await call.RequestStream.CompleteAsync(); // Signal done sending
var result = await call.ResponseAsync;   // Get final response
Console.WriteLine($"Created {result.TotalCount} products");

// ─── Bidirectional streaming ──────────────────────────────────────────
using var call = client.SyncProducts();

// Read and write concurrently
var readTask = Task.Run(async () =>
{
    await foreach (var response in call.ResponseStream.ReadAllAsync())
        Console.WriteLine($"Synced: {response.Name}");
});

foreach (var product in productsToSync)
    await call.RequestStream.WriteAsync(MapToProto(product));

await call.RequestStream.CompleteAsync();
await readTask;
```

---

## Error Handling

### gRPC Status Codes

```csharp
// Server — throw RpcException with appropriate status code
public override async Task<Product> GetProduct(
    GetProductRequest request, ServerCallContext context)
{
    if (request.Id <= 0)
        throw new RpcException(new Status(StatusCode.InvalidArgument,
            "Product ID must be positive"));

    var product = await _repository.GetByIdAsync(request.Id);

    if (product is null)
        throw new RpcException(new Status(StatusCode.NotFound,
            $"Product with ID {request.Id} was not found"));

    if (!await _authService.CanAccessProduct(context.GetHttpContext().User, request.Id))
        throw new RpcException(new Status(StatusCode.PermissionDenied,
            "You don't have access to this product"));

    return MapToProto(product);
}

// Client — catch RpcException
try
{
    var product = await client.GetProductAsync(new GetProductRequest { Id = productId });
}
catch (RpcException ex) when (ex.StatusCode == StatusCode.NotFound)
{
    Console.WriteLine($"Not found: {ex.Status.Detail}");
}
catch (RpcException ex) when (ex.StatusCode == StatusCode.Unauthenticated)
{
    await RefreshTokenAsync();
}
catch (RpcException ex)
{
    Console.WriteLine($"gRPC error: {ex.StatusCode} - {ex.Status.Detail}");
}
```

### Common Status Codes

```
OK                  ← Success
INVALID_ARGUMENT    ← Bad request (client error)
NOT_FOUND           ← Resource doesn't exist
ALREADY_EXISTS      ← Resource already exists
PERMISSION_DENIED   ← Auth OK, but not authorized
UNAUTHENTICATED     ← Not authenticated
RESOURCE_EXHAUSTED  ← Rate limited / quota exceeded
INTERNAL            ← Server error
UNAVAILABLE         ← Service down (retry-able)
DEADLINE_EXCEEDED   ← Timeout
CANCELLED           ← Request cancelled by client
```

---

## Interceptors (Middleware Equivalent)

### Server Interceptor

```csharp
// Logging interceptor
public class LoggingInterceptor : Interceptor
{
    private readonly ILogger<LoggingInterceptor> _logger;

    public LoggingInterceptor(ILogger<LoggingInterceptor> logger)
    {
        _logger = logger;
    }

    // Intercept unary calls
    public override async Task<TResponse> UnaryServerHandler<TRequest, TResponse>(
        TRequest request,
        ServerCallContext context,
        UnaryServerMethod<TRequest, TResponse> continuation)
    {
        var method = context.Method;
        _logger.LogInformation("Handling {Method}", method);

        var stopwatch = Stopwatch.StartNew();
        try
        {
            var response = await continuation(request, context);
            _logger.LogInformation("{Method} completed in {Elapsed}ms",
                method, stopwatch.ElapsedMilliseconds);
            return response;
        }
        catch (RpcException ex)
        {
            _logger.LogWarning("{Method} failed: {StatusCode} - {Detail}",
                method, ex.StatusCode, ex.Status.Detail);
            throw;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "{Method} threw unexpected exception", method);
            throw new RpcException(new Status(StatusCode.Internal, "Internal error"));
        }
    }
}

// Register interceptor
builder.Services.AddGrpc(options =>
{
    options.Interceptors.Add<LoggingInterceptor>();
});
// Or per-service:
app.MapGrpcService<ProductGrpcService>()
   .AddInterceptor<LoggingInterceptor>();
```

---

## Authentication

```csharp
// Program.cs - add auth to gRPC
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* JWT config */ });

builder.Services.AddAuthorization();

builder.Services.AddGrpc();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
app.MapGrpcService<ProductGrpcService>();

// Protect service or methods
[Authorize]
public class ProductGrpcService : ProductService.ProductServiceBase
{
    [AllowAnonymous]  // Override Authorize on this method
    public override Task<Product> GetProduct(...) { }

    [Authorize(Roles = "Admin")]
    public override Task<Google.Protobuf.WellKnownTypes.Empty> DeleteProduct(...) { }
}

// Access user in handler
public override async Task<Product> GetProduct(
    GetProductRequest request, ServerCallContext context)
{
    var user = context.GetHttpContext().User;
    var userId = user.FindFirst(ClaimTypes.NameIdentifier)?.Value;
}
```

---

## gRPC-Web (Browser Support)

Browsers can't call gRPC directly. gRPC-Web adds a compatibility layer.

```bash
dotnet add package Grpc.AspNetCore.Web
```

```csharp
// Program.cs
builder.Services.AddGrpc();

var app = builder.Build();

app.UseGrpcWeb(new GrpcWebOptions { DefaultEnabled = true }); // Enable for all

app.MapGrpcService<ProductGrpcService>().EnableGrpcWeb(); // Or per-service

app.Run();
```

```javascript
// Browser JavaScript client
import { GrpcWebFetchTransport } from "@protobuf-ts/grpcweb-transport";
import { ProductServiceClient } from "./generated/products.client";

const transport = new GrpcWebFetchTransport({ baseUrl: "https://localhost:7001" });
const client = new ProductServiceClient(transport);

const { response } = await client.getProduct({ id: 1 });
console.log(response.name);
```

---

## Best Practices

### 1. Organize .proto Files Well

```
MyApp.Grpc/
├── Protos/
│   ├── products.proto
│   ├── orders.proto
│   └── common.proto      ← Shared messages (PagedRequest, etc.)
```

### 2. Version Your Proto Files

```protobuf
// ✅ Never remove or renumber fields — clients will break
// ✅ Add new fields with new numbers (old clients ignore them)
// ✅ Deprecate fields with a comment, don't delete

message Product {
    int32 id = 1;
    string name = 2;
    // deprecated: use category_id instead
    string category_name = 3 [deprecated = true];
    int32 category_id = 4;  // New field - old clients ignore this
}
```

### 3. Always Set Deadlines on Client

```csharp
// ✅ Set deadline to prevent hanging calls
var deadline = DateTime.UtcNow.AddSeconds(30);
var product = await client.GetProductAsync(
    new GetProductRequest { Id = 1 },
    deadline: deadline);
```

### 4. Use Metadata for Cross-Cutting Concerns

```csharp
// Client: send metadata (headers)
var headers = new Metadata
{
    { "x-correlation-id", correlationId },
    { "x-tenant-id", tenantId }
};
var product = await client.GetProductAsync(request, headers);

// Server: read metadata
var correlationId = context.RequestHeaders
    .FirstOrDefault(h => h.Key == "x-correlation-id")?.Value;
```

### 5. Use reflection for dev/debugging only

```csharp
// ✅ Disable reflection in production
if (app.Environment.IsDevelopment())
    app.MapGrpcReflectionService();
```

---

## Quick Reference

```protobuf
// ─── .proto template ─────────────────────────────────────────────────
syntax = "proto3";
option csharp_namespace = "MyApp.Grpc";
import "google/protobuf/empty.proto";
import "google/protobuf/timestamp.proto";

message MyRequest { int32 id = 1; string name = 2; }
message MyResponse { int32 id = 1; string result = 2; }

service MyService {
    rpc GetItem (MyRequest) returns (MyResponse);               // Unary
    rpc ListItems (MyRequest) returns (stream MyResponse);      // Server stream
    rpc CreateItems (stream MyRequest) returns (MyResponse);    // Client stream
    rpc Sync (stream MyRequest) returns (stream MyResponse);    // Bidi stream
}
```

```csharp
// ─── Server ──────────────────────────────────────────────────────────
builder.Services.AddGrpc();
app.MapGrpcService<MyGrpcService>();

public class MyGrpcService : MyService.MyServiceBase
{
    public override async Task<MyResponse> GetItem(
        MyRequest request, ServerCallContext context)
    {
        return new MyResponse { Id = request.Id, Result = "OK" };
    }
}

// ─── Client ──────────────────────────────────────────────────────────
builder.Services.AddGrpcClient<MyService.MyServiceClient>(o =>
    o.Address = new Uri("https://grpc-server:7001"));

// In service:
var response = await _client.GetItemAsync(new MyRequest { Id = 1 });

// ─── Error Handling ──────────────────────────────────────────────────
throw new RpcException(new Status(StatusCode.NotFound, "Item not found"));
catch (RpcException ex) when (ex.StatusCode == StatusCode.NotFound) { }

// ─── Deadline ────────────────────────────────────────────────────────
var result = await _client.GetItemAsync(request,
    deadline: DateTime.UtcNow.AddSeconds(30));
```

---

**Guide Complete!** This comprehensive guide covers Protobuf basics, all four gRPC call types (unary, server streaming, client streaming, bidirectional), .NET server and client setup, error handling, interceptors, authentication, gRPC-Web for browsers, and best practices for building high-performance gRPC services in .NET! 🚀
