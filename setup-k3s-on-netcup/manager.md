## create vlan
```bash
sudo vi /etc/netplan/02-vlan.yaml
```
```yaml
network:
   version: 2
   renderer: networkd
   ethernets:
      eth1:
        dhcp4: no
        addresses:
        - 192.168.100.1/24
```
```bash
sudo netplan generate
sudo netplan apply
```

## install k3s
```bash
export EXTERNAL_IP=""
export INTERNAL_IP=""
export INTERNAL_INTERFACE="eth1"
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--flannel-iface=$INTERNAL_INTERFACE --node-ip=$INTERNAL_IP --node-external-ip=$EXTERNAL_IP --tls-san $EXTERNAL_IP" sh -
```

## copy kubeconfig to the right place
```bash
sudo cp /etc/rancher/k3s/k3s.yaml .kube/config
```

## check node
```bash
kubectl get node -o wide
```

## get node token (set it as TOKEN environment variable on worker node)
```bash
cat /var/lib/rancher/k3s/server/node-token
```

## ufw rules
```bash
sudo ufw allow ssh comment "SSH"
sudo ufw allow http comment "HTTP"
sudo ufw allow https comment "HTTPS"
sudo ufw allow 6443/tcp comment "Kubernetes API"
sudo ufw allow from 192.168.100.0/24 comment "Kubernetes Flannel Network Communication (internal)"
sudo ufw allow in on eth1 from 192.168.100.0/24 comment "Kubernetes Flannel Network Communication (internal) on eth1 incoming"
sudo ufw allow out on eth1 from 192.168.100.0/24 "Kubernetes Flannel Network Communication (internal) on eth1 outgoing"
sudo ufw enable
```

## add cert manager
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.0/cert-manager.yaml
```

## add letsencrypt-staging
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: mail@example.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: staging-issuer-account-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: traefik
```
```bash
kubectl apply -f letsencrypt-staging.yaml
```

## add letsencrypt-prod
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: mail@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: prod-issuer-account-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: traefik
```
```bash
kubectl apply -f letsencrypt-prod.yaml
```
