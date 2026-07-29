# 5. SRP Violation Example

## Problem Statement

A common violation of the **Single Responsibility Principle (SRP)** occurs when a single class handles multiple unrelated responsibilities.

Consider an Order Management System.

---

# Example Without SRP

```csharp
public class OrderService
{
    public void CreateOrder(Order order)
    {
        Console.WriteLine("Order Created");
    }

    public void SaveOrder(Order order)
    {
        Console.WriteLine("Saving to SQL Server");
    }

    public void SendEmail(Order order)
    {
        Console.WriteLine("Email Sent");
    }

    public void GenerateInvoice(Order order)
    {
        Console.WriteLine("Invoice Generated");
    }

    public void PrintInvoice(Order order)
    {
        Console.WriteLine("Invoice Printed");
    }

    public void LogActivity(string message)
    {
        Console.WriteLine(message);
    }
}
```

---

## Why is this an SRP Violation?

This class has multiple responsibilities:

| Method | Responsibility |
|---------|---------------|
| CreateOrder() | Business Logic |
| SaveOrder() | Database Access |
| SendEmail() | Notification |
| GenerateInvoice() | Invoice Generation |
| PrintInvoice() | Printing |
| LogActivity() | Logging |

This class has **six different reasons to change**.

For example:

- Database changes → Modify `SaveOrder()`
- Email provider changes → Modify `SendEmail()`
- Invoice format changes → Modify `GenerateInvoice()`
- Logging framework changes → Modify `LogActivity()`

Every change affects the same class, increasing maintenance cost and the risk of introducing bugs.

---

# Refactored Solution

```csharp
public class OrderService
{
    public void CreateOrder(Order order)
    {
    }
}

public class OrderRepository
{
    public void Save(Order order)
    {
    }
}

public class EmailService
{
    public void Send(Order order)
    {
    }
}

public class InvoiceService
{
    public void Generate(Order order)
    {
    }
}

public class InvoicePrinter
{
    public void Print(Order order)
    {
    }
}

public class LoggerService
{
    public void Log(string message)
    {
    }
}
```

Now each class has a **single responsibility** and only one reason to change.

---

# Benefits After Applying SRP

- Easier unit testing
- Lower coupling
- Higher cohesion
- Better code reuse
- Easier maintenance
- Independent deployment of changes

---

# 6. Explain Open Closed Principle (OCP)

## Definition

The **Open/Closed Principle (OCP)** states:

> **"Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification."**  
> — Robert C. Martin (Uncle Bob)

This means:

- **Open for Extension** → You should be able to add new functionality.
- **Closed for Modification** → Existing, tested code should not need to be changed.

Instead of modifying existing classes whenever requirements change, extend the system by adding new classes or implementations.

---

# Why is OCP Important?

Without OCP:

- Every new requirement changes existing code.
- Existing functionality may break.
- Regression testing becomes extensive.
- Code becomes difficult to maintain.

With OCP:

- Existing code remains stable.
- New features are added through extension.
- Easier maintenance and scalability.
- Lower risk of introducing bugs.

---

# Example Without OCP

Suppose an application supports only Credit Card payments.

```csharp
public class PaymentService
{
    public void ProcessPayment(string paymentType)
    {
        if (paymentType == "CreditCard")
        {
            Console.WriteLine("Credit Card Payment");
        }
        else if (paymentType == "PayPal")
        {
            Console.WriteLine("PayPal Payment");
        }
    }
}
```

---

## Problems

Tomorrow the business adds:

- UPI
- Net Banking
- Apple Pay
- Google Pay

Every time a new payment type is added, you must modify `PaymentService`.

This violates OCP because the class is **not closed for modification**.

---

# Applying OCP

## Step 1: Create an abstraction

```csharp
public interface IPaymentMethod
{
    void Pay();
}
```

---

## Step 2: Implement different payment methods

```csharp
public class CreditCardPayment : IPaymentMethod
{
    public void Pay()
    {
        Console.WriteLine("Credit Card Payment");
    }
}
```

```csharp
public class PayPalPayment : IPaymentMethod
{
    public void Pay()
    {
        Console.WriteLine("PayPal Payment");
    }
}
```

```csharp
public class UpiPayment : IPaymentMethod
{
    public void Pay()
    {
        Console.WriteLine("UPI Payment");
    }
}
```

---

## Step 3: Use the abstraction

```csharp
public class PaymentService
{
    public void ProcessPayment(IPaymentMethod paymentMethod)
    {
        paymentMethod.Pay();
    }
}
```

Usage:

```csharp
var service = new PaymentService();

service.ProcessPayment(new CreditCardPayment());
service.ProcessPayment(new PayPalPayment());
service.ProcessPayment(new UpiPayment());
```

Adding a new payment method requires creating a new implementation of `IPaymentMethod`; `PaymentService` remains unchanged.

---

# Before vs After OCP

| Without OCP | With OCP |
|--------------|----------|
| Uses if-else or switch for every type | Uses polymorphism |
| Existing class changes frequently | Existing class remains unchanged |
| High risk of regression | Low risk |
| Difficult to scale | Easy to extend |

---

# Real-World Analogy

Consider a USB port on a laptop.

The laptop's USB interface does not change when you connect:

- Keyboard
- Mouse
- Printer
- Webcam
- External Hard Drive

The laptop is **closed for modification**, while new USB devices **extend** its capabilities.

---

# 7. OCP in ASP.NET Core

ASP.NET Core is designed around OCP and extensibility.

---

# Example 1: Dependency Injection

Interface:

```csharp
public interface IMessageService
{
    void Send(string message);
}
```

Email implementation:

```csharp
public class EmailService : IMessageService
{
    public void Send(string message)
    {
        Console.WriteLine("Email: " + message);
    }
}
```

SMS implementation:

```csharp
public class SmsService : IMessageService
{
    public void Send(string message)
    {
        Console.WriteLine("SMS: " + message);
    }
}
```

Register in DI:

```csharp
builder.Services.AddScoped<IMessageService, EmailService>();
```

Later, if the business wants SMS instead of Email:

```csharp
builder.Services.AddScoped<IMessageService, SmsService>();
```

The consuming classes remain unchanged.

---

# Example 2: Authentication Providers

ASP.NET Core supports multiple authentication schemes.

You can configure:

- JWT Bearer
- Cookies
- OAuth 2.0
- OpenID Connect

Each provider extends the authentication pipeline without modifying your controllers or business logic.

---

# Example 3: Logging Providers

ASP.NET Core supports different logging providers.

Examples:

- Console
- Debug
- Azure Application Insights
- Seq
- Serilog
- NLog

Adding a new provider extends the logging capabilities without changing application code.

---

# Example 4: Middleware Pipeline

ASP.NET Core middleware demonstrates OCP.

```csharp
app.UseAuthentication();

app.UseAuthorization();

app.UseMiddleware<RequestLoggingMiddleware>();

app.UseMiddleware<ExceptionMiddleware>();

app.MapControllers();
```

You can introduce new middleware without modifying existing middleware components.

---

# Example 5: Repository Pattern

Interface:

```csharp
public interface ICustomerRepository
{
    Customer Get(int id);
}
```

SQL implementation:

```csharp
public class SqlCustomerRepository : ICustomerRepository
{
}
```

MongoDB implementation:

```csharp
public class MongoCustomerRepository : ICustomerRepository
{
}
```

The business layer depends only on `ICustomerRepository`, making it easy to switch storage implementations.

---

# Advantages of OCP in ASP.NET Core

- Supports Dependency Injection naturally.
- Encourages interface-based design.
- Makes applications easier to extend.
- Reduces regression bugs.
- Improves maintainability and scalability.
- Enables plug-and-play implementations.

---

# Interview Questions

### Basic

1. What is the Open/Closed Principle?
2. Why is OCP important?
3. Give a real-world example of OCP.

### Intermediate

4. How does polymorphism help implement OCP?
5. Why are long `if-else` or `switch` statements often a sign of an OCP violation?
6. How does Dependency Injection support OCP?

### Advanced

7. Explain OCP with an ASP.NET Core example.
8. How does middleware demonstrate OCP?
9. How do authentication schemes in ASP.NET Core follow OCP?
10. What are the trade-offs of applying OCP in large enterprise applications?

---

# Summary

The **Single Responsibility Principle (SRP)** ensures that a class has only one responsibility and one reason to change, resulting in focused, maintainable, and testable code. The **Open/Closed Principle (OCP)** states that software should be **open for extension but closed for modification**, allowing new functionality to be added without altering stable, existing code. ASP.NET Core embraces OCP through Dependency Injection, middleware, authentication providers, logging providers, and interface-driven design, making it easier to build scalable and extensible enterprise applications.