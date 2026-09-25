# Current Session — 25-09-2026

## Project
Project 1 — Mini Platform

## Topic
Kubernetes networking → DNS → CoreDNS → ConfigMap → Deployment

## Status
🟡 IN PROGRESS

---

# 1. STARTPUNT VANDAAG

De nginx workload werkte al via NodePort:

Laptop
  ↓
192.168.0.10:30008
  ↓
NodePort Service
  ↓
nginx Pod

Doel van vandaag:

http://web.home.arpa

in plaats van:

http://192.168.0.10:30008

Daarvoor moeten twee verschillende problemen worden opgelost:

1. DNS:
   web.home.arpa → 192.168.0.10

2. HTTP routing:
   192.168.0.10:80 → Traefik → Service → Pod

---

# 2. DNS — HUIDIGE SITUATIE ONDERZOCHT

Vanaf laptop:

dig web.home.arpa

Resultaat:

NXDOMAIN
ANSWER: 0
SERVER: 127.0.0.53

Onderzocht met:

resolvectl status

Werkelijke DNS-keten:

Laptop
  ↓
systemd-resolved
127.0.0.53
  ↓
192.168.0.1
router / huidige DNS
  ↓
web.home.arpa onbekend
  ↓
NXDOMAIN

Belangrijk begrip:

127.0.0.53 is de lokale systemd-resolved stub.
192.168.0.1 is momenteel de upstream DNS-server van de laptop.

---

# 3. COREDNS BINNEN KUBERNETES

Bestaande Kubernetes DNS onderzocht:

kube-dns
Type: ClusterIP
ClusterIP: 10.43.0.10
Ports:
- 53/UDP
- 53/TCP
- 9153/TCP

Belangrijk onderscheid:

Kubernetes CoreDNS
  → interne Kubernetes DNS
  → bijvoorbeeld cluster.local

Homelab DNS
  → DNS voor laptop/LAN
  → bijvoorbeeld web.home.arpa

Architectuurkeuze:

NIET de bestaande Kubernetes CoreDNS uitbreiden/exposen.

Reden:
separation of concerns + kleinere blast radius.

We bouwen daarom een aparte DNS-workload.

---

# 4. ROUTING NAAR KUBERNETES CLUSTERIP ONDERZOCHT

Laptop:

ip route get 10.43.0.10

Resultaat:

10.43.0.10 via 192.168.0.1

Traceroute:

Laptop
192.168.0.111
  ↓
192.168.0.1
  ↓
192.168.1.1
  ↓
...

Conclusie:

10.43.0.10 is een Kubernetes ClusterIP en is niet rechtstreeks
LAN-facing.

Belangrijk onderscheid:

ip route get
  → welke route Linux kiest

traceroute
  → welke netwerkroute het verkeer daadwerkelijk begint te volgen

---

# 5. SERVICE TYPES VERDIEPT

ClusterIP
  → intern Kubernetes-adres

NodePort
  → node-IP + hoge poort
  → bijvoorbeeld 192.168.0.10:30008

LoadBalancer
  → extern/LAN bereikbaar adres en normale servicepoort

Voor homelab DNS is gekozen:

LoadBalancer

Doel:

192.168.0.10:53 UDP
192.168.0.10:53 TCP

---

# 6. K3S LOADBALANCER ONDERZOCHT

Pods in kube-system:

kubectl get pods -n kube-system

Onder andere:

traefik-...
svclb-traefik-...

svclb-traefik onderzocht:

kubectl describe pod svclb-traefik-... -n kube-system

Belangrijke evidence:

Node:
homeserver/192.168.0.10

Container lb-tcp-80:
Port:      80/TCP
Host Port: 80/TCP

SRC_PORT: 80
DEST_PORT: 80
DEST_IPS: 10.43.58.247

Container lb-tcp-443:
Port:      443/TCP
Host Port: 443/TCP

Traefik Service:
ClusterIP: 10.43.58.247

Mentale route:

Laptop
  ↓
192.168.0.10:80
  ↓
k3s ServiceLB / svclb-traefik
  ↓
10.43.58.247:80
  ↓
Traefik Service
  ↓
Traefik Pod

Nieuw begrip:

k3s gebruikt ServiceLB / klipper-lb om een LoadBalancer Service
op de node beschikbaar te maken.

---

# 7. POORT 53 ONDERZOCHT

Op homeserver:

ss -tulpn

Onder andere:

127.0.0.53:53
127.0.0.54:53

systemd-resolved gebruikt dus poort 53 alleen op loopback-adressen.

Belangrijk geleerd:

0.0.0.0:PORT
  → luistert op alle IPv4 interfaces

127.x.x.x:PORT
  → loopback / alleen lokaal

Een poortconflict moet worden bekeken als combinatie van:

IP + protocol + poort

Voor DNS:

192.168.0.10:53/UDP
192.168.0.10:53/TCP

Er is nog geen bewijs van een conflict op deze LAN-combinaties.

---

# 8. HOMELAB DNS REQUIREMENTS

Nieuwe aparte DNS moet:

1. draaien in Kubernetes
2. bereikbaar zijn vanaf LAN
3. bereikbaar zijn op 192.168.0.10:53
4. UDP 53 ondersteunen
5. TCP 53 ondersteunen
6. web.home.arpa → 192.168.0.10 beantwoorden
7. overige DNS-vragen doorsturen naar upstream DNS
8. declaratief configureerbaar zijn
9. later vanuit Git beheerd kunnen worden

Architectuur:

Laptop
  │
  │ DNS :53
  ▼
192.168.0.10
  │
  ▼
LoadBalancer Service
  │
  ▼
aparte CoreDNS Pod
  │
  ├── hosts
  │    web.home.arpa → 192.168.0.10
  │
  └── forward
       overige DNS → 192.168.0.1

---

# 9. COREDNS PLUGINS

Uit documentatie onderzocht.

hosts
  → eigen/static DNS-namen beantwoorden

forward
  → overige DNS-vragen doorsturen naar upstream resolver

fallthrough
  → wanneer hosts een naam niet kent, laat de query verdergaan

Mentale route:

web.home.arpa
  ↓
hosts
  ↓
192.168.0.10


google.com
  ↓
hosts
  ↓
niet gevonden
  ↓
fallthrough
  ↓
forward
  ↓
192.168.0.1

Exacte CoreDNS-syntax valt onder:

📖 OPZOEKEN

Conceptuele werking valt onder:

🧠 KENNEN

---

# 10. NAMESPACE

Aangemaakt:

kubectl create namespace homelab-dns

Resultaat:

namespace/homelab-dns created

Architectuur:

Kubernetes
├── kube-system
│   └── CoreDNS
│       interne cluster DNS
│
├── web
│   └── nginx
│
└── homelab-dns
    └── onze LAN DNS

---

# 11. CONFIGMAP

Bestand:

coredns-config.yaml

Inhoud:

apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-config
  namespace: homelab-dns
data:
  Corefile: |
    .:53 {
      hosts {
        192.168.0.10 web.home.arpa
        fallthrough
      }
      forward . 192.168.0.1
      log
      errors
    }

Toegepast:

kubectl apply -f coredns-config.yaml -n homelab-dns

Resultaat:

configmap/coredns-config created

Belangrijk begrip:

ConfigMap
  → configuratie los van container-image

Image:
  "welke software?"

ConfigMap:
  "hoe moet deze software zich in deze omgeving gedragen?"

---

# 12. CONFIGMAP → CONTAINER

Nieuw Kubernetes-patroon geleerd:

ConfigMap
  ↓
Volume
  ↓
VolumeMount
  ↓
bestand in container

Onze concrete keten:

ConfigMap: coredns-config
  │
  │ bevat key: Corefile
  ▼
Pod volume: coredns-config
  ▼
volumeMount
  ▼
/etc/coredns
  ▼
/etc/coredns/Corefile
  ▼
CoreDNS

---

# 13. DEPLOYMENT

Bestand:

coredns-deployment.yaml

Huidige inhoud:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: coredns
  namespace: homelab-dns
  labels:
    app: coredns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: coredns
  template:
    metadata:
      labels:
        app: coredns
    spec:
      containers:
      - name: coredns
        image: coredns/coredns:1.14.7
        args:
          - -conf
          - /etc/coredns/Corefile
        volumeMounts:
          - name: coredns-config
            mountPath: /etc/coredns
            readOnly: true
      volumes:
        - name: coredns-config
          configMap:
            name: coredns-config

LET OP:

Deployment is nog NIET toegepast.

Dit is het exacte stoppunt.

---

# 14. DEPLOYMENT REGEL-VOOR-REGEL DOORGENOMEN

matchLabels:
  selector/filter op labels

app: coredns
  key=value label waarop gezocht wordt

template:
  bouwtekening voor toekomstige Pods

metadata:
  identificerende/beschrijvende informatie van de Pod-template

labels:
  labels die toekomstige Pods krijgen

app: coredns
  Pod krijgt label app=coredns

spec:
  gewenste technische opbouw van de Pod

containers:
  lijst containers binnen de Pod

- name: coredns
  naam van onze container

image: coredns/coredns:1.14.7
  container-image/software + gepinde versie

args:
  argumenten voor het programma bij starten

-conf
  CoreDNS argument: gebruik specifiek configuratiebestand

/etc/coredns/Corefile
  pad naar dat configuratiebestand

volumeMounts:
  welke volumes deze container gebruikt en waar

name: coredns-config
  verwijst naar Pod-volume met dezelfde naam

mountPath: /etc/coredns
  volume wordt daar zichtbaar in container

readOnly: true
  container mag configuratie alleen lezen

volumes:
  volumes die de Pod beschikbaar heeft

name: coredns-config
  naam van het Pod-volume

configMap:
  bron van volume is een Kubernetes ConfigMap

name: coredns-config
  specifieke ConfigMap die als bron wordt gebruikt

---

# 15. BELANGRIJKE VERBINDINGEN VANDAAG

DNS ≠ Traefik.

DNS:
naam → IP

Traefik:
HTTP-request → juiste Kubernetes Service

Gewenste eindroute:

Browser
  │
  │ web.home.arpa
  ▼
Homelab DNS
  │
  │ 192.168.0.10
  ▼
Traefik :80
  │
  │ host/path routing
  ▼
Service web
  │
  │ selector
  ▼
nginx Pod

---

# 16. LEEROBSERVATIES

Sterk vandaag:

- Zelfstandig juiste namespace aangemaakt.
- LoadBalancer gekozen op basis van requirements.
- Bewust gekozen voor aparte homelab-DNS wegens separation of concerns.
- Correct begrepen dat svclb verkeer doorstuurt naar Traefik Service.
- `hosts` correct gekoppeld aan lokale DNS-records.
- `fallthrough` conceptueel correct uitgelegd.
- selector → label relatie opnieuw herkend.
- `/etc/coredns/Corefile` correct afgeleid uit mountPath + ConfigMap key.
- Deployment/ConfigMap koppeling uiteindelijk correct opgebouwd.
- Bestaande evidence kritisch gebruikt.

Nog versterken:

- CoreDNS versus Traefik blijft herhaling nodig hebben.
- ClusterIP / NodePort / LoadBalancer nog niet als retained kennis beschouwen.
- `ss` nog niet zelfstandig teruggehaald.
- ConfigMap/volume/volumeMount is nieuw en begeleid.
- YAML-hiërarchie nog begeleid.
- CoreDNS Corefile-syntax is 📖 OPZOEKEN.
- DNS UDP/TCP 53 nog herhalen.
- `0.0.0.0` versus loopback later opnieuw zelfstandig testen.

Geen 🟢 promoties op basis van vandaag alleen.
Nieuwe kennis moet later zelfstandig en na ≥48 uur opnieuw bewezen worden.

---

# 17. EXACT STOPPUNT / VOLGENDE SESSIE

NIET opnieuw beginnen.

We staan hier:

ConfigMap       ✅ aangemaakt
Namespace       ✅ aangemaakt
Deployment YAML ✅ voorbereid
Deployment      ❌ nog niet toegepast
CoreDNS Pod     ❌ nog niet bewezen Running
Service         ❌ nog niet gemaakt
DNS test        ❌ nog niet gedaan
Resolver switch ❌ nog niet gedaan
Ingress         ❌ nog niet gemaakt

VOLGENDE ACTIE:

1. Kort reconstrueren wat Deployment + ConfigMap doen.
2. coredns-deployment.yaml toepassen.
3. Observeren wat Kubernetes werkelijk maakt.
4. Pod-status controleren.
5. Bij fout: NIET direct repareren; eerst evidence/logs/events.
6. CoreDNS intern testen.
7. Daarna pas LoadBalancer Service ontwerpen voor UDP/TCP 53.
8. Testen met directe DNS-query naar 192.168.0.10.
9. Daarna resolver/DHCP-route ontwerpen.
10. Daarna Traefik Ingress voor web.home.arpa.
11. Uiteindelijk NodePort 30008 overbodig maken.
12. Git diff → commit → push als evidence.

---

# 18. CURRENT ARCHITECTURE

Laptop 192.168.0.111
│
├── DNS momenteel
│      ↓
│   systemd-resolved
│      ↓
│   192.168.0.1
│
└── Kubernetes access
       ↓
    homeserver 192.168.0.10
       ↓
    k3s
       │
       ├── kube-system
       │    ├── Kubernetes CoreDNS
       │    ├── Traefik
       │    └── ServiceLB
       │
       ├── web
       │    ├── nginx Deployment
       │    ├── nginx Pod
       │    └── NodePort Service :30008
       │
       └── homelab-dns
            ├── ConfigMap coredns-config ✅
            ├── CoreDNS Deployment YAML prepared
            ├── CoreDNS Pod ❌
            └── LoadBalancer Service ❌

---

# 19. DAILY SKILL PROGRESS — 25-09-2026

╔════════════════════ DAILY SKILL PROGRESS — 25-09-2026 ════════════════════╗

                                        LEVEL
 1  Git / Repository / Governance        🟡 2
 2  Linux                                🟡 2
 3  Server Foundation                    🟡 2
 4  Network / DNS / Ingress              🟡 2
 5  Kubernetes Platform                  🟡 2
 6  Storage                              🟡 2
 7  Secrets                              🔴 0
 8  CI/CD                                🟠 1
 9  GitOps                               🟠 1
10  Observability                        🟠 1
11  Security                             🟡 2
12  Developer Platform / Self-Service    🟠 1
13  Reliability / Backup / DR            🟠 1
14  Cloud Platform                       🔴 0
15  Hybrid / Multi-environment           🔴 0
16  Chaos / Incident Response            🟠 1
17  Employer Portfolio / Assessment      🟠 1
18  Final Zero-to-Production Rebuild     🔴 0

TODAY'S EVIDENCE

Network / DNS:
+ DNS resolver chain investigated
+ routing to ClusterIP investigated
+ traceroute evidence
+ port 53 binding investigated
+ DNS architecture designed

Kubernetes:
+ Service types reinforced
+ k3s ServiceLB investigated
+ ConfigMap created
+ ConfigMap → Volume → VolumeMount understood
+ CoreDNS Deployment YAML built with guidance

Architecture:
+ separation of concerns applied
+ separate LAN DNS chosen instead of modifying cluster DNS
+ blast radius considered
+ minimal-complexity principle applied

LEVELS
🔴 0 Niet bekend    🟠 1 Herkenning    🟡 2 Begeleid
🟢 3 Zelfstandig    🔵 4 Engineer      🟣 5 Architect

IMPORTANT:
No level promotion today.
🟢 requires independent evidence, transfer to another situation,
explanation of why it works, and reproduction ≥48 hours later.

Percentages remain omitted until objective capability criteria
have been defined.

╚═════════════════════════════════════════════════════════════════════════════╝
