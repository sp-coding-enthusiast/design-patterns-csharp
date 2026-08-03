# 113. Ambassador Pattern

## Introduction

The **Ambassador Pattern** is a **cloud-native deployment pattern** in which a **helper proxy (Ambassador)** sits alongside an application and manages **all outbound communication** to external services.

Instead of the application communicating directly with databases, APIs, or external services, it communicates with the Ambassador, which then forwards the request.

> **An Ambassador acts as a local proxy for outbound communication, allowing the application to remain unaware of networking complexities.**

---

# Definition

> **The Ambassador Pattern deploys a proxy alongside an application to handle outbound communication, encapsulating networking concerns such as retries, load balancing, TLS, service discovery, and failover.**

---

# Why Do We Need Ambassador?

Suppose an Order Service calls:

- Payment Service
- Inventory Service
- Shipping Service

Without an Ambassador:

```
Order Service

↓

Payment Service

↓

Inventory Service

↓

Shipping Service
```

The application must implement:

- Retry logic
- Timeouts
- TLS
- Service discovery
- Load balancing
- Circuit breaker

This networking logic is duplicated across services.

---

# Solution

Introduce an Ambassador.

```
Order Service

↓

Ambassador

↓

Payment Service
```

The application communicates only with the Ambassador.

---

# Architecture

```
       Kubernetes Pod

+-----------------------------------+

|                                   |

|   Order Service                   |

|                                   |

|-----------------------------------|

|                                   |

|     Ambassador Proxy              |

|                                   |

+-----------------------------------+
```

Both containers run in the same Pod.

---

# Request Flow

```
Application

↓

localhost

↓

Ambassador

↓

External Service
```

The application usually calls `localhost`, while the Ambassador forwards requests to the actual destination.

---

# Internal Workflow

```
Order Service

↓

localhost:8080

↓

Envoy Ambassador

↓

Payment Service

↓

Response
```

The application does not know where the real Payment Service is located.

---

# Real-World Example

Suppose an application needs to call a Payment API.

Without Ambassador

```
Application

↓

HTTPS

↓

Payment API
```

Application responsibilities:

- Retry
- Timeout
- TLS
- Logging
- Circuit breaker

---

With Ambassador

```
Application

↓

localhost

↓

Ambassador

↓

Payment API
```

The Ambassador handles networking concerns.

---

# Responsibilities

An Ambassador commonly provides:

- Service Discovery
- Load Balancing
- Retry Policies
- Circuit Breaking
- TLS/mTLS
- Authentication
- Authorization
- Request Logging
- Metrics
- Traffic Routing
- Failover

---

# Retry Example

Without Ambassador

```csharp
try
{
    await paymentApi.Pay();
}
catch
{
    Retry();
}
```

Every application contains retry logic.

---

With Ambassador

```
Application

↓

Ambassador

↓

Retry Automatically

↓

Payment API
```

No retry code is required in the application.

---

# Service Discovery

Without Ambassador

```
Application

↓

payment-service.default.svc.cluster.local
```

The application needs service addresses.

---

With Ambassador

```
Application

↓

localhost

↓

Ambassador

↓

Service Discovery

↓

Payment Service
```

---

# Load Balancing

```
Application

↓

Ambassador

↓

Payment Instance 1

Payment Instance 2

Payment Instance 3
```

The Ambassador selects the target instance.

---

# Circuit Breaker

```
Application

↓

Ambassador

↓

Payment Service

↓

Failure?

↓

Open Circuit

↓

Fallback
```

The application remains unaware of circuit-breaking logic.

---

# Kubernetes Example

```
Pod

+-----------------------------+

| Order API                   |

| Envoy Proxy                 |

+-----------------------------+
```

The application communicates with the local Envoy proxy.

---

# Envoy as Ambassador

One of the most common implementations.

```
Application

↓

Envoy

↓

Payment Service
```

Envoy provides:

- Retry
- TLS
- Routing
- Metrics
- Circuit Breaker

---

# Architecture Diagram

```
Customer

↓

Order Service

↓

Envoy Ambassador

↓

Payment Service

↓

Inventory Service

↓

Shipping Service
```

The application never directly calls downstream services.

---

# Ambassador vs Sidecar

| Ambassador | Sidecar |
|-------------|----------|
| Specialized sidecar | General deployment pattern |
| Handles outbound communication | Can handle logging, monitoring, networking, security, etc. |
| Usually a proxy | Any helper process/container |
| Networking-focused | Multiple responsibilities |

> **Every Ambassador is a type of Sidecar, but not every Sidecar is an Ambassador.**

---

# Ambassador vs API Gateway

| Ambassador | API Gateway |
|-------------|-------------|
| Per application instance | Single entry point |
| Outbound traffic | Inbound traffic |
| East-West communication | North-South communication |
| Runs with the application | Runs independently |

---

# Ambassador vs Service Mesh

| Ambassador | Service Mesh |
|-------------|--------------|
| Single application | Entire cluster |
| Handles one application's outbound traffic | Manages communication between all services |
| Local proxy | Network of proxies + control plane |

---

# Enterprise Example

An Order Service calls multiple downstream services.

Without Ambassador

```
Order Service

↓

Payment

↓

Inventory

↓

Shipping
```

Each call implements:

- Retry
- TLS
- Timeout
- Logging

---

With Ambassador

```
Order Service

↓

Envoy Ambassador

↓

Payment

Inventory

Shipping
```

The proxy provides consistent networking behavior for all outbound calls.

---

# Benefits

- Removes networking logic from applications
- Consistent retry and timeout policies
- Easier service discovery
- Simplifies TLS and mTLS
- Centralized traffic management
- Better observability

---

# Drawbacks

- Additional container/process
- More CPU and memory usage
- Increased deployment complexity
- Extra network hop
- More components to monitor

---

# Best Practices

- Keep the Ambassador focused on networking concerns.
- Keep business logic in the application.
- Configure retries and timeouts carefully to avoid cascading failures.
- Monitor proxy metrics and latency.
- Use correlation IDs for distributed tracing.

---

# Common Technologies

- Envoy Proxy
- NGINX
- HAProxy
- Linkerd Proxy
- Istio Sidecar (for service mesh)
- Dapr Sidecar (communication building blocks)

---

# Interview Scenario

### Scenario

Your Order Service communicates with:

- Payment Service
- Inventory Service
- Shipping Service

Requirements:

- Retry
- TLS
- Load Balancing
- Circuit Breaker

without modifying application code.

### Answer

Deploy an **Envoy Ambassador** alongside the Order Service.

```
           Kubernetes Pod

+------------------------------+

| Order Service                |

|                              |

| Envoy Ambassador             |

+------------------------------+
```

The application sends requests to `localhost`, and Envoy manages retries, load balancing, TLS, and routing.

---

# Interview Questions

### Basic

1. What is the Ambassador Pattern?
2. Why is it used?
3. What problems does it solve?

### Intermediate

4. Ambassador vs Sidecar?
5. Ambassador vs API Gateway?
6. Why does the application usually communicate with `localhost`?

### Advanced

7. How does Envoy implement the Ambassador Pattern?
8. Ambassador vs Service Mesh?
9. How would you configure retries and circuit breakers using an Ambassador?
10. Design an outbound communication layer for a microservices application using the Ambassador Pattern.

---

# Summary

The **Ambassador Pattern** is a cloud-native deployment pattern in which a **local proxy** handles **outbound communication** for an application. It abstracts networking concerns such as **service discovery**, **load balancing**, **retries**, **timeouts**, **TLS**, and **circuit breaking**, allowing the application to focus solely on business logic. In Kubernetes, the Ambassador typically runs as a **sidecar container** using technologies such as **Envoy** or **NGINX**. It is a specialized form of the **Sidecar Pattern** and is commonly used in modern microservices and service mesh architectures to simplify and standardize service-to-service communication.