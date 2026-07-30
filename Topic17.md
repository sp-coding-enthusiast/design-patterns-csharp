# 43. Dependency Injection (DI) Pattern

## Introduction

The **Dependency Injection (DI) Pattern** is one of the most widely used design patterns in modern software development.

Although it is often grouped with **Creational Patterns**, it is more accurately an **Architectural / Behavioral Pattern** because its primary purpose is **managing object dependencies**, not simply creating objects.

Dependency Injection allows an object to receive its dependencies from an external source instead of creating them itself.

---

# Definition

> **Dependency Injection is a design pattern in which an object's dependencies are provided (injected) by an external component rather than being created by the object itself.**

---

# What is a Dependency?

A dependency is any object that another object needs to perform its work.

Example:

```text
OrderService

↓

ILogger
```

Here, `OrderService` depends on `ILogger`.

---

# Without Dependency Injection

Consider an order service.

```csharp
public class OrderService
{
    private readonly EmailService _emailService;

    public OrderService()
    {
        _emailService = new EmailService();
    }

    public void PlaceOrder()
    {
        _emailService.Send();
    }
}
```

### Problems

- Tight coupling
- Difficult to test
- Cannot replace `EmailService`
- Violates the Dependency Inversion Principle (DIP)

---

# With Dependency Injection

```csharp
public class OrderService
{
    private readonly INotificationService _notificationService;

    public OrderService(
        INotificationService notificationService)
    {
        _notificationService = notificationService;
    }

    public void PlaceOrder()
    {
        _notificationService.Send();
    }
}
```

Now:

```csharp
OrderService

↓

INotificationService

↓

EmailService

OR

SmsService

OR

WhatsAppService
```

The service depends on an abstraction, not a concrete class.

---

# Why Use Dependency Injection?

Without DI:

```
OrderService

↓

new EmailService()
```

Changing the notification method requires modifying `OrderService`.

---

With DI:

```
OrderService

↓

INotificationService

↓

Email

SMS

Push Notification

WhatsApp
```

New implementations can be added without changing the consumer.

---

# Real-World Analogy

Imagine a restaurant.

Without DI:

```
Chef

↓

Owns Stove

Owns Fridge

Owns Oven

Owns Ingredients
```

The chef is responsible for creating and maintaining everything.

---

With DI:

```
Restaurant

↓

Provides Stove

Provides Ingredients

Provides Oven

↓

Chef Cooks
```

The chef focuses only on cooking.

Similarly, classes should focus on business logic, not object creation.

---

# Types of Dependency Injection

There are three common types.

---

## 1. Constructor Injection (Recommended)

Dependencies are provided through the constructor.

```csharp
public class OrderService
{
    private readonly ILogger _logger;

    public OrderService(ILogger logger)
    {
        _logger = logger;
    }
}
```

### Advantages

- Dependencies are mandatory.
- Easy to test.
- Immutable after creation.
- Recommended by ASP.NET Core.

---

## 2. Property Injection

Dependencies are assigned through properties.

```csharp
public class OrderService
{
    public ILogger Logger { get; set; }
}
```

Usage:

```csharp
var service = new OrderService();

service.Logger = new ConsoleLogger();
```

### Disadvantages

- Dependency may be forgotten.
- Object can be left in an invalid state.

---

## 3. Method Injection

Dependency is supplied only when needed.

```csharp
public class OrderService
{
    public void PlaceOrder(ILogger logger)
    {
        logger.Log("Placed");
    }
}
```

Useful when the dependency is required only for a specific operation.

---

# How DI Works Internally

```
Application Starts

↓

DI Container

↓

Registers Services

↓

Controller Requested

↓

Container Creates Controller

↓

Injects Dependencies

↓

Returns Fully Constructed Object
```

The class never creates its own dependencies.

---

# Dependency Injection in ASP.NET Core

## Step 1: Interface

```csharp
public interface INotificationService
{
    void Send();
}
```

---

## Step 2: Implementation

```csharp
public class EmailService : INotificationService
{
    public void Send()
    {
        Console.WriteLine("Email Sent");
    }
}
```

---

## Step 3: Register with DI Container

```csharp
builder.Services.AddScoped<
    INotificationService,
    EmailService>();
```

---

## Step 4: Consume the Service

```csharp
public class OrderService
{
    private readonly INotificationService
        _notificationService;

    public OrderService(
        INotificationService notificationService)
    {
        _notificationService =
            notificationService;
    }

    public void PlaceOrder()
    {
        _notificationService.Send();
    }
}
```

The framework injects the dependency automatically.

---

# Service Lifetimes

ASP.NET Core provides three lifetimes.

## Singleton

```csharp
builder.Services.AddSingleton<
    ILogger,
    FileLogger>();
```

One instance for the entire application.

```
Application

↓

One Logger
```

---

## Scoped

```csharp
builder.Services.AddScoped<
    IOrderService,
    OrderService>();
```

One instance per HTTP request.

```
Request 1

↓

OrderService #1

Request 2

↓

OrderService #2
```

---

## Transient

```csharp
builder.Services.AddTransient<
    IParser,
    JsonParser>();
```

A new instance is created every time it is requested.

---

# Constructor Injection Flow

```
Controller

↓

OrderService

↓

Repository

↓

DbContext
```

The DI container resolves each dependency recursively until the entire object graph is created.

---

# Advantages

- Loose coupling
- Easier unit testing
- Better maintainability
- Supports SOLID principles
- Easier replacement of implementations
- Centralized object creation
- Encourages modular architecture

---

# Disadvantages

- More initial setup
- Too many dependencies can indicate poor class design
- Misconfigured registrations cause runtime errors
- Can make application startup more complex

---

# Common Mistakes

## 1. Using `new` Instead of DI

```csharp
var logger = new Logger();
```

Prefer:

```csharp
public OrderService(ILogger logger)
{
}
```

---

## 2. Injecting Too Many Dependencies

```csharp
public OrderService(
    ILogger logger,
    IRepository repository,
    ICache cache,
    IEmail email,
    ISms sms,
    IAudit audit,
    IConfiguration config,
    IMapper mapper)
{
}
```

This often violates the **Single Responsibility Principle**.

Consider splitting the class into smaller services.

---

## 3. Incorrect Lifetime Selection

For example:

- Injecting a **Scoped** service into a **Singleton** can cause lifetime issues.
- Use lifetimes that match the intended scope of the service.

---

## 4. Service Locator Anti-Pattern

Avoid manually resolving services everywhere.

```csharp
var service =
    serviceProvider.GetService<IOrderService>();
```

Prefer constructor injection.

---

# Dependency Injection vs Dependency Inversion

| Dependency Injection | Dependency Inversion Principle |
|-----------------------|--------------------------------|
| Design pattern | SOLID principle |
| Supplies dependencies | Encourages depending on abstractions |
| Implementation technique | Design guideline |
| Achieved using DI containers | Achieved through interfaces and abstractions |

**Relationship:** DI is one of the most common ways to implement the Dependency Inversion Principle.

---

# DI Container Responsibilities

A DI container performs four main tasks:

1. Registers services.
2. Creates objects.
3. Resolves dependencies recursively.
4. Manages object lifetimes.

Example:

```text
Controller

↓

OrderService

↓

Repository

↓

DbContext

↓

SQL Connection
```

The container builds this dependency graph automatically.

---

# Real-World Examples

- ASP.NET Core Controllers
- Entity Framework Core `DbContext`
- Logging (`ILogger<T>`)
- `IHttpClientFactory`
- Options Pattern (`IOptions<T>`)
- Background Services
- Authentication and Authorization services

---

# Interview Questions

### Basic

1. What is Dependency Injection?
2. What problem does DI solve?
3. What is a dependency?

### Intermediate

4. What are the three types of Dependency Injection?
5. What are the service lifetimes in ASP.NET Core?
6. Why is constructor injection preferred?

### Advanced

7. Explain how the ASP.NET Core DI container resolves dependencies.
8. What happens if a Scoped service is injected into a Singleton?
9. What is the Service Locator anti-pattern?
10. How does Dependency Injection help implement the Dependency Inversion Principle?

---

# Summary

The **Dependency Injection Pattern** promotes **loose coupling** by supplying dependencies from an external source instead of allowing classes to create them directly. This makes applications easier to test, extend, and maintain while supporting the **Dependency Inversion Principle**. ASP.NET Core has a built-in DI container that automatically creates object graphs, injects dependencies, and manages service lifetimes through **Singleton**, **Scoped**, and **Transient** registrations. Constructor injection is the preferred approach because it makes dependencies explicit, ensures required services are available, and produces more robust, testable code.