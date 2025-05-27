# Module 1 : Cluster Architecture, Installation & Configuration

## Lab 1: Introduction
### Fresh Install
Login as root
```
sudo su
```

Change Hostname
```
hostnamectl set-hostname xxx
```

Set timezone
```
timedatectl set-timezone Asia/Jakarta
```

Change static ip
```
vim /etc/netplan/00-installer-config.yaml
```

change this
```
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens160:
      dhcp4: false
      addresses: [172.23.x.x/22]
      routes:
        - to: default
          via: 172.23.0.1
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
  version: 2
```

load network config
```
netplan apply
```

Verify hostname
```
hostname
```

Verify date
```
date
```

Verify ip address
```
ip a
```

Update and Upgrade Repository and Package
```
apt update -y && apt upgrade -y
```

### Docker
#### Installation
Install Docker
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

Check service docker
```
systemctl status docker
```

#### Containerize an application
Download the image to local
```
docker pull nginx 
```

Running the container on port 80
```
docker run -d -p 80:80 --name ct-name nginx
```

Check container status
```
docker ps
```

Stop container
```
docker stop ct-name
```

Delete container
```
docker rm -f ct-name
```

#### Update an application
Execution shell on container
```
docker exec -it ct-name /bin/bash
```

on container shell
```
cat cat /etc/nginx/conf.d/default.conf

echo "<p>Belajar docker</p>" > /usr/share/nginx/html/index.html
```

Create index file
```
echo "<p>Belajar docker copy</p>" > index.html
```

Copy custom content to container
```
docker cp index.html ct-name:destination-path 
```

#### Build your own application
Create directory
```
mkdir my-image
cd my-image
mkdir apps
```

create index file
```
vim apps/index.html
```

index.html
```
<title>Training Kubernetes</title>
<h1>Belajar Kubernetes Hari 1</h1>
```

Create Dockerfile
```
vim Dockerfile 
```

Dockerfile
```
FROM nginx
ADD ./apps /usr/share/nginx/html/
CMD nginx -g "daemon off;"
```

Build image
```
docker build -t image-name .
```

Check the image
```
docker images
```

Running a own image
```
docker run -d -p 8080:80 --name ct-name image-name
```

#### Share the application
Create account dockerhub, you can refers this link [Dockerhub](https://hub.docker.com/)

Login dockerhub account on docker
```
docker login 
```
Check info docker 
```
docker info 
``` 
Tag name image with repository name on dockerhub 
```
docker tag old-name-image repository/new-name-image
``` 
Push image to dockerhub 
```
docker push repository/new-name-image
``` 

## Lab 1: Setup Kubernetes Cluster
### Section 1: Pre-install cluster
#### Containerd
Install containerd 
```
apt install containerd
```

Check containerd package
```
apt list --installed | grep containerd
```

Check containerd service
```
systemctl status containerd
```

### Swap Partition
Check swap partition
```
cat /proc/swaps
```

Disable swap partition
```
swapoff -a
```

Deleted mount point swap partition on fstab
```
vim /etc/fstab
```

/etc/fstab
```
#/swap.img       none    swap    sw      0       0
```

Remounting partition
```
mount -a
```

#### Forwarding IPv4 and letting iptables see bridged traffic
Execute the below mentioned instructions
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

Verify that the br_netfilter, overlay modules are loaded by running the following commands
```
lsmod | grep br_netfilter
lsmod | grep overlay
```

Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command
```
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

#### Containerd Config
Load default configuration containerd
```
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Edit configuration containerd
```
vim /etc/containerd/config.toml
```

/etc/containerd/config.toml
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

restart service
```
systemctl restart containerd
```

### Section 2: Install kubeadm
Update the apt package index and install packages needed to use the Kubernetes apt repository
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories so you can disregard the version in the URL
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add the Kubernetes apt repository
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update apt package index, install kubelet, kubeadm and kubectl, and pin their version
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Check kubeadm was installed
```
apt list --installed | grep kube
```

Check kubeadm version
```
kubeadm version
```

### Section 3: Install cluster with kubeadm
#### Setup on master node
Download images core kubernetes services
```
kubeadm config images list
kubeadm config images pull
```

Initialize cluster on master node
```
kubeadm init
```

Create kubernetes config home dir 
```
mkdir .kube
```

Copy kubeconfig fle to kubernetes config home dir
```
cp /etc/kubernetes/admin.conf .kube/config
```

Check node on cluster
```
kubectl get node
```

#### Join cluster worker node
Join cluster on worker node
```
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

Generate new token for join cluster on MASTER
```
kubeadm token create --print-join-command
```

Check node on cluster
```
kubectl get node -o wide
```

Add label worker node
```
kubectl label node node-name node-role.kubernetes.io/worker=worker
```

### Section 4: Install Addons
#### CNI Plugins
Install Cilium CLI
```
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

Install Cilium
```
cilium install --version 1.17.1
```

Check cilium status
```
cilium status --wait
```

Cilium check config
```
kubectl edit cm -n kube-system cilium-config
```

#### Metric Server
Install latest version metric-server
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Edit deployment to enable metric server deployment to use certificate cluster
```
kubectl edit deployment.apps/metrics-server -n kube-system
```

deployment/metrics-server
```
spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
#Add    - --kubelet-insecure-tls
```

Check pod status
```
kubectl get pod -n kube-system
```

Check resource usage for pod & node
```
kubectl top node
kubectl top pod
```

### Section 5: Verify
Verify all node was Ready
```
kubectl get node
```
