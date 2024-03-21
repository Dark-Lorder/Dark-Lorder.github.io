---
title: Kubernetes概述
date: 2023-11-07 10:06:55
categories:
- Kubernetes
tags:
- Kubernetes
---

<!-- more -->



## Kubernetes概述

Kubernetes 是一个可移植、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。 Kubernetes 拥有一个庞大且快速增长的生态，其服务、支持和工具的使用范围相当广泛。

Kubernetes 这个名字源于希腊语，意为“舵手”或“飞行员”。k8s 这个缩写是因为 k 和 s 之间有八个字符的关系。 Google 在 2014 年开源了 Kubernetes 项目。 Kubernetes 建立在 Google 大规模运行生产工作负载十几年经验的基础上， 结合了社区中最优秀的想法和实践。



### Kubernetes集群架构

<img src="https://img.darklorder.com/img/202309061618958.webp"/>



### Kubernetes集群组件

<img src="https://img.darklorder.com/img/202310091208219.png"/>


#### 控制平面组件（Control Plane Components）

**kube-apiserver**
API 服务器是 Kubernetes 控制平面的组件， 该组件负责公开了 Kubernetes API，负责处理接受请求的工作。 API 服务器是 Kubernetes 控制平面的前端。

Kubernetes API 服务器的主要实现是 kube-apiserver。 kube-apiserver 设计上考虑了水平扩缩，也就是说，它可通过部署多个实例来进行扩缩。 你可以运行 kube-apiserver 的多个实例，并在这些实例之间平衡流量。

**etcd**
一致且高可用的键值存储，用作 Kubernetes 所有集群数据的后台数据库。

如果你的 Kubernetes 集群使用 etcd 作为其后台数据库， 请确保你针对这些数据有一份 备份计划。

你可以在官方文档中找到有关 etcd 的深入知识。

**kube-scheduler**
kube-scheduler 是控制平面的组件， 负责监视新创建的、未指定运行节点（node）的 Pods， 并选择节点来让 Pod 在上面运行。

调度决策考虑的因素包括单个 Pod 及 Pods 集合的资源需求、软硬件及策略约束、 亲和性及反亲和性规范、数据位置、工作负载间的干扰及最后时限。

**kube-controller-manager**
kube-controller-manager 是控制平面的组件， 负责运行控制器进程。

从逻辑上讲， 每个控制器都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在同一个进程中运行。

有许多不同类型的控制器。以下是一些例子：

- 节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
- 任务控制器（Job Controller）：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
- 端点分片控制器（EndpointSlice controller）：填充端点分片（EndpointSlice）对象（以提供 Service 和 Pod 之间的链接）。
- 服务账号控制器（ServiceAccount controller）：为新的命名空间创建默认的服务账号（ServiceAccount）。

#### Node 组件

**kubelet**
kubelet 会在集群中每个节点（node）上运行。 它保证容器（containers）都运行在 Pod 中。

kubelet 接收一组通过各类机制提供给它的 PodSpec，确保这些 PodSpec 中描述的容器处于运行状态且健康。 kubelet 不会管理不是由 Kubernetes 创建的容器。

**kube-proxy**
kube-proxy 是集群中每个节点（node）上所运行的网络代理， 实现 Kubernetes 服务（Service） 概念的一部分。

kube-proxy 维护节点上的一些网络规则， 这些网络规则会允许从集群内部或外部的网络会话与 Pod 进行网络通信。

如果操作系统提供了可用的数据包过滤层，则 kube-proxy 会通过它来实现网络规则。 否则，kube-proxy 仅做流量转发。

**容器运行时（Container Runtime）**
这个基础组件使 Kubernetes 能够有效运行容器。 它负责管理 Kubernetes 环境中容器的执行和生命周期。

Kubernetes 支持许多容器运行环境，例如 containerd、 CRI-O 以及 Kubernetes CRI (容器运行环境接口) 的其他任何实现。


#### 插件（Addons）

有关可用插件的完整列表，请参见 [插件（Addons）](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/addons/)。

**DNS**
尽管其他插件都并非严格意义上的必需组件，但几乎所有 Kubernetes 集群都应该有集群 DNS， 因为很多示例都需要 DNS 服务。

集群 DNS 是一个 DNS 服务器，和环境中的其他 DNS 服务器一起工作，它为 Kubernetes 服务提供 DNS 记录。

Kubernetes 启动的容器自动将此 DNS 服务器包含在其 DNS 搜索列表中。

**网络插件**
网络插件 是实现容器网络接口（CNI）规范的软件组件。它们负责为 Pod 分配 IP 地址，并使这些 Pod 能在集群内部相互通信。



### Kubernetes安装工具
https://landscape.cncf.io/card-mode?category=certified-kubernetes-installer



### [Kubernetes对象管理](https://kubernetes.io/zh-cn/docs/tasks/manage-kubernetes-objects/)

> kubectl 工具支持三种对象管理：
> 命令式命令
> 命令式对象配置
> 声明式对象配置

- 使用指令式命令管理 Kubernetes 对象
- 使用配置文件对 Kubernetes 对象进行命令式管理
- 使用配置文件对 Kubernetes 对象进行声明式管理


### Kubernetes资源对象

<img src="https://img.darklorder.com/img/202112042053985.png" alt="image-20211204205342922" style="zoom: 80%;" />

依据资源的主要功能作为分类标准，Kubernetes的API对象大体可分为工作负载（Workload）、发现和负载均衡（Discovery & LB）、配置和存储（Conf?ig & Storage）、集群（Cluster）以及元数据（Metadata）五个类别。
它们基本上都是围绕一个核心目的而设计：如何更好地运行和丰富Pod资源，从而为容器化应用提供更灵活、更完善的操作与管理组件


### [Kubernetes端口协议](https://kubernetes.io/zh-cn/docs/reference/networking/ports-and-protocols/#control-plane)

#### 控制面

| 协议 | 方向 | 端口范围  | 目的                    | 使用者               |
| ---- | ---- | --------- | ----------------------- | -------------------- |
| TCP  | 入站 | 6443      | Kubernetes API server   | 所有                 |
| TCP  | 入站 | 2379-2380 | etcd server client API  | kube-apiserver, etcd |
| TCP  | 入站 | 10250     | Kubelet API             | 自身, 控制面         |
| TCP  | 入站 | 10259     | kube-scheduler          | 自身                 |
| TCP  | 入站 | 10257     | kube-controller-manager | 自身                 |

尽管 etcd 的端口也列举在控制面的部分，但你也可以在外部自己托管 etcd 集群或者自定义端口。

#### 工作节点

| 协议 | 方向 | 端口范围    | 目的               | 使用者       |
| ---- | ---- | ----------- | ------------------ | ------------ |
| TCP  | 入站 | 10250       | Kubelet API        | 自身, 控制面 |
| TCP  | 入站 | 30000-32767 | NodePort Services† | 所有         |


### [Kubernetes网络模型](https://kubernetes.io/zh-cn/docs/concepts/services-networking/#the-kubernetes-network-model)

集群网络系统是 Kubernetes 的核心部分，但是想要准确了解它的工作原理可是个不小的挑战。 下面列出的是网络系统的的四个主要问题：

- 高度耦合的容器间通信：这个已经被 Pod 和 localhost 通信解决了。
- Pod 间通信：这是本文档讲述的重点。
- Pod 与 Service 间通信：涵盖在 Service 中。
- 外部与 Service 间通信：也涵盖在 Service 中。

Kubernetes 强制要求所有网络设施都满足以下基本要求（从而排除了有意隔离网络的策略）：

- Pod 能够与所有其他节点上的 Pod 通信， 且不需要网络地址转译（NAT）
- 节点上的代理（比如：系统守护进程、kubelet）可以和节点上的所有 Pod 通信

Kubernetes 网络解决四方面的问题：

- 一个 Pod 中的容器之间通过本地回路（loopback）通信。
- 集群网络在不同 Pod 之间提供通信。
- Service API 允许你向外暴露 Pod 中运行的应用， 以支持来自于集群外部的访问。
  - Ingress 提供专门用于暴露 HTTP 应用程序、网站和 API 的额外功能。
  - Gateway API 是一个插件， 为服务网络建模提供富有表现力、可扩展和面向角色的 API 系列类别。
- 你也可以使用 Service 来发布仅供集群内部使用的服务。


![](https://img.darklorder.com/img/202112032101273.png)




### Kubernetes资源规范
#### API资源规范


**示例**

```yaml
apiVersion: <GROUP/VERSION>			# 类型元数据 类型所隶属的API群组
kind: <RESOURCE KIND> 				# 类型元数据 类型标识
metadata:							# 对象元数据
  name: … 								# 对象名称
  namespace: … 						    # 对象名称空间
  labels: { … } 						# 对象标签
  annotations: { … }					# 对象注解
spec: { … }							# 资源规范/期望状态
status: { … }	                    # 实际状态
```




**参考资料**
[Kubernetes 文档 | Kubernetes](https://kubernetes.io/zh-cn/docs/home/)
[Kubernetes进阶实战](https://book.douban.com/subject/30435129/)
