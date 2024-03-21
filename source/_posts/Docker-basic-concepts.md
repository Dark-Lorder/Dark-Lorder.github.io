---
title: Docker基本概念
date: 2022-09-03 00:00:00
categories: Docker
tags:
- Docker
---

<!-- more -->

### Docker 概念

  当我们请求 Docker 运行容器时，Docker 会在计算机上设置一个资源隔离的环境。然后将打包的应用程序和关联的文件复制到 Namespace 内的文件系统中，此时环境的配置就完成了。之后 Docker 会执行我们预先指定的命令，运行应用程序。

> 镜像不包含任何动态数据，其内容在构建之后也不会被改变。


**核心概念**

- `Build`, `Ship and Run`（搭建、运输、运行）；

- `Build once`, `Run anywhere`（一次搭建，处处运行）；

- Docker 本身并不是容器，它是创建容器的工具，是应用容器引擎；

- Docker 三大核心概念，分别是：镜像 Image，容器 Container、仓库 Repository；

- Docker 技术使用 Linux 内核和内核功能（例如 Cgroups 和 namespaces）来分隔进程，以便各进程相互独立运行。

- 由于 Namespace 和 Cgroups 功能仅在 Linux 上可用，因此容器无法在其他操作系统上运行。那么 Docker 如何在 macOS 或 Windows 上运行？Docker 实际上使用了一个技巧，并在非 Linux 操作系统上安装 Linux 虚拟机，然后在虚拟机内运行容器。

- 镜像是一个可执行包，其包含运行应用程序所需的代码、运行时、库、环境变量和配置文件，容器是镜像的运行时实例。


**Docker 的主要用途，目前有三大类。**

**（1）提供一次性的环境。** 比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。

**（2）提供弹性的云服务。** 因为 Docker 容器可以随开随关，很适合动态扩容和缩容。

**（3）组建微服务架构。** 通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

<img src="https://img.darklorder.com/img/202111271935108.png"/>

### Docker 容器

dotCloud公司的Docker项目最初也是建立在**LXC**之上，以促进容器技术对开发人员和用户更加友好；

- 不久之后，Docker便使用了自行研发的**libcontainer**取代了LXC
- 在Docker项目声名大噪之后，dotCloud公司也更名为Docker

最终，**Docker**于**2013**年发布，解决了开发人员在端到端到运行容器时遇到的许多问题

- 容器镜像格式
- 构建容器镜像：Dockerfile、docker build
- 容器镜像管理：docker image、docker rmi
- 容器实例管理：docker ps、docker rm、……
- 共享容器镜像：docker push/pull
- 运行容器镜像的方式：docker run


### Docker 系统组件

<img src="https://img.darklorder.com/img/202111271940584.png"/>

Docker系统有三个关键组件  (C/S架构)

- Docker CLI（**客户端**）
- Docker Daemon（**守护进程** 管理本地组件）
- Registry（**注册表** 存，检索）

Docker Daemon

- REST API（衔接 Docker Daemon和Docker CLI）（http）

- Objects （**对象**：Resource资源实例化）

  Image（**镜像**）

  Container （**容器**)

  Volume（**数据卷**）

  Network（**网络**）


### Docker 镜像

运行容器时，基于Mount名称空间建立的隔离文件系统中的内容来自镜像

- Docker镜像是一个特殊的文件系统，它必须包含运行应用程序所需的一切——所有依赖项、配置、脚本、二进制文件等
- 镜像还包含容器的其他配置，例如环境变量、要运行的默认命令、 和其他元数据
- 通常，镜像还要定义默认启动的应用

Docker镜像由许多层（Layer）叠加而成

- 依赖于特殊的存储驱动，例如aufs、devicemapper、overlay2等
- 尽管每种存储驱动程序实现的管理方式不尽相同，但它们都使用可堆叠的镜像层和写时复制（CoW）策略



### Docker 容器

运行有Docker Daemon的主机负责管理本地容器实例的生命周期

- Docker Daemon通过其监听的Socket API接收Docker对象的管理请求，包括容器的生命周期中的各类管理操作
- 容器实例的创建要基于本地存储的Docker镜像进行

- 实例启动后，要在前台（foreground，不能转为守护进程模式）运行镜像中定义的默认应用，或用户指定的应用

创建容器时，指定了本地不存在的镜像时，则需要由Docker Daemon自动至Registry上进行下载

- Docker Daemon 默认使用的Registry是[DockerHub](https://registry.hub.docker.com/)

### 从Docker说起

OCI (Open Container Initiative)

- 较早版本的Docker是一个单体系统，但其内部的各类功能间并不存在依赖关系，每类功能都可以在一个独立的工具中实现，而且每个工具都可以基于通用格式和同一个容器标准来协作

- 于是，2015年6月，Docker、Google、CoreOS和其他供应商创建了OCI标准，它包括

  		image-spec：镜像格式规范

  		runtime-spec：运行时规范

- Docker将用于运行容器的代码（libcontainer）作为一个名为runC的项目分解出来，并捐赠给了OCI作为参考实现

<img src="https://img.darklorder.com/img/202111272119098.png" style="zoom:50%;" />



### Docker 程序组件

为了兼容OCI规范，自1.11.0版本起始，Docker引擎由一个单一组件拆分成4个独立的项目

- Docker engine（Docker-daemon）

- containerd：守护进程，高级别的container runtime

  	几乎囊括了容器运行时所需要的容器创建、启动、停止、中止、信号处理和删除，以及镜像管理（镜像和元信息等）等所有功能

  	通过grpc向上层调用者公开其API，可被兼容的任何上层系统所调用，例如Docker Engine或kubernetes等容器编排系统

  	但具体的容器管理还需要OCI兼容的runtime负责完成

- containerd-shim：支持多种不同的OCI runtime

  	containerd-shim 的意思是垫片，类似于拧螺丝时夹在螺丝和螺母之间的垫片。containerd-shim 的主要作用是将 containerd 和真正的容器进程解耦，使用 containerd-shim 作为容器进程的父进程，从而实现重启 containerd 不影响已经启动的容器进程。

- runc：低级别的container runtime

<img src="https://img.darklorder.com/img/202111272129706.png" style="zoom: 55%;" />


<img src="https://img.darklorder.com/img/202111272128656.png" style="zoom: 39%;" />


### 低级和高级容器运行时

当人们想到容器运行时，可能会想到一系列示例；runc、lxc、lmctfy、Docker（容器）、rkt、cri-o。这些中的每一个都是为不同的情况而构建的，并实现了不同的功能。有些，如 containerd 和 cri-o，实际上使用 runc 来运行容器，在High-Level实现镜像管理和 API。与 runc 的Low-Level实现相比，可以将这些功能（包括镜像传输、镜像管理、镜像解包和 API）视为High-Level功能。考虑到这一点，您可以看到容器运行时空间相当复杂。每个运行时都涵盖了这个Low-Level到High-Level频谱的不同部分。这是一个非常主观的图表：

<img src="https://img.darklorder.com/img/202308030314104.png"/>

因此，从实际出发，通常只专注于正在运行的容器的runtime通常称为“Low-Level容器运行时”，支持更多高级功能（如镜像管理和gRPC / Web API）的运行时通常称为“High-Level容器运行时”，“High-Level容器运行时”或通常仅称为“容器运行时”，我将它们称为“High-Level容器运行时”。值得注意的是，Low-Level容器运行时和High-Level容器运行时是解决不同问题的、从根本上不同的事物。

Low-Level容器运行时：容器是通过Linux nanespace和Cgroups实现的，Namespace能让你为每个容器提供虚拟化系统资源，像是文件系统和网络，Cgroups提供了限制每个容器所能使用的资源的如内存和CPU使用量的方法。在最低级别的运行时中，容器运行时负责为容器建立namespaces和cgroups,然后在其中运行命令，Low-Level容器运行时支持在容器中使用这些操作系统特性。目前来看低级容器运行时有：runc ：我们最熟悉也是被广泛使用的容器运行时，代表实现Docker。runv：runV 是一个基于虚拟机管理程序（OCI）的运行时。它通过虚拟化 guest kernel，将容器和主机隔离开来，使得其边界更加清晰，这种方式很容易就能帮助加强主机和容器的安全性。代表实现是kata和Firecracker。runsc：runsc = runc + safety ，典型实现就是谷歌的gvisor，通过拦截应用程序的所有系统调用，提供安全隔离的轻量级容器运行时沙箱。截止目前，貌似并不没有生产环境使用案例。wasm : Wasm的沙箱机制带来的隔离性和安全性，都比Docker做的更好。但是wasm 容器处于草案阶段，距离生产环境尚有很长的一段路。

High-Level容器运行时：通常情况下，开发人员想要运行一个容器不仅仅需要Low-Level容器运行时提供的这些特性，同时也需要与镜像格式、镜像管理和共享镜像相关的API接口和特性，而这些特性一般由High-Level容器运行时提供。就日常使用来说，Low-Level容器运行时提供的这些特性可能满足不了日常所需，因为这个缘故，唯一会使用Low-Level容器运行时的人是那些实现High-Level容器运行时以及容器工具的开发人员。那些实现Low-Level容器运行时的开发者会说High-Level容器运行时比如containerd和cri-o不像真正的容器运行时，因为从他们的角度来看，他们将容器运行的实现外包给了runc。但是从用户的角度来看，它们只是提供容器功能的单个组件，可以被另一个的实现替换，因此从这个角度将其称为runtime仍然是有意义的。即使containerd和cri-o都使用runc，但是它们是截然不同的项目，支持的特性也是非常不同的。dockershim, containerd 和cri-o都是遵循CRI的容器运行时，我们称他们为高层级运行时（High-level Runtime）。
Kubernetes 只需支持 containerd 等high-level container runtime即可。由containerd 按照OCI 规范去对接不同的low-level container runtime，比如通用的runc，安全增强的gvisor，隔离性更好的runv。

### CRI和容器运行时

我们知道 Kubernetes 提供了一个 CRI 的容器运行时接口，那么这个 CRI 到底是什么呢？这个其实也和 Docker 的发展密切相关的。

在 Kubernetes 早期的时候，当时 Docker 实在是太火了，Kubernetes 当然会先选择支持 Docker，而且是通过硬编码的方式直接调用 Docker API，后面随着 Docker 的不断发展以及 Google 的主导，出现了更多容器运行时，Kubernetes 为了支持更多更精简的容器运行时，Google 就和红帽主导推出了 CRI 标准，用于将 Kubernetes 平台和特定的容器运行时（当然主要是为了干掉 Docker）解耦。

CRI（Container Runtime Interface 容器运行时接口）本质上就是 Kubernetes 定义的一组与容器运行时进行交互的接口，所以只要实现了这套接口的容器运行时都可以对接到 Kubernetes 平台上来。不过 Kubernetes 推出 CRI 这套标准的时候还没有现在的统治地位，所以有一些容器运行时可能不会自身就去实现 CRI 接口，于是就有了 shim（垫片）， 一个 shim 的职责就是作为适配器将各种容器运行时本身的接口适配到 Kubernetes 的 CRI 接口上，其中 dockershim 就是 Kubernetes 对接 Docker 到 CRI 接口上的一个垫片实现。

<img src="https://img.darklorder.com/img/202308030329960.png"/>

Kubelet 通过 gRPC 框架与容器运行时或 shim 进行通信，其中 kubelet 作为客户端，CRI shim（也可能是容器运行时本身）作为服务器。

CRI 定义的 API(https://github.com/kubernetes/kubernetes/blob/release-1.5/pkg/kubelet/api/v1alpha1/runtime/api.proto) 主要包括两个 gRPC 服务，ImageService 和 RuntimeService，ImageService 服务主要是拉取镜像、查看和删除镜像等操作，RuntimeService 则是用来管理 Pod 和容器的生命周期，以及与容器交互的调用（exec/attach/port-forward）等操作，可以通过 kubelet 中的标志 --container-runtime-endpoint 和 --image-service-endpoint 来配置这两个服务的套接字。

<img src="https://img.darklorder.com/img/202308030351171.png"/>

不过这里同样也有一个例外，那就是 Docker，由于 Docker 当时的江湖地位很高，Kubernetes 是直接内置了 dockershim 在 kubelet 中的，所以如果你使用的是 Docker 这种容器运行时的话是不需要单独去安装配置适配器之类的，当然这个举动似乎也麻痹了 Docker 公司。

<img src="https://img.darklorder.com/img/202308030351558.png"/>

现在如果我们使用的是 Docker 的话，当我们在 Kubernetes 中创建一个 Pod 的时候，首先就是 kubelet 通过 CRI 接口调用 dockershim，请求创建一个容器，kubelet 可以视作一个简单的 CRI Client, 而 dockershim 就是接收请求的 Server，不过他们都是在 kubelet 内置的。

dockershim 收到请求后, 转化成 Docker Daemon 能识别的请求, 发到 Docker Daemon 上请求创建一个容器，请求到了 Docker Daemon 后续就是 Docker 创建容器的流程了，去调用 containerd，然后创建 containerd-shim 进程，通过该进程去调用 runc 去真正创建容器。

其实我们仔细观察也不难发现使用 Docker 的话其实是调用链比较长的，真正容器相关的操作其实 containerd 就完全足够了，Docker 太过于复杂笨重了，当然 Docker 深受欢迎的很大一个原因就是提供了很多对用户操作比较友好的功能，但是对于 Kubernetes 来说压根不需要这些功能，因为都是通过接口去操作容器的，所以自然也就可以将容器运行时切换到 containerd 来。

<img src="https://img.darklorder.com/img/202308030329927.png"/>

切换到 containerd 可以消除掉中间环节，操作体验也和以前一样，但是由于直接用容器运行时调度容器，所以它们对 Docker 来说是不可见的。 因此，你以前用来检查这些容器的 Docker 工具就不能使用了。

你不能再使用 docker ps 或 docker inspect 命令来获取容器信息。由于不能列出容器，因此也不能获取日志、停止容器，甚至不能通过 docker exec 在容器中执行命令。

当然我们仍然可以下载镜像，或者用 docker build 命令构建镜像，但用 Docker 构建、下载的镜像，对于容器运行时和 Kubernetes，均不可见。为了在 Kubernetes 中使用，需要把镜像推送到镜像仓库中去。

从上图可以看出在 containerd 1.0 中，对 CRI 的适配是通过一个单独的 CRI-Containerd 进程来完成的，这是因为最开始 containerd 还会去适配其他的系统（比如 swarm），所以没有直接实现 CRI，所以这个对接工作就交给 CRI-Containerd 这个 shim 了。

然后到了 containerd 1.1 版本后就去掉了 CRI-Containerd 这个 shim，直接把适配逻辑作为插件的方式集成到了 containerd 主进程中，现在这样的调用就更加简洁了。

<img src="https://img.darklorder.com/img/202308030329202.png"/>

与此同时 Kubernetes 社区也做了一个专门用于 Kubernetes 的 CRI 运行时 [CRI-O](https://cri-o.io/)，直接兼容 CRI 和 OCI 规范。

<img src="https://img.darklorder.com/img/202308030329388.png"/>

这个方案和 containerd 的方案显然比默认的 dockershim 简洁很多，不过由于大部分用户都比较习惯使用 Docker，所以大家还是更喜欢使用 dockershim 方案。

但是随着 CRI 方案的发展，以及其他容器运行时对 CRI 的支持越来越完善，Kubernetes 社区在 2020 年 7 月份就开始着手移除 dockershim 方案了：https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/2221-remove-dockershim，现在的移除计划是在 1.20 版本中将 kubelet 中内置的 dockershim 代码分离，将内置的 dockershim 标记为维护模式，当然这个时候仍然还可以使用 dockershim，目标是在 1.23/1.24 版本发布没有 dockershim 的版本（代码还在，但是要默认支持开箱即用的 docker 需要自己构建 kubelet，会在某个宽限期过后从 kubelet 中删除内置的 dockershim 代码）。

那么这是否就意味这 Kubernetes 不再支持 Docker 了呢？当然不是的，这只是废弃了内置的 dockershim 功能而已，Docker 和其他容器运行时将一视同仁，不会单独对待内置支持，如果我们还想直接使用 Docker 这种容器运行时应该怎么办呢？可以将 dockershim 的功能单独提取出来独立维护一个 cri-dockerd 即可，就类似于 containerd 1.0 版本中提供的 CRI-Containerd，当然还有一种办法就是 Docker 官方社区将 CRI 接口内置到 Dockerd 中去实现。

但是我们也清楚 Dockerd 也是去直接调用的 Containerd，而 containerd 1.1 版本后就内置实现了 CRI，所以 Docker 也没必要再去单独实现 CRI 了，当 Kubernetes 不再内置支持开箱即用的 Docker 的以后，最好的方式当然也就是直接使用 Containerd 这种容器运行时，而且该容器运行时也已经经过了生产环境实践的，接下来我们就来学习下 Containerd 的使用。

**Containerd**

我们知道很早之前的 Docker Engine 中就有了 containerd，只不过现在是将 containerd 从 Docker Engine 里分离出来，作为一个独立的开源项目，目标是提供一个更加开放、稳定的容器运行基础设施。分离出来的 containerd 将具有更多的功能，涵盖整个容器运行时管理的所有需求，提供更强大的支持。

containerd 是一个工业级标准的容器运行时，它强调简单性、健壮性和可移植性，containerd 可以负责干下面这些事情：

- 管理容器的生命周期（从创建容器到销毁容器）
- 拉取/推送容器镜像
- 存储管理（管理镜像及容器数据的存储）
- 调用 runc 运行容器（与 runc 等容器运行时交互）
- 管理容器网络接口及网络

**架构**


containerd 可用作 Linux 和 Windows 的守护程序，它管理其主机系统完整的容器生命周期，从镜像传输和存储到容器执行和监测，再到底层存储到网络附件等等。

<img src="https://img.darklorder.com/img/202308030352947.png"/>
上图是 containerd 官方提供的架构图，可以看出 containerd 采用的也是 C/S 架构，服务端通过 unix domain socket 暴露低层的 gRPC API 接口出去，客户端通过这些 API 管理节点上的容器，每个 containerd 只负责一台机器，Pull 镜像，对容器的操作（启动、停止等），网络，存储都是由 containerd 完成。具体运行容器由 runc 负责，实际上只要是符合 OCI 规范的容器都可以支持。

为了解耦，containerd 将系统划分成了不同的组件，每个组件都由一个或多个模块协作完成（Core 部分），每一种类型的模块都以插件的形式集成到 Containerd 中，而且插件之间是相互依赖的，例如，上图中的每一个长虚线的方框都表示一种类型的插件，包括 Service Plugin、Metadata Plugin、GC Plugin、Runtime Plugin 等，其中 Service Plugin 又会依赖 Metadata Plugin、GC Plugin 和 Runtime Plugin。每一个小方框都表示一个细分的插件，例如 Metadata Plugin 依赖 Containers Plugin、Content Plugin 等。比如:

- Content Plugin: 提供对镜像中可寻址内容的访问，所有不可变的内容都被存储在这里。
- Snapshot Plugin: 用来管理容器镜像的文件系统快照，镜像中的每一层都会被解压成文件系统快照，类似于 Docker 中的 graphdriver。

总体来看 containerd 可以分为三个大块：Storage、Metadata 和 Runtime。

<img src="https://img.darklorder.com/img/202308030352693.png"/>



**参考资料**
Kubernetes进阶实战
[Docker、Containerd、RunC分别是什么-腾讯云开发者社区-运维开发故事](https://cloud.tencent.com/developer/article/1895805)
[真正运行容器的工具：深入了解 runc 和 OCI 规范-腾讯云开发者社区-运维开发故事](https://cloud.tencent.com/developer/article/1895808)
[一文搞懂容器运行时 Containerd-阳明的博客](https://www.qikqiak.com/post/containerd-usage/)