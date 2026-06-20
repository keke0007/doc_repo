# Kubernetes Day02 知识点梳理

> 基于 `day02.md` 整理。内容覆盖 Pod 控制器、ReplicaSet、Deployment、数据卷、调度策略与 Service。涉及多组件/多资源协作时使用 ASCII 流程图说明。

---

## 目录

1. [Pod 控制器](#一pod-控制器)
2. [ReplicaSet](#二replicaset)
3. [Deployment](#三deployment)
4. [数据存储](#四数据存储)
5. [Pod 调度策略](#五pod-调度策略)
6. [Service](#六service)
7. [端口区分](#七端口区分)

---

## 一、Pod 控制器

### 1.1 控制器是什么

Pod 控制器用于自动调度和管理 Pod，使实际运行状态持续逼近期望状态。

控制器会通过 API Server 持续监控资源状态，当副本数、版本、节点故障等导致实际状态变化时，自动创建、删除或替换 Pod。

### 1.2 控制器的三个基本组成

- 标签选择器：匹配并关联 Pod
- 期望副本数：希望运行多少个 Pod
- Pod 模板：创建新 Pod 时使用的模板

### 1.3 为什么需要控制器

裸 Pod 不具备集群级自愈能力。容器进程崩溃时 kubelet 可按策略重启容器，但如果 Pod 被删除、Node 故障或资源漂移，就需要控制器补足。

### 1.4 常见控制器

| 控制器 | 作用 |
| --- | --- |
| ReplicaSet | 保证指定数量的 Pod 副本 |
| Deployment | 管理无状态应用，支持滚动更新和回滚 |
| DaemonSet | 保证每个节点运行一个特定 Pod |

---

## 二、ReplicaSet

### 2.1 概述

ReplicaSet 用于确保匹配标签选择器的 Pod 数量始终等于期望值。

### 2.2 核心字段

| 字段 | 说明 |
| --- | --- |
| `replicas` | 期望副本数 |
| `selector` | 匹配 Pod 的标签选择器 |
| `template` | 创建 Pod 使用的模板 |
| `minReadySeconds` | Pod 启动后多少秒视为可用 |

### 2.3 工作流程

```text
ReplicaSet controller
        |
        v
watch pods by selector
        |
        v
compare actual replicas with desired replicas
        |
        +-- actual < desired -> create Pod from template
        |
        +-- actual > desired -> delete extra Pod
        |
        +-- actual = desired -> keep watching
```

### 2.4 更新特点

直接修改 ReplicaSet 的 Pod 模板不会自动替换已有 Pod。新模板通常只影响后续新建 Pod，已有 Pod 需要删除后由 RS 重建才会使用新模板。

```text
update ReplicaSet template
        |
        v
existing Pods unchanged
        |
        v
delete old Pod manually
        |
        v
ReplicaSet creates new Pod with new template
```

### 2.5 扩缩容

- 命令方式：`kubectl scale rs <name> --replicas=<n>`
- 配置方式：修改 YAML 中 `spec.replicas` 后 `kubectl apply`

---

## 三、Deployment

### 3.1 概述

Deployment 工作在 ReplicaSet 之上，提供声明式更新能力。用户只需要描述目标状态，Deployment Controller 会协调 ReplicaSet 和 Pod 完成变化。

### 3.2 Deployment 与 ReplicaSet / Pod 关系

```text
Deployment
    |
    v
ReplicaSet revision A
    |
    +-- Pod A1
    +-- Pod A2

update image
    |
    v
Deployment
    |
    +-- ReplicaSet revision A -> replicas gradually to 0
    |
    +-- ReplicaSet revision B -> replicas gradually to desired
```

### 3.3 Deployment 能力

- 查看升级进度和状态
- 滚动更新
- 回滚到上一个或指定历史版本
- 版本记录
- 暂停/继续发布
- 声明式管理

### 3.4 更新策略

#### Recreate

先删除所有旧 Pod，再创建新 Pod。过程简单，但中间会不可用。

```text
old Pods running
    |
    v
delete all old Pods
    |
    v
create new Pods
```

#### RollingUpdate

逐步创建新 Pod、删除旧 Pod，尽量保持服务可用。

```text
old RS replicas = 2
new RS replicas = 0
        |
        v
create one new Pod
        |
        v
delete one old Pod
        |
        v
repeat until new RS reaches desired replicas
```

关键参数：

- `maxSurge`：更新期间最多可超过期望副本数的数量
- `maxUnavailable`：更新期间最多允许不可用的副本数

两者不能同时为 0。

### 3.5 更新与回滚命令

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.15
kubectl rollout undo deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=<revision>
```

### 3.6 Deployment 回滚流程

```text
rollout undo
    |
    v
Deployment finds previous ReplicaSet
    |
    v
scale previous RS up
    |
    v
scale current RS down
    |
    v
Pods run previous image
```

### 3.7 扩缩容与删除

- 扩缩容：修改 `replicas` 或使用 `kubectl scale`
- 删除 Deployment 时，受其管理的 ReplicaSet 和 Pod 会随之删除

---

## 四、数据存储

### 4.1 为什么需要 Volume

Pod 有生命周期，容器重启或 Pod 删除可能导致容器文件系统数据丢失。Volume 解决两个问题：

- Pod 内多个容器共享数据
- 数据独立于容器文件系统

### 4.2 Volume 与 Pod / Container 关系

```text
Pod
 |
 +-- Volume: html
 |
 +-- Container: nginx
 |       |
 |       +-- mount /usr/share/nginx/html
 |
 +-- Container: sidecar
         |
         +-- mount /html
```

### 4.3 emptyDir

`emptyDir` 是 Pod 生命周期内的临时目录。

特点：

- Pod 创建时创建
- Pod 删除时删除
- 容器崩溃不会删除数据，因为 Pod 还在
- 适合临时缓存、Pod 内容器共享数据

```text
Pod created -> emptyDir created on node
        |
        v
containers share same directory
        |
        +-- container restart -> data stays
        |
        +-- Pod deleted -> data removed
```

### 4.4 hostPath

`hostPath` 将 Node 上的目录或文件挂载到 Pod。

特点：

- 数据独立于 Pod 生命周期
- Pod 删除后 Node 本地数据仍在
- Pod 漂移到其他 Node 后，不能读取原 Node 的本地目录
- 属于半持久化方案

```text
Pod on node1
    |
    v
mount node1:/tmp/k8s/data -> container path
    |
    v
Pod deleted and recreated on node2
    |
    v
mount node2:/tmp/k8s/data
    |
    v
old data on node1 is not visible
```

### 4.5 存储类型速记

- 非持久性：`emptyDir`、`hostPath`
- 网络存储：NFS、iSCSI
- 分布式存储：GlusterFS、CephFS、RBD
- 云存储：AWS EBS、Azure Disk 等

---

## 五、Pod 调度策略

### 5.1 nodeName

`spec.nodeName` 强制指定 Pod 到某个 Node，跳过 Scheduler 调度逻辑。

```text
Pod spec.nodeName = k8s-node1
        |
        v
skip scheduler
        |
        v
kubelet on k8s-node1 runs Pod
```

### 5.2 nodeSelector

先给 Node 打标签，再在 Pod 中使用 `nodeSelector` 匹配标签。

```text
kubectl label node k8s-node2 disk=ssd
        |
        v
Pod spec.nodeSelector:
  disk: ssd
        |
        v
scheduler selects nodes with disk=ssd
```

注意：如果没有满足标签的 Node，Pod 不会被调度。

---

## 六、Service

### 6.1 Service 是什么

Service 是一组 Pod 的稳定访问入口，通过 Label Selector 选中后端 Pod。

它解决 Pod IP 会变化的问题：客户端访问稳定的 Service IP/端口，由 Kubernetes 转发到后端 Pod。

### 6.2 Service 实现模型

Service 不直接连接 Pod，中间有 Endpoints。

```text
Service
  selector: app=nginx
        |
        v
Endpoints
  - podIP1:targetPort
  - podIP2:targetPort
        |
        v
Pods
```

Service 通过 API Server watch 匹配到的 Pod 变化，并维护对应 Endpoints。

```text
Pod created/deleted/IP changed
        |
        v
API Server state changes
        |
        v
Endpoints updated
        |
        v
kube-proxy updates iptables/ipvs rules
        |
        v
traffic goes to healthy backend Pods
```

### 6.3 Service 类型

| 类型 | 作用 |
| --- | --- |
| `ClusterIP` | 默认类型，仅集群内部访问 |
| `NodePort` | 在每个 Node 暴露端口，支持集群外访问 |
| `LoadBalancer` | 借助云厂商负载均衡器暴露 |
| `ExternalName` | 将外部服务映射到集群内部名称 |

### 6.4 ClusterIP

ClusterIP 是集群内部虚拟 IP，适合集群内部服务调用。

```text
client pod
    |
    v
ClusterIP:port
    |
    v
kube-proxy rules
    |
    v
backend podIP:targetPort
```

### 6.5 NodePort

NodePort 在每台 Node 上开放一个端口，外部可通过 `nodeIP:nodePort` 访问。

```text
external client
    |
    v
nodeIP:nodePort
    |
    v
Service ClusterIP:port
    |
    v
Pod targetPort
```

---

## 七、端口区分

| 字段 | 所属对象 | 作用 |
| --- | --- | --- |
| `port` | Service | Service 暴露给集群内部访问的端口 |
| `targetPort` | Pod/Container | 最终转发到容器的端口 |
| `nodePort` | Node | 暴露给集群外部访问的 Node 端口 |

访问链路：

```text
inside cluster:
client -> serviceIP:port -> podIP:targetPort

outside cluster:
client -> nodeIP:nodePort -> serviceIP:port -> podIP:targetPort
```

---

## 八、速记

- ReplicaSet 保副本数，Deployment 管版本和发布。
- Deployment 更新会创建新 ReplicaSet，旧 ReplicaSet 保留用于回滚。
- emptyDir 随 Pod 消失，hostPath 随 Node 保存。
- nodeName 是强制指定节点，nodeSelector 是基于标签调度。
- Service 给 Pod 提供稳定入口，Endpoints 保存真实后端地址。
- ClusterIP 面向集群内，NodePort 面向集群外。
