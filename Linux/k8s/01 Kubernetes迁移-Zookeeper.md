# Kubernetes中间件迁移-

## 目录

  - [2.2 zoo.cfg](#2.2-zoo.cfg)
- [1.本地搭建ZK集群](#1.本地搭建zk集群)
  - [1.0 环境说明](#1.0-环境说明)
  - [1.1 Node1节点](#1.1-node1节点)
  - [1.2 Node2节点](#1.2-node2节点)
  - [1.3 Node3节点](#1.3-node3节点)
  - [1.4 集群状态检查](#1.4-集群状态检查)
- [2.制作ZK集群镜像](#2.制作zk集群镜像)
  - [2.1 Dockerfile](#2.1-dockerfile)
  - [2.3 entrypoint](#2.3-entrypoint)
  - [2.4 构建镜像并推送仓库](#2.4-构建镜像并推送仓库)
- [3.迁移ZK集群至K8S](#3.迁移zk集群至k8s)
  - [3.1 迁移ZK思路分析](#3.1-迁移zk思路分析)
  - [3.2 创建headless](#3.2-创建headless)
  - [3.3 创建StatefulSet](#3.3-创建statefulset)
  - [3.4 检查Zookeeper集群](#3.4-检查zookeeper集群)

Zookeeper

Kubernetes中间件迁移-Zookeeper

### 2.2 zoo.cfg

## 1.本地搭建ZK集群

### 1.0 环境说明

IP地址 主机名 称 系统版本 内核版本 CPU 内 存

10.0.0.204

node1 CentOS7.9

```
3.10.0-

1160.el7.x86_64 1Core 2G
```
10.0.0.205

node2 CentOS7.9

```
3.10.0-

1160.el7.x86_64 1Core 2G
```
10.0.0.206

node3 CentOS7.9

```
3.10.0-

1160.el7.x86_64 1Core 2G
```
### 1.1 Node1节点

```
1、安装java环境 [root@node01 ~]# yum install java -y 2、下载并解压Zookeeper [root@node01 ~]# wget https:Վˌdlcdn.apache.org/zookeeper/zookeeper- 3.8.0/apache-zookeeper-3.8.0-bin.tar.gz [root@node01 ~]# tar xf apache-zookeeper-3.8.0- bin.tar.gz -C /opt [root@node01 ~]# ln -s /opt/apache-zookeeper-

3.8.0-bin/ /opt/zookeeper
```
3、修改配置 [root@node01 ~]# cat /opt/zookeeper/conf/zoo.cfg # 服务器之间或客户端与服务器之间维持心跳的时间间隔 tickTime以毫秒为单位。 tickTime=2000

集群中的follower服务器(F)与leader服务器(L)之间的初 始连接心跳数 10* tickTime initLimit=10 # 集群中的follower服务器与leader服务器之间请求和应答 之间能容忍的最多心跳数 5 * tickTime syncLimit=5

数据保存目录 dataDir=Վʡ/data # 日志保存目录 dataLogDir=Վʡ/logs # 客户端连接端口 clientPort=2181 # 客户端最大连接数。# 根据自己实际情况设置，默认为60个 maxClientCnxns=60 # 客户端获取 zookeeper 服务的当前状态及相关信息 4lw.commands.whitelist=* # 三个接点配置，格式为： server.服务编号=服务地址、LF 通信端口、选举端口 server.1=10.0.0.204:2888:3888 server.2=10.0.0.205:2888:3888 server.3=10.0.0.206:2888:3888 4、创建数据目录 [root@node01 ~]# mkdir Վʡ/data

```
5、创建节点标记ID [root@node01 ~]# mkdir /opt/zookeeper/data [root@node01 ~]#
echo "1" > /opt/zookeeper/data/myid 6、启动Zookeeper [root@node01 ~]# cd /opt/zookeeper/bin/ [root@node01 bin]# ./zkServer.sh start
```
### 1.2 Node2节点

```
1、安装java环境 [root@node02 ~]# yum install java -y 1、下载并解压Zookeeper [root@node02 ~]# wget https:Վˌdlcdn.apache.org/zookeeper/zookeeper- 3.8.0/apache-zookeeper-3.8.0-bin.tar.gz [root@node02 ~]# tar xf apache-zookeeper-3.8.0- bin.tar.gz -C /opt [root@node02 ~]# ln -s /opt/apache-zookeeper-

3.8.0-bin/ /opt/zookeeper

2、修改配置 [root@node02 ~]# cat /opt/zookeeper/conf/zoo.cfg
```
服务器之间或客户端与服务器之间维持心跳的时间间隔 tickTime以毫秒为单位。 tickTime=2000 # 集群中的follower服务器(F)与leader服务器(L)之间的初 始连接心跳数 10* tickTime initLimit=10 # 集群中的follower服务器与leader服务器之间请求和应答 之间能容忍的最多心跳数 5 * tickTime syncLimit=5

数据保存目录 dataDir=Վʡ/data # 日志保存目录 dataLogDir=Վʡ/logs # 客户端连接端口 clientPort=2181 # 客户端最大连接数。# 根据自己实际情况设置，默认为60个 maxClientCnxns=60 # 客户端获取 zookeeper 服务的当前状态及相关信息 4lw.commands.whitelist=* # 三个接点配置，格式为： server.服务编号=服务地址、LF 通信端口、选举端口 server.1=10.0.0.204:2888:3888 server.2=10.0.0.205:2888:3888 server.3=10.0.0.206:2888:3888

```
3、创建数据目录 [root@node02 ~]# mkdir /opt/zookeeper/data 4、创建节点标记ID [root@node02 ~]#
echo "2" > /opt/zookeeper/data/myid 5、启动Zookeeper [root@node02 ~]# cd /opt/zookeeper/bin/ [root@node02 bin]# ./zkServer.sh start
```
### 1.3 Node3节点

```
1、安装java环境 [root@node03 ~]# yum install java -y 2、下载并解压Zookeeper [root@node03 ~]# wget https:Վˌdlcdn.apache.org/zookeeper/zookeeper- 3.8.0/apache-zookeeper-3.8.0-bin.tar.gz [root@node03 ~]# tar xf apache-zookeeper-3.8.0- bin.tar.gz -C /opt [root@node03 ~]# ln -s /opt/apache-zookeeper-

3.8.0-bin/ /opt/zookeeper
```
3、修改配置

```
[root@node03 ~]# cat /opt/zookeeper/conf/zoo.cfg # 服务器之间或客户端与服务器之间维持心跳的时间间隔 tickTime以毫秒为单位。 tickTime=2000 # 集群中的follower服务器(F)与leader服务器(L)之间的初 始连接心跳数 10* tickTime initLimit=10 # 集群中的follower服务器与leader服务器之间请求和应答 之间能容忍的最多心跳数 5 * tickTime syncLimit=5
```
数据保存目录 dataDir=Վʡ/data # 日志保存目录 dataLogDir=Վʡ/logs # 客户端连接端口 clientPort=2181 # 客户端最大连接数。# 根据自己实际情况设置，默认为60个 maxClientCnxns=60 # 客户端获取 zookeeper 服务的当前状态及相关信息 4lw.commands.whitelist=* # 三个接点配置，格式为： server.服务编号=服务地址、LF 通信端口、选举端口 server.1=10.0.0.204:2888:3888

```
server.2=10.0.0.205:2888:3888 server.3=10.0.0.206:2888:3888 4、创建数据目录 [root@node03 ~]# mkdir /opt/zookeeper/data 5、创建节点标记ID [root@node03 ~]#
echo "3" > /opt/zookeeper/data/myid 6、启动Zookeeper [root@node03 ~]# cd /opt/zookeeper/bin/ [root@node03 bin]# ./zkServer.sh start
```
### 1.4 集群状态检查

```
[root@node01 bin]# ./zkServer.sh status Mode: follower [root@node02 bin]# ./zkServer.sh status Mode: leader [root@node03 bin]# ./zkServer.sh status Mode: follower
```
## 2.制作ZK集群镜像

### 2.1 Dockerfile

```
[root@node03 zookeeper]# cat Dockerfile FROM openjdk:8-jre # 1、调整时区 RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime ՎҐ \ echo 'Asia/Shanghai' >/etc/timezone # 2、拷贝Zookeeper及配置文件 ENV VERSION=3.8.0 ADD ./apache-zookeeper-${VERSION}-bin.tar.gz / ADD ./zoo.cfg /apache-zookeeper-${VERSION}- bin/conf # 3、对Zookeeper进行重命名 RUN mv /apache-zookeeper-${VERSION}-bin /zookeeper # 4、拷贝Entrypoint启动脚本 ADD ./entrypoint.sh /entrypoint.sh # 5、设定ZK对外暴露的端口；[客户端端口,LF通信端口,选举 端口] EXPOSE 2181 2888 3888 CMD ["/bin/bash","/entrypoint.sh"]
```
2.2 zoo.cfg [root@node03 zookeeper]# cat zoo.cfg # 服务器之间或客户端与服务器之间维持心跳的时间间隔 tickTime以毫秒为单位。 tickTime={ZOOK_TICK_TIME} # 集群中的follower服务器(F)与leader服务器(L)之间的初 始连接心跳数 10* tickTime initLimit={ZOOK_INIT_LIMIT} # 集群中的follower服务器与leader服务器之间请求和应答 之间能容忍的最多心跳数 5 * tickTime syncLimit={ZOOK_SYNC_LIMIT}

数据保存目录 dataDir={ZOOK_DATA_DIR} # 日志保存目录 dataLogDir={ZOOK_LOG_DIR} # 客户端连接端口 clientPort={ZOOK_CLIENT_PORT} # 客户端最大连接数。# 根据自己实际情况设置，默认为60个 maxClientCnxns={ZOOK_MAX_CLIENT_CNXNS} # 客户端获取 zookeeper 服务的当前状态及相关信息 4lw.commands.whitelist=* # 集群节点地址：格式为： server.服务编号=服务地址、LF 通信端口、选举端口

不建议将地址写死配置文件，建议通过entrypoint脚本传递 进来

### 2.3 entrypoint

```
[root@node03 zookeeper]# cat entrypoint.sh # 1、定义zk相关变量 ZOOK_BIN_DIR=/zookeeper/bin ZOOK_CONF_DIR=/zookeeper/conf/zoo.cfg # 2、生成zookeeper配置文件,如果需要可以在K8s的脚本中 定义env环境变量传入参数 sed -i s# {ZOOK_TICK_TIME}#${ZOOK_TICK_TIME:-2000}#g ${ZOOK_CONF_DIR} sed -i s# {ZOOK_INIT_LIMIT}#${ZOOK_INIT_LIMIT:-10}#g ${ZOOK_CONF_DIR} sed -i s# {ZOOK_SYNC_LIMIT}#${ZOOK_SYNC_LIMIT:-5}#g ${ZOOK_CONF_DIR} sed -i s# {ZOOK_DATA_DIR}#${ZOOK_DATA_DIR:-/data}#g ${ZOOK_CONF_DIR} sed -i s# {ZOOK_LOG_DIR}#${ZOOK_LOG_DIR:-/logs}#g ${ZOOK_CONF_DIR} sed -i s# {ZOOK_CLIENT_PORT}#${ZOOK_CLIENT_PORT:-2181}#g ${ZOOK_CONF_DIR}

sed -i s# {ZOOK_MAX_CLIENT_CNXNS}#${ZOOK_MAX_CLIENT_CNXNS :-60}#g ${ZOOK_CONF_DIR} # 3、后期通过ENV的方式注入ZK的地址，然后对传递的地址进 行遍历循环，最后追加至配置文件； for server in ${ZOOK_SERVERS:- server.1=localhost:2181:2888:3888} do echo ${server} ՎҴ ${ZOOK_CONF_DIR} done # 4、建立myid文件，通过pod的主机的名称提取，这就要求采 用StatefulSet方式来编排，否则无法提取到匹配的主机名 ZOOK_MYID=$(( $(hostname | sed 's#.*-Վˁg') + 1 )) echo "${ZOOK_MYID:-99}" > ${ZOOK_DATA_DIR:-/data}/myid # 5、前台运行zookeeper cd ${ZOOK_BIN_DIR} ./zkServer.sh start-foreground
```
### 2.4 构建镜像并推送仓库

```
[root@node03 zookeeper]# docker build -t harbor.oldxu.net/base/zookeeper:3.8.0 . [root@node03 zookeeper]# docker push harbor.oldxu.net/base/zookeeper:3.8.0
```
## 3.迁移ZK集群至K8S

### 3.1 迁移ZK思路分析

1、Zookeeper属于有状态服务； 2、Zookeeper集群存在角色之分； 2、Zookeeper集群每个节点都需要存储自己的数据； 4、Zookeeper集群每个节点都需要有一个唯一的地址；

### 3.2 创建headless

```
[root@master zookeeper]# cat 01-zookeeper- headless.yaml apiVersion: v1 kind: Service metadata: name: zk-svc spec: clusterIP: None selector: app: zk ports:

- name: client

port: 2181 targetPort: 2181

- name: leader-fllow

port: 2888 targetPort: 2888

- name: selection

port: 3888 targetPort: 3888
```
### 3.3 创建StatefulSet

```
[root@master zookeeper]# cat 02-zookeeper- sts.yaml apiVersion: apps/v1 kind: StatefulSet metadata: name: zookeeper spec: serviceName: "zk-svc" replicas: 3 selector: matchLabels: app: zk template: metadata: labels: app: zk spec: affinity:                         # 避免 Pod运行到同一个节点上了 podAntiAffinity:

requiredDuringSchedulingIgnoredDuringExecution:

- labelSelector:

matchExpressions:

- key: app

operator: In values: ["zk"] topologyKey: "kubernetes.io/hostname" imagePullSecrets:

- name: harbor-admin

containers:

- name: zk

image: harbor.oldxu.net/base/zookeeper:3.8.0 imagePullPolicy: Always env:

- name: ZOOK_SERVERS        # 域名地址要

写全，zk启动会报错 value: "server.1=zookeeper-0.zk- svc.default.svc.cluster.local:2888:3888 server.2=zookeeper-1.zk- svc.default.svc.cluster.local:2888:3888 server.3=zookeeper-2.zk- svc.default.svc.cluster.local:2888:3888" ports:

- name: client

containerPort: 2181

- name: leader-fllow

containerPort: 2888

- name: selection

containerPort: 3888 readinessProbe:         # 就绪探针（不就 绪则不接入流量） exec: command:

- "/bin/bash"

- "-c"

- '[[ "$(/zookeeper/bin/zkServer.sh

status 2>/dev/null|grep 2181)" ]] ՎҐ exit 0 Վҗ exit 1' initialDelaySeconds:

timeoutSeconds: 5 livenessProbe:          # 存活探针（不存 活则重启Pod） exec: command:

- "/bin/bash"

- "-c"

- '[[ "$(/zookeeper/bin/zkServer.sh

status 2>/dev/null|grep 2181)" ]] ՎҐ exit 0 Վҗ exit 1' initialDelaySeconds: 10 timeoutSeconds: 5 volumeMounts:

- name: data

mountPath: /data volumeClaimTemplates:

- metadata:

name: data spec: accessModes: ["ReadWriteMany"] storageClassName: "nfs-provisioner- storage" resources: requests: storage: 50Gi
```
### 3.4 检查Zookeeper集群

1、查看Pod以及Service

```
[root@master zookeeper]# kubectl get pod,service NAME READY   STATUS    RESTARTS        AGE pod/zookeeper-0 1/1     Running   0               5m20s pod/zookeeper-1 1/1     Running   2 (4m ago)      4m9s pod/zookeeper-2 1/1     Running   1 (5m13s ago)   5m18s NAME                 TYPE        CLUSTER-IP EXTERNAL-IP   PORT(S)                      AGE service/kubernetes   ClusterIP   10.96.0.1 <none>        443/TCP                     2d6h service/zk-svc       ClusterIP   None <none>        2181/TCP,2888/TCP,3888/TCP   30m 2、检查集群状态

[root@master zookeeper]# kubectl exec -it zookeeper-0 Վʔ /zookeeper/bin/zkServer.sh status Mode: follower [root@master zookeeper]# kubectl exec -it zookeeper-1 Վʔ /zookeeper/bin/zkServer.sh status Mode: follower [root@master zookeeper]# kubectl exec -it zookeeper-2 Վʔ /zookeeper/bin/zkServer.sh status Mode: leader 3、连接Zookeeper集群 [root@master zookeeper]# kubectl exec -it zookeeper-2 Վʔ /bin/bash root@zookeeper-2:/# root@zookeeper-2:/# /zookeeper/bin/zkCli.sh - server zk-svc  # 连接zk的serrvice地址 [zk: zk-svc(CONNECTED) 0] create /hello oldxu Created /hello [zk: zk-svc(CONNECTED) 1] get /hello oldxu
```
