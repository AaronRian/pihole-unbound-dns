# Chapter 10: Installing Unbound

Pi-hole is filtering and serving DNS for the network, but it's still leaning on Cloudflare behind the scenes. This chapter removes that dependency by installing Unbound as a fully local recursive resolver — the piece that lets Event Horizon answer DNS queries itself, directly from the global DNS hierarchy, rather than trusting a third party to do it.

## Why Unbound

Several recursive resolvers were considered before settling on Unbound:

| Resolver | Advantages | Disadvantages |
|---|---|---|
| BIND9 | Mature, feature-rich | Complex configuration |
| Knot Resolver | High performance | Smaller community |
| PowerDNS Recursor | Flexible | Additional complexity |
| Unbound | Lightweight, DNSSEC-aware, actively maintained | None significant for this deployment |

Unbound was selected for its combination of full recursive resolution, native DNSSEC validation, efficient memory usage well suited to the Pi Zero W's limited RAM, strong cache performance, sensible security defaults out of the box, and wide adoption in both enterprise and open-source environments. The full reasoning behind this choice — including trade-offs like added deployment complexity and a slower first lookup — is recorded separately in `ADR-001-Why-Unbound.md`.

## Installation

```bash
sudo apt update
sudo apt install unbound -y
```

```bash
unbound -V
```

Confirmed version 1.22.0 installed from the Raspberry Pi OS package repository.

## Initial Startup Failure

Immediately after installation, the Unbound service failed to start. The system journal reported:

```
error: can't bind socket: Address already in use for ::1 port 53
fatal error: could not open ports
```

**Root cause:** Pi-hole's FTL service was already bound to port 53. Unbound's default configuration also attempts to bind to port 53 by default — two services cannot bind the same address and port combination, so the second to start simply fails.

This wasn't a defect in either piece of software. It's the expected outcome of installing two DNS services side by side without first deciding who owns port 53.

## Architectural Solution

Rather than treating this as a reason to replace Pi-hole, Unbound was isolated behind it instead. Pi-hole keeps its role as the network-facing DNS service, while Unbound operates purely as an internal recursive resolver that only Pi-hole talks to:

```
Client → Pi-hole (port 53) → Unbound (127.0.0.1:5335) → Root DNS Infrastructure
```

## Custom Configuration

A dedicated configuration file was created rather than editing Unbound's vendor-supplied defaults directly:

```
/etc/unbound/unbound.conf.d/pi-hole.conf
```

Keeping custom settings in a separate file isolates them from package upgrades — future Unbound updates won't silently overwrite configuration this deployment depends on. The configuration was built around a specific set of design goals:

- Listen only on `127.0.0.1` — never exposed beyond the loopback interface.
- Use port `5335`, avoiding the port 53 conflict entirely.
- Enable DNSSEC validation.
- Enable QNAME minimization, limiting how much of each query is revealed to upstream servers during recursion.
- Tune cache sizes appropriately for the Pi Zero W's limited RAM.
- Harden against DNS rebinding attacks.
- Minimize unnecessary logging, consistent with the low-noise philosophy established in Chapter 7.

## Configuration Validation

```bash
sudo unbound-checkconf
```

Returned no errors. Validating configuration before restarting a live service is a small habit that avoids a specific failure mode: a bad config causing downtime that then has to be diagnosed under pressure, instead of caught calmly beforehand.

## Successful Startup

```bash
sudo systemctl status unbound
```

Confirmed **active**, **running**, and **enabled at boot**.

```bash
sudo ss -tulpn | grep :5335
```

Confirmed Unbound listening exclusively on `127.0.0.1:5335` — reachable only by local processes on the device itself, with no exposure to the network.

## Functional Testing

```bash
dig openai.com @127.0.0.1 -p 5335
dig github.com @127.0.0.1 -p 5335
```

Both queries returned valid, successful responses (`status: NOERROR`). The first lookup took several hundred milliseconds, since it required a complete recursive walk through the DNS hierarchy with nothing yet cached. Subsequent queries benefited immediately from Unbound's own cache and returned much faster — the expected trade-off of recursive resolution over forwarding, and one worth remembering when interpreting performance numbers in Part 4.

## Lessons Learned

- A port conflict between Pi-hole and Unbound isn't a bug — it's the default outcome of running two DNS services on the same host without first deciding who owns port 53.
- Isolating Unbound on `127.0.0.1:5335` behind Pi-hole is a clean, widely adopted pattern for exactly this pairing, not a workaround specific to this deployment.
- Validating configuration with `unbound-checkconf` before restarting the service catches mistakes before they become downtime.
- Testing Unbound directly with `dig ... -p 5335`, independent of Pi-hole, made it possible to confirm the resolver worked correctly on its own before introducing any integration complexity.

---

Unbound is now running and resolving queries correctly in isolation. The next chapter connects it to Pi-hole as the sole upstream resolver, and verifies the complete recursive DNS pipeline end to end.
