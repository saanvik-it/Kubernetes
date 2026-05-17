# ☸️ Kubernetes Glossary & Interview Preparation Guide

A comprehensive reference of Kubernetes terms, concepts, and definitions — organized by category. Ideal for interview prep, onboarding, and day-to-day reference.

---

## 🗺️ Architecture Diagram

![Kubernetes Cluster Architecture](./kubernetes-architecture.svg)

> Full cluster view — control plane, worker nodes, networking, storage, config, and security layers.

---

## 📑 Table of Contents

- [Core Architecture](#-core-architecture)
- [Workloads](#-workloads)
- [Networking](#-networking)
- [Storage](#-storage)
- [Configuration & Scaling](#-configuration--scaling)
- [Security & RBAC](#-security--rbac)
- [Ops & Tooling](#-ops--tooling)
- [Observability & Interfaces](#-observability--interfaces)
- [Pod Troubleshooting Guide](#-pod-troubleshooting-guide)
- [Interview Tips](#-interview-tips)

---

## 🏗️ Core Architecture

| Term | Abbreviation | Definition |
|------|-------------|------------|
| **Cluster** | — | A set of nodes (machines) that run containerized applications managed by Kubernetes. |
| **Node** | — | A physical or virtual machine in the cluster that runs Pods. Can be a control plane or worker node. |
| **Control Plane** | — | The set of components that manage the cluster — API Server, etcd, Scheduler, and Controller Manager. |
| **Worker Node** | — | A node that runs application workloads (Pods). Managed by the control plane. |
| **API Server** | `kube-apiserver` | The front-end of the control plane. Exposes the Kubernetes REST API and is the entry point for all commands. |
| **etcd** | — | A consistent, highly-available key-value store used to store all cluster configuration and state data. |
| **Scheduler** | `kube-scheduler` | Watches for newly created Pods and assigns them to a suitable node based on resource requirements and constraints. |
| **Controller Manager** | `kube-controller-manager` | Runs controller loops that watch the cluster state and make changes to bring actual state to desired state. |
| **Kubelet** | — | An agent on each worker node that ensures containers in a Pod are running and healthy. |
| **Kube-proxy** | — | A network proxy on each node that maintains network rules for communication to Pods from inside or outside the cluster. |
| **Namespace** | `ns` | A virtual cluster within a Kubernetes cluster used to isolate resources and workloads. |
| **kubectl** | — | The command-line tool used to interact with and manage Kubernetes clusters. |

---

## 📦 Workloads

| Term | Abbreviation | Definition |
|------|-------------|------------|
| **Pod** | `po` | The smallest deployable unit in Kubernetes. A Pod wraps one or more containers that share network and storage. |
| **Deployment** | `deploy` | Manages a ReplicaSet to ensure a specified number of identical Pods are running. Supports rolling updates and rollbacks. |
| **ReplicaSet** | `rs` | Ensures a specified number of Pod replicas are running at any given time. Usually managed by a Deployment. |
| **StatefulSet** | `sts` | Manages stateful applications. Provides stable network identities and persistent storage for each Pod. |
| **DaemonSet** | `ds` | Ensures a copy of a Pod runs on all (or selected) nodes. Used for cluster-wide services like log collectors. |
| **Job** | — | Creates one or more Pods and ensures they run to completion successfully. Ideal for batch tasks. |
| **CronJob** | `cj` | Schedules Jobs to run at specific times or intervals, similar to a Unix cron. |
| **ReplicationController** | `rc` | Legacy resource that ensures a specified number of Pod replicas are running. Superseded by ReplicaSet. |
| **Init Container** | — | A special container that runs before app containers in a Pod, used for setup tasks. |
| **Sidecar Container** | — | A helper container running alongside the main app container in the same Pod, sharing its lifecycle. |

---

## 🌐 Networking

| Term | Abbreviation | Definition |
|------|-------------|------------|
| **Service** | `svc` | An abstraction that exposes a set of Pods as a stable network endpoint with a consistent IP and DNS name. |
| **ClusterIP** | — | The default Service type. Exposes the Service on a cluster-internal IP — only reachable within the cluster. |
| **NodePort** | — | Exposes a Service on a static port on each node's IP, making it accessible from outside the cluster. |
| **LoadBalancer** | `lb` | Exposes a Service externally using a cloud provider's load balancer. |
| **ExternalName** | — | A Service type that maps a Service to a DNS name (e.g., an external database) instead of a selector. |
| **Ingress** | `ing` | Manages external HTTP/HTTPS access to Services, providing routing rules, TLS termination, and virtual hosting. |
| **Ingress Controller** | — | A controller that implements Ingress rules (e.g., NGINX, Traefik). Required for Ingress resources to work. |
| **NetworkPolicy** | `netpol` | Specifies how groups of Pods are allowed to communicate with each other and with external endpoints. |
| **Endpoint** | `ep` | Lists the IP addresses and ports of Pods backing a Service. Auto-managed by Kubernetes. |
| **DNS (CoreDNS)** | — | Kubernetes includes a built-in DNS service (CoreDNS) that assigns DNS names to Services and Pods. |

---

## 💾 Storage

| Term | Abbreviation | Definition |
|------|-------------|------------|
| **Volume** | — | A directory accessible to containers in a Pod. Survives container restarts but not Pod deletion (unless persistent). |
| **PersistentVolume** | `PV` | A piece of storage in the cluster provisioned by an admin or dynamically. Independent of any Pod lifecycle. |
| **PersistentVolumeClaim** | `PVC` | A request for storage by a user. Binds to a matching PersistentVolume. |
| **StorageClass** | `sc` | Describes different classes of storage (e.g., SSD vs HDD). Enables dynamic provisioning of PersistentVolumes. |
| **emptyDir** | — | A temporary directory created when a Pod is assigned to a node. Deleted when the Pod is removed. |
| **hostPath** | — | Mounts a file or directory from the host node's filesystem into a Pod. |
| **ConfigMap Volume** | — | Mounts ConfigMap data as files into a Pod's filesystem. |

---

## ⚙️ Configuration & Scaling

| Term | Abbreviation | Definition |
|------|-------------|------------|
| **ConfigMap** | `cm` | Stores non-confidential configuration data as key-value pairs, injected into Pods as env vars or files. |
| **Secret** | — | Stores sensitive data (passwords, tokens) in base64 encoding. Injected into Pods similar to ConfigMaps. |
| **ResourceQuota** | `quota` | Limits total resource consumption (CPU, memory, object count) per namespace. |
| **LimitRange** | — | Enforces default and maximum resource limits for Pods and containers within a namespace. |
| **HorizontalPodAutoscaler** | `HPA` | Automatically scales the number of Pod replicas based on observed CPU/memory or custom metrics. |
| **VerticalPodAutoscaler** | `VPA` | Automatically adjusts CPU and memory requests for containers based on historical usage. |
| **PodDisruptionBudget** | `PDB` | Limits the number of Pods that can be down simultaneously during voluntary disruptions. |
| **PriorityClass** | `pc` | Assigns a priority to Pods to control scheduling order and eviction during resource pressure. |

---

## 🔐 Security & RBAC

| Term | Abbreviation | Definition |
|------|-------------|------------|
| **ServiceAccount** | `sa` | Provides an identity for processes running in a Pod to authenticate with the Kubernetes API. |
| **RBAC** | Role-Based Access Control | An authorization mechanism that regulates access to Kubernetes resources based on roles assigned to users. |
| **Role** | — | A set of permissions (rules) within a namespace. Used in RBAC. |
| **ClusterRole** | — | A set of permissions that apply cluster-wide or can be used in any namespace. |
| **RoleBinding** | `rb` | Binds a Role to a user, group, or ServiceAccount within a namespace. |
| **ClusterRoleBinding** | `crb` | Binds a ClusterRole to a user, group, or ServiceAccount cluster-wide. |
| **PodSecurityContext** | — | Defines security settings (user ID, file permissions) applied at the Pod level. |
| **SecurityContext** | — | Defines security settings for individual containers, such as running as non-root or read-only filesystem. |
| **Admission Controller** | — | A plugin that intercepts API requests after authentication to validate or mutate them before persisting. |

---

## 🛠️ Ops & Tooling

| Term | Abbreviation | Definition |
|------|-------------|------------|
| **Helm** | — | A package manager for Kubernetes. Charts bundle Kubernetes resources into reusable, configurable packages. |
| **Helm Chart** | — | A collection of Kubernetes manifest templates packaged together for deployment via Helm. |
| **Kustomize** | — | A tool to customize Kubernetes manifests without templating, using overlays and patches. |
| **Operator** | — | A pattern that extends Kubernetes to manage complex stateful apps using custom controllers and CRDs. |
| **CustomResourceDefinition** | `CRD` | Extends the Kubernetes API by defining new resource types specific to an application. |
| **Custom Resource** | `CR` | An instance of a CRD — a user-defined object managed by the Kubernetes API. |
| **Taint** | — | Applied to a node to repel Pods that don't explicitly tolerate it. Used for node isolation. |
| **Toleration** | — | Applied to a Pod to allow it to be scheduled on nodes with matching taints. |
| **Affinity** | — | Rules that attract Pods to specific nodes or other Pods based on labels. |
| **Anti-affinity** | — | Rules that repel Pods from being placed on the same nodes as certain other Pods. |
| **Label** | — | Key-value pairs attached to objects for identification and selection. |
| **Annotation** | — | Key-value pairs attached to objects to store non-identifying metadata (e.g., version info, tool config). |
| **Selector** | — | A query that filters Kubernetes objects by their labels. |

---

## 📊 Observability & Interfaces

| Term | Abbreviation | Definition |
|------|-------------|------------|
| **Liveness Probe** | — | Checks if a container is still running. Restarts it if the probe fails. |
| **Readiness Probe** | — | Checks if a container is ready to receive traffic. Removes it from Service endpoints if it fails. |
| **Startup Probe** | — | Checks if an app within a container has started. Disables liveness/readiness probes until it succeeds. |
| **Metrics Server** | — | A cluster-wide aggregator of resource usage data (CPU, memory) used by HPA and `kubectl top`. |
| **Container Runtime** | — | Software responsible for running containers (e.g., containerd, CRI-O). Implements the CRI interface. |
| **CRI** | Container Runtime Interface | A plugin interface that enables the kubelet to use different container runtimes. |
| **CNI** | Container Network Interface | A standard for configuring network interfaces in Linux containers. Plugins like Flannel and Calico implement it. |
| **CSI** | Container Storage Interface | A standard for exposing storage systems to containerized workloads on Kubernetes. |

---

## 🔧 Pod Troubleshooting Guide

A practical reference for diagnosing and fixing the most common Pod issues in Kubernetes.

---

### Issue 1 — Pod stuck in `Pending` state

**Symptoms**
```
NAME        READY   STATUS    RESTARTS   AGE
my-pod      0/1     Pending   0          5m
```

**Common causes**
- Insufficient CPU or memory on all nodes
- No node matches the Pod's `nodeSelector` or `affinity` rules
- Node has a taint that the Pod does not tolerate
- PersistentVolumeClaim is unbound (no matching PV)

**Diagnosis**
```bash
kubectl describe pod <pod-name>
# Look for "Events" section at the bottom — it shows the exact reason
# Common messages:
#   "0/3 nodes are available: 3 Insufficient memory"
#   "0/3 nodes are available: 3 node(s) had taint..."
#   "persistentvolumeclaim <name> not found"
```

**Fix**
```bash
# Check node resource availability
kubectl describe nodes | grep -A5 "Allocated resources"

# Check if PVC is bound
kubectl get pvc -n <namespace>

# Check taints on nodes
kubectl describe node <node-name> | grep Taints
```

---

### Issue 2 — Pod in `CrashLoopBackOff`

**Symptoms**
```
NAME        READY   STATUS             RESTARTS   AGE
my-pod      0/1     CrashLoopBackOff   8          12m
```

**Common causes**
- Application error or uncaught exception at startup
- Missing environment variable or config
- Incorrect command/entrypoint in the container spec
- Liveness probe failing immediately

**Diagnosis**
```bash
# Check logs from the current (crashing) container
kubectl logs <pod-name>

# Check logs from the previous crashed container
kubectl logs <pod-name> --previous

# Describe for probe and restart details
kubectl describe pod <pod-name>
```

**Fix**
```bash
# Override the entrypoint to debug interactively
kubectl run debug --image=<your-image> -it --restart=Never -- /bin/sh

# Check env vars are correctly set
kubectl exec -it <pod-name> -- env | grep MY_VAR

# If liveness probe is too aggressive, increase initialDelaySeconds:
# livenessProbe:
#   initialDelaySeconds: 30   # give the app time to start
```

---

### Issue 3 — Pod stuck in `ImagePullBackOff` or `ErrImagePull`

**Symptoms**
```
NAME        READY   STATUS             RESTARTS   AGE
my-pod      0/1     ImagePullBackOff   0          3m
```

**Common causes**
- Image name or tag is incorrect (typo, wrong registry)
- Image is private and no pull secret is configured
- Registry is unreachable from the cluster network
- Image tag does not exist (e.g., using `latest` that was never pushed)

**Diagnosis**
```bash
kubectl describe pod <pod-name>
# Events will show:
#   "Failed to pull image: rpc error: ... not found"
#   "Failed to pull image: ... unauthorized: authentication required"
```

**Fix**
```bash
# Verify the image exists and tag is correct
docker pull <image>:<tag>

# Create an image pull secret for private registries
kubectl create secret docker-registry regcred \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>

# Reference the secret in the Pod spec:
# spec:
#   imagePullSecrets:
#     - name: regcred
```

---

### Issue 4 — Pod in `OOMKilled` (Out of Memory)

**Symptoms**
```
NAME        READY   STATUS      RESTARTS   AGE
my-pod      0/1     OOMKilled   3          8m
```
```bash
# kubectl describe output shows:
# Last State: Terminated  Reason: OOMKilled
```

**Common causes**
- Container memory limit is set too low
- Application has a memory leak
- Sudden traffic spike causing high memory usage

**Diagnosis**
```bash
kubectl describe pod <pod-name>
# Look for: "Last State: Terminated  Reason: OOMKilled  Exit Code: 137"

# Check current memory usage
kubectl top pod <pod-name>
kubectl top pod <pod-name> --containers
```

**Fix**
```yaml
# Increase memory limit in your manifest:
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"   # increase this value

# Apply the change
kubectl apply -f deployment.yaml
```

---

### Issue 5 — Pod in `Error` state (Exit Code issues)

**Symptoms**
```
NAME        READY   STATUS   RESTARTS   AGE
my-pod      0/1     Error    0          1m
```

**Common exit codes**

| Exit Code | Meaning |
|-----------|---------|
| `0` | Success — container completed normally |
| `1` | General application error |
| `2` | Misuse of shell command |
| `126` | Command found but not executable |
| `127` | Command not found |
| `128` | Invalid exit argument |
| `137` | Container killed (SIGKILL) — often OOMKill |
| `143` | Graceful termination (SIGTERM) |

**Diagnosis**
```bash
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
# Check "Exit Code" under "Last State"
```

**Fix**
```bash
# For exit code 127 (command not found) — check entrypoint:
kubectl run test --image=<your-image> -it --restart=Never -- which myapp

# For exit code 1 — check application logs for the actual error
kubectl logs <pod-name> --previous | tail -50
```

---

### Issue 6 — Pod not receiving traffic (`Endpoints` empty)

**Symptoms**
- Service exists but requests time out or return connection refused
- Application is running but unreachable

**Common causes**
- Service `selector` labels do not match Pod labels
- Pod's `containerPort` does not match Service `targetPort`
- Readiness probe is failing — Pod is Running but not Ready

**Diagnosis**
```bash
# Check if endpoints are populated
kubectl get endpoints <service-name>
# If ADDRESS column is empty — selector mismatch or pod not Ready

# Compare service selector vs pod labels
kubectl get svc <service-name> -o yaml | grep selector -A5
kubectl get pod <pod-name> --show-labels

# Check readiness
kubectl describe pod <pod-name> | grep -A10 "Readiness"
```

**Fix**
```yaml
# Ensure labels match exactly. Example:
# Service selector:
selector:
  app: my-app
  tier: backend

# Pod labels must include BOTH:
metadata:
  labels:
    app: my-app
    tier: backend
```

---

### Issue 7 — Pod evicted due to node pressure

**Symptoms**
```
NAME        READY   STATUS    RESTARTS   AGE
my-pod      0/1     Evicted   0          30m
```

**Common causes**
- Node is under disk pressure (`DiskPressure`)
- Node is under memory pressure (`MemoryPressure`)
- Node PID pressure (`PIDPressure`)
- Low-priority Pods are evicted to make room for high-priority ones

**Diagnosis**
```bash
kubectl describe pod <pod-name>
# Message: "The node was low on resource: memory. Threshold quantity: 100Mi"

# Check node conditions
kubectl describe node <node-name> | grep -A10 "Conditions"

# Check node disk usage
kubectl exec -it <any-pod> -- df -h
```

**Fix**
```bash
# Delete the evicted pod (it won't self-heal)
kubectl delete pod <pod-name>

# Free up disk space on the node (if disk pressure)
# Remove unused images:
docker image prune -a

# Set proper resource requests to trigger proper scheduling:
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"

# Assign a PriorityClass to protect critical pods from eviction
```

---

### Issue 8 — `Init:Error` or `Init:CrashLoopBackOff`

**Symptoms**
```
NAME        READY   STATUS              RESTARTS   AGE
my-pod      0/1     Init:CrashLoopBackOff  3       5m
```

**Common causes**
- Init container fails to complete (non-zero exit code)
- Init container is waiting for a dependency (DB, config service) that is unavailable
- Wrong command or missing binary in the init container image

**Diagnosis**
```bash
# List init containers
kubectl describe pod <pod-name> | grep -A5 "Init Containers"

# Get logs from the failing init container
kubectl logs <pod-name> -c <init-container-name>

# Get logs from previous failed run
kubectl logs <pod-name> -c <init-container-name> --previous
```

**Fix**
```yaml
# Common pattern: wait for a service to be ready before starting
initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c',
      'until nc -z db-service 5432; do echo waiting for db; sleep 2; done']
```

---

### Issue 9 — Pod restarts repeatedly but shows `Running`

**Symptoms**
```
NAME        READY   STATUS    RESTARTS   AGE
my-pod      1/1     Running   47         2h
```

**Common causes**
- Liveness probe is too sensitive (low timeout, aggressive thresholds)
- Application deadlocks after some time and stops responding
- Memory leak causing periodic OOM kills
- External dependency flapping

**Diagnosis**
```bash
# Check restart count and last termination reason
kubectl describe pod <pod-name>
# Look for: "Last State: Terminated  Reason: Error / OOMKilled"

# Watch restarts in real-time
kubectl get pod <pod-name> -w

# Check logs around the time of restart
kubectl logs <pod-name> --previous --timestamps | tail -100
```

**Fix**
```yaml
# Tune liveness probe to be less aggressive:
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 60   # wait before first check
  periodSeconds: 30          # check every 30s (not every 5s)
  failureThreshold: 3        # allow 3 failures before restart
  timeoutSeconds: 5          # give app 5s to respond
```

---

### Issue 10 — Pod stuck in `Terminating` state

**Symptoms**
```
NAME        READY   STATUS        RESTARTS   AGE
my-pod      1/1     Terminating   0          2h
```

**Common causes**
- Application is not handling `SIGTERM` and not shutting down within `terminationGracePeriodSeconds`
- A finalizer is blocking deletion
- Node is unreachable (kubelet cannot confirm termination)

**Diagnosis**
```bash
kubectl describe pod <pod-name>
# Check for finalizers
kubectl get pod <pod-name> -o yaml | grep finalizers -A5

# Check node status
kubectl get node <node-name>
```

**Fix**
```bash
# Force delete when node is unreachable or finalizer is stuck
kubectl delete pod <pod-name> --force --grace-period=0

# Remove a stuck finalizer manually
kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":[]}}' --type=merge

# Fix app code: handle SIGTERM gracefully
# In your app, listen for SIGTERM and shut down cleanly within
# terminationGracePeriodSeconds (default: 30s)
```

---

### 🩺 General Troubleshooting Workflow

```bash
# Step 1 — Check pod status
kubectl get pods -n <namespace>

# Step 2 — Describe the pod for events and state
kubectl describe pod <pod-name> -n <namespace>

# Step 3 — Check current logs
kubectl logs <pod-name> -n <namespace>

# Step 4 — Check previous container logs (if restarted)
kubectl logs <pod-name> --previous -n <namespace>

# Step 5 — Exec into the pod for interactive debugging
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh

# Step 6 — Check cluster events sorted by time
kubectl get events -n <namespace> --sort-by='.lastTimestamp'

# Step 7 — Check node health
kubectl get nodes
kubectl describe node <node-name>

# Step 8 — Check resource usage
kubectl top pods -n <namespace>
kubectl top nodes
```

---

### 📋 Pod Status Quick Reference

| Status | Meaning | Where to look |
|--------|---------|---------------|
| `Pending` | Not scheduled yet | Node resources, taints, PVC |
| `Running` | At least one container is running | Logs, probes |
| `CrashLoopBackOff` | Container crashes and restarts repeatedly | `logs --previous`, entrypoint |
| `ImagePullBackOff` | Cannot pull the container image | Image name, pull secret |
| `OOMKilled` | Killed due to exceeding memory limit | Memory limits, leaks |
| `Error` | Container exited with non-zero code | `logs --previous`, exit code |
| `Evicted` | Removed by kubelet due to node pressure | Node disk/memory, events |
| `Terminating` | Deletion in progress | Finalizers, node status |
| `Init:Error` | Init container failed | Init container logs |
| `Completed` | All containers finished successfully | Normal for Jobs |
| `Unknown` | Cannot communicate with the node | Node status, kubelet |

---

## 🎯 Interview Tips

### Common interview question areas

**Architecture**
- Explain the difference between the control plane and worker nodes.
- What happens when you run `kubectl apply -f deployment.yaml`?
- How does the Kubernetes Scheduler decide where to place a Pod?

**Workloads**
- When would you use a StatefulSet over a Deployment?
- What is the difference between a Job and a CronJob?
- What are Init Containers used for?

**Networking**
- Explain the difference between ClusterIP, NodePort, and LoadBalancer.
- What is an Ingress and why do you need an Ingress Controller?
- How does Kubernetes DNS work for service discovery?

**Storage**
- What is the difference between a PersistentVolume and a PersistentVolumeClaim?
- When would you use `emptyDir` vs a PVC?

**Security**
- What is RBAC and how does it work in Kubernetes?
- What is the difference between a Role and a ClusterRole?
- Why should you avoid running containers as root?

**Scaling & Config**
- How does HPA work? What metrics can it use?
- What is the difference between a ConfigMap and a Secret?
- What is a PodDisruptionBudget and when would you use it?

### Quick `kubectl` cheat sheet

```bash
# Get resources
kubectl get pods -n <namespace>
kubectl get all -n <namespace>
kubectl describe pod <pod-name>

# Apply / delete
kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml

# Logs & exec
kubectl logs <pod-name> -c <container-name>
kubectl exec -it <pod-name> -- /bin/sh

# Scaling
kubectl scale deployment <name> --replicas=3

# Context & namespace
kubectl config get-contexts
kubectl config use-context <context>
kubectl config set-context --current --namespace=<namespace>

# Debugging
kubectl get events --sort-by='.lastTimestamp'
kubectl top pods
kubectl top nodes
```

---

## 📚 Resources

- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Patterns Book](https://www.oreilly.com/library/view/kubernetes-patterns/9781492050278/)
- [CKA Exam Curriculum](https://github.com/cncf/curriculum)

---

> ⭐ Star this repo if you found it helpful! Contributions and corrections are welcome via pull requests.
