# Module 2 : Workloads and Scheduling

## Workloads
### Lab 1: Pod
#### Section 1: Single Pod
Create pod declarative
```
kubectl run pod-test --image nginx
```

Check pod
```
kubectl get pod
```

Check log pod
```
kubectl logs pod-test
```

Create template pod manifest kubernetes
```
kubectl run pod-nginx --image nginx --dry-run=client -o yaml > pod.yaml
```

pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
spec:
  containers:
  - image: nginx
    name: ct-nginx
```

Deploy pod
```
kubectl apply -f pod.yaml
```

Check pod
```
kubectl get pod
```

Check log pod
```
kubectl logs pod-nginx
```

Execution command on pod
```
kubectl exec -it pod-nginx -- bash
```

Delete pod
```
# declarative
kubectl delete pod pod-test

# with manifest
kubectl delete -f pod.yaml
```

#### Section 2: Sidecar Container
Create template pod sidecar container manifest kubernetes
```
cp pod.yaml pod-sidecar.yaml
```

pod-sidecar.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-sidecar
spec:
  containers:
  - image: nginx
    name: ct-nginx
  - image: tomcat
    name: ct-tomcat
```

Deploy pod
```
kubectl apply -f pod-sidecar.yaml
```

Check pod
```
kubectl get pod
```

Check log pod
```
# log on container ct-nginx
kubectl logs pod-sidecar -c ct-nginx

# log on container ct-tomcat
kubectl logs pod-sidecar -c ct-tomcat
```

Execution command on pod
```
# execute command on container ct-nginx
kubectl exec -it pod-sidecar -c ct-nginx -- bash

# execute command on container ct-tomcat
kubectl exec -it pod-sidecar -c ct-tomcat -- bash
```

#### Section 3: Label and Selector
Show labels on pod/node
```
kubectl get pod --show-labels
```

labeling on pod declarative
```
# labelling pod nginx
kubectl label pod pod-nginx app=nginx

# labelling pod sidecar
kubectl label pod pod-sidecar app=nginx

# labelling pod sidecar
kubectl label pod pod-sidecar app2=tomcat
```

overwrite label on pod declarative
```
kubectl label pod pod-nginx app=nginx-new --overwrite
```

delete label on pod declarative
```
kubectl label pod pod-sidecar app2-
```

Create template pod with label manifest kubernetes
```
cp pod.yaml pod-label.yaml
```

pod-label.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-label
  labels:
    app: pod-label
spec:
  containers:
  - image: nginx
    name: ct-nginx
```

Deploy pod
```
kubectl apply -f pod-label.yaml
```

filter output pod with label
```
kubectl get pod -l app=pod-label
```

filter output log pod with label
```
kubectl logs -l app=pod-label
```

#### Section 4: Annotation
Show detail info and annotation on pod
```
kubectl describe pod pod-nginx
```

Add annotation declarative
```
kubectl annotate pod pod-nginx environtment="development"
```

Create template pod with annotation manifest kubernetes
```
cp pod-label.yaml pod-annotation.yaml
```

pod-annotation.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-annotation
  labels:
    app: pod-annotation
  annotations:
    environtment: "development"
spec:
  containers:
  - image: nginx
    name: ct-nginx
```

Deploy pod
```
kubectl apply -f pod-annotation.yaml
```

### Lab 2: Namespace
#### Section 1: Create Namespace
Check namespace
```
kubectl get ns
```

Show resource on specific namespace
```
kubectl get pod -n kube-system
```

Create namespace
```
kubectl create ns ns-a
kubectl create ns ns-b
```

#### Section 2: Running object on namespace
Create template pod running on ns-a manifest kubernetes
```
cp pod.yaml pod-a.yaml
```

pod-a.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
  namespace: ns-a
spec:
  containers:
  - image: nginx
    name: ct-nginx
```

Deploy pod
```
kubectl apply -f pod-a.yaml
```

Create template pod running on ns-b manifest kubernetes
```
cp pod-a.yaml pod-b.yaml
```

pod-b.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
  namespace: ns-b
spec:
  containers:
  - image: nginx
    name: ct-nginx
```

Deploy pod
```
kubectl apply -f pod-b.yaml
```

Check pod on namespace
```
# on namespace a
kubectl get pod -n ns-a

# on namespace b
kubectl get pod -n ns-b

# on all namespace
kubectl get pod -A
```

### Lab 3: ReplicaSet
#### Section 1: Deploy ReplicaSet
Create template replicaset manifest kubernetes
```
vim replica-nginx.yaml
```

replica-nginx.yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-nginx
  labels:
    app: replica-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: replica-nginx
  template:
    metadata:
      labels:
        app: replica-nginx
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

Deploy replicaset
```
kubectl apply -f replica-nginx.yaml
```

Check replicaset
```
kubectl get rs,pod -l app=replica-nginx
```

#### Section 2: Scaling ReplicaSet
Scaling replicaset
```
# scale out replicaset
kubectl scale rs replica-nginx --replicas 3

# scale down replicaset
kubectl scale rs replica-nginx --replicas 1
```

Check replicaset
```
kubectl get rs,pod -l app=replica-nginx
```

#### Section 3: Self Healing ReplicaSet
Kill pod
```
kubectl delete pod replica-nginx-xxxx
```

Check replicaset
```
kubectl get rs,pod -l app=replica-nginx
```

### Lab 4: Deployment
#### Section 1: Deploy Deployment
Create template deployment manifest kubernetes
```
vim deployment-nginx.yaml
```

deployment-nginx.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nginx
  labels:
    app: deployment-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployment-nginx
  template:
    metadata:
      labels:
        app: deployment-nginx
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

Deploy deployment
```
kubectl apply -f deployment-nginx.yaml
```

Check deployment
```
kubectl get deploy,rs,pod -l app=deployment-nginx
```

#### Section 2: Scaling Deployment
Scaling deployment
```
# scale out deployment
kubectl scale deployment deployment-nginx --replicas 3

# scale down deployment
kubectl scale deployment deployment-nginx --replicas 1
```

Check deployment
```
kubectl get deploy,rs,pod -l app=deployment-nginx
```

#### Section 3: Self Healing Deployment
Kill pod
```
kubectl delete pod deployment-nginx-xxx-xxx
```

Check deployment
```
kubectl get deploy,rs,pod -l app=deployment-nginx
```

#### Section 4: Rollout Deployment
Check versioning on deployment
```
# check revision version on deployment
kubectl describe deployment deployment-nginx

# check history revision
kubectl rollout history deployment deployment-nginx
```

Create template deployment manifest kubernetes
```
vim deployment-nginx.yaml
```

deployment-nginx.yaml
```
    ...
    spec:
      containers:
      - name: ct-nginx
        image: nginx:stable-alpine3.20-perl
        ports:
        - containerPort: 80
```

Deploy deployment
```
kubectl apply -f deployment-nginx.yaml
```

Rollback deployment
```
kubectl rollout undo deployment deployment-nginx
```

Verify
```
# check rollout status
kubectl rollout status deployment deployment-nginx

# check history revision
kubectl rollout history deployment deployment-nginx 

# check detail revision id 2
kubectl rollout history deployment deployment-nginx --revision=2

# check detail revision id 3
kubectl rollout history deployment deployment-nginx --revision=3
```

### Lab 5: DaemonSet
#### Section 1: Deploy DaemonSet
Create template daemonset manifest kubernetes
```
vim daemonset-nginx.yaml
```

daemonset-nginx.yaml
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-nginx
  labels:
    app: daemonset-nginx
spec:
  selector:
    matchLabels:
      app: daemonset-nginx
  template:
    metadata:
      labels:
        app: daemonset-nginx
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

Deploy daemonset
```
kubectl apply -f daemonset-nginx.yaml
```

Check daemonset
```
kubectl get ds,pod -l app=daemonset-nginx
```

### Lab 6: Resource
#### Section 1: Test pod without allocation resource
Stress Test on pod-nginx
```
kubectl exec -it pod-nginx -- bash
```

On pod-nginx
```
# download stress package
apt-get update && apt-get install -y stress

# testing stress test
stress --vm 1 --vm-bytes 500M --vm-keep -t 10s
```

#### Section 2: Resource Request
Check resource usage on pod/node
```
# check on pod usage
kubectl top pod

# check on node usage
kubectl top node
```

Create template pod request alocation manifest kubernetes
```
cp pod.yaml pod-request.yaml
```

pod-request.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-request
spec:
  containers:
  - image: nginx
    name: ct-nginx
    resources:
      requests:
        memory: "5000Mi"
        cpu: "4000m"
```

Deploy pod
```
kubectl apply -f pod-request.yaml
```

Check pod
```
# check list pod
kubectl get pod

# check detail pod
kubectl describe pod pod-request
```

#### Section 3: Resource Limit
Create template pod limit allocation manifest kubernetes
```
cp pod-request.yaml pod-limit.yaml
```

pod-limit.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-limit
spec:
  containers:
  - image: nginx
    name: ct-nginx
    resources:
      requests:
        memory: "100Mi"
        cpu: "100m"
      limits:
        memory: "500Mi"
        cpu: "500m"
```

Deploy pod
```
kubectl apply -f pod-limit.yaml
```

Check pod
```
# check list pod
kubectl get pod

# check detail pod
kubectl describe pod pod-limit
```

#### Section 4: Test pod with allocation resource
Stress test on pod-limit
```
kubectl exec -it pod-limit -- bash
```

on pod pod-limit
```
# download stress package
apt-get update && apt-get install -y stress

# testing stress test
stress --vm 1 --vm-bytes 500M --vm-keep -t 10s
```

### Lab 7: HPA
#### Section 1: Deploy Deployment
Create template deployment manifest kubernetes
```
vim deployment.yaml
```

deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hpa-deployment
  labels:
    app: nginx-hpa
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-hpa
  template:
    metadata:
      labels:
        app: nginx-hpa
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
          limits:
            cpu: "500m"
            memory: "500Mi"
        ports:
        - containerPort: 80
```

Deploy deployment
```
kubectl apply -f deployment.yaml
```

Check deployment
```
kubectl get all -l app=nginx-hpa
```

#### Section 2: Deploy Service
Create template Service manifest kubernetes
```
vim service.yaml
```

service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-hpa-service
  labels:
    app: nginx-hpa
spec:
  type: ClusterIP
  selector:
    app: nginx-hpa
  ports:
  - port: 80
    targetPort: 80
```

Deploy service
```
kubectl apply -f service.yaml
```

Check service
```
kubectl get all -l app=nginx-hpa
```

#### Section 3: Deploy HPA
Create template HPA manifest kubernetes
```
vim hpa.yaml
```

hpa.yaml
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
  labels:
    app: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-hpa-deployment
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

Deploy HPA
```
kubectl apply -f hpa.yaml
```

Check HPA
```
kubectl get all -l app=nginx-hpa
```

#### Section 4: Test HPA
Install apache-benchmark
```
apt install apache2-utils
```

Test load test to service hpa
```
end=$((SECONDS+30))
while [ $SECONDS -lt $end ]; do ab -n 1000 -c 100 http://ip-service/; done
```

## Configuration
### Lab 1: ConfigMap
#### Section 1: Create ConfigMap
Create configmap manifest
```
vim config-env.yaml
```

config-env.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-env
data:
  database_url: "mongodb://localhost:27017"
  app_message: "Hello from ConfigMap!"
```

Deploy configmap
```
kubectl apply -f config-env.yaml
```

Check configmap
```
kubectl get cm
kubectl describe cm config-env
```

#### Section 2: Create ConfigMap from file
Create file .env
```
vim .env
```

.env
```
database_url: "mongodb://localhost:27017"
app_message: "Hello from ConfigMap!"
```

Create configmap from file
```
kubectl create cm config-file-env --from-file=.env
```

Check configmap
```
kubectl get cm
kubectl describe cm config-file-env
```

#### Section 3: Deploy Pod consume ConfigMap as Environtment Variable
Create template pod to load configmap as environtment variable manifest kubernetes
```
vim pod-config-env.yaml
```

pod-config-env.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-config-env
spec:
  containers:
  - image: nginx
    name: ct-nginx
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: config-env
          key: database_url
    - name: APP_MESSAGE
      valueFrom:
        configMapKeyRef:
          name: config-env
          key: app_message
```

Deploy pod
```
kubectl apply -f pod-config-env.yaml
```

Check environtment variable on pod
```
kubectl exec -it pod-config-env -- env
```

#### Section 4: Deploy Pod consume ConfigMap as Volume
Create template pod to load configmap as file manifest kubernetes
```
vim pod-config-file.yaml
```

pod-config-file.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-config-file
spec:
  containers:
  - image: nginx
    name: ct-nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: config-file-env
```

Deploy pod
```
kubectl apply -f pod-config-file.yaml
```

Check file configuration on pod
```
kubectl exec -it pod-config-file -- ls -la /etc/config
kubectl exec -it pod-config-file -- cat /etc/config/.env
```

#### Section 5: Update ConfigMaps as Environtment Variable
Update template configmap
```
vim config-env.yaml
```

config-env.yaml
```
...
data:
  database_url: "mongodb://localhost:32001"
  app_message: "Hello from Modified ConfigMap!"
```

Deploy configmap
```
kubectl apply -f config-env.yaml
```

Check environtment variable on pod
```
kubectl exec -it pod-config-env -- env
```

Recreate pod
```
# delete pod
kubectl delete -f pod-config-env.yaml

# create new pod
kubectl apply -f pod-config-env.yaml
```

#### Section 6: Update ConfigMaps as Volume
Update file .env
```
database_url: "mongodb://localhost:32001"
app_message: "Hello from Modified ConfigMap!"
```

Replace existing configmaps using new update file
```
kubectl create configmap config-file-env --from-file .env -o yaml --dry-run=client | kubectl apply -f -
```

Check file configuration on pod
```
# check file
kubectl exec -it pod-config-file -- ls -la /etc/config

# read file .env
kubectl exec -it pod-config-file -- cat /etc/config/.env
```

### Lab 2: Secret
#### Section 1: Create Secret
Create secret
```
kubectl create secret generic secret-env --from-literal=username=admin --from-literal=password=admin123
```

Check secret
```
kubectl get secrets
kubectl describe secret secret-env
kubectl get secret secret-env -o yaml
```

#### Section 2: Create Secret from file
Create file .env
```
vim .env
```

.env
```
username: admin
password: admin123
```

Create secret from file
```
kubectl create secret generic secret-file-env --from-file=.env
```

Check secret
```
kubectl get secrets
kubectl describe secret secret-file-env
kubectl get secret secret-file-env -o yaml
```

#### Section 3: Deploy Pod consume Secret as Environtment Variable
Create template pod to load secret as environtment variable manifest kubernetes
```
vim pod-secret-env.yaml
```

pod-secret-env.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-env
spec:
  containers:
  - image: nginx
    name: ct-nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: secret-env
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: secret-env
          key: password
```

Deploy pod
```
kubectl apply -f pod-secret-env.yaml
```

Check environtment variable on pod
```
kubectl exec -it pod-secret-env -- env
```

#### Section 4: Deploy Pod consume Secret as Volume
Create template pod to load secret as file manifest kubernetes
```
vim pod-secret-file.yaml
```

pod-secret-file.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-file
spec:
  containers:
  - image: nginx
    name: ct-nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
  volumes:
  - name: secret-volume
    secret:
      secretName: secret-file-env
```

Deploy pod
```
kubectl apply -f pod-secret-file.yaml
```

Check file configuration on pod
```
kubectl exec -it pod-secret-file -- ls -la /etc/secrets
kubectl exec -it pod-secret-file -- cat /etc/secrets/.env
```

#### Section 5: Update Secret as Environtment Variable
Recreate secret
```
# delete existing secret
kubectl delete secret secret-env

# create new secret
kubectl create secret generic secret-env --from-literal=username=admin --from-literal=password=@dmin123!
```

Check environtment variable on pod
```
kubectl exec -it pod-secret-env -- env
```

Recreate pod
```
# delete pod
kubectl delete -f pod-secret-env.yaml

# create new pod
kubectl apply -f pod-secret-env.yaml
```

#### Section 6: Update Secret as Volume
Update file .env
```
username: admin
password: @dmin123!
```

Replace existing secret using new update file
```
kubectl create secret generic secret-file-env --from-file .env -o yaml --dry-run=client | kubectl apply -f -
```

Check file configuration on pod
```
# check file
kubectl exec -it pod-secret-file -- ls -la /etc/secrets

# read file .env
kubectl exec -it pod-secret-file -- cat /etc/secrets/.env
```

## Scheduling
### Lab 1: nodeName
#### Section 1: Deploy Deployment use schedule nodeName 
Check pod running
```
kubectl get pod -o wide
```

copy deployment template to create deployment running on specific node
```
vim deployment-worker2.yaml
```

deployment-worker2.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-worker2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployment-worker2
  template:
    metadata:
      labels:
        app: deployment-worker2
    spec:
      nodeName: k8s-worker2
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

Deploy deployment
```
kubectl apply -f deployment-worker2.yaml
```

### Lab 2: nodeSelector
#### Section 1: Labeling a node
Check label on node
```
kubectl get node --show-labels
```

Labeling node
```
kubectl label node k8s-worker1 disk-type=ssd
kubectl label node k8s-worker2 disk-type=hdd
```

#### Section 2: Deploy Deployment use schedule nodeSelector
copy deployment template to create deployment running on specific node
```
cp deployment-worker2.yaml deployment-ssd.yaml
```

deployment-ssd.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-ssd
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployment-ssd
  template:
    metadata:
      labels:
        app: deployment-ssd
    spec:
      nodeSelector:
        disk-type: ssd
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

Deploy deployment
```
kubectl apply -f deployment-worker2.yaml
```

### Lab 3: nodeAffinity
#### Section 1: requiredDuringSchedulingIgnoredDuringExecution
copy deployment template to create deployment running on specific node with node affinity requiredDuringSchedulingIgnoredDuringExecution
```
vim deployment-node-affinity.yaml
```

deployment-node-affinity.yaml
```
apiVersion: apps/v1
kind: Deployment                      
metadata:                                
  name: deployment-node-affinity
spec:        
  replicas: 2   
  selector:                                
    matchLabels:
      app: deployment-node-affinity
  template:          
    metadata:                                                                                            
      labels:                 
        app: deployment-node-affinity
    spec:                              
      affinity:                                                                                                                         
        nodeAffinity:         
          requiredDuringSchedulingIgnoredDuringExecution:                                                                                                     
            nodeSelectorTerms:         
            - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values:    
                - amd64     
      containers:                                                   
      - name: ct-nginx                                              
        image: nginx                                                
        ports:                                                      
        - containerPort: 80                                                    
```

deploy deployment
```
kubectl apply -f deployment-node-affinity.yaml
```

check deployment
```
kubectl get all -l app=deployment-node-affinity -o wide 
```

#### Section 2: preferredDuringSchedulingIgnoredDuringExecution
copy deployment template to create deployment running on specific node with node affinity preferredDuringSchedulingIgnoredDuringExecution
```
vim deployment-node-affinity.yaml
```

deployment-node-affinity.yaml
```
apiVersion: apps/v1
kind: Deployment                      
metadata:                                
  name: deployment-node-affinity
spec:        
  replicas: 2   
  selector:                                
    matchLabels:
      app: deployment-node-affinity
  template:          
    metadata:                                                                                            
      labels:                 
        app: deployment-node-affinity
    spec:                              
      affinity:                                                                                                                         
        nodeAffinity:         
          requiredDuringSchedulingIgnoredDuringExecution:                                                                                                     
            nodeSelectorTerms:         
            - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values:    
                - amd64     
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: disk-type
                operator: In
                values:
                - hdd
          - weight: 10
            preference:
              matchExpressions:
              - key: disk-type
                operator: In
                values:
                - ssd
      containers:                                                   
      - name: ct-nginx                                              
        image: nginx                                                
        ports:                                                      
        - containerPort: 80   
```

deploy deployment
```
kubectl apply -f deployment-node-affinity.yaml
```

check deployment
```
kubectl get all -l app=deployment-node-affinity -o wide 
```

### Lab 4: podAffinity and podAntiAffinity
#### Section 1: podAffinity
copy deployment template to create deployment running on specific node
```
vim deployment-pod-affinity.yaml
```

deployment-pod-affinity.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-pod-affinity
spec:
  selector:
    matchLabels:
      app: deployment-pod-affinity
  template:
    metadata:
      labels:
        app: deployment-pod-affinity
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - deployment-pod-affinity
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

deploy deployment
```
kubectl apply -f deployment-pod-affinity.yaml
```

check deployment
```
kubectl get all -l app=deployment-pod-affinity -o wide 
```

#### Section 2: podAntiAffinity
copy deployment template to create deployment running on specific node
```
vim deployment-pod-anti-affinity.yaml
```

deployment-pod-anti-affinity.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-anti-affinity
spec:
  selector:
    matchLabels:
      app: deployment-anti-affinity
  template:
    metadata:
      labels:
        app: deployment-anti-affinity
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - deployment-anti-affinity
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

deploy deployment
```
kubectl apply -f deployment-pod-anti-affinity.yaml
```

check deployment
```
kubectl get all -l app=deployment-anti-affinity -o wide 
```

### Lab 5: Taints and Toleration
#### Section 1: Taints
Check node
```
kubectl get node
```

Check taints on node
```
kubectl describe node | grep -i taints
```

Add taints to node
```
kubectl taint node k8s-worker1 critical=yes:NoSchedule
kubectl taint node k8s-worker2 nvidia-gpu=yes:NoExecute
```

copy deployment template to create deployment running on specific node
```
deployment-without-toleration.yaml
```

deployment-without-toleration.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-without-toleration
  labels:
    app: deployment-without-toleration
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployment-without-toleration
  template:
    metadata:
      labels:
        app: deployment-without-toleration
    spec:
      containers:
      - name: ct-nginx
        image: nginx:stable-alpine3.20-perl
        ports:
        - containerPort: 80
```

Deploy deployment
```
kubectl apply -f deployment-without-toleration.yaml
```

Check pending pod
```
kubectl get pod -l app=deployment-without-toleration
```

Untaints node
```
kubectl taint node k8s-worker2 nvidia-gpu=yes:NoExecute-
```

#### Section 2: Toleration
Copy deployment template to create deployment with toleration
```
vim deployment-with-toleration.yaml
```

deployment-with-toleration.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-with-toleration
  labels:
    app: deployment-with-toleration
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployment-with-toleration
  template:
    metadata:
      labels:
        app: deployment-with-toleration
    spec:
      containers:
      - name: ct-nginx
        image: nginx:stable-alpine3.20-perl
        ports:
        - containerPort: 80
      tolerations:
      - key: "critical"
        operator: "Equal"
        value: "yes"
        effect: "NoSchedule"
```

Deploy deployment
```
kubectl apply -f deployment-with-toleration.yaml
```

check deployment
```
kubectl get all -l app=deployment-with-toleration -o wide 
```