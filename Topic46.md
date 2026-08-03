# 111. Backend for Frontend (BFF)

## Introduction

**Backend for Frontend (BFF)** is an architectural pattern where **each client application has its own dedicated backend**.

Instead of all clients using the same API Gateway or backend APIs directly, each frontend (Web, Mobile, Desktop, etc.) communicates with a backend specifically designed for its needs.

> **Backend for Frontend (BFF) provides a dedicated backend optimized for a specific frontend application.**

---

# Definition

> **Backend for Frontend (BFF) is an architectural pattern where each frontend has its own backend service responsible for aggregating data, transforming responses, handling client-specific logic, and communicating with downstream microservices.**

---

# Why Do We Need BFF?

Suppose an e-commerce application has:

- Web Application
- Mobile Application
- Smart TV Application

Each frontend requires different data.

---

### Without BFF

```
                Web

                Mobile

                TV

                  │

                  ▼

             API Gateway

                  │

     ┌────────────┼────────────┐

     ▼            ▼            ▼

 Product API  Order API  Payment API
```

### Problems

- Every client receives the same API.
- Mobile downloads unnecessary data.
- UI-specific logic is duplicated across clients.
- Frequent API changes affect all clients.

---

# With BFF

```
               Web App

                  │

                  ▼

             Web BFF

                  │

      ┌───────────┼───────────┐

      ▼           ▼           ▼

 Product API  Order API  Payment API


            Mobile App

                 │

                 ▼

            Mobile BFF

                 │

      ┌──────────┼──────────┐

      ▼          ▼          ▼

 Product API  Order API  Payment API


              Smart TV

                 │

                 ▼

              TV BFF
```

Each frontend has its own optimized backend.

---

# Responsibilities of a BFF

- Aggregate data from multiple services
- Return UI-specific responses
- Authentication
- Authorization
- Response transformation
- Caching
- Client-specific validation
- Error handling
- API composition

---

# Example

## Mobile Product Page

Needs only:

- Product Name
- Price
- Image

Response

```json
{
    "name": "Laptop",
    "price": 75000,
    "image": "laptop.png"
}
```

---

## Web Product Page

Needs:

- Product
- Reviews
- Ratings
- Stock
- Similar Products
- Seller Details

Response

```json
{
    "name": "Laptop",
    "price": 75000,
    "reviews": [],
    "stock": 15,
    "seller": {},
    "relatedProducts": []
}
```

The BFF returns only the data needed by its client.

---

# Request Flow

```
Mobile App

↓

Mobile BFF

↓

Product Service

Inventory Service

Review Service

↓

Combined Response

↓

Mobile App
```

---

# API Aggregation

Without BFF

```
Mobile

↓

Product API

↓

Inventory API

↓

Review API

↓

Seller API
```

Four network calls.

---

With BFF

```
Mobile

↓

Mobile BFF

↓

All Services

↓

One Response
```

Only one network call from the client.

---

# ASP.NET Core Example

### Controller

```csharp
[ApiController]
[Route("api/mobile/products")]
public class ProductController
    : ControllerBase
{
    private readonly IProductService
        _productService;

    public ProductController(
        IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet("{id}")]
    public async Task<
        MobileProductDto>
        Get(int id)
    {
        return await
            _productService
                .GetMobileProduct(id);
    }
}
```

The mobile BFF returns a lightweight DTO.

---

# Web BFF

```text
GET /products/10

↓

Product Service

↓

Inventory Service

↓

Review Service

↓

Recommendation Service

↓

Return Combined Response
```

---

# Mobile BFF

```text
GET /products/10

↓

Product Service

↓

Return Basic Details
```

Smaller payload.

---

# Architecture

```
                Internet

                   │

             API Gateway

        ┌──────────┼───────────┐

        ▼                      ▼

     Web BFF             Mobile BFF

        │                      │

        └──────────┬───────────┘

                   ▼

             Microservices

      Product

      Order

      Inventory

      Payment
```

An API Gateway is still commonly used in front of the BFFs.

---

# BFF vs API Gateway

| API Gateway | BFF |
|--------------|-----|
| Shared by all clients | Dedicated per client |
| Infrastructure concerns | Client-specific logic |
| Routing | API composition |
| Authentication | UI optimization |
| Rate limiting | Response shaping |
| Generic | Frontend-focused |

---

# API Gateway + BFF

They are complementary, not competing patterns.

```
Client

↓

API Gateway

↓

Web BFF

↓

Microservices
```

or

```
Client

↓

API Gateway

↓

Mobile BFF

↓

Microservices
```

The API Gateway handles cross-cutting concerns, while the BFF handles client-specific requirements.

---

# BFF vs GraphQL

| BFF | GraphQL |
|------|----------|
| Backend controlled by developers | Client specifies required fields |
| Fixed endpoints | Flexible queries |
| Easier to enforce business rules | More flexible data fetching |
| Good for client-specific workflows | Good for reducing over-fetching and under-fetching |

---

# Enterprise Example

An online shopping platform has:

### Web

Needs:

- Reviews
- Inventory
- Related Products
- Seller Information
- Offers

---

### Mobile

Needs:

- Name
- Price
- Image

---

### Admin Portal

Needs:

- Supplier Details
- Sales Statistics
- Purchase History
- Inventory Trends

Each frontend has its own BFF.

---

# Benefits

- Optimized APIs for each client
- Reduced over-fetching
- Reduced under-fetching
- Better frontend performance
- Independent frontend evolution
- Cleaner separation of responsibilities

---

# Drawbacks

- More services to maintain
- Potential duplication between BFFs
- Additional deployment and monitoring overhead
- Requires clear ownership boundaries

---

# Best Practices

- Keep business logic in microservices, not in the BFF.
- Limit the BFF to orchestration, aggregation, and response transformation.
- Reuse shared libraries where appropriate.
- Keep DTOs client-specific.
- Monitor latency because a BFF often calls multiple downstream services.

---

# Common Technologies

- ASP.NET Core Web API
- Node.js (Express, NestJS)
- Spring Boot
- Azure Functions
- AWS Lambda

---

# Interview Scenario

### Scenario

An e-commerce platform supports:

- Website
- Android App
- iOS App
- Admin Portal

How would you design the backend?

### Answer

```
                API Gateway

                     │

      ┌──────────────┼──────────────┐

      ▼              ▼              ▼

   Web BFF      Mobile BFF     Admin BFF

      │              │              │

      └──────────────┼──────────────┘

                     ▼

              Microservices

 Product

 Order

 Inventory

 Payment

 Review
```

Each BFF exposes APIs tailored to its frontend while sharing the same underlying microservices.

---

# Interview Questions

### Basic

1. What is Backend for Frontend (BFF)?
2. Why is BFF used?
3. What problems does it solve?

### Intermediate

4. BFF vs API Gateway?
5. Can BFF and API Gateway be used together?
6. What responsibilities belong in a BFF?

### Advanced

7. BFF vs GraphQL?
8. How would you design BFFs for Web, Mobile, and Admin clients?
9. How do you prevent code duplication across multiple BFFs?
10. When would you avoid using the BFF pattern?

---

# Summary

The **Backend for Frontend (BFF)** pattern provides a **dedicated backend for each client application**, allowing APIs to be optimized for different frontend needs. A BFF focuses on **API composition**, **response transformation**, and **client-specific orchestration**, while leaving core business logic inside microservices. In enterprise architectures, a BFF is commonly used alongside an **API Gateway**, where the gateway handles cross-cutting concerns such as authentication and routing, and the BFF delivers optimized responses for Web, Mobile, or Admin applications. This pattern improves performance, reduces over-fetching, and enables frontend teams to evolve independently.