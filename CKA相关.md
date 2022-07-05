# 使用 RBAC 鉴权

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

