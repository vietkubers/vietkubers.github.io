---
layout: post
title: Linux Network Namespace
date: 2018-09-11
categories: [linux, networking]
tags: [Linux, Networking, Namespace]
---
**Những nội dung chính:**

<!-- MarkdownTOC -->
[1. Network namespace là gì?](#-what-is-network-namespace)  
[2. Làm việc với network namespace](#-working-with-network-namespace)  
[3. Ping 2 namespaces](#-ping-between-2-namespaces)  
[4. Tổng hợp những dòng lệnh đã dùng trong bài viết](#-commands-refers)  
<!-- /MarkdownTOC -->

<a name="-what-is-network-namespace"><a/>
## 1. Network namespace là gì?

Network namespace giúp chúng ta có các mạng riêng biệt trên một host.

Mỗi một namespace sẽ có những giao diện (***interface***) và bảng định tuyến (***routing table***) của riêng nó và tách biệt với các namespace khác. Ngoài ra, tiến trình (***process***) trên hệ thống có thể được liên kết với một network-namespace cụ thể.

Network namespace được sử dụng nhiều trong các dự án OpenStack và Docker. Để có thể hiểu sâu được những dự án đó, bạn phải sử dụng thông thạo network-namespace.


<a name="-working-with-network-namespace"><a/>
## 2. Làm việc với network namespace

Khi khởi động Linux, hệ thống sẽ có một namespace và mỗi tiến trình (***process***) khi được tạo mới sẽ kế thừa (***inherit***) namespace này từ tiến trình cha ([***parent process***](https://en.wikipedia.org/wiki/Parent_process)). Vì vậy, tất cả các tiến trình đều kế thừa network-namespace của tiến trình ***init*** (PID=1).

![Default namespace](/static/img/network-namespace/default-namespace.png)

### Danh sách các namespace

Để làm việc với network-namespace, sử dụng lệnh `ip netns`

Liệt kê các network-namespace trên hệ thống:
```bash
$ ip netns list

qrouter-68d092ba-2509-462c-87fb-2db5e9678eb5  
qdhcp-5dfc8455-bb8d-46af-a815-29bcc74a4640
``` 

### Tạo mới một namespace

Để tạo mới một namespace:
```bash
$ ip netns add truongnh1
$ ip netns add truongnh2
```
2 namespace được tạo mới: **truongnh1** & **truongnh2**

![Add namespace](/static/img/network-namespace/add-namespace.png)

Như hình vẽ trên, 2 namespace mới đã được tạo độc lập với namespace mặc định của hệ thống.

Một lần nữa liệt kê các network-namespace:
```bash
$ ip netns list

truongnh2  
truongnh1  
qrouter-68d092ba-2509-462c-87fb-2db5e9678eb5
qdhcp-5dfc8455-bb8d-46af-a815-29bcc74a4640
```
Mỗi khi một namespace được tạo mới có một file tương ứng có cùng tên với tên của namespace tạo ra trong thư mục `/var/run/netns`.
Lúc này:
```bash
$ ls /var/run/netns  

qdhcp-5dfc8455-bb8d-46af-a815-29bcc74a4640
qrouter-68d092ba-2509-462c-87fb-2db5e9678eb5
truongnh1
truongnh2
```

### Thực thi lệnh (commands) trong namespaces

Thực thi một lệnh trong một namespace:
```bash
$ sudo ip netns exec truongnh1 ip a

1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

Ví dụ ở trên, lệnh `ip a` chạy trong namespace có tên **truongnh1**.

<a name="-ping-between-2-namespaces"><a/>
## 3. Ping 2 namespaces

Một kịch bản đơn giản được đưa ra:  
- Mô phỏng 2 node (mỗi namespace sẽ đại diện cho 1 node)
- Kết nối 2 namespace tới một switch ảo (**virtual switch**)
- ping từ namespace này đến namespace kia

### Tạo virtual switch

Cài đặt Open vSwitch và khởi động service của nó
```bash
$ sudo apt-get install openvswitch-switch  
$ sudo /etc/init.d/openvswitch-switch start
```

Tiếp theo, tạo 1 virtual switch trong namespace mặc định của hệ thống có tên là **my_switch**.  
```bash
$ sudo ovs-vsctl add-br my_switch
```

Kiểm tra xem **my_switch** đã được tạo thành công hay chưa?  
```bash
$ sudo ovs-vsctl show

4e279dd7-5211-4f79-a8bc-76422c831e0b
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
    Bridge br-ex
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        Port br-ex
            Interface br-ex
                type: internal
        Port phy-br-ex
            Interface phy-br-ex
                type: patch
                options: {peer=int-br-ex}
    Bridge my_switch
        Port my_switch
            Interface my_switch
                type: internal

```

Để kết nối namespace đến **my_switch** vừa được tạo, dùng cặp ***veth*** - sẽ được giái thích sơ lược như dưới đây:

### veth (virtual ethernet)

***veth*** là một loại thiết bị mạng mà luôn luôn đi theo 1 cặp (pairs). Có thể tưởng tượng rằng, từ ***pair*** nhắc đến ở đây như là một cái ống: mọi thứ gửi đến một đầu ống sẽ đi ra ngoài ở đầu bên kia.

```bash
$ sudo ip link add type veth

$ ip address

14: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 76:75:43:9f:a8:9e brd ff:ff:ff:ff:ff:ff
15: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether fa:92:ed:48:bd:b4 brd ff:ff:ff:ff:ff:ff
```

Có 2 device đi theo một cặp: mọi thứ khi được gửi đến **veth0@veth1** sẽ đi ra ở **veth1@veth0**.

Bây giờ, khi xóa 1 trong 2 **veth** ở trên thì device còn lại cũng sẽ bị xóa theo.
```bash
$ sudo ip link del veth0
```

### veth

Quay trở lại việc kết nối namespace với **my_switch**, tạo **veth** kết nối namespace **truongnh1** với **my_switch**
```bash
$ sudo ip link add truongnh1-netns type veth peer name truongnh1-ovs 
```

Câu lệnh trên đã xác định tên của mỗi thành phần trong cặp. Vì vậy, **truongnh1-netns** sẽ được thiết lập ở trong namespace **truongnh1** và **truongnh1-ovs** sẽ kết nối với virtual switch.
```bash
$ ip a

16: truongnh1-ovs@truongnh1-netns: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether d2:e3:de:1d:22:e5 brd ff:ff:ff:ff:ff:ff
17: truongnh1-netns@truongnh1-ovs: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 92:fb:c5:a8:6a:df brd ff:ff:ff:ff:ff:ff
```

Đặt **truongnh1-netns** vào trong namespace **truongnh1**:
```bash
$ sudo ip link set truongnh1-netns netns truongnh1
```

**Nhấn mạnh rằng: namespace là một môi trường mạng độc lập và namespace mặc định của hệ thống là tách biệt với các namespace khác**. Nếu điều này là đúng, **truongnh1-netns** sẽ không được thấy trong namespace mặc định của hệ thống kể từ khi nó được đặt trong namespace **truongnh1**.
```bash
$ ip a

13: my_switch: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1
    link/ether a2:71:a7:0e:62:40 brd ff:ff:ff:ff:ff:ff
16: truongnh1-ovs@if17: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether d2:e3:de:1d:22:e5 brd ff:ff:ff:ff:ff:ff link-netnsid 0

$ sudo ip netns exec truongnh1 ip a

1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
17: truongnh1-netns@if16: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 92:fb:c5:a8:6a:df brd ff:ff:ff:ff:ff:ff link-netnsid 0
```
Như đã thấy, **truongnh1-netns** hiện tại đã ở trong namespace **truongnh1**.

Kết nối veth còn lại trong cặp (**truongnh1-ovs**) với **my_switch**

```bash
$ sudo ovs-vsctl add-port my_switch truongnh1-ovs
$ sudo ovs-vsctl show

4e279dd7-5211-4f79-a8bc-76422c831e0b
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
    Bridge my_switch
        Port "truongnh1-ovs"
            Interface "truongnh1-ovs"
        Port my_switch
            Interface my_switch
                type: internal
```

**truongnh1-ovs** đã trở thành 1 port của **my_switch**.

Làm các bước tương tự với namespace **truongnh2**
```bash
$ sudo ip link add truongnh2-netns type veth peer name truongnh2-ovs
$ sudo ip link set truongnh2-netns netns truongnh2
$ sudo ovs-vsctl add-port my_switch truongnh2-ovs
```

Đến đây, mỗi namespace đã có một cặp **veth** vì thế **my_switch** có 2 port: **truongnh1-ovs** và **truongnh2-ovs**

![Create veth](/static/img/network-namespace/veth.png)

Điều này có nghĩa là 2 namespace **truongnh1** và **truongnh2** đã có thể "thông nhau"? Câu trả lời là KHÔNG. Việc cần làm tiếp theo là gán địa chỉ cho nó và mang nó lên (bring devices up).

### Bringing up devices

Trong namespace mặc định của hệ thống:
```bash
$ sudo ip link set truongnh1-ovs up
$ sudo ip link set truongnh2-ovs up
```

 Trong namespace truongnh1 và truongnh2:
 ```bash
$ sudo ip netns exec truongnh1 ip link set dev lo up
$ sudo ip netns exec truongnh1 ip link set dev truongnh1-netns up
$ sudo ip netns exec truongnh2 ip link set dev lo up
$ sudo ip netns exec truongnh2 ip link set dev truongnh2-netns up
```

### Gán địa chỉ IP

Bước tiếp theo sẽ gán địa chỉ IP cho các device truongnh1-netns và truongnh2-netns trong các namespace truongnh1, truongnh2 tương ứng.
```bash
$ sudo ip netns exec truongnh1 ip addr add 10.0.0.1/24 dev truongnh1-netns
$ sudo ip netns exec truongnh2 ip addr add 10.0.0.2/24 dev truongnh2-netns
```

```bash
$ sudo ip netns exec truongnh1 ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
17: truongnh1-netns@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 92:fb:c5:a8:6a:df brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.0.0.1/24 scope global truongnh1-netns
       valid_lft forever preferred_lft forever
    inet6 fe80::90fb:c5ff:fea8:6adf/64 scope link 
       valid_lft forever preferred_lft forever

```

### Ping test

Các namespace truongnh1 và truongnh2 đã được nối với my_switch, các device được bật và gán địa chỉ IP. Thực hiện lệnh `ping` từ truongnh1 đến truongnh2:

```bash
$ sudo ip netns exec truongnh1 ping 10.0.0.2

PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.680 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.075 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.089 ms
64 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=0.079 ms
^C
--- 10.0.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2997ms
rtt min/avg/max/mdev = 0.075/0.230/0.680/0.260 ms
```

<a name="-commands-refers"><a/>
## 4. Tổng hợp những dòng lệnh đã dùng trong bài viết
  
```bash
# List all namespace, except for the default/global one.
$ ip netns

# Execute command inside a specific namespace                                    
$ ip netns exec <namespace_name> <command>
 
# Sets specified interface in the specified namespace 
$ ip link set <interface> netns <namespace>
 
# Bring interface up               
$ ip link set <interface> up       
 
# Add veth pair                         
$ ip link add type veth
 
# Add veth pair by specifying the name for each peer
$ ip link add <veth-peer1> type veth peer name <veth-peer2>
 
# Adds a new bridge
$ ovs-vsctl add-br <bridge_name>
 
# Adds a new port in the specific bridge
$ ovs-vsctl add-port <bridge_name> <port_name>
```

*Bài viết được dịch sang Tiếng Việt từ bài gốc tại: http://abregman.com*.
