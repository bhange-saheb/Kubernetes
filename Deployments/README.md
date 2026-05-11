# Kubernetes Deployments

A **Deployment** in Kubernetes provides declarative updates for Pods and ReplicaSets. It’s the industry‑standard way to manage stateless applications, ensuring scalability, reliability, and easy rollouts/rollbacks.


## Introduction
Deployments manage **ReplicaSets**, which in turn manage Pods. They allow you to:
- Scale applications up or down.
- Perform rolling updates with zero downtime.
- Roll back to previous versions if needed.


## example yaml file

```bash

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: example-daemonset
  labels:
    app: example
spec:
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: pause
        image: registry.k8s.io/pause

```
### Apply the yaml manifest

```bash
kubectl apply -f daemonset.yaml
```
or echo '
data of yaml file' | kubectl -apply -f -

### Verify Pods across nodes
```bash
kubectl get pods -o wide
```

## Best Practices
- **Use labels consistently**: Ensure selectors match Pod labels.  
- **Set resource requests/limits**: Prevent resource starvation.  
- **Configure liveness/readiness probes**: Improve reliability.  
- **Use rolling updates**: Avoid downtime during upgrades.  
- **Version your images**: Always tag images (`v1`, `v2`) instead of `latest`.  
- **Monitor deployments**: Use tools like Prometheus, Grafana, or Kubernetes Dashboard.  


## Common Commands
- **View Deployments**:  
  ```bash
  kubectl get deployments
  ```
- **Scale Deployment**:  
  ```bash
  kubectl scale deployment kubeapp --replicas=10
  ```
- **Check rollout status**:  
  ```bash
  kubectl rollout status deployment kubeapp
  ```
- **Delete Deployment**:  
  ```bash
  kubectl delete deployment kubeapp
  ```

---

## Cleanup
When finished, delete resources to free cluster capacity:
```bash
kubectl delete deployment kubeapp
kubectl delete svc kubeapp
```


