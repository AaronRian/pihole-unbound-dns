# Chapter 3: System Requirements

Chapter 2 defined what Event Horizon needed to achieve. This chapter defines the concrete hardware, software, network, and knowledge prerequisites required to build it — before any hands-on configuration begins in Part 2.

## Hardware Requirements

| Requirement | Minimum | Used in This Deployment |
|---|---|---|
| Single-board computer | ARMv6/ARMv7-capable SBC, 512 MB RAM | Raspberry Pi Zero W (ARM1176JZF-S, 512 MB RAM) |
| Storage | 16 GB microSD (Class 10 recommended) | 64 GB microSD |
| Wired networking | USB Ethernet adapter (via OTG) | USB 2.0 Fast Ethernet Adapter |
| Wireless networking | Optional — used only for initial setup | Onboard 802.11 b/g/n Wi-Fi |
| Power | Stable 5V micro-USB supply | Official Raspberry Pi 5V micro-USB power supply |

**Note:** The Pi Zero W has a single micro-USB port used in OTG mode. A USB Ethernet adapter cannot be plugged in directly — it requires a micro-USB OTG adapter or hub. This is easy to overlook when sourcing hardware and is worth confirming before ordering parts.

Wired Ethernet is treated as a requirement rather than an option for the final deployment. A DNS server benefits from a stable, low-latency link, and Wi-Fi was intentionally removed after initial setup (see Chapter 10, Troubleshooting).

## Software Requirements

| Component | Version | Purpose |
|---|---|---|
| Raspberry Pi OS Lite (32-bit) | Debian 13 "Trixie" | Base operating system |
| Pi-hole | v6.4.3 (FTL v6.7) | DNS filtering and network-wide ad/tracker blocking |
| Unbound | v1.22.0 | Recursive DNS resolution with DNSSEC validation |
| OpenSSH | Latest from Raspberry Pi OS repository | Remote administration |
| Git | Latest from Raspberry Pi OS repository | Version control for configuration and documentation |

A "Lite" (headless, no desktop environment) image is used deliberately — a DNS appliance has no need for a GUI, and the reduced footprint leaves more of the Pi Zero W's limited RAM and CPU available for Pi-hole and Unbound.

> **Note:** The versions above reflect the state of the system at the time of writing, not a live status page. Automatic security updates (Part 3) mean the deployed versions may have since moved forward.

## Network Requirements

- A router capable of DHCP reservations, so the Pi can be reliably reached at a fixed address without manual network configuration on the device itself.
- LAN access to assign that fixed address — `192.168.0.20` in this deployment.
- Administrative access to the router, both to configure the reservation and to point client devices at the Pi as their DNS server.
- A stable local network segment, since every DNS lookup on the network will depend on this device being reachable.

## Knowledge Prerequisites

Readers following this documentation as a guide will get the most out of it with:

- Basic Linux command-line familiarity (package management, systemd services, file editing).
- A working understanding of how DNS resolution works end-to-end.
- Comfort administering a headless device over SSH.
- Basic home networking concepts — DHCP, static addressing, and port forwarding.

None of these need to be expert-level; Part 6 (DNS Theory) exists specifically to fill in the conceptual gaps for readers who want the "why" behind the configuration choices made in Part 2.

---

With requirements established, the next chapter takes a closer look at the hardware itself — why the Raspberry Pi Zero W specifically, and how it holds up under sustained load as a dedicated DNS appliance.
