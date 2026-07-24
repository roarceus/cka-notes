# Ingress

## Background: Exposing Apps Without Ingress

- A typical setup: app Deployment + pod, backed by a MySQL pod exposed internally via a **ClusterIP** service
- To expose the app externally, you could use a **NodePort** service — users then hit `http://<node_IP>:<node_port>`
- For a friendlier UX, you'd point DNS at the node IPs and run a reverse proxy to forward port 80/443 → the NodePort
- **On a cloud platform**, a **LoadBalancer** service simplifies this: Kubernetes provisions a cloud load balancer that gets an external IP, which you point your DNS to directly

---

## The Need for Ingress

- Once you have multiple services (e.g. `myonlinestore.com/watch` for video, `myonlinestore.com/wear` for the store), giving each its own LoadBalancer means multiple cloud load balancers — more cost, more complexity, and messier SSL/TLS management

![GCP load balancers routing to separate wear/video services](https://kodekloud.com/kk-media/image/upload/v1752869848/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Ingress/frame_310.jpg)

- **Ingress** solves this: a single externally accessible entry point that handles URL-based routing, SSL termination, and auth — acting as a built-in **layer 7 load balancer**
- You still need an initial exposure mechanism (NodePort or cloud LoadBalancer) to reach the Ingress controller itself — after that, routing changes happen through Ingress resources

![Load balancer distributing traffic to wear/video services](https://kodekloud.com/kk-media/image/upload/v1752869849/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Ingress/frame_380.jpg)

![GCP Ingress architecture managing wear/video services](https://kodekloud.com/kk-media/image/upload/v1752869850/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Ingress/frame_440.jpg)

> [!tip] CKA Exam
> A Kubernetes cluster **does not include an Ingress controller by default**. Ingress resources have no effect until a controller (NGINX, HAProxy, Traefik, GCE, Contour, Istio, etc.) is deployed.

---

## Deploying an Ingress Controller (NGINX example)

Minimal Deployment:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
```

Exposing it via NodePort:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
  selector:
    name: nginx-ingress
```

A full setup adds a **ConfigMap** (decouples settings like SSL, timeouts, log paths from the image) and a **ServiceAccount** (with Roles/ClusterRoles/RoleBindings so the controller can watch/modify Ingress resources):

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
  selector:
    name: nginx-ingress
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
```

---

## Creating Ingress Resources

### Single backend (route everything to one service)

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
    servicePort: 80
```

```bash
kubectl create -f Ingress-wear.yaml
kubectl get ingress
NAME           HOSTS   ADDRESS   PORTS  AGE
ingress-wear   *       <none>    80     2s
```

### Routing by URL path

Route `/wear` → `wear-service`, `/watch` → `watch-service`:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
    - http:
        paths:
          - path: /wear
            backend:
              serviceName: wear-service
              servicePort: 80
          - path: /watch
            backend:
              serviceName: watch-service
              servicePort: 80
```

```bash
kubectl describe ingress ingress-wear-watch
```
```
Name:             ingress-wear-watch
Namespace:        default
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host              Path    Backends
  ----              ----    --------
  *                 /wear   wear-service:80 (<none>)
                    /watch  watch-service:80 (<none>)
```

- An undefined path (e.g. `/listen`) can be handled by a **default backend**, typically serving a 404 page

### Routing by hostname

Route `myonlinestore.com` → `primary-service`, `www.myonlinestore.com` → `secondary-service`:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-domain-routing
spec:
  rules:
    - host: myonlinestore.com
      http:
        paths:
          - path: /
            backend:
              serviceName: primary-service
              servicePort: 80
    - host: www.myonlinestore.com
      http:
        paths:
          - path: /
            backend:
              serviceName: secondary-service
              servicePort: 80
```

![Ingress rules routing different URL paths to different backends, with a 404 fallback](https://kodekloud.com/kk-media/image/upload/v1752869852/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Ingress/frame_1030.jpg)

---

> [!tip] CKA Exam
> - **Path-based routing** = one rule, multiple `paths`. **Domain-based routing** = multiple rules, each with its own `host`
> - No `host` specified in a rule → it matches **all incoming traffic** regardless of domain
> - An Ingress controller must be deployed first — Ingress resources alone do nothing
> - Know the difference between a single-backend Ingress (`spec.backend`) vs rule-based Ingress (`spec.rules`)
> - `kubectl describe ingress <name>` is the fastest way to check configured rules and backends
