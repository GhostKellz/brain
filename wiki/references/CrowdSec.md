---
type: reference
title: "CrowdSec"
created: 2026-06-22
updated: 2026-06-22
tags:
  - crowdsec
  - security
  - ips
  - observability
  - prometheus
status: seed
related:
  - "[[Prometheus Monitoring]]"
  - "[[Grafana]]"
  - "[[Wazuh]]"
---

> [!key-insight] Generalized from a homelab security stack; all IPs/hostnames are placeholders.

**CrowdSec** is a collaborative IPS: agents parse logs, detect attack patterns
via **scenarios**, and the **Local API (LAPI)** issues *decisions* (ban/captcha)
that **bouncers** (firewall, nginx, WAF) enforce. This page covers the deploy
plus the angle used here — **scraping CrowdSec metrics into
[[Prometheus Monitoring]]** so edge-security decisions show up in [[Grafana]].

## Architecture

```
log sources → crowdsec agent (parsers + scenarios) → LAPI (decisions)
                                                        ├→ bouncers enforce (firewall/nginx)
                                                        └→ :6060 /metrics → Prometheus
```

A central LAPI can serve **distributed agents** across hosts (DMZ web, edge
proxies, etc.); each agent exposes its own `:6060` metrics endpoint.

## Install

```bash
# Agent + LAPI
curl -s https://install.crowdsec.net | sudo sh
sudo apt install crowdsec

# A bouncer (firewall enforcement shown)
sudo apt install crowdsec-firewall-bouncer-iptables
```

## Core operations

```bash
cscli metrics                          # local acquisition/parse/scenario stats
cscli decisions list                   # active bans/captchas
cscli decisions add --ip <ip> --duration 4h
cscli decisions delete --ip <ip>
cscli alerts list
cscli bouncers list                    # registered enforcers + last pull
cscli collections list                 # installed parser/scenario bundles
cscli hub update && cscli hub upgrade
```

## Enable the Prometheus endpoint

In `config.yaml`, expose metrics so Prometheus can scrape them — and bind
off-loopback if a remote Prometheus does the scraping:

```yaml
prometheus:
  enabled: true
  level: full
  listen_addr: 0.0.0.0      # 127.0.0.1 if Prometheus is local
  listen_port: 6060
```

Scrape job (see [[Prometheus Monitoring]]):

```yaml
- job_name: crowdsec
  static_configs:
    - targets: ["<lapi-ip>:6060"]
      labels: { host: <name>, zone: <zone> }
    - targets: ["<edge-host-ip>:6060"]
      labels: { host: <edge>, zone: <dmz> }
```

> [!warning]
> Setting `listen_addr: 0.0.0.0` exposes the metrics endpoint on the network —
> restrict `:6060` at the firewall to the Prometheus host only. Keep
> `127.0.0.1` whenever the scraper is on the same box.

## Metrics worth alerting/graphing

| Metric | Meaning |
|--------|---------|
| `cs_active_decisions{reason,origin}` | current bans/captchas |
| `cs_alerts{reason}` | alerts by scenario |
| `cs_lapi_route_requests_total{route}` | LAPI request rate |
| `cs_lapi_bouncer_requests_total{bouncer}` | bouncer decision pulls |
| `cs_papi_last_pull_timestamp` | last community-pull sync |

A simple "scraper down" alert catches a silently-stopped agent:

```yaml
- alert: CrowdsecScraperDown
  expr: up{job="crowdsec"} == 0
  for: 5m
  labels: { severity: warning }
```

> [!key-insight]
> CrowdSec's value is two-layered: **scenarios** decide *what* is an attack, and
> the **community blocklist** (PAPI pull) pre-bans IPs other users have already
> seen attacking. Watch `cs_papi_last_pull_timestamp` — if it goes stale you've
> lost the crowd-sourced half of the protection.

## Related

- [[Prometheus Monitoring]] — scrapes the `:6060` metrics endpoint
- [[Grafana]] — dashboards for decisions/alerts
- [[Wazuh]] — host-side SIEM that complements network-edge decisioning
