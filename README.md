# minio-alertmanager

**minio-alertmanager** is a project that integrates MinIO, Prometheus, Grafana, and AlertManager. It allows you to monitor MinIO metrics, set up alerts, and visualize data using Grafana dashboards. Additionally, it includes an alert producer service written in Node.js that sends alerts to a Gotify server.

## Features

1. **MinIO**: A high-performance object storage service.
2. **Prometheus**: A monitoring and alerting toolkit.
3. **Grafana**: A powerful visualization platform.
4. **Gotify**: A simple server for sending and receiving messages in real-time per WebSocket. 
5. **AlertManager**: A tool for handling alerts and notifications.
6. **Alert Producer (Node.js)**: A custom service that generates alerts and sends them to a Gotify server.

## Installation and Configuration

1. **Clone the Repository**:

   ```
   git clone https://github.com/azita-abdollahi/minio-alertmanager.git
   ```

   

2. **Configure Environment Variables**:

   - Create a `config` directory with necessary services config files (e.g., Alert rules settings, Prometheus settings).
   - Customize the configuration according to your deployment.

3. **Docker Compose**:

   - Use `docker-compose.yml` to define your services (Prometheus, Grafana, Alert producer and AlertManager).
   - Adjust ports, volumes, and other settings as needed.

## Monitoring with Prometheus

- MinIO publishes cluster, node, bucket, and resource metrics using the Prometheus Data Model.
- Configure Prometheus to scrape MinIO metrics:
  - For the MinIO cluster: Use `mc admin prometheus generate ALIAS` (replace ALIAS with your MinIO alias).
  - [For nodes, buckets, and resources: Adjust the scrape configuration accordingly](https://min.io/docs/minio/container/operations/monitoring/collect-minio-metrics-using-prometheus.html)[1](https://min.io/docs/minio/container/operations/monitoring/collect-minio-metrics-using-prometheus.html).

## Setting Up Grafana Dashboards

1. **Access Grafana**:
   - After starting the services, access Grafana at `http://localhost:3000`.
   - Log in with the default credentials (admin/admin).
2. **Add Prometheus as a Data Source**:
   - In Grafana, add Prometheus as a data source.
   - Use the URL of your Prometheus instance (e.g., `http://prometheus:9090`).
3. **create Dashboards**:
   - Import MinIO-related dashboards from Grafana’s community dashboards or create custom ones. (i use [minio-dashbord](https://github.com/azita-abdollahi/minio-alertmanager/blob/master/config/grafana/dashboards/minio-rev26.json))
   - Visualize MinIO metrics, such as drive usage, latency, and cluster health.

## Alerts with AlertManager

- Define alert rules in Prometheus or Grafana.

- Example alert rule:

  - Alert if MinIO cluster tolerance drops below a certain threshold:

    ```yaml
    - alert: High Minio Requests
        expr: minio_s3_requests_total > 100000
        for: 5m
        labels:
          severity: warn
        annotations:
          title: "Instance {{ $labels.instance }} High MinIO usage detected"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has too MinIO HTTP requests in the last 5 minutes."
          priority: "5"
    ```

- Define alert rules in Prometheus or Grafana (as mentioned in the previous section).
- AlertManager will handle notifications based on these rules.

## Custom Alert Producer (Node.js)

- The alert producer service in Node.js generates custom alerts.
- Configure it to send alerts to your Gotify server.
- Example alert scenarios:
  - Low disk space on MinIO nodes.
  - High latency for MinIO requests.

for setup `minio` see my [minio-repository](https://github.com/azita-abdollahi/minio).

`Note`: be sure to leave this line in the minio service environment setting.

```yaml
- MINIO_PROMETHEUS_AUTH_TYPE=public
```

The `MINIO_PROMETHEUS_AUTH_TYPE` environment variable controls authentication for scraping MinIO metrics endpoints by Prometheus. 

1. **Default Behavior**:
   - By default, MinIO requires authentication for scraping metrics endpoints.
   - Prometheus must include a bearer token in the scrape configuration to access MinIO metrics.
2. **Setting `MINIO_PROMETHEUS_AUTH_TYPE` to ‘public’**:
   - When you set `MINIO_PROMETHEUS_AUTH_TYPE` to ‘public’, MinIO allows unauthenticated access to its metrics endpoints.
   - This means you can scrape MinIO metrics without the need for a token in your Prometheus scrape configuration.