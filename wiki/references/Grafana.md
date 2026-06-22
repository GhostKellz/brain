---
type: reference
title: "Grafana"
created: 2026-06-22
updated: 2026-06-22
tags:
  - grafana
  - observability
  - dashboards
  - provisioning
  - docker
status: seed
related:
  - "[[Prometheus Monitoring]]"
  - "[[Wazuh]]"
  - "[[CrowdSec]]"
---

> [!key-insight] Generalized from a homelab observability stack; all IPs/hostnames/credentials are placeholders.

**Grafana** as the visualization layer over [[Prometheus Monitoring]] (metrics),
Loki (logs), Alertmanager (alerts), and — via the OpenSearch datasource —
[[Wazuh]] alerts. The emphasis here is **provisioning as code**: datasources and
dashboards declared in files, so the whole instance is reproducible.

## Compose service

```yaml
grafana:
  image: grafana/grafana:11.6.1
  container_name: grafana
  restart: unless-stopped
  environment:
    - GF_SECURITY_ADMIN_USER=<admin-user>
    - GF_SECURITY_ADMIN_PASSWORD=<admin-password>   # secret — inject from .env
    - GF_SERVER_ROOT_URL=<https://grafana.example.com>
    - GF_INSTALL_PLUGINS=grafana-opensearch-datasource
  volumes:
    - ./grafana/provisioning:/etc/grafana/provisioning:ro
    - ./grafana/dashboards:/var/lib/grafana/dashboards:ro
    - grafana-data:/var/lib/grafana
  ports:
    - "3000:3000"
```

> [!warning]
> `GF_SECURITY_ADMIN_PASSWORD` is a secret — keep it in `.env`/a secret store,
> never in a committed compose file or wiki. Set `GF_SERVER_ROOT_URL` to the
> public URL or share links and OAuth redirects break.

## Provisioning layout

```
grafana/
├── provisioning/
│   ├── datasources/datasources.yml
│   └── dashboards/dashboards.yml
└── dashboards/            # JSON dashboards, subfolders → Grafana folders
    ├── host/
    ├── containers/
    ├── logs/
    └── security/
```

## Datasources (as code)

```yaml
# provisioning/datasources/datasources.yml
apiVersion: 1
datasources:
  - name: Prometheus
    uid: prometheus
    type: prometheus
    access: proxy
    url: http://127.0.0.1:9090
    isDefault: true
    jsonData:
      timeInterval: 15s

  - name: Loki
    uid: loki
    type: loki
    access: proxy
    url: http://127.0.0.1:3100
    jsonData:
      maxLines: 2000

  - name: Alertmanager
    uid: alertmanager
    type: alertmanager
    access: proxy
    url: http://127.0.0.1:9093
    jsonData:
      implementation: prometheus

  - name: Wazuh
    uid: wazuh
    type: grafana-opensearch-datasource
    access: proxy
    url: <https://indexer-ip:9200>
    basicAuth: true
    basicAuthUser: <indexer-user>
    secureJsonData:
      basicAuthPassword: <indexer-password>     # secret
    jsonData:
      database: "[wazuh-alerts-4.x-]YYYY.MM.DD"  # date-math, NOT a literal *
      interval: Daily
      flavor: opensearch
      version: "2.0.0"
      timeField: "timestamp"
      tlsSkipVerify: true                        # self-signed indexer cert
```

> [!key-insight]
> The Wazuh/OpenSearch datasource needs a **date-math index pattern**
> (`[wazuh-alerts-4.x-]YYYY.MM.DD`), not `wazuh-alerts-*`. A literal wildcard
> fails with "index not found". Also set `timeField: timestamp` (the alert time)
> rather than `@timestamp` (the ingest time).

## Dashboard provisioning

```yaml
# provisioning/dashboards/dashboards.yml
apiVersion: 1
providers:
  - name: <stack-name>
    orgId: 1
    folder: <Folder>
    type: file
    disableDeletion: false
    updateIntervalSeconds: 30
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards
      foldersFromFilesStructure: true   # subdirs become Grafana folders
```

Drop exported dashboard JSON into the matching subfolder; Grafana picks them up
within `updateIntervalSeconds`. `allowUiUpdates: true` lets you tweak in the UI
and re-export back to the file.

## Useful dashboards to import

| Dashboard | Source | Datasource |
|-----------|--------|------------|
| Node Exporter Full | grafana.com #1860 | Prometheus |
| cAdvisor / containers | community | Prometheus |
| Loki logs explorer | built-in Explore | Loki |
| Wazuh alerts | hand-built panels | OpenSearch |
| CrowdSec decisions | see [[CrowdSec]] | Prometheus |

## Related

- [[Prometheus Monitoring]] — the primary metrics datasource
- [[Wazuh]] — security alerts surfaced via the OpenSearch datasource
- [[CrowdSec]] — decision/alert metrics panels
