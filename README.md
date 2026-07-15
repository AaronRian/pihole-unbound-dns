# Event Horizon

> A production-grade recursive DNS infrastructure engineered for a home laboratory.

Event Horizon is a self-hosted, network-wide DNS filtering and recursive resolution platform built on a Raspberry Pi Zero W. It combines Pi-hole, Unbound, WireGuard, HTTPS, UFW, and Fail2Ban to create a secure, privacy-focused, and fully documented DNS infrastructure.

The name is deliberate. Just as nothing escapes a black hole's event horizon, no DNS request leaves this network without first being observed, filtered, validated, and controlled.

This is not a weekend tutorial project or a collection of installation notes. It is a documented engineering case study that records the complete lifecycle of designing, deploying, securing, validating, operating, and maintaining a modern recursive DNS platform for a home laboratory environment.

The accompanying handbook explains not only **how** the infrastructure was built, but also **why** each architectural decision was made, including the trade-offs, troubleshooting process, security considerations, and operational practices required to maintain the system over time.
