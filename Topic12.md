# 33. Fluent Builder Pattern

## Introduction

The **Fluent Builder Pattern** is an enhanced version of the **Builder Pattern** that uses a **Fluent Interface**.

A Fluent Interface allows multiple methods to be chained together by returning the builder object (`this`) from each method.

Instead of writing:

```csharp
builder.SetCpu("Intel i9");

builder.SetRam("32 GB");

builder.SetStorage("1 TB SSD");
```

You can write:

```csharp
builder
    .WithCpu("Intel i9")
    .WithRam("32 GB")
    .WithStorage("1 TB SSD")
    .Build();
```

This results in code that is more readable, expressive, and easier to maintain.

---

# Why is it Called "Fluent"?

Because the code reads almost like an English sentence.

Example:

```csharp
new PizzaBuilder()
    .Large()
    .ThinCrust()
    .ExtraCheese()
    .AddOlives()
    .Build();
```

It clearly describes how the object is being created.

---

# Without Fluent Builder

```csharp
LaptopBuilder builder = new LaptopBuilder();

builder.SetCpu("Intel");

builder.SetRam("16 GB");

builder.SetStorage("1 TB SSD");

Laptop laptop = builder.Build();
```

---

# With Fluent Builder

```csharp
Laptop laptop =
    new LaptopBuilder()
        .WithCpu("Intel")
        .WithRam("16 GB")
        .WithStorage("1 TB SSD")
        .Build();
```

Notice how each method returns the same builder instance.

---

# How Fluent Builder Works Internally

```
Client

↓

WithCpu()

↓

returns Builder

↓

WithRam()

↓

returns Builder

↓

WithStorage()

↓

returns Builder

↓

Build()

↓

Laptop Object
```

Every configuration method returns the builder itself.

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

# Step 2: Fluent Builder

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

# Client Code

```csharp
Laptop laptop =
    new LaptopBuilder()
        .WithCpu("Intel i9")
        .WithRam("32 GB")
        .WithStorage("1 TB SSD")
        .EnableWifi()
        .Build();
```

Output:

```
Laptop Created
```

---

# How Method Chaining Works

Consider:

```csharp
builder
    .WithCpu("Intel")
    .WithRam("16 GB")
    .EnableWifi();
```

Internally it behaves like:

```csharp
builder.WithCpu("Intel");

builder.WithRam("16 GB");

builder.EnableWifi();
```

Each method returns:

```csharp
return this;
```

allowing the next method to be called.

---

# Real-World Example

Imagine ordering a burger.

```
BurgerBuilder

↓

Large()

↓

ExtraCheese()

↓

NoOnion()

↓

AddFries()

↓

Build()
```

Each option modifies the same order before it is finalized.

---

# Fluent Builder in ASP.NET Core

ASP.NET Core heavily uses Fluent APIs.

## Example 1: WebApplicationBuilder

```csharp
builder.Services
       .AddControllers()
       .AddJsonOptions(options =>
       {
       });
```

---

## Example 2: Entity Framework Core

```csharp
modelBuilder.Entity<User>()
    .HasKey(x => x.Id);
```

---

## Example 3: HttpClient

```csharp
builder.Services
       .AddHttpClient();
```

Many .NET libraries use Fluent APIs because they improve readability.

---

# Advantages

- Easy to read
- Supports method chaining
- Reduces constructor complexity
- Highly expressive
- Easy to extend
- Widely used in modern .NET APIs

---

# Disadvantages

- Slightly more code than constructors
- Not needed for simple objects
- Can become difficult to navigate if the builder grows too large

---

# 34. Immutable Builder Pattern

## What is Immutability?

An **immutable object** cannot be changed after it is created.

Example:

```csharp
string name = "John";
```

When you write:

```csharp
name = "David";
```

The original string is not modified. A new string object is created.

The same idea applies to immutable domain objects.

---

# Why Use Immutable Objects?

Benefits:

- Thread-safe
- Predictable behavior
- Easier debugging
- Prevent accidental modification
- Ideal for Domain-Driven Design (DDD)
- Excellent for concurrent applications

---

# Problem with Mutable Objects

```csharp
public class Employee
{
    public string Name { get; set; }

    public decimal Salary { get; set; }
}
```

Usage:

```csharp
Employee emp = new Employee();

emp.Name = "John";

emp.Salary = 50000;

emp.Salary = 100000;
```

The object changes throughout its lifetime.

This may introduce bugs if multiple parts of the application modify it.

---

# Immutable Object

```csharp
public class Employee
{
    public string Name { get; }

    public decimal Salary { get; }

    public Employee(string name, decimal salary)
    {
        Name = name;
        Salary = salary;
    }
}
```

Now:

```csharp
emp.Salary = 100000;
```

This is not allowed.

The object remains unchanged after creation.

---

# Why Builder with Immutable Objects?

Suppose an immutable object has many properties.

```csharp
public Employee(
    string name,
    string email,
    string department,
    string designation,
    decimal salary,
    bool active,
    string manager)
```

The constructor becomes difficult to use.

Instead:

```csharp
Employee employee =
    new EmployeeBuilder()
        .WithName("John")
        .WithEmail("john@test.com")
        .WithDepartment("IT")
        .WithSalary(70000)
        .Build();
```

The builder collects values and creates the immutable object only once.

---

# Immutable Builder Example

## Product

```csharp
public class Employee
{
    public string Name { get; }

    public string Department { get; }

    public decimal Salary { get; }

    public Employee(
        string name,
        string department,
        decimal salary)
    {
        Name = name;
        Department = department;
        Salary = salary;
    }
}
```

---

## Builder

```csharp
public class EmployeeBuilder
{
    private string _name = "";

    private string _department = "";

    private decimal _salary;

    public EmployeeBuilder WithName(string name)
    {
        _name = name;
        return this;
    }

    public EmployeeBuilder WithDepartment(string department)
    {
        _department = department;
        return this;
    }

    public EmployeeBuilder WithSalary(decimal salary)
    {
        _salary = salary;
        return this;
    }

    public Employee Build()
    {
        return new Employee(
            _name,
            _department,
            _salary);
    }
}
```

---

# Client

```csharp
Employee employee =
    new EmployeeBuilder()
        .WithName("John")
        .WithDepartment("IT")
        .WithSalary(70000)
        .Build();
```

After `Build()` returns, the `Employee` object cannot be modified.

---

# How It Works Internally

```
Builder

↓

Stores Temporary Values

↓

Build()

↓

Creates Immutable Object

↓

Builder Ends
```

The builder is mutable, but the final object is immutable.

---

# Real-World Example

Imagine applying for a passport.

While filling the form, you can edit:

- Name
- Address
- Date of Birth

Once the passport is issued, those details cannot be changed without a formal update process.

The application form acts like the builder, while the issued passport is the immutable object.

---

# Immutable Builder in ASP.NET Core

Immutable patterns are common in:

- Configuration objects
- Authentication tokens
- Request models
- Domain entities
- Record types

Example using a C# record:

```csharp
public record User(
    string Name,
    string Email,
    string Department);
```

A builder can simplify creating records when there are many optional values.

---

# Fluent Builder vs Immutable Builder

| Fluent Builder | Immutable Builder |
|----------------|-------------------|
| Focuses on readability through method chaining | Focuses on creating immutable objects |
| Usually returns `this` from each method | Often also uses fluent chaining |
| Final object may be mutable or immutable | Final object is immutable |
| Common in APIs and configuration | Common in domain models and DDD |
| Goal: Easy configuration | Goal: Safe, predictable objects |

---

# Advantages of Immutable Builder

- Thread-safe
- Prevents accidental modification
- Easier unit testing
- Predictable state
- Works well with Domain-Driven Design
- Ideal for concurrent systems

---

# Interview Questions

### Basic

1. What is a Fluent Builder?
2. What is method chaining?
3. Why does each builder method return `this`?

### Intermediate

4. What is an immutable object?
5. Why combine Builder with immutable objects?
6. What are the benefits of immutable objects?

### Advanced

7. Explain Fluent Builder with an ASP.NET Core example.
8. Explain Immutable Builder using C# records.
9. Why are immutable objects preferred in multi-threaded applications?
10. Can a Fluent Builder also create immutable objects?

---

# Summary

The **Fluent Builder Pattern** enhances the Builder Pattern by using **method chaining**, making object construction more readable and expressive. The **Immutable Builder Pattern** combines the Builder Pattern with immutable objects, allowing complex objects to be configured step by step and then created in a state that cannot be modified. In modern C# and ASP.NET Core applications, these patterns are widely used to improve readability, maintainability, thread safety, and domain model integrity.