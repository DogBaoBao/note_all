---
description: 目标：梳理大部分常用概念，思想层面的洗涤
---

# 一拳搞定 Kubenetes \| 基础知识1

> Kubernetes 有比较多的概念，建议用看待系统的眼光去看，它不是我们常见的某个工具，它是个平台，是一个强大的生态体系。

## 集群

一个 Kubernetes 集群包含两种类型的资源:

* **Master** 调度整个集群
* **Nodes** 负责运行应用

**Master 负责管理整个集群。** Master 协调集群中的所有活动，例如调度应用、维护应用的所需状态、应用扩容以及推出新的更新。

**Node 是一个虚拟机或者物理机，它在 Kubernetes 集群中充当工作机器的角色** 每个Node都有 Kubelet , 它管理 Node 而且是 Node 与 Master 通信的代理。 Node 还应该具有用于​​处理容器操作的工具，例如 Docker 或 rkt 。处理生产级流量的 Kubernetes 集群至少应具有三个 Node 。

Master 管理集群，Node 用于托管正在运行的应用。

在 Kubernetes 上部署应用时，您告诉 Master 启动应用容器。 Master 就编排容器在群集的 Node 上运行。 **Node 使用 Master 暴露的 Kubernetes API 与 Master 通信。**终端用户也可以使用 Kubernetes API 与集群交互。

补充：

* master 和 node 熟悉分布式的同学都比较好理解，主来控制 or 干活，下面的小弟就只有埋头苦干
* API 的模式方便我们直接和 k8s 交互，可以进行 CI/CD 各种自动化的功能

### Node

一个 pod 总是运行在 **工作节点**。工作节点是 Kubernetes 中的参与计算的机器，可以是虚拟机或物理计算机，具体取决于集群。每个工作节点由主节点管理。工作节点可以有多个 pod ，Kubernetes 主节点会自动处理在群集中的工作节点上调度 pod 。 主节点的自动调度考量了每个工作节点上的可用资源。

每个 Kubernetes 工作节点至少运行:

* Kubelet，负责 Kubernetes 主节点和工作节点之间通信的过程; 它管理 Pod 和机器上运行的容器。
* 容器运行时（如 Docker）负责从仓库中提取容器镜像，解压缩容器以及运行应用程序。

> 一般就是对应我们的宿主机，当你购买 ECS 安装的时候，它就指你的 ECS 机器。

## 部署应用

> 记住控制的思想，是弹性伸缩的核心

### Deployment

Deployment 指挥 Kubernetes 如何创建和更新应用程序的实例。创建 Deployment 后，Kubernetes master 将应用程序实例调度到集群中的各个节点上

创建应用程序实例后，Kubernetes Deployment 控制器会持续监视这些实例。 如果托管实例的节点关闭或被删除，则 Deployment 控制器会将该实例替换为群集中另一个节点上的实例。 **这提供了一种自我修复机制来解决机器故障维护问题。**

#### **核心参数**

* 容器映像
* 运行的副本

### Pod

Pod 是 Kubernetes 抽象出来的，表示一组一个或多个应用程序容器（如 Docker），以及这些容器的一些共享资源。这些资源包括:

* 共享存储，当作卷
* 网络，作为唯一的集群 IP 地址
* 有关每个容器如何运行的信息，例如容器映像版本或要使用的特定端口。

Pod 为特定于应用程序的“逻辑主机”建模，并且可以包含相对紧耦合的不同应用容器。

> 使用时，pod 就如同之前的虚拟机一样愉快的跑着我们的应用。

Pod 中的容器共享 IP 地址和端口，始终位于同一位置并且共同调度，并在同一工作节点上的共享上下文中运行。

Pod是 Kubernetes 平台上的原子单元。 当我们在 Kubernetes 上创建 Deployment 时，该 Deployment 会在其中创建包含容器的 Pod （而不是直接创建容器）。每个 Pod 都与调度它的工作节点绑定，并保持在那里直到终止（根据重启策略）或删除。 如果工作节点发生故障，则会在群集中的其他可用工作节点上调度相同的 Pod。

### Service

Kubernetes [Pod](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-overview/) 是转瞬即逝的。 Pod 实际上拥有 [生命周期](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/)。 当一个工作 Node 挂掉后, 在 Node 上运行的 Pod 也会消亡。 [ReplicaSet](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicaset/) 会自动地通过创建新的 Pod 驱动集群回到目标状态，以保证应用程序正常运行。 换一个例子，考虑一个具有3个副本数的用作图像处理的后端程序。这些副本是可替换的; 前端系统不应该关心后端副本，即使 Pod 丢失或重新创建。也就是说，Kubernetes 集群中的每个 Pod \(即使是在同一个 Node 上的 Pod \)都有一个惟一的 IP 地址，因此需要一种方法自动协调 Pod 之间的变更，以便应用程序保持运行。

> 试想一下，面向故障编程，一切都是动态的变化，而不是以前一个单体应用跑着跑着挂了，那么我们必须去重启它，只有重启后再能继续服务。而 k8s 的思想里面，提供服务的 Pod 可以消亡，Pods 是一个整体提供能力，里面对每一个而言不是必要的。

Kubernetes 中的服务\(Service\)是一种抽象概念，它定义了 Pod 的逻辑集和访问 Pod 的协议。Service 使从属 Pod 之间的松耦合成为可能。 和其他 Kubernetes 对象一样, Service 用 YAML [\(更推荐\)](https://kubernetes.io/zh/docs/concepts/configuration/overview/#general-configuration-tips) 或者 JSON 来定义. Service 下的一组 Pod 通常由 LabelSelector \(请参阅下面的说明为什么您可能想要一个 spec 中不包含`selector`的服务\)来标记。

> Service 可以理解你的一个服务（比如账号服务），它负责去访问你启动的一百个账号服务（Pod）。

尽管每个 Pod 都有一个唯一的 IP 地址，但是如果没有 Service ，这些 IP 不会暴露在群集外部。Service 允许您的应用程序接收流量。Service 也可以用在 ServiceSpec 标记`type`的方式暴露

* ClusterIP \(默认\) - 在集群的内部 IP 上公开 Service 。这种类型使得 Service 只能从集群内访问。
* NodePort - 使用 NAT 在集群中每个选定 Node 的相同端口上公开 Service 。使用`<NodeIP>:<NodePort>` 从集群外部访问 Service。是 ClusterIP 的超集。
* LoadBalancer - 在当前云中创建一个外部负载均衡器\(如果支持的话\)，并为 Service 分配一个固定的外部IP。是 NodePort 的超集。
* ExternalName - 通过返回带有该名称的 CNAME 记录，使用任意名称\(由 spec 中的`externalName`指定\)公开 Service。不使用代理。这种类型需要`kube-dns`的v1.7或更高版本。

> 17年使用的时候还只有3个选项。会用阿里云的负载均衡配置 LoadBalancer 让 API 网关暴露出去，其它常规的服务会用 NodePort。

#### Service 和 Label

Service 匹配一组 Pod 是使用 [标签\(Label\)和选择器\(Selector\)](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/labels), 它们是允许对 Kubernetes 中的对象进行逻辑操作的一种分组原语。标签\(Label\)是附加在对象上的键/值对，可以以多种方式使用:

* 指定用于开发，测试和生产的对象
* 嵌入版本标签
* 使用 Label 将对象进行分类

实操图：

#### pod的标签

![pod](../../.gitbook/assets/image%20%2821%29.png)

![pod](../../.gitbook/assets/image%20%2814%29.png)

#### service的标签

有效：

![service](../../.gitbook/assets/image%20%2816%29.png)

![service](../../.gitbook/assets/image%20%2810%29.png)

失效：

把 `selector` 改 `app: hello-minikube-1`

![](../../.gitbook/assets/image%20%2819%29.png)

更新后，再次 curl 请求无法正常访问：

```bash
tiechengdeMacBook-Pro-2:.kube tc$ curl http://192.168.64.3:31259
curl: (7) Failed to connect to 192.168.64.3 port 31259: Connection refused
```

