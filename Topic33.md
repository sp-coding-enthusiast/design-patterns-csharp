# 80. Decorator Pattern in .NET

## Introduction

The **Decorator Pattern** is one of the most widely used patterns in the **.NET Framework** and **ASP.NET Core**.

Instead of modifying an existing class, .NET often **wraps** an object with another object that implements the same interface and adds additional behavior.

This approach follows the **Open/Closed Principle (OCP)**:

- Existing classes remain unchanged.
- New functionality is added through composition.

> **In .NET, decorators are commonly used for logging, caching, validation, authorization, compression, encryption, retry policies, and stream processing.**

---

# How Decorator Works in .NET

```
Application

↓

Decorator

↓

Original Service

↓

Business Logic
```

The decorator executes code before and/or after delegating to the wrapped service.

---

# Common Places Where Decorator Appears

- `Stream` classes
- Repository pattern
- MediatR pipeline behaviors
- Caching repositories
- Logging services
- Validation services
- Retry policies (e.g., Polly)
- Metrics and monitoring
- Authorization wrappers

---

# Example Structure

```
IService

▲

├── ProductService

└── LoggingDecorator

        │

        ▼

ProductService
```

The client depends only on `IService`.

---

# 81. Stream Decorators

One of the **best real-world examples** of the Decorator Pattern in .NET is the `Stream` hierarchy.

---

## Stream Class Hierarchy

```
Stream

├── FileStream

├── MemoryStream

├── NetworkStream

└── BufferedStream
```

Additional decorators include:

```
Stream

↓

FileStream

↓

BufferedStream

↓

GZipStream

↓

CryptoStream
```

Each wrapper adds new behavior while preserving the `Stream` interface.

---

# Example

```csharp
using var file =
    new FileStream("data.txt",
        FileMode.Open);

using var buffered =
    new BufferedStream(file);

using var compressed =
    new GZipStream(
        buffered,
        CompressionMode.Decompress);
```

Workflow:

```
Application

↓

GZipStream

↓

BufferedStream

↓

FileStream

↓

Disk
```

Each decorator contributes one responsibility:

- `FileStream` → Reads the file
- `BufferedStream` → Improves I/O performance
- `GZipStream` → Compresses/Decompresses data
- `CryptoStream` → Encrypts/Decrypts data

---

# Why Streams Use Decorator

Without decorators, you would need classes like:

```
CompressedBufferedFileStream

EncryptedBufferedFileStream

CompressedEncryptedFileStream
```

This would lead to **class explosion**.

Decorators allow behaviors to be combined dynamically.

---

# 82. Logging Decorator

## Problem

You want logging for every service call without changing the service itself.

---

### Service

```csharp
public interface IOrderService
{
    void PlaceOrder();
}
```

---

### Real Service

```csharp
public class OrderService
    : IOrderService
{
    public void PlaceOrder()
    {
        Console.WriteLine(
            "Order Created");
    }
}
```

---

### Logging Decorator

```csharp
public class LoggingOrderService
    : IOrderService
{
    private readonly IOrderService
        _service;

    public LoggingOrderService(
        IOrderService service)
    {
        _service = service;
    }

    public void PlaceOrder()
    {
        Console.WriteLine(
            "Logging Started");

        _service.PlaceOrder();

        Console.WriteLine(
            "Logging Finished");
    }
}
```

---

### Client

```csharp
IOrderService service =
    new LoggingOrderService(
        new OrderService());

service.PlaceOrder();
```

Output

```
Logging Started

Order Created

Logging Finished
```

---

# ASP.NET Core Example

```
Controller

↓

Logging Decorator

↓

Order Service

↓

Database
```

The service remains unchanged while logging is added externally.

---

# 83. Validation Decorator

## Problem

Validation code is repeated across services.

---

### Decorator

```csharp
public class ValidationOrderService
    : IOrderService
{
    private readonly IOrderService
        _service;

    public ValidationOrderService(
        IOrderService service)
    {
        _service = service;
    }

    public void PlaceOrder()
    {
        Console.WriteLine(
            "Validating Order");

        _service.PlaceOrder();
    }
}
```

---

### Workflow

```
Client

↓

Validation

↓

Order Service
```

---

# ASP.NET Core Example

```
API

↓

Validation Decorator

↓

Business Service

↓

Repository
```

Although ASP.NET Core often uses model validation, filters, or libraries like FluentValidation, a validation decorator is useful for validating business commands before executing the service.

---

# 84. Caching Decorator

## Problem

The database is queried repeatedly for the same data.

---

### Repository

```csharp
public interface IProductRepository
{
    Product Get(int id);
}
```

---

### Database Repository

```csharp
public class ProductRepository
    : IProductRepository
{
    public Product Get(int id)
    {
        Console.WriteLine(
            "Database");

        return new Product();
    }
}
```

---

### Caching Decorator

```csharp
public class CachedRepository
    : IProductRepository
{
    private readonly IProductRepository
        _repository;

    private readonly Dictionary<int,
        Product> _cache = new();

    public CachedRepository(
        IProductRepository repository)
    {
        _repository = repository;
    }

    public Product Get(int id)
    {
        if (_cache.ContainsKey(id))
        {
            Console.WriteLine(
                "Cache");

            return _cache[id];
        }

        var product =
            _repository.Get(id);

        _cache[id] = product;

        return product;
    }
}
```

---

### Client

```csharp
var repository =
    new CachedRepository(
        new ProductRepository());

repository.Get(1);

repository.Get(1);
```

Output

```
Database

Cache
```

The database is accessed only once.

---

# ASP.NET Core Version

A production implementation typically uses `IMemoryCache`.

```csharp
public class CachedProductRepository
    : IProductRepository
{
    private readonly IProductRepository
        _repository;

    private readonly IMemoryCache
        _cache;

    public CachedProductRepository(
        IProductRepository repository,
        IMemoryCache cache)
    {
        _repository = repository;
        _cache = cache;
    }

    public Product Get(int id)
    {
        return _cache.GetOrCreate(id, entry =>
        {
            entry.AbsoluteExpirationRelativeToNow =
                TimeSpan.FromMinutes(5);

            return _repository.Get(id);
        })!;
    }
}
```

---

# Chaining Multiple Decorators

Decorators can be composed.

```text
Client

↓

Logging Decorator

↓

Validation Decorator

↓

Caching Decorator

↓

Repository
```

Execution order:

```
Logging Start

↓

Validation

↓

Cache

↓

Repository

↓

Logging End
```

---

# Decorator Registration with DI

Manual registration:

```csharp
builder.Services.AddScoped<
    IOrderService,
    OrderService>();

// Decorator registration typically
// requires a decorator library such
// as Scrutor or manual composition.
```

In production, libraries such as **Scrutor** make decorator registration much easier.

---

# Real-World Uses

- Logging
- Validation
- Caching
- Retry policies
- Metrics
- Authorization
- Compression
- Encryption
- Performance monitoring

---

# Decorator vs Middleware

| Decorator | Middleware |
|-----------|------------|
| Wraps objects | Wraps HTTP request pipeline |
| Adds behavior to services | Adds behavior to requests/responses |
| Used throughout application layers | Used only in the web pipeline |
| Same interface as wrapped object | Uses `RequestDelegate` |

---

# Decorator vs Proxy

| Decorator | Proxy |
|-----------|-------|
| Adds behavior | Controls access |
| Extends functionality | Restricts, delays, or manages access |
| Focuses on enhancement | Focuses on indirection |

---

# Best Practices

- Give each decorator a single responsibility.
- Keep decorators stateless whenever possible.
- Chain decorators in a well-defined order.
- Use dependency injection to compose decorators.
- Prefer decorators over modifying existing business services for cross-cutting concerns.

---

# Interview Questions

### Basic

1. What is the Decorator Pattern?
2. Why is `Stream` a classic Decorator example?
3. What are common uses of decorators in .NET?

### Intermediate

4. How would you implement a logging decorator?
5. Why is a caching repository a decorator?
6. How can multiple decorators be chained?

### Advanced

7. How would you register decorators using dependency injection?
8. Decorator vs Middleware?
9. Decorator vs Proxy?
10. How would you design a production-ready caching decorator using `IMemoryCache`?

---

# Summary

The **Decorator Pattern** is deeply integrated into the .NET ecosystem and provides a flexible way to add behavior to existing objects without modifying their implementation. The `Stream` hierarchy (`FileStream`, `BufferedStream`, `GZipStream`, and `CryptoStream`) is one of the best-known examples, where each wrapper contributes a single capability while preserving the same interface. In enterprise ASP.NET Core applications, decorators are commonly used for **logging**, **validation**, **caching**, **retry policies**, **metrics**, and **authorization**, allowing cross-cutting concerns to remain separate from core business logic. By favoring composition over inheritance, decorators produce cleaner, more maintainable, and more extensible software.