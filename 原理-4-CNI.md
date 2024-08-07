网络栈包括：**网卡（Network Interface）、回环设备（Loopback Device）、路由表（Routing Table）和 iptables 规则**。

把每一个容器看做一台主机，它们都有一套独立的“网络栈”。如果想要实现两台主机之间的通信，最直接的办法，就是把它们用一根网线连接起来；而如果想要实现多台主机之间的通信，那就需要用网线，把它们连接在一台交换机上。

在 Linux 中，能够起到虚拟交换机作用的网络设备，是网桥（Bridge）。它是一个工作在数据链路层（Data Link）的设备，主要功能是根据 MAC 地址学习来将数据包转发到网桥的不同端口（Port）上。

Docker 项目会默认在宿主机上创建一个名叫 docker0 的网桥，凡是连接在 docker0 网桥上的容器，就可以通过它来进行通信。而容器“连接”到 docker0 网桥上，需要使用一种名叫 Veth Pair 的虚拟设备。

网络插件真正要做的事情，则是通过某种方法，把不同宿主机上的特殊设备连通，从而达到容器跨主机通信的目的。

Kubernetes 是通过一个叫作 CNI 的接口，维护了一个单独的网桥来代替 docker0。这个网桥的名字就叫作：CNI 网桥，它在宿主机上的设备名称默认是：cni0



### 1. 容器到容器（同一Pod内）通信流程

- 在Kubernetes中，默认情况下，同一Pod内的所有容器共享相同的网络命名空间，这意味着它们可以通过localhost直接通信。
- 不涉及物理网卡和网桥，因为它们都在同一个虚拟网络环境里，可以通过进程间通信（IPC）或者网络套接字在同一网络命名空间内直接交流。

### 2. pod之间的通信（以Calico为例）

- 当创建Pod时，Calico CNI插件会为Pod分配一个全局唯一IP地址，并将其添加到Calico创建的BGP网络中。
- Pod的数据包通过veth pair虚拟网卡进行传输，每个Pod都有一个veth对，一端连接到Pod的网络命名空间，另一端连接到主机上的Calico网桥（比如cali+随机字符串）。
- 主机上的Calico节点代理（bird/bird6）将Pod的IP地址及其所在主机的信息通过BGP协议传播到集群内的其他节点。
- 当一个Pod需要与另一个Pod通信时，数据包首先通过其veth对发送到Calico网桥，然后由Calico节点代理根据BGP路由表进行转发。
  如果目标Pod在本地节点，则直接通过内核路由到目标Pod的veth对，进而进入目标Pod的网络命名空间。
- 如果目标Pod在远程节点，则数据包通过主机的物理网卡（如eth0）发送到数据中心网络，通过交换机到达目标节点的物理网卡，然后再通过该节点上的Calico网络栈将数据包路由到目标Pod。
  

参考：

https://blog.csdn.net/xixihahalelehehe/article/details/119485267

https://blog.csdn.net/xixihahalelehehe/article/details/119535258

https://blog.csdn.net/lhq1363511234/article/details/138045261

