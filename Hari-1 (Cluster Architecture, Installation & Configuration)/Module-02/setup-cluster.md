# Setup Cluster Kubernetes with kubeadm
### Setup cluster on master-node
initialize cluster on master node
```
kubeadm init --pod-network-cidr 10.100.0.0/16 --control-plane-endpoint=kube-master
```

### export kube config
```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf
```

### Setting Completion and Alias k
```
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc

```

### join cluster on worker-node
join worker node to clustser kubernetes
```
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

### generate new token for join cluster
if you lose a command to join worker, you can generate again a token for join worker node to cluster kubernetes
```
kubeadm token create --print-join-command
```

### check node on cluster
check a node in a cluster
```
kubectl get node
```

### Install CNI Plugin
install operator calico
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/custom-resources.yaml -O
kubectl create -f custom-resources.yaml
```

### Install Metric Server
Install latest version metric-server
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Edit deployment to enable metric server deployment to use certificate cluster
```
kubectl edit deployment.apps/metrics-server -n kube-system
```
Add ths
```
spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
#add    - --kubelet-insecure-tls
```

Check pod status
```
kubectl get pod -n kube-system
```

check resource usage for pod & node
```
kubectl top pod
kubectl top node
```

### Add label role worker node
add label worker
```
kubectl label node node-name node-role.kubernetes.io/worker=worker
```
