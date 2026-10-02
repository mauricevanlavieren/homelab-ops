# Incident: ...

## Issue
23 Issue: failure to reach web.home.arpa on server from browser

---

## Investigation
steps to isolate and pinpoint the issue;
* **1 test network dns and ip via curl**
  * curl response dns 503 and ip 404
* **2 test network again with resolvectl for more detailed info**
  * resolvectl response ip and host ok
* **3 due to test 1 and 2 the origin of the issue could be reduced to an internal kubernets issue**
* **4 review the service, deployments, ingress**
* **5 checking endpoints, the inquiry revealed the missing endpoint of the pod in namespace web.**
  * next step check status of pods in namespace web
* **6 the status of the pod in namespace web was 0/0**

---

## Root Cause
* **the live deployment replicas was set to 0 instead of 1, causing missing endpoints and a 503 error**

---

## Recovery
* **7 verified with the git as source of truth the desired state.**
  * according to the git docs the desired state of the deployment is 1 replica
* **8 verified the deployment manifest agains git. these where identical**
* **9 restored the deployment according the desired state**

---

## Verification
* **10 tested the browser and curl to make sure the issue was solved**
* **11 tests positive - issue solved**

