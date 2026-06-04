# Network Policies

**NetworkPolicies** give us control over traffic flow at the IP or port level (OSI layers 3 and 4).

* **What they do:** Define traffic rules within our cluster (Pod-to-Pod) and between Pods and external networks.
* **Prerequisite:** Our cluster must utilize a network plugin that supports NetworkPolicy enforcement.



**Kubernetes NetworkPolicies** allow you to control application traffic at the IP address or port level for **TCP, UDP, and SCTP** protocols. As an application-centric tool, NetworkPolicies define how a Pod communicates with various network "entities" (a broad term used here to avoid confusion with Kubernetes-specific "Endpoints" or "Services").

These policies specifically govern connections where a Pod is on at least one end; they do not impact other cluster traffic.
 
 ### How Entities are Identified
 A Pod's permitted network entities are defined using a combination of three identifiers:
* **Pods:** Specific Pods allowed to communicate (Note: A Pod can never block access to itself).
* **Namespaces:** Whole namespaces that are granted access.
* **IP Blocks:** Specific CIDR ranges (Note: Traffic to and from the Pod’s host Node is always allowed).

 ### How Policies are Defined
 
 * **Pod/Namespace-based:** Uses **selectors** to filter which traffic is allowed to or from matching Pods.
 * **IP-based:** Uses **CIDR ranges** to define strict IP block boundaries.

---

## Follow Below Step to achieve the Network policy task or play with networks.

 **STEP 1** Create Namespaces for resource isolation and more access control
```bash
kubectl create ns production
kubectl create ns development
kubectl create ns qualityAssurance
```
 **STEP 2** Apply Labels to Namespaces to restrict/control traffic between namespaces
```bash
kubectl label ns production env=prod
kubectl label ns development env=dev
kubectl label ns qualityAssurance env=qa
```
 **STEP 3** Deploy the pods in all namespaces and check the IPs
 
 Refer `deployment.yaml`.
 ```bash
 kubectl apply -f deployment.yaml

kubectl get pods -A -l 'env in (prod, dev, qa)' -o wide --no-headers
 ```
* All namespaces `-A`
* Label flag `-l`
**Look across the entire cluster for in all namespace (-A) for any pods (get pods) tagged as production, development, or QA (-l 'env in...')**