# 56. Singleton in ILogger

## Introduction

One of the most common interview questions in ASP.NET Core is:

> **Is `ILogger` a Singleton?**

The answer is:

> **The logging infrastructure is registered as Singleton, but the injected `ILogger<T>` behaves like a lightweight logger associated with the requested type.**

This distinction is important.

---

# How Logging Works in ASP.NET Core

When the application starts:

```csharp
builder.Logging.AddConsole();

builder.Logging.AddDebug();
```

ASP.NET Core builds a logging pipeline.

```
Application

↓

Logging Factory

↓

Console Logger

↓

Debug Logger

↓

File Logger

↓

Application Insights
```

---

# Registering Logging

Normally you don't register `ILogger` manually.

```csharp
var builder =
    WebApplication.CreateBuilder(args);
```

Internally, ASP.NET Core registers:

- `ILoggerFactory`
- `ILogger<T>`
- Logging providers

---

# What Gets Registered?

Internally (simplified):

```
ILoggerFactory

↓

Singleton

↓

Creates ILogger<T>

↓

Controllers

↓

Services
```

---

# Example

```csharp
public class OrderService
{
    private readonly ILogger<OrderService>
        _logger;

    public OrderService(
        ILogger<OrderService> logger)
    {
        _logger = logger;
    }

    public void PlaceOrder()
    {
        _logger.LogInformation(
            "Order Created");
    }
}
```

You never instantiate the logger yourself.

The DI container injects it automatically.

---

# Internal Workflow

```
Application Starts

↓

LoggerFactory Created

↓

Singleton

↓

Controller Requested

↓

LoggerFactory Creates ILogger<OrderController>

↓

Logger Returned
```

---

# Why Singleton?

Creating logging infrastructure repeatedly would be expensive.

```
File Logger

↓

Open File

↓

Initialize Resources

↓

Write Logs
```

Instead:

```
One Logging Infrastructure

↓

Shared by Entire Application
```

---

# Logging Flow

```
Controller

↓

ILogger<T>

↓

LoggerFactory

↓

Console Logger

↓

Application Insights

↓

File Logger
```

Each provider receives the log message.

---

# Is ILogger Thread-Safe?

Yes.

The built-in logging infrastructure is designed for concurrent access.

Multiple requests can safely log simultaneously.

---

# Real-World Analogy

Think of a newspaper printing press.

```
Many Reporters

↓

One Printing System

↓

Newspapers Printed
```

The printing system is shared.

Each reporter submits articles independently.

---

# Advantages

- Efficient
- Thread-safe
- Shared infrastructure
- Minimal allocations
- Centralized logging

---

# 57. Lifetime Issues with Singleton

## Introduction

A Singleton exists for the **entire lifetime of the application**.

This means every request uses the same instance.

While this is useful, it can also cause problems if dependencies have shorter lifetimes.

---

# Service Lifetimes

```
Singleton

↓

Application Lifetime
```

```
Scoped

↓

One HTTP Request
```

```
Transient

↓

Every Resolution
```

---

# The Biggest Problem

### Singleton depending on Scoped Service

Example:

```csharp
builder.Services.AddSingleton<
    ReportService>();

builder.Services.AddScoped<
    AppDbContext>();
```

Now:

```csharp
public class ReportService
{
    public ReportService(
        AppDbContext db)
    {
    }
}
```

This is **incorrect**.

---

# Why?

```
Singleton

↓

AppDbContext (Scoped)
```

The Singleton lives forever.

The DbContext exists only for one request.

Eventually:

```
Singleton

↓

Disposed DbContext
```

This causes runtime failures or incorrect behavior.

---

# ASP.NET Core Prevents This

When the application starts, you may see an error similar to:

```
Cannot consume scoped service
from singleton.
```

This is a built-in lifetime validation.

---

# Why is DbContext Scoped?

```
Request

↓

DbContext

↓

Database

↓

Dispose
```

Each request gets its own `DbContext`.

Sharing one across requests would lead to concurrency issues.

---

# Correct Solutions

## Solution 1: Make Both Scoped

```csharp
builder.Services.AddScoped<
    ReportService>();

builder.Services.AddScoped<
    AppDbContext>();
```

Both now have the same lifetime.

---

## Solution 2: Use IServiceScopeFactory

Sometimes a Singleton genuinely needs to perform work with a Scoped service.

Example:

```csharp
public class Worker
{
    private readonly IServiceScopeFactory
        _scopeFactory;

    public Worker(
        IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    public void Execute()
    {
        using var scope =
            _scopeFactory.CreateScope();

        var db =
            scope.ServiceProvider
                 .GetRequiredService<
                    AppDbContext>();
    }
}
```

This creates a temporary scope and resolves the `DbContext` safely.

---

# Singleton Depending on Transient

```
Singleton

↓

Transient
```

This is allowed.

However:

```
Singleton Created

↓

Transient Created Once

↓

Same Instance Forever
```

The transient effectively behaves like a Singleton because it is captured by the Singleton.

If a fresh instance is required each time, resolve it inside a scope or redesign the service.

---

# Mutable State

Bad example:

```csharp
public class Counter
{
    public int Count;
}
```

Registered as Singleton:

```
Request 1

↓

Count++

Request 2

↓

Count++
```

All requests share the same state.

Without synchronization, race conditions can occur.

---

# Thread Safety

Singletons must be thread-safe.

Unsafe:

```csharp
_counter++;
```

Safer approaches include:

- `Interlocked.Increment()`
- `ConcurrentDictionary`
- Locks when appropriate

---

# Lifetime Rules

| Consumer | Dependency | Valid? |
|----------|------------|--------|
| Singleton | Singleton | ✅ Yes |
| Singleton | Scoped | ❌ No |
| Singleton | Transient | ✅ Yes (use carefully) |
| Scoped | Singleton | ✅ Yes |
| Scoped | Scoped | ✅ Yes |
| Scoped | Transient | ✅ Yes |
| Transient | Singleton | ✅ Yes |
| Transient | Scoped | ✅ Yes |
| Transient | Transient | ✅ Yes |

---

# Best Practices

- Keep Singletons stateless whenever possible.
- Never inject `DbContext` directly into a Singleton.
- Avoid storing request-specific or user-specific data.
- Use `IServiceScopeFactory` if a Singleton occasionally needs Scoped services.
- Ensure all mutable shared state is thread-safe.

---

# 58. Singleton in Distributed Systems

## Introduction

A common misconception is:

> **Singleton means there is only one instance in the entire system.**

This is **not true** in distributed environments.

Singleton means:

> **One instance per application process (or DI container).**

---

# Single Server

```
Server

↓

One Application

↓

One Singleton
```

Only one instance exists.

---

# Multiple Servers

Suppose an application runs on three servers.

```
Load Balancer

↓

Server A

↓

Singleton A

Server B

↓

Singleton B

Server C

↓

Singleton C
```

There are now **three Singleton instances**.

---

# Kubernetes Example

```
Kubernetes

↓

Pod 1

↓

Singleton

Pod 2

↓

Singleton

Pod 3

↓

Singleton
```

Each pod has its own Singleton.

---

# Azure App Service

Scaling from one to five instances:

```
App Service

↓

Instance 1

↓

Singleton

Instance 2

↓

Singleton

...

Instance 5

↓

Singleton
```

Five application instances mean five Singleton instances.

---

# Why This Matters

Imagine:

```csharp
public class Counter
{
    public int Count;
}
```

Requests:

```
Server A

Count = 10

Server B

Count = 3

Server C

Count = 18
```

Each server has a different value.

Singleton state is **not shared across servers**.

---

# Shared Cache Instead

Use distributed infrastructure.

```
Application

↓

Redis

↓

Shared Data
```

Every application instance accesses the same cache.

---

# Leader Election

Sometimes only one application instance should perform a task.

Examples:

- Nightly reports
- Billing jobs
- Cleanup tasks

A Singleton cannot guarantee this across multiple servers.

Solutions include:

- Distributed locks
- Leader election
- Job schedulers

---

# Distributed Lock Example

```
Server A

↓

Acquire Lock

↓

Run Job

Server B

↓

Cannot Acquire Lock
```

Only one server performs the work.

---

# Common Technologies

Instead of Singleton state:

- Redis
- SQL Server
- Azure Service Bus
- Azure Blob Lease
- Distributed caches
- Distributed lock providers

These coordinate work across multiple application instances.

---

# Singleton vs Distributed Singleton

| Singleton | Distributed Singleton |
|-----------|-----------------------|
| One instance per process | One logical instance across the entire system |
| Managed by DI container | Managed using distributed coordination |
| No network communication | Requires shared infrastructure |
| Suitable for a single application instance | Suitable for scaled-out systems |

---

# Real-World Analogy

A company has offices in:

- Bangalore
- London
- New York

Each office has **one manager**.

```
Office

↓

Manager
```

There isn't one manager for the entire company—there is one manager **per office**.

Similarly, a Singleton exists per application instance, not per distributed deployment.

---

# Best Practices

- Treat Singleton as **per application instance**, not globally unique.
- Do not store business-critical shared state in Singleton services.
- Use distributed caches or databases for shared state.
- Use distributed locking or leader election for jobs that must run only once across a cluster.
- Keep Singleton services focused on stateless infrastructure such as logging, configuration, or caching abstractions.

---

# Interview Questions

### Basic

1. Is `ILogger<T>` a Singleton?
2. Why is the logging infrastructure shared?
3. What is the difference between Singleton and Scoped services?

### Intermediate

4. Why can't a Singleton depend directly on a Scoped service?
5. How can a Singleton safely use `DbContext`?
6. Why must Singleton services be thread-safe?

### Advanced

7. What happens to Singleton services when an ASP.NET Core application scales to multiple servers?
8. Why can't a Singleton coordinate scheduled jobs across Kubernetes pods?
9. How would you implement a distributed lock in a cloud-native application?
10. When should you choose Redis over a Singleton for shared application state?

---

# Summary

In ASP.NET Core, the **logging infrastructure** is built around shared singleton components such as `ILoggerFactory`, while `ILogger<T>` provides a type-specific logging interface for consumers. Singleton services live for the **entire application lifetime**, so they must be thread-safe and should not depend directly on Scoped services like `DbContext`. When a Singleton occasionally requires a Scoped dependency, `IServiceScopeFactory` should be used to create a temporary scope. Finally, in **distributed systems**, a Singleton is **not globally unique**—each application instance, server, or Kubernetes pod has its own Singleton. Shared state and single-execution guarantees require distributed technologies such as Redis, distributed locks, databases, or leader-election mechanisms.