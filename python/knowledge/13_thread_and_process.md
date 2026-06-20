# python13 线程与进程 知识点梳理

> 原文档:`python/Article/PythonBasis/python13/` 下 `1.md ~ 3.md`
> 整理对象:进程/线程概念、GIL、threading、multiprocessing、Pool、Queue、async(指引)
> **本章是多文件多组件协作场景,会画进程/线程之间数据流的 ASCII 图**

---

## 一、进程 vs 线程 vs 协程

### 1. 一句话定义

| 类型 | 调度方 | 内存空间 | 切换成本 | Python 中的关键问题 |
|------|--------|----------|----------|---------------------|
| 进程(process) | 操作系统 | 各自独立 | 高 | 没有 GIL 互相干扰,真正并行 |
| 线程(thread) | 操作系统 | 同进程内共享 | 中 | **GIL** 让 CPU 密集型多线程几乎无加速 |
| 协程(coroutine) | 用户层(asyncio 调度器) | 共享 | 极低 | 单线程内 N 个并发任务,IO 密集型首选 |

### 2. GIL(全局解释器锁) —— 必须知道

CPython 在任何时刻 **只允许一个线程执行 Python 字节码**。即使你启了 8 个线程跑纯计算,也跑不过单线程。

- IO 密集型(网络/磁盘/sleep) → 多线程 **能加速**(IO 期间会释放 GIL)。
- CPU 密集型(图像处理、数值计算) → 必须用 **多进程** 或 C 扩展才能用满核心。
- Python 3.13(2024+) 开始实验性提供 **无 GIL 构建**(PEP 703),但 2026 年绝大多数生态仍跑在带 GIL 的解释器上。

---

## 二、多线程 threading

### 1. 模块说明

> ⚠️ 原文错误一(`2.md` 第 22 行)
> 原文写"Python 提供两个模块进行多线程的操作,分别是 `thread` 和 `threading`"。
> **正确说法**:Python 3 中底层模块叫 `_thread`(下划线开头),不是 `thread`。**绝大多数情况下都应该用 `threading`**,只在写底层基础设施时才考虑 `_thread`。

### 2. 创建线程

#### 方式 A:函数式(推荐)

```python
import threading, time

def worker(i):
    print(f'worker {i} start')
    time.sleep(1)
    print(f'worker {i} done')

t = threading.Thread(target=worker, args=(1,))
t.start()
t.join()                      # 等子线程结束
```

#### 方式 B:子类化

```python
class MyThread(threading.Thread):
    def __init__(self, n): super().__init__(); self.n = n
    def run(self):
        for i in range(self.n): print(self.name, i)
```

### 3. 线程的关键方法/属性

| 项 | 作用 |
|----|------|
| `t.start()` | 启动并进入 run() |
| `t.join(timeout=None)` | 阻塞当前线程,等 t 结束 |
| `t.is_alive()` | 是否还在运行 |
| `t.daemon = True` | **必须在 start() 之前设置**;主线程退出时守护线程立即终止 |
| `threading.current_thread()` | 当前线程对象 |
| `threading.active_count()` | 当前活跃线程数 |

> ⚠️ 原文错误二(`2.md` §6 / `3.md` §3)
> 原文用 `setDeamon(True)`。
> **正确说法**:首先关键字拼写应为 **daemon**,其次 `setDaemon()` 在 Python 3.10 起 **已废弃**,推荐 **直接给 `daemon` 属性赋值**:`t.daemon = True`。同样 `getName/setName` 也建议改用 `t.name` 属性。

### 4. 同步原语

| 原语 | 用途 |
|------|------|
| `threading.Lock` | 互斥锁,**同一线程重复 acquire 会死锁** |
| `threading.RLock` | 可重入锁,同线程 N 次 acquire 需 N 次 release |
| `threading.Semaphore(n)` | 计数信号量,限并发数 |
| `threading.Event` | 一发布多订阅的事件标志 |
| `threading.Condition` | 配合谓词的等待/通知,适合生产消费 |
| `threading.Barrier(n)` | n 个线程都到了再放行 |

#### 锁的安全写法(`with` 自动 release)

```python
lock = threading.Lock()
with lock:                # __enter__ acquire,__exit__ release
    shared_state += 1
```

> ⚠️ 原文错误三(`2.md` §3)
> 原文用 `lock.acquire()` / `lock.release()` 但 **没有用 `with`**,这在异常发生时会忘记 release 而死锁。**最佳实践应改为 `with lock:`**。

> ⚠️ 原文错误四(`2.md` §5 Event 例子)
> 原文用 `event.isSet()`、`self.getName()`。
> **正确说法**:Python 3.10+ 推荐 `event.is_set()`(下划线 PEP 8 风格),`self.name` 属性;原驼峰写法是 Python 2 的旧名,虽然仍可用但已不推荐。

### 5. 线程间通信:`queue.Queue`

```python
from queue import Queue
from threading import Thread

q = Queue(maxsize=10)            # 内置锁,线程安全

def producer():
    for x in items:
        q.put(x)
    q.put(None)                  # 哨兵,通知消费者结束

def consumer():
    while True:
        x = q.get()
        if x is None: q.task_done(); break
        process(x)
        q.task_done()

Thread(target=producer).start()
Thread(target=consumer).start()
q.join()                         # 等所有 put 的项都 task_done
```

`queue.Queue` 内部已经做好了锁和条件变量,**几乎所有线程间数据传递都该用它**。

#### ASCII:生产者-消费者数据流

```
       ┌────────────┐                  ┌────────────┐
       │ Producer T1│ ── put(item) ──▶ │ Queue(线程  │ ── get() ──▶┌───────────┐
       └────────────┘                  │ 安全,内置锁) │              │ Consumer T2│
       ┌────────────┐                  │            │              └───────────┘
       │ Producer T3│ ── put(item) ──▶ │  | item3   │
       └────────────┘                  │  | item2   │
                                       │  | item1   │              ┌───────────┐
                                       │            │ ── get() ──▶ │ Consumer T4│
                                       └────────────┘              └───────────┘

  - put/get 都在锁保护下,无需用户加锁
  - 阻塞:队列空时 get 阻塞;队列满时 put 阻塞
  - 哨兵(None)是常用的"结束信号"
```

---

## 三、多进程 multiprocessing

### 1. 创建进程

```python
import multiprocessing as mp

def worker(name):
    print(f'{name} start')

if __name__ == '__main__':       # 必须放 main 守护,否则 Windows 会无限递归启动子进程
    p = mp.Process(target=worker, args=('a',))
    p.start()
    p.join()
```

### 2. 进程池 Pool

```python
from multiprocessing import Pool

def square(x): return x * x

if __name__ == '__main__':
    with Pool(processes=4) as pool:
        results = pool.map(square, range(10))
```

进阶接口:`pool.apply_async`、`pool.imap_unordered`,适合海量任务流式处理。

更现代的替代:`concurrent.futures.ProcessPoolExecutor`,统一了线程池与进程池接口:

```python
from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor(max_workers=4) as ex:
    for r in ex.map(square, range(10)):
        print(r)
```

### 3. 进程间通信

| 工具 | 适用 |
|------|------|
| `multiprocessing.Queue` | 多生产多消费的安全队列 |
| `multiprocessing.Pipe` | 两端点的双向管道 |
| `multiprocessing.Manager().dict/list` | 共享可变状态(慢但通用) |
| `multiprocessing.shared_memory`(3.8+) | 大块二进制数据共享,零拷贝 |
| `multiprocessing.Value/Array` | 简单数值/数组共享 |

#### ASCII:多进程协作

```
                  ┌───────────────────┐
                  │  Main Process     │
                  │   (创建 Queue q)   │
                  └───────────────────┘
                          │
            spawn          │           spawn
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
  ┌──────────┐      ┌──────────┐      ┌──────────┐
  │ Worker P1│      │ Worker P2│      │ Reader P3│
  │ put 到 q │      │ put 到 q │      │ get 从 q │
  └──────────┘      └──────────┘      └──────────┘
        │                 │                 │
        └─────────┬───────┘                 │
                  ▼                         │
        ┌─────────────────────────┐         │
        │  mp.Queue(进程安全)      │ ◀───────┘
        │  内部用 OS 管道+序列化   │
        └─────────────────────────┘

注意:跨进程传输需要 **pickle 序列化**,所以不可 pickle 的对象
     (lambda、文件句柄、套接字)不能直接放队列。
```

### 4. fork vs spawn

- macOS(从 3.8 起)与 Windows 默认 **spawn**(全新解释器,加载 main 模块,需 `if __name__ == '__main__':` 守护)。
- Linux 默认 **fork**(更快,但与多线程混用易死锁)。
- 显式选择:`mp.set_start_method('spawn')`。

### 5. 子进程死锁的常见坑

- `pool.join()` 之前 **必须** `pool.close()` 或 `pool.terminate()`。
- 主进程意外退出(异常)会留下僵尸子进程,建议 `with Pool() as p:` 由上下文管理器收尾。

---

## 四、协程 asyncio(引子)

详见 `python22 async/await 与并发`,这里只说本章未提的对比:

- IO 密集型推荐顺序:`asyncio` > 多线程 > 多进程。
- CPU 密集型推荐顺序:多进程 > C 扩展 > asyncio 跑死。

---

## 五、Condition 生产消费(原文例子的修正版)

> ⚠️ 原文错误五(`2.md` §4 Condition 例子)
> 原文用 `notify` + `wait` 串起两个线程的"对话",但 **没有用谓词**(while loop check),严格意义上是"虚假唤醒不安全"的。
> 推荐写法:

```python
from threading import Condition, Thread

cond = Condition()
queue = []

def producer():
    for x in range(10):
        with cond:
            queue.append(x)
            cond.notify()           # 唤醒一个等待者

def consumer():
    while True:
        with cond:
            while not queue:        # 谓词,防止虚假唤醒
                cond.wait()
            x = queue.pop(0)
        process(x)
```

工程上 **能用 `queue.Queue` 就用,不要自己玩 Condition**。

---

## 六、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 2.md §1 | 模块叫 `thread` 和 `threading` | Python 3 底层叫 `_thread`;一般直接用 `threading` |
| 2 | 2.md §6 / 3.md §3 | `setDeamon(True)` | 拼写 daemon,且应直接写 `t.daemon = True`;`setDaemon` 3.10+ 已废弃 |
| 3 | 2.md §3 | 手动 acquire/release | 应 `with lock:` 防异常死锁 |
| 4 | 2.md §5 | `event.isSet()` / `getName` | Python 3 推荐 `event.is_set()` / `t.name` |
| 5 | 2.md §4 | Condition 例子缺谓词 while | `cond.wait()` 必须包在 `while not 条件:` 中防虚假唤醒 |
| 6 | 全章 | 没提 GIL | Python 多线程绕不开的话题,应作为第一条原理介绍 |
| 7 | 全章 | 没提 `concurrent.futures` | 现代统一接口,应当作为推荐 |
| 8 | 全章 | 没提 fork/spawn 差异 | 跨平台代码必须了解,否则 macOS/Windows 上死锁/无限递归 |

> 本章是多组件、多文件协作场景,已用 ASCII 数据流图展示线程间 Queue 通信、多进程 Queue 通信。
