# 73. Proxy Pattern

## Introduction

The **Proxy Pattern** is a **Structural Design Pattern** that provides a **placeholder or surrogate object** to control access to another object.

Instead of communicating directly with the real object, the client communicates with a **Proxy**, which decides how and when the real object should be accessed.

The proxy may:

- Delay object creation
- Check permissions
- Log requests
- Cache results
- Connect to remote services

> **Proxy controls access to an object without changing its interface.**

---

# Definition

> **The Proxy Pattern provides a surrogate or placeholder for another object to control access to it.**

---

# Why Do We Need Proxy?

Suppose loading an image takes several seconds.

Without Proxy

```
Application

↓

Load Large Image

↓

Display
```

The application always pays the loading cost.

With Proxy

```
Application

↓

Image Proxy

↓

Load Only When Needed

↓

Display
```

---

# Real-World Analogy

Imagine entering a company building.

```
Visitor

↓

Security Guard

↓

Office
```

The visitor doesn't directly access the office.

The guard verifies identity before allowing entry.

The security guard acts as a **Proxy**.

---

# Structure

```
             Client

                │

                ▼

           IImage

           ▲     ▲

           │     │

      RealImage ProxyImage
```

Both Proxy and Real Object implement the same interface.

---

# Components

| Component | Responsibility |
|------------|----------------|
| Subject | Common interface |
| Real Subject | Actual implementation |
| Proxy | Controls access |
| Client | Uses the subject interface |

---

# Example

## Subject

```csharp
public interface IImage
{
    void Display();
}
```

---

## Real Object

```csharp
public class RealImage : IImage
{
    public RealImage()
    {
        Console.WriteLine(
            "Loading image...");
    }

    public void Display()
    {
        Console.WriteLine(
            "Displaying image");
    }
}
```

---

## Proxy

```csharp
public class ImageProxy : IImage
{
    private RealImage? _image;

    public void Display()
    {
        _image ??= new RealImage();

        _image.Display();
    }
}
```

---

## Client

```csharp
IImage image = new ImageProxy();

image.Display();

image.Display();
```

Output

```
Loading image...

Displaying image

Displaying image
```

Notice that the image is loaded only once.

---

# Internal Workflow

```
Client

↓

Proxy

↓

Real Object

↓

Operation
```

The proxy decides whether to create, reuse, authorize, or log access.

---

# Advantages

- Controls access
- Improves performance
- Supports lazy loading
- Adds security
- Adds logging and caching

---

# Disadvantages

- Extra layer of abstraction
- More classes
- Can increase complexity

---

# 74. Types of Proxy

There are several commonly used Proxy variants.

```
Proxy

├── Virtual Proxy

├── Protection Proxy

├── Remote Proxy

├── Smart Proxy
```

Each solves a different problem.

---

# Comparison

| Type | Purpose |
|-------|----------|
| Virtual Proxy | Lazy initialization |
| Protection Proxy | Authorization |
| Remote Proxy | Access remote objects |
| Smart Proxy | Add extra behavior |

---

# 75. Virtual Proxy

## Purpose

A **Virtual Proxy** delays the creation of an expensive object until it is actually needed.

---

### Example

```
Application

↓

Image Proxy

↓

Real Image

↓

Load File
```

The image is loaded only when `Display()` is called.

---

### Code

```csharp
public class ImageProxy : IImage
{
    private RealImage? _image;

    public void Display()
    {
        _image ??= new RealImage();

        _image.Display();
    }
}
```

---

### Real-World Example

PDF Viewer

```
Open PDF

↓

First Page Loaded

↓

Remaining Pages

↓

Loaded When Scrolled
```

---

### ASP.NET Core Example

Lazy loading large reports.

```
Controller

↓

Report Proxy

↓

Generate Report

↓

Return PDF
```

The expensive report generation happens only when required.

---

# Advantages

- Faster startup
- Lower memory usage
- Better performance
- Lazy initialization

---

# 76. Protection Proxy

## Purpose

Controls access based on permissions.

Only authorized users can access the real object.

---

### Workflow

```
User

↓

Proxy

↓

Check Permission

↓

Real Service
```

---

### Example

```csharp
public class SecureDocumentProxy
    : IDocument
{
    private readonly RealDocument
        _document = new();

    public void Open(string role)
    {
        if (role != "Admin")
        {
            Console.WriteLine(
                "Access Denied");

            return;
        }

        _document.Open(role);
    }
}
```

---

### Output

```
User

↓

Access Denied
```

or

```
Admin

↓

Document Opened
```

---

### ASP.NET Core Example

```
Controller

↓

Authorization Proxy

↓

Business Service
```

Although ASP.NET Core commonly uses authorization middleware and filters, the underlying concept is similar—access is verified before the protected operation executes.

---

# Advantages

- Centralized authorization
- Prevents unauthorized access
- Keeps security separate from business logic

---

# 77. Remote Proxy

## Purpose

Represents an object that exists in another process, machine, or service.

The client interacts with the proxy as though it were local.

---

### Workflow

```
Application

↓

Remote Proxy

↓

HTTP

↓

Remote Server
```

---

### Example

Suppose an application calls an external Weather API.

```
Weather Service

↓

Proxy

↓

HTTP Request

↓

Weather API
```

The client does not deal with networking details.

---

### ASP.NET Core Example

```csharp
public interface IWeatherService
{
    Task<Weather> GetAsync();
}
```

Implementation:

```
IWeatherService

↓

HttpClient

↓

Weather API
```

The service acts as a remote proxy over the HTTP endpoint.

---

### Modern Examples

- REST APIs
- gRPC clients
- SOAP clients
- Database gateways
- Microservices

---

# Advantages

- Hides networking complexity
- Simplifies remote communication
- Keeps client code clean

---

# 78. Smart Proxy

## Purpose

A **Smart Proxy** performs additional work whenever an object is accessed.

Typical responsibilities include:

- Logging
- Caching
- Reference counting
- Performance measurement
- Transactions

---

### Workflow

```
Client

↓

Smart Proxy

↓

Log

↓

Measure Time

↓

Real Service
```

---

### Example

```csharp
public class LoggingProxy
    : IService
{
    private readonly IService
        _service;

    public LoggingProxy(
        IService service)
    {
        _service = service;
    }

    public void Execute()
    {
        Console.WriteLine(
            "Before");

        _service.Execute();

        Console.WriteLine(
            "After");
    }
}
```

---

### Output

```
Before

Real Service

After
```

---

### ASP.NET Core Example

```
Controller

↓

Logging Proxy

↓

Repository

↓

Database
```

Every repository call is logged without modifying repository code.

---

# Proxy Pattern in ASP.NET Core

Proxy-like concepts appear throughout the framework.

### Lazy<T>

```
Object

↓

Lazy Proxy

↓

Create When Needed
```

---

### HttpClient

```
Application

↓

HttpClient

↓

Remote API
```

The client acts as a local abstraction over remote communication.

---

### Caching Repository

```
Repository Proxy

↓

Memory Cache

↓

Database
```

The proxy decides whether to serve cached data or query the database.

---

### Authorization

```
User

↓

Authorization

↓

Controller
```

Access is checked before the protected action executes.

---

# Proxy vs Decorator

| Proxy | Decorator |
|--------|-----------|
| Controls access | Adds behavior |
| May deny or delay execution | Always enhances behavior |
| Focuses on access management | Focuses on feature extension |
| Client often doesn't know a proxy exists | Client intentionally composes decorators |

---

# Proxy vs Adapter

| Proxy | Adapter |
|--------|---------|
| Same interface | Different interface |
| Controls access | Converts interfaces |
| Wraps an existing object | Translates incompatible APIs |

---

# Proxy vs Facade

| Proxy | Facade |
|--------|---------|
| Represents one object | Simplifies many objects |
| Controls access | Simplifies complexity |
| Same interface | Usually exposes a different simplified interface |

---

# Best Practices

- Keep the proxy implementing the same interface as the real object.
- Use Virtual Proxy for expensive object creation.
- Use Protection Proxy for authorization and access control.
- Use Remote Proxy to hide networking details.
- Use Smart Proxy for cross-cutting concerns like logging, caching, and metrics.
- Keep business logic inside the real object rather than the proxy.

---

# Real-World Uses

- Lazy image loading
- Authorization
- REST API clients
- gRPC clients
- Database repositories
- Distributed caching
- Logging
- Monitoring
- Performance measurement

---

# Interview Questions

### Basic

1. What is the Proxy Pattern?
2. Why do we use a Proxy instead of calling the object directly?
3. What are the different types of Proxy?

### Intermediate

4. What is the difference between Virtual Proxy and Smart Proxy?
5. How does a Protection Proxy improve security?
6. How does a Remote Proxy simplify distributed systems?

### Advanced

7. How is `Lazy<T>` related to the Virtual Proxy pattern?
8. How would you implement a caching repository using a Smart Proxy?
9. Proxy vs Decorator vs Adapter?
10. Design a secure document management system using different Proxy types.

---

# Summary

The **Proxy Pattern** is a structural design pattern that introduces a surrogate object to control access to another object while exposing the same interface. Different proxy types address different concerns: **Virtual Proxy** delays expensive object creation through lazy initialization, **Protection Proxy** enforces authorization, **Remote Proxy** represents objects located on remote systems, and **Smart Proxy** adds cross-cutting capabilities such as logging, caching, monitoring, or performance measurement. In ASP.NET Core, proxy concepts appear in lazy loading, authorization, remote service clients, repository caching, and distributed applications, making the pattern an essential tool for building secure, scalable, and maintainable systems.