---
type: reference
title: "ScreenConnect"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "ConnectWise Control"
  - "ConnectWise ScreenConnect"
  - "screlay"
tags:
  - screenconnect
  - connectwise
  - remote-support
  - rmm
  - msp
  - tailscale
status: developing
related:
  - "[[Tailscale]]"
  - "[[Azure Key Vault Code Signing]]"
  - "[[Proxmox]]"
  - "[[Managed IT Services]]"
  - "[[Zero Trust Networking]]"
---

# ScreenConnect

**ScreenConnect** (now **ConnectWise Control**) is a self-hosted remote-support and
remote-access tool: a technician console, unattended agents, and a relay that
brokers the session traffic. It's the hands-on remote-control half of the MSP
toolkit (the RMM does monitoring/automation; ScreenConnect does *"let me drive your
machine"*).

> [!key-insight]
> The smart deployment pattern is **split exposure**: the **relay/session
> endpoint** is the only thing on the public internet (agents and clients have to
> reach it from anywhere), while the **web administration portal is kept off the
> internet entirely** and reached only over the [[Tailscale]] tailnet. Public
> surface shrinks to "session traffic only"; the admin login — the juicy target —
> simply isn't routable from outside the mesh.

## How I run it

Self-hosted on a **[[Proxmox]] VM** (PVE-hosted), fronted by that split-exposure
model:

```
                 ┌──────────────────────────────────────────┐
 Agents/clients  │  screlay.<domain>  (PUBLIC)               │
 connect in ───▶ │  relay / session traffic only · :8041     │
   from anywhere │                                            │
                 │  Web admin portal  (NOT public)           │
 Technician ───▶ │  reachable only via Tailscale tailnet     │
 over tailnet    │  http(s) admin UI · :8040                 │
                 └──────────────────────────────────────────┘
```

- **`screlay.<domain>` is public** — it carries the **relay** (default TCP **8041**)
  so unattended agents and ad-hoc support clients can phone home from any network.
  Session traffic only; no admin login lives here.
- **The web portal (default 8040) is bound to the tailnet** — administration,
  technician sign-in, and configuration are only reachable once you're on
  [[Tailscale]]. From the public internet the admin UI doesn't answer.

> [!note]
> ScreenConnect normally serves **both** the web UI (8040) and the relay (8041) from
> one host. The split here is about *exposure*, not separate servers: publish only
> the relay port outward (DNS `screlay`), and reach 8040 over the tailnet (or bind
> it to the tailnet interface / firewall it to tailnet CIDR).

## Architecture

| Piece | Role |
|-------|------|
| **Web server (8040)** | Admin portal + technician console (the "Host" page) + agent download/build. **Tailnet-only here.** |
| **Relay (8041)** | Brokers the real-time session between technician and remote endpoint; the only public service (`screlay`). |
| **Access agent** | Unattended install on managed machines — persistent, calls home to the relay. |
| **Support session** | One-time/ad-hoc client a user runs for attended support. |

Session types: **Access** (unattended, always-on) vs **Support** (attended,
on-demand) vs **Meeting**.

## Why the split exposure matters

- The relay must be public (clients connect inbound from anywhere), but it only
  carries brokered session traffic — not a credentialed admin surface.
- The **admin/host web UI is the high-value attack surface** (login, config, agent
  builder, extension marketplace). Keeping it tailnet-only means an attacker can't
  even reach the login page without already being on the mesh — a clean
  [[Zero Trust Networking|zero-trust]] posture.
- Pairs well with signing your deployed agents — see
  [[Azure Key Vault Code Signing]] for Authenticode-signing the ScreenConnect /
  ConnectWise Control agents so endpoints trust the installer.

## In-session toolbox (the day-to-day wins)

Once connected, the bits that make it fast:

- **Toolbox** — push your own tools/scripts/installers down to the remote machine
  on demand (PsExec-style helpers, sysinternals, fix-it scripts). Run them without
  manually copying files.
- **Copy/paste passthrough** — shared clipboard between the technician and the
  remote session: paste a long command, license key, or password straight in
  instead of retyping. Drag-and-drop / file transfer for moving files both ways.
- **Backstage** — work the machine via a command shell / file system **without
  interrupting the logged-in user's desktop**.
- **Command toolbox** — fire one-liners or saved commands at the session.
- **Blank the remote monitor / lock input** during sensitive work.

> [!key-insight]
> The clipboard passthrough + Toolbox combo is the real time-saver: paste in a
> remediation command, or drop a signed installer down via Toolbox, and you're not
> fumbling with file transfer dialogs mid-call.

## Pros / trade-offs

**Pros**
- Self-hosted → you own the data, the relay, and the update cadence.
- Split exposure shrinks public surface to session traffic only.
- Toolbox + clipboard + Backstage make real support fast.
- Signs/deploys cleanly with [[Azure Key Vault Code Signing|HSM-backed signing]].

**Trade-offs**
- A public relay is still a public service — patch ScreenConnect promptly (it has
  had notable CVEs); the tailnet-only admin UI limits but doesn't eliminate risk.
- Self-hosting means you own patching, TLS, and backups of the VM.
- Web UI behind the tailnet means techs must be on [[Tailscale]] to administer.

## Related

- [[Tailscale]] — the tailnet that fronts the admin portal (admin = mesh-only)
- [[Azure Key Vault Code Signing]] — Authenticode-signing the Control agents
- [[Proxmox]] — the VM host ScreenConnect runs on
- [[Managed IT Services]] — where remote support sits in the MSP stack
- [[Zero Trust Networking]] — the "don't expose the admin plane" principle
