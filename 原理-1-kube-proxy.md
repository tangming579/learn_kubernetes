`Kubernetes` 中每个Node上都部署着一个以 `DaemonSet` 形式运行的 `kube-proxy`，当一个Pod需要访问service时，kube-proxy会根据service的定义将请求转发到正确的Pod上。

`kube-proxy` 有三种模式

- `userspace`：它在用户空间监听一个端口，所有服务通过iptables转发到这个端口，然后在其内部负载均衡到实际的Pod。该方式最主要的问题是效率低，有明显的性能瓶颈。
- `iptables`：完全以 iptables 规则的方式来实现 service 负载均衡。服务多的时候产生太多的 iptables 规则，非增量式更新会引入一定的时延，大规模情况下有明显的性能问题。
-  `IPVS`：为解决iptables模式的性能问题，采用增量式更新，并可以保证 service 更新期间连接保持不断开
