# Pod
you can create pod directly
```
kubectl run pod-test --image nginx
```
or you can create template for reusable create pod
```
  kubectl run pod-test --image nginx --dry-run=client -o yaml > pod.yml
```
and to create pod from a template like this
```
kubectl apply -f pod.yml
```

# Pod with Label & Selector
add label directly
```
kubectl label pod pod-name key=value
```

overwrite label
```
kubectl label pod pod-name key=value --overwrite
```

delete label
```
kubectl label pod pod-name key-
```

add label on template
```
cp pod.yml pod-label.yml
vim pod-label.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
# add this
  labels:   
    apps-type: frontend
  name: pod-label
spec:
  containers:
  - image: nginx
    name: pod-test
```

deploy pod label
```
kubectl apply -f pod-label.yml
```

# Pod with annotation
add annotate directly
```
kubectl annotate pod pod-name key=value
```

copy template
```
cp pod.yml pod-annotation.yml
vim pod-annotation.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    apps-type: frontend
# add this
  annotations:
    description: ini belajar membuat pod
  name: pod-annotate
spec:
  containers:
  - image: nginx
    name: pod-test
```

deploy pod annotations
```
kubectl apply -f pod-annotation.yml
```

# Pod with resource limit container
copy template
```
cp pod.yml pod-limit.yml
vim pod-limit.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-limit
  labels:
    apps-type: backend
  annotations:
    deskripsi: ini aplikasi dibuat untuk testing
spec:
  containers:
  - name: pod-limit
    image: nginx
# add this
    resources:
      requests:
        memory: "1024Mi"
        cpu: "100m"
      limits:
        memory: "1024Mi"
        cpu: "200m"
    ports:
      - containerPort: 80
```

deploy pod with limit resource
```
kubectl apply -f pod-limit.yml
```

testing benchmark
```
apt install apache2-utils
ab -n 10000 -c 10 http://url_address/
```

# Pod static
copy template
```
cd /etc/kubernetes/manifest
vim pod-static.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
```

restart kubernetes agent
```
systemctl restart kubelet
```

# Namespace
create namespace dev
```
kubectl create ns dev
```

create pod specific on namespace
```
kubectl run pod-dev -n dev --image nginx
```

show pod on namespace
```
kubectl get pod -n dev
```

create template pod specific on namespace
```
cp pod.yml pod-ns-dev.yml
vim pod-ns-dev.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    env: dev
    apps-type: frontend-dev
  name: pod-nginx
# add this
  namespace: dev
spec:
  containers:
  - name: ct-nginx
    image: nginx
    ports:
      - containerPort: 80
```

deploy pod namespace
```
kubectl apply -f pod-ns-dev.yml
```

# Pod with env variable
copy template
```
cp pod.yml pod-env.yml
vim pod-env.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-env
  labels:
    type: backend
  annotations:
    deskripsi: ini aplikasi dibuat untuk testing
spec:
  containers:
  - name: ct-nginx
    image: nginx
    ports:
      - containerPort: 80
# add this
    env:
      - name: USER
        value: admin
      - name: PASS
        value: admin123
```

deploy pod env
```
kubectl apply -f pod-env.yml
```

# Configmaps
create configmaps env
```
vim config-env.yml
```

add this
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-env
data:
  HOST: 10.23.0.1
  USER: admin
  PASS: admin123
```

deploy configmaps
```
kubectl apply -f config-env.yml
```

create pod to use configmaps
```
cp pod.yml pod-config-env.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-config-env
  labels:
    type: backend
  annotations:
    deskripsi: ini aplikasi dibuat untuk testing
spec:
  containers:
  - name: ct-nginx
    image: nginx
    ports:
      - containerPort: 80
# add this
    envFrom:
      - configMapRef:
          name: config-env
```

deploy pod configmaps
```
kubectl apply -f pod-config-env.yml
```

create configmaps volume
```
vim nginx-config.yml
```

add this
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-config
data:
  nginx.conf: |
    events {
    }
    http {
      server {
        listen 80 default_server;
        listen [::]:80 default_server;
        
        # Set nginx to serve files from the shared volume!
        root /usr/share/nginx/html;
        server_name _;
        location / {
          try_files $uri $uri/ =404;
        }
      }
    }
```

deploy configmaps
```
kubectl apply -f nginx-config.yml
```

create pod to use configmaps
```
cp pod.yml pod-nginx-config.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx-config
  labels:
    apps-type: backend
  annotations:
    deskripsi: ini aplikasi dibuat untuk testing
spec:
# add this
  volumes:
  - name: nginx-config-volume
    configMap:
      name: nginx-config
      items:
        - key: nginx.conf
          path: nginx.conf
  containers:
  - name: ct-nginx
    image: nginx
    ports:
      - containerPort: 80
# add this
    volumeMounts:
      - name: nginx-config-volume
        mountPath: /etc/nginx/
        readOnly: true
```

deploy pod configmaps
```
kubectl apply -f pod-nginx-config.yml
```
# Secret
create .env
```
vim .env
```

add this
```
HOST: 10.23.0.1
USER: admin
PASS: admin123
```

create secret from .env file
```
kubectl create secret generic env-file --from-file .env
```

create pod to use secret
```
vim pod-secret.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-env-secret
  labels:
    apps-type: backend
  annotations:
    deskripsi: ini aplikasi dibuat untuk testing
spec:
  volumes:
  - name: secret-env
    secret:
      secretName: env-file
      items:
        - key: .env
          path: .env
  containers:
  - name: ct-nginx
    image: nginx
    ports:
      - containerPort: 80
    volumeMounts:
      - name: secret-env
        mountPath: /usr/share/nginx/html/.env
        subPath: .env
        readOnly: true
```

deploy pod configmaps
```
kubectl apply -f pod-secret.yml
```

# ReplicaSets 
create replicaset template
```
vim replica-nginx.yml
```

add this
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      apps-type: nginx-replicaset
  template:
    metadata:
      labels:
        apps-type: nginx-replicaset
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

deploy replicaset
```
kubectl apply -f replica-nginx.yml
```

# Deployment
create deployment template
```
vim deploy-nginx.yml
```

add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      apps-type: nginx-deployment
  template:
    metadata:
      labels:
        apps-type: nginx-deployment
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

deploy deployment
```
kubectl apply -f replicaset.yml
```

# DaemonSet
create daemonset template
```
vim daemon-nginx.yml
```

add this
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-nginx
spec:
  selector:
    matchLabels:
      apps-type: nginx-daemonset
  template:
    metadata:
      labels:
        apps-type: nginx-daemonset
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
          - containerPort: 80
```

deploy daemonset
```
kubectl apply -f daemonset.yml
```

# Horizontal Pod Autoscaler
create deployment HPA template
```
vim deployment-hpa.yml
```

add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-hpa
spec:
  selector:
    matchLabels:
      type: testing-hpa
  template:
    metadata:
      labels:
        type: testing-hpa
    spec:
      containers:
      - name: ct-php-apache
        image: idndevops/hpa-example
        resources:
          limits:
            memory: "100Mi"
            cpu: "100m"
        ports:
        - containerPort: 80
```

deploy deployment hpa
```
kubectl apply -f deployment-hpa.yml
```

create service HPA template
```
vim service-hpa.yml
```

add this
```
apiVersion: v1
kind: Service
metadata:
 name: svc-hpa
spec:
 ports:
 - port: 80
 selector:
  type: testing-hpa
```

deploy service hpa
```
kubectl apply -f service-hpa.yml
```

create hpa template
```
vim hpa.yml
```

add this
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-testing
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: deployment-hpa
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

deploy hpa
```
kubectl apply -f hpa.yml
```

Testing HPA
```
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://svc-hpa; done"
```
