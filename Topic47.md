# 112. Sidecar Pattern

## Introduction

The **Sidecar Pattern** is a **Deployment Pattern** used primarily in **Microservices**, **Kubernetes**, and **Cloud-Native Architectures**.

In this pattern, an application is deployed alongside a **companion service (called a Sidecar)** that provides supporting capabilities such as logging, monitoring, service discovery, security, or networking.

The application focuses only on business logic, while the sidecar handles infrastructure concerns.

> **The Sidecar Pattern moves common infrastructure responsibilities out of the application and into a companion process or container.**

---

# Definition

> **The Sidecar Pattern deploys a helper service alongside an application instance, allowing both to share the same lifecycle while separating business logic from cross-cutting infrastructure concerns.**

---

# Why Do We Need Sidecar?

Imagine every microservice implements:

- Logging
- Metrics
- Distributed Tracing
- Retry Logic
- TLS
- Service Discovery
- Traffic Management

Every service contains duplicate infrastructure code.

---

## Without Sidecar

```
Order Service

├── Business Logic

├── Logging

├── Metrics

├── Retry

├── TLS

├── Tracing
```

Every service repeats the same code.

---

## Problems

- Code duplication
- Difficult maintenance
- Inconsistent implementations
- Larger applications
- Harder upgrades

---

# Solution

Move common functionality into a Sidecar.

```
Order Service

↓

Sidecar

↓

Infrastructure Features
```

The application focuses only on business logic.

---

# Architecture

```
          Kubernetes Pod

+------------------------------------+

|                                    |

|   Order Service Container          |

|                                    |

|------------------------------------|

|                                    |

|      Sidecar Container             |

|                                    |

+------------------------------------+
```

Both containers run inside the **same Pod**.

---

# Shared Resources

Since both containers share a Pod, they can share:

- Network Namespace (`localhost`)
- Storage Volumes
- Lifecycle
- Configuration
- Secrets

---

# Request Flow

```
Client

↓

Sidecar

↓

Order Service

↓

Database
```

or

```
Order Service

↓

Sidecar

↓

Other Services
```

The sidecar intercepts or assists communication.

---

# Real-World Example

Suppose an application needs logging.

Without Sidecar

```
Application

↓

Log File

↓

Splunk
```

Every service implements logging.

---

With Sidecar

```
Application

↓

Log File

↓

Fluent Bit Sidecar

↓

Splunk
```

The application only writes logs.

The sidecar ships them to the logging platform.

---

# Logging Sidecar

```
Order Service

↓

application.log

↓

Fluent Bit

↓

ElasticSearch

↓

Kibana
```

The application has no knowledge of Elasticsearch.

---

# Monitoring Sidecar

```
Application

↓

Metrics

↓

Prometheus Exporter

↓

Prometheus

↓

Grafana
```

---

# Service Mesh Sidecar

One of the most common uses of the Sidecar Pattern.

```
Application

↓

Envoy Proxy

↓

Other Services
```

The application never directly communicates with other services.

---

# Istio Example

```
Client

↓

Ingress Gateway

↓

Envoy Sidecar

↓

Application

↓

Envoy Sidecar

↓

Destination Service
```

Every service has its own Envoy sidecar.

---

# Kubernetes Example

```
Pod

+-------------------------------+

| Product API                   |

|                               |

| Envoy Proxy                   |

+-------------------------------+
```

---

# Sidecar Responsibilities

- Logging
- Monitoring
- Distributed Tracing
- Authentication
- Authorization
- TLS Encryption
- Retry Policies
- Circuit Breaking
- Service Discovery
- Metrics Collection
- Configuration Refresh

---

# Sidecar vs Library

### Library

```
Application

↓

Logging Library
```

Every application contains the library.

---

### Sidecar

```
Application

↓

Sidecar
```

Infrastructure stays outside the application.

---

# Sidecar vs API Gateway

| Sidecar | API Gateway |
|----------|-------------|
| Per service instance | Single entry point |
| Internal communication | External communication |
| Runs with application | Runs independently |
| East-West traffic | North-South traffic |

---

# Sidecar vs Service Mesh

| Sidecar | Service Mesh |
|----------|--------------|
| Deployment pattern | Infrastructure architecture |
| Companion container/process | Network of sidecars |
| Single application | Entire cluster |

> A **Service Mesh** (such as **Istio** or **Linkerd**) is typically implemented **using Sidecar proxies**.

---

# Sidecar vs Ambassador Pattern

| Sidecar | Ambassador |
|----------|------------|
| General-purpose helper | Specialized proxy for outbound communication |
| Logging, metrics, security, networking | Communication with external services |
| Multiple responsibilities | Primarily networking |

---

# Enterprise Example

An e-commerce platform:

```
Order Service

↓

Envoy Sidecar

↓

Payment Service

↓

Envoy Sidecar

↓

Inventory Service

↓

Envoy Sidecar
```

Each service communicates through its sidecar.

Benefits:

- Mutual TLS (mTLS)
- Retry
- Circuit Breaker
- Observability
- Traffic Policies

No changes are required in application code.

---

# Sidecar in Kubernetes

Example Pod

```yaml
apiVersion: v1
kind: Pod

spec:

  containers:

  - name: order-api

    image: order-api

  - name: fluent-bit

    image: fluent/fluent-bit
```

Two containers share one Pod.

---

# Popular Sidecars

| Sidecar | Purpose |
|----------|----------|
| Envoy | Proxy, Service Mesh |
| Fluent Bit | Log Collection |
| Fluentd | Log Processing |
| Linkerd Proxy | Service Mesh |
| Prometheus Exporter | Metrics |
| Vault Agent | Secret Management |
| Dapr Sidecar | Distributed Application Runtime |

---

# Advantages

- Separation of concerns
- Reusable infrastructure capabilities
- Consistent implementations
- Easier upgrades
- Better observability
- Improved security
- Supports polyglot applications

---

# Disadvantages

- Additional CPU and memory usage
- More containers to monitor
- Increased deployment complexity
- Extra network hop in some scenarios
- Debugging can be more complex

---

# Best Practices

- Keep business logic inside the application.
- Move only infrastructure concerns into the sidecar.
- Monitor sidecar resource usage.
- Keep sidecars lightweight.
- Use sidecars consistently across services.

---

# Interview Scenario

### Scenario

Your microservices need:

- mTLS
- Logging
- Distributed Tracing
- Retry
- Circuit Breaking

without changing application code.

### Answer

Deploy each service with an **Envoy Sidecar** (for networking features) and, if required, a logging sidecar such as **Fluent Bit**.

```
               Kubernetes

                     │

        +-------------------------+

        | Order API               |

        | Envoy Sidecar           |

        | Fluent Bit Sidecar      |

        +-------------------------+
```

The sidecars provide networking and observability while the application remains focused on business logic.

---

# Interview Questions

### Basic

1. What is the Sidecar Pattern?
2. Why is it used?
3. What problems does it solve?

### Intermediate

4. Sidecar vs Library?
5. Sidecar vs API Gateway?
6. Why do Kubernetes Pods commonly use sidecars?

### Advanced

7. How does Istio use the Sidecar Pattern?
8. Sidecar vs Service Mesh?
9. How would you implement centralized logging using sidecars?
10. What are the trade-offs of adding sidecars to every Pod?

---

# Summary

The **Sidecar Pattern** is a cloud-native deployment pattern in which an application runs alongside a companion process or container that provides shared infrastructure capabilities such as **logging**, **monitoring**, **distributed tracing**, **security**, **service discovery**, and **traffic management**. In **Kubernetes**, sidecars typically run in the same Pod as the application and share networking and storage resources. Technologies such as **Envoy**, **Fluent Bit**, **Linkerd**, **Vault Agent**, and **Dapr** are common examples. The Sidecar Pattern promotes separation of concerns, consistency, and maintainability, and it forms the foundation of modern **Service Mesh** architectures.