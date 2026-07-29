# 11. Rectangle-Square Problem

## Introduction

The **Rectangle-Square Problem** is one of the most well-known examples used to explain the **Liskov Substitution Principle (LSP)**.

At first glance, it seems logical that:

> A **Square** *is a* **Rectangle**

because a square has four sides and four right angles.

However, from a **software design** perspective, making `Square` inherit from `Rectangle` can break the expected behavior of the base class and violate the Liskov Substitution Principle.

---

# Mathematical Relationship

Mathematically:

```
Rectangle
 ├── Width
 └── Height

Square
 ├── Width = Height
```

Every square is a rectangle.

However, inheritance is about **behavior**, not just "is-a" relationships.

---

# Rectangle Class

```csharp
public class Rectangle
{
    public virtual int Width { get; set; }

    public virtual int Height { get; set; }

    public int Area()
    {
        return Width * Height;
    }
}
```

Expected behavior:

```text
Width = 5
Height = 10

Area = 50
```

---

# Square Class

```csharp
public class Square : Rectangle
{
    public override int Width
    {
        set
        {
            base.Width = value;
            base.Height = value;
        }
    }

    public override int Height
    {
        set
        {
            base.Width = value;
            base.Height = value;
        }
    }
}
```

A square must always maintain:

```
Width == Height
```

---

# Client Code

```csharp
Rectangle rectangle = new Square();

rectangle.Width = 5;
rectangle.Height = 10;

Console.WriteLine(rectangle.Area());
```

---

## Expected Result

```
5 × 10 = 50
```

---

## Actual Result

```
10 × 10 = 100
```

Why?

When `Height = 10` is assigned, the `Square` implementation also changes the `Width` to `10`.

The client expects independent width and height because it is working with a `Rectangle`.

This unexpected behavior breaks the contract of the base class.

---

# Why Does This Violate LSP?

LSP states:

> A subclass must be usable anywhere its parent class is expected without changing the correctness of the program.

The client expects:

```
Rectangle
Width = 5
Height = 10
Area = 50
```

But receives:

```
Square
Width = 10
Height = 10
Area = 100
```

The subclass changes the behavior of the base class.

Therefore, `Square` **cannot safely substitute** `Rectangle`.

---

# Better Design

Instead of inheritance, use a common abstraction.

```csharp
public interface IShape
{
    int Area();
}
```

Rectangle:

```csharp
public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }

    public int Area()
    {
        return Width * Height;
    }
}
```

Square:

```csharp
public class Square : IShape
{
    public int Side { get; set; }

    public int Area()
    {
        return Side * Side;
    }
}
```

Now both classes satisfy the same contract without violating LSP.

---

# Real-World Analogy

Imagine a **Car** class with the operation:

```text
StartEngine()
```

If `ElectricScooter` inherits from `Car` but throws an exception because it has no engine, the inheritance relationship is incorrect.

The subclass no longer behaves as the base class promises.

---

# Interview Tip

When discussing the Rectangle-Square problem, explain that the issue is **behavioral**, not mathematical. In object-oriented design, inheritance should model compatible behavior, not just a real-world classification.

---

# 12. Explain Interface Segregation Principle (ISP)

## Definition

The **Interface Segregation Principle (ISP)** states:

> **"Clients should not be forced to depend upon interfaces they do not use."**  
> — Robert C. Martin

In simple terms:

> **Many small, focused interfaces are better than one large interface.**

A class should only implement the methods it actually needs.

---

# Why is ISP Important?

Without ISP:

- Classes implement unnecessary methods.
- Some methods remain empty.
- Developers throw `NotImplementedException`.
- Interfaces become large and difficult to maintain.

With ISP:

- Interfaces are focused.
- Classes remain simple.
- Loose coupling increases.
- Unit testing becomes easier.
- New implementations become easier.

---

# Example Without ISP

Suppose we have one interface for every worker.

```csharp
public interface IWorker
{
    void Work();

    void Eat();

    void Sleep();
}
```

Human worker:

```csharp
public class HumanWorker : IWorker
{
    public void Work()
    {
    }

    public void Eat()
    {
    }

    public void Sleep()
    {
    }
}
```

Robot worker:

```csharp
public class RobotWorker : IWorker
{
    public void Work()
    {
    }

    public void Eat()
    {
        throw new NotImplementedException();
    }

    public void Sleep()
    {
        throw new NotImplementedException();
    }
}
```

---

# Problems

Robot workers:

- Do not eat.
- Do not sleep.

Yet they are forced to implement these methods.

This violates ISP.

---

# Applying ISP

Split the interface into smaller, focused interfaces.

```csharp
public interface IWorkable
{
    void Work();
}
```

```csharp
public interface IEatable
{
    void Eat();
}
```

```csharp
public interface ISleepable
{
    void Sleep();
}
```

Human worker:

```csharp
public class HumanWorker :
    IWorkable,
    IEatable,
    ISleepable
{
    public void Work()
    {
    }

    public void Eat()
    {
    }

    public void Sleep()
    {
    }
}
```

Robot worker:

```csharp
public class RobotWorker : IWorkable
{
    public void Work()
    {
    }
}
```

Now every class implements only the interfaces it requires.

---

# Benefits of ISP

- Smaller interfaces
- Better readability
- Higher cohesion
- Lower coupling
- Easier maintenance
- Better testability

---

# 13. ISP Violation Example

## Example: Multi-Function Printer

Suppose every printer is forced to implement:

```csharp
public interface IMachine
{
    void Print();

    void Scan();

    void Fax();
}
```

---

## Modern Printer

```csharp
public class ModernPrinter : IMachine
{
    public void Print()
    {
    }

    public void Scan()
    {
    }

    public void Fax()
    {
    }
}
```

Works perfectly.

---

## Basic Printer

```csharp
public class BasicPrinter : IMachine
{
    public void Print()
    {
        Console.WriteLine("Printing...");
    }

    public void Scan()
    {
        throw new NotImplementedException();
    }

    public void Fax()
    {
        throw new NotImplementedException();
    }
}
```

---

# Why is this an ISP Violation?

The `BasicPrinter` only supports printing.

However, it is forced to implement:

- Scan
- Fax

This leads to:

- Empty methods
- Exceptions
- Confusing APIs
- Difficult maintenance

---

# Correct Design

Create focused interfaces.

```csharp
public interface IPrinter
{
    void Print();
}
```

```csharp
public interface IScanner
{
    void Scan();
}
```

```csharp
public interface IFax
{
    void Fax();
}
```

Basic printer:

```csharp
public class BasicPrinter : IPrinter
{
    public void Print()
    {
        Console.WriteLine("Printing...");
    }
}
```

Multi-function printer:

```csharp
public class MultiFunctionPrinter :
    IPrinter,
    IScanner,
    IFax
{
    public void Print()
    {
    }

    public void Scan()
    {
    }

    public void Fax()
    {
    }
}
```

Now each class depends only on the functionality it actually supports.

---

# ISP in ASP.NET Core

A common example is service interfaces.

Instead of:

```csharp
public interface IUserService
{
    void CreateUser();
    void DeleteUser();
    void SendEmail();
    void GenerateReport();
}
```

Split responsibilities:

```csharp
public interface IUserManagementService
{
    void CreateUser();
    void DeleteUser();
}
```

```csharp
public interface IEmailService
{
    void SendEmail();
}
```

```csharp
public interface IReportService
{
    void GenerateReport();
}
```

Each service implements only the operations it requires, making the application more modular and easier to maintain.

---

# Interview Questions

### Rectangle-Square Problem

1. Why does the Rectangle-Square example violate LSP?
2. Why is the issue behavioral rather than mathematical?
3. How would you redesign the hierarchy?

### Interface Segregation Principle

4. What is ISP?
5. Why are large interfaces considered a bad practice?
6. How does ISP improve testability?
7. What is the difference between SRP and ISP?
8. Give a real-world example of ISP.
9. Explain ISP using a printer example.
10. How is ISP applied in ASP.NET Core applications?

---

# Summary

The **Rectangle-Square Problem** demonstrates that inheritance should preserve behavior, not just represent an "is-a" relationship. The **Interface Segregation Principle (ISP)** encourages designing **small, focused interfaces** so that classes only implement the functionality they actually need. Applying ISP leads to cleaner APIs, reduced coupling, improved maintainability, and better adherence to clean architecture principles.