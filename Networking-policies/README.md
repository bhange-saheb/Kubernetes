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

## Deploying cluster with calico for private networking

```bash
kops create cluster --name=Ankitdevops.xyz \
--state=s3://Ankitdevops.xyz  --zones=us-east-1a,us-east-1b \
--node-count=3 --control-plane-count=1 --node-size=t3.medium --control-plane-size=t3.medium \
--control-plane-zones=us-east-1a --control-plane-volume-size 10 --node-volume-size 10 \
--topology private --networking calico \
--ssh-public-key ~/.ssh/id_ed25519.pub \
--dns-zone=Ankitdevops.xyz --yes
```

## Follow Below Step to achieve the Network policy task or play with networks.

 **STEP 1** Create Namespaces for resource isolation and more access control
```bash
kubectl create ns production
kubectl create ns development
kubectl create ns qualityassurance
```
 **STEP 2** Apply Labels to Namespaces to restrict/control traffic between namespaces
```bash
kubectl label ns production env=prod
kubectl label ns development env=dev
kubectl label ns qualityassurance env=qa
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

 **STEP 4** Check Connection between all pods from all namespace

`production pods to development and qualityassurance pods`

 ```bash
kubectl exec -it prod-tools-56d9b959f5-d5d8x -n production -- ping -c 3 100.116.152.197 \
&& kubectl exec -it prod-tools-56d9b959f5-d6mgh -n production -- ping -c 3 100.127.239.197 \
&& kubectl exec -it prod-tools-56d9b959f5-d5d8x -n production -- ping -c 3 100.109.134.3 \
&& kubectl exec -it prod-tools-56d9b959f5-d6mgh -n production -- ping -c 3 100.116.152.198
```

`development pods to production and qualityassurance pods`

```bash
kubectl exec -it dev-tools-58b98fcd4c-jrzsb -n development -- ping -c 3 100.116.152.196 \
&& kubectl exec -it dev-tools-58b98fcd4c-nfm2s -n development -- ping -c 3 100.127.239.196 \
&& kubectl exec -it dev-tools-58b98fcd4c-jrzsb -n development -- ping -c 3 100.109.134.3 \
&& kubectl exec -it dev-tools-58b98fcd4c-nfm2s -n development -- ping -c 3 100.116.152.198 
```
`qualityassurance pods to production and development pods`

```bash
kubectl exec -it qa-tools-7fdcc59f58-l5j5l -n qualityassurance -- ping -c 3 100.116.152.196 \
&& kubectl exec -it qa-tools-7fdcc59f58-ldthr -n qualityassurance -- ping -c 3 100.127.239.196 \
&& kubectl exec -it qa-tools-7fdcc59f58-l5j5l -n qualityassurance -- ping -c 3 100.116.152.197 \
&& kubectl exec -it qa-tools-7fdcc59f58-ldthr -n qualityassurance -- ping -c 3 100.127.239.197
```