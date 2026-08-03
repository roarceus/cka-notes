# Troubleshooting: Application Failure

## Setup Context

- Two-tier app: a web pod (frontend) served via a web Service, talking to a database pod served via a DB Service
- Before troubleshooting, it helps to have a mental (or written) map of the components and how they connect

---

## Troubleshooting the Web Application

### 1. Test Reachability
```bash
curl http://web-service-ip:node-port
# curl: (7) Failed to connect ... Connection timed out
```

### 2. Check Service Endpoints
```bash
kubectl describe service web-service
```
Look at the `Selector` and `Endpoints` fields:
```
Selector:    name=webapp-mysql
Endpoints:   10.32.0.6:8080
```
- **A common root cause: the Service's selector doesn't match the pod's labels**, resulting in an empty `Endpoints` list — the service has nothing to route traffic to

### 3. Verify Pod Config and Status
```bash
kubectl get pod
```
```
NAME   READY   STATUS    RESTARTS   AGE
Web    1/1     Running   5          50m
```
- High restart counts are a red flag even if status currently shows `Running`

```bash
kubectl describe pod web
```
- Check the `Events` section for scheduling/image-pull/container-start problems

### 4. Review Logs
```bash
kubectl logs web
kubectl logs web -f     # stream in real time if the issue is intermittent/ongoing
```

---

## Troubleshooting the Database Service

Apply the same sequence to the DB side:
1. Inspect the DB service (`kubectl describe service`)
2. Confirm endpoints are populated
3. Check DB pod status
4. Review DB pod logs

> [!tip] CKA Exam
> Selector/label mismatches between a Service and its target Pod are one of the most common causes of "can't connect" issues — always check `Endpoints` in `kubectl describe service` first.

---

## Dependency-Chain Troubleshooting Order

For a dependent application chain, troubleshoot from the **bottom up** — start at the database, then its service, then the web pod, then its service:

![Troubleshooting flow: DB → DB-Service → WEB → WEB-Service](https://kodekloud.com/kk-media/image/upload/v1752869994/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Application-Failure/frame_160.jpg)

---

> [!tip] CKA Exam
> - Checklist for any "app not reachable" issue: (1) `curl`/test connectivity, (2) `kubectl describe service` → check selector + endpoints, (3) `kubectl get pod` / `describe pod` → check status, restarts, events, (4) `kubectl logs [-f]`
> - For dependent services, troubleshoot the **lowest layer first** (e.g. database before the web tier that depends on it)
> - `kubectl describe pods ${POD_NAME}` is the fastest way to get comprehensive pod state + events in one command
