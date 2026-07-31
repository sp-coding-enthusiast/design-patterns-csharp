# 47. Static Factory Pattern

## Introduction

A **Static Factory** is a variation of the Factory Pattern where the factory methods are declared as **static methods**.

Unlike a normal factory, you do **not** create an instance of the factory class.

Instead of:

```csharp
var factory = new CarFactory();

Car car = factory.Create();
```

You write:

```csharp
Car car = CarFactory.Create();
```

---

# Definition

> **A Static Factory provides static methods to create objects without requiring an instance of the factory class.**

---

# Why Use Static Factory?

Instead of allowing clients to instantiate objects directly:

```csharp
Car car = new Car();
```

Use:

```csharp
Car car = CarFactory.Create();
```

Benefits:

- Centralizes creation logic
- Hides complex initialization
- Prevents invalid object creation
- Easier maintenance

---

# Example

## Product

```csharp
public class Car
{
    public string Model { get; }

    public Car(string model)
    {
        Model = model;
    }
}
```

---

## Static Factory

```csharp
public static class CarFactory
{
    public static Car CreateTesla()
    {
        return new Car("Tesla");
    }

    public static Car CreateBMW()
    {
        return new Car("BMW");
    }
}
```

---

## Client

```csharp
Car tesla = CarFactory.CreateTesla();

Car bmw = CarFactory.CreateBMW();
```

---

# Real-World Example

The .NET Framework uses static factories extensively.

Example:

```csharp
DateTime now = DateTime.Now;

Guid id = Guid.NewGuid();

Encoding utf8 = Encoding.UTF8;
```

`Guid.NewGuid()` is a classic example of a static factory method.

---

# Advantages

- Simple to use
- No factory object required
- Centralized object creation
- Good for utility classes

---

# Disadvantages

- Cannot implement interfaces
- Difficult to mock in tests
- Less flexible than instance factories

---

# 48. Generic Factory Pattern

## Introduction

A **Generic Factory** uses **generics** to create objects without writing a separate factory for every type.

Instead of:

```
CarFactory

BikeFactory

TruckFactory
```

You have one reusable factory.

---

# Example

```csharp
public class Factory<T>
    where T : new()
{
    public T Create()
    {
        return new T();
    }
}
```

---

## Usage

```csharp
Factory<Customer> customerFactory =
    new();

Customer customer =
    customerFactory.Create();

Factory<Order> orderFactory =
    new();

Order order =
    orderFactory.Create();
```

---

# Generic Factory with Constraints

```csharp
public interface IEntity
{
}

public class Factory<T>
    where T : IEntity, new()
{
    public T Create()
    {
        return new T();
    }
}
```

Only classes implementing `IEntity` can be created.

---

# Real-World Example

Imagine a toy factory.

Instead of separate machines for every toy:

```
Toy Factory

↓

Car

↓

Robot

↓

Train

↓

Doll
```

The same machine can produce different toys based on the generic type.

---

# ASP.NET Core Example

Generic Repository:

```csharp
public class RepositoryFactory<T>
    where T : class, new()
{
    public T Create()
    {
        return new T();
    }
}
```

Although simple examples use the `new()` constraint, enterprise applications often inject dependencies into generic factories instead of directly calling constructors.

---

# Advantages

- Eliminates duplicate factories
- Reusable
- Type-safe
- Easy to extend

---

# Disadvantages

- Limited when constructors require parameters
- Cannot easily choose implementations at runtime
- Complex generic constraints can reduce readability

---

# 49. Parameterized Factory Pattern

## Introduction

A **Parameterized Factory** creates different objects based on the parameters supplied by the client.

Instead of multiple methods:

```csharp
CreateEmail()

CreateSms()

CreatePush()
```

Use one method:

```csharp
Create("Email")
```

---

# Example

Interface:

```csharp
public interface INotification
{
    void Send();
}
```

Implementations:

```csharp
public class EmailNotification : INotification
{
    public void Send()
    {
        Console.WriteLine("Email");
    }
}

public class SmsNotification : INotification
{
    public void Send()
    {
        Console.WriteLine("SMS");
    }
}
```

---

## Factory

```csharp
public static class NotificationFactory
{
    public static INotification Create(
        string type)
    {
        return type switch
        {
            "Email" => new EmailNotification(),

            "SMS" => new SmsNotification(),

            _ => throw new ArgumentException(
                "Invalid Type")
        };
    }
}
```

---

## Usage

```csharp
var notification =
    NotificationFactory.Create("SMS");

notification.Send();
```

---

# Better Approach Using Enum

```csharp
public enum NotificationType
{
    Email,
    Sms
}
```

Factory:

```csharp
public static INotification Create(
    NotificationType type)
{
    return type switch
    {
        NotificationType.Email =>
            new EmailNotification(),

        NotificationType.Sms =>
            new SmsNotification(),

        _ => throw new ArgumentException()
    };
}
```

Enums reduce typing mistakes and improve readability.

---

# Real-World Example

ATM Machine

```
Withdraw

Deposit

Balance
```

The selected option determines which operation is created or executed.

---

# Advantages

- Single entry point
- Easy runtime selection
- Simple implementation

---

# Disadvantages

- Large `switch` statements can violate the Open/Closed Principle
- Factory grows as new types are added
- Requires modification for every new product

---

# 50. Named Factory Pattern

## Introduction

A **Named Factory** creates objects using a **name or key** instead of directly instantiating a class.

It is commonly used in:

- Plugin systems
- Payment gateways
- Logging providers
- Cloud providers
- Dependency Injection containers

---

# Example

```
"Stripe"

↓

StripePayment

"PayPal"

↓

PayPalPayment
```

The client knows only the name.

---

# Implementation

```csharp
public interface IPayment
{
    void Pay();
}
```

---

```csharp
public class StripePayment : IPayment
{
    public void Pay()
    {
        Console.WriteLine("Stripe");
    }
}
```

---

```csharp
public class PayPalPayment : IPayment
{
    public void Pay()
    {
        Console.WriteLine("PayPal");
    }
}
```

---

## Named Factory

```csharp
public class PaymentFactory
{
    private readonly Dictionary<
        string,
        IPayment> _payments;

    public PaymentFactory()
    {
        _payments = new()
        {
            ["Stripe"] = new StripePayment(),

            ["PayPal"] = new PayPalPayment()
        };
    }

    public IPayment Get(string name)
    {
        return _payments[name];
    }
}
```

---

## Usage

```csharp
var factory = new PaymentFactory();

IPayment payment =
    factory.Get("Stripe");

payment.Pay();
```

---

# ASP.NET Core Example

Suppose multiple implementations are registered.

```csharp
builder.Services.AddScoped<
    IPayment,
    StripePayment>();

builder.Services.AddScoped<
    IPayment,
    PayPalPayment>();
```

A named factory can receive:

```csharp
IEnumerable<IPayment>
```

and choose the appropriate implementation based on a key.

```csharp
public class PaymentFactory
{
    private readonly Dictionary<string, IPayment> _map;

    public PaymentFactory(IEnumerable<IPayment> payments)
    {
        _map = payments.ToDictionary(
            p => p.GetType().Name.Replace("Payment", ""),
            p => p);
    }

    public IPayment Get(string name)
    {
        return _map[name];
    }
}
```

This approach is more extensible than using large `switch` statements.

---

# Static Factory vs Generic Factory vs Parameterized Factory vs Named Factory

| Feature | Static Factory | Generic Factory | Parameterized Factory | Named Factory |
|---------|----------------|-----------------|----------------------|---------------|
| Factory Instance Required | No | Usually Yes | Optional | Usually Yes |
| Uses Parameters | Optional | Generic Type | Runtime Parameter | Name/Key |
| Runtime Selection | Limited | Compile-time | Yes | Yes |
| Type Safety | High | Very High | Depends | Depends |
| Best Use Case | Utility creation | Generic reusable creation | Runtime selection | Plugin/provider lookup |

---

# When to Use Each

### Static Factory

Use when:

- Creation logic is simple.
- Factory has no state.
- Utility-style API is preferred.

Examples:

- `Guid.NewGuid()`
- `DateTime.UtcNow`
- Value object creation

---

### Generic Factory

Use when:

- Multiple object types follow the same creation process.
- You want reusable factory logic.
- Generic constraints fit the domain.

Examples:

- Generic repositories
- Generic DTO builders
- Test object factories

---

### Parameterized Factory

Use when:

- Runtime input determines the implementation.
- There are a manageable number of options.

Examples:

- Notification type
- Payment gateway
- File parser

---

### Named Factory

Use when:

- Objects are identified by a unique key.
- Supporting plugins or multiple providers.
- Working with DI and multiple implementations.

Examples:

- Payment providers
- AI model selection
- Cloud providers
- Export formats

---

# Interview Questions

### Basic

1. What is a Static Factory?
2. What is a Generic Factory?
3. What is a Parameterized Factory?
4. What is a Named Factory?

### Intermediate

5. Why is `Guid.NewGuid()` considered a static factory?
6. When should you use a Generic Factory?
7. Why are enums preferred over strings in Parameterized Factories?

### Advanced

8. How would you implement a Named Factory using ASP.NET Core Dependency Injection?
9. What are the trade-offs between Parameterized and Named Factories?
10. Which factory variation would you use for a payment gateway supporting multiple providers and why?

---

# Summary

Static, Generic, Parameterized, and Named Factories are practical variations of the Factory Pattern, each solving a different problem. A **Static Factory** exposes static creation methods and is ideal for simple utility-style object creation. A **Generic Factory** leverages C# generics to eliminate duplicate factory code for multiple types. A **Parameterized Factory** selects an implementation based on runtime input, while a **Named Factory** maps keys to implementations and is particularly useful for plugin architectures and applications with multiple registered services. Choosing the right variation depends on whether the emphasis is on simplicity, reuse, runtime selection, or extensibility.