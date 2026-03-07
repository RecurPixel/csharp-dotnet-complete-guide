# Docker & CI/CD Pipelines Quick Reference

---

## What is Docker?

**Docker** = Platform for developing, shipping, and running applications in containers

**Container** = Lightweight, standalone, executable package that includes everything needed to run an application

**Key Concepts:**
- ✅ **Isolation** - Each container runs independently
- ✅ **Portability** - Run anywhere (dev, test, prod)
- ✅ **Consistency** - Same environment everywhere
- ✅ **Efficiency** - Lightweight compared to VMs
- ✅ **Scalability** - Easy to scale horizontally

### Docker vs Virtual Machines

```
Virtual Machine:
┌─────────────────────────────┐
│     Application             │
├─────────────────────────────┤
│     Guest OS (GB)           │
├─────────────────────────────┤
│     Hypervisor              │
├─────────────────────────────┤
│     Host OS                 │
└─────────────────────────────┘

Docker Container:
┌─────────────────────────────┐
│     Application             │
├─────────────────────────────┤
│     Docker Engine           │
├─────────────────────────────┤
│     Host OS                 │
└─────────────────────────────┘
```

**Advantages of Docker:**
- ⚡ Faster startup (seconds vs minutes)
- 💾 Less disk space (MB vs GB)
- 🚀 Better resource utilization
- 📦 Easier to distribute

---

## Docker Installation

### Windows

```powershell
# Download Docker Desktop from docker.com
# Install and restart

# Verify installation
docker --version
docker-compose --version

# Test Docker
docker run hello-world
```

### Linux

```bash
# Update package index
sudo apt-get update

# Install Docker
sudo apt-get install docker.io

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER
newgrp docker

# Verify
docker --version
docker run hello-world
```

---

## Docker Core Concepts

### Images

**Image** = Read-only template with instructions for creating a container

```bash
# List images
docker images

# Pull image from Docker Hub
docker pull mcr.microsoft.com/dotnet/aspnet:8.0

# Build image from Dockerfile
docker build -t myapp:1.0 .

# Remove image
docker rmi myapp:1.0

# Remove unused images
docker image prune
```

### Containers

**Container** = Runnable instance of an image

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Run container
docker run myapp:1.0

# Run container with name
docker run --name my-container myapp:1.0

# Run container in background (detached)
docker run -d myapp:1.0

# Run with port mapping
docker run -p 8080:80 myapp:1.0

# Stop container
docker stop container-id

# Start stopped container
docker start container-id

# Remove container
docker rm container-id

# Remove all stopped containers
docker container prune
```

---

## Dockerfile for .NET Applications

### Basic Dockerfile

```dockerfile
# Use official .NET runtime as base image
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Use SDK image to build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApp.API/MyApp.API.csproj", "MyApp.API/"]
RUN dotnet restore "MyApp.API/MyApp.API.csproj"
COPY . .
WORKDIR "/src/MyApp.API"
RUN dotnet build "MyApp.API.csproj" -c Release -o /app/build

# Publish app
FROM build AS publish
RUN dotnet publish "MyApp.API.csproj" -c Release -o /app/publish

# Final stage - runtime image
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.API.dll"]
```

### Multi-Stage Dockerfile Explained

```dockerfile
# Stage 1: Base runtime image (smallest)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

# Stage 2: Build stage (has SDK)
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy only project files first (for caching)
COPY ["MyApp.API/MyApp.API.csproj", "MyApp.API/"]
COPY ["MyApp.Application/MyApp.Application.csproj", "MyApp.Application/"]
COPY ["MyApp.Domain/MyApp.Domain.csproj", "MyApp.Domain/"]
COPY ["MyApp.Infrastructure/MyApp.Infrastructure.csproj", "MyApp.Infrastructure/"]

# Restore dependencies
RUN dotnet restore "MyApp.API/MyApp.API.csproj"

# Copy everything else
COPY . .

# Build
WORKDIR "/src/MyApp.API"
RUN dotnet build "MyApp.API.csproj" -c Release -o /app/build

# Stage 3: Publish stage
FROM build AS publish
RUN dotnet publish "MyApp.API.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Stage 4: Final runtime stage
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.API.dll"]
```

**Why Multi-Stage?**
- ✅ Smaller final image (no SDK, only runtime)
- ✅ Build tools not in production image
- ✅ Better security
- ✅ Faster deployment

### Dockerfile with Environment Variables

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

# Set environment variables
ENV ASPNETCORE_ENVIRONMENT=Production
ENV ConnectionStrings__DefaultConnection=""

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApp.API/MyApp.API.csproj", "MyApp.API/"]
RUN dotnet restore "MyApp.API/MyApp.API.csproj"
COPY . .
WORKDIR "/src/MyApp.API"
RUN dotnet build "MyApp.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyApp.API.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.API.dll"]
```

---

## Building and Running Docker Images

### Build Image

```bash
# Build image with tag
docker build -t myapp:1.0 .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t myapp:1.0 .

# Build without cache
docker build --no-cache -t myapp:1.0 .
```

### Run Container

```bash
# Basic run
docker run myapp:1.0

# Run in background
docker run -d myapp:1.0

# Run with port mapping (host:container)
docker run -d -p 8080:80 myapp:1.0

# Run with name
docker run -d --name myapp-container -p 8080:80 myapp:1.0

# Run with environment variables
docker run -d -p 8080:80 \
  -e ASPNETCORE_ENVIRONMENT=Development \
  -e ConnectionStrings__DefaultConnection="Server=db;Database=MyDb" \
  myapp:1.0

# Run with volume mount
docker run -d -p 8080:80 \
  -v /host/path:/app/data \
  myapp:1.0

# Run with network
docker run -d -p 8080:80 \
  --network my-network \
  myapp:1.0

# Run and remove after stop
docker run --rm -p 8080:80 myapp:1.0
```

### Container Management

```bash
# View logs
docker logs container-id

# Follow logs (live)
docker logs -f container-id

# Execute command in running container
docker exec -it container-id bash

# Copy files from container
docker cp container-id:/app/logs/app.log ./app.log

# Copy files to container
docker cp ./config.json container-id:/app/config.json

# Inspect container
docker inspect container-id

# View container stats
docker stats container-id
```

---

## Docker Compose

**Docker Compose** = Tool for defining and running multi-container applications

### Basic docker-compose.yml

```yaml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=db;Database=MyDb;User=sa;Password=YourPassword123!
    depends_on:
      - db
    networks:
      - myapp-network

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourPassword123!
    ports:
      - "1433:1433"
    volumes:
      - sqldata:/var/opt/mssql
    networks:
      - myapp-network

volumes:
  sqldata:

networks:
  myapp-network:
    driver: bridge
```

### Complete docker-compose.yml with Multiple Services

```yaml
version: '3.8'

services:
  # Web API
  api:
    build:
      context: .
      dockerfile: MyApp.API/Dockerfile
    container_name: myapp-api
    ports:
      - "5000:80"
      - "5001:443"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ConnectionStrings__DefaultConnection=Server=db;Database=MyAppDb;User=sa;Password=YourStrong@Password
      - ConnectionStrings__Redis=redis:6379
    depends_on:
      - db
      - redis
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # SQL Server
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: myapp-db
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrong@Password
      - MSSQL_PID=Developer
    ports:
      - "1433:1433"
    volumes:
      - sqldata:/var/opt/mssql
    networks:
      - backend
    restart: unless-stopped

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    networks:
      - backend
    restart: unless-stopped
    command: redis-server --appendonly yes

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: myapp-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - api
    networks:
      - backend
    restart: unless-stopped

volumes:
  sqldata:
  redisdata:

networks:
  backend:
    driver: bridge
```

### Docker Compose Commands

```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# Build and start
docker-compose up --build

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# View logs for specific service
docker-compose logs api

# List running services
docker-compose ps

# Execute command in service
docker-compose exec api bash

# Restart services
docker-compose restart

# Scale service
docker-compose up -d --scale api=3
```

---

## .dockerignore File

```dockerignore
# Git
.git
.gitignore
.gitattributes

# Build artifacts
**/bin
**/obj
**/out

# VS
.vs
.vscode
*.user
*.suo

# Documentation
*.md
README.md
LICENSE

# Docker
docker-compose*
Dockerfile*
.dockerignore

# Tests
**/*Tests
**/TestResults

# CI/CD
.github
.gitlab-ci.yml
azure-pipelines.yml

# Environment
.env
.env.*

# Logs
**/logs
*.log

# OS
.DS_Store
Thumbs.db

# NuGet
**/packages
*.nupkg
```

---

## Docker Best Practices

### 1. Use Multi-Stage Builds

```dockerfile
# ✅ Good - Multi-stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "MyApp.dll"]

# ❌ Bad - Single stage with SDK in final image
FROM mcr.microsoft.com/dotnet/sdk:8.0
WORKDIR /app
COPY . .
RUN dotnet publish -c Release -o /out
ENTRYPOINT ["dotnet", "out/MyApp.dll"]
```

### 2. Leverage Build Cache

```dockerfile
# ✅ Good - Copy project files first for caching
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy project files first
COPY ["MyApp/MyApp.csproj", "MyApp/"]
RUN dotnet restore "MyApp/MyApp.csproj"

# Then copy everything else
COPY . .
RUN dotnet build

# ❌ Bad - Copy everything at once
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet restore
RUN dotnet build
```

### 3. Use Specific Image Tags

```dockerfile
# ✅ Good - Specific version
FROM mcr.microsoft.com/dotnet/aspnet:8.0.1

# ❌ Bad - Latest tag (unpredictable)
FROM mcr.microsoft.com/dotnet/aspnet:latest
```

### 4. Run as Non-Root User

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app

# Create non-root user
RUN addgroup --system --gid 1000 appuser && \
    adduser --system --uid 1000 --ingroup appuser appuser

COPY --from=publish /app/publish .

# Change ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### 5. Minimize Layers

```dockerfile
# ✅ Good - Combined commands
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# ❌ Bad - Separate layers
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*
```

---

## CI/CD Fundamentals

**CI/CD** = Continuous Integration / Continuous Deployment

### Continuous Integration (CI)

**Process:**
1. Developer commits code
2. CI server detects change
3. Code is built
4. Tests are run
5. Feedback is provided

**Benefits:**
- ✅ Early bug detection
- ✅ Automated testing
- ✅ Consistent builds
- ✅ Faster integration

### Continuous Deployment (CD)

**Process:**
1. Code passes CI
2. Automated deployment to staging
3. Run integration tests
4. Deploy to production (automatically or with approval)

**Benefits:**
- ✅ Faster releases
- ✅ Reduced manual errors
- ✅ Consistent deployments
- ✅ Quick rollbacks

---

## GitHub Actions

### Basic Workflow

```yaml
# .github/workflows/dotnet.yml
name: .NET CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 8.0.x
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build
      run: dotnet build --no-restore --configuration Release
    
    - name: Test
      run: dotnet test --no-build --verbosity normal --configuration Release
```

### Build, Test, and Docker

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ main ]
    tags:
      - 'v*'

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 8.0.x
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build
      run: dotnet build --no-restore --configuration Release
    
    - name: Run tests
      run: dotnet test --no-build --configuration Release --verbosity normal
    
    - name: Publish
      run: dotnet publish -c Release -o ./publish

  build-docker:
    needs: build-and-test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Log in to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
```

### Deploy to Azure

```yaml
name: Deploy to Azure

on:
  push:
    branches: [ main ]

env:
  AZURE_WEBAPP_NAME: myapp-prod
  DOTNET_VERSION: '8.0.x'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}
    
    - name: Build and publish
      run: |
        dotnet restore
        dotnet build --configuration Release
        dotnet publish -c Release -o ./publish
    
    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ./publish
```

### Matrix Strategy (Multiple Versions)

```yaml
name: Multi-Version Testing

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        dotnet-version: ['6.0.x', '7.0.x', '8.0.x']
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup .NET ${{ matrix.dotnet-version }}
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ matrix.dotnet-version }}
    
    - name: Restore
      run: dotnet restore
    
    - name: Build
      run: dotnet build --no-restore
    
    - name: Test
      run: dotnet test --no-build
```

---

## Azure DevOps Pipelines

### Basic Pipeline

```yaml
# azure-pipelines.yml
trigger:
  - main
  - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '8.0.x'

steps:
- task: UseDotNet@2
  displayName: 'Install .NET SDK'
  inputs:
    version: $(dotnetVersion)

- task: DotNetCoreCLI@2
  displayName: 'Restore packages'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Build'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration) --no-restore'

- task: DotNetCoreCLI@2
  displayName: 'Run tests'
  inputs:
    command: 'test'
    projects: '**/*Tests/*.csproj'
    arguments: '--configuration $(buildConfiguration) --no-build'

- task: DotNetCoreCLI@2
  displayName: 'Publish'
  inputs:
    command: 'publish'
    publishWebProjects: true
    arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'

- task: PublishBuildArtifacts@1
  displayName: 'Publish artifacts'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
```

### Docker Build and Push

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  dockerRegistryServiceConnection: 'MyDockerRegistry'
  imageRepository: 'myapp'
  containerRegistry: 'myregistry.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
  tag: '$(Build.BuildId)'

stages:
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: Build
    steps:
    - task: UseDotNet@2
      inputs:
        version: '8.0.x'
    
    - task: DotNetCoreCLI@2
      displayName: 'dotnet build'
      inputs:
        command: 'build'
        configuration: 'Release'
    
    - task: DotNetCoreCLI@2
      displayName: 'dotnet test'
      inputs:
        command: 'test'
        configuration: 'Release'

- stage: Docker
  displayName: 'Build and Push Docker Image'
  dependsOn: Build
  jobs:
  - job: Docker
    steps:
    - task: Docker@2
      displayName: 'Build Docker image'
      inputs:
        command: 'build'
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        tags: |
          $(tag)
          latest
    
    - task: Docker@2
      displayName: 'Push Docker image'
      inputs:
        command: 'push'
        repository: $(imageRepository)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)
          latest
```

### Multi-Stage Pipeline (Build, Test, Deploy)

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  displayName: 'Build Stage'
  jobs:
  - job: BuildJob
    displayName: 'Build and Test'
    steps:
    - task: UseDotNet@2
      inputs:
        version: '8.0.x'
    
    - task: DotNetCoreCLI@2
      displayName: 'Restore'
      inputs:
        command: 'restore'
    
    - task: DotNetCoreCLI@2
      displayName: 'Build'
      inputs:
        command: 'build'
        arguments: '--configuration $(buildConfiguration)'
    
    - task: DotNetCoreCLI@2
      displayName: 'Test'
      inputs:
        command: 'test'
        arguments: '--configuration $(buildConfiguration) --no-build'
    
    - task: DotNetCoreCLI@2
      displayName: 'Publish'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'

- stage: DeployToStaging
  displayName: 'Deploy to Staging'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployStaging
    displayName: 'Deploy to Staging Environment'
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'MyAzureSubscription'
              appType: 'webApp'
              appName: 'myapp-staging'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'

- stage: DeployToProduction
  displayName: 'Deploy to Production'
  dependsOn: DeployToStaging
  condition: succeeded()
  jobs:
  - deployment: DeployProduction
    displayName: 'Deploy to Production Environment'
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'MyAzureSubscription'
              appType: 'webApp'
              appName: 'myapp-production'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

---

## Deployment Strategies

### 1. Blue-Green Deployment

```
Production (Blue)  →  New Version (Green)
     ↓                      ↓
   Users  ←───────────  Switch Traffic
```

**Advantages:**
- ✅ Zero downtime
- ✅ Easy rollback
- ✅ Test in production-like environment

**Docker Compose Example:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app-blue
      - app-green

  app-blue:
    image: myapp:1.0
    environment:
      - VERSION=blue

  app-green:
    image: myapp:2.0
    environment:
      - VERSION=green
```

### 2. Rolling Deployment

```
Instance 1: v1 → v2
Instance 2: v1 → v2  (after Instance 1 healthy)
Instance 3: v1 → v2  (after Instance 2 healthy)
```

**Advantages:**
- ✅ No additional infrastructure needed
- ✅ Gradual rollout
- ✅ Can stop if issues detected

**Kubernetes Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

### 3. Canary Deployment

```
95% Users  →  Old Version
 5% Users  →  New Version (Canary)
```

**Advantages:**
- ✅ Test with real users
- ✅ Limited blast radius
- ✅ Gradual rollout

---

## Health Checks

### ASP.NET Core Health Checks

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        name: "database",
        timeout: TimeSpan.FromSeconds(3))
    .AddRedis(
        builder.Configuration.GetConnectionString("Redis"),
        name: "redis");

var app = builder.Build();

app.MapHealthChecks("/health");
app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
});

app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false
});
```

### Docker Health Check

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=publish /app/publish .

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80/health || exit 1

ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### Docker Compose with Health Check

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "5000:80"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

## Secrets Management

### GitHub Secrets

```yaml
# Access secrets in workflow
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to Azure
      env:
        CONNECTION_STRING: ${{ secrets.DB_CONNECTION_STRING }}
        API_KEY: ${{ secrets.API_KEY }}
      run: |
        echo "Deploying with connection string"
```

### Azure Key Vault

```yaml
# Azure DevOps Pipeline
steps:
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'MySubscription'
    KeyVaultName: 'mykeyvault'
    SecretsFilter: '*'
    RunAsPreJob: true

- task: DotNetCoreCLI@2
  inputs:
    command: 'run'
  env:
    ConnectionStrings__DefaultConnection: $(DB-CONNECTION-STRING)
```

### Docker Secrets

```bash
# Create secret
echo "mypassword" | docker secret create db_password -

# Use in docker-compose
version: '3.8'

services:
  db:
    image: postgres:15
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    external: true
```

---

## Monitoring and Logging

### Application Insights in Docker

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app

# Install Application Insights
COPY --from=publish /app/publish .

ENV APPLICATIONINSIGHTS_CONNECTION_STRING=""

ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### Logging with Serilog

```csharp
// Program.cs
using Serilog;

Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.File("/app/logs/app-.log", rollingInterval: RollingInterval.Day)
    .CreateLogger();

var builder = WebApplication.CreateBuilder(args);
builder.Host.UseSerilog();
```

### Docker Volume for Logs

```yaml
version: '3.8'

services:
  api:
    build: .
    volumes:
      - ./logs:/app/logs
    environment:
      - Logging__LogLevel__Default=Information
```

---

## Best Practices Summary

### Docker

1. ✅ Use multi-stage builds
2. ✅ Leverage build cache
3. ✅ Use specific image tags
4. ✅ Run as non-root user
5. ✅ Keep images small
6. ✅ Use .dockerignore
7. ✅ Add health checks
8. ✅ Set resource limits

### CI/CD

1. ✅ Run tests before deploy
2. ✅ Use environment-specific configs
3. ✅ Implement health checks
4. ✅ Use secrets management
5. ✅ Enable rollback capability
6. ✅ Monitor deployments
7. ✅ Use staging environment
8. ✅ Automate everything

---

## Quick Reference: Docker Commands

```bash
# Images
docker images                    # List images
docker pull image:tag           # Pull image
docker build -t name:tag .      # Build image
docker rmi image:tag            # Remove image

# Containers
docker ps                       # List running containers
docker ps -a                    # List all containers
docker run image:tag            # Run container
docker run -d image:tag         # Run detached
docker run -p 8080:80 image     # Run with port mapping
docker stop container-id        # Stop container
docker rm container-id          # Remove container
docker logs container-id        # View logs
docker exec -it container bash  # Execute command

# Docker Compose
docker-compose up              # Start services
docker-compose up -d           # Start detached
docker-compose down            # Stop services
docker-compose logs            # View logs
docker-compose ps              # List services
docker-compose exec service bash  # Execute command

# Cleanup
docker system prune            # Remove unused data
docker container prune         # Remove stopped containers
docker image prune             # Remove unused images
docker volume prune            # Remove unused volumes
```

---

**Guide Complete!** This comprehensive Docker & CI/CD guide covers containerization, multi-stage builds, Docker Compose, GitHub Actions, Azure DevOps Pipelines, deployment strategies, and best practices for deploying .NET applications! 🐳
