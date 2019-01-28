---
title: How to test my code (Go) on Kubernetes (K8s)
tags: [go, tutorials, kubernetes]
keywords: go
last_updated: January 28, 2019
summary: "Làm thế nào để tôi có thể test (kiểm tra) code của tôi trên cụm cluster của Kubernetes là mục tiêu của bài viết này. Trong bài viết này tôi đề cập đến trường hợp multi-node với kubeadm. Với mô hình [all-in-one](https://github.com/kubernetes/kubernetes) thì apply code trực tiếp vào ``$GOPATH/src/k8s.io/kubernetes`` và ``./hack/local-up-cluster.sh`` là ok."
sidebar: mydoc_sidebar
permalink: 2019-01-28-run-my-code-in-k8s.html
folder: mydoc
---


## 1. Các bước chuẩn bị

- Mô hình cụm kubernetes 3 nodes (3 VM) hoặc nhiều hơn: 1 master và 2 worker nodes. [Link](https://vietkubers.github.io/2018-11-21-deploying-multiplenodes-with-kubeadm.html)
- Setup $GOPATH, $GOROOT và $PATH [Link](https://vietkubers.github.io/2019-01-23-preparing-env-for-golang.html)

### Check status của các Nodes và POD

Thông tin 3 Nodes của cụm Kubernetes
![K8s Items](/static/img/compile_go_k8s/node_status.png)

Thông tin các POD & services của Kubernetes
![K8s Items](/static/img/compile_go_k8s/ready_node.png)


## 2. Các giải pháp để test code trên Kubernetes

Hiện tại, mình mới tìm ra được 2 giải pháp để hỗ trợ cho việc test code cho mô hình ``multi-nodes`` với ``kubeadm``, gồm:

* Solution 1: Thực hiện trực tiếp trên POD chạy services của Kubernetes
* Solution 2: Crash POD chạy services mình muốn và chạy trực tiếp service đó trên Node

Trong bài này, mình lấy ví dụ với service ``Scheduler`` để thực hiện. Như mọi người đã biết, Kubernetes hỗ trợ [Self-hosting](https://thenewstack.io/kubernetes-now-does-self-hosting-with-kubeadm/). Vì vậy chúng ta sẽ thấy các services sẽ chạy dưới dạng POD. Vì vậy các container sẽ run và lưu logs trên mỗi container riêng biệt.

Để thực hiện việc re-test code trên k8s, thì các developer cần clone Kubernetes thông qua lệnh ``go get -d k8s.io/kubernetes`` để lưu vào ``$GOPATH``. Tiến hành chỉnh sửa, lưu lại và compile lại code.

```bash
$ go get -d k8s.io/kubernetes
$ cd $GOPATH/src/kubernetes
$ make # Perform compile all of services and write into _output folder
$ cd _output/ # To check all compiled components.
```
{% include note.html content="  

Chúng ta có thể compile từng services/component trong K8s như kubeadm, kubectl, kube-api hoặc kube-scheduler. e.g: ``make WHAT=cmd/kubeadm``

" %}
![K8s Items](/static/img/compile_go_k8s/compile.png)

### Solution 1: Thực hiện trực tiếp trên POD chạy services của Kubernetes

Để thực hiện được giải pháp này, chúng ta cần truy cập và container và đẩy code mới vào trong container chứa service cần thay đổi.

Liệt kê các POD của Scheduler services trong ``Master Node``
```bash
$ docker ps | grep Scheduler # List out all POD of scheduler services
```
![K8s Items](/static/img/compile_go_k8s/pod_scheduler_docker.png)

Compile soure-code mới nhất của ``Scheduler``
```bash
$ cd $GOPATH/src/k8s.io/kubernetes
$ make WHAT=cmd/kube-scheduler
``` 

Kiểm tra nguồn kube-schedule binary bên trong POD scheduler
```bash
$ docker exec -it <name_of_pod> sh
```
![K8s Items](/static/img/compile_go_k8s/pod_exec.png)

Copy source-code đã compile của scheduler vào POD scheduler qua docker và restart lại POD
```bash
$ docker cp <?> <?>
$ docker restart <docker_id>
```

### Solution 2: Crash POD chạy services mình muốn và chạy trực tiếp service đó trên Node

Đây là cách tiếp cận của mình và nó sẽ giúp cho việc Debug bằng ``GoLand``. Với giải pháp này thì mình sẽ `break` POD của scheduler và chạy trực tiếp service đó trên Host. Như vậy, sẽ giúp mình check logs dễ dàng hơn.

Break POD của scheduler #Hiện mình chưa tìm được giải pháp để xóa POD trên kube-system.
Chỉnh sửa thông tin của kube-scheduler.yaml, như host hoặc port để làm break Scheduler POD.
```bash
$ vim /etc/kubernetes/manifests/kube-scheduler.yaml
```
![K8s Items](/static/img/compile_go_k8s/scheduler_manifests.png)

Check logs của scheduler sau khi chỉnh sủa trong manifests
```bash
$ kubectl --v=8 logs kube-scheduler-masternode -n kube-system
```
![K8s Items](/static/img/compile_go_k8s/error_pod_svc.png)

Và 
```bash
$ Kubectl get pod –all-namespaces
```
![K8s Items](/static/img/compile_go_k8s/error_pod.png)

Tiến hành compile lại source-code mới nhất của ``Scheduler``
```bash
$ cd $GOPATH/src/k8s.io/kubernetes
$ make WHAT=cmd/kube-scheduler
``` 

Bật ``Scheduler`` service trên ``Host``
```bash
$ cd $GOPATH/kubernetes/_output/bin
$ sudo ./kube-scheduler --address=127.0.0.1 --kubeconfig=/etc/kubernets/scheduler.conf --leader-elect=true -v=4
``` 

Check trạng thái của ``Scheduler`` trên ``Host``
```bash
$ ps -aux | grep scheduler
```
![K8s Items](/static/img/compile_go_k8s/new_scheduler.png)

Kiểm tra logs của ``Scheduler`` trên ``Host``
```bash
$ ps -aux | grep scheduler
``` 
![K8s Items](/static/img/compile_go_k8s/log_new_scheduler.png)


Bây giờ chúng ta có thể sử dụng ``Scheduler`` được chạy trực tiếp trên Host. Mỗi lần cần test, chúng ta cần compile lại code và restart lại service
```bash
$ cd $GOPATH/src/k8s.io/kubernetes
$ make WHAT=cmd/kube-scheduler
$ cd $GOPATH/kubernetes/_output/bin
$ sudo ./kube-scheduler --address=127.0.0.1 --kubeconfig=/etc/kubernets/scheduler.conf --leader-elect=true -v=4
```

Author: [Nguyễn Văn Trung](https://github.com/trungnvfet) - IRC: trungnv

## 3. References:
- https://docs.microsoft.com/en-us/virtualization/windowscontainers/kubernetes/compiling-kubernetes-binaries
