---
author: "Yifan Xu"
handle: "@yifanxu_ephai"
source: "https://x.com/yifanxu_ephai/status/2125473409715097754"
date: 2026-11-25T06:15:24.000Z
type: article
likes: 69
reposts: 9
replies: 4
bookmarks: 34
views: 4225
---
# 让 Agent 给 LayerFS 加 PostgreSQL 翻了车，让我对SQL语句、事务和网络往返有了更深刻的理解

*By Yifan Xu (@yifanxu_ephai)*

![Banner](<让%20Agent%20给%20LayerFS%20加%20PostgreSQL%20翻了车，让我对SQL语句、事务和网络往返有了更深刻的理解.assets/image-01.jpg>)

主包也不是数据库专家。LayerFS 原本只有 SQLite 后端，这次我们保留 SQLite，再加一个 PostgreSQL 后端。[github.com/Ephemeral-AI-Lab/layerfs/tree/layerfs_v2_0_0](https://github.com/Ephemeral-AI-Lab/layerfs/tree/layerfs_v2_0_0)

我原以为存储接口已经有了，业务逻辑也跑通了，让 Agent 照着现有实现接上就行。功能确实很快跑起来了。可性能测试一跑，问题就出来了：一个已经挂载的 Workspace，只改一个文件，提交一次要和 PostgreSQL 来回 **46 次**。

每次往返按 **10 ms** 算，光等网络就要 **460 ms**。这下翻大车了。

## 语句、事务和往返

这两天，我总算对 **statement（语句）** 和 **transaction（事务）** 有了具体的感觉。

一条语句就是一条 SQL 命令。拿转账举例，扣款和入账是两条 UPDATE。事务把这两条语句包在一起，最后用一次 COMMIT 让它们同时生效。

除了数语句和事务，还要数第三样东西：**round trip（网络往返）**。应用发出请求，等到数据库回复，算一次往返。同样两条 SQL，可以发一条、等一条，也可以一起发出去，只等一轮。

**在 SQLite 上，往返几乎不用数。** SQLite 不是一个单独运行的数据库服务，而是一个直接链接进应用进程的库。所谓“发一条 SQL、等结果”，其实就是在同一个进程里调用一次函数：不走网络，不用跨进程通信，也不用等另一个服务排队处理。数据页就在本进程的缓存或操作系统的页缓存里，一次轻量调用通常是微秒级，和 10 ms 的网络往返差了几千倍。所以几十次调用加起来，往往还比不上一次网络往返。

因此，以前给 LayerFS 做 profiling，我们看的多是 SQL 执行次数和 **VM steps**，也就是 SQLite 虚拟机走了多少步，关心的是数据库本身干了多少活。写入时更该关心的是提交了几次，以及写事务占着唯一的写者位置有多久。

**换成 PostgreSQL，还得数往返次数。** PostgreSQL 是独立的服务进程，通常还在另一台机器上，每条 SQL 都要通过网络发过去，再等结果传回来。Agent 把原来的调用顺序原封不动地搬了过去：发一条 SQL，拿到结果，再发下一条。在 SQLite 上几乎不花时间的“等结果”，到这里每次都要付一段网络延迟。如果事务已经拿了锁，其他要用这把锁的写入也只能跟着等。

可以粗略这样算：

**耗时 ≈ 数据库处理时间 + 往返次数 × 单次往返耗时**

同样是一个事务里的十条 UPDATE，只看这十条更新的网络等待：

- **SQLite** 在进程内执行，网络等待为 **0**。
- **PostgreSQL** 发一条、等一条，网络等待为 **10 × 10 ms = 100 ms**。
- **PostgreSQL** 一起发、只等一轮，网络等待为 **1 × 10 ms = 10 ms**。

三种写法都是十条更新、一次提交。时间差全出在客户端中途停下来等了几次。

## LayerFS 怎么从 46 次降到 2 次

**46 → 36：把能一起做的合起来。** 原来先查一次 Branch，再查项目的 epoch，最后又查一次 Branch。现在一条 SELECT 就能从同一个快照里拿到这些信息。另一处改动是把写入通道的检查和 COMMIT 一起发出。数据库照样执行两条命令，客户端只需等一次。

**36 → 24：复用应用手里已有的东西。** 挂载时读过的删除标记可以直接复用，预留了但没用完的 ID 可以留给下一次，刚构建好的对象可以直接拿来验证。这一阶段的 24 次是根据测试计数推算出来的。

**24 → 2：整个写事务一次性流水线发出。** 热路径先用一次查询确认哪些对象已经存在，再把整个写事务流水线发给 PostgreSQL。写 pack、检查 Branch head、记录 Commit、推进 head、提交，数据库按顺序执行，客户端中途不必逐条等回复。只要有一项检查失败，事务就中止。[LayerFS 往返明细与实现](https://github.com/Ephemeral-AI-Lab/layerfs/blob/bc402e62a24dc58123de27fdcc1050616b6f0b96/docs/architecture/engine/store/postgresql.md#commit-round-trips)

这 **2 次**指的是已挂载 Workspace 提交单个文件的热路径，已用 PostgreSQL 服务端日志核对过。刚挂载后的第一次提交，测试里是 **23 次**，因为还要读基础数据、预留 ID、准备语句。第二次提交是 **3 次**，多出的一次用来准备首次用到的语句。[LayerFS 测试断言](https://github.com/Ephemeral-AI-Lab/layerfs/blob/bc402e62a24dc58123de27fdcc1050616b6f0b96/engine/crates/layerfs-store-service/tests/mount_round_trips.rs#L269)

以前让 Agent 加数据库后端，我主要验收结果对不对。现在我还会挑一个真实的小操作，看它**执行了几条 SQL、提交了几次、走了几次网络往返**。按 10 ms RTT 算，这次热路径光网络等待就从 **460 ms** 降到了 **20 ms**。给 LayerFS 换数据库后端，真正费功夫的地方在于重新决定：**应用在哪一步需要停下来等数据库。  
  
球球大家支持一下主包项目，点个小星星⭐️谢谢啦：**  
  
[github.com/Ephemeral-AI-Lab/layerfs/tree/layerfs_v2_0_0](https://github.com/Ephemeral-AI-Lab/layerfs/tree/layerfs_v2_0_0)