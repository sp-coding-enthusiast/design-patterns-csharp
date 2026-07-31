# 71. Flyweight Pattern

## Introduction

The **Flyweight Pattern** is a **Structural Design Pattern** that minimizes **memory usage** by **sharing common object state** among multiple objects.

Instead of creating thousands or millions of identical objects, Flyweight stores the shared (intrinsic) state once and lets multiple objects reuse it while supplying only their unique (extrinsic) state at runtime.

> **Flyweight reduces memory by sharing common data.**

---

# Definition

> **The Flyweight Pattern uses sharing to efficiently support a large number of fine-grained objects.**

---

# Why Do We Need Flyweight?

Imagine a game with **1 million trees**.

Without Flyweight:

```
Tree 1

Texture

Color

Mesh

Position

Height

↓

Tree 2

Texture

Color

Mesh

Position

Height

↓

...
```

Every tree stores the same texture and mesh.

Huge memory waste.

---

# With Flyweight

Shared data:

```
Tree Type

↓

Texture

Mesh

Color
```

Individual tree:

```
Position

Height

Rotation
```

Now:

```
1 Shared Tree

+

1 Million Positions
```

Memory usage drops dramatically.

---

# Intrinsic vs Extrinsic State

## Intrinsic State

Shared among objects.

Examples:

- Texture
- Font
- Color
- Icon
- Character shape

Stored inside the Flyweight.

---

## Extrinsic State

Unique for every object.

Examples:

- X Position
- Y Position
- Rotation
- Username
- Size

Passed during method execution.

---

# Real-World Analogy

Imagine a library.

Without sharing:

```
Student 1

Book

↓

Student 2

Book

↓

Student 3

Book
```

Three identical books.

---

With sharing:

```
One Book

↓

Borrowed by

↓

Many Students
```

The book is shared.

The borrower information is separate.

---

# Structure

```
           Client

              │

              ▼

      Flyweight Factory

              │

        Shared Objects

              ▲

              │

         Flyweight
```

---

# Components

| Component | Responsibility |
|-----------|----------------|
| Flyweight | Shared object |
| Concrete Flyweight | Stores intrinsic state |
| Flyweight Factory | Reuses existing objects |
| Client | Supplies extrinsic state |

---

# Example

## Step 1: Flyweight

```csharp
public class Character
{
    private readonly char _symbol;

    public Character(char symbol)
    {
        _symbol = symbol;
    }

    public void Display(
        int row,
        int column)
    {
        Console.WriteLine(
            $"{_symbol} at ({row}, {column})");
    }
}
```

Here:

```
Symbol

↓

Shared
```

Position is supplied later.

---

## Step 2: Factory

```csharp
public class CharacterFactory
{
    private readonly Dictionary<char,
        Character> _characters = new();

    public Character GetCharacter(
        char symbol)
    {
        if (!_characters.ContainsKey(symbol))
        {
            _characters[symbol] =
                new Character(symbol);
        }

        return _characters[symbol];
    }
}
```

---

## Step 3: Client

```csharp
var factory =
    new CharacterFactory();

var a1 =
    factory.GetCharacter('A');

var a2 =
    factory.GetCharacter('A');

Console.WriteLine(
    ReferenceEquals(a1, a2));
```

Output

```
True
```

Only one object exists for `'A'`.

---

# Internal Workflow

```
Request Character A

↓

Factory

↓

Already Exists?

↓

Yes

↓

Return Existing Object
```

If not:

```
Create

↓

Store

↓

Return
```

---

# Memory Comparison

Without Flyweight

```
100,000 Icons

↓

100,000 Objects
```

With Flyweight

```
1 Icon

↓

100,000 References
```

Huge reduction in memory.

---

# Real-World Uses

- Game engines
- Text editors
- Icons
- Fonts
- Map rendering
- GIS systems
- CAD software
- Graphics rendering

---

# Advantages

- Greatly reduces memory usage
- Improves scalability
- Reuses existing objects
- Supports large object collections
- Centralized object management

---

# Disadvantages

- More complex implementation
- Need to separate intrinsic and extrinsic state
- May slightly increase runtime due to indirection

---

# 72. Flyweight Pattern Example (ASP.NET Core)

## Example 1: Country Cache

Suppose an e-commerce application stores customer addresses.

Thousands of customers belong to the same country.

Without Flyweight:

```
Customer

↓

Country = India

↓

Customer

↓

Country = India

↓

Customer

↓

Country = India
```

Thousands of duplicate country objects.

---

With Flyweight

```
Country Factory

↓

India

↓

Shared

↓

Customer 1

Customer 2

Customer 3
```

---

### Country

```csharp
public class Country
{
    public string Name { get; }

    public Country(string name)
    {
        Name = name;
    }
}
```

---

### Factory

```csharp
public class CountryFactory
{
    private readonly Dictionary<
        string,
        Country> _countries = new();

    public Country GetCountry(
        string name)
    {
        if (!_countries.ContainsKey(name))
        {
            _countries[name] =
                new Country(name);
        }

        return _countries[name];
    }
}
```

---

### Usage

```csharp
var factory =
    new CountryFactory();

Country c1 =
    factory.GetCountry("India");

Country c2 =
    factory.GetCountry("India");

Console.WriteLine(
    ReferenceEquals(c1, c2));
```

Output

```
True
```

---

# Example 2: Product Categories

Instead of:

```
Laptop

Category

↓

Laptop

Category

↓

Laptop

Category
```

Share:

```
Electronics

↓

Shared

↓

All Products
```

---

# Example 3: Permission Objects

```
Read Permission

↓

Shared

↓

User A

User B

User C
```

Permission definitions are immutable and shared across users.

---

# Example 4: AI Token Dictionary

Suppose an LLM tokenizes text.

```
Token

↓

Embedding

↓

Shared
```

Repeated tokens reuse the same metadata instead of recreating it.

---

# Example 5: Icons in Web Applications

```
Home Icon

↓

Shared

↓

Dashboard

↓

Navigation

↓

Footer
```

One icon definition is reused in multiple places.

---

# Flyweight in .NET

### String Interning

```csharp
string a = "Hello";

string b = "Hello";

Console.WriteLine(
    ReferenceEquals(a, b));
```

Output

```
True
```

The CLR stores one shared copy of identical string literals.

---

### Memory Cache

Applications often cache immutable reference data.

```
Cache

↓

Country

↓

Currency

↓

Language
```

Rather than recreating the same objects.

---

### Dependency Injection

Singleton services are conceptually similar in that a single shared instance is reused, but **Singleton and Flyweight are different patterns**:

- **Singleton:** One instance for the application's lifetime.
- **Flyweight:** Many logical objects share common immutable state.

---

# Flyweight vs Singleton

| Flyweight | Singleton |
|-----------|-----------|
| Many shared objects | One global object |
| Optimizes memory | Controls instance count |
| Factory often manages objects | Singleton manages itself |
| Shared immutable state | Shared service instance |

---

# Flyweight vs Object Pool

| Flyweight | Object Pool |
|-----------|-------------|
| Shared simultaneously | Borrowed and returned |
| Usually immutable | Usually mutable |
| Exists for reuse by all clients | Exists for temporary exclusive use |
| Optimizes memory | Optimizes expensive object creation |

---

# Flyweight vs Prototype

| Flyweight | Prototype |
|-----------|-----------|
| Shares objects | Clones objects |
| One object reused | New object created |
| Memory optimization | Faster object creation |

---

# Best Practices

- Use Flyweight only when you have **large numbers of similar objects**.
- Keep intrinsic state immutable.
- Store shared objects in a factory or cache.
- Pass extrinsic state as method parameters.
- Avoid Flyweight if the number of objects is small, as the added complexity may not be worthwhile.

---

# Interview Questions

### Basic

1. What is the Flyweight Pattern?
2. What problem does it solve?
3. What is intrinsic state?

### Intermediate

4. What is extrinsic state?
5. How does a Flyweight Factory work?
6. Flyweight vs Singleton?

### Advanced

7. How does .NET use Flyweight internally?
8. Why is string interning considered a Flyweight example?
9. Flyweight vs Object Pool?
10. Design a map application using the Flyweight Pattern.

---

# Summary

The **Flyweight Pattern** is a structural design pattern that minimizes memory consumption by sharing immutable, common (intrinsic) state across many objects while keeping unique (extrinsic) state separate. It is especially valuable in systems that manage very large numbers of similar objects, such as game engines, text editors, map rendering, icons, and enterprise reference data. In .NET, **string interning** is a classic example of the Flyweight concept. Unlike **Singleton**, which guarantees a single service instance, or **Object Pool**, which temporarily lends reusable objects, Flyweight focuses on efficiently sharing common state to improve memory usage and scalability.