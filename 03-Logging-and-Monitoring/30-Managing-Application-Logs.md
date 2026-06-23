# Managing Application Logs

## kubectl logs — Core Commands

```bash
# Stream live logs from a pod (single container)
kubectl logs -f <pod-name>

# Fetch logs without following
kubectl logs <pod-name>

# Specify a container (required if pod has multiple containers)
kubectl logs -f <pod-name> <container-name>
# or
kubectl logs -f <pod-name> -c <container-name>

# Logs from a previous (crashed) container instance
kubectl logs <pod-name> -c <container-name> --previous

# Last N lines only
kubectl logs <pod-name> --tail=50

# Logs since a time window
kubectl logs <pod-name> --since=1h
```

---

## Multi-Container Pods

If a pod has more than one container, you **must** specify the container name — omitting it returns an error:

```bash
# This errors on a multi-container pod:
kubectl logs -f event-simulator-pod

# This works:
kubectl logs -f event-simulator-pod -c event-simulator
```

---

> [!tip] CKA Exam
> - `-f` = follow/stream logs in real time (like `tail -f`)
> - Always use `-c <container>` for multi-container pods
> - `--previous` is invaluable for debugging crash-looping containers — shows logs from the last failed instance
> - For control plane component logs: `kubectl logs -n kube-system <pod-name>` (e.g. `kube-apiserver-controlplane`)
