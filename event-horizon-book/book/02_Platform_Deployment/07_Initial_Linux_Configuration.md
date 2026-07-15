# Chapter 7: Initial Linux Configuration

Chapter 6 ended with a freshly imaged, successfully booted system. This chapter covers everything done to that system before any application service — Pi-hole or Unbound — was installed: verifying the environment, updating the OS, and establishing the security baseline everything else would be built on top of.

## System Verification

Before making any changes, the running system was checked against what the image was expected to produce.

```bash
uname -a
```

Confirmed the kernel version, ARMv6 architecture, Raspberry Pi OS identity, and a successful boot.

```bash
free -h
```

Showed approximately **426 MiB usable RAM**, with swap enabled. This is not a discrepancy against the 512 MB physical RAM listed in Chapter 4 — it reflects the firmware's GPU memory split, which reserves a portion of total RAM before the OS ever sees it. The remaining usable memory is comfortably sufficient for Pi-hole and Unbound.

```bash
uptime
```

Confirmed a stable boot with low load averages.

```bash
systemctl --failed
```

Returned no failed units — a clean slate with nothing already broken before configuration began.

## Operating System Update

```bash
sudo apt update
sudo apt full-upgrade
```

The OS was fully updated before installing anything else, on the principle that starting from outdated or vulnerable packages only compounds risk once application services are layered on top.

## User Account Configuration

The `admin` user, created during imaging (Chapter 6), was confirmed as the sole account used for administration, with `sudo` privileges for anything requiring elevation. Direct root login was intentionally left disabled — every administrative action goes through `sudo` and is attributable to a named account rather than an anonymous root session.

## Network Migration: Wi-Fi to Ethernet

Raspberry Pi Imager's headless setup configures Wi-Fi credentials by default, which meant both Wi-Fi and Ethernet were present and available immediately after first boot. For a permanent DNS appliance, only one of these should remain active:

- Wi-Fi configuration removed.
- Ethernet (`eth0`) designated the sole active network interface.

| Interface | Status |
|---|---|
| eth0 | Active |
| wlan0 | Disabled / unused |

The full network configuration — DHCP reservation setup on the router side, and confirming this interface state persists across reboots — is covered in the next chapter.

## Address Consistency via DHCP Reservation

Rather than hardcoding an IP address inside Linux itself, address consistency is handled entirely by the router: the AX10's DHCP server reserves `192.168.0.20` for this device's MAC address. This keeps IP management centralized in one place (the router) instead of split across router configuration and device-level network config, and avoids the risk of the two drifting out of sync.

## System Time

Accurate system time underpins several other parts of this project directly:

- **DNSSEC validation** — signature validity windows depend on correct time.
- **TLS certificate validation** — for the web dashboard and any HTTPS upstream connections.
- **System logging** — accurate timestamps for troubleshooting and audit purposes.
- **Automatic updates** — scheduling depends on a correct system clock.

Time synchronization is provided by the operating system at boot, and later supplemented by Pi-hole's integrated NTP service for devices on the local network.

## Base Security State

At the end of this phase, before any application service was installed:

| Security Feature | Status |
|---|---|
| SSH Enabled | Yes |
| Root Login | Disabled |
| sudo | Enabled |
| Operating System Updated | Yes |
| Minimal Packages | Yes |
| Ethernet Only | Yes |

## Lessons Learned

- Raspberry Pi Imager can retain Wi-Fi settings that are unnecessary — and worth explicitly removing — in a wired deployment.
- DHCP reservations simplify long-term address management compared to manually configured static addresses on the device itself.
- Verifying system health immediately after installation surfaces problems before they get tangled up with application-level troubleshooting later.
- Updating the OS before installing additional software keeps the dependency and patch history simpler to reason about.

---

With a clean, updated, hardened baseline in place, the next chapter covers the network configuration in full — finalizing the Ethernet-only setup and the DHCP reservation that gives this device its permanent address on the LAN.
