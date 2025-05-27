# Module 4 : Troubleshooting and Maintenance Cluster

## Logging
### Section 1: Logging Application
Using kubectl for logging
```
kubectl logs -f pod-name
kubectl logs -f -l key=value
kubectl logs -f deployment-name/pod-name
```

Create sample pod
```
vim pod-log.yaml
```

pod-log.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-log
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
```

Deploy pod
```
kubectl apply -f pod-log.yaml
```

Check log pod
```
kubectl logs -f pod-log
kubectl logs pod-log > pod-log.log
```

### Section 2: Logging Cluster
Check log kube-api-server
```
tail -f /var/log/containers/kube-apiserver-xxx/xxx.log
```

Check log etcd
```
tail -f /var/log/containers/etcd-xxx/xxx.log
```

Check log kubelet
```
journalctl -fu kubelet
```

## Troubleshooting
### Section 1: Troubleshooting Application
Check event on cluster
```
kubectl events
```

Check event on namespace
```
kubectl get events -n default
```

Check event on deployment
```
kubectl describe deployment <deployment-name>
```

Check event on pod
```
kubectl describe pod <pod-name>
```

### Section 2: Troubleshooting Cluster
Check node
```
kubectl get node
```

Check detail info node
```
kubectl describe node <node-name>
```

ssh console to worker2 and then stop service kubelet
```
systemctl stop kubelet
```

Check node status on worker2
```
kubectl get node
kubectl describe node <node-name>
```

ssh console to worker2 and then start service kubelet
```
systemctl start kubelet
```

## Maintenance
### Cluster Management
#### Section 1: Maintenance Node
Check node
```
kubectl get node
kubectl describe node k8s-worker1
```

Cordon node
```
kubectl cordon k8s-worker1
```

Drain node 
```
kubectl drain k8s-worker1
```

Uncordon node
```
kubectl uncordon node k8s-worker1
```

#### Section 2: Unregister/Register Node to Cluster
Check node
```
kubectl get node
```

Unregister node
```
ssh -l <user> <k8s-worker1-ip>

kubeadm reset
```

Delete node from cluster
```
kubectl delete node <node>
```

Register node
```
ssh -l <user> <k8s-worker1-ip>

kubeadm join
```

## Manage RBAC
### Section 1: User Account
Check user account
```
kubectl config get-users
```

Check on kubeconfig
```
cat /etc/kubernetes/admin.conf
```

### Section 2: Service Account
Check service account
```
kubectl get sa
```

Check on pod
```
kubectl get pod -o yaml | grep serviceAccount
```

### Section 3: Role and ClusterRole 
Check Role
```
kubectl get role
```

Check ClusterRole
```
kubectl get clusterrole
```

Check config clusterrole cluster-admin
```
kubectl edit clusterrole cluster-admin
```

### Section 4: RoleBinding and ClusterRoleBinding
Check Role
```
kubectl get rolebinding
```

Check ClusterRole
```
kubectl get clusterrolebinding
```

Check config clusterrolebinding cluster-admin
```
kubectl edit clusterrolebinding cluster-admin
```

### Section 5: Lab Scenario User Account
Generate PKI Private Key and CSR
```
openssl genrsa -out dev.key 2048
openssl req -new -key dev.key -out dev.csr -subj "/CN=dev/O=client"
```

Create a CertificateSigningRequest
```
cat dev.csr | base64 | tr -d "\n"

cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: dev
spec:
  groups:
  - system:authenticated
  request: $(cat dev.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

Approve the CertificateSigningRequest
```
kubectl get csr
kubectl certificate approve dev
```

Get the certificate
```
kubectl get csr/dev -o yaml
kubectl get csr dev -o jsonpath='{.status.certificate}'| base64 -d > dev.crt
```

Check User Account
```
kubectl config get-users 
```

Create User Account
```
kubectl config set-credentials dev --client-key=dev.key --client-certificate=dev.crt --embed-certs=true
```

Check context
```
kubectl config get-contexts
```

Create context
```
kubectl config set-context dev@kubernetes --cluster=kubernetes --namespace=default --user=dev
```

Check permission
```
kubectl auth can-i list pod --as=dev
kubectl auth can-i create pod --as=dev
kubectl auth can-i delete pod --as=dev
```

Switching context
```
kubectl config use-context dev@kubernetes
kubectl config use-context kubernetes-admin@kubernetes
```

Create role developer
```
kubectl create role developer --verb=get,list --resource=pods --namespace default
```

Binding role developer to user dev
```
kubectl create rolebinding dev-binding --role=developer --user=dev --namespace default
```

### Section 6: Lab Scenario Service Account
Create pod without service account
```
vim pod-non-sa.yml
```

pod-non-sa.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-non-sa
spec:
  containers:
  - image: nginx
    name: ct-nginx
```

Test privileged pod-non-sa
```
kubectl exec -it pod-non-sa -- bash

ls /var/run/secrets/kubernetes.io/serviceaccount/
cat /var/run/secrets/kubernetes.io/serviceaccount/token

APISERVER=https://kubernetes.default.svc
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
TOKEN=$(cat ${SERVICEACCOUNT}/token)
CACERT=${SERVICEACCOUNT}/ca.crt

curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/default/pods
```

Check service account
```
kubectl get sa
```

Create service account
```
kubectl create sa pod-account
```

Binding role to service account
```
kubectl create rolebinding dev-sa-binding --role=developer --serviceaccount=default:pod-account --namespace default
```

Create pod use Service Account
```
vim pod-sa.yml
```

pod-sa.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-sa
spec:
  serviceAccount: pod-account
  containers:
  - image: nginx
    name: ct-nginx
```

Test privileged pod-sa
```
kubectl exec -it pod-sa -- bash

ls /var/run/secrets/kubernetes.io/serviceaccount/
cat /var/run/secrets/kubernetes.io/serviceaccount/token

APISERVER=https://kubernetes.default.svc
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
TOKEN=$(cat ${SERVICEACCOUNT}/token)
CACERT=${SERVICEACCOUNT}/ca.crt

curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/default/pods
```

## Lab 4: ETCD Backup and Restore
### Section 1: Check ETCD environtment
Check etcd pod
```
kubectl get all -n kube-system
```

Check etcd configuration
```
vim /etc/kubernetes/manifests/etcd.yaml
```

Check etcd tls cert and key
```
# etcd cert
ls /etc/kubernetes/pki/etcd/server.crt

# etcd key
ls /etc/kubernetes/pki/etcd/server.key

# etcd ca cert
ls /etc/kubernetes/pki/etcd/ca.crt
```

Check etcd data directory
```
ls /var/lib/etcd/member/snap/
ls /var/lib/etcd/member/wal/
```

Check etcd status
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
-w table endpoint status
```

Check etcd health
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
-w table endpoint health
```

Check etcd member
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
-w table member list
```

Check etcd keys
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
get /registry/ --prefix --keys-only
```

Check etcd keys for pod on namespace default
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
get /registry/ --prefix --keys-only | grep pods/default
```

Check etcd value on key pod-sa
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
get /registry/pods/default/pod-sa
```

### Section 2: ETCD Backup Scenario
Create simple pod test
```
kubectl run pod-backup --image nginx
```

Backup etcd
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save backup-etcd
```

Check backup etcd
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot status backup-etcd
```

### Section 3: ETCD Restore
Delete pod on namespace default
```
kubectl delete pod -n default --all
```

Restore etcd
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--data-dir="/var/lib/etcd-backup" \
snapshot restore backup-etcd
```

Edit config etcd
```
vim /etc/kubernetes/manifests/etcd.yaml
```

etcd.yaml
```
spec:
  containers:
  - command:
    - etcd
    ---
    - --data-dir=/var/lib/etcd-backup
    volumeMounts:
    - mountPath: /var/lib/etcd-backup
      name: etcd-data
  - hostPath:
      path: /var/lib/etcd-backup
      type: DirectoryOrCreate
    name: etcd-data
```

Restart kubelet
```
systemctl restart kubelet
```

## Lab 5: Upgrade Cluster
Upgrading a kubeadm cluster from 1.31 to 1.32<br>
The upgrade workflow at high levels is the following:
1. upgrade a primary control plane node.
2. upgrade additional control plane nodes.
3. upgrade worker node

### Section 1: Upgrade on master node
Check package and repository
```
cat /etc/apt/sources.list.d/kubernetes.list

apt-cache madison kubeadm
```

Upgrade repository
```
vim /etc/apt/sources.list.d/kubernetes.list
```

kubernetes.list
```
#deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
```

Update repo and check package
```
apt update
apt-cache madison kubeadm
```

Upgrade kubeadm
```
# replace x in 1.32.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.32.2-*' && \
sudo apt-mark hold kubeadm
```

Check kubeadm version
```
kubeadm version
```

Verify the upgrade plan
```
kubeadm upgrade plan
```

Pull image for upgrade cluster
```
kubeadm config images pull
```

Choose a version to upgrade to, and run the appropriate command. For example
```
kubeadm upgrade node
```

Drain the node
```
kubectl drain k8s-master --ignore-daemonsets

# Force drain
kubectl drain k8s-master --ignore-daemonsets --force --grace-period=0
```

Upgrade the kubelet and kubectl
```
# replace x in 1.32.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.2-*' kubectl='1.32.2-*' && \
sudo apt-mark hold kubelet kubectl
```

Restart the kubelet
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Uncordon node
```
kubectl uncordon k8s-master
```

### Section 2: Upgrade on worker node
Check package and repository
```
cat /etc/apt/sources.list.d/kubernetes.list

apt-cache madison kubeadm
```

Upgrade repository
```
vim /etc/apt/sources.list.d/kubernetes.list
```

kubernetes.list
```
#deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
```

Update repo and check package
```
apt update
apt-cache madison kubeadm
```

Upgrade kubeadm
```
# replace x in 1.30.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.32.2-*' && \
sudo apt-mark hold kubeadm
```

Check kubeadm version
```
kubeadm version
```

For worker nodes this upgrades the local kubelet configuration
```
kubeadm upgrade node
```

On master node: Prepare the node for maintenance by marking it unschedulable and evicting the workloads
```
kubectl drain k8s-worker --ignore-daemonsets

# Force drain
kubectl drain k8s-worker --ignore-daemonsets --force --grace-period=0
```

Upgrade the kubelet and kubectl
```
# replace x in 1.32.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.32.2-*' kubectl='1.32.2-*' && \
sudo apt-mark hold kubelet kubectl
```

Restart the kubelet
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Uncordon node
```
kubectl uncordon k8s-worker
```
