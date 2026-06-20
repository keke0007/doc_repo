# Arthas Java 故障分析工具梳理

> **版本**: 基于原文 ppt-merged.md 第 1888-2194 行  
> **日期**: 2026-05-24

---

## 一、工具概述

### Arthas 是什么

**Arthas** 是 Java 平台下的动态追踪工具，主要用于线上故障排查。

**核心能力**：
- 观测 Java 方法的调用参数、返回值
- 反编译类文件（检验代码版本）
- 线程剖析与分析
- 堆内存转储与分析
- 火焰图生成
- 字节码增强与监控

**设计理念**：无需修改代码、无需重启应用，即可进行深度诊断。

---

## 二、核心命令分类体系

### 2.1 系统概览类（Dashboard/Thread/Memory）

#### 1. dashboard 命令
**功能**：类似 Linux `top` 的 JVM 概览

```
arthas> dashboard
```

**显示内容**：
- JVM 整体运行信息（堆内存、GC 统计）
- 线程数量与状态分布
- CPU 利用率
- 类加载统计

**适用场景**：快速获取系统全景、初步定位高 CPU/高内存问题

---

#### 2. thread 命令
**功能**：查看详细的线程信息（常用）

```bash
arthas> thread
arthas> thread -1                    # 查看所有线程
arthas> thread {tid}                 # 查看指定线程
arthas> thread -b                    # 查看阻塞的线程
arthas> thread -i 1000               # 采样间隔 1000ms
```

**显示内容**：
- 线程 ID、状态、名称
- 是否为守护线程
- CPU 时间与用户时间
- 阻塞信息与调用栈

**关键应用**：
- 排查死锁（DeadLock）
- 发现阻塞线程
- 高 CPU 消耗线程定位

---

#### 3. memory 命令
**功能**：查看内存占用情况（常用）

```bash
arthas> memory
```

**显示内容**：
- Heap 堆内存（总、已用、可用）
- NonHeap 堆外内存（元数据、代码缓存等）
- GC 统计（YGC 次数、耗时、FGC 次数、耗时）

**关键指标**：
- `heap.used / heap.max` → 堆内存使用率
- `gc.mark-sweep.count / gc.mark-sweep.time` → FullGC 压力

**常见问题**：内存泄漏检测、GC 压力分析

---

### 2.2 类操作类（SC/SM/JAD）

#### 4. sc 命令 - 查找类
**功能**：搜索已加载的类

```bash
arthas> sc java.lang.String
arthas> sc -d java.lang.String              # 显示详细信息
arthas> sc *Exception                       # 模糊查询
arthas> sc -f com.example.MyClass           # 显示类的字段
```

**关键输出**：
- `code-source` → 类加载源路径（jar 包位置）

**应用场景**：
- **排查 Jar 包冲突**：对比不同版本类的加载路径
- **检验类是否存在**：确认发布内容
- **识别类加载器**：多类加载器场景

---

#### 5. sm 命令 - 查找方法
**功能**：查看类中的所有方法

```bash
arthas> sm java.lang.String
arthas> sm -d java.lang.String              # 显示方法详细签名
arthas> sm java.lang.String toString        # 查找特定方法
```

**输出内容**：
- 方法名、参数类型、返回类型
- 方法签名（descriptor）
- 修饰符（public/private/static 等）

**应用**：确认方法是否存在、参数类型校验

---

#### 6. jad 命令 - 反编译类
**功能**：将 .class 字节码反编译为 Java 源代码

```bash
arthas> jad java.lang.String
arthas> jad --source-only com.example.MyClass    # 仅输出源代码
arthas> jad --source-only com.example.MyClass | wc -l  # 统计行数
```

**用途**：
- **验证代码版本**：检查线上代码是否是最新版本
- **排查业务逻辑**：没有源代码时在线查看实现
- **对比差异**：反编译线上代码与本地代码进行对比

**重要提示**：
- 反编译的代码是从字节码推导，可能不保留注释和变量名
- 适合快速查看逻辑，不能作为文档

---

### 2.3 方法调用追踪类（Monitor/Watch/Trace/Stack）

#### 7. monitor 命令 - 方法调用统计
**功能**：监视方法的整体调用情况

```bash
arthas> monitor -c 1 com.example.UserService login
```

**核心指标**：
- `QPS` → 每秒调用次数
- `avg` → 平均耗时（ms）
- `max` → 最大耗时（ms）
- `fail` → 失败次数

**示例输出解读**：
```
QPS: 70/s | avg: 1ms | max: 5ms | fail: 0
```
表示方法每秒被调用约 70 次，平均耗时 1ms，无失败。

**应用**：
- 方法性能基准测量
- QPS 波动分析
- 性能衰退检测

---

#### 8. watch 命令 - 观测方法参数与返回值
**功能**：实时观测方法调用的参数、返回值、异常

```bash
arthas> watch com.example.UserService login "{params[0], returnObj}"
arthas> watch com.example.UserService login "{params, returnObj, throwExp}" -x 3
```

**内置变量**：
- `params[0]` / `params[1]` → 第 1、2 个参数
- `target` → 调用对象本身（this）
- `returnObj` → 方法返回值
- `throwExp` → 异常对象
- `#cost` → 方法耗时（ms）

**条件过滤**：
```bash
# 仅观测耗时 > 200ms 的调用
arthas> watch com.example.UserService login "{params[0]}" '#cost > 200'

# 仅观测参数 id > 1000 的调用
arthas> watch com.example.UserService queryUser "{params[0]}" 'params[0] > 1000'
```

**高级用法**：
```bash
# 监控数据库查询 SQL
arthas> watch com.example.dao.UserDao query "{params, returnObj}"

# 展开对象深度
arthas> watch com.example.UserService login "{params, returnObj}" -x 5
```

**应用场景**：
- 获取慢查询的 SQL 语句
- 验证参数与返回值
- 异常分析

---

#### 9. trace 命令 - 方法耗时追踪
**功能**：追踪方法的调用链路及各环节耗时，定位性能瓶颈

```bash
arthas> trace com.example.UserService login
arthas> trace -E com.example..*Service .*                    # 正则匹配
arthas> trace com.example.UserService login '#cost > 100'    # 耗时过滤
```

**输出形式**：树形结构，展示子方法调用关系与耗时

**示例解读**：
```
login() @ms
  └─ queryUser()      @2ms
     └─ db.select()   @1.9ms
  └─ checkAuth()      @0.5ms
  └─ updateLog()      @1.5ms
total cost: 4ms
```

**含义**：
- 主方法总耗时 4ms
- 其中数据库查询占 1.9ms（最主要的性能瓶颈）
- 其他操作分别占 0.5ms 和 1.5ms

**条件优化**：
```bash
# 仅关注耗时超过 50ms 的调用链
arthas> trace com.example.UserService login '#cost > 50'
```

**应用**：
- 定位致命的性能热点子方法
- 分析调用链深度
- 优化决策制定

---

#### 10. stack 命令 - 调用栈追踪
**功能**：追踪方法的调用栈，找到代码发起入口

```bash
arthas> stack com.example.UserService login
arthas> stack com.example.db.execute '#cost > 100'          # 条件过滤
```

**输出形式**：方法从入口到目标方法的完整调用栈（反序）

**应用**：
- 追溯问题方法被谁调用
- 找到业务请求的发起点
- 排查意外调用的来源

---

### 2.4 性能热点剖析（Profiler）

#### 11. profiler 命令 - 函数热点分析

**说明**：Arthas 对 async-profiler 的封装，采用采样模式（非插桩），开销极小。

##### a) On-CPU 分析

**原理**：采样 CPU 正在执行的栈，识别 CPU 时间消耗最多的方法

```bash
# 采集 30 秒 on-cpu 火焰图
arthas> profiler start --duration 30 --format html

# 停止并输出 HTML 火焰图
arthas> profiler stop --format html --file /tmp/cpu.html
```

**火焰图解读**：
- 横轴宽度 → 方法消耗的 CPU 时间比例（越宽越耗 CPU）
- 纵轴 → 调用栈深度
- 顶部最宽的方法 → 最大的 CPU 瓶颈

**应用**：
- 识别 CPU 热点方法
- 算法优化决策
- 编译器工作量诊断

---

##### b) Off-CPU 分析（Wall Clock 模式）

**原理**：wall clock（墙上挂钟）采集模式，周期性采集所有线程栈，统计墙时钟时间

```bash
# wall 模式采集 30 秒
arthas> profiler start --mode wall --duration 30 --format html

# 过滤特定线程（排除线程池空闲线程）
arthas> profiler start --mode wall --duration 30 --thread 'http-nio.*'
```

**特点**：
- 可以捕捉包括 Lock/IO 等待的时间
- 适合排查高耗时问题
- 慢调用栈在火焰图上会更宽

**Wall vs On-CPU 对比**：
| 模式 | 采集内容 | 适用场景 |
|------|--------|--------|
| on-cpu | 仅 CPU 执行时间 | 纯计算热点 |
| wall | CPU + 等待时间 | 高延迟、IO 阻塞 |

**注意事项**：
- Wall 模式会采集所有线程栈，包括闲置线程
- 为避免噪音，一般需要按线程名过滤（如 `http-nio-8080-exec-*`）

---

##### c) JFR 格式采集（Case-by-Case 分析）

**场景**：问题偶现（如请求偶尔慢、每天只出现一次），需要捕获单个案例

```bash
# 采集为 jfr 格式（可保存完整调用栈）
arthas> profiler start --format jfr --file /tmp/profile.jfr
arthas> profiler stop

# 使用 shell 脚本循环采集直到问题出现
#!/bin/bash
while true; do
  arthas> profiler start --format jfr --file /tmp/profile-$(date +%s).jfr
  sleep 60
  arthas> profiler stop
  # 问题出现后手动停止脚本
done
```

**JFR vs HTML 火焰图**：
| 格式 | 优点 | 缺点 |
|------|------|------|
| HTML 火焰图 | 易于快速查看热点 | 相同栈折叠，丢失单个调用信息 |
| JFR | 保留完整栈、单个请求链路 | 需要 JMC 工具分析 |

**分析工具**：JFR 文件用 **Java Mission Control（JMC）** 打开分析

**实例应用**：
- 发现某线程（如 `http-nio-8080-exec-28`）在 21:14:10 - 21:14:18 期间耗时 8s
- 在 JMC 中查找该时段的完整调用栈，识别致命方法

---

##### d) Native 方法采集

**功能**：采集 C/C++ 级函数调用栈（如 malloc、pthread 等）

```bash
arthas> profiler start --event cpu --cstack lbr --format html
arthas> profiler stop --format html
```

**应用**：
- 诊断堆外内存泄漏
- 分析 JNI 调用性能
- 系统库调用热点

**示例**：
- 发现 malloc 占用大量时间 → 堆外内存频繁分配
- 发现 pthread_mutex_lock → 锁争用问题

---

##### e) 其他事件采集

```bash
# 查看支持的所有事件
arthas> profiler list

# 例：堆内存分配追踪
arthas> profiler start --event alloc --format html

# 例：Lock 争用分析
arthas> profiler start --event lock --format html
```

**支持事件**（取决于 perf_events）：
- `cpu` - CPU 时间
- `alloc` - 堆内存分配
- `lock` - Lock 争用
- 其他 perf_events 支持的事件

---

### 2.5 对象查询与管理（vmtool/mbean/logger/getstatic/ognl）

#### 12. vmtool 命令 - 查询 Java 对象

**基础**：基于 JVMTI（Java Virtual Machine Tool Interface）技术

```bash
# 查询 Tomcat 线程池对象
arthas> vmtool --action forceGc
arthas> vmtool --action instances --className org.apache.catalina.Executor

# 遍历对象列表（instances 数组）
arthas> vmtool --action instances --className org.apache.catalina.Executor -x 5
```

**核心用途**：
- **查询对象实例**：找到内存中的对象
- **强制 GC**：`forceGc` 触发垃圾回收
- **对象内省**：查看对象的字段状态

**应用场景**：诊断资源池状态

---

#### 13. vmtool 应用实例

**场景 1：查看 Tomcat 线程池使用情况**

```bash
arthas> vmtool --action instances --className org.apache.catalina.ThreadPoolExecutor
```

输出示例：
```
instances count: 2
  instance[0]: org.apache.catalina.ThreadPoolExecutor@0x7f1a2c3d4e5f
    - poolSize: 10
    - corePoolSize: 5
    - activeCount: 8
```

**含义**：线程池有 8 个活跃线程在工作，总池大小 10，核心池大小 5。

---

**场景 2：查看 Druid 连接池状态**

```bash
arthas> vmtool --action instances --className com.alibaba.druid.pool.DruidDataSource -x 3
```

关键字段：
- `activeCount` → 活跃连接数
- `poolingCount` → 闲置连接数
- `maxActive` → 最大连接数

**应用**：检测连接池耗尽

---

**场景 3：查看 Apache HttpClient 连接池**

```bash
arthas> vmtool --action instances --className org.apache.http.impl.conn.PoolingClientConnectionManager
```

---

#### 14. mbean 命令 - JMX Beans 查询

**原理**：Java 组件可将内部状态暴露为 MBean，供 JMX 监控

```bash
# 查看所有 MBeans
arthas> mbean

# 查看特定 MBean（如 Druid）
arthas> mbean java.lang:type=Memory
arthas> mbean com.alibaba.druid:*
```

**对比 vmtool**：
| 工具 | 方式 | 适用对象 |
|------|------|--------|
| vmtool | 内存扫描 | 所有对象 |
| mbean | JMX 接口 | 支持 MBean 的组件 |

---

#### 15. logger 命令 - 日志配置管理

**功能**：查看和动态修改日志级别

```bash
# 查看所有 logger
arthas> logger

# 查看特定 logger
arthas> logger -n com.example.service

# 修改日志级别为 DEBUG
arthas> logger -n com.example.service -l debug

# 修改为 INFO
arthas> logger -n com.example.service -l info
```

**应用**：
- 动态增加日志级别排查问题
- 无需重启应用生效
- 临时诊断后可恢复

---

#### 16. getstatic 命令 - 获取静态变量值

```bash
# 获取静态变量
arthas> getstatic com.example.Config FTP_PORT

# 示例输出
arthas> getstatic com.example.Config FTP_PORT
  FTP_PORT: 0
```

**应用**：
- 查看静态配置值
- 验证初始化是否正确
- 问题排查（如端口未初始化）

---

#### 17. ognl 命令 - 在 JVM 中执行表达式

**OGNL**（Object-Graph Navigation Language）：对象图导航语言，支持强大的表达式

```bash
# 获取静态变量
arthas> ognl '@java.lang.Runtime@getRuntime()'

# 调用方法
arthas> ognl '@com.example.Util@parseConfig()'

# 操作对象
arthas> ognl '#obj=new java.util.Date(), #obj.toString()'

# 创建集合
arthas> ognl '#list={"a","b","c"}, #list'
```

**OGNL 常用语法**：

| 分类 | 语法 | 示例 |
|------|------|------|
| 内置变量（watch/trace 中）| `params`, `target`, `returnObj`, `throwExp`, `#cost` | `params[0]`, `#cost > 100` |
| 属性访问 | `.property` | `.user.name` |
| 数组/集合索引 | `[index]`, `["key"]` | `[0]`, `["userId"]` |
| 方法调用 | `.method()` | `.getUser()` |
| 静态成员 | `@Class@member` / `@Class@method()` | `@java.lang.Runtime@getRuntime()` |
| 条件判断 | `> >= == != < <=` | `.id > 100`, `.name=="admin"` |
| 逻辑运算 | `&& \|\| !` | `params[0]>10 && returnObj!=null` |
| 数组/集合包含 | `in`, `not in` | `"admin" in .roles` |
| 变量赋值 | `#var=expression` | `#obj=new java.util.Date()` |
| 列表构造 | `{e1, e2, ...}` | `{"red", "green", "blue"}` |
| Map 构造 | `#{"key": value, ...}` | `#{"id": 1, "name": "lisi"}` |
| 列表转换 | `.collection.{expression}` | `.users.{name}` 提取所有名字 |
| 列表过滤 | `.collection.{?expression}` | `.users.{? age > 18}` 过滤成年用户 |

**特殊语法**：
```bash
# OGNL 用逗号分隔语句，返回最后一个对象
arthas> ognl '#a=1, #b=2, #a+#b'
# 输出：3
```

**指定类加载器**：
```bash
# -c 参数指定类加载器（通过 classloader -t 获取）
arthas> ognl -c 0x7f1a2c3d4e5f 'expression'
```

**应用**：
- 动态调用方法验证逻辑
- 修改运行时变量（风险较大，谨慎使用）
- 复杂查询与转换

---

### 2.6 JVM 信息查询（jvm/sysenv/vmoption/sysprop/perfcounter）

#### 18. jvm 命令 - 查看 JVM 运行状态

```bash
arthas> jvm
```

**输出内容**：
- 类加载统计（loaded、total、unloaded）
- GC 统计（YGC、FGC 次数和耗时）
- 内存分布（Heap、NonHeap）
- 线程数统计（live、peak）
- 文件描述符（open、max）

**关键指标**：
- `class loaded: 5000` → 已加载类数
- `gc.mark-sweep.count: 3` → FullGC 次数
- `thread live: 100` → 当前活跃线程

---

#### 19. sysenv 命令 - 系统环境变量

```bash
arthas> sysenv
arthas> sysenv JAVA_HOME                    # 查看特定变量
```

**应用**：验证运行环境配置

---

#### 20. vmoption 命令 - JVM 启动参数

```bash
arthas> vmoption
arthas> vmoption -n MaxHeapSize              # 查看特定参数
```

**示例输出**：
```
-Xmx2048m
-Xms1024m
-XX:+UseG1GC
```

---

#### 21. sysprop 命令 - Java 系统属性

```bash
arthas> sysprop
arthas> sysprop java.version                # 查看 Java 版本
```

**常见属性**：
- `java.version` → Java 版本号
- `java.home` → Java 安装路径
- `user.timezone` → 时区设置
- `file.encoding` → 文件编码

---

#### 22. perfcounter 命令 - JVM 性能计数器

```bash
arthas> perfcounter
```

**输出**：各种实时性能指标
- 内存分配速率
- GC 相关指标
- 线程创建率
- 类加载率

---

## 三、关键流程与架构图

### 3.1 Arthas Attach 流程

```
┌─────────────────────────────────────────────────────────────────┐
│                     应用启动（目标 JVM）                          │
│              ┌────────────────────────────────┐                 │
│              │   JVM Process (Target App)    │                 │
│              │  ├─ Bytecode               │                 │
│              │  ├─ Runtime Classes        │                 │
│              │  └─ JVM Memory             │                 │
│              └────────────────────────────┘                 │
└─────────────────────────────────────────────────────────────────┘
                              △
                              │
                              │ Socket Connection
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    Arthas 客户端进程                             │
│         ┌──────────────────────────────────┐                    │
│         │  1. java -jar arthas-boot.jar   │                    │
│         │  2. 扫描本地 JVM 进程列表       │                    │
│         │  3. 用户选择目标 PID            │                    │
│         │  4. AttachAPI 连接目标 JVM     │                    │
│         │  5. 建立 Socket 通信            │                    │
│         │  6. REPL 交互命令循环           │                    │
│         └──────────────────────────────────┘                    │
│                  ↓                                              │
│    ┌──────────────────────────────────┐                       │
│    │  命令解析与执行                    │                       │
│    │  - thread / dashboard / monitor   │                       │
│    │  - watch / trace / stack          │                       │
│    │  - profiler / jad / sc            │                       │
│    └──────────────────────────────────┘                       │
└─────────────────────────────────────────────────────────────────┘

时序：
  1. Attach 请求 ─────────────────→ 目标 JVM
  2. 目标 JVM 加载 Arthas Agent ─→ 初始化
  3. 建立通信通道 ←───────────────→ Arthas 客户端
  4. REPL 循环 ←───────────────→ 命令执行反馈
```

---

### 3.2 Watch/Trace 字节码增强链路

```
┌─────────────────────────────────────────────────────────────────┐
│           用户执行 watch / trace 命令                             │
│  例：watch com.example.UserService login "{params, returnObj}"  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│         1. 命令解析 & 表达式编译                                 │
│  ┌──────────────────────────────────────┐                      │
│  │ 解析：目标类=UserService             │                      │
│  │       目标方法=login                 │                      │
│  │       观测表达式={params,returnObj}  │                      │
│  │       条件=#cost>100(可选)           │                      │
│  └──────────────────────────────────────┘                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│         2. 类加载与字节码定位                                    │
│  ┌──────────────────────────────────────┐                      │
│  │ 通过类名定位 UserService class       │                      │
│  │ 查找 login(String username) 方法   │                      │
│  │ 获取方法的字节码指令序列             │                      │
│  └──────────────────────────────────────┘                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│         3. 字节码注入与增强                                      │
│  ┌──────────────────────────────────────┐                      │
│  │ 原方法：login()                      │                      │
│  │   │                                  │                      │
│  │   ├─ 执行业务逻辑                    │                      │
│  │   └─ 返回 result                     │                      │
│  │                                     │                      │
│  │ 增强后：login()                      │                      │
│  │   ├─ [增强前处理] 记录 params     │                      │
│  │   ├─ 执行原业务逻辑                │                      │
│  │   ├─ [增强后处理] 记录 returnObj  │                      │
│  │   ├─ [异常处理] 记录 throwExp    │                      │
│  │   └─ 返回 result                   │                      │
│  └──────────────────────────────────────┘                      │
│                                                                 │
│  使用技术：Javassist / ASM（字节码操作库）                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│         4. 类重新加载 (HotSwap)                                  │
│  ┌──────────────────────────────────────┐                      │
│  │ 使用 Instrumentation API 重新加载   │                      │
│  │ 已定义的类，覆盖原始字节码           │                      │
│  │ (不中断正在执行的方法)               │                      │
│  └──────────────────────────────────────┘                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│         5. 监听与输出                                            │
│  ┌──────────────────────────────────────┐                      │
│  │ 每次调用时，增强代码执行:            │                      │
│  │  - 捕获 params 值                    │                      │
│  │  - 记录调用时间戳                    │                      │
│  │  - 执行条件判断 (#cost > 100 ?)    │                      │
│  │  - 捕获 returnObj / throwExp        │                      │
│  │  - 发送数据到客户端输出              │                      │
│  └──────────────────────────────────────┘                      │
└─────────────────────────────────────────────────────────────────┘

开销分析：
  - 未触发条件的调用：极小开销（仅条件判断）
  - 触发条件的调用：记录参数、返回值、表达式求值
  - 复杂表达式（深层对象遍历）：开销相对较大
```

---

### 3.3 Profiler 火焰图采集流程

```
┌─────────────────────────────────────────────────────────────────┐
│                 用户启动 Profiler                                │
│     arthas> profiler start --duration 30 --format html          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│      1. 采样配置                                                │
│  ┌────────────────────────────────────────┐                    │
│  │ 选择采样事件：                          │                    │
│  │  - on-cpu (默认): CPU 执行时间          │                    │
│  │  - wall: 墙时钟（包含等待）            │                    │
│  │  - alloc: 堆内存分配                   │                    │
│  │  - lock: 锁争用                        │                    │
│  │                                       │                    │
│  │ 采样频率: 99 Hz (每秒 99 次采样)      │                    │
│  │ 采样间隔: ~10ms 一次                   │                    │
│  └────────────────────────────────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│      2. 采集阶段 (30s 循环)                                     │
│  ┌────────────────────────────────────────┐                    │
│  │ 每 ~10ms                               │                    │
│  │  │                                    │                    │
│  │  ├─ 触发采样中断                      │                    │
│  │  ├─ 遍历所有线程                      │                    │
│  │  ├─ 捕获当前调用栈                    │                    │
│  │  │   例：                             │                    │
│  │  │   String.equals (at CPU)           │                    │
│  │  │   └─ AbstractList.indexOf          │                    │
│  │  │      └─ UserService.query          │                    │
│  │  │         └─ main (http handler)     │                    │
│  │  │                                    │                    │
│  │  ├─ 累积计数 (stack frequency++)      │                    │
│  │  └─ 继续执行业务逻辑                  │                    │
│  │                                       │                    │
│  │ 重复 3000 次（30s × 99Hz）            │                    │
│  └────────────────────────────────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│      3. 数据聚合                                                │
│  ┌────────────────────────────────────────┐                    │
│  │ 合并相同调用栈：                        │                    │
│  │                                       │                    │
│  │ String.equals           [采样 500 次]  │                    │
│  │  └─ indexOf             [采样 500 次]  │                    │
│  │      └─ query           [采样 500 次]  │                    │
│  │                                       │                    │
│  │ → query() 总采样 500 次                │                    │
│  │ → 消耗总 CPU 时间: 500 × 10ms = 5s   │                    │
│  │ → CPU 占比: 5s / 30s ≈ 16.7%         │                    │
│  └────────────────────────────────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│      4. 火焰图生成                                              │
│  ┌────────────────────────────────────────┐                    │
│  │ 构建树形结构，按采样频率计算宽度：    │                    │
│  │                                       │                    │
│  │   ┌─────────────────┐                 │                    │
│  │   │  main (root)    │  [30s]           │                    │
│  │   └─────────────────┘                 │                    │
│  │     ├─ http-handler [20s] ══════════  │                    │
│  │     │   ├─ query     [5s]  ═══════    │                    │
│  │     │   │   └─ indexOf [5s] ═════     │                    │
│  │     │   │       └─ equals [5s] ════   │                    │
│  │     │   └─ other    [15s]  ═══════════│                    │
│  │     └─ gc-thread    [10s] ═════════   │                    │
│  │                                       │                    │
│  │ 宽度 ∝ 采样频率（CPU 消耗）           │                    │
│  │ 高度 = 调用栈深度                     │                    │
│  └────────────────────────────────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│      5. 输出文件                                                │
│  ┌────────────────────────────────────────┐                    │
│  │ --format html   → 交互式 HTML 火焰图   │                    │
│  │ --format jfr    → 二进制 JFR 文件      │                    │
│  │ --format txt    → 文本堆栈表示         │                    │
│  │                                       │                    │
│  │ 文件保存位置                          │                    │
│  └────────────────────────────────────────┘                    │
└─────────────────────────────────────────────────────────────────┘

性能特性：
  - 采样式：开销 < 5%（非插桩）
  - 无需重启：动态 attach
  - 支持 Linux/Mac/Windows
```

---

## 四、OGNL 高级语法详解

### 4.1 表达式基础

```bash
# 简单属性访问
arthas> ognl '@java.lang.Runtime@getRuntime().availableProcessors()'

# 变量赋值与使用
arthas> ognl '#a=1, #b=2, #a+#b'
# 输出: 3

# 对象创建
arthas> ognl '#obj=new java.util.Date(), #obj.toString()'
```

### 4.2 集合操作

```bash
# List 构造与遍历
arthas> ognl '{"apple", "banana", "cherry"}'

# Map 构造
arthas> ognl '#{"id": 1, "name": "Alice", "age": 30}'

# List 投影（提取特定字段）
arthas> ognl '.users.{name}'         # 提取所有用户名
arthas> ognl '.orders.{totalPrice}'  # 提取所有订单总额

# List 过滤
arthas> ognl '.users.{? age > 18}'                # 过滤成年人
arthas> ognl '.orders.{? status == "completed"}' # 过滤已完成订单

# 集合包含判断
arthas> ognl '"admin" in .roles'       # 判断是否有 admin 角色
arthas> ognl '"guest" not in .roles'   # 判断是否没有 guest 角色
```

### 4.3 watch/trace 中的内置变量

```bash
# watch 中的使用
arthas> watch com.example.Service method "{params[0], target, returnObj, #cost}" '#cost > 100'

# trace 中的使用
arthas> trace com.example.Service method '#cost > 50'

# stack 中的使用
arthas> stack com.example.Service method '#cost > 200'
```

---

## 五、Arthas 命令速查表

### 快速诊断路径

| 问题类型 | 推荐命令链 | 预期耗时 |
|---------|----------|--------|
| **高 CPU** | dashboard → thread -b → profiler on-cpu | 5-10min |
| **高内存** | memory → dashboard → vmtool instances | 2-5min |
| **方法慢** | monitor -c 1 → trace → watch | 5-10min |
| **偶现慢请求** | profiler jfr（循环）→ JMC 分析 | 依赖问题复现周期 |
| **死锁** | thread -b | < 1min |
| **代码版本确认** | jad --source-only | < 1min |
| **连接池耗尽** | vmtool instances | < 1min |
| **日志级别动态调整** | logger -n com.pkg -l debug | < 1min |

---

### 常用命令速查

```bash
# ━━━ 系统概览 ━━━
dashboard                           # JVM 总体状态（类似 top）
thread                             # 线程列表
thread -b                          # 阻塞的线程
memory                             # 内存占用

# ━━━ 类与方法 ━━━
sc java.lang.String               # 查找类（查看 code-source）
sm java.lang.String               # 查看类的所有方法
jad java.lang.String              # 反编译类

# ━━━ 方法追踪 ━━━
monitor -c 1 com.ex.Service m1    # 监控方法 QPS/耗时
watch com.ex.Service m1 "{params, returnObj}"   # 观测参数与返回值
trace com.ex.Service m1           # 追踪子方法耗时链路
stack com.ex.Service m1           # 追踪方法调用栈

# ━━━ 热点分析 ━━━
profiler start --duration 30      # 采集 on-cpu 30 秒
profiler start --mode wall        # 采集 wall clock（包含等待）
profiler stop --format html       # 输出火焰图

# ━━━ 对象查询 ━━━
vmtool --action instances --className org.example.Pool -x 3
mbean java.lang:type=Memory       # 查看 MBean
vmtool --action forceGc           # 强制 GC

# ━━━ 日志与配置 ━━━
logger                            # 查看所有 logger
logger -n com.ex -l debug         # 修改日志级别
getstatic com.ex.Config PORT      # 获取静态变量
ognl '@java.lang.Runtime@getRuntime().availableProcessors()'  # 执行表达式

# ━━━ JVM 信息 ━━━
jvm                               # 详细 JVM 统计
sysenv JAVA_HOME                  # 系统环境变量
vmoption MaxHeapSize              # JVM 启动参数
sysprop java.version              # Java 系统属性
perfcounter                       # 性能计数器
```

---

## 六、⚠️ 原文勘误与补充

### 勘误 1：trace 输出时间单位不清
**原文**（第 1982 行）：
> "trace命令用于追踪方法耗时，由哪个子方法造成。"

**补充**：
- 输出中的时间单位为 **毫秒 (ms)**
- 火焰图从上到下显示调用嵌套关系
- 同级方法按耗时排序

---

### 勘误 2：profiler wall 模式过滤说明不够明确
**原文**（第 2015 行）：
> "由于wall采集模式，会采集所有线程栈，但一般线程池中大量线程都是等待任务的状态，故wall采集模式一般要过滤一下"

**补充**：
过滤语法示例：
```bash
# 仅采集 HTTP 处理线程
arthas> profiler start --mode wall --duration 30 --thread 'http-nio-8080-exec-*'

# 仅采集业务线程（排除 GC、IO 等待线程）
arthas> profiler start --mode wall --duration 30 --thread 'pool-*'
```

---

### 勘误 3：JFR 分析工具名称
**原文**（第 2034 行）：
> "jfr文件可以通过jmc工具来分析"

**补充**：
- **JMC** = Java Mission Control（图形化分析工具）
- JFR 是记录格式，JMC 是查看器
- 开源替代：Async Profiler 的 viewer.html

---

### 勘误 4：profiler 采集事件的可用性
**原文**（第 2054 行）：
> "profiler还可以剖析一些其它事件，如堆内存分配、lock等"

**补充**：
- 可用事件取决于操作系统与 perf_events 支持
- Linux: 支持最全面
- macOS: 部分事件支持
- Windows: 支持有限

查看支持的事件：
```bash
arthas> profiler list
```

---

### 勘误 5：ognl 逗号分隔的返回值
**原文**（第 2143 行）：
> "ognl是用，号分隔语句的，且返回表达式中最后一个对象"

**补充**：
```bash
# 示例：返回最后一个值
arthas> ognl '#a=1, #b=2, #a+#b'
# 输出：3（不是 a 或 b，而是表达式结果）

# 示例：返回对象
arthas> ognl '#obj=new java.util.Date(), #obj.toString()'
# 输出：日期字符串（toString() 结果）
```

---

### 补充 1：Arthas Attach 超时
- 默认超时时间：30 秒
- 如果目标 JVM 无响应，会连接失败
- 某些高 CPU 场景下可能超时

---

### 补充 2：字节码增强的生命周期
- 增强仅在当前会话有效
- 断开连接后增强会被撤销（恢复原始字节码）
- 多个 watch/trace 命令可叠加

---

### 补充 3：profiler 采样模式对比

| 模式 | 采样对象 | 精度 | 开销 | 适用场景 |
|------|--------|------|------|--------|
| on-cpu | CPU 正在执行 | 高 | 低 | 纯 CPU 热点 |
| wall | 所有线程当前栈 | 中 | 中 | 高延迟、阻塞 |
| alloc | 内存分配调用 | 中 | 中 | 内存泄漏 |
| lock | 锁争用事件 | 低 | 低 | 并发竞争 |

---

## 七、Arthas 学习路径建议

### 初级（快速定位）
1. `dashboard` - 获取系统全景
2. `thread -b` - 发现阻塞
3. `jad` - 查看代码版本
4. `memory` - 检查内存

### 中级（深度分析）
1. `monitor` - 方法性能基准
2. `watch` - 参数与返回值观测
3. `trace` - 子方法耗时链路
4. `profiler on-cpu` - CPU 热点

### 高级（复杂问题）
1. `profiler wall` + 线程过滤 - 排查高延迟
2. `vmtool instances` - 对象状态诊断
3. `ognl` - 动态代码执行与修改
4. `profiler jfr` + JMC - case-by-case 分析

---

## 八、常见陷阱与最佳实践

### 陷阱 1：watch 表达式复杂度过高
```bash
# 不推荐：深层嵌套会导致开销大
arthas> watch com.ex.Service m1 "{params[0].user.profile.settings.deepConfig}"

# 推荐：分步骤观测
arthas> watch com.ex.Service m1 "{params[0].user}"
```

### 陷阱 2：trace 命令在高 QPS 下输出过多
```bash
# 推荐：加过滤条件
arthas> trace -E com.ex..*Service .* '#cost > 100'
```

### 陷阱 3：profiler 采集时间设置不合理
```bash
# 不推荐：太短（< 10s）噪音太多，太长（> 5min）数据过大
# 推荐：30-60 秒为佳
arthas> profiler start --duration 60
```

### 陷阱 4：wall 模式不过滤导致数据无用
```bash
# 推荐：按业务线程过滤
arthas> profiler start --mode wall --thread 'http-nio-8080-exec-*' --duration 30
```

---

**本文完成**

---

## 统计

- **章节数**：8 个主要章节 + 40+ 子知识点
- **ASCII 流程图**：3 个
  1. Arthas Attach 流程
  2. Watch/Trace 字节码增强链路
  3. Profiler 火焰图采集流程
- **勘误数**：5 条 + 3 条补充说明
- **速查表**：2 个（诊断路径表 + 命令速查）

**生成文件**：`C:\Users\ke\Desktop\claude\troubleshooting-梳理\11_arthas_梳理.md`
