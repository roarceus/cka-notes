# Troubleshooting: Control Plane Failure

## 1. Check Node Health First

```bash
kubectl get nodes
```
- Quick first pass to rule out node-level issues before diving into control plane specifics

---

## 2. Confirm Control Plane Component Status

Control plane components are deployed either as **pods** (typical kubeadm setups, in `kube-system`) or as **native systemd services** — check accordingly.

### Native services (systemd)
```bash
service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status
```
- Each should show `Active: active (running)` with a valid PID

### Pods (kubeadm setups)
- Check pod health in `kube-system` for `kube-apiserver-*`, `kube-controller-manager-*`, `kube-scheduler-*`, `etcd-*` (see the Static Pods note — these live at `/etc/kubernetes/manifests/`)
- Also check `kube-proxy` on worker nodes

---

## 3. Review Logs

### Pods (kubeadm)
```bash
kubectl logs kube-apiserver-master -n kube-system
```
- Shows startup info (version, loaded admission controllers, skipped API groups, etc.) — useful for spotting config/plugin issues

### Native services
```bash
sudo journalctl -u kube-apiserver
```
- `journalctl -u <service-name>` works the same way for `kube-controller-manager`, `kube-scheduler`, etc.

---

> [!tip] CKA Exam
> - Know **both** paths for control plane troubleshooting: pod-based (`kubectl logs ... -n kube-system`) vs native-service-based (`journalctl -u <service>` / `service <name> status`) — the exam environment may use either
> - Standard order: node health → component status (pod or service) → logs
> - On kubeadm clusters, static pod manifests for control plane components live at `/etc/kubernetes/manifests/` — check there if a component pod is missing or crash-looping
