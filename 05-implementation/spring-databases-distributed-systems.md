# Observability for Spring, Databases and Distributed Systems

## Wall Note / A4

For distributed applications, instrument the boundaries:

- inbound requests;
- outbound HTTP/RPC;
- message produce/consume;
- database queries;
- cache calls;
- scheduled/background jobs;
- executor/thread-pool handoffs.

For every boundary capture: **rate, errors, latency, saturation/context, trace propagation**.

## Spring applications

### Minimum operational picture

A production Spring service should expose enough evidence to understand:

- request throughput, status and latency;
- active/queued server threads where applicable;
- JVM CPU, heap, non-heap and GC;
- executor pool utilization/queueing;
- DB connection pool active/idle/pending;
- outbound dependency latency/errors;
- cache hit/miss behavior;
- application-specific SLIs;
- release/version metadata;
- health/readiness state.

Spring Boot Actuator and Micrometer can provide much of the mechanics, but the important engineering task is deciding which metrics and tags are operationally meaningful.

### Avoid tag explosion

Do not tag metrics with raw URI, user ID, order ID, exception message or arbitrary query parameters.

Prefer route templates and bounded dimensions.

### Thread pools and async work

A common hidden bottleneck:

- request rate normal;
- CPU moderate;
- latency high;
- executor active threads at max;
- queue depth rising.

Observe executor active count, max size, queue depth/age, rejection count and task duration.

### JVM interpretation

High heap usage alone is not an incident. Ask:

- does memory recover after GC?
- are pause times increasing?
- is allocation rate unusual?
- is container memory limit near?
- are OOM kills/restarts occurring?
- is CPU consumed by GC?

Deep JVM mechanics belong in [java-mastery](https://github.com/YosrBennagra/java-mastery).

## Databases

### Observe from application and database sides

Application evidence:

- query/transaction latency;
- connection acquisition time;
- pool saturation;
- timeout/error classes;
- retry count.

Database evidence:

- active sessions;
- lock waits;
- slow queries;
- buffer/cache behavior;
- disk latency;
- transaction conflicts;
- replication lag;
- connection limits;
- query plan changes.

### Connection-pool trap

If requests wait for connections, application latency can rise while database CPU is low.

Differentiate:

1. time waiting for a pool connection;
2. time executing on DB;
3. time committing/locking.

### Query cardinality

Never put full SQL text with user values into high-cardinality metric labels. Use normalized query identity where supported and retain detailed samples/logs separately.

## Distributed systems

### Partial failure is normal

A request may succeed in service A, timeout in B, be processed in C and duplicate in D. Trace context and idempotency identifiers help reconstruct reality.

### Messaging

Track:

- publish rate/errors;
- consume rate/errors;
- queue/partition backlog;
- **oldest-message age**;
- processing duration;
- retry/dead-letter rate;
- rebalance activity;
- consumer saturation.

Backlog depth alone can be misleading when message size or processing cost varies.

### Caches

Watch:

- hit ratio;
- latency;
- evictions;
- memory pressure;
- stale-data risk;
- fallback load on origin.

A cache outage can overload the database even if the cache itself is "optional."

### Distributed trace interpretation

~~~mermaid
sequenceDiagram
    participant C as Client
    participant S as Spring API
    participant Q as Queue
    participant W as Worker
    participant D as DB
    C->>S: request trace=T
    S->>D: transaction span
    S->>Q: publish span + context
    Q->>W: consume with linked context
    W->>D: write span
~~~

For asynchronous messaging, parent/child semantics may be replaced by span links depending on processing model. Preserve causality without pretending asynchronous work has synchronous timing.

## Practical investigation examples

### Spring endpoint slow, CPU low

Check in order:

1. RED metrics by route/version;
2. trace latency breakdown;
3. connection/executor pools;
4. DB waits and dependency latency;
5. thread dump if work appears blocked.

### Database suddenly slow after release

Compare:

- query shape/plan;
- transaction boundaries;
- N+1 behavior;
- connection pool settings;
- lock duration;
- result-set size;
- index usage.

Do not conclude "database problem" merely because DB spans are slow; application-generated contention may be the cause.

## Exercises / Senior Questions

1. Design metric tags for Spring HTTP requests without cardinality explosion.
2. How do you distinguish DB execution time from connection-pool wait time?
3. Why can a cache failure become a database incident?
4. Which signal best captures user impact in a queue-backed pipeline?
5. Explain trace propagation through async messaging.

## Related

- [Spring mastery](https://github.com/YosrBennagra/spring-mastery)
- [Java mastery](https://github.com/YosrBennagra/java-mastery)
- [DevOps/platform engineering](https://github.com/YosrBennagra/devops-platform-engineering-)
