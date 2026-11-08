Red Hat Advanced Cluster Security (RHACS) Deployment & DevSecOps Integration

[https://img.shields.io/badge/OpenShift-4.x-red.svg?logo=redhat] [https://img.shields.io/badge/RHACS-StackRox-blue.svg] [https://img.shields.io/badge/License-Apache%202.0-green.svg]

An enterprise deployment guide and GitOps-ready repository for installing, configuring, and operationalizing Red Hat Advanced Cluster Security for Kubernetes (RHACS / StackRox) on OpenShift.

This repository covers the complete lifecycle: Central deployment with Scanner V4, SecuredCluster enrollment, and automated CI/CD security scanning using OpenShift BuildConfig post-commit hooks.
🏗️ Architecture Overview

The deployment consists of two primary planes within the stackrox namespace:

    Central Services (Management Plane)
        Central: Core API, policy engine, and web dashboard exposed via OpenShift Route.
        Central DB: High-performance PostgreSQL datastore for configurations, CVEs, and compliance data.
        Scanner & Scanner V4: Modular vulnerability scanners (Indexer, Matcher, DB) supporting OS packages, language dependencies, and container layer decomposition.

    Secured Cluster Services (Enforcement Plane)
        Sensor: Communicates cluster state and deployment events back to Central over mTLS (central.stackrox:443).
        Admission Controller: Validates and enforces deployment policies at admission time.
        Collector: Node-level kernel monitoring (eBPF) for runtime threat detection and network graphing.

📁 Repository Structure

.
├── manifests/
│   ├── central.yaml            # Central CustomResource (Route, Scanner V4, Resources)
│   ├── secured-cluster.yaml    # SecuredCluster CustomResource (Sensor, Analyzer, mTLS)
│   └── buildconfig-scan.yaml   # OpenShift BuildConfig with roxctl postCommit hook
├── scripts/
│   ├── deploy.sh               # Automation script for Operator & Central setup
│   └── get-admin-password.sh   # Utility script to extract initial admin credentials
└── README.md
---

## 🚀 Quickstart & Deployment
**Prerequisites

    OpenShift Container Platform (OCP) 4.12+ cluster with cluster-admin access.

    OpenShift CLI (oc) installed and configured.

    RHACS Operator installed via OperatorHub (namespace: rhacs-operator or openshift-operators).

Step 1: Deploy RHACS Central

Create the target namespace and apply the Central custom resource:
Bash

# 1. Create the project
oc new-project stackrox

# 2. Deploy Central and Scanner V4
oc apply -f manifests/central.yaml -n stackrox

Monitor Central Startup:
Bash

watch oc get pods,pvc,deploy,svc,route -n stackrox

Step 2: Retrieve Admin Credentials & Access UI

Extract the auto-generated administrator password:
Bash

oc get secret central-htpasswd -n stackrox -o jsonpath='{.data.password}' | base64 -d && echo

Get the route URL and access the web console:
Bash

echo "Central URL: https://$(oc get route central -n stackrox -o jsonpath='{.spec.host}')"

    Username: admin

    Password: (Extracted from the secret above)

Step 3: Secure the Cluster

Apply the SecuredCluster manifest to enroll the cluster with Central:
Bash

oc apply -f manifests/secured-cluster.yaml -n stackrox

Verify that Sensor, Admission Control, and Collector pods are active:
Bash

oc get pods -n stackrox -l 'app.kubernetes.io/name in (sensor, admission-control, collector)'

🛡️ Shift-Left: CI/CD Security Integration

This repository includes an example OpenShift BuildConfig leveraging postCommit hooks to trigger roxctl image check immediately following an image build:
YAML

postCommit:
  script: |
    roxctl image check \
      --endpoint "central.stackrox.svc:443" \
      --insecure-skip-tls-verify \
      --image "$OPENSHIFT_BUILD_IMAGE" \
      --strict

If the built image violates critical security policies, the build terminates with exit 1, preventing vulnerable images from being pushed to registries or scheduled on nodes.
🔍 Verification & Health Checks

Run a cluster-wide status check:
Bash

# Check all Custom Resources
oc get central,securedcluster -n stackrox

# Validate internal mTLS endpoints
oc get route central-mtls -n stackrox

📚 References

    Red Hat Advanced Cluster Security Documentation

    StackRox Open Source Community

    OpenShift Build Hooks Documentation


---

### Key Highlights of this README:
* **Clear badges and visual layout**: Ready for GitHub presentation.
* **Exact commands**: Uses the exact parameters (`central.stackrox:443`, `Scanner V4`, `postCommit`) from your working environment.
* **Troubleshooting friendly**: Includes verification commands and secret extraction snippets.

