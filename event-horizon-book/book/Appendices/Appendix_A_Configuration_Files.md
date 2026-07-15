# Appendix A: Configuration File Reference

> **Note:** These configurations reflect the exact settings and design goals documented throughout this book (loopback-only binding, port 5335, DNSSEC validation, QNAME minimization, cache sizing for the Pi Zero W, the UFW rule set, and so on). Where your actual deployed file differs in formatting or additional local tweaks, replace the version here with your real file directly from the device — this appendix is meant to be a working reference, not a substitute for the source of truth on disk.

## Unbound — `/etc/unbound/unbound.conf.d/pi-hole.conf`

```
server:
    # Loopback only — never exposed beyond this host (Chapter 10)
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-ip6: no
    do-udp: yes
    do-tcp: yes

    # Local access only
    access-control: 127.0.0.1/32 allow
    access-control: 0.0.0.0/0 refuse

    # DNSSEC validation (Chapter 11 / Part 6)
    auto-trust-anchor-file: "/var/lib/unbound/root.key"

    # Privacy — QNAME minimisation
    qname-minimisation: yes

    # Hardening defaults
    harden-glue: yes
    harden-dnssec-stripped: yes
    harden-below-nxdomain: yes
    harden-referral-path: yes
    unwanted-reply-threshold: 10000000

    # DNS rebinding protection
    private-address: 192.168.0.0/16
    private-address: 10.0.0.0/8
    private-address: 172.16.0.0/12

    # Cache sizing — tuned for the Pi Zero W's 512 MB RAM (Chapter 4)
    rrset-cache-size: 8m
    msg-cache-size: 4m
    cache-min-ttl: 300
    cache-max-ttl: 86400

    # Performance
    prefetch: yes
    prefetch-key: yes

    # Minimal logging (Chapter 7's low-noise philosophy)
    verbosity: 0
    log-queries: no
```

## Pi-hole — Upstream Resolver (`pihole.toml` excerpt)

```toml
[dns]
upstreams = ["127.0.0.1#5335"]
```

Verified with:

```bash
sudo pihole-FTL --config
```

## Fail2ban — `/etc/fail2ban/jail.local`

```ini
[sshd]
enabled  = true
port     = 22
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 5
bantime  = 3600
findtime = 600
```

## UFW Firewall Rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp      # SSH
sudo ufw allow 53          # DNS (TCP + UDP)
sudo ufw allow 123/udp     # NTP
sudo ufw allow 443/tcp     # Pi-hole HTTPS

sudo ufw enable
sudo ufw status verbose
```

## Backup Script Reference

```bash
sudo tar -czf ~/event-horizon-backup-$(date +%F).tar.gz \
    /etc/pihole \
    /etc/unbound \
    /etc/fail2ban \
    /etc/ssh \
    /etc/systemd/system \
    /etc/hosts
```

---

**Appendix B** consolidates the network and DNS-flow diagrams referenced throughout this book into a single visual reference.
