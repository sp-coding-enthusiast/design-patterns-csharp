# 103. Mediator Pattern

## Introduction

The **Mediator Pattern** is a **Behavioral Design Pattern** that reduces direct communication between objects by introducing a **Mediator** object.

Instead of objects talking to each other directly, they communicate through the mediator.

> **Mediator centralizes communication between objects, reducing coupling and making interactions easier to manage.**

---

# Definition

> **Define an object that encapsulates how a set of objects interact. The Mediator promotes loose coupling by preventing objects from referring to each other explicitly.**

---

# Why Do We Need Mediator?

Imagine an e-commerce system.

Without Mediator:

```
Order Service

↓

Inventory Service

↓

Payment Service

↓

Shipping Service

↓

Notification Service
```

Every service knows about the others.

If Payment changes, multiple services may need updates.

---

# Problems

- Tight coupling
- Complex dependencies
- Difficult testing
- Hard to extend
- Circular dependencies

---

# Solution

Introduce a Mediator.

```
                Mediator

      ↙      ↓      ↓      ↘

 Order   Inventory  Payment  Shipping

                 ↓

          Notification
```

Now services communicate only with the mediator.

---

# Real-Life Example

Think of an **Air Traffic Controller**.

Without a controller:

```
Plane ↔ Plane ↔ Plane ↔ Plane
```

Every aircraft communicates with every other aircraft.

With a controller:

```
Plane

↓

Air Traffic Controller

↓

Other Planes
```

Each plane communicates with one central authority.

---

# Components

| Component | Responsibility |
|------------|----------------|
| Mediator | Defines communication contract |
| Concrete Mediator | Coordinates interactions |
| Colleague | Uses the mediator |
| Concrete Colleague | Performs business logic |

---

# UML Structure

```
                 IMediator

                     ▲

                     │

             OrderMediator

          ↙      ↓       ↘

 Inventory  Payment  Shipping
```

---

# Example

## Mediator Interface

```csharp
public interface IMediator
{
    void Send(string message);
}
```

---

## Concrete Mediator

```csharp
public class ChatMediator
    : IMediator
{
    public void Send(string message)
    {
        Console.WriteLine(
            $"Broadcast: {message}");
    }
}
```

---

## Colleague

```csharp
public class User
{
    private readonly IMediator
        _mediator;

    public User(IMediator mediator)
    {
        _mediator = mediator;
    }

    public void SendMessage(
        string message)
    {
        _mediator.Send(message);
    }
}
```

---

## Client

```csharp
var mediator =
    new ChatMediator();

var user =
    new User(mediator);

user.SendMessage(
    "Hello Everyone");
```

---

# Enterprise Example

Without Mediator

```text
Order

↓

Inventory

↓

Payment

↓

Shipping

↓

Notification
```

With Mediator

```text
Order

↓

Mediator

↓

Inventory

↓

Payment

↓

Shipping

↓

Notification
```

The order service only communicates with the mediator.

---

# Internal Workflow

```
Client

↓

Mediator

↓

Handler

↓

Response
```

---

# Advantages

- Loose coupling
- Easier maintenance
- Centralized workflow
- Simpler testing
- Follows the Single Responsibility Principle

---

# Disadvantages

- The mediator can become too large ("God Object") if it contains excessive business logic.
- Adds an extra abstraction layer.
- Overkill for very small applications.

---

# Common Uses

- Chat applications
- UI components
- Workflow engines
- Enterprise applications
- CQRS
- ASP.NET Core applications

---

# Observer vs Mediator

| Observer | Mediator |
|-----------|----------|
| Broadcast notifications | Coordinates communication |
| One publisher, many subscribers | Many colleagues communicate through one mediator |
| Event-driven | Request/response or command coordination |

---

# Strategy vs Mediator

| Strategy | Mediator |
|-----------|----------|
| Chooses an algorithm | Coordinates object interaction |
| One active implementation | Multiple collaborating objects |

---

# Facade vs Mediator

| Facade | Mediator |
|---------|----------|
| Simplifies a subsystem | Coordinates communication |
| Clients call the facade | Colleagues communicate through the mediator |

---

# 104. MediatR

## What is MediatR?

**MediatR** is a popular .NET library that implements the **Mediator Pattern**.

Instead of controllers directly calling services, repositories, or other components, they send **requests** or **notifications** to a mediator.

The mediator locates the appropriate handler and executes it.

---

# Traditional ASP.NET Core

```
Controller

↓

Service

↓

Repository

↓

Database
```

The controller depends on the service.

---

# Using MediatR

```
Controller

↓

IMediator

↓

Handler

↓

Repository

↓

Database
```

The controller depends only on `IMediator`.

---

# Installation

```bash
dotnet add package MediatR
```

For ASP.NET Core, register MediatR during startup (the exact registration API depends on the MediatR version in use).

---

# Request/Response

## Request

```csharp
public record GetProductQuery(
    int Id)
    : IRequest<Product>;
```

---

## Handler

```csharp
public class GetProductHandler
    : IRequestHandler<
        GetProductQuery,
        Product>
{
    public async Task<Product>
        Handle(
            GetProductQuery request,
            CancellationToken cancellationToken)
    {
        return await repository.GetByIdAsync(
            request.Id);
    }
}
```

---

## Controller

```csharp
public class ProductController
{
    private readonly IMediator
        _mediator;

    public ProductController(
        IMediator mediator)
    {
        _mediator = mediator;
    }

    public async Task<Product> Get(
        int id)
    {
        return await _mediator.Send(
            new GetProductQuery(id));
    }
}
```

---

# Internal Workflow

```
Controller

↓

Mediator

↓

Request

↓

Handler

↓

Repository

↓

Database
```

---

# Commands and Queries

### Command

Changes data.

```
CreateOrderCommand

↓

Handler

↓

Database
```

---

### Query

Reads data.

```
GetOrderQuery

↓

Handler

↓

Database
```

This aligns well with the **CQRS (Command Query Responsibility Segregation)** pattern.

---

# Notifications

MediatR also supports publish/subscribe.

---

## Notification

```csharp
public class OrderCreatedNotification
    : INotification
{
    public int OrderId
    {
        get;
        set;
    }
}
```

---

## Handler 1

```csharp
public class EmailHandler
    : INotificationHandler<
        OrderCreatedNotification>
{
}
```

---

## Handler 2

```csharp
public class InventoryHandler
    : INotificationHandler<
        OrderCreatedNotification>
{
}
```

---

## Publish

```csharp
await _mediator.Publish(
    new OrderCreatedNotification
    {
        OrderId = 10
    });
```

Execution

```
Mediator

↓

Email

Inventory

Analytics
```

All registered notification handlers are invoked.

---

# Pipeline Behaviors

One of MediatR's most powerful features is **Pipeline Behaviors**, which work similarly to middleware.

```
Request

↓

Logging

↓

Validation

↓

Authorization

↓

Performance

↓

Handler
```

Example use cases:

- Logging
- Validation
- Performance measurement
- Transactions
- Exception handling

---

# MediatR Architecture

```
Controller

↓

Mediator

↓

Pipeline Behaviors

↓

Handler

↓

Repository

↓

Database
```

---

# ASP.NET Core Example

```text
POST /orders

↓

CreateOrderCommand

↓

Mediator

↓

Validation

↓

Logging

↓

CreateOrderHandler

↓

Repository
```

The controller has no knowledge of repositories or business logic.

---

# Benefits

- Loose coupling
- Better separation of concerns
- Easier unit testing
- Supports CQRS
- Centralized cross-cutting concerns via pipeline behaviors

---

# Drawbacks

- Adds abstraction and indirection.
- Can make debugging request flow more challenging.
- Not necessary for very small applications.

---

# When to Use MediatR

Use MediatR when:

- Building medium or large ASP.NET Core applications.
- Implementing CQRS.
- You need request/response and notification patterns.
- You want centralized logging, validation, and other cross-cutting concerns.

Avoid it when:

- Building very small CRUD applications with minimal business logic.
- The added abstraction outweighs the benefits.

---

# MediatR vs Service Layer

| Service Layer | MediatR |
|---------------|----------|
| Controller calls services directly | Controller sends requests to mediator |
| Multiple service dependencies | Single `IMediator` dependency |
| Cross-cutting logic spread across services | Centralized using pipeline behaviors |
| Simpler for small apps | Better scalability for complex apps |

---

# Interview Questions

### Basic

1. What is the Mediator Pattern?
2. What problem does it solve?
3. What is MediatR?

### Intermediate

4. How does MediatR reduce coupling?
5. What is the difference between `Send()` and `Publish()`?
6. What are pipeline behaviors?

### Advanced

7. How does MediatR support CQRS?
8. MediatR vs Observer?
9. MediatR vs Service Layer?
10. When would you avoid using MediatR?

---

# Summary

The **Mediator Pattern** centralizes communication between collaborating objects, reducing direct dependencies and improving maintainability. In the .NET ecosystem, **MediatR** is a widely used implementation of this pattern. It enables controllers to send **commands**, **queries**, and **notifications** through a mediator instead of calling services directly. Combined with **pipeline behaviors**, MediatR provides a clean architecture for handling validation, logging, authorization, transactions, and other cross-cutting concerns, making it especially valuable in enterprise ASP.NET Core applications that follow **CQRS** and **Clean Architecture** principles.