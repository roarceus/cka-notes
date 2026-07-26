# High Availability

## Why HA Matters

- If the master node goes down but workers stay alive, running applications keep serving traffic — for now
- Problems start once something needs to change: if a pod crashes, the replication controller (on the master) needs to instruct scheduling of a replacement — but with the master down, **no controller and no scheduler are available** to recreate or place it
- The **kube-apiserver** is also down, so `kubectl`/API access for cluster management is unavailable
- **HA = redundancy across every layer**: master nodes, worker nodes, control plane components, and applications (already handled via ReplicaSets/Services) — this lecture focuses on master/control plane HA

---

## How Each Control Plane Component Handles Multiple Masters

### kube-apiserver — Active-Active
- Processes one request at a time and doesn't hold cluster-wide state itself, so **multiple API server instances can run simultaneously**, all active
- With multiple masters, you can't point `kubectl` at just one master anymore — put a **load balancer** (NGINX, HAProxy, etc.) in front of the API servers, and point `kubectl`/kubeconfig at the load balancer instead of a single master's port 6443

### controller-manager & scheduler — Active-Standby
- These continuously watch cluster state and take action (e.g. replication controller recreating failed pods)
- Running multiple instances **in parallel** would cause duplicate actions (e.g. extra pods) — so only **one instance is active at a time**, decided via **leader election**

**Leader election mechanics** (controller-manager, and scheduler follows the same approach with the same flags):
- `--leader-elect` (default `true`): enables leader election
- On startup, each instance tries to acquire a lock on an **Endpoint object** named `kube-controller-manager` (in Kubernetes)
- Whichever instance updates that endpoint first **becomes the leader (active)**; the rest stay **passive**
- `--leader-elect-lease-duration` (default **15s**): how long the lock is held before it can be taken over
- `--leader-elect-renew-deadline` (default **10s**): how often the active instance renews its lease
- `--leader-elect-retry-period` (default **2s**): how often instances attempt to acquire leadership
- If the active master crashes, another instance can acquire the lock once the lease expires and become the new leader

### etcd — Two Topologies

| | Stacked (etcd on control plane nodes) | External etcd |
|---|---|---|
| Setup | etcd runs on the same nodes as the control plane (what's been used throughout the course) | etcd runs on its own dedicated set of servers |
| Ease of setup | Easier, fewer nodes required | Harder, needs roughly **2x the nodes** |
| Failure impact | Losing a control plane node loses **both** a control plane instance *and* an etcd member — redundancy is compromised | A failed control plane node does **not** affect the etcd cluster or its data — lower risk |

- Only the **kube-apiserver** talks directly to etcd — its config specifies the etcd server address(es)
- Since etcd is a **distributed system**, the API server can read/write via **any** available etcd instance — this is why the API server config lists **multiple etcd server addresses**

---

## Resulting Cluster Design

- Original plan: 1 master, 2 workers
- With HA: multiple masters + a load balancer in front of the API servers
- New total: **5 nodes** (multiple masters + load balancer consideration + the 2 original workers)

---

> [!tip] CKA Exam
> - **kube-apiserver**: active-active, needs a **load balancer** in front when there are multiple masters
> - **controller-manager** and **scheduler**: active-standby via **leader election**, not active-active — running both active would cause duplicate scheduling actions
> - Know the leader-election flags and defaults: `--leader-elect=true`, `--leader-elect-lease-duration=15s`, `--leader-elect-renew-deadline=10s`, `--leader-elect-retry-period=2s`
> - Leader election lock is held on an **Endpoint object** (`kube-controller-manager`)
> - **Stacked etcd** = simpler, fewer nodes, but control plane + etcd failure are coupled. **External etcd** = more resilient, but needs ~2x the servers
> - Only the **API server** talks to etcd directly — check its config for the etcd server address list when troubleshooting
