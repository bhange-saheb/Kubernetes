# ConfigMap and Secrets In K8s

In Kubernetes, **ConfigMaps** and **Secrets** are the dynamic duo used to separate your application code from its configuration. This keeps your containerized apps portable and secure without requiring you to bake settings directly into your container images.

---

## ConfigMaps: For Non-Sensitive Data

A ConfigMap is used to store API URLs, database hostnames, environment flags (like `DEBUG=true`), or entire configuration files (like `nginx.conf`).

### Key Characteristics

* **Plain Text:** Data is stored in clear text. Anyone with access to the cluster can read it.
* **Versatile:** Can be injected into containers as environment variables, command-line arguments, or mounted as files in a volume.

---

## Secrets: For Sensitive Data

A Secret is designed to hold confidential information like passwords, OAuth tokens, SSH keys, or database credentials.

### Key Characteristics

* **Base64 Encoded (Not Encrypted by Default):** By default, Kubernetes obfuscates Secret data using Base64 encoding. *Warning:* Base64 is **not** encryption; anyone can easily decode it. To truly secure Secrets, you must enable Encryption at Rest in your cluster or use an external vault.
* **Memory-Backed:** Secrets are often mounted into pods using `tmpfs` (RAM-backed storage), meaning the sensitive data never touches the node's physical disk.

---

##  Quick Comparison

| Feature | ConfigMap | Secret |
| --- | --- | --- |
| **Primary Use Case** | Public/Non-sensitive configuration | Private/Sensitive credentials |
| **Data Storage** | Plain text string | Base64 encoded string |
| **Size Limit** | 1 MB | 1 MB |
| **Types Available** | Generic only | Multiple (Opaque, TLS, ServiceAccount, etc.) |

---

 **Kubernetes Configuration Guide for ConfigMaps & Secrets** structured with step-by-step procedures to be followed based on scenerios.

---

# Kubernetes Configuration Guide: ConfigMaps & Secrets

This guide provides operational workflows for managing non-sensitive and sensitive configuration data in Kubernetes. By separating configuration from your application code, you ensure workload portability and security.

**Kubernetes Configuration Guide for ConfigMaps & Secrets** structured with step-by-step procedures to be followed based on scenerios.
---

## Scenario 1: Private Registry Authentication (`imagePullSecrets`)

When pulling images from a private container registry (like Docker Hub private repos, ECR, or GitHub Packages), Kubernetes needs authentication credentials.

### Step 1: Create the Docker Registry Secret

Run the following command to create a specialized secret of type `kubernetes.io/dockerconfigjson`:

```bash
kubectl create secret docker-registry docker-pwd \
  --docker-username="ankit2507" \
  --docker-password="YOUR_DOCKER_HUB_PASSWORD_OR_TOKEN" \
  --docker-email="your-email@example.com"

```

### Step 2: Reference the Secret in your Deployment

To use this secret, attach the `imagePullSecrets` array at the **Pod spec** level (not the container level):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - name: worker
        image: ankit2507/votingapp:results
      imagePullSecrets:
      - name: docker-pwd

```

---

## Scenario 2: Generic Secrets (AWS Credentials)

Use generic secrets to inject operational credentials safely into your pods.

### Approach A: Imperative Creation (CLI)

You can create secrets rapidly from literal values without handling Base64 formatting yourself:

```bash
kubectl create secret generic db-user --from-literal=username=MY_AWS_USER
kubectl create secret generic db-pass --from-literal=password='MY_AWS_PASSWORD'

```

### Approach B: Declarative Creation (YAML)

If storing configurations in code repositories, values **must** be Base64 encoded.

1. **Encode your strings:**
```bash
echo -n "AKIAIOSFODNN7EXAMPLE" | base64
# Output: REtJQUlPU0ZPRE5ON0VYQU1QTEU=

echo -n "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" | base64
# Output: d0phbHJYVXRuRkVNSS9LN01ERU5HL2JQeFJmaUNZRVhBTVBMRUtFWQ==

```


2. **Construct `secret.yaml`:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: aws-credentials
type: Opaque
data:
  accesskey: REtJQUlPU0ZPRE5ON0VYQU1QTEU=
  secretkey: d0phbHJYVXRuRkVNSS9LN01ERU5HL2JQeFJmaUNZRVhBTVBMRUtFWQ==

```


3. **Apply the manifest:**
```bash
kubectl apply -f secret.yaml

```

### Step 3: Verify & Decode Secrets

To check if the configuration loaded correctly, query the API using `jsonpath` and pass it to a decoder:

```bash
kubectl get secret aws-credentials -o jsonpath="{.data.accesskey}" | base64 --decode

```

---

## Scenario 3: Consuming Secrets via File Mount

Instead of exposing credentials as environment variables (which can accidentally leak into application logs), you can securely mount them as files.

### Step 1: Deploy with a Volume Mount

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aws-tools-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aws-cli
  template:
    metadata:
      labels:
        app: aws-cli
    spec:
      containers:
      - name: aws-container
        image: amazon/aws-cli:latest
        command: ["sh", "-c", "sleep 3600"]
        volumeMounts:
        - name: aws-creds-volume
          mountPath: /root/.aws
          readOnly: true
      volumes:
      - name: aws-creds-volume
        secret:
          secretName: aws-credentials
          items:
          - key: accesskey
            path: credentials   # Files will drop at /root/.aws/credentials

```

### Step 2: Validate Storage and AWS Access

1. Exec into the running pod:
```bash
kubectl exec -it <pod_name> -- /bin/bash

```


2. Verify the configuration files and test the cloud provider connectivity:
```bash
# Check file payload
cat /root/.aws/credentials

# Run test AWS command
aws ec2 describe-vpcs --region us-east-2
aws s3 ls

```

---

## Scenario 4: ConfigMaps for File Injection (Nginx Configuration)

ConfigMaps are highly effective for managing decoupling configurations like reverse-proxy files (`nginx.conf`).

### Step 1: Prepare the Configuration File locally

Create a local configuration block named `default.conf`:

```nginx
server {
    listen       2507;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}

```

### Step 2: Convert the File into a Kubernetes ConfigMap

Generate the resource natively from the file asset:

```bash
kubectl create configmap default-config --from-file=default.conf

```

Validate the contents inside the cluster metadata:

```bash
kubectl describe cm default-config

```

### Step 3: Deploy Nginx and Mount the Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 2507
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: config-volume
        configMap:
          name: default-config

```

Apply the setup configuration:

```bash
kubectl apply -f deployment.yaml

```

### Step 4: Expose the Deployment to Traffic

Expose your new deployment to external clients by provisioning a cloud-managed load balancer:

```bash
kubectl expose deployment nginx-web --type=LoadBalancer --name=nginx-public-service --port=2507

```