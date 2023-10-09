# Overview
**Prometheus** and **Grafana** is primarily a pull-based monitoring system, which means:
- It retrieves metrics by querying the monitored services or hosts at regular intervals.

Prometheus is designed as a pull-based system due to several reasons:
- **Simplicity**: The monitoring server (Prometheus) only needs to know the location (IP address and port) of the targets it should monitor. This simplifies the configuration and management of the monitoring system.
- **Reliability**: If a target goes down, the monitoring system is immediately aware because it cannot pull metrics from the target. In a push-based system, the monitoring system has to infer that a target is down if it stops receiving metrics, which can lead to delays in alerting.
- **Scalability**: You can add more targets without having to reconfigure or restart the monitoring server. This makes it easier to scale the system as your infrastructure grows.
- **Security**: The monitored targets do not need to have access to the monitoring server. This reduces the attack surface and makes it easier to secure the system.
- **Control over Scraping Interval**: You have control over when and how often you scrape data. This can be particularly useful when dealing with targets that can't handle high request rates.

While pull-based systems like Prometheus have these advantages, they also have some disadvantages. For example, they can be more challenging to use in environments with dynamic or ephemeral targets, and they require the monitoring server to be able to reach all targets, which can be difficult in complex network environments.

## Exporter
An exporter is a service that fetches metrics from a specific source and translates them into a format that Prometheus can understand.
- Many systems do not natively expose their metrics in a format that Prometheus can ingest. An exporter acts as a bridge between that system and Prometheus, gathering data from the system and presenting it in a way that Prometheus can scrape and store.
- Exporters will get the metrics the sources like (Database, Linux servers, cloud watch ...) and Prometheus will pull the data from the exporter.

The process of connecting to an exporter and pulling their metrics into Prometheus is called **Scraping**. 
**Scraping** can be configured in Prometheus config file. The default pull interval is 15 seconds, every 15 seconds Prometheus will connect to the exporter and pull the metrics.

The exporter in Prometheus is typically installed and run on the same machine or in the same environment as the service you want to monitor.

## Push Gateway
When an application wants to send its metrics to Prometheus in a push-based manner, we'd need a **Push Gateway**.

**Push Gateway** is a component of Prometheus, it acts as a temporary storage where applications can send their metrics to. In addition, it has a built-in exporter in it so Prometheus can pull the metrics from the exporter within the **Push Gateway**

## Configuring Prometheus
```yaml
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds.
  evaluation_interval: 15s # Evaluate rules every 15 seconds.

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['somehost:9100']
```
The global section sets default values for scrape_interval and evaluation_interval. This means Prometheus will scrape metrics from its targets every 15 seconds and evaluate rules every 15 seconds.

The scrape_configs section defines the jobs and targets that Prometheus will scrape metrics from. In this case, there are two jobs defined:

- The first job is named `prometheus` and scrapes metrics from Prometheus itself (which exposes metrics about its own operation at localhost:9090).
- The second job is named `node_exporter` and scrapes metrics from the Node Exporter running on the machine with domain `somehost` (which exposes metrics at somehost:9100).

## Data Types
Sure, let's look at some examples of queries using these data types in PromQL, Prometheus's query language.

1. **Scalar**: 

A scalar represents a simple numeric floating point value. It's often used in arithmetic operations with instant vectors. For example:

```javascript
http_requests_total > 100
```
This query will return an instant vector consisting of the current values of the `http_requests_total` metric for all time series where the value is greater than 100.

**Returns**: All HTTP requests that are larger than the current value of 100.

```javascript
http_requests_total{method='GET' ...} 101
http_requests_total{method='POST' ...} 311
```

2. **Instant Vector**: 

An instant vector consists of a set of time series containing a single sample for each time series, all sharing the same timestamp. Here's an example of a query that returns an instant vector:

```javascript
http_requests_total
```

This query will return the current value of the `http_requests_total` metric for each time series that has this metric.

3. **Range Vector**: 

A range vector consists of a set of time series containing a range of data points over time for each time series. Here's an example of a query that uses a range vector:

```javascript
rate(http_requests_total[5m])
```

This query will return an instant vector where each value is the per-second rate of increase of the `http_requests_total` counter over the last 5 minutes.

**Returns**:
```javascript
http_requests_total{method="get", code="200"} 2.3
http_requests_total{method="post", code="200"} 1.7
http_requests_total{method="get", code="404"} 0.1
http_requests_total{method="post", code="500"} 0.05
```
This means:
- Your server has been receiving GET requests that result in a 200 status code at an average rate of 2.3 requests per second over the last 5 minutes.
- Your server has been receiving POST requests that result in a 200 status code at an average rate of 1.7 requests per second over the last 5 minutes.
- Your server has been receiving GET requests that result in a 404 status code at an average rate of 0.1 requests per second over the last 5 minutes.
- Your server has been receiving POST requests that result in a 500 status code at an average rate of 0.05 requests per second over the last 5 minutes.

4. **Aggregation**:

PromQL also supports aggregation operators that can be used to aggregate the same kind of data across many different time series, based on their labels. Here's an example:

```javascript
sum(rate(http_requests_total[5m])) by (method)
```

This query will return the sum of the rate of `http_requests_total` over the last 5 minutes, grouped by the `method` label (e.g., GET, POST).

Remember, these are just simple examples. PromQL is a powerful query language that supports a wide range of functions, operators, and aggregations, allowing you to construct complex queries to analyze your metrics.
