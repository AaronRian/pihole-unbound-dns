# Chapter 4: Hardware Overview

Chapter 3 listed what the hardware needed to be capable of. This chapter looks at the specific board selected, why it was chosen over larger alternatives, and how it holds up as a dedicated, always-on DNS appliance.

## Selected Platform

| Component | Specification |
|---|---|
| Board | Raspberry Pi Zero W |
| CPU | ARM1176JZF-S (single-core) |
| Architecture | ARMv6 |
| RAM | 512 MB |
| Storage | 64 GB microSD card |
| Wired Networking | USB 2.0 Fast Ethernet Adapter (via micro-USB OTG) |
| Wireless Networking | Onboard 802.11 b/g/n Wi-Fi (used for initial setup only) |
| Power | Official 5V micro-USB power supply |

## Why the Raspberry Pi Zero W

A DNS resolver and filtering service is not a resource-intensive workload once it's up and running — the bottleneck is almost always the network, not the CPU. That makes the Pi Zero W's modest specs a deliberate fit rather than a compromise:

- **Cost and power draw.** The Zero W draws a fraction of the power of a full-size Raspberry Pi, which matters for a device meant to run continuously, 24/7, indefinitely.
- **Right-sized for the workload.** Pi-hole and Unbound are both lightweight services. Even under continuous operation, CPU utilization stays low and memory usage remains comfortably within the Zero W's 512 MB, with headroom left for logging and routine maintenance tasks.
- **Small footprint.** No case fan, no significant heat management, and no meaningful desk or rack space required — useful for a homelab where the DNS appliance is one of several always-on devices.
- **Headless by design.** Paired with Raspberry Pi OS Lite, the Zero W runs with no desktop environment and no wasted resources on a GUI that a DNS appliance will never need.

A larger board (such as a Raspberry Pi 4) would have added capability that this workload doesn't call for, at a higher cost and higher continuous power draw for no practical benefit here.

## Physical Setup Considerations

- **OTG adapter required.** The Zero W has a single micro-USB port used in OTG mode — a standard USB Ethernet adapter cannot be connected directly and requires a micro-USB OTG adapter or hub in between.
- **Wired over wireless.** Onboard Wi-Fi is present but only used during initial setup. The production deployment relies on the wired Ethernet adapter for a stable, low-latency connection, since every DNS lookup on the network depends on this link.
- **Storage.** A 64 GB microSD card is well beyond what the OS, Pi-hole, and Unbound require on their own — the extra headroom is allocated for query logs and long-term log retention rather than the base install.
- **Power stability.** An official 5V micro-USB supply is used specifically to avoid the under-voltage issues that generic phone chargers are known to cause on Raspberry Pi boards, which can quietly cause instability rather than an obvious failure.

## Resource Utilization

Actual measured performance figures — boot time, RAM usage at each stage, and cold vs. warm DNS lookup latency — are recorded in Part 4 (Validation) once benchmarking is complete, rather than estimated here.

---

With the hardware established, the next chapter moves to the network itself: how the Raspberry Pi fits into the dual-NAT topology, and the addressing and routing decisions that make the rest of the build possible.
