# Appendix E: Security Audit Checklist

A periodic self-audit checklist for re-verifying the hardening covered in Part 3, styled after CIS-benchmark-type reviews but scoped specifically to this deployment. Intended to be run quarterly, alongside the SD card image and restore test from Part 5's operational checklist.

## Access Control & Authentication

| Item | Expected State | Verify With |
|---|---|---|
| Root SSH login disabled | Disabled | `sudo grep PermitRootLogin /etc/ssh/sshd_config` |
| SSH key authentication enforced | Password auth disabled or de-prioritized | `sudo grep PasswordAuthentication /etc/ssh/sshd_config` |
| Pi-hole admin password | Strong, rotated periodically | `sudo pihole setpassword` |
| Two-factor authentication (TOTP) | Enabled on admin dashboard | Manual login check |
| Dedicated non-root admin account (custom, non-default username) | Sole account used for administration | `who` / `last` |

## Network Exposure

| Item | Expected State | Verify With |
|---|---|---|
| UFW default policy | Deny incoming, allow outgoing | `sudo ufw status verbose` |
| Only required ports open | 22, 53, 123, 443 | `sudo ufw status verbose` |
| No unexpected listening services | Only Pi-hole, Unbound, SSH | `sudo ss -tulpn` |
| Unbound bound to loopback only | `127.0.0.1:5335`, not exposed | `sudo ss -tulpn \| grep :5335` |
| Wi-Fi radio disabled | `wlan0` unmanaged/inactive | `nmcli device status` |

## Encryption

| Item | Expected State | Verify With |
|---|---|---|
| Pi-hole admin interface | HTTPS only, HTTP disabled | Browser check on port 443 |
| WireGuard tunnel | Active, encrypting all remote traffic | `sudo wg show` |
| DNSSEC validation | Trust anchor loaded, `ad` flag present on valid queries | `dig +dnssec openai.com @127.0.0.1` |

## Monitoring & Intrusion Prevention

| Item | Expected State | Verify With |
|---|---|---|
| Fail2ban active | `sshd` jail enabled and running | `sudo fail2ban-client status sshd` |
| No unexpected banned/failed-login patterns | Reviewed, nothing anomalous | Fail2ban log review |
| Failed systemd units | None | `systemctl --failed` |

## Patching & Updates

| Item | Expected State | Verify With |
|---|---|---|
| OS packages up to date | No pending security updates | `sudo apt update && apt list --upgradable` |
| Automatic security updates | Enabled | `systemctl status unattended-upgrades` |
| Pi-hole core/FTL/web versions | Current stable release | `pihole -v` |
| Unbound version | Current stable release | `unbound -V` |

## Backup & Recovery

| Item | Expected State | Verify With |
|---|---|---|
| Configuration backup exists | Less than 7 days old | Check backup archive timestamp |
| Full SD card image exists | Less than 1 quarter old | Check image file timestamp |
| Restore procedure tested | Verified within the last quarter | Manual restore test (Part 5) |

## Remote Access

| Item | Expected State | Verify With |
|---|---|---|
| No admin interface exposed directly to the internet | Confirmed — access only via WireGuard | External port scan of public IP |
| WireGuard port-forward target correct | `192.168.1.212`, matching AX10's DHCP reservation | GPON router admin panel |
| TP-Link DDNS hostname resolves correctly | Points to current public IP | `dig <ddns-hostname>` from outside the LAN |

---

Running through this checklist quarterly turns "the system is secure" from an assumption made once during Part 3 into something re-verified on a schedule — the same philosophy Part 5 applies to operational health in general.

**Appendix F** closes out the appendices with a list of concrete future enhancements for this project.
