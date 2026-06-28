---
type: concept
title: "Reverse Shells"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "Reverse Shell"
  - "Bind Shell"
  - "Connect-Back Shell"
tags:
  - security
  - networking
  - pentest
  - blue-team
  - shells
status: seed
related:
  - "[[Linux Server Hardening]]"
  - "[[nftables Firewall]]"
  - "[[Endpoint Security Tooling]]"
  - "[[Zero Trust Networking]]"
  - "[[Networking Reference]]"
---

# Reverse Shells

A **reverse shell** is a command shell where the **target initiates the connection
back to you**, instead of you connecting in. You run a **listener**; the target
runs a small payload that **dials out** to your listener and wires its
stdin/stdout/stderr to the socket — giving you interactive control. It's a
foundational technique in both offensive security (authorized pentests, CTFs) and
legitimate remote administration of your own machines.

> [!key-insight]
> The whole trick is **direction**. Inbound connections are blocked by NAT and
> firewalls by default; **outbound is usually allowed** (especially 443/53). A
> reverse shell flips the call so the victim/host reaches *out* — which is why it
> sails through firewalls a bind shell can't. That same property is why defenders
> hunt **egress**, not ingress, to catch it.

> [!warning]
> This page is for **authorized** use only: your own systems, lab environments, CTF
> targets, and engagements with written permission. Running a shell on a system you
> don't own/control is illegal. The defensive sections exist so you can *detect and
> stop* unauthorized use.

## Bind shell vs reverse shell

| | Bind shell | Reverse shell |
|--|-----------|---------------|
| Who listens | the **target** opens a port | **you** open a port |
| Who connects | you connect **in** to the target | the target connects **out** to you |
| Beats NAT/firewall? | no — inbound usually blocked | **yes** — outbound usually allowed |
| Typical use | target is directly reachable | target is behind NAT/firewall |

## How it works (the mechanics)

1. **You start a listener** on a host the target can reach (your box, or a VPS):
   ```bash
   nc -lvnp 4444          # listen on TCP 4444
   ```
2. **The target connects back** and redirects its shell's I/O to the socket. The
   classic shape (conceptually): open a TCP socket to `you:4444`, then attach a
   shell's stdin/stdout/stderr to it. On modern Linux a `/dev/tcp` one-liner or a
   scripting-language socket does this; tooling like the **pentest-monkey cheat
   sheet**, **revshells.com**, **socat**, **msfvenom**, or a C2 agent generate the
   exact payload for the target's available interpreter.
3. **You get an interactive shell** over that outbound socket.

> [!note]
> A raw `nc`/`/dev/tcp` shell is "dumb" — no job control, no tab-complete, Ctrl-C
> kills it. Operators **upgrade to a PTY** (e.g. `python3 -c 'import pty;
> pty.spawn("/bin/bash")'`, then background and `stty raw -echo; fg`) to get a fully
> interactive terminal. `socat` can negotiate a proper PTY directly.

### Why it evades naive controls

- **Egress-permissive firewalls** let outbound 80/443/53 through, so the connect-back
  succeeds where an inbound bind shell would be dropped.
- **Encryption** (`socat` OpenSSL, TLS C2 over 443) hides the payload from plaintext
  inspection and blends with normal HTTPS.
- **Common ports** (443, 53) make the flow look ordinary in netflow.

## How threat actors use them

- **Post-exploitation foothold:** after RCE (web shell, deserialization, phishing
  macro), the first move is often to spawn a reverse shell for interactive access.
- **Living off the land:** reuse what's already installed (`bash`, `python`,
  `powershell`, `nc`) to avoid dropping detectable binaries.
- **C2 staging:** the dumb shell is an upgrade path to a full command-and-control
  implant (Cobalt Strike, Sliver, Metasploit) with encryption, persistence, and
  pivoting.
- **Egress over 443/DNS:** chosen specifically to look like normal traffic and slip
  past perimeter rules.

## Legitimate / internal use cases

The same technique, used on systems you control:

- **Reaching boxes behind NAT/CGNAT** without port-forwarding — a host on a remote
  network dials back to a jump box you own (a poor-man's tunnel; a tidier version is
  an SSH reverse tunnel `ssh -R` or a [[Mesh VPN]] like [[Tailscale]]).
- **Lab / CTF practice:** the bread-and-butter of HackTheBox/TryHackMe and OSCP-style
  training — spin them up freely in **your own lab**.
- **Authorized pentest engagements:** demonstrating impact and establishing a
  foothold within scope and rules of engagement.
- **Break-glass admin** on a box where inbound is firewalled but outbound works.

> [!note]
> For routine remote admin, prefer **SSH** (`ssh -R` reverse tunnels) or a
> [[Mesh VPN]] over an ad-hoc `nc` shell: you get auth, encryption, and integrity
> for free. Reverse shells shine when you *can't* pre-install those (recovery,
> CTFs, pentests).

## Detection & defense (blue team)

Assume attackers will try; make it loud and hard:

- **Egress filtering** — default-deny outbound; only allow known destinations/ports.
  This is the single biggest blocker. See [[nftables Firewall]] and
  [[Zero Trust Networking]].
- **Hunt outbound anomalies** — servers initiating connections to random external
  IPs on 443/53, long-lived flows, beaconing intervals. Netflow + IDS.
- **Process/EDR telemetry** — a shell (`bash`, `sh`, `powershell`) with a network
  socket as its parent/child, or interpreters spawning shells
  (`python -c 'pty.spawn'`), is high-signal. See [[Endpoint Security Tooling]].
- **Host hardening** — drop unneeded interpreters/`nc` where feasible, restrict who
  can run them, app-allowlisting; general baseline in [[Linux Server Hardening]].
- **DNS monitoring** — catch DNS-tunneled C2 (high entropy, high volume to one zone).

> [!key-insight]
> You don't stop reverse shells at the exploit — you stop them at **egress**. A host
> that can't make arbitrary outbound connections can be popped but can't phone home.
> Default-deny egress turns a full compromise into a contained one.

## Related

- [[nftables Firewall]] — egress filtering, the primary mitigation
- [[Zero Trust Networking]] — default-deny posture that breaks connect-back
- [[Endpoint Security Tooling]] — EDR detection of shell+socket behavior
- [[Linux Server Hardening]] — reduce the interpreters/tooling available to abuse
- [[Networking Reference]] — NAT/firewall direction, why outbound is the soft spot
