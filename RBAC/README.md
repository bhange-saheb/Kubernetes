STEP 1:

Install Kubectx and kubens for contex and Namespaces

cd /usr/local/bin 

Kubectx

```bash
wget https://github.com/ahmetb/kubectx/releases/download/v0.11.0/kubectx_v0.11.0_linux_x86_64.tar.gz
tar zxfv kubectx_v0.11.0_linux_x86_64.tar.gz
```

Kubenx

```bash
wget https://github.com/ahmetb/kubectx/releases/download/v0.11.0/kubens_v0.11.0_linux_x86_64.tar.gz
tar zxfv kubens_v0.11.0_linux_x86_64.tar.gz
```

check version

```bash
kubectx --version
kubens --version
```

Remove uneccesar files

```bash
rm -rf *.gz
```

STEP2 Create NS  development and Production.

STEP3
Copy cluster authecator certificate ```( .crt and .key )``` from master node to our management server

Files will be available at below location

``` /etc/kubernetes/kops-controller ```


Create Users

user1

openssl genrsa -out user1.key 2048
openssl req -new -key user1.key -out user1.csr -subj "/CN=user1/O=development"
openssl x509 -req -in user1.csr -CA ca.crt -CAkey ca.key  -CAcreateserial -out user1.crt -days 365


user2 
openssl genrsa -out user2.key 2048
openssl req -new -key user2.key -out user2.csr -subj "/CN=user2/O=production"
openssl x509 -req -in user2.csr -CA ca.crt -CAkey ca.key  -CAcreateserial -out user2.crt -days 365

Admin-user Note Admin user didn't require namespace
openssl genrsa -out komal.key 2048
openssl req -new -key komal.key -out komal.csr -subj "/CN=komal/O=clusteradmin"
openssl x509 -req -in komal.csr -CA ca.crt -CAkey ca.key  -CAcreateserial -out komal.crt -days 365


# Here we are sigining the key with Certificate Authority 

Step4 Now Copy keys from mangement server to master node and paste it in root folder it is needed for config file

Step5 Now we have to setup config file for created user in master node
Refer "kubeConfig.yaml"

blueprint can be copied from ~/.kube/ in master node


create CONFIG files without extensions copy data from above mentioned file and paste it

#User1
```bash
nano USER1-CONFIG
```

#User2
```bash
nano USER2-CONFIG
```
#User3
```bash
nano KOMAL-CONFIG
```

STEP6 
export KUBECONFIG=/root/USER1-CONFIG

Here we are setting the KUBECONFIG environment variable to point to a different file (/root/USER1-CONFIG). 
From that moment on, every kubectl command in that shell session will use the configuration inside that file instead of the default one.

STEP6  Creating and Assigning of roles to users in management server

Refer "roles.yaml" file

STEP7 Creat rolebing file to bind roles with each user.

Now we have achieved role based acces control in k8s cluster.

we can merge all config file inti single file by 

```bash
KUBECONFIG=USER1-CONFIG:USER2-CONFIG:SAIKIRAN-CONFIG kubectl config view --merge --flatten > mixed-config.txt
```
