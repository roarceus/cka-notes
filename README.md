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
- [x] Backup and Restore

### 06 — Security
- [x] Kubernetes Security Primitives
- [x] Authentication
- [x] TLS Certificates
- [x] Certificate API
- [x] KubeConfig
- [x] API Groups
- [x] Authorization
- [x] RBAC
- [x] Cluster Roles
- [x] Service Accounts
- [x] Image Security
- [x] Security Contexts
- [x] Network Policies
- [x] Developing Network Policies
- [x] Custom Resource Definition (CRD)
- [x] Custom Controllers
- [x] Operator Framework

### 07 — Storage
- [x] Storage in Docker
- [x] Container Storage Interface
- [x] Volumes
- [x] Persistent Volumes
- [x] Persistent Volume Claims
- [x] Storage Classes

### 08 — Networking
- [x] Prerequisite Switching Routing Gateways CNI in Kubernetes
- [x] Prerequisite DNS
- [x] Prerequisite CoreDNS
- [x] Prerequisite Network Namespaces
- [x] Prerequisite Docker Networking
- [x] Prerequisite CNI
- [x] Cluster Networking
- [x] Pod Networking
- [x] CNI in Kubernetes
- [x] Service Networking
- [x] DNS in Kubernetes
- [x] CoreDNS in Kubernetes
- [x] Ingress
- [x] Introduction to Gateway API
- [x] Gateway API Practical Guide

### 09 — Design and Install a Kubernetes Cluster
- [x] Design a Kubernetes Cluster
- [x] Choosing Kubernetes Infrastructure
- [x] Configure High Availability
- [x] ETCD in HA

### 10 — Install "Kubernetes the kubeadm way"
- [x] Deployment with kubeadm - Introduction
- [x] Deployment with kubeadm - Provision VMs with Vagrant
- [x] Deployment with kubeadm

### 11 — Helm Basics
- [x] Helm - Introduction
- [x] Installation and Configuration
- [x] A Quick Note on Helm2 vs Helm3
- [x] Helm Components
- [x] Helm Charts
- [x] Working with Helm - Basics
- [x] Customizing Chart Parameters
- [x] Lifecycle Management with Helm

### 12 — Kustomize Basics
- [x] Kustomize Problem Statement and Ideology
- [x] Kustomize vs Helm
- [x] Installation/Setup
- [x] The kustomization.yaml File
- [x] Kustomize Output
- [x] Managing Directories
- [x] Managing Directories Demo
- [x] Common Transformers
- [x] Image Transformers
- [x] Transformers Demo
- [x] Patches Intro
- [x] Differect Types of Patches
- [x] Patches Dictionary
- [x] Patches List
- [x] Overlays
- [ ] Components

### 13 — Troubleshooting
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