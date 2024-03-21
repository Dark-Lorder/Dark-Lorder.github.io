---
title: Docker安装部署
date: 2022-09-01 00:00:00
categories: Docker
tags:
- Docker
---
关于 Docker安装部署
<!-- more -->

## Docker 安装部署
目前，Docker支持在多个平台上进行安装部署，包括Linux、Windows和Mac。每个平台会有对应的系统版本要求，具体可以参见官方说明。

<img src="https://img.darklorder.com/img/202305121626702.png"/>

在实际应用中，Docker使用最多的场景是在Linux系统上。本文将基于市面上最常用的Centos和Ubuntu系统，对Docker的安装部署进行介绍。

##### 初始化环境 CentOS

Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker

通过 uname -r 命令查看你当前的内核版本
~~~bash
uname -r
~~~

使用root 权限登录 CentOS。确保 yum 包更新到最新
~~~bash
sudo yum update
~~~

卸载旧版本(如果安装过旧版本的话)

```bash
sudo yum remove docker  docker-common docker-selinux docker-engine
```

安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

设置yum源

官方源

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

阿里云源

```bash
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

清华大学源

```bash
sudo yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

##### 安装Docker CentOS

安装docker

```bash
sudo yum install docker-ce
```

也可以查看所有仓库中所有docker版本，并选择特定版本安装

```bash
yum list docker-ce --showduplicates | sort -r
sudo yum install docker-ce-版本号.ce
```

启动并加入开机启动

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

```bash
docker version
```



##### 初始化环境 Ubuntu

Docker CE 可以安装在 64 位的 x86平台或 ARM 平台上。Ubuntu 发行版中，LTS（Long-Term-Support）长期支持版本，会获得 5 年的升级维护支持，这样的版本会更稳定，因此在生产环境中推荐使用 LTS 版本。



卸载旧版本

旧版本的 Docker 称为 docker 或者 docker-engine，使用以下命令卸载旧版本：

```bash
sudo apt-get remove docker \
               docker-engine \
               docker.io
```

##### 安装Docker Ubuntu

使用脚本自动安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，Ubuntu 系统上可以使用这套安装脚本：

```bash
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 Edge 版本安装在系统中

启动Docker CE

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

卸载Docker

```bash
## 先执行命令
apt-get autoremove docker-ce
```

删除 /etc/apt/sources.list.d 目录下的 docker.list 文件



###### 安装Docker Ubuntu

~~~bash
## 修改源地址
root@ubuntu2004:~# vim /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

root@ubuntu2004:~#
root@ubuntu2004:~# visudo
%sudo ALL=(ALL:ALL) NOPASSWD:ALL
root@ubuntu2004:~$ 
## 更新本地包数据库
root@ubuntu2004:~$ apt update
## 更新所有已安装的包（也可以使用 full-upgrade）
root@ubuntu2004:~$ apt upgrade
root@ubuntu2004:~$ apt install screenfetch
root@ubuntu2004:~$ apt install vim net-tools aptitude git
root@ubuntu2004:~$ apt-get install uml-utilities
root@ubuntu2004:~$ apt install bridge-utils
root@ubuntu2004:~$ ufw disable 
root@ubuntu2004:~$ screenfetch
                          ./+o+-       root@ubuntu2004
                  yyyyy- -yyyyyy+      OS: Ubuntu 20.04 focal
               ://+//////-yyyyyyo      Kernel: x86_64 Linux 5.4.0-91-generic
           .++ .:/++++++/-.+sss/`      Uptime: 1m
         .:++o:  /++++++++/:--:/-      Packages: 656
        o:+o+:++.`..```.-/oo+++++/     Shell: bash 5.0.17
       .:+o:+o/.          `+sssoo+/    Resolution: No X Server
  .++/+:+oo+o:`             /sssooo.   WM: Not Found
 /+++//+:`oo+o               /::--:.   GTK Theme: Adwaita [GTK3]
 \+/+o+++`o++o               ++////.   Disk: 6.9G / 52G (15%)
  .++.o+++oo+:`             /dddhhh.   CPU: 11th Gen Intel Core i7-11700K @ 4x 3.6GHz
       .+.o+oo:.          `oddhhhh+    GPU: VMware SVGA II Adapter
        \+.++o+o``-````.:ohdhhhhh+     RAM: 509MiB / 3907MiB
         `:o+++ `ohhhhhhhhyo++os:     
           .o:`.syhhhhhhh/.oo++o`     
               /osyyyyyyo++ooo+++/    
                   ````` +oo+++o\:    
                          `oo++.      
root@ubuntu2004:~$ 
root@ubuntu2004:~$ vim /etc/ssh/sshd_config
PermitRootLogin yes              #允许root直接登录
PermitEmptyPasswords no          #因为设置了root密码，所以需要修改为no
root@ubuntu2004:~$ service ssh restart  重启ssh服务
root@ubuntu2004:~$ vim ~/.bash_profile
export PS1="[\u@\h \W]\\$ "
root@ubuntu2004:~$ source ~/.bash_profile
[root@ubuntu2004 ~]# 
~~~

~~~bash
## Ubuntu（使用 apt-get 进行安装）
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
~~~

~~~bash
root@ubuntu2004:~$ docker version
Client: Docker Engine - Community
 Version:           20.10.11
 API version:       1.41
 Go version:        go1.16.9
 Git commit:        dea9396
 Built:             Thu Nov 18 00:37:06 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied
root@ubuntu2004:~$ 
~~~

##### Docker 镜像加速器

~~~bash
## 镜像加速器
root@ubuntu2004:~# 
root@ubuntu2004:~# sudo mkdir -p /etc/docker
root@ubuntu2004:~# sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
          "https://docker.mirrors.ustc.edu.cn",
          "https://hub-mirror.c.163.com",
          "https://reg-mirror.qiniu.com"
  ]
}
EOF
root@ubuntu2004:~# systemctl daemon-reload
root@ubuntu2004:~# systemctl restart docker
root@ubuntu2004:~# docker info
 ...
 Registry Mirrors:
  https://docker.mirrors.ustc.edu.cn/
  https://hub-mirror.c.163.com/
  https://reg-mirror.qiniu.com/
 Live Restore Enabled: false
root@ubuntu2004:~# 
~~~



**参考资料**
[CSDN【云原生】Docker镜像详细讲解](https://micromaple.blog.csdn.net/article/details/125727576)
[阿里云 Docker CE镜像](https://developer.aliyun.com/mirror/docker-ce)
[阿里云 Ubuntu 镜像](https://developer.aliyun.com/mirror/ubuntu)

