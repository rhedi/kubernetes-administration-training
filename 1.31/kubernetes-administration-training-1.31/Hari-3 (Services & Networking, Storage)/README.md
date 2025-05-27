# Module 3 : Services and Networking

## Service
### Section 1: Create Deployment
Create deployment simple apps
```
vim deployment.yaml
```

deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

Deploy deployment
```
kubectl apply -f deployment.yaml
```

Check all resource with label app=nginx
```
kubectl get all -l app=nginx -o wide
```

Access to pod
```
curl <pod ip address>
```

### Section 2: Service type ClusterIP
Create Service nginx
```
vim service.yaml
```

service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Deploy Service
```
kubectl apply -f service.yaml
```

Check Service
```
kubectl get svc
```

Check list endpoints
```
kubectl get endpoints
```

Access to service
```
curl <service-ip>
```

### Section 3: Service type NodePort
Update Service nginx
```
vim service.yaml
```

service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30000
```

Deploy Service
```
kubectl apply -f service.yaml
```

Check Service
```
kubectl get svc
```

Access service nodeport from local browser
```
http://<node-ip>:<node-port>
```

### Section 4: Service type LoadBalancer
Update Service nginx
```
vim service.yaml
```

service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30000
```

Deploy Service
```
kubectl apply -f service.yaml
```

Check Service
```
kubectl get svc
```

Access service loadbalancer from local browser
```
http://<service-loadbalancer-ip>
```

## Ingress
### Section 1: Install Ingress Controller
Installation NGINX ingress-controller with manifest
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/baremetal/deploy.yaml
```

Check ingress-controller resource
```
kubectl get all -n ingress-nginx
```

Check ingressclass resource
```
kubectl get ingressclass
```

### Section 2: Deploy Ingress without SSL
Create ingress rule
```
vim ingress.yaml
```

ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  labels:
    app: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: test.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

Deploy ingress
```
kubectl apply -f ingress.yaml
```

Check ingress
```
kubectl get ing
```

Access host domain ingress
```
# access to cluster ip ingress controller
curl http://test.com --resolve 'test.com:80:<cluster-ip>'

# access to node port ingress controller
curl http://test.com:<node-port> --resolve 'test.com:<node-port>:<node-ip>'
```

### Section 3: Deploy Ingress with SSL
Create self signed certificate
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout test.key -out test.crt
```

Create secret ssl from certificate
```
kubectl create secret tls test-secret --key test.key --cert test.crt 
```

Update ingress rule
```
vim ingress.yaml
```

ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  labels:
    app: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - test.com
    secretName: test-secret
  rules:
  - host: test.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

Deploy ingress
```
kubectl apply -f ingress.yaml
```

Check ingress
```
kubectl get ing
```

Access host domain ingress
```
# access to cluster ip ingress controller
curl -k https://test.com --resolve 'test.com:443:<cluster-ip>'

# access to node port ingress controller
curl -k https://test.com:<node-port> --resolve 'test.com:<node-port>:<node-ip>'
```

## MetalLB
### Section 1: Installation MetalLB
Config kube-proxy
```
kubectl edit configmap -n kube-system kube-proxy
```

kube-proxy
```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

Installation MetalLB by manifest
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
```

Check MetalLB resource
```
kubectl get all -n metallb-system
```

### Section 2: Configuration MetalLB
Create IPAddressPool
```
vim ip-pool-172-23.yaml
```

ip-pool-172-23.yaml
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ip-pool-172-23
  namespace: metallb-system
spec:
  addresses:
  - 172.23.x.x-172.23.x.x
```

Create L2advertisement
```
vim l2-advertise-ens160.yaml
```

l2-advertise-ens160.yaml
```
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advertise-ens160
  namespace: metallb-system
spec:
  ipAddressPools:
  - ip-pool-172-23
  interfaces:
  - ens160
```

Deploy Configuration
```
kubectl apply -f ip-pool-172-23.yaml
kubectl apply -f l2-advertise-ens160.yaml
```

### Section 3: Use Service Loadbalancer
Check Service using type LoadBalancer
```
kubectl get svc
```

### Check network traffic
Check network traffic ARP Loadbalancer
```
tcpdump -vnni ens160 arp host <service-ip-loadbalancer>
```

## CoreDNS
### Section 1: CoreDNS Configuration
Check kube-dns was running
```
kubectl get all -n kube-system -l k8s-app=kube-dns
```

Check config kube-dns
```
# check list configmaps on namespace kube-syste
kubectl get cm -n kube-system

# check detailts configmaps coredns
kubectl get cm -n kube-system coredns -o yaml
```

Create pod to check dns config
```
kubectl run -it --rm --restart=Never dns-utils --image=busybox -- /bin/sh
```

Test dns for service
```
nslookup nginx.default.svc.cluster.local
```

Test dns for pod
```
nslookup 10-100-194-90.default.pod.cluster.local
```

### Section 2: Pod config to use external DNS
Create pod using configuration external dns
```
vim pod-ext-dns.yaml
```

pod-ext-dns.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-ext-dns
spec:
  containers:
    - name: ct-nginx
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 8.8.8.8
    searches:
      - google.com
```

Deploy pod
```
kubectl apply -f pod-ext-dns.yaml
```

Check config dns on pod
```
kubectl exec -it pod-ext-dns -- bash
```

### Section 3: Pod config to define hosts configuration
Create pod using configuration localdomain /etc/hosts
```
vim pod-local-hosts.yaml
```

pod-local-hosts.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-local-hosts
spec:
  containers:
    - name: ct-nginx
      image: nginx
  hostAliases:
  - ip: "10.10.10.1"
    hostnames:
    - "domain.localdomain"
```

Deploy pod
```
kubectl apply -f pod-local-hosts.yaml
```

Check config hosts on pod
```
kubectl exec -it pod-local-hosts -- bash
```

## Network Policies
### Section 1: Create Pod and Service
Create pod and service frontend
```
kubectl run frontend --image nginx
kubectl expose pod frontend --port 80
```

Create pod and service backend
```
kubectl run backend --image nginx
kubectl expose pod backend --port 80
```

Create namespace database
```
kubectl create ns database
```

Create pod and service database
```
kubectl -n database run db --image nginx
kubectl -n database expose pod db --port 80
```

Testing connection frontend <=> backend
```
kubectl exec -it frontend -- curl <ip service backend>
kubectl exec -it backend -- curl <ip service frontend>
```

Testing connection backend <=> db
```
kubectl exec -it backend -- curl <ip service db>
kubectl -n database exec -it db -- curl <ip service backend>
```

Testing connection frontend <=> db
```
kubectl exec -it frontend -- curl <ip service db>
kubectl -n database exec -it db -- curl <ip service frontend>
```

### Section 2: Create deny policy on inbound and outbound network policy
Create deny policy to all pod on namespace default
```
vim default-policy.yml
```

default-policy.yml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-policy
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
```

Create deny policy to all pod on namespace database
```
vim database-policy.yml
```

database-policy.yml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: database
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
```

Testing connection frontend <=> backend
```
kubectl exec -it frontend -- curl <ip service backend>
kubectl exec -it backend -- curl <ip service frontend>
```

Testing connection backend <=> db
```
kubectl exec -it backend -- curl <ip service db>
kubectl -n database exec -it db -- curl <ip service backend>
```

Testing connection frontend <=> db
```
kubectl exec -it frontend -- curl <ip service db>
kubectl -n database exec -it db -- curl <ip service frontend>
```

### Section 3: Create specific rule on network policy
Create network policy to allow frontend outbound access to backend
```
vim frontend-policy.yaml
```

frontend-policy.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          run: backend
```

Create network policy to allow backend inbound access from frontend
```
vim backend-policy.yaml
```

backend-policy.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: frontend
```

Create network policy to allow backend outbound access to database
```
vim backend-policy.yaml
```

backend-policy.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
  - Ingress
  # add this
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: frontend
  # add this
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            ns: database
      - podSelector:                                                
          matchLabels:                                              
            run: db
```

Create network policy to allow database inbound access from backend
```
vim db-policy.yaml
```

db-policy.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
  namespace: database
spec:
  podSelector:
    matchLabels:
      run: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          ns: default
    - podSelector:
        matchLabels:
          run: backend
```

Testing connection frontend <=> backend
```
kubectl exec -it frontend -- curl <ip service backend>
kubectl exec -it backend -- curl <ip service frontend>
```

Testing connection backend <=> db
```
kubectl exec -it backend -- curl <ip service db>
kubectl -n database exec -it db -- curl <ip service backend>
```

Testing connection frontend <=> db
```
kubectl exec -it frontend -- curl <ip service db>
kubectl -n database exec -it db -- curl <ip service frontend>
```


# Module 4: Storage

## Connect NFS Server
Install nfs support client
```
apt install nfs-common
```

Check mount information
```
showmount -e <nfs.server.ip>
```

Mount nfs server
```
mount -t nfs <nfs.server.ip>:/var/nfs /mnt
```

Check mount
```
df -h /mnt
```

Test
```
# access folder /mnt
cd /mnt

# create new folder
mkdir folder

# create file on folder
touch folder/file{1..3}.txt

# check list file on folder
ls -l folder
```

## Volume
### Section 1: Volume type hostPath
Create deployment
```
vim deployment-vol-hostpath.yaml
```

deployment-vol-hostpath.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-vol-hostpath
  labels:
    app: deployment-vol-hostpath
spec:
  selector:
    matchLabels:
      app: deployment-vol-hostpath
  template:
    metadata:
      labels:
        app: deployment-vol-hostpath
    spec:
      volumes:
        - name: volume-hostpath
          hostPath:
            path: /var/data
            type: Directory
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: volume-hostpath
          mountPath: /usr/share/nginx/html
```

Deploy deployment
```
kubectl apply -f deployment-vol-hostpath.yaml
```

Check pod
```
kubectl get pod -l app=deployment-vol-hostpath
```

Access to pod
```
kubectl exec-it deployment-vol-hostpath-xxx -- ls /usr/share/nginx/html
```

Create file on pod
```
kubectl exec-it deployment-vol-hostpath-xxx -- bash

echo "<p>Test write</p>" > /usr/share/nginx/html/index.html
```

Check file on /var/data on worker
```
ls /var/data
```

### Section 2: Testing hostPath
Test Scaling pod
```
kubectl scale deployment deployment-vol-hostpath --replicas 2
```

Check pod
```
kubectl get pod -l app=deployment-vol-hostpath
```

Check file on pod
```
kubectl exec-it deployment-vol-hostpath-xxx -- ls /usr/share/nginx/html
```

Check file on /var/data on worker
```
ls /var/data
```

### Section 1: Volume type NFS
Create deployment
```
vim deployment-vol-nfs.yaml
```

deployment-vol-nfs.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-vol-nfs
  labels:
    app: deployment-vol-nfs
spec:
  selector:
    matchLabels:
      app: deployment-vol-nfs
  template:
    metadata:
      labels:
        app: deployment-vol-nfs
    spec:
      volumes:
        - name: volume-nfs
          nfs:
            server: <nfs.server.ip>
            path: /var/nfs
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: volume-nfs
          mountPath: /usr/share/nginx/html
```

Deploy deployment
```
kubectl apply -f deployment-vol-nfs.yaml
```

Check pod
```
kubectl get pod -l app=deployment-vol-nfs
```

Access to pod
```
kubectl exec-it deployment-vol-nfs-xxx -- ls /usr/share/nginx/html
```

Create file on pod
```
kubectl exec-it deployment-vol-nfs-xxx -- bash

echo "<p>Test write</p>" > /usr/share/nginx/html/index.html
```

Check file on /mnt on master
```
ls /mnt
```

### Section 2: Testing NFS
Test Scaling pod
```
kubectl scale deployment deployment-vol-nfs --replicas 2
```

Check pod
```
kubectl get pod -l app=deployment-vol-nfs
```

Access file on pod
```
kubectl exec-it deployment-vol-nfs-xxx -- ls /usr/share/nginx/html
```

## Persistent Volume
### Section 1: Create Persistent Volume
Create Persistent Volume
```
vim pv-a.yaml
```

pv-a.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-a
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: pv-a
  nfs:
    path: /var/nfs/pv-a
    server: <nfs.server.ip>
```

Deploy Persistent Volume
```
kubectl apply -f pv-a.yaml
```

Check Persistent Volume
```
# check pv
kubectl get pv

# check detail pv
kubectl describe pv pv-a
```

### Section 2: Create Persistent Volume Claim
Create Persistent Volume Claim
```
vim pvc-a.yaml
```

pvc-a.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-a
spec:
  storageClassName: pv-a
  resources:
    requests:
      storage: 100M
  accessModes:
    - ReadWriteMany
```

Deploy Persistent Volume Claim
```
kubectl apply -f pvc-a.yaml
```

Check Persistent Volume Claim
```
# check pvc
kubectl get pvc

# check detail pvc
kubectl describe pv pvc-a
```

### Section 3: Create Deployment
Create Deployment
```
vim deployment-vol-pv.yaml
```

deployment-vol-pv.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-vol-pv
  labels:
    app: deployment-vol-pv
spec:
  selector:
    matchLabels:
      app: deployment-vol-pv
  template:
    metadata:
      labels:
        app: deployment-vol-pv
    spec:
      volumes:
        - name: volume-pv
          persistentVolumeClaim:
            claimName: pvc-a
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: volume-pv
          mountPath: /usr/share/nginx/html
```

Deploy deployment
```
kubectl apply -f deployment-vol-pv.yaml
```

Check pod
```
kubectl get pod -l app=deployment-vol-pv
```

Access to pod
```
kubectl exec-it deployment-vol-pv-xxx -- ls /usr/share/nginx/html
```

Create file on pod
```
kubectl exec-it deployment-vol-pv-xxx -- bash

echo "<p>Test write</p>" > /usr/share/nginx/html/index.html
```

Check file on /mnt on master
```
ls /mnt/pv-a/
```

## Storage Classes
### Section 1: Install Helm
Install Helm from script
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### Section 2: Install NFS Dynamic Provisioner
Install NFS dynamic provisioner
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=<nfs.server.ip> --set nfs.path=/var/nfs/
```

### Section 3: Create storageClass
Create storage class
```
vim sc-nfs.yaml
```

sc-nfs.yaml
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sc-nfs
provisioner: cluster.local/nfs-subdir-external-provisioner
parameters:
  server: <nfs.server.ip>
  path: /var/nfs
  readOnly: "false"
```

Deploy storage class
```
kubectl apply -f sc-nfs.yaml
```

Check storage class
```
kubectl get sc
```

### Section 4: Create Deployment
Create deployment with claim name
```
vim deployment-sc-nfs.yaml
```

deployment-sc-nfs.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-sc-nfs
  labels:
    app: deployment-sc-nfs
spec:
  selector:
    matchLabels:
      app: deployment-sc-nfs
  template:
    metadata:
      labels:
        app: deployment-sc-nfs
    spec:
      volumes:
        - name: volume-sc-nfs
          persistentVolumeClaim:
            claimName: pvc-sc-nfs
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: volume-sc-nfs
          mountPath: /usr/share/nginx/html
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-sc-nfs
spec:
  storageClassName: sc-nfs
  resources:
    requests:
      storage: 1Gi
  accessModes:
    - ReadWriteMany
```

Deploy deployment
```
kubectl apply -f deployment-sc-nfs.yaml
```

Check pod
```
kubectl get pod -l app=deployment-sc-nfs
```

Access to pod
```
kubectl exec-it deployment-sc-nfs-xxx -- ls /usr/share/nginx/html
```

Create file on pod
```
kubectl exec-it deployment-sc-nfs-xxx -- bash

echo "<p>Test write</p>" > /usr/share/nginx/html/index.html
```

Check file on /mnt on master
```
ls /mnt/
```