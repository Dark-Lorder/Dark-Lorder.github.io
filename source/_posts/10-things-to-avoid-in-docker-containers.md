---
title: Docker容器十诫
date: 2022-09-16 00:00:00
categories:
- Docker
tags:
- Docker
---

<!-- more -->

![](https://img.darklorder.com/img/202308151811281.jpeg )

当你刚开始使用容器时，会发现容器能解决许多问题，而且好处很多：

-  首先：容器是不可变的 —— 操作系统、库版本、配置、文件夹以及应用全都包裹在容器内。你可以确保，在 QA 阶段测试的一张图片，肯定会在生产环境中出现，并且行为保持一致。
-  其次：容器是轻量级的 —— 容器的内存占用很小。容器只会给主进程分配内存，因此无需十几万个 MB 的内存空间。
-  最后：容器速度很快 —— 启动容器就跟启动典型的 linux 进程一样快。无需好几分钟，一个新的容器可以在几秒内启动完毕。

然而，许多用户仍然只是将容器视为典型的虚拟机。他们忘记了容器的一个重要特征：**容器是可丢弃的**。

围绕容器的咒语：“容器是临时的”。

![](https://img.darklorder.com/img/202308151811693.png )

鉴于这一特征，用户必须转变他们使用以及管理容器时的心态。下面，笔者将介绍为了充分利用 Docker 容器的好处，用户应该**避免**的十个陷阱：

**1）不要在容器内存储数据** —— 容器可以被停止、销毁或者取代。运行在容器中的应用1.0版本应该能够轻易地被1.1版本所取代，且不产生任何影响或数据丢失。因此，如果你需要存储数据，请将其存储在卷组(volume)中。在这种情况下，你要格外小心两个容器向同一卷组写入数据的情况，因为这很容易导致数据污染。总之，要确保自己的应用向共享的数据存储区填写数据。

**2）不要将应用分开发布** —— 有些人会将容器视为虚拟机，他们中的大部分人认为，应该在现有的运行容器中部署应用。在开发阶段，因为需要不断地修改配置并调试应用，这样做无可厚非。但是，当持续交付管道行进至 QA 与生产阶段时，不应该把镜像和应用分开。记住：容器是**不可变**的。

**3）不要创建太大的镜像** —— 镜像越大，越难以分发。确保只留有运行应用或进程所需的文件和库。不要安装不必要的包或运行“update”(yum update)指令下载太多文件到新的镜像层。

更新：关于这条建议，有一篇解释更为详尽的文章：[《保持小巧：细究 Docker 镜像大小》](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fdevelopers.redhat.com%2Fblog%2F2016%2F03%2F09%2Fmore-about-docker-images-size%2F)

**4）不要使用单层镜像** —— 为了有效利用分层的文件系统，总是为操作系统创建基础镜像层，此外，分别为[用户名定义](https://yq.aliyun.com/go/articleRenderRedirect?url=https%3A%2F%2Fgithub.com%2Fjboss-dockerfiles%2Fbase%2Fblob%2Fmaster%2FDockerfile)、[运行时安装](https://yq.aliyun.com/go/articleRenderRedirect?url=https%3A%2F%2Fgithub.com%2Fjboss-dockerfiles%2Fwildfly%2Fblob%2Fmaster%2FDockerfile)、配置、以及自己的应用创建不同的镜像层。这样一来，重现、管理以及传送镜像会变得更为简单。

**5）不用为运行中的容器创建镜像** —— 换句话说，不要使用 “docker commit” 指令创建镜像。这种创建镜像的方法是不可重现的，应该完全避免。相反，总是使用 Dockerfile 或任何 S2I (source-to-image，源码到镜像) —— 完全可重现的方法来创建镜像。这样一来，如果你将 Dockerfile 保存在源码存储控制库(git)内，就可以追踪其后续变化。

**6）不要单独使用“latest(最新)”标签** —— 最新标签就像 Maven 用户眼中的 “SNAPSHOT(快照)”。因为容器本身的分层式文件系统，我们鼓励使用标签。但是，你可不想在几个月后正打算创建镜像时，却惊讶地发现应用无法运行，而原因居然是一个父层(Dockerfile 中的 FROM)已经被无法向后兼容的新版取代，或是创建缓存中检索出的“最新”版本是错的。此外，由于你无法追踪当前运行镜像的版本，“最新”标签也不应该在生产环境中部署容器时使用。

**7）不要在一个容器内运行多个进程** —— 容器在运行单一进程(http 后台进程、应用服务器、数据库)时几乎无可挑剔，但是，如果运行多个进程，你可能会在管理、检索日志以及独个更新进程时遇到麻烦。

**8）不要在镜像中存储凭证** —— 使用环境变量。不要将镜像中的任何用户名或密码写死。使用环境变量从容器外部检索这些信息。[Postgress 镜像](https://yq.aliyun.com/go/articleRenderRedirect?url=https%3A%2F%2Fgithub.com%2Fdocker-library%2Fpostgres%2Fblob%2F443c7947d548b1c607e06f7a75ca475de7ff3284%2F9.5%2Fdocker-entrypoint.sh)就是践行该准则的好榜样。

**9）不要以 root 用户运行进程** —— “默认情况下，docker 容器以 root 权限运行。(…)随着 Docker 的不断完善，更多[安全](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fblog.oneapm.com%2Ftags-%25E5%25AE%2589%25E5%2585%25A8.html)的默认选项会逐渐出现。就当下而言，要求 root 权限对有些用户而言比较危险，可能无法在所有环境中实现。你的镜像应该使用 USER 指令为容器确定一个非 root 运行权限。”（摘自[《Docker 镜像作者指南》](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fwww.projectatomic.io%2Fdocs%2Fdocker-image-author-guidance%2F)）

**10）不要依赖 IP 地址** —— 每个容器都有其内部 IP 地址，该地址可能因为启动或停止容器而发生改变。如果你的应用或微服务需要与另一个容器交换消息，应该使用环境变量在容器间传送合适的主机名与端口号。





**参考资料**
[OneAPM 官方博客](http://developerblog.redhat.com/2016/02/24/10-things-to-avoid-in-docker-containers/)
