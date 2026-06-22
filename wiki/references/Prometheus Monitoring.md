---
type: reference
title: "Prometheus Monitoring"
created: 2026-06-22
updated: 2026-06-22
tags:
  - prometheus
  - monitoring
  - observability
  - metrics
  - docker
status: seed
related:
  - "[[Grafana]]"
  - "[[CrowdSec]]"
  - "[[Docker and Portainer]]"
---

> [!key-insight] Generalized from a homelab observability stack; all IPs/hostnames are placeholders.

Standing up **Prometheus** as the metrics core of an observability stack:
container deploy, a sane `prometheus.yml`, the standard exporters
(node_exporter, cadvisor), file-based service discovery, and alert rules handed
off to Alertmanager. Pairs with [[Grafana]] for dashboards and [[CrowdSec]] for
edge-security metrics.

## Compose service

```yaml
prometheus:
  image: prom/prometheus:v3.4.1
  container_name: prometheus
  restart: unless-stopped
  command:
    - --config.file=/etc/prometheus/prometheus.yml
    - --storage.tsdb.path=/prometheus
    - --storage.tsdb.retention.time=30d
    - --storage.tsdb.retention.size=40GB
    - --web.listen-address=:9090
    - --web.enable-lifecycle          # allows POST /-/reload
  volumes:
    - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
    - ./prometheus/alerts:/etc/prometheus/alerts:ro
    - ./prometheus/targets:/etc/prometheus/targets:ro
    - prometheus-data:/prometheus
  ports:
    - "9090:9090"
```

> [!key-insight]
> Set **both** retention bounds: `retention.time` (e.g. 30d) *and*
> `retention.size` (e.g. 40GB). Time alone lets a noisy target fill the disk;
> size is the backstop that keeps the TSDB from ever running you out of space.

## prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: <stack-name>

alerting:
  alertmanagers:
    - static_configs:
        - targets: ["127.0.0.1:9093"]

rule_files:
  - /etc/prometheus/alerts/*.yml

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ["127.0.0.1:9090"]

  - job_name: node                    # file-based service discovery
    file_sd_configs:
      - files: ["/etc/prometheus/targets/node-*.yml"]

  - job_name: cadvisor
    static_configs:
      - targets: ["127.0.0.1:8085"]

  - job_name: crowdsec                # see [[CrowdSec]]
    static_configs:
      - targets: ["<lapi-ip>:6060"]
        labels: { host: <name>, zone: <zone> }
```

## File-based service discovery

Instead of editing `prometheus.yml` for every new host, drop target files in
`targets/` — Prometheus reloads them live:

```yaml
# /etc/prometheus/targets/node-<group>.yml
- targets:
    - "<host-ip>:9100"
  labels:
    host: <name>
    role: <hypervisor|host|edge>
    zone: <zone>
```

## Exporters

| Exporter | Port | Exposes |
|----------|------|---------|
| node_exporter | 9100 | CPU, memory, disk, filesystem, network, systemd units |
| cadvisor | 8085 | per-container CPU/mem/net/IO |

node_exporter on a host (extra collectors enabled):

```yaml
node-exporter:
  image: prom/node-exporter:v1.9.1
  command:
    - --path.rootfs=/host
    - --collector.systemd
    - --collector.processes
  pid: host
  volumes:
    - /:/host:ro,rslave
```

## Alert rules

```yaml
# /etc/prometheus/alerts/infra.yml
groups:
  - name: infra
    rules:
      - alert: TargetDown
        expr: up == 0
        for: 2m
        labels: { severity: critical }
        annotations:
          summary: "{{ $labels.job }} target {{ $labels.instance }} is down"

      - alert: NodeHighCpu
        expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
        for: 10m
        labels: { severity: warning }

      - alert: NodeDiskFillingUp
        expr: (1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 > 85
        for: 15m
        labels: { severity: warning }
```

Reload after editing rules or config (no restart needed with
`--web.enable-lifecycle`):

```bash
curl -X POST http://127.0.0.1:9090/-/reload
promtool check config prometheus.yml          # validate first
```

## Federation (pull from another Prometheus)

```yaml
  - job_name: federate
    honor_labels: true
    metrics_path: /federate
    params:
      match[]:
        - '{__name__=~"up|node_.*"}'
    static_configs:
      - targets: ["<remote-prom-ip>:9090"]
```

## Related

- [[Grafana]] — dashboards on top of this datasource
- [[CrowdSec]] — security decisions scraped as metrics
- [[Docker and Portainer]] — the container runtime this stack lives on
