## TLDR;

Here is a loom video explaining how to apply advanced label filters to the metrics using OpenTelemetry collector.

https://www.loom.com/share/87499a7bbf3d4028acc89fe4c09655f7

If you like reading instead of watching video, I have you covered. [Filtering Metrics by Labels in OpenTelemetry Collector](https://last9.io/blog/filtering-metrics-by-labels-in-opentelemetry-collector/)


## Getting started

Make sure you have docker and docker-compose setup as we will be using it to run the opentelemetry collector as well
as the golang app that generates metrics for the purpose this demo.

You should also use a metric storage backend where you can verify the filtered data. I am using [Levitate](https://last9.io/levitate-tsdb) by Last9 as my metric storage here but you can use any other as well.

> Refer to [Getting started with Levitate](https://docs.last9.io/docs/levitate-onboard) guide to setup cluster on Levitate and obtain the basic auth username, password and remote write URL.

Replace the following sections from `otel-collector-config.yaml` with the your metric storage destination's
username, password and remote write URL respectively.

>

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

- Start the setup using `docker-compose up`.
- Try hitting cURL requests to `localhost:9000/hello` and `localhost:9000/world` multiple times to generate time series for app metrics.
- Observe the metrics on `localhost:8000/metrics` and update the filter configuration in the `otel-collector-config.yaml`.


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

After that, restart the process again using `docker-compose`, and verify that collector is able to pass on filtered time series to different backends based on the filters you created.
