# Pre install server on all node
### login as root
```
sudo su
```

### change hostname
```
hostnamectl set-hostname hostname
```

### set timezone
```
timedatectl set-timezone Asia/Jakarta
```

### change static ip
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
      addresses: [10.10.x.x/22]
      routes:
        - to: default
          via: 10.10.0.1
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
  version: 2
```

reload configuration
```
netplan apply
```

check ip
```
ip a
```

### update & upgrade repository
```
apt update -y && apt upgrade -y
```

### add host (example)
```
vim /etc/hosts
```

```
10.10.10.11 kube-master
10.10.10.12 kube-worker-01
10.10.10.13 kube-worker-02
```