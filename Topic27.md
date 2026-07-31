# 65. Decorator Pattern

## Introduction

The **Decorator Pattern** is a **Structural Design Pattern** that allows you to **add new behavior to an existing object dynamically without modifying its source code**.

Instead of changing the original class or creating numerous subclasses, a decorator **wraps** the original object and adds extra functionality before or after delegating to it.

> **Decorator extends behavior, not the class.**

---

# Definition

> **The Decorator Pattern attaches additional responsibilities to an object dynamically while preserving its original interface.**

---

# Why Do We Need Decorator?

Suppose you have a notification service.

Basic notification:

```text
Send Email
```

Now the business requires:

- Logging
- Retry
- Validation
- Encryption
- Metrics

Without Decorator:

```
EmailService

↓

LoggedEmailService

↓

LoggedRetryEmailService

↓

LoggedRetryEncryptedEmailService
```

This quickly becomes unmanageable.

With Decorator:

```
Logging

↓

Retry

↓

Encryption

↓

Email Service
```

Each responsibility is independent and reusable.

---

# Real-World Analogy

Imagine ordering a coffee.

Base coffee:

```
Coffee
```

You can add:

```
Coffee

↓

Milk

↓

Sugar

↓

Whipped Cream
```

The coffee remains the same, but additional features are layered on top.

Each topping is a decorator.

---

# Structure

```
          Client

             │

             ▼

        IService

             ▲

      ┌──────┴──────┐

      │             │

Concrete Service  Decorator

                     │

                     ▼

              Concrete Service
```

---

# Components

| Component | Responsibility |
|-----------|----------------|
| Component | Common interface |
| Concrete Component | Original implementation |
| Decorator | Wraps another component |
| Concrete Decorator | Adds new behavior |

---

# Example

## Step 1: Component

```csharp
public interface INotifier
{
    void Send(string message);
}
```

---

## Step 2: Concrete Component

```csharp
public class EmailNotifier
    : INotifier
{
    public void Send(string message)
    {
        Console.WriteLine(
            $"Email: {message}");
    }
}
```

---

## Step 3: Base Decorator

```csharp
public abstract class NotifierDecorator
    : INotifier
{
    protected readonly INotifier
        _notifier;

    protected NotifierDecorator(
        INotifier notifier)
    {
        _notifier = notifier;
    }

    public virtual void Send(
        string message)
    {
        _notifier.Send(message);
    }
}
```

---

## Step 4: Logging Decorator

```csharp
public class LoggingDecorator
    : NotifierDecorator
{
    public LoggingDecorator(
        INotifier notifier)
        : base(notifier)
    {
    }

    public override void Send(
        string message)
    {
        Console.WriteLine(
            "Logging Request");

        base.Send(message);
    }
}
```

---

## Step 5: Retry Decorator

```csharp
public class RetryDecorator
    : NotifierDecorator
{
    public RetryDecorator(
        INotifier notifier)
        : base(notifier)
    {
    }

    public override void Send(
        string message)
    {
        Console.WriteLine(
            "Retry Enabled");

        base.Send(message);
    }
}
```

---

## Client

```csharp
INotifier notifier =
    new RetryDecorator(
        new LoggingDecorator(
            new EmailNotifier()));

notifier.Send("Welcome");
```

Output

```
Retry Enabled

Logging Request

Email: Welcome
```

---

# Internal Workflow

```
Client

↓

RetryDecorator

↓

LoggingDecorator

↓

EmailNotifier
```

Each decorator performs its work and delegates to the wrapped object.

---

# ASP.NET Core Example

Imagine a repository.

```csharp
public interface IProductRepository
{
    Task<Product> GetAsync(int id);
}
```

Original repository:

```csharp
public class ProductRepository
    : IProductRepository
{
    public Task<Product> GetAsync(int id)
    {
        // Database access
    }
}
```

Caching decorator:

```csharp
public class CachedProductRepository
    : IProductRepository
{
    private readonly IProductRepository
        _repository;

    public CachedProductRepository(
        IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<Product> GetAsync(
        int id)
    {
        Console.WriteLine("Checking Cache");

        return await _repository.GetAsync(id);
    }
}
```

The repository remains unchanged while caching behavior is added.

---

# Common Uses

- Logging
- Caching
- Retry
- Validation
- Authorization
- Metrics
- Compression
- Encryption
- Rate limiting

---

# Advantages

- Adds behavior dynamically
- Follows the Open/Closed Principle
- Avoids subclass explosion
- Promotes composition over inheritance
- Flexible and reusable

---

# Disadvantages

- More classes
- Object chains become deeper
- Debugging can be harder
- Order of decorators may affect behavior

---

# 66. Decorator Pattern vs Middleware

## Introduction

A very common interview question is:

> **Is ASP.NET Core Middleware an implementation of the Decorator Pattern?**

The answer is:

> **Middleware is heavily inspired by the Decorator Pattern.** Each middleware wraps the next middleware in the pipeline, creating a chain of responsibility that also resembles decoration.

---

# Middleware Pipeline

```text
Request

↓

Authentication

↓

Authorization

↓

Logging

↓

Routing

↓

Endpoint

↓

Response
```

Each middleware performs work before and/or after calling the next middleware.

---

# Middleware Example

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Before");

    await next();

    Console.WriteLine("After");
});
```

Workflow:

```
Before

↓

Next Middleware

↓

After
```

This is conceptually similar to a decorator wrapping another object.

---

# Decorator Workflow

```
LoggingDecorator

↓

RetryDecorator

↓

EmailService
```

Each decorator wraps another implementation of the same interface.

---

# Middleware Workflow

```
Logging Middleware

↓

Authentication Middleware

↓

Authorization Middleware

↓

Endpoint
```

Each middleware wraps the remainder of the request pipeline.

---

# Similarities

Both patterns:

- Wrap another component
- Add behavior without modifying the wrapped component
- Follow the Open/Closed Principle
- Promote composition
- Execute code before and/or after delegating

---

# Differences

### Decorator

Works with objects.

```
Decorator

↓

Concrete Object
```

The wrapped object implements the same interface.

---

### Middleware

Works with HTTP requests.

```
Middleware

↓

Next Middleware

↓

HTTP Request
```

The focus is request processing rather than extending an object's behavior.

---

# Example Comparison

## Decorator

```csharp
LoggingDecorator

↓

EmailNotifier
```

Purpose:

```
Add logging
```

---

## Middleware

```csharp
LoggingMiddleware

↓

AuthenticationMiddleware

↓

Endpoint
```

Purpose:

```
Process HTTP requests
```

---

# ASP.NET Core Pipeline

```csharp
app.UseAuthentication();

app.UseAuthorization();

app.UseMiddleware<LoggingMiddleware>();

app.MapControllers();
```

Internally:

```
HTTP Request

↓

Authentication

↓

Authorization

↓

Logging

↓

Controller

↓

Response
```

Each middleware calls the next delegate.

---

# Internal Middleware Construction

At startup:

```
Application Builder

↓

Register Middleware

↓

Build Pipeline

↓

RequestDelegate
```

Each middleware receives the next `RequestDelegate`, effectively wrapping it.

Conceptually:

```
LoggingMiddleware
    ↓
AuthenticationMiddleware
    ↓
AuthorizationMiddleware
    ↓
Endpoint
```

This layering is why middleware is often described as a practical application of the Decorator Pattern.

---

# Decorator vs Middleware

| Feature | Decorator | Middleware |
|----------|-----------|------------|
| Pattern Type | Structural | Pipeline built using delegates (Decorator-inspired) |
| Works On | Objects | HTTP Requests |
| Wraps | Another object | Next middleware delegate |
| Primary Goal | Add object behavior | Process request/response pipeline |
| Interface Required | Yes | `RequestDelegate` |
| Typical Scope | Business logic | Web request processing |

---

# Decorator vs Proxy

| Decorator | Proxy |
|-----------|-------|
| Adds new behavior | Controls access to an object |
| Enhances functionality | Protects, delays, or manages access |
| Focuses on extension | Focuses on indirection |

---

# Real-World Analogy

### Decorator

A gift box.

```
Gift

↓

Gift Wrap

↓

Ribbon

↓

Greeting Card
```

Each layer enhances the presentation without changing the gift.

---

### Middleware

Airport security.

```
Passenger

↓

Security Check

↓

Immigration

↓

Boarding

↓

Flight
```

Each stage processes the passenger before passing them to the next stage.

---

# Best Practices

- Keep decorators focused on a single responsibility.
- Chain decorators in a deliberate order when behavior depends on execution sequence.
- Use decorators for cross-cutting concerns such as caching, logging, metrics, or retries.
- Use middleware only for HTTP request/response concerns.
- Avoid placing business logic in middleware that belongs inside services or decorators.

---

# Interview Questions

### Basic

1. What is the Decorator Pattern?
2. Why is it preferred over inheritance in many scenarios?
3. What problem does it solve?

### Intermediate

4. How does the Decorator Pattern support the Open/Closed Principle?
5. What are common uses of decorators in enterprise applications?
6. How is Middleware similar to the Decorator Pattern?

### Advanced

7. How would you implement a caching repository using decorators?
8. Can multiple decorators be combined?
9. Why is middleware considered Decorator-inspired rather than a pure Decorator implementation?
10. Decorator vs Proxy vs Middleware?

---

# Summary

The **Decorator Pattern** is a structural design pattern that adds new behavior to an object dynamically by wrapping it with one or more decorator classes. It enables flexible extension of functionality without modifying existing code and is commonly used for logging, caching, validation, retries, authorization, metrics, and encryption. In **ASP.NET Core**, the middleware pipeline follows a similar wrapping concept: each middleware surrounds the next `RequestDelegate`, performing work before and after invoking it. While middleware is designed specifically for HTTP request processing rather than object enhancement, it shares the same core idea of incrementally extending behavior through composition.