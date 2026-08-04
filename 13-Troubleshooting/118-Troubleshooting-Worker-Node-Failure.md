# Troubleshooting: Worker Node Failure

## 1. Check Node Status

```bash
kubectl get nodes
NAME       STATUS     ROLES     AGE   VERSION
worker-1   Ready      <none>    8d    v1.13.0
worker-2   NotReady   <none>    8d    v1.13.0
```

If `NotReady`, dig deeper:
```bash
kubectl describe node worker-1
```
- Check the **Conditions** section: `OutOfDisk`, `MemoryPressure`, `DiskPressure`, `PIDPressure`, `Ready` — each is `true`/`false` and points directly at the resource under pressure
- Check **`LastHeartbeatTime`** — shows when the node last communicated with the control plane, useful for spotting when it actually went down

---

## 2. Check the Node and Kubelet Itself

- Check the node's own CPU/memory/disk usage
- Check kubelet service status:
```bash
service kubelet status
```
- Should show `Active: active (running)` with a valid PID

- Check kubelet logs for errors:
```bash
sudo journalctl -u kubelet
```

---

## 3. Verify Kubelet Certificates

```bash
openssl x509 -in /var/lib/kubelet/worker-1.crt -text
```

Check:
- **`Issuer`** — should be the correct CA (e.g. `CN = KUBERNETES-CA`)
- **`Validity`** (`Not Before` / `Not After`) — certificate must not be expired
- **`Subject`** — should identify the correct node (e.g. `CN = system:node:worker-1`)

> [!tip] CKA Exam
> Expired or wrongly-issued kubelet certificates are a common cause of a node silently failing to communicate with the control plane — the node may show `NotReady` with no obvious resource pressure condition, which should point you toward checking certs.

---

> [!tip] CKA Exam
> - Troubleshooting order: `kubectl get nodes` → `kubectl describe node` (conditions + heartbeat) → kubelet service/logs → kubelet certificate validity
> - The four resource pressure conditions to know: `OutOfDisk`, `MemoryPressure`, `DiskPressure`, `PIDPressure`
> - `journalctl -u kubelet` is the standard way to inspect kubelet logs on the node itself
