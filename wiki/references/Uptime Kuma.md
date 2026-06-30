---
type: reference
title: "Uptime Kuma"
aliases:
  - "Uptime"
  - "Uptime Kuma"
created: 2026-06-28
updated: 2026-06-28
tags:
  - monitoring
  - observability
  - alerting
  - uptime
  - selfhosted
  - reference
status: developing
related:
  - "[[Prometheus Monitoring]]"
  - "[[Grafana]]"
  - "[[Docker]]"
  - "[[Self-Hosted Services]]"
  - "[[Tailscale]]"
  - "[[nftables Firewall]]"
  - "[[OAuth2 Proxy]]"
---

> [!key-insight] Uptime Kuma is the **black-box / synthetic** layer — it probes
> services the way a user would (HTTP, TCP, ping, DNS, cert) and pages you when
> they break. Prometheus + [[Grafana]] are the **white-box** layer (internal
> metrics). Run both: Kuma for "is it up and reachable?", Prometheus for "why,
> and what's trending?". They meet at Alertmanager (see [Alertmanager](#alertmanager--prometheus-integration)).

Self-hosted status-and-alert monitor ([louislam/uptime-kuma](https://github.com/louislam/uptime-kuma)).
Single container, SQLite-backed, with a large catalog of probe types and 90+
notification providers. This note is the catalog: **what you can monitor** and
**how you can be alerted**.

## Deploy

```yaml
# ~/uptime-kuma/docker-compose.yml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    volumes:
      - uptime-kuma:/app/data
    ports:
      - "127.0.0.1:3001:3001"     # front with nginx + TLS; or bind tailnet IP

volumes:
  uptime-kuma:
```

```bash
cd ~/uptime-kuma && docker compose up -d
```

Front it with [[Nginx Reference]] + TLS, or bind it to the host's Tailscale IP
and reach it over the tailnet ([[Tailscale]]). Back up the `/app/data` volume —
it holds the whole config + history (see [[Docker]] → backing up volumes).

## Monitor types — what you can watch

| Type | Checks | Notes |
|------|--------|-------|
| **HTTP(s)** | status code + response time | accepted codes, max redirects, ignore-TLS toggle, auth, custom headers/body |
| **HTTP(s) – Keyword** | response **contains** (or not) a string | catches "200 OK but error page" |
| **HTTP(s) – Json Query** | JSONPath expression vs expected value | API health/field assertions |
| **TCP Port** | raw port reachability | the socket-proxy / DB-port pattern |
| **Ping** | ICMP echo + latency | host liveness |
| **DNS** | resolves a record (A/AAAA/CNAME/MX/TXT/…) | against a chosen resolver |
| **Certificate expiry** | TLS cert days-remaining | (built into HTTPS monitors) early-warning before expiry |
| **Docker Container** | container state via the daemon API | use **docker-socket-proxy**, not the raw socket → [[Docker]] |
| **Push** | dead-man's-switch — *you* POST a heartbeat | cron jobs, backups, batch tasks: alert if the ping **stops** |
| **Database** | Postgres, MySQL/MariaDB, MS SQL Server, MongoDB, Redis | connection/query health |
| **gRPC(s)** | gRPC health endpoint | |
| **MQTT** | broker topic/message | IoT/home-automation buses |
| **RADIUS** | auth check | |
| **SNMP** | OID value vs condition | network gear |
| **Steam / GameDig** | game-server population/status | |
| **Real Browser** | headless Chromium renders the page, keyword in rendered DOM | catches JS-rendered failures (needs the browser image) |
| **Group** | logical parent to nest/organize monitors | drives status-page grouping |

Per-monitor knobs that matter:

- **Heartbeat Interval** — how often to probe (default 60 s).
- **Retries** — consecutive failures before the monitor flips **DOWN** (debounces flaps).
- **Heartbeat Retry Interval** — faster re-check cadence while in the retry/DOWN state.
- **Resend Notification every N beats** — re-page if it's *still* down (escalation).
- **Upside-Down Mode** — invert: success = unreachable (e.g. confirm a port is closed).
- **Accepted status codes / keyword / JSON query** — define "healthy" precisely.

## Notifications — how you get paged

Set up providers under **Settings → Notifications**, then enable per-monitor.
Each monitor can fan out to several providers (e.g. Discord for noise + PagerDuty
for on-call). Highlights from the 90+ available:

| Category | Providers |
|----------|-----------|
| **Chat** | Discord, Slack, Microsoft Teams, Telegram, Matrix, Rocket.Chat, Mattermost, Google Chat, Zoho Cliq, LINE/LINE Notify, Feishu/Lark, DingDing, WeCom |
| **On-call / incident** | **PagerDuty**, Opsgenie, Squadcast, GoAlert, Alerta, Splunk On-Call |
| **Push (self-hosted)** | **ntfy**, **Gotify**, Apprise (meta — 80+ targets) |
| **Push (hosted)** | Pushover, Pushbullet, Pushy, Bark (iOS), Home Assistant |
| **Email** | SMTP (any server) |
| **SMS / voice** | Twilio, ClickSend, AliyunSMS, SMSEagle, Serwersms, PromoSMS, WhatsApp gateways, … |
| **Generic** | **Webhook** (custom JSON → anything), Stackfield, Notifery |
| **Metrics / alerting** | **Prometheus Alertmanager** (see below) |

Two providers do the heavy lifting when something isn't natively supported:

- **Webhook** — POSTs the event JSON to any URL; wire it into your own handler,
  an n8n flow, or a chatops bot.
- **Apprise** — a single provider that proxies to 80+ services via the Apprise
  CLI. If Kuma lacks a native integration, Apprise probably has it.

Notification tips:

- Use **Retries + Resend** so you get a first alert on confirmed-down and a
  reminder if it stays down, without spam on a single flap.
- Route **noise → chat**, **real pages → PagerDuty/Opsgenie**, and keep a
  **self-hosted fallback** (ntfy/Gotify) that doesn't depend on a third party.
- A **Push** monitor + any provider = a dead-man's-switch for backups/cron:
  `curl -fsS https://kuma/api/push/<token>?status=up` at the end of the job;
  Kuma pages if the heartbeat stops arriving.

## Remote-site monitoring (behind NAT / client LAN)

You usually **can't probe into a client LAN** — there's no inbound, just NAT. Flip
the direction: a small agent **inside** the remote network probes the local hosts
and **pushes outbound** to your central Kuma. Outbound 443 is almost always open,
so nothing has to be port-forwarded.

Pattern: one **Push** monitor per remote host (each gets its own push token). A
cron/systemd-timer agent on a box inside the client LAN pings each local IP and,
**only if it's up**, hits that host's push URL. If a host goes down the agent
stops pushing for that token → Kuma flips it DOWN after the retry window → alert.

```bash
#!/usr/bin/env bash
# remote-probe.sh — run on a box inside the client LAN (cron every 60s)
KUMA="https://status.example.com/api/push"

# map: local IP  ->  that monitor's push token
declare -A HOSTS=(
  ["10.50.0.1"]="tok_gateway"
  ["10.50.0.10"]="tok_nas"
  ["10.50.0.20"]="tok_dvr"
)

for ip in "${!HOSTS[@]}"; do
  tok="${HOSTS[$ip]}"
  if ping -c1 -W2 "$ip" >/dev/null 2>&1; then
    curl -fsS "$KUMA/$tok?status=up&msg=OK&ping=$(true)" >/dev/null
  fi
  # down → simply don't push; Kuma's "push timeout" handles the alert
done
```

> [!key-insight] Push monitors are a **dead-man's-switch**: the alert fires on the
> *absence* of a heartbeat, not a failed probe. That's exactly what you want
> behind NAT — the agent only needs **outbound** reachability to your tailnet/
> public Kuma, and you can monitor an arbitrary list of internal IPs without
> opening a single inbound port.

Set each Push monitor's **Heartbeat Interval** a bit longer than the agent's cron
cadence (e.g. agent every 60 s → monitor timeout 90–120 s) so a single missed run
doesn't false-alarm. Push the agent's traffic over [[Tailscale]] if the central
Kuma lives on the tailnet rather than a public URL.

## Status pages

- Multiple **public status pages**, each grouping selected monitors.
- Custom domain, logo, theme, custom CSS, and an embeddable **badge**.
- **Incidents** (post an outage notice) and published **uptime %/history**.
- Good for an external "are we up?" page without exposing the admin UI.

## Maintenance windows

Schedule one-off or recurring **maintenance** windows per monitor/group to
suppress alerts and status-page red during planned work — no false pages during
patching or reboots ([[Linux Server Hardening]] patch cycles, [[Proxmox]] host updates).

## Alertmanager / Prometheus integration

Two distinct paths — they're complementary:

1. **Kuma → Alertmanager (notification provider).** Add a *Prometheus
   Alertmanager* notification pointing at your Alertmanager URL. Kuma pushes its
   up/down events into the same Alertmanager pipeline (routes, silences,
   inhibition, receivers) that your Prometheus alerts use — one on-call funnel.

2. **Prometheus → Kuma metrics (`/metrics`).** Kuma exposes a Prometheus
   endpoint at `/metrics` (per-monitor `monitor_status`, response time, cert
   days-remaining). Scrape it from [[Prometheus Monitoring]], alert in
   **Alertmanager**, and graph in [[Grafana]] alongside white-box metrics.

```yaml
# prometheus.yml — scrape Kuma's synthetic metrics
scrape_configs:
  - job_name: uptime-kuma
    metrics_path: /metrics
    basic_auth: { username: "", password: "<api-key-or-pw>" }
    static_configs:
      - targets: ["100.x.y.z:3001"]
```

> [!key-insight] Pick the path by where you want the **routing logic to live**.
> If Alertmanager is already your single pane for on-call (silences, schedules,
> escalation), prefer **path 1** (Kuma → Alertmanager) so all alerts share one
> routing tree. Use **path 2** when you also want Kuma's probe data trended in
> Grafana and alerted on with PromQL.

## Hardening & access

- Don't expose the admin UI publicly — bind to loopback behind nginx+TLS, or to
  the **tailnet** ([[Tailscale]]); allow only trusted sources at the firewall
  ([[nftables Firewall]]).
- Enable **2FA** on the admin account.
- When monitoring remote Docker hosts, use a **read-only docker-socket-proxy**
  over the tailnet (GET → 200, POST → 403), not the raw socket → [[Docker]].

## Notes

- White-box metrics, dashboards, long-term trends: [[Prometheus Monitoring]] · [[Grafana]].
- What it runs on / alongside: [[Docker]] · [[Self-Hosted Services]].
- Reverse proxy + TLS: [[Nginx Reference]].
