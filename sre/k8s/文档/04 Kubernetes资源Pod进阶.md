# 04 Kubernetes资源Pod进阶

## 目录

- [1.Pod资源限制](#1.pod资源限制)
  - [1.1 什么是资源限制](#1.1-什么是资源限制)
  - [1.2 如何实现资源限制](#1.2-如何实现资源限制)
  - [1.3 资源限制的⽬的与意义](#1.3-资源限制的的与意义)
- [2.资源限制单位换算](#2.资源限制单位换算)
  - [2.1 CPU限制单位](#2.1-cpu限制单位)
  - [2.2 内存分配单位](#2.2-内存分配单位)
- [3.CPU资源限制实践](#3.cpu资源限制实践)
  - [3.1 设置容器的CPU请求和限制](#3.1-设置容器的cpu请求和限制)
  - [3.2 设置超过节点的CPU请求](#3.2-设置超过节点的cpu请求)
  - [3.3 如果不指定CPU的limits](#3.3-如果不指定cpu的limits)
- [4.内存资源限制实践](#4.内存资源限制实践)
  - [4.1 设置容器的内存请求和限制](#4.1-设置容器的内存请求和限制)
  - [4.2 运⾏超过容器内存限制的应⽤](#4.2-运超过容器内存限制的应)
  - [4.3 超过节点的内存分配](#4.3-超过节点的内存分配)
  - [4.4 如果没有指定内存限制](#4.4-如果没有指定内存限制)
- [5.Pod服务质量QoS](#5.pod服务质量qos)
  - [5.1 什么是QoS](#5.1-什么是qos)
  - [5.2 QoS类别](#5.2-qos类别)
  - [5.3 创建Guaranteed的Pod](#5.3-创建guaranteed的pod)
  - [5.3 创建Burstable的Pod](#5.3-创建burstable的pod)
  - [5.3 创建BestEffort的Pod](#5.3-创建besteffort的pod)
  - [5.4 创建多容器Pod](#5.4-创建多容器pod)
- [6.Downward API](#6.downward-api)
  - [5.1 什么是DownwardAPI](#5.1-什么是downwardapi)
  - [5.2 可注⼊的元数据信息](#5.2-可注的元数据信息)
  - [5.3 环境变量⽅式注⼊元数据](#5.3-环境变量式注元数据)
  - [5.4 存储卷⽅式注⼊元数据](#5.4-存储卷式注元数据)
  - [5.3 为注册服务注⼊Pod名称](#5.3-为注册服务注pod名称)
  - [5.5 为Tomcat注⼊堆内存限制](#5.5-为tomcat注堆内存限制)

www.xuliangwei.com

## 1.Pod资源限制

### 1.1 什么是资源限制

在Kubernetes集群中，为了使系统能够稳定的运⾏，通常会对Pod的资 源使⽤量进⾏限制。 在Kubernetes集群中，如果有⼀个程序出现异常，并占⽤⼤量的系统资 源。如果未对该Pod进⾏资源限制的话，可能会影响其他的Pod正常运 ⾏，从⽽造成业务的不稳定性。

### 1.2 如何实现资源限制

```
www.xuliangwei.com Kubernetes通过 Requests 和 Limits 字段来实现对Pod的资源进 ⾏限制 Requests：启动 Pod 时申请分配的资源⼤⼩；（Pod在调度的时 候requests⽐较重要） Limits：限制 Pod 运⾏时最⼤可⽤的资源⼤⼩；（Pod在运⾏时 limits⽐较重要） spec.containers[].resources.request.cpu spec.containers[].resources.request.memory spec.containers[].resources.limits.cpu spec.containers[].resources.limits.memory
```
### 1.3 资源限制的⽬的与意义

CPU：为集群中运⾏的容器配置CPU请求和限制，可以有效利⽤集群上可 ⽤的 CPU 资源。 设置 Pod CPU请求 设定在较低的数值，可以使 Pod 更有机会被 调度。

设置 CPU 限制⼤于 CPU 请求，可以完成如下两件事： 1、当 Pod 碰到⼀些突发负载时，它可以合理利⽤可⽤的 CPU 资源。 当 Pod 在突发流量期间 可使⽤的CPU被限制为合理的数值， 从⽽可以避免影响其他Pod的正常运⾏；

内存：为集群中运⾏的容器配置内存请求和限制，可以有效利⽤集群节点 上可⽤的内存资源。 通过将Pod 的内存请求设定在较低的数值，可以使 Pod 更有机会 被调度。 通过让内存限制⼤于内存请求，可以完成如下两件事： 当 Pod 碰到⼀些突发负载时，可以更好的利⽤其主机上的可⽤ 内存。 当 Pod 在突发负载期间可使⽤的内存被限制为合理的数值，从 ⽽可以避免影响其他Pod的运⾏。 www.xuliangwei.com

## 2.资源限制单位换算

### 2.1 CPU限制单位

1核CPU等于1000毫核，当定义容器为0.5时，所能⽤到的CPU资源时1核 ⼼CPU的⼀半，对于 CPU 资源单位，表达式 0.1 等价于表达式 100m，可以看作 100 millicpu。

```
1 核⼼ =  1000 millicpu （1 Core = 1000m）
```
0.5 核 =  500  millicpu （0.5 Core = 500m） 举例：当我们有1个物理CPU，16核⼼，如果某个Pod最多使⽤⼀半的核 ⼼数，则表达式可以写⼊如下两种： limits.cpu:

limits.cpu: 8000m 计算公式：(16000*0.5=8000m) 注意：Kubernetes不允许设置精度⼩于 1m 的CPU资源。因此当 CPU 单位⼩于1时，只能使⽤毫核来表示。 例如：期望使⽤1个CPU的0.5%，应该写 5m ⽽不是 0.005

### 2.2 内存分配单位

```
内存的基本单位是字节数(Bytes)，也可以加上国际单位，⼗进制的 E、P、T、G、M，K、m，或⼆进制的 Ei、Pi、Ti、Gi、Mi、Ki。 1MB = 1000 KB = 1000000 Bytes 1Mi = 1024 KB = 1048576 bytes www.xuliangwei.com
```
## 3.CPU资源限制实践

```
metrics-server
```
### 3.1 设置容器的CPU请求和限制

```
1.创建⼀个具有⼀个容器的 Pod。容器将请求 0.5 个 CPU，最多限制 使⽤ 1 个 CPU。 [root@master ~]# vim cpu-requests-limits.yaml apiVersion: v1 kind: Pod metadata: name: cpu-demo spec: containers:

- name: cpu-demo-ctr

image: vish/stress
```
args:         # 容器启动命令，容器尝试使⽤2核CPU

```
- -cpus

- "2"

resources: requests:       # 限制启动Pod时最多申请0.5核的CPU cpu: "500m" limits:       # 限制Pod最多使⽤1核CPU cpu: "1000m" 2.查看Pod详细信息，输出显示 Pod 中的⼀个容器的 CPU 请求为 500 milli CPU，并且 CPU 限制为 1 个 CPU。 [root@master ~]# kubectl get pod cpu-demo -o yaml resources: www.xuliangwei.com limits: cpu: "1" requests: cpu: 500m 3.检查资源限制情况，容器配置为尝试使⽤2个CPU，但是容器只被允许 最⼤使⽤1个CPU。 所以容器的 CPU ⽤量受到限制 [root@master ~]# kubectl apply -f https:!"linux.oldxu.net/metrics-server.yaml [root@master ~]# kubectl top pod cpu-demo NAME       CPU(cores)   MEMORY(bytes) cpu-demo   913m         0Mi
```
### 3.2 设置超过节点的CPU请求

1.创建⼀个 Pod，设置该 Pod 中容器的请求为 100 核，这个值会⼤ 于集群中的任何⼀个节点。

```
[root@master ~]# cat cpu-requests-limits-2.yaml apiVersion: v1 kind: Pod metadata: name: cpu-demo-2 spec: containers:

- name: cpu-demo-ctr-2

image: vish/stress args:

- -cpus

- "2"

resources: requests: www.xuliangwei.com cpu: "100" limits: cpu: "100" 2.查看该 Pod 的状态，输出显示 Pod 状态为 Pending。也就是说， Pod 未被调度到任何节点上运⾏ [root@master pod-request]# kubectl get pod NAME                         READY   STATUS RESTARTS          AGE cpu-demo-2                   0/1     Pending
```
3.查看该 Pod 的详细信息以及事件，输出显示由于节点上的 CPU 资源 不⾜，⽆法调度容器。

```
[root@master ~]# kubectl describe pod cpu-demo-2

Events: Type     Reason            Age From               Message ----     ------            ----                !!#

-               -------

Warning  FailedScheduling  49s (x2 over 116s) default-scheduler  0/3 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 2 Insufficient cpu.

www.xuliangwei.com [root@master ~]# kubectl delete pod cpu-demo-2 pod "cpu-demo-2" deleted
```
### 3.3 如果不指定CPU的limits

如果没有为容器指定 CPU 限制，那么容器在可以使⽤的 CPU 资源是没 有上限。因⽽可以使⽤所在节点上所有的可⽤ CPU 资源，这样可能会造 成某⼀个Pod占⽤了⼤量的CPU时间，可能会影响其他的Pod正常运⾏， 从⽽造成业务的不稳定性。 这个也不⽤担⼼，在Kubernetes中，可以通过 LimitRange ⾃动为容 器设定，所使⽤的CPU资源和内存资源最⼤最⼩值。

## 4.内存资源限制实践

### 4.1 设置容器的内存请求和限制

```
1.创建⼀个拥有⼀个容器的Pod。 容器将会请求 100 MiB 内存，并且 内存会被限制在 200 MiB 以内。 [root@master ~]# cat memory-request-limit.yaml apiVersion: v1 kind: Pod metadata: name: memory-demo spec: containers:

- name: memory-demo-ctr

image: polinux/stress command: ["stress"] args: ["!$vm", "1", "!$vm-bytes", "150M", "!$vm- www.xuliangwei.com hang", "1" ] # 告知容器尝试分配 150 MiB 内存 resources: requests: memory: "100Mi" limits: memory: "200Mi" 2.检查Pod，结果显示该 Pod 中容器的内存请求为 100 MiB，内存限 制为 200 MiB。 [root@master ~]# kubectl get  pod memory-demo -o yaml resources: limits: memory: 200Mi requests: memory: 100Mi

3.获取该 Pod 的指标数据：输出结果显示Pod正在使⽤的内存约为150 MiB。这⼤于Pod请求的100MiB，但⼜在Pod 限制的200MiB之内。 [root@master ~]# kubectl top pod memory-demo NAME          CPU(cores)   MEMORY(bytes) memory-demo   46m          150Mi

[root@master ~]# kubectl delete pod memory-demo pod "memory-demo" deleted
```
### 4.2 运⾏超过容器内存限制的应⽤

www.xuliangwei.com 当节点拥有⾜够的可⽤内存时，容器可以使⽤其请求的内存。但是，容器 不允许使⽤超过其限制的内存。如果容器分配的内存超过其限制，该容器 会成为被终⽌的候选容器。如果容器继续消耗超出其限制的内存，则终⽌ 容器。如果终⽌的容器可以被重启，则 kubelet 会重新启动它。 1.创建⼀个 Pod，其拥有⼀个容器，该容器的内存请求为 100MiB，内 存限制为 200MiB，尝试分配超出其限制的内存。 [root@master ~]# cat memory-request-limit-2.yaml apiVersion: v1 kind: Pod metadata: name: memory-demo-2 spec: containers:

```
- name: memory-demo-2-ctr

image: polinux/stress command: ["stress"]   # 容器会尝试分配 250 MiB 内 存，这远⾼于 100 MiB 的限制。

args: ["!$vm", "1", "!$vm-bytes", "250M", "!$vm- hang", "1" ]  # 模拟1个进程产⽣250M内存 resources: requests: memory: "100Mi" limits: memory: "200Mi" 2.查看Pod，此时，容器可能正在运⾏或被杀死。重复前⾯的命令，直到 容器被杀掉： [root@master ~]# kubectl get pod NAME            READY   STATUS      RESTARTS   AGE memory-demo-2   0/1     OOMKilled   0          13s www.xuliangwei.com 3.查看容器更详细的信息，其输出结果为，内存溢出（OOM），容器已被 杀掉： [root@master ~]# kubectl get pod memory-demo-2 -o yaml lastState: terminated: containerID: docker:!"43d2278dacdac6cea191c6c04f2147025128ce45c96 1ab0c2d366840d0bc5e40 exitCode: 1 finishedAt: "2022-04-11T15:45:17Z" reason: OOMKilled startedAt: "2022-04-11T15:45:17Z"

[root@master ~]# kubectl delete pod memory-demo-2 pod "memory-demo-2" deleted
```
### 4.3 超过节点的内存分配

Pod 的调度基于请求。只有当节点拥有⾜够满⾜ Pod 内存请求的内存 时，才会将 Pod 调度⾄节点上运⾏。 1.创建⼀个 Pod，其拥有⼀个请求 1000 GiB 内存的容器，这应该超 过了集群中任何⼀台节点所拥有的内存。 [root@master pod-request]# cat memory-request-limit-

3.yaml

```
apiVersion: v1 www.xuliangwei.com kind: Pod metadata: name: memory-demo-3 spec: containers:

- name: memory-demo-3-ctr

image: polinux/stress command: ["stress"] args: ["!$vm", "1", "!$vm-bytes", "250M", "!$vm- hang", "1" ] # 告知容器尝试分配 250 MiB 内存 resources: requests: memory: "100Gi" limits: memory: "200Gi" 2.查看 Pod 状态，发现处于 PENDING 状态。 这意味着，该 Pod 没 有被调度⾄任何节点上运⾏

[root@master ~]# kubectl get pod NAME            READY   STATUS    RESTARTS   AGE memory-demo-3   0/1     Pending   0          46s 3.查看Pod详情，输出结果显示，由于节点内存不⾜，该容器⽆法被调 度： [root@master ~]# kubectl describe pod memory-demo-3 Events: Type     Reason            Age   From Message ----     ------            ----  ---- ------- Warning  FailedScheduling  78s   default-scheduler www.xuliangwei.com 0/3 nodes are available: 1 node(s) had taint {node- role.kubernetes.io/master: }, that the pod didn't tolerate, 2 Insufficient memory.

[root@master ~]# kubectl delete pod memory-demo-3 pod "memory-demo-3" deleted
```
### 4.4 如果没有指定内存限制

如果没有为容器指定内存限制，容器可⽆限制地使⽤其所在节点的所有可 ⽤内存，进⽽可能导致该节点调⽤OOM Killer。 此外，如果发⽣OOM Kill，没有配置资源限制的容器将被杀掉的可⾏性更⼤。 不⽤担⼼，在Kubernetes中，可以通过[LimitRange]⾃动为其容器设 定，所使⽤的内存资源最⼤最⼩值。

## 5.Pod服务质量QoS

### 5.1 什么是QoS

QoS（Quality of Service），可译为 "服务质量等级"，或者译作 "服务质量保证"，是作⽤在 Pod 上的⼀个配置，当 Kubernetes 创建 ⼀个 Pod 时，它就会给这个 Pod 分配⼀个 QoS 等级。 在Kubernetes的环境中，Kubernetes允许节点的Pod过载使⽤资源， 这意味着节点⽆法同时满⾜所有Pod以过载的⽅式运⾏。因此在内存资源 紧缺的情况下，Kubernetes需要借助Pod对象的服务质量和优先级等完 成判定，进⽽挑选对应的Pod杀死。Kubernetes根据pod的Requests和 Limits属性，把Pod对象归类为三类 BestEffort、BurStable、 Guaranteed。 www.xuliangwei.com

### 5.2 QoS类别

Guaranteed：Pod对象为每个容器都设置了CPU资源需求和资源限 制，且两者的值相同；还同时为每个容器设置了内存需求与内存限 制，并且两者的值相同。这类Pod对象具有最⾼级别服务质量。 Burstable：⾄少有⼀个容器设置了CPU或内存资源Requests属 性，但不满⾜Guaranteed，这类Pod具有中级服务质量。 BestEffort：没有为任何容器设置Requests和Limits属性，这类 Pod对象服务质量是最低级别。

当 Kubernetes 集群内存资源紧缺，优先杀死BestEffort类别的容 器，因为系统不为该类资源提供任何服务保证，但此类资源最⼤的好处就 是能够尽可能的使⽤资源。 如果系统中没有BestEffort类别的容器，接下来就轮到Burstable类别 的容器，如果有多个Burstable类别的容器，就看谁的内存资源占⽤ 多，就优先⼲掉谁。⽐如A容器申请1G内存资源，实际使⽤了95%，⽽B容 器申请了2G内存资源，实际使⽤了80%，但任然会优先⼲掉A容器，虽然A

容器的⽤量少，但与⾃身的Requests值相⽐，它的占⽐要⼤于B容器。 对于Guaranteed类别的容器拥有最⾼优先级，它们不会被杀死，除⾮其 内存资源需求超限，或者OOM时没有其他更低优先级的Pod对象存在，才 会⼲掉Guaranteed类容器。

### 5.3 创建Guaranteed的Pod

对于 QoS 类为 Guaranteed 的 Pod： Pod 中的每个容器都必须指定内存请求和内存限制，且Pod中每个容 器内存请求必须等于内存限制。 od 中的每个容器都必须指定CPU请求和CPU限制，且Pod中每个容器 CPU请求必须等于CPU限制。 1.创建⼀个Pod，容器设置了内存请求和内存限制，值都是200MiB。容 www.xuliangwei.com 器设置了CPU请求和CPU限制，值都是700 milliCPU [root@master ~]# cat pod-qos-guaranteed.yaml apiVersion: v1 kind: Pod metadata: name: pod-qos-guarantee spec: containers:

```
image: nginx resources: requests: cpu: "700m" memory: "200Mi" limits: cpu: "700m" memory: "200Mi"

spec: containers: !!% Limits: cpu:     700m memory:  200Mi Requests: cpu:      700m memory:   200Mi !!% status: qosClass: Guaranteed www.xuliangwei.com
```
### 5.3 创建Burstable的Pod

如果满⾜下⾯条件，将会指定 Pod 的 QoS 类为 Burstable： Pod 不符合 Guaranteed QoS 类的标准。 Pod 中⾄少⼀个容器指定了，内存或 CPU 的请求或限制。 1.创建⼀个Pod，容器设置了内存请求 100 MiB，以及内存限制 200 MiB。 [root@master ~]# cat pod-qos-burstable.yaml apiVersion: v1 kind: Pod metadata: name: pod-qos-burstable spec: containers:

```
image: nginx

resources: requests: memory: "100Mi" limits: memory: "200Mi"

spec: containers: !!% Limits: memory:  200Mi Requests: www.xuliangwei.com memory:     100Mi !!% status: qosClass: Burstable
```
### 5.3 创建BestEffort的Pod

对于 QoS 类为 BestEffort 的 Pod，Pod 中的容器必须没有设置内 存和 CPU 限制或请求。 1.创建⼀个Pod，容器没有设置内存和 CPU 限制或请求。

```
[root@master tmp]#  cat pod-qos-besteffort.yaml apiVersion: v1 kind: Pod metadata: name: pod-qos-besteffort spec: containers:

image: nginx

spec: containers: www.xuliangwei.com !!% resources: {} !!% status: qosClass: BestEffort
```
### 5.4 创建多容器Pod

1.创建⼀个Pod，⼀个容器指定了内存请求 200 MiB。 另外⼀个容器 没有指定任何请求和限制。此 Pod 满⾜ Burstable QoS 类的标准。 但它不满⾜ Guaranteed QoS 类标准，因为它的⼀个容器设有内存请 求。 [root@master ~]#  cat pod-qos-mutil.yaml apiVersion: v1 kind: Pod metadata: name: pod-qos-mutil spec:

```
containers:

image: nginx resources: requests: memory: "100Mi"

- name: qos-demo2

image: redis

spec: www.xuliangwei.com containers: !!% name: qos-demo resources: Requests: memory:     100Mi !!% name: qos-demo2 resources: {} !!% status: qosClass: Burstable
```
## 6.Downward API

### 5.1 什么是DownwardAPI

DownwardAPI可以让容器获取Pod的相关元数据信息，⽐如Pod名称， Pod的IP，Pod的资源限制等，获取后通过env、volume的⽅式将相关的 环境信息注⼊到容器中，从⽽让容器通过这些信息，来设定容器的运⾏特 性。 例如：Nginx进程根据节点的CPU核⼼数量⾃动设定要启动的worker 进程数。 例如：JVM虚拟根据Pod的内存资源限制，来设定对应容器的堆内存 ⼤⼩。 例如：获取Pod名称，以Pod名称注册到某个服务，当Pod结束后， 调⽤prestop清理对应名称的注册信息。

### 5.2 可注⼊的元数据信息

```
使⽤ pod.spec.containers.env.valueFrom.fieldRef 可以注⼊ www.xuliangwei.com 的字段有： metadata.name：Pod对象的名称 metadata.namespace：Pod对象⾪属的名称空间 metadata.uid：Pod对象的UID metadata.labels['<KEY>']`：获取Label指定KEY对应的值 metadata.annotations['<KEY>']：获取Annotations对应 KEY的值 status.podIP：Pod对象的IP地址 status.hostIP：节点IP status.nodeName：节点名称 spec.serviceAccountName：Pod对象使⽤的ServiceAccount 资源名称 使⽤ pod.spec.containers.env.valueFrom.resourceFieldRef 可 以注⼊的字段有： requests.cpu requests.memory limits.cpu limits.memory
```
### 5.3 环境变量⽅式注⼊元数据

```
1.创建Pod容器，将Pod相关环境变量注⼊到容器中，⽐如（pod名称、 命名空间、标签、以及cpu、内存的请求和限制） [root@master tmp]# cat pod-downward.yaml apiVersion: v1 kind: Pod metadata: name: pod-downward labels: app: pod-app spec: containers: www.xuliangwei.com

- name: pod-downward

image: nginx command: ["/bin/sh", "-c" , "env"] resources: requests: cpu: "200m" memory: "32Mi" limits: cpu: "200m" memory: "64Mi" env:

- name: THIS_POD_NAME

valueFrom: fieldRef: fieldPath: metadata.name

- name: THIS_POD_NAMESPACE

valueFrom: fieldRef: fieldPath: metadata.namespace

- name: THIS_POD_APP_LABEL

valueFrom: fieldRef: fieldPath: metadata.labels['app']

- name: THIS_CPU_LIMITS

valueFrom: resourceFieldRef: resource: limits.cpu

- name: THIS_MEMORY_REQUEST

valueFrom: resourceFieldRef: resource: requests.memory #divisor: 1Mi                  # 默认显示为 字节，通过divisor调整显示单位为兆 www.xuliangwei.com

[root@master tmp]# kubectl logs pod-downward  |grep ^THIS THIS_CPU_LIMITS=1         # 200毫核，不⾜1核，则进⾏取 整。 THIS_POD_APP_LABEL=pod-app THIS_POD_NAME=pod-downward THIS_MEMORY_REQUEST=32        # 单位为兆，默认字节 THIS_POD_NAMESPACE=default
```
### 5.4 存储卷⽅式注⼊元数据

```
apiVersion: v1 kind: Pod metadata: name: pod-downward-volumes labels:

app: pod-app zone: beijing role: backend spec: containers:

- name: pod-downward

image: nginx resources: requests: cpu: "200m" memory: "32Mi" limits: cpu: "200m" memory: "64Mi" www.xuliangwei.com volumeMounts:     # 将环境变量挂载到/etc/podinfo⽬ 录中，没注⼊⼀条元数据都会产⽣⼀个⽂件。

- name: podinfo

mountPath: /etc/podinfo

volumes:

- name: podinfo

downwardAPI: items:

- path: pod_name

fieldRef: fieldPath: metadata.name

- path: pod_labels

fieldRef: fieldPath: metadata.labels

- path: pod_namespace

fieldRef:

fieldPath: metadata.namespace

- path: mem_limits

resourceFieldRef: resource: limits.memory containerName: pod-downward #divisor: 1Mi                  # 默认显示为 字节，通过divisor调整显示单位为兆

- path: mem_requests

resourceFieldRef: resource: requests.memory containerName: pod-downward #divisor: 1Mi                  # 默认显示为 www.xuliangwei.com 字节，通过divisor调整显示单位为兆
```
### 5.3 为注册服务注⼊Pod名称

```
使⽤DownwardAPI实现注册与卸载 [root@master ~]# cat 11-pod-register.yaml apiVersion: apps/v1 kind: Deployment metadata: name: pod-mysql-register    # 如果使⽤-数据库会报错 spec: replicas: 10 selector: matchLabels: app: tools

template: metadata: labels: app: tools spec: containers:

- name: register

image: oldxu3957/tools:latest imagePullPolicy: IfNotPresent command:

- "/bin/bash"

- "-c"

- |

mysql -h 10.0.0.206 -uoldxu -p123 -e www.xuliangwei.com "create database ${POD_NAME!"-/_}" sleep 99999999

env:

- name: POD_NAME

valueFrom: fieldRef: fieldPath: metadata.name

lifecycle: preStop: exec: command:

- "/bin/bash"

- "-c"

- mysql -h 10.0.0.206 -uoldxu -p123 -e

"drop database ${POD_NAME!"-/_}"
```
### 5.5 为Tomcat注⼊堆内存限制

```
默认Tomcat应⽤会使⽤Pod所在的物理节点内存，初始堆内存为1/64， 最⼤堆内存为1/4 [root@master tmp]# cat 11-pod-lifecycle-2.yaml apiVersion: v1 kind: Pod metadata: name: pod-tomcat spec: containers:

- name: tomcat-web

image: tomcat:9.0.63 www.xuliangwei.com ports:

- containerPort: 8080

env:

- name: JAVA_OPTS

value: -server -Xms${JVM_XMS} -Xmx${JVM_XMX} - XX:+UseConcMarkSweepGC # downward API

- name: JVM_XMS

valueFrom: resourceFieldRef: resource: requests.memory

- name: JVM_XMX

valueFrom: resourceFieldRef: resource: limits.memory resources: requests: memory: "250Mi" limits:

memory: "500Mi" www.xuliangwei.com
```
