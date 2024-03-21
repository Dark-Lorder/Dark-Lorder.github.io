---
title: Harbor的高可用方案
date: 2022-09-17 00:00:00
categories:
- Docker
tags:
- Docker
---

<!-- more -->

随着 Harbor 被越来越多地部署在生产环境下，Harbor 的高可用性成为用户关注的热点。对于一些大中型企业用户，如果只有单实例的 Harbor，则一旦发生故障，其从开发到交付的流水线就可能被迫停止，无法满足高可用需求。

本文提供基于 Harbor 的不同安装包的高可用方案，目标是移除单点故障，提高系统的高可用性。其中，基于 Harbor Helm Chart 的高可用方案为官方验证过的方案，基于多 Kubernetes 集群和基于离线安装包的高可用方案为参考方案。

## **1\. 基于 Harbor Helm Chart 的高可用方案**

Kubernetes 平台具有自愈（self-healing）能力，当容器崩溃或无响应时，可自动重启容器，必要时可把容器从失效的节点调度到正常的节点。本方案通过 Helm 部署 Harbor Helm Chart 到 Kubernetes 集群来实现高可用，确保每个Harbor 组件都有多于一个副本运行在 Kubernetes 集群中，当某个 Harbor 容器不可用时，Harbor 服务依然可正常使用。

为实现 Harbor 在 Kubernetes 集群中的高可用，Harbor 的大部分组件都是无状态组件。有状态组件的状态信息被保存在共享存储而非内存中。这样一来，在 Kubernetes 集群中只需配置组件的副本个数，即可借助Kubernetes平台实现高可用。（本文来自公众号：亨利笔记）

◎ Kubernetes平台通过协调调度（Reconciliation Loop）机制使Harbor各组件达到期望的副本数，从而实现服务的高可用。

◎ [PostgreSQL](https://cloud.tencent.com/product/postgresql?from_column=20065&from=20065)、[Redis](https://cloud.tencent.com/product/crs?from_column=20065&from=20065) 集群实现数据的高可用性、一致性和前端会话（session）的共享。

◎ 共享数据存储实现 Artifact 数据的一致性。

![](https://img.darklorder.com/img/202309041520461.png)

### **多Kubernetes集群的高可用架构**

上述介绍了使用 Harbor Helm Chart 在单个Kubernetes集群中搭建 Harbor 高可用环境的方案，其中实现了 Harbor 服务的高可用，但服务的整体可用性还是受到其运行所依赖的 Kubernetes 集群可用性的影响，如果集群崩溃，则会导致服务的不可用。在某些生产环境下会对可用性有更高的要求，因而基于多数据中心部署的多 Kubernetes 集群的高可用方案尤为重要。下面是在多个跨数据中心的 Kubernetes 集群上构建 Harbor 高可用环境的参考方案。

![](https://img.darklorder.com/img/202309041520571.png)

上图可以看到，Harbor 在两个数据中心分别拥有独立的数据和内容存储。在两个数据中心之间配置了 Harbor 自带的远程复制功能，实现了对 Artifact （制品） 数据的复制（如镜像复制）。也就是说，在两个 Kubernetes 集群的数据存储上，通过远程复制来保证 Artifact 的一致性。而对于两个数据中心之间的PostgreSQL 和 Redis 的数据一致性，这里需要用户基于不同类型的数据中心提供自己的数据备份方案，目的是保持两个数据中心的 PostgreSQL 和 Redis 数据的一致性。

本方案使用了 Harbor 主从（Active-Standby）模式，由于采用了镜像等 Artifact 远程复制，在数据同步上有一定的延时，在实际使用中需要留意对应用的影响。对实时性要求不高的用户，可参考此方案搭建跨数据中心多 Kubernetes 集群的高可用方案。

**注意：**在多次安装过程中都需要保证 values.yml 配置项 core.secretName 和core.xsrfKey 的值相同，其他配置项可根据不同数据中心的需求自行配置。

关于core.secretName 和 core.xsrfKey 值相同的具体原因，详见下文关于多 Harbor 实例之间需要共享的文件或者配置部分的内容。

## **2\. 基于离线安装包的高可用方案**

基于Kubernetes集群搭建的高可用架构是 Harbor 官方提供的方案。但用户可能出于某种原因无法部署独立的 Kubernetes 集群，更希望创建基于 Harbor 离线安装包的高可用方案。基于 Harbor 离线安装包搭建高可用系统是一项复杂的任务，需要用户具有高可用的相关技术基础，并深入了解 Harbor 的架构和配置。本节介绍的两种常规模式仅为参考方案，主要说明基于离线安装包实现高可用时，用户需要解决的问题和需要注意的地方。建议先阅读本章的其他内容，理解 Harbor 的安装及部署方式，在此基础上再结合各自的实际生产情况进行修改并实施。（本文来自公众号：亨利笔记）

### **（1）基于共享服务的高可用方案**

此方案的基本思想是多个 Harbor 实例共享 PostgreSQL、Redis 及存储，通过[负载均衡](https://cloud.tencent.com/product/clb?from_column=20065&from=20065)器实现多台服务器提供 Harbor 服务。

![](https://img.darklorder.com/img/202309041521513.png)

**重要配置：多个 Harbor 实例需要适当的配置才能工作，下面介绍相关原理，实际中用户可以灵活运用。**

**a）关于负载均衡器的设置**

需要设置每个 Harbor 实例的配置文件的 external\_url 项，把该项地址指定为负载均衡器的地址。通过该项指定负载均衡器的地址后，Harbor 将不再使用配置文件中的 hostname 作为访问地址。客户端（ Docker 和浏览器等）通过 external\_url 提供的地址（即负载均衡器的地址）访问后端服务的API。（本文来自公众号：亨利笔记）

如果不设置该值，则客户端会依据 hostname 的地址来访问后端服务的API，负载均衡在这里并没有起到作用。也就是说，服务访问并没有通过负载均衡直接到达后端，当后端地址不被外部识别时（如有NAT或防火墙等情况），服务访问还会失败。

如果 Harbor 实例使用了 HTTPS，特别是自持证书时，需要配置负载均衡器信任其后端每个 Harbor 实例的证书。同时，需要将负载均衡器的证书放置于每个Harbor实例中，其位置为 harbor.yml 配置项中 data\_volume 指定路径下的 “ca\_download” 文件夹中，该文件夹需要手动创建。这样，用户从任意 Harbor 实例的UI下载的证书就是负载均衡器的证书，如图所示。

![](https://img.darklorder.com/img/202309041521818.png)

**b）外置数据库的配置**

用户需要自行创建 PostgreSQL 共享实例或者集群，并将其信息配置到每个 Harbor 实例外置的数据库配置项中。注意：外置 PostgreSQL 数据库需要预先为Harbor Core、Clair、Notary Server及Notary Signer组件分别创建空数据库 registry、clair、notary\_server 及 notary\_singer ，并将创建的数据库信息配置到相应组件外置的数据库信息部分。Harbor 在启动时，会自动创建对应数据库的数据库表。（本文来自公众号：亨利笔记）

**c）外置Redis的配置**

用户需要自行创建Redis共享实例或者集群，并将其信息配置到每个 Harbor 实例外置的Redis配置项中。

**d）外置存储的配置**

用户需要提供本地或云端共享存储，并将其信息配置到每个 Harbor 实例的外置存储配置项中。

**e）多个 Harbor 实例之间需要共享的文件或者配置**

基于离线安装包安装的高可用方案需要保证以下文件在多个实例之间的一致性。同时，由于这些文件是在各个 Harbor 实例的安装过程中默认生成的，所以需要用户手动复制这些文件来保证一致性。

**private\_key.pem和root.crt文件**

Harbor在客户端认证流程中提供了证书和私钥文件供 Distribution 创建和校验请求中的Bearer token。在多实例 Harbor 的高可用方案中，多实例之间需要做到任何一个实例创建的Bearer token都可被其他实例识别并校验，也就是说，所有实例都需要使用**相同的** private\_key.pem 和 root.crt 文件。（本文来自公众号：亨利笔记）

如果多实例 Harbor 之间的这两个文件不同，在认证过程中就可能发生随机性的成功或失败。失败的原因是请求被负载均衡器转发到非创建该Bearer token的实例中，该实例无法解析非自身创建的token，从而导致认证失败。

private\_key.pem文件位于harbor.yml配置项data\_volume 指定路径的“secret/core”子目录下。root.crt 文件位于 harbor.yml 配置项 data\_volume 指定路径的“secret/registry”子目录下。

**csrf\_key**

为防止跨站攻击（Cross Site Request Forgery），Harbor 启用了 csrf 的 token 校验。Harbor 会生成一个随机数作为 csrf 的 token 附加在 cookie 中，用户提交请求时，客户端会从 cookie 中提取这个随机数，并将其作为 csrf 的 token 一并提交。Harbor 会依据这个值是否为空或者无效来拒绝该访问请求。那么，多实例之间需要做到任何一个实例创建的 token 都可被其他任意实例成功校验，也就是需要统一各个实例的 csrf token 私钥值。

该配置位于 Harbor 安装目录下的“common/config/core/env”文件中，用户需要把一个 Harbor 实例的值手动复制到其他实例上，使该值在所有实例上保持一致。

**注意：**手动修改以上文件或配置时，均需要通过 docker-compose 重启 Harbor 实例以使配置生效。另外，如果后续要使用 Harbor 安装包中的 prepare 脚本，则需要重复上述手动复制过程。（本文来自公众号：亨利笔记）

### **（2）基于复制策略的高可用方案**

此方案的基本思想是多个 Harbor 实例使用 Harbor 原生的远程复制功能实现Artifact 的一致性，通过负载均衡器实现多台服务器提供单一的 Harbor 服务。

![](https://img.darklorder.com/img/202309041521392.png)

方案(2)与方案(1)不同的是，在安装 Harbor 实例时不需要指定外置的 PostgreSQL、Redis 及存储，每个实例都使用自己独立的存储。Harbor 的多实例之间通过远程复制功能实现 Artifact 数据的一致性。关于 PostgreSQL 和 Redis 的数据一致性问题，需要用户自行实现数据同步的解决方案。基于复制的多实例解决方案，其实时性不如基于共享存储的方案，但相比之下搭建更为简单，用户使用 Harbor 离线安装包提供的 PostgreSQL、Redis 即可。



**参考资料**
[Harbor docs | Deploying Harbor with High Availability via Helm](https://goharbor.io/docs/edge/install-config/harbor-ha-helm/)
[一文读懂 Harbor 的高可用方案 | 腾讯云开发者社区](https://cloud.tencent.com/developer/article/1757879)
