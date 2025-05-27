# Logging & Troubleshooting

## Logging
### Logging pod
create pod object
```
vim pod
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-logging
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
```

apply pod object
```
kubectl apply -f pod-logging.yml
```

logging pod
```
kubectl logs -f pod-logging
kubectl logs pod-logging > pod-logging.log
```

### Logging system
Check log system
```
journalctl -u docker
journalctl -u kubelet
dmesg
```
### Event log
Check event system
```
kubectl get events -n namespace
```

## Troubleshooting

### Troubleshoot node
Check node
```
kubectl get nodes
kubectl describe node
```
### Troubleshoot object
Check other object
```
kubectl describe pod
kubectl describe replicaset
kubectl describe deployment

kubectl describe service
kubectl describe ingress
```

