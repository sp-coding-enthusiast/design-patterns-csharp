# 110. API Gateway

## Introduction

An **API Gateway** is a server that acts as a **single entry point** for all client requests in a distributed system, especially in a **Microservices Architecture**.

Instead of clients calling multiple microservices directly, they communicate only with the API Gateway. The gateway then routes requests to the appropriate backend services.

> **An API Gateway is the front door of a microservices architecture.**

---

# Definition

> **An API Gateway is a reverse proxy that accepts client requests, routes them to the appropriate backend services, aggregates responses when needed, and handles cross-cutting concerns such as authentication, authorization, rate limiting, logging, and caching.**

---

# Why Do We Need an API Gateway?

### Without API Gateway

```
                Client
                   │
      ┌────────────┼─────────────┐
      ▼            ▼             ▼
 Order API    Product API    Payment API
      │            │             │
      ▼            ▼             ▼
 Database     Database      Database
```

### Problems

- Client must know every service URL.
- Multiple network calls.
- Authentication implemented in every service.
- Difficult version management.
- No centralized logging or rate limiting.

---

# With API Gateway

```
                 Client
                    │
                    ▼
              API Gateway
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
 Order API     Product API    Payment API
     │              │              │
     ▼              ▼              ▼
 Database      Database      Database
```

The client only communicates with the gateway.

---

# Responsibilities of an API Gateway

- Request Routing
- Authentication
- Authorization
- SSL/TLS Termination
- Rate Limiting
- Load Balancing
- Request/Response Transformation
- Caching
- Logging
- Monitoring
- API Versioning
- Response Aggregation

---

# Request Flow

```
Client

↓

API Gateway

↓

Authentication

↓

Authorization

↓

Routing

↓

Microservice

↓

Response

↓

Client
```

---

# Example

A mobile application needs:

- Product Details
- Product Reviews
- Inventory Status

### Without Gateway

```
Mobile App

↓

Product API

↓

Review API

↓

Inventory API
```

Three HTTP requests are required.

---

### With Gateway

```
Mobile App

↓

API Gateway

↓

Product API

↓

Review API

↓

Inventory API

↓

Combined Response
```

The client sends **one request** and receives **one aggregated response**.

---

# API Aggregation

```
GET /products/100

↓

Gateway

↓

Product Service

Review Service

Inventory Service

↓

Merge Responses

↓

Return JSON
```

Example Response

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 75000,
  "reviews": [
    {
      "rating": 5
    }
  ],
  "stock": 25
}
```

---

# Authentication

Instead of every service validating a JWT token:

```
Client

↓

Gateway

↓

Validate JWT

↓

Forward Request
```

Backend services trust the gateway or perform additional authorization if required.

---

# Rate Limiting

Protect services from abuse.

```
Client

↓

Gateway

↓

100 Requests/Minute

↓

Allowed

OR

429 Too Many Requests
```

---

# API Versioning

```
/api/v1/products

/api/v2/products
```

The gateway can route each version to different backend services.

---

# Load Balancing

```
Gateway

↓

Order Service

↓

Instance 1

Instance 2

Instance 3
```

Traffic is distributed across multiple instances.

---

# Request Transformation

Client sends

```json
{
   "productId": 1
}
```

Gateway transforms it into

```json
{
   "id": 1
}
```

before forwarding to the backend.

---

# Response Transformation

Backend returns

```json
{
    "product_name": "Laptop"
}
```

Gateway converts it into

```json
{
    "name": "Laptop"
}
```

without changing the backend service.

---

# Caching

Frequently requested data can be cached.

```
Client

↓

Gateway Cache

↓

Hit?

↓

Yes

↓

Return Cached Response

↓

No

↓

Call Service
```

This reduces latency and backend load.

---

# Logging and Monitoring

The gateway can log:

- Request URL
- Response Time
- User ID
- Status Code
- Correlation ID

Example Log

```
GET /products/10

200 OK

120 ms
```

---

# API Gateway in ASP.NET Core

Popular implementations include:

- **YARP (Yet Another Reverse Proxy)** (Microsoft)
- **Ocelot**
- **NGINX**
- **Kong**
- **Traefik**
- **Azure API Management**
- **AWS API Gateway**

---

# YARP Architecture

```
Client

↓

YARP

↓

Product API

Order API

Payment API
```

---

# Ocelot Architecture

```
Client

↓

Ocelot

↓

Authentication

↓

Routing

↓

Microservices
```

---

# Azure API Management

```
Internet

↓

Azure API Management

↓

Authentication

↓

Policies

↓

Azure Functions

↓

App Services

↓

AKS
```

---

# API Gateway vs Load Balancer

| API Gateway | Load Balancer |
|--------------|--------------|
| Routes based on APIs | Distributes traffic |
| Authentication | Usually No |
| Authorization | Usually No |
| Rate Limiting | No |
| Response Transformation | No |
| API Aggregation | No |
| API Versioning | No |
| Business-aware routing | Infrastructure-level routing |

---

# API Gateway vs Reverse Proxy

| Reverse Proxy | API Gateway |
|--------------|-------------|
| Primarily forwards requests | Adds API-specific capabilities |
| Basic routing | Advanced routing |
| Limited policies | Authentication, caching, rate limiting, transformations |
| Infrastructure component | API management layer |

---

# API Gateway vs Service Mesh

| API Gateway | Service Mesh |
|--------------|-------------|
| North-South Traffic | East-West Traffic |
| Client → Services | Service → Service |
| Public APIs | Internal communication |
| External security | Internal resilience and observability |

---

# Common Interview Scenario

### Scenario

An e-commerce application has:

- Order Service
- Product Service
- Payment Service
- Inventory Service
- Notification Service

Design an architecture.

### Answer

```
                Mobile/Web App

                      │

                      ▼

               API Gateway

        ┌─────────────┼──────────────┐

        ▼             ▼              ▼

 Product API     Order API     Payment API

        │             │              │

        ▼             ▼              ▼

 Inventory     Notification     Database
```

Use the API Gateway for:

- JWT Authentication
- Routing
- Rate Limiting
- Logging
- Caching
- Response Aggregation

---

# Best Practices

- Keep the gateway lightweight; avoid business logic.
- Centralize authentication, authorization, and rate limiting.
- Use correlation IDs for distributed tracing.
- Cache only appropriate responses.
- Keep routing rules version-controlled.
- Monitor latency and error rates.

---

# Advantages

- Single entry point
- Simplified client applications
- Centralized security
- Response aggregation
- Easier monitoring
- API versioning support
- Reduced network calls

---

# Disadvantages

- Can become a single point of failure if not deployed redundantly.
- Adds an extra network hop.
- Requires scaling with traffic.
- Poor design can turn it into a bottleneck.

---

# Interview Questions

### Basic

1. What is an API Gateway?
2. Why is it used in microservices?
3. What problems does it solve?

### Intermediate

4. API Gateway vs Load Balancer?
5. API Gateway vs Reverse Proxy?
6. What is response aggregation?

### Advanced

7. How would you secure an API Gateway?
8. How does Azure API Management differ from YARP?
9. API Gateway vs Service Mesh?
10. Design an API Gateway for a high-traffic e-commerce platform.

---

# Summary

An **API Gateway** is the centralized entry point for clients in a microservices architecture. It handles **routing**, **authentication**, **authorization**, **rate limiting**, **caching**, **logging**, **API versioning**, and **response aggregation**, allowing backend services to focus on business logic. Technologies such as **YARP**, **Ocelot**, **Azure API Management**, **AWS API Gateway**, and **Kong** are widely used to implement API gateways in enterprise systems. It is one of the most frequently discussed topics in **ASP.NET Core**, **Microservices**, **Cloud**, and **System Design** interviews.