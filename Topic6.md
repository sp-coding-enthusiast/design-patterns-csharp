# 14. Explain Dependency Inversion Principle (DIP)

## Definition

The **Dependency Inversion Principle (DIP)** is the fifth principle of SOLID.

It states:

> **"High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions."**
> — Robert C. Martin (Uncle Bob)

In simple words:

> **Depend on interfaces or abstractions, not on concrete implementations.**

---

# Understanding High-Level and Low-Level Modules

### High-Level Module

Contains the **business logic** of the application.

Example:

- OrderService
- PaymentService
- UserService

---

### Low-Level Module

Contains implementation details.

Example:

- SQL Repository
- MongoDB Repository
- Email Service
- SMS Service
- Azure Blob Storage

---

# Problem Without DIP

Suppose an order service directly depends on an email service.

```csharp
public class EmailService
{
    public void Send(string message)
    {
        Console.WriteLine("Email Sent");
    }
}
```

```csharp
public class OrderService
{
    private readonly EmailService _emailService = new EmailService();

    public void PlaceOrder()
    {
        Console.WriteLine("Order Created");

        _emailService.Send("Order Confirmation");
    }
}
```

---

## Problems

`OrderService` is tightly coupled to `EmailService`.

If the business decides to:

- Use SMS
- Use WhatsApp
- Use Azure Service Bus
- Use Push Notifications

`OrderService` must be modified.

This violates the Dependency Inversion Principle.

---

# Applying DIP

## Step 1: Create an Abstraction

```csharp
public interface IMessageService
{
    void Send(string message);
}
```

---

## Step 2: Implement the Abstraction

### Email Service

```csharp
public class EmailService : IMessageService
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}
```

---

### SMS Service

```csharp
public class SmsService : IMessageService
{
    public void Send(string message)
    {
        Console.WriteLine($"SMS: {message}");
    }
}
```

---

## Step 3: Depend on the Interface

```csharp
public class OrderService
{
    private readonly IMessageService _messageService;

    public OrderService(IMessageService messageService)
    {
        _messageService = messageService;
    }

    public void PlaceOrder()
    {
        Console.WriteLine("Order Created");

        _messageService.Send("Order Confirmation");
    }
}
```

---

## Usage

```csharp
IMessageService service = new EmailService();

OrderService orderService = new OrderService(service);

orderService.PlaceOrder();
```

To switch to SMS:

```csharp
IMessageService service = new SmsService();
```

No changes are required in `OrderService`.

---

# Before vs After DIP

| Without DIP | With DIP |
|--------------|----------|
| Depends on concrete class | Depends on interface |
| Tightly coupled | Loosely coupled |
| Difficult to test | Easy to mock |
| Hard to extend | Easy to extend |
| Changes affect business logic | Business logic remains unchanged |

---

# Real-World Analogy

Imagine a person using a **wall power socket**.

The person plugs in:

- Laptop charger
- Phone charger
- Television
- Fan

The person interacts with the **socket interface**, not the internal electrical wiring.

The socket is the abstraction.

Different appliances are implementations.

This is exactly how DIP works.

---

# Benefits of DIP

- Loose coupling
- Better maintainability
- Easier testing
- Improved scalability
- Easier replacement of implementations
- Supports Clean Architecture

---

# 15. DIP vs Dependency Injection (DI)

Many developers confuse **Dependency Inversion Principle (DIP)** and **Dependency Injection (DI)**.

They are related but **not the same**.

---

# Dependency Inversion Principle (DIP)

DIP is a **design principle**.

It answers:

> **How should classes depend on each other?**

Answer:

> Through abstractions (interfaces or abstract classes), not concrete implementations.

Example:

```csharp
public interface ILogger
{
    void Log(string message);
}
```

Business logic:

```csharp
public class UserService
{
    private readonly ILogger _logger;

    public UserService(ILogger logger)
    {
        _logger = logger;
    }
}
```

---

# Dependency Injection (DI)

Dependency Injection is a **design pattern** and a **technique**.

It answers:

> **How do we provide those dependencies?**

Instead of creating objects inside a class:

```csharp
private readonly EmailService service =
    new EmailService();
```

DI supplies them from outside.

Example:

```csharp
public UserService(ILogger logger)
{
    _logger = logger;
}
```

The object is injected rather than created internally.

---

# Comparison

| Dependency Inversion Principle (DIP) | Dependency Injection (DI) |
|--------------------------------------|----------------------------|
| SOLID design principle | Design pattern / implementation technique |
| Focuses on abstractions | Focuses on object creation and delivery |
| Reduces coupling | Supplies dependencies |
| Answers "What should we depend on?" | Answers "How do we get the dependency?" |
| Can exist without DI | Commonly used to implement DIP |

---

# Simple Analogy

Imagine ordering food.

### Without DI

You cook your own meal.

```
You
 ↓
Kitchen
```

You are responsible for preparing everything.

---

### With DI

A restaurant delivers the meal.

```
Restaurant
      ↓
You
```

You simply consume the meal.

Similarly, your class should consume dependencies rather than create them.

---

# 16. How Does Dependency Injection Implement DIP?

Dependency Injection is one of the most common ways to **implement the Dependency Inversion Principle**.

---

# Without DI

```csharp
public class NotificationService
{
    public void Send()
    {
    }
}

public class OrderService
{
    private NotificationService notification =
        new NotificationService();
}
```

Problems:

- Tight coupling
- Difficult to mock
- Difficult to replace
- Violates DIP

---

# With DI

Create an abstraction.

```csharp
public interface INotificationService
{
    void Send();
}
```

Implementation:

```csharp
public class EmailNotificationService
    : INotificationService
{
    public void Send()
    {
        Console.WriteLine("Email Sent");
    }
}
```

Consumer:

```csharp
public class OrderService
{
    private readonly INotificationService _notification;

    public OrderService(
        INotificationService notification)
    {
        _notification = notification;
    }

    public void PlaceOrder()
    {
        _notification.Send();
    }
}
```

Now `OrderService` depends on the interface instead of a concrete implementation.

---

# ASP.NET Core Dependency Injection

Register the service in `Program.cs`:

```csharp
builder.Services.AddScoped<
    INotificationService,
    EmailNotificationService>();
```

Controller:

```csharp
public class OrdersController : ControllerBase
{
    private readonly INotificationService _notification;

    public OrdersController(
        INotificationService notification)
    {
        _notification = notification;
    }
}
```

If the business later decides to use SMS:

```csharp
builder.Services.AddScoped<
    INotificationService,
    SmsNotificationService>();
```

No changes are needed in:

- `OrdersController`
- `OrderService`
- Business logic

Only the registration changes.

---

# How ASP.NET Core DI Container Works

```
          Request

             │
             ▼

     OrdersController
             │
             ▼

 INotificationService
             │
             ▼

 EmailNotificationService

             ▲

Registered in DI Container
```

The controller requests an abstraction, and the DI container provides the configured implementation.

---

# Benefits of DI + DIP

- Loose coupling
- Easy unit testing using mocks
- Runtime implementation switching
- Better maintainability
- Cleaner architecture
- Easier scalability
- Improved code reuse

---

# Interview Questions

### Basic

1. What is the Dependency Inversion Principle?
2. What are high-level and low-level modules?
3. Why should classes depend on abstractions?

### Intermediate

4. What is the difference between DIP and Dependency Injection?
5. Can DIP exist without Dependency Injection?
6. Why does constructor injection support DIP?

### Advanced

7. How does ASP.NET Core implement DIP?
8. Explain the DI container in ASP.NET Core.
9. Why is constructor injection preferred over property injection?
10. How do mocking frameworks such as Moq benefit from DIP?

---

# Summary

The **Dependency Inversion Principle (DIP)** states that high-level business logic should depend on **abstractions**, not concrete implementations. **Dependency Injection (DI)** is the design pattern used to supply those abstractions to dependent classes. In ASP.NET Core, the built-in DI container resolves interfaces to their registered implementations, allowing applications to remain loosely coupled, testable, and easy to extend without modifying business logic.