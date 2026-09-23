# Current Session — 2026-09-23

## Project

**Project 1 — Mini Platform**

Goal:

```text
Laptop
  ↓
Git / GitHub
  ↓
Ubuntu Server
  ↓
k3s
  ↓
Kubernetes
  ↓
Deployment
  ↓
Pod / Application
  ↓
Service
  ↓
Browser
```

Current phase:

**Linux Foundation — completing the connected mental model before continuing with k3s.**

---

# 1. Software / Packages

Today the Ubuntu package-management chain was connected and practiced.

Mental model:

```text
Repository
    ↓
apt update
    ↓
local package catalog
    ↓
Package
    ├── version
    └── dependencies
            ↓
      install / remove
            ↓
    installed software
```

Important distinction:

```text
apt update
→ refreshes local package information

apt list --upgradable
→ compares installed packages with local catalog

apt upgrade
→ actually changes installed packages
```

## Repository configuration

Observed:

```text
/etc/apt/sources.list.d/ubuntu.sources
```

Repositories include:

```text
http://nl.archive.ubuntu.com/ubuntu/
http://security.ubuntu.com/ubuntu/
```

Ubuntu release:

```text
noble = Ubuntu 24.04 LTS
```

Important distinction:

```text
URI
→ where repository data comes from

Signed-By
→ key used to verify repository metadata
```

Repository signing/security will be covered later.

---

# 2. Permissions transfer exercise

Running:

```bash
apt update
```

without sudo produced:

```text
Could not open lock file /var/lib/apt/lists/lock
Permission denied
```

Investigated:

```bash
ls -ld /var/lib/apt/lists/
```

Result:

```text
drwxr-xr-x root root
```

Needed to determine which permission category user `mau` belongs to.

Discovered:

```bash
id mau
```

Result:

```text
uid=1000(mau)
gid=1000(mau)
groups=1000(mau),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),101(lxd)
```

Conclusion:

```text
directory owner = root
directory group = root

mau ≠ root user
mau ∉ root group

→ mau falls under "others"
→ others has r-x
→ no write permission
→ apt cannot modify /var/lib/apt/lists/
```

Using:

```bash
sudo apt update
```

worked because APT was then executed with elevated privileges.

This was transfer evidence for:

- UID/GID
- groups
- owner/group/others
- rwx
- sudo
- least privilege

Not yet sufficient for 🟢 because ≥48h retention evidence is still required.

---

# 3. Package lifecycle practical lab

Used `cowsay` as a safe test package.

Initial verification:

```bash
apt list --installed cowsay
```

No package returned.

Installed:

```bash
sudo apt install cowsay
```

Observed lifecycle:

```text
Get
 ↓
Selecting
 ↓
Unpacking
 ↓
Setting up
```

Verified:

```bash
apt list --installed cowsay
```

Result:

```text
cowsay/noble,now 3.03+dfsg2-8 all [installed]
```

Removed:

```bash
sudo apt remove cowsay
```

APT reported:

```text
1 to remove
93.2 kB disk space will be freed
```

Verified removal:

```bash
apt list --installed cowsay
```

No package returned.

## Manual vs automatic packages

Observed earlier:

```text
curl [installed,automatic]
cowsay [installed]
```

Mental model:

```text
manual
→ explicitly requested package

automatic
→ installed because another package needed it as a dependency
```

APT can therefore later recognize dependencies that may no longer be required.

---

# 4. Linux troubleshooting mini-boss

Scenario:

```text
ssh: connect to host 192.168.0.10 port 22:
Connection refused
```

Goal was to troubleshoot conceptually instead of trying random commands.

Investigation route developed:

```text
Laptop
  ↓
Network reachability
  ↓
Server
  ↓
Service
  ↓
Process
  ↓
Listening socket
  ↓
Firewall
  ↓
Authentication / application
```

## Network reachability

First hypothesis:

```text
Can the laptop reach the homeserver?
```

Use ping.

If ping succeeds, basic IP reachability exists and investigation moves higher in the stack.

## SSH service

Investigated:

```bash
systemctl status ssh
```

Evidence:

```text
ssh.service - OpenBSD Secure Shell server
Active: active (running)
Main PID: 1088 (sshd)
```

Connected model:

```text
systemd
   ↓ manages
ssh.service
   ↓ manages/starts
sshd
   ↓
process PID 1088
```

## Listening socket

Investigated:

```bash
ss -tln
```

Evidence:

```text
0.0.0.0:22 LISTEN
[::]:22      LISTEN
```

Meaning:

```text
0.0.0.0:22
→ listen on TCP port 22 on all local IPv4 interfaces

[::]:22
→ IPv6 equivalent
```

Also observed:

```bash
ss -tl
```

shows:

```text
:ssh
```

while:

```bash
ss -tln
```

shows:

```text
:22
```

`-n` prevents service-name translation and shows numeric ports.

## Client configuration

Investigated:

```bash
ssh -G server
```

Evidence:

```text
user mau
hostname 192.168.0.10
port 22
```

Therefore the client configuration targets:

```text
mau@192.168.0.10:22
```

---

# 5. Firewall — missing prerequisite discovered

The mini-boss exposed a missing connection in the Linux mental model.

Previously:

```text
client
 ↓
network
 ↓
socket
 ↓
service
```

Improved model:

```text
client
 ↓
network
 ↓
firewall
 ↓
socket / port
 ↓
service
 ↓
process
```

Important distinction:

```text
ss
→ Is something listening on this port?

firewall
→ Is network traffic allowed to reach it?
```

Investigated UFW:

```bash
sudo ufw status verbose
```

Result:

```text
Status: inactive
```

Therefore UFW currently does not enforce firewall rules on the homeserver.

Important security lesson:

Do not blindly enable a firewall on a remotely administered server.

If incoming traffic defaults to DENY and SSH is not allowed first:

```text
Laptop
  ↓
SSH :22
  ↓
FIREWALL
  ✕
  ↓
sshd
```

the administrator can lock themselves out.

Firewall concept is now connected but not independently proven.

---

# 6. Linux filesystem layout

Started building a filesystem search map.

Root filesystem:

```text
/
├── etc
├── home
├── usr
├── var
├── run
├── tmp
├── boot
└── ...
```

Current useful mental labels:

```text
/etc
→ system-wide configuration

/home
→ personal files/environment of normal users

/var
→ changing system/application data

/usr
→ installed programs, libraries and related files

/run
→ runtime information for the current boot
```

Only `/etc`, `/home`, `/var` and the beginning of `/usr` were explored today.

---

# 7. /etc

Previously observed:

```text
/etc/netplan/
/etc/apt/
/etc/passwd
/etc/group
/etc/shadow
/etc/sudoers
```

Mental label:

```text
/etc
→ system-wide configuration
```

Search principle:

```text
"How is this Linux system configured?"
              ↓
            /etc
```

---

# 8. /home

For user `mau`:

```text
/home/mau
```

Contains the personal environment/files/directories of the user.

Special case:

```text
root user's home
→ /root
```

not:

```text
/home/root
```

---

# 9. /var and logs

Observed:

```text
/var
├── backups
├── cache
├── crash
├── lib
├── log
├── mail
├── spool
└── tmp
```

Mental model:

```text
/var
→ variable/changing data produced during system operation
```

Observed:

```text
/var/log
├── apt
├── auth.log
├── dpkg.log
├── journal
├── kern.log
└── syslog
```

Used filesystem structure as a search strategy:

```text
Question:
"What did APT do?"
       ↓
Need historical information
       ↓
logs
       ↓
/var/log
       ↓
APT
       ↓
/var/log/apt
```

Observed:

```text
/var/log/apt/
├── history.log
├── term.log
└── eipp.log.xz
```

`history.log` contained evidence from today's package lab:

```text
Start-Date: 2026-09-23 08:52:19
Commandline: apt install cowsay
Requested-By: mau (1000)
Install: cowsay:amd64 (3.03+dfsg2-8)

Start-Date: 2026-09-23 08:56:26
Commandline: apt remove cowsay
Requested-By: mau (1000)
Remove: cowsay:amd64 (3.03+dfsg2-8)
```

This connected:

```text
filesystem
   +
logs
   +
APT
   +
user identity
   +
UID 1000
```

---

# 10. /usr and command discovery

Investigated where `ls` exists:

```bash
whereis ls
```

Result:

```text
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

Meaning:

```text
/usr/bin/ls
→ executable program

/usr/share/man/man1/ls.1.gz
→ manual/documentation
```

This introduced the question:

```text
Why can the user type:

ls

instead of:

/usr/bin/ls
```

---

# 11. PATH — CURRENT LEARNING EDGE

Found Bash documentation:

```text
PATH
The search path for commands.
```

PATH is a colon-separated list of directories.

Observed current PATH:

```bash
echo $PATH
```

Result:

```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

Initial mental model:

```text
command: ls
   ↓
Bash
   ↓
PATH
   ↓
search directories
   ↓
/usr/bin
   ↓
/usr/bin/ls
```

However this concept is NOT yet sufficiently connected.

The important conceptual problem discovered:

Previous programming experience created this question:

> "If a variable exists, where was it originally defined?"

Current distinction being developed:

```text
ON SSD                         DURING EXECUTION / RAM

configuration/code      →      process environment
                                  │
                                  └── PATH=...
```

Analogy with Python:

```text
app.py on SSD
naam = "Maurice"
      ↓
Python starts
      ↓
Python process in RAM
      ↓
variable exists during execution
```

For Linux/Bash we need to understand:

```text
Fresh PC
  ↓
Ubuntu installation
  ↓
configuration/files on SSD
  ↓
boot
  ↓
kernel
  ↓
systemd
  ↓
login
  ↓
Bash starts
  ↓
environment is constructed
  ↓
PATH exists for the running shell/process
```

This connection is NOT finished.

---

# Exact stopping point

Do NOT continue directly with more PATH syntax.

Next session starts with the conceptual question:

> **Where does the PATH value originally come from on disk/configuration when a fresh Ubuntu system boots and user `mau` logs in?**

Build the connection:

```text
SSD/configuration
      ↓
boot/login
      ↓
Bash process
      ↓
environment
      ↓
PATH
      ↓
command lookup
      ↓
/usr/bin/ls
```

Use the real homeserver to investigate this chain.

After PATH:

1. finish `/usr`
2. `/run`
3. boot process
4. DNS/resolution connection
5. Linux Foundation mini-boss / assessment
6. return to Project 1
7. install/build k3s
8. first Kubernetes workload

---

# Evidence collected today

Practical evidence:

- `apt update` permission failure investigated
- `id mau`
- permissions on `/var/lib/apt/lists`
- successful `sudo apt update`
- APT repository configuration inspected
- package versions inspected
- `cowsay` installed
- installation verified
- `cowsay` removed
- removal verified
- APT history verified in `/var/log/apt/history.log`
- `systemctl status ssh`
- `ss -tl`
- `ss -tln`
- `ssh -G server`
- `sudo ufw status verbose`
- Linux filesystem root inspected
- `/var` inspected
- `/var/log` inspected
- `/var/log/apt` inspected
- `/usr/bin/ls` discovered
- current `$PATH` inspected

---

# Skill observations — 2026-09-23

## Stronger evidence today

### Linux troubleshooting

Demonstrated ability to move through:

```text
network
→ service
→ process
→ socket
```

Firewall was not spontaneously recognized because this prerequisite had not yet been connected.

### Users & Permissions

Previous knowledge was successfully reused in a new APT permission problem.

This is transfer evidence, but it occurred less than 48 hours after the original learning session.

Therefore:

**NO promotion to 🟢 yet.**

Earliest retention test remains:

**2026-09-24**

### Software / Packages

Package-management mental model and practical lifecycle completed under guidance.

Current status:

🟡 2 — Begeleid

### Firewall

Concept now recognized and connected to network troubleshooting.

Current evidence:

🟠/🟡 transition — not independently demonstrated yet.

### Filesystem layout

Beginning to use filesystem structure as a search strategy instead of memorizing paths.

Current status:

🟡 2 — Begeleid

### PATH / environment

Concept currently incomplete.

Do not score as independently understood.

---

# Skill Passport

No level promotion today.

Important rule:

```text
Seeing ≠ knowing
Using once ≠ independent mastery
```

🟢 requires:

- criteria completed
- independently reproduced/applied
- applied in another practical situation
- reproduced after ≥48 hours
- explanation of why it works

---

# DAILY SKILL PROGRESS — 23-09-2026

```text
╔════════════════════ DAILY SKILL PROGRESS — 23-09-2026 ════════════════════╗

                                      LEVEL
 1  Git / Repository / Governance     🟡 2
 2  Linux                             🟡 2
 3  Server Foundation                 🟡 2
 4  Network / DNS / Ingress           🟡 2
 5  Kubernetes Platform               🟡 2
 6  Storage                           🟡 2
 7  Secrets                           🔴 0
 8  CI/CD                             🟠 1
 9  GitOps                            🟠 1
10  Observability                     🟠 1
11  Security                          🟡 2
12  Developer Platform / Self-Service 🟠 1
13  Reliability / Backup / DR         🟠 1
14  Cloud Platform                    🔴 0
15  Hybrid / Multi-environment        🔴 0
16  Chaos / Incident Response         🟠 1
17  Employer Portfolio / Assessment   🟠 1
18  Final Zero-to-Production Rebuild  🔴 0

TODAY'S EVIDENCE

Linux
  ▲ package management lifecycle
  ▲ troubleshooting across multiple layers
  ▲ filesystem search strategy
  ▲ /etc /home /var /usr connections

Security
  ▲ sudo/permissions transfer
  ▲ firewall introduced
  ▲ least-privilege reasoning reinforced

Troubleshooting
  ▲ ping → service → process → socket
  ▲ hypotheses eliminated using evidence
  ▲ missing firewall prerequisite identified

RETENTION

Users & Permissions:
next eligible independent retention evidence ≥ 24-09-2026

LEVELS
🔴 0 Niet bekend    🟠 1 Herkenning    🟡 2 Begeleid
🟢 3 Zelfstandig    🔵 4 Engineer      🟣 5 Architect

🟢 requires independent evidence + reproduction ≥48h later

NOTE:
Percentages intentionally omitted until objective per-skill criteria are defined.

╚═════════════════════════════════════════════════════════════════════════════╝
```
