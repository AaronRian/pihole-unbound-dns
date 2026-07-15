# Chapter 2: Project Objectives and Scope

Chapter 1 explained why Event Horizon exists. This chapter defines exactly what it set out to do — and, just as importantly, what it deliberately did not attempt.

## Objectives

Event Horizon was built with the following objectives:

- Deploy a dedicated, always-on DNS appliance on a Raspberry Pi Zero W.
- Implement network-wide advertisement and tracker blocking using Pi-hole.
- Configure a fully recursive resolver using Unbound, resolving queries directly against the DNS hierarchy (Root → TLD → Authoritative) rather than forwarding to a third-party recursive resolver.
- Enable DNSSEC validation so DNS responses can be cryptographically verified as authentic.
- Reduce DNS latency through local caching.
- Improve DNS privacy by eliminating reliance on public resolvers such as Cloudflare or Google DNS.
- Develop and demonstrate practical Linux system administration skills.
- Produce professional, reproducible documentation suitable for a cybersecurity portfolio.

## Scope

### In Scope

- Raspberry Pi OS installation and initial system configuration.
- System hardening.
- Static IP assignment via DHCP reservation.
- Pi-hole deployment and configuration.
- Unbound deployment and integration with Pi-hole as its sole upstream resolver.
- DNS resolution, DNSSEC, and performance validation.
- Troubleshooting documentation for issues encountered during the build.
- Ongoing operational and maintenance documentation.

### Out of Scope

The following were deliberately excluded from this iteration of the project:

- High Availability (HA) DNS or a redundant secondary resolver.
- Multi-site DNS deployments.
- Enterprise Active Directory integration.
- Cloud-based DNS services.
- DNS load balancing.

These may be explored as future extensions, but they are not required for Event Horizon to be considered complete.

## Success Criteria

Event Horizon is considered successful when:

- The Raspberry Pi operates continuously as the network's DNS server.
- Pi-hole successfully blocks advertisements and trackers network-wide.
- Unbound performs full recursive resolution with DNSSEC validation enabled.
- All client devices on the network resolve DNS queries correctly through this infrastructure.
- The configuration is reproducible from documentation alone, without relying on undocumented manual steps.
- The documentation itself is complete and of a quality suitable for inclusion in a professional cybersecurity portfolio.

## Target Audience

This documentation is written for:

- Cybersecurity enthusiasts
- Linux system administrators
- Network engineers
- Students learning networking and Linux
- Home lab enthusiasts

The next chapter defines the concrete system requirements — hardware, software, and network prerequisites — needed to meet these objectives.
