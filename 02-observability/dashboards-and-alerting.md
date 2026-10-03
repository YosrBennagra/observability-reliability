# Dashboards and Alerting

## Wall Note / A4

Dashboards are for **situational awareness and investigation**.  
Alerts are for **action**.

An alert should make clear:

1. what symptom is occurring;
2. how severe it is;
3. what scope is affected;
4. who owns the response;
5. what first diagnostic steps are useful.

If nobody should act, it is probably not a page-worthy alert.

## Detailed Notes

### Dashboard hierarchy

A useful dashboard typically follows symptom → cause:

1. user experience / SLO state;
2. traffic, errors, latency;
3. dependencies;
4. resource saturation;
5. deployments and other change markers.

Avoid giant "wall of graphs" dashboards that require tribal knowledge.

### Alert classes

A pragmatic model:

- **Page**: immediate human action is required to protect users or data.
- **Ticket/task**: meaningful issue, but not urgent.
- **Dashboard only**: diagnostic context with no action threshold.

Page on symptoms when possible. Component alerts are justified when they reliably predict imminent user harm and have a clear response.

### Alert quality

High-quality alerts have a stable signal, meaningful threshold or burn rate, appropriate duration, grouping/deduplication, clear ownership, severity tied to impact, a runbook and evidence rather than speculation.

### Multi-window thinking

Short windows detect fast incidents but can be noisy. Long windows confirm persistent impact but detect slowly.

For SLO burn alerts, pair a fast window with a longer confirmation window. Example conceptually:

    fast burn:  5m + 1h
    slow burn:  30m + 6h

Exact windows depend on SLO period, traffic and operating model.

### Dashboard anti-patterns

- panels without units;
- red/green thresholds with no operational meaning;
- averages only;
- mixed environments;
- no deployment markers;
- ratios without denominators;
- too many panels above the fold;
- duplicating every raw metric.

### Alert flow

~~~mermaid
flowchart LR
    A[Telemetry] --> B[Rule evaluation]
    B --> C{Actionable?}
    C -- No --> D[Dashboard / analysis]
    C -- Yes --> E[Route by ownership]
    E --> F[Deduplicate / group]
    F --> G[Page or ticket]
    G --> H[Runbook + investigation]
~~~

### Runbooks

A runbook should provide service purpose and owner, impact definition, critical dashboards/queries, known dependencies, safe mitigations, rollback/failover steps, escalation path and links to recent changes.

It should guide reasoning, not prescribe blind button-clicking.

## Practical examples

A weak alert:

    JVM heap > 80%

A stronger approach:

- page when request success or latency SLO burns rapidly;
- use JVM heap/GC as investigation evidence;
- create a lower-urgency capacity alert if sustained memory pressure predicts a near-term incident.

For a queue-backed worker:

- symptom: oldest-message age exceeds tolerated freshness;
- diagnostic panels: enqueue rate, dequeue rate, backlog, worker saturation, dependency latency;
- mitigation options: scale consumers, pause noncritical producers, shed optional work.

## Failure modes

- paging on every local anomaly;
- thresholds copied from another service;
- alert storms during dependency incidents;
- alerts that fire after users already report the issue;
- no owner or runbook;
- dashboard changes not reviewed with service changes.

## Exercises / Senior Questions

1. Rewrite "CPU > 80%" into a more actionable alerting strategy.
2. Which panels belong on the first row of a checkout dashboard?
3. When is a component-level page justified?
4. Explain how poor deduplication can worsen an incident.
5. Design one runbook entry for elevated latency after a deployment.

## Related

- [Golden signals / RED / USE](golden-signals-red-use.md)
- [SLIs and SLOs](../03-reliability/sli-slo-sla-error-budgets.md)
- [Incident response](../04-operations/incidents-troubleshooting-postmortems.md)
