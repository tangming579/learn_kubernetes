### 基本概念

**Containerd** 是一个开源的**容器运行时**（Container Runtime），由 Docker 公司捐赠给云原生计算基金会（CNCF）。它是现代容器生态系统的核心组件之一，专注于管理容器的生命周期，提供容器运行环境

Containerd 以 Daemon 的形式运行在系统上，通过暴露底层的gRPC API，上层系统可以通过这些API管理机器上的容器。

- **CRI**： 容器运行时规范，定义了两大规范：
  - **OCI 镜像规范**（Image Spec）：定义容器镜像格式（如 Docker 镜像）。
  - **OCI 运行时规范**（Runtime Spec）：定义如何运行容器文件系统及其配置。

- **runc** ： OCI 运行时规范的**参考实现**（其他实现还有 `crun`、`Kata Containers` 等）

**kubelet 如何与 containerd 交互**

路径：kubelet → CRI Plugin（containerd 内部）→ containerd Core

<div>
    <image src="./img/cri.png"></image>
</div>

1. **kubelet** 通过 CRI 接口发起请求（如创建/删除容器）。
2. **containerd** 通过内置的 `cri` 插件（默认启用）接收 gRPC 请求。
3. `cri` 插件调用 containerd 的核心模块（如容器管理、镜像拉取等）完成操作。
4. 底层通过 **OCI 运行时**（如 `runc`）实际启动容器。