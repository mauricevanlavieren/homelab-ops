Current Session --- 2026-09-28

Project

Project 1 --- Mini Platform

Today's focus

Make the homelab DNS design persistent for clients through DHCP and
verify the DNS path.

Context / Architecture

Desired local request flow:

Laptop / Browser
      │
      │ DNS query
      ▼
systemd-resolved
127.0.0.53
      │
      │ DNS server received via DHCP
      ▼
CoreDNS
192.168.0.10:53
      │
      ├── local name known
      │      web.home.arpa → 192.168.0.10
      │
      └── unknown/public name
             → forward to upstream DNS 192.168.0.1

HTTP path after DNS resolution:

Browser
  │
  │ http://web.home.arpa
  ▼
192.168.0.10:80
  │
  ▼
Traefik
  │
  ▼
Ingress: host = web.home.arpa
  │
  ▼
Service: web:80
  │ selector app=nginx
  ▼
nginx Pod

DNS design decision

TP-Link DHCP is configured to provide the homelab DNS server to clients.

Primary DNS:   192.168.0.10
Secondary DNS: left blank

Current accepted limitation:

192.168.0.10 / CoreDNS is currently a Single Point of Failure
(SPOF).

True DNS redundancy will be added later using an independent
machine/node.

Two DNS Pods on the same physical server would not solve physical
server failure.

Architecture documentation: docs/architecture/network-architecture.md

What we observed

1. Network connectivity to homeserver works

ping 192.168.0.10

Result:

6 packets transmitted
6 received
0% packet loss

Conclusion: laptop → homeserver network connectivity works.

2. First dig test contained a syntax mistake

Used:

dig 192.168.0.10 web.home.arpa

This does not select 192.168.0.10 as DNS server. It asks the
current resolver to resolve both arguments as names.

Evidence showed:

SERVER: 127.0.0.53#53

Correct syntax for explicitly selecting a DNS server:

dig @192.168.0.10 web.home.arpa

3. Direct CoreDNS test succeeded

Result:

status: NOERROR
web.home.arpa.  IN A  192.168.0.10
SERVER: 192.168.0.10#53

Conclusion: CoreDNS itself correctly resolves the local record.

4. Laptop initially still had old DHCP DNS configuration

Before reconnecting Ethernet:

Current DNS Server: 192.168.0.1
DNS Servers:        192.168.0.1

Hypothesis: existing DHCP lease still contained the previous DNS
configuration.

Ethernet was disconnected briefly and reconnected so the laptop obtained
fresh DHCP configuration.

After reconnect:

Current DNS Server: 192.168.0.10
DNS Servers:        192.168.0.10 192.168.0.1

Important open observation:

The router's Secondary DNS field was left blank, but the laptop
still received 192.168.0.1 as an additional DNS server.

Do not change this blindly. Investigate later why the TP-Link/DHCP
configuration supplies it.

5. Automatic local DNS resolution succeeded

Without manually specifying @192.168.0.10:

dig web.home.arpa

Result:

status: NOERROR
web.home.arpa → 192.168.0.10
SERVER: 127.0.0.53#53

Interpretation:

dig
 ↓
systemd-resolved (127.0.0.53)
 ↓
CoreDNS (192.168.0.10)
 ↓
web.home.arpa = 192.168.0.10

This proves the laptop now automatically uses the homelab DNS path.

6. Public/upstream DNS resolution succeeded

Test:

dig google.nl

Result:

status: NOERROR
google.nl → 216.58.198.35

Conclusion: public DNS resolution still works. The intended forwarding
path is functioning end-to-end from the client perspective.

Acceptance criteria status

Laptop receives 192.168.0.10 automatically as DNS through
DHCP.

web.home.arpa resolves to 192.168.0.10 without manual
resolvectl.

Public DNS names resolve.

Verify http://web.home.arpa in Brave after the persistent DHCP
DNS change.

Investigate why 192.168.0.1 is also supplied as DNS while
Secondary DNS is blank.

Deliberately break/test DNS and troubleshoot it later.

Update architecture documentation with final as-built state
if investigation changes anything.

Exact stopping point

We were about to perform the final user-facing end-to-end test:

Open in Brave:
http://web.home.arpa

Next session starts here.

First observe whether the nginx page loads. Do not change configuration
before this test.

Evidence / learning today

Network / DNS

Practiced and connected:

Same-subnet laptop → homeserver connectivity.

Difference between a DNS server and a default gateway.

DHCP distributes DNS configuration to clients.

Existing DHCP leases can temporarily retain old configuration.

systemd-resolved is the laptop's local resolver.

127.0.0.53 is the local stub resolver, not the homelab DNS server
itself.

dig @server name explicitly queries a chosen DNS server.

dig name uses the client's configured resolver path.

CoreDNS local records and upstream resolution were tested
separately.

Evidence was gathered at multiple layers instead of immediately
changing configuration.

Troubleshooting method

Used:

observe
  ↓
form hypothesis
  ↓
test one layer
  ↓
collect evidence
  ↓
move to next layer

No skill is promoted to 🟢 based on today's work alone; independent
reproduction and retention evidence ≥48 hours later are still required.

Open Kubernetes inventory issue --- parked

Do not continue this first next session unless it becomes relevant
to the current network work.

Known Service inventory includes:

default/web           NodePort    30007   no endpoints
web/my-web-service    ClusterIP            → nginx Pod
web/web               NodePort    30008   → nginx Pod

Ingress currently routes to:

web Service :80 → nginx Pod

There may be unnecessary/orphan Services, but nothing should be deleted
until we deliberately return to inventory/cleanup.

Daily Skill Progress --- 28-09-2026

╔════════════════════ DAILY SKILL PROGRESS — 28-09-2026 ════════════════════╗

                                                LEVEL
 1  Git / Repository / Governance               🟡 2
 2  Linux                                       🟡 2
 3  Server Foundation                           🟡 2
 4  Network / DNS / Ingress                     🟡 2
 5  Kubernetes Platform                         🟡 2
 6  Storage                                     🟡 2
 7  Secrets                                     🔴 0
 8  CI/CD                                       🟠 1
 9  GitOps                                      🟠 1
10  Observability                               🟠 1
11  Security                                    🟡 2
12  Developer Platform / Self-Service           🟠 1
13  Reliability / Backup / Disaster Recovery    🟠 1
14  Cloud Platform                              🔴 0
15  Hybrid / Multi-environment Architecture     🔴 0
16  Chaos / Incident Response                   🟠 1
17  Employer Portfolio / Assessment             🟠 1
18  Final Zero-to-Production Rebuild             🔴 0

TODAY'S EVIDENCE
Network / DNS:
  DHCP → client DNS configuration             ✓
  Direct CoreDNS query                        ✓
  Automatic local DNS query                   ✓
  Public/upstream DNS query                   ✓
  Layer-by-layer troubleshooting              ✓

No level promotion today.

LEVELS
🔴 0 Niet bekend    🟠 1 Herkenning    🟡 2 Begeleid
🟢 3 Zelfstandig    🔵 4 Engineer      🟣 5 Architect

🟢 requires independent evidence + transfer + reproduction ≥48h later
╚═════════════════════════════════════════════════════════════════════════════╝

Next session

Start with: open http://web.home.arpa in Brave and observe the
result.
