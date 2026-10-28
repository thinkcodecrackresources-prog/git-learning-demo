# 90% of Developers Get These Interview Questions Wrong!

**Backend Engineering | 8 essential interview concepts | In-depth technical guide**

A practical study guide for understanding the trade-offs behind popular backend interview answers. The “90%” headline is a creative hook, not a measured statistic.

## How to use this guide

For each question, first explain the core concept in 30–60 seconds, then describe the trade-offs and give one production example. Use the follow-up questions to test depth, not memorization.

## 1. Why do we need Apache Kafka?

### Architecture and delivery details

```text
Order Service -> Kafka topic (partitioned event log)
                        |-> Inventory consumer group
                        |-> Notification consumer group
                        `-> Analytics consumer group
```

A **producer** sends records to a topic. A **broker** stores partition logs; each record has an offset within its partition. A **consumer group** shares partition assignments among its members. Multiple groups can independently read the same records. Use the order ID as a record key when per-order ordering is required, so related events usually map to the same partition.

**Failure scenario:** if a consumer stops, it can restart from committed offsets as long as the records are retained. At-least-once processing may repeat a record after a crash, so downstream effects should be idempotent. `acks=all`, suitable replication and idempotent producers strengthen delivery guarantees but do not magically make an external database update exactly once.

**Java producer example:**

```java
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.ACKS_CONFIG, "all");
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");
try (KafkaProducer<String, String> producer = new KafkaProducer<>(props)) {
    producer.send(new ProducerRecord<>("order-events", "order-101", "ORDER_CREATED")).get();
}
```

**Trade-off:** Kafka is not a drop-in replacement for synchronous queries or a relational database. Design retention, partition counts, schema evolution, poison-record handling, consumer lag alerts and retries explicitly.


**Common incomplete answer:** Kafka is just a messaging system.

**Interview-ready answer:** Kafka is a distributed, partitioned event log that lets producers and consumers exchange events asynchronously, retain and replay data, and scale processing independently.

### How it works

- Topics contain partitions; records within a partition have ordered offsets. Ordering is guaranteed within a partition, not globally across a topic.

- Producers publish records; brokers persist them according to topic retention and replication settings. Consumer groups divide partitions among members for parallel processing.

- Consumers track offsets and can replay retained records. Replay is useful for rebuilding projections, debugging and onboarding new consumers.

- Durability depends on replication, acknowledgement settings, producer behavior and cluster health; Kafka is not automatically loss-proof.

- Kafka works well for high-throughput event pipelines and decoupled services, but introduces operational complexity and eventual consistency.

### Practical example

An order service publishes OrderPlaced. Billing, inventory and analytics consume independently; analytics can replay retained events to rebuild a dashboard.

### Follow-up questions

- How do partitions affect ordering and throughput?

- What is the difference between at-most-once, at-least-once and exactly-once processing?

- How do consumer offsets and rebalances work?

**Watch out:** Kafka is not always a replacement for request-response APIs. “Exactly once” requires carefully defined boundaries and downstream idempotency.

## 2. Why is Redis so fast?

### Internal mechanics and caching pattern

```text
API -> Redis GET -> HIT  -> respond
                 -> MISS -> database -> Redis SET (TTL) -> respond
```

Redis keeps the active dataset in RAM, uses efficient specialized structures and an event-driven networking model. Many commands execute in a primary command-processing thread; modern Redis can also use I/O threads and background threads. Typical `GET`/`SET` operations are O(1), while operations on large collections can be more expensive. RAM alone does not explain its throughput.

**Example commands:**

```redis
SET product:101 '{"name":"Laptop","price":75000}' EX 600
GET product:101
HSET user:101 name Alex city Bengaluru
HGET user:101 city
```

**Cache-aside pitfalls:** stale values require TTL or invalidation; many simultaneous misses can cause a cache stampede; large keys and expensive commands can block execution; memory pressure can trigger eviction. Persistence can use RDB snapshots and/or AOF with different durability costs. Never assume Redis is automatically the system of record.


**Common incomplete answer:** Because all data is in RAM.

**Interview-ready answer:** Redis combines memory-first access, efficient purpose-built data structures, a lightweight execution path and event-driven network I/O; most commands avoid disk reads on the critical path.

### How it works

- In-memory operations remove many storage-device latency costs. Persistence can still run via snapshots or append-only files.

- The event loop multiplexes many connections efficiently. Core command execution is primarily serialized, avoiding much fine-grained locking overhead.

- Operations such as GET and SET are typically O(1), while some commands (large scans, sorting, huge collections or scripts) can be expensive.

- Redis supports strings, hashes, lists, sets, sorted sets and streams, each with different complexity and memory trade-offs.

- Network round trips, serialization, contention, memory pressure and slow commands still affect latency.

### Practical example

Cache frequently requested product metadata with TTLs. A cache hit avoids repeated database queries; handle misses and invalidation explicitly.

### Follow-up questions

- What causes cache stampedes and how do you prevent them?

- How do TTL and eviction policies differ?

- When can a Redis command block other clients?

**Watch out:** “Redis is single-threaded” is an oversimplification: Redis uses additional threads for some I/O and background work, depending on version and configuration.

## 3. Synchronous vs asynchronous: which is better?

### Example workflow and implementation

```text
Synchronous:  Checkout -> Payment API -> payment result -> checkout response
Asynchronous: Order confirmed -> event queue -> Email worker -> email sent later
```

Synchronous calls are appropriate when the caller cannot proceed without an answer. Asynchronous work is useful for long-running, bursty or noncritical operations, but it introduces queue lag, retry handling and eventual consistency. **Asynchronous does not necessarily mean parallel or faster.** A future or callback may run on an event loop; parallelism means operations actually run simultaneously.

**Java example:**

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
CompletableFuture<Void> future = CompletableFuture.runAsync(
    () -> emailService.sendConfirmation(orderId), pool);
```

An in-process future is **not durable**: queued tasks may be lost when the process crashes. For important background jobs use a durable queue, idempotent consumers, a status model (`PENDING`, `RUNNING`, `FAILED`, `COMPLETED`) and appropriate retry/dead-letter policies. An HTTP endpoint can return `202 Accepted` plus a job ID for later polling.


**Common incomplete answer:** Async is always faster or always better.

**Interview-ready answer:** Synchronous calls make the caller wait for completion; asynchronous workflows let work proceed without waiting for the final result. Choose based on latency, coupling, consistency and operational needs.

### How it works

- Synchronous request-response is straightforward when the caller immediately needs the result; its latency and availability depend on the downstream service.

- Asynchronous systems often use queues, event streams, callbacks or futures; they absorb spikes and separate producer and worker lifecycles.

- Async does not guarantee lower end-to-end completion time. It improves responsiveness and concurrency for suitable workloads.

- Async flows need correlation IDs, retries, dead-letter handling, idempotency, status tracking and explicit eventual-consistency behavior.

- Nonblocking I/O and asynchronous business workflows are related but not identical concepts.

### Practical example

A checkout API synchronously validates payment authorization, then asynchronously schedules receipt email and analytics updates.

### Follow-up questions

- How do you return status for a long-running job?

- What happens if a consumer processes a message twice?

- When is a synchronous call preferable despite higher coupling?

**Watch out:** Async adds complexity and may be unsuitable when the user must see a final, consistent result before proceeding.

## 4. What happens if one microservice goes down?

### Failure propagation and resilience

```text
API Gateway -> Order Service -> Recommendation Service (down)
                      |-> timeout -> circuit breaker OPEN -> safe fallback
                      `-> payment path continues (if independent)
```

A **timeout** bounds how long a call waits. A **retry** can recover transient errors but should have a bounded budget, backoff and jitter; retrying non-idempotent operations can duplicate effects. A **circuit breaker** transitions from CLOSED (normal) to OPEN (fail fast), then HALF-OPEN (probe recovery). A **bulkhead** isolates thread or connection capacity so a slow optional service cannot exhaust all resources. A **fallback** must be business-safe: omitting recommendations is acceptable; pretending payment succeeded is not.

**Spring/Resilience4j example:**

```java
@CircuitBreaker(name = "recommendations", fallbackMethod = "fallback")
public List<String> getRecommendations() { return client.fetch(); }
public List<String> fallback(Throwable ex) { return List.of(); }
```

A production system should also monitor latency, error rate, saturation and dependency health. For durable asynchronous processing, a broker can buffer events while consumers recover, subject to retention and capacity constraints.


**Common incomplete answer:** The entire application crashes.

**Interview-ready answer:** A well-designed distributed system isolates failures and degrades gracefully using bounded timeouts, circuit breakers, fallbacks, bulkheads, retries and queues.

### How it works

- Timeouts cap waiting; circuit breakers stop repeated calls to an unhealthy dependency and periodically probe recovery.

- Circuit breaker states are usually CLOSED (allow), OPEN (reject fast) and HALF-OPEN (test recovery). An OPEN breaker—not a CLOSED breaker—blocks requests.

- Retries should be bounded, jittered and used only when safe; unbounded retries can amplify outages.

- Bulkheads limit resource usage per dependency; fallbacks offer reduced functionality where business semantics allow.

- Message queues buffer work but do not make it disappear; monitoring, dead-letter handling and recovery are essential.

### Practical example

If recommendations are unavailable, a product page can still show product details with an empty recommendation panel rather than fail the entire page.

### Follow-up questions

- When does retrying make an outage worse?

- What is the difference between circuit breaker and bulkhead?

- How would you design a fallback for a payment dependency?

**Watch out:** Graceful degradation is not automatic: critical dependencies can still make the user journey unavailable.

## 5. REST vs gRPC: what should you choose?

### Protocol and contract examples

REST commonly models resources using HTTP semantics:

```http
GET /products/101
Accept: application/json
```

```json
{"id":101,"name":"Laptop","price":75000}
```

gRPC typically defines typed methods in Protocol Buffers and transports them over HTTP/2:

```protobuf
syntax = "proto3";
service ProductService {
  rpc GetProduct(ProductRequest) returns (ProductResponse);
}
message ProductRequest { int64 product_id = 1; }
message ProductResponse { int64 product_id = 1; string name = 2; }
```

**gRPC supports** unary, client-streaming, server-streaming and bidirectional-streaming RPCs. Protobuf is compact and code generation helps contract consistency, but it is less human-readable than JSON and browser access often needs gRPC-Web or a compatible gateway. REST does **not** require JSON, and gRPC is **not** guaranteed to outperform REST for every workload. Choose using latency benchmarks, client interoperability, API lifecycle and operational tooling.


**Common incomplete answer:** REST uses JSON, gRPC uses Protobuf.

**Interview-ready answer:** REST is an architectural style for resource-oriented APIs; gRPC is an RPC framework with strongly defined service contracts, commonly using Protocol Buffers and HTTP/2.

### How it works

- REST is not restricted to JSON or HTTP/1.1; it can use other representations and modern HTTP versions.

- gRPC commonly uses generated clients, binary Protobuf payloads, streaming and HTTP/2 transport.

- Contract-first gRPC can improve type safety and tooling, while REST is often convenient for public web APIs and human-friendly debugging.

- gRPC may improve latency or payload size in some workloads, but real performance depends on message sizes, network, server implementation and workload.

- Browser gRPC usage often requires gRPC-Web or another gateway; public API interoperability may favor REST.

### Practical example

Use REST for an externally exposed product catalog and gRPC for strongly typed internal calls between inventory and pricing services, if the ecosystem supports it.

### Follow-up questions

- What are unary, server-streaming and bidirectional-streaming RPCs?

- Can REST run over HTTP/2?

- How do you version protobuf schemas safely?

**Watch out:** Neither approach is universally faster, and REST is not a wire protocol competing directly with gRPC.

## 6. HashMap vs ConcurrentHashMap in Java

### Thread safety is not the same as atomic workflows

A `HashMap` uses hashed buckets with linked nodes and sometimes tree bins; it permits one null key and null values. It is unsafe for unsynchronized concurrent writes. Modern `ConcurrentHashMap` uses CAS, visibility guarantees and fine-grained synchronization, disallows null keys/values, and supports weakly consistent iteration.

```java
ConcurrentHashMap<String, Integer> counts = new ConcurrentHashMap<>();
counts.put("orders", 0);
// WRONG as a compound atomic increment under concurrency:
counts.put("orders", counts.get("orders") + 1);
// RIGHT for a single-key atomic update:
counts.merge("orders", 1, Integer::sum);
```

For very hot counters consider `ConcurrentHashMap<String, LongAdder>` and `computeIfAbsent`. `Collections.synchronizedMap(...)` commonly serializes operations through a single monitor; ConcurrentHashMap permits more concurrent activity. In single-threaded code, HashMap may be simpler and faster.


**Common incomplete answer:** ConcurrentHashMap is just a synchronized HashMap.

**Interview-ready answer:** HashMap is not safe for unsynchronized concurrent mutation. ConcurrentHashMap supports concurrent access with atomic operations, volatile/CAS techniques and fine-grained synchronization where needed.

### How it works

- HashMap allows one null key and multiple null values; ConcurrentHashMap rejects null keys and values.

- ConcurrentHashMap was introduced in Java 5; modern implementations differ substantially from older segmented-lock designs.

- In modern Java, reads are generally nonblocking and updates use CAS and per-bin synchronization where necessary.

- Operations such as putIfAbsent, compute and merge offer atomic per-key behaviors; a separate get-then-put sequence is not equivalent.

- Iterators are weakly consistent rather than fail-fast snapshots; compound invariants across keys still need additional coordination.

### Practical example

For a shared per-user request counter, use ConcurrentHashMap<String, LongAdder> with computeIfAbsent rather than unsafely updating a regular HashMap.

### Follow-up questions

- Why does ConcurrentHashMap reject null?

- What is wrong with containsKey followed by put?

- Are its iterators snapshots?

**Watch out:** “Bucket-level locking” is a helpful shorthand, but modern implementations combine CAS, volatile reads and synchronized bins; it is not one lock per bucket at all times.

## 7. SQL vs NoSQL: which database is better?

### Data modeling, ACID and CAP

Relational data example:

```sql
SELECT c.name, o.order_id, o.amount
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id
WHERE c.customer_id = 101;
```

A document database might embed a customer's recent orders in one document for read convenience, but unbounded embedded arrays can cause document growth and update contention. NoSQL includes document (MongoDB), key-value (Redis), wide-column (Cassandra) and graph (Neo4j) models. Several NoSQL databases support ACID transactions; relational databases can also scale horizontally.

**ACID:** Atomicity (all-or-nothing), Consistency (constraints preserved), Isolation (concurrent transaction semantics), Durability (committed changes survive according to guarantees).

**CAP nuance:** during a network partition, a distributed system cannot simultaneously guarantee both strong consistency and availability for all partitioned operations. CAP is not a universal instruction to "pick any two" at all times.

**Production example:** PostgreSQL for payments/orders, a document database for flexible catalog content, Redis for caching. Choose based on query patterns, correctness, operational cost and consistency—not database fashion.


**Common incomplete answer:** NoSQL is always faster and scales horizontally.

**Interview-ready answer:** Choose by access patterns, data model, query needs, transaction semantics, operational maturity and scale. Both relational and non-relational databases can be highly scalable.

### How it works

- Relational systems model entities and relationships with schemas, constraints, joins and transactions; PostgreSQL and MySQL are common examples.

- NoSQL covers multiple models: document, key-value, wide-column and graph. These systems have very different capabilities.

- Many SQL databases support replication, partitioning and sharding. Many NoSQL databases support strong consistency and transactions.

- Indexes, query patterns, cardinality, hot keys and operational design often matter more than the SQL/NoSQL label.

- Consider durability, consistency, migration needs, analytical queries, developer productivity and total cost.

### Practical example

A banking ledger may benefit from relational constraints and multi-row transactions; a high-volume session store may fit a key-value database.

### Follow-up questions

- What is the difference between ACID and eventual consistency?

- How would you choose partition keys?

- When do joins help versus denormalized documents?

**Watch out:** Do not equate SQL with vertical-only scaling or NoSQL with eventual consistency: these are implementation-specific trade-offs.

## 8. What is a deadlock?

### Four conditions and Java demonstration

```text
Thread 1 holds A -> waits for B
Thread 2 holds B -> waits for A
```

Classic deadlock requires **mutual exclusion**, **hold and wait**, **no preemption**, and **circular wait**. A blocked thread alone is not necessarily deadlocked. A livelock has activity without useful progress; starvation is repeated denial of access to a resource.

```java
Object a = new Object(), b = new Object();
Thread t1 = new Thread(() -> {
    synchronized (a) { synchronized (b) { System.out.println("T1"); } }
});
Thread t2 = new Thread(() -> {
    synchronized (b) { synchronized (a) { System.out.println("T2"); } }
});
t1.start();
t2.start();
```

This code has a **possible**, not guaranteed, deadlock because scheduling determines lock acquisition. To prevent it, acquire locks in a consistent global order, keep critical sections small, avoid blocking I/O while holding locks, or use timed `ReentrantLock.tryLock` with correct release logic. To diagnose Java-level deadlocks, inspect thread dumps (`jstack`) or use `ThreadMXBean.findDeadlockedThreads()`.


**Common incomplete answer:** A thread is blocked.

**Interview-ready answer:** Deadlock occurs when a set of threads or processes cannot proceed because each waits for resources or actions held by others in a dependency cycle.

### How it works

- Four classic necessary conditions are mutual exclusion, hold-and-wait, no preemption and circular wait.

- Example: Thread A holds lock X and waits for Y; Thread B holds Y and waits for X.

- Consistent global lock ordering breaks circular wait. Timed lock acquisition and rollback can limit indefinite waiting.

- Deadlock detection can identify wait-for cycles; recovery may require aborting work or restarting a process.

- Deadlock differs from starvation (never scheduled/granted resources) and livelock (active but no useful progress).

### Practical example

Two account-transfer threads lock accounts in opposite orders. Sorting account IDs and always acquiring the smaller ID first avoids this lock-order deadlock.

### Follow-up questions

- What are the four Coffman conditions?

- How do you diagnose a Java deadlock?

- What is the difference between deadlock, starvation and livelock?

**Watch out:** Blocking alone is not deadlock. A waiting thread may be progressing normally once another thread releases a resource.

## Quick revision: 30-second answers

**Kafka:** Partitioned, replayable distributed event log for decoupled event processing.

**Redis:** Fast memory-first data structures plus efficient event-driven execution.

**Sync vs async:** Choose immediate results vs decoupled work based on consistency and latency needs.

**Microservice outage:** Isolate failures with timeouts, open circuit breakers, bulkheads and safe fallbacks.

**REST vs gRPC:** Resource-oriented architectural style vs typed RPC framework; choose for consumers and workloads.

**HashMap vs ConcurrentHashMap:** Unsynchronized map vs concurrency-safe map with atomic per-key operations.

**SQL vs NoSQL:** Choose based on queries, data relationships, transactions and scale—not hype.

**Deadlock:** Circular wait for held resources; prevent with consistent lock ordering.

## Interview practice checklist

- Can I explain the concept without relying on a buzzword?

- Can I identify the trade-off and a failure mode?

- Can I give a realistic production use case?

- Can I explain what changes at scale?

- Can I distinguish a guarantee from a typical implementation detail?

Keep learning: save this guide and revisit the follow-up questions before your next backend interview.
