---
type: reference
title: "FortiGate Administration"
created: 2026-06-21
updated: 2026-06-22
tags:
  - fortigate
  - firewall
  - networking
  - voip
  - vpn
  - sdwan
status: developing
related:
  - "[[UniFi Controller]]"
  - "[[Networking Reference]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

FortiGate CLI diagnostics, certificates, VoIP/SIP ALG handling, and DHCP option 43 for UniFi L3 adoption. Replace `<interface>`, `<ip>`, `<mac>` as needed.

## Diagnostics

```
# Resource usage (press M to sort by memory)
diagnose sys top 5 30
diagnose hardware sysinfo memory
get system performance status
get system performance top
diagnose system top

# Crash log
diagnose debug crashlog read

# Continuous ping
exe ping-options repeat-count 10000
exe ping <ip>

# Packet sniffer
diagnose sniffer packet any 'host <ip> and port 53'
```

## Interfaces

```
# Set a MAC address on an interface
config sys int
  edit <interface>
    set macaddr <mac>
  end

# Restart the routing engine
exec router restart
```

## GUI HTTPS Certificate (recover lockout)

```
config system global
  show full | grep admin-server-cert
  set admin-server-cert <cert-name>     # e.g. Fortinet_Factory or an imported cert
end
```

## DHCP Option 43 (UniFi Layer 3 Adoption)

UniFi devices on a different subnet from the controller can't discover it over
Layer 2. Option 43 hands them the controller IP inside their DHCP lease, so they
inform the right endpoint automatically. The value is a vendor sub-option TLV:

```
01 04 <controller-ip-in-hex>
   │  │  └─ the four IP octets, one hex byte each
   │  └──── length = 4 bytes (an IPv4 address)
   └─────── sub-option type 0x01 = "controller IP"
```

Worked example — controller at `192.168.1.2`:

| Octet | 192 | 168 | 1  | 2  |
|-------|-----|-----|----|----|
| Hex   | C0  | A8  | 01 | 02 |

→ option 43 value = `0104C0A80102`.

Set it on the DHCP server that leases the **device** VLAN (run `get system dhcp
server` to find the right server ID):

```
config system dhcp server
  edit <server-id>
    config options
      edit 1
        set code 43
        set type hex
        set value 0104C0A80102        # 0104 + controller IP in hex
      next
    end
  next
end
```

> [!key-insight]
> The leading `01 04` is mandatory — it's the UniFi vendor sub-option header, not
> the option-43 code itself (FortiOS already supplies the `43` via `set code`).
> Sending just the raw IP hex, or prefixing `2b`/`2b06`, is the usual reason
> devices DHCP fine but never adopt.

See [[UniFi Controller]] for the adoption side, the Windows Server DNS A-record
alternative, and a cross-vendor value table.

## VoIP / SIP ALG (disable for SIP trunks)

Many SIP issues are caused by the FortiGate's SIP ALG / session helper mangling traffic.

```
# 1. Disable SIP helper + NAT trace
config system settings
  set sip-helper disable
  set sip-nat-trace disable
  set default-voip-alg-mode kernel-helper-based
end

# 2. Remove the SIP session-helper entry (number varies by model/firmware; find via 'show')
config system session-helper
  show
  delete <sip-entry-number>
end

# 3. Disable RTP processing in the default VoIP profile
config voip profile
  edit default
    config sip
      set rtp disable
    end
  end

# 4. Restart the IPS engine to apply
diag test app ipsmonitor 99
```

## SSL Inspection — Self-Signed Root CA (OpenSSL)

Generate a root CA + service cert for deep SSL inspection, import the root into clients' Trusted Root store, and import the cert chain into the FortiGate (Import -> Certificate). Replace `<companyID>` and SAN values.

```bash
# Root CA
openssl genrsa -aes256 -out <companyID>_root_private.key 2048
openssl req -new -x509 -days 3650 -extensions v3_ca \
  -key <companyID>_root_private.key -out <companyID>_root_ca.crt

# Service key + CSR
openssl genrsa -aes256 -out <companyID>_service1_private.key 2048
openssl req -new -key <companyID>_service1_private.key -out <companyID>_service1.csr

# Sign with the root CA (uses an .ext file for SANs)
openssl x509 -req -in <companyID>_service1.csr \
  -CA <companyID>_root_ca.crt -CAkey <companyID>_root_private.key -CAcreateserial \
  -out <companyID>_service1.crt -days 3650 -sha256 -extfile <companyID>_service1.ext
```

Example `.ext` file:

```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = <fqdn>
IP.1  = <ip>
```

## Interface Configuration

```
config system interface
  edit "<wan>"
    set mode static            # or dhcp / pppoe
    set ip <ip> <netmask>
    set allowaccess ping https ssh
    set alias "Primary-WAN"
  next
end

# VLAN sub-interface on a physical port
config system interface
  edit "<vlan-name>"
    set interface "<parent-if>"
    set vlanid <id>
    set ip <ip> <netmask>
  next
end

# LACP aggregate
config system interface
  edit "<agg-name>"
    set type aggregate
    set member "<port1>" "<port2>"
    set lacp-mode active
  next
end
```

> [!key-insight]
> `set allowaccess` is the management-access whitelist per interface. Leaving
> `ping`/`https`/`ssh` on a WAN interface exposes the box to the internet — keep
> WAN allowaccess minimal (often empty) and manage over the LAN/VPN.

## Routing (Static, Policy, ECMP)

```
# Default route with administrative distance (failover by distance)
config router static
  edit 1
    set dst 0.0.0.0/0
    set gateway <gw>
    set device "<wan>"
    set distance 10
  next
end

# Blackhole / null route
config router static
  edit 2
    set dst <network>/<prefix>
    set blackhole enable
  next
end

# Source/policy-based routing
config router policy
  edit 1
    set input-device "<lan>"
    set src "<src>/<mask>"
    set output-device "<wan2>"
    set gateway <gw2>
  next
end
```

Inspect:

```
get router info routing-table all
diagnose ip route list
```

## Address & Service Objects

```
config firewall address
  edit "<name>"
    set subnet <ip> <netmask>        # type ipmask (default)
  next
  edit "<fqdn-name>"
    set type fqdn
    set fqdn "<fqdn>"
  next
  edit "<geo-name>"
    set type geography
    set country "<CC>"               # ISO country code
  next
end

config firewall addrgrp
  edit "<group>"
    set member "<a>" "<b>"
  next
end

config firewall service custom
  edit "<svc>"
    set tcp-portrange <port|range>
    set udp-portrange <port|range>
  next
end
```

## Firewall Policy

```
config firewall policy
  edit 0                              # 0 = auto-assign next ID
    set name "<name>"
    set srcintf "<in>"
    set dstintf "<out>"
    set srcaddr "<src-obj>"
    set dstaddr "<dst-obj>"
    set action accept                 # or deny
    set schedule "always"
    set service "<svc>"
    set nat enable
    set utm-status enable
    set ssl-ssh-profile "certificate-inspection"
    set av-profile "default"
    set ips-sensor "default"
    set logtraffic all
  next
end

# Reorder (order = match priority) and remove
config firewall policy
  move <id> before <id>
  delete <id>
end
```

> [!key-insight]
> Policies match top-down, first-match-wins. New policies append to the bottom
> (often below a catch-all deny) — always `move` them above the deny, then
> verify hit counts with `diagnose firewall iprope list`.

## VIP (Port-Forward / DNAT) & NAT Pools

```
config firewall vip
  edit "<vip>"
    set extip <public-ip>
    set mappedip <internal-ip>
    set extintf "<wan>"
    set portforward enable
    set extport <port>
    set mappedport <port>
  next
end

# Source-NAT pool (PAT/overload or 1:1)
config firewall ippool
  edit "<pool>"
    set startip <start>
    set endip <end>
    set type overload                 # or one-to-one
  next
end
```

Reference the VIP as the **dstaddr** in a WAN→LAN policy to publish a service.

## IPsec Site-to-Site VPN

```
config vpn ipsec phase1-interface
  edit "<tunnel>"
    set interface "<wan>"
    set remote-gw <peer-ip>
    set psksecret "<psk>"
    set ike-version 2
    set proposal aes256-sha256
    set dpd on-idle
  next
end

config vpn ipsec phase2-interface
  edit "<tunnel>-p2"
    set phase1name "<tunnel>"
    set proposal aes256-sha256
    set pfs enable
    set src-subnet <local>/<mask>
    set dst-subnet <remote>/<mask>
  next
end
```

Then add a route to the remote subnet via the tunnel interface and matching
inbound/outbound policies.

```
diagnose vpn tunnel list
diagnose vpn ike gateway list
diagnose vpn tunnel up <tunnel>
```

## SD-WAN (Members, Health Check, Rules)

```
config system sdwan
  set status enable
  config members
    edit 1
      set interface "<wan1>"
      set gateway <gw1>
    next
    edit 2
      set interface "<wan2>"
      set gateway <gw2>
    next
  end
  config health-check
    edit "ping-check"
      set server "8.8.8.8" "1.1.1.1"
      set protocol ping
      set members 1 2
    next
  end
  config service
    edit 1
      set name "steer"
      set mode sla                    # priority | sla | load-balance
      set dst "all"
      config sla
        edit "ping-check"
          set latency-threshold 100
          set packetloss-threshold 1
        next
      end
      set priority-members 1 2
    next
  end
end
```

```
diagnose sys sdwan health-check
diagnose sys sdwan service
```

## High Availability (Active-Passive)

```
config system ha
  set mode a-p
  set group-name "<cluster>"
  set group-id <id>
  set password "<ha-secret>"
  set priority 200                    # higher = primary
  set hbdev "<ha1>" 50 "<ha2>" 100
  set session-pickup enable
  set monitor "<wan>" "<lan>"
end
```

```
get system ha status
diagnose sys ha checksum cluster      # confirm config sync
execute ha manage 1 admin            # shell into the secondary
```

## Logging & Syslog

```
config log syslogd setting
  set status enable
  set server <syslog-ip>
  set port 514
  set format rfc5424
end
```

> [!key-insight]
> FortiGate emits `key=value` (logfmt) payloads. On the collector side parse
> with `| logfmt`, and classify the firewall by **sender IP** rather than
> per-field labels — its fields are high-cardinality and will blow up a
> log/metrics store if used as labels.

## Backup, Restore & Firmware

```
execute backup config tftp <file> <tftp-ip>
execute restore config tftp <file> <tftp-ip>
execute restore image tftp <fw-file> <tftp-ip>

# Boot partition control (dual firmware)
diagnose sys flash list
execute set-next-reboot secondary
execute reboot

# Last resort
execute factoryreset keepvmlicense
```

## Flow Debug & Session Table

The packet-flow debugger answers "why didn't my policy match?":

```
diagnose debug reset
diagnose debug flow filter addr <ip>
diagnose debug flow filter port <port>
diagnose debug flow trace start 100
diagnose debug enable
# ...reproduce traffic, read the per-stage verdict...
diagnose debug flow trace stop
diagnose debug disable
```

Session lookups:

```
diagnose sys session filter src <ip>
diagnose sys session filter dport <port>
diagnose sys session list
diagnose sys session clear            # clears sessions matching the filter
```

## Automation Stitches

Trigger → action automation (e.g. email/webhook on an IPS event, or auto-block a
source IP). Useful replacement variables: `%%log.srcip%%`, `%%log.logdesc%%`,
`%%log.action%%`.

```
config system automation-trigger
  edit "<trigger>"
    set event-type event-log
    set logid <id>
  next
end
config system automation-action
  edit "<action>"
    set action-type webhook
    set uri "<webhook-url>"
    set http-body "{\"text\": \"FortiGate: %%log.logdesc%% from %%log.srcip%%\"}"
    set method post
  next
end
config system automation-stitch
  edit "<stitch>"
    set trigger "<trigger>"
    config actions
      edit 1
        set action "<action>"
      next
    end
  next
end
```

## Notes

- Subnetting helper table: [[Networking Reference]].
- Forward FortiGate syslog into an observability stack: see [[Prometheus Monitoring]] / [[Grafana]] and [[CrowdSec]] for edge-security correlation.
