# Services

## What They Do

Services provide **stable networking** for pods — a consistent IP/DNS name that doesn't change even as pods come and go. They also enable external access to applications running inside the cluster.

---

## Service Types

![Service types overview](https://kodekloud.com/kk-media/image/upload/v1752869746/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Services/frame_280.jpg)

| Type | Purpose |
|---|---|
| **ClusterIP** | Internal-only virtual IP — default type. Used for pod-to-pod communication (e.g. front-end → back-end) |
| **NodePort** | Exposes a port on every node → forwards to a pod. Used for external access |
| **LoadBalancer** | Provisions a cloud load balancer (AWS/GCP/Azure). Builds on NodePort |

---

## NodePort — Port Terminology

Three ports are involved:

```
External user
     ↓
[NodePort]  30000–32767  — port on the Node
     ↓
[Port]      80           — port on the Service (virtual)
     ↓
[targetPort] 80          — port on the Pod
```

- `targetPort` defaults to `port` if omitted
- `nodePort` is auto-assigned if omitted (within 30000–32767)

![NodePort diagram](https://kodekloud.com/kk-media/image/upload/v1752869747/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Services/frame_750.jpg)

---

## NodePort Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80    # pod port
    port: 80          # service port
    nodePort: 30008   # external node port (30000-32767)
  selector:
    app: myapp        # must match pod labels
    type: front-end
```

```bash
kubectl create -f service-definition.yaml

kubectl get services
# NAME           TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
# myapp-service  NodePort   10.106.127.123   <none>        80:30008/TCP   5m

# Access externally
curl http://<node-ip>:30008
```

---

## Multi-Pod Behaviour

- Service with a selector **automatically discovers all matching pods** across nodes
- Traffic is distributed via **round-robin** across all matching pods — built-in load balancing
- If pods are on multiple nodes, the NodePort is opened on **all nodes** — any node IP works

---

## ClusterIP Service YAML

Default type — used for **internal pod-to-pod communication** (e.g. front-end → back-end). No external access.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP    # default — can be omitted
  ports:
  - port: 80         # service port
    targetPort: 80   # pod port
  selector:
    app: myapp
    type: back-end
```

Other pods reach this service via its **ClusterIP** or **DNS name** (e.g. `http://back-end`).

---

## Essential Commands

```bash
# Create
kubectl create -f service-definition.yaml

# Imperative (expose an existing pod/deployment)
kubectl expose pod nginx --port=80 --type=NodePort
kubectl expose deployment myapp --port=80 --type=NodePort --name=myapp-service

# List
kubectl get services
kubectl get svc          # shorthand

# Describe
kubectl describe service myapp-service

# Delete
kubectl delete service myapp-service
```

---

> [!tip] CKA Exam
> - **ClusterIP is the default** — if no `type` is specified, you get ClusterIP
> - NodePort range is **30000–32767** — assigning outside this range will error
> - The `selector` links the service to pods — if it doesn't match, endpoints will be empty (`kubectl describe svc` shows `Endpoints: <none>`)
> - `kubectl expose` is the fastest imperative way to create a service in the exam
> - Check endpoints if a service isn't routing: `kubectl get endpoints myapp-service`
