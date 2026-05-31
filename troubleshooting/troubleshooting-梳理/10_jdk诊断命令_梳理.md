# JDK 诊断命令介绍 - 梳理

> **原文源位置**: ppt-merged.md 第 1764-1887 行  
> **主题**: jps、jstack、jstat、jmap、MAT 内存分析、jinfo、jcmd 完整梳理

---

## 目录

1. [概述](#概述)
2. [知识点详解](#知识点详解)
   - [2.1 jps 命令](#jps-命令)
   - [2.2 jstack 命令](#jstack-命令)
   - [2.3 jstat 命令](#jstat-命令)
   - [2.4 jmap 命令](#jmap-命令)
   - [2.5 MAT 内存分析](#mat-内存分析)
   - [2.6 jinfo 命令](#jinfo-命令)
   - [2.7 jcmd 命令](#jcmd-命令)
3. [ASCII 流程图](#ascii-流程图)
4. [原文勘误](#原文勘误)
5. [速查表](#速查表)

---

## 概述

JDK 诊断命令是 Java 开发者进行运行时故障排查、性能分析的核心工具集。这些命令可以帮助定位线程死锁、内存泄漏、GC 异常等问题。从 JDK 7 开始，`jcmd` 成为了统一的诊断入口，逐步替代散布的独立命令。

**核心命令家族**:
- **进程查询**: jps
- **线程诊断**: jstack、jcmd Thread.print
- **GC 监控**: jstat、jcmd GC.heap_info
- **堆内存分析**: jmap、jcmd GC.heap_dump、MAT
- **JVM 配置查询**: jinfo、jcmd VM.flags
- **统一入口**: jcmd (JDK 7+)

---

## 知识点详解

### jps 命令

**功能**: 列举本机所有 Java 进程（Java Process Status）

**用途**:
- 获取 Java 进程 ID (PID)
- 快速定位目标应用进程
- 是其他诊断工具的前置步骤

**基本语法**:
```bash
jps [options]
jps -l          # 显示完整的类名
jps -v          # 显示传递给 JVM 的参数
jps -m          # 显示传递给 main() 的参数
```

**应用场景**:
1. 多个 Java 应用同时运行时，快速定位目标进程
2. 与 jstack、jmap 等命令配合使用

---

### jstack 命令

**功能**: 获取 Java 线程栈快照（Java Stack Trace）

**用途**:
- 查看当前各个线程的执行状态
- 定位线程死锁问题
- 分析线程阻塞原因
- 检查无限循环或长时间运行的代码段

**基本语法**:
```bash
jstack <pid>           # 输出线程堆栈
jstack -l <pid>        # 显示锁相关信息，用于死锁检测
jstack -F <pid>        # 强制执行（如果 jstack 挂起）
jstack -m <pid>        # 打印混合的 Java 和本地 C/C++ 帧
```

**输出内容解读**:
- **Thread ID**: 线程 ID 和线程名称
- **prio**: 线程优先级
- **tid**: 线程内部 ID
- **nid**: 本地线程 ID（十六进制）
- **State**: 线程状态（RUNNABLE、WAITING、BLOCKED 等）
- **Lock**: 锁信息（用于死锁分析）

**线程状态说明**:
- `RUNNABLE`: 线程正在执行或等待 CPU 时间片
- `WAITING`: 线程等待其他线程的通知（Object.wait()、Thread.join()）
- `TIMED_WAITING`: 等待超时的状态（sleep、wait(timeout)）
- `BLOCKED`: 线程被阻塞，等待获取监视器锁
- `NEW`: 新创建的线程
- `TERMINATED`: 已终止的线程

**关键应用**:
- **死锁检测**: 查找互相等待的线程对，通过 lock 信息追踪
- **性能瓶颈**: 查看哪些线程经常处于 BLOCKED/WAITING 状态

---

### jstat 命令

**功能**: 查看 JVM 统计数据（JVM Statistics）

**监控对象**:
- **GC 统计**: 垃圾回收次数、时间、堆内存占用
- **类加载统计**: 已加载/卸载的类数量
- **JIT 编译**: 编译方法数、编译时间

**基本语法**:
```bash
jstat -gc <pid> [interval] [count]        # GC 统计
jstat -gccapacity <pid>                   # 堆内存容量统计
jstat -gcutil <pid>                       # GC 占用比例统计
jstat -class <pid>                        # 类加载统计
jstat -compiler <pid>                     # JIT 编译统计
```

**GC 输出列说明**:
- `S0`: Survivor 0 区占用比例 (%)
- `S1`: Survivor 1 区占用比例 (%)
- `E`: Eden 区占用比例 (%)
- `O`: Old 区占用比例 (%)
- `M/CCS`: 元空间/压缩指针占用比例 (%)
- `YGC`: Young GC 次数
- `YGCT`: Young GC 总耗时 (s)
- `FGC`: Full GC 次数
- `FGCT`: Full GC 总耗时 (s)
- `GCT`: 总 GC 耗时 (s)

**应用场景**:
- 实时监控 GC 频率和耗时
- 快速判断是否存在内存泄漏（Old 区持续增长）
- 评估垃圾回收器效率

---

### jmap 命令

**功能**: 分析 Java 堆内存（Java Memory Map）

**主要功能**:

#### 1. 查看堆使用概况
```bash
jmap -heap <pid>              # 显示堆的使用情况
jmap -histo <pid>             # 显示对象分布（按大小排序）
jmap -histo:live <pid>        # 只显示活跃对象分布
```

**输出示例**:
```
num     #instances         #bytes  class name
--------------------------------------------------
   1:         12345       987654  [C (char 数组)
   2:          8765       456789  java.lang.String
   3:          1234       123456  java.util.HashMap
   ...
```

#### 2. 堆内存 Dump
```bash
jmap -dump:live,format=b,file=heap.hprof <pid>
# live: 仅 dump 存活对象（可选）
# format=b: 二进制 HPROF 格式（不支持其他格式）
# file: 输出文件路径
```

**重要注意**:
- ⚠️ **该命令可能导致 JVM 长时间暂停**（Stop-The-World）
- 建议先摘除流量，再执行操作
- 生成的 HPROF 文件为二进制格式，无法直接查看
- 文件通常较大，建议 gzip 压缩后再下载

#### 3. 自动 Dump 配置
在应用启动时添加 JVM 参数，使得 OOM 时自动生成堆 dump 文件：
```bash
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/home/work/logs/applogs/
```

**优势**:
- OOM 异常时自动捕获堆快照
- 便于事后分析
- 避免手动操作时的 STW 问题

---

### MAT 内存分析

**工具**: Memory Analysis Tool（堆内存可视化分析工具）

**分析流程** (3 个关键步骤):

#### 第一步: 查找 GC Root
- 在 MAT 中打开 HPROF 文件
- 使用"Dominator Tree"或"GC Roots"视图
- 识别占用内存最多的 GC Root 对象

**目标**: 找到造成内存泄漏的根源对象

#### 第二步: 追踪 GC Root 源头
如果 GC Root 是**线程对象**:
- 在 MAT 中查看该线程栈
- 逐层展开线程的本地变量和引用
- 找出哪个对象被线程长期持有

#### 第三步: 识别问题代码
展开线程栈的引用链，通常会发现:
- **大量查询结果被缓存**: 如 SQL 查询结果未及时释放
- **循环引用**: 对象之间互相引用无法释放
- **监听器未注销**: Event Listener 持有过大的对象引用

**实例分析**:
原文提到的案例：某个 SQL 查询返回了大量数据被 ResultSet 对象持有，通过线程栈追踪发现这个 ResultSet 被业务逻辑层缓存，最终导致堆占用高。

**使用建议**:
- 打开 HPROF 文件后，先查看"Leak Suspects Report"（泄漏嫌疑报告）
- 分析"Dominator Tree"中的大对象
- 右键"Analyze Retained Objects"查看对象持有的内存

---

### jinfo 命令

**功能**: 查看和设置 JVM 配置项（Java Information）

**用途**:
- 查看 JVM 运行时参数
- 查看系统属性（System Properties）
- 动态修改某些 JVM 标志（限制条件较多）

**基本语法**:
```bash
jinfo <pid>                    # 显示所有配置和系统属性
jinfo -flags <pid>             # 仅显示 JVM 参数
jinfo -sysprops <pid>          # 仅显示系统属性
jinfo -flag +<flag> <pid>      # 打开某个标志
jinfo -flag -<flag> <pid>      # 关闭某个标志
jinfo -flag <flag>=<value> <pid> # 设置标志值
```

**输出示例**:
```
VM Arguments: -Xmx4096m -Xms2048m -XX:+UseG1GC
System Properties: java.version=11.0.1, os.name=Linux ...
```

**限制**:
- 只能修改 manageable 的标志
- 大部分 GC 参数在启动后无法修改

---

### jcmd 命令

**背景**: 从 JDK 7 开始引入，用来统一和扩展 JDK 诊断功能

**设计理念**:
- 替代散布的命令（jps、jstack、jmap、jinfo）
- 作为新功能的统一入口
- 支持传递给进程 ID `0` 在所有 Java 进程上执行

**基本语法**:
```bash
jcmd <pid> <command> [command-args]
jcmd 0 <command>               # 在所有进程上执行
```

**常用命令映射关系**:

| jcmd 子命令 | 等同于 | 功能 |
|---|---|---|
| `jcmd` | `jps` | 列举 Java 进程 |
| `jcmd <pid> Thread.print` | `jstack <pid>` | 获取线程栈 |
| `jcmd <pid> GC.heap_info` | `jmap -heap` | 查看堆信息 |
| `jcmd <pid> GC.class_histogram` | `jmap -histo` | 对象分布统计 |
| `jcmd <pid> GC.heap_dump /path/heap.hprof` | `jmap -dump:file=...` | 堆转储 |
| `jcmd <pid> VM.flags` | `jinfo -flags` | JVM 参数 |
| `jcmd <pid> VM.system_properties` | `jinfo -sysprops` | 系统属性 |
| `jcmd <pid> VM.command_line` | `jinfo` | 命令行 |
| `jcmd <pid> PerfCounter.print` | `jstat` | GC 等数据 |

**新增高级功能** (JDK 8+):

| 子命令 | 描述 |
|---|---|
| `jcmd 0 VM.native_memory` | 查看 JVM Native 内存分配情况 |
| `jcmd 0 GC.run` | 手动触发 Full GC |
| `jcmd 0 GC.rotate_log` | 强制滚动 GC 日志 |
| `JFR.configure` | JFR（Java Flight Recorder）配置 |
| `JFR.start` / `JFR.stop` | 启动/停止 JFR 录制 |
| `JFR.dump` | 导出 JFR 录制数据 |
| `JFR.check` | 检查 JFR 状态 |

**优势**:
- 统一接口，学习成本低
- 支持批量操作（PID=0）
- 新功能优先通过 jcmd 提供
- 参数更灵活（如 GC.heap_dump 支持指定输出路径）

---

## ASCII 流程图

### 流程图 1: 线程死锁定位完整链路

```
┌──────────────────────────────────────────────────────────────────┐
│              线程死锁问题定位 - jstack 诊断流程                    │
└──────────────────────────────────────────────────────────────────┘

   应用出现线程等待/卡顿现象
             │
             ▼
    ┌──────────────────┐
    │  第 1 步: 获取 PID │
    │  $ jps -l         │
    └──────────────────┘
             │
             ▼
    ┌─────────────────────────────────┐
    │  第 2 步: 获取线程栈快照          │
    │  $ jstack <pid> > dump.txt       │
    │  或 jcmd <pid> Thread.print      │
    └─────────────────────────────────┘
             │
             ▼
    ┌──────────────────────────────────────┐
    │  第 3 步: 分析 dump 文件              │
    │  1. 搜索关键字: "deadlock"           │
    │  2. 查找 BLOCKED 状态的线程           │
    │  3. 跟踪 lock address                │
    └──────────────────────────────────────┘
             │
             ▼
    ┌──────────────────────────────────────┐
    │  第 4 步: 查看线程栈详情              │
    │  逐行分析:                           │
    │  - at java.lang.Object.wait()      │
    │  - locked <0x00007f8b2c0d0a00>     │
    │  - waiting to lock <0x...>         │
    └──────────────────────────────────────┘
             │
             ▼
    ┌──────────────────────────────────────┐
    │  第 5 步: 定位代码                    │
    │  1. 找出持有锁的线程                  │
    │  2. 找出等待锁的线程                  │
    │  3. 形成完整的循环等待链              │
    │  4. 定位源代码位置修复                │
    └──────────────────────────────────────┘
             │
             ▼
        [问题解决]
```

### 流程图 2: 内存泄漏定位完整链路

```
┌──────────────────────────────────────────────────────────────────┐
│            内存泄漏问题诊断 - jmap + MAT 分析流程                   │
└──────────────────────────────────────────────────────────────────┘

   应用内存占用持续增长或 OOM 异常
             │
             ▼
    ┌───────────────────────────────────────┐
    │  第 1 步: 生成堆 dump 文件             │
    │  摘除应用流量                         │
    │  $ jmap -dump:live,format=b,           │
    │         file=heap.hprof <pid>         │
    │  或在启动参数中添加:                  │
    │  -XX:+HeapDumpOnOutOfMemoryError       │
    │  -XX:HeapDumpPath=/path/to/logs/      │
    └───────────────────────────────────────┘
             │
             ▼
    ┌─────────────────────────────────┐
    │  第 2 步: 压缩 dump 文件         │
    │  $ gzip heap.hprof              │
    │  下载到本地开发环境              │
    └─────────────────────────────────┘
             │
             ▼
    ┌──────────────────────────────────────┐
    │  第 3 步: 在 MAT 中打开 dump 文件     │
    │  File → Open Heap Dump               │
    │  选择 heap.hprof 文件                │
    └──────────────────────────────────────┘
             │
             ▼
    ┌──────────────────────────────────────┐
    │  第 4 步: 查找 GC Root                │
    │  Dominator Tree → 按内存降序排列      │
    │  找出占用内存最多的对象               │
    └──────────────────────────────────────┘
             │
             ▼
    ┌──────────────────────────────────────┐
    │  第 5 步: 追踪引用关系                │
    │  若 GC Root 是线程:                  │
    │  ├─ List Details 查看线程栈          │
    │  └─ 展开局部变量和字段引用            │
    │  若 GC Root 是其他对象:              │
    │  └─ Path to GC Root 查看链路         │
    └──────────────────────────────────────┘
             │
             ▼
    ┌──────────────────────────────────────┐
    │  第 6 步: 识别问题                    │
    │  常见泄漏源:                         │
    │  1. ResultSet/Connection 未关闭      │
    │  2. 静态集合无限增长                  │
    │  3. 监听器/回调未注销                │
    │  4. 定时任务错误处理                  │
    │  5. 线程池任务堆积                    │
    └──────────────────────────────────────┘
             │
             ▼
    ┌──────────────────────────────────────┐
    │  第 7 步: 修复代码                    │
    │  定位源代码，确保:                    │
    │  ✓ 资源及时释放                      │
    │  ✓ 循环引用断开                      │
    │  ✓ 监听器正确注销                    │
    └──────────────────────────────────────┘
             │
             ▼
        [内存正常]
```

### 流程图 3: GC 性能诊断链路

```
┌──────────────────────────────────────────────────────────────────┐
│            GC 性能异常诊断 - jstat 监控 + 分析流程                 │
└──────────────────────────────────────────────────────────────────┘

   观察应用响应时间增加/延迟毛刺
             │
             ▼
    ┌────────────────────────────────┐
    │  第 1 步: 获取 PID              │
    │  $ jps -l                       │
    └────────────────────────────────┘
             │
             ▼
    ┌────────────────────────────────────────┐
    │  第 2 步: 实时监控 GC 统计             │
    │  $ jstat -gc <pid> 1000 60             │
    │  (每秒统计一次，共 60 次)              │
    │  观察指标:                             │
    │  ✓ S0, S1, E, O 占用趋势              │
    │  ✓ YGC/FGC 频率                       │
    │  ✓ YGCT/FGCT 耗时                     │
    └────────────────────────────────────────┘
             │
             ▼
    ┌──────────────────────────────────┐
    │  第 3 步: 判断异常特征            │
    │                                  │
    │  异常情形 A:                     │
    │  ├─ YGC 频繁 (每秒多次)          │
    │  ├─ YGCT 占比高                  │
    │  └─ 原因: Eden 区过小             │
    │                                  │
    │  异常情形 B:                     │
    │  ├─ Old 区持续增长               │
    │  ├─ FGC 频繁                     │
    │  └─ 原因: 内存泄漏或堆过小       │
    │                                  │
    │  异常情形 C:                     │
    │  ├─ 单次 GC 耗时长 (FGCT > 1s)  │
    │  └─ 原因: 堆内存过大或 GC 器差   │
    └──────────────────────────────────┘
             │
             ▼
    ┌────────────────────────────────────────┐
    │  第 4 步: 按特征采取措施               │
    │                                        │
    │  情形 A 解决方案:                      │
    │  └─ 增大 Eden 区                       │
    │     -XX:NewSize= -XX:MaxNewSize=      │
    │                                        │
    │  情形 B 解决方案:                      │
    │  ├─ 使用 jmap -histo 查找大对象       │
    │  ├─ 或用 jmap -dump 生成 heap dump   │
    │  └─ 在 MAT 中分析内存泄漏             │
    │                                        │
    │  情形 C 解决方案:                      │
    │  ├─ 增加堆内存 -Xmx                    │
    │  ├─ 或切换 GC 器 (G1GC/ZGC)           │
    │  └─ 调整 GC 参数优化吞吐量             │
    └────────────────────────────────────────┘
             │
             ▼
    ┌────────────────────────────────────────┐
    │  第 5 步: 验证改进                     │
    │  再次运行 jstat 对比优化前后的指标    │
    │  确保 GC 频率下降，耗时减少           │
    └────────────────────────────────────────┘
             │
             ▼
        [GC 性能正常]
```

---

## 原文勘误

### 勘误 1: jstat 命令的输出列名不完整

**原文位置**: 第 1792-1800 行

**原文内容**:
```
jstat用来查看jvm gc、类加载、JIT编译情况。
```

**问题**: 
原文未明确列举 jstat 输出的各列含义，容易让初学者困惑。

**建议修正**:
```
jstat 可查看 JVM 的多项运行时数据:
- GC 统计: -gc (包含 S0/S1/E/O/M 占用比例，YGC/FGC 次数及耗时)
- 容量统计: -gccapacity (显示各代的最小/当前/最大容量)
- 占用比例: -gcutil (显示各代占用百分比，常用于快速判断)
- 类加载: -class (已加载/卸载的类数)
- JIT 编译: -compiler (编译方法数和编译耗时)
```

---

### 勘误 2: jmap 导致 JVM 暂停的风险表述不够突出

**原文位置**: 第 1824 行

**原文内容**:
```
注：此命令可能会导致jvm长时间暂停，建议摘除流量后，再操作。
```

**问题**:
表述相对温和，初学者可能未意识到这是生产环境的 **严重风险**。

**建议修正**:
```
⚠️ 重要警告: jmap -dump 执行时会触发 Full GC 并导致 Stop-The-World (STW)，
JVM 所有线程会完全暂停，暂停时长取决于堆大小(可能 10秒-几分钟)。
- 生产环境必须先摘除应用流量
- 考虑在流量低谷时段执行
- 大堆内存(>10GB)场景建议改用自动 dump 功能
  (-XX:+HeapDumpOnOutOfMemoryError 仅在 OOM 时 dump，减少主动操作)
```

---

### 勘误 3: MAT 分析步骤的描述过于简化

**原文位置**: 第 1826-1840 行

**原文内容**:
```
第一步：找GC Root
第二步：若GC Root是线程，查看线程栈
第三步：展开线程栈，这个SQL查询了大量数量导致堆占用高！
```

**问题**:
1. 未说明如何找 GC Root（应该用 Dominator Tree）
2. 第三步结论过于具体（仅针对该案例），不具一般性
3. 未提及"Leak Suspects Report"这一快速入门方法

**建议修正**:
```
第一步：打开 HPROF 文件并找 GC Root
  └─ 推荐: 左侧菜单选 "Dominator Tree" → 按内存 $size 降序排列
  └─ 快速入门: 可先查看 "Leak Suspects Report" 自动分析结果

第二步：如果 GC Root 是线程对象
  └─ 右键选择 "List Objects" → 查看具体是哪个线程
  └─ 展开该线程的局部变量和字段

第三步：沿引用链向下追踪
  └─ 观察哪个对象占用内存最多
  └─ 常见泄漏源: 未关闭的数据库连接、缓存集合、事件监听器

第四步：定位源代码位置
  └─ 双击对象查看其源代码位置
  └─ 修复资源未释放的逻辑
```

---

### 勘误 4: jcmd 命令的表格中 GC.heap_dump 用法不规范

**原文位置**: 第 1868-1875 行

**原文内容**:
```
| jcmd 0 GC.heap_info jcmd 0 GC.class_histogram jcmd 0 GC.heap_dump | jmap -heap jmap -histo jmap -dump | ...
```

**问题**:
1. 表格格式混乱，三个 jcmd 命令挤在一个单元格
2. GC.heap_dump 缺少文件路径参数的示例

**建议修正**:
```
应分成三行:
| jcmd <pid> GC.heap_info                    | jmap -heap <pid>                               |
| jcmd <pid> GC.class_histogram              | jmap -histo <pid>                              |
| jcmd <pid> GC.heap_dump filename=/path/heap.hprof | jmap -dump:format=b,file=/path/heap.hprof <pid> |
```

---

### 勘误 5: Native Memory 诊断功能文档缺失

**原文位置**: 第 1882 行

**原文内容**:
```
| jcmd 0 VM.native_memory | 查看jvm native内存分配。 |
```

**问题**:
未说明需要启用 NMT（Native Memory Tracking），用户直接执行会报错。

**建议修正**:
```
查看 JVM Native 内存分配（需在启动参数中启用 NMT）:
  启动参数: -XX:NativeMemoryTracking=detail
  查询命令: jcmd <pid> VM.native_memory summary
  
说明: 会列出 heap/thread/code/symbol 等各部分的内存使用
常用场景: 排查是否存在 native 内存泄漏(如 buffer 池、c++ 对象)
```

---

## 速查表

### 快速命令参考

#### 线程问题诊断

| 问题类型 | 命令 | 输出关键词 |
|---------|------|----------|
| **线程死锁** | `jstack <pid>` | Found one Java-level deadlock |
| **线程阻塞** | `jstack <pid>` | BLOCKED, "locked" |
| **线程等待** | `jstack <pid>` | WAITING, Object.wait() |
| **高 CPU 线程** | `jstack <pid>` + top/ps | RUNNABLE 线程对应 tid |

#### 内存问题诊断

| 问题类型 | 命令 | 后续步骤 |
|---------|------|---------|
| **对象分布** | `jmap -histo:live <pid>` | 查看前 10 个大对象 |
| **堆内存概览** | `jmap -heap <pid>` | 检查堆大小配置是否合理 |
| **内存泄漏** | `jmap -dump:live,format=b,file=heap.hprof <pid>` | 用 MAT 分析 HPROF |
| **OOM 自动捕获** | 启动参数: `-XX:+HeapDumpOnOutOfMemoryError` | 检查 dump 文件位置 |

#### GC 问题诊断

| 问题类型 | 命令 | 判断标准 |
|---------|------|---------|
| **GC 频繁** | `jstat -gc <pid> 1000 60` | YGC > 50/min or YGCT > 10% |
| **GC 长暂停** | `jstat -gccapacity <pid>` | FGCT > 1000ms (每次 Full GC) |
| **Old 区溢出** | `jstat -gcutil <pid>` | O > 80% and 持续增长 |
| **类卸载异常** | `jstat -class <pid>` | Unloaded 数量异常多 |

#### JVM 配置查询

| 需求 | 命令 |
|------|------|
| 查看所有 JVM 参数 | `jinfo <pid>` 或 `jcmd <pid> VM.flags` |
| 查看系统属性 | `jinfo -sysprops <pid>` 或 `jcmd <pid> VM.system_properties` |
| 动态调整参数 | `jinfo -flag <flag>=<value> <pid>` (限制条件多) |
| 查看命令行 | `jcmd <pid> VM.command_line` |

### 常用命令组合

#### 场景 1: 应用假死，快速定位线程

```bash
# Step 1: 获取进程 ID
jps -l

# Step 2: 查看所有线程状态
jstack 12345 | grep -A 5 "BLOCKED\|WAITING"

# Step 3: 检查是否存在死锁
jstack 12345 | grep -i "deadlock"
```

#### 场景 2: 内存持续增长，定位泄漏源

```bash
# Step 1: 查看对象分布
jmap -histo:live 12345 | head -20

# Step 2: 生成 dump 文件(注意流量摘除)
jmap -dump:live,format=b,file=heap.hprof 12345

# Step 3: 本地用 MAT 打开分析
# (使用 MAT 的 Dominator Tree 和 Path to GC Root 功能)
```

#### 场景 3: GC 停顿频繁，优化垃圾回收

```bash
# Step 1: 实时监控 GC
jstat -gc 12345 1000 30

# Step 2: 查看堆容量配置
jmap -heap 12345

# Step 3: 根据结果调整启动参数
# 如增大 Eden 区、切换 GC 器等
```

#### 场景 4: 线程池积压，检查任务堆积

```bash
# Step 1: 获取线程栈
jstack 12345 > dump.txt

# Step 2: 统计线程状态分布
grep "java.lang.Thread.State" dump.txt | sort | uniq -c

# Step 3: 查看阻塞线程的堆栈
grep -A 3 "java.util.concurrent.BlockingQueue" dump.txt
```

### jcmd 快速参考 (JDK 7+)

```bash
# 列举所有 Java 进程
jcmd

# 查看单个进程的所有可用命令
jcmd 12345 help

# 查看线程栈 (替代 jstack)
jcmd 12345 Thread.print

# 查看堆信息 (替代 jmap -heap)
jcmd 12345 GC.heap_info

# 生成 heap dump (替代 jmap -dump)
jcmd 12345 GC.heap_dump filename=/tmp/heap.hprof

# 查看对象分布 (替代 jmap -histo)
jcmd 12345 GC.class_histogram

# 查看 JVM 参数
jcmd 12345 VM.flags

# 查看系统属性
jcmd 12345 VM.system_properties

# 在所有进程上执行命令
jcmd 0 Thread.print
jcmd 0 GC.run              # 触发 Full GC
jcmd 0 VM.native_memory    # 查看 Native 内存(需启用 NMT)
```

### MAT 常用操作速查

| 操作 | 菜单路径 | 用途 |
|------|---------|------|
| 查看泄漏嫌疑 | 右上角 Reports → Leak Suspects | 快速定位可能的泄漏对象 |
| Dominator Tree | 左侧菜单 → Dominator Tree | 按内存占用大小排序，找最大对象 |
| 查看 GC Roots | 右键对象 → GC Roots → with incoming references | 了解对象被谁持有 |
| 查看引用链 | 右键对象 → Path to GC Roots | 追踪完整的引用链路 |
| 查看线程栈 | 展开线程对象 → 查看 Local Variables | 追踪线程局部变量 |
| 导出结果 | File → Export | 导出分析结果为 CSV/TXT |

---

## 总结

JDK 诊断工具体系完整、功能强大，是 Java 故障排查的核心武器：

1. **初级阶段**: 掌握 jps、jstack、jstat、jmap 的基本用法
2. **进阶阶段**: 学会组合使用，快速定位线程死锁、内存泄漏、GC 异常
3. **高级阶段**: 熟练使用 MAT、jcmd 的高级功能，理解 JVM 内部机制
4. **生产实践**: 建立监控告警、自动 dump、离线分析的完整工作流

掌握这些工具后，大多数 Java 运行时问题都能快速诊断和修复。

---

**文档更新时间**: 2026-05-24  
**适用 JDK 版本**: Java 7 及以上  
**难度等级**: 中级 ★★★

