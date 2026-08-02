# 108. Chain of Responsibility (CoR)

## Introduction

The **Chain of Responsibility (CoR)** is a **Behavioral Design Pattern** that allows a request to pass through a chain of handlers. Each handler decides whether it can process the request or pass it to the next handler in the chain.

> **Chain of Responsibility decouples the sender of a request from its receivers by giving multiple objects a chance to handle the request.**

---

# Definition

> **Avoid coupling the sender of a request to its receiver by giving multiple objects a chance to handle the request. Chain the receiving objects and pass the request along the chain until one handles it or the chain ends.**

---

# Why Do We Need Chain of Responsibility?

Imagine an API request that requires:

- Authentication
- Authorization
- Validation
- Logging
- Rate Limiting
- Business Processing

Without CoR:

```csharp
public void ProcessRequest(Request request)
{
    Authenticate(request);

    Authorize(request);

    Validate(request);

    Log(request);

    ProcessBusinessLogic(request);
}
```

Problems:

- Tightly coupled code
- Difficult to add or remove steps
- Violates Open/Closed Principle
- Hard to reuse individual steps

---

# Solution

Create a chain of handlers.

```
Request

↓

Authentication

↓

Authorization

↓

Validation

↓

Logging

↓

Business Logic
```

Each handler performs one responsibility and passes the request forward.

---

# Structure

```
              Client

                 │

                 ▼

             Handler

                 ▲

     ┌───────────┼───────────┐

     ▼           ▼           ▼

 Authentication Validation Logging
```

---

# Components

| Component | Responsibility |
|------------|----------------|
| Handler | Defines request processing contract |
| Concrete Handler | Processes request or forwards it |
| Client | Sends the request |
| Request | Data being processed |

---

# Example

## Handler

```csharp
public abstract class Handler
{
    protected Handler? Next;

    public void SetNext(Handler next)
    {
        Next = next;
    }

    public abstract void Handle(
        string request);
}
```

---

## Authentication Handler

```csharp
public class AuthenticationHandler
    : Handler
{
    public override void Handle(
        string request)
    {
        Console.WriteLine(
            "Authentication Passed");

        Next?.Handle(request);
    }
}
```

---

## Validation Handler

```csharp
public class ValidationHandler
    : Handler
{
    public override void Handle(
        string request)
    {
        Console.WriteLine(
            "Validation Passed");

        Next?.Handle(request);
    }
}
```

---

## Business Handler

```csharp
public class BusinessHandler
    : Handler
{
    public override void Handle(
        string request)
    {
        Console.WriteLine(
            "Business Logic Executed");
    }
}
```

---

## Client

```csharp
var auth =
    new AuthenticationHandler();

var validation =
    new ValidationHandler();

var business =
    new BusinessHandler();

auth.SetNext(validation);
validation.SetNext(business);

auth.Handle("Create Order");
```

---

## Output

```
Authentication Passed

Validation Passed

Business Logic Executed
```

---

# Internal Workflow

```
Request

↓

Handler 1

↓

Handler 2

↓

Handler 3

↓

Completed
```

---

# Enterprise Example

Order Processing

```
HTTP Request

↓

Authentication

↓

Authorization

↓

Validation

↓

Logging

↓

Order Service
```

Every handler performs one task.

---

# Advantages

- Loose coupling
- Easy to add or remove handlers
- Follows Open/Closed Principle
- Improves maintainability
- Reusable handlers

---

# Disadvantages

- Harder to debug long chains
- Request may never be handled if the chain is configured incorrectly
- Performance overhead with very long chains

---

# Real-World Examples

- ASP.NET Core Middleware
- HTTP Pipelines
- Logging Pipelines
- Validation Pipelines
- MediatR Pipeline Behaviors
- Exception Handling Pipelines

---

# Best Practices

- Keep handlers focused on one responsibility.
- Avoid embedding business logic in infrastructure handlers.
- Build chains dynamically when possible.
- Short-circuit the chain when appropriate (e.g., authentication failure).

---

# 109. ASP.NET Core Middleware as Chain of Responsibility

## Why Middleware Uses CoR

ASP.NET Core Middleware is one of the best real-world implementations of the Chain of Responsibility Pattern.

Every incoming request passes through a sequence of middleware components.

Each middleware can:

- Process the request
- Modify the request
- Stop the pipeline
- Pass it to the next middleware
- Process the response on the way back

---

# Request Pipeline

```
HTTP Request

↓

Exception Middleware

↓

HTTPS Redirection

↓

Static Files

↓

Routing

↓

Authentication

↓

Authorization

↓

Endpoint

↓

HTTP Response
```

Each middleware decides whether to continue.

---

# ASP.NET Core Pipeline

```csharp
app.UseHttpsRedirection();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

Execution order:

```
Request

↓

HTTPS

↓

Authentication

↓

Authorization

↓

Controller

↓

Response
```

---

# Custom Middleware Example

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate
        _next;

    public LoggingMiddleware(
        RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(
        HttpContext context)
    {
        Console.WriteLine(
            "Request Started");

        await _next(context);

        Console.WriteLine(
            "Request Finished");
    }
}
```

---

# Registration

```csharp
app.UseMiddleware<
    LoggingMiddleware>();
```

---

# Internal Workflow

```
Request

↓

Logging Middleware

↓

Authentication Middleware

↓

Authorization Middleware

↓

Controller

↓

Response

↓

Authorization Middleware

↓

Authentication Middleware

↓

Logging Middleware
```

Notice that the response travels back through the chain in reverse order.

---

# Short-Circuiting the Pipeline

A middleware can stop further processing.

Example:

```csharp
public async Task InvokeAsync(
    HttpContext context)
{
    if (!context.User.Identity.IsAuthenticated)
    {
        context.Response.StatusCode = 401;
        return;
    }

    await _next(context);
}
```

If authentication fails:

```
Request

↓

Authentication

↓

401 Unauthorized

↓

End Pipeline
```

No subsequent middleware or controller executes.

---

# Middleware Order Matters

Incorrect order:

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

Authorization runs before the user is authenticated, causing authorization to fail.

Correct order:

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

---

# Middleware vs Chain of Responsibility

| Chain of Responsibility | ASP.NET Core Middleware |
|--------------------------|--------------------------|
| Handler | Middleware |
| Request | HttpContext |
| Next Handler | RequestDelegate |
| Client | HTTP Client |
| Chain | Middleware Pipeline |

---

# Middleware vs Decorator

| Middleware | Decorator |
|------------|-----------|
| Processes HTTP requests | Adds behavior to objects |
| Uses `RequestDelegate` chain | Wraps another object |
| Request/Response pipeline | Object composition |

---

# Middleware vs MediatR Pipeline

| Middleware | MediatR Pipeline Behavior |
|------------|---------------------------|
| Works on HTTP requests | Works on commands/queries |
| Executes before controllers | Executes before handlers |
| Uses `HttpContext` | Uses MediatR requests |

---

# Enterprise Middleware Pipeline

```
HTTP Request

↓

Exception Middleware

↓

Request Logging

↓

Rate Limiting

↓

Authentication

↓

Authorization

↓

Correlation ID

↓

Performance Monitoring

↓

Controller

↓

HTTP Response
```

Each middleware has a single responsibility.

---

# Benefits

- Modular request processing
- Easy to add or remove middleware
- Reusable components
- Supports cross-cutting concerns
- Aligns with SOLID principles

---

# Common Built-in Middleware

- Exception Handling
- HTTPS Redirection
- Static Files
- Routing
- CORS
- Authentication
- Authorization
- Response Compression
- Response Caching
- Rate Limiting

---

# Interview Questions

### Basic

1. What is the Chain of Responsibility Pattern?
2. What problem does it solve?
3. Give a real-world example.

### Intermediate

4. Why is ASP.NET Core Middleware an example of Chain of Responsibility?
5. What is `RequestDelegate`?
6. What happens if middleware doesn't call `await _next(context)`?

### Advanced

7. How does short-circuiting work in middleware?
8. Why is middleware order important?
9. Chain of Responsibility vs Decorator?
10. Middleware vs MediatR Pipeline Behaviors?

---

# Summary

The **Chain of Responsibility Pattern** is a behavioral design pattern that passes a request through a sequence of independent handlers, allowing each handler to process the request or forward it to the next one. In **ASP.NET Core**, the middleware pipeline is a direct implementation of this pattern. Each middleware performs a focused responsibility, such as authentication, logging, or authorization, before delegating to the next component through `RequestDelegate`. The pipeline supports modularity, extensibility, short-circuiting, and clean separation of concerns, making it a fundamental part of modern ASP.NET Core applications.