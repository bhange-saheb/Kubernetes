# To Generate TLS Keys 

```bash
sudo certbot certonly --manual --preferred-challenges dns \
--key-type rsa \
--email zzzzccffff@gmail.com \
--agree-tos \
-d *.kkkdevops.xxy \
--server https://acme-v02.api.letsencrypt.org/directory

```

### Breakdown of your flags:

* **`certonly`**: Obtain the cert but don't try to install it into your web server automatically.
* **`--manual`**: You will perform the verification steps yourself.
* **`--preferred-challenges dns`**: You will prove ownership by adding a **TXT record** to your DNS settings.
* **`--key-type rsa`**: Generates a standard RSA key (default is usually 2048 or 4096 bits) instead of ECDSA.
* **`--server`**: Points to the Let's Encrypt V2 API.

---

### Important Steps After Running:

1. **The Prompt:** Certbot will stop and show you a **Digest Value**.
2. **The DNS Change:** You must log into your DNS provider (e.g., Cloudflare, GoDaddy, AWS) and create a record:
* **Type:** `TXT`
* **Host:** `_acme-challenge`
* **Value:** (The long string Certbot provides)


3. **The Wait:** **Do not press Enter immediately.** Wait 1 2 minutes for DNS to update.
4. **The Key:** Once it finishes, your TLS key will be at:
`/etc/letsencrypt/live/[yourdomain.com/privkey.pem](https://yourdomain.com/privkey.pem)`



Create Secret for TLS Certificate

```bash
kubectl create secret tls nginx-tls-default --key="tls.key" --cert="tls.crt"
```
To get see all secrets

```bash
kubectl get secrets
```

INGRESS CONTROLLER DEPLOYEMENT

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/aws/deploy.yaml
```

For Gate API

Install the Gateway API CRDs (they’re not part of Kubernetes core yet).

```bash 
kubectl apply -k github.com/kubernetes-sigs/gateway-api/config/crd 
```
Deploy a Gateway controller

```bash 
kubectl apply -f https://github.com/envoyproxy/gateway/releases/latest/download/install.yaml
```
For Traefik

Install Traefik via Helm or manifests:

```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
helm install traefik traefik/traefik
```

Giving access to vote app to read from docker hub private repo

```bash
kubectl create secret docker-registry docker-pwd --docker-username=ankit2507 --docker-password=dckr\_pat\_2qwqvOO-1Nh-5DTMmC2VJmb5yQU --docker-email=ankitbhange@gmail.com
```


