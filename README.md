# OpenTelemetry Service Dashboard

Grafana dashboards for OpenTelemetry services.

[Grafana dashboard for OpenTelemetry services](docs/images/otel-svc-dashboard-screenshot-01.png)

## Setup

* Instrument services with OpenTelemetry SDK libraries and auto-instrumentation agents. Ensure the instrumentation produces:
  * Traces
  * Metrics: [HTTP metrics](https://opentelemetry.io/docs/specs/semconv/http/http-metrics/), [gRPC metrics](https://opentelemetry.io/docs/specs/semconv/rpc/rpc-metrics/), or [Database Client Metrics](https://opentelemetry.io/docs/specs/semconv/database/database-metrics/)
  * Logs: optional
* Send generated telemetry (details in FAQ below):
  * Traces to [Grafana Tempo](https://grafana.com/oss/tempo/)
  * Metrics to a [Prometheus](https://prometheus.io/) database like [Grafana Mimir](https://grafana.com/oss/mimir/)
  * Logs to [Grafana Loki](https://grafana.com/oss/loki/) v3.0+ using the Loki OTLP/HTTP endpoint
* Ensure that a datasource is setup in Grafana for each of these Tempo, Prometheus, and Loki databases
* In Grafana, create the "OpenTelemetry Service" dashboard:
  * Navigate to "Dashboards" then click on the "New / New dashboard" button
  * Click on "Import a dashboard"
  * On the "Import dashboard" screen, enter the ID `TODO` then click on the "Load" button

## FAQ

## How to send OpenTelemetry traces, metrics, and logs to Grafana Tempo, Mimir, and Loki

### Grafana Cloud

When using Grafana Cloud, follow the instructions of he Grafana Cloud documentation page
 [OpenTelemetry > Send data to the Grafana Cloud OTLP endpoint](https://grafana.com/docs/grafana-cloud/send-data/otlp/send-data-otlp/).

### Self managed Tempo, Mimir, and Loki

Example OpenTelemetry Collector configuration to send to self managed instances of Tempo, Mimir, and Loki:

Replace `tempo.example.com` , `mimir.example.com` , and `loki.example.com` by the desired host names.

For production deployments, enable TLS security and remove `insecure: true`.

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:

exporters:
  otlphttp/metrics:
    endpoint: http://mimir.example.com:9090/api/v1/otlp
    tls:
      insecure: true
  otlphttp/traces:
    endpoint: http://tempo.example.com:4418
    tls:
      insecure: true
  otlphttp/logs:
    endpoint: http://loki.example.com:3100/otlp
    tls:
      insecure: true
  debug/metrics:
    verbosity: detailed
  debug/traces:
    verbosity: detailed
  debug/logs:
    verbosity: detailed

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlphttp/traces]
      #exporters: [otlphttp/traces,debug/traces]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlphttp/metrics]
      #exporters: [otlphttp/metrics,debug/metrics]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlphttp/logs]
      #exporters: [otlphttp/logs,debug/logs]
```



## The drop down list for services is empty

## License

 Apache-2.0 license.
