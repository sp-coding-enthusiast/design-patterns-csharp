# 26. What are Creational Design Patterns?

## Introduction

**Creational Design Patterns** are one of the three categories of design patterns defined by the **Gang of Four (GoF)**.

The three categories are:

1. **Creational Patterns** – Deal with object creation.
2. **Structural Patterns** – Deal with object composition.
3. **Behavioral Patterns** – Deal with communication between objects.

Creational patterns focus on **how objects are created**. Instead of creating objects directly using the `new` keyword throughout the application, these patterns provide flexible and reusable ways to create objects.

---

# Why Do We Need Creational Patterns?

Consider the following code:

```csharp
public class OrderService
{
    public void PlaceOrder()
    {
        EmailService emailService = new EmailService();

        emailService.Send();
    }
}
```

Problems:

- Tight coupling
- Difficult to replace implementations
- Hard to unit test
- Difficult to extend

Creational patterns solve these issues by separating **object creation** from **object usage**.

---

# Advantages of Creational Patterns

- Loose coupling
- Better code reuse
- Easier testing
- Better scalability
- Centralized object creation
- Improved maintainability
- Supports Dependency Injection
- Encourages SOLID principles

---

# Types of Creational Design Patterns

The Gang of Four defines **five** creational design patterns.

| Pattern | Purpose | Typical Use Case |
|---------|---------|------------------|
| **Factory Method** | Creates one object through subclasses or factory methods | Selecting one implementation based on input |
| **Abstract Factory** | Creates families of related objects | Cross-platform UI, multiple database providers |
| **Builder** | Constructs complex objects step by step | Building immutable or configurable objects |
| **Prototype** | Creates objects by cloning existing ones | Expensive object creation |
| **Singleton** | Ensures only one instance exists | Logging, configuration, caching |

---

# When Should We Use Creational Patterns?

Use them when:

- Object creation is complex.
- Different implementations may be chosen at runtime.
- The client should not know concrete classes.
- Objects require many configuration options.
- The same object is created repeatedly.
- Future implementations are expected.

---

# Real-World Analogy

Imagine ordering coffee at a café.

You tell the cashier:

> "Give me a Cappuccino."

You don't:

- Grind the beans
- Heat the milk
- Brew the espresso

The café creates the coffee and gives you the finished product.

Similarly, your application asks a factory for an object instead of creating it directly.

---

# Creational Patterns and SOLID

| SOLID Principle | Benefit |
|-----------------|---------|
| SRP | Object creation is separated from business logic |
| OCP | New object types can be added without changing clients |
| LSP | Factories return abstractions that can be substituted |
| ISP | Clients depend only on required interfaces |
| DIP | Clients depend on interfaces instead of concrete classes |

---

# Example Without a Creational Pattern

```csharp
public class NotificationService
{
    public void Send()
    {
        EmailService email = new EmailService();

        email.Send();
    }
}
```

Problems:

- Tight coupling
- Difficult to switch to SMS or Push Notifications
- Difficult to test

---

# Example With a Factory

```csharp
INotificationService service =
    NotificationFactory.Create("Email");

service.Send();
```

The client does not know which concrete class is instantiated.

---

# Common Uses in ASP.NET Core

- Dependency Injection Container
- ILogger<T>
- DbContext creation
- IHttpClientFactory
- IConfiguration
- Identity services

Although these are not always direct implementations of GoF patterns, they rely heavily on creational concepts.

---

# 27. Factory Pattern

## Definition

The **Factory Pattern** is a creational design pattern that provides an interface or method for creating objects **without exposing the object creation logic** to the client.

Instead of using:

```csharp
new EmailService();
```

the client asks a factory:

```csharp
NotificationFactory.Create("Email");
```

The factory decides which object to create.

---

# Why Do We Need the Factory Pattern?

Without the Factory Pattern:

```csharp
public class NotificationService
{
    public void Send(string type)
    {
        if(type == "Email")
        {
            EmailService service = new EmailService();
            service.Send();
        }
        else if(type == "SMS")
        {
            SmsService service = new SmsService();
            service.Send();
        }
    }
}
```

Problems:

- Tight coupling
- Long `if-else` chains
- Violates the Open/Closed Principle
- Difficult to add new notification types

---

# Factory Pattern Structure

```
                Client
                   │
                   ▼
          NotificationFactory
                   │
     ┌─────────────┼─────────────┐
     ▼             ▼             ▼
EmailService   SmsService   PushNotificationService
```

The client knows only about the factory and the abstraction.

---

# Step 1: Create an Interface

```csharp
public interface INotificationService
{
    void Send();
}
```

---

# Step 2: Create Concrete Classes

### Email

```csharp
public class EmailService : INotificationService
{
    public void Send()
    {
        Console.WriteLine("Email Sent");
    }
}
```

### SMS

```csharp
public class SmsService : INotificationService
{
    public void Send()
    {
        Console.WriteLine("SMS Sent");
    }
}
```

### Push Notification

```csharp
public class PushNotificationService : INotificationService
{
    public void Send()
    {
        Console.WriteLine("Push Notification Sent");
    }
}
```

---

# Step 3: Create the Factory

```csharp
public class NotificationFactory
{
    public static INotificationService Create(string type)
    {
        switch(type)
        {
            case "Email":
                return new EmailService();

            case "SMS":
                return new SmsService();

            case "Push":
                return new PushNotificationService();

            default:
                throw new ArgumentException("Invalid notification type");
        }
    }
}
```

---

# Step 4: Client Code

```csharp
INotificationService service =
    NotificationFactory.Create("Email");

service.Send();
```

Output:

```
Email Sent
```

---

# Adding a New Notification Type

Suppose the business introduces WhatsApp notifications.

```csharp
public class WhatsAppService : INotificationService
{
    public void Send()
    {
        Console.WriteLine("WhatsApp Message Sent");
    }
}
```

Update the factory:

```csharp
case "WhatsApp":
    return new WhatsAppService();
```

The client code remains unchanged.

---

# Factory Pattern vs Direct Object Creation

| Direct `new` | Factory Pattern |
|--------------|-----------------|
| Client creates object | Factory creates object |
| Tight coupling | Loose coupling |
| Hard to replace implementations | Easy to switch implementations |
| Scattered object creation | Centralized creation logic |
| Difficult to test | Easier to mock and test |

---

# Real-World Example

An ATM is a good example of a factory.

You request:

- Cash Withdrawal
- Balance Inquiry
- Mini Statement

The ATM internally decides which service or process to invoke. You interact with a single interface without knowing how each operation is created or executed.

---

# Factory Pattern in ASP.NET Core

Although ASP.NET Core primarily uses **Dependency Injection**, the Factory Pattern is still common.

## Example: Payment Gateway Factory

Interface:

```csharp
public interface IPaymentGateway
{
    void Pay(decimal amount);
}
```

Implementations:

```csharp
public class RazorpayGateway : IPaymentGateway
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ₹{amount} using Razorpay");
    }
}

public class StripeGateway : IPaymentGateway
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ₹{amount} using Stripe");
    }
}
```

Factory:

```csharp
public class PaymentGatewayFactory
{
    public static IPaymentGateway Create(string provider)
    {
        return provider switch
        {
            "Razorpay" => new RazorpayGateway(),
            "Stripe" => new StripeGateway(),
            _ => throw new ArgumentException("Invalid provider")
        };
    }
}
```

Usage:

```csharp
var gateway = PaymentGatewayFactory.Create("Stripe");

gateway.Pay(1000);
```

---

# Advantages

- Encapsulates object creation.
- Reduces coupling.
- Promotes code reuse.
- Simplifies client code.
- Supports OCP and DIP.
- Easier unit testing.

---

# Disadvantages

- Adds an extra layer of abstraction.
- Factory classes may grow if too many object types are handled.
- Can become a maintenance bottleneck if not refactored into Factory Method or Abstract Factory when the application grows.

---

# Factory Pattern vs Factory Method

| Factory Pattern (Simple Factory) | Factory Method |
|----------------------------------|----------------|
| One factory class decides which object to create | Subclasses decide which object to create |
| Usually uses `if`, `switch`, or mappings | Uses inheritance and polymorphism |
| Simpler | More extensible |
| Good for small to medium applications | Better for larger, evolving systems |

---

# Interview Questions

### Basic

1. What are creational design patterns?
2. What problem does the Factory Pattern solve?
3. Why is the Factory Pattern considered a creational pattern?

### Intermediate

4. How does the Factory Pattern support the Open/Closed Principle?
5. What is the difference between direct object creation and the Factory Pattern?
6. How does the Factory Pattern improve testability?

### Advanced

7. What is the difference between Simple Factory, Factory Method, and Abstract Factory?
8. How is the Factory Pattern used alongside Dependency Injection in ASP.NET Core?
9. When would you prefer a Factory over a Strategy Pattern?
10. Can the Factory Pattern violate SRP or OCP if implemented poorly?

---

# Summary

**Creational Design Patterns** focus on creating objects in a flexible, reusable, and maintainable way. The **Factory Pattern** centralizes object creation, hides implementation details from clients, and reduces coupling by returning abstractions instead of concrete classes. It works well with SOLID principles—especially **OCP** and **DIP**—and is widely used in enterprise applications and ASP.NET Core when object creation depends on runtime conditions or complex initialization logic.