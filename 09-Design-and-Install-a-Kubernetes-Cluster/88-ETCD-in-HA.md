# ETCD in HA

## What is ETCD

- A distributed, reliable **key-value store** — fast and secure
- Stores data as documents (JSON/YAML/etc.), not tables — each document is self-contained, so changing one doesn't affect others

---

## Distributed Cluster Behavior

- A typical ETCD cluster runs identical instances across multiple servers (e.g. 3 nodes) — if one fails, the rest still hold accurate data
- **Reads** can be served by any node — since all nodes hold the same data
- **Writes** are more restrictive: only the **leader** processes writes. If a write lands on a follower, it's forwarded to the leader
- A write is only confirmed once the leader gets acknowledgement from a **majority of nodes**

---

## Raft Consensus Protocol

- On cluster startup, there's no leader — each node has a randomized timeout; when a node's timeout expires, it starts an **election**
- The candidate requests votes from peers; once it gets a majority, it becomes **leader** and sends periodic **heartbeats** to maintain control
- If the leader fails or loses network connectivity, remaining nodes trigger a **new election**
- This process keeps writes consistent and all nodes' data in sync

---

## Quorum and Fault Tolerance

- A write must be replicated to a **majority (quorum)** of nodes to succeed
- **Quorum formula:** `(total nodes / 2) + 1`
  - 3 nodes → quorum of 2
  - 5 nodes → quorum of 3
- **Odd-numbered clusters are preferred** — even-numbered clusters can split into two equal halves during a network partition, and neither half reaches quorum, causing full cluster failure
  - Example: 6-node cluster splits 3/3 → neither half has quorum (needs 4) → cluster fails
  - Example: 6-node cluster splits 4/2 → the 4-side has quorum → cluster survives
  - Example: 7-node cluster splits 4/3 → the 4-side has quorum → cluster survives
- Recommended cluster sizes: **3, 5, or 7** nodes

![Table of instance count, quorum, and fault tolerance, alongside node diagram with etcd/API server/controller-manager/scheduler](https://kodekloud.com/kk-media/image/upload/v1752869774/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-ETCD-in-HA/frame_690.jpg)

---

## Installing and Configuring ETCD

```bash
wget -q --https-only "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"
tar -xvf etcd-v3.3.9-linux-amd64.tar.gz
```

Example systemd service configuration — note the **`--initial-cluster`** option, which introduces each instance to its peers:

```bash
ExecStart=/usr/local/bin/etcd \
  --name ${ETCD_NAME} \
  --cert-file=/etc/etcd/kubernetes.pem \
  --key-file=/etc/etcd/kubernetes-key.pem \
  --peer-cert-file=/etc/etcd/kubernetes.pem \
  --peer-key-file=/etc/etcd/kubernetes-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth \
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \
  --listen-peer-urls https://${INTERNAL_IP}:2380 \
  --listen-client-urls https://${INTERNAL_IP}:2380,https://127.0.0.1:2379 \
  --advertise-client-urls https://${INTERNAL_IP}:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster peer-1=https://${PEER1_IP}:2380,peer-2=https://${PEER2_IP}:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
```

---

## Using etcdctl

```bash
export ETCDCTL_API=3

etcdctl put name john
etcdctl get name
# name
# john
```

---

## Choosing Cluster Size

- Minimum **3 nodes** for real HA — 1 or 2 nodes can't maintain quorum if a node fails
- **5 nodes** gives stronger resilience without excessive overhead
- A 2-node setup (e.g. on a laptop for experimentation) provides **no real fault tolerance**
- Fewer than 3 nodes in production risks a **split-brain** scenario during network partitions

![Network design with load balancer, ETCD master nodes, and worker nodes](https://kodekloud.com/kk-media/image/upload/v1752869776/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-ETCD-in-HA/frame_740.jpg)

---

> [!tip] CKA Exam
> - Quorum formula: **`(N / 2) + 1`** — know how to calculate for any node count
> - **Odd-numbered clusters (3, 5, 7)** are preferred over even — even clusters can split into two equal halves with neither reaching quorum
> - Only the **leader** processes writes; reads can go to any node
> - Writes require acknowledgement from a **majority** of nodes before being confirmed
> - Minimum **3 nodes** for genuine HA in production — fewer risks split-brain
> - `--initial-cluster` in the etcd service config is what tells each instance about its peers
> - `etcdctl` v3 API requires `export ETCDCTL_API=3` before use
