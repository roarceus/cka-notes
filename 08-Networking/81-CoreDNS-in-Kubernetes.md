# CoreDNS in Kubernetes

## Why a Centralized DNS Server

- Manually adding hosts-file entries for every pod doesn't scale when pods are created/destroyed constantly
- Kubernetes solves this with a **centralized DNS server** — every pod is pre-configured (via `/etc/resolv.conf`) to query it
- Kubernetes doesn't create per-pod DNS records by default; it creates records for **services**, and (if enabled) converts pod IPs into dashed hostnames
- **CoreDNS** replaced **Kube-DNS** as the default cluster DNS server starting in Kubernetes v1.12

---

## CoreDNS Deployment

- Runs as a pod in the **`kube-system`** namespace
- Deployed with **2 replicas** (via a Deployment) for high availability
- Configuration lives in a file called **Corefile**, mounted from a ConfigMap:

```bash
kubectl get configmap -n kube-system
NAME      DATA   AGE
coredns   1      168d
```

Example Corefile:
```
.:53 {
    errors
    health
    kubernetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure
        upstream
        fallthrough in-addr.arpa ip6.arpa
    }
    prometheus :9153
    proxy . /etc/resolv.conf
    cache 30
    reload
}
```

| Directive | Function |
|---|---|
| `errors` | Logs errors |
| `health` | Health check endpoint |
| `kubernetes cluster.local ...` | Enables the Kubernetes plugin — root domain `cluster.local`, converts pod IPs to dashed hostnames |
| `prometheus :9153` | Exposes metrics |
| `proxy . /etc/resolv.conf` | Forwards unresolved (external) queries upstream |
| `cache 30` | Caches responses for 30s |
| `reload` | Auto-reloads config on change |

- To change CoreDNS behavior, edit the **ConfigMap** (not the pod directly)
- CoreDNS continuously watches the API server so DNS records stay in sync with new pods/services

---

## Connecting Pods to CoreDNS

- Kubernetes creates a service named **`kube-dns`** (in `kube-system`) that fronts the CoreDNS pods:

```bash
kubectl get service -n kube-system
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP   1d
```

- The **kubelet** automatically configures every pod's `/etc/resolv.conf` to use this service IP as the nameserver, plus a `search` list of domain suffixes:

```bash
cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
```

- This behavior comes from the kubelet config file:

```yaml
# /var/lib/kubelet/config.yaml
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
```

---

## Resolving Services and Pods

- Thanks to the `search` suffixes, a service can be reached with any of these forms:

```bash
curl http://web-service
curl http://web-service.default
curl http://web-service.default.svc
curl http://web-service.default.svc.cluster.local
```

```bash
host web-service
web-service.default.svc.cluster.local has address 10.107.37.188
```

- **Pod DNS records require the full FQDN** — the `search` suffixes don't apply to short pod hostnames:

```bash
host 10-244-2-5
Host 10-244-2-5 not found: 3(NXDOMAIN)

host 10-244-2-5.default.pod.cluster.local
10-244-2-5.default.pod.cluster.local has address 10.244.2.5
```

---

> [!tip] CKA Exam
> - CoreDNS runs in **`kube-system`**, fronted by the **`kube-dns`** service (default IP `10.96.0.10`)
> - CoreDNS config is a **Corefile**, stored in a **ConfigMap** — edit the ConfigMap to change DNS behavior, not the pod
> - Pod `/etc/resolv.conf` nameserver + search domains are set automatically by the **kubelet**, based on `clusterDNS` / `clusterDomain` in its config file
> - Services can be resolved with short names (thanks to `search`), but **pods always need their full FQDN**
> - If DNS resolution is broken, check: is the `coredns` pod running in `kube-system`? Is the `kube-dns` service up? Does the pod's `/etc/resolv.conf` point to the right nameserver?
