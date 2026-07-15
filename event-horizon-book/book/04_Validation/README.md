# Part 4: Validation

## Overview

Parts 2 and 3 verified each component as it was built — Pi-hole tested on its own, Unbound tested on its own, each security control checked as it was introduced. This part consolidates those checks into a single formal validation pass, confirming the system as a whole meets the objectives and success criteria set out in Chapter 2.

## Core DNS Functionality

| Test | Expected Result | Actual Result | Status |
|---|---|---|---|
| Pi-hole service | Running | Running | ✅ |
| Unbound service | Running | Running | ✅ |
| Port 53 | Listening | Listening | ✅ |
| Port 5335 | Listening | Listening | ✅ |
| Recursive DNS | Success | Success | ✅ |
| DNSSEC trust anchor | Loaded | Loaded | ✅ |
| Pi-hole upstream | `127.0.0.1#5335` | Verified | ✅ |

## Security Controls Verification

Each control introduced in Part 3 was confirmed operational, not just configured:

| Control | Verification Method | Status |
|---|---|---|
| UFW default-deny firewall | `sudo ufw status verbose` | ✅ Active |
| Fail2ban (sshd jail) | `sudo fail2ban-client status sshd` | ✅ Running |
| HTTPS admin interface | Port 443/TCP reachable, HTTP disabled | ✅ Active |
| SSH key authentication | Key-based login confirmed | ✅ Active |
| WireGuard + TP-Link DDNS | Remote tunnel connects via DDNS hostname | ✅ Reachable |

## Network Configuration Verification

| Check | Result |
|---|---|
| DHCP reservation (`192.168.0.20`) | Persists across reboot |
| Ethernet-only operation | `wlan0` disabled, `eth0` sole active interface |
| DNS distribution to clients | Confirmed via router DHCP, no per-device config needed |

## Performance Benchmarking

| Metric | Result | Source |
|---|---|---|
| RAM after first boot (pre-services) | ~426 MiB usable | Chapter 7 (`free -h`) |
| Cold DNS lookup | 300–700 ms | Chapter 11 |
| Warm (cached) DNS lookup | < 20 ms | Chapter 11 |
| Boot time |  1min 49.729s | — |
| RAM after Pi-hole installed | *Pending measurement* | — |
| RAM after Unbound installed |  283Mi | — |
| CPU idle utilization | 2.5% | — |

**Note:** Some metrics remain unmeasured rather than estimated here, to avoid presenting invented figures as fact.



## Validation Summary

Against the success criteria defined in Chapter 2:

- ✅ The Raspberry Pi operates continuously as the network's DNS server.
- ✅ Pi-hole blocks advertisements and trackers network-wide.
- ✅ Unbound performs full recursive resolution with DNSSEC validation enabled.
- ✅ Client devices resolve DNS correctly through this infrastructure.
- ✅ Security controls are configured and independently confirmed operational.
- ⏳ Full performance benchmarking — pending the four measurements above.

The system meets its core functional and security objectives. What remains is closing the small gap in performance documentation, rather than any open functional question.

---

Part 5 moves into Operations: the ongoing maintenance schedule, backup and disaster recovery procedures, and capacity planning that keep this infrastructure reliable over time.
