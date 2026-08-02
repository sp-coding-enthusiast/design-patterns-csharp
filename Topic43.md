# 105. Command Pattern

## Introduction

The **Command Pattern** is a **Behavioral Design Pattern** that encapsulates a request as an object.

Instead of calling a method directly, the request is wrapped inside a **Command object**, which can be executed, queued, logged, retried, or undone.

> **The Command Pattern turns a request into an object.**

---

# Definition

> **Encapsulate a request as an object, thereby allowing you to parameterize clients with different requests, queue or log requests, and support undoable operations.**

---

# Why Do We Need Command?

Suppose you have a remote control.

Without Command:

```csharp
public class RemoteControl
{
    public void PressButton()
    {
        television.TurnOn();
    }
}
```

Now support:

- TV
- Fan
- AC
- Music System
- Lights

The remote becomes tightly coupled to every device.

---

# Solution

Use Command.

```
Remote

↓

ICommand

↓

TV Command

Fan Command

AC Command
```

The remote only knows about `ICommand`.

---

# Components

| Component | Responsibility |
|------------|----------------|
| Command | Defines Execute() |
| Concrete Command | Implements request |
| Receiver | Performs the work |
| Invoker | Calls Execute() |
| Client | Creates command |

---

# UML Structure

```
             Client

               │

               ▼

          ICommand

               ▲

               │

      TurnOnTvCommand

               │

               ▼

           Television

Invoker

↓

Execute()
```

---

# Example

## Command

```csharp
public interface ICommand
{
    void Execute();
}
```

---

## Receiver

```csharp
public class Television
{
    public void TurnOn()
    {
        Console.WriteLine(
            "TV Turned On");
    }
}
```

---

## Concrete Command

```csharp
public class TurnOnTvCommand
    : ICommand
{
    private readonly Television _tv;

    public TurnOnTvCommand(
        Television tv)
    {
        _tv = tv;
    }

    public void Execute()
    {
        _tv.TurnOn();
    }
}
```

---

## Invoker

```csharp
public class RemoteControl
{
    private ICommand _command;

    public void SetCommand(
        ICommand command)
    {
        _command = command;
    }

    public void PressButton()
    {
        _command.Execute();
    }
}
```

---

## Client

```csharp
var tv = new Television();

var command =
    new TurnOnTvCommand(tv);

var remote =
    new RemoteControl();

remote.SetCommand(command);

remote.PressButton();
```

Output

```
TV Turned On
```

---

# Internal Workflow

```
Client

↓

Create Command

↓

Invoker

↓

Execute()

↓

Receiver
```

---

# Enterprise Example

Order Processing

```
Controller

↓

CreateOrderCommand

↓

Handler

↓

Repository

↓

Database
```

Notice:

The command contains the request.

The handler performs the work.

---

# Advantages

- Loose coupling
- Easy to queue requests
- Supports undo/redo
- Supports logging
- Supports retry
- Easy testing

---

# Disadvantages

- More classes
- Slightly more complex
- Overkill for trivial operations

---

# Real-World Examples

- MediatR Commands
- Azure Service Bus messages
- RabbitMQ messages
- Undo functionality
- Job schedulers
- Background workers

---

# 106. Command vs Strategy Pattern

Although both patterns encapsulate behavior, **their purpose is different**.

---

# Strategy

Purpose:

Choose **how** to perform an operation.

Example

```
Payment

↓

PayPal

Stripe

UPI
```

Different algorithms.

---

# Command

Purpose:

Represent **what** should be executed.

Example

```
Create Order

↓

Execute
```

The request itself becomes an object.

---

# Visual Comparison

## Strategy

```
Client

↓

Strategy

↓

Algorithm
```

---

## Command

```
Client

↓

Command

↓

Receiver
```

---

# Strategy Example

```csharp
payment.Pay();
```

Algorithm changes.

---

# Command Example

```csharp
command.Execute();
```

Request changes.

---

# Responsibility

### Strategy

```
Choose Best Algorithm
```

---

### Command

```
Package Work
```

---

# Internal Difference

## Strategy

```
Context

↓

Selected Strategy

↓

Execute Algorithm
```

---

## Command

```
Invoker

↓

Command

↓

Receiver

↓

Action
```

---

# Comparison Table

| Feature | Command | Strategy |
|----------|----------|----------|
| Purpose | Encapsulate a request | Encapsulate an algorithm |
| Focus | What to execute | How to execute |
| Supports Undo | Yes | No |
| Supports Queue | Yes | No |
| Supports Retry | Yes | No |
| Receiver | Yes | Usually No |
| Common Use | CQRS, Jobs | Payment, Validation, Compression |

---

# Enterprise Examples

### Strategy

```
Discount Strategy

↓

Festival

Member

Coupon
```

---

### Command

```
Create Invoice

↓

Queue

↓

Execute
```

---

# Interview Question

**Q:** Can Command and Strategy be used together?

**Answer:** Yes.

Example:

```
CreateOrderCommand

↓

Payment Strategy

↓

PayPal
```

The command represents the request to create an order, while the payment strategy decides **how** the payment is processed.

---

# 107. Command Queue

## What is a Command Queue?

One of the biggest strengths of the Command Pattern is that commands can be stored and executed later.

Instead of executing immediately:

```
Client

↓

Execute
```

they are placed into a queue.

```
Client

↓

Queue

↓

Worker

↓

Execute
```

---

# Why Queue Commands?

Imagine an e-commerce website receives

```
100,000 Orders
```

Processing immediately may overload the server.

Instead:

```
API

↓

Queue

↓

Background Worker

↓

Database
```

The API responds quickly while work continues in the background.

---

# Example

## Queue

```csharp
Queue<ICommand> queue =
    new Queue<ICommand>();
```

---

## Add Command

```csharp
queue.Enqueue(
    new CreateOrderCommand());
```

---

## Process Queue

```csharp
while(queue.Any())
{
    var command =
        queue.Dequeue();

    command.Execute();
}
```

---

# Internal Workflow

```
API

↓

Create Command

↓

Queue

↓

Background Worker

↓

Execute

↓

Database
```

---

# Enterprise Architecture

```
User

↓

Web API

↓

CreateOrderCommand

↓

RabbitMQ

↓

Worker Service

↓

Database
```

The command is serialized into a message and processed asynchronously.

---

# Azure Example

```
API

↓

Azure Service Bus

↓

Azure Function

↓

Execute Command
```

---

# Kafka Example

```
Producer

↓

Kafka Topic

↓

Consumer

↓

Execute Command
```

---

# Benefits

- Asynchronous processing
- Retry failed commands
- Better scalability
- Load leveling
- Fault tolerance
- Decoupled producers and consumers

---

# Retry Flow

```
Command

↓

Failed?

↓

Yes

↓

Retry Queue

↓

Execute Again
```

---

# Dead Letter Queue

If retries continue to fail:

```
Queue

↓

Retry

↓

Retry

↓

Dead Letter Queue
```

The failed command can be investigated without blocking other work.

---

# Command Queue vs Message Queue

| Command Queue | Message Queue |
|---------------|---------------|
| Usually contains executable commands | Contains messages/events |
| Often processed by a worker | Delivered to consumers |
| Represents an action | Represents information or an action |
| Frequently used with the Command Pattern | Infrastructure technology (RabbitMQ, Azure Service Bus, Kafka) |

---

# Command Pattern in ASP.NET Core

Example flow:

```
POST /orders

↓

Controller

↓

Mediator.Send()

↓

CreateOrderCommand

↓

Handler

↓

Repository
```

For asynchronous processing:

```
POST /orders

↓

Controller

↓

CreateOrderCommand

↓

Azure Service Bus

↓

Worker

↓

Handler

↓

Database
```

---

# Best Practices

- Keep commands immutable whenever possible.
- A command should represent a single business action.
- Do not place business logic inside the command object; keep it in the handler or receiver.
- Make command handlers idempotent when commands may be retried.
- Use queues for long-running or resource-intensive operations.

---

# Interview Questions

### Basic

1. What is the Command Pattern?
2. Why is it called "Command"?
3. What are the components of the Command Pattern?

### Intermediate

4. Command vs Strategy?
5. Why is Command useful for CQRS?
6. What is a Command Queue?

### Advanced

7. How would you implement undo/redo using the Command Pattern?
8. How would you process commands asynchronously with RabbitMQ or Azure Service Bus?
9. How would you make command handlers idempotent?
10. How does MediatR implement the Command Pattern?

---

# Summary

The **Command Pattern** encapsulates a request as an object, enabling requests to be executed, queued, retried, logged, or undone independently of the caller. In enterprise .NET applications, it is widely used with **CQRS**, **MediatR**, background workers, and messaging systems such as **RabbitMQ**, **Azure Service Bus**, and **Kafka**. Unlike the **Strategy Pattern**, which selects **how** an operation is performed, the **Command Pattern** represents **what** action should be executed. When combined with command queues, it forms the foundation of scalable and resilient asynchronous architectures.