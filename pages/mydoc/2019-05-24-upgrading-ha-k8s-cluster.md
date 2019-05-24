---
title: Upgrading kubeadm HA K8S cluster from v1.13.5 to v1.14.0
tags: [tutorials, kubernetes]
keywords: tutorials, kubernetes
last_updated: May 24, 2019
summary: "kubeadm is a tool which is a part of the Kubernetes project. It helps you deploy a Kubernetes cluster. This article will show you the way to upgrade a Highly Available Kubernetes cluster from v1.13.5 to v1.14.0."
sidebar: mydoc_sidebar
permalink: 2019-05-24-upgrading-ha-k8s-cluster.html
folder: mydoc
---

<span class="label label-success">Kubernetes</span>
<span class="label label-danger">High Availability</span>
<span class="label label-primary">kubeadm</span>
<span class="label label-info">Upgrading Cluster</span>

### 1. Deploying multi-master nodes (High Availability) K8S cluster

Follow this [tutorial guide](https://vietkubers.github.io/2019-01-31-ha-cluster-with-kubeadm.html) in order to deploy multi-master node (HA) K8S cluster.

The bare-metal server runs Ubuntu Server 16.04 and there are 7 Virtual Machines (VMs) will be installed on it. Both of the VMs also run Ubuntu Server 16.04.

* 3 master nodes
* 3 worker nodes
* 1 HAproxy load balancer


![nodes_configuration](/static/img/multi-master-ha/stacketcd.png)

{:.image-caption}
*The stacked etcd cluster*

<span class="label label-danger">The result:</span>
```sh
master1@k8s-master1:~$ sudo kubectl get node
NAME          STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP
k8s-master1   Ready    master   20h   v1.13.5   10.164.178.161   <none>     
k8s-master2   Ready    master   19h   v1.13.5   10.164.178.162   <none>      
k8s-master3   Ready    master   19h   v1.13.5   10.164.178.163   <none>      
k8s-worker1   Ready    <none>   19h   v1.13.5   10.164.178.233   <none>        
k8s-worker2   Ready    <none>   19h   v1.13.5   10.164.178.234   <none>
k8s-worker3   Ready    <none>   19h   v1.13.5   10.164.178.235   <none>
```

### 2. Upgrading the first control plane node (Master 1)  


**2.1. Find the version to upgrade to**
```sh
sudo apt update
sudo apt-cache policy kubeadm
```

**2.2. Upgrade `kubeadm` to the version that matches the version of K8s**
```sh
sudo apt-mark unhold kubeadm
sudo apt update && sudo apt upgrade
sudo apt-get install kubeadm=1.14.0-00
sudo apt-mark hold kubeadm
```

**2.3. Verify that the download works and has the expected version**
```sh
sudo kubeadm version
```

**2.4. Modify `configmap/kubeadm-config` for this control plane node by removing the `etcd` section completely**
```sh
kubectl edit configmap -n kube-system kubeadm-config
```
  
```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  ClusterConfiguration: |
    apiServer:
      certSANs:
      - 10.164.178.238
      extraArgs:
        authorization-mode: Node,RBAC
      timeoutForControlPlane: 4m0s
    apiVersion: kubeadm.k8s.io/v1beta1
    certificatesDir: /etc/kubernetes/pki
    clusterName: kubernetes
    controlPlaneEndpoint: 10.164.178.238:6443
    controllerManager: {}
    dns:
      type: CoreDNS
    etcd:
      local:
        dataDir: /var/lib/etcd
    imageRepository: k8s.gcr.io
    kind: ClusterConfiguration
    kubernetesVersion: v1.14.0
    networking:
      dnsDomain: cluster.local
      podSubnet: ""
      serviceSubnet: 10.96.0.0/12
    scheduler: {}
  ClusterStatus: |
    apiEndpoints:
      k8s-master1:
        advertiseAddress: 10.164.178.161
        bindPort: 6443
      k8s-master2:
        advertiseAddress: 10.164.178.162
        bindPort: 6443
      k8s-master3:
        advertiseAddress: 10.164.178.163
        bindPort: 6443
    apiVersion: kubeadm.k8s.io/v1beta1
    kind: ClusterStatus
kind: ConfigMap
metadata:
  creationTimestamp: "2019-05-21T10:08:03Z"
  name: kubeadm-config
  namespace: kube-system
  resourceVersion: "209870"
  selfLink: /api/v1/namespaces/kube-system/configmaps/kubeadm-config
  uid: 52419642-7bb0-11e9-8a89-0800270fde1d
```

**2.5. Upgrade the `kubelet` and `kubectl`**
```sh
sudo apt-mark unhold kubelet
sudo apt-get install kubelet=1.14.0-00 kubectl=1.14.0-00
sudo systemctl restart kubelet
```

**2.6. Start the upgrade**
```sh
sudo kubeadm upgrade apply v1.14.0
```
Logs of the upgrading process can be found [here](https://raw.githubusercontent.com/truongnh1992/upgrade-kubeadm-cluster/master/logs/cluster-upgraded-to-v1140).

### 3. Upgrading additional control plane nodes (Master 2, Master 3)

**3.1. Find the version to upgrade to**
```sh
sudo apt update
sudo apt-cache policy kubeadm
```

**3.2. Upgrade `kubeadm`**
```sh
sudo apt-mark unhold kubeadm
sudo apt update && sudo apt upgrade
sudo apt-get install kubeadm=1.14.0-00
sudo apt-mark hold kubeadm
```

**3.3. Verify that the download works and has the expected version**
```sh
sudo kubeadm version
```

**3.4. Upgrade the `kubelet` and `kubectl`**
```sh
sudo apt-mark unhold kubelet
sudo apt-get install kubelet=1.14.0-00 kubectl=1.14.0-00
sudo systemctl restart kubelet
```

**3.5 Start the upgrade**
```sh
sudo kubeadm upgrade node experimental-control-plane
```
Logs when upgrading master 2: [log-master2](https://raw.githubusercontent.com/truongnh1992/upgrade-kubeadm-cluster/master/logs/logs-master2)

Logs when upgrading master 3: [log-master3](https://raw.githubusercontent.com/truongnh1992/upgrade-kubeadm-cluster/master/logs/logs-master3)

### 4. Upgrading worker nodes (worker 1, worker 2 and worker 3)

**4.1. Upgrade `kubeadm` on all worker nodes**
```sh
sudo apt-mark unhold kubeadm
sudo apt update && sudo apt upgrade
sudo apt-get install kubeadm=1.14.0-00
sudo apt-mark hold kubeadm
```

**4.2. Cordon the worker nodes**
{% include note.html content="Run the below command on a Master node." %}
```sh
sudo kubectl drain $WORKERNODE --ignore-daemonsets
```

`$WORKERNODE`: k8s-worker1, k8s-worker2, k8s-worker3  


**4.3. Upgrade the `kubelet` config on worker nodes**
```sh
sudo kubeadm upgrade node config --kubelet-version v1.14.0
```

**4.4. Upgrade `kubelet` and `kubectl`**
```sh
sudo apt update && sudo apt upgrade
sudo apt-get install kubelet=1.14.0-00 kubectl=1.14.0-00
sudo systemctl restart kubelet
```

**4.5. Uncordon the worker nodes, bring the node back online by marking it schedulable**
```sh
sudo kubectl uncordon $WORKERNODE
```

### 5. Verifying the K8s cluster version

<span class="label label-danger">The cluster is upgraded successfully to v1.14.0</span>
```
master1@k8s-master1:~$ sudo kubectl get node
NAME          STATUS   ROLES    AGE   VERSION
k8s-master1   Ready    master   21h   v1.14.0
k8s-master2   Ready    master   21h   v1.14.0
k8s-master3   Ready    master   21h   v1.14.0
k8s-worker1   Ready    <none>   20h   v1.14.0
k8s-worker2   Ready    <none>   20h   v1.14.0
k8s-worker3   Ready    <none>   20h   v1.14.0
```

### 6. References

[1] https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-ha-1-13/  
[2] https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-14/  
[3] https://github.com/truongnh1992/upgrade-kubeadm-cluster  

*Author: [truongnh1992](https://github.com/truongnh1992)* - Email: nguyenhaitruonghp[at]gmail[dot]com

{% include links.html %}
