## 组件介绍

- Thanos Sidecar：和Pormetheus 实例一起部署
  1. 代理Querier 组件对本地Prometheus数据读取
  2. 将Prometheus 本地监控数据通过对象存储接口上传到对象存储中
  3. 监视Prometheus的本地存储，若发现有新的监控数据保存到磁盘，会将这些监控数据上传至对象存储。
- Thanos Query：实现了 Prometheus API，将来自下游组件提供的数据进行聚合最终返回给查询数据的 client (如 grafana)，类似数据库中间件。
- Thanos Store Gateway: 将对象存储的数据暴露给 Thanos Query 去查询。
- Thanos Ruler: 对监控数据进行评估和告警，还可以计算出新的监控数据，将这些新数据提供给 Thanos Query 查询并且/或者上传到对象存储，以供长期存储。
- Thanos Compact: 将对象存储中的数据进行压缩和降低采样率，加速大时间区间监控数据查询的速度。

## 安装

thanos各组件使用的是同一个二进制包 只是启动参数不同

1. Thanos Sidecar

   ```
   sidecar --prometheus.url=http://192.168.1.202:9090 --tsdb.path=/alidata1/admin/data/prometheus --objstore.config-file=/alidata1/admin/tools/thanos-0.30.1/conf/store.yaml --grpc-address=192.168.1.202:10901 --http-address=192.168.1.202:10902 --log.level=error
   ```

   