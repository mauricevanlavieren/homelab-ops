# Mini Platform Architecture

## Purpose

Project 01 builds a small Kubernetes platform on a single physical server.

The goal is not production high availability, but to build and understand the basic platform components, their relationships, and the workflow from Git desired state to a working application.

---

## Platform Overview

```text
                    LAPTOP / CLIENT
                          │
            ┌─────────────┴─────────────┐
            │                           │
         DNS query                 HTTP request
            │                           │
            ▼                           ▼
     192.168.0.10:53             192.168.0.10:80
            │                           │
         CoreDNS                     Traefik
            │                           │
     web.home.arpa                     ▼
       → .0.10                      Ingress
                                        │
                                        ▼
                                  Service web
                                   ClusterIP
                                        │
                                 selector app=nginx
                                        │
                                        ▼
                                   nginx Pod
```

---

## Host

Physical homeserver:

- OS: Ubuntu 24.04
- Server IP: `192.168.0.10`
- Kubernetes distribution: k3s
- Container runtime: containerd
- Kubernetes node: `homeserver`

The platform currently consists of one Kubernetes node.

---

## Application

Namespace:

`web`

The application consists of:

```text
Deployment
    ↓
ReplicaSet
    ↓
nginx Pod
```

The Deployment defines the desired nginx workload.

Current desired replica count:

`1`

The Pod is selected using:

`app=nginx`

---

## Application Networking

External HTTP traffic enters the platform through Traefik.

```text
web.home.arpa
      ↓
192.168.0.10:80
      ↓
Traefik
      ↓
Ingress
      ↓
Service web:80
      ↓
nginx Pod:80
```

The `web` Service is a `ClusterIP` Service.

No NodePort is required because external HTTP traffic enters through Traefik and the Ingress.

This reduces unnecessary external exposure.

---

## DNS

Namespace:

`homelab-dns`

CoreDNS provides DNS resolution for the homelab.

```text
web.home.arpa
      ↓
192.168.0.10:53
      ↓
CoreDNS LoadBalancer Service
      ↓
CoreDNS Pod
      ↓
Corefile / ConfigMap
      ↓
web.home.arpa = 192.168.0.10
```

CoreDNS is exposed to the LAN using a Kubernetes `LoadBalancer` Service.

DNS uses port `53` over TCP and UDP.

---

## Source of Truth

GitHub repository:

`homelab-ops`

Relevant desired state:

```text
kubernetes/
├── apps/
│   └── web/
│       ├── deployment.yaml
│       ├── ingress.yaml
│       └── service.yaml
└── infrastructure/
    └── dns/
        ├── configmap.yaml
        ├── deployment.yaml
        └── service.yaml
```

Git contains the intended Kubernetes configuration.

The live cluster can be compared with Git to detect configuration drift.

---

## Reconciliation

Kubernetes maintains workload desired state.

Example:

```text
Deployment: replicas = 1
          ↓
ReplicaSet
          ↓
Pod deleted
          ↓
ReplicaSet creates replacement Pod
```

This behavior was tested during Project 01.

---

## Troubleshooting Model

The platform can be investigated layer by layer:

```text
DNS
 ↓
Network / port
 ↓
Traefik
 ↓
Ingress
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod
 ↓
Application
```

Evidence should be collected before changing configuration.

---

## Cleanup Performed

During the architecture review:

- `web` Service was changed from `NodePort` to `ClusterIP`.
- unnecessary NodePort `30008` was removed.
- orphaned `default/web` NodePort Service on port `30007` was identified.
- the orphaned Service had no endpoints.
- it was not part of the Git Kubernetes desired state.
- the orphaned Service was removed.
- `web.home.arpa` continued working after removal.

---

## Known Limitations

Project 01 intentionally remains a small learning platform.

Current limitations include:

- single Kubernetes node
- single physical server
- no high availability
- CoreDNS is a single point of failure
- no GitOps controller
- no automated CI/CD deployment pipeline
- no production secrets management
- no automated backup/disaster recovery
- no cloud or hybrid environment

These limitations are intentionally addressed in later projects.

---

## Project 01 Architecture Principle

The platform separates:

```text
Git              → desired configuration
Kubernetes       → workload reconciliation
DNS              → name resolution
Traefik/Ingress  → application routing
Service          → stable access to Pods
Pod              → workload instance
nginx            → application
```

Project 01 provides the foundation that later projects will make more automated, reliable, secure, observable, and portable.
