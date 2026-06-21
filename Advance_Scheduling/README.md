# Advanced Scheduling in Kubernetes

The Kubernetes scheduler’s default behavior works well for most cases -- for example, it ensures that pods are only placed on nodes that have sufficient free resources, it ties to spread pods from the same set (ReplicaSet, StatefulSet, etc.) across nodes, it tries to balance out the resource utilization of nodes, etc.

But sometimes you want to control how your pods are scheduled. For example, perhaps you want to ensure that certain pods only schedule on nodes with specialized hardware, or you want to co-locate services that communicate frequently, or you want to dedicate a set of nodes to a particular set of users. Ultimately, you know much more about how your applications should be scheduled and deployed than Kubernetes ever will. So Kubernetes 1.6 offers four advanced scheduling features: node affinity/anti-affinity, taints and tolerations, pod affinity/anti-affinity, and custom schedulers. Each of these features are now in beta in Kubernetes 1.6.

By default, the Kubernetes scheduler does a pretty solid job. It makes sure pods only land on nodes with enough free resources, spreads replicas across nodes to improve resilience, and balances workloads so no single node gets overwhelmed. For most applications, this “out-of-the-box” behavior works just fine.

But sometimes, you need more control. Imagine you’re running workloads that require specialized hardware like GPUs, or you want two services that talk to each other constantly to run side by side for faster communication. Maybe you even want to reserve a group of nodes for a specific team or project. In these cases, the default scheduler can’t anticipate your unique needs—because you know your applications better than Kubernetes ever will.

That’s why Kubernetes 1.6 introduced **advanced scheduling features** to give you more flexibility:

- **Node affinity/anti-affinity** → Decide which nodes pods should (or shouldn’t) run on.  
- **Taints and tolerations** → Mark nodes for special use and control which pods can tolerate them.  
- **Pod affinity/anti-affinity** → Influence how pods are placed relative to each other.  
- **Custom schedulers** → Build your own scheduling logic tailored to your workloads.  

All of these features were released in beta with Kubernetes 1.6, opening the door to smarter, more intentional scheduling strategies.

---
✨ Think of it this way: the default scheduler is like a reliable autopilot, but these advanced features let you take the controls when you need to fine-tune how your cluster runs.  

Here is the updated version of your blog post. I've added a brand-new section for **Custom Scheduler** as the new section 5, using a distinct naming convention (`custom-scheduler=scheduler-alpha` and `<custom-scheduler-pod>`), and integrated its corresponding commands into the final overview list.

---

### 1. Node Selector

**Node Selector** is a simple way to ensure that your pods run on specific nodes by assigning labels. This is useful when you have nodes with special hardware or software configurations and you want certain workloads to run only on those nodes.

* **Example Use Case:** Targeting a node designated specifically for the core engineering team's workloads.

```sh
# Label a node with a custom label
kubectl label node <node-name> project-owner=team-alpha

# Deploy a pod targeting the node with the specified label
kubectl apply -f <alpha-team-deployment>.yaml

# Verify that the pod is running on the correct node
kubectl get pods -o wide --no-headers | awk -F" " '{print $1, $8}'

```

### 2. Node Affinity

**Node Affinity** allows more complex scheduling decisions based on node labels. Unlike Node Selector, Node Affinity supports multiple label expressions and can differentiate between preferred and required conditions.

* **Example Use Case:** Ensuring that pods are deployed only on nodes assigned to specific pricing tiers or geographical zones.

```sh
# Label nodes with different tier labels
kubectl label node <node1-id> service-tier=gold
kubectl label node <node2-id> service-tier=silver
kubectl label node <node3-id> service-tier=bronze

# Deploy a workload with Node Affinity rules
kubectl apply -f <tiered-workload-spec>.yaml

# Scale the deployment to observe how pods are distributed
kubectl scale deployment <app-deployment-name> --replicas=8

```

### 3. Taints & Tolerations

**Taints** are applied to nodes to prevent pods that don't tolerate the taint from being scheduled on them. **Tolerations** are applied to pods to allow them to be scheduled on tainted nodes. This feature is essential for keeping certain workloads separate or ensuring critical workloads are not disrupted.

* Taints are applied on **Nodes Level** while Toleration are applied on **Pods Level**.

* **Example Use Case:** Reserving dedicated nodes for compliance-heavy or secure processing workloads, preventing standard applications from landing there.

```sh
# Taint nodes to control scheduling
kubectl taint node <node-id> secure-zone=restricted:NoSchedule
  

# Describe the node to see its taints
kubectl describe node <node-id> | grep -i secure

# Deploy a pod to see if it gets scheduled based on tolerations
kubectl apply -f <secure-pod-deployment>.yaml

# Remove taints from nodes if needed
kubectl taint node <node-id> secure-zone-
kubectl taint node <node-id> legacy-runtime-

```

### 4. Pod Affinity & Anti-Affinity

**Pod Affinity** allows you to schedule pods together on the same node, while **Pod Anti-Affinity** ensures that pods are placed on separate nodes. This is useful for reducing latency between pods or ensuring high availability by spreading pods across nodes.

* **Example Use Case:** Grouping API and caching services together to speed up responses, or spreading payment gateway replicas across different underlying nodes to maximize uptime.

```sh
# Use pod affinity or anti-affinity in your deployment YAML
kubectl apply -f <distributed-app-spec>.yaml

# Scale the deployment and observe the behavior
kubectl scale deployment <distributed-app-name> --replicas=2

# Drain a node to see how pods are rescheduled
kubectl drain <node-id>
kubectl uncordon <node-id>

```

### 5. Custom Scheduler

**Custom Schedulers** allow you to bypass the default Kubernetes scheduling logic entirely for specific workloads. If you have unique business requirements—like complex batch-job queuing or co-dependent multi-pod positioning—you can deploy your own scheduling engine alongside the default one and tell individual pods to use it.

* **Example Use Case:** Routing a data science workflow to a custom-compiled scheduler designed to optimize parallel job sequences.

```sh
# Deploy your custom scheduler into the cluster
kubectl apply -f <custom-scheduler-deployment>.yaml

# Check that your custom scheduler is running properly in the kube-system namespace
kubectl get pods -n kube-system | grep scheduler-alpha

# Deploy a pod that specifies 'schedulerName: scheduler-alpha' in its manifest
kubectl apply -f <custom-scheduled-pod>.yaml

# Verify which scheduler handled the pod assignment by checking events
kubectl describe pod <custom-scheduled-pod-name> | grep -i scheduled

```

## Commands Summary

Here’s a summary of the key commands used for the topics covered:

```sh
# Label a node
kubectl label node <node-name> project-owner=team-alpha

# Apply a deployment file
kubectl apply -f <alpha-team-deployment>.yaml

# Verify pod placement
kubectl get pods -o wide --no-headers | awk -F" " '{print $1, $8}'

# Label nodes for Node Affinity
kubectl label node <node1-id> service-tier=gold
kubectl label node <node2-id> service-tier=silver
kubectl label node <node3-id> service-tier=bronze

# Scale a deployment
kubectl scale deployment <app-deployment-name> --replicas=8

# Taint nodes
kubectl taint node <node-id> secure-zone=restricted:NoSchedule
kubectl taint node <node-id> legacy-runtime=true:NoExecute

# Remove taints from nodes
kubectl taint node <node-id> secure-zone-
kubectl taint node <node-id> legacy-runtime-

# Drain and uncordon a node
kubectl drain <node-id>
kubectl uncordon <node-id>

# Deploy and verify a Custom Scheduler
kubectl apply -f <custom-scheduler-deployment>.yaml
kubectl get pods -n kube-system | grep scheduler-alpha

# Deploy a custom-scheduled pod and inspect its scheduling events
kubectl apply -f <custom-scheduled-pod>.yaml
kubectl describe pod <custom-scheduled-pod-name> | grep -i scheduled

```

## Conclusion

This guide provides practical examples and commands to help us to understand and implement advanced scheduling techniques in Kubernetes. By mastering these concepts—from simple node selectors to completely bespoke custom schedulers—you can ensure that your applications run efficiently, securely, and reliably in any Kubernetes environment.
