# python22 async/await · 知识点梳理

> 原文档:`python/Article/PythonBasis/python22/1.md`
> 整理对象:`async def`、`await`、事件循环、`asyncio.gather`、`Task`、`TaskGroup`(3.11+)、`Semaphore`、`asyncio.to_thread`、httpx 并发示例

---

## 一、核心概念

```python
import asyncio

# 定义协程函数
async def fetch(url: str) -> str:
    print(f"fetching {url}")
    await asyncio.sleep(1)        # 模拟 IO 等待，交出控制权
    return f"result of {url}"

# 顶层入口
async def main():
    result = await fetch("https://example.com")

asyncio.run(main())   # 创建事件循环，运行 main，清理
```

| 术语 | 含义 |
|------|------|
| **协程函数** | `async def` 定义的函数，调用它返回一个 coroutine 对象 |
| **协程对象** | 调用协程函数的返回值，必须用 `await` 驱动或包装为 `Task` |
| **await** | 暂停当前协程，等待 awaitable 完成，把控制权交还给事件循环 |
| **事件循环** | 单线程调度器，管理所有协程的暂停/恢复 |
| **Task** | 将协程包装为 Task 后事件循环开始调度它(并发执行) |

> ⚠️ 关键概念:`await` 不是"阻塞等待"，而是"在此处暂停，让事件循环去做其他事情，等结果准备好了再恢复我"。**IO 密集任务的并发用 asyncio；CPU 密集用 multiprocessing。**

## 二、并发执行

### 串行(慢)

```python
async def main():
    r1 = await fetch(url_a)   # 等 a 完成
    r2 = await fetch(url_b)   # 再等 b 完成
    # 总时间 = 2 秒
```

### `gather` 并发(快)

```python
async def main():
    r1, r2 = await asyncio.gather(
        fetch(url_a),
        fetch(url_b),
    )
    # 总时间 ≈ 1 秒(并发)
```

### `create_task` + `gather`(推荐，任务管理清晰)

```python
async def main():
    task_a = asyncio.create_task(fetch(url_a))
    task_b = asyncio.create_task(fetch(url_b))
    # task 已提交给事件循环，此时 fetch_a 和 fetch_b 已经在跑
    r1 = await task_a
    r2 = await task_b
```

`create_task` 将协程包装为 `Task` 并立即提交到事件循环。此时协程开始执行(直到遇到第一个 `await`)。

### `TaskGroup`(3.11+，推荐用于批量任务)

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(fetch("a"))
        t2 = tg.create_task(fetch("b"))
        t3 = tg.create_task(fetch("c"))
    # 退出 async with 时自动等待所有 task 完成
```

`TaskGroup` 的优势:
- 任何一个 task 失败，其他 task 自动取消。
- 多个 task 同时失败时，用 `ExceptionGroup` / `except*` 收集所有异常。

## 三、超时控制

```python
# 3.10 及之前
await asyncio.wait_for(fetch("url"), timeout=5.0)

# 3.11+: asyncio.timeout 上下文(更灵活)
async with asyncio.timeout(5.0):
    result = await fetch("url")
```

## 四、取消任务

```python
task = asyncio.create_task(fetch("url"))
task.cancel()
try:
    await task
except asyncio.CancelledError:
    print("任务已取消")
```

- `task.cancel()` 在协程的下一个 `await` 点注入 `CancelledError`。
- 捕获 `CancelledError` 后应做清理并重新 `raise`(或让它自然传播)。

## 五、信号量(并发限流)

```python
sem = asyncio.Semaphore(5)   # 最多同时 5 个

async def limited_fetch(url):
    async with sem:
        return await fetch(url)

# 100 个 URL，但最多 5 个并发
results = await asyncio.gather(*[limited_fetch(u) for u in urls])
```

## 六、在异步中运行同步代码

```python
# 在线程池中运行阻塞函数
result = await asyncio.to_thread(some_blocking_function, arg1, arg2)
```

`asyncio.to_thread`(3.9+) 将同步函数交给线程池执行，主事件循环不被阻塞。

## 七、IO 密集型 vs CPU 密集型

| 特征 | 方案 | 适用 |
|------|------|------|
| IO 密集(网络、磁盘) | asyncio | Web 请求、数据库查询、文件操作 |
| CPU 密集(计算) | multiprocessing / concurrent.futures | 图像处理、数学计算、数据转换 |
| 混合 | asyncio + `to_thread` / `run_in_executor` | 主 IO + 少量 CPU |

> ⚠️ 原文差异1
> `time.sleep()` 是同步阻塞，在 async 函数中会卡死整个事件循环。**必须用 `await asyncio.sleep()`**。

## 八、完整示例:httpx 并发 50 个 URL

```python
import asyncio
import httpx

async def fetch_one(client: httpx.AsyncClient, url: str) -> dict:
    resp = await client.get(url)
    return {"url": url, "status": resp.status_code}

async def main():
    urls = [f"https://httpbin.org/delay/1?n={i}" for i in range(50)]
    async with httpx.AsyncClient(timeout=10) as client:
        sem = asyncio.Semaphore(10)
        async def limited(url):
            async with sem:
                return await fetch_one(client, url)
        results = await asyncio.gather(*[limited(u) for u in urls])
    return results

asyncio.run(main())   # 50 个请求 ~5s(10并发 × 5轮 × 1s)
```

---

## 九、事件循环调度流(ASCII 图)

```
asyncio 事件循环调度示意:

    ┌────────────────────────────────────────────────┐
    │                Event Loop (单线程)               │
    │                                                │
    │   ┌──────────┐   ┌──────────┐   ┌──────────┐  │
    │   │ Task A   │   │ Task B   │   │ Task C   │  │
    │   │ fetch(a) │   │ fetch(b) │   │ fetch(c) │  │
    │   └────┬─────┘   └────┬─────┘   └────┬─────┘  │
    │        │              │              │         │
    │   await client.get()  │         await asyncio   │
    │   (IO 等待，挂起)     │              .sleep()  │
    │        │              │              │         │
    │        ▼              ▼              ▼         │
    │   ┌──────────────────────────────────────┐     │
    │   │        IO 多路复用 (selector)         │     │
    │   │  监听所有 socket / 定时器 / 信号     │     │
    │   │  哪个 ready 就唤醒哪个 Task          │     │
    │   └──────────────────────────────────────┘     │
    │                                                │
    │   时间轴:                                       │
    │   t0: Task A await → 挂起，切换到 B             │
    │   t0: Task B await → 挂起，切换到 C             │
    │   t0: Task C await → 挂起，无事可做，等 IO      │
    │   t1: 某个 socket ready → 恢复对应 Task          │
    └────────────────────────────────────────────────┘


create_task vs await 对比:

    async def main():
        t1 = asyncio.create_task(fetch("a"))   ← 提交到事件循环，t1 开始执行
        t2 = asyncio.create_task(fetch("b"))   ← 提交到事件循环，t2 开始执行 (t1 可能在 await 处挂起了)
        r1 = await t1                          ← 等待 t1 完成(同时 t2 也在跑)
        r2 = await t2                          ← 等待 t2 完成
        # t1 和 t2 并发执行

    async def main():
        r1 = await fetch("a")                  ← 执行 fetch("a")，在 await 处挂起
        r2 = await fetch("b")                  ← fetch("a") 完成后才开始执行
        # t1 和 t2 串行执行
```

---

## 十、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 协程函数调用后未 await 只是 warning | 3.11+ 对"coroutine was never awaited"的处理更严格 |
| 2 | 1.md | 未区分 `gather` vs `create_task` vs `TaskGroup` | TaskGroup(3.11+) 是结构化并发的推荐方式 |
| 3 | 1.md | `asyncio.wait_for` 的超时处理未提 CancelledError | 超时后会取消 task 并抛出 `TimeoutError`(原 task 收到 CancelledError) |
| 4 | 1.md | 未提 `asyncio.to_thread`(3.9+) | 异步中调用同步阻塞函数的官方方案 |
| 5 | 1.md | 未强调 `time.sleep` 在 async 中会阻塞整个事件循环 | 这是初学 asyncio 的最高频陷阱 |
