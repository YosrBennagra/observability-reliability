# SLIs, SLOs, SLAs and Error Budgets

## Wall Note / A4

**SLI** = measured behavior.  
**SLO** = internal target for that behavior.  
**SLA** = external/business commitment, often with consequences.  
**Error budget** = tolerated unreliability implied by the SLO.

For a 99.9% success SLO:

    error budget = 1 - 0.999 = 0.001 = 0.1%

Do not set an SLO because "three nines sounds good." Set it from user need, architecture, cost and operating capability.

## Detailed Notes

### Good SLIs

A good SLI is:

- user-centered;
- measurable with a clear numerator and denominator;
- hard to game;
- stable enough for decisions;
- scoped to a defined population/window.

Examples:

Availability SLI:

    good requests / eligible requests

Latency SLI:

    requests completed under 300 ms / eligible requests

Freshness SLI:

    records delivered within 5 minutes / records expected

Correctness SLI:

    correctly processed operations / operations validated

### Eligibility matters

Not every event should count. Define:

- which endpoints/operations;
- whether invalid client requests count;
- whether maintenance windows count;
- how retries are handled;
- which regions/tenants;
- what constitutes success.

A denominator that changes silently makes SLO history meaningless.

### SLO windows

Common windows are 7, 28 or 30 days. The important property is that the window matches business/operational decisions.

Rolling windows respond continuously. Calendar windows align with reporting but create boundary effects.

### Error budgets

If the SLO is 99.9% over 30 days, the service can spend at most 0.1% of eligible events as bad events while remaining within objective.

Error budgets support explicit trade-offs:

- healthy budget: normal delivery pace;
- fast burn: freeze risky changes and mitigate;
- exhausted budget: prioritize reliability work;
- consistently unused budget: objective may be stricter than users need or the system may have excessive reliability cost.

### Burn rate

Burn rate compares current error consumption with the allowed long-term rate.

    burn rate = observed bad-event ratio / allowed bad-event ratio

For a 99.9% SLO, allowed bad ratio is 0.1%.

If observed errors are 1%, burn rate is 10x.

A 10x burn sustained for long enough will exhaust the budget quickly.

### Relationship

~~~mermaid
flowchart LR
    A[User expectation] --> B[SLI]
    B --> C[SLO target]
    C --> D[Allowed bad fraction]
    D --> E[Error budget]
    E --> F[Burn-rate policy]
    F --> G[Release / reliability decisions]
~~~

### SLA caution

SLAs should usually be looser than internal SLOs so engineering has room to react before contractual impact. Do not equate marketing uptime claims with a technical reliability strategy.

### Composite journeys

A user journey crossing several services should be measured at the journey boundary when possible. Multiplying component availabilities can estimate theoretical reliability, but the end-to-end SLI is more trustworthy.

## Practical example

Checkout objective:

- population: authenticated checkout submissions;
- success: order accepted and durable confirmation returned;
- SLO: 99.95% success over rolling 28 days;
- latency SLO: 99% under 1.5 s;
- excluded: explicit client validation errors;
- included: downstream timeouts, server errors, malformed internal responses.

This is stronger than "API uptime 99.95%" because it describes a user outcome.

## Failure modes

- measuring proxy health instead of user success;
- excluding failures to make the SLO look good;
- using one SLO for very different traffic classes;
- setting unrealistic objectives that normalize constant violation;
- setting overly loose objectives that provide no pressure for quality;
- treating error budget as permission to intentionally cause failures.

## Exercises / Senior Questions

1. Define a freshness SLI for an ETL pipeline.
2. Why should an SLA usually not equal the internal SLO?
3. What denominator would you use for an authenticated API with many 4xx responses?
4. Explain a 20x burn rate to a product manager.
5. How would you handle premium and free-tier users with different expectations?

## Related

- [Dashboards and alerting](../02-observability/dashboards-and-alerting.md)
- [Senior reliability trade-offs](../06-senior/reliability-tradeoffs.md)
- [System design](https://github.com/YosrBennagra/system-design)
