# SOLID Principles

## 1. What is SOLID?

**SOLID** is a set of **five object-oriented design principles** introduced by **Robert C. Martin (Uncle Bob)** that help developers write software that is:

- Easy to understand
- Easy to maintain
- Easy to extend
- Loosely coupled
- Highly testable
- Scalable

Instead of writing tightly coupled classes with multiple responsibilities, SOLID encourages designing software where each class has a clear purpose and interacts with others through abstractions.

---

## Why was SOLID introduced?

As software grows, code often becomes:

- Difficult to modify
- Hard to test
- Full of duplicated logic
- Tightly coupled
- Prone to bugs whenever a change is made

SOLID principles solve these problems by promoting clean architecture and better object-oriented design.

---

# The Five SOLID Principles

| Letter | Principle | Meaning |
|---------|-----------|---------|
| S | Single Responsibility Principle | One class should have one reason to change |
| O | Open/Closed Principle | Open for extension, closed for modification |
| L | Liskov Substitution Principle | Child classes should replace parent classes without breaking behavior |
| I | Interface Segregation Principle | Don't force classes to implement methods they don't need |
| D | Dependency Inversion Principle | Depend on abstractions, not concrete classes |

---

# 2. Why is SOLID Important?

SOLID principles make software:

## 1. Easier to Maintain

If every class has only one responsibility, changing one feature won't affect unrelated functionality.

Without SOLID:

```
Changing invoice printing breaks invoice calculation.
```

With SOLID:

```
InvoiceCalculator
InvoicePrinter
InvoiceRepository

Each changes independently.
```

---

## 2. Easier to Extend

New functionality can be added without modifying existing code.

Example:

Suppose your application supports:

- Credit Card
- PayPal

Later, the business asks for:

- UPI
- Apple Pay

Using SOLID, you simply add another payment class instead of modifying existing code.

---

## 3. Better Testability

Small classes are easier to unit test.

Example:

```
InvoiceCalculator

Only calculates totals.
```

Testing becomes simple.

Without SOLID:

```
InvoiceService

- Calculates
- Saves
- Prints
- Sends Email
```

Testing becomes difficult.

---

## 4. Reduced Coupling

Classes depend on interfaces rather than concrete implementations.

Instead of

```
OrderService
      ↓
SqlRepository
```

Use

```
OrderService
      ↓
IRepository
      ↓
SqlRepository

OR

MongoRepository
```

Changing the database requires no changes to business logic.

---

## 5. Better Reusability

Well-designed classes can be reused in other projects.

Example:

```
EmailSender
```

can be reused in:

- Ecommerce
- HR System
- Banking
- Hospital Management

---

## 6. Easier Team Development

When classes have clear responsibilities, multiple developers can work simultaneously with fewer merge conflicts.

Example:

Developer A:

```
InvoiceCalculator
```

Developer B:

```
InvoicePrinter
```

Developer C:

```
InvoiceRepository
```

No one interferes with another's code.

---

## 7. Supports Clean Architecture

Frameworks such as:

- ASP.NET Core
- Spring Boot
- Angular
- React
- Java Enterprise
- Azure Functions

all encourage SOLID principles.

---

# Real World Analogy

Imagine a restaurant.

Without SOLID:

One employee:

- Takes orders
- Cooks food
- Cleans tables
- Handles payment
- Delivers food

If one task changes, everything is affected.

With SOLID:

```
Cashier
Chef
Waiter
Cleaner
Manager
```

Each person has one responsibility.

This makes the restaurant efficient and easier to manage.

---

# Practical Example

Suppose we build an E-Commerce application.

Without SOLID:

```csharp
public class OrderService
{
    public void CreateOrder()
    {
        // Business Logic
    }

    public void SaveOrder()
    {
        // Database Logic
    }

    public void PrintInvoice()
    {
        // Printing Logic
    }

    public void SendEmail()
    {
        // Email Logic
    }
}
```

Problems:

- Too many responsibilities
- Difficult to test
- Difficult to modify
- Violates Single Responsibility Principle

---

Using SOLID:

```csharp
public class OrderService
{
    public void CreateOrder()
    {
        // Business Logic
    }
}

public class OrderRepository
{
    public void Save()
    {
    }
}

public class InvoicePrinter
{
    public void Print()
    {
    }
}

public class EmailService
{
    public void Send()
    {
    }
}
```

Benefits:

- Easier maintenance
- Easier testing
- Independent development
- Reusable components

---

# Where is SOLID Used?

SOLID is widely used in:

- ASP.NET Core
- Java Spring Boot
- Angular
- React
- Microservices
- Azure Functions
- AWS Lambda
- Enterprise Applications
- Domain Driven Design (DDD)
- Clean Architecture
- Onion Architecture
- Hexagonal Architecture

---

# Interview Questions

### Basic

- What is SOLID?
- Who introduced SOLID?
- Why do we need SOLID?

---

### Intermediate

- Explain each SOLID principle.
- Which SOLID principle is used most frequently?
- Can you give a real-world example of SOLID?
- How does Dependency Injection relate to SOLID?
- Which design patterns support SOLID?

---

### Advanced

- Which SOLID principle does ASP.NET Core Dependency Injection support?
- How does SOLID improve Microservices?
- Can SOLID be violated intentionally?
- How does SOLID improve testability?
- Explain a production issue caused by violating SOLID.

---

# Summary

SOLID is a collection of five object-oriented design principles that help developers create software that is easier to maintain, extend, test, and scale. By assigning a single responsibility to each class, programming against abstractions, and designing components that can evolve without modifying existing code, SOLID reduces coupling, increases flexibility, and supports clean, maintainable architectures. These principles form the foundation of modern software development practices and are widely used in enterprise applications, microservices, and frameworks like ASP.NET Core.
