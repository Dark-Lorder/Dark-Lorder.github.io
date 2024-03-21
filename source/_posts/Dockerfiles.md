---
title: Dockerfiles
date: 2022-09-10 00:00:00
categories:
- Docker
tags:
- Docker
---

<!-- more -->

在使用Docker的过程中，除了官方镜像外，在某些场景下我们也会构建定制化的专属镜像 。例如：需要在现有镜像中添加功能、将开发的应用软件容器化等。

目前，Docker官方提供的构建方案有两种：1. 基于容器创建 ，2 基于Dockerfile创建。两种方案之间各有特点，通过本文我们可以对此进行了解。

## **一. 基于容器创建**

这种方式最直观明了，在操作上也非常简单，整个过程只需要三个步骤：

-   创建容器

-   修改容器内容

-   将容器保存为镜像

下面以Nginx的镜像为例来演示该方案。

1. 创建一个目标容器

```
$ docker run --name nginx -d nginx
```

2. 修改容器内容

使用docker  \[container\] exec 命令可进入容器

```
$ docker exec -it nginx bash
root@7d944b6e893f:/#
```

在本示例中，我们在容器内安装一个进程查看工具，增加镜像的功能。

```
root@11e7ee9aed3f:/# apt-get update
root@11e7ee9aed3f:/# apt-get install -y procps
```

安装完成后，执行exit退出容器

3. 将容器保存为镜像

使用docker \[container\] commit 指令，可将容器转换为镜像。格式为”docker \[container\] commit + <container\_name> + <image\_name>:<tag>”。

```
$ docker commit nginx nginx-new:1.0
sha256:c478994adb8093a4425a7ededa88d651cc42352af0baa9e68cafe8c65b411703
```

查看镜像列表，可看到新的镜像已生成。

```
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx-new    1.0       c478994adb80   2 minutes ago   162MB
nginx        latest    605c77e624dd   3 months ago    141MB
```

基于容器创建的方式虽然简单且直观，但并不被推荐使用。首先，这种方式需要创建容器且手工操作，这使得该方式的效率低下。其次，通过这种方式生成的镜像，使用者无法知道整个镜像的创建过程，导致可能发生安全方面的隐患。

对此，官方推荐的方式是使用Dockerfile 来构建镜像。


## **二. 基于Dockerfile创建**

Dockerfile本身为一个文本文件，在文件中包含了镜像的创建指令，用户可以通过它快速构建定制化的镜像，这也是目前最常用的方式。

我们以上面的nginx镜像制作为例，来看一下Dockerfile的使用。

新建一个空目录（名称无要求），在目录内创建一个名为Dockerfile的文件，并写入以下内容：

```
$ vi Dockerfile
FROM nginx 
RUN apt-get update && apt-get install -y procps
```

完成以后 ，执行docker \[image\] build命令生成镜像。

```
 docker build -t nginx-new:2.0 .
```

**注释：**docker \[image\] build命令用于生成镜像，使用 -t 参数为镜像打tag，tag用于描述镜像的版本信息，如果我们没有指定 tag的话，则默认tag为latest；nginx-new:2.0 为新的镜像名和tag，最后面的 "." 表示当前目录为执行目录，命令将在当前目录中查看Dockerfile文件并执行。

命令运行结束后，查看镜像列表可看到已成功创建。

```
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx-new    2.0       2c99e0f4fc85   2 minutes ago   162MB
nginx-new    1.0       c478994adb80   6 minutes ago   162MB
nginx        latest    605c77e624dd   3 months ago    141MB
```

Dockerfile可以看成是docker commit的代码化过程，它的底层也是使用docker commit一层层的构建起新镜像。通过Dockerfile的方式构建镜像能带来显著的收益，首先是让镜像创建的流程变得自动化，这无疑会极大地提升效率和准确性；其次，Dockerfile可以与CI/CD流水线结合使用，实现持续集成；同时，这种方式也使得镜像构建的过程变得透明，所见即所得。

Dockerfile的语法格式如下所示，左边为指令，右边为对应的参数，构建时会从上到下逐行执行。这种语法简单且容易理解，对于用户而言学习成本较低。

```
INSTRUCTION arguments
```

下面我们来学习一下常用的Dockerfile指令，根据功能可以简单分为两类：配置指令和操作指令。

### **Dockerfile中的常用指令**

| 指令       | 功能简介                                                     |
| :--------- | :----------------------------------------------------------- |
| FROM       | 指定构建新Image时使用的基础Image，通常必须是Dockerfile的第一个有效指令，但其前面也可以出现ARG指令 |
| LABEL      | 附加到Image之上的元数据，键值格式                            |
| ENV        | 以键值格式设定环境变量，可被其后的指令所调用，且基于新生成的Image运行的Container中也会存在这些变量 |
| RUN        | 以FROM中定义的Image为基础环境运行指定命令，生成结果将作为新Image的一个镜像层，并可由后续指令所使用 |
| CMD        | 基于该Dockerfile生成的Image运行Container时，CMD能够指定容器中默认运行的程序，因而其只应该定义一次 |
| ENTRYPOINT | 类似于CMD指令的功能，但不能被命令行指定要运行的应用程序覆盖，且与CMD共存时，CMD的内容将作为该指令中定义的程序的参数 |
| WORKDIR    | 为RUN、CMD、ENTRPOINT、COPY和ADD等指令设定工作目录           |
| COPY       | 复制主机上或者前一阶段构建结果中（需要使用--from选项）文件或目录生成新的镜像层 |
| ADD        | 与COPY指令的功能相似，但ADD额外也支持使用URL指定的资源作为源文件 |
| VOLUME      | 指定基于新生成的Image运行Container时期望作为Volume使用的目录 |
| EXPOSE      | 指定基于新生成的Image运行Container时期望暴露的端口，但实际暴露与否取决于“docker run”命令的选项，支持TCP和UDP协议 |
| USER        | 为Dockerfile中该指令后面的RUN、CMD和ENTRYPOING指令中要运行的应用程序指定运行者身份UID，以及一个可选的GID |
| ARG         | 定义专用于build过程中的变量，但仅对该指标之后的调用生效，其值可由命令行选项“--build-arg”进行传递 |
| ONBUILD     | 触发器，生效于由该Dockerfile构建出的新Image被用于另一个Dockerfile中的FROM指令作为基础镜像时 |
| STOPSIGNAL  | 用于通知Container终止的系统调用信号                          |
| HEALTHCHECK | 定义检测容器应用的健康状态的具体方法                         |
| SHELL       | 为容器定义运行时使用的默认shell程序，Linux系统默认使用 [“/bin/sh”,”-c”]，Windows默认使用 ["cmd", "/S", "/C"] |




### **配置指令**

-   FROM指令

格式：FROM <image\_name\> 或 FROM <image\_name>:<tag>

该指令作为Dockerfile的第一条指令，用于指定基于哪个基础镜像来制作新的镜像，这条指令是必须存在的。为了避免最终产生的镜像变得臃肿，推荐在满足需求的前提下，尽量采用精简型的镜像，如Alpine等作为基础镜像。

示例：

```
FROM python:3.8-alpine
```

-   MAINTAINER指令

格式：MAINTAINER <author\_name>

指定镜像的作者信息。

示例：

```
MAINTAINER  Alex
```

-   EXPOSE指令
```
格式 ：格式为EXPOSE <port> [<port>]
```

用于声明容器内服务的监控端口，该指令只是起到声明作用，并不会自动完成端口映射。如果要映射端口，需要在启动容器时使用-p 参数指定。

示例：

```
EXPOSE 80
```


-   ENV指令
```
格式 ：ENV <key> <value>
```
指定容器的环境变量，该变量可以被RUN指令使用，并在容器启动后保持。

示例：

```
ENV MY_NAME="Alex"
```

-   VOLUME指令

格式：VOLUME  \["/data"\]

创建一个数据卷挂载点，一般用来存放数据库和需要持久化的数据等。

```
VOLUME /mydata
```

-   WORKDIR指令
```
格式：WORKDIR <workdir>
```
指定容器的工作目录，在它后面执行的指令将以 <workdir>做为当前工作目录。

示例：

```
WORKDIR /opt
```

### **操作指令**

-   RUN指令
```
格式：RUN <command> 
```
指定在容器中运行的命令，经常用于安装软件包等操作。当执行RUN指令时，会生成一个新的镜像层来保存这些内容。为了减少镜像体积，可以用&& 将多个命令放到单个RUN指令中执行。

示例：

```
RUN apt-get update && apt-get install -y procps
```

-   COPY指令
```
格式为：COPY <src> <dest>
```
用于复制本机的文件或目录到容器中，<src>指定源文件路径，<dest>指定目的路径。

```
COPY  test.txt /opt/
```

-   ADD指令
```
格式：ADD <src> <dest>
```
该指令与COPY相似，但如果<src>是tar文件的话，ADD会将其解压到<dest>。另外，<src>也可以指定为下载的url。

示例：

```
ADD  test.tar.gz  /opt/
```

-   CMD指令

格式：

CMD \["executable","param1","param2"\] (官方推荐方式）

CMD \["param1","param2"\]

CMD command param1 param2 

配置容器启动后默认运行的命令，每个Dockerfile只能有一条CMD命令，如果指定了多条，则只有最后一条被执行。CMD可以在docker run 启动容器时被替换。

示例：

```
CMD ["nginx", "-g", "daemon off;"]
```

-   ENTRYPOINT指令

格式：

ENTRYPOINT\["executable"，"param1"，"param2"\]

ENTRYPOINT command param1 param2 

配置容器启动后运行的命令，与CMD指令相似，但区别在于ENTRYPOINT指令无法被覆盖。

```
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

在理解Dockerfile相关的指令后，我们以一个Python程序为例，来实践一下将应用容器化的操作。

相关代码仓库地址：https://github.com/rui789/myapp.git

下载代码

```
$ git clone https://github.com/rui789/myapp.git
```

编写Dockerfile

```
$cd myapp
$ vi Dockerfile
FROM python:3.8-alpine
MAINTAINER alex
WORKDIR /opt/myapp
ENV TZ=Asia/Shanghai
COPY . .
RUN pip install -r requirements.txt 
EXPOSE 80
CMD ["/bin/sh","-c","python /opt/myapp/main.py"]
```

运行命令开始构建 

```
$ docker build -t myapp:1.0 .
```

查看镜像，已成功构建。

```
$ docker images myapp
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
myapp        1.0       903ea6552f8b   2 minutes ago   53.7MB
```

使用docker \[image\] history命令可查看镜像在构建过程执行了哪些操作，这有利于使用者了解镜像的情况。

```
 docker history myapp:1.0
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
903ea6552f8b   6 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "pyth…   0B        
2097b72b5704   6 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
......
```

使用镜像创建容器

```
$ docker run -d -p 80:80 myapp:1.0 
38ca1c8ae8ee12deff1db1df001ee2e47186de4637986bde3dc0582dd05fe0a6
```

浏览器访问地址，可看到服务已正常运行





**参考资料**
[Docker容器实战六：构建定制化镜像](https://mp.weixin.qq.com/s?__biz=MzU2OTc4NDI2MQ==&mid=2247484427&idx=1&sn=f901d9f97cdb30339475e3e153b08380&chksm=fcf826a1cb8fafb722111fe38c09ecafc6096b875169864b0c1a294199ebca24a3f6737596de&scene=21#wechat_redirect)
[Docker Dockerfile|菜鸟教程](https://www.runoob.com/docker/docker-dockerfile.html)