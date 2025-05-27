# Cluster Maintenance

## Cluster Management
check cluster
```
kubectl config view
kubectl config get-contexts
```

config cluster name
```
kubectl edit configmaps kubeadm-config -n kube-system
```

edit this
```
# clusterName: cluster-slipi
```

config cluster name
```
vim /etc/kubernetes/kubelet.conf
```

edit this
```
# name: cluster-slipi
# cluster: cluster-slipi
```

config kubernetes credentials
```
vim /etc/kubernetes/admin.conf
```

edit this
```
# name: cluster-slipi
# cluster: cluster-slipi
# cluster: cluster-slipi
```

check cluster
```
kubectl config view
kubectl config get-contexts
```

## User Management
### Create certificate user account
Generate key
```
openssl genrsa -out client.key
```

Generate csr
```
openssl req -new -key client.key -out client.csr -subj "/CN=client/O=dev"
```

Generate crt 
```
openssl x509 -req -in client.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out client.crt -days 3650
```

### Create context kubeconfig user account
Generate context kubeconfig
```
kubectl config set-credentials client --client-certificate=client.crt --client-key=client.key
kubectl config set-context client-kubernetes --cluster=kubernetes --namespace=dev --user=client
kubectl config get-contexts
```

Manage cluster with custom kubeconfig
```
kubectl config use-context client-kubernetes
```

### Create Role
Create role
```
kubectl create role dev-role --verb=get,list --resource=pods --namespace dev
```
Edit permission role
```
kubectl edit role dev-role -n dev
```

### Create role binding
Create cluster role binding
```
kubectl create rolebinding dev-rolebinding --role=dev-role --user=client --namespace dev
```
Edit cluster role binding
```
kubectl edit rolebinding dev-rolebinding -n dev
```
### Manage cluster with new account
Manage cluster with custom kubeconfig file
```
kubectl config use-context client-kubernetes
```

## Dashboard
### Install dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

### expose dashboard
```
kubectl get ns
kubectl get all -n kubernetes-dashboard
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
```

### Create service account
```
kubectl create sa admin-user --namespace kubernetes-dashboard
```
### Create ClusterRoleBinding
```
vim admin-access.yml
```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-access
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

```
kubectl apply -f admin-access.yml
```

### Getting a Bearer Token
```
kubectl -n kubernetes-dashboard create token admin-user
```

### Install weave scope
```
kubectl apply -f https://github.com/weaveworks/scope/releases/download/v1.13.2/k8s-scope.yaml
```

### expose weave scope
```
kubectl get ns
kubectl get all -n weave
kubectl edit svc weave-scope-app -n weave
```

## Node Management
```
kubectl get node
kubectl describe node node-name
```

```
kubectl cordon node node-name
```

```
kubectl drain node node-name
```

```
kubectl uncordon node node-name
```

## Backup & Restore 
Define etcd variable
```
export ETCDCTL_API=3
export ENDPOINT=127.0.0.1:2379
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
```

Manage etcd cluster
```
etcdctl --endpoints $ENDPOINT -w table endpoint status
etcdctl --endpoints $ENDPOINT -w table endpoint health
etcdctl --endpoints $ENDPOINT -w table member list
```

Manage etcd kubernetes
```
etcdctl --endpoints $ENDPOINT get /registry/ --prefix --keys-only
etcdctl --endpoints $ENDPOINT get /registry/ --prefix --keys-only | grep pods/default
etcdctl --endpoints $ENDPOINT get /registry/pods/dev/pod-dev
```

Backup etcd
```
etcdctl --endpoints $ENDPOINT snapshot save backup-cluster-1.26
etcdctl --endpoints $ENDPOINT snapshot status backup-cluster-1.26
```

Restore etcd
```
etcdctl --endpoints $ENDPOINT --name master-rafi --data-dir /var/lib/etcd-cluster-restore --initial-cluster=master-rafi=https://127.0.0.1:2380 --initial-cluster-token etcd-cluster-restore --initial-advertise-peer-urls https://127.0.0.1:2380 snapshot restore backup-cluster-1.26
```

edit manifest
```
vim /etc/kubernetes/manifests/etcd.yaml
```
## Cluster Upgrade
### Upgrading kubeadm clusters
Upgrading a kubeadm cluster from 1.28.2 to 1.28.15

The upgrade workflow at high levels is the following:
1. upgrade a primary control plane node.
2. upgrade additional control plane nodes.
3. upgrade worker node

### Determine which version to upgrade to 
Find the latest patch release for kubernetes 1.28 using the OS package manager
```
apt update
apt-cache madison kubeadm
```

### Upgrade control plane nodes
Upgrade kubeadm
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.28.15-*' && \
sudo apt-mark hold kubeadm
```
Verify that the download works and has the expected version
```
kubeadm version
```
Verify the upgrade plan
```
kubeadm upgrade plan
```
pull image for upgrade cluster
```
kubeadm config images pull
```
Choose a version to upgrade to, and run the appropriate command. For example
```
sudo kubeadm upgrade apply v1.28.15
```
Verify that the download works and has the expected version
```
kubeadm version
```
Drain the node
```
kubectl drain kube-master --ignore-daemonsets
```
Upgrade kubelet and kubectl
```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.28.15-*' kubectl='1.28.15-*' && \
sudo apt-mark hold kubelet kubectl
```
Restart the kubelet
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
Uncordon the node
```
kubectl uncordon kube-master
```

### Upgrade worker nodes
Install kubeadm
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.28.15-*' && \
sudo apt-mark hold kubeadm
```
Upgrade kubeadm
```
sudo kubeadm upgrade node
```
Drain node on **master**
```
kubectl drain kube-worker-01 --ignore-daemonsets
```
Upgrade kubelet and kubectl 
```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.28.15-*' kubectl='1.28.15-*' && \
sudo apt-mark hold kubelet kubectl
```
Restart the kubelet
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
Bring the node back online by marking it schedulable
```
kubectl uncordon worker
```
