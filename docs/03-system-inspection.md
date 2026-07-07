# Baseline System Inspection

## Purpose

This document records the initial state of the Raspberry Pi before any applications or services are installed. Establishing a baseline allows future configuration changes to be tracked, simplifies troubleshooting, and provides a reference point for system validation throughout the project.

## System Information

| Property | Value |
|----------|-------|
| Operating System | Raspberry GNU/Linux 13 (Trixie) |
| Kernel Version | 6.18.34+rpt-rpi-v6 |
| CPU Architecture | armv6l |
| Memory | 512 MB |
| Storage | 64 GB microSD Card |
| Network Interfaces | eth0, wlan0 |

---

## Inspection Details

| Property | Value |
|----------|-------|
| Inspection Date | 2026-07-08 |
| Hostname | event-horizon |
| Inspector | Aaron Rian |

## Initial Observations

- Raspberry Pi OS completed the first boot successfully without errors.
- SSH connectivity was successfully established from a Windows 11 workstation.
- The USB Ethernet adapter was detected automatically without requiring additional drivers.
- The Raspberry Pi obtained an IPv4 address from the DHCP server.
- The configured hostname (`event-horizon`) was applied successfully.
- The operating system is in a clean state with no application-specific services installed.
- The system is ready for initial hardening and software installation.

## Verification

The following checks were completed successfully:

- Raspberry Pi booted successfully.
- SSH login verified.
- Operating system identified.
- Kernel version verified.
- Hostname verified.
- Network connectivity confirmed.

## Next Steps

The next phase of the project includes:

- System update
- Initial Linux hardening
- Static IP configuration
- Pi-hole installation
- Unbound installation
---
