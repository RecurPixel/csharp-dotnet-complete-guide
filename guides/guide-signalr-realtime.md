# SignalR & Real-time Communication Quick Reference

---

## What is SignalR?

**SignalR** = ASP.NET Core library for adding real-time, bidirectional communication between server and clients

**Without SignalR (polling):**
```
Client: "Any updates?"  → Server: "No"      (every 5 seconds, forever)
Client: "Any updates?"  → Server: "No"
Client: "Any updates?"  → Server: "Yes! Here's the data"
```

**With SignalR (push):**
```
Server: "Here's new data!"  → Client (instantly, when it happens)
```

**Key Features:**
- ✅ **Bidirectional** - Server pushes to clients, clients call server methods
- ✅ **Automatic transport selection** - WebSockets → Server-Sent Events → Long Polling
- ✅ **Groups** - Broadcast to subsets of connected clients
- ✅ **Auto-reconnect** - Handles dropped connections gracefully
- ✅ **Scale-out** - Redis backplane for multiple servers

### Transport Selection

```
SignalR tries in order:
┌─────────────────────────────────────────────────────────┐
│  1. WebSockets         Full duplex, lowest latency       │
│  2. Server-Sent Events Server → Client only, HTTP        │
│  3. Long Polling       Fallback, works everywhere        │
└─────────────────────────────────────────────────────────┘
```

---

## Hub Setup

**Hub** = Central class for real-time communication. Clients connect to a Hub and call its methods.

### Installation

```bash
# SignalR is built into ASP.NET Core — no NuGet needed for server
# For .NET client:
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

### Basic Hub

```csharp
using Microsoft.AspNetCore.SignalR;

public class NotificationHub : Hub
{
    private readonly ILogger<NotificationHub> _logger;

    public NotificationHub(ILogger<NotificationHub> logger)
    {
        _logger = logger;
    }

    // ─── Client → Server ─────────────────────────────────────────────

    // Clients call this method
    public async Task SendMessage(string user, string message)
    {
        _logger.LogInformation("Message from {User}: {Message}", user, message);

        // Broadcast to ALL connected clients
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }

    public async Task JoinGroup(string groupName)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
        await Clients.Group(groupName).SendAsync("UserJoined", Context.ConnectionId);
    }

    public async Task LeaveGroup(string groupName)
    {
        await Groups.RemoveFromGroupAsync(Context.ConnectionId, groupName);
        await Clients.Group(groupName).SendAsync("UserLeft", Context.ConnectionId);
    }

    // ─── Connection Lifecycle ─────────────────────────────────────────

    public override async Task OnConnectedAsync()
    {
        _logger.LogInformation("Client connected: {ConnectionId}", Context.ConnectionId);
        await Clients.Caller.SendAsync("Connected", Context.ConnectionId);
        await base.OnConnectedAsync();
    }

    public override async Task OnDisconnectedAsync(Exception? exception)
    {
        _logger.LogInformation("Client disconnected: {ConnectionId}", Context.ConnectionId);

        if (exception != null)
            _logger.LogError(exception, "Client disconnected with error");

        await base.OnDisconnectedAsync(exception);
    }
}
```

### Register in Program.cs

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add SignalR
builder.Services.AddSignalR(options =>
{
    options.EnableDetailedErrors = builder.Environment.IsDevelopment();
    options.ClientTimeoutInterval = TimeSpan.FromSeconds(60);
    options.HandshakeTimeout = TimeSpan.FromSeconds(15);
    options.KeepAliveInterval = TimeSpan.FromSeconds(15);
    options.MaximumReceiveMessageSize = 32 * 1024; // 32KB
});

var app = builder.Build();

// Map hub to URL
app.MapHub<NotificationHub>("/hubs/notifications");
app.MapHub<ChatHub>("/hubs/chat");
```

---

## Typed Hub (Hub\<T\>)

**Typed Hubs** = Strongly-typed contract for what the server can send to clients

```csharp
// ─── Define the client interface ────────────────────────────────────
public interface INotificationClient
{
    Task ReceiveNotification(NotificationDto notification);
    Task ReceiveMessage(string user, string message);
    Task UserJoined(string userId);
    Task UserLeft(string userId);
    Task Connected(string connectionId);
}

// ─── Typed Hub ───────────────────────────────────────────────────────
public class NotificationHub : Hub<INotificationClient>
{
    // ✅ Strongly typed - compiler error if method name is wrong
    public async Task SendMessage(string user, string message)
    {
        await Clients.All.ReceiveMessage(user, message); // Not .SendAsync("ReceiveMessage", ...)
    }

    public async Task SendNotificationToUser(string userId, NotificationDto notification)
    {
        await Clients.User(userId).ReceiveNotification(notification);
    }

    public override async Task OnConnectedAsync()
    {
        await Clients.Caller.Connected(Context.ConnectionId); // ✅ Type-safe
        await base.OnConnectedAsync();
    }
}
```

---

## Clients API Reference

```csharp
// Inside a Hub method:
await Clients.All.SendAsync("Method");                   // Every connected client
await Clients.Caller.SendAsync("Method");                // Only the calling client
await Clients.Others.SendAsync("Method");                // Everyone except caller
await Clients.Client(connectionId).SendAsync("Method"); // Specific connection
await Clients.Clients(connectionIds).SendAsync("Method"); // List of connections
await Clients.Group("groupName").SendAsync("Method");   // All in a group
await Clients.OthersInGroup("groupName").SendAsync("Method"); // Group except caller
await Clients.Groups(groupNames).SendAsync("Method");   // Multiple groups
await Clients.User(userId).SendAsync("Method");         // All connections for a user
await Clients.Users(userIds).SendAsync("Method");       // Multiple users
await Clients.AllExcept(connectionIds).SendAsync("Method"); // All except list

// With typed hub (INotificationClient):
await Clients.All.ReceiveMessage(user, message);        // Type-safe
```

---

## JavaScript Client

```bash
# Install SignalR JavaScript client
npm install @microsoft/signalr
# Or via CDN:
# <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/7.0.5/signalr.min.js"></script>
```

### Basic JavaScript Client

```javascript
// signalr-client.js
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/hubs/notifications", {
        accessTokenFactory: () => getAuthToken() // JWT for authenticated hubs
    })
    .withAutomaticReconnect([0, 2000, 5000, 10000, 30000]) // Retry delays in ms
    .configureLogging(signalR.LogLevel.Information)
    .build();

// ─── Listen for server messages ──────────────────────────────────────
connection.on("ReceiveMessage", (user, message) => {
    console.log(`${user}: ${message}`);
    appendMessageToUI(user, message);
});

connection.on("ReceiveNotification", (notification) => {
    showToast(notification.title, notification.body);
});

// ─── Connection lifecycle ─────────────────────────────────────────────
connection.onreconnecting((error) => {
    console.log("Reconnecting...", error);
    showReconnectingMessage();
});

connection.onreconnected((connectionId) => {
    console.log("Reconnected! ConnectionId:", connectionId);
    hideReconnectingMessage();
});

connection.onclose((error) => {
    console.log("Connection closed", error);
    showDisconnectedMessage();
});

// ─── Start connection ─────────────────────────────────────────────────
async function startConnection() {
    try {
        await connection.start();
        console.log("Connected to SignalR hub");
    } catch (err) {
        console.error("Connection failed:", err);
        setTimeout(startConnection, 5000); // Retry after 5s
    }
}

startConnection();

// ─── Call server methods ──────────────────────────────────────────────
async function sendMessage(user, message) {
    try {
        await connection.invoke("SendMessage", user, message);
    } catch (err) {
        console.error("Failed to send message:", err);
    }
}

async function joinRoom(roomName) {
    await connection.invoke("JoinGroup", roomName);
}
```

---

## .NET Client

```csharp
// Program.cs or service class
using Microsoft.AspNetCore.SignalR.Client;

public class SignalRClientService : IAsyncDisposable
{
    private readonly HubConnection _connection;
    private readonly ILogger<SignalRClientService> _logger;

    public SignalRClientService(ILogger<SignalRClientService> logger)
    {
        _logger = logger;

        _connection = new HubConnectionBuilder()
            .WithUrl("https://myapp.com/hubs/notifications", options =>
            {
                options.AccessTokenProvider = async () => await GetTokenAsync();
            })
            .WithAutomaticReconnect()
            .Build();

        // Register handlers
        _connection.On<string, string>("ReceiveMessage", OnMessageReceived);
        _connection.On<NotificationDto>("ReceiveNotification", OnNotificationReceived);

        _connection.Reconnecting += OnReconnecting;
        _connection.Reconnected += OnReconnected;
        _connection.Closed += OnClosed;
    }

    public async Task StartAsync(CancellationToken cancellationToken = default)
    {
        await _connection.StartAsync(cancellationToken);
        _logger.LogInformation("Connected to hub");
    }

    public async Task SendMessageAsync(string user, string message)
    {
        await _connection.InvokeAsync("SendMessage", user, message);
    }

    private void OnMessageReceived(string user, string message)
    {
        _logger.LogInformation("Message from {User}: {Message}", user, message);
    }

    private void OnNotificationReceived(NotificationDto notification)
    {
        _logger.LogInformation("Notification: {Title}", notification.Title);
    }

    private Task OnReconnecting(Exception? error)
    {
        _logger.LogWarning("Reconnecting: {Error}", error?.Message);
        return Task.CompletedTask;
    }

    private Task OnReconnected(string? connectionId)
    {
        _logger.LogInformation("Reconnected: {ConnectionId}", connectionId);
        return Task.CompletedTask;
    }

    private Task OnClosed(Exception? error)
    {
        _logger.LogWarning("Connection closed: {Error}", error?.Message);
        return Task.CompletedTask;
    }

    public async ValueTask DisposeAsync()
    {
        await _connection.DisposeAsync();
    }
}
```

---

## Authentication & Authorization

```csharp
// Program.cs - authenticate the hub
app.MapHub<NotificationHub>("/hubs/notifications")
   .RequireAuthorization(); // Requires authenticated user

// Or on the hub class
[Authorize]
public class NotificationHub : Hub<INotificationClient>
{
    // Access user info
    public async Task SendPersonalNotification(string message)
    {
        var userId = Context.UserIdentifier;  // From ClaimTypes.NameIdentifier
        var username = Context.User?.FindFirst(ClaimTypes.Name)?.Value;

        await Clients.Caller.ReceiveMessage("System", $"Hello {username}!");
    }
}

// Role-based authorization
[Authorize(Roles = "Admin")]
public async Task BroadcastToAll(string message)
{
    await Clients.All.ReceiveMessage("Admin", message);
}

// Policy-based authorization
[Authorize(Policy = "CanSendAlerts")]
public async Task SendAlert(AlertDto alert)
{
    await Clients.All.ReceiveNotification(new NotificationDto { Title = alert.Title });
}
```

### JWT Auth from JavaScript

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/hubs/notifications", {
        // ✅ JWT sent as query string (SignalR can't set Authorization header)
        accessTokenFactory: () => {
            return localStorage.getItem("jwt_token"); // Or from your auth state
        }
    })
    .build();
```

```csharp
// Program.cs - configure JWT to read from query string (needed for SignalR)
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Events = new JwtBearerEvents
        {
            // ✅ SignalR sends JWT as query string: ?access_token=...
            OnMessageReceived = context =>
            {
                var accessToken = context.Request.Query["access_token"];
                var path = context.HttpContext.Request.Path;

                if (!string.IsNullOrEmpty(accessToken) &&
                    path.StartsWithSegments("/hubs"))
                {
                    context.Token = accessToken;
                }
                return Task.CompletedTask;
            }
        };
    });
```

---

## Real-time Notifications Pattern

### Notification Service (Push from Server)

```csharp
// Interface for pushing notifications from anywhere in the app
public interface INotificationService
{
    Task SendToUserAsync(string userId, NotificationDto notification);
    Task SendToGroupAsync(string groupName, NotificationDto notification);
    Task BroadcastAsync(NotificationDto notification);
}

// Implementation using IHubContext (inject into any service)
public class NotificationService : INotificationService
{
    private readonly IHubContext<NotificationHub, INotificationClient> _hubContext;

    public NotificationService(
        IHubContext<NotificationHub, INotificationClient> hubContext)
    {
        _hubContext = hubContext;
    }

    public async Task SendToUserAsync(string userId, NotificationDto notification)
    {
        await _hubContext.Clients.User(userId).ReceiveNotification(notification);
    }

    public async Task SendToGroupAsync(string groupName, NotificationDto notification)
    {
        await _hubContext.Clients.Group(groupName).ReceiveNotification(notification);
    }

    public async Task BroadcastAsync(NotificationDto notification)
    {
        await _hubContext.Clients.All.ReceiveNotification(notification);
    }
}

// Register
builder.Services.AddScoped<INotificationService, NotificationService>();

// Use in a controller or service
public class OrdersController : ControllerBase
{
    private readonly INotificationService _notifications;
    private readonly IOrderService _orderService;

    public OrdersController(
        INotificationService notifications,
        IOrderService orderService)
    {
        _notifications = notifications;
        _orderService = orderService;
    }

    [HttpPost("{id}/approve")]
    public async Task<IActionResult> ApproveOrder(Guid id)
    {
        var order = await _orderService.ApproveAsync(id);

        // Notify the customer in real time
        await _notifications.SendToUserAsync(order.CustomerId.ToString(),
            new NotificationDto
            {
                Title = "Order Approved!",
                Body = $"Your order #{order.OrderNumber} has been approved.",
                Type = "order_approved",
                Timestamp = DateTimeOffset.UtcNow
            });

        return Ok(order);
    }
}
```

---

## Chat Application

### Chat Hub

```csharp
public class ChatHub : Hub<IChatClient>
{
    private static readonly Dictionary<string, string> _userNames = new();

    public override async Task OnConnectedAsync()
    {
        await base.OnConnectedAsync();
    }

    public override async Task OnDisconnectedAsync(Exception? exception)
    {
        if (_userNames.TryGetValue(Context.ConnectionId, out var username))
        {
            _userNames.Remove(Context.ConnectionId);
            await Clients.All.UserLeft(username);
        }

        await base.OnDisconnectedAsync(exception);
    }

    public async Task RegisterUser(string username)
    {
        _userNames[Context.ConnectionId] = username;
        await Clients.All.UserJoined(username);
        await Clients.Caller.UserList(_userNames.Values.ToList());
    }

    public async Task SendMessage(string message)
    {
        if (_userNames.TryGetValue(Context.ConnectionId, out var username))
        {
            var chatMessage = new ChatMessageDto
            {
                Username = username,
                Message = message,
                Timestamp = DateTimeOffset.UtcNow
            };
            await Clients.All.ReceiveChatMessage(chatMessage);
        }
    }

    public async Task SendPrivateMessage(string targetConnectionId, string message)
    {
        if (_userNames.TryGetValue(Context.ConnectionId, out var senderName))
        {
            var dm = new DirectMessageDto
            {
                From = senderName,
                Message = message,
                Timestamp = DateTimeOffset.UtcNow
            };
            await Clients.Client(targetConnectionId).ReceiveDirectMessage(dm);
            await Clients.Caller.ReceiveDirectMessage(dm); // Echo to sender too
        }
    }

    public async Task JoinRoom(string room)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, room);

        if (_userNames.TryGetValue(Context.ConnectionId, out var username))
            await Clients.Group(room).UserJoined(username);
    }

    public async Task SendToRoom(string room, string message)
    {
        if (_userNames.TryGetValue(Context.ConnectionId, out var username))
        {
            await Clients.Group(room).ReceiveChatMessage(new ChatMessageDto
            {
                Username = username,
                Message = message,
                Room = room,
                Timestamp = DateTimeOffset.UtcNow
            });
        }
    }
}

// Client interface
public interface IChatClient
{
    Task ReceiveChatMessage(ChatMessageDto message);
    Task ReceiveDirectMessage(DirectMessageDto message);
    Task UserJoined(string username);
    Task UserLeft(string username);
    Task UserList(List<string> users);
}

// DTOs
public record ChatMessageDto
{
    public string Username { get; init; } = string.Empty;
    public string Message { get; init; } = string.Empty;
    public string? Room { get; init; }
    public DateTimeOffset Timestamp { get; init; }
}
```

### Chat JavaScript Client

```javascript
// chat.js
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/hubs/chat")
    .withAutomaticReconnect()
    .build();

// Listen for messages
connection.on("ReceiveChatMessage", (message) => {
    appendMessage(message.username, message.message, message.timestamp);
});

connection.on("UserJoined", (username) => {
    appendSystemMessage(`${username} joined the chat`);
});

connection.on("UserLeft", (username) => {
    appendSystemMessage(`${username} left the chat`);
});

connection.on("UserList", (users) => {
    updateUserList(users);
});

// Connect and register
async function connect(username) {
    await connection.start();
    await connection.invoke("RegisterUser", username);
}

// Send message
async function sendMessage(message) {
    await connection.invoke("SendMessage", message);
}

// Join room
async function joinRoom(room) {
    await connection.invoke("JoinRoom", room);
}

// Send to room
async function sendToRoom(room, message) {
    await connection.invoke("SendToRoom", room, message);
}
```

---

## Scaling with Redis Backplane

When running multiple server instances, SignalR connections are distributed. Use Redis to share messages across instances.

```
Server 1 ←─── Redis ───→ Server 2
  ↑                           ↑
Clients A, B             Clients C, D
```

```bash
dotnet add package Microsoft.AspNetCore.SignalR.StackExchangeRedis
```

```csharp
// Program.cs
builder.Services.AddSignalR()
    .AddStackExchangeRedis(
        builder.Configuration.GetConnectionString("Redis")!,
        options =>
        {
            options.Configuration.ChannelPrefix = RedisChannel.Literal("MyApp");
        });
```

---

## SignalR vs WebSockets vs SSE

| Feature | SignalR | Raw WebSockets | Server-Sent Events |
|---|---|---|---|
| **Direction** | Bidirectional | Bidirectional | Server → Client only |
| **Auto-reconnect** | ✅ Built-in | ❌ Manual | ✅ Browser built-in |
| **Fallback** | ✅ Automatic | ❌ None | ❌ None |
| **Browser support** | ✅ All | ✅ All modern | ✅ All (not IE) |
| **Groups** | ✅ Built-in | ❌ Manual | ❌ Manual |
| **Auth** | ✅ Built-in | ❌ Manual | ❌ Manual |
| **Complexity** | Low | Medium | Low |
| **Best for** | .NET apps | Custom protocols | Read-only feeds |

### When to Use What

```
Need server push only (notifications, live feeds)?
    → Server-Sent Events OR SignalR

Need bidirectional (chat, collaboration, live editing)?
    → SignalR

Already in .NET ecosystem?
    → SignalR (always — it handles everything for you)

Building a game or custom protocol with very low latency?
    → Raw WebSockets
```

---

## Best Practices

### 1. Always Use Typed Hubs

```csharp
// ✅ Typed Hub - compile-time safety
public class MyHub : Hub<IMyClient> { }

// ❌ Avoid stringly-typed SendAsync in large apps
await Clients.All.SendAsync("MethodThatMightBeTypo", data);
```

### 2. Keep Hub Methods Thin

```csharp
// ✅ Thin hub - delegate to services
public class OrderHub : Hub<IOrderClient>
{
    private readonly IOrderService _orderService;
    public OrderHub(IOrderService orderService) => _orderService = orderService;

    public async Task PlaceOrder(CreateOrderDto dto)
    {
        var order = await _orderService.PlaceOrderAsync(dto, Context.UserIdentifier!);
        await Clients.Caller.OrderConfirmed(order);
    }
}
```

### 3. Handle Disconnections Gracefully

```csharp
// ✅ Clean up state on disconnect
public override async Task OnDisconnectedAsync(Exception? exception)
{
    await _presenceService.UserDisconnectedAsync(Context.UserIdentifier!);
    await Groups.RemoveFromGroupAsync(Context.ConnectionId, "active-users");
    await base.OnDisconnectedAsync(exception);
}
```

### 4. Use IHubContext to Push from Outside Hubs

```csharp
// ✅ Inject IHubContext<THub, TClient> into any service
public class BackgroundNotifier : BackgroundService
{
    private readonly IHubContext<NotificationHub, INotificationClient> _hub;
    public BackgroundNotifier(IHubContext<NotificationHub, INotificationClient> hub)
        => _hub = hub;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await _hub.Clients.All.ReceiveNotification(new NotificationDto
            {
                Title = "Server heartbeat",
                Timestamp = DateTimeOffset.UtcNow
            });
            await Task.Delay(30_000, stoppingToken);
        }
    }
}
```

### 5. Configure CORS for SignalR

```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("SignalRPolicy", policy =>
    {
        policy
            .WithOrigins("https://myapp.com", "https://www.myapp.com")
            .AllowAnyHeader()
            .AllowAnyMethod()
            .AllowCredentials(); // ✅ Required for SignalR cookies/auth
    });
});

app.UseCors("SignalRPolicy");
app.MapHub<NotificationHub>("/hubs/notifications");
```

---

## Quick Reference

```csharp
// ─── Program.cs ──────────────────────────────────────────────────────
builder.Services.AddSignalR();
app.MapHub<NotificationHub>("/hubs/notifications");

// ─── Hub ─────────────────────────────────────────────────────────────
public class MyHub : Hub<IMyClient>
{
    public async Task DoWork(string data)
    {
        await Clients.All.ReceiveResult(data);       // All clients
        await Clients.Caller.ReceiveResult(data);    // Just caller
        await Clients.User(userId).ReceiveResult(data); // Specific user
        await Clients.Group("room1").ReceiveResult(data); // Group
    }
}

// ─── Groups ──────────────────────────────────────────────────────────
await Groups.AddToGroupAsync(Context.ConnectionId, "room1");
await Groups.RemoveFromGroupAsync(Context.ConnectionId, "room1");

// ─── Push from service (IHubContext) ─────────────────────────────────
public class MyService
{
    private readonly IHubContext<MyHub, IMyClient> _hub;
    public MyService(IHubContext<MyHub, IMyClient> hub) => _hub = hub;
    public async Task Notify() => await _hub.Clients.All.ReceiveResult("data");
}

// ─── JavaScript ──────────────────────────────────────────────────────
// const conn = new signalR.HubConnectionBuilder().withUrl("/hubs/my").withAutomaticReconnect().build();
// conn.on("ReceiveResult", data => console.log(data));
// await conn.start();
// await conn.invoke("DoWork", "mydata");
```

---

**Guide Complete!** This comprehensive guide covers SignalR hubs, typed hubs, the full Clients API, JavaScript and .NET clients, authentication, real-time notification patterns, a full chat application, Redis backplane for scale-out, and when to use SignalR vs alternatives! ⚡
