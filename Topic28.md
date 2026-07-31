# 67. Composite Pattern

## Introduction

The **Composite Pattern** is a **Structural Design Pattern** that allows you to **treat individual objects and groups of objects uniformly**.

It organizes objects into a **tree structure** representing **part-whole hierarchies**.

Using Composite, a client does not need to know whether it is working with:

- A single object (Leaf)
- A collection of objects (Composite)

Both expose the same interface.

> **Composite lets clients treat individual objects and object collections in the same way.**

---

# Definition

> **The Composite Pattern composes objects into tree structures to represent part-whole hierarchies and lets clients treat individual objects and composites uniformly.**

---

# Why Do We Need Composite?

Imagine a file system.

```
Root

├── Documents
│   ├── Resume.pdf
│   └── Notes.txt
│
├── Images
│   ├── Photo1.jpg
│   └── Photo2.jpg
│
└── Video.mp4
```

A file is a single object.

A folder contains many files and folders.

The client should be able to call:

```csharp
Display();
```

on both a **File** and a **Folder**.

---

# Without Composite

```csharp
if(item is File)
{
    // Display file
}
else if(item is Folder)
{
    // Iterate children
}
```

As the hierarchy grows, the code becomes difficult to maintain.

---

# With Composite

```
Display()

↓

File

OR

Folder
```

No type checking is required.

---

# Real-World Analogy

Imagine a company structure.

```
CEO

├── Engineering Manager
│   ├── Developer A
│   └── Developer B
│
└── Sales Manager
    ├── Sales A
    └── Sales B
```

Whether it's:

- CEO
- Manager
- Employee

Everyone can implement:

```text
DisplayHierarchy()
```

---

# Structure

```
                Component

               ▲        ▲

               │        │

            Leaf    Composite

                        │

                   List<Component>
```

---

# Components

| Component | Responsibility |
|-----------|----------------|
| Component | Common interface |
| Leaf | Individual object |
| Composite | Contains child components |
| Client | Works with Component interface |

---

# Example

## Step 1: Component

```csharp
public interface IFileSystemItem
{
    void Display();
}
```

---

## Step 2: Leaf

```csharp
public class File
    : IFileSystemItem
{
    public string Name { get; }

    public File(string name)
    {
        Name = name;
    }

    public void Display()
    {
        Console.WriteLine(Name);
    }
}
```

---

## Step 3: Composite

```csharp
public class Folder
    : IFileSystemItem
{
    public string Name { get; }

    private readonly List<IFileSystemItem>
        _items = new();

    public Folder(string name)
    {
        Name = name;
    }

    public void Add(
        IFileSystemItem item)
    {
        _items.Add(item);
    }

    public void Display()
    {
        Console.WriteLine(Name);

        foreach(var item in _items)
        {
            item.Display();
        }
    }
}
```

---

## Client

```csharp
var root = new Folder("Root");

root.Add(new File("Resume.pdf"));

var images =
    new Folder("Images");

images.Add(new File("Photo1.jpg"));

images.Add(new File("Photo2.jpg"));

root.Add(images);

root.Display();
```

Output

```
Root

Resume.pdf

Images

Photo1.jpg

Photo2.jpg
```

---

# Internal Workflow

```
Display()

↓

Root Folder

↓

Resume.pdf

↓

Images Folder

↓

Photo1.jpg

↓

Photo2.jpg
```

The client only calls:

```csharp
Display();
```

The Composite traverses the hierarchy.

---

# Tree Structure

```
Folder

├── File

├── File

└── Folder

      ├── File

      └── File
```

Each node implements the same interface.

---

# Composite in Memory

```
Folder

↓

List<Component>

↓

File

↓

Folder

↓

File
```

The Composite stores references to child components.

---

# Advantages

- Uniform treatment of objects
- Supports recursive structures
- Simplifies client code
- Follows the Open/Closed Principle
- Easy to extend

---

# Disadvantages

- Harder to restrict valid child types
- Can make the design overly general
- Recursive operations can become expensive for very deep trees

---

# 68. Composite Pattern Example (ASP.NET Core)

## Example 1: Menu System

Suppose an admin portal has:

```
Dashboard

Users

Reports

Settings

Security

↓

Roles

↓

Permissions
```

Every menu item can be:

- A simple link
- A menu containing submenus

---

### Interface

```csharp
public interface IMenuItem
{
    void Render();
}
```

---

### Leaf

```csharp
public class MenuLink
    : IMenuItem
{
    private readonly string _title;

    public MenuLink(string title)
    {
        _title = title;
    }

    public void Render()
    {
        Console.WriteLine(_title);
    }
}
```

---

### Composite

```csharp
public class MenuGroup
    : IMenuItem
{
    private readonly string _title;

    private readonly List<IMenuItem>
        _items = new();

    public MenuGroup(string title)
    {
        _title = title;
    }

    public void Add(IMenuItem item)
    {
        _items.Add(item);
    }

    public void Render()
    {
        Console.WriteLine(_title);

        foreach(var item in _items)
        {
            item.Render();
        }
    }
}
```

---

### Usage

```csharp
var admin =
    new MenuGroup("Admin");

admin.Add(new MenuLink("Dashboard"));

var settings =
    new MenuGroup("Settings");

settings.Add(new MenuLink("Roles"));

settings.Add(new MenuLink("Permissions"));

admin.Add(settings);

admin.Render();
```

Output

```
Admin

Dashboard

Settings

Roles

Permissions
```

---

# Example 2: Organization Hierarchy

```
CEO

↓

Managers

↓

Employees
```

Every employee implements:

```csharp
ShowHierarchy();
```

Managers recursively call the same method on their team members.

---

# Example 3: Product Categories

```
Electronics

├── Laptop

├── Mobile

└── Accessories

      ├── Mouse

      └── Keyboard
```

Each category can contain products or subcategories.

---

# Example 4: AI Agent Workflow

```
Main Agent

├── Search Agent

├── Planner Agent

└── Coding Agent

      ├── Unit Test Agent

      └── Documentation Agent
```

Every agent exposes:

```csharp
Execute();
```

The orchestrator executes the root agent, which delegates to its children.

---

# Composite in ASP.NET Core

The Composite Pattern appears in many frameworks.

### Configuration Tree

```
Configuration

↓

Logging

↓

Console

↓

Level
```

Configuration sections form a hierarchy.

---

### Claims Hierarchy

```
Principal

↓

Identity

↓

Claims
```

Collections are traversed uniformly.

---

### UI Component Trees

```
Page

↓

Panel

↓

Button

↓

Textbox
```

Each UI component exposes a common rendering contract.

---

# Composite vs Decorator

| Composite | Decorator |
|-----------|-----------|
| Represents part-whole hierarchies | Adds behavior dynamically |
| Tree structure | Wrapper chain |
| Parent contains children | Decorator contains one component |
| Focuses on composition | Focuses on extension |

---

# Composite vs Facade

| Composite | Facade |
|-----------|---------|
| Represents hierarchical objects | Simplifies a subsystem |
| Client works with every node | Client works only with the facade |
| Recursive | Non-recursive orchestration |

---

# Composite vs Iterator

| Composite | Iterator |
|-----------|----------|
| Organizes tree structures | Traverses collections |
| Stores hierarchy | Navigates hierarchy |

Often both patterns are used together.

---

# Best Practices

- Use Composite when your domain naturally forms a tree.
- Keep both Leaf and Composite implementing the same interface.
- Delegate recursive work to child components instead of adding type checks.
- Avoid exposing child-management methods (`Add`, `Remove`) on leaf objects unless your design truly requires a transparent composite.
- Combine Composite with Iterator when traversal logic becomes complex.

---

# Common Use Cases

- File systems
- Organization charts
- Menus
- UI controls
- Product categories
- Workflow engines
- Expression trees
- AI agent hierarchies
- XML/HTML DOM trees

---

# Interview Questions

### Basic

1. What is the Composite Pattern?
2. What problem does it solve?
3. What are Leaf and Composite?

### Intermediate

4. Why is Composite useful for tree structures?
5. Composite vs Decorator?
6. Composite vs Facade?

### Advanced

7. How would you design a file explorer using the Composite Pattern?
8. How is the Composite Pattern used in UI frameworks?
9. Can Composite and Iterator be used together?
10. How would you model an AI multi-agent workflow using Composite?

---

# Summary

The **Composite Pattern** is a structural design pattern that organizes objects into **tree structures**, allowing clients to treat individual objects and groups of objects through a common interface. It is ideal for representing hierarchical domains such as file systems, organization charts, menus, product categories, workflow engines, and UI component trees. By eliminating explicit type checks and enabling recursive operations, Composite simplifies client code, promotes extensibility, and adheres to the Open/Closed Principle. In enterprise applications and ASP.NET Core, it is commonly applied wherever part-whole relationships naturally exist.