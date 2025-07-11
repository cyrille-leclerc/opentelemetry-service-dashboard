







## Error percentage alert

```promql
(
  sum by(deployment_environment_name, service_namespace, service_name) (
    rate(
      http_server_request_duration_seconds_count{
        deployment_environment_name=~"production",
        service_namespace=~"ecommerce",
        service_name="frontend",
        http_request_method="POST",
        http_route="/api/orders",
        http_response_status_code=~"5.."
      }[$__rate_interval] offset 2m
    )
  )
  * 100
)
/
sum by(deployment_environment_name, service_namespace, service_name) (
  rate(
    http_server_request_duration_seconds_count{
      deployment_environment_name=~"production",
      service_namespace=~"ecommerce",
      service_name="frontend",
      http_request_method="POST",
      http_route="/api/orders"
    }[$__rate_interval] offset 2m
  )
)
```

## P95 response time alert

### PromQL

```promql
histogram_quantile(
  0.95,
  sum by(le, deployment_environment_name, service_namespace, service_name, http_request_method, http_route, service_instance_id) (
    rate(
      http_server_request_duration_seconds_bucket{
        deployment_environment_name="production",
        service_namespace="ecommerce",
        service_name="frontend",
        http_request_method="POST",
        http_route="/api/orders"
      }[$__rate_interval] offset 2m
    )
  )
)
```

### Prometheus Alert Manager

```yaml
alert: EcommerceFrontendServicePlaceOrderHighLatencyProm
expr: |-
  histogram_quantile(
    0.95,
    sum by(le, deployment_environment_name, service_namespace, service_name, http_request_method, http_route, service_instance_id) (
      rate(
        http_server_request_duration_seconds_bucket{
          deployment_environment_name="production",
          service_namespace="ecommerce",
          service_name="frontend",
          http_request_method="POST",
          http_route="/api/orders"
        }[2m] offset 2m
      )
    )
  ) >2
for: 2m
labels:
  severity: critical
  team_name: ecommerce
annotations:
  summary: >-
    High P95 for {{ $labels.service_namespace }}/{{ $labels.service_name }} "{{
    $labels.http_request_method }} {{ $labels.http_route }}" ({{
    $labels.deployment_environment_name }})
  description: >-
    The 95th percentile response time for operation {{ $labels.service_namespace
    }}/{{ index $labels.service_name }} "{{ $labels.http_request_method }} {{
    $labels.http_route }}" ({{ $labels.deployment_environment_name }}) has been
    above 2 seconds for 2 minutes on {{ $labels.service_instance_id}}. Current
    value: {{ $value }}.
```


### Grafana Alert Manager

* PromQL

```
histogram_quantile(
  0.95,
  sum by(le, deployment_environment_name, service_namespace, service_name, http_request_method, http_route, service_instance_id) (
    rate(
      http_server_request_duration_seconds_bucket{
        deployment_environment_name="production",
        service_namespace="ecommerce",
        service_name="frontend",
        http_request_method="POST",
        http_route="/api/orders"
      }[$__rate_interval] offset 2m
    )
  )
)
```

* Alert condition / when query: `is above` value `1` (decimal, number of seconds)
* Labels
  * `severity: critical`
  * `team_name:  ecommerce`
* Summary
  
```
High P95 for {{ index $labels "service_namespace" }}/{{ index $labels "service_name" }} "{{ index $labels "http_request_method" }} {{ index $labels "http_route" }}" ({{ index $labels "deployment_environment_name" }})
```

* Description

```
The 95th percentile response time for operation {{ index $labels "service_namespace" }}/{{ index $labels "service_name" }} "{{ index $labels "http_request_method" }} {{ index $labels "http_route" }}" ({{ index $labels "deployment_environment_name" }}) has been above 2 seconds for 2 minutes on {{ index $labels "service_instance_id"}}. Current value: {{ index $values "A" }}.
```

YAML Export

<details>
<summary>Grafana Alert rule YAML export</summary>

```yaml
apiVersion: 1
groups:
    - orgId: 1
      name: ecommerce
      folder: Cyrille
      interval: 1m
      rules:
        - uid: eerlkai0j203kb
          title: EcommerceFrontendServicePlaceOrderHighLatency
          condition: C
          data:
            - refId: A
              relativeTimeRange:
                from: 600
                to: 0
              datasourceUid: grafanacloud-prom
              model:
                editorMode: code
                expr: |-
                    histogram_quantile(
                      0.95,
                      sum by(le, deployment_environment_name, service_namespace, service_name, http_request_method, http_route, service_instance_id) (
                        rate(
                          http_server_request_duration_seconds_bucket{
                            deployment_environment_name="production",
                            service_namespace="ecommerce",
                            service_name="frontend",
                            http_request_method="POST",
                            http_route="/api/orders"
                          }[$__rate_interval] offset 2m
                        )
                      )
                    )
                instant: true
                intervalMs: 1000
                legendFormat: __auto
                maxDataPoints: 43200
                range: false
                refId: A
            - refId: C
              datasourceUid: __expr__
              model:
                conditions:
                    - evaluator:
                        params:
                            - 2
                        type: gt
                      operator:
                        type: and
                      query:
                        params:
                            - C
                      reducer:
                        params: []
                        type: last
                      type: query
                datasource:
                    type: __expr__
                    uid: __expr__
                expression: A
                intervalMs: 1000
                maxDataPoints: 43200
                refId: C
                type: threshold
          noDataState: NoData
          execErrState: Error
          for: 1m
          annotations:
            description: 'The 95th percentile response time for operation {{ index $labels "service_namespace" }}/{{ index $labels "service_name" }} "{{ index $labels "http_request_method" }} {{ index $labels "http_route" }}" ({{ index $labels "deployment_environment_name" }}) has been above 2 seconds for 2 minutes on {{ index $labels "service_instance_id"}}. Current value: {{ index $values "A" }}.'
            summary: High P95 for {{ index $labels "service_namespace" }}/{{ index $labels "service_name" }} "{{ index $labels "http_request_method" }} {{ index $labels "http_route" }}" ({{ index $labels "deployment_environment_name" }})
          labels:
            severity: critical
            team_name: ecommerce
          isPaused: false
          notification_settings:
            receiver: grafana-oncall

```

</details>


# SLO

Requests complete successfully in `1s` max:

```
(
  sum(
    rate(
      http_server_request_duration_seconds_bucket{
        deployment_environment_name="production",
        http_request_method="POST",
        http_response_status_code=~"2..",
        http_route="/api/orders",
        le="1",
        service_name="frontend",
        service_namespace="ecommerce"
      }[$__rate_interval] offset 2m
    )
  )
  or
  0
)
/
sum(
  rate(
    http_server_request_duration_seconds_bucket{
      deployment_environment_name="production",
      http_request_method="POST",
      http_route="/api/orders",
      le="+Inf",
      service_name="frontend",
      service_namespace="ecommerce"
    }[$__rate_interval] offset 2m
  )
)
```


