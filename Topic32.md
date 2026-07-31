# 79. Composite Pattern in UI

## Introduction

The **Composite Pattern** is widely used in **User Interface (UI) frameworks** because UI elements naturally form a **tree structure**.

Every screen is composed of:

- Windows
- Panels
- Layouts
- Buttons
- TextBoxes
- Labels
- Images

Using the Composite Pattern, the framework treats both **individual controls** and **containers of controls** through the same interface.

> **A button and a window can both be treated as UI components.**

---

# Why Composite in UI?

Imagine a login page.

```
Login Page

├── Header
│
├── Login Panel
│     ├── Username TextBox
│     ├── Password TextBox
│     └── Login Button
│
└── Footer
```

Without Composite:

```csharp
if(control is Button)
{
    // Render button
}
else if(control is Panel)
{
    // Render children
}
else if(control is TextBox)
{
    // Render textbox
}
```

As more controls are added, this quickly becomes difficult to maintain.

With Composite:

```csharp
control.Render();
```

Every UI component implements the same interface.

---

# UI Tree Structure

```
Window

├── Menu

│     ├── File

│     ├── Edit

│     └── View

├── Toolbar

├── Content

│     ├── Panel

│     │     ├── TextBox

│     │     ├── Label

│     │     └── Button

└── Status Bar
```

Every node is a UI component.

---

# Structure

```
           UIComponent

             ▲

      ┌──────┴───────┐

      │              │

   Control      Container

                     │

            List<UIComponent>
```

---

# Components

| Component | Responsibility |
|-----------|----------------|
| UIComponent | Common interface |
| Control | Leaf component |
| Container | Composite component |
| Client | Uses UIComponent |

---

# Example

## Step 1: Component

```csharp
public interface IUIComponent
{
    void Render();
}
```

---

## Step 2: Leaf

```csharp
public class Button
    : IUIComponent
{
    public void Render()
    {
        Console.WriteLine(
            "Render Button");
    }
}
```

---

## Step 3: Composite

```csharp
public class Panel
    : IUIComponent
{
    private readonly List<IUIComponent>
        _controls = new();

    public void Add(
        IUIComponent control)
    {
        _controls.Add(control);
    }

    public void Render()
    {
        Console.WriteLine(
            "Render Panel");

        foreach(var control in _controls)
        {
            control.Render();
        }
    }
}
```

---

## Client

```csharp
var panel = new Panel();

panel.Add(new Button());

panel.Add(new Button());

panel.Render();
```

Output

```
Render Panel

Render Button

Render Button
```

The client doesn't care whether it is rendering a **Button** or a **Panel**.

---

# Internal Workflow

```
Render()

↓

Panel

↓

Button

↓

Button

↓

TextBox

↓

Label
```

Rendering recursively traverses the UI tree.

---

# Nested Panels

Composite supports unlimited nesting.

```
Window

↓

Main Panel

↓

Login Panel

↓

Button

↓

TextBox
```

Each panel simply renders its children.

---

# Real-World UI Frameworks

## Windows Forms

```
Form

↓

Panel

↓

Button

↓

TextBox
```

Every control derives from a common base (`Control`), and container controls hold child controls.

---

## WPF

```
Window

↓

Grid

↓

StackPanel

↓

Button

↓

TextBox
```

The visual tree is a classic Composite structure.

---

## ASP.NET Web Forms

```
Page

↓

Panel

↓

UserControl

↓

Button
```

Server controls are organized hierarchically.

---

## .NET MAUI

```
ContentPage

↓

VerticalStackLayout

↓

HorizontalStackLayout

↓

Label

↓

Button
```

Layouts contain other layouts or controls, forming a tree.

---

## Blazor

```
Page

↓

Layout

↓

Component

↓

Child Component

↓

Button
```

Blazor builds a component hierarchy that closely resembles the Composite Pattern.

---

# Example: Dashboard

```
Dashboard

├── Header

├── Sidebar

│      ├── Menu

│      ├── Reports

│      └── Settings

├── Content

│      ├── Chart

│      ├── Table

│      └── Card

└── Footer
```

Calling:

```csharp
dashboard.Render();
```

automatically renders the complete UI tree.

---

# Event Propagation

Composite is useful beyond rendering.

Suppose a click occurs.

```
Window

↓

Panel

↓

Button
```

The event can propagate through the hierarchy.

Similarly, enabling or disabling a parent container can affect all its children.

---

# UI Operations Using Composite

Every UI component can expose methods like:

```csharp
Render();

Hide();

Show();

Refresh();

Resize();
```

A container simply performs the operation on itself and then recursively delegates it to its children.

---

# Composite + Iterator

Composite organizes the hierarchy.

Iterator traverses it.

```
Window

↓

Panel

↓

Button
```

Traversal can be:

- Depth-first
- Breadth-first

This separation keeps traversal logic independent of the UI structure.

---

# Composite + Decorator

Suppose every button needs logging.

```
Logging Decorator

↓

Button
```

Panels remain composites.

Buttons can be decorated individually.

The patterns work together seamlessly.

---

# Advantages

- Uniform treatment of controls and containers
- Natural representation of UI trees
- Recursive rendering
- Easy to add new control types
- Supports the Open/Closed Principle

---

# Disadvantages

- Deep hierarchies may impact rendering performance
- Traversing large trees can be expensive
- Child management needs careful design to avoid invalid structures

---

# Composite vs Container

| Composite | Container |
|-----------|-----------|
| Design Pattern | UI concept/class |
| Defines hierarchical behavior | Holds child controls |
| Can be applied in many domains | Specific implementation in UI frameworks |

A `Panel` is a container **implemented using** the Composite Pattern.

---

# Composite vs Decorator

| Composite | Decorator |
|-----------|-----------|
| Builds tree structures | Adds behavior dynamically |
| Parent owns many children | Decorator wraps one component |
| Focuses on hierarchy | Focuses on extension |

---

# Best Practices

- Model every UI element through a common abstraction.
- Let containers delegate operations recursively to their children.
- Keep leaf controls focused on rendering and interaction.
- Avoid embedding business logic directly into UI components.
- Use virtualization or incremental rendering for very large UI trees to improve performance.

---

# Interview Questions

### Basic

1. Why is the Composite Pattern useful in UI frameworks?
2. What is the difference between a leaf and a composite control?
3. How does recursive rendering work?

### Intermediate

4. How is the Composite Pattern used in WPF or .NET MAUI?
5. Why can a `Panel` and a `Button` share the same interface?
6. Composite vs Container?

### Advanced

7. How would you design a dashboard using the Composite Pattern?
8. How does Composite help with event propagation?
9. Why is the UI visual tree considered a Composite?
10. How can Composite be combined with Iterator and Decorator?

---

# Summary

The **Composite Pattern** is fundamental to modern UI frameworks because user interfaces naturally form hierarchical trees of controls and containers. By exposing a common interface for both individual controls (such as buttons and text boxes) and composite controls (such as panels, grids, and layouts), frameworks like **Windows Forms**, **WPF**, **ASP.NET Web Forms**, **Blazor**, and **.NET MAUI** enable recursive rendering, event propagation, layout management, and state updates. This approach simplifies client code, promotes extensibility, and makes complex user interfaces easier to build and maintain.