`Kubernetes` 中的 `Service` 就是一组同 label 类型 `Pod` 的服务抽象，为服务提供了负载均衡和反向代理能力，在集群中表示一个微服务的概念。`kube-proxy` 组件则是 Service 的具体实现。

kube-proxy 有三种模式

- `userspace`：很少使用
- `iptables`：服务多的时候产生太多的 iptables 规则，非增量式更新会引入一定的时延，大规模情况下有明显的性能问题。
-  `IPVS`：采用增量式更新，并可以保证 service 更新期间连接保持不断开

http://team.jiunile.com/blog/2020/10/k8s-kube-proxy.html