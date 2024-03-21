---
title: 名称空间
date: 2022-08-31 00:00:00
categories:
- Cloud Native
tags:
- Cloud Native
---
Namespaces、CGroups、Containerruntime
<!-- more -->

#### Namespaces

内核名称空间（Kernel Namespace）是Linux操作系统中的一个重要概念，用于隔离和管理系统资源。在Linux中，名称空间是一种资源隔离机制，它允许不同的进程看到和访问的资源独立于其他进程，从而实现进程间的隔离性。内核名称空间特别用于隔离内核级别的资源。

如果把**Linux操作系统**比作一个**大房子**，那**命名空间**指的就是这个房子中的一个个**房间**，住在每个房间里的人都自以为独享了整个房子的资源，但其实大家仅仅只是在共享的基础之上互相隔离，共享指的是共享全局的资源，而隔离指的是局部上彼此保持隔离，因而命名空间的本质就是指：一种在空间上**隔离**的概念，当下盛行的许多容器虚拟化技术（典型代表如LXC、Docker）就是基于linux命名空间的概念而来的。

Docker容器十分类似LXC容器，他们实现了相同的安全特性。在你使用 `docker run`,启动一个Docker容器的时候，  Docker 会创建设置一个 namespaces 和 control groups 来配合容器。

命名空间提供的隔离，是第一个也是最简单的安全形式，在容器中运行的进程在其他容器或主机中是看不到的，基本上不会相互影响。

每一个容器都有自己的网络机制，这意味这不同的容器能访问其他容器接口的sockets。当然如果你希望容器能相互配置使用，也可以将容器的网络接口释放出来，利用端口转发，让容器像主机一样可以在网络中被识别。都你使用一个公共的端口来连接容器直接内部的网络，你就是尝试去在容器之间进行ping。实际上所有的容器都是利用桥接方式来共享端口连接，一台主机就可以连接运行在上面的所有容器。

**名称空间管理相关的系统调用**

- clone()：创建子进程，并将其隔离至新建的名称空间之中； 负责创建一个子进程，若同时使用了CLONE_NEW*相关的标志，则为每个标志创建出名称空间，并将该进程置于该名称空间中；

- setns()：将进程加入到指定的现有名称空间中； 通过操作进程相关的/proc/[pid]/ns/目录完成

- unshare()：将进程隔离至新建的名称空间中； 与clone()类似，但不同之处在于，unshare()在当前进程中创建名称空间，一旦调用完成，当前进程即位于新的名称空间之中；

- ioctl()：获取名称空间的相关信息。



目前，内核(5.13)支持8种名称空间

|  名称空间   |    调用标志     |         隔离内容          | 引入时的内核版本 |
| :---------: | :-------------: | :-----------------------: | :--------------: |
|  **Mount**  |   CLONE_NEWNS   |     挂载点、文件系统      |      2.4.19      |
|   **UTS**   |  CLONE_NEWUFS   |      主机名和NIS域名      |      2.6.19      |
|   **IPC**   |  CLONE_NEWIPC   | SysV IPC、POSIX messages  |      2.6.19      |
|   **PID**   |  CLONE_NEWPID   |            PID            |      2.6.24      |
| **Network** |  CLONE_NEWNET   | 网络设备、协议栈、端口等  |      2.6.29      |
|  **User**   |  CLONE_NEWUSER  |     User ID和Group ID     |       3.8        |
|   Cgroup    | CLONE_NEWCGROUP |  Cgroup根目录及层级结构   |       4.6        |
|    Time     |  CLONE_NEWTIME  | Boot time和monotonic time |       5.6        |

*备注：
monotonic time：单调递增时钟，自系统开机后开始累加计时，但系统休眠时间不计入；
boot time：类似于monotonic time，不同之处是，boot time会计入系统休眠时间；*


#### CGroups

cgroups(Control Groups) 是 linux 内核提供的一种机制，这种机制可以根据需求把一系列系统任务及其子任务整合(或分隔)到按资源划分等级的不同组内，从而为系统资源管理提供一个统一的框架。

简单说，cgroups 可以限制、记录任务组所使用的物理资源。本质上来说，cgroups 是内核附加在程序上的一系列钩子(hook)，通过程序运行时对资源的调度触发相应的钩子以达到资源追踪和限制的目的。

cgroups 的主要目的是为不同用户层面的资源管理提供一个统一化的接口。从单个任务的资源控制到操作系统层面的虚拟化，cgroups 提供了以下功能：

- 资源限制：cgroups 可以对任务的资源总额进行限制。比如设定任务运行时使用的内存上限，一旦超出就发 OOM（内存溢出）。
- 优先级分配：通过分配的 CPU 时间片数量和磁盘 IO 带宽，实际上就等同于控制了任务运行的优先级。
- 资源统计：cgoups 可以统计系统的资源使用量，比如 CPU 使用时长、内存用量等。这个功能非常适合当前云端产品按使用量计费的方式。
- 隔离：为组隔离命名空间，这样一个组不会看到另一个组的进程、网络连接和文件系统
- 任务控制：cgroups 可以对任务执行挂起、恢复等操作。

**CGroups能够限制的资源：**

- blkio：块设备IO
- cpu：CPU
- cpuacct：CPU资源使用报告
- cpuset：多处理器平台上的CPU集合
- devices：设备访问
- freezer：挂起或恢复任务
- memory：内存用量及报告
- perf_event：对cgroup中的任务进行统一性能测试
- net_cls：cgroup中的任务创建的数据报文的类别标识符

安装Docker后，用户可以在/sys/fs/cgroup/memory/docker/目录下看到对Docker组应用的各种限制项

~~~bash
[root@localhost ~]# cd /sys/fs/cgroup/memory/
[root@localhost memory]# ls
cgroup.clone_children           memory.kmem.slabinfo                memory.memsw.limit_in_bytes      memory.swappiness
cgroup.event_control            memory.kmem.tcp.failcnt             memory.memsw.max_usage_in_bytes  memory.usage_in_bytes
cgroup.procs                    memory.kmem.tcp.limit_in_bytes      memory.memsw.usage_in_bytes      memory.use_hierarchy
cgroup.sane_behavior            memory.kmem.tcp.max_usage_in_bytes  memory.move_charge_at_immigrate  notify_on_release
memory.failcnt                  memory.kmem.tcp.usage_in_bytes      memory.numa_stat                 release_agent
memory.force_empty              memory.kmem.usage_in_bytes          memory.oom_control               system.slice
memory.kmem.failcnt             memory.limit_in_bytes               memory.pressure_level            tasks
memory.kmem.limit_in_bytes      memory.max_usage_in_bytes           memory.soft_limit_in_bytes
memory.kmem.max_usage_in_bytes  memory.memsw.failcnt                memory.stat
~~~
~~~bash
[root@localhost ~]# cd /sys/fs/cgroup/memory/
[root@localhost memory]# ls
cgroup.clone_children       memory.kmem.max_usage_in_bytes      memory.memsw.failcnt             memory.stat
cgroup.event_control        memory.kmem.slabinfo                memory.memsw.limit_in_bytes      memory.swappiness
cgroup.procs                memory.kmem.tcp.failcnt             memory.memsw.max_usage_in_bytes  memory.usage_in_bytes
cgroup.sane_behavior        memory.kmem.tcp.limit_in_bytes      memory.memsw.usage_in_bytes      memory.use_hierarchy
init.scope                  memory.kmem.tcp.max_usage_in_bytes  memory.move_charge_at_immigrate  notify_on_release
memory.failcnt              memory.kmem.tcp.usage_in_bytes      memory.numa_stat                 release_agent
memory.force_empty          memory.kmem.usage_in_bytes          memory.oom_control               system.slice
memory.kmem.failcnt         memory.limit_in_bytes               memory.pressure_level            tasks
memory.kmem.limit_in_bytes  memory.max_usage_in_bytes           memory.soft_limit_in_bytes       user.slice
~~~

用户可以通过修改这些文件值来控制组限制Docker应用资源。

#### Container runtime

容器运行时（Container Runtime）是一种软件组件，负责在宿主机上管理和运行容器。它是容器化技术的核心组成部分，允许将应用程序及其依赖项打包并隔离在宿主系统之上。容器运行时与宿主操作系统内核进行交互，并管理容器的生命周期、网络、存储等重要功能。


容器并非Linux内核中的“一等公民”，它从根本上来说就是由名称空间、CGroups和LSM(Linux内核安全模块) 等几个内核原语组成；

- 借助于这些内核原语即可设置安全、隔离的进程运行环境，但这也意味着每次创建都得手动执行相关的操作；

- “容器运行时”便是一组简化该类操作的工具集

“运行时”是进程的生命周期管理工具，容器运行时是一种特指运行和管理容器所需要的软件

用于帮助用户轻松、高效、安全地部署容器，而且是容器管理的关键组件

2007年，CGroups引入到Linux内核之后，便出现了一些容器运行时项目，例如LXC(Linux containers)和LMCTFY(Google)等


容器运行时的主要任务包括：

- 容器创建：它加载容器镜像，该镜像包含应用程序及其运行时依赖项，并创建一个具有独立文件系统的容器实例。

- 容器执行：运行时启动容器进程，管理其资源，并提供一个独立的执行环境，使其表现为一个隔离的应用程序实例。

- 容器网络：设置网络接口并管理容器的网络连接，使其能够与其他容器或外部网络进行通信。

- 资源管理：运行时确保容器访问所需的资源，同时根据配置的限制限制其对宿主资源的访问。

- 容器终止：当容器内的应用程序完成任务或被显式停止时，运行时负责清理资源并终止容器。


#### 容器技术

借助于称之为 “容器运行时 Container Runtime” 的软件技术，在同一个内核之上生成多个彼此隔离的用户空间

- 各用户空间可独立管理运行其内部进程
- 每个用户空间 “自以为” 独占该内核及硬件资源
- 每一个用户空间就叫一个容器

需要将内核级的共享资源进行隔离，每个容器暴露的是系统接口

- 依赖于内核中称为 “名称空间” 的技术进行

  名称空间是Linux内核特性，用于隔离部分系统资源，从而使得进程仅可访问同一名称空间中的相应资源；

  名称空间只能做到隔离，做不到资源的分配；

- 资源限制则依赖于由Google贡献的 “CGroups”

  cgroups(Control Groups) 是 linux 内核提供的一种机制，这种机制可以根据需求把一系列系统任务及其子任务整合(或分隔)到按资源划分等级的不同组内，从而为系统资源管理提供一个统一的框架。


支撑容器创建的功能是内置于内核当中的，需要通过系统调用来实现；用户空间虚拟化是内核的功能。

每一个容器都是一组独立的名称空间，一组名称空间组合起来形成的。

**参考资料**
[Namespace in Operation](https://lwn.net/Articles/531114/)
