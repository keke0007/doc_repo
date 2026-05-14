分布式检索引擎 ElasticSearch

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/6471f88dc6ee523c25769cdd3636a65882020a22b77718124e52dac29615caea.jpg)

# elasticsearch

# 集群管理

# 集群健康检查

Elasticsearch 的集群监控信息中包含了许多的统计数据，其中最为重要的一项就是集群健康，它在 status 字段中展示为 green、yellow 或者 red

# 查看集群状态

可以通过如下的命令查看集群的状态

```
GET/_cluster/health 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/9711b944be68b0737794c4b9c9cf64d29339741050dd7b4cb3dc310e67bb95aa.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/db18c7ed418a6385ce22b8dc9db43e997cbbecd2a349274c5570d7a099fec0e4.jpg)

# 集群状态

status 字段是我们最关心的，status 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下

- green：所有的主分片和副本分片都正常运行

- yellow：所有数据可用，但有些副本尚未分配(集群功能完全)

- red：有主分片没能正常运行。

注意: 当集群处于红色状态时，正常的分片将继续提供搜索服务，但你可能要尽快修复它。

# 分片验证

下面我们验证以下分片的使用

# 验证一个分片

首先我们通过建立索引的方式来看下什么是分片，会产生哪些变化。

# 创建索引

我们首先创建一个名为 test 的索引，让它有一个分片，我们看看结果，在 kibana 执行以下命令

```
PUT test
{
    "settings": {
    "index": {
    "number_of_shards" : "1",
    "number_of_replicas" : "0"
    }
    }
} 
```

这样我们就创建了一个分片

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/4d90782b5687a22dd3cf49a460f0aa49a4b1333810d771dc63fd277526f41b8b.jpg)

# 查看分片

我们在 cerebro 中查看结果，我们看到 test 只在 node-2 节点上：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/edd9e8b645a1d74d07ec166e48e92feb1b77ffa3b6207298d1279bf9dc0ce9e7.jpg)

# 验证两个分片

我们再来创建两个分片看看会发生上面

# 创建索引

我们再次创建两个分片的test1

```
PUT test1
{
    "settings": {
    "index": {
    "number_of_shards": "2",
    "number_of_replicas": "0"
    }
    }
} 
```

# 这样我们就创建了一个索引

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/638647985598ecaa76cbac2774ce1591c71483447ac0e60023bdf5e390f5a4fa.jpg)

# 查看分片

我们在 cerebro 中查看结果，我们看到 test1 在 node-1 和 node-3 上面

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/224fa227c79780048667c31e5a2a3cc3b40137158f45a9a68885b1e645e647d4.jpg)

如上图看到分片分别分布在两个节点上，不用想，如果是3个那么肯定均匀分布在三个节点上

# 验证四个分片

如果我们创建四个分片，多于节点数会发生什么呢

# 创建索引

我们创建四个分片的索引

```
PUT test2
{
    "settings": {
    "index": {
    "number_of_shards": "4",
    "number_of_replicas": "0"
    }
    }
} 
```

# 这样我们就创建了一个索引

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/4bdcf27fa25b7f570a6026468963cd31c1cb932ee850e557800052683e2f305a.jpg)

# 查看分片

我们发现有两个分片在 node-2 的节点上

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/21ffffc13369d7a8ce1a7374f327debaf3cb016edfb164e8e5e81c407db37609.jpg)

# 验证副本

下面我们对副本进行验证

# 验证两副本分片

# 创建索引

创建一个含有一个分片，两个副本的test3:

```
PUT test3
{
    "settings": {
    "index": {
    "number_of_shards" : "1",
    "number_of_replicas" : "2"
    }
    }
} 
```

# 这样我们就创建出来了一个分片带两个分片的索引

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/d1a89c514b193be80f9aede0d80622f2e73298f340812b656f5c5e11e74824f1.jpg)

# 查看索引

如下图所示，我们看到了3个绿色的0，其中在 node-1 节点的边框是粗体的，这个表示分片，而另外两个节点的0的边框是虚线的，这两个就是分片的副本。

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/82cad02cdc2d6fc649db52523b429f4d34dad56b3dd621a642a34979e2bcaa53.jpg)

通常我们三个节点建立两个副本就可以了，三份数据均匀得到分布在三个节点

# 验证三副本分片

如果建立三个副本会怎么样呢？

# 创建索引

创建一个分片三个副本的索引

```
PUT test4
{
    "settings": {
    "index": {
    "number_of_shards" : "1",
    "number_of_replicas" : "3"
    }
    }
} 
```

← → ⚫ 不安全 | 192.168.245.151:5601/app/dev_tools#/console

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/0a991b0e9e43fef1d37f8d7ff9cf3306ebde2feccb79ce3497dcd4a0c360a56e.jpg)

# 查看索引

如下所示我们看到多出一个Unassigned的副本，这个副本其实是多余的了，因为每个节点已经包含了分片的本身和其副本，多于这个没有意义！

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/2c95df3236b7c6c20807c614351fb398da757458ce93f3633a87b4ebe08aca2b.jpg)

并且因为多出来了一个副本无法分配，整个集群都变成了 yellow 的状态

# 分片与副本的组合

# 默认组合

如果不指定分片和副本会怎么样呢

# 创建索引

创建一个索引不指定副本和索引数量

```
PUT test5
{
    "settings": {
    "index": {
    }
    }
} 
```

← → C 不安全 | 192.168.245.151:5601/app/dev tools#/console

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/ea1122c2da65d6bbe9b1938f26a37098e149ec9d91f12d9b61cf030b99493b6d.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/b9b3fd9744cbde055463785b55fc51b9984373a13a163a29a3d09d60e5f48e17.jpg)

# 查看索引

如下图所示看到默认是有一个分片和一个副本的。

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/0b39353b4ff761f841499928f665e112a6058e61d9a40a254d2192e743ad47ab.jpg)

# 两副本两分片

下面我们试验一下两副本两分片，看看会发生什么

创建索引

下面我们创建两个分片和两个副本

```
PUT test6
{
    "settings": {
    "index": {
    "number_of_shards" : "2",
    "number_of_replicas" : "2"
    }
    }
} 
```

← → ⬆ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/f23eed7c825c97e3849b8d8c2a4b2fbdececdc0fb9726d5aa32f1ee9c659eade.jpg)

# 查看索引

我们看到下面就是两副本两分片的情况，每一个分片都有两个副本，如果再多一个副本就会无法再分配了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/957e66dd33fe00712b19e834816ac8593dce362f7ac480abcf8d742109b02c08.jpg)

# 三分片两副本

下面我们验证以下三分片两副本的情况

# 创建索引

我们创建一个三个分片两个副本的索引

```
PUT test7
{
    "settings": {
    "index": {
    "number_of_shards" : "3",
    "number_of_replicas" : "2"
    }
    }
} 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/fb49b1ea68eeeb0a884b6bfd14cd26fbbdf86759f4022c736f80489fa9377ba6.jpg)

# 查看索引

如下所示看到分片和副本都均匀的分布在每个节点上。

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/8c1ee69007cb21975318540b3c294c89cb1c2b3ebe53ecab7f51d276fcd474de.jpg)

# 故障与恢复

# 故障转移

集群的master节点会监控集群中的节点状态，如果发现有节点宕机，会立即将宕机节点的分片数据迁移到其它节点，确保数据安全，这个叫做故障转移。

# 正常集群状态

下图是正常集群的状态，node1是主节点，其它两个节点是从节点

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/03d66abb641ce0d95afb4e332276b588877a872f75aab70e138396bf3f31bb2f.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/6c47ab94fe0d3839882dc88895e530801cb843459479c356e4cd298dc1261007.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/558caa8b63fe294b6b9e92c3206c17fd90852357b9e9074e50f124b6460a3b8c.jpg)

# 节点故障

突然，node1发生了故障

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/542f427d3788ffe6eae53930fdb51adcddcda595cf461369a550c588a88d73b6.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/717be98becc024633d09ffd8924de4b58df5d3654aa549b72881a5aa5bd76fa1.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/b7ccf62b09179306b16503afb4d0bc4bd573c80f010f1f0f426b169db0ac7c54.jpg)

宕机后的第一件事，需要重新选主，例如选中了node2

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/4e23cb79120f3f173d3f41dd0a9ef893c1558acd20cfd3c9b781b496920f6e3e.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/faa79b24fa18c4f50e702d2f77e700348538b77ef57a29837b0ba1f8e981c41a.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/ef7dc7318ca06d883318b7e7d769c611eede6cb7210685c2f0fd82e99a8a46f8.jpg)

# 数据迁移

node2成为主节点后，会检测集群监控状态，发现：shard-1、shard-0没有副本节点，因此需要将node1上的数据迁移到node2、node3

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/e52cb00ab5ab7647a9d2cf83046cb2a550db041578245398e3926c45500b0660.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/50bef9e04e9e10a4d7b13588332f98f1722b65dd8b46dba96ed304afc01afcfc.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/6b528f88da7cbcd24667828bf12b29309df2315a9d3b86b1604b69fa05b42116.jpg)

# 节点故障

我们观察下如果三个节点有一个节点宕机了，上文的 test7 的分片和副本会有哪些变化

# 关闭node2节点

我们关闭 node-2 节点

# 执行关闭命令

我们暂停 node-2 节点

```
docker pause node-2 
```

# 查看索引

如下所示，原本的node-2节点变成了Unassigned

注意我标注的三个红框内的分片，这三个分片已经随着节点的宕机消息了，这就造成了数据的丢失，反观后面几个，虽然node2宕机了，但是由于我们做了分片与备份，索引仍然可以正常的工作，

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/1648c39cdc4f0db9245174b2a8183c1dfaa187cb369c0b9f732744744de4af7a.jpg)

原本在node-2的2号分片移动到了node-1节点，在使用es集群的过程中，一定要注意分片和副本的使用，保证我们整个集群的高可用性

# 关闭node3节点

我们关闭以下node-3节点看下情况

# 执行关闭命令

执行如下命令暂停node-3

```
docker pause node-3 
```

# 查看集群状态

通过 Cerebro 我们发现整个集群已经无法进行访问了

← → C 不安全 | 192.168.245.151:9100/#!/connect 更新：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/cb741664a02fb2e373d512f89332c66c9f6b734708ac74bbf8d6a2fb3e4dc795.jpg)

当集群中的节点数少于半数，将导致整个集群不可用

# 节点恢复

经过上面的宕机试验后，我们现在要对宕机的服务进行启动

# 启动node2节点

我们先启动node2节点

# 执行启动命令

执行下面的命令恢复node-2节点

docker unpause node-2

# 查看集群状态

如上发现集群已经能够恢复访问了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/cfd05ab4ad292ff17eddba09977fe8fe32b109ead0641566b62707697ee3bb61.jpg)

# 创建索引

此时我们在两个几点可用的情况下创建一个有三分片，两个副本的索引

```
PUT test8
{
    "settings": {
    "index": {
    "number_of_shards": "3",
    "number_of_replicas": "2"
    }
    }
} 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/e030aa5bf285c33fd9657b0a03faa4d3a5b1a7d2cf4b37a9e653e4c4c769fcc0.jpg)

# 查看索引状态

如下所示，分片与副本的分布没有问题，有三个副本未分配

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/7837616328fcb092233f0e74d44c2157b7a49b97470da5413c621534039a6128.jpg)

# 启动node3节点

下面我们就启动node3节点

# 执行启动命令

执行下面的命令恢复node-2节点

```
docker unpause node-3 
```

# 查看索引

所有的未分配副本移动到了node-3节点，并没有将分片移动到node-2上

← → ⬆ 不安全 | 192.168.245.151:9100/#!/overview?host=http:%2F%2F192.168.245.151:9200 更新：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/84dea51a8545a193c2d6de5f36bb2663e8c1254d1de0b9ffe22504319cc60d84.jpg)

# 扩缩容

# 服务布局

我们整体采用Docker方式进行布局，以下是我们需要部署的服务，标红色的我们需要新增的节点

| 服务名称   | 服务名称 | 开放端口 | 内存限制 |
| ---------- | -------- | -------- | -------- |
| ES-node1   | node-1   | 9200     | 1G       |
| ES-node2   | node-2   | 9201     | 1G       |
| ES-node3   | node-3   | 9202     | 1G       |
| ES-node4   | node-4   | 9203     | 1G       |
| ES-cerebro | cerebro  | 9000     | 不限     |
| kibana     |          |          |          |

# 扩容节点

# 创建节点目录

创建ES的节点目录

```
mkdir -p /tmp/data/elasticsearch/node-4/{config,plugins,data,log}
#进行授权
chmod 777 /tmp/data/elasticsearch/node-4/{config,plugins,data,log}
```

# 添加IK分词器

只要将其他节点的IK分词器复制过来就可

```
cp -R ik/ /tmp/data/elasticsearch/node-4/plugins/ 
```

编写配置文件

我们边界node4节点的配置文件

```
vi /tmp/data/elasticsearch/node-4/config/elasticsearch.yml 
```

\#集群名称

```
cluster.name: elastic 
```

\#当前该节点的名称

```
node.name: node-4 
```

\#是不是有资格竞选主节点

```
node.master: true 
```

\#是否存储数据

```
node.data: true 
```

\#最大集群节点数

```
node.max_local_storage_nodes: 3 
```

\#给当前节点自定义属性（可以省略）

```
#node.attr.rack: r1 
```

\#数据存档位置

```
path.data: /usr/share/elasticsearch/data 
```

\#日志存放位置

```
path.logs: /usr/share/elasticsearch/log 
```

\#是否开启时锁定内存（默认为是）

```
#bootstrap.memory_lock: true 
```

\#设置网关地址，我是被这个坑死了，这个地址我原先填写了自己的实际物理IP地址，

\#然后启动一直报无效的IP地址，无法注入9300端口，这里只需要填写0.0.0.0

```
network.host: 0.0.0.0 
```

\#设置映射端口

```
http.port: 9200 
```

\#内部节点之间沟通端口

```
transport.tcp.port: 9300 
```

\#集群发现默认值为127.0.0.1:9300,如果要在其他主机上形成包含节点的群集,如果搭建集群则需要填写

\#es7.x 之后新增的配置，写入候选主节点的设备地址，在开启服务后可以被选为主节点，也就是说把所有的节点都写上

```
discovery.seed_hosts: ["node-1", "node-2", "node-3"] 
```

\#当你在搭建集群的时候，选出合格的节点集群，有些人说的太官方了，

\#其实就是，让你选择比较好的几个节点，在你节点启动时，在这些节点中选一个做领导者，

\#如果你不设置呢，elasticsearch就会自己选举，这里我们把三个节点都写上

```
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"] 
```

\#在群集完全重新启动后阻止初始恢复，直到启动N个节点

\#简单点说在集群启动后，至少复活多少个节点以上，那么这个服务才可以被使用，否则不可以被使用，

```
gateway.recover_after_nodes: 2 
```

\#删除索引是是否需要显示其名称，默认为显示

```
#action.destructive_requires_name: true 
```

# 禁用安全配置，否则查询的时候会提示警告

```
xpack.security.enabled: false 
```

# 编写部署文档

我们在部署脚本增加node-4节点

```
vi docker-compose.yml 
version: "3" 
services:
    node-1:
    image: elasticsearch:7.17.5
    container_name: node-1
    environment:
    - "ES_JAVA_OPTS=-Xms1024m -Xmx1024m"
    - "TZ=Asia/Shanghai"
    ulimits:
    memlock:
    soft: -1
    hard: -1
    nofile:
    soft: 65536
    hard: 65536
    ports:
    - "9200:9200"
    logging:
    driver: "json-file"
    options:
    max-size: "50m"
    volumes:
    - /tmp/data/elasticsearch/node-1/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    - /tmp/data/elasticsearch/node-1/plugins:/usr/share/elasticsearch/plugins
    - /tmp/data/elasticsearch/node-1/data:/usr/share/elasticsearch/data
    - /tmp/data/elasticsearch/node-1/log:/usr/share/elasticsearch/log networks:
    - elastic
    node-2:
    image: elasticsearch:7.17.5
    container_name: node-2
    environment:
    - "ES_JAVA_OPTS=-Xms1024m -Xmx1024m"
    - "TZ=Asia/Shanghai"
    ulimits:
    memlock:
    soft: -1
    hard: -1
    nofile:
    soft: 65536
    hard: 65536
    ports:
    - "9201:9200"
    logging:
    driver: "json-file"
    options:
    max-size: "50m"
    volumes:
    - /tmp/data/elasticsearch/node-2/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    - /tmp/data/elasticsearch/node-2/plugins:/usr/share/elasticsearch/plugins
    - /tmp/data/elasticsearch/node-2/data:/usr/share/elasticsearch/data
    - /tmp/data/elasticsearch/node-2/log:/usr/share/elasticsearch/log networks:
    - elastic 
node-3:
    image: elasticsearch:7.17.5
    container_name: node-3
    environment:
    - "ES_JAVA_OPTS=-Xms1024m -Xmx1024m"
    - "TZ=Asia/Shanghai"
    ulimits:
    memlock:
    soft: -1
    hard: -1
    nofile:
    soft: 65536
    hard: 65536
    ports:
    - "9202:9200"
    logging:
    driver: "json-file"
    options:
    max-size: "50m"
    volumes:
    - /tmp/data/elasticsearch/node-/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    - /tmp/data/elasticsearch/node-3/plugins:/usr/share/elasticsearch/plugins
    - /tmp/data/elasticsearch/node-3/data:/usr/share/elasticsearch/data
    - /tmp/data/elasticsearch/node-3/log:/usr/share/elasticsearch/log networks:
    - elastic
    node-4:
    image: elasticsearch:7.17.5
    container_name: node-4
    environment:
    - "ES_JAVA_OPTS=-Xms1024m -Xmx1024m"
    - "TZ=Asia/Shanghai"
    ulimits:
    memlock:
    soft: -1
    hard: -1
    nofile:
    soft: 65536
    hard: 65536
    ports:
    - "9203:9200"
    logging:
    driver: "json-file"
    options:
    max-size: "50m"
    volumes:
    - /tmp/data/elasticsearch/node-/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    - /tmp/data/elasticsearch/node-4/plugins:/usr/share/elasticsearch/plugins
    - /tmp/data/elasticsearch/node-4/data:/usr/share/elasticsearch/data
    - /tmp/data/elasticsearch/node-4/log:/usr/share/elasticsearch/log networks:
    - elastic
kibana: 
container_name: kibana
image: kibana:7.17.5
volumes:
- /tmp/data/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml
ports:
- 5601:5601
networks:
- elastic
cerebro:
image: lmenezes/cerebro:0.9.4
container_name: cerebro
environment:
TZ: 'Asia/Shanghai'
ports:
- '9000:9000'
networks:
- elastic
networks:
elastic:
driver: bridge 
```

# 启动服务

我们完成配置后就可以启动服务了

```
docker-compose upd -d 
```

# 查看节点信息

我们可以在监控界面看到节点的信息，我们发现节点4已经加入进来了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/1b69a795c86fc9fdf5421de14922eecff53c212fc386fd20e2e2369c943e7483.jpg)

# 节点缩容

接下来我们要将node4节点去掉完成缩容操作

# 禁止数据分配

我们先排除node-4节点，禁止数据在该节点分配数据，然后才能停止节点，如果想正常缩容，这里填上所有要缩容的节点名称就可以了

```
PUT/_cluster/settings
{
    "persistent": {
    "cluster.routing.allocation.exclude._name": "node-4"
    }
} 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/9cbc48d2ce6aa6fd91a43f94dfb620b6f46a6431bd6e38d82928649c68c96787.jpg)

# 检查数据分配

接下来检查一下缩容节点的数据迁移情况，我们发现数据已经全部迁移完成了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/95719d9ed604568009ba51caf27aae92470bcda04486f7d8a801bdaf27e83076.jpg)

# 关闭节点

等到数据全部迁移完成后就可以进行缩容节点了

```
docker-compose stop node-4 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/a976c62e922932eef68cf94be785131df8db56fa4187b96264446c10785e6c0b.jpg)

查看集群情况

接下来看一下集群的情况，我们发现已经缩容成功了，并且没有出现主分片丢失的情况

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/52bb7b263903cfb35234aaaf4c12d2456873a282296aea9fc63733954876eb31.jpg)

# 分布式文档原理

下面我们讲解下文档搜索的原理

# 索引的路由计算

当索引一个文档的时候，文档会被存储到一个主分片中，Elasticsearch如何知道一个文档应该存放到哪个分片中呢？

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/7f7dcbcfbee722d6ab418e8b092d9458ff658feae42798ae33660c767cae654c.jpg)

首先这肯定不会是随机的，否则将来要获取文档的时候我们就不知道从何处寻找了，实际上，这个过程是根据下面这个算法决定的：

```
shard = hash(routing) % number_of_primary_shards 
```

- routing值是一个任意字符串，它默认是_id，但也可以自定义。

- 这个routing字符串通过哈希函数生成一个数字，然后除以主切片的数量得到一个余数(remainder)，余数的范围永远是0到number_of_primary_shards - 1，这个数字就是特定文档所在的分片。

# 注意事项

通过上面的公式，我们理解并且也需要记住一个重要的规律

创建索引的时候就确定好主分片的数量，并且永远不会改变这个数量，数量的改变将导致上述公式的结果变化，最终会导致我们的数据无法被找到。

# 文档的写操作

新建、索引和删除请求都是写(write)操作，它们必须在主分片上成功完成才能复制到相关的复制分片上

下图是数据写入 P0 主分片的过程，master 在这里起到一个协调节点的作用

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/35f82908c14a5993a987429c69140572dd9c898835d777496e69c291d342f4c3.jpg)

# 详细步骤

下面我们罗列在主分片和复制分片上成功新建、索引或删除一个文档必要的顺序步骤：

新增文档流程：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/89ee674c6f9e5b56cfece51a0729828f1331a6129b4e7aa54ecd6d291c35d3ea.jpg)

1. 客户端给 Node 1 发送新建、索引或删除请求。

1. 节点使用文档的 _id 确定文档属于分片0，它转发请求到 Node 3，分片0位于这个节点上。

1. Node 3 在主分片上执行请求

1. Node 3保存文档，将数据保存到主分片

1. 保存成功后，它转发请求到相应的位于 Node 1 和 Node 2 的复制节点上

1. 当所有的复制节点报告成功，Node 3 报告成功到请求的节点

1. 请求的节点再报告给客户端，客户端接收到成功响应的时候，文档的修改已经被应用于主分片和所有的复制分片

# 注意事项

把文档存储写入到 primary shard，如果设置了 index.write.wait_for_active_shards=1，那么写完主节点，直接返回客户端，如果 index.write.wait_for_active_shards=all，那么必须要把所有的副本写入完成才返回客户端

# 搜索文档(单个)

我们根据文档ID查询的时候ES是如何搜索到我们的文档的呢？

CLUSTER

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/ea506d3e333bf32f08a49da047b4cf0332fc72092e6188f5ed376475f0072a03.jpg)

# 详细步骤

下面我们罗列在主分片或复制分片上检索一个文档必要的顺序步骤：

1. 客户端给 Node 1 发送 get 请求。

2.节点使用文档的_id确定文档属于分片0，对应的复制分片在三个节点上都有，此时它转发请求到Node2

1. Node 2 返回文档(document)给 Node 1 然后返回给客户端

# 注意事项

对于读请求，为了平衡负载，请求节点会为每个请求选择不同的分片——它会循环所有分片副本

一个被索引的文档已经存在于主分片上却还没来得及同步到副本分片上，这时副本分片会报告文档未找到，如果查询主分片则会成功返回文档，这种情况下会产生读写不一致的情况

由于可能存在primary shard的数据还没同步到 replica shard上的情况，所以客户端可能查询到旧的数据，我们可以做相应的调整，保证读取到最新的数据。

# 更新文档(单个)

更新文档，必须先定位到主分片，修改文档后，再次同步到其他副本中才算完成

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/d7138bd52130887c39acbe68d8d7e12033e6041c7a4e55a03cab9ef9f9e6ae9e.jpg)

# 详细步骤

以下是部分更新一个文档的步骤:

1. 客户端向 Node 1 发送更新请求，发现主分片在 Node 3

1. 它将请求转发到主分片所在的 Node 3

1. Node 3 从主分片检索文档，修改 _source 字段中的 JSON，并且尝试重新索引主分片的文档，如果文档已经被另一个进程修改，它会重试步骤 3，超过 retry_on_conflict 次后放弃。

1. 如果 Node 3 成功地更新文档，它将新版本的文档并行转发到 Node 1 和 Node 2 上的副本分片，重新建立索引，一旦所有副本分片都返回成功，Node 3 向协调节点也返回成功，协调节点向客户端返回成功。

# 文档复制

当主分片把更改转发到副本分片时，它不会转发更新请求，相反，它转发完整文档的新版本

注意，这些更改将会异步转发到副本分片，并且不能保证它们以发送它们相同的顺序到达，如果Elasticsearch仅转发更改请求，则可能导致更新顺序错误，导致文档更新结果错误。

# 全文搜索

对于全文搜索而言，文档可能分散在各个节点上，那么在分布式的情况下，如何搜索文档呢？

搜索，分为2个阶段，搜索（query）+取回（fetch）

# 搜索 (query)

在初始查询阶段时，查询会广播到索引中每一个分片拷贝（主分片或者副本分片），每个分片在本地执行搜索并构建一个匹配文档的优先队列。

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/7229238a37ca7b7b8934f2ee2f06bd8d474ed11fd7581245fe4ab142a5da4938.jpg)

# 详细步骤

查询阶段包含以下三步：

- 客户端发送一个 search（搜索）请求发送给 Node 3，他会创建了一个长度为 from+size 的空优先级队

- Node 3 转发这个搜索请求到索引中每个分片的主分片或副本分片，每个分片在本地执行这个该查询并且结果将结果存储到一个大小为 from+size 的本地有序优先队列里去。

- 每个分片返回 document 的 ID 和该节点优先队列里的所有 document 的排序值给协调节点 Node 3，而 Node 3 会把这些值合并到自己的优先队列里产生全局排序结果。

# 什么是优先级队列

一个 优先队列 仅仅是一个存有 top-n 匹配文档的有序列表，优先队列的大小取决于分页参数 from 和 size，如下搜索请求将需要足够大的优先队列来放入100条文档

```
GET/_search
{
    "from": 90,
    "size": 10
} 
```

# 注意事项

当一个搜索请求被发送到某个节点时，这个节点就变成了协调节点

这个节点的任务是广播查询请求到所有相关分片并将它们的响应整合成全局排序后的结果集合，这个结果集合会返回给客户端。

第一步是广播请求到索引中每一个节点的分片拷贝，查询请求可以被某个主分片或某个副本分片处理，这就是为什么更多的副本（当结合更多的硬件）能够增加搜索吞吐率，协调节点将在之后的请求中轮询所有的分片来分摊负载。

每个分片在本地执行查询请求并且创建一个长度为 from + size 的本地优先队列，也就是说，每个分片创建的结果集足够大，均可以满足全局的搜索请求，分片返回一个轻量级的结果列表到协调节点，它仅包含文档 ID 集合以及任何排序需要用到的值，例如 _score

协调节点将这些分片级的结果合并到自己的有序优先队列里，它代表了全局排序结果集合，至此查询过程结束。

# 取回 (fetch)

查询阶段标识哪些文档满足搜索请求，但是我们仍然需要取回这些文档

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/40f76c22b4a295a65ae8dee78e1182d894176251e740e5f3b88981a630e236e7.jpg)

# 详细步骤

分发阶段由以下步骤构成：

1. 协调节点辨别出哪个 document 需要取回，并且向相关分片发出 GET 请求。

1. 每个分片加载 document 并且根据需要丰富它们，然后再将 document 返回协调节点。

1. 一旦所有的 document 都被取回，协调节点会将结果返回给客户端。

# 注意事项

协调节点首先决定哪些文档确实需要被取回。

例如，如果我们的查询指定了 { "from": 90, "size": 10 }，最初的90个结果会被丢弃，只有从第91个开始的10个结果需要被取回，这些文档可能来自和最初搜索请求有关的一个或者多个甚至全部分片。

协调节点给持有相关文档的每个分片创建一个 multi-get request，并发送请求给同样处理查询阶段的分片副本

# 路由机制

假设你有一个100个分片的索引，当一个请求在集群上执行时会发生什么呢？

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/995ab589208a436b0bda087da783f08143920388a608d731d31a7f743903758f.jpg)

1. 这个搜索的请求会被发送到一个节点

1. 接收到这个请求的节点，将这个查询广播到这个索引的每个分片上（可能是主分片，也可能是复本分片）

1. 每个分片执行这个搜索查询并返回结果

1. 结果在通道节点上合并、排序并返回给用户

# 为什么使用路由

因为默认情况下，Elasticsearch使用文档的ID（类似于关系数据库中的自增ID），如果插入数据量比较大，文档会平均的分布于所有的分片上，如果不按照分片键进行搜索会导致了Elasticsearch不能确定文档的位置，所以它必须将这个请求广播到所有的N个分片上去执行这种操作会给集群带来负担，增大了网络的开销。

如果你根本就不使用路由，Elasticsearch将确保你的文档以均衡的方式分布在所有不同的分片中，那么为什么还需要使用路由？定制路由允许你将同一个路由值得多篇文档归集到某一个分片中，而一旦这些文档放入到同一索引中，就可以路由某些查询，让它们可以在索引分片得子集中执行（简而言之：根据指定的散列值决定相关文档放在哪些分片上），类似于分库分表的路由键的概念。

# 路由查询

下面我们演示以下路由的使用

# 普通查询

下面我们介绍下不加路由的查询方式

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "match": {
    "name": "龙苑居住区"
    }
    }
}
```

# 我们发现查询的时候扫描了三个分片

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/968799cc17923539e750885428c548a2b8d80e8f81493e70ede186db7cb35982.jpg)

# 路由查询

下面我们通过路由的方式进行查询试试，路由查询只需要在请求后面加上路由key即可

```
GET logstash-village-2022.08.22/_search?routing=routingKey
{
    "query": {
    "match": {
    "name": "龙苑居住区"
    }
    }
}
```

这个路由key可以随意写，默认查询的路由key是_id，现在我们就换成了routingKey这样我们发现，查询只查询了一个分片，这样查询效率会更高，但是我们写入的时候是通过_id 写入的，查询的时候通过指定路由键，有些数据会查询不出来的，比如

← → ⬆ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新 :

← → ⚫ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新 :

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/88e3aac87c7183f77742f7287c3463c46baae49373fd8670ff30ce50dcbd2663.jpg)

```
GET logstash-village-2022.08.22/_search?routing=key
{
    "query": {
    "match": {
    "name": "龙苑居住区"
    }
    }
}
```

这样直接搜索是查不到数据的，根据 key 路由键定位的分片是没有数据的，如何解决呢，就需要写和读都是用相同的路由键，再写入的时候也指定路由键即可

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/eef94f2819ff70b2d2ece48395f04dacbdc72be1ccbbb8f98d6f3c1ae093ef92.jpg)

# 自定义路由(拓展)

自定义路由的方式非常简单，只需要在插入数据的时候指定路由的key即可，虽然使用简单，但有许多的细节需要注意

# 创建索引

先创建一个名为route_test的索引，该索引有3个shard，0个副本

```
PUT route_test/
{
    "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 0
    }
} 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/b75045a119ef4bd4cdf623cad708e83385bacc0b51c17c7a7b88fb6e98140547.jpg)

# 查看分片

我们接下来查看以下分片信息

```
GET _cat/shards/route_test?v 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/f1dd2d6cd731ddeb2407af40caa42df73e27bd235ebcbb2df480047e5c508385.jpg)

# 插入数据

接下来我们就需要插入数据

# 插入第一条数据

```
PUT route_test/_doc/a?refresh
{
    "data": "A"
} 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/d639b9d579d59318826a683b045e4f436ba9406625de66cf0fd6ac66cdd9b621.jpg)

# 查看分片

我们插入数据后再次来查看分片信息

```
GET _cat/shards/route_test?v 
```

我们发现我们插入的数据加入了0分片

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/d0bddf59e6891903c3f939cd5ba042c3ce19c91137b27db2415986c7095a95d6.jpg)

# 插入第二条数据

接下来我们插入第二条数据

```
PUT route_test/_doc/b?refresh
{
    "data": "B"
} 
← → C 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/a10fb988645a5b5fff87e20cfbb17f4c2f2a163b52fff353a314c172e9dbfece.jpg)

# 查看分片

我们插入数据后再次来查看分片信息

```
GET _cat/shards/route_test?v 
```

我们发现我们插入的数据加入了2分片

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/e8fa6299ec4a83b0824c06cfc346fd6bcefe135cafd489488f7bbe1b6f7c0014.jpg)

查询数据

接下来我们查询数据

```
GET route_test/_search 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/8c69f09626aff1fd910db2113c9c4ea15d260cd9096f530f73b2a707edd7e896.jpg)

上面这个例子比较简单，先创建了一个拥有3个shard，0个副本（为了方便观察）的索引route_test，创建完之后查看两个shard的信息，此时shard为空，里面没有任何文档（docs列为0）。

接着我们插入了两条数据，每次插完之后，都检查shard的变化，通过对比可以发现docid=a的第一条数据写入了0号shard，docid=b的第二条数据写入了2号shard。

需要注意的是这里的doc_id我选用的是字母"a"和"b"，而非数字，原因是连续的数字很容易路由到一个shard中去，以上的过程就是不指定routing时候的默认行为。

# 指定路由

接着，我们指定routing，看看会发生什么

# 插入第三条数据

接下来我们插入第三条数据，但是这条数据我们加上一个路由键

```
PUT route_test/_doc/c?routing=key1&refresh
{
    "data": "C"
} 
```

← → ⚫ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/48f78b58f59de095a1733bd68bbcf53d01e9e8d0054cedfb059a89f156200508.jpg)

# 查看分片

我们插入数据后再次来查看分片信息

```
GET _cat/shards/route_test?v 
```

我们发现我们插入的数据加入了0分片

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/f97fc967f0325d5cb3a2024284a09178c752efabaa36a88a2ae2a1f7f1ffc38c.jpg)

# 查询索引数据

```
GET route_test/_search 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/7a088f1f3c8a1e486f1a1712cdcf86aa51a1d2ce3dea8e37834d4b53a2b06a38.jpg)

我们又插入了1条 docid=c 的新数据，但这次我们指定了路由，路由的值是一个字符串"key1"，通过查看shard信息，能看出这条数据路由到了0号shard，也就是说用"key1"做路由时，文档会写入到0号shard。

# 指定路由插入

接着我们使用该路由再插入两条数据，但这两条数据的 docid 分别为之前使用过的 "a" 和 "b"

# 再次插入数据

插入 docid=a 的数据，并指定 routing=key1

```
PUT route_test/_doc/a?routing=key1&refresh
{
    "data": "A with routing key1"
} 
```

注意返回的状态为updated，之前的三次插入返回都为created

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/f638d036fa6b11c5053763aca61ccdeb3ec10839f8f06636158316502142342d.jpg)

# 查看分片

我们插入数据后再次来查看分片信息

```
GET _cat/shards/route_test?v 
```

我们发现分片的数据没有变化

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/bcb903e50f6d1a6524127b4bd12f2dd7f615976a5deafbf78c3f5a4266baf6ff.jpg)

# 查询数据

```
GET route_test/_search 
```

之前 docid=a 的数据就在0号shard中，这次依旧写入到0号shard中了，因为docid重复，所以文档被更新了

← → ⚫ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/d22a31563055eb2fab997fb7c35f96081ab366d789386c7019c95982b0b20cca.jpg)

# 再次插入数据

这次插入 docid=b 的数据，使用 key1 作为路由字段的值

```
PUT route_test/_doc/b?routing=key1&refresh
{
    "data": "B with routing key1"
} 
```

我们发现这次编程创建了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/a35ebd45fde8fb57e99a7cfc4f4fdcb74e9dbaaf6db6c832d866ab12b1f8306e.jpg)

# 查看分片信息

我们再次查看分片信息

```
GET _cat/shards/route_test?v 
```

我们发现数据存储到了0分片中

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/38c7c77621a6fd0b3af828357a3e7cd2f7280df3cee1a1697d4439c023de8f22.jpg)

查询数据

我们再次来查询数据

```
GET route_test/_search 
```

和上面插入docid=a的那条数据相比，这次这个有些不同，我们来分析一下

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/63da0237a0d83ce7ddcd744be1e22b623c273873c4358f8771da91c05282f675.jpg)

# 路由带来的问题

这个就是我们自定义routing后会导致的一个问题：docid不再全局唯一

ES shard的实质是Lucene的索引，所以其实每个shard都是一个功能完善的倒排索引，ES能保证docid全局唯一是采用docid作为了路由，所以同样的docid肯定会路由到同一个shard上面，如果出现docid重复，就会update或者抛异常，从而保证了集群内docid唯一标识一个doc。

但如果我们换用其它值做routing，那这个就保证不了了，如果用户还需要docid的全局唯一性，那只能自己保证了，因为docid不再全局唯一，所以doc的增删改查API就可能产生问题

# 索引别名

别名，有点类似数据库的视图，别名一般都会和一些过滤条件相结合，可以做到即使是同一个索引上，让不同人看到不同的数据

# 别名的作用

在开发中，一般随着业务需求的迭代，较老的业务逻辑就要面临更新甚至是重构，对于es来说为了适应新的业务逻辑，就要对原有的索引做一些修改，比如对某些字段做调整

而做这些操作的时候，可能会对业务造成影响，甚至是停机调整等问题，因为es提供了索引的别名来解决这个问题，索引的别名就像一个快捷方式或者是软连接，可以指向一个或者多个索引，也可以给任意一个需要索引名的API来使用

# 别名操作

下面我们看下别名的基本操作

# 查询别名

直接调用 _alias API的GET方法可以看到索引的别名

```
GET logstash-village-2022.08.22/_alias 
```

我们看到现在可以看到当前的索引有一个别名 logstash-village

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/49cba2311a6add597004e5420c2218316c23cf9321fba00e149a3eba119b3e5b.jpg)

# 别名查询

我们查询的时候可以指定别名进行查询

```
GET logstash-village/_search
{
    "query": {
    "match": {
    "name": "龙苑居住区"
    }
    }
}
```

这样我们可以通过别名查询出来数据的

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/cce4071984903e1e11db4993647146604cf0f146fe221e6af5a795e180d19735.jpg)

# 创建别名

我们还可以在建立一个别名，别名和索引的关系是多对多的关系，一个索引可以有多个别名，同样一个别名也可以有多个索引

```
POST / _aliases
{
    "actions": [
    {
    "add": {
    "index": "logstash-village-2022.08.22",
    "alias": "logstash-village-1.0"
    }
    } ]
} 
```

这样我们就创建了一个别名 logstash-village-1.0

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/9e1ea6689e534b0c576b2f048d5dba6ddec01d06c9d16dfb01818a16eb86b9d1.jpg)

接下来我们直接进行别名查询就好

```
GET logstash-village-1.0/_search
{
    "query": {
    "match": {
    "name": "龙苑居住区"
    }
    }
}
```

这样就检索出来数据了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/4f812c1d399497527898c169521f887a1116d283d28d5138dc52b653c770fbd5.jpg)

# 别名修改

有时候还需要修改别名，特别是涉及到索引迁移的时候，修改操作我们可以实现运行中的es集群无缝切换索引，我们可以将索引指向一个新准备的别名中，也可以为别名关联新的索引

```
POST/_aliases
{
    "actions": [
    {
    "remove": {
    "index": "logstash-village-2022.08.22",
    "alias": "logstash-village-1.0"
    }
    },
    {
    "add": {
    "index": "logstash-village-2022.08.22",
    "alias": "logstash-village-2.0"
    }
    }
    ]
} 
```

这样我们就可以做到无缝的索引别名修改了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/46f23d1e9b4025fb24bbb5b81a6feaec9867ba0775172777b80f99fd76a8235c.jpg)

我们再来查询试试

```
GET logstash-village-2.0/_search
{
    "query": {
    "match": {
    "name": "龙苑居住区"
    }
    }
}
```

# 过滤器别名

我们可以创建一个带过滤器的别名，这样别人通过这个别名查询的时候，数据都是筛选过后的数据，起到一个数据权限的作用

# 创建别名

下面我们创建一个只能查询河南省房产信息的别名

```
POST / _aliases
{
    "actions": [
    {
    "add": {
    "index": "logstash-village-2022.08.22",
    "alias": "logstash-village-hn",
    "filter": {
    "term": {
    "province": "河南省"
    }
    }
    }
    }
    ]
}
```

这样我们就创建了一个只能查询到河南省房产信息的别名 logstash-village-hn

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/1e5c54654a944faf08102705d9046201d2eb14648422f1a175c1d1a4125b151a.jpg)

# 数据查询

下面我们通过这个别名查询北京沁春家园的小区信息

```
GET logstash-village-hn/_search
{
    "query": {
    "match": {
    "name": "沁春家园"
    }
    }
}
```

我们发现根本就查询不出来

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/7ea1a6d606a5125e9d37710e7bf9e8be065d8848647b0d543c80b06e5e3faef6.jpg)

但是我们查询 龙苑居住区 却可以查询出来

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/8dfdfc3c4aeeafb6380a2b246045b74317502d06b1c14ad61e05b0298b1dc6ce.jpg)

# 路由别名

我们上面介绍了路由的使用，但是有一个问题，我们查询的时候都需要携带路由参数，很麻烦，我们可以将我们的路由参数写进别名中，这样查询起来会更加方便

# 创建别名

下面我们就创建一个以 key 为路由键

```
POST / _aliases
{
    "actions": [
    {
    "add": {
    "index": "logstash-village-2022.08.22",
    "alias": "logstash-village-route_key",
    "routing": "key"
    }
    }
    ]
} 
```

这样我们就以 key 为路由键创建了一个索引

← → ⚠️ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

← → ⬆ 不安全 | 192.168.245.151:5601/app/dev_tools#/console

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/fac6bf30ac378b37bd0e33dac6bfaad8224e8f701a53c3c4b4781b3bc7997df4.jpg)

# 数据查询

下面我们就对索引进行一些查询

```
GET logstash-village-route_key/_search
{
    "query": {
    "match": {
    "name": "沁春家园"
    }
    }
}
```

我们看到查询沁春家园是可以查询出来数据的，但是查询龙苑居住区是查询不出来数据的

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/d68d26ed0a5e69f5e0e4206e7ef8115c6731dabcf1303590a02f12044e954d6f.jpg)

# 删除别名

创建了很多的别名，有时候别名不用了，需要定期删除以下

# 查看所有别名

现在我们查询以下当前索引下的别名有哪些

```
GET logstash-village-2022.08.22/_alias 
```

当前有这么多的别名，我们准备删除一些

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/0fc6584327e0f49eaa8df2d8b0d2cd29690b26212d12e480f968639cf841c4b9.jpg)

# 删除别名

删除的时候直接指定别名就可以的

```
DELETE logstash-village-2022.08.22/_alias/logstash-village-route_key 
```

这样我们就把当前这个别名删除了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/2ad00b43c2fb0c750f0b441e76e4ea5cf948576b019755891d55136c8e1f0030.jpg)

# 重建索引

Elasticsearch使用时间长了后，到了后期可能有各种原因重建索引

ES是不支持索引字段类型变更的，不可变的原因是一个字段的类型进行修改之后，ES会重新建立对这个字段的索引信息，影响到ES对该字段分词方式，相关度，TF/IDF倒排索引创建等。

# 索引重建的步骤

1. 创建旧索引

1. 给索引创建别名

1. 向oldindex中插入数据

1. 创建新的索引newindex

1. 重建索引

1. 实现不重启服务索引的切换

# 创建旧索引

```
PUT oldindex
{
    "mappings": {
    "properties": {
    "name": {
    "type": "text"
    },
    "price": {
    "type": "double"
    }
    }
} 
← → C 不安全 | 192.168.245.151:5601/app/dev_tools#/console
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/8d4c34e75dc7bdc132480d6d9eefa3f45c3ab7b1e1cb84cc3ffa3823237ced36.jpg)

# 添加数据

```
POST oldindex/_doc/_bulk
{"create":{"_id":1}}
{"name":"name 01","price":1}
{"create":{"_id":2}}
{"name":"name 02","price":2}
{"create":{"_id":3}}
{"name":"name 03","price":3}
{"create":{"_id":4}}
{"name":"name 04","price":4}
{"create":{"_id":5}}
{"name":"name 05","price":5}
{"create":{"_id":6}}
{"name":"name 06","price":6}
{"create":{"_id":7}}
{"name":"name 07","price":7}
{"create":{"_id":8}}
{"name":"name 08","price":8}
{"create":{"_id":9}}
{"name":"name 09","price":9} 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/a4b4b1c899337f4bd63e78c24178f0a1ddd35d25de7c2aea6401f778548a9825.jpg)

# 查询数据

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/a70427e002e1fc9b2a05d099043a742bcad5648821c6e46afbd5785ccd05f8f2.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/bbe5d6fae8d8a8c7b29bf5c3429f18ce95c1a852d503ee565275c37bf06dcbf0.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/a04f0d69b7a9a14aeb723a3c8bf9ded5c89f4a5b9d389fb7fcf7b11a0b5df721.jpg)

# 创建别名

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/7fa4f6328baa8660de57c72420a765272cd747c63d02d34dd2ce1767feb5d72e.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/d149244657de5984993031b9517d6396c115918789904364bad64b74fd37ad01.jpg)

# 查询数据

我们使用别名查询数据

GET search_index/_search

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/4588fabb2c2ae02b70c1590ad9f1f07be90bb68a1175eed6929f54be8305c7e3.jpg)

# 创建新索引

根据需求我们创建一个新的索引，价格字段改为integer类型

```
PUT newindex
{
    "mappings": {
    "properties": {
    "name": {
    "type": "text"
    },
    "price": {
    "type": "integer"
    }
    }
} 
```

← → C ▲ 不安全 | 192.168.245.151:5601/app/dev_tools#/console

elastic 搜索 Elastic

- 2017: 10.45

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/ed575a8122160d257612d14c80e754eeef1122c99c36d5dc1538244ab686d569.jpg)

# 重建索引

数据量大的话可以异步执行，如果 reindex 时间过长，建议加上 wait_for_completion=false 的参数条件，这样 reindex 将直接返回 taskId

```
POST _reindex?wait_for_completion=false
{
    "conflicts": "proceed", // 如果新的索引中数据冲突，程序继续往下执行，删除则会导致程序会终止
    "source": {
    "index": "oldindex" // 表示从oldindex中同步数据
    },
    "dest": {
    "index": "newindex", // 表示数据插入新索引newindex中
    "op_type": "create" // 数据插入的类型为创建，如果存在就会版本冲突
    }
}
```

# 更多参数

更高级的用法可以参考下下面的例子

```
POST _reindex?wait_for_completion=false
{
    "size": 5, // 表示只获取5条数据插入到新的索引中
    "conflicts": "proceed", // 如果新的索引中数据冲突，程序继续往下执行，删除则会导致程序会终止
    "source": {
    "size": 2, // 默认情况下，_reindex使用1000进行批量操作，调整批量插入2条
    "index": "oldindex", // 表示从oldindex,类型product中查询出price字段的值
    "_source": [
    "price" //只需要同步price字段
    ],
    "query": {
    "range": {
    "price": {
    "gte": 2,
    "lte": 8
    }
    }
    }
    },
    "dest": {
    "index": "newindex", // 表示数据插入新索引newindex中
    "op_type": "create" // 数据插入的类型为创建，如果存在就会版本冲突
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/52242481313636219787910f46acf6d0231b62cbdabd69b1bd4916b1298a90a8.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/c999e4732bc42ae855a876894aa999519b9adfcad4e31ec9f076b8b3fbd02d55.jpg)

# 查看任务

GET _tasks/0f73ybYqQTOc960mN_PSEw:72228

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/2017d4d857ef85d371d9bc56d29446debb871370569c4392257c2a9f31ec3871.jpg)

# 取消任务

如果任务还没有完成，需要取消任务可以使用如下的命令

```
POST _tasks/0f73ybYqQTOc960mN_PSEw:72228/_cancel 
```

# 别名切换

我们需要将别名切换到另刚刚重建的索引上，切换索引可以实现不重启服务索引的切换

```
POST _aliases
{
    "actions": [
    {
    "remove": {
    "index": "oldindex",
    "alias": "search_index"
    }
    ]
} 
},
{
    "add": {
    "index": "newindex",
    "alias": "search_index"
    }
}
] 
```

# 这样就实现了快速索引切换

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/ec60272add6ba418a89492319e34e7dcb0c000bb48e59e78f0ba4146eff05049.jpg)

# 删除旧的索引

DELETE oldindex

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/e4802e4bc60f491d440f19caaef20375b37f59d8d17e0cb710d8e64401d139ab.jpg)

# 查询数据

```
GET search_index/_search
{} 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/72aee51ddf9ad1dd61f41ffa89c9322df2fad3dbb177d2c762a248da8ebfceba.jpg)

# 实战演练

ES的主要功能就是用来查询的，我们就用我们的30万的房产数据进行数据的查询，下面我们先看下基本查询

# 查看所有索引

我们要进行查询首先要确定索引是什么，我们先来查看下当前的索引

```
GET _cat/indices 
```

这个索引就是我们logstash自动创建的索引

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/d547036db28ad31f517b138e19b0e4bb32b48bb4f0284abd5f117e5dbaf0828a.jpg)

search查询也是比较简单的一种查询方式，下面看看常见的用法

# 查询所有文档

确定了索引我们需要查询以下所有的数据，我们看下如何进行查询

```
get logstash-village-2022.08.22/_search
{
    "query": {
    "match_all": {}
    }
}
# 可以简写为以下形式
get logstash-village-2022.08.22/_search
```

这样我们就查询出来数据了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/543b5490d742e2859d797755bd1782c126a8901f85878955672119c835f84170.jpg)

返回结果说明

```
{
    "took": 6,    #该命令请求花费了多长时间，单位：毫秒
    "timed_out": false,    #本次搜索是否超时
    "_shards": {    #本次搜索的分片信息
    "total": 3,    #本次搜索的分片个数
    "successful": 3,    #搜索成功的分片数量
    "skipped": 0,    #没有搜索的分片，跳过的分片
    "failed": 0    #搜索失败的分片数量
},
"hits": {    #搜索结果集，项目中，我们需要的一切数据都是从hits中获取
    "total": {
    "value": 10000,    #总共返回多少条数据
    "relation": "gte"
    },
    "max_score": 1.0,    #返回结果中，最大的匹配度分值
    "hits": [    #默认查询前十条数据，根据分值降序排序
    {
    "_index": "logstash-village-2022.08.22",    #索引库名称
    "_type": "_doc",    #索引类型名称
    "_id": "Pi6Qw4IB8w-RNlYJ6ATP",    #文档ID
    "_score": 1.0,    #文档评分，关键字与该条数据的匹配度分值
    "_source": {    #索引库中类型，返回结果字段，不指
```

定的话，默认全部显示出来

```
"name"："蓝色海岸",
"built_year"："2001",
"area"："中山",
"location" {
    "lon"："121.66547055689303",
    "lat"："38.92055689059745"
},
"floorage"："50500.0",
"houses"："247",
"volume"："1.35",
"property_type"："公寓",
"producer"："大连东亚房地产开发有限公司",
"@timestamp"："2022-08-22T11:20:28.606z",
"property_company"："联华物业管理公司",
"province"："辽宁省",
"property_cost"："0.0",
"parkings"："0",
"@version"："1",
"city"："大连市",
"info" """园林景观:小区景观优美、环境清静、建筑错落有致小区内部配套：小区内部配套设施完善"""
"greening"："40.0",
"addr"："中山-寺儿沟-勤俭街110号"
}
}
```

# 统计小区数量

下面我们结合一个统计小区数量来讲解下ES的基本查询

# 查询小区数量

下面我们查询河南省有的小区的数量

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "match": {
    "province": "河南省"
    }
    }
}
```

我们这样就将河南省的小区数量查询出来了，我们发现查询的结果是大于一万条，在ES中为了性能问题，大于一万的数据就不直接显示，直接提示大于一万条，我们可以加入其他条件来精确查找

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/88706c152b408cd2205abe2d7b5cbe949771602e54533d9dd5c63ac4a1d718fa.jpg)

# 不显示_source

对于这种统计查询来说，不需要显示_source，这样可以提高查询效率

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "match": {
    "province": "河南省"
    }
},
"_source": false
}
```

这样查询不显示_source，能够很大程度上提高查询效率，节省网络流量

← → ⬆ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新 :

elastic 搜索 Elastic

≡ 默 开发工具

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/964ac89f9a95734a7030959e60a6e55850cfff70a5a5996ecb4a08174558120c.jpg)

# 复合条件查询

我们可以加入郑州市来缩小搜索范围，这里就用到了bool查询

# 常用参数

- must 查询必须匹配某些条件才可以返回

- must_not 查询必须不匹配某些条件

- should 当查询满足此条件时，会增加其_score 值

- filter 必须匹配，但是结果不会计算分值。

# 执行查询

执行下面的脚本进行查询

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "bool": {
    "must": [
    {
    "match": {
    "province": "河南省"
    }
    },
    {
    "match": {
    "city": "郑州市"
    }
    }
    ]
    }
},
"_source": false
}
```

我们发现本次查询的结果已经减少了很多，剩下了几千条

← → C 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/35932ed0c987d0e2c802fdb5b78c276b526baea18d8f7a9ac85d259d57d28340.jpg)

# 排查金水区

我们因为一些原因，金水区的房子不纳入统计，我们需要排查以下，可以使用如下的命令

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "bool": {
    "must": [
    {
    "match": {
    "province": "河南省"
    }
    },
    {
    "match": {
    "city": "郑州市"
    }
    }
    ],
    "must_not": [
    {
    "match": {
    "area": "金水区"
    }
    }
    ]
    }
},
"_source": false
}
```

这样我们就统计出来结果了，比原来的结果少了一些

← → ⚠️ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

elastic 搜索 Elastic

■ 景状 开发工具

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/138b45a4e02c890563caf5591c7877d41b0ad06ffebb44ff80db0f81c39213f2.jpg)

# 忽略评分

这里我们统计会有一个问题，就是评分问题，查询数据还需要打分影响我们的统计查询性能，我们忽略评分后效率会有一定提高，可以使用 filter 代替 match 查询

# filter查询

我们这里使用了filter替代了match，我们就不需要关心平分了，效率会有所提高这里使用 constant_score 是因为我们这里不关心相关度的排名，仅仅是过滤数据，使用 constant_score 将_score 都设置为 1

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "constant_score": {
    "filter": {
    "bool": {
    "must": [
    {
    "term": {
    "province": "河南省"
    }
    },
    {
    "term": {
    "city": "郑州市"
    }
    }
    ],
    "must_not": [
    {
    "term": {
    "area": "金水区"
    }
    }
    ]
    }
},
"boost": 1
},
"_source": false
} 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/7650a2a212765b54177ba1e800f9368a248fd4c2deef6a897ca92cc8756000b2.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/db33679f4e65d85d36a408317b053059b7b90edf771ac8aeb6ca07055804ecf5.jpg)

# 绿化率处理

我们接着上文进行效率绿化率的处理

# 显示绿化率

我们现在要显示以下小区的绿化率，因为隐藏了_source 所以导致看不到，我们可以通过如下方式进行处理

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "constant_score": {
    "filter": {
    "bool": {
    "must": [
    {
    "term": {
    "province": "河南省"
    }
    },
    {
    "term": {
    "city": "郑州市"
    }
    },
    {
    "exists": {
    "field": "greening"
    }
    }
    ],
    "must_not": [
{
    "term": {
    "area": "金水区"
    }
}
],
"boost": 1
},
"_source": [
    "name", "greening"
]
}
```

这里我们又加入了一个条件，如果没有绿化率这个值的数据就不显示，并且_source字段只显示小区名字和绿化率

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/da9f398343c712cb3fe112d7ac3c6c0eb5debd2092a524e1140ce6b5475a9d3e.jpg)

# 按照绿化率排序

下面我们要对绿化率进行排序处理，可以使用 sort 进行排序

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "constant_score": {
    "filter": {
    "bool": {
    "must": [
    {
    "term": {
    "province": "河南省"
    }
    },
    {
    "term": {
    "city": "郑州市"
    }
    ]
    }
}
},
{
    "exists": {
    "field": "greening"
    }
},
"must_not": [
    {
    "term": {
    "area": "金水区"
    }
    }
],
},
"boost": 1
},
"sort": [
    {
    "greening": {
    "order": "desc"
    }
    }
],
"_source": [
    "name",
    "greening"
]
}
```

# 这样我们就实现了小区的绿化率的排序

← → ⚠️ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/0117b76a98fcf6a17e2eb62b89c5d22face4cb1d38669a1dc2c35408ed9d2d6c.jpg)

# 分页排序

下面我们说下分页如何进行分页排序，因为默认只显示前十条数据，后面如何分页呢

from+size 的分页查询称为"浅"分页，它的原理很简单，就是查询前20条数据，然后截断前10条，只返回10-20的数据，这样其实白白浪费了前10条的查询

在深度分页的情况下，这种使用方式效率是非常低的，比如from = 50000, size=10, es需要在各个分片上匹配排序并得到50010条数据，协调节点拿到这些数据再进行全局排序处理，然后结果集中取最后10条数据返回。

# 分页示例

我们将上面的查询进行分页

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "constant_score": {
    "filter": {
    "bool": {
    "must": [
    {
    "term": {
    "province": "河南省"
    }
    },
    {
    "term": {
    "city": "郑州市"
    }
    },
    {
    "exists": {
    "field": "greening"
    }
    }
    ],
    "must_not": [
    {
    "term": {
    "area": "金水区"
    }
    }
    ]
    }
},
"boost": 1
},
"sort": [
    {
    "greening": {
    "order": "desc"
    }
}
],
"from": 20,
"size": 10,
"_source": [
"name",
"greening"
]
} 
```

其中，from定义了目标数据的偏移值，size定义当前返回的事件数目，默认from为0，size为10，即所有的查询默认仅仅返回前10条数据

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/3770b75c264872548e0e7b14993ff536006de336fdf8e7c6b0316251a748e086.jpg)

# 深度分页的问题

现在查询看起来没有什么问题，条件放宽，在查询（from：9999；size：10）第9999后面的10条数据的话就会报错

因为es最大深度是支持搜索到底10000条数据，这个：index.max_result_window字段控制的，默认是10000条数据

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "constant_score": {
    "filter": {
    "bool": {
    "must": [
    {
    "term": {
    "province": "河南省"
    }
    }
    ],
    "must_not": [
    {
    "term": {
    "area": "金水区"
    }
    }
    ]
    }
}
]
},
"boost": 1
},
"sort": [
{
    "greening": {
    "order": "desc"
    }
}
],
"from": 9999,
"size": 10,
"_source": [
    "name",
    "greening"
]
} 
```

我们在查询 from: 9999, size: 10 的时候会报错，这是因为es最大深度是支持搜索到底10000条数据，这个：index.max_result_window字段控制的，默认是10000条数据

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/47d26bccdbade254e8b9ca1c98d28789c25b94c27119bcec3f61d63324b0ab39.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/1996bf40988f7c603d1fc55e7345e19e57e0f28ac32e21eb6f9562a656b2c381.jpg)

# 为什么会报错

这是因为ES为了查询性能，限制了我们分页的深度，es目前支持的最大的max_result_window = 10000

也就是说我们不能获取10000个以上的文档，当ES分页查询超过一定的值（10000）后，会报错，如果数据量非常大的情况下进行查询可能会产生OOM

# ES分页查询的结果是这样的

- 协调节点或者客户端节点，需要讲请求发送到所有的分片

- 每个分片把 from + size 个结果，返回给协调节点或者客户端节点

- 协调节点或者客户端节点进行结果合并，如果有n个分片，则查询数据是 n * (from+size)，如果 from很大的话，会造成oom或者网络资源的浪费。

加入我有三个shard（分片），每个分片有10w条数据如果要查询9999-10009的数据，就会分别从每个分片中获取10009条数据，一共30027条数据，然后进行排序获取出10条数据，所以深度分页会给系统带来很大的压力

深度分页

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/f97af87541722522838c648654adc73c550ae9b3b6bb825ea7aaa0985b79d850.jpg)

# 小区精准搜索

上面我们讲解了小区统计搜索，讲解下常用的精确搜索

# 小区名称搜索

# match的使用

我们可以使用 match 来对小区名字进行精确搜索，match 会对字段进行全文检索，这里需要注意下，如果是 text 类型字段会进行分词后到倒排索引库进行搜索，如果是 keyword 则直接进行检索，不会分词

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "match": {
    "name": "龙苑居住区"
    }
    }
}
```

这样我们就将数据给搜索出来了

← → C 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

elastic 搜索 Elastic

≡ 默 开发工具

← → ⚫ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新

elastic 搜索 Elastic

开发工具

控制台 Search Profiler Grok Debugger Painless实验室 公测版

历史记录 设置 帮助 200-OK 18ms

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/24ead6c90e68f74d59241b2a06531df64014a7200c73f0261b022ee42bac1bae.jpg)

# 不完全匹配

因为该字段类型是 keyword 的，我们尝试下把小区名字写错试试 龙源居住区

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "match": {
    "name":"龙源居住区"
    }
    }
}
```

我们发现现在查询不出来数据的，根据映射我们知道 name 是 keyword，所以是一个完全匹配，如果匹配不上就会查询不出来

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/a32ff1a78520f8231afb6a2fa6f281ccc5cb843206d60eddf75f63f27e428a19.jpg)

# 开发商搜索

我们继续使用 match 针对 上海新安房地产有限公司 房产开发商进行搜索

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "match": {
    "producer": "上海新安房地产有限公司"
    }
    }
}
```

因为 producer 是 text 类型，搜索的条件会被分词，所以我们看到结果有些不符合我们的需求的

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/b04acedb99690f5351772edcf985023ed9147622bf1d8db5101c4556992600f5.jpg)

# 查看分词结果

match 碰到 text 类型字段会对我们搜索的关键字进行分析，分词后会到倒排索引库进行检索，只要有任何一个倒排索引符合就会返回结果，下面我们看下分析后的结果

```
GET _analyze
{
    "text": "上海新安房地产有限公司",
    "analyzer": "ik_smart"
}
```

只要倒排索引库有任何一个包含以下分词结果的都会被筛选出来

← → ⚫ 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

elastic 搜索 Elastic

≡ 默 开发工具

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/8608bc0a58f993418d99fc40a7f0389c04c8ad507b79475e5b27a7a8ed7f62e4.jpg)

# term查询

term 不会对查询字符进行分词，直接拿查询字符去倒排索引中比对，下面我们直接进行搜索

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "term": {
    "producer": "上海新安房地产有限公司"
    }
    }
}
```

我们发现现在根本就查询不出来，这是因为我们的 text 类型的开发商已经被分词了，进入了倒排索引库，我们直接搜索使用 term 时不进行分词直接搜索倒排索引库，索引查询不出来的，如何解决呢

# bool查询

Bool query 对应 lucene 的 BooleanQuery，一般由一个或者多个查询子句组成，如下表格所示

| 用法     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| must     | 查询一定包含匹配查询内容,并且提供得分                        |
| filter   | 查询一定包含匹配查询内容,但是不提供得分,会对查询结果进行缓存 |
| should   | 子查询不一定包含查询内容                                     |
| must_not | 查询一定不包含查询内容,来自于filter上下文,所以不会由评分,但是会缓存 |

在上面我们也使用过了bool查询，这里我们需要使用term查询也需要使用bool来组合条件

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "bool": { 
"must": [
    {
    "term": {
    "producer": "上海"
    }
    },
    {
    "term": {
    "producer": "新安"
    }
    },
    {
    "term": {
    "producer": "房地产"
    }
    },
    {
    "term": {
    "producer": "有限公司"
    }
    }
]
}
```

# 我们只进行搜索全部满足上面条件的文档

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/9f30204158d899d4b75683a242149f5082975b45881128cc0c02f90565f7fdc2.jpg)

# match和term区别

es中常用的两种查询方式：term和match的区别如下

- term：代表完全匹配，也就是精确查询，搜索前不会再对搜索词进行分词解析，直接对搜索词进行查找；

- match：代表模糊匹配，搜索前会对搜索词进行分词解析，然后按分词匹配查找；

- term主要用于精确搜索，match则主要用于模糊搜索；

- term精确搜索相较match模糊查询而言，效率较高；

同时总结了两种数据类型：text和keyword

- text: 查询时会进行分词解析;

- keyword: keyword类型的词不会被分词器进行解析，直接作为整体进行查询；

# 高亮显示

通过 highlight 设置查询关键字高亮，这样可以更好的显示搜索的关键字

Highlight就是我们所谓的高亮，即允许对一个或者对个字段在搜索结果中高亮显示，比如字体加粗或者字体呈现和其他文本普通颜色等。

# 查询分析

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "term": {
    "name": "龙苑居住区"
    }
},
"_source": false,
"highlight": {
    "fields": {
    "name": {}
    }
}
}
```

标记这一块就是高亮提示显示的内容

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/5d192509cb97616431466e197d9fb7b8219c01f71eee1c90c1ad500f7c7842c3.jpg)

# 模糊搜索

在实际搜索中，有时我们可能会打错字，从而导致搜索不到，比如上面搜索的龙源居住区，一字之差搜索不到

在 Elasticsearch 基于全文的查询中，除了与短语相关的查询以外，其余查询都包含有一个名为 fuzziness 的参数用于支持模糊查询，Elasticsearch 支持的模糊查询与 SQL 语言中模糊查询还不一样，SQL 的模糊查询使用“%keyword%”的形式，效果是查询字段值中包含 keyword 的记录。

Elasticsearch支持的模糊查询比这个要强大得多，它可以根据一个拼写错误的词项匹配正确的结果，例如根据firefox匹配firefox，在自然语言处理领域，两个词项之间的差异通常称为距离或编辑距离，距离的大小用于说明两个词项之间差异的大小

# 编辑距离算法

给定两个单词 word1 和 word2，计算出将 word1 转换成 word2 所使用的最少操作数，你可以对一个单词进行如下三种操作

- 插入一个字符

- 删除一个字符

- 替换一个字符

以下是一个编辑距离算法的举类

```
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse（将 'h' 替换为 'r')
rorse -> rose（删除 'r')
rose -> ros（删除 'e')
```

# 相似度查询

# fuzzy模糊查询

下面我们就是用 fuzzy 进行相似度查询

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "fuzzy": {
    "name":"龙源居住区"
    }
    }
}
```

这样就将原来查询不出来的小区搜索出来了，虽然写错了一个汉字

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/9f59f5086e7afcc0168060d3cff615a588e368ec6957b132c7c5c33ef1778187.jpg)

# 纠错提示

纠错是在用户提交了错误的词项时给出正确词项的提示，而输人提示则是在用户输人关键字时给出智能提示，甚至可以将用户未输人完的内容自动补全

Elasticsearch也同时支持纠错与提示功能，由于这两个功能从实现的角度来说并没有本质区别，所以它们都由一种被称为提示器或建议器(Suggester)的特殊检索实现，由于输入提示需要在用户输入的同时给出提示词，所以这种功能要求速度必须快，否则就失去了提示的意义

# 实现方式

下面就是使用纠错提示，我们输入龙源居住区，整体给出如下的提示选项

```
GET logstash-village-2022.08.22/_search
{
    "suggest": {
    "name-suggestion": {
    "text": "龙源居住区",
    "term": {
    "field": "name"
    }
    }
    }
}
```

# 下面就是纠错提示所显示的内容

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/697b58092689770c3da7db00b36d6e662a4757d9ca045ed14fd6ade3fdaf26d5.jpg)

# 聚合查询（扩展）

下面我们来说下聚合查询

# 概述

# 什么是容积率

小区容积率是指小区的地上总建造面积与净用面积之比，也叫建造面积毛密度，积率一般是由政府规定的，现行城市规划法规体系下编制的各类居住用地的控制性详细规划，一般而言，容积率分为

1. 独栋别墅为0.2~0.5;

1. 联排别墅为0.4~0.7;

3.6层以下多层住宅为0.8~1.2;

4.11层小高层住宅为1.5~2.0;

5.18层高层住宅为1.8~2.5;

6.19层以上住宅为2.4~4.5;

1. 住宅小区容积率小于1.0的，为非普通住宅

# 指标聚合

下面我们说先常用的统计查询

# 计算容积率最高的小区

我们计算下小区容积率最高的小区

```
GET logstash-village-2022.08.22/_search
{
    "aggs": {
    "max_volume": {
    "max": {
    "field": "volume",
    "missing": 0
    }
    }
},
"_source": false
} 
```

我们发现 容器率最高的小区高达 100，小区一定非常拥挤，missing 代表如果值为空给出默认值

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/4cebfd710712a52160a902e6a0539ebdfef53824cae8955259658d500f4dc2dc.jpg)

# 计算小区停车位最少的小区

这个和上面的统计很类似，只是统计类型变成了最小值

```
GET logstash-village-2022.08.22/_search
{
    "aggs": {
    "min_parkings": {
    "min": {
    "field": "parkings",
    "missing": 0
    }
    }
},
"_source": false
} 
```

我们发现全国小区中停车位最少的小区，停止位为负值，离了谱了

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/769d4f560b98b3f393feb95eeda77cb4d42fa20cd45057da439aa7bafdfde7d8.jpg)

# 计算河南省绿化率的情况

看绿化率要从多个维度来查看，我们就是用Stats来统计，可以同时返回min、max、sum、count、avg结果

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "match": {
    "province": "河南省"
    }
},
"aggs": {
    "stats_greenings": {
    "stats": {
    "field": "greening",
    "script": {
    "source": "if (doc['greening'].size() != 0) {doc.greening.value}"
    }
    }
    }
},
"size": 0
```

}

这里引入了一个 script 它可以帮我们忽略问题文档，让我们的统计结果更加精准，“size”：0 可以让我们不关注文档，只关注与统计

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/bf267c38e8de217aca904dd9602c6ca60b3cb535e6a365d48b14e00fc07a0179.jpg)

我们发现河南省绿化率的平均值是 32%，最高的绿化率是 70%，最低的绿化率只有 1%，居住环境差异还是很大的。

# 分桶聚合

这个也就是我们常说的分组统计

# 各省小区的数量

我们用分桶聚合，统计下各省的小区数量

```
GET logstash-village-2022.08.22/_search
{
    "aggs": {
    "bucket_terms": {
    "terms": {
    "field": "province",
    "size": 10
    }
    }
},
"size": 0
} 
```

这样我们就统计出来了各省的小区的数量有多少，我们发现小区最多的省份是广东省

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/f22c2f54f019d5bcafb40918518034b9a4a7c6f80bd921a6ab99b673d989246e.jpg)

# 统计各省小区认数

除了统计上面节结果，我们还需要统计下各省住小区的人数总和是多少，我们会用到 sum 进行求合，这个就类似于聚合后在进行统计

```
GET logstash-village-2022.08.22/_search
{
    "aggs": {
    "bucket_terms": {
    "terms": {
    "field": "province",
    "size": 10
    },
    "aggs": {
    "sum_houses": {
    "sum": {
    "field": "houses"
    }
    }
    }
    }
},
"size": 0
} 
```

这样我们就求出来了各省住小区的总人数有多少

← → C 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/a29125b4ec25b0fa51fb637b0592f030bb058e4aa62a194d253dee7a651a6cf8.jpg)

# 增加学区房条件

我们在上面统计的基础上再加上学区房的限制，计算哪个省多少人在学区房，多少人住在学区房中，我们可以先进筛选，筛选完成的结果在进行聚合

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "exists": {
    "field": "school"
    }
    },
    "aggs": {
    "bucket_terms": {
    "terms": {
    "field": "province",
    "size": 10
    },
    "aggs": {
    "sum_houses": {
    "sum": {
    "field": "houses"
    }
    }
    }
    }
},
"size": 0
} 
```

这样我们就筛选出来结果了，我们发现结果差异还是比较大的

← → C 不安全 | 192.168.245.151:5601/app/dev_tools#/console 更新：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/921f6fef125446ef39d2c8bd2f6d0e58f970c2dd8ecefcff3da51f2550f18f8e.jpg)

# 管道聚合

管道聚合相当于在之前聚合的基础上，进行再次聚合

# 查找学区房人数最多的省份

根据上面的聚合结果再次进行聚合，找到学区房最多人数的省份

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "exists": {
    "field": "school"
    }
    },
    "aggs": {
    "bucket_terms": {
    "terms": {
    "field": "province",
    "size": 10
    },
    "aggs": {
    "sum_houses": {
    "sum": {
    "field": "houses"
    }
    }
    }
    },
    "max_houses": {
    "max_bucket": {
    "buckets_path": "bucket_terms>sum_houses"
    }
    }
},
"size": 0
} 
```

最终我们查到结果是广东省，这里面 buckets_path 指的是聚合的路径，

bucket_terms>sum_houses 指的是聚合 bucket_terms 下的 sum_houses 聚合进行在聚合

← → C ▲ 不安全 | 192.168.245.151:5601/app/dev_tools#/console

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/26c3207ce8adfc6002de371d873ab2b435630fbd288de1dd2475a3f8e0f8c4f1.jpg)

# 整体情况反应

只靠一个聚合很难看到具体结果，我们可以通过 Stats 来进行显示更多的聚合结果

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "exists": {
    "field": "school"
    }
    },
    "aggs": {
    "bucket_terms": {
    "terms": {
    "field": "province",
    "size": 10
    },
    "aggs": {
    "sum_houses": {
    "sum": {
    "field": "houses"
    }
    }
    }
    },
    "stats_houses": {
    "max_bucket": {
    "buckets_path": "bucket_terms>sum_houses"
    }
    }
},
"size": 0
} 
```

反映出来全国的学区房的情况

← → ⚠️ 不安全 | 192.168.245.151:5601/app/dev_tools#/console

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/7c867a264abc2c84567aa7a8608dc77a417e833e514c00c3b4e83126453febc7.jpg)

# GEO查询

GEO查询现在也用的比较多的，我们下面举几个例子

# 定位坐标

我们GEO查询需要有一个定位点，我们找到金燕龙办公楼的GEO坐标，这里我们使用百度的坐标反查系统，地址是http://api.map.baidu.com/lbsapi/getpoint/

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/acd07c2f718bfad9b9f99bdae209a3710ece88fcbee918a45c8a7f10c696d5b5.jpg)

# 范围查询

我们查询以金燕龙范围10KM以内的小区

```
GET logstash-village-2022.08.22/_search
{
    "query": {
    "bool": {
    "filter": [
    {
    "geo_distance": {
    "distance":"5km",
    "location": {
    "lat": 40.066258,
    }
    }
    }
    }
} 
"lon": 116.349936
}
}
}
]
}
} 
```

这样我们就查询出来 金燕龙办公楼 周围5公里内的小区

```
192.168.245.151:5601/app/dev_tools#/console
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/a21d4c0e-590b-4308-b7b2-cf4bcaef3141/8b69facd089ea7df7382fb3cf29973070f9a2b25fa56251517359a98412c4f0e.jpg)