# Incident Detection, Response, Troubleshooting, RCA and Postmortems

## Wall Note / A4

During an incident, optimize for:

1. **protect users/data;**
2. **stabilize the system;**
3. **establish a shared factual timeline;**
4. **diagnose with evidence;**
5. **recover safely;**
6. **learn afterward.**

Mitigation and root-cause analysis are different activities. Do not delay a safe rollback while searching for perfect causal certainty.

## Incident roles

For significant incidents, explicitly assign:

- **Incident commander** — coordinates priorities and decisions.
- **Operations/technical lead** — drives diagnosis and mitigation.
- **Communications lead** — updates stakeholders/status channels.
- **Scribe** — records timeline, hypotheses, actions and results.

Small incidents may combine roles, but the responsibilities still exist.

## Response flow

~~~mermaid
flowchart TD
    A[Detection] --> B[Assess impact and severity]
    B --> C[Declare incident / assign roles]
    C --> D[Contain or mitigate]
    D --> E[Collect evidence]
    E --> F[Form ranked hypotheses]
    F --> G[Test cheapest discriminating hypothesis]
    G --> H{Recovered?}
    H -- No --> D
    H -- Yes --> I[Verify stability]
    I --> J[Close incident]
    J --> K[Postmortem and follow-up]
~~~

## Troubleshooting method

Avoid random command execution. Use a narrowing loop.

### 1. Define the symptom precisely

Bad: "the system is slow."

Better:

- p95 checkout latency rose from 400 ms to 3.2 s;
- started at 14:07 UTC;
- EU region only;
- write endpoints affected, reads mostly healthy;
- coincides with release 2026.10.3-4.

### 2. Bound the scope

Slice by:

- service;
- endpoint/operation;
- tenant/customer tier;
- region/zone;
- deployment version;
- pod/node;
- dependency;
- request type;
- time.

### 3. Build a timeline

Record:

- first known impact;
- alerts;
- deployments/config changes;
- autoscaling/failover;
- dependency incidents;
- mitigation actions;
- recovery.

Correlation in time is evidence, not proof of causation.

### 4. Rank hypotheses

Prefer hypotheses that explain **all** important observations.

Example:

| Hypothesis | Explains latency? | Explains only writes? | Explains EU only? |
|---|---:|---:|---:|
| JVM GC | yes | weak | weak |
| primary DB saturation in EU | yes | yes | yes |
| frontend regression | maybe | weak | weak |

Test the cheapest hypothesis that most strongly separates alternatives.

### 5. Compare healthy vs unhealthy

Powerful comparisons:

- previous vs current version;
- affected vs healthy region;
- one pod vs fleet;
- slow trace vs normal trace;
- before vs after config change;
- warm vs cold cache.

### 6. Verify mitigation

A rollback is not complete because deployment succeeded. Verify user-centric signals return to expected ranges and no secondary failure appears.

## Root-cause analysis

A root cause is rarely a single line of code. Distinguish:

- **trigger** — event that started the incident;
- **contributing factors** — conditions increasing severity/probability;
- **detection gap** — why it was not identified earlier;
- **response gap** — what slowed mitigation;
- **systemic cause** — design/process condition allowing recurrence.

Example:

- trigger: schema migration added blocking index build;
- contributing factor: traffic peak and small connection pool;
- detection gap: no DB lock-wait dashboard;
- response gap: rollback procedure unclear;
- systemic cause: migration review did not model production lock behavior.

## Five whys caution

"Five whys" is a prompt, not a proof technique. Complex incidents have branching causal graphs. Do not force one linear cause when multiple necessary factors existed.

## Postmortem structure

1. summary and user impact;
2. severity and duration;
3. timeline with evidence;
4. detection;
5. technical analysis;
6. contributing factors;
7. what worked;
8. what failed;
9. corrective actions with owners;
10. lessons and follow-up verification.

### Action quality

Weak:

- "be more careful";
- "add more monitoring";
- "developer should test better."

Strong:

- add migration lock-timeout policy;
- pre-production test with production-scale cardinality;
- add SLO burn alert for write latency;
- automate safe rollback validation;
- reduce blast radius with progressive rollout.

Actions should remove classes of failure, reduce blast radius or improve detection/response.

## Communication

Good incident updates state:

- current impact;
- what is known;
- what is being done;
- next update time/condition.

Avoid premature causal claims.

## Exercises / Senior Questions

1. API errors start immediately after a deployment. When do you rollback before proving causality?
2. Build three discriminating hypotheses for high latency with normal CPU.
3. What makes an incident timeline reliable?
4. Rewrite "developer forgot validation" into systemic postmortem analysis.
5. Design two corrective actions that reduce blast radius rather than only detection time.

## Related

- [Dashboards and alerting](../02-observability/dashboards-and-alerting.md)
- [Production debugging](../05-implementation/production-debugging-chaos.md)
- [Engineering toolbox troubleshooting](https://github.com/YosrBennagra/engineering-toolbox)
