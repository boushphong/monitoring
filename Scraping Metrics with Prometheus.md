# Overview
**Prometheus** is primarily a pull-based monitoring system, which means:
- It retrieves metrics by querying the monitored services or hosts at regular intervals.

Prometheus is designed as a pull-based system due to several reasons:
- **Simplicity**: The monitoring server (Prometheus) only needs to know the location (IP address and port) of the targets it should monitor. This simplifies the configuration and management of the monitoring system.
- **Reliability**: If a target goes down, the monitoring system is immediately aware because it cannot pull metrics from the target. In a push-based system, the monitoring system has to infer that a target is down if it stops receiving metrics, which can lead to delays in alerting.
- **Scalability**: You can add more targets without having to reconfigure or restart the monitoring server. This makes it easier to scale the system as your infrastructure grows.
- **Security**: The monitored targets do not need to have access to the monitoring server. This reduces the attack surface and makes it easier to secure the system.
- **Control over Scraping Interval**: You have control over when and how often you scrape data. This can be particularly useful when dealing with targets that can't handle high request rates.

While pull-based systems like Prometheus have these advantages, they also have some disadvantages. For example, they can be more challenging to use in environments with dynamic or ephemeral targets, and they require the monitoring server to be able to reach all targets, which can be difficult in complex network environments.

**NOTE:** When **Prometheus** scrapes a target, it attaches a timestamp to each sample at the moment of the scrape. This timestamp is used when querying the data and for the storage of the data in the time-series database. However, it's important to note that the timestamp is not visible when you're looking at the raw metrics exposed by your service. The timestamp is added by Prometheus at the time of the scrape, and it's used internally for data storage and queries.

## Exporter
An exporter is a service that fetches metrics from a specific source and translates them into a format that Prometheus can understand.
- Many systems do not natively expose their metrics in a format that Prometheus can ingest. An exporter acts as a bridge between that system and Prometheus, gathering data from the system and presenting it in a way that Prometheus can scrape and store.
- Exporters will get the metrics the sources like (Database, Linux servers, cloud watch ...) and Prometheus will pull the data from the exporter.

The process of connecting to an exporter and pulling their metrics into Prometheus is called **Scraping**. 
**Scraping** can be configured in Prometheus config file. The default pull interval is 15 seconds, every 15 seconds Prometheus will connect to the exporter and pull the metrics.

The exporter in Prometheus is typically installed and run on the same machine or in the same environment as the service you want to monitor.

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

## Alert Manager
**Alert Manager** is a component of Prometheus that handles alerts sent by client applications such as the Prometheus server. It takes cares of deduplicating, grouping, and routing them to the correct receiver integrations such as email, PagerDuty ..., 

1. **Alerting rules in Prometheus servers**: Prometheus servers send alerts to the Alertmanager. These alerts are based on alerting rules in the Prometheus servers. When the conditions in an alerting rule are met (i.e., when certain thresholds are breached), an alert is fired to the Alertmanager.
2. **Grouping of alerts**: Alertmanager groups received alerts by the label value in the alerts. This is useful in grouping similar alerts together, which can help in reducing the noise and focusing on the problem at hand.
3. **Inhibition**: Alertmanager supports a concept of inhibition where an alert can mute notifications for certain other alerts if certain label sets match.
4. **Silencing**: Alertmanager also supports manually silencing notifications for a given alert.
5. **Sending notifications**: After deduplication, grouping, and applying inhibition and silencing, Alertmanager sends notification about the alerts to the specified receiver.

## Exposing Metrics with Python Prometheus Client
```python
from prometheus_client import start_http_server, Summary, Counter
import random
import time

# Create a metric to track time spent and requests made.
REQUEST_TIME = Summary('request_processing_seconds', 'Time spent processing request')
MY_COUNTER = Counter('my_counter', 'Description of counter', ['function'])


@REQUEST_TIME.time()
def process_request(t):
    """A dummy function that takes some time."""
    MY_COUNTER.labels('process_request').inc()
    time.sleep(t)


if __name__ == '__main__':
    # Start up the server to expose the metrics.
    start_http_server(8000)

    process_request(random.random())
    # Generate some requests.
    while True:  # Also to run indefinitely to keep the server alive on port 8000
        process_request(random.random())
        time.sleep(3)
```
**Result:**
```
# HELP request_processing_seconds Time spent processing request
# TYPE request_processing_seconds summary
request_processing_seconds_count 1.0
request_processing_seconds_sum 0.47328010000001086
# HELP request_processing_seconds_created Time spent processing request
# TYPE request_processing_seconds_created gauge
request_processing_seconds_created 1.6968679443503273e+09
# HELP my_counter_total Description of counter
# TYPE my_counter_total counter
my_counter_total{function="process_request"} 2.0
# HELP my_counter_created Description of counter
# TYPE my_counter_created gauge
my_counter_created{function="process_request"} 1.6968679443512368e+09
```

## Scraping Metrics from a Flask App
`main.py`
```python
from flask import Flask, Response
from prometheus_client import Counter, generate_latest

app = Flask(__name__)

# Create a counter metric
requests_total = Counter("requests_total", "Total number of HTTP requests.")


@app.route('/')
def hello_world():
    # Increment the counter
    requests_total.inc()
    return 'Hello, World!'


@app.route('/metrics')
def metrics():
    # Expose the metrics
    return Response(generate_latest(), mimetype='text/plain')


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
`prometheus.yml`
```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: 'flask_app'
    static_configs:
      - targets: ['host.docker.internal:5000']  # Mac OS internal host
```

## Service Discovery
Service discovery in Prometheus works by continuously scanning for changes in your environment and updating the list of targets (i.e., instances to be monitored). This is done by querying a service discovery mechanism, which could be a cloud provider's API, a Kubernetes API server, a DNS server, or a configuration management database. There are many service discovery mechanisms, including:
- **Static Configurations**: Manually specify the targets and parameters.
- **DNS-Based Discovery**: Use DNS SRV records to discover target services.
- **File-Based Discovery**: Use files (in JSON or YAML format) to specify the targets.
- **Kubernetes**: Discover services in a Kubernetes cluster.
- And many more ... Cloud, Consul, OpenStack ...

## Push Gateway
When an application wants to send its metrics to Prometheus in a push-based manner, we'd need a **Push Gateway**.

**Push Gateway** is a component of Prometheus, it acts as a temporary storage where applications can send their metrics to. In addition, it has a built-in exporter in it so Prometheus can pull the metrics from the exporter within the **Push Gateway**

**Sample code:**
```python
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway

# Create a new registry
registry = CollectorRegistry()

# Create a new Gauge metric
g = Gauge('job_last_success_unixtime', 'Last time a batch job successfully finished', registry=registry)

# Set the Gauge to the current unixtime
g.set_to_current_time()

# Push the metric to the Pushgateway
push_to_gateway('localhost:9091', job='batch_job', registry=registry)
```
