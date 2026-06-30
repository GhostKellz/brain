---
type: domain
title: "Security"
created: 2026-06-21
updated: 2026-06-22
tags:
  - domain
  - security
status: developing
related:
  - "[[Networking]]"
  - "[[Cloud and Microsoft 365]]"
---

# Security

Endpoint, identity, email, and network security — plus cryptographic identity.

## Endpoint & identity
[[Endpoint Security Tooling]] · [[Huntress]] (managed EDR/ITDR/SIEM/SOC) ·
[[Network and Security Services]] · [[Zero Trust Networking]] ·
[[OAuth2 Proxy]] (Microsoft SSO gate in front of self-hosted apps)

## Email & messaging
[[Email DNS Security]] (SPF/DKIM/DMARC) · [[Avanan]] (API email security) ·
[[Exchange Online Administration]]

## Network security
[[nftables Firewall]] · [[FortiGate Administration]] · [[CrowdSec]] (collaborative IPS)

## Offensive / techniques
[[Reverse Shells]] (how they work, threat-actor + authorized use, egress defense)

## SIEM & monitoring
[[Wazuh]] (SIEM/XDR) · [[CrowdSec]] (edge decisions) ·
[[Prometheus Monitoring]] + [[Grafana]] (metrics/dashboards)

## Code signing
[[Azure Key Vault Code Signing]] (HSM-backed signing certs)

## Cryptographic identity
[[PGP and GPG]] · [[Web Key Directory]] · [[GPG Key Verification Workflow]] ·
[[zcrypto]] (Zig crypto library)

> [!gap]
> The generalized patterns for the security/observability stack are documented in
> the pages above. Host-, IP-, and client-specific values from the private
> deployment are intentionally excluded — they live in the private tier.
