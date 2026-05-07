# Kubernetes ![Test Image 4](https://i.imgur.com/0HhNox3.png)

Kubernetes is an open source container orchestration engine for automating deployment, scaling, and management of containerized applications. The open source project is hosted by the Cloud Native Computing Foundation (CNCF).


## What is kOps?¶
We like to think of it as kubectl for clusters.

kops will not only help you create, destroy, upgrade and maintain production-grade, highly available, Kubernetes cluster, but it will also provision the necessary cloud infrastructure.

To install Kubernetes Operations kOps

```bash
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops
sudo mv kops /usr/local/bin/kops
```

#kubectl
The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. 
You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.


To install kubectl

```bash
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops
sudo mv kops /usr/local/bin/kops
```

Configure .bashrc for k8s cluster

export NAME=Ankitdevops.xyz 

export KOPS_STATE_STORE=s3://Ankitdevops.xyz

export AWS_REGION=us-east-1

export CLUSTER_NAME=Ankit.xyz

export EDITOR='/usr/bin/nano'

alias ko='kubectl'