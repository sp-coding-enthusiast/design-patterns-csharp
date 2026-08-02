# 104. CQRS with MediatR

## Introduction

**CQRS (Command Query Responsibility Segregation)** is an architectural pattern that separates **read operations (Queries)** from **write operations (Commands)**.

**MediatR** is one of the most popular libraries in ASP.NET Core for implementing CQRS because it naturally separates each command and query into its own handler.

> **CQRS separates reads from writes, while MediatR acts as the mediator that routes commands and queries to the appropriate handlers.**

---

# What is CQRS?

Traditional applications use the same model for both reading and writing data.

```
Client

↓

Product Service

↓

Repository

↓

Database
```

The same service handles:

- Create
- Update
- Delete
- Get
- Search

As the application grows, the service becomes large and difficult to maintain.

---

# CQRS Solution

CQRS separates responsibilities.

```
              Client

                 │

      ┌──────────┴──────────┐

      ▼                     ▼

 Commands               Queries

      ▼                     ▼

Command Handler      Query Handler

      ▼                     ▼

 Write DB            Read DB

      └──────────┬──────────┘

                 ▼

              Database
```

> In many systems, the read and write sides still use the **same database**. Separate databases are optional and usually introduced only when scaling demands it.

---

# CQRS Components

| Component | Responsibility |
|------------|----------------|
| Command | Changes data |
| Command Handler | Executes business logic |
| Query | Reads data |
| Query Handler | Retrieves data |
| Mediator | Routes requests to handlers |

---

# Traditional ASP.NET Core

```
Controller

↓

Product Service

↓

Repository

↓

Database
```

---

# CQRS with MediatR

```
Controller

↓

IMediator

↓

Command / Query

↓

Handler

↓

Repository

↓

Database
```

The controller depends only on `IMediator`.

---

# Folder Structure

```text
Application
│
├── Commands
│   ├── CreateProductCommand.cs
│   ├── UpdateProductCommand.cs
│   └── DeleteProductCommand.cs
│
├── Queries
│   ├── GetProductQuery.cs
│   ├── GetProductsQuery.cs
│   └── SearchProductsQuery.cs
│
├── Handlers
│   ├── CreateProductHandler.cs
│   ├── UpdateProductHandler.cs
│   ├── DeleteProductHandler.cs
│   ├── GetProductHandler.cs
│   └── GetProductsHandler.cs
│
└── Behaviors
    ├── ValidationBehavior.cs
    ├── LoggingBehavior.cs
    └── PerformanceBehavior.cs
```

---

# Command Example

## CreateProductCommand

```csharp
using MediatR;

public record CreateProductCommand(
    string Name,
    decimal Price)
    : IRequest<int>;
```

The command returns the newly created product ID.

---

# Command Handler

```csharp
using MediatR;

public class CreateProductHandler
    : IRequestHandler<
        CreateProductCommand,
        int>
{
    private readonly IProductRepository
        _repository;

    public CreateProductHandler(
        IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<int> Handle(
        CreateProductCommand request,
        CancellationToken cancellationToken)
    {
        var product = new Product
        {
            Name = request.Name,
            Price = request.Price
        };

        await _repository.AddAsync(product);

        return product.Id;
    }
}
```

---

# Query Example

## GetProductQuery

```csharp
using MediatR;

public record GetProductQuery(
    int Id)
    : IRequest<ProductDto>;
```

---

# Query Handler

```csharp
using MediatR;

public class GetProductHandler
    : IRequestHandler<
        GetProductQuery,
        ProductDto>
{
    private readonly IProductRepository
        _repository;

    public GetProductHandler(
        IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<ProductDto> Handle(
        GetProductQuery request,
        CancellationToken cancellationToken)
    {
        return await _repository
            .GetByIdAsync(request.Id);
    }
}
```

---

# Controller

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController
    : ControllerBase
{
    private readonly IMediator
        _mediator;

    public ProductsController(
        IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpGet("{id}")]
    public async Task<ProductDto> Get(
        int id)
    {
        return await _mediator.Send(
            new GetProductQuery(id));
    }

    [HttpPost]
    public async Task<int> Create(
        CreateProductCommand command)
    {
        return await _mediator.Send(command);
    }
}
```

---

# Internal Workflow

## Query Flow

```
HTTP GET

↓

Controller

↓

Mediator

↓

GetProductQuery

↓

GetProductHandler

↓

Repository

↓

Database

↓

Response
```

---

## Command Flow

```
HTTP POST

↓

Controller

↓

Mediator

↓

CreateProductCommand

↓

CreateProductHandler

↓

Repository

↓

Database

↓

Product ID
```

---

# Pipeline Behaviors

Every request can pass through reusable behaviors.

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

Transaction

↓

Handler

↓

Response
```

This avoids duplicating cross-cutting concerns in every handler.

---

# Read and Write Models

CQRS allows different models for reads and writes.

## Write Model

```text
Product

Id

Name

Price

Description

CreatedDate

ModifiedDate
```

---

## Read Model

```text
ProductDto

Name

Price

Category

AverageRating

Stock
```

The read model is optimized for displaying data instead of persistence.

---

# CQRS in Clean Architecture

```text
Presentation

↓

Application
│
├── Commands
├── Queries
├── Handlers
└── Behaviors

↓

Domain

↓

Infrastructure
```

The **Application** layer contains commands, queries, and handlers, while repositories are implemented in the **Infrastructure** layer.

---

# CQRS with Separate Databases

Large systems may separate read and write databases.

```
               Client

                  │

        ┌─────────┴─────────┐

        ▼                   ▼

     Commands           Queries

        ▼                   ▼

 Write Database      Read Database

        │                   ▲

        └──── Synchronization ────┘
```

Synchronization can be performed using events or message brokers.

---

# Enterprise Example

Imagine an online shopping platform.

### Commands

- Create Order
- Cancel Order
- Update Inventory
- Process Payment

### Queries

- Get Order
- Search Orders
- Get Customer Orders
- Get Dashboard Statistics

Each request has its own dedicated handler.

---

# CQRS + MediatR + Event-Driven Architecture

```
Controller

↓

Mediator

↓

CreateOrderCommand

↓

CreateOrderHandler

↓

Database

↓

OrderCreatedEvent

↓

Email Service

Inventory Service

Analytics Service
```

Here, MediatR handles the command, and after the command succeeds, the application can publish an event for other parts of the system to react.

---

# Benefits

- Clear separation of read and write logic
- Smaller, focused handlers
- Easier testing
- Better scalability
- Supports Clean Architecture
- Integrates naturally with MediatR

---

# Drawbacks

- More files and classes
- Additional abstraction
- Overkill for simple CRUD applications
- Requires good organization

---

# CQRS vs CRUD

| CRUD | CQRS |
|------|-------|
| Single model | Separate command and query models |
| Shared service | Dedicated handlers |
| Simpler | More scalable |
| Best for small apps | Best for medium and large systems |

---

# CQRS vs Repository Pattern

| Repository | CQRS |
|------------|------|
| Data access abstraction | Application architecture pattern |
| Focuses on persistence | Focuses on separating reads and writes |
| Can be used with or without CQRS | Often uses repositories internally |

---

# CQRS vs Event Sourcing

| CQRS | Event Sourcing |
|------|----------------|
| Separates reads and writes | Stores state as a sequence of events |
| Can exist without Event Sourcing | Often paired with CQRS, but independent |
| Current state stored normally | State reconstructed from events |

---

# Best Practices

- Keep one handler per command or query.
- Use commands only for state-changing operations.
- Use queries only for read operations.
- Return DTOs from query handlers instead of domain entities when exposing data externally.
- Keep business rules inside handlers or the domain layer—not in controllers.
- Use pipeline behaviors for validation, logging, transactions, and performance monitoring.

---

# Interview Questions

### Basic

1. What is CQRS?
2. Why separate commands and queries?
3. What is the role of MediatR?

### Intermediate

4. How does `Send()` work in MediatR?
5. Why have separate handlers?
6. What are pipeline behaviors?

### Advanced

7. CQRS vs CRUD?
8. CQRS vs Event Sourcing?
9. How would you scale the read side independently?
10. Design an order management system using CQRS, MediatR, and Clean Architecture.

---

# Summary

**CQRS** separates **commands** (write operations) from **queries** (read operations), resulting in a cleaner and more maintainable architecture. **MediatR** provides an elegant implementation by routing each command or query to its dedicated handler through a single `IMediator` interface. Combined with **Clean Architecture**, **pipeline behaviors**, and optional event-driven messaging, CQRS enables highly scalable, testable, and extensible enterprise ASP.NET Core applications while keeping controllers thin and business logic well organized.