# 44. Service Locator Pattern

## Introduction

The **Service Locator Pattern** is a design pattern that provides a **centralized registry** from which classes can request the services they need.

Instead of dependencies being **injected** into a class, the class **asks** a service locator to provide them.

Although Service Locator was popular before modern Dependency Injection (DI) containers, it is **generally discouraged in modern software development**, especially in ASP.NET Core.

---

# Definition

> **The Service Locator Pattern provides a central object that knows how to locate and return requested services.**

---

# Why Was Service Locator Introduced?

Before Dependency Injection frameworks became common, developers needed a way to avoid writing code like:

```csharp
var logger = new FileLogger();

var repository = new SqlRepository();
```

A Service Locator centralized object creation.

---

# Without Service Locator

```csharp
public class OrderService
{
    private readonly ILogger _logger =
        new FileLogger();

    public void PlaceOrder()
    {
        _logger.Log("Order Placed");
    }
}
```

Problems:

- Tight coupling
- Difficult to replace implementations
- Difficult to unit test

---

# With Service Locator

```csharp
public class OrderService
{
    public void PlaceOrder()
    {
        ILogger logger =
            ServiceLocator.Get<ILogger>();

        logger.Log("Order Placed");
    }
}
```

The service is resolved at runtime.

---

# Structure

```
                Client

                   │

                   ▼

          Service Locator

          ▲            ▲

          │            │

      ILogger     IRepository

          │            │

    FileLogger   SqlRepository
```

The client asks the locator for services whenever they are needed.

---

# Simple Implementation

## Interface

```csharp
public interface ILogger
{
    void Log(string message);
}
```

---

## Implementation

```csharp
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

---

## Service Locator

```csharp
public static class ServiceLocator
{
    private static readonly Dictionary<Type, object>
        _services = new();

    public static void Register<T>(T service)
    {
        _services[typeof(T)] = service!;
    }

    public static T Get<T>()
    {
        return (T)_services[typeof(T)];
    }
}
```

---

## Registration

```csharp
ServiceLocator.Register<ILogger>(
    new ConsoleLogger());
```

---

## Usage

```csharp
ILogger logger =
    ServiceLocator.Get<ILogger>();

logger.Log("Hello");
```

---

# Internal Workflow

```
Application Starts

↓

Register Services

↓

Service Locator

↓

Client Requests ILogger

↓

Returns ConsoleLogger
```

---

# Real-World Analogy

Imagine a hotel reception desk.

Instead of every guest knowing where to find:

- Restaurant
- Laundry
- Taxi
- Spa

They simply ask the reception.

```
Guest

↓

Reception

↓

Requested Service
```

The reception acts as the Service Locator.

---

# Service Locator in ASP.NET Core

Although ASP.NET Core uses a Dependency Injection container internally, you can still misuse it as a Service Locator.

Example:

```csharp
public class OrderService
{
    private readonly IServiceProvider
        _serviceProvider;

    public OrderService(
        IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public void Process()
    {
        var logger =
            _serviceProvider
                .GetRequiredService<ILogger>();

        logger.LogInformation("Done");
    }
}
```

This is effectively using the DI container as a Service Locator.

While technically possible, it is usually discouraged.

---

# Dependency Injection vs Service Locator

## Dependency Injection

```
Container

↓

Injects ILogger

↓

OrderService
```

The class receives its dependencies.

---

## Service Locator

```
OrderService

↓

Service Locator

↓

ILogger
```

The class must locate its own dependencies.

---

# Advantages

Historically, Service Locator offered several benefits:

- Centralized service lookup
- Reduced direct object creation
- Runtime flexibility
- Fewer constructor parameters

However, most of these advantages are now better addressed by Dependency Injection.

---

# Disadvantages

- Hidden dependencies
- Harder testing
- Runtime failures instead of compile-time errors
- Violates the Dependency Inversion Principle when overused
- Encourages global state

---

# 45. Why Service Locator is Considered an Anti-Pattern

Today, most experienced developers consider Service Locator an **anti-pattern**.

An anti-pattern is **a common solution that appears useful but usually leads to poor software design**.

---

# Problem 1: Hidden Dependencies

Consider this class:

```csharp
public class OrderService
{
    public void PlaceOrder()
    {
        var logger =
            ServiceLocator.Get<ILogger>();

        logger.Log("Placed");
    }
}
```

Looking only at the constructor:

```csharp
public OrderService()
{
}
```

You cannot tell that the class depends on `ILogger`.

The dependency is hidden inside the method.

---

### Dependency Injection Version

```csharp
public class OrderService
{
    private readonly ILogger _logger;

    public OrderService(
        ILogger logger)
    {
        _logger = logger;
    }
}
```

Now the dependency is explicit.

---

# Problem 2: Violates Explicit Dependencies Principle

Good classes clearly declare what they need.

Service Locator hides those requirements.

```
OrderService

↓

???

↓

ServiceLocator
```

A developer must inspect the implementation to discover dependencies.

---

# Problem 3: Runtime Errors

Suppose registration is forgotten.

```csharp
ILogger logger =
    ServiceLocator.Get<ILogger>();
```

Result:

```
KeyNotFoundException
```

The application fails at runtime.

With Dependency Injection, missing registrations are often detected during application startup or object construction.

---

# Problem 4: Difficult Unit Testing

Testing:

```csharp
ServiceLocator.Register<ILogger>(
    new FakeLogger());
```

Every test must manipulate global state.

Tests may interfere with each other.

---

Dependency Injection:

```csharp
var service =
    new OrderService(
        new FakeLogger());
```

No global registration is required.

Each test is isolated.

---

# Problem 5: Global State

Service Locator is often implemented as a static class.

```
Application

↓

Static Locator

↓

Everything Uses It
```

Global state makes applications harder to understand and maintain.

---

# Problem 6: Violates the Dependency Inversion Principle

Instead of depending only on abstractions:

```text
OrderService

↓

ILogger
```

The class also depends on:

```text
ServiceLocator
```

This introduces an unnecessary dependency on a global infrastructure component.

---

# Problem 7: Poor Readability

Constructor:

```csharp
public OrderService()
{
}
```

Question:

> What services does this class need?

Answer:

You cannot know without reading the entire implementation.

---

Constructor Injection:

```csharp
public OrderService(
    ILogger logger,
    IRepository repository,
    ICache cache)
{
}
```

The dependencies are immediately visible.

---

# Visual Comparison

## Dependency Injection

```
Container

↓

Injects Dependencies

↓

OrderService

↓

Business Logic
```

---

## Service Locator

```
OrderService

↓

Ask Locator

↓

Find Service

↓

Business Logic
```

The class takes on the additional responsibility of locating services.

---

# When Can Service Locator Be Acceptable?

Although generally discouraged, there are limited scenarios where it may be reasonable:

- Legacy applications that predate DI frameworks
- Plugin systems with dynamic service discovery
- Framework internals
- Infrastructure code where constructor injection is impractical

Even in these cases, its use should be carefully justified and isolated.

---

# Best Practice in ASP.NET Core

Prefer constructor injection.

Good:

```csharp
public class OrderService
{
    public OrderService(
        ILogger<OrderService> logger)
    {
    }
}
```

Avoid:

```csharp
_serviceProvider
    .GetRequiredService<ILogger>();
```

unless there is a specific architectural reason (for example, resolving a service whose type is determined dynamically at runtime).

---

# Service Locator vs Dependency Injection

| Feature | Service Locator | Dependency Injection |
|----------|-----------------|----------------------|
| Dependencies | Hidden | Explicit |
| Testability | Difficult | Excellent |
| Compile-time visibility | Poor | Excellent |
| Runtime failures | More likely | Less likely |
| Coupling | Higher | Lower |
| Uses global state | Often | No |
| SOLID friendly | Generally No | Yes |
| Recommended in ASP.NET Core | Rarely | Yes |

---

# Interview Questions

### Basic

1. What is the Service Locator Pattern?
2. How does it differ from Dependency Injection?
3. Why was Service Locator popular before DI containers?

### Intermediate

4. Why are hidden dependencies a problem?
5. Why does Service Locator make testing harder?
6. What is the Explicit Dependencies Principle?

### Advanced

7. Why is Service Locator considered an anti-pattern?
8. Can `IServiceProvider` become a Service Locator in ASP.NET Core?
9. In what rare situations might a Service Locator be acceptable?
10. How does constructor injection improve maintainability compared to Service Locator?

---

# Summary

The **Service Locator Pattern** centralizes service resolution by allowing classes to request their own dependencies from a shared registry. While this reduces direct object creation, it also **hides dependencies**, introduces **global state**, increases **runtime failure risk**, and makes **unit testing more difficult**. For these reasons, it is widely regarded as an **anti-pattern** in modern application development. In ASP.NET Core, **constructor injection** is the preferred approach because it makes dependencies explicit, improves testability, supports SOLID principles, and allows the Dependency Injection container to manage object creation and lifetimes effectively.