# 95. Structural Design Pattern Coding Challenge

## Objective

This coding challenge is designed to test your understanding of **all seven Structural Design Patterns** in a single enterprise application.

Instead of asking isolated questions, many senior .NET interviews present a real-world scenario where you must decide **which pattern fits which problem**.

---

# Problem Statement

Design an **Enterprise E-Commerce Platform** with the following requirements:

### Functional Requirements

1. Customers can purchase products using multiple payment gateways.
2. The application should support multiple databases.
3. Product categories should be hierarchical.
4. Frequently accessed products should be cached.
5. Every API request should be logged.
6. Only administrators can delete products.
7. Related products should be loaded only when required.
8. The order process should expose a single API to clients.
9. Product images should not load until viewed.
10. The application should support millions of products efficiently.

---

# Step 1: Identify the Pattern

| Requirement | Pattern |
|-------------|----------|
| Payment gateways | Adapter |
| Multiple databases | Bridge |
| Category hierarchy | Composite |
| Logging & Caching | Decorator |
| Single order endpoint | Facade |
| Shared immutable data | Flyweight |
| Authorization/Lazy Loading | Proxy |

---

# Overall Architecture

```text
                   Client
                      │
                      ▼
               Order Facade
                      │
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
 Inventory        Payment         Notification
  Service          Adapter            Service
                      │
          ┌───────────┼────────────┐
          ▼           ▼            ▼
       Stripe      PayPal      Razorpay

Repository
     │
     ▼
Logging Decorator
     │
Caching Decorator
     │
Authorization Proxy
     │
Repository
     │
     ▼
Database Bridge
     │
 ┌───┴───────────────┐
 ▼                   ▼
SQL Server      PostgreSQL

Product Categories
      │
      ▼
Composite Tree

Reference Data
      │
      ▼
Flyweight Cache
```

---

# Challenge 1 – Adapter Pattern

## Problem

Different payment providers expose different APIs.

### Stripe

```csharp
ChargeAsync()
```

### PayPal

```csharp
CreatePayment()
```

### Razorpay

```csharp
CaptureOrder()
```

---

### Your Task

Create

```csharp
IPaymentGateway
```

that hides all provider differences.

Expected API:

```csharp
await gateway.ProcessPayment(1000);
```

---

# Challenge 2 – Bridge Pattern

Support

```
SQL Server

PostgreSQL

MongoDB
```

with

```
Customer Repository

Product Repository

Order Repository
```

---

### Wrong Design

```
CustomerSqlRepository

CustomerMongoRepository

CustomerPostgresRepository

OrderSqlRepository

...
```

---

### Your Task

Create

```
Repository

↓

Database Provider
```

using Bridge.

---

# Challenge 3 – Composite Pattern

Represent

```
Electronics

├── Laptop

├── Mobile

└── Accessories

      ├── Mouse

      └── Keyboard
```

---

### Your Task

Implement

```csharp
Category

Add()

Remove()

Display()
```

so that both **categories** and **individual products** can be treated uniformly.

---

# Challenge 4 – Decorator Pattern

Repositories must support:

- Logging
- Validation
- Caching

without modifying repository code.

---

### Expected Chain

```
Logging

↓

Validation

↓

Caching

↓

Repository
```

---

### Your Task

Implement

```csharp
IRepository
```

and create decorators that can be composed dynamically.

---

# Challenge 5 – Facade Pattern

Clients should call

```text
POST /orders
```

instead of invoking multiple services.

---

### Internally

```
Inventory

↓

Payment

↓

Shipping

↓

Notification
```

---

### Your Task

Create

```csharp
OrderFacade
```

with

```csharp
PlaceOrder()
```

that orchestrates the complete workflow.

---

# Challenge 6 – Flyweight Pattern

Suppose there are

```
10 Million Products
```

Each stores

```
Country

Currency

Tax Rule

Brand
```

Thousands of products share the same values.

---

### Wrong Design

```
Product

↓

Country Object

↓

Currency Object

↓

Brand Object
```

for every product.

---

### Your Task

Implement a

```text
ReferenceDataFactory
```

that shares immutable objects.

---

# Challenge 7 – Proxy Pattern

Only administrators may delete products.

---

### Workflow

```
User

↓

Proxy

↓

Repository
```

---

### Your Task

Implement

```
Protection Proxy
```

that checks user roles before calling the repository.

---

# Bonus Challenge 1 – Virtual Proxy

Products contain

```
100 MB Images
```

Images should load only when viewed.

---

### Expected Workflow

```
Product

↓

Image Proxy

↓

Load Image

↓

Display
```

---

# Bonus Challenge 2 – Smart Proxy

Measure execution time.

Expected output

```
Start

↓

Repository

↓

Elapsed 34 ms
```

without modifying repository code.

---

# Bonus Challenge 3 – Remote Proxy

The inventory service runs on another server.

Implement

```
IInventoryService

↓

HttpClient

↓

Inventory API
```

so the client treats the remote service like a local object.

---

# Suggested Project Structure

```text
ECommerceApp
│
├── Adapters
│   ├── StripeAdapter.cs
│   ├── PayPalAdapter.cs
│   └── RazorpayAdapter.cs
│
├── Bridges
│   ├── Repository.cs
│   ├── SqlProvider.cs
│   ├── PostgreSqlProvider.cs
│   └── MongoProvider.cs
│
├── Composites
│   ├── Category.cs
│   ├── Product.cs
│   └── INode.cs
│
├── Decorators
│   ├── LoggingDecorator.cs
│   ├── ValidationDecorator.cs
│   └── CachingDecorator.cs
│
├── Facades
│   └── OrderFacade.cs
│
├── Flyweights
│   ├── CountryFactory.cs
│   ├── CurrencyFactory.cs
│   └── BrandFactory.cs
│
├── Proxies
│   ├── AuthorizationProxy.cs
│   ├── ImageProxy.cs
│   ├── LoggingProxy.cs
│   └── InventoryProxy.cs
│
└── Program.cs
```

---

# Pattern Mapping

| Module | Pattern Used | Reason |
|----------|--------------|--------|
| Payment Integration | Adapter | Normalize third-party APIs |
| Database Layer | Bridge | Separate repositories from database implementations |
| Product Categories | Composite | Represent hierarchical structures |
| Repository Pipeline | Decorator | Add logging, validation, and caching dynamically |
| Order Processing | Facade | Provide a simplified orchestration layer |
| Reference Data Cache | Flyweight | Share immutable objects to reduce memory |
| Authorization | Protection Proxy | Restrict access |
| Product Images | Virtual Proxy | Lazy load expensive resources |
| Remote Inventory | Remote Proxy | Hide network communication |
| Performance Monitoring | Smart Proxy | Measure execution time transparently |

---

# Expected Design Principles

Your implementation should demonstrate:

- **SRP** – Each class has one responsibility.
- **OCP** – Add new gateways, databases, or decorators without changing existing code.
- **LSP** – Decorators and proxies should be substitutable for the original interfaces.
- **ISP** – Keep interfaces focused and small.
- **DIP** – Depend on abstractions (`IPaymentGateway`, `IRepository`, `IDatabaseProvider`) instead of concrete classes.

---

# Interview Follow-up Questions

### Basic

1. Which structural pattern would you use for payment gateway integration?
2. Why is Composite suitable for product categories?
3. Why is a Facade useful in an order-processing workflow?

### Intermediate

4. How do Decorator and Proxy differ in this application?
5. Why is Bridge better than inheritance for supporting multiple databases?
6. What problem does Flyweight solve?

### Advanced

7. If a new payment provider is added tomorrow, which classes change?
8. How would you prevent cache stampede in the caching decorator?
9. How would you avoid the N+1 query problem when using EF Core lazy-loading proxies?
10. How would you extend this architecture to support event-driven microservices?

---

# What Interviewers Evaluate

During this challenge, interviewers typically look for:

- Correct identification of the appropriate pattern for each requirement.
- Proper use of interfaces and dependency injection.
- Separation of concerns and adherence to SOLID principles.
- Extensibility without modifying existing code.
- Ability to justify design decisions and discuss trade-offs.
- Clean, maintainable, and testable architecture.

---

# Summary

This coding challenge combines all **seven Structural Design Patterns** into a realistic enterprise e-commerce application. **Adapter** standardizes payment gateways, **Bridge** decouples repositories from database providers, **Composite** models hierarchical product categories, **Decorator** adds logging, validation, and caching, **Facade** orchestrates order processing, **Flyweight** minimizes memory usage by sharing immutable reference data, and **Proxy** handles authorization, lazy loading, remote communication, and monitoring. Successfully designing this system demonstrates a strong understanding of structural patterns, SOLID principles, and enterprise-level .NET architecture.