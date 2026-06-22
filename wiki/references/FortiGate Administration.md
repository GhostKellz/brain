---
type: reference
title: "FortiGate Administration"
created: 2026-06-21
updated: 2026-06-21
tags:
  - fortigate
  - firewall
  - networking
  - voip
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

Encode the controller IP as hex and serve it via option 43. The hex value is `2b <len> <controller-ip-in-hex>` — use a converter for your controller IP.

```
config system dhcp server
  edit 1
    config options
      edit 1
        set code 43
        set type hex
        set value 2b 1a <controller-ip-hex>
end
```

See [[UniFi Controller]] for the adoption side and the local-DNS alternative.

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

## Notes

- Subnetting helper table: [[Networking Reference]].
