# Chapter 8: Network Configuration

Chapter 7 established the baseline system and identified two network changes still pending: removing the Wi-Fi configuration in favor of Ethernet-only operation, and confirming the DHCP reservation that gives this device its permanent address. This chapter executes both.

## Disabling Wi-Fi

Raspberry Pi OS (Trixie) manages networking through NetworkManager by default. The Wi-Fi profile configured during imaging was removed, and the radio disabled outright:

```bash
nmcli connection show
sudo nmcli connection delete "<wifi-connection-name>"
sudo nmcli radio wifi off
```

This leaves `wlan0` present but unmanaged and inactive — the interface exists at the hardware level but no longer participates in networking.

## Confirming Ethernet-Only Operation

```bash
ip addr show
nmcli device status
```

Expected result: `eth0` up and holding an address, `wlan0` disconnected. This was confirmed and re-checked after a reboot, since a configuration that only looks correct in the current session isn't good enough for a device meant to run unattended indefinitely.

## DHCP Reservation (Router Side)

The actual address assignment lives on the TP-Link Archer AX10, not on the Pi itself:

1. Logged into the AX10 admin panel at `192.168.0.1`.
2. Located the DHCP reservation / address reservation section.
3. Reserved `192.168.0.20` against the Pi's Ethernet MAC address.
4. Saved and rebooted the Pi to confirm the same address was reissued.

Keeping this configuration on the router rather than hardcoded on the device means the Pi's network identity survives a full OS reinstall without any device-side network config to restore — a deliberate trade-off already established in Chapter 7.

## Verification

```bash
ip addr show eth0
ping -c 4 192.168.0.1
```

Confirmed:

- `eth0` consistently receives `192.168.0.20` across reboots.
- The gateway (`192.168.0.1`) is reachable.
- `wlan0` remains disabled with no active connection.

## DNS Distribution Confirmation

As established in Chapter 5, the AX10's DHCP server hands out `192.168.0.20` as the DNS server to every client on the LAN — no per-device configuration required. With Ethernet-only networking and the reservation both confirmed here, that DNS distribution now rests on a stable, verified foundation rather than an assumption.

---

With addressing and interface configuration locked in, the next chapter installs Pi-hole itself — the first application service running on top of everything built so far.
