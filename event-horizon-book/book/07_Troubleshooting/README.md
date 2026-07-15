# Part 7: Troubleshooting

## Overview

This part consolidates every significant issue encountered across the entire build into one reference, organized by category rather than by the chronological order things happened to go wrong. Each entry follows the same shape: symptom, root cause, and resolution — the same discipline applied throughout the build itself.

## DNS Service Conflicts

### Port 53 Conflict (Unbound vs. Pi-hole)

**Symptom:** Unbound failed to start immediately after installation.

**Investigation:** The system journal reported:
```
error: can't bind socket: Address already in use for ::1 port 53
fatal error: could not open ports
```

**Root cause:** Pi-hole's FTL service was already bound to port 53. Unbound's default configuration also attempts to bind port 53, and two services can't share the same address-and-port combination.

**Resolution:** Unbound was reconfigured to listen exclusively on `127.0.0.1:5335`, with Pi-hole forwarding to it at that address. This is now the standard architecture for this pairing, not a one-off workaround (see Chapter 10).

### Pi-hole v6 Upstream Syntax

**Symptom:** Pi-hole v6 rejected `127.0.0.1:5335` as a valid upstream resolver entry.

**Root cause:** Pi-hole v6's configuration syntax for a custom upstream with a non-standard port differs from what worked in earlier versions.

**Resolution:** Used the correct syntax, `127.0.0.1#5335` — a `#` separating host and port, not a `:`.

### Deprecated CLI Command

**Symptom:** `pihole restartdns` was unavailable.

**Root cause:** Pi-hole v6 changed its CLI surface compared to v5.

**Resolution:** Used `pihole reloaddns`, the v6 equivalent.

## Network Configuration Issues

### Stale Wi-Fi Profile

**Symptom:** The Raspberry Pi retained an old Wi-Fi configuration after the deployment moved to wired Ethernet.

**Root cause:** Raspberry Pi Imager's headless setup configures Wi-Fi by default, and that profile persists even once it's no longer needed.

**Resolution:** The Wi-Fi profile was explicitly removed and Ethernet-only routing was verified (Chapter 8) — leaving it in place, even unused, is an unnecessary attack surface on a device meant to run wired permanently.

### DHCP Reservation Outside the Pool

**Symptom:** The router rejected the DHCP reservation for the Pi.

**Root cause:** The reserved address fell outside the router's configured DHCP pool range.

**Resolution:** The DHCP pool range was adjusted first, and the reservation was created afterward. A reservation can't exist for an address the router isn't configured to manage in the first place.

## Remote Access

### SSH Host Key Change After Reinstall

**Symptom:** Connecting from Windows produced:
```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

**Root cause:** A Raspberry Pi OS reinstall generated new SSH host keys — the server's cryptographic identity legitimately changed, which this warning is specifically designed to flag.

**Resolution:** The change was verified as self-caused before doing anything else. Only then was the old entry removed from the local `known_hosts` file and the new fingerprint accepted (Part 3).

**Why this matters beyond this one incident:** this warning exists to catch actual man-in-the-middle attacks. Being expected this time doesn't make it safe to dismiss reflexively next time — the verification step is the point, not a formality to click past.

## Installation Warnings

### Backup Permission Warnings During Pi-hole Setup

**Symptom:** The Pi-hole installer produced non-fatal warnings:
```
copy_file(): Failed to open ...
Permission denied
```

**Root cause:** Related to the installer's attempt to back up existing configuration files during setup.

**Resolution:** These warnings did not block installation or affect normal operation, and were documented for future review rather than either ignored outright or treated as a blocking failure.

## Troubleshooting Methodology

A few practices recurred across nearly every incident above, and are worth naming explicitly as the general approach rather than one-off fixes:

- **Diagnose with the tools built for it.** `systemctl status`, `journalctl`, and `ss -tulpn` identified the actual cause in every service-level issue here, rather than guessing from symptoms alone.
- **Test components in isolation.** Querying Unbound directly with `dig ... -p 5335`, separate from Pi-hole, made it possible to confirm exactly which layer of the stack a problem belonged to.
- **Validate before restarting.** `unbound-checkconf` catching a bad config before a service restart turns a potential outage into a non-event.
- **Keep vendor and custom configuration separate.** Isolating custom settings in `/etc/unbound/unbound.conf.d/pi-hole.conf` rather than editing package defaults directly meant upgrades couldn't silently break the deployment.
- **Record issues as they happen, not after the fact.** Every entry in this part exists because it was written down close to when it occurred — troubleshooting notes reconstructed afterward tend to lose exactly the details that make them useful later.

## Lessons Learned

**Technical**
- Separating filtering (Pi-hole) from recursive resolution (Unbound) is a deliberate architectural choice, not just a way to work around a port conflict.
- Port conflicts are diagnosable quickly with systemd status output and socket inspection — no guesswork required.
- Pi-hole v5 and v6 differ meaningfully enough that assuming old syntax or commands still apply is a reliable way to lose time.

**Operational**
- Validating configuration before restarting a service is cheap insurance against avoidable downtime.
- Incremental testing — one component at a time — makes root-causing a failure dramatically faster than testing the whole pipeline at once and guessing where it broke.

**Documentation**
- Recording troubleshooting steps in the moment, rather than reconstructing them later, is what makes a document like this one actually possible to write accurately.

---

With every major issue encountered now documented in one place, the appendices that follow turn this handbook into a complete reference: full configuration files, consolidated diagrams, a command cheat sheet, and possible future enhancements.
