# 90. Structural Design Pattern Comparison

## Introduction

Structural Design Patterns describe **how classes and objects are composed** to form larger, flexible, and maintainable systems.

Unlike Creational Patterns (which focus on object creation) or Behavioral Patterns (which focus on communication), Structural Patterns focus on **object relationships**.

The Gang of Four (GoF) defines **7 Structural Patterns**:

1. Adapter
2. Bridge
3. Composite
4. Decorator
5. Facade
6. Flyweight
7. Proxy

---

# Structural Pattern Overview

| Pattern | Intent | Real-World Example |
|----------|---------|-------------------|
| Adapter | Convert one interface into another | Payment Gateway Adapter |
| Bridge | Separate abstraction from implementation | Database Providers |
| Composite | Represent tree structures | UI Components |
| Decorator | Add behavior dynamically | Logging, Caching |
| Facade | Simplify complex systems | Order Processing |
| Flyweight | Share objects to save memory | String Interning |
| Proxy | Control access to objects | EF Core Lazy Loading |

---

# Classification

```
                 Structural Patterns

                        │

 ┌──────────┬──────────┬──────────┬──────────┐

 Adapter   Bridge   Composite  Decorator

 └──────────┬──────────┬──────────┬──────────┘

         Facade    Flyweight    Proxy
```

---

# Primary Purpose

## Adapter

```
Problem:

Incompatible Interfaces
```

Solution:

```
Convert Interface
```

---

## Bridge

```
Problem:

Class Explosion

(Product × Database)
```

Solution:

```
Separate Hierarchies
```

---

## Composite

```
Problem:

Tree Structures
```

Solution:

```
Treat Parent and Child Equally
```

---

## Decorator

```
Problem:

Need More Functionality
```

Solution:

```
Wrap Object
```

---

## Facade

```
Problem:

Complex Subsystem
```

Solution:

```
Simplified Interface
```

---

## Flyweight

```
Problem:

Too Much Memory
```

Solution:

```
Share Objects
```

---

## Proxy

```
Problem:

Need Access Control
```

Solution:

```
Control Access
```

---

# Visual Comparison

## Adapter

```
Client

↓

Adapter

↓

Third-party API
```

---

## Bridge

```
Notification

↓

Email

SMS

Push
```

---

## Composite

```
Window

↓

Panel

↓

Button
```

---

## Decorator

```
Logging

↓

Caching

↓

Repository
```

---

## Facade

```
Client

↓

OrderFacade

↓

Inventory

↓

Payment

↓

Shipping
```

---

## Flyweight

```
Character Factory

↓

Shared Character

↓

Thousands of References
```

---

## Proxy

```
Client

↓

Proxy

↓

Real Object
```

---

# Enterprise Examples

| Pattern | Enterprise Scenario |
|----------|---------------------|
| Adapter | Stripe, PayPal, Razorpay integration |
| Bridge | SQL Server, MongoDB, PostgreSQL providers |
| Composite | Organization hierarchy, UI components |
| Decorator | Logging, Validation, Retry, Caching |
| Facade | Order Processing, Travel Booking |
| Flyweight | Shared Country, Currency, Permission objects |
| Proxy | EF Core Lazy Loading, Authorization, Remote APIs |

---

# Problem Solved

| Problem | Pattern |
|----------|----------|
| Legacy API | Adapter |
| Multiple implementations | Bridge |
| Tree hierarchy | Composite |
| Cross-cutting concerns | Decorator |
| Too many subsystem calls | Facade |
| Memory optimization | Flyweight |
| Security/Lazy loading | Proxy |

---

# Relationship Between Patterns

```
                    Structural Patterns

                             │

          ┌──────────────────┴──────────────────┐

          │                                     │

     Add Functionality                    Organize Structure

          │                                     │

     Decorator                          Composite

          │

     Control Access

          │

        Proxy

------------------------------------------------------

Adapter

↓

Compatibility

------------------------------------------------------

Bridge

↓

Flexibility

------------------------------------------------------

Facade

↓

Simplification

------------------------------------------------------

Flyweight

↓

Memory Optimization
```

---

# Internal Comparison

| Feature | Adapter | Bridge | Composite | Decorator | Facade | Flyweight | Proxy |
|----------|----------|---------|-----------|-----------|---------|------------|--------|
| Pattern Type | Structural | Structural | Structural | Structural | Structural | Structural | Structural |
| Primary Goal | Compatibility | Decoupling | Tree Hierarchy | Extend Behavior | Simplify System | Save Memory | Control Access |
| Uses Composition | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Changes Interface | Yes | No | No | No | Usually Yes | No | No |
| Wraps Another Object | Yes | No | No | Yes | Yes | Yes | Yes |
| Supports OCP | Yes | Yes | Yes | Yes | Yes | Yes | Yes |

---

# Complexity Comparison

| Pattern | Difficulty |
|----------|------------|
| Adapter | ⭐⭐ |
| Facade | ⭐⭐ |
| Proxy | ⭐⭐⭐ |
| Decorator | ⭐⭐⭐ |
| Composite | ⭐⭐⭐ |
| Flyweight | ⭐⭐⭐⭐ |
| Bridge | ⭐⭐⭐⭐ |

---

# Performance Impact

| Pattern | Performance Impact |
|----------|--------------------|
| Adapter | Negligible |
| Bridge | Negligible |
| Composite | Small recursive overhead |
| Decorator | Small call-chain overhead |
| Facade | Negligible |
| Flyweight | Improves memory usage significantly |
| Proxy | Depends on implementation (lazy loading, network calls, etc.) |

---

# SOLID Principles Supported

| Pattern | SOLID Principles |
|----------|------------------|
| Adapter | OCP, DIP |
| Bridge | OCP, DIP |
| Composite | OCP, LSP |
| Decorator | SRP, OCP |
| Facade | SRP, DIP |
| Flyweight | SRP |
| Proxy | SRP, OCP |

---

# ASP.NET Core Examples

| Pattern | ASP.NET Core Example |
|----------|----------------------|
| Adapter | Payment gateway integration |
| Bridge | Database provider abstraction |
| Composite | Blazor component tree, Menu hierarchy |
| Decorator | Repository logging and caching |
| Facade | Order orchestration service |
| Flyweight | Shared reference data cache |
| Proxy | EF Core lazy-loading proxies |

---

# When to Use Which Pattern

## Adapter

Use when:

- Integrating third-party libraries
- Migrating legacy systems
- Standardizing different APIs

---

## Bridge

Use when:

- Two dimensions vary independently
- Supporting multiple providers
- Avoiding class explosion

---

## Composite

Use when:

- Building tree structures
- UI hierarchies
- File systems
- Organization charts

---

## Decorator

Use when:

- Adding logging
- Adding caching
- Adding validation
- Adding retry logic
- Adding metrics

---

## Facade

Use when:

- Multiple services need orchestration
- Simplifying complex APIs
- Exposing a clean entry point

---

## Flyweight

Use when:

- Millions of similar objects exist
- Memory consumption is high
- Shared immutable state is possible

---

## Proxy

Use when:

- Lazy loading
- Authorization
- Remote communication
- Access logging
- Performance monitoring

---

# Decision Tree

```
Need to integrate different APIs?

↓

Adapter

----------------------------

Need multiple implementations?

↓

Bridge

----------------------------

Need hierarchical objects?

↓

Composite

----------------------------

Need additional behavior?

↓

Decorator

----------------------------

Need one simple interface?

↓

Facade

----------------------------

Need memory optimization?

↓

Flyweight

----------------------------

Need controlled access?

↓

Proxy
```

---

# Common Interview Scenarios

| Interview Requirement | Best Pattern |
|-----------------------|--------------|
| Integrate Stripe and PayPal | Adapter |
| Support SQL Server and MongoDB | Bridge |
| Build a file explorer | Composite |
| Add logging to repositories | Decorator |
| Simplify order processing | Facade |
| Optimize a text editor | Flyweight |
| Secure document access | Protection Proxy |
| Load images on demand | Virtual Proxy |
| Call remote microservices | Remote Proxy |

---

# Pattern Combination in Enterprise Applications

A modern e-commerce application often uses multiple structural patterns together:

```
                    Client
                       │
                       ▼
                 Order Facade
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
 Payment Adapter   Inventory Service  Notification Service
        │
        ▼
 Stripe / PayPal / Razorpay

Repository
    │
    ▼
Logging Decorator
    │
Caching Decorator
    │
Authorization Proxy
    │
Customer Repository
    │
    ▼
Database Provider (Bridge)
    │
 ┌──┴───────────────┐
 ▼                  ▼
SQL Server      PostgreSQL

UI
│
▼
Composite Tree
(Page → Layout → Panel → Button)

Reference Data
│
▼
Flyweight Cache
(Country, Currency, Permissions)
```

This demonstrates how structural patterns complement each other rather than compete.

---

# Best Practices

- Choose the pattern based on the problem, not because it is familiar.
- Prefer composition over inheritance whenever possible.
- Keep each pattern focused on a single responsibility.
- Don't combine multiple patterns unless they solve distinct problems.
- Use Dependency Injection with Adapter, Bridge, Decorator, Facade, and Proxy to improve flexibility and testability.

---

# Interview Questions

### Basic

1. What are Structural Design Patterns?
2. Name all seven Structural Patterns.
3. Which pattern is used for third-party integration?

### Intermediate

4. Bridge vs Adapter?
5. Decorator vs Proxy?
6. Composite vs Facade?
7. Flyweight vs Singleton?

### Advanced

8. Design an enterprise order management system using multiple structural patterns.
9. Which structural patterns are commonly used in ASP.NET Core?
10. Can multiple structural patterns be combined in a single application? Explain with an example.

---

# Summary

Structural Design Patterns define how objects are composed to build flexible, reusable, and maintainable software. **Adapter** solves compatibility problems, **Bridge** separates abstractions from implementations, **Composite** models hierarchical structures, **Decorator** adds responsibilities dynamically, **Facade** hides subsystem complexity, **Flyweight** optimizes memory by sharing immutable state, and **Proxy** controls access to objects. In enterprise .NET applications, these patterns are frequently combined to solve real-world problems such as integrating third-party services, supporting multiple database providers, building UI hierarchies, implementing logging and caching, orchestrating microservices, optimizing memory usage, and enabling lazy loading and authorization.