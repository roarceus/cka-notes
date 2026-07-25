# Gateway API: Practical Guide (NGINX)

## Installing a Gateway Controller

- The Gateway API defines CRDs, but you still need a **controller** to implement them (e.g. NGINX Gateway Fabric)
- Typical install: apply the Gateway API CRDs, then install the controller via Helm:

```bash
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.6.2" | kubectl apply -f -

kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/experimental?ref=v1.6.2" | kubectl apply -f -

helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway
```

---

## GatewayClass

- Decouples Gateway config from the underlying controller implementation
- Lets a single cluster run multiple Gateway implementations side by side (e.g. NGINX + Istio)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: nginx.org/gateway-controller
```
- `controllerName` must match the string the controller itself expects (controller-specific, not user-defined)

---

## Gateway (HTTP Listener)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: All
```
- `allowedRoutes.namespaces.from: All` lets routes from **any namespace** attach to this listener (also supports `Same` or `Selector`)

---

## HTTPRoute

Routes traffic from a Gateway to a backend Service based on match conditions:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: basic-route
  namespace: default
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /app
    backendRefs:
    - name: my-app
      port: 80
```
- `parentRefs` attaches the route to a Gateway
- All requests matching path prefix `/app` are forwarded to `my-app:80`

---

## Redirects and Rewrites

**HTTP → HTTPS redirect:**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: https-redirect
  namespace: default
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - filters:
    - type: RequestRedirect
      requestRedirect:
        scheme: https
```

**Path rewrite** (`/old` → `/new`):
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: rewrite-path
  namespace: default
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /old
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          replacePrefixMatch: /new
    backendRefs:
    - name: my-app
      port: 80
```

---

## Header Modification

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: header-mod
  namespace: default
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        add:
          x-env: staging
    backendRefs:
    - name: my-app
      port: 80
```
- `RequestHeaderModifier` can `add`, `set`, or `remove` request headers before forwarding

---

## Traffic Splitting

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: traffic-split
  namespace: default
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - backendRefs:
    - name: v1-service
      port: 80
      weight: 80
    - name: v2-service
      port: 80
      weight: 20
```
- Weighted natively, no annotations — useful for canary rollouts / A-B testing

---

## Request Mirroring

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: request-mirror
  namespace: default
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - filters:
    - type: RequestMirror
      requestMirror:
        backendRef:
          name: mirror-service
          port: 80
    backendRefs:
    - name: my-app
      port: 80
```
- Sends a **copy** of each request to `mirror-service` (for testing/analysis) while the original request still goes to `my-app` — the primary flow is unaffected

---

## TLS Termination

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway-tls
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - kind: Secret
        name: tls-secret
    allowedRoutes:
      namespaces:
        from: All
```
- `tls.mode: Terminate` decrypts traffic at the Gateway; backend services receive plain HTTP
- `certificateRefs` points to a Secret holding the cert + key

---

## Non-HTTP Protocols (TCP / UDP / gRPC)

**TCP** (e.g. exposing a database):
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: tcp-gateway
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
  - name: tcp
    protocol: TCP
    port: 3306
    allowedRoutes:
      namespaces:
        from: All
```

**UDP** (e.g. DNS):
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: udp-gateway
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
  - name: udp
    protocol: UDP
    port: 53
    allowedRoutes:
      namespaces:
        from: All
```

**gRPC** (via HTTPRoute, matching on service/method):
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: grpc-route
  namespace: default
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - matches:
    - method:
        service: my.grpc.Service
        method: GetData
    backendRefs:
    - name: grpc-service
      port: 50051
```

---

> [!tip] CKA Exam
> - Gateway API supports **non-HTTP protocols** (TCP, UDP) directly on Gateway listeners — Ingress cannot do this natively
> - Redirects/rewrites/header-mods/mirroring/traffic-splits are all expressed as **`filters`** or native fields on `HTTPRoute` — no controller-specific annotations needed
> - `RequestRedirect` (scheme change), `URLRewrite` (path rewrite), `RequestHeaderModifier`, and `RequestMirror` are the key filter types to recognize
> - TLS termination is configured on the **Gateway listener** (`tls.mode: Terminate` + `certificateRefs`), not on the HTTPRoute
> - `allowedRoutes.namespaces.from` controls cross-namespace route attachment (`All`, `Same`, or `Selector`)
