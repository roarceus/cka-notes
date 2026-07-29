# Deployment with kubeadm - Introduction

## What kubeadm Does

- kubeadm bootstraps a multi-node Kubernetes cluster following Kubernetes best practices
- Without it, you'd have to manually install each control plane component (kube-apiserver, etcd, controller-manager, scheduler, etc.), wire up their configs to point to each other, and set up all the required certificates — a tedious, error-prone process
- kubeadm automates all of that: installing and configuring the right components on the right nodes, in the right order

---

## High-Level Setup Steps

1. **Provision systems/VMs** for the cluster
2. **Designate roles** — one node as master, the rest as workers
3. **Install a container runtime** on every node (e.g. containerd)
4. **Install kubeadm** on every node
5. **Initialize the master** — kubeadm installs and configures all required control plane components on it
6. **Set up pod networking** before joining workers — normal network connectivity between nodes is *not* sufficient; Kubernetes requires a dedicated **pod network** solution (CNI) between master and workers
7. **Join worker nodes** to the master

Once workers are joined, the cluster is ready to run applications.

---

> [!tip] CKA Exam
> - Order matters: container runtime → kubeadm → init master → **pod network** → join workers
> - The pod network must be set up **before** joining worker nodes, or nodes/pods won't have proper connectivity
> - kubeadm handles component installation/config/certificates automatically — know this is its core value versus manual setup
