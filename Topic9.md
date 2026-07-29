# 28. Factory Method Pattern

## Introduction

The **Factory Method Pattern** is a **Creational Design Pattern** defined by the Gang of Four (GoF).

It provides an interface for creating objects but allows **subclasses** to decide which concrete class to instantiate.

Unlike the Simple Factory, where one factory class creates all objects, the Factory Method delegates object creation to subclasses.

---

# Definition

> **Define an interface for creating an object, but let subclasses decide which class to instantiate.**

---

# Why Do We Need Factory Method?

Suppose we have multiple notification providers.

Without Factory Method:

```csharp
public class NotificationFactory
{
    public INotificationService Create(string type)
    {
        switch(type)
        {
            case "Email":
                return new EmailService();

            case "SMS":
                return new SmsService();

            default:
                throw new Exception();
        }
    }
}
```

Problems:

- Factory grows with every new type.
- Violates the Open/Closed Principle.
- Difficult to maintain as the application grows.

---

# Factory Method Structure

```
                  Client
                     │
                     ▼
             NotificationFactory
                     ▲
        ┌────────────┴────────────┐
        │                         │
EmailFactory                SmsFactory
        │                         │
        ▼                         ▼
 EmailService               SmsService
```

Each factory creates only one type of object.

---

# Step 1: Product Interface

```csharp
public interface INotificationService
{
    void Send();
}
```

---

# Step 2: Concrete Products

### Email

```csharp
public class EmailService : INotificationService
{
    public void Send()
    {
        Console.WriteLine("Email Sent");
    }
}
```

---

### SMS

```csharp
public class SmsService : INotificationService
{
    public void Send()
    {
        Console.WriteLine("SMS Sent");
    }
}
```

---

# Step 3: Abstract Factory

```csharp
public abstract class NotificationFactory
{
    public abstract INotificationService CreateNotification();
}
```

---

# Step 4: Concrete Factories

### Email Factory

```csharp
public class EmailFactory : NotificationFactory
{
    public override INotificationService CreateNotification()
    {
        return new EmailService();
    }
}
```

---

### SMS Factory

```csharp
public class SmsFactory : NotificationFactory
{
    public override INotificationService CreateNotification()
    {
        return new SmsService();
    }
}
```

---

# Step 5: Client

```csharp
NotificationFactory factory = new EmailFactory();

INotificationService service =
    factory.CreateNotification();

service.Send();
```

Output:

```
Email Sent
```

---

# Adding WhatsApp

Product:

```csharp
public class WhatsAppService : INotificationService
{
    public void Send()
    {
        Console.WriteLine("WhatsApp Sent");
    }
}
```

Factory:

```csharp
public class WhatsAppFactory
    : NotificationFactory
{
    public override INotificationService CreateNotification()
    {
        return new WhatsAppService();
    }
}
```

Existing factories remain unchanged.

This satisfies the **Open/Closed Principle**.

---

# Real-World Example

Think of a **Document Editor**.

```
Document

↓

Word Document

PDF Document

Excel Document
```

Each document type has its own factory.

```
WordFactory

PdfFactory

ExcelFactory
```

The editor requests a document through the factory without knowing its concrete implementation.

---

# Factory Method in ASP.NET Core

Imagine multiple storage providers.

Interface:

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
        return Task.CompletedTask;
    }
}

public class AwsS3Storage : IFileStorage
{
    public Task UploadAsync(Stream file)
    {
        return Task.CompletedTask;
    }
}
```

Factories:

```csharp
public abstract class StorageFactory
{
    public abstract IFileStorage Create();
}

public class AzureStorageFactory : StorageFactory
{
    public override IFileStorage Create()
    {
        return new AzureBlobStorage();
    }
}

public class AwsStorageFactory : StorageFactory
{
    public override IFileStorage Create()
    {
        return new AwsS3Storage();
    }
}
```

---

# Advantages

- Supports Open/Closed Principle
- Reduces coupling
- Easier testing
- Better scalability
- Cleaner object creation

---

# Disadvantages

- More classes
- Slightly more complex
- Can introduce unnecessary abstraction for very small applications

---

# 29. Abstract Factory Pattern

## Introduction

The **Abstract Factory Pattern** is a creational design pattern that creates **families of related or dependent objects** without specifying their concrete classes.

Unlike the Factory Method, which creates **one object**, the Abstract Factory creates **multiple related objects**.

---

# Definition

> **Provide an interface for creating families of related or dependent objects without specifying their concrete classes.**

---

# Factory Method vs Abstract Factory

| Factory Method | Abstract Factory |
|----------------|------------------|
| Creates one product | Creates a family of related products |
| Uses inheritance | Uses composition |
| One factory → One product | One factory → Multiple related products |

---

# Example Problem

Suppose we are building a cross-platform UI.

Windows requires:

- Windows Button
- Windows Checkbox

macOS requires:

- Mac Button
- Mac Checkbox

We should not mix Windows buttons with Mac checkboxes.

---

# Structure

```
                 GUI Factory
                     ▲
          ┌──────────┴──────────┐
          │                     │
 WindowsFactory          MacFactory
     │     │              │     │
     ▼     ▼              ▼     ▼
Button Checkbox      Button Checkbox
```

Each factory creates a complete family of compatible objects.

---

# Step 1: Product Interfaces

```csharp
public interface IButton
{
    void Paint();
}

public interface ICheckBox
{
    void Paint();
}
```

---

# Step 2: Windows Products

```csharp
public class WindowsButton : IButton
{
    public void Paint()
    {
        Console.WriteLine("Windows Button");
    }
}
```

```csharp
public class WindowsCheckBox : ICheckBox
{
    public void Paint()
    {
        Console.WriteLine("Windows Checkbox");
    }
}
```

---

# Step 3: Mac Products

```csharp
public class MacButton : IButton
{
    public void Paint()
    {
        Console.WriteLine("Mac Button");
    }
}
```

```csharp
public class MacCheckBox : ICheckBox
{
    public void Paint()
    {
        Console.WriteLine("Mac Checkbox");
    }
}
```

---

# Step 4: Abstract Factory

```csharp
public interface IGuiFactory
{
    IButton CreateButton();

    ICheckBox CreateCheckBox();
}
```

---

# Step 5: Concrete Factories

Windows:

```csharp
public class WindowsFactory : IGuiFactory
{
    public IButton CreateButton()
    {
        return new WindowsButton();
    }

    public ICheckBox CreateCheckBox()
    {
        return new WindowsCheckBox();
    }
}
```

Mac:

```csharp
public class MacFactory : IGuiFactory
{
    public IButton CreateButton()
    {
        return new MacButton();
    }

    public ICheckBox CreateCheckBox()
    {
        return new MacCheckBox();
    }
}
```

---

# Client

```csharp
IGuiFactory factory = new WindowsFactory();

IButton button = factory.CreateButton();

ICheckBox checkbox =
    factory.CreateCheckBox();

button.Paint();

checkbox.Paint();
```

Output:

```
Windows Button

Windows Checkbox
```

---

# ASP.NET Core Example

Suppose an application supports multiple cloud providers.

Azure Family:

```
Azure Blob Storage

Azure Queue

Azure Key Vault
```

AWS Family:

```
Amazon S3

Amazon SQS

AWS Secrets Manager
```

Abstract Factory:

```csharp
public interface ICloudFactory
{
    IStorage CreateStorage();

    IQueue CreateQueue();

    ISecretManager CreateSecretManager();
}
```

Azure implementation:

```csharp
public class AzureFactory : ICloudFactory
{
    // Creates Azure-specific services
}
```

AWS implementation:

```csharp
public class AwsFactory : ICloudFactory
{
    // Creates AWS-specific services
}
```

This ensures all cloud services come from the same provider and remain compatible.

---

# Real-World Example

A furniture company sells complete furniture collections.

Modern Collection:

- Modern Chair
- Modern Sofa
- Modern Table

Victorian Collection:

- Victorian Chair
- Victorian Sofa
- Victorian Table

You choose a **collection**, not each item individually.

The collection factory guarantees that all products belong to the same family.

---

# Advantages

- Ensures compatibility between related objects.
- Supports Open/Closed Principle.
- Centralizes object creation.
- Reduces coupling.
- Makes switching entire product families easy.

---

# Disadvantages

- Large number of classes.
- More complex than Factory Method.
- Adding a new product type requires changes to all factories.

---

# Factory Pattern vs Factory Method vs Abstract Factory

| Feature | Simple Factory | Factory Method | Abstract Factory |
|---------|----------------|----------------|------------------|
| Creates | One object | One object | Family of objects |
| Object creation | Single factory class | Subclasses decide | Factory creates related products |
| Uses inheritance | No | Yes | Usually composition with multiple factory methods |
| OCP support | Limited | Excellent | Excellent |
| Complexity | Low | Medium | High |
| Typical use | Small applications | Extensible object creation | Multiple related product families |

---

# Interview Questions

### Basic

1. What is the Factory Method Pattern?
2. What is the Abstract Factory Pattern?
3. What is the difference between Factory Method and Abstract Factory?

### Intermediate

4. When would you choose Factory Method over a Simple Factory?
5. Why does Abstract Factory create product families?
6. How do these patterns support the Open/Closed Principle?

### Advanced

7. Explain Factory Method and Abstract Factory using an ASP.NET Core example.
8. How does Dependency Injection reduce the need for factories?
9. Can Factory Method and Abstract Factory be used together?
10. What are the trade-offs of using Abstract Factory in enterprise applications?

---

# Summary

The **Factory Method Pattern** delegates object creation to subclasses, making it easy to extend the system without modifying existing factories. The **Abstract Factory Pattern** goes a step further by creating **families of related objects**, ensuring that compatible components are created together. Both patterns are fundamental creational patterns that promote **loose coupling**, **extensibility**, and adherence to the **Open/Closed Principle**, and they are widely used in enterprise applications and frameworks such as ASP.NET Core.