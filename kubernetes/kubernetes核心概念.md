### Kubernetes核心概念

###### 1. Pod

  > 运行于Node节点上，若干相关容器的组合。Pod内包含的容器运行在同一宿主机上，使用相同的网络命名空间、IP地址和端口，能够通过localhost进行通。Pod是Kurbernetes进行创建、调度和管理的最小单位，它提供了比容器更高层次的抽象，使得部署和管理更加灵活。一个Pod可以包含一个容器或者多个相关容器。

  > Pod其实有两种类型：普通Pod和静态Pod，后者比较特殊，它并不存在Kubernetes的etcd存储中，而是存放在某个具体的Node上的一个具体文件中，并且只在此Node上启动。普通Pod一旦被创建，就会被放入etcd存储中，随后会被Kubernetes Master调度到摸个具体的Node上进行绑定，随后该Pod被对应的Node上的kubelet进程实例化成一组相关的Docker容器启动起来，在默认情况下，当Pod里的某个容器停止时，Kubernetes会自动检测到这个问起并且重启这个Pod（重启Pod里的所有容器），如果Pod所在的Node宕机，则会将这个Node上的所有Pod重新调度到其他节点上。

###### 2.6.Label

> Kubernetes中的任意API对象都是通过Label进行标识，Label的实质是一系列的Key/Value键值对，其中key于value由用户自己指定。Label可以附加在各种资源对象上，如Node、Pod、Service、RC等，一个资源对象可以定义任意数量的Label，同一个Label也可以被添加到任意数量的资源对象上去。Label是Replication Controller和Service运行的基础，二者通过Label来进行关联Node上运行的Pod。
我们可以通过给指定的资源对象捆绑一个或者多个不同的Label来实现多维度的资源分组管理功能，以便于灵活、方便的进行资源分配、调度、配置等管理工作。

> 一些常用的Label如下：
  - 版本标签："release":"stable","release":"canary"......
  - 环境标签："environment":"dev","environment":"qa","environment":"production"
  - 架构标签："tier":"frontend","tier":"backend","tier":"middleware"
  - 分区标签："partition":"customerA","partition":"customerB"
  - 质量管控标签："track":"daily","track":"weekly"

> Label相当于我们熟悉的标签，给某个资源对象定义一个Label就相当于给它大了一个标签，随后可以通过Label Selector（标签选择器）查询和筛选拥有某些Label的资源对象，Kubernetes通过这种方式实现了类似SQL的简单又通用的对象查询机制。

> Label Selector在Kubernetes中重要使用场景如下:
  - kube-Controller进程通过资源对象RC上定义Label Selector来筛选要监控的Pod副本的数量，从而实现副本数量始终符合预期设定的全自动控制流程
  - kube-proxy进程通过Service的Label Selector来选择对应的Pod，自动建立起每个Service岛对应Pod的请求转发路由表，从而实现Service的智能负载均衡
　- 通过对某些Node定义特定的Label，并且在Pod定义文件中使用Nodeselector这种标签调度策略，kuber-scheduler进程可以实现Pod”定向调度“的特性

###### 3.Deployment

  > Deployment用来管理Pod的副本，保证集群中存在指定数量的Pod副本。集群中副本的数量大于指定数量，则会停止指定数量之外的多余容器数量，反之，则会启动少于指定数量个数的容器，保证数量不变。Deployment是实现弹性伸缩、动态扩容和滚动升级的核心。

###### 4.Service

> Service定义了Pod的逻辑集合和访问该集合的策略，是真实服务的抽象。Service提供了一个统一的服务访问入口以及服务代理和发现机制，关联多个相同Label的Pod，用户不需要了解后台Pod是如何运行。

> 外部系统访问Service的问题
```shell
首先需要弄明白Kubernetes的三种IP这个问题
　Node IP：Node节点的IP地址
  Pod IP： Pod的IP地址
　Cluster IP：Service的IP地址
```

- 1. Node IP是Kubernetes集群中节点的物理网卡IP地址，所有属于这个网络的服务器之间都能通过这个网络直接通信。这也表明Kubernetes集群之外的节点访问Kubernetes集群之内的某个节点或者TCP/IP服务的时候，必须通过Node IP进行通信
- 2. Pod IP是每个Pod的IP地址，他是Docker Engine根据docker0网桥的IP地址段进行分配的，通常是一个虚拟的二层网络。
- 3. Cluster IP是一个虚拟的IP，但更像是一个伪造的IP网络，原因有以下几点
Cluster IP仅仅作用于Kubernetes Service这个对象，并由Kubernetes管理和分配P地址
Cluster IP无法被ping，他没有一个“实体网络对象”来响应
Cluster IP只能结合Service Port组成一个具体的通信端口，单独的Cluster IP不具备通信的基础，并且他们属于Kubernetes集群这样一个封闭的空间。
Kubernetes集群之内，Node IP网、Pod IP网于Cluster IP网之间的通信，采用的是Kubernetes自己设计的一种编程方式的特殊路由规则。

###### Namespace

> Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的 pods, services,deployments 等都是属于某一个 namespace 的（默认是default），而 Node, PersistentVolumes 等则不属于任何 namespace。
