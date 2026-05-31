# 02 · MongoDB 服务器启动优化

## 一、知识点梳理

### 1. 启动告警来源
首次以 `mongod` 启动后，`mongo` 客户端连接时会打印若干 WARNING，需逐项消除：
- 文件系统建议 XFS（WiredTiger 引擎对 XFS 兼容性最好）。
- 未启用访问控制（auth）。
- 以 root 身份运行。
- 透明大页（THP）开启。
- soft rlimit 过低（进程数、文件数）。

### 2. 关闭透明大页（THP）
```bash
echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled
echo 'never' > /sys/kernel/mm/transparent_hugepage/defrag
```
为保证重启后生效，写入 `/etc/rc.d/rc.local` 并赋予执行权限：
```bash
chmod +x /etc/rc.d/rc.local
```

### 3. 放开 ulimit
- `/etc/security/limits.conf`：
  ```
  *  -  nofile  65535
  *  -  nproc   65536
  ```
- CentOS 7 还有 `/etc/security/limits.d/20-nproc.conf` 会覆盖默认进程数限制，需同步调整。
- 修改后**重新登录会话**才生效。
- MongoDB 官方建议：`nproc ≥ 32768`、`nofile ≥ 64000`。

### 4. 使用普通用户运行 mongod
```bash
useradd mongodb -s /sbin/nologin
chown -R mongodb:mongodb /data/mongodb/ /usr/local/mongodb/
su - mongodb -s /bin/bash
/usr/local/mongodb/bin/mongod -f /data/mongodb/27017/mongodb.conf
```

### 5. 安全建议
- 启用访问控制（`security.authorization: enabled`）。
- 使用 `bindIp` + 防火墙做 IP 白名单。
- 公网部署务必关闭 0.0.0.0 监听。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "soft rlimits too low. rlimits set to 3895 processes, 65535 files. Number of processes should be at least 32767.5 : 0.5 times number of files." 解读 | 原文未解释含义 | MongoDB 要求 `nproc ≥ 0.5 × nofile`，即至少 32767 个进程 |
| 2 | `* - nproc 65536` | 仅写了 nproc，未提 nofile | 同时配置 `* - nofile 65535` 才能消除文件数告警 |
| 3 | "centos7 默认还有进程数限制 /etc/security/limits.d/20-nproc.conf" | 文档未给出修改示例 | 该文件中 `* soft nproc 4096` 行需调高，否则普通用户 nproc 仍受限 |
| 4 | 关闭 THP 写到 `/etc/rc.local` | CentOS 7 真正生效路径是 `/etc/rc.d/rc.local`（`/etc/rc.local` 是它的软链） | 推荐直接编辑 `/etc/rc.d/rc.local` 并 `chmod +x` |
| 5 | 未提及 access control 落地 | 仅"建议设置 ip 白名单" | 真正的 access control 需要 `security.authorization: enabled` 并创建管理用户 |

## 三、内核与启动关系图（ASCII）

```
       ┌──────────────────────────────────────────────────────────┐
       │ /etc/security/limits.conf       /etc/security/limits.d/  │
       │ /etc/rc.d/rc.local              /sys/kernel/mm/THP       │
       └─────────────────────────┬────────────────────────────────┘
                                 │ 登录会话/开机自启加载
                                 ▼
       ┌──────────────────────────────────────────────────────────┐
       │              shell 会话（mongodb 用户）                  │
       │   ulimit -n 65535   ulimit -u 65536   THP=never          │
       └─────────────────────────┬────────────────────────────────┘
                                 │ exec mongod -f conf
                                 ▼
       ┌──────────────────────────────────────────────────────────┐
       │   mongod 进程（继承 ulimit / 内核参数）                  │
       │   ▶ 启动检查 → 无 WARNING                                │
       │   ▶ WiredTiger 申请大块内存（不再被 THP 拖慢）           │
       └──────────────────────────────────────────────────────────┘
```
