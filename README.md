# Observability & Reliability Engineering — 0 → Expert

A practical knowledge repository for understanding, operating and improving production systems. It is part of the wider software-engineering knowledge system indexed by [software-engineer-roadmap](https://github.com/YosrBennagra/software-engineer-roadmap).

The goal is not to memorize monitoring products. The goal is to build the mental models required to answer:

- **What is happening?**
- **Why is it happening?**
- **Who is affected?**
- **How bad is it?**
- **What should we do next?**
- **How do we prevent recurrence without over-engineering?**

## Learning order

1. [Foundations: observability vs monitoring](00-foundations/observability-and-reliability.md)
2. [Logs and correlation](01-signals/logs-and-correlation.md)
3. [Metrics](01-signals/metrics.md)
4. [Tracing and OpenTelemetry](01-signals/traces-and-opentelemetry.md)
5. [Dashboards and alerting](02-observability/dashboards-and-alerting.md)
6. [Golden signals, RED and USE](02-observability/golden-signals-red-use.md)
7. [SLIs, SLOs, SLAs and error budgets](03-reliability/sli-slo-sla-error-budgets.md)
8. [Failure modes, resilience and graceful degradation](03-reliability/failure-modes-resilience.md)
9. [Health, capacity and Kubernetes](04-operations/health-capacity-kubernetes.md)
10. [Incidents, troubleshooting, RCA and postmortems](04-operations/incidents-troubleshooting-postmortems.md)
11. [Spring, databases and distributed systems](05-implementation/spring-databases-distributed-systems.md)
12. [Production debugging and chaos/failure testing](05-implementation/production-debugging-chaos.md)
13. [Senior trade-offs](06-senior/reliability-tradeoffs.md)
14. [Exercises and senior questions](exercises/senior-questions.md)

## Topic map

| Area | Core questions |
|---|---|
| Telemetry | What signals exist and how are they correlated? |
| Observability | Can the system answer questions we did not predict in advance? |
| Reliability | Does the service consistently satisfy user expectations? |
| Detection | Are failures detected by symptoms before users report them? |
| Diagnosis | Can engineers narrow scope, reproduce evidence and identify causes? |
| Response | Can incidents be stabilized safely and communicated clearly? |
| Learning | Do postmortems produce durable system improvements? |

## Core telemetry model

~~~mermaid
flowchart LR
    A[Applications / Infrastructure] --> B[Instrumentation]
    B --> C[Logs]
    B --> D[Metrics]
    B --> E[Traces]
    C --> F[Collector / Pipeline]
    D --> F
    E --> F
    F --> G[Storage / Backends]
    G --> H[Dashboards]
    G --> I[Alerts]
    G --> J[Ad-hoc investigation]
~~~

Telemetry is only useful when it supports a decision. High-volume data with poor semantics is expensive noise.

## Progress checklist

- [ ] I can distinguish monitoring from observability.
- [ ] I can design structured logs without leaking secrets.
- [ ] I understand metric types, labels and cardinality.
- [ ] I can explain trace/span propagation and sampling.
- [ ] I can build symptom-first dashboards and alerts.
- [ ] I can apply RED, USE and golden signals appropriately.
- [ ] I can define meaningful SLIs and SLOs.
- [ ] I understand error-budget policy and burn-rate thinking.
- [ ] I can reason about failure modes and graceful degradation.
- [ ] I can separate readiness, liveness and startup concerns.
- [ ] I can run a structured incident investigation.
- [ ] I can write a blameless, evidence-based postmortem.
- [ ] I can debug Spring, databases and Kubernetes from symptoms inward.
- [ ] I can explain reliability trade-offs to product and engineering stakeholders.

## Repository boundaries

Deep platform setup belongs in [devops-platform-engineering](https://github.com/YosrBennagra/devops-platform-engineering-). Testing mechanics belong in [testing-engineering](https://github.com/YosrBennagra/testing-engineering). Security telemetry belongs in [application-security](https://github.com/YosrBennagra/application-security). Java/JVM internals belong in [java-mastery](https://github.com/YosrBennagra/java-mastery). Spring implementation details belong in [spring-mastery](https://github.com/YosrBennagra/spring-mastery). Practical diagnostic commands are cross-linked with [engineering-toolbox](https://github.com/YosrBennagra/engineering-toolbox).

This repository owns the **operational mental model** tying those areas together.
