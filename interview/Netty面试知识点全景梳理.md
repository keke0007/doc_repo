# Netty 面试知识点全景梳理

> 本文基于本目录下 28 篇面试题目录所归纳。整体思路:从底层 I/O 模型出发,逐步抬升到 Reactor 线程模型,再到 Netty 的核心组件、内存模型、零拷贝、TCP 粘拆包、心跳长连接,最后回到「Netty 为什么高性能」这道总纲题。所有的知识点都以 ASCII 流程图串接,便于快速建立大局观和检索关联点。

---

## 0. 知识点总览(全景图)

下图标注了 Netty 面试中最常考的 9 大模块,以及它们之间的依赖与延伸关系。**箭头方向 = 依赖方向**(下层为上层提供支撑)。

```
                       ┌───────────────────────────────────────┐
                       │  ⑨ Netty 高性能(综合题)               │
                       │  异步 NIO + Reactor + 零拷贝 + 内存池 │
                       └───────────────▲───────────────────────┘
                                       │依赖
        ┌──────────────────────────────┼──────────────────────────────┐
        │                              │                              │
┌───────┴────────┐     ┌───────────────┴───────────┐    ┌─────────────┴────────────┐
│ ④ 核心组件     │     │  ⑥ 内存模型 / ByteBuf     │    │ ⑦ 零拷贝(应用层)         │
│ Bootstrap      │     │  Pooled vs Unpooled       │    │ CompositeByteBuf         │
│ Channel        │     │  Heap vs Direct           │    │ slice / wrap             │
│ EventLoop(Group)│←───│  PoolArena/Chunk/Subpage  │    │ FileRegion(sendfile)     │
│ ChannelPipeline│     │  Recycler 对象池          │    └─────────────▲────────────┘
│ ChannelHandler │     └───────────────▲───────────┘                  │
│ (Context)      │                     │                              │
│ ByteBuf        │                     │依赖                          │依赖
└────▲───────────┘                     │                              │
     │实现                             │                       ┌──────┴───────┐
     │                                 │                       │ OS 零拷贝     │
┌────┴─────────────────────────┐       │                       │ mmap/sendfile│
│ ③ Reactor 线程模型           │       │                       │ SG-DMA       │
│ 单 Reactor 单线程            │       │                       └──────────────┘
│ 单 Reactor 多线程            │       │
│ 主从 Reactor 多线程(Netty)   │       │
└────▲─────────────────────────┘       │
     │依赖                              │
┌────┴──────────────────┐               │     ┌──────────────────────────┐
│ ② I/O 多路复用        │               │     │ ⑧ TCP 粘拆包 / 心跳长连接│
│ select / poll / epoll │               │     │ 4 种解码器               │
└────▲──────────────────┘               │     │ IdleStateHandler         │
     │依赖                              │     │ SO_KEEPALIVE             │
┌────┴──────────────────┐               │     └──────────────────────────┘
│ ① 五种 I/O 模型       │               │
│ BIO / 非阻塞 / IO 复用│               │     ⑤ NIO Bug 与 Selector 重建
│ 信号驱动 / AIO        │               │     (Channel 与 EventLoop 关系)
└───────────────────────┘               │
                                        │
                              所有面试题最终落在这张图里
```

---

## 1. ① 五种 I/O 模型(根基)

### 1.1 一次 I/O 的两个阶段

```
        应用层  │                      内核态
                │
recv()调用 ───▶ │
                │   ① 数据准备:等设备准备好
                │   ② 数据拷贝:内核缓冲区 → 用户缓冲区
   返回 ◀────── │
```

> 阻塞与非阻塞、同步与异步,本质就是描述 **应用程序在这两个阶段是否被挂起**。

### 1.2 五种 I/O 模型对比

```
┌─────────────────┬───────────────┬───────────────┬────────────┐
│ 模型            │ 数据准备阶段  │ 数据拷贝阶段  │ 是否同步   │
├─────────────────┼───────────────┼───────────────┼────────────┤
│ 阻塞 IO         │   阻塞        │    阻塞       │   同步     │
│ 非阻塞 IO       │   轮询        │    阻塞       │   同步     │
│ IO 多路复用     │   阻塞 select │    阻塞       │   同步     │
│ 信号驱动 IO     │   不阻塞      │    阻塞       │   同步     │
│ 异步 IO(AIO)    │   不阻塞      │    不阻塞     │   异步     │
└─────────────────┴───────────────┴───────────────┴────────────┘
```

### 1.3 BIO / NIO / AIO 对照

```
        ┌──────────────────────┐
 BIO  ──▶ 一个连接一个线程     │  Java Stream(InputStream)
        ├──────────────────────┤
 NIO  ──▶ 一个线程多个连接     │  Channel + Buffer + Selector
        ├──────────────────────┤
 AIO  ──▶ 一个有效请求一个线程 │  AsynchronousSocketChannel
        └──────────────────────┘
```

> 面试要点:Java NIO 是 **同步非阻塞** + **I/O 多路复用** 的组合,Netty 基于 NIO 又做了大量优化。

---

## 2. ② select / poll / epoll(NIO 的底层支撑)

```
┌──────────┬──────────────┬──────────────┬──────────────────────────┐
│          │   select     │    poll      │         epoll            │
├──────────┼──────────────┼──────────────┼──────────────────────────┤
│ FD 上限  │ 1024(FDSET)  │ 无上限       │ 无上限                   │
│ 数据结构 │ bitmap       │ pollfd 数组  │ 红黑树 + 就绪链表        │
│ 拷贝次数 │ 每次全部拷贝 │ 每次全部拷贝 │ 一次注册多次使用         │
│ 触发模式 │ LT           │ LT           │ LT + ET                  │
│ 复杂度   │ O(n)         │ O(n)         │ O(1)(只返回就绪)         │
│ 典型 API │ select()     │ poll()       │ epoll_create/ctl/wait    │
└──────────┴──────────────┴──────────────┴──────────────────────────┘
```

```
   epoll 工作流程
   ┌─────────────┐    ┌──────────────┐    ┌──────────────┐
   │epoll_create │──▶ │  epoll_ctl   │──▶ │  epoll_wait  │
   │ 创建实例    │    │ 注册/改/删FD │    │ 阻塞等就绪   │
   └─────────────┘    └──────────────┘    └──────────────┘
                                                 │
                                  返回:仅活跃 FD 列表(无需遍历全集合)
```

> **LT(水平触发)**:只要可读就一直通知。**ET(边缘触发)**:只在状态变化时通知一次,要求一次性把数据读完。

---

## 3. ③ Reactor 线程模型(连接 I/O 多路复用与 Netty)

### 3.1 三种 Reactor 模型递进

```
                      单 Reactor 单线程
                      ┌─────────────────────┐
                      │  Reactor(select)    │
                      │     ↓ dispatch      │
                      │ Acceptor    Handler │  ← 业务也在这跑,瓶颈明显
                      └─────────────────────┘

                      单 Reactor 多线程
                      ┌─────────────────────┐
                      │  Reactor(select)    │
                      │     ↓ dispatch      │
                      │ Acceptor    Handler │
                      │              ↓      │
                      │       Worker线程池  │ ← Handler 只 read/send
                      └─────────────────────┘

                      主从 Reactor 多线程  ★ Netty 采用
       ┌─────────────────────┐        ┌─────────────────────┐
       │  MainReactor(boss)  │ accept │  SubReactor(worker) │
       │     ↓               │ ─────▶ │     ↓               │
       │   Acceptor          │  分发  │  Handler            │
       │  (只处理 accept)    │        │    ↓                │
       └─────────────────────┘        │ Worker 线程池       │
                                      └─────────────────────┘
```

### 3.2 Reactor 模型对应到 Netty

```
   Reactor 模式概念   │  Netty 实现
   ───────────────────┼──────────────────────────
   MainReactor        │  BossGroup(NioEventLoopGroup)
   SubReactor         │  WorkerGroup(NioEventLoopGroup)
   Reactor 单线程     │  一个 EventLoop = 一个线程
   Acceptor           │  ServerBootstrapAcceptor
   Handler            │  ChannelHandler(用户自定义)
   Dispatch           │  ChannelPipeline 事件传播
```

---

## 4. ④ Netty 核心组件(必背)

### 4.1 组件关系一张图

```
   ServerBootstrap (引导器:把所有组件串联起来)
        │
        │ group(bossGroup, workerGroup)
        │ channel(NioServerSocketChannel.class)
        │ option(...) / childOption(...)
        │ childHandler(ChannelInitializer { ... })
        ▼
   ┌─────────────────────────────────────────────────────┐
   │   BossGroup (EventLoopGroup)                        │
   │   ├── EventLoop ─ Thread ─ Selector ─ Channel(N..)  │
   │   └── EventLoop ...                                 │
   └─────────────────┬───────────────────────────────────┘
                     │ accept 后注册到 WorkerGroup
                     ▼
   ┌─────────────────────────────────────────────────────┐
   │  WorkerGroup (EventLoopGroup)                       │
   │  ┌──────────────────────────────────────────┐       │
   │  │ EventLoop ─ Thread ─ Selector            │       │
   │  │     │                                    │       │
   │  │     ▼                                    │       │
   │  │   Channel ── ChannelPipeline             │       │
   │  │              │                           │       │
   │  │              ▼                           │       │
   │  │   Head ⇄ Ctx ⇄ Handler ⇄ ... ⇄ Tail      │       │
   │  │              │                           │       │
   │  │              ▼                           │       │
   │  │           ByteBuf(读写数据载体)          │       │
   │  └──────────────────────────────────────────┘       │
   └─────────────────────────────────────────────────────┘
```

### 4.2 关键基数关系

```
   EventLoopGroup ─── 1 : N ─── EventLoop ─── 1 : 1 ─── Thread
        │                              │
        │                              │
        │                          1 : N (一个 EventLoop 管多个 Channel)
        │                              │
        ▼                              ▼
       管理                        Channel ─── 1 : 1 ─── ChannelPipeline
                                       │
                                       │ 1 : N
                                       ▼
                                  ChannelHandlerContext ⇄ ChannelHandler
                                  (双向链表是 Context 串成的,不是 Handler)
```

**核心规则**:
- 一个 Channel 在它生命周期内只与一个 EventLoop 绑定 → 同 Channel 的所有 I/O 由同一线程串行处理,**天然线程安全**。
- 一个 EventLoop 可绑定多个 Channel。
- `NioEventLoopGroup` 默认线程数 = **CPU 核数 × 2**(`io.netty.eventLoopThreads` 可改)。

### 4.3 Channel 四种状态

```
   new          bind/connect       数据传输/异常        close
   ─────▶ Open ───────────▶ Active ──────────▶ Inactive ──────▶ Closed
   isOpen()     isActive()                                       (终态)
```

### 4.4 为什么 Netty 要重新设计 Channel(而不是用 JDK NIO 的)

1. JDK NIO 的 Channel 是 SPI,功能少,扩展难;
2. Netty 需要把 Channel 与 EventLoop、Pipeline、ByteBufAllocator 等组件深度耦合;
3. Netty Channel 提供更友好的状态机、Future/Promise 异步语义、统一的元数据配置。

---

## 5. ⑤ ChannelPipeline / ChannelHandler / ChannelHandlerContext

### 5.1 三者职责

```
┌──────────────────────┬─────────────────────────────────────────┐
│ ChannelPipeline      │ 责任链容器,管理 Handler 的有序传播      │
│ ChannelHandler       │ 数据加工厂,处理具体业务逻辑             │
│ ChannelHandlerContext│ 单一职责拆出来的"上下文",真正串成双向链 │
└──────────────────────┴─────────────────────────────────────────┘
```

> **重点**:Pipeline 中的双向链表,串起来的是 **Context**,不是 Handler 本身。

### 5.2 事件传播方向

```
              入站事件(InboundHandler)顺序执行
   ┌──▶───────────────────────────────────────────▶──┐
   │                                                 ▼
  Head ⇄ Ctx ⇄ InH1 ⇄ InH2 ⇄ OutH1 ⇄ OutH2 ⇄ InH3 ⇄ Tail
   ▲                                                 │
   └──◀─────────────────────────────────────────◀────┘
              出站事件(OutboundHandler)逆序执行
```

- **入站**(读、连接建立、异常):`Head → Tail`,依次跳到下一个 Inbound;
- **出站**(写、bind、close):`Tail → Head`,依次跳到上一个 Outbound;
- Head 既是 Inbound 又是 Outbound;Tail 只是 Inbound(终止入站传播)。

### 5.3 Netty 如何找到下一个可执行的 Handler

```
   每个 Handler 的 Class 在加载时,Netty 用反射扫描覆盖了哪些方法,
   生成一个 17 位的 mask(8 个入站事件 + 9 个出站事件):

   ┌─────────────────────────────────────────────────────────┐
   │ ctx.executionMask = 0b...10110011                       │
   └─────────────────────────────────────────────────────────┘
                              ▲
              查找下一个 Handler 时,做位运算:
              (ctx.executionMask & expectedMask) != 0
              → 命中  ✔
              → 跳过  ✘ (相当于该 Handler 没有重写这个事件)
```

避免了反射、避免了 instanceof 判断,性能极高。

### 5.4 自定义 Handler 的 4 种方式

```
┌────────────────────────────────────┬──────────────────────────┐
│ 方式                               │ 适用场景                 │
├────────────────────────────────────┼──────────────────────────┤
│ 实现接口 ChannelInbound/Outbound   │ 所有方法都要实现,少用    │
│ 继承 ChannelInbound/OutboundAdapter│ 经典做法,要手动 release  │
│ 继承 SimpleChannelInboundHandler<T>│ 自动释放消息(防泄漏)推荐 │
│ 继承 ChannelDuplexHandler          │ 同时处理出入站           │
└────────────────────────────────────┴──────────────────────────┘
```

> `SimpleChannelInboundHandler` 通过 `TypeParameterMatcher` 做泛型匹配,处理完后自动 `ReferenceCountUtil.release(msg)`。

### 5.5 Channel / Ctx / Pipeline 三种 writeAndFlush 的区别

```
   channel.writeAndFlush(msg)      → 从 Tail 出发,遍历完整出站链
   pipeline.writeAndFlush(msg)     → 等价于 channel.writeAndFlush()
   ctx.writeAndFlush(msg)          → 从当前 ctx 出发,只走它之前的出站 Handler
```

---

## 6. ⑥ ByteBuf 与内存模型

### 6.1 ByteBuf 内部结构

```
   0          readerIndex      writerIndex     capacity     maxCapacity
   ├──────────┼────────────────┼────────────────┼────────────┤
   │ 已废弃   │   可读字节     │    可写字节    │  可扩容    │
   │ 字节区   │ (writer-reader)│                │            │
   └──────────┴────────────────┴────────────────┴────────────┘

   discardReadBytes() 可以回收"已废弃"那段空间。
```

### 6.2 ByteBuf vs ByteBuffer

```
┌────────────────┬─────────────────────┬────────────────────────┐
│                │  Netty ByteBuf      │  JDK ByteBuffer        │
├────────────────┼─────────────────────┼────────────────────────┤
│ 读写索引       │ reader/writer 分离  │ position 一个指针      │
│ 模式切换       │ 不需要 flip()       │ 需要 flip()            │
│ 引用计数       │ 支持(防泄漏)       │ 不支持                 │
│ 池化           │ 支持(PooledByteBuf)│ 不支持                 │
│ 容量           │ 可动态扩展          │ 创建即定               │
│ 类型组合       │ 8 种(池/堆/安全)    │ 单一                   │
└────────────────┴─────────────────────┴────────────────────────┘
```

8 种实现的组合(三维组合):

```
   Pooled / Unpooled  ×  Heap / Direct  ×  Safe / Unsafe   ⇒  2³ = 8 种
```

### 6.3 内存模型(简化的 jemalloc4)

```
   PooledByteBufAllocator (全局分配器)
        │  每个线程取一个 Arena(2*CPU 个 Arena,round-robin)
        ▼
   PoolArena
        │
        ├── tinySubpagePools / smallSubpagePools  (<28KB)
        │
        └── PoolChunkList(q000/qInit/q025/q050/q075/q100)
                │
                ▼
            PoolChunk (16MB 大块)
                │  run + subpage + free
                ▼
            PoolSubpage  (再分小块,bitmap 管理)
```

内存规格:
```
   Small   :  0 ~ 28KB     → Subpage 分配
   Normal  : 28KB ~ 16MB   → Run 分配
   Huge    : > 16MB        → 直接分配,不池化
```

### 6.4 对象池 Recycler

```
   ┌────────────────────────────────────────┐
   │  Recycler<T>                           │
   │   ↓ get()                              │
   │  LocalPool(每线程一个,FastThreadLocal) │
   │   ├── MessagePassingQueue(Mpsc 队列)   │
   │   └── DefaultHandle(包装对象+state)    │
   │                                        │
   │   recycle() ─▶ 把 Handle 标记为 AVAILABLE 入队 │
   └────────────────────────────────────────┘
```

- 状态:`CLAIMED`(使用中) ⇄ `AVAILABLE`(可被复用),防止重复回收。
- 适合「创建开销大、生命周期短、复用频繁」的对象,例如 ByteBuf。

---

## 7. ⑦ 零拷贝(分两层理解)

### 7.1 OS 层零拷贝

普通 read + write 的开销:

```
   用户态 ───┐                                            ┌─── 用户态
            │  ① 切换  ④ 切换       ③ 切换  ⑥ 切换       │
   内核态 ──▼──────────────────────────────────────────▼──── 内核态
            │ DMA       CPU         CPU       DMA          │
            │ 磁盘─▶内核─▶用户缓冲─▶套接字缓冲─▶网卡       │
            └─ 2次CPU拷贝 + 2次DMA拷贝 + 4次上下文切换 ───┘
```

三种优化:

```
┌──────────────────┬──────────┬──────────┬──────────┐
│ 技术             │ 切换次数 │ CPU 拷贝 │ DMA 拷贝 │
├──────────────────┼──────────┼──────────┼──────────┤
│ read + write     │    4     │    2     │    2     │
│ mmap + write     │    4     │    1     │    2     │
│ sendfile         │    2     │    1     │    2     │
│ sendfile+SG-DMA  │    2     │  ★ 0    │    2     │ ← 真零拷贝
└──────────────────┴──────────┴──────────┴──────────┘
```

### 7.2 Netty 应用层"零拷贝"(更偏数据操作优化)

```
   ① CompositeByteBuf : 多个 ByteBuf 逻辑合并,免复制
   ② slice()          : 一个 ByteBuf 切多块,共享底层
   ③ wrap()           : byte[]/ByteBuffer 直接包装成 ByteBuf
   ④ FileRegion       : 借助 OS 的 sendfile 直接发文件,不经用户态
```

```
   header ┐
   body   ┴─▶ CompositeByteBuf ───┐
                                  │  逻辑合并,
                                  │  内部仍是两块内存,
                                  ▼  发送时不做 memcpy
                              writeAndFlush
```

---

## 8. ⑧ TCP 粘拆包 + 心跳长连接

### 8.1 粘拆包成因

```
   TCP 是字节流,没有"消息"概念
   ┌──────────────────────────────────────┐
   │  应用 write: [msgA][msgB][msgC]      │
   ├──────────────────────────────────────┤
   │  网络可能:                           │
   │    包1: msgA + 一半msgB              │ ← 拆包
   │    包2: 一半msgB + msgC              │ ← 粘包
   └──────────────────────────────────────┘
   根因:MTU(1500B)、MSS(1460B)分片、Nagle 算法合并。
```

### 8.2 Netty 的四种解码器

```
┌───────────────────────────────┬───────────────────────────────┐
│ FixedLengthFrameDecoder       │ 定长,简单粗暴,浪费空间       │
│ LineBasedFrameDecoder         │ 以 \n / \r\n 分隔,适合文本   │
│ DelimiterBasedFrameDecoder    │ 自定义分隔符                  │
│ LengthFieldBasedFrameDecoder  │ ★最通用:消息头携带长度字段  │
└───────────────────────────────┴───────────────────────────────┘
```

LengthFieldBasedFrameDecoder 四参数:

```
   |---------------- 完整帧 ---------------|
   |  Header  | Length(N) | Header2 | Data |
              ▲
              │
   lengthFieldOffset      lengthFieldLength
   lengthAdjustment       initialBytesToStrip
```

### 8.3 心跳与长连接

```
   两种心跳方式
   ┌─────────────────────────────────────┐
   │ ① TCP 层 SO_KEEPALIVE               │
   │    优点:操作系统帮你做              │
   │    缺点:周期 ~ 2 小时,无法定制    │
   ├─────────────────────────────────────┤
   │ ② 应用层 IdleStateHandler  ★推荐    │
   │    new IdleStateHandler(            │
   │       readerIdle,                   │
   │       writerIdle,                   │
   │       allIdle, TimeUnit.SECONDS)    │
   │    → 触发 IdleStateEvent            │
   │    → userEventTriggered() 中处理    │
   └─────────────────────────────────────┘
```

```
   IdleStateHandler 三种事件
   ┌─────────────┬──────────────────────────────┐
   │ READER_IDLE │ N 秒没读到任何数据           │
   │ WRITER_IDLE │ N 秒没写过任何数据           │
   │ ALL_IDLE    │ N 秒没读也没写               │
   └─────────────┴──────────────────────────────┘

   常见做法:服务端 READER_IDLE 触发 → close;
            客户端 WRITER_IDLE 触发 → 发心跳包。
```

---

## 9. 其他高频小题

### 9.1 Netty 如何解决 JDK Selector 空轮询 Bug

```
   一次 select() 循环中,如果 select() 返回 0 又没事件
   ┌────────────────────────────────────────┐
   │ selectCnt ++                           │
   │ if (selectCnt >= 512) {                │ ← 阈值可调:
   │   rebuildSelector();                   │   io.netty.selectorAutoRebuildThreshold
   │   selectCnt = 0;                       │
   │ }                                      │
   └────────────────────────────────────────┘

   rebuildSelector() 四步:
     1) open() 一个新 Selector
     2) 把旧 Selector 上的所有 Channel & 关心的事件重新注册到新 Selector
     3) 替换 EventLoop 的 selector 引用
     4) close() 旧的 Selector
```

### 9.2 Netty 大文件传输

- **FileRegion(零拷贝)**:借助 OS sendfile,直接把磁盘数据发到 Socket;
- **ChunkedWriteHandler + ChunkedFile**:把大文件切成 chunk(默认 8KB),异步逐块写出。

### 9.3 SSL/TLS

```
   ServerBootstrap
        ▼
   childHandler:
     ch.pipeline().addFirst(sslContext.newHandler(ch.alloc()))
     ─────────────────────────────────────────────────
     │ SslHandler 必须放在 Pipeline 的最前面,         │
     │ 这样所有出入站数据都先经过加解密。             │
     ─────────────────────────────────────────────────
```

---

## 10. ⑨ Netty 高性能(综合总纲)

```
                 ┌────────────────────────────────────────┐
                 │     Netty 为什么这么快?                │
                 └─────────────────┬──────────────────────┘
                                   │
   ┌───────────┬───────────────────┼──────────────────┬────────────┐
   │           │                   │                  │            │
   ▼           ▼                   ▼                  ▼            ▼
异步非阻塞   主从Reactor         零拷贝             内存池      高性能编解码
NIO          (boss/worker)       CompositeByteBuf   PooledByteBuf  & Pipeline
             EventLoop+Selector  slice/wrap         PoolArena      责任链 +
             单线程串行+无锁     FileRegion         Recycler对象池 Mask查找
   ↑           ↑                   ↑                  ↑            ↑
   │           │                   │                  │            │
   ① I/O 模型  ③ Reactor 模型      ⑦ 零拷贝           ⑥ 内存模型   ⑤ Pipeline
```

可以把这张图作为 **「Netty 高性能」** 一题的回答提纲:

1. **异步非阻塞 I/O**:基于 NIO + epoll,单线程支撑成千上万连接;
2. **主从 Reactor 多线程模型**:boss/worker 分工明确,Channel 绑定单线程,**串行无锁**;
3. **零拷贝**:`CompositeByteBuf` / `slice` / `wrap` / `FileRegion` 避免内存复制;
4. **内存池**:`PooledByteBufAllocator` + `PoolArena` + `Recycler`,降低 GC 压力;
5. **高效 Pipeline**:位运算 Mask 找下一个 Handler,无反射,无 instanceof;
6. **TCP 层调优**:`TCP_NODELAY`、`SO_KEEPALIVE` 等参数 + 自带粘拆包解码器;
7. **解决 JDK Bug**:Selector 空轮询自动 rebuild。

---

## 11. 推荐复习顺序(自上而下背 / 自下而上理解)

```
   理解顺序(先打地基)              复习/背诵顺序(先记结论)
   ─────────────────────         ─────────────────────────
   ① I/O 模型                    ⑨ Netty 高性能(总纲)
   ② select/poll/epoll           ④ 核心组件 6 大件
   ③ Reactor 模式                ⑤ Pipeline 事件传播方向
   ④ Netty 核心组件              ⑥ ByteBuf vs ByteBuffer
   ⑤ Pipeline / Handler          ⑦ 零拷贝四点
   ⑥ ByteBuf / 内存模型          ⑧ 粘拆包四种解码器
   ⑦ 零拷贝                      ① ~ ③ 底层原理
   ⑧ TCP 粘拆 / 心跳
   ⑨ 高性能综合
```

---

## 附录:一句话面试速答模板

| 题目 | 一句话答案 |
| --- | --- |
| BIO/NIO/AIO 区别 | 阻塞同步 / 非阻塞同步多路复用 / 异步,服务端模型分别是一连接一线程 / 一线程多连接 / 一请求一线程。 |
| select/poll/epoll 区别 | 前两者每次拷贝全集合且 O(n) 遍历;epoll 一次注册多次使用,只返回就绪 FD,O(1),支持 ET。 |
| Reactor 三种模型 | 单线程 / 单 Reactor 多线程 / 主从 Reactor 多线程(Netty 采用)。 |
| Netty 核心组件 | Bootstrap、Channel、EventLoop(Group)、ChannelPipeline、ChannelHandler(Context)、ByteBuf。 |
| Channel 与 EventLoop | 一个 Channel 终生绑定一个 EventLoop,一个 EventLoop 可管多个 Channel,所有 I/O 在 EventLoop 的单线程内串行执行,天然线程安全。 |
| NioEventLoopGroup 默认线程数 | `Math.max(1, CPU 核数 × 2)`,可通过 `io.netty.eventLoopThreads` 调整。 |
| ByteBuf 与 ByteBuffer 区别 | 读写索引分离、引用计数、支持池化与扩容、API 更丰富。 |
| Netty 零拷贝 | CompositeByteBuf 合并、slice 切分、wrap 包装、FileRegion 借助 OS sendfile。 |
| 粘拆包怎么解决 | 用四种 FrameDecoder,生产里最常用 LengthFieldBasedFrameDecoder。 |
| 心跳怎么做 | IdleStateHandler + userEventTriggered,优于 TCP 的 SO_KEEPALIVE。 |
| Selector 空轮询 | Netty 计数到阈值(512)就 rebuildSelector,把 Channel 迁移到新 Selector。 |
| Pipeline 找下一个 Handler | 位运算 mask 匹配事件,跳过未重写该事件的 Handler。 |
| Netty 为什么高性能 | 异步 NIO + 主从 Reactor + 零拷贝 + 内存池 + 无锁串行 + 位运算 Pipeline + 解决 JDK Bug。 |
