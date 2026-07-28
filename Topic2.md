# 3. Explain Single Responsibility Principle (SRP)

## Definition

The **Single Responsibility Principle (SRP)** states:

> **"A class should have only one reason to change."**  
> — Robert C. Martin (Uncle Bob)

In simple terms, a class should have **one responsibility** and perform **one well-defined job**. If a class handles multiple responsibilities, changes in one area can unintentionally affect the others, making the code harder to maintain and test.

---

# What does "One Reason to Change" Mean?

A class should change only when its **single responsibility** changes.

For example, consider an `Invoice`:

Possible responsibilities include:

- Calculating the invoice total
- Saving the invoice to a database
- Printing the invoice
- Sending the invoice by email

These are four different responsibilities. If all of them are implemented in one class:

- A database change affects the class.
- A printing requirement change affects the class.
- An email template change affects the class.
- A calculation rule change affects the class.

This violates SRP because the class has **multiple reasons to change**.

---

# Why is SRP Important?

Applying SRP provides several benefits:

- Easier to understand because each class has a clear purpose.
- Easier to maintain since changes are isolated.
- Easier to test because each class focuses on one task.
- Improved code reuse.
- Reduced coupling between different parts of the application.
- Lower risk of introducing bugs when making changes.

---

# Example Without SRP

```csharp
public class InvoiceService
{
    public void CalculateTotal()
    {
        Console.WriteLine("Calculating invoice total...");
    }

    public void SaveToDatabase()
    {
        Console.WriteLine("Saving invoice...");
    }

    public void PrintInvoice()
    {
        Console.WriteLine("Printing invoice...");
    }

    public void SendEmail()
    {
        Console.WriteLine("Sending invoice email...");
    }
}
```

### Problems

This class performs four different jobs:

- Business logic
- Database operations
- Printing
- Email communication

Any change in these areas requires modifying the same class.

---

# Refactored Using SRP

## Invoice Calculator

```csharp
public class InvoiceCalculator
{
    public decimal CalculateTotal(decimal amount, decimal tax)
    {
        return amount + tax;
    }
}
```

---

## Invoice Repository

```csharp
public class InvoiceRepository
{
    public void Save()
    {
        Console.WriteLine("Invoice saved.");
    }
}
```

---

## Invoice Printer

```csharp
public class InvoicePrinter
{
    public void Print()
    {
        Console.WriteLine("Invoice printed.");
    }
}
```

---

## Email Service

```csharp
public class EmailService
{
    public void Send()
    {
        Console.WriteLine("Invoice emailed.");
    }
}
```

Each class now has a **single responsibility** and therefore only **one reason to change**.

---

# Before vs After SRP

| Without SRP | With SRP |
|-------------|----------|
| One class performs many tasks | Each class performs one task |
| Difficult to test | Easy to test |
| Tightly coupled | Loosely coupled |
| High maintenance cost | Easier maintenance |
| Low reusability | High reusability |

---

# Real-World Banking Example

### Without SRP

```csharp
public class BankAccountService
{
    public void WithdrawMoney()
    {
    }

    public void SaveTransaction()
    {
    }

    public void SendSms()
    {
    }

    public void GenerateStatement()
    {
    }
}
```

This class has multiple responsibilities:

- Banking operations
- Database persistence
- SMS notifications
- Statement generation

Any change to one feature impacts the same class.

---

### With SRP

```csharp
public class AccountService
{
    public void WithdrawMoney()
    {
    }
}

public class TransactionRepository
{
    public void Save()
    {
    }
}

public class SmsService
{
    public void Send()
    {
    }
}

public class StatementGenerator
{
    public void Generate()
    {
    }
}
```

Now each class focuses on a single responsibility.

---

# 4. Real-World Example of SRP

## Restaurant Example

Imagine a restaurant.

### Without SRP

One employee is responsible for:

- Taking customer orders
- Cooking food
- Serving food
- Cleaning tables
- Collecting payment

Problems:

- Work becomes slow.
- A mistake in one task affects all others.
- Replacing the employee is difficult.
- Training takes longer.

---

### With SRP

Each employee has a dedicated responsibility:

| Employee | Responsibility |
|----------|----------------|
| Waiter | Takes orders and serves food |
| Chef | Cooks meals |
| Cashier | Handles payments |
| Cleaner | Cleans tables |
| Manager | Oversees operations |

Each role has **one responsibility**, making the restaurant more efficient and easier to manage.

---

# E-Commerce Example

Suppose a customer places an order.

Instead of one class doing everything:

```
OrderService
 ├── Validate Order
 ├── Calculate Price
 ├── Save Order
 ├── Send Email
 ├── Generate Invoice
 └── Update Inventory
```

We split the responsibilities:

```
OrderValidator
        │
        ▼
PriceCalculator
        │
        ▼
OrderRepository
        │
        ▼
InvoiceService
        │
        ▼
EmailService
        │
        ▼
InventoryService
```

Benefits:

- Each component is independent.
- Features can evolve without affecting unrelated code.
- Easier unit testing.
- Better scalability and maintainability.

---

# How SRP Improves Unit Testing

Without SRP:

```text
InvoiceService
 ├── Calculate()
 ├── Save()
 ├── Print()
 └── SendEmail()
```

To test `Calculate()`, you may need to mock database, printer, and email dependencies.

With SRP:

```text
InvoiceCalculator
 └── Calculate()
```

Only the calculation logic needs to be tested, making tests simpler and faster.

---

# Common Mistakes

- Creating "God Classes" that perform many unrelated tasks.
- Mixing business logic with database or UI code.
- Combining validation, logging, and persistence in one class.
- Treating utility classes as a place for unrelated methods.

---

# Interview Questions

### Basic

1. What is the Single Responsibility Principle?
2. What does "one reason to change" mean?
3. Why is SRP important?

### Intermediate

4. Can a class have two responsibilities?
5. How does SRP improve maintainability?
6. How does SRP improve unit testing?
7. Give a real-world example of SRP.

### Advanced

8. How does SRP support Clean Architecture?
9. What problems arise when SRP is violated?
10. How does Dependency Injection complement SRP in ASP.NET Core?

---

# Summary

The **Single Responsibility Principle (SRP)** states that a class should have **only one responsibility and one reason to change**. By separating different concerns into dedicated classes, applications become easier to understand, test, maintain, and extend. SRP is a foundational principle of clean code and is widely applied in enterprise applications, microservices, and modern frameworks such as ASP.NET Core.