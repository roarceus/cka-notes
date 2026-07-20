# Prerequisite — CoreDNS

CoreDNS is the DNS server used by Kubernetes for service discovery. This note covers the basics of how it works.

---

## What CoreDNS Does

CoreDNS is a flexible DNS server that can resolve hostnames from various sources (files, Kubernetes API, etc.) using a **plugin-based** architecture.

- Listens on port **53** (standard DNS port)
- Configured via a file called **`Corefile`**
- In Kubernetes: runs as a Deployment in `kube-system`, resolves service/pod DNS names

---

## Basic Setup (standalone)

```bash
# Download and extract
curl -LO https://github.com/coredns/coredns/releases/download/v1.12.4/coredns_1.12.4_linux_amd64.tgz
tar -zxf coredns_1.12.4_linux_amd64.tgz

# Run (listens on port 53 by default)
./coredns
```

### Corefile — basic config using /etc/hosts

```
.:53 {
    hosts /etc/hosts {      # resolve from /etc/hosts
        reload 1m
        fallthrough
    }
    forward . /etc/resolv.conf {   # forward unmatched queries upstream
        max_concurrent 1000
    }
    cache 30
    log
    errors
}
```

---

## In Kubernetes

CoreDNS uses the **`kubernetes` plugin** (not `hosts`) to resolve:
- Services: `<service>.<namespace>.svc.cluster.local`
- Pods: `<pod-ip>.<namespace>.pod.cluster.local`

CoreDNS config in Kubernetes is stored in a **ConfigMap**:

```bash
kubectl get configmap coredns -n kube-system -o yaml
```

---

> [!tip] CKA Exam
> - CoreDNS runs as a **Deployment** in `kube-system` — `kubectl get pods -n kube-system | grep coredns`
> - Its config is in a **ConfigMap** named `coredns` in `kube-system`
> - DNS issues in pods → check CoreDNS pods are running and the ConfigMap is correct
> - Service DNS format: `<service>.<namespace>.svc.cluster.local`
