# Pre-Rebuild Baseline

## Doel

Dit document legt de toestand van mijn homelab vast vóór de geplande clean rebuild.

De huidige omgeving wordt bewust verwijderd zodat ik het platform vanaf nul opnieuw kan ontwerpen, bouwen, testen en documenteren.

Deze baseline dient als:
- technisch bewijs van de huidige situatie;
- startpunt van mijn Platform Engineering apprenticeship;
- referentie om de nieuwe omgeving later mee te vergelijken.

## 1. Server Foundation

### Operating System

- OS: Ubuntu 24.04.4 LTS
- Kernel: 6.8.0-138-generic
- Architectuur: x86_64

## 2. Storage

### Fysieke disks

- `/dev/sda`: 232.9 GiB
- `/dev/sdb`: 111.8 GiB

### Systeemdisk `/dev/sda`

- `sda1`: 1 GiB, FAT32, gemount op `/boot/efi`
- `sda2`: 2 GiB, ext4, gemount op `/boot`
- `sda3`: 229.8 GiB, LVM physical volume
- Logical Volume `ubuntu--vg-ubuntu--lv`: 100 GiB, ext4, gemount op `/`
- Root filesystem: ongeveer 32 GiB gebruikt en 62 GiB beschikbaar

### Tweede disk `/dev/sdb`

- `sdb1`: 111.8 GiB
- Filesystem: NTFS
- Was standaard niet gemount
- Voor inspectie handmatig read-only gemount op `/mnt/sdb1`
- Bevatte bestaande Windows-gerelateerde bestanden en oude projectdata

## 3. Network

### Fysiek LAN

- Interface: `enp3s0`
- Status: UP
- IPv4: `192.168.0.189/24`
- Adres werd dynamisch toegewezen

### Tailscale

- Interface: `tailscale0`
- IPv4: `100.82.72.81/32`
- Biedt een overlay-netwerk voor externe/private toegang tot de server

### Kubernetes networking

- `cni0`: Kubernetes bridge, `10.42.0.1/24`
- `flannel.1`: onderdeel van het k3s/Flannel pod-netwerk
- Pod-IP's bevinden zich in het `10.42.0.x` netwerk
- `veth` interfaces verbinden pods met het host/container-netwerk

### Docker

- Interface: `docker0`
- IPv4: `172.17.0.1/16`
- Interface was DOWN / zonder actieve carrier

## 4. Kubernetes / k3s

### Cluster

- Distributie: k3s
- Kubernetes/k3s server: `v1.36.3+k3s1`
- Single-node cluster
- Node: `server`
- Container runtime: containerd
- Pod-netwerk: Flannel/CNI

### Aanwezige namespaces

- `beszel`
- `cluster-monitor`
- `default`
- `devops-learning-os`
- `homarr`
- `k8s-gateway`
- `kb-monitor`
- `kube-system`
- `metallb-system`
- `monitoring`

### Belangrijkste workloads

In het cluster draaiden onder andere:

- Beszel
- Homarr
- Memos
- DevOps Learning OS
- PostgreSQL
- k8s-gateway
- Traefik
- CoreDNS
- MetalLB
- Prometheus
- Grafana
- Alertmanager
- kube-state-metrics
- node-exporter

### Persistent Storage

Kubernetes gebruikte de `local-path` StorageClass voor lokale persistent volumes.

Onder andere vastgesteld:

- `memos-pvc`: 2 GiB, Bound, RWO
- Kubernetes persistent data stond op de lokale systeemdisk

## 5. Rebuild Decision

### Data-classificatie

De huidige server is een persoonlijk homelab. Ik ben zelf de eigenaar van de aanwezige data en workloads.

Na inventarisatie is besloten dat:

- geen applicatie- of bedrijfsdata behouden hoeft te blijven;
- de bestaande Kubernetes workloads opnieuw opgebouwd mogen worden;
- de aanwezige persistent volumes verwijderd mogen worden;
- de data op de tweede NTFS-disk niet behouden hoeft te blijven;
- de huidige omgeving volledig vernietigd mag worden.

### Professionele werkwijze

In een productieomgeving zou een wipe pas plaatsvinden na:

1. identificeren van de data-eigenaar;
2. bepalen welke data behouden moet blijven;
3. controleren of een geldige backup bestaat;
4. testen of recovery mogelijk is;
5. verkrijgen van expliciete goedkeuring voor verwijdering.

Voor dit homelab is bewust besloten dat recovery van de bestaande workloads en data niet vereist is.

### Doel van de rebuild

De server wordt vanaf nul opnieuw opgebouwd om iedere laag van het platform bewust te ontwerpen, configureren, testen en documenteren.

De nieuwe omgeving wordt daarmee onderdeel van het leer- en bewijsproces richting Platform Engineer / Cloud Architect.
