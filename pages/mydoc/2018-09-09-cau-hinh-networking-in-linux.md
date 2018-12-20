---
title: Cấu hình mạng tĩnh trên Linux
tags: [linux, networking, news]
keywords: linux, networking
last_updated: September 09, 2018
sidebar: mydoc_sidebar
permalink: 2018-09-09-cau-hinh-networking-in-linux.html
---

Khi làm việc trên môi trường GNU/Linux, địa chỉ IP thường được cấu hình một cách tự động bởi hệ điều hành và chúng ta sẽ có một địa chỉ IP động (*dynamic IP address*) được cấp phát cho mỗi card mạng (*NIC - Network Interface Card*).

Nhưng đôi khi vì một mục đích nào đó, ví dụ như việc cài đặt một server RADIUS/server FTP, người quản trị cần thiết lập một địa chỉ IP tĩnh (*static IP address*). Bài viết này sẽ giúp chúng ta giải quyết vấn đề đó.

Để xác định những card mạng trên máy, ta dùng một trong các lệnh sau:

```
ifconfig -a | grep Link
ls /sys/class/net
```

Lệnh sẽ trả về kết quả là danh sách những card mạng của máy:  
`lo   wlan0   eth0`

Tiếp theo sẽ tiến hành cấu hình cho interface eth0 (tuỳ thuộc vào phiên bản hệ điều hành, card mạng này có thể là eth1, enp0s3,...)

Mở file **/etc/network/interfaces** với trình soạn thảo trên máy Linux, có thể dùng vi, gedit hoặc nano để thêm vào nội dung dưới đây:

```bash
auto eth0
iface eth0 inet static
address 192.168.2.102
netmask 255.255.255.0
network 192.168.2.0
broadcast 192.168.2.255
gateway 192.168.2.1
```
![Network Interface](/static/img/network-inteface.jpg)  

Khởi động lại network service của HĐH để cài đặt trên có hiệu lực.  
```
# /etc/init.d/networking restart
```
