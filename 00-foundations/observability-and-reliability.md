# Foundations: Observability and Reliability

## Wall Note / A4

**Monitoring** asks known questions with predefined signals.  
**Observability** is the ability to infer internal system state from externally available evidence, including questions not predicted before the failure.  
**Reliability** is the probability that a service delivers its intended user-visible behavior over time.

A system can be highly observable and still unreliable. A reliable system with weak observability becomes difficult to operate safely.

### Fast mental model

    User expectation -> SLI measurement -> SLO target -> operational policy
                                  |
                             telemetry
                                  |
                        detect -> diagnose -> act

Do not optimize telemetry volume. Optimize **time to trustworthy understanding**.

## Detailed Notes

### Monitoring vs observability

Monitoring typically begins with explicit conditions: CPU > 90%, queue depth > 10,000, error count > threshold. These are useful but limited by what engineers anticipated.

Observability requires telemetry with enough structure and context to answer broader questions:

- Which customers are affected?
- Did the failure begin after a deployment?
- Is the problem isolated to one dependency, region or tenant?
- Are retries hiding a downstream failure?
- Is latency caused by CPU saturation, lock contention, database waits or network loss?

Observability therefore depends on three properties more than on any product:

1. **Semantic instrumentation** — events and measurements describe domain-relevant behavior.
2. **Correlation** — evidence can be joined by request, trace, tenant, release or resource.
3. **Explorability** — engineers can slice data without having predicted every query.

### Reliability dimensions

Availability is only one dimension. Users experience reliability through correctness, latency, durability, freshness, consistency, recoverability, capacity, dependency behavior and operational safety.

For a checkout API, a 200 response containing the wrong order total is not reliable. For an analytics job, 99.99% request availability may matter less than data freshness.

### Resilience

Resilience is the system's ability to absorb, contain and recover from faults. Reliability is the resulting user-visible behavior. Resilience mechanisms include redundancy, timeouts, backpressure, circuit breakers, load shedding, failover and graceful degradation.

Every resilience mechanism has cost and can create new failure modes. Retries can amplify overload. Caches can serve stale data. Failover can move traffic into an already weak region. Senior engineering is about evaluating these second-order effects.

### Reliability loop

~~~mermaid
flowchart LR
    A[User expectations] --> B[SLIs]
    B --> C[SLOs]
    C --> D[Telemetry]
    D --> E[Detect]
    E --> F[Diagnose]
    F --> G[Mitigate]
    G --> H[Learn]
    H --> I[Improve design]
    I --> D
~~~

### Failure domains

Always identify boundaries: process, host/node, availability zone, region, network path, database shard, queue/partition, tenant, deployment version and dependency.

Redundancy only improves reliability if replicas do not share the same failure domain.

## Workflow

When assessing a system:

1. Define user-visible critical journeys.
2. Identify measurable success criteria.
3. Map dependencies and failure domains.
4. Decide which signals prove or disprove each hypothesis.
5. Define operational thresholds from user impact, not infrastructure aesthetics.
6. Revisit after incidents and architecture changes.

## Common failure modes

- collecting telemetry without ownership or retention policy;
- alerting on every component instead of user-impact symptoms;
- treating averages as representative of tail latency;
- confusing health endpoints with end-to-end correctness;
- assuming redundancy equals independence;
- measuring what is easy rather than what users care about.

## Exercises / Senior Questions

1. A service has 99.99% HTTP availability but users report stale data for hours. What reliability dimension is missing?
2. When does an infrastructure alert add value compared with an SLO alert?
3. Explain why adding more telemetry can make incident response slower.
4. Name three shared-fate risks in an apparently redundant architecture.
5. Design one user-centric SLI for a payment flow and one for a background ingestion pipeline.

## Related

- [SLIs, SLOs and error budgets](../03-reliability/sli-slo-sla-error-budgets.md)
- [Failure modes and resilience](../03-reliability/failure-modes-resilience.md)
- [System design](https://github.com/YosrBennagra/system-design)
- [DevOps/platform engineering](https://github.com/YosrBennagra/devops-platform-engineering)
