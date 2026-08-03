# 116. Circuit Breaker Pattern

## Introduction

The **Circuit Breaker Pattern** is a **Resiliency Pattern** used in **Microservices**, **Cloud Applications**, and **Distributed Systems** to prevent cascading failures when a downstream service becomes slow or unavailable.

Instead of continuously calling a failing service, the circuit breaker **temporarily stops requests**, allowing the service time to recover.

> **A Circuit Breaker fails fast instead of repeatedly calling an unhealthy service.**

---

# Definition

> **The Circuit Breaker Pattern monitors calls to a remote service. When failures exceed a configured threshold, it opens the circuit, preventing further calls until the service is likely to have recovered.**

---

# Why Do We Need Circuit Breaker?

Imagine an e-commerce system.

```
Order Service

↓

Payment Service

↓

Database
```

If the Payment Service goes down, the Order Service continues sending requests.

```
Order

↓

Payment ❌

↓

Timeout

↓

Retry

↓

Timeout

↓

Retry
```

Problems:

- Long response times
- Thread exhaustion
- Increased CPU usage
- Cascading failures
- Poor user experience

---

# Solution

Use a Circuit Breaker.

```
Order Service

↓

Circuit Breaker

↓

Payment Service
```

If the Payment Service is unhealthy, the Circuit Breaker immediately rejects requests or returns a fallback response.

---

# Real-World Analogy

Think of an **electrical circuit breaker** in your home.

```
Power Surge

↓

Circuit Breaker

↓

Power Cut Off
```

It prevents further damage.

Similarly,

```
Service Failure

↓

Circuit Breaker

↓

Stop Requests
```

It prevents the application from repeatedly calling a failing service.

---

# Circuit Breaker States

A Circuit Breaker has **three states**:

```
Closed

↓

Open

↓

Half-Open

↓

Closed
```

---

# 1. Closed State (Normal)

Everything works normally.

```
Order Service

↓

Payment Service

↓

Success
```

All requests are allowed.

---

# 2. Open State

Failures exceed the configured threshold.

```
Order Service

↓

Circuit Breaker

↓

Request Blocked

↓

Fallback
```

The downstream service is **not called**.

This is called **Fail Fast**.

---

# 3. Half-Open State

After a configured wait time, the Circuit Breaker allows a **small number of test requests**.

```
Order

↓

Payment

↓

Success?

↓

Yes → Closed

No → Open
```

If successful, normal traffic resumes.

If not, the breaker returns to the Open state.

---

# State Diagram

```
          Success

 Closed ------------→ Closed

    │

Failures

    ▼

 Open

    │

Wait Time

    ▼

Half-Open

   │      │

Success  Failure

   ▼      ▼

Closed   Open
```

---

# Internal Workflow

```
Client

↓

Circuit Breaker

↓

Service

↓

Success?

↓

Return Result

OR

Fallback
```

---

# Example Using Polly (.NET)

**Polly** is the most widely used resilience library in .NET.

```csharp
var circuitBreaker =
    Policy
        .Handle<HttpRequestException>()
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 3,
            durationOfBreak: TimeSpan.FromSeconds(30));
```

Meaning:

- After **3 consecutive failures**, the circuit opens.
- It remains open for **30 seconds**.
- After 30 seconds, it transitions to **Half-Open**.

---

# Calling a Service

```csharp
await circuitBreaker.ExecuteAsync(async () =>
{
    await httpClient.GetAsync(
        "https://payment-api");
});
```

The Circuit Breaker monitors the call automatically.

---

# Failure Timeline

```
Request 1

↓

Fail

↓

Request 2

↓

Fail

↓

Request 3

↓

Fail

↓

Circuit Opens

↓

Request 4

↓

Rejected Immediately
```

No unnecessary network call is made.

---

# Fallback Example

Instead of returning an error:

```
Payment Service

↓

Unavailable

↓

Return Cached Response
```

or

```
Return

"Payment service is temporarily unavailable."
```

This provides a better user experience.

---

# Enterprise Example

```
Customer

↓

Order Service

↓

Circuit Breaker

↓

Payment Service

↓

Database
```

If Payment fails:

```
Circuit Breaker

↓

Open

↓

Fallback Response
```

The Order Service remains responsive.

---

# Retry vs Circuit Breaker

Retries attempt to recover from **temporary failures**.

```
Request

↓

Retry

↓

Retry

↓

Success
```

Circuit Breaker prevents **continuous failures**.

```
Request

↓

Circuit Open

↓

Fail Fast
```

---

# Retry + Circuit Breaker

They are commonly used together.

```
Request

↓

Retry (2 Times)

↓

Still Fails?

↓

Open Circuit
```

Retries handle transient issues, while the Circuit Breaker protects the system from persistent failures.

---

# Timeout + Circuit Breaker

```
Request

↓

Timeout (2 sec)

↓

Failure Recorded

↓

Circuit Breaker
```

Timeout prevents long waits, and the Circuit Breaker tracks repeated failures.

---

# Bulkhead + Circuit Breaker

```
Request

↓

Bulkhead

↓

Circuit Breaker

↓

Service
```

Bulkhead limits resource usage, while the Circuit Breaker prevents repeated failures.

---

# Circuit Breaker vs Retry

| Circuit Breaker | Retry |
|-----------------|-------|
| Stops repeated failures | Attempts the request again |
| Protects downstream services | Handles transient failures |
| Opens after threshold reached | Retries immediately or after a delay |
| Fails fast | Gives the service another chance |

---

# Circuit Breaker vs Timeout

| Circuit Breaker | Timeout |
|-----------------|----------|
| Monitors repeated failures | Limits execution time |
| Maintains state | Stateless |
| Opens and blocks calls | Ends slow requests |
| Protects overall system | Protects individual requests |

---

# Circuit Breaker vs Bulkhead

| Circuit Breaker | Bulkhead |
|-----------------|----------|
| Prevents repeated calls to failing services | Isolates resource usage |
| Failure management | Resource isolation |
| Protects downstream services | Prevents one workload from consuming all resources |

---

# Circuit Breaker in Microservices

```
Customer

↓

API Gateway

↓

Order Service

↓

Circuit Breaker

↓

Payment Service
```

If Payment becomes unavailable, the Order Service remains healthy.

---

# Circuit Breaker in Service Mesh

With **Istio** or **Linkerd**, circuit breakers can often be configured at the proxy level.

```
Order Service

↓

Envoy Sidecar

↓

Payment Service
```

The application code remains unchanged.

---

# Circuit Breaker in ASP.NET Core

Using **Microsoft.Extensions.Http.Resilience** (recommended for modern .NET) or **Polly**, you can configure resilient `HttpClient` instances with:

- Retry
- Timeout
- Circuit Breaker
- Rate Limiter
- Hedging
- Fallback

---

# Common Use Cases

- Payment APIs
- Shipping providers
- Third-party REST APIs
- gRPC services
- Database connections
- Message brokers
- External SaaS integrations

---

# Benefits

- Prevents cascading failures
- Reduces unnecessary load
- Improves application responsiveness
- Supports graceful degradation
- Allows downstream services time to recover
- Improves overall system resilience

---

# Drawbacks

- Incorrect thresholds can cause unnecessary circuit openings.
- Adds configuration complexity.
- Requires monitoring and tuning.
- May temporarily reject requests even after a service has recovered if the break duration is too long.

---

# Best Practices

- Combine Circuit Breaker with Retry and Timeout.
- Use exponential backoff for retries.
- Define meaningful fallback responses.
- Monitor circuit state changes.
- Configure thresholds based on production traffic patterns.
- Avoid opening the circuit after a single transient failure.

---

# Enterprise Architecture

```
                Customer

                     │

                     ▼

               API Gateway

                     │

                     ▼

               Order Service

                     │

          Retry + Timeout

                     │

             Circuit Breaker

                     │

                     ▼

             Payment Service
```

---

# Interview Scenario

### Scenario

Your Order Service calls a third-party Payment API.

Sometimes the Payment API becomes unavailable for several minutes.

How would you prevent your application from hanging?

### Answer

Use:

- Timeout (to avoid long waits)
- Retry (for transient failures)
- Circuit Breaker (to stop repeated failures)
- Fallback (to return a graceful response)

```
Order Service

↓

Retry

↓

Circuit Breaker

↓

Payment API
```

---

# Interview Questions

### Basic

1. What is the Circuit Breaker Pattern?
2. Why is it needed?
3. What problem does it solve?

### Intermediate

4. Explain the Closed, Open, and Half-Open states.
5. Circuit Breaker vs Retry?
6. Why combine Retry and Circuit Breaker?

### Advanced

7. How would you configure a Circuit Breaker in ASP.NET Core?
8. How does Istio implement Circuit Breakers?
9. How would you determine failure thresholds?
10. Design a resilient payment integration using Retry, Timeout, Circuit Breaker, and Fallback.

---

# Summary

The **Circuit Breaker Pattern** is a key resiliency pattern for distributed systems. It monitors calls to downstream services and **opens the circuit** when repeated failures occur, preventing further requests until the service has had time to recover. By transitioning through **Closed**, **Open**, and **Half-Open** states, it protects applications from cascading failures and improves system stability. In modern .NET applications, Circuit Breakers are commonly implemented using **Microsoft.Extensions.Http.Resilience** or **Polly**, and are frequently combined with **Retry**, **Timeout**, **Fallback**, and **Bulkhead** policies to build highly resilient microservices.