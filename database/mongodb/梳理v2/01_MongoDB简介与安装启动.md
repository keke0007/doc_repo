# 01 · MongoDB 简介与安装启动

## 一、知识点梳理

### 1. MongoDB 是什么
- 非关系型（NoSQL）数据库，但操作风格与关系型数据库较接近，便于上手。
- 面向文档存储，单条记录以 BSON（二进制 JSON）形式存放，对外表现为 JSON。
- 既能做持久化存储，也能当作缓存使用。
- 原生提供副本集（Replica Set）和分片集群（Sharded Cluster）。

### 2. 实战环境约定
- 操作系统：CentOS 7（64 位）。
- 关闭 iptables 与 selinux，避免端口/权限问题影响实验。

### 3. 二进制免编译安装
```bash
cd /usr/local/src/
wget 'https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.2.tgz'
tar -zxvf mongodb-linux-x86_64-4.0.2.tgz
mv mongodb-linux-x86_64-4.0.2 /usr/local/mongodb
```
- 解压目录的 `bin/` 下含核心二进制：`mongod`（服务端）、`mongo`（客户端）、`mongos`（分片路由）、`mongodump/mongorestore`（备份恢复）、`mongostat`（监控）。
- 验证：
  ```bash
  /usr/local/mongodb/bin/mongod --version
  /usr/local/mongodb/bin/mongod --help
  ```

### 4. 单实例配置文件 `/data/mongodb/27017/mongodb.conf`
```yaml
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/27017/mongodb.log
storage:
  dbPath: /data/mongodb/27017/
  journal:
    enabled: true
processManagement:
  fork: true
net:
  port: 27017
  bindIp: 0.0.0.0
```
要点：
- `systemLog`：日志落盘并追加写入。
- `storage.journal.enabled`：开启预写日志，保障崩溃恢复。
- `processManagement.fork: true`：以守护进程方式后台运行。
- `net.bindIp`：监听地址，公网建议改为 `127.0.0.1` 或内网网段。

### 5. 启动与验证
```bash
/usr/local/mongodb/bin/mongod -f /data/mongodb/27017/mongodb.conf
ll -h /data/mongodb/27017/      # 数据文件
ps -ef | grep mongod            # 进程
ss -tlnp | grep mongo           # 端口（替代过时的 netstat）
```

### 6. 关闭方式
- 不要使用 `kill -9` 或硬断电，会丢失内存中尚未刷盘的数据。
- 推荐方式：进入 `mongo` 客户端后执行
  ```js
  use admin
  db.shutdownServer()
  ```
- 也可向 `mongod` 进程发送 SIGTERM（即普通 `kill <pid>`，不带 -9），它会触发优雅关闭。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.2.tgz`（前文写 http） | 官方仅提供 https，下载链接应统一为 https | 使用 `https://fastdl.mongodb.org/...` |
| 2 | "建议监听在 127.0.0.1:2017" | 端口写错，少一位 | 应为 `127.0.0.1:27017` |
| 3 | "kill 关闭 #不建议" | 表述含糊。普通 `kill` 发送 SIGTERM 实际是安全的优雅关闭，**真正危险的是 `kill -9`** | 不建议的是 `kill -9` 与硬断电；普通 `kill <pid>` 等价于 SIGTERM，可安全关闭 |
| 4 | "monogdb 单例配置文件" | 拼写错误 | mongodb |
| 5 | 下载链接示意为 4.0.2 | 4.0.x 已停止维护，演示可用，但生产应选当前 LTS（4.4 / 5.0 / 6.0 / 7.0 之一） | 学习沿用 4.0.2，生产请改用受支持版本 |

## 三、安装流程图（ASCII）

```
 ┌────────────┐  wget   ┌──────────────────────────┐
 │  官方镜像   │ ──────▶ │ /usr/local/src/*.tgz      │
 └────────────┘         └─────────────┬────────────┘
                                       │ tar -zxvf
                                       ▼
                       ┌──────────────────────────┐
                       │ /usr/local/mongodb/bin    │
                       │   mongod / mongo / mongos │
                       └─────────────┬────────────┘
                                     │ -f conf
                                     ▼
   ┌──────────────────────────┐    fork    ┌──────────────────────────┐
   │ /data/mongodb/27017/     │◀──────────│  mongod 守护进程          │
   │   mongodb.conf           │            │  监听 0.0.0.0:27017      │
   │   mongodb.log            │   写入     │                          │
   │   journal/、collection-* │◀──────────│  WiredTiger 引擎         │
   └──────────────────────────┘            └─────────────┬────────────┘
                                                          │ TCP 27017
                                                          ▼
                                              ┌──────────────────────┐
                                              │ mongo 客户端 / 应用   │
                                              └──────────────────────┘
```
