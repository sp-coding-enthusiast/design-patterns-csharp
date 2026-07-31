# 61. Adapter Pattern

## Introduction

The **Adapter Pattern** is a **Structural Design Pattern** that allows **two incompatible interfaces to work together** without modifying their existing code.

It acts as a **bridge** between an existing class (called the **Adaptee**) and the interface expected by the client (called the **Target**).

> Think of an Adapter as a **translator** between two people speaking different languages.

---

# Definition

> **The Adapter Pattern converts the interface of an existing class into another interface that clients expect, enabling classes with incompatible interfaces to collaborate.**

---

# Why Do We Need Adapter?

Imagine an application expects this interface:

```csharp
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
}
```

But a third-party library provides:

```csharp
public class StripeGateway
{
    public void MakePayment(decimal amount)
    {
        Console.WriteLine($"Paid ₹{amount}");
    }
}
```

The interfaces don't match.

Without Adapter:

```
Application

↓

IPaymentProcessor

❌

StripeGateway
```

With Adapter:

```
Application

↓

Payment Adapter

↓

StripeGateway
```

---

# Real-World Analogy

Consider a mobile charger.

```
Indian Socket

↓

Power Adapter

↓

US Laptop Charger
```

The adapter converts one interface into another.

Another example:

```
English Speaker

↓

Translator

↓

Japanese Speaker
```

Neither side changes; the translator adapts communication.

---

# Structure

```
          Client

             │

             ▼

        Target Interface

             ▲

             │

         Adapter

             │

             ▼

         Adaptee
```

---

# Components

| Component | Responsibility |
|-----------|----------------|
| Client | Uses the target interface |
| Target | Interface expected by the client |
| Adapter | Converts one interface to another |
| Adaptee | Existing class with incompatible interface |

---

# Example

## Step 1: Target Interface

```csharp
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
}
```

---

## Step 2: Adaptee

```csharp
public class StripeGateway
{
    public void MakePayment(decimal amount)
    {
        Console.WriteLine(
            $"Stripe processed ₹{amount}");
    }
}
```

---

## Step 3: Adapter

```csharp
public class StripeAdapter
    : IPaymentProcessor
{
    private readonly StripeGateway
        _gateway;

    public StripeAdapter(
        StripeGateway gateway)
    {
        _gateway = gateway;
    }

    public void ProcessPayment(
        decimal amount)
    {
        _gateway.MakePayment(amount);
    }
}
```

---

## Step 4: Client

```csharp
IPaymentProcessor processor =
    new StripeAdapter(
        new StripeGateway());

processor.ProcessPayment(500);
```

Output:

```
Stripe processed ₹500
```

---

# Internal Workflow

```
Client

↓

ProcessPayment()

↓

Adapter

↓

MakePayment()

↓

Stripe Gateway
```

The client never interacts directly with the third-party class.

---

# Object Adapter vs Class Adapter

### Object Adapter (Preferred in C#)

Uses composition.

```
Adapter

↓

Contains

↓

Adaptee
```

Example:

```csharp
private readonly StripeGateway
    _gateway;
```

---

### Class Adapter

Uses inheritance.

```
Adapter

↓

Inherits

↓

Adaptee
```

This is less common in C# because multiple inheritance of classes is not supported.

---

# Advantages

- Reuses existing code
- No need to modify third-party libraries
- Supports the Open/Closed Principle
- Promotes loose coupling
- Simplifies legacy integration

---

# Disadvantages

- Adds an extra layer of abstraction
- Can increase the number of classes
- Too many adapters can complicate maintenance

---

# Adapter vs Facade

| Adapter | Facade |
|----------|---------|
| Makes incompatible interfaces compatible | Simplifies a complex subsystem |
| Focuses on compatibility | Focuses on usability |
| Usually wraps one class | Often wraps many classes |

---

# Adapter vs Decorator

| Adapter | Decorator |
|----------|------------|
| Changes the interface | Keeps the same interface |
| Enables compatibility | Adds new behavior |
| Solves integration problems | Solves extension problems |

---

# 62. Adapter Pattern Example (ASP.NET Core)

## Scenario

Suppose an application supports multiple payment providers.

The application expects:

```csharp
IPaymentProcessor
```

However, third-party gateways expose different APIs.

---

### Stripe SDK

```csharp
public class StripeGateway
{
    public void MakePayment(
        decimal amount)
    {
        Console.WriteLine(
            "Stripe Payment");
    }
}
```

---

### PayPal SDK

```csharp
public class PayPalGateway
{
    public void SendPayment(
        decimal amount)
    {
        Console.WriteLine(
            "PayPal Payment");
    }
}
```

Both expose different method names.

---

# Common Interface

```csharp
public interface IPaymentProcessor
{
    void ProcessPayment(
        decimal amount);
}
```

---

# Stripe Adapter

```csharp
public class StripeAdapter
    : IPaymentProcessor
{
    private readonly StripeGateway
        _gateway;

    public StripeAdapter(
        StripeGateway gateway)
    {
        _gateway = gateway;
    }

    public void ProcessPayment(
        decimal amount)
    {
        _gateway.MakePayment(amount);
    }
}
```

---

# PayPal Adapter

```csharp
public class PayPalAdapter
    : IPaymentProcessor
{
    private readonly PayPalGateway
        _gateway;

    public PayPalAdapter(
        PayPalGateway gateway)
    {
        _gateway = gateway;
    }

    public void ProcessPayment(
        decimal amount)
    {
        _gateway.SendPayment(amount);
    }
}
```

---

# Client Code

```csharp
IPaymentProcessor stripe =
    new StripeAdapter(
        new StripeGateway());

IPaymentProcessor paypal =
    new PayPalAdapter(
        new PayPalGateway());

stripe.ProcessPayment(500);

paypal.ProcessPayment(1000);
```

Output:

```
Stripe Payment

PayPal Payment
```

Notice that the client code is identical regardless of the underlying provider.

---

# Dependency Injection Example

Register adapters:

```csharp
builder.Services.AddScoped<
    StripeGateway>();

builder.Services.AddScoped<
    StripeAdapter>();
```

Inject into a service:

```csharp
public class CheckoutService
{
    private readonly IPaymentProcessor
        _processor;

    public CheckoutService(
        IPaymentProcessor processor)
    {
        _processor = processor;
    }

    public void Checkout(decimal amount)
    {
        _processor.ProcessPayment(amount);
    }
}
```

The service depends only on the abstraction.

---

# Real-World ASP.NET Core Uses

Adapter Pattern appears frequently in enterprise applications:

### External Payment Providers

```
Application

↓

Payment Adapter

↓

Stripe

↓

PayPal

↓

Razorpay
```

---

### SMS Providers

```
Application

↓

SMS Adapter

↓

Twilio

↓

AWS SNS

↓

Azure Communication Services
```

---

### Cloud Storage

```
Application

↓

Storage Adapter

↓

Azure Blob Storage

↓

Amazon S3

↓

Google Cloud Storage
```

---

### AI Providers

```
Application

↓

LLM Adapter

↓

OpenAI

↓

Azure OpenAI

↓

Claude

↓

Gemini
```

Each provider exposes different SDKs, but the application can work against a common interface.

---

# Internal Flow

```
Controller

↓

IPaymentProcessor

↓

Adapter

↓

Third-party SDK

↓

External API
```

The controller never depends directly on vendor-specific APIs.

---

# Best Practices

- Prefer **composition** over inheritance when implementing adapters.
- Keep adapters thin—they should translate interfaces, not contain business logic.
- Depend on interfaces (`IPaymentProcessor`) rather than concrete adapters.
- Register adapters with Dependency Injection when integrating external services.
- Create one adapter per external API to isolate vendor-specific code.

---

# Interview Questions

### Basic

1. What is the Adapter Pattern?
2. Why do we need an Adapter?
3. What are the Target and Adaptee?

### Intermediate

4. What is the difference between Object Adapter and Class Adapter?
5. How does the Adapter Pattern support the Open/Closed Principle?
6. Adapter vs Facade?

### Advanced

7. How would you integrate multiple payment gateways using the Adapter Pattern?
8. Why is composition preferred over inheritance for adapters in C#?
9. How is the Adapter Pattern used in cloud integrations?
10. Can the Adapter Pattern be combined with Dependency Injection?

---

# Summary

The **Adapter Pattern** is a structural design pattern that enables classes with incompatible interfaces to work together by introducing an intermediary that translates one interface into another. It is particularly valuable when integrating third-party libraries, legacy systems, cloud providers, payment gateways, AI services, or external SDKs without modifying existing code. In modern ASP.NET Core applications, adapters are typically implemented using **composition**, registered through **Dependency Injection**, and exposed via application-specific interfaces. This approach improves maintainability, reduces coupling, and allows external implementations to be replaced with minimal impact on the rest of the application.