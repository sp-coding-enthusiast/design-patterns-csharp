# 99. Observer Pattern

## Introduction

The **Observer Pattern** is a **Behavioral Design Pattern** that defines a **one-to-many dependency** between objects.

When one object (called the **Subject**) changes its state, all its dependent objects (called **Observers**) are automatically notified and updated.

> **Observer enables automatic notification when an object's state changes.**

---

# Definition

> **Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified automatically.**

---

# Why Do We Need Observer?

Imagine an e-commerce website.

When an order is placed:

- Send Email
- Send SMS
- Update Inventory
- Generate Invoice
- Notify Warehouse
- Update Analytics

Without Observer:

```csharp
public void PlaceOrder()
{
    SaveOrder();

    SendEmail();

    SendSMS();

    UpdateInventory();

    GenerateInvoice();

    NotifyWarehouse();

    UpdateAnalytics();
}
```

Problems:

- Tight coupling
- Difficult to add new notifications
- Violates Open/Closed Principle

---

# With Observer

```
Order Service

↓

Notify

↓

Email

SMS

Inventory

Analytics

Warehouse
```

The order service only publishes an event.

Observers decide what to do.

---

# Structure

```
             Subject

             ▲

      Attach()

      Detach()

      Notify()

             │

      ┌──────┴──────┐

      ▼             ▼

 Observer      Observer
```

---

# Components

| Component | Responsibility |
|------------|----------------|
| Subject | Maintains observers |
| Observer | Receives updates |
| Concrete Subject | Publishes notifications |
| Concrete Observer | Handles notifications |

---

# Example

## Observer Interface

```csharp
public interface IObserver
{
    void Update(string message);
}
```

---

## Concrete Observer

```csharp
public class EmailObserver
    : IObserver
{
    public void Update(string message)
    {
        Console.WriteLine(
            $"Email: {message}");
    }
}
```

---

## Another Observer

```csharp
public class SmsObserver
    : IObserver
{
    public void Update(string message)
    {
        Console.WriteLine(
            $"SMS: {message}");
    }
}
```

---

## Subject

```csharp
public class OrderService
{
    private readonly List<IObserver>
        _observers = new();

    public void Subscribe(
        IObserver observer)
    {
        _observers.Add(observer);
    }

    public void Notify(
        string message)
    {
        foreach(var observer in _observers)
        {
            observer.Update(message);
        }
    }
}
```

---

## Client

```csharp
var orderService =
    new OrderService();

orderService.Subscribe(
    new EmailObserver());

orderService.Subscribe(
    new SmsObserver());

orderService.Notify(
    "Order Created");
```

Output

```
Email: Order Created

SMS: Order Created
```

---

# Internal Workflow

```
Order Created

↓

Subject

↓

Notify()

↓

Email

↓

SMS

↓

Analytics
```

---

# Real-World Examples

- Stock market updates
- Weather applications
- Chat applications
- Social media notifications
- Order processing
- Event-driven systems
- GUI button click events

---

# Advantages

- Loose coupling
- Easy to add observers
- Supports Open/Closed Principle
- Dynamic subscriptions
- Excellent for event-driven systems

---

# Disadvantages

- Notification order may matter
- Too many observers can impact performance
- Debugging event chains can be difficult

---

# 100. Observer Pattern in .NET

The Observer Pattern is deeply integrated into .NET.

---

# .NET Events

The most common implementation is through **events**.

```csharp
public class OrderService
{
    public event Action<string>
        OrderCreated;

    public void CreateOrder()
    {
        OrderCreated?.Invoke(
            "Order Created");
    }
}
```

---

## Subscriber

```csharp
var service =
    new OrderService();

service.OrderCreated +=
    message =>
{
    Console.WriteLine(message);
};
```

Execution:

```
CreateOrder()

↓

Event Raised

↓

Subscribers
```

---

# EventHandler Pattern

A more common .NET pattern uses `EventHandler<TEventArgs>`.

```csharp
public class OrderEventArgs
    : EventArgs
{
    public int OrderId
    {
        get;
        set;
    }
}
```

Publisher:

```csharp
public class OrderService
{
    public event EventHandler<
        OrderEventArgs> OrderCreated;

    public void Create()
    {
        OrderCreated?.Invoke(
            this,
            new OrderEventArgs
            {
                OrderId = 10
            });
    }
}
```

Subscriber:

```csharp
service.OrderCreated +=
    (sender, args) =>
{
    Console.WriteLine(
        args.OrderId);
};
```

---

# IObservable<T> and IObserver<T>

.NET also provides built-in interfaces.

```csharp
IObservable<T>

↓

Subscribe()

↓

IObserver<T>
```

Example:

```csharp
public class TemperatureSensor
    : IObservable<int>
{
}
```

Subscribers implement:

```csharp
IObserver<int>
```

This model is commonly used in **Reactive Extensions (Rx.NET)**.

---

# Common .NET Examples

- Button Click
- Timer events
- FileSystemWatcher
- BackgroundWorker
- Reactive Extensions (Rx)
- SignalR events

---

# 101. Event-Driven Observer

## What is Event-Driven Architecture?

Instead of directly calling services:

```
Order

↓

Email

↓

Inventory

↓

Shipping
```

the application publishes an event.

```
Order Created

↓

Event Bus

↓

Email

Inventory

Shipping

Analytics
```

Every consumer reacts independently.

---

# Event Flow

```
Order Service

↓

Publish Event

↓

Event Bus

↓

Consumers
```

The publisher knows nothing about the consumers.

---

# Example

Suppose an order is placed.

Publisher:

```csharp
OrderCreated
```

Consumers:

```
Email Service

Inventory Service

Shipping Service

Analytics Service

Loyalty Service
```

Each service processes the event independently.

---

# ASP.NET Core Example

Controller:

```csharp
await _publisher.Publish(
    new OrderCreatedEvent());
```

Handlers:

```text
OrderCreatedHandler

InventoryHandler

EmailHandler

ShippingHandler
```

This is commonly implemented using libraries such as **MediatR** for in-process notifications or a message broker for distributed systems.

---

# Message Broker Example

```
Order Service

↓

RabbitMQ

↓

Email Service

Inventory Service

Notification Service
```

Popular brokers:

- RabbitMQ
- Apache Kafka
- Azure Service Bus
- AWS SQS/SNS
- Google Pub/Sub

---

# Event-Driven Microservices

```
Order Service

↓

Kafka Topic

↓

Inventory Service

↓

Notification Service

↓

Billing Service

↓

Analytics Service
```

Each service evolves independently.

---

# Observer vs Pub/Sub

They are related but not identical.

### Observer

```
Subject

↓

Observers
```

Usually in-memory and within the same process.

---

### Publish-Subscribe

```
Publisher

↓

Message Broker

↓

Subscribers
```

Often distributed across multiple processes or machines.

---

# Observer vs Event Bus

| Observer | Event Bus |
|-----------|-----------|
| In-process | Cross-process or distributed |
| Direct subscriptions | Broker-mediated |
| Low latency | Network latency |
| Suitable for applications | Suitable for distributed systems |

---

# Enterprise Examples

### Order Processing

```
Order Created

↓

Email

↓

Inventory

↓

Shipping

↓

Analytics
```

---

### Banking

```
Money Deposited

↓

Fraud Detection

↓

Notification

↓

Audit

↓

Reporting
```

---

### Healthcare

```
Patient Registered

↓

Billing

↓

Insurance

↓

Appointment

↓

Medical Record
```

---

# Best Practices

- Keep observers independent.
- Avoid heavy processing inside synchronous event handlers.
- Use asynchronous messaging for long-running work.
- Handle failures so one observer doesn't block others.
- Prevent memory leaks by unsubscribing when appropriate.
- Make event handlers idempotent in distributed systems.

---

# Observer vs Mediator

| Observer | Mediator |
|-----------|----------|
| One publisher, many observers | Central object coordinates interactions |
| Broadcast notifications | Controls communication |
| Loose coupling | Centralized coordination |

---

# Observer vs Strategy

| Observer | Strategy |
|-----------|----------|
| Notifies multiple listeners | Selects one algorithm |
| Event-driven | Algorithm-driven |
| One-to-many | One active implementation |

---

# Interview Questions

### Basic

1. What is the Observer Pattern?
2. What problem does it solve?
3. Give a real-world example.

### Intermediate

4. How is Observer implemented in .NET?
5. What is the difference between events and delegates?
6. What are `IObservable<T>` and `IObserver<T>`?

### Advanced

7. Observer vs Publish-Subscribe?
8. How would you implement event-driven order processing?
9. How does RabbitMQ differ from in-process events?
10. How would you prevent one failing observer from affecting the others?

---

# Summary

The **Observer Pattern** is a behavioral design pattern that establishes a one-to-many relationship between a subject and its observers, enabling automatic notifications when the subject's state changes. In .NET, the pattern is implemented through **events**, **delegates**, and the **`IObservable<T>` / `IObserver<T>`** interfaces, making it fundamental to UI programming, asynchronous processing, and reactive applications. In modern distributed architectures, the same concept evolves into **event-driven systems**, where events are published to message brokers such as RabbitMQ, Kafka, or Azure Service Bus, allowing multiple independent services to react without being tightly coupled to the publisher.