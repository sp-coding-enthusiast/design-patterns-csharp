# 53. Builder Pattern in ASP.NET Core

## Introduction

The **Builder Pattern** is used extensively throughout **ASP.NET Core** to configure complex objects in a readable, step-by-step manner.

Instead of creating large constructors with many parameters, ASP.NET Core exposes fluent builder APIs.

Example:

```csharp
builder.Services
       .AddControllers()
       .AddJsonOptions(options =>
       {
           options.JsonSerializerOptions
                  .WriteIndented = true;
       });
```

Each method configures part of the application and returns the builder, allowing method chaining.

---

# Why Builder Pattern?

Without Builder:

```csharp
var app = new WebApplication(
    services,
    configuration,
    logging,
    middleware,
    endpoints,
    authentication,
    authorization);
```

This would quickly become difficult to read and maintain.

With Builder:

```csharp
builder.Services.AddControllers();

builder.Services.AddAuthentication();

builder.Services.AddAuthorization();
```

Each step configures one aspect of the application.

---

# Builder Flow

```
Application Starts

↓

WebApplicationBuilder

↓

Configure Services

↓

Configure Logging

↓

Configure Middleware

↓

Build()

↓

WebApplication
```

---

# WebApplicationBuilder

When you create an ASP.NET Core application:

```csharp
var builder =
    WebApplication.CreateBuilder(args);
```

Internally:

```
Create Builder

↓

Configuration

↓

Logging

↓

Dependency Injection

↓

Hosting

↓

Environment
```

Everything is collected into a builder object.

---

# Configure Services

```csharp
builder.Services
       .AddControllers();

builder.Services
       .AddEndpointsApiExplorer();

builder.Services
       .AddSwaggerGen();
```

Each method:

- Configures services
- Returns the same builder
- Allows fluent chaining

---

# Build the Application

```csharp
var app = builder.Build();
```

Internally:

```
Builder

↓

Validate Configuration

↓

Create DI Container

↓

Build Middleware Pipeline

↓

Create WebApplication
```

---

# Configure Middleware

```csharp
app.UseHttpsRedirection();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

The middleware pipeline itself follows a builder-like approach.

---

# Real-World Analogy

Imagine building a custom laptop.

```
Choose CPU

↓

Choose RAM

↓

Choose Storage

↓

Choose GPU

↓

Build Laptop
```

Each step modifies the same product.

This is exactly how ASP.NET Core builds an application.

---

# Advantages

- Readable configuration
- Fluent API
- Easy to extend
- Supports optional settings
- Reduces constructor complexity

---

# 54. Configuration Builder

## Introduction

ASP.NET Core uses **ConfigurationBuilder** to construct application configuration from multiple sources.

Instead of reading a single configuration file, it combines several providers into one configuration object.

---

# Configuration Sources

A typical application may read configuration from:

```
appsettings.json

↓

appsettings.Development.json

↓

Environment Variables

↓

User Secrets

↓

Azure Key Vault

↓

Command Line

↓

Configuration Object
```

---

# Basic Example

```csharp
var builder =
    new ConfigurationBuilder()
        .AddJsonFile(
            "appsettings.json")
        .AddEnvironmentVariables()
        .AddCommandLine(args);

IConfiguration configuration =
    builder.Build();
```

---

# Internal Workflow

```
ConfigurationBuilder

↓

Add Provider

↓

Add Provider

↓

Add Provider

↓

Merge Values

↓

Build()

↓

IConfiguration
```

---

# Reading Configuration

Suppose:

```json
{
  "Database": {
    "Connection":
      "Server=SQL01"
  }
}
```

Read it:

```csharp
string connection =
    configuration[
        "Database:Connection"];
```

---

# Configuration Precedence

Later providers override earlier ones.

Example:

```
appsettings.json

↓

Environment Variables

↓

Command Line
```

If the same key exists in all three:

```
Command Line Wins
```

This enables environment-specific overrides.

---

# Configuration Builder in ASP.NET Core

Normally you do not create it manually.

Instead:

```csharp
var builder =
    WebApplication.CreateBuilder(args);
```

The framework automatically configures:

- JSON files
- Environment variables
- Command-line arguments
- User secrets (Development)
- Host configuration

---

# Adding Custom Configuration

```csharp
builder.Configuration
       .AddJsonFile(
           "custom.json",
           optional: true);
```

or

```csharp
builder.Configuration
       .AddEnvironmentVariables(
           prefix: "APP_");
```

---

# Real-World Analogy

Imagine collecting information from different departments.

```
HR

↓

Finance

↓

IT

↓

Management

↓

Final Company Report
```

ConfigurationBuilder merges multiple sources into one consistent view.

---

# Advantages

- Multiple configuration sources
- Environment-specific configuration
- Extensible
- Secure secret management
- Strong integration with DI

---

# 55. Builder Pattern in Entity Framework Core

## Introduction

Entity Framework Core uses the Builder Pattern extensively to configure:

- Database providers
- Entity mappings
- Relationships
- Constraints
- Indexes
- Model metadata

Instead of huge constructors, EF Core exposes fluent builders.

---

# DbContextOptionsBuilder

Example:

```csharp
builder.Services.AddDbContext<
    AppDbContext>(options =>
{
    options.UseSqlServer(
        connectionString);
});
```

Internally:

```
DbContextOptionsBuilder

↓

Provider

↓

Connection String

↓

Retry Policy

↓

Logging

↓

Build Options
```

---

# EntityTypeBuilder

Inside `OnModelCreating()`:

```csharp
protected override void
    OnModelCreating(
        ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Customer>()
        .HasKey(c => c.Id);
}
```

`EntityTypeBuilder` is a builder.

---

# Configure Properties

```csharp
modelBuilder.Entity<Customer>()
    .Property(c => c.Name)
    .HasMaxLength(100)
    .IsRequired();
```

Workflow:

```
Entity

↓

Property

↓

MaxLength

↓

Required
```

Each method returns the same builder.

---

# Configure Relationships

```csharp
modelBuilder.Entity<Order>()
    .HasOne(o => o.Customer)
    .WithMany(c => c.Orders)
    .HasForeignKey(o => o.CustomerId);
```

Builder flow:

```
Order

↓

Customer

↓

Many Orders

↓

Foreign Key
```

---

# Configure Indexes

```csharp
modelBuilder.Entity<Customer>()
    .HasIndex(c => c.Email)
    .IsUnique();
```

---

# Configure Table Name

```csharp
modelBuilder.Entity<Customer>()
    .ToTable("Customers");
```

---

# Configure Default Values

```csharp
modelBuilder.Entity<Customer>()
    .Property(c => c.CreatedDate)
    .HasDefaultValueSql(
        "GETDATE()");
```

---

# Internal Workflow

```
DbContext

↓

ModelBuilder

↓

EntityTypeBuilder

↓

PropertyBuilder

↓

RelationshipBuilder

↓

Database Model
```

During application startup:

```
Build Model

↓

Cache Model

↓

Reuse Model

↓

Generate SQL
```

The model is built once and reused for better performance.

---

# Fluent API

Every configuration method returns the builder itself.

Example:

```csharp
modelBuilder.Entity<Customer>()
    .Property(c => c.Name)
    .HasMaxLength(100)
    .HasColumnName("FullName")
    .IsRequired();
```

This is both a **Builder Pattern** and a **Fluent Interface**.

---

# Real-World Analogy

Imagine designing a house.

```
Choose Foundation

↓

Choose Walls

↓

Choose Doors

↓

Choose Windows

↓

Build House
```

EF Core builds the database model in the same incremental way.

---

# Builder Pattern Usage Across ASP.NET Core

| Component | Builder Class | Purpose |
|-----------|---------------|---------|
| Application Startup | `WebApplicationBuilder` | Configure host and services |
| Configuration | `ConfigurationBuilder` | Build configuration from multiple sources |
| Dependency Injection | `IServiceCollection` | Register services |
| Logging | `ILoggingBuilder` | Configure logging providers |
| Authentication | `AuthenticationBuilder` | Configure authentication schemes |
| Authorization | `AuthorizationBuilder` | Configure authorization policies |
| EF Core | `DbContextOptionsBuilder` | Configure DbContext |
| EF Core | `ModelBuilder` | Configure entities |
| EF Core | `EntityTypeBuilder<T>` | Configure individual entity mappings |

---

# Builder Pattern vs Fluent Interface

| Builder Pattern | Fluent Interface |
|----------------|------------------|
| Focuses on constructing complex objects | Focuses on readable chained APIs |
| Produces a final object with `Build()` or equivalent | May not produce a separate object |
| Guides configuration step by step | Improves API readability |
| Often uses Fluent Interface internally | Can exist without being a Builder |

**Relationship:** Most ASP.NET Core builders implement a Fluent Interface, but not every fluent API is a Builder.

---

# Best Practices

- Use framework-provided builders instead of manual configuration where possible.
- Keep configuration organized by concern (services, middleware, logging, authentication, etc.).
- Prefer strongly typed configuration using the Options pattern for application settings.
- Keep EF Core entity configuration in separate configuration classes for large applications using `IEntityTypeConfiguration<T>`.
- Avoid placing excessive startup logic inside `Program.cs`; move reusable configuration into extension methods.

---

# Interview Questions

### Basic

1. Why does ASP.NET Core use the Builder Pattern?
2. What is `WebApplicationBuilder`?
3. What is `ConfigurationBuilder`?

### Intermediate

4. How does `builder.Build()` work internally?
5. Why does `ConfigurationBuilder` support multiple providers?
6. What is `DbContextOptionsBuilder`?

### Advanced

7. How does EF Core use `EntityTypeBuilder` to build the database model?
8. Why is the Builder Pattern preferred over constructors with many parameters?
9. How does the Builder Pattern improve extensibility in ASP.NET Core?
10. What is the relationship between the Builder Pattern and Fluent Interface in ASP.NET Core?

---

# Summary

The **Builder Pattern** is deeply integrated into the .NET ecosystem. In **ASP.NET Core**, `WebApplicationBuilder` incrementally constructs the application's hosting environment, services, configuration, and middleware before producing a `WebApplication`. `ConfigurationBuilder` combines multiple configuration providers into a unified `IConfiguration` object while supporting provider precedence and environment-specific settings. In **Entity Framework Core**, builders such as `DbContextOptionsBuilder`, `ModelBuilder`, and `EntityTypeBuilder<T>` enable developers to configure database providers, entity mappings, relationships, indexes, and constraints through a fluent, extensible API. Together, these builders make application configuration more readable, maintainable, and scalable.