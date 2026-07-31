# 85. Proxy Pattern in EF Core

## Introduction

One of the most practical uses of the **Proxy Pattern** in .NET is found in **Entity Framework Core**.

EF Core can generate **proxy objects** instead of returning your actual entity classes. These proxies intercept property access to provide additional behavior such as:

- Lazy Loading
- Change Tracking (primarily in earlier EF versions; EF Core's default tracking does not require proxies)
- Relationship Loading

> **EF Core proxies wrap your entity objects and intercept method/property calls without changing your code.**

---

# Why EF Core Uses Proxies

Suppose you have:

```text
Customer

↓

Orders
```

When querying a customer:

```csharp
var customer =
    context.Customers.First();
```

Should EF Core immediately load:

- Customer
- Orders
- OrderItems
- Products

Loading everything can be expensive.

Instead, EF Core loads only the customer.

When you later access:

```csharp
customer.Orders
```

the proxy automatically loads the orders.

---

# Without Proxy

```
Application

↓

Customer

↓

Orders

↓

Database
```

Everything must be loaded explicitly.

---

# With Proxy

```
Application

↓

Customer Proxy

↓

Orders Accessed?

↓

Yes

↓

Database
```

---

# Internal Architecture

```
Application

↓

CustomerProxy

↓

Customer Entity

↓

DbContext

↓

Database
```

The proxy intercepts property access.

---

# Requirements

To use lazy-loading proxies:

1. Install:

```text
Microsoft.EntityFrameworkCore.Proxies
```

2. Enable proxies:

```csharp
builder.Services.AddDbContext<AppDbContext>(
    options =>
        options.UseSqlServer(connectionString)
               .UseLazyLoadingProxies());
```

3. Navigation properties must typically be `virtual`.

---

# Example Entity

```csharp
public class Customer
{
    public int Id { get; set; }

    public string Name { get; set; }

    public virtual ICollection<Order>
        Orders { get; set; }
}
```

Notice:

```csharp
virtual
```

This allows EF Core to override the property in the generated proxy.

---

# 86. Lazy Loading Proxy

## What is Lazy Loading?

Lazy loading means:

> **Load related data only when it is first accessed.**

---

# Example

```csharp
var customer =
    context.Customers.First();
```

Database query:

```sql
SELECT * FROM Customers
```

Only the customer is loaded.

Later:

```csharp
var orders =
    customer.Orders;
```

The proxy detects the access and executes another query.

```sql
SELECT * FROM Orders
WHERE CustomerId = 1
```

---

# Workflow

```
Customer Loaded

↓

Orders Requested?

↓

Yes

↓

Proxy

↓

Database
```

---

# Example

```csharp
var customer =
    context.Customers
           .First();

Console.WriteLine(
    customer.Name);

Console.WriteLine(
    customer.Orders.Count);
```

Execution:

```
Query 1

↓

Customer

↓

Access Orders

↓

Query 2
```

The second query happens automatically.

---

# Generated Proxy

Conceptually:

```csharp
CustomerProxy
    : Customer
{
    public override ICollection<Order>
        Orders
    {
        get
        {
            LoadOrders();

            return base.Orders;
        }
    }
}
```

You never write this class yourself.

---

# Advantages

- Loads only required data
- Reduces initial query cost
- Improves startup performance
- Transparent to application code

---

# Disadvantages

### N+1 Query Problem

```csharp
foreach(var customer in customers)
{
    Console.WriteLine(
        customer.Orders.Count);
}
```

If there are 100 customers:

```
1 Query

+

100 Queries
```

Total:

```
101 Queries
```

This is the famous **N+1 problem**.

---

# When to Avoid Lazy Loading

Avoid it when:

- Returning API responses
- Loading large graphs
- Generating reports
- Performance is critical

Instead use:

```csharp
.Include(x => x.Orders)
```

---

# Eager vs Lazy Loading

| Eager Loading | Lazy Loading |
|---------------|--------------|
| Loads everything upfront | Loads on demand |
| Uses `.Include()` | Uses proxies |
| Fewer database round trips | Can generate many queries |
| Predictable performance | Convenient but requires monitoring |

---

# 87. Dynamic Proxy

## Introduction

A **Dynamic Proxy** is a proxy class that is **generated at runtime** instead of being written manually.

Instead of:

```csharp
class LoggingProxy
```

the runtime creates:

```
LoggingProxy_873298
```

automatically.

---

# Why Dynamic Proxy?

Without dynamic proxies:

```
RepositoryLoggingProxy

↓

ServiceLoggingProxy

↓

PaymentLoggingProxy

↓

OrderLoggingProxy
```

Hundreds of classes.

With Dynamic Proxy:

```
Runtime

↓

Generate Proxy

↓

Intercept Calls
```

No manual proxy classes are required.

---

# Workflow

```
Client

↓

Dynamic Proxy

↓

Interceptor

↓

Real Object
```

---

# Advantages

- Less boilerplate
- Runtime generation
- Cross-cutting concerns
- Flexible interception

---

# Common Uses

- Logging
- Caching
- Validation
- Authorization
- Performance measurement
- AOP (Aspect-Oriented Programming)

---

# 88. Castle DynamicProxy

## Introduction

**Castle DynamicProxy** is one of the most popular .NET libraries for generating dynamic proxies.

It is used internally or alongside many frameworks that support interception and AOP.

---

# Architecture

```
Application

↓

Castle Proxy

↓

Interceptor

↓

Service
```

---

# Interceptor

```csharp
using Castle.DynamicProxy;

public class LoggingInterceptor
    : IInterceptor
{
    public void Intercept(
        IInvocation invocation)
    {
        Console.WriteLine("Before");

        invocation.Proceed();

        Console.WriteLine("After");
    }
}
```

---

# Creating Proxy

```csharp
var generator =
    new ProxyGenerator();

var service =
    generator.CreateInterfaceProxyWithTarget(
        new OrderService(),
        new LoggingInterceptor());
```

Workflow:

```
Client

↓

Generated Proxy

↓

Logging

↓

Order Service
```

Output

```
Before

Order Created

After
```

---

# Features

- Interface proxies
- Class proxies
- Multiple interceptors
- Runtime generation
- Method interception

---

# Real Uses

- AOP frameworks
- Logging
- Validation
- Retry policies
- Metrics
- Auditing

---

# 89. DispatchProxy

## Introduction

`DispatchProxy` is Microsoft's built-in API for creating **dynamic interface proxies**.

Unlike Castle DynamicProxy, it is included in the .NET runtime and does not require an external package.

---

# Workflow

```
Application

↓

DispatchProxy

↓

Intercept Method

↓

Real Object
```

---

# Example

```csharp
public interface IMessageService
{
    void Send(string message);
}
```

---

```csharp
public class LoggingProxy
    : DispatchProxy
{
    public IMessageService Target
    {
        get;
        set;
    }

    protected override object Invoke(
        MethodInfo method,
        object[] args)
    {
        Console.WriteLine("Before");

        var result =
            method.Invoke(Target, args);

        Console.WriteLine("After");

        return result;
    }
}
```

---

Creating the proxy:

```csharp
var proxy =
    DispatchProxy.Create<
        IMessageService,
        LoggingProxy>();
```

Every interface call passes through `Invoke()`.

---

# Castle DynamicProxy vs DispatchProxy

| Feature | Castle DynamicProxy | DispatchProxy |
|----------|---------------------|---------------|
| Included in .NET | No | Yes |
| External Package | Yes | No |
| Interface Proxies | Yes | Yes |
| Class Proxies | Yes | No (interfaces only) |
| Performance | Generally faster | Generally slower |
| Flexibility | Very High | Moderate |
| Common Use | Enterprise AOP frameworks | Lightweight interception |

---

# Dynamic Proxy vs Static Proxy

| Static Proxy | Dynamic Proxy |
|--------------|---------------|
| Written manually | Generated at runtime |
| More code | Minimal code |
| Compile-time | Runtime |
| Simple scenarios | Cross-cutting concerns |

---

# EF Core Proxy vs Dynamic Proxy

| EF Core Proxy | Dynamic Proxy |
|---------------|---------------|
| Generated for entities | Generated for services or interfaces |
| Primarily enables lazy loading | Enables interception |
| Focused on ORM behavior | General-purpose AOP |
| Requires EF Core proxy package | Uses libraries or `DispatchProxy` |

---

# Best Practices

- Use EF Core lazy-loading proxies only when on-demand loading truly benefits the application.
- Prefer eager loading (`Include`) for APIs and performance-critical queries.
- Monitor SQL generated by lazy loading to avoid the N+1 query problem.
- Use dynamic proxies for cross-cutting concerns such as logging, caching, validation, and metrics.
- Choose **Castle DynamicProxy** for advanced interception scenarios and **DispatchProxy** when you want a lightweight, built-in solution for interface interception.

---

# Interview Questions

### Basic

1. What is a Proxy in EF Core?
2. Why are navigation properties marked `virtual`?
3. What is lazy loading?

### Intermediate

4. What is the N+1 query problem?
5. What is a Dynamic Proxy?
6. Castle DynamicProxy vs DispatchProxy?

### Advanced

7. How does EF Core generate lazy-loading proxies?
8. When would you avoid lazy loading?
9. How would you implement logging using Castle DynamicProxy?
10. Why does `DispatchProxy` only support interface proxies?

---

# Summary

The **Proxy Pattern** plays an important role in the .NET ecosystem. **EF Core** uses proxy objects to enable features such as **lazy loading**, where related entities are fetched only when accessed. While this can improve initial query performance, developers must be aware of issues like the **N+1 query problem** and choose eager loading when appropriate. Beyond EF Core, **Dynamic Proxies** enable runtime method interception for cross-cutting concerns such as logging, validation, caching, authorization, and metrics. **Castle DynamicProxy** is the most feature-rich solution, supporting both interface and class proxies, while **DispatchProxy** provides a built-in .NET mechanism for generating interface-based proxies without external dependencies.