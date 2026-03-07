# Azure Fundamentals Quick Reference

---

## What is Microsoft Azure?

**Azure** = Microsoft's cloud computing platform offering 200+ services

**Key Benefits:**
- ☁️ **Scalability** - Scale up/down on demand
- 💰 **Cost-effective** - Pay only for what you use
- 🌍 **Global reach** - 60+ regions worldwide
- 🔒 **Security** - Enterprise-grade security
- 🚀 **Fast deployment** - Minutes instead of weeks
- 🔄 **High availability** - Built-in redundancy

**Common Services for .NET Developers:**
- Azure App Service (Web Apps)
- Azure SQL Database
- Azure Storage (Blob, Table, Queue)
- Azure Functions (Serverless)
- Azure Key Vault (Secrets)
- Azure Service Bus (Messaging)
- Azure Container Instances/AKS

---

## Azure App Service

**Azure App Service** = Fully managed platform for building web apps

### Deployment via Azure CLI

```bash
# Login to Azure
az login

# Create resource group
az group create --name MyResourceGroup --location eastus

# Create App Service plan
az appservice plan create \
  --name MyAppServicePlan \
  --resource-group MyResourceGroup \
  --sku B1 \
  --is-linux

# Create web app
az webapp create \
  --name MyUniqueAppName \
  --resource-group MyResourceGroup \
  --plan MyAppServicePlan \
  --runtime "DOTNET|8.0"

# Deploy from local folder
az webapp deploy \
  --resource-group MyResourceGroup \
  --name MyUniqueAppName \
  --src-path ./publish.zip \
  --type zip
```

### Deployment via Visual Studio

```csharp
// Right-click project → Publish
// Select Azure → Azure App Service (Windows/Linux)
// Create new or select existing
// Publish
```

### Configuration

```csharp
// Program.cs - Read from Azure App Service Configuration
var builder = WebApplication.CreateBuilder(args);

// Connection strings
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");

// App settings
var apiKey = builder.Configuration["ApiKey"];
var setting = builder.Configuration["MyApp:Setting"];
```

**Azure Portal Configuration:**
```
App Service → Configuration → Application settings
- Name: ApiKey
- Value: your-api-key

App Service → Configuration → Connection strings
- Name: DefaultConnection
- Value: Server=...
- Type: SQLAzure
```

### Scaling

```bash
# Scale up (vertical) - Change plan
az appservice plan update \
  --name MyAppServicePlan \
  --resource-group MyResourceGroup \
  --sku P1V2

# Scale out (horizontal) - Add instances
az appservice plan update \
  --name MyAppServicePlan \
  --resource-group MyResourceGroup \
  --number-of-workers 3

# Enable autoscale
az monitor autoscale create \
  --resource-group MyResourceGroup \
  --resource MyAppServicePlan \
  --resource-type Microsoft.Web/serverfarms \
  --name autoscale-settings \
  --min-count 1 \
  --max-count 10 \
  --count 2
```

### Deployment Slots

```bash
# Create staging slot
az webapp deployment slot create \
  --name MyUniqueAppName \
  --resource-group MyResourceGroup \
  --slot staging

# Deploy to staging
az webapp deploy \
  --resource-group MyResourceGroup \
  --name MyUniqueAppName \
  --slot staging \
  --src-path ./publish.zip

# Swap staging to production
az webapp deployment slot swap \
  --resource-group MyResourceGroup \
  --name MyUniqueAppName \
  --slot staging \
  --target-slot production
```

---

## Azure SQL Database

### Creating Database

```bash
# Create SQL Server
az sql server create \
  --name myuniquesqlserver \
  --resource-group MyResourceGroup \
  --location eastus \
  --admin-user sqladmin \
  --admin-password YourPassword123!

# Configure firewall
az sql server firewall-rule create \
  --resource-group MyResourceGroup \
  --server myuniquesqlserver \
  --name AllowAllAzureIPs \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Create database
az sql db create \
  --resource-group MyResourceGroup \
  --server myuniquesqlserver \
  --name MyDatabase \
  --service-objective S0
```

### Connection String

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=tcp:myuniquesqlserver.database.windows.net,1433;Initial Catalog=MyDatabase;Persist Security Info=False;User ID=sqladmin;Password=YourPassword123!;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
  }
}
```

### Using in .NET

```csharp
// Program.cs
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// With retry policy
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        sqlServerOptions => sqlServerOptions.EnableRetryOnFailure(
            maxRetryCount: 5,
            maxRetryDelay: TimeSpan.FromSeconds(30),
            errorNumbersToAdd: null)));
```

### Managed Identity Authentication

```csharp
// No password needed!
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
    var credential = new DefaultAzureCredential();
    var token = credential.GetToken(
        new TokenRequestContext(new[] { "https://database.windows.net/.default" }));
    
    var connection = new SqlConnection(connectionString)
    {
        AccessToken = token.Token
    };
    
    options.UseSqlServer(connection);
});
```

---

## Azure Storage

### Blob Storage (Files)

```bash
# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group MyResourceGroup \
  --location eastus \
  --sku Standard_LRS

# Create container
az storage container create \
  --name mycontainer \
  --account-name mystorageaccount
```

**Using in .NET:**

```csharp
// Install package
// dotnet add package Azure.Storage.Blobs

using Azure.Storage.Blobs;

public class BlobStorageService
{
    private readonly BlobServiceClient _blobServiceClient;
    
    public BlobStorageService(IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("AzureStorage");
        _blobServiceClient = new BlobServiceClient(connectionString);
    }
    
    // Upload file
    public async Task<string> UploadFileAsync(string containerName, string fileName, Stream fileStream)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        await containerClient.CreateIfNotExistsAsync();
        
        var blobClient = containerClient.GetBlobClient(fileName);
        await blobClient.UploadAsync(fileStream, overwrite: true);
        
        return blobClient.Uri.ToString();
    }
    
    // Download file
    public async Task<Stream> DownloadFileAsync(string containerName, string fileName)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        var blobClient = containerClient.GetBlobClient(fileName);
        
        var response = await blobClient.DownloadAsync();
        return response.Value.Content;
    }
    
    // Delete file
    public async Task DeleteFileAsync(string containerName, string fileName)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        var blobClient = containerClient.GetBlobClient(fileName);
        await blobClient.DeleteIfExistsAsync();
    }
    
    // List files
    public async Task<List<string>> ListFilesAsync(string containerName)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        var blobs = new List<string>();
        
        await foreach (var blobItem in containerClient.GetBlobsAsync())
        {
            blobs.Add(blobItem.Name);
        }
        
        return blobs;
    }
}

// Controller usage
[ApiController]
[Route("api/[controller]")]
public class FilesController : ControllerBase
{
    private readonly BlobStorageService _blobService;
    
    [HttpPost("upload")]
    public async Task<ActionResult<string>> Upload(IFormFile file)
    {
        if (file == null || file.Length == 0)
            return BadRequest("No file uploaded");
        
        using var stream = file.OpenReadStream();
        var url = await _blobService.UploadFileAsync("uploads", file.FileName, stream);
        
        return Ok(new { url });
    }
    
    [HttpGet("{fileName}")]
    public async Task<IActionResult> Download(string fileName)
    {
        var stream = await _blobService.DownloadFileAsync("uploads", fileName);
        return File(stream, "application/octet-stream", fileName);
    }
}
```

### Table Storage (NoSQL)

```csharp
// dotnet add package Azure.Data.Tables

using Azure.Data.Tables;

public class CustomerEntity : ITableEntity
{
    public string PartitionKey { get; set; } = default!;
    public string RowKey { get; set; } = default!;
    public DateTimeOffset? Timestamp { get; set; }
    public ETag ETag { get; set; }
    
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}

public class TableStorageService
{
    private readonly TableClient _tableClient;
    
    public TableStorageService(IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("AzureStorage");
        var serviceClient = new TableServiceClient(connectionString);
        _tableClient = serviceClient.GetTableClient("customers");
        _tableClient.CreateIfNotExists();
    }
    
    // Insert
    public async Task AddCustomerAsync(CustomerEntity customer)
    {
        await _tableClient.AddEntityAsync(customer);
    }
    
    // Get
    public async Task<CustomerEntity?> GetCustomerAsync(string partitionKey, string rowKey)
    {
        try
        {
            var response = await _tableClient.GetEntityAsync<CustomerEntity>(partitionKey, rowKey);
            return response.Value;
        }
        catch (RequestFailedException ex) when (ex.Status == 404)
        {
            return null;
        }
    }
    
    // Update
    public async Task UpdateCustomerAsync(CustomerEntity customer)
    {
        await _tableClient.UpdateEntityAsync(customer, customer.ETag);
    }
    
    // Delete
    public async Task DeleteCustomerAsync(string partitionKey, string rowKey)
    {
        await _tableClient.DeleteEntityAsync(partitionKey, rowKey);
    }
    
    // Query
    public async Task<List<CustomerEntity>> QueryCustomersAsync(string partitionKey)
    {
        var customers = new List<CustomerEntity>();
        
        await foreach (var customer in _tableClient.QueryAsync<CustomerEntity>(
            c => c.PartitionKey == partitionKey))
        {
            customers.Add(customer);
        }
        
        return customers;
    }
}
```

### Queue Storage (Messaging)

```csharp
// dotnet add package Azure.Storage.Queues

using Azure.Storage.Queues;

public class QueueService
{
    private readonly QueueClient _queueClient;
    
    public QueueService(IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("AzureStorage");
        _queueClient = new QueueClient(connectionString, "orders");
        _queueClient.CreateIfNotExists();
    }
    
    // Send message
    public async Task SendMessageAsync(string message)
    {
        await _queueClient.SendMessageAsync(message);
    }
    
    // Receive message
    public async Task<string?> ReceiveMessageAsync()
    {
        var response = await _queueClient.ReceiveMessageAsync();
        
        if (response.Value != null)
        {
            var message = response.Value;
            
            // Process message
            var messageText = message.MessageText;
            
            // Delete message after processing
            await _queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
            
            return messageText;
        }
        
        return null;
    }
    
    // Peek messages (without removing)
    public async Task<List<string>> PeekMessagesAsync(int maxMessages = 10)
    {
        var messages = new List<string>();
        var peekedMessages = await _queueClient.PeekMessagesAsync(maxMessages);
        
        foreach (var message in peekedMessages.Value)
        {
            messages.Add(message.MessageText);
        }
        
        return messages;
    }
}
```

---

## Azure Functions (Serverless)

### HTTP Trigger Function

```csharp
// dotnet new func -n MyFunctionApp
// dotnet add package Microsoft.Azure.Functions.Worker

using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;

public class HttpTriggerFunction
{
    [Function("GetCustomer")]
    public async Task<HttpResponseData> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get", Route = "customers/{id}")] 
        HttpRequestData req,
        string id)
    {
        // Your logic here
        var customer = await GetCustomerAsync(id);
        
        var response = req.CreateResponse(System.Net.HttpStatusCode.OK);
        await response.WriteAsJsonAsync(customer);
        
        return response;
    }
}
```

### Timer Trigger Function

```csharp
public class TimerTriggerFunction
{
    private readonly ILogger<TimerTriggerFunction> _logger;
    
    public TimerTriggerFunction(ILogger<TimerTriggerFunction> logger)
    {
        _logger = logger;
    }
    
    [Function("CleanupFunction")]
    public async Task Run(
        [TimerTrigger("0 0 2 * * *")] TimerInfo timerInfo) // Every day at 2 AM
    {
        _logger.LogInformation("Cleanup function started at {Time}", DateTime.UtcNow);
        
        // Cleanup logic
        await CleanupOldDataAsync();
        
        _logger.LogInformation("Cleanup function completed");
    }
}
```

### Queue Trigger Function

```csharp
public class QueueTriggerFunction
{
    [Function("ProcessOrder")]
    public async Task Run(
        [QueueTrigger("orders", Connection = "AzureWebJobsStorage")] 
        string orderMessage)
    {
        // Process order from queue
        var order = JsonSerializer.Deserialize<Order>(orderMessage);
        await ProcessOrderAsync(order);
    }
}
```

### Blob Trigger Function

```csharp
public class BlobTriggerFunction
{
    [Function("ProcessImage")]
    public async Task Run(
        [BlobTrigger("images/{name}", Connection = "AzureWebJobsStorage")] 
        Stream imageStream,
        string name)
    {
        // Process image
        var processedImage = await ProcessImageAsync(imageStream);
        
        // Save to another container
        await SaveProcessedImageAsync(name, processedImage);
    }
}
```

---

## Azure Key Vault

**Azure Key Vault** = Secure storage for secrets, keys, and certificates

### Setup

```bash
# Create Key Vault
az keyvault create \
  --name MyUniqueKeyVault \
  --resource-group MyResourceGroup \
  --location eastus

# Add secret
az keyvault secret set \
  --vault-name MyUniqueKeyVault \
  --name ConnectionString \
  --value "Server=..."

# Grant access to App Service
az keyvault set-policy \
  --name MyUniqueKeyVault \
  --object-id <app-service-managed-identity-id> \
  --secret-permissions get list
```

### Using in .NET

```csharp
// dotnet add package Azure.Identity
// dotnet add package Azure.Extensions.AspNetCore.Configuration.Secrets

// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add Key Vault
var keyVaultUrl = new Uri($"https://{builder.Configuration["KeyVaultName"]}.vault.azure.net/");
builder.Configuration.AddAzureKeyVault(keyVaultUrl, new DefaultAzureCredential());

// Now secrets are available like any configuration
var connectionString = builder.Configuration["ConnectionString"];
var apiKey = builder.Configuration["ApiKey"];
```

### Direct Access to Key Vault

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

public class KeyVaultService
{
    private readonly SecretClient _secretClient;
    
    public KeyVaultService(IConfiguration configuration)
    {
        var keyVaultUrl = new Uri($"https://{configuration["KeyVaultName"]}.vault.azure.net/");
        _secretClient = new SecretClient(keyVaultUrl, new DefaultAzureCredential());
    }
    
    public async Task<string> GetSecretAsync(string secretName)
    {
        var secret = await _secretClient.GetSecretAsync(secretName);
        return secret.Value.Value;
    }
    
    public async Task SetSecretAsync(string secretName, string secretValue)
    {
        await _secretClient.SetSecretAsync(secretName, secretValue);
    }
}
```

---

## Azure Service Bus

**Azure Service Bus** = Enterprise messaging service for reliable message delivery

### Setup

```bash
# Create Service Bus namespace
az servicebus namespace create \
  --name MyServiceBusNamespace \
  --resource-group MyResourceGroup \
  --location eastus \
  --sku Standard

# Create queue
az servicebus queue create \
  --namespace-name MyServiceBusNamespace \
  --resource-group MyResourceGroup \
  --name orders

# Create topic
az servicebus topic create \
  --namespace-name MyServiceBusNamespace \
  --resource-group MyResourceGroup \
  --name order-events

# Create subscription
az servicebus topic subscription create \
  --namespace-name MyServiceBusNamespace \
  --resource-group MyResourceGroup \
  --topic-name order-events \
  --name email-service
```

### Using Service Bus Queue

```csharp
// dotnet add package Azure.Messaging.ServiceBus

using Azure.Messaging.ServiceBus;

public class ServiceBusService
{
    private readonly ServiceBusClient _client;
    private readonly ServiceBusSender _sender;
    private readonly ServiceBusProcessor _processor;
    
    public ServiceBusService(IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("ServiceBus");
        _client = new ServiceBusClient(connectionString);
        _sender = _client.CreateSender("orders");
        _processor = _client.CreateProcessor("orders");
    }
    
    // Send message
    public async Task SendOrderAsync(Order order)
    {
        var message = new ServiceBusMessage(JsonSerializer.Serialize(order))
        {
            ContentType = "application/json",
            Subject = "OrderCreated",
            MessageId = Guid.NewGuid().ToString()
        };
        
        await _sender.SendMessageAsync(message);
    }
    
    // Send batch
    public async Task SendOrderBatchAsync(List<Order> orders)
    {
        using var messageBatch = await _sender.CreateMessageBatchAsync();
        
        foreach (var order in orders)
        {
            var message = new ServiceBusMessage(JsonSerializer.Serialize(order));
            
            if (!messageBatch.TryAddMessage(message))
            {
                throw new Exception("Message too large for batch");
            }
        }
        
        await _sender.SendMessagesAsync(messageBatch);
    }
    
    // Receive messages
    public async Task StartProcessingAsync()
    {
        _processor.ProcessMessageAsync += MessageHandler;
        _processor.ProcessErrorAsync += ErrorHandler;
        
        await _processor.StartProcessingAsync();
    }
    
    private async Task MessageHandler(ProcessMessageEventArgs args)
    {
        var order = JsonSerializer.Deserialize<Order>(args.Message.Body.ToString());
        
        // Process order
        await ProcessOrderAsync(order);
        
        // Complete message
        await args.CompleteMessageAsync(args.Message);
    }
    
    private Task ErrorHandler(ProcessErrorEventArgs args)
    {
        Console.WriteLine($"Error: {args.Exception}");
        return Task.CompletedTask;
    }
}
```

### Using Service Bus Topic/Subscription

```csharp
public class ServiceBusTopicService
{
    private readonly ServiceBusClient _client;
    
    public ServiceBusTopicService(IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("ServiceBus");
        _client = new ServiceBusClient(connectionString);
    }
    
    // Publish to topic
    public async Task PublishOrderEventAsync(OrderEvent orderEvent)
    {
        var sender = _client.CreateSender("order-events");
        
        var message = new ServiceBusMessage(JsonSerializer.Serialize(orderEvent))
        {
            Subject = orderEvent.EventType,
            ApplicationProperties =
            {
                ["OrderId"] = orderEvent.OrderId,
                ["CustomerId"] = orderEvent.CustomerId
            }
        };
        
        await sender.SendMessageAsync(message);
    }
    
    // Subscribe to topic
    public async Task SubscribeToOrderEventsAsync()
    {
        var processor = _client.CreateProcessor("order-events", "email-service");
        
        processor.ProcessMessageAsync += async args =>
        {
            var orderEvent = JsonSerializer.Deserialize<OrderEvent>(args.Message.Body.ToString());
            
            // Handle event
            if (args.Message.Subject == "OrderCreated")
            {
                await SendOrderConfirmationEmailAsync(orderEvent);
            }
            
            await args.CompleteMessageAsync(args.Message);
        };
        
        processor.ProcessErrorAsync += args =>
        {
            Console.WriteLine($"Error: {args.Exception}");
            return Task.CompletedTask;
        };
        
        await processor.StartProcessingAsync();
    }
}
```

---

## Azure Container Instances (ACI)

**Azure Container Instances** = Run containers without managing servers

```bash
# Create container instance
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --dns-name-label mycontainer-unique \
  --ports 80

# Create from Docker Hub
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myusername/myapp:latest \
  --registry-username myusername \
  --registry-password mypassword \
  --dns-name-label myapp-unique \
  --ports 80 \
  --environment-variables \
    ASPNETCORE_ENVIRONMENT=Production \
    ConnectionStrings__DefaultConnection="Server=..."

# View logs
az container logs \
  --resource-group MyResourceGroup \
  --name mycontainer

# Delete container
az container delete \
  --resource-group MyResourceGroup \
  --name mycontainer
```

---

## Azure Kubernetes Service (AKS)

**AKS** = Managed Kubernetes for container orchestration

```bash
# Create AKS cluster
az aks create \
  --resource-group MyResourceGroup \
  --name MyAKSCluster \
  --node-count 2 \
  --enable-addons monitoring \
  --generate-ssh-keys

# Get credentials
az aks get-credentials \
  --resource-group MyResourceGroup \
  --name MyAKSCluster

# Deploy application
kubectl apply -f deployment.yaml

# Get services
kubectl get services

# Scale deployment
kubectl scale deployment myapp --replicas=5

# View logs
kubectl logs -f deployment/myapp
```

**deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry.azurecr.io/myapp:latest
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
---
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: myapp
```

---

## Azure Active Directory (Azure AD)

### Authentication in ASP.NET Core

```csharp
// dotnet add package Microsoft.Identity.Web

// Program.cs
using Microsoft.Identity.Web;

var builder = WebApplication.CreateBuilder(args);

// Add Azure AD authentication
builder.Services.AddMicrosoftIdentityWebAppAuthentication(builder.Configuration);

// Or for APIs
builder.Services.AddMicrosoftIdentityWebApiAuthentication(builder.Configuration);

builder.Services.AddControllers();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

**appsettings.json:**
```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "yourdomain.onmicrosoft.com",
    "TenantId": "your-tenant-id",
    "ClientId": "your-client-id",
    "ClientSecret": "your-client-secret",
    "CallbackPath": "/signin-oidc"
  }
}
```

### Protected API Endpoint

```csharp
[Authorize]
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetCustomers()
    {
        var userId = User.FindFirst("sub")?.Value;
        var userName = User.Identity?.Name;
        
        // Your logic
        return Ok();
    }
    
    [Authorize(Roles = "Admin")]
    [HttpDelete("{id}")]
    public IActionResult DeleteCustomer(int id)
    {
        // Only admins can delete
        return NoContent();
    }
}
```

---

## Cost Optimization

### 1. Right-sizing Resources

```bash
# Check current pricing tier
az appservice plan show \
  --name MyAppServicePlan \
  --resource-group MyResourceGroup \
  --query sku.name

# Scale down when not needed
az appservice plan update \
  --name MyAppServicePlan \
  --resource-group MyResourceGroup \
  --sku B1  # Basic tier

# Use reserved instances for predictable workloads (up to 72% savings)
```

### 2. Use Appropriate Storage Tiers

```bash
# Hot tier: Frequently accessed data
# Cool tier: Infrequently accessed (30 days+)
# Archive tier: Rarely accessed (180 days+)

az storage blob set-tier \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --tier Cool
```

### 3. Enable Autoscaling

```csharp
// Scale based on metrics
// Azure Portal → App Service → Scale out (App Service plan)
// Enable autoscale
// Rules:
// - If CPU > 70%, scale out
// - If CPU < 30%, scale in
// Min instances: 1
// Max instances: 5
```

### 4. Delete Unused Resources

```bash
# List all resources in resource group
az resource list --resource-group MyResourceGroup --output table

# Delete unused resources
az resource delete --ids <resource-id>

# Delete entire resource group (careful!)
az group delete --name MyResourceGroup
```

---

## Monitoring and Diagnostics

### Application Insights

```csharp
// Already covered in Logging guide
// Automatic telemetry collection
// Custom events, metrics, exceptions
// Performance monitoring
// Live metrics
```

### Azure Monitor

```bash
# View metrics
az monitor metrics list \
  --resource <resource-id> \
  --metric-names Percentage CPU \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z

# Create alert
az monitor metrics alert create \
  --name HighCPUAlert \
  --resource-group MyResourceGroup \
  --scopes <resource-id> \
  --condition "avg Percentage CPU > 80" \
  --description "Alert when CPU exceeds 80%"
```

---

## Best Practices

### 1. Use Managed Identities

```csharp
// ✅ Good - No credentials in code
var credential = new DefaultAzureCredential();
var secretClient = new SecretClient(new Uri("https://myvault.vault.azure.net/"), credential);

// ❌ Bad - Hardcoded credentials
var connectionString = "AccountKey=...";
```

### 2. Store Secrets in Key Vault

```csharp
// ✅ Good - Secrets in Key Vault
builder.Configuration.AddAzureKeyVault(keyVaultUrl, new DefaultAzureCredential());

// ❌ Bad - Secrets in appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=...;Password=MyPassword123!"
  }
}
```

### 3. Use Resource Tags

```bash
# Tag resources for cost tracking
az resource tag \
  --tags Environment=Production Department=IT CostCenter=1234 \
  --ids <resource-id>
```

### 4. Implement Health Checks

```csharp
// Add health checks for all dependencies
builder.Services.AddHealthChecks()
    .AddSqlServer(connectionString)
    .AddAzureBlobStorage(storageConnectionString)
    .AddAzureServiceBusTopic(serviceBusConnectionString, topicName);
```

### 5. Enable Diagnostics Logging

```bash
# Enable App Service logs
az webapp log config \
  --name MyUniqueAppName \
  --resource-group MyResourceGroup \
  --application-logging filesystem \
  --detailed-error-messages true \
  --failed-request-tracing true

# Stream logs
az webapp log tail \
  --name MyUniqueAppName \
  --resource-group MyResourceGroup
```

### 6. Use Deployment Slots

```bash
# Deploy to staging first
az webapp deployment slot create --slot staging

# Test in staging
curl https://myapp-staging.azurewebsites.net

# Swap to production
az webapp deployment slot swap --slot staging
```

---

## Common Azure CLI Commands

```bash
# Login
az login

# Set subscription
az account set --subscription "My Subscription"

# List resources
az resource list --output table

# Create resource group
az group create --name MyRG --location eastus

# Delete resource group
az group delete --name MyRG

# List locations
az account list-locations --output table

# Get resource info
az resource show --ids <resource-id>

# Export template
az group export --name MyRG > template.json
```

---

## Quick Reference: Azure Service Selection

| Requirement | Service |
|-------------|---------|
| Web app hosting | App Service |
| Database | Azure SQL Database |
| File storage | Blob Storage |
| NoSQL database | Table Storage / Cosmos DB |
| Message queue | Service Bus / Storage Queue |
| Serverless functions | Azure Functions |
| Secrets management | Key Vault |
| Container hosting | Container Instances (simple) / AKS (complex) |
| Authentication | Azure AD |
| Monitoring | Application Insights |
| Static files/CDN | Azure CDN + Blob Storage |

---

**Guide Complete!** This comprehensive Azure Fundamentals guide covers essential Azure services for .NET developers including App Service, SQL Database, Storage, Functions, Key Vault, Service Bus, containers, authentication, and best practices for cloud deployment! ☁️
