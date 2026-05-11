
---

# Kops (Kubernetes Operations)

Kops is a CLI tool for creating, managing, and deleting Kubernetes clusters. It’s widely used in production environments to provision clusters on cloud providers like AWS and GCP.

---

##  What This README Covers
- Introduction  
- Prerequisites  
- Cluster Lifecycle  
- Best Practices  
- Common Commands  
- Cleanup  

---

## Introduction
Kops automates cluster management tasks:
- **Cluster creation** with infrastructure provisioning.  
- **Cluster upgrades** with rolling updates.  
- **Cluster deletion** with safe teardown of resources.  

It’s often described as “kubectl for clusters.”

---

## Prerequisites
- AWS, GCP, or other supported cloud provider account.  
- DNS configured for your cluster domain (e.g., `k8s.example.com`).  
- `kubectl` installed and configured.  
- `kops` installed on your workstation.  

---

## Cluster Lifecycle

### 1. Create a cluster
```bash
kops create cluster --name=k8s.example.com --state=s3://my-kops-state-store --zones=us-east-1a,us-east-1b --node-count=3 --node-size=t3.medium --master-size=t3.medium
```

### 2. Apply the configuration
```bash
kops update cluster k8s.example.com --yes
```

### 3. Validate the cluster
```bash
kops validate cluster
```

### 4. Upgrade the cluster
```bash
kops rolling-update cluster --yes
```

### 5. Delete the cluster
```bash
kops delete cluster k8s.example.com --yes
```

---

## Best Practices
- **Use a state store**: Keep cluster state in an S3 bucket or GCS bucket.  
- **Version control manifests**: Store `cluster.yaml` in Git for reproducibility.  
- **Enable logging/monitoring**: Integrate with tools like Prometheus and ELK.  
- **Use multiple zones**: Improve availability by spreading nodes across zones.  
- **Automate with CI/CD**: Apply cluster changes via pipelines for consistency.  
- **Secure secrets**: Use KMS or Vault for sensitive data.  

---

## 🛠️ Common Commands
- **List clusters**:  
  ```bash
  kops get clusters
  ```
- **Edit cluster configuration**:  
  ```bash
  kops edit cluster k8s.example.com
  ```
- **Export kubeconfig**:  
  ```bash
  kops export kubecfg k8s.example.com
  ```
- **Rolling update nodes**:  
  ```bash
  kops rolling-update cluster --yes
  ```

---

## 🧹 Cleanup
To safely tear down a cluster:
```bash
kops delete cluster k8s.example.com --yes
```
This removes all associated cloud resources.

---