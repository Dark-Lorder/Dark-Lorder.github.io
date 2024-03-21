---
title: Docker网络
date: 2022-09-08 00:00:00
categories:
- Docker
tags:
- Docker
---
Docker网络架构主要由三部分组成：CNM，Libnetwork和Driver。
<!-- more -->



## Docker的默认网络类型
```
root@dk:~# 
root@dk:~# docker info -f '{{.Plugins.Network}}'   
[bridge host ipvlan macvlan null overlay]
root@dk:~# 
```
- bridge（桥接）
- host（主机）
- ipvlan（三层模式划分vlan）
- macvlan（二层模式划分vlan）
- null（躺平：不与别人交互）
- overlay（叠加网络）

## Docker的网络模式
安装Docker时，它会自动创建三个网络，bridge（创建容器默认连接到此网络）、 none 、host
```
root@dk:~# 
root@dk:~# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
b44c04c3ee09   bridge    bridge    local
231d383cae90   host      host      local
432ab6b3b127   none      null      local
root@dk:~# 
```

| 网络模式   | 简介                                                         |
| ---------- | ------------------------------------------------------------ |
| Host       | 容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。 |
| Bridge     | 此模式会为每一个容器分配、设置IP等，并将容器连接到一个docker0虚拟网桥，通过docker0网桥以及Iptables nat表配置与宿主机通信。 |
| None       | 该模式关闭了容器的网络功能。                                 |
| Container  | 创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围。 |


<img src="https://img.darklorder.com/img/202308251837508.png"/>

- Closed container：只有loop接口，就是null类型

- Bridged container A：桥接式类型，容器网络接入到docker0网络上

- joined container A：联盟式网络，让两个容器有一部分名称空间隔离（User、Mount、Pid），这样两个容器间就拥有同一个网络接口，网络协议栈

- Open container：开放式网络：直接共享物理机的三个名称空间（UTS、IPC、Net），世界使用物理主机的网卡通信，赋予容器管理物理主机网络的特权


## 容器网络第一个标准CNM

围绕Docker生态，目前有两种主流的网络接口方案，即Docker主导的Container Network Model（CNM）和Kubernetes社区主推的Container Network Interface（CNI），其中CNM相对CNI较早提出。不管是CNM还是CNI，其最终目标都是以一致的编程接口，抽象网络实现。**CNI和CNM并非是完全不可调和的两个模型**，二者是可以进行转化的。

| CNM阵营                                                                               | CNI阵营                                                      |
|-------------------------------------------------------------------------------------| ------------------------------------------------------------ |
| Docker Libnetwork的优势就是Docker原生与Docker容器生命周期结合紧密，缺点是与Docker耦合度过高                     | CNI的优势是兼容其他容器技术（例如rkt）及上层编排系统 （Kubernetes&Mesos），而且社区活跃势头迅猛，再加上Kubernetes主推 |
| Docker Swarm overlay <br/>Macvlan & IP network <br/>drivers <br/>Calico <br/>Contiv | Weave <br/>Macvlan <br/>flannel <br/>Calico <br/>Contiv <br/>Mesos CNI                |

在顶层设计中，Docker 网络架构由 3 个主要部分构成：CNM、Libnetwork 和驱动。

- CNM： 是设计标准。在 CNM 中，规定了 Docker 网络架构的基础组成要素。
- Libnetwork： 是 CNM 的具体实现，并且被 Docker 采用，Libnetwork 通过 Go 语言编写，并实现了 CNM 中列举的核心组件。
- 驱动：通过实现特定网络拓扑的方式来拓展该模型的能力。


##  容器网络模型（CNM）

在最初的版本中，Docker的网络功能集成在Docker Daemon的代码中，这使得整体架构变得臃肿且缺乏灵活性，无法适应复杂的网络需求。为此，Docker公司在后面提出了CNM（Container Network Model，可译为容器网络模型）规范，并将网络功能独立出来作为一个组件，即Libnetwork网络库。

Libnetwork的设计遵循着CNM规范，在该规范中包含着三个重要概念：Sandbox（沙盒）、Endpoint（端点）和 Network（网络）。

**1\. Sandbox（沙盒）**

Sandbox可以看成是在容器中独立的网络空间，在里面包含了容器的网络栈配置，包括网络接口、路由表和DNS设置等。Sandbox的标准实现基于Linux中的Network Namespace特性。一个Sandbox可以包含多个Endpoint，并且连接到不同的网络中。

**2. Endpoint（端点）**

Endpoint的作用在于将容器连接到网络，Endpoint通常由一对Veth Pair（成对出现的一种虚拟网络设备接口）组成，其中一端在Sandbox中，另一端连接到网络中。

**3.Network（网络）**

可以连接多个Endpoint的一个子网。

**CNM示例图：**

![图片](https://img.darklorder.com/img/202308282232623.png)

如图所示，三个容器中都有独立的Sandbox，而每个Sandbox中包含自己的Endpoint，并且通过Endpoint连接到网络。可以看到第二个容器有两个Endpoint，这使得其可以同时连接两个不同的网络。而容器1和容器3由于处于不同的网络中，彼此之间将无法直接通信 。

使用CNM规范的好处，在于容器可以不用关心网络的具体实现，只要能提供网络接入点给到容器接入即可。而对于网络的具体实现，则交由驱动来完成。这种方式解耦了容器与网络，容器可以接入不同类型的网络，而第三方只要按照CNM规范开发驱动，即可保证容器的无缝接入，架构的灵活性得到很大的提升。

##  容器网络

在安装完Docker后会自动创建三个网络，我们通过 docker network ls 命令可查看相关的网络信息。

```
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
ad87b0b5afaa   bridge    bridge    local
19bd38b4d728   host      host      local
a83dd07d2ba1   none      null      local
```

Docker默认创建的网络类型有bridge、host和null，在启动容器时，可以通过--network 选项指定使用的网络。如果不指定，则默认会使用bridge网络。

**1\. bridge网络**

当启动docker时，会自动生成一个名为docker0的Linux bridge(网桥)。bridge可以看成是一个软件交换机，负责对挂载在它上面的接口进行包转发。

```
$ ifconfig docker0
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:8bff:fe36:8c00  prefixlen 64  scopeid 0x20<link>
        ether 02:42:8b:36:8c:00  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5  bytes 446 (446.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

运行brctl show 命令查看网桥信息，可以看到在没有启动任何容器的情况下，docker0的 "interfaces" 处为空，表明该网桥目前未挂载任何接口。

```
$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.02425cf9d61c       no
```

现在，我们启动一个容器后，再次来查看网桥情况。可以看到，当前网桥已经挂载了一个接口，名称为veth1ec2b44。

```
$ docker run -d --name nginx -p 80:80 nginx:1.20-alpine
07a46c03d17b40545a090dab60eac9a75bfa8050c572ae2c330ca98700ce68d5

$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.02428b368c00       no              veth1ec2b44
```

我们查看一下容器里面的网络配置，可以看到容器的网卡名称为eth0@if11，它与挂载到网桥的veth1ec2b44接口即是一对Veth Pair。

```
$ docker exec nginx ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
10: eth0@if11: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```


前面我们说过，Veth Pair是成对的网络设备，此时的容器网络架构如下所示：

![图片](https://img.darklorder.com/img/202308282236353.png)

Veth Pair就如一根网线的两端，其中一端收到的网络包会从另一端出去，这使得连接到同一个网桥的容器可以彼此通信，在容器与网桥间共同组成了一个虚拟局域网。


![图片](https://img.darklorder.com/img/202308282232733.png)

当bridge网络被创建时，系统会为其分配一个子网段，用于容器使用。当容器接入时，会从IP池中分配IP给到容器网卡，而容器的网关则会指向bridge，也即是docker0的IP地址。

**2.  host网络** 

在启动容器时使用--network host指定为本机网络，则容器将与宿主机共享网络栈，此时容器会使用主机的IP以及其他网络配置。

如下：

```
$ docker run -d --network host --name nginx2 nginx:1.20-alpine
6f2023e6e714f9bb212bca9aec12dd7c3befd51a01d28216cea12c64136f6924

$ docker exec -it nginx2 ip addr            
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:7b:a8:71 brd ff:ff:ff:ff:ff:ff
    inet 192.168.214.112/24 brd 192.168.214.255 scope global dynamic noprefixroute ens33
       valid_lft 2592730sec preferred_lft 2592730sec
    inet6 fe80::f992:e993:bd8e:2de6/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
......
```

使用host网络时，对容器做端口映射将会无效， 容器会忽略 -p 所指定的端口，而直接在宿主机网络层面开启对应端口。例如，容器中如果开放了80端口，那么此时宿主机也将开放同样端口。

host网络相比bridge网络具有更高的性能，同时在容器有大量端口要开放的时候 ，也会省事很多。但同时也增加了不安全性，此时可在容器内对主机的网络栈进行操作，所以并不推荐使用。

**3. null网络**

null网络会禁用容器的网络栈，使得容器与外部隔离。在启动时通过 --network none 实现。此时，容器除lo外，将不再有其他网卡，完全与外部网络隔离。

```
$ docker run -d --network none --name nginx3 nginx:1.20-alpine
513e0ce37dab8dd6cb0b15fe0311af5ca7a050378ebcdb3b2c20d5a6bc39c2b5

$ docker exec -it nginx3 ifconfig
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

这种场景一般使用很少，只有在某些不需要与外界联网，并且对安全要求非常高的程序才有可能用到。

##   外部网络访问 

docker0本身是作为宿主机的一个本地接口，因此，容器默认情况下可以访问到宿主机自身的网络。但当容器需要访问外部网络时，则需要宿主机做一层NAT转发。

在安装docker时，默认会启用Linux的包转发功能，如下：

```
$ sudo sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```

查看宿主机iptable的nat表上面的POSTROUTING链规则，可看到有一条转发规则。

```
$ sudo iptables -t nat -nvL POSTROUTING
Chain POSTROUTING (policy ACCEPT 71 packets, 5396 bytes)
 pkts bytes target       prot  opt   in   out        source            destination         
 0     0    MASQUERADE    all   --   *  !docker0   172.17.0.0/16      0.0.0.0/0
```

该规则的作用是当docker0收到来自容器网段（172.17.0.0/16)的网络包时，将其交给MASQUERADE处理。MASQUERADE与传统的SNAT相似，会将网络包的源IP替换为网卡IP，这样可以保证容器的网络包能够正常外出。

那么，外部网络又是如何访问容器的呢？在前面的学习中，我们知道启动容器时可以使用 -p 的方式，将容器的端口映射出来。在这个过程中，Docker会在本地添加相应的iptable规则，用于网络包的DNAT转换。

以本示例的容器为例，这里我们开放了80端口。当查看相关的iptable规则时，可看到多了一个关于80端口的DNAT规则。

```
$ docker run -d --name nginx -p 80:80 nginx:1.20-alpine
07a46c03d17b40545a090dab60eac9a75bfa8050c572ae2c330ca98700ce68d5

$ iptables -t nat -nvL  DOCKER
Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
......       
    0     0 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.2:80
```

该规则将访问到主机80端口的网络包，转发到容器地址（172.17.0.2）的80端口，以此实现外部网络对容器的访问。

此时，容器的整体网络架构如下图所示：

![图片](https://img.darklorder.com/img/202308282231667.png)

**附录: Docker网络常用命令**

**1. 创建网络**  

除了Docker默认生成的网络外，用户也可以创建自定义的网络。docker network create命令用于创建新的网络。

示例：此处创建一个名为mynet的bridge网络。

```
$ docker network create -d bridge mynet
592da9b6f8197cb7f11bae42f83f4429d9a97371a4aaccd3701d2998181763e8
```

**2. 列出所有网络**

docker network ls命令用于列出当前所有网络。

示例：

```
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
12bb6c359b1c   bridge    bridge    local
19bd38b4d728   host      host      local
12e7db06ad9b   mynet     bridge    local
a83dd07d2ba1   none      null      local
```

**3. 查看网络详情**

docker network inspect命令可查看一个网络的详细信息，包括接入的容器、网络配置等。

示例：

```
$ docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "12e7db06ad9bf7aaaedb0cb33042ea48170d4ef2026da9964acba1cb984f441b",
        "Created": "2022-05-30T03:40:51.977036849-04:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
......
```

**4. 连接网络**

docker network connect命令用于将容器连接到网络，通过该命令可以更改容器的接入网络。

示例：

```
$ docker network connect mynet nginx
```

**5. 删除网络** 

docker network rm 命令可用于删除指定网络。只有当网络没有容器连接时，才能正常删除。

示例：

```
$ docker network rm mynet
mynet
```



**参考资料**
[《k8s网络指南》Chapter2-Docker网络模型简介-知乎](https://zhuanlan.zhihu.com/p/480034319?utm_id=0)
[Docker容器实战十：容器网络](https://mp.weixin.qq.com/s?__biz=MzU2OTc4NDI2MQ==&mid=2247484536&idx=1&sn=e317e4e07bc74ff8148e1cc9fa91bb74&chksm=fcf826d2cb8fafc474433f7918ad1de6c5937da631ee39d66c476d66b30c1b46c6d23eae8511&scene=21#wechat_redirect)