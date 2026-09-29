# Network Architecture

## 1. Goal

The laptop should automatically use the DNS server running on the homeserver, so local names such as `web.home.arpa` work without manually configuring `resolvectl`.

If CoreDNS does not know a name locally, the DNS query should be forwarded to an upstream DNS server.

In a later phase, redundancy will be added so DNS remains available when the homeserver is unavailable.


## 2. Current Network

TP-Link DHCP
    │
    │ provides network configuration
    ▼
Laptop
├── IP:      192.168.0.111
├── Gateway: 192.168.0.1
└── DNS:     192.168.0.10 192.168.0.1


## 3. DNS Design

TP-Link DHCP
    │
    │ provides DNS = 192.168.0.10
    ▼
Laptop
    │
    │ DNS query
    ▼
CoreDNS on homeserver
192.168.0.10:53
    │
    ├── Local name → answer from local configuration
    │                 web.home.arpa = 192.168.0.10
    │
    └── Unknown name → forward to upstream DNS
                       192.168.0.1


## 4. Known Limitations

The homeserver and CoreDNS currently form a **Single Point of Failure (SPOF)**.

If either becomes unavailable, clients using `192.168.0.10` as their DNS server can no longer resolve DNS names.

Network connectivity may still exist and services may still be reachable directly by IP address.

DNS unavailable does not necessarily mean the network is unavailable:

- `web.home.arpa` → cannot be resolved
- `192.168.0.10` → may still be reachable directly

- The laptop currently receives both `192.168.0.10` and `192.168.0.1` as DNS servers, although only `192.168.0.10` was intentionally configured as the homelab DNS server. The source of `192.168.0.1` still needs to be investigated.


## 5. Future Design

DNS A and DNS B should run on independent machines or nodes to prevent the physical server from remaining a **Single Point of Failure (SPOF)**.

Both DNS servers should provide the same local DNS records so clients can continue resolving local names if one DNS server becomes unavailable.


## 6. Verification

- The laptop automatically receives `192.168.0.10` as its DNS server through DHCP.
- `web.home.arpa` resolves to `192.168.0.10` without manually using `resolvectl`.
- Public DNS names can still be resolved through the configured upstream DNS server.
- `http://web.home.arpa` is reachable from the laptop.
