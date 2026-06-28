---
type: reference
title: "Discord"
aliases:
  - "Discord Webhooks"
  - "Discord Bots"
created: 2026-06-28
updated: 2026-06-28
tags:
  - discord
  - chatops
  - webhooks
  - bots
  - alerting
  - reference
status: developing
related:
  - "[[Uptime Kuma]]"
  - "[[Prometheus Monitoring]]"
  - "[[Grafana]]"
  - "[[Nginx Reference]]"
  - "[[ghostkellz.sh]]"
  - "[[Self-Hosted Services]]"
---

> [!key-insight] For an IT shop Discord is a free **chatops + alerting hub**: a
> **webhook** is the 30-second path (one URL, POST JSON, done) for *outbound*
> notifications; a **bot** is the heavier path for *interactive* control (slash
> commands, buttons, reading messages, automation). Reach for a webhook first;
> build a bot only when you need two-way.

What an IT professional actually does with Discord: pipe monitoring/CI/Git alerts
into channels, run chatops, and hand out a clean vanity invite. This note covers
webhooks, bots, the common integrations, and the security gotchas.

## Webhooks (outbound notifications)

Create one per channel: **Channel → Edit → Integrations → Webhooks → New
Webhook**, copy the URL. Anyone with that URL can post — treat it as a secret.

```bash
# Minimal message
curl -fsS -H "Content-Type: application/json" \
  -d '{"content":"deploy finished on web01"}' \
  "$DISCORD_WEBHOOK_URL"
```

### Rich embeds

```bash
curl -fsS -H "Content-Type: application/json" -d '{
  "username": "alertbot",
  "embeds": [{
    "title": "Service DOWN",
    "description": "nginx on web01 stopped responding",
    "color": 15158332,
    "fields": [
      {"name": "Host",   "value": "web01", "inline": true},
      {"name": "Check",  "value": "HTTP 200", "inline": true}
    ],
    "footer": {"text": "Uptime Kuma"}
  }]
}' "$DISCORD_WEBHOOK_URL"
```

- `color` is a **decimal** RGB int (red `15158332` = `0xE74C3C`, green `3066993`).
- Up to 10 embeds per message; `content` ≤ 2000 chars.
- **Rate limits:** ~5 requests/sec per webhook; on `429` honour the
  `retry_after` in the JSON body. Batch noisy sources.

### Common producers (no bot needed)

| Source | How |
|--------|-----|
| [[Uptime Kuma]] | built-in **Discord** notifier — paste the webhook URL |
| [[Grafana]] | contact point type **Discord** |
| **Prometheus Alertmanager** | via a small webhook-bridge receiver, or route through Kuma/Grafana → [[Prometheus Monitoring]] |
| **GitHub / GitLab** | repo webhook → append `/github` or `/slack` to the Discord webhook URL for native formatting |
| CI / cron / scripts | the `curl` above |

> [!key-insight] Discord webhooks accept GitHub's and Slack's payload formats:
> append **`/github`** or **`/slack`** to the webhook URL and point a GitHub repo
> webhook or any Slack-compatible integration straight at it — no translation layer.

## Bots (interactive / two-way)

Build a bot when you need slash commands, buttons, reading channels, role
management, or scheduled automation.

1. **developer portal** → New Application → **Bot** → copy the **token** (secret).
2. Enable required **Privileged Intents** (e.g. *Message Content* only if you read
   message text — otherwise leave off for least privilege).
3. **OAuth2 → URL Generator** → scopes `bot` + `applications.commands`, tick only
   the permissions the bot needs, use the generated URL to invite it to a server.

```python
# discord.py — a slash command + a status push
import discord
from discord import app_commands

intents = discord.Intents.default()          # no message-content intent needed
client = discord.Client(intents=intents)
tree = app_commands.CommandTree(client)

@tree.command(description="Ping the bot")
async def ping(interaction: discord.Interaction):
    await interaction.response.send_message("pong", ephemeral=True)

@client.event
async def on_ready():
    await tree.sync()
    print(f"logged in as {client.user}")

client.run("<BOT_TOKEN>")   # load from env/secret, never hardcode
```

Libraries: **discord.py** (Python), **discord.js** (Node), **serenity**/**twilight**
(Rust). Host the bot as a container ([[Docker]]) or a hardened systemd service
([[systemd]]) — it only needs **outbound** to Discord's gateway, so it runs fine
behind NAT with no inbound ports.

- **Gateway bots** hold a WebSocket (real-time events) — most bots.
- **HTTP interactions** (no gateway) — Discord POSTs to your endpoint; you must
  verify the **Ed25519 signature** header and reply within 3 s.

## Chatops use-cases (IT pro)

- **Single alert pane** — Kuma + Grafana + Alertmanager + CI all into `#alerts`.
- **Deploy/Git feed** — push/merge/release events into `#dev` via the `/github` URL.
- **Chatops control** — a bot slash command kicks a job/restart (gate behind a
  role; log every action). Keep the bot's host creds least-privilege.
- **On-call escalation** — role-ping (`<@&roleid>`) on `DOWN`; pair with a
  real pager (PagerDuty/Opsgenie via [[Uptime Kuma]]) for anything critical.
- **Backups/cron heartbeats** — a job posts success; silence = investigate (or use
  a Kuma **Push** monitor that also notifies Discord).

## Vanity invite redirect (nginx)

Hand out a memorable URL instead of a raw `discord.gg` code. `discord.ghostkellz.sh`
→ the live invite, via a one-line nginx redirect ([[Nginx Reference]],
[[ghostkellz.sh]]):

```nginx
server {
    listen 80;
    server_name discord.ghostkellz.sh;
    return 301 https://$host$request_uri;      # force HTTPS
}

server {
    listen 443 ssl http2;
    server_name discord.ghostkellz.sh;

    ssl_certificate     /etc/nginx/certs/ghostkellz.sh/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/ghostkellz.sh/privkey.pem;

    return 302 https://discord.gg/AKKmRWX38b;   # 302 so you can rotate the invite
}
```

> [!note] Use **302** (temporary) on the invite hop so you can swap the
> `discord.gg` code later without clients caching the old one. Keep an
> auto-renewing cert via [[acme.sh - DNS-01 Certificates]] /
> [[Let's Encrypt - Certbot|Let's Encrypt / Certbot]].

## Security

- **Webhook URLs and bot tokens are secrets.** Anyone with a webhook URL can post
  as it; anyone with a token controls the bot. Keep them in env/secret stores, not
  in repos; rotate on leak (regenerate in the portal).
- **Least-privilege bots** — grant only the OAuth scopes/permissions needed; leave
  *Message Content* intent off unless you truly parse message text.
- **Verify interaction signatures** for HTTP-interaction bots (Ed25519) — reject
  unsigned/replayed requests.
- **Rate-limit / dedupe** noisy alert sources before they hit a webhook to avoid
  `429`s and channel spam.

## Notes

- Alert producers: [[Uptime Kuma]] · [[Grafana]] · [[Prometheus Monitoring]].
- Reverse proxy + TLS for the invite host: [[Nginx Reference]].
- Owner's public properties: [[ghostkellz.sh]].
