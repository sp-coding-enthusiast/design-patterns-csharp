# 90. Structural Design Pattern Comparison

## Introduction

Structural Design Patterns focus on **how classes and objects are composed** to create larger, flexible, and maintainable systems.

The GoF Structural Patterns are:

1. Adapter
2. Bridge
3. Composite
4. Decorator
5. Facade
6. Flyweight
7. Proxy

Each pattern solves a different architectural problem.

---

# Structural Pattern Overview

| Pattern | Primary Goal | Typical Use Case |
|----------|--------------|------------------|
| Adapter | Convert incompatible interfaces | Third-party SDK integration |
| Bridge | Separate abstraction from implementation | Multiple providers (storage, database, notifications) |
| Composite | Represent tree structures | UI controls, menus, file systems |
| Decorator | Add behavior dynamically | Logging, caching, validation |
| Facade | Simplify a complex subsystem | Order processing, microservices |
| Flyweight | Reduce memory usage | Icons, text editors, game objects |
| Proxy | Control access | Lazy loading, security, remote APIs |

---

# Decision Tree

```
Need to integrate incompatible APIs?

↓

Adapter

-------------------------

Need independent hierarchies?

↓

Bridge

-------------------------

Need tree structures?

↓

Composite

-------------------------

Need additional behavior?

↓

Decorator

-------------------------

Need to simplify complexity?

↓

Facade

-------------------------

Need memory optimization?

↓

Flyweight

-------------------------

Need access control?

↓

Proxy
```

---

# Real-World Examples

| Pattern | Example |
|----------|----------|
| Adapter | Stripe SDK adapted to your payment interface |
| Bridge | Notification → Email/SMS/Push |
| Composite | Windows, Panels, Buttons |
| Decorator | Logging repository |
| Facade | Order Processing Service |
| Flyweight | Shared country objects |
| Proxy | EF Core Lazy Loading |

---

# Performance Comparison

| Pattern | Runtime Cost | Memory Cost |
|----------|--------------|-------------|
| Adapter | Very Low | Very Low |
| Bridge | Very Low | Low |
| Composite | Low | Medium |
| Decorator | Low | Medium |
| Facade | Negligible | Negligible |
| Flyweight | Very Low | Saves Memory |
| Proxy | Low | Low |

---

# SOLID Principles Supported

| Pattern | SOLID Principles |
|----------|------------------|
| Adapter | OCP, DIP |
| Bridge | OCP, DIP |
| Composite | OCP, LSP |
| Decorator | OCP, SRP |
| Facade | SRP, DIP |
| Flyweight | SRP |
| Proxy | SRP, OCP |

---

# Interview Tip

A common interview question is:

> **Which structural pattern should you choose?**

| Requirement | Pattern |
|-------------|----------|
| Integrate legacy API | Adapter |
| Add logging | Decorator |
| Secure access | Proxy |
| Build UI hierarchy | Composite |
| Hide complexity | Facade |
| Support multiple providers | Bridge |
| Reduce memory | Flyweight |

---

# 91. Facade Pattern in Microservices

## Problem

An e-commerce application has multiple microservices.

```
Order Service

Inventory Service

Payment Service

Shipping Service

Notification Service
```

A client must call each service individually.

```
Client

↓

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

This creates tight coupling.

---

# Solution

Introduce a Facade (Aggregator) service.

```
Client

↓

Order Facade

↓

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

The client calls only one endpoint.

---

# Example

```csharp
public class OrderFacade
{
    public async Task PlaceOrder()
    {
        await _inventory.Reserve();

        await _payment.Pay();

        await _shipping.Schedule();

        await _notification.Send();
    }
}
```

---

# Benefits

- Simplifies client interactions
- Reduces network calls
- Centralizes orchestration
- Shields clients from internal service changes

---

# Typical Architecture

```
Mobile App

↓

API Gateway

↓

Order Facade

↓

Microservices
```

> **Note:** An API Gateway routes requests, while a Facade orchestrates business workflows. In many systems, they work together.

---

# 92. Bridge Pattern in Database Providers

## Problem

An application supports multiple databases.

```
SQL Server

PostgreSQL

MongoDB

MySQL
```

And multiple repository types.

```
Customer Repository

Product Repository

Order Repository
```

Without Bridge:

```
CustomerSqlRepository

CustomerMongoRepository

CustomerPostgresRepository

ProductSqlRepository

ProductMongoRepository

...
```

Class explosion occurs.

---

# Solution

Separate repository abstraction from database implementation.

---

## Implementor

```csharp
public interface IDatabaseProvider
{
    Task ExecuteAsync(string query);
}
```

---

## Implementations

```csharp
public class SqlProvider
    : IDatabaseProvider
{
}

public class MongoProvider
    : IDatabaseProvider
{
}
```

---

## Abstraction

```csharp
public abstract class Repository
{
    protected readonly IDatabaseProvider
        Provider;

    protected Repository(
        IDatabaseProvider provider)
    {
        Provider = provider;
    }
}
```

---

## Concrete Repository

```csharp
public class CustomerRepository
    : Repository
{
    public CustomerRepository(
        IDatabaseProvider provider)
        : base(provider)
    {
    }
}
```

---

# Workflow

```
Customer Repository

↓

SQL Provider
```

or

```
Customer Repository

↓

Mongo Provider
```

Both hierarchies evolve independently.

---

# 93. Adapter Pattern in Payment Gateway

## Problem

Your application expects:

```csharp
ProcessPayment()
```

Stripe SDK provides:

```csharp
ChargeAsync()
```

PayPal SDK provides:

```csharp
CreatePayment()
```

Razorpay SDK provides:

```csharp
CaptureOrder()
```

Each gateway has a different interface.

---

# Solution

Define a common interface.

```csharp
public interface IPaymentGateway
{
    Task ProcessPayment(
        decimal amount);
}
```

---

# Stripe Adapter

```csharp
public class StripeAdapter
    : IPaymentGateway
{
    private readonly StripeClient
        _client;

    public async Task ProcessPayment(
        decimal amount)
    {
        await _client.ChargeAsync(amount);
    }
}
```

---

# PayPal Adapter

```csharp
public class PayPalAdapter
    : IPaymentGateway
{
    private readonly PayPalClient
        _client;

    public async Task ProcessPayment(
        decimal amount)
    {
        await _client.CreatePayment(amount);
    }
}
```

---

# Client

```csharp
IPaymentGateway gateway =
    new StripeAdapter();

await gateway.ProcessPayment(1000);
```

The client is independent of the underlying payment provider.

---

# Benefits

- Replace payment providers easily
- Isolate SDK changes
- Simplify testing using mocks
- Maintain a consistent API across gateways

---

# 94. Proxy Pattern Interview Scenario

## Scenario

An interviewer asks:

> **Design a document management system where only administrators can access confidential files, documents are loaded only when opened, and every access must be logged.**

---

# Solution

Use multiple Proxy types.

```
Client

↓

Protection Proxy

↓

Virtual Proxy

↓

Smart Proxy

↓

Real Document
```

---

# Responsibilities

### Protection Proxy

```
Check Role

↓

Admin?

↓

Yes

↓

Continue
```

---

### Virtual Proxy

```
Document Loaded?

↓

No

↓

Load From Disk
```

---

### Smart Proxy

```
Log Access

↓

Measure Time

↓

Return Document
```

---

# Benefits

- Security
- Lazy loading
- Auditing
- Better performance

---

# Interview Discussion

The interviewer may ask:

**Why not use one proxy?**

Answer:

Each proxy has a **single responsibility**, making the design easier to extend and test while following the **Single Responsibility Principle (SRP)**.

---

# 95. Structural Pattern Coding Challenge

## Problem Statement

Design an **Enterprise E-Commerce Platform**.

Requirements:

1. Support Stripe, PayPal, and Razorpay.
2. Support SQL Server and MongoDB.
3. Cache frequently accessed products.
4. Log every request.
5. Secure admin operations.
6. Display nested product categories.
7. Simplify order processing.

---

# Suggested Solution

## Adapter

```
Stripe Adapter

↓

IPaymentGateway
```

Purpose:

Support multiple payment gateways.

---

## Bridge

```
Repository

↓

SQL Provider

Mongo Provider
```

Purpose:

Support multiple databases.

---

## Decorator

```
Logging

↓

Caching

↓

Repository
```

Purpose:

Add logging and caching without modifying repositories.

---

## Proxy

```
Authorization Proxy

↓

Repository
```

Purpose:

Protect sensitive operations.

---

## Composite

```
Electronics

├── Laptop

├── Mobile

└── Accessories

      ├── Mouse

      └── Keyboard
```

Purpose:

Represent hierarchical product categories.

---

## Facade

```
Order Facade

↓

Inventory

↓

Payment

↓

Shipping

↓

Notification
```

Purpose:

Provide a single entry point for order placement.

---

# Complete Architecture

```text
                Client
                   │
                   ▼
             Order Facade
                   │
      ┌────────────┼────────────┐
      ▼            ▼            ▼
 Payment Adapter  Repository   Notification
      │               │
      ▼               ▼
 Stripe/PayPal   Logging Decorator
                 ▼
             Caching Decorator
                 ▼
           Authorization Proxy
                 ▼
         Customer Repository
                 ▼
      SQL Provider / Mongo Provider
```

Product categories:

```text
Electronics
├── Laptop
├── Mobile
└── Accessories
    ├── Mouse
    └── Keyboard
```

---

# Pattern Selection Cheat Sheet

| Requirement | Pattern |
|-------------|----------|
| Integrate third-party SDK | Adapter |
| Support multiple implementations | Bridge |
| Build hierarchical structures | Composite |
| Add logging, caching, validation | Decorator |
| Hide subsystem complexity | Facade |
| Optimize memory | Flyweight |
| Control access or lazy loading | Proxy |

---

# Interview Questions

### Basic

1. List all seven Structural Design Patterns.
2. Which pattern is used for third-party integrations?
3. Which pattern simplifies a complex subsystem?

### Intermediate

4. Bridge vs Adapter?
5. Decorator vs Proxy?
6. Why is Composite ideal for UI frameworks?

### Advanced

7. Design a payment platform using Adapter and Bridge.
8. Design a secure document system using multiple Proxy types.
9. How would you architect a microservice-based order system using Facade?
10. Explain how multiple structural patterns work together in a real enterprise application.

---

# Summary

Structural Design Patterns help organize classes and objects to build flexible, maintainable, and scalable systems. **Adapter** integrates incompatible interfaces, **Bridge** separates abstractions from implementations, **Composite** models hierarchical structures, **Decorator** adds responsibilities dynamically, **Facade** simplifies complex subsystems, **Flyweight** minimizes memory usage by sharing state, and **Proxy** controls access to objects. In enterprise .NET applications, these patterns are often combined—for example, an order-processing system may use **Facade** for orchestration, **Adapter** for payment gateways, **Bridge** for database providers, **Decorator** for logging and caching, **Proxy** for authorization, and **Composite** for product category hierarchies.