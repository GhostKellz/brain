---
type: reference
title: "nftables Firewall"
created: 2026-06-21
updated: 2026-06-21
tags:
  - nftables
  - firewall
  - networking
  - linux
status: seed
related:
  - "[[Zero Trust Networking]]"
  - "[[Tailscale]]"
  - "[[Sysctl Performance Tuning]]"
---

# nftables Firewall

**nftables** is the modern Linux packet-filtering framework, replacing the
iptables family with a single unified tool, one syntax, and atomic rule reloads.
Docker and others still speak iptables via the `iptables-nft` compatibility shim,
so both can coexist.

## A sane default ruleset

```nft
#!/usr/sbin/nft -f
# /etc/nftables.conf
flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        ct state established,related accept     # allow replies
        iif lo accept                            # loopback
        ct state invalid drop                    # bogus packets

        ip protocol icmp accept                  # ping / path MTU
        ip6 nexthdr icmpv6 accept

        tcp dport 22 ct state new limit rate 4/minute accept   # rate-limited SSH
        udp dport 41641 accept                   # Tailscale direct (else DERP)

        log prefix "nft-drop: " flags all counter drop
    }
    chain forward { type filter hook forward priority 0; policy drop;
        ct state established,related accept
    }
    chain output { type filter hook output priority 0; policy accept; }
}
```

> [!key-insight]
> `policy drop` + an explicit allowlist is the safe shape: anything not matched is
> dropped. Always allow `established,related` and `lo` first, or you'll lock
> yourself out.

## Anti-spoofing (reverse-path filter)

```nft
table inet raw {
    chain prerouting {
        type filter hook prerouting priority -300; policy accept;
        fib saddr . iif oif missing drop   # drop packets with no valid return path
    }
}
```

## Service management

```bash
sudo systemctl enable --now nftables
sudo nft -f /etc/nftables.conf      # reload after edits
sudo systemctl reload nftables
```

## Coexisting with Docker and Tailscale

- **Docker** creates and manages its own chains via `iptables-nft`. Don't flush or
  edit them; just leave them alone (`CONFIG_NFT_COMPAT` must be in the kernel).
- **Tailscale** manages its own NAT/MASQUERADE rules and needs CONNMARK kernel
  modules. Allow its UDP port inbound (above) for direct paths, or it falls back
  to relays. See [[Tailscale]].

```bash
sudo nft list ruleset                # everything
sudo nft list ruleset -a             # with handles/counters
sudo conntrack -L                    # live connection table
```

## Performance tuning (high throughput)

Connection-tracking limits live in sysctl — see [[Sysctl Performance Tuning]]:

```ini
net.netfilter.nf_conntrack_max = 262144
net.netfilter.nf_conntrack_buckets = 65536
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 30
```

## Related

- [[Zero Trust Networking]] — per-host defense-in-depth
- [[Tailscale]] — firewall interplay
- [[Sysctl Performance Tuning]] — conntrack sizing
