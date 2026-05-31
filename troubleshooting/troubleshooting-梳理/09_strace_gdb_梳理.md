# 09. 原生程序调试工具-strace、gdb等 知识点梳理

## 一、strace 命令详解

### 1.1 核心概念

**strace** 是 Linux 中用来观测系统调用(System Call)的工具。

- 系统调用是应用程序访问操作系统功能的唯一接口
- 包括：申请内存、文件 I/O、网络 I/O 等操作
- strace 可以拦截、记录、回放系统调用及其返回值

### 1.2 strace 基础选项

| 选项 | 作用 |
|------|------|
| `-e trace=file` | 只追踪文件相关系统调用 |
| `-e trace=network` | 只追踪网络相关系统调用 |
| `-e trace=process` | 只追踪进程相关系统调用 |
| `-e trace=ipc` | 只追踪 IPC 相关系统调用 |
| `-yy` | 文件描述符(FD)解码，显示 FD 对应的实际文件/socket 等 |
| `-e read=fd` | 打印从文件描述符读取的数据内容 |
| `-e write=fd` | 打印写入文件描述符的数据内容 |
| `-k` | 打印系统调用的调用栈(call stack) |
| `-c` | 统计模式，汇总各系统调用的调用次数、耗时、错误数 |
| `-T` | 显示每个系统调用的耗时(秒为单位) |
| `-o file` | 输出重定向到文件 |

### 1.3 strace 典型用途

#### 用途一：追踪特定类型系统调用

```
# 追踪文件相关系统调用，并解码文件描述符
strace -e trace=file -yy -p <pid>

# 追踪网络相关系统调用
strace -e trace=network -p <pid>

# 打印 read/write 内容
strace -e read=3 -e write=4 -p <pid>
```

#### 用途二：打印调用栈

```
# 显示每个系统调用的调用栈
strace -k -e trace=file -p <pid>
```

#### 用途三：统计系统调用

```
# 统计所有系统调用
strace -c -p <pid>

# 输出格式：系统调用名、次数、总耗时、平均耗时、调用失败次数
```

#### 用途四：学习命令实现

```
# 追踪 free 命令，发现数据来自 /proc/meminfo
strace free

# 追踪 dig 命令，发现 DNS 查询延迟
strace dig example.com
```

### 1.4 案例：调用端追踪 SQL 耗时

**场景**：Java 程序调用数据库慢，需要诊断 SQL 执行耗时

**思路**：
1. Java 通过 `sendto` 系统调用发送 SQL 查询
2. 通过 `recvfrom` 系统调用接收查询结果
3. 计算 `recvfrom` - `sendto` 的时间差 = SQL 耗时

**实现步骤**：

```bash
# 1. 启动 strace 追踪，重定向输出到文件
strace -e trace=network -T -p <java_pid> > /tmp/strace.log 2>&1

# 2. 使用 awk 脚本解析日志
awk '
  /sendto.*<IP:PORT>/ { 
    start_time = $(NF-1)
    next
  }
  /recvfrom.*<IP:PORT>/ { 
    end_time = $(NF-1)
    delay = end_time - start_time
    print "SQL耗时: " delay "秒"
    start_time = 0
  }
' /tmp/strace.log
```

**追踪链路**（请求处理过程）：
1. 接收 HTTP 请求
2. 发送 SQL (sendto 系统调用)
3. 收到查询结果 (recvfrom 系统调用)
4. 返回 HTTP 响应
5. 写 access.log 日志

---

## 二、peekfd 命令

### 2.1 核心功能

**peekfd** 用于读取进程在指定文件描述符(FD)上的读写数据，实时查看数据流内容。

### 2.2 peekfd vs strace 区别

| 工具 | 优点 | 缺点 |
|------|------|------|
| strace | 功能全面，可追踪所有系统调用 | 输出冗长，数据格式难读 |
| peekfd | 输出精简，可读性强 | 功能单一，只能查看 I/O 数据 |

### 2.3 典型用法

```bash
# 查看进程 <pid> 的文件描述符 3 的数据
peekfd -n 100 <pid> 3

# 查看所有文件描述符
peekfd <pid>
```

---

## 三、gdb 命令详解

### 3.1 核心概念

**gdb** 是 C/C++ 语言的命令式调试器(debugger)。

- 支持断点、单步执行、变量监视等调试功能
- 命令式接口允许编写调试脚本自动化问题分析
- 可用于追踪函数调用、检查内存状态、获取线程栈等

### 3.2 gdb 基础命令

| 命令 | 作用 |
|------|------|
| `handle <signal> <action>` | 设置 Linux 信号处理方式 |
| `break <function>` | 在函数处设置断点 |
| `continue` | 继续执行程序，直到遇到断点 |
| `printf` | 格式化打印变量/寄存器值 |
| `shell <cmd>` | 执行 shell 命令 |
| `thread <id>` | 打印或切换线程 |
| `bt` | 打印调用栈(backtrace) |
| `info locals` | 显示本地变量 |
| `info registers` | 显示 CPU 寄存器值 |
| `dump memory <file> <start> <end>` | 堆内存转储 |

### 3.3 C++ 函数名编码(Name Mangling)

C++ 编译器为了兼容 C 的二进制 ABI，会改写(mangle)函数名：

```
原始函数名: G1CollectedHeap::humongous_obj_allocate(unsigned long, unsigned char)
编码后名称: _ZN15G1CollectedHeap22humongous_obj_allocateEmh
            ^^^^^^^ ^^  ^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^^^ ^ ^
            前缀   命名空间长度 类名长度    函数名长度    参数类型
```

**查询编码后的函数名**：
```bash
nm <binary_file> | grep humongous_obj_allocate
```

### 3.4 x86-64 寄存器与函数参数关系

调用约定(Calling Convention)：C++ 实例方法第一个参数是 `this` 指针

| 参数位置 | 寄存器 | 说明 |
|---------|--------|------|
| 第 1 参数(this) | `rdi` | 在 x86-64 中 |
| 第 2 参数 | `rsi` | |
| 第 3 参数 | `rdx` | |
| 第 4 参数 | `rcx` | |
| 返回值 | `rax` | |

### 3.5 案例：定位 Java 大对象分配

**问题**：JVM 申请大对象导致频繁 Full GC，需要定位哪些代码分配大对象

**解决方案**：使用 gdb 脚本在 G1 垃圾回收器的大对象分配函数处设置断点

**gdb 脚本关键步骤**：

#### 第 1 步：设置信号处理

```gdb
# 不处理任何信号（除 SIGINT 用于退出）
handle SIGHUP nostop noprint pass
handle SIGPIPE nostop noprint pass
handle SIGTERM nostop noprint pass
```

**原因**：调试 JVM 时 gdb 会收到各类系统信号，需要让 JVM 自己处理这些信号，gdb 不要干预。

#### 第 2 步：设置断点并运行

```gdb
# 在大对象分配函数处设置断点
break _ZN15G1CollectedHeap22humongous_obj_allocateEmh

# 启动程序
continue
```

#### 第 3 步：打印函数参数

```gdb
# 打印 word_size 参数（对象字长数，第 2 参数在 $rsi 中）
printf "object_size = %lu words\n", $rsi
```

**示例输出**：
```
object_size = 65536 words  # 表示分配了极大对象
object_size = 32768 words
...
```

#### 第 4 步：获取 Java 调用栈

**问题**：`bt` 命令只能显示 Native 调用栈，无法显示 Java 调用栈

**解决方案**：利用 JVM 的信号处理机制
1. JVM 收到 `SIGQUIT` 信号时，会输出线程栈信息到标准输出
2. 通过 gdb 的 python 扩展执行 `kill -3 <pid>` 发送 SIGQUIT
3. 在 JVM 日志中查看 Java 调用栈

```gdb
# gdb python 扩展
python import os
python os.system("kill -3 <jvm_pid>")

# 或直接执行
shell kill -3 <jvm_pid>
```

#### 第 5 步：分析根因

JVM 标准输出中显示大对象分配的 Java 线程栈：

```
"RPC-Handler" tid=0x00007f8a1c0d2800 nid=0x1a3d runnable
  at sun.misc.Unsafe.allocateMemory(Native Method)
  at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:123)
  at com.alibaba.remoting.Decoder.decode(Decoder.java:456)  # <-- 这里调用 thrift 反序列化
  ...
```

**根因**：Thrift 反序列化时未限制最大数据长度，异常数据导致创建超大数组，触发大对象分配。

### 3.6 gdb 其他常用功能

#### 获取原生线程栈

```bash
# 附加到进程
gdb -p <pid>

# gdb 命令行
(gdb) thread apply all bt

# 或生成线程转储文件
(gdb) generate-core-file dump.core
```

#### 堆内存转储

```gdb
# 转储 JVM 堆内存
dump memory heap_dump.bin 0x7f0000000000 0x7f8000000000

# 用于调试段错误(Segmentation Fault)、内存损坏等
```

---

## 四、strace/gdb 性能开销

### 4.1 原理分析

**strace、peekfd、gdb 都基于 Linux ptrace 机制实现**

ptrace 系统调用流程：
```
Debugger(strace/gdb)
       ↓
    ptrace()
       ↓ 阻塞被追踪进程
  Kernel Scheduler
       ↓ 切换到被追踪进程
Target Process
       ↓ 执行一条指令/系统调用
  Kernel 捕获事件
       ↓
 返回到 Debugger
```

### 4.2 性能影响因素

| 因素 | 影响程度 |
|------|---------|
| 系统调用频率 | **直接相关** - 频率越高影响越大 |
| 上下文切换次数 | **线性增加** - 每次追踪都触发上下文切换 |
| 数据大小 | **中等影响** - strace 追踪大数据时开销更大 |

### 4.3 性能评估

- **低频场景**（如数据库连接）：影响通常在 5%-10%
- **高频场景**（如 mmap、futex 调用）：影响可能达到 50%-200%
- **极高频场景**（微秒级操作）：可能导致程序卡死

### 4.4 最佳实践

1. **优先级排序**：
   - 第一选择：性能分析工具(perf、flamegraph)
   - 第二选择：日志/监控系统
   - 第三选择：strace/gdb（使用时谨慎）

2. **使用建议**：
   - 在非生产环境或低流量时使用
   - 尽量使用过滤选项（`-e trace=file` 等）缩小追踪范围
   - 限制追踪时间，不要长时间运行
   - 大流量系统需提前评估性能影响

---

## 五、ASCII 流程图

### 5.1 strace 追踪系统调用机制 - ptrace 链路

```
┌─────────────────────────────────────────────────────────────┐
│                      strace/gdb 工作流程                      │
└─────────────────────────────────────────────────────────────┘

用户启动 strace -p <pid>
  │
  ├─→ strace 进程调用 ptrace(PTRACE_ATTACH, <pid>)
  │
  └─→ ┌──────────────────────────────────────────┐
      │       Linux Kernel - ptrace 机制        │
      └──────────────────────────────────────────┘
            │
            ├─→ 发送 SIGSTOP 到目标进程
            │
            ├─→ 目标进程暂停，进入 T(Traced) 状态
            │
            ├─→ 返回控制权给 strace 进程
            │
            └─→ ┌──────────────────────────────────┐
                │     strace 进入 wait 循环         │
                └──────────────────────────────────┘
                      │
                      ├─→ ptrace(PTRACE_SYSCALL)
                      │   │ 恢复目标进程运行
                      │   │
                      │   ├─→ 目标进程执行系统调用
                      │   │
                      │   └─→ 系统调用陷入(trap)
                      │       │
                      │       └─→ Kernel 暂停目标进程
                      │           （产生 SIGTRAP 信号）
                      │
                      ├─→ strace 被唤醒，读取：
                      │   ├─→ 系统调用号
                      │   ├─→ 参数寄存器(rdi,rsi,rdx...)
                      │   ├─→ 返回值寄存器(rax)
                      │
                      ├─→ 格式化输出到 stdout/文件
                      │   如: "sendto(3, "SELECT...", 100, ...) = 100"
                      │
                      ├─→ ptrace(PTRACE_SYSCALL) 继续
                      │
                      └─→ 循环直到进程退出

┌──────────────────────────────────────────────────────────────┐
│                   性能开销来源                                │
├──────────────────────────────────────────────────────────────┤
│ 1. 每个系统调用 → 两次上下文切换                              │
│    (目标进程陷入 → strace 读取 → 目标进程恢复)               │
│ 2. 高频系统调用 → 上下文切换频繁 → 缓存失效                  │
│ 3. 数据拷贝开销 → ptrace 需要读内存获取参数值                │
└──────────────────────────────────────────────────────────────┘
```

### 5.2 gdb 断点定位流程 - 大对象分配场景

```
┌─────────────────────────────────────────────────────────────┐
│           gdb 脚本定位 Java 大对象分配流程                    │
└─────────────────────────────────────────────────────────────┘

1. 初始化 gdb 环境
   ├─→ handle SIGPIPE nostop    # 不处理 SIGPIPE
   ├─→ handle SIGTERM nostop    # 不处理 SIGTERM
   └─→ handle SIGINT stop       # SIGINT 停止(用于 Ctrl+C 退出)

2. 设置断点
   ├─→ break _ZN15G1CollectedHeap22humongous_obj_allocateEmh
   │   │
   │   └─→ Kernel 在函数入口插入 INT3 指令(0xCC)
   │       
   └─→ continue              # 启动 JVM

3. 监听断点事件
   ├─→ JVM 执行 → 命中 INT3 指令
   │
   ├─→ Kernel SIGTRAP 信号 → gdb 捕获
   │
   ├─→ 提取信息
   │   ├─→ $rdi = this 指针 (G1CollectedHeap 对象)
   │   ├─→ $rsi = size 参数 (对象字长数)
   │   ├─→ 程序计数器(RIP) = 断点函数地址
   │
   ├─→ printf "size = %lu words\n", $rsi
   │   输出例: "size = 65536 words"  (≈ 512MB 内存)
   │
   └─→ continue              # 恢复执行

4. 获取 Java 调用栈
   ├─→ bt                    # 只显示 Native 调用栈，无法看到 Java 代码
   │
   ├─→ python import os; os.system("kill -3 <jvm_pid>")
   │   │
   │   └─→ 发送 SIGQUIT 信号给 JVM
   │
   └─→ JVM 收到 SIGQUIT → 线程栈输出到 stdout
       ├─→ "RPC-Handler" tid=xxx nid=0x1a3d
       ├─→ at com.alibaba.remoting.Decoder.decode()
       └─→ ...

5. 分析根因
   └─→ 发现 Decoder.decode() 中调用 Thrift 反序列化
       └─→ Thrift 未限制最大长度
           └─→ 异常数据 → 创建超大数组
               └─→ 大对象分配 → Full GC 频繁

┌──────────────────────────────────────────────────────────────┐
│                  调用约定说明                                 │
├──────────────────────────────────────────────────────────────┤
│ x86-64 函数调用约定(System V AMD64 ABI):                      │
│                                                               │
│ C++ 实例方法参数传递顺序:                                     │
│ ┌─────────┬──────────────────────────────────┐               │
│ │ 参数位置 │         寄存器                    │               │
│ ├─────────┼──────────────────────────────────┤               │
│ │ this    │ rdi (第 1 参数)                   │               │
│ │ param2  │ rsi (第 2 参数) ← word_size 在此  │               │
│ │ param3  │ rdx (第 3 参数)                   │               │
│ │ param4  │ rcx (第 4 参数)                   │               │
│ │ param5+ │ 栈上传递                          │               │
│ └─────────┴──────────────────────────────────┘               │
│                                                               │
│ 因此: printf "$rsi" 获取大对象的字长数                        │
└──────────────────────────────────────────────────────────────┘
```

### 5.3 SQL 耗时追踪链路

```
┌─────────────────────────────────────────────────────────────┐
│              Java 调用 MySQL 的系统调用链路                   │
└─────────────────────────────────────────────────────────────┘

用户请求 (HTTP GET /api/user?id=123)
  │
  ├─→ Tomcat 接收请求
  │   strace 追踪: accept() → 返回连接 FD
  │
  ├─→ 业务代码处理请求
  │
  └─→ 生成 SQL: SELECT * FROM users WHERE id=123
      │
      ├─→ [T1] sendto(fd=3, SQL数据包, len=100, ...)
      │   │
      │   └─→ strace 记录:
      │       sendto(3, "SELECT * FROM users...", 100) = 100
      │       耗时: 0.001s
      │
      ├─→ 等待 MySQL 响应... (网络延迟/数据库处理)
      │
      ├─→ [T2] recvfrom(fd=3, 缓冲区, 4096, ...)
      │   │
      │   └─→ strace 记录:
      │       recvfrom(3, "result_data", 256) = 256
      │       耗时: 0.152s
      │
      ├─→ SQL 耗时计算: T2 - T1 = 0.152 - 0.001 = 0.151s
      │
      ├─→ 业务代码继续处理结果
      │
      ├─→ 生成 HTTP 响应
      │   strace 追踪: write(fd=1, response_data, ...)
      │
      └─→ 写入 access.log
          strace 追踪: open(), write(), close() 等

使用 awk 脚本自动化解析:
┌──────────────────────────────────────────┐
│ awk '                                    │
│   /sendto.*IP:PORT/ {                   │
│     start = $NF                          │
│     next                                 │
│   }                                      │
│   /recvfrom.*IP:PORT/ {                 │
│     end = $NF                            │
│     delay = end - start                  │
│     print "SQL耗时:", delay, "秒"        │
│   }                                      │
│ ' strace.log                             │
└──────────────────────────────────────────┘
```

---

## 六、原文勘误

### 勘误 1：ptrace 机制的上下文切换说明

**原文位置**：第 1748-1749 行

**原文内容**：
> strace/peekfd/gdb 命令都是基本 Linux 的 ptrace 机制实现的，ptrace 机制是 Linux 设计用来调试程序的一种机制，所以并没有太考虑性能因素，执行过程中会导致线程上下文切换。

**勘误内容**：
- 说法不够精确：应强调每个系统调用会导致**两次上下文切换**（陷入 → 返回）
- 原文缺少对上下文切换影响的详细解释（缓存失效、TLB 刷新等）

**建议补充**：
> strace/peekfd/gdb 都基于 ptrace 机制实现。每个被追踪的系统调用会导致至少两次上下文切换：
> 1. 系统调用陷入时：目标进程 → debugger 进程
> 2. 返回时：debugger 进程 → 目标进程
> 
> 频繁的上下文切换会导致 CPU 缓存失效、TLB 刷新，性能开销甚大。

---

### 勘误 2：C++ 名称改写的解释不够深入

**原文位置**：第 1685-1687 行

**原文内容**：
> 因为 C++为了兼容 C 的二进制 ABI，编译时函数名会改写(mangle)，可通过 nm 查询改写后的函数名。

**勘误内容**：
- 原文没有解释名称改写的具体机制
- 没有给出如何手工解析编码函数名的方法

**建议补充**：
> C++ 函数名编码(Name Mangling)规则遵循 Itanium ABI 标准：
> - 前缀 `_Z` 标记为 C++ 符号
> - 后跟命名空间长度与名称、类名长度与名称
> - 最后跟参数类型的单字符编码(e=unsigned long, h=unsigned char, P=指针等)
> 
> 例: `_ZN15G1CollectedHeap22humongous_obj_allocateEmh`
> - `_Z` = C++ 符号
> - `N...E` = 嵌套命名空间
> - `15G1CollectedHeap` = 类名长度(15) + 类名
> - `22humongous_obj_allocate` = 方法名长度(22) + 方法名
> - `Emh` = 参数(unsigned long m, unsigned char h)

---

### 勘误 3：gdb 寄存器与参数对应关系的说明不够完整

**原文位置**：第 1696 行

**原文内容**：
> printf指令：格式化打印当前rsi寄存器的值，x86架构中rsi保存第二个参数的值，由于C++中第一个参数是this，故word_size的值在$rsi中。

**勘误内容**：
- 原文仅提及 x86 架构但没有明确说是 x86-64
- 没有列出其他参数寄存器的映射关系
- 没有说明为什么是 System V AMD64 ABI

**建议补充**：
> x86-64 架构(System V AMD64 ABI)的函数参数传递约定：
> - 第 1 参数(this): rdi
> - 第 2 参数: rsi
> - 第 3 参数: rdx  
> - 第 4 参数: rcx
> - 第 5 参数: r8
> - 第 6 参数: r9
> - 第 7+ 参数：栈上传递
> 
> 因此本例中 word_size(第 2 参数)在 $rsi 中是正确的。

---

### 勘误 4：Java 线程栈信息的获取细节

**原文位置**：第 1712-1713 行

**原文内容**：
> 由于JVM在收到SIGQUIT信号时，会在标准输出中打印线程栈信息。
> 因此，通过gdb内嵌的python扩展，执行kill -3命令，以实现打印java线程栈的需求。

**勘误内容**：
- 原文没有说明线程栈输出的具体位置（stdout vs stderr vs JVM 日志文件）
- 没有说明线程 ID 与 JVM 输出中的 nid 关系（需要十进制转十六进制）
- 没有说明使用 `jstack` 工具作为更好的替代方案

**建议补充**：
> 获取 Java 线程栈的标准方法：
> 
> 方法一(gdb 方式)：
> ```gdb
> (gdb) python import os; os.system("kill -3 <jvm_pid>")
> (gdb) shell sleep 2  # 等待输出
> # JVM 线程栈输出到 catalina.out 或标准输出
> ```
> 注意：线程 ID 需转换为十六进制与 nid 对比
> 
> 方法二(推荐，jstack 工具)：
> ```bash
> jstack <jvm_pid> > jvm_threads.txt
> # 输出包含完整的 Java 调用栈，格式更规范
> ```

---

### 勘误 5：Thrift 反序列化的具体问题描述

**原文位置**：第 1728 行

**原文内容**：
> 原因是没有限制thrift反序列化的最大长度，异常数据会导致thrift创建非常大的数组，导致了大对象分配。

**勘误内容**：
- 原文没有解释"异常数据"的具体含义
- 没有说明 Thrift 协议中哪个字段导致问题（list/map 长度字段等）
- 没有给出修复建议

**建议补充**：
> Thrift 反序列化大对象问题详解：
> 
> 问题原因：
> Thrift 协议在反序列化 list/map 等集合类型时，首先读取一个 4 字节的长度字段。
> 如果网络传输中该字段被篡改或损坏为极大值(如 0xFFFFFFFF)，
> Thrift 会尝试一次性分配对应大小的数组，导致大对象分配。
> 
> 典型场景：
> ```
> Thrift List<String> 反序列化流程：
> 1. 读取 4 字节长度: 0x40000000 (1GB)
> 2. 调用 humongous_obj_allocate() 分配 1GB 数组
> 3. 后续内存压力 → Full GC → 系统卡顿
> ```
> 
> 修复方案：
> 1. 配置 Thrift transport 的 maxFrameSize(限制单个请求大小)
> 2. 在应用层验证数据大小，拒绝异常数据
> 3. 使用 Protocol Buffer 代替 Thrift(更严格的大小验证)

---

## 七、速查表

### 7.1 strace 常用命令速查

| 场景 | 命令 |
|------|------|
| 追踪进程所有系统调用 | `strace -p <pid>` |
| 只追踪文件 I/O | `strace -e trace=file -p <pid>` |
| 只追踪网络 I/O | `strace -e trace=network -T -p <pid>` |
| 打印文件描述符解码 | `strace -e trace=file -yy -p <pid>` |
| 查看读写的数据内容 | `strace -e read=3 -e write=3 -p <pid>` |
| 显示系统调用耗时 | `strace -T -p <pid>` |
| 打印调用栈 | `strace -k -e trace=file -p <pid>` |
| 统计系统调用 | `strace -c -p <pid>` |
| 输出到文件 | `strace -o /tmp/trace.log -p <pid>` |
| 追踪命令执行 | `strace -o /tmp/trace.log free` |
| 追踪新进程 | `strace -f -p <pid>` (含子进程) |

### 7.2 strace 日志解析技巧

```bash
# 查找慢操作
grep "+" trace.log | sort -t= -k2 -rn | head -20

# 统计某类系统调用
grep "sendto" trace.log | wc -l

# 提取特定 FD 的操作
grep "fd=3" trace.log

# 查找失败的调用
grep " -1 " trace.log
```

### 7.3 gdb 常用命令速查

| 命令 | 作用 |
|------|------|
| `gdb -p <pid>` | 附加到运行中的进程 |
| `gdb ./binary` | 启动程序调试 |
| `break func_name` | 在函数处设置断点 |
| `break file.cpp:100` | 在文件特定行设置断点 |
| `continue` | 继续执行 |
| `next` | 单步执行(不进入函数) |
| `step` | 单步执行(进入函数) |
| `print <var>` | 打印变量值 |
| `printf "fmt", expr` | 格式化打印 |
| `info locals` | 显示本地变量 |
| `info registers` | 显示寄存器值 |
| `backtrace` (bt) | 打印调用栈 |
| `thread apply all bt` | 所有线程调用栈 |
| `dump memory file addr1 addr2` | 内存转储 |
| `quit` | 退出 gdb |

### 7.4 nm 命令查询编码函数名

```bash
# 查找包含特定名称的符号
nm /path/to/binary | grep allocate

# 输出示例:
# 00007f0000123456 T _ZN15G1CollectedHeap22humongous_obj_allocateEmh

# 使用 c++filt 解码
c++filt _ZN15G1CollectedHeap22humongous_obj_allocateEmh
# 输出: G1CollectedHeap::humongous_obj_allocate(unsigned long, unsigned char)
```

### 7.5 peekfd 常用命令

| 命令 | 作用 |
|------|------|
| `peekfd <pid>` | 查看所有文件描述符的数据 |
| `peekfd -n 100 <pid> 3` | 查看 FD 3，最多 100 行 |
| `peekfd -n 50 <pid> 3 4 5` | 查看多个 FD |

### 7.6 问题诊断决策树

```
程序性能问题
  │
  ├─→ 症状：频繁 GC / OOM
  │   └─→ 方案：gdb 断点 + Java 调用栈
  │       (定位大对象分配代码)
  │
  ├─→ 症状：SQL 执行慢
  │   └─→ 方案：strace 网络调用 + awk 解析
  │       (计算 sendto-recvfrom 时间差)
  │
  ├─→ 症状：文件 I/O 慢
  │   └─→ 方案：strace -e trace=file -T
  │       (找出耗时最长的调用)
  │
  ├─→ 症状：进程卡死
  │   └─→ 方案：gdb attach + bt + info threads
  │       (查看是否死锁或陷入系统调用)
  │
  └─→ 症状：网络连接问题
      └─→ 方案：strace -e trace=network -yy
          (查看网络操作与 FD 对应关系)
```

---

## 八、总结速记

| 工具 | 主要用途 | 使用难度 | 性能影响 |
|------|---------|---------|---------|
| **strace** | 追踪系统调用、诊断 I/O 问题 | 低 | 中-高(高频调用时) |
| **peekfd** | 查看 I/O 数据流内容 | 低 | 中 |
| **gdb** | 函数调试、内存检查、线程分析 | 中-高 | 中(断点场景) |

### 核心要点

1. **strace 适合场景**：
   - 诊断"为什么程序这么慢"
   - 学习命令如何实现
   - SQL/网络 I/O 耗时定位

2. **gdb 适合场景**：
   - 定位大对象分配、内存泄漏
   - 获取精确的调用栈信息
   - 条件断点调试复杂逻辑

3. **性能考虑**：
   - 不要在高流量生产环境直接使用
   - 优先考虑 perf、flamegraph 等采样工具
   - 需要时使用过滤选项降低开销

---

生成完成。统计信息如下：

- **章节数**：8 个主要章节
- **ASCII 流程图数**：3 个
- **勘误条目数**：5 条
- **速查表数**：6 个表格
