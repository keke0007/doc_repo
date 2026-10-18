---
author: "Jerry Fang"
handle: "@Jerry_fsy"
source: "https://x.com/Jerry_fsy/status/2105612519050227818"
date: 2026-10-01T10:55:19.000Z
type: thread
likes: 60
reposts: 15
replies: 2
bookmarks: 65
views: 3505
---
# Jerry Fang (@Jerry_fsy)

感谢@KakaluoteW45042和@yifanxu_ephai 等人的支持，今天先写Agent Infra需要的分布式知识的基础篇：  
我在学习AI Infra/Agent Infra的时候发现，在这些领域涉及到很多分布式方面的知识，如果对分布式这块不是很了解的话，很多AI Infra/Agent Infra的设计和取舍可能比较难以理解。为了帮助想入门AI Infra/Agent Infra的朋友更好的学习这些领域，我整理了一些最好能提前理解的分布式知识。  
  
这是第一篇，涉及到分布式的基础知识，这是后续学习其他分布式概念的根基：  
  
1.Process / Node  
Process即进程，就是正在运行的一个程序实例，比如我们在终端中执行”codex”就是启动了一个CodeX的process。如果我们再执行”claude”的话，则是启动了一个Claude Code的process。  
  
在一台服务器中可以同时运行很多进程，不同进程拥有各自独立的虚拟地址空间，默认不能直接访问彼此的内存。  
  
那么进程间想通信怎么办呢？如果进程之间需要交换数据，就需要使用进程间通信机制。这里面会涉及到IPC / RPC / HTTP / gRPC / Message Queue这些概念。  
  
Node一般指整个分布式系统中的一个计算节点，它可以是一台服务器或者虚拟机。  
  
例如下面的一个简单的架构图，就涉及到4个node，每一个node都有各自的职责：

![Image](<Jerry%20Fang%20(@Jerry_fsy).assets/image-01.jpg>)

---

2.RPC  
我们已经介绍了node，一个分布式系统中会涉及到很多nodes，不同node之间的进程process时常需要通信，其中最重要的一种通信方式就是网络通信了。  
  
还是用下面这张架构图来说明，假如Node B中的worker想调用Node C中的工具search_web，那么Node

![Image](<Jerry%20Fang%20(@Jerry_fsy).assets/image-02.jpg>)

---

3.HTTP / gRPC  
前面说的RPC其实只是一个概念，它不是具体的通信协议。真实实现RPC时可以采用不同的方案，我们这篇主要对比下面两种方案：  
  
HTTP + JSON和gRPC + Protobuf  
  
总体来说，HTTP + JSON更加通用，可读性更强，调试也比较方便；而gRPC +

![Image](<Jerry%20Fang%20(@Jerry_fsy).assets/image-03.jpg>)

---

4.Serialization/Deserialization 序列化/反序列化  
我们在程序中往往使用数据结构来存储数据，当我们想将数据结构中存储的数据通过网络传输到另一个node中的进程时，会发现网络中是不认识dict,

![Image](<Jerry%20Fang%20(@Jerry_fsy).assets/image-04.jpg>)

---

5.Latency 延迟  
我们调用本地函数，可能是微秒级；但是如果涉及到跨节点的调用，其中的延迟就可能很高，也涉及到很多种（“延迟”是一个总体概念），包括网络延迟，LLM prefill和decode的计算延迟，排队延迟等等。  
  
对于Agent来说，因为涉及到的步骤很多，所以它对于降低延迟的需求比较强烈。因此，Agent

---

6.Timeout 超时  
假如Agent远程发送一个请求给Tool Server，但是Tool Server卡死了，那么这种情况下，Agent就不应该一直等待Tool

---

7.Partial Failure 部分失效  
我们有提到，分布式会涉及到很多node节点，每个节点上会有各自的服务。  
  
有的时候会出现，其中一部分服务失效的情况，即部分服务失效，剩下的服务稳定运行。这种情况被称为Partial Failure。  
  
因为Agent

---

8. Network Partition  
假设我们Node B节点上的Scheduler发布了一个任务给了Node C节点的worker A，该worker正在平稳运行着，执行着相应的操作，但是，这个时候，如果我们的Node B节点联系不上Node C节点，比如Node B上测试Node C心跳缺失，Node B节点会认为Node C挂了。  
  
这种情况下，Node

---

9.Clock / Time  
分布式涉及到很多node节点，每一个node可能都有自己的时间。  
  
从分布式系统的角度看，这些时间很可能是不一样的，有一点差异。这种机器之间的时间偏差叫做Clock Skew（时钟偏差）。这就会导致，不同节点对同一个事件记录的时间不同，可能会导致出现一些问题。

---

10.CAP  
CAP是三个单词的首字母，对应的单词分别是：Consistency（一致性），Availability（可用性）和Partition（分区容忍性）。  
  
Consistency可以理解为包括状态的一致性，比如Agent State这块，对于一个任务task 1，如果 task1 已经成功更新为

![Image](<Jerry%20Fang%20(@Jerry_fsy).assets/image-05.jpg>)

---

对分布式感兴趣的朋友们，可以看看《数据密集型应用系统设计》这本书。