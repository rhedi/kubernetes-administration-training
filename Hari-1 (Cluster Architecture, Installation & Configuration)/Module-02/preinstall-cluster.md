# Pre install cluster kubernetes on all node

### Install containerd on worker node
install package containerd
```
apt install containerd -y
```

### Check container runtime
ensure container runtime was installed on all server
```
systemctl status containerd
```

### Off partition swap on all server
disable swap partition temporary
```
cat /proc/swaps
swapoff -a
```

disable swap partition permanently
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```

remounting partition
```
mount -a
```
### Allow iptables
enable iptables to see bridge traffic
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

verify
```
lsmod | grep br_netfilter
lsmod | grep overlay
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

### Container runtime config
Load default configuration containerd
```
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

edit configuration containerd
```
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```

restart service
```
sudo systemctl restart containerd
```

If you want use dockerhub account to pull image, you can add thic config on config.toml. add on line 11
```
[plugins."io.containerd.grpc.v1.cri".registry.configs."registry-1.docker.io".auth]
  username = "username_dockerhub"
  password = "password/token"
```

### Testing 
```
apt install jq -y
```
```
TOKEN=$(curl --user 'user_dockerhub:token/password' "https://auth.docker.io/token?service=registry.docker.io&scope=repository:ratelimitpreview/test:pull" | jq -r .token)
curl --head -H "Authorization: Bearer $TOKEN" https://registry-1.docker.io/v2/ratelimitpreview/test/manifests/latest | grep -i rate
```
```
systemctl restart containerd
```

### Install kubeadm
Update the apt package index and install packages needed to use the Kubernetes apt repository
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

add apt repo & install kubelet, kubeadm, kubectl (version 1.32.0)

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update -y
sudo apt-get install -y kubelet=1.32.0-* kubeadm=1.32.0-* kubectl=1.32.0-*
sudo apt-mark hold kubelet kubeadm kubectl
```