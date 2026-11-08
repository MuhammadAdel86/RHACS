# Red Hat Advanced Cluster Security (RHACS) Deployment & DevSecOps Integration

[](https://www.redhat.com/en/technologies/cloud-computing/openshift)
[](https://www.redhat.com/en/technologies/cloud-computing/openshift/advanced-cluster-security-kubernetes)
[License](https://www.google.com/search?q=LICENSE)

An enterprise deployment guide and GitOps-ready repository for installing, configuring, and operationalizing **Red Hat Advanced Cluster Security for Kubernetes (RHACS / StackRox)** on OpenShift.

This repository covers the complete lifecycle: Central deployment with Scanner V4, SecuredCluster enrollment, and automated CI/CD security scanning using OpenShift `BuildConfig` post-commit hooks.

---

## 🏗️ Architecture Overview

The deployment consists of two primary planes within the `stackrox` namespace:

### 1. Central Services (Management Plane)
* **Central**: Core API, policy engine, and web dashboard exposed via OpenShift Route.
* **Central DB**: High-performance PostgreSQL datastore for configurations, CVEs, and compliance data.
* **Scanner & Scanner V4**: Modular vulnerability scanners (Indexer, Matcher, DB) supporting OS packages, language dependencies, and container layer decomposition.

### 2. Secured Cluster Services (Enforcement Plane)
* **Sensor**: Communicates cluster state and deployment events back to Central over mTLS (`central.stackrox:443`).
* **Admission Controller**: Validates and enforces deployment policies at admission time.
* **Collector**: Node-level kernel monitoring (eBPF) for runtime threat detection and network graphing.


---

## 🚀 Quickstart & Deployment

### Prerequisites
* OpenShift Container Platform (OCP) 4.12+ cluster with cluster-admin access.
* OpenShift CLI (`oc`) installed and configured.
* RHACS Operator installed via OperatorHub (`rhacs-operator` or `openshift-operators`).

---

### Step 1: Deploy RHACS Central

Create the `stackrox` project:

```bash
oc new-project stackrox
```

Apply the Central Custom Resource:

```bash
oc apply -f manifests/central.yaml -n stackrox
```

Monitor the deployment rollout in real time:

```bash
watch oc get pods,pvc,deploy,svc,route -n stackrox
```

---

### Step 2: Retrieve Admin Credentials & Access UI

Extract the auto-generated administrator password:

```bash
oc get secret central-htpasswd -n stackrox -o jsonpath='{.data.password}' | base64 -d && echo
```

Print the external Central web console route:

```bash
echo "Central URL: https://$(oc get route central -n stackrox -o jsonpath='{.spec.host}')"
```

* **Default Username**: `admin`
* **Password**: *(Copied from the command above)*

---

### Step 3: Secure the Cluster (SecuredCluster)

Apply the SecuredCluster Custom Resource to enroll the cluster:

```bash
oc apply -f manifests/secured-cluster.yaml -n stackrox
```

Verify that Sensor, Admission Control, and Collector pods are active:

```bash
oc get pods -n stackrox -l 'app.kubernetes.io/name in (sensor, admission-control, collector)'
```


## 🔍 Verification & Health Checks

Check the status of Central and SecuredCluster resources:

```bash
oc get central,securedcluster -n stackrox
```

Verify the internal mTLS route for Sensor-to-Central connectivity:

```bash
oc get route central-mtls -n stackrox
```

---

## 📚 References

* [Red Hat Advanced Cluster Security Documentation](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_security_for_kubernetes)
* [StackRox Open Source Community](https://www.stackrox.io/)
* [OpenShift Build Hooks Documentation](https://docs.openshift.com/container-platform/latest/cicd/builds/triggering-builds-build-hooks.html)
