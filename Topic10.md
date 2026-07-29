# 30. Factory vs Abstract Factory

## Introduction

Both **Factory (Simple Factory)** and **Abstract Factory** are **Creational Design Patterns** used to create objects while hiding the object creation logic from the client.

The key difference is:

- **Factory Pattern** creates **one object**.
- **Abstract Factory Pattern** creates **a family of related objects**.

---

# Quick Comparison

| Factory Pattern | Abstract Factory Pattern |
|-----------------|--------------------------|
| Creates one object | Creates a family of related objects |
| One factory class | Multiple factory methods |
| Simpler | More complex |
| Good for selecting one implementation | Good for selecting an entire product family |
| Limited scalability | Highly scalable for product families |

---

# Factory Pattern

## Purpose

Hide the creation logic of **a single object**.

Example:

```
NotificationFactory

↓

EmailService

OR

SmsService

OR

PushNotificationService
```

The client requests one notification service.

---

## Example

Interface:

```csharp
public interface INotificationService
{
    void Send();
}
```

Factory:

```csharp
public class NotificationFactory
{
    public static INotificationService Create(string type)
    {
        return type switch
        {
            "Email" => new EmailService(),
            "SMS" => new SmsService(),
            _ => throw new ArgumentException()
        };
    }
}
```

Client:

```csharp
var service = NotificationFactory.Create("Email");

service.Send();
```

Only **one object** is created.

---

# Abstract Factory Pattern

## Purpose

Create **multiple related objects** that belong to the same family.

Example:

Windows Family

```
Windows Button

Windows Checkbox

Windows TextBox
```

Mac Family

```
Mac Button

Mac Checkbox

Mac TextBox
```

Instead of creating each object separately, the factory creates an entire compatible family.

---

## Example

Abstract Factory:

```csharp
public interface IGuiFactory
{
    IButton CreateButton();

    ICheckBox CreateCheckBox();

    ITextBox CreateTextBox();
}
```

Windows Factory:

```csharp
public class WindowsFactory : IGuiFactory
{
    public IButton CreateButton()
    {
        return new WindowsButton();
    }

    public ICheckBox CreateCheckBox()
    {
        return new WindowsCheckBox();
    }

    public ITextBox CreateTextBox()
    {
        return new WindowsTextBox();
    }
}
```

Client:

```csharp
IGuiFactory factory = new WindowsFactory();

IButton button = factory.CreateButton();

ICheckBox checkbox = factory.CreateCheckBox();

ITextBox textBox = factory.CreateTextBox();
```

The client receives a complete family of compatible Windows UI components.

---

# Visual Comparison

## Factory Pattern

```
               NotificationFactory
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
EmailService   SmsService   PushService
```

One factory creates **one selected product**.

---

## Abstract Factory Pattern

```
                GUI Factory
                    ▲
          ┌─────────┴─────────┐
          │                   │
 WindowsFactory         MacFactory
    │   │   │             │   │   │
    ▼   ▼   ▼             ▼   ▼   ▼
Button Checkbox TextBox Button Checkbox TextBox
```

One factory creates **an entire product family**.

---

# Real-World Example

## Factory Pattern

Imagine ordering a drink.

You choose one item:

- Coffee
- Tea
- Juice

The café prepares only the selected drink.

---

## Abstract Factory Pattern

Imagine ordering a meal combo.

A **Veg Combo** includes:

- Burger
- Fries
- Drink

A **Kids Combo** includes:

- Small Burger
- Small Fries
- Juice

A single order gives you a complete, compatible set of products.

---

# ASP.NET Core Example

## Factory Pattern

Suppose an application supports multiple payment gateways.

```
PaymentFactory

↓

Stripe

OR

Razorpay

OR

PayPal
```

Only one payment gateway object is created.

---

## Abstract Factory Pattern

Suppose the application supports multiple cloud providers.

Azure Factory creates:

```
Azure Blob Storage

Azure Queue

Azure Key Vault
```

AWS Factory creates:

```
Amazon S3

Amazon SQS

AWS Secrets Manager
```

The application switches the **entire cloud provider** by changing only the factory.

---

# Advantages

## Factory Pattern

- Simple implementation
- Centralized object creation
- Reduces coupling
- Easy to understand
- Suitable for a single product

---

## Abstract Factory Pattern

- Creates compatible product families
- Prevents mixing incompatible objects
- Easy to switch entire product families
- Excellent support for the Open/Closed Principle
- Well suited for enterprise applications

---

# Disadvantages

## Factory Pattern

- Factory can become large if many product types are added.
- Usually creates only one type of object.

---

## Abstract Factory Pattern

- More interfaces and classes.
- Higher complexity.
- Adding a new product type requires changes to every concrete factory.

---

# When to Use Which?

## Use Factory Pattern When

- You need to create one object.
- Object creation depends on runtime input.
- The application is relatively simple.
- You want to centralize object creation.

Examples:

- Notification service
- Payment gateway
- File parser
- Logger

---

## Use Abstract Factory When

- Products belong to related families.
- Objects must work together.
- The application supports multiple environments or platforms.
- Entire product families may change.

Examples:

- Windows vs macOS UI components
- Azure vs AWS services
- SQL Server vs MongoDB repositories
- Light theme vs Dark theme UI controls

---

# Factory vs Abstract Factory

| Feature | Factory Pattern | Abstract Factory |
|---------|-----------------|------------------|
| Category | Creational | Creational |
| Creates | One object | Family of related objects |
| Complexity | Low | High |
| Number of factory methods | Usually one | Multiple |
| Uses inheritance | Not necessarily | Commonly uses interfaces and multiple implementations |
| Product compatibility | Not guaranteed | Guaranteed within a family |
| Best for | Single implementation selection | Platform or family selection |
| OCP Support | Good | Excellent |
| Enterprise Usage | Medium | Very High |

---

# Factory, Factory Method, and Abstract Factory

| Pattern | Creates | Typical Use Case |
|---------|----------|------------------|
| **Factory (Simple Factory)** | One object using a single factory class | Notification service, payment gateway |
| **Factory Method** | One object, but subclasses decide which implementation to create | Frameworks, extensible libraries |
| **Abstract Factory** | A family of related objects | Cross-platform UI, cloud providers, database providers |

---

# Interview Questions

### Basic

1. What is the difference between Factory and Abstract Factory?
2. Which pattern creates a family of objects?
3. Which pattern is simpler?

### Intermediate

4. Give a real-world example of both patterns.
5. When would you choose Abstract Factory over Factory?
6. How do both patterns support the Open/Closed Principle?

### Advanced

7. Explain Factory vs Abstract Factory using an ASP.NET Core example.
8. How does Dependency Injection reduce the need for factories?
9. Can Factory Method and Abstract Factory be combined?
10. Which pattern would you use for supporting Azure and AWS services, and why?

---

# Summary

The **Factory Pattern** is used to create **a single object** while hiding its creation logic from the client. It is simple, easy to implement, and ideal when the application needs to choose one implementation at runtime. The **Abstract Factory Pattern** extends this concept by creating **families of related objects** that are designed to work together, making it ideal for cross-platform applications, multiple cloud providers, or different UI themes. While the Factory Pattern emphasizes simplicity, the Abstract Factory Pattern emphasizes consistency and scalability across related product families.