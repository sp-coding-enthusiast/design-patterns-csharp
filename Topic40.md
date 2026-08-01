# 101. Observer Pattern in .NET

## Introduction

The **Observer Pattern** is one of the most commonly used design patterns in the .NET ecosystem.

.NET provides multiple built-in implementations of the Observer Pattern, including:

- Events and Delegates
- `EventHandler<TEventArgs>`
- `IObservable<T>` / `IObserver<T>`
- Reactive Extensions (Rx.NET)
- UI Events (WinForms, WPF, MAUI)
- FileSystemWatcher
- Timers
- SignalR
- MediatR Notifications (application-level implementation)

> **Whenever one object notifies multiple interested objects about a change, .NET is typically using the Observer Pattern.**

---

# Observer Pattern Structure

```
          Subject

       (Publisher)

             │

      Notify Observers

             │

    ┌────────┼─────────┐

    ▼        ▼         ▼

 Email   Inventory   Analytics

 (Observers)
```

---

# Observer Using Events

The simplest implementation in .NET uses **events**.

---

## Publisher

```csharp
public class OrderService
{
    public event Action<string>? OrderCreated;

    public void CreateOrder()
    {
        Console.WriteLine("Order Created");

        OrderCreated?.Invoke("Order #1001");
    }
}
```

---

## Subscriber 1

```csharp
public class EmailService
{
    public void SendEmail(string order)
    {
        Console.WriteLine(
            $"Email Sent : {order}");
    }
}
```

---

## Subscriber 2

```csharp
public class InventoryService
{
    public void UpdateInventory(string order)
    {
        Console.WriteLine(
            $"Inventory Updated : {order}");
    }
}
```

---

## Client

```csharp
var orderService =
    new OrderService();

var email =
    new EmailService();

var inventory =
    new InventoryService();

orderService.OrderCreated +=
    email.SendEmail;

orderService.OrderCreated +=
    inventory.UpdateInventory;

orderService.CreateOrder();
```

---

## Output

```
Order Created

Email Sent : Order #1001

Inventory Updated : Order #1001
```

---

# Internal Workflow

```
CreateOrder()

↓

Raise Event

↓

All Subscribers

↓

Execute Handlers
```

---

# Using EventHandler<T>

Although `Action<T>` works, the recommended .NET approach is `EventHandler<TEventArgs>`.

---

## Event Arguments

```csharp
public class OrderCreatedEventArgs
    : EventArgs
{
    public int OrderId
    {
        get;
        set;
    }

    public decimal Amount
    {
        get;
        set;
    }
}
```

---

## Publisher

```csharp
public class OrderService
{
    public event EventHandler<
        OrderCreatedEventArgs>?
        OrderCreated;

    public void CreateOrder()
    {
        OrderCreated?.Invoke(
            this,
            new OrderCreatedEventArgs
            {
                OrderId = 101,
                Amount = 2500
            });
    }
}
```

---

## Subscriber

```csharp
service.OrderCreated +=
(sender, e) =>
{
    Console.WriteLine(
        $"Order {e.OrderId}");

    Console.WriteLine(
        $"Amount {e.Amount}");
};
```

---

## Output

```
Order 101

Amount 2500
```

---

# Why EventHandler<T>?

Advantages:

- Standard .NET convention
- Strongly typed event data
- Includes sender information
- Widely supported across frameworks

---

# Observer Using Delegates

Events are built on delegates.

Example:

```csharp
public delegate void Notify(
    string message);
```

Publisher:

```csharp
public Notify NotifyObservers;
```

However, using **events** is preferred because subscribers cannot invoke the event directly.

---

# IObservable<T> and IObserver<T>

.NET also provides a formal Observer implementation.

---

## Publisher

```csharp
public class TemperatureSensor
    : IObservable<int>
{
}
```

---

## Subscriber

```csharp
public class Display
    : IObserver<int>
{
}
```

---

# Architecture

```
Temperature Sensor

↓

Subscribe()

↓

Display

↓

Mobile App

↓

Logger
```

Multiple observers receive updates.

---

# Subscription Flow

```
Observer

↓

Subscribe()

↓

Observable

↓

Receive Updates

↓

Dispose()
```

The `Subscribe()` method returns an `IDisposable`, allowing observers to unsubscribe cleanly.

---

# Reactive Extensions (Rx.NET)

`IObservable<T>` is the foundation of **Reactive Extensions (Rx.NET)**.

Example:

```csharp
IObservable<int> numbers =
    Observable.Range(1, 5);

numbers.Subscribe(
    x => Console.WriteLine(x));
```

Output

```
1
2
3
4
5
```

Rx enables powerful event composition, filtering, throttling, and asynchronous stream processing.

---

# UI Framework Examples

## Windows Forms

```csharp
button.Click += Button_Click;
```

---

## WPF

```csharp
button.Click += Button_Click;
```

---

## .NET MAUI

```csharp
button.Clicked += OnClicked;
```

All of these use the Observer Pattern.

---

# FileSystemWatcher

```csharp
var watcher =
    new FileSystemWatcher();

watcher.Created +=
    OnFileCreated;
```

Whenever a file is created:

```
File Created

↓

Watcher

↓

Subscribers
```

---

# Timer Example

```csharp
var timer =
    new System.Timers.Timer(1000);

timer.Elapsed +=
    TimerElapsed;

timer.Start();
```

Every second:

```
Timer

↓

Elapsed Event

↓

Subscribers
```

---

# SignalR

SignalR also follows an observer-style model.

Server:

```csharp
await Clients.All.SendAsync(
    "OrderCreated");
```

Clients subscribe:

```javascript
connection.on(
    "OrderCreated",
    handler);
```

Workflow:

```
Server

↓

SignalR Hub

↓

Connected Clients
```

---

# MediatR Notifications

A common application-layer implementation of Observer.

Notification:

```csharp
public class OrderCreatedNotification
    : INotification
{
    public int OrderId { get; set; }
}
```

Handlers:

```text
EmailHandler

InventoryHandler

AnalyticsHandler
```

Publisher:

```csharp
await mediator.Publish(
    new OrderCreatedNotification
    {
        OrderId = 100
    });
```

All registered handlers execute.

---

# Enterprise Example

Suppose an order is placed.

```
Order Service

↓

Raise Event

↓

Email Service

↓

Inventory Service

↓

Analytics Service

↓

Loyalty Service
```

Adding another observer requires **no changes** to the `OrderService`.

---

# Advantages

- Loose coupling
- Easy to extend
- Supports Open/Closed Principle
- One publisher, many subscribers
- Built into .NET

---

# Disadvantages

- Harder to trace event flow
- Possible memory leaks if observers never unsubscribe
- Long-running synchronous handlers can delay other observers
- Exception handling requires care

---

# Best Practices

- Prefer `EventHandler<TEventArgs>` for custom events.
- Unsubscribe from events when objects have shorter lifetimes to avoid memory leaks.
- Keep event handlers lightweight.
- Use asynchronous messaging or background processing for expensive work.
- Use `IObservable<T>` and Rx.NET for complex event streams.

---

# Comparison of .NET Observer Implementations

| Mechanism | Typical Use | Supports Multiple Observers |
|-----------|-------------|-----------------------------|
| Events | General application events | Yes |
| EventHandler<T> | Standard .NET events | Yes |
| Delegates | Simple callbacks | Yes (multicast) |
| IObservable<T> | Reactive programming | Yes |
| Rx.NET | Complex event processing | Yes |
| MediatR Notifications | Application events | Yes |
| SignalR | Real-time client notifications | Yes |

---

# Interview Questions

### Basic

1. How is the Observer Pattern implemented in .NET?
2. Why are events preferred over plain delegates?
3. What is `EventHandler<TEventArgs>`?

### Intermediate

4. What is the difference between events and delegates?
5. What are `IObservable<T>` and `IObserver<T>`?
6. How does `Subscribe()` work?

### Advanced

7. How does Rx.NET extend the Observer Pattern?
8. Why can event subscriptions cause memory leaks?
9. When would you use MediatR instead of .NET events?
10. How would you build a scalable order notification system using the Observer Pattern?

---

# Summary

The **Observer Pattern** is deeply embedded in the .NET platform. It is implemented through **events**, **delegates**, **`EventHandler<TEventArgs>`**, and the **`IObservable<T>` / `IObserver<T>`** interfaces. Technologies such as **Windows Forms**, **WPF**, **.NET MAUI**, **FileSystemWatcher**, **SignalR**, **Rx.NET**, and **MediatR** all rely on the same core concept of notifying multiple subscribers when an event occurs. Understanding these implementations is essential for designing loosely coupled, event-driven, and highly extensible .NET applications.