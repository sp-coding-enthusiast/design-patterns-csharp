# 31. Builder Pattern

## Introduction

The **Builder Pattern** is one of the **Gang of Four (GoF) Creational Design Patterns**.

It is used to **construct complex objects step by step**. Instead of passing many parameters to a constructor or creating multiple constructors, the Builder Pattern lets you build an object incrementally and then create the final object.

---

# Definition

> **Separate the construction of a complex object from its representation so that the same construction process can create different representations.**

In simple words:

> Build an object **step by step**, instead of creating it all at once.

---

# Why Do We Need Builder Pattern?

Imagine creating a `Laptop` object.

Without Builder:

```csharp
public class Laptop
{
    public Laptop(
        string cpu,
        string ram,
        string storage,
        string gpu,
        bool wifi,
        bool bluetooth,
        bool webcam,
        bool fingerprint)
    {
    }
}
```

Object creation:

```csharp
var laptop = new Laptop(
    "Intel i9",
    "32 GB",
    "1 TB SSD",
    "RTX 4070",
    true,
    true,
    true,
    false);
```

Problems:

- Constructor has too many parameters.
- Hard to remember parameter order.
- Difficult to read.
- Difficult to maintain.
- Easy to introduce bugs.

---

# Builder Pattern Solution

Instead of supplying everything at once:

```csharp
var laptop = new LaptopBuilder()
                .WithCpu("Intel i9")
                .WithRam("32 GB")
                .WithStorage("1 TB SSD")
                .WithGpu("RTX 4070")
                .EnableWifi()
                .EnableBluetooth()
                .Build();
```

Each step clearly describes what is being configured.

---

# Builder Pattern Structure

```
          Client
             │
             ▼
      LaptopBuilder
             │
             ▼
          Laptop
```

---

# Step 1: Product

```csharp
public class Laptop
{
    public string Cpu { get; set; }

    public string Ram { get; set; }

    public string Storage { get; set; }

    public bool Wifi { get; set; }
}
```

---

# Step 2: Builder

```csharp
public class LaptopBuilder
{
    private readonly Laptop _laptop = new();

    public LaptopBuilder WithCpu(string cpu)
    {
        _laptop.Cpu = cpu;
        return this;
    }

    public LaptopBuilder WithRam(string ram)
    {
        _laptop.Ram = ram;
        return this;
    }

    public LaptopBuilder WithStorage(string storage)
    {
        _laptop.Storage = storage;
        return this;
    }

    public LaptopBuilder EnableWifi()
    {
        _laptop.Wifi = true;
        return this;
    }

    public Laptop Build()
    {
        return _laptop;
    }
}
```

---

# Step 3: Client

```csharp
Laptop laptop =
    new LaptopBuilder()
        .WithCpu("Intel i9")
        .WithRam("32 GB")
        .WithStorage("1 TB SSD")
        .EnableWifi()
        .Build();
```

---

# Fluent Interface

The Builder Pattern often uses a **Fluent Interface**.

Example:

```csharp
builder
    .WithCpu("Intel")
    .WithRam("16 GB")
    .EnableWifi()
    .Build();
```

Each method returns the builder itself, allowing method chaining.

---

# Real-World Example

Imagine ordering a pizza.

You choose:

- Size
- Crust
- Cheese
- Toppings
- Sauce

The restaurant builds your pizza step by step before serving it.

This is exactly how the Builder Pattern works.

---

# ASP.NET Core Examples

## Example 1: Configuration Builder

```csharp
var configuration =
    new ConfigurationBuilder()
        .AddJsonFile("appsettings.json")
        .AddEnvironmentVariables()
        .Build();
```

The configuration object is built step by step.

---

## Example 2: Host Builder

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddSwaggerGen();

var app = builder.Build();
```

The application is configured in stages before being built.

---

## Example 3: HttpRequestMessage

```csharp
var request = new HttpRequestMessage
{
    Method = HttpMethod.Post,
    RequestUri = new Uri(url)
};
```

Many frameworks use builder-like APIs to configure complex objects before use.

---

# Advantages

- Eliminates constructors with many parameters.
- Improves readability.
- Supports optional parameters naturally.
- Easier maintenance.
- Supports immutable objects.
- Fluent API is intuitive.

---

# Disadvantages

- More classes.
- Additional abstraction.
- Unnecessary for simple objects.

---

# 32. Builder Pattern vs Factory Pattern

Although both are **Creational Design Patterns**, they solve different problems.

---

# Main Difference

**Factory Pattern**

Creates **which object** to instantiate.

**Builder Pattern**

Creates **how the object should be constructed**.

---

# Example

Factory:

```
Choose a Car

↓

BMW

Audi

Tesla
```

Builder:

```
Configure a BMW

↓

Color

Engine

Sunroof

Seats

Music System
```

Factory decides the type.

Builder decides the configuration.

---

# Visual Comparison

## Factory Pattern

```
             CarFactory

                 │

        ┌────────┼────────┐

        ▼        ▼        ▼

      BMW      Audi     Tesla
```

One object is created based on a choice.

---

## Builder Pattern

```
            CarBuilder

                 │

        Engine

        Color

        Wheels

        Seats

        Sunroof

                 │

                 ▼

              Car
```

The object is assembled step by step.

---

# Code Comparison

## Factory Pattern

```csharp
Car car = CarFactory.Create("BMW");
```

Simple and focused on selecting a concrete implementation.

---

## Builder Pattern

```csharp
Car car =
    new CarBuilder()
        .WithColor("Black")
        .WithEngine("Electric")
        .WithSunroof()
        .Build();
```

Focused on configuring a complex object.

---

# Real-World Analogy

## Factory

You walk into a showroom and say:

> "I want a Tesla."

The showroom hands you the selected model.

---

## Builder

You tell the showroom:

- Black color
- Leather seats
- Panoramic sunroof
- Premium sound system
- Performance package

The showroom builds the exact configuration you requested.

---

# ASP.NET Core Comparison

## Factory Example

```csharp
ILogger logger =
    LoggerFactory.Create(builder =>
    {
        builder.AddConsole();
    });
```

The factory creates the logger.

---

## Builder Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddAuthentication();

builder.Services.AddAuthorization();

var app = builder.Build();
```

The application is configured in multiple steps before being built.

---

# When to Use Builder

Use Builder when:

- The object has many optional properties.
- Constructors become difficult to read.
- You want a fluent API.
- You are creating immutable objects.
- Object construction is complex.

Examples:

- HTTP requests
- Configuration objects
- Reports
- SQL queries
- Complex DTOs

---

# When to Use Factory

Use Factory when:

- You need to choose one implementation.
- Object creation depends on runtime input.
- Clients should not know concrete classes.
- You want to centralize creation logic.

Examples:

- Payment gateways
- Notification services
- Storage providers
- Database providers

---

# Builder vs Factory

| Feature | Builder Pattern | Factory Pattern |
|---------|-----------------|-----------------|
| Category | Creational | Creational |
| Purpose | Construct complex objects | Create appropriate objects |
| Focus | Object configuration | Object selection |
| Object creation | Step by step | Usually one step |
| Fluent API | Common | Rare |
| Handles optional parameters | Excellent | Limited |
| Complexity | Medium | Low |
| Typical Use | Complex configuration | Runtime implementation selection |

---

# Builder vs Factory Method vs Abstract Factory

| Pattern | Creates | Best Use Case |
|---------|----------|---------------|
| **Factory (Simple Factory)** | One object | Select one implementation |
| **Factory Method** | One object using subclasses | Extensible object creation |
| **Abstract Factory** | Family of related objects | Platform or product family selection |
| **Builder** | One complex object | Step-by-step configuration |

---

# Interview Questions

### Basic

1. What is the Builder Pattern?
2. Why do we use the Builder Pattern?
3. What problem does it solve?

### Intermediate

4. How is Builder different from Factory?
5. What is a Fluent Interface?
6. Why is Builder useful for immutable objects?

### Advanced

7. Explain the Builder Pattern with an ASP.NET Core example.
8. When would you choose Builder over Factory?
9. Can Builder and Factory be used together?
10. What are the trade-offs of the Builder Pattern?

---

# Summary

The **Builder Pattern** is a creational design pattern used to **construct complex objects step by step**, making object creation more readable, flexible, and maintainable. It is especially useful when objects have many optional parameters or require staged configuration. In contrast, the **Factory Pattern** focuses on **deciding which object to create**, while the Builder Pattern focuses on **how that object is configured and assembled**. Both patterns complement each other and are widely used in enterprise applications and ASP.NET Core APIs.