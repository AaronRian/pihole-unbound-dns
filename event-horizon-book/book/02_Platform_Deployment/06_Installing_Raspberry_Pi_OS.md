# Chapter 6: Installing Raspberry Pi OS

Part 1 established what Event Horizon needed to be and the network it would live in. Part 2 begins the actual build, starting with getting an operating system onto the Raspberry Pi Zero W — configured headlessly from the very first boot, since this device was never meant to have a monitor or keyboard attached.

## Selecting the OS Image

Raspberry Pi OS Lite (32-bit), based on Debian 13 "Trixie," was selected as the base image. The "Lite" variant ships without a desktop environment, which matters on a Pi Zero W: every megabyte of RAM and every cycle of the single-core ARM1176JZF-S not spent on a GUI is a megabyte and a cycle available to Pi-hole and Unbound instead.

## Flashing the SD Card

The 64 GB microSD card was flashed using the official **Raspberry Pi Imager**, which handles both writing the OS image and pre-configuring the device in one step — the second part is what makes a fully headless first boot possible.

## Headless Configuration via Advanced Options

Before writing the image, Raspberry Pi Imager's advanced options (accessible via the gear icon, or `Ctrl+Shift+X`) were used to pre-configure the device so it would be reachable over the network the moment it powered on:

| Setting | Value |
|---|---|
| Hostname | `event-horizon` |
| SSH | Enabled |
| Username | `admin` (a custom, non-default username was used in the real deployment; genericized here for public documentation) |
| Wi-Fi | Configured for initial network connectivity |

Configuring these at flash time — rather than attaching a display and keyboard after the fact — meant the Pi Zero W never needed to be anything other than headless, from the very first boot onward. Wi-Fi was used to get the device onto the network initially; it was later superseded by the wired Ethernet connection once that was confirmed working, which Chapter 8 (Network Configuration) covers in detail.

## First Boot Verification

With the card flashed and inserted, the Pi powered on and was inspected before any packages or services were installed, establishing a clean baseline to compare against later.

| Property | Value |
|---|---|
| Operating System | Raspberry GNU/Linux 13 (Trixie) |
| Kernel Version | 6.18.34+rpt-rpi-v6 |
| CPU Architecture | armv6l |
| Hostname | event-horizon |
| Network Interfaces | eth0, wlan0 |

**Verification checklist:**

- [x] Raspberry Pi booted successfully, first try, no errors.
- [x] SSH login verified from a Windows 11 workstation.
- [x] Operating system and kernel version confirmed.
- [x] Configured hostname (`event-horizon`) applied correctly.
- [x] USB Ethernet adapter detected automatically, no additional drivers required.
- [x] Network connectivity confirmed — an IPv4 address was obtained via DHCP.

At this point the system was in a clean, unmodified state, with no application-specific services installed — ready for the initial Linux configuration covered in the next chapter.

## Establishing Remote Access

With SSH enabled and the hostname set, the device was reachable immediately at `admin@event-horizon.local` (via mDNS) or by its DHCP-assigned IP address, with no further setup required. This initial address came from regular DHCP over Wi-Fi; the fixed `192.168.0.20` reservation on wired Ethernet is established in Chapter 8.

---

The next chapter picks up from this baseline: initial Linux configuration, including system updates and the first round of hardening before Pi-hole and Unbound are ever installed.
