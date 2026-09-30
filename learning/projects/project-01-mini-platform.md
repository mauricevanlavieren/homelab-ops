# Project 01 — Mini Platform

## Status

**Status:** IN PROGRESS  
**Current phase:** Phase 5 — Break / Fix  
**Current step:** Create a controlled platform failure  
**Next milestone:** Diagnose and recover the platform using evidence

---

# 1. Project Goal

Build a small but complete Kubernetes platform from a clean Ubuntu Server.

The purpose is not only to make the platform work.

The project must demonstrate the complete engineering cycle:

**design → build → test → document → version → break → troubleshoot → recover → assess**

At the end of this project the platform should be reproducible from documented and version-controlled configuration.

---

# 2. Target Architecture

```text
Laptop
  │
  │ Git / SSH / HTTP / DNS
  ▼
GitHub
  │
  │ Source of Truth
  ▼
Ubuntu Homeserver
192.168.0.10
  │
  ▼
k3s
  │
  ├── CoreDNS
  │      │
  │      └── web.home.arpa → 192.168.0.10
  │
  ├── Traefik
  │      │
  │      ▼
  │    Ingress
  │      │
  │      ▼
  │    Service
  │      │
  │      ▼
  │    nginx Pod
  │
  └── containerd
```

## User Request Path

```text
Browser
   ↓
DNS
   ↓
192.168.0.10:80
   ↓
Traefik
   ↓
Ingress
   ↓
Service
   ↓
nginx Pod
   ↓
HTML response
```

---

# 3. Phase 1 — Server Foundation

- [x] Clean Ubuntu Server installation
- [x] Define storage layout
- [x] Configure hostname
- [x] Configure static server IP
- [x] Verify network connectivity
- [x] Configure SSH access
- [x] Document server foundation
- [x] Store foundation documentation in Git

Result:

```text
Ubuntu homeserver
IP: 192.168.0.10
SSH reachable
Network stable
```

---

# 4. Phase 2 — Kubernetes Foundation

- [x] Install k3s
- [x] Verify k3s service
- [x] Verify Kubernetes node
- [x] Inspect system Pods
- [x] Understand Deployment → ReplicaSet → Pod
- [x] Deploy first nginx workload
- [x] Delete Pod and observe reconciliation

Result:

```text
k3s
 ↓
Deployment
 ↓
ReplicaSet
 ↓
nginx Pod
```

---

# 5. Phase 3 — Networking

## Service

- [x] Create Service
- [x] Test NodePort
- [x] Reach nginx from laptop

## DNS

- [x] Design local DNS architecture
- [x] Deploy separate CoreDNS workload
- [x] Create local `web.home.arpa` record
- [x] Test CoreDNS directly
- [x] Configure DHCP to distribute DNS
- [x] Verify laptop receives DNS automatically
- [x] Verify local DNS resolution
- [x] Verify public DNS resolution

## Ingress

- [x] Identify Traefik as Ingress Controller
- [x] Create Ingress
- [x] Route `web.home.arpa` through Traefik
- [x] Verify HTTP on port 80
- [x] Verify nginx response with curl
- [x] Verify `http://web.home.arpa` in Brave

Result:

```text
web.home.arpa
      ↓
     DNS
      ↓
192.168.0.10
      ↓
   Traefik
      ↓
   Ingress
      ↓
   Service
      ↓
  nginx Pod
```

---

# 6. Phase 4 — Source of Truth

## COMPLETED

### Problem

The Kubernetes manifests originally existed only on the homeserver.

The platform worked, but the Kubernetes desired state was not yet safely stored in the Git repository.

### Goal

```text
GitHub
   ↓
homelab-ops
   ↓
Kubernetes desired state
```

### Tasks

- [x] Inspect current Kubernetes manifests
- [x] Determine correct repository structure
- [x] Move/copy app manifests into `homelab-ops`
- [x] Move/copy infrastructure manifests into `homelab-ops`
- [x] Review manifests
- [x] Compare Git configuration with live Kubernetes state
- [x] Check `git diff`
- [x] Commit desired state
- [x] Push to GitHub
- [x] Verify GitHub contains the reproducible baseline

### Repository Structure

```text
homelab-ops/
├── kubernetes/
│   ├── apps/
│   │   └── web/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── ingress.yaml
│   │
│   └── infrastructure/
│       └── dns/
│           ├── configmap.yaml
│           ├── deployment.yaml
│           └── service.yaml
│
├── learning/
│   ├── progress/
│   ├── projects/
│   ├── assessments/
│   └── skill-passport.md
│
└── docs/
    └── architecture/
```

### Desired State Verification

The Git configuration was manually compared with the live Kubernetes state.

Verified resources:

```text
web namespace
├── Deployment web
├── Service web
└── Ingress web

homelab-dns namespace
├── Deployment coredns
├── Service coredns
└── ConfigMap coredns-config
```

The managed CoreDNS configuration in Git matches the relevant live Kubernetes configuration.

During verification an additional Service was discovered:

```text
web/my-web-service
```

The Service was investigated before removal.

The active Ingress routes to:

```text
Ingress web
   ↓
Service web
   ↓
Pods matching app=nginx
```

`my-web-service` was not part of the active request path and was not present in the Git desired state.

The Service was deleted and the application was tested again successfully.

This provided practical evidence of identifying and removing live-state drift.

### Current Source of Truth Model

GitOps is intentionally **not** introduced yet.

Current model:

```text
Git
 ↓
human applies configuration
 ↓
Kubernetes
```

Future model:

```text
Git
 ↓
GitOps Controller
 ↓
Kubernetes
```

---

# 7. Phase 5 — Break / Fix

## CURRENT PHASE

### Goal

Prove that the platform can be troubleshot and recovered using evidence rather than guessing.

### Tasks

- [ ] Create controlled failure
- [ ] Observe symptoms
- [ ] Form hypothesis
- [ ] Test network layer
- [ ] Test DNS layer
- [ ] Test TCP/HTTP layer
- [ ] Test Kubernetes layer
- [ ] Identify root cause
- [ ] Recover from known configuration
- [ ] Verify end-to-end recovery
- [ ] Explain why the failure happened

Troubleshooting model:

```text
observe
   ↓
hypothesis
   ↓
test
   ↓
evidence
   ↓
isolate layer
   ↓
root cause
   ↓
recover
   ↓
verify
```

The failure must be controlled and recoverable.

The purpose is not simply to repair the platform.

The purpose is to demonstrate the ability to determine:

```text
What is the symptom?
        ↓
Which layer could cause it?
        ↓
What evidence can prove or disprove that?
        ↓
Where is the actual failure?
        ↓
How can the known desired state be used for recovery?
```

---

# 8. Phase 6 — Review & Documentation

- [ ] Review final repository structure
- [ ] Review Kubernetes manifests
- [ ] Verify architecture documentation matches reality
- [ ] Record known limitations
- [ ] Record technical debt
- [ ] Record evidence
- [ ] Create final as-built overview

## Known Issue to Investigate

Laptop receives both:

```text
192.168.0.10
192.168.0.1
```

as DNS servers although Secondary DNS is blank in the router configuration.

The source of `192.168.0.1` still needs to be investigated.

## Kubernetes Cleanup

During Phase 4 an orphaned Service was discovered:

```text
web/my-web-service
```

It was investigated and safely removed.

Before Project 1 is closed, the cluster-wide resource inventory should be reviewed once more for additional unmanaged or obsolete resources.

---

# 9. Phase 7 — Project Assessment

Assessment areas:

- [ ] Linux fundamentals
- [ ] Server foundation
- [ ] Networking
- [ ] DNS
- [ ] Kubernetes concepts
- [ ] Kubernetes networking
- [ ] Git / GitHub
- [ ] Troubleshooting
- [ ] Recovery
- [ ] Security awareness
- [ ] Architecture explanation

During the assessment:

**No hints unless explicitly requested.**

Working configuration alone is not sufficient.

The architecture and troubleshooting decisions must also be explainable.

---

# 10. Phase 8 — Project Gate

Before Project 2:

- [ ] Review project evidence
- [ ] Update Skill Passport
- [ ] Identify skills needing reinforcement
- [ ] Identify retention tests
- [ ] Record project lessons
- [ ] Complete Project 1 review
- [ ] Approve progression to Project 2

Skill promotion to 🟢 requires:

- independent application
- application in another practical situation
- reproduction after at least 48 hours
- explanation of why it works

A skill is not promoted simply because it was successfully used once.

---

# 11. Out of Scope for Project 1

These are intentionally postponed:

- GitOps controller
- full CI/CD pipeline
- Prometheus / Grafana platform
- production secrets management
- advanced RBAC
- high availability
- multi-node Kubernetes
- automated backups / disaster recovery
- Infrastructure as Code
- Terraform
- cloud Kubernetes
- hybrid architecture
- developer self-service platform

These will be introduced in later projects when they solve a real problem.

---

# 12. Current Position

```text
Server Foundation       [DONE]
        ↓
Kubernetes Foundation   [DONE]
        ↓
Networking / DNS        [DONE]
        ↓
Ingress                 [DONE]
        ↓
Source of Truth         [DONE]
        ↓
BREAK / FIX             [CURRENT]
        ↓
Review & Documentation
        ↓
Assessment
        ↓
Project Gate
        ↓
Project 1 Complete
```

---

# 13. Evidence So Far

Evidence produced during Project 1 includes:

```text
Clean Ubuntu Server installation
Static network configuration
Server foundation documentation
Working k3s installation
Kubernetes Deployment / ReplicaSet / Pod
Pod reconciliation test
NodePort connectivity
Local CoreDNS deployment
DHCP-delivered DNS configuration
Local and public DNS resolution
Traefik Ingress
End-to-end HTTP request path
Kubernetes manifests stored in Git
GitHub desired-state baseline
Manual Git ↔ live Kubernetes comparison
Identification of live-state drift
Controlled removal of orphaned Service
Successful application verification after cleanup
```

GitHub remains the technical source of truth for project configuration and evidence.

---

# 14. Next Step

Create a controlled failure in the working platform.

The failure will be used to test the complete troubleshooting workflow:

```text
observe
   ↓
hypothesis
   ↓
investigate
   ↓
isolate
   ↓
root cause
   ↓
recover
   ↓
verify
```

The troubleshooting exercise must be completed using evidence rather than random configuration changes.
