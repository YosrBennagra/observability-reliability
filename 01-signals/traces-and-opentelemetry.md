# Distributed Tracing and OpenTelemetry

## Wall Note / A4

A **trace** represents one end-to-end operation.  
A **span** represents one timed unit of work inside that trace.

Important fields: trace_id, span_id, parent_span_id, service, operation, start/end, status, attributes and events.

Tracing explains **where time and failure propagate across boundaries**.

OpenTelemetry (OTel) standardizes instrumentation and telemetry transport for traces, metrics and logs. It is not itself a single storage backend.

## Detailed Notes

### Trace propagation

~~~mermaid
sequenceDiagram
    participant U as Client
    participant G as API Gateway
    participant O as Order Service
    participant I as Inventory Service
    participant D as Database
    U->>G: HTTP request
    Note over G: trace_id=T1, span=S1
    G->>O: propagate context
    Note over O: parent=S1, span=S2
    O->>I: propagate context
    Note over I: parent=S2, span=S3
    O->>D: SQL call
    Note over D: span=S4
~~~

Context must propagate through HTTP, messaging and asynchronous execution. A broken propagation boundary creates disconnected traces.

### Span design

Create spans around meaningful boundaries: inbound requests, outbound dependency calls, database operations, message produce/consume, expensive internal stages and asynchronous jobs.

Do not create a span for every small function. Excessively fine spans add cost and obscure request shape.

Use bounded attributes such as service version, route template, dependency type and error classification. Avoid secrets and unbounded values.

### Sampling

Strategies include head sampling, tail sampling, probability sampling and policies that retain errors or high-latency traces.

Trade-off: aggressive sampling reduces cost but can hide rare failures. Tail sampling can preserve interesting traces but requires more pipeline resources.

### OpenTelemetry pipeline

~~~mermaid
flowchart LR
    A[Application] --> B[OTel SDK / Agent]
    B --> C[OTLP]
    C --> D[OpenTelemetry Collector]
    D --> E[Processors]
    E --> F[Trace backend]
    E --> G[Metrics backend]
    E --> H[Log backend]
~~~

Collector processors can batch, filter, sample, redact or enrich data. Keep instrumentation vendor-neutral where practical.

### Baggage

Baggage is context propagated across service boundaries. It can carry bounded business metadata, but misuse causes overhead and privacy risk. Never treat baggage as a trusted authorization channel.

### Combining signals

1. metric alert detects elevated error ratio;
2. dashboard narrows service/version;
3. exemplar or trace link opens a failing request;
4. trace finds slow/failing dependency;
5. span links to structured logs;
6. logs provide detailed event context.

## Failure modes

- missing propagation through async executors;
- instrumenting inbound but not outbound operations;
- collecting every trace without cost controls;
- storing secrets in attributes;
- using raw URL instead of route template;
- sampling away rare failures.

## Exercises / Senior Questions

1. Draw the trace for API → queue → worker → database.
2. What is lost when context is not propagated into an async task?
3. Compare head and tail sampling for a rare 500-error investigation.
4. When is a span too fine-grained?
5. Explain how you would correlate an SLO alert to one representative trace.

## Related

- [Logs and correlation](logs-and-correlation.md)
- [Metrics](metrics.md)
- [Spring observability](../05-implementation/spring-databases-distributed-systems.md)
