# LVS + Keepalived 高可用部署梳理

目录：[1. 高可用基础概念](#1-高可用基础概念) | [2. LVS 负载均衡](#2-lvs-负载均衡) | [3. Keepalived 故障转移](#3-keepalived-故障转移) | [4. 速查表](#速查表)

---

## 1. 高可用基础概念

### 1.1 高可用定义与衡量

**高可用集群（HA Cluster）**：通过保护业务程序对外部不间断提供服务，把故障对业务的影响降到最低。目标：7×24 小时不宕机。

**可用性衡量公式**：
$$HA = \frac{MTTF}{MTTF + MTTR} \times 100\%$$

其中：
- MTTF（Mean Time To Failure）：平均无故障时间
- MTTR（Mean Time To Repair）：平均故障维修时间

| 可用性等级 | Nines | 百分比 | 年度宕机时间 |
|-----------|-------|--------|-----------|
| 基本可用 | 2个9 | 99% | 87.6小时 |
| 较高可用 | 3个9 | 99.9% | 8.8小时 |
| 故障自动恢复 | 4个9 | 99.99% | 53分钟 |
| 极高可用 | 5个9 | 99.999% | 5分钟 |

### 1.2 高可用三大支柱

1. **负载均衡**：分散请求到多个节点
2. **健康检测**：监控服务器状态
3. **故障转移**：自动切换到正常服务器

### 1.3 负载均衡技术对比

| 技术 | 协议层 | 优点 | 缺点 |
|-----|--------|------|------|
| **Nginx** | 应用层(7) | 简单易配置，性能好，支持HTTP/HTTPS/Email | 仅支持HTTP/Email，健康检测单一 |
| **HAProxy** | 传输层(4) | 高效率，支持TCP，支持URL检测 | 不支持POP/SMTP，无HTTP缓存 |
| **LVS** | 网络层(4) | **抗负载能力强，性能最好，应用范围广** | 不支持正则，不支持动静分离 |

**推荐方案**：LVS（性能） + Keepalived（高可用）

### 1.4 Keepalived vs Heartbeat

| 维度 | Keepalived | Heartbeat |
|-----|-----------|-----------|
| 易用性 | 简单 | 复杂 |
| 功能 | 基础（倒换） | 强大（管理工具多） |
| 适用场景 | 中小型集群切换 | 大型集群管理 |

---

## 2. LVS 负载均衡

### 2.1 LVS 基础概念

**LVS（Linux Virtual Server）**：
- 1998年由章文嵩博士创建
- 工作在 OSI 模型第4层（传输层）
- 基于IP进行负载均衡
- Linux 2.2 以补丁形式出现，2.4+ 成为内核标准

**术语定义**：

| 术语 | 说明 |
|-----|------|
| **VIP** | Virtual IP - 虚拟IP，对外提供服务的IP |
| **RIP** | Real IP - 真实IP，后端真实服务器IP |
| **DIP** | Director IP - 调度器IP，负载均衡器的物理网卡IP |
| **CIP** | Client IP - 客户端IP |
| **DS** | Director Server - 负载均衡器（LVS服务器） |
| **RS** | Real Server - 真实服务器 |

### 2.2 LVS 三种工作模式

#### 2.2.1 NAT 模式（Network Address Translation）

**特点**：
- 请求和响应都经过LVS处理
- LVS 需要修改请求的目标IP和响应的源IP
- 性能瓶颈明显（需处理双向流量）

**工作流程**：

```
┌──────────────────────────────────────────┐
│          Client (CIP)                    │
│       192.168.3.31:random                │
└────────────┬─────────────────────────────┘
             │ 1. 请求 → VIP:80
             ↓
        ┌────────────────────┐
        │  LVS (NAT mode)    │ ens33: 192.168.3.30 (VIP)
        │  (修改目标IP)      │ ens37: 192.168.10.30 (DIP)
        └────────┬───────────┘
                 │ 2. 转发 → RIP:80
        ┌────────┴──────┬──────────────┐
        │               │              │
        ↓               ↓              ↓
    RS1(31)         RS2(32)       RS3(33)
  192.168.10.31   192.168.10.32  192.168.10.33
    :80 处理          :80 处理
        │               │              │
        │ 3. 响应回应   │ 4. LVS修改   │
        └──────────┬─────────────────┘
                   │ 源IP: VIP:80
                   ↓
            返回给 Client

问题：双向流量都经过LVS，易成为性能瓶颈
```

**配置示例**：
```bash
# 添加VIP和调度算法
ipvsadm -A -t 192.168.3.30:80 -s rr

# 添加RS (NAT模式用 -m)
ipvsadm -a -t 192.168.3.30:80 -r 192.168.10.31 -m
ipvsadm -a -t 192.168.3.30:80 -r 192.168.10.32 -m

# 查看配置
ipvsadm -Ln
```

**适用场景**：小规模集群（RS数量少，并发量小）

---

#### 2.2.2 DR 模式（Direct Routing）- 推荐

**特点**：
- LVS 只处理入站请求，RS 直接返回响应
- 响应流量不经过LVS（性能最优）
- RS 和 LVS 必须在同一网段
- RS 和 LVS 都需要绑定VIP
- 需要ARP抑制配置

**工作流程**：

```
Client Request Flow:
┌──────────────────────────────────────┐
│        Client (CIP)                  │
│    192.168.10.33:random              │
└────────────┬──────────────────────────┘
             │
             │ 1. 请求：SRC=CIP, DST=VIP:80
             ↓
        ┌─────────────────┐
        │  LVS (DR mode)  │ 192.168.10.30 + VIP
        │ 修改目标MAC地址 │ (同网段)
        └────────┬────────┘
                 │ 2. 转发：SRC=CIP, DST=RIP
        ┌────────┴──────┬──────────────┐
        │               │              │
        ↓               ↓              ↓
    RS1(31)         RS2(32)
  绑定VIP在lo       绑定VIP在lo
  192.168.10.31    192.168.10.32
        │               │
        │ 3. 本地响应    │ (直接返回给Client)
        │ SRC=VIP, DST=CIP
        │               │
        └──────────┬────┘
                   │ 响应直接返回Client（不经过LVS）
                   ↓
            Client 收到响应

关键：响应流量不经过LVS，大幅提升性能
```

**配置要点**：

```bash
# RS1 和 RS2 配置（两台都做）
# 1. ARP抑制（防止ARP冲突）
echo 1 > /proc/sys/net/ipv4/conf/ens33/arp_ignore
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
echo 2 > /proc/sys/net/ipv4/conf/ens33/arp_announce

# 2. 绑定VIP到lo网卡（子网掩码必须是255.255.255.255）
ifconfig lo:9 192.168.10.34 netmask 255.255.255.255

# 3. 删除网关（重要！）
# 编辑 /etc/sysconfig/network-scripts/ifcfg-ens33，删除GATEWAY
systemctl restart network

# LVS 配置（使用 -g 参数）
ipvsadm -A -t 192.168.10.34:80 -s rr
ipvsadm -a -t 192.168.10.34:80 -r 192.168.10.31 -g
ipvsadm -a -t 192.168.10.34:80 -r 192.168.10.32 -g
```

**性能优势**：
- RS可直接返回，响应流量不经过LVS
- 可支持更高的并发量
- 单台LVS可支持更多RS

---

#### 2.2.3 TUN 模式（Tunneling）

**特点**：
- RS 通过IP隧道与LVS通信
- 需要操作系统和网络设备支持IP隧道
- 配置复杂，限制多

**工作流程**：

```
Client Request:
         │ 请求 → VIP:80
         ↓
    ┌─────────────┐
    │ LVS TUN模式 │
    │ 封装请求    │
    └──────┬──────┘
           │ 以隧道方式传输
           │ (IP in IP)
           ↓
    ┌──────────┐
    │ RS 解包  │
    │ 处理请求 │
    └──────────┘
    │ 直接返回
    ↓ 给Client
```

**状态**：一般不使用（配置复杂，场景有限）

---

### 2.3 LVS 调度算法

#### 2.3.1 静态调度算法（不考虑服务器状态）

| 算法 | 名称 | 说明 |
|-----|------|------|
| **RR** | Round Robin | 轮询，依次轮转分配请求 |
| **WRR** | Weighted RR | 加权轮询，按权重比例轮询 |
| **SH** | Source Hashing | 源地址哈希，同一源IP总是转发到同一RS |
| **DH** | Destination Hashing | 目标地址哈希，同一目标IP总转发到同一RS |

**轮询示例**：
```
RR: RS1 → RS2 → RS1 → RS2 → ...

WRR: RS1(w=2) + RS2(w=1)
     RS1 → RS1 → RS2 → RS1 → RS1 → RS2 → ...
```

#### 2.3.2 动态调度算法（考虑服务器连接数/性能）

| 算法 | 全称 | 说明 |
|-----|------|------|
| **LC** | Least Connections | 最小连接数，选择连接数最少的RS |
| **WLC** | Weighted LC | 加权最小连接数（**默认**） |
| **SED** | Shortest Expection Delay | 初始连接高权重优先 |
| **NQ** | Never Queue | 第一轮平均分配，后续用SED |
| **LBLC** | Locality-Based LC | 基于局部性的最小连接，同一源IP倾向同一RS |
| **LBLCR** | LBLC with Replication | 带复制功能的LBLC |
| **FO** | Weighted Fail Over | 加权故障转移（kernel 4.15+） |
| **OVF** | Overflow-connection | 溢出连接（kernel 4.15+） |

**选择建议**：
- 长连接应用 → SED、LC
- 短连接应用 → RR、WRR
- 需要会话保持 → SH、LBLC

---

### 2.4 LVS 命令速查

**安装**：
```bash
yum install -y ipvsadm
```

**集群服务管理**：

| 命令 | 说明 |
|-----|------|
| `ipvsadm -A -t VIP:PORT -s 算法` | 添加虚拟服务 |
| `ipvsadm -C` | 清空所有规则 |
| `ipvsadm -L` | 查看规则（域名） |
| `ipvsadm -Ln` | 查看规则（IP） |
| `ipvsadm -Lnc` | 查看连接表 |
| `ipvsadm -S` | 保存规则 |
| `ipvsadm -R` | 恢复规则 |

**RS 管理**：

| 命令 | 说明 |
|-----|------|
| `ipvsadm -a -t VIP:PORT -r RIP:PORT -m` | 添加RS（NAT模式） |
| `ipvsadm -a -t VIP:PORT -r RIP:PORT -g` | 添加RS（DR模式） |
| `ipvsadm -a -t VIP:PORT -r RIP:PORT -i` | 添加RS（TUN模式） |
| `ipvsadm -d -t VIP:PORT -r RIP:PORT` | 删除RS |

---

## 3. Keepalived 故障转移

### 3.1 Keepalived 简介

**作用**：
- 检测服务器（RS 和 LVS）状态
- 故障时自动剔除故障节点
- 支持故障转移，自动切换

**核心功能**：
1. 健康检查（Health Check）
2. 故障转移（Failover）基于VRRP协议

### 3.2 健康检查机制

#### 3.2.1 检查方式

| 检查类型 | 工作层 | 说明 |
|---------|--------|------|
| **TCP_CHECK** | 4层 | 检测TCP连接，超时则判定故障 |
| **HTTP_GET** | 7层 | 请求指定URL，验证返回码或MD5 |
| **HTTPS_GET/SSL_GET** | 7层 | HTTPS版本的HTTP_GET |
| **SMTP_CHECK** | 7层 | SMTP协议检测（邮件系统） |
| **MISC_CHECK** | 脚本 | 执行自定义脚本检测 |

#### 3.2.2 HTTP_GET 检查配置

```tcl
HTTP_GET {
    url {
        path /              # 检测URL路径
        status 200          # 期望HTTP状态码
    }
    connect_timeout 3       # 连接超时时间
    nb_get_retry 3          # 重试次数
    delay_before_retry 3    # 重试间隔
}
```

### 3.3 VRRP 协议与故障转移

#### 3.3.1 VRRP 协议概念

**Virtual Router Redundancy Protocol（虚拟路由冗余协议）**：
- 一种容错的**主备模式**协议
- 解决路由器单点故障问题
- 主节点故障时，备节点透明接管

#### 3.3.2 故障转移原理

```
正常状态：
Master ──心跳(多播)──→ Backup
   │                    │
   │                    │
   提供VIP服务          等待中
   优先级: 100          优先级: 80

故障转移：
Master 宕机 ──无心跳── Backup 检测到故障
                        │
                        ├─ 等待一定时间
                        │
                        └─→ 接管VIP
                            成为新Master
                            优先级: 100

恢复过程：
Original Master 恢复
   │
   └─→ 发送高优先级心跳
       │
       └─→ Backup 识别并释放VIP
           │
           └─→ 恢复到Backup角色
```

#### 3.3.3 故障转移心跳流程

```
VRRP Heartbeat Exchange:

MASTER (优先级100)              BACKUP (优先级80)
    │                              │
    ├──────多播心跳───────────────→ │
    │  (1秒一次，默认)              │
    │                              │
    │  [确认MASTER健康]            │
    │                              │
    ┴                              ┴
    
当MASTER故障：
[故障1]                            │
    ×                              │
    ×xxxxxxxx 无心跳               │
    ×        (检测超时)            │
    │                        │─ 检测故障
    │                        │
    │                        └─ 启动故障转移脚本
    │                        └─ 绑定VIP到本地
    │                        └─ 成为新MASTER
    │                              │
    │                        [VRRP BACKUP→MASTER]
    │                              │
    │         【此时Backup成为Master】
    │
[故障恢复]
    │───────→ 新心跳信息
    │         (告知自己恢复)
    │
    BACKUP  ←─── 发送新的心跳
     │
     └─ 收到原MASTER的高优先级心跳
        │
        └─ 释放VIP给原MASTER
           恢复到BACKUP角色
```

### 3.4 分布式选主策略

#### 3.4.1 基于 Priority

**原理**：
- Priority 最高的成为 MASTER
- 其他都是 BACKUP
- MASTER故障后，BACKUP间进行民主选举

**示例**：
```
LVS_MASTER: priority = 100
LVS_BACKUP: priority = 80

结果：LVS_MASTER 成为主
```

#### 3.4.2 基于 Priority + Weight

**权值计算**：

**Weight为正数时**：
```
检测成功时：权值 = priority + weight
检测失败时：权值 = priority

例：
MASTER: priority=100, weight=10, 检测成功
  权值 = 100 + 10 = 110
  
BACKUP: priority=80, weight=20, 检测失败
  权值 = 80（不加weight）

110 > 80，MASTER保持主角色
```

**切换条件**（Weight为正）：
```
MASTER检测失败且：
  priority < BACKUP的(priority + weight)
  则发生切换
```

**Weight为负数时**：
```
检测成功时：权值 = priority
检测失败时：权值 = priority + weight（负数）

例：
MASTER: priority=100, weight=-20, 检测失败
  权值 = 100 + (-20) = 80
  
BACKUP: priority=80
  权值 = 80

80 = 80，相等触发切换（BACKUP接管）
```

**Weight设置标准**：
```
|weight| > |priority_MASTER - priority_BACKUP|

例：MASTER priority=100, BACKUP priority=80
    差值 = 20
    weight的绝对值应该 > 20
    建议设置 weight = ±30 或 ±50
```

#### 3.4.3 选主示例

```
场景1：仅设置Priority
├─ MASTER: state=MASTER, priority=100
├─ BACKUP: state=BACKUP, priority=80
└─ 结果：MASTER当选，优先级更高

场景2：Priority + Weight（LVS故障检测）
├─ MASTER: priority=100, weight=10
│         脚本检测服务状态
├─ BACKUP: priority=80, weight=0
│
├─ MASTER的LVS进程故障
│  └─ 权值 = 100 + (-10) = 90
│
└─ 90 > 80 → MASTER保持（仍是主）
   （因为LVS故障但主节点优先级仍高）

场景3：MASTER故障时
├─ MASTER无响应（宕机）
│  └─ BACKUP停止收到心跳
│
├─ BACKUP启动故障转移
│  ├─ 绑定VIP到本地
│  ├─ 启动LVS服务
│  └─ 成为新MASTER
│
└─ MASTER恢复后
   ├─ 发送心跳，说明自己恢复
   ├─ BACKUP识别高优先级
   └─ 释放VIP给原MASTER
```

---

### 3.5 Keepalived 工作层次

| 工作层 | 说明 |
|--------|------|
| **网络层(3)** | ICMP Ping 检测节点存活，无响应则标记为故障 |
| **传输层(4)** | TCP 端口连接检测（如HTTP 80端口，SSH 22端口），检测应用可用性 |
| **应用层(7)** | 自定义脚本检测，可检测应用逻辑正确性，如数据库连接、服务状态等 |

---

## 4. LVS + Keepalived 实战配置

### 4.1 系统拓扑

```
┌────────────────────────────────────────────────┐
│                   Internet                     │
│                  192.168.3.x                   │
└────────────────────────────────────────────────┘
                        │
                   VIP: 192.168.3.30
                        │
        ┌───────────────┴────────────────┐
        │                                │
    ┌───────────┐                  ┌──────────┐
    │  LVS_MAIN │ ens33            │LVS_BACKUP│ ens33
    │(MASTER)   │192.168.10.30     │(BACKUP)  │192.168.10.35
    │priority100│                  │priority80│
    └──────┬────┘                  └────┬─────┘
           │ eth1: 192.168.10.30        │ eth1: 192.168.10.35
           │ (DIP)                      │ (DIP)
           └──────────────┬─────────────┘
                          │
              ┌───────────┴────────────┐
              │                        │
         ┌────────┐               ┌────────┐
         │  RS1   │               │  RS2   │
         │192.168 │               │192.168 │
         │.10.31  │               │.10.32  │
         │:80     │               │:80     │
         └────────┘               └────────┘

特点：
1. LVS_MAIN 和 LVS_BACKUP 都绑定VIP
2. 同网段，使用DR模式
3. RS也在同网段，需要ARP抑制
```

### 4.2 Keepalived 配置详解

#### 4.2.1 主LVS配置（MASTER）

文件：`/etc/keepalived/keepalived.conf`

```tcl
! Configuration File for keepalived

# 全局配置
global_defs {
    notification_email {                    # 故障通知邮箱
        admin@example.com
    }
    notification_email_from keepalived@example.com
    smtp_server 192.168.1.1                # SMTP服务器
    smtp_connect_timeout 30
    router_id LVS_MAIN                     # 本机标识
}

# VRRP 实例：定义主备切换逻辑
vrrp_instance VI_1 {
    state MASTER                            # 本机角色：MASTER or BACKUP
    interface ens33                         # 心跳网卡（必须与VIP同网卡）
    virtual_router_id 51                    # 虚拟路由ID（主备相同）
    priority 100                            # 优先级：100(主) > 80(备)
    advert_int 1                            # 心跳间隔（秒）
    
    authentication {                        # 主备认证（必须相同）
        auth_type PASS                      # 认证方式
        auth_pass keepalived123             # 认证密码（≤8字符）
    }
    
    # VIP配置（主备都需配置，会自动在主上激活）
    virtual_ipaddress {
        192.168.10.34/24 dev ens33 label ens33:9
    }
    
    # 追踪脚本（可选）
    track_script {
        check_lvs_process 10                # 追踪脚本权重
    }
}

# 虚拟服务定义：对应 ipvsadm -A 命令
virtual_server 192.168.10.34 80 {          # VIP:PORT
    delay_loop 6                            # 健康检查间隔（秒）
    lb_algo rr                              # 调度算法：rr, wrr, lc, wlc等
    lb_kind DR                              # 工作模式：NAT, DR, TUN
    persistence_timeout 0                   # 会话保持时间(0=无持久化)
    protocol TCP                            # 协议类型
    
    # 真实服务器1
    real_server 192.168.10.31 80 {
        weight 1                            # 权重
        HTTP_GET {
            url {
                path /                      # 检测路径
                status 200                  # 期望HTTP状态码
            }
            connect_timeout 3               # 连接超时
            nb_get_retry 3                  # 重试次数
            delay_before_retry 3            # 重试间隔
        }
    }
    
    # 真实服务器2
    real_server 192.168.10.32 80 {
        weight 1
        HTTP_GET {
            url {
                path /
                status 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

# 可选：自定义检测脚本
vrrp_script check_lvs_process {
    script "/usr/local/bin/check_lvs.sh"   # 检测脚本路径
    interval 5                              # 检测间隔
    weight -10                              # 失败时权值调整
    fall 3                                  # 3次失败判定为故障
    rise 2                                  # 2次成功判定为恢复
}
```

#### 4.2.2 备LVS配置（BACKUP）

```tcl
! Configuration File for keepalived

global_defs {
    notification_email {
        admin@example.com
    }
    notification_email_from keepalived@example.com
    smtp_server 192.168.1.1
    smtp_connect_timeout 30
    router_id LVS_BACKUP                    # 不同标识
}

vrrp_instance VI_1 {
    state BACKUP                            # 改为 BACKUP
    interface ens33
    virtual_router_id 51                    # 与MASTER相同（重要！）
    priority 80                             # 比MASTER低
    advert_int 1
    
    authentication {
        auth_type PASS
        auth_pass keepalived123             # 与MASTER相同（重要！）
    }
    
    virtual_ipaddress {
        192.168.10.34/24 dev ens33 label ens33:9
    }
}

# 虚拟服务定义完全相同
virtual_server 192.168.10.34 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    persistence_timeout 0
    protocol TCP
    
    real_server 192.168.10.31 80 {
        weight 1
        HTTP_GET {
            url {
                path /
                status 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    
    real_server 192.168.10.32 80 {
        weight 1
        HTTP_GET {
            url {
                path /
                status 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

### 4.3 完整部署步骤

#### 步骤1：准备4台虚拟机

```bash
# 4台机器的规划
LVS_MAIN:    192.168.10.30 (主节点)
LVS_BACKUP:  192.168.10.35 (备节点)
RS1:         192.168.10.31 (真实服务器)
RS2:         192.168.10.32 (真实服务器)
VIP:         192.168.10.34 (浮动VIP)
```

#### 步骤2：配置RS节点

```bash
# 两台RS都做如下配置

# 安装httpd
yum install -y httpd

# 设置首页
echo "this is RS1" > /var/www/html/index.html  # RS1
echo "this is RS2" > /var/www/html/index.html  # RS2

# 启动服务
systemctl start httpd
systemctl enable httpd

# ARP抑制（重要！）
echo 1 > /proc/sys/net/ipv4/conf/ens33/arp_ignore
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
echo 2 > /proc/sys/net/ipv4/conf/ens33/arp_announce

# 永久生效（写入系统配置）
cat >> /etc/sysctl.conf << EOF
net.ipv4.conf.ens33.arp_ignore = 1
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.ens33.arp_announce = 2
EOF
sysctl -p

# 绑定VIP到lo网卡（必须子网掩码255.255.255.255）
ifconfig lo:9 192.168.10.34 netmask 255.255.255.255

# 持久化（写入网络配置）
cat > /etc/sysconfig/network-scripts/ifcfg-lo:9 << EOF
DEVICE=lo:9
IPADDR=192.168.10.34
NETMASK=255.255.255.255
ONBOOT=yes
EOF

# 删除网关（重要！RS不能有网关）
# 编辑 /etc/sysconfig/network-scripts/ifcfg-ens33
# 删除或注释 GATEWAY 行
systemctl restart network

# 验证
ifconfig | grep -A 2 "lo:9"
route -n  # 确保无GATEWAY
curl localhost  # 验证httpd正常
```

#### 步骤3：配置LVS节点

```bash
# 两台LVS都需安装

# 安装ipvsadm和keepalived
yum install -y ipvsadm keepalived

# 配置网络转发（如果使用NAT模式）
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# 绑定VIP（LVS主要网卡）
ifconfig ens33:9 192.168.10.34 netmask 255.255.255.255

# 编辑keepalived.conf（见上面配置）
vi /etc/keepalived/keepalived.conf

# 验证配置文件语法
keepalived -t

# 启动服务
systemctl start keepalived
systemctl enable keepalived

# 查看VIP是否绑定（主上能看到）
ifconfig | grep -A 2 "ens33:9"

# 查看keepalived日志
journalctl -u keepalived -f
```

#### 步骤4：验证LVS规则

```bash
# 在LVS主节点上查看
ipvsadm -Ln

# 输出示例：
# IP Virtual Server version 1.2.1 (size=4096)
# Prot LocalAddress:Port Scheduler Flags
# -> RemoteAddress:Port Forward Weight ActiveConn InActConn
# TCP 192.168.10.34:80 rr
# -> 192.168.10.31:80 Route 1 0 0
# -> 192.168.10.32:80 Route 1 0 0
```

#### 步骤5：客户端测试

```bash
# 客户端同网段测试（192.168.10.33）
client$ for i in {1..10}; do curl 192.168.10.34; done

# 预期输出：轮流出现 RS1 和 RS2
# this is RS01
# this is RS02
# this is RS01
# this is RS02
# ...
```

### 4.4 故障转移测试

#### 测试A：RS故障自动转移

```bash
# 关闭RS1
[RS1]$ ifconfig ens33 down

# 客户端继续请求（Keepalived自动检测到RS1故障）
[CLIENT]$ for i in {1..5}; do curl 192.168.10.34; done

# 输出：全是RS02，说明RS1已被剔除
# this is RS02
# this is RS02
# this is RS02

# 查看ipvs连接表
[LVS]$ ipvsadm -Lnc  # 只能看到到RS2的连接

# 恢复RS1
[RS1]$ ifconfig ens33 up

# 再请求：恢复轮转
[CLIENT]$ curl 192.168.10.34
# this is RS02
# this is RS01   ← RS1恢复了
```

#### 测试B：LVS主节点故障自动转移

```bash
# 关闭LVS主节点网卡
[LVS_MAIN]$ ifconfig ens33 down

# LVS_BACKUP 检测不到心跳（等待超时）
[LVS_BACKUP]$ # 日志显示：BACKUP → MASTER 转换
               # VIP自动绑定到 ens33:9

# 验证VIP转移
[LVS_BACKUP]$ ifconfig | grep -A 2 "ens33:9"
# ens33:9: inet 192.168.10.34

# 客户端继续正常访问（无感知）
[CLIENT]$ curl 192.168.10.34
# this is RS02  ← 服务不中断
# this is RS01

# 恢复LVS主节点
[LVS_MAIN]$ ifconfig ens33 up

# LVS_MAIN 发送心跳（优先级100）
[LVS_BACKUP]$ # 识别到原MASTER恢复，释放VIP

# VIP 回归主节点
[LVS_MAIN]$ ifconfig | grep -A 2 "ens33:9"
# ens33:9: inet 192.168.10.34
```

---

## 5. 常见问题与故障排查

### 5.1 问题1：RS故障时的处理

**症状**：后端RS宕机，请求仍转发给故障RS

**原因**：
- Keepalived 健康检查未配置
- 检查间隔太长
- 检查脚本返回异常

**解决**：
```bash
# 检查Keepalived日志
journalctl -u keepalived | grep -i "removed\|deleted"

# 手动删除故障RS
ipvsadm -d -t 192.168.10.34:80 -r 192.168.10.31:80

# 重新启动Keepalived
systemctl restart keepalived
```

### 5.2 问题2：LVS故障时的处理

**症状**：主LVS宕机，VIP无法转移

**原因**：
- Keepalived 未启动
- 心跳网络断连
- 密码认证不符

**解决**：
```bash
# 检查Keepalived状态
systemctl status keepalived

# 检查配置文件语法
keepalived -t

# 查看系统日志
journalctl -u keepalived -n 50

# 检查防火墙（VRRP使用IP协议112）
# 需要允许VRRP多播 224.0.0.18:112
```

### 5.3 问题3：RS检测失败

**症状**：HTTP_GET 检测不通过，RS被频繁下线

**原因**：
- 检测URL返回状态码不是200
- RS服务未启动
- 网络连接超时

**解决**：
```bash
# 手动验证健康检查
curl -i http://192.168.10.31/
curl -i http://192.168.10.32/

# 如果返回不是200，修复RS或调整检测配置
# 如果连接超时，增加 connect_timeout 值

# 临时禁用持久化（便于测试轮询）
vi /etc/keepalived/keepalived.conf
# persistence_timeout 0

systemctl restart keepalived
```

### 5.4 问题4：ARP冲突

**症状**：
- 客户端无法访问VIP
- 网络中出现ARP冲突警告
- `arp -a` 看不到VIP

**原因**：
- RS未进行ARP抑制
- RS有网关配置
- LVS和RS的VIP配置不一致

**解决**：
```bash
# RS检查ARP抑制配置
cat /proc/sys/net/ipv4/conf/all/arp_ignore       # 应该是1
cat /proc/sys/net/ipv4/conf/all/arp_announce     # 应该是2

# 重新配置
echo 1 > /proc/sys/net/ipv4/conf/ens33/arp_ignore
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
echo 2 > /proc/sys/net/ipv4/conf/ens33/arp_announce

# 重启网络
systemctl restart network

# 验证VIP
ip addr show | grep 192.168.10.34
```

---

## 6. ASCII 流程图汇总

### 6.1 DR模式包转路流程（详细版）

```
┌─────────────────────────────────────────────────────────────┐
│                   Client (CIP)                              │
│               192.168.10.33:random                          │
│                                                             │
│   step 1: 构造请求包                                        │
│   ┌─────────────────┐                                      │
│   │ SRC: 192.168.10.33                                    │
│   │ DST: 192.168.10.34 (VIP)                              │
│   │ Port: random -> 80                                    │
│   │ Payload: HTTP GET /                                   │
│   └─────────────────┘                                      │
└────────────┬─────────────────────────────────────────────────┘
             │ 请求发送
             ↓
        ┌─────────────────────────────────────┐
        │    LVS Scheduler (DR Mode)          │
        │  192.168.10.30 (ens33)              │
        │  VIP: 192.168.10.34 (ens33:9)       │
        │                                     │
        │ step 2: LVS 处理                     │
        │ ┌──────────────────────────────┐    │
        │ │ 1. 匹配规则                   │    │
        │ │    VIP:80 -> RR 调度          │    │
        │ │                              │    │
        │ │ 2. 选中RS                     │    │
        │ │    当前转: RS1 (192.168.10.31)│   │
        │ │                              │    │
        │ │ 3. 修改目标MAC地址            │    │
        │ │    原MAC: LVS的MAC             │    │
        │ │    新MAC: RS1的MAC             │    │
        │ │                              │    │
        │ │ 4. 转发请求                   │    │
        │ │    SRC: 192.168.10.33 (不变)  │    │
        │ │    DST: 192.168.10.31 (RS)    │    │
        │ │    MAC: RS1的MAC (改变)        │    │
        │ └──────────────────────────────┘    │
        └────────────┬────────────────────────┘
                     │ 转发（二层转发，IP不变）
                     ↓
        ┌─────────────────────────────────────┐
        │  Real Server RS1                    │
        │  IP: 192.168.10.31                  │
        │  VIP: 192.168.10.34 (lo:9)          │
        │                                     │
        │ step 3: RS1 处理                     │
        │ ┌──────────────────────────────┐    │
        │ │ 1. 接收数据包                 │    │
        │ │ 2. 本地地址检查               │    │
        │ │    DST: 192.168.10.31         │    │
        │ │    VIP: 192.168.10.34(lo)     │    │
        │ │    都匹配，接收                │    │
        │ │                              │    │
        │ │ 3. 应用处理                   │    │
        │ │    Apache 处理HTTP请求        │    │
        │ │    返回: "this is RS01"       │    │
        │ │                              │    │
        │ │ 4. 构造响应包                 │    │
        │ │    SRC: 192.168.10.31         │    │
        │ │    DST: 192.168.10.33 (Client)│   │
        │ │    或                         │    │
        │ │    SRC: 192.168.10.34 (VIP)   │    │
        │ │    DST: 192.168.10.33 (Client)│   │
        │ │                              │    │
        │ │ 5. 发送响应                   │    │
        │ │    响应直接返回，不经LVS      │    │
        │ └──────────────────────────────┘    │
        └────────────┬────────────────────────┘
                     │ 响应发送（直连Client）
                     ↓
┌─────────────────────────────────────────────────────────────┐
│                   Client (CIP)                              │
│               192.168.10.33:random                          │
│                                                             │
│   step 4: 接收响应包                                        │
│   ┌─────────────────┐                                      │
│   │ SRC: 192.168.10.34 (VIP)                              │
│   │     或 192.168.10.31 (RS)                             │
│   │ DST: 192.168.10.33 (Client)                           │
│   │ Payload: HTTP/1.1 200 OK                              │
│   │          this is RS01                                 │
│   └─────────────────┘                                      │
│                                                             │
│   ✓ 响应接收完成                                            │
│   ✓ LVS不处理响应流量，性能最优                            │
└─────────────────────────────────────────────────────────────┘

关键特点：
1. LVS只修改MAC地址，IP不变（二层操作）
2. RS直接回应Client，不经过LVS（单向流量）
3. 响应流量完全绕过LVS，避免性能瓶颈
4. 需要RS和LVS同网段，以及ARP抑制
```

### 6.2 VRRP 主备切换流程

```
时间轴：VRRP状态转换

T0: 初始化
┌────────────────────────┐      ┌────────────────────────┐
│   LVS_MAIN (MASTER)    │      │  LVS_BACKUP (BACKUP)   │
│  Priority: 100         │      │  Priority: 80          │
│  State: MASTER         │      │  State: BACKUP         │
│                        │      │                        │
│  ✓ VIP: 绑定在ens33:9 │      │  ✗ VIP: 未绑定         │
│  ✓ LVS: 正常           │      │  ✓ LVS: 等待           │
└───────────┬────────────┘      └────────────┬───────────┘
            │ 心跳: 多播到224.0.0.18 │
            ├────────────────────────────→│
            │  VRRP Advertisement         │
            │  [MASTER, Priority=100]     │

T1: 心跳正常（持续1秒一次）
            │                            │
            ├───────── 心跳 ─────────────→│ [BACKUP收到心跳]
            │ VRRP ADV                   │ [继续等待]
            │ [MASTER, prio=100]         │
            │                            │
            └───────── 心跳 ─────────────→│ [仍是BACKUP]

T2: LVS_MAIN 宕机！网络断连
            │ ✗✗✗ 故障                   │
            │ ✗✗✗ 无法发送心跳           │
            │                            │
            ├────────────────────X       │ [等待心跳]
            │ (无多播)                    │
            │                            │
            │                            │ [T2+3s: 超时检测]
            │                            │ → 未收到MASTER心跳
            │                            │ → 启动故障转移

T3: LVS_BACKUP 接管（转换为MASTER）
            │                            │ [执行转换脚本]
            │                            │ 1. 绑定VIP到ens33:9
            │                            │ 2. 更新ARP
            │                            │ 3. 启动LVS服务
            │                            │ 4. 改state = MASTER
            │                            ↓
            │                      ┌──────────────┐
            │                      │ BACKUP→MASTER│
            │                      │ VIP绑定成功 │
            │                      │ 开始提供服务 │
            │                      └──────────────┘
            │
            │ [LVS_BACKUP 成为新的主]
            │ [开始发送VRRP Advertisement]
            │ ├────────────────────────→ (无LVS_MAIN响应)

T4: LVS_MAIN 恢复！
            │ [重启启动]
            │ [启动Keepalived]
            │ [发送高优先级心跳]
            │ [Priority=100 > 80]
            │                            │
            ├────────────────────→ │ [LVS_BACKUP收到心跳]
            │ VRRP ADV               │ [识别原MASTER已恢复]
            │ [MASTER, prio=100]     │
            │                        │
            │ ✓ 恢复VIP绑定         │ [执行转换脚本]
            │ ✓ 恢复LVS             │ 1. 删除VIP绑定
            │ ✓ 重新为MASTER        │ 2. 更新ARP
            │                        │ 3. 关闭LVS
            │                        │ 4. 改state = BACKUP
            │                        ↓
            │                   ┌──────────────┐
            │                   │ MASTER→BACKUP│
            │                   │ VIP释放      │
            │                   │ 回到待命     │
            │                   └──────────────┘

T5: 系统恢复正常
┌────────────────────────┐      ┌────────────────────────┐
│   LVS_MAIN (MASTER)    │      │  LVS_BACKUP (BACKUP)   │
│  Priority: 100         │      │  Priority: 80          │
│  State: MASTER         │      │  State: BACKUP         │
│                        │      │                        │
│  ✓ VIP: 绑定          │      │  ✗ VIP: 已释放         │
│  ✓ LVS: 提供服务      │      │  ✓ LVS: 待命           │
└───────────┬────────────┘      └────────────┬───────────┘
            │ 心跳: 持续发送 │
            ├────────────────────────────→│
            │ VRRP Advertisement         │
            │ [MASTER, Priority=100]     │

总结时间点：
  T0: 初始化完成（MASTER绑定VIP，BACKUP待命）
  T1-T2: 正常心跳
  T2+3s: BACKUP检测故障（通常3倍心跳间隔）
  T3: BACKUP升级为MASTER，接管VIP
  T4: MAIN恢复，发送高优先级心跳
  T4+3s: BACKUP识别并释放VIP
  T5: 系统恢复正常

关键因素：
  - 心跳间隔: advert_int = 1秒
  - 故障检测: 通常3倍心跳 ≈ 3秒
  - 优先级: MASTER 100 > BACKUP 80（确保可靠转换）
  - 认证: 防止非授权转换
```

### 6.3 LVS+Keepalived 整体高可用拓扑

```
┌───────────────────────────────────────────────────────────────┐
│                        Internet Client                        │
│                     192.168.3.31:random                       │
│  (或内网客户端 192.168.10.33)                                 │
└─────────────┬─────────────────────────────────────────────────┘
              │
              │ 1. 请求VIP: 192.168.10.34:80
              │
    ┌─────────────────────────────────────────────────────────┐
    │               LVS 高可用集群                             │
    │                                                          │
    │  ┌─────────────────────────────────────────────────┐    │
    │  │ [MASTER] LVS_MAIN                              │    │
    │  │ ens33: 192.168.10.30                          │    │
    │  │ VIP: 192.168.10.34 (ens33:9) ← 绑定在主      │    │
    │  │ Priority: 100                                 │    │
    │  │                                               │    │
    │  │ ┌──────────────────────────────────────────┐  │    │
    │  │ │ Keepalived                               │  │    │
    │  │ ├─ VRRP: 发送心跳 (224.0.0.18, 112)       │  │    │
    │  │ ├─ 健康检查                                │  │    │
    │  │ │  ├─ RS1 HTTP_GET http://192.168.10.31/ │  │    │
    │  │ │  ├─ RS2 HTTP_GET http://192.168.10.32/ │  │    │
    │  │ │  └─ 故障自动剔除                        │  │    │
    │  │ └─ LVS 服务管理 (ipvsadm)                 │  │    │
    │  │                                               │    │
    │  │ ┌──────────────────────────────────────────┐  │    │
    │  │ │ IPVS 规则                                │  │    │
    │  │ │ VIP: 192.168.10.34:80                    │  │    │
    │  │ │ 算法: rr (轮询)                          │  │    │
    │  │ │ 模式: DR (直接路由)                      │  │    │
    │  │ │ → RS1: 192.168.10.31:80                 │  │    │
    │  │ │ → RS2: 192.168.10.32:80                 │  │    │
    │  │ └──────────────────────────────────────────┘  │    │
    │  └────┬─────────────────────────────────────────┘    │
    │       │ 2. 调度到RS(转发请求，接收响应)              │
    │       │                                              │
    │  ┌────┴─────────────────────────────────────────────┐    │
    │  │ [BACKUP] LVS_BACKUP                             │    │
    │  │ ens33: 192.168.10.35                           │    │
    │  │ VIP: 192.168.10.34 (ens33:9) ← 未激活          │    │
    │  │ Priority: 80                                   │    │
    │  │                                                │    │
    │  │ ┌──────────────────────────────────────────┐   │    │
    │  │ │ Keepalived                               │   │    │
    │  │ ├─ VRRP: 监听心跳 (224.0.0.18)            │   │    │
    │  │ ├─ 故障转移逻辑（待命）                     │   │    │
    │  │ │  └─ [故障时] 接管VIP，升级为MASTER      │   │    │
    │  │ └─ LVS 配置准备（同主节点）                │   │    │
    │  │                                                │    │
    │  │ ┌──────────────────────────────────────────┐   │    │
    │  │ │ IPVS 规则（与主相同，待激活）            │   │    │
    │  │ │ VIP: 192.168.10.34:80                    │   │    │
    │  │ │ → RS1: 192.168.10.31:80                 │   │    │
    │  │ │ → RS2: 192.168.10.32:80                 │   │    │
    │  │ └──────────────────────────────────────────┘   │    │
    │  └──────────────────────────────────────────────────┘    │
    │                                                          │
    │          [故障转移场景]                                   │
    │          LVS_MAIN故障 ──→ LVS_BACKUP接管VIP              │
    │                                                          │
    └──────────────┬──────────────────────────────────────────┘
                   │ 3. 转发给RS处理
        ┌──────────┴──────────┐
        │                     │
        ↓                     ↓
    ┌───────────┐         ┌───────────┐
    │   RS1     │         │   RS2     │
    │ 192.168.  │         │ 192.168.  │
    │ 10.31:80  │         │ 10.32:80  │
    │           │         │           │
    │ ┌─────┐   │         │ ┌─────┐   │
    │ │ lo  │   │         │ │ lo  │   │
    │ │VIP: │   │         │ │VIP: │   │
    │ │10.34│   │         │ │10.34│   │
    │ └─────┘   │         │ └─────┘   │
    │           │         │           │
    │ ┌──────┐  │         │ ┌──────┐  │
    │ │httpd │  │         │ │httpd │  │
    │ │:80   │  │         │ │:80   │  │
    │ └──────┘  │         │ └──────┘  │
    │           │         │           │
    │ ARP抑制   │         │ ARP抑制   │
    │ arp_ig=1  │         │ arp_ig=1  │
    │ arp_an=2  │         │ arp_an=2  │
    │           │         │           │
    │ 无网关    │         │ 无网关    │
    │ (重要!)   │         │ (重要!)   │
    └───────────┘         └───────────┘
        │ 4. 直接响应Client
        │    (不经过LVS)
        └─────────────────────────────────→

可靠性机制：
┌────────────────────────────────────────────────────────────┐
│ 多层次故障保护                                              │
├────────────────────────────────────────────────────────────┤
│ L1: RS故障检测 (应用层HTTP_GET)                            │
│     └─ 故障RS自动剔除，流量转向正常RS                      │
│                                                            │
│ L2: LVS故障转移 (传输层心跳)                               │
│     └─ LVS宕机时，备机自动接管VIP和LVS服务                 │
│                                                            │
│ L3: 网络隔离保护 (VRRP认证)                                │
│     └─ 密码认证防止非授权节点入侵                          │
│                                                            │
│ 服务连续性：                                                │
│ Client 全程无感知，自动故障转移 ≤ 3秒                     │
└────────────────────────────────────────────────────────────┘
```

---

## 原文勘误

### ⚠️ 勘误1：TUN模式说明不完整（行113-114）

**原文**：
```
TUN(Tunneling)模式需要服务器支持IP隧道...一般不用。 udy32路
```

**问题**：
- "udy32路" 是OCR或编辑错误，无实际含义

**正确表述**：
TUN模式是IP隧道技术，LVS将请求封装在IP包内转发给RS，RS解包后处理，响应直接返回客户端。由于配置复杂、网络设备支持要求高、场景有限，实际使用较少。

---

### ⚠️ 勘误2：RS1/RS2配置IP地址笔误（行178）

**原文**：
```
RS1设置静态IP为192.168.10.31，RS2设置静态IP为192.168.10.3 2。
```

**问题**：
- "192.168.10.3 2" 应为 "192.168.10.32"（多了空格）

**正确表述**：
```
RS1: 192.168.10.31
RS2: 192.168.10.32
```

---

### ⚠️ 勘误3：首页内容设置命令误导（行186-189）

**原文**：
```
3) 设置首页内容(RS2把内容改为this is RS2)
echo this is RS01 > /var/www/html/index.html
```

**问题**：
- 注释说"RS2把内容改为this is RS2"，但代码仍为"RS01"，不清楚RS2应改什么

**正确表述**：
```bash
# RS1 执行
echo "this is RS01" > /var/www/html/index.html

# RS2 执行（修改此处）
echo "this is RS02" > /var/www/html/index.html
```

---

### ⚠️ 勘误4：VIP和DIP概念混淆（行234-236）

**原文**：
```
仅主机网卡一块，IP配置为192.168.3.30，此IP是接受外部请求的VIP

NAT网卡一块，IP配置为192.168.10.30，此IP是与后端RS服务器通信的DIP
```

**问题**：
- 说法不够准确，192.168.3.30 不一定是VIP，可能只是LVS的管理IP
- 实际NAT模式：
  - VIP 可以是 192.168.3.30 或其他网段
  - DIP 必须与RS在同网段（192.168.10.0/24）

---

### ⚠️ 勘误5：DR模式网关配置说明（行356-357）

**原文**：
```
注意：RS1和RS2在之前进行NAT模式实验时设置了网关为LVS的DIP，这里进行DR试验时需要把网关删除
```

**问题**：
- 说法不够清楚，"删除网关"容易引起理解混淆

**正确表述**：
```
DR模式特点：RS和LVS同网段，RS不需要通过网关访问外界。
必须删除RS的网关配置（GATEWAY行），否则会导致：
- RS响应包的源IP冲突
- 数据包转发异常
- 客户端无法正确接收

操作：编辑 /etc/sysconfig/network-scripts/ifcfg-ens33
      删除或注释 GATEWAY 和 GATEWAY0 行
```

---

### ⚠️ 勘误6：网卡配置命令不规范（行368）

**原文**：
```
ifconfig ens33:9 192.168.10.34/24
```

**问题**：
- 子网掩码格式 `/24` 对 `ifconfig` 不兼容

**正确命令**：
```bash
# 方法1：使用 ifconfig（指定掩码）
ifconfig ens33:9 192.168.10.34 netmask 255.255.255.0

# 方法2：使用 ip 命令（支持CIDR）
ip addr add 192.168.10.34/24 dev ens33:9

# 方法3：持久化配置（推荐）
cat > /etc/sysconfig/network-scripts/ifcfg-ens33:9 << EOF
DEVICE=ens33:9
IPADDR=192.168.10.34
NETMASK=255.255.255.0
ONBOOT=yes
EOF
systemctl restart network
```

---

### ⚠️ 勘误7：Keepalived配置文件格式错误（行922）

**原文**：
```
注意：配置文件中的key和大括号之间一定要有空格
```

**问题**：
- 表述不完整，没有清楚说明是哪种情况

**正确说明**：
```tcl
# ✓ 正确格式：key 后跟空格，再是大括号或值
global_defs {
    router_id LVS_MAIN
}

vrrp_instance VI_1 {
    priority 100
}

# ✗ 错误格式：无空格
global_defs{
    router_id LVS_MAIN
}

# ✗ 错误格式：key后直接值
priority=100 (不是Keepalived语法)
```

---

### ⚠️ 勘误8：客户端测试网段混乱（行279）

**原文**：
```
配置网卡为仅主机模式，IP为192.168.3.31，网关无需配置即可。
```

**问题**：
- 前面NAT实验中用 192.168.3.x，此时又改 192.168.10.x
- 后面DR实验改为同网段（192.168.10.33），但说明不清

**正确说明**：
```
NAT 模式测试：
  - 客户端：192.168.3.31（与VIP 192.168.3.30 同网段）
  - 网关：192.168.3.1 或配置为LVS的仅主机网卡

DR 模式测试：
  - 客户端：192.168.10.33（与所有节点同网段）
  - 网关：192.168.10.254 或其他
  - VIP: 192.168.10.34（与LVS/RS同网段）
```

---

### ⚠️ 勘误9：虚拟网卡配置不规范（行351-353）

**原文**：
```
ifconfig lo:9 192.168.10.34
netmask 255.255.255.255
```

**问题**：
- 命令分行，易导致执行错误

**正确命令**：
```bash
# 单行执行（完整）
ifconfig lo:9 192.168.10.34 netmask 255.255.255.255

# 或持久化配置
cat > /etc/sysconfig/network-scripts/ifcfg-lo:9 << EOF
DEVICE=lo:9
IPADDR=192.168.10.34
NETMASK=255.255.255.255
ONBOOT=yes
EOF
systemctl restart network
```

---

### ⚠️ 勘误10：日志文件位置不准确（未明确说明）

**原文**：未提及如何查看Keepalived日志

**补充**：
```bash
# 查看Keepalived日志的多种方法

# 方法1：systemctl 方式（推荐）
systemctl status keepalived
journalctl -u keepalived -f      # 实时跟踪
journalctl -u keepalived -n 50   # 查看最近50行
journalctl -u keepalived --since="10 minutes ago"

# 方法2：日志文件方式
tail -f /var/log/messages        # CentOS/RHEL
tail -f /var/log/syslog          # Ubuntu/Debian

# 方法3：直接启动Keepalived查看输出
systemctl stop keepalived
keepalived -f /etc/keepalived/keepalived.conf -d  # 前台调试模式
```

---

## 速查表

### 快速命令速查

| 任务 | 命令 |
|------|------|
| **安装LVS** | `yum install -y ipvsadm` |
| **安装Keepalived** | `yum install -y keepalived` |
| **添加虚拟服务** | `ipvsadm -A -t VIP:PORT -s 算法` |
| **添加RS(NAT)** | `ipvsadm -a -t VIP:PORT -r RIP:PORT -m` |
| **添加RS(DR)** | `ipvsadm -a -t VIP:PORT -r RIP:PORT -g` |
| **查看规则(IP)** | `ipvsadm -Ln` |
| **查看连接表** | `ipvsadm -Lnc` |
| **清空规则** | `ipvsadm -C` |
| **删除RS** | `ipvsadm -d -t VIP:PORT -r RIP:PORT` |
| **启动Keepalived** | `systemctl start keepalived` |
| **查看Keepalived状态** | `systemctl status keepalived` |
| **检查配置语法** | `keepalived -t` |
| **查看日志** | `journalctl -u keepalived -f` |

### LVS 模式对比速查

| 特性 | NAT | DR | TUN |
|------|-----|-----|-----|
| 工作层 | 网络层(3) | 网络层(3) | 网络层(3) |
| 请求处理 | LVS转发 | LVS转发 | LVS转发 |
| 响应处理 | **LVS处理** | **RS直接** | **RS直接** |
| 性能 | 低 | **高** | 高 |
| 瓶颈 | 明显 | 无 | 无 |
| RS网关 | 必需(DIP) | **禁止** | 可选 |
| 网段要求 | 可不同 | **必须同** | **必须同** |
| 配置复杂度 | 简单 | 中等 | 复杂 |
| 应用场景 | 小规模 | **推荐** | 特殊 |
| ARP抑制 | 无需 | **必需** | **必需** |

### 调度算法对比速查

| 算法 | 类型 | 选择条件 | 特点 |
|------|------|---------|------|
| RR | 静态 | 任何 | 轮转，简单 |
| WRR | 静态 | RS配置不同 | 加权轮转 |
| **WLC** | 动态 | **默认推荐** | **最小连接** |
| SED | 动态 | 长连接 | 初始权重优先 |
| LC | 动态 | 简单情况 | 最小连接 |
| SH | 静态 | 会话粘性 | 同源同RS |
| LBLC | 动态 | Web负载 | 局部性优化 |

### Keepalived 主要配置参数速查

| 参数 | 说明 | 示例值 |
|------|------|--------|
| `state` | MASTER或BACKUP | MASTER |
| `priority` | 优先级 | 100(主)/80(备) |
| `advert_int` | 心跳间隔(秒) | 1 |
| `virtual_router_id` | 虚拟路由ID(主备相同) | 51 |
| `auth_pass` | 认证密码(≤8字符) | keepalived123 |
| `persistence_timeout` | 会话保持(秒) | 0(无) |
| `lb_algo` | 调度算法 | rr, wlc等 |
| `lb_kind` | LVS模式 | NAT, DR, TUN |
| `connect_timeout` | 检测超时(秒) | 3 |
| `nb_get_retry` | 检测重试次数 | 3 |
| `delay_before_retry` | 重试间隔(秒) | 3 |

### 故障排查检查清单

```
□ 检查网络连接
  └─ ping VIP
  └─ ping RS1, RS2
  └─ 确保LVS/RS同网段

□ 检查LVS配置
  └─ ipvsadm -Ln (查看规则)
  └─ ipvsadm -Lnc (查看连接)
  └─ ifconfig (确认VIP绑定)

□ 检查Keepalived配置
  └─ keepalived -t (检查语法)
  └─ systemctl status keepalived
  └─ journalctl -u keepalived -f

□ 检查RS配置(DR模式)
  └─ 确认ARP抑制: cat /proc/sys/net/ipv4/conf/all/arp_ignore
  └─ 确认VIP绑定: ip addr show | grep VIP
  └─ 确认无网关: route -n | grep GATEWAY
  └─ 确认httpd运行: systemctl status httpd
  └─ 测试本地: curl localhost

□ 检查网络安全
  └─ firewall-cmd --list-all
  └─ iptables -L -n

□ 调整Keepalived心跳间隔
  └─ advert_int (减小为加快转移)
  └─ 检测超时: 通常3倍心跳

□ 检查健康检查
  └─ curl 每个RS的检测URL
  └─ 验证HTTP状态码是否为期望值
```

---

**文档完成**
- 章节数：6
- ASCII流程图数：3
- 原文勘误数：10
