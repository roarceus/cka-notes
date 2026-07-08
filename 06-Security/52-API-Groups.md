# API Groups

## Why This Matters

Understanding API groups is essential for RBAC — when creating Roles, you specify which `apiGroups` a rule applies to.

---

## Two Categories

### Core API Group (`/api`)
The original Kubernetes APIs — no group name needed in YAML (`apiVersion: v1`).

Resources: pods, nodes, namespaces, services, configmaps, secrets, persistentvolumes, persistentvolumeclaims, endpoints, events, replicationcontrollers

### Named API Groups (`/apis`)
Newer, organised by feature area. Referenced by their group name in `apiVersion`.

| Group | apiVersion | Resources |
|---|---|---|
| `apps` | `apps/v1` | Deployments, ReplicaSets, StatefulSets, DaemonSets |
| `batch` | `batch/v1` | Jobs, CronJobs |
| `networking.k8s.io` | `networking.k8s.io/v1` | NetworkPolicies, Ingresses |
| `storage.k8s.io` | `storage.k8s.io/v1` | StorageClasses |
| `rbac.authorization.k8s.io` | `rbac.authorization.k8s.io/v1` | Roles, RoleBindings, ClusterRoles |
| `certificates.k8s.io` | `certificates.k8s.io/v1` | CertificateSigningRequests |
| `autoscaling` | `autoscaling/v2` | HorizontalPodAutoscalers |

---

## API Verbs (Actions)

Each resource supports a set of verbs used in RBAC rules:

`get`, `list`, `watch`, `create`, `update`, `patch`, `delete`, `deletecollection`

---

## Querying the API Server

```bash
# List all available API groups and resources
kubectl api-resources

# List resources in a specific group
kubectl api-resources --api-group=apps

# Use kubectl proxy to browse the API without cert flags
kubectl proxy          # starts on 127.0.0.1:8001
curl http://localhost:8001/apis/apps/v1
```

> **`kube-proxy` ≠ `kubectl proxy`**
> - `kube-proxy` = cluster networking (pod-to-service routing)
> - `kubectl proxy` = local HTTP proxy to reach the API server using your kubeconfig credentials

---

> [!tip] CKA Exam
> - In RBAC rules: core group resources use `apiGroups: [""]` (empty string); named groups use their full name e.g. `apiGroups: ["apps"]`
> - `kubectl api-resources` is the fastest way to find a resource's API group and shortname
> - `kubectl api-resources --namespaced=true/false` to see which resources are namespace-scoped vs cluster-scoped
