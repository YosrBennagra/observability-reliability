# Logs, Structured Logging and Correlation IDs

## Wall Note / A4

Logs are **discrete events**, not a database dump.

A useful log event answers: **what happened, when, where, to whom, under which operation, with what outcome?**

Prefer structured fields over message parsing.

Essential fields commonly include:

    timestamp, severity, service, environment, version,
    trace_id, span_id, request_id/correlation_id,
    operation, outcome, duration_ms, error.type

Never log secrets, tokens, passwords or unnecessary personal data.

## Detailed Notes

### Event design

Bad:

    Something failed for user

Better:

    {
      "timestamp":"2026-10-03T18:30:15Z",
      "level":"ERROR",
      "service":"orders",
      "operation":"create_order",
      "outcome":"failure",
      "error.type":"InventoryUnavailable",
      "trace_id":"...",
      "tenant_id":"t-42",
      "duration_ms":182
    }

Structured fields allow filtering, aggregation and stable dashboards. Do not encode every field into a free-text message.

### Correlation IDs vs trace IDs

A **correlation/request ID** is an application-level identifier used to connect related events. A **trace ID** belongs to a distributed trace and should usually become the primary cross-service correlation key when tracing is available.

Practical rule:

- generate or accept a request identifier at ingress;
- validate externally supplied identifiers before trusting them;
- propagate it downstream;
- attach it to logs;
- include trace/span IDs when using OpenTelemetry.

Do not use correlation IDs as authentication or authorization evidence.

### Severity

- **DEBUG**: high-detail diagnostics, normally disabled or sampled in production.
- **INFO**: meaningful lifecycle/business events.
- **WARN**: abnormal condition that was handled but deserves attention.
- **ERROR**: operation failed or user-visible behavior was degraded.

Severity should reflect consequence, not developer emotion.

### Exception logging

Log an exception once at the boundary that can add meaningful context. Repeatedly logging the same exception through every layer creates noise and duplicate incident counts.

Include exception type, operation, stable error code, trace ID and safe context. Avoid logging full request/response bodies by default.

### Cost and cardinality

Logs can be expensive because each event is retained and indexed. Control volume, field indexing, retention, duplicate stack traces, verbose framework logs and high-frequency success events.

Use metrics for cheap aggregation, traces for request shape and logs for high-value discrete evidence.

## Spring example

    try {
        orderService.create(command);
        log.info("order_create_success orderId={} customerId={} traceId={}",
                orderId, safeCustomerId, traceId);
    } catch (InventoryUnavailable ex) {
        log.warn("order_create_inventory_unavailable sku={} traceId={}",
                sku, traceId);
        throw ex;
    }

Prefer an MDC/logging context for request-scoped fields so every line carries correlation automatically.

## Investigation workflow

~~~mermaid
flowchart LR
    A[Alert or symptom] --> B[Find affected trace/request]
    B --> C[Filter logs by trace/service/version]
    C --> D[Build event timeline]
    D --> E[Compare healthy vs failing path]
    E --> F[Form hypothesis]
    F --> G[Verify with metrics/traces]
~~~

## Failure modes

- multiline unstructured logs that are difficult to parse;
- unique dynamic text preventing aggregation;
- high-cardinality fields indexed unnecessarily;
- sensitive information leakage;
- clock skew breaking event ordering;
- missing deployment version;
- logging only errors but no important state transitions;
- logging successful high-frequency requests individually when metrics suffice.

## Exercises / Senior Questions

1. Design a structured event for a failed payment attempt without leaking card data.
2. When should a trace ID replace a custom correlation ID?
3. Why can logging every SQL statement become an operational hazard?
4. How would you detect duplicate exception logging?
5. Which fields would you include to compare failures across deployment versions?

## Related

- [Tracing and OpenTelemetry](traces-and-opentelemetry.md)
- [Production debugging](../05-implementation/production-debugging-chaos.md)
- [Application security](https://github.com/YosrBennagra/application-security)
