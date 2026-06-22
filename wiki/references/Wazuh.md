---
type: reference
title: "Wazuh"
created: 2026-06-22
updated: 2026-06-22
tags:
  - wazuh
  - siem
  - xdr
  - security
  - opensearch
status: seed
related:
  - "[[Grafana]]"
  - "[[CrowdSec]]"
  - "[[Endpoint Security Tooling]]"
---

> [!key-insight] Generalized from a homelab security stack; all IPs/hostnames/credentials are placeholders.

**Wazuh** is an open-source SIEM/XDR: agents on endpoints ship log/FIM/rootcheck
events to a manager, which writes alerts into an **OpenSearch** indexer. This
page covers the integration pattern used here — running Wazuh as the alert
store and **consuming it from [[Grafana]]** via the OpenSearch datasource rather
than only using the bundled Wazuh dashboard.

## Components

| Component | Role | Port |
|-----------|------|------|
| Wazuh manager | rules engine, agent enrollment | 1514/1515 |
| Wazuh indexer | OpenSearch — stores alerts | 9200 |
| Wazuh dashboard | bundled OpenSearch Dashboards UI | 5601 |

## Index pattern

Wazuh writes **daily** indices: `wazuh-alerts-4.x-YYYY.MM.DD`. Anything querying
them must use a **date-math** pattern, not a literal wildcard:

```
[wazuh-alerts-4.x-]YYYY.MM.DD      # correct (date-math)
wazuh-alerts-*                     # fails: "index not found" in the Grafana plugin
```

Time field is `timestamp` (alert time), distinct from `@timestamp` (ingest
time). Common fields for panels/queries: `agent.name`, `agent.id`, `rule.level`,
`rule.description`, `rule.groups`, `rule.mitre.technique`, `data.srcip`.

## Consuming Wazuh from Grafana

Add the OpenSearch datasource (full config in [[Grafana]]):

```yaml
- name: Wazuh
  type: grafana-opensearch-datasource
  url: <https://indexer-ip:9200>
  basicAuth: true
  basicAuthUser: <indexer-user>
  secureJsonData:
    basicAuthPassword: <indexer-password>     # secret
  jsonData:
    database: "[wazuh-alerts-4.x-]YYYY.MM.DD"
    interval: Daily
    flavor: opensearch
    version: "2.0.0"
    timeField: "timestamp"
    tlsSkipVerify: true
```

> [!key-insight]
> For an external tool (Grafana) to reach the indexer, the indexer must bind
> **off-loopback** (`network.host: 0.0.0.0`). That flips OpenSearch into
> production mode and enforces bootstrap checks — notably
> `vm.max_map_count >= 262144` on the host. Restrict `:9200` at the firewall to
> just the Grafana host; it's an admin-credentialed endpoint.

## Dashboard panels worth building

- Total alerts / high-severity count (`rule.level >= 8`)
- Authentication failures (`rule.groups: authentication_failed`)
- Active agents (cardinality of `agent.id`)
- Alerts over time, stacked by `rule.level`
- Top rules by `rule.description`, top agents by `agent.name`
- Top MITRE ATT&CK techniques (`rule.mitre.technique`)
- Top source IPs (`data.srcip`)

## Agent enrollment (outline)

```bash
# On an endpoint, register against the manager
/var/ossec/bin/agent-auth -m <manager-ip>
systemctl enable --now wazuh-agent
```

> [!warning]
> The indexer admin password and agent enrollment keys are secrets — keep them
> out of the wiki and any tracked repo.

## Related

- [[Grafana]] — surfacing Wazuh alerts via the OpenSearch datasource
- [[CrowdSec]] — complementary network/edge decisioning
- [[Endpoint Security Tooling]] — where endpoint agents fit overall
