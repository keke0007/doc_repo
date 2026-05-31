# Java 高效 Debug 排错技巧 - 知识梳理

## 目录

1. [调试基础概念](#1-调试基础概念)
2. [IDEA 断点系统](#2-idea-断点系统)
3. [自上而下调试法](#3-自上而下调试法)
4. [自底向上调试法](#4-自底向上调试法)
5. [高级调试技巧](#5-高级调试技巧)
6. [调试流程图](#6-调试流程图)
7. [速查表](#7-速查表)
8. [原文勘误](#8-原文勘误)

---

## 1. 调试基础概念

### 1.1 调试的核心思路

Java 程序调试主要分为两大类思路：

#### 自上而下调试
- **适用场景**：调试比较熟悉的代码（如业务代码）
- **核心方法**：一步步执行代码，追踪程序执行流
- **优点**：逻辑清晰，易于定位业务逻辑问题
- **缺点**：需要熟悉代码结构

#### 自底向上调试
- **适用场景**：调试不熟悉的代码（如框架源码）
- **核心方法**：从底层关键方法入手，逐层向上追踪调用栈
- **优点**：可快速找到调试入口
- **缺点**：需要了解框架核心方法

### 1.2 调试工作流程

```
1. 启动程序 → 以调试模式启动应用
2. 设置断点 → 在目标代码位置标记中断点
3. 等待触发 → 程序运行至断点自动暂停
4. 逐步执行 → 单步、进入、退出、继续等控制
5. 观察状态 → 查看变量、调用栈、表达式结果
6. 分析问题 → 根据观察结果推断问题根因
```

---

## 2. IDEA 断点系统

### 2.1 行断点（Line Breakpoint）

**设置方法**：在代码行号左侧单击鼠标左键

**特点**：
- 最基础的断点类型
- 程序执行到该行时自动暂停
- 支持快速打/删除断点

**使用场景**：
- 追踪特定代码行的执行
- 快速定位程序异常位置

```java
// 示例：在 Line 10 打断点
public void processOrder(Order order) {
    validateOrder(order);           // 第 8 行
    applyDiscount(order);           // 第 9 行
    saveToDatabase(order);          // 第 10 行 <- 在此打断点
}
```

### 2.2 方法断点（Method Breakpoint）

**设置方法**：在方法定义行的左侧单击

**特点**：
- 在方法入口处触发暂停
- 可区分方法进入和退出
- 开销相对较大

**使用场景**：
- 追踪方法调用流
- 观察方法参数和返回值

### 2.3 异常断点（Exception Breakpoint）

**设置方法**：
1. 打开 Run → View Breakpoints（或 Ctrl+Shift+F8）
2. 点击 "+" → Exception
3. 选择异常类型（如 SQLException、NullPointerException）

**特点**：
- 在特定异常抛出时触发断点
- 自动捕获异常堆栈信息
- 支持选择捕获时机（Throw 或 Catch）

**使用场景**：
- 快速定位异常源头
- 追踪特定类型的异常处理流程

**示例**：为 SQLException 设置异常断点

```
Run → View Breakpoints
  → "+" 按钮
  → Exception
  → 搜索 "SQLException"
  → 配置捕获时机（Throw at 抛出处）
```

### 2.4 字段断点（Field Breakpoint）

**设置方法**：
1. 在字段定义处打断点
2. 或在 Breakpoints 窗口中配置字段观察

**特点**：
- 在字段被访问或修改时触发
- 支持读取、写入、读写三种模式
- 开销最大，应谨慎使用

**使用场景**：
- 追踪共享状态变化
- 定位字段非预期修改的源头

### 2.5 条件断点（Conditional Breakpoint）

**设置方法**：
1. 右击断点 → Edit Breakpoint（或按 Ctrl+Shift+F8）
2. 在 "Condition" 字段输入布尔表达式

**条件表达式规则**：
- 使用 Java 语言编写
- 可访问当前作用域的所有局部变量和 this
- 支持方法调用（但可能有性能影响）
- 返回 true 时断点被命中

**示例**：

```java
// 仅当 userId == 1 时命中断点
userId == 1

// 仅当订单总额超过 1000 时
order.getTotalAmount() > 1000

// 仅当列表非空且第一个元素为特定值
!list.isEmpty() && list.get(0).getId() == 42

// 组合条件
status.equals("ERROR") && requestTime > 1000L
```

**性能注意**：
- 条件表达式会影响调试性能
- 避免在条件中调用耗时方法
- 复杂条件应该使用日志而非断点

---

## 3. 自上而下调试法

### 3.1 基本流程

```
选择调试代码
    ↓
启动调试模式 (Debug Run)
    ↓
在关键位置设置断点
    ↓
手动触发程序逻辑
    ↓
程序暂停在断点处
    ↓
单步执行 & 观察变量状态
    ↓
进入函数 / 退出函数
    ↓
重复直到找到问题根因
```

### 3.2 IDE 启动调试模式

**IDEA 中的方式**：
- 菜单：Run → Debug (项目名)
- 快捷键：Shift+F9（默认）
- 工具栏：点击 Debug 按钮（虫子图标）

**启动后状态**：
- 应用在调试模式下运行
- 断点已激活
- Debug 窗口在下方打开

### 3.3 单步执行的四种控制方式

#### 3.3.1 单步调试 (Step Over)

**快捷键**：F6（Eclipse 快捷键）/ F10（IDEA 默认）

**行为**：
- 执行当前行代码
- 跳过函数调用内部细节
- 光标移到下一行

**使用场景**：
- 快速跳过已验证的代码
- 专注于特定逻辑块

#### 3.3.2 调试进入函数 (Step Into)

**快捷键**：F5（Eclipse 快捷键）/ F11（IDEA 默认）

**行为**：
- 跳入当前行的函数内部
- 暂停在函数的第一行
- 允许逐行观察函数内部逻辑

**使用场景**：
- 追踪函数内部实现
- 定位函数调用中的问题

**智能进入变体**（IDEA 特有）：
- Smart Step Into：在有多个函数调用时，让用户选择进入哪个
- 快捷键：Shift+F7
- 用途：多参数函数或链式调用时区分调用对象

#### 3.3.3 调试退出函数 (Step Out)

**快捷键**：F7（Eclipse 快捷键）/ Shift+F8（IDEA 默认）

**行为**：
- 执行完当前函数
- 暂停在函数调用处的下一行
- 立即退出函数调试

**使用场景**：
- 确认函数不存在问题后快速退出
- 返回调用者继续调试

#### 3.3.4 继续运行 (Resume)

**快捷键**：F8（Eclipse 快捷键）/ F9（IDEA 默认）

**行为**：
- 继续程序执行
- 直到下一个断点命中或程序结束
- 程序恢复正常速度运行

**使用场景**：
- 跳过不关心的代码段
- 快速到达下一个感兴趣的位置

### 3.4 运行到指定位置 (Run to Cursor)

**方法**：
- 菜单：Run → Run to Cursor
- 快捷键：Ctrl+Alt+F10（IDEA 默认）
- 或在代码行右击选择 "Run to Cursor"

**行为**：
- 程序运行至光标所在行
- 不需要预先设置断点
- 灵活快速

**使用场景**：
- 临时跳过大量代码
- 快速到达需要调试的位置

### 3.5 变量观察

#### 3.5.1 变量窗口

**位置**：Debug 面板左下方的 "Variables" 标签

**显示内容**：
- 当前作用域的局部变量
- this 对象及其字段值
- 基本类型的值
- 对象的完整结构

**互动功能**：
- 右击变量 → "Set Value" 修改变量值
- 支持在调试中改变程序状态
- 用于测试特定数据流向

#### 3.5.2 监视窗口 (Watches)

**使用方法**：
- 右击变量 → "Add to Watches"
- 或在 Watches 面板直接添加表达式

**特点**：
- 自定义监视表达式
- 持久化保存（调试会话间保留）
- 支持复杂的 Java 表达式

**示例**：

```java
// 可在 Watches 中添加：
order.getTotalAmount() * 1.1  // 计算打折后价格
user.getName().toUpperCase()  // 处理字符串
list.stream().filter(x -> x > 0).count()  // 流式计算
```

#### 3.5.3 表达式求值 (Evaluate Expression)

**打开方法**：
- Alt+F9（IDEA 默认）
- 或 Run → Evaluate Expression
- 或在 Debug 窗口右击 → Evaluate Expression

**功能**：
- 在当前暂停位置执行任意 Java 代码
- 查看表达式的计算结果
- 调用方法、创建对象等

**示例用途**：

```java
// 计算复杂表达式
Math.pow(order.getAmount(), 2) + calculateBonus()

// 调用方法获取信息
user.getProfile().toJson()

// 创建对象进行测试
new SimpleDateFormat("yyyy-MM-dd").format(new Date())
```

**注意事项**：
- 表达式执行可能修改程序状态
- 避免调用有副作用的方法
- 不支持创建新的类或修改字节码

### 3.6 调用栈分析

#### 3.6.1 调用栈窗口

**位置**：Debug 面板的 "Frames" 或 "Call Stack" 标签

**显示内容**：
```
getCurrentUser()  [line 42 in UserService.java]
    ↑
findUserById()    [line 128 in UserRepository.java]
    ↑
query()           [line 456 in JdbcTemplate.java]
    ↑
...
```

**导航功能**：
- 点击任何栈帧可跳转到该方法的对应代码行
- 在该栈帧的上下文中查看变量
- 理解程序的调用链路

#### 3.6.2 栈帧操作

**Reset Frame（重置栈帧）**

**快捷键**：无（需在栈帧上右击）

**行为**：
- 将调试指针重置到该函数的入口
- 重新执行函数（从头开始）
- 用于重新调试同一函数

**使用场景**：
- 发现函数内某个变量设置错误后
- 想重新执行该函数时

**示例**：

```
调试流程中错误地跳过了一些代码行
    ↓
Reset Frame 回到函数起始
    ↓
重新执行函数，这次更仔细观察
```

**Force Return（强制返回）**

**方法**：在栈帧上右击 → Force Return

**行为**：
- 立即返回当前函数
- 指定返回值（如 null、0、false 等）
- 跳过函数剩余的代码

**使用场景**：
- 测试上游代码对特定返回值的处理
- 绕过有问题的子函数
- 模拟不同的函数返回值

**示例**：

```java
// 调试中发现某个外部服务调用可能超时
// 可强制返回一个默认值继续调试
Force Return: return null;  
// 观察调用者是否正确处理 null 返回值
```

**Throw Exception（抛出异常）**

**方法**：在栈帧上右击 → Throw Exception

**行为**：
- 立即抛出指定异常
- 打断当前执行流
- 用于模拟异常场景

**使用场景**：
- 测试异常处理代码
- 模拟特定异常情况

---

## 4. 自底向上调试法

### 4.1 核心思路

自底向上调试适合于调试不熟悉的框架代码或第三方库，通过在底层关键方法设置断点，沿着调用栈逐层向上追踪，快速找到合适的调试入口。

```
问题现象（如数据查询失败）
    ↓
定位底层关键方法（如 ResultSet.next()）
    ↓
在底层打条件断点
    ↓
观察断点命中时的完整调用栈
    ↓
按调用栈逐层分析框架代码
    ↓
找到问题所在的框架层级
    ↓
再用自上而下法精细调试该层
```

### 4.2 JDBC 核心方法梳理

理解 JDBC 调用链是自底向上调试的基础：

#### 4.2.1 JDBC 执行流

```
1. DataSource.getConnection()       ← 获取数据库连接
2. Connection.prepareStatement()    ← 编译 SQL 语句
3. PreparedStatement.setXXX()       ← 设置参数
4. PreparedStatement.execute()      ← 执行 SQL
5. Statement.getResultSet()         ← 获取结果集
6. ResultSet.next()                 ← 逐行遍历结果
7. ResultSet.getXXX()               ← 提取列值
```

#### 4.2.2 关键断点位置

**最常用的两个**：

| 方法 | 用途 | 代码片段 |
|------|------|--------|
| `Connection.prepareStatement()` | 观察 SQL 语句和参数 | SQL 准备阶段 |
| `ResultSet.next()` | 观察查询结果处理 | 结果遍历阶段 |

### 4.3 案例：MyBatis 查不到数据问题

#### 4.3.1 问题现象

```
业务代码：SELECT * FROM user WHERE id = 1
数据库查询：确实有数据
但程序查到：NULL / 空集合
```

#### 4.3.2 排查过程

**第一步：自底向上定位问题层级**

在 `ResultSet.next()` 打条件断点

```
观察调用栈：
├─ com.mysql.jdbc.ResultSet.next()           ← JDBC 层
├─ org.apache.ibatis.mapping.ResultSet.xxx() ← MyBatis 层
├─ org.apache.ibatis.executor.xxx()          ← 执行器
└─ com.example.service.UserService.xxx()     ← 业务层
```

**第二步：定位 MyBatis 处理结果的方法**

从调用栈发现 MyBatis 在 `handleRowValuesForSimpleResultMap` 处理结果

**第三步：自上而下深入 MyBatis 源码**

在 `getRowValue()` 打断点，逐步执行：

```
getRowValue()
  ↓
applyAutomaticMappings()
  ↓
createAutomaticMappings()
  ↓
MetaObject.findProperty()  ← 查找属性
```

**第四步：发现问题**

```java
// 数据库列名：user_id
// Java 属性名：userId
// 匹配失败！

// 查看 useCamelCaseMapping = false
// 表示未启用下划线→驼峰转换
```

**第五步：解决方案**

MyBatis 配置文件添加：

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

---

## 5. 高级调试技巧

### 5.1 Stream 和 Lambda 调试

**问题**：Stream 和 Lambda 的链式调用难以用传统单步调试跟踪

#### 5.1.1 打断点策略

```java
// 原始代码
List<User> result = users.stream()
    .filter(u -> u.getAge() > 18)           // Line 1
    .map(u -> u.getName())                  // Line 2
    .collect(Collectors.toList());          // Line 3
```

**策略 1：在 Lambda 表达式中打断点**

```java
List<User> result = users.stream()
    .filter(u -> {
        System.out.println("过滤: " + u);  // 打印代替断点
        return u.getAge() > 18;
    })
    .map(u -> u.getName())
    .collect(Collectors.toList());
```

**策略 2：终端操作处打断点**

在 `collect()` 处打条件断点，观察最终结果

**策略 3：使用 peek() 进行中间观察**

```java
List<User> result = users.stream()
    .filter(u -> u.getAge() > 18)
    .peek(u -> System.out.println("过滤后: " + u))  // 观察中间结果
    .map(u -> u.getName())
    .peek(n -> System.out.println("映射后: " + n))
    .collect(Collectors.toList());
```

#### 5.1.2 Stream 调试的日志方案

相比断点，日志更适合 Stream 调试：

```java
users.stream()
    .filter(u -> {
        boolean pass = u.getAge() > 18;
        if (!pass) {
            log.debug("User filtered out: {}", u.getName());
        }
        return pass;
    })
    .collect(Collectors.toList());
```

### 5.2 远程调试（JDWP - Java Debug Wire Protocol）

远程调试用于调试运行在远程服务器上的 Java 应用程序。

#### 5.2.1 JDWP 工作原理

```
本地 IDE (IDEA)
    ↓ (JDWP 协议)
网络 (TCP Socket)
    ↓ (JDWP 协议)
远程 JVM
```

#### 5.2.2 启用远程调试

**远程 JVM 启动参数**（添加到 JVM 启动脚本）：

```bash
# 推荐用法（JDWP 5.0+，Java 9+）
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 MyApp

# 兼容用法（Java 8）
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=0.0.0.0:5005 MyApp

# 详细参数解释
# transport=dt_socket    : 使用 TCP Socket 传输
# server=y               : JVM 作为调试服务器
# suspend=n              : 启动时不暂停，立即运行
# address=*:5005         : 监听所有网卡的 5005 端口
```

**参数详解**：

| 参数 | 值 | 说明 |
|------|-----|------|
| transport | dt_socket / dt_shmem | 网络(Socket)或本地(共享内存) |
| server | y / n | JVM 是否作为服务器 |
| suspend | y / n | 启动时是否等待调试器连接 |
| address | host:port | 监听的地址和端口 |

#### 5.2.3 IDEA 中连接远程调试

**步骤 1**：创建 Remote 调试配置

```
Run → Edit Configurations
  → "+" 按钮
  → Remote → 新建
```

**步骤 2**：配置连接参数

```
Name: MyApp Remote Debug
Host: 远程服务器 IP（如 192.168.1.100）
Port: 5005  (与远程 JVM 启动参数一致)
```

**步骤 3**：启动调试

```
Run → Debug 'MyApp Remote Debug'
```

**步骤 4**：提示成功连接

```
Connected to the target VM, address: '192.168.1.100:5005', transport: 'socket'
```

#### 5.2.4 远程调试流程图

```
启动远程 JVM
(指定 JDWP 端口 5005)
    ↓
IDE 发起 TCP 连接
(目标地址 192.168.1.100:5005)
    ↓
握手协议
(JDWP 认证)
    ↓
传输调试命令
(设置断点、单步、获取变量等)
    ↓
JVM 返回调试信息
(变量值、栈帧、执行状态)
    ↓
IDE 更新 UI
(显示变量、断点状态)
    ↓
循环直到调试结束
```

#### 5.2.5 远程调试注意事项

**安全性**：
- 不要在公网暴露调试端口
- 在生产环境应使用防火墙限制 JDWP 访问
- 考虑使用 SSH 隧道安全连接

```bash
# 通过 SSH 隧道安全连接远程调试
ssh -L 5005:localhost:5005 user@remote-server.com

# 然后在 IDEA 配置本地连接
Host: localhost
Port: 5005
```

**性能**：
- JDWP 调试会显著降低 JVM 性能
- 避免在高流量生产环境启用
- 生产调试改用日志或其他非中断方式

**同步问题**：
- 确保本地代码与远程运行的代码版本一致
- 否则行号映射会出错

### 5.3 表达式求值进阶

#### 5.3.1 表达式求值常用场景

| 场景 | 表达式示例 |
|------|-----------|
| 计算数值 | `user.getAge() * 1.5` |
| 字符串操作 | `user.getName().substring(0, 3).toUpperCase()` |
| 集合操作 | `users.stream().filter(u -> u.getAge() > 30).count()` |
| 对象创建 | `new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01")` |
| 方法调用 | `calculateBonus(user.getSalary())` |
| 条件判断 | `order.getTotalAmount() > 100 && order.isActive()` |

#### 5.3.2 表达式求值的限制

```java
// ❌ 不支持：创建新类
class NewClass { }

// ❌ 不支持：修改方法定义
public void newMethod() { }

// ✓ 支持：调用现有方法
user.getName()

// ✓ 支持：创建对象实例
new User("John", 30)

// ⚠️ 注意：可能有副作用的操作
list.add(newElement)  // 修改了 list
user.setAge(100)      // 修改了 user
```

### 5.4 Drop Frame（回退栈帧）

Drop Frame 功能允许将调试指针回退到上一个函数调用处，重新执行该函数。

#### 5.4.1 使用场景

```java
// 调试过程
procesOrder()
  ↓ Step Into
  validateOrder()    // ← 进入此函数
    // 单步执行几行后发现问题
    // 想重新执行该函数

// 解决方案
1. 选中 validateOrder 栈帧
2. 点击 "Drop Frame" 按钮
3. 回到 processOrder 的调用处
4. 重新 Step Into 进入 validateOrder
```

#### 5.4.2 Drop Frame 的限制

- 仅当 JVM 支持时可用
- 某些优化过的方法无法回退
- 适合调试层级不太深的调用

### 5.5 HotSwap 和类重定义

HotSwap 允许在调试时修改代码并立即重新加载，无需重启 JVM。

#### 5.5.1 HotSwap 工作原理

```
编辑代码文件
    ↓
按 Ctrl+Shift+F9 (或菜单 Build → Recompile)
    ↓
IDEA 将新字节码发送到 JVM
    ↓
JVM 的 JDWP 接收到类重定义请求
    ↓
如果修改合法，JVM 立即替换旧类
    ↓
继续调试（无需重启）
```

#### 5.5.2 HotSwap 支持的修改

✓ **支持**：
- 修改方法体内的代码
- 添加新的局部变量
- 修改字段初始值

❌ **不支持**：
- 添加/删除方法
- 添加/删除字段
- 修改方法签名
- 修改类结构

#### 5.5.3 IDEA 中启用 HotSwap

**配置步骤**：

```
File → Settings → Build, Execution, Deployment → Debugger → HotSwap
  ✓ 勾选 "Enable on frame deactivation"
  ✓ 设置 "Reload classes after compilation" 为 Always
```

**使用方法**：

```
1. 启动调试
2. 修改代码
3. Ctrl+Shift+F9 (重新编译)
4. IDEA 自动尝试 HotSwap 加载
5. 如果成功，调试继续；如果失败，提示需要重启
```

### 5.6 Arthas 动态调试

Arthas 是一个开源的 Java 诊断工具，可在不中断程序的情况下进行实时调试。

#### 5.6.1 Arthas 安装和启动

```bash
# 下载 Arthas
curl -O https://arthas.aliyun.com/arthas-boot.jar

# 启动 Arthas
java -jar arthas-boot.jar

# 选择要诊断的 Java 进程
# 输入进程号或名字，进入 Arthas 命令行
```

#### 5.6.2 Arthas 与 HotSwap 类似功能：ognl 和 jad

**ognl - 动态执行表达式**

```bash
# 列出所有 Bean
ognl -x 3 #context

# 调用方法
ognl @java.lang.System@currentTimeMillis()

# 获取字段值
ognl @com.example.Config@INSTANCE.name
```

**jad - 实时反编译查看字节码**

```bash
# 查看 UserService 类的源代码
jad com.example.service.UserService

# 查看特定方法
jad com.example.service.UserService getUserById
```

#### 5.6.3 Arthas 类重定义（redefine）

虽然 Arthas 本身不直接支持 HotSwap，但可以通过 jad/sc 导出字节码，修改后用 redefine 重新加载：

```bash
# 1. 反编译类
jad com.example.service.UserService > UserService.java

# 2. 修改 UserService.java

# 3. 编译
javac UserService.java

# 4. 将修改的类文件加载回 JVM
# （需要配合其他工具）
```

### 5.7 日志打点和 Log Placeholder

相比中断式调试，日志打点在生产环境和异步代码中更适用。

#### 5.7.1 日志打点策略

```java
// 策略 1：简单日志
public void processOrder(Order order) {
    log.info("开始处理订单：{}", order.getId());
    
    boolean valid = validateOrder(order);
    log.debug("订单验证结果：{}", valid);
    
    if (valid) {
        saveToDatabase(order);
        log.info("订单保存成功：{}", order.getId());
    }
}

// 策略 2：条件日志（避免无用日志）
public List<User> findUsers(String name) {
    List<User> result = userRepository.findByName(name);
    
    if (result == null || result.isEmpty()) {
        log.warn("未找到用户，搜索条件：{}", name);
    } else if (log.isDebugEnabled()) {
        log.debug("找到 {} 个用户", result.size());
    }
    
    return result;
}

// 策略 3：使用 MDC 追踪请求
MDC.put("requestId", UUID.randomUUID().toString());
try {
    log.info("处理请求开始");
    processRequest();
    log.info("处理请求结束");
} finally {
    MDC.remove("requestId");
}
```

#### 5.7.2 Log Placeholder（日志占位符）

**SLF4J 占位符**：

```java
// ✓ 推荐用法（使用占位符）
log.info("用户登录，ID：{}，时间：{}", userId, loginTime);

// ❌ 不推荐（字符串拼接）
log.info("用户登录，ID：" + userId + "，时间：" + loginTime);
// 问题：即使日志级别不输出，拼接操作也会执行
```

**日志级别的占位符链式操作**：

```java
// 避免在日志级别不输出时仍然执行操作
if (log.isDebugEnabled()) {
    log.debug("复杂对象：{}", expensiveToString(complexObject));
}

// 或使用 Lambda（SLF4J 1.8+）
log.debug("复杂对象：{}", () -> expensiveToString(complexObject));
```

### 5.8 jdb（Java Debugger）

jdb 是 JDK 自带的命令行调试工具，用于调试本地或远程 Java 程序。

#### 5.8.1 本地调试

```bash
# 启动程序并等待调试器连接
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005 MyApp

# 另一个终端连接调试器
jdb -attach localhost:5005
```

#### 5.8.2 jdb 常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `help` | 显示帮助 | `help` |
| `threads` | 列出所有线程 | `threads` |
| `thread <id>` | 切换线程 | `thread 1` |
| `stop in` | 在指定方法设置断点 | `stop in MyClass.myMethod` |
| `stop at` | 在指定行设置断点 | `stop at MyClass:50` |
| `clear` | 删除断点 | `clear MyClass:50` |
| `step` | 单步执行 | `step` |
| `next` | 下一行 | `next` |
| `cont` | 继续执行 | `cont` |
| `print` | 打印表达式 | `print user.getName()` |
| `dump` | 转储对象 | `dump user` |
| `where` | 显示调用栈 | `where` |
| `list` | 列出源代码 | `list` |

#### 5.8.3 jdb 优缺点

**优点**：
- 纯命令行，无需 IDE
- 适合服务器环境

**缺点**：
- 用户体验不如 IDE
- 命令较多，学习曲线陡
- 代码展示不直观

### 5.9 JFR（Java Flight Recorder）

JFR 是 Java 的轻量级连续性能监控工具，可在生产环境记录 JVM 的详细运行信息。

#### 5.9.1 JFR 工作原理

```
JFR 持续记录 JVM 事件
(无明显性能开销)
    ↓
事件存储到循环缓冲区
(常驻内存，自动覆盖旧事件)
    ↓
问题发生时导出记录
(从最近 N 分钟的记录中)
    ↓
JDK Mission Control 分析
(展示性能指标、火焰图等)
```

#### 5.9.2 启用 JFR

**启动 JVM 时启用**：

```bash
# Java 11+ 推荐用法
java -XX:+UnlockCommercialFeatures -XX:+FlightRecorder \
     -XX:StartFlightRecording=duration=60s,filename=myrecording.jfr \
     MyApp

# Java 8u262+ 用法
java -XX:+UnlockCommercialFeatures -XX:+FlightRecorder \
     -XX:+UnlockDiagnosticVMOptions -XX:+DebugNonSafepoints \
     -XX:StartFlightRecording=duration=60s,filename=myrecording.jfr \
     MyApp
```

#### 5.9.3 jcmd 命令启动 JFR

```bash
# 查看运行的 JVM 进程
jcmd -l

# 启动 JFR 记录
jcmd <pid> JFR.start duration=120s filename=recording.jfr

# 检查 JFR 状态
jcmd <pid> JFR.check

# 停止并导出
jcmd <pid> JFR.dump filename=recording.jfr
```

#### 5.9.4 分析 JFR 记录

```bash
# 使用 JDK Mission Control 打开文件
jmc recording.jfr

# 或使用命令行分析
jcmd <pid> JFR.dump filename=recording.jfr
# 然后用 IDE（IDEA、Eclipse）打开 JFR 文件查看分析报告
```

#### 5.9.5 JFR 适用场景

**适合于**：
- 生产环境性能监控
- 长期运行的问题诊断
- CPU、内存、IO、锁竞争分析
- 垃圾回收性能分析

**不适合于**：
- 开发调试（太笨重）
- 实时交互调试（无法中断和单步）

---

## 6. 调试流程图

### 6.1 IDEA 远程调试流程图（JDWP Attach 链路）

```
┌─────────────────────────────────────────────────────────────────┐
│                     JDWP 远程调试链路                            │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────┐
│  远程服务器      │
│  192.168.1.100   │
│                  │
│  ┌─────────────┐ │
│  │   JVM App   │ │
│  │             │ │
│  │ (JDWP      │ │
│  │  Server)   │ │
│  │ :5005      │ │
│  └─────────────┘ │
└────────┬─────────┘
         │
         │ TCP Socket
         │ 192.168.1.100:5005
         │
┌────────▼──────────────────────────────────────┐
│              网络                             │
│         (JDWP 协议通信)                       │
└────────┬──────────────────────────────────────┘
         │
┌────────▼─────────────────────────────────┐
│          本地 IDE (IDEA)                  │
│                                           │
│  ┌─────────────────────────────────────┐ │
│  │  Debug Configurations               │ │
│  │  Host: 192.168.1.100                │ │
│  │  Port: 5005                         │ │
│  └─────────────────────────────────────┘ │
│                                           │
│  ┌─────────────────────────────────────┐ │
│  │  Debug Client                       │ │
│  │  - 发送断点命令                     │ │
│  │  - 接收事件通知                     │ │
│  │  - 获取变量值                       │ │
│  └─────────────────────────────────────┘ │
│                                           │
│  ┌─────────────────────────────────────┐ │
│  │  UI 更新                            │ │
│  │  - 展示变量窗口                     │ │
│  │  - 显示调用栈                       │ │
│  │  - 代码行高亮                       │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────────┘

通信流程：
1. JVM 启动时注册调试端口
2. IDE 连接到调试端口
3. 握手确认协议版本
4. IDE 下发调试命令（断点、单步等）
5. JVM 在事件发生时回复数据
6. IDE 解析数据更新 UI
7. 用户控制调试过程
```

### 6.2 条件断点求值流程

```
┌─────────────────────────────────────────────────┐
│       条件断点求值流程                          │
└─────────────────────────────────────────────────┘

程序执行到断点行
    │
    ▼
┌──────────────────────────────┐
│ 1. 执行条件表达式            │
│    userId == 1               │
└──────────────────────────────┘
    │
    ├─────────────┬─────────────┐
    │             │             │
    ▼             ▼             ▼
  true         false         异常
    │             │             │
    │             │             ▼
    │             │        ┌──────────────────┐
    │             │        │ 异常处理         │
    │             │        │ - 记录异常       │
    │             │        │ - 暂停执行       │
    │             │        │ - 提示用户       │
    │             │        └──────────────────┘
    │             │
    │             ▼
    │        ┌──────────────────┐
    │        │ 继续执行          │
    │        │ (不中断)         │
    │        └──────────────────┘
    │
    ▼
┌──────────────────────────────┐
│ 2. 中断程序执行              │
│    - 保存现场（栈帧）       │
│    - 收集调用栈             │
│    - 获取变量值             │
└──────────────────────────────┘
    │
    ▼
┌──────────────────────────────┐
│ 3. IDE 接收事件             │
│    - 更新变量窗口           │
│    - 显示调用栈             │
│    - 高亮当前行             │
└──────────────────────────────┘
    │
    ▼
等待用户操作：
├─ Step Over  (F6)
├─ Step Into  (F5)
├─ Step Out   (F7)
├─ Continue   (F8)
└─ Resume     (F9)
```

### 6.3 自上而下调试与自底向上调试的组合流程

```
┌──────────────────────────────────────────────────┐
│  发现问题现象                                    │
│  （如查询返回空）                                │
└──────────┬───────────────────────────────────────┘
           │
           ▼
    ┌──────────────┐
    │ 熟悉代码？   │
    └──┬─────────┬──┘
       │是      │否
       │        │
       ▼        ▼

┌──────────────────────┐  ┌───────────────────────┐
│ 自上而下调试         │  │ 自底向上调试          │
│                      │  │                       │
│ 1. 在入口打断点      │  │ 1. 定位底层方法      │
│    (business layer)  │  │    (JDBC layer)      │
│                      │  │                      │
│ 2. 单步执行          │  │ 2. 打条件断点        │
│                      │  │                      │
│ 3. 观察变量          │  │ 3. 观察调用栈       │
│    状态变化          │  │                      │
│                      │  │ 4. 逐层向上分析     │
│ 4. 逐步深入          │  │                      │
│    找到问题          │  │ 5. 找到合适入口     │
│                      │  │                      │
└──────────┬───────────┘  └─────────────┬────────┘
           │                           │
           │                           ▼
           │              ┌─────────────────────┐
           │              │ 切换回自上而下法   │
           │              │ 深入代码细节        │
           │              └─────────────┬────────┘
           │                           │
           └─────────────┬─────────────┘
                         │
                         ▼
           ┌──────────────────────────┐
           │ 找到根本原因              │
           │ (Root Cause)            │
           └──────────────────────────┘
                    │
                    ▼
           ┌──────────────────────────┐
           │ 修复 Bug                │
           │ 验证修复                │
           └──────────────────────────┘
```

### 6.4 HotSwap 类重定义流程

```
┌──────────────────────────────────────────────────┐
│         HotSwap 类重定义流程                     │
└──────────────────────────────────────────────────┘

调试中发现 Bug
    │
    ▼
┌──────────────────────┐
│ 修改源代码           │
│ (本地 IDE)          │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Ctrl+Shift+F9        │
│ (重新编译)           │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ 编译器生成新字节码   │
│ (.class 文件)        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────────────┐
│ IDEA 通过 JDWP 发送          │
│ 类重定义请求                 │
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ JVM 接收请求                 │
│ - 验证修改合法性             │
│   ✓ 方法体修改 OK           │
│   ✓ 添加局部变量 OK         │
│   ✗ 添加/删除方法 FAIL      │
└──────────┬───────────────────┘
           │
        ┌──┴──┐
        │     │
        ▼ OK  ▼ FAIL
        │     │
        │     ▼
        │  ┌──────────────────┐
        │  │ 提示用户         │
        │  │ 不支持的修改     │
        │  │ 需要重启 JVM    │
        │  └──────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ JVM 替换旧类字节码           │
│ (立即生效)                   │
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ IDE 通知加载完成             │
│ 调试继续（无需重启）         │
└──────────────────────────────┘
```

---

## 7. 速查表

### 7.1 快捷键速查表

**IDEA 默认快捷键**（Eclipse 风格可配置，此为 IDEA 原生风格）

| 功能 | 快捷键 | 备注 |
|------|--------|------|
| **调试启动 / 控制** |
| Debug 模式启动 | Shift+F9 | 启动调试 |
| Resume（继续） | F9 | 继续执行到下一断点 |
| Step Over（下一行） | F10 | 执行当前行 |
| Step Into（进入函数） | F11 | 进入函数内部 |
| Step Out（退出函数） | Shift+F8 | 执行完函数后退出 |
| Run to Cursor（运行到光标） | Ctrl+Alt+F10 | 快速跳转到光标位置 |
| Force Return（强制返回） | Ctrl+Shift+F8 | 立即返回（在栈帧上） |
| **变量 / 表达式** |
| Evaluate Expression（求值） | Alt+F9 | 计算任意表达式 |
| Add to Watches（添加监视） | Ctrl+Shift+F9 | 将表达式添加到监视窗口 |
| **断点相关** |
| Toggle Breakpoint（打/删断点） | Ctrl+F8 | 在当前行打/删断点 |
| View Breakpoints（查看断点） | Ctrl+Shift+F8 | 打开断点窗口 |
| Mute Breakpoints（禁用所有断点） | Ctrl+Alt+Shift+F8 | 临时禁用所有断点 |

### 7.2 JDBC 核心方法对照表

| JDBC 方法 | 调试用途 | 典型应用 |
|----------|---------|--------|
| `DataSource.getConnection()` | 追踪连接池 | 连接泄漏、连接等待 |
| `Connection.prepareStatement()` | 观察 SQL 语句和参数 | SQL 注入、语句错误 |
| `PreparedStatement.setXXX()` | 验证参数绑定 | 参数类型错误、NULL 处理 |
| `PreparedStatement.execute()` | 追踪 SQL 执行 | SQL 执行超时、死锁 |
| `Statement.getResultSet()` | 获取结果集 | 结果集为 NULL |
| `ResultSet.next()` | 逐行遍历 | 查询无结果、结果映射错误 |
| `ResultSet.getXXX()` | 提取列值 | 类型转换错误、列不存在 |

### 7.3 常见问题速查

| 问题现象 | 可能原因 | 调试方法 |
|---------|--------|--------|
| 查询返回空值 | 1. SQL 语法错误<br>2. 条件不匹配<br>3. 字段映射错误 | 在 ResultSet.next() 打条件断点 |
| OOM 异常 | 1. 内存泄漏<br>2. 集合无限增长<br>3. 大对象分配 | 使用 JFR 或 jmap 分析堆 |
| NPE 异常 | 1. 未初始化<br>2. 方法返回 null<br>3. 数据库数据缺失 | 设置 NPE 异常断点 |
| 数据不一致 | 1. 并发修改<br>2. 事务隔离<br>3. 缓存同步 | 追踪修改调用栈、观察变量变化 |
| 性能下降 | 1. 死循环<br>2. 锁竞争<br>3. 频繁 GC | 使用 JFR 或 jstack 分析 |

### 7.4 调试技巧对照表

| 技巧 | 应用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **行/方法/异常/字段/条件断点** | 本地开发调试 | 直观、功能完整 | 需要修改代码 |
| **自上而下调试** | 业务代码追踪 | 逻辑清晰 | 需要熟悉代码 |
| **自底向上调试** | 框架代码分析 | 快速定位层级 | 需要了解关键方法 |
| **表达式求值** | 快速计算结果 | 灵活、无需改代码 | 可能有副作用 |
| **Drop Frame** | 函数调试回退 | 可重新执行函数 | 功能受限、不常用 |
| **HotSwap** | 快速修复 Bug | 无需重启 JVM | 修改类型受限 |
| **远程调试** | 生产环境诊断 | 在实际环境调试 | 安全风险、性能影响 |
| **日志打点** | 异步/生产诊断 | 低开销、可持久化 | 需要预埋代码 |
| **jdb** | 服务器调试 | 无需 IDE | 体验不佳 |
| **JFR** | 长期性能监控 | 开销极小、功能强 | 需要后处理分析 |

---

## 8. 原文勘误

### 8.1 快捷键标注不一致

**问题**：原文在幻灯片 09 中展示了多种快捷键配置（Eclipse、IDEA 等），但未明确说明当前使用的是哪套标准。

**建议**：
- 明确说明不同 IDE 的快捷键差异
- 建议用户在 IDE 设置中查看当前配置
- 或提供快捷键修改指南

### 8.2 无异常断点的详细配置说明

**问题**：原文仅展示了异常断点的截图，未详细说明：
- 异常捕获时机（Throw vs Catch）
- 多个异常的同时配置
- 异常断点的过滤条件

**建议**：补充配置步骤和参数说明

### 8.3 缺少 Stream/Lambda 调试的实际示例

**问题**：原文中 20 章未涉及 Stream 和 Lambda 的调试技巧，这是现代 Java 代码的常见场景。

**建议**：补充章节说明：
- Lambda 表达式中的断点行为
- Stream 终端操作与中间操作的断点区别
- 使用 peek() 进行中间观察

### 8.4 缺少远程调试的安全建议

**问题**：原文虽有远程调试的基本说明，但缺少安全性警告。

**建议**：强调：
- 不要在生产公网暴露调试端口
- 使用 SSH 隧道的安全连接方法
- JDWP 认证机制（如有）

### 8.5 缺少 HotSwap 的支持边界说明

**问题**：原文未明确说明 HotSwap 支持和不支持的修改类型。

**建议**：明确列表：
- ✓ 支持：方法体、局部变量、字段初始值
- ✗ 不支持：方法签名、字段声明、类层级修改

---

## 附录：学习路线建议

### 初级阶段
1. 掌握基础断点（行、条件）
2. 熟练单步调试流程
3. 学会查看变量和调用栈

### 中级阶段
1. 理解自上而下与自底向上调试法
2. 学会设置异常断点
3. 掌握表达式求值

### 高级阶段
1. 远程调试的配置和应用
2. 使用 JFR 进行性能分析
3. 理解 JDWP 协议
4. 掌握 HotSwap 动态修改

### 实战建议
- 在实际项目中练习每种技巧
- 记录每次调试的成功和失败原因
- 定期复盘调试过程，积累经验
- 学习他人的调试技巧分享

---

**文档生成日期**: 2024 年  
**覆盖技巧**: IDEA 基础调试、自上而下、自底向上、远程调试、HotSwap、日志打点、JFR 等  
**目标应用**: Java 开发人员学习和参考

