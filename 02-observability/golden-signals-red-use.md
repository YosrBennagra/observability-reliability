# Golden Signals, RED and USE

## Wall Note / A4

Three useful lenses:

### Golden signals
- **Latency**
- **Traffic**
- **Errors**
- **Saturation**

### RED — request-oriented services
- **Rate**
- **Errors**
- **Duration**

### USE — resources
- **Utilization**
- **Saturation**
- **Errors**

Use RED/golden signals to start from service behavior. Use USE to investigate resource causes.

## Detailed Notes

These are not competing frameworks. They answer different layers of the same investigation.

### RED

Best fit for request/operation flows such as HTTP, RPC, jobs and message processing.

Questions:

- Rate: how much work is arriving?
- Errors: what fraction fails?
- Duration: how long does work take?

For queues, rate can include enqueue/dequeue throughput and duration can include queue age plus processing time.

### USE

Best fit for finite resources: CPU, memory, disk, network, thread pools, DB connection pools and executor queues.

**Utilization** is the fraction of capacity busy.  
**Saturation** is work waiting because immediate capacity is exhausted.  
**Errors** are explicit failures attributable to the resource.

A resource can show moderate utilization but high saturation due to lock contention, throttling or serialized work.

### Combined diagnosis

~~~mermaid
flowchart TD
    A[High user latency] --> B[RED: duration elevated?]
    B --> C[Errors also elevated?]
    B --> D[Dependency spans]
    D --> E[USE on suspected resource]
    E --> F[CPU / pool / DB / network saturation]
    F --> G[Validate cause]
~~~

Example:

- p95 latency rises;
- request rate is unchanged;
- error rate rises slightly;
- traces show DB waits;
- DB connection pool active=100%, pending=80;
- DB CPU is normal.

The bottleneck is likely pool/concurrency related rather than raw database CPU.

### Saturation examples

- CPU run queue or throttling;
- thread pool queue depth;
- connection waiters;
- disk I/O queue;
- network send queue;
- memory reclaim pressure;
- Kafka consumer lag.

Choose the saturation metric that represents **work waiting for scarce capacity**.

### Golden-signal interpretation

Traffic establishes demand. Latency and errors capture visible quality. Saturation tells you how close you are to a capacity ceiling.

Do not use a framework mechanically. A batch pipeline may care more about freshness, throughput and backlog age than per-request latency.

## Failure modes

- treating CPU utilization as the only capacity signal;
- using RED for resources where USE is clearer;
- alerting on every framework metric;
- ignoring queue age while watching queue size;
- assuming lower utilization is always better;
- comparing rates without accounting for traffic mix.

## Exercises / Senior Questions

1. Apply RED to a message consumer.
2. Apply USE to a database connection pool.
3. Give an example where utilization is low but saturation is high.
4. Which golden signal often gives early evidence of a capacity ceiling?
5. Build a symptom-to-resource investigation path for rising API latency.

## Related

- [Metrics](../01-signals/metrics.md)
- [Capacity and Kubernetes](../04-operations/health-capacity-kubernetes.md)
- [Dashboards and alerts](dashboards-and-alerting.md)
