# Health, Readiness, Liveness, Capacity and Kubernetes

## Wall Note / A4

**Liveness**: should this process be restarted?  
**Readiness**: should this instance receive traffic?  
**Startup**: has initialization completed enough for normal health checks?

Do not make liveness depend on every external dependency. A database outage should not cause all application pods to restart in a loop.

Capacity requires watching both **utilization** and **saturation**.

## Detailed Notes

### Health semantics

A health endpoint is an operational contract.

Liveness should usually answer whether the process is irrecoverably stuck. Readiness can reflect whether the instance can safely serve its intended traffic.

Examples:

- thread deadlock → possibly liveness failure;
- database temporarily unavailable → readiness may fail, liveness usually remains healthy;
- cache unavailable when cache is optional → readiness may remain healthy but degradation metric should increase.

### Kubernetes behavior

Kubernetes probes influence routing and restarts:

- readiness removes a pod from service endpoints;
- liveness can trigger container restart;
- startup probe protects slow initialization from premature liveness failure.

A badly designed probe can turn a dependency incident into a restart storm.

### Capacity model

Capacity is not just "CPU percentage."

Monitor:

- throughput;
- request concurrency;
- CPU utilization and throttling;
- memory working set / OOM events;
- GC and JVM pauses where relevant;
- thread pools;
- connection pools;
- queue depth and age;
- disk latency/space;
- network throughput/loss;
- database waits;
- autoscaler signals.

### Saturation

Saturation tells you where additional work is waiting:

- executor queue;
- connection waiters;
- request backlog;
- throttled CPU time;
- consumer lag;
- disk queue;
- pending pods.

### Kubernetes diagnosis flow

~~~mermaid
flowchart TD
    A[User symptom] --> B[Service RED metrics]
    B --> C{Single pod or fleet?}
    C --> D[Pod restarts / readiness / version]
    C --> E[Node / cluster saturation]
    D --> F[CPU throttling / memory / GC / app pools]
    E --> G[Node pressure / scheduling / network / storage]
    F --> H[Dependency traces]
    G --> H
~~~

### Autoscaling cautions

Autoscaling is delayed feedback. It can fail when:

- metric is not causal;
- startup is slower than demand growth;
- downstream dependency cannot scale;
- traffic spikes faster than provisioning;
- scaling adds contention;
- limits/requests are badly chosen.

Scaling consumers against queue depth without considering downstream capacity can move the bottleneck rather than solve it.

### Capacity planning

Use observed demand and headroom:

- peak vs average;
- growth trend;
- seasonal events;
- failure scenarios such as one zone lost;
- deployment overlap;
- cache warm-up;
- recovery surge.

Plan for degraded-state capacity, not only happy-path steady state.

## Practical commands

The command mechanics belong in [engineering-toolbox](https://github.com/YosrBennagra/engineering-toolbox), but common investigation targets include:

    kubectl get pods
    kubectl describe pod <pod>
    kubectl logs <pod>
    kubectl top pod
    kubectl get events --sort-by=.lastTimestamp

Use commands to validate hypotheses, not as a random checklist.

## Failure modes

- liveness check calls a dependency and restarts healthy processes;
- readiness remains green while critical internal queue is jammed;
- autoscaling only on CPU for I/O-bound service;
- no alert on pending work age;
- resource limits causing throttling mistaken for application slowness;
- capacity plan assumes all zones available.

## Exercises / Senior Questions

1. Design liveness/readiness for an API that requires DB writes but optional cache reads.
2. Why can aggressive liveness checks cause cascading failure?
3. Which metrics would you use for an I/O-bound worker autoscaler?
4. How much headroom is needed if one of three zones can fail?
5. Explain CPU throttling when observed utilization looks moderate.

## Related

- [Golden signals and USE](../02-observability/golden-signals-red-use.md)
- [DevOps/platform engineering](https://github.com/YosrBennagra/devops-platform-engineering-)
- [Engineering toolbox](https://github.com/YosrBennagra/engineering-toolbox)
