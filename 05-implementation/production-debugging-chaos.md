# Production Debugging and Chaos / Failure Testing

## Wall Note / A4

Production debugging should be:

- evidence-first;
- minimally invasive;
- reversible;
- scoped;
- audited;
- safe for user data.

Chaos engineering is not random breakage. It is a controlled experiment against a reliability hypothesis.

## Production debugging workflow

### 1. Preserve evidence

Before changing the system, preserve:

- incident timeline;
- relevant logs/traces;
- metric snapshots;
- deployment/config versions;
- process/pod state;
- thread/heap/profile evidence when justified.

A restart may mitigate the incident but destroys useful in-memory evidence.

### 2. Prefer observation over mutation

Start with read-only inspection:

- dashboards;
- traces;
- logs;
- health endpoints;
- thread dumps;
- process/resource metrics;
- DB activity views;
- Kubernetes events.

Only mutate production when the expected risk/reward is clear.

### 3. Reduce scope

Target a single pod, canary, tenant or traffic slice when possible. Avoid fleet-wide debug logging or profiling without understanding overhead.

### 4. Form and falsify hypotheses

A good hypothesis is specific and testable:

> Requests are slow because all HikariCP connections are occupied by transactions blocked on one database lock.

Evidence to seek:

- connection pending count;
- DB lock waits;
- trace spans stuck in SQL;
- thread stacks waiting in JDBC;
- healthy comparison sample.

### Thread dumps

Useful for:

- deadlocks;
- lock contention;
- thread-pool exhaustion;
- blocking I/O;
- repeated stack patterns.

One dump is a snapshot. Several spaced samples show persistence.

### Heap dumps

Useful for suspected memory leaks/dominators, but they can be large, sensitive and expensive. Treat them as production data and secure them accordingly.

### Continuous profiling

Lower-overhead continuous profilers can reveal CPU, allocation, lock and wall-clock patterns over time. Sampling changes the question from "what is one thread doing now?" to "where does the fleet spend time?"

## Debug logging

Dynamic log-level changes can help, but:

- scope narrowly;
- set a rollback/expiry;
- avoid secrets;
- watch volume;
- avoid enabling verbose DB/HTTP bodies globally.

## Chaos / failure testing

### Hypothesis format

> Given healthy steady state, when dependency X is unavailable for 60 seconds, checkout success for payment-independent paths remains above 99%, and the system recovers without manual restart.

Good experiments specify:

- steady-state metric;
- injected failure;
- blast radius;
- duration;
- abort conditions;
- expected behavior;
- recovery criteria.

### Failure types

- dependency latency;
- connection refusal;
- packet loss;
- pod/node loss;
- CPU/memory pressure;
- disk latency;
- queue backlog;
- expired credentials/certificates in controlled environments;
- zone isolation;
- DNS failure.

Prefer pre-production first, then production only when safeguards and value justify it.

### Chaos experiment flow

~~~mermaid
flowchart LR
    A[Define reliability hypothesis] --> B[Choose smallest safe blast radius]
    B --> C[Define abort conditions]
    C --> D[Inject controlled failure]
    D --> E[Observe SLIs and recovery]
    E --> F[Stop / restore]
    F --> G[Analyze gaps]
    G --> H[Improve system and runbook]
~~~

### Abort conditions

Examples:

- SLO burn exceeds threshold;
- unexpected data-loss risk;
- impact escapes test tenant/zone;
- recovery path fails;
- telemetry becomes unreliable.

## Anti-patterns

- debugging directly by changing multiple variables;
- restarting everything before collecting evidence;
- production heap dump copied to insecure laptop;
- enabling TRACE logs fleet-wide;
- chaos tests with no hypothesis;
- failure injection without recovery verification;
- calling ordinary load tests "chaos engineering."

## Exercises / Senior Questions

1. What evidence should you capture before restarting a stuck JVM?
2. When is a heap dump inappropriate?
3. Write a chaos hypothesis for losing one Kubernetes node.
4. Why are abort conditions part of experiment design?
5. How would you distinguish lock contention from CPU saturation?

## Related

- [Incident response](../04-operations/incidents-troubleshooting-postmortems.md)
- [Testing engineering](https://github.com/YosrBennagra/testing-engineering)
- [Engineering toolbox](https://github.com/YosrBennagra/engineering-toolbox)
