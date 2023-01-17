## helm 是什么

helm 是 kubernetes 生态系统中的一个软件包管理工具，类似 Ubuntu 的 apt, CentOS 的 yum 或 python 的 pip 一样，专门负责管理 kubernetes 应用资源；使用 helm 可以对 kubernetes 应用进行统一打包、分发、安装、升级以及回退等操作。

helm 利用Chart来封装 kubernetes 原生应用程序的一些列 yaml 文件，可以在部署应用的时候自定义应用程序的一些 Metadata ，以便于应用程序的分发。

### helm 为什么出现

利用Kubernetes部署一个应用，需要Kubernetes原生资源文件如 deployment、replicationcontroller、service 或 pod 等。这些 k8s 资源过于分散，不方便进行管理，直接通过 kubectl 来管理一个应用，非常的不方便。

helm主要作用：

- 应用程序封装
- 版本管理
- 依赖检查
- 便于应用程序分发

### Helm 架构

Helm中有三个重要概念，分别为Chart、Repository和Release。

- Chart：是一个Helm包。它包含在K8s集群内部运行应用程序，工具或服务所需的所有资源定义。可以类比成yum中的RPM。

- 仓库（Repository）：用来存放和共享Chart的地方，可以类比成Maven仓库。

- 发布（Release）：运行在K8s集群中的Chart的实例，一个Chart可以在同一个集群中安装多次。Chart就像流水线中初始化好的模板，Release就是这个“模板”所生产出来的各个产品。

Helm作为K8s的包管理软件，每次安装Charts 到K8s集群时，都会创建一个新的 release。你可以在Helm 的Repository中寻找需要的Chart。Helm对于部署过程的优化的点在于简化了原先完成配置文件编写后还需使用一串kubectl命令进行的操作、统一管理了部署时的可配置项以及方便了部署完成后的升级和维护。

## Helm 安装

### 二进制安装

helm 二进制包安装，helm 只是是一个单纯的可执行程序

下载最新 release：https://github.com/helm/helm

```sh
tar -zxvf helm-v3.10.3-linux-amd64.tar.gz
cp -av linux-amd64/helm /usr/bin/
```

常用命令

```
helm create 创建一个Helm Chart初始安装包工程
helm search 在Helm仓库中查找应用
helm install 安装Helm
helm list 罗列K8s集群中的部署的Release列表
helm lint 对一个Helm Chart进行语法检查和校验
```

### 配置Helm仓库

- 微软仓库 http://mirror.azure.cn/kubernetes/charts/
- 阿里云仓库 https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
- 官方仓库 https://hub.kubeapps.com/charts/incubator