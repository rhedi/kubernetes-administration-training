# Networking Practice Docs

## Network Policy
check namespace and pod label
```
kubectl get ns --show-labels
kubectl get pod -n dev --show-labels
```

create default network policy on namespace dev
```
vim dev-netpolicy.yml
```

add this
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: dev-netpolicy
  namespace: dev
spec:
  podSelector:
    matchLabels:
      run: pod-dev
  policyTypes:
  - Ingress
  - Egress
  ##ingress:
  ##- from:
  ##  - namespaceSelector:
  ##     matchLabels:
  ##        kubernetes.io/metadata.name: default
  ##egress:
  ##- to:
  ##  ports:
  ##  - protocol: TCP
  ##    port: 80
```

testing connection on host
```
curl http://ip_pod//
```

testing connection on pod on namespace default
```
kubectl exec -it pod-name -- /bin/bash
curl http://ip_pod//
```

create rule to allow inbound
```
vim dev-netpolicy.yml
```

add this
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: dev-netpolicy
  namespace: dev
spec:
  podSelector:
    matchLabels:
      run: pod-dev
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
       matchLabels:
          kubernetes.io/metadata.name: default
  ##egress:
  ##- to:
  ##  ports:
  ##  - protocol: TCP
  ##    port: 80
```

testing connection on host
```
curl http://ip_pod//
```

testing connection on pod on namespace default
```
kubectl exec -it pod-name -- /bin/bash
curl http://ip_pod//
```

create rule to allow outbound
```
vim dev-netpolicy.yml
```

add this
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: dev-netpolicy
  namespace: dev
spec:
  podSelector:
    matchLabels:
      run: pod-dev
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
       matchLabels:
          kubernetes.io/metadata.name: default
  egress:
  - to:
    ports:
    - protocol: TCP
      port: 80
```

testing connection on host
```
curl http://ip_pod//
```

testing connection on pod on namespace default
```
kubectl exec -it pod-name -- /bin/bash
curl http://ip_pod//
```

## Service
### Servie ClusterIp
create service clusterip
```
vim svc-nginx.yml
```

add this
```
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  selector:
    app: nginx-deploy
  ports:
  - port: 80
    targetPort: 80
```

test connection
```
kubectl get svc
curl http://cluster_ip
```
### Service NodePort
create service nodeport
```
vim svc-nginx.yml
```

add this
```
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  selector:
    app: nginx-deploy
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30000
```

test connection
```
kubectl get svc
```

open url on browser
```
http://ip_node:node_port
```

### Service Load Balancer
create service load balancer
```
vim svc-nginx.yml
```

add this
```
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  selector:
    app: nginx-deploy
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
```

test connection
```
kubectl get svc
```

open url on browser
```
http://ip_loadbalancer/
```

## Metallb
If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.

Note, you don’t need this if you’re using kube-router as service-proxy because it is enabling strict ARP by default.

You can achieve this by editing kube-proxy config in current cluster:
```
kubectl edit configmap -n kube-system kube-proxy
```

and set:
```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true #set this to true
```

To install MetalLB, apply the manifest:
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

## set range ip for metallb loadbalancer
Layer 2 mode is the simplest to configure: in many cases, you don’t need any protocol-specific configuration, only IP addresses.

Layer 2 mode does not require the IPs to be bound to the network interfaces of your worker nodes. It works by responding to ARP requests on your local network directly, to give the machine’s MAC address to clients.

In order to advertise the IP coming from an **IPAddressPool**, an **L2Advertisement** instance must be associated to the **IPAddressPool**.

Create centralize directory
```
mkdir Metallb
```

Create ip pool range
```
vim Metallb/ip-pool.yml
```

Add this
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ip-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.23.x.x-172.23.x.x
```

Create l2 advertisement
```
vim Metallb/l2-advertisement.yml
```

Add this
```
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advertise
  namespace: metallb-system
spec:
  ipAddressPools:
  - ip-pool
```

Apply metallb
```
kubectl apply -f Metallb/
```
## Ingress
### Install nginx ingress

Installation manifest for cloud / baremetal with metallb
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
```
### single path
create ingress 
```
vim ingress.yml
```

add this
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: domainku.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: svc-nginx
            port: 
              number: 80
```

apply ingress
```
kubectl apply -f ingress.yml
```

### virtual hosting
create ingress
```
vim ingress.yml
```
add this
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: domainku.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: svc-nginx
            port: 
              number: 80
  - host: apps.domainku.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: svc-apps
            port: 
              number: 80
```
apply ingress
```
kubectl apply -f ingress.yml
```

### TLS
Generate certificate
```
cd /etc/ssl
mkdir domainku.com
openssl req -newkey rsa:2048 -nodes -keyout domainku.key -x509 -days 365 -out domainku.crt
```

create secret ssl from certificate
```
kubectl create secret tls ssl-domainku --key domainku.key --cert domainku.crt
kubectl get secret
kubectl describe secrets/ssl-domainku
```

deploy ingress with ssl
```
vim ingress.yml
```

add this
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - domainku.com
    secretName: ssl-domainku

  rules:
  - host: domainku.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: svc-nginx
            port: 
              number: 80
  - host: apps.domainku.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: svc-apps
            port: 
              number: 80
```

apply ingress
```
kubectl apply -f ingress.yml
```

## DNS
### Services domain
```
#format domain
my-svc.my-namespace.svc.cluster-domain.example

#example
svc-hpa.default.svc.cluster.local 
```

### Pods domain
```
#format domain
pod-ip-address.my-namespace.pod.cluster-domain.example

#example domain
172-17-0-3.default.pod.cluster.local 
```
