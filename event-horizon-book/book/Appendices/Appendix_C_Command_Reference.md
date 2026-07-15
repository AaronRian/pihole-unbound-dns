# Appendix C: Command Reference

Every command used across this book, grouped by purpose, for quick lookup without paging back through individual chapters.

## System Verification

```bash
uname -a               # kernel, architecture, OS identity
free -h                # memory usage
uptime                  # load averages, time since boot
systemctl --failed     # any failed systemd units
systemd-analyze        # boot time breakdown
top -bn1 | grep "Cpu(s)"  # CPU idle / utilization snapshot
```

## OS Updates

```bash
sudo apt update
sudo apt full-upgrade
```

## Network & Interfaces

```bash
nmcli connection show                    # list configured connections
sudo nmcli connection delete "<name>"    # remove a connection profile (e.g. old Wi-Fi)
sudo nmcli radio wifi off                # disable the Wi-Fi radio entirely
nmcli device status                      # interface status at a glance
ip addr show                             # all interfaces and addresses
ip addr show eth0                        # Ethernet interface specifically
ping -c 4 192.168.0.1                    # verify gateway reachability
```

## Pi-hole

```bash
# Installation
curl -fsSL https://install.pi-hole.net -o basic-install.sh
ls -lh basic-install.sh && head -40 basic-install.sh && tail -40 basic-install.sh
sudo bash basic-install.sh

# Status & verification
sudo systemctl status pihole-FTL
sudo ss -tulpn | grep :53
sudo ss -tulpn | grep :80      # or :443 once HTTPS is active
pihole status
pihole -v

# Administration
sudo pihole setpassword
pihole -g               # refresh Gravity / blocklists (weekly)
pihole -up               # update Pi-hole itself (monthly)
pihole reloaddns         # Pi-hole v6 equivalent of the old "restartdns"
sudo pihole tail         # live query log
sudo pihole-FTL --config  # dump current FTL configuration
```

## Unbound

```bash
sudo apt install unbound -y
unbound -V                          # confirm installed version
sudo unbound-checkconf              # validate config before restarting
sudo systemctl status unbound
sudo ss -tulpn | grep :5335         # confirm loopback-only binding
```

## DNS Testing

```bash
dig openai.com @127.0.0.1              # query through Pi-hole
dig openai.com @127.0.0.1 -p 5335      # query Unbound directly
dig github.com @127.0.0.1              # second domain / cache comparison
```

## Security

```bash
# UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 53
sudo ufw allow 123/udp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose

# Fail2ban
sudo systemctl status fail2ban
sudo fail2ban-client status sshd
```

## Monitoring & Logs

```bash
lscpu                          # CPU information
vcgencmd measure_temp          # thermal state (Pi-specific)
free -m
df -h                          # disk usage

journalctl -u unbound
journalctl -u ssh
journalctl -u fail2ban
journalctl -k                  # kernel log
```

## Backup & Disaster Recovery

```bash
sudo tar -czf ~/event-horizon-backup-$(date +%F).tar.gz \
    /etc/pihole \
    /etc/unbound \
    /etc/fail2ban \
    /etc/ssh \
    /etc/systemd/system \
    /etc/hosts

# Recovery — restart services after restoring the archive
sudo systemctl restart pihole-FTL
sudo systemctl restart unbound
sudo systemctl restart fail2ban
```

---

**Appendix D** (Troubleshooting Knowledge Base) is covered in full in Part 7 rather than repeated here. **Appendix E** provides a security audit checklist for periodically re-verifying the hardening covered in Part 3.
