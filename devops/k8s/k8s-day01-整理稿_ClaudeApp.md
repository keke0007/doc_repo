# Kubernetes day01(基础)整理稿

> 原笔记版本基准:**Kubernetes 1.20+** 时代的基础课
> 涵盖:发展史、整体架构、控制平面/数据平面组件、Pod 使用、生命周期、健康检查探针

---

## 一、知识点总览

```
Kubernetes(K8s)
├─ 发展脉络
│   物理机 → 虚拟化(VMware/IaaS) → 容器(Docker 2013) → 编排(K8s 2014)
├─ 集群架构
│   ├─ Control Plane(Master)
│   │   ├─ kube-apiserver         所有操作的唯一入口,REST + etcd 网关
│   │   ├─ etcd                   分布式 KV,集群所有状态唯一持久化点
│   │   ├─ kube-scheduler         为新 Pod 选 Node
│   │   ├─ kube-controller-manager  内置一堆控制器(Replication/Node/Endpoint/...)
│   │   └─ cloud-controller-manager 与云厂商对接(LB/Volume/Node)
│   └─ Node(Worker)
│       ├─ kubelet                与 apiserver 通信,管理本机 Pod
│       ├─ kube-proxy             Service 的 iptables / IPVS 规则下发
│       └─ Container Runtime      containerd / CRI-O(Docker 已在 1.24 弃用)
├─ 核心对象
│   ├─ Pod        最小调度单位(可含多容器,共享 net+ipc+uts namespace)
│   ├─ Label / Selector  解耦对象关联
│   ├─ Namespace  逻辑隔离
│   ├─ Service    稳定虚拟 IP + 端口,代理一组 Pod
│   └─ ReplicaSet 维护副本数(由 Deployment 管理)
├─ Pod 生命周期
│   Pending → Running → Succeeded / Failed (+ Unknown)
│   ├─ initContainers(顺序执行,全部成功才进入主容器)
│   └─ 主容器 + lifecycle hooks(postStart / preStop)
└─ 健康检查
    ├─ livenessProbe   失败 → 重启容器
    ├─ readinessProbe  失败 → 从 Service Endpoints 摘除
    └─ startupProbe    1.16+,启动期专用,通过前抑制其他探针
```

---

## 二、整体架构与一次 `kubectl apply` 调用链

```
┌─── 用户 ───┐
│ kubectl apply -f pod.yaml │
└─────┬─────┘
      │ HTTPS,TLS 双向认证
      ▼
┌────────────────── Master ─────────────────────────────────────┐
│ kube-apiserver                                                │
│   ├─ Auth (Authn → Authz(RBAC) → Admission)                   │
│   ├─ 解析 yaml → Pod 对象                                     │
│   └─ etcd.put("/registry/pods/<ns>/<name>", obj)              │
│                                                                │
│ kube-scheduler  watch Pod (spec.nodeName == "")               │
│   ├─ 过滤(Predicates):资源/亲和性/污点/PV                    │
│   ├─ 打分(Priorities):最少使用、镜像已存在、Pod 反亲和...    │
│   └─ Bind: apiserver.patch(pod, spec.nodeName=nodeX)          │
│                                                                │
│ controller-manager (一堆 informer watch + reconcile)          │
│   └─ ReplicaSet/Deployment controller 维护期望状态            │
└──────────────────────────────────────────────────────────────┘
      ▲ watch
      │
┌─── Node X ─────────────────────────────────────┐
│ kubelet  watch Pod (spec.nodeName == self)     │
│   ├─ CRI(gRPC) → containerd                    │
│   │     ├─ PullImage                           │
│   │     ├─ CreateContainer(配 net/cgroup)      │
│   │     └─ StartContainer                      │
│   ├─ CNI 插件配 Pod 网络(eth0,IP,路由)         │
│   └─ CSI 插件挂载 Volume                       │
│                                                │
│ kube-proxy watch Endpoints / EndpointSlice     │
│   └─ 下发 iptables/IPVS 规则                   │
└────────────────────────────────────────────────┘
```

**⚠ 原笔记纠错(关键点 1):** 原文写"kubelet 直接与 Docker 交互启容器"。
- 自 **K8s 1.20** dockershim 进入 deprecated;
- **1.24** 起 dockershim 已从 kubelet 移除,kubelet **只通过 CRI(Container Runtime Interface)** 与运行时通信,默认实现是 **containerd** 或 CRI-O;
- 用 Docker 需另装 `cri-dockerd` shim,生产几乎都已切到 containerd。

**⚠ 原笔记纠错(关键点 2):** 原文写"scheduler 直接把 Pod 放到 Node 上"。
- scheduler **只做 Bind**,即给 Pod 写一个 `spec.nodeName`;
- 实际 **拉起容器的是目标 Node 上的 kubelet**(它 watch 到属于自己的 Pod 后开始 Run);
- 这是 K8s "声明式 + 控制循环" 设计的关键。

---

## 三、控制平面组件详解

| 组件 | 职责 | 失联后影响 |
| --- | --- | --- |
| kube-apiserver | 所有读写的唯一入口、认证鉴权、Admission、与 etcd 交互 | 整个集群不可控制;**已运行的 Pod 继续跑** |
| etcd | Raft 强一致 KV,存所有对象 | apiserver 不能读写,等价于整集群停服 |
| scheduler | Pod 调度 | 新 Pod 一直 `Pending`,已绑定 Pod 不受影响 |
| controller-manager | 各类控制循环 | 副本数/Endpoints/Node 状态不再自愈 |
| kubelet | Node 上 Pod 生命周期 | 该 Node 进入 NotReady,5 分钟后驱逐 Pod |
| kube-proxy | Service 规则 | 旧规则仍生效,新 Service/Endpoint 变更不下发 |

**⚠ 原笔记纠错(关键点 3):** 原文说"etcd 是 k8s 的数据库,存所有 Pod 的实时数据"。
更准确:etcd 存的是 **声明的期望状态 + 由控制器维护的当前状态**(spec/status),**不是节点上容器的实时进程数据**。容器的实时 cpu/mem 不入 etcd,而是由 metrics-server / cAdvisor 单独提供。

---

## 四、Pod:最小调度单位

### 4.1 多容器 Pod 共享什么

```
                Pod (= 一组共享命名空间的容器)
       ┌──────────────────────────────────────────┐
       │ pause 容器(infra container)              │
       │   - 占据 Pod 的 network/IPC/UTS namespace │
       │   - 1 号进程,负责回收 zombie              │
       ├──────────────────────────────────────────┤
       │ app 容器 1   nginx                        │
       │ app 容器 2   sidecar(log/metrics)        │
       │      ↑                                    │
       │      └─ 共享:net / IPC / UTS / volume    │
       │         不共享:PID(默认)/ mount / user  │
       └──────────────────────────────────────────┘
```

**⚠ 原笔记纠错(关键点 4):** 原文说"Pod 内容器共享一切命名空间"。
- 默认情况下 **不共享 PID 命名空间**(各容器看不到对方进程);
- 若需共享,要显式设 `spec.shareProcessNamespace: true`;
- Mount/User Namespace 也是不共享的,只共享 net/IPC/UTS。

### 4.2 自主 Pod vs 控制器管理的 Pod

| 类型 | 生命周期 | 故障自愈 | 适用场景 |
| --- | --- | --- | --- |
| 自主 Pod(裸 Pod) | 不会自动重建 | 否 | 调试、一次性任务 |
| Deployment / ReplicaSet 管理 | 删 Pod 会被重建 | 是 | 生产无状态应用 |
| DaemonSet 管理 | 每个 Node 一个 | 是 | 日志/监控代理 |
| StatefulSet 管理 | 顺序+稳定名+PVC | 是 | 有状态服务 |

---

## 五、镜像拉取策略(注意默认值)

| imagePullPolicy | 行为 |
| --- | --- |
| `Always` | 每次创建容器都拉镜像(适合 `:latest`) |
| `IfNotPresent` | 本地有就用,无才拉 |
| `Never` | 只用本地,没有就失败 |

**⚠ 原笔记纠错(关键点 5):** 原文说"K8s 默认 `IfNotPresent`"。**不完全:**
- tag 为 `:latest` 或**省略 tag**(等价 `:latest`) → 默认 `Always`;
- 显式带版本 tag(`:1.20`)→ 默认 `IfNotPresent`;
- 这是为什么生产环境一律 **禁止用 latest**,否则可能在每次重启时静默换镜像。

---

## 六、Pod 生命周期 + 探针

### 6.1 状态机

```
        ┌──────────┐  调度成功
        │ Pending  │ ─────────────► 拉镜像、起 initC、起主 C
        └────┬─────┘
             │ 容器至少一个 Running 且通过 startupProbe
             ▼
        ┌──────────┐
        │ Running  │ ◄─── readinessProbe 失败 → 仍 Running 但从 Endpoints 摘除
        └────┬─────┘
             │
   ┌─────────┼─────────┐
   │ 所有容器 │   任一容器 │
   │ 正常退出 │   非 0 退出 │
   │ 且不重启 │   且达到 RestartPolicy 上限
   ▼          ▼
┌─────────┐ ┌────────┐
│Succeeded│ │ Failed │
└─────────┘ └────────┘

RestartPolicy: Always(默认) / OnFailure / Never
```

### 6.2 三种探针

```
启动期:      ──startupProbe──►(通过)──┐
                                       │
                                       ▼
运行期(并行):
   livenessProbe   连续失败 → kubelet kill 容器 → 按 RestartPolicy 重启
   readinessProbe  连续失败 → Pod IP 从 Service Endpoints 移除(流量不进)
```

每种探针都有四种触发方式:
- **exec**:在容器里跑命令,exit 0 = 通过
- **httpGet**:HTTP 状态码 2xx/3xx = 通过
- **tcpSocket**:能建 TCP 连接 = 通过
- **grpc**(1.24+,1.27 GA):调用 gRPC Health Checking 协议

关键参数:`initialDelaySeconds` / `periodSeconds` / `timeoutSeconds` / `successThreshold` / `failureThreshold`

**⚠ 原笔记纠错(关键点 6):** 原文说"liveness 和 readiness 都不通过 → Pod 被删除"。
**不会被删除**:
- liveness 失败 → kubelet **重启容器**(不是删 Pod);
- readiness 失败 → 仅从 Endpoints 摘除,Pod 还活着,继续被探测;
- Pod 被删只在 RestartPolicy=Never 且容器最终 Failed,或 controller(Deployment 等)主动删除时。

**⚠ 原笔记纠错(关键点 7):** 原文没提 **startupProbe**(1.16 引入,1.20 GA)。
作用是给慢启动应用(JVM 应用、初始化几十秒)一个独立探针:**startupProbe 通过前,liveness/readiness 都不会执行**,避免应用还没启动好就被 livenessProbe 反复重启。

---

## 七、自主 Pod 资源清单关键字段

```yaml
apiVersion: v1                  # Pod 用 v1
kind: Pod
metadata:
  name: my-pod
  namespace: default            # 不写默认 default
  labels:
    app: nginx
spec:
  restartPolicy: Always         # Always(默认) / OnFailure / Never
  nodeSelector:                 # 调度到带特定 label 的 Node
    disktype: ssd
  containers:
  - name: nginx
    image: nginx:1.21
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80         # 仅声明,不发布;真正暴露靠 Service
    resources:
      requests: {cpu: 100m, memory: 128Mi}
      limits:   {cpu: 500m, memory: 256Mi}
    livenessProbe:
      httpGet: {path: /healthz, port: 80}
      initialDelaySeconds: 10
      periodSeconds: 5
```

**⚠ 原笔记纠错(关键点 8):** 原文将 `containerPort` 描述为"对外暴露的端口"。**错。**
`containerPort` 只是 **文档性声明**,主要用于:
1. 让 Service 的 `targetPort: <name>` 引用;
2. 让人读 yaml 时知道容器内监听什么端口;
- 实际网络在 Linux 上是看进程是否真的 listen,不写 `containerPort` 容器照样能被访问。要让外部访问,**必须用 Service / Ingress / hostPort**。

---

## 八、关键纠错清单(汇总)

| # | 原笔记表述 | 正确表述 |
| --- | --- | --- |
| 1 | "kubelet 直接调用 Docker" | 1.24+ 走 CRI,默认 containerd;Docker 需 cri-dockerd shim |
| 2 | "scheduler 把 Pod 放到 Node 上" | scheduler 只 Bind(写 spec.nodeName),拉起靠目标 kubelet |
| 3 | "etcd 存所有 Pod 的实时进程数据" | 只存 spec/status 声明,实时指标由 metrics-server 提供 |
| 4 | "Pod 内容器共享所有 namespace" | 仅共享 net/IPC/UTS;PID 默认不共享,需 shareProcessNamespace |
| 5 | "默认 imagePullPolicy = IfNotPresent" | latest/无 tag → Always;显式 tag → IfNotPresent |
| 6 | "liveness 失败 → Pod 被删" | liveness 失败 → 重启容器,不删 Pod;readiness 失败 → 摘 Endpoints |
| 7 | 未提 startupProbe | 慢启动应用必备,1.20 GA,通过前抑制其他探针 |
| 8 | "containerPort 是对外端口" | 仅为声明字段,真实暴露靠 Service/Ingress/hostPort |

---
