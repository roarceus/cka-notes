# Service Networking

## Services vs Pods

- Pods get their own network namespace, a virtual interface attached to a node-local bridge, and an IP from that node's subnet — pod-to-pod traffic across nodes relies on routes or an overlay network
- Services are **virtual, cluster-wide constructs** — not tied to a single node or interface
- Rather than having pods talk to each other directly, you expose a pod via a **service**, which gets a stable IP and DNS name that other pods can use to connect

---

## ClusterIP vs NodePort

- **ClusterIP** (default type): the service is reachable from any pod in the cluster, regardless of node — used for internal-only access (e.g. a database pod)
- **NodePort**: builds on ClusterIP (still gets an internal cluster IP) but also exposes the service on a fixed port on **every node**, allowing access from outside the cluster

![ClusterIP setup across three nodes](https://kodekloud.com/kk-media/image/upload/v1752869866/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Service-Networking/frame_110.jpg)

![NodePort configuration exposing a port on every node](https://kodekloud.com/kk-media/image/upload/v1752869867/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Service-Networking/frame_140.jpg)

---

## How Service Networking Works

- Every node runs a **kubelet** (creates pods via the API server + CNI plugin) and a **kube-proxy** (watches the API server for service changes and programs forwarding rules)
- When a service is created, Kubernetes assigns it an IP from the range set by the API server's `--service-cluster-ip-range` flag
- kube-proxy on each node then configures forwarding rules so traffic to the service IP:port gets routed to a backend pod

![Cluster with kubelet + kube-proxy on each node, connected to kube-apiserver](https://kodekloud.com/kk-media/image/upload/v1752869868/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Service-Networking/frame_210.jpg)

![Forwarding paths set up by kube-proxy across nodes](https://kodekloud.com/kk-media/image/upload/v1752869869/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Service-Networking/frame_280.jpg)

- kube-proxy supports multiple modes — **userspace**, **iptables** (default), and **ipvs**:
```bash
kube-proxy --proxy-mode [userspace | iptables | ipvs] ...
```

---

## Example: IP Assignment and Traffic Forwarding

A pod `db` runs on node-1 at `10.244.1.2`. Exposing it via a ClusterIP service `db-service` assigns it a service IP from the cluster's service range:

```bash
kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP           NODE
db     1/1     Running   0          14h   10.244.1.2   node-1

kubectl get service
NAME         TYPE        CLUSTER-IP      PORT(S)    AGE
db-service   ClusterIP   10.103.132.104  3306/TCP   12h
```

- Service IP range and pod network CIDR are configured separately and **must not overlap** — e.g. services from `--service-cluster-ip-range=10.96.0.0/12` (10.96.0.0–10.111.255.255) vs pods from a `10.244.0.0/16` CNI range
- kube-proxy creates a DNAT rule so traffic to the service IP:port is redirected to the pod IP:port:

```bash
iptables -L -t nat | grep db-service
KUBE-SVC-XA5OGUC7YRHOS3PU  tcp  --  anywhere  10.103.132.104  /* default/db-service: cluster IP */ tcp dpt:3306
DNAT                      tcp  --  anywhere  anywhere  /* default/db-service: */ tcp to:10.244.1.2:3306
KUBE-SEP-JBWCWHHQM57V2WN7  all  --  anywhere  anywhere  /* default/db-service: */
```

- A NodePort service works the same way, but adds iptables rules that forward traffic arriving on the node port (on every node) to the backend pods

---

## Verifying kube-proxy

```bash
cat /var/log/kube-proxy.log
```
- Log location can vary by installation — check verbosity settings if expected entries are missing

---

> [!tip] CKA Exam
> - ClusterIP is the **default** service type — internal-only, reachable from any node in the cluster
> - NodePort exposes the service externally on a fixed port on **every node**, in addition to still having a ClusterIP
> - Service IPs come from `--service-cluster-ip-range` (API server flag) — this range must **not overlap** with the pod network CIDR
> - kube-proxy is what actually programs the forwarding rules (iptables by default) — no kube-proxy, no working services
> - To debug service connectivity: check `iptables -L -t nat | grep <service-name>` for the DNAT rule, and confirm kube-proxy is running/logging on each node
