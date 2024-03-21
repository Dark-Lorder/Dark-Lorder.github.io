---
title: Kubernetes Pod概述
date: 2022-10-03 00:00:00
categories:
- Kubernetes
tags:
- Kubernetes
---
Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。
<!-- more -->

#### 关于Pod 

<img src="https://img.darklorder.com/img/202112062209720.jpeg"/>

#### Yaml文件格式

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



#### [镜像拉取策略](https://kubernetes.io/zh-cn/docs/concepts/containers/images/#updating-images)
```shell
[root@k8s-master01 ~]# kubectl explain pods.spec.containers.imagePullPolicy
KIND:     Pod
VERSION:  v1

FIELD:    imagePullPolicy <string>

DESCRIPTION:
     Image pull policy. One of Always, Never, IfNotPresent. Defaults to Always
     if :latest tag is specified, or IfNotPresent otherwise. Cannot be updated.
     More info:
     https://kubernetes.io/docs/concepts/containers/images#updating-images
[root@k8s-master01 ~]# 
```
**IfNotPresent**
只有当镜像在本地不存在时才会拉取。
**Always**
每当 kubelet 启动一个容器时，kubelet 会查询容器的镜像仓库， 将名称解析为一个镜像摘要。 如果 kubelet 有一个容器镜像，并且对应的摘要已在本地缓存，kubelet 就会使用其缓存的镜像； 否则，kubelet 就会使用解析后的摘要拉取镜像，并使用该镜像来启动容器。
**Never**
Kubelet 不会尝试获取镜像。如果镜像已经以某种方式存在本地， kubelet 会尝试启动容器；否则，会启动失败。 更多细节见提前拉取镜像。


#### [Pod的相位](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)

##### Pod phase

![](https://img.darklorder.com/img/202112062216597.png)

Pod 的 status 字段是一个 PodStatus 对象，其中包含一个 phase 字段。
Pod phase的数量和含义是严格定义的。 除了本文档中列举的内容外，不应该再假定 Pod 有其他的 phase 值。

| 状态 |    取值     | 描述                                                         |
|:----:|:-----------:|:-------------------------------------------------------------|
| 等待 |  `Pending`  | Pod 已被 Kubernetes 系统接受，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间。 |
| 运行 |  `Running`  | Pod 已经绑定到了某个节点，Pod 中所有的容器都已被创建。至少有一个容器仍在运行，或者正处于启动或重启状态。 |
| 成功 | `Succeeded` | Pod 中的所有容器都已成功终止，并且不会再重启。               |
| 失败 |  `Failed`   | Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止。 |
| 未知 |  `Unknown`  | 因为某些原因无法取得 Pod 的状态。这种情况通常是因为与 Pod 所在主机通信失败。 |

> **说明：**
> 当一个 Pod 被删除时，执行一些 kubectl 命令会展示这个 Pod 的状态为 `Terminating`（终止）。 这个 `Terminating` 状态并不是 Pod 阶段之一。 Pod 被赋予一个可以体面终止的期限，默认为 30 秒。 你可以使用 `--force` 参数来[强制终止 Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination-forced)。

从 Kubernetes 1.27 开始，除了静态 Pod 和没有 Finalizer 的强制终止 Pod 之外，kubelet 会将已删除的 Pod 转换到终止阶段 （Failed 或 Succeeded 具体取决于 Pod 容器的退出状态），然后再从 API 服务器中删除。

如果某节点死掉或者与集群中其他节点失联，Kubernetes 会实施一种策略，将失去的节点上运行的所有 Pod 的 phase 设置为 Failed。


##### 容器状态

![](https://img.darklorder.com/img/202112062217881.png)

一旦调度器将 Pod 分派给某个节点，kubelet 就通过容器运行时开始为 Pod 创建容器。容器的状态有三种：Waiting（等待）、Running（运行中）和 Terminated（已终止）。

**`Waiting` （等待）**

如果容器并不处在 `Running` 或 `Terminated` 状态之一，它就处在 `Waiting` 状态。 处于 `Waiting` 状态的容器仍在运行它完成启动所需要的操作：例如， 从某个容器镜像仓库拉取容器镜像，或者向容器应用 [Secret](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/) 数据等等。 当你使用 `kubectl` 来查询包含 `Waiting` 状态的容器的 Pod 时，你也会看到一个 Reason 字段，其中给出了容器处于等待状态的原因。

**`Running`（运行中）**

`Running` 状态表明容器正在执行状态并且没有问题发生。 如果配置了 `postStart` 回调，那么该回调已经执行且已完成。 如果你使用 `kubectl` 来查询包含 `Running` 状态的容器的 Pod 时， 你也会看到关于容器进入 `Running` 状态的信息。

**`Terminated`（已终止）**

处于 `Terminated` 状态的容器已经开始执行并且或者正常结束或者因为某些原因失败。 如果你使用 `kubectl` 来查询包含 `Terminated` 状态的容器的 Pod 时， 你会看到容器进入此状态的原因、退出代码以及容器执行期间的起止时间。

如果容器配置了 `preStop` 回调，则该回调会在容器进入 `Terminated` 状态之前执行。


##### 容器重启策略

Pod 的 `spec` 中包含一个 `restartPolicy` 字段，其可能取值包括 Always、OnFailure 和 Never。默认值是 Always。

`restartPolicy` 适用于 Pod 中的所有容器。`restartPolicy` 仅针对同一节点上 `kubelet` 的容器重启动作。当 Pod 中的容器退出时，`kubelet` 会按指数回退方式计算重启的延迟（10s、20s、40s、...），其最长延迟为 5 分钟。 一旦某容器执行了 10 分钟并且没有出现问题，`kubelet` 对该容器的重启回退计时器执行重置操作。

- Always：无论何种exit code，都要重启容器
- OnFailure：仅在exit code为非0值（即错误退出）时才重启容器
- Never：无论何种exit code，都不重启容器



#### [Pod的健康状态监测机制](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
probe 是由 kubelet 对容器执行的定期诊断。 要执行诊断，kubelet 既可以在容器内执行代码，也可以发出一个网络请求。

<img src="https://img.darklorder.com/img/202112071120419.webp" alt="img" style="zoom:50%;" />


##### 检查机制

使用探针来检查容器有四种不同的方法。 每个探针都必须准确定义为这四种机制中的一种：

- `exec`

  在容器内执行指定命令。如果命令退出时返回码为 0 则认为诊断成功。

- `grpc`

  使用 [gRPC](https://grpc.io/) 执行一个远程过程调用。 目标应该实现 [gRPC 健康检查](https://grpc.io/grpc/core/md_doc_health-checking.html)。 如果响应的状态是 "SERVING"，则认为诊断成功。

- `httpGet`

  对容器的 IP 地址上指定端口和路径执行 HTTP `GET` 请求。如果响应的状态码大于等于 200 且小于 400，则诊断被认为是成功的。

- `tcpSocket`

  对容器的 IP 地址上的指定端口执行 TCP 检查。如果端口打开，则诊断被认为是成功的。 如果远程系统（容器）在打开连接后立即将其关闭，这算作是健康的。

> **注意：** 和其他机制不同，`exec` 探针的实现涉及每次执行时创建/复制多个进程。 因此，在集群中具有较高 pod 密度、较低的 `initialDelaySeconds` 和 `periodSeconds` 时长的时候， 配置任何使用 exec 机制的探针可能会增加节点的 CPU 负载。 这种场景下，请考虑使用其他探针机制以避免额外的开销。


##### 探测结果

每次探测都将获得以下三种结果之一：

- `Success`（成功）

  容器通过了诊断。

- `Failure`（失败）

  容器未通过诊断。

- `Unknown`（未知）

  诊断失败，因此不会采取任何行动。

##### 探测类型

针对运行中的容器，`kubelet` 可以选择是否执行以下三种探针，以及如何针对探测结果作出反应：

- `livenessProbe` (存活探针)

  指示容器是否正在运行。如果存活态探测失败，则 kubelet 会杀死容器， 并且容器将根据其[重启策略](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)决定未来。如果容器不提供存活探针， 则默认状态为 `Success`。

- `readinessProbe` (就绪探针)

  指示容器是否准备好为请求提供服务。如果就绪态探测失败， 端点控制器将从与 Pod 匹配的所有服务的端点列表中删除该 Pod 的 IP 地址。 初始延迟之前的就绪态的状态值默认为 `Failure`。 如果容器不提供就绪态探针，则默认状态为 `Success`。

- `startupProbe`  (初始化探针)

  指示容器中的应用是否已经启动。如果提供了启动探针，则所有其他探针都会被 禁用，直到此探针成功为止。如果启动探测失败，`kubelet` 将杀死容器， 而容器依其[重启策略](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)进行重启。 如果容器没有提供启动探测，则默认状态为 `Success`。


##### 配置探针

[Probe](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#probe-v1-core) 有很多配置字段，可以使用这些字段精确地控制启动、存活和就绪检测的行为：

- `initialDelaySeconds`：容器启动后要等待多少秒后才启动启动、存活和就绪探针。 如果定义了启动探针，则存活探针和就绪探针的延迟将在启动探针已成功之后才开始计算。 如果 `periodSeconds` 的值大于 `initialDelaySeconds`，则 `initialDelaySeconds` 将被忽略。默认是 0 秒，最小值是 0。
- `periodSeconds`：执行探测的时间间隔（单位是秒）。默认是 10 秒。最小值是 1。
- `timeoutSeconds`：探测的超时后等待多少秒。默认值是 1 秒。最小值是 1。
- `successThreshold`：探针在失败后，被视为成功的最小连续成功数。默认值是 1。 存活和启动探测的这个值必须是 1。最小值是 1。

- `failureThreshold`：探针连续失败了 `failureThreshold` 次之后， Kubernetes 认为总体上检查已失败：容器状态未就绪、不健康、不活跃。 对于启动探针或存活探针而言，如果至少有 `failureThreshold` 个探针已失败， Kubernetes 会将容器视为不健康并为这个特定的容器触发重启操作。 kubelet 遵循该容器的 `terminationGracePeriodSeconds` 设置。 对于失败的就绪探针，kubelet 继续运行检查失败的容器，并继续运行更多探针； 因为检查失败，kubelet 将 Pod 的 `Ready` [状况](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions)设置为 `false`。

- `terminationGracePeriodSeconds`：为 kubelet 配置从为失败的容器触发终止操作到强制容器运行时停止该容器之前等待的宽限时长。 默认值是继承 Pod 级别的 `terminationGracePeriodSeconds` 值（如果不设置则为 30 秒），最小值为 1。 更多细节请参见[探针级别 `terminationGracePeriodSeconds`](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#probe-level-terminationgraceperiodseconds)。

> **注意：** 如果就绪态探针的实现不正确，可能会导致容器中进程的数量不断上升。 如果不对其采取措施，很可能导致资源枯竭的状况。




**参考资料**
[Pod | Kubernetes](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/)
[Kubernetes API 参考文档](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#api-overview)

