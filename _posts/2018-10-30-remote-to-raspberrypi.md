---
title: Xác định địa chỉ IP và remote đến Raspberry Pi
tags: [raspberry, networking, tutorial]
keywords: raspberry, networking, tutorial
last_updated: October 30, 2018
sidebar: mydoc_sidebar
permalink: remote-to-raspberrypi.html
folder: mydoc
---

Sau khi cài đặt hệ điều hành [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) cho chiếc Raspberry Pi, ta có thể dùng một màn hình rời kết nối với nó qua cáp HDMI và cùng với bộ bàn phím, chuột, ta đã có một bộ *"mini PC"* sử dụng được cho mục đích giải trí, học tập và lập trình.  

Một cách khác hiệu quả hơn để có thể làm việc với chiếc Raspberry Pi là **remote** đến nó thông qua giao thức [SSH (Secure Shell)](https://en.wikipedia.org/wiki/Secure_Shell).    

Muốn *ssh* đến Pi, ta cần phải biết: `hostname` và `địa chỉ IP` của nó. Mặc định sau khi cài Raspbian, Pi sẽ có `hostname` là **pi**.

Khi máy tính của bạn và Pi ở trong cùng một mạng LAN, dùng lệnh [`nmap`](https://nmap.org) quét toàn bộ subnet (mạng con) để xác định IP của các thiết bị khác trong subnet.  

Giả sử subnet có địa chỉ: `192.168.2.0/24`, chạy lệnh:
```sh
nmap -sN 192.168.2.*
```
Kết quả trả về:
```
Starting Nmap 7.12 ( https://nmap.org ) at 2018-10-30 19:43 +07
Nmap scan report for 192.168.2.1
Host is up (0.033s latency).
All 1000 scanned ports on 192.168.2.1 are open|filtered
MAC Address: X4:DE:XF:EX:X9:6X (Tp-link Technologies)

Nmap scan report for 192.168.2.102
Host is up (0.011s latency).
All 1000 scanned ports on 192.168.2.102 are open|filtered (828) or closed (172)
MAC Address: X4:X2:XF:X1:X5:XF (raspberrypi)

Nmap scan report for 192.168.2.101
Host is up (0.000067s latency).
All 1000 scanned ports on 192.168.2.101 are closed (500) or open|filtered (500)

Nmap done: 256 IP addresses (3 hosts up) scanned in 16.01 seconds
```
Dễ thấy địa chỉ IP của Raspberry Pi là `192.168.2.102`, ssh đến nó theo cú pháp: `ssh hostname@IP-address`

```sh
ssh pi@192.168.2.102
```
