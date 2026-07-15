# Chapter 11: Integrating Pi-hole and Unbound

Unbound now resolves queries correctly on its own. This chapter connects it to Pi-hole as the sole upstream resolver and verifies the complete pipeline end to end — closing out Part 2 of this book.

## Reconfiguring Pi-hole's Upstream

Pi-hole's temporary upstream (Cloudflare, set in Chapter 9 purely as a placeholder) was replaced with the local Unbound instance:

```
127.0.0.1#5335
```

Verified directly against Pi-hole's own configuration:

```bash
sudo pihole-FTL --config
```

Output confirmed:

```
dns.upstreams = [127.0.0.1#5335]
```

Pi-hole no longer has any path to a third-party recursive resolver — every query it can't already answer from cache or blocklists now goes to Unbound, and nowhere else.

## End-to-End Verification

A query against Pi-hole itself confirms the full chain works together, not just Unbound in isolation:

```bash
dig openai.com @127.0.0.1
```

The response was served by Pi-hole, consulting Unbound behind the scenes when the answer wasn't already cached. Real-time logging provided direct confirmation of this path:

```bash
sudo pihole tail
```

Typical log entries showed:

```
forwarded openai.com
to 127.0.0.1#5335
```

This is worth treating as its own verification step, separate from testing Unbound alone in Chapter 10 — a component working correctly in isolation doesn't guarantee it's actually being used correctly by the service in front of it.

## DNSSEC Validation

DNSSEC validation is active within the recursive resolution process, confirming that responses haven't been altered anywhere between the authoritative server and the resolver. Successful validation is visible directly in query responses via the **`ad`** (Authenticated Data) flag — a quick way to confirm cryptographic validation is actually happening, rather than just assuming it because it's configured.

## Performance Characteristics

| Query Type | Typical Latency |
|---|---|
| Initial recursive lookup | 300–700 ms |
| Cached lookup | < 20 ms |

This is the practical trade-off of recursive resolution made concrete: a slower first lookup for any given domain, in exchange for zero dependency on a third-party recursive resolver and very fast responses for everything after. Full system-level benchmarking — boot time, RAM at each stage, and so on — is formalized in Part 4 (Validation); these numbers specifically characterize the DNS resolution path itself.

## Lessons Learned

- Real-time query logs (`pihole tail`) are the most reliable way to confirm Pi-hole is actually forwarding to the intended upstream — trusting the configuration file alone isn't the same as watching the traffic itself.
- Testing the full path through Pi-hole is a distinct verification step from testing Unbound directly. Each component working correctly on its own doesn't guarantee the integration between them is correct — both need to be checked.

## Chapter Summary

With this integration complete, Event Horizon has moved from a DNS filtering appliance into a complete recursive DNS infrastructure. Pi-hole handles network-wide filtering and client-level visibility; Unbound performs recursive resolution and DNSSEC validation, entirely locally. For normal operation, the system no longer depends on any public recursive DNS provider.

---

This closes Part 2 — Platform Deployment. Part 3 picks up from here with Security: HTTPS administration, SSH key authentication, Fail2ban, UFW, automatic updates, configuration backups, and the WireGuard remote access setup — including the TP-Link DDNS layer that makes it reachable from anywhere.
