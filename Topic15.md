# 39. Prototype Pattern

## Introduction

The **Prototype Pattern** is one of the **Gang of Four (GoF) Creational Design Patterns**.

Instead of creating a new object from scratch, it creates a new object by **copying (cloning)** an existing object.

This pattern is especially useful when object creation is:

- Expensive
- Time-consuming
- Complex
- Resource-intensive

---

# Definition

> **Create new objects by copying an existing object instead of creating them from scratch.**

---

# Why Do We Need Prototype Pattern?

Imagine creating a Machine Learning model.

```text
Load Model

↓

Read 2 GB File

↓

Initialize GPU

↓

Create Object
```

This may take several seconds.

Instead of repeating the initialization every time, create one object and clone it whenever another instance is needed.

---

# Without Prototype

```csharp
Employee employee1 = new Employee();

employee1.Name = "John";

employee1.Department = "IT";

Employee employee2 = new Employee();

employee2.Name = "John";

employee2.Department = "IT";
```

Every object is built from scratch.

---

# With Prototype

```csharp
Employee employee1 = new Employee
{
    Name = "John",
    Department = "IT"
};

Employee employee2 = employee1.Clone();
```

The second object is created by copying the first.

---

# Prototype Structure

```
            Client

               │

               ▼

        Existing Object

               │

            Clone()

               │

               ▼

          New Object
```

---

# Step 1: Prototype Interface

```csharp
public interface IPrototype<T>
{
    T Clone();
}
```

---

# Step 2: Concrete Prototype

```csharp
public class Employee : IPrototype<Employee>
{
    public string Name { get; set; }

    public string Department { get; set; }

    public Employee Clone()
    {
        return (Employee)this.MemberwiseClone();
    }
}
```

---

# Step 3: Client

```csharp
Employee employee1 = new Employee
{
    Name = "John",
    Department = "IT"
};

Employee employee2 = employee1.Clone();

employee2.Name = "David";
```

Result:

```
employee1.Name = John

employee2.Name = David
```

They are different objects.

---

# How It Works Internally

```
Employee Object

↓

MemberwiseClone()

↓

Memory Copied

↓

New Employee Object
```

No constructor logic is executed during cloning.

---

# Real-World Example

Suppose a company prints employee ID cards.

Instead of designing every card from scratch:

```
Template

↓

Copy

↓

Change Name

↓

Print
```

The template acts as the prototype.

---

# Deep Copy vs Shallow Copy

Understanding these concepts is essential when using the Prototype Pattern.

---

## Shallow Copy

A shallow copy duplicates the object itself but **shares references** to nested objects.

Example:

```csharp
public class Address
{
    public string City { get; set; }
}

public class Employee
{
    public string Name { get; set; }

    public Address Address { get; set; }
}
```

Clone:

```csharp
Employee employee2 =
    (Employee)employee1.MemberwiseClone();
```

Memory representation:

```
Employee1

↓

Address (Shared)

↑

Employee2
```

Changing the address:

```csharp
employee2.Address.City = "Mumbai";
```

Also changes:

```text
employee1.Address.City
```

because both employees reference the same `Address` object.

---

## Deep Copy

Deep copy duplicates both the object and all nested objects.

Example:

```csharp
public Employee Clone()
{
    return new Employee
    {
        Name = Name,
        Address = new Address
        {
            City = Address.City
        }
    };
}
```

Memory:

```
Employee1

↓

Address1

Employee2

↓

Address2
```

Now modifying one address does not affect the other.

---

# Prototype Pattern with Deep Copy

```csharp
public class Employee
{
    public string Name { get; set; }

    public Address Address { get; set; }

    public Employee Clone()
    {
        return new Employee
        {
            Name = Name,

            Address = new Address
            {
                City = Address.City
            }
        };
    }
}
```

---

# Prototype in ASP.NET Core

Examples include:

- Cloning configuration objects
- Creating request templates
- Copying report definitions
- Duplicating workflow definitions
- Creating reusable AI prompt templates

Suppose an AI application stores a base prompt.

```
Base Prompt

↓

Clone

↓

Customize User Input

↓

Send to LLM
```

The original template remains unchanged.

---

# Advantages

- Faster than creating complex objects repeatedly.
- Reduces expensive initialization.
- Simplifies object creation.
- Preserves object state.
- Reduces duplicate initialization code.

---

# Disadvantages

- Deep copying complex object graphs can be difficult.
- Circular references require special handling.
- Incorrect cloning can lead to shared mutable state.
- Maintenance becomes harder as object structures evolve.

---

# 40. Prototype Pattern vs Clone

Many developers use these terms interchangeably, but they are **not the same**.

---

# What is Clone?

Cloning is simply the **operation** of copying an object.

Example:

```csharp
Employee employee2 =
    employee1.Clone();
```

`Clone()` is just a method.

---

# What is Prototype?

Prototype is a **design pattern** that uses cloning as its primary mechanism for object creation.

```
Prototype Pattern

↓

Clone Existing Object

↓

Create New Object
```

So:

- **Clone** is the mechanism.
- **Prototype** is the design pattern.

---

# Analogy

Imagine a photocopier.

**Clone**

```
Photocopy one document
```

This is the act of copying.

---

**Prototype Pattern**

```
Maintain one master template.

↓

Photocopy whenever needed.
```

This is a reusable strategy for object creation.

---

# Comparison

| Clone | Prototype Pattern |
|--------|-------------------|
| Operation | Design Pattern |
| Copies an object | Creates new objects by cloning prototypes |
| Can be used anywhere | Specifically addresses object creation |
| Simple method | Full creational design pattern |
| May use shallow or deep copy | Usually defines a cloning strategy |

---

# Prototype vs Factory

| Prototype | Factory |
|-----------|---------|
| Creates by copying | Creates by instantiating |
| Uses existing object | Uses constructors |
| Good for expensive objects | Good for selecting implementations |
| Avoids repeated initialization | Encapsulates creation logic |

---

# Prototype vs Builder

| Prototype | Builder |
|-----------|----------|
| Copies an existing object | Builds a new object step by step |
| Fast duplication | Flexible construction |
| Best for similar objects | Best for configurable objects |

---

# Prototype vs Singleton

| Prototype | Singleton |
|-----------|-----------|
| Creates many objects | Creates only one object |
| Uses cloning | Uses a single shared instance |
| Multiple independent copies | One global instance |

---

# Clone Techniques in C#

## 1. MemberwiseClone()

```csharp
Employee clone =
    (Employee)this.MemberwiseClone();
```

Creates a shallow copy.

---

## 2. Manual Deep Copy

```csharp
return new Employee
{
    Name = Name,
    Address = new Address
    {
        City = Address.City
    }
};
```

Creates a deep copy.

---

## 3. Copy Constructor

```csharp
public Employee(Employee other)
{
    Name = other.Name;

    Department = other.Department;
}
```

Usage:

```csharp
Employee clone =
    new Employee(existingEmployee);
```

Provides explicit control over copying behavior.

---

# Why `ICloneable` is Rarely Recommended

Although .NET provides:

```csharp
public interface ICloneable
{
    object Clone();
}
```

it does **not specify** whether cloning should be **shallow** or **deep**.

This ambiguity can confuse API consumers.

Modern .NET code often prefers:

- Strongly typed `Clone()` methods
- Copy constructors
- Factory methods
- Immutable records with `with` expressions

Example:

```csharp
public record Employee(
    string Name,
    string Department);

Employee employee2 =
    employee1 with
    {
        Department = "HR"
    };
```

---

# Interview Questions

### Basic

1. What is the Prototype Pattern?
2. Why do we use the Prototype Pattern?
3. What is cloning?

### Intermediate

4. What is the difference between shallow copy and deep copy?
5. Why is `MemberwiseClone()` a shallow copy?
6. Why is `ICloneable` generally discouraged?

### Advanced

7. Explain Prototype using an ASP.NET Core example.
8. When would you choose Prototype over Factory?
9. How would you implement deep cloning for nested object graphs?
10. How can immutable records reduce the need for traditional cloning?

---

# Summary

The **Prototype Pattern** is a creational design pattern that creates new objects by **cloning existing ones**, making it ideal when object creation is expensive or complex. Cloning can be performed as a **shallow copy**, where nested references are shared, or a **deep copy**, where the entire object graph is duplicated. While **cloning** is simply the act of copying an object, the **Prototype Pattern** is a structured approach to object creation that relies on cloning. Modern C# applications often use strongly typed `Clone()` methods, copy constructors, or immutable records instead of the ambiguous `ICloneable` interface.