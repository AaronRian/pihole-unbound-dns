# Part 6: DNS Theory

## Why This Part Exists

Every previous part documented *what* was built and *how*. This part exists to explain *why* it works the way it does — the DNS concepts underpinning the entire deployment, for anyone reading this documentation who wants to genuinely understand the system rather than just reproduce it.

## Why DNS Exists

Humans remember names. Computers communicate using IP addresses:

```
openai.com  →  104.18.33.45 / 172.64.154.211
```

Without DNS, every application would require users to remember and type numeric addresses instead of names. DNS is the internet's distributed directory service — it exists purely to bridge that gap.

## Why Recursive DNS Matters

Most home networks rely on an external recursive DNS provider:

```
Laptop → Cloudflare (1.1.1.1) → Internet
```

Google DNS and Quad9 work the same way. In this model, the provider performs recursion on your behalf — which also means the provider sees every cache miss, maintains the recursive cache, performs DNSSEC validation, and becomes part of your trust chain, whether or not that's something you'd choose deliberately.

Event Horizon replaces that provider with itself:

```
Laptop → Pi-hole → Unbound → Root Servers
```

The resolver performing recursion is the same device sitting on the LAN, not a third party.

## A Complete DNS Resolution Walkthrough

Consider a browser requesting `https://openai.com`:

1. **Browser** needs an IP address to connect to.
2. **Operating system** checks its local DNS cache and hosts file. If absent, it queries Pi-hole.
3. **Pi-hole** checks its blocklists, allowlists, local DNS records, and cache. If the domain isn't blocked and isn't already cached, it forwards the query to `127.0.0.1:5335`.
4. **Unbound** checks its own recursive cache. If empty, it begins recursion from scratch.
5. **Root servers** are asked who's responsible for `.com`.
6. **TLD servers** (`.com`) are asked who's authoritative for `openai.com`.
7. **The authoritative server** for `openai.com` returns the actual answer.
8. **DNSSEC validation** checks signatures, the delegation chain, and the root trust anchor before the answer is trusted.
9. **Caching** stores the result with its TTL (for example, 300 seconds) — the next request for the same domain skips the entire hierarchy walk.

```
Browser
   │
Windows DNS Cache
   │
Pi-hole (192.168.0.20) — blocklists + local cache
   │
Unbound (127.0.0.1:5335) — recursive cache
   │
Root Name Servers → TLD Name Servers → Authoritative Server
   │
DNSSEC Validation
   │
Pi-hole Cache → Client
```

## Why Port 5335

Chapter 10 covers the full story of the port 53 conflict between Pi-hole and Unbound, and the fix — isolating Unbound on `127.0.0.1:5335`. Conceptually, this separation is now a standard, widely adopted pattern for exactly this kind of pairing: Pi-hole owns the network-facing port, Unbound stays purely internal.

## The Cache Hierarchy

Caching exists at multiple layers simultaneously, each one reducing both latency and the number of external queries that ever need to happen:

```
Browser Cache → OS DNS Cache → Pi-hole Cache → Unbound Cache → Root Servers
```

A query only travels as far up this chain as it needs to — most real-world traffic never reaches the root servers at all, because something closer to the client already has the answer.

## Why the First Query Is Slower

As established in Chapter 11, an initial recursive lookup typically takes 300–700 ms, while a cached lookup completes in under 20 ms. The first request has to traverse the full hierarchy — root, then TLD, then authoritative — while the second is served entirely from local cache. This isn't a performance flaw; it's the expected shape of recursive resolution, and it's the specific trade-off accepted in exchange for not depending on a third-party resolver.

## TTL (Time to Live)

Every DNS record carries a TTL — for example, 300 seconds. This tells every cache along the chain how long the answer can be reused before it must be re-fetched. A short TTL keeps records fresh at the cost of more frequent lookups; a long TTL favors speed at the cost of staleness if the record changes in the meantime.

## The DNS Hierarchy

**Root servers** are where the internet's naming system begins. There are 13 logical root server identities (A through M), each implemented by many geographically distributed anycast instances for resilience and low latency. Root servers don't know individual domains — their job is purely to direct resolvers to the correct TLD servers.

**TLD servers** (`.com`, `.org`, `.net`, `.in`, `.gov`, and others) answer a narrower question: who is authoritative for a specific domain within that TLD.

**Authoritative servers** are the final source of truth — the only servers that can definitively answer what a given domain actually resolves to.

## DNSSEC in Practice

Without DNSSEC, a forged DNS response can simply be accepted by a client with no way to detect the forgery. With DNSSEC, Unbound validates the cryptographic signature chain — up through the delegation path to the root trust anchor — before ever returning a result to Pi-hole. A forged response fails that validation and is rejected outright, rather than silently trusted.

## Why This Architecture Matters

Most Pi-hole installations simply forward queries to Cloudflare, Google Public DNS, or Quad9 — functional, but placing every cache miss in a third party's hands. Event Horizon performs recursion locally instead, which brings genuine administrative control over DNS infrastructure, reduced reliance on any third-party recursive resolver, local caching under direct control, DNSSEC validation happening on hardware the operator owns, and real educational value in understanding how DNS actually works end to end.

Worth being precise about what this does and doesn't achieve: it does not make browsing anonymous — authoritative servers still see queries for the domains actually visited, the same as they would under any other resolver. What it removes is dependence on an *external recursive resolver* for routine name resolution — a meaningfully narrower, but still significant, privacy improvement.

## Knowledge Check

By this point, a reader following this documentation should be able to explain:

- The distinction between Pi-hole's role and Unbound's role.
- Why Unbound listens on `127.0.0.1:5335` specifically.
- The full sequence of a recursive DNS lookup, root to authoritative.
- What DNSSEC actually protects against.
- Why a first lookup is slower than every subsequent one.
- How caching improves performance at multiple layers simultaneously.
- The distinction between a recursive resolver and an authoritative name server.
- Why multiple layers of DNS caching exist in a typical client-to-server path.

---

Part 7 returns to the practical side: a consolidated troubleshooting reference covering every significant issue encountered across this entire build, with root causes and resolutions documented together in one place.
