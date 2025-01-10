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
- Docker container images (Loki,Pomtail,Nginx,httpd)
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


- Go to directory /promtail and do the following steps:

- create promtail configuration file:
  ```
  vim promtail-config.yml

copy and paste the following contents:
```
server:
  http_listen_port: 9080
  grpc_listen_port: 0

clients:
  - url: http://192.168.124.100:3100/loki/api/v1/push

positions:
  filename: /tmp/positions.yaml

scrape_configs:
  - job_name: containers
    static_configs:
      - targets:
          - localhost
        labels:
          job: docker
          __path__: /var/lib/docker/containers/*/*.log
```
  For more explanation on this configuration, please read readme file inside this directory.


Start promtail container image:

```
docker compose up -d
```

- Go directory /loki and do the following steps:


- create promtail configuration file:

  ```
  vim Loki-config.yml
  ```

copy and paste the following contents:

```
auth_enabled: false

server:
  http_listen_port: 3100  # Port to listen for HTTP requests
  http_listen_address: 0.0.0.0

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
  chunk_idle_period: 5m
  chunk_retain_period: 30s
  max_transfer_retries: 0

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb:
    directory: /loki/index
  filesystem:
    directory: /loki/chunks

chunk_store_config:
  max_look_back_period: 0

table_manager:
  retention_deletes_enabled: true
  retention_period: 720h

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h

```
For more explanation on this configuration, please read readme file inside this directory.



Start Loki container image:

  ```
  docker compose up -d

  ```
  - Go to directory /containers to start two web containers images:
 
    ```
    docker compose up -d
    ```
        
- Make sure all container are running successfully :

```
docker ps
```

The output should be like this:

![Screenshot From 2025-01-10 19-21-32](https://github.com/user-attachments/assets/9f5435d4-79cd-4821-9200-a004f675c417)


Now, we have 4 running containers. To test that all running containers send their logs to Promtail (in my case htttpd and nginx containers) run the following command:


```
curl http://192.168.124.100:8080
```

I sent a curl request to my nginx container to the exposed port of 8080 to test logging.


Check that promtail recieved the logs :

```
docker logs promtail grep container ID
docker logs promtail grep c67dfb75ec9d
```

Note: You will find container ID in the first column of docker ps output.

The output should be like this:

![Screenshot From 2025-01-10 19-44-22](https://github.com/user-attachments/assets/8f6b4c75-de27-45f5-baf9-16f94811aa18)


Lets go to Grafana to check and visualize logs:

- Open Grafana in your browser (http://<grafana-host>:3000).
- Add Loki as a data source:
- URL: http://<loki-host>:3100
- Explore logs in the Explore section of Grafana.

  The easiest way to check that Loki is receiving logs from promtail:
  - Click on Grafana Menu.
  - Press 'Explore'
  - Choose Data source to be Loki:
  - 
  You will see screen like this:

![Screenshot From 2025-01-10 20-11-44](https://github.com/user-attachments/assets/04b5177b-0c94-4649-8ffc-b967724f2ca9)


At label filters, select label: filename then choose the container ID file as I discussed earlier.

![Screenshot From 2025-01-10 20-14-24](https://github.com/user-attachments/assets/6bb418e9-d0b2-4c22-a22d-9ae3bf875d2e)


![Screenshot From 2025-01-10 20-15-13](https://github.com/user-attachments/assets/da501cc4-552b-44aa-a32c-047f0dc83d81)


![Screenshot From 2025-01-10 20-15-48](https://github.com/user-attachments/assets/a2e7df09-42c1-42c2-af07-8d1322ef3ec9)


- Another method is by setup a configured Grafana dashboard to view logs collected in the Loki container image.

- Import dashboard ID 13639 to your Grafana, select Loki to be your datasource.

  Dashboard output:

  ![Screenshot From 2025-01-10 20-18-17](https://github.com/user-attachments/assets/1b375eba-1c51-4733-9d4a-f8e615e342a6)

Finally, we have successfully able to collect any running container to Promtail and send them for visualisation and anlaysis to Grafana.

Acknowledgments:

- Grafana Loki Documentation: https://grafana.com/docs/loki/latest/
- Promtail Documentation : https://grafana.com/docs/loki/latest/send-data/promtail/

If you have any enquires, please feel free to reach out.

Thank you.



