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

设置 kubelet 标志 `--register-node=false`。

可以结合使用 Node 上的标签和 Pod 上的选择算符来控制调度

一个节点的状态包含以下信息:

- 地址（Addresses）：用法取决于你的云服务商或者物理机配置
- 状况（Condition)：字段描述了所有 `Running` 节点的状况
- 容量与可分配（Capacity）：描述节点上的可用资源：CPU、内存和可以调度到节点上的 Pod 的个数上限
- 信息（Info）： 指的是节点的一般信息，如内核版本、Kubernetes 版本等

可以使用 `kubectl` 来查看节点状态和其他细节信息：

```shell
kubectl describe node <节点名称>
```

| 节点状况             | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| `Ready`              | 如节点是健康的并已经准备好接收 Pod 则为 `True`；`False` 表示节点不健康而且不能接收 Pod；`Unknown` 表示节点控制器在最近 `node-monitor-grace-period` 期间（默认 40 秒）没有收到节点的消息 |
| `DiskPressure`       | `True` 表示节点存在磁盘空间压力，即磁盘可用量低, 否则为 `False` |
| `MemoryPressure`     | `True` 表示节点存在内存压力，即节点内存可用量低，否则为 `False` |
| `PIDPressure`        | `True` 表示节点存在进程压力，即节点上进程过多；否则为 `False` |
| `NetworkUnavailable` | `True` 表示节点网络配置不正确；否则为 `False`                |

### 心跳 

Kubernetes 节点发送的心跳帮助集群确定每个节点的可用性，并在检测到故障时采取行动。

对于 Nodes 节点，有两种形式的心跳:

- 更新节点的 `.status`
- `kube-node-lease` namespaces 中的 Lease（租约）对象。 每个节点都有一个关联的 Lease 对象。

与 Node 的 `.status` 更新相比，Lease 是一种轻量级资源。 使用 Lease 来表达心跳在大型集群中可以减少这些更新对性能的影响。

kubelet 负责创建和更新节点的 `.status`，以及更新它们对应的 Lease。

### 节点控制器

节点控制器在节点的生命周期中扮演多个角色：

1. 当节点注册时为它分配一个 CIDR 区段（如果启用了 CIDR 分配）。
2. 保持节点控制器内的节点列表与云服务商所提供的可用机器列表同步
3. 监控节点的健康状况

### 节点拓扑

如果启用了 TopologyManager 特性门控， kubelet 可以在作出资源分配决策时使用拓扑提示。 

## 控制面到节点通信