# Image Security

## Image Naming Convention

```
registry/account/image:tag

docker.io / library / nginx : latest
    ↑          ↑        ↑
 registry   account   image
```

- `nginx` → `docker.io/library/nginx` (Docker Hub official image)
- `myaccount/myapp` → `docker.io/myaccount/myapp`
- `gcr.io/google-containers/pause` → Google Container Registry
- `private-registry.io/apps/internal-app` → private registry

---

## Using a Private Registry

### Step 1 — Create a docker-registry Secret

```bash
kubectl create secret docker-registry regcred \
  --docker-server=private-registry.io \
  --docker-username=registry-user \
  --docker-password=registry-password \
  --docker-email=registry-user@org.com
```

### Step 2 — Reference it in the pod spec

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: private-registry.io/apps/internal-app
  imagePullSecrets:
  - name: regcred
```

The kubelet uses the credentials from `regcred` to authenticate with the private registry when pulling the image.

---

> [!tip] CKA Exam
> - Secret type for registry credentials is **`docker-registry`** (not `generic`)
> - The command is `kubectl create secret docker-registry` — know this exact syntax
> - `imagePullSecrets` goes at the **pod spec level** (same level as `containers`), not inside a container
> - Without `imagePullSecrets`, pulling from a private registry will fail with `ImagePullBackOff`
> - You can add `imagePullSecrets` to a ServiceAccount so all pods using that SA automatically get registry access
