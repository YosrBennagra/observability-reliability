# Observability & Reliability — Cheat Sheet

> Optimise for **time to trustworthy understanding**, not telemetry volume. Hub: [software-engineer-roadmap](https://github.com/YosrBennagra/software-engineer-roadmap)

```
User expectation → SLI (measure) → SLO (target) → error budget → operational policy
                        telemetry → detect → diagnose → act
```
| Term | One line |
|---|---|
| Monitoring | answers **known** questions with predefined signals |
| Observability | lets you answer **new** questions about internal state from outside evidence |
| Reliability | probability the service delivers its user-visible behaviour over time |

## Three signals (+ profiles)
| Signal | Is | Best for | Watch out |
|---|---|---|---|
| **Logs** | discrete structured events | details of one event, errors, audit | volume/cost, PII, unstructured text |
| **Metrics** | numeric time series | trends, alerting, capacity | **cardinality**: no user/order IDs in labels |
| **Traces** | one request across services (spans) | where time/failure goes across boundaries | sampling, context propagation |
- Log fields: `timestamp, level, service, env, version, trace_id, span_id, request_id, operation, outcome, duration_ms, error.type`. JSON. No secrets.
- Metric types: **Counter** (only up: requests, errors) · **Gauge** (up/down: queue depth, memory) · **Histogram** (distribution → p50/p95/p99, aggregatable) · Summary (client-side quantiles, not aggregatable).
- Trace = tree of spans (`trace_id`, `span_id`, `parent_span_id`). W3C `traceparent` header propagates context.
- **OpenTelemetry** = vendor-neutral APIs/SDKs/Collector for traces, metrics, logs. **Not** a storage backend.
- Stack examples: Prometheus (pull metrics) + Grafana · Loki/ELK (logs) · Tempo/Jaeger (traces) · Micrometer in Spring.

## Lenses
| Lens | Signals | Use for |
|---|---|---|
| Golden signals | Latency, Traffic, Errors, Saturation | any user-facing service |
| **RED** | Rate, Errors, Duration | request-driven services (start here) |
| **USE** | Utilisation, Saturation, Errors | resources: CPU, memory, disk, pools, queues |
- Averages hide pain → use **percentiles** (p95/p99). Count failed requests' latency too.

## SLI / SLO / SLA / error budget
- **SLI** = measured good/total (e.g. % of requests < 300 ms with 2xx/3xx/4xx).
- **SLO** = internal target (99.9% over 30 days). **SLA** = external contract with penalties (looser than the SLO).
- **Error budget** = 1 − SLO. 99.9% → 0.1% ≈ **43 min / 30 days**. 99.99% ≈ 4.3 min. 99% ≈ 7.2 h.
- Budget left → ship faster. Budget burnt → freeze risky changes, invest in reliability.
- Alert on **burn rate** (multi-window: fast burn pages, slow burn tickets), not on raw CPU.
- Set SLOs from user need + cost + capability, not "three nines sounds good". 100% is the wrong target.

## Alerting & dashboards
- Dashboards = awareness and investigation. **Alerts = action.**
- A page must say: symptom, severity, scope, owner, first diagnostic step (runbook link).
- Alert on **symptoms** (users hurt), use causes for diagnosis. If nobody must act now → not a page.

## Resilience patterns
| Pattern | Purpose | Misuse risk |
|---|---|---|
| Timeout | bound waiting (every remote call) | too long = thread exhaustion |
| Retry + exponential backoff + **jitter** | ride over transient faults | **retry storms**, non-idempotent duplicates |
| Circuit breaker | stop calling a failing dependency, fail fast | wrong thresholds flap |
| Bulkhead | isolate pools so one failure can't sink all | under-sized pools |
| Rate limit / load shedding | protect capacity | dropping important traffic |
| Backpressure | slow producers to the consumer's speed | unbounded queues hide it |
| Fallback / cache | degrade gracefully | stale or incorrect data |
| Idempotency | make retries safe | key store races |
| Redundancy / failover | survive node or zone loss | split brain, untested failover |
- Retry budget: retry at **one** layer only, with limits. Retries × layers = amplification.

## Kubernetes health probes
| Probe | Question | On failure |
|---|---|---|
| **Liveness** | is the process stuck? | **restart** container |
| **Readiness** | can it take traffic now? | removed from Service endpoints |
| **Startup** | finished booting? | delays the other probes |
- **Never** make liveness depend on the DB or downstream services → restart storms during a DB outage.

## Incidents
```
Detect → Triage (severity) → Mitigate (rollback / flag off / scale) → Recover → Postmortem
```
- Priorities: protect users and data → stabilise → shared timeline → diagnose with evidence → recover → learn.
- **Mitigate before root cause.** Roll back first, investigate later.
- Roles: incident commander, ops/investigator, communications.
- **Blameless postmortem:** timeline, impact, contributing factors (not "human error"), what went well, action items with owners.
- MTTD / MTTR (detect / restore). Measure and reduce them.

## Production debugging & chaos
- Evidence-first, minimally invasive, reversible, scoped, audited, safe for user data.
- **Chaos engineering** = controlled experiment: steady-state hypothesis → inject a fault → small blast radius → observe → abort switch.

## Instrument every boundary
Inbound HTTP · outbound HTTP/RPC · message produce/consume · DB queries · cache · scheduled jobs · thread-pool handoffs → capture **rate, errors, latency, saturation, trace propagation**.
- Spring: Actuator + Micrometer + Micrometer Tracing/OTel. Trace ID in the log MDC. Watch Hikari pool, executor queue and Kafka consumer lag metrics.

## Senior gotchas
- High-cardinality labels (`userId`) kill Prometheus.
- Logging the same error at every layer. Alert fatigue → real pages get ignored.
- Health check = `return "UP"` (lies) or checks every dependency (cascades).
- Async boundaries (`@Async`, Kafka) breaking trace context.
- Retries with no jitter → synchronised thundering herd.
- **Ask:** what failure are we buying protection from, and what new failure does the protection add?
