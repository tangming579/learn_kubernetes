## 概念

Operator 是一种 kubernetes 的扩展形式，可以帮助用户以 Kubernetes 的声明式 API 风格自定义来管理应用及服务，`Operator` 就可以看成是 CRD 和 Controller 的一种组合特例，Operator 是一种思想，它结合了特定领域知识并通过 CRD 机制扩展了 Kubernetes API 资源，使用户管理 Kubernetes 的内置资源（Pod、Deployment等）一样创建、配置和管理应用程序。

Operator 通过扩展 Kubernetes API 资源以代表 Kubernetes 用户创建、配置和管理复杂应用程序的实例，通常包含资源模型定义和控制器，通过 `Operator` 通常是为了实现某种特定软件（通常是有状态服务）的自动化运维。

相关概念

- **CRD (Custom Resource Definition)**：对自定义资源的描述，是一个类型，但仅有CRD的定义并没有实际作用，用户还需要提供管理CRD对象的CRD控制器（一般是自定义的控制器），才能实现对CRD对象的管理。；
- **CR (Custom Resourse)**：CRD 的一个具体实例；

Operator有时也被称为CRD机制：

- 狭义上来说，operator = CRD + 自定义Controller

简单理解，就是：通过自定义的Controller监听CRD对象实例的增删改事件，然后执行相应的业务逻辑。

- 广义上来说，operator = CRD + 自定义Controller + WebHook

即在前者的基础上，还可以在CRD对象的生命周期里设置相关的事件回调（WebHook），回调程序一般为外部自定义的一个HTTP URL。

## Operator SDK

Operator SDK 提供了用于开发 Go、Ansible 以及 Helm 中的 Operator 的工作流，下面的工作流适用于 Golang 的 Operator：

1. 使用 SDK 创建一个新的 Operator 项目
2. 通过添加自定义资源（CRD）定义新的资源 API
3. 指定使用 SDK API 来 watch 的资源
4. 定义 Operator 的协调（reconcile）逻辑
5. 使用 Operator SDK 构建并生成 Operator 部署清单文件

### 安装

参考：https://andblog.cn/3209

前置安装

```
安装gcc和make。
安装golang1.17以上版本。
一个可进入的公共的docker registry服务，并且准备一个域名作为registry服务的域名。
```

安装gcc和make

```sh
# Linux系统：
apt-get install gcc automake autoconf libtool make

# Windows系统-方法1：
直接安装cygwin，选择gcc和make等组件即可

# Windows系统-方法2：
1.下载并安装 MinGW。
2.将 MinGW 的 bin 目录添加到系统的环境变量 path 中。412
3.将 MinGW 的 bin 目录中的 mingw32-make.exe 改名为 make.exe。41
4.在命令行中输入 make -v 来检查是否安装成功。4
```

安装SDK（Windows下需要在 Cygwin Terminal 中执行）

https://sdk.operatorframework.io/docs/installation/

Windows下Make命令：

https://cygwin.com/install.html

https://www.mingw-w64.org/downloads/#mingw-builds

https://jmeubank.github.io/tdm-gcc/download/

```sh
$ git clone https://github.com/operator-framework/operator-sdk
$ cd operator-sdk
$ git checkout master
$ make install

#确定结果
$ operator-sdk version
```

### 常用命令

#### 1. operator-sdk build

命令编译代码并生成可执行文件

```
operator-sdk build quay.io/example/operator:v0.0.1
```

#### 2. operator-sdk completion

生成bash补全

```
operator-sdk completion
```

#### 3. operator-sdk print-deps

命令显示操作员所需的最新Golang软件包和版本。默认情况下，它以列格式打印

```
operator-sdk print-deps --as-file
```

#### 4. operator-sdk generate

命令将调用特定的生成器以根据需要生成代码

#### 5. operator-sdk olm-catalog gen-csv

子命令将集群服务版本（CSV）清单和可选的自定义资源定义（CRD）文件写入 `deploy/olm-catalog/<operator_name>/<csv_version>`

```
$ operator-sdk olm-catalog gen-csv --csv-version 0.1.0 --update-crds
INFO[0000] Generating CSV manifest version 0.1.0
INFO[0000] Fill in the following required fields in file deploy/olm-catalog/operator-name/0.1.0/operator-name.v0.1.0.clusterserviceversion.yaml:
    spec.keywords
    spec.maintainers
    spec.provider
    spec.labels
INFO[0000] Created deploy/olm-catalog/operator-name/0.1.0/operator-name.v0.1.0.clusterserviceversion.yaml
```

#### 6. operator-sdk new

```
$ mkdir $GOPATH/src/github.com/example.com/
$ cd $GOPATH/src/github.com/example.com/
$ operator-sdk new app-operator
$ operator-sdk new app-operator \
    --type=ansible \
    --api-version=app.example.com/v1alpha1 \
    --kind=AppService
```

#### 7. operator-sdk add

命令将控制器或资源添加到项目中。该命令必须从Operator项目的根目录运行。

#### 8. operator-sdk up local

该local子命令通过构建可以使用kubeconfig文件访问Kubernetes集群的操作员二进制文件在本地计算机上启动操作员 。

```
$ operator-sdk up local \
  --kubeconfig "mycluster.kubecfg" \
  --namespace "default" \
  --operator-flags "--flag1 value1 --flag2=value2"
$ operator-sdk up local --operator-flags "--resync-interval 10"
$ operator-sdk up local --namespace "testing"
```

### 初始化

1. 开启 go module 和代理

   ```sh
   $ go env -w GO111MODULE=on
   $ go env -w GOPROXY=https://goproxy.cn,direct
   ```

2. 设置环境变量

   macOS 或 Linux

   ```sh
   $ echo "export GO111MODULE=on" >> ~/.profile
   $ echo "export GOPROXY=https://goproxy.cn" >> ~/.profile
   $ source ~/.profile
   ```

   Windows 

   ```sh
   # 打开Powershell执行：
   C:\> $env:GO111MODULE = "on"
   C:\> $env:GOPROXY = "https://goproxy.cn"
   ```

3. 初始化项目

   ```sh
   # 创建项目目录
   $ mkdir -p opdemo && cd opdemo
   # 使用 sdk 创建一个名为 opdemo 的 operator 项目，如果在 GOPATH 之外需要指定 repo 参数
   $ go mod init github.com/tangming579/opdemo/v2
   # 使用下面的命令初始化项目
   $ operator-sdk init --domain tangming579.io --license apache2 --owner "tangming579"
   ```

4. 升级后项目结构如下

   ```sh
   $ tree -L 2
   .
   ├── config
   │   ├── default
   │   ├── manager
   │   ├── manifests
   │   ├── prometheus
   │   ├── rbac
   │   └── scorecard
   ├── Dockerfile
   ├── go.mod
   ├── go.sum
   ├── hack
   │   └── boilerplate.go.txt
   ├── main.go
   ├── Makefile
   ├── PROJECT
   └── README.md
   
   9 directories, 8 files
   ```

   ### 添加API

   使用 `operator-sdk init` 命令创建新的 Operator 项目结构：

   - go.mod/go.sum  – Go Modules 包管理清单，用来描述当前 Operator 的依赖包。

   - main.go 文件，使用 operator-sdk API 初始化和启动当前 Operator 的入口。
   - deploy – 包含一组用于在 Kubernetes 集群上进行部署的通用的 Kubernetes 资源清单文件。
   - pkg/apis – 包含定义的 API 和自定义资源（CRD）的目录树，这些文件允许 sdk 为 CRD 生成代码并注册对应的类型，以便正确解码自定义资源对象。
   - pkg/controller – 用于编写所有的操作业务逻辑的地方
   - version – 版本定义
   - build – Dockerfile 定义目录

   我们主要需要编写的是 `pkg` 目录下面的 api 定义以及对应的 controller 实现。

   Operator 相关根目录下面执行如下命令添加新 API

   ```sh
   $ operator-sdk create api --group app --version v1beta1 --kind AppService
   Create Resource [y/n]
   y
   Create Controller [y/n]
   y
   Writing kustomize manifests for you to edit...
   Writing scaffold for you to edit...
   api/v1beta1/appservice_types.go
   controllers/appservice_controller.go
   Update dependencies:
   $ go mod tidy
   Running make:
   $ make generate
   mkdir -p /root/opdemo/bin
   test -s /root/opdemo/bin/controller-gen && /root/opdemo/bin/controller-gen --version | grep -q v0.11.1 || \
   GOBIN=/root/opdemo/bin go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.11.1
   
   ```

   
