## 虚拟机规划

|         | 主机名      | IP             |
| :------ | :---------- | :------------- |
| 主节点  | master-node | 192.168.56.102 |
| 从节点1 | work-node1  | 192.168.56.103 |
| 从节点2 | work-node2  | 192.168.56.104 |

## 必要的准备

## 关闭防火墙

防火墙一定要提前关闭，否则在后续安装K8S集群的时候是个trouble maker。执行下面语句关闭，并禁用开机启动：

```bash
systemctl stop firewalld & systemctl disable firewalld
```

### 关闭Swap

类似ElasticSearch集群，在安装K8S集群时，Linux的Swap内存交换机制是一定要关闭的，否则会因为内存交换而影响性能以及稳定性。这里，我们可以提前进行设置：

执行swapoff -a可临时关闭，但系统重启后恢复
编辑/etc/fstab，注释掉包含swap的那一行即可，重启后可永久关闭，如下所示：

```
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=c32383b8-5912-4536-8cd5-4d6ab99d8c45 /boot                   xfs     defaults        0 0
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

## 关闭SeLinux

临时关闭：

```bash
setenforce 0
```

要永久禁用SELinux，请使用您最喜欢的文本编辑器打开/etc/sysconfig/selinux文件，如下所示：

```
vi /etc/sysconfig/selinux
```

然后将配置SELinux=enforcing改为SELinux=disabled，如下图所示。

```
SELINUX=disabled
```

然后，保存并退出文件，为了使配置生效，需要重新启动系统，然后使用sestatus命令检查SELinux的状态，如下所示：

```
sestatus
```