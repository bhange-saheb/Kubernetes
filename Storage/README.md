# K8s Storage Options:

Kubernetes divides storage into two main operational philosophies: **Ephemeral Storage** (dies when the pod dies) and **Persistent Storage** (lives independently of the pod).

To manage this properly, Kubernetes abstracts the infrastructure so developers can request storage without needing to understand the underlying cloud or physical disks.

---

##  1. Ephemeral Storage (Temporary)

Ephemeral options are excellent for caching, scratch space, or injecting system data. When a Pod is deleted, this data disappears forever.

### `emptyDir`

* **What it is:** A blank directory created on the host Node when a Pod starts up.
* **Use Case:** Scratch space for multi-step data processing, or a shared space where a helper container downloads files that the main application container needs to read.
* **Storage Medium:** Can be backed by the host's hard drive or RAM (`tmpfs`).

### `HostPath`
* Purpose: Mounts a directory from the host machine into the pod.
* Example: You can use HostPath to access Docker’s Unix socket at /var/run/docker.sock from inside the container. This lets the  container communicate with Docker on the host.

### `configMap` / `secret` / `downwardAPI`

* **What it is:** Specialized volume types that fetch configuration details from the Kubernetes API and inject them as readable files into the pod.

---

## 2. Persistent Storage (The Core Framework)

For stateful applications like databases (PostgreSQL, MySQL, MongoDB), data must survive pod restarts, updates, and node crashes. Kubernetes manages this using three distinct resources.

### A. PersistentVolume (PV)

Think of a PV as a **physical piece of storage** (like a network disk, AWS EBS volume, Google Persistent Disk, or NFS share) that an administrator has attached to the cluster. It has a life cycle independent of any individual Pod that uses it.

### B. PersistentVolumeClaim (PVC)

A PVC is a **ticket or request** for storage made by a user. It says: *"I need 10GB of storage with ReadWriteOnce access."* Kubernetes looks at the available PV pool and binds the claim to an eligible volume.

### C. StorageClass (SC)

Instead of an administrator manually provisioning 100 different PVs ahead of time (**Static Provisioning**), they create a `StorageClass`. When a user requests a PVC, the `StorageClass` automatically talks to the cloud provider to spin up a disk on demand (**Dynamic Provisioning**).

`Stroage class automatically created when we create cluster via KOPS`  

---

##  Understanding Access Modes

When requesting persistent storage via a PVC, you must specify how your application needs to connect to the backend disk. Not all cloud providers or storage types support every mode.

| Access Mode | Abbreviation | Explanation | Typical Backend |
| --- | --- | --- | --- |
| **ReadWriteOnce** | `RWO` | Can be mounted as read-write by a **single node** at a time. Multiple pods on that *same* node can read/write to it. | AWS EBS, GCP Persistent Disk |
| **ReadOnlyMany** | `ROX` | Can be mounted as read-only by **many nodes** simultaneously. Great for sharing static assets. | Shared Network File Systems |
| **ReadWriteMany** | `RWX` | Can be mounted as read-write by **many nodes** simultaneously. | AWS EFS, NFS, Ceph, GlusterFS |
| **ReadWriteOncePod** | `RWOP` | Restricts access so that exactly **one single pod** in the entire cluster can read or write to it. | Modern CSI Supported Disks |

---

## 🛠️ Step-by-Step Example: Dynamic Provisioning

Here is how a developer requests and uses persistent cloud storage seamlessly using a `StorageClass`.

### Step 1: The Request (PVC)

The developer writes a `pvc.yaml` file asking for storage without hardcoding cloud disk details:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: database-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: standard  # References the cluster's defined StorageClass

```

### Step 2: Consuming the Storage in a Pod

The developer mounts that exact claim directly inside the application container manifest (`pod.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
spec:
  containers:
  - name: mysql
    image: mysql:8.0
    volumeMounts:
    - name: mysql-persistent-storage
      mountPath: /var/lib/mysql # Where MySQL saves data
  volumes:
  - name: mysql-persistent-storage
    persistentVolumeClaim:
      claimName: database-pvc # Binds the Pod to the PVC created above

```

When you apply these files, the `StorageClass` automatically instructions your cloud provider to provision a 20GB disk, maps it as a `PersistentVolume`, binds it to your `PersistentVolumeClaim`, and hooks it straight into your MySQL container directory.

---
