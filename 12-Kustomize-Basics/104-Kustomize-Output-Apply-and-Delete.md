# Kustomize Output: Apply and Delete

## Recap: Deploying with Kustomize

`kustomize build` only renders output — nothing is deployed until it's combined with `kubectl`. Two equivalent ways to deploy:

```bash
kustomize build k8s/ | kubectl apply -f -
# service/nginx-loadbalancer-service created
# deployment.apps/nginx-deployment created
```

```bash
kubectl apply -k k8s/
# service/nginx-loadbalancer-service created
# deployment.apps/nginx-deployment created
```
- `-k` is kubectl's native Kustomize support — same result as piping `kustomize build` into `kubectl apply -f -`, without needing the separate `kustomize` binary in the pipeline

---

## Deleting Resources

Same pattern as applying, just swap `apply` for `delete`:

```bash
kustomize build k8s/ | kubectl delete -f -
# service "nginx-loadbalancer-service" deleted
# deployment.apps "nginx-deployment" deleted
```

```bash
kubectl delete -k k8s/
# service "nginx-loadbalancer-service" deleted
# deployment.apps "nginx-deployment" deleted
```

---

> [!tip] CKA Exam
> - `kubectl apply -k <dir>` / `kubectl delete -k <dir>` are the direct native equivalents of piping `kustomize build <dir>` into `kubectl apply -f -` / `kubectl delete -f -`
> - Both apply and delete operate on whatever `kustomization.yaml` currently defines — deleting removes exactly what would currently be created by an apply
