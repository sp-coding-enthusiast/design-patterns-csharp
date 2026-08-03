# 114. Adapter Service Pattern

## Introduction

The **Adapter Service Pattern** is an architectural pattern used in **Microservices**, **Enterprise Integration**, and **Cloud Applications** to enable communication between systems that have **different APIs, protocols, data formats, or contracts**.

It is based on the **Adapter Design Pattern**, but instead of adapting classes or interfaces inside the same application, it adapts **entire services or external systems**.

> **An Adapter Service acts as a translator between two incompatible services.**

---

# Definition

> **An Adapter Service is an intermediary service that transforms requests, responses, protocols, authentication mechanisms, or data formats so that two systems with incompatible interfaces can communicate without modifying either system.**

---

# Why Do We Need an Adapter Service?

Suppose your Order Service must integrate with a third-party Payment Gateway.

Your application expects:

```json
{
    "amount": 1000,
    "currency": "INR"
}
```

The third-party API expects:

```json
{
    "totalAmount": 1000,
    "currencyCode": "INR"
}
```

The APIs are incompatible.

---

# Without Adapter

```
Order Service

↓

Payment Gateway
```

Problems:

- Third-party logic leaks into your application.
- Vendor-specific code is scattered.
- Hard to replace the payment provider.
- Every service must understand the external API.

---

# With Adapter Service

```
Order Service

↓

Payment Adapter

↓

Payment Gateway
```

The Order Service communicates only with the adapter.

---

# Architecture

```
Client

↓

Order Service

↓

Payment Adapter

↓

Stripe / Razorpay / PayPal
```

The adapter hides vendor-specific implementation details.

---

# Responsibilities

An Adapter Service can:

- Transform request formats
- Transform response formats
- Convert protocols (REST ↔ SOAP, REST ↔ gRPC)
- Translate authentication methods
- Normalize error responses
- Map data models
- Hide third-party APIs
- Handle API version differences

---

# Request Flow

```
Order Service

↓

Adapter

↓

External Service

↓

Adapter

↓

Order Service
```

The adapter translates both the request and the response.

---

# Example - Payment Adapter

Application request

```json
{
    "amount": 500,
    "currency": "USD"
}
```

Adapter converts it to

```json
{
    "totalAmount": 500,
    "currencyCode": "USD"
}
```

The payment provider processes the request.

---

# Response Transformation

Third-party response

```json
{
    "paymentStatus": "SUCCESS"
}
```

Adapter converts it into

```json
{
    "status": "Completed"
}
```

The application always receives a consistent response.

---

# Protocol Conversion

Suppose your application uses REST, but an external ERP system exposes SOAP.

```
Order Service (REST)

↓

Adapter Service

↓

SOAP ERP
```

The adapter:

- Accepts REST
- Calls SOAP
- Converts SOAP response to JSON

---

# Authentication Translation

Application

```
JWT Token
```

External service

```
API Key
```

The Adapter:

```
JWT

↓

Validate

↓

API Key

↓

External Service
```

The application never deals with the external authentication mechanism.

---

# API Version Translation

```
Application

↓

Adapter

↓

API v1

or

API v2
```

When the external provider upgrades its API, only the adapter changes.

---

# Enterprise Example

An e-commerce platform integrates with multiple payment providers.

```
Order Service

↓

Payment Adapter

↓

Stripe

↓

Razorpay

↓

PayPal
```

The Order Service uses one unified contract regardless of the payment provider.

---

# Shipping Example

```
Order Service

↓

Shipping Adapter

↓

FedEx

↓

DHL

↓

UPS
```

Each carrier exposes different APIs.

The adapter provides one consistent interface.

---

# ASP.NET Core Example

## Interface

```csharp
public interface IPaymentService
{
    Task<PaymentResponse>
        ProcessPayment(
            PaymentRequest request);
}
```

---

## Adapter

```csharp
public class StripeAdapter
    : IPaymentService
{
    public async Task<
        PaymentResponse>
        ProcessPayment(
            PaymentRequest request)
    {
        // Convert request

        // Call Stripe API

        // Convert response

        return response;
    }
}
```

---

## Client

```csharp
public class OrderService
{
    private readonly IPaymentService
        _paymentService;

    public OrderService(
        IPaymentService paymentService)
    {
        _paymentService = paymentService;
    }
}
```

The Order Service depends only on the interface.

---

# Adapter Service vs Adapter Pattern

| Adapter Pattern | Adapter Service |
|-----------------|-----------------|
| Inside an application | Between services |
| Adapts classes/interfaces | Adapts APIs and services |
| Object-oriented pattern | Distributed architecture pattern |
| Same process | Network communication |

---

# Adapter Service vs API Gateway

| Adapter Service | API Gateway |
|-----------------|-------------|
| Translates one integration | Routes many APIs |
| Handles protocol and data transformation | Handles routing, authentication, rate limiting |
| Usually service-specific | Shared entry point |

---

# Adapter Service vs Backend for Frontend (BFF)

| Adapter Service | BFF |
|-----------------|-----|
| Adapts external services | Optimizes APIs for frontends |
| Hides third-party APIs | Hides backend complexity from clients |
| Integration-focused | Client-focused |

---

# Adapter Service vs Ambassador

| Adapter Service | Ambassador |
|-----------------|------------|
| Changes request/response formats | Manages networking concerns |
| Understands business contracts | Understands transport and routing |
| Performs protocol conversion | Performs retries, TLS, load balancing |

---

# Adapter Service vs Anti-Corruption Layer (ACL)

| Adapter Service | Anti-Corruption Layer |
|-----------------|-----------------------|
| Translates APIs | Protects the domain model from external models |
| Focuses on integration | Focuses on domain isolation |
| Often part of an ACL | Broader DDD pattern that may include adapters, translators, and mappers |

---

# Benefits

- Loose coupling
- Easy provider replacement
- Consistent API for consumers
- Centralized integration logic
- Easier testing
- Reduced vendor lock-in

---

# Drawbacks

- Additional service to maintain
- Extra network hop
- Potential latency
- Requires mapping logic
- Can become complex when supporting many providers

---

# Best Practices

- Keep adapters provider-specific.
- Expose a stable internal contract.
- Do not expose third-party models to your application.
- Centralize error handling and retries.
- Add logging and monitoring around external integrations.

---

# Enterprise Architecture

```
                    Order Service

                          │

                          ▼

                 Payment Adapter

                          │

        ┌─────────────────┼─────────────────┐

        ▼                 ▼                 ▼

     Stripe          Razorpay          PayPal
```

Adding a new payment provider only requires a new adapter implementation.

---

# Interview Scenario

### Scenario

Your company currently uses Stripe but may switch to Razorpay next year.

How would you design the system?

### Answer

Create an abstraction:

```text
IPaymentService
```

Implement provider-specific adapters:

```
StripeAdapter

RazorpayAdapter

PayPalAdapter
```

The Order Service depends only on `IPaymentService`. Changing providers requires only changing the injected adapter implementation.

---

# Interview Questions

### Basic

1. What is an Adapter Service?
2. Why do we need it?
3. What problems does it solve?

### Intermediate

4. Adapter Service vs API Gateway?
5. Adapter Service vs BFF?
6. How does an Adapter Service reduce vendor lock-in?

### Advanced

7. How would you integrate a SOAP service into a REST-based microservices architecture?
8. How would you support multiple payment providers?
9. Adapter Service vs Anti-Corruption Layer?
10. Design an integration layer for an enterprise application that communicates with multiple external vendors.

---

# Summary

The **Adapter Service Pattern** extends the classic **Adapter Design Pattern** into distributed systems by acting as a translator between incompatible services. It converts **data formats**, **protocols**, **authentication mechanisms**, and **API contracts**, allowing internal services to remain independent of external systems. Adapter Services are widely used for integrating **payment gateways**, **shipping providers**, **ERP systems**, **SOAP services**, and other third-party platforms. By isolating vendor-specific logic, they improve maintainability, reduce vendor lock-in, and make enterprise integrations more resilient and easier to evolve.