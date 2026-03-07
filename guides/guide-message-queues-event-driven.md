# Message Queues & Event-Driven Architecture Quick Reference

---

## What is Event-Driven Architecture?

**Event-Driven Architecture (EDA)** = Architecture pattern where services communicate through events

**Message Queue** = Middleware for asynchronous message passing between services

**Benefits:**
- 🔄 **Decoupling** - Services don't need to know about each other
- 📈 **Scalability** - Scale services independently
- 🛡️ **Resilience** - Services can be down without data loss
- ⚡ **Performance** - Asynchronous processing
- 🔌 **Flexibility** - Easy to add new services
- 📊 **Event sourcing** - Audit trail of all events

**Use Cases:**
- Order processing
- Email notifications
- Background tasks
- Microservices communication
- Real-time updates
- Data synchronization

---

## Message Queue Patterns

### 1. Point-to-Point (Queue)

```
Producer → [Queue] → Consumer
```

**Characteristics:**
- One message consumed by ONE consumer
- Guaranteed delivery (at-least-once)
- Messages processed in order (FIFO)

**Use Cases:**
- Order processing
- Task distribution
- Work queue

### 2. Publish-Subscribe (Topic)

```
Publisher → [Topic] → Subscriber 1
                   → Subscriber 2
                   → Subscriber 3
```

**Characteristics:**
- One message consumed by MULTIPLE consumers
- Each subscriber gets a copy
- Broadcast pattern

**Use Cases:**
- Event notifications
- Real-time updates
- Logging/monitoring

### 3. Request-Reply

```
Service A → [Request Queue] → Service B
         ← [Reply Queue]    ←
```

**Characteristics:**
- Synchronous-like behavior
- Correlation ID to match request/response
- Timeout handling

**Use Cases:**
- RPC-style communication
- API gateway to microservices

---

## RabbitMQ

### Installation (Docker)

```bash
# Run RabbitMQ with management UI
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management

# Access management UI: http://localhost:15672
# Default credentials: guest/guest
```

### Setup in .NET

```bash
# Install package
dotnet add package RabbitMQ.Client
```

```csharp
// Connection factory
public class RabbitMqConnectionFactory
{
    private readonly IConfiguration _configuration;
    
    public RabbitMqConnectionFactory(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public IConnection CreateConnection()
    {
        var factory = new ConnectionFactory
        {
            HostName = _configuration["RabbitMQ:HostName"] ?? "localhost",
            Port = int.Parse(_configuration["RabbitMQ:Port"] ?? "5672"),
            UserName = _configuration["RabbitMQ:UserName"] ?? "guest",
            Password = _configuration["RabbitMQ:Password"] ?? "guest",
            VirtualHost = _configuration["RabbitMQ:VirtualHost"] ?? "/",
            AutomaticRecoveryEnabled = true,
            NetworkRecoveryInterval = TimeSpan.FromSeconds(10)
        };
        
        return factory.CreateConnection();
    }
}
```

### Producer (Publisher)

```csharp
public class RabbitMqProducer
{
    private readonly IConnection _connection;
    private readonly IModel _channel;
    private readonly ILogger<RabbitMqProducer> _logger;
    
    public RabbitMqProducer(
        RabbitMqConnectionFactory connectionFactory,
        ILogger<RabbitMqProducer> logger)
    {
        _connection = connectionFactory.CreateConnection();
        _channel = _connection.CreateModel();
        _logger = logger;
        
        // Declare queue (idempotent)
        _channel.QueueDeclare(
            queue: "orders",
            durable: true,
            exclusive: false,
            autoDelete: false,
            arguments: null);
    }
    
    public void PublishMessage(string queueName, object message)
    {
        var json = JsonSerializer.Serialize(message);
        var body = Encoding.UTF8.GetBytes(json);
        
        var properties = _channel.CreateBasicProperties();
        properties.Persistent = true; // Message survives broker restart
        properties.ContentType = "application/json";
        properties.MessageId = Guid.NewGuid().ToString();
        properties.Timestamp = new AmqpTimestamp(DateTimeOffset.UtcNow.ToUnixTimeSeconds());
        
        _channel.BasicPublish(
            exchange: "",
            routingKey: queueName,
            basicProperties: properties,
            body: body);
        
        _logger.LogInformation("Published message to queue {Queue}: {MessageId}", 
            queueName, properties.MessageId);
    }
    
    public void Dispose()
    {
        _channel?.Close();
        _connection?.Close();
    }
}

// Usage
public class OrderService
{
    private readonly RabbitMqProducer _producer;
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var order = await ProcessOrderAsync(request);
        
        // Publish event
        _producer.PublishMessage("orders", new OrderCreatedEvent
        {
            OrderId = order.Id,
            CustomerId = order.CustomerId,
            TotalAmount = order.TotalAmount,
            CreatedAt = DateTime.UtcNow
        });
        
        return order;
    }
}
```

### Consumer (Subscriber)

```csharp
public class RabbitMqConsumer : BackgroundService
{
    private readonly IConnection _connection;
    private readonly IModel _channel;
    private readonly ILogger<RabbitMqConsumer> _logger;
    private readonly IServiceProvider _serviceProvider;
    
    public RabbitMqConsumer(
        RabbitMqConnectionFactory connectionFactory,
        ILogger<RabbitMqConsumer> logger,
        IServiceProvider serviceProvider)
    {
        _connection = connectionFactory.CreateConnection();
        _channel = _connection.CreateModel();
        _logger = logger;
        _serviceProvider = serviceProvider;
        
        // Declare queue
        _channel.QueueDeclare(
            queue: "orders",
            durable: true,
            exclusive: false,
            autoDelete: false,
            arguments: null);
        
        // Set prefetch count (don't send new message until current is processed)
        _channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
    }
    
    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var consumer = new EventingBasicConsumer(_channel);
        
        consumer.Received += async (model, ea) =>
        {
            try
            {
                var body = ea.Body.ToArray();
                var json = Encoding.UTF8.GetString(body);
                var orderEvent = JsonSerializer.Deserialize<OrderCreatedEvent>(json);
                
                _logger.LogInformation("Received message: {MessageId}", ea.BasicProperties.MessageId);
                
                // Process message
                using var scope = _serviceProvider.CreateScope();
                var orderProcessor = scope.ServiceProvider.GetRequiredService<IOrderProcessor>();
                await orderProcessor.ProcessOrderAsync(orderEvent);
                
                // Acknowledge message (remove from queue)
                _channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);
                
                _logger.LogInformation("Message processed successfully: {MessageId}", 
                    ea.BasicProperties.MessageId);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error processing message: {MessageId}", 
                    ea.BasicProperties.MessageId);
                
                // Reject and requeue (or send to dead letter queue)
                _channel.BasicNack(deliveryTag: ea.DeliveryTag, multiple: false, requeue: false);
            }
        };
        
        _channel.BasicConsume(
            queue: "orders",
            autoAck: false, // Manual acknowledgment
            consumer: consumer);
        
        return Task.CompletedTask;
    }
    
    public override void Dispose()
    {
        _channel?.Close();
        _connection?.Close();
        base.Dispose();
    }
}

// Register as hosted service
builder.Services.AddHostedService<RabbitMqConsumer>();
```

### Publish-Subscribe Pattern

```csharp
public class RabbitMqPublisher
{
    private readonly IModel _channel;
    
    public RabbitMqPublisher(RabbitMqConnectionFactory connectionFactory)
    {
        var connection = connectionFactory.CreateConnection();
        _channel = connection.CreateModel();
        
        // Declare exchange (fanout = broadcast to all queues)
        _channel.ExchangeDeclare(
            exchange: "order-events",
            type: ExchangeType.Fanout,
            durable: true);
    }
    
    public void PublishEvent(string exchange, object @event)
    {
        var json = JsonSerializer.Serialize(@event);
        var body = Encoding.UTF8.GetBytes(json);
        
        _channel.BasicPublish(
            exchange: exchange,
            routingKey: "", // Not used for fanout
            basicProperties: null,
            body: body);
    }
}

public class RabbitMqSubscriber
{
    public RabbitMqSubscriber(RabbitMqConnectionFactory connectionFactory)
    {
        var connection = connectionFactory.CreateConnection();
        var channel = connection.CreateModel();
        
        // Declare exchange
        channel.ExchangeDeclare(
            exchange: "order-events",
            type: ExchangeType.Fanout,
            durable: true);
        
        // Create queue for this service
        var queueName = channel.QueueDeclare().QueueName;
        
        // Bind queue to exchange
        channel.QueueBind(
            queue: queueName,
            exchange: "order-events",
            routingKey: "");
        
        // Consume messages
        var consumer = new EventingBasicConsumer(channel);
        consumer.Received += (model, ea) =>
        {
            var body = ea.Body.ToArray();
            var json = Encoding.UTF8.GetString(body);
            // Process event
        };
        
        channel.BasicConsume(queue: queueName, autoAck: true, consumer: consumer);
    }
}
```

### Topic Exchange (Routing)

```csharp
public class RabbitMqTopicPublisher
{
    private readonly IModel _channel;
    
    public RabbitMqTopicPublisher(RabbitMqConnectionFactory connectionFactory)
    {
        var connection = connectionFactory.CreateConnection();
        _channel = connection.CreateModel();
        
        // Declare topic exchange
        _channel.ExchangeDeclare(
            exchange: "logs",
            type: ExchangeType.Topic,
            durable: true);
    }
    
    public void PublishLog(string severity, string message)
    {
        var routingKey = $"log.{severity}"; // log.info, log.warning, log.error
        var body = Encoding.UTF8.GetBytes(message);
        
        _channel.BasicPublish(
            exchange: "logs",
            routingKey: routingKey,
            basicProperties: null,
            body: body);
    }
}

public class RabbitMqTopicSubscriber
{
    public RabbitMqTopicSubscriber(RabbitMqConnectionFactory connectionFactory, string pattern)
    {
        var connection = connectionFactory.CreateConnection();
        var channel = connection.CreateModel();
        
        channel.ExchangeDeclare(exchange: "logs", type: ExchangeType.Topic, durable: true);
        
        var queueName = channel.QueueDeclare().QueueName;
        
        // Bind with pattern
        // * = exactly one word
        // # = zero or more words
        channel.QueueBind(queue: queueName, exchange: "logs", routingKey: pattern);
        // Examples:
        // "log.*" - matches log.info, log.warning
        // "log.#" - matches log.info, log.info.critical
        // "#" - matches everything
    }
}
```

---

## Azure Service Bus (Covered in Azure Guide)

Quick recap for comparison:

```csharp
// Queue
var sender = client.CreateSender("orders");
await sender.SendMessageAsync(new ServiceBusMessage(json));

// Topic
var sender = client.CreateSender("order-events");
await sender.SendMessageAsync(new ServiceBusMessage(json));

// Subscription
var processor = client.CreateProcessor("order-events", "email-service");
```

---

## Event-Driven Patterns

### 1. Event Notification

**Simple event announcing something happened**

```csharp
// Event
public record OrderCreatedEvent(
    int OrderId,
    int CustomerId,
    decimal TotalAmount,
    DateTime CreatedAt);

// Publisher
public class OrderService
{
    private readonly IEventPublisher _eventPublisher;
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var order = await SaveOrderAsync(request);
        
        // Publish event
        await _eventPublisher.PublishAsync(new OrderCreatedEvent(
            order.Id,
            order.CustomerId,
            order.TotalAmount,
            DateTime.UtcNow));
        
        return order;
    }
}

// Subscriber 1: Send email
public class EmailNotificationHandler : IEventHandler<OrderCreatedEvent>
{
    public async Task HandleAsync(OrderCreatedEvent @event)
    {
        await SendOrderConfirmationEmailAsync(@event.CustomerId, @event.OrderId);
    }
}

// Subscriber 2: Update analytics
public class AnalyticsHandler : IEventHandler<OrderCreatedEvent>
{
    public async Task HandleAsync(OrderCreatedEvent @event)
    {
        await RecordOrderAnalyticsAsync(@event);
    }
}
```

### 2. Event-Carried State Transfer

**Event contains full state, subscribers don't need to query**

```csharp
public record CustomerUpdatedEvent(
    int CustomerId,
    string FirstName,
    string LastName,
    string Email,
    string Address,
    string Phone,
    DateTime UpdatedAt);

// Subscriber maintains local copy of customer data
public class OrderServiceCustomerCache : IEventHandler<CustomerUpdatedEvent>
{
    private readonly ICustomerCacheRepository _cache;
    
    public async Task HandleAsync(CustomerUpdatedEvent @event)
    {
        // Update local cache
        await _cache.UpdateAsync(new CustomerCache
        {
            CustomerId = @event.CustomerId,
            FirstName = @event.FirstName,
            LastName = @event.LastName,
            Email = @event.Email,
            Address = @event.Address,
            Phone = @event.Phone
        });
    }
}
```

### 3. Event Sourcing

**Store events as the source of truth**

```csharp
// Events
public abstract record OrderEvent(int OrderId, DateTime Timestamp);
public record OrderCreatedEvent(int OrderId, DateTime Timestamp, int CustomerId, decimal Total) 
    : OrderEvent(OrderId, Timestamp);
public record OrderItemAddedEvent(int OrderId, DateTime Timestamp, int ProductId, int Quantity) 
    : OrderEvent(OrderId, Timestamp);
public record OrderShippedEvent(int OrderId, DateTime Timestamp, string TrackingNumber) 
    : OrderEvent(OrderId, Timestamp);

// Aggregate
public class Order
{
    public int Id { get; private set; }
    public int CustomerId { get; private set; }
    public List<OrderItem> Items { get; private set; } = new();
    public OrderStatus Status { get; private set; }
    public string? TrackingNumber { get; private set; }
    
    // Apply events to rebuild state
    public void Apply(OrderEvent @event)
    {
        switch (@event)
        {
            case OrderCreatedEvent created:
                Id = created.OrderId;
                CustomerId = created.CustomerId;
                Status = OrderStatus.Created;
                break;
                
            case OrderItemAddedEvent itemAdded:
                Items.Add(new OrderItem
                {
                    ProductId = itemAdded.ProductId,
                    Quantity = itemAdded.Quantity
                });
                break;
                
            case OrderShippedEvent shipped:
                Status = OrderStatus.Shipped;
                TrackingNumber = shipped.TrackingNumber;
                break;
        }
    }
    
    // Rebuild from events
    public static Order FromEvents(IEnumerable<OrderEvent> events)
    {
        var order = new Order();
        foreach (var @event in events)
        {
            order.Apply(@event);
        }
        return order;
    }
}

// Event Store
public class EventStore
{
    private readonly IDbConnection _connection;
    
    public async Task SaveEventAsync(OrderEvent @event)
    {
        await _connection.ExecuteAsync(@"
            INSERT INTO Events (AggregateId, EventType, EventData, Timestamp)
            VALUES (@AggregateId, @EventType, @EventData, @Timestamp)",
            new
            {
                AggregateId = @event.OrderId,
                EventType = @event.GetType().Name,
                EventData = JsonSerializer.Serialize(@event),
                @event.Timestamp
            });
    }
    
    public async Task<List<OrderEvent>> GetEventsAsync(int orderId)
    {
        var rows = await _connection.QueryAsync<EventRow>(@"
            SELECT EventType, EventData
            FROM Events
            WHERE AggregateId = @OrderId
            ORDER BY Timestamp",
            new { OrderId = orderId });
        
        return rows.Select(r => JsonSerializer.Deserialize<OrderEvent>(r.EventData)!).ToList();
    }
}
```

### 4. CQRS (Command Query Responsibility Segregation)

**Separate read and write models**

```csharp
// Commands (writes)
public record CreateOrderCommand(int CustomerId, List<OrderItem> Items);
public record ShipOrderCommand(int OrderId, string TrackingNumber);

// Command Handler
public class CreateOrderCommandHandler
{
    private readonly IEventStore _eventStore;
    private readonly IEventPublisher _eventPublisher;
    
    public async Task<int> HandleAsync(CreateOrderCommand command)
    {
        var orderId = GenerateOrderId();
        
        // Create event
        var @event = new OrderCreatedEvent(
            orderId,
            DateTime.UtcNow,
            command.CustomerId,
            CalculateTotal(command.Items));
        
        // Save to event store
        await _eventStore.SaveEventAsync(@event);
        
        // Publish for read models
        await _eventPublisher.PublishAsync(@event);
        
        return orderId;
    }
}

// Queries (reads)
public record GetOrderQuery(int OrderId);
public record GetCustomerOrdersQuery(int CustomerId);

// Query Handler (reads from read model/cache)
public class GetOrderQueryHandler
{
    private readonly IOrderReadRepository _readRepository;
    
    public async Task<OrderDto?> HandleAsync(GetOrderQuery query)
    {
        // Read from optimized read model
        return await _readRepository.GetByIdAsync(query.OrderId);
    }
}

// Read Model Updater (subscribes to events)
public class OrderReadModelUpdater : IEventHandler<OrderCreatedEvent>
{
    private readonly IOrderReadRepository _readRepository;
    
    public async Task HandleAsync(OrderCreatedEvent @event)
    {
        // Update read model
        await _readRepository.InsertAsync(new OrderReadModel
        {
            OrderId = @event.OrderId,
            CustomerId = @event.CustomerId,
            TotalAmount = @event.TotalAmount,
            Status = "Created",
            CreatedAt = @event.CreatedAt
        });
    }
}
```

---

## Saga Pattern

**Saga** = Sequence of local transactions with compensating actions

### Orchestration-Based Saga

```csharp
// Saga orchestrator coordinates all steps
public class OrderSaga
{
    private readonly IOrderService _orderService;
    private readonly IPaymentService _paymentService;
    private readonly IInventoryService _inventoryService;
    private readonly IShippingService _shippingService;
    
    public async Task<SagaResult> ExecuteAsync(CreateOrderRequest request)
    {
        var sagaState = new OrderSagaState();
        
        try
        {
            // Step 1: Create order
            sagaState.OrderId = await _orderService.CreateOrderAsync(request);
            
            // Step 2: Reserve inventory
            await _inventoryService.ReserveAsync(sagaState.OrderId, request.Items);
            sagaState.InventoryReserved = true;
            
            // Step 3: Process payment
            await _paymentService.ProcessPaymentAsync(sagaState.OrderId, request.PaymentInfo);
            sagaState.PaymentProcessed = true;
            
            // Step 4: Schedule shipping
            await _shippingService.ScheduleShipmentAsync(sagaState.OrderId);
            sagaState.ShippingScheduled = true;
            
            return SagaResult.Success(sagaState.OrderId);
        }
        catch (Exception ex)
        {
            // Compensate (rollback)
            await CompensateAsync(sagaState);
            return SagaResult.Failure(ex.Message);
        }
    }
    
    private async Task CompensateAsync(OrderSagaState state)
    {
        // Undo in reverse order
        if (state.ShippingScheduled)
            await _shippingService.CancelShipmentAsync(state.OrderId);
        
        if (state.PaymentProcessed)
            await _paymentService.RefundPaymentAsync(state.OrderId);
        
        if (state.InventoryReserved)
            await _inventoryService.ReleaseReservationAsync(state.OrderId);
        
        if (state.OrderId > 0)
            await _orderService.CancelOrderAsync(state.OrderId);
    }
}

public class OrderSagaState
{
    public int OrderId { get; set; }
    public bool InventoryReserved { get; set; }
    public bool PaymentProcessed { get; set; }
    public bool ShippingScheduled { get; set; }
}
```

### Choreography-Based Saga

```csharp
// Each service reacts to events and publishes new events

// Order Service
public class OrderService
{
    public async Task CreateOrderAsync(CreateOrderRequest request)
    {
        var order = await SaveOrderAsync(request);
        
        // Publish event
        await _eventPublisher.PublishAsync(new OrderCreatedEvent
        {
            OrderId = order.Id,
            Items = order.Items
        });
    }
}

// Inventory Service (reacts to OrderCreated)
public class InventoryEventHandler : IEventHandler<OrderCreatedEvent>
{
    public async Task HandleAsync(OrderCreatedEvent @event)
    {
        try
        {
            await ReserveInventoryAsync(@event.Items);
            
            // Publish success
            await _eventPublisher.PublishAsync(new InventoryReservedEvent
            {
                OrderId = @event.OrderId
            });
        }
        catch
        {
            // Publish failure
            await _eventPublisher.PublishAsync(new InventoryReservationFailedEvent
            {
                OrderId = @event.OrderId
            });
        }
    }
}

// Payment Service (reacts to InventoryReserved)
public class PaymentEventHandler : IEventHandler<InventoryReservedEvent>
{
    public async Task HandleAsync(InventoryReservedEvent @event)
    {
        try
        {
            await ProcessPaymentAsync(@event.OrderId);
            
            await _eventPublisher.PublishAsync(new PaymentProcessedEvent
            {
                OrderId = @event.OrderId
            });
        }
        catch
        {
            // Publish failure - triggers compensation
            await _eventPublisher.PublishAsync(new PaymentFailedEvent
            {
                OrderId = @event.OrderId
            });
        }
    }
}

// Order Service (handles compensation)
public class OrderCompensationHandler : IEventHandler<PaymentFailedEvent>
{
    public async Task HandleAsync(PaymentFailedEvent @event)
    {
        // Cancel order
        await CancelOrderAsync(@event.OrderId);
        
        // Publish to trigger other compensations
        await _eventPublisher.PublishAsync(new OrderCancelledEvent
        {
            OrderId = @event.OrderId
        });
    }
}
```

---

## Outbox Pattern

**Problem:** Ensure database update and event publishing are atomic

**Solution:** Store events in database, then publish in separate process

```csharp
// Outbox table
public class OutboxMessage
{
    public Guid Id { get; set; }
    public string EventType { get; set; } = string.Empty;
    public string EventData { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
    public bool Published { get; set; }
    public DateTime? PublishedAt { get; set; }
}

// Service with outbox
public class OrderService
{
    private readonly IDbConnection _connection;
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        using var transaction = _connection.BeginTransaction();
        
        try
        {
            // 1. Save order
            var order = await SaveOrderAsync(request, transaction);
            
            // 2. Save event to outbox (same transaction)
            var @event = new OrderCreatedEvent
            {
                OrderId = order.Id,
                CustomerId = order.CustomerId,
                TotalAmount = order.TotalAmount
            };
            
            await SaveToOutboxAsync(@event, transaction);
            
            transaction.Commit();
            return order;
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
    
    private async Task SaveToOutboxAsync(object @event, IDbTransaction transaction)
    {
        await _connection.ExecuteAsync(@"
            INSERT INTO OutboxMessages (Id, EventType, EventData, CreatedAt, Published)
            VALUES (@Id, @EventType, @EventData, @CreatedAt, @Published)",
            new
            {
                Id = Guid.NewGuid(),
                EventType = @event.GetType().Name,
                EventData = JsonSerializer.Serialize(@event),
                CreatedAt = DateTime.UtcNow,
                Published = false
            },
            transaction);
    }
}

// Background service to publish outbox messages
public class OutboxPublisher : BackgroundService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<OutboxPublisher> _logger;
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                await PublishPendingMessagesAsync();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error publishing outbox messages");
            }
            
            await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
        }
    }
    
    private async Task PublishPendingMessagesAsync()
    {
        using var scope = _serviceProvider.CreateScope();
        var connection = scope.ServiceProvider.GetRequiredService<IDbConnection>();
        var eventPublisher = scope.ServiceProvider.GetRequiredService<IEventPublisher>();
        
        // Get unpublished messages
        var messages = await connection.QueryAsync<OutboxMessage>(@"
            SELECT TOP 100 *
            FROM OutboxMessages
            WHERE Published = 0
            ORDER BY CreatedAt");
        
        foreach (var message in messages)
        {
            try
            {
                // Publish event
                var eventType = Type.GetType(message.EventType);
                var @event = JsonSerializer.Deserialize(message.EventData, eventType!);
                await eventPublisher.PublishAsync(@event);
                
                // Mark as published
                await connection.ExecuteAsync(@"
                    UPDATE OutboxMessages
                    SET Published = 1, PublishedAt = @PublishedAt
                    WHERE Id = @Id",
                    new { message.Id, PublishedAt = DateTime.UtcNow });
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Failed to publish message {MessageId}", message.Id);
            }
        }
    }
}
```

---

## Idempotency

**Problem:** Message might be delivered multiple times

**Solution:** Ensure processing is idempotent

```csharp
// Track processed messages
public class MessageTracker
{
    private readonly IDbConnection _connection;
    
    public async Task<bool> IsProcessedAsync(string messageId)
    {
        var exists = await _connection.ExecuteScalarAsync<bool>(@"
            SELECT CAST(CASE WHEN EXISTS(
                SELECT 1 FROM ProcessedMessages WHERE MessageId = @MessageId
            ) THEN 1 ELSE 0 END AS BIT)",
            new { MessageId = messageId });
        
        return exists;
    }
    
    public async Task MarkAsProcessedAsync(string messageId)
    {
        await _connection.ExecuteAsync(@"
            INSERT INTO ProcessedMessages (MessageId, ProcessedAt)
            VALUES (@MessageId, @ProcessedAt)",
            new { MessageId = messageId, ProcessedAt = DateTime.UtcNow });
    }
}

// Idempotent consumer
public class IdempotentOrderProcessor
{
    private readonly MessageTracker _messageTracker;
    private readonly IOrderProcessor _orderProcessor;
    
    public async Task ProcessAsync(string messageId, OrderCreatedEvent @event)
    {
        // Check if already processed
        if (await _messageTracker.IsProcessedAsync(messageId))
        {
            _logger.LogInformation("Message {MessageId} already processed, skipping", messageId);
            return;
        }
        
        // Process
        await _orderProcessor.ProcessOrderAsync(@event);
        
        // Mark as processed
        await _messageTracker.MarkAsProcessedAsync(messageId);
    }
}
```

---

## Dead Letter Queue (DLQ)

**Purpose:** Store messages that failed processing

```csharp
public class MessageProcessor
{
    private readonly int _maxRetries = 3;
    
    public async Task ProcessMessageAsync(Message message)
    {
        try
        {
            await ProcessAsync(message);
            await AcknowledgeAsync(message);
        }
        catch (Exception ex)
        {
            message.RetryCount++;
            
            if (message.RetryCount >= _maxRetries)
            {
                // Move to dead letter queue
                await MoveToDeadLetterQueueAsync(message, ex);
                await AcknowledgeAsync(message);
            }
            else
            {
                // Requeue for retry
                await RequeueAsync(message);
            }
        }
    }
    
    private async Task MoveToDeadLetterQueueAsync(Message message, Exception ex)
    {
        await _deadLetterQueue.SendAsync(new DeadLetterMessage
        {
            OriginalMessage = message,
            Error = ex.Message,
            StackTrace = ex.StackTrace,
            FailedAt = DateTime.UtcNow,
            RetryCount = message.RetryCount
        });
        
        _logger.LogError(ex, 
            "Message {MessageId} moved to DLQ after {RetryCount} retries",
            message.Id, message.RetryCount);
    }
}
```

---

## Best Practices

### 1. Message Design

```csharp
// ✅ Good - Immutable event with all necessary data
public record OrderCreatedEvent(
    int OrderId,
    int CustomerId,
    decimal TotalAmount,
    DateTime CreatedAt,
    Guid CorrelationId);

// ❌ Bad - Mutable, missing context
public class OrderEvent
{
    public int OrderId { get; set; }
}
```

### 2. Error Handling

```csharp
// ✅ Good - Retry with exponential backoff
public async Task ProcessWithRetryAsync(Message message)
{
    int retryCount = 0;
    int maxRetries = 3;
    
    while (retryCount < maxRetries)
    {
        try
        {
            await ProcessAsync(message);
            return;
        }
        catch (TransientException ex)
        {
            retryCount++;
            var delay = TimeSpan.FromSeconds(Math.Pow(2, retryCount));
            _logger.LogWarning(ex, "Retry {Count}/{Max} after {Delay}s", 
                retryCount, maxRetries, delay.TotalSeconds);
            await Task.Delay(delay);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Non-transient error, moving to DLQ");
            await MoveToDeadLetterQueueAsync(message, ex);
            return;
        }
    }
    
    await MoveToDeadLetterQueueAsync(message, 
        new Exception($"Failed after {maxRetries} retries"));
}
```

### 3. Monitoring

```csharp
public class MonitoredMessageProcessor
{
    private readonly IMetrics _metrics;
    
    public async Task ProcessAsync(Message message)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            await ProcessMessageAsync(message);
            
            stopwatch.Stop();
            _metrics.RecordMessageProcessed(stopwatch.ElapsedMilliseconds);
            _logger.LogInformation(
                "Message {MessageId} processed in {Ms}ms",
                message.Id, stopwatch.ElapsedMilliseconds);
        }
        catch (Exception ex)
        {
            _metrics.RecordMessageFailed();
            _logger.LogError(ex, "Failed to process message {MessageId}", message.Id);
            throw;
        }
    }
}
```

### 4. Versioning

```csharp
// Version events for backward compatibility
public record OrderCreatedEventV1(
    int OrderId,
    int CustomerId,
    decimal TotalAmount);

public record OrderCreatedEventV2(
    int OrderId,
    int CustomerId,
    decimal TotalAmount,
    string Currency,
    DateTime CreatedAt);

// Handler supports both versions
public class OrderEventHandler
{
    public async Task HandleAsync(object @event)
    {
        switch (@event)
        {
            case OrderCreatedEventV2 v2:
                await ProcessV2Async(v2);
                break;
                
            case OrderCreatedEventV1 v1:
                // Convert v1 to v2
                var v2Event = new OrderCreatedEventV2(
                    v1.OrderId,
                    v1.CustomerId,
                    v1.TotalAmount,
                    "USD", // Default
                    DateTime.UtcNow);
                await ProcessV2Async(v2Event);
                break;
        }
    }
}
```

### 5. Testing

```csharp
[Fact]
public async Task OrderCreatedEvent_ShouldSendEmail()
{
    // Arrange
    var @event = new OrderCreatedEvent(1, 100, 50.00m, DateTime.UtcNow, Guid.NewGuid());
    var emailService = new Mock<IEmailService>();
    var handler = new EmailNotificationHandler(emailService.Object);
    
    // Act
    await handler.HandleAsync(@event);
    
    // Assert
    emailService.Verify(
        x => x.SendOrderConfirmationAsync(100, 1),
        Times.Once);
}

[Fact]
public async Task ProcessMessage_ShouldBeIdempotent()
{
    // Arrange
    var messageId = Guid.NewGuid().ToString();
    var @event = new OrderCreatedEvent(1, 100, 50.00m, DateTime.UtcNow, Guid.NewGuid());
    
    // Act - Process twice
    await _processor.ProcessAsync(messageId, @event);
    await _processor.ProcessAsync(messageId, @event);
    
    // Assert - Should only process once
    var processedCount = await GetProcessedCountAsync(@event.OrderId);
    Assert.Equal(1, processedCount);
}
```

---

## Comparison: RabbitMQ vs Azure Service Bus

| Feature | RabbitMQ | Azure Service Bus |
|---------|----------|-------------------|
| **Hosting** | Self-hosted / CloudAMQP | Fully managed |
| **Protocol** | AMQP, STOMP, MQTT | AMQP |
| **Message Size** | Limited by config | 256 KB (Standard), 1 MB (Premium) |
| **Ordering** | Per queue | Per session |
| **Transactions** | Yes | Yes |
| **Dead Letter Queue** | Yes | Yes |
| **Topics** | Yes (exchanges) | Yes |
| **Cost** | Infrastructure cost | Pay per operation |
| **Scaling** | Manual | Automatic |
| **Reliability** | Depends on setup | 99.9% SLA |
| **Best For** | On-premise, full control | Cloud-native, managed |

---

## Quick Reference: When to Use What

| Scenario | Pattern/Tool |
|----------|--------------|
| Simple async processing | Queue (RabbitMQ/Service Bus) |
| Multiple consumers | Topic/Fanout |
| Workflow orchestration | Saga (Orchestration) |
| Loosely coupled services | Saga (Choreography) |
| Event history | Event Sourcing |
| Read/Write separation | CQRS |
| Guaranteed delivery | Outbox Pattern |
| Idempotent processing | Message tracking |
| Failed messages | Dead Letter Queue |
| On-premise | RabbitMQ |
| Cloud-native | Azure Service Bus |
| Multiple protocols | RabbitMQ |
| Managed service | Azure Service Bus |

---

## Event-Driven Architecture Checklist

### Design
- [ ] Events are immutable
- [ ] Events contain all necessary data
- [ ] Events have unique IDs
- [ ] Events include correlation IDs
- [ ] Events are versioned
- [ ] Consider backward compatibility

### Implementation
- [ ] Use message queues for reliability
- [ ] Implement idempotency
- [ ] Use outbox pattern for atomicity
- [ ] Handle failures gracefully
- [ ] Implement retry with backoff
- [ ] Use dead letter queues

### Monitoring
- [ ] Track message processing time
- [ ] Monitor queue depth
- [ ] Alert on failures
- [ ] Log correlation IDs
- [ ] Track message flow

### Testing
- [ ] Test idempotency
- [ ] Test failure scenarios
- [ ] Test compensation logic
- [ ] Load test message processing
- [ ] Test with delayed messages

---

**🎉 Guide Complete! 🎉**

This comprehensive Message Queues & Event-Driven Architecture guide covers RabbitMQ, Azure Service Bus, messaging patterns, CQRS, Event Sourcing, Sagas, Outbox Pattern, idempotency, and best practices for building distributed, event-driven systems!

**🏆 ALL 12 GUIDES COMPLETE! 🏆**

You now have a complete, comprehensive collection of backend development guides covering everything you need for .NET developer interviews and real-world development! 🚀
