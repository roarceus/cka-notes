# Helm 2 vs Helm 3

## Helm 2: The Tiller Component

- Early Kubernetes lacked RBAC and CRDs, so Helm 2 needed an extra in-cluster component, **Tiller**, sitting between the Helm CLI and the cluster to manage chart installs/upgrades
- Problems with this design:
  - **Complexity**: an extra layer between CLI and cluster
  - **Security risk**: Tiller ran with broad, often near-cluster-admin privileges ("God mode") — any misconfiguration was a serious security exposure

![Helm 2 architecture: Helm CLI → Tiller → Kubernetes](https://kodekloud.com/kk-media/image/upload/v1752869777/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-A-quick-note-about-Helm2-vs-Helm3/helm-2-architecture-diagram-kubernetes.jpg)

---

## Helm 3: Tiller Removed

- Helm 3 removes Tiller entirely — the Helm client talks **directly** to the Kubernetes API, using Kubernetes' own **RBAC**
- Every Helm action is now governed by the same RBAC permissions your user/service account already has via `kubectl` — simpler architecture, better security

![Helm 3 architecture: Helm CLI → Kubernetes directly, using RBAC/CRDs](https://kodekloud.com/kk-media/image/upload/v1752869778/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-A-quick-note-about-Helm2-vs-Helm3/helm-3-architecture-diagram.jpg)

---

## Key Differences

| Feature | Helm 2 | Helm 3 |
|---|---|---|
| Intermediary component | Requires Tiller | None — direct communication |
| Security model | Elevated risk (broad Tiller privileges) | Uses Kubernetes RBAC directly |
| Rollback mechanism | Basic revision comparison | Three-way strategic merge patch |

---

## Rollback: Three-Way Strategic Merge (Helm 3)

Example flow:
```bash
helm install wordpress
# → revision 1 (e.g. wordpress:4.8-apache)

helm upgrade wordpress
# → revision 2 (e.g. wordpress:5.8-apache)

helm rollback wordpress
# → reverts to revision 1's state (recorded as a new revision, e.g. revision 3)
```

- Every install/upgrade/rollback creates a **new revision** in Helm's history
- Helm 3's rollback logic compares **three** things:
  1. The previous chart revision
  2. The desired chart state
  3. The **live state** of the actual Kubernetes objects
- This three-way comparison correctly reconciles discrepancies — e.g. if someone manually changed a live object with `kubectl` outside of Helm

**Helm 2 vs Helm 3 handling of manual (out-of-band) changes:**
- Helm 2: manual `kubectl` edits (e.g. `kubectl set image`) weren't tracked in Helm's revision history, so rollbacks could miss/ignore those changes
- Helm 3: compares live state against both current and desired revisions, so manual changes are more likely to be preserved and not silently overwritten

---

> [!tip] CKA Exam
> - **Helm 3 removed Tiller** — direct CLI-to-Kubernetes communication using native RBAC is the single most important Helm 2→3 change to remember
> - Helm 3 rollback uses a **three-way strategic merge patch** (previous revision + desired state + live state), not just a simple revision diff
> - Every `install`/`upgrade`/`rollback` action creates a new **revision**
