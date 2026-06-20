# Kubernetes day02(高级)整理稿

> 原笔记版本基准:**Kubernetes 1.20+**
> 涵盖:Pod 控制器(RS/Deployment/DaemonSet)、Volume(emptyDir/hostPath)、调度策略、Service(ClusterIP/NodePort)。

---

## 一、知识点总览

```
K8s 高级使用
├─ Pod 控制器
│   ├─ ReplicaSet       维护期望副本数(底层)
│   ├─ Deployment       管理 RS,提供滚动/回滚/暂停
│   ├─ DaemonSet        每个(符合条件的)Node 一个 Pod
│   ├─ StatefulSet      有状态:顺序、稳定名、PVC
│   └─ Job / CronJob    一次性 / 定时任务
├─ 更新策略
│   ├─ Recreate         先全删旧再起新(有停服窗口)
│   └─ RollingUpdate    滚动(maxSurge / maxUnavailable)
├─ 存储卷
│   ├─ emptyDir         Pod 级,Pod 删则丢
│   ├─ hostPath         绑节点目录,Pod 漂走就丢上下文
│   ├─ ConfigMap/Secret 配置注入
│   ├─ PV / PVC         持久卷声明
│   └─ StorageClass     动态供给
├─ 调度
│   ├─ nodeSelector     简单 label 匹配
│   ├─ nodeAffinity     表达式更丰富(Required/Preferred)
│   ├─ podAffinity/AntiAffinity  Pod 与 Pod 之间
│   └─ taints + tolerations      节点驱逐 / 容忍
└─ Service
    ├─ ClusterIP        集群内虚拟 IP(默认)
    ├─ NodePort         每 Node 开静态端口
    ├─ LoadBalancer     云厂商外网 LB
    ├─ ExternalName     CNAME 到外部域名
    └─ Headless(clusterIP: None)  直接返回 Pod IP 列表
```

---

## 二、控制器层级:RS 与 Deployment 的关系

```
       Deployment
            │ owns
            ▼
       ReplicaSet v1    ←─── 一次更新会创建新 RS
            │ owns
            ▼
        Pod × N

滚动更新时:
   Deployment
     ├─ ReplicaSet v1 (期望副本 N → 逐步缩到 0)
     └─ ReplicaSet v2 (期望副本 0 → 逐步扩到 N)

Pod 不直接归 Deployment 拥有,而是归 RS 拥有(via ownerReferences)。
```

**⚠ 原笔记纠错(关键点 1):** 原文说"现在已经不用 ReplicaSet 了,Deployment 直接管 Pod"。**错。**
- Deployment 内部 **始终** 通过 ReplicaSet 管 Pod;
- 每次 `image:` 变更或 PodTemplate 哈希变化,Deployment 都会新建一个 RS;
- 这是回滚能瞬间生效的根因(老 RS 仍在,只是副本数为 0)。

**ReplicationController** 已废弃,但 **ReplicaSet 仍是 Deployment 的底层基础对象**,没有被淘汰。

---

## 三、Deployment 滚动更新流程

### 3.1 参数

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%           # 更新过程中,Pod 总数最多比 replicas 多多少
      maxUnavailable: 25%     # 更新过程中,允许不可用的最大 Pod 数
```

期望副本 `replicas = 4` 时:
- `maxSurge=25% → 1` → 总数最多 `4+1=5`
- `maxUnavailable=25% → 1` → 可用至少 `4-1=3`

### 3.2 多文件调用链

```
kubectl set image deploy/web nginx=nginx:1.22
       │
       ▼
apiserver  patch Deployment.spec.template.spec.containers[0].image
       │
       │  controller-manager watch 到 Deployment 变化
       ▼
DeploymentController.syncDeployment
   ├─ 计算 newRS(PodTemplateHash 不一样 → 新 RS)
   ├─ 不存在则 createRS(replicas=0)
   ├─ rolloutRolling():
   │    ├─ scaleUpNewRS(按 maxSurge 上限)
   │    │     └─ patch newRS.spec.replicas += step
   │    │
   │    │   ReplicaSetController watch newRS
   │    │     └─ createPod() → 新版本 Pod
   │    │           └─ scheduler 调度 → kubelet 拉起
   │    │
   │    ├─ 等新 RS Ready 数达标
   │    ├─ scaleDownOldRS(按 maxUnavailable 上限)
   │    │     └─ patch oldRS.spec.replicas -= step
   │    │
   │    │   ReplicaSetController watch oldRS
   │    │     └─ deletePod() → 调用 kubelet preStop + SIGTERM
   │    │
   │    └─ 直到 newRS.replicas == 期望,oldRS.replicas == 0
   └─ 更新 Deployment.status (UpdatedReplicas / AvailableReplicas)
```

### 3.3 回滚

```
kubectl rollout history deployment/web      # 列出 Revision
kubectl rollout undo deployment/web         # 回退到上一个
kubectl rollout undo deployment/web --to-revision=2
```

**⚠ 原笔记纠错(关键点 2):** 原文说"回滚是新建 RS 部署旧版本"。**错。**
- 旧 RS 一直存在(只是 `replicas=0`);
- 回滚做的是:把旧 RS `replicas` 调回去、新 RS `replicas` 调成 0;
- 默认保留 10 个旧 RS(`spec.revisionHistoryLimit`),设小可节省 etcd。

---

## 四、Service:实现原理与四种类型

### 4.1 一次访问 Service 的数据路径(iptables 模式)

```
Pod A 访问 svc/web (ClusterIP 10.96.1.100:80)
   │
   ▼
Pod A netns 内的路由 → 走主机 root netns
   │
   ▼
┌─────────── Node 主机 ───────────┐
│ iptables PREROUTING/OUTPUT      │
│   nat 表 KUBE-SERVICES 链       │
│     dst=10.96.1.100:80          │
│       └─ KUBE-SVC-XXXX          │
│            ├─ 50% → KUBE-SEP-A  │  ─── DNAT → 10.244.1.5:8080
│            └─ 50% → KUBE-SEP-B  │  ─── DNAT → 10.244.2.7:8080
│                                  │
│ DNAT 后包出 cni0/veth → 目标 Pod │
└─────────────────────────────────┘

后端 Pod 列表由谁维护?
   EndpointController(或 1.21+ EndpointSliceController) watch:
     - Service.spec.selector
     - 对应 Pod.status.podIP + readinessProbe
   写入 Endpoints / EndpointSlice 对象
   kube-proxy watch Endpoints → 增删 iptables 规则
```

### 4.2 四种 Service 类型

| 类型 | 暴露范围 | 典型用法 |
| --- | --- | --- |
| ClusterIP(默认) | 集群内 | 微服务内部互调 |
| NodePort | 每个 Node 的 30000-32767 端口 | 临时外部访问、测试 |
| LoadBalancer | 云厂商 LB | 生产对外 |
| ExternalName | DNS CNAME | 把外部服务伪装成集群内 svc |
| Headless(`clusterIP: None`) | DNS 直接返回 Pod IP | StatefulSet / 自定义负载均衡 |

### 4.3 三类端口辨析

```
Service 端口模型:
     外网用户                Node                Service ClusterIP        Pod
       │                      │                       │                   │
       │  访问 NodeIP:30080   │                       │                   │
       ├────────────────────► │                       │                   │
       │   (nodePort: 30080)  │  iptables DNAT       │                   │
       │                      ├─────────────────────► │                   │
       │                      │     port: 80          │ targetPort: 8080  │
       │                      │                       ├─────────────────► │
       │                      │                       │                   │ 容器 listen 8080

字段总结:
  port        Service ClusterIP 暴露的端口(集群内访问用)
  targetPort  容器实际监听端口(可写名字引用 containerPort.name)
  nodePort    NodePort 类型下每个 Node 监听的端口(30000-32767)
```

**⚠ 原笔记纠错(关键点 3):** 原文 `nodePort` 描述为"映射宿主机端口给 Pod"。
更准确:`nodePort` 不是 docker 风格的 `-p` 端口映射,而是 **kube-proxy 在每个 Node 上下发 iptables/IPVS 规则**,监听 `0.0.0.0:<nodePort>`,DNAT 到 Service 后端的 Pod。即使该 Node **没有目标 Pod**,流量也会被转到其它 Node 上的 Pod。

**⚠ 原笔记纠错(关键点 4):** 原文写"kube-proxy 是负载均衡器,流量都走 kube-proxy"。**错。**
- kube-proxy 只是 **规则下发器**,流量不经过它的用户态进程;
- 真正的转发由 **内核 netfilter/iptables 或 IPVS** 完成(数据路径全内核态);
- 1.0 时代曾有 userspace 模式真的走 kube-proxy 进程,早已弃用。

---

## 五、Endpoints 与 EndpointSlice

```
Service web                     EndpointSlice web-abcde
  selector: app=nginx              ports: [{port:8080,name:http}]
                                   endpoints:
                                   - addresses: [10.244.1.5]
                                     conditions: {ready: true}
                                     targetRef: Pod/web-pod-1
                                   - addresses: [10.244.2.7]
                                     conditions: {ready: false}  ← readinessProbe 失败
                                     targetRef: Pod/web-pod-2
```

**⚠ 原笔记纠错(关键点 5):** 原笔记仅提到 `Endpoints`。
自 K8s **1.21 起 EndpointSlice 默认启用并是 kube-proxy 的事实数据源**。区别:
- `Endpoints` 是单个对象,后端 Pod 多时(>1000)更新极慢;
- `EndpointSlice` 按 100 个为一组分片,推送增量小,大集群必备。

---

## 六、Volume:emptyDir vs hostPath

```
emptyDir (Pod 级临时空间)
   存储位置  /var/lib/kubelet/pods/<podUID>/volumes/kubernetes.io~empty-dir/<name>
   生命周期  与 Pod 绑定,Pod 删除 → 数据丢
   适用     多容器共享文件;临时缓存
   特点     可设 medium: Memory 用 tmpfs(放内存)

hostPath (绑定节点目录)
   存储位置  Node 上指定路径
   生命周期  Pod 删了文件还在;Pod 漂到其他 Node 就读不到了
   适用     单机临时调试;访问宿主机数据(/var/log、/var/run/docker.sock)
   风险     绕过容器隔离,有安全/移植风险
```

**⚠ 原笔记纠错(关键点 6):** 原文说"hostPath 是持久存储"。**不准确:**
- hostPath 数据 **只在那个 Node 上**;Pod 因为重新调度跑到别的 Node,数据就"看不到"了;
- 真正跨节点持久化要用 **PV/PVC + 网络存储(NFS / Ceph / 云盘)**;
- hostPath 在生产里几乎只用于"读宿主机本身的东西"(/var/log/、/sys 等)。

---

## 七、调度策略

### 7.1 nodeSelector(简单)

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

### 7.2 nodeAffinity(灵活)

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:   # 硬约束
        nodeSelectorTerms:
        - matchExpressions:
          - {key: zone, operator: In, values: [cn-east-1a]}
      preferredDuringSchedulingIgnoredDuringExecution:  # 软偏好
      - weight: 50
        preference:
          matchExpressions:
          - {key: disktype, operator: In, values: [ssd]}
```

字段名含义:
- `requiredDuringScheduling`:**调度时必须满足**
- `IgnoredDuringExecution`:**已运行的 Pod 不会因 Node label 变化被驱逐**(只在调度那一刻判断)

### 7.3 Taints & Tolerations

```
Node 打污点:    kubectl taint node node-1 key=value:NoSchedule
                                                 ├─ NoSchedule      新 Pod 不可调度
                                                 ├─ PreferNoSchedule 尽量不调度
                                                 └─ NoExecute       已运行的也被驱逐

Pod 加容忍:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

**⚠ 原笔记纠错(关键点 7):** 原笔记只讲了 nodeSelector,没提 Taint。
- nodeSelector / nodeAffinity 是 **Pod 主动选 Node**;
- Taint / Toleration 是 **Node 主动拒绝 Pod**;
- 两套机制是 **正交** 的,生产同时使用(如 master 节点默认带 `node-role.kubernetes.io/control-plane:NoSchedule` 污点,普通 Pod 调度不上去)。

---

## 八、DaemonSet 与无 Service 的特殊用法

DaemonSet 保证 **每个匹配的 Node 上恰好一个 Pod**,常用于:
- 日志采集:fluentd / filebeat
- 监控:node-exporter
- 网络插件:calico-node、flannel
- CSI 节点代理

新 Node 加入 → DaemonSetController 自动在其上创建 Pod;Node 移除 → 该 Pod 也消失。**不需要 replicas 字段**。

---

## 九、关键纠错清单(汇总)

| # | 原笔记表述 | 正确表述 |
| --- | --- | --- |
| 1 | "Deployment 直接管 Pod,不再用 RS" | Deployment **始终通过 RS** 管 Pod,每次更新新建 RS |
| 2 | "回滚 = 新建 RS 部署旧版本" | 旧 RS 一直存(replicas=0),回滚只是调副本数 |
| 3 | "nodePort 是宿主机端口映射" | 是 iptables/IPVS 规则,任意 Node 接到流量都会 DNAT 到后端 Pod |
| 4 | "kube-proxy 做实际转发" | 只下发规则,转发由内核 netfilter/IPVS 完成 |
| 5 | 只提 Endpoints | 1.21+ kube-proxy 默认走 **EndpointSlice**,Endpoints 已是兼容对象 |
| 6 | "hostPath 是持久存储" | 仅绑定单 Node 目录,Pod 漂走数据看不到;生产应用 PV/PVC |
| 7 | 未讲 Taint/Toleration | 与 nodeAffinity 正交;master 默认带污点 |
| 8 | "containerPort 必须写,Service 才能找到容器" | targetPort 直接写数字即可;containerPort 仅为名字引用入口 |
| 9 | "ClusterIP 在每个 Node 上能 ping" | ClusterIP **不能 ping**(没有真实接口),只能用对应协议+端口访问 |

---
