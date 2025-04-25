## 基本概念

**Containerd** 是一个开源的**容器运行时**（Container Runtime），由 Docker 公司捐赠给云原生计算基金会（CNCF）。它是现代容器生态系统的核心组件之一，专注于管理容器的生命周期，提供容器运行环境

Containerd 以 Daemon 的形式运行在系统上，通过暴露底层的gRPC API，上层系统可以通过这些API管理机器上的容器。

### OCI

Open Container Initiative

- **目标**：定义容器镜像和运行时的**开放标准**（中立规范，不绑定具体实现）。
- **发起者**：由 Docker、Google、CoreOS 等公司发起，现由 Linux 基金会管理。

**两大核心规范**

| 规范名称             | 作用                                                         |
| -------------------- | ------------------------------------------------------------ |
| **OCI Image Spec**   | 定义容器镜像格式（如层（layers）、配置、manifest等），兼容 Docker 镜像格式。 |
| **OCI Runtime Spec** | 定义如何运行容器的标准（如根文件系统、namespaces、cgroups 配置等）。 |

**常见 OCI 运行时实现**

- **`runc`**（默认实现，被 containerd、CRI-O 使用）
- **`crun`**（Red Hat 开发的轻量实现）
- **`Kata Containers`**（基于虚拟机的运行时）
- **`gVisor`**（用户态内核隔离）

### CRI

Container Runtime Interface

- **目标**：定义 Kubernetes **kubelet** 与**容器运行时**之间的通信接口（API 规范）。

- **发起者**：由 Kubernetes 社区提出，用于解耦 kubelet 与具体运行时。

- **核心功能**：

  - 标准化 kubelet 对容器的操作（如创建/删除 Pod、执行命令、获取日志等）。

  - 支持多种运行时通过插件方式接入 Kubernetes（如 containerd、CRI-O、Docker 等）。

**kubelet 如何与 containerd 交互**

路径：kubelet → CRI Plugin（containerd 内部）→ containerd Core

<div>
    <image src="./img/cri.png"></image>
</div>

1. **kubelet** 通过 CRI 接口发起请求（如创建/删除容器）。
2. **containerd** 通过内置的 `cri` 插件（默认启用）接收 gRPC 请求。
3. `cri` 插件调用 containerd 的核心模块（如容器管理、镜像拉取等）完成操作。
4. 底层通过 **OCI 运行时**（如 `runc`）实际启动容器。

## 配置

配置文件：

| **功能**       | **Docker**                | **Containerd**                |
| -------------- | ------------------------- | ----------------------------- |
| **主配置文件** | `/etc/docker/daemon.json` | `/etc/containerd/config.toml` |
| **日志配置**   | 在 `daemon.json` 中调整   | 在 `config.toml` 中调整       |

## 命令

- `ctr`：containerd 原生命令行工具（不兼容 Docker 命令格式）。
- `crictl`：Kubernetes CRI 工具，命令类似 Docker（建议优先使用）。

### 镜像管理

| **操作**     | **Docker**          | **Containerd**                                   |
| ------------ | ------------------- | ------------------------------------------------ |
| **拉取镜像** | `docker pull nginx` | `ctr images pull docker.io/library/nginx:latest` |
| **列出镜像** | `docker images`     | `crictl images` 或 `ctr images ls`               |
| **删除镜像** | `docker rmi nginx`  | `ctr images rm docker.io/library/nginx:latest`   |

### 容器管理

| **操作**     | **Docker**                | **Containerd**                                    |
| ------------ | ------------------------- | ------------------------------------------------- |
| **运行容器** | `docker run -d nginx`     | `crictl runp` (Pod) + `crictl create` (Container) |
| **列出容器** | `docker ps`               | `crictl ps` 或 `ctr containers ls`                |
| **进入容器** | `docker exec -it <ID> sh` | `crictl exec -it <ID> sh`                         |
| **停止容器** | `docker stop <ID>`        | `crictl stop <ID>`                                |
| **查看日志** | `docker logs <ID>`        | `crictl logs <ID>`                                |
| **删除容器** | `docker rm <ID>`          | `crictl rm <ID>`                                  |

### 资源监控

| **Docker**       | **Containerd**   |
| ---------------- | ---------------- |
| `docker stats`   | `crictl stats`   |
| `docker inspect` | `crictl inspect` |