---
title: 云计算（Cloud Computing）
date: 2022-08-05 00:00:00
categories: Cloud Computing
tags:
- Cloud Computing
---
关于云计算（Cloud Computing） 看懂云计算、虚拟化和容器，这一篇就够啦！
<!-- more -->

## 云计算这个词，相信大家都非常熟悉了。

云计算的准确定义，有比较多的先驱定义过，具体如下：

**Gartner公司**将云计算定义为：「一种计算方式，能够通过Internet技术将可扩展的和弹性的IT能力作为服务交付给外部用户。」

**Forrester Research公司**，将云计算定义为：「一种标准化的IT性能（服务，软件或者基础设施），以按使用付费和自助服务方式，通过Internet技术进行交付。」

**NIST（美国国家标准和技术研究院）**，将云计算定义为：「云计算是一种模型，可以随时随地，便捷地，按需地从可配置计算资源共享池中获取所需的资源（例如，网络，服务器，存储，应用程序及服务），资源可以快速供给和释放，使管理的工作量和服务提供者的介入降低至最少。这种云模型，包括五个基本特征，三种服务模型和四种部署模型。」

“云计算”这个词，相信大家都非常熟悉。

作为信息科技发展的主流趋势，它频繁地出现在我们的眼前。伴随它一起出现的，还有这些概念名词——OpenStack、Hypervisor、KVM、Docker、K8S...

<center><img src="https://img.darklorder.com/img/202305112240666.jpg"  alt="文件名" style="zoom:50%;" /></center>

这些名词概念，全部都属于云计算技术领域的范畴。

对于初学者来说，理解这些概念的具体含义并不是一件容易的事情。

所以，今天这篇文章，将给大家做一个通俗易懂的介绍，解释这些云计算概念以及它们之间的关系，希望对大家入门有所帮助。

### 什么是云计算

在介绍那些古怪名词之前，我先介绍一下**云计算。**

以前电脑被发明的时候，还没有网络，每个电脑（PC），就是一个单机。

<center><img src="https://img.darklorder.com/img/202305112242464.jpg"  alt="文件名" style="zoom:50%;" /></center>

这台单机，包括CPU、内存、硬盘、显卡等硬件。用户在单机上，安装操作系统和应用软件，完成自己的工作。

后来，有了**网络（Network），**单机与单机之间，可以交换信息，协同工作。

<center><img src="https://img.darklorder.com/img/202305112243322.jpg"  alt="文件名" style="zoom:50%;" /></center>

再后来，单机性能越来越强，就有了**服务器（Server）。**人们发现，可以把一些服务器集中起来，放在机房里，然后让用户通过网络，去访问和使用机房里的计算机资源。

<center><img src="https://img.darklorder.com/img/202305112243607.jpg"  alt="文件名" style="zoom:50%;" /></center>

再再后来，小型网络变成了大型网络，就有了**互联网（Internet）。**小型机房变成了大型机房，就有了**IDC（Internet Data Center，互联网数据中心）。**

当越来越多的计算机资源和应用服务（Application，例如看网页，下电影）被集中起来，就变成了——**“云计算（Cloud Computing）”。**无数的大型机房，就成了“云端”。

<center><img src="https://img.darklorder.com/img/202305112244542.jpg"  alt="文件名" style="zoom:50%;" /></center>

是不是觉得太简单？别急，开始深入。

云计算的道理是简单的，说白了，就是把计算机资源集中起来，放在网络上。但是，云计算的实现方式，就非常复杂了。

举个例子，如果你只是在公司小机房摆了一个服务器，开个FTP下载服务，然后用于几个同事之间的电影分享，当然是很简单的。

如果是“双11”的淘宝购物节，全球几十亿用户访问阿里巴巴的淘宝网站，单日几十**PB（1PB=1024TB=1024×1024GB）**的访问量，每秒几百**GB**的流量……这个，就不是几根网线几台服务器能解决的了。

这时，需要设计一个**超大容量、超高并发（同时访问）、超快速度、超强安全**的云计算系统，才能满足业务平稳运行的要求。

这才是云计算的复杂之处。

刚才说了，我们把计算机资源放在云端。这个计算机资源，实际上分为好几种层次：

**第一层次，**是最底层的硬件资源，主要包括CPU（计算资源），硬盘（存储资源），还有网卡（网络资源）等。

**第二层次，**要高级一些，我不打算直接使用CPU、硬盘、网卡，我希望你把操作系统（例如Windows、Linux）装好，把数据库软件装好，我再来使用。

**第三层次，**更高级一些，你不但要装好操作系统这些基本的，还要把具体的应用软件装好，例如FTP服务端软件、在线视频服务端软件等，我可以直接使用服务。

这三种层次，就是大家经常听到的**IaaS、Paas、SaaS。**

<center><img src="https://img.darklorder.com/img/202305112246787.jpg"  alt="文件名" style="zoom:50%;" /></center>

SaaS: Software-as-a-Service（软件即服务）
PaaS: Platform-as-a-Service（平台即服务）
IaaS: Infrastructure-as-a-Service（基础设施即服务）

再补一张图，可能更直观：

<center><img src="https://img.darklorder.com/img/202305112246104.jpg"  alt="文件名" style="zoom:50%;" /></center>

目前主流的云计算服务提供商，例如亚马逊AWS、阿里云、华为云、天翼云、腾讯云，说白了，都是为大家提供以上三个层次的云资源。你想要什么，它们就提供什么。你想要多少，它们就提供多少。

<center><img src="https://img.darklorder.com/img/202305112247813.jpg"  alt="文件名" style="zoom:50%;" /></center>

继续，继续。

这么多样化多层次的云计算服务，阿里、华为们又是怎么提供的呢？

难道说，是人工安排？——

如果你要八核CPU、16GB内存、500GB硬盘的服务器，阿里临时安排工程师帮你组装？如果你要装CentOS 7.2（一种类Linux操作系统），MySQL 5.5.60（一种数据库系统），阿里也临时让工程师帮你安装配置？

这显然是不可能的，耗不起人力，也等不起时间。

**于是，就有了各种软件和平台，负责对资源进行快速调用和集中管理。**

### 什么是虚拟化

如果要对物理资源进行管理，第一步，就是**“虚拟化”。**

虚拟化是云计算的基础。简单来说，虚拟化就是在一台物理服务器上，运行多台“虚拟服务器”。这种虚拟服务器，也叫**虚拟机（VM，Virtual Machine）。**

从表面来看，这些虚拟机都是独立的服务器，但实际上，它们共享物理服务器的CPU、内存、硬件、网卡等资源。

<center><img src="https://img.darklorder.com/img/202305112248949.png"  alt="文件名" style="zoom:50%;" /></center>

物理机，通常称为“宿主机（Host）”。虚拟机，则称为“客户机（Guest）”。

谁来完成物理资源虚拟化的工作呢？

就是大名鼎鼎的 **Hypervisor 。**

Hypervisor，汉译过来是“超级监督者”，也叫做VMM（Virtual Machine Monitor，虚拟机监视器）。它不是一款具体的软件，而是一类软件的统称。

Hypervisor分为两大类：

第一类，hypervisor直接运行在物理机之上。虚拟机运行在hypervisor之上。第二类，物理机上安装正常的操作系统（例如Linux或Windows），然后在正常操作系统上安装hypervisor，生成和管理虚拟机。

<center><img src="https://img.darklorder.com/img/202305112249807.jpg"  alt="文件名" style="zoom:50%;" /></center>

像**VMware、KVM、Xen、Virtual Box，**都属于Hypervisor。

VMware大家应该很熟悉，就是VMware Workstation。学习Linux的话，很多人都是在windows系统下安装WMware，然后创建Linux虚拟机。

<center><img src="https://img.darklorder.com/img/202305112250795.jpg"  alt="文件名" style="zoom:50%;" /></center>

但是，真正厉害的，是但是，真正厉害的，是 **KVM（kernel-based virtual machine，基于Linux内核的虚拟机）。**它是目前最热门最受追捧的虚拟化方案。

<center><img src="https://img.darklorder.com/img/202305112251358.jpg"  alt="文件名" style="zoom:50%;" /></center>

KVM这样的Hypervisor软件，实际上是提供了一种虚拟化能力，模拟CPU的运行，更为底层。但是它的用户交互并不良好，不方便使用。

于是，为了更好地管理虚拟机，就需要OpenStack这样的云管理平台。

<center><img src="https://img.darklorder.com/img/202305112252953.jpg"  alt="文件名" style="zoom:50%;" /></center>

关于OpenStack，我之前曾经介绍过（[链接](https://mp.weixin.qq.com/s?__biz=MzI1NTA0MDUyMA==&mid=2456660029&idx=1&sn=1a900a0c45ff77355693e7902c3d8f38&chksm=fda5055acad28c4c34d015b72843ef14c5529f81651cf38daddd1ce4d7e595c9f937f1ee3327&scene=21#wechat_redirect)）。它有点像个商店，负责管理商品（计算资源、存储资源、网络资源等），卖给用户，但它本身不制造商品（不具备虚拟化能力），它的商品，来自于KVM。当然，如果不用KVM，也可以用Xen等其它hypervisor。

<center><img src="https://img.darklorder.com/img/202305112252467.jpg"  alt="文件名" style="zoom:50%;" /></center>

OpenStack的管理界面，比命令行好多了吧？

请记住，上面所说的几个概念，包括VM、KVM、OpenStack等，都主要属于IaaS（基础设施即服务）。这个不难理解吧？

### 什么是容器

继续往下说。

那么，容器是什么呢？大佬们经常说的Docker和K8S，又是什么呢？

前面我们介绍了虚拟化。人们在使用虚拟化一段时间后，发现它存在一些问题：

不同的用户，有时候只是希望运行各自的一些简单程序，跑一个小进程。为了不相互影响，就要建立虚拟机。如果建虚拟机，显然浪费就会有点大，而且操作也比较复杂，花费时间也会比较长。

而且，有的时候，想要迁移自己的服务程序，就要迁移整个虚拟机。显然，迁移过程也会很复杂。

有没有办法**更灵活快速一**些呢？

有，这就引入了**“容器（Container）”。**

容器也是虚拟化，但是属于“轻量级”的虚拟化。它的目的和虚拟机一样，都是为了创造“隔离环境”。但是，它又和虚拟机有很大的不同——虚拟机是操作系统级别的资源隔离，而容器本质上是进程级的资源隔离。

<center><img src="https://img.darklorder.com/img/202305112253652.jpg"  alt="文件名" style="zoom:50%;" /></center>

虚拟化 VS 容器

而大家常听说的**Docker，**就是创建容器的工具，是应用容器引擎。

Docker的中文意思，就是码头工人。而它的LOGO，就是一只鲸鱼背着很多货柜箱。

<center><img src="https://img.darklorder.com/img/202305112254729.jpg"  alt="文件名" style="zoom:50%;" /></center>

相比于传统的虚拟机，Docker的优势很明显，它启动时间很快，是秒级，而且对资源的利用率很高（一台主机可以同时运行几千个Docker容器）。此外，它占的空间很小，虚拟机一般要几GB到几十GB，而容器只需要MB级甚至KB级。

<center><img src="https://img.darklorder.com/img/202305112254245.jpg"  alt="文件名" style="zoom:50%;" /></center>

除了Docker对容器进行创建之外，我们还需要一个工具，对容器进行**编排。**

这个工具，就是**K8S。**

**K8S，**就是**Kubernetes，**中文意思是舵手或导航员。Kubernetes这个单词很长，所以大家把中间8个字母缩写成8，就成了K8S。

<center><img src="https://img.darklorder.com/img/202305112255615.jpg"  alt="文件名" style="zoom:50%;" /></center>

K8S是一个容器集群管理系统，主要职责是**容器编排（Container Orchestration）**——启动容器，自动化部署、扩展和管理容器应用，还有回收容器。

简单来说，K8S有点像容器的保姆。它负责管理容器在哪个机器上运行，监控容器是否存在问题，控制容器和外界的通信，等等。

通过下面这张K8S系统结构图，就能够看出K8S和容器之间的关系。

<center><img src="https://img.darklorder.com/img/202305112255372.jpg"  alt="文件名" style="zoom:80%;" /></center>

除了K8S之外，还有很多种容器管理平台，例如**Compose，Marathon，Swarm，Mesos**等。

Docker和K8S，关注的不再是基础设施和物理资源，而是应用层，所以，就属于PaaS。明白了吧？

好啦，今天就先到这里了。再说下去，估计很多人又要晕啦。

正如文章开头所说，今天主要是介绍KVM、Hypervisor、OpenStack、Docker、K8S这些名词的意思，它们在云计算系统中的位置，以及它们之间的关系。云计算涉及到大量的需求。同一个需求，会有很多不同的技术来实现。同一个技术，往往又有多个不同的厂家互相竞争。所以，概念和名词就会特别多，发展变化也会很快。

不管怎么说，梳理清楚最关键的名词概念，是学好云计算的第一步。





**参考资料**
[看懂云计算、虚拟化和容器，这一篇就够啦！鲜枣课堂-作者小枣君](
https://mp.weixin.qq.com/s?__biz=MzI1NTA0MDUyMA==&mid=2456665252&idx=1&sn=365da9095c9a1ed46886eab43b5bc003&chksm=fda511c3cad298d5679642973577a7380ad050fddc15fde6c5bd81f354aacc787fb9ee86bb7f&scene=21#wechat_redirect)