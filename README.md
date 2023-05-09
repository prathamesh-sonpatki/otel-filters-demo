## Getting started

Replace the following sections from `otel-config.yaml` with the Levitate cluster's
username, password and remote write URL respectively.

```yaml
extensions:
  basicauth/prw/1:
    client_auth:
      # Replace these variables by their actual values
      username: username1
      password: password1
  basicauth/prw/2:
    client_auth:
      # Replace these variables by their actual values
      username: username2
      password: password2
```

```yaml
exporters:
  prometheusremotewrite/1:
    auth:
      authenticator: basicauth/prw
    endpoint: "<Cluster 1>" # Replace with backend 1 URL
  prometheusremotewrite/2:
    auth:
      authenticator: basicauth/prw
    endpoint: "<Cluster 2>" # Replace with backend 2 URL
```

Start the setup using `docker-compose up`

Observe the metrics on `localhost:8888/metrics` and update the filter configuration in the `otel-config.yaml`

> Here is more [information](https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/processor/filterprocessor/README.md#using-an-expr-match_type) on how label based filters can be applied on metrics.

```yaml
processors:
  filter/1:
    metrics:
      exclude:
        match_type: expr
        expressions:
        - MetricName == "my.metric" && Label("my_label") == "abc123"
  filter/2:
    metrics:
      exclude:
        match_type: expr
        expressions:
        - MetricName == "my.other.metric"
```

After that, restart the process again, and verify that collector is able to pass on filtered samples
to different backends.
