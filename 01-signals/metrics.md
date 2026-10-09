# Metrics

## Wall Note / A4

Metrics are **numeric time series** optimized for aggregation and trends.

Use them to answer: how much, how often, how fast, how saturated and whether behavior is changing.

| Type | Use |
|---|---|
| Counter | monotonically increasing events: requests, failures |
| Gauge | value that rises/falls: queue depth, memory |
| Histogram | distribution: latency, payload size |
| Summary | client-side quantiles/aggregates; harder to aggregate globally |

**Cardinality is a design constraint.** Never put unbounded IDs in metric labels.

## Detailed Notes

A time series is identified by metric name plus label set:

    http_server_requests_total{
      service="orders",
      method="POST",
      status_class="5xx"
    }

If a label can take millions of values—user ID, order ID, raw URL, trace ID—it can create millions of series and overwhelm storage/query systems.

### Prometheus mental model

Prometheus-style systems typically scrape or receive exported metrics, store time series and evaluate queries/rules. Grafana commonly visualizes those time series. The important concepts are independent of products:

- stable metric names;
- bounded dimensions;
- meaningful units;
- aggregation-safe design;
- recording rules for expensive repeated queries;
- retention appropriate to decision-making.

### Latency distributions

Averages hide tails. If nine requests take 10 ms and one takes 2,000 ms, the 209 ms average describes almost nobody.

Percentiles answer different questions:

- p50: typical;
- p95: common tail;
- p99/p99.9: extreme tail.

A percentile is not a percentage of requests "meeting" a target unless the threshold is chosen accordingly.

### Rates vs counts

A raw counter since process start is rarely useful. Derive rate over a window.

    request_rate = delta(request_counter) / delta(time)
    error_ratio = error_rate / request_rate

Use windows long enough to avoid noise but short enough to detect real change.

### Histograms

Histograms bucket observations. Bucket boundaries should reflect meaningful thresholds. If an SLO is 300 ms, include buckets around that boundary.

Histograms can usually be aggregated across instances. Client-side summaries often cannot be combined into globally correct quantiles.

### Resource metrics

Useful capacity dimensions include CPU utilization/throttling, memory pressure, disk latency/IOPS/space, network loss/retransmits, thread/executor pool usage, connection pool usage, queue backlog/age and database waits.

Use resource metrics to explain symptoms, not as a substitute for user-centric SLIs.

## PromQL-style examples

    sum(rate(http_requests_total{status_class="5xx"}[5m]))
    /
    sum(rate(http_requests_total[5m]))

    histogram_quantile(
      0.95,
      sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
    )

## Failure modes

- labels with user/request IDs;
- mixing seconds and milliseconds;
- dashboards based on averages only;
- alerting on absolute CPU without service impact;
- counting retries as successful independent requests;
- ignoring reset behavior after process restarts;
- calculating ratios with mismatched denominators.

## Exercises / Senior Questions

1. Choose counter/gauge/histogram for queue depth, request latency and failed login attempts.
2. Why can p99 from each instance not simply be averaged?
3. Design labels for an HTTP metric without path-cardinality explosion.
4. When would queue age be more meaningful than queue depth?
5. Explain why CPU at 90% can be healthy for one workload and catastrophic for another.

## Related

- [Golden signals, RED and USE](../02-observability/golden-signals-red-use.md)
- [Capacity monitoring](../04-operations/health-capacity-kubernetes.md)
- [DevOps/platform engineering](https://github.com/YosrBennagra/devops-platform-engineering)
