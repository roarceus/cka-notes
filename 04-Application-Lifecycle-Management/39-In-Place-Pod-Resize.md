# In-Place Pod Resize

## What It Is

Normally, changing resource requests/limits on a running pod causes Kubernetes to **terminate and recreate** the pod. In-place resizing allows resource updates **without pod recreation** — reducing downtime for stateful workloads.

> ⚠️ **Alpha since v1.27** — not enabled by default. Enable with feature gate:
> ```bash
> FEATURE_GATES=InPlacePodVerticalScaling=true
> ```

---

## How It Works

Once enabled, updating CPU/memory in a deployment's pod spec applies the change to the **running pod directly** instead of triggering a rollout.

### Optional: Resize Policy

Control whether a resource change requires a container restart:

```yaml
spec:
  containers:
  - name: my-app
    image: nginx
    resizePolicy:
    - resourceName: cpu
      restartPolicy: NotRequired    # CPU can be updated without restart
    - resourceName: memory
      restartPolicy: RestartContainer  # memory changes require restart
    resources:
      requests:
        cpu: "250m"
        memory: "256Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"
```

---

## Limitations

- Only **CPU and memory** can be resized in place
- **Init containers and ephemeral containers** are not supported
- Memory limit cannot be reduced **below current usage** (resize stays pending until achievable)
- Cannot shift resource assignments between containers
- **Windows pods** not supported
- Pod QoS class changes not supported

---

> [!tip] CKA Exam
> - This is an **alpha feature** — likely tested as awareness/concept knowledge rather than hands-on configuration
> - Default behaviour (without the feature gate): resource changes = pod recreation
> - VPA (next note) automates vertical scaling and builds on this mechanism
