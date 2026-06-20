# 并发编程知识点梳理

> 基于 `concurrent.md` 整理。内容按线程池、Fork/Join、原子操作、AQS、并发容器、线程协同、JMM、volatile、synchronized、ThreadLocal 归类。涉及多类/多方法协作的流程用 ASCII 图说明。

---

## 目录

1. [线程与线程池](#一线程与线程池)
2. [Fork/Join](#二forkjoin)
3. [原子操作与 CAS](#三原子操作与-cas)
4. [AQS 与自定义锁](#四aqs-与自定义锁)
5. [并发容器](#五并发容器)
6. [线程协同工具](#六线程协同工具)
7. [线程三大特性与 JMM](#七线程三大特性与-jmm)
8. [volatile](#八volatile)
9. [synchronized](#九synchronized)
10. [ThreadLocal](#十threadlocal)

---

## 一、线程与线程池

### 1.1 线程创建方式

- 继承 `Thread`
- 实现 `Runnable`

### 1.2 线程状态

- `NEW`：新建未启动
- `RUNNABLE`：可运行
- `BLOCKED`：等待锁
- `WAITING`：无限期等待
- `TIMED_WAITING`：限时等待
- `TERMINATED`：结束

### 1.3 线程池价值

- 复用线程，降低创建/销毁成本
- 提高响应速度
- 控制并发数量，避免无限创建线程导致 OOM
- 减少 CPU 上下文切换成本
- 支持定时/调度等扩展能力

### 1.4 ThreadPoolExecutor 核心参数

| 参数 | 作用 |
| --- | --- |
| `corePoolSize` | 核心线程数 |
| `maximumPoolSize` | 最大线程数 |
| `keepAliveTime` | 空闲线程存活时间 |
| `workQueue` | 任务队列 |
| `threadFactory` | 线程工厂 |
| `handler` | 拒绝策略 |

### 1.5 execute 执行流程

```text
submit task
    |
    v
check current worker count
    |
    +-- less than corePoolSize --> addWorker(task, core=true) --> start thread
    |
    +-- else if running --> offer task to workQueue
    |                       |
    |                       +-- queue accepted --> maybe add empty worker if none
    |
    +-- queue full --> addWorker(task, core=false)
                           |
                           +-- success --> start thread
                           +-- fail    --> reject task
```

### 1.6 getTask 取任务流程

```text
worker loop
    |
    v
task exists?
    |
    +-- yes -> run task
    |
    +-- no  -> getTask()
                 |
                 +-- core thread or not timed out -> workQueue.take()
                 |
                 +-- non-core / timed -> workQueue.poll(keepAliveTime)
                 |
                 +-- timeout and eligible -> worker count--
```

### 1.7 常见线程池

- `newCachedThreadPool()`：弹性线程池
- `newFixedThreadPool(int)`：固定线程池
- `newSingleThreadExecutor()`：单线程池
- `newScheduledThreadPool(int)`：定时调度线程池

### 1.8 线程池面试要点

- 核心线程并不会因为空闲立即销毁，而是在 `getTask()` 中阻塞等待任务。
- 线程池中的线程状态常见为 `RUNNABLE` 和 `WAITING`。
- 核心线程与非核心线程本质上没有特殊身份差异，销毁与否取决于 `getTask()` 的获取策略。

---

## 二、Fork/Join

### 2.1 概念

Fork/Join 用于把大任务拆分成小任务并行执行，再合并结果。

### 2.2 组成

- `ForkJoinPool`：执行任务的线程池
- `ForkJoinTask`：任务抽象
- `RecursiveTask`：有返回值任务
- `RecursiveAction`：无返回值任务
- `ForkJoinWorkerThread`：工作线程
- `WorkQueue`：每个工作线程绑定的双端队列

### 2.3 执行流程

```text
large task
    |
    v
fork split into subtasks
    |
    +--> worker 1 deque
    +--> worker 2 deque
    +--> worker 3 deque
    |
    v
each worker executes local deque in LIFO order
    |
    v
if idle -> steal from others in FIFO order
    |
    v
join merge results
```

### 2.4 设计思想

- 普通线程池是一个共享队列。
- Fork/Join 是每个 worker 一个双端队列。
- 本地任务优先后进先出执行，空闲线程通过 work-stealing 窃取别人的任务。

---

## 三、原子操作与 CAS

### 3.1 原子操作

原子操作是不可被中断的一个或一系列操作。

### 3.2 CAS

CAS = Compare And Swap，用“比较旧值并替换新值”的方式实现无锁并发控制。

### 3.3 Atomic 包

- `AtomicBoolean`
- `AtomicInteger`
- `AtomicLong`
- `AtomicReference`
- `AtomicIntegerArray`
- `AtomicStampedReference`

### 3.4 CAS 的问题

- 自旋开销大
- 只能保证单个共享变量原子性
- 存在 ABA 问题

### 3.5 原子性边界

CAS 只能保证它覆盖的那一步是原子的，后续再做普通赋值就可能破坏整体原子性。

---

## 四、AQS 与自定义锁

### 4.1 为什么要学 AQS

AQS 是 JUC 大量同步器的基础，`ReentrantLock`、`Semaphore`、`CountDownLatch` 等都建立在它之上。

### 4.2 AQS 核心结构

- `state`：同步状态
- `head` / `tail`：等待队列
- `Node`：等待节点，保存线程信息
- `ConditionObject`：条件队列

### 4.3 AQS 支持的模式

- 独占模式
- 共享模式

### 4.4 模板方法

- `acquire / release`
- `acquireShared / releaseShared`
- 由子类实现 `tryAcquire / tryRelease` 等方法

### 4.5 自定义锁执行流程

```text
lock()
  |
  v
tryAcquire()
  |
  +-- success --> return
  |
  +-- fail --> addWaiter() -> enqueue
                     |
                     v
                acquireQueued()
                     |
                     v
               wait until predecessor releases
```

### 4.6 公平锁与非公平锁

- **非公平锁**：线程来直接抢，不看队列，吞吐高但可能插队。
- **公平锁**：先看队列里是否有人排队，有就老实排队。

### 4.7 可重入锁

同一线程再次获取已持有的锁时，不应被阻塞，而是增加重入次数；释放时按次数递减，直到 `state = 0` 才真正释放。

---

## 五、并发容器

常见并发容器：

- `ConcurrentHashMap`
- `CopyOnWriteArrayList`
- `CopyOnWriteArraySet`
- `ConcurrentSkipListMap`
- `ConcurrentSkipListSet`
- `ConcurrentLinkedQueue`
- `BlockingQueue`

核心思路：

- 读写分离
- 分段/分桶/无锁化
- 写时复制
- 有序并发结构

---

## 六、线程协同工具

### 6.1 Object.wait / notify / notifyAll

- `wait()`：释放锁并进入等待
- `notify()`：唤醒一个等待线程
- `notifyAll()`：唤醒所有等待线程

### 6.2 LockSupport.park / unpark

用于更底层的线程阻塞与唤醒，AQS 内部大量使用。

### 6.3 Thread.sleep / yield / join

- `sleep()`：暂停执行，不释放锁
- `yield()`：让出 CPU 竞争权，不释放锁
- `join()`：当前线程等待目标线程结束

### 6.4 协同流程图

```text
wait/notify:
thread A holds lock
    |
    v
wait() -> release lock + block
    |
    v
thread B gets lock -> notify/notifyAll
    |
    v
thread A wakes up and competes for lock again
```

```text
join:
main thread
    |
    v
start sub thread
    |
    v
join()
    |
    v
main waits until sub thread terminates
```

---

## 七、线程三大特性与 JMM

### 7.1 三大特性

- **可见性**：一个线程修改，另一个线程能及时看到
- **有序性**：程序执行顺序不一定等于代码顺序
- **原子性**：操作不可被中断

### 7.2 根源

- CPU 缓存
- 线程切换
- 编译器与处理器重排序

### 7.3 JMM

JMM 规定了主内存和工作内存之间的数据交互规则，用来解决可见性和有序性问题。

### 7.4 主内存与工作内存

- 主内存保存共享变量
- 每个线程有自己的工作内存
- 线程对变量的读写必须先在工作内存中进行，再与主内存交互

### 7.5 JMM 的 8 种交互操作

- `lock`
- `unlock`
- `read`
- `load`
- `use`
- `assign`
- `store`
- `write`

核心约束：

- `read/load`、`store/write` 必须成对出现
- 工作内存变更后需要同步回主内存
- `lock/unlock` 必须成对

### 7.6 Happens-Before

核心规则包括：

- 程序顺序规则
- volatile 规则
- 锁规则
- 线程启动规则
- 线程 join 规则
- 线程中断规则
- 对象终结规则

### 7.7 顺序性与重排序

- 编译器会做指令重排序优化
- CPU 可能乱序执行
- 单线程下要求 as-if-serial
- 多线程下需要借助 volatile、锁等手段约束可见性和顺序性

---

## 八、volatile

### 8.1 作用

- 保证可见性
- 禁止部分指令重排序

### 8.2 volatile 写读流程

```text
write volatile:
thread working memory
    |
    v
change volatile copy
    |
    v
flush to main memory immediately
```

```text
read volatile:
main memory
    |
    v
load latest value into working memory
    |
    v
thread reads local copy
```

### 8.3 重排序约束

- volatile 写之前的普通操作，不能被重排序到 volatile 写之后。
- volatile 读之后的普通操作，不能被重排序到 volatile 读之前。
- DCL 单例中的 `instance` 需要使用 `volatile`，防止对象引用发布早于对象初始化完成。

### 8.4 局限

- 不保证复合操作原子性
- 只能解决“看得见”，不能自动解决“做得对”
- 修饰引用类型时，只保证引用本身可见，不保证被引用对象内部字段都可见

### 8.5 典型用途

- 状态标志
- 单例发布
- 配合 CAS 或锁实现并发控制

---

## 九、synchronized

### 9.1 基本作用

- 同步代码块/方法
- 保证原子性、可见性、有序性

### 9.2 锁的对象

- 实例方法锁的是对象实例
- 静态方法锁的是 Class 对象
- 代码块锁的是传入对象

### 9.3 锁模型与内核机制

- synchronized 是 Java 管程实现
- 底层互斥最终依赖操作系统 Mutex
- 线程切换不会释放锁
- 互斥的核心是“同一时刻只允许一个线程执行临界区”

### 9.4 字节码与对象头

```text
sync block
   |
   v
monitorenter
   |
   v
critical section
   |
   v
monitorexit
```

- 对象锁信息存放在对象头 mark word 中
- mark word 关键位包含无锁、偏向锁、轻量级锁、重量级锁、GC 标记等状态

### 9.5 锁升级

典型过程：

- 无锁
- 偏向锁
- 轻量级锁
- 重量级锁

```text
new object
   |
   v
unlocked / biased ready
   |
   +-- no contention --> biased lock / lightweight lock
   |
   +-- contention appears --> heavyweight lock
```

- 偏向锁：偏向第一个获取锁的线程
- 轻量级锁：CAS 竞争锁记录
- 重量级锁：进入 OS 互斥量，代价更高

### 9.6 其他优化

- 锁消除
- 锁粗化

### 9.7 synchronized 执行链路

```text
Java code
    |
    v
compiler adds monitorenter/monitorexit
    |
    v
JVM runtime manages object mark word
    |
    v
lock acquisition / release / upgrade
```

### 9.8 经验

- 尽量降低锁级别
- 减少锁持有时间
- 减少锁粒度
- 减少加锁/解锁次数
- 读多写少场景可考虑读写锁

---

## 十、ThreadLocal

### 10.1 概念

ThreadLocal 为每个线程提供独立变量副本，避免共享数据竞争。

### 10.2 适用场景

- 线程内上下文传递
- 事务上下文
- 用户信息缓存
- 日期格式化器等线程不安全对象隔离

### 10.3 原理

```text
Thread
  |
  v
ThreadLocalMap
  |
  v
key = ThreadLocal
value = thread-specific data
```

### 10.4 set/get/remove 流程

```text
ThreadLocal.set(value)
    |
    v
Thread.currentThread()
    |
    v
currentThread.threadLocals
    |
    +-- exists -> map.set(this, value)
    |
    +-- null   -> create ThreadLocalMap
```

```text
ThreadLocal.get()
    |
    v
Thread.currentThread()
    |
    v
currentThread.threadLocals
    |
    +-- find Entry by current ThreadLocal key -> return value
    |
    +-- not found -> return initialValue()
```

### 10.5 内存泄漏风险

- `ThreadLocalMap.Entry` 的 key 是 `ThreadLocal` 的弱引用
- key 被 GC 后，value 仍可能留在线程的 `ThreadLocalMap` 中
- `get/set/remove` 会顺带清理部分 key 为 null 的 Entry
- 在线程池中线程长期复用，使用完应主动 `remove()`

### 10.6 注意点

- 记得清理，避免线程池场景下的内存泄漏
- 适合线程封闭，不适合跨线程共享
- 如果多个线程放入同一个引用对象，仍然会共享对象内部状态，隔离会失效

---

## 十一、调优与面试速记

- 线程池：先看 core，再看 queue，再看 max，再看 reject。
- Fork/Join：大任务拆小任务，空闲线程偷别人的任务。
- CAS：无锁高效，但有 ABA 和自旋成本。
- AQS：JUC 的底座，重点看 state、队列、模板方法。
- volatile：解决可见性，不解决复合原子性。
- synchronized：简单可靠，依赖 JVM 级别锁升级机制。
- ThreadLocal：线程隔离，不是线程同步。
