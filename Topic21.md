# 51. Factory with Reflection

## Introduction

A **Reflection Factory** creates objects **dynamically at runtime** using .NET Reflection instead of directly calling constructors.

Instead of writing:

```csharp
var payment = new StripePayment();
```

The factory creates the object based on its **type name**.

```csharp
var payment =
    ReflectionFactory.Create("StripePayment");
```

This allows applications to create objects **without knowing their concrete types at compile time**.

---

# What is Reflection?

Reflection is a .NET feature that allows a program to:

- Inspect assemblies
- Inspect classes
- Discover methods
- Read properties
- Invoke methods
- Create objects dynamically

Example:

```csharp
Type type = typeof(string);

Console.WriteLine(type.Name);
```

Output

```
String
```

---

# Why Use Reflection Factory?

Imagine a plugin-based application.

Instead of:

```csharp
if(type == "Stripe")
    return new StripePayment();

if(type == "PayPal")
    return new PayPalPayment();
```

Use:

```
Configuration

↓

Reflection

↓

Create Object
```

New implementations can be added without modifying the factory.

---

# Structure

```
Configuration

↓

Reflection Factory

↓

Reflection API

↓

Concrete Class

↓

Object
```

---

# Example

## Interface

```csharp
public interface IPayment
{
    void Pay();
}
```

---

## Implementations

```csharp
public class StripePayment : IPayment
{
    public void Pay()
    {
        Console.WriteLine("Stripe");
    }
}
```

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

## Reflection Factory

```csharp
public static class ReflectionFactory
{
    public static object Create(
        string typeName)
    {
        Type? type =
            Type.GetType(typeName);

        if(type == null)
            throw new Exception(
                "Type not found");

        return Activator.CreateInstance(type)!;
    }
}
```

---

## Usage

```csharp
var payment =
    (IPayment)ReflectionFactory.Create(
        "MyApp.StripePayment");

payment.Pay();
```

Output

```
Stripe
```

---

# Using Assembly

If the type belongs to another assembly:

```csharp
Assembly assembly =
    Assembly.Load("PaymentLibrary");

Type type =
    assembly.GetType(
        "PaymentLibrary.StripePayment");

var payment =
    Activator.CreateInstance(type!);
```

---

# How It Works Internally

```
Type Name

↓

Reflection

↓

Locate Type

↓

Find Constructor

↓

Execute Constructor

↓

Return Object
```

---

# Activator.CreateInstance()

Reflection factories usually use:

```csharp
Activator.CreateInstance(type);
```

Internally:

```
Find Constructor

↓

Allocate Memory

↓

Call Constructor

↓

Return Object
```

---

# Generic Reflection Factory

```csharp
public static class Factory
{
    public static T Create<T>()
        where T : new()
    {
        return new T();
    }
}
```

or

```csharp
public static T Create<T>(Type type)
{
    return (T)Activator
        .CreateInstance(type)!;
}
```

---

# Real-World Example

Suppose an application supports multiple report formats.

Configuration:

```json
{
  "Report":
  "PdfReport"
}
```

Factory:

```
Read Configuration

↓

Reflection

↓

Create PdfReport
```

No code changes are required to support another implementation.

---

# Advantages

- Dynamic object creation
- Supports plugins
- No large switch statements
- Highly extensible
- Configuration-driven

---

# Disadvantages

- Slower than direct object creation
- Runtime errors if type names are incorrect
- Harder to debug
- Reduced compile-time safety
- Reflection has additional overhead

---

# Modern Alternative

Instead of Reflection:

```
Configuration

↓

Dependency Injection

↓

Factory

↓

Resolved Service
```

Modern .NET applications usually prefer Dependency Injection because it is:

- Faster
- Type-safe
- Easier to test
- Easier to maintain

---

# Reflection Factory vs Factory Method

| Reflection Factory | Factory Method |
|-------------------|----------------|
| Runtime discovery | Compile-time creation |
| Uses Reflection | Uses constructors |
| Highly dynamic | Highly type-safe |
| Slightly slower | Faster |
| Ideal for plugins | Ideal for business logic |

---

# 52. Factory using Dependency Injection

## Introduction

In modern ASP.NET Core applications, factories are usually implemented **using the built-in Dependency Injection (DI) container** instead of manually creating objects.

Rather than:

```csharp
new EmailService()
```

or

```csharp
Activator.CreateInstance()
```

the factory asks the DI container for the required implementation.

---

# Why Combine Factory with DI?

Suppose multiple payment providers exist.

```
Stripe

PayPal

Razorpay
```

The application chooses one at runtime.

The factory decides **which service** to use.

The DI container decides **how to create it**.

This keeps responsibilities separate.

---

# Step 1: Interface

```csharp
public interface IPayment
{
    void Pay();
}
```

---

# Step 2: Implementations

```csharp
public class StripePayment
    : IPayment
{
    public void Pay()
    {
        Console.WriteLine("Stripe");
    }
}
```

```csharp
public class PayPalPayment
    : IPayment
{
    public void Pay()
    {
        Console.WriteLine("PayPal");
    }
}
```

---

# Step 3: Register Services

```csharp
builder.Services.AddScoped<
    StripePayment>();

builder.Services.AddScoped<
    PayPalPayment>();
```

Notice that the concrete types are registered because the factory will resolve them directly.

---

# Step 4: Factory

```csharp
public class PaymentFactory
{
    private readonly IServiceProvider
        _provider;

    public PaymentFactory(
        IServiceProvider provider)
    {
        _provider = provider;
    }

    public IPayment Get(
        string type)
    {
        return type switch
        {
            "Stripe" =>
                _provider
                    .GetRequiredService<
                        StripePayment>(),

            "PayPal" =>
                _provider
                    .GetRequiredService<
                        PayPalPayment>(),

            _ => throw new Exception(
                "Unknown Provider")
        };
    }
}
```

---

# Usage

```csharp
var payment =
    factory.Get("Stripe");

payment.Pay();
```

---

# Better Approach (Without `IServiceProvider`)

Using `IServiceProvider` directly can become a **Service Locator**.

A cleaner design injects all implementations and builds a lookup table.

### Registration

```csharp
builder.Services.AddScoped<
    IPayment,
    StripePayment>();

builder.Services.AddScoped<
    IPayment,
    PayPalPayment>();

builder.Services.AddScoped<
    PaymentFactory>();
```

---

### Factory

```csharp
public class PaymentFactory
{
    private readonly Dictionary<
        string,
        IPayment> _payments;

    public PaymentFactory(
        IEnumerable<IPayment> payments)
    {
        _payments = payments.ToDictionary(
            p => p.GetType().Name
                    .Replace("Payment", ""),
            p => p);
    }

    public IPayment Get(string name)
    {
        return _payments[name];
    }
}
```

This approach:

- Avoids `IServiceProvider`
- Makes dependencies explicit
- Is easier to test
- Aligns better with Dependency Injection principles

---

# Internal Workflow

```
Application Starts

↓

DI Container

↓

Registers Services

↓

Factory Created

↓

Client Requests "Stripe"

↓

Factory Selects Implementation

↓

DI Supplies Instance

↓

Return Object
```

---

# Real-World Example

Cloud Storage Factory

```
Configuration

↓

Factory

↓

Azure Blob Storage

OR

Amazon S3

OR

Google Cloud Storage
```

The application code only depends on:

```csharp
IStorageService
```

---

# Reflection Factory vs DI Factory

| Reflection Factory | DI Factory |
|-------------------|------------|
| Uses Reflection | Uses DI Container |
| Runtime type lookup | Registered services |
| Slower | Faster |
| Less type-safe | Type-safe |
| Better for plugins | Better for enterprise applications |
| Harder to test | Easy to test |

---

# Factory with DI vs Factory Method

| Factory Method | Factory with DI |
|---------------|-----------------|
| Creates objects directly | Delegates creation to DI |
| Knows constructors | DI knows constructors |
| Limited dependency management | Automatic dependency resolution |
| Suitable for smaller designs | Preferred in ASP.NET Core |

---

# When to Use Reflection Factory

- Plugin frameworks
- External modules
- Dynamic assemblies
- Configuration-driven loading
- Runtime extensibility

---

# When to Use Factory with DI

- ASP.NET Core applications
- Microservices
- Enterprise applications
- Domain services
- Cloud-native applications
- Multiple service implementations

---

# Best Practices

- Prefer Dependency Injection over Reflection for normal application code.
- Use Reflection only when runtime type discovery is genuinely required.
- Avoid injecting `IServiceProvider` into business services unless there is a clear need for dynamic resolution.
- Prefer constructor injection and `IEnumerable<T>` when selecting among multiple implementations.
- Keep factories focused on selecting implementations, not managing business logic.

---

# Interview Questions

### Basic

1. What is a Reflection Factory?
2. What is `Activator.CreateInstance()`?
3. Why would you use Reflection for object creation?

### Intermediate

4. What are the disadvantages of Reflection-based factories?
5. How do you implement a factory using Dependency Injection?
6. Why is a DI-based factory preferred in ASP.NET Core?

### Advanced

7. What are the trade-offs between Reflection Factory and DI Factory?
8. Why can injecting `IServiceProvider` lead to the Service Locator anti-pattern?
9. How would you design a factory that supports multiple payment providers using `IEnumerable<T>`?
10. In what scenarios is Reflection still the right choice despite its performance overhead?

---

# Summary

A **Factory with Reflection** creates objects dynamically at runtime using .NET Reflection and `Activator.CreateInstance()`, making it ideal for plugin systems and configuration-driven applications where types are not known until execution. However, Reflection sacrifices compile-time safety and incurs additional runtime overhead. In contrast, a **Factory using Dependency Injection** delegates object creation to the ASP.NET Core DI container, providing type safety, automatic dependency resolution, better performance, and easier testing. For most enterprise .NET applications, a DI-based factory is the recommended approach, while Reflection should be reserved for scenarios that genuinely require runtime extensibility.