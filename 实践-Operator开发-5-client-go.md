## 整体流程

<div>
    <image src="./template/informer.png"></image>
</div>

### Informer 端

- Reflector：Reflector 从 apiserver 监听特定类型的资源，对比resourceVersion，拿到变更通知后，将其放到 DeltaFIFO 队列中。
- Informer：Informer 会不断地从 DeltaFIFO 中读取对象
  - 通知 Indexer：根据对象创建或更新本地的缓存，也就是 store
  - 通知 controller：controller 会调用事先注册的 ResourceEventHandler 回调函数进行处理。
- Indexer：Indexer 中有 informer 维护的指定资源对象的相对于etcd数据的一份本地内存缓存，可通过该缓存获取资源对象，以减少对apiserver、对etcd的请求压力；

### Controller 端

- Resource Event Handlers：用于添加一些过滤条件，判断哪些对象需要加到 WorkQueue 中进一步处理
- WorkQueue：WorkQueue 一般使用的是延时队列实现
- Worker：我们自己业务代码的处理过程
  - 收到 WorkQueue 中的任务
  - 通过 Indexer 从本地缓存检索对象
  - 通过 ClientSet 实现对象的增删改查

