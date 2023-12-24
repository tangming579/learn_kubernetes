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



## alertmanager安装

