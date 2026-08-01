# 96. Strategy Pattern

## Introduction

The **Strategy Pattern** is a **Behavioral Design Pattern** that allows you to define a family of algorithms, encapsulate each one, and make them interchangeable at runtime.

Instead of writing multiple `if-else` or `switch` statements, the client delegates the work to a **strategy object**.

> **Strategy allows you to change an algorithm at runtime without modifying the client code.**

---

# Definition

> **Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from the clients that use it.**

---

# Why Do We Need Strategy?

Suppose an e-commerce application supports multiple payment methods.

Without Strategy

```csharp
public class PaymentService
{
    public void Pay(
        string paymentType,
        decimal amount)
    {
        if(paymentType == "CreditCard")
        {
            // Credit Card Logic
        }
        else if(paymentType == "PayPal")
        {
            // PayPal Logic
        }
        else if(paymentType == "UPI")
        {
            // UPI Logic
        }
    }
}
```

Problems:

- Large `if-else` blocks
- Violates Open/Closed Principle
- Difficult to test
- Hard to extend

---

# With Strategy

```
Client

↓

IPaymentStrategy

↓

CreditCard

PayPal

UPI
```

The client works only with the abstraction.

---

# Structure

```
              Client

                 │

                 ▼

          IPaymentStrategy

         ▲       ▲       ▲

         │       │       │

 CreditCard  PayPal   UPI
```

---

# Components

| Component | Responsibility |
|------------|----------------|
| Strategy | Common algorithm interface |
| Concrete Strategy | Specific implementation |
| Context | Uses a strategy |
| Client | Chooses strategy |

---

# Example

## Strategy

```csharp
public interface IPaymentStrategy
{
    void Pay(decimal amount);
}
```

---

## Credit Card Strategy

```csharp
public class CreditCardPayment
    : IPaymentStrategy
{
    public void Pay(decimal amount)
    {
        Console.WriteLine(
            $"Paid {amount} using Credit Card");
    }
}
```

---

## PayPal Strategy

```csharp
public class PayPalPayment
    : IPaymentStrategy
{
    public void Pay(decimal amount)
    {
        Console.WriteLine(
            $"Paid {amount} using PayPal");
    }
}
```

---

## Context

```csharp
public class PaymentContext
{
    private readonly IPaymentStrategy
        _strategy;

    public PaymentContext(
        IPaymentStrategy strategy)
    {
        _strategy = strategy;
    }

    public void Pay(decimal amount)
    {
        _strategy.Pay(amount);
    }
}
```

---

## Client

```csharp
var payment =
    new PaymentContext(
        new PayPalPayment());

payment.Pay(500);
```

Output

```
Paid 500 using PayPal
```

---

# Internal Workflow

```
Client

↓

Context

↓

Selected Strategy

↓

Execute Algorithm
```

---

# Real-World Examples

- Payment gateways
- Tax calculation
- Shipping charges
- Discount calculation
- Authentication providers
- Notification channels
- File compression
- Sorting algorithms

---

# ASP.NET Core Example

Authentication providers.

```
Authentication

↓

JWT

Cookie

OAuth
```

Each authentication mechanism represents a different strategy.

---

# Advantages

- Removes large conditional statements
- Supports Open/Closed Principle
- Easy to add new algorithms
- Easy to unit test
- Algorithms are independent

---

# Disadvantages

- More classes
- Client must select an appropriate strategy
- Slightly higher complexity for simple scenarios

---

# 97. Strategy Pattern in Dependency Injection

## Why DI Works Well with Strategy

ASP.NET Core's built-in Dependency Injection container makes implementing the Strategy Pattern straightforward.

Instead of manually creating strategies, the container provides them.

---

# Registration

```csharp
builder.Services.AddScoped<
    IPaymentStrategy,
    CreditCardPayment>();

builder.Services.AddScoped<
    IPaymentStrategy,
    PayPalPayment>();

builder.Services.AddScoped<
    IPaymentStrategy,
    UpiPayment>();
```

Now multiple implementations are registered for the same interface.

---

# Injecting All Strategies

```csharp
public class PaymentService
{
    private readonly IEnumerable<
        IPaymentStrategy> _strategies;

    public PaymentService(
        IEnumerable<IPaymentStrategy>
            strategies)
    {
        _strategies = strategies;
    }
}
```

The DI container injects all registered strategies.

---

# Selecting a Strategy

A common approach is to let each strategy expose a key.

```csharp
public interface IPaymentStrategy
{
    string Name { get; }

    void Pay(decimal amount);
}
```

Example:

```csharp
public class PayPalPayment
    : IPaymentStrategy
{
    public string Name => "PayPal";

    public void Pay(decimal amount)
    {
        Console.WriteLine(
            "Paid using PayPal");
    }
}
```

Selecting the strategy:

```csharp
var strategy =
    _strategies.First(
        s => s.Name == "PayPal");

strategy.Pay(1000);
```

---

# Workflow

```
Request

↓

Payment Service

↓

DI Container

↓

All Strategies

↓

Select One

↓

Execute
```

---

# Alternative: Factory + Strategy

Instead of searching manually:

```
Client

↓

Strategy Factory

↓

PayPal Strategy
```

The factory encapsulates the selection logic.

---

# Enterprise Examples

### Payment Processing

```
Stripe

PayPal

Razorpay
```

---

### Notification

```
Email

SMS

Push Notification
```

---

### File Export

```
PDF

Excel

CSV
```

---

### Discount Engine

```
Festival Discount

Member Discount

Coupon Discount
```

Each implementation is a strategy.

---

# Benefits with DI

- Easy to extend
- No client changes
- Excellent testability
- Supports runtime selection
- Works naturally with SOLID principles

---

# 98. Strategy vs State Pattern

Although both patterns have similar class diagrams, **their intent is completely different**.

---

# Similar Structure

```
Context

↓

Interface

↓

Concrete Classes
```

However, the purpose differs.

---

# Strategy

### Goal

Choose **which algorithm** to execute.

The client (or a factory) usually selects the strategy.

---

Example

```
Payment

↓

Credit Card

PayPal

UPI
```

The payment method changes, but the object's internal state does not.

---

# State

### Goal

Change behavior based on the object's **current internal state**.

The object changes its own state over time.

---

Example

```
Order

↓

Created

↓

Paid

↓

Shipped

↓

Delivered
```

Each state changes the behavior of the order.

---

# Strategy Workflow

```
Client

↓

Choose Strategy

↓

Execute Algorithm
```

---

# State Workflow

```
Client

↓

Current State

↓

Behavior

↓

State Changes

↓

Next Behavior
```

---

# Example Comparison

### Strategy

```csharp
payment.Pay();
```

Runtime selection:

```
PayPal

↓

Pay()
```

Changing to another strategy:

```
Credit Card

↓

Pay()
```

The client decides which algorithm to use.

---

### State

```csharp
order.Ship();
```

Behavior depends on the current state.

```
Created

↓

Cannot Ship
```

After payment:

```
Paid

↓

Ship Allowed
```

The object changes its own behavior as its state changes.

---

# Comparison Table

| Feature | Strategy | State |
|----------|----------|-------|
| Pattern Type | Behavioral | Behavioral |
| Purpose | Select an algorithm | Represent object state |
| Who changes behavior? | Client or Factory | The object itself |
| Focus | Interchangeable algorithms | State transitions |
| Typical Example | Payment methods | Order lifecycle |
| Runtime Change | Yes | Yes |
| Object State Changes | No | Yes |

---

# When to Use Strategy

Use Strategy when:

- Multiple algorithms solve the same problem.
- You want to eliminate `if-else` or `switch` statements.
- Algorithms should be interchangeable.
- New algorithms will be added frequently.

Examples:

- Payment gateways
- Discount engines
- Compression algorithms
- Authentication providers

---

# When to Use State

Use State when:

- An object's behavior depends on its current state.
- State transitions are well-defined.
- Complex conditional logic depends on object status.

Examples:

- Order processing
- Workflow engines
- Traffic lights
- Document approval
- Media player states

---

# Interview Tips

### Common Questions

**Q:** Why is Strategy preferred over `if-else`?

**Answer:** It removes conditional complexity, supports the Open/Closed Principle, and allows new algorithms to be added without modifying existing code.

---

**Q:** Why does Strategy work well with Dependency Injection?

**Answer:** DI can register multiple implementations of the same interface, inject them as `IEnumerable<T>`, and allow the appropriate strategy to be selected at runtime without coupling the client to concrete classes.

---

**Q:** Strategy vs Factory?

**Answer:** Strategy defines **how** work is performed (the algorithm), while a Factory decides **which strategy** to create or return. They are frequently used together.

---

# Best Practices

- Keep each strategy focused on a single algorithm.
- Depend on abstractions instead of concrete implementations.
- Use DI or a factory to select strategies.
- Avoid placing selection logic inside the strategy itself.
- Use meaningful names that describe the algorithm being implemented.

---

# Interview Questions

### Basic

1. What is the Strategy Pattern?
2. What problem does it solve?
3. Give a real-world example.

### Intermediate

4. How is Strategy implemented using Dependency Injection?
5. Why is Strategy better than `switch` statements?
6. Strategy vs Factory?

### Advanced

7. Strategy vs State?
8. How would you implement a payment gateway using Strategy and DI?
9. How would you support hundreds of strategies efficiently?
10. Design a notification engine using the Strategy Pattern.

---

# Summary

The **Strategy Pattern** is a behavioral design pattern that encapsulates interchangeable algorithms behind a common interface, enabling runtime selection without modifying client code. In ASP.NET Core, it integrates naturally with **Dependency Injection**, where multiple implementations of an interface can be registered and selected dynamically. Common enterprise uses include payment gateways, authentication providers, notification systems, discount engines, and file export services. Although **Strategy** and **State** have similar structures, their intent differs significantly: **Strategy** selects among interchangeable algorithms, while **State** models an object's changing behavior as it transitions through different internal states.