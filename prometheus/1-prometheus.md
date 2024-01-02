## consul安装

1. 安装

   ```
   wget https://releases.hashicorp.com/consul/1.17.1/consul_1.17.1_linux_amd64.zip
   unzip consul_1.17.1_linux_amd64.zip -C /usr/local/consul/
   ```

2. 启动服务

   ```
   ./consul agent -server -bootstrap-expect 1 -data-dir=/usr/local/consul/data -config-dir=/etc/consul/ -node=n1 -bind=127.0.0.1 -client=0.0.0.0 -ui
   ```

3. 新增acl配置在/etc/consul下

   ```
   {
     "acl" : {
       "enabled" : true,
       "default_policy" : "deny",
       "down_policy" : "extend-cache"
     }
   }
   ```

4. 创建acl token（日志就会输出 `consul.acl: ACL bootstrap completed`）

   ```
   $ ./consul acl bootstrap
   AccessorID:       26c008ee-ca29-16b2-1c5c-c6b8b515fc82
   SecretID:         a689274a-8f8b-214e-d8d8-3d79982b39cc
   Description:      Bootstrap Token (Global Management)
   Local:            false
   Create Time:      2023-12-24 20:52:45.8803759 +0800 CST
   Policies:
      00000000-0000-0000-0000-000000000001 - global-management
   ```

5. 节点增加token信息

   ```
   {
     "acl": {
       "enabled": true,
       "default_policy": "deny",
       "enable_token_persistence": true,
       "tokens": {
           "master": "a689274a-8f8b-214e-d8d8-3d79982b39cc"
       }
     }
   }
   ```

## prometheus安装

1. 下载并解压服务

   ```
   tar xvfz prometheus-*.tar.gz
   cd prometheus-*
   ```

2. 配置prometheus

   https://www.python100.com/html/120606.html

   https://blog.csdn.net/w2009211777/article/details/124005822

3. 启动prometheus

   ```
   ./prometheus --config.file=prometheus.yml
   ```

4. 配置说明，参考：

   - https://prometheus.io/docs/prometheus/latest/configuration/configuration/
   - https://www.cnblogs.com/zhoujinyi/p/11944176.html

   ```
   global:
     # 默认情况下抓取目标的频率.
     [ scrape_interval: <duration> | default = 1m ]
     # 抓取超时时间.
     [ scrape_timeout: <duration> | default = 10s ]
     # 评估规则的频率.
     [ evaluation_interval: <duration> | default = 1m ]
     # 与外部系统通信时添加到任何时间序列或警报的标签（联合，远程存储，Alertmanager）.即添加到拉取的数据并存到数据库中
     external_labels:
       [ <labelname>: <labelvalue> ... ]
   # 规则文件指定了一个globs列表. 
   # 从所有匹配的文件中读取规则和警报.
   rule_files:
     [ - <filepath_glob> ... ]
   # 抓取配置列表.
   scrape_configs:
     [ - <scrape_config> ... ]
   # 警报指定与Alertmanager相关的设置.
   alerting:
     alert_relabel_configs:
       [ - <relabel_config> ... ]
     alertmanagers:
       [ - <alertmanager_config> ... ]
   # 与远程写入功能相关的设置.
   remote_write:
     [ - <remote_write> ... ]
   # 与远程读取功能相关的设置.
   remote_read:
     [ - <remote_read> ... ]
   ```

## alertmanager安装

