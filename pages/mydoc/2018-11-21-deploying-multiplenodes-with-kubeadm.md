---
title: Deploying multiple nodes with Kubeadm
tags: [docker, tutorials, kubernetes]
keywords: docker, networking, kubernetes
last_updated: November 21, 2018
sidebar: mydoc_sidebar
permalink: 2018-11-21-deploying-multiplenodes-with-kubeadm.html
folder: mydoc
---


## Preparing Environment

OS: Ubuntu 16.04 LTS

**Master Node:**
- enp0s3: NAT (Access to Internet)
- enp0s8: 
  - Host Only
  - 192.168.205.10 (Master Node IP)
        
**Worker Node:**
- enp0s3: NAT (Access to Internet)
- enp0s8:
  - Host Only
  - 192.168.205.11 (Worker Node IP)
 
Configure static ip for enp0s8 interface:
```sh
$ sudo vim /etc/network/interfaces
...
auto enp0s8
iface enp0s8 inet static
address 192.168.205.10
netmask 255.255.255.0
```
  
Configure proxy for apt:
```sh
$ sudo vim /etc/apt/apt.conf
...
Acquire::http::proxy "http://[Proxy_Server]:[Proxy_Port]/";
Acquire::HTTP::proxy "http://[Proxy_Server]:[Proxy_Port]/";
```
   
Configure proxy for docker:
```sh
$ sudo mkdir -p /etc/systemd/system/docker.service.d
$ sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://[Proxy_Server]:[Proxy_Port]/"
```
   
## Installation & Deploy

Adding kubernetes repo:
```sh
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list
$ sudo mv ~/kubernetes.list /etc/apt/sources.list.d
$ sudo apt-get update
```
 
Install docker kubelet kubeadm kubectl kubernetes-cni for each node:
```sh
$ sudo apt-get install -y docker.io kubelet kubeadm kubectl kubernetes-cni --allow-unauthenticated
```

**In case of installing a specific version of Kubernetes**
```sh
$ sudo apt-get install -qy kubelet=<version> kubectl=<version> kubeadm=<version> kubernetes-cni --allow-unauthenticated
```
**version** is:
```sh
$ curl -s https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages | grep Version | awk '{print $2}'
```
 
Deploy Master node: (In case of using flannel overlay network)
```sh
annp@k8s-master$ sudo kubeadm init --apiserver-advertise-address=${master-ip-address} --pod-network-cidr=10.244.0.0/16
```

Apply a pods network:
```sh
annp@k8s-master$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```
  
Join cluster for worker node:
```sh
annp@k8s-worker$ kubeadm join 192.168.205.10:6443 --token 2r1hcb.aa7rqdfi6rfasfqo --discovery-token-ca-cert-hash sha256:e8b11f2fe6cb313cd605ffb4eb506cb9d8dffc332af5b3f77015fefb245cd13b
```  

## Happy Learning!
