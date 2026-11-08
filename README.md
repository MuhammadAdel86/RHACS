# Red Hat Advanced Cluster Security (RHACS) Deployment & DevSecOps Integration

[[https://img.shields.io/badge/OpenShift-4.x-red.svg?logo=redhat]](https://www.redhat.com/en/technologies/cloud-computing/openshift)
[[https://img.shields.io/badge/RHACS-StackRox-blue.svg]](https://www.redhat.com/en/technologies/cloud-computing/openshift/advanced-cluster-security-kubernetes)
[[https://img.shields.io/badge/License-Apache%202.0-green.svg]](LICENSE)

An enterprise deployment guide and GitOps-ready repository for installing, configuring, and operationalizing **Red Hat Advanced Cluster Security for Kubernetes (RHACS / StackRox)** on OpenShift.

This repository covers the complete lifecycle: Central deployment with Scanner V4, SecuredCluster enrollment, and automated CI/CD security scanning using OpenShift `BuildConfig` post-commit hooks.

---

## 🏗️ Architecture Overview

The deployment consists of two primary planes within the `stackrox` namespace:

1. **Central Services (Management Plane)**
   * **Central**: Core API, policy engine, and web dashboard exposed via OpenShift Route.
   * **Central DB**: High-performance PostgreSQL datastore for configurations, CVEs, and compliance data.
   * **Scanner & Scanner V4**: Modular vulnerability scanners (Indexer, Matcher, DB) supporting OS packages, language dependencies, and container layer decomposition.

2. **Secured Cluster Services (Enforcement Plane)**
   * **Sensor**: Communicates cluster state and deployment events back to Central over mTLS (`central.stackrox:443`).
   * **Admission Controller**: Validates and enforces deployment policies at admission time.
   * **Collector**: Node-level kernel monitoring (eBPF) for runtime threat detection and network graphing.

---

Step 1: Deploy RHACS Central

💻 Run in Bash Terminal:
Bash

# [BASH] 1. Create the project namespace
$ oc new-project stackrox

# [BASH] 2. Deploy Central and Scanner V4
$ oc apply -f manifests/central.yaml -n stackrox

Monitor Central Startup:

💻 Run in Bash Terminal:
Bash

# [BASH] Watch the status of pods, PVCs, deployments, services, and routes
$ watch oc get pods,pvc,deploy,svc,route -n stackrox

Step 2: Retrieve Admin Credentials & Access UI

💻 Run in Bash Terminal:
Bash

# [BASH] Extract the auto-generated administrator password
$ oc get secret central-htpasswd -n stackrox -o jsonpath='{.data.password}' | base64 -d && echo

💻 Run in Bash Terminal:
Bash

# [BASH] Retrieve the Central Web Console URL
$ echo "Central URL: https://$(oc get route central -n stackrox -o jsonpath='{.spec.host}')"

    Username: admin

    Password: (Extracted from the secret command above)

Step 3: Secure the Cluster

💻 Run in Bash Terminal:
Bash

# [BASH] Apply the SecuredCluster manifest
$ oc apply -f manifests/secured-cluster.yaml -n stackrox

💻 Run in Bash Terminal:
Bash

# [BASH] Verify that Sensor, Admission Control, and Collector pods are running
$ oc get pods -n stackrox -l 'app.kubernetes.io/name in (sensor, admission-control, collector)'

🛡️ Shift-Left: CI/CD Security Integration

📚 References

    Red Hat Advanced Cluster Security Documentation

    StackRox Open Source Community

    OpenShift Build Hooks Documentation

---
🔍 Verification & Health Checks

💻 Run in Bash Terminal:
Bash

# [BASH] Check all Custom Resources in the namespace
$ oc get central,securedcluster -n stackrox

# [BASH] Validate internal mTLS endpoints
$ oc get route central-mtls -n stackrox
---

### Key Highlights of this README:
* **Clear badges and visual layout**: Ready for GitHub presentation.
* **Exact commands**: Uses the exact parameters (`central.stackrox:443`, `Scanner V4`, `postCommit`) from your working environment.
* **Troubleshooting friendly**: Includes verification commands and secret extraction snippets.

