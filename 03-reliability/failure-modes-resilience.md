# Failure Modes, Resilience and Graceful Degradation

## Wall Note / A4

Assume every dependency can become:

- slow;
- unavailable;
- partially available;
- inconsistent;
- overloaded;
- partitioned;
- incorrect.

Resilience goal: **contain faults and preserve the most important user outcomes**.

Useful patterns:

- timeout;
- retry with backoff/jitter;
- circuit breaker;
- bulkhead;
- rate limit;
- backpressure;
- load shedding;
- fallback;
- cache;
- idempotency;
- redundancy/failover.

Every pattern can make things worse when misapplied.

## Detailed Notes

### Timeouts

Without timeouts, failures become resource leaks: threads, sockets and requests wait indefinitely.

Timeouts should be based on end-to-end budgets. A downstream timeout must leave time for the caller to recover or respond.

### Retries

Retries are only safe when the operation is idempotent or duplicate effects are controlled.

Use bounded retries with exponential backoff and jitter. Never retry all failures blindly.

Bad retry conditions:

- overload;
- validation failures;
- deterministic 4xx errors;
- long-running operations near deadline;
- non-idempotent writes without idempotency protection.

Retry storms are a classic cascading-failure amplifier.

### Circuit breakers

A circuit breaker stops repeated calls to a dependency that is already failing and periodically probes recovery.

It protects resources; it does not repair the dependency.

### Bulkheads

Separate resource pools for independent traffic classes so one failure does not consume everything. Examples: dedicated thread pools, connection pools, queues or worker groups.

### Backpressure and load shedding

When demand exceeds processing capacity:

- slow producers;
- reject low-priority work;
- cap queues;
- shed optional features;
- preserve critical paths.

An unbounded queue converts overload into latency, memory pressure and delayed failure.

### Graceful degradation

Examples:

- serve cached recommendations if recommendation service is down;
- hide nonessential personalization;
- accept order but delay analytics;
- switch from rich search ranking to basic search;
- return read-only mode instead of full outage.

Degradation must be visible in telemetry so hidden partial failures do not become normal.

### Failure-mode analysis

For each dependency ask:

| Question | Example |
|---|---|
| What if it is down? | inventory unavailable |
| What if it is slow? | timeout consumes request budget |
| What if it lies? | stale cache / inconsistent replica |
| What if it overloads us? | webhook flood |
| What if we overload it? | retry storm |
| What if only one region fails? | asymmetric traffic shift |

### Cascading failure

~~~mermaid
flowchart LR
    A[Dependency slows] --> B[Caller requests pile up]
    B --> C[Thread/connection pool saturation]
    C --> D[Latency rises]
    D --> E[Retries increase load]
    E --> A
~~~

Break the loop with deadlines, backpressure, isolation and controlled retries.

## Practical design checklist

Before adding a resilience pattern:

1. define the failure it addresses;
2. define user behavior during failure;
3. define resource boundary;
4. define telemetry;
5. test recovery as well as failure;
6. identify second-order effects.

## Exercises / Senior Questions

1. When can a retry reduce reliability?
2. Design graceful degradation for an e-commerce home page.
3. Explain why unbounded queues are dangerous.
4. Where would you put bulkheads in a multi-tenant service?
5. Which failures should open a circuit breaker and which should not?

## Related

- [Production debugging and chaos](../05-implementation/production-debugging-chaos.md)
- [Testing engineering](https://github.com/YosrBennagra/testing-engineering)
- [System design](https://github.com/YosrBennagra/system-design)
