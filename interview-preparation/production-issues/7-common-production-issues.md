# 7 Common Production Issues Every Developer Should Know

> **Decode Tech | Production Debugging Guide**  
> Practical symptoms, root causes, Java/Spring Boot examples, diagnostics, fixes, and prevention strategies.

## Table of Contents

1. [Memory Leaks](#1-memory-leaks)
2. [Slow Database Queries](#2-slow-database-queries)
3. [Connection Pool Exhaustion](#3-connection-pool-exhaustion)
4. [API Timeouts](#4-api-timeouts)
5. [Thread Pool Exhaustion](#5-thread-pool-exhaustion)
6. [Cascading Failures](#6-cascading-failures)
7. [Retry Storms](#7-retry-storms)
8. [Troubleshooting Matrix](#8-troubleshooting-matrix)
9. [Incident Response Workflow](#9-incident-response-workflow)
10. [Useful Tools and Commands](#10-useful-tools-and-commands)
11. [Interview Questions](#11-interview-questions)
12. [Production Readiness Checklist](#12-production-readiness-checklist)

## Introduction

Code that works locally can fail in production because of concurrency, real data volumes, downstream dependencies, and limited resources. This guide focuses on **how to recognize, diagnose, mitigate, and prevent** seven frequent failure patterns.

> **Safety:** All snippets are illustrative. Test in non-production environments, review access permissions, and avoid exposing customer data in logs, traces, and heap dumps.

---

## 1. Memory Leaks

### What happens?

A memory leak occurs when an application retains references to objects it no longer needs. In Java, reachable objects cannot be reclaimed by garbage collection (GC), even if they are no longer useful.

**Symptoms:** steadily increasing *post-GC* heap usage, rising GC activity, longer pauses, `OutOfMemoryError`, or container `OOMKilled` events. High memory usage alone is not proof of a leak.

### Incident triage

1. Confirm whether the process was killed by the JVM, the container runtime, or an orchestrator; capture the termination reason and memory limit.
2. Compare **used heap after full/major GC** over several equivalent traffic windows. A leak normally shows an upward baseline; a traffic spike may not.
3. Check non-heap memory too: metaspace, direct buffers, thread stacks, native libraries, and memory-mapped files can exhaust the container while heap looks healthy.
4. Preserve evidence before restarting when safe: GC logs, JFR recording, heap histogram, pod events, and a heap dump only if capacity and data-handling rules allow it.

### Common hidden causes

- Static maps, `ThreadLocal` values, listeners, scheduled tasks, or callbacks that are never removed.
- ORM persistence contexts or HTTP response bodies retained beyond a request.
- Unbounded queues, caches, metrics labels, or trace/log buffers with high-cardinality keys.
- Direct `ByteBuffer` allocations, class-loader leaks after hot reloads, and too many live threads.

### Example: unbounded cache

```java
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class ProductCache {
    private static final Map<String, String> CACHE = new ConcurrentHashMap<>();

    public static void put(String id, String details) {
        CACHE.put(id, details); // No eviction: grows indefinitely
    }
}
```

### Diagnose

```bash
jcmd <PID> GC.heap_info
jcmd <PID> GC.heap_dump /tmp/heapdump.hprof
```

Open the dump with **Eclipse MAT** or **VisualVM** and inspect retained heap, dominator trees, and reference paths. Heap dumps can be large and contain sensitive data.

### Fix: bound and expire entries

Using the [Caffeine](https://github.com/ben-manes/caffeine) library:

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import java.time.Duration;

public class ProductCache {
    private final Cache<String, String> cache = Caffeine.newBuilder()
        .maximumSize(1_000)
        .expireAfterWrite(Duration.ofMinutes(10))
        .build();

    public void put(String id, String details) { cache.put(id, details); }
    public String get(String id) { return cache.getIfPresent(id); }
}
```

**Prevent:** bound caches, close resources, remove listeners, monitor post-GC heap, and load-test realistic workloads.

**Verify the fix:** repeat the representative workload, force/observe several GC cycles, and confirm that post-GC memory returns to a stable band without unacceptable GC pause time.

---

## 2. Slow Database Queries

### What happens?

A slow query delays API responses and can tie up database connections. Causes include missing indexes, table scans, inefficient joins, lock contention, large result sets, and N+1 query patterns.

**Severity signal:** a query can be fast in isolation yet harmful when it runs concurrently. Prioritize queries that drive p95/p99 API latency, hold locks, scan large tables, or execute very frequently.

### Triage order

1. Identify the exact normalized query, its call rate, p95/p99 duration, rows returned, and affected endpoint.
2. Determine whether time is spent executing, waiting on a lock, acquiring a connection, or transferring a large result.
3. Compare the actual execution plan with the expected plan and check changes in data distribution, schema, statistics, or deployments.
4. Check whether one request causes many queries (N+1) or repeatedly fetches the same data.

### Example

```sql
SELECT * FROM orders WHERE customer_id = 101;
```

On a large table without a useful index, this might require scanning many rows.

### Diagnose (PostgreSQL)

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT order_id, status, total_amount
FROM orders
WHERE customer_id = 101;
```

Inspect execution time, actual rows, scans, join choices, and buffer activity. `EXPLAIN ANALYZE` **executes** the statement; use care with expensive or mutating statements.

### Possible fix

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

SELECT order_id, status, total_amount
FROM orders
WHERE customer_id = 101
ORDER BY created_at DESC
LIMIT 20;
```

Recheck the execution plan after changes. Indexes increase write and storage costs, and the best index depends on the workload.

### Safe mitigation options

- Reduce the requested date range or page size; add a deterministic `ORDER BY` and cursor/keyset pagination for deep pages.
- Cancel or throttle non-critical batch jobs that compete for the same database resources.
- Add a narrowly targeted index after validating write impact and rollout/lock behavior.
- Avoid `SELECT *`; return only the columns needed by the caller.
- Treat a sudden plan regression as a possible stale-statistics or parameter-sensitivity issue, not automatically as an indexing problem.

### N+1 problem

Fetching 100 orders and then separately querying items for each order can produce **101 queries**. Consider batch fetching, suitable joins, or ORM fetch strategies.

**Prevent:** monitor p95/p99 query latency, use pagination, profile production-like datasets, and review execution plans.

**Verify the fix:** compare before/after plan, duration, buffer reads, rows scanned, lock wait, API latency, and database CPU under realistic concurrent load.

---

## 3. Connection Pool Exhaustion

### What happens?

Applications reuse database connections through a pool. If all connections are busy, new requests wait; if none becomes available before the acquisition timeout, they fail.

**Symptoms:** rising pending requests, acquisition timeouts, high active connection counts, and increased API latency even when database CPU is not saturated.

### Why it happens

Pool exhaustion is often a *secondary symptom*. Connections may be held by slow SQL, long transactions, lock waits, streaming responses, leaked resources, or too many application instances competing for a fixed database connection limit.

### Incident triage

1. Check pool `active`, `idle`, `pending`, acquisition time, and timeout count per instance.
2. Correlate the first rise in active connections with slow-query, lock-wait, deploy, traffic, or downstream events.
3. Inspect database sessions: which application/transaction is holding connections and for how long?
4. Calculate total possible connections: `maximum pool size × instance count`, plus admin, batch, and other application usage. Do not exceed the database's safe capacity.

### Example Spring Boot / HikariCP configuration

```properties
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.leak-detection-threshold=60000
```

These are **illustrative** values, not universal recommendations. Account for the number of app instances and total database connection limits.

### Diagnose

Monitor HikariCP **active**, **idle**, **pending**, **acquisition time**, and **timeout** metrics. Correlate with slow queries and transaction duration.

### Fix connection handling

```java
import java.sql.Connection;
import java.sql.SQLException;
import javax.sql.DataSource;

public class OrderRepository {
    private final DataSource dataSource;
    public OrderRepository(DataSource dataSource) { this.dataSource = dataSource; }

    public void runQuery() throws SQLException {
        try (Connection connection = dataSource.getConnection()) {
            // Execute query with a PreparedStatement here.
        } // Returns pooled connection to pool.
    }
}
```

**Prevent:** close resources, shorten transactions, fix slow queries, measure pool wait time, and tune capacity based on load. Increasing pool size without investigation can overload the database.

**Safe immediate action:** shed optional work, reduce concurrency, or scale only after checking database headroom. Raising the pool size can worsen contention if the database is already saturated.

---

## 4. API Timeouts

### What happens?

A timeout means the caller stopped waiting; it **does not necessarily mean** the downstream operation failed. This distinction is critical for payments, orders, and other side effects.

### Read a timeout correctly

| Timeout type | Meaning | Typical next check |
|---|---|---|
| DNS / connect | A connection could not be established in time | DNS, network path, TLS, target availability |
| Pool acquisition | No outbound connection was available | Client pool saturation, leaks, connection limits |
| Read / response | Connected service did not respond in time | Downstream latency, queueing, DB/dependency health |
| End-to-end deadline | The complete request exceeded its budget | Which hop consumed the budget; deadline propagation |

**Common timeouts:** connection establishment, connection acquisition, read/response, and end-to-end deadlines.

### Example

`Order Service -> Payment Service -> Payment Provider`

The provider processes a payment but its response arrives after the order service's timeout. Retrying blindly could duplicate the operation.

### Configure timeouts with Spring RestClient

```java
import java.time.Duration;
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.web.client.RestClient;

SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
factory.setConnectTimeout(Duration.ofSeconds(2));
factory.setReadTimeout(Duration.ofSeconds(3));
RestClient client = RestClient.builder().requestFactory(factory).build();
```

This configures connection and read timeouts, **not** a full end-to-end deadline.

### Diagnose and mitigate

- Inspect distributed traces and dependency p95/p99 latency.
- Check DNS/network failures, downstream health, connection pools, and recent releases.
- Set timeouts from measured service-level budgets.
- Retry only transient errors and only when the operation is safe.
- For payments, use idempotency keys and reconcile uncertain outcomes.

**Operational rule:** use a deadline budget across the call chain. The caller's timeout should leave enough time for its own work and a controlled fallback; downstream services should not have longer independent timeouts that outlive the original request.

**Verify the fix:** test normal, slow, unavailable, and partially completed dependency cases. Confirm the caller returns within its budget, does not leak work after cancellation, and produces an outcome that can be reconciled.

---

## 5. Thread Pool Exhaustion

### What happens?

All worker threads are busy or blocked, so incoming tasks queue or fail. CPU usage can remain low if threads are waiting on I/O.

### What to look for in a thread dump

| State/pattern | What it may indicate |
|---|---|
| Many threads blocked in the same HTTP/DB call | Slow or unavailable dependency; missing timeout |
| Many threads waiting for the same monitor | Lock contention or a possible deadlock |
| Queue grows while workers are active | Insufficient capacity, expensive tasks, or downstream blocking |
| Rejections increase | Executor/queue capacity has been reached; callers need backpressure behavior |

Capture two or three thread dumps 10–30 seconds apart. Repeated identical stacks identify *where work is stuck* better than a single snapshot.

### Example: blocking operation

```java
public String getProduct() throws InterruptedException {
    Thread.sleep(10_000); // Occupies the calling thread for 10 seconds
    return "Product";
}
```

A small request thread pool can be exhausted under concurrent load.

### Diagnose

```bash
jcmd <PID> Thread.print
# Alternative on supported JDKs:
jstack <PID>
```

Inspect stack traces for repeated blocking calls, locks, deadlocks, and stalled dependencies. `RUNNABLE`, `WAITING`, `TIMED_WAITING`, and `BLOCKED` are clues, not diagnoses by themselves.

### Fix

- Reduce blocking I/O and improve slow dependencies.
- Add timeouts and backpressure.
- Isolate background jobs from request workers.
- Use bounded executors and queues.
- Consider non-blocking I/O when the architecture benefits from it.

**Prevent:** monitor active threads, queue depth, rejection counts, and task duration; test concurrency limits.

**Safe immediate action:** disable optional asynchronous work, reject low-priority requests predictably, and isolate a failing dependency. Avoid replacing a bounded queue with an unbounded one; that trades fast failure for memory pressure and delayed failure.

---

## 6. Cascading Failures

### What happens?

One failing dependency consumes resources in its callers, which then degrade and affect other services.

```text
User -> Order Service -> Payment Service -> Payment Provider (unavailable)
          |                  |
          +-- threads wait --+-- requests accumulate
```

### Diagnose

Use distributed traces, dependency maps, error rates, pool utilization, and retry metrics to locate the **first failing component**.

### Failure progression

1. A dependency slows or fails.
2. Callers wait, retry, and hold threads/connections longer.
3. Queues, pools, and CPU saturate in callers.
4. Healthy features sharing those resources become slow or fail.

The remediation target is usually the shared resource boundary, not merely the service reporting the most errors.

### Mitigation patterns

| Pattern | Purpose |
|---|---|
| Timeout | Bound waiting time |
| Circuit breaker | Temporarily stop calls to an unhealthy dependency |
| Bulkhead | Isolate resource pools |
| Rate limiting | Protect limited capacity |
| Graceful degradation | Preserve unaffected functionality |

### Circuit breaker states

- **Closed:** requests flow normally.
- **Open:** calls are rejected temporarily.
- **Half-open:** limited test calls check recovery.

Illustrative Resilience4j annotation (requires the appropriate dependency, Spring integration, configuration, and supporting classes):

```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
public PaymentResponse pay(PaymentRequest request) {
    return paymentClient.pay(request);
}

public PaymentResponse fallback(PaymentRequest request, Throwable error) {
    return PaymentResponse.pendingVerification();
}
```

For payments, a fallback must **not** falsely claim failure or success if the outcome is unknown.

**Prevent:** failure injection tests, dependency isolation, bounded queues, clear fallback semantics, and monitored circuit-breaker states.

**Verify the containment:** when the dependency is unavailable, unaffected endpoints should remain within their SLOs, resource utilization should remain bounded, and circuit-breaker recovery should be gradual rather than causing a recovery surge.

---

## 7. Retry Storms

### What happens?

Clients retry failures simultaneously and amplify load on an already struggling dependency.

### Amplification checklist

- Count retries at every layer: browser/client, API gateway, service client, message consumer, job scheduler, and infrastructure proxy.
- Determine whether retries occur for the same operation concurrently or after a response was already received.
- Check whether failed work is re-queued faster than it can be processed.
- Measure *attempts per successful request*, not only request rate; this exposes hidden retry load.

If 1,000 requests each retry three additional times, the service may receive **4,000 total attempts**. Nested retries across layers can amplify this further.

### Fix: exponential backoff + jitter

Instead of retrying immediately, spread retries over time. Example delays might be ~1s, ~2s, ~4s, and ~8s, plus randomness.

```java
import java.util.concurrent.ThreadLocalRandom;

public class RetryPolicy {
    public static long fullJitterDelayMs(int attempt) {
        if (attempt < 0 || attempt > 20) {
            throw new IllegalArgumentException("attempt out of range");
        }
        long cap = 10_000L;
        long exponential = Math.min(cap, 500L * (1L << attempt));
        return ThreadLocalRandom.current().nextLong(exponential + 1);
    }
}
```

This only calculates a delay; a production retry policy also needs **max attempts, total time budget, retryable error classification, cancellation, and observability**.

### When to retry

**Potentially retryable:** transient network errors, selected `503`/`502`/`504` responses, and `429` when permitted (respect `Retry-After`).

**Usually not retryable without correction:** `400`, `401`, `403`, and other permanent failures.

### Idempotency for side effects

```http
POST /payments HTTP/1.1
Idempotency-Key: order-123-payment
Content-Type: application/json
```

The backend must atomically coordinate requests using the same logical key, persist operation state/results appropriately, and reconcile unknown payment outcomes. A key stored without concurrency control is insufficient.

**Prevent:** retry budgets, bounded attempts, jitter, safe error classification, idempotency, and avoidance of retries at every service layer.

**Safe immediate action:** reduce retry attempts, honor `Retry-After`, apply a retry budget or circuit breaker, and pause non-critical replay consumers. Preserve failed messages for controlled replay rather than immediately reprocessing them at full speed.

**Verify the fix:** confirm total attempt rate falls, downstream error rate stabilizes, success rate recovers, and no duplicate side effects are created during replay.

---

## 8. Troubleshooting Matrix

| Issue | Symptoms | First checks | Typical mitigation |
|---|---|---|---|
| Memory leak | Rising post-GC heap, OOM | Heap dump, GC logs | Remove references, bound caches |
| Slow queries | High DB/API latency | `EXPLAIN`, slow query logs | Optimize SQL and indexes |
| Pool exhaustion | Connection acquisition timeouts | Active/pending pool metrics | Fix leaks, shorten transactions |
| API timeout | Downstream wait exceeded | Traces, latency, timeout config | Budgets, safe retries |
| Thread exhaustion | Queued/stuck requests | Thread dumps, executor metrics | Reduce blocking, bound queues |
| Cascading failure | Several services degrade | Dependency traces, saturation | Circuit breakers, bulkheads |
| Retry storm | Traffic spike during outage | Retry counts, request rates | Backoff, jitter, retry limits |

---

## 9. Incident Response Workflow

1. **Assess impact:** Which customers, endpoints, and regions are affected?
2. **Check the four golden signals:** latency, traffic, errors, and saturation.
3. **Correlate logs and traces:** use trace IDs to find failing dependencies.
4. **Review changes:** deployments, configuration, feature flags, schema changes.
5. **Mitigate safely:** rollback, isolate a dependency, throttle traffic, or disable a feature as appropriate.
6. **Verify recovery:** error rates, latency, queues, and critical user journeys.
7. **Document the RCA:** timeline, root cause, contributing factors, impact, and prevention actions.

> During an incident, restore service safely before making speculative optimizations. Preserve evidence and follow your team's incident process.

---

## 10. Useful Tools and Commands

| Tool | Use |
|---|---|
| Grafana / Prometheus | Metrics and alerting |
| OpenTelemetry / Jaeger | Distributed traces |
| OpenSearch / ELK | Log investigation |
| Java Flight Recorder | JVM profiling |
| Eclipse MAT / VisualVM | Heap analysis |
| `jcmd` / `jstack` | JVM diagnostics |
| PostgreSQL `EXPLAIN` | Query planning and execution |
| `kubectl` | Kubernetes diagnostics |

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --tail=100
kubectl logs <pod-name> -n <namespace> --previous
kubectl top pods -n <namespace>  # Requires Metrics Server
```

Use approved access and protect sensitive information in logs, dumps, and traces.

---

## 11. Interview Questions

**Q1. API latency jumps from 100 ms to 5 seconds. Where do you start?**  
Check latency, traffic, errors, and saturation; then use traces to isolate DB, downstream, thread, or connection-pool bottlenecks.

**Q2. Java pods restart every few hours. What do you inspect?**  
Termination reason (`OOMKilled` or otherwise), heap and GC trends, logs, memory limits, and heap dumps.

**Q3. Database CPU is normal, but requests time out. Why?**  
Pool exhaustion, lock contention, network problems, blocked threads, or slow downstream services are possible.

**Q4. Timeout vs circuit breaker?**  
A timeout bounds how long one call waits; a circuit breaker temporarily stops calls after unhealthy behavior is detected.

**Q5. Why can retries worsen an outage?**  
They amplify traffic, especially without caps, jitter, or budgets.

**Q6. How do you prevent duplicate payment charges?**  
Use durable, concurrency-safe idempotency handling and reconcile uncertain provider outcomes.

**Q7. How do you confirm a memory leak?**  
Compare retained heap after GC under similar workloads and analyze object retention paths.

---

## 12. Production Readiness Checklist

### Reliability

- [ ] Appropriate connection/read/overall timeouts are defined.
- [ ] Retries are bounded, classified, and jittered.
- [ ] Side-effecting operations have safe idempotency behavior where required.
- [ ] Critical dependencies have isolation/fallback strategies.
- [ ] Thread pools and queues have capacity limits.

### Data layer

- [ ] Critical queries have been profiled on realistic datasets.
- [ ] Indexes and pagination have been reviewed.
- [ ] Connection pools and transaction durations are monitored.
- [ ] Database connection capacity across all instances is understood.

### Observability

- [ ] Latency, traffic, errors, and saturation dashboards exist.
- [ ] Logs include correlation IDs without leaking sensitive data.
- [ ] Distributed tracing is available for important paths.
- [ ] Alerts map to user impact and actionable runbooks.

### Recovery

- [ ] Rollback and feature-flag procedures are documented.
- [ ] Health checks and readiness probes are appropriate.
- [ ] Incident ownership and escalation are clear.
- [ ] Failure scenarios are tested periodically.

---

## Final Takeaway

> **Good engineers write code. Great engineers know how to debug, recover, and prevent production failures.**

**Decode Tech — Concepts Every Software Engineer Should Know**
