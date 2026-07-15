# Appendix B: Network & DNS Flow Diagrams

A consolidated visual reference pulling together every diagram referenced across this book, in one place.

## Physical & Logical Topology (Dual-NAT)

Referenced in Chapter 5. Shows the two independent NAT layers and where each device sits.

```mermaid
flowchart TD
    Internet["Internet"] --> ISP["FTTH ISP Connection"]
    ISP --> GPON

    subgraph NAT1["NAT Layer 1 — 192.168.1.0/24"]
        GPON["TP-Link GPON Router<br/>WAN: ISP-assigned<br/>LAN: 192.168.1.1"]
    end

    GPON -->|"UDP 51820<br/>Port Forward"| AX10

    subgraph NAT2["NAT Layer 2 — 192.168.0.0/24"]
        AX10["TP-Link Archer AX10<br/>WAN: 192.168.1.212<br/>LAN: 192.168.0.1<br/>WireGuard Server"]
        AX10 --> Pi["Raspberry Pi Zero W<br/>event-horizon<br/>192.168.0.20<br/>Pi-hole + Unbound"]
        AX10 --> Clients["LAN Clients<br/>192.168.0.21–149<br/>DNS via DHCP → .20"]
    end

    Remote["Remote WireGuard Client<br/>(anywhere)"] -.->|"via TP-Link DDNS"| GPON
```

## DNS Resolution Flow

Referenced in Part 6 (DNS Theory). Shows a single query's full path from client to authoritative server and back.

```mermaid
flowchart TD
    Client["Client Device"] --> OSCache["OS DNS Cache / Hosts File"]
    OSCache -->|"cache miss"| Pihole["Pi-hole (192.168.0.20)<br/>Blocklists + Cache"]
    Pihole -->|"not blocked, not cached"| Unbound["Unbound (127.0.0.1:5335)<br/>Recursive Cache"]
    Unbound -->|"cache miss — begin recursion"| Root["Root Name Servers"]
    Root --> TLD["TLD Name Servers (.com, .org, ...)"]
    TLD --> Auth["Authoritative Name Servers"]
    Auth --> DNSSEC["DNSSEC Validation"]
    DNSSEC --> PiCache["Pi-hole Cache"]
    PiCache --> Client
```

## Remote Access & Defense-in-Depth

Referenced in Chapter 5 and Part 3 (Security). Shows the full layered path a remote administrator's connection passes through.

```mermaid
flowchart TD
    Internet["Internet"] --> DDNS["TP-Link Dynamic DNS"]
    DDNS --> WG["WireGuard VPN<br/>(AX10, port 51820)"]
    WG --> UFW["UFW Firewall"]
    UFW --> F2B["Fail2ban"]
    F2B --> SSHKeys["SSH Public Key Auth"]
    SSHKeys --> HTTPS["HTTPS (TLS)"]
    HTTPS --> Pihole2["Pi-hole v6"]
    Pihole2 --> Unbound2["Unbound"]
    Unbound2 --> DNSSEC2["DNSSEC Validation"]
    DNSSEC2 --> RootServers["Root DNS Servers"]
```

---

**Appendix C** provides a consolidated command reference — every command from every chapter, grouped by purpose, for quick lookup without hunting back through individual chapters.
