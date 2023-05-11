## 概念

Operator 是一种 kubernetes 的扩展形式，可以帮助用户以 Kubernetes 的声明式 API 风格自定义来管理应用及服务

相关概念

- **CRD (Custom Resource Definition)**：对自定义资源的描述，是一个类型，但仅有CRD的定义并没有实际作用，用户还需要提供管理CRD对象的CRD控制器（一般是自定义的控制器），才能实现对CRD对象的管理。；
- **CR (Custom Resourse)**：CRD 的一个具体实例；

Operator有时也被称为CRD机制：

- 狭义上来说，operator = CRD + 自定义Controller

简单理解，就是：通过自定义的Controller监听CRD对象实例的增删改事件，然后执行相应的业务逻辑。

- 广义上来说，operator = CRD + 自定义Controller + WebHook

即在前者的基础上，还可以在CRD对象的生命周期里设置相关的事件回调（WebHook），回调程序一般为外部自定义的一个HTTP URL。

## Operator SDK

### 安装

前置安装

```
安装gcc和make。
安装golang1.17以上版本。
一个可进入的公共的docker registry服务，并且准备一个域名作为registry服务的域名。
```

安装gcc和make

```
apt-get install gcc automake autoconf libtool make
window下直接安装cygwin，选择gcc和make等组件即可
```

安装SDK

https://sdk.operatorframework.io/docs/installation/

```
$ git clone https://github.com/operator-framework/operator-sdk
$ cd operator-sdk
$ git checkout master
$ make install

#确定结果
$ operator-sdk version
```

