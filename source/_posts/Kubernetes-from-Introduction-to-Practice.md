---
title: Kubernetes从入门到实践
date: 2023-10-01 00:00:00
categories:
- Kubernetes
tags:
- Kubernetes
---

<!-- more -->

### 

第一部分为系统基础（第1～2章）， 介绍Kubernetes系统基础概念及基本应用。
    第1章介绍容器编排系统出现的背景，以及Kubernetes系统的功能、特性、核心概念、系统组件及应用模型。
    第2章讲解Kubernetes的部署模式，包括kubeadm部署工具的部署方式及部署过程，并给出了使用直接命令式操作管理资源对象的方法，以帮助读者快速入门。
第二部分为核心资源（第3～8章）， 介绍各种核心资源及应用。
    第3章介绍资源管理模型以及命令式和声明式管理接口，并通过命令对比说明两种操作方式的不同。
    第4章介绍Pod资源的常用配置、生命周期、存储状态和就绪状态检测，以及计算资源的需求与限制等。
    第5章主要介绍存储卷类型及常见存储卷的使用方式，PV和PVC出现的原因与应用，以及存储类资源的应用与存储卷的动态供给。
    第6章介绍使用ConfigMap和Secret资源为容器应用提供配置及敏感信息的方式。
    第7章讲解Service资源，分别介绍了Service类型、功用及其实现，并深入分析了各种类型Service的实现方式。
    第8章介绍Pod控制器资源类型，重点讲解了控制无状态应用的ReplicaSet、Deployment、StatefulSet和DaemonSet控制器，并介绍了Job和CronJob控制器。
第三部分为安全（第9～10章）， 介绍安全相关的话题，主要涉及认证、授权、准入控制、网络模型及网络策略等。
    第9章重点讲解认证方式、ServiceAccount和TLS认证、授权插件类型及RBAC，并在最后介绍了LimitRange、ResourceQuota和PodSecurityPolicy这3种类型的准入控制器及相关的资源类型。
    第10章主要介绍网络插件基础及Flannel的3种后端实现及其应用，Calico网络插件IPIP和BGP模型的实现，以及网络策略的实现及应用。
第四部分为进阶（第11～13章），主要介绍调度框架和调度插件、资源扩展和路由网关等高级话题。
    第11章介绍Pod资源的经典调度策略、新式的调度框架及调度插件，包括节点亲和、Pod资源亲和及基于污点与容忍度的调度等话题。
    第12章介绍系统资源的扩展方式，包括自定义资源类型、自定义资源对象、自定义API及控制器、Master节点的高可用等话题。
    第13章介绍Ingress资源及其实现、Ingress Nginx配置与应用案例，以及基于Contour的高级应用发布机制，例如蓝绿部署、流量迁移、流量镜像、超时和重试等。
第五部分为必备生态组件（第14～16章），重点介绍Kubernetes上用于支撑核心功能的关键附加组件。
    第14章讲解大规模应用部署管理工具Kustomize与Helm的基础及应用案例。
    第15章介绍资源指标、自定义指标、Prometheus监控系统及HPA控制器的应用。
    第16章介绍如何为Kubernetes系统提供统一日志收集及管理工具栈EFK。


**参考资料**
[Kubernetes 文档 | Kubernetes](https://kubernetes.io/zh-cn/docs/home/)
[Kubernetes进阶实战](https://book.douban.com/subject/30435129/)
