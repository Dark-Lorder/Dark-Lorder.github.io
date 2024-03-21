---
title: Docker存储卷
date: 2022-09-07 00:00:00
categories:
- Docker
tags:
- Docker
---

<!-- more -->

## 存储卷

## COW机制（写时复制）

Docker镜像由多个只读层叠加而成，启动容器时，Docker会加载只读镜像层并在镜像栈顶部添加一个读写层。

**如果容器修改了镜像的原有文件，那么只读层的文件不会被修改，而是会被复制到读写层做修改，只读版本依然存在，只是被读写层中的副本隐藏起来了，这就是“写时复制(COW)”机制。**

![](https://img.darklorder.com/img/202308161627460.png)

因为隔着很多层镜像，用上面这种方式，去访问一个文件，然后进行修改和删除等一类的操作，其效率会非常的低，  
而要想绕过这种限制，我们可通过使用存储卷来实现。

## 什么是存储卷

存储卷会把**宿主机本地文件系统中的目录 与容器文件系统上的目录建立绑定关系**。

**简单来说，当我们在容器中目录下写入数据时，因为目录绑定关系的存在，容器会将其内容直接写入到宿主机上的对应目录。**

![](https://img.darklorder.com/img/202308161627538.png)

在宿主机上的这个与容器形成绑定关系的目录被称作存储卷。

## 使用存储卷的好处

**①防止数据丢失**  
当容器所有运行进程的有效数据都保存在存储卷时，如果容器关闭甚至被删除时，只要不删除容器绑定在宿主机上的目录，我们就不用担心数据丢失了，以此来实现数据脱离容器的生命周期，被持久保存的目的。

**②摆脱容器和主机的对应限制**  
我们通过这种方式管理容器，容器就可以脱离主机的限制。任何一台主机安装docker以后，运行容器后，其数据可以置于一个共享存储文件系统上，比如nfs。

## 为什么要用存储卷

关闭并重启容器，其数据不受影响，但删除Docker容器，则其更改将会全部丢失。  
因此Docker存在的问题有：

-   存储于联合挂载文件系统中，不易于宿主机访问
-   容器间数据共享不便（容器之间本身是隔离的）
-   删除容器其数据会丢失  
    使用存储卷就是来解决以上问题的。

## 存储卷管理方式

存储卷（Data Volume）于容器初始化时被自动创建，由base image提供的卷中的数据会于此期间完成复制。

存储卷为Docker提供了独立于容器的数据管理机制，我们可以把镜像想象成静态文件，例如“程序”，把卷类比为动态内容，例如“数据”。所以镜像可以重复利用，而存储卷则可以共享。

**存储卷既实现了“程序(镜像)”和“数据(卷)”的分离，也实现了“程序(镜像)”和“制作镜像的主机”的分离，所以用户制作镜像时，无须再考虑镜像运行容器所在的主机环境。**

![](https://img.darklorder.com/img/202308161627876.png)

## 存储卷的分类

Docker有两种类型的卷，每种类型都在容器中存在一个挂载点，但其在宿主机上的位置有所不同：

-   **绑定挂载卷**

    -   指向主机文件系统上，用户指定位置的卷(常用，就算容器删了，数据也不会丢失)
-   **Docker 管理的卷**

    -   Docker 守护进程，在 Docker 所在的主机文件系统中，创建的卷（容器删了，数据也会消失）

![](https://img.darklorder.com/img/202308161627408.png)

**bind mount与 docker managed volume对比**


|                         | bind mount                | docker managed volume       |
| ----------------------- | ------------------------- | --------------------------- |
| volume位置              | 可任意指定                | /var/lib/docker/volumes/... |
| 对已有的mount point影响 | 隐藏并替换为volume        | 原有的数据复制到volume      |
| 是否支持单个文件        | 支持                      | 不支持，只能是目录          |
| 权限控制                | 可设置为只读，默认为读写  | 无控制，均为读写权限        |
| 移植性                  | 移植性弱，与host path绑定 | 移植性强，无需指定host目录  |



## 容器数据管理

用户在使用Docker的过程中，往往需要能查看容器内应用产生的数据，或者需要把容器内的数据进行备份，甚至多个容器之间进行数据的共享，这必然涉及容器的数据管理操作。

容器中管理数据主要有两种方式：

-   数据卷（Data Volumes）
-   数据卷容器（Data Volumes Containers）

容器Volume使用语法：  
**docker管理的卷（容器删除，数据同步丢失**）  
`docker run -it --name CONTAINER_NAME -v VOLUMEDIR IMAGE_NAME`

```
# 容器管理的卷的数据存储示例
[root@rookie ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND     CREATED      STATUS                    PORTS     NAMES
98b74ab02ea5   busybox   "/bin/sh"   5 days ago   Exited (137) 5 days ago             xyx
[root@rookie ~]# docker start xyx
xyx
[root@rookie ~]# docker inspect  xyx   # 找到存储的目录
[root@rookie ~]# cd /var/lib/docker/overlay2/3da71eb0eaa2667d371eec65f35c77485925ba08b53e1bd60d80068b6a709238/
[root@rookie 3da71eb0eaa2667d371eec65f35c77485925ba08b53e1bd60d80068b6a709238]# ls
diff  link  lower  merged  work
[root@rookie 3da71eb0eaa2667d371eec65f35c77485925ba08b53e1bd60d80068b6a709238]# ls merged/
bin  dev  etc  home  proc  root  sys  tmp  usr  var

# 容器内添加内容 容器外数据同步添加
[root@rookie ~]# docker exec -ti xyx /bin/sh
/ # touch 123 
/ # ls
123   bin   dev   etc   home  proc  root  sys   tmp   usr   var
[root@rookie 3da71eb0eaa2667d371eec65f35c77485925ba08b53e1bd60d80068b6a709238]# ls merged/
123  bin  dev  etc  home  proc  root  sys  tmp  usr  var


# 删除容器 容器外数据同步消失
/ # exit
[root@rookie ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND     CREATED      STATUS         PORTS     NAMES
98b74ab02ea5   busybox   "/bin/sh"   5 days ago   Up 7 minutes             xyx
[root@rookie ~]# docker rm -f xyx
xyx

[root@rookie 3da71eb0eaa2667d371eec65f35c77485925ba08b53e1bd60d80068b6a709238]# ls 
[root@rookie 3da71eb0eaa2667d371eec65f35c77485925ba08b53e1bd60d80068b6a709238]# ls merged/
ls: 无法访问'merged/': 没有那个文件或目录

```

**用户绑定的卷（容器删除，数据依旧保留）**  
`docker run -it --name CONTAINER_NAME -v HOSTDIR:VOLUMEDIR IMAGE_NAME`

```
#让真机上的b0test目录与容器内的data目录绑定
[[root@rookie ~]# docker run -it --name b0 -v /root/b0test:/data busybox 
/ # ls
bin   data  dev   etc   home  proc  root  sys   tmp   usr   var
/ # ls /data/
[root@rookie ~]# ls b0test/

#真机写入文件 容器同步添加
[root@rookie ~]# echo "nihao" > b0test/123
[root@rookie ~]# ls b0test/
123
[root@rookie ~]# cat b0test/123 
nihao

/ # cat /data/123 
nihao


#容器内创建文件 退出并删除容器 真机文件保留
/ # cd data/
/data # touch xieyanxin 
/data # ls
123        xieyanxin
/data # exit

[root@rookie ~]# docker rm b0
b0
[root@rookie ~]# docker ps -a 
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

[root@rookie ~]# ls b0test/
123  xieyanxin

```

## 在容器中使用数据卷

**下面部署一个网站（源码来自于源码之家），然后使用apache镜像创建一个b1容器，并创建一个数据卷`/root/b1test`挂载到容器的`/usr/loacl/apache2/htdocs`目录下：**

### 准备好数据卷的网站内容

```
#下载源码 并解压缩 创建好数据卷 并把网站文件放到此目录
[root@rookie ~]# ls
anaconda-ks.cfg  sayTheMoney.github.io-master.zip
[root@rookie ~]# dnf -y install unzip

[root@rookie ~]# mkdir b1test
[root@rookie ~]# mv sayTheMoney.github.io-master/* b1test/
[root@rookie ~]# ls b1test/
服务器之家.url                        android-chrome-512x512.png  favicon.ico
★★精品过期已备案域名，即买即用★★.url  apple-touch-icon.png        global.css
精品免费商业源码下载.url              build                       index.html
★★★★香港免备案云主机★★★★.url          favicon-16x16.png           site.webmanifest
android-chrome-192x192.png            favicon-32x32.png           thumbnail.png

```

### 运行容器 挂载一个主机目录作为数据卷，查看效果

-   注意  
    ①这里的-P是允许外部访问容器需要暴露的端口；  
    ②本地目录的路径必须是绝对路径，如果目录不存在，Docker会自动创建。

```
[root@rookie ~]# docker run -d --name b1 -v /root/b1test:/usr/local/apache2/htdocs -p 80:80  httpd 
084e3104515cd581ef8e1857c3f4ad1b0d45c9cd5c6e1002c1010f3d5cd86313
[root@rookie ~]# docker ps -a 
CONTAINER ID   IMAGE     COMMAND              CREATED         STATUS         PORTS                               NAMES
084e3104515c   httpd     "httpd-foreground"   7 seconds ago   Up 6 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   b1
[root@rookie ~]# ifconfig ens33
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.177.101  netmask 255.255.255.0  broadcast 192.168.177.255

```

-   **真机访问查看效果**  
    ![](https://img.darklorder.com/img/202308161627107.png)

此功能在测试的时候非常方便，我们可以放置一些程序或数据到本地目录中，然后在容器内运行和使用。

### 另一种只读的方式跑这个容器（网站）

-   **Docker挂载数据卷的默认权限是读写(rw)，用户也可以通过(ro)指定为只读：**

```
# 先把容器停掉删除
[root@rookie ~]# docker stop b1
b1
[root@rookie ~]# docker rm b1 
b1
#加上ro  指定为只读
[root@rookie ~]# docker run -d --name b1 -v /root/b1test:/usr/local/apache2/htdocs:ro -p 80:80  httpd 
fc106bac4858f65209b75b06adc970cc05b3451ddafb2a9cdc270ab49b312294
#进入容器 试着创建文件 测试效果（此时可以访问 但无法修改内容）
[root@rookie ~]# docker exec -it b1 /bin/bash
root@fc106bac4858:/usr/local/apache2# cd htdocs/
root@fc106bac4858:/usr/local/apache2/htdocs# ls
 android-chrome-192x192.png
 android-chrome-512x512.png
 apple-touch-icon.png
 build
 favicon-16x16.png
 favicon-32x32.png
 favicon.ico
 global.css
 index.html
 site.webmanifest
 
#提示为只读 
root@fc106bac4858:/usr/local/apache2/htdocs# touch 123
touch: cannot touch '123': Read-only file system  

#但容器外可以修改 且容器内数据也同步
[root@rookie ~]# cd b1test/
[root@rookie b1test]# touch xyx
[root@rookie b1test]# ls 
服务器之家.url                        favicon-16x16.png
★★精品过期已备案域名，即买即用★★.url  favicon-32x32.png
精品免费商业源码下载.url              favicon.ico
★★★★香港免备案云主机★★★★.url          global.css
android-chrome-192x192.png            index.html
android-chrome-512x512.png            site.webmanifest
apple-touch-icon.png                  thumbnail.png
build                                 xyx

root@fc106bac4858:/usr/local/apache2/htdocs# ls
 android-chrome-192x192.png
 android-chrome-512x512.png
 apple-touch-icon.png
 build
 favicon-16x16.png
 favicon-32x32.png
 favicon.ico
 global.css
 index.html
 site.webmanifest
 thumbnail.png
 xyx 

```

**加了:ro以后，外部可以修改数据，但容器内挂载的数据卷的数据就无法修改了。**

### 挂载一个本地主机文件作为数据卷（不常用，通常是目录）

\-v选项也可以从主机挂载单个文件到容器中作为数据卷：  
\[root@rookie ~\]# docker run -it --rm -v ~/.bash\_history:/.bash\_history centos /bin/bash

这样就可以记录在容器输入过的命令历史了。  
如果直接挂载一个文件到容器，使用文件编辑工具，包括vi或者sed去修改文件内容的时候，可能会造成inode的改变，这样将会导致错误。所以**推荐的方式是直接挂载文件所在的目录。**

## 数据卷容器

如果用户需要在容器之间共享一些持续更新的数据，可以使用数据卷容器。**数据卷容器本质上还是一个普通的容器，但它可以用来专门提供数据卷 供其他容器挂载使用**，方法如下：

-   **①首先，创建一个数据卷容器xieyanxin ，并在其中创建一个数据卷挂载到/dbdata：**

```
[root@rookie ~]# docker run -itd --name xieyanxin -v /dbdata  busybox 
b61ed4bf69e479a7cc45e5b0c7b2f10e7f4ad24c5ebacd1cc3a2947c0223be3f
[root@rookie ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
b61ed4bf69e4   busybox   "sh"      3 seconds ago   Up 2 seconds             xieyanxin

# 查看xieyanxin容器的dbdata目录是否存在
[root@rookie ~]# docker exec -it xieyanxin /bin/sh
/ # ls
bin     dev     home    root    tmp     var
dbdata  etc     proc    sys     usr


```

-   **②然后可以在其他容器中使用–volumes-from来挂载dbdata容器中的数据卷，例如创建xiaoxie1和xiaoxie2两个容器，并从xieyanxin容器挂载数据卷：**

```
[root@rookie ~]# docker run -itd --name xiaoxie2 --volumes-from xieyanxin busybox
468d1f865f3d0859ba468c8c0bbb751a07eb2703ae09c4cdd0915a8746a71b56
[root@rookie ~]# docker run -itd --name xiaoxie1 --volumes-from xieyanxin busybox
e301456fbb2298310bd2a741c6186c800e6e53ffe12dbb42824cba854d0c4ad3
[root@rookie ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
e301456fbb22   busybox   "sh"      6 seconds ago    Up 5 seconds              xiaoxie1
468d1f865f3d   busybox   "sh"      11 seconds ago   Up 10 seconds             xiaoxie2
b61ed4bf69e4   busybox   "sh"      4 minutes ago    Up 4 minutes              xieyanxin

# 容器xiaoxie1和xiaoxie2都挂载同一个数据卷到相同的/dbdata目录
[root@rookie b1test]# docker exec -it xiaoxie1 /bin/sh
/ # ls
bin     dev     home    root    tmp     var
dbdata  etc     proc    sys     usr
[root@rookie ~]# docker exec -it xiaoxie2 /bin/sh
/ # ls
bin     dev     home    root    tmp     var
dbdata  etc     proc    sys     usr

```

也**正是因为容器xiaoxie1和xiaoxie2都挂载同一个数据卷到相同的/dbdata目录，xieyanxin，iaoxie1，xiaoxie2这三个容器任何一方在该目录下的写入，其他容器都可以看到**。

例如，在xiaoxie1容器中创建一个guaguagua文件：

```
/ # cd /dbdata/
/dbdata # echo "nishihangua" > guaguagua
/dbdata # ls
guaguagua
```

在xieyanxin容器中查看：

```
[root@rookie ~]# docker exec -it xieyanxin /bin/sh
/ # ls
bin     dev     home    root    tmp     var
dbdata  etc     proc    sys     usr
/ # cat /dbdata/guaguagua 
nishihangua
```

-   **③可以多次使用–volumes-from参数来从多个容器挂载多个数据卷。还可以从其他已挂载了容器卷的容器来挂载数据卷：**

```
#xiaoxie999容器以xiaoxie1容器挂载数据卷  
[root@rookie ~]# docker run -itd --name  xiaoxie999 --volumes-from xiaoxie1 busybox
8de8ea4edfdda1cac392e6e9c854ef3037af3c76b5ed2821915d5346b0bae638
[root@rookie ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
8de8ea4edfdd   busybox   "sh"      5 seconds ago    Up 3 seconds              xiaoxie999
e301456fbb22   busybox   "sh"      16 minutes ago   Up 16 minutes             xiaoxie1
468d1f865f3d   busybox   "sh"      16 minutes ago   Up 16 minutes             xiaoxie2
b61ed4bf69e4   busybox   "sh"      21 minutes ago   Up 21 minutes             xieyanxin

#进入xiaoxie999容器创建 内容 ，到xieyanxin容器查看 
[root@rookie ~]# docker exec -it xiaoxie999 /bin/sh
/ # ls /dbdata/
guaguagua
/ # cd /dbdata/
/dbdata # echo "xiaoxie999laila" > 999
/dbdata # ls
999        guaguagua
/dbdata # exit

[root@rookie ~]# docker exec -it xieyanxin /bin/sh
/ # cat /dbdata/999 
xiaoxie999laila
```

-   **④使用–volumes-from参数所挂载数据卷的容器自身并不需要保持在运行状态。**

```
#停掉xieyanxin容器的运行状态
[root@rookie ~]# docker stop xieyanxin
[root@rookie ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                        PORTS     NAMES
8de8ea4edfdd   busybox   "sh"      7 minutes ago    Up 7 minutes                            xiaoxie999
e301456fbb22   busybox   "sh"      23 minutes ago   Up 23 minutes                           xiaoxie1
468d1f865f3d   busybox   "sh"      23 minutes ago   Up 23 minutes                           xiaoxie2
b61ed4bf69e4   busybox   "sh"      28 minutes ago   Exited (137) 44 seconds ago             xieyanxin  # xieyanxin已停止

#去xiaoxie2查看 然后创建一个文件
[root@rookie ~]# docker exec -it xiaoxie2 /bin/sh
/ # cd /dbdata/
/dbdata # ls
999        guaguagua
/dbdata # touch xiaoxie111
/dbdata # ls
999         guaguagua   xiaoxie111
/dbdata # exit

#去xiaoxie1查看 数据依旧是同步的
[root@rookie ~]# docker exec -it xiaoxie1 /bin/sh
/ # ls /dbdata/
999         guaguagua   xiaoxie111
```

-   ⑤如果删除了挂载的容器（包括xieyanxin、xiaoxie1和xiaoxie2），数据卷并不会被自动删除。**如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时显式使用docker rm -v命令来指定同时删除关联的容器。**

```
#删除xieyanxin并确认
[root@rookie ~]# docker rm -f xieyanxin
xieyanxin
[root@rookie ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
8de8ea4edfdd   busybox   "sh"      15 minutes ago   Up 14 minutes             xiaoxie999
e301456fbb22   busybox   "sh"      31 minutes ago   Up 31 minutes             xiaoxie1
468d1f865f3d   busybox   "sh"      31 minutes ago   Up 31 minutes             xiaoxie2

# 去xiaoxie1 xiaoxie2 查看 数据卷关系依然存在
[root@rookie b1test]# docker exec -it xiaoxie1 /bin/sh
/ #  cd /dbdata/
/dbdata # touch xixi

[root@rookie ~]# docker exec -it xiaoxie2 /bin/sh
/ # cd /dbdata/
/dbdata # ls
999         guaguagua   xiaoxie111  xixi

#删除全部容器 只保留xiaoxie999
 [root@rookie ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
8de8ea4edfdd   busybox   "sh"      15 minutes ago   Up 14 minutes             xiaoxie999
e301456fbb22   busybox   "sh"      31 minutes ago   Up 31 minutes             xiaoxie1
468d1f865f3d   busybox   "sh"      31 minutes ago   Up 31 minutes             xiaoxie2
[root@rookie ~]# docker rm -f xiaoxie1
xiaoxie1
[root@rookie ~]# docker rm -f xiaoxie2
xiaoxie2
[root@rookie ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
8de8ea4edfdd   busybox   "sh"      17 minutes ago   Up 17 minutes             xiaoxie999

#找到xiaoxie999对应在真机上的目录
[root@rookie ~]# docker inspect xiaoxie999
[root@rookie ~]# cd /var/lib/docker/volumes/b1622bf1ecdac3cd92f938c8958b32ba93535a000befc0a913431f650ba810e2/_data
[root@rookie _data]# ls
999  guaguagua  xiaoxie111  xixi

#用docker rm（不加v）删除容器，在真机的目录上数据还是在
[root@rookie ~]# docker rm -f xiaoxie999
xiaoxie999
[root@rookie ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@rookie _data]# ls
999  guaguagua  xiaoxie111  xixi
```

-   用docker rm（不加v）删除数据卷容器时，数据依旧会保留 ，想要全部删除也很简单 ，docker rm -vm 命令指定删除的容器，全部删除就可以了。


**参考资料**
[容器存储卷的介绍与使用_容器云 存储卷_先饮乌龙茶的博客-CSDN博客](https://blog.csdn.net/m0_67758799/article/details/124512038)
