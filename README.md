# CKA Study Notes

Personal notes for the **Certified Kubernetes Administrator (CKA)** exam, distilled from the [KodeKloud CKA course](https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Introduction/Course-Introduction/page).

Notes are concise and exam-focused — each file covers the key concepts, commands, and YAML for one topic, with `> [!tip] CKA Exam` callouts for high-priority exam content. Formatted for [Obsidian](https://obsidian.md).

---

## Progress

### 00 — Exam Tips
- [x] Imperative Commands & Dry Run

### 01 — Core Concepts
- [x] Cluster Architecture
- [x] Docker vs ContainerD
- [x] Docker Deprecation
- [x] ETCD for Beginners
- [x] ETCD in Kubernetes
- [x] Kube API Server
- [x] Kube Controller Manager
- [x] Kube Scheduler
- [x] Kubelet
- [x] Kube Proxy
- [x] Pods
- [x] ReplicaSets
- [x] Deployments
- [x] Services
- [x] Namespaces
- [x] Imperative vs Declarative

### 02 — Scheduling
- [x] Manual Scheduling
- [x] Labels and Selectors
- [x] Taints and Tolerations
- [x] Node Selectors
- [x] Node Affinity
- [x] Resource Requirements and Limits
- [x] DaemonSets
- [x] Static Pods
- [x] Priority Classes
- [x] Multiple Schedulers
- [x] Scheduler Profiles
- [x] Admission Controllers

### 03 — Logging and Monitoring
- [x] Monitor Cluster Components
- [x] Managing Application Logs

### 04 — Application Lifecycle Management
- [x] Rolling Updates and Rollbacks
- [x] Commands and Arguments in Docker and Kubernetes
- [x] Environment Variables
- [x] ConfigMaps
- [x] Secrets
- [x] Multi Container Pods
- [x] Autoscaling Introduction
- [x] Horizontal Pod Autoscaler
- [x] In Place Resize of Pods
- [x] Vertical Pod Autoscaler

### 05 — Cluster Maintenance
- [x] OS Upgrades
- [x] Cluster Upgrade
- [ ] Backup and Restore

### 06 — Security
- [ ] Kubernetes Security Primitives
- [ ] Authentication
- [ ] TLS Certificates
- [ ] Certificate API
- [ ] KubeConfig
- [ ] RBAC
- [ ] Cluster Roles
- [ ] Service Accounts
- [ ] Image Security
- [ ] Network Policies

### 07 — Storage
- [ ] Volumes
- [ ] Persistent Volumes
- [ ] Persistent Volume Claims
- [ ] Storage Classes

### 08 — Networking
- [ ] Networking Basics
- [ ] CNI
- [ ] Pod Networking
- [ ] Service Networking
- [ ] DNS
- [ ] Ingress

### 09 — Troubleshooting
- [ ] Application Failure
- [ ] Control Plane Failure
- [ ] Worker Node Failure
- [ ] Network Troubleshooting

---

## Quick Reference

| Need to... | Command |
|---|---|
| Generate YAML without creating | `kubectl <cmd> --dry-run=client -o yaml` |
| Filter resources by label | `kubectl get pods -l key=value` |
| Check all namespaces | `kubectl get pods -A` |
| Describe a resource (debug) | `kubectl describe <resource> <name>` |
| Check logs | `kubectl logs <pod> [-c container]` |
| Execute into a pod | `kubectl exec -it <pod> -- bash` |
| Check node taints | `kubectl describe node <name> \| grep Taint` |
| Switch namespace | `kubectl config set-context --current --namespace=<ns>` |

---

## Resources

- [KodeKloud CKA Notes](https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Introduction/Course-Introduction/page)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [kubectl Conventions](https://kubernetes.io/docs/reference/kubectl/conventions/)