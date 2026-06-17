# Kube Controller Manager

## What It Does

The Controller Manager is a single process that bundles **all controllers** in Kubernetes. Each controller is a control loop that watches the cluster state and works to drive it toward the desired state.

Think of each controller as a department with one specific job.

---

## Key Controllers

### Node Controller
Monitors node health via the API server every **5 seconds**.

| Timing | Value |
|---|---|
| Heartbeat check interval | 5 seconds |
| Grace period before marking unreachable | 40 seconds |
| Time allowed for recovery before rescheduling pods | 5 minutes |

### Replication Controller
Ensures the desired number of pods is always running. If a pod dies, it creates a new one.

### Other Built-in Controllers
deployments, daemonsets, cronjobs, jobs, namespaces, endpoints, serviceaccounts, persistentvolumes, statefulsets, and more — all bundled into one process.

---

## Inspecting the Controller Manager

### kubeadm clusters
Runs as a static pod in `kube-system`:
```bash
kubectl get pods -n kube-system
# kube-controller-manager-master   1/1   Running   0

# View config/flags
cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```

### Manual (non-kubeadm) clusters
```bash
cat /etc/systemd/system/kube-controller-manager.service
# or
ps -aux | grep kube-controller-manager
```

---

## Key Configuration Flags

| Flag | Purpose |
|---|---|
| `--controllers` | Which controllers to enable (`*` = all, `-foo` disables `foo`) |
| `--leader-elect` | Enable leader election for HA (always `true` in production) |
| `--cluster-cidr` | Pod IP range for the cluster |
| `--node-monitor-period` | How often to check node status (default 5s) |
| `--node-monitor-grace-period` | Time before node marked unreachable (default 40s) |
| `--pod-eviction-timeout` | Time before pods are rescheduled off failed node (default 5m) |

### Selectively enabling/disabling controllers:
```bash
# Disable only tokencleaner, enable everything else
--controllers=*,-tokencleaner

# Enable only specific controllers
--controllers=deployment,replicaset,namespace
```

---

> [!tip] CKA Exam
> - Controller Manager manifest is at `/etc/kubernetes/manifests/kube-controller-manager.yaml` in kubeadm clusters
> - `--leader-elect=true` is required in HA multi-master setups — only one instance is active at a time
> - If pods aren't being rescheduled after a node failure, the Node Controller timing flags are worth checking
> - All controllers are **enabled by default** — `--controllers=*` is the default value
