# 117. Retry Pattern

## Introduction

The **Retry Pattern** is a **Resiliency Pattern** used in **Distributed Systems**, **Microservices**, and **Cloud Applications** to automatically retry an operation that fails due to a **transient (temporary) fault**.

Instead of immediately returning an error, the application retries the operation after a configurable delay. Many failures in distributed systems are temporary (for example, a brief network interruption or a service restart), and a retry can often succeed without any manual intervention.

> **Retry Pattern automatically retries failed operations that are expected to succeed if attempted again after a short delay.**

---

# Definition

> **The Retry Pattern detects transient failures and retries the failed operation according to a configured policy such as retry count, delay, exponential backoff, and jitter.**

---

# Why Do We Need Retry?

Consider the following architecture.

```
Customer

↓

Order Service

↓

Payment Service
```

The Payment Service may fail because of:

- Temporary network issues
- Service restart
- Database connection timeout
- DNS lookup delay
- Load balancer failover
- Cloud service throttling

Without Retry

```
Request

↓

Temporary Failure

↓

Return Error
```

Even though the service may recover in a second, the request fails.

---

# Solution

Retry the operation.

```
Request

↓

Failure

↓

Wait

↓

Retry

↓

Success
```

The user receives a successful response without noticing the temporary issue.

---

# Real-World Analogy

Imagine calling a friend.

```
Call

↓

Busy

↓

Wait

↓

Call Again

↓

Connected
```

You don't assume the call will always fail after one attempt.

---

# Retry Workflow

```
Request

↓

Call Service

↓

Success?

↓

Yes

↓

Return Result

↓

No

↓

Wait

↓

Retry

↓

Success?

↓

Yes

↓

Return Result

↓

No

↓

Maximum Retries

↓

Return Error
```

---

# Retry Strategies

## 1. Immediate Retry

Retry immediately after failure.

```
Request

↓

Fail

↓

Retry

↓

Success
```

Useful only for extremely short-lived failures.

---

## 2. Fixed Delay Retry

Wait a constant duration.

```
Retry 1

↓

Wait 2 sec

↓

Retry 2

↓

Wait 2 sec

↓

Retry 3
```

---

## 3. Incremental Retry

Increase the delay linearly.

```
Retry 1

↓

2 sec

Retry 2

↓

4 sec

Retry 3

↓

6 sec
```

---

## 4. Exponential Backoff (Recommended)

The delay doubles after each retry.

```
Retry 1

↓

1 sec

Retry 2

↓

2 sec

Retry 3

↓

4 sec

Retry 4

↓

8 sec
```

This gives the downstream service time to recover.

---

## 5. Exponential Backoff with Jitter (Best Practice)

Randomize the delay.

```
Retry 1

↓

1.3 sec

Retry 2

↓

2.5 sec

Retry 3

↓

4.8 sec
```

This prevents all clients from retrying simultaneously.

---

# Why Jitter?

Without Jitter

```
10,000 Clients

↓

Retry at Same Time

↓

Service Overloaded
```

With Jitter

```
10,000 Clients

↓

Random Retry Times

↓

Balanced Traffic
```

This prevents the **Thundering Herd Problem**.

---

# Retry Timeline

```
Attempt 1

↓

Fail

↓

1 sec

↓

Attempt 2

↓

Fail

↓

2 sec

↓

Attempt 3

↓

Success
```

---

# Retry Flow Diagram

```
Client

↓

Retry Policy

↓

Remote Service

↓

Success?

↓

Yes

↓

Return Response

↓

No

↓

Retry Until Limit
```

---

# Polly Example (.NET)

```csharp
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(
        retryCount: 3,
        sleepDurationProvider:
            retryAttempt =>
                TimeSpan.FromSeconds(
                    Math.Pow(2, retryAttempt)));
```

---

# Executing the Policy

```csharp
await retryPolicy.ExecuteAsync(async () =>
{
    await httpClient.GetAsync(
        "https://payment-api");
});
```

---

# ASP.NET Core (.NET 8/9)

```csharp
builder.Services
    .AddHttpClient<PaymentClient>()
    .AddStandardResilienceHandler();
```

The standard resilience handler provides:

- Retry
- Timeout
- Circuit Breaker
- Rate Limiter
- Hedging (when configured)

---

# Retry Configuration Example

```
Maximum Retries : 3

Delay : Exponential

Initial Delay : 1 Second

Maximum Delay : 30 Seconds

Use Jitter : Yes
```

---

# Which Errors Should Be Retried?

Retry these:

| HTTP Code | Reason |
|------------|--------|
| 408 | Request Timeout |
| 429 | Too Many Requests (respect Retry-After if present) |
| 500 | Internal Server Error (if transient) |
| 502 | Bad Gateway |
| 503 | Service Unavailable |
| 504 | Gateway Timeout |

---

# Which Errors Should NOT Be Retried?

Never retry:

| HTTP Code | Reason |
|------------|--------|
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict (depends on business scenario) |
| Validation Errors | Permanent |
| Business Rule Failures | Permanent |

---

# Retry + Timeout

```
Request

↓

Timeout (2 sec)

↓

Retry

↓

Success
```

Timeout prevents waiting forever.

Retry attempts recovery.

---

# Retry + Circuit Breaker

```
Request

↓

Retry

↓

Retry

↓

Retry

↓

Still Fails

↓

Circuit Opens
```

Retry handles temporary failures.

Circuit Breaker handles long-term failures.

---

# Retry + Fallback

```
Request

↓

Retry

↓

Still Fails

↓

Fallback

↓

Cached Response
```

---

# Retry + Bulkhead

```
Request

↓

Bulkhead

↓

Retry

↓

Service
```

Bulkhead prevents one failing dependency from exhausting all application resources.

---

# Retry in Azure

Common Azure services with built-in retry support:

- Azure Service Bus
- Azure Event Hubs
- Azure Cosmos DB
- Azure Storage
- Azure SQL
- Azure Key Vault

Most Azure SDKs include configurable retry policies.

---

# Retry in Kafka

```
Producer

↓

Kafka

↓

Consumer

↓

Failure

↓

Retry

↓

Success
```

If retries fail repeatedly:

```
Dead Letter Queue (DLQ)
```

---

# Retry in RabbitMQ

```
Message

↓

Consumer

↓

Failure

↓

Retry Queue

↓

Consumer

↓

Success
```

---

# Retry in Microservices

```
Customer

↓

API Gateway

↓

Order Service

↓

Retry

↓

Inventory Service
```

The Retry Policy shields users from temporary outages.

---

# Enterprise Example

```
Customer

↓

Order API

↓

Retry

↓

Circuit Breaker

↓

Payment API

↓

Database
```

Flow:

```
Temporary Failure

↓

Retry

↓

Recovered

↓

Success
```

If recovery does not happen:

```
Circuit Breaker

↓

Fallback

↓

Return Friendly Message
```

---

# Retry vs Circuit Breaker

| Retry | Circuit Breaker |
|--------|-----------------|
| Attempts the operation again | Stops sending requests |
| Handles transient failures | Handles persistent failures |
| Improves success rate | Protects downstream services |
| Stateless | Stateful |

---

# Retry vs Timeout

| Retry | Timeout |
|--------|----------|
| Multiple attempts | Single attempt |
| Improves success rate | Limits waiting time |
| Handles transient failures | Prevents hanging requests |

---

# Retry vs Fallback

| Retry | Fallback |
|--------|----------|
| Attempts recovery | Returns an alternative response |
| Calls downstream service | Avoids dependency after failure |

---

# Best Practices

- Retry only transient failures.
- Prefer **Exponential Backoff with Jitter**.
- Limit retry attempts.
- Respect the `Retry-After` header for HTTP 429/503 responses.
- Retry only **idempotent operations** unless duplicate handling is implemented.
- Combine Retry with Timeout and Circuit Breaker.
- Monitor retry metrics to detect unhealthy dependencies.

---

# Common Mistakes

❌ Infinite retries

❌ Retrying validation failures

❌ Retrying authentication failures

❌ Retrying every exception

❌ Very short retry intervals

❌ No timeout configured

---

# Interview Scenario

### Scenario

Your Order Service calls a Payment Service hosted in Azure.

Sometimes requests fail because the Payment Service is restarting during deployments.

How would you make the application resilient?

### Answer

Implement:

- Timeout
- Retry with Exponential Backoff and Jitter
- Circuit Breaker
- Fallback

```
Order Service

↓

Retry Policy

↓

Circuit Breaker

↓

Payment Service
```

Temporary failures are retried automatically, while persistent failures trigger the Circuit Breaker.

---

# Interview Questions

### Basic

1. What is the Retry Pattern?
2. Why is Retry needed?
3. What are transient failures?

### Intermediate

4. What is Exponential Backoff?
5. Why should Jitter be used?
6. Which HTTP status codes should be retried?

### Advanced

7. Retry vs Circuit Breaker?
8. Why should Retry only be used with idempotent operations?
9. How would you configure Retry in ASP.NET Core?
10. Design a resilient HTTP client using Timeout, Retry, Circuit Breaker, and Fallback.

---

# Summary

The **Retry Pattern** is one of the most important resiliency patterns in distributed systems. It automatically retries operations that fail because of **temporary faults**, improving reliability and user experience. The recommended strategy is **Exponential Backoff with Jitter**, which reduces load on downstream services and avoids synchronized retry storms. In modern **ASP.NET Core**, Retry is commonly implemented using **Microsoft.Extensions.Http.Resilience** or **Polly** and is typically combined with **Timeout**, **Circuit Breaker**, and **Fallback** to build highly resilient cloud-native applications.