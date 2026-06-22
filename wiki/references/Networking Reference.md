---
type: reference
title: "Networking Reference"
created: 2026-06-21
updated: 2026-06-21
tags:
  - networking
  - subnetting
  - reference
status: developing
related:
  - "[[FortiGate Administration]]"
  - "[[UniFi Controller]]"
  - "[[Nginx Reference]]"
---

> [!key-insight] Generalized from field notes; host/client-specific values are placeholders.

Quick-reference tables: CIDR/subnet masks and the NATO phonetic alphabet.

## CIDR / Subnet Mask Cheat Sheet

| CIDR | Subnet Mask | Wildcard | # IPs | # Usable |
| --- | --- | --- | --- | --- |
| /32 | 255.255.255.255 | 0.0.0.0 | 1 | 1 |
| /31 | 255.255.255.254 | 0.0.0.1 | 2 | 2* |
| /30 | 255.255.255.252 | 0.0.0.3 | 4 | 2 |
| /29 | 255.255.255.248 | 0.0.0.7 | 8 | 6 |
| /28 | 255.255.255.240 | 0.0.0.15 | 16 | 14 |
| /27 | 255.255.255.224 | 0.0.0.31 | 32 | 30 |
| /26 | 255.255.255.192 | 0.0.0.63 | 64 | 62 |
| /25 | 255.255.255.128 | 0.0.0.127 | 128 | 126 |
| /24 | 255.255.255.0 | 0.0.0.255 | 256 | 254 |
| /23 | 255.255.254.0 | 0.0.1.255 | 512 | 510 |
| /22 | 255.255.252.0 | 0.0.3.255 | 1,024 | 1,022 |
| /21 | 255.255.248.0 | 0.0.7.255 | 2,048 | 2,046 |
| /20 | 255.255.240.0 | 0.0.15.255 | 4,096 | 4,094 |
| /19 | 255.255.224.0 | 0.0.31.255 | 8,192 | 8,190 |
| /18 | 255.255.192.0 | 0.0.63.255 | 16,384 | 16,382 |
| /17 | 255.255.128.0 | 0.0.127.255 | 32,768 | 32,766 |
| /16 | 255.255.0.0 | 0.0.255.255 | 65,536 | 65,534 |
| /15 | 255.254.0.0 | 0.1.255.255 | 131,072 | 131,070 |
| /14 | 255.252.0.0 | 0.3.255.255 | 262,144 | 262,142 |
| /13 | 255.248.0.0 | 0.7.255.255 | 524,288 | 524,286 |
| /12 | 255.240.0.0 | 0.15.255.255 | 1,048,576 | 1,048,574 |
| /11 | 255.224.0.0 | 0.31.255.255 | 2,097,152 | 2,097,150 |
| /10 | 255.192.0.0 | 0.63.255.255 | 4,194,304 | 4,194,302 |
| /9 | 255.128.0.0 | 0.127.255.255 | 8,388,608 | 8,388,606 |
| /8 | 255.0.0.0 | 0.255.255.255 | 16,777,216 | 16,777,214 |
| /0 | 0.0.0.0 | 255.255.255.255 | 4,294,967,296 | 4,294,967,294 |

\* /31 carries 2 usable hosts for point-to-point links (RFC 3021).

## NATO Phonetic Alphabet

| | | | |
| --- | --- | --- | --- |
| A Alpha | B Bravo | C Charlie | D Delta |
| E Echo | F Foxtrot | G Golf | H Hotel |
| I India | J Juliet | K Kilo | L Lima |
| M Mike | N November | O Oscar | P Papa |
| Q Quebec | R Romeo | S Sierra | T Tango |
| U Uniform | V Victor | W Whiskey | X X-ray |
| Y Yankee | Z Zulu | | |

## Notes

- Firewall CLI: [[FortiGate Administration]]. Wireless adoption: [[UniFi Controller]].
