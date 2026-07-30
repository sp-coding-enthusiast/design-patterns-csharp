# 46. Registry Pattern

## Introduction

The **Registry Pattern** is a design pattern that provides a **central repository** for storing and retrieving shared objects or services using a unique key.

Unlike the **Singleton Pattern**, which ensures there is only one instance of a specific class, a **Registry** can store **multiple objects** and allows them to be accessed from a central location.

The Registry Pattern is commonly used in:

- Plugin systems
- Driver registration
- ORM mappings
- Event handlers
- Serialization frameworks
- Application configuration

---

# Definition

> **The Registry Pattern maintains a centralized collection of shared objects that can be registered and retrieved using a unique identifier.**

---

# Why Do We Need Registry Pattern?

Imagine an application that supports multiple payment providers.

Without Registry:

```csharp
if (provider == "Stripe")
{
    return new StripePayment();
}

if (provider == "PayPal")
{
    return new PayPalPayment();
}

if (provider == "Razorpay")
{
    return new RazorpayPayment();
}
```

Problems:

- Large `if-else` or `switch` statements
- Difficult to extend
- Violates the Open/Closed Principle

---

# Registry Solution

```
Application

↓

Registry

↓

Stripe

↓

PayPal

↓

Razorpay
```

Adding a new provider only requires registering it.

---

# Registry Structure

```
                 Client

                    │

                    ▼

              Service Registry

           ▲      ▲       ▲

           │      │       │

      Payment   Logger   Cache

           │      │       │

     Stripe   FileLogger Redis
```

The client requests an object using a key instead of creating it directly.

---

# Simple Implementation

## Step 1: Interface

```csharp
public interface IPayment
{
    void Pay(decimal amount);
}
```

---

## Step 2: Implementations

```csharp
public class StripePayment : IPayment
{
    public void Pay(decimal amount)
    {
        Console.WriteLine("Paid using Stripe");
    }
}
```

```csharp
public class PayPalPayment : IPayment
{
    public void Pay(decimal amount)
    {
        Console.WriteLine("Paid using PayPal");
    }
}
```

---

## Step 3: Registry

```csharp
public static class PaymentRegistry
{
    private static readonly Dictionary<string, IPayment>
        _payments = new();

    public static void Register(
        string key,
        IPayment payment)
    {
        _payments[key] = payment;
    }

    public static IPayment Get(string key)
    {
        return _payments[key];
    }
}
```

---

## Step 4: Registration

```csharp
PaymentRegistry.Register(
    "Stripe",
    new StripePayment());

PaymentRegistry.Register(
    "PayPal",
    new PayPalPayment());
```

---

## Step 5: Usage

```csharp
IPayment payment =
    PaymentRegistry.Get("Stripe");

payment.Pay(500);
```

Output:

```
Paid using Stripe
```

---

# Internal Workflow

```
Application Starts

↓

Register Services

↓

Store in Dictionary

↓

Client Requests "Stripe"

↓

Registry Returns Object
```

---

# Real-World Analogy

Think of a **hotel directory**.

```
Room 101 → Housekeeping

Room 102 → Maintenance

Room 103 → Laundry
```

Guests don't need to know who provides the service.

They simply ask the directory.

The directory acts as the registry.

---

# Registry vs Dictionary

At first glance, a Registry looks like a `Dictionary`.

The difference is **intent**.

A Dictionary is a general-purpose data structure.

A Registry is a design pattern that uses a collection (often a dictionary) to provide centralized registration and lookup of shared objects.

---

# Registry in ASP.NET Core

ASP.NET Core's Dependency Injection container internally maintains registrations similar to a registry.

Example:

```csharp
builder.Services.AddScoped<
    INotificationService,
    EmailService>();

builder.Services.AddSingleton<
    ICacheService,
    RedisCacheService>();
```

The DI container stores service registrations and resolves them later.

Conceptually:

```
Service Type

↓

Implementation

↓

Lifetime
```

Although the built-in DI container behaves like a registry internally, developers interact with it through Dependency Injection rather than direct lookups.

---

# Registry in Plugin Systems

Suppose an application supports plugins.

```csharp
PluginRegistry.Register(
    "PDF",
    new PdfExporter());

PluginRegistry.Register(
    "Excel",
    new ExcelExporter());
```

Later:

```csharp
var exporter =
    PluginRegistry.Get("PDF");
```

Adding a new exporter requires only registration.

---

# Registry in ORMs

ORM frameworks maintain metadata registries.

Example:

```
User

↓

Users Table

↓

Mapping Registry
```

Instead of repeatedly discovering mappings, the framework looks them up from a registry.

---

# Registry in Serialization

Serialization libraries maintain registries that map:

```
Type

↓

Serializer
```

Example:

```
Customer

↓

JsonSerializer

Order

↓

XmlSerializer
```

---

# Advantages

- Centralized registration
- Easy lookup
- Reduces large conditional statements
- Easy to extend
- Supports plugin architectures
- Promotes configurability

---

# Disadvantages

- Can introduce global state
- Hidden dependencies if overused
- Difficult to test when implemented as static
- Similar risks to the Service Locator Pattern if business code accesses it directly
- Thread safety must be considered

---

# Registry vs Singleton

| Registry | Singleton |
|-----------|-----------|
| Stores many objects | Stores one object |
| Key-based lookup | Direct instance access |
| Multiple services | Single shared instance |
| Central repository | Single global object |

---

# Registry vs Factory

| Registry | Factory |
|-----------|---------|
| Returns registered objects | Creates new objects |
| Objects often already exist | Objects are created on demand |
| Lookup-based | Creation-based |

---

# Registry vs Service Locator

This comparison is often confusing.

### Registry

```
Registry

↓

Stores Objects

↓

Returns Objects
```

Its main responsibility is **registration and lookup**.

---

### Service Locator

```
Client

↓

Service Locator

↓

Resolves Dependencies
```

Its main responsibility is **providing dependencies to application code**.

A Service Locator often uses a Registry internally.

---

# Common Use Cases

- Plugin registration
- Driver registration
- Event handler registration
- Serializer registration
- Command handlers
- Workflow registration
- AI model registry
- Machine learning pipeline registration

---

# Modern Alternative

In modern .NET applications, instead of creating custom registries, developers often rely on the built-in Dependency Injection container.

Example:

```csharp
builder.Services.AddSingleton<
    IPayment,
    StripePayment>();
```

or use collections of services:

```csharp
IEnumerable<IPayment>
```

combined with a factory or strategy to select the appropriate implementation.

This approach provides registration, lifetime management, and dependency resolution in a unified way.

---

# Best Practices

- Keep registries focused on infrastructure rather than business logic.
- Avoid using static registries throughout the application.
- Ensure thread safety if registrations can change at runtime.
- Prefer Dependency Injection for application services.
- Use registries for metadata, plugins, handlers, or extensibility points rather than replacing DI.

---

# Interview Questions

### Basic

1. What is the Registry Pattern?
2. Why do we use a Registry?
3. How is a Registry different from a Dictionary?

### Intermediate

4. What is the difference between Registry and Factory?
5. What is the difference between Registry and Singleton?
6. Where is the Registry Pattern commonly used?

### Advanced

7. How does the ASP.NET Core DI container resemble a Registry?
8. Why can a Registry become similar to a Service Locator?
9. What are the drawbacks of implementing registries as static classes?
10. How would you design a plugin system using the Registry Pattern?

---

# Summary

The **Registry Pattern** provides a **centralized repository** for registering and retrieving shared objects using unique keys. It is especially useful in plugin architectures, serializers, event handlers, and framework infrastructure where components must be discovered dynamically. Unlike the **Factory Pattern**, which creates new objects, a Registry typically stores and returns existing registrations. Although useful for infrastructure, overusing registries in business logic can introduce hidden dependencies and global state. In modern ASP.NET Core applications, the built-in Dependency Injection container often fulfills many of the responsibilities that custom registries previously handled.