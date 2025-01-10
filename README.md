# Central-logging-with-promtail-and-Loki
Setup a simple solution to monitor logs for all running containers on docker.

Centralized Docker Logging with Promtail and Loki

Introduction:

In containerized environments, managing logs from multiple Docker containers can be challenging. By default, Docker uses a logging driver to capture logs from containers. These logs are typically stored locally or routed to external systems. The most common logging drivers include:

- json-file (default): Stores container logs as JSON files on the host.
- local: A lightweight logging mechanism storing logs on the host.
- syslog or journald: Sends logs to system-level logging services.
- Remote logging drivers: Forward logs to external tools like Fluentd, AWS CloudWatch, or Elasticsearch.
- However, to enable advanced querying, filtering, and visualization, integrating Docker logs with systems like Grafana Loki offers a modern, scalable approach.
  

By default, Docker stores all all running contaienrs in the the path:/var/lib/docker/containers/. In this task, I will configure promtail to collect all logs from default logging path and send them to Loki to view these logs in Loki dashboard.

For more information on Docker Logging: https://docs.docker.com/engine/logging/configure/ 


Why Use Promtail and Loki?

Loki: A log aggregation system designed for storing and querying log data, similar to "Prometheus for logs."

Promtail: A lightweight log shipper that collects logs from containers, files, or systemd/journald and sends them to Loki.

Using Promtail and Loki together allows you to:

- Collect logs from all running Docker containers.
- Centralize and store logs efficiently in Loki.
- Visualize and query logs seamlessly in Grafana.
- This repository demonstrates how to configure Promtail and Loki to collect and aggregate logs from Docker containers, enabling real-time monitoring and debugging.

Promtail and Loki Setup for Container Log Aggregation
Overview

This repository contains a Docker Compose setup for centralized log aggregation and visualization using Grafana Loki and Promtail. It is configured to collect logs from running Docker containers and send them to Loki, where they can be visualized and queried in Grafana.

Features:
- Centralized logging using Loki.
- Log collection from Docker containers via Promtail.
- Easy-to-deploy setup using Docker Compose.
- Seamless integration with Grafana for log visualization.

Requirements:
- Docker (v20.10 or later).
- Docker Compose (v2.0 or later).
- Basic knowledge of Docker and Grafana.

Setup and Usage:

1- Clone the Repository:

git clone https://github.com/Omar-Moh-Ibrahim/Central-logging-with-promtail-and-Loki.git 


2- Once you downlaoded the repo in your computer, lets start:

- Make sure that the default docker defualt logging driver is json.file by running the following command:


```
docker info | grep "Logging Driver"
```

The output should be like this:


![Screenshot From 2025-01-10 18-41-01](https://github.com/user-attachments/assets/38566747-19c4-41a8-b056-17f24028b754)

        







