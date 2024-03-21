---
title: Docker镜像
date: 2022-09-06 00:00:00
categories:
- Docker
tags:
- Docker
---

<!-- more -->

### Docker Images

运行容器时，基于Mount名称空间建立的隔离文件系统中的内容来自镜像

- Docker镜像是一个特殊的文件系统，它必须包含运行应用程序所需的一切——所有依赖项、配置、脚本、二进制文件等

- 镜像还包含容器的其他配置，例如环境变量、要运行的默认命令、和其他元数据

- 通常，镜像还要定义默认启动的应用


Docker镜像由许多层（Layer）叠加而成

- 依赖于特殊的存储驱动，例如aufs (高级联合文件系统)、devicemapper (centos又丑又笨)、overlay2等
- 
- 尽管每种存储驱动程序实现的管理方式不尽相同，但它们都使用可堆叠的镜像层和写时复制（CoW）策略

<img src="https://img.darklorder.com/img/202111292153842.png" style="zoom:50%;" />

### Docker Image Layer

位于下层的镜像称为父镜像(parent image)，最底层的称为基础镜像(base image)

最上层为“可读写”层，其下的均为“只读”层

<img src="https://img.darklorder.com/img/202111281644358.jpeg" alt="See the source image" style="zoom: 80%;" />


### Docker Registry


Registry是一个无状态、高度可扩展的服务器端应用程序，用于存储和分发Docker images。

<img src="https://img.darklorder.com/img/202111281711188.png"/>

Registry上的镜像存储于“仓库（Repository）”中，仓库可隶属于根名称空间或特定的名称空间

- 每个镜像由“仓库名:标签名”标识

- 也可由镜像的Hash码标识


| Namespaces        | Examples (<namespace>/<name>)                |
| ----------------- | -------------------------------------------- |
| organization      | readhat/kubernetes，google/kubernetes        |
| login (user name) | alice/application，bob/application           |
| role              | devel/database，test/database，prod/database |

Docker Registry中的镜像通常由开发人员制作，而后推送至“公共”或“私有”Registry上保存，供其他人员使用，例如“部署”到生产环境；

<img src="https://img.darklorder.com/img/202111281721319.jpeg" alt="img" style="zoom: 80%;" />



**容器配置变更的几种方式**

- 重新打镜像：Hard Coding（硬编码）

- 存储卷：/etc/nginx/nginx.conf --> /data/nginx/conf/nginx.conf

- 环境变量：entrypoint.sh

- 配置中心：appollo nacoss

### Docker Hub

### 什么是镜像层？

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需要的所有内容，包括代码，运行时（一个程序在运行或者在被执行的依赖）、库，环境变量和配置文件。

每个镜像都由多个镜像层组成。这些镜像层都是只读的，从下往上，以栈的方式组合在一起，组成镜像的根文件系统。

### 什么是容器层？

当容器启动时，一个新的可写层被加载到镜像的顶部。 这一层通常被称作容器层（Container layer），容器层之下的都叫镜像层（Image layers）。所有对容器的改动，无论添加、删除、还是修改文件都只会发生在容器层中。

每个容器运行时都有自己的容器层，保存容器运行相关数据（所有文件变化数据），因为镜像层是只读的，所以多个容器可以共享同一个镜像。删除容器时，Docker Daemon会删除容器层，保留镜像层。

### 各种ID的含义

#### imageID

镜像的唯一标识，其数值根据该镜像的元数据配置文件采用sha256算法的计算获得。
```
[root@iZbp1bum6107bp8mgzkeunZ ~]# docker image inspect nginx | jq ".[0].Id"
"sha256:12766a6745eea133de9fdcd03ff720fa971fdaf21113d4bc72b417c123b15619"
```

#### chainID

Docker内容寻址机制采用的索引ID，docker image inspect <image_id>查到的 RootFS 的 ID 都是 chainID， 每一个镜像的 RootFS 的最低层的 chainID 都可以在/var/lib/docker/image/overlay2/layerdb/sha256/目录下找到对应的文件夹。
/var/lib/docker/image/overlay2/layerdb/sha256/{chainID}目录下可以找到cacheID和diffID
```
[root@iZbp1bum6107bp8mgzkeunZ ~]# docker image inspect nginx | jq ".[0].RootFS"
{
  "Type": "layers",
  "Layers": [
    "sha256:608f3a074261105f129d707e4d9ad3d41b5baa94887f092b7c2857f7274a2fce",
    "sha256:ea207a4854e73eca698e94f323fadb920bbc5fc2af83d4bda9f47fd33fa1a076",
    "sha256:33cf1b723f65c22ccc4660c44fe8b919b75e7bb9ffcfe80432bb75363be34a5b",
    "sha256:5c77d760e1f430188e860d79c2f4efa0f082f6831046e5584810bc5ead42dc5d",
    "sha256:fac199a5a1a59d93dd9b23d4c3445c39070ce0d94f94def585085476f89610cf",
    "sha256:ea4bc0cd4a9350584676b6aa3669984eb614f559229c11dc56a85140b49f0228"
  ]
}

[root@iZbp1bum6107bp8mgzkeunZ ~]# ls /var/lib/docker/image/overlay2/layerdb/sha256/ | grep 608f3a074261105f129d707e4d9ad3d41b5baa94887f092b7c2857f7274a2fce
608f3a074261105f129d707e4d9ad3d41b5baa94887f092b7c2857f7274a2fce

[root@iZbp1bum6107bp8mgzkeunZ ~]# ls /var/lib/docker/image/overlay2/layerdb/sha256/608f3a074261105f129d707e4d9ad3d41b5baa94887f092b7c2857f7274a2fce
cache-id  diff  size  tar-split.json.gz
```

#### cacheID

由宿主机随即生成的一个uuid，存放于/var/lib/docker/image/overlay2/layerdb/sha256/{chainID}/cache-id文件中（容器层不会有 cacheID），每一个 cacheID 都对应着一个镜像层，每一个 cacheID 对应着/var/lib/docker/overlay2/${cache-id}目录
```
[root@iZbp1bum6107bp8mgzkeunZ ~]# cat /var/lib/docker/image/overlay2/layerdb/sha256/085831b1436186cd069fbc921d93064bfe0f119cb99fb41360e20d004824e4a0/cache-id && echo
1710201f3395fe390c5ddcd9a735654da29f2cfa77eb7263a98ed4ac73abe14f
[root@iZbp1bum6107bp8mgzkeunZ ~]# ls /var/lib/docker/overlay2/1710201f3395fe390c5ddcd9a735654da29f2cfa77eb7263a98ed4ac73abe14f
committed  diff  link  lower  work
```

#### diffID

镜像层校验ID，是根据该镜像层的打包文件校验获得。存放于/var/lib/docker/image/overlay2/layerdb/sha256/{chainID}/diff。diffID 采用 SHA256 算法，基于镜像层文件包的内容计算得到。而 chainID 是基于内容存储的索引，它是根据当前层与所有上层镜像层的diffID计算出来的。

#### containerID

容器的唯一标识，每一个容器对应一个/var/lib/docker/containers/{container_id}目录。
```
[root@iZbp1bum6107bp8mgzkeunZ ~]# docker inspect `docker ps | grep nginx-hostname | awk '{print$1}'` | jq ".[0].Id"
"e38891c4d9a6b2ca9048db18e12bd2b05fa3a8d68687ea7acedab288bebf6441"
[root@iZbp1bum6107bp8mgzkeunZ ~]# ls -l /var/lib/docker/containers | grep "e38891c4d9a6b2ca9048db18e12bd2b05fa3a8d68687ea7acedab288bebf6441"
drwx--x--- 4 root root 4096 Nov  1 23:52 e38891c4d9a6b2ca9048db18e12bd2b05fa3a8d68687ea7acedab288bebf6441
```

### 镜像的存储和管理方式

#### 镜像的设计

一个镜像都由多个镜像层组成。

为了区别镜像层，Docker为每个镜像层都计算了UUID，根据镜像层中的数据使用加密哈希算法生成UUID。

所有镜像层和容器层都保存在宿主机的文件系统/var/lib/docker/中，由存储驱动进行管理。

在下载镜像时，Docker Daemon会检查镜像中的镜像层与宿主机文件系统中的镜像层进行对比，如果存在则不下载，只下载不存在的镜像层。

<img src="https://img.darklorder.com/img/202308111424196.png"/>


#### 栈层式管理镜像层

docker 中存储驱动用于管理镜像层和容器层。不同的存储驱动使用不同的算法和管理方式。在容器和镜像管理中，使用的两大技术是**栈式层管理和写时复制。**

Dockerfile中的每一条指令都会对应于Docker镜像中的一层，因此在docker build完毕之后，镜像的总大小将等于每一层镜像的大小总和。

每个镜像都由多个镜像层组成，从下往上以栈的方式组合在一起形成容器的根文件系统，Docker的存储驱动用于管理这些镜像层，对外提供单一的文件系统。Docker 的镜像实际上由一层一层的文件系统组成，这种层级的文件系统叫 UnionFS（联合文件系统）。

>
> **由于联合文件系统的存在，容器文件系统内容的大小不等于Docker镜像大小。**
> 
> <img src="https://img.darklorder.com/img/202308111429367.png"/>
>
> 由于联合文件系统的存在，如果镜像层是相同的，则不同的镜像会共享该层。
> 
> 如上图中，镜像A与镜像B就共享第二层镜像，使得A+B镜像文件大小并不等于A+B镜像占用宿主机存储空间容量的大小。
> 
> doker images命令列出的镜像体积总和并不能代表实际使用的磁盘空间，需要使用docker system df命令来代替。
>


#### UnionFS （联合文件系统）

联合文件系统（UnionFS）是一种**分层**、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。

>
> **分层的原因：**
>
> 分层最大的一个好处就是共享资源
> 
> 有多个镜像都从相同的base镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像；
>
> 同时内存中也只需加载一份base镜像，就可以为所有容器服务了，而且镜像的每一层都可以被共享。
>

联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。


>
> 如图，在下载镜像时，也是一层一层下载的
>
> <img src="https://img.darklorder.com/img/202308111433968.png"/>
>

### 镜像结构

要了解docker的镜像结构，需要先对linux的文件系统结构有所了解。

#### linux文件系统结构

Linux 文件系统由 bootfs和 rootfs 两部分组成。

bootfs(boot file system) 主要包含 bootloader 和 kernel，bootloader 主要是引导加载 kernel(内核)，当 kernel 被加载到内存中后 bootfs 就被 umount (卸载)了。

rootfs (root file system) 包含的就是典型 Linux 系统中的 /dev，/proc，/bin，/etc 等标准目录和文件。rootfs就是各种Linux发行版。比如redcat、centOS。

<img src="https://img.darklorder.com/img/202308111435559.png" style="zoom: 80%;" />

**docker镜像结构**

<img src="https://img.darklorder.com/img/202111281644358.jpeg" alt="See the source image" style="zoom: 80%;" />

docker的分层镜像结构如图所示，镜像的最底层必须是一个启动文件系统（bootfs）的镜像层。bootfs的上层镜像称为根镜像（rootfs）或者基础镜像（Base Image），它一般是操作系统，比如centos、debian或者Ubuntu。

用户的镜像必须构建在基础镜像之上。如图所示， emacs镜像层就是在基础镜像上安装emacs创建出来的镜像，在此基础上安装apache又创建了新的镜像层。利用这个新的镜像层启动的容器里运行的是一个已经安装好emacs和apache的Debian系统。

#### docker镜像分层的理解

查看镜像分层方式可以通过docker image inspect [IMAGE] 命令。其中RootFS部分则是表示了分层信息。

```
[root@iZbp1bum6107bp8mgzkeunZ ~]# docker image inspect redis
[
    {
        "Id": "sha256:53aa81e8adfa939348cd4c846c0ab682b16dc7641714e36bfc57b764f0b947dc",
        ...
        ...
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:ad6562704f3759fb50f0d3de5f80a38f65a85e709b77fd24491253990f30b6be",
                "sha256:49cba0f0997b2bb3a24bcfe71c7cbd6e9f6968ef7934e3ad56b0f1f9361b6b91",
                "sha256:309498e524b3e2da1f036d00cd5155e0b74cf9e1d964a3636c8ed63ca4a00d43",
                "sha256:f7c9b429437f7ada2d3d455ac4ea90ff38e0cb7ef2551b08d152264b74116309",
                "sha256:4dabdd56bbf16307e2328cb6ed1d42b0bb9b8f40551421271c0b38dc9a685dcc",
                "sha256:ea450ad6ef893e998f88a35dc9cc22f952c62b88d58f948344cf4eda1a6264fc"
            ]
        },
    }
]
```

所有的Docker镜像都起始于一个基础镜像层，当镜像修改或者新增新的内容时，就会在当前镜像层之上，创建新的镜像层。**即在添加额外的镜像层的同时，镜像始终保持是当前所有镜像的组合。** docker通过存储引擎（新版本采用快照机制）的方式实现镜像层堆栈，并保证多个镜像层对外展示为统一的文件系统。示例：

<img src="https://img.darklorder.com/img/202308111438925.png"/>

这个镜像中包含了三个镜像层，第一层有三个文件，第二层也有三个文件，第三层镜像中仅有一个文件，且这个文件是对第二层镜像中的文件5的一个更新版本。在这种情况下，上层镜像层中的文件会覆盖底层镜像层的文件，这样就使得文件的更新版本作为一个新的镜像层添加到镜像当中。

最后docker通过存储引擎将所有镜像层堆叠并合并，对外提供统一的视图。

<img src="https://img.darklorder.com/img/202308111439188.png"/>

Dockerfile中的操作对于镜像分层的影响：**在镜像构建过程中需要向镜像写入数据的时候会产生分层，一个写操作指令产生一个分层。**

```
// 实例，验证Dockerfile中的操作对于镜像分层的影响

// 第一步，简单编写一个Dockerfile，复制宿主机的文件到容器中，并且RUN 执行相关命令
[root@iZbp1bum6107bp8mgzkeunZ test]# cat Dockerfile
FROM centos:7
 # 写指令-COPY、RUN
COPY * /root/test/                   
RUN touch /etc/pidstst.log \
    && yum -y install sysstat
CMD /root/test/process.sh

// 第二步，创建镜像
[root@iZbp1bum6107bp8mgzkeunZ test]# docker build -f Dockerfile -t test .
Sending build context to Docker daemon  4.096kB
Step 1/4 : FROM centos:7
 ---> eeb6ee3f44bd
Step 2/4 : COPY * /root/test/
 ---> 897bd9a5ead0
Step 3/4 : RUN  touch /etc/pidstst.log     && yum -y install sysstat
 ---> Running in 4b90b3c273b0
Step 4/4 : CMD /root/test/test.sh
 ---> Running in dd115ea2e7f7
Removing intermediate container dd115ea2e7f7
 ---> 205226fedbc6
Successfully built 205226fedbc6
Successfully tagged test:latest

// 查看创建的镜像
[root@iZbp1bum6107bp8mgzkeunZ docker]# docker images
REPOSITORY           TAG       IMAGE ID       CREATED          SIZE
test                 latest    205226fedbc6   12 minutes ago   407MB

// 查看该镜像的分层结构信息
// 所有的Docker镜像都起始于一个基础镜像层，因此第一个layer为dockerfile中指定的基础镜像层
// 然后dockerfile中有多少个写指令，就会有多少个lay。 这里是三个layer，跟dockerfile中刚好对应上。
[root@iZbp1bum6107bp8mgzkeunZ docker]# docker image inspect 205226fedbc6
...
"RootFS": {
    "Type": "layers",
    "Layers": [
        "sha256:174f5685490326fc0a1c0f5570b8663732189b327007e47ff13d2ca59673db02",
        "sha256:114bd86a861ccc7b8fcc9c5702529d3e2df0fa7ad3e753e0ad072362db417e7c",
        "sha256:9759cbc1af5e6651027f437b3e902ab08725eda7fcf16cdeb5acc45cefb90a5b"
    ]
}
...
```


### 写时复制策略（Copy On Write）

> 当某个容器修改了基础镜像的内容，比如 /bin文件夹下的文件，这时其他容器的/bin文件夹是否会发生变化呢？
> 
> 答案是不会的。根据容器镜像的写时复制（Copy-on-Write）技术，某个容器对基础镜像的修改会被限制在单个容器内。
>

写时复制策略采用了共享和复制技术，**针对相同的数据系统只保留一份，所有操作都访问这一份数据。** 当有操作需要修改或添加数据时，操作系统会把这部分数据复制到新的地方再进行修改或添加，而其他操作仍然访问原数据区数据，这项技术节约了镜像的存储空间，加快了系统启动时间。

<img src="https://img.darklorder.com/img/202308111440457.png"/>

如图所示，当需要对镜像中的文件进行修改时，会将文件复制到容器层进行修改，上层文件会覆盖原始镜像文件。**注意：该文件存在于容器层，容器重启之后容器层重新建立，上一次容器运行时对于文件的修改全部丢失！**

只有当需要修改时才复制一份数据，这种特性被称作Copy-on-Write。可见，容器层保存的是镜像变化的部分，不会对镜像本身进行任何修改。


#### 内容寻址原理

在docker中，内容寻址就是根据文件内容来索引对应的镜像和镜像层，实际上就是对于镜像层的内容计算和校验后生成一个内容哈希值，并作为这个镜像层的唯一ID。在构造镜像时，根据这个ID来索引镜像层。



### Docker Overlay2 文件系统原理

>
> overlayFS是被称为联合文件系统的其中一个解决方案。在2014年，发布了第一个版本并且合并到了Linux的内核3.18版本中，此时，在docker被称为是overlay文件驱动。后来在Linux 内核4.0 版本中进行了改进，称为overlay2。（overlay存在诸多性能和不稳定的问题，不推荐使用overlay，直接使用默认的overlay2即可）
>

#### **overlay2工作原理**

![](https://img.darklorder.com/img/202308161527037.png)

overlayfs 通过三个目录：lower 目录、upper 目录、以及 work 目录实现，其中 lower 目录可以是多个，work 目录为工作基础目录，挂载后内容会被清空，且在使用过程中其内容用户不可见，最后联合挂载完成给用户呈现的统一视图称为为 merged 目录。

-   lowerdir对应底层文件系统，是能被上层文件系统upperdir所共享的只读层
-   workdir则可以理解为overlay2运作的一个工作目录，用于完成copy-on-write等操作
-   overlay2运作时（也就是容器启动时），会将lowerdir、upperdir和workdir联合挂载到merged目录，为使用者提供一个“**统一视图**”

查看 `/var/lib/docker/overlay2/容器ID`目录结构，使用`mount | grep overlay`查看overlay的挂载情况。如图，确实此目录下的link、lower、work、diff等目录是通过挂载多个目录后，合并显示在一起的。  
![](https://img.darklorder.com/img/202308161527930.png)

### **Overlay2 是如何存储文件的？**

#### **镜像怎么存储的？**

1、为了更好的演示，使用一个纯净的环境：没有任何镜像和容器，/var/lib/docker/overlay2目录也是空的  
![](https://img.darklorder.com/img/202308161527822.png)  
2、拉取一个nginx镜像，观察拉取过程：可以看到镜像一共被分为6层拉取。  
![](https://img.darklorder.com/img/202308161527361.png)  
3、/var/lib/docker/overlay2/ 目录下也多了6个文件夹  
![](https://img.darklorder.com/img/202308161527441.png)  
4、首先来查看一下l目录，可以看到l目录是一堆软连接，把一些较短的随机串软连到镜像层的 diff 文件夹下，这样做是为了避免达到mount命令参数的长度限制  
![](https://img.darklorder.com/img/202308161528058.png)  
5、`docker image inspect nginx` 查看nginx镜像的信息，每个镜像都会有一个`GraphDriver.Data`信息，这个信息指示了镜像是怎么存的  
![](https://img.darklorder.com/img/202308161528556.png)  
6、将这6个文件夹全部展开，可以看到目录结构几乎都是一致的，需要重点关注的是diff文件夹和lower文件。  
可以看到`5160f86fbe7acce3826ed5c7d1acdb351b931d67978b1c91138d86b7eef8d0ab`文件夹中不存在lower文件，说明它是最底层的，等于是根镜像，即docker pull时下载的第一层。  
同时，`diff`文件夹下的文件，正是Linux文件目录结构。说明在nginx的dockerfile中，肯定有FROM centos的操作。  
![](https://img.darklorder.com/img/202308161528219.png)  
**实例说明：`Dockerfile`的每一个命令都可能引起了系统的变化，它的每一个变化都会记录一层diff文件。**

#### **容器怎么存储的？**

1、当前环境有一个nginx镜像，/var/lib/docker/overlay2/ 目录下只有镜像层的存储目录。
![](https://img.darklorder.com/img/202308161538342.png)
2、docker run 启动一个容器
![](https://img.darklorder.com/img/202308161538353.png)
3、查看/var/lib/docker/overlay2/ 目录，发现新增了两个目录：其中带`-init`的目录是只读的；没有init的容器目录才是容器的读写目录
![](https://img.darklorder.com/img/202308161538349.png)
4、`link`和`lower`文件与镜像层的功能一致，`link`文件内容为该容器层的`短 ID`，`lower`文件为该层的所有父层镜像的`短 ID`。`diff`目录为容器的读写层，容器内修改的文件都会在`diff`中出现，`merged`目录为分层文件联合挂载后的结果，也是容器内的工作目录。
![](https://img.darklorder.com/img/202308161538303.png)
![](https://img.darklorder.com/img/202308161538307.png)
5、根据 docker inspect nginx获取到的`GraphDriver.Data`数据显示，merged目录，是lowerDir各个目录合并UpperDir各个目录后的结果。
![](https://img.darklorder.com/img/202308161538321.png)
6、当我们进入容器创建文件时，文件也会出现在这里。
![](https://img.darklorder.com/img/202308161538202.png)


**结论：** overlay2将镜像层和容器层都放在单独的目录，并且有唯一 ID，每一层仅存储发生变化的文件，最终使用联合挂载技术将容器层和镜像层的所有文件统一挂载到容器中，使得容器中看到完整的系统文件。




**参考资料**
[docker·PHP知识总结·看云](http://static.kancloud.cn/chunyu/php_basic_knowledge/3083129)