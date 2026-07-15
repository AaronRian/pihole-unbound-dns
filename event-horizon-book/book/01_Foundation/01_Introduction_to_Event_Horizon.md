# Chapter 1: Introduction to Event Horizon

## What is Event Horizon?

Event Horizon is a self-hosted, network-wide ad-blocking and recursive DNS system, built from scratch on a Raspberry Pi Zero W, paired with a WireGuard VPN gateway for secure remote access to the entire home network. The name is deliberate: just as nothing escapes a black hole's event horizon, nothing — trackers, ad requests, or third-party DNS queries — escapes this network without being seen, filtered, and controlled first.

This isn't a weekend tutorial project. It's a working piece of home network infrastructure, built layer by layer, and this documentation is the record of how and why it was built the way it was.

## Why This Project Exists

Three problems motivated Event Horizon, and each shaped a specific design decision covered later in this book:

**Intrusive advertising.** Ads on modern networks aren't limited to browsers — they show up in apps, smart TVs, and IoT devices that don't offer an opt-out. A network-wide blocker solves this at the DNS layer, once, for every device on the network, instead of per-app or per-browser.

**Privacy.** Most home networks route every DNS query through an ISP resolver or a "free" third-party DNS provider by default. Both are in the business of knowing what domains you look up. Running a fully recursive resolver removes that middleman entirely — queries go straight to the authoritative source, with no third party positioned to log or monetize them.

**Data ownership.** Public DNS providers explicitly reserve the right to log and, in some cases, sell aggregated query data. Self-hosting the resolver was a deliberate refusal to accept that trade-off.

## Why the Network is Deliberately Complicated

A dual-router, dual-NAT topology is not the easiest way to run a home network — most guides actively tell you to avoid it. Here, it's intentional.

Rather than learning networking theory from a PDF, this project treats the home network itself as the lab. Every problem a dual-NAT setup introduces — double port forwarding, NAT traversal for the WireGuard tunnel, routing between two private subnets, DNS resolution across NAT boundaries — is a forced, hands-on lesson that a single-router setup would never surface. The complexity isn't a byproduct; it's the curriculum.

Later chapters (particularly Network Architecture and Troubleshooting) walk through the specific problems this topology created and how each was solved, since those problems — and their solutions — are where most of the actual learning happened.

## What This Documentation Represents

Beyond being a build log, this documentation is meant to stand as evidence of applied skill: DNS internals, Linux system administration, network security, VPN configuration, and the ability to debug a non-trivial multi-layer network from first principles. It's written to be legible to someone evaluating those skills directly — including as part of a case for a cybersecurity role — not just to future-me troubleshooting at 2 a.m.

## How This Book is Organized

Event Horizon's documentation follows the build in order:

- **Part 1 — Foundation:** project scope, requirements, hardware, and the network architecture this book keeps referring back to
- **Part 2 — Platform Deployment:** OS setup, Pi-hole, Unbound, and how the two are integrated into one resolution pipeline
- **Part 3 — Security:** hardening decisions and the WireGuard VPN gateway
- **Part 4 — Validation:** how each component was tested and confirmed working
- **Part 5 — Operations:** day-to-day maintenance and monitoring
- **Part 6 — DNS Theory:** the underlying concepts, for readers who want the "why" behind the config
- **Part 7 — Troubleshooting:** real problems hit during the build, and how they were diagnosed and fixed

The next chapter defines the project's exact objectives and scope — what Event Horizon is meant to do, and just as importantly, what it isn't.
