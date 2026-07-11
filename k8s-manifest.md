# Kubernetes Manifests — Interview Preparation Guide

A reference of every commonly asked Kubernetes object `kind`, with the anatomy explained, a minimal working YAML example, and the interview angle you're likely to get quizzed on.

---

## 1. How to Read Any Manifest (the 4 required fields)

Every Kubernetes manifest has the same skeleton:

```yaml
apiVersion: <group/version>   # which API this object belongs to
kind: <ObjectType>            # what kind of object (Pod, Deployment, Service...)
metadata:                     # name, namespace, labels, annotations
  name: my-object
  labels:
    app: my-app
spec:                         # desired state (object-specific)
  ...
```

**Interview tip:** Interviewers love asking "what's the difference between `spec` and `status`?"
> `spec` = desired state (you write it). `status` = current observed state (the cluster writes it, via controllers reconciling actual vs desired — this reconciliation loop is the core idea behind *everything* in K8s).

---

## 2. Pod

The smallest deployable unit — one or more containers sharing network/storage.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "250m"
          memory: "256Mi"
```

**Interview tip:** "Why don't we deploy bare Pods in production?"
> No self-healing — if a Pod dies, nothing recreates it. That's why we wrap Pods in a Deployment/StatefulSet/DaemonSet/Job, which use a **controller** to maintain the desired replica count.

---

## 3. ReplicaSet

Ensures N identical Pod replicas are running. Rarely created directly — Deployments manage ReplicaSets for you.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
```

**Interview tip:** "Deployment vs ReplicaSet?"
> A Deployment *manages* ReplicaSets and adds rolling updates, rollback, and revision history. A ReplicaSet alone has no update strategy — change the image and nothing happens until Pods are manually deleted.

---

## 4. Deployment

The standard way to run stateless applications. Manages ReplicaSets, supports rolling updates and rollbacks.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 15
```

**Interview tips:**
- "How does a rolling update actually work?" → New ReplicaSet is created, scaled up gradually while old one scales down, governed by `maxSurge`/`maxUnavailable`.
- "How do you rollback?" → `kubectl rollout undo deployment/nginx-deployment` — uses ReplicaSet revision history (`revisionHistoryLimit`).
- "Readiness vs Liveness probe?" → Readiness = "should this Pod receive traffic?" (removes from Service endpoints if failing). Liveness = "should this container be restarted?" (kubelet kills and restarts container if failing).

---

## 5. StatefulSet

For stateful apps needing stable network identity and stable storage (databases, Kafka, etc.).

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

**Interview tips:**
- Pods get stable, predictable names: `mysql-0`, `mysql-1`, `mysql-2` (not random hashes like Deployments).
- Each replica gets its **own** PVC via `volumeClaimTemplates` (Deployments share nothing like this).
- Requires a **headless Service** (`clusterIP: None`) for stable DNS per-Pod: `mysql-0.mysql-headless.default.svc.cluster.local`.
- Pods are created/deleted in order (0, 1, 2... and reverse for deletion) — important for clustered apps like etcd/Kafka.

---

## 6. DaemonSet

Ensures exactly one Pod runs on every (or selected) node. Used for node-level agents.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
        - name: fluentd
          image: fluentd:v1.16
          resources:
            limits:
              memory: 200Mi
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          effect: NoSchedule
```

**Interview tip:** Classic real-world examples — log collectors (Fluentd/Filebeat), monitoring agents (node-exporter), CNI plugins (Calico/Cilium). Notice the `tolerations` — needed if you want the DaemonSet Pod to also run on control-plane/master nodes (which are normally tainted).

---

## 7. Job

Runs a Pod to completion, for one-off or batch work.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  completions: 1
  backoffLimit: 4
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: migrate/migrate
          command: ["migrate", "-path", "/migrations", "-database", "$(DB_URL)", "up"]
```

**Interview tip:** `restartPolicy` for a Job must be `Never` or `OnFailure` (never `Always`). `backoffLimit` caps retries before marking the Job failed. `parallelism` + `completions` control how many Pods run concurrently vs total needed.

---

## 8. CronJob

Runs a Job on a schedule.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"          # 2 AM daily, standard cron syntax
  concurrencyPolicy: Forbid       # don't overlap runs
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: backup-tool:latest
```

**Interview tip:** `concurrencyPolicy: Allow | Forbid | Replace` is a favorite question — "what happens if a job is still running when the next schedule fires?"

---

## 9. Service

Stable networking endpoint for a set of Pods (selected via labels).

### ClusterIP (default — internal only)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

### NodePort (exposes on every node's IP at a static port 30000-32767)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### LoadBalancer (provisions cloud LB — AWS ELB, Azure LB, etc.)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

### Headless (for StatefulSets — no ClusterIP, direct Pod DNS)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
```

**Interview tips:**
- `port` = the Service's own port. `targetPort` = the container's port. They can differ.
- "How does a Service find its Pods?" → label selector matching, tracked via **Endpoints/EndpointSlice** objects.
- Service types are cumulative in capability: `ClusterIP` ⊂ `NodePort` ⊂ `LoadBalancer` (each builds on the previous).
- kube-proxy implements Service routing via iptables or IPVS rules on each node.

---

## 10. Ingress

HTTP(S) routing layer sitting in front of Services — host/path-based routing, TLS termination. Requires an Ingress Controller (NGINX, Traefik, AGIC on Azure, ALB on AWS) to actually do anything.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
    - hosts: ["app.example.com"]
      secretName: app-tls-secret
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-svc
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-svc
                port:
                  number: 80
```

**Interview tip:** "Ingress vs LoadBalancer Service?" → LoadBalancer Service = 1 external IP per Service, layer 4. Ingress = 1 entrypoint for many Services, layer 7 (host/path routing, TLS), cheaper at scale (one cloud LB instead of many).

---

## 11. ConfigMap

Non-sensitive configuration data, decoupled from image/Pod spec.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
  app.properties: |
    server.port=8080
    server.timeout=30
```

Consuming it in a Pod:
```yaml
      containers:
        - name: app
          image: myapp:1.0
          envFrom:
            - configMapRef:
                name: app-config
          volumeMounts:
            - name: config-vol
              mountPath: /etc/config
      volumes:
        - name: config-vol
          configMap:
            name: app-config
```

**Interview tip:** "How do Pods pick up ConfigMap changes?" → Env vars injected via `envFrom`/`env` do **not** auto-update (Pod restart needed). Volume-mounted ConfigMaps **do** auto-update (with a sync delay, usually within ~1 minute), because kubelet periodically re-syncs the mounted file.

---

## 12. Secret

Same idea as ConfigMap but for sensitive data (base64-encoded at rest, not encrypted by default).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=        # base64 for "admin"
  password: cGFzc3dvcmQxMjM=
```

Or let kubectl encode it for you:
```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=password123
```

**Interview tips:**
- base64 is **encoding**, not encryption — anyone with API access can decode it. Real security needs **encryption at rest** (etcd encryption) + RBAC restricting who can `get`/`list` Secrets, and ideally an external secret store (Azure Key Vault via CSI driver, HashiCorp Vault).
- Common `type` values: `Opaque` (generic), `kubernetes.io/tls`, `kubernetes.io/dockerconfigjson` (image pull secrets).

---

## 13. PersistentVolume (PV) & PersistentVolumeClaim (PVC)

Decouples storage provisioning from storage consumption.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```

Mounted into a Pod:
```yaml
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: pvc-example
```

**Interview tips:**
- PV = actual storage resource (cluster-scoped, admin/cloud provisions it). PVC = a *request* for storage by a user/app (namespace-scoped).
- Access modes: `ReadWriteOnce` (1 node r/w), `ReadOnlyMany` (many nodes read-only), `ReadWriteMany` (many nodes r/w — needs NFS/Azure Files/etc., not block storage like Azure Disk).
- Reclaim policy: `Retain` (keep data, manual cleanup), `Delete` (auto-delete underlying storage), `Recycle` (deprecated).

---

## 14. StorageClass

Enables **dynamic** provisioning of PVs on demand (no admin pre-creating PVs manually).

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium
provisioner: disk.csi.azure.com
parameters:
  skuName: Premium_LRS
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

**Interview tip:** `volumeBindingMode: WaitForFirstConsumer` delays binding/provisioning until a Pod using the PVC is actually scheduled — important in multi-zone clusters so the volume gets created in the same zone as the Pod.

---

## 15. Namespace

Logical partition of a cluster for multi-tenancy, resource isolation, and RBAC scoping.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-team
  labels:
    environment: development
```

**Interview tip:** Not everything is namespaced — Nodes, PVs, StorageClasses, ClusterRoles are **cluster-scoped**. Pods, Deployments, Services, ConfigMaps, Secrets, PVCs are **namespace-scoped**. (`kubectl api-resources --namespaced=true` to check.)

---

## 16. ServiceAccount

Identity for **processes running inside Pods** to authenticate to the Kubernetes API (distinct from User accounts, which are for humans).

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: dev-team
```

Used in a Pod:
```yaml
      serviceAccountName: app-sa
```

**Interview tip:** Every Pod runs as a ServiceAccount even if you don't specify one (`default` SA in the namespace). Best practice: create dedicated SAs per app with least-privilege RBAC rather than using `default`.

---

## 17. RBAC — Role, RoleBinding, ClusterRole, ClusterRoleBinding

### Role (namespace-scoped permissions)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev-team
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

### RoleBinding (attaches Role to a subject, in that namespace)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev-team
subjects:
  - kind: ServiceAccount
    name: app-sa
    namespace: dev-team
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRole / ClusterRoleBinding (cluster-wide, or reusable across namespaces)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-reader-binding
subjects:
  - kind: User
    name: ramakrishna@example.com
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

**Interview tip — this table comes up constantly:**

| | Scope of permission | Bound within |
|---|---|---|
| Role + RoleBinding | namespace | that namespace only |
| ClusterRole + ClusterRoleBinding | cluster-wide | entire cluster |
| ClusterRole + RoleBinding | cluster-wide *rule set* | but binding restricts it to one namespace (common pattern for reusable roles) |

---

## 18. NetworkPolicy

Firewall rules for Pod-to-Pod traffic (requires a CNI that supports it — Calico, Cilium; the default kubenet does not enforce these).

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: dev-team
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

**Interview tip:** By default, all Pods can talk to all Pods (no isolation). Once **any** NetworkPolicy selects a Pod, that Pod becomes "default deny" for the direction(s) specified (Ingress/Egress) — only explicitly allowed traffic gets through. This trips people up in interviews.

---

## 19. HorizontalPodAutoscaler (HPA)

Automatically scales replica count based on observed metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

**Interview tip:** Requires the **metrics-server** to be running in-cluster. "HPA vs VPA vs Cluster Autoscaler?"
> HPA = scales **replica count** (out/in). VPA = scales **Pod resource requests/limits** (up/down). Cluster Autoscaler = scales the **number of Nodes** in the cluster. All three can be combined but HPA + VPA together on CPU/memory needs care (they can fight each other).

---

## 20. ResourceQuota & LimitRange

Governance at the namespace level.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-team-quota
  namespace: dev-team
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
---
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: dev-team
spec:
  limits:
    - default:
        cpu: 500m
        memory: 256Mi
      defaultRequest:
        cpu: 250m
        memory: 128Mi
      type: Container
```

**Interview tip:** `ResourceQuota` caps the **total** consumption of a namespace. `LimitRange` sets **defaults and min/max per Pod/Container** if not explicitly specified — without a LimitRange, a Pod with no `resources:` block has no limit at all.

---

## 21. Quick Reference — apiVersion Cheat Sheet

| Kind | apiVersion |
|---|---|
| Pod, Service, ConfigMap, Secret, Namespace, PV, PVC, ServiceAccount | `v1` |
| Deployment, ReplicaSet, StatefulSet, DaemonSet | `apps/v1` |
| Job, CronJob | `batch/v1` |
| Ingress, NetworkPolicy | `networking.k8s.io/v1` |
| Role, RoleBinding, ClusterRole, ClusterRoleBinding | `rbac.authorization.k8s.io/v1` |
| HorizontalPodAutoscaler | `autoscaling/v2` |
| StorageClass | `storage.k8s.io/v1` |

**Interview tip:** "How do you find the correct apiVersion for a resource?" → `kubectl api-resources` lists Kind, shortname, apiVersion, and namespaced status in one shot. `kubectl explain <kind>` gives the full field reference for any object.

---

## 22. Rapid-Fire Interview Q&A

**Q: What decides which Pods a Deployment/Service manages?**
> Label selectors (`spec.selector.matchLabels` for Deployment, `spec.selector` for Service) matched against Pod `metadata.labels`.

**Q: What happens if you delete a Pod managed by a Deployment?**
> The ReplicaSet controller notices actual (2) < desired (3) and creates a replacement immediately — this is the reconciliation loop in action.

**Q: Difference between `kubectl apply` and `kubectl create`?**
> `create` fails if the object already exists (imperative). `apply` does a 3-way diff (last-applied-config, current live state, new file) and patches only the changed fields — safe to re-run (declarative, idempotent).

**Q: What's an `initContainer`?**
> A container that runs to completion *before* the main containers start — used for setup tasks (wait-for-dependency, DB migrations, config generation).

**Q: Sidecar container pattern?**
> A helper container running alongside the main app container in the same Pod (shares network/volumes) — e.g. a log-shipper or service-mesh proxy (Envoy in Istio).

**Q: `kubectl` commands you should know cold for the exam/interview:**
```bash
kubectl get pods -o wide -n dev-team
kubectl describe deployment nginx-deployment
kubectl logs -f pod-name -c container-name
kubectl exec -it pod-name -- /bin/sh
kubectl rollout status deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml
kubectl explain deployment.spec.strategy
```

---

## 23. Suggested Study Order (for a Lead DevOps interview)

1. Pod → ReplicaSet → Deployment (get the controller/reconciliation story solid)
2. Service types + Ingress (networking is always asked)
3. ConfigMap/Secret (config management questions are common)
4. StatefulSet + PV/PVC/StorageClass (storage is where people get tripped up)
5. RBAC (Role vs ClusterRole table — memorize it)
6. NetworkPolicy + HPA (advanced/senior-level differentiators)
7. Practice explaining **the reconciliation loop** in your own words — nearly every object in this guide is graded on that same underlying pattern: desired state (spec) vs actual state (status), continuously reconciled by a controller.
