# Overview
**Prometheus** and **Grafana** is primarily a pull-based monitoring system, which means:
- It retrieves metrics by querying the monitored services or hosts at regular intervals.

Prometheus is designed as a pull-based system due to several reasons:
- **Simplicity**: The monitoring server (Prometheus) only needs to know the location (IP address and port) of the targets it should monitor. This simplifies the configuration and management of the monitoring system.
- **Reliability**: If a target goes down, the monitoring system is immediately aware because it cannot pull metrics from the target. In a push-based system, the monitoring system has to infer that a target is down if it stops receiving metrics, which can lead to delays in alerting.
- **Scalability**: You can add more targets without having to reconfigure or restart the monitoring server. This makes it easier to scale the system as your infrastructure grows.
- **Security**: The monitored targets do not need to have access to the monitoring server. This reduces the attack surface and makes it easier to secure the system.
- **Control over Scraping Interval**: you have control over when and how often you scrape data. This can be particularly useful when dealing with targets that can't handle high request rates.

While pull-based systems like Prometheus have these advantages, they also have some disadvantages. For example, they can be more challenging to use in environments with dynamic or ephemeral targets, and they require the monitoring server to be able to reach all targets, which can be difficult in complex network environments.

### Exporter
An exporter is a service that fetches metrics from a specific source and translates them into a format that Prometheus can understand.
- Many systems do not natively expose their metrics in a format that Prometheus can ingest. An exporter acts as a bridge between that system and Prometheus, gathering data from the system and presenting it in a way that Prometheus can scrape and store.
- Exporters will get the metrics the sources like (Database, Linux servers, cloud watch ...) and Prometheus will pull the data from the exporter.

The process of connecting to an exporter and pulling their metrics into Prometheus is called **Scraping**. 
**Scraping** can be configured in Prometheus config file. The default pull interval is 15 seconds, every 15 seconds Prometheus will connect to the exporter and pull the metrics.

The exporter in Prometheus is typically installed and run on the same machine or in the same environment as the service you want to monitor.

### Push Gateway
When an application wants to send its metrics to Prometheus in a push-based manner, we'd need a **Push Gateway**.

**Push Gateway** is a component of Prometheus, it acts as a temporary storage where applications can send their metrics to. In addition, it has a built-in exporter in it so Prometheus can pull the metrics from the exporter within the **Push Gateway**
