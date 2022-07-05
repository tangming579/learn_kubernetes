# 使用 RBAC 鉴权

https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/

基于角色（Role）的访问控制（RBAC）是一种基于组织中用户的角色来调节控制对 计算机或网络资源的访问的方法。

RBAC 鉴权机制使用 `rbac.authorization.k8s.io` 来驱动鉴权决定，允许通过 Kubernetes API 动态配置策略。

RBAC API 声明了四种 Kubernetes 对象：

- Role：Namespace 内设置访问权限
- ClusterRole：集群内设置访问权限
- RoleBinding
- ClusterRoleBinding

> RoleBinding 也可以引用 ClusterRole，这种引用使得你可以跨整个集群定义一组通用的角色， 之后在多个名字空间中复用。



> 创建了绑定之后，不能再修改绑定对象所引用的 Role 或 ClusterRole。 试图改变绑定对象的 `roleRef` 将导致合法性检查错误。 如果你想要改变现有绑定对象中 `roleRef` 字段的内容，必须删除重新创建绑定对象。



## 常用命令

```shell
kubectl create role pod-reader --verb=get,list,watch --resource=pods --namespace=acme

kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

kubectl create rolebinding bob-admin-binding --clusterrole=admin --user=bob --namespace=acme

kubectl create clusterrolebinding root-cluster-admin-binding --clusterrole=cluster-admin --user=root
```

# 节点控制

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#cordon

## cordon、drain

```
kubectl cordon NODE
kubectl drain NODE
kubectl uncordon NODE
```

在1.2之前，由于没有相应的命令支持，如果要维护一个节点，只能stop该节点上的kubelet将该节点退出集群，是集群不在将新的pod调度到该节点上。如果该节点上本身就没有pod在运行，则不会对业务有任何影响。如果该节点上有pod正在运行，kubelet停止后，master会发现该节点不可达，而将该节点标记为notReady状态，不会将新的节点调度到该节点上。同时，会在其他节点上创建新的pod替换该节点上的pod。

如此虽能够保证集群的健壮性，但可能还有点问题，如果业务只有一个副本，而且该副本正好运行在被维护节点上的话，可能仍然会造成业务的短暂中断。

- cordon：影响最小，只会将node调为SchedulingDisabled，之后再发创建pod，不会被调度到该节点
- drain：驱逐node上的pod，其他节点重新创建，接着，将节点调为 SchedulingDisabled

# 升级集群

https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

# NetworkPolicy

在 IP 地址或端口层面（OSI 第 3 层或第 4 层）控制网络流量

NetworkPolicies 适用于一端或两端与 Pod 的连接，与其他连接无关。 

Pod 可以通信的 Pod 是通过如下三个标识符的组合来辩识的：

1. 其他被允许的 Pods（例外：Pod 无法阻塞对自身的访问）
2. 被允许的 Namespace
3. IP 组块（例外：与 Pod 运行所在的节点的通信总是被允许的， 无论 Pod 或节点的 IP 地址）

**Pod 隔离的两种类型**

Pod 有两种隔离: 出口的隔离和入口的隔。这两种隔离（或不隔离）是独立声明的， 并且都与从一个 Pod 到另一个 Pod 的连接有关。

默认情况下，一个 Pod 的出口是非隔离的，即所有外向连接都是被允许的。如果有任何的 NetworkPolicy 选择该 Pod 并在其 `policyTypes` 中包含 “Egress”，则该 Pod 是出口隔离的

默认情况下，一个 Pod 对入口是非隔离的，即所有入站连接都是被允许的。如果有任何的 NetworkPolicy 选择该 Pod 并在其 `policyTypes` 中包含 “Ingress”，则该 Pod 被隔离入口， 我们称这种策略适用于该 Pod 的入口

网络策略是相加的，所以不会产生冲突

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```

