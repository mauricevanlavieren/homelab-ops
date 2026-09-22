# Current Learning Session

**Date:** 2026-09-22  
**Project:** Project 1 — Mini Platform  
**Phase:** Linux Foundation / Mental Map  
**Status:** In progress

---

# 1. Current Goal

Build the Linux foundation required before installing k3s.

The purpose is not to memorize Linux commands, but to understand the
main Linux layers and learn how to investigate a system independently.

Current mental model:

Linux
├── Network
├── Storage
├── Processes
├── Services
├── Users & Permissions
├── Logs
└── Software / Packages

The first six areas have now been explored practically.

Software / Packages is the next Linux domain.

---

# 2. Users & Permissions

## Identity model

Linux users are security identities.

A user can represent:

- a human
- an administrator
- a service account

A process runs under a user identity.

Example:

mau
 └── bash
      └── cat

root
 └── privileged processes

service account
 └── application process

Important distinction:

A service or process is not itself a user.
A process runs AS a user.

---

## `/etc/passwd`

Observed:

root:x:0:0:root:/root:/bin/bash
mau:x:1000:1000:Mau:/home/mau:/bin/bash

Fields investigated:

username
password placeholder
UID
primary GID
description
home directory
login shell

`x` does not contain the password.

Password information is stored separately in `/etc/shadow`.

---

## File permissions

Permission categories:

OWNER | GROUP | OTHERS

Permissions:

r = read
w = write
x = execute / traverse

Numeric representation:

r = 4
w = 2
x = 1

Examples practiced:

644
600
400

Test file:

permissions-test.txt

Permissions were deliberately changed.

At `400`, writing to the file failed with:

Permission denied

This demonstrated the difference between:

- ownership
- file permissions
- ability to change file contents
- ability of the owner to change the permission mode

---

## `/etc/shadow`

Observed permissions:

-rw-r----- root shadow /etc/shadow

Interpretation:

owner: root
permissions: rw-

group: shadow
permissions: r--

others:
permissions: ---

User `mau` is neither root nor a member of the shadow group.

Therefore `mau` falls into:

OTHERS

and cannot directly read `/etc/shadow`.

This was verified experimentally.

---

# 3. sudo and Least Privilege

Investigated `/etc/group`.

Relevant evidence:

sudo:x:27:mau

This shows that `mau` is a member of the supplementary `sudo` group.

Investigated sudo policy.

Relevant `/etc/sudoers` rule:

%sudo ALL=(ALL:ALL) ALL

Interpretation:

%sudo
→ WHO: members of the sudo group

first ALL
→ WHERE: hosts

(ALL:ALL)
→ run command as any user / group

final ALL
→ allowed commands

Important correction learned:

The final `ALL` does NOT directly grant all file permissions.

It allows commands to be executed with another identity.

The resulting process identity and filesystem permissions determine
what that process can actually access.

Effective sudo permissions were checked with:

sudo --list

Observed:

User mau may run the following commands on homeserver:
    (ALL : ALL) ALL

---

# 4. Services and Processes

Observed SSH:

systemctl status ssh

Evidence:

ssh.service
→ systemd service/unit

sshd
→ actual running process

PID 1088
→ identifies the process

Important distinction:

SERVICE
→ management abstraction / desired functionality

PROCESS
→ actual running instance of a program executed by the CPU

Connection:

systemd
   ↓ manages
ssh.service
   ↓ starts/manages
sshd
   ↓
process PID 1088

This distinction still needs repetition in future exercises.

---

# 5. Network Sockets

Investigated active TCP connections with:

ss -tn

Observed established SSH connections between:

homeserver:
192.168.0.10:22

laptop:
192.168.0.111:<ephemeral-port>

Then investigated listening TCP sockets:

ss -tln

Flags:

-t = TCP
-l = listening
-n = numeric

Observed:

0.0.0.0:22
[::]:22

SSH is listening on TCP port 22.

Also observed:

127.0.0.53:53
127.0.0.54:53

Port 53 was recognized as DNS-related.

---

# 6. Cross-Layer Investigation

Used:

sudo ss -tlnp

Additional flag:

-p = show process using socket

Without sufficient privileges, process information may not be visible.

With sudo, observed:

0.0.0.0:22
→ sshd
→ PID 1088

127.0.0.53:53
→ systemd-resolve
→ PID 761

This connected multiple Linux layers:

SERVICE
   ↓
PROCESS
   ↓
PID
   ↓
NETWORK SOCKET
   ↓
IP : PORT

Example:

ssh.service
   ↓
sshd
   ↓
PID 1088
   ↓
TCP
   ↓
0.0.0.0:22

This also reinforced least privilege:

sudo was used because additional process information was required,
not simply because "the command doesn't work".

---

# 7. Logs

Mental model:

systemctl status <service>
→ What is the service doing NOW?

journalctl -u <service>
→ What happened over TIME?

Investigated SSH logs:

journalctl -u ssh

Observed events including:

- SSH service startup
- listening on port 22
- successful authentication
- session creation
- service shutdown
- reboot boundary
- service restart

Example troubleshooting path:

Problem
   ↓
identify domain
   ↓
Service
   ↓
systemctl status
   ↓
need historical explanation
   ↓
Logs
   ↓
journalctl
   ↓
filter evidence

---

## Time filtering

Found and used:

-S / --since

Final successful query:

journalctl -u ssh -S "2026-9-22 16:00"

Result contained only SSH events after the requested time.

This exercise also exposed an important troubleshooting lesson:

A command can be syntactically correct while the input value is wrong.

Two incorrect dates were detected and corrected before obtaining the
intended evidence.

---

# 8. Connected Linux Mental Model

Today's strongest result was connecting previously separate concepts.

Current model:

USER / IDENTITY
      │
      │ permissions
      ▼
SERVICE ── managed by ──> systemd
      │
      ▼
PROCESS
      │
      ├── PID
      │
      └── runs as USER
      │
      ▼
NETWORK SOCKET
      │
      ▼
IP : PORT

And for troubleshooting:

PROBLEM
   ↓
DOMAIN
   ↓
OBSERVATION TOOL
   ↓
EVIDENCE
   ↓
HYPOTHESIS
   ↓
VERIFY

Example:

SSH problem
   ↓
Services
   ↓
systemctl status ssh
   ↓
sshd running?
   ↓
Logs
   ↓
journalctl -u ssh
   ↓
Network
   ↓
ss -tlnp
   ↓
Is port 22 actually listening?

---

# 9. Search / Investigation Strategy

Current Linux investigation route:

1. Identify the domain/layer
2. Formulate the exact question
3. Identify the manager/source of truth
4. Find the appropriate tool
5. Inspect help/man page when needed
6. Observe evidence
7. Interpret before changing anything

General model:

PROBLEM
   ↓
DOMAIN / LAYER
   ↓
MANAGER / SOURCE
   ↓
TOOL
   ↓
EVIDENCE

Commands and flags themselves are mostly:

📖 OPZOEKEN

The mental model and investigation route are:

🧠 KENNEN

Using them on real systems is:

🔧 KUNNEN

---

# 10. Evidence Created Today

Real evidence used during the session:

- `/etc/passwd`
- `/etc/group`
- `/etc/shadow` permissions
- `/etc/sudoers`
- `sudo --list`
- file permission experiment
- `systemctl status ssh`
- `journalctl -u ssh`
- `journalctl --since`
- `ss -tn`
- `ss -tln`
- `sudo ss -tlnp`
- real SSH connections
- real listening sockets
- real process IDs
- real DNS resolver sockets

No simulated environment was used.

---

# 11. Skill Status

Important:

Today's exercises provide evidence of guided understanding and practical
application.

They do NOT yet prove independent mastery.

Current retention rule:

A skill can only become 🟢 Independent after:

- criteria have been completed
- it can be reproduced independently
- it is applied in another practical context
- it is reproduced again after at least 48 hours
- the learner can explain why it works

Earliest retention evidence from today's Users & Permissions / Logs work:

2026-09-24

Do not simply repeat the same questions.

Future assessment should test transfer into another real problem.

---

# 12. Current Linux Foundation Status

Network              🟡 Guided
Storage              🟡 Guided
Processes            🟡 Guided
Services             🟡 Guided
Users & Permissions  🟡 Guided
Logs                 🟡 Guided
Software / Packages  🔴 Not started

Note:

🟡 does not mean all commands are remembered.

It means the concepts have been explored and applied with guidance.

---

# 13. Exact Stopping Point

Linux Foundation is intentionally paused BEFORE:

Software / Packages

Next question:

"How does software actually get onto an Ubuntu server?"

Start from the existing Windows mental model:

Windows software installation
        ↓
compare similarities/differences
        ↓
Ubuntu package/software model
        ↓
real investigation on homeserver

Do NOT start with k3s installation yet.

After Software / Packages, reconnect the complete Linux mental map and
then return to Project 1 / k3s.

---

# DAILY SKILL PROGRESS — 2026-09-22

No unsupported percentages are recorded yet.

Percentages will only be added after objective evidence criteria have
been defined for each major skill.

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

LEVELS

🔴 0 Not known
🟠 1 Recognition
🟡 2 Guided
🟢 3 Independent
🔵 4 Engineer
🟣 5 Architect

🟢 requires independent evidence and retention evidence ≥48h later.

---

# Next Session

START HERE:

Software / Packages

First conceptual question:

"On Windows, how would you normally install software such as Chrome?"

Then build the bridge:

Windows software model
        ↓
Linux software/package model
        ↓
Ubuntu package management
        ↓
real homeserver investigation
        ↓
complete Linux foundation
        ↓
return to k3s
