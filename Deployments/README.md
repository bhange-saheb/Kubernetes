# Kubernetes Deployments , DaemonSet and StatefullSet  

The built-in APIs for managing workloads are:
1. Deployments
2. DaemonSet
3. StatefullSet

A **Deployment** in Kubernetes provides declarative updates for Pods and ReplicaSets. It’s the industry‑standard way to manage stateless applications, ensuring scalability, reliability, and easy rollouts/rollbacks.

## Introduction
Deployments manage **ReplicaSets**, which in turn manage Pods. They allow you to:
- Scale applications up or down.
- Perform rolling updates with zero downtime.
- Roll back to previous versions if needed.

Production use case: Deploying a REST API, frontend application, authentication service, or microservice.


## example yaml file

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
  labels:
    app: example
spec:
  replicas: 3
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
kubectl apply -f deployment.yaml
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
  kubectl scale deployment example-deployment --replicas=10
  ```
- **Check rollout status**:  
  ```bash
  kubectl rollout status deployment example-deployment
  ```
- **Delete Deployment**:  
  ```bash
  kubectl delete deployment example-deployment
  ```

---

## Cleanup
When finished, delete resources to free cluster capacity:
```bash
kubectl delete deployment example-deployment
kubectl delete svc example-deployment-service
```

## DaemonSet — Node-Level Agents

Production use case: Running a log collector, node monitoring agent, security agent, or networking component on every eligible Kubernetes node.

A DaemonSet ensures that a Pod runs on each node matching its scheduling requirements. When eligible nodes are added, Kubernetes creates the corresponding Pods automatically.

Important: Shown example is a scheduling and resource-management example, not a complete log-collection deployment. The --dry-run argument is for illustrative validation behavior and must be replaced with the actual collector configuration. A working log agent also needs suitable log-volume mounts, input configuration, and an output destination.

Why these settings matter
DaemonSet — Maintains an agent Pod on every eligible node.

nodeSelector — Restricts scheduling to Linux nodes.

tolerations — Allows scheduling onto tainted nodes where appropriate; review the broad toleration before using it in production.

maxUnavailable: 1 — Limits the number of DaemonSet Pods that can be unavailable during a rolling update.

resources — Helps prevent node-level agents from consuming excessive CPU or memory.

automountServiceAccountToken: false — Avoids mounting API credentials when Kubernetes API access is unnecessary.

##  StatefulSet — Stateful Applications
Production use case: Running databases, distributed data stores, and clustered services that need stable Pod identities and persistent storage.

A StatefulSet provides stable Pod names, ordered deployment and scaling behavior, and persistent-volume claim management.

This example demonstrates a three-replica PostgreSQL workload using a StatefulSet and per-Pod storage. It is a storage and identity example, not a complete production PostgreSQL cluster.

Important production caveats
Create the production namespace and the postgres-credentials Secret before deployment.

The Secret must contain a password key. Avoid committing real credentials to Git; use a secret manager or an appropriate external-secrets integration.

Ensure the cluster has a suitable persistent-volume provisioner and StorageClass.

The postgres:17.6 image is an example version pin; select a supported, tested version and preferably pin its image digest.

Three PostgreSQL Pods do not automatically form a highly available PostgreSQL cluster. Each Pod initializes its own independent database. Replication, leader election, failover, backups, and recovery require additional configuration or a database operator.

The readiness probe above checks database connectivity, not replication health or application-level correctness.

Why these settings matter
serviceName — Associates the StatefulSet with a headless Service for stable network identities.

volumeClaimTemplates — Creates a separate persistent volume claim for each Pod.

whenDeleted: Retain — Retains associated PVCs when the StatefulSet is deleted.

OrderedReady — Creates and scales Pods in order, waiting for readiness.

Secret references — Keeps the database password out of the Pod manifest itself.

Production additions: Use a database operator or a properly configured replication architecture, encrypted storage, tested backups, restore drills, disruption planning, monitoring, and a controlled upgrade procedure.

### Deploy and validate the manifests

# Validate YAML against the Kubernetes API server
kubectl apply --dry-run=server -f deployment.yaml
kubectl apply --dry-run=server -f daemonset.yaml
kubectl apply --dry-run=server -f statefulset.yaml

# Deploy after completing the required configuration
kubectl apply -f deployment.yaml
kubectl apply -f daemonset.yaml
kubectl apply -f statefulset.yaml

# Inspect workload status
kubectl get deployments -n production
kubectl get daemonsets -n monitoring
kubectl get statefulsets -n production

# Inspect Pods and persistent storage
kubectl get pods -A -o wide
kubectl get pvc -n production

# Troubleshoot an individual workload
kubectl describe deployment payment-api -n production
kubectl describe daemonset node-log-agent -n monitoring
kubectl describe statefulset postgres -n production

#  The most important production lesson
Think about the requirement first, then choose the controller:

Deployment: "I need multiple copies of my application, and they can be replaced interchangeably."

DaemonSet: "I need an agent on every eligible node."

StatefulSet: "I need stable Pod identities and persistent storage, or ordered management of stateful instances."