# Cluster Roles

## Why ClusterRoles Exist

**Role** + **RoleBinding** are namespace-scoped — they can't control access to cluster-wide resources like nodes or persistent volumes (which don't belong to any namespace).

**ClusterRole** + **ClusterRoleBinding** are used for:
- Cluster-scoped resources (nodes, PVs, namespaces, clusterroles, storageclasses...)
- Granting access to namespaced resources **across all namespaces** at once

```bash
# See which resources are cluster-scoped
kubectl api-resources --namespaced=false
```

---

## Role vs ClusterRole — Scope Summary

| | Role + RoleBinding | ClusterRole + ClusterRoleBinding |
|---|---|---|
| Scope | Single namespace | Entire cluster |
| Can manage | Namespaced resources in one namespace | Cluster-scoped resources + namespaced resources in ALL namespaces |

> A ClusterRole bound to pods via ClusterRoleBinding gives access to pods in **every namespace**.

---

## ClusterRole YAML

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
```

## ClusterRoleBinding YAML

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

---

## Imperative Commands

```bash
# Create a ClusterRole
kubectl create clusterrole cluster-administrator \
  --verb=list,get,create,delete \
  --resource=nodes

# Create a ClusterRoleBinding
kubectl create clusterrolebinding cluster-admin-role-binding \
  --clusterrole=cluster-administrator \
  --user=cluster-admin
```

---

## Viewing ClusterRoles

```bash
kubectl get clusterroles
kubectl get clusterrolebindings

kubectl describe clusterrole cluster-administrator
kubectl describe clusterrolebinding cluster-admin-role-binding
```

---

> [!tip] CKA Exam
> - Use ClusterRole for anything returned by `kubectl api-resources --namespaced=false`
> - Default cluster roles exist: `cluster-admin`, `admin`, `edit`, `view` — check with `kubectl get clusterroles`
> - A RoleBinding can reference a **ClusterRole** — this gives cluster-role permissions scoped to one namespace (useful pattern)
> - `kubectl auth can-i <verb> <resource> --as <user>` works for cluster-scoped resources too
> - ClusterRoles have no `namespace` field in metadata — they're cluster-wide by definition
