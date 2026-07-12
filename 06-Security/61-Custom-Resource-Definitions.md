# Custom Resource Definitions (CRDs)

## What They Are

CRDs let you extend Kubernetes with **your own resource types**. Once defined, your custom resources can be managed with `kubectl` just like built-in resources (create, get, delete, etc.).

Two parts needed:
1. **CRD** — tells Kubernetes the resource type exists and its schema
2. **Custom Controller** — watches for the custom resource and takes action (e.g. calls an external API)

Without a controller, the resource is just stored in etcd — no automated behaviour.

---

## Custom Resource Example

```yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Mumbai
  to: London
  number: 2
```

```bash
kubectl create -f flightticket.yaml
kubectl get flightticket
kubectl delete flightticket my-flight-ticket
```

Without the CRD registered first, this errors with: `no matches for kind "FlightTicket" in version "flights.com/v1"`

---

## CRD YAML

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com    # must be <plural>.<group>
spec:
  group: flights.com                 # API group (matches apiVersion prefix)
  scope: Namespaced                  # Namespaced or Cluster
  names:
    plural: flighttickets
    singular: flightticket
    kind: FlightTicket
    shortNames:
    - ft                             # kubectl get ft
  versions:
  - name: v1
    served: true                     # this version is served by the API
    storage: true                    # this version is used for storage in etcd
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              from:
                type: string
              to:
                type: string
              number:
                type: integer
                minimum: 1
```

```bash
kubectl create -f flightticket-crd.yaml
# customresourcedefinition "flighttickets.flights.com" created

kubectl get crd
kubectl get flighttickets    # now works
kubectl get ft               # shortname works too
```

---

## Key Fields

| Field | Purpose |
|---|---|
| `metadata.name` | Must be `<plural>.<group>` |
| `spec.group` | API group — matches `apiVersion` prefix in resource YAML |
| `spec.scope` | `Namespaced` or `Cluster` |
| `served: true` | This version responds to API requests |
| `storage: true` | This version is used to store objects in etcd (only one per CRD) |

---

> [!tip] CKA Exam
> - CRD `apiVersion` is **`apiextensions.k8s.io/v1`**
> - CRD `metadata.name` must follow the pattern `<plural>.<group>` exactly
> - `kubectl get crd` lists all registered custom resource definitions
> - Creating a CRD alone doesn't "do" anything — a **custom controller** must watch the resource to take action
> - This is conceptual knowledge for CKA — you're unlikely to write a full CRD from scratch, but you should understand what they are and how to inspect them
