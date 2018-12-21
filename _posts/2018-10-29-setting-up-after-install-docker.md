---
title: Các bước sau khi cài đặt Docker trên hệ thống Linux
tags: [news]
last_updated: October 29, 2018
sidebar: mydoc_sidebar
permalink: setting-up-after-install-docker.html
folder: mydoc
---

Bài viết này sẽ trình bày các cài đặt hệ thống Linux để làm việc tốt hơn với [Docker](https://www.docker.com/).

## 1. Quản lý Docker với một người dùng non-root

Docker daemon gắn với một [UNIX socket](https://en.wikipedia.org/wiki/Unix_domain_socket) thay vì một cổng [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol). Mặc định, một Unix socket được sở hữu bởi người dùng `root` và những người dùng khác trên hệ thống chỉ có thể truy cập nó bằng cách dùng lệnh `sudo`. Docker daemon luôn chạy như người dùng `root`.  

Nếu bạn không muốn phải chạy lệnh `docker` với `sudo`, hãy tạo một nhóm người dùng (group) có tên là `docker` và thêm người dùng hiện tại vào nhóm đó. Khi Docker daemon khởi động, nó sẽ tạo một Unix socket được truy cập bởi các thành viên của nhóm người dùng `docker`.  

{% include note.html content="Nhóm người dùng `docker` cấp các quyền tương đương với người dùng `root`." %}

* Tạo nhóm người dùng `docker`
```sh
$ sudo groupadd docker
```
* Thêm người dùng vào nhóm `docker`
```sh
$ sudo usermod -aG docker $USER
```
* Đăng xuất và đăng nhập lại phiên của người dùng hiện tại, kiểm tra xem người dùng hiện tại đã được thêm vào nhóm `docker` hay chưa?
```sh
$ groups
```
* Chạy `docker` mà không cần dùng `sudo`
```sh
$ docker run -it ubuntu bash
```
Dòng lệnh trên sẽ download một test image về và chạy nó trong một container. Done:)

## 2. Cấu hình Docker để khởi động cùng với hệ thống

Hầu hết các bản phần phối Linux (RHEL, CentOS, Fedora, Ubuntu 16.04 hoặc mới hơn) dùng `systemd` để quản lý các service khởi động cùng với hệ thống. Bản Ubuntu 14.10 hoặc cũ hơn dùng `upstart`.

* Dùng `systemd`:

```sh
$ sudo systemctl enable docker

Synchronizing state of docker.service with SysV init with /lib/systemd/systemd-sysv-install...
Executing /lib/systemd/systemd-sysv-install enable docker
```

Để vô hiệu quá docker:
```sh
$ sudo systemctl disable docker
```

* Dùng `upstart`:

Docker được tự động cấu hình để khởi động cùng với hệ thống dùng `upstart`. Để vô hiệu hóa, ta dùng lệnh sau đây:
```sh
$ echo manual | sudo tee /etc/init/docker.override
```

## 3. Cấu hình nơi Docker daemon lắng nghe các kết nối

Mặc định, Docker daemon lắng nghe các kết nối trên một Unix socket để chấp nhận các yêu cầu (requests) từ các máy khách cụ bộ (local clients). Có thể cho phép Docker chấp nhận các yêu cầu từ các máy chủ từ xa (remote hosts) bằng cách cấu hình nó để lắng nghe trên một địa chỉ IP và cổng (port) cũng nhưng là một Unix socket. Ở đây có thể hiểu rằng: `Unix socket = IP + Port`.

### 3.1. Cấu hình truy cập từ xa với `systemd`

* Mở file `docker.service` bằng lệnh:
```sh
$ sudo systemctl edit docker.service
```
* Thêm hoặc chỉnh sửa nội dung dưới đây vào file
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375
```
* Reload lại cấu hình `systemctl`
```sh
$ sudo systemctl daemon-reload
```
* Khởi động lại Docker
```sh
$ sudo systemctl restart docker.service
```
* Kiểm tra lại thay đổi đã được áp dụng chưa bằng cách xem đầu ra của `netstat` để xác nhận rằng `dockerd` đang được lắng nghe trên cổng đã được cấu hình.

```sh
$ sudo netstat -lntp | grep dockerd

tcp        0      0 127.0.0.1:2375          0.0.0.0:*               LISTEN      1196/dockerd
``` 

### 3.2. Cấu hình truy cập từ xa với `daemon.json`

* Thiết lập mảng `hosts` trong file `/etc/docker/daemon.json` để kết nối đến Unix socket và địa chỉ IP
```sh
$ sudo vi /etc/docker/daemon.json
```
* Thêm vào nội dung sau vào file `/etc/docker/daemon.json`
```
{
"hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"]
}
```
* Khởi động lại Docker
```sh
$ sudo systemctl restart docker.service
```
* Kiểm tra lại thay đổi đã được áp dụng chưa bằng cách xem đầu ra của `netstat` để xác nhận rằng `dockerd` đang được lắng nghe trên cổng đã được cấu hình.

```sh
$ sudo netstat -lntp | grep dockerd

tcp        0      0 127.0.0.1:2375          0.0.0.0:*               LISTEN      1196/dockerd
``` 

*Author: [truongnh1992](https://github.com/truongnh1992)*