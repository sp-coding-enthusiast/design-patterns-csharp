# 115. Service Discovery

## Introduction

**Service Discovery** is a mechanism that enables microservices to **automatically find and communicate with each other** without hardcoding IP addresses or hostnames.

In modern cloud environments such as **Kubernetes**, containers are created, destroyed, and rescheduled frequently. Since their IP addresses change dynamically, services need a way to locate each other.

> **Service Discovery allows services to find each other dynamically at runtime.**

---

# Definition

> **Service Discovery is the process by which services register themselves and discover the network locations of other services automatically, eliminating the need for hardcoded endpoints.**

---

# Why Do We Need Service Discovery?

Imagine an e-commerce application with:

- Order Service
- Payment Service
- Inventory Service

Without Service Discovery:

```
Order Service

↓

192.168.1.101
```

Problems:

- IP addresses change.
- Containers restart.
- Scaling creates multiple instances.
- Configuration becomes difficult.

---

# Solution

Instead of calling an IP address, services call a logical name.

```
Order Service

↓

payment-service

↓

Actual Instance
```

The Service Discovery mechanism resolves the name to a healthy instance.

---

# Traditional Architecture

```
Order Service

↓

192.168.1.20

↓

Payment Service
```

Hardcoded IP.

---

# With Service Discovery

```
Order Service

↓

Service Registry

↓

Payment Service

↓

Instance Selected
```

The registry knows where services are running.

---

# Components

| Component | Responsibility |
|------------|----------------|
| Service Provider | Registers itself |
| Service Registry | Stores service locations |
| Service Consumer | Discovers services |
| Health Check | Removes unhealthy instances |

---

# Architecture

```
                 Service Registry

                 /      |       \

                /       |        \

        Order      Payment     Inventory

          ↑

     Client Request
```

All services register with the registry.

---

# Registration Process

```
Payment Service Starts

↓

Register

↓

Service Registry

↓

Ready
```

When the service stops:

```
Payment Service

↓

Deregister

↓

Registry Updated
```

---

# Discovery Process

```
Order Service

↓

Request Payment Service

↓

Service Registry

↓

Payment Instance

↓

Communication
```

---

# Example

Suppose three Payment Service instances exist.

```
payment-service

↓

Instance 1

Instance 2

Instance 3
```

The registry returns a healthy instance.

---

# Health Checks

Every service periodically reports its health.

```
Payment Service

↓

Health Check

↓

Registry
```

If unhealthy:

```
Registry

↓

Remove Instance
```

Future requests are routed only to healthy instances.

---

# Client-Side Service Discovery

The client asks the registry for a service location and then calls the service directly.

```
Order Service

↓

Service Registry

↓

Payment Instance

↓

Payment Service
```

Examples:

- Netflix Eureka + Ribbon
- Consul client libraries

---

# Server-Side Service Discovery

The client sends requests to a load balancer or proxy, which performs service discovery.

```
Order Service

↓

Load Balancer / Proxy

↓

Service Registry

↓

Payment Service
```

Examples:

- Kubernetes Services
- Envoy
- Istio
- AWS Elastic Load Balancer

---

# Client-Side vs Server-Side

| Client-Side | Server-Side |
|-------------|-------------|
| Client discovers service | Proxy discovers service |
| More client logic | Simpler clients |
| Registry accessed by clients | Registry hidden behind proxy |
| Common in older microservice platforms | Common in Kubernetes |

---

# Kubernetes Service Discovery

In Kubernetes, Pods have temporary IP addresses.

```
Order Pod

↓

payment-service.default.svc.cluster.local

↓

Payment Pods
```

The application calls the **Service**, not individual Pods.

---

# Kubernetes Architecture

```
Order Pod

↓

Kubernetes Service

↓

Payment Pod 1

Payment Pod 2

Payment Pod 3
```

The Service provides a stable DNS name and load balancing.

---

# DNS-Based Discovery

```
inventory-service.default.svc.cluster.local
```

Kubernetes DNS resolves this to the correct service.

Example:

```csharp
var client = new HttpClient();

await client.GetAsync(
    "http://payment-service/api/pay");
```

No IP address is required.

---

# Service Registry Technologies

| Technology | Description |
|------------|-------------|
| Kubernetes Service | Built-in service discovery |
| CoreDNS | DNS-based discovery in Kubernetes |
| HashiCorp Consul | Service registry and health checking |
| Netflix Eureka | Service registry for Spring Cloud |
| Apache ZooKeeper | Coordination and discovery |
| etcd | Distributed key-value store used by Kubernetes |

---

# Service Discovery with Consul

```
Order Service

↓

Consul

↓

Payment Service
```

Consul tracks service registration, health checks, and discovery.

---

# Service Discovery with Eureka

```
Payment Service

↓

Register

↓

Eureka Server

↓

Order Service

↓

Lookup

↓

Payment Service
```

---

# Service Discovery in Service Mesh

With a service mesh:

```
Order Service

↓

Envoy Sidecar

↓

Istio Control Plane

↓

Payment Service
```

The application simply calls the service name, while the sidecar handles discovery and routing.

---

# Service Discovery + Load Balancing

```
Order Service

↓

Service Discovery

↓

Payment 1

Payment 2

Payment 3
```

After discovering available instances, traffic is distributed across them.

---

# Enterprise Example

```
Customer

↓

Order Service

↓

payment-service

↓

Payment Pod 1

Payment Pod 2

Payment Pod 3
```

If **Payment Pod 2** crashes:

```
Registry

↓

Removes Pod 2

↓

Traffic continues to Pods 1 and 3
```

No application changes are required.

---

# Service Discovery vs API Gateway

| Service Discovery | API Gateway |
|-------------------|-------------|
| Internal service communication | External client entry point |
| Resolves service locations | Routes client requests |
| East-West traffic | North-South traffic |

---

# Service Discovery vs Load Balancer

| Service Discovery | Load Balancer |
|-------------------|---------------|
| Finds service instances | Distributes traffic |
| Maintains registry information | Selects target instance |
| Often works together with load balancing | Uses discovery information |

---

# Service Discovery vs DNS

| DNS | Service Discovery |
|------|-------------------|
| Resolves names to IPs | Discovers healthy service instances |
| Static or infrastructure-managed | Dynamic and application-aware |
| No health awareness by itself | Often includes health checks and registration |

---

# Benefits

- No hardcoded IP addresses
- Automatic scaling support
- Dynamic service registration
- Improved fault tolerance
- Easier deployments
- Supports cloud-native applications

---

# Drawbacks

- Additional infrastructure
- Registry availability is critical
- Misconfigured health checks can affect routing
- More operational complexity than static endpoints

---

# Best Practices

- Use logical service names instead of IP addresses.
- Enable health checks for all services.
- Prefer platform-native discovery (e.g., Kubernetes Services in Kubernetes).
- Combine service discovery with load balancing and retries.
- Monitor registry health and discovery latency.

---

# Interview Scenario

### Scenario

Your application has:

- 20 microservices
- Autoscaling enabled
- Kubernetes deployment

How should services communicate?

### Answer

Use **Kubernetes Service Discovery**.

```
Order Service

↓

payment-service

↓

Kubernetes Service

↓

Payment Pods
```

The application never uses Pod IP addresses. Kubernetes automatically discovers healthy Pods and load-balances requests among them.

---

# Interview Questions

### Basic

1. What is Service Discovery?
2. Why is it needed in microservices?
3. What problems does it solve?

### Intermediate

4. Client-side vs Server-side Service Discovery?
5. How does Kubernetes implement Service Discovery?
6. What is a Service Registry?

### Advanced

7. How does Istio use Service Discovery?
8. Service Discovery vs API Gateway?
9. Service Discovery vs Load Balancer?
10. Design Service Discovery for a Kubernetes-based e-commerce platform.

---

# Summary

**Service Discovery** enables microservices to locate and communicate with each other dynamically without relying on hardcoded network addresses. Services register themselves with a **Service Registry**, which keeps track of healthy instances and provides their locations to consumers. Platforms such as **Kubernetes** implement built-in service discovery using **Services** and **CoreDNS**, while tools like **Consul** and **Eureka** provide registry-based discovery. Service Discovery is a foundational component of modern cloud-native architectures, working closely with **load balancing**, **service meshes**, and **API Gateways** to provide scalable, resilient, and fault-tolerant communication.