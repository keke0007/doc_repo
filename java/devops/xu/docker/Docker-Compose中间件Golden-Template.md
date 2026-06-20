# Docker Compose 中间件 Golden Template

> 本文档把当前 `docker-compose/` 目录中各项目反复使用的中间件安装配置整理成一份可复用模板。新项目优先复制本文中的目录结构、`.env`、`docker-compose.yaml`，再按需删除不用的服务。

## 1. 使用方式

1. 新建项目中间件目录，例如 `docker-compose/my-project/`。
2. 按本文第 2 节创建目录。
3. 按本文第 3 节创建 `.env`。
4. 按本文第 4 节创建 `docker-compose.yaml`。
5. 修改 `.env` 中密码、宿主机 IP、数据库名、镜像版本。
6. 只保留当前项目需要的服务。
7. 启动：

```bash
docker compose up -d
docker compose ps
docker compose logs -f nacos
```

停止：

```bash
docker compose down
```

停止并删除数据卷/挂载数据前要先确认数据可丢弃。本文模板主要使用 bind mount，数据目录在当前项目目录下。

## 2. 推荐目录结构

```text
my-project/
├── .env
├── docker-compose.yaml
├── mysql/
│   ├── conf/my.cnf
│   ├── data/
│   └── source/
├── nacos/logs/
├── redis/data/
├── mongo/
│   ├── config/mongod.conf
│   ├── data/
│   ├── init/
│   └── log/
├── rabbitmq/
│   ├── data/
│   └── plugins/
├── elasticsearch/
│   ├── config/elasticsearch.yml
│   ├── data/
│   └── plugins/
├── minio/
│   ├── config/
│   └── data/
├── rocketmq/
│   ├── namesrv/logs/
│   ├── namesrv/store/
│   ├── broker/conf/broker.conf
│   ├── broker/logs/
│   └── broker/store/
├── seata/application.yml
├── canal/
│   ├── conf/canal.properties
│   ├── conf/example/instance.properties
│   └── logs/
├── influxdb/
│   ├── config/influxdb.conf
│   └── data/
└── xxl-job/data/
```

Linux 下经常需要授权：

```bash
chmod -R 777 mysql/data redis/data mongo/data rabbitmq/data elasticsearch/data minio/data rocketmq influxdb/data xxl-job/data
```

## 3. 标准 `.env`

```dotenv
COMPOSE_PROJECT_NAME=my-project
TZ=Asia/Shanghai

# 宿主机 IP：host 网络模式、Seata、MinIO 回调、RocketMQ brokerIP1 常用
HOST_IP=192.168.1.100

MYSQL_VERSION=8.0.42
MYSQL_ROOT_PASSWORD=root
MYSQL_NACOS_DB=nacos
MYSQL_XXL_JOB_DB=xxl_job

NACOS_VERSION=v2.2.2
NACOS_AUTH_TOKEN=VGhpc0lzTXlDdXN0b21TZWNyZXRLZXkwMTIzNDU2Nzg=
NACOS_AUTH_IDENTITY_KEY=serverIdentity
NACOS_AUTH_IDENTITY_VALUE=security

REDIS_VERSION=7.0
REDIS_PASSWORD=123456

MONGO_VERSION=4.4
MONGO_ROOT_USER=admin
MONGO_ROOT_PASSWORD=admin

RABBITMQ_VERSION=3.8.3-management
RABBITMQ_USER=admin
RABBITMQ_PASSWORD=admin

ELASTIC_VERSION=7.17.21
ELASTIC_PASSWORD=elastic

MINIO_VERSION=RELEASE.2024-04-18T19-09-19Z
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin

ROCKETMQ_VERSION=5.2.0

SEATA_VERSION=2.0.0

CANAL_VERSION=v1.1.7

INFLUXDB_VERSION=1.8.0

XXL_JOB_VERSION=2.1.2
```

## 4. 标准 `docker-compose.yaml`

按需保留服务。默认使用 bridge 网络，服务之间用服务名访问，例如 `mysql:3306`、`redis:6379`、`nacos:8848`。

```yaml
version: "3.8"

services:
  mysql:
    image: mysql:${MYSQL_VERSION}
    container_name: mysql
    restart: always
    ports:
      - "3306:3306"
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/conf/my.cnf:/etc/my.cnf
      - ./mysql/source:/docker-entrypoint-initdb.d/
    environment:
      TZ: ${TZ}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    command:
      - --default-authentication-plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    networks:
      - middleware_net

  nacos:
    image: nacos/nacos-server:${NACOS_VERSION}
    container_name: nacos
    restart: always
    depends_on:
      - mysql
    ports:
      - "8848:8848"
      - "9848:9848"
      - "9849:9849"
    volumes:
      - ./nacos/logs:/home/nacos/logs
    environment:
      TZ: ${TZ}
      MODE: standalone
      SPRING_DATASOURCE_PLATFORM: mysql
      MYSQL_SERVICE_HOST: mysql
      MYSQL_SERVICE_PORT: 3306
      MYSQL_SERVICE_DB_NAME: ${MYSQL_NACOS_DB}
      MYSQL_SERVICE_USER: root
      MYSQL_SERVICE_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      NACOS_AUTH_ENABLE: "true"
      NACOS_AUTH_TOKEN: ${NACOS_AUTH_TOKEN}
      NACOS_AUTH_IDENTITY_KEY: ${NACOS_AUTH_IDENTITY_KEY}
      NACOS_AUTH_IDENTITY_VALUE: ${NACOS_AUTH_IDENTITY_VALUE}
      JVM_XMS: 512m
      JVM_XMX: 512m
      JVM_XMN: 256m
    networks:
      - middleware_net

  redis:
    image: redis:${REDIS_VERSION}
    container_name: redis
    restart: always
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"
    volumes:
      - ./redis/data:/data
    networks:
      - middleware_net

  mongo:
    image: mongo:${MONGO_VERSION}
    container_name: mongo
    restart: always
    ports:
      - "27017:27017"
    volumes:
      - ./mongo/data:/data/db
      - ./mongo/log:/var/log/mongodb
      - ./mongo/config/mongod.conf:/etc/mongod.conf
      - ./mongo/init:/docker-entrypoint-initdb.d
    environment:
      TZ: ${TZ}
      MONGO_INITDB_DATABASE: admin
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_ROOT_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
    command: ["--config", "/etc/mongod.conf"]
    networks:
      - middleware_net

  rabbitmq:
    image: rabbitmq:${RABBITMQ_VERSION}
    container_name: rabbitmq
    restart: always
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - ./rabbitmq/data:/var/lib/rabbitmq
      - ./rabbitmq/plugins:/opt/plugins
    environment:
      TZ: ${TZ}
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
    command: >
      bash -c "if ls /opt/plugins/*.ez >/dev/null 2>&1;
      then cp /opt/plugins/*.ez /plugins && rabbitmq-plugins enable --offline rabbitmq_delayed_message_exchange;
      fi;
      /usr/local/bin/docker-entrypoint.sh rabbitmq-server"
    networks:
      - middleware_net

  elasticsearch:
    image: elasticsearch:${ELASTIC_VERSION}
    container_name: elasticsearch
    restart: always
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/plugins:/usr/share/elasticsearch/plugins
      - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    environment:
      TZ: ${TZ}
      discovery.type: single-node
      ES_JAVA_OPTS: -Xms512m -Xmx512m
      xpack.security.enabled: "false"
    networks:
      - middleware_net

  kibana:
    image: kibana:${ELASTIC_VERSION}
    container_name: kibana
    restart: always
    depends_on:
      - elasticsearch
    ports:
      - "5601:5601"
    environment:
      TZ: ${TZ}
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    networks:
      - middleware_net

  minio:
    image: minio/minio:${MINIO_VERSION}
    container_name: minio
    restart: always
    command: server --console-address ":9001" /data
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - ./minio/data:/data
      - ./minio/config:/root/.minio
    environment:
      TZ: ${TZ}
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
      MINIO_BROWSER_REDIRECT_URL: http://${HOST_IP}:9001
    networks:
      - middleware_net

  rocketmq-namesrv:
    image: apache/rocketmq:${ROCKETMQ_VERSION}
    container_name: rocketmq-namesrv
    restart: always
    ports:
      - "9876:9876"
    volumes:
      - ./rocketmq/namesrv/logs:/home/rocketmq/logs
      - ./rocketmq/namesrv/store:/home/rocketmq/store
    environment:
      JAVA_OPT_EXT: -Duser.home=/home/rocketmq -Xms512M -Xmx512M -Xmn256M
    command: ["sh", "mqnamesrv"]
    networks:
      - middleware_net

  rocketmq-broker:
    image: apache/rocketmq:${ROCKETMQ_VERSION}
    container_name: rocketmq-broker
    restart: always
    depends_on:
      - rocketmq-namesrv
    ports:
      - "10909:10909"
      - "10911:10911"
    volumes:
      - ./rocketmq/broker/logs:/home/rocketmq/logs
      - ./rocketmq/broker/store:/home/rocketmq/store
      - ./rocketmq/broker/conf/broker.conf:/etc/rocketmq/broker.conf
    environment:
      JAVA_OPT_EXT: -Duser.home=/home/rocketmq -Xms512M -Xmx512M -Xmn256M -XX:-AssumeMP
    command: ["sh", "mqbroker", "-c", "/etc/rocketmq/broker.conf", "-n", "rocketmq-namesrv:9876", "autoCreateTopicEnable=true"]
    networks:
      - middleware_net

  rocketmq-dashboard:
    image: apacherocketmq/rocketmq-dashboard:2.0.0
    container_name: rocketmq-dashboard
    restart: always
    depends_on:
      - rocketmq-namesrv
    ports:
      - "8080:8080"
    environment:
      JAVA_OPTS: -Drocketmq.namesrv.addr=rocketmq-namesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false
    networks:
      - middleware_net

  seata-server:
    image: seataio/seata-server:${SEATA_VERSION}
    container_name: seata-server
    restart: always
    depends_on:
      - nacos
      - mysql
    ports:
      - "7091:7091"
      - "8091:8091"
    volumes:
      - ./seata/application.yml:/seata-server/resources/application.yml
    environment:
      TZ: ${TZ}
      STORE_MODE: db
      SEATA_IP: ${HOST_IP}
      SEATA_PORT: 8091
    networks:
      - middleware_net

  canal:
    image: canal/canal-server:${CANAL_VERSION}
    container_name: canal
    restart: always
    depends_on:
      - mysql
    ports:
      - "11111:11111"
    volumes:
      - ./canal/conf/example:/home/admin/canal-server/conf/example
      - ./canal/conf/canal.properties:/home/admin/canal-server/conf/canal.properties
      - ./canal/logs:/home/admin/canal-server/logs
    networks:
      - middleware_net

  influxdb:
    image: influxdb:${INFLUXDB_VERSION}
    container_name: influxdb
    restart: always
    ports:
      - "8086:8086"
      - "8088:8088"
    volumes:
      - ./influxdb/data:/var/lib/influxdb
      - ./influxdb/config/influxdb.conf:/etc/influxdb/influxdb.conf
    networks:
      - middleware_net

  xxl-job:
    image: xuxueli/xxl-job-admin:${XXL_JOB_VERSION}
    container_name: xxl-job-admin
    restart: always
    depends_on:
      - mysql
    ports:
      - "8280:8080"
    volumes:
      - ./xxl-job/data:/data/applogs
    environment:
      TZ: ${TZ}
      PARAMS: >-
        --spring.datasource.url=jdbc:mysql://mysql:3306/${MYSQL_XXL_JOB_DB}?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
        --spring.datasource.username=root
        --spring.datasource.password=${MYSQL_ROOT_PASSWORD}
    networks:
      - middleware_net

networks:
  middleware_net:
    driver: bridge
```

## 5. 常用配置文件模板

### 5.1 `mysql/conf/my.cnf`

MySQL 8 可用：

```ini
[mysqld]
default-time-zone = '+8:00'
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
default-authentication-plugin = mysql_native_password
lower_case_table_names = 1
max_connections = 1000

[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4
```

MySQL 5.7 不支持 `default-authentication-plugin` 时删除该行。

### 5.2 `mongo/config/mongod.conf`

```yaml
storage:
  dbPath: /data/db
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
net:
  bindIp: 0.0.0.0
  port: 27017
security:
  authorization: enabled
```

### 5.3 `elasticsearch/config/elasticsearch.yml`

```yaml
cluster.name: docker-es
node.name: elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
```

如需开启账号密码，把 compose 中 `xpack.security.enabled` 改为 `"true"`，并增加：

```yaml
ELASTIC_USERNAME: elastic
ELASTIC_PASSWORD: ${ELASTIC_PASSWORD}
```

### 5.4 `rocketmq/broker/conf/broker.conf`

bridge 网络内使用：

```properties
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
brokerIP1=rocketmq-broker
listenPort=10911
autoCreateTopicEnable=true
```

如果宿主机或局域网应用要直接访问 Broker，`brokerIP1` 改为宿主机 IP：

```properties
brokerIP1=192.168.1.100
```

### 5.5 `seata/application.yml`

Seata 强依赖项目侧的注册中心、配置中心和数据库表。以下是 Nacos + MySQL 常用骨架：

```yaml
server:
  port: 7091

spring:
  application:
    name: seata-server

seata:
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: nacos:8848
      group: SEATA_GROUP
      namespace:
      username: nacos
      password: nacos
  config:
    type: nacos
    nacos:
      server-addr: nacos:8848
      group: SEATA_GROUP
      namespace:
      username: nacos
      password: nacos
  store:
    mode: db
    db:
      datasource: druid
      db-type: mysql
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://mysql:3306/seata?rewriteBatchedStatements=true&serverTimezone=Asia/Shanghai
      user: root
      password: root
```

### 5.6 Canal 关键项

`canal/conf/example/instance.properties` 至少关注：

```properties
canal.instance.master.address=mysql:3306
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset=UTF-8
canal.instance.tsdb.enable=true
canal.instance.filter.regex=.*\\..*
```

MySQL 需要开启 binlog，并创建 canal 用户：

```sql
CREATE USER 'canal'@'%' IDENTIFIED BY 'canal';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```

## 6. 服务访问清单

| 服务 | 宿主机访问 | 容器内访问 | 默认账号 |
| --- | --- | --- | --- |
| MySQL | `127.0.0.1:3306` | `mysql:3306` | `root/${MYSQL_ROOT_PASSWORD}` |
| Nacos | `http://127.0.0.1:8848/nacos` | `nacos:8848` | 视镜像版本和初始化配置而定 |
| Redis | `127.0.0.1:6379` | `redis:6379` | 密码 `${REDIS_PASSWORD}` |
| MongoDB | `127.0.0.1:27017` | `mongo:27017` | `${MONGO_ROOT_USER}/${MONGO_ROOT_PASSWORD}` |
| RabbitMQ | `http://127.0.0.1:15672` | `rabbitmq:5672` | `${RABBITMQ_USER}/${RABBITMQ_PASSWORD}` |
| Elasticsearch | `http://127.0.0.1:9200` | `elasticsearch:9200` | 默认未启用安全认证 |
| Kibana | `http://127.0.0.1:5601` | `kibana:5601` | 跟随 Elasticsearch |
| MinIO API | `http://127.0.0.1:9000` | `minio:9000` | `${MINIO_ROOT_USER}/${MINIO_ROOT_PASSWORD}` |
| MinIO Console | `http://127.0.0.1:9001` | `minio:9001` | `${MINIO_ROOT_USER}/${MINIO_ROOT_PASSWORD}` |
| RocketMQ NameServer | `127.0.0.1:9876` | `rocketmq-namesrv:9876` | 无 |
| RocketMQ Dashboard | `http://127.0.0.1:8080` | `rocketmq-dashboard:8080` | 无 |
| Seata | `127.0.0.1:7091/8091` | `seata-server:7091/8091` | 视配置而定 |
| Canal | `127.0.0.1:11111` | `canal:11111` | 视配置而定 |
| InfluxDB | `http://127.0.0.1:8086` | `influxdb:8086` | 视初始化而定 |
| XXL-Job | `http://127.0.0.1:8280/xxl-job-admin` | `xxl-job:8080` | 默认 `admin/123456`，以初始化库为准 |

## 7. 复用注意事项

- `depends_on` 只保证启动顺序，不保证 MySQL/Nacos 已经完全可用；应用启动失败时要看日志并重试。
- `mysql/source` 中的 SQL 只会在 `mysql/data` 为空的首次启动时执行。
- 单文件挂载前必须先创建文件，例如 `mysql/conf/my.cnf`、`mongo/config/mongod.conf`，否则 Docker 可能把宿主机路径创建成目录。
- Nacos 2.x 除 `8848` 外还需要 `9848/9849`，否则部分客户端连接异常。
- RabbitMQ 延迟队列插件要把 `.ez` 文件放进 `rabbitmq/plugins/`；不需要延迟队列时可以删掉 `command` 中的插件启用逻辑。
- Elasticsearch 在 Linux 上可能需要执行 `sysctl -w vm.max_map_count=262144`。
- RocketMQ 如果客户端跑在宿主机或其他机器上，`broker.conf` 的 `brokerIP1` 必须能被客户端访问。
- Seata、Canal、XXL-Job 都依赖数据库初始化脚本，模板只提供容器配置，业务库表仍需从对应项目复制。
- 生产环境不要沿用模板密码；`.env` 不要提交到公共仓库。

## 8. 最小组合建议

按项目类型裁剪：

| 场景 | 建议保留 |
| --- | --- |
| 单体 Java 后端 | MySQL、Redis |
| Spring Cloud 微服务 | MySQL、Redis、Nacos |
| 消息队列 | RabbitMQ 或 RocketMQ 二选一 |
| 搜索 | Elasticsearch、Kibana |
| 文件存储 | MinIO |
| 分布式事务 | Seata、MySQL、Nacos |
| 数据同步/binlog | Canal、MySQL |
| 定时任务 | XXL-Job、MySQL |
| 时序数据 | InfluxDB |

