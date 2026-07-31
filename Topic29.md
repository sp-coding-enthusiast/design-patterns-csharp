# 69. Bridge Pattern

## Introduction

The **Bridge Pattern** is a **Structural Design Pattern** that **decouples an abstraction from its implementation**, allowing both to evolve independently.

Instead of tightly coupling an abstraction with a specific implementation, Bridge separates them into two independent hierarchies connected through composition.

> **Bridge separates "what" from "how".**

---

# Definition

> **The Bridge Pattern separates an abstraction from its implementation so that both can vary independently.**

---

# Why Do We Need Bridge?

Imagine building a notification system.

You have multiple notification types:

- Alert
- Reminder
- Promotion

And multiple delivery channels:

- Email
- SMS
- Push Notification

Without Bridge:

```
AlertEmail

AlertSMS

AlertPush

ReminderEmail

ReminderSMS

ReminderPush

PromotionEmail

PromotionSMS

PromotionPush
```

If there are:

- 5 notification types
- 6 delivery channels

You end up with:

```
5 × 6 = 30 classes
```

This is known as **class explosion**.

---

# With Bridge

Separate the abstraction from the implementation.

```
Notification

↓

Email

SMS

Push
```

Now both hierarchies grow independently.

---

# Real-World Analogy

Think of a TV and a remote control.

```
Remote

↓

Sony TV

LG TV

Samsung TV
```

Different remotes can control different TVs because they communicate through a common interface.

The remote (**abstraction**) is independent of the TV implementation.

---

# Structure

```
        Abstraction

             │

             ▼

     Implementor Interface

        ▲            ▲

        │            │

 EmailSender    SmsSender
```

The abstraction contains a reference to the implementation instead of inheriting from it.

---

# Components

| Component | Responsibility |
|-----------|----------------|
| Abstraction | High-level functionality |
| Refined Abstraction | Extended abstraction |
| Implementor | Defines implementation interface |
| Concrete Implementor | Actual implementation |

---

# Example

## Step 1: Implementor

```csharp
public interface IMessageSender
{
    void Send(string message);
}
```

---

## Step 2: Concrete Implementations

### Email

```csharp
public class EmailSender
    : IMessageSender
{
    public void Send(string message)
    {
        Console.WriteLine(
            $"Email: {message}");
    }
}
```

---

### SMS

```csharp
public class SmsSender
    : IMessageSender
{
    public void Send(string message)
    {
        Console.WriteLine(
            $"SMS: {message}");
    }
}
```

---

## Step 3: Abstraction

```csharp
public abstract class Notification
{
    protected readonly IMessageSender
        _sender;

    protected Notification(
        IMessageSender sender)
    {
        _sender = sender;
    }

    public abstract void Notify(
        string message);
}
```

---

## Step 4: Refined Abstraction

```csharp
public class AlertNotification
    : Notification
{
    public AlertNotification(
        IMessageSender sender)
        : base(sender)
    {
    }

    public override void Notify(
        string message)
    {
        _sender.Send(
            $"Alert: {message}");
    }
}
```

---

## Client

```csharp
Notification notification =
    new AlertNotification(
        new EmailSender());

notification.Notify(
    "Server Down");
```

Output

```
Email: Alert: Server Down
```

Changing to SMS requires only a different implementation.

```csharp
Notification notification =
    new AlertNotification(
        new SmsSender());
```

---

# Internal Workflow

```
Client

↓

Notification

↓

Message Sender

↓

Email
```

The abstraction delegates the actual work.

---

# How Bridge Prevents Class Explosion

Without Bridge

```
AlertEmail

AlertSMS

AlertPush

ReminderEmail

ReminderSMS

ReminderPush

PromotionEmail

PromotionSMS

PromotionPush
```

Nine classes.

---

With Bridge

```
Notifications

↓

Alert

Reminder

Promotion

+

Senders

↓

Email

SMS

Push
```

Only:

```
3 + 3 = 6 classes
```

instead of nine.

As the number of variations grows, the savings become even greater.

---

# ASP.NET Core Example

Suppose an application supports multiple storage providers.

Storage types:

- Image Storage
- Document Storage

Storage providers:

- Azure Blob
- Amazon S3
- Local Disk

Without Bridge:

```
ImageAzureStorage

ImageS3Storage

ImageLocalStorage

DocumentAzureStorage

DocumentS3Storage

DocumentLocalStorage
```

With Bridge:

```
Storage Service

↓

Azure Provider

↓

Amazon Provider

↓

Local Provider
```

The abstraction and implementation evolve independently.

---

# Advantages

- Prevents class explosion
- Promotes composition over inheritance
- Independent extensibility
- Better maintainability
- Supports the Open/Closed Principle

---

# Disadvantages

- More classes
- Slightly higher complexity
- Overkill for very small applications

---

# Common Uses

- Notification systems
- Payment providers
- Cloud storage providers
- Rendering engines
- Device drivers
- Database providers
- Reporting systems

---

# 70. Bridge Pattern vs Adapter

## Introduction

Bridge and Adapter are often confused because both use **composition**.

However, they solve completely different design problems.

---

# Core Difference

### Adapter

Problem:

```
Existing interfaces
don't match.
```

Solution:

```
Convert Interface
```

---

### Bridge

Problem:

```
Abstraction and implementation
are tightly coupled.
```

Solution:

```
Separate Hierarchies
```

---

# Intent

## Adapter

```
Old API

↓

Adapter

↓

New API
```

Goal:

```
Compatibility
```

---

## Bridge

```
Abstraction

↓

Implementation
```

Goal:

```
Independent evolution
```

---

# Visual Comparison

## Adapter

```
Client

↓

Adapter

↓

Legacy Class
```

---

## Bridge

```
Client

↓

Abstraction

↓

Implementation
```

---

# Example

### Adapter

Application expects:

```csharp
ProcessPayment()
```

Third-party library provides:

```csharp
MakePayment()
```

Adapter converts:

```
ProcessPayment()

↓

MakePayment()
```

---

### Bridge

Notification types:

```
Alert

Reminder
```

Message senders:

```
Email

SMS
```

Bridge allows any notification to work with any sender.

---

# Real-World Analogy

## Adapter

Traveling abroad.

```
Indian Charger

↓

Travel Adapter

↓

US Power Socket
```

The adapter makes incompatible connectors work together.

---

## Bridge

A television and a remote.

```
Remote

↓

Sony TV

LG TV

Samsung TV
```

The remote and TV evolve independently as long as they follow the agreed interface.

---

# ASP.NET Core Example

### Adapter

```
Application

↓

Storage Adapter

↓

Azure Blob SDK
```

Purpose:

```
Translate interfaces
```

---

### Bridge

```
Document Storage

↓

Azure Provider

↓

Amazon Provider
```

Purpose:

```
Separate abstraction
from implementation
```

---

# Can They Be Used Together?

Yes.

Example:

```
Controller

↓

Storage Service
(Bridge)

↓

Azure Storage Adapter

↓

Azure SDK
```

The Bridge separates the application's storage abstraction from storage implementations, while the Adapter translates a cloud provider's SDK into the application's expected interface.

---

# Bridge vs Adapter Comparison

| Feature | Bridge | Adapter |
|----------|---------|---------|
| Design Pattern Type | Structural | Structural |
| Primary Goal | Separate abstraction and implementation | Make incompatible interfaces compatible |
| Changes Existing Interface | No | Yes |
| Prevents Class Explosion | Yes | No |
| Usually Planned During Design | Yes | Often introduced later |
| Works With | Parallel hierarchies | Existing classes |
| Focus | Flexibility | Compatibility |

---

# Bridge vs Strategy

| Bridge | Strategy |
|----------|----------|
| Separates abstraction from implementation | Encapsulates interchangeable algorithms |
| Two independent hierarchies | One family of algorithms |
| Structural Pattern | Behavioral Pattern |

Example:

- **Bridge:** Notification → Email/SMS/Push
- **Strategy:** Payment calculation → Credit Card/UPI/Wallet algorithms

---

# Best Practices

- Use Bridge when two dimensions of variation can change independently.
- Favor composition instead of inheritance.
- Design the implementation interface to be stable and focused.
- Avoid Bridge for simple scenarios with only one implementation.
- Combine Bridge with Dependency Injection for runtime selection of implementations.

---

# Interview Questions

### Basic

1. What is the Bridge Pattern?
2. What problem does the Bridge Pattern solve?
3. Why does Bridge use composition?

### Intermediate

4. How does Bridge prevent class explosion?
5. Bridge vs Adapter?
6. Bridge vs Strategy?

### Advanced

7. How would you design a cloud storage system using Bridge?
8. How is the Bridge Pattern useful in enterprise applications?
9. Can Bridge and Adapter be used together?
10. Why is Bridge considered a design-time pattern while Adapter is often a retrofit pattern?

---

# Summary

The **Bridge Pattern** is a structural design pattern that separates an abstraction from its implementation, allowing both to evolve independently through composition. It is especially useful when a system has multiple dimensions of variation, such as notification types and delivery channels or storage services and cloud providers. By preventing class explosion and reducing tight coupling, Bridge improves flexibility and maintainability. In contrast, the **Adapter Pattern** focuses on making incompatible interfaces work together, typically when integrating existing or third-party components. Bridge is generally applied during system design, whereas Adapter is commonly introduced later to integrate incompatible APIs without modifying existing code.