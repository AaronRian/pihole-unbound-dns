# Appendix F: Future Enhancements

Event Horizon meets every objective set out in Chapter 2, but a few deliberate boundaries were drawn around scope (Chapter 2's "Out of Scope" section) and a few natural next steps surfaced along the way. This appendix records them as a roadmap, not a backlog of unfinished work.

## Reliability

- **High-availability secondary Pi-hole.** A second instance, kept in sync, would remove the single point of failure this deployment currently accepts by design (Chapter 2 explicitly scoped HA out of this iteration).
- **UPS integration.** Protects against corruption from unclean shutdowns during power loss — a real risk for the SD card specifically, given how Part 5 already treats card longevity as a finite resource.

## Privacy & Protocol

- **DNS-over-TLS (DoT) testing.** Would encrypt the recursive lookups Unbound performs upstream of the root hierarchy, closing one of the few remaining points where DNS traffic leaves the device in cleartext.
- **IPv6-only testing.** The current deployment runs IPv4 exclusively (Chapter 9); validating full IPv6 support would extend Event Horizon's coverage to networks or devices that prefer or require it.

## Observability

- **Prometheus and Grafana monitoring.** Would turn Part 5's manual daily/weekly checks into continuously visualized metrics and historical trends, rather than point-in-time command output.
- **Automated configuration backups.** Currently a manual weekly step (Part 5); scripting this as a scheduled job removes the dependency on remembering to run it.

## Additional Security Hardening

- **Move WireGuard off its default port.** Port 51820 is UDP-only and WireGuard doesn't respond to unauthenticated packets, so this isn't a pressing risk — but migrating to a non-default port is a low-cost way to reduce noise from automated internet-wide scans that specifically probe for default VPN ports.

## Documentation & Process

- **CI validation for documentation.** Automated checks (broken links, Markdown linting, diagram syntax validation) would catch documentation drift the same way tests catch code regressions.

## Capacity-Driven Enhancements

Per Part 5's capacity planning, the current hardware comfortably handles this deployment as-is. The following would be reasonable triggers to revisit the hardware or architecture rather than reasons to change anything today:

- Serving hundreds of clients rather than a single household.
- Heavy DNS query logging retained for long periods.
- Additional co-located services (an IDS, for example, alongside Prometheus/Grafana above).
- Large local DNS zones beyond simple ad-blocking and basic local records.

---

This closes out the appendices. What remains is the top-level `README.md` and `SUMMARY.md` — the front door and table of contents tying the entire book together.
