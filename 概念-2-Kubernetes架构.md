# Kubernetes架构

## Nodes

Kubernetes 通过将容器放入在节点（Node）上运行的 Pod 中来执行你的工作负载。 节点可以是一个虚拟机或者物理机器，取决于所在的集群配置。 每个节点包含运行 Pods 所需的服务； 这些节点由 control-plane 负责管理。

节点上的组件包括 kubelet、container-runtimes 以及 kube-proxy。

### 管理

向 API 服务器 添加节点的方式主要有两种：

1. 节点上的 `kubelet` 向 control plane 执行自注册；
2. 手动添加一个 Node 对象。