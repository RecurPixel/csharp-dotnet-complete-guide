# Caching: Memory, Redis & Distributed Patterns Quick Reference

---

## What is Caching?

**Caching** = Storing frequently accessed data in fast-access storage to improve performance

**Benefits:**
- ⚡ **Faster response times** - Avoid expensive operations
- 💰 **Reduced costs** - Fewer database queries
- 📈 **Better scalability** - Handle more requests
- 🛡️ **Resilience** - Serve cached data if backend is down
- 🔋 **Resource efficiency** - Less CPU/memory/network usage

**Common Use Cases:**
- Database query results
- API responses
- Computed values
- User sessions
- Configuration data
- Static content

---

## Caching Strategies

### When to Cache?

```
✅ Cache when:
- Data is read frequently
- Data changes infrequently
- Computation is expensive
- Data is shared across users
- Network latency is high

❌ Don't cache when:
- Data changes frequently
- Data is user-specific and not shared
- Data is small and cheap to retrieve
- Cache overhead > retrieval cost
- Data must be real-time
```

### Cache Levels

```
┌────────────────────────────────────┐
│   Client Browser (HTTP Cache)     │  ← Fastest
├────────────────────────────────────┤
│   CDN (Content Delivery Network)  │
├────────────────────────────────────┤
│   Application Memory Cache         │
├────────────────────────────────────┤
│   Distributed Cache (Redis)        │
├────────────────────────────────────┤
│   Database                         │  ← Slowest
└────────────────────────────────────┘
```

---

## In-Memory Caching (IMemoryCache)

### Setup

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add memory cache
builder.Services.AddMemoryCache();

var app = builder.Build();
```

### Basic Usage

```csharp
using Microsoft.Extensions.Caching.Memory;

public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _repository;
    
    public ProductService(IMemoryCache cache, IProductRepository repository)
    {
        _cache = cache;
        _repository = repository;
    }
    
    public async Task<Product?> GetProductAsync(int id)
    {
        string cacheKey = $"product_{id}";
        
        // Try to get from cache
        if (_cache.TryGetValue(cacheKey, out Product? cachedProduct))
        {
            return cachedProduct;
        }
        
        // Get from database
        var product = await _repository.GetByIdAsync(id);
        
        if (product != null)
        {
            // Store in cache
            _cache.Set(cacheKey, product, TimeSpan.FromMinutes(10));
        }
        
        return product;
    }
}
```

### Cache Options

```csharp
public async Task<Product?> GetProductWithOptionsAsync(int id)
{
    string cacheKey = $"product_{id}";
    
    var cacheOptions = new MemoryCacheEntryOptions()
        // Absolute expiration (fixed time)
        .SetAbsoluteExpiration(TimeSpan.FromMinutes(10))
        
        // Sliding expiration (extends on access)
        .SetSlidingExpiration(TimeSpan.FromMinutes(5))
        
        // Priority (importance when memory is low)
        .SetPriority(CacheItemPriority.Normal)
        
        // Size (for size-based eviction)
        .SetSize(1)
        
        // Callback when removed
        .RegisterPostEvictionCallback((key, value, reason, state) =>
        {
            Console.WriteLine($"Cache item {key} was removed. Reason: {reason}");
        });
    
    return await _cache.GetOrCreateAsync(cacheKey, async entry =>
    {
        entry.SetOptions(cacheOptions);
        return await _repository.GetByIdAsync(id);
    });
}
```

### GetOrCreate Pattern

```csharp
// Best practice - GetOrCreate
public async Task<List<Product>> GetAllProductsAsync()
{
    return await _cache.GetOrCreateAsync("all_products", async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
        return await _repository.GetAllAsync();
    });
}

// With custom logic
public async Task<List<Product>> GetActiveProductsAsync()
{
    string cacheKey = "active_products";
    
    if (!_cache.TryGetValue(cacheKey, out List<Product>? products))
    {
        products = await _repository.GetActiveAsync();
        
        var cacheOptions = new MemoryCacheEntryOptions()
            .SetAbsoluteExpiration(TimeSpan.FromMinutes(10));
        
        _cache.Set(cacheKey, products, cacheOptions);
    }
    
    return products ?? new List<Product>();
}
```

### Cache Invalidation

```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _repository;
    
    public async Task<int> CreateProductAsync(Product product)
    {
        var id = await _repository.AddAsync(product);
        
        // Invalidate cache
        _cache.Remove("all_products");
        _cache.Remove("active_products");
        
        return id;
    }
    
    public async Task UpdateProductAsync(Product product)
    {
        await _repository.UpdateAsync(product);
        
        // Invalidate specific product and lists
        _cache.Remove($"product_{product.Id}");
        _cache.Remove("all_products");
        _cache.Remove("active_products");
    }
    
    public async Task DeleteProductAsync(int id)
    {
        await _repository.DeleteAsync(id);
        
        // Invalidate all related caches
        _cache.Remove($"product_{id}");
        _cache.Remove("all_products");
        _cache.Remove("active_products");
    }
}
```

### Cache Keys Helper

```csharp
public static class CacheKeys
{
    public static string Product(int id) => $"product_{id}";
    public static string AllProducts => "all_products";
    public static string ActiveProducts => "active_products";
    public static string UserOrders(int userId) => $"user_{userId}_orders";
    public static string UserCart(int userId) => $"user_{userId}_cart";
}

// Usage
public async Task<Product?> GetProductAsync(int id)
{
    return await _cache.GetOrCreateAsync(CacheKeys.Product(id), async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
        return await _repository.GetByIdAsync(id);
    });
}
```

---

## Distributed Caching (IDistributedCache)

### Why Distributed Cache?

```
In-Memory Cache:
- ✅ Very fast
- ✅ No network overhead
- ❌ Not shared between servers
- ❌ Lost on app restart
- ❌ Limited by server memory

Distributed Cache:
- ✅ Shared between servers
- ✅ Survives app restart
- ✅ Scalable
- ❌ Network overhead
- ❌ Slightly slower
```

### Setup (Redis)

```bash
# Install packages
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

```csharp
// Program.cs
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "MyApp_";
});
```

### Basic Usage

```csharp
using Microsoft.Extensions.Caching.Distributed;
using System.Text.Json;

public class ProductService
{
    private readonly IDistributedCache _cache;
    private readonly IProductRepository _repository;
    
    public ProductService(IDistributedCache cache, IProductRepository repository)
    {
        _cache = cache;
        _repository = repository;
    }
    
    public async Task<Product?> GetProductAsync(int id)
    {
        string cacheKey = $"product_{id}";
        
        // Try to get from cache
        var cachedData = await _cache.GetStringAsync(cacheKey);
        
        if (!string.IsNullOrEmpty(cachedData))
        {
            return JsonSerializer.Deserialize<Product>(cachedData);
        }
        
        // Get from database
        var product = await _repository.GetByIdAsync(id);
        
        if (product != null)
        {
            // Store in cache
            var serialized = JsonSerializer.Serialize(product);
            var options = new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            };
            
            await _cache.SetStringAsync(cacheKey, serialized, options);
        }
        
        return product;
    }
}
```

### Helper Extension Methods

```csharp
public static class DistributedCacheExtensions
{
    public static async Task<T?> GetAsync<T>(
        this IDistributedCache cache,
        string key,
        CancellationToken cancellationToken = default)
    {
        var data = await cache.GetStringAsync(key, cancellationToken);
        
        if (string.IsNullOrEmpty(data))
            return default;
        
        return JsonSerializer.Deserialize<T>(data);
    }
    
    public static async Task SetAsync<T>(
        this IDistributedCache cache,
        string key,
        T value,
        DistributedCacheEntryOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        var serialized = JsonSerializer.Serialize(value);
        await cache.SetStringAsync(key, serialized, options ?? new DistributedCacheEntryOptions(), cancellationToken);
    }
    
    public static async Task<T> GetOrCreateAsync<T>(
        this IDistributedCache cache,
        string key,
        Func<Task<T>> factory,
        DistributedCacheEntryOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        var cached = await cache.GetAsync<T>(key, cancellationToken);
        
        if (cached != null)
            return cached;
        
        var value = await factory();
        await cache.SetAsync(key, value, options, cancellationToken);
        
        return value;
    }
}

// Usage
public async Task<Product?> GetProductAsync(int id)
{
    return await _cache.GetOrCreateAsync(
        $"product_{id}",
        async () => await _repository.GetByIdAsync(id),
        new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
        });
}
```

---

## Redis Deep Dive

### Connection Setup

```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "Redis": "localhost:6379,password=yourpassword,ssl=false,abortConnect=false"
  }
}

// Advanced configuration
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "MyApp_";
    
    options.ConfigurationOptions = new ConfigurationOptions
    {
        EndPoints = { "localhost:6379" },
        Password = "yourpassword",
        AbortOnConnectFail = false,
        ConnectTimeout = 5000,
        SyncTimeout = 5000,
        AsyncTimeout = 5000,
        ConnectRetry = 3
    };
});
```

### Direct Redis Operations

```csharp
using StackExchange.Redis;

public class RedisService
{
    private readonly IConnectionMultiplexer _redis;
    private readonly IDatabase _db;
    
    public RedisService(IConnectionMultiplexer redis)
    {
        _redis = redis;
        _db = redis.GetDatabase();
    }
    
    // String operations
    public async Task SetStringAsync(string key, string value, TimeSpan? expiry = null)
    {
        await _db.StringSetAsync(key, value, expiry);
    }
    
    public async Task<string?> GetStringAsync(string key)
    {
        return await _db.StringGetAsync(key);
    }
    
    // Hash operations
    public async Task SetHashAsync(string key, string field, string value)
    {
        await _db.HashSetAsync(key, field, value);
    }
    
    public async Task<string?> GetHashAsync(string key, string field)
    {
        return await _db.HashGetAsync(key, field);
    }
    
    public async Task<HashEntry[]> GetAllHashAsync(string key)
    {
        return await _db.HashGetAllAsync(key);
    }
    
    // List operations
    public async Task AddToListAsync(string key, string value)
    {
        await _db.ListRightPushAsync(key, value);
    }
    
    public async Task<string?> GetFromListAsync(string key, long index)
    {
        return await _db.ListGetByIndexAsync(key, index);
    }
    
    // Set operations
    public async Task AddToSetAsync(string key, string value)
    {
        await _db.SetAddAsync(key, value);
    }
    
    public async Task<bool> IsInSetAsync(string key, string value)
    {
        return await _db.SetContainsAsync(key, value);
    }
    
    // Sorted Set operations
    public async Task AddToSortedSetAsync(string key, string value, double score)
    {
        await _db.SortedSetAddAsync(key, value, score);
    }
    
    public async Task<SortedSetEntry[]> GetSortedSetRangeAsync(string key, long start = 0, long stop = -1)
    {
        return await _db.SortedSetRangeByRankWithScoresAsync(key, start, stop);
    }
    
    // Key operations
    public async Task<bool> DeleteKeyAsync(string key)
    {
        return await _db.KeyDeleteAsync(key);
    }
    
    public async Task<bool> KeyExistsAsync(string key)
    {
        return await _db.KeyExistsAsync(key);
    }
    
    public async Task SetExpiryAsync(string key, TimeSpan expiry)
    {
        await _db.KeyExpireAsync(key, expiry);
    }
}
```

### Redis Data Structures Examples

```csharp
public class RedisCacheService
{
    private readonly IDatabase _db;
    
    // User session as Hash
    public async Task SaveUserSessionAsync(string userId, UserSession session)
    {
        var key = $"session:{userId}";
        
        await _db.HashSetAsync(key, new HashEntry[]
        {
            new("userId", session.UserId),
            new("username", session.Username),
            new("email", session.Email),
            new("lastActivity", session.LastActivity.ToString("o"))
        });
        
        await _db.KeyExpireAsync(key, TimeSpan.FromHours(1));
    }
    
    public async Task<UserSession?> GetUserSessionAsync(string userId)
    {
        var key = $"session:{userId}";
        var entries = await _db.HashGetAllAsync(key);
        
        if (entries.Length == 0)
            return null;
        
        var dict = entries.ToDictionary(e => e.Name.ToString(), e => e.Value.ToString());
        
        return new UserSession
        {
            UserId = dict["userId"],
            Username = dict["username"],
            Email = dict["email"],
            LastActivity = DateTime.Parse(dict["lastActivity"])
        };
    }
    
    // Recent items as List
    public async Task AddRecentViewAsync(string userId, string productId)
    {
        var key = $"recent:{userId}";
        
        // Add to front of list
        await _db.ListLeftPushAsync(key, productId);
        
        // Keep only last 10 items
        await _db.ListTrimAsync(key, 0, 9);
        
        // Set expiry
        await _db.KeyExpireAsync(key, TimeSpan.FromDays(7));
    }
    
    public async Task<List<string>> GetRecentViewsAsync(string userId)
    {
        var key = $"recent:{userId}";
        var values = await _db.ListRangeAsync(key);
        return values.Select(v => v.ToString()).ToList();
    }
    
    // Leaderboard as Sorted Set
    public async Task UpdateLeaderboardAsync(string userId, int score)
    {
        var key = "leaderboard";
        await _db.SortedSetAddAsync(key, userId, score);
    }
    
    public async Task<List<(string UserId, int Score)>> GetTopPlayersAsync(int count = 10)
    {
        var key = "leaderboard";
        var entries = await _db.SortedSetRangeByRankWithScoresAsync(
            key, 
            0, 
            count - 1, 
            Order.Descending);
        
        return entries
            .Select(e => (e.Element.ToString(), (int)e.Score))
            .ToList();
    }
    
    // Tags as Set
    public async Task AddProductTagsAsync(string productId, params string[] tags)
    {
        var key = $"product:{productId}:tags";
        await _db.SetAddAsync(key, tags.Select(t => (RedisValue)t).ToArray());
    }
    
    public async Task<List<string>> GetProductTagsAsync(string productId)
    {
        var key = $"product:{productId}:tags";
        var values = await _db.SetMembersAsync(key);
        return values.Select(v => v.ToString()).ToList();
    }
}
```

---

## Cache Patterns

### 1. Cache-Aside (Lazy Loading)

**Most common pattern**

```csharp
public async Task<Product?> GetProductAsync(int id)
{
    // 1. Check cache
    var cached = await _cache.GetAsync<Product>($"product_{id}");
    if (cached != null)
        return cached;
    
    // 2. Get from database
    var product = await _repository.GetByIdAsync(id);
    
    // 3. Store in cache
    if (product != null)
    {
        await _cache.SetAsync($"product_{id}", product, new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
        });
    }
    
    return product;
}

// Pros:
// - Simple to implement
// - Only caches what's requested
// - Cache miss doesn't break app
//
// Cons:
// - Cache miss penalty (slower first request)
// - Risk of stale data
```

### 2. Write-Through

**Cache updated when database is updated**

```csharp
public async Task UpdateProductAsync(Product product)
{
    // 1. Update database
    await _repository.UpdateAsync(product);
    
    // 2. Update cache
    await _cache.SetAsync($"product_{product.Id}", product, new DistributedCacheEntryOptions
    {
        AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
    });
}

// Pros:
// - Cache always up-to-date
// - No stale data
// - Read performance consistent
//
// Cons:
// - Write penalty (two operations)
// - Caching data that might not be read
```

### 3. Write-Behind (Write-Back)

**Cache updated immediately, database updated asynchronously**

```csharp
public class WriteBackCacheService
{
    private readonly IDistributedCache _cache;
    private readonly IProductRepository _repository;
    private readonly IBackgroundTaskQueue _queue;
    
    public async Task UpdateProductAsync(Product product)
    {
        // 1. Update cache immediately
        await _cache.SetAsync($"product_{product.Id}", product);
        
        // 2. Queue database update
        await _queue.QueueBackgroundWorkItemAsync(async token =>
        {
            await _repository.UpdateAsync(product);
        });
    }
}

// Pros:
// - Very fast writes
// - Reduced database load
// - Better write performance
//
// Cons:
// - Risk of data loss
// - Complex to implement
// - Consistency issues
```

### 4. Refresh-Ahead

**Proactively refresh cache before expiration**

```csharp
public class RefreshAheadCache
{
    private readonly IDistributedCache _cache;
    private readonly IProductRepository _repository;
    
    public async Task<Product?> GetProductAsync(int id)
    {
        var cacheKey = $"product_{id}";
        var ttlKey = $"product_{id}_ttl";
        
        // Check cache
        var cached = await _cache.GetAsync<Product>(cacheKey);
        if (cached != null)
        {
            // Check if nearing expiration
            var ttl = await _cache.GetStringAsync(ttlKey);
            if (ttl != null && DateTime.Parse(ttl) < DateTime.UtcNow.AddMinutes(2))
            {
                // Refresh in background
                _ = Task.Run(async () =>
                {
                    var fresh = await _repository.GetByIdAsync(id);
                    if (fresh != null)
                    {
                        await _cache.SetAsync(cacheKey, fresh, new DistributedCacheEntryOptions
                        {
                            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
                        });
                        
                        await _cache.SetStringAsync(ttlKey, 
                            DateTime.UtcNow.AddMinutes(10).ToString("o"));
                    }
                });
            }
            
            return cached;
        }
        
        // Cache miss - load and cache
        var product = await _repository.GetByIdAsync(id);
        if (product != null)
        {
            await _cache.SetAsync(cacheKey, product, new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            });
            
            await _cache.SetStringAsync(ttlKey, 
                DateTime.UtcNow.AddMinutes(10).ToString("o"));
        }
        
        return product;
    }
}

// Pros:
// - No cache miss penalty
// - Always warm cache
// - Predictable performance
//
// Cons:
// - Increased complexity
// - May refresh unused data
// - More resource usage
```

---

## Cache Invalidation Strategies

### 1. Time-Based Expiration

```csharp
// Absolute expiration (fixed time)
var options = new DistributedCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
};

// Sliding expiration (extends on access)
var options = new MemoryCacheEntryOptions()
    .SetSlidingExpiration(TimeSpan.FromMinutes(5));
```

### 2. Event-Based Invalidation

```csharp
public class ProductService
{
    private readonly IDistributedCache _cache;
    private readonly IProductRepository _repository;
    
    public async Task UpdateProductAsync(Product product)
    {
        await _repository.UpdateAsync(product);
        
        // Invalidate affected caches
        await _cache.RemoveAsync($"product_{product.Id}");
        await _cache.RemoveAsync("all_products");
        await _cache.RemoveAsync($"category_{product.CategoryId}_products");
    }
}
```

### 3. Tag-Based Invalidation

```csharp
public class TagBasedCache
{
    private readonly IDistributedCache _cache;
    
    public async Task SetAsync<T>(string key, T value, params string[] tags)
    {
        // Store value
        await _cache.SetAsync(key, value);
        
        // Store tags
        foreach (var tag in tags)
        {
            var tagKey = $"tag:{tag}";
            var keys = await _cache.GetAsync<List<string>>(tagKey) ?? new List<string>();
            keys.Add(key);
            await _cache.SetAsync(tagKey, keys);
        }
    }
    
    public async Task InvalidateByTagAsync(string tag)
    {
        var tagKey = $"tag:{tag}";
        var keys = await _cache.GetAsync<List<string>>(tagKey);
        
        if (keys != null)
        {
            foreach (var key in keys)
            {
                await _cache.RemoveAsync(key);
            }
            
            await _cache.RemoveAsync(tagKey);
        }
    }
}

// Usage
await tagCache.SetAsync("product_1", product, "products", "category_electronics");
await tagCache.InvalidateByTagAsync("products"); // Invalidates all products
```

### 4. Version-Based Invalidation

```csharp
public class VersionedCache
{
    private readonly IDistributedCache _cache;
    private int _version = 1;
    
    public async Task<T?> GetAsync<T>(string key)
    {
        return await _cache.GetAsync<T>($"{key}_v{_version}");
    }
    
    public async Task SetAsync<T>(string key, T value)
    {
        await _cache.SetAsync($"{key}_v{_version}", value);
    }
    
    public void InvalidateAll()
    {
        _version++; // All previous versions become invalid
    }
}
```

---

## Hybrid Caching (L1 + L2)

**Combine in-memory (L1) and distributed (L2) cache**

```csharp
public class HybridCache
{
    private readonly IMemoryCache _l1Cache;
    private readonly IDistributedCache _l2Cache;
    
    public HybridCache(IMemoryCache l1Cache, IDistributedCache l2Cache)
    {
        _l1Cache = l1Cache;
        _l2Cache = l2Cache;
    }
    
    public async Task<T?> GetAsync<T>(string key)
    {
        // Check L1 (memory) first
        if (_l1Cache.TryGetValue(key, out T? l1Value))
        {
            return l1Value;
        }
        
        // Check L2 (distributed)
        var l2Value = await _l2Cache.GetAsync<T>(key);
        
        if (l2Value != null)
        {
            // Promote to L1
            _l1Cache.Set(key, l2Value, TimeSpan.FromMinutes(5));
        }
        
        return l2Value;
    }
    
    public async Task SetAsync<T>(string key, T value, TimeSpan expiration)
    {
        // Set in L1 (shorter expiration)
        _l1Cache.Set(key, value, TimeSpan.FromMinutes(5));
        
        // Set in L2 (longer expiration)
        await _l2Cache.SetAsync(key, value, new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = expiration
        });
    }
    
    public async Task RemoveAsync(string key)
    {
        // Remove from both
        _l1Cache.Remove(key);
        await _l2Cache.RemoveAsync(key);
    }
    
    public async Task<T> GetOrCreateAsync<T>(
        string key,
        Func<Task<T>> factory,
        TimeSpan l1Expiration,
        TimeSpan l2Expiration)
    {
        // Check L1
        if (_l1Cache.TryGetValue(key, out T? l1Value))
            return l1Value!;
        
        // Check L2
        var l2Value = await _l2Cache.GetAsync<T>(key);
        if (l2Value != null)
        {
            _l1Cache.Set(key, l2Value, l1Expiration);
            return l2Value;
        }
        
        // Factory
        var value = await factory();
        
        // Store in both
        _l1Cache.Set(key, value, l1Expiration);
        await _l2Cache.SetAsync(key, value, new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = l2Expiration
        });
        
        return value;
    }
}

// Usage
public class ProductService
{
    private readonly HybridCache _cache;
    private readonly IProductRepository _repository;
    
    public async Task<Product?> GetProductAsync(int id)
    {
        return await _cache.GetOrCreateAsync(
            $"product_{id}",
            async () => await _repository.GetByIdAsync(id),
            l1Expiration: TimeSpan.FromMinutes(5),   // Memory cache
            l2Expiration: TimeSpan.FromMinutes(30)   // Redis cache
        );
    }
}
```

---

## Response Caching

### Output Caching (.NET 7+)

```csharp
// Program.cs
builder.Services.AddOutputCache(options =>
{
    options.AddBasePolicy(builder => builder.Expire(TimeSpan.FromMinutes(10)));
    
    options.AddPolicy("Products", builder => builder
        .Expire(TimeSpan.FromMinutes(30))
        .Tag("products"));
});

var app = builder.Build();
app.UseOutputCache();

// Controller
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [OutputCache(Duration = 60)] // Cache for 60 seconds
    public async Task<ActionResult<List<Product>>> GetProducts()
    {
        var products = await _service.GetAllProductsAsync();
        return Ok(products);
    }
    
    [HttpGet("{id}")]
    [OutputCache(PolicyName = "Products")]
    public async Task<ActionResult<Product>> GetProduct(int id)
    {
        var product = await _service.GetProductAsync(id);
        return product != null ? Ok(product) : NotFound();
    }
    
    [HttpPost]
    public async Task<ActionResult<int>> CreateProduct(Product product)
    {
        var id = await _service.CreateProductAsync(product);
        
        // Invalidate cache by tag
        await HttpContext.RequestServices
            .GetRequiredService<IOutputCacheStore>()
            .EvictByTagAsync("products", default);
        
        return CreatedAtAction(nameof(GetProduct), new { id }, id);
    }
}
```

### Response Caching Middleware (.NET 6)

```csharp
// Program.cs
builder.Services.AddResponseCaching();

var app = builder.Build();
app.UseResponseCaching();

// Controller
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any)]
    public async Task<ActionResult<List<Product>>> GetProducts()
    {
        var products = await _service.GetAllProductsAsync();
        return Ok(products);
    }
    
    [HttpGet("{id}")]
    [ResponseCache(Duration = 300, VaryByQueryKeys = new[] { "id" })]
    public async Task<ActionResult<Product>> GetProduct(int id)
    {
        var product = await _service.GetProductAsync(id);
        return product != null ? Ok(product) : NotFound();
    }
}
```

---

## Best Practices

### 1. Choose Appropriate Cache Duration

```csharp
// Static data - long cache
public async Task<List<Country>> GetCountriesAsync()
{
    return await _cache.GetOrCreateAsync("countries", async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromDays(1);
        return await _repository.GetCountriesAsync();
    });
}

// Frequently changing data - short cache
public async Task<decimal> GetProductPriceAsync(int productId)
{
    return await _cache.GetOrCreateAsync($"price_{productId}", async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
        return await _repository.GetPriceAsync(productId);
    });
}

// Real-time data - no cache or very short
public async Task<int> GetCurrentUsersOnlineAsync()
{
    return await _cache.GetOrCreateAsync("users_online", async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromSeconds(10);
        return await _repository.GetOnlineCountAsync();
    });
}
```

### 2. Cache Null Results

```csharp
// ❌ Bad - Doesn't cache null (cache miss every time)
public async Task<Product?> GetProductAsync(int id)
{
    var product = await _cache.GetOrCreateAsync($"product_{id}", async entry =>
    {
        var result = await _repository.GetByIdAsync(id);
        if (result == null)
            return null; // Not cached!
        
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
        return result;
    });
    
    return product;
}

// ✅ Good - Cache null with shorter expiration
public async Task<Product?> GetProductOptimizedAsync(int id)
{
    return await _cache.GetOrCreateAsync($"product_{id}", async entry =>
    {
        var result = await _repository.GetByIdAsync(id);
        
        // Cache null results with shorter expiration
        entry.AbsoluteExpirationRelativeToNow = result != null 
            ? TimeSpan.FromMinutes(10) 
            : TimeSpan.FromMinutes(1);
        
        return result;
    });
}
```

### 3. Use Structured Cache Keys

```csharp
public static class CacheKeys
{
    // Namespace pattern
    public static string Product(int id) => $"myapp:product:{id}";
    public static string Products() => "myapp:products:all";
    public static string ProductsByCategory(int categoryId) => $"myapp:products:category:{categoryId}";
    
    // User-specific
    public static string UserCart(int userId) => $"myapp:user:{userId}:cart";
    public static string UserOrders(int userId) => $"myapp:user:{userId}:orders";
    
    // With version
    private const int Version = 1;
    public static string VersionedKey(string baseKey) => $"{baseKey}:v{Version}";
}
```

### 4. Handle Cache Failures Gracefully

```csharp
public async Task<Product?> GetProductAsync(int id)
{
    try
    {
        // Try cache
        var cached = await _cache.GetAsync<Product>($"product_{id}");
        if (cached != null)
            return cached;
    }
    catch (Exception ex)
    {
        _logger.LogWarning(ex, "Cache get failed for product {ProductId}", id);
        // Continue to database
    }
    
    // Get from database
    var product = await _repository.GetByIdAsync(id);
    
    if (product != null)
    {
        try
        {
            // Try to cache
            await _cache.SetAsync($"product_{id}", product, new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            });
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, "Cache set failed for product {ProductId}", id);
            // Continue anyway - cache failure shouldn't break app
        }
    }
    
    return product;
}
```

### 5. Avoid Cache Stampede

```csharp
// Cache stampede: Multiple requests miss cache simultaneously
// and all hit database

public class AntiStampedeCacheService
{
    private readonly IDistributedCache _cache;
    private readonly IProductRepository _repository;
    private readonly SemaphoreSlim _semaphore = new(1, 1);
    
    public async Task<Product?> GetProductAsync(int id)
    {
        var cacheKey = $"product_{id}";
        
        // Check cache
        var cached = await _cache.GetAsync<Product>(cacheKey);
        if (cached != null)
            return cached;
        
        // Use semaphore to prevent multiple database calls
        await _semaphore.WaitAsync();
        try
        {
            // Double-check cache
            cached = await _cache.GetAsync<Product>(cacheKey);
            if (cached != null)
                return cached;
            
            // Get from database
            var product = await _repository.GetByIdAsync(id);
            
            if (product != null)
            {
                await _cache.SetAsync(cacheKey, product, new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
                });
            }
            
            return product;
        }
        finally
        {
            _semaphore.Release();
        }
    }
}
```

### 6. Monitor Cache Performance

```csharp
public class MonitoredCacheService
{
    private readonly IDistributedCache _cache;
    private readonly ILogger<MonitoredCacheService> _logger;
    private readonly IMetrics _metrics;
    
    public async Task<T?> GetAsync<T>(string key)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var value = await _cache.GetAsync<T>(key);
            
            stopwatch.Stop();
            
            if (value != null)
            {
                _metrics.RecordCacheHit(stopwatch.ElapsedMilliseconds);
                _logger.LogDebug("Cache hit for {Key} in {Ms}ms", key, stopwatch.ElapsedMilliseconds);
            }
            else
            {
                _metrics.RecordCacheMiss();
                _logger.LogDebug("Cache miss for {Key}", key);
            }
            
            return value;
        }
        catch (Exception ex)
        {
            _metrics.RecordCacheError();
            _logger.LogError(ex, "Cache error for {Key}", key);
            return default;
        }
    }
}
```

---

## Performance Comparison

### Memory Cache vs Distributed Cache

```csharp
[MemoryDiagnoser]
public class CacheBenchmark
{
    private IMemoryCache _memoryCache;
    private IDistributedCache _distributedCache;
    private Product _product;
    
    [GlobalSetup]
    public void Setup()
    {
        _memoryCache = new MemoryCache(new MemoryCacheOptions());
        _distributedCache = new MemoryDistributedCache(
            Options.Create(new MemoryDistributedCacheOptions()));
        
        _product = new Product { Id = 1, Name = "Test Product" };
    }
    
    [Benchmark(Baseline = true)]
    public void MemoryCache_Get()
    {
        _memoryCache.Set("product_1", _product);
        var result = _memoryCache.Get<Product>("product_1");
    }
    
    [Benchmark]
    public async Task DistributedCache_Get()
    {
        await _distributedCache.SetAsync("product_1", _product);
        var result = await _distributedCache.GetAsync<Product>("product_1");
    }
}

// Typical results:
// MemoryCache:      ~100 ns
// DistributedCache: ~50 μs  (500x slower)
// Redis (network):  ~1 ms   (10,000x slower)
```

---

## Common Mistakes

### ❌ 1. Not Setting Expiration

```csharp
// Bad - Cache never expires
_cache.Set("key", value);

// Good - Always set expiration
_cache.Set("key", value, TimeSpan.FromMinutes(10));
```

### ❌ 2. Caching Too Much

```csharp
// Bad - Caching entire database
public async Task<List<Product>> GetAllProducts()
{
    return await _cache.GetOrCreateAsync("all_products", async entry =>
    {
        return await _repository.GetAllAsync(); // Could be millions of products!
    });
}

// Good - Cache with pagination
public async Task<List<Product>> GetProducts(int page, int pageSize)
{
    return await _cache.GetOrCreateAsync($"products_p{page}_s{pageSize}", async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
        return await _repository.GetPagedAsync(page, pageSize);
    });
}
```

### ❌ 3. Not Invalidating on Updates

```csharp
// Bad - Cache not invalidated
public async Task UpdateProduct(Product product)
{
    await _repository.UpdateAsync(product);
    // Cache still has old data!
}

// Good - Invalidate cache
public async Task UpdateProduct(Product product)
{
    await _repository.UpdateAsync(product);
    await _cache.RemoveAsync($"product_{product.Id}");
    await _cache.RemoveAsync("all_products");
}
```

---

## Quick Reference: When to Use What

| Scenario | Solution |
|----------|----------|
| Single server | IMemoryCache |
| Multiple servers | IDistributedCache (Redis) |
| User sessions | Redis with Hash |
| Recent items | Redis with List |
| Leaderboard | Redis with Sorted Set |
| API responses | Output Caching |
| Static content | Response Caching |
| Fast reads, slow writes | Cache-Aside |
| Consistency critical | Write-Through |
| Best performance | Hybrid (Memory + Redis) |

---

**Guide Complete!** This comprehensive caching guide covers in-memory caching, distributed caching with Redis, cache patterns, invalidation strategies, hybrid caching, and best practices for building high-performance .NET applications! 🚀
