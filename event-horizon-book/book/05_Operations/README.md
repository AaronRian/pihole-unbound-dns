# Part 5: Operations, Maintenance & Disaster Recovery

## Overview

Once deployed, a DNS server becomes one of the most critical components on the network — every browser, operating system, mobile device, and IoT appliance on the LAN depends on it for name resolution. This part defines the operational procedures that keep Event Horizon healthy across its full lifecycle: monitoring, preventive maintenance, controlled updates, backups, disaster recovery, and capacity planning.

## Operational Philosophy

> Maintain proactively rather than recover reactively.

Routine maintenance catches small issues before they become service interruptions. Nothing here is reactive troubleshooting — it's a schedule designed to make troubleshooting rarely necessary in the first place.

## Daily Health Checks

```bash
sudo systemctl status pihole-FTL     # expect: active (running)
sudo systemctl status unbound        # expect: active (running)
sudo systemctl status fail2ban       # expect: active (running)
sudo ufw status                      # expect: active
dig openai.com @127.0.0.1            # expect: status: NOERROR
```

## Weekly Maintenance

```bash
sudo pihole -g          # refresh blocklists, remove obsolete domains, add new trackers
sudo apt update
sudo apt upgrade
sudo fail2ban-client status sshd    # review failed logins and banned IPs
```

Alongside these, the Pi-hole dashboard's query log is reviewed manually for false positives, suspicious domains, and unusual client behavior — the kind of thing automated checks won't flag on their own.

## Monthly Maintenance

```bash
sudo pihole -up          # update Pi-hole itself
sudo reboot               # verifies auto-start, service dependencies, boot reliability
sudo ss -tulpn            # confirm only expected services are listening
df -h                     # storage should remain comfortably below capacity
free -h                   # RAM is limited on the Zero W — worth checking regularly
```

Rebooting monthly isn't just routine — it's a live test that everything comes back up correctly on its own, rather than assuming it would.

## Monitoring

```bash
uptime                       # system load
lscpu                        # CPU information
vcgencmd measure_temp        # thermal state
free -m                      # memory
df -h                        # filesystem usage
systemctl --failed           # any failed units
sudo pihole tail             # live DNS query activity
```

## Performance Verification

```bash
dig github.com @127.0.0.1
```

Comparing the first query (~300–700 ms) against a second, cached query (<20 ms) confirms caching behavior is still working correctly — a quick sanity check to run after any update.

## Backup Strategy

Configuration is worth more than the packages that use it — packages can always be reinstalled; hand-tuned configuration often can't be reconstructed from memory. The backup scope expands on what Part 3 introduced, now also covering Fail2ban and SSH:

```bash
sudo tar -czf ~/event-horizon-backup-$(date +%F).tar.gz \
    /etc/pihole \
    /etc/unbound \
    /etc/fail2ban \
    /etc/ssh \
    /etc/systemd/system \
    /etc/hosts
```

| Item | Frequency |
|---|---|
| Configuration archive | Weekly |
| Full SD card image | Quarterly |
| GitHub documentation | Continuous |

## Disaster Recovery

If the SD card fails:

1. Install Raspberry Pi OS Lite.
2. Restore the hostname (`event-horizon`).
3. Restore SSH (keys and configuration).
4. Install Pi-hole, Unbound, Fail2ban, and UFW.
5. Restore the configuration archive from the latest backup.
6. Restart services:
   ```bash
   sudo systemctl restart pihole-FTL
   sudo systemctl restart unbound
   sudo systemctl restart fail2ban
   ```
7. Verify functionality end to end.

## Recovery Validation

A backup is only as good as its tested restore. Recovery testing should confirm:

- DNS resolution
- Web interface
- HTTPS
- SSH
- WireGuard
- Fail2ban
- Firewall rules
- DNSSEC validation

## Logging

| Service | Log Location |
|---|---|
| Pi-hole | `/var/log/pihole/` |
| Unbound | `journalctl -u unbound` |
| SSH | `journalctl -u ssh` |
| Fail2ban | `journalctl -u fail2ban` |
| Kernel | `journalctl -k` |

## Capacity Planning

| Resource | Utilization |
|---|---|
| CPU | Low |
| Memory | Low |
| Storage | Low |
| Network | Low |

The Raspberry Pi Zero W comfortably supports a typical household deployment as-is. Reasonable triggers to reconsider the hardware in the future include serving hundreds of clients, heavy DNS logging, adding further services (Grafana, Prometheus, an IDS), or maintaining large local DNS zones — none of which apply to the current deployment.

## Operational Checklist

**Daily** — verify services, confirm DNS resolution.
**Weekly** — update Gravity, update packages, review logs and blocked domains.
**Monthly** — upgrade Pi-hole, reboot, verify listening ports, create a configuration backup.
**Quarterly** — create a full SD card image, test the restore procedure, review SSH keys, audit firewall rules.

## Operational Maturity

At this stage, Event Horizon provides:

✓ Recursive DNS &nbsp; ✓ DNSSEC validation &nbsp; ✓ Network-wide filtering &nbsp; ✓ HTTPS administration &nbsp; ✓ SSH key authentication &nbsp; ✓ Fail2ban protection &nbsp; ✓ UFW firewall &nbsp; ✓ WireGuard remote access &nbsp; ✓ Automatic updates &nbsp; ✓ Configuration backups &nbsp; ✓ Disaster recovery procedures &nbsp; ✓ A defined maintenance schedule

## Lessons Learned

- Routine verification beats troubleshooting after something has already failed.
- Configuration backups are lightweight enough that there's no good reason to skip them.
- Rebooting on a schedule confirms services actually recover correctly — an assumption is not the same as a test.
- Reviewing logs regularly surfaces unusual client behavior and potential issues before they escalate into something bigger.

## Chapter Summary

Event Horizon is now more than a functioning DNS server — it's an operational platform with a defined maintenance routine, tested recovery procedures, and a documented lifecycle plan. Treating the Raspberry Pi as infrastructure, rather than a fire-and-forget appliance, is what makes the deployment predictable and resilient rather than merely functional.

---

Part 6 shifts from operations back to concepts: the DNS theory underpinning everything built so far, for anyone reading this documentation who wants the "why" behind the configuration, not just the "how."
