---
title: Docker三剑客
date: 2022-09-11 00:00:00
categories:
- Docker
tags:
- Docker
---
Compose、Machine 和 Swarm 是Docker 原生提供的三大编排工具。简称Docker三剑客。
<!-- more -->

## Docker Compose 介绍

Dockerfile 可以让用户管理一个单独的应用容器；而 Compose 则允许用户在一个模板（YAML 格式）中定义一组相关联的应用容器（被称为一个 project，即项目），例如一个 Web 服务容器再加上后端的数据库服务容器等。

Docker-Compose 是 Docker 的一种编排服务，是一个用于在 Docker 上定义并运行复杂应用的工具，可以让用户在集群中部署分布式应用。

通过 Docker-Compose 用户可以很容易地用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖，完成构建。Docker-Compose 解决了容器与容器之间如何管理编排的问题。

Docker Compose 工作原理图

![](https://img.darklorder.com/img/202308251109146.png)

Compose 中有两个重要的概念：

-   服务 (service) ：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
-   项目 (project) ：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

一个项目可以由多个服务（容器）关联而成，Compose 面向项目进行管理，通过子命令对项目中的一组容器进行便捷地生命周期管理。

Compose 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。


## Docker Machine 介绍

Docker Machine 是 Docker 官方编排（Orchestration）项目之一，负责在多种平台上快速安装 Docker 环境。

Docker Machine 是一个工具，它允许你在虚拟宿主机上安装 Docker Engine ，并使用 docker-machine 命令管理这些宿主机。你可以使用 Machine 在你本地的 Mac 或 Windows box、公司网络、数据中心、或像 AWS 或 Digital Ocean 这样的云提供商上创建 Docker 宿主机。

使用 docker-machine 命令，你可以启动、审查、停止和重新启动托管的宿主机、升级 Docker 客户端和守护程序、并配置 Docker 客户端与你的宿主机通信。

### **为什么要使用它？**

Docker Machine 使你能够在各种 Linux 上配置多个远程 Docker 宿主机。 此外，Machine 允许你在较早的 Mac 或 Windows 系统上运行 Docker，如上一主题所述。 Docker Machine 有这两个广泛的用例。

-   我有一个较旧的桌面系统，并希望在 Mac 或 Windows 上运行 Docker

![](https://img.darklorder.com/img/202308251112595.png)

如果你主要在不符合新的 Docker for Mac 和 Docker for Windows 应用程序的旧 Mac 或 Windows 笔记本电脑或台式机上工作，则需要 Docker Machine 来在本地“运行Docker”（即Docker Engine）。在 Mac 或 Windows box 中使用 Docker Toolbox 安装程序安装 Docker Machine 将为 Docker Engine 配置一个本地的虚拟机，使你能够连接它、并运行 docker 命令。

-   我想在远程系统上配置 Docker 宿主机

![](https://img.darklorder.com/img/202308251112370.png)

Docker Engine Linux 系统上原生地运行。如果你有一个 Linux 作为你的主系统，并且想要运行 docker 命令，所有你需要做的就是下载并安装 Docker Engine 。然而，如果你想要在网络上、云中甚至本地配置多个 Docker 宿主机有一个有效的方式，你需要 Docker Machine。

无论你的主系统是 Mac、Windows 还是 Linux，你都可以在其上安装 Docker Machine，并使用 docker-machine 命令来配置和管理大量的 Docker 宿主机。它会自动创建宿主机、在其上安装 Docker Engine 、然后配置 docker 客户端。每个被管理的宿主机（“machine”）是 Docker 宿主机和配置好的客户端的结合。

### **Docker Engine 和 Docker Machine 有什么区别？**

当人们说“Docker”时，他们通常是指 Docker Engine，它是一个客户端 - 服务器应用程序，由 Docker 守护进程、一个REST API指定与守护进程交互的接口、和一个命令行接口（CLI）与守护进程通信（通过封装REST API）。Docker Engine 从 CLI 中接受docker 命令，例如`docker run <image>`、`docker ps` 来列出正在运行的容器、`docker images`来列出镜像，等等。

![](https://img.darklorder.com/img/202308251112326.jpeg)

Docker Machine 是一个用于配置和管理你的宿主机（上面具有 Docker Engine 的主机）的工具。通常，你在你的本地系统上安装 Docker Machine。Docker Machine有自己的命令行客户端 docker-machine 和 Docker Engine 客户端 docker。你可以使用 Machine 在一个或多个虚拟系统上安装 Docker Engine。

这些虚拟系统可以是本地的（就像你在 Mac 或 Windows 上使用 Machine 在 VirtualBox 中安装和运行 Docker Engine 一样）或远程的（就像你使用 Machine 在云提供商上 provision Dockerized 宿主机一样）。Dockerized 宿主机本身可以认为是，且有时就称为，被管理的“machines”。

![](https://img.darklorder.com/img/202308251112620.png)


### Docker Machine 最主要有两个作用：

-   使用 Docker Machine 方便在不同的环境中使用 Docker ，比如：Win/Mac
-   使用 Docker Machine 方便在云环境下批量部署 Docker环境，比如：私有云，公有云批量安装Docker环境

> virtualbox 安装很麻烦，我使用的虚拟机和云主机来做实验均没有安装成功，最后使用的是 Docker 官方提供的 Windows 安装包来完成的 virtualbox 相关操作。附 virtualbox 官网：[https://www.virtualbox.org/wiki/Downloads](https://link.zhihu.com/?target=https%3A//www.virtualbox.org/wiki/Downloads)


## Docker Swarm 介绍

实践中会发现，生产环境中使用单个 Docker 节点是远远不够的，搭建 Docker 集群势在必行。然而，面对 Kubernetes, Mesos 以及 Swarm 等众多容器集群系统，我们该如何选择呢？它们之中，Swarm 是 Docker 原生的，同时也是最简单，最易学，最节省资源的，比较适合中小型公司使用。

Swarm 在 Docker 1.12 版本之前属于一个独立的项目，在 Docker 1.12 版本发布之后，该项目合并到了 Docker 中，成为 Docker 的一个子命令。目前，Swarm 是 Docker 社区提供的唯一一个原生支持 Docker 集群管理的工具。它可以把多个 Docker 主机组成的系统转换为单一的虚拟 Docker 主机，使得容器可以组成跨主机的子网网络。

Docker Swarm 是一个为 IT 运维团队提供集群和调度能力的编排工具。用户可以把集群中所有 Docker Engine 整合进一个「虚拟 Engine」的资源池，通过执行命令与单一的主 Swarm 进行沟通，而不必分别和每个 Docker Engine 沟通。在灵活的调度策略下，IT 团队可以更好地管理可用的主机资源，保证应用容器的高效运行。

![](https://img.darklorder.com/img/202308251114853.png)

### Docker Swarm 优点

**任何规模都有高性能表现**

对于企业级的 Docker Engine 集群和容器调度而言，可拓展性是关键。任何规模的公司——不论是拥有五个还是上千个服务器——都能在其环境下有效使用 Swarm。 经过测试，Swarm 可拓展性的极限是在 1000 个节点上运行 50000 个部署容器，每个容器的启动时间为亚秒级，同时性能无减损。

**灵活的容器调度**

Swarm 帮助 IT 运维团队在有限条件下将性能表现和资源利用最优化。Swarm 的内置调度器（scheduler）支持多种过滤器，包括：节点标签，亲和性和多种容器部策略如 binpack、spread、random 等等。

**服务的持续可用性**

Docker Swarm 由 Swarm Manager 提供高可用性，通过创建多个 Swarm master 节点和制定主 master 节点宕机时的备选策略。如果一个 master 节点宕机，那么一个 slave 节点就会被升格为 master 节点，直到原来的 master 节点恢复正常。 此外，如果某个节点无法加入集群，Swarm 会继续尝试加入，并提供错误警报和日志。在节点出错时，Swarm 现在可以尝试把容器重新调度到正常的节点上去。

**和 Docker API 及整合支持的兼容性** Swarm 对 Docker API 完全支持，这意味着它能为使用不同 Docker 工具（如 Docker CLI，Compose，Trusted Registry，Hub 和 UCP）的用户提供无缝衔接的使用体验。

**Docker Swarm 为 Docker 化应用的核心功能（诸如多主机网络和存储卷管理）提供原生支持。**开发的 Compose 文件能（通过 docker-compose up ）轻易地部署到测试服务器或 Swarm 集群上。Docker Swarm 还可以从 Docker Trusted Registry 或 Hub 里 pull 并 run 镜像。

**综上所述，Docker Swarm 提供了一套高可用 Docker 集群管理的解决方案，完全支持标准的 Docker API，方便管理调度集群 Docker 容器，合理充分利用集群主机资源**。

> **并非所有服务都应该部署在Swarm集群内。数据库以及其它有状态服务就不适合部署在Swarm集群内。**

### 相关概念

### 节点

运行 Docker 的主机可以主动初始化一个 Swarm 集群或者加入一个已存在的 Swarm 集群，这样这个运行 Docker 的主机就成为一个 Swarm 集群的节点 (node) 。节点分为管理 (manager) 节点和工作 (worker) 节点。

管理节点用于 Swarm 集群的管理，docker swarm 命令基本只能在管理节点执行（节点退出集群命令 docker swarm leave 可以在工作节点执行）。一个 Swarm 集群可以有多个管理节点，但只有一个管理节点可以成为 leader，leader 通过 raft 协议实现。

工作节点是任务执行节点，管理节点将服务 (service) 下发至工作节点执行。管理节点默认也作为工作节点。你也可以通过配置让服务只运行在管理节点。下图展示了集群中管理节点与工作节点的关系。

![](https://img.darklorder.com/img/202308251115465.png)

### 服务和任务

任务 （Task）是 Swarm 中的最小的调度单位，目前来说就是一个单一的容器。 服务 （Services） 是指一组任务的集合，服务定义了任务的属性。服务有两种模式：

-   replicated services 按照一定规则在各个工作节点上运行指定个数的任务。
-   global services 每个工作节点上运行一个任务

两种模式通过 docker service create 的 –mode 参数指定。下图展示了容器、任务、服务的关系。

![](https://img.darklorder.com/img/202308251115592.png)








**参考资料**
[Docker(四)：Docker 三剑客之 Docker Compose - 知乎](https://zhuanlan.zhihu.com/p/34935579)
[Docker(五)：Docker 三剑客之 Docker Machine - 知乎](https://zhuanlan.zhihu.com/p/35102874)
[Docker(六)：Docker 三剑客之 Docker Swarm - 知乎](https://zhuanlan.zhihu.com/p/35961230)



