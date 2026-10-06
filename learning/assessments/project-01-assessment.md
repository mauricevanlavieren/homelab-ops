# Project 01 — Mini Platform Assessment

**Project:** Project 01 — Mini Platform  
**Assessment date:** 2026-10-06  
**Status:** Completed  
**Assessment type:** Practical + conceptual  
**Support during assessment:** No hints unless explicitly requested

---

## 1. Assessment Goal

The purpose of this assessment was to determine which concepts and skills from Project 01 can be reproduced independently.

The assessment does not measure whether a topic has been seen before.

Evidence is evaluated using:

1. Seen
2. Recognized
3. Understood
4. Applied
5. Independent

A skill is not promoted to independent solely because it was successfully used once.

---

# Assessment Results

## Assessment 1 — Git / Source of Truth

### Scenario

A direct change may have been made to the Kubernetes cluster without updating Git.

### Response

Used:

`kubectl diff -R -f ./kubernetes/`

and checked the exit code.

Result:

`0`

### Evaluation

**PASS**

The comparison between repository manifests and live Kubernetes resources was performed independently.

Important limitation recognized during review:

`kubectl diff` verifies resources represented by the manifests but does not prove that additional orphaned resources do not exist elsewhere in the cluster.

### Evidence

- Correct comparison method selected independently.
- Exit code checked.
- Difference between managed desired state and possible orphaned resources understood.

---

## Assessment 2 — Kubernetes Reconciliation

### Scenario

The running nginx Pod is manually deleted.

### Response

Correctly explained that Kubernetes will recreate the Pod because the desired replica count requires one Pod.

Initial explanation attributed creation too directly to the Deployment.

### Technical Model

`Deployment → ReplicaSet → Pod`

The Deployment defines the desired replica count.

The ReplicaSet maintains the required number of Pods.

### Evaluation

**PARTIAL PASS**

Kubernetes reconciliation and desired state are understood.

The exact responsibility between Deployment and ReplicaSet still needs reinforcement.

---

## Assessment 3 — Troubleshooting

### Scenario

`web.home.arpa` returns:

`503 Service Unavailable`

### Initial Response

Tools identified:

- `dig`
- `curl`
- `resolvectl`

The initial response focused on tools rather than the investigation strategy.

### Improved Response

The problem was then approached layer by layer:

`DNS → Traefik → Ingress → Service → Endpoint → Pod`

### Evaluation

**PASS / GUIDED DEPTH**

The troubleshooting principle is understood:

**collect evidence per layer before changing configuration.**

Technical observation points and exact tools still require further repetition.

---

## Assessment 4 — Service Architecture / Security

### Scenario

Why was the web Service changed from:

`NodePort 30008`

to:

`ClusterIP`

### Response

The initial explanation connected NodePort incorrectly with IP addresses versus hostnames.

### Correct Architecture

Normal application traffic already follows:

`Client → Traefik → Ingress → ClusterIP Service → Pod`

The NodePort created an additional external path:

`Node IP:30008 → Service → Pod`

This path was unnecessary.

Removing it:

- simplifies the architecture;
- removes unnecessary external exposure;
- reduces attack surface.

### Evaluation

**NOT YET PASSED**

The practical change was successfully implemented during Project 01, but the architectural reason was not independently reproduced during the assessment.

This concept must return naturally in future projects.

---

## Assessment 5 — Git Staging / Commit Review

### Scenario

Determine exactly what will be included in the next Git commit.

### Initial Response

`git diff`

### Correct Distinction

`git diff`

compares:

`working tree ↔ staging area`

`git diff --staged`

compares:

`HEAD ↔ staging area`

and therefore shows what is prepared for the next commit.

### Evaluation

**PARTIAL PASS**

The workflow has been used successfully multiple times during Project 01, but the distinction was not independently reproduced during the assessment.

Retention is not yet sufficient for independent status.

---

## Assessment 6 — Default Gateway

### Scenario

Homeserver:

- IP: `192.168.0.10`
- Default gateway: `192.168.0.1`

Question:

What is the default gateway used for?

### Response

Correctly explained that the default gateway is used for traffic destined outside the local network.

### Technical Model

Same local subnet:

`192.168.0.10 → local host`

Traffic can be delivered directly.

Different network:

`192.168.0.10 → 192.168.0.1 → other network`

### Evaluation

**PASS**

Core concept reproduced independently.

---

## Assessment 7 — Linux Services / Logs

### Scenario

A Linux service is not working.

Determine:

1. whether the service is running;
2. what recently happened to the service.

### Response

For service state:

The correct investigation domain was identified:

`systemctl`

The strategy was to use:

`systemctl --help`

to find the required operation instead of guessing syntax.

For historical/recent service information:

No independent investigation route was available.

### Evaluation

**PARTIAL PASS**

Positive evidence:

`problem → service/systemd → systemctl → documentation/help`

This demonstrates useful tool-discovery behavior rather than command memorization.

Linux journal/log investigation requires reinforcement.

---

## Assessment 8 — Kubernetes Service Backends

### Scenario

A Pod is Running and a Service exists, but the application does not work.

Question:

How can we determine whether the Service actually has a backend?

### Response

Correctly identified:

**Endpoints**

### Technical Model

`Service → selector → Endpoint/EndpointSlice → Pod IP:port`

A Service existing does not prove that it has a usable backend.

### Evaluation

**PASS**

The correct Kubernetes observation point was identified independently.

---

## Assessment 9 — Drift and GitOps

### Scenario

Git desired state:

`replicas = 1`

Live Kubernetes state:

`replicas = 2`

No GitOps controller exists.

### Response

Correctly identified the situation as:

**configuration drift**

The distinction between Kubernetes reconciliation and Git reconciliation was also correctly explained.

### Technical Model

Without GitOps:

`Git: replicas = 1`

does not automatically change:

`Kubernetes: replicas = 2`

Kubernetes reconciles against the desired state stored inside Kubernetes.

It does not automatically read Git.

Future GitOps model:

`Git → GitOps controller → Kubernetes Deployment → ReplicaSet → Pods`

### Evaluation

**PASS**

Important distinction between Kubernetes reconciliation and GitOps reconciliation understood.

---

## Assessment 10 — Recovery / Rebuild

### Scenario

The physical homeserver is permanently destroyed.

GitHub and the `homelab-ops` repository remain available.

Question:

Can the complete platform be restored from Git?

### Response

Correctly identified that the Kubernetes configuration can be recovered from the repository, but the replacement server must first be prepared.

Missing automated foundation includes:

- operating system installation/configuration;
- server configuration;
- networking;
- k3s installation;
- other platform prerequisites.

### Evaluation

**PASS**

Correctly recognized the boundary between:

**Kubernetes desired state**

and:

**complete infrastructure/server recovery**

Future projects must automate more of the server/platform foundation.

---

# Overall Assessment

## Strongest Evidence

Project 01 produced strong evidence in:

- Kubernetes desired state reasoning;
- Kubernetes reconciliation;
- Service endpoints;
- configuration drift;
- Git as source of truth;
- basic network routing concepts;
- evidence-first troubleshooting;
- identifying and removing orphaned resources;
- recognizing current recovery limitations.

---

## Skills Requiring Reinforcement

The following topics should return in future projects:

### Git

- `git diff` versus `git diff --staged`
- staged content versus working tree
- repeated independent Git workflow

### Linux

- systemd service investigation
- journal/log investigation
- connecting service state with historical evidence

### Kubernetes

- Deployment versus ReplicaSet responsibility
- Service types
- NodePort versus ClusterIP
- Service exposure and attack surface

### Networking

The complete:

`DNS → Traefik → Ingress → Service → Pod`

request path is recognizable and understandable when visible, but cannot yet be reconstructed independently.

This should not be memorized through repetition.

It should return naturally during future build projects.

---

# Practical Evidence from Project 01

Project 01 included real evidence from:

- Git commits;
- Kubernetes manifests;
- live cluster inspection;
- `kubectl diff`;
- deliberate Pod deletion;
- deliberate Service selector failure;
- deliberate Ingress drift;
- deliberate Deployment scaling failure;
- endpoint investigation;
- HTTP verification;
- DNS verification;
- incident documentation;
- orphaned resource investigation;
- NodePort removal;
- architecture documentation;
- recovery verification.

---

# Learning Decision

Project 01 does **not** need to be repeated.

The next project should increase in scope while deliberately reusing weaker Project 01 concepts.

The learning strategy remains:

`build → encounter old concept → investigate → apply → evidence → retention test`

rather than:

`memorize → repeat → memorize`

---

# Skill Promotion Decision

No skill is promoted to independent solely because Project 01 was completed.

Independent status requires:

- successful application;
- independent reproduction;
- application in another practical context;
- retention evidence after time has passed;
- ability to explain why the solution works.

Project 01 therefore provides important evidence toward future skill promotion but is not by itself sufficient proof of mastery.

---

# Project 01 Assessment Conclusion

**Project 01 assessment completed.**

The project successfully established the first working Platform Engineering foundation.

The main result is not only a functioning k3s platform, but demonstrated progress in:

- building;
- observing;
- troubleshooting;
- recovering;
- reviewing;
- documenting;
- reasoning from desired state.

## Gate Recommendation

**Proceed to Project 02.**

Project 02 should be larger than Project 01 but should not make a major difficulty jump.

It should introduce new platform capability while naturally requiring reuse of:

- Git workflow;
- Linux service/log investigation;
- Kubernetes Services;
- networking;
- troubleshooting;
- desired state;
- drift detection.

Weak areas from Project 01 should be reinforced through building rather than by repeating Project 01.
