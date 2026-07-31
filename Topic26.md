# 63. Facade Pattern

## Introduction

The **Facade Pattern** is a **Structural Design Pattern** that provides a **simple, unified interface** to a complex subsystem.

Instead of interacting with multiple classes directly, the client communicates with a **single Facade**, which coordinates all the underlying components.

> **Facade hides complexity, while Adapter hides incompatibility.**

---

# Definition

> **The Facade Pattern provides a simplified interface to a complex set of classes, libraries, or subsystems.**

---

# Why Do We Need Facade?

Imagine booking a vacation.

Without Facade:

```
Customer

↓

Flight Service

↓

Hotel Service

↓

Car Rental

↓

Payment Service

↓

Insurance Service
```

The customer has to coordinate everything.

With Facade:

```
Customer

↓

Travel Facade

↓

Flight

Hotel

Car

Insurance

Payment
```

The customer only communicates with the **Travel Facade**.

---

# Real-World Analogy

Consider ordering food from an online delivery app.

Without Facade:

```
Restaurant

↓

Delivery Partner

↓

Payment Gateway

↓

Tracking System

↓

Notification Service
```

You would have to call every service individually.

With Facade:

```
Customer

↓

Food Delivery App

↓

All Internal Services
```

The app is the **Facade**.

---

# Structure

```
          Client

             │

             ▼

          Facade

      ┌────┼────┐

      ▼    ▼    ▼

 ServiceA ServiceB ServiceC
```

The client knows only the Facade.

---

# Components

| Component | Responsibility |
|-----------|----------------|
| Client | Uses the simplified interface |
| Facade | Coordinates subsystem operations |
| Subsystem | Performs the actual work |

---

# Example

## Step 1: Subsystems

### Payment Service

```csharp
public class PaymentService
{
    public void Pay(decimal amount)
    {
        Console.WriteLine("Payment Done");
    }
}
```

---

### Inventory Service

```csharp
public class InventoryService
{
    public void ReserveItem()
    {
        Console.WriteLine("Inventory Reserved");
    }
}
```

---

### Shipping Service

```csharp
public class ShippingService
{
    public void Ship()
    {
        Console.WriteLine("Order Shipped");
    }
}
```

---

# Step 2: Facade

```csharp
public class OrderFacade
{
    private readonly PaymentService
        _payment;

    private readonly InventoryService
        _inventory;

    private readonly ShippingService
        _shipping;

    public OrderFacade()
    {
        _payment = new PaymentService();
        _inventory = new InventoryService();
        _shipping = new ShippingService();
    }

    public void PlaceOrder(decimal amount)
    {
        _payment.Pay(amount);

        _inventory.ReserveItem();

        _shipping.Ship();
    }
}
```

---

# Step 3: Client

```csharp
var facade = new OrderFacade();

facade.PlaceOrder(1000);
```

Output:

```
Payment Done

Inventory Reserved

Order Shipped
```

The client does not know anything about the subsystem classes.

---

# Internal Workflow

```
Client

↓

OrderFacade

↓

Payment

↓

Inventory

↓

Shipping
```

The Facade coordinates the workflow.

---

# ASP.NET Core Example

Suppose an order requires:

- Payment
- Email
- Inventory
- Logging

Instead of injecting four services into a controller:

```csharp
public class OrderController
{
    public OrderController(
        PaymentService payment,
        EmailService email,
        InventoryService inventory,
        LoggingService logger)
    {
    }
}
```

Inject a Facade:

```csharp
public class OrderController
{
    private readonly OrderFacade
        _facade;

    public OrderController(
        OrderFacade facade)
    {
        _facade = facade;
    }
}
```

The controller becomes much simpler.

---

# Dependency Injection

Register services:

```csharp
builder.Services.AddScoped<
    PaymentService>();

builder.Services.AddScoped<
    InventoryService>();

builder.Services.AddScoped<
    ShippingService>();

builder.Services.AddScoped<
    OrderFacade>();
```

The DI container injects the subsystem services into the Facade.

A better implementation than the previous example is:

```csharp
public class OrderFacade
{
    private readonly PaymentService _payment;
    private readonly InventoryService _inventory;
    private readonly ShippingService _shipping;

    public OrderFacade(
        PaymentService payment,
        InventoryService inventory,
        ShippingService shipping)
    {
        _payment = payment;
        _inventory = inventory;
        _shipping = shipping;
    }

    public void PlaceOrder(decimal amount)
    {
        _payment.Pay(amount);
        _inventory.ReserveItem();
        _shipping.Ship();
    }
}
```

---

# Advantages

- Simplifies complex APIs
- Reduces coupling
- Improves readability
- Centralizes workflows
- Easier maintenance

---

# Disadvantages

- Facade can become too large ("God Object")
- May hide useful subsystem functionality
- Can introduce an extra abstraction layer

---

# Real-World Uses

- Payment processing
- Order processing
- Banking systems
- Travel booking
- AI orchestration
- Microservice aggregation
- File conversion pipelines

---

# 64. Facade vs Adapter

## Introduction

These two patterns are often confused because both **wrap existing classes**.

However, they solve **completely different problems**.

---

# Core Difference

### Adapter

```
Problem:

Interfaces don't match.
```

Solution:

```
Convert Interface
```

---

### Facade

```
Problem:

System is too complex.
```

Solution:

```
Simplify Interface
```

---

# Visual Comparison

## Adapter

```
Client

↓

Adapter

↓

Third-Party Library
```

Purpose:

```
Compatibility
```

---

## Facade

```
Client

↓

Facade

↓

Subsystem A

↓

Subsystem B

↓

Subsystem C
```

Purpose:

```
Simplification
```

---

# Example

## Adapter

Application expects:

```csharp
ProcessPayment()
```

Third-party SDK provides:

```csharp
MakePayment()
```

Adapter:

```csharp
ProcessPayment()
↓

MakePayment()
```

The method name and interface are translated.

---

## Facade

Subsystem:

```
Payment

Inventory

Shipping

Email

Logging
```

Facade exposes:

```csharp
PlaceOrder()
```

One method internally calls five services.

---

# Intent

| Adapter | Facade |
|----------|---------|
| Makes incompatible interfaces compatible | Provides a simplified interface to a subsystem |

---

# Number of Classes

Adapter:

```
One Client

↓

One Adaptee
```

Facade:

```
One Client

↓

Many Subsystems
```

---

# Focus

Adapter:

```
Compatibility
```

Facade:

```
Usability
```

---

# Real-World Analogy

## Adapter

Traveling from India to the USA.

```
Indian Plug

↓

Power Adapter

↓

US Socket
```

The plug works because the interface is converted.

---

## Facade

Travel agency.

```
Customer

↓

Travel Agency

↓

Flights

Hotels

Insurance

Visa

Taxi
```

The agency hides all the complexity.

---

# ASP.NET Core Example

## Adapter

```
Application

↓

Storage Adapter

↓

Azure Blob

↓

Amazon S3
```

Different cloud SDKs expose different APIs.

The Adapter makes them look the same.

---

## Facade

```
Controller

↓

OrderFacade

↓

Payment

↓

Inventory

↓

Email

↓

Shipping
```

The Facade simplifies the business workflow.

---

# Can They Be Used Together?

Yes.

Example:

```
Controller

↓

OrderFacade

↓

Payment Adapter

↓

Stripe SDK
```

The Facade orchestrates the workflow, while the Adapter hides differences between external providers.

---

# Comparison Table

| Feature | Adapter | Facade |
|----------|----------|---------|
| Design Pattern Type | Structural | Structural |
| Primary Goal | Compatibility | Simplicity |
| Changes Interface | Yes | No |
| Simplifies API | No | Yes |
| Supports Legacy Systems | Yes | Sometimes |
| Wraps | Usually one class | Usually multiple classes |
| Client Sees | Expected interface | Simplified interface |
| Common Usage | Third-party SDKs, legacy systems | Complex business workflows |

---

# Adapter vs Facade vs Decorator

| Pattern | Purpose |
|----------|----------|
| Adapter | Convert interfaces |
| Facade | Simplify subsystem |
| Decorator | Add behavior without changing the interface |

---

# Best Practices

- Use **Adapter** when integrating third-party or legacy components with incompatible APIs.
- Use **Facade** to expose a clean, task-oriented API over a complex subsystem.
- Keep Facades focused on orchestration rather than implementing business rules.
- Avoid creating "God Facades" that coordinate unrelated responsibilities.
- Combine Facade with Dependency Injection so subsystem services remain loosely coupled and testable.

---

# Interview Questions

### Basic

1. What is the Facade Pattern?
2. What problem does the Facade Pattern solve?
3. What is the difference between Facade and Adapter?

### Intermediate

4. Why is Facade useful in ASP.NET Core applications?
5. Can a Facade use Dependency Injection?
6. Why can a Facade become a God Object?

### Advanced

7. How would you design an order processing system using the Facade Pattern?
8. Can Adapter and Facade be used together?
9. Facade vs Mediator?
10. Facade vs Service Layer?

---

# Summary

The **Facade Pattern** provides a **single, simplified interface** to a complex subsystem by coordinating multiple services behind one cohesive API. It improves readability, reduces coupling, and is widely used in enterprise applications to orchestrate workflows such as order processing, payment handling, travel booking, and AI pipelines. In contrast, the **Adapter Pattern** focuses on **making incompatible interfaces compatible** by translating one interface into another. While Adapter solves integration problems, Facade solves complexity problems. In many real-world ASP.NET Core applications, both patterns are used together: Adapters integrate external systems, and Facades orchestrate the overall business workflow.