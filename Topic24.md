# 59. Prototype Pattern - Practical Use Cases

## Introduction

Many developers understand the **Prototype Pattern** theoretically but rarely recognize where it is actually used in production applications.

The Prototype Pattern is useful whenever:

- Creating an object is expensive.
- Similar objects are created repeatedly.
- Existing objects act as templates.
- Deep copying is preferred over rebuilding.

---

# Practical Use Case 1: Document Templates

Suppose Microsoft Word has predefined templates.

```
Invoice Template

↓

Clone

↓

Fill Customer Details

↓

Generate Invoice
```

Instead of creating a document from scratch every time, the template is cloned.

---

### Example

```csharp
Document invoice =
    invoiceTemplate.Clone();

invoice.CustomerName = "John";
```

The original template remains unchanged.

---

# Practical Use Case 2: AI Prompt Templates

Consider an enterprise AI application.

Base Prompt

```text
You are an AI assistant.

Always answer politely.

Respond in JSON.
```

Instead of rebuilding the prompt:

```
Base Prompt

↓

Clone

↓

Append User Query

↓

Send to LLM
```

Example:

```csharp
Prompt prompt =
    basePrompt.Clone();

prompt.UserQuestion =
    "Explain SOLID";
```

This avoids modifying the original prompt.

---

# Practical Use Case 3: Game Development

Games create thousands of similar objects.

Example:

```
Enemy Template

↓

Clone

↓

Position

↓

Health

↓

Spawn
```

Instead of repeatedly loading textures and configuration, a prototype is cloned.

---

### Example

```csharp
Enemy enemy =
    enemyPrototype.Clone();

enemy.Position = new Position(20, 40);
```

---

# Practical Use Case 4: Database Entity Templates

Suppose an HR application creates employees with common defaults.

```
Default Employee

↓

Clone

↓

Change Name

↓

Save
```

```csharp
Employee employee =
    employeeTemplate.Clone();

employee.Name = "Alice";
```

---

# Practical Use Case 5: Report Generation

```
Monthly Report Template

↓

Clone

↓

Insert Sales Data

↓

Generate PDF
```

Every report begins with the same layout.

---

# Practical Use Case 6: Email Templates

```
Welcome Email

↓

Clone

↓

Replace Name

↓

Send
```

Instead of constructing each email from scratch.

---

# Practical Use Case 7: Workflow Definitions

Enterprise workflow systems often maintain templates.

```
Workflow Template

↓

Clone

↓

Modify Approvers

↓

Execute
```

The template remains reusable.

---

# Practical Use Case 8: Machine Learning Pipelines

Suppose a pipeline has:

```
Tokenizer

↓

Embedding

↓

Classifier
```

Instead of rebuilding the pipeline:

```
Pipeline Template

↓

Clone

↓

Change Model

↓

Run
```

---

# Practical Use Case 9: UI Components

Imagine a dashboard.

```
Chart Template

↓

Clone

↓

Update Title

↓

Update Data

↓

Render
```

Many dashboards use the same base component.

---

# Practical Use Case 10: Cloud Infrastructure

Infrastructure-as-Code templates work similarly.

```
VM Template

↓

Clone

↓

Modify CPU

↓

Deploy
```

Only the differing properties change.

---

# Prototype in ASP.NET Core

Prototype is less common than Factory or Dependency Injection but appears in:

- Email templates
- PDF templates
- Report templates
- AI prompt templates
- Workflow definitions
- DTO cloning
- Configuration templates
- Object snapshots

---

# Prototype vs Copy Constructor

Prototype:

```csharp
Employee clone =
    employee.Clone();
```

Copy Constructor:

```csharp
Employee clone =
    new Employee(employee);
```

Both duplicate objects, but Prototype emphasizes cloning as a **creational pattern**, while copy constructors are a language-level technique.

---

# Advantages

- Fast object creation
- Reuses expensive initialization
- Simplifies object creation
- Reduces duplicate setup
- Supports template-based design

---

# Drawbacks

- Deep copying can be complex
- Circular references require care
- Shared references may introduce bugs if only shallow copies are made
- Clone logic must evolve with the object

---

# 60. Creational Pattern Comparison

## Introduction

Creational Patterns solve one fundamental question:

> **How should objects be created?**

Each pattern addresses a different object creation problem.

---

# All Creational Patterns

The Gang of Four (GoF) defines **five** creational design patterns.

```
Creational Patterns

↓

Factory Method

↓

Abstract Factory

↓

Builder

↓

Prototype

↓

Singleton
```

Modern .NET applications also commonly use:

- Dependency Injection
- Object Pool
- Static Factory
- Generic Factory

These are widely adopted patterns but are not part of the original GoF five.

---

# Comparison Overview

| Pattern | Purpose | Creates | Best For |
|----------|---------|----------|-----------|
| Factory Method | Decide which object to create | One object | Runtime polymorphism |
| Abstract Factory | Create related object families | Multiple related objects | Cross-platform/product families |
| Builder | Build complex objects step by step | One complex object | Many optional configurations |
| Prototype | Copy an existing object | Clone | Expensive object creation |
| Singleton | Ensure one instance | One shared object | Shared infrastructure |
| Object Pool | Reuse objects | Many reusable objects | Performance optimization |
| Dependency Injection | Supply dependencies | Object graphs | Enterprise applications |

---

# Factory Method

```
Client

↓

Factory

↓

Car

OR

Bike
```

Use when:

- One object
- Runtime decision
- Polymorphism

Example:

```csharp
vehicleFactory.Create();
```

---

# Abstract Factory

```
Windows Factory

↓

Button

Textbox

Checkbox
```

Creates related families.

Example:

```csharp
factory.CreateButton();

factory.CreateTextbox();
```

---

# Builder

```
Builder

↓

CPU

↓

RAM

↓

Storage

↓

Build()
```

Creates one complex object.

Example:

```csharp
Computer computer =
    builder
        .SetCpu("i9")
        .SetRam(32)
        .Build();
```

---

# Prototype

```
Existing Object

↓

Clone()

↓

New Object
```

Use when object initialization is expensive.

Example:

```csharp
Employee clone =
    employee.Clone();
```

---

# Singleton

```
Application

↓

One Instance
```

Example:

```csharp
Logger.Instance
```

or

```csharp
builder.Services.AddSingleton<
    ILogger,
    FileLogger>();
```

---

# Object Pool

```
Pool

↓

Borrow

↓

Return

↓

Reuse
```

Example:

```csharp
ObjectPool<StringBuilder>
```

---

# Dependency Injection

```
Container

↓

Resolve

↓

Inject

↓

Object
```

Example:

```csharp
builder.Services.AddScoped<
    IRepository,
    Repository>();
```

---

# Which Pattern Should You Choose?

## Need one shared object?

Use:

```
Singleton
```

Examples:

- Logging
- Configuration
- Cache managers

---

## Need one of many implementations?

Use:

```
Factory Method
```

Examples:

- Payment gateways
- Notifications
- Parsers

---

## Need a family of related objects?

Use:

```
Abstract Factory
```

Examples:

- UI toolkits
- Database providers
- Cross-platform controls

---

## Need many optional parameters?

Use:

```
Builder
```

Examples:

- Configuration
- Complex DTOs
- ASP.NET Core startup
- EF Core model configuration

---

## Need to duplicate expensive objects?

Use:

```
Prototype
```

Examples:

- Templates
- Reports
- AI prompts
- Game entities

---

## Need maximum performance?

Use:

```
Object Pool
```

Examples:

- Buffers
- Database connections
- StringBuilder
- Parsers

---

## Need loose coupling?

Use:

```
Dependency Injection
```

Examples:

- ASP.NET Core
- Microservices
- Enterprise systems

---

# Interview Comparison Table

| Question | Recommended Pattern |
|-----------|---------------------|
| One instance only | Singleton |
| Clone an object | Prototype |
| Step-by-step object construction | Builder |
| Select implementation at runtime | Factory Method |
| Create related object families | Abstract Factory |
| Reuse expensive objects | Object Pool |
| Inject dependencies automatically | Dependency Injection |

---

# How They Work Together

Enterprise applications often combine several patterns.

Example:

```
Application

↓

Dependency Injection

↓

Factory

↓

Prototype

↓

Object Pool

↓

Singleton Logger
```

Each pattern solves a different concern rather than competing with the others.

---

# ASP.NET Core Mapping

| ASP.NET Core Feature | Pattern |
|----------------------|---------|
| `WebApplicationBuilder` | Builder |
| `ConfigurationBuilder` | Builder |
| `ILoggerFactory` | Factory |
| `ILogger<T>` Infrastructure | Singleton-based infrastructure |
| `AddSingleton()` | Singleton |
| `AddScoped()` | Dependency Injection |
| `ObjectPool<T>` | Object Pool |
| `IHttpClientFactory` | Factory + Object Pool |
| EF Core `ModelBuilder` | Builder |

---

# Common Interview Questions

### Basic

1. What are the five GoF creational patterns?
2. When would you use Prototype?
3. What is the difference between Factory Method and Builder?

### Intermediate

4. Builder vs Prototype?
5. Singleton vs Object Pool?
6. Factory Method vs Abstract Factory?

### Advanced

7. Which creational patterns are most commonly used in ASP.NET Core?
8. Can multiple creational patterns be used together?
9. Why is Dependency Injection not considered a GoF creational pattern?
10. Design a payment processing system using Factory, DI, and Singleton.

---

# Summary

The **Prototype Pattern** is valuable in real-world applications where existing objects serve as templates for creating new ones. Common examples include document templates, AI prompt templates, workflow definitions, report generation, game entities, and reusable UI components. Among all creational patterns, each addresses a distinct object creation challenge: **Factory Method** selects implementations, **Abstract Factory** creates related families of objects, **Builder** constructs complex objects step by step, **Prototype** clones existing objects, **Singleton** ensures a single shared instance, **Object Pool** reuses expensive objects, and **Dependency Injection** assembles object graphs while promoting loose coupling. Understanding when to apply each pattern is a key skill for designing maintainable and scalable .NET applications.