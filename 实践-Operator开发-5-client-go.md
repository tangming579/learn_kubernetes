

- Reflector：Reflector 从 apiserver 监听特定类型的资源，拿到变更通知后，将其放到 DeltaFIFO 队列中。
- Informer：Informer 从 DeltaFIFO 中弹出相应对象
- Indexer：Indexer 主要提供一个对象根据一定条件检索的能力
- WorkQueue：WorkQueue 一般使用的是延时队列实现
- Resource Event Handlers：用于添加一些简单的过滤条件，判断哪些对象需要加到 WorkQueue 中进一步处理
- Worker：我们自己业务代码的处理过程
  - 收到 WorkQueue 中的任务
  - 通过 Indexer 从本地缓存检索对象
  - 通过 ClientSet 实现对象的增删改查