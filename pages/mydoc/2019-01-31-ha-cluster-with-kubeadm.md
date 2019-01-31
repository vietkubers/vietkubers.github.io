---
title: Deploying multi-master nodes (High Availability) K8S
tags: [tutorials, kubernetes]
keywords: tutorials, kubernetes
last_updated: January 31, 2019
summary: "kubeadm is a tool which is a part of the Kubernetes project. It helps you deploy a Kubernetes cluster but it still has some limitations and one of these is that it doesn't support multi-master nodes (HA). This article will show you the way to create a HA Cluster with kubeadm."
sidebar: mydoc_sidebar
permalink: 2019-01-31-ha-cluster-with-kubeadm.html
folder: mydoc
---

<span class="label label-success">Kubernetes</span>
<span class="label label-info">High Availability</span>

## 1. Preparation

### 1.1. Installing bare-metal server and creating necessary VMs

The bare-metal server runs Ubuntu Server 16.04 and there are 9 Virtual Machines (VMs) will be installed on it. Both of the VMs also run Ubuntu Server 16.04.

* 3 master nodes
* 5 worker nodes
* 1 HAproxy load balancer

![nodes_configuration](/static/img/multi-master-ha/nodes_configuration.PNG)

{:.image-caption}
*The configurations of nodes*

### 1.2. Installing docker kubelet kubeadm kubectl kubernetes-cni on `master nodes` and `worker nodes`

Adding kubernetes repo:
```sh
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list
$ sudo mv ~/kubernetes.list /etc/apt/sources.list.d
$ sudo apt-get update
```
```sh
$ sudo apt-get install -y docker.io kubelet kubeadm kubectl kubernetes-cni --allow-unauthenticated
```

{% include note.html content="If your machines (all of the above VMs) run behind the **proxy**, please follow the instructions below. If NO, skip it and go to [section 1.2](12-installing-haproxy-load-balancer)" %}

**Configuring proxy for apt**
```sh
$ sudo vim /etc/apt/apt.conf

Acquire::http::proxy "http://[Proxy_Server]:[Proxy_Port]/";
Acquire::HTTP::proxy "http://[Proxy_Server]:[Proxy_Port]/";
```

**Configuring proxy for docker**
```sh
$ sudo mkdir -p /etc/systemd/system/docker.service.d
$ sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://[Proxy_Server]:[Proxy_Port]/"
```

### 1.3. Installing HAproxy load balancer

Installing haproxy on `ha machine` (IP: 192.168.1.33)
```sh
$ sudo apt-get install haproxy
```

Configuring HAProxy to load balance the traffic between 3 master nodes.

```sh
$ sudo vim /etc/haproxy/haproxy.cfg
```
Modifying content of file `haproxy.cfg` as below:
```python
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
...
...

frontend kubernetes
        bind 192.168.1.33:6443
        option tcplog
        mode tcp
        default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
        mode tcp
        balance roundrobin
        option tcp-check
        server k8s-master1 192.168.1.11:6443 check fall 3 rise 2
        server k8s-master2 192.168.1.12:6443 check fall 3 rise 2
        server k8s-master3 192.168.1.13:6443 check fall 3 rise 2
```

{% include note.html content="  

- The health check for an apiserver is a TCP check on the port which the kube-apiserver listen on. The default value: **6443**    

- In frontend section: bind to `ha machine` IP address (192.168.1.33)  

- In backend section: Notice the hostname and IP address of 3 master nodes    

" %}

Restart the HAproxy
```sh
$ sudo systemctl restart haproxy.service
```

## 2. Creating HA cluster with kubeadm

### 2.1. Steps for the 1st master node

On the master node `master1` (IP: 192.168.1.11), create a configuration file `kubeadm-config.yaml`:

```yaml
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: stable
apiServer:
  certSANs:
  - "192.168.1.33"
controlPlaneEndpoint: "192.168.1.33:6443"
```

{% include note.html content="  

- The `kubernetesVersion` is the Kubernetes version which is using. This configuration uses `stable`    

- The `controlPlaneEndpoint` is the `ha machine`'s IP address with port 6443     

" %}

Deploying node master1:

```sh
$ sudo kubeadm init --config=kubeadm-config.yaml
```

The terminal will print something like this:
```
...
You can now join any number of machines by running the following on each node
as root:

kubeadm join 192.168.1.33:6443 --token 7ju4yg.5x2xaj96xqx18qwq --discovery-token-ca-cert-hash sha256:4d7c5ef142e4faca3573984119df92a1a188115723f1e81dbb27eeb039cac1e0
```

{% include tip.html content="Save the output **kubeadm join 192.168.1.33:6443 --token...** to a text file in order to join other `master nodes` to the cluster." %}

Applying the [Weave](https://www.weave.works/blog/cni-for-docker-containers/) CNI plugin:
```sh
$ sudo kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

**Verifying that the pods of the components are ready**:
```sh
$ sudo kubectl get pod -n kube-system -w
```

```sh
NAME                                  READY   STATUS    RESTARTS   AGE
coredns-86c58d9df4-8vcxh              1/1     Running   0          4h38m
coredns-86c58d9df4-ts6x2              1/1     Running   0          4h38m
etcd-k8s-master1                      1/1     Running   0          4h37m
kube-apiserver-k8s-master1            1/1     Running   0          4h37m
kube-controller-manager-k8s-master1   1/1     Running   0          4h37m
kube-proxy-dhnjk                      1/1     Running   0          4h38m
kube-scheduler-k8s-master1            1/1     Running   0          4h37m
weave-net-cqb88                       2/2     Running   0          4h22m
```

{% include tip.html content="Make sure that after the 1st master node has finished initializing, then join new master nodes." %}

**Copy the certificate files from the `1st master node` to the `master2` and `master3`**

```sh
$ vim copy.sh

USER=root
MASTER_NODE_IPS="192.168.1.12 192.168.1.13"
for host in ${MASTER_NODE_IPS}; do
   scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
   scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
   scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
   scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
   scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
   scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
   scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:etcd-ca.crt
   scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:etcd-ca.key
   scp /etc/kubernetes/admin.conf "${USER}"@$host:
done
```

{% include note.html content="Run above script with user `root` of **1st master node**." %}

```sh
root@k8s-master1:~# sh copy.sh
```
After running successfully `copy.sh`, the certificates will be located in directory `/root` of nodes: **master1** and **master2**.

### 2.2. Steps for the rest of the master nodes

Move the files created by the previous step where `scp` was used:

```sh
$ vim move.sh

USER=root
mkdir -p /etc/kubernetes/pki/etcd
mv /${USER}/ca.crt /etc/kubernetes/pki/
mv /${USER}/ca.key /etc/kubernetes/pki/
mv /${USER}/sa.pub /etc/kubernetes/pki/
mv /${USER}/sa.key /etc/kubernetes/pki/
mv /${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
mv /${USER}/front-proxy-ca.key /etc/kubernetes/pki/
mv /${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
mv /${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
mv /${USER}/admin.conf /etc/kubernetes/admin.conf
```

**On node master2:**
```sh
root@k8s-master2:~# sh move.sh
```

**On node master3:**
```sh
root@k8s-master3:~# sh move.sh
```

Start **`kubeadm join`** on nodes master2 and master3 using the join command in section 2.1 and add the flag `--experimental-control-plane`

**On node master2:**
```sh
$ sudo kubeadm join 192.168.1.33:6443 --token 7ju4yg.5x2xaj96xqx18qwq --discovery-token-ca-cert-hash sha256:4d7c5ef142e4faca3573984119df92a1a188115723f1e81dbb27eeb039cac1e0 --experimental-control-plane
```

**On node master3:**
```sh
$ sudo kubeadm join 192.168.1.33:6443 --token 7ju4yg.5x2xaj96xqx18qwq --discovery-token-ca-cert-hash sha256:4d7c5ef142e4faca3573984119df92a1a188115723f1e81dbb27eeb039cac1e0 --experimental-control-plane
```

**Verifying that the pods of the components are ready**:
```sh
$ sudo kubectl get pod -n kube-system -w
```

### 2.3. Installing workers

All of worker nodes can be joined to the cluster by command:

```sh
$ sudo kubeadm join 192.168.1.33:6443 --token 7ju4yg.5x2xaj96xqx18qwq --discovery-token-ca-cert-hash sha256:4d7c5ef142e4faca3573984119df92a1a188115723f1e81dbb27eeb039cac1e0
```
### 3. References

[1] https://kubernetes.io/docs/setup/independent/high-availability/


Author: [truongnh1992](https://github.com/truongnh1992) - Email: nguyenhaitruonghp[at]gmail[dot]com

{% include links.html %}
