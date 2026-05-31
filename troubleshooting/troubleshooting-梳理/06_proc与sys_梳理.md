# 06. /proc 与 /sys 目录介绍 - 知识梳理

## 一、核心概念

### /proc 目录
- **定义**：Linux 虚拟出来的伪文件系统目录
- **功能**：向外输出内核统计信息和进程运行信息
- **特点**：大多数文件不占用磁盘空间；部分文件可写，用于动态修改内核配置
- **关键作用**：许多观测命令（如 `free`、`vmstat`）都通过读取 /proc 文件获取数据

### /sys 目录
- **定义**：Linux 内核虚拟的目录（sysfs 虚拟文件系统）
- **功能**：存放设备配置参数、统计数据、内核参数信息
- **特点**：按功能分类组织；部分文件可写，用于修改设备配置
- **关键作用**：提供统一的设备和内核信息接口

---

## 二、/proc 目录详解

### 1. /proc 顶层文件

| 文件名 | 作用 | 对应命令 |
|-------|------|--------|
| `/proc/stat` | CPU 使用率相关统计 | `vmstat` |
| `/proc/cpuinfo` | CPU 核心信息、频率、缓存等 | `lscpu` |
| `/proc/loadavg` | 系统平均负载 | `uptime` |
| `/proc/meminfo` | 物理内存详细信息 | `free` |
| `/proc/vmstat` | 内存/分页相关统计 | `vmstat` |
| `/proc/diskstats` | 磁盘 I/O 统计信息 | `iostat`、`vmstat` |
| `/proc/mounts` | 文件系统挂载信息 | `mount` |
| `/proc/interrupts` | 硬中断统计 | `irqtop` |
| `/proc/softirqs` | 软中断统计 | `irqtop` |
| `/proc/schedstat` | 线程调度器运行数据 | - |
| `/proc/uptime` | 系统启动时间和空闲时间 | `uptime` |
| `/proc/version` | 内核版本信息 | `uname -a` |
| `/proc/kallsyms` | 内核符号表 | - |
| `/proc/swaps` | Swap 使用情况 | - |
| `/proc/zoneinfo` | 内存区域（zone）统计 | - |
| `/proc/pagetypeinfo` | 内存页面类型统计 | - |
| `/proc/buddyinfo` | 内存伙伴分配器统计 | - |
| `/proc/slabinfo` | SLAB 内存分配器统计 | `slabtop` |

### 2. /proc/sys/* - 内核参数

- **功能**：可读写内核参数，用于动态调整系统行为
- **映射关系**：`/proc/sys/vm/swappiness` ←→ `vm.swappiness` 内核参数
- **操作方式**：
  - 读取：`cat /proc/sys/vm/swappiness`
  - 修改：`echo value > /proc/sys/vm/swappiness` 或使用 `sysctl` 命令
- **特点**：临时修改，重启后失效

### 3. /proc/[pid]/* - 进程相关文件

#### 进程基本信息
| 文件名 | 作用 | 对应命令 |
|-------|------|--------|
| `/proc/[pid]/status` | 进程基本信息（扩展格式） | `pidstat` |
| `/proc/[pid]/cmdline` | 进程启动时的完整命令行 | `ps` |
| `/proc/[pid]/cwd` | 进程当前工作目录（符号链接） | `pwdx` |
| `/proc/[pid]/environ` | 进程环境变量 | - |
| `/proc/[pid]/limits` | 进程资源限制（ulimit） | `prlimit` |
| `/proc/[pid]/cgroup` | 进程所属的 cgroup 控制组 | - |

#### 进程运行/调度信息
| 文件名 | 作用 | 对应命令 |
|-------|------|--------|
| `/proc/[pid]/stat` | 进程基本统计（CPU、内存、状态等） | `ps`、`top` |
| `/proc/[pid]/sched` | 进程调度类、优先级、运行队列等 | - |
| `/proc/[pid]/schedstat` | 进程调度统计（运行时间、等待时间等） | - |
| `/proc/[pid]/task/` | 进程包含的所有线程子目录 | `ps -L` |
| `/proc/[pid]/task/[tid]/syscall` | 线程当前执行的系统调用号和参数 | - |
| `/proc/[pid]/task/[tid]/wchan` | 线程当前阻塞的内核函数名 | - |
| `/proc/[pid]/task/[tid]/stack` | 线程当前的内核栈回溯 | - |

#### 进程内存信息
| 文件名 | 作用 | 对应命令 |
|-------|------|--------|
| `/proc/[pid]/mem` | 进程虚拟内存伪文件（可直接读写进程内存） | - |
| `/proc/[pid]/maps` | 进程虚拟内存映射段（地址范围、权限、文件等） | `pmap` |
| `/proc/[pid]/smaps` | `maps` 的详细版（额外包含内存统计） | `pmap -x` |
| `/proc/[pid]/statm` | 进程内存简要统计（代码、数据、栈等段大小） | - |
| `/proc/[pid]/oom_score` | 进程当前的 OOM 评分（越高越容易被杀） | - |
| `/proc/[pid]/oom_score_adj` | OOM 评分调整参数 | `choom` |

#### 进程 I/O 和网络
| 文件名 | 作用 | 对应命令 |
|-------|------|--------|
| `/proc/[pid]/fd/` | 进程打开的文件描述符（符号链接指向实际文件） | `lsof` |
| `/proc/[pid]/fdinfo/` | 文件描述符的详细信息（打开模式、标志等） | `lsof` |
| `/proc/[pid]/io` | 进程的磁盘 I/O 统计 | `pidstat -d` |
| `/proc/[pid]/net/` | 进程网络命名空间相关信息 | - |

---

## 三、/proc 目录树结构

```
/proc/
├── cpuinfo              # CPU 信息
├── meminfo              # 内存信息
├── loadavg              # 负载信息
├── stat                 # CPU 统计
├── vmstat               # 内存/分页统计
├── diskstats            # 磁盘统计
├── mounts               # 挂载信息
├── interrupts           # 硬中断
├── softirqs             # 软中断
├── uptime               # 启动时间
├── version              # 内核版本
├── swaps                # Swap 状态
├── buddyinfo            # 内存伙伴信息
├── slabinfo             # SLAB 信息
├── sys/
│   ├── vm/
│   │   ├── swappiness   # Swap 倾向
│   │   └── ...
│   ├── kernel/
│   └── ...
├── net/
│   ├── dev              # 网络设备统计
│   ├── tcp              # TCP 连接
│   ├── udp              # UDP 连接
│   └── ...
│
└── [pid]/               # 进程相关（每个进程一个目录）
    ├── status           # 进程状态摘要
    ├── cmdline          # 命令行
    ├── cwd              # 工作目录 → 真实路径
    ├── environ          # 环境变量
    ├── limits           # 资源限制
    ├── cgroup           # cgroup 信息
    ├── stat             # 进程统计
    ├── sched            # 调度信息
    ├── schedstat        # 调度统计
    ├── maps             # 内存映射
    ├── smaps            # 内存映射（详细）
    ├── mem              # 虚拟内存伪文件
    ├── statm            # 内存统计简要
    ├── oom_score        # OOM 评分
    ├── oom_score_adj    # OOM 评分调整
    ├── fd/              # 文件描述符
    │   ├── 0            # stdin
    │   ├── 1            # stdout
    │   ├── 2            # stderr
    │   └── ...
    ├── fdinfo/          # 文件描述符详情
    ├── io               # I/O 统计
    ├── net/             # 网络信息
    └── task/            # 线程信息
        └── [tid]/
            ├── syscall  # 当前系统调用
            ├── wchan    # 阻塞函数
            └── stack    # 内核栈
```

---

## 四、/proc/[pid] 主要文件作用流程图

```
进程 ID (PID)
    |
    ├─→ /proc/[pid]/status ──→ 进程基本信息（状态、UID、GID、VmRSS等）
    |
    ├─→ /proc/[pid]/cmdline ──→ 启动命令行
    |
    ├─→ /proc/[pid]/stat ──→ CPU 时间、内存、状态代码等
    |
    ├─→ 内存相关
    |   ├─→ /proc/[pid]/maps ──→ 虚拟内存段映射
    |   ├─→ /proc/[pid]/smaps ──→ 内存段详细统计
    |   ├─→ /proc/[pid]/statm ──→ 简要内存统计
    |   └─→ /proc/[pid]/oom_score ──→ OOM 评分
    |
    ├─→ 调度相关
    |   ├─→ /proc/[pid]/sched ──→ 调度类、优先级
    |   └─→ /proc/[pid]/schedstat ──→ 调度统计
    |
    ├─→ 线程信息
    |   └─→ /proc/[pid]/task/[tid]/
    |       ├─→ syscall ──→ 当前系统调用
    |       ├─→ wchan ──→ 阻塞内核函数
    |       └─→ stack ──→ 内核栈回溯
    |
    ├─→ 文件描述符
    |   ├─→ /proc/[pid]/fd/ ──→ 打开的文件（符号链接）
    |   └─→ /proc/[pid]/fdinfo/ ──→ 文件描述符详情
    |
    └─→ I/O 信息
        └─→ /proc/[pid]/io ──→ 磁盘 I/O 统计
```

---

## 五、/proc 案例：无 lsof 时的替代方案

当系统中没有 `lsof` 命令且无法安装时，可以通过 `/proc` 目录查看进程打开的文件：

### 场景
- 需要查看进程 PID 1234 打开了哪些文件
- 系统不可用 `lsof` 命令

### 解决方案

```bash
# 方法 1：直接列出符号链接
ls -la /proc/1234/fd/

# 输出示例：
# lrwx------ 1 user group 64 May 24 10:30 0 -> /dev/pts/0
# lrwx------ 1 user group 64 May 24 10:30 1 -> /dev/pts/0
# lrwx------ 1 user group 64 May 24 10:30 2 -> /dev/pts/0
# lrwx------ 1 user group 64 May 24 10:30 3 -> /var/log/app.log
# lrwx------ 1 user group 64 May 24 10:30 4 -> socket:[12345678]

# 方法 2：显示文件描述符对应的文件类型和信息
readlink /proc/1234/fd/*

# 方法 3：结合 fdinfo 查看更详细的打开模式
cat /proc/1234/fdinfo/3

# 输出示例：
# pos:    0
# flags:  0100002   # O_WRONLY|O_CREAT
# mnt_id: 25
```

### 核心原理
- `/proc/[pid]/fd/` 目录中的每个数字都是一个符号链接，指向进程打开的文件
- 符号链接的目标是实际文件路径或特殊资源（如 socket、管道、设备）
- `/proc/[pid]/fdinfo/` 提供每个文件描述符的额外信息（打开标志、文件偏移等）

---

## 六、/sys 目录详解

### 1. /sys 概述

| 特性 | 描述 |
|------|------|
| 虚拟文件系统 | sysfs 虚拟文件系统 |
| 主要用途 | 设备配置、设备统计、内核参数 |
| 组织方式 | 分类层级结构 |
| 可写性 | 部分文件可写，用于修改设备配置 |

### 2. /sys 设备相关子目录

| 子目录 | 作用 | 例子 |
|-------|------|-----|
| `/sys/devices/` | 系统中所有设备的配置与统计信息（按物理地址）| `/sys/devices/pci0000:00/` |
| `/sys/class/` | 设备按功能分类组织 | `/sys/class/net/` (网卡)、`/sys/class/block/` (块设备) |
| `/sys/block/` | 块设备专用（磁盘、分区等）| `/sys/block/sda/stat` (iostat 数据源) |
| `/sys/bus/` | 设备按总线拓扑组织 | `/sys/bus/pci/`、`/sys/bus/usb/` |
| `/sys/dev/` | 块设备和字符设备的主次设备号 | - |

### 3. /sys 内核信息相关子目录

| 子目录 | 作用 | 例子 |
|-------|------|-----|
| `/sys/fs/` | 文件系统相关配置和状态信息 | `/sys/fs/cgroup/` (控制组) |
| `/sys/fs/cgroup/` | **控制组（cgroup）配置**，用于资源隔离和限制 | `/sys/fs/cgroup/cpu.stat`、`/sys/fs/cgroup/memory.stat` |
| `/sys/kernel/` | 内核的配置与统计信息 | `/sys/kernel/debug/` |
| `/sys/module/` | 已加载内核模块的配置与统计 | `/sys/module/ext4/` |

### 4. /sys 目录树结构

```
/sys/
├── devices/              # 所有设备（按物理拓扑）
│   ├── pci0000:00/       # PCI 设备
│   ├── platform/         # 平台设备
│   └── ...
│
├── class/                # 设备分类视图
│   ├── net/              # 网络接口
│   │   ├── eth0/
│   │   ├── ens33/
│   │   │   └── statistics/
│   │   │       ├── rx_packets
│   │   │       ├── tx_packets
│   │   │       └── ...
│   │   └── ...
│   ├── block/            # 块设备
│   ├── mem/              # 内存设备
│   └── ...
│
├── block/                # 块设备专用
│   ├── sda/
│   │   ├── stat         # I/O 统计（iostat 数据源）
│   │   ├── queue/
│   │   │   └── scheduler
│   │   └── sda1/        # 分区
│   └── ...
│
├── bus/                  # 总线组织
│   ├── pci/
│   ├── usb/
│   └── ...
│
├── fs/
│   ├── cgroup/          # 控制组信息
│   │   ├── cpu.stat     # CPU 使用统计
│   │   ├── memory.stat  # 内存使用统计
│   │   └── ...
│   └── ext4/            # 文件系统
│
├── kernel/              # 内核信息
│   ├── debug/
│   ├── mm/              # 内存管理
│   └── ...
│
└── module/              # 已加载模块
    ├── ext4/
    ├── nf_conntrack/
    └── ...
```

### 5. 网卡配置查询示例

```bash
# 查看 ens33 网卡的 MTU、速度、MAC 地址等
cat /sys/class/net/ens33/mtu
cat /sys/class/net/ens33/speed
cat /sys/class/net/ens33/address

# 查看网络接口统计
cat /sys/class/net/ens33/statistics/rx_packets   # 接收包数
cat /sys/class/net/ens33/statistics/tx_packets   # 发送包数
cat /sys/class/net/ens33/statistics/rx_bytes     # 接收字节
cat /sys/class/net/ens33/statistics/tx_bytes     # 发送字节
```

---

## 七、/sys/block 与 iostat 的关系

```bash
# iostat 命令读取磁盘统计数据来自：
cat /sys/block/sda/stat

# 输出示例：
# 8 0 sda 1234 5678 9012 3456 ...
# 字段说明：
# 1. 读完成次数
# 2. 读合并次数
# 3. 读扇区数
# 4. 读耗时（ms）
# 5. 写完成次数
# 6. 写合并次数
# 7. 写扇区数
# 8. 写耗时（ms）
# ...
```

---

## 八、/proc 与 /sys 关系总结

| 特性 | /proc | /sys |
|------|-------|------|
| **虚拟文件系统** | procfs | sysfs |
| **主要功能** | 内核、进程运行信息 | 设备配置、设备统计、内核参数 |
| **组织方式** | 按进程 ID 和系统模块 | 按设备分类、总线拓扑、内核模块 |
| **可读性** | 大部分可读 | 大部分可读 |
| **可写性** | 部分文件可写 | 部分文件可写 |
| **常用命令** | free、vmstat、ps、top、pidstat | iostat、lscpu、ethtool |
| **数据源** | free ← /proc/meminfo | iostat ← /sys/block/*/stat |

---

## 九、常用场景应用

### 1. 查看系统内存使用
```bash
cat /proc/meminfo | head -10
# 相当于 free 命令
```

### 2. 查看进程调度延迟
```bash
cat /proc/[pid]/schedstat
# 自定义 Python 脚本可读此文件获取调度延迟
```

### 3. 查看进程 OOM 评分
```bash
cat /proc/[pid]/oom_score
cat /proc/[pid]/oom_score_adj
# 调整优先级：echo -500 > /proc/[pid]/oom_score_adj
```

### 4. 查看进程网络命名空间
```bash
ls -la /proc/[pid]/net/
# 在容器/命名空间中能看到该进程的网络接口
```

### 5. 动态修改内核参数
```bash
# 查看
cat /proc/sys/vm/swappiness

# 临时修改（重启失效）
echo 10 > /proc/sys/vm/swappiness

# 持久修改
sysctl vm.swappiness=10
```

---

## 十、速查表

### /proc 常用文件快速查询

| 需求 | 查看文件 | 相应命令 |
|------|---------|--------|
| 内存使用 | `/proc/meminfo` | `free` |
| CPU 使用 | `/proc/stat` | `top`、`vmstat` |
| 系统负载 | `/proc/loadavg` | `uptime` |
| 磁盘 I/O | `/proc/diskstats` | `iostat` |
| 网络连接 | `/proc/net/tcp`、`/proc/net/udp` | `netstat` |
| 进程内存 | `/proc/[pid]/maps`、`/proc/[pid]/statm` | `pmap` |
| 打开文件 | `/proc/[pid]/fd/` | `lsof` |
| 进程状态 | `/proc/[pid]/status` | `pidstat` |
| 系统调用 | `/proc/[pid]/task/[tid]/syscall` | `strace` |
| 阻塞位置 | `/proc/[pid]/task/[tid]/wchan` | - |
| 内核栈 | `/proc/[pid]/task/[tid]/stack` | `dmesg` |

### /sys 常用文件快速查询

| 需求 | 查看文件 | 相应命令 |
|------|---------|--------|
| 磁盘 I/O 统计 | `/sys/block/sda/stat` | `iostat` |
| 网卡信息 | `/sys/class/net/eth0/` | `ethtool` |
| CPU 信息 | `/proc/cpuinfo` | `lscpu` |
| 控制组资源 | `/sys/fs/cgroup/cpu.stat` | - |
| 已加载模块 | `/sys/module/` | `lsmod` |

---

## 十一、原文勘误

### 勘误 1：文件路径表述
**原文位置**：第 1095 行  
**原文**：`/proc/net/*` 网络相关信息

**勘误**：表述过于宽泛。应细化为：
- `/proc/net/tcp` - TCP 连接信息
- `/proc/net/udp` - UDP 连接信息
- `/proc/net/dev` - 网络接口统计
- `/proc/net/route` - 路由表

**说明**：`/proc/net/` 下包含多个具体的网络统计文件，不同文件提供不同的信息。

---

### 勘误 2：命令对应关系遗漏
**原文位置**：第 1133 行  
**原文**：`/proc/[pid]/stat` 进程基本信息，如 cpu、内存等 | prtstat/ps

**勘误**：应补充：
- 对应命令：`ps`、`top`、`pidstat`（注：原文 `prtstat` 笔误，应为 `pidstat`）

**说明**：`/proc/[pid]/stat` 是 ps、top、pidstat 的主要数据源，这三个命令都会读取此文件。

---

### 勘误 3：/proc/net 与进程网络隔离的描述不清
**原文位置**：第 1159 行  
**原文**：`/proc/[pid]/net/*` 进程网络相关信息

**勘误**：应补充说明：
- 在容器或网络命名空间中，`/proc/[pid]/net/` 是该命名空间的网络接口视图
- 与 `/proc/net/` 可能不同，反映的是进程隔离后的网络拓扑

**说明**：在 Docker 容器或 K8s Pod 中尤其重要，因为容器有独立的网络命名空间。

---

### 勘误 4：cgroup 信息位置
**原文位置**：第 1206-1208 行  
**原文**：常用的容器相关信息在 `/sys/fs/cgroup` 中

**勘误**：cgroup 信息同时存在于两个位置：
1. `/proc/[pid]/cgroup` - 显示该进程所属的 cgroup
2. `/sys/fs/cgroup/` - 显示 cgroup 本身的资源限制和统计

**说明**：原文只强调了 `/sys/fs/cgroup`，应同时提及 `/proc/[pid]/cgroup`。

---

### 勘误 5：OOM 相关命令遗漏
**原文位置**：第 1149 行  
**原文**：`/proc/[pid]/oom_score_adj` ... | choom

**勘误**：应补充说明：
- 对应命令：`choom`（改变 OOM 评分调整值）
- 但查看当前评分更常用：直接 `cat /proc/[pid]/oom_score`

**说明**：`choom` 命令的用法示例应补充说明。

---

## 十二、扩展知识点

### /proc 与命令实现原理

许多系统命令的实现原理是读取 `/proc` 文件：

| 命令 | 数据源 | 作用 |
|------|--------|------|
| `free` | `/proc/meminfo` | 显示内存使用统计 |
| `uptime` | `/proc/loadavg`、`/proc/uptime` | 显示系统运行时间和负载 |
| `top` | `/proc/stat`、`/proc/[pid]/stat` | 实时系统和进程监控 |
| `ps` | `/proc/[pid]/stat`、`/proc/[pid]/cmdline` | 进程列表 |
| `vmstat` | `/proc/stat`、`/proc/vmstat`、`/proc/diskstats` | 虚拟内存和 I/O 统计 |
| `iostat` | `/sys/block/*/stat` | 磁盘 I/O 统计 |
| `pidstat` | `/proc/[pid]/stat`、`/proc/[pid]/schedstat` | 进程统计 |
| `lsof` | `/proc/[pid]/fd/`、`/proc/[pid]/fdinfo/` | 打开文件列表 |
| `pmap` | `/proc/[pid]/maps`、`/proc/[pid]/smaps` | 进程内存映射 |

**启示**：熟悉这些 `/proc` 文件后，可以基于它们编写自己的监控和诊断工具（如 Python 脚本读取调度延迟）。

---

## 十三、学习资源

### 手册查询

```bash
# 查看 /proc 目录详细说明
man proc

# 查看 /sys 目录详细说明
man 5 sysfs

# 查看特定命令的数据源
man 1 free    # 了解 free 读取 /proc/meminfo 的细节
man 1 top     # 了解 top 的数据源
```

---

**笔记完成于**：2026 年 5 月 24 日

