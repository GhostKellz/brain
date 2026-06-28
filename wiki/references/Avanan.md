---
type: reference
title: "Avanan"
created: 2026-06-28
updated: 2026-06-28
tags:
  - security
  - email-security
  - m365
  - checkpoint
  - msp
  - anti-phishing
status: developing
related:
  - "[[Microsoft 365 Services]]"
  - "[[Exchange Online Administration]]"
  - "[[Email DNS Security]]"
  - "[[Huntress]]"
  - "[[CK Technology LLC]]"
---

> [!key-insight] Avanan's whole trick is that it sits **inside** Microsoft 365 via
> API, not in front of it via MX records. A traditional Secure Email Gateway (SEG)
> changes your MX so mail flows through it first; Avanan deploys in minutes by
> connecting to the M365/Google API, scanning *after* Microsoft's own filters but
> *before* (or just after) the inbox — catching what the native filter misses
> without re-architecting mail flow.

**Avanan** is API-based email and collaboration security for cloud suites
(Microsoft 365, Google Workspace) plus Teams, Slack, OneDrive, Google Drive,
Dropbox. Acquired by Check Point (Aug 2021), rebranded **Harmony Email &
Collaboration**, then **Check Point Email Security** — but everyone still says
"Avanan," and the Avanan-branded portal still exists. Tenant identifiers and API
secrets are **secrets** — keep real values in the MSP secret store, never in notes.

## Why API-based vs a gateway (SEG)

| | SEG (MX-based) | Avanan (API-based) |
|--|----------------|--------------------|
| Deployment | Change MX records | Connect via OAuth/API, minutes |
| Position | In front of M365 | After M365's native filter, layered |
| Mail flow | Re-routes all mail | Untouched in Detect mode |
| Visibility | External only | Internal + outbound traffic too |
| Bypass risk | Attackers can detect the SEG | Hidden behind Microsoft |

Because it deploys *after* the native filter, it's an additional smart layer
(AI/ML phishing, malware, account takeover) rather than a replacement for
Microsoft's baseline.

## Protection modes

| Mode | When it scans | Trade-off |
|------|---------------|-----------|
| **Monitor only** | After delivery, log/alert only | No enforcement; no mail-flow change |
| **Detect & Remediate** | After delivery, claws back malicious mail | Brief exposure window before remediation |
| **Prevent (Inline)** | *Before* the inbox ("virtual inline") | Strongest; adds SPF/DKIM + delay considerations |

> [!key-insight] "Virtual inline" is the clever bit: in Prevent mode Avanan uses
> the SaaS provider's own API + mail-flow rules to hold the message for scanning
> (≈10 s – 5 min) *after* Microsoft has scanned it but *before* it lands in the
> mailbox — gateway-level prevention without an MX change. Switching a tenant into
> inline mode can take up to ~an hour to fully propagate.

### SPF / DKIM caveat (inline only)

API-only modes (Monitor / Detect) don't touch the mail flow, so **no SPF change is
needed**. **Inline (Prevent) mode** routes outbound mail through Check Point IPs,
so you must add Check Point's `include:` to the SPF record, and be aware header
rewriting can invalidate DKIM signatures. See [[Email DNS Security]] for SPF/DKIM/
DMARC mechanics.

## Click-Time Protection (URL rewriting)

Avanan rewrites links in email bodies *and attachments* to point at Check Point's
inspection service, so every click is checked at click time — defeating links that
are clean at delivery but weaponized hours/days later.

- Two engines on click: **URL Reputation** (known-bad/references) + **URL
  Emulation** (detonates the page to catch zero-day phishing).
- **Dual-layer with Microsoft Safe Links** — the Check Point link nests inside the
  Safe Links wrapper; both layers run.
- **Persists on forwarding** — protection follows the URL to external recipients.
- **URL hiding** — masks the original destination so users can't copy it to bypass
  the check.

## File / attachment scanning

Uploads and attachments across email + collaboration apps are inspected for
malware and against DLP policy; hits are quarantined/vaulted. Powered by Check
Point **SandBlast**:

- **Threat Emulation** — sandbox detonation for first-seen / zero-day malware.
- **Threat Extraction (CDR)** — strips active content and delivers a sanitized
  file in seconds.
- Also scans **password-protected attachments**, **hidden links inside
  attachments**, and **QR codes** for malicious URLs.

## Account takeover protection

Avanan profiles user behavior inside M365 (login + correspondence patterns) to
flag compromised accounts and auto- or manually-block them before damage spreads —
the overlap with identity threat detection ([[Huntress]] Managed ITDR).

## Threat intelligence

Verdicts come from Check Point **ThreatCloud** (50+ AI engines; HEC's detection
uses 40+ classification models and 300+ signals per email), continuously retrained
on hundreds of millions of emails monthly.

## Smart API (MSP automation)

Avanan exposes a REST **Smart API** for SIEM/SOAR and multi-tenant MSP automation:

1. Authenticate with a **Client ID** + **Client Secret** → receive a **JWT** access
   token (valid ~1 hour).
2. Send it as the `x-av-token` header on every subsequent call.

MSP specifics:

- One app client can be **associated with multiple Avanan tenants** in the same
  region — query many customer portals with one credential.
- **Regional data residency is enforced**: regions are isolated; you can't read one
  region's data via another region's endpoint.

> [!note] Treat the Client ID/Secret like any API credential — store in the MSP
> secret manager, rotate, and scope per region/tenant. Never commit them.

## Where it fits

- The thing it protects: [[Microsoft 365 Services]] / [[Exchange Online Administration]].
- DNS-side authentication it interacts with (esp. inline): [[Email DNS Security]].
- Complements endpoint/identity coverage from [[Huntress]] in the MSP stack at
  [[CK Technology LLC]].

## Related

- [[Microsoft 365 Services]] — the SaaS suite it secures
- [[Email DNS Security]] — SPF/DKIM/DMARC the inline mode touches
- [[Huntress]] — endpoint EDR + Managed ITDR alongside it
