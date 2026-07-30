# 41. Object Pool Pattern

## Introduction

The **Object Pool Pattern** is a **Creational Design Pattern** that manages a pool of reusable objects instead of creating and destroying them repeatedly.

Instead of:

```
Create Object

↓

Use Object

↓

Destroy Object
```

we use:

```
Create Pool

↓

Borrow Object

↓

Use Object

↓

Return Object

↓

Reuse Object
```

The same objects are reused multiple times.

---

# Definition

> **Reuse expensive objects instead of creating and destroying them repeatedly.**

---

# Why Do We Need Object Pool?

Creating some objects is expensive.

Examples:

- Database connections
- Network sockets
- HttpClient handlers
- Large buffers
- Machine Learning models
- Image processors

Creating them repeatedly wastes:

- CPU
- Memory
- Time

Pooling avoids unnecessary object creation.

---

# Real-World Analogy

Imagine a library.

Without pooling:

```
Need Book

↓

Print New Book

↓

Read

↓

Throw Away
```

Very expensive.

With pooling:

```
Need Book

↓

Borrow

↓

Read

↓

Return

↓

Next Person Uses Same Book
```

The same book is reused.

---

# Without Object Pool

```csharp
for (int i = 0; i < 1000; i++)
{
    var parser = new JsonParser();

    parser.Parse();
}
```

Result:

```
1000 Objects Created

↓

1000 Objects Destroyed
```

This increases memory allocations and garbage collection pressure.

---

# With Object Pool

```
Pool

↓

Parser #1

Parser #2

Parser #3

Parser #4
```

Workflow:

```
Borrow Parser

↓

Parse

↓

Return Parser
```

No new object is created unless the pool is exhausted.

---

# Object Pool Structure

```
               Client

                  │

                  ▼

            Object Pool

          ▲            ▼

     Return         Borrow

          ▲            ▼

        Reusable Objects
```

---

# Simple Implementation

## Product

```csharp
public class Parser
{
    public void Parse()
    {
        Console.WriteLine("Parsing...");
    }
}
```

---

## Pool

```csharp
public class ParserPool
{
    private readonly Queue<Parser> _pool =
        new();

    public ParserPool(int size)
    {
        for (int i = 0; i < size; i++)
        {
            _pool.Enqueue(new Parser());
        }
    }

    public Parser Get()
    {
        return _pool.Count > 0
            ? _pool.Dequeue()
            : new Parser();
    }

    public void Return(Parser parser)
    {
        _pool.Enqueue(parser);
    }
}
```

---

## Client

```csharp
ParserPool pool = new ParserPool(5);

Parser parser = pool.Get();

parser.Parse();

pool.Return(parser);
```

---

# Internal Workflow

```
Application Starts

↓

Pool Creates 5 Objects

↓

Client Requests Object

↓

Pool Gives Object

↓

Client Uses Object

↓

Client Returns Object

↓

Pool Stores Object

↓

Next Client Reuses It
```

---

# Benefits

- Fewer object allocations
- Less garbage collection
- Better performance
- Lower memory usage
- Reduced initialization cost
- Better throughput

---

# When Should You Use Object Pool?

Use it when:

- Object creation is expensive.
- Objects are reused frequently.
- The object is expensive to initialize.
- Performance is critical.

Examples:

- Database connections
- HTTP handlers
- Serialization buffers
- Image processing
- AI inference models
- Compression streams

---

# When Should You NOT Use It?

Do **not** pool:

- Small lightweight objects
- Immutable objects
- Value types
- Objects that are cheap to create

Example:

```csharp
new Point(10,20)
```

Pooling such objects usually adds unnecessary complexity.

---

# Object Pool vs Singleton

| Object Pool | Singleton |
|--------------|-----------|
| Many reusable objects | One shared object |
| Borrow and return | Always same instance |
| Multiple users simultaneously | One global instance |
| Focuses on reuse | Focuses on uniqueness |

---

# Object Pool vs Factory

| Object Pool | Factory |
|--------------|---------|
| Reuses existing objects | Creates new objects |
| Better for expensive resources | Better for selecting implementations |
| Manages object lifetime | Focuses on object creation |

---

# 42. Pooling in .NET

.NET provides several built-in pooling mechanisms to improve performance and reduce memory allocations.

Instead of implementing pools manually, it is usually better to use the framework-provided solutions.

---

# 1. Database Connection Pooling (ADO.NET)

This is the most common example.

Without pooling:

```
Open Connection

↓

Close Connection

↓

Destroy Connection
```

For every request.

---

With pooling:

```
Application

↓

Connection Pool

↓

Reuse Existing Connection
```

Connections are returned to the pool when closed and reused for future requests.

---

## Example

```csharp
using var connection =
    new SqlConnection(connectionString);

connection.Open();
```

Even though a new `SqlConnection` object is created, the underlying physical database connection is typically reused from the connection pool.

---

# 2. ASP.NET Core ObjectPool<T>

ASP.NET Core provides an object pooling library.

Namespace:

```csharp
using Microsoft.Extensions.ObjectPool;
```

---

## Create a Pool

```csharp
var provider =
    new DefaultObjectPoolProvider();

ObjectPool<StringBuilder> pool =
    provider.CreateStringBuilderPool();
```

---

## Borrow an Object

```csharp
StringBuilder builder = pool.Get();

builder.Append("Hello");
```

---

## Return the Object

```csharp
builder.Clear();

pool.Return(builder);
```

The next request can reuse the same `StringBuilder`.

---

# Why Pool `StringBuilder`?

Without pooling:

```
1000 Requests

↓

1000 StringBuilders

↓

Garbage Collection
```

With pooling:

```
Pool

↓

10 StringBuilders

↓

Reused Thousands of Times
```

This reduces allocations and improves performance.

---

# 3. HttpClient and IHttpClientFactory

Creating a new `HttpClient` for every request can lead to socket exhaustion.

Instead, ASP.NET Core provides `IHttpClientFactory`.

Registration:

```csharp
builder.Services.AddHttpClient();
```

Usage:

```csharp
public class WeatherService
{
    private readonly HttpClient _client;

    public WeatherService(
        HttpClient client)
    {
        _client = client;
    }
}
```

The factory manages the underlying handlers efficiently and reuses them.

---

# 4. ArrayPool<T>

Large arrays are expensive to allocate.

Use `ArrayPool<T>`:

```csharp
using System.Buffers;

var pool = ArrayPool<byte>.Shared;

byte[] buffer = pool.Rent(1024);
```

Use:

```csharp
// Work with buffer
```

Return:

```csharp
pool.Return(buffer);
```

The array can now be reused.

---

# 5. MemoryPool<T>

For high-performance scenarios:

```csharp
MemoryPool<byte>.Shared
```

Commonly used in:

- Kestrel
- Pipelines
- Networking
- High-throughput APIs

---

# 6. RecyclableMemoryStream

Instead of repeatedly allocating large `MemoryStream` objects, libraries such as `Microsoft.IO.RecyclableMemoryStream` reuse internal buffers to reduce memory fragmentation and garbage collection pressure.

---

# Object Pool in ASP.NET Core

Example:

```csharp
builder.Services.AddSingleton<
    ObjectPoolProvider,
    DefaultObjectPoolProvider>();

builder.Services.AddSingleton(
    serviceProvider =>
    {
        var provider =
            serviceProvider.GetRequiredService<ObjectPoolProvider>();

        return provider.CreateStringBuilderPool();
    });
```

The pool can then be injected into services using Dependency Injection.

---

# Advantages of Pooling in .NET

- Lower GC pressure
- Better throughput
- Lower latency
- Reduced allocations
- Better scalability
- Improved performance under load

---

# Potential Pitfalls

### 1. Forgetting to Return Objects

```csharp
var builder = pool.Get();

// Forgot pool.Return(builder);
```

Objects remain unavailable to the pool.

---

### 2. Returning Dirty Objects

Always reset the object's state.

Example:

```csharp
builder.Clear();

pool.Return(builder);
```

---

### 3. Pooling Cheap Objects

Pooling simple objects often provides little or no benefit while increasing complexity.

---

### 4. Sharing Stateful Objects

If a pooled object retains state, one user may accidentally observe data from a previous user.

Always reset mutable state before returning an object to the pool.

---

# Object Pool vs Connection Pool

| Object Pool | Connection Pool |
|--------------|----------------|
| Pools any reusable object | Pools database connections |
| General-purpose | Database-specific |
| Managed by application or framework | Managed by ADO.NET provider |
| Examples: `StringBuilder`, buffers | SQL Server, PostgreSQL, Oracle connections |

---

# Interview Questions

### Basic

1. What is the Object Pool Pattern?
2. Why is object pooling useful?
3. Give a real-world example of object pooling.

### Intermediate

4. What is the difference between Object Pool and Singleton?
5. When should you avoid object pooling?
6. Why is `StringBuilder` commonly pooled?

### Advanced

7. Explain `ObjectPool<T>` in ASP.NET Core.
8. What is `ArrayPool<T>` and when would you use it?
9. Why is `IHttpClientFactory` preferred over creating new `HttpClient` instances?
10. What problems can occur if pooled objects are not reset before reuse?

---

# Summary

The **Object Pool Pattern** is a creational design pattern that improves performance by **reusing expensive objects** instead of repeatedly creating and destroying them. It is especially valuable for resources such as database connections, buffers, parsers, and large reusable objects. .NET provides built-in pooling mechanisms—including **ADO.NET connection pooling**, **`ObjectPool<T>`**, **`ArrayPool<T>`**, **`MemoryPool<T>`**, and **`IHttpClientFactory`**—to reduce memory allocations, minimize garbage collection, and improve application scalability. The key to successful pooling is to use it only for expensive reusable objects and to always return them to the pool in a clean state.