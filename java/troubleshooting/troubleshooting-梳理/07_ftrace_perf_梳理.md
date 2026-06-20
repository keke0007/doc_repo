# 07. 内核级诊断工具 - ftrace、perf 梳理

**文档说明**：本文档基于 `ppt-merged.md` 第 1231-1400 行内容梳理，按知识点分类组织，配置 ASCII 流程图和速查表。

---

## 一、Linux 追踪机制综述

### 1.1 观测事件源类型

Linux 内置了多种事件源，用于追踪系统及软件的执行情况：

| 事件源 | 类型 | 描述 |
|------|------|------|
| **tracepoint** | 静态 | Linux 在内核关键函数上硬编码的静态追踪点，用于观测系统运行情况 |
| **kprobe** | 动态 | 内核态动态追踪机制，可追踪任意内核函数，无需事先埋点 |
| **uprobe** | 动态 | 用户态动态追踪机制，可追踪任意用户态程序函数 |
| **usdt** | 静态 | 用户级静态追踪，用户空间版本的 tracepoint，需应用程序加入探针（Java、MySQL、libc 等已内置） |
| **PMC 硬件事件** | 硬件 | CPU 的性能监控计数器产生，包含各种硬件性能指标 |
| **软件事件** | 内核 | 与硬件事件相关，由内核运行机制触发（缺页中断、上下文切换等） |

### 1.2 追踪机制（内核层）

基于上述事件源，Linux 开发了以下追踪机制：

| 追踪机制 | 简介 | 用途 |
|---------|------|------|
| **ftrace** | Linux 早期 tracepoint 函数追踪机制，后扩展支持 kprobe、uprobe；通过 tracefs 向用户空间提供接口 | 函数追踪、事件追踪 |
| **perf_events** | 基于事件的性能分析机制，通过 perf_event_open 系统调用对用户空间开放 | CPU 采样、硬件事件分析 |
| **eBPF** | 在 BPF 基础上扩展的事件追踪机制，通过在内核虚拟机上执行自定义字节码实现 | 高级诊断、自定义观测 |

### 1.3 诊断工具（前端工具链）

基于内核追踪机制演化的诊断工具：

| 工具 | 底层机制 | 描述 |
|------|---------|------|
| **trace-cmd** | ftrace | 对 ftrace 操作的封装，简化 tracefs 目录的手动操作 |
| **perf-tools** | ftrace、perf_events | 对 ftrace 和部分 perf 功能的场景化封装 |
| **perf** | perf_events、ftrace、eBPF | Linux 主推工具，支持 CPU 采样、硬件事件、函数追踪 |
| **bcc** | eBPF | 各种场景诊断工具（syscount、biolatency 等） |
| **bpftrace** | eBPF | 提供 AWK 类脚本语法，快速自定义 eBPF 工具 |

---

## 二、ftrace 使用详解

### 2.1 tracefs 文件系统

#### 挂载与目录结构

ftrace 基于 tracefs 文件系统，位于 `/sys/kernel/debug/tracing` 目录：

```
/sys/kernel/debug/tracing/
├── trace                    # 当前追踪输出（只读）
├── trace_pipe              # 流式输出追踪数据
├── available_tracers       # 可用的追踪器列表
├── current_tracer          # 当前激活的追踪器
├── events/                 # 事件目录
│   ├── tracepoints/        # 静态追踪点
│   ├── kprobes/            # 动态 kprobe
│   └── uprobes/            # 动态 uprobe
├── available_events        # 所有可用事件
├── set_event               # 设置启用的事件
├── buffer_size_kb          # 缓冲区大小
├── tracing_on              # 启用/禁用追踪
└── set_ftrace_filter       # 设置函数过滤
```

### 2.2 ftrace 函数追踪示例

**以追踪 do_sys_open 函数为例**：

```bash
# 1. 启用 function 追踪器
echo function > /sys/kernel/debug/tracing/current_tracer

# 2. 设置函数过滤（仅追踪 do_sys_open）
echo do_sys_open > /sys/kernel/debug/tracing/set_ftrace_filter

# 3. 启用追踪
echo 1 > /sys/kernel/debug/tracing/tracing_on

# 4. 运行被追踪程序或等待事件

# 5. 查看输出
cat /sys/kernel/debug/tracing/trace
# 或实时观看
cat /sys/kernel/debug/tracing/trace_pipe

# 6. 关闭追踪
echo 0 > /sys/kernel/debug/tracing/tracing_on
```

### 2.3 ftrace kprobe 追踪示例

**以追踪 do_sys_openat2 函数为例**：

```bash
# 1. 添加 kprobe（动态探针）
echo 'p:my_openat2 do_sys_openat2' > /sys/kernel/debug/tracing/kprobe_events

# 2. 启用该事件
echo 1 > /sys/kernel/debug/tracing/events/kprobes/my_openat2/enable

# 3. 启用追踪
echo 1 > /sys/kernel/debug/tracing/tracing_on

# 4. 查看输出
cat /sys/kernel/debug/tracing/trace

# 5. 清理（关闭追踪后）
echo 0 > /sys/kernel/debug/tracing/tracing_on
echo '-:my_openat2' > /sys/kernel/debug/tracing/kprobe_events
```

### 2.4 ftrace 特点与局限

**优点**：
- 原生内核支持，无需额外依赖
- 支持 tracepoint、kprobe、uprobe 等多种事件源
- 开销相对较小

**缺点**：
- 操作繁琐，需要手动修改 tracefs 文件
- 追踪功能固定，自定义能力不足
- 只能提供调用时间、进程、函数名、参数等固定观测形式

---

## 三、trace-cmd 工具

### 3.1 简介

trace-cmd 是对 ftrace 的命令行封装，极大简化了 ftrace 的使用流程，无需手动操作 tracefs 文件系统。

### 3.2 函数追踪

```bash
# 追踪 do_sys_open 函数
trace-cmd record -p function -l do_sys_open [command]

# 查看追踪结果
trace-cmd report

# 保存到文件
trace-cmd report > output.txt
```

### 3.3 tracepoint 追踪

```bash
# 列出所有可用的 tracepoint
trace-cmd list -e

# 追踪特定事件（例如系统调用）
trace-cmd record -e syscalls:sys_enter_open [command]

# 或追踪多个事件
trace-cmd record -e syscalls:sys_enter_open -e syscalls:sys_exit_open [command]

# 查看追踪结果
trace-cmd report
```

### 3.4 trace-cmd 优势

- 自动管理 tracefs，无需手动操作
- 支持多种追踪类型（函数、tracepoint、kprobe 等）
- 输出格式统一，易于解析

---

## 四、perf-tools 工具集

### 4.1 简介

perf-tools 是基于 ftrace 开发的场景化诊断工具集，提供了针对常见问题的预制解决方案。

### 4.2 常用工具

| 工具 | 功能 |
|------|------|
| **opensnoop** | 追踪文件打开的调用，显示进程名、PID、打开的文件名等 |
| **syscount** | 统计系统调用次数，按系统调用类型或进程分组 |
| 其他工具 | 可在 GitHub 页面查看完整工具列表 |

### 4.3 使用示例

```bash
# 使用 opensnoop 追踪文件打开
opensnoop -d 10          # 持续 10 秒

# 使用 syscount 统计系统调用
syscount -i 1 -d 5       # 每秒显示一次，共 5 次
```

### 4.4 优势

- 预制解决方案，无需从零开始编写追踪规则
- 命令简洁，适合快速诊断
- 基于成熟的 ftrace 机制

---

## 五、perf 命令详解

### 5.1 简介

perf 是 Linux 主推的性能分析工具，提供了 CPU 采样分析、PMC 硬件性能分析等功能，同时也支持 ftrace 和少量 eBPF 支持。

### 5.2 perf record - 采样分析

```bash
# 基础用法：采样指定程序
perf record [command]

# 采样 30 秒
perf record -d 30 [command]

# 采样特定事件（CPU cycle）
perf record -e cycles [command]

# 采样多个事件
perf record -e cycles,instructions [command]

# 采样频率设置
perf record -F 1000 [command]    # 每秒 1000 次采样

# 追踪特定 CPU
perf record -C 0-3 [command]     # 仅追踪 CPU 0-3

# 采样数据保存在 perf.data
```

### 5.3 perf report - 查看性能数据

```bash
# 基础用法
perf report

# 文本模式输出
perf report --stdio

# 输出示例：
# Samples: 1234 of event 'cycles'
# Event count (approx.): 1234000000
# Overhead  Command    Shared Object       Function
# ========  =========  ==================  ========================
#    30.45%  myapp     myapp               [.]  function_a
#    20.30%  myapp     libc-2.29.so        [.]  malloc
#    10.15%  myapp     myapp               [.]  function_b
```

### 5.4 perf script - 文本化输出

```bash
# 输出原始追踪数据为文本格式
perf script

# 输出示例：
# myapp  1234 [000] 12345.123456: cycles:
#     ffffffff81234567 [kernel.kallsyms] function_a
#     ffffffff81234568 [kernel.kallsyms] function_b
#     7ffff7234567  /lib64/libc-2.29.so malloc
```

**用途**：便于其他工具（如火焰图脚本）处理原始数据。

### 5.5 perf list - 查询可用事件

```bash
# 列出所有可用事件
perf list

# 按类型筛选
perf list hw              # 硬件事件
perf list sw              # 软件事件
perf list tracepoint      # tracepoint 事件

# 搜索特定事件
perf list | grep syscall
```

### 5.6 perf stat - 事件统计

```bash
# 统计程序执行期间的各种事件
perf stat [command]

# 统计输出示例：
# Performance counter stats for 'command':
#
#        1,234.456789 task-clock                #    0.999 CPUs utilized
#                234 context-switches          #    0.189 K/sec
#                 12 cpu-migrations            #    0.010 K/sec
#                 45 page-faults               #    0.036 K/sec
#         1,234,567,890 cycles                 #    1.000 GHz
#           567,890,123 instructions           #    0.46  insn per cycle
```

### 5.7 perf trace - 系统调用追踪

```bash
# 追踪程序的系统调用
perf trace [command]

# 追踪特定系统调用
perf trace -e open,close [command]

# 输出示例：
#     0.000 ( 0.015 ms): test/1234 open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
#     0.020 ( 0.010 ms): test/1234 fstat(3, { ... }) = 0
```

### 5.8 perf ftrace - 函数追踪封装

```bash
# 追踪特定函数
perf ftrace -F do_sys_open [command]

# 类似于原生 ftrace 的函数追踪，但操作更简洁
```

### 5.9 动态/静态追踪支持

```bash
# 追踪 tracepoint 事件
perf record -e syscalls:sys_enter_open [command]

# 追踪 kprobe（动态探针）
perf record -e 'kprobes:my_probe' [command]

# 追踪 uprobe（用户态动态探针）
perf record -e 'uprobes:/bin/bash:main' [command]
```

### 5.10 perf 相比 ftrace 的优势

| 特性 | ftrace | perf |
|------|--------|------|
| CPU 采样 | ❌ | ✅ |
| 硬件事件（PMC） | ❌ | ✅ |
| 事件统计 | ❌ | ✅ |
| 函数追踪 | ✅ | ✅ |
| Tracepoint | ✅ | ✅ |
| 动态追踪 | ✅ | ✅ |
| 命令易用性 | ❌ | ✅ |

---

## 六、perf 火焰图

### 6.1 简介

火焰图是性能优化大师 Brendan Gregg 发明的可视化工具，将采样数据以图形方式展示，便于识别性能瓶颈。

### 6.2 火焰图原理

- 将多次采集的相同线程栈聚合在一起显示
- **图中越宽的栈**表示该栈在运行过程中被抓取到的次数越多
- **越宽 = CPU 时间占比越高**

### 6.3 生成火焰图

```bash
# 1. 下载火焰图工具
git clone https://github.com/brendangregg/FlameGraph.git
cd FlameGraph

# 2. 采样数据
perf record -F 99 -g [command]    # -g: 记录调用栈

# 3. 转换为火焰图格式
perf script > out.perf
./stackcollapse-perf.pl out.perf > out.folded

# 4. 生成 SVG 文件
./flamegraph.pl out.folded > out.svg

# 5. 用浏览器打开 out.svg
```

### 6.4 火焰图解读

- **横轴**：样本数多少（宽度表示 CPU 占用时间比例）
- **纵轴**：调用栈深度
- **颜色**：无特殊含义，仅用于区分不同函数
- **点击**：可交互探索调用栈

### 6.5 火焰图优势

- 直观展示 CPU 时间分布
- 快速定位性能瓶颈
- 支持交互式探索

---

## 七、知识体系总结

### 7.1 观测数据流

```
┌─────────────────────────────────────────────────────────────┐
│                      Linux 观测体系                           │
└─────────────────────────────────────────────────────────────┘

【事件源层】
  ├─ 静态源: tracepoint、usdt
  ├─ 动态源: kprobe、uprobe
  ├─ 硬件源: PMC 硬件计数器
  └─ 内核源: 软件事件（缺页、上下文切换等）

         ↓

【追踪机制层】
  ├─ ftrace      (函数/事件追踪)
  ├─ perf_events (采样/事件分析)
  └─ eBPF        (可编程观测)

         ↓

【工具前端层】
  ├─ trace-cmd       (ftrace 封装)
  ├─ perf-tools      (ftrace 场景工具)
  ├─ perf            (主推性能工具)
  ├─ bcc             (eBPF 诊断工具)
  └─ bpftrace        (eBPF 脚本工具)

         ↓

【输出与分析】
  ├─ 实时追踪输出
  ├─ 性能报告（perf report）
  ├─ 文本化数据（perf script）
  └─ 火焰图可视化
```

### 7.2 工具选择决策树

```
                   ┌─────────────────────────┐
                   │   我想做什么？            │
                   └────────────┬────────────┘
                                │
               ┌────────────────┼────────────────┐
               │                │                │
        ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
        │ 追踪函数调用 │  │ 采样性能分析 │  │ 统计事件    │
        │ 或事件       │  │ 或 CPU 采样  │  │ 或系统调用  │
        └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
               │                │                │
       ┌───────▼────────┐  ┌────▼────────┐  ┌────▼─────────┐
       │ • ftrace       │  │ • perf      │  │ • perf stat  │
       │   (原生)       │  │   record    │  │ • perf trace │
       │ • trace-cmd    │  │ • perf      │  │ • syscount   │
       │ • perf ftrace  │  │   report    │  │ (perf-tools) │
       │ • perf trace   │  │ • 火焰图    │  │              │
       │   (syscalls)   │  │              │  │              │
       └────────────────┘  └───────────────┘  └──────────────┘
```

### 7.3 perf 工作流

```
【perf record 采样阶段】
    ↓
  指定事件类型、采样频率、采样时长
    ↓
  生成 perf.data 采样文件
    ↓
    ├─→ 【perf report】         (交互式查看)
    ├─→ 【perf script】         (导出文本)
    │     ↓
    │   stackcollapse-perf.pl   (栈聚合)
    │     ↓
    │   flamegraph.pl           (火焰图生成)
    │     ↓
    │   out.svg                 (浏览器查看)
    │
    └─→ 【perf stat】           (事件统计)
```

---

## 八、核心概念对比

### 8.1 ftrace vs perf

| 维度 | ftrace | perf |
|------|--------|------|
| 追踪能力 | 函数、事件 | 函数、事件、采样、硬件 |
| 易用性 | 低（需操作 tracefs） | 高（命令行工具） |
| 采样分析 | ❌ | ✅ |
| 硬件事件 | ❌ | ✅ |
| 自定义能力 | 低 | 中等 |
| 学习曲线 | 陡峭 | 平缓 |

### 8.2 trace-cmd vs perf-tools vs perf

| 工具 | 底层 | 场景 | 学习成本 |
|------|------|------|--------|
| trace-cmd | ftrace | 函数/事件追踪 | 低 |
| perf-tools | ftrace | 特定场景诊断 | 低 |
| perf | perf_events | 全面性能分析 | 中 |

### 8.3 追踪方式对比

| 追踪方式 | 开销 | 精度 | 应用场景 |
|---------|------|------|---------|
| tracepoint 静态 | 低 | 高 | 内核关键路径 |
| kprobe 动态 | 中 | 高 | 任意内核函数 |
| uprobe 动态 | 中 | 高 | 用户态程序 |
| perf 采样 | 低 | 中 | 全局性能分析 |

---

## 九、常见使用场景速查表

### 9.1 函数追踪

```bash
# 方案 1: ftrace (原生)
echo function > /sys/kernel/debug/tracing/current_tracer
echo my_function > /sys/kernel/debug/tracing/set_ftrace_filter
echo 1 > /sys/kernel/debug/tracing/tracing_on
# 应用运行...
cat /sys/kernel/debug/tracing/trace

# 方案 2: trace-cmd (推荐)
trace-cmd record -p function -l my_function myapp
trace-cmd report

# 方案 3: perf ftrace
perf ftrace -F my_function myapp
```

### 9.2 系统调用追踪

```bash
# 方案 1: perf trace (推荐)
perf trace -e open,close myapp

# 方案 2: trace-cmd
trace-cmd record -e syscalls:sys_enter_open myapp
trace-cmd report

# 方案 3: perf-tools opensnoop
opensnoop -d 10
```

### 9.3 CPU 热点分析

```bash
# 基础采样
perf record -F 99 -g myapp
perf report

# 生成火焰图
perf script | ./stackcollapse-perf.pl | ./flamegraph.pl > out.svg

# 快速统计
perf stat myapp
```

### 9.4 特定事件分析

```bash
# 硬件事件统计
perf stat -e cycles,instructions,cache-references,cache-misses myapp

# 采样特定事件
perf record -e cycles:u myapp    # 用户态 CPU cycle
perf record -e branch-misses myapp

# 查看可用事件
perf list
```

### 9.5 性能诊断工作流

```bash
# 1. 快速定位热点
perf record -F 99 -g -d 30 myapp
perf report --stdio

# 2. 深入函数分析
trace-cmd record -p function -l hot_function myapp
trace-cmd report

# 3. 事件级统计
perf stat -e cycles,cache-misses,branch-misses myapp

# 4. 生成可视化
perf script > out.perf
./stackcollapse-perf.pl out.perf > out.folded
./flamegraph.pl out.folded > out.svg
```

---

## 十、原文勘误与注记

### 10.1 勘误项

| 行号 | 原文 | 勘误 | 说明 |
|------|------|------|------|
| 1246 | "usdt：用户级静态追踪，是用户空间版本的 **traceport**" | 应为 "**tracepoint**" | 单词拼写错误 |
| 1268 | trace-cmd 的观测事件源列 | 应补充 "tracepoint kprobe uprobe pmc" 的完整列表 | 表格信息不完整 |
| 1269 | perf-tools 的观测事件源列 | 应列出 "ftrace、tracepoint、kprobe、uprobe" 等 | 表格信息缺失 |
| 1270 | perf 的观测事件源列 | 应列出 "PMC、硬件事件、tracepoint、kprobe、uprobe" 等 | 表格信息缺失 |

### 10.2 补充注记

- **Linux 演化原因**：Linux 有多个动态追踪机制（ftrace、perf_events、eBPF），是因为 Linux 处于持续快速演化阶段，不同时期的设计选择导致了机制多样化。
  
- **ftrace vs eBPF**：
  - ftrace 是早期设计，追踪功能固定，自定义能力不足
  - eBPF 是现代方案，允许自定义 BPF 字节码，灵活性强但学习曲线陡峭

- **perf 的优势**：perf 不仅是 ftrace 工具的包装，还额外提供了 CPU 采样和事件统计能力，是 Linux 社区主推的现代性能分析工具。

- **火焰图使用建议**：
  - 需要单独安装 FlameGraph 工具集
  - 采样时务必使用 `-g` 标志记录完整调用栈
  - 适合在生产环境识别全局性能瓶颈

---

## 十一、ASCII 流程图详解

### 11.1 事件源→追踪机制→工具链关系

```
┌────────────────────────────────────────────────────────────────┐
│              Linux 内核级诊断完整体系                            │
└────────────────────────────────────────────────────────────────┘

【第1层：观测事件源】(提供原始数据)
    
    ┌──────────────┐
    │  静态事件源   │
    ├──────────────┤
    │• tracepoint  │
    │• USDT        │
    └──────────────┘
           │
    ┌──────────────┐       ┌──────────────┐       ┌──────────────┐
    │  动态事件源   │       │  硬件事件源   │       │  内核事件源   │
    ├──────────────┤       ├──────────────┤       ├──────────────┤
    │• kprobe      │       │• PMC计数器   │       │• page-fault  │
    │• uprobe      │       │• cycles      │       │• context-sw  │
    │              │       │• instructions│       │• ...         │
    └──────────────┘       └──────────────┘       └──────────────┘
           │                      │                       │
           └──────────────────────┼───────────────────────┘
                                  │
                                  ↓
┌────────────────────────────────────────────────────────────────┐
│【第2层：内核追踪机制】(收集、汇聚、过滤数据)                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌───────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │    ftrace     │    │ perf_events  │    │    eBPF      │   │
│  │  (tracefs)    │    │ (系统调用)    │    │  (虚拟机)    │   │
│  │               │    │              │    │              │   │
│  │ 函数/事件追踪 │    │ 采样/统计分析 │    │ 可编程观测   │   │
│  └───────────────┘    └──────────────┘    └──────────────┘   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
           │                      │                       │
           └──────────────────────┼───────────────────────┘
                                  │
                                  ↓
┌────────────────────────────────────────────────────────────────┐
│【第3层：用户空间前端工具】(命令行或脚本接口)                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │  trace-cmd   │  │ perf-tools   │  │      perf        │   │
│  │  (ftrace包装)│  │ (ftrace场景) │  │   (综合性能工具)  │   │
│  └──────────────┘  └──────────────┘  └──────────────────┘   │
│                                                                │
│  ┌──────────────┐  ┌──────────────┐                          │
│  │     bcc      │  │  bpftrace    │                          │
│  │ (eBPF诊断)   │  │ (eBPF脚本)   │                          │
│  └──────────────┘  └──────────────┘                          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
           │                      │                       │
           └──────────────────────┼───────────────────────┘
                                  │
                                  ↓
┌────────────────────────────────────────────────────────────────┐
│【第4层：输出与分析】(可视化与解释)                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ├─ 实时输出 (trace_pipe / trace-cmd report)                  │
│  ├─ 文本报告 (perf report --stdio)                            │
│  ├─ 原始数据 (perf script)                                    │
│  └─ 可视化   (火焰图 flamegraph.svg)                          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 11.2 perf 完整工作流

```
┌─────────────────────────────────────────────────────────────────┐
│                    perf 采样分析完整流程                          │
└─────────────────────────────────────────────────────────────────┘

【第1阶段：采样配置】
    ↓
  $ perf record -F 99 -g -d 30 myapp
    │
    ├─ -F 99       : 采样频率 99 Hz（每秒99次）
    ├─ -g          : 记录完整调用栈 (callchain)
    ├─ -d 30       : 采样时间 30 秒
    └─ myapp       : 目标应用程序

    ↓
【采样执行】(内核周期性中断记录 CPU 栈)
    ↓
  perf.data (采样数据库)
    │
    ├─ 样本数: 1000+ samples
    ├─ 事件类型: cycles (默认)
    ├─ 调用栈: 完整 call chain
    └─ 时间戳: 精确时间信息


【第2阶段：数据分析】
    │
    ├──────────────────────────────────────┐
    │                                      │
    ↓                                      ↓
【交互式查看】                         【导出原始数据】
$ perf report                          $ perf script > out.perf
    │                                      │
    ├─ 图表展示                            ├─ 文本行格式
    ├─ 按命令/库/函数分组                  ├─ 便于脚本处理
    ├─ 支持交互式探索                      └─ 每行一个样本
    └─ 计算百分比占有率
                                           ↓
                                    【转换为火焰图格式】
                                    $ ./stackcollapse-perf.pl \
                                        out.perf > out.folded
                                           │
                                    ├─ 聚合相同栈
                                    ├─ 计算栈出现次数
                                    └─ 生成 folded 格式


【第3阶段：可视化】
    ↓
  $ ./flamegraph.pl out.folded > out.svg
    │
    ├─ SVG 矢量图
    ├─ 交互式 (点击放大)
    └─ 浏览器查看

    ↓
【火焰图解读】
    ├─ 横轴宽度   → 调用次数/时间占比
    ├─ 纵轴高度   → 调用栈深度
    ├─ 颜色       → 函数区分 (无特殊含义)
    └─ 热点区域   → 需要优化的函数


【补充：事件统计】
$ perf stat myapp
    │
    ├─ task-clock      : CPU 时间
    ├─ context-switches: 上下文切换
    ├─ cycles          : CPU 周期
    ├─ instructions    : 指令数
    ├─ L1-dcache-loads : L1 缓存读
    └─ ...其他硬件指标

```

### 11.3 ftrace 手动操作流程

```
┌─────────────────────────────────────────────────────────────────┐
│              ftrace 基于 tracefs 的使用流程                       │
└─────────────────────────────────────────────────────────────────┘

【初始化】
    ↓
$ mount -t tracefs tracefs /sys/kernel/debug/tracing/
    ↓
/sys/kernel/debug/tracing/ 目录准备完毕


【配置1：选择追踪器】
    ↓
$ echo function > /sys/kernel/debug/tracing/current_tracer
    │
    ├─ 可选值: function / kprobe / event / ...
    └─ 查看可用: cat available_tracers


【配置2：设置过滤条件】(可选)
    ↓
$ echo my_function > /sys/kernel/debug/tracing/set_ftrace_filter
    │
    ├─ 支持通配符: echo "sys_*" > set_ftrace_filter
    └─ 查看可用函数: cat available_filter_functions


【配置3：设置缓冲区大小】(可选)
    ↓
$ echo 10000 > /sys/kernel/debug/tracing/buffer_size_kb
    │
    └─ 大缓冲 = 更多样本 + 更高开销


【启用追踪】
    ↓
$ echo 1 > /sys/kernel/debug/tracing/tracing_on


【等待/运行被追踪程序】
    ↓
$ ./myapp          # 或等待自然事件发生
    ↓
[追踪进行中...]


【查看输出（实时）】
    ↓
$ cat /sys/kernel/debug/tracing/trace_pipe
    │
    ├─ 流式输出，持续追踪
    ├─ 阻塞模式
    └─ Ctrl+C 停止


【查看输出（文件）】
    ↓
$ cat /sys/kernel/debug/tracing/trace
    │
    ├─ 查看已缓冲数据
    ├─ 格式化输出
    └─ 示例:
    │   # tracer: function
    │   # TASK-PID  CPU#  ||||  TIMESTAMP  FUNCTION
    │   myapp-1234 [000]  d...   123.456: sys_open
    │   myapp-1234 [000]  d...   123.457:  do_sys_open
    │   myapp-1234 [000]  d...   123.458:  do_sys_openat2


【禁用追踪】
    ↓
$ echo 0 > /sys/kernel/debug/tracing/tracing_on
    ↓
[追踪停止，数据保留]


【清空缓冲区】
    ↓
$ echo > /sys/kernel/debug/tracing/trace
    ↓
[缓冲区清空，可重新追踪]

```

### 11.4 trace-cmd 简化工作流

```
┌─────────────────────────────────────────────────────────────────┐
│         trace-cmd 自动化 ftrace 管理流程 (推荐)                   │
└─────────────────────────────────────────────────────────────────┘

【一条命令：完整追踪】
    ↓
$ trace-cmd record -p function -l my_function myapp
    │
    ├─ -p function     : 选择 function 追踪器
    ├─ -l my_function  : 设置函数过滤
    └─ myapp           : 运行应用

    ↓
【内部自动处理】
    ├─ 挂载 tracefs (如需要)
    ├─ 配置 current_tracer
    ├─ 设置 set_ftrace_filter
    ├─ 启用 tracing_on
    ├─ 监听 trace_pipe
    ├─ 禁用 tracing_on
    └─ 保存数据到 trace.dat

    ↓
【输出查看】
    ↓
$ trace-cmd report
    │
    ├─ 自动格式化显示
    ├─ 同 ftrace trace 的输出
    └─ 支持导出


【对比：手动 ftrace (繁琐)】
    ├─ 需要 5+ 个 echo 命令
    ├─ 手动挂载 tracefs
    ├─ 手动读写 tracefs 文件
    ├─ 手动清理
    └─ 总步骤数: ~15 步

【trace-cmd (简洁)】
    ├─ 1 条 record 命令
    ├─ 1 条 report 命令
    └─ 总步骤数: 2 步
```

---

## 十二、速查表

### 12.1 命令速查

| 任务 | 命令 | 说明 |
|------|------|------|
| **查看可用事件** | `perf list` | 显示所有可用追踪事件 |
| **采样分析** | `perf record -F 99 -g myapp` | 采样频率99Hz，记录调用栈 |
| **查看报告** | `perf report` | 交互式查看采样报告 |
| **导出数据** | `perf script > out.perf` | 导出为文本供火焰图处理 |
| **事件统计** | `perf stat myapp` | 统计程序执行期间的事件 |
| **系统调用追踪** | `perf trace -e open,close myapp` | 追踪特定系统调用 |
| **函数追踪(简化)** | `trace-cmd record -p function -l func myapp` | trace-cmd 函数追踪 |
| **事件追踪(简化)** | `trace-cmd record -e syscalls:sys_enter_open myapp` | trace-cmd 事件追踪 |
| **快速诊断** | `opensnoop -d 10` | perf-tools: 追踪文件打开 |
| **系统调用统计** | `syscount -i 1 -d 5` | perf-tools: 统计系统调用 |
| **生成火焰图** | 见下表 | 多步骤工作流 |

### 12.2 火焰图生成工作流

```bash
# 1. 采样数据
perf record -F 99 -g -d 30 myapp

# 2. 导出为文本
perf script > out.perf

# 3. 栈聚合 (需下载 FlameGraph)
cd FlameGraph
./stackcollapse-perf.pl ../out.perf > out.folded

# 4. 生成 SVG
./flamegraph.pl out.folded > out.svg

# 5. 浏览器查看
firefox out.svg
```

### 12.3 常用 perf 选项

| 选项 | 含义 | 示例 |
|------|------|------|
| `-F` | 采样频率 (Hz) | `perf record -F 99` |
| `-g` | 记录调用栈 | `perf record -g` |
| `-e` | 指定事件 | `perf record -e cycles,instructions` |
| `-d` | 采样时长 (秒) | `perf record -d 30` |
| `-C` | 指定 CPU | `perf record -C 0-3` |
| `-p` | 指定进程 PID | `perf record -p 1234` |
| `--stdio` | 文本输出 | `perf report --stdio` |
| `-i` | 输入数据文件 | `perf report -i custom.data` |

### 12.4 常见问题排查

| 问题 | 原因 | 解决 |
|------|------|------|
| "权限不足" | 非 root 用户 | `sudo perf record ...` 或配置 cap |
| "没有找到事件" | 硬件/内核不支持 | `perf list` 检查可用事件 |
| "采样点过少" | 采样频率太低或时间太短 | 增加 `-F` 值或 `-d` 时间 |
| "火焰图为空" | 没有记录调用栈 | 添加 `-g` 标志 |
| "缓冲区溢出" | 采样数据过多 | 增大 `buffer_size_kb` |

---

## 十三、关键要点总结

1. **观测体系分层**：事件源 → 追踪机制 → 工具前端 → 输出分析

2. **事件源多样性**：
   - 静态源（tracepoint、USDT）：精度高，覆盖有限
   - 动态源（kprobe、uprobe）：灵活，但开销较高
   - 硬件源（PMC）：性能监控，需硬件支持

3. **工具选择原则**：
   - 快速诊断 → `perf stat` 或 `opensnoop`
   - 函数追踪 → `trace-cmd` 或 `perf ftrace`
   - 全面分析 → `perf record` + 火焰图
   - 自定义需求 → eBPF（bcc 或 bpftrace）

4. **perf 相比 ftrace 的优势**：
   - CPU 采样能力
   - 硬件事件统计
   - 命令行界面更友好
   - 输出格式更易处理

5. **火焰图优势**：
   - 直观展示 CPU 时间分布
   - 快速定位热点函数
   - 交互式探索调用栈

6. **自定义能力梯度**：ftrace < perf < eBPF

---

**文档版本**：1.0  
**生成日期**：2026-05-24  
**来源**：ppt-merged.md 第 1231-1400 行内容整理

