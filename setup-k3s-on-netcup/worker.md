## set hostname
```bash
sudo hostnamectl set-hostname k8s-manager
```

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
        - 192.168.100.2/24
```
```bash
sudo netplan generate
sudo netplan apply
```

## update /etc/hosts
```bash
echo "192.168.100.1 k8s-manager" >> /etc/hosts
echo "192.168.100.2 k8s-worker" >> /etc/hosts
```

## install k3s
```bash
export EXTERNAL_IP_MANAGER=""
export EXTERNAL_IP=""
export INTERNAL_IP=""
export INTERNAL_INTERFACE="eth1"
export TOKEN="<look at manager.md>"
curl -sfL https://get.k3s.io | K3S_URL=https://$EXTERNAL_IP_MANAGER:6443 K3S_TOKEN=$TOKEN INSTALL_K3S_EXEC="--flannel-iface=eth1 --node-ip=$INTERNAL_IP --node-external-ip=$EXTERNAL_IP" sh -
```

## ufw rules
```bash
sudo ufw allow ssh comment "SSH"
sudo ufw allow http comment "HTTP"
sudo ufw allow https comment "HTTPS"
sudo ufw allow from 192.168.100.0/24 comment "Kubernetes Flannel Network Communication (internal)"
sudo ufw allow in on eth1 from 192.168.100.0/24 comment "Kubernetes Flannel Network Communication (internal) on eth1 incoming"
sudo ufw allow out on eth1 from 192.168.100.0/24 "Kubernetes Flannel Network Communication (internal) on eth1 outgoing"
sudo ufw enable
```