# Exercises and Senior Interview / Review Questions

Use these to test reasoning, not vocabulary.

## Exercise 1 — latency incident

Observed:

- traffic unchanged;
- p50 unchanged;
- p99 rises from 700 ms to 8 s;
- CPU 45%;
- DB CPU 35%;
- DB connection pool pending threads rises sharply;
- traces show some transactions waiting on the same update statement.

Tasks:

1. Rank three hypotheses.
2. Name the next two pieces of evidence.
3. Propose immediate mitigation.
4. Propose a durable fix.
5. Define an alert that would detect recurrence earlier.

## Exercise 2 — retry storm

Service A calls B. B begins returning 503. A retries three times. Gateway also retries twice.

Tasks:

- calculate worst-case downstream amplification per original request;
- redesign retry ownership;
- define deadline/backoff behavior;
- choose metrics that prove the fix.

## Exercise 3 — SLO design

Design SLIs/SLOs for:

- checkout API;
- nightly payroll batch;
- search suggestions;
- event-ingestion pipeline.

For each define population, good event, window, target and excluded conditions.

## Exercise 4 — Kubernetes restart loop

Symptoms:

- database outage starts;
- application pods fail liveness;
- Kubernetes restarts all pods;
- database recovers, but app recovery remains slow due to synchronized startup load.

Explain the causal chain and redesign probes/recovery.

## Exercise 5 — observability cost

Trace storage cost doubled after adding user ID and raw URL attributes.

Tasks:

- identify cardinality/privacy concerns;
- redesign attributes;
- decide what detail belongs in logs vs traces vs metrics;
- define sampling/retention options.

## Exercise 6 — postmortem quality

Weak statement:

> The outage happened because an engineer deployed a bad config.

Rewrite the analysis to include trigger, systemic contributors, detection gap, response gap and durable actions.

## Senior questions

### Concepts

1. Monitoring vs observability?
2. Reliability vs resilience?
3. SLI vs SLO vs SLA?
4. Why is error budget useful?
5. Head vs tail trace sampling?
6. Histogram vs summary?
7. What is metric cardinality?
8. RED vs USE?
9. Readiness vs liveness?
10. What is saturation?

### Design

11. How would you instrument a new payment service?
12. What would be on its first dashboard?
13. Which alerts page and which become tickets?
14. How do you design correlation across HTTP and Kafka?
15. How do you observe DB connection-pool starvation?
16. How do you detect a stale-data incident?
17. How do you design graceful degradation?
18. Where would you apply bulkheads?
19. How do you prevent retry amplification?
20. How do you test failover?

### Operations

21. First five minutes of a major incident?
22. What evidence do you capture before restarting?
23. How do you form useful hypotheses?
24. Why compare healthy and unhealthy cohorts?
25. When do you rollback immediately?
26. What belongs in a postmortem?
27. How do you prevent alert fatigue?
28. How do you know a mitigation actually worked?
29. How do you debug high latency with low CPU?
30. How do you distinguish application vs DB bottleneck?

### Senior trade-offs

31. When is multi-region worth it?
32. When is a stricter SLO harmful?
33. How do you balance telemetry cost and diagnostic depth?
34. When should a cache outage be allowed to degrade the service?
35. How much capacity headroom is enough?
36. Which failure modes are introduced by autoscaling?
37. When is a fallback semantically unsafe?
38. How should reliability affect release policy?
39. What makes a reliability mechanism operationally unaffordable?
40. What evidence would make you simplify an architecture?

## Related learning system

- [software-engineer-roadmap](https://github.com/YosrBennagra/software-engineer-roadmap)
- [engineering-toolbox](https://github.com/YosrBennagra/engineering-toolbox)
- [devops-platform-engineering](https://github.com/YosrBennagra/devops-platform-engineering-)
- [testing-engineering](https://github.com/YosrBennagra/testing-engineering)
- [application-security](https://github.com/YosrBennagra/application-security)
- [java-mastery](https://github.com/YosrBennagra/java-mastery)
- [spring-mastery](https://github.com/YosrBennagra/spring-mastery)
