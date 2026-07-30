# 35. Singleton Pattern

## Introduction

The **Singleton Pattern** is one of the **Gang of Four (GoF) Creational Design Patterns**.

It ensures that:

1. **Only one instance** of a class exists throughout the application's lifetime.
2. A **global access point** is provided to access that instance.

---

# Definition

> **Ensure a class has only one instance and provide a global point of access to it.**

---

# Why Do We Need Singleton?

Some resources should exist only once in an application.

Examples:

- Application configuration
- Logging service
- Cache manager
- License manager
- Feature flag manager

Having multiple instances may cause inconsistent behavior or unnecessary resource usage.

---

# Without Singleton

```csharp
Logger logger1 = new Logger();

Logger logger2 = new Logger();
```

Result:

```
Two different logger objects
```

This may lead to duplicate state or wasted resources.

---

# With Singleton

```csharp
Logger logger1 = Logger.Instance;

Logger logger2 = Logger.Instance;
```

Result:

```
logger1 == logger2
```

Both variables reference the same object.

---

# Singleton Structure

```
            Client

               │

               ▼

      Logger.Instance

               │

               ▼

        Single Object
```

---

# Basic Singleton Implementation

```csharp
public sealed class Logger
{
    private static readonly Logger _instance =
        new Logger();

    private Logger()
    {
    }

    public static Logger Instance
    {
        get
        {
            return _instance;
        }
    }

    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

---

# Usage

```csharp
Logger logger = Logger.Instance;

logger.Log("Application Started");
```

---

# Why `sealed`?

```csharp
public sealed class Logger
```

This prevents inheritance.

Without `sealed`, another class could inherit from `Logger`, potentially breaking the "single instance" guarantee.

---

# Why is the Constructor Private?

```csharp
private Logger()
{
}
```

A private constructor prevents external code from creating new instances.

This will fail:

```csharp
Logger logger = new Logger();
```

---

# How It Works Internally

```
Application Starts

↓

Static Field Initialized

↓

Logger Object Created

↓

Logger.Instance Returns Same Object
```

The instance is created only once and reused.

---

# Real-World Analogy

Imagine the **President of a company**.

There is only one president at a time.

Whenever anyone needs approval, they go to the same person instead of creating a new president.

This is similar to a Singleton.

---

# Singleton in ASP.NET Core

ASP.NET Core Dependency Injection supports the Singleton lifetime.

```csharp
builder.Services.AddSingleton<ILogger, FileLogger>();
```

Every request receives the same `FileLogger` instance.

---

# Common Uses

- Logging
- Caching
- Configuration
- Application settings
- Feature flags
- In-memory repositories (for demos)

---

# Advantages

- Single shared instance
- Reduced memory usage
- Centralized state
- Easy access
- Controlled object creation

---

# Disadvantages

- Global state
- Harder to unit test if overused
- Can introduce hidden dependencies
- Thread-safety must be considered

---

# 36. Thread-Safe Singleton

## Why Thread Safety Matters

In multi-threaded applications, two threads might try to create the Singleton at the same time.

Example:

```
Thread 1

↓

Instance == null

↓

Create Object
```

At the same moment:

```
Thread 2

↓

Instance == null

↓

Create Object
```

Now there are two instances, violating the Singleton pattern.

---

# Incorrect Implementation

```csharp
public class Logger
{
    private static Logger _instance;

    public static Logger Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = new Logger();
            }

            return _instance;
        }
    }

    private Logger()
    {
    }
}
```

This is **not thread-safe**.

---

# Thread-Safe Using Lock

```csharp
public sealed class Logger
{
    private static Logger _instance;

    private static readonly object _lock =
        new object();

    private Logger()
    {
    }

    public static Logger Instance
    {
        get
        {
            lock (_lock)
            {
                if (_instance == null)
                {
                    _instance = new Logger();
                }

                return _instance;
            }
        }
    }
}
```

Only one thread can enter the critical section at a time.

---

# Double-Checked Locking

A more efficient approach avoids locking after the instance has been created.

```csharp
public sealed class Logger
{
    private static Logger _instance;

    private static readonly object _lock =
        new object();

    private Logger()
    {
    }

    public static Logger Instance
    {
        get
        {
            if (_instance == null)
            {
                lock (_lock)
                {
                    if (_instance == null)
                    {
                        _instance = new Logger();
                    }
                }
            }

            return _instance;
        }
    }
}
```

The lock is used only during the first initialization.

---

# Thread Safety in ASP.NET Core

When registering a Singleton service:

```csharp
builder.Services.AddSingleton<ICacheService, CacheService>();
```

The DI container guarantees only one instance.

However, **your service implementation must still be thread-safe** if multiple requests access shared mutable state.

---

# Best Practice

For modern C#, prefer:

- Static initialization
- `Lazy<T>`
- ASP.NET Core DI container

instead of writing manual locking code.

---

# 37. Lazy Singleton

## What is Lazy Initialization?

Lazy initialization means:

> **Create the object only when it is needed.**

Without lazy initialization:

```
Application Starts

↓

Object Created
```

Even if it is never used.

With lazy initialization:

```
Application Starts

↓

Object Not Created

↓

First Request

↓

Object Created
```

---

# Why Use Lazy Initialization?

Imagine an expensive object:

- Loads configuration
- Opens database connections
- Reads large files
- Loads machine learning models

There is no need to create it unless it is actually used.

---

# Lazy Singleton Using `Lazy<T>`

```csharp
public sealed class Logger
{
    private static readonly Lazy<Logger> _instance =
        new Lazy<Logger>(() => new Logger());

    private Logger()
    {
    }

    public static Logger Instance
    {
        get
        {
            return _instance.Value;
        }
    }
}
```

---

# Usage

```csharp
Logger logger = Logger.Instance;

logger.Log("Hello");
```

The `Logger` object is created only when `Instance` is accessed for the first time.

---

# Internal Flow

```
Application Starts

↓

Lazy Object Created

↓

No Logger Yet

↓

First Instance Access

↓

Logger Created

↓

Subsequent Calls Reuse Same Instance
```

---

# Benefits of `Lazy<T>`

- Thread-safe by default
- Simple implementation
- No manual locks
- Delays expensive initialization
- Recommended for modern .NET applications

---

# 38. Singleton Pitfalls

Although Singleton is useful, it is one of the most frequently misused design patterns.

---

# Pitfall 1: Global State

A Singleton acts like a global variable.

Example:

```
OrderService

↓

Logger

↑

PaymentService
```

Any part of the application can modify shared state, making debugging difficult.

---

# Pitfall 2: Hidden Dependencies

Instead of using Dependency Injection:

```csharp
Logger.Instance.Log("Saved");
```

The dependency is hidden.

Better:

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

Dependencies are explicit and easier to test.

---

# Pitfall 3: Difficult Unit Testing

Suppose the Singleton stores internal state.

```csharp
Logger.Instance.Log("A");
```

One test may affect another because the same instance is shared across tests.

Mocking is also difficult.

---

# Pitfall 4: Thread Safety

Mutable shared data can lead to race conditions.

Example:

```csharp
public int Counter;

Counter++;
```

Multiple threads incrementing `Counter` simultaneously may produce incorrect results.

Synchronization or thread-safe collections may be required.

---

# Pitfall 5: Lifetime Issues

A Singleton remains alive for the application's lifetime.

Avoid storing:

- Request-specific data
- User-specific data
- Database contexts
- Temporary state

These belong to shorter lifetimes such as Scoped or Transient.

---

# Pitfall 6: Memory Leaks

A Singleton holding large objects indefinitely can prevent memory from being reclaimed.

Example:

```
Singleton

↓

Large Cache

↓

Large Images

↓

Application Lifetime
```

If data is never released, memory usage continues to grow.

---

# Pitfall 7: Breaking SOLID

Overusing Singletons may violate:

- **SRP**: One class manages business logic and global state.
- **DIP**: Classes depend directly on `Singleton.Instance` instead of abstractions.

---

# Singleton vs Static Class

| Singleton | Static Class |
|-----------|--------------|
| One object instance | No object instance |
| Can implement interfaces | Cannot implement interfaces |
| Can be injected using DI | Cannot be injected |
| Can maintain instance state | Only static state |
| Supports inheritance (unless sealed) | Cannot inherit |
| Better for testability | Difficult to mock |

---

# Singleton vs Scoped vs Transient (ASP.NET Core)

| Lifetime | Instances | Typical Use |
|----------|-----------|-------------|
| Singleton | One for the entire application | Configuration, caching, logging |
| Scoped | One per HTTP request | DbContext, business services |
| Transient | New instance every time | Lightweight, stateless services |

---

# Best Practices

- Keep Singleton classes stateless whenever possible.
- Prefer ASP.NET Core DI `AddSingleton()` over manual Singleton implementations.
- Use `Lazy<T>` for expensive initialization.
- Avoid storing request or user-specific data.
- Make shared mutable state thread-safe.
- Inject abstractions instead of accessing `Singleton.Instance` throughout the codebase.

---

# Interview Questions

### Basic

1. What is the Singleton Pattern?
2. Why is the constructor private?
3. Why is the class often marked as `sealed`?

### Intermediate

4. What is a thread-safe Singleton?
5. What is lazy initialization?
6. What is the difference between Singleton and a static class?

### Advanced

7. Why is `Lazy<T>` preferred for modern Singleton implementations?
8. What are the risks of using Singleton in multi-threaded applications?
9. When should you use `AddSingleton()` in ASP.NET Core?
10. What are the common pitfalls of overusing the Singleton Pattern?

---

# Summary

The **Singleton Pattern** ensures that only one instance of a class exists and provides a global access point to it. In modern .NET applications, **thread-safe** implementations are essential because multiple threads may access the same object concurrently. The **Lazy Singleton** delays object creation until it is actually needed and is typically implemented using `Lazy<T>`, which is thread-safe and concise. While Singletons are useful for shared services such as configuration and logging, they should be used carefully to avoid global state, hidden dependencies, thread-safety issues, and reduced testability. In ASP.NET Core, the built-in Dependency Injection container with `AddSingleton()` is generally the preferred approach over manually implementing the pattern.