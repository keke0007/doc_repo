# 中间件 Docker 安装知识点梳理

> 本文档基于 `docker/docker-compose/` 下的 13 个真实项目（docker-hmall、docker-mall、docker-spzx、javal4mall、nacos1.4、nacos2.0.2-mysql5.71、nacos2.0.2-mysql8.0、NFTurbo、pd-auth、ry-demo、ry_v1、ry_v2、sfbx）梳理而成，统一覆盖 MySQL、Nacos、Redis、MongoDB、RabbitMQ、Elasticsearch+Kibana、MinIO、RocketMQ、Seata、Canal、InfluxDB、XXL-Job 等常用中间件的 docker-compose 安装方法、目录结构、典型错误与正确写法。

---

## 目录

1. 项目目录通用约定
2. Docker Compose 通用知识点
3. MySQL
4. Nacos（1.4 / 2.x）
5. Redis
6. MongoDB
7. RabbitMQ（含延迟队列插件）
8. Elasticsearch + Kibana
9. MinIO
10. RocketMQ（NameServer + Broker + Dashboard）
11. Seata
12. Canal
13. InfluxDB
14. XXL-Job
15. 现有 docker-compose 文件中发现的错误清单（含修正）

---

## 1. 项目目录通用约定

绝大多数项目（ry_v1 / ry_v2 / pd-auth / nacos1.4 / nacos2.0.2-* / docker-hmall / docker-mall / docker-spzx / NFTurbo / sfbx）都采用如下"一个项目一个 docker-compose.yaml + 中间件子目录挂载"的模式：

```
docker-compose/<project>/
├── docker-compose.yaml         # compose 主文件
├── mysql/
│   ├── conf/my.cnf             # 挂到 /etc/my.cnf
│   ├── data/                   # 挂到 /var/lib/mysql（数据持久化）
│   ├── source/*.sql            # 挂到 /docker-entrypoint-initdb.d（初始化脚本）
│   └── mydir/                  # 自定义临时目录（用于 docker cp、备份）
├── redis/data/                 # /data
├── mongo/{data,log,config,init}
├── rabbitmq/{data,plugins}
├── elasticsearch/{data,plugins,config}
├── minio/{data,config}
├── rocketmq/{namesrv,broker}/{logs,store,conf}
└── nacos/logs
```

要点：

- `*/source/`（或 `mysql/initdb`）通过 bind mount 到 `/docker-entrypoint-initdb.d/`，**仅在 data 目录为空时**首次启动会自动执行 `.sql / .sh / .sql.gz`，二次启动会被跳过。想强制重新初始化必须先 `rm -rf mysql/data`。
- 单文件 bind mount（如 `./mysql/conf/my.cnf:/etc/my.cnf`）若宿主机文件不存在，Docker 会**自动把它当目录创建**导致容器启动失败，这是初学者最常踩的坑。
- 全部使用 `restart: always` 让容器随 Docker 守护进程自动起。

---

## 2. Docker Compose 通用知识点

| 关键字            | 作用                                                                                                |
| ----------------- | --------------------------------------------------------------------------------------------------- |
| `version`         | 当前项目里有 `3.5` / `3.8` 两种。3.8 已是 Compose v1 末期版本，**新版 docker compose v2 实际忽略它**，仅为兼容老 docker-compose（v1）保留 |
| `services`        | 一个服务即一个容器                                                                                  |
| `image`           | 镜像名:tag，建议固定 tag（项目里有 `mysql:5.7`、`mysql:8.0.26`、`mysql:8.0.42`、`mysql:8.0.35` 多种） |
| `container_name`  | 容器名固定，方便在同网络内用 DNS 直连（如 `mysql:3306`）                                            |
| `restart: always` | 异常退出自动重启                                                                                    |
| `ports`           | `宿主机:容器`端口映射                                                                               |
| `expose`          | 仅暴露给同 network 容器，不发布到宿主机                                                             |
| `volumes`         | 数据持久化 / 配置注入                                                                               |
| `environment`     | 容器环境变量                                                                                        |
| `depends_on`      | **仅控制启动顺序，不等待服务真正就绪**                                                              |
| `networks`        | 自定义网络，同一网络内容器可用 `container_name` 互相 DNS 解析                                       |
| `network_mode: host` | 容器共享宿主机网络栈，端口直接落到宿主机上，此时 `ports` / `expose` 都无效                       |
| `command`         | 覆盖镜像 CMD                                                                                        |

### 网络模式对比

```
+----------------------------------------------------------------+
|  bridge（项目里 mall_net / ry_net / nacos_net / extnetwork）    |
|----------------------------------------------------------------|
|  容器间通过 container_name 直连：jdbc:mysql://mysql:3306        |
|  通过 ports 把端口发布到宿主机                                   |
|                                                                |
|  host（javal4mall 使用）                                        |
|----------------------------------------------------------------|
|  容器与宿主机共用网络栈，容器内 127.0.0.1 = 宿主机              |
|  ports / expose 失效，必须直接用宿主机 IP（192.168.1.46）        |
+----------------------------------------------------------------+
```

---

## 3. MySQL

### 3.1 通用 compose 片段（项目里几乎一模一样）

```yaml
mysql:
  image: mysql:5.7              # 或 mysql:8.0.26 / 8.0.35 / 8.0.42
  container_name: mysql
  restart: always
  ports:
    - 3306:3306
  volumes:
    - ./mysql/mydir:/mydir
    - ./mysql/data:/var/lib/mysql
    - ./mysql/conf/my.cnf:/etc/my.cnf
    - ./mysql/source:/docker-entrypoint-initdb.d/
  environment:
    MYSQL_ROOT_PASSWORD: root
  networks:
    - mall_net
```

### 3.2 my.cnf 关键参数

`nacos1.4/mysql/conf/my.cnf` 为标准模板：

```ini
[mysqld]
user=root
default-storage-engine=INNODB
character-set-client-handshake=FALSE
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
lower_case_table_names=1          # 表名不区分大小写（Linux 必加，否则迁移大小写问题）
[client]
default-character-set=utf8mb4
[mysql]
default-character-set=utf8mb4
```

> ⚠ `lower_case_table_names=1` **必须在 mysql 首次初始化时设置**，MySQL 8 要求和 data 目录初始化时一致，否则启动失败（"Different lower_case_table_names settings"）。如果是已有 data 目录，**不能中途改这个值**。

`javal4mall/mysql/conf.d/docker.cnf` 为高级模板（带 binlog、为 Canal 服务）：

```ini
[mysqld]
skip-host-cache
skip-name-resolve
default-authentication-plugin=mysql_native_password   # MySQL 8 兼容老客户端
sql_mode=STRICT_TRANS_TABLES,...
default-time-zone='+8:00'
log-bin=mysql-binlog                                  # ← Canal 必须
binlog-format=ROW                                     # ← Canal 必须
server_id=1                                           # ← Canal 必须
max_connections=1000
```

### 3.3 初始化 SQL 加载流程图

```
docker compose up
        │
        ▼
┌──────────────────────────┐
│ mysql 容器启动            │
│ entrypoint.sh 检查        │
│ /var/lib/mysql 是否为空    │
└─────────────┬────────────┘
              │ 空（首次启动）
              ▼
   ┌────────────────────────────────┐
   │ 初始化系统表 + 设置 root 密码   │
   └─────────────┬──────────────────┘
                 ▼
   ┌────────────────────────────────────────────────┐
   │ 按字母顺序执行 /docker-entrypoint-initdb.d/ 内 │
   │   ./mysql/source/*.sql                         │
   │   （nacos.sql / xxl-job.sql / ry_*.sql ...）   │
   └─────────────┬──────────────────────────────────┘
                 ▼
   ┌────────────────────────────────┐
   │ 监听 3306，对外提供服务         │
   └────────────────────────────────┘

⚠ 二次启动：data 非空，跳过 initdb.d，不再执行 SQL
```

---

## 4. Nacos

### 4.1 三种典型版本

| 项目                       | 镜像                          | MySQL 版本 | 鉴权          |
| -------------------------- | ----------------------------- | ---------- | ------------- |
| nacos1.4 / docker-mall     | `nacos/nacos-server:1.4.2`    | 5.7        | 默认关闭      |
| nacos2.0.2-mysql5.71       | `nacos/nacos-server:2.0.2`    | 5.7        | 默认关闭      |
| nacos2.0.2-mysql8.0        | `nacos/nacos-server:2.0.3`    | 8.0.26     | 默认关闭      |
| docker-hmall / docker-spzx | `nacos/nacos-server:v2.2.2`   | 8.0.42     | 开启          |
| javal4mall                 | `nacos/nacos-server:v2.3.2`   | 8.0.35     | 开启 + 自定义 |

### 4.2 端口知识点（**这是项目里最容易出错的地方**）

- **Nacos 1.x**：只需 `8848`
- **Nacos 2.x**：除 8848 外还引入了 gRPC 端口：
  - `9848`：客户端 gRPC（offset = 8848 + 1000）
  - `9849`：集群间 gRPC（offset = 8848 + 1001）

```
Nacos 2.x 端口家族
┌──────────────────────────────────────┐
│ 8848  HTTP/控制台/OpenAPI             │
│ 9848  client gRPC（SDK 必须能通）     │
│ 9849  raft / 集群 gRPC（集群必须）    │
└──────────────────────────────────────┘
```

> 项目里 `nacos2.0.2-mysql5.71` 和 `nacos2.0.2-mysql8.0` **只映射了 8848**，没映射 9848 —— 这是**典型错误**。如果客户端用的是 nacos-client 2.x，会在 9848 上一直连接失败、日志疯狂报 grpc 异常。`docker-hmall` 与 `docker-spzx` 这两个项目是对的，映射了 8848+9848。

### 4.3 数据库初始化 + Nacos 启动多文件协作流程图

```
                       ┌──────────────────────────────┐
                       │ docker compose up -d         │
                       └──────────────┬───────────────┘
                                      │
        ┌─────────────────────────────┴────────────────────────────┐
        ▼                                                          ▼
┌──────────────────────┐                            ┌──────────────────────────┐
│ mysql 容器先启动      │                            │ nacos 容器（depends_on   │
│ ./mysql/conf/my.cnf  │                            │  mysql；容器启动顺序）   │
│ ./mysql/source/      │  initdb.d                  │                          │
│  └ nacos.sql ────────┼──► 建 nacos 库 + 17 张表    │                          │
│ ./mysql/data 持久化   │                            │                          │
└──────────┬───────────┘                            └─────────────┬────────────┘
           │ 监听 3306                                            │ 监听 8848/9848
           │                                                      │
           └──────────────────  TCP/3306  ◄────────────────────────┘
                  环境变量：MYSQL_SERVICE_HOST=mysql
                            MYSQL_SERVICE_DB_NAME=nacos
                            MYSQL_SERVICE_USER=root
                            MYSQL_SERVICE_PASSWORD=root
                            SPRING_DATASOURCE_PLATFORM=mysql

⚠ depends_on 不等 mysql "ready"。
   mysql 慢启动时 nacos 会因连不上 DB 直接退出 ——
   nacos 有 restart: always，最终会自愈，但日志会刷大量异常。
   正确做法：mysql 增加 healthcheck，nacos 用 depends_on.condition: service_healthy。
```

### 4.4 鉴权环境变量（Nacos 2.2+）

```yaml
- NACOS_AUTH_ENABLE=true                              # 开启鉴权
- NACOS_AUTH_IDENTITY_KEY=example                     # 服务端身份 key
- NACOS_AUTH_IDENTITY_VALUE=example                   # 服务端身份 value
- NACOS_AUTH_TOKEN=<base64,长度>=32 字符>             # JWT secret，必须 base64 后 >=32 字符
```

> Nacos 2.3+ 还推荐使用新版变量名：
> ```
> NACOS_CORE_AUTH_PLUGIN_NACOS_TOKEN_SECRET_KEY
> NACOS_CORE_AUTH_SERVER_IDENTITY_KEY
> NACOS_CORE_AUTH_SERVER_IDENTITY_VALUE
> ```
> javal4mall 使用了新版变量，但同时 token-secret-key 已通过 `NACOS_CORE_AUTH_PLUGIN_NACOS_TOKEN_SECRET_KEY` 配置（base64 解码后必须 ≥ 32 字节）。

---

## 5. Redis

### 5.1 标准片段

```yaml
redis:
  image: redis:6.2.7                               # 或 redis:5.0.0、redis:7.0
  container_name: redis
  restart: always
  command: redis-server --requirepass 123456       # ⭐ 在 command 里加密码
  ports:
    - 6379:6379
  volumes:
    - ./redis/data:/data                            # AOF/RDB 持久化目录
```

### 5.2 关键点

- 官方 redis 镜像默认 **不开持久化、不带配置文件**，命令行参数足以覆盖大多数场景。
- 想开 AOF：`command: redis-server --requirepass 123456 --appendonly yes`。
- 想用自定义 redis.conf：再挂一行 `./redis/redis.conf:/etc/redis/redis.conf`，command 改成 `redis-server /etc/redis/redis.conf`。
- 项目里大量用 `--requirepass 123456` 这种弱密码，**生产必须改强密码并改成 `--requirepass "$REDIS_PASSWORD"` 通过 env_file 注入**。

---

## 6. MongoDB

### 6.1 标准片段（来自 docker-mall / ry_v1 / ry_v2 / NFTurbo）

```yaml
mongodb:
  image: mongo:4.4
  container_name: mongo
  restart: always
  environment:
    - TZ=Asia/Shanghai
    - MONGO_INITDB_DATABASE=admin
    - MONGO_INITDB_ROOT_USERNAME=admin
    - MONGO_INITDB_ROOT_PASSWORD=admin
  ports:
    - 27017:27017
  volumes:
    - ./mongo/data:/data/db
    - ./mongo/log:/var/log/mongodb
    - ./mongo/config/mongod.conf:/etc/mongod.conf
    - ./mongo/init:/docker-entrypoint-initdb.d
  command: ["--config", "/etc/mongod.conf"]
```

### 6.2 mongod.conf 关键

```yaml
processManagement:
  fork: false                # ⭐ Docker 内必须 false，否则容器立刻退出
security:
  authorization: enabled     # 配合 MONGO_INITDB_ROOT_USERNAME/PASSWORD
storage:
  engine: wiredTiger
  wiredTiger:
    engineConfig:
      cacheSizeGB: 1         # 限制内存，避免容器 OOM
```

### 6.3 命令 + 配置 + 初始化脚本协作

```
docker compose up
       │
       ▼
mongo 容器 entrypoint
       │
       │ MONGO_INITDB_ROOT_USERNAME/PASSWORD 触发
       │ 创建 admin 库 + root 用户
       ▼
读取 command: ["--config","/etc/mongod.conf"]
       │
       │ → /etc/mongod.conf 来自 ./mongo/config/mongod.conf
       │   security.authorization=enabled
       │
       ▼
data 目录为空时执行
   ./mongo/init/*.js  →  /docker-entrypoint-initdb.d/
       │
       ▼
监听 27017
```

---

## 7. RabbitMQ（含延迟消息插件）

### 7.1 项目里几乎一模一样的片段

```yaml
rabbitmq:
  image: rabbitmq:3.8.3-management
  container_name: rabbitmq
  restart: always
  ports:
    - 15672:15672                    # 管理控制台
    - 5672:5672                      # AMQP
  volumes:
    - ./rabbitmq/data:/var/lib/rabbitmq
    - ./rabbitmq/plugins:/opt/plugins
  command: bash -c "cp /opt/plugins/*.ez /plugins && \
                    rabbitmq-plugins enable --offline rabbitmq_delayed_message_exchange && \
                    /usr/local/bin/docker-entrypoint.sh rabbitmq-server"
  environment:
    RABBITMQ_DEFAULT_USER: admin
    RABBITMQ_DEFAULT_PASS: admin
```

### 7.2 延迟队列插件多文件协作流程图

```
宿主机                                       容器内
┌─────────────────────────────┐         ┌────────────────────────────────────────┐
│ ./rabbitmq/plugins/         │         │ /opt/plugins/                          │
│   rabbitmq_delayed_message_  ├────────►│   rabbitmq_delayed_message_exchange.ez │
│   exchange.ez                │  挂载    └───────────────┬────────────────────────┘
└─────────────────────────────┘                          │
                                                         ▼ command 第 1 步
                                          cp /opt/plugins/*.ez  /plugins/
                                                         │
                                                         ▼ command 第 2 步
                                          rabbitmq-plugins enable --offline \
                                              rabbitmq_delayed_message_exchange
                                                         │
                                                         ▼ command 第 3 步
                                          docker-entrypoint.sh rabbitmq-server
                                                         │
                                                         ▼
                                          监听 5672 / 15672
                                          支持 x-delayed-message 类型交换机
```

> ⚠ 关键点：插件文件必须**版本匹配 rabbitmq 主版本**。`rabbitmq:3.8.3-management` 必须放 `rabbitmq_delayed_message_exchange-3.8.x.ez`，放 3.9/3.10 的会启动失败。

---

## 8. Elasticsearch + Kibana

### 8.1 docker-hmall（单机+无鉴权，老版本）

```yaml
elasticsearch:
  image: elasticsearch:7.12.1
  container_name: elasticsearch
  ports:
    - 9200:9200    # HTTP
    - 9300:9300    # TCP，节点间
  volumes:
    - ./elasticsearch/data:/usr/share/elasticsearch/data
    - ./elasticsearch/plugins:/usr/share/elasticsearch/plugins
  environment:
    - ES_JAVA_OPTS=-Xms512m -Xmx512m
    - discovery.type=single-node
  networks:
    - mall_net

kibana:
  image: kibana:7.12.1
  environment:
    - ELASTICSEARCH_HOSTS=http://elasticsearch:9200   # 通过 container_name 解析
  ports:
    - 5601:5601
  depends_on:
    - elasticsearch
```

### 8.2 javal4mall（带 xpack 鉴权）

```yaml
mall4cloud-elasticsearch:
  image: elasticsearch:7.17.21
  environment:
    - discovery.type=single-node
    - ES_JAVA_OPTS=-Xms512m -Xmx512m
    - ELASTICSEARCH_USERNAME=elastic
    - ELASTIC_PASSWORD=80jpnH4.r5g
    - xpack.security.enabled=true
  volumes:
    - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    - ./elasticsearch/data:/usr/share/elasticsearch/data
    - ./elasticsearch/plugins:/usr/share/elasticsearch/plugins
```

### 8.3 ES + Kibana 协作图

```
┌──────────────────────┐                ┌────────────────────────┐
│  Browser :5601       │ ───HTTP────►   │  kibana:5601            │
└──────────────────────┘                │  ELASTICSEARCH_HOSTS=   │
                                        │  http://elasticsearch:  │
                                        │       9200              │
                                        └────────────┬────────────┘
                                                     │
                                                     ▼ HTTP via mall_net
                                        ┌────────────────────────┐
                                        │ elasticsearch:9200      │
                                        │ discovery=single-node   │
                                        │ data → ./elasticsearch/ │
                                        │        data            │
                                        │ plugins → ik 分词器 等  │
                                        └────────────────────────┘
```

### 8.4 注意点

- `./elasticsearch/data` 在宿主机上必须为 ES 运行用户（uid=1000）可写：`chown -R 1000:1000 ./elasticsearch`。
- `vm.max_map_count` 在宿主机必须 ≥ 262144（`sysctl -w vm.max_map_count=262144`），否则 ES 启动报错。
- IK 分词器通过 `plugins` 挂载，重启 ES 即生效。

---

## 9. MinIO

### 9.1 ry_v1 / ry_v2 写法（双端口）

```yaml
minio:
  image: minio/minio:RELEASE.2024-04-18T19-09-19Z
  container_name: minio
  restart: always
  command: server --console-address ":9001" /data
  ports:
    - 9000:9000   # S3 API
    - 9001:9001   # 控制台
  volumes:
    - ./minio/data:/data
    - ./minio/config:/root/.minio
  environment:
    - MINIO_ROOT_USER=minioadmin
    - MINIO_ROOT_PASSWORD=minioadmin
```

### 9.2 知识点

- 新版 MinIO 强制把控制台与 API 分到两个端口，启动时**必须用 `--console-address ":9001"`** 否则控制台端口会随机变化。
- `MINIO_ROOT_PASSWORD` 长度必须 ≥ 8，否则 `minioadmin` 这种弱密码新版会直接拒启。
- `MINIO_BROWSER_REDIRECT_URL=http://192.168.x.x:9001`（javal4mall 用）解决控制台登录后跳到内网域名的问题。

---

## 10. RocketMQ

### 10.1 ry_v2 项目里的三件套

```yaml
rocketmq-namesrv:
  image: apache/rocketmq:5.2.0
  ports:
    - 9876:9876
  command: ["sh", "mqnamesrv"]
  environment:
    JAVA_OPT_EXT: "-Duser.home=/home/rocketmq -Xms512M -Xmx512M -Xmn256M"

rocketmq-broker:
  image: apache/rocketmq:5.2.0
  ports:
    - 10909:10909          # VIP 通道
    - 10911:10911          # 主对外端口
  volumes:
    - ./rocketmq/broker/conf/broker.conf:/etc/rocketmq/broker.conf
  command: ["sh","mqbroker","-c","/etc/rocketmq/broker.conf",
            "-n","rocketmq-namesrv:9876","autoCreateTopicEnable=true"]
  depends_on:
    - rocketmq-namesrv

rocketmq-dashboard:
  image: apacherocketmq/rocketmq-dashboard:2.0.0
  ports:
    - 8080:8080
  environment:
    JAVA_OPTS: "-Drocketmq.namesrv.addr=rocketmq-namesrv:9876 \
                -Dcom.rocketmq.sendMessageWithVIPChannel=false"
  depends_on:
    - rocketmq-namesrv
```

### 10.2 三组件协作流程

```
                   ┌────────────────────────┐
                   │ rocketmq-namesrv:9876   │  (路由注册中心)
                   └─────────▲────────▲──────┘
                  心跳/注册   │        │   查询/订阅
                            │        │
        ┌─────────────────┐ │        │ ┌──────────────────────┐
        │ rocketmq-broker │─┘        └─│ rocketmq-dashboard    │
        │  10909/10911    │            │  Web :8080           │
        │  broker.conf     │            │  -Drocketmq.namesrv. │
        │  brokerIP1=宿主 │             │     addr=...:9876    │
        └────────┬────────┘            └──────────────────────┘
                 │
                 ▼
       业务 Producer/Consumer
       NameSrv: 宿主机:9876
```

### 10.3 broker.conf 重点（容器内 RocketMQ 必坑）

```properties
brokerClusterName = DefaultCluster
brokerName        = broker-a
brokerId          = 0
deleteWhen        = 04
fileReservedTime  = 48
brokerRole        = ASYNC_MASTER
flushDiskType     = ASYNC_FLUSH
# ⭐ 容器场景必加：注册到 NameServer 的 IP
brokerIP1         = <宿主机IP，或对客户端可见的 IP>
# ⭐ 默认 1G CommitLog，开发机可以调小
mapedFileSizeCommitLog = 1073741824
```

> 不写 `brokerIP1` 时，broker 会把容器内部 IP（172.x.x.x）注册到 NameServer，外部 Producer 连得到 NameServer 但发不出消息——这是 Docker 部署 RocketMQ 最高频的坑。

---

## 11. Seata

### 11.1 sfbx 项目（注册中心走 Nacos）

```yaml
seata-server:
  image: seataio/seata-server:1.5.2
  ports:
    - 7091:7091   # 控制台
    - 8091:8091   # TC
  volumes:
    - ./seata-server/resources/:/seata-server/resources
  environment:
    SEATA_IP: 192.168.12.129    # 注册到 Nacos 的对外 IP（同 RocketMQ 的 brokerIP1）
  depends_on:
    - nacos
```

对应 `seata-server/resources/application.yml`：

```yaml
seata:
  config:
    type: nacos
    nacos:
      server-addr: nacos:8848
      group: DEFAULT_GROUP
      namespace: ""
      dataId: seataServer.properties
      username: nacos
      password: nacos
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: nacos:8848
```

### 11.2 javal4mall（注册=file，存储=db）

```yaml
seata:
  store:
    mode: db
    db:
      driverClassName: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://192.168.1.46:3306/mall4cloud_seata?...
      user: root
      password: root
```

### 11.3 Seata 文件依赖关系图

```
┌──────────────────────────────────────────────┐
│ docker-compose.yaml                          │
│  seata-server:                               │
│    volumes:                                  │
│      ./seata/application.yml ──┐              │
│      → /seata-server/resources/application.yml│
│    env: SEATA_IP=宿主机IP                    │
└──────────────────────┬───────────────────────┘
                       ▼
        ┌────────────────────────────────┐
        │ application.yml                 │
        │  registry.type = nacos          │   ─► 把 seata-server 服务注册到
        │  config.type = nacos            │      nacos:8848，业务方 TC 客户端
        │  (或 store.mode=db, db.url=...) │      通过 Nacos 拿到 TC 地址
        └────────────────────────────────┘
                       │
                       ▼
            Nacos: 注册 seata-server (8091)
                       │
                       ▼
            业务 RM/TM 通过 Nacos 发现 TC
```

---

## 12. Canal

### 12.1 javal4mall（canal-server 单实例）

```yaml
mall4cloud-canal:
  image: canal/canal-server:v1.1.7
  network_mode: "host"
  volumes:
    - ./canal/conf/example:/home/admin/canal-server/conf/example
    - ./canal/conf/canal.properties:/home/admin/canal-server/conf/canal.properties
    - ./canal/logs:/home/admin/canal-server/logs
```

### 12.2 Canal 配置链路（多文件！）

```
┌────────────────────┐
│ MySQL              │
│ my.cnf:            │
│  log-bin=mysql-binlog  ── 必开                  ┐
│  binlog-format=ROW                              ├─► 没开 binlog，canal 什么也读不到
│  server_id=1                                    ┘
└─────────┬──────────┘
          │ MySQL Slave 协议（IO + SQL）
          ▼
┌──────────────────────────────────────────────┐
│ canal-server  /home/admin/canal-server/      │
│                                              │
│  conf/canal.properties                       │
│   canal.port = 11111                         │
│   canal.serverMode = rocketMQ  ──┐           │
│   rocketmq.namesrv.addr = ...    │           │
│                                  │           │
│  conf/example/instance.properties             │
│   canal.instance.master.address=mysql:3306    │
│   canal.instance.dbUsername=canal             │
│   canal.instance.dbPassword=canal             │
│   canal.instance.filter.regex=db\\..*         │
│   canal.mq.topic=order_binlog                 │
└──────────────┬────────────────────────────────┘
               │ 解析 binlog
               ▼
        canal.serverMode：
          tcp        → Java 客户端直连 11111
          kafka      → 推 Kafka topic
          rocketMQ   → 推 RocketMQ topic   ← 项目使用
          rabbitMQ   → 推 RabbitMQ
```

> ⚠ 业务用 canal 的最常见踩坑：
> 1. MySQL `log_bin` / `binlog_format=ROW` 没开 —— canal 直接连不上。
> 2. MySQL 没建专用账号 + GRANT REPLICATION SLAVE / CLIENT 权限。
> 3. `canal.instance.filter.regex` 用 `.` 而不是 `\\.`，匹配不到。

---

## 13. InfluxDB

### 13.1 sfbx 项目片段

```yaml
influxdb:
  image: influxdb:1.8.0
  container_name: influxdb
  ports:
    - 9083:8083   # Web admin（1.0 老遗留）
    - 8086:8086   # HTTP API
    - 8088:8088   # RPC backup/restore
  privileged: true
  volumes:
    - ./influxdb/data/influxdb:/var/lib/influxdb
    - ./influxdb/config/influxdb.conf:/etc/influxdb/influxdb.conf
```

### 13.2 知识点

- **1.8 是 1.x 最后一个稳定版**；如果用 2.x，认证、存储引擎、CLI 完全不同。
- 主端口只用 `8086`，老的 8083（admin）实际已弃用。
- 配置文件路径 `/etc/influxdb/influxdb.conf` 与镜像默认一致。
- 数据目录 `/var/lib/influxdb/{meta,data,wal}`，data 用 TSM 存储引擎。

---

## 14. XXL-Job

### 14.1 sfbx 项目片段

```yaml
xxl-job:
  image: xuxueli/xxl-job-admin:2.1.2
  container_name: xxl-job-admin
  ports:
    - 8280:8080
  volumes:
    - ./xxl-job/data:/data/applogs
  environment:
    PARAMS: "--spring.datasource.url=jdbc:mysql://mysql:3306/xxl-job?\
             useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&\
             serverTimezone=Asia/Shanghai \
             --spring.datasource.username=root \
             --spring.datasource.password=pass"
  depends_on:
    - mysql
```

### 14.2 知识点

- 通过 `PARAMS` 把 Spring Boot 启动参数注入容器，避免改镜像里的 application.properties。
- 依赖 MySQL 中已存在 `xxl-job` 库（项目里 `mysql/source/xxl-job.sql` 就是初始化脚本）。
- 浏览器访问 `http://宿主机:8280/xxl-job-admin`，默认账号 `admin/123456`。
- **`xuxueli/xxl-job-admin:2.1.2` 是 2018 年版本，已停止维护**。生产应至少升 `2.4.x` 以上。

---

## 15. 现有 docker-compose 文件中发现的错误清单

以下错误是在阅读 `docker/docker-compose/` 真实文件时发现的，逐项给出**错误位置 → 错误原因 → 正确写法**。

### 错误 1：NFTurbo/docker-compose/docker-compose.yaml — YAML 缩进错误，文件无法解析

```yaml
# 第 20~22 行（错误）
  #nacos服务脚本
 nacos:                            # ←—— 这一行只有 1 个空格缩进
    image: nacos/nacos-server:v2.2.2
```

`services:` 下所有服务必须保持同样 2 空格缩进。这里 `nacos:` 比 `mysql:` 少 1 个空格，**`docker compose config` 直接报 `mapping values are not allowed in this context`**，整个文件根本起不来。

**修正**：

```yaml
  #nacos服务脚本
  nacos:                           # 改成 2 个空格
    image: nacos/nacos-server:v2.2.2
```

### 错误 2：NFTurbo — nacos 服务挂在了未声明的网络上

```yaml
nacos:
  ...
  networks:
    - nacos2_net           # ←—— 整个文件只声明了 mall_net
```

文件底部 `networks:` 只有 `mall_net`，引用了 `nacos2_net` 会导致 compose 报 `network nacos2_net is declared as external, but could not be found`。

**修正**：把 `nacos2_net` 改成 `mall_net`，否则 nacos 与 mysql/redis/mongo/rabbitmq 不在同一网络，**互相也 DNS 解析不到**（即 `MYSQL_SERVICE_HOST=mysql` 会失败）。

### 错误 3：sfbx/docker-compose.yaml — JVM_MMS 拼写错误

```yaml
nacos:
  environment:
    JVM_XMS: 512m
    JVM_MMS: 512m         # ←—— ❌ 没有 JVM_MMS 这个参数
    JVM_XMN: 256m
```

Nacos 启动脚本只识别 `JVM_XMS / JVM_XMX / JVM_XMN`。`JVM_MMS` 会被忽略，导致最大堆没设上，默认走 JVM 默认值。

**修正**：

```yaml
    JVM_XMX: 512m
```

### 错误 4：nacos2.0.2-mysql5.71 与 nacos2.0.2-mysql8.0 — 漏映射 gRPC 端口 9848

```yaml
nacos:
  image: nacos/nacos-server:2.0.2     # 或 2.0.3
  ports:
    - 8848:8848         # ←—— 只有 8848
```

Nacos 2.x 客户端 SDK 默认走 gRPC，必须能连 9848（= 8848 + 1000）。

**修正**：

```yaml
  ports:
    - 8848:8848
    - 9848:9848
    - 9849:9849         # 集群部署时也要开
```

### 错误 5：docker-mall/mall — Nacos 1.4.2 与 mysql:5.7 默认编码冲突

```yaml
nacos:
  image: nacos/nacos-server:1.4.2
  ...
  - MYSQL_SERVICE_DB_NAME=nacos
```

Nacos 1.4.2 内嵌的 mysql-connector 是 5.1.x，连 MySQL 5.7 默认 url 是不带 `useSSL=false&serverTimezone=Asia/Shanghai` 的，启动可能报 `SSL connection error` 或 `serverTimezone unrecognized`。

**修正**：环境变量加上扩展参数：

```yaml
  - MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&serverTimezone=Asia/Shanghai
```

或者把 MySQL 升到 8.0，并使用 nacos 2.x。

### 错误 6：所有 mysql 8.x 项目 — 没设 default-authentication-plugin

`nacos2.0.2-mysql8.0/mysql/conf/my.cnf` / `docker-hmall/mysql/conf/my.cnf` / `docker-spzx/mysql/conf/my.cnf` 与 `nacos1.4/mysql/conf/my.cnf` 完全一致，**都没有 `default-authentication-plugin=mysql_native_password`**。

MySQL 8 默认是 `caching_sha2_password`，老客户端（Nacos 1.4.2、xxl-job-admin 2.1.2、seata 1.5.2 旧驱动等）会报 `Authentication plugin 'caching_sha2_password' cannot be loaded`。

**修正**（在 my.cnf 的 `[mysqld]` 段加）：

```ini
default-authentication-plugin=mysql_native_password
```

`javal4mall/mysql/conf.d/docker.cnf` 这一项是正确的，可以参考。

### 错误 7：javal4mall — `network_mode: host` 与 `expose` 同时存在

```yaml
mall4cloud-mysql:
  network_mode: "host"
  expose:
    - 3306              # ❌ host 模式下 expose 完全无效
```

`network_mode: host` 已经让容器与宿主机共享网络栈，`ports` 和 `expose` 都被 Docker 忽略。这里写了只是噪音。

**修正**：删掉 `expose:` 块。如需要给非 host 容器联通，应该改回 bridge 网络 + ports/expose。

### 错误 8：javal4mall — 数据源 host 硬编码内网 IP，强绑机器

```yaml
- MYSQL_SERVICE_HOST=192.168.1.46
```

如果不是 `192.168.1.46` 这台机器部署，nacos 启动直接报错。

**修正**：在 host 模式下应使用 `127.0.0.1`；或者改成 bridge 网络 + `MYSQL_SERVICE_HOST=mall4cloud-mysql`，借助 docker DNS 解决。

### 错误 9：pd-auth — MYSQL_SERVICE_DB_NAME 与 SQL 初始化脚本不一致

```yaml
nacos:
  - MYSQL_SERVICE_DB_NAME=nacos_config
```

而 `pd-auth/mysql/source/nacos-mysql.sql` 默认建的库名通常是 `nacos`（官方脚本名），需要校对 SQL 第一行：

```sql
CREATE DATABASE IF NOT EXISTS nacos_config ...    -- 必须与上面 env 一致
```

如果脚本里写的是 `nacos`，nacos 容器会因为找不到 `nacos_config` 库而启动失败。

**修正**：二选一对齐——要么把 env 改成 `nacos`，要么改 SQL 脚本里的库名为 `nacos_config`。

### 错误 10：docker-hmall / docker-mall / sfbx — RabbitMQ 插件路径假定

```yaml
command: bash -c "cp /opt/plugins/*.ez /plugins && ..."
```

`rabbitmq:3.8.3-management` 的 enabled-plugins 扫描目录其实是 `/opt/rabbitmq/plugins`（或软链到 `/plugins`）。 Docker 镜像内 `/plugins` 是一个软链到 `${RABBITMQ_HOME}/plugins`。大多数情况下能 work，但**一旦镜像换成 3.9+/3.12+ 该软链不存在**，命令直接失败。

**修正**（更鲁棒的写法）：

```yaml
command: bash -c "cp /opt/plugins/*.ez $(rabbitmq-plugins directories -s | awk -F= '/plugins_dir/{print $2}') && \
                  rabbitmq-plugins enable --offline rabbitmq_delayed_message_exchange && \
                  /usr/local/bin/docker-entrypoint.sh rabbitmq-server"
```

或者最稳的办法：**自己写一层 Dockerfile**，在构建阶段就把 `.ez` 拷到正确目录并 enable，避免运行时副作用。

### 错误 11：所有用到 depends_on 的 nacos / seata / xxl-job — 不等 mysql ready

```yaml
nacos:
  depends_on:
    - mysql
```

`depends_on` **只等容器进程启动，不等 mysqld 真正监听 3306**。MySQL 首次启动要做初始化（包括 source 里几十 MB 的 SQL），通常需 20~60 秒。nacos 会在 0~2 秒之内开始连接，必败，容器退出，再 `restart: always`，反复几次后才稳定。

**正确做法**（Compose 3.8+ 都支持）：

```yaml
mysql:
  healthcheck:
    test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-proot"]
    interval: 5s
    timeout: 3s
    retries: 30

nacos:
  depends_on:
    mysql:
      condition: service_healthy
```

### 错误 12：ry_v2 — RocketMQ broker 缺 brokerIP1

```yaml
rocketmq-broker:
  command: ["sh", "mqbroker", "-c", "/etc/rocketmq/broker.conf", "-n", "rocketmq-namesrv:9876", "autoCreateTopicEnable=true"]
```

`broker.conf` 里如果没显式设 `brokerIP1=宿主机IP`，broker 会把容器内 IP 注册到 NameServer，导致**宿主机外部 Producer/Consumer 可以连 NameServer，但拿到的 broker 地址是 172.x 内网，发不出/消费不到消息**。

**修正**：在 `./rocketmq/broker/conf/broker.conf` 中加：

```properties
brokerIP1 = <宿主机对客户端可见的 IP>
brokerIP2 = <宿主机对客户端可见的 IP>
```

### 错误 13：docker-spzx — 没声明 redis 等其他中间件，但项目实际依赖

`docker-spzx/spzx/docker-compose.yaml` 里只有 `mysql + nacos`，但项目 `spzx/redis` 目录已存在，说明本应该有 redis 服务。**当前 compose 实际并不会启动 redis**，缺漏。

**修正**：补上 redis 服务块（参考 docker-hmall 的写法），并把 networks 加入 `nacos2_net`。

### 错误 14：所有项目 — `version: "3.x"` 过时但无害

Compose v2（即 `docker compose` 而不是 `docker-compose`）**完全忽略** `version` 字段，只是兼容性保留。不算错误，但**新项目可以直接删掉**让告警消失。

### 错误 15：sfbx — extnetwork 固定子网且固定 IP，与现网冲突风险

```yaml
networks:
  extnetwork:
    name: extnetwork
    ipam:
      config:
        - subnet: 172.21.0.0/16
```

`172.21.0.0/16` 在某些公司内网/WireGuard/VPN 路由表里已使用，会导致**整个宿主机断网**。除非有明确隔离需求，没必要在单机 compose 里固定 IP。

**修正**：改为默认 bridge 即可，让 Docker 自己分子网：

```yaml
networks:
  extnetwork:
    driver: bridge
```

---

## 附：所有项目的中间件能力速查矩阵

| 项目                  | MySQL | Redis | Nacos | Mongo | RabbitMQ | ES+Kibana | MinIO | RocketMQ | Seata | Canal | InfluxDB | XXL-Job |
| --------------------- | ----- | ----- | ----- | ----- | -------- | --------- | ----- | -------- | ----- | ----- | -------- | ------- |
| docker-hmall          | 8.0.42| 6.2.7 | 2.2.2 |       | 3.8.3-mgr| 7.12.1    |       |          |       |       |          |         |
| docker-mall           | 5.7   | 6.2.7 | 1.4.2 | 4.4   | 3.8.3-mgr|           |       |          |       |       |          |         |
| docker-spzx           | 8.0.42|       | 2.2.2 |       |          |           |       |          |       |       |          |         |
| javal4mall            | 8.0.35| 7.0   | 2.3.2 |       |          | 7.17.21   | 2024  | 5.2.0    | 2.0.0 | 1.1.7 |          |         |
| nacos1.4              | 5.7   |       | 1.4.2 |       |          |           |       |          |       |       |          |         |
| nacos2.0.2-mysql5.71  | 5.7   |       | 2.0.2 |       |          |           |       |          |       |       |          |         |
| nacos2.0.2-mysql8.0   | 8.0.26|       | 2.0.3 |       |          |           |       |          |       |       |          |         |
| NFTurbo               | 5.7   | 6.2.7 | 2.2.2 | 4.4   | 3.8.3-mgr|           |       |          |       |       |          |         |
| pd-auth               | 5.7   | 6.2.7 | 1.4.2 |       |          |           |       |          |       |       |          |         |
| ry-demo               | 5.7   | 6.2.7 |       |       |          |           |       |          |       |       |          |         |
| ry_v1                 | 5.7   | 6.2.7 |       | 4.4   |          |           | 2024  |          |       |       |          |         |
| ry_v2                 | 5.7   | 6.2.7 |       | 4.4   |          |           | 2024  | 5.2.0    |       |       |          |         |
| sfbx                  | 5.7   | 5.0.0 | 2.0.2 |       | 3.8.3-mgr|           |       |          | 1.5.2 |       | 1.8.0    | 2.1.2   |

---

## 最佳实践小结

1. **目录约定固化**：每个中间件单独子目录，子目录里 `data / conf / source(initdb) / logs` 四件套。
2. **配置文件总是显式挂载**，不要靠镜像默认配置；my.cnf、mongod.conf、broker.conf、application.yml 必须可被 git 追踪。
3. **数据卷一定是 bind mount**（`./xxx`）而不是 named volume，方便备份与迁移。
4. **`depends_on` 必须配合 `healthcheck`**，否则容器启动顺序不可靠。
5. **2.x Nacos 必须开 9848 端口**。
6. **MySQL 8 必须设 `default-authentication-plugin=mysql_native_password`** 兼容旧客户端。
7. **RocketMQ 容器必须设 `brokerIP1=宿主机IP`**。
8. **生产环境不要 `network_mode: host`**，除非确实需要绕开 NAT。
9. **不要把弱密码（root/root、admin/admin、minioadmin/minioadmin）写进 compose**，用 `.env + env_file` 或 secret 注入。
10. **`version:` 字段在 Compose v2 已废弃**，新文件可省略。
