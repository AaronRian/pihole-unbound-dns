# Part 3: Security Hardening & Operational Resilience

## Overview

Deploying a working DNS server is only half the job. DNS infrastructure is an attractive target — it's always available, it serves every client on the network, and once other services come to depend on it, it becomes critical infrastructure whether or not it was ever treated that way.

Event Horizon was hardened using a layered security model rather than any single control. The guiding principle throughout this phase was simple:

> Every exposed service should be justified, authenticated, encrypted, monitored, and recoverable.

## Security Objectives

- Encrypt all administrative web traffic.
- Eliminate password-based SSH authentication where practical.
- Prevent brute-force attacks against exposed services.
- Restrict unnecessary network exposure.
- Protect against unauthorized configuration changes.
- Ensure recovery is possible after hardware failure.
- Maintain secure remote administration from outside the LAN.
- Minimize the overall attack surface.

## Threat Model

The deployment targets realistic threats facing an internet-connected homelab — not nation-state adversaries, but the kind of automated and opportunistic attacks any always-on device actually receives.

| Threat | Mitigation |
|---|---|
| Password guessing | SSH keys, Fail2ban |
| Credential interception | HTTPS |
| DNS spoofing | DNSSEC |
| Service exposure | UFW |
| Configuration loss | Automated backups |
| Unauthorized remote access | WireGuard VPN |
| Outdated software | Automatic security updates |

This model explicitly does not attempt to defend against physical compromise of the Raspberry Pi itself, or compromise of the local network it sits on — those are different threat models with different mitigations, outside this project's scope.

## HTTPS Administration

The Pi-hole admin interface was initially reachable over plain HTTP at `http://192.168.0.20/admin` — acceptable for the verification steps in Chapter 9, but not for ongoing use. Unencrypted management traffic risks exposing credentials to interception, even on a trusted LAN.

The interface was migrated to HTTPS, with the built-in web server listening on port 443/TCP for secure administration. Routine HTTP access was disabled entirely. This brought encrypted administrator sessions, protection against credential interception, and a more modern security posture overall.

## Administrative Authentication

The installer's default password was replaced with a strong administrator password:

```bash
sudo pihole setpassword
```

The password is stored as a secure hash, never in plaintext.

## Two-Factor Authentication

Time-based One-Time Passwords (TOTP) were added on top of the administrator password, using a standard authenticator app generating six-digit codes synchronized with the server. Logging into the dashboard now requires three things together: the session, the password, and a one-time code — meaningfully reducing the risk of a compromised password alone being enough to gain access.

## SSH Hardening

SSH public-key authentication is the preferred method for remote administration, rather than relying on passwords alone:

```
Administrator → SSH Private Key → Encrypted Channel → Raspberry Pi (Authorized Keys)
```

This provides stronger cryptographic authentication, protection against password guessing, faster logins, and easier automation for future tooling.

**A real lesson from this project:** during a Raspberry Pi OS reinstall, new SSH host keys were generated, which caused Windows to display:

```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

This warning exists specifically to catch man-in-the-middle attacks — seeing it is alarming by design. In this case it was expected, since the server's identity had legitimately changed. The old host key entry was removed from the local `known_hosts` file only after confirming the change was self-caused, and the new fingerprint was then accepted. The operational rule this reinforces: always verify *why* a host key changed before trusting the new one — the warning being expected this time doesn't make it safe to ignore reflexively next time.

## Fail2ban

Fail2ban monitors authentication logs and dynamically blocks IPs responsible for repeated failed login attempts against SSH:

```
Repeated Failed Logins → systemd Journal → Fail2ban Filter → Firewall Rule Added → Attacker Temporarily Blocked
```

The `sshd` jail was confirmed active and running.

## Firewall (UFW)

A host firewall was configured with UFW, denying all incoming connections by default and allowing all outgoing:

| Port | Protocol | Purpose |
|---|---|---|
| 22 | TCP | SSH |
| 53 | TCP/UDP | DNS |
| 123 | UDP | NTP |
| 443 | TCP | Pi-hole HTTPS |

Everything else is denied by default, confirmed via:

```bash
sudo ufw status verbose
```

## Automatic Security Updates

Automatic security updates were enabled so patches install without waiting on manual intervention — reducing the exposure window for known vulnerabilities and improving long-term stability without adding to routine maintenance overhead.

## Configuration Backups

Configuration represents the actual operational state of this infrastructure — packages can always be reinstalled from scratch, but configuration often can't be reconstructed from memory. An initial backup archive was created covering the core services deployed so far:

```bash
sudo tar -czf ~/pihole-backup-YYYY-MM-DD.tar.gz \
    /etc/pihole \
    /etc/unbound \
    /etc/systemd/system \
    /etc/hosts
```

Part 5 (Operations) expands this scope to also include `/etc/fail2ban` and `/etc/ssh`, once those services introduced their own configuration worth preserving — the backup strategy grows alongside the infrastructure it protects, rather than being fixed at this early stage.

## Remote Administration via WireGuard

Rather than exposing the Pi-hole admin interface directly to the internet, remote administrators connect through WireGuard first:

```
Internet → TP-Link DDNS → WireGuard VPN → Home LAN → Pi-hole
```

As established in Chapter 5, the GPON router's dynamic public IP is tracked via TP-Link's built-in DDNS service, and the WireGuard server itself runs on the AX10. This gives administrators end-to-end encrypted access to the entire internal network — not just Pi-hole — with no admin interface ever exposed publicly, keeping the attack surface to essentially just the WireGuard endpoint itself.

## Operational Monitoring

A small set of routine checks confirms the security stack is functioning, not just configured:

```bash
sudo systemctl status pihole-FTL
sudo systemctl status unbound
sudo systemctl status fail2ban
sudo ss -tulpn
dig openai.com @127.0.0.1
dig openai.com @127.0.0.1 -p 5335
sudo pihole tail
```

Part 5 (Operations) turns these into a full daily/weekly/monthly maintenance schedule; here, they simply confirm each control introduced in this chapter is actually doing its job.

## Defense-in-Depth Summary

```
Internet
    │
Dynamic DNS
    │
WireGuard VPN
    │
UFW Firewall
    │
Fail2ban
    │
SSH Public Keys
    │
HTTPS (TLS)
    │
Pi-hole v6
    │
Unbound
    │
DNSSEC Validation
    │
Root DNS Servers
```

Each layer addresses a distinct class of threat. No single control here is expected to provide complete protection on its own — that's the entire point of defense in depth.

## Lessons Learned

- Encrypt administrative interfaces from the start of production use, not as an afterthought once something feels "done."
- Multi-factor authentication meaningfully raises the bar for web-based administration, even on a home network.
- A host firewall is still worth running even on a trusted LAN — trust in the network doesn't guarantee trust in every device on it.
- SSH host key changes demand verification, especially right after a reinstall — the warning is doing its job even when the cause turns out to be benign.
- VPN-based remote administration beats exposing management interfaces directly to the internet, full stop.
- Regular backups matter more on small infrastructure than large infrastructure — a microSD card has a finite lifespan, and there's no redundant hardware underneath this deployment to fall back on.

## Chapter Summary

By the end of this phase, Event Horizon matured from a functioning DNS platform into a hardened one: administrative traffic encrypted, remote access tightly controlled, brute-force attempts mitigated, unnecessary services blocked by default, updates automated, and recovery procedures documented rather than assumed.

---

Part 4 covers Validation — formally testing and confirming that everything built across Parts 2 and 3 actually works as intended, rather than relying on the ad hoc verification steps used along the way.
