# Kubernetes架构

## Nodes

Kubernetes 通过将容器放入在节点（Node）上运行的 Pod 中来执行你的工作负载。 节点可以是一个虚拟机或者物理机器，取决于所在的集群配置。 每个节点包含运行 Pods 所需的服务； 这些节点由 control-plane 负责管理。

节点上的组件包括 kubelet、container-runtimes 以及 kube-proxy。

### 管理

向 API 服务器 添加节点的方式主要有两种：

1. 节点上的 `kubelet` 向 control plane 执行自注册；
2. 手动添加一个 Node 对象。

> Kubernetes 会一直保存着非法节点对应的对象，并持续检查该节点是否已经变得健康。 必须显式地删除该 Node 对象以停止健康检查操作。

**Node 对象的名称约束**

- 不能超过 253 个字符
- 只能包含小写字母、数字，以及 '-' 和 '.'
- 必须以字母数字开头
- 必须以字母数字结尾

### 节点自注册

当 kubelet 标志 `--register-node` 为 true（默认）时，它会尝试向 API 服务注册自己。 这是首选模式。

自注册模式，kubelet 使用下列参数启动：

- `--kubeconfig` - 身份认证所用的凭据的路径。
- `--cloud-provider` - 与云驱动通信。
- `--register-node` - 自动向 API 服务注册。
- `--register-with-taints` - 使用所给的[污点](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/taint-and-toleration/)列表。当 `register-node` 为 false 时无效。
- `--node-ip` - 节点 IP 地址。
- `--node-labels` - 在集群中注册节点时要添加的 Labels
- `--node-status-update-frequency` - 指定 kubelet 向控制面发送状态的频率。

> 当 Node 的配置需要被更新时， 一种好的做法是重新向 API 服务器注册该节点。

### 手动节点管理