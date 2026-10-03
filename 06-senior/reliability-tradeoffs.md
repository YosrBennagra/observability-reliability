# Senior-Level Reliability Trade-offs

## Wall Note / A4

Senior reliability engineering is choosing **where to spend complexity and cost**.

There is no maximum-reliability architecture. There is only reliability appropriate to:

- user impact;
- business value;
- failure cost;
- compliance;
- recovery requirements;
- team maturity;
- budget.

Ask: **what failure are we buying protection from, and what new failure does the protection introduce?**

## Trade-off 1: Reliability vs delivery speed

Stricter change controls can reduce change risk but also slow fixes and learning.

Prefer mechanisms that increase both safety and throughput:

- automated tests;
- progressive delivery;
- fast rollback;
- feature flags;
- observability;
- small changes;
- ownership.

Use error budgets to turn the debate into policy rather than opinion.

## Trade-off 2: Redundancy vs complexity

Replicas, regions and failover improve availability only when failure modes are sufficiently independent.

Multi-region adds:

- data consistency challenges;
- replication lag;
- routing complexity;
- split-brain risk;
- higher cost;
- harder testing;
- operational skill requirements.

Do not deploy multi-region purely because it sounds senior.

## Trade-off 3: Strong consistency vs availability/latency

Under network partitions, distributed systems must make trade-offs. The correct choice depends on domain invariants.

Payment ledger: stale/inconsistent writes may be unacceptable.  
Product recommendations: temporary stale reads may be fine.

State the invariant first, then choose architecture.

## Trade-off 4: Retries vs overload

Retries improve transient-failure success but multiply traffic precisely when dependencies may be weakest.

Senior design considers:

- total retry budget across layers;
- deadlines;
- idempotency;
- backoff/jitter;
- retryable failure classes;
- concurrency amplification.

Avoid retries at every layer.

## Trade-off 5: Telemetry depth vs cost/privacy

More telemetry can improve diagnosis but increases:

- storage;
- indexing cost;
- network overhead;
- developer noise;
- privacy/security exposure.

Use sampling, aggregation, retention tiers and field governance.

## Trade-off 6: Alert sensitivity vs fatigue

Sensitive alerts detect earlier but increase false positives. Noisy pages train engineers to ignore pages.

Tune against:

- precision: fraction of alerts requiring action;
- recall: fraction of meaningful incidents detected;
- detection delay;
- human cost.

An alerting system is a socio-technical system.

## Trade-off 7: Capacity headroom vs cost

Headroom absorbs bursts and failures. Excessive idle capacity wastes money.

Model:

- normal peak;
- growth;
- autoscaling delay;
- one-zone/one-node loss;
- deployment overlap;
- recovery surge.

## Trade-off 8: Graceful degradation vs correctness

Fallbacks can preserve availability but may violate semantics.

Examples:

- cached price may be unacceptable;
- cached recommendation is likely acceptable;
- read-only mode may be safer than accepting writes that cannot be durably stored.

Classify operations by criticality and correctness requirements.

## Trade-off 9: SLO strictness vs meaningfulness

An impossibly strict SLO becomes permanently red and loses decision value. An easy SLO creates no reliability pressure.

Use historical performance as evidence but do not merely codify the current state. Start from user need.

## Senior review framework

Before approving a reliability design ask:

1. What user outcome is protected?
2. What SLI proves protection works?
3. What failure modes are in scope?
4. What is the blast radius?
5. What shared dependencies remain?
6. How is failure detected?
7. What is the safe mitigation?
8. What is the recovery path?
9. How is the mechanism tested?
10. What are the cost and operational burden?

## Scenario

A team proposes active-active multi-region for an internal reporting app with a four-hour recovery tolerance.

A senior response is not "multi-region is best." It asks whether:

- restore automation;
- tested backups;
- warm standby;
- infrastructure-as-code;
- faster DNS/routing recovery

could meet the objective at far lower complexity.

## Exercises / Senior Questions

1. When is 99.99% materially better than 99.9%?
2. When should a service deliberately fail fast rather than retry?
3. Give one case where graceful degradation is unsafe.
4. How do you justify observability cost to product leadership?
5. Which questions reveal hidden shared-fate in a redundant design?
6. What would make you reject active-active multi-region?
7. How do error budgets change release governance?

## Related

- [SLIs/SLOs/error budgets](../03-reliability/sli-slo-sla-error-budgets.md)
- [System design](https://github.com/YosrBennagra/system-design)
- [Software architecture](https://github.com/YosrBennagra/software-architecture)
