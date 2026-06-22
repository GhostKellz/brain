---
type: reference
title: "Tailscale Operations"
created: 2026-06-21
updated: 2026-06-21
tags:
  - tailscale
  - vpn
  - networking
status: developing
related:
  - "[[Tailscale]]"
  - "[[Linux Administration]]"
  - "[[Networking Reference]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Tailscale client version management, auto-update, and exit-node / DNS override behavior.

## Version & Updates

```bash
tailscale version
tailscale update                    # latest in current track
tailscale update --yes              # no prompts
tailscale update --version=1.34.0   # pin a version
tailscale update --track=unstable   # switch track
tailscale update --dry-run          # preview only
```

Enable automatic updates (Linux):

```bash
tailscale set --auto-update
```

## Exit Node + Local DNS

To advertise an exit node while letting a local DNS resolver (e.g. a self-hosted Pi-hole) handle DNS rather than Tailscale:

```bash
tailscale up --accept-routes=true --accept-dns=false --advertise-exit-node
```

> [!note] `--accept-dns=false` keeps the node using its local resolver. Adjust based on whether you want Tailscale to push DNS.

## Notes

- General Linux host setup: [[Linux Administration]].
