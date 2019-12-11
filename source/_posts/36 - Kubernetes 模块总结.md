---
title: Kubernetes 模块总结
date: 2019-12-11 10:14:08
tags:
categories:
thumbnail: https://piccdn.freejishu.com/images/2019/12/11/Pnw9Lb.jpg
---

本文参考 : [Kubernetes核心概念总结](https://www.cnblogs.com/zhenyuyaodidiao/p/6500720.html)

Kubernetes (以下简称 K8s) 由一个主节点 Master 和多个从节点 Node 构成 

![img](https://images2015.cnblogs.com/blog/704717/201703/704717-20170304103633345-155330022.png)

### 1. 基本架构

#### Master

Master 节点由四个模块组成 ，分别是 APIserver, scheduler, manager, etcd 



**APIserver** 负责对外提供 RESTful 的 K8s API 服务 ，是系统管理指令的统一入口 ， 任何对于资源的增删改查都需要交给 APIserver 处理后再交给 etcd 。如架构图所示 ，kubectl（K8s 提供的客户端工具 ，工具内部就是对 K8s API 的调用）是直接和 APIserver 交互的 。



**schedule** 负责调度 pod 到合适的 Node 上 ，schedule 根据 pod 的需求以及 Node 的状态将 pod 部署到合适的 Node 上 。K8s 提供了默认的调度算法 ，也保留了可供自定义调度算法的接口 。



**controller manager** 每一个 pod 资源都对应了一个控制器 ，控制器负责保持 pod 在预期的服务状态， 而 controller manager 负责管理控制器 。



**etcd** 是一个高可用键值存储系统 ，被用于存储各资源状态





#### Node

每个 Node 节点由 kubelet, kube-proxy, runtime 组成



**runtime** 指容器运行环境 ，目前 K8s 支持 docker 和 rkt 两种容器



**kube-proxy** 实现了 K8s 中的服务发现和反向代理功能 。

- 反向代理中 kube-proxy 支持 TCP 和 UDP 连接转发 ，默认基于 Round Robin 算法将客户端流量转发到 service 对应的一组后端 pod
- 服务发现中 kube-proxy 使用 etcd 的 watch 机制 ，监控集群中 service 和 endpoint 对象数据的动态变化 ，并维护一个 service 到 endpoint 的映射关系 ，从而保证了后端 pod 的 IP 变化不会对访问者造成影响 。另外 kube-proxy 还支持 session affinity



**kubelet** 是 Master 在每个 Node 阶段上的代理 (agent) ，是 Node 上最重要的模块 ，负责维护和管理 Node 上的所有容器 ，但它只会管理通过 K8s 创建的容器 ，本质上是 kubelet 负责维护 pod 运行状态与期望状态一致 





### 2. Pod

Pod 是 K8s 的基础操作单元，是应用运行的载体 。K8s 的系统是围绕着 Pod 展开的，不如如何部署运行 Pod ，如何保障 pod 数量 ，如何访问 pod 等 。同时 pod 还是一个或多个容器的集合 ，K8s 提供了一种容器的组合模型



**pod 和 容器** 

在 Docker 中容器是最小的处理单元 ，增删改查的对象是容器 ，容器是一种虚拟化技术，容器之间是相互隔离的 ，隔离基于 Namespace 实现 。在 K8s ，Pod包含一个或多个相关容器 ，可以说 Pod 就是容器的延伸 ，一个 Pod 也是一个隔离体 ，而 Pod 内部包含的容器又是共享的 (PID, Network, IPC, UTS) 。同时 Pod 中的容器可以访问共同的数据卷来实现文件系统的共享 。



**镜像** 

在kubernetes中，镜像的下载策略为：

-　　　 **Always**：每次都下载最新的镜像

-　　　 **Never**：只使用本地镜像，从不下载

-　　　 **IfNotPresent**：只有当本地没有的时候才下载镜像

　　Pod被分配到Node之后会根据镜像下载策略进行镜像下载，可以根据自身集群的特点来决定采用何种下载策略。无论何种策略，都要确保Node上有正确的镜像可用。



**Pod 的生命周期**

Pod 被分配到一个 Node 上之后 ，知道被删除前不会离开这个 Node 。当某个 Pod 失败 ，首先会被Kubernetes 清理掉 ，之后 ReplicationController 将会在其它机器上（或本机）重建 Pod ，重建之后Pod 的 ID 发生了变化 ，那将会是一个新的 Pod 。所以，Kubernetes 中 Pod 的迁移，实际指的是在新 Node 上重建 Pod 。以下给出 Pod 的生命周期图 。

![img](https://images2015.cnblogs.com/blog/704717/201703/704717-20170304103856954-1091445106.png)



Pod 的生命周期函数有两个 ，一个是 PostStart (容器创建成功后调用) , PreStop ( 容器终止前调用 ) 





### 3. Replication Controller

Replication Controller (Rc) 是 K8s 的另一个核心概念 ，RC 保证应用在托管在 K8s 之后能保证持续运行 ，它会确保任何时间 K8s 中都有指定的 Pod 在运行 。此外还提供了滚动升级 ，升级回滚等高级功能 。



**Label** 

label 是 RC 和 Pod 关联的枢纽 ，通过 Label 实现的弱关联可以灵活地进行分类和选择 。对于 Pod ，需要设置自身的 Label 来进行辨识 ，Label 是一些列的键值对 ，在Pod-->metadata-->labeks中进行设置 。

label 可以任意定义 ，但我们尽量要保证 label 具有可读性 ，如使用 Pod 的应用名称和版本号等 。另外 Label 应当是唯一的 ，为了准确标识一个 Pod 可以为 Pod 设置多个维度的 Label ，例如 :

```
"release" : "stable", "release" : "canary"
"environment" : "dev", "environment" : "qa", "environment" : "production"
"tier" : "frontend", "tier" : "backend", "tier" : "cache"
"partition" : "customerA", "partition" : "customerB"
"track" : "daily", "track" : "weekly"
```



**弹性伸缩** 

弹性伸缩指的是适应负载变化 ，以弹性可伸缩的方式提供资源 ，反映在 K8s 中指的是可以根据负载的高低动态调整 Pod 的副本数量 。调整 Pod 是根据修改 Rc 中的 Pod 副本来实现的 



**滚动升级**

滚动升级是一种平滑过渡的升级方式 ，通过逐步替换的策略 ，保证整体系统的稳定 ，在初始升级的时候就可以发现 ，调整问题 ，以保证问题的影响度不会扩大 。

升级开始时， K8s 根据提供的文件创建 V2 版本的 RC ，然后每隔一段时间(--update-period = 10s) 逐步增加 V2 版本 Pod 数量并减少 V1 版本副本数量 。在升级完成之后保留 V2 版本的 RC ，删除 V1 版本的 RC ，滚动升级完成 。

如果在升级过程中发生了错误而导致中途退出时，可以选择继续升级 ，K8s 能够智能判断升级中断之前的状态 ，让后紧接着执行升级 ，也可以中断这一次升级 ，进行回退 。

回退的本质就是进行滚动升级的逆操作 ，逐步增加 V1 版本的 Pod 数量 ，逐步减少 V2 版本的 Pod 数量 。



**副本控制器 replica set** 

replica set 可以被认为是 “升级版” RC ，replica set 也被用于保证与 label selector 匹配的 pod 数量维持在期望状态 。

二者的区别在于 ，replica set 引入了对于基于子集的 selector 查询条件 ，而 Replication Controller 仅支持基于值相等的 selector 条件查询 。这是从操作者的角度来看二者的唯一显著差异 。引入这一 API 的初衷是用于取代 VL 中的 Replication Controller ，也就是说 ，当 V1 版本被废弃时 ，Replication Controller 就完成了他的使命 ，而由 replica set 来接管其工作 。虽然 replica set 可以被单独使用 ，但目前多被用于 Deployment 用于进行 Pod 的创建更新与删除 。





### 4. Job

从程序的运行形态上来区分，我们可以将 Pod 分为两类：长时运行服务 (jboss, mysql 等)  和一次性任务 (数据计算、测试) 。RC 创建的 Pod 都是长时运行的服务 ，而 Job 创建的 Pod 都是一次性任务 。

　　在 Job 的定义中 ， restartPolicy (重启策略) 只能是 Never 和 OnFailure 。 Job 可以控制一次性任务的 Pod 的完成次数 (Job-->spec-->completions) 和并发执行数 (Job-->spec-->parallelism) ，当 Pod 成功执行指定次数后 ，即认为 Job 执行完毕 。





### 5. Service 

为了适应快速的业务需求 ，微服务架构已经逐渐成为主流 ，微服务架构的应用需要有非常好的服务编排支持 。Kubernetes 中的核心要素 Service 便提供了一套简化的服务代理和发现机制 ，天然适应微服务架构 。



**原理**

在 Kubernetes 中 ，在受到 RC 调控的时候 ，Pod 副本是变化的 ，对于的虚拟 IP 也是变化的 ，比如发生迁移或者伸缩的时候 ，这对于 Pod 的访问者来说是不可接受的 。Kubernetes 中的 Service 是一种抽象概念 ，它定义了一个 Pod 逻辑集合以及访问它们的策略，Service 同 Pod 的关联同样是居于 Label 来完成的 。Service 的目标是提供一种桥梁 ，它会为访问者提供一个固定访问地址 ，用于在访问时重定向到相应的后端 ，这使得非 Kubernetes 原生应用程序 ，在无须为 Kubemces 编写特定代码的前提下 ，轻松访问后端 。

　　Service 同 RC 一样 ，都是通过 Label 来关联 Pod 的 。当你在 Service 的 yaml 文件中定义了该 Service 的 selector 中的 label 为 app:my-web ，那么这个 Service 会将 Pod-->metadata-->labeks 中 label 为 app:my-web 的 Pod 作为分发请求的后端 。当 Pod 发生变化时（增加、减少、重建等）， Service 会及时更新 。这样一来 ， Service 就可以作为 Pod 的访问入口 ，起到代理服务器的作用 ，而对于访问者来说 ，通过 Service 进行访问 ，无需直接感知 Pod 。

　　需要注意的是 ，Kubernetes 分配给 Service 的固定 IP 是一个虚拟 IP ，并不是一个真实的 IP ，在外部是无法寻址的 。真实的系统实现上 ，Kubernetes 是通过 Kube-proxy 组件来实现的虚拟IP路由及转发 。所以在之前集群部署的环节上，我们在每个 Node 上均部署了 Proxy 这个组件 ，从而实现了 Kubernetes 层级的虚拟转发网络 。



**service 内部负载均衡**

当 Service 的 EndPoints 包含多个 IP 时 ，且服务器代理存在多个后端 ，将进行请求的负载均衡 。默认的负载均衡策略是轮询或者随机 (取决于 kube-proxy的模式) 。同时 Service 通过设置 sessionAffinity=ClientIP 来实现基于源 IP 地址的会话保持 。



**service 发布**

Service 的虚拟 IP 是由 K8s 虚拟出来的内部网络 ，外部是无法寻址到的 。但是有些服务 ，例如web服务又需要被外部访问到 ，这时候就需要加一层网络转发  。Kubernetes 提供了 NodePort ， LoadBalancer， Ingress 三种方式

- **NodePort**  NodePort的原理是，Kubernetes会在每一个Node上暴露出一个端口 :nodePort ，外部网络可以通过（任一Node）[NodeIP]:[NodePort] 访问到后端的 Service

- **LoadBalancer**  在 NodePort 基础上 ，Kubernetes 可以请求底层云平台创建一个负载均衡器，将每个 Node 作为后端，进行服务分发。该模式需要底层云平台（例如 GCE ）支持

- **Ingress**  是一种 HTTP 方式的路由转发机制，由 Ingress Controller 和 HTTP 代理服务器组合而成。Ingress Controller 实时监控 Kubernetes API ，实时更新 HTTP 代理服务器的转发规则。HTTP 代理服务器有 GCE Load-Balancer 、HaProxy 、Nginx 等开源方案



**service 自发性机制**

K8s 中有一个很重要的服务自发现特性 。一旦一个 service 被创建 ，该 service 的 service IP 和 service port 等信息都可以被注入到 pod 中供它们使用 。Kubernetes 主要支持两种 service 发现机制 :环境变量和 DNS

- **环境变量方式**

　　Kubernetes 创建 Pod 时会自动添加所有可用的 service 环境变量到该 Pod 中 。如有需要 ，这些环境变量就被注入 Pod 内的容器里 。需要注意的是 ，环境变量的注入只发送在 Pod 创建时 ，**且不会被自动更新**。这个特点暗含了 service 和访问该 servic e的 Pod 的创建时间的先后顺序 ，即任何想要访问 service 的 pod 都需要在 service 已经存在后创建 ，否则与 service 相关的环境变量就无法注入该 Pod 的容器中 ，这样先创建的容器就无法发现后创建的 service 。

- **DNS 方式**

Kubernetes 集群现在支持增加一个可选的组件 —— DNS 服务器 。这个 DNS 服务器使用 Kubernetes 的 watchAPI ，不间断的监测新的 service 的创建并为每个 service 新建一个 DNS 记录。如果 DNS 在整个集群范围内都可用 ，那么所有的 Pod 都能够自动解析 service 的域名 。 



**多个 service 如何避免地址和端口冲突**

此处设计思想是 ，Kubernetes 通过为每个 service 分配一个唯一的 ClusterIP ，所以当使用 ClusterIP:port 的组合访问一个 service 的时候 ，不管 port 是什么 ，这个组合是一定不会发生重复的 。另一方面 ，kube-proxy 为每个 service 真正打开的是一个绝对不会重复的随机端口 ，用户在 service 描述文件中指定的访问端口会被映射到这个随机端口上 。这就是为什么用户可以在创建 service 时随意指定访问端口



**service目前存在的不足**

Kubernetes 使用 iptables 和 kube-proxy 解析 service 的入口地址 ，在中小规模的集群中运行良好 ，但是当 service 的数量超过一定规模时 ，仍然有一些小问题 。首当其冲的便是 service 环境变量泛滥 ，以及 service 与使用 service 的 pod 两者创建时间先后的制约关系 。目前来看 ，很多使用者在使用 Kubernetes 时往往会开发一套自己的 Router 组件来替代 service ，以便更好地掌控和定制这部分功能





### 6. Deployment

Deployment 是一种 k8s 用于更新 RC 和 Pod 的机制 。在 Deployment 中描述期望的集群状态之后 Deployment Controller 会将现有的集群状态在可控的速度下逐步更新为期望的集群状态 ，Deployment 主要职责同样是为了保证 pod 的数量和健康，90% 的功能与 Replication Controller 完全一样，可以看做新一代的 Replication Controller ，带来了以下新特性 ：

- **Replication Controller 全部功能**：Deployment 继承了 Replication Controller 全部功能

- **事件和状态查看**：可以查看 Deployment 的升级详细进度和状态

- **回滚**：当升级 pod 镜像或者相关参数的时候发现问题，可以使用回滚操作回滚到上一个稳定的版本或者指定的版本

- **版本记录**: 每一次对 Deployment 的操作，都能保存下来，给予后续可能的回滚使用

- **暂停和启动**：对于每一次升级，都能够随时暂停和启动

- **多种升级方案**：
    - **Recreate**  重新创建所有 Pod
    - **RollingUpdate**  滚动升级 ，支持更多的附加参数 (最大不可用pod数量，最小升级间隔时间等)



Deployment 还支持多重升级 ，即可以再升级中途修改升级要求 。举例 ，你创建了一个 nginx1.7 的 Deployment ，要求副本数量为 5 ，Deployment Controller 会逐步的将 5 个 1.7 的 Pod 启动起来 。当启动到 3 个的时候 ，你又发出更新 Deployment 中 Nginx 到 1.9 的命令，这时 Deployment Controller 会立即将已启动的 3 个 1.7 Pod 杀掉 ，然后逐步启动 1.9 的 Pod 。Deployment Controller 不会等到 1.7 的 Pod 都启动完成之后，再依次杀掉 1.7，启动 1.9 





### 7. Volume

在 Docker 的设计实现中 ，容器中的数据是临时的 ，即当容器被销毁时 ，其中的数据将会丢失 。如果需要持久化数据 ，需要使用 Docker 数据卷挂载宿主机上的文件或者目录到容器中 。在 K8s 中 ，当 Pod 重建的时候 ，数据是会丢失的 ，K8s 也是通过数据卷挂载来提供 Pod 数据的持久化的。K8s 数据卷是对 Docker 数据卷的扩展 ，K8s 数据卷是 Pod 级别的 ，可以用来实现 Pod 中容器的文件共享 。目前 ，K8s 支持的数据卷类型如下：

| 支持数据类型            |
| ----------------------- |
| EmptyDir                |
| HostPath                |
| GCE Persistent Disk     |
| AWS Elastic Block Store |
| NFS                     |
| iSCSI                   |
| Flocker                 |
| GlusterFS               |
| RBD                     |
| Git Repo                |
| Secret                  |
| Persistent Volume Claim |
| Downward API            |



**本地数据卷**

EmptyDir ，HostPath 这两种类型的数据卷，只能最用于本地文件系统 。本地数据卷中的数据只会存在于一台机器上 ，所以当Pod发生迁移的时候，数据便会丢失 。该类型 Volume 的用途是 ：Pod 中容器间的文件共享、共享宿主机的文件系统 

- **EmptyDir**

如果 Pod 配置了 EmpyDir 数据卷 ，在 Pod 的生命周期内都会存在 ，当 Pod 被分配到 Node 上的时候 ，会在 Node 上创建 EmptyDir 数据卷 ，并挂载到 Pod 的容器中 。只要 Pod 存在，EmpyDir 数据卷都会存在（容器删除不会导致 EmpyDir 数据卷丟失数据），但是如果 Pod 的生命周期终结（Pod被删除），EmpyDir 数据卷也会被删除 ，并且永久丢失。

EmpyDir 数据卷非常适合实现 Pod 中容器的文件共享 。Pod 的设计提供了一个很好的容器组合的模型 ，容器之间各司其职 ，通过共享文件目录来完成交互 ，比如可以通过一个专职日志收集容器 ，在每个 Pod 中和业务容器中进行组合 ，来完成日志的收集和汇总 

- **HostPath**

HostPath 数据卷允许将容器宿主机上的文件系统挂载到 Pod 中。如果 Pod 需要使用宿主机上的某些文件 ，可以使用 HostPath 



**网络数据卷**

Kubernetes 提供了很多类型的数据卷以集成第三方的存储系统 ，包括一些非常流行的分布式文件系统，也有在 IaaS 平台上提供的存储支持 ，这些存储系统都是分布式的 ，通过网络共享文件系统 ，因此我们称这一类数据卷为网络数据卷 。

网络数据卷能够满足数据的持久化需求 ，Pod 通过配置使用网络数据卷 ，每次 Pod 创建的时候都会将存储系统的远端文件目录挂载到容器中 ，数据卷中的数据将被水久保存 ，即使 Pod 被删除 ，只是除去挂载数据卷 ，数据卷中的数据仍然保存在存储系统中 ，且当新的 Pod 被创建的时候 ，仍是挂载同样的数据卷 。网络数据卷包含以下几种：NFS、iSCISI、GlusterFS、RBD（Ceph Block Device）、Flocker、AWS Elastic Block Store、GCE Persistent Disk



**Persistent Volume 和 Persistent Volume Claim**
理解每个存储系统是一件复杂的事情 ，特别是对于普通用户来说 ，有时候并不需要关心各种存储实现 ，只希望能够安全可靠地存储数据 。Kubernetes 中提供了 Persistent Volume 和 Persistent Volume Claim 机制 ，这是存储消费模式 。Persistent Volume 是由系统管理员配置创建的一个数据卷（目前支持 HostPath 、GCE Persistent Disk、AWS Elastic Block Store、NFS、iSCSI、GlusterFS、RBD），它代表了某一类存储插件实现 。而对于普通用户来说 ，通过 Persistent Volume Claim 可请求并获得合适的 Persistent Volume ，而无须感知后端的存储实现 。Persistent Volume 和 Persistent Volume Claim 的关系其实类似于 Pod 和 Node ，Pod 消费 Node 资源，Persistent Volume Claim 则消费 Persistent Volume 资源。Persistent Volume 和 Persistent Volume Claim 相互关联，有着完整的生命周期管理：

- **准备**：系统管理员规划或创建一批 Persistent Volume

- **绑定**：用户通过创建 Persistent Volume Claim 来声明存储请求 ，Kubernetes 发现有存储请求的时候 ，就去查找符合条件的 Persistent Volume（最小满足策略）。找到合适的就绑定上 ，找不到就一直处于等待状态

- **使用**：创建 Pod 的时候使用 Persistent Volume Claim 

- **释放**：当用户删除绑定在 Persistent Volume 上的 Persistent Volume Claim 时 ， Persistent Volume 进入释放状态 ，此时 Persistent Volume 中还残留着上一个 Persistent Volume Claim 的数据 ，状态还不可用

- **回收**：Persistent Volume 需要回收才能再次使用 。回收策略可以是人工的也可以是 Kubernetes 自动进行清理（仅支持 NFS 和 HostPath ）



**信息数据卷**
Kubernetes 中有一些数据卷 ，主要用来给容器传递配置信息 ，我们称之为信息数据卷 ，比如Secret（处理敏感配置信息，密码、Token等）、Downward API（通过环境变量的方式告诉容器Pod的信息）、Git Repo（将Git仓库下载到Pod中），都是将 Pod 的信息以文件形式保存 ，然后以数据卷方式挂载到容器中 ，容器通过读取文件获取相应的信息 。





### 8. Pet Sets/StatefulSet

K8s 在 1.3 版本里发布了 Alpha 版的 PetSet 功能 。在云原生应用的体系里 ，有下面两组近义词；第一组是无状态（stateless）、牲畜（cattle）、无名（nameless）、可丢弃（disposable）；第二组是有状态（stateful）、宠物（pet）、有名（having name）、不可丢弃（non-disposable）。RC 和 RS 主要是控制提供无状态服务的 ，其所控制的 Pod 的名字是随机设置的 ，一个 Pod 出故障了就被丢弃掉 ，在另一个地方重启一个新的 Pod ，名字变了、名字和启动在哪儿都不重要，重要的只是 Pod 总数 。 而 PetSet 是用来控制有状态服务，PetSet 中的每个 Pod 的名字都是事先确定的，不能更改。PetSet 中 Pod 的名字的作用，是用来关联与该 Pod 对应的状态 。

对于 RC 和 RS 中的 Pod ，一般不挂载存储或者挂载共享存储 ，保存的是所有 Pod 共享的状态 ， Pod 像牲畜一样没有分别 。对于 PetSet 中的 Pod ，每个 Pod 挂载自己独立的存储 ，如果一个 Pod 出现故障 ，从其他节点启动一个同样名字的 Pod ，要挂在上原来 Pod 的存储继续以它的状态提供服务 。

适合于 PetSet 的业务包括数据库服务 MySQL 和 PostgreSQL ，集群化管理服务 Zookeeper、etcd 等有状态服务 。 PetSet 的另一种典型应用场景是作为一种比普通容器更稳定可靠的模拟虚拟机的机制 。传统的虚拟机正是一种有状态的宠物 ，运维人员需要不断地维护它 ，容器刚开始流行时 ，我们用容器来模拟虚拟机使用 ，所有状态都保存在容器里 ，而这已被证明是非常不安全 、不可靠的 。使用 PetSet ，Pod 仍然可以通过漂移到不同节点提供高可用 ，而存储也可以通过外挂的存储来提供高可靠性 ， PetSet 做的只是将确定的 Pod 与确定的存储关联起来保证状态的连续性 。





 ### 9. ConfigMap

很多生产环境中的应用程序配置较为复杂 ，可能需要多个 config 文件、命令行参数和环境变量的组合 。并且 ，这些配置信息应该从应用程序镜像中解耦出来 ，以保证镜像的可移植性以及配置信息不被泄露 。社区引入 ConfigMap 这个 API 资源来满足这一需求 。

ConfigMap 包含了一系列的键值对 ，用于存储被 Pod 或者系统组件（如controller）访问的信息 。这与 secret 的设计理念有异曲同工之妙 ，它们的主要区别在于 ConfigMap 通常不用于存储敏感信息 ，而只存储简单的文本信息 。





### 10. Horizontal Pod Autoscaler

自动扩展作为一个长久的议题 ，一直为人们津津乐道 。系统能够根据负载的变化对计算资源的分配进行自动的扩增或者收缩 ，无疑是一个非常吸引人的特征 ，它能够最大可能地减少费用或者其他代价（如电力损耗）。自动扩展主要分为两种 ，其一为水平扩展 ，针对于实例数目的增减 。其二为垂直扩展 ，即单个实例可以使用的资源的增减 。Horizontal Pod Autoscaler（HPA）属于前者 。



**Horizontal Pod Autoscaler如何工作**
Horizontal Pod Autoscaler 的操作对象是 Replication Controller、ReplicaSet 或 Deployment对应的Pod ，根据观察到的 CPU 实际使用量与用户的期望值进行比对 ，做出是否需要增减实例数量的决策 。controller 目前使用 heapSter 来检测 CPU 使用量 ，检测周期默认是 30秒 。



**Horizontal Pod Autoscaler的决策策略**

在 HPA Controller 检测到 CPU 的实际使用量之后 ，会求出当前的 CPU 使用率（实际使用量与 pod 请求量的比率) 。然后，HPA Controller 会通过调整副本数量使得 CPU 使用率尽量向期望值靠近 。另外 ，考虑到自动扩展的决策可能需要一段时间才会生效 ，甚至在短时间内会引入一些噪声 ． 例如当 pod 所需要的 CPU 负荷过大 ，从而运行一个新的 pod 进行分流 ，在创建的过程中 ，系统的 CPU 使用量可能会有一个攀升的过程 。所以 ，在每一次作出决策后的一段时间内 ，将不再进行扩展决策 。对于 ScaleUp 而言 ，这个时间段为 3 分钟 ，Scaledown 为 5 分钟 。再者 HPA Controller 允许一定范围内的CPU 使用量的不稳定，也就是说，只有当 aVg（CurrentPodConsumption／Target 低于 0.9 或者高于 1.1 时才进行实例调整 ，这也是出于维护系统稳定性的考虑 。