# Project 01 — Mini Platform

## Status

**Status:** IN PROGRESS  
**Current phase:** Phase 4 — Source of Truth  
**Current step:** Move Kubernetes manifests into `homelab-ops`  
**Next milestone:** Kubernetes desired state stored in GitHub

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

## CURRENT PHASE

### Problem

The Kubernetes manifests currently exist on the homeserver.

The platform works, but the Kubernetes desired state is not yet safely stored in the Git repository.

### Goal

```text
GitHub
   ↓
homelab-ops
   ↓
Kubernetes desired state
```

### Tasks

- [ ] Inspect current Kubernetes manifests
- [ ] Determine correct repository structure
- [ ] Move/copy app manifests into `homelab-ops`
- [ ] Move/copy infrastructure manifests into `homelab-ops`
- [ ] Review manifests
- [ ] Compare Git configuration with live Kubernetes state
- [ ] Check `git diff`
- [ ] Commit desired state
- [ ] Push to GitHub
- [ ] Verify GitHub contains the reproducible baseline

### Expected Repository Direction

```text
homelab-ops/
├── apps/
│   └── web/
│
├── infrastructure/
│   └── dns/
│
└── docs/
    ├── architecture/
    ├── progress/
    └── projects/
```

GitOps is intentionally **not** introduced yet.

For now:

```text
Git
 ↓
human applies configuration
 ↓
Kubernetes
```

Later:

```text
Git
 ↓
GitOps Controller
 ↓
Kubernetes
```

---

# 7. Phase 5 — Break / Fix

## Goal

Prove that the platform can be troubleshot and recovered using evidence rather than guessing.

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

## Known Kubernetes Cleanup Issue

Current Service inventory contains potentially unnecessary/orphaned Services.

These must be investigated before deleting anything.

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
SOURCE OF TRUTH         [CURRENT]
        ↓
Break / Fix
        ↓
Review
        ↓
Assessment
        ↓
Project 1 Complete
```

---

# 13. Next Step

Inspect the existing Kubernetes manifests on the homeserver and design their permanent structure inside `homelab-ops` before copying anything.
