# Chapter 9: Installing Pi-hole

With networking finalized, this chapter installs the first application service on top of it: Pi-hole, the DNS filtering layer that gives Event Horizon its network-wide ad and tracker blocking.

## Why Pi-hole

Browser-based ad blockers only protect the browser they're installed in — smartphones, smart TVs, IoT devices, and game consoles are left completely unprotected. Pi-hole solves this at the DNS layer instead: it filters requests to advertising and tracking domains before a connection to them is ever established, covering every device on the network simultaneously, regardless of what that device is or what software runs on it.

## Installation Method

Rather than piping the official installer straight into Bash — a common but opaque pattern — the script was downloaded first, inspected, and only then executed:

```bash
mkdir -p ~/Downloads
cd ~/Downloads

curl -fsSL https://install.pi-hole.net -o basic-install.sh

ls -lh basic-install.sh
head -40 basic-install.sh
tail -40 basic-install.sh
```

Only after reviewing the script's size and contents was it run:

```bash
sudo bash basic-install.sh
```

This adds a small amount of friction in exchange for a real benefit: nothing runs on the system without having been looked at first, and the script itself is archived alongside the rest of the project's history rather than disappearing after a single `curl | bash` execution.

## Installation Parameters

The installer detected the environment automatically:

| Parameter | Value |
|---|---|
| Interface | eth0 |
| IPv4 Address | 192.168.0.20 |
| IPv6 | Disabled |
| Initial Upstream DNS | Cloudflare (temporary) |
| Query Logging | Enabled |
| Privacy Level | 0 |
| Blocklist | StevenBlack Unified Hosts |

Cloudflare was accepted as the upstream resolver deliberately, and only temporarily — Unbound had not been installed yet at this point in the build, and Pi-hole needs *some* working upstream to function and be tested in the meantime. This is replaced with the local Unbound resolver in the next chapter.

## Gravity Database

Immediately after installation, Pi-hole built its initial Gravity database — the compiled blocklist it filters queries against:

| Metric | Value |
|---|---|
| Domains Imported | 78,188 |
| Exact Domains | 78,188 |
| Regex Rules | 0 |
| Allowlist Entries | 0 |
| Denylist Entries | 0 |

The StevenBlack Unified Hosts list was used as the baseline blocklist — a well-maintained list that offers strong coverage while keeping false positives low, making it a sensible default before any custom allow/deny rules are added.

## Verifying the Installation

```bash
sudo systemctl status pihole-FTL
```
Confirmed the service **active**, **running**, and **enabled on boot**.

```bash
sudo ss -tulpn | grep :53
```
Confirmed Pi-hole listening on port 53 over both UDP and TCP, IPv4 and IPv6.

```bash
sudo ss -tulpn | grep :80
```
Confirmed the integrated web server listening on HTTP at this stage — this is migrated to HTTPS during hardening, covered in Part 3 (Security).

```bash
pihole status
```
Confirmed DNS service listening, blocking enabled.

```bash
pihole -v
```

| Component | Version |
|---|---|
| Core | v6.4.3 |
| Web | v6.6 |
| FTL | v6.7 |

## Functional Testing

Before Unbound entered the picture, Pi-hole was tested against its temporary upstream:

```bash
dig openai.com @127.0.0.1
```

A successful response confirmed the DNS listener, cache, and forwarding path were all functioning correctly — a working baseline to build on before adding any further complexity.

## Administrative Interface

The web dashboard became available at:

```
http://192.168.0.20/admin
```

At this stage it was reachable over plain HTTP with the installer's default credentials — acceptable for initial verification on a trusted LAN, but not for ongoing use. Part 3 (Security) covers migrating this to HTTPS, setting a strong administrator password, and adding TOTP two-factor authentication.

## Initial Challenges

**Temporary upstream resolver.** Relying on Cloudflare briefly was intentional, not an oversight — it let every other part of the Pi-hole deployment be verified independently, before Unbound's own installation introduced its own set of problems (covered in the next chapter) into the mix at the same time.

**Pi-hole v6 configuration model.** Earlier Pi-hole versions relied primarily on `dnsmasq` configuration files. Version 6 introduces a TOML-based configuration system managed through `pihole-FTL`, which meant existing guides and assumptions based on v5 needed to be re-checked rather than trusted outright.

**Backup permission warnings.** The installer produced non-fatal warnings during setup:

```
copy_file(): Failed to open ...
Permission denied
```

These did not affect installation or normal operation, but were noted for follow-up rather than dismissed outright — a warning that doesn't break anything today isn't necessarily one to ignore permanently.

## Operational State at End of Phase

| Component | Status |
|---|---|
| Pi-hole Installed | ✓ |
| DNS Service Running | ✓ |
| Web Dashboard | ✓ |
| Gravity Database | ✓ |
| Query Logging | ✓ |
| DHCP Service | Disabled (the AX10 handles DHCP for the LAN) |
| HTTPS | Pending |
| Unbound | Pending |

## Lessons Learned

- Downloading and reviewing an installer script before execution is a small habit with a real payoff in transparency.
- A temporary public upstream resolver is a reasonable way to stage a deployment without conflating multiple new services at once.
- Verifying a service immediately after installation — rather than moving straight to the next task — makes it much easier to isolate where a later problem actually originated.
- Pi-hole v6's configuration model differs meaningfully from v5, and that difference is worth documenting rather than assuming prior knowledge still applies.

---

Pi-hole is now filtering and serving DNS for the entire LAN, but still leaning on a third-party resolver behind the scenes. The next chapter replaces that dependency entirely: installing Unbound as a fully local recursive resolver.
