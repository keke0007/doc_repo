# Kubernetes Day01 知识点梳理

> 基于 `day01.md` 整理。内容覆盖 K8s 基础、核心概念、集群组件、Pod 使用、Pod 生命周期与健康检查。涉及多组件协作时使用 ASCII 流程图说明。

---

## 目录

1. [部署模式演进](#一部署模式演进)
2. [Kubernetes 概述](#二kubernetes-概述)
3. [核心概念](#三核心概念)
4. [集群组件](#四集群组件)
5. [Pod 使用](#五pod-使用)
6. [Pod 生命周期](#六pod-生命周期)
7. [健康检查](#七健康检查)
8. [常用命令](#八常用命令)

---

## 一、部署模式演进

### 1.1 物理单机

- 应用直接部署在物理服务器上
- 资源边界弱，一个应用可能占用大部分 CPU/内存
- 扩容通常依赖新增机器，成本高、利用率低

### 1.2 虚拟化

- VMware、Xen、KVM 推动虚拟机普及
- IaaS 以 VM 为基本计算单元
- 虚拟化提升资源利用率，也提供了隔离能力

### 1.3 容器化

- Docker 将 LXC、cgroups、UnionFS 等技术封装成标准镜像与容器运行方式
- 容器相对虚拟机更轻量，启动更快，资源占用更少
- 应用与运行时依赖被打包在一起，便于迁移和交付

### 1.4 云原生

- 基础前提：应用容器化、微服务化
- 核心能力：容器编排、自动调度、资源优化、自愈、弹性伸缩
- Kubernetes 成为云原生容器编排事实标准

```text
physical server
    |
    v
virtual machine
    |
    v
container
    |
    v
cloud native orchestration
    |
    v
Kubernetes
```

---

## 二、Kubernetes 概述

### 2.1 K8s 是什么

Kubernetes 是用于自动部署、扩展和管理容器化应用的开源系统。

### 2.2 来源

- 源自 Google Borg/Omega 的经验
- 2014 年开源
- 2015 年发布 1.0，并成为 CNCF 种子项目

### 2.3 为什么使用 K8s

- 可管理任意可容器化应用
- 屏蔽云厂商差异，便于迁移
- 自动调度，提高资源利用率
- 自动扩缩容
- 简化 CI/CD
- 具备故障恢复和自愈能力

---

## 三、核心概念

### 3.1 Master 节点

也称控制节点，负责集群管理控制。

核心组件：

- `kube-apiserver`
- `kube-scheduler`
- `kube-controller-manager`
- `etcd`

### 3.2 Node 节点

承载具体工作负载。

核心组件：

- `kubelet`
- `kube-proxy`
- 容器运行时，如 Docker、containerd、CRI-O

### 3.3 Pod

Pod 是 Kubernetes 中最小调度单元。一个 Pod 可以包含一个或多个容器，这些容器共享：

- 网络命名空间
- Pod IP
- 端口空间
- Volume

```text
Pod
 |
 +-- pause container
 |     |
 |     +-- owns pod network namespace
 |
 +-- app container 1
 |
 +-- app container 2
 |
 +-- shared volumes
```

### 3.4 Label 与 Selector

- Label 是资源上的键值标记
- Selector 用于筛选资源
- Service、ReplicaSet、Deployment 都依赖 Label Selector 关联 Pod

### 3.5 ReplicaSet

定义期望 Pod 副本数，并持续维持实际副本数接近期望值。

```text
ReplicaSet desired replicas = 2
       |
       v
watch pods by label
       |
       +-- actual < desired -> create pod
       |
       +-- actual > desired -> delete pod
```

### 3.6 Service

Service 提供稳定访问入口，通过 Label Selector 关联后端 Pod。即使 Pod IP 变化，客户端也访问稳定的 Service。

### 3.7 Namespace

用于资源隔离，可按项目、环境、团队划分资源空间。

---

## 四、集群组件

### 4.1 控制平面

- `kube-apiserver`：集群资源操作入口，处理 kubectl、组件和外部系统请求
- `kube-scheduler`：根据资源、约束和策略为 Pod 选择 Node
- `kube-controller-manager`：运行各类控制器，持续将实际状态修正为期望状态
- `etcd`：保存集群状态和配置，是集群数据的最终事实来源

### 4.2 工作节点

- `kubelet`：接收 PodSpec，调用容器运行时创建、启动、停止容器
- `kube-proxy`：维护 Service 网络转发规则
- `container runtime`：真正运行容器

### 4.3 创建 Pod 的多组件执行流程

```text
kubectl apply -f pod.yml
        |
        v
kube-apiserver
        |
        +-- validate resource
        +-- persist desired state
        v
      etcd
        |
        v
kube-scheduler watches unscheduled pod
        |
        v
select target Node
        |
        v
write binding through apiserver
        |
        v
kubelet on target Node watches pod
        |
        v
container runtime pulls image and starts container
        |
        v
Pod Running
```

---

## 五、Pod 使用

### 5.1 Pod 特点

- 每个 Pod 有唯一 IP
- 同一个 Pod 内容器可通过 `localhost` 通信
- Pod 内容器可共享 Volume

### 5.2 自主式 Pod

- 直接创建 Pod
- 不具备集群级自愈能力
- Node 故障、Pod 删除后不会由自身自动补足

### 5.3 控制器管理的 Pod

通常使用 Deployment、ReplicaSet 等控制器管理 Pod，可获得：

- 副本管理
- 滚动更新
- 自动重建
- 节点故障迁移

### 5.4 Pod YAML 关键字段

- `apiVersion`
- `kind`
- `metadata.name`
- `metadata.labels`
- `spec.containers`
- `image`
- `ports`

### 5.5 镜像拉取策略

| 策略 | 说明 |
| --- | --- |
| `Always` | 总是从远程仓库拉取 |
| `IfNotPresent` | 本地不存在才拉取，默认常见策略 |
| `Never` | 从不拉取，只使用本地镜像 |

---

## 六、Pod 生命周期

### 6.1 Pod 阶段

| 状态 | 含义 |
| --- | --- |
| `Pending` | 已被接受，但还未完成调度或镜像创建 |
| `Running` | 已绑定 Node，至少一个容器运行或启动中 |
| `Succeeded` | 所有容器成功终止且不再重启 |
| `Failed` | 所有容器终止，至少一个失败 |
| `Unknown` | 无法获取 Pod 状态 |
| `Terminating` | 删除中，实践中常见 |

### 6.2 重启策略

| 策略 | 说明 |
| --- | --- |
| `Always` | 容器失效就重启 |
| `OnFailure` | 非 0 退出才重启 |
| `Never` | 不重启 |

重启由 Pod 所在 Node 的 `kubelet` 执行，属于本地重启，不等于重新调度。

### 6.3 Init Container

- 主容器启动前执行
- 多个 Init Container 串行执行
- 必须成功完成后主容器才会启动

```text
Pod starts
   |
   v
init container 1
   |
   v
init container 2
   |
   v
main containers start
```

---

## 七、健康检查

### 7.1 为什么需要健康检查

默认情况下，Kubernetes 只能根据容器主进程退出码判断故障。对于进程还活着但服务不可用的情况，需要探针。

### 7.2 Liveness Probe

存活探针判断容器是否还活着。

- 成功：继续运行
- 失败：kubelet 杀掉容器，并按 restartPolicy 重启

```text
kubelet periodically probes container
        |
        +-- success -> keep running
        |
        +-- failure -> kill container -> restart by policy
```

### 7.3 Readiness Probe

就绪探针判断容器是否可以接收流量。

- 成功：加入 Service Endpoints
- 失败：从 Endpoints 移除，不接收流量

```text
readiness probe
    |
    +-- success -> Pod IP stays in Service endpoints
    |
    +-- failure -> Pod IP removed from Service endpoints
```

### 7.4 Liveness 与 Readiness 对比

| 探针 | 目的 | 失败后行为 |
| --- | --- | --- |
| Liveness | 判断是否需要重启 | 重启容器 |
| Readiness | 判断是否可接流量 | 移出服务端点 |

### 7.5 探针类型

- `exec`：在容器内执行命令，退出码 0 表示成功
- `httpGet`：HTTP 返回 2xx/3xx 表示成功
- `tcpSocket`：端口可连接表示成功

### 7.6 常用参数

- `initialDelaySeconds`：延迟多久开始探测
- `timeoutSeconds`：探测超时时间
- `periodSeconds`：探测周期

---

## 八、常用命令

```bash
kubectl apply -f xxx.yml
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl delete pod <pod-name>
kubectl exec <pod-name> -it -- /bin/bash
kubectl get pods -w
```

---

## 九、速记

- K8s 的核心是“声明期望状态，控制器持续逼近期望状态”。
- Pod 是最小调度单元，不是容器本身。
- 生产中一般用 Deployment/ReplicaSet 管理 Pod，而不是裸 Pod。
- Liveness 管重启，Readiness 管流量。
- Service 通过 Label Selector 找 Pod，给客户端稳定入口。
