---
title: Go Series - Preparing environment for Go
tags: [go]
last_updated: January 01, 2019
sidebar: mydoc_sidebar
permalink: preparing-env-for-go.html
folder: mydoc
---

## Sơ lược về ngôn ngữ GO:

Go là một ngôn ngữ lập trình mã nguồn mở [Open Source](https://github.com/golang/go), là ngôn ngữ dạng biên dịch. Go có cộng đồng phát triển lớn và có 1 hệ thống để kiểm duyệt code như [Openstack](https://review.openstack.org) là [Go review](https://go-review.googlesource.com/q/status:open).

Trước khi Go hỗ trợ [Go module](https://github.com/golang/go/wiki/Modules), giúp cho các lập trình viên đơn giản hóa đối với các package dependency, thì khái niệm GOPATH là khái niệm mà bất kì lập trình viên nào cũng phải biết và xây dựng code theo cấu trúc của nó. Vậy GOPATH và GO MODULE là gì?

### GOPATH là gì?

GOPATH là nơi mà ngôn ngữ Go sẽ tìm đến và đọc các module bên trong đó cho project.

GOPATH chứa các thư mục dưới đây:
```
* $GOPATH/src - Chứa tất cả mã nguồn Go của bạn và thư mục mã nguồn 3rd party.
* $GOPATH/pkg - Chứa các gói.
* $GOPATH/bin - Chứa các file nhị phân.
```
![Go Items](/static/img/go-series/thu-muc.png)

### GO MODULE là gì?

Đây là một bước tiến quan trọng của ngôn ngữ Go, việc quản lý Packages sẽ đơn giản hơn, từ phiên bản [Go v1.11](https://golang.org/doc/go1.11) thì project không còn phụ thuộc vào việc khai báo GOPATH nữa. tuy nhiên, GOPATH vẫn hỗ trợ song song ở phiên bản này.

## Chuẩn bị và cài đặt:

Trong bài này sẽ sử dụng môi trường Linux (ubuntu 16.04) để làm môi trường cho lập trình Go. Các môi trường khác như MacOS hay Windows thì mọi người có thể tải từ [Following link](https://golang.org/dl/). Khuyến khích sử dụng Linux hoặc MacOS cho việc dev Go.

Tải và cài đặt phiên bản 1.11.2 (đã hỗ trợ cho 1.11.5):
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
Tạo một đường dẫn default bằng cách đưa các tham số GOROOT, GOPATH và PATH vào ``~/.bashrc`` để khởi tạo cùng ``terminal``.

Kiểm tra lại version của Go:
```bash
$ go version
```


Note:
```
* Mỗi một project nên tạo 1 folder mới và trỏ GOPATH vào folder đó.
* Nếu dùng GOLANG thì có thể thay đổi trong ``File/Setting/Go/`` và uncheck ``Use GOPATH that's defined in system environment``.
* Nếu dùng CLI thì ``export GOPATH=<đường_dẫn_của_folder>``.
* Sử dụng ``go get <đường_dẫn_package>`` để cài package dependency. e.g: `go get -t golang.org/x/oauth2/...`
```

Với Go module thì chúng ta quên các thứ ở trên đi và chỉ cần ``go mod init <name>`` là xong package dependency và sẽ tạo ra một file là <name>.mod. Để build ứng dụng thì chạy ``go build <name>`` và ``./<name>`` để running.
```
$ go mod init <name>
$ go build <name>
$ ./<name>
```

### (Option) Cài đặt và cấu hình công cụ jupyer thần thánh (đã dùng cho python) để code với GO.

Như mọi người đã biết, jupyter là một công cụ mạnh mẽ để lập trình và lưu lại các codes đã được chạy trước đó. Dưới đây là hướng dẫn setup cho jupyter:

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

Sửa nội dung file `kernel.json` trong `$HOME/.local/share/jupyter/kernels/gophernotes` theo bên dưới:
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

Và chạy ``jupyter notebook`` từ Host để bật server.

Kết quả đạt được như sau:
![jupyter go](/static/img/go-series/jupyter.png)

Tạm kết phần môi trường ở đây và đợi phần 2!


Author: Nguyễn Văn Trung

Reference:
* https://www.melvinvivas.com/go-version-1-11-modules/
* https://www.digitalocean.com/community/tutorials/how-to-install-go-1-6-on-ubuntu-16-04
