---
title: Go Series - Preparing environment for Go
tags: [go]
last_updated: January 01, 2019
sidebar: mydoc_sidebar
permalink: preparing-env-for-go.html
folder: mydoc
---

## So lu?c v? ngôn ng? GO:

Go là m?t ngôn ng? l?p trình mã ngu?n m? [Open Source](https://github.com/golang/go), là ngôn ng? d?ng biên d?ch. Go có c?ng d?ng phát tri?n l?n và có 1 h? th?ng d? ki?m duy?t code nhu [Openstack](https://review.openstack.org) là [Go review](https://go-review.googlesource.com/q/status:open).

Tru?c khi Go h? tr? [Go module](https://github.com/golang/go/wiki/Modules), giúp cho các l?p trình viên don gi?n hóa d?i v?i các package dependency, thì khái ni?m GOPATH là khái ni?m mà b?t kì l?p trình viên nào cung ph?i bi?t và xây d?ng code theo c?u trúc c?a nó. V?y GOPATH và GO MODULE là gì?

### GOPATH là gì?

GOPATH là noi mà ngôn ng? Go s? tìm d?n và d?c các module bên trong dó cho project.

GOPATH ch?a các thu m?c du?i dây:
```
* $GOPATH/src - Ch?a t?t c? mã ngu?n Go c?a b?n và thu m?c mã ngu?n 3rd party.
* $GOPATH/pkg - Ch?a các gói.
* $GOPATH/bin - Ch?a các file nh? phân.
```
![Go Items](/static/img/go-series/thu-muc.png)

### GO MODULE là gì?

Ðây là m?t bu?c ti?n quan tr?ng c?a ngôn ng? Go, vi?c qu?n lý Packages s? don gi?n hon, t? phiên b?n [Go v1.11](https://golang.org/doc/go1.11) thì project không còn ph? thu?c vào vi?c khai báo GOPATH n?a. tuy nhiên, GOPATH v?n h? tr? song song ? phiên b?n này.

## Chu?n b? và cài d?t:

Trong bài này s? s? d?ng môi tru?ng Linux (ubuntu 16.04) d? làm môi tru?ng cho l?p trình Go. Các môi tru?ng khác nhu MacOS hay Windows thì m?i ngu?i có th? t?i t? [Following link](https://golang.org/dl/). Khuy?n khích s? d?ng Linux ho?c MacOS cho vi?c dev Go.

T?i và cài d?t phiên b?n 1.11.2 (dã h? tr? cho 1.11.5):
```bash
$ cd /tmp
$ curl -O https://storage.googleapis.com/golang/go1.11.2.linux-amd64.tar.gz
$ tar -xvf go1.11.2.linux-amd64.tar.gz
$ sudo cp -r go /usr/local
$ export GOROOT=/usr/local/go
$ mkdir $HOME/go
$ export GOPATH=$HOME/go
$ export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
$ export PATH=$PATH:/usr/local/go/bin
```
T?o m?t du?ng d?n default b?ng cách dua các tham s? GOROOT, GOPATH và PATH vào ``~/.bashrc`` d? kh?i t?o cùng ``terminal``.

Ki?m tra l?i version c?a Go:
```bash
$ go version
```


Note:
```
* M?i m?t project nên t?o 1 folder m?i và tr? GOPATH vào folder dó.
* N?u dùng GOLANG thì có th? thay d?i trong ``File/Setting/Go/`` và uncheck ``Use GOPATH that's defined in system environment``.
* N?u dùng CLI thì ``export GOPATH=<du?ng_d?n_c?a_folder>``.
* S? d?ng ``go get <du?ng_d?n_package>`` d? cài package dependency. e.g: `go get -t golang.org/x/oauth2/...`
```

V?i Go module thì chúng ta quên các th? ? trên di và ch? c?n ``go mod init <name>`` là xong package dependency và s? t?o ra m?t file là <name>.mod. Ð? build ?ng d?ng thì ch?y ``go build <name>`` và ``./<name>`` d? running.
```
$ go mod init <name>
$ go build <name>
$ ./<name>
```

### (Option) Cài d?t và c?u hình công c? jupyer th?n thánh (dã dùng cho python) d? code v?i GO.

Nhu m?i ngu?i dã bi?t, jupyter là m?t công c? m?nh m? d? l?p trình và luu l?i các codes dã du?c ch?y tru?c dó. Du?i dây là hu?ng d?n setup cho jupyter:

```bash
$ sudo apt install python3-pip
$ sudo pip3 install jupyter
$ sudo apt-get install libzmq3-dev pkg-config
$ export GOROOT=/usr/local/go
$ export PATH=$GOROOT/bin:$PATH
$ go get golang.org/x/tools/cmd/goimports
$ go get -tags zmq_4_x github.com/gopherds/gophernotes
$ mkdir -p ~/.local/share/jupyter/kernels/gophernotes
$ cp -r $GOPATH/src/github.com/gopherds/gophernotes/kernel/* ~/.local/share/jupyter/kernels/gophernotes
```

S?a n?i dung file `kernel.json` trong `$HOME/.local/share/jupyter/kernels/gophernotes` theo bên du?i:
```json
{
	"argv": [
		"/home/stack/.go/bin/gophernotes",
		"{connection_file}"
	],
	"display_name": "Go",
	"language": "go",
	"name": "go"
}
```

Và ch?y ``jupyter notebook`` t? Host d? b?t server.

K?t qu? d?t du?c nhu sau:
![jupyter go](/static/img/go-series/jupyter.png)

T?m k?t ph?n môi tru?ng ? dây và d?i ph?n 2!


Author: Nguy?n Van Trung

Reference:
* https://www.melvinvivas.com/go-version-1-11-modules/
* https://www.digitalocean.com/community/tutorials/how-to-install-go-1-6-on-ubuntu-16-04
