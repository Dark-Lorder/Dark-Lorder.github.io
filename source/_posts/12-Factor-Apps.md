---
title: 12因素应用程序图解
date: 2023-08-16 12:03:46
categories:
- Cloud Native
tags:
- Cloud Native
---
云原生架构设计方法论——12因素应用程序图解，12 Factor
<!-- more -->


## 严重参考

[The Twelve-Factor App （简体中文） (12factor.net)](https://resources.sei.cmu.edu/asset_files/Presentation/2016_017_001_454665.pdf)

[An illustrated guide to 12 Factor Apps | Enable Architect (redhat.com)](https://www.redhat.com/architect/12-factor-app)

[Creating cloud-native applications: 12-factor applications – IBM Developer](https://developer.ibm.com/articles/creating-a-12-factor-application-with-open-liberty/)

[12 Factor App (hashnode.dev)](https://ngeninv.hashnode.dev/12-factor-app)

[12 Factor Apps: A Scorecard (cmu.edu)](https://resources.sei.cmu.edu/asset_files/Presentation/2016_017_001_454665.pdf)

## 前言

![](https://img.darklorder.com/img/202308161207731.jpeg)

12要素应用程序（12-Factor App）早在2011年就由 Adam Wiggins 发布。

12要素应用程序是一套原则，一种软件工程的方法论，产出的代码，能够**可靠地发布，快速的扩展，并以一致和可预知的方式**维护。

然而，这种方法论创建于十年前，云技术自最初创建以来已经取得了进步。

为了使应用程序能够**真正利用现代云基础设施和工具，并在云中蓬勃发展**，而不是仅仅在云中生存，Kevin Hoffman 修订了最初的12个因素，并增加了三个额外的因素——15因素应用程序方法论: [超越12因素应用程序](https://www.oreilly.com/library/view/beyond-the-twelve-factor/9781492042631/)。

12要素应用程序的原文，不太好理解。这里编译了一些资料，以图文的方式梳理如下。

## 概述

12要素应用程序是由 Heroku 的开发人员定义的一种方法论，用于构建软件即服务（SaaS）或者云原生（cloud-native）的应用程序:

-   使用**声明式（declarative）**格式自动化安装，这样可以最大限度地减少新加入项目的开发人员的时间和成本
-   与底层操作系统有一个**整洁的契约（clean contract）**，这样可以提供运行环境之间的**最大可移植性（maximum portability）**
-   适合在**现代云平台（cloud platform）**上**部署（deployment）**，这样可以除去服务器和系统管理；
-   在开发和生产环境之间**最小化分歧（minimize divergence）**，这样可以支持最高敏捷程度的**持续部署（continuous deployment）**
-   可以在不对工具、架构或开发实践进行重大更改的情况下进行**扩展（scale up）**

[12因素](https://12factor.net/)

1.  One codebase, one application - 一个代码库，一个应用程序
2.  Dependency management - 依赖管理
3.  Design, build, release, and run - 设计、构建、发布和运行
4.  Configuration, credentials, and code - 配置、证书和代码
5.  Logs - 日志
6.  Disposability - 易处理
7.  Backing services - 后端服务
8.  Environment parity - 环境等价
9.  Administrative processes - 管理进程
10.  Port binding - 端口绑定
11.  Stateless processes - 无状态进程
12.  Concurrency - 并发性

为了云原生应用程序而[新增加3个因素](https://developer.ibm.com/articles/15-factor-applications/)：

1.  API first - API 优先
2.  Telemetry - 遥测
3.  Authentication and authorization - 认证和授权

## 1\. Codebase

**1\. 基准代码**

> One codebase tracked in revision control, many deploys  
> 一份基准代码记录在版本控制中，多份部署

![](https://img.darklorder.com/img/202308161207746.jpeg)

**基准代码**原则：所有与应用程序相关的资产，包括源代码、配置脚本和配置设置，都存储在一个可供开发人员、测试人员和系统管理人员访问的源代码仓库中。

所有自动化脚本都可以访问源代码仓库，这是持续集成/持续交付（Continuous Integration/Continuous Delivery，CI/CD）过程的一部分，这些过程是企业软件开发生命周期（Software Development Lifecycle，SDLC）的一部分。

**云原生应用程序，必须始终由单个基准代码组成，且跟踪在版本控制系统中。**基准代码是一个源代码仓库或一组存储库，它们共享一个公共根，用于生成任意数量的不可变发行版。应用程序和基准代码之间应该存在**1:1**的关系，但应用程序的基准代码和部署之间应该存在**一对多**的关系。

这个单独的基准代码有助于支持开发团队之间的协作，并有助于实现应用程序的正确版本控制。

Git是目前最常见的版本控制工具之一。

## II. Dependencies

**2\. 依赖关系**

> Explicitly declare and isolate dependencies  
> 显式声明和隔离依赖关系

![](https://img.darklorder.com/img/202308161207939.jpeg)

**依赖关系**原则：只有与应用程序的用途唯一相关的代码存储在源代码控制系统中。

外部构件，如 Node.js 的 package、Java 的 Jar、NET 的 dll，在开发、测试和生产运行时，都应该引用自被加载到内存中的依赖关系清单。需要避免将构件和源代码一起存储在源代码仓库中。

**云原生应用程序永远不能依赖于系统级包的隐式存在。**这就是这个要素所关注的——鼓励显式声明和隔离应用程序的依赖关系。这有助于提供开发和生产环境之间的一致性，简化应用程序新手开发人员的设置，并支持云平台之间的可移植性。

大多数当代编程语言都有管理这些依赖关系的工具或工具。这些工具有助于简化依赖管理固有的复杂性，使开发人员能够声明他们的依赖关系，然后让工具负责实际确保这些依赖关系得到满足。

## III. Config

3\. **配置**

> Store configuration in the environment  
> 在环境中存储配置

![](https://img.darklorder.com/img/202308161207372.jpeg)

**配置**原则：配置信息以环境变量或独立配置文件中定义的设置，注入到各种运行环境中。在某些情况下，允许在代码中存储可以直接被覆盖的默认设置，例如端口号、依赖 URL，但是 DEBUG 等状态设置应该单独存在并在部署时进行覆盖。

外部配置文件的好例子是 Java properties文件、 Kubernetes manifest文件或 docker-compose.yml 文件。

**云原生应用程序不应该关心它在什么环境中运行，我们也不应该需要改变应用程序来在不同的环境中运行它。**将这些配置值存储在环境变量中被认为是外化此配置的最佳实践。这种方法有助于简化应用程序在多个环境中的部署，降低证书和密码泄漏的风险，并支持更有效的发布管理。

## IV. Backing Services

**4\. 后端服务**

> Treat backing services as attached resources  
> 将后端服务视为附加资源

![](https://img.darklorder.com/img/202308161207292.jpeg)

**后端服务**原则：鼓励架构师将外部组件，如数据库、电子邮件服务器、消息代理以及可由系统人员提供和维护的独立服务作为附加资源。将资源视为后端服务可以提高软件开发生命周期(SDLC)中的灵活性和效率。

**云原生应用程序应声明其对给定后端服务的需求，但允许云环境执行实际的资源绑定。**应用程序与其后端服务的绑定应该通过外部配置完成。应该可以随意附加和分离应用程序的后端服务，而不需要重新部署应用程序。应用程序中不应该有一行代码将应用程序与特定的后端服务紧密耦合。

将后端服务作为绑定资源使得云原生应用程序具有更大的灵活性和弹性，从而支持服务和部署之间的松散耦合。

## V. Build, Release, Run

**5\. 构建，发布，运行**

> Strictly separate build and run stages  
> 严格区分构建和运行阶段

![](https://img.darklorder.com/img/202308161207549.jpeg)

**构建、发布和运行**的原则：将部署过程分解为三个可复制的阶段，可以在任何时候进行实例化。

1.  **构建**阶段是从源代码管理系统检出代码，并构建/编译成存储在构件仓库(如 Docker Hub 或 Maven 仓库)中的构件的阶段。
2.  在编译代码之后，**发布**阶段应用配置设置。
3.  在**运行**阶段，使用 Ansible 之类的工具通过脚本提供一个执行运行环境。应用程序及其依赖关系被部署到新配置的运行环境中。

**构建、发布和运行**的关键是这个过程是完全瞬息的。如果流水线上的任何东西被破坏，所有的构件和环境都可以使用存储在源代码仓库中的资产从零再造。

**对于云原生应用程序而言，关键在于，每个部署阶段都是独立的，并且是单独发生的。**一旦运行，云运行时将负责其维护、健康和动态扩展。

## VI. Processes

**6\. Processes** **进程**

> Execute the app as one or more stateless processes  
> 将应用程序作为一个或多个无状态进程执行

![](https://img.darklorder.com/img/202308161207401.jpeg)

**进程**原则：可以更准确地称为无状态进程，在12要素应用程序结构下开发的应用程序将作为无状态进程的集合运行。这意味着没有单个进程跟踪另一个进程的状态，也没有进程跟踪会话或工作流状态等信息。

**无状态的云原生进程使扩展更加容易。**当流程是无状态的时候，可以添加和删除实例，以解决给定时间点的特定负载负担。由于每个进程都是独立运行的，无状态可以防止意想不到的副作用。

## VII. Port Binding

**Port binding** **端口绑定**

> Export services via port binding  
> 通过端口绑定发布服务

![](https://img.darklorder.com/img/202308161207767.jpeg)

**端口绑定**原则：服务或应用程序可以通过端口号而不是域名来给网络识别。其理由是，域名和相关的 IP 地址可以通过手动操作和自动服务发现机制动态分配。因此，用它们作为参考点是不可靠的。但是，根据端口号将服务或应用程序暴露给网络更可靠，也更容易管理。

端口绑定原则背后的基本思想是，统一使用端口号是向网络公开进程的最佳方式。

例如，已经出现的模式是端口80是传统的运行在 HTTP 下的 web 服务器，端口443是 HTTPS 的默认端口号，端口22是 SSH，端口3306是 MySQL 的默认端口，端口27017是 MongoDB 的默认端口。

**云原生应用程序应该使用端口绑定输出服务。**

## VIII. Concurrency

**Concurrency** **并发性**

> Scale-out via the process model  
> 通过进程模型横向扩展

![](https://img.darklorder.com/img/202308161207801.jpeg)

**并发**原则：建议根据进程的用途来组织进程，然后将这些进程分开，以便可以根据需要扩展和缩小（scale up and down）这些进程。

如上图所示，应用程序通过在负载均衡器后面运行的 web 服务器向网络公开。负载均衡器后面的 web 服务器组，依次使用业务服务流程中的业务逻辑，这些业务逻辑在自己的负载均衡器后面运行。如果 web 服务器的负载增加，可以单独扩大这个小组，以满足眼前的需求。但是，如果由于业务服务的负载而出现瓶颈，则可以独立地扩展该层。

支持并发意味着应用程序的不同部分可以扩展以满足眼前的需要。否则，当不支持并发时，架构除了垂直扩展整个应用程序之外别无选择。

**云原生应用程序的一种云支持的弹性可伸缩性的理想方法，是向外扩展，或者水平扩展。**与其让单个大进程变得更大，不如创建多个进程，并将应用程序的负载分配给这些进程。

## IX. Disposability

易处理

> Maximize robustness with fast start-up and graceful shutdown  
> 快速启动和优雅终止可最大化稳健性

![](https://img.darklorder.com/img/202308161207800.jpeg)

看待这个问题的一种方式是牛和宠物的对比模型。应用实例应该更多地被视为牛(即，对它们没有情感上的依恋，相当容易替换，编号没有命名等) ，而不是宠物(即，有情感上的依恋，健康恢复护理而不是替换，等等)。

**易处理**原则：应用程序应该优雅地启动和停止。这意味着在使用者可以访问应用程序之前，必须完成所有必需的“家务活”。例如，一个优雅的启动将确保所有数据库连接和对其他网络资源的访问都是可使用的。此外，需要进行的任何其他配置工作都已经完成。

在关机方面，易处理倡导者确保所有数据库连接和其他网络资源正确终止，并确保所有关机活动都被记录下来，如下面的代码示例所示。

```
const shutdown = async (signal) => {
   logger.info(`Disconnecting message broker at ${new Date()}`);
   messageBroker.disconnect();

   logger.info(`Disconnecting database at ${new Date()}`);
   database.disconnect();

   let shutdownMessage;

   if (!signal) {
       shutdownMessage = (`MyCool service shutting down at ${new Date()}`);
   } else {
       shutdownMessage = (`Signal ${signal} : MyCool service shutting down at ${new Date()}`);
   }
   const obj = {status: "SHUTDOWN", shutdownMessage, pid: process.pid};
   await server.close(() => {
       console.log(obj);
       process.exit(0);
   });
};
```

云原生应用程序的进程必须是易处理的，这意味着它们可以快速启动或停止。如果应用程序不能快速启动并优雅地关闭，那么它就不能迅速扩展、部署、发布或恢复。如果启动一个应用程序，并且需要几分钟才能进入稳定状态，在当今流量高的世界中，这可能意味着在应用程序启动时，成百上千的请求被拒绝。

无法足够快速地关闭也可能带来无法处理资源的风险，这可能会破坏数据。

## X. Dev/Prod Parity

**10\. 开发与生产环境等价**

> Keep development, staging, and production as similar as possible  
> 尽可能保持开发、预发布和生产环境相同

![](https://img.darklorder.com/img/202308161207186.jpeg)

**开发与生产环境等价**原则：所有的部署路径都是相似的但又是独立的，并且没有部署“蛙跳”到另一个部署目标。

上图显示了应用程序代码的两个版本。V1版本的目标是发布到生产环境。新版本 V2的目标是一个开发环境。V1和 V2遵循类似的部署路径，从**构建**到**发布**，然后**运行**。如果代码的 V2版本已经准备好投入生产，那么与 V2相关的构件和设置将**不会**被复制到生产环境中（图中红**X**）。

相反，CI/CD 流程将被调整以设置 V2到 生产环境的部署目标。CI/CD 流程将遵循预期的针对新目标的**构建、发布和运行**模式。

**开发与生产环境等价**与**构建、发布和运行**非常相似。重要的区别在于**开发与生产环境等价**确保了生产部署过程与开发部署过程相同。这样就可以确保所有潜在的错误/故障都可以在开发和测试中识别出来，而不是在应用程序投入生产时识别出来。

像 Docker 这样的工具可以帮助实现这种开发/测试/生产环境的等价。容器的好处是它为运行代码创建和使用相同的映像，提供了绝对统一的环境。它还有助于确保在每个环境中使用相同的后端服务。

## XI. Logs

**11\. Logs** **日志**

> Treat logs as event streams  
> 将日志视为事件流

![](https://img.darklorder.com/img/202308161207315.jpeg)

**日志**原则：将日志数据发送到一个流中，各种感兴趣的消费者可以访问这个流。路由日志数据的过程需要与处理日志数据分开。

例如，一个消费者可能只对 Error 数据感兴趣，而另一个消费者可能对 Request/Response 数据感兴趣。另一个消费者可能对存储用于事件归档的所有日志数据感兴趣。

一个额外的好处是，即使一个应用程序消亡，日志数据在之后仍然存在。

云原生应用程序的日志聚合、处理和存储，是云提供商或其他工具套件(例如，ELK 技术栈、 Splunk 等)的职责，这些工具套件与正在使用的云平台一起运行。**通过简化应用程序在日志聚合和分析中的部分，可以简化应用程序的代码库，并更多地关注业务逻辑。**

## XII. Admin Processes

**管理进程**

> Run admin/management tasks as one-off processes  
> 将管理任务作为一次性进程运行

![](https://img.darklorder.com/img/202308161207330.jpeg)

**管理进程**原则：管理进程是软件开发生命周期中的一等公民，需要被这样对待。

上图显示了作为 Docker 容器部署的名为 Orders 的服务。此外，还有一个管理服务名为 dataSeeder，可以将数据种子化到 Orders 服务中。服务 dataSeeder 是一个管理进程，用于与 Orders 一起使用，如下图所示。

![](https://img.darklorder.com/img/202308161207998.jpeg)

但是，即使 dataSeeder 是一个管理进程，它也会被赋予一个类似于 Orders 服务的 Build、 Release 和 Run 部署路径。此外，它是根据**基准代码**和**开发与生产环境等价**的原则发布的。**管理进程** dataSeeder **与整个 SDLC 不是分离的，而是它的一部分。**

## XIII. API first

**13\. API优先**

![](https://img.darklorder.com/img/202308161208639.jpeg)

**API优先**原则：应用程序通常会成为服务生态系统的参与者。但是，如果应用程序中没有明确定义 API，这可能会导致在这个生态系统中的集成失败。

API 优先方法包括开发一致且可重用的 API，使团队能够针对彼此的公共契约进行工作，而不会干扰内部开发过程。通过使用 API 优先的方法，并清楚地规划客户端应用程序和服务将使用的各种 API，每个 API 都可以被设计得尽可能有效，并且可以很容易地进行模拟。

API 描述语言可以帮助建立一个关于 API 应该如何行为的契约。API 描述语言是领域特定语言（DSL），这适合于描述 API。与编程语言或 API 实现语言相比，API 描述语言使用更高层次的抽象和声明性范式，这意味着可以使用它们来帮助表达“什么”而不是“如何”。

**API 的引入是为了强调 API 在云原生应用程序开发中的重要性**。为云开发的应用程序通常是分布式服务生态系统的参与者，如果没有明确定义 API，这可能会导致集成失败的噩梦。因此，在设计在云中蓬勃发展的应用程序时，这个因素非常重要。

## XIV. Telemetry

**14\. 遥测**

![](https://img.darklorder.com/img/202308161208382.jpeg)

**遥测**原则：遥测让云端的应用程序及其行为具有更深层次的可见性。运行在云端的应用程序实例可能在很少或没有警告的情况下移动到世界的任何地方。除此之外，可能开始时只有应用程序的一个实例，几分钟后，您可能已经运行了数百个应用程序副本。

遥测技术可以包括领域特定的指标，以及应用程序的健康和系统指标。健康度量和系统度量包括应用程序启动、关闭、伸缩、 web 请求跟踪和定期健康检查的结果。

**云原生应用程序的遥测技术是日志记录的重要补充**。遥测技术的重点是数据收集。遥测和实时应用程序监控使开发人员能够在这个复杂和高度分布式的环境中监控应用程序的性能、健康状况和关键指标。

## XV. Authentication and authorization

**15\. 认证和授权**

![](https://img.darklorder.com/img/202308161208669.jpeg)

**认证和授权**原则：安全性是任何应用程序和云环境的重要组成部分。

云原生应用程序可以使用基于以角色为基础的访问控制(role-based access control，RBAC)来保护其端点。这些角色规定调用的客户端是否具有足够的权限，以便应用程序能够响应请求，并为审计需要帮助跟踪发出请求的人。

**认证和授权要素的添加是为了强调云原生应用程序的安全性**。在云环境中部署应用程序意味着应用程序可以跨越世界各地的许多数据中心传输，在多个容器中执行，并由几乎无限数量的客户端访问。

不管这个应用程序是为企业、移动设备还是云服务而设计的，安全性都不应该是事后才考虑的问题。

## 12因素应用程序打分表

最后，是一份使用12因素评估应用程序的打分表，看看你的应用程序有几分。


| 12因素                 | 打分（分数越高，越符合）                                     |
| ---------------------- | ------------------------------------------------------------ |
| 基准代码               | 1. 使用电子邮件发送不同名称的src zip <br />2. 频繁地提交到源代码控制系统（所有代码使用同一个仓卡）  <br />3. 应用程序拆分成独立的部分，每个部分都有自有的仓库 |
| 依赖管理               | 1. 手动下载jar到/lib。  <br />2. 使用软件包管理器（mvn、npm），期望提供工具(curl)  <br />3. 使用 artifact 管理器（Artifactory），捆绑依赖项和工具 |
| 配置、证书和代码       | 1. 硬代码 URL，密码在代码中，使用像if(Mode.PROD) 的代码  <br />2. 使用配置文件，不同环境的多个配置文件  <br />3. 使用配置服务（Spring Cloud Config、Zookeeper） |
| 后端服务               | 1. 供应商特定的连接库，硬代码连接字符串  <br />2. 连接参数存在在配置文件中  <br />3. 动态发现资源，独立地更新后端服务 |
| 设计、构建、发布和运行 | 1. 开发从本地构建和部署代码，生产环境由手动推送  <br />2. 使用构建/发布工具(Jenkins，TravisCI)，清晰地分离构建和部署步骤  <br />3. 有一键点击发布流水线，每个发布都用版本控制并保存用于回滚，没有人工干预 |
| 无状态进程             | 1. 粘性会话，将应用程序数据写入到本地文件系统  <br />2. 不依赖于本地存储的数据  <br />3. 无状态，将会话数据存储在数据存储中(redis)，缓存中间阶段的事务步骤 |
| 端口绑定               | 1. 部署到应用程序容器  <br />2. 单机的，但监听特定的端口  <br />3. Web服务器是应用程序的一部分(node，netty)，应用程输出HTTP服务 |
| 并发性                 | 1. 必须顺序运行的阻塞式任务  <br />2. 非阻塞式IO服务器（node、netty） <br />3. 水平扩展，小型、独立的微服务 |
| 易处理                 | 1. 需要一个开发人员来协调重启  <br />2. 快速启动  <br />3. 优雅地崩溃，低于1秒重新启动，存储状态以快速恢复 |
| 环境等价               | 1. 开发人员对生产环境没有了解，开发环境不同于生产环境  <br />2. 用轻量级替换品（内存H2、SQLite）替代  <br />3. 环境是相同的 |
| 日志                   | 1. System.out.print()  <br />2. 写入到web服务器上的日志文件  <br />3. 将日志当作流来处理(ELK) |
| 管理进程               | 1. 手动编辑数据库条目  <br />2. 将迁移脚本存储在仓库中  <br />3. 使用框架的工具链 |



**参考资料**
[云原生架构设计方法论——12因素应用程序图解，12 Factor - 知乎](https://zhuanlan.zhihu.com/p/452532308)

