# 

Kubernetes 是开源的**容器集群管理系统**
功能：
* 基于容器的应用部署，维护和滚动升级；
* 负载均衡和服务发现；
* 跨机器和跨地区的集群调度；
* 自动伸缩；
* 无状态服务和有状态服务；
* 插件机制保证扩展性。

Kubernetes 架构
![](../../static/images/kubernetes.png)
Kubernetes 集群可以分为两部分：控制平面和计算设备
### Control Plane
控制平面是 Kubernetes 集群的神经中枢，可以找到
* kube-apiserver：k8s Master 上的组件，是整个集群的 API 网关。用于和 Kubernetes 集群进行交互，可以通过 REST 调用，kubectl 命令行等来访问 API，也将数据存储 etcd。(认证，鉴权，存储)
* kube-scheduler：调度程序会考虑容器集群的资源需求以及集群的运行状况（每建一个 pod，该 pod 就会被 scheduler 调度）
* kube-controller-manager：控制器负责实际运行集群，控制器管理器则将多个控制器功能合而为一。
* etcd：配置数据以及有关集群状态的信息位于 etcd 中。

### Compute machines
容器集经过调度和编排后会在节点上运行。
* 容器集：容器集是 Kubernetes 对象模型中最小、最简单的单元。它代表了应用的单个实例。每个容器集都由一个容器（或一系列紧密耦合的容器）以及若干控制容器运行方式的选件组成。容器集可以连接至持久存储，以运行有状态应用。
* 容器运行时引擎：为了运行容器，每个计算节点都有一个容器运行时引擎。比如 Docker，但 Kubernetes 也支持其他符合开源容器运动（OCI）标准的运行时，例如 rkt 和 CRI-O。
* kubelet：1.节点状态上报 API Server；
* kube-proxy：用于优化 Kubernetes 网络服务的网络代理。kube-proxy 负责处理集群内部或外部的网络通信——靠操作系统的数据包过滤层，或者自行转发流量。
* 

## Reference
()[https://www.redhat.com/zh/topics/containers/kubernetes-architecture]
