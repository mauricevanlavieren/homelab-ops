# Current Session — 24-09-2026

## Project
Production-Grade Platform Engineering Lab

### Current project
Project 1 — Mini Platform

### Current phase
Kubernetes foundation and first workload

---

# Session Goal

Continue from the clean Ubuntu Server foundation and build the first working Kubernetes platform.

Target architecture:

```text
Laptop
  ↓
Ubuntu Server
  ↓
k3s
  ↓
Kubernetes
  ↓
Deployment
  ↓
ReplicaSet
  ↓
Pod
  ↓
Container
  ↓
Service
  ↓
NodePort
  ↓
Browser
```

---

# 1. k3s Installation

Before installation we verified that k3s was not already installed as a normal service.

```bash
dpkg -l k3s
```

Result:

```text
dpkg-query: no packages found matching k3s
```

And:

```bash
systemctl status k3s
```

Result:

```text
Unit k3s.service could not be found.
```

We used the official k3s documentation as the primary source.

Official installation command:

```bash
curl -sfL https://get.k3s.io | sh -
```

Before executing it, the command was inspected conceptually:

```text
curl
  ↓
downloads installation script
  ↓
|
  ↓
sh -
  ↓
executes downloaded script
```

Security lesson:

`curl ... | sh` means downloaded code is executed directly.

Therefore the script was first inspected without piping it into `sh`:

```bash
curl -sfL https://get.k3s.io
```

After confirming that the response was a shell installation script, k3s was installed.

Installed release:

```text
v1.36.4+k3s1
```

Installer created among other things:

```text
/usr/local/bin/k3s
/usr/local/bin/kubectl
/usr/local/bin/crictl
/usr/local/bin/ctr

/etc/systemd/system/k3s.service
/etc/systemd/system/k3s.service.env
```

The k3s systemd service was enabled and started.

---

# 2. Verify k3s Linux Service

```bash
systemctl status k3s
```

Important evidence:

```text
Loaded: loaded
Active: active (running)
Main PID: 22944 (k3s-server)
```

Processes visible underneath the service included:

```text
k3s-server
containerd
containerd-shim-runc-v2
```

Mental model:

```text
systemd
  ↓
k3s.service
  ↓
k3s-server
  ↓
containerd
  ↓
containers
```

Important distinction:

```text
systemctl status k3s
        ↓
proves the Linux service/process is running

kubectl
        ↓
is needed to verify Kubernetes itself
```

An active k3s Linux service does NOT automatically prove that the Kubernetes node is healthy.

---

# 3. Kubernetes Node Verification

Detailed inspection:

```bash
sudo kubectl describe node
```

Node:

```text
Name:       homeserver
Role:       control-plane
InternalIP: 192.168.0.10
```

Important condition:

```text
Ready: True
Reason: KubeletReady
```

Container runtime:

```text
containerd://2.3.4-k3s1.36
```

Kubernetes/k3s version:

```text
v1.36.4+k3s1
```

Compact verification:

```bash
sudo kubectl get nodes
```

Evidence:

```text
NAME         STATUS   ROLES           VERSION
homeserver   Ready    control-plane   v1.36.4+k3s1
```

Mental model:

```text
Physical homeserver
        ↓
Ubuntu Linux
        ↓
systemd
        ↓
k3s.service
        ↓
k3s-server
        ↓
Kubernetes cluster
        ↓
Node: homeserver
        ↓
Ready
```

---

# 4. Kubernetes Components Already Present

The k3s installation automatically created several Kubernetes workloads.

Observed examples:

```text
coredns
local-path-provisioner
metrics-server
traefik
svclb-traefik
```

k3s therefore provides more than the Kubernetes binary itself. It installs a usable Kubernetes distribution with several integrated platform components.

Traefik is already present and will become relevant when Ingress is introduced.

---

# 5. Pod Mental Model

A Pod is the smallest deployable unit managed by Kubernetes.

```text
Pod
 └── Container
      └── Application
```

A Pod is NOT the same thing as a container.

A Pod can contain multiple containers, although the current workload uses one container per Pod.

---

# 6. Deployment Mental Model

Instead of creating a standalone Pod, a Deployment was used.

A Deployment represents desired state for an application workload.

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
```

Example desired state:

```text
replicas = 1
image    = nginx
```

Important correction learned:

```text
replicas: 3
```

means:

```text
Pod 1
 └── nginx container

Pod 2
 └── nginx container

Pod 3
 └── nginx container
```

It does NOT mean three containers inside one Pod.

---

# 7. Mini-Boss 2 — First Deployment

Support mode:

```text
A — Independent
```

Assignment:

Create an nginx Deployment.

Acceptance criteria:

```text
Deployment: web
Desired replicas: 1
Container image: nginx
Pod: Running
Deployment: available/healthy
```

Pod evidence:

```bash
kubectl get pods
```

Result:

```text
NAME                   READY   STATUS    RESTARTS
web-6c5f67b5f7-q9fx7   1/1     Running   0
```

Detailed evidence:

```bash
kubectl describe pod web-6c5f67b5f7-q9fx7
```

Important values:

```text
Namespace: web
Status: Running
IP: 10.42.0.9
Controlled By: ReplicaSet/web-6c5f67b5f7

Container:
nginx

Image:
nginx:1.14.2

Port:
80/TCP

Ready:
True
```

Deployment evidence:

```bash
kubectl get deployments
```

Result:

```text
NAME   READY   UP-TO-DATE   AVAILABLE
web    1/1     1            1
```

Result:

**Mini-Boss 2 PASSED independently.**

---

# 8. Namespace Decision

A separate namespace was created:

```text
web
```

The namespace was set as current context rather than placing the workload in `default`.

Reason:

Keep application resources organized and avoid unnecessarily filling the default namespace.

This was an independent design decision outside the minimum assignment requirements.

---

# 9. Mini-Boss 3 — Reconciliation

Goal:

Prove Kubernetes desired-state reconciliation instead of merely reading about it.

Initial Pod:

```text
web-6c5f67b5f7-q9fx7
```

The Pod was deliberately deleted:

```bash
kubectl delete pod web-6c5f67b5f7-q9fx7
```

Immediately afterwards:

```bash
kubectl get pods
```

Result:

```text
NAME                   READY   STATUS
web-6c5f67b5f7-sxhgl   1/1     Running
```

Old Pod:

```text
web-6c5f67b5f7-q9fx7
```

New Pod:

```text
web-6c5f67b5f7-sxhgl
```

This proved:

```text
Desired replicas = 1

Pod deleted
    ↓
Actual replicas = 0
    ↓
ReplicaSet controller detects drift
    ↓
New Pod created
    ↓
Actual replicas = 1
```

The ReplicaSet is directly responsible for maintaining the requested number of Pods.

The Deployment manages the ReplicaSet.

Result:

**Mini-Boss 3 PASSED independently.**

---

# 10. Desired State and Reconciliation

Current understanding in own words:

If a certain state of the Pods is desired, the Deployment/controller structure ensures that Kubernetes keeps moving the actual state back toward the desired state.

```text
DESIRED STATE
      ↓
controller observes
      ↓
ACTUAL STATE
      ↓
difference/drift?
      ↓
reconcile
      ↓
DESIRED ≈ ACTUAL
```

This concept has now been:

- explained
- built
- deliberately broken
- observed
- independently tested

This is strong evidence, but NOT yet sufficient for 🟢 because retention and transfer still need to be demonstrated later.

---

# 11. Why Pod IP Is Not the Application Interface

After reconciliation, the replacement Pod received:

```text
10.42.0.10
```

The earlier Pod had:

```text
10.42.0.9
```

Important lesson:

**Pods are ephemeral.**

Therefore an application should not depend directly on one specific Pod IP.

```text
Client
  ↓
10.42.0.9
  ↓
Pod disappears
  ↓
address is no longer a stable application endpoint
```

Solution:

Use a Kubernetes Service.

---

# 12. Service Mental Model

A Service provides a stable abstraction in front of Pods.

```text
Service
   ↓
selector
   ↓
Pods with matching labels
```

Current selector:

```text
app=nginx
```

Pod label:

```text
app=nginx
```

This allows the Service to find replacement Pods without depending on Pod names or Pod IPs.

With multiple replicas:

```text
             Service
                │
      ┌─────────┼─────────┐
      ↓         ↓         ↓
    Pod 1     Pod 2     Pod 3
 app=nginx  app=nginx  app=nginx
```

---

# 13. Mini-Boss 4 — ClusterIP Service

Support mode:

```text
A — Independent
```

Initial Service was created as:

```text
my-web-service
```

Evidence:

```text
TYPE:       ClusterIP
ClusterIP:  10.43.79.182
Port:       80
Selector:   app=nginx
TargetPort: 80
Endpoint:   10.42.0.10:80
```

Pod evidence:

```text
Pod IP: 10.42.0.10
Port:   80/TCP
```

This proved:

```text
Service
  ↓
10.42.0.10:80
  ↓
nginx Pod
```

However, the assignment required:

```text
Service name: web
```

The initial name:

```text
my-web-service
```

did not satisfy the acceptance criteria.

Important engineering distinction:

```text
"it works"
        ≠
"it satisfies the requirements"
```

The Service was corrected declaratively using:

```bash
kubectl apply -f service.yaml
```

Final manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Because no explicit Service type was configured, Kubernetes used the default:

```text
ClusterIP
```

Result:

**Mini-Boss 4 PASSED independently.**

---

# 14. Kubernetes Network Model Observed

Current addresses:

```text
LAN / physical network
192.168.0.x

Homeserver / Node
192.168.0.10

Kubernetes Pod network
10.42.x.x

Kubernetes Service network
10.43.x.x
```

The laptop has a route to:

```text
192.168.0.10
```

but not automatically to the Kubernetes internal Pod and Service networks.

Therefore the ClusterIP cannot simply be used as the external application address from the laptop.

---

# 15. NodePort Mental Model

Problem:

Expose the Service outside the Kubernetes internal network.

Solution introduced:

```text
NodePort
```

Mental model:

```text
Laptop
   ↓
Node IP : NodePort
   ↓
Kubernetes Service
   ↓
Pod
   ↓
Application
```

Important distinction:

```text
Node IP
192.168.0.10
```

is reachable from the laptop.

The NodePort provides an entry point on the node.

The Service then routes traffic to matching Pods.

---

# 16. Mini-Boss 5 — External Reachability

The existing Service `web` was changed to:

```text
Type: NodePort
```

Evidence:

```bash
kubectl get svc -n web
```

Result:

```text
NAME   TYPE       CLUSTER-IP    PORT(S)
web    NodePort   10.43.33.30   80:30008/TCP
```

Therefore:

```text
Service port: 80
NodePort:     30008
```

Server-side test:

```bash
curl -I http://localhost:30008
```

Result:

```text
HTTP/1.1 200 OK
Server: nginx/1.14.2
```

This proved:

```text
NodePort
   ↓
Service
   ↓
Pod
   ↓
nginx
```

was functioning on the node.

---

# 17. Troubleshooting External Access

Initially the application appeared unreachable from the laptop.

Troubleshooting was performed layer by layer.

First:

```text
Laptop → ping → 192.168.0.10
```

worked.

This proved IP-level reachability to the homeserver.

However:

```text
ping
```

does NOT prove that TCP port `30008` works.

A direct HTTP test was then performed from the laptop:

```bash
curl -I http://192.168.0.10:30008
```

Result:

```text
HTTP/1.1 200 OK
Server: nginx/1.14.2
```

This proved end-to-end HTTP connectivity from the laptop through Kubernetes to nginx.

The browser still appeared broken.

Evidence comparison found that the browser was using:

```text
192.168.0.10:3000
```

instead of:

```text
192.168.0.10:30008
```

After using:

```text
http://192.168.0.10:30008
```

the nginx page loaded successfully.

Troubleshooting lesson:

Do not change the platform immediately when one client appears broken.

Compare evidence between layers first.

```text
Browser :3000    → failed
curl    :30008   → HTTP 200
```

The problem was the client request using the wrong port, not Kubernetes.

Result:

**Mini-Boss 5 PASSED.**

Support:

Mostly independent, with one small hint during TCP/HTTP testing.

---

# 18. Current End-to-End Architecture

The following chain is now WORKING and VERIFIED:

```text
Laptop / Brave
192.168.0.x
       │
       │ HTTP
       ▼
192.168.0.10:30008
       │
       ▼
Kubernetes NodePort
       │
       ▼
Service: web
ClusterIP: 10.43.33.30
Port: 80
Selector: app=nginx
       │
       ▼
Pod
10.42.0.10:80
       │
       ▼
nginx container
       │
       ▼
HTTP 200 OK
```

Controller chain:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
```

Network chain:

```text
Laptop
    ↓
Node IP
    ↓
NodePort
    ↓
Service
    ↓
Pod IP
    ↓
Container port
```

---

# 19. Important Concepts Practiced Today

## Linux / Platform boundary

```text
systemd
   ↓
k3s.service
   ↓
k3s-server
   ↓
Kubernetes
```

## Kubernetes health

```text
k3s active
```

does not automatically mean:

```text
Node Ready
```

Health must be checked at the correct layer.

## Desired state

Kubernetes controllers continuously compare desired state with actual state.

## Reconciliation

Deleting a managed Pod caused Kubernetes to automatically create a replacement.

## Ephemeral Pods

Pod identity and Pod IP should not be treated as stable application endpoints.

## Labels and selectors

Services discover Pods through labels/selectors.

## Service

Provides a stable abstraction in front of changing Pods.

## ClusterIP

Internal Kubernetes Service address.

## NodePort

Makes a Service reachable through a port on a Kubernetes node.

## Evidence-driven troubleshooting

Test each layer independently before changing configuration.

---

# 20. Evidence Produced Today

- Official k3s installation source used.
- k3s install script inspected before execution.
- k3s installed on clean Ubuntu Server.
- k3s systemd service verified.
- Kubernetes node verified Ready.
- First Deployment built independently.
- Deployment health verified.
- Pod inspected.
- Namespace `web` used.
- Pod deliberately deleted.
- Kubernetes reconciliation independently demonstrated.
- ClusterIP Service created.
- Service selector → Pod endpoint relationship verified.
- Service requirement mismatch detected and corrected.
- Service manifest created declaratively.
- NodePort configured.
- nginx tested locally through NodePort.
- laptop → homeserver connectivity tested.
- laptop → NodePort → Service → Pod → nginx tested with curl.
- browser access verified.
- incorrect browser port diagnosed through evidence comparison.

---

# 21. Skill Evidence Status

No skill is promoted to 🟢 solely because today's work succeeded.

Promotion to 🟢 still requires:

- independent evidence
- reproduction in another practical context
- retention evidence at least 48 hours later
- explanation of why it works

Today's Kubernetes work provides strong independent evidence that can later be used toward promotion.

| Skill | Level |
|---|---|
| Git / Repository / Governance | 🟡 2 |
| Linux | 🟡 2 |
| Server Foundation | 🟡 2 |
| Network / DNS / Ingress | 🟡 2 |
| Kubernetes Platform | 🟡 2 |
| Storage | 🟡 2 |
| Secrets | 🔴 0 |
| CI/CD | 🟠 1 |
| GitOps | 🟠 1 |
| Observability | 🟠 1 |
| Security | 🟡 2 |
| Developer Platform / Self-Service | 🟠 1 |
| Reliability / Backup / DR | 🟠 1 |
| Cloud Platform | 🔴 0 |
| Hybrid / Multi-environment | 🔴 0 |
| Chaos / Incident Response | 🟠 1 |
| Employer Portfolio / Assessment | 🟠 1 |
| Final Zero-to-Production Rebuild | 🔴 0 |

---

# 22. Next Architecture Problem

NodePort works:

```text
http://192.168.0.10:30008
```

But this is not how applications should ultimately be exposed.

Target:

```text
http://web.home.arpa
```

Future architecture:

```text
Browser
   │
   │ web.home.arpa
   ▼
DNS
   │
   │ name → IP
   ▼
192.168.0.10:80
   │
   ▼
Ingress Controller
Traefik
   │
   │ host/path routing
   ▼
Service: web
   │
   ▼
Pod
   │
   ▼
nginx
```

Important distinction:

```text
DNS
name → IP address

Ingress
HTTP request → correct Kubernetes Service
```

k3s already installed Traefik, so an Ingress Controller appears to already exist.

This must still be investigated and verified rather than assumed.

---

# EXACT STOPPING POINT

The last question before stopping was:

> If `web.home.arpa` is entered into the browser, what must happen first before the request can reach Traefik?

Resume here next session.

Do NOT give the answer first.

Let Maurice reason from the current network chain.

---

# Next Session

Continue with:

```text
DNS
 ↓
Ingress / Traefik
 ↓
Service
 ↓
Pod
```

Likely build target:

```text
web.home.arpa
      ↓
192.168.0.10
      ↓
Traefik :80
      ↓
Ingress rule
      ↓
Service web :80
      ↓
nginx Pod
```

Keep build-first approach.

Do not introduce unnecessary production complexity yet.

---

# DAILY SKILL PROGRESS — 24-09-2026

No percentages are assigned yet because objective percentage criteria have not yet been defined.

```text
╔════════════════════ DAILY SKILL PROGRESS — 24-09-2026 ════════════════════╗

                                      LEVEL       TODAY
 1  Git / Repository / Governance     🟡 2         —
 2  Linux                              🟡 2         ▲
 3  Server Foundation                  🟡 2         ▲
 4  Network / DNS / Ingress            🟡 2         ▲
 5  Kubernetes Platform                🟡 2         ▲▲
 6  Storage                            🟡 2         —
 7  Secrets                            🔴 0         —
 8  CI/CD                              🟠 1         —
 9  GitOps                             🟠 1         —
10  Observability                      🟠 1         —
11  Security                           🟡 2         ▲
12  Developer Platform / Self-Service  🟠 1         —
13  Reliability / Backup / DR          🟠 1         ▲
14  Cloud Platform                     🔴 0         —
15  Hybrid / Multi-environment         🔴 0         —
16  Chaos / Incident Response          🟠 1         ▲
17  Employer Portfolio / Assessment    🟠 1         ▲
18  Final Zero-to-Production Rebuild   🔴 0         —

TODAY'S STRONGEST EVIDENCE

Kubernetes:
  Deployment → ReplicaSet → Pod → Container        ✓
  Desired state / reconciliation                   ✓
  Service selector → Pod                           ✓
  ClusterIP                                        ✓
  NodePort                                         ✓
  End-to-end browser access                        ✓

Troubleshooting:
  layer-by-layer investigation                     ✓
  ping vs HTTP distinction                         ✓
  wrong client port identified                     ✓

LEVELS
🔴 0 Niet bekend    🟠 1 Herkenning    🟡 2 Begeleid
🟢 3 Zelfstandig    🔵 4 Engineer      🟣 5 Architect

🟢 requires:
independent evidence
+ transfer to another situation
+ reproduction ≥48h later
+ explanation of why it works

═══════════════════════════════════════════════════════════════════════════════
```

---

# Session Summary

Today the clean Ubuntu homeserver became a functioning Kubernetes platform.

The most important achievement was not simply installing k3s.

The complete working chain was built, tested, deliberately changed and troubleshot:

```text
Laptop
  ↓
Ubuntu Server
  ↓
k3s
  ↓
Kubernetes Node
  ↓
Deployment
  ↓
ReplicaSet
  ↓
Pod
  ↓
Container
  ↑
Service
  ↑
NodePort
  ↑
Browser
```

Next step:

```text
DNS → Ingress → Service → Pod
```
