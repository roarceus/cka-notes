# Multiple Schedulers

## Concept

Kubernetes allows you to run multiple schedulers simultaneously. Each pod can specify which scheduler should handle its placement. Useful when you need custom scheduling logic for specific workloads.

- Default scheduler is named `default-scheduler`
- Every custom scheduler needs a **unique name**
- Multiple schedulers can coexist — only one handles each pod

---

## Scheduler Configuration File

Each scheduler is defined by a `KubeSchedulerConfiguration` file:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

---

## Deploying a Custom Scheduler as a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - name: kube-scheduler
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    command:
    - kube-scheduler
    - --config=/etc/kubernetes/my-scheduler-config.yaml
```

> In HA environments, set `leaderElect: false` if running only one instance of the custom scheduler, to avoid conflicting with the default scheduler's leader election.

---

## Assigning a Pod to a Custom Scheduler

Add `schedulerName` to the pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  schedulerName: my-custom-scheduler   # must match the scheduler's configured name
```

If the scheduler name is wrong or the scheduler isn't running → pod stays `Pending`.

---

## Verifying Which Scheduler Was Used

```bash
# Check events to see which scheduler assigned the pod
kubectl get events -o wide
# Look for SOURCE column — shows the scheduler name

# Check custom scheduler logs
kubectl logs my-custom-scheduler -n kube-system
```

---

> [!tip] CKA Exam
> - Use `schedulerName` in the pod spec to target a specific scheduler
> - If a pod is stuck `Pending` after adding `schedulerName`, the named scheduler likely isn't running or is misconfigured
> - Verify scheduling via `kubectl get events -o wide` — the `SOURCE` column shows which scheduler acted
> - The full deployment setup (RBAC, ConfigMaps, ServiceAccounts) is complex — the exam is more likely to test assigning pods to an existing custom scheduler than building one from scratch
