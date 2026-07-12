# Custom Controllers

## What They Do

A custom controller is a process that **continuously monitors** custom resources (CRDs) in Kubernetes and **takes action** when they are created, updated, or deleted.

Without a controller, a custom resource is just data sitting in etcd — nothing happens.

---

## How It Works

```
CRD created → API server stores FlightTicket in etcd
                        ↓
            Custom Controller watches for FlightTicket events
                        ↓
            Controller calls external API (e.g. bookflight.com)
                        ↓
            Ticket booked / cancelled / updated
```

The controller runs a **reconciliation loop** — constantly comparing desired state (what's in etcd) with actual state (what's happened in the real world) and taking steps to close the gap.

---

## Implementation

- Typically written in **Go** using the Kubernetes `client-go` library
- Uses **shared informers** for built-in queuing and caching (avoids expensive repeated API calls)
- Deployed as a **pod or Deployment** inside the cluster (uses kubeconfig or in-cluster config to authenticate)

```bash
# Run locally against a cluster
./my-controller --kubeconfig=$HOME/.kube/config

# Or deploy in-cluster as a Deployment
kubectl apply -f my-controller-deployment.yaml
```

---

## CRD + Controller = Operator

A **Kubernetes Operator** is the combination of a CRD and a custom controller that together manage a complex stateful application (e.g. databases, message queues) using Kubernetes-native patterns.

---

> [!tip] CKA Exam
> - You won't be asked to write a controller — understand **what they are conceptually**
> - Know that a CRD alone is inert — a controller gives it behaviour
> - Controllers are deployed as regular pods/Deployments — inspect them with `kubectl get pods` / `kubectl logs`
> - The pattern CRD + Controller = **Operator** is a key Kubernetes ecosystem concept
