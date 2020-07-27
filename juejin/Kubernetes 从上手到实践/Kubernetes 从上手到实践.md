

# 开篇： Kubernetes 是什么以及为什么需要它

Kubernetes 是一个可扩展的，用于容器化应用程序编排，管理的平台。由 Google 于 2014 年基于其大规模生产实践经验而开源出来的。Kubernetes 目前在容器编排领域已经成为事实上的标准，社区也非常活跃。

Kubernetes 在国内外都已经得到广泛的应用，无论是Google, Amazon, GitHub 等还是国内的阿里，腾讯，百度，华为，京东或其他中小公司等也都已全力推进 Kubernetes 在生产中的使用。

现在无论是运维，后端，DBA，亦或者是前端，机器学习工程师等都需要在工作中或多或少的用到 Docker， 而在生产中大量使用的话 Kubernetes 也将会成为趋势，所以了解或掌握 Kubernetes 也成为了工程师必不可少的技能之一。

## Kubernetes 是什么?

当提到 Kubernetes 的时候，大多数人的可能会想到它可以容器编排，想到它是 PaaS (Platform as a Service) 系统，但其实不然，Kubernetes 并不是 PasS 系统，因为它工作在容器层而不是硬件层，它只不过提供了一些与 PasS 类似或者共同的功能，类似部署，扩容，监控，负载均衡，日志记录等。然而它并不是个完全一体化的平台，这些功能基本都是可选可配置的。

Kubernetes 可支持公有云，私有云及混合云等，具备良好的可移植性。我们可直接使用它或在其之上构建自己的容器/云平台，以达到快速部署，快速扩展，及优化资源使用等。

它致力于提供通用接口类似 CNI( Container Network Interface ), CSI（Container Storage Interface）, CRI（Container Runtime Interface）等规范，以便有更多可能, 让更多的厂商共同加入其生态体系内。它的目标是希望在以后，任何团队都可以在不修改 Kubernetes 核心代码的前提下，可以方便的扩展和构建符合自己需求的平台。

## 为什么需要 Kubernetes

我们回到实际的工作环境中。

- 如果你是个前端，你是否遇到过 npm 依赖安装极慢，或是 node sass 安装不了或者版本不对的情况？
- 如果你是个后端，是否遇到过服务器与本地环境不一致的情况，导致部分功能出现非预期的情况？
- 如果你是个运维，是否遇到过频繁部署环境，但中间可能出现各种安装不了或者版本不对的问题？

目前来看，对于这些问题，最好的解决方案便是标准化，容器化，现在用到最多的也就是 Docker。 Docker 通过 Dockerfile 来对环境进行描述，通过镜像进行交付，使用时不再需要关注环境不一致相关的问题。

现在面试的时候，无论前后端，我们总会多问下是否了解或者使用过 Docker 。如果使用过，那自然会问如果规模变大或者在生产中如何进行容器编排，部署扩容机制如何。

多数人在这个时候都已经回答不上来了，一方面是因为非运维相关岗位的同学，可能在实际工作中并不了解整体的架构体系，没有相关的知识积累。另一方面，对于运维同学可能尚未接触到这部分。

作为一个技术人员，我们应该对整体的体系架构有所了解, 掌握更多的技能，了解软件的完整生命周期，包括开发，交付，部署，以及当流量变大时的扩容等。

在容器编排领域，比较著名的主要有三个：Kubernetes, Mesos, 及 Docker 自家的 Swarm . 对这三者而言，较为简单的是 Swarm, 因为它本身只专注于容器编排，并且是官方团队所作，从各方面来看，对于新手都相对友好一些。但如果是用于生产中大规模使用，反而就略有不及。

而 Mesos 也并不仅限于容器编排，它的创建本身是为了将数据中心的所有资源进行抽象，比如 CPU，内存，网络，存储等，将整个 Mesos 集群当作是一个大的资源池，允许各种 Framework 来进行调度。比如，可以使用 Marathon 来实现 PaaS，可以运行 Spark，Hadoop 等进行计算等。同时因为它支持比如 Docker 或者 LXC 等作资源隔离，所以前几年也被大量用于容器编排。

随着 Kubernetes 在目前的认可度已经超过 Mesos， Docker Swarm 等，无疑它是生产环境中容器应用管理的不二之选。

本小册的目标是帮助更多开发者（不局限于运维，后端，前端等）认识并掌握 Kubernetes 的基础技能，了解其基础架构等。但是 Kubernetes 涉及知识点很多，且更新迭代很快，本小册集中于使用轻快的文字帮助大家掌握 K8S 的通用基础技能，对于其中需掌握的关于 Docker，及 Linux 内核相关的知识不会过于深入解释。主要以最常见 case 入手，帮助大家更快的掌握相关知识并将其用于生产实践中。

# 初步认识：Kubernetes 基础概念

好了，总算开始进入正题，抛弃掉死板的说教模式，我们以一个虚构的新成立的项目组为例开始我们的 Kubernetes 探索。(以下统一将 Kubernetes 简写为 K8S) 项目组目前就只有一个成员，我们称他为小张。项目组刚成立的时候，小张也没想好，具体要做什么，但肯定要对外提供服务的，所以先向公司申请了一台服务器。

## Node

这台服务器可以用来做什么呢？跑服务，跑数据库，跑测试之类的都可以，我们将它所做的事情统称为工作(work) 那么，它便是工作节点 (worker Node) 对应于 K8S 中，这就是我们首先要认识的 Node 。

Node 可以是一台物理机，也可以是虚拟机，对于我们此处的项目来讲，这台服务器便是 K8S 中的 Node 。

### Node 状态

当我们拿到这台服务器后，首先我们登录服务器查看下服务器的基本配置和信息。其实对于一个新加入 K8S 集群的 Node 也是一样，需要先检查它的状态，并将状态上报至集群的 master 。我们来看看服务器有哪些信息是我们所关心的。

#### 地址

首先，我们所关心的是我们服务器的 IP 地址，包括内网 IP 和外网 IP。对应于 K8S 集群的话这个概念是类似的，内部 IP 可在 K8S 集群内访问，外部 IP 可在集群外访问。

其次，我们也会关心一下我们的主机名，比如在服务器上执行 `hostname` 命令，便可得到主机名。K8S 集群中，每个 Node 的主机名也会被记录下来。当然，我们可以通过给 Kubelet 传递一个 `--hostname-override` 的参数来覆盖默认的主机名。 (Kubelet 是什么，我们后面会解释)

#### 信息

再之后，我们需要看下服务器的基本信息，比如看看系统版本信息， `cat /etc/issue` 或者 `cat /etc/os-release` 等方法均可查看。对于 K8S 集群会将每个 Node 的这些基础信息都记录下来。

#### 容量

我们通常也都会关注下，我们有几个核心的 CPU ，可通过 `cat /proc/cpuinfo` 查看，有多大的内存 通过 `cat /proc/meminfo` 或 `free` 等查看。对于 K8S 集群，会默认统计这些信息，并计算在此 Node 上可调度的 Pod 数量。（Pod 后面做解释）

#### 条件

对于我们拿到的服务器，我们关心上述的一些基本信息，并根据这些信息进行判断，这台机器是否能满足我们的需要。对 K8S 集群也同样，当判断上述信息均满足要求时候，便将集群内记录的该 Node 信息标记为 `Ready` （`Ready = True`），这样我们的服务器便正式的完成交付。我们来看下其他的部分。

## Deployment 和 Pod

现在小张拿到的服务器已经是可用状态，虽然此时尚不知要具体做什么，但姑且先部署一个主页来宣布下项目组的成立。

我们来看下一般情况下的做法，先写一个静态页面，比如叫 index.html 然后在服务器上启动一个 Nginx 或者其他任何 Web 服务器，来提供对 index.html 的访问。

Nginx 的安装及配置可参考 [Nginx 的官方文档](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/)。最简单的配置大概类似下面这样（仅保留关键部分）：

```
location / {
    root   /www;
    index  index.html;
}
```

对于 K8S 而言，我们想要的，能提供对 index.html 访问的服务便可理解为 Deployment 的概念，表明一种我们预期的目标状态。

而对于 Nginx 和 index.html 这个组合可以理解为其中的 Pod 概念，作为最小的调度单元。

## Container Runtime

虽然此刻部署的内容只有官网，但是为了避免单点故障，于是小张又申请了两台服务器（虽然看起来可能是浪费了点），现在要对原有的服务进行扩容，其实在新的服务器上我们所做的事情也还保持原样，部署 Nginx，提供对 index.html 的访问，甚至配置文件都完全是一样的。可以看到在这种情况下，增加一台服务器，我们需要做一件完全重复的事情。

本着不浪费时间做重复的工作的想法，小张想，要不然用 [Ansible](https://www.ansible.com/) 来统一管理服务器操作和配置吧，但考虑到后续服务器上还需要部署其他的服务，常规的这样部署，容易干扰彼此的环境。

所以我们想到了用虚拟化的技术，但是根据一般的经验，类似 [KVM](https://www.linux-kvm.org/page/Main_Page) 这样的虚拟化技术，可能在资源消耗上较多, 不够轻量级。而容器化相对来看，比较轻量级，也比较符合我们的预期，一次构建，随处执行。我们选择当前最热门的 Docker .

既然技术选型确定了，那很简单，在我们现在三台服务器上安装 Docker ，安装过程不再赘述，可以参考 [Docker 的官方安装文档](https://docs.docker.com/install/linux/docker-ce/centos/) 。

此时，我们需要做的事情，也便只是将我们的服务构建成一个镜像，需要编写一个 Dockerfile，构建一个镜像并部署到每台服务器上便可。

聊了这么多，我们现在已经将我们的服务运行到了容器中，而此处的 Docker 便是我们选择的容器运行时。选择它的最主要原因，便是为了环境隔离和避免重复工作。

而 Docker 如果对应于 K8S 集群中的概念，便是 Container Runtime，这里还有其他的选择，比如 rkt，runc 和其他实现了 OCI 规范的运行时。

## 总结

在这节里面，我们了解到了 Node 其实就是用于工作的服务器，它有一些状态和信息，当这些条件都满足一些条件判断时，Node 便处于 Ready 状态，可用于执行后续的工作。

Deployment 可理解为一种对期望状态的描述， Pod 作为集群中可调度的最小单元，我们会在后面详细讲解其细节。

Docker 是我们选择的容器运行时，可运行我们构建的服务镜像，减少在环境方面所做的重复工作，并且也非常便于部署。除了 Docker 外还存在其他的容器运行时。

了解到这些基本概念后，下节我们从宏观的角度上来认识 K8S 的整体架构，以便我们后续的学习和实践。

# 宏观认识：整体架构

工欲善其事，必先利其器。本节我们来从宏观上认识下 K8S 的整体架构，以便于后续在此基础上进行探索和实践。

## C/S 架构

从更高层来看，K8S 整体上遵循 C/S 架构，从这个角度来看，可用下面的图来表示其结构：

```
                               +-------------+                              
                               |             |                              
                               |             |               +---------------+
                               |             |       +-----> |     Node 1    |
                               | Kubernetes  |       |       +---------------+
+-----------------+            |   Server    |       |                      
|       CLI       |            |             |       |       +---------------+
|    (Kubectl)    |----------->| ( Master )  |<------+-----> |     Node 2    |
|                 |            |             |       |       +---------------+
+-----------------+            |             |       |       
                               |             |       |       +---------------+
                               |             |       +-----> |     Node 3    |
                               |             |               +---------------+
                               +-------------+               
```

左侧是一个官方提供的名为 `kubectl` 的 CLI （Command Line Interface）工具，用于使用 K8S 开放的 API 来管理集群和操作对象等。

右侧则是 K8S 集群的后端服务及开放出的 API 等。根据上一节的内容，我们知道 Node 是用于工作的机器，而 Master 是一种角色（Role），表示在这个 Node 上包含着管理集群的一些必要组件。具体组件的详细介绍参考第 11 小节对各组件的详细剖析。

当然在这里，只画出了一个 Master，在生产环境中，为了保障集群的高可用，我们通常会部署多个 Master 。

## Master

下面我们来逐层分解， 首先是 Master ，这里我们只介绍其管理集群的相关组件。Master 是整个 K8S 集群的“大脑”，与大脑类似，它有几个重要的功能：

- 接收：外部的请求和集群内部的通知反馈
- 发布：对集群整体的调度和管理
- 存储：存储

这些功能，也通过一些组件来共同完成，通常情况下，我们将其称为 control plane 。如下图所示：

```
+----------------------------------------------------------+          
| Master                                                   |          
|              +-------------------------+                 |          
|     +------->|        API Server       |<--------+       |          
|     |        |                         |         |       |          
|     v        +-------------------------+         v       |          
|   +----------------+     ^      +--------------------+   |          
|   |                |     |      |                    |   |          
|   |   Scheduler    |     |      | Controller Manager |   |          
|   |                |     |      |                    |   |          
|   +----------------+     v      +--------------------+   |          
| +------------------------------------------------------+ |          
| |                                                      | |          
| |                Cluster state store                   | |          
| |                                                      | |          
| +------------------------------------------------------+ |          
+----------------------------------------------------------+          
```

它主要包含以下几个重要的组成部分。

### Cluster state store

存储集群所有需持久化的状态，并且提供 watch 的功能支持，可以快速的通知各组件的变更等操作。

因为目前 Kubernetes 的存储层选择是 etcd ，所以一般情况下，大家都直接以 etcd 来代表集群状态存储服务。即：将所有状态存储到 etcd 实例中。

刚才我们说 Master 相当于是 K8S 集群的大脑，更细化来看，etcd 则是大脑中的核心，为什么这么说？可以参考后面详细剖析的章节，本章我们先从更高的层次来看集群的整体架构。

你可能会问， etcd 是必须的吗？就目前而言，etcd 是必须的，这主要是 Kubernetes 的内部实现。

而早在 2014 年左右，社区就一直在提议将存储层抽象出来，后端的实际存储作为一种插件化的存在。[呼声](https://github.com/kubernetes/kubernetes/issues/1957)比较大的是另一种提供 k/v 存储功能的 [Consul](https://www.consul.io/) 。

不过得益于 etcd 的开发团队较为活跃，而且根据 K8S 社区的反馈做了相当大的改进，并且当时 K8S 团队主要的关注点也不在此，所以直到现在 etcd 仍不是一个可选项。

如果现在去看下 Kubernetes 的源代码，你会发现存储层的代码还比较简洁清晰，后续如果有精力也许将此处插件化也不是不可能。

### API Server

这是整个集群的入口，类似于人体的感官，接收外部的信号和请求，并将一些信息写入到 etcd 中。

实际处理逻辑比三次握手简单的多：

- 请求 API Server ：“嗨，我有些东西要放到 etcd 里面”
- API Server 收到请求：“你是谁？我为啥要听你的”
- 从请求中，拿出自己的身份凭证（一般是证书）：“是我啊，你的master，给我把这些东西放进去”
- 这时候就要看是些什么内容了，如果这些内容 API Server 能理解，那就放入 etcd 中 “好的 master 我放进去了”；如果不能理解，“抱歉 master 我理解不了”

可以看到，它提供了认证相关的功能，用于判断是否有权限进行操作。当然 API Server 支持多种认证方法，不过一般情况下，我们都使用 x509 证书进行认证。

API Server 的目标是成为一个极简的 server，只提供 REST 操作，更新 etcd ，并充当着集群的网关。至于其他的业务逻辑之类的，通过插件或者在其他组件中完成。关于这部分的详细实现，可以参考后面的 API Server 剖析相关章节。

### Controller Manager

Controller Manager 大概是 K8S 集群中最繁忙的部分，它在后台运行着许多不同的控制器进程，用来调节集群的状态。

当集群的配置发生变更，控制器就会朝着预期的状态开始工作。

### Scheduler

顾名思义，Scheduler 是集群的调度器，它会持续的关注集群中未被调度的 Pod ，并根据各种条件，比如资源的可用性，节点的亲和性或者其他的一些限制条件，通过绑定的 API 将 Pod 调度/绑定到 Node 上。

在这个过程中，调度程序一般只考虑调度开始时， Node 的状态，而不考虑在调度过程中 Node 的状态变化 (比如节点亲和性等，截至到目前 v1.11.2 也暂未加入相关功能的稳定特性)

## Node

Node 的概念我们在上节已经提过了，这里不再过多赘述，简单点理解为加入集群中的机器即可。

那 Node 是如何加入集群接受调度，并运行服务的呢？这都要归功于运行在 Node 上的几个核心组件。我们先来看下整体结构：

```
+--------------------------------------------------------+       
| +---------------------+        +---------------------+ |       
| |      kubelet        |        |     kube-proxy      | |       
| |                     |        |                     | |       
| +---------------------+        +---------------------+ |       
| +----------------------------------------------------+ |       
| | Container Runtime (Docker)                         | |       
| | +---------------------+    +---------------------+ | |       
| | |Pod                  |    |Pod                  | | |       
| | | +-----+ +-----+     |    |+-----++-----++-----+| | |       
| | | |C1   | |C2   |     |    ||C1   ||C2   ||C3   || | |       
| | | |     | |     |     |    ||     ||     ||     || | |       
| | | +-----+ +-----+     |    |+-----++-----++-----+| | |       
| | +---------------------+    +---------------------+ | |       
| +----------------------------------------------------+ |       
+--------------------------------------------------------+  
```

### Kubelet

Kubelet 实现了集群中最重要的关于 Node 和 Pod 的控制功能，如果没有 Kubelet 的存在，那 Kubernetes 很可能就只是一个纯粹的通过 API Server CRUD 的应用程序。

K8S 原生的执行模式是操作应用程序的容器，而不像传统模式那样，直接操作某个包或者是操作某个进程。基于这种模式，可以让应用程序之间相互隔离，互不影响。此外，由于是操作容器，所以应用程序可以说和主机也是相互隔离的，毕竟它不依赖于主机，在任何的容器运行时（比如 Docker）上都可以部署和运行。

我们在上节介绍过 Pod，Pod 可以是一组容器（也可以包含存储卷），K8S 将 Pod 作为可调度的基本单位， 分离开了构建时和部署时的关注点：

- 构建时，重点关注某个容器是否能正确构建，如何快速构建
- 部署时，关心某个应用程序的服务是否可用，是否符合预期，依赖的相关资源是否都能访问到

这种隔离的模式，可以很方便的将应用程序与底层的基础设施解耦，极大的提高集群扩/缩容，迁移的灵活性。

在前面，我们提到了 Master 节点的 `Scheduler` 组件，它会调度未绑定的 Pod 到符合条件的 Node 上，而至于最终该 Pod 是否能运行于 Node 上，则是由 `Kubelet` 来裁定的。关于 Kubelet 的具体原理，后面有详细剖析的章节。

### Container runtime

容器运行时最主要的功能是下载镜像和运行容器，我们最常见的实现可能是 [Docker](https://www.docker.com/) , 目前还有其他的一些实现，比如 [rkt](https://github.com/rkt/rkt), [cri-o](https://github.com/kubernetes-sigs/cri-o)。

K8S 提供了一套通用的容器运行时接口 CRI (Container Runtime Interface), 凡是符合这套标准的容器运行时实现，均可在 K8S 上使用。

### Kube Proxy

我们都知道，想要访问某个服务，那要么通过域名，要么通过 IP。而每个 Pod 在创建后都会有一个虚拟 IP，K8S 中有一个抽象的概念，叫做 `Service` ，`kube-proxy` 便是提供一种代理的服务，让你可以通过 `Service` 访问到 Pod。

实际的工作原理是在每个 Node 上启动一个 `kube-proxy` 的进程，通过编排 `iptables` 规则来达到此效果。深入的解析，在后面有对应的章节。

## 总结

本节中，我们了解到了 K8S 的整体遵循 C/S 架构，集群的 Master 包含着几个重要的组成部分，比如 `API Server`, `Controller Manager` 等。

而 Node 上，则运行着三个必要的组件 `kubelet`, `container runtime` (一般是 Docker), `kube-proxy` 。

通过所有组件的分工协作，最终实现了 K8S 对容器的编排和调度。

完成了这节的学习，那我们就开始着手搭建一个属于我们自己的集群吧。

# 搭建 Kubernetes 集群 - 本地快速搭建

通过之前的学习，我们已经知道了 K8S 中有一些组件是必须的，集群中有不同的角色。本节，我们在本地快速搭建一个集群，以加深我们学习到的东西。

## 方案选择

在上一节中，我们知道 K8S 中有多种功能组件，而这些组件要在本地全部搭建好，需要一些基础知识，以及在搭建过程中会浪费不少的时间，从而可能会影响我们正常的搭建集群的目标。

所以，我们这里提供两个最简单，最容易实现我们目标的工具

- [KIND](https://github.com/kubernetes-sigs/kind/) 。
- [Minikube](https://github.com/kubernetes/minikube) 。

## KIND

### 介绍

KIND（Kubernetes in Docker）是为了能提供更加简单，高效的方式来启动 K8S 集群，目前主要用于比如 `Kubernetes` 自身的 CI 环境中。

### 安装

- 可以直接在项目的 [Release 页面](https://github.com/kubernetes-sigs/kind/releases) 下载已经编译好的二进制文件。(下文中使用的是 v0.1.0 版本的二进制包)

注意：如果不直接使用二进制包，而是使用 `go get sigs.k8s.io/kind` 的方式下载，则与下文中的配置文件不兼容。**请参考[使用 Kind 搭建你的本地 Kubernetes 集群](https://zhuanlan.zhihu.com/p/60464867)** 这篇文章。

更新（2020年2月5日）：KIND 已经发布了 v0.7.0 版本，如果你想使用新版本，建议参考 [使用 Kind 在离线环境创建 K8S 集群](https://zhuanlan.zhihu.com/p/105173589) ，这篇文章使用了最新版本的 KIND。



![img](https://user-gold-cdn.xitu.io/2019/1/29/16898b9e3d57fcab?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 创建集群

**在使用 KIND 之前，你需要本地先安装好 Docker 的环境** ，此处暂不做展开。

由于网络问题，我们此处也需要写一个配置文件，以便让 `kind` 可以使用国内的镜像源。（KIND 最新版本中已经内置了所有需要的镜像，无需此操作）

```
apiVersion: kind.sigs.k8s.io/v1alpha1
kind: Config

kubeadmConfigPatches:
- |
  apiVersion: kubeadm.k8s.io/v1alpha3
  kind: InitConfiguration
  nodeRegistration:
  kubeletExtraArgs:
    pod-infra-container-image: registry.aliyuncs.com/google_containers/pause-amd64:3.1
- |
  apiVersion: kubeadm.k8s.io/v1alpha3
  kind: ClusterConfiguration
  imageRepository: registry.aliyuncs.com/google_containers
  kubernetesVersion: v1.12.2
  networking:
    serviceSubnet: 10.0.0.0/16
```

将上面的内容保存成 `kind-config.yaml` 文件，执行以下命令即可。

```
kind create cluster --image kindest/node:v1.12.2 --config kind-config.yaml --name moelove                  
```

下面为在我机器上执行的程序输出：

```
(MoeLove) ➜  kind ✗ kind create cluster --image kindest/node:v1.12.2 --config kind-config.yaml --name moelove                  
Creating cluster 'kind-moelove' ...
 ✓ Ensuring node image (kindest/node:v1.12.2) 🖼
 ✓ [kind-moelove-control-plane] Creating node container 📦                                                         
 ✓ [kind-moelove-control-plane] Fixing mounts 🗻
 ✓ [kind-moelove-control-plane] Starting systemd 🖥
 ✓ [kind-moelove-control-plane] Waiting for docker to be ready 🐋                                                  
 ✓ [kind-moelove-control-plane] Starting Kubernetes (this may take a minute) ☸                                     
Cluster creation complete. You can now use the cluster with:

export KUBECONFIG="$(kind get kubeconfig-path --name="moelove")"
kubectl cluster-info
```

这里，通过传递上面的 `kind-config.yaml` 文件给 `kind create cluster`, 并传递了一个名字通过 `--name` 参数。

我们按照程序输出的提示进行操作：

```
export KUBECONFIG="$(kind get kubeconfig-path --name="moelove")"
kubectl cluster-info
```

下面为在我机器上执行的程序输出：

```
(MoeLove) ➜  kind ✗ export KUBECONFIG="$(kind get kubeconfig-path --name="moelove")"
(MoeLove) ➜  kind ✗ kubectl cluster-info
Kubernetes master is running at https://localhost:35911
KubeDNS is running at https://localhost:35911/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
(MoeLove) ➜  kind ✗ kubectl version
Client Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T18:02:47Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:43:59Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
```

注意，这里需要安装 `kubectl`。 `kubectl` 的安装可参考下面的内容。

当你执行 `kubectl cluster-info` 如果可以看到类似我上面的输出，那你本地的 K8S 集群就已经部署好了。你可以直接阅读第 5 节或者第 6 节的内容。

如果你已经对 K8S 有所了解，并且对 Dashboard 有比较强烈需求的话, 可直接参考第 20 节的内容。

## Minikube

### 介绍

Minikube 是 K8S 官方为了开发者能在个人电脑上运行 K8S 而提供的一套工具。实现上是通过 Go 语言编写，通过调用虚拟化管理程序，创建出一个运行在虚拟机内的单节点集群。

注：从这里也可以看出，对于 K8S 集群的基本功能而言，节点数并没有什么限制。只有一个节点同样可以创建集群。

### 前期准备

- 首先需要确认 BIOS 已经开启了 `VT-x` 或者 `AMD-v` 虚拟化的支持。具体方式可参考 [确认是否已开启 BIOS 虚拟化](https://www.shaileshjha.com/how-to-find-out-if-intel-vt-x-or-amd-v-virtualization-technology-is-supported-in-windows-10-windows-8-windows-vista-or-windows-7-machine/), [开启 BIOS 虚拟化支持](https://www.howtogeek.com/213795/how-to-enable-intel-vt-x-in-your-computers-bios-or-uefi-firmware/) 这两篇文章。
- 其次我们需要安装一个虚拟化管理程序，这里的选择可根据你实际在用的操作系统来决定。官方推荐如下:
  - macOS: [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 或 [VMware Fusion](https://www.vmware.com/products/fusion) 或 [HyperKit](https://github.com/moby/hyperkit)。如果使用 Hyperkit 需要注意系统必须是 `OS X 10.10.3 Yosemite` 及之后版本的。
  - Linux: [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 或 [KVM](http://www.linux-kvm.org/)。
  - Windows: [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 或 [Hyper-V](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)。

我个人推荐无论你在以上哪种操作系统中使用 Minikube 都选择用 `Virtualbox` 作为虚拟化管理程序，1. Virtualbox 无论操作体验还是安装都比较简单 2. Minikube 对其支持更完备，并且也已经经过大量用户测试，相关问题均已基本修复。

*如果你是在 Linux 系统上面，其实还有一个选择，便是将 Minikube 的 `--vm-driver` 参数设置为 `none` ，并且在本机已经正确安装 Docker。 这种方式是无需虚拟化支持的。*

### 安装 kubectl

上一节我们已经学到 K8S 集群是典型的 C/S 架构，有一个官方提供的名为 `kubectl` 的 CLI 工具。在这里，我们要安装 `kubectl` 以便后续我们可以对搭建好的集群进行管理。

**注：由于 API 版本兼容的问题，尽量保持 `kubectl` 版本与 K8S 集群版本保持一致，或版本相差在在一个小版本内。**

官方文档提供了 `macOS`, `Linux`, `Windows` 等操作系统上的安装方式，且描述很详细，这里不过多赘述，[文档地址](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl)。

**此处提供一个不同于官方文档中的安装方式。**

- 访问 K8S 主仓库的 [CHANGELOG 文件](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG.md) 找到你所需要的版本。 由于我们将要使用的 Minikube 是官方最新的稳定版 v0.28.2，而它内置的 Kubernetes 的版本是 v1.10 所以，我们选择使用对应的 [1.10 版本](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md)的 `kubectl`。当然，我们也可以通过传递参数的方式来创建不同版本的集群。如 `minikube start --kubernetes-version v1.11.3` 用此命令创建 `v1.11.3` 版本的集群，当然 `kubectl` 的版本也需要相应升级。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="687" height="538"></svg>)



点击[Client Binaries](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#client-binaries) 找到你符合所需系统架构的对应包下载即可。这里我以 [Linux 下 64 位的包](https://dl.k8s.io/v1.10.7/kubernetes-client-linux-amd64.tar.gz)为例。

```
➜  wget https://dl.k8s.io/v1.10.7/kubernetes-client-linux-amd64.tar.gz
➜  echo '169b57c6707ed8d8be9643b0088631e5c0c6a37a5e99205f03c1199cd32bc61e  kubernetes-client-linux-amd64.tar.gz' | sha256sum -c -
kubernetes-client-linux-amd64.tar.gz: 成功
➜  tar zxf kubernetes-client-linux-amd64.tar.gz
➜  sudo mv kubernetes/client/bin/kubectl /usr/local/bin/kubectl
➜  /usr/local/bin/kubectl version --client
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.7", GitCommit:"0c38c362511b20a098d7cd855f1314dad92c2780", GitTreeState:"clean", BuildDate:"2018-08-20T10:09:03Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```

执行以上命令即可完成 `kubectl` 的安装，最后一步会看到当前安装的版本信息等。

### 安装 Minikube

先查看 Minikube 的 [Release 页面](https://github.com/kubernetes/minikube/releases)，当前最新的稳定版本是 v0.28.2，找到你所需系统的版本，点击下载，并将下载好的可执行文件加入你的 PATH 中。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="948" height="637"></svg>)



**注：当前 Windows 系统下的安装包还处于实验性质，如果你是在 Windows 环境下，同样是可以下载安装使用的**

以 Linux 下的安装为例：

```
➜  wget -O minikube https://github.com/kubernetes/minikube/releases/download/v0.28.2/minikube-linux-amd64
➜  chmod +x minikube
➜  sudo mv minikube /usr/local/bin/minikube

➜  /usr/local/bin/minikube version
minikube version: v0.28.2
```

最后一步可查看当前已安装好的 Minikube 的版本信息。如果安装成功将会看到和我上面内容相同的结果。

### 创建第一个 K8S 集群

使用 Minikube 创建集群，只要简单的执行 `minikube start` 即可。正常情况下，你会看到和我类似的输出。

```
➜  ~ minikube start
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.

➜  ~ minikube status
minikube: Running
cluster: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100         
```

为了验证我们的集群目前是否均已配置正确，可以执行以下命令查看。

```
➜  ~ kubectl cluster-info 
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

➜  ~ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    1d       v1.10.0
```

如果出现类似拒绝连接之类的提示，那可能是因为你的 `kubectl` 配置不正确，可查看 `$HOME/.kube/config` 文件检查配置。示例输出如下：

```
➜  ~ cat .kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/tao/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/tao/.minikube/client.crt
    client-key: /home/tao/.minikube/client.key
```

如果没有该文件，可按上面示例内容进行创建，替换掉其中的路径及 `server` 地址配置。 `server` 地址可通过 `minikube status` 或者 `minikube ip` 查看或检查。

```
(Tao) ➜  ~ minikube ip
192.168.99.100

(Tao) ➜  ~ minikube status
minikube: Running
cluster: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

### 通过 Dashboard 查看集群当前状态

使用 `Minikube` 的另一个好处在于，你可以不用关注太多安装方面的过程，直接在终端下输入 `minikube dashboard` 打开系统 Dashboard 并通过此来查看集群相关状态。

执行 `minikube dashboard` 后会自动打开浏览器，默认情况下监听在通过 `minikube ip` 所获得 IP 的 3000 端口。如下图所示：



![img](https://user-gold-cdn.xitu.io/2018/9/16/165de25c7c517d78?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 相关链接:

- [安装 VirtualBox](https://websiteforstudents.com/installing-virtualbox-ubuntu-17-04/)
- [使用 Kind 搭建你的本地 Kubernetes 集群](https://juejin.im/post/5c99ed6c6fb9a0710e47ebeb)

## 总结

本节中，我们为了能更快的体验到 K8S 集群，避免很多繁琐的安装步骤，我们选择了使用官方提供的 `Minikube` 工具来搭建一个本地集群。

Minikube 的本质其实是将一套 “定制化” 的 K8S 集群打包成 ISO 镜像，当执行 `minikube start` 的时候，便通过此镜像启动一个虚拟机，在此虚拟机上通过 `kubeadm` 工具来搭建一套只有一个节点的 K8S 集群。关于 `kubeadm` 工具，我们在下节进行讲解。

同时，会通过虚拟机的相关配置接口拿到刚才所启动虚拟机的地址信息等，并完成本地的 `kubectl` 工具的配置，以便于让用户通过 `kubectl` 工具对集群进行操作。

事实上，当前 Docker for Mac 17.12 CE Edge 和 Docker for Windows 18.02 CE Edge ，以及这两种平台更高的 Edge 版本, 均已内置了对 K8S 的支持，但均为 Edge 版本，此处暂不做过多介绍。

# 动手实践：搭建一个 Kubernetes 集群 - 生产可用

通过上一节的学习，我们快速的使用 `Minikube` 搭建了一个本地可用的 K8S 集群。默认情况下，节点是一个虚拟机实例，我们可以在上面体验一些基本的功能。

大多数人的需求并不只是包含一个虚拟机节点的本地测试集群，而是一个可在服务器运行，可自行扩/缩容，具备全部功能的，达到生产可用的集群。

K8S 集群的搭建，一直让很多人头疼，本节我们来搭建一个生产可用的集群，便于后续的学习或使用。

## 方案选择

K8S 生产环境可用的集群方案有很多，本节我们选择一个 Kubernetes 官方推荐的方案 `kubeadm` 进行搭建。

`kubeadm` 是 Kubernetes 官方提供的一个 CLI 工具，可以很方便的搭建一套符合官方最佳实践的最小化可用集群。当我们使用 `kubeadm` 搭建集群时，集群可以通过 K8S 的一致性测试，并且 `kubeadm` 还支持其他的集群生命周期功能，比如升级/降级等。

我们在此处选择 `kubeadm` ，因为我们可以不用过于关注集群的内部细节，便可以快速的搭建出生产可用的集群。我们可以通过后续章节的学习，快速上手 K8S ，并学习到 K8S 的内部原理。在此基础上，想要在物理机上完全一步步搭建集群，便轻而易举。

## 安装基础组件

### 前期准备

使用 `kubeadm` 前，我们需要提前做一些准备。

- **我们需要禁用 `swap`**。通过之前的学习，我们知道每个节点上都有个必须的组件，名为 `kubelet`，自 K8S 1.8 开始，启动 `kubelet` 时，需要禁用 `swap` 。或者需要更改 `kubelet` 的启动参数 `--fail-swap-on=false`。

  虽说可以更改参数让其可用，但是我建议还是禁用 `swap` 除非你的集群有特殊的需求，比如：有大内存使用的需求，但又想节约成本；或者你知道你将要做什么，否则可能会出现一些非预期的情况，尤其是做了内存限制的时候，当某个 Pod 达到内存限制的时候，它可能会溢出到 swap 中，这会导致 K8S 无法正常进行调度。

  如何禁用：

  1. 使用 `sudo cat /proc/swaps` 验证 swap 配置的设备和文件。
  2. 通过 `swapoff -a` 关闭 swap 。
  3. 使用 `sudo blkid` 或者 `sudo lsblk` 可查看到我们的设备属性，请注意输出结果中带有 `swap` 字样的信息。
  4. 将 `/etc/fstab` 中和上一条命令中输出的，和 swap 相关的挂载点都删掉，以免在机器重启或重挂载时，再挂载 `swap` 分区。

  执行完上述操作，`swap` 便会被禁用，当然你也可以再次通过上述命令，或者 `free` 命令来确认是否还有 `swap` 存在。

  ```
  [root@master ~]# free 
                total        used        free      shared  buff/cache   available
  Mem:        1882748       85608     1614836       16808      182304     1630476
  Swap:             0           0           0
  ```

- 通过 `sudo cat /sys/class/dmi/id/product_uuid` 可查看机器的 `product_uuid` 请确保要搭建集群的所有节点的 `product_uuid` 均不相同。同时所有节点的 Mac 地址也不能相同，通过 `ip a` 或者 `ifconfig -a` 可进行查看。

  我们在第二章提到过，每个 Node 都有一些信息会被记录进集群内，而此处我们需要保证的这些唯一的信息，便会记录在集群的 `nodeInfo` 中，比如 `product_uuid` 在集群内以 `systemUUID` 来表示。具体信息我们可以通过集群的 `API Server` 获取到，在后面的章节会详细讲述。

- 第三章中，我们已经谈过 K8S 是 C/S 架构，在启动后，会固定监听一些端口用于提供服务。可以通过 `sudo netstat -ntlp |grep -E '6443|23[79,80]|1025[0,1,2]'` 查看这些端口是否被占用，如果被占用，请手动释放。

  如果你执行上述命令时，提示 `command not found`，则表明你需要先安装 `netstat`，在 CentOS 系统中需要通过 `sudo yum install net-tools` 安装，而在 Debian/Ubuntu 系统中，则需要通过 `sudo apt install net-tools` 进行安装。

- 前面我们也提到了，我们需要一个容器运行时，通常情况下是 `Docker`，我们可以通过[官方的 Docker 文档](https://docs.docker.com/install/overview/) 进行安装，安装完成后记得启动服务。

  官方推荐使用 `17.03` ，但我建议你可以直接安装 `18.03` 或者更新的版本，因为 `17.03` 版本的 Docker 已经在 2018 年 3 月 `EOL`（End Of Life）了。对于更新版本的 Docker，虽然 K8S 尚未在新版本中经过大量测试，但毕竟新版本有很多 Bugfix 和新特性的增加，也能规避一些可能遇到的问题（比如个别情况下 container 不会自动删除的情况 (17.09) ）。

  另外，由于 Docker 的 API 都是带有版本的，且有良好的兼容性，当使用低版本 API 请求时会自动降级，所以一般情况下也不会有什么问题。

### 安装 kubectl

第三章中，我们已经说过 `kubectl` 是集群的客户端，我们现在搭建集群时，也必须要安装它，用于验证集群功能。

安装步骤在第 4 章已经详细说明了，此处不做赘述，可查阅第 4 章或参考下面的内容。

### 安装 kubeadm 和 kubelet

首先是版本的选择，我们可以通过下面的命令获取到当前的 stable 版本号。要访问这个地址，需要自行处理网络问题或使用我后面提供的解决办法。

```
[root@master ~]# curl -sSL https://dl.k8s.io/release/stable.txt
v1.11.3
```

下载二进制包，并通过 `kubeadm version` 验证版本是否正确。

```
[root@master ~]# curl -sSL https://dl.k8s.io/release/v1.11.3/bin/linux/amd64/kubeadm > /usr/bin/kubeadm
[root@master ~]# chmod a+rx /usr/bin/kubeadm
[root@master ~]# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T17:59:42Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
```

当然，我们其实可以使用如同上一章的方式，直接进入到 `kubernetes` 的[官方仓库](https://github.com/kubernetes/kubernetes)，找到我们所需版本 [v1.11.3](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.11.md#v1113) 下载 `Server Binaries`，如下图：



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1019" height="670"></svg>)



终端下可使用如下方式下载：

```
[root@master tmp]# wget -q https://dl.k8s.io/v1.11.3/kubernetes-server-linux-amd64.tar.gz
```

**对于国内用户，我已经准备了下面的方式，方便使用。**

```
链接: https://pan.baidu.com/s/1FSEcEUplQQGsjyBIZ6j2fA 提取码: cu4s
```

下载完成后，验证文件是否正确无误，验证通过后进行解压。

```
[root@master tmp]# echo 'e49d0db1791555d73add107d2110d54487df538b35b9dde0c5590ac4c5e9e039 kubernetes-server-linux-amd64.tar.gz' | sha256sum -c -
kubernetes-server-linux-amd64.tar.gz: 确定
[root@master tmp]# tar -zxf kubernetes-server-linux-amd64.tar.gz
[root@master tmp]# ls kubernetes
addons  kubernetes-src.tar.gz  LICENSES  server
[root@master tmp]# ls kubernetes/server/bin/ | grep -E 'kubeadm|kubelet|kubectl'
kubeadm
kubectl
kubelet
```

可以看到在 `server/bin/` 目录下有我们所需要的全部内容，将我们所需要的 `kubeadm` `kubectl` `kubelet` 等都移动至 `/usr/bin` 目录下。

```
[root@master tmp]# mv kubernetes/server/bin/kube{adm,ctl,let} /usr/bin/
[root@master tmp]# ls /usr/bin/kube*
/usr/bin/kubeadm  /usr/bin/kubectl  /usr/bin/kubelet
[root@master tmp]# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T17:59:42Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
[root@master tmp]# kubectl version --client
Client Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T18:02:47Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
[root@master tmp]# kubelet --version
Kubernetes v1.11.3
```

可以看到我们所需的组件，版本均为 `v1.11.3` 。

## 配置

为了在生产环境中保障各组件的稳定运行，同时也为了便于管理，我们增加对 `kubelet` 的 `systemd` 的配置，由 `systemd` 对服务进行管理。

### 配置 kubelet

```
[root@master tmp]# cat <<'EOF' > /etc/systemd/system/kubelet.service
[Unit]
Description=kubelet: The Kubernetes Agent
Documentation=http://kubernetes.io/docs/

[Service]
ExecStart=/usr/bin/kubelet
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
[root@master tmp]# mkdir -p /etc/systemd/system/kubelet.service.d
[root@master tmp]# cat <<'EOF' > /etc/systemd/system/kubelet.service.d/kubeadm.conf
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
EOF
[root@master tmp]# systemctl enable kubelet
Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /etc/systemd/system/kubelet.service.
```

在这里我们添加了 `kubelet` 的 systemd 配置，然后添加了它的 `Drop-in` 文件，我们增加的这个 `kubeadm.conf` 文件，会被 systemd 自动解析，用于复写 `kubelet` 的基础 systemd 配置，可以看到我们增加了一系列的配置参数。在第 17 章中，我们会对 `kubelet` 做详细剖析，到时再进行解释。

## 启动

此时，我们的前期准备已经基本完成，可以使用 `kubeadm` 来创建集群了。别着急，在此之前，我们还需要安装两个工具，名为`crictl` 和 `socat`。

### 安装前置依赖 crictl

`crictl` 包含在 [`cri-tools`](https://github.com/kubernetes-sigs/cri-tools.git) 项目中，这个项目中包含两个工具：

- `crictl` 是 `kubelet` CRI (Container Runtime Interface) 的 CLI 。
- `critest` 是 `kubelet` CRI 的测试工具集。

安装可以通过进入 `cri-tools` 项目的 [Release 页面](https://github.com/kubernetes-sigs/cri-tools/releases) ，根据项目 [README](https://github.com/kubernetes-sigs/cri-tools#current-status) 文件中的版本兼容关系，选择自己所需的安装包，下载即可，由于我们安装 K8S 1.11.3 所以选择最新的 v1.11.x 的安装包。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)



```
[root@master ~]# wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.11.1/crictl-v1.11.1-linux-amd64.tar.gz
[root@master ~]# echo 'ccf83574556793ceb01717dc91c66b70f183c60c2bbec70283939aae8fdef768 crictl-v1.11.1-linux-amd64.tar.gz' | sha256sum -c -
crictl-v1.11.1-linux-amd64.tar.gz: 确定
[root@master ~]# tar zxvf crictl-v1.11.1-linux-amd64.tar.gz
[root@master ~]# mv crictl /usr/bin/
```

### 安装前置依赖 socat

`socat` 是一款很强大的命令行工具，可以建立两个双向字节流并在其中传输数据。这么说你也许不太理解，简单点说，它其中的一个功能是可以实现端口转发。

无论在 K8S 中，还是在 Docker 中，如果我们需要在外部访问服务，端口转发是个必不可少的部分。当然，你可能会问基本上没有任何地方提到说 `socat` 是一个依赖项啊，别急，我们来看下[ K8S 的源码](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/dockershim/docker_streaming.go#L189-L192)便知道了。

```
func portForward(client libdocker.Interface, podSandboxID string, port int32, stream io.ReadWriteCloser) error {
    // 省略了和 socat 无关的代码

	socatPath, lookupErr := exec.LookPath("socat")
	if lookupErr != nil {
		return fmt.Errorf("unable to do port forwarding: socat not found.")
	}

	args := []string{"-t", fmt.Sprintf("%d", containerPid), "-n", socatPath, "-", fmt.Sprintf("TCP4:localhost:%d", port)}

    // ...
}
```

`socat` 的安装很简单 CentOS 下执行 `sudo yum install -y socat` ，Debian/Ubuntu 下执行 `sudo apt-get install -y socat` 即可完成安装。

### 初始化集群

所有的准备工作已经完成，我们开始真正创建一个 K8S 集群。 **注意：如果需要配置 Pod 网络方案，请先阅读本章最后的部分 [配置集群网络](https://juejin.im/book/5b9b2dc86fb9a05d0f16c8ac/section/5b9b8346f265da0af03375ed#配置集群网络)**

```
[root@master ~]# kubeadm init                                                                                             
[init] using Kubernetes version: v1.11.3               
[preflight] running pre-flight checks
...
I0920 01:09:09.602908   17966 kernel_validator.go:81] Validating kernel version
I0920 01:09:09.603001   17966 kernel_validator.go:96] Validating kernel config
        [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 18.03.1-ce. Max validated version: 17.03
[preflight/images] Pulling images required for setting up a Kubernetes cluster
[preflight/images] This might take a minute or two, depending on the speed of your internet connection
[preflight/images] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[preflight] Activating the kubelet service
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
...
[markmaster] Marking the node master as master by adding the label "node-role.kubernetes.io/master=''"
[markmaster] Marking the node master as master by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

...

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 202.182.112.120:6443 --token t14kzc.vjurhx5k98dpzqdc --discovery-token-ca-cert-hash sha256:d64f7ce1af9f9c0c73d2d737fd0095456ad98a2816cb5527d55f984c8aa8a762
```

以上省略了部分输出。

我们从以上日志中可以看到，创建集群时会检查内核版本，Docker 版本等信息，这里提示 Docker 版本较高，我们忽略这个提示。

然后会下载一些镜像，当然这里提示我们可以通过执行 `kubeadm config images pull` 提前去下载镜像。我们来看下

```
[root@master ~]# kubeadm config images pull
[config/images] Pulled k8s.gcr.io/kube-apiserver-amd64:v1.11.3
[config/images] Pulled k8s.gcr.io/kube-controller-manager-amd64:v1.11.3
[config/images] Pulled k8s.gcr.io/kube-scheduler-amd64:v1.11.3
[config/images] Pulled k8s.gcr.io/kube-proxy-amd64:v1.11.3
[config/images] Pulled k8s.gcr.io/pause:3.1
[config/images] Pulled k8s.gcr.io/etcd-amd64:3.2.18
[config/images] Pulled k8s.gcr.io/coredns:1.1.3
```

对于国内用户使用 `kubeadm` 创建集群时，可能遇到的问题便是这些镜像下载不下来，最终导致创建失败。所以我在国内的代码托管平台提供了一个[仓库](https://gitee.com/K8S-release/kubeadm) 可以 clone 该项目，进入 `v1.11.3` 目录，对每个 `tar` 文件执行 `sudo docker load -i xx.tar` 即可将镜像导入。

或者可使用[阿里云提供的镜像](https://dev.aliyun.com/list.html?namePrefix=google-containers)，只需要将 `k8s.gcr.io` 替换为 `registry.aliyuncs.com/google_containers` ，执行 `docker pull` 后再 `docker tag` 重 tag 即可。

继续看上面的日志，`kubeadm init` 执行起见生成了一些文件，而这些文件我们先前在 kubelet server 的 `Drop-in` 的配置中配置过。

生成这些配置文件后，将启动 `kubelet` 服务，生成一系列的证书和相关的配置之类的，并增加一些扩展。

最终集群创建成功，并提示可在任意机器上使用指定命令加入集群。

## 验证

在上面的步骤中，我们已经安装了 K8S 的 CLI 工具 `kubectl`，我们使用此工具查看集群信息：

```
[root@master ~]# kubectl cluster-info
Kubernetes master is running at http://localhost:8080

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
The connection to the server localhost:8080 was refused - did you specify the right host or port?
[root@master ~]# kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

使用 `kubectl cluster-info` 可查看集群 master 和集群服务的地址，但我们也注意到最后有一句报错 `connection ... refused` 很显然这里存在错误。

`kubectl get nodes` 可查看集群中 `Node` 信息，同样报错。

在上面我们提到过，K8S 默认会监听一些端口，但并不是 `8080` 端口，由此可知，我们的 `kubectl` 配置有误。

### 配置 kubectl

- 使用 `kubectl` 的参数 `--kubeconfig` 或者环境变量 `KUBECONFIG` 。

  ```
  [root@master ~]# kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes                                                
  NAME      STATUS     ROLES     AGE       VERSION
  master    NotReady   master    13h       v1.11.3
  [root@master ~]#
  [root@master ~]# KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes                                                  
  NAME      STATUS     ROLES     AGE       VERSION
  master    NotReady   master    13h       v1.11.3
  ```

- 使用传参的方式未免太繁琐，我们也可以更改默认配置文件

  ```
  [root@master ~]# mkdir -p $HOME/.kube
  [root@master ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  [root@master ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
  [root@master ~]# kubectl get nodes                                                  
  NAME      STATUS     ROLES     AGE       VERSION
  master    NotReady   master    13h       v1.11.3
  ```

### 配置集群网络

通过上面的配置，我们已经可以正常获取 `Node` 信息。但通过第 2 章，我们了解到 `Node` 都有 `status`，而此时我们唯一的 `Node` 是 `NotReady`。我们通过给 `kubectl` 传递 `-o` 参数更改输出格式，查看更详细的情况。

```
[root@master ~]# kubectl get nodes -o yaml
apiVersion: v1       
items:                                       
- apiVersion: v1                              
  kind: Node
  ...
  status:
    addresses:
    - address: master
      type: Hostname
    ...
    - lastHeartbeatTime: 2018-09-20T14:45:45Z
      lastTransitionTime: 2018-09-20T01:09:48Z
      message: 'runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady
        message:docker: network plugin is not ready: cni config uninitialized'
      reason: KubeletNotReady
      status: "False"
      type: Ready
    ...
```

从以上输出中，我们可以看到 master 处于 `NotReady` 的原因是 `network plugin is not ready: cni config uninitialized` 那么 `CNI` 是什么呢？`CNI` 是 Container Network Interface 的缩写，是 K8S 用于配置 Linux 容器网络的接口规范。

关于网络的选择，我们此处不做过多介绍，我们暂时选择一个被广泛使用的方案 `flannel`。 但注意，如果要使用 `flannel` 需要在 `kubeadm init` 的时候，传递 `--pod-network-cidr=10.244.0.0/16` 参数。另外需要查看 `/proc/sys/net/bridge/bridge-nf-call-iptables` 是否已设置为 `1`。 可以通过 `sysctl net.bridge.bridge-nf-call-iptables=1` 更改配置。

我们在前面创建集群时，并没有传递任何参数。为了能使用 `flannel` , 所以我们需要先将集群重置。使用 `kubeadm reset`

```
[root@master ~]# kubeadm reset 
[reset] WARNING: changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] are you sure you want to proceed? [y/N]: y
[preflight] running pre-flight checks
[reset] stopping the kubelet service
[reset] unmounting mounted directories in "/var/lib/kubelet"
[reset] removing kubernetes-managed containers
[reset] cleaning up running containers using crictl with socket /var/run/dockershim.sock
[reset] failed to list running pods using crictl: exit status 1. Trying to use docker instead[reset] deleting contents of stateful directories: [/var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes /var/lib/etcd]
[reset] deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
```

重新初始化集群，并传递参数。

```
[root@master ~]# kubeadm init --pod-network-cidr=10.244.0.0/16
[init] using Kubernetes version: v1.11.3
...
Your Kubernetes master has initialized successfully!
```

**注意：这里会重新生成相应证书等配置，需要按上面的内容重新配置 kubectl。**

此时，`CNI` 也尚未初始化完成，我们还需完成以下的步骤。

```
# 注意，这里的 flannel 配置仅适用于 1.11 版本的 K8S，若安装其他版本的 K8S 需要替换掉此链接
[root@master ~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds created
```

稍等片刻，再次查看 Node 状态：

```
[root@master ~]# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    12m       v1.11.3
```

可以看到 status 已经是 `Ready` 状态。根据第 3 章的内容，我们知道 K8S 中最小的调度单元是 `Pod` 我们来看下集群中现有 `Pod` 的状态。

```
[root@master ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                             READY     STATUS              RESTARTS   AGE 
kube-system   coredns-78fcdf6894-h7pkc         0/1       ContainerCreating   0          12m
kube-system   coredns-78fcdf6894-lhlks         0/1       ContainerCreating   0          12m
kube-system   etcd-master                      1/1       Running             0          5m
kube-system   kube-apiserver-master            1/1       Running             0          5m
kube-system   kube-controller-manager-master   1/1       Running             0          5m
kube-system   kube-flannel-ds-tqvck            1/1       Running             0          6m
kube-system   kube-proxy-25tk2                 1/1       Running             0          12m
kube-system   kube-scheduler-master            1/1       Running             0          5m
```

我们发现有两个 `coredns` 的 `Pod` 是 `ContainerCreating` 的状态，但并未就绪。根据第 3 章的内容，我们知道 `Pod` 实际会有一个调度过程，此处我们暂且不论，后续章节再对此进行解释。

### 新增 Node

我们按照刚才执行完 `kubeadm init` 后，给出的信息，在新的机器上执行 `kubeadm join` 命令。

```
[root@node1 ~]# kubeadm join 202.182.112.120:6443 --token t14kzc.vjurhx5k98dpzqdc --discovery-token-ca-cert-hash sha256:d64f7ce1af9f9c0c73d2d737fd0095456ad98a2816cb5527d55f984c8aa8a762
[preflight] running pre-flight checks
        [WARNING RequiredIPVSKernelModulesAvailable]: the IPVS proxier will not be used, because the following required kernel modules are not loaded: [ip_vs ip_vs_rr ip_vs_wrr ip_vs_sh] or no builtin kernel ipvs support: map[ip_vs:{} ip_vs_rr:{} ip_vs_wrr:{} ip_vs_sh:{} nf_conntrack_ipv4:{}]
you can solve this problem with following methods:
 1. Run 'modprobe -- ' to load missing kernel modules;
2. Provide the missing builtin kernel ipvs support

I0921 04:00:54.805439   10677 kernel_validator.go:81] Validating kernel version                                                  
I0921 04:00:54.805604   10677 kernel_validator.go:96] Validating kernel config                                                   
        [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 18.03.1-ce. Max validated version: 17.03
[discovery] Trying to connect to API Server "202.182.112.120:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://202.182.112.120:6443"
[discovery] Requesting info from "https://202.182.112.120:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "202.182.112.120:6443"
[discovery] Successfully established connection with API Server "202.182.112.120:6443"
[kubelet] Downloading configuration for the kubelet from the "kubelet-config-1.11" ConfigMap in the kube-system namespace
[kubelet] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[preflight] Activating the kubelet service
[tlsbootstrap] Waiting for the kubelet to perform the TLS Bootstrap...
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "node1" as an annotation

This node has joined the cluster:
* Certificate signing request was sent to master and a response
  was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```

上面的命令执行完成，提示已经成功加入集群。 此时，我们在 master 上查看下当前集群状态。

```
[root@master ~]# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    12m       v1.11.3
node1     Ready     <none>    7m        v1.11.3
```

可以看到 node1 已经加入了集群。

## 总结

在本节中，我们选择官方推荐的 `kubeadm` 工具在服务器上搭建了一套有两个节点的集群。

`kubeadm` 可以自动的拉取相关组件的 Docker 镜像，并将其“组织”起来，免去了我们逐个部署相关组件的麻烦。

我们首先学习到了部署 K8S 时需要对系统做的基础配置，其次安装了一些必要的工具，以便 K8S 的功能可正常运行。

其次，我们学习到了 CNI 相关的知识，并在集群中部署了 `flannel` 网络方案。

最后，我们学习了增加 Node 的方法，以便后续扩展集群。

集群搭建方面的学习暂时告一段落，但这并不是结束，这才是真正的开始，从下一章开始，我们要学习集群管理相关的内容，学习如何真正使用 K8S 。

# 集群管理：初识 kubectl

从本节开始，我们来学习 K8S 集群管理相关的知识。通过前面的学习，我们知道 K8S 遵循 C/S 架构，官方也提供了 CLI 工具 `kubectl` 用于完成大多数集群管理相关的功能。当然凡是你可以通过 `kubectl` 完成的与集群交互的功能，都可以直接通过 API 完成。

对于我们来说 `kubectl` 并不陌生，在第 3 章讲 K8S 整体架构时，我们首次提到了它。在第 4 章和第 5 章介绍了两种安装 `kubectl` 的方式故而本章不再赘述安装的部分。

## 整体概览

首先我们在终端下执行下 `kubectl`:

```
➜  ~ kubectl                
kubectl controls the Kubernetes cluster manager.
...
Usage:
  kubectl [flags] [options]
```

`kubectl` 已经将命令做了基本的归类，同时显示了其一般的用法 `kubectl [flags] [options]` 。

使用 `kubectl options` 可以看到所有全局可用的配置项。

## 基础配置

在我们的用户家目录，可以看到一个名为 `.kube/config` 的配置文件，我们来看下其中的内容（此处以本地的 minikube 集群为例）。

```
➜  ~ ls $HOME/.kube/config 
/home/tao/.kube/config
➜  ~ cat $HOME/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/tao/.minikube/ca.crt
    server: https://192.168.99.101:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/tao/.minikube/client.crt
    client-key: /home/tao/.minikube/client.key
```

`$HOME/.kube/config` 中主要包含着：

- K8S 集群的 API 地址
- 用于认证的证书地址

当然，我们在第 5 章时，也已经说过，也可以使用 `--kubeconfig` 或者环境变量 `KUBECONFIG` 来传递配置文件。

另外如果你并不想使用配置文件的话，你也可以通过使用直接传递相关参数来使用，例如：

```
➜  ~ kubectl --client-key='/home/tao/.minikube/client.key' --client-certificate='/home/tao/.minikube/client.crt' --server='https://192.168.99.101:8443'  get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    2d        v1.11.3
```

## 从 `get` 说起

无论是第 4 章还是第 5 章，当我们创建集群后，我们都做了两个相同的事情，一个是执行 `kubectl get nodes` 另一个则是 `kubectl cluster-info`，我们先从查看集群内 `Node` 开始。

这里我们使用了一个本地已创建好的 `minikube` 集群。

```
➜  ~ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    2d        v1.11.3
➜  ~ kubectl get node
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    2d        v1.11.3
➜  ~ kubectl get no
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    2d        v1.11.3
```

可以看到以上三种“名称”均可获取当前集群内 `Node` 信息。这是为了便于使用而增加的别名和缩写。

如果我们想要看到更详细的信息呢？可以通过传递 `-o` 参数以得到不同格式的输出。

```
➜  ~ kubectl get nodes -o wide 
NAME       STATUS    ROLES     AGE       VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready     master    2d        v1.11.3   10.0.2.15     <none>        Buildroot 2018.05   4.15.0           docker://17.12.1-ce
```

当然也可以传递 `-o yaml` 或者 `-o json` 得到更加详尽的信息。

使用 `-o json` 将内容以 JSON 格式输出时，可以配合 [`jq`](https://stedolan.github.io/jq/) 进行内容提取。例如：

```
➜  ~ kubectl get nodes -o json | jq ".items[] | {name: .metadata.name} + .status.nodeInfo"
{
  "name": "minikube",
  "architecture": "amd64",
  "bootID": "d675d75b-e58e-40db-8910-6e5dda9e7cf9",
  "containerRuntimeVersion": "docker://17.12.1-ce",
  "kernelVersion": "4.15.0",
  "kubeProxyVersion": "v1.11.3",
  "kubeletVersion": "v1.11.3",
  "machineID": "078e2d22629747178397e29cf1c96cc7",
  "operatingSystem": "linux",
  "osImage": "Buildroot 2018.05",
  "systemUUID": "4073906D-69A1-46EE-A08C-0252D9F79893"
}
```

以此方法可得到 `Node` 的基础信息。

那么除了 `Node` 外我们还可以查看那些资源或别名呢？可以通过 `kubectl api-resources` 查看服务端支持的 API 资源及别名和描述等信息。

## 答疑解惑 `explain`

当通过上面的命令拿到所有支持的 API 资源列表后，虽然后面基本都有一个简单的说明，是不是仍然感觉一头雾水？

别担心，在我们使用 Linux 的时候，我们有 `man` ，在使用 `kubectl` 的时候，我们除了 `--help` 外还有 `explain` 可帮我们进行说明。 例如：

```
➜  ~ kubectl explain node                                                                                                              
KIND:     Node
VERSION:  v1

DESCRIPTION:              
     Node is a worker node in Kubernetes. Each node will have a unique
     identifier in the cache (i.e. in etcd).

# ... 省略输出
```

## 总结

本节我们大致介绍了 `kubectl` 的基础使用，尤其是最常见的 `get` 命令。可通过传递不同参数获取不同格式的输出，配合 `jq` 工具可方便的进行内容提取。

以及关于 `kubectl` 的配置文件和无配置文件下通过传递参数直接使用等。

对应于我们前面提到的 K8S 架构，本节相当于 `CURD` 中的 `R` 即查询。查询对于我们来说，既是我们了解集群的第一步，同时也是后续验证操作结果或集群状态必不可少的技能。

当然，你在集群管理中可能会遇到各种各样的问题，单纯依靠 `get` 并不足以定位问题，我们在第 21 节中将介绍 Troubleshoot 的思路及方法。

下节我们来学习关于 `C` 的部分，即创建。

# 集群管理：以 Redis 为例-部署及访问

上节我们已经学习了 `kubectl` 的基础使用，本节我们使用 `kubectl` 在 K8S 中进行部署。

**前面我们已经说过，Pod 是 K8S 中最小的调度单元，所以我们无法直接在 K8S 中运行一个 container 但是我们可以运行一个 Pod 而这个 Pod 中只包含一个 container 。**

## 从 `kubectl run` 开始

`kubectl run` 的基础用法如下：

```
Usage:
  kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...] [options]
```

`NAME` 和 `--image` 是必需项。分别代表此次部署的名字及所使用的镜像，其余部分之后进行解释。当然，在我们实际使用时，推荐编写配置文件并通过 `kubectl create` 进行部署。

## 使用最小的 Redis 镜像

在 Redis 的[官方镜像列表](https://hub.docker.com/_/redis/)可以看到有很多的 tag 可供选择，其中使用 [Alpine Linux](https://alpinelinux.org/) 作为基础的镜像体积最小，下载较为方便。我们选择 `redis:alpine` 这个镜像进行部署。

## 部署

现在我们只部署一个 Redis 实例。

```
➜  ~ kubectl run redis --image='redis:alpine'
deployment.apps/redis created
```

可以看到提示 `deployment.apps/redis created` 这个稍后进行解释，我们使用 `kubectl get all` 来看看到底发生了什么。

```
➜  ~ kubectl get all
NAME                         READY     STATUS    RESTARTS   AGE
pod/redis-7c7545cbcb-2m6rp   1/1       Running   0          30s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   32s

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1         1         1            1           30s

NAME                               DESIRED   CURRENT   READY     AGE
replicaset.apps/redis-7c7545cbcb   1         1         1         30s
```

可以看到其中有我们刚才执行 `run` 操作后创建的 `deployment.apps/redis`，还有 `replicaset.apps/redis-7c7545cbcb`, `service/kubernetes` 以及 `pod/redis-7c7545cbcb-f984p`。

使用 `kubectl get all` 输出内容的格式 `/` 前代表类型，`/` 后是名称。

这些分别代表什么含义？

### Deployment

`Deployment` 是一种高级别的抽象，允许我们进行扩容，滚动更新及降级等操作。我们使用 `kubectl run redis --image='redis:alpine` 命令便创建了一个名为 `redis` 的 `Deployment`，并指定了其使用的镜像为 `redis:alpine`。

同时 K8S 会默认为其增加一些标签（`Label`）。我们可以通过更改 `get` 的输出格式进行查看。

```
➜  ~ kubectl get deployment.apps/redis -o wide 
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES         SELECTOR
redis     1         1         1            1           40s       redis        redis:alpine   run=redis
➜  ~ kubectl get deploy redis -o wide          
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES         SELECTOR
redis     1         1         1            1           40s       redis        redis:alpine   run=redis
```

那么这些 `Label` 有什么作用呢？它们可作为选择条件进行使用。如：

```
➜  ~ kubectl get deploy -l run=redis -o wide 
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES         SELECTOR
redis     1         1         1            1           11h       redis        redis:alpine   run=redis
➜  ~ kubectl get deploy -l run=test -o wide  # 由于我们并没有创建过 test 所以查不到任何东西
No resources found.
```

我们在应用部署或更新时总是会考虑的一个问题是如何平滑升级，利用 `Deployment` 也能很方便的进行金丝雀发布（Canary deployments）。这主要也依赖 `Label` 和 `Selector`， 后面我们再详细介绍如何实现。

`Deployment` 的创建除了使用我们这里提到的方式外，更推荐的方式便是使用 `yaml` 格式的配置文件。在配置文件中主要是声明一种预期的状态，而其他组件则负责协同调度并最终达成这种预期的状态。当然这也是它的关键作用之一，将 `Pod` 托管给下面将要介绍的 `ReplicaSet`。

### ReplicaSet

`ReplicaSet` 是一种较低级别的结构，允许进行扩容。

我们上面已经提到 `Deployment` 主要是声明一种预期的状态，并且会将 `Pod` 托管给 `ReplicaSet`，而 `ReplicaSet` 则会去检查当前的 `Pod` 数量及状态是否符合预期，并尽量满足这一预期。

`ReplicaSet` 可以由我们自行创建，但一般情况下不推荐这样去做，因为如果这样做了，那其实就相当于跳过了 `Deployment` 的部分，`Deployment` 所带来的功能或者特性我们便都使用不到了。

除了 `ReplicaSet` 外，我们还有一个选择名为 `ReplicationController`，这两者的主要区别更多的在选择器上，我们后面再做讨论。现在推荐的做法是 `ReplicaSet` 所以不做太多解释。

`ReplicaSet` 可简写为 `rs`，通过以下命令查看：

```
➜  ~ kubectl get rs -o wide
NAME               DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES         SELECTOR                           
redis-7c7545cbcb   1         1         1         11h       redis        redis:alpine   pod-template-hash=3731017676,run=redis
```

在输出结果中，我们注意到这里除了我们前面看到的 `run=redis` 标签外，还多了一个 `pod-template-hash=3731017676` 标签，这个标签是由 `Deployment controller` 自动添加的，目的是为了防止出现重复，所以将 `pod-template` 进行 hash 用作唯一性标识。

### Service

`Service` 简单点说就是为了能有个稳定的入口访问我们的应用服务或者是一组 `Pod`。通过 `Service` 可以很方便的实现服务发现和负载均衡。

```
➜  ~ kubectl get service -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE       SELECTOR
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   16m        <none>
```

通过使用 `kubectl` 查看，能看到主要会显示 `Service` 的名称，类型，IP，端口及创建时间和选择器等。我们来具体拆解下。

#### 类型

`Service` 目前有 4 种类型：

- `ClusterIP`： 是 K8S 当前默认的 `Service` 类型。将 service 暴露于一个仅集群内可访问的虚拟 IP 上。
- `NodePort`： 是通过在集群内所有 `Node` 上都绑定固定端口的方式将服务暴露出来，这样便可以通过 `:` 访问服务了。
- `LoadBalancer`： 是通过 `Cloud Provider` 创建一个外部的负载均衡器，将服务暴露出来，并且会自动创建外部负载均衡器路由请求所需的 `Nodeport` 或 `ClusterIP` 。
- `ExternalName`： 是通过将服务由 DNS CNAME 的方式转发到指定的域名上将服务暴露出来，这需要 `kube-dns` 1.7 或更高版本支持。

#### 实践

上面已经说完了 `Service` 的基本类型，而我们也已经部署了一个 Redis ,当还无法访问到该服务，接下来我们将刚才部署的 Redis 服务暴露出来。

```
➜  ~ kubectl expose deploy/redis --port=6379 --protocol=TCP --target-port=6379 --name=redis-server  
service/redis-server exposed
➜  ~ kubectl get svc -o wide                                                                       
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE       SELECTOR
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP    49m       <none>
redis-server   ClusterIP   10.108.105.63   <none>        6379/TCP   4s        run=redis
```

通过 `kubectl expose` 命令将 redis server 暴露出来，这里需要进行下说明：

- `port`： 是 `Service` 暴露出来的端口，可通过此端口访问 `Service`。
- `protocol`： 是所用协议。当前 K8S 支持 TCP/UDP 协议，在 1.12 版本中实验性的加入了对 [SCTP 协议](https://zh.wikipedia.org/zh-hans/流控制传输协议)的支持。默认是 TCP 协议。
- `target-port`： 是实际服务所在的目标端口，请求由 `port` 进入通过上述指定 `protocol` 最终流向这里配置的端口。
- `name`： `Service` 的名字，它的用处主要在 dns 方面。
- `type`： 是前面提到的类型，如果没指定默认是 `ClusterIP`。

现在我们的 redis 是使用的默认类型 `ClusterIP`，所以并不能直接通过外部进行访问，我们使用 `port-forward` 的方式让它可在集群外部访问。

```
➜  ~ kubectl port-forward svc/redis-server 6379:6379
Forwarding from 127.0.0.1:6379 -> 6379
Forwarding from [::1]:6379 -> 6379
Handling connection for 6379
```

在另一个本地终端内可通过 redis-cli 工具进行连接：

```
➜  ~ redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> ping
PONG
```

当然，我们也可以使用 `NodePort` 的方式对外暴露服务。

```
➜  ~ kubectl expose deploy/redis --port=6379 --protocol=TCP --target-port=6379 --name=redis-server-nodeport --type=NodePort
service/redis-server-nodeport exposed
➜  ~ kubectl get service/redis-server-nodeport -o wide 
NAME                    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE       SELECTOR
redis-server-nodeport   NodePort   10.109.248.204   <none>        6379:31913/TCP   11s       run=redis
```

我们可以通过任意 `Node` 上的 31913 端口便可连接我们的 redis 服务。当然，这里需要注意的是这个端口范围其实是可以通过 `kube-apiserver` 的 `service-node-port-range` 进行配置的，默认是 `30000-32767`。

### Pod

第二节中，我们提到过 `Pod` 是 K8S 中的最小化部署单元。我们看下当前集群中 `Pod` 的状态。

```
➜  ~ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
redis-7c7545cbcb-jwcf2   1/1       Running   0          8h
```

我们进行一次简单的扩容操作。

```
➜  ~ kubectl scale deploy/redis --replicas=2
deployment.extensions/redis scaled
➜  ~ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
redis-7c7545cbcb-jwcf2   1/1       Running   0          8h
redis-7c7545cbcb-wzh6w   1/1       Running   0          4s
```

可以看到 `Pod` 数已经增加，并且也已经是 `Running` 的状态了。(当然在生产环境中 Redis 服务的扩容并不是使用这种方式进行扩容的，需要看实际的部署方式以及业务的使用姿势。)

## 总结

本节我们使用 Redis 作为例子，学习了集群管理相关的基础知识。学习了如何进行应用部署， `Service` 的基础类型以及如何通过 `port-forward` 或 `NodePort` 等方式将服务提供至集群的外部访问。

同时我们学习了应用部署中主要会涉及到的几类资源 `Deployment`，`Replicaset`，`Service` 和 `Pod` 等。对这些资源及它们之间关系的掌握，对于后续集群维护或定位问题有很大的帮助。

下节，我们开始学习在生产环境中使用 K8S 至关重要的一环，权限控制。

# 安全重点: 认证和授权

本节我们将开始学习将 K8S 应用于生产环境中至关重要的一环，权限控制。当然，不仅是 K8S 对于任何应用于生产环境中的系统，权限管理或者说访问控制都是很重要的。

## 整体概览

通过前面的学习，我们已经知道 K8S 中几乎所有的操作都需要经过 `kube-apiserver` 处理，所以为了安全起见，K8S 为它提供了三类安全访问的措施。分别是：用于识别用户身份的认证（Authentication），用于控制用户对资源访问的授权（Authorization）以及用于资源管理方面的准入控制（Admission Control）。

下面的图基本展示了这一过程。来自客户端的请求分别经过认证，授权，准入控制之后，才能真正执行。

当然，这里说**基本展示**是因为我们可以直接通过 `kubectl proxy` 的方式直接通过 HTTP 请求访问 `kube-apiserver` 而无需任何认证过程。

另外，也可通过在 `kube-apiserver` 所启动的机器上，直接访问启动时 `--insecure-port` 参数配置的端口进行绕过认证和授权，默认是 8080。为了避免安全问题，也可将此参数设置为 0 以规避问题。注意：这个参数和 `--insecure-bind-address` 都已过期，并将在未来的版本移除。

```
+-----------------------------------------------------------------------------------------------------------+
|                                                                                                           |
|               +---------------------------------------------------------------------------+    +--------+ |
|               |                                                                           |    |        | |
| +--------+    |   +------------------+   +----------------+   +--------------+   +------+ |    |        | |
| |        |    |   |                  |   |                |   | Admission    |   |      | |    |        | |
| | Client +------> | Authentication   +-> | Authorization  +-> | Control      +-> |Logic | +--> | Others | |
| |        |    |   |                  |   |                |   |              |   |      | |    |        | |
| +--------+    |   +------------------+   +----------------+   +--------------+   +------+ |    |        | |
|               |                                                                           |    |        | |
|               |                                                                           |    |        | |
|               |                          Kube-apiserver                                   |    |        | |
|               +---------------------------------------------------------------------------+    +--------+ |
|                                                                                                           |
+-----------------------------------------------------------------------------------------------------------+
```

## 认证（Authentication）

认证，无非是判断当前发起请求的用户身份是否正确。例如，我们通常登录服务器时候需要输入用户名和密码，或者 SSH Keys 之类的。

在讲认证前，我们应该先理一下 K8S 中的用户。

K8S 中有两类用户，一般用户及 `Service Account`。

- 一般用户：一般用户只能通过外部服务进行管理，由管理员进行私钥分发。这也意味着 K8S 中并没有任何表示一般用户的对象，所以一般用户是无法通过 API 直接添加到集群的。
- `Service Account`：由 K8S API 管理的用户，与特定的 `NameSpace`（命名空间）绑定。由 `API Server` 自动创建或者通过 API 手动进行创建。 同时，它会自动挂载到 `Pod` 中容器的 `/var/run/secrets/kubernetes.io/serviceaccount/` 目录中，其中会包含 `NameSpace` `token` 等信息，并允许集群内进程与 `API Server` 进行交互。

对集群操作的 API 都是与用户相关联的，或者被视为匿名请求。匿名请求可通过 `kube-apiserver` 的 `--anonymous-auth` 参数进行控制，默认是开启的，匿名用户默认的用户名为 `system:anonymous`，所属组为 `system:unauthenticated`。

理完 K8S 中的用户，我们来看下 K8S 中的认证机制。

K8S 支持以下认证机制：

- X509 客户端证书：这个认证机制我们并不陌生，我们前面搭建集群时，虽然没有指定配置文件，但 `kubeadm` 已经添加了默认参数 `--client-ca-file=/etc/kubernetes/pki/ca.crt` 而在进行认证时，将会使用客户端证书 subject 的 `CN` 域（Common Name）用作用户名，`O` 域（Organization）用作组名。
- 引导 Token：这个我们也不会陌生，前面我们搭建集群时，当集群通过 `kubeadm init` 初始化完成后，将会展示一行提示，其中便携带着引导 Token。如果不使用 `kubeadm` 时，需要设置 `--enable-bootstrap-token-auth=true`。
- 静态 Token 文件：启动 `Kube-apiserver` 时，设置 `--token-auth-file=SOMEFILE` 并在请求时，加上 `Authorization: Bearer TOKEN` 的请求头即可。
- 静态密码文件：与静态 Token 文件类似，设置 `--basic-auth-file=SOMEFILE` 并在请求时，加上 `Authorization: Basic BASE64ENCODED(USER:PASSWORD)` 的头即可。
- Service Account Token：这是默认启用的机制，关于 `Service Account` 前面也已经介绍过了，不再赘述。
- OpenID：其实是提供了 [OAuth2](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html) 的认证支持，像 Azure 或 Google 这类云厂商都提供了相关支持。
- 认证代理：主要是配合身份验证代理进行使用，比如提供一个通用的授权网关供用户使用。
- Webhook：提供 Webhook 配合一个远端服务器使用。

可选择同时开启多个认证机制。比如当我们使用 `kubeadm` 创建集群时，默认便会开启 X509 客户端证书和引导 Token 等认证机制。

## 授权（Authorization）

授权，也就是在验证当前发起请求的用户是否有相关的权限。例如，我们在 Linux 系统中常见的文件夹权限之类的。

授权是以认证的结果为基础的，授权机制检查用户通过认证后的请求中所包含的属性来进行判断。

K8S 支持多种授权机制，用户想要正确操作资源，则必须获得授权，所以 K8S 默认情况下的权限都是拒绝。当某种授权机制通过或者拒绝后，便会立即返回，不再去请求其他的授权机制；当所有授权机制都未通过时便会返回 403 错误了。

K8S 支持以下授权机制：

- ABAC(Attribute-Based Access Control)：基于属性的访问控制，在使用时需要先配置 `--authorization-mode=ABAC` 和 `--authorization-policy-file=SOME_FILENAME` 。ABAC 本身设计是非常好的，但是在 K8S 中使用却有点过于繁琐，这里不再赘述。
- RBAC(Role-based access control)：基于角色的访问控制，自 K8S 1.6 开始 beta，1.8 进入稳定版，已被大量使用。而当我们使用 `kubeadm` 安装集群的时候，默认将会添加 `--authorization-mode=Node,RBAC` 的参数，表示同时开启 `Node` 和 `RBAC` 授权机制。当然，如果你对 [MongoDB](https://www.mongodb.com/cn) 有所了解或者比较熟悉的话，这部分的内容就会很容易理解，因为 MongoDB 的权限控制也使用了 `RBAC` （Role-based access control）。
- Node：这是一种特殊用途的授权机制，专门用于对 `kubelet` 发出的 API 请求做授权验证。
- Webhook：使用外部的 Server 通过 API 进行授权校验，需要在启动时候增加 `--authorization-webhook-config-file=SOME_FILENAME` 以及 `--authorization-mode=Webhook`
- AlwaysAllow：默认配置，允许全部。
- AlwaysDeny：通常用于测试，禁止全部。

## 角色（Role）

上面提到了 `RBAC`，为了能更好的理解，我们需要先认识下 K8S 中的角色。K8S 中的角色从类别上主要有两类，`Role` 和 `ClusterRole`。

- `Role`：可以当作是一组权限的集合，但被限制在某个 `Namespace` 内（K8S 的 `Namespace`）。
- `ClusterRole`：对于集群级别的资源是不被 `Namespace` 所限制的，并且还有一些非资源类的请求，所以便产生了它。

当已经了解到角色后，剩下给用户授权也就只是需要做一次绑定即可。在 K8S 中将这一过程称之为 binding，即 `rolebinding` 和 `clusterrolebinding`。 我们来看下集群刚初始化后的情况：

```
➜  ~ kubectl get roles --all-namespaces=true
NAMESPACE     NAME                                             AGE
kube-public   kubeadm:bootstrap-signer-clusterinfo             1h
kube-public   system:controller:bootstrap-signer               1h
kube-system   extension-apiserver-authentication-reader        1h
kube-system   kube-proxy                                       1h
kube-system   kubeadm:kubelet-config-1.12                      1h
kube-system   kubeadm:nodes-kubeadm-config                     1h
kube-system   system::leader-locking-kube-controller-manager   1h
kube-system   system::leader-locking-kube-scheduler            1h
kube-system   system:controller:bootstrap-signer               1h
kube-system   system:controller:cloud-provider                 1h
kube-system   system:controller:token-cleaner                  1h
➜  ~ kubectl get rolebindings --all-namespaces=true
NAMESPACE     NAME                                             AGE
kube-public   kubeadm:bootstrap-signer-clusterinfo             1h
kube-public   system:controller:bootstrap-signer               1h
kube-system   kube-proxy                                       1h
kube-system   kubeadm:kubelet-config-1.12                      1h
kube-system   kubeadm:nodes-kubeadm-config                     1h
kube-system   system::leader-locking-kube-controller-manager   1h
kube-system   system::leader-locking-kube-scheduler            1h
kube-system   system:controller:bootstrap-signer               1h
kube-system   system:controller:cloud-provider                 1h
kube-system   system:controller:token-cleaner                  1h
```

可以看到默认已经存在了一些 `role` 和 `rolebindings`。 对于这部分暂且不做过多说明，我们来看下对于集群全局有效的 `ClusterRole` 。

```
➜  ~ kubectl get clusterroles
NAME                                                                   AGE
admin                                                                  1h
cluster-admin                                                          1h
edit                                                                   1h
flannel                                                                1h
system:aggregate-to-admin                                              1h
system:aggregate-to-edit                                               1h
system:aggregate-to-view                                               1h
system:auth-delegator                                                  1h
system:aws-cloud-provider                                              1h
system:basic-user                                                      1h
system:certificates.k8s.io:certificatesigningrequests:nodeclient       1h
system:certificates.k8s.io:certificatesigningrequests:selfnodeclient   1h
system:controller:attachdetach-controller                              1h
system:controller:certificate-controller                               1h
system:controller:clusterrole-aggregation-controller                   1h
system:controller:cronjob-controller                                   1h
system:controller:daemon-set-controller                                1h
system:controller:deployment-controller                                1h
system:controller:disruption-controller                                1h
system:controller:endpoint-controller                                  1h
system:controller:expand-controller                                    1h
system:controller:generic-garbage-collector                            1h
system:controller:horizontal-pod-autoscaler                            1h
system:controller:job-controller                                       1h
system:controller:namespace-controller                                 1h
system:controller:node-controller                                      1h
system:controller:persistent-volume-binder                             1h
system:controller:pod-garbage-collector                                1h
system:controller:pv-protection-controller                             1h
system:controller:pvc-protection-controller                            1h
system:controller:replicaset-controller                                1h
system:controller:replication-controller                               1h
system:controller:resourcequota-controller                             1h
system:controller:route-controller                                     1h
system:controller:service-account-controller                           1h
system:controller:service-controller                                   1h
system:controller:statefulset-controller                               1h
system:controller:ttl-controller                                       1h
system:coredns                                                         1h
system:csi-external-attacher                                           1h
system:csi-external-provisioner                                        1h
system:discovery                                                       1h
system:heapster                                                        1h
system:kube-aggregator                                                 1h
system:kube-controller-manager                                         1h
system:kube-dns                                                        1h
system:kube-scheduler                                                  1h
system:kubelet-api-admin                                               1h
system:node                                                            1h
system:node-bootstrapper                                               1h
system:node-problem-detector                                           1h
system:node-proxier                                                    1h
system:persistent-volume-provisioner                                   1h
system:volume-scheduler                                                1h
view                                                                   1h
➜  ~ kubectl get clusterrolebindings
NAME                                                   AGE
cluster-admin                                          1h
flannel                                                1h
kubeadm:kubelet-bootstrap                              1h
kubeadm:node-autoapprove-bootstrap                     1h
kubeadm:node-autoapprove-certificate-rotation          1h
kubeadm:node-proxier                                   1h
system:aws-cloud-provider                              1h
system:basic-user                                      1h
system:controller:attachdetach-controller              1h
system:controller:certificate-controller               1h
system:controller:clusterrole-aggregation-controller   1h
system:controller:cronjob-controller                   1h
system:controller:daemon-set-controller                1h
system:controller:deployment-controller                1h
system:controller:disruption-controller                1h
system:controller:endpoint-controller                  1h
system:controller:expand-controller                    1h
system:controller:generic-garbage-collector            1h
system:controller:horizontal-pod-autoscaler            1h
system:controller:job-controller                       1h
system:controller:namespace-controller                 1h
system:controller:node-controller                      1h
system:controller:persistent-volume-binder             1h
system:controller:pod-garbage-collector                1h
system:controller:pv-protection-controller             1h
system:controller:pvc-protection-controller            1h
system:controller:replicaset-controller                1h
system:controller:replication-controller               1h
system:controller:resourcequota-controller             1h
system:controller:route-controller                     1h
system:controller:service-account-controller           1h
system:controller:service-controller                   1h
system:controller:statefulset-controller               1h
system:controller:ttl-controller                       1h
system:coredns                                         1h
system:discovery                                       1h
system:kube-controller-manager                         1h
system:kube-dns                                        1h
system:kube-scheduler                                  1h
system:node                                            1h
system:node-proxier                                    1h
system:volume-scheduler                                1h
```

可以看到 K8S 中默认已经有很多的 `ClusterRole` 和 `clusterrolebindings` 了，我们选择其中一个做下探究。

## 查看用户权限

我们一直都在使用 `kubectl` 对集群进行操作，那么当前用户是什么权限呢？ 对应于 `RBAC` 中又是什么情况呢？

```
➜  ~ kubectl config current-context   # 获取当前上下文
kubernetes-admin@kubernetes  # 名为 kubernetes-admin 的用户，在名为 kubernetes 的 cluster 上
➜  ~ kubectl config view users -o yaml  # 查看 user 配置，以下省略了部分内容
apiVersion: v1
clusters:                                   
- cluster:                            
    ...
contexts:                    
- context:  
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

`client-certificate-data` 的部分默认是不显示的，而它的**内容实际是通过 `base64` 加密后的证书内容**。我们可以通过通过以下方式进行查看

```
➜  ~ kubectl config view users --raw -o jsonpath='{ .users[?(@.name == "kubernetes-admin")].user.client-certificate-data}' |base64 -d  
-----BEGIN CERTIFICATE-----
MIIC8jCCAdqgAwIBAgIIGuC27C9B8LIwDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
...
kae1A/d4D5Cm5Qt7M5gr3SxqE5t+O7DP0YhuEPlfY7RzYDksYa8=
-----END CERTIFICATE-----
➜  ~ kubectl config view users --raw -o jsonpath='{ .users[?(@.name == "kubernetes-admin")].user.client-certificate-data}' |base64 -d |openssl x509 -text -noout  # 限于篇幅 省略部分输出
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 1936748965290700978 (0x1ae0b6ec2f41f0b2)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        Subject: O=system:masters, CN=kubernetes-admin
        ...
```

根据前面认证部分的内容，我们知道当前的用户是 `kubernetes-admin` （CN 域），所属组是 `system:masters` （O 域） 。

我们看下 `clusterrolebindings` 中的 `cluster-admin` 。

```
➜  ~ kubectl get clusterrolebindings  cluster-admin  -o yaml                         
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  ...
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: cluster-admin
  resourceVersion: "116"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/cluster-admin
  uid: 71c550f1-e0e4-11e8-866a-fa163e938a99
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```

重点内容在 `roleRef` 和 `subjects` 中，名为 `cluster-admin` 的 `ClusterRole` 与名为 `system:masters` 的 `Group` 相绑定。我们继续探究下它们所代表的含义。

先看看这个 `ClusterRole` 的实际内容：

```
➜  ~ kubectl get clusterrole cluster-admin -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  ...
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: cluster-admin
  resourceVersion: "58"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/cluster-admin
  uid: 71307108-e0e4-11e8-866a-fa163e938a99
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
```

`rules` 中定义了它所能操作的资源及对应动作，`*` 是通配符。

到这里，我们就可以得出结论了，当前用户 `kubernetes-admin` 属于 `system:masters` 组，而这个组与 `cluster-admin` 这个 `ClusterRole` 所绑定，所以用户也就继承了其权限。具备了对多种资源和 API 的相关操作权限。

## 实践：创建权限可控的用户

前面是通过实际用户来反推它所具备的权限，接下来我们开始实践的部分，创建用户并为它进行授权。

我们要创建的用户名为 `backend` 所属组为 `dev`。

### 创建 NameSpace

为了演示，这里创建一个新的 `NameSpace` ，名为 `work`。

```
➜  ~ kubectl create namespace work
namespace/work created
➜  ~ kubectl get ns work
NAME   STATUS   AGE
work   Active   14s
```

### 创建用户

#### 创建私钥

```
➜  ~ mkdir work
➜  ~ cd work
➜  work openssl genrsa -out backend.key 2048
Generating RSA private key, 2048 bit long modulus
..........................................+++                            
........................+++                   
e is 65537 (0x10001)            
➜  work ls                                       
backend.key 

➜  work cat backend.key                                              
-----BEGIN RSA PRIVATE KEY-----                                          
MIIEpAIBAAKCAQEAzk7blZthwSzachPxrk6pHsuaImTVh6Iw8mNDmtn6sqOqBfZS
...
bNKDWDk8HZREugaOAwjt7xaWOlr9SPCCoXrWoaA1z2215IC4qSA2Nw==
-----END RSA PRIVATE KEY-----
```

#### 使用私钥生成证书请求。前面已经讲过关于认证的部分，在这里需要指定 `subject` 信息，传递用户名和组名

```
➜  work openssl req -new -key backend.key -out backend.csr -subj "/CN=backend/O=dev"
➜  work ls
backend.csr  backend.key
➜  work cat backend.csr
-----BEGIN CERTIFICATE REQUEST-----
MIICZTCCAU0CAQAwIDEQMA4GA1UEAwwHYmFja2VuZDEMMAoGA1UECgwDZGV2MIIB
...
lpoSVlNA0trJoiEiZjUqMfXX6ogBhQC4aeRfmbXkW2ZCNxsIm3PDk1Y=
-----END CERTIFICATE REQUEST-----
```

#### 使用 CA 进行签名。K8S 默认的证书目录为 `/etc/kubernetes/pki`。

```
➜  work openssl x509 -req -in backend.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out backend.crt -days 365
Signature ok
subject=/CN=backend/O=dev
Getting CA Private Key
➜  work ls
backend.crt  backend.csr  backend.key
```

查看生成的证书文件

```
➜  work openssl x509 -in backend.crt -text -noout
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            d9:7f:62:f7:38:66:2a:7b
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        Subject: CN=backend, O=dev
        ...
```

可以看到 `CN` 域和 `O` 域已经正确设置

#### 添加 context

```
➜  work kubectl config set-credentials backend --client-certificate=/root/work/backend.crt  --client-key=/root/work/backend.key
User "backend" set.
➜  work kubectl config set-context backend-context --cluster=kubernetes --namespace=work --user=backend
Context "backend-context" created.
```

#### 使用新用户测试访问

```
➜  work kubectl --context=backend-context get pods
Error from server (Forbidden): pods is forbidden: User "backend" cannot list resource "pods" in API group "" in the namespace "work"
# 可能看得不够清楚，我们添加 `-v` 参数来显示详情
➜  work kubectl --context=backend-context get pods -n work -v 5
I1109 05:35:11.870639   18626 helpers.go:201] server response object: [{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"backend\" cannot list resource \"pods\" in API group \"\" in the namespace \"work\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}]
F1109 05:35:11.870688   18626 helpers.go:119] Error from server (Forbidden): pods is forbidden: User "backend" cannot list resource "pods" in API group "" in the namespace "work"
```

可以看到已经使用了新的 `backend` 用户，并且默认的 `Namespace` 设置成了 `work`。

#### 创建 Role

我们想要让这个用户只具备查看 `Pod` 的权限。先来创建一个配置文件。

```
➜  work cat <<EOF > backend-role.yaml 
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: work
  name: backend-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
EOF
```

创建并查看已生成的 `Role`。

```
➜  work kubectl create -f backend-role.yaml 
role.rbac.authorization.k8s.io/backend-role created
➜  work kubectl get roles  -n work -o yaml
apiVersion: v1
items:
- apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    ...
    name: backend-role
    namespace: work
    selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/work/roles/backend-role
  rules:
  - apiGroups:
    - ""
    resources:
    - pods
    verbs:
    - get
    - list
    - watch
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

#### 创建 Rolebinding

先创建一个配置文件。

```
➜  work cat <<EOF > backend-rolebind.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: backend-rolebinding          
  namespace: work
subjects:      
- kind: User
  name: backend
  apiGroup: ""     
roleRef:    
  kind: Role 
  name: backend-role
  apiGroup: ""
EOF
```

创建并查看已生成的 Rolebinding 。

```
➜  work kubectl create -f backend-rolebind.yaml
rolebinding.rbac.authorization.k8s.io/backend-rolebinding created
➜  work kubectl get rolebinding -o yaml -n work
apiVersion: v1
items:
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: backend-rolebinding
    namespace: work
    ...
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: backend-role
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: backend
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

#### 测试用户权限

```
➜  work kubectl --context=backend-context get pods -n work
No resources found.
➜  work kubectl --context=backend-context get ns
Error from server (Forbidden): namespaces is forbidden: User "backend" cannot list resource "namespaces" in API group "" at the cluster scope
➜  work kubectl --context=backend-context get deploy -n work
Error from server (Forbidden): deployments.extensions is forbidden: User "backend" cannot list resource "deployments" in API group "extensions" in the namespace "work"
```

可以看到用户已经具备查看 `Pod` 的权限，但并不能查看 `Namespace` 或者 `deployment` 等其他资源。当然，K8S 也内置了一种很方便的调试机制。

```
➜  work kubectl auth can-i list pods -n work --as="backend"
yes
➜  work kubectl auth can-i list deploy -n work --as="backend"
no - no RBAC policy matched
```

`--as` 是一种建立在 K8S 认证机制之上的机制，可以便于系统管理员验证授权情况，或进行调试。

你也可以仿照 `~/.kube/config` 文件的内容，将当前生成的证书及私钥文件等写入到配置文件中，通过指定 `KUBECONFIG` 的环境变量，或者给 `kubectl` 传递 `--kubeconfig` 参数来使用。

## 总结

本节中，我们学习了 K8S 的认证及授权逻辑，K8S 支持多种认证及授权模式，可按需使用。通过 X509 客户端证书认证的方式使用比较方便也比较推荐，在客户端证书的 `CN` 域和 `O` 域可以指定用户名和所属组名。

RBAC 的授权模式现在使用最多，可以通过对 `Role` 和 `subjects` (可以是用户或组) 进行绑定，以达到授权的目的。

最后，我们实际新创建了一个用户，并对其授予了预期的权限。在此过程中也涉及到了 `openssl` 客户端的常规操作，在之后也会常常用到。

下节，我们将开始部署实际的项目到 K8S 中，逐步掌握生成环境中对 K8S 的使用实践。

PS: 也许你会觉得切换 `Namespace` 之类的操作很繁琐，有一个项目：[kubectx](https://github.com/ahmetb/kubectx) 可帮你简化这些步骤，推荐尝试。

留言

# 应用发布: 部署实际项目

本节我们开始学习如何将实际项目部署至 K8S 中，开启生产实践之路。

## 整体概览

本节所用示例项目是一个混合了 Go，NodeJS，Python 等语言的项目，灵感来自于知名程序员 [Kenneth Reitz](https://github.com/kennethreitz) 的 [Say Thanks](https://saythanks.io/) 项目。本项目实现的功能主要有两个：1. 用户通过前端发送感谢消息 2. 有个工作进程会持续的计算收到感谢消息的排行榜。项目代码可在 [GitHub 上获得](https://github.com/tao12345666333/saythx)。接下来几节中如果需要用到此项目我会统一称之为 saythx 项目。

saythx 项目的基础结构如下图：



![img](https://user-gold-cdn.xitu.io/2018/11/18/16722d23f282fa8b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 构建镜像

### 前端

我们使用了前端框架 [Vue](https://vuejs.org/)，所以在做生产部署时，需要先在 [Node JS](http://nodejs.org/) 的环境下进行打包构建。包管理器使用的是 [Yarn](https://yarnpkg.com/)。然后使用 [Nginx](http://nginx.com/) 提供服务，并进行反向代理，将请求正确的代理至后端。

```
FROM node:10.13 as builder

WORKDIR /app
COPY . /app

RUN yarn install \
        && yarn build


FROM nginx:1.15

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html/

EXPOSE 80
```

Nginx 的配置文件如下：

```
upstream backend-up {
    server saythx-backend:8080;
}
server {
    listen       80;
    server_name  localhost;

    charset     utf-8;

    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }

    location ~ ^/(api) {
        proxy_pass   http://backend-up;
    }
}
```

将 API 的请求反向代理到后端服务上。其余请求全部留给前端进行处理。

### 后端

后端是使用 [Golang](http://golang.org/) 编写的 API 服务，对请求进行相应处理，并将数据存储至 [Redis](http://redis.io/) 当中。依赖管理使用的是 [dep](https://github.com/golang/dep)。由于 Golang 是编译型语言，编译完成后会生成一个二进制文件，为了让镜像尽可能小，所以 Dockerfile 和前端的差不多，都使用了[多阶段构建](https://docs.docker.com/develop/develop-images/multistage-build/)的特性。

```
FROM golang:1.11.1 as builder

WORKDIR /go/src/be
COPY . /go/src/be
RUN go get -u github.com/golang/dep/cmd/dep \
        && dep ensure \
        && go build

FROM debian:stretch-slim
COPY --from=builder /go/src/be/be /usr/bin/be
ENTRYPOINT ["/usr/bin/be"]
EXPOSE 8080
```

注意这里会暴露出来后端服务所监听的端口。

### Work

Work 端使用的是 [Python](http://python.org/)，用于计算已经存储至 Redis 当中的数据，并生成排行榜。依赖使用 [pip](https://github.com/pypa/pip) 进行安装。对于 Python 的镜像选择，我做了一组[性能对比的测试](http://moelove.info/docker-python-perf/) 有兴趣可以了解下。

```
FROM python:3.7-slim

WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt

ENTRYPOINT ["python", "work.py"]
```

### 构建发布

接下来，我们只要在对应项目目录中，执行 `docker build [OPTIONS] PATH` 即可。一般我们会使用 `-t name:tag` 的方式打 tag。

```
# 以下分别是在各模块自己的目录内
➜  be docker build -t  taobeier/saythx-be .
➜  fe docker build -t  taobeier/saythx-fe .
➜  work docker build -t  taobeier/saythx-work .
```

需要注意的是，前端项目由于目录内包含开发时的 `node_modules` 等文件，需要注意添加 `.dockerignore` 文件，忽略一些非预期的文件。关于 Docker 的 build 原理，有想深入理解的，可参考我之前写的 [Docker 深入篇之 Build 原理](http://moelove.info/2018/09/04/Docker-深入篇之-Build-原理/) 。 当镜像构建完成后，我们需要将它们发布至镜像仓库。这里我们直接使用官方的 [Docker Hub](http://hub.docker.com/)，执行 `docker login` 输入用户名密码验证成功后便可进行发布（需要先去 Docker Hub 注册帐号）。

登录成功后，默认情况下在 `$HOME/.docker/config.json` 中会存储用户相关凭证。

接下来进行发布只需要执行 `docker push` 即可。

```
➜  ~ docker push taobeier/saythx-be
➜  ~ docker push taobeier/saythx-fe
➜  ~ docker push taobeier/saythx-work
```

## 容器编排 Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) 是一种较为简单的可进行容器编排的技术，需要创建一个配置文件，通常情况下为 `docker-compose.yml` 。在 saythx 项目的根目录下我已经创建好了 `docker-compose.yml` 的配置文件。

```
version: '3'

services:
  saythx-frontend:
    build:
      context: fe/.
    image: taobeier/saythx-fe
    ports:
      - "8088:80"
    depends_on:
      - saythx-backend
    networks:
      - saythx

  saythx-backend:
    build:
      context: be/.
    image: taobeier/saythx-be
    depends_on:
      - saythx-redis
    networks:
      - saythx
    environment:
      - REDIS_HOST=saythx-redis
    
  saythx-work:
    build:
      context: work/.
    image: taobeier/saythx-work
    depends_on:
      - saythx-redis
    networks:
      - saythx
    environment:
      - REDIS_HOST=saythx-redis
      - REDIS_PORT=6379

  saythx-redis:
    image: "redis:5"
    networks:
      - saythx

networks:
  saythx:
```

在项目的根目录下执行 `docker-compose up` 即可启动该项目。在浏览器中访问 http://127.0.0.1:8088/ 即可看到项目的前端界面。如下图：



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="628"></svg>)



打开另外的终端，进入项目根目录内，执行 `docker-compose ps` 命令即可看到当前的服务情况。

```
➜  saythx git:(master) ✗ docker-compose ps

          Name                        Command               State          Ports
----------------------------------------------------------------------------------------
saythx_saythx-backend_1    /usr/bin/be                      Up      8080/tcp
saythx_saythx-frontend_1   nginx -g daemon off;             Up      0.0.0.0:8088->80/tcp
saythx_saythx-redis_1      docker-entrypoint.sh redis ...   Up      6379/tcp
saythx_saythx-work_1       python work.py                   Up
```

可以看到各组件均是 `Up` 的状态，相关端口也已经暴露出来。

可在浏览器直接访问体验。由于 `Docker Compose` 并非本册的重点，故不做太多介绍，可参考官方文档进行学习。接下来进入本节的重点内容，将项目部署至 K8S 中。

## 编写配置文件并部署

在 K8S 中进行部署或者说与 K8S 交互的方式主要有三种：

- 命令式
- 命令式对象配置
- 声明式对象配置

第 7 节介绍过的 `kubectl run redis --image='redis:alpine'` 这种方式便是命令式，这种方式很简单，但是可重用性低。毕竟你的命令执行完后，其他人也并不清楚到底发生了什么。

命令式对象配置，主要是编写配置文件，但是通过类似 `kubectl create` 之类命令式的方式进行操作。

再有一种便是声明式对象配置，主要也是通过编写配置文件，但是使用 `kubectl apply` 之类的放好似进行操作。与第二种命令式对象配置的区别主要在于对对象的操作将会得到保留。但同时这种方式有时候也并不好进行调试。

接下来，为 saythx 项目编写配置文件，让它可以部署至 K8S 中。当然，这里我们已经创建过了 `docker-compose.yml` 的配置文件，并且也验证了其可用性，可以直接使用 [Kompose](https://github.com/kubernetes/kompose) 工具将 `docker-compose.yml` 的配置文件进行转换。

但这里采用直接编写的方式。同时，我们部署至一个新的名为 `work` 的 `Namespace` 中。

### Namespace

```
apiVersion: v1
kind: Namespace
metadata:
  name: work
```

指定了 `Namespace` name 为 `work`。然后进行部署

```
➜  conf git:(master) ✗ kubectl apply -f namespace.yaml 
namespace/work created
```

### Redis 资源

从前面的 `docker-compose.yml` 中也能发现，saythx 中各个组件，只有 Redis 是无任何依赖的。我们先对它进行部署。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: saythx-redis
  namespace: work
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis:5
        name: redis
        ports:
        - containerPort: 6379
```

由于这是本册内第一次出现完整的 `Deployment` 配置文件，故而进行重点介绍。

- `apiVersion` ：指定了 API 的版本号，当前我们使用的 K8S 中， `Deployment` 的版本号为 `apps/v1`，而在 1.9 之前使用的版本则为 `apps/v1beta2`，在 1.8 之前的版本使用的版本为 `extensions/v1beta1`。在编写配置文件时需要格外注意。
- `kind` ：指定了资源的类型。这里指定为 `Deployment` 说明是一次部署。
- `metadata` ：指定了资源的元信息。例如其中的 `name` 和 `namespace` 分别表示资源名称和所归属的 `Namespace`。
- `spec` ：指定了对资源的配置信息。例如其中的 `replicas` 指定了副本数当前指定为 1 。`template.spec` 则指定了 `Pod` 中容器的配置信息，这里的 `Pod` 中只部署了一个容器。

配置文件已经生产，现在对它进行部署。

```
➜  conf git:(master) ✗ kubectl -n work get all
No resources found.
➜  conf git:(master) ✗ kubectl apply -f redis-deployment.yaml
deployment.apps/saythx-redis created
➜  conf git:(master) ✗ kubectl -n work get all
NAME                                READY     STATUS    RESTARTS   AGE
pod/saythx-redis-79d8f9864d-x8fp9   1/1       Running   0          4s

NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/saythx-redis   1         1         1            1           4s

NAME                                      DESIRED   CURRENT   READY     AGE
replicaset.apps/saythx-redis-79d8f9864d   1         1         1         4s
```

可以看到 `Pod` 已经在正常运行了。我们进入 `Pod` 内进行测试。

```
➜  conf git:(master) ✗ kubectl -n work exec -it saythx-redis-79d8f9864d-x8fp9 bash
root@saythx-redis-79d8f9864d-x8fp9:/data# redis-cli
127.0.0.1:6379> ping
PONG
```

响应正常。

### Redis service

由于 `Redis` 是后端服务的依赖，我们将它作为 `Service` 暴露出来。

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis
  name: saythx-redis
  namespace: work
spec:
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
  selector:
    app: redis
  type: NodePort
```

关于 `Service` 的内容，可参考第 7 节，我们详细做过解释。这里直接使用配置文件进行部署。

```
➜  conf git:(master) ✗ kubectl apply -f redis-service.yaml
service/saythx-redis created
➜  conf git:(master) ✗ kubectl get svc -n work
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
saythx-redis   NodePort   10.107.31.242   <none>        6379:31467/TCP   1m
```

### 后端服务

接下来，我们对后端服务进行部署。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: backend
  name: saythx-backend
  namespace: work
spec:
  selector:
    matchLabels:
      app: backend
  replicas: 1
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - env:
        - name: REDIS_HOST
          value: saythx-redis
        image: taobeier/saythx-be
        name: backend
        ports:
        - containerPort: 8080
```

可以看到这里通过环境变量的方式，将 `REDIS_HOST` 传递给了后端服务。

```
➜  conf git:(master) ✗ kubectl apply -f backend-deployment.yaml
deployment.apps/saythx-backend created
➜  conf git:(master) ✗ kubectl -n work get all
NAME                                 READY     STATUS              RESTARTS   AGE
pod/saythx-backend-c5f9f6d95-lmtxn   0/1       ContainerCreating   0          5s
pod/saythx-redis-8558c7d7d-kcmzk     1/1       Running             0          17m

NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/saythx-redis   NodePort   10.107.31.242   <none>        6379:31467/TCP   1m

NAME                             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/saythx-backend   1         1         1            0           5s
deployment.apps/saythx-redis     1         1         1            1           17m

NAME                                       DESIRED   CURRENT   READY     AGE
replicaset.apps/saythx-backend-c5f9f6d95   1         1         0         5s
replicaset.apps/saythx-redis-8558c7d7d     1         1         1         17m
```

### 后端 Service

后端服务是前端项目的依赖，故而我们也将其作为 `Service` 暴露出来。

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: backend
  name: saythx-backend
  namespace: work
spec:
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  selector:
    app: backend
  type: NodePort
```

通过配置文件进行部署。

```
➜  conf git:(master) ✗ kubectl apply -f backend-service.yaml
service/saythx-backend created
➜  conf git:(master) ✗ kubectl get svc -n work
NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/saythx-backend   NodePort   10.104.0.47     <none>        8080:32051/TCP   8s
service/saythx-redis     NodePort   10.107.31.242   <none>        6379:31467/TCP   3m
```

我们同样使用 `NodePort` 将其暴露出来，并在本地进行测试。

```
➜  conf git:(master) ✗ curl http://127.0.0.1:32051/api/v1/list
{"HonorDatas":null}
```

服务可正常响应。

### 前端

接下来我们编写前端的配置文件。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: frontend
  name: saythx-frontend
  namespace: work
spec:
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - image: taobeier/saythx-fe
        name: frontend
        ports:
        - containerPort: 80
```

需要注意的是，我们必须在后端 `Service` 暴露出来后才能进行前端的部署，因为前端镜像中 Nginx 的反向代理配置中会去检查后端是否可达。使用配置文件进行部署。

```
➜  conf git:(master) ✗ kubectl apply -f frontend-deployment.yaml
deployment.apps/saythx-frontend created
➜  conf git:(master) ✗ kubectl -n work get all
NAME                                   READY     STATUS    RESTARTS   AGE
pod/saythx-backend-c5f9f6d95-lmtxn     1/1       Running   0          16m
pod/saythx-frontend-678d544b86-wp9gr   1/1       Running   0          30s
pod/saythx-redis-8558c7d7d-kcmzk       1/1       Running   0          34m

NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/saythx-backend   NodePort   10.104.0.47     <none>        8080:32051/TCP   15m
service/saythx-redis     NodePort   10.107.31.242   <none>        6379:31467/TCP   18m

NAME                              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/saythx-backend    1         1         1            1           16m
deployment.apps/saythx-frontend   1         1         1            1           30s
deployment.apps/saythx-redis      1         1         1            1           34m

NAME                                         DESIRED   CURRENT   READY     AGE
replicaset.apps/saythx-backend-c5f9f6d95     1         1         1         16m
replicaset.apps/saythx-frontend-678d544b86   1         1         1         30s
replicaset.apps/saythx-redis-8558c7d7d       1         1         1         34m
```

### 前端 Service

接下来，我们需要让前端可以被直接访问到，同样的需要将它以 `Service` 的形式暴露出来。

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: frontend
  name: saythx-frontend
  namespace: work
spec:
  ports:
  - name: "80"
    port: 80
    targetPort: 80
  selector:
    app: frontend
  type: NodePort
```

创建 `Service` 。

```
➜  conf git:(master) ✗ kubectl apply -f frontend-service.yaml
service/saythx-frontend created
➜  conf git:(master) ✗ kubectl -n work get svc
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
saythx-backend    NodePort   10.104.0.47     <none>        8080:32051/TCP   17m
saythx-frontend   NodePort   10.96.221.71    <none>        80:32682/TCP     11s
saythx-redis      NodePort   10.107.31.242   <none>        6379:31467/TCP   20m
```

我们可以直接通过 Node 的 32682 端口进行访问。

### Work

最后，是我们的 Work 组件，为它编写配置文件。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: work
  name: saythx-work
  namespace: work
spec:
  selector:
    matchLabels:
      app: work
  replicas: 1
  template:
    metadata:
      labels:
        app: work
    spec:
      containers:
      - env:
        - name: REDIS_HOST
          value: saythx-redis
        - name: REDIS_PORT
          value: "6379"
        image: taobeier/saythx-work
        name: work
```

同样的，我们通过环境变量的方式传递了 Redis 相关的配置进去。

```
➜  conf git:(master) ✗ kubectl apply -f work-deployment.yaml
deployment.apps/saythx-work created
➜  conf git:(master) ✗ kubectl -n work get all
NAME                                   READY     STATUS              RESTARTS   AGE
pod/saythx-backend-c5f9f6d95-lmtxn     1/1       Running             0          22m
pod/saythx-frontend-678d544b86-wp9gr   1/1       Running             0          5m
pod/saythx-redis-8558c7d7d-kcmzk       1/1       Running             0          39m
pod/saythx-work-6b9958dc47-hh9td       0/1       ContainerCreating   0          7s

NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/saythx-backend    NodePort   10.104.0.47     <none>        8080:32051/TCP   20m
service/saythx-frontend   NodePort   10.96.221.71    <none>        80:32682/TCP     3m
service/saythx-redis      NodePort   10.107.31.242   <none>        6379:31467/TCP   23m

NAME                              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/saythx-backend    1         1         1            1           22m
deployment.apps/saythx-frontend   1         1         1            1           5m
deployment.apps/saythx-redis      1         1         1            1           39m
deployment.apps/saythx-work       1         1         1            0           7s

NAME                                         DESIRED   CURRENT   READY     AGE
replicaset.apps/saythx-backend-c5f9f6d95     1         1         1         22m
replicaset.apps/saythx-frontend-678d544b86   1         1         1         5m
replicaset.apps/saythx-redis-8558c7d7d       1         1         1         39m
replicaset.apps/saythx-work-6b9958dc47       1         1         0         7s
```

现在均已经部署完成。并且可直接通过 Node 端口进行访问。

## 扩缩容

如果我们觉得排行榜生成效率较低，则可通过扩容 Work 来得到解决。具体做法是可修改 work 的 `Deployment` 配置文件，将 `spec.replicas` 设置为预期的数值，之后执行 `kubectl -f work-deployment.yaml` 即可。

或者可直接通过命令行进行操作

```
➜  conf git:(master) ✗ kubectl -n work scale --replicas=2  deployment/saythx-work
```

上面的命令是将 `saythx-work` 的部署副本数设置为 2 。缩容也差不多是类似的操作。

## 总结

通过本节的学习，我们已经学习到了如何将项目实际部署至 K8S 中，可以手动编写配置也可以利用一些工具进行辅助。同时也了解到了如何应对应用的扩缩容。

但如果应用需要进行升级的话，则需要去更改配置文件中相关的配置，这个过程会较为繁琐，并且整体项目线上的版本管理也是个问题：比如组件的个人升级，回滚之间如果有依赖的话，会比较麻烦。我们在接下来的两节来学习如何解决这个问题。

# 应用管理: 初识 Helm

## 整体概览

上节，我们已经学习了如何通过编写配置文件的方式部署项目。而在实际生产环境中，项目所包含组件可能不止 3 个，并且可能项目数会很多，如果每个项目的发布，更新等都通过手动去编写配置文件的方式，实在不利于管理。

并且，当线上出现个别组件升级回滚之类的操作，如果组件之间有相关版本依赖等情况，那事情会变得复杂的多。我们需要有更简单的机制来辅助我们完成这些事情。

## Helm 介绍

[Helm](https://www.helm.sh/) 是构建于 K8S 之上的包管理器，可与我们平时接触到的 `Yum`，`APT`，`Homebrew` 或者 `Pip` 等包管理器相类比。

使用 Helm 可简化包分发，安装，版本管理等操作流程。同时它也是 CNCF 孵化项目。

## Helm 安装

Helm 是 C/S 架构，主要分为客户端 `helm` 和服务端 `Tiller`。安装时可直接在 [Helm 仓库的 Release 页面](https://github.com/helm/helm/releases) 下载所需二进制文件或者源码包。

由于当前项目的二进制文件存储已切换为 GCS，我已经为国内用户准备了最新版本的二进制包，可通过以下链接进行下载。

```
链接: https://pan.baidu.com/s/1n1zj3rlv2NyfiA6kRGrHfg 提取码: 5huw 
```

下载后对文件进行解压，我这里以 Linux amd64 为例。

```
➜  /tmp tar -zxvf helm-v2.11.0-linux-amd64.tar.gz
linux-amd64/
linux-amd64/tiller
linux-amd64/README.md
linux-amd64/helm
linux-amd64/LICENSE
➜  /tmp tree linux-amd64 
linux-amd64
├── helm
├── LICENSE
├── README.md
└── tiller

0 directories, 4 files
```

解压完成后，可看到其中包含 `helm` 和 `tiller` 二进制文件。

### 客户端 helm

`helm` 是个二进制文件，直接将它移动至 `/usr/bin` 目录下即可。

```
➜  /tmp sudo mv linux-amd64/helm /usr/bin/helm 
```

这时候便可直接通过 `helm` 命令使用了。比如，我们验证当前使用的版本。

```
➜  /tmp helm version
Client: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
Error: Get http://localhost:8080/api/v1/namespaces/kube-system/pods?labelSelector=app%3Dhelm%2Cname%3Dtiller: dial tcp 127.0.0.1:8080: connect: connection refused
```

可以看到上面有明显的报错，并且很像 `kubectl` 未正确配置时的错误。这是因为 `helm` 默认会去读取 `$HOME/.kube/config` 的配置文件，用于正确的连接至目标集群。

当我们正确的配置好 `$HOME/.kube/config` 文件时，再次执行：

```
➜  /tmp helm version
Client: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
Error: could not find tiller
```

这次报错是因为找不到服务端 `Tiller`，接下来我们部署服务端。

### 服务端 Tiller

以下讨论中，前提都是 `$HOME/.kube/config` 已正确配置，并且 `kebectl` 有操作集群的权限。

#### 本地安装

刚才我们解压的文件中，还有一个二进制文件 `tiller` 。我们可以直接执行它，用于在本地启动服务。

```
➜  /tmp ./linux-amd64/tiller
[main] 2018/11/18 23:47:10 Starting Tiller v2.11.0 (tls=false)
[main] 2018/11/18 23:47:10 GRPC listening on :44134
[main] 2018/11/18 23:47:10 Probes listening on :44135
[main] 2018/11/18 23:47:10 Storage driver is ConfigMap
[main] 2018/11/18 23:47:10 Max history per release is 0
```

直接执行时，默认会监听 `44134` 和 `44135` 端口，`44134` 端口用于和 `helm` 进行通信，而 `44135` 主要是用于做探活的，在部署至 K8S 时使用。

当我们使用客户端连接时，只需设置 `HELM_HOST` 环境变量即可。

```
➜  ~ export HELM_HOST=localhost:44134
➜  ~ helm version
Client: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
```

**注意** 一定要正确配置 `$HOME/.kube/config` 文件，否则会影响正常功能使用。

#### 默认安装

官方提供了一种一键式安装的方式。那便是 `helm init` 执行这条命令后，会同时在 K8S 中部署服务端 Tiller 和初始化 helm 的默认目录 `$HELM_HOME` 默认值为 `$HOME/.helm`。

这种方式下会默认使用官方镜像 `gcr.io/kubernetes-helm/tiller` 网络原因可能会导致安装失败。所以我已将官方镜像进行同步。可使用以下方式进行使用：

```
➜  ~ helm init --tiller-image taobeier/tiller:v2.11.0 
Creating /root/.helm
Creating /root/.helm/repository
Creating /root/.helm/repository/cache
Creating /root/.helm/repository/local
Creating /root/.helm/plugins
Creating /root/.helm/starters
Creating /root/.helm/cache/archive
Creating /root/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
➜  ~ helm version
Client: &version.Version{SemVer:"v2.11.0", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f93090", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
```

可以看到 `$HELM_HOME` 目录已经初始化完成，客户端与服务端已可以正常通信。查看下当前 K8S 集群中的情况：

```
➜  ~ kubectl -n kube-system get deploy tiller-deploy
NAME            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
tiller-deploy   1         1         1            1           6m
```

可以看到已正常部署。

#### 手动安装

通过上面的描述，可能你已经发现，安装服务端，其实也就是一次普通的部署，我们可以通过以下方式来自行通过 `kubectl` 完成部署。

```
➜  ~ helm init --dry-run --debug  # 篇幅原因，以下内容进行了省略
---                               
apiVersion: extensions/v1beta1
kind: Deployment                                    
metadata:                            
  creationTimestamp: null
  labels:         
    app: helm              
    name: tiller       
  name: tiller-deploy           
  namespace: kube-system   
spec:               
  replicas: 1 
  strategy: {}                
  ...
status: {}

---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: helm
    name: tiller
  name: tiller-deploy
  namespace: kube-system
spec:
  ports:
  - name: tiller
    port: 44134
    targetPort: tiller
  selector:
    app: helm
    name: tiller
  type: ClusterIP
status:
  loadBalancer: {}
```

将输出内容保存至文件中，自行修改后，通过 `kubectl` 进行部署即可。建议在修改过程中，尽量不要去更改标签及选择器。

#### RBAC 使用

上面的内容中，均未提及到权限控制相关的内容，但是在生产环境中使用，我们一般都是会进行权限控制的。

在第 8 节中，我们已经详细的解释了认证授权相关的内容。所以下面的内容不做太多详细解释。

这里我们创建一个 `ServiceAccount` 命名为 `tiller`，为了简单，我们直接将它与 `cluster-admin` 进行绑定。

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

将此内容保存为 `tiller-rbac.yaml`，开始进行部署操作。

```
➜  ~ kubectl apply -f tiller-rbac.yaml
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created
➜  ~ helm init --service-account tiller
Creating /root/.helm
Creating /root/.helm/repository
Creating /root/.helm/repository/cache
Creating /root/.helm/repository/local
Creating /root/.helm/plugins
Creating /root/.helm/starters
Creating /root/.helm/cache/archive
Creating /root/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
➜  ~ helm version
Client: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
```

以此方式完成部署。

## Helm 概念

### Chart

`chart` 就是 Helm 所管理的包，类似于 `Yum` 所管理的 `rpm` 包或是 `Homebrew` 管理的 `Formulae`。它包含着一个应用要部署至 K8S 上所必须的所有资源。

### Release

`Release` 就是 `chart` 在 K8S 上部署后的实例。`chart` 的每次部署都将产生一次 `Release`。这和上面类比的包管理器就有所不同了，多数的系统级包管理器所安装的包只会在系统中存在一份。我们可以以 `Pip` 在虚拟环境下的包安装，或者 `Npm` 的 local install 来进行类比。

### Repository

`Repository` 就是字面意思，存储 `chart` 的仓库。还记得我们上面执行 `helm init` 时的输出吗？默认情况下，初始化 Helm 的时候，会添加两个仓库，一个是 `stable` 仓库 [kubernetes-charts.storage.googleapis.com](https://kubernetes-charts.storage.googleapis.com/) 另一个则是 `local` 仓库，地址是 http://127.0.0.1:8879/charts 。

### Config

前面提到了 `chart` 是应用程序所必须的资源，当然我们实际部署的时候，可能就需要有些自定义的配置了。`Config` 便是用于完成此项功能的，在部署时候，会将 `config` 与 `chart` 进行合并，共同构成我们将部署的应用。

## Helm 的工作原理

`helm` 通过 `gRPC` 将 `chart` 发送至 `Tiller` ，`Tiller` 则通过内置的 `kubernetes` 客户端库与 K8S 的 API server 进行交流，将 `chart` 进行部署，并生成 `Release` 用于管理。

前面只说到了 `helm` 与 `Tiller` 交互的协议，但尚未说其数据链路。

我们来看看 `Tiller` 的部署情况。主要看 `Service`：

```
➜  ~ kubectl -n kube-system get svc
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kube-dns        ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP   1h
tiller-deploy   ClusterIP   10.107.204.164   <none>        44134/TCP       33m
```

`Tiller` 默认采用 `ClusterIP` 类型的 `Service` 进行部署。而我们知道的 `ClusterIP` 类型的 `Service` 是仅限集群内访问的。

在这里所依赖的技术，便是在第 5 节，我们提到的 `socat` 。`helm` 通过 `socat` 的端口转发（或者说 K8S 的代理），进而实现了本地与 `Tiller` 的通信。

当然，以上内容均以当前最新版本 `2.11.0` 为例。当下一个大版本 Helm v3 出现时， `Tiller` 将不复存在，通信机制和工作原理也将发生变化。

## 总结

通过本节，我们已经学习到了 Helm 的基础知识和工作原理，了解到了 Helm 的用途以及如何在本地和 K8S 中部署它。需要注意的是 `$HOME/.kube/config` 需要提前配置好，以及 `socat` 工具需要提前安装，可参考第 5 节的内容。

接下来，我们将上节中的示例项目使用 Helm 部署至 K8S 集群中。

# 部署实践: 以 Helm 部署项目

## 概览

上节，我们学习到了 Helm 的基础概念和工作原理，本节我们将 Helm 用于我们的实际项目，编写 Helm `chart` 以及通过 Helm 进行部署。

## Helm chart

上节我们解释过 `chart` 的含义，现在我们要将项目使用 Helm 部署，那么首先，我们需要创建一个 `chart`。

### Chart 结构

在我们项目的根目录下，通过以下命令创建一个 `chart`。

```
➜  saythx git:(master) helm create saythx
Creating saythx
➜  saythx git:(master) ✗ tree -a saythx
saythx
├── charts
├── Chart.yaml
├── .helmignore
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml

2 directories, 8 files
```

创建完成后，我们可以看到默认创建的 `chart` 中包含了几个文件和目录。我们先对其进行解释。

#### Chart.yaml

```
➜  saythx git:(master) ✗ cat saythx/Chart.yaml
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for Kubernetes
name: saythx
version: 0.1.0
```

这个文件是每个 `chart` 必不可少的一个文件，其中包含着几个重要的属性，如：

- `apiVersion`：目前版本都为 `v1`
- `appVersion`：这是应用的版本号，需要与 `apiVersion`， `version` 等字段注意区分
- `name`: 通常要求 `chart` 的名字必须和它所在目录保持一致，且此字段必须
- `version`：表明当前 `chart` 的版本号，会直接影响 `Release` 的记录，且此字段必须
- `description`：描述

#### charts

`charts` 文件夹是用于存放依赖的 `chart` 的。当有依赖需要管理时，可添加 `requirements.yaml` 文件，可用于管理项目内或者外部的依赖。

#### .helmignore

`.helmignore` 类似于 `.gitignore` 和 `.dockerignore` 之类的，用于忽略掉一些不想包含在 `chart` 内的文件。

#### templates

`templates` 文件夹内存放着 `chart` 所使用的模板文件，也是 `chart` 的实际执行内容。在使用 `chart` 进行安装的时候，会将 下面介绍的 `values.yaml` 中的配置项与 `templates` 中的模板进行组装，生成最终要执行的配置文件。

`templates` 中，推荐命名应该清晰，如 `xx-deployment.yaml`，中间使用 `-` 进行分割，避免使用驼峰式命名。

`Notes.txt` 文件在 `helm install` 完成后，会进行回显，可用于解释说明如何访问服务等。

#### values.yaml

`values.yaml` 存放着项目的一些可配置项，如镜像的名称或者 tag 之类的。作用就是用于和模板进行组装。

### 编写 chart

了解完结构之后，我们来实际编写我们的 chart 。所有完整的代码可在 [SayThx 项目](https://github.com/tao12345666333/saythx) 获取。

```
# Chart.yaml
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for SayThx.
name: saythx
version: 0.1.0
maintainers:
  - name: Jintao Zhang
```

可添加 `maintainers` 字段，表示维护者。

```
# values.yaml

# backend is the values for backend
backend:
  image: taobeier/saythx-be
  tag: "1.0"
  pullPolicy: IfNotPresent
  replicas: 1

# namespace is the values for deploy namespace
namespace: work

# service.type is the values for service type
service:
  type: NodePort
```

`values.yaml` 文件中定义了我们预期哪些东西是可配置的，比如 `namespace` 以及镜像名称 tag 等。这里只是贴出了部分内容，仅做说明使用，完整内容可查看我们的[示例项目](https://github.com/tao12345666333/saythx) 。

写 `values.yaml` 文件的时候，由于是使用 `YAML` 格式的配置，所以它非常的灵活，即可以使用如上面例子中的 `backend` 那种字典类型的， 也可以写成简单的 k-v 形式。但通常来讲，应该尽可能的将它写的清晰明确。并且容易被替换。

```
# templates/backend-service.yaml 

apiVersion: v1
kind: Service
metadata:
  labels:
    app: backend
  name: saythx-backend
  namespace: {{ .Values.namespace }}
spec:
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  selector:
    app: backend
  type: {{ .Values.service.type }}
```

将我们之前写的部署文件模板化，与配置项进行组装。

```
1. Get the application URL by running these commands:
{{- if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get --namespace {{ .Values.namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services saythx-frontend)
  export NODE_IP=$(kubectl get nodes --namespace {{ .Values.namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
{{- else if contains "ClusterIP" .Values.service.type }}
  export POD_NAME=$(kubectl get pods --namespace {{ .Values.namespace }} -l "app=frontend" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace {{ .Values.namespace }} port-forward $POD_NAME 8080:80
{{- end }}
```

上面这是 `NOTES.txt` 文件内的内容。 这些内容会在 `helm install` 执行成功后显示在终端，用于说明服务如何访问或者其他注意事项等。

当然，这里的内容主要是为了说明如何编写 `chart` ，在实践中，尽量避免硬编码配置在里面。

## 部署

### 直接部署

Helm 的 `chart` 可以直接在源码目录下通过 `helm install` 完成部署。例如：

```
➜  saythx helm install saythx
NAME:   handy-seastar
LAST DEPLOYED: Tue Nov 20 23:33:42 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Namespace
NAME  STATUS  AGE
work  Active  1s

==> v1/Service
NAME             TYPE      CLUSTER-IP      EXTERNAL-IP  PORT(S)         AGE
saythx-backend   NodePort  10.102.206.213  <none>       8080:30663/TCP  0s
saythx-frontend  NodePort  10.96.109.45    <none>       80:30300/TCP    0s
saythx-redis     NodePort  10.97.174.8     <none>       6379:30589/TCP  0s

==> v1/Deployment
NAME             DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
saythx-backend   1        1        1           0          0s
saythx-frontend  1        1        1           0          0s
saythx-redis     1        1        1           0          0s
saythx-work      1        1        1           0          0s

==> v1/Pod(related)
NAME                              READY  STATUS             RESTARTS  AGE
saythx-backend-7f6d86d9c8-xqttg   0/1    ContainerCreating  0         0s
saythx-frontend-777fc64997-9zmq6  0/1    Pending            0         0s
saythx-redis-8558c7d7d-lh5df      0/1    ContainerCreating  0         0s
saythx-work-9b4446d84-c2pr4       0/1    ContainerCreating  0         0s


NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace work -o jsonpath="{.spec.ports[0].nodePort}" services saythx-frontend)
  export NODE_IP=$(kubectl get nodes --namespace work -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

### 打包

当然，我们也可以将 `chart` 打包，以便于分发。

```
➜  saythx helm package saythx 
Successfully packaged chart and saved it to: /root/saythx/saythx-0.1.0.tgz
```

可以看到打包时是按照 `chart` 的名字加版本号进行命名的。

至于部署，和前面没什么太大区别， `helm install saythx-0.1.0.tgz` 即可。

### 访问服务

前面在部署完成后，有一些返回信息，我们来按照其内容访问我们的服务：

```
➜  saythx export NODE_PORT=$(kubectl get --namespace work -o jsonpath="{.spec.ports[0].nodePort}" services saythx-frontend)
➜  saythx export NODE_IP=$(kubectl get nodes --namespace work -o jsonpath="{.items[0].status.addresses[0].address}")
➜  saythx echo http://$NODE_IP:$NODE_PORT
http://172.17.0.5:30300
➜  saythx curl http://172.17.0.5:30300
<!DOCTYPE html><html lang=en><head><meta charset=utf-8><meta http-equiv=X-UA-Compatible content="IE=edge"><meta name=viewport content="width=device-width,initial-scale=1"><link rel=icon href=/favicon.ico><title>fe</title><link href=/css/app.0a6f0b04.css rel=preload as=style><link href=/css/chunk-vendors.ea3fa8e3.css rel=preload as=style><link href=/js/app.ee469174.js rel=preload as=script><link href=/js/chunk-vendors.14b9b088.js rel=preload as=script><link href=/css/chunk-vendors.ea3fa8e3.css rel=stylesheet><link href=/css/app.0a6f0b04.css rel=stylesheet></head><body><noscript><strong>We're sorry but fe doesn't work properly without JavaScript enabled. Please enable it to continue.</strong></noscript><div id=app></div><script src=/js/chunk-vendors.14b9b088.js></script><script src=/js/app.ee469174.js></script></body></html>
```

服务可以正常访问。

## 总结

通过本节我们学习到了 `chart` 的实际结构，及编写方式。以及编写了我们自己的 `chart` 并使用该 `chart` 部署了服务。

示例项目还仅仅是个小项目，试想当我们需要部署一个大型项目，如果不通过类似 Helm 这样的软件进行管理，每次的更新发布，维护 `YAML` 的配置文件就会很繁琐了。

另外，Helm 的功能还不仅限于此，使用 Helm 我们还可以管理 `Release` ，并进行更新回滚等操作。以及，我们可以搭建自己的私有 `chart` 仓库等。

下节开始，我们将进入深入学习阶段，逐个讲解 K8S 的核心组件，以便后续遇到问题时，可快速定位和解决。

# 庖丁解牛：kube-apiserver

## 整体概览

```
+----------------------------------------------------------+          
| Master                                                   |          
|              +-------------------------+                 |          
|     +------->|        API Server       |<--------+       |          
|     |        |                         |         |       |          
|     v        +-------------------------+         v       |          
|   +----------------+     ^      +--------------------+   |          
|   |                |     |      |                    |   |          
|   |   Scheduler    |     |      | Controller Manager |   |          
|   |                |     |      |                    |   |          
|   +----------------+     v      +--------------------+   |          
| +------------------------------------------------------+ |          
| |                                                      | |          
| |                Cluster state store                   | |          
| |                                                      | |          
| +------------------------------------------------------+ |          
+----------------------------------------------------------+          
```

在第 3 节《宏观认识：整体架构》 中，我们初次认识到了 `kube-apiserver` 的存在（以下内容中将统一称之为 `kube-apiserver`），知道了它作为集群的统一入口，接收来自外部的信号和请求，并将一些信息存储至 `etcd` 中。

但这只是一种很模糊的说法，本节我们来具体看看 `kube-apiserver` 的关键功能以及它的工作原理。

注意：本节所有的源码均以 `v1.11.3` 为准 commit id `a4529464e4629c21224b3d52edfe0ea91b072862`。

## REST API Server

先来说下 `kube-apiserver` 作为整个集群的入口，接受外部的信号和请求所应该具备的基本功能。

首先，它对外提供接口，可处理来自客户端（无论我们在用的 `kubeclt` 或者 `curl` 或者其他语言实现的客户端）的请求，并作出响应。

在第 5 节搭建集群时，我们提到要先去检查 `6443` 端口是否被占用。这样检查的原因在于 `kube-apiserver` 有个 `--secure-port` 的参数，通过这个参数来配置它将要监听在哪个端口，默认情况下是 `6443`。

当然，它还有另一个参数 `--insecure-port` ，这个参数可将 `kube-apiserver` 绑定到其指定的端口上，且通过该端口访问时无需认证。

在生产环境中，建议将其设置为 `0` 以禁用该功能。另外，这个参数也已经被标记为废弃，将在之后版本中移除。如果未禁用该功能，建议通过防火墙策略禁止从外部访问该端口。该端口会绑定在 `--insecure-bind-address` 参数所设置的地址上，默认为 `127.0.0.1`。

那么 `secure` 和 `insecure` 最主要的区别是什么呢？ 这就引出来了 `kube-apiserver` 作为 API Server 的一个最主要功能：认证。

### 认证（Authentication）

在第 8 节《认证和授权》中，我们已经讲过认证相关的机制。这里，我们以最简单的获取集群版本号为例。

通常，我们使用 `kubeclt version` 来获取集群和当前客户端的版本号。

```
master $ kubectl version
Client Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T18:02:47Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T17:53:03Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
```

获取集群版本号的时候，其实也是向 `kube-apiserver` 发送了一个请求进行查询的，我们可以通过传递 `-v` 参数来改变 log level 。

```
master $ kubectl version -v 8
I1202 03:15:06.360838   13581 loader.go:359] Config loaded from file /root/.kube/config
I1202 03:15:06.362106   13581 round_trippers.go:383] GET https://172.17.0.99:6443/version?timeout=32s
I1202 03:15:06.362130   13581 round_trippers.go:390] Request Headers:
I1202 03:15:06.362139   13581 round_trippers.go:393]     Accept: application/json, */*
I1202 03:15:06.362146   13581 round_trippers.go:393]     User-Agent: kubectl/v1.11.3 (linux/amd64) kubernetes/a452946
I1202 03:15:06.377653   13581 round_trippers.go:408] Response Status: 200 OK in 15 milliseconds
I1202 03:15:06.377678   13581 round_trippers.go:411] Response Headers:
I1202 03:15:06.377686   13581 round_trippers.go:414]     Content-Type: application/json
I1202 03:15:06.377693   13581 round_trippers.go:414]     Content-Length: 263
I1202 03:15:06.377699   13581 round_trippers.go:414]     Date: Sun, 02 Dec 2018 03:15:06 GMT
I1202 03:15:06.379314   13581 request.go:897] Response Body: {
  "major": "1",
  "minor": "11",
  "gitVersion": "v1.11.3",
  "gitCommit": "a4529464e4629c21224b3d52edfe0ea91b072862",
  "gitTreeState": "clean",
  "buildDate": "2018-09-09T17:53:03Z",
  "goVersion": "go1.10.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
Client Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T18:02:47Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T17:53:03Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
```

通过日志就可以很明显看到，首先会加载 `$HOME/.kube/config` 下的配置，获的集群地址，进而请求 `/version` 接口，最后格式化输出。

我们使用 `curl` 去请求同样的接口：

```
master $ curl -k https://172.17.0.99:6443/version
{
  "major": "1",
  "minor": "11",
  "gitVersion": "v1.11.3",
  "gitCommit": "a4529464e4629c21224b3d52edfe0ea91b072862",
  "gitTreeState": "clean",
  "buildDate": "2018-09-09T17:53:03Z",
  "goVersion": "go1.10.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

得到了相同的结果。你可能会有些奇怪，使用 `curl -k` 相当于忽略了认证的过程，为何还能拿到正确的信息。别急，我们来看下一个例子：

```
master $ kubectl get ns  -v 8
I1202 03:25:40.607886   16620 loader.go:359] Config loaded from file /root/.kube/config
I1202 03:25:40.608862   16620 loader.go:359] Config loaded from file /root/.kube/config
I1202 03:25:40.611187   16620 loader.go:359] Config loaded from file /root/.kube/config
I1202 03:25:40.622737   16620 loader.go:359] Config loaded from file /root/.kube/config
I1202 03:25:40.623495   16620 round_trippers.go:383] GET https://172.17.0.99:6443/api/v1/namespaces?limit=500
I1202 03:25:40.623650   16620 round_trippers.go:390] Request Headers:
I1202 03:25:40.623730   16620 round_trippers.go:393]     Accept: application/json;as=Table;v=v1beta1;g=meta.k8s.io, application/json
I1202 03:25:40.623820   16620 round_trippers.go:393]     User-Agent: kubectl/v1.11.3 (linux/amd64) kubernetes/a452946
I1202 03:25:40.644280   16620 round_trippers.go:408] Response Status: 200 OK in 20 milliseconds
I1202 03:25:40.644308   16620 round_trippers.go:411] Response Headers:
I1202 03:25:40.644327   16620 round_trippers.go:414]     Content-Type: application/json
I1202 03:25:40.644334   16620 round_trippers.go:414]     Content-Length: 2061
I1202 03:25:40.644338   16620 round_trippers.go:414]     Date: Sun, 02 Dec 2018 03:25:40 GMT
I1202 03:25:40.644398   16620 request.go:897] Response Body: {"kind":"Table","apiVersion":"meta.k8s.io/v1beta1","metadata":{"selfLink":"/api/v1/namespaces","resourceVersion":"3970"},"columnDefinitions":[{"name":"Name","type":"string","format":"name","description":"Name must be unique within anamespace. Is required when creating resources, although some resources may allow a client to request the generation of an appropriate name automatically. Name is primarily intended for creation idempotence and configuration definition. Cannot be updated. More info: http://kubernetes.io/docs/user-guide/identifiers#names","priority":0},{"name":"Status","type":"string","format":"","description":"The status of the namespace","priority":0},{"name":"Age","type":"string","format":"","description":"CreationTimestamp is a timestamp representing the server time when this object was created. It is not guaranteed to be set in happens-before order across separate operations. Clients may not set this value. It is represented in RFC3339 form and is in UTC.\n\nPopulated by the system. Read-only. [truncated 1037 chars]
I1202 03:25:40.645111   16620 get.go:443] no kind is registered for the type v1beta1.Table
NAME          STATUS    AGE
default       Active    45m
kube-public   Active    45m
kube-system   Active    45m
```

使用 `curl` 去请求：

```
master $ curl -k  https://172.17.0.99:6443/api/v1/namespaces
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "namespaces is forbidden: User \"system:anonymous\" cannot list namespaces at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "namespaces"
  },
  "code": 403
}
```

看到这里，应该就很明显了，当前忽略掉认证过程的 `curl` 被判定为 `system:anonymous` 用户，而此用户不具备列出 `namespace` 的权限。

那我们是否有其他办法使用 `curl` 获取资源呢？ 当然有，使用 `kubectl proxy` 可以在本地和集群之间创建一个代理，就像这样：

```
master $ kubectl proxy &
[1] 22205
master $ Starting to serve on 127.0.0.1:8001

master $ curl http://127.0.0.1:8001/api/v1/namespaces
{
  "kind": "NamespaceList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces",
    "resourceVersion": "5363"
  },
  "items": [
    {
      "metadata": {
        "name": "default",
        "selfLink": "/api/v1/namespaces/default",
        "uid": "a5124131-f5db-11e8-9237-0242ac110063",
        "resourceVersion": "4",
        "creationTimestamp": "2018-12-02T02:40:35Z"
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    },
    {
      "metadata": {
        "name": "kube-public",
        "selfLink": "/api/v1/namespaces/kube-public",
        "uid": "a5153f73-f5db-11e8-9237-0242ac110063",
        "resourceVersion": "10",
        "creationTimestamp": "2018-12-02T02:40:35Z"
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    },
    {
      "metadata": {
        "name": "kube-system",
        "selfLink": "/api/v1/namespaces/kube-system",
        "uid": "a514ad25-f5db-11e8-9237-0242ac110063",
        "resourceVersion": "9",
        "creationTimestamp": "2018-12-02T02:40:35Z"
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    }
  ]
}
```

可以看到已经能正确的获取资源了，这是因为 `kubectl proxy` 使用了 `$HOME/.kube/config` 中的配置。

在 `staging/src/k8s.io/client-go/tools/clientcmd/loader.go` 中，有一个名为 `LoadFromFile` 的函数用来提供加载配置文件的功能。

```
func LoadFromFile(filename string) (*clientcmdapi.Config, error) {
	kubeconfigBytes, err := ioutil.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	config, err := Load(kubeconfigBytes)
	if err != nil {
		return nil, err
	}
	glog.V(6).Infoln("Config loaded from file", filename)

	// set LocationOfOrigin on every Cluster, User, and Context
	for key, obj := range config.AuthInfos {
		obj.LocationOfOrigin = filename
		config.AuthInfos[key] = obj
	}
	for key, obj := range config.Clusters {
		obj.LocationOfOrigin = filename
		config.Clusters[key] = obj
	}
	for key, obj := range config.Contexts {
		obj.LocationOfOrigin = filename
		config.Contexts[key] = obj
	}

	if config.AuthInfos == nil {
		config.AuthInfos = map[string]*clientcmdapi.AuthInfo{}
	}
	if config.Clusters == nil {
		config.Clusters = map[string]*clientcmdapi.Cluster{}
	}
	if config.Contexts == nil {
		config.Contexts = map[string]*clientcmdapi.Context{}
	}

	return config, nil
}
```

逻辑其实很简单，读取指定的文件（一般在调用此函数前，都会先去检查是否有 `KUBECONFIG` 的环境变量或 `--kubeconfig`，如果没有才会使用默认的 `$HOME/.kube/config` 作为文件名）。

从以上的例子中，使用当前配置的用户可以获取资源，而 `system:anonymous` 不可以。可以得出 `kube-apiserver` 又一个重要的功能：授权。

### 授权（Authorization）

在第 8 节中，我们也已经讲过，K8S 支持多种授权机制，现在多数都在使用 `RBAC` ，我们之前使用 `kubeadm` 创建集群时，默认会开启 `RBAC`。如何创建权限可控的用户在第 8 节也已经说过。所以本节中不过多赘述了，直接看授权后的处理逻辑。

### 准入控制（Admission Control）

在请求进来时，会先经过认证、授权接下来会进入准入控制环节。准入控制和前两项内容不同，它不只是关注用户和行为，它还会处理请求的内容。不过它对读操作无效。

准入控制与我们前面说提到的认证、授权插件类似，支持同时开启多个。在 `v1.11.3` 中，默认开启的准入控制插件有：

```
NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeClaimResize,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,Priority
```

相关的代码可查看 `pkg/kubeapiserver/options/plugins.go`

```
func DefaultOffAdmissionPlugins() sets.String {
	defaultOnPlugins := sets.NewString(
		lifecycle.PluginName,                //NamespaceLifecycle
		limitranger.PluginName,              //LimitRanger
		serviceaccount.PluginName,           //ServiceAccount
		setdefault.PluginName,               //DefaultStorageClass
		resize.PluginName,                   //PersistentVolumeClaimResize
		defaulttolerationseconds.PluginName, //DefaultTolerationSeconds
		mutatingwebhook.PluginName,          //MutatingAdmissionWebhook
		validatingwebhook.PluginName,        //ValidatingAdmissionWebhook
		resourcequota.PluginName,            //ResourceQuota
	)

	if utilfeature.DefaultFeatureGate.Enabled(features.PodPriority) {
		defaultOnPlugins.Insert(podpriority.PluginName) //PodPriority
	}

	return sets.NewString(AllOrderedPlugins...).Difference(defaultOnPlugins)
}
```

在这里写了一些默认开启的配置。事实上，在早之前，`PersistentVolumeClaimResize` 默认是不开启的，并且开启了 `PersistentVolumeLabel`，对于移除 `Persistentvolumelabel` 感兴趣的朋友可以参考下 [Remove the PersistentVolumeLabel Admission Controller](https://github.com/kubernetes/kubernetes/issues/52617) 。

这里对几个比较常见的插件做下说明：

- NamespaceLifecycle：它可以保证正在终止的 `Namespace` 不允许创建对象，不允许请求不存在的 `Namespace` 以及保证默认的 `default`, `kube-system` 之类的命名空间不被删除。核心的代码是：

  ```
  if a.GetOperation() == admission.Delete && a.GetKind().GroupKind() == v1.SchemeGroupVersion.WithKind("Namespace").GroupKind() && l.immortalNamespaces.Has(a.GetName()) {
  	return errors.NewForbidden(a.GetResource().GroupResource(), a.GetName(), fmt.Errorf("this namespace may not be deleted"))
  }
  ```

  如果删除默认的 `Namespace` 则会得到下面的异常：

  ```
  master $ kubectl delete ns kube-system
  Error from server (Forbidden): namespaces "kube-system" is forbidden: this namespace may not be deleted
  master $ kubectl delete ns kube-public
  Error from server (Forbidden): namespaces "kube-public" is forbidden: this namespace may not be deleted
  master $ kubectl delete ns default
  Error from server (Forbidden): namespaces "default" is forbidden: this namespace may not be deleted
  ```

- LimitRanger：为 `Pod` 设置默认请求资源的限制。

- ServiceAccount：可按照预设规则创建 `Serviceaccount` 。比如都有统一的前缀：`system:serviceaccount:`。

- DefaultStorageClass：为 `PVC` 设置默认 `StorageClass`。

- DefaultTolerationSeconds：设置 `Pod` 的默认 forgiveness toleration 为 5 分钟。这个可能常会看到。

- MutatingAdmissionWebhook 和 ValidatingAdmissionWebhook：这两个都是通过 Webhook 验证或者修改请求，唯一的区别是一个是顺序进行，一个是并行进行的。

- ResourceQuota：限制 `Pod` 请求配额。

- AlwaysPullImages：总是拉取镜像。

- AlwaysAdmit：总是接受所有请求。

### 处理请求

前面已经说到，一个请求依次会经过认证，授权，准入控制等环节，当这些环节都已经通过后，该请求便到了 `kube-apiserver` 的实际处理逻辑中了。

其实和普通的 Web server 类似，`kube-apiserver` 提供了 `restful` 的接口，增删改查等基本功能都基本类似。这里先暂时不再深入。

## 总结

通过本节，我们学习到了 `kube-apiserver` 的基本工作逻辑，各类 API 请求先后通过认证，授权，准入控制等一系列环节后，进入到 `kube-apiserver` 的 `Registry` 进行相关逻辑处理。

至于需要进行持久化或者需要与后端存储交互的部分，我们在下节会介绍 `etcd` 到时再看 K8S 是如何将后端存储抽象化，从 `etcd` v2 升级至 v3 的。

`kube-apiserver` 包含的东西有很多，当你在终端下执行 `./kube-apiserver -h` 时，会发现有大量的参数。

这些参数除了认证，授权，准入控制相关功能

# 庖丁解牛：etcd

## 整体概览

```
+----------------------------------------------------------+          
| Master                                                   |          
|              +-------------------------+                 |          
|     +------->|        API Server       |<--------+       |          
|     |        |                         |         |       |          
|     v        +-------------------------+         v       |          
|   +----------------+     ^      +--------------------+   |          
|   |                |     |      |                    |   |          
|   |   Scheduler    |     |      | Controller Manager |   |          
|   |                |     |      |                    |   |          
|   +----------------+     v      +--------------------+   |          
| +------------------------------------------------------+ |          
| |                                                      | |          
| |                Cluster state store                   | |          
| |                                                      | |          
| +------------------------------------------------------+ |          
+----------------------------------------------------------+          
```

在第 3 节《宏观认识：整体架构》 中，我们也认识到了 `etcd` 的存在，知道了 Master 是 K8S 是集群的大脑，而 `etcd` 则是大脑的核心。为什么这么说？本节我们一同来看看 `etcd` 为何如此重要。

## `etcd` 是什么

先摘录[官方文档](https://etcd.readthedocs.io/en/latest/faq.html#what-is-etcd)的一句说明:

> etcd is a consistent distributed key-value store. Mainly used as a separate coordination service, in distributed systems. And designed to hold small amounts of data that can fit entirely in memory.

`etcd` 是由 CoreOS 团队发起的一个分布式，强一致的键值存储。它用 Go 语言编写，使用 `Raft` 协议作为一致性算法。多数情况下会用于分布式系统中的服务注册发现，或是用于存储系统的关键数据。

## `etcd` 有什么作用

`etcd` 在 K8S 中，最主要的作用便是其高可用，强一致的键值存储以及监听机制。

在 `kube-apiserver` 收到对应请求经过一系列的处理后，最终如果是集群所需要存储的数据，便会存储至 `etcd` 中。主部分主要是集群状态信息和元信息。

我们来实际操作 K8S 集群中的 `etcd` 看下真实情况。

- 查找集群中的 `etcd` Pod

```
# 默认集群中的 etcd 会放到 kube-system Namespace 中
master $ kubectl -n kube-system get pods | grep etcd
etcd-master                      1/1       Running   0          1h
```

- 进入该 Pod 并查看 `etcd` 集群的 `member`

```
master $ kubectl -n kube-system exec -it etcd-master sh
/ # ETCDCTL_API=3 etcdctl --key=/etc/kubernetes/pki/etcd/server.key  --cert=/etc/kubernetes/pki/etcd/server.crt  --cacert=/etc/kubernetes/pki/etcd/ca.crt member list
a874c87fd42044f, started, master, https://127.0.0.1:2380, https://127.0.0.1:2379
```

这里由于在 K8S 1.11.3 中默认使用的是 `etcd` 3.2 版本，所以需要加入 `ETCDCTL_API=3` 的环境变量，且 `etcd` 从 2 到 3 很明显的一个变化也是使用上的变化，在 2 中是 HTTP 接口的。

我们通过传递证书等相关参数进去，完成校验，查看 `member` 。

- 查看存储的元信息

`etcd` 中存储的 K8S 集群元信息基本都是 `/registry` 下，我们可通过下面的方式进行查看

```
/ # ETCDCTL_API=3 etcdctl --key=/etc/kubernetes/pki/etcd/server.key  --cert=/etc/kubernetes/pki/etcd/server.crt  --cacert=/etc/kubernetes/pki/etcd/ca.crt get /registry --prefix --keys-only
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/clusterroles/system:aggregate-to-admin
/registry/configmaps/kube-public/cluster-info
/registry/masterleases/172.17.0.53
/registry/namespaces/default
/registry/namespaces/kube-public
/registry/namespaces/kube-system
/registry/pods/kube-system/etcd-master
/registry/pods/kube-system/kube-apiserver-master
/registry/ranges/serviceips
/registry/ranges/servicenodeports
...
# 篇幅原因，删掉了很多输出
```

可以看到有各种类型的资源。我们直接以 Namespaces 为例。

- 查看 Namespaces 信息

```
/ # ETCDCTL_API=3 etcdctl --key=/etc/kubernetes/pki/etcd/server.key  --cert=/etc/kubernetes/pki/etcd/server.crt  --cacert=/etc/kubernetes/pki/etcd/ca.crt get /registry/namespaces --prefix --keys-only
/registry/namespaces/default

/registry/namespaces/kube-public

/registry/namespaces/kube-system
```

- 使用 `kubectl` 创建名为 `moelove` 的 Namespaces

```
master $ kubectl create  ns moelove
namespace/moelove created
```

- 查看 Namespaces 信息

```
/ # ETCDCTL_API=3 etcdctl --key=/etc/kubernetes/pki/etcd/server.key  --cert=/etc/kubernetes/pki/etcd/server.crt  --cacert=/etc/kubernetes/pki/etcd/ca.crt get /registry/namespaces --prefix --keys-only
/registry/namespaces/default

/registry/namespaces/kube-public

/registry/namespaces/kube-system

/registry/namespaces/moelove
```

发现刚创建的 `moelove` 的 Namespaces 已经在 `etcd` 中了。

## `etcd` 是如何被使用的

首先，在 `staging/src/k8s.io/apiserver/pkg/server/options/etcd.go` 中，存在一处声明：

```
var storageTypes = sets.NewString(
	storagebackend.StorageTypeUnset,
	storagebackend.StorageTypeETCD2,
	storagebackend.StorageTypeETCD3,
)
```

**在 1.13 发布时，`etcd 2` 的相关代码已经移除，其中就包含上面声明中的 `storagebackend.StorageTypeETCD2`**

这里是在过渡期间为了同时兼容 `etcd` 2 和 3 而增加的。

我们来看下实际对各类资源的操作，还是以对 `Namespace` 的操作为例：代码在 `pkg/registry/core/namespace/storage/storage.go` 中，

比如，`Get`：

```
func (r *REST) Get(ctx context.Context, name string, options *metav1.GetOptions) (runtime.Object, error) {
	return r.store.Get(ctx, name, options)
}
```

而 `REST` 则是这样定义的：

```
type REST struct {
	store  *genericregistry.Store
	status *genericregistry.Store
}
```

通过 `REST` 实现了一个 `RESTStorage`，实际使用时，也是调用了 `staging/src/k8s.io/apiserver/pkg/registry/generic/registry/store.go` 对接口的封装。

```
func (e *Store) Get(ctx context.Context, name string, options *metav1.GetOptions) (runtime.Object, error) {
	obj := e.NewFunc()
	key, err := e.KeyFunc(ctx, name)
	if err != nil {
		return nil, err
	}
	if err := e.Storage.Get(ctx, key, options.ResourceVersion, obj, false); err != nil {
		return nil, storeerr.InterpretGetError(err, e.qualifiedResourceFromContext(ctx), name)
	}
	if e.Decorator != nil {
		if err := e.Decorator(obj); err != nil {
			return nil, err
		}
	}
	return obj, nil
}
```

在该处实现了对各类基本方法的封装，各种资源类型都会统一去调用。而更上层则是对 `storagebackend` 的统一封装，最终会调用 `etcd` 客户端的实现完成想对应的操作，这里就不再多展开了。

## 总结

在本节中，我们介绍了 `etcd` 以及它在 K8S 中的作用以及如何使用。我们还介绍了如何通过 `etcdctl` 工具去操作 `etcd`。

在某些极端情况下，也许你需要通过直接操作 `etcd` 集群去变更数据，这里没有介绍所有的操作命令，感兴趣的可以自行通过下方的链接看官方文档进行学习。

但通常情况下，不建议直接操作 `etcd` ，除非你已经明确自己在做什么。

另外，由于 `etcd` 集群使用 `Raft` 一致性算法，通常情况下 `etcd` 集群需要部署奇数个节点，如 3，5，7 等。`etcd` 集群维护也相对容易，很容易可以做成高可用集群。（这也是保障 K8S 集群高可用的重要一环）

学习了 `etcd` 之后，下节我们来学习同样很重要的一个组件 `Controller Manager`，学习它是如何保障节群符合预期状态的。

## 参考链接

- [What is etcd](https://etcd.readthedocs.io/en/latest/faq.html#what-is-etcd)

# 庖丁解牛：controller-manager

## 整体概览

```
+----------------------------------------------------------+          
| Master                                                   |          
|              +-------------------------+                 |          
|     +------->|        API Server       |<--------+       |          
|     |        |                         |         |       |          
|     v        +-------------------------+         v       |          
|   +----------------+     ^      +--------------------+   |          
|   |                |     |      |                    |   |          
|   |   Scheduler    |     |      | Controller Manager |   |          
|   |                |     |      |                    |   |          
|   +----------------+     v      +--------------------+   |          
| +------------------------------------------------------+ |          
| |                                                      | |          
| |                Cluster state store                   | |          
| |                                                      | |          
| +------------------------------------------------------+ |          
+----------------------------------------------------------+          
```

在第 3 节《宏观认识：整体架构》 中，我们也认识到了 `Controller Manager` 的存在，知道了 Master 是 K8S 是集群的大脑，而它则是 Master 中最繁忙的部分。为什么这么说？本节我们一同来看看它为何如此繁忙。

**注意：`Controller Manager` 实际由 `kube-controller-manager` 和 `cloud-controller-manager` 两部分组成，`cloud-controller-manager` 则是为各家云厂商提供了一个抽象的封装，便于让各厂商使用各自的 `provide`。本文只讨论 `kube-controller-manager`，为了避免混淆，下文统一使用 `kube-controller-manager`。**

## `kube-controller-manager` 是什么

一句话来讲 `kube-controller-manager` 是一个嵌入了 K8S 核心控制循环的守护进程。

这里的重点是

- 嵌入：它已经内置了相关逻辑，可独立进行部署。我们在第 5 节下载 K8S 服务端二进制文件解压后，便可以看到 `kube-controller-manager` 的可执行文件，不过我们使用的是 `kubeadm` 进行的部署，它会默认使用 `k8s.gcr.io/kube-controller-manager` 的镜像。我们直接来看下实际情况：

```
master $ kubectl -n kube-system describe pods -l component=kube-controller-manager
Name:               kube-controller-manager-master
Namespace:          kube-system
Priority:           2000000000
PriorityClassName:  system-cluster-critical
Node:               master/172.17.0.35
Start Time:         Mon, 10 Dec 2018 07:14:21 +0000
Labels:             component=kube-controller-manager
                    tier=control-plane
Annotations:        kubernetes.io/config.hash=c7ed7a8fa5c430410e84970f8ee7e067
                    kubernetes.io/config.mirror=c7ed7a8fa5c430410e84970f8ee7e067
                    kubernetes.io/config.seen=2018-12-10T07:14:21.685626322Z
                    kubernetes.io/config.source=file
                    scheduler.alpha.kubernetes.io/critical-pod=
Status:             Running
IP:                 172.17.0.35
Containers:
  kube-controller-manager:
    Container ID:  docker://0653e71ae4287608726490b724c3d064d5f1556dd89b7d3c618e97f0e7f2a533
    Image:         k8s.gcr.io/kube-controller-manager-amd64:v1.11.3
    Image ID:      docker-pullable://k8s.gcr.io/kube-controller-manager-amd64@sha256:a6d115bb1c0116036ac6e6e4d504665bc48879c421a450566c38b3b726f0a123
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-controller-manager
      --address=127.0.0.1
      --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
      --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
      --controllers=*,bootstrapsigner,tokencleaner
      --kubeconfig=/etc/kubernetes/controller-manager.conf
      --leader-elect=true
      --root-ca-file=/etc/kubernetes/pki/ca.crt
      --service-account-private-key-file=/etc/kubernetes/pki/sa.key
      --use-service-account-credentials=true
    State:          Running
      Started:      Mon, 10 Dec 2018 07:14:24 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        200m
    Liveness:     http-get http://127.0.0.1:10252/healthz delay=15s timeout=15s period=10s #success=1 #failure=8
    Environment:  <none>
    Mounts:
      /etc/ca-certificates from etc-ca-certificates (ro)
      /etc/kubernetes/controller-manager.conf from kubeconfig (ro)
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/ssl/certs from ca-certs (ro)
      /usr/libexec/kubernetes/kubelet-plugins/volume/exec from flexvolume-dir (rw)
      /usr/local/share/ca-certificates from usr-local-share-ca-certificates (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  usr-local-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/local/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  etc-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ca-certificates
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  kubeconfig:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/controller-manager.conf
    HostPathType:  FileOrCreate
  flexvolume-dir:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/libexec/kubernetes/kubelet-plugins/volume/exec
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute
Events:            <none>
master
```

这是使用 `kubeadm` 搭建的集群中的 `kube-controller-manager` 的 `Pod`，首先可以看到它所使用的镜像，其次可以看到它使用的一系列参数，最后它在 `10252` 端口提供了健康检查的接口。稍后我们再展开。

- 控制循环：这里拆解为两部分： **控制** 和 **循环** ，它所控制的是集群的状态；至于循环它当然是会有个循环间隔的，这里有个参数可以进行控制。
- 守护进程：这个就不单独展开了。

## `kube-controller-manager` 有什么作用

前面已经说了它一个很关键的点 “控制”：它通过 `kube-apiserver` 提供的信息持续的监控集群状态，并尝试将集群调整至预期的状态。由于访问 `kube-apiserver` 也需要通过认证，授权等过程，所以可以看到上面启动 `kube-controller-manager` 时提供了一系列的参数。

比如，当我们创建了一个 `Deployment`，默认副本数为 1 ，当我们把 `Pod` 删除后，`kube-controller-manager` 会按照原先的预期，重新创建一个 `Pod` 。下面举个例子：

```
master $ kubectl run redis --image='redis'
deployment.apps/redis created
master $ kubectl get all
NAME                        READY     STATUS    RESTARTS   AGE
pod/redis-bb7894d65-w2rsp   1/1       Running   0          3m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   18m

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1         1         1            1           3m

NAME                              DESIRED   CURRENT   READY     AGE
replicaset.apps/redis-bb7894d65   1         1         1         3m
master $ kubectl delete pod/redis-bb7894d65-w2rsp
pod "redis-bb7894d65-w2rsp" deleted
master $ kubectl get all  # 可以看到已经重新运行了一个 Pod
NAME                        READY     STATUS    RESTARTS   AGE
pod/redis-bb7894d65-62ftk   1/1       Running   0          16s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   19m

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1         1         1            1           4m

NAME                              DESIRED   CURRENT   READY     AGE
replicaset.apps/redis-bb7894d65   1         1         1         4m
```

我们来看下 `kube-controller-manager` 的日志：

```
master $ kubectl -n kube-system logs -l component=kube-controller-manager --tail=5
I1210 09:30:17.125377       1 node_lifecycle_controller.go:945] Controller detected that all Nodes are not-Ready. Entering master disruption mode.
I1210 09:31:07.140539       1 node_lifecycle_controller.go:972] Controller detected that some Nodes are Ready. Exiting master disruption mode.
I1210 09:43:30.377649       1 event.go:221] Event(v1.ObjectReference{Kind:"Deployment", Namespace:"default", Name:"redis", UID:"0d1cb2d7-fc60-11e8-a361-0242ac110074", APIVersion:"apps/v1", ResourceVersion:"1494", FieldPath:""}): type: 'Normal' reason: 'ScalingReplicaSet' Scaled up replica setredis-bb7894d65 to 1
I1210 09:43:30.835149       1 event.go:221] Event(v1.ObjectReference{Kind:"ReplicaSet", Namespace:"default", Name:"redis-bb7894d65", UID:"0d344d15-fc60-11e8-a361-0242ac110074", APIVersion:"apps/v1", ResourceVersion:"1495", FieldPath:""}): type: 'Normal' reason: 'SuccessfulCreate' Created pod:redis-bb7894d65-w2rsp
I1210 09:47:41.658781       1 event.go:221] Event(v1.ObjectReference{Kind:"ReplicaSet", Namespace:"default", Name:"redis-bb7894d65", UID:"0d344d15-fc60-11e8-a361-0242ac110074", APIVersion:"apps/v1", ResourceVersion:"1558", FieldPath:""}): type: 'Normal' reason: 'SuccessfulCreate' Created pod:redis-bb7894d65-62ftk
```

可以看到它先观察到有 `Deployment` 的事件，然后 `ScalingReplicaSet` 进而创建了对应的 `Pod`。 而当我们删掉正在运行的 `Pod` 后，它便会重新创建 `Pod` 使集群状态符合原先的预期状态。

同时，注意 `Pod` 的名字已经发生了变化。

## `kube-controller-manager` 是如何工作的

在 `cmd/kube-controller-manager/app/controllermanager.go` 中列出了大多数的 `controllermanager`，他们对 `controllermanager` 函数的实际调用都在 `cmd/kube-controller-manager/app/core.go` 中，我们以 `PodGC` 为例：

```
func startPodGCController(ctx ControllerContext) (bool, error) {
	go podgc.NewPodGC(
		ctx.ClientBuilder.ClientOrDie("pod-garbage-collector"),
		ctx.InformerFactory.Core().V1().Pods(),
		int(ctx.ComponentConfig.PodGCController.TerminatedPodGCThreshold),
	).Run(ctx.Stop)
	return true, nil
}
```

在前两节中我们已经对 `kube-apiserver` 和 `etcd` 有了一些基本的认识，这里它主要会去 watch 相关的资源，但是出于性能上的考虑，也不能过于频繁的去请求 `kube-apiserver` 或者永久 watch ，所以在实现上借助了 [client-go](https://github.com/kubernetes/client-go) 的 `informer` 包，相当于是实现了一个本地的二级缓存。这里不做过多展开。

它最终会调用 `PodGC` 的具体实现，位置在 `pkg/controller/podgc/gc_controller.go` 中：

```
func NewPodGC(kubeClient clientset.Interface, podInformer coreinformers.PodInformer, terminatedPodThreshold int) *PodGCController {
	if kubeClient != nil && kubeClient.CoreV1().RESTClient().GetRateLimiter() != nil {
		metrics.RegisterMetricAndTrackRateLimiterUsage("gc_controller", kubeClient.CoreV1().RESTClient().GetRateLimiter())
	}
	gcc := &PodGCController{
		kubeClient:             kubeClient,
		terminatedPodThreshold: terminatedPodThreshold,
		deletePod: func(namespace, name string) error {
			glog.Infof("PodGC is force deleting Pod: %v:%v", namespace, name)
			return kubeClient.CoreV1().Pods(namespace).Delete(name, metav1.NewDeleteOptions(0))
		},
	}

	gcc.podLister = podInformer.Lister()
	gcc.podListerSynced = podInformer.Informer().HasSynced

	return gcc
}
```

代码也比较直观，不过这里可以看到有一个注册 `metrics` 的过程，实际上 `kube-controller-manager` 在前面的 `10252` 端口上不仅暴露出来了一个 `/healthz` 接口，还暴露出了一个 `/metrics` 的接口，可用于进行监控之类的。

```
master $ kubectl -n kube-system get pod -l component=kube-controller-manager
NAME                             READY     STATUS    RESTARTS   AGE
kube-controller-manager-master   1/1       Running   1          2m
master $ kubectl -n kube-system exec -it kube-controller-manager-master sh
/ # wget -qO- http://127.0.0.1:10252/metrics|grep gc_controller
# HELP gc_controller_rate_limiter_use A metric measuring the saturation of the rate limiter for gc_controller
# TYPE gc_controller_rate_limiter_use gauge
gc_controller_rate_limiter_use 0
```

## 总结

在本节中，我们介绍了 `kube-controller-manager` 以及它在 K8S 中主要是将集群调节至预期的状态，并提供出了 `/metrics` 的接口可供监控。

`kube-controller-manager` 中有很多的 controller 大多数是默认开启的，当然也有默认关闭的，比如 `bootstrapsigner` 和 `tokencleaner`，在我们启动 `kube-controller-manager` 的时候，可通过 `--controllers` 的参数进行控制，就比如上面例子中 `--controllers=*,bootstrapsigner,tokencleaner` 表示开启所有默认开启的以及 `bootstrapsigner` 和 `tokencleaner` 。

下节，我们将学习另一个与资源调度有关的组件 `kube-scheduler`，了解下它对我们使用集群所带来的意义。

留言

- [楊歌](https://juejin.im/user/5937807c2f301e006b268c6c)

  Controller Manager作为集群内部的管理控制中心，负责集群内的Node、Pod副本、服务端点（Endpoint）、命名空间（Namespace）、服务账号（ServiceAccount）、资源定额（ResourceQuota）的管理，当某个Node意外宕机时，Controller Manager会及时发现并执行自动化修复流程，确保集群始终处于预期的工作状态。

# 庖丁解牛：kube-scheduler

## 整体概览

```
+----------------------------------------------------------+          
| Master                                                   |          
|              +-------------------------+                 |          
|     +------->|        API Server       |<--------+       |          
|     |        |                         |         |       |          
|     v        +-------------------------+         v       |          
|   +----------------+     ^      +--------------------+   |          
|   |                |     |      |                    |   |          
|   |   Scheduler    |     |      | Controller Manager |   |          
|   |                |     |      |                    |   |          
|   +----------------+     v      +--------------------+   |          
| +------------------------------------------------------+ |          
| |                                                      | |          
| |                Cluster state store                   | |          
| |                                                      | |          
| +------------------------------------------------------+ |          
+----------------------------------------------------------+          
```

在第 3 节《宏观认识：整体架构》 中，我们也认识到了 `Scheduler` 的存在，知道了 Master 是 K8S 是集群的大脑，`Controller Manager` 负责将集群调整至预期的状态，而 `Scheduler` 则是集群调度器，将预期的 `Pod` 资源调度到正确的 `Node` 节点上，进而令该 `Pod` 可完成启动。本节我们一同来看看它如何发挥如此大的作用。

**下文统一使用 `kube-scheduler` 进行表述**

## `kube-scheduler` 是什么

引用[官方文档](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)一句话：

> The Kubernetes scheduler is a policy-rich, topology-aware, workload-specific function that significantly impacts availability, performance, and capacity.

`kube-scheduler` 是一个策略丰富，拓扑感知的调度程序，会显著影响可用性，性能和容量。

我们知道资源调度本就是 K8S 这类系统中的一个很复杂的事情，既要能满足系统对资源利用率的需要，同样还需要避免资源竞争，比如说端口冲突之类的。

为了能完成这样的需求，`kube-scheduler` 便在不断的迭代和发展，通过支持多种策略满足各类需求，通过感知拓扑避免资源竞争和保障系统的可用性及容量等。

我们在第 5 节下载服务端二进制文件解压后，便可看到 `kube-scheduler` 的可执行文件。当给它传递 `--help` 查看其支持参数的时候，便可以看到它支持使用 `--address` 或者 `--bind-address` 等参数指定所启动的 HTTP server 所绑定的地址之类的。

它和 `kube-controller-manager` 有点类似，同样是通过定时的向 `kube-apiserver` 请求获取信息，并进行处理。而他们所起到的作用并不相同。

## `kube-scheduler` 有什么作用

从上层的角度来看，`kube-scheduler` 的作用就是将待调度的 `Pod` 调度至最佳的 `Node` 上，而这个过程中则需要根据不同的策略，考虑到 `Node` 的资源使用情况，比如端口，内存，存储等。

## `kube-scheduler` 是如何工作的

整体的过程可通过 `pkg/scheduler/core/generic_scheduler.go` 的代码来看

```
func (g *genericScheduler) Schedule(pod *v1.Pod, nodeLister algorithm.NodeLister) (string, error) {
	trace := utiltrace.New(fmt.Sprintf("Scheduling %s/%s", pod.Namespace, pod.Name))
	defer trace.LogIfLong(100 * time.Millisecond)

	if err := podPassesBasicChecks(pod, g.pvcLister); err != nil {
		return "", err
	}

	nodes, err := nodeLister.List()
	if err != nil {
		return "", err
	}
	if len(nodes) == 0 {
		return "", ErrNoNodesAvailable
	}

	err = g.cache.UpdateNodeNameToInfoMap(g.cachedNodeInfoMap)
	if err != nil {
		return "", err
	}

	trace.Step("Computing predicates")
	startPredicateEvalTime := time.Now()
	filteredNodes, failedPredicateMap, err := g.findNodesThatFit(pod, nodes)
	if err != nil {
		return "", err
	}

	if len(filteredNodes) == 0 {
		return "", &FitError{
			Pod:              pod,
			NumAllNodes:      len(nodes),
			FailedPredicates: failedPredicateMap,
		}
	}
	metrics.SchedulingAlgorithmPredicateEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPredicateEvalTime))
	metrics.SchedulingLatency.WithLabelValues(metrics.PredicateEvaluation).Observe(metrics.SinceInSeconds(startPredicateEvalTime))

	trace.Step("Prioritizing")
	startPriorityEvalTime := time.Now()
	// When only one node after predicate, just use it.
	if len(filteredNodes) == 1 {
		metrics.SchedulingAlgorithmPriorityEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPriorityEvalTime))
		return filteredNodes[0].Name, nil
	}

	metaPrioritiesInterface := g.priorityMetaProducer(pod, g.cachedNodeInfoMap)
	priorityList, err := PrioritizeNodes(pod, g.cachedNodeInfoMap, metaPrioritiesInterface, g.prioritizers, filteredNodes, g.extenders)
	if err != nil {
		return "", err
	}
	metrics.SchedulingAlgorithmPriorityEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPriorityEvalTime))
	metrics.SchedulingLatency.WithLabelValues(metrics.PriorityEvaluation).Observe(metrics.SinceInSeconds(startPriorityEvalTime))

	trace.Step("Selecting host")
	return g.selectHost(priorityList)
}
```

它的输入有两个：

- `pod`：待调度的 `Pod` 对象；
- `nodeLister`：所有可用的 `Node` 列表

备注：`nodeLister` 的实现稍微用了点技巧，返回的是 `[]*v1.Node` 而不是 `v1.NodeList` 可避免拷贝带来的性能损失。

```
type NodeLister interface {
	List() ([]*v1.Node, error)
}
```

### 处理阶段

`kube-scheduler` 将处理阶段主要分为三个阶段 `Computing predicates`，`Prioritizing`和 `Selecting host`：

- `Computing predicates`：主要解决的问题是 `Pod` 能否调度到集群的 `Node` 上；

  主要是通过一个名为 `podFitsOnNode` 的函数进行实现，在检查的过程中也会先去检查下是否已经有已缓存的判断结果， 当然也会检查 `Pod` 是否是可调度的，以防有 `Pod Affinity` (亲合性) 之类的存在。

- `Prioritizing`：主要解决的问题是在上个阶段通过 `findNodesThatFit` 得到了 `filteredNodes` 的基础之上解决哪些 `Node` 是最优的，得到一个优先级列表 `priorityList`;

  至于优先级的部分，主要是通过下面的代码：

  ```
  for i := range nodes {
  	result = append(result, schedulerapi.HostPriority{Host: nodes[i].Name, Score: 0})
  	for j := range priorityConfigs {
  		result[i].Score += results[j][i].Score * priorityConfigs[j].Weight
  	}
  }
  ```

  给每个经过第一步筛选出来的 `Node` 一个 `Score`，再按照各种条件进行打分，最终得到一个优先级列表。

- `Selecting host`：则是最终选择 `Node` 调度到哪台机器上。

  最后，则是通过 `selectHost` 选择出最终要调度到哪台机器上。

  ```
  func (g *genericScheduler) selectHost(priorityList schedulerapi.HostPriorityList) (string, error) {
      if len(priorityList) == 0 {
          return "", fmt.Errorf("empty priorityList")
      }
  
      sort.Sort(sort.Reverse(priorityList))
      maxScore := priorityList[0].Score
      firstAfterMaxScore := sort.Search(len(priorityList), func(i int) bool { return priorityList[i].Score < maxScore })
  
      g.lastNodeIndexLock.Lock()
      ix := int(g.lastNodeIndex % uint64(firstAfterMaxScore))
      g.lastNodeIndex++
      g.lastNodeIndexLock.Unlock()
  
      return priorityList[ix].Host, nil
  }
  ```

## 总结

在本节中，我们介绍了 `kube-scheduler` 以及它在调度 `Pod` 的过程中的大致步骤。

不过它实际使用的各种策略及判断条件很多，无法在一节中完全都详细介绍，感兴趣的朋友可以按照本节中提供的思路大致去看看它的实现。

我们通过前面几节的介绍，已经知道了当实际进行部署操作的时候，首先会通过 `kubectl` 之类的客户端工具与 `kube-apiserver` 进行交互，在经过一系列的处理后，数据将持久化到 `etcd` 中；

此时，`kube-controller-manager` 通过持续的观察，开始按照我们的配置，将集群的状态调整至预期状态；

而 `kube-scheduler` 也在发挥作用，决定 `Pod` 应该调度至哪个或者哪些 `Node` 上；之后则通过其他组件的协作，最总将该 `Pod` 在相应的 `Node` 上部署启动。

我们在下节将要介绍的 `kubelet` 便是后面这部分“实际部署动作”相关的组件中尤为重要的一个，下节我们再详细介绍它是如何完成这些功能的。

# 庖丁解牛：kubelet

## 整体概览

```
+--------------------------------------------------------+       
| +---------------------+        +---------------------+ |       
| |      kubelet        |        |     kube-proxy      | |       
| |                     |        |                     | |       
| +---------------------+        +---------------------+ |       
| +----------------------------------------------------+ |       
| | Container Runtime (Docker)                         | |       
| | +---------------------+    +---------------------+ | |       
| | |Pod                  |    |Pod                  | | |       
| | | +-----+ +-----+     |    |+-----++-----++-----+| | |       
| | | |C1   | |C2   |     |    ||C1   ||C2   ||C3   || | |       
| | | |     | |     |     |    ||     ||     ||     || | |       
| | | +-----+ +-----+     |    |+-----++-----++-----+| | |       
| | +---------------------+    +---------------------+ | |       
| +----------------------------------------------------+ |       
+--------------------------------------------------------+  
```

在第 3 节《宏观认识：整体架构》 中，我们知道了 K8S 中 Node 由一些必要的组件构成，而其中最为核心的当属 `kubelet` 了，如果没有 `kubelet` 的存在，那我们预期的各类资源就只能存在于 `Master` 的相关组件中了，而 K8S 也很能只是一个 CRUD 的普通程序了。本节，我们来介绍下 `kubelet` 及它是如何完成这一系列任务的。

## `kubelet` 是什么

按照一般架构设计上的习惯，`kubelet` 所承担的角色一般会被叫做 `agent`，这里叫做 `kubelet` 很大程度上受 `Borg` 的命名影响，`Borg` 里面也有一个 `Borglet` 的组件存在。`kubelet` 便是 K8S 中的 `agent`，负责 `Node` 和 `Pod` 相关的管理任务。

同样的，在我们下载 K8S 二进制文件解压后，便可以得到 `kubelet` 的可执行文件。在第 5 节中，我们也完成了 `kubelet` 以 `systemd` 进行启动和管理的相关配置。

## `kubelet` 有什么作用

通常来讲 `agent` 这样的角色起到的作用首先便是要能够注册，让 `server` 端知道它的存在，所以这便是它的第一个作用：节点管理。

### 节点管理

当我们执行 `kubelet --help` 的时候，会看到它所支持的可配置参数，其中有一个 `--register-node` 参数便是用于控制是否向 `kube-apiserver` 注册节点的，默认是开启的。

我们在第 5 节中还介绍了如何新增一个 `Node`，当 `kubeadm join` 执行成功后，你便可以通过 `kubectl get node` 查看到新加入集群中的 `Node`，与此同时，你也可以在该节点上通过以下命令查看 `kubelet` 的状态。

```
master $ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Agent
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─kubeadm.conf
   Active: active (running) since Thu 2018-12-13 07:49:51 UTC; 32min ago
     Docs: http://kubernetes.io/docs/
 Main PID: 3876259 (kubelet)
   Memory: 66.3M
   CGroup: /system.slice/kubelet.service
           └─3876259 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernete...
```

当我们查看 `Node` 信息时，也能得到如下输出：

```
master $ kubectl get nodes node01 -o yaml
apiVersion: v1
kind: Node
metadata:
  annotations:
    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
    node.alpha.kubernetes.io/ttl: "0"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: 2018-12-13T07:50:47Z
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/hostname: node01
  name: node01
  resourceVersion: "4242"
  selfLink: /api/v1/nodes/node01
  uid: cd612df6-feab-11e8-9a0b-0242ac110096
spec: {}
status:
  addresses:
  - address: 172.17.0.152
    type: InternalIP
  - address: node01
    type: Hostname
  allocatable:
    cpu: "4"
    ephemeral-storage: "89032026784"
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 3894788Ki
    pods: "110"
  capacity:
    cpu: "4"
    ephemeral-storage: 96605932Ki
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 3997188Ki
    pods: "110"
  conditions:
  - lastHeartbeatTime: 2018-12-13T08:39:41Z
    lastTransitionTime: 2018-12-13T07:50:47Z
    message: kubelet has sufficient disk space available
    reason: KubeletHasSufficientDisk
    status: "False"
    type: OutOfDisk
  - lastHeartbeatTime: 2018-12-13T08:39:41Z
    lastTransitionTime: 2018-12-13T07:50:47Z
    message: kubelet has sufficient memory available
    reason: KubeletHasSufficientMemory
    status: "False"
    type: MemoryPressure
  - lastHeartbeatTime: 2018-12-13T08:39:41Z
    lastTransitionTime: 2018-12-13T07:50:47Z
    message: kubelet has no disk pressure
    reason: KubeletHasNoDiskPressure
    status: "False"
    type: DiskPressure
  - lastHeartbeatTime: 2018-12-13T08:39:41Z
    lastTransitionTime: 2018-12-13T07:50:47Z
    message: kubelet has sufficient PID available
    reason: KubeletHasSufficientPID
    status: "False"
    type: PIDPressure
  - lastHeartbeatTime: 2018-12-13T08:39:41Z
    lastTransitionTime: 2018-12-13T07:51:37Z
    message: kubelet is posting ready status. AppArmor enabled
    reason: KubeletReady
    status: "True"
    type: Ready
  daemonEndpoints:
    kubeletEndpoint:
      Port: 10250
  images:
  - names:
    - k8s.gcr.io/kube-apiserver-amd64@sha256:956bea8c139620c9fc823fb81ff9b5647582b53bd33904302987d56ab24fc187
    - k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    sizeBytes: 186676561
  nodeInfo:
    architecture: amd64
    bootID: 89ced22c-f7f8-4c4d-ad0d-d10887ab900e
    containerRuntimeVersion: docker://18.6.0
    kernelVersion: 4.4.0-62-generic
    kubeProxyVersion: v1.11.3
    kubeletVersion: v1.11.3
    machineID: 26ba042302eea8095d6576975c120eeb
    operatingSystem: linux
    osImage: Ubuntu 16.04.2 LTS
    systemUUID: 26ba042302eea8095d6576975c120eeb
```

可以看到 `kubelet` 不仅将自己注册给了 `kube-apiserver`，同时它所在机器的信息也都进行了上报，包括 CPU，内存，IP 信息等。

这其中有我们在第 2 节中提到的关于 `Node` 状态相关的一些信息，可以对照着看看。

当然这里除了这些信息外，还有些值得注意的，比如 `daemonEndpoints` 之类的，可以看到目前 `kubelet` 监听在了 `10250` 端口，这个端口可通过 `--port` 配置，但是之后会被废弃掉，我们是写入了 `/var/lib/kubelet/config.yaml` 的配置文件中。

```
master $ cat /var/lib/kubelet/config.yaml
address: 0.0.0.0
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: cgroupfs
cgroupsPerQOS: true
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
containerLogMaxFiles: 5
containerLogMaxSize: 10Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebuggingHandlers: true
enforceNodeAllocatable:
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: true
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kind: KubeletConfiguration
kubeAPIBurst: 10
kubeAPIQPS: 5
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
port: 10250
registryBurst: 10
registryPullQPS: 5
resolvConf: /etc/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 2m0s
serializeImagePulls: true
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
volumeStatsAggPeriod: 1m0s
master $
```

这其中有一些需要关注的配置：

- `maxPods`：最大的 `Pod` 数

- `healthzBindAddress` 和 `healthzPort`：配置了健康检查所监听的地址和端口

  我们可用以下方式进行验证：

  ```
  master $ curl 127.0.0.1:10248/healthz
  ok 
  ```

- `authentication` 和 `authorization` ：认证授权相关

- `evictionHard`：涉及到 `kubelet` 的驱逐策略，对 `Pod` 调度分配之类的影响很大

其余部分，可参考[手册内容](https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/)

### Pod 管理

从上面的配置以及我们之前的介绍中，`kube-scheduler` 处理了 `Pod` 应该调度至哪个 `Node`，而 `kubelet` 则是保障该 `Pod` 能按照预期，在对应 `Node` 上启动并保持工作。

同时，`kubelet` 在保障 `Pod` 能按预期工作，主要是做了两方面的事情：

- 健康检查：通过 `LivenessProbe` 和 `ReadinessProbe` 探针进行检查，判断是否健康及是否已经准备好接受请求。
- 资源监控：通过 `cAdvisor` 进行资源监。

## `kubelet` 是如何工作的

大致的功能已经介绍了，我们来看下它大体的实现。

首先是在 `cmd/kubelet/app/server.go` 文件中的 `Run` 方法：

```
func Run(s *options.KubeletServer, kubeDeps *kubelet.Dependencies, stopCh <-chan struct{}) error {
	glog.Infof("Version: %+v", version.Get())
	if err := initForOS(s.KubeletFlags.WindowsService); err != nil {
		return fmt.Errorf("failed OS init: %v", err)
	}
	if err := run(s, kubeDeps, stopCh); err != nil {
		return fmt.Errorf("failed to run Kubelet: %v", err)
	}
	return nil
}
```

这个方法看起来很简单那，它是在读取完一系列的配置和校验之后开始被调用的，在调用过程中，会在日志中输出当前的版本号，如果你的 `kubelet` 已经正常运行，当你执行 `journalctl -u kubelet` 的时候，便会看到一条相关的日志输出。

之后，便是一个 `run` 方法，其中包含着各种环境检查，容器管理，`cAdvisor` 初始化之类的操作，直到 `kubelet` 基本正确运行后，则会调用 `pkg/kubelet/kubelet.go` 中的一个 `BirthCry` 方法，该方法从命名就可以看出来，它其实就是宣告 `kubelet` 已经启动：

```
func (kl *Kubelet) BirthCry() {
	kl.recorder.Eventf(kl.nodeRef, v1.EventTypeNormal, events.StartingKubelet, "Starting kubelet.")
}
```

后续则是关于注册，`Pod` 管理以及资源相关的处理逻辑，内容较多，这里就不再展开了。

## 总结

本节中我们介绍了 `kubelet` 的主要功能和基本实现，了解到了它不仅可将自身注册到集群，同时还承担着保障 `Pod` 可在该 `Node` 上按预期工作。另外 `kubelet` 其实还承担着清理 `Node` 上一些由 K8S 调度 `Pod` 所造成的磁盘占用之类的工作。

从上面的配置中基本能看出来一些，这部分的功能大多数情况下不需要大家人为干预，所以也就不再展开了。

当 `Pod` 在 `Node` 上正常运行之后，若是需要对外提供服务，则需要将其暴露出来。下节，我们来介绍下 `kube-proxy` 是如何来处理这些工作的。

# 庖丁解牛：kube-proxy

## 整体概览

在第 3 节中，我们了解到 `kube-proxy` 的存在，而在第 7 中，我们学习到了如何将运行于 K8S 中的服务以 `Service` 的方式暴露出来，以供访问。

本节，我们来介绍下 `kube-proxy` 了解下它是如何支撑起这种类似服务发现和代理相关功能的。

## `kube-proxy` 是什么

`kube-proxy` 是 K8S 运行于每个 `Node` 上的网络代理组件，提供了 TCP 和 UDP 的连接转发支持。

我们已经知道，当 `Pod` 在创建和销毁的过程中，IP 可能会发生变化，而这就容易造成对其有依赖的服务的异常，所以通常情况下，我们都会使用 `Service` 将后端 `Pod` 暴露出来，而 `Service` 则较为稳定。

还是以我们之前的 [`SayThx`](https://github.com/tao12345666333/saythx) 项目为例，但我们只部署其中没有任何依赖的后端资源 `Redis` 。

```
master $ git clone https://github.com/tao12345666333/saythx.git
Cloning into 'saythx'...
remote: Enumerating objects: 110, done.
remote: Counting objects: 100% (110/110), done.
remote: Compressing objects: 100% (82/82), done.
remote: Total 110 (delta 27), reused 102 (delta 20), pack-reused 0
Receiving objects: 100% (110/110), 119.42 KiB | 0 bytes/s, done.
Resolving deltas: 100% (27/27), done.
Checking connectivity... done.
master $ cd saythx/deploy
master $ ls
backend-deployment.yaml  frontend-deployment.yaml  namespace.yaml         redis-service.yaml
backend-service.yaml     frontend-service.yaml     redis-deployment.yaml  work-deployment.yaml
```

进入配置文件所在目录后，开始创建相关资源：

```
master $ kubectl apply -f namespace.yaml
namespace/work created
master $ kubectl apply -f redis-deployment.yaml
deployment.apps/saythx-redis created
master $ kubectl  apply -f redis-service.yaml
service/saythx-redis created
master $ kubectl -n work get all
NAME                               READY     STATUS    RESTARTS   AGE
pod/saythx-redis-8558c7d7d-wsn2w   1/1       Running   0          21s

NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/saythx-redis   NodePort   10.103.193.175   <none>        6379:31269/TCP   6s

NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/saythx-redis   1         1         1            1           21s

NAME                                     DESIRED   CURRENT   READY     AGE
replicaset.apps/saythx-redis-8558c7d7d   1         1         1         21s
```

可以看到 Redis 正在运行，并通过 `NodePort` 类型的 `Service` 暴露出来，我们访问来确认下。

```
master $ docker run --rm -it --network host redis:alpine redis-cli -p 31269
Unable to find image 'redis:alpine' locally
alpine: Pulling from library/redis
4fe2ade4980c: Already exists
fb758dc2e038: Pull complete
989f7b0c858b: Pull complete
8dd99d530347: Pull complete
7137334fa8f0: Pull complete
30610ca64487: Pull complete
Digest: sha256:8fd83c5986f444f1a5521e3eda7395f0f21ff16d33cc3b89d19ca7c58293c5dd
Status: Downloaded newer image for redis:alpine
127.0.0.1:31269> set name kubernetes
OK
127.0.0.1:31269> get name 
"kubernetes"
```

可以看到已经可以正常访问。接下来，我们来看下 `31269` 这个端口的状态。

```
master $ netstat  -ntlp |grep 31269
tcp6       0      0 :::31269                :::*                    LISTEN      2716/kube-proxy
```

可以看到该端口是由 `kube-proxy` 所占用的。

接下来，查看当前集群的 `Service` 和 `Endpoint`

```
master $ kubectl -n work get svc
NAME           TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
saythx-redis   NodePort   10.103.193.175   <none>        6379:31269/TCP   10m
master $ kubectl -n work get endpoints
NAME           ENDPOINTS        AGE
saythx-redis   10.32.0.2:6379   10m
master $ kubectl -n work get pod -o wide
NAME                           READY     STATUS    RESTARTS   AGE       IP          NODE      NOMINATED NODE
saythx-redis-8558c7d7d-wsn2w   1/1       Running   0          12m       10.32.0.2   node01    <none>
```

可以很直观的看到 `Endpoint` 当中的便是 `Pod` 的 IP，现在我们将该服务进行扩容（实际情况下并不会这样处理）。

直接通过 `kubectl scale` 操作

```
master $ kubectl  -n work scale --replicas=2 deploy/saythx-redis
deployment.extensions/saythx-redis scaled
master $ kubectl  -n work get all
NAME                               READY     STATUS    RESTARTS   AGE
pod/saythx-redis-8558c7d7d-sslpj   1/1       Running   0          10s
pod/saythx-redis-8558c7d7d-wsn2w   1/1       Running   0          16m

NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/saythx-redis   NodePort   10.103.193.175   <none>        6379:31269/TCP   16m

NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/saythx-redis   2         2         2            2           16m
```

查看 `Endpoint` 信息：

```
master $ kubectl -n work get endpoints
NAME           ENDPOINTS                       AGE
saythx-redis   10.32.0.2:6379,10.32.0.3:6379   17m
```

可以看到 `Endpoint` 已经自动发生了变化，而这也意味着 `Service` 代理的后端节点将增加一个。

## `kube-proxy` 如何工作

`kube-proxy` 在 Linux 系统上当前支持三种模式，可通过 `--proxy-mode` 配置：

- `userspace`：这是很早期的一种方案，但效率上显著不足，不推荐使用。
- `iptables`：当前的默认模式。比 `userspace` 要快，但问题是会给机器上产生很多 `iptables` 规则。
- `ipvs`：为了解决 `iptables` 的性能问题而引入，采用增量的方式进行更新。

下面我们以 `iptables` 的模式稍作介绍。

```
master $ iptables -t nat -L 
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
KUBE-SERVICES  all  --  anywhere             anywhere             /* kubernetes service portals */
DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
KUBE-SERVICES  all  --  anywhere             anywhere             /* kubernetes service portals */
DOCKER     all  --  anywhere            !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
KUBE-POSTROUTING  all  --  anywhere             anywhere             /* kubernetes postrouting rules */
MASQUERADE  all  --  172.18.0.0/24        anywhere

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere

Chain KUBE-MARK-DROP (0 references)
target     prot opt source               destination
MARK       all  --  anywhere             anywhere             MARK or 0x8000

Chain KUBE-MARK-MASQ (7 references)
target     prot opt source               destination
MARK       all  --  anywhere             anywhere             MARK or 0x4000

Chain KUBE-NODEPORTS (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  tcp  --  anywhere             anywhere             /* work/saythx-redis: */ tcp dpt:31269
KUBE-SVC-SMQNAAUIAENDDGYQ  tcp  --  anywhere             anywhere             /* work/saythx-redis: */ tcp dpt:31269

Chain KUBE-POSTROUTING (1 references)
target     prot opt source               destination
MASQUERADE  all  --  anywhere             anywhere             /* kubernetes service traffic requiring SNAT */ mark match 0x4000/0x4000

Chain KUBE-SEP-2LZPYBS4HUAJKDFL (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  all  --  10.32.0.2            anywhere             /* kube-system/kube-dns:dns-tcp */
DNAT       tcp  --  anywhere             anywhere             /* kube-system/kube-dns:dns-tcp */ tcp to:10.32.0.2:53

Chain KUBE-SEP-3E4LNQKKWZF7G6SH (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  all  --  10.32.0.1            anywhere             /* kube-system/kube-dns:dns-tcp */
DNAT       tcp  --  anywhere             anywhere             /* kube-system/kube-dns:dns-tcp */ tcp to:10.32.0.1:53

Chain KUBE-SEP-3IDG7DUGN3QC2UZF (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  all  --  172.17.0.120         anywhere             /* default/kubernetes:https */
DNAT       tcp  --  anywhere             anywhere             /* default/kubernetes:https */ tcp to:172.17.0.120:6443

Chain KUBE-SEP-JZWS2VPNIEMNMNB2 (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  all  --  10.32.0.2            anywhere             /* kube-system/kube-dns:dns */
DNAT       udp  --  anywhere             anywhere             /* kube-system/kube-dns:dns */ udp to:10.32.0.2:53

Chain KUBE-SEP-OEY6JJQSBCQPRKHS (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  all  --  10.32.0.1            anywhere             /* kube-system/kube-dns:dns */
DNAT       udp  --  anywhere             anywhere             /* kube-system/kube-dns:dns */ udp to:10.32.0.1:53

Chain KUBE-SEP-QX7VDAS5KDY6V3EV (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  all  --  10.32.0.2            anywhere             /* work/saythx-redis: */
DNAT       tcp  --  anywhere             anywhere             /* work/saythx-redis: */ tcp to:10.32.0.2:6379

Chain KUBE-SERVICES (2 references)
target     prot opt source               destination
KUBE-SVC-SMQNAAUIAENDDGYQ  tcp  --  anywhere             10.103.193.175       /* work/saythx-redis: cluster IP */ tcp dpt:6379
KUBE-NODEPORTS  all  --  anywhere             anywhere             /* kubernetes service nodeports; NOTE: this must be the last rule in this chain */ ADDRTYPE match dst-type LOCAL

Chain KUBE-SVC-ERIFXISQEP7F7OF4 (1 references)
target     prot opt source               destination
KUBE-SEP-3E4LNQKKWZF7G6SH  all  --  anywhere             anywhere             /* kube-system/kube-dns:dns-tcp */ statistic mode random probability 0.50000000000
KUBE-SEP-2LZPYBS4HUAJKDFL  all  --  anywhere             anywhere             /* kube-system/kube-dns:dns-tcp */


Chain KUBE-SVC-SMQNAAUIAENDDGYQ (2 references)
target     prot opt source               destination
KUBE-SEP-QX7VDAS5KDY6V3EV  all  --  anywhere             anywhere             /* work/saythx-redis: */
```

以上输出已经尽量删掉了无关的内容。

当开始访问的时候先要经过 `PREROUTING` 链，转到 `KUBE-SERVICES` 链，当查询到匹配的规则之后，请求将转向 `KUBE-SVC-SMQNAAUIAENDDGYQ` 链，进而到达 `KUBE-SEP-QX7VDAS5KDY6V3EV` 对应于我们的 `Pod`。(注：为了简洁，上述 iptables 规则是部署一个 `Pod` 时的场景)

当搞懂了这些之后，如果你想了解这些 `iptables` 规则实际又是如何创建和维护的，那可以参考下 `proxier` 的具体实现，这里不再展开。

## 总结

本节中我们介绍了 `kube-proxy` 的主要功能和基本流程，了解到了它对于服务注册发现和代理访问等起到了很大的作用。而它在 Linux 下的代理模式也有 `userspace`，`iptables` 和 `ipvs` 等。

默认情况下我们使用 `iptables` 的代理模式，当创建新的 `Service` ，或者 `Pod` 进行变化时，`kube-proxy` 便会去维护 `iptables` 规则，以确保请求可以正确的到达后端服务。

当然，本节中并没有提到 `kube-proxy` 的 `session affinity` 相关的特性，如有需要可进行下尝试。

下节，我们将介绍实际运行着容器的 `Docker`，大致了解下在 K8S 中它所起的作用，及他们之间的交互方式。

# 庖丁解牛：Container Runtime （Docker）

## 整体概览

我们在第 3 节的时候，提到过 `Container Runtime` 的概念，也大致介绍过它的主要作用在于下载镜像，运行容器等。

经过我们前面的学习，`kube-scheduler` 决定了 `Pod` 将被调度到哪个 `Node` 上，而 `kubelet` 则负责 `Pod` 在此 `Node` 上可按预期工作。如果没有 `Container Runtime`，那 `Pod` 中的 `container` 在该 `Node` 上也便无法正常启动运行了。

本节中，我们以当前最为通用的 `Container Runtime` Docker 为例进行介绍。

## Container Runtime 是什么

`Container Runtime` 我们通常叫它容器运行时，而这一概念的产生也是由于容器化技术和 K8S 的大力发展，为了统一工业标准，也为了避免 K8S 绑定于特定的容器运行时，所以便成立了 [Open Container Initiative (OCI)](https://www.opencontainers.org/) 组织，致力于将容器运行时标准化和容器镜像标准化。

凡是遵守此标准的实现，均可由标准格式的镜像启动相应的容器，并完成一些特定的操作。

## Docker 是什么

Docker 是一个容器管理平台，它最初是被设计用于快速创建，发布和运行容器的工具，不过随着它的发展，其中集成了越来越多的功能。

Docker 也可以说是一个包含标准容器运行时的工具集，当前版本中默认的 `runtime` 称之为 `runc`。 关于 `runc` 相关的一些内容可参考[我之前的一篇文章](http://moelove.info/2018/11/23/runc-1.0-rc6-发布之际/)。

当然，这里提到了 **默认的运行时** 那也就意味着它可支持其他的运行时实现。

## CRI 是什么

说到这里，我们就会发现，K8S 作为目前云原生技术体系中最重要的一环，为了让它更有扩展性，当然也不会将自己完全局限于某一种特定的容器运行时。

自 K8S 1.5 （2016 年 11 月）开始，新增了一个容器运行时的插件 API，并称之为 `CRI` （Container Runtime Interface），通过 `CRI` 可以支持 `kubelet` 使用不同的容器运行时，而不需要重新编译。

`CRI` 主要是基于 gRPC 实现了 `RuntimeService` 和 `ImageService` 这两个服务，可以参考 `pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto` 中的 API 定义。由于本节侧重于 `Container Runtime/Docker` 这里就不对 `CRI` 的具体实现进行展开了。

只要继续将 `kubelet` 当作 agent 的角色，而它与基于 `CRI` 实现的 `CRI shim` 服务进行通信理解即可。

## Docker 如何工作

这里我们主要介绍在 K8S 中一些 Docker 常见的动作。

### 部署一个 Redis

```
master $ kubectl run redis --image=redis
deployment.apps/redis created
master $ kubectl get all
NAME                        READY     STATUS              RESTARTS   AGE
pod/redis-bb7894d65-7vsj8   0/1       ContainerCreating   0          6s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   26m

NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1         1         1            0           6s

NAME                              DESIRED   CURRENT   READY     AGE
replicaset.apps/redis-bb7894d65   1         1         0         6s
```

我们直接使用 `kubectl run` 的方式部署了一个 Redis

### 查看详情

```
master $ kubectl describe pod/redis-bb7894d65-7vsj8
Name:               redis-bb7894d65-7vsj8
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               node01/172.17.0.21
Start Time:         Sat, 15 Dec 2018 04:48:49 +0000
Labels:             pod-template-hash=663450821
                    run=redis
Annotations:        <none>
Status:             Running
IP:                 10.40.0.1
Controlled By:      ReplicaSet/redis-bb7894d65
Containers:
  redis:
    Container ID:   docker://ab87085456aca76825dd639bcde27160d9c2c84cac5388585bcc9ed3afda6522
    Image:          redis
    Image ID:       docker-pullable://redis@sha256:010a8bd5c6a9d469441aa35187d18c181e3195368bce309348b3ee639fce96e0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 15 Dec 2018 04:48:57 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-zxt27 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-zxt27:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-zxt27
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  7m    default-scheduler  Successfully assigned default/redis-bb7894d65-7vsj8to node01
  Normal  Pulling    7m    kubelet, node01    pulling image "redis"
  Normal  Pulled     7m    kubelet, node01    Successfully pulled image "redis"
  Normal  Created    7m    kubelet, node01    Created container
  Normal  Started    7m    kubelet, node01    Started container
```

可以通过 `kubectl describe` 查看该 `Pod` 的事件详情。这里主要有几个阶段。

#### 调度

```
Normal  Scheduled  7m    default-scheduler  Successfully assigned default/redis-bb7894d65-7vsj8to node01
```

在第 15 小节 `kube-scheduler` 中我们介绍过，通过 `kube-scheduler` 可以决定 `Pod` 会调度到哪个 `Node`。本例中，`redis-bb7894d65-7vsj8to` 被调度到了 `node01`。

#### pull 镜像

```
Normal  Pulling    7m    kubelet, node01    pulling image "redis"
Normal  Pulled     7m    kubelet, node01    Successfully pulled image "redis"
```

这里 `kubelet` 及该节点上的 `Container Runtime` （Docker）开始发挥作用，先拉取镜像。如果此刻你登录 `node01` 的机器，执行 `docker pull redis` 便可同步看到拉取进度。

#### 创建镜像并启动

```
Normal  Created    7m    kubelet, node01    Created container
Normal  Started    7m    kubelet, node01    Started container
```

拉取镜像完成后，便会开始创建并启动该容器，并返回任务结果。此刻登录 `node01` 机器，便会看到当前在运行的容器了。

```
node01 $ docker ps |grep redis
ab87085456ac        redis@sha256:010a8bd5c6a9d469441aa35187d18c181e3195368bce309348b3ee639fce96e0  "docker-entrypoint..."   19 minutes ago      Up 19 minutes                           k8s_redis_redis-bb7894d65-7vsj8_default_b693b56c-0024-11e9-9bab-0242ac11000a_0
8f264abd82fe        k8s.gcr.io/pause:3.1  "/pause"                 19 minutes ago      Up 19 minutes                           k8s_POD_redis-bb7894d65-7vsj8_default_b693b56c-0024-11e9-9bab-0242ac11000a_0
```

## 总结

本节我们介绍了 `Container Runtime` 的基本概念，及 K8S 为了能增加扩展性，提供了统一的 `CRI` 插件接口，可用于支持多种容器运行时。

当前使用最为广泛的是 [`Docker`](https://github.com/moby/moby/)，当前还支持的主要有 [`runc`](https://github.com/opencontainers/runc)，[`Containerd`](https://github.com/containerd/containerd)，[`runV`](https://github.com/hyperhq/runv) 以及 [`rkt`](https://github.com/rkt/rkt) 等。

由于 Docker 的知识点很多，关于 Docker 的实践和内部原理可参考我之前的一次分享 [Docker 实战和基本原理](https://github.com/tao12345666333/slides/raw/master/2018.09.13-Tech-Talk-Time/Docker实战和基本原理-张晋涛.pdf)。

在使用 K8S 时，也有极个别情况需要通过排查 Docker 的日志来分析问题。

至此，K8S 中主要的核心组件我们已经介绍完毕，下节我们主要集中于在 K8S 环境中，如何定位和解决问题，以及类似刚才提到的需要通过 Docker 进行排查问题的情况。

# Troubleshoot

## 整体概览

通过前面的介绍，我们已经了解到了 K8S 的基础知识，核心组件原理以及如何在 K8S 中部署服务及管理服务等。

但在生产环境中，我们所面临的环境多种多样，可能会遇到各种问题。本节将结合我们已经了解到的知识，介绍一些常见问题定位和解决的思路或方法，以便大家在生产中使用 K8S 能如鱼得水。

## 应用部署问题

首先我们从应用部署相关的问题来入手。这里仍然使用我们的[示例项目 SayThx](https://github.com/tao12345666333/saythx)。

clone 该项目，进入到 deploy 目录中，先 `kubectl apply -f namespace.yaml` 或者 `kubectl create ns work` 来创建一个用于实验的 `Namespace` 。

### 使用 `describe` 排查问题

对 `redis-deployment.yaml` 稍作修改，按以下方式操作：

```
master $ kubectl apply -f redis-deployment.yaml
deployment.apps/saythx-redis created
master $ kubectl -n work get all
NAME                                READY     STATUS             RESTARTS   AGE
pod/saythx-redis-7574c98f5d-v66fx   0/1       ImagePullBackOff   0          9s

NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/saythx-redis   1         1         1            0           9s

NAME                                      DESIRED   CURRENT   READY     AGE
replicaset.apps/saythx-redis-7574c98f5d   1         1         0         9s
```

可以看到 `Pod` 此刻的状态是 `ImagePullBackOff`，这个状态表示镜像拉取失败，`kubelet` 退出镜像拉取。

我们在前面的内容中介绍过 `kubelet` 的作用之一就是负责镜像拉取，而实际上，在镜像方面的错误主要预设了 6 种，分别是 `ImagePullBackOff`，`ImageInspectError`，`ErrImagePull`，`ErrImageNeverPull`，`RegistryUnavailable`，`InvalidImageName`。

当遇到以上所述情况时，便可定位为镜像相关异常。

我们回到上面的问题当中，定位问题所在。

```
master $ kubectl -n work describe pod/saythx-redis-7574c98f5d-v66fx
Name:               saythx-redis-7574c98f5d-v66fx
Namespace:          work
Priority:           0
PriorityClassName:  <none>
Node:               node01/172.17.0.132
Start Time:         Tue, 18 Dec 2018 17:27:56 +0000
Labels:             app=redis
                    pod-template-hash=3130754918
Annotations:        <none>
Status:             Pending
IP:                 10.40.0.1
Controlled By:      ReplicaSet/saythx-redis-7574c98f5d
Containers:
  redis:
    Container ID:
    Image:          redis:5xx
    Image ID:
    Port:           6379/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-787w5 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-787w5:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-787w5
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason          Age                 From               Message
  ----     ------          ----                ----               -------
  Normal   Scheduled       11m                 default-scheduler  Successfully assigned work/saythx-redis-7574c98f5d-v66fx to node01
  Normal   SandboxChanged  10m                 kubelet, node01    Pod sandbox changed, it will bekilled and re-created.
  Normal   BackOff         9m (x6 over 10m)    kubelet, node01    Back-off pulling image "redis:5xx"
  Normal   Pulling         9m (x4 over 10m)    kubelet, node01    pulling image "redis:5xx"
  Warning  Failed          9m (x4 over 10m)    kubelet, node01    Failed to pull image "redis:5xx": rpc error: code = Unknown desc = Error response from daemon: manifest for redis:5xx not found
  Warning  Failed          9m (x4 over 10m)    kubelet, node01    Error: ErrImagePull
  Warning  Failed          49s (x44 over 10m)  kubelet, node01    Error: ImagePullBackOff
```

可以看到我们现在 pull 的镜像是 `redis:5xx` 而实际上并不存在此 tag 的镜像，所以导致拉取失败。

### 使用 `events` 排查问题

当然，我们还有另一种方式同样可进行问题排查：

```
master $ kubectl -n work get events
LAST SEEN   FIRST SEEN   COUNT     NAME                                             KIND         SUBOBJECT                TYPE      REASON              SOURCE                  MESSAGE
21m         21m          1         saythx-redis.15717d6361a741a8                    Deployment                        Normal    ScalingReplicaSet   deployment-controller   Scaled up replica set saythx-redis-7574c98f5d to 1
21m         21m          1         saythx-redis-7574c98f5d-qwxgm.15717d6363eb60ff   Pod                        Normal    Scheduled           default-scheduler       Successfully assigned work/saythx-redis-7574c98f5d-qwxgm to node01
21m         21m          1         saythx-redis-7574c98f5d.15717d636309afa8         ReplicaSet                        Normal    SuccessfulCreate    replicaset-controller   Created pod: saythx-redis-7574c98f5d-qwxgm
20m         21m          2         saythx-redis-7574c98f5d-qwxgm.15717d63fa501b3f   Pod          spec.containers{redis}   Normal    BackOff             kubelet, node01         Back-off pulling image "redis:5xx"
20m         21m          2         saythx-redis-7574c98f5d-qwxgm.15717d63fa5049a9   Pod          spec.containers{redis}   Warning   Failed              kubelet, node01         Error: ImagePullBackOff
20m         21m          3         saythx-redis-7574c98f5d-qwxgm.15717d6393a1993c   Pod          spec.containers{redis}   Normal    Pulling             kubelet, node01         pulling image "redis:5xx"
20m         21m          3         saythx-redis-7574c98f5d-qwxgm.15717d63e11efc7a   Pod          spec.containers{redis}   Warning   Failed              kubelet, node01         Error: ErrImagePull
20m         21m          3         saythx-redis-7574c98f5d-qwxgm.15717d63e11e9c25   Pod          spec.containers{redis}   Warning   Failed              kubelet, node01         Failed to pull image "redis:5xx": rpc error: code = Unknown desc = Error response from daemon: manifest for redis:5xxnot found
20m         20m          1         saythx-redis-54984ff94-2bb6g.15717d6dc03799cd    Pod          spec.containers{redis}   Normal    Killing             kubelet, node01         Killing container with id docker://redis:Need to kill Pod
19m         19m          1         saythx-redis-7574c98f5d-v66fx.15717d72356528ec   Pod                        Normal    Scheduled           default-scheduler       Successfully assigned work/saythx-redis-7574c98f5d-v66fx to node01
19m         19m          1         saythx-redis-7574c98f5d.15717d722f7f1732         ReplicaSet                        Normal    SuccessfulCreate    replicaset-controller   Created pod: saythx-redis-7574c98f5d-v66fx
19m         19m          1         saythx-redis.15717d722b49e758                    Deployment                        Normal    ScalingReplicaSet   deployment-controller   Scaled up replica set saythx-redis-7574c98f5d to 1
19m         19m          1         saythx-redis-7574c98f5d-v66fx.15717d731a09b0ad   Pod                        Normal    SandboxChanged      kubelet, node01         Pod sandbox changed, it will be killed and re-created.
18m         19m          6         saythx-redis-7574c98f5d-v66fx.15717d733ab20b3d   Pod          spec.containers{redis}   Normal    BackOff             kubelet, node01         Back-off pulling image "redis:5xx"
18m         19m          4         saythx-redis-7574c98f5d-v66fx.15717d729de13541   Pod          spec.containers{redis}   Normal    Pulling             kubelet, node01         pulling image "redis:5xx"
18m         19m          4         saythx-redis-7574c98f5d-v66fx.15717d72e6ded95d   Pod          spec.containers{redis}   Warning   Failed              kubelet, node01         Error: ErrImagePull
18m         19m          4         saythx-redis-7574c98f5d-v66fx.15717d72e6de7b1c   Pod          spec.containers{redis}   Warning   Failed              kubelet, node01         Failed to pull image "redis:5xx": rpc error: code = Unknown desc = Error response from daemon: manifest for redis:5xxnot found
4m          19m          66        saythx-redis-7574c98f5d-v66fx.15717d733ab23f2c   Pod          spec.containers{redis}   Warning   Failed              kubelet, node01         Error: ImagePullBackOff
master
```

我们在之前介绍时，也提到过 `kubelet` 或者 `kube-scheduler` 等组件会接受某些事件等，`event` 便是用于记录集群内各处发生的事件之类的。

### 修正错误

- 修正配置文件

  修正配置文件，然后 `kubectl apply -f redis-deployment.yaml` 便可应用修正后的配置文件。这种方法比较推荐，并且可以将修改过的位置纳入到版本控制系统中，有利于后续维护。

- 在线修改配置

  使用 `kubectl -n work edit deploy/saythx-redis`，会打开默认的编辑器，我们可以将使用的镜像及 tag 修正为 `redis:5` 保存退出，便会自动应用新的配置。这种做法比较适合比较紧急或者资源是直接通过命令行创建等情况。 **非特殊情况尽量不要在线修改。** 且这样修改并不利于后期维护。

### 通过详细内容排查错误

```
master $ kubectl apply -f namespace.yaml
namespace/work created
master $ kubectl apply -f redis-deployment.yaml
deployment.apps/saythx-redis created
master $ vi redis-service.yaml # 稍微做了点修改
master $ kubectl apply -f redis-service.yaml
service/saythx-redis created
master $ kubectl -n work get pods,svc
NAME                               READY     STATUS    RESTARTS   AGE
pod/saythx-redis-8558c7d7d-z8prg   1/1       Running   0          47s

NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/saythx-redis   NodePort   10.108.202.170   <none>        6379:32355/TCP   16s
```

通过以上的输出，大多数情况下我们的 `Service` 应该是可以可以正常访问了，现在我们进行下测试：

```
master $ docker run --rm -it --net host redis redis-cli -p 32355
Unable to find image 'redis:latest' locally
latest: Pulling from library/redis
a5a6f2f73cd8: Pull complete
a6d0f7688756: Pull complete
53e16f6135a5: Pull complete
f52b0cc4e76a: Pull complete
e841feee049e: Pull complete
ccf45e5191d0: Pull complete
Digest: sha256:bf65ecee69c43e52d0e065d094fbdfe4df6e408d47a96e56c7a29caaf31d3c35
Status: Downloaded newer image for redis:latest
Could not connect to Redis at 127.0.0.1:32355: Connection refused
not connected>
```

我们先来介绍这里的测试方法。 使用 Docker 的 Redis 官方镜像， `--net host` 是使用宿主机网络； `--rm` 表示停止完后即清除； `-it` 分别表示获取输入及获取 TTY。

通过以上测试发现不能正常连接，故而说明 `Service` 还是未配置好。使用前面提到的方法也可以进行排查，不过这里提供另一种排查这类问题的思路。

```
master $ kubectl -n work get endpoints
NAME           ENDPOINTS        AGE
saythx-redis   10.32.0.4:6380   9m
```

通过之前的章节，我们已经知道 `Service` 工作的时候是按 `Endpoints` 来的，这里我们发现此处的 `Endpoints` 是 `6380` 与我们预期的 `6379` 并不相同。所以问题定位于端口配置有误。

前面已经说过修正方法了，不再赘述。当修正完成后，再次验证：

```
master $ kubectl -n work get endpoints
NAME           ENDPOINTS        AGE
saythx-redis   10.32.0.4:6379   15m
```

`Endpoints` 已经正常，验证下服务是否可用：

```
master $ docker run --rm -it --net host redis redis-cli -p 32355
127.0.0.1:32355> ping
PONG
```

验证无误。

## 集群问题

由于我们有多个节点，况且在集群搭建和维护过程中，也会比较常见到集群相关的问题。这里我们先举个实际例子进行分析：

```
master $ kubectl get nodes
NAME      STATUS     ROLES     AGE       VERSION
master    Ready      master    58m       v1.11.3
node01    NotReady   <none>    58m       v1.11.3
```

通过 kubectl 查看，发现有一个节点 NotReady ，这在搭建集群的过程中也有可能遇到。

```
master $ kubectl get  node/node01 -o yaml
apiVersion: v1
kind: Node
metadata:
  annotations:
    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
    node.alpha.kubernetes.io/ttl: "0"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: 2018-12-19T16:46:59Z
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/hostname: node01
  name: node01
  resourceVersion: "4850"
  selfLink: /api/v1/nodes/node01
  uid: b440d3d5-03ad-11e9-917e-0242ac110035
spec: {}
status:
  addresses:
  - address: 172.17.0.66
    type: InternalIP
  - address: node01
    type: Hostname
  allocatable:
    cpu: "4"
    ephemeral-storage: "89032026784"
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 3894652Ki
    pods: "110"
  capacity:
    cpu: "4"
    ephemeral-storage: 96605932Ki
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 3997052Ki
    pods: "110"
  conditions:
  - lastHeartbeatTime: 2018-12-19T17:42:16Z
    lastTransitionTime: 2018-12-19T17:43:00Z
    message: Kubelet stopped posting node status.
    reason: NodeStatusUnknown
    status: Unknown
    type: OutOfDisk
  - lastHeartbeatTime: 2018-12-19T17:42:16Z
    lastTransitionTime: 2018-12-19T17:43:00Z
    message: Kubelet stopped posting node status.
    reason: NodeStatusUnknown
    status: Unknown
    type: MemoryPressure
  - lastHeartbeatTime: 2018-12-19T17:42:16Z
    lastTransitionTime: 2018-12-19T17:43:00Z
    message: Kubelet stopped posting node status.
    reason: NodeStatusUnknown
    status: Unknown
    type: DiskPressure
  - lastHeartbeatTime: 2018-12-19T17:42:16Z
    lastTransitionTime: 2018-12-19T16:46:59Z
    message: kubelet has sufficient PID available
    reason: KubeletHasSufficientPID
    status: "False"
    type: PIDPressure
  - lastHeartbeatTime: 2018-12-19T17:42:16Z
    lastTransitionTime: 2018-12-19T17:43:00Z
    message: Kubelet stopped posting node status.
    reason: NodeStatusUnknown
    status: Unknown
    type: Ready
  daemonEndpoints:
    kubeletEndpoint:
      Port: 10250
  ...
```

我们之前介绍 `kubelet` 时说过， `kubelet` 的作用之一便是将自身注册至 `kube-apiserver`。

这里的 message 信息说明 `kubelet` 不再向 `kube-apiserver` 发送心跳包之类的了，所以被判定为 NotReady 的状态。

接下来，我们登录 node01 机器查看 `kubelet` 的状态。

```
node01 $ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─kubeadm.conf
   Active: inactive (dead) since Wed 2018-12-19 17:42:17 UTC; 18min ago
     Docs: https://kubernetes.io/docs/home/
  Process: 1693 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_
 Main PID: 1693 (code=exited, status=0/SUCCESS)
```

可以看到该机器上 `kubelet` 没有启动。现在将其启动，稍等片刻看看节群中 `Node` 的状态。

```
master $ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    1h        v1.11.3
node01    Ready     <none>    1h        v1.11.3
```

## 总结

本节我们介绍了 K8S 中常用的问题排查和解决思路，但实际生产环境中情况会有和更多不确定因素，掌握本节中介绍的基础，有利于之后生产环境中进行常规问题的排查。

当然，本节只是介绍通过 kubectl 来定位和解决问题，个别情况下我们需要登录相关的节点，实际去使用 `Docker` 工具等进行问题的详细排查。

至此，K8S 的基础原理和常规问题排查思路等都已经通过包括本节在内的 19 小节介绍完毕，相信你现在已经迫不及待的想要使用 K8S 了。

不过 kubectl 作为命令行工具也许有些人会不习惯使用，下节，我们将介绍 K8S 的扩展组件 `kube-dashboard` 了解它的主要功能及带给我们的便利。

# 扩展增强：Dashboard

## 整体概览

通过前面的介绍，想必你已经迫不及待的想要将应用部署至 K8S 中，但总是使用 `kubectl` 或者 `Helm` 等命令行工具也许不太直观，你可能想要一眼就看到集群当前的状态，或者想要更方便的对集群进行管理。

本节将介绍一个 Web 项目 [`Dashboard`](https://github.com/kubernetes/dashboard) 可用于部署容器化的应用程序，管理集群中的资源，甚至是排查和解决问题。

当然它和大多数 Dashboard 类的项目类似，也为集群的状态提供了一个很直观的展示。



![img](https://user-gold-cdn.xitu.io/2018/12/23/167d6b6e66c60e89?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 如何安装

要想使用 Dashboard，首先我们需要安装它，而 Dashboard 的安装其实也很简单。不过对于国内用户需要注意的是需要解决网络问题，或替换镜像地址等。

这里我们安装当前最新版 `v1.10.1` 的 Dashboard：

- 对于已经解决网络问题的用户：

  可使用官方推荐做法进行安装，以下链接是使用了我提交了 path 的版本，由于官方最近的一次更新导致配置文件中的镜像搞错了。

  ```
  master $ kubectl apply -f https://raw.githubusercontent.com/tao12345666333/dashboard/67970554aa9275cccec1d1ee5fbf89ae81b3b614/aio/deploy/recommended/kubernetes-dashboard.yaml
  secret/kubernetes-dashboard-certs created
  serviceaccount/kubernetes-dashboard created
  role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
  rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
  deployment.apps/kubernetes-dashboard created
  service/kubernetes-dashboard created
  ```

- 也可使用我修改过的这份（使用 Docker Hub 同步了镜像）仓库地址 [GitHub](https://github.com/tao12345666333/k8s-dashboard), 国内 [Gitee](https://gitee.com/K8S-release/k8s-dashboard)：

  ```
  master $ kubectl apply -f https://gitee.com/K8S-release/k8s-dashboard/raw/master/kubernetes-dashboard.yaml
  secret/kubernetes-dashboard-certs created
  serviceaccount/kubernetes-dashboard created
  role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
  rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
  deployment.apps/kubernetes-dashboard created
  service/kubernetes-dashboard created
  ```

当已经执行完以上步骤后，可检查下是否安装成功：

```
master $ kubectl -n kube-system get all  -l k8s-app=kubernetes-dashboard
NAME                                        READY     STATUS    RESTARTS   AGE
pod/kubernetes-dashboard-67896bc598-dhdpz   1/1       Running   0          3m

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes-dashboard   ClusterIP   10.109.92.207   <none>        443/TCP   3m

NAME                                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kubernetes-dashboard   1         1         1            1           3m

NAME                                              DESIRED   CURRENT   READY     AGE
replicaset.apps/kubernetes-dashboard-67896bc598   1         1         1         3m
```

可以看到 `Pod` 已经在正常运行，接下来便是访问 Dashboard.

## 访问 Dashboard

以当前的部署方式，`Service` 使用了 `ClusterIP` 的类型，所以在集群外不能直接访问。我们先使用 `kubectl` 提供的 `port-forward` 功能进行访问。

```
master $ kubectl -n kube-system port-forward pod/kubernetes-dashboard-67896bc598-dhdpz 8443
Forwarding from 127.0.0.1:8443 -> 8443
Forwarding from [::1]:8443 -> 8443
```

还记得，我们在第 5 节时候安装过一个名为 `socat` 的依赖项吗？ `socat` 的主要功能便是端口转发。

现在在浏览器打开 [`https://127.0.0.1:8443`](https://127.0.0.1:8443/) 便可看到如下的登录界面。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1176" height="565"></svg>)



对于我们的 **新版本** 而言，我们 **使用令牌登录** 的方式。

### 查找 Token

```
master $ kubectl -n kube-system get serviceaccount -l k8s-app=kubernetes-dashboard -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":"kube-system"}}
    creationTimestamp: 2018-12-20T17:27:14Z
    labels:
      k8s-app: kubernetes-dashboard
    name: kubernetes-dashboard
    namespace: kube-system
    resourceVersion: "1400"
    selfLink: /api/v1/namespaces/kube-system/serviceaccounts/kubernetes-dashboard
    uid: 7e01ddda-047c-11e9-b55c-0242ac11002a
  secrets:
  - name: kubernetes-dashboard-token-6ck2l
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

首先，我们查看刚才创建出的 `serviceaccount` 可以看到其中有配置 `secrets` 。

查看该 `secret` 详情获得 Token

```
master $ kubectl -n kube-system describe secrets kubernetes-dashboard-token-6ck2l
Name:         kubernetes-dashboard-token-6ck2l
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=kubernetes-dashboard
              kubernetes.io/service-account.uid=7e01ddda-047c-11e9-b55c-0242ac11002a

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi02Y2sybCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjdlMDFkZGRhLTA0N2MtMTFlOS1iNTVjLTAyNDJhYzExMDAyYSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.WZ5YRUkGlKRSpkBFCk3BrZ6p2t1qVxEs7Kb18DP5X2C2lfMhDrB931PeN05uByLD6biz_4IQvKh4xmvY2RqekfV1BLCfcIiMUbc1lcXGbhH4g4vrsjYx3NZifaBh_5HuBlEL5zs5e_zFkPEhhIqjsY3KueFEuGwxTAsqGBQwawc-v6wqzB3Gzb01o1iN5aTb37PVG5gTTE8cQLih_urKhvdNEKBSRg_zHQlYjFrtUUWYRYMlYz_sWmamYVXHy_7NvKrBfw44WU5tLxMITkoUEGVwROBnHf_BcWVedozLg2uLVontB12YvhmTfJCDEAJ8o937bS-Fq3tLfu_xM40fqw
```

将此处的 token 填入输入框内便可登录，**注意这里使用的是 `describe`。**



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="635"></svg>)



### 修正权限

但是我们注意到这里有很多提示 `configmaps is forbidden: User "system:serviceaccount:kube-system:kubernetes-dashboard" cannot list resource "configmaps" in API group "" in the namespace "default"` 。根据我们前面的介绍，这很明显就是用户权限不足。

我们已经知道，当前我们的集群是开启了 `RBAC` 的，所以这里我们还是以前面学到的方法创建一个用户并进行授权。

- 创建 ServiceAccount：

  ```
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: admin-user
    namespace: kube-system
  ```

- 创建 RoleBinding: 这里为了方便直接绑定了 `cluster-admin` 的 ClusterRole ，但是生产环境下，请按照实际情况进行授权，参考前面第 8 节相关的内容。

  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: admin-user
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
  subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kube-system
  ```

使用以上配置创建了用户和绑定，然后还是同样的办法获取 Token。

点击 Dashboard 右上角，退出登录后，重新使用新的 Token 进行登录。登录完成后便可看到如下图：



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="607"></svg>)



## 部署应用

点击右上角的 **+创建** 可进入创建页面，现在支持三种模式：从文本框输入；从文件创建；直接创建应用。

我们仍然以我们的示例项目 [SayThx](https://github.com/tao12345666333/saythx) 为例。先 `clone` 该项目，并进入项目的 `deploy` 目录中。将 `namespace.yaml` 的内容复制进输入框，点击上传按钮，便可创建名为 `work` 的 `Namespace` 了。

通过以下命令验证：

```
master $ kubectl get ns
NAME          STATUS    AGE
default       Active    2h
kube-public   Active    2h
kube-system   Active    2h
work          Active    10s
```

可以看到 Namespace 已经创建成功。或者刷新下网页，点击左侧的命名空间即可看到当前的所有 `Namespace` 。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="196" height="246"></svg>)



我们先将左侧的命名空间选择为 **全部命名空间** 或 **work** (当刷新过网页后) ，接下来继续点击右上角的 **+创建** 按钮，将 `redis-deployment.yaml` 的内容复制进输入框，点击上传按钮，部署 Redis 。

部署成功后，点击 部署 ，点击刚才的 `saythx-redis` 便可看到其详情。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="603"></svg>)



点击左侧的容器组，便可看到刚才部署的 Pod，



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="612"></svg>)



在此页面的右上角，可以点击命令行按钮，打开新标签页进入其内部执行命令。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="609"></svg>)



或者是点击日志按钮，可打开新标签页，查看日志。



![img](https://user-gold-cdn.xitu.io/2018/12/23/167d6bafed569ab1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 总结

本节我们介绍了 `Kubernetes Dashboard` 的基本功能，以及如何安装和使用它。

Dashboard 相比 `kubectl` 为用户提供了一种相对直观的 Web 端操作方式，但是并不能完全取代 `kubectl`，这两者应该是相辅相成的。

如果你所需的功能相对简单或是想要给某些用户提供一种通过 Web 操作的方式，那便推荐使用 Dashboard。Dashboard 的后端使用了 K8S 的 [`client-go`](https://github.com/kubernetes/client-go) ，前端主要使用了 [Angular](https://angular.io/)，有兴趣可以大致看看其源代码，对于开发基于 K8S 的云平台会有些启发。

下节，我们将介绍用于 DNS 和服务发现的插件 [CoreDNS](https://coredns.io/)，学习如何利用它完成这些需求。并且它在 K8S 1.13 版本中，已经成为了默认的 DNS server。

# 扩展增强：CoreDNS

## 整体概览

通过前面的学习，我们知道在 K8S 中有一套默认的[集群内 DNS 服务](https://github.com/kubernetes/dns)，我们通常把它叫做 `kube-dns`，它基于 SkyDNS，为我们在服务注册发现方面提供了很大的便利。

比如，在我们的示例项目 [SayThx](https://github.com/tao12345666333/saythx) 中，各组件便是依赖 DNS 进行彼此间的调用。

本节，我们将介绍的 [CoreDNS](https://coredns.io/) 是 CNCF 旗下又一孵化项目，在 K8S 1.9 版本中加入并进入 Alpha 阶段。我们当前是以 K8S 1.11 的版本进行介绍，它并不是默认的 DNS 服务，但是[它作为 K8S 的 DNS 插件的功能已经 GA](https://github.com/kubernetes/enhancements/issues/427) 。

CoreDNS 在 K8S 1.13 版本中才正式成为[默认的 DNS 服务](https://kubernetes.io/blog/2018/12/03/kubernetes-1-13-release-announcement/)。

## CoreDNS 是什么

首先，我们需要明确 CoreDNS 是一个独立项目，它不仅可支持在 K8S 中使用，你也可以在你任何需要 DNS 服务的时候使用它。

CoreDNS 使用 Go 语言实现，部署非常方便。

它的扩展性很强，很多功能特性都是通过插件完成的，它不仅有大量的[内置插件](https://coredns.io/plugins/)，同时也有很丰富的[第三方插件](https://coredns.io/explugins/)。甚至你自己[写一个插件](https://coredns.io/2016/12/19/writing-plugins-for-coredns/)也非常的容易。

## 如何安装使用 CoreDNS

我们这里主要是为了说明如何在 K8S 环境中使用它，所以对于独立安装部署它不做说明。

本小册中我们使用的是 K8S 1.11 版本，在第 5 小节 《搭建 Kubernetes 集群》中，我们介绍了使用 `kubeadm` 搭建集群。

使用 `kubeadm` 创建集群时候 `kubeadm init` 可以传递 `--feature-gates` 参数，用于启用一些额外的特性。

比如在之前版本中，我们可以通过 `kubeadm init --feature-gates CoreDNS=true` 在创建集群时候启用 CoreDNS。

而在 1.11 版本中，使用 `kubeadm` 创建集群时 `CoreDNS` 已经被默认启用，这也从侧面证明了 CoreDNS 在 K8S 中达到了生产可用的状态。

我们来看一下创建集群时的日志输出：

```
[root@master ~]# kubeadm init
[init] using Kubernetes version: v1.11.3               
[preflight] running pre-flight checks
...
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!
```

可以看到创建时已经启用了 CoreDNS 的扩展，待集群创建完成后，可用过以下方式进行查看：

```
master $ kubectl -n kube-system get all  -l k8s-app=kube-dns -o wide
NAME                           READY     STATUS    RESTARTS   AGE       IP          NODE      NOMINATED NODE
pod/coredns-78fcdf6894-5zbx4   1/1       Running   0          1h        10.32.0.3   node01    <none>
pod/coredns-78fcdf6894-cxdw8   1/1       Running   0          1h        10.32.0.2   node01    <none>

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE       SELECTOR
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   1h        k8s-app=kube-dns

NAME                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES                     SELECTOR
deployment.apps/coredns   2         2         2            2           1h        coredns      k8s.gcr.io/coredns:1.1.3   k8s-app=kube-dns

NAME                                 DESIRED   CURRENT   READY     AGE       CONTAINERS   IMAGES                   SELECTOR
replicaset.apps/coredns-78fcdf6894   2         2         2         1h        coredns      k8s.gcr.io/coredns:1.1.3   k8s-app=kube-dns,pod-template-hash=3497892450
```

这里主要是为了兼容 K8S 原有的 `kube-dns` 所以标签和 `Service` 的名字都还使用了 `kube-dns`，但实际在运行的则是 CoreDNS。

## 验证 CoreDNS 功能

从上面的输出我们看到 CoreDNS 的 `Pod` 运行正常，现在测试下它是否能正确解析。仍然以我们的示例项目 [SayThx](https://github.com/tao12345666333/saythx) 为例，先 clone 项目，进入到项目的 deploy 目录中。

```
master $ cd saythx/deploy/
master $ ls
backend-deployment.yaml  frontend-deployment.yaml  namespace.yaml         redis-service.yaml
backend-service.yaml     frontend-service.yaml     redis-deployment.yaml  work-deployment.yaml
master $ kubectl apply -f namespace.yaml
namespace/work created
master $ kubectl apply -f redis-deployment.yaml
deployment.apps/saythx-redis created
master $ kubectl apply -f redis-service.yaml
service/saythx-redis created
```

- 查看其部署情况：

```
master $ kubectl -n work get all
NAME                               READY     STATUS    RESTARTS   AGE
pod/saythx-redis-8558c7d7d-8v4lp   1/1       Running   0          2m

NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/saythx-redis   NodePort   10.109.215.147   <none>        6379:31438/TCP   2m

NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/saythx-redis   1         1         1            1           2m

NAME                                     DESIRED   CURRENT   READY     AGE
replicaset.apps/saythx-redis-8558c7d7d   1         1         1         2m
```

- 验证 DNS 是否正确解析:

```
# 使用 AlpineLinux 的镜像创建一个 Pod 并进入其中
master $ kubectl run alpine -it --rm --restart='Never' --image='alpine' sh
If you don't see a command prompt, try pressing enter.
/ # apk add --no-cache bind-tools
fetch http://dl-cdn.alpinelinux.org/alpine/v3.8/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.8/community/x86_64/APKINDEX.tar.gz
(1/5) Installing libgcc (6.4.0-r9)
(2/5) Installing json-c (0.13.1-r0)
(3/5) Installing libxml2 (2.9.8-r1)
(4/5) Installing bind-libs (9.12.3-r0)
(5/5) Installing bind-tools (9.12.3-r0)
Executing busybox-1.28.4-r2.trigger
OK: 9 MiB in 18 packages

# 安装完 dig 命令所在包之后，使用 dig 命令进行验证
/ # dig @10.32.0.2 saythx-redis.work.svc.cluster.local +noall +answer

; <<>> DiG 9.12.3 <<>> @10.32.0.2 saythx-redis.work.svc.cluster.local +noall +answer
; (1 server found)
;; global options: +cmd
saythx-redis.work.svc.cluster.local. 5 IN A     10.109.215.147
```

通过以上操作，可以看到相应的 `Service` 记录可被正确解析。这里有几个点需要注意：

- 域名解析是可跨 `Namespace` 的

  刚才的示例中，我们没有指定 `Namespace` 所以刚才我们所在的 `Namespace` 是 `default`。而我们的解析实验成功了。说明 CoreDNS 的解析是全局的。**虽然解析是全局的，但不代表网络互通**

- 域名有特定格式

  可以看到刚才我们使用的完整域名是 `saythx-redis.work.svc.cluster.local` , 注意开头的便是 **Service 名.Namespace 名** 当然，我们也可以直接通过 `host` 命令查询:

  ```
  / # host -t srv  saythx-redis.work
  saythx-redis.work.svc.cluster.local has SRV record 0 100 6379 saythx-redis.work.svc.cluster.local.
  ```

## 配置和监控

CoreDNS 使用 `ConfigMap` 的方式进行配置，但是如果更改了配置，`Pod` 重启后才会生效。

我们通过以下命令可查看其配置：

```
master $ kubectl -n kube-system get configmap coredns -o yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           upstream
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . /etc/resolv.conf
        cache 30
        reload
    }
kind: ConfigMap
metadata:
  creationTimestamp: 2018-12-22T16:45:47Z
  name: coredns
  namespace: kube-system
  resourceVersion: "217"
  selfLink: /api/v1/namespaces/kube-system/configmaps/coredns
  uid: 0882e51b-0609-11e9-b25e-0242ac110057
```

`Corefile` 便是它的配置文件，可以看到它启动了类似 `kubernetes`, `prometheus` 等插件。

注意 `kubernetes` 插件的配置，使用的域是 `cluster.local` ，这也是上面我们提到域名格式时候后半部分未解释的部分。

至于 `prometheus` 插件，则是监听在 9153 端口上提供了符合 Prometheus 标准的 metrics 接口，可用于监控等。关于监控的部分，可参考第 23 节。

## 总结

在本节中，我们介绍了 CoreDNS 的基本情况，它是以 Go 编写的灵活可扩展的 DNS 服务器。

使用 CoreDNS 代替 kube-dns 主要是为了解决一些 kube-dns 时期的问题，比如说原先 kube-dns 的时候，一个 Pod 中还需要包含 `kube-dns`, `sidecar` 和 `dnsmasq` 的容器，而每当 `dnsmasq` 出现漏洞时，就不得不让 K8S 发个安全补丁才能进行更新。

CoreDNS 有丰富的插件，可以满足更多样的应用需求，同时 `kubernetes` 插件还包含了一些独特的功能，比如 Pod 验证之类的，可增加安全性。

同时 CoreDNS 在 1.13 版本中会作为默认的 DNS 服务器使用，所以应该给它更多的关注。

在下节，我们将介绍 `Ingress`，看看如果使用不同于之前使用的 `NodePort` 的方式将服务暴露于外部。

# 服务增强：Ingress

## 整体概览

通过前面的学习，我们已经知道 K8S 中有 `Service` 的概念，同时默认情况下还有 `CoreDNS` 完成集群内部的域名解析等工作，以此完成基础的服务注册发现能力。

在第 7 节中，我们介绍了 `Service` 的 4 种基础类型，在前面的介绍中，我们一般都在使用 `ClusterIP` 或 `NodePort` 等方式将服务暴露在集群内或者集群外。

本节，我们将介绍另一种处理服务访问的方式 `Ingress`。

## Ingress 是什么

通过 `kubectl explain ingress` 命令，我们来看下对 Ingress 的描述。

> Ingress is a collection of rules that allow inbound connections to reach the endpoints defined by a backend. An Ingress can be configured to give services externally-reachable urls, load balance traffic, terminate SSL, offer name based virtual hosting etc.

`Ingress` 是一组允许外部请求进入集群的路由规则的集合。它可以给 `Service` 提供集群外部访问的 URL，负载均衡，SSL 终止等。

直白点说，`Ingress` 就类似起到了智能路由的角色，外部流量到达 `Ingress` ，再由它按已经制定好的规则分发到不同的后端服务中去。

看起来它很像我们使用的负载均衡器之类的。那你可能会问，`Ingress` 与 `LoadBalancer` 类型的 `Service` 的区别是什么呢？

- `Ingress` 不是一种 `Service` 类型

  `Ingress` 是 K8S 中的一种资源类型，我们可以直接通过 `kubectl get ingress` 的方式获取我们已有的 `Ingress` 资源。

- `Ingress` 可以有多种控制器（实现）

  通过之前的介绍，我们知道 K8S 中有很多的 `Controller` (控制器)，而这些 `Controller` 已经打包进了 `kube-controller-manager` 中，通过 `--controllers` 参数控制启用哪些。

  但是 `Ingress` 的 `Controller` 并没有包含在其中，而且有多种选择。

  由社区维护（或是说官方支持的）有两个：适用于 Google Cloud 的 [GLBC](https://github.com/kubernetes/ingress-gce)，当你使用 GKE 的时候，便会看到它；和 [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx) 它是使用 `ConfigMap` 存储 NGINX 配置实现的。

  第三方的实现还有：基于 Envoy 的 [Contour](https://github.com/heptio/contour); F5 的 [F5 BIG-IP Controller](https://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/v1.7/); 基于 HAProxy 的 [haproxy-ingress](https://github.com/jcmoraisjr/haproxy-ingress); 基于 Istio 的 [Control Ingress Traffic](https://istio.io/docs/tasks/traffic-management/ingress/); 现代化的反向代理服务器 [Traefik](https://github.com/containous/traefik); 以及 Kong 支持的 [Kong Ingress Controller for Kubernetes](https://konghq.com/blog/kubernetes-ingress-controller-for-kong/) 和 NGINX 官方支持的 [NGINX Ingress Controller](https://github.com/nginxinc/kubernetes-ingress)。

  这里可以看到 K8S 社区和 NGINX 都有 NGINX Ingress Controller，很多人在一开始接触 Ingress 的时候便陷入了选择的苦恼中，除去前面的那些选择外，单 NGINX 的控制器就有两个，到底应该怎么选。

  这里提供两点建议：

  - 可能多数人使用的都是 NGINX 而非 NGINX Plus，如果你需要会话保持（Session persistence）的话，那你应该选择 K8S 社区维护的版本
  - 即使我们平时使用 NGINX 的时候，也常常会有动态配置的需求，如果你仍然有这样的需求，那你还是继续使用 K8S 社区维护的版本（其中内置了 Lua 支持）。

## 如何使用

前面也已经说到了，单纯的创建一个 `Ingress` 资源没什么意义，我们需要先配置一个 `Controller` ，才能让它正常工作。国内使用 GKE 的可能不是很多，为了更加通用，这里我们选择 K8S 社区维护的 NGINX Ingress Controller。

### 安装

整个安装过程其实也比较简单，具体步骤如下（以下步骤中都将直接展示该步骤所需的 YAML 配置文件）：

- 创建 `Namespace`

  ```
  apiVersion: v1
  kind: Namespace
  metadata:
    name: ingress-nginx
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  ```

  将以上内容保存为 namespace.yaml 文件，然后执行 `kubectl apply -f namespace.yaml` 即可。以下步骤均类似，不再赘述。 注意：这里创建 `Namespace` 只是为了保持集群相对规范，非强制，但推荐此做法。

- 创建 `ConfigMap`

  ```
  kind: ConfigMap
  apiVersion: v1
  metadata:
    name: nginx-configuration
    namespace: ingress-nginx
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  
  ---
  kind: ConfigMap
  apiVersion: v1
  metadata:
    name: tcp-services
    namespace: ingress-nginx
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  
  ---
  kind: ConfigMap
  apiVersion: v1
  metadata:
    name: udp-services
    namespace: ingress-nginx
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  
  ---
  ```

  这里创建了几个 `ConfigMap`，主要是给 `Controller` 使用。

- 由于我们的集群使用 `kubeadm` 创建时，默认开启了 `RBAC` ，所以这里需要相应的创建对应的 `Role` 和 `RoleBinding`。

  ```
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  
  ---
  apiVersion: rbac.authorization.k8s.io/v1beta1
  kind: ClusterRole
  metadata:
    name: nginx-ingress-clusterrole
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  rules:
    - apiGroups:
        - ""
      resources:
        - configmaps
        - endpoints
        - nodes
        - pods
        - secrets
      verbs:
        - list
        - watch
    - apiGroups:
        - ""
      resources:
        - nodes
      verbs:
        - get
    - apiGroups:
        - ""
      resources:
        - services
      verbs:
        - get
        - list
        - watch
    - apiGroups:
        - "extensions"
      resources:
        - ingresses
      verbs:
        - get
        - list
        - watch
    - apiGroups:
        - ""
      resources:
        - events
      verbs:
        - create
        - patch
    - apiGroups:
        - "extensions"
      resources:
        - ingresses/status
      verbs:
        - update
  
  ---
  apiVersion: rbac.authorization.k8s.io/v1beta1
  kind: Role
  metadata:
    name: nginx-ingress-role
    namespace: ingress-nginx
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  rules:
    - apiGroups:
        - ""
      resources:
        - configmaps
        - pods
        - secrets
        - namespaces
      verbs:
        - get
    - apiGroups:
        - ""
      resources:
        - configmaps
      resourceNames:
        - "ingress-controller-leader-nginx"
      verbs:
        - get
        - update
    - apiGroups:
        - ""
      resources:
        - configmaps
      verbs:
        - create
    - apiGroups:
        - ""
      resources:
        - endpoints
      verbs:
        - get
  
  ---
  apiVersion: rbac.authorization.k8s.io/v1beta1
  kind: RoleBinding
  metadata:
    name: nginx-ingress-role-nisa-binding
    namespace: ingress-nginx
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: nginx-ingress-role
  subjects:
    - kind: ServiceAccount
      name: nginx-ingress-serviceaccount
      namespace: ingress-nginx
  
  ---
  apiVersion: rbac.authorization.k8s.io/v1beta1
  kind: ClusterRoleBinding
  metadata:
    name: nginx-ingress-clusterrole-nisa-binding
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: nginx-ingress-clusterrole
  subjects:
    - kind: ServiceAccount
      name: nginx-ingress-serviceaccount
      namespace: ingress-nginx
  
  ---
  ```

  关于 `RBAC` 相关的内容，可查看第 8 节 《安全重点: 认证和授权》，了解此处的配置及其含义。

- 部署 NGINX Ingress Controller

  ```
  apiVersion: extensions/v1beta1
  kind: Deployment
  metadata:
    name: nginx-ingress-controller
    namespace: ingress-nginx
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  spec:
    replicas: 1
    selector:
      matchLabels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
    template:
      metadata:
        labels:
          app.kubernetes.io/name: ingress-nginx
          app.kubernetes.io/part-of: ingress-nginx
        annotations:
          prometheus.io/port: "10254"
          prometheus.io/scrape: "true"
      spec:
        serviceAccountName: nginx-ingress-serviceaccount
        containers:
          - name: nginx-ingress-controller
            image: taobeier/nginx-ingress-controller:0.21.0
            args:
              - /nginx-ingress-controller
              - --configmap=$(POD_NAMESPACE)/nginx-configuration
              - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
              - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
              - --publish-service=$(POD_NAMESPACE)/ingress-nginx
              - --annotations-prefix=nginx.ingress.kubernetes.io
            securityContext:
              capabilities:
                drop:
                  - ALL
                add:
                  - NET_BIND_SERVICE
              # www-data -> 33
              runAsUser: 33
            env:
              - name: POD_NAME
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.name
              - name: POD_NAMESPACE
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.namespace
            ports:
              - name: http
                containerPort: 80
              - name: https
                containerPort: 443
            livenessProbe:
              failureThreshold: 3
              httpGet:
                path: /healthz
                port: 10254
                scheme: HTTP
              initialDelaySeconds: 10
              periodSeconds: 10
              successThreshold: 1
              timeoutSeconds: 1
            readinessProbe:
              failureThreshold: 3
              httpGet:
                path: /healthz
                port: 10254
                scheme: HTTP
              periodSeconds: 10
              successThreshold: 1
              timeoutSeconds: 1
  ```

  注意，这里的镜像是我从官方镜像直接同步的，为了解决国内无法下载镜像的情况。

  另外，在启动参数中，指定了我们第二步中创建的 `ConfigMap` 。以及，在此部署中，用到了之前尚未详细说明的 `readinessProbe` 和 `livenessProbe`：我们之前在详解 `kubelet` 时，大致提到过关于它所具备的职责，这两个配置主要是用于做探针，用户检查 Pod 是否已经准备好接受请求流量和是否存活。

  这里还进行了 `annotations` 里面标注了关于 `Prometheus` 的相关内容，我们会在下节中描述。

  ```
  master $ kubectl -n ingress-nginx get all
  NAME                                            READY     STATUS    RESTARTS   AGE
  pod/nginx-ingress-controller-6f647f7866-659ph   1/1       Running   0          75s
  
  NAME                                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/nginx-ingress-controller   1         1         1            1           75s
  
  NAME                                                  DESIRED   CURRENT   READY     AGE
  replicaset.apps/nginx-ingress-controller-6f647f7866   1         1         1         75s
  ```

  可以看到 NGINX Ingress Controller 已经部署成功。

- **将 NGINX Ingress Controller 暴露至集群外**

  经过前面的介绍，我们已经知道 Ingress 的作用在于将集群外的请求流量转向集群内的服务，而我们知道，默认情况下集群外和集群内是不互通的，所以必须将 NGINX Ingress Controller 暴露至集群外，以便让其能接受来自集群外的请求。

  将其暴露的方式有很多种，这里我们选择我们之前已经介绍过的 `NodePort` 的方式。选择它主要有以下原因：

  - 我们可以使用纯的 LB 实现完成服务暴露，比如 [MetalLB](https://metallb.universe.tf/)，但它还处于 Beta 阶段，尚未有大规模生产环境使用的验证。
  - 我们可以直接使用宿主机的网络，只需设置 `hostNetwork: true` 即可，但这个方式可能会带来安全问题。
  - 我们可以选择 External IPs 的方式，但这种方式无法保留请求的源 IP，所以并不是很好。
  - 其实我们一般会选择自己提供边缘节点的方式，不过这种方式是建立在 `NodePort` 的方式之上，并且需要提供额外的组件，此处就暂不做展开了。

  我们使用以下的配置，将 NGINX Ingress Controller 暴露至集群外

  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: ingress-nginx
    namespace: ingress-nginx
    labels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  spec:
    type: NodePort
    ports:
      - name: http
        port: 80
        targetPort: 80
        protocol: TCP
      - name: https
        port: 443
        targetPort: 443
        protocol: TCP
    selector:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  ```

  创建该 `Service`。

  ```
  master $ kubectl -n ingress-nginx get svc                                                  
  NAME            TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
  ingress-nginx   NodePort   10.0.38.53   <none>        80:30871/TCP,443:30356/TCP   11s
  ```

  现在，我们直接访问 `Node:Port` 便可访问到该 Controller。

  ```
  master $ curl 172.17.0.3:30871                    
  <html>
  <head><title>404 Not Found</title></head>
  <body>
  <center><h1>404 Not Found</h1></center>
  <hr><center>nginx/1.15.6</center>
  </body>
  </html>
  ```

  由于我们并没有设置任何的默认响应后端，所以当直接请求时，便返回 404 。

### 实践

将我们的示例项目 [SayThx](https://github.com/tao12345666333/saythx) 通过 `Ingress` 的方式进行访问。

该示例项目的部署，不再进行赘述。可在 [ingress 分支](https://github.com/tao12345666333/saythx/blob/ingress/deploy/ingress.yaml) 查看此处所需配置。

在我们将 NGINX Ingress Controller 及 SayThx 项目部署好之后，我们使用以下的配置创建 `Ingress` 资源。

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: saythx-ing
  namespace: work
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - host: saythx.moelove.info
    http:
      paths:
      - path: /
        backend:
          serviceName: saythx-frontend
          servicePort: 80
```

- 创建

  ```
  master $ kubectl apply -f ingress.yaml         
  ingress.extensions/saythx-ing created
  master $ kubectl -n work get ing
  NAME         HOSTS                 ADDRESS   PORTS     AGE
  saythx-ing   saythx.moelove.info             80        23s
  ```

- 验证

  这里来解释下刚才的配置文件。首先，指定了 `host: saythx.moelove.info` 表示我们想要以 `saythx.moelove.info` 这个域名来访问它。`path` 直接写 `/` 表示所有的请求都转发至名为 `saythx-frontend` 的服务。

  与我们平时使用 NGINX 基本一致。 现在编辑本地的 HOSTS 文件绑定 Node 的IP 与 `saythx.moelove.info` 这个域名。使用浏览器进行访问 `saythx.moelove.info:刚才 Controller 使用 NodePort 暴露服务时的端口`：



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="586"></svg>)



```
可以看到已经成功访问。
```

## 总结

在本节中，我们介绍了 `Ingress` 的基本情况，了解了它是 K8S 中的一种资源对象，主要负责将集群外部流量与集群内服务的通信。但它的正常工作离不开 `Ingress Controller` ，当前官方团队维护的主要有两个 GLBC 和 NGINX Ingress Controller。

我们大致介绍了现有的 Controller 实现，也实践了如何部署 NGINX Ingress Controller 以及如何使用 Ingress 将我们的示例项目暴露至集群外。

NGINX Ingress Controller 的使用，比较符合我们平时使用 NGINX 的习惯，相对来说也比较灵活，后续可看实际情况再进行更多的实践。

至此，K8S 集群的管理，相关原理以及服务的部署等内容就基本介绍完毕。下节，我们将介绍生产实践中至关重要的一环 **监控** 相关的实践。

# 监控实践：对 K8S 集群进行监控

## 整体概览

通过前面的学习，我们对 K8S 有了一定的了解，也具备了一定的集群管理和排错能力。但如果要应用于生产环境中，不可能随时随地的都盯着集群，我们需要扩展我们对集群的感知能力。

本节，我们将介绍下 K8S 集群监控相关的内容。

## 监控什么

除去 K8S 外，我们平时自己开发的系统或者负责的项目，一般都是有监控的。监控可以提升我们的感知能力，便于我们及时了解集群的变化，以及知道哪里出现了问题。

K8S 是一个典型的分布式系统，组件很多，那么监控的目标，就变的很重要了。

总体来讲，对 K8S 集群的监控的话，主要有以下方面：

- 节点情况
- K8S 集群自身状态
- 部署在 K8S 内的应用的状态

## Prometheus

对于 K8S 的监控，我们选择 CNCF 旗下次于 K8S 毕业的项目[ Prometheus ](https://prometheus.io/)。

Prometheus 是一个非常灵活易于扩展的监控系统，它通过各种 `exporter` 暴露数据，并由 `prometheus server` 定时去拉数据，然后存储。

它自己提供了一个简单的前端界面，可在其中使用 [PromQL ](https://prometheus.io/docs/prometheus/latest/querying/basics/)的语法进行查询，并进行图形化展示。

## 安装 Prometheus

> 这里推荐一个项目 [Prometheus Operator](https://github.com/coreos/prometheus-operator), 尽管该项目还处于 Beta 阶段，但是它给在 K8S 中搭建基于 Prometheus 的监控提供了很大的便利。

我们此处选择以一般的方式进行部署，带你了解其整体的过程。

- 创建一个独立的 `Namespace`：

  ```
  apiVersion: v1
  kind: Namespace
  metadata:
    name: monitoring
  
  # 将文件保存为 namespace.yaml 的文件，并执行 kubectl apply -f namespace.yaml 即可，后面不再赘述。
  ```

  ```
  master $ kubectl apply -f namespace.yaml
  namespace/monitoring created
  ```

- RBAC

  我们的集群使用 `kubeadm` 创建，默认开启了 `RBAC`，所以现在需要创建相关的 Role 和 binding。

  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: prometheus
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: prometheus
  subjects:
  - kind: ServiceAccount
    name: prometheus-k8s
    namespace: monitoring
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: prometheus
  rules:
  - apiGroups: [""]
    resources:
    - nodes
    - nodes/proxy
    - services
    - endpoints
    - pods
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources:
    - configmaps
    verbs: ["get"]
  - nonResourceURLs: ["/metrics"]
    verbs: ["get"]
  ---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: prometheus-k8s
    namespace: monitoring
  ```

  执行创建

  ```
  master $ kubectl  apply -f rbac.yaml
  clusterrolebinding.rbac.authorization.k8s.io/prometheus created
  clusterrole.rbac.authorization.k8s.io/prometheus created
  serviceaccount/prometheus-k8s created
  ```

- 创建 Promethes 的配置文件

  其中的内容主要参考 [Prometheus 官方提供的示例](https://github.com/prometheus/prometheus/blob/master/documentation/examples/prometheus-kubernetes.yml) 和 [Prometheus 官方文档](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config)。

  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: prometheus-core
    namespace: monitoring
  data:
    prometheus.yaml: |
      global:
        scrape_interval: 30s
        scrape_timeout: 30s
      scrape_configs:
      - job_name: 'kubernetes-apiservers'
      
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
      
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
      
      # Scrape config for nodes (kubelet).
      - job_name: 'kubernetes-nodes'
        scheme: https
      
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      
        kubernetes_sd_configs:
        - role: node
      
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics
      
      # Scrape config for Kubelet cAdvisor.
      - job_name: 'kubernetes-cadvisor'
      
        scheme: https
      
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      
        kubernetes_sd_configs:
        - role: node
      
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
      
      - job_name: 'kubernetes-service-endpoints'
      
        kubernetes_sd_configs:
        - role: endpoints
      
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          action: replace
          target_label: kubernetes_name
      
      - job_name: 'kubernetes-services'
      
        metrics_path: /probe
        params:
          module: [http_2xx]
      
        kubernetes_sd_configs:
        - role: service
      
        relabel_configs:
        - source_labels: [__address__]
          target_label: __param_target
        - target_label: __address__
          replacement: blackbox-exporter.example.com:9115
        - source_labels: [__param_target]
          target_label: instance
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          target_label: kubernetes_name
      
      - job_name: 'kubernetes-ingresses'
      
        metrics_path: /probe
        params:
          module: [http_2xx]
      
        kubernetes_sd_configs:
        - role: ingress
      
        relabel_configs:
        - source_labels: [__meta_kubernetes_ingress_scheme,__address__,__meta_kubernetes_ingress_path]
          regex: (.+);(.+);(.+)
          replacement: ${1}://${2}${3}
          target_label: __param_target
        - target_label: __address__
          replacement: blackbox-exporter.example.com:9115
        - source_labels: [__param_target]
          target_label: instance
        - action: labelmap
          regex: __meta_kubernetes_ingress_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_ingress_name]
          target_label: kubernetes_name
      
      - job_name: 'kubernetes-pods'
      
        kubernetes_sd_configs:
        - role: pod
      
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_pod_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_pod_name]
          action: replace
          target_label: kubernetes_pod_name
  ```

- 部署 Prometheus

  ```
  apiVersion: extensions/v1beta1
  kind: Deployment
  metadata:
    name: prometheus-core
    namespace: monitoring
    labels:
      app: prometheus
      component: core
  spec:
    replicas: 1
    template:
      metadata:
        name: prometheus-main
        labels:
          app: prometheus
          component: core
      spec:
        serviceAccountName: prometheus-k8s
        containers:
        - name: prometheus
          image: taobeier/prometheus:v2.6.0
          args:
            - '--storage.tsdb.retention=24h'
            - '--storage.tsdb.path=/prometheus'
            - '--config.file=/etc/prometheus/prometheus.yaml'
          ports:
          - name: webui
            containerPort: 9090
          resources:
            requests:
              cpu: 500m
              memory: 500M
            limits:
              cpu: 500m
              memory: 500M
          volumeMounts:
          - name: data
            mountPath: /prometheus
          - name: config-volume
            mountPath: /etc/prometheus
        volumes:
        - name: data
          emptyDir: {}
        - name: config-volume
          configMap:
            name: prometheus-core
  ```

- 查看部署情况

  ```
  master $ kubectl  -n monitoring get all
  NAME                                   READY     STATUS    RESTARTS   AGE
  pod/prometheus-core-86b8455f76-mvrn4   1/1       Running   0          12s
  
  NAME                              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/prometheus-core   1         1         1            1           12s
  
  NAME                                         DESIRED   CURRENT   READY     AGE
  replicaset.apps/prometheus-core-86b8455f76   1         1         1         12s
  ```

  Prometheus 的主体就已经部署完成。

- 使用 `Service` 将 `Promethes` 的服务暴露出来

  ```
  apiVersion: v1
  kind: Service
  metadata:
    labels:
      app: prometheus
      component: core
    name: prometheus
    namespace: monitoring
  spec:
    ports:
    - protocol: TCP
      port: 9090
      targetPort: 9090
    selector:
      app: prometheus
      component: core
    type: NodePort
  ```

  这里为了方便演示，直接使用了 `NodePort` 的方式暴露服务。当然你也可以参考上一节，使用 `Ingress` 的方式将服务暴露出来。

- 查询当前状态

  我们使用 Promethes 自带的 PromQL 语法，查询在当前 `monitoring` Namespace 中 up 的任务。这里对查询的结果暂不进行展开。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1275" height="766"></svg>)



## 安装 Node exporter

我们刚才在介绍时，提到过 `Promethes` 支持多种 `exporter` 暴露指标。我们现在使用 [Node exporter](https://github.com/prometheus/node_exporter) 完成对集群中机器的基础监控。

这里有一个需要考虑的点：

- 使用什么方式部署 Node exporter ？

  Node exporter 有已经编译好的二进制文件，可以很方便的进行部署。当我们要监控集群中所有的机器时，我们是该将它直接部署在机器上，还是部署在集群内？

  我建议是直接部署在集群内，使用 `DaemonSet` 的方式进行部署。这里的考虑是当我们直接部署在宿主机上时，我们最起码需要保证两点：1. Promethes 服务可与它正常通信（Promethes 采用 Pull 的方式采集数据） ；2. 需要服务保活，如果 exporter 挂掉了，那自然就取不到数据。

  `DaemonSet` 是一种很合适的的部署方式，可直接将 Node exporter 部署至集群的每个节点上。

- 创建 `DaemonSet`

  ```
  apiVersion: extensions/v1beta1
  kind: DaemonSet
  metadata:
    name: prometheus-node-exporter
    namespace: monitoring
    labels:
      app: prometheus
      component: node-exporter
  spec:
    template:
      metadata:
        name: prometheus-node-exporter
        labels:
          app: prometheus
          component: node-exporter
      spec:
        tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
        containers:
        - image: taobeier/node-exporter:v0.17.0
          name: prometheus-node-exporter
          ports:
          - name: prom-node-exp
            containerPort: 9100
            hostPort: 9100
        hostNetwork: true
        hostPID: true
  ```

- 让 Promethes 抓取数据

  ```
  apiVersion: v1
  kind: Service
  metadata:
    annotations:
      prometheus.io/scrape: 'true'
    name: prometheus-node-exporter
    namespace: monitoring
    labels:
      app: prometheus
      component: node-exporter
  spec:
    clusterIP: None
    ports:
      - name: prometheus-node-exporter
        port: 9100
        protocol: TCP
    selector:
      app: prometheus
      component: node-exporter
    type: ClusterIP
  ```

  这里我们直接使用了添加 `annotations` 的方式，让 Promethes 自动的通过 Kubernetes SD 发现我们新添加的 exporter （或者说资源）

  我们访问 Promethes 的 web 端，进行验证。



![img](https://user-gold-cdn.xitu.io/2018/12/24/167df6106b58b2fb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 总结

在本节中，我们介绍了 `Prometheus` 的基本情况，也部署了 `Prometheus` 的主体服务。

但这是结束么？这并不是，这才刚刚开始。

我们提到 `Prometheus` 支持多种 `exporter` 暴露各种指标，而且我们还可以使用 [Grafana](https://grafana.com/) 作为我们监控的展示手段。

如果要做 Dashboard 推荐使用 [Kubernetes cluster monitoring (via Prometheus)](https://grafana.com/dashboards/162) 。

另外，监控其实涉及的内容很多，包括数据持久化方式。以及是否考虑与集群外的 Prometheus 集群做邦联模式等。这里需要考虑的实际情况较多，暂不一一展开了。

Prometheus 已经从 CNCF 毕业，其在云原生时代下作为标准的监控技术栈也基本确立。至于应用监控，也可使用它的 SDK 来完成。

下节，我们将对本小册进行一次总结。

# 总结

## 快速回顾

经过了前面 23 节的内容，我们从 K8S 的基础概念入手，通过其基础架构了解到了 K8S 中所涉及到的各类组件。

通过动手实践，使用 `minikube` 搭建了本地的集群，使用 `kubeadm` 完成了服务器上的集群搭建，对 K8S 的部署有了更加清晰的认识。

这里再推荐另一种正在快速迭代的方式 [Kubernetes In Docker](https://github.com/kubernetes-sigs/kind) 可以很方便的创建廉价的 K8S 集群，目前至支持单节点集群，多节点支持正在开发中。

后面，我们通过学习 `kubectl` 的使用，部署了 Redis 服务，了解到了一个服务在 K8S 中部署的操作，以及如何将服务暴露至集群外，以便访问。

当集群真正要被使用之前，权限管控也愈发重要，我们通过学习 `RBAC` 的相关知识，学习到了如何在 K8S 集群中创建权限可控的用户，而这部分的内容在后续小节中也被频繁用到。

接下来，我们以我们实际的一个项目 [SayThx](https://github.com/tao12345666333/saythx) 为例，一步步的完成了项目的部署，在此过程中也学习到了配置文件的编写规范和要求。

当项目变大时，维护项目的更新也变成了一件很麻烦的事情。由此，我们引入了 `Helm` 作为我们的包管理软件，并使用它进行了项目的部署。

在此过程中也学习到了 Helm 的架构，以及如何编写一个 `Chart` 等知识。

前面我们主要集中于如何使用 K8S 上，接下了庖丁解牛系列便带我们一同深入至 K8S 内部，了解到了各基础组件的实际工作原理，也深入到了源码内部，了解其实现逻辑。

有这些理论知识作为基础，我们便可以大胆的将应用部署至 K8S 之上了。但实际环境可能多种多样，你可以会遇到各种各样的问题。

这里我们介绍了一些常见的 Troubleshoot 的方法，以便你在后续使用 K8S 的过程中遇到问题也可以快速的定位并解决问题。

此外，我们学习了 K8S 的一些扩展，比如 Dashboard 和 CoreDNS ， Dashboard 是一个比较直观的管理资源的方式，它也还在快速的发展和迭代中。

CoreDNS 在 K8S 1.13 中已经成为默认的 DNS 服务器，相信在不久之后， CoreDNS 也将会从 CNCF 毕业。

我们介绍了 `Ingress` 和在 K8S 中使用 `Promethes` 进行监控，不过监控涉及的方面很多，除了集群自身的监控外，应用层的监控也同样很重要。另外，监控和告警也是相辅相成的，在已有监控数据的前提下，如何更智能更优雅的告警也是我们需要考虑的点。否则，很容易造成告警风暴，有用的告警被忽略之类的。

## 扩展阅读

基于 K8S 的生态已经在逐步形成，只靠一本小册还远远不够，我们需要更多的对操作系统的了解，对 K8S 及其生态的了解。

以下推荐一些扩展阅读：

- [K8S 生态](https://zhuanlan.zhihu.com/container)
- [K8S 网站](https://kubernetes.io/)
- [CNCF 博客](https://www.cncf.io/newsroom/blog/)
- [K8S 组织](https://github.com/kubernetes/)
- [Docker 文档](https://docs.docker.com/)
- [Promethes 文档](https://prometheus.io/docs/introduction/overview/)
- [Grafana 主页](https://grafana.com/)
- [Fluentd 主页](https://www.fluentd.org/)

## 总结

围绕 K8S 的云原生生态已经逐步形成，希望本小册能在你未来发展道路上起到一定的帮助。

K8S 涉及的知识面很广，小册中篇幅有限未能一一详解，欢迎大家共同讨探。










