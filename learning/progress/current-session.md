# Current Session — 2026-10-06

## Project

Project 01 — Mini Platform

## Current Status

**PROJECT 01 COMPLETED**

Project 01 has progressed through:

1. Server Foundation
2. Kubernetes Foundation
3. Networking / DNS / Ingress
4. Source of Truth
5. Break / Fix
6. Review & Documentation
7. Assessment
8. Project Gate

The practical assessment has been completed.

Assessment evidence:

`learning/assessments/project-01-assessment.md`

---

# What We Built

The current mini platform consists of:

```text
Laptop / Browser
       │
       │ DNS
       ▼
CoreDNS
192.168.0.10:53
       │
       │ web.home.arpa → 192.168.0.10
       ▼
Homeserver
192.168.0.10
       │
       │ HTTP :80
       ▼
Traefik
       │
       ▼
Ingress
host: web.home.arpa
       │
       ▼
Service
web:80
ClusterIP
       │
       ▼
nginx Pod
```

Platform:

- Ubuntu Server
- k3s
- containerd
- Kubernetes
- Traefik
- CoreDNS for homelab DNS
- nginx test workload
- GitHub repository as configuration source of truth

---

# Repository Structure

```text
homelab-ops/
├── docs/
│   └── architecture/
│       ├── mini-platform-architecture.md
│       └── network-architecture.md
│
├── kubernetes/
│   ├── apps/
│   │   └── web/
│   │       ├── deployment.yaml
│   │       ├── ingress.yaml
│   │       └── service.yaml
│   │
│   └── infrastructure/
│       └── dns/
│           ├── configmap.yaml
│           ├── deployment.yaml
│           └── service.yaml
│
└── learning/
    ├── assessments/
    │   └── project-01-assessment.md
    │
    ├── incidents/
    │   └── break_fix_mini-platform.md
    │
    ├── progress/
    │   ├── current-session.md
    │   ├── pre-rebuild-baseline.md
    │   └── server-foundation.md
    │
    └── projects/
        └── project-01-mini-platform.md
```

---

# Project 01 Architecture

## Application Request Path

```text
Browser
   │
   │ web.home.arpa
   ▼
DNS
   │
   │ 192.168.0.10
   ▼
Traefik :80
   │
   ▼
Ingress
   │
   ▼
Service web:80
   │
   ▼
Endpoint
   │
   ▼
nginx Pod
```

Important mental model:

```text
Name → Machine → Application → Instance
```

DNS:

```text
Name → Machine
```

Traefik / Ingress:

```text
Machine → Application
```

Service:

```text
Application → Pod / Instance
```

This complete chain is understood when visible but is not yet considered independently retained.

It should return naturally in future projects rather than being memorized mechanically.

---

# Source of Truth

GitHub is the technical source of truth for the platform configuration currently stored in the repository.

Current relationship:

```text
Git
 │
 │ desired configuration
 ▼
Kubernetes manifests
```

However, there is currently **no GitOps controller**.

Therefore:

```text
Git
   X
   │ automatic reconciliation
   ▼
Kubernetes
```

A manual change in Kubernetes can therefore create:

**configuration drift**

Example:

```text
Git:
replicas = 1

Live Kubernetes:
replicas = 2

Result:
DRIFT
```

Kubernetes itself only reconciles against the desired state stored inside Kubernetes.

Future GitOps will add:

```text
Git
 │
 ▼
GitOps Controller
 │
 ▼
Kubernetes Deployment
 │
 ▼
ReplicaSet
 │
 ▼
Pods
```

---

# Break / Fix Evidence

Project 01 deliberately introduced failures.

## Service Selector Failure

The Service selector was changed so it no longer matched the nginx Pod.

Observed result:

```text
Service
   │
   X
   │
Endpoints: none
```

The problem was found through inspection and repaired using the desired configuration.

## Ingress Drift

The live Ingress hostname was deliberately changed.

Git still contained:

`web.home.arpa`

Live Kubernetes contained:

`broken.home.arpa`

This demonstrated configuration drift between Git and the cluster.

The known-good Git configuration was used for recovery.

## Deployment Failure

The Deployment was deliberately scaled to:

`replicas = 0`

Observed:

```text
web.home.arpa
      │
      ▼
Traefik
      │
      ▼
Ingress
      │
      ▼
Service
      │
      X
      │
no endpoints
```

HTTP result:

`503 Service Unavailable`

Git still specified:

`replicas = 1`

The Deployment manifest was reapplied.

Result:

- Pod recreated
- Endpoint returned
- HTTP 200 returned
- browser application recovered

Incident evidence:

`learning/incidents/break_fix_mini-platform.md`

---

# Platform Cleanup

Project 01 also included cleanup of unnecessary resources.

## NodePort Removal

The web application originally exposed:

`NodePort 30008`

The normal request path already used:

```text
Client
  ↓
Traefik
  ↓
Ingress
  ↓
Service
  ↓
Pod
```

The NodePort was therefore unnecessary.

The Service was changed to:

`ClusterIP`

Current intended Service:

```text
web
type: ClusterIP
port: 80/TCP
```

This reduces unnecessary external exposure and simplifies the architecture.

## Orphan Service Removal

An old Service existed in the `default` namespace:

```text
default/web
NodePort 30007
```

Investigation showed:

- it had no endpoints;
- it was not part of the active request path;
- it was not present in the Kubernetes desired-state manifests;
- deleting it did not affect `web.home.arpa`.

The orphaned Service was removed.

---

# Troubleshooting Model

The main troubleshooting principle practiced during Project 01:

```text
Observe
   ↓
Form hypothesis
   ↓
Identify layer
   ↓
Collect evidence
   ↓
Move one layer deeper
   ↓
Find failure boundary
   ↓
Recover
   ↓
Verify
```

For the current application:

```text
Browser
   ↓
DNS
   ↓
TCP / Entry Point
   ↓
Traefik
   ↓
Ingress
   ↓
Service
   ↓
Endpoint / EndpointSlice
   ↓
Pod
```

Do not immediately change configuration.

First determine:

**Where does the expected relationship stop working?**

---

# Project 01 Assessment

Assessment completed:

`learning/assessments/project-01-assessment.md`

## Strong Evidence

Project 01 produced strong evidence for:

- Kubernetes desired-state reasoning
- Kubernetes reconciliation
- Service endpoints
- configuration drift
- Git as source of truth
- basic network routing
- evidence-first troubleshooting
- orphan resource investigation
- recovery from deliberately introduced failures

---

# Carry-Forward Learning Gaps

These are not reasons to repeat Project 01.

They must return naturally in Project 02 and later projects.

## Git

Reinforce:

```text
git diff
```

versus:

```text
git diff --staged
```

Current model:

```text
Working Tree
     │
     │ git add
     ▼
Staging Area
     │
     │ git commit
     ▼
Local Repository
     │
     │ git push
     ▼
GitHub
```

The staged-diff distinction was correctly applied again after the assessment, but retention and transfer still need future evidence.

## Linux

Reinforce:

- systemd service investigation
- service status
- journal/log investigation
- historical evidence

The research strategy:

```text
problem
  ↓
domain
  ↓
manager/tool
  ↓
help/documentation
  ↓
evidence
```

is developing correctly.

## Kubernetes

Reinforce the responsibility boundary:

```text
Deployment
    ↓
desired rollout / replica configuration
    ↓
ReplicaSet
    ↓
maintains required Pods
    ↓
Pod
```

Also reinforce:

- ClusterIP
- NodePort
- exposure
- attack surface
- namespace boundaries

## Networking

Continue reinforcing:

```text
DNS
 ↓
Traefik
 ↓
Ingress
 ↓
Service
 ↓
Endpoint
 ↓
Pod
```

Do not force memorization.

Rebuild this mental model through future practical use.

---

# Recovery Limitation Discovered

Project 01 configuration in Git is not yet sufficient to rebuild the entire platform from a completely new physical server.

Current recovery capability:

```text
New server
   │
   ├── Ubuntu installation        MANUAL
   │
   ├── server configuration       MANUAL
   │
   ├── networking                 MANUAL / PARTLY DOCUMENTED
   │
   ├── k3s installation           MANUAL
   │
   ▼
Kubernetes available
   │
   ▼
Git manifests
   │
   ▼
Kubernetes workloads recoverable
```

Git currently contains important desired-state configuration, but not complete server/platform provisioning.

Future projects must progressively improve:

- reproducibility
- server provisioning
- automation
- backup
- disaster recovery
- GitOps
- infrastructure as code

---

# Project 01 Gate Decision

**PASS — Proceed to Project 02**

Project 01 does not need to be repeated.

Project 02 may increase in scope, but should continue exercising weaker Project 01 skills.

Difficulty should increase through:

- more responsibility
- more components
- less guidance
- repeated use of existing skills
- new failure scenarios
- stronger evidence requirements

---

# Skill Passport — Project 01 Exit State

No skill is promoted to 🟢 solely because Project 01 was completed.

A 🟢 level requires:

- independent application;
- transfer to another practical situation;
- reproduction after at least 48 hours;
- ability to explain why it works.

Current conservative levels:

```text
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
16  Chaos / Incident Response                   🟡 2
17  Employer Portfolio / Assessment             🟡 2
18  Final Zero-to-Production Rebuild             🔴 0
```

Changes since the previous session:

```text
Chaos / Incident Response:
🟠 1 → 🟡 2

Employer Portfolio / Assessment:
🟠 1 → 🟡 2
```

Reason:

Project 01 now contains guided practical evidence for deliberate failure, investigation, recovery, incident documentation, architecture documentation and formal assessment.

This is sufficient for **guided level**, but not yet independent level.

---

# Important Project 01 Evidence

Git commits include:

```text
ee29458  progress project 01 mini-platform
f448edc  break fix mini-platform
2e24427  manifest cleanup. removed nodeport from apps/service
6bf065f  mini platform architecture
db32cdc  project-01-assessment
```

Evidence exists for:

- architecture
- desired state
- troubleshooting
- deliberate failure
- recovery
- cleanup
- Git workflow
- assessment

---

# Exact Stopping Point

Project 01 is complete.

The next activity is **not another Project 01 exercise**.

Next session begins with:

# Project 02 — Architecture / Requirements

Before selecting technology:

```text
Problem
  ↓
Requirements
  ↓
Options
  ↓
Trade-offs
  ↓
Architecture choice
  ↓
Build
```

Project 02 should build a larger real platform capability while reusing the foundations from Project 01.

---

# Next Session

Start with:

**Design Project 02 based on the Project 01 assessment and Skill Passport.**

The first question should be:

**What real platform problem will Project 02 solve?**
