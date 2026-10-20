# Grafana Dashboards

## Conflict Monitor — Actuator & Streams

Provisioned file: [`conflictmonitor-actuator.json`](conflictmonitor-actuator.json)  
UID: `cm-actuator-streams` · Title: **Conflict Monitor — Actuator & Streams**

### Metrics covered

Published by Conflict Monitor and scraped from `/actuator/prometheus`.

| Source | Prometheus names | Tags |
|--------|------------------|------|
| Micrometer JVM / process / system binders | `process_cpu_usage`, `system_cpu_usage`, `system_cpu_count`, `process_uptime_seconds`, `jvm_memory_*`, `jvm_threads_*`, `jvm_gc_pause_*` | `application=conflictmonitor` |
| `KafkaStreamsMetricsBinder` | `cm_streams_process_ratio`, `cm_streams_process_latency_avg_ms`, `cm_streams_process_latency_max_ms`, `cm_streams_process_rate`, `cm_streams_poll_ratio` | `topology`, `application_id`, `application`, plus scrape labels `job` and `instance` |

Dashboard panels:

1. **Overview** — process/system CPU, heap, live threads, max topology process ratio  
2. **Kafka Streams Topology Compute** — snapshot table of all five `cm_streams_*` gauges (sorted by process ratio), plus time series for process ratio, latency avg/max, process rate, and poll ratio  
3. **JVM / Process Detail** — CPU over time, memory by area, GC pause rate, threads, uptime, CPU count, registered topology gauge count

The dashboard scopes series by job and instance so JVMs remain distinct when multiple targets are selected. The custom `cm_streams_*` gauges are sufficient for the topology panels. Optional native Micrometer Kafka metrics appear only when instrumented clients register them.

### Requirements

1. Conflict Monitor running with scrape on port **8082**
2. Prometheus job `conflictmonitor` in [`../../prometheus/prometheus.yml`](../../prometheus/prometheus.yml) (`metrics_path: /actuator/prometheus`)
3. Monitoring stack up from `jpo-utils` (e.g. `docker compose -f docker-compose.yml --profile monitoring_full up`)

Open Grafana → **Conflict Monitor — Actuator & Streams**.

Template variables: Datasource · Job · Application · Instance · Topology.

### Verify scrape

```bash
curl -sS -o /dev/null -w "%{http_code}\n" http://localhost:8082/actuator/prometheus
curl -sS http://localhost:8082/actuator/prometheus | findstr cm_streams
curl -sS http://localhost:8082/health/streams/cpu
```

The process and poll ratios are Kafka Streams wall-time fractions averaged across thread metrics, so they indicate topology activity rather than CPU utilization. Use `process_cpu_usage` for JVM CPU. Process latency gauges are milliseconds; GC pause rate is seconds of pause per second. Registered topology gauges reset to zero when a topology stops, so the count does not prove that topologies are running.

Quick cross-check without Grafana: [http://localhost:8082/health/streams/cpu](http://localhost:8082/health/streams/cpu)

---

## ODE & MEC Deposit

Provisioned file: [`ode-mec-deposit.json`](ode-mec-deposit.json)  
UID: `afm5eiwynxgcga` · Title: **ODE Metrics**

Tracks Kafka produce rates from the ODE, plus MQTT deposit throughput/latency/staleness from MEC Deposit.

### Metrics covered

Scraped from `/actuator/prometheus` on both services (see [`../../prometheus/prometheus.yml`](../../prometheus/prometheus.yml)).

| Source | Prometheus names | Job / labels |
|--------|------------------|--------------|
| ODE (Kafka RSU produce) | `kafka_produced_rsu_messages_total` | `jpo-ode-svcs` · `application=ode` (series labeled by `topic`) |
| MEC Deposit (ETX MQTT) | `mec_deposit_etx_mqtt_publish_rate`, `mec_deposit_etx_mqtt_processing_seconds_sum`, `mec_deposit_etx_mqtt_stale_total` | `jpo-mec-deposit` · `application=mec-deposit` |

Dashboard panels:

1. **ODE Message Counts** — `sum by(topic) (rate(kafka_produced_rsu_messages_total[$__rate_interval]))`
2. **MEC Deposit Publish Rate** — `mec_deposit_etx_mqtt_publish_rate`
3. **MQTT Deposit Latency** — `rate(mec_deposit_etx_mqtt_processing_seconds_sum[$__rate_interval])`
4. **Stale Message Count** — `rate(mec_deposit_etx_mqtt_stale_total[$__rate_interval])`

Default time range: last 3 hours.

### Requirements

1. ODE running with scrape on port **8080** (`ode:8080`)
2. MEC Deposit running with scrape on port **8080** (`mec-deposit:8080`)
3. Prometheus jobs `jpo-ode-svcs` and `jpo-mec-deposit` in [`../../prometheus/prometheus.yml`](../../prometheus/prometheus.yml) (`metrics_path: /actuator/prometheus`, scrape interval 15s)
4. Monitoring stack up from `jpo-utils` (e.g. `docker compose -f docker-compose.yml --profile monitoring_full up`)

Open Grafana → **ODE Metrics**.

### Verify scrape

```bash
curl -sS -o /dev/null -w "%{http_code}\n" http://localhost:8080/actuator/prometheus
curl -sS http://localhost:8080/actuator/prometheus | findstr kafka_produced_rsu
curl -sS http://localhost:8080/actuator/prometheus | findstr mec_deposit_etx_mqtt
```

ODE and MEC Deposit both expose `/actuator/prometheus` on **8080** inside the compose network; from the host, map whichever service port you published and run the matching `findstr` filter.

---

## Node Exporter Full

Provisioned: [`node-exporter-1860_rev37.json`](node-exporter-1860_rev37.json)  
Upstream: https://grafana.com/grafana/dashboards/10242-node-exporter-full/

## Kafka Lag Exporter

Provisioned: [`kafka-lag-exporter.json`](kafka-lag-exporter.json)  
Upstream: https://github.com/seglo/kafka-lag-exporter/blob/master/grafana/Kafka_Lag_Exporter_Dashboard.json

## MongoDB Dashboard

Provisioned: [`mongo-exporter.json`](mongo-exporter.json)  
Upstream: [20867](https://grafana.com/grafana/dashboards/20867-mongodb-dashboard/)
