# Kube Scheduler

## What It Does

The scheduler **decides which node a pod runs on** — it does not actually run the pod. That's the kubelet's job.

It evaluates every unscheduled pod and finds the best node through a two-phase process.

---

## Scheduling Process

### Phase 1 — Filtering
Eliminates nodes that **can't** run the pod (e.g. not enough CPU/memory, wrong node selector, taints not tolerated).

![Scheduler filtering nodes by CPU](https://kodekloud.com/kk-media/image/upload/v1752869723/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Kube-Scheduler/frame_110.jpg)

### Phase 2 — Ranking
Scores remaining nodes **0–10** using a priority function. The node with the highest score wins. Example: a node that would have more free CPU after placing the pod scores higher.

![Scheduler ranking nodes](https://kodekloud.com/kk-media/image/upload/v1752869724/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Kube-Scheduler/frame_160.jpg)

---

## Factors That Influence Scheduling
- Resource requests & limits
- Taints & tolerations
- Node selectors & node affinity
- Pod affinity / anti-affinity

*(These are covered in depth in later sections)*

---

## Inspecting the Scheduler

### kubeadm clusters
Runs as a static pod in `kube-system`:
```bash
cat /etc/kubernetes/manifests/kube-scheduler.yaml

# Or verify the running process
ps -aux | grep kube-scheduler
# kube-scheduler --kubeconfig=/etc/kubernetes/scheduler.conf --leader-elect=true
```

### Manual (non-kubeadm) clusters
```bash
cat /etc/systemd/system/kube-scheduler.service
# ExecStart=/usr/local/bin/kube-scheduler --config=/etc/kubernetes/config/kube-scheduler.yaml
```

---

> [!tip] CKA Exam
> - Scheduler only **assigns** pods to nodes — kubelet does the actual work
> - If a pod is stuck in `Pending`, the scheduler couldn't find a suitable node — check resource requests, taints, and node selectors
> - Scheduler manifest: `/etc/kubernetes/manifests/kube-scheduler.yaml`
> - `--leader-elect=true` is required in HA setups
> - You can write and deploy a **custom scheduler** if needed (advanced topic)
