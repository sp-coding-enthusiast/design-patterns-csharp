# 8. How does Strategy Pattern satisfy Open/Closed Principle (OCP)?

## Introduction

The **Strategy Pattern** is a **behavioral design pattern** that allows an algorithm or behavior to be selected at runtime by encapsulating each algorithm in a separate class.

Instead of using multiple `if-else` or `switch` statements, each behavior is implemented as a separate strategy class that shares a common interface.

This directly supports the **Open/Closed Principle (OCP)** because:

- Existing code **does not need to change** when a new strategy is added.
- New functionality is added by creating a **new strategy class**, not by modifying existing classes.

---

# Problem Without Strategy Pattern

Suppose we are building a payment system.

```csharp
public class PaymentService
{
    public void Pay(string paymentType)
    {
        if (paymentType == "CreditCard")
        {
            Console.WriteLine("Paid using Credit Card");
        }
        else if (paymentType == "UPI")
        {
            Console.WriteLine("Paid using UPI");
        }
        else if (paymentType == "PayPal")
        {
            Console.WriteLine("Paid using PayPal");
        }
    }
}
```

### Problems

Every time a new payment method is introduced:

- Apple Pay
- Google Pay
- Net Banking
- Crypto Wallet

You must modify the existing `PaymentService`.

This **violates OCP** because the class is constantly being modified.

---

# Applying Strategy Pattern

## Step 1: Create the Strategy Interface

```csharp
public interface IPaymentStrategy
{
    void Pay(decimal amount);
}
```

---

## Step 2: Implement Different Strategies

### Credit Card Strategy

```csharp
public class CreditCardPayment : IPaymentStrategy
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ₹{amount} using Credit Card");
    }
}
```

---

### UPI Strategy

```csharp
public class UpiPayment : IPaymentStrategy
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ₹{amount} using UPI");
    }
}
```

---

### PayPal Strategy

```csharp
public class PayPalPayment : IPaymentStrategy
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ₹{amount} using PayPal");
    }
}
```

---

## Step 3: Context Class

```csharp
public class PaymentService
{
    private readonly IPaymentStrategy _paymentStrategy;

    public PaymentService(IPaymentStrategy paymentStrategy)
    {
        _paymentStrategy = paymentStrategy;
    }

    public void ProcessPayment(decimal amount)
    {
        _paymentStrategy.Pay(amount);
    }
}
```

---

## Usage

```csharp
IPaymentStrategy strategy = new UpiPayment();

PaymentService service = new PaymentService(strategy);

service.ProcessPayment(5000);
```

Output:

```
Paid ₹5000 using UPI
```

---

## Adding a New Payment Method

```csharp
public class ApplePayPayment : IPaymentStrategy
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ₹{amount} using Apple Pay");
    }
}
```

No changes are required in:

- `PaymentService`
- Existing payment strategies

Only a new strategy is added.

This is the **Open/Closed Principle** in action.

---

# Relationship Between Strategy Pattern and OCP

```
                 IPaymentStrategy
                        ▲
      ┌─────────────────┼─────────────────┐
      │                 │                 │
CreditCardPayment   UpiPayment     PayPalPayment
      │                 │                 │
      └─────────────────┼─────────────────┘
                        │
                 PaymentService
```

The `PaymentService` depends on an abstraction (`IPaymentStrategy`), allowing new behaviors to be introduced without modifying existing code.

---

# Real-World Example

Think about a GPS navigation app.

The app supports multiple route strategies:

- Fastest Route
- Shortest Route
- Avoid Highways
- Avoid Tolls

Each routing algorithm is a separate strategy.

When a new routing option is introduced, the navigation engine remains unchanged.

---

# Benefits

- Supports OCP
- Eliminates long `if-else` chains
- Easier unit testing
- Runtime flexibility
- Loosely coupled code
- Easy to add new behaviors

---

# 9. Explain Liskov Substitution Principle (LSP)

## Definition

The **Liskov Substitution Principle (LSP)** states:

> **"Objects of a derived class should be replaceable with objects of the base class without altering the correctness of the program."**  
> — Barbara Liskov

In simple words:

If class **B** inherits from class **A**, then anywhere class **A** is expected, class **B** should work correctly without causing unexpected behavior.

A derived class **must honor the contract** of its base class.

---

# Why is LSP Important?

LSP ensures that inheritance is used correctly.

Benefits include:

- Reliable polymorphism
- Predictable behavior
- Easier maintenance
- Better extensibility
- Reduced runtime errors

---

# Correct Example

```csharp
public abstract class Bird
{
    public abstract void Eat();
}
```

Derived classes:

```csharp
public class Sparrow : Bird
{
    public override void Eat()
    {
        Console.WriteLine("Sparrow is eating.");
    }
}

public class Parrot : Bird
{
    public override void Eat()
    {
        Console.WriteLine("Parrot is eating.");
    }
}
```

Usage:

```csharp
Bird bird = new Sparrow();
bird.Eat();

bird = new Parrot();
bird.Eat();
```

Both subclasses can replace `Bird` without breaking the program.

This satisfies LSP.

---

# Characteristics of LSP

A derived class should:

- Preserve expected behavior.
- Not throw unexpected exceptions.
- Not weaken the contract of the base class.
- Not require additional conditions that the base class does not require.
- Behave consistently with the expectations of callers.

---

# Real-World Analogy

Imagine a universal charger.

If a laptop advertises support for a standard USB-C charger, any compliant USB-C charger should work.

If one charger damages the laptop or refuses to charge under normal conditions, it violates the expected contract.

Similarly, subclasses should behave as expected wherever the base class is used.

---

# 10. LSP Violation Example

The classic example is **Rectangle and Square**.

---

## Base Class

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

---

## Derived Class

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

---

## Client Code

```csharp
Rectangle rectangle = new Square();

rectangle.Width = 5;
rectangle.Height = 10;

Console.WriteLine(rectangle.Area());
```

Expected Output:

```
50
```

Actual Output:

```
100
```

Why?

When `Height` is set to `10`, the `Square` implementation also changes the `Width` to `10`.

The behavior differs from what callers expect from a `Rectangle`.

This **violates LSP** because substituting `Square` for `Rectangle` changes the correctness of the program.

---

# Better Design

Instead of forcing `Square` to inherit from `Rectangle`, define a common abstraction.

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

Now each shape implements the same contract without violating expected behavior.

---

# ASP.NET Core Example of LSP

Suppose you have:

```csharp
public interface IFileStorage
{
    Task UploadAsync(Stream file);
}
```

Implementations:

```csharp
public class AzureBlobStorage : IFileStorage
{
    public Task UploadAsync(Stream file)
    {
        // Upload to Azure Blob Storage
    }
}

public class AwsS3Storage : IFileStorage
{
    public Task UploadAsync(Stream file)
    {
        // Upload to Amazon S3
    }
}
```

The application depends only on `IFileStorage`.

Any implementation can replace another without changing client code or behavior, satisfying LSP.

---

# Interview Questions

### Basic

1. What is the Liskov Substitution Principle?
2. Why is LSP important?
3. Give a real-world example of LSP.

### Intermediate

4. Why does the Rectangle-Square example violate LSP?
5. How is LSP different from inheritance?
6. How does LSP improve polymorphism?

### Advanced

7. How does LSP relate to Dependency Injection?
8. How can violating LSP introduce runtime bugs?
9. How do interfaces help satisfy LSP?
10. Explain LSP with an ASP.NET Core example.

---

# Summary

The **Strategy Pattern** satisfies the **Open/Closed Principle** by encapsulating behaviors behind a common interface, allowing new strategies to be added without modifying existing code. The **Liskov Substitution Principle (LSP)** ensures that derived classes can replace their base classes without changing the expected behavior of the application. Designing against abstractions and honoring contracts results in reliable polymorphism, maintainable code, and scalable software architectures.