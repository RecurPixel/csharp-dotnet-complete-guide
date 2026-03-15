# C# Networking and Resilient I/O Quick Reference

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│               HTTP CLIENT LIFECYCLE — CRITICAL RULE                      │
│                                                                          │
│  ❌ new HttpClient() per request → socket exhaustion, DNS staleness      │
│  ✅ Singleton HttpClient         → works but no DNS refresh              │
│  ✅ IHttpClientFactory           → pooled, rotated handlers (USE THIS)   │
│                                                                          │
│  RESILIENCE LAYERS (outer → inner)                                       │
│  Retry → Circuit Breaker → Timeout → Request                            │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 1. HttpClient Lifetime and Factory Pattern

### IHttpClientFactory — The Correct Approach

```csharp
// In Program.cs / Startup
services.AddHttpClient("GitHub", client =>
{
    client.BaseAddress = new Uri("https://api.github.com/");
    client.DefaultRequestHeaders.Add("User-Agent", "MyApp/1.0");
    client.DefaultRequestHeaders.Add("Accept", "application/vnd.github.v3+json");
    client.Timeout = TimeSpan.FromSeconds(30);
});

// Typed client — cleaner for DI
services.AddHttpClient<GitHubClient>();

// In your service — inject IHttpClientFactory
public class MyService(IHttpClientFactory factory)
{
    public async Task<string> GetDataAsync(CancellationToken ct = default)
    {
        using HttpClient client = factory.CreateClient("GitHub");
        return await client.GetStringAsync("/repos/dotnet/runtime", ct);
    }
}
```

### Typed HttpClient Pattern

```csharp
public class GitHubClient(HttpClient http)
{
    public async Task<GitHubRepo?> GetRepoAsync(string owner, string repo,
        CancellationToken ct = default)
    {
        using HttpResponseMessage response =
            await http.GetAsync($"repos/{owner}/{repo}", ct);
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<GitHubRepo>(ct);
    }
}

// Registration
services.AddHttpClient<GitHubClient>(c =>
{
    c.BaseAddress = new Uri("https://api.github.com/");
    c.DefaultRequestHeaders.Add("User-Agent", "MyApp/1.0");
});
```

---

## 2. Timeouts and Cancellation

```csharp
// Global timeout on the client
client.Timeout = TimeSpan.FromSeconds(10);

// Per-request timeout via CancellationToken (more flexible)
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
try
{
    string result = await client.GetStringAsync(url, cts.Token);
}
catch (OperationCanceledException) when (cts.IsCancellationRequested)
{
    // timeout or cancellation
}

// Combined: external cancel + per-request timeout
using var requestCts = CancellationTokenSource.CreateLinkedTokenSource(
    externalToken, new CancellationTokenSource(TimeSpan.FromSeconds(5)).Token);

await client.GetAsync(url, requestCts.Token);
```

> ⚠️ **Pitfall:** `HttpClient.Timeout` throws `TaskCanceledException`, not `TimeoutException`. Check `ex.InnerException is TimeoutException` to distinguish from user cancellation.

---

## 3. Transient vs Permanent Error Handling

### Error Classification

| HTTP Status | Type | Retry? |
|------------|------|--------|
| 408 Request Timeout | Transient | ✅ Yes |
| 429 Too Many Requests | Transient (with backoff) | ✅ Yes |
| 500 Internal Server Error | Maybe transient | ✅ With limit |
| 502, 503, 504 | Transient | ✅ Yes |
| 400 Bad Request | Permanent | ❌ No |
| 401, 403 | Permanent (auth) | ❌ No |
| 404 Not Found | Permanent | ❌ No |

```csharp
static bool IsTransient(HttpResponseMessage? response, Exception? ex)
{
    if (ex is OperationCanceledException or TimeoutException) return true;
    if (response is null) return false;
    return response.StatusCode is
        HttpStatusCode.RequestTimeout or          // 408
        HttpStatusCode.TooManyRequests or         // 429
        HttpStatusCode.InternalServerError or     // 500
        HttpStatusCode.BadGateway or              // 502
        HttpStatusCode.ServiceUnavailable or      // 503
        HttpStatusCode.GatewayTimeout;            // 504
}
```

---

## 4. Retry with Exponential Backoff and Jitter

```csharp
static async Task<HttpResponseMessage> SendWithRetryAsync(
    HttpClient client,
    HttpRequestMessage request,
    int maxAttempts = 3,
    CancellationToken ct = default)
{
    var rng = Random.Shared;
    HttpResponseMessage? response = null;

    for (int attempt = 1; attempt <= maxAttempts; attempt++)
    {
        // HttpRequestMessage can only be sent once — clone it
        using var req = Clone(request);
        try
        {
            response = await client.SendAsync(req, ct);
            if (response.IsSuccessStatusCode || !IsTransient(response, null))
                return response;
        }
        catch (Exception ex) when (IsTransient(null, ex) && attempt < maxAttempts)
        {
            // fall through to delay
        }

        if (attempt < maxAttempts)
        {
            int baseMs = (int)Math.Pow(2, attempt) * 100;  // 200, 400, 800ms
            int jitter  = rng.Next(0, 50);
            await Task.Delay(baseMs + jitter, ct);
        }
    }

    return response!;
}

static HttpRequestMessage Clone(HttpRequestMessage req)
    => new(req.Method, req.RequestUri)
    {
        Content = req.Content,
        Version = req.Version,
    };
```

---

## 5. Resilience with Microsoft.Extensions.Http.Resilience (.NET 8+)

```csharp
// Install: Microsoft.Extensions.Http.Resilience
services.AddHttpClient<GitHubClient>()
    .AddStandardResilienceHandler(options =>
    {
        options.Retry.MaxRetryAttempts = 3;
        options.Retry.Delay            = TimeSpan.FromMilliseconds(200);
        options.CircuitBreaker.SamplingDuration = TimeSpan.FromSeconds(30);
        options.CircuitBreaker.FailureRatio      = 0.5;
        options.TotalRequestTimeout.Timeout      = TimeSpan.FromSeconds(30);
    });

// Or with Polly directly (Microsoft.Extensions.Http.Polly)
services.AddHttpClient<GitHubClient>()
    .AddTransientHttpErrorPolicy(p =>
        p.WaitAndRetryAsync(3, attempt =>
            TimeSpan.FromSeconds(Math.Pow(2, attempt))));
```

---

## 6. Circuit Breaker Concept

```csharp
// Conceptual circuit breaker states:
//
//  CLOSED   → requests pass through normally
//      ↓ (failure threshold hit)
//  OPEN     → requests fail fast without calling the service
//      ↓ (after cooldown period)
//  HALF-OPEN → test request allowed through
//      ↓ (success) → CLOSED
//      ↓ (failure) → OPEN

// Simple circuit breaker implementation
class CircuitBreaker(int failureThreshold, TimeSpan cooldown)
{
    private int    _failures;
    private bool   _open;
    private DateTime _openedAt;

    public bool IsOpen => _open && DateTime.UtcNow - _openedAt < cooldown;

    public void RecordSuccess() => _failures = 0;

    public void RecordFailure()
    {
        if (++_failures >= failureThreshold)
        {
            _open     = true;
            _openedAt = DateTime.UtcNow;
        }
    }

    public async Task<T> ExecuteAsync<T>(Func<Task<T>> action)
    {
        if (IsOpen) throw new BrokenCircuitException("Circuit is open");
        try
        {
            var result = await action();
            RecordSuccess();
            return result;
        }
        catch
        {
            RecordFailure();
            throw;
        }
    }
}

class BrokenCircuitException(string msg) : Exception(msg);
```

---

## 7. Streaming Request / Response

```csharp
// Stream large response body — avoids loading everything into memory
static async Task DownloadFileAsync(
    HttpClient client, string url, string destPath, CancellationToken ct = default)
{
    using HttpResponseMessage response =
        await client.GetAsync(url, HttpCompletionOption.ResponseHeadersRead, ct);
    response.EnsureSuccessStatusCode();

    await using Stream responseStream = await response.Content.ReadAsStreamAsync(ct);
    await using FileStream fileStream =
        new(destPath, FileMode.Create, FileAccess.Write, FileShare.None,
            bufferSize: 8192, useAsync: true);

    await responseStream.CopyToAsync(fileStream, ct);
}

// Stream upload
static async Task UploadFileAsync(HttpClient client, string url, string filePath)
{
    await using var fs = File.OpenRead(filePath);
    using var content = new StreamContent(fs);
    content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
    var response = await client.PostAsync(url, content);
    response.EnsureSuccessStatusCode();
}
```

---

## 8. JSON API Client Pattern

```csharp
// A clean, reusable JSON API client
public class ApiClient(HttpClient http)
{
    private static readonly JsonSerializerOptions JsonOptions =
        new(JsonSerializerDefaults.Web);

    public async Task<T?> GetAsync<T>(string path, CancellationToken ct = default)
    {
        using HttpResponseMessage resp = await http.GetAsync(path, ct);
        resp.EnsureSuccessStatusCode();
        return await resp.Content.ReadFromJsonAsync<T>(JsonOptions, ct);
    }

    public async Task<TResponse?> PostAsync<TRequest, TResponse>(
        string path, TRequest body, CancellationToken ct = default)
    {
        using HttpResponseMessage resp = await http.PostAsJsonAsync(path, body, JsonOptions, ct);
        resp.EnsureSuccessStatusCode();
        return await resp.Content.ReadFromJsonAsync<TResponse>(JsonOptions, ct);
    }

    public async Task<ApiError?> TryGetErrorAsync(HttpResponseMessage resp,
        CancellationToken ct = default)
    {
        if (resp.IsSuccessStatusCode) return null;
        try { return await resp.Content.ReadFromJsonAsync<ApiError>(JsonOptions, ct); }
        catch { return new ApiError { Message = resp.ReasonPhrase ?? "Unknown error" }; }
    }
}

public record ApiError { public string Message { get; init; } = ""; }
```

---

## 9. Resilient File I/O

```csharp
// Atomic write — write to temp file, then rename (prevents corruption)
static async Task WriteAtomicAsync(string path, byte[] data, CancellationToken ct = default)
{
    string temp = path + ".tmp";
    await File.WriteAllBytesAsync(temp, data, ct);
    File.Move(temp, path, overwrite: true);
}

// Read with retry (file locked by another process)
static async Task<string> ReadWithRetryAsync(string path, int maxAttempts = 3)
{
    for (int attempt = 1; attempt <= maxAttempts; attempt++)
    {
        try   { return await File.ReadAllTextAsync(path); }
        catch (IOException) when (attempt < maxAttempts)
        { await Task.Delay(attempt * 200); }
    }
    return await File.ReadAllTextAsync(path);
}

// Stream large files without loading into memory
static async Task ProcessLargeFileAsync(string path, Func<string, Task> processLine,
    CancellationToken ct = default)
{
    await using var fs     = new FileStream(path, FileMode.Open, FileAccess.Read,
                                FileShare.Read, 4096, FileOptions.Asynchronous | FileOptions.SequentialScan);
    using var reader       = new StreamReader(fs);
    string? line;
    while ((line = await reader.ReadLineAsync(ct)) is not null)
        await processLine(line);
}
```

---

## 10. Recipes

### Recipe: Paginated API Consumer

```csharp
static async IAsyncEnumerable<T> FetchAllPagesAsync<T>(
    HttpClient client,
    string urlTemplate,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    int page = 1;
    while (true)
    {
        var results = await client.GetFromJsonAsync<List<T>>(
            string.Format(urlTemplate, page), ct);

        if (results is null || results.Count == 0) yield break;

        foreach (var item in results) yield return item;
        page++;
    }
}

// Usage
await foreach (var order in FetchAllPagesAsync<Order>(client, "/api/orders?page={0}", ct))
    await ProcessOrderAsync(order, ct);
```

---

## Anti-Patterns

```
❌ new HttpClient() per request — socket exhaustion
❌ HttpClient.Timeout only — can't distinguish timeout from cancellation
❌ Retrying non-transient errors (400, 401, 404) — wastes resources
❌ Not disposing HttpResponseMessage — memory leaks
❌ Loading entire large response into memory — use stream
❌ Missing CancellationToken on all async I/O calls
❌ No timeout on HttpClient — hangs indefinitely
❌ Reusing HttpRequestMessage across retries — it can only be sent once
```

---

## Best Practices

```
✅ Use IHttpClientFactory or typed HttpClient for all HTTP work
✅ Always set both a global and per-request cancellation timeout
✅ Classify errors before retrying — never retry 4xx (except 408, 429)
✅ Add jitter to all retry delays — prevents thundering herd
✅ Stream large responses with ResponseHeadersRead
✅ Use HttpCompletionOption.ResponseHeadersRead for streaming
✅ Write files atomically (temp + rename) to prevent corruption
✅ Use FileOptions.Asynchronous for truly async file I/O
✅ Always dispose HttpResponseMessage (use using)
```

---

## Quick Reference Summary

| Task | API / Package |
|------|--------------|
| HTTP client (DI) | `IHttpClientFactory` |
| Typed client | `services.AddHttpClient<T>()` |
| JSON GET | `client.GetFromJsonAsync<T>()` |
| JSON POST | `client.PostAsJsonAsync()` |
| Stream download | `HttpCompletionOption.ResponseHeadersRead` |
| Built-in resilience | `AddStandardResilienceHandler()` |
| Retry with Polly | `AddTransientHttpErrorPolicy()` |
| Atomic file write | Write temp + `File.Move(overwrite: true)` |
| Async file stream | `FileOptions.Asynchronous` |
| Pagination | `IAsyncEnumerable<T>` + loop |

---

**Guide Complete!** The golden rule: never create HttpClient per-request, always add timeouts and cancellation, and retry only transient errors with exponential backoff + jitter. 📘
