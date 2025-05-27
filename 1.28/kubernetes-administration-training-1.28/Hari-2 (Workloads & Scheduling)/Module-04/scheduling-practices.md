# node name
create deployment with nodename
```
vim deployment-nodename.yml
```

add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nodename
spec:
  replicas: 2
  selector:
    matchLabels:
      type: nodename-test
  template:
    metadata:
      labels:
        type: nodename-test
    spec:
      nodeName: worker2-rafi
      containers:
      - name: ct-nodename
        image: nginx
        ports:
        - containerPort: 80
```

deploy deployment nodename
```
kubectl apply -f deployment-nodename.yml
```
# node selector
create label to node
```
kubectl label node node-name key=value
```

create deployment with node selector
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-selector
spec:
  replicas: 2
  selector:
    matchLabels:
      type: selector-test
  template:
    metadata:
      labels:
        type: selector-test
    spec:
      containers:
      - name: ct-selector
        image: nginx
        ports:
        - containerPort: 80
      nodeSelector:
        cpu: intel
```

deploy deployment node selector
```
kubectl apply -f deployment-selector.yml
```
# node affinity
create deployment with node affinity (hard)
```
vim deployment-required-affinity.yml
```

add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-required-affinity
spec:
  replicas: 2
  selector:
    matchLabels:
      type: required-affinity-test
  template:
    metadata:
      labels:
        type: required-affinity-test
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: disktype
                operator: In
                values:
                - sata
      containers:
      - name: ct-required-affinity
        image: nginx
        ports:
        - containerPort: 80
```

deploy deployment with node affinity (hard)
```
kubectl apply -f required-affinity.yml
```

create deployment with node affinity (soft)
```
vim deployment-preffered-affinity.yml
```

add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-preffered-affinity
spec:
  replicas: 2
  selector:
    matchLabels:
      type: preffered-affinity-test
  template:
    metadata:
      labels:
        type: preffered-affinity-test
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd
      containers:
      - name: ct-preferred-affinity
        image: nginx
        ports:
        - containerPort: 80
```

deploy deployment with node affinity (soft)
```
kubectl apply -f preferred-affinity.yml
```

# taints
add taint label to node
```
kubectl taint node node key=value:NoSchedule
kubectl taint node node key=value:NoExecute
```

# tolerations
create deployment with toleration
```
vim deployment-toleration.yml
```

add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-toleration
spec:
  replicas: 2
  selector:
    matchLabels:
      type: toleration-test
  template:
    metadata:
      labels:
        type: toleration-test
    spec:
      containers:
      - name: ct-toleration
        image: nginx
        ports:
        - containerPort: 80
      tolerations:
      - key: "ready"
        operator: "Equal"
        value: "yes"
        effect: "NoSchedule"
      #- key: "ready"
      #  operator: "Equal"
      #  value: "no"
      #  effect: "NoExecute"
      #  tolerationSeconds: 60
```

deploy deployment with toleration
```
kubectl apply -f deployment-toleration.yml
```
