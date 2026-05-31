# 尚硅谷大数据技术之 InfluxDB

（作者：尚硅谷研究院）

版本：V1.0

# 第1章 认识 InfluxDB

# 1.1 InfluxDB 的使用场景

InfluxDB 是一种时序数据库，时序数据库通常被用在监控场景，比如运维和 IOT（物联网）领域。这类数据库旨在存储时序数据并实时处理它们。

比如。我们可以写一个程序将服务器上 CPU 的使用情况每隔 10秒钟向 InfluxDB 中写入一条数据。接着，我们写一个查询语句，查询过去 30 秒 CPU 的平均使用情况，然后让这个查询语句也每隔 10秒钟执行一次。最终，我们配置一条报警规则，如果查询语句的执行结果>xxx，就立刻触发报警。

上述就是一个指标监控的场景，在 IOT 领域中，也有大量的指标需要我们监控。比如，机械设备的轴承震动频率，农田的湿度温度等等。

# 1.2 为什么不用关系型数据库

# 1.2.1 写入性能

关系型数据库也是支持时间戳的，也能够基于时间戳进行查询。但是，从我们的使用场景出发，需要注意数据库的写入性能。通常，关系型数据库会采用 B+树数据结构，在数据写入时，有可能会触发叶裂变，从而产生了对磁盘的随机读写，降低写入速度。

当前市面上的时序数据库通常都是采用LSM Tree的变种，顺序写磁盘来增强数据的写入能力。网上有不少关于性能测试的文章，同学们可以自己去参考学习，通常时序数据库都会保证在单点每秒数十万的写入能力。

# 1.2.2 数据价值

我们之前说，时序数据库一般用于指标监控场景。这个场景的数据有一个非常明显的特点就是冷热差别明显。通常，指标监控只会使用近期一段时间的数据，比如我只查询某个设备最近 10 分钟的记录，10 分钟前的数据我就不再用了。那么这 10 分钟前的数据，对我们来说就是冷数据，应该被压缩放到磁盘里去来节省空间。而热数据因为经常要用，数

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

据库就应该让它留在内存里，等待查询。而市面上的时序数据库大都有类似的设计。

# 1.2.3 时间不可倒流，数据只写不改

时序数据是描述一个实体在不同时间所处的不同状态。

![image](assets/0dbccd87ef133a8246bb728dec32818a6d56f594679cd5b2e6b865d71fe4d29f.jpg)


就像是我们打开任务管理器，查看 CPU 的使用情况。我发现 CPU 占用率太高了，于是杀死了一个进程，但10秒前的数据不会因为我关闭进程再发生改变了。

这是时序数据的一大特点。与之相应，时序数据库基本上是插入操作较多，而且还没有什么更新需求。

# 1.3 1.X 的 TICK 技术栈与 2.X 的进一步融合

根据上文的介绍，我们首先可以知道时序数据一般用在监控场景。大体上，数据的应用可以分为4步走。

（1）数据采集

（2）存储

（3）查询（包括聚合操作）

（4）报警

这样一看，只给一个数据库其实只能完成数据的存储和查询功能，上游的采集和下游的报警都需要自己来实现。因此 InfluxData在 InfluxDB1.X的时候推出了 TICK生态来推出start全套的解决方案。

TICK4个字母分别对应4个组件。

⚫ T : Telegraf -数据采集组件，收集&发送数据到 InfluxDB。

⚫ I : InfluxDB - 存储数据&发送数据到 Chronograf。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ C : Chronograf - 总的用户界面，起到总的管理功能。

⚫ K : Kapacitor - 后台处理报警信息。

![image](assets/f9eab9b28e818d669d8cd1b86bd427994b2ed20367fa44452b532c547fa6db97.jpg)


到了 2.x，TICK 进一步融合，ICK 的功能全部融入了 InfluxDB，仅需安装 InfluxDB 就能得到一个管理页面，而且附带了定时任务和报警功能。

# 1.4 influxDB 版本比较与选型

# 1.4.1 版本特性比较

2020年 InfluxDB推出了 2.0的正式版。2.x同 1.x相比，底层引擎原理相差不大，但会涉及一些概念的转变（例如 db/rp 换成了 org/bucket）。另外，对于 TICK 生态来说，1.x 需要自己配置各个组件。2.x则是更加方便集成，有很棒的管理页面。

另外，在查询语言方面，1.x是使用 InfluxQL进行查询，它的风格近似 SQL。2.x推出了 FLUX 查询语言，可以使用函数与管道符，是一种更符合时序数据特性的更具表现力的查询语言。

# 1.4.2 选型，本文档使用 InfluxDB 2.4

⚫ 市场现状：目前企业里面用 InfluxDB 1.X 和 InfluxDB 2.X 都有人在用，数量上InfluxDB1.X 占多一些。

⚫ 易用性：在开发中，InfluxDB 1.X集成生态会比较麻烦，InfluxDB 2.X相对来说更加便利。

⚫ 性能：InfluxDB 1.X和2.X的内核原理基本一致，性能上差距不大。

⚫ 集群：InfluxDB 从 0.11 版本开始，就闭源了集群功能的代码。也就是说，你只能免

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网费试用 InfluxDB 的单节点版（开源），想要集群等功能就需要购买企业版。不过就InfluxDB 1.8 来说，有开源项目根据 0.11 的代码思路提供了 InfluxDB 开源的集群方案。也有开源项目给 InfluxDB 2.3 增加了反向代理功能，让我们可以横向拓展 InfluxDB 的服务能力。项目参考地址：

InfluxDB Cluster 对应 1.8.10：https://github.com/chengshiwen/influxdb-cluster

InfluxDB Proxy 对应 1.2 - 1.8：https://github.com/chengshiwen/influx-proxy

InfluxDB Proxy 对应 2.3：https://github.com/chengshiwen/influx-proxy/tree/influxdb-v2

⚫ FLUX 语言支持：自 InfluxDB 1.7 和 InfluxDB 2.0 以来，InfluxDB 推出了一门独立的新的查询语言 FLUX，而且作为一个独立的项目来运作。InfluxData公司希望 FLUX语言能够成为一个像 SQL 一样的通用标准，而不仅仅是查询 InfluxDB 的特定语言。而且不管是你是选择 InfluxDB 1.X 还是 2.X 最终都会接触到 FLUX。不过 2.X 对 FLUX 的支持性要更好一些。

⚫ InfluxDB 产品概况：

InfluxDB 1.8 在小版本上还在更新，主要是修复一些BUG，不再添加新特性

InfluxDB 2.4 这是 InfluxDB 较新的版本，仍然在增加新的特性。

InfluxDB 企业版1.9 需要购买，相比开源版，它有集群功能。

InfluxDB Cloud，免部署，跑在 InfluxData 公司的云服务器上，你可以使用客户端来操作。功能上对应开源版的 2.4

⚫ 2.x 与 1.x的主要区别：两个版本的内核原理基本一致，性能上的差别不大。差别主要是在，权限管理方式不同，2.x TICK 的集成性比 1.x 好，1.x 中的 database到了 2.x 中变成了 bucket 等。

最终，本课程选择 Influx 2.4 来进行教学，学会使用 InfluxDB 2.4 后应当也能胜任InfluxDB 1.7 及以上版本的开发。授课过程遇到与 InfluxDB1.8 不同的地方，会做一些提醒。

# 第2章 安装部署 InfluxDB

# 2.1 下载安装

在linux环境下有两种安装方式

⚫ 通过包管理工具安装，比如 apt 和 yum

⚫ 直接下载可执行二进制程序的压缩包

本课程选用第二种方式，你可以使用下面的命令下载程序包。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```batch
wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.4.0-linux-amd64.tar.gz 
```

如果下载失败的话，你也可以在课程资料中获取此安装包。课程资料可以通过关注尚硅谷微信公众号，回复“大数据”关键字领取。

```txt
一 22 8月 - 01:22 /opt/software
@dengziql wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.4.0-linux-amd64.tar.gz
--2022-08-22 01:22:30-- https://dl.influxdata.com/influxdb/releases/influxdb2-2.4.0-linux-amd64.tar.gz
正在解析主机 dl.influxdata.com (dl.influxdata.com)... 143.204.89.102, 143.204.89.49, 143.204.89.84, ...
正在连接 dl.influxdata.com (dl.influxdata.com)|143.204.89.102|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 92098809 (88M) [application/x-tar]
```

压缩包下载好后，将其解压到目标路径。

```batch
tar -zxvf influxdb2-2.4.0-linux-amd64.tar.gz -C /opt/module 
```

Go 语言开发的项目一般来说会只打包成单独的二进制可执行文件，也就是解压后目录下的 influxd 文件，这一文件中全是编译后的本地码，可以直接跑在操作系统上，不需要安装额外的运行环境或者依赖。

```txt
-rwxr-xr-x. 1 atguigu atguigu 150164784 Aug 19 03:44 influxd
-rw-rw-r--. 1 atguigu atguigu 1067 Aug 19 03:44 LICENSE
-rw-rw-r--. 1 atguigu atguigu 9830 Aug 19 03:44 README.md 
```

现在，可以运行使用下面的命令，正式开启InfluxDB服务进程。

```txt
./influxd 
```

# 2.2 进行初始化配置

使用浏览器访问 http://hadoop102:8086。如果是安装后的首次使用，InfluxDB 会返回一个初始化的引导界面。按照给定步骤完成操作就好。

![image](assets/c7d94052b69292eeaf17ae6649e58188c4e31413adbc3af42397dc436cf5b174.jpg)


# 2.2.1 创建用户和初始化存储桶

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

点击GET STARTED按钮，进入下一个步骤（添加用户）。如图所示，你需要填写、组织名称、用户名称、用户密码。

![image](assets/28cc37db636a287b38e6eb9038b0306723312c314a82f656926a93bc61f86002.jpg)


填写完后点击CONTINUE按钮进入下一步。

# 2.2.2 配置完成

看到如图所示的页面，说明我们已经开始使用tony这一用户身份和InfluxDB交互了。

![image](assets/80794088b7a8fc368c98c3a1e321dfc1079cd1cf88de7b839755b6aeafdd8c5f.jpg)


# 第3章 InfluxDB 入门（借助 Web UI）

借助 Web UI，我们可以更好地理解 InfluxDB的功能划分。接下来，我们就从 Web UI入手，先了解InfluxDB的基本功能。

# 3.1 数据源相关

# 3.1.1 Load Data（加载数据）

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/c47e1ecc606b8dc1ef591e71b6b5ee04b8fbd847c805e1224f60af3242243b1c.jpg)


如图所示，页面上左侧的向上箭头，对应着 InfluxDB Web UI的 Load Data（加载数据）页面。

# 3.1.1.1 上传数据文件

在 Web UI 上，你可以用文件的方式上传数据，前提是文件中的数据符合 InfluxDB 支持的类型，包括 CSV、带 Flux 注释的 CSV 和 InfluxDB 行协议。

![image](assets/7b5f11ff5da8e79d1fd62bac90fe6c166817b814fcbc16fc23258e777cc9fa48.jpg)


点击其中任意一个按钮，将进入数据的上传页面，页面中包含了详细的说明文档，包含你的数据应该符合什么格式，你要把数据放到哪个存储桶里，还包括用命令行来上传数据的命令模板。

![image](assets/d4a9d55969ae82d6dc9e14e35e603ea40d5d4589b1ddefe103a5ec7d2fa73a48.jpg)


# 3.1.1.2 写入 InfluxDB 的代码模板

InfluxDB 提供了各种编程语言的连接库，你甚至可以在前端嵌入向 InfluxDB 写入数据的代码，因为InfluxDB向外提供了一套功能完整的REST API。

![image](assets/9239344e90ab361dadfe2e991b189ed1b8cbf34df75b2deca9e55f9a80a4b7b0.jpg)


点击任何一个语言的 LOGO，你会看到使用这门语言，将数据写入到 InfluxDB 的代码模板。

![image](assets/1832a2827ad6d30351f00a2ba7868d41c23fd6ce3f0fde18e5266f8aab0c9346.jpg)


建议从这里拷贝初始化客户端的代码。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 配置 Telegraf 的输入插件

![image](assets/bda3a012a9bdd163b1e39ebcc7affe65daa81bc41eb06ef6c9f89f9cc01ac509.jpg)


Telegraf是一个插件化的数据采集组件，在这里你可以找一下没有对应你的目标数据源的插件，点击它的 logo。可以看到这个插件配置的写法，但是关于这方面的内容，还是建议参考Telegraf的官方文档，那个更细更全一些。

![image](assets/4a8dd7f821ec92db6f0445a99907c0b4a99cb55debfc5ef77d50e3dee044efce.jpg)


# 3.1.2 管理存储桶

你可以将 InfluxDB 中的 bucket 理解为普通关系型数据库中的 database。在 Load data 页面上，点击上访的BUCKETS选项卡，就可以进入bucket管理页面了。

![image](assets/18ce18a0e8319e20c104eb633741ecc7a64de750175064bebc40d01073a0f35a.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 3.1.2.1 创建 Bucket

点击右上角的 CREATE BUCKET 按钮，会有一个创建存储桶的弹窗，这里你可以给bucket 指定一个名称和数据的过期时间。比如，你设置过期时间为 6 小时，那 InfluxDB 就会自动把这个存储桶中距离当前时间超过6小时的数据删除。

![image](assets/b375d88d29131ac6264ce03aeec5aff471844f358949ae729a6af701b05d795f.jpg)


# 3.1.2.2 调整 Bucket 的设置

存储桶的过期时间的名称都是可以修改的，点击任一 Bucket信息卡的 SETTINGS按钮会弹出一个调整设置的会话框。

![image](assets/e5b5bd123b9d58fb940dbbea46251a32fbbc270bb6724c6856e74119a4d39e55.jpg)


重命名是 InfluxDB 不建议的操作，因为大量的代码和 InfluxDB 定时任务都需要通过指定 Bucket 的名称来进行连接，贸然更改 Bucket 的名称可能导致这些程序无法正常工作。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 3.1.2.3 设置 Label

在每个 Bucket 信息卡的左下方都有一个 Add a label 按钮，点击这个按钮，你可以为Bucket添加一个标签。不过这个功能一般很少用

![image](assets/f1b59c770d7e90eb1fa32126345ba5eb19aad213d6563e8c211badf13db4b413.jpg)


# 3.1.2.4 向 Bucket 添加数据

每个存储桶信息卡的右边都有一个添加数据按钮，点击这个按钮可以快速导入一些数据。这里还可以创建一个抓取任务（被抓取的数据在格式上必须符合 prometheus 数据格式）

# 3.1.3 示例 1：创建 Bucket 并从文件导入数据

# 3.1.3.1 创建 Bucket

（1）将鼠标悬停在 四 左侧的按钮上，点击 Buckets，进入 Bucketde 的管理页面。

![image](assets/d029ed8f89c1b038002fb007972d543935d020dfe8e3b5114fe2a4d1490e4df1.jpg)


（2）点击 CREATE BUCKET 按钮，指定一个名称，这里我们将其设为 example01，删除策略保留默认的NEVER，表示永远不会删除数据

![image](assets/5b3496820368128284a7c8986d7e2c2b7a009359020d78f9c213400219e026bd.jpg)


（3）点击CREATE按钮，可以看到我们的Buckets已经创建成功了。

# 3.1.3.2 进入上传数据引导页面

在 Load Data 页面，点击 Line Prtocol 进入 InfluxDB 行协议格式数据的上传引导页面。

![image](assets/494953bb906dc0b44c7ab4044e027ac93b2d19eda40bf740a34464ec46213cbd.jpg)


# 3.1.3.3 录入数据

![image](assets/0055ecba304d7a95a611f04aae49589014bee45557f3fdfa62719e27a2cdd3bc.jpg)


（1）点击选择存储桶

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

（2）选择 ENTER MANUALLY，手动输入数据

（3）将数据粘到输入框

（4）在右侧指明时间精度，包括纳秒、微秒、毫秒和秒

数据如下：

```txt
people, name=tony age=12
people, name=xiaohong age=13
people, name=xiaobai age=14
people, name=xiaohei age=15
people, name=xiaohua age=12 
```

当前我们写的数据格式叫做 InfluxDB 行协议。你可以查看附录 2来了解这一数据格式的知识。

最后点击 WRITE DATA，将数据写到 InfluxDB。如果出现 Data Written Successfully，那么说明数据写入成功。

![image](assets/e34b54ff949342417e0e88a6528c96c5b66a73520b7442a8b08f5a01be07bd9a.jpg)


# 3.1.3.4 小结

InfluxDB 是一个无模式的数据库，也就是除了在输入数据之前需要显示创建存储桶（数据库），你不需要手动创建 measurement 或者指定各个 field 都是什么类型，你甚至可以前后在同一个 measurement下插入 filed不同的数据。

# 3.1.4 管理 Telegraf 数据源

点击 Load Data 页面的 TELEGRAF 选项卡，可以快速生成一些 Telegraf 配置文件。并向外暴露一个端口，允许 telegraf远程使用InfluxDB中生成的配置。

![image](assets/3e1b0e99aecbc00e1c2585738c9e7e6455cb8dd55c4acf02ab6b84e6fb39e810.jpg)


# 3.1.4.1 什么是 Telegraf

Telegraf 是 InfluxDB 生态中的一个数据采集组件，它可以讲各种时序数据自动采集到InfluxDB。现在，Telegraf 不仅仅是 InfluxDB 的数据采集组件了，很多时序数据库都支持与 Telegraf 进行协作，不少类似的时序数据收集组件选择在 Telegraf 的基础上二次开发。所以，我们将 Telegraf 录成了一门专门的课，大家可以到 B 站上找尚硅谷的 Telegraf 课程，将课程看到示例 3，就可以理解本课程中使用到的关于 Telegraf 的知识点了。

# 3.1.4.2 创建 Telegraf 配置文件

InfluxDB 的 Web UI 为我们提供了几种最常用的 telegraf 配置模板，包括监控主机指标、云原生容器状态指标，nginx和redis等。

![image](assets/fb948efe9d012c07d15e2dcef5b7201242cbee28305951dd3972db8ce4a9ddec.jpg)


通过页面，你可以勾选几种监控目标，然后一步步操作去创建一个 Telegraf 的配置文出来。

# 3.1.4.3 管理 Telegraf 配置文件接口

完成Telegraf的配置后，页面上会多出一个关于 telegraf实例的信息卡。如图所示：

![image](assets/3ae2e07dc85d25956d11cf1c4f82a655304c0c19f2f1aa8eaa33978050530070.jpg)


点击蓝色的 Setup Instructions。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/86479b70055f37257bb5882e71a1b33e44045fa73336bf92e787068807dd67ae.jpg)


会弹出一个对话框，引导你完成 telegraf的配置。可以看到第三步的命令。

```batch
telegraf --config
http://localhost:8086/api/v2/telegrafs/09dc7d49c444f000 
```

这个命令中有一个 URL，其实意思也就是 InluxDB 向外提供了一个 API，通过这个API你可以访问到刚才生成的配置文件。

# 3.1.4.4 修改 Telegraf 配置

已经生成的配置文件如何去修改呢？你可以点击卡片的标题。

![image](assets/148e73596bf48274c103b3c804e456fdd4ae06a3b07415a22dc66f002aea257d.jpg)


这个时候，会弹出一个配置文件的编辑页面，不过这个时候没有交互式的选项了，你需要自己直接面对配置文件。

![image](assets/e60b4dbf7b7d5d663f46c139816d9e7eef908f290c4db98912f11466f6bf31af.jpg)


修改完配置文件后，记得点击右方的 SAVE CHANGES保存修改。

# 3.1.5 示例 2：使用 Telegraf 将数据收集到 InfluxDB

在本示例中，我们会使用 Telegraf 这个工具将一台机器上的 CPU 使用情况转变成时序数据，写到我们的InfluxDB中。

# 3.1.5.1 下载 Telegraf

可以使用下面的命令下载 telegraf，也可以在本课程的配套资料中获取（关注尚硅谷微信公众号，回复“大数据”）。

```txt
wget https://dl.influxdata.com/telegraf/releases/telegraf-1.23.4_linux_amd64.tar.gz 
```

# 3.1.5.2 解压压缩包

将telegraf解压到目标路径。

```batch
tar -zxvf telegraf-1.23.4_linux_amd64.tar.gz -C /opt/module/ 
```

# 3.1.5.3 创建一个新的 Bucket

回到 Web UI界面

（1）点击左侧工具栏中的 Buckets按钮

（2）点击右侧蓝色的 CREATE BUCKET 按钮

![image](assets/b009b8310485be9204b342df047b9c4ba2f019efbde5c2bd73cf0fd1aba283c2.jpg)


（3）创建一个名为 example02 的 buckets，因为是演示，所以这里将过期时间设为 1小时。设置好后点击 CREATE

![image](assets/2dddd30e458c444bc49dc706020f71cfd23ad7cbc480d9e39ac839a5c79b9535.jpg)


（4）如果出现相应的 example02的卡片，说明存储桶已经创建成功。

![image](assets/d9f517c77f0fbef50b01bdafb56678aa35c8e7c1f521baee997b0e75d4122720.jpg)


# 3.1.5.4 在 Web UI 上创建 telegraf 配置文件

（1）在左侧的工具栏上点击 Telegraf 按钮。

（2）点击右侧蓝色的 CREATE CONFIGURATION 创建 telegraf 配置文件

![image](assets/ba802eaa9398db97f980492d52dd370123f5e3f6fa3a50bbd7beab90a955d983.jpg)


（3）在 Bucket 栏选择 example02，表示让 telegraf 将抓取到的数据写到 example02 存储桶中，下面的选项卡勾选 System。点击CONTINUE。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/133756ca2194eb5d84c02c059ec7209329d2d534d16bb972c165c24a01c89c8a.jpg)


（4）点击 CONTINUE 按钮后，会进入一个配置插件的页面。你可以自己决定是否启用这些插件。这里需要给生成的 Telegraf 配置起一个名字，方便管理。

![image](assets/21fa3210cf4ea19edb244285d3bbe12122b2c7e16f40babf6394e4d3b80e6b24.jpg)


（5）点击 CREATE AND VERIFY 按钮，这个时候其实 Telegraf 的配置就已经创建好了，你会进入一个Telegraf的配置引导界面，如图所示：

![image](assets/05956af13c530735a7ff809b07ec19318729c7ecf3547f8ea2f53b42e3cb9d0b.jpg)


# 3.1.5.5 声明 Telegraf 环境变量

按照 Web UI 上的建议，首先，你要在部署 Telegraf 的主机上声明一个环境变量叫INFLUX_TOKEN，它是用来赋予 Telegraf 向 InfluxDB 写数据权限的。这里我们就不配环境变量了，请在单一的shell会话下完成后面的操作。

所以到你下载好 Telegraf 的机器上，执行下面的命令。（注意！TOKEN 是随机生成的，请按照自己的情况修改命令）

```javascript
export INFLUX_TOKEN=v4TsUzZWtqgot18kt_adS1r-7PTsMIQkbnhEQ7oqLCP2TQ5Q-PcUP6RMyTHLy4IryP1_2rIamNarsNqDc_S_eA== 
```

# 3.1.5.6 启动 Telegraf

首先cd到我们解压的 telegraf目录。

```txt
cd /opt/module/telegraf-1.23.4 
```

![image](assets/299ff451660a1e6c85cf701d392e8c5ddd9e76a6af11c434de5697c264903f16.jpg)


telegraf 的可执行文件在 ./usr/bin 目录下。cd 过去。

```txt
cd ./usr/bin 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

从 Web UI 中复制运行 telegraf 的命令，修改 host 然后执行，老师的 telegraf 和InfluxDB在同一台机器上，所以可以使用 localhost。最终命令如下。

```batch
telegraf --config
http://localhost:8086/api/v2/telegrafs/09dcf4afcfd90000telegraf 
```

运行效果如下图所示。

```txt
一 22 8月 - 15:46 /opt/module/telegraf-1.23.4/usr/bin
@atguigu ./telegraf --config http://localhost:8086/api/v2/telegrafs/09dcf4afcfd90000
2022-08-22T07:46:48Z I! Starting Telegraf 1.23.4
2022-08-22T07:46:48Z I! Loaded inputs: cpu disk diskio mem net processes swap system
2022-08-22T07:46:48Z I! Loaded aggregators:
2022-08-22T07:46:48Z I! Loaded processors:
2022-08-22T07:46:48Z I! Loaded outputs: influxdb_v2
2022-08-22T07:46:48Z I! Tags enabled: host=dengziqi-WUJIE-16
2022-08-22T07:46:48Z I! [agent] Config: Interval:10s, Quiet:false, Hostname:"dengziqi-WUJIE-16", Flush Interval:10s
```

# 3.1.5.7 验证数据采集结果

人（1）点击左侧 按钮进入 Data Explorer 页面。

（2）在左下角第一个选项卡选择 example02，表示要从 example02 这个存储桶中查数据。

（3）点击好第一个选项卡后，会自动弹出第二个选项卡，勾选 cpu。

（4）点击右上方的 SUBMIT按钮。

（5）如果出现折线图，说明我们成功地使用Telegraf把数据导进来了。

![image](assets/2de40d0895b0d890570011fd3c3507d94f8c31cbf1f48e6f9773c0c4c0fff6ab.jpg)


# 3.1.5.8 编写启停脚本

后面我们很多时候都要使用 telegraf抓取的主机监控数据来进行查询演示。为了方便启停，我们编写一个shell脚本来管理 telegraf任务。

（1）首先 cd 到~/bin 路径下，如果~路径下没有 bin，就创建 bin 这个目录。通常，

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

~/bin是 PATH环境变量包含的一个目录。

```txt
cd ~
mkdir bin
cd ~/bin 
```

（2）到~/bin 路径下创建一个文件 host_tel.sh

```batch
vim host_tel.sh 
```

（3）键入如下内容

```shell
#!/bin/bash

is_exist() {
    pid=`ps -ef | grep telegraf | grep -v grep | awk '{print $2}'`
    # 如果不存在返回 1，存在返回 0
    if [ -z "${pid}" ]; then
    return 1
    else
    return 0
    fi
}

stop() {
    is_exist
    if [ $? -eq "0" ]; then
    kill ${pid}
    if [ $? -eq "0" ]; then
    echo "进程号:${pid},弄死你"
    else
    echo "进程号:${pid},没弄死"
    fi
    else
    echo "本来没有 telegraf 进程"
    fi
}

start() {
    is_exist
    if [ $? -eq "0" ]; then
    echo "跑着呢, pid 是${pid}"
    else
    export INFLUX_TOKEN=v4TsUzZWtqgot18kt_adS1r-
7PTsMIQkbnhEQ7oqLCP2TQ5Q-PcUP6RMyTHLy4IryP1_2rIamNarsNqDc_S_eA==
    /opt/module/telegraf-1.23.4/usr/bin/telegraf --config
http://localhost:8086/api/v2/telegrafs/09dcf4afcfd90000
fi
}

status() {
    is_exist
    if [ $? -eq "0" ]; then
    echo "telegraf 跑着呢"
    else
    echo "telegraf 没有跑"
    fi
}
```

```txt
usage () {
    echo "哦! 请你 start 或 stop 或 status"
    exit 1
}

case "$1" in
    "start")
    start
    ;;
    "stop")
    stop
    ;;
    "status")
    status
    ;;
    *)
    usage
    ;;
esac 最后
```

（4）最后给这个脚本加上一个执行权限，你可以执行下面的代码。

```txt
chmod 755 ./host_tel.sh 
```

![image](assets/74a228b12a7d12a14b9facc11860e90cefdaed60276ae2252108bb6f6b3313e3.jpg)


# 3.1.5.9 小结

最终需要注意。InfluxDB 只是帮你管理了一下 Telegraf 的配置文件。InfluxDB 并不能管理 Telegraf 的启停和运行状态。如何运行 Telegraf 还是需要开发者手动或者编写脚本来维护的。

# 3.1.6 管理抓取任务

# 3.1.6.1 什么是抓取任务

抓取任务就是你给定一个 URL，InfluxDB 每隔一段时间去访问这个链接，把访问到的数据入库。

在 InfluxDB 1.x 的时候，类似的任务只能由 Telegraf 来实现。在 InfluxDB 2.x 中，内置了抓取功能（但是定制性上不如 Telegraf，比如轮询间隔只能是 10秒）

![image](assets/0aef4693b16b1a4999ee64b7b7cfeae9416ed6de30c07f6538b484c105faab7f.jpg)


另外，目标 URL 暴露出来的数据格式必须得是 Prometheus 数据格式。关于Prometheus 数据格式的详细介绍同学们可以参考本文档的附录 3

# 3.1.6.2 InfluxDB 自身暴露的监控接口

你可以访问 http://localhost:8086/metrics 来查看 InfluxDB 暴露出来的性能数据。这里面有，InfluxDB 的 GC 情况

```tcl
← → Ⓗ ⓘ localhost:8086/metrics

# HELP boltdb_reads_total Total number of boltdb reads
# TYPE boltdb_reads_total counter
boltdb_reads_total 6018
# HELP boltdb_writes_total Total number of boltdb writes
# TYPE boltdb_writes_total counter
boltdb_writes_total 37
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.7819e-05
go_gc_duration_seconds{quantile="0.25"} 7.2565e-05
go_gc_duration_seconds{quantile="0.5"} 0.000115633
go_gc_duration_seconds{quantile="0.75"} 0.00016729
go_gc_duration_seconds{quantile="1"} 0.005084312
go_gc_duration_seconds_sum 0.015866833
go_gc_duration_seconds_count 88
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 1222
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.18.5"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge 
```

以及各个API的使用情况，如图所示，说的是各个API被谁请求过多少次。

```csv
http_api_requests_total{handler="platform",method="GET",path="/6d375d8acf.png",response_code="200",status="2XX",user_agent="Edg"} 1
http_api_requests_total{handler="platform",method="GET",path="/9763d95516.png",response_code="200",status="2XX",user_agent="Edg"} 1
http_api_requests_total{handler="platform",method="GET",path="/:fallback_path",response_code="200",status="2XX",user_agent="Chrome"} 1
http_api_requests_total{handler="platform",method="GET",path="/:fallback_path",response_code="200",status="2XX",user_agent="Edg"} 1
http_api_requests_total{handler="platform",method="GET",path="/:file_name.js",response_code="200",status="2XX",user_agent="Edg"} 23
http_api_requests_total{handler="platform",method="GET",path="/:file_name.svg",response_code="200",status="2XX",user_agent="Edg"} 161
http_api_requests_total{handler="platform",method="GET",path="/:file_name.wasm",response_code="200",status="2XX",user_agent="Edg"} 1
http_api_requests_total{handler="platform",method="GET",path="/:file_name.woff2",response_code="200",status="2XX",user_agent="Edg"} 7
http_api_requests_total{handler="platform",method="GET",path="/api/v2/authorizations",response_code="200",status="2XX",user_agent="Edg"} 3
http_api_requests_total{handler="platform",method="GET",path="/api/v2/buckets",response_code="200",status="2XX",user_agent="Edg"} 10
http_api_requests_total{handler="platform",method="GET",path="/api/v2/dashboards",response_code="200",status="2XX",user_agent="Chrome"} 13
http_api_requests_total{handler="platform",method="GET",path="/api/v2/dashboards",response_code="200",status="2XX",user_agent="Edg"} 1
http_api_requests_total{handler="platform",method="GET",path="/api/v2/flags",response_code="200",status="2XX",user_agent="Chrome"} 9
http_api_requests_total{handler="platform",method="GET",path="/api/v2/flags",response_code="200",status="2XX",user_agent="Edg"} 2
http_api_requests_total{handler="platform",method="GET",path="/api/v2/labels",response_code="200",status="2XX",user_agent="Chrome"} 13
http_api_requests_total{handler="platform",method="GET",path="/api/v2/labels",response_code="200",status="2XX",user_agent="Edg"} 2
http_api_requests_total{handler="platform",method="GET",path="/api/v2/me",response_code="200",status="2XX",user_agent="Chrome"} 19
http_api_requests_total{handler="platform",method="GET",path="/api/v2/me",response_code="200",status="2XX",user_agent="Edg"} 124
http_api_requests_total{handler="platform",method="GET",path="/api/v2/orgs",response_code="200",status="2XX",user_agent="Chrome"} 9
http_api_requests_total{handler="platform",method="GET",path="/api/v2/orgs",response_code="200",status="2XX",user_agent="Edg"} 2
http_api_requests_total{handler="platform",method="GET",path="/api/v2/scrapers",response_code="200",status="2XX",user_agent="Edg"} 3
http_api_requests_total{handler="platform",method="GET",path="/api/v2/setup",response_code="200",status="2XX",user_agent="Chrome"} 10
http_api_requests_total{handler="platform",method="GET",path="/api/v2/setup",response_code="200",status="2XX",user_agent="Edg"} 5
http_api_requests_total{handler="platform",method="GET",path="/api/v2/tasks",response_code="200",status="2XX",user_agent="Chrome"} 1
http_api_requests_total{handler="platform",method="GET",path="/api/v2/telegrafs",response_code="200",status="2XX",user_agent="Edg"} 2 
```

# 3.1.7 示例 3：让 InfluxDB 主动拉取数据

# 3.1.7.1 创建一个存储桶

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

如图所示，我们创建了一个名为 example03的存储桶。数据的过期时间设为1小时。

![image](assets/1b3f956784c3040b07578734461c33fde3b07b626d9884037a78b17cc558f0f7.jpg)


# 3.1.7.2 创建抓取任务

（1）进入抓取任务的管理页面

（2）点击 CREATE SCRAPER 按钮，创建抓取任务。

![image](assets/6994213c317a04e9ce4c67f97d0d6c31cf8672dfc413223577946c64f6d5f19f.jpg)


（3）在对话框上，给抓取任务起一个名字，此处命名为 example03_scraper

（4）右方的下拉框上，选择我们刚才创建的存储桶，example03。

（5）最下方设置一下目标路径，最后点击 CREATE

![image](assets/65b6b24eeeaa6ca0c0fa227aeff5ee9e804d262dba50506667daf7a375a57277.jpg)


（6）如果页面上出现新的卡片，说明配置成功。接下来去看一下数据有没有进来。

![image](assets/4e3668d2192f8c455b596a1cb47b158ad9044524ee4172adf92ae5e71402efd9.jpg)


# 3.1.7.3 验证抓取结果

（1）点击左侧的按钮，打开 Data Explorer

（2）在左下角第一个卡片选择要从哪个存储桶抽取数据，本例对应的是 example03

（3）第一个卡片选择好后，会自动弹出第二个卡片，你可以选择任意一个指标名称。

（4）点击右侧的SUBMIT按钮，提交查询。

（5）如果折线图成功加载，说明有数据了，抓取成功！

![image](assets/c0bd7e89e38cba044f7f9108caeb00491e3f634a0e51b9bb53ed539c4a9643bc.jpg)


# 3.1.7.4 补充

# 1） InfluxDB的监控数据默认会被抓取到初始化的存储桶中

抓取任务管理面板上，我们发现自己还没创建什么东西呢，就有一个抓取任务。

![image](assets/550af6a21943e0e1b117e07e2d82e9f748de9dc50689f7504ff963453cf51c83.jpg)


这个抓取任务是 InfluxDB 自动为我们创建的，它会把我们刚才访问/metrics 拿到的数据写到 test_init 这个存储桶中去，而 test_init 这个存储桶是我们首次登录的时候为了初始化而创建的。所以大家要知道 test_init中的一些监控数据是怎么产生的。

# 2） InfluxDB的抓取任务都是10秒一次，无法自定义设置

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

至少截至目前（2.4版本），用户无法去自定义抓取间隔。InfluxDB会每隔 10秒一次去抓取数据，这一点需要注意。

# 3.1.8 管理 API Token

点击左侧的 API Tokens 按钮，进入 API Token 的管理页面。

![image](assets/cca0968e0582bfae8ea4df6124663dcfbc0b63030424c00bdc93e11508f3e857.jpg)


# 3.1.8.1 API Token 是干什么用的

简单来说，influxdb 会向外暴露一套 HTTP API。我们后面要学的命令行工具什么的，其实都是封装的对 influxdb 的 http 请求。所以，在 InfluxDB 中，对权限的管理主要就体现在 API 的 Tokens 上。客户端会将 token 放到 http 的请求头上，influxdb 服务端就根据客户端发来的请求头部的 token，来判断你能不能对某个存储桶读写，能不能删除存储桶，创建仪表盘等。

# 3.1.8.2 查看 API Token 权限

截至目前，我们还没有自己手动创建过 API Token。但是可以看到页面上已经有一些Token了，这些Token是由我们之前示例里面的操作自动生成的。

![image](assets/ce79e7ee31d44cb4cf5826f6ec181b7af362ee7d4eb1f528b2011236235f615e.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 3.1.8.3 了解 tony's Token

现在，我们围绕着 InfluxDB 中已有的 Token 来学习相关的知识，我们的 InfluxDB 上现在只有初始化时创建的 tony 账户，在 Token 列表中，我们可以看到有一个名为 tony's Token的 token。

![image](assets/bec96eca302bdcf441d6ac767e07c29d96b8aab818bdb8b130d3704c7f6704e6.jpg)


# 1） 修改 token 的名称

点击token右边的 符号，可以修改token名称。

⚫ 没有客户端会用 token 的名称来调用 token，所以修改 token 名称不会影响已经部署的应用。

⚫ InfluxDB从未要求 token的名称必须全局唯一，所以名称重复也是可以的。如图：

![image](assets/dc0b060fc2d90693a6e34db059346fb3341d10e532d9a6cd7ddf80378bf338c3.jpg)


# 2） token可以临时关停、也可以删除

正如你说看到，token卡片下面的 Active按钮是一个开关，可以在启用和停用之间进行切换。

![image](assets/258c474f4ccd1b69a195d971305b0137f0e05875aa0081fe81081c7ce14846e5.jpg)


同时，你也可以删除 token，但是这可能对你已经部署的应用产生不可挽回的影响。

# 3） 查看 Token 权限

点击token的名称，可以看到这个 token具体有哪些权限。

这里我们比较两个token，可以看到tony'Token的权限很高。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/70b81ef883fc3050bbc85d500b74961098b6a219d2a1286a7a4b5aa24dbe8ef0.jpg)


下面这个Token是我们前面示例，生成Telegraf配置的时候自动生成的 token。

WRITE example02 bucket / READ example02_conf telegraf config 

Created at: 2022-08-22 12:35:15 

![image](assets/eda28f86fc7f2581bea152568ba09800e1561de9dc81b99b5a2aa51060011b2e.jpg)


Active 

点开看一下它的权限。

![image](assets/d9d4899ae0e737f8f34e5c3e66e782d049bcc7d10ed506a83cd0d3e4cf8fbd37.jpg)


可以看到这个 token 的权限就小得多了，它只能向一个存储桶里写数据，查的权限都没有呢。

# 3.1.8.4 创建 API Token

页面的右方有一个 GENERATE API TOKEN。点一下会出来一个下拉菜单，这其实是Web UI上的权限模板

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/d49414985a3dbd72907e8bffa4d16061722e0256e2cb77c9875f37d430b8dd2d.jpg)


在 Web UI 上，有两种类型的模板让你可以快速创建 token。

⚫ Read/Write API Token 仅读写存储桶的 Token

创建Token时还可以限定这个Token能操作哪些存储桶。

![image](assets/b0fd99bc3c038973ce9da388046fcd69efdaf0a67c7b9a11fadde542252837b2.jpg)


⚫ All Access API Token 生成带所有权限的 Token

![image](assets/d56c5827d8087a36b61c3676949cac229d34d6c3cfa029852fcf1050c4b1a2e2.jpg)


注意！InfluxDB的 Token是可以进行更细的管理的，Web UI上给的只是生成 Token的模板，准备了用户的常用需求，但不代表它的全部功能。

# 3.2 查询工具

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

如果继续后面内容的学习，最好先掌握附录4：时序数据库中的数据模型中的知识。

# 3.2.1 前言

关于 InfluxDB的查询，需要用户掌握一门叫 FLUX的语言。本节暂时不讲解 FLUX语言的知识，而是先了解 InfluxDB 重要的两个开发工具——Data Explorer 和 Notebook。

# 3.2.2 了解 Data Explorer

# 3.2.2.1 什么是 Data Explorer

explorer，探险家、探索者的意思。所以正如其字面意思，你可以使用 Data Explorer 探索数据，理解数据。说白了，就是你可以尝试性地写写 FLUX 查询语言（InfluxDB 独创的一门独立查询语言，课程后面会讲解），看一下数据的效果。开发过程中，你可以将它作为一个 FLUX 语言的 IDE。但是，目前我们不会向大家讲解 FLUX 语言。后面会这门语言起一个专门的章节。

# 3.2.2.2 认识 Data Explorer 的页面

点击左边的 图标，进入 Data Explorer。

我们可以将 Data Explorer 的界面简单分为两个区域，上半部分为数据预览区，下半部分为查询编辑区。

![image](assets/563eb01680c5254ac2c617343a4435372b692b873487c1c3643f7671c473f903.jpg)


# 3.2.2.3 查询编辑区

查询编辑区为你提供了两种查询工具，一个是查询构造器，一个是 FLUX 脚本编辑器。

# （1）查询构造器

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

你一进入 Data Explorer 页面，默认会打开查询构造器。使用查询构造器，你可以通过点按的方式完成查询。它背后的原理其实是根据你的设置，自动生成一条 FLUX 语句，提交给数据库完成查询。

能够出现查询构造器这种东西，说明时序数据的查询之间遵循着某种规律。不同业务之间的查询步骤可能高度相似。

![image](assets/94f77a1163ed9dd7e4b6b423614adaa1f486be900ecd0d64f5537e61d75dd748.jpg)


如上图，这是查询构造器的极简介绍。在后面的示例中，我们会详细讲解它的使用

# （2）FLUX脚本编辑器

你可以手动将查询构造器切换为 FLUX 脚本编辑器。然后愉快地编写 FLUX 脚本，实现各种奇葩查询。编辑器十分友好，还带自动提示和函数文档。

![image](assets/ff88c967eea5d4eaec6298c36952643f146bc4d4547e424c1481c7b079d7507c.jpg)


# 3.2.2.4 数据预览区

数据预览区可以将你的数据展示出来。下图是一个效果图。

![image](assets/0e57dfb77dcba34af9ac2f702e1ada66d1e710f04ca6b537a34809d80764d92e.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

默认情况下，数据预览区会将你的数据展示为一个折线图。不过除此之外，你还可以让数据展示为散点图、饼图或者查看原始数据等等。

# 3.2.2.5 其他功能

除了查询和展示数据的功能外。

Data Explorer 还有一些拓展功能

# 1） 将数据导出为 CSV

在执行查询之后，DataExplorer允许你快速地将数据导出为一个CSV文件。

![image](assets/83604a0f2e7dee02b0df3fff8e5e45647b065efe9fcc5edcc1be853fecd3c3d5.jpg)


# 2） 将当前查询和可视化效果保存为仪表盘的一个单元

你可以将当前的查询逻辑和图形展示保存为某个仪表盘的一部分。这个功能需要在查询逻辑已经实现的前提下，点击右上角的SAVE AS触达。

![image](assets/dede697c39affc77bedc9e257a20202ee9c8bc425ac826ef43c818e6e02b5ba0.jpg)


# 3） 创建定时任务

![image](assets/5ab17519a335b5194f8022b33d5730c650bf354ea3945e84d543900298907517.jpg)


Data Explorer 中的查询逻辑可以保存为一个定时任务，也就是 TASK。这里提前说一下 InfluxDB 中的 TASK 是什么。TASK 其实是一个定时执行的 FLUX 语言写的脚本。因为FLUX是一个脚本语言，所以它其实有一定的 IO能力。可以使用 http与外面的系统进行通信，还可以将计算完的数据回写给 InfluxDB。所以通常TASK有两种使用场景。

（1）数据检查与报警。对查询后的结果进行一下条件判断，如果不合规，就使用 http向外通知报警。

（2）聚合操作。在 InfluxDB 里开窗完成聚合计算，计算后的数据再写回到 InfluxDB，这样下游 BI（数据看板）可以直接去查询聚合后的数据了，而不是每次都把数据从InfluxDB 里拉出来重新计算。这样可以减少 IO，不过会增加 InfluxDB 的压力。生产环境下需要根据实际情况进行取舍。

# 4） 定义全局变量

在 DataExplorer 里，你可以声明一些全局变量。全局变量的类型可以是 Map（键值对）、CSV 和 FLUX 脚本。这样，将来你可以直接引用这些变量，比如你的数据里有地区编码。你就可以将编码到地区名称的映射保存为一个全局 Map，供以后每次查询时使用。

![image](assets/6857658949fef5e48cc2548cb67176e3f7a7940a0fc20c50a083f2d1753dcbe4.jpg)


# 3.2.3 示例 4：在 Data Explorer 使用查询构造器进行查询和可视化

# 3.2.3.1 打开 Data Explorer

点击左侧的 按钮，进入 Data Explorer 页面。

![image](assets/36b7c2d313f9bd879bf028044d1e21ca776beb7563d5a38e1a7777d9c4ec3513.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 3.2.3.2 设置查询条件

我们现在要查询的是 test_init 存储桶下的 go_goroutines 测量，这个测量反应的是我们InfluxDB 进程中的 goroutines（轻量级线程）数量。

首先，在左下角的查询构造器的 FROM选项卡，选择 test_init存储桶

![image](assets/fca5cefcbc5353c70bca283a8a2e6df84c12b090fb09ca5c6c607e4d4cfd3ebb.jpg)


接着会弹出一个 Filter 选项卡，默认情况下这里是选择_measurement，此处我们选择go_goroutines。

# 3.2.3.3 注意查询时间范围

右上角有一个带时钟符号的下拉菜单，这个菜单可以帮你纵向选择要查询数据的时间范围，通常默认是1h。如下图所示：

![image](assets/733cee9ca4b9afa093ab9d693333f2309830839f892d92ee1927fd424e704fe8.jpg)


# 3.2.3.4 注意右侧的窗口聚合选项

在查询构造器的最右边，有一个开窗聚合选项卡。使用查询构造器进行查询，就必须使用开窗聚合。默认情况下，DataExplorer 会根据你设置的查询时间范围，自动调整窗口大小，此处查询范围1h对应窗口大小10s。

![image](assets/4707deab58c7479ee2cc8c34d09f4c024a87210d1e58309706784cfa5c2b2ede.jpg)


同时，聚合方式默认是平均值。

# 3.2.3.5 提交查询

点击右侧的 SUBMIT 按钮可以立刻提交查询。之后，数据展示区会出现相应的折线图。如下图所示：

![image](assets/7adc89957d009e80bc466353291a76b1344cf4ad6cde45f71486200909561a01.jpg)


点击 View Raw Data，可以看到原始数据。

![image](assets/25ac259880ab916039a62c20092df529089817353981d778adbd55f014711a6d.jpg)


# 3.2.3.6 查询原理

我们使用查询构造器进行查询，其实是 Web UI 根据我们指定的查询条件生成了一套FLUX 查询脚本。点击 SCRIPT EDITOR 按钮，可以看到查询构造器生成的 FLUX 脚本。

![image](assets/0c410cb99b6ffdb37b8df35a0745f212de9410e7c209369d1df751df38b872c4.jpg)


# 3.2.3.7 可视化原理

其实默认情况下的可视化，是依据返回数据中的_value 来展示的，但是有些时候，你想查询的数据可能字段名不会被判别为_value。它会安静地躺在原始数据中。

![image](assets/ffd8bbe389fcf061ab9510a6f292a7ca841874c728bd422a7108e957c3d84506.jpg)


# 3.2.4 了解 Notebook

# 3.2.4.1 什么是 Notebook

Notebook 是 InfluxDB2.x 推出的功能，交互上模仿了 Jupyter NoteBook。它可以用于开发、文档编写、运行代码和展示结果。

你可以将 InfluxDB 笔记本视为按照顺序处理数据的集合。每个步骤都由一个“单元格”表示。一个单元格可以执行查询、可视化、处理或将数据写入存储桶等操作。Notebook 可以帮你完成下述操作

⚫ 执行FLUX代码、可视化数据和添加注释性的片段

⚫ 创建报警或者计划任务

⚫ 对数据进行降采样或者清洗

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ 生成要和团队分享的 Runbooks

⚫ 将数据回写到存储桶

Notebook 和 DataExplorer 相比，主要是交互风格上的不同。DataExplorer 倾向于一锤子买卖，而 Notebook 可以将数据展示拆分为一个又一个具体的步骤。另外，NoteBook 可以用来开发告警任务 DataExplorer 则不能。

# 3.2.4.2 进入 Notebook 的导航界面

点击左侧的 按钮，即可进入 Notebook的导航页面。

![image](assets/aed89590944bf9c50e8a0258d3fbfa165b981044411f938ff17112cfffa8ae8c.jpg)


导航页面分两个部分：

⚫ 上面是创建引导，除了创建一个空白的 Notebook，InfluxDB 还为你提供了 3 个模板。分别是 Set an Alert（设置一个报警）、Schedule a Task （调度一个任务）、write a Flux Script（写一个Flux脚本）。

⚫ 下面是Notebook列表，过去你创建过的NoteBook再这里都会展示出来。

![image](assets/6f797723f5bbb19ad67c475652c6ede5517a853f960d41ef8219ff7ed2a1f598.jpg)


卡片上还有这个 Notebook 对应的创建时间和修改时间。通过卡片你可以对一个Notebook重命名，还可以将它复制和删除。

# 3.2.4.3 创建一个空白的 notebook

想要继续后面的步骤，我们必须先创建一个 Notebook。如下图所示，在页面上方点击New Notebook 按钮即可。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/28332643d7dafb2b904c5bd65017e50be740267de0944230b2e669cc647a540c.jpg)


现在，你看到的就是 Notebook的操作页面了。

![image](assets/76303b17de97c570f0953161dd112692264ecc256b804a7865521dc322cf6920.jpg)


# 3.2.4.4 NoteBook 工作流

目前你看到的页面应当是如下图所示的样子。

![image](assets/f742e02bded39c8066747bf5a70b12e3ff8e6e7c08454071a40b9a803bcb947e.jpg)


我们在页面中看到的一个又一个卡片，在 NoteBook 中叫做 Cell。一个 NoteBook 工作

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网流就是多个 Cell按照先后顺序组合起来的执行流程。这些 Cell中间随时可以插入别的 Cell，而且Cell和Cell还可以调换顺序。

按照Cell功能，Cell可以按照下面的方式分类。

![image](assets/fe4cd174efb6c203c6f1fa9d981158eba7be6ef4892b1ef8881a1b8923093681.jpg)


⚫ 数据源相关的Cell

查询构造器

直接编写FLUX脚本

⚫ 可视化相关的Cell

将数据展示为一个Table

将数据展示为一张图

添加笔记。

⚫ 行为 Cell

进行报警

定时任务设定

# 3.2.4.5 工作流范式

在NoteBook里编写工作流通常是有套路可循的。

![image](assets/a4bb34a0ac6becb88d87d1c2a02a2099491d22f256392651ee2a1ce99f4edea6.jpg)


通常一个 notebook工作流以查询数据开始，后面的 Cell跟上把数据展示出来，当数据需要进一步修改的时候，可以再加一个 FLUX 脚本 cell，notebook 为我们留了一个接口，通过这种方式，后面的Flux cell可以将前面的数据作为数据源进行查询。

最终，notebook 工作流可以以任务设置或者报警操作作为整个工作流的终点，当然这不是强制要求。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 3.2.4.6 NoteBook 控件

在notebook上存在下述几种控件

# 1） 时区转换

右上角有一个 Local 按钮，通过这个按钮，你可以选择将日期时间显示为系统所设时区还是UTC时间。

![image](assets/b23bd4823be604c87b3cfbf155726ad49394c4f3dcb21be5b2bbeb12496d381b.jpg)


# 2） 仅显示可视化

点击 Presentation 按钮，可以选择是否仅显示数据展示的 cell。如果开启这个选项，那么查询构造器和FLUX脚本的Cell就会被折叠。

![image](assets/1b342ed16afda06ca5b594a60cf4a061ef74b6ebc0062aa53322c607a13bd2da.jpg)


# 3） 删除按钮

点击确定后，可以删除整个 notebook。

![image](assets/8eb76b85dbe1ab5764d2694b061aa55b634cd51435be4f2d996106156b001909.jpg)


# 4） 复制按钮

右上角的复制按钮可以立刻为当前NoteBook创建一个副本。

![image](assets/a5d233de63dcd528751fe18828c9c7a935db6f3dbb484446f4d1b55562ad85c9.jpg)


# 5） 运行按钮

RUN按钮可以快速地执行Notebook中的查询操作并重新渲染其中的可视化Cell。

# 3.2.5 示例 5：使用 NoteBook 查询和可视化数据

# 3.2.5.1 使用查询构造器记性查询

默认情况下，你创建的空白 NoteBook，自带3个 cell。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/03ce3fd37c9cc66b4e4b63bfca5ad4b74acf36bf444ed3dd1bb146f122002567.jpg)


第一个 cell，默认是一个查询构造器，相对于 DataExplorer来说，notebook的查询构造器不同的地方在于它没有开窗聚合操作。

此处，同样还是查询 test_init 中的 go_goroutines 测量。

![image](assets/dee60c5714a8089f9dfb333b88b9009e7493a8bddbae2110063d28a8556b5f05.jpg)


# 3.2.5.2 提交查询

点击RUN按钮。

![image](assets/7e839b9871aad7e7cf42a3e4f6e0424c1c6490a36fd4c712905aedddb55dc716.jpg)


可以看到下面的原始数据和折线图都出现了

![image](assets/e0d2115b7e3638ecde9e5477a1598151ab5f6245a5b5668fa629089fc94049fc.jpg)


# 3.2.5.3 添加说明 cell

notebook允许用户在工作流中加入说明性的 cell。我们选择在最前面加一个说明性 cell。首先，点击左侧的紫色＋号。

![image](assets/4040c01ea4638864ca43315362166adeca43d4423e70cf352ad77e565f432fb3.jpg)


点击 NOTE 按钮。可以看到，我们已经创建了一个说明 cell。这里面还支持MarkDown 语法，

![image](assets/c4b9488e62fd5e9cc601d5370ca563ae2faf1606fd9eac3934dc9fb0d88a626f.jpg)


现在，我们随便写点东西

![image](assets/6d89a0293067779c5bbf9b1b12357d3f7c2c1596c09e2c383787fadefe755ec6.jpg)


点击右上右上角的PREVIEW按钮，markdown就会被渲染展示。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/05d7fc137981be877f3d80e8d7af5fc68aa25c082e5e8154c524d27f823fe77b.jpg)


# 第4章 FLUX 语法

# 4.1 认识 FLUX 语言

Flux 是一种函数式的数据脚本语言，它旨在将查询、处理、分析和操作数据统一为一种语法。

想要从概念上理解 FLUX，你可以想想水处理的过程。我们从源头把水抽取出来，然后按照我们的用水需求，在管道上进行一系列的处理修改（去除沉积物，净化）等，最终以消耗品的方式输送到我们的目的地（饮水机、灌溉等）。

![image](assets/558bb76331ba2c4f19c6bc5cde7b2998a032e06e49d2740347d6c8550c6cfd0c.jpg)


注意：InfluxData 公司对 FLUX 语言构想并不是仅仅让它作为 InfluxDB 的特定查询语言，而是希望它像 SQL 一样，成为一种标准。按照这个计划，FLUX 语言应该具备处理来自不同数据源的数据的能力。

# 4.2 最简示例

与处理水一样，使用 FLUX语言进行查询时会执行以下操作。

（1）从数据源中查询指定数量的数据

（2）根据时间或字段筛选数据

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

（3）将数据进行处理或者聚合以得到预期结果

（4）返回最终的结果

下面3个示例的处理逻辑都是一样的，只不过数据源有所不同，

这3个示例只是让大家看一下语法，不需要运行。

示例 1：从InfluxDB查询数据并聚合

```txt
from (bucket: "example-bucket")
    | > range(start: -1d)
    | > filter(fn: (r) => r._measurement == "example-measurement")
    | > mean()
    | > yield(name: "_results") 
```

示例2：从CSV文件查询数据并聚合

```typescript
import "csv"
csv.from(file: "path/to/example/data.csv")
| > range(start: -1d)
| > filter(fn: (r) => r._measurement == "example-measurement")
| > mean()
| > yield(name: "_results") 
```

示例3：从PostgreSQL数据库查询数据并聚合

```python
import "sql"
sql.from(
    driverName: "postgres",
    dataSourceName: "postgresql://user:password@localhost",
    query: "SELECT * FROM TestTable",
)
| > filter(fn: (r) => r.UserID == "123ABC456DEF")
| > mean(column: "purchase_total")
| > yield(name: "_results") 
```

上面3个示例用的函数都是一模一样的，下面来讲解示例中出现的代码：

⚫ from( )函数可以指定数据源。

⚫ | > 管道转发符，将一个函数的输出转发给下一个函数。

⚫ range( )，fliter( ) 两个函数在根据列的值对数据进行过滤

⚫ mean( )函数在计算所剩数据的平均值。

⚫ yield( ) 将最终的计算结果返回给用户。

# 4.3 铭记 FLUX 是一门查询语言

虽然，FLUX 语言的自我定位一个脚本语言，但是我们必须注意它也是一个查询语言的事实。因此，一个 FLUX脚本想要成功执行，它就必须返回一个表流。就像是 SQL语言想要正确执行，它就必须返回一张表。

表流是 FLUX 里提出一种数据结构，在后面的课程里我们会表流的概念进行深度的讲更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

解。另外需要注意，我们后面的代码，如果只返回一个单值，比如单个整数或者字符串这种，那就必须把这个值转换成表流才能运行。这个时候必须使用 array.from 函数。

示例如下：

```python
from "array
x = 1
array.from(rows: [{"value":x}] 
```

array.from 函数的作用就是把 x 这个单值，包装在了一个表流里面返回了。

# 4.4 注意 InfluxDB 支持的 FLUX 语言版本

需要注意，因为 InfluxDB 是一个用 Go 语言编写的数据库，它的整个项目成果就是一个单独的可执行二进制文件，所以 FLUX 语言其实也会被编译到同一个文件里。这意味着InfluxDB 和 FLUX 会有版本绑定的关系。

这里，我放了一个链接 https://docs.influxdata.com/flux/v0.x/influxdb-versions/ ，它是官方FLUX文档的一部分，这里明确记录了 InfluxDB版本的 FLUX语言版本的对应关系。


InfluxDB Open Source (OSS)


<table><tr><td>InfluxDB OSS version</td><td>Flux version</td></tr><tr><td>InfluxDB nightly</td><td>0.185.0</td></tr><tr><td>InfluxDB 2.4</td><td>0.179.0</td></tr><tr><td>InfluxDB 2.3</td><td>0.171.0</td></tr><tr><td>InfluxDB 2.2</td><td>0.162.0</td></tr><tr><td>InfluxDB 2.1</td><td>0.139.0</td></tr><tr><td>InfluxDB 2.0</td><td>0.131.0</td></tr><tr><td>InfluxDB 1.8</td><td>0.65.1</td></tr><tr><td>InfluxDB 1.7</td><td>0.50.2</td></tr></table>

# 4.5 FLUX 的基本语法

# 4.5.1 注释

在 FLUX 脚本中，没有多行注释一说，用户只能写单行注释。如果一行以两个斜杠开

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

头，那么这一行中的所有内容会被视为注释。

示例：

// 这是一行注释。

# 4.5.2 变量与复制

使用赋值运算符（=）将表达式的结果赋值变量，最终你可以使用变量名来返回变量的值。

示例：

```txt
s = "foo" // string
i = 1 // integer
f = 2.0 // float (floating point number)
s // Returns foo
i // Returns 1
f // Returns 2.0 
```

# 4.5.3 基本表达式

FLUX支持基本的表达式，比如：

⚫ + 数字相加或字符串拼接

⚫ -数字减法

⚫ *数字相乘

⚫ /数字除法

⚫ % 取模

示例：

```txt
1 + 1
// Returns 2

10 * 3
// Returns 30

(12.0 + 18.0) / (2.0 ^ 2.0) + (240.0 % 55.0)
// Returns 27.5

"John " + "Doe " + "is here!"
// Returns John Doe is here! 
```

# 4.5.4 谓词表达式

# 4.5.4.1 比较运算符

谓词表达式使用比较运算符和逻辑运算符来实现，谓词表达式的最后的返回结果只能为 true 或 false

示例：

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```txt
"John" == "John"
// Returns true
41 < 30
// Returns false
"John" == "John" and 41 < 30
// Returns false
"John" == "John" or 41 < 30
// Returns true 
```

另外

⚫ =~可以判断一个字符串时候能被正则表达式匹配上。

⚫ !~是=~的反操作，判断一个字符串是不是不能被某个正则表达式匹配。

例如：

```hcl
"abcdefg" =~ "abc|bcd"
// Returns true
"abcdefg" !~ "abc|bcd"
// Returns false 
```

# 4.5.4.2 逻辑运算符

在FLUX语言中，表示与逻辑需要使用关键字and，表示或逻辑需要使用关键字 or。

示例：

```toml
a = true
b = false
x = a and b
// Returns false
y = a or b
// Returns true 
```

最后，not可以用来进行逻辑取反。

示例：

```txt
a = true
b = not a
// Returns false 
```

# 4.5.5 控制语句

所谓控制语句是指一个编程语言中用来空值代码执行顺序的语法。

比如：

⚫ if else 

⚫ for while 循环

⚫ try catch 异常捕获

不过，在 InfluxDB中，这些语法统统没有。唯一一个和 if else比较像的是 FLUX语言

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网中的条件子句，它和 python中的条件子句功能一样且语法相似，和 java语言相比的话它有些像三元表达式。

示例如下：

```txt
x = 0
y = if x == 0 then "hello" else "world" 
```

此处，if then else 被我们成为条件子句，你需要先指定一个条件，然后当条件为 true的时候，条件子句会返回 then 后面的内容，也就是"hello"。如果是 flase，那么就会返回else 后面的内容，也就是"world"。

# 第5章 FLUX 中的数据类型

# 5.1 10个基本数据类型

# 5.1.1 Boolean （布尔型）

# 5.1.1.1 将数据类型转换为 boolean

使用bool( )函数可以将下述的4个基本数据类型转换为 boolean：

string（字符串）：字符串必须是 "true" 或 "false"

float（浮点数）：值必须是 0.0（false）或 1.0（true）

int（整数）：值必须是 0（false）或 1（true）

uint（无符号整数）：值必须是 0（false）或 1（true）

示例：

```txt
bool(v: "true")
// Returns true
bool(v: 0.0)
// Returns false
bool(v: 0)
// Returns false
bool(v: uint(v: 1))
// Returns true 
```

# 5.1.2 bytes （字节）

注意是bytes（复数）不是byte，bytes类型表示一个由字节组成的序列。

# 5.1.2.1 定义 bytes

FLUX 没有提供关于 bytes 的语法。可以使用 bytes 函数将字符串转为 bytes。

```txt
bytes(v:"hello")
// Returns [104 101 108 108 111] 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

注意：只有字符串类型可以转换为 bytes。

# 5.1.2.2 将表示十六进制的字符串转为 bytes

（1）引入"contrib/bonitoo-io/hex"包

（2）使用 hex.bytes() 将表示十六进制的字符串转为 bytes

```swift
import "contrib/bonitoo-io/hex"
hex.bytes(v: "FF5733")
// Returns [255 87 51] (bytes) 
```

# 5.1.2.3 使用 display( )函数获取 bytes 的字符串形式

使用 display( )返回字节的字符串表示形式。bytes 的字符串表示是 0x 开头的十六进制表示。

示例:

```python
import "sampledata"
sampledata.string()
    |> map(fn: (r) => ( {r with _value: display(v: bytes(v: r._value)) }))) 
```

<table><tr><td colspan="4">_result</td></tr><tr><td>table</td><td>_time</td><td>_value</td><td>tag*</td></tr><tr><td>0</td><td>2021-01-01T00:00:00Z</td><td>0x736d706c5f673971637a73</td><td>t1</td></tr><tr><td>0</td><td>2021-01-01T00:00:10Z</td><td>0x736d706c5f306d6776396e</td><td>t1</td></tr><tr><td>0</td><td>2021-01-01T00:00:20Z</td><td>0x736d706c5f706877363634</td><td>t1</td></tr><tr><td>0</td><td>2021-01-01T00:00:30Z</td><td>0x736d706c5f6775767a7934</td><td>t1</td></tr></table>

# 5.1.3 Duration 持续时间

持续时间提供了纳秒级精度的时间长度。

# 5.1.3.1 持续时间的语法

⚫ ns：纳秒

⚫ us：微秒

⚫ ms：毫秒

⚫ s ：秒

⚫ m ：分钟

⚫ h ：小时

⚫ d ：天

⚫ w ：周

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ mo：日历月

⚫ y ：日历年

示例：

```txt
1ns // 1 纳秒
1us // 1 微妙
1ms // 1 毫秒
1s // 1 秒
1m // 1 分钟
1h // 1 小时
1d // 1 天
1w // 1 星期
1mo // 1 日历月
1y // 1 日历年
3d12h4m25s // 3 天 12 小时 4 分钟又 25 秒
```

注意！持续时间的声明不要包含先导 0

比如：

```txt
01m // 解析为整数 0 和 1 分钟的持续时间
02h05m // 解析为整数 0、2 小时的持续时间，整数 0 和 5 分钟的持续时间。而不是 2 小时又 5 分钟
```

# 5.1.3.2 将其他数据类型解释为持续时间

使用duration( )函数可以将以下基本数据类型转换为持续时间

字符串：将表示持续时间字符串的函数转换为持续时间。

int：将整数先解释为纳秒再转换为持续时间

unit：将整数先解释为纳秒再转换为持续时间。

```txt
duration(v: "1h30m")
// Returns 1h30m
duration(v: 1000000)
// Returns 1ms
duration(v: uint(v: 3000000000))
// Returns 3s 
```

注意！你可以在 FLUX 语言中使用 duration 类型的变量与时间做运算，但是你不能在table 中创建 duration 类型的列。

# 5.1.3.3 duration 的算术运算

要对duration进行加法、减法、乘法或除法操作，需要按下面的步骤来。

（1）使用 int( )或 unit()将持续时间转换为 int 数值

（2）使用算术运算符进行运算

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# （3）把运算的结果再转换回Duration类型

示例：

```typescript
duration(v: int(v: 6h4m) + int(v: 22h32s)) // 返回 1d4h4m32s
duration(v: int(v: 22h32s) - int(v: 6h4m)) // 返回 15h56m32s
duration(v: int(v: 32m10s) * 10) // 返回 5h21m40s
duration(v: int(v: 24h) / 2) // 返回 12h
```

注意！声明持续时间的时候不要包含前导0，前面的零会被 FLUX识别为整数

# 5.1.3.4 时间和持续时间相加运算

（1）导入 date 包

（2）使用 date.add( )函数将持续时间和时间相加

示例：

```txt
import "date"
date.add(d: 1w, to: 2021-01-01T00:00:00Z)
// 2021-01-01 加上一周
// Returns 2021-01-08T00:00:00.000000000Z
```

# 5.1.3.5 时间和持续时间相减运算

（1）导入 date 包

（2）使用 date.add( )函数从时间中减去持续时间

示例：

```txt
import "date"
date.sub(d: 1w, from: 2021-01-01T00:00:00z)
// 2021-01-01 减去一周
// Returns 2020-12-25T00:00:00.000000000z
```

# 5.1.4 Regular expression 正则表达式

# 5.1.4.1 定义一个正则表达式

FLUX 语言是 GO 语言实现的，因此使用 GO 的正则表达式语法。正则表达式需要声明在正斜杠之间/ /

# 5.1.4.2 使用正则表达式进行逻辑判断

使用正则表达式进行逻辑判断，需要使用 =~ 和 != 操作符。=~ 的意思是左值（字符串）

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

能够被右值匹配，!~表示左值（字符串）不能被右值匹配。

```txt
"abc" =~ / \w/
// Returns true
"z09se89" =~ /^[a-z0-9]{7}$/
// Returns true
"foo" !~ /^f/
// Returns false
"FOO" =~ / (?i)foo/
// Returns true 
```

# 5.1.4.3 将字符串转为正则表达式

（1）引入 regexp 包

（2）使用 regexp.compile( ) 函数可以将字符串转为正则表达式

```txt
import "regexp"
regexp.compile(v: "^- [a-z0-9]{7}")
// Returns ^- [a-z0-9]{7} (regexp type) 
```

# 5.1.4.4 将匹配的子字符串全部替换

（1）引入 regexp 包

（2）使用 regexp.replaceAllString( )函数，并提供下列参数：

⚫ r：正则表达式

⚫ v：要搜索的字符串

⚫ t：一旦匹配，就替换为该字符串

示例：

```txt
import "regexp"
regexp.replaceAllString(r: /a(x*)b/, v: "-ab-axxb-", t: "T") // Returns "-T-T-" 
```

# 5.1.4.5 得到字符串中第一个匹配成功的结果

（1）导入 regexp 包

（2）使用 regexp.findString( )来返回正则表达式匹配中的第一个字符串，需要传递以下参数：

⚫ r：正则表达式

⚫ v：要进行匹配的字符串

示例：

```javascript
import "regexp"
regexp.findString(r:"abc|bcd",v:"xxabcwwed") 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```txt
// Returns "abc" 
```

# 5.1.5 String 字符串

# 5.1.5.1 定义一个字符串

字符串类型表示一个字符序列。字符串是不可改变的，一旦创建就无法修改。

字符串是一个由双引号括起来的字符序列，在 FLUX 中，还支持你用\x 作为前缀的十六进制编码来声明字符串。

示例：

```csv
"abc"
"string with double \ " quote"
"string with backslash \\
"日本語"
"\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"
```

# 5.1.5.2 将其他基本数据类型转换为字符串

使用srting( )函数可以将下述基本类型转换为字符串：

⚫ boolean 布尔值

⚫ bytes 字节序列

⚫ duration 持续时间

⚫ float 浮点数

uint 无符号整数

time 时间

```txt
string(v: 42) 
```

// 返回 "42"

# 5.1.5.3 将正则表达式转换为字符串

因为正则表达式也是一个基本数据类型，所以正则表达式也可以转换为字符串，但是需要借助额外的包。

（1）引入 regexp 包

（2）使用 regexp.compile( )将

# 5.1.6 Time 时间点

# 5.1.6.1 定义一个时间点

一个time类型的变量其实是一个纳秒精度的时间点。

示例：时间点必须使用RFC3339的时间格式进行声明

```txt
YYYY-MM-DD
YYYY-MM-DDT00:00:00Z 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```txt
YYYY-MM-DDT00:00:00.000Z 
```

# 5.1.6.2 date 包

date包里的函数主要是用来从Time类型的值里提取年月日秒等信息的。

比如 date.hour：

```txt
import "date"
x = 2020-01-01T19:22:31Z
date.hour(t:x)
//Returns 19 
```

# 5.1.7 Float 浮点数

# 5.1.7.1 定义一个浮点数

FLUX中的浮点数是 64位的浮点数。

一个浮点数包含整数位，小数点，和小数位。

示例：

```csv
0.0
123.4
-123.456 
```

# 5.1.7.2 科学计数法

FLUX 没有直接提供科学计数法语法，但是你可以使用字符换写出一个科学计数法表示的浮点数，再使用float( )函数将该字符串转换为浮点数。

示例：

```txt
1.23456e+78
// Error: error @1:8-1:9: undefined identifier e
float(v: "1.23456e+78")
// Returns 1.23456e+78 (float) 
```

# 5.1.7.3 无限

FLUX 也没有提供关于无限的语法，定义无限要使用字符串与 float( )函数结合的方式。

示例：

```txt
+Inf
// Error: error @1:2-1:5: undefined identifier Inf
float(v: "+Inf")
// Returns +Inf (float) 
```

# 5.1.7.4 Not a Number 非数字

FLUX语言不支持直接从语法上声明 NAN，但是你可以使用字符串与 float( )函数的方法声明一个NaN的float类型变量。

示例：

```txt
NaN 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```txt
// Error: error @1:2-1:5: undefined identifier NaN
float(v: "NaN")
// Returns NaN (float) 
```

# 5.1.7.5 将其他基本类型转换为 float

使用float函数可以将基本数据类型转换为 float类型的值。

string：必须得是一个符合数字格式的字符串或者科学计数法。

bool：true 转换为 1.0，false 转换为 0.0

int（整数）

uint（无符号整数）

示例：

```txt
float(v: "1.23")
// 1.23
float(v: true)
// Returns 1.0
float(v: 123)
// Returns 123.0 
```

# 5.1.7.6 对浮点数进行逻辑判断

使用FLUX表达式来比较浮点数。逻辑表达式两侧必须是同一种类型。

示例：

```txt
12345600.0 == float(v: "1.23456e+07")
// Returns true
1.2 > -2.1
// Returns true 
```

# 5.1.8 Integer 整数

# 5.1.8.1 定义一个整数

一个 integer 的变量是一个 64 位有符号的整数。

类型名称：int

最小值：-9223372036854775808

最大值：9223372036854775807

一个整数的声明就是普通的整数写法，前面可以加-表示负数。-0 和0是等效的。

示例：

```csv
0
2
1254
-1254 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 5.1.8.2 将数据类型转换为整数

使用int( )函数可以将下述的基本类型转换为整数：

string：字符串必须符合整数格式，由数字[0-9]组成

bool：true 返回 1，0 返回 false

duration：返回持续时间的纳秒数

time：返回时间点对应的Unix时间戳纳秒数

float：返回小数点前的整数部分，也就是截断

unit：返回等效于无符号整数的整数，如果超出范围，就会发生整数环绕

```txt
int(v: "123")
// 123
int(v: true)
// Returns 1
int(v: 1d3h24m)
// Returns 98640000000000
int(v: 2021-01-01T00:00:00Z)
// Returns 1609459200000000000
int(v: 12.54)
// Returns 12 
```

你可以在将浮点数转换为整数之前进行舍入操作。

当你将浮点数转换为整数时，会进行截断操作。如果你想进行四舍五入，可以使用math 包中的 round( )函数。

# 5.1.8.3 将表示十六进制数字的字符串转换为整数

将表示十六进制数字的字符串转换为整数，需要。

（1）引入 contrib/bonito-io/hex 包

（2）使用 hex.int( )函数将表示十六进制数字的字符串转换为整数

```txt
import "contrib/bonitoo-io/hex"
hex.int(v: "e240")
// Returns 123456 
```

# 5.1.9 UIntegers 无符号整数

FLUX 语言里不能直接声明无符号整数，但这却是一个 InfluxDB 中具备的类型。在FLUX 语言中，我们需要使用 uint 函数来讲字符串、整数或者其他数据类型转换成无符号整数。

示例：

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```go
uint(v: "123")
// 123

uint(v: true)
// Returns 1

uint(v: 1d3h24m)
// Returns 98640000000000

uint(v: 2021-01-01T00:00:00Z)
// Returns 1609459200000000000

uint(v: 12.54)
// Returns 12

uint(v: -54321)
// Returns 18446744073709497295 
```

# 5.1.10 Null 空值

# 5.1.10.1 定义一个 Null 值

FLUX 语言并不能在语法上直接支持声明一个 Null，但是我们可以通过 debug.null 这个函数来声明一个指定类型的空值。

示例：

```typescript
import "internal/debug"
// Return a null string
debug.null(type: "string")
// Return a null integer
debug.null(type: "int")
// Return a null boolean
debug.null(type: "bool") 
```

# 5.1.10.2 定义一个 null

截至目前，还无法在 FLUX语言中手动地声明一个NULL值。

注意！空字符串不是 null值

# 5.1.10.3 判断值是否为 null

你可以使用 exists（存在）这个关键字来判断目标值是不是非空，如果是空值我们会得到一个false，如果不是空值我们会得到一个 true。

示例：

```txt
import "array"
import "internal/debug"
x = debug.null(type: "string")
y = exists x
// Returns false 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 5.1.11 正则表达式类型

正则表达式在 FLUX 中作为一种数据类型，而且在语法上提供直接的支持，可以在谓词表达式中使用正则表达式。

示例：

```gitattributes
regex = /^foo/
"foo" =~ regex
// Returns true
"bar" =~ regex
// Returns false 
```

# 5.1.12 display 函数

使用 display( )函数可以将任何类型的值输出为相应的字符串类型。

示例：

```txt
x = bytes(v: "foo")
display(v: x)
// Returns "0x666f6f" 
```

# 5.2 FLUX 类型不代表 InfluxDB 类型

需要注意，FLUX 语言里有些基本数据类型比如持续时间(Duration)和正则表达式是不能放在表流里面充当字段类型的。简单来说，Duration 类型和正则表达式类型都是 FLUX语言特有的。有些类型是为了让 FLUX 在编写代码时更加方便，让它能够拥有更多的特性，但这并不代表这些类型能够存储到 InfluxDB中。

# 5.3 4 个复合类型

# 5.3.1 Record（记录）

# 5.3.1.1 定义一个 Record

一个记录是一堆键值对的集合，其中键必须是字符串，值可以是任意类型，在键上没有空白字符的前提下，键上的双引号可以省略。

在语法上，record 需要使用{}声明，键值对之间使用英文逗号（,）分开。另外，一个Record的内容可以为空，也就是里面没有键值对。

示例：0

```jsonl
{foo: "bar", baz: 123.4, quiz: -2}
{"Company Name": "ACME", "Street Address": "123 Main St.", id: 1123445} 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 5.3.1.2 从 record 中取值

# 1） 点表示法取值

如果 key 中没有空白字符，那么你可以使用 .key 的方式从 record 中取值。

示例：

```javascript
c = {name: "John Doe", address: "123 Main St.", id: 1123445}
c.name
// Returns John Doe
c.id
// Returns 1123445 
```

# 2） 中括号方式取值

可以使用[" "]的方式取值，当 key中有空白字符的时候，也只能用这种方式来取值。

```txt
c = {"Company Name": "ACME", "Street Address": "123 Main St.", id: 1123445}
c["Company Name"]
// Returns ACME
c["id"]
// Returns 1123445 
```

# 5.3.1.3 嵌套与链式取值

Record类型可以进行嵌套引用。

从嵌套的 Record 中引用值的时候可以采用链式调用的方式。链式调用时，点表示法和中括号还可以混用。

```javascript
customer =
{
    name: "John Doe",
    address: {
    street: "123 Main St.",
    city: "Pleasantville",
    state: "New York"
    }
}

customer.address.street
// Returns 123 Main St.

customer["address"]["city"]
// Returns Pleasantville

customer["address"].state
// Returns New York 
```

# 5.3.1.4 record 的 key 是静态的

record类型变量中的 key是静态的，一旦声明，其中的 key就被定死了。一旦你访问这

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

个 record 中一个没有的 key，就会直接抛出异常。正常的话应该返回 null。

```txt
o = {foo: "bar", baz: 123.4}
o.key
// Error: type error: record is missing label haha
// 错误：类型错误：record 找不到 haha 这个标签
```

# 5.3.1.5 操作 records

# 1） 拓展一个 record

使用 with 操作符可以拓展一个 record，当原始的 record 中有这个 key 时，原先 record的值会被覆盖；如果原先的 record 中没有制定的 key，那么会将旧 record 中的所有元素和with中指定的元素复制到一个新的 record中。

示例： 覆盖原先的值，并添加一个 key 为 pet，value 为"Spot"的元素。

```txt
c = {name: "John Doe", id: 1123445}
{c with name: "Xiao Ming", pet: "Spot"}
// Returns {id: 1123445, name: Xiao Ming, pet: Spot} 
```

![image](assets/ccc5ce9e9183edea791a27da204ebb07fb00cf3dd7ee88a7ad56d5c4c51eff6f.jpg)


# 5.3.1.6 列出一个 record 中所有的 keys

（1）导入 experimental（实验的）包。

（2）使用 expertimental.objectyKeys(o:c)方法来拿到一个 record 的所有 key。

示例：

```typescript
import "experimental"
c = {name: "John Doe", id: 1123445}
experimental.objectKeys(o: c)
// Returns [name, id] 
```

# 5.3.1.7 比较两个 record 是否相等

可以使用双等号= =来判断两个 record 是否相等。如果两个 record 的每个 key，每个更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

key 对应的 value 和类型都相同，那么两个 record 就相等。

示例：

```javascript
{id: 1, msg: "hello"} == {id: 1, msg: "goodbye"}
// Returns false
{foo: 12300.0, bar: 34500.0} == {bar: float(v: "3.45e+04"), foo: float(v: "1.23e+04")]
// Returns true 
```

# 5.3.1.8 将 record 转为字符串

使用 display( )函数可以将 record 转为字符串。

示例：

```javascript
x = {a: 1, b: 2, c: 3}
display(v: x)
// Returns " {a: 1, b: 2, c: 3}" 
```

# 5.3.1.9 嵌套 Record 的意义

注意，嵌套的 Record无法放到 FLUX语言返回的表流中，这个时候会发生类型错误，它会说 Record 类型不能充当某一列的类型。那 FLUX 为什么还支持对 Record 进行嵌套使用呢？

其实这是为了一些网络通讯的功能来服务，在 FLUX 语言中我们有一个 http 库。借助这个函数库，我们可以向外发送 http post 请求，而这种时候我们就有可能要发送嵌套的json。细心的同学可能发现，我们的 record 在语法层面上和 json 语法是统一的，而且FLUX 语言提供了一个 json 函数库，借助这个库中的 encode函数，我们可以轻易地将一个record转为json字符串然后发送出去。

# 5.3.2 Array（数组）

# 5.3.2.1 定义一个 Array

数据是一个由相同类型的值构成的有序序列。

在语法上，数组是用方括号[ ]起来的一堆同类型元素，元素之间用英文逗号( , )分隔，并且类型必须相同。

示例：

```jsonl
["1st", "2nd", "3rd"]
[1.23, 4.56, 7.89]
[10, 25, -15] 
```

# 5.3.2.2 从 Array 中取值

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

可以使用中括号[ ] 加索引的方式从数组中取值，数组索引从0开始。

示例：

```hcl
arr = ["one", "two", "three"]
arr[0]
// Returns one
arr[2]
// Returns two 
```

# 5.3.2.3 遍历一个数组

# 5.3.2.4 检查一个数组中是否包含某元素

示例

使用 contains( )函数可以检查一个数组中是否包含某个元素。

```txt
names = ["John", "Jane", "Joe", "Sam"]
contains(value: "Joe", set: names)
// Returns true 
```

# 5.3.3 Dictionary（字典）

# 5.3.3.1 定义一个字典

字典和记录很像，但是 key-value上的要求有所不同。

一个字典是一堆键值对的集合，其中所有键的类型必须相同，且所有值的的类型必须相同。

在语法上，dictionary需要使用方括号[ ]声明，键的后面跟冒号（:）键值对之间需要使用英文逗号（,）分隔。

示例：

```yaml
[0: "Sun", 1: "Mon", 2: "Tue"]
["red": "#FF0000", "green": "#00FF00", "blue": "#0000FF"]
[1.0: {stable: 12, latest: 12}, 1.1: {stable: 3, latest: 15}] 
```

# 5.3.3.2 引用字典中的值

（1）导入 dict包

（2）使用 dict.get( )并提供下述参数:

a) dict：要取值的字典

b) key：要用到的 key

c) default：默认值，如果对应的 key不存在就返回该值

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网


示例：


```txt
import "dict"
positions =
[
    "Manager": "Jane Doe",
    "Asst. Manager": "Jack Smith",
    "Clerk": "John Doe",
]

dict.get(dict: positions, key: "Manager", default: "Unknown position")
// Returns Jane Doe

dict.get(dict: positions, key: "Teller", default: "Unknown position")
// Returns Unknown position 
```

# 5.3.3.3 从列表创建字典

（1）导入 dict包

（2）使用 dict.fromList( )函数从一个由 records 组成的数组中创建字典。其中，数组中的每个 record 必须是{key:xxx,value:xxx}形式


示例：


```txt
import "dict"  
list = [{key: "k1", value: "v1"}, {key: "k2", value: "v2"}]  
dict.fromList(pairs: list)  
// Returns [k1: v1, k2: v2] 
```

# 5.3.3.4 向字典中插入键值对

（1）导入 dict包

（2）使用 dict.insert( )函数添加一个新的键值对，如果key早就存在，那么就会覆盖这个 key 对应的 value。


示例：


```swift
import "dict"
exampleDict = ["k1": "v1", "k2": "v2"]
dict.insert(dict: exampleDict, key: "k3", value: "v3") // Returns [k1: v1, k2: v2, k3: v3] 
```

# 5.3.3.5 从字典中移除键值对

（1）引入 dict包

（2）使用 dict.remove方法从字典中删除一个键值对

示例：

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```txt
import "dict"
exampleDict = ["k1": "v1", "k2": "v2"]
dict.remove(dict: exampleDict, key: "k2") // Returns [k1: v1] 
```

# 5.3.4 function（函数）

# 5.3.4.1 声明一个函数

一个函数是使用一组参数来执行操作的代码块。函数可以是命名的，也可以是匿名的。在小括号()中声明参数，并使用箭头=>将参数传递到代码块中。

示例：

```txt
square = (n) => n * n
square(n:3)
// Returns 9 
```

FLUX不支持位置参数。调用函数时，必须显示指定参数名称。

# 5.3.4.2 为函数提供默认值

我们可以为某些函数指定默认值，如果为函数指定了默认值，也就意味着在调用这个函数时，有默认值的函数时非必须的。

示例：

```txt
chengfa = (a, b=100) => a* b
chengfa(a:3)
// Returns 300 
```

# 5.4 函数包

Flux 的标准库使用包组织起来的。包将 Flux 的众多函数分门别类，默认情况下加载universe 包，这个包中的函数可以不用 import 直接使用。其他包的函数需要在你的 Flux 脚本中先用import语法导入一下。

示例：

```txt
import "array"
import "math"
import "influxdata/influxdb/sample" 
```

但是，截至目前，虽然你可以自定义函数，但是你无法自定义包。如果希望将自己的自定义函数封装在一个包里以供复用，那就必须从源码上进行修改。

# 第6章 如何使用 FLUX 语言的文档

# 6.1 如何查看函数文档

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

这是 FLUX 语言的文档 https://docs.influxdata.com/flux/v0.x/ ，通常来说我们使用FLUX的文档主要是用它来查看一些函数怎么用，如图所示：

![image](assets/c6823a00a2b2bc7b163daf992bf07aa5a65dcb8e407ed092099aebafd770bd86.jpg)


点击Standard libaray，就可以看到FLUX的所有函数包了。效果如下图所示：

![image](assets/d8e381b459b18d7046086d099fb419bd5c96229f7070ad68960ebda515470d71.jpg)


点击一个包的左侧的+按钮，就可以看到这个包里的所有函数，任意点击其中一个，就可以看到这个函数的详细说明，包括会返回什么，调用的时候需要传递什么参数等等。

![image](assets/2c28559f318379f04528e6540452e08bdc60e37c1c48b639a6bdd2fb41b6deb2.jpg)


再往下拉，你还可以看到每个函数都有很详细的使用示例。代码基本上是可以拿来改改就用的。

![image](assets/1d6825a27b24b1f57e08cbda7ad907d40cff0418db97666e8a0d82f50cc93fd8.jpg)


# 6.2 避免使用实验中的函数

另外，需要额外注意有一个函数库的名字叫 experimental，这个单词是实验的意思，也就是在未来的 FLUX 版本中，这个函数有可能会变，参数名可能也不是很确定，甚至这个

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

函数可能会在未来的某个版本被放弃。

![image](assets/c07edabed4bede40542d415d1aee3c592272e5bdc08deda0d68ceca72bcf6f58.jpg)


如果你有升级的打算，那么 experimental 里面的函数应该敬而远之，否则在未来的某个时间，很有可能会导致重复开发。

# 6.3 查看函数可以在哪些版本中使用

另外需要注意，每个函数的文档标题正下方都会标记这个函数是从哪个 FLUX 版本开始加入的。比如从下图我们就可以知道 request.do()函数是从 0.173 之后才能用的。

requests.do() function 

下面这张图告诉我们 array.concat()函数从0.173版本之后就不能再用了。

array.concat() function 

# 第7章 FLUX 查询 InfluxDB

# 7.1 前言

本章内容强烈建议跟随视频课学习

# 7.2 FLUX 查询 InfluxDB 的语法

使用 FLUX 语言查询 InfluxDB，必须以 from -> range 打头。

例如：

```python
from(bucket: "test_init")
|> range(start: -1h) 
```

range必须紧跟在from后面，不这么写的话会直接报错。

# 7.3 表、表流以及序列

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

我们知道 InfluxDB是使用序列的方式去管理数据的。而 FLUX语言又企图兼容一些关系型数据库的查询，而关系型数据库里的数据结构就是一个有行有列的 table。因此对于FLUX 语言来说，就需要将序列和表统一成一个东西。

所以FLUX引入了表流的概念。

简单来说，FLUX 可以一次性查出多个序列，这个时候一个序列对应一张表，而表流其实就是多张表的集合。同时表流和表的关系其实是全表和子表的关系，子表是全表按照_field，tag_set 和_measurement 进行 group by 之后的结果。在这种情况下，如果调用聚合函数，其实只会在子表中进行聚合。

![image](assets/c0373ccd1203b646887e2ea5fb7eed7ee1b77584a306201997f1bc943c814ad1.jpg)


最后，如果一张表对应的是一个序列了，那么一张表里的一行其实就对应着序列中的一个数据点了。

# 7.4 filter 维度过滤

使用 filert 函数可以对数据按照_measurement、标签集和字段级进行过滤，后面的课程会给大家讲解filter的性能问题。

# 7.5 类型转换函数与下划线字段

Flux 语言中有很多不用指定字段名称的管道函数，比如 toInt()。其实 toInt()这个函数默认要求你的字段中必须要有_value字段，没有_value字段的话也会直接报错。

其实在我们查询出来的数据中，以下划线开头的字段其实代表了一种约定，就是FLUX中有很多函数想要正常运行时要依赖于这些下划线打头的字段的。

所以原则上来说，程序员应该遵守这些约定，不要擅自更改下划线开头的字段。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 7.6 map 函数

map 函数的作用是遍历表流中的每一条数据。

示例：

```txt
import "array"
array.from(rows: [{"name":"tony"},{"name":"jack"}])
|> map(fn: (r)=> {
    return if r["name"] == "tony" then {"_name": "tony 不是 jack"} else {"_name":"jack 不是 tony"}
})
```

这里需要注意，map 函数需要我们传递一个参数 fn，它要求传递一个单个参数输入，且输出必须是record的函数，其中输入数据的类型会是 record。

# 7.7 自定义管道函数

此处，我们定义一个管道函数，它可以将表流中的_value 字段的值乘上 x 倍。请同学们在接下来的示例中注意声明管道函数时所用的语法。

```javascript
big100 = (table=<-, x) => {
    return table
    | > map(fn: (r) => ( {r with "_value": r["_value"] * x})) }
} 
```

接下来我们调用刚才声明的函数，最终整个脚本如下:

```txt
big100 = (table=<-,x) => {
    return table
    |> map(fn: (r) => ( {r with "_value": r["_value"] * x})) 
}
from (bucket: "test_init")
    |> range(start: -1h)
    |> filter(fn: (r) => r["_measurement"] == "go_goroutines")
    |> big100(x:100) 
```

可以自行运行查看函数效果。

这里需要强调的是，管道函数的第一个参数必须写成 table=<-，它表示通过管道符输入进来的表流数据，需要注意，table并不一定写成table但是=<-的格式绝对不能变。

# 7.8 在文档中区分管道函数和普通函数

再次来到函数文档。

![image](assets/f8d47a309e3a72dc2c4fa258c761fa2bde7303657bfd9f0038220b8b3f3d9c6e.jpg)


当我们看到一个函数文档，它会有一个区域叫做 Function type Signature（函数签名），它表示着函数接收哪些参数以及会返回什么。最前面的小括号里的内容就是参数列表，如果参数列表的第一个参数是<-tables: stream[A]，那就表示它是一个可以接收表流输入的管道函数。

反之，如果没有<-tables: stream[A]，那么它就是一个普通函数。

# 7.9 window 和 aggregateWindow 函数

window 函数和 aggregateWindow 函数其实代表着 InfluxDB 中的两种开窗方式，两者不同的地方在于，window 函数会将整个表流重新分组。window 开窗后，是按照序列+窗口的方式对整个表流进行分组。但是 aggregateWindow 函数会保留原来的分组方式，这样一来，使用 aggregateWindow 函数进行开窗后的表流，仍然是按照序列的方式来分组的。

# 7.10 yield 和 join

当 flux 脚本中出现未被赋值给某个变量的表流时，InfluxDB 执行 FLUX 脚本时会自动帮他们在管道的最后面加上|> yield(name: "_result")函数，yield 函数其实是指定了我们当前这个表流是整个 FLUX 脚本最后返回的结果，而且这个结果的名字叫"_result"。当 FLUX脚本中出现多个为赋值给变量的表流时，给多个表流自动补上|>yield(name:"_result")就会出问题了，这是因为当有多个表流后面都有|>yield 时，其实相当于一个 FLUX 脚本会返回多个结果。但是此处要求名称是不能重复的，所以当有多个未赋值的表流时，就必须显示指定 yield(name:"xxx")，而且名称千万不可重复。

但是，在一个 FLUX 脚本里同时返回多个结果集并不是推荐的操作，这通常会让程序的逻辑变的很奇怪，我们之所以能在一个 FLUX脚本里面写多次 from函数，其实是为了方更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

便我们进行join的。

再但是，老师并不建议在 FLUX 脚本中使用 join 操作，这必须要谈到 FLUX脚本的常见使用场景，就是每隔一段时间进行一次查询。如果这个时候，我用一个 from 从 InfluxDB中查询数据，其中有 code=01 等机器编号信息。然后我再用一个 from 去查询 mysql，得到一张机器的属性表。接下来对两张表进行 join，这在逻辑上很合理，但最大的问题就是FLUX脚本无法实现数据的缓存。如果我这个 FLUX脚本是每 15秒执行一次，那就会导致我们需要每 15 秒要去 mysql 上全表扫描一遍机器信息表，效率十分低下。

个人建议仅使用 FLUX 进行简单的查询，然后在应用层的程序里进行 join 操作。因此，本课程并不讲解FLUX语言的 join操作。

# 第8章 前言：如何与 InfluxDB 交互

InfluxDB启动后，会向外提供一套 HTTP API。外部程序可以也仅能通过 HTTP API与InfluxDB进行通信。我们后面要讲到的 influx命令行、Web UI 和各编程语言的客户端库，其内部都是封装的对HTTP API的调用。

![image](assets/a95de57255527ae7a6f2c856968bef667db43d2083b38a406339c662c9273afd.jpg)


所以各种客户端同 InfluxDB 交互时，都离不开 API TOKEN。因为 HTTP 是一种支持官方且简单的协议，这也方便了用户进行二次开发。

# 第9章 InfluxDB HTTP API

InfluxDB 提供了丰富的 API 和客户端库，可以随时和你的应用程序集成。你也可以随时使用 curl 和 ApiPost、Postman 这类程序来测试 API 接口。

本课程会先带大家看一些最常用的 API，然后再告诉大家如何使用 API 文档。但本课程不会对InfluxDB的全部API进行讲解。

# 9.1 准备 token

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

在你想尝试使用 HTTP API与 InfluxDB进行交互时，首先应该用账号和密码登到 WebUI 上选择或创建一个对应的 API TOKEN。课程中，我们使用 tony's Token，这是一个具有全部权限的API Token，实际开发时应谨慎使用，防止 Token被劫持出现安全问题。

![image](assets/cda6c6a7b09ef0eaa991dc8b9924d6e41214b951a1b205a45ffad7e78a8b056f.jpg)


在后面的操作中，你每次发出 HTTP 请求时都需要在请求头上携带 token。

# 9.2 准备接口测试工具

在shell中你可以使用 curl测试接口，不过带图形界面的程序终归是更易用一些。

本课程选用 ApiPost 这一专门的接口测试软件进行演示。ApiPost 是一款国产软件，对标的是 google 的 postman，截至视频课录制时，ApiPost 的最新版本是 6，易用性比上一个版本有大幅提升，用起来很顺手。

# 9.2.1 安装 ApiPost

同学们可以直接访问 ApiPost 的官网下载对应系统的安装包 https://www.apipost.cn/

![image](assets/a75536314efe19861fa0d7f6e55f83da96d644ccd2ea5c66b235bd6eac8224aa.jpg)


后面请同学们自行完成软件的安装。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 9.2.2 准备调试环境

（1）在左侧的目录栏上有一个文件夹 按钮，点一下，创建一个新的目录。

![image](assets/9edf99c6be3e53edaf19c889662cd7fbb18e0e603dc9dc1b91d21c7ff5a11189.jpg)


（2）给目录命名，同时为这个目录添加一个公用 header。这样这个目录下的所有接口都会自动带上这个 header，不需要我们再一个个地手动设置了。我们之前提到过，要想使用 InfluxDB 的 API，请求头上必须要加上 token。所以，我们就把 token 设为公用 header。

![image](assets/42abe49cd174807b4df6e60ca5e236866fb0bc14d81bd1d921439e09adc1bc31.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 9.3 接口的授权

# 9.3.1 Token 授权方式

# 9.3.1.1 成功什么样

现在我们先来看一下授权是否是成功的。

（1）首先，点击左侧的目录名称，右键会弹出一个菜单栏。点击新建接口

![image](assets/814048619a97cb58ae1d1c3efa0df6f22b8f9a0691d811ee208a4d9df18cb69a.jpg)


（2）首先你可以自定义一个接口的名称，然后在接下来的 URL 栏里，填写http://localhost:8086/api/v2/authorizations 点击发送。

![image](assets/a2dc1c647a03ef83bf06a87ea7480cd5814a8a426f50bc8e4d0fafae979ec0b2.jpg)


（3）接下来我们可以看到页面的下方弹出了返回的数据。这个接口返回的数据我们InfluxDB上目前所有的Token信息，包括他们拥有什么权限。

<table><tr><td>美化</td><td>原生</td><td>预览</td><td>断言</td><td>可视化</td><td>回</td><td>Q</td></tr><tr><td>1</td><td colspan="6">{</td></tr><tr><td>2</td><td colspan="6">&quot;authorizations&quot;: &quot;/api/v2/authorizations&quot;,</td></tr><tr><td>3</td><td colspan="6">&quot;backup&quot;: &quot;/api/v2/backup&quot;,</td></tr><tr><td>4</td><td colspan="6">&quot;buckets&quot;: &quot;/api/v2/buckets&quot;,</td></tr><tr><td>5</td><td colspan="6">&quot;checks&quot;: &quot;/api/v2/checks&quot;,</td></tr><tr><td>6</td><td colspan="6">&quot;dashboards&quot;: &quot;/api/v2/dashboards&quot;,</td></tr><tr><td>7</td><td colspan="6">&quot;delete&quot;: &quot;/api/v2/delete&quot;,</td></tr><tr><td>8</td><td colspan="6">&quot;external&quot;: {</td></tr><tr><td>9</td><td colspan="6">&quot;statusFeed&quot;: &quot;https://www.influxdata.com/feed/json&quot;</td></tr><tr><td>10</td><td colspan="6">},</td></tr><tr><td>11</td><td colspan="6">&quot;flags&quot;: &quot;/api/v2/flags&quot;,</td></tr><tr><td>12</td><td colspan="6">&quot;labels&quot;: &quot;/api/v2/labels&quot;,</td></tr><tr><td>13</td><td colspan="6">&quot;me&quot;: &quot;/api/v2/me&quot;,</td></tr><tr><td>14</td><td colspan="6">&quot;notificationEndpoints&quot;: &quot;/api/v2/notificationEndpoints&quot;,</td></tr><tr><td>15</td><td colspan="6">&quot;notificationRules&quot;: &quot;/api/v2/notificationRules&quot;,</td></tr><tr><td>16</td><td colspan="6">&quot;orgs&quot;: &quot;/api/v2/orgs&quot;,</td></tr><tr><td>17</td><td colspan="6">&quot;plugins&quot;: &quot;/api/v2/telegraf/plugins&quot;,</td></tr><tr><td>18</td><td colspan="6">&quot;query&quot;: {</td></tr><tr><td>19</td><td colspan="6">&quot;analyze&quot;: &quot;/api/v2/query/analyze&quot;,</td></tr></table>

成功看到数据，说明我们的 Token 是有效的。s

（4）最后记得点击保存，或者使用 Ctrl+S 快捷键。这样，我们目录下面才会真正留下一个接口。方便你日后访问。

![image](assets/42e674ca1cf700a4b2631ece6bf4ecd7d91b40eac6f359fb7705323173870044.jpg)


# 9.3.1.2 失败什么样

我们也可以看一下授权失败是什么效果。

（1）在目录上点击右键，再点击编辑目录。

![image](assets/c37b00308ba9bb99e94a23e22f2ef8a7d4145216f53fc789f3cc917802cb0a0f.jpg)


（2）将Authotization请求头关掉. 点击右下角的保存

![image](assets/d47bf2fe9eb133e3ed0279fb377e07c7ef77ffef84d96766545d89b40579635c.jpg)


（3）现在回到我们的接口调试页面上，再次点击发送。

![image](assets/2b802e4e8dd890419b5775a5b81986e9fbc26c1c243399edbc55588186342ef7.jpg)


（4）可以看到状态码为 401，而且数据的json告诉我们没有授权。

（5）记得回去将目录的公共请求头放开，继续进行后面的操作。

# 9.3.2 登录授权方式

登录授权其实是留给 Web UI用的，但是你也可以尝试用这种方式获取授权。InfluxDB服务端会判断你的 cookie 是否合法、以及是否过期。符合要求的话就能调用接口实现一系列操作。

进行接下来的操作前，记得关闭目录下的公用请求头。

![image](assets/d6f3623bb3b108b94613e30f1148b9fd30f3c8cb6d5e443d4a802b63af56c8bf.jpg)


# 9.3.2.1 创建登录会话

（1）在 InlfluxDB api v2 目录下创建一个新的接口

![image](assets/f4816fd825e0c4348029629904740617e3be1188abc8a87c876e50bd7067496f.jpg)


（2）给接口自定义一个名称

![image](assets/6dd0d3baab3196a8784965ae3ae680ac10d0b5b21e1bb881439ecc4cfcd10eb6.jpg)


（3）请求的类型选择 POST

（4）填写目标的URL

# （5）配置登录信息

a) 在请求连接的下方点击一下“认证”按钮

b) 在认证方式上选择 Basic auth 认证

c) 在下方的输入框上输入 InfluxDB 的用户名和密码，课程中是 tony, 11111111。

（6）点击发送。

# 9.3.2.2 登录原理

# 1） 什么是 Basic auth 认证

你常见的认证方式可能是将用户名和密码放到 post 请求的请求体中，再发送给服务端进行认证。不过我们刚才并没有在请求体里放用户名和密码，而是配置了一个叫Basic auth认证的东西。这个功能叫做 http 基本认证，是 http 协议的一部分。

基本认证的默认实现是：

（1）把用户名和密码用英文冒号拼起来，也就成了 tony:11111111

（2）再将拼起来的字符串用 Base64 算法编码。（tony:11111111 的 Base64 编码为dG9ueToxMTExMTExMQ==）

（3）给编码结果 dG9ueToxMTExMTExMQ==，添加一个前缀 Basic 所以最后的结果就是。

```txt
Basic dG9ueToxMTExMTExMQ== 
```

（4）把这个字符串放到一个 key 为 authorization 的请求头中，发送给服务端。

# 2） 查看请求头

所以，你可以在页面下方查看这次请求的请求头，如图所示，它就是我们基本认证的结晶。

![image](assets/bf52946928b706fd00df1bdb333553956a85af6aac6e389fd305105c7279b2a9.jpg)


在众多的编程语言中，base64 算法都会作为标准库的一部分存在。你可以在 python 中验证 tony:11111111 的 base64 编码结果。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/fe3fb97c23c3ed53d6ff1da8b6cf30df17daf3d3bf3d1dc06f5cfdf51bc583c7.jpg)


# 3） 查看响应头

我们还可以看一下这次请求的响应头，你可以看到响应头上有一个 key为 set-cookie的键值对。set-cookie 键其实会向浏览器，或者编程语言中的 Session 对象添加一个全局cookie。

![image](assets/c977220056069b86b069c013aba3553b5590fa6c0312c88519db6c4e7ec80829.jpg)


以后每次的请求就会自动携带这个 cookie，以后 InfluxDB 的接口服务就会依据这个cookie来判断请求方是否有权限进行相关操作。ApiPost也有记录 cookie的功能，你可以在ApiPost 的 Cookie 管理器中，查看已经设置的全局 cookie。

# 4） 查看 Cookie 管理器

（1）ApiPost的 Cookie管理器在页面的最上方。

![image](assets/cc07616a5dcfd3282347181ebf7b2148c885a9d9e3990efa5d760071cd98f90f.jpg)


（2）弹出的窗口就是 Cookie 管理器。下面首先会列出你的域名或者 host，老师这里是localhost，点一下，可以看到它下面的全部cookie。

![image](assets/a0b96c69c076a5ae8159ec040d82ec20484183f50c1c28f5d05502bce63f02aa.jpg)


（3）再点一下 influxdb-oss-seesion，就可以看到这个 cookie 的内容可，可以看到它跟刚才响应头的set-cookie内容一模一样。

# 9.3.2.3 验证授权效果

接下来，我们会到之前的列出所有 token 的接口里去，在目录共享请求头关闭的前提下，调用 api。

![image](assets/2bfd8ea32a73fd4a26e88b5dca115078d5813d71b2c06b001e0d21730ea4052e.jpg)


（1）直接点击发送按钮

![image](assets/2521c49f21a303dc4d716a0e32d4bec546a0f7337b1b1399420dc97eeded1c7b.jpg)


（2）响应码为 200，且成功出现了数据，说明我们现在是有权限的，可以点击下面的请求头按钮，看一下这次请的请求头。

![image](assets/11090f5009f150277cd5f703851d5f085348dafea6cbd543b3a6a391577863f9.jpg)


# 9.3.2.4 关闭全局 cookie 再查看效果

接下来我们关闭全局 cookie再查看效果。

（1）在 ApiPost 中打开 Cookie 管理器，按图中操作，关闭 Cookie。

![image](assets/ea13f177a0b6cb3f9393a53fcb31ac64cc57b12ab1b7d63a63efea02ede8d9c8.jpg)


（2）再次向 /api/v2/authorizations 发送请求。可以看到我们这次没权限了

![image](assets/8adfd53a535a8d24f66f6c4ba3b5f5b0ab044b58889fecb47056e45269d3cd32.jpg)


（3）再次查看请求头。这次我们失去了 cookie。

![image](assets/ea04c1041442e110d424ce7b4a0e14d8f637300a549a1fb21a3b81584e5bb2d4.jpg)


# 9.3.2.5 小结

这一节，我们学会了用登录的方式获取授权。但是，大家还要注意两点

⚫ http基本认证默认实现的安全问题

我们前面讲过，http基本认证其实就是把 tony:11111111的 Base64编码放在的请求头上，但是 Base64只是一种数据的编码方式，它不是加密算法也不是信息摘要算法。这也就是说，一旦我的请求被拦截，那对方就能看到我的用户名和密码，对我实施中间人攻击。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

如图所示，Base64编码的字符串也可以被解码为明文。

![image](assets/e201cd835404200c9f84ac733a94312a759290308719c636f542a07258f6e14a.jpg)


所以，从安全角度考虑，不应当在开发时将 Web UI 暴露在公网上。而且集成应用时，授权也千万不可以用登录方式，应该全部使用 token。

⚫ Cookie 有过期时间

当你和别的应用进行集成时，也不应该使用登录的方式，登录授予的 cookie 是有过期时间的，大概半小时，cookie 就会过期。用户必须重新登录拿到新的 cookie 才能和InfluxDB 继续交互。

# 授权的内容就讲到这了。

同学们进行后面操作的时候，记得恢复目录的公用请求头，并关闭 Cookie。这样，我们就还是使用token授权的方式完成后面的一系列操作。

# 9.4 接口安全：配置 HTTPS

HTTP 是一种纯文本通信协议。早期很多互联网协议都是使用明文的方式来传输数据的。这样，最大的问题就是如果我们的网络请求被劫持，那么劫持的一方可以看到我们请求中的所有数据（包括 Token）,这样就算是使用 Token 进行授权比 user:password 安全一些，但泄漏Token也会带来很多麻烦事。所以，InfluxDB官方强烈建议我们开启HTTPS。

# 9.4.1 使用 openssl 生成证书

下面是官方给出的命令模板。

```shell
sudo openssl req -x509 -nodes -newkey rsa:2048 \
-keyout /etc/ssl/influxdb-selfsigned.key \
-out /etc/ssl/influxdb-selfsigned.crt \
-days <NUMBER_OF_DAYS> 
```

自己跑的时候可以参考做一下调整

# 命令解释：

⚫ req -x509，指定生成自签名证书的格式。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ -newkey rsa:2048，生成证书请求或者自签名整数时自动生成密钥，然后生成的密钥名称由 keyout 指定。rsa:2048 意思是产生 rsa 密钥，位数是 2048。

⚫ -keyout，指定生成的密钥名称。

⚫ -out，证书的保存路径

⚫ -days，证书的有效期限，单位是 day（天）,默认是 365 天。

现在，我们执行下面的命令：

```shell
openssl req -x509 -nodes -newkey rsa:2048 \
-keyout /opt/module/influxdb2_linux_amd64/selfsigned.key \
-out /opt/module/influxdb2_linux_amd64/selfsigned.crt \
-days 60 
```

执行这个命令后，会让你输入更多信息。你可以直接全部敲回车，将这些字段留空。不影响生成我们有效的证书文件。

执行完这个命令后，/opt/module/influxdb2_linux_amd/ 目录下会产生两个文件，一个是selfsigned.crt（证书文件）另一个是 selfsigned.key（密钥文件）。而且他们的有效期是 60 天至此，你的密钥文件就成功生成了！

# 9.4.2 确保启动 influxd 的用户对密钥整数文件有读取权限

![image](assets/6fe3aedd30dd18e2a01df51a38841ebbc17da2d5d30062ed55e885b0e67244e3.jpg)


# 9.4.3 启动influxd服务时指定证书和密钥路径

使用influxd命令启动 InfluxDB服务时，记得指定一下整数的密钥的路径。

```txt
./influxd \
--tls-cert="/opt/module/influxdb2_linux_amd64/selfsigned.crt" \
--tls-key="/opt/module/influxdb2_linux_amd64/selfsigned.key" 
```

# 9.4.4 验证 HTTPS 协议是否生效

回到我们的 ApiPost6，再次向 http:/localhost:8086/api/v2/authorizations 发送 GET 请求。

![image](assets/853894ee3891747a7cd7aae5050e30f5651bd9625ad17ab6af0ec28da9983259.jpg)


可以看到，我们使用 http 的协议头再进行访问，响应的状态码为 400，并提示我们向HTTPS 服务器发送了一个 HTTP 请求。现在我们将 URL 前面的 http 改成 https 再试一下。

![image](assets/6bcaf3bb947fae1502a8a9611e5c33206b4099ad695646c1f6eebaff7803c7ec.jpg)


如果你也能达到这个效果，说明 influxd 的 ssl/tls 认证已经开启，服务端和客户端传递的将会是加密数据而非明文数据。

# 9.4.5 记得更改已存在的 telegraf 配置和 Scrapers

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

我们之前在 WebUI中配置过 Telegraf配置和指标的抓取任务，当时我们配的是 http 协议的 URL ，现在也需要全部换成 https。

![image](assets/77edc4accff2fd034432c2707262026b8a2f1f035c9b5be29a6752aa62e7d239.jpg)


# 9.5 其他生产安全考虑

HTPS 是 InfluxDB开发时，最基础也是最该考虑的安全措施，除此之外 InfluxDB 在设计时还为用户考虑了其他的安全措施，这里给大家简单地介绍一下，不再进行操作演示。

# 9.5.1 IP 白名单

可以参考: https://docs.influxdata.com/influxdb/v2.4/security/enable-hardening/

这个IP白名单并不是限制谁可以访问我的。

而是限制，我 InfluxDB 的查询可以访问谁的。因为 FLUX 语言具有发送网络请求的能力，你可以使用 InfluxDB 的相关配置限定，FLUX 脚本可以向哪些地址发送请求。

# 9.5.2 机密管理

这一块的内容可以参考：https://docs.influxdata.com/influxdb/v2.4/security/secrets/

假如，我们的自己的应用程序和 InfluxDB集成，而用到的一段 FLUX脚本刚好需要使用某个第三方服务的用户名和密码（比如查询 mysql）。

比如：

```txt
import "influxdata/influxdb/secrets"
import "sql"

sql.from(
    driverName: "postgres",
    dataSourceName: "postgresql://tony:11111111@localhost",
    query:"SELECT * FROM example-table",
) 
```

应用和 InfluxDB 服务之间走的也是 HTTP 通信，那么写在脚本中的用户名和密码是有可能泄漏的。这个时候，你可以把用户名和密码用键值的方式放到InfluxDB管理起来，

然后，你就可以在脚本里用 key的方式在InfluxDB里获取tony的用户名和密码了。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```txt
import "influxdata/influxdb/secrets"
import "sql"

username = secrets.get(key: "POSTGRES_USERNAME")
password = secrets.get(key: "POSTGRES_PASSWORD")

sql.from(
    driverName: "postgres",
    dataSourceName: "mysql://${username}:${password}@localhost",
    query:"SELECT * FROM example-table",
) 
```

这样，我们 Mysql 的用户名和密码，就没有在网络上泄漏的风险了。

# 9.5.3 token 管理

可以参考：https://docs.influxdata.com/influxdb/v2.4/security/tokens/

我们之前讲过在 Web 上去创建 token。这里再给大家补充一下 Token 的类型。

⚫ 操作者 Token。操作者令牌有跨越组织的管理权限，它对 InfluxDB OSS 2.x 上的所有组织和资源有完全的读写访问权限。某些操作必须需要操作员权限（比如 查看服务器配置）。操作者 Token 是在 InfluxDB 初始化设置的过程中创建的。要想再创建一个操作者Token，就必须使用先有的操作者 Token。

由于操作者 Token对 InfluxDB中所有的组织具有完全的读写访问权限。因此 InfluxDB建议为每个组织创建一个全权限 Token，并用这些 Token 来管理 InfluxDB。这有助于防止组织间不小心误操作对方资源。

⚫ 全权限Token。对单个组织中所有资源的完全读取和写入访问权限

⚫ 读/写Token。对组织中特定的存储桶进行读取和写入。

# 9.5.4 禁用部分开发功能

可以参考：https://docs.influxdata.com/influxdb/v2.4/security/disable-devel/

InfluxDB的 API中，有一部分是为了方便外部系统去监控和观测 InfluxDB的状态和性能的。如果你觉得这部分可能影响安全，那么你可以随时把它们禁了。

比如：

⚫ /metrics，上文给大家演示过，这里面有各种监控 InfluxDB运行的指标

⚫ WebUI，用户的图形界面交互。

⚫ /debug/pprof，这个接口里面是 Go 语言程序的运行时指标，比如堆内存用了多少，有多少线程数等等。

# 9.6 如何使用 API文档

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 9.6.1 查看 API 文档

⚫ 可以直接在浏览器上访问 http(s)://localhost:8086/docs ，这样可以直接看到对应当前InfluxDB 版本对应的 API 文档。

⚫ 另外也可以在 InfluxDB 官网上查看在线文档。

https://docs.influxdata.com/influxdb/v2.4/api/ 

# 9.6.2 测试工具与 OpenAPI

如果你访问的是本地部署的 InfluxDB，那么访问 http://localhost:8086 还能下载相应的OpenAPI 文档。

![image](assets/34ff92ffe06a7e7817f35b44951c5c38e3bee16b4be266f119c47ac3fd28df59.jpg)


访问 http://localhost:8086/doc 页面的顶部有一个 Download 按钮，点一下。浏览器里会说下载了一个json文件。

![image](assets/54dd1382e08477fbf9c3af3642687250124db9f11158d457d051ae94d6706095.jpg)


可以打开看一下，这其实是一个符合 OpenApi3.0 格式的 API 文档定义文件。现在的ApiPost和Postman对这一格式都能自动生成接口测试。此处我们拿 postman作为演示。

# 9.6.3 示例：Postman 快速生成测试项目

# 9.6.3.1 使用 postman 导入 openapi

![image](assets/6fcb3f4e1fb64f1edfd3cbfcec24baf6ff0cbe1fe503abc7cfaa1fd091978123.jpg)


在postman的左侧目录上方，有一个 import按钮，点一下，会弹出一个对话框。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/4ade07418d79ee2ae3872e2b26d1bff2385c4c9c578f26924440f6b00f67d5af.jpg)


可以看到，这里他说支持上传 OpenApi 格式的文件。点击 Upload Files 按钮，选择刚才下载的 swagger(2).json。最后点击右下角的 Import 按钮。

![image](assets/2af40319b394cea1fa4221eb1bdff6e610296a1a75594d03c60bcfa74a345a74.jpg)


# 9.6.3.2 查看导入效果

![image](assets/9f87d823d81c17fe176352a82a46bdf36f6d39aadd4070bb77f9b963ec2e7bb0.jpg)


可以看到，InfluxDB 中的全部 API 已经导入到 postman 中了。但是这里没有文档中的说明性文字了。找回他们的方法是在 postman 的左边找到 draft，点击一下，再点击右方的Documentation。如下图所示：

![image](assets/12fd188e56f42a03c0b41d0ca9916d1b4460d6e7ecc18164458411153c4cc369.jpg)


![image](assets/51bb176aa2aa2559bda68ef989bdfae8dfc3811f2c781bdbde50a5d21dd56d9f.jpg)


现在，你就可以看到既能阅读，又能立刻进行测试的 API 文档了。

# 第10章 使用 influx 命令行工具

从 InfluxDB 2.1 版本之后，influx 命令行和 InfluxDB 数据库服务程序是分开打包的，所以安装InfluxDB并不会附带 influx命令行工具。用户必须单独安装influx命令行工具。

influx 命令行工具包含很多管理 influxDB 的命令。包括存储桶、组织、用户、任务等。

从 2.1 版本之后，安装 InfluxDB 不会附带 influx 命令行工具，现在 influx 工具和InfluxDB在源码上也已经分开维护了，下载时需要注意对上版本。

# 10.1 安装 influx 命令行工具

这次，我们另辟蹊径，不看着官方文档安装了，改从 github上下载安装。

# 10.1.1 如何去找开源项目的发行版

# 10.1.1.1 什么是发行版

这一部分的内容可以详细参考 Gitee 官方文档 https://gitee.com/help/articles/4328#article-header0

所谓发行，就是这个开源项目进行到一定程度，各种特性和功能已经趋于完善和稳定，到了可以出一个阶段性版本的时候了。

通常来说，github 或者 gitee 上放的是一个项目的源码，但是源码需要经过编译之后才能运行的，那么当作者觉得自己的项目，目前开发进度差不多，应该没什么坑的时候，他就可以自己创建一个发行版。这个时候，作者需要自己上传一些附件，比如 v1.0.0 的编译

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

后程序，v1.0.0的文档和源码等。

规范的发行信息里面应该还有比如 changelog（修改记录）这些信息，告诉用户，这个版本相比上个版本，增加了哪些新的功能，又修复了哪些 bug。

# 10.1.1.2 如何去找一个项目的发行版

首先，你可以去访问官网，通常来说一个开源项目通常应该有它自己的官网，在它的官网上，应该可以找到它的历史版本。但是，有些官网就是新版发布了之后就下架旧版的下载资源，比如 InfluxDB 就是这么干的。

另外，通常开源项目都会在 github 或者 gitee 上去维护一个版本的时间线。

打开你关注的开源项目首页，如图所示是InfluxDB的项目首页。

![image](assets/adf6ed4251cedf22e68ed426e34288374a3b53eab94258f973052b65300c350e.jpg)


点击右下角的 Release。

![image](assets/4c57aac5faeb833ae32be9103df185b280d2a74bb1ca062dcbb776d5b0679454.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

可以看到这个框架从盘古开天辟地至今的所有发行版。通常，在一个版本记录的最下方，会有这个版本对应的已编译好的可执行程序和源码。你可以把它下载下来使用。

<table><tr><td rowspan="5"></td><td>OSS BINARY FILES</td><td>SHA256</td></tr><tr><td>influxdb2-2.4.0-darwin-amd64.tar.gz</td><td>156c52b6999134284abae7b110d25416740414c56702c29fd8efe00b6be98682</td></tr><tr><td>influxdb2-2.4.0-linux-arm64.tar.gz</td><td>4fde9265af5ab9a2b8fbb10bb5df42bd809fd24af352cd7eca0ee9ecf4fdb3b9</td></tr><tr><td>influxdb2-2.4.0-windows-amd64.zip</td><td>07769bd83e5f211490e1d3dac3cfc0cb674a25edc860137f5df444856308fa4e</td></tr><tr><td>influxdb2-2.4.0-linux-amd64.tar.gz</td><td>51ddd49a482490752647a4f134c3c838adab64903f2a6ed9c90daff8418b7c58</td></tr><tr><td rowspan="3"></td><td>OSS UBUNTU &amp; DEBIAN PACKAGE FILES</td><td>SHA256</td></tr><tr><td>influxdb2-2.4.0-arm64.deb</td><td>f0579ede760b2b65b327b95a6fb4bcfeab4cb06fa26aa961fb9810c3dafdf620</td></tr><tr><td>influxdb2-2.4.0-amd64.deb</td><td>7de3ccf9672d259a82e79d629397913512045a4dba1e2cb1ef98e8887d2da599</td></tr><tr><td rowspan="3"></td><td>OSS REDHAT &amp; CENTOS PACKAGE FILES</td><td>SHA256</td></tr><tr><td>influxdb2-2.4.0.x86_64.rpm</td><td>02b3e1843fe232c2944e23322be228530bb752d50bdfc22abd5b2201ad12b6fc</td></tr><tr><td>influxdb2-2.4.0.aarch64.rpm</td><td>25b30d342a0ae9e275d25a23070115363ddd6e1412934c28aa9d2085e26833da</td></tr><tr><td rowspan="3"></td><td colspan="2">▼Assets 2</td></tr><tr><td>Source code (zip)</td><td>8 days ago</td></tr><tr><td>Source code (tar.gz)</td><td>8 days ago</td></tr></table>

# 10.1.2 去找 influx 命令行工具的开源项目

大多数时候，你会在 github 上通过搜索项目名称的方式来从查找你关注的项目。但是如果项目本身的热度不高，那它可能不会出现在搜索结果的第一页里。最后你要向后翻好久才能找到你的项目。

当前 InfluxDB 的热度还算行，但是它周围对应的工具热度就不一定高了。这个时候，你可以将目光聚焦于单个公司下的所有项目。

![image](assets/b1a0e5647bd9279d0732c3c429efc5845169f56d11a3b6c6781ad1131ec0bd5f.jpg)



找到 influx-cli 项目，打开之。https://github.com/influxdata/influx-cli


![image](assets/7fd228b442db9f7b2fe75a117a5aa17c2a1fed050f1f758565e68c727e84b725.jpg)


# 10.1.3 下载安装发行版

点击Releases链接，看到最新的版本。

![image](assets/80540ef24403e13f6941cd8d6b0293455eecd02ae0029e649a08000456d9e36d.jpg)


往下看页面，找到 linux-amd64.tar.gz

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

11.051a6aa: Clarity difterence invirtual vs physical dbrps when listing 

<table><tr><td>OSS BINARY FILES</td><td>SHA256</td></tr><tr><td>influxdb2-client-2.4.0-darwin-amd64.tar.gz</td><td>bf36c892cd85f4d16d3b94f204cd90df5feb52ab6fa0dd3b7c831720c58ed87f</td></tr><tr><td>influxdb2-client-2.4.0-linux-arm64.tar.gz</td><td>ccfb6695f82fcd14a891fda47fbbd02b2bc8cd67a50976817b1344e6f5eff843</td></tr><tr><td>influxdb2-client-2.4.0-windows-amd64.zip</td><td>f5575b2bc18641568868e699f2fa8efcf6070d34f8dd68d84709a228e8c0d581</td></tr><tr><td>influxdb2-client-2.4.0-linux-amd64.tar.gz</td><td>[IMAGE]点击下载</td></tr><tr><td>OSS UBUNTU &amp; DEBIAN PACKAGE FILES</td><td>SHA256</td></tr><tr><td>influxdb2-client-2.4.0-arm64.deb</td><td>b5844d371d2659e535d7020ed175ef227de32b6e8be0a1cb7070c1db66390a94</td></tr><tr><td>influxdb2-client-2.4.0-amd64.deb</td><td>f28b86f21bf1deaf359eaa90104bd45c35d8be9f5c4ebe0f7c41a7cbd2b0872f</td></tr><tr><td>OSS REDHAT &amp; CENTOS PACKAGE FILES</td><td>SHA256</td></tr><tr><td>influxdb2-client-2.4.0.x86_64.rpm</td><td>125fc3f00bed1177fd3a34ec32e6b5c3d826d9129f78ab61e8c5eb9fee478743</td></tr><tr><td>influxdb2-client-2.4.0.aarch64.rpm</td><td>18ae12522d9d7f355c934d84c7630954281b400362198b3164aa1d08140dea60</td></tr></table>

Assets 2 

```txt
Source code (zip)
Source code (tar.g) 
```

```txt
8 days ago
8 days ago 
```

下载到 /opt/software/

```txt
五 26 8月 - 18:54 /opt/software
@atguigu ll
总用量 133M
-rw-rw-r-- 1 atguigu atguigu 88M 8月 19 03:49 influxdb2-2.4.0-linux-amd64.tar.gz
-rw-rw-r-- 1 atguigu atguigu 5.8M 8月 19 03:29 influxdb2-client-2.4.0-linux-amd64.tar.gz
-rw-rw-r-- 1 atguigu atguigu 39M 8月 17 04:25 telegraf-1.23.4_linux_amd64.tar.gz
```

解压到 /opt/module/

```txt
tar -zxvf influxdb2-client-2.4.0-linux-amd64.tar.gz -C /opt/module/ 
```

# 10.2 配置 influx-cli

# 10.2.1 创建配置

influx 命令行工具是你每执行一次操作时，调用一次命令。并不是开启一个持续性的会话。而 influx其实底层还是封装的对 InfluxDB 的服务进程的 http 请求。也就是它还是需要配置Token什么的来获取授权。

所以，为了避免以后每次请求的时候都在命令行里面写一遍 token。我们应该先去搞个配置文件。

使用下面的命令可以创 influx命令行的配置。

```shell
./influx config create --config-name influx.conf \
--host-url http://localhost:8086 \
--org atguigu \
--token
ZA8uWTSRFflhKhFvNW4TcZwwvd2NHFW1YIV1cj9Am5iJ4ueHawWh49_jszoKybEym HqgR5mAWg4XMv4tb9TP3w== \
--active 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ 这个命令其实会在~/.influxdbv2/目录下创建一个 configs 文件，这个文件中，就是我们命令行中写的各项配置。如图所示：

```toml
vim configs 111x24
["influx.conf"]
url = "http://localhost:8086"
token = "ZA8uwTSRFflhKhFvNW4TcZwwvd2NHFW1YIVlcj9Am5iJ4ueHawWh49_jszoKybEymHqgR5mAWg4XMv4tb9TP3w==" org = "atguigu"
active = true
# 
```

# 10.2.2 更改配置

如果你中途配置错误了，再使用上文的命令，它会说这个配置已经存在。

```shell
五 26 8月 - 20:01 /opt/module/influxdb2-client-2.4.0-linux-amd64
@atguigu ./influx config create --config-name influx.conf \
--host-url http://localhost:8086 \
--org atguigu \
--token ZA8uWTSRFflhKhFvNW4TcZwwvd2NHFW1YIVlcj9Am5iJ4ueHawWh49_jszoKybEymHqqR5mAWg4XMv4tb9TP3w== \
--active
Error: failed to create config "influx.conf": config "influx.conf" already exists
```

也就是说，在 /home/dengziqi/.influxdbv2/configs 文件中，["name"]配置快不能重复必须全局唯一。

这个时候如果你想调整配置，应该把 create 换成 update。

也就是

```batch
./influx config update --config-name influx.conf xxxxxxxxx 
```

# 10.2.3 在多份配置之间切换

我们现在用下面的命令再创建一个配置，直接复制 influx.conf 中的内容，把名字修改成 influx2.conf

```shell
./influx config create --config-name influx2.conf \
--host-url http://localhost:8086 \
--org atguigu \
--token
ZA8uWTSRFflhKhFvNW4TcZwwvd2NHFW1YIVlcj9Am5iJ4ueHawWh49_jszoKybEym HqgR5mAWg4XMv4tb9TP3w== \
--active 
```

命令成功执行后，再次打开 ~/.influxdbv2/configs 文件。

![image](assets/a1c9bfedb0ea5d6f2437ebdf5039d36bc1dc175dd78cc6793c9b343c36336b68.jpg)


可以看到 configs 中的文件内容变了，多了一个名为["influx2.conf"]的配置块，而且，旧的["influx.conf"]从 active="true"变成了 previous="true"，同时["influx2.conf"]中有一个active="true"的键值对。说明，如果现在使用 influx-cli 执行操作，那会直接使用influx2.conf 配置块中的内容。

你还可以使用下面的命令切换当前正在使用的配置。

```txt
influx config influx.conf 
```

![image](assets/b761636663d69330a64e05d0c699a7d25b80e01992772a327b907cef9edcd506.jpg)


再次查看 ~/.influxdbv2/configs 文件

```txt
vim ~/.influxdbv2/configs 
```

![image](assets/ca92b5579a8034782c20f694879384b768667e5d3e1a4cf2722199304a85c047.jpg)


# 10.2.4 删除一个配置

influx2.conf现在对我们来说是多余的了，现在，我们将它删除掉。

使用下面的命令删除 influx2.conf。

```txt
./influx config remove influx2.conf 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/6d4f5564c85a325bc2be7e2541a72e64923024dd3adb32e4339b5884a03bf0a7.jpg)


执行后，再次查看~/.influxdbv2/config 文件

<table><tr><td># Phi
    vim ~/.influxdbv2/configs 107x24</td></tr><tr><td>[&quot;influx.conf&quot;]
url = &quot;http://localhost:8086&quot;
token = &quot;ZA8uWTSRFflhKhFvNW4TcZwwvd2NHFW1YIVlcj9Am5iJ4ueHawWh49_jszoKybEymHqgR5mAWg4XMv4tb9TP3w==&quot;&quot;
org = &quot;atguigu&quot;
active = true</td></tr><tr><td>#</td></tr><tr><td># [eu-central]</td></tr><tr><td># url = &quot;https://eu-central-1-1.aws.cloud2.influxdata.com&quot;</td></tr><tr><td># token = &quot;XXX&quot;</td></tr><tr><td># org = &quot;&quot;</td></tr><tr><td>#</td></tr><tr><td># [us-central]</td></tr></table>


可以看到，["influx2.conf"]消失了。而且，我们的 influx.conf 自动变成了 active=true。

# 10.3 influx-cli 命令罗列

我们已经知道 influx-cli 背后封装的是对 InfluxDB HTTP API 的请求。那么 influx-cli 有多少功能基本上就取决于它封装了多少命令，本课程不会介绍 influx-cli 的全部功能。通过下表，同学们可以一探influx-cli的功能。

详情可以参考：https://docs.influxdata.com/influxdb/v2.4/reference/cli/influx/

<table><tr><td>命令</td><td>直译</td><td>解释</td></tr><tr><td>apply</td><td>应用</td><td>应用一个 InfluxDB 模板</td></tr><tr><td>auth</td><td>认证</td><td>管理 API Token 的相关</td></tr><tr><td>backup</td><td>备份</td><td>备份数据(只支持 InfluxDB OSS)</td></tr><tr><td>bucket</td><td>桶</td><td>管理存储桶的命令</td></tr><tr><td>bucket-schema</td><td>桶模式</td><td>管理存储桶模式的命令()</td></tr><tr><td>completion</td><td>完成</td><td>生成完成的脚本</td></tr><tr><td>config</td><td>配置</td><td>管理配置文件</td></tr><tr><td>dashboards</td><td>仪表盘</td><td>列出所有的仪表盘</td></tr><tr><td>delete</td><td>删除</td><td>在 InfluxDB 中删除数据点。</td></tr><tr><td>export</td><td>导出</td><td>将 InfluxDB 中的资源导出为模板</td></tr><tr><td>help</td><td>帮助</td><td>查看任何命令的帮助手册</td></tr><tr><td>org</td><td>组织</td><td>组织管理的命令</td></tr><tr><td>ping</td><td>乒</td><td>访问 InfluxDB 的/health api 作为健康状态检查</td></tr><tr><td>query</td><td>查询</td><td>执行一个 FLUX 脚本的查询</td></tr><tr><td>restore</td><td>恢复</td><td>从备份出来的数据恢复(只有 InfluxOSS 支持)</td></tr><tr><td>scripts</td><td>脚本</td><td>管理 InfluxDB 上的交互本(只有 InfluxDB Cloud 支持)</td></tr><tr><td>secret</td><td>机密</td><td>管理机密</td></tr><tr><td>setup</td><td>设置</td><td>命令行版本的初始化操作,首次安装 InfluxDB 时的设置用户名、密码、组织、存储桶等等。</td></tr><tr><td>stacks</td><td>堆栈</td><td>你可以将堆栈理解为一个 InfluxDB 模板的实例。你可以使用 stacks 管理这些实例。</td></tr><tr><td>task</td><td>任务</td><td>任务管理命令</td></tr><tr><td>telegrafs</td><td>telegrafs</td><td>管理 telegraf 配置的命令</td></tr><tr><td>template</td><td>模板</td><td>总结和验证 InfluxDB 模板</td></tr><tr><td>user</td><td>用户</td><td>管理用户相关的命令</td></tr><tr><td>v1</td><td>版本 1</td><td>使用 InfluxDB V1 兼容的 API</td></tr><tr><td>version</td><td>版本</td><td>打印 influx-cli 的版本</td></tr><tr><td>write</td><td>写</td><td>向 InfluxDB 写数据</td></tr></table>

# 第11章 JAVA 操作 InfluxDB

InfluxDB 客户端可以参考：https://github.com/influxdata/influxdb-client-java

# 11.1 创建一个 maven 项目

这里我创建了一个名为 java4influx 的 maven 项目

![image](assets/59a2a666cbfae24c18dfd7c4dabd45e556326ab2007bc4f4877ff168dd33ac13.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 11.2 导入 maven 依赖

在pom.xml里加入如下依赖。

![image](assets/6b59a3ce1ce885aca07a90bd1ecbf9cd1caa716bbe7bd672aea659f95a9fb77c.jpg)


```xml
<dependencies>
    <dependency>
    <groupId>com.influxdb</groupId>
    <artifactId>influxdb-client-java</artifactId>
    <version>6.5.0</version>
    </dependency>
</dependencies> 
```

刷新一下 maven，下载依赖。

# 11.3 创建一个 package

在 src/main/java 下创建一个 package。这里名为 com.atguigu.influxdb.client。

![image](assets/80b25077eaa2885a333513a9390e1f5597a9311ed35bea0e6af8a4036f9e1257.jpg)


最后，项目结构如上图所示。

# 11.4 示例：查看 InfluxDB 健康状态

# 11.4.1 创建 ExampleHealthy 类

如图所示

![image](assets/a839f0cc56b9fdc91e200f887f42626bc709a0d87915545a9cac6932727868b9.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 11.4.2 创建客户端对象

influxdb-client-java 内部其实封装的是各种 HTTP 请求，所以 token，org 什么 HTTPAPI上需要的东西，在创建客户端的时候都需要考虑。

![image](assets/716ce3ceb14d6816ed34a0a103d8dd95f2cdf3d8972d568480a284c9d557fd64.jpg)


从上图可以看到 InfluxDBClientFactory.create 方法其实有多种重载。这是因为不同的接口它需要的权限和操作的范围不同。比如一个读写权限的 token，它只能对某个存储桶进行操作，那么建立连接时就应该指定 bucket，也就是使用下图的重载。

![image](assets/b4679869f7ac7b7f94f1ee76e7e419ff0f4aefe83ae230c5aee48dd9424b464b.jpg)


但如果你用的是操作员 token，希望完成一些创建组织，删除用户的操作，那么就不应该在创建连接时指定存储桶。此时，应该使用下图所示的重载。

![image](assets/95fc44d40e00566115a62cc6512b4ef0afb784afc3b349dda08c93d54fd94b8f.jpg)


不过、检查 InfluxDB 的健康状态不需要任何权限和 token。此时，我们只需指定一个URL，那就可以使用下图所示的重载了。

![image](assets/fc67f9ee34f9e19a514803a2636c9ca84a6a99a20154e2a2dc92b8607b9f0bb0.jpg)


老师这里是在 Ubuntu 上演示，目标 URL 是 http://localhost:8086 。所以最终代码如下。

```javascript
InfluxDBClient influxDBClient =
InfluxDBClientFactory.create("http://localhost:8086"); 
```

<table><tr><td>public class ExampleHealthy {
    public static void main(String[] args) {
        | InfluxDBClient influxDBClient = InfluxDBClientFactory.create(&quot;http://localhost:8086&quot;);
        }
}</td></tr></table>


InfluxDBClient 对象就是我们的客户端对象。

InfluxDBClient 可以返回各种 Api 对象。

如下图所示。这体现了 java 对 InfluxDB HTTP API 的封装。

![image](assets/8b1388393eae267488cae5b51733c569badd0b2bdf5f349037120a4ab8be77f2.jpg)


# 11.4.3 调用 API

一些简单的 api，也可以通过 InfluxDBClient 对象直接调用。比如我们的检查 InfluxDB健康状态，就可以直接调用 InfluxDBClient 对象的 ping 方法。如下所示。

```javascript
System.out.println(influxDBClient.ping()); 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```java
public class ExampleHealthy {
    public static void main(String[] args) {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create("http://localhost:8086");
    System.out.println(influxDBClient.ping());
    }
} 
```

# 11.4.4 运行

ping方法会返回一个布尔值。如果 InfluxDB可以 ping通，那么就会得到 true，否则返回flase，并记录一条失败日志。

![image](assets/f02fa6cb41edeb6eb4c10bf1bf3230b88694b0fdfe9820e9c56a7feca45990c4.jpg)


# 11.4.5 补充

在之前的版本，有一个测试 InfluxDB 是否健康的 API 叫做 health，不过现在这个接口已经被标记为废弃。health 方法返回一个 HealthCheck 对象，相对而言，对这个对象的处理比直接处理布尔值要麻烦很多。在以后的版本，提倡用 ping方法检查健康状态。

```java
public class ExampleHealthy {
    public static void main(String[] args) {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create("http://localhost:8086");
    System.out.println(influxDBClient.health());
    }
} 
```

# 11.5 示例：查询 InfluxDB 中的数据

# 11.5.1 创建一个 JAVA 类

在 com.atguigu.influxdb.client 下创建一个新的 java 类，ExampleQuery。

![image](assets/5915c10c7424e419092562f6b35a4d452bd673f7d8d5e9c540dd51d84bbb263e.jpg)


# 11.5.2 加入一个 main 方法

稍后main方法里面会写我们的查询逻辑。

![image](assets/3d6c2cd6c984a1e1d3529b2d0309a9618cddba3515495e3ceb7c492fbd8f225d.jpg)


# 11.5.3 创建 InfluxDB 客户端对象

这次我们要操作 InfluxDB 中具体存储桶的数据，建立连接时，推荐选择图中的重载方法。

![image](assets/15071b735a5d4a8bb5d56cea336c09b12747075d7916fea58d5ac79e28d84ccd.jpg)


这个方法需要4个参数。

⚫ url，InfluxDB 服务的 URL，在老师这里就是 http://localhost:8086。

⚫ token，授权的 token，而且类型还必须得是 char[ ]。

org，指定要访问的组织。

⚫ bucket，要访问的存储桶。

现在，我们在ExampleQuery下声明4个静态变量。

如下图所示：

<table><tr><td>public class ExampleQuery {</td></tr><tr><td>1usage
private static String org = &quot;atguigu_com&quot;;</td></tr><tr><td>1usage
private static String bucket = &quot;test_init&quot;;</td></tr><tr><td>1usage
private static String url = &quot;http://localhost:8086&quot;;</td></tr><tr><td>1usage
private static char[] token = &quot;ZA8uWTSRFf1hKhFvNW4TcZwwvd2NHFW1YIVlcj9Am5iJ4ueHawWh49_jszoKybEymHqqR5mAWg4XMv4tb9TP3w==&quot;.toCharArray();</td></tr><tr><td>public static void main(String[] args) throws InterruptedException {</td></tr><tr><td>InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);</td></tr><tr><td>}</td></tr></table>


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 11.5.4 获取查询 API 对象

使用 InfluxDBClient 对象点一下，可以看到 InfluxDBClient 其实提供了两种 API。这也是为了兼容性来考虑的。InfluxQLQueryAPI 是 InfluxDB2.x 做的向前兼容。这里我们选择第一个方法，也就是 getQueryApi，这意味着我们使用 v2 api 进行查询。

![image](assets/58631f5658f683a1148bc5ae697c8a05c902c4b73a84a7ee8044c65b8ec2958b.jpg)


# 11.5.5 了解查询 API

概括性地说，QueryApi 对象下有两个方法 query 和 queryRaw。

![image](assets/5da5fa6d0701edef2f4dd6716e3d5474440e9140883795c5702abf6b530f3c3f.jpg)


两个方法都需要传入一个 FLUX 脚本作为查询语句。但主要的不同点在于返回的结果上。

⚫ queryRaw 方法返回 API 中的 CSV 格式数据（String 类型）。

⚫ query方法视图将查询后的结果封装为各种对象（可以自己指定也可以使用 influxdb-client-java 提供的 FluxTable）。

两个方法各有很多不同的实现，其中一大部分是用来制定连接参数的，比如你创建连接对象的时候没有制定org和bucket，那么可以延迟到调用具体api的时候再指定。

# 11.5.6 query

现在，我们先用 query 方法去查询一下 InfluxDB 中的数据，现在我们要查询 test_init存储桶最近2分钟的数据。

代码如下：

```txt
List<FluxTable> query = queryApi.query("from(bucket:\ "test_init\") | > range(start:-2m)"); 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```java
public static void main(String[] args) throws InterruptedException {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    QueryApi queryApi = influxDBClient.getQueryApi();
    List<FluxTable> query = queryApi.query("from(bucket:\test_init\)| > range(start:-2m)");
} 
```

我们的查询结果 List<FluxTable>其实对应了 FLUX 查询语言中的表流概念。

我们可以打印一下 query 变量。

```txt
ExampleQuery
/home/dengziqi/dev_lang/jdk1.8.0_341/bin/java ...
[FluxTable[columns=8, records=12], FluxTable[columns=8, records=12], FluxTab 
```

现在，我们只取第一个 FluxTable，看看里面有什么。

之前我们讲 FLUX 的时候，有讲过表流和 groupKey 之间的关系。

```java
public static void main(String[] args) throws InterruptedException {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    QueryApi queryApi = influxDBClient.getQueryApi();
    List<FluxTable> query = queryApi.query("from(bucket:\ "test_init\") | > range(start:-2m)");
    FluxTable fluxTable = query.get(0);
    fluxTable.get
    m getRecords() List<FluxRecord>
    m getColumns() List<FluxColumn>
    m getClass() Class<? extends FluxTable>
    m getGroupKey() List<FluxColumn>
    Press Ctrl+. to choose the selected (or first) suggestion and insert a dot afterwards Next Tip 
```

现在可以打印一下 groupKey，看看里面有什么东西。

代码如下：

```java
public static void main(String[] args) throws InterruptedException {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    QueryApi queryApi = influxDBClient.getQueryApi();
    List<FluxTable> query = queryApi.query("from(bucket:\"test_init\") | > range(start:-2m)");
    FluxTable fluxTable = query.get(0);
    List<FluxColumn> groupKey = fluxTable.getGroupKey();
    for (FluxColumn fluxColumn : groupKey) {
    System.out.println(fluxColumn);
    }
} 
```

输出结果如下：

```python
/home/dengziqi/dev_lang/jdk1.8.0_341/bin/java ...
FluxColumn[index=2, label='_start', dataType='dateTime:RFC3339', group=true, defaultValue='']
FluxColumn[index=3, label='_stop', dataType='dateTime:RFC3339', group=true, defaultValue='']
FluxColumn[index=6, label='_field', dataType='string', group=true, defaultValue='']
FluxColumn[index=7, label='_measurement', dataType='string', group=true, defaultValue=''] 
```

这其实可以说明我们的整个表流是以_start，_stop，_field，_measurement 4 列为groupKey 分组的结果。

现在，我们可以尝试打印一下数据。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网


代码如下：


```java
public static void main(String[] args) throws InterruptedException {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    QueryApi queryApi = influxDBClient.getQueryApi();
    List<FluxTable> query = queryApi.query("from(bucket:\ "test_init\") | > range(start:-2m)");
    for (FluxTable fluxTable : query) {
    List<FluxRecord> records = fluxTable.getRecords();
    for (FluxRecord record : records) {
    System.out.println(record Values());
    }
} 
```


结果如下：


```txt
ExampleQuery
{result_result, table419, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, time=2022-08-31T18:54:40.563374778Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table419, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, time=2022-08-31T18:54:50.561307985Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table419, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, time_2022-08-31T18:55:00.561984618Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table419, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, 时间=2022-08-31T18:55:10.562899881Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table419, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, 时间_2022-08-31T18:55:20.559854053Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table419, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, 时间 2022-08-31T18:55:30.56898255Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table419, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, 时间 = 2022-08-31T18:55:40.562896254Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table428, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, time=2022-08-31T18:53:50.5613434251Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table428, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, time_2022-08-31T18:54:00.562468432Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table428, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, time 2022-08-31T18:54:10.561674019Z, _value=0.0, field=10000, measurement=storage_writer_d
{result_result, table428, _start=2022-08-31T18:53:46.54296867Z, _stop=2022-08-31T18:55:46.54296867Z, time = 2022-08-31T18:54:20.569728997Z, _value=0.0, field=10000, measurement=storage_writer_d
```

剩下的功能大家可以自己探索。

# 11.5.7 queryRaw

老师在这里创建了一个新类，叫 ExampleQueryRaw。

代码复制的都是 ExampleQuery 的。

唯一不同的地方就是把 queryApi.query 改为了 queryApi.queryRaw。

同时，query 变量的类型也从 List<FluxTable>变成了 String

```java
public class ExampleQueryRaw {
    l usage
    private static String org = "atguigu_com";

    l usage
    private static String bucket = "test_init";

    l usage
    private static String url = "http://localhost:8086";

    l usage
    private static char[] token =
    "ZA8uWTSRFfLhKhFvNW4TcZwwvd2NHFW1YIVlcj9Am5iJ4ueHawWh49_jszoKybEymHqgR5mAWg4XMv4tb9TP3w==".toCharArray();

    public static void main(String[] args) throws InterruptedException {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    QueryApi queryApi = influxDBClient.getQueryApi();
    String| query = queryApi.queryRaw("from(bucket:\"test_init\") |> range(start:-2m)");
    }
} 
```

现在，我们将查询的结果打印一下。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```csv
/home/dengziqi/dev_lang/jdk1.8.0_341/bin/java ...
，result，table，_start，_stop，_time，_value，_field，_measurement
，_result，0,2022-08-31T19:04:52.816677498Z,2022-08-31T19:05:52.816677498Z,2022-08-31T19:05:00.561977614Z,690,counter,boltdb_reads_total
，_result，0,2022-08-31T19:04:52.816677498Z,2022-08-31T19:05:52.816677498Z,2022-08-31T19:05:10.562503598Z,692,counter,boltdb_reads_total
，_result，0,2022-08-31T19:04:52.816677498Z,2022-08-31T19:05:52.816677498Z,2022-08-31T19:05:20.562306067Z,694,counter,boltdb_reads_total
，_result，0,2022-08-31T19:04:52.816677498Z,2022-08-31T19:05:52.816677498Z,2022-08-31T19:05:30.560729138Z,696,counter,boltdb_reads_total
，_result，0,2022-08-31T19:04:52.816677498Z,2022-08-31T19:05:52.816677498Z,2022-08-31T19:05:40.561031696Z,702,counter,boltdb_reads_total
，_result，0,2022-08-31T19:04:52.816677498Z,2022-08-31T19:05:52.816677498Z,2022-08-31T19:05:50.562204648Z,784,counter,boltdb_reads_total
，_result，1,2022-08-31T19:04:52.816677498Z,2022-08-31T19:05:52.816677498Z,2022-08-31T19:05:00.561977614Z,27,counter,boltdb_writes_total
，_result，1,2022-08-31T19:04:52.816677498Z,2022-08-31T19:05:52.816677498Z,2022-08-31T19:05:10.562503598Z,27,counter,boltdb_writes_total
，_result，1,2022-08-31T19:04:52.816677498Z,2022-08-31T19:05:52.816677498Z,2022-08-31T19:05:20.562306067Z,27,counter,boltdb_writes_total
result 1 , 2022 - 08 - 31T 19 : 04 : 52 . 816677498 Z ,  2  8  8  8  8  8  8  8  8  8  8  8  8  8  8  8  8  8  8  8  8 
```

可以看到，我们打印出了CSV格式的数据，这是因为 InfluxDB HTTP API本来在请求体中放的就是 CSV 格式的数据。所以 QueryRaw 方法其实就是返回原始的 CSV。

# 11.6 同步写和异步写的区别

同步写，就是当我调用写入方法时，立刻向 InfluxDB 发起一个请求，将数据传送过去，而且当前线程会一直阻塞等待写入操作完成。

异步写，其实是我调用写入方法的时候，先不执行写入这个操作，而是将数据放入一个缓冲区。当缓冲区满了，我再真正地将数据发送给 InfluxDB，这样相当于实现了一个攒批的效果。

在后面的示例中，我们会向建议先在 InfluxDB 中创建一个名为 example_java 的存储桶，再学习后面的写入示例。

# 11.7 示例：同步写入 InfluxDB

# 11.7.1 创建一个类

这次我们创建一个名为 ExampleWriteSync 的类，org,bucket,url,token 什么的还是复制之前例子中的，但是此处我们把 bucket 改为 example_java。

总体代码如下：

```java
public class ExampleWriteSync {
    private static String org = "atguigu";
    private static String bucket = "example_java";
    private static String url = "http://localhost:8086";
    private static char[] token = "ZA8uWTSRFflhKhFvNW4TcZwwvd2NHFW1YIVlcj9Am5iJ4ueHawWh49_jszoKybEy mHqgR5mAWg4XMv4tb9TP3w==".toCharArray();
    public static void main(String[] args) {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    }
} 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```java
public class ExampleWriteSync {

    lusage
    private static String org = "atguigu_com";

    lusage
    private static String bucket = "example_java";

    lusage
    private static String url = "http://localhost:8086";

    lusage
    private static char[] token = "ZA8uWTSRFfLhKhFvNW4TcZwwvd2NHFW1YIVLcj9Am5iJ4ueHawWh49_jszoKybEymHqqR5mAWg4XMv4tb9TP3w==".toCharArray();

    public static void main(String[] args) throws URIsyntaxException, IOException {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket); 
```

# 11.7.2 获取 API 对象

我们可以看到，InfluxDBClient 上有多种方法获取写操作 API 对象。其 中WriteApiBlocking 是同步写入，WriteApi 是异步写入。

```java
public static void main(String[] args) {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    influxDBClient.wri
}
    m getWriteApiBlocking()    WriteApiBlocking
    m makeWriteApi()    WriteApi
    m getWriteApi()    WriteApi
    m makeWriteApi(WriteOptions writeOptions)    WriteApi
    m getWriteApi(WriteOptions writeOptions)    WriteApi
Press Ctrl+. to choose the selected (or first) suggestion and insert a dot afterwards Next Tip 
```

我们现在先使用 getWriteApiBlocking 方法获取同步写入的 API。

# 11.7.3 有哪些写入方法

```txt
private static Str
1 usage
private static Str
1 usage
private static cha
"ZA8uWTSRF"
public static void
InfluxDBClient
WriteApiBlocks
writeApiBlocking.w 
```

简单归纳，写入API给用户提供了3类方法写入数据。

⚫ writeMeasurement，用户可以写入自己的 POJO 类

⚫ writePoint，influxdb-client-java 提供了一个 Point 类，用户可以将一条条数据封装为一个个 Point，写入 InfluxDB。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ writeRecord，用户可以用符合 InfluxDB 行协议的字符串向 InfluxDB 写入数据。

另外，这三类方法都有与之对应的带 s 后缀的版本，表示可以一次写入多条。

# 11.7.4 通过 Point 对象写入 InfluxDB

# 11.7.4.1 构建 Point 对象

使用下面的代码创建一个point对象。

```txt
Point point = Point.measurement("temperature")
    .addTag("location", "west")
    .addField("value", 55D)
    .time(Instant.now(), WritePrecision.MS); 
```

这是典型的构造器设计模式，measurement 是一个静态方法，它会帮我们 new 一个Point。addTag 和 addField 不再解释。最终的 time，我们通过第二个参数指定写入时间戳的精度。这里是将写入的时间精度确定为了毫秒，如果你传入了一个纳秒时间戳，但精度指明了毫秒，那超出毫秒的部分会被直接截断。

# 11.7.4.2 将 point 写出

使用下面的代码，直接将 point 写到 InfluxDB 中。记得在此之前创建 example_java 存储桶。

```javascript
writeApiBlocking.writePoint(point); 
```

# 11.7.4.3 验证写入结果

执行程序后，在 InfluxDB DataExplorer 查看 example_java 里面有没有新的数据，并将它展示出来。

![image](assets/e8f297ae92d079fb417f0a23b274b7df1fc4aace2c8b6de814fec9aa5fd25c03.jpg)


# 11.7.5 通过行协议写入 InfluxDB

# 11.7.5.1 注释上一个示例的代码

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

现在，我们将前面通过 Point写数据的代码注释掉。

![image](assets/6b81c34ee0e7733c57a959a4d114d41a8c06e1181ed5427bed29628dcf34ec3b.jpg)


# 11.7.5.2 编写代码

在main方法中追下下面的代码

```javascript
writeApiBlocking.writeRecord(WritePrecision.NS, "temperature, location=west value=60.0"); 
```

此处我们在行协议中省略时间戳，让 InfluxDB自动帮我们把时间补上。

# 11.7.5.3 验证写入结果

运行代码后，一样还是去 InfluxDB上查看数据。

![image](assets/8fd3a9ce9c99e5454cea5d2c1e5fb8003dce797c45430d5c97455da7766c29e1.jpg)


如图所示，第二条数据已经成功进入 InfluxDB了。

# 11.7.6 通过 POJO 类写入 InfluxDB

# 11.7.6.1 注释上一个示例的代码

同样，我们还是先注释掉上一次用 InfluxDB行协议写入数据的代码。

```java
public static void main(String[] args) throws URISyntaxException, IOException {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    WriteApiBlocking writeApiBlocking = influxDBClient.getWriteApiBlocking();

    // Point point = Point.measurement("temperature")
    // .addTag("location", "west")
    // .addField("value", 55D)
    // .time(Instant.now(), WritePrecision.MS);

    // writeApiBlocking viewpoint(point);

    // writeApiBlocking.writeRecord(WritePrecision.NS, "temperature, location=west value=60.0");
} 
```

# 11.7.6.2 添加一个静态内部类

```txt
private static class Temperature {
} 
```

# 11.7.6.3 @Measurement 注解

给静态内部类加一个注解。

```bib
@Measurement(name = "temperature")
private static class Temperature {
} 
```

@Measuremet注解必须加到类上，表示这个类对应 InfluxDB中的哪个测量名称。

# 11.7.6.4 添加成员变量

```txt
@Measurement(name = "temperature")
private static class Temperature {
    String location;
    Double value;
    Instant time;
} 
```

# 11.7.6.5 @Column 注解

@Column注解只能用在成员变量上。

@Column有4种实现，如下图所示。

![image](assets/0e9e9e6232d3556536442af99bd3de7114e87d011cb6297680c1483e1be1ed61.jpg)


你可以将一个成员变量指定为 tag、measurement、timestamp 还是 field。

最终代码如下：

```txt
@Measurement(name = "temperature")
private static class Temperature { 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```txt
@Column (tag = true)
String location;

@Column
Double value;

@Column (timestamp = true)
Instant time;
} 
```

# 11.7.6.6 创建一个 Temperature 对象并给其属性赋值

代码如下

```txt
Temperature temperature = new Temperature();
temperature.location = "west";
temperature.value = 40D;
temperature.time = Instant.now(); 
```

# 11.7.6.7 写到 InfluxDB

现在，我们将这个POJO类的对象写到InfluxDB

```javascript
writeApiBlocking.writeMeasurement(WritePrecision.NS, temperature); 
```

最终的代码如下图所示：

```java
public static void main(String[] args) throws URISyntaxException, IOException {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    WriteApiBlocking writeApiBlocking = influxDBClient.getWriteApiBlocking();

    Point point = Point.measurement("temperature")
    .addTag("location", "west")
    .addField("value", 55D)
    .time(Instant.now(), WritePrecision.MS);

    writeApiBlocking BrigPoint(point);

    writeApiBlocking.writeRecord(WritePrecision.NS, "temperature, location=west value=60.0");
    Temperature temperature = new Temperature();
    temperature.location = "west";
    temperature.value = 40D;
    temperature.time = Instant.now();

    writeApiBlocking.writeMeasurement(WritePrecision.NS, temperature);

}

2 usages
@Measurement(name = "temperature")
private static class Temperature {
    1 usage
    @Column(tag = true)
    String location;

    1 usage
    @Column()
    Double value;

    1 usage
    @Column(timestamp = true)
    Instant time;
} 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 11.7.6.8 验证写入效果

运行 main 方法，去 DataExplorer 上查看输出效果。

![image](assets/5556d5bf9caf169ab27d511d9a449026b190fe7d7679791f6ebdcc60e68a2e5c.jpg)


如图所示，数据已经成功进入 InfluxDB。

# 11.8 示例：异步写入 InfluxDB

# 11.8.1 创建一个类

创建一个名为 ExampleWriteAsync 的类，一样还是复用我们之前的 org、bucket、url 和token。并创建 InfluxDBClient 对象，基础代码如下图所示。

<table><tr><td>public class ExampleWriteAsync {</td></tr><tr><td>1 usage
private static String org = &quot;atguigu_com&quot;;</td></tr><tr><td>1 usage
private static String bucket = &quot;example_java&quot;;</td></tr><tr><td>1 usage
private static String url = &quot;http://localhost:8086&quot;;</td></tr><tr><td>1 usage
private static char[] token = &quot;ZA8uWTSRFfLhKhFvNW4TcZwwvd2NHEW1YIVlcj9Am5iJ4ueHawWh49_jszoKybEymHqqR5mAWg4XMv4tb9TP3w==&quot;.toCharArray();</td></tr><tr><td>public static void main(String[] args) { 
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);</td></tr><tr><td>}</td></tr><tr><td>}</td></tr></table>


# 11.8.2 获取 API 对象

可以看到getWriteApi方法已经被标记为弃用了，

![image](assets/422fc8d0a84c00a9dca3c1c5aab24fbb5d171fcf3171e493084cbc96ae7db930.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

现在鼓励使用的是 makeWriteApi方法。实际上，在目前版本，getWriteApi的内部实现已经是直接调用 makeWriteApi 了。

```java
public static void main(String[] args) {
    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    influxDBClient.make
}
    makeWriteApi()    WriteApi
    makeWriteApi(WriteOptions writeOptions)    WriteApi
    Press Ctrl+. to choose the selected (or first) suggestion and insert a dot afterwards Next Tip 
```

# 11.8.3 编写写入代码

和之前的同步写入一样，writeApi 对象也有

writeRecord、writePoint、wirteMeasurement 多种写入方法，这里不再赘述。

这里，我们只用最简单的writeRecord方法插入一条数据，把代码跑通。

```javascript
writeApi.writeRecord(WritePrecision.NS, "temperature, location=north value=60.0"); 
```

# 11.8.4 验证写入结果（写入失败）

此时，我们运行的代码如下图所示。

```java
public class ExampleWriteAsync {

    lusage
    private static String org = "atquigu_com";

    lusage
    private static String bucket = "example_java";

    lusage
    private static String url = "http://localhost:8086";

    lusage
    private static char[] token = "ZA8uWTSRFlhKhFvNW4TcZwwvd2NHFW1YIVlcj9Am5iJ4ueHawWh49_jszoKybEymHggR5mAWg4XMv4tb9TP3w==".toCharArray();

    public static void main(String[] args) {

    InfluxDBClient influxDBClient = InfluxDBClientFactory.create(url, token, org, bucket);
    WriteApi writeApi = influxDBClient.makeWriteApi();

    writeApi.writeRecord(WritePrecision.NS, record: "temperature, location=north value=60.0");
    }
} 
```

在 Web UI 上打开 Data Explorer 查看写入结果。

![image](assets/62e29de2c7677a7ff0f3a13d28f1862d4ed4dfe7948c0a875c9941c8920b0d5c.jpg)


可以看到，这里没有出现我们刚才的数据，这表示我们刚才的写入失败了。但是，我们的 java 程序没有报错。

这是因为 WriteApi 会使用一个守护线程，帮我们管理缓冲区，它会在缓冲区满或者距离上次写出数据过 1 秒时将数据写出去。我们刚才就放了一条数据，缓冲区没满、write 方法调用完程序就立刻退出了，所以后台线程压根就没有做写的操作。

# 11.8.5 修改代码

现在，有两种方式让守护线程执行写的操作。

⚫ 手动触发缓冲区刷写

```javascript
writeApi.flush(); 
```

⚫ 关闭 InfluxDBClient

```txt
influxDBClient.close(); 
```

这里，我们先用第一种。

修改后的代码如下

![image](assets/53162eb3faad24f04af40210796419f64b20031627979b56bd1a6a788e0b8579.jpg)


运行，之后去 Data Explorer 上查看结果。

# 11.8.6 验证写入结果（写入成功）

如果能看到 location Tag 上出现的 north 标签，而且能够查出来一条数据，那么写入操作就成功了！

![image](assets/ebb957b22d8846414903c12892a8c1d8e95b155e4d69d536627a8c39adfad841.jpg)


# 11.8.7 小结：异步写入工作逻辑

⚫ writeApi里有一个缓冲区，这个缓冲区的大小默认是 10000条数据。

⚫ 虽然有缓冲区但是 writeApi 写出数据并不是一次把整个缓冲区都写出去，而是按照批次（默认是1000条）的单位来写。

⚫ 当产生被压或者写入失败时，守护线程会自动重试写入数据。

# 11.8.8 异步写入的配置

异步攒批的操作的守护线程隐式进行的，好在它的行为我们可以进行具体的配置。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/4e98ebe62a844dc31c93f1eb3ba93ebc88ec6e86f03146189e166b21d82181a3.jpg)


influxdb-client-java 为我们提供了一个 WriteOption 对象，调用 makeWriteApi 时可以传入这个对象，通过上图的提示我们可以看到，缓冲区的大小，批的大小，刷写的间隔我们都是可以进行明确指定的。

# 11.8.9 默认配置

```java
31 usages
@ThreadSafe
public final class WriteOptions implements WriteApi.RetryOptions {

    1 usage
    public static final int DEFAULT_BATCH_SIZE = 1000;
    1 usage
    public static final int DEFAULT_FLUSH_INTERVAL = 1000;
    1 usage
    public static final int DEFAULT_JITTER_INTERVAL = 0;
    1 usage
    public static final int DEFAULT_RETRY_INTERVAL = 5000;
    1 usage
    public static final int DEFAULT_MAX_RETRIES = 5;
    1 usage
    public static final int DEFAULT_MAX_RETRY_DELAY = 125_000;
    1 usage
    public static final int DEFAULT_MAX_RETRY_TIME = 180_000;
    1 usage
    public static final int DEFAULT_EXPONENTIAL_BASE = 2;
    1 usage
    public static final int DEFAULT_BUFFER_LIMIT = 10000; 
```

# 11.9 兼容 V1 Api

这里只做简单的介绍。

使用 InfluxDBClientFactory 创建 Client 对象时，调用 createV1 方法。

![image](assets/1a93a67944dc281df90bdee807b1dd42cfdbb846bf8b05bf4c6f747200d70309.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

这个时候你获取的就是兼容 V1 Api 的 Client 对象。

# 第12章 使用 InfluxDB 模板

# 12.1 什么是 InfluxDB 模板

InfluxDB 模板是一份 yaml 风格的配置文件。它包含了一套完整的仪表盘、Telegraf 配置和报警配置。InfluxDB 模板力图保证开箱即用，把 yaml 文件下载下来，往 InfluxDB 里一导，从数据采集一直到数据监控报警就全部为你创建好。

InfluxDB 官方在 github 上收录了一批模板。开发前可以在这里逛一逛，看有没有可以直接拿来用的。

https://github.com/influxdata/community-templates 

# 12.2 示例：使用模板快速部署

在这节示例中，我们会使用社区模板快速创建一套 docker 的监控模板。要完成这个示例，你需要提前掌握docker的相关知识。

# 12.2.1 找到 Docker 模板文档

访问上一节的 https://github.com/influxdata/community-templates 找到 Docker 模板的目录，点进去。

可以看到，有一节的标题是 Quick install里面有详细的配置说明。


Quick install


```txt
InfluxDB UI
In the InfluxDB UI, go to Settings->Templates and enter this URL: https://raw.githubusercontent.com/influxdata/community-templates/master/docker/docker.yml
Influx CLI
If you have your InfluxDB credentials configured in the CLI, you can install this template with:
influx apply -f https://raw.githubusercontent.com/influxdata/community-templates/master/docker/docker.yml 
```


Included resources


```yaml
- 1 Bucket: docker, 7d retention
- Labels: Telegraf Plugin Labels
- 1 Telegraf Configuration
- 1 Dashboard: Docker
- 1 Variable: bucket
- 4 Alerts: Container cpu, mem, disk, non-zero exit
- 1 Notification Endpoint: Http Post
- 1 Notification Rules: Crit Alert 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 12.2.2 安装模板

使用 influx-cli 安装模板。

```txt
influx apply -f
https://raw.githubusercontent.com/influxdata/community-templates/master/docker/docker.yml 
```

命令执行后，会弹出下面的消息，询问你是否使用上面的资源。

<table><tr><td>+</td><td>Dashboards</td><td>Docker</td><td>000000000000000</td><td>elegant-yonath-d4d015</td><td>inputs.system</td><td>09edf888e5a92000</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>000000000000000</td><td>unruffled-benz-d4d007</td><td>inputs.cpu</td><td>09edf888e7a92000</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>000000000000000</td><td>dreamy-heyrovsky-d4d011</td><td>inputs.disk</td><td>09edf888e8292000</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>000000000000000</td><td>goofy-yalow-d4d005</td><td>inputs.diskio</td><td>09edf888e7292000</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>000000000000000</td><td>inspiring-goodall-d4d00b</td><td>inputs.docker</td><td>09edf888e6292000</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>000000000000000</td><td>agreeing-goldberg-d4d013</td><td>inputs.kernel</td><td>09edf888e7e92000</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>000000000000000</td><td>vigilant-chatelet-d4d00f</td><td>inputs.mem</td><td>09edf888e6a92000</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>000000000000000</td><td>adoring-pasteur-d4d00d</td><td>inputs.net</td><td>09edf888e6692000</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>000000000000000</td><td>gallant-sanderson-d4d003</td><td>inputs.processes</td><td>09edf888e6e9200</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>00000000000000</td><td>elastic-morse-d4d01</td><td>inputs.swap</td><td>O9edf888e869200</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>OOOOOOOOOOOOOOOO</td><td>elegant-yonath-d4d15</td><td>inputs.system</td><td>O9edf888e5a920</td></tr><tr><td>+</td><td>telegrafs</td><td>Docker Monitoring</td><td>OOOOOOOOOOOOOOOO</td><td>nice-kilby-d4d11111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111</td><td></td><td></td></tr><tr><td></td><td></td><td></td><td></td><td></td><td></td><td>TOTAL 13</td></tr></table>

这里所指的资源，涉及要在你的 InfluxDB中创建什么名称的 Bucket，创建什么定时任务和报警任务，创建什么仪表盘等等。如果你是在一个正在生产、且存在相似业务的InfluxDB上，那这个列表还是要好好看一看的，避免出现存储桶重名之类的现象。

确定没有问题之后，敲y回车。

<table><tr><td colspan="5">LABEL ASSOCIATIONS</td></tr><tr><td>RESOURCE TYPE</td><td>RESOURCE NAME</td><td>RESOURCE ID</td><td>LABEL NAME</td><td>LABEL ID</td></tr><tr><td>+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+---</td><td>+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+---- +---</td><td>-</td><td>-</td><td>-</td></tr><tr><td>| dashboards |</td><td>Docker | 09ee20c80f3b5000 | inputs.docker | 09edf888e6292000 |</td><td></td><td></td><td></td></tr><tr><td>| dashboards |</td><td>Docker | 09ee20c80f3b5000 | inputs.system | 09edf888e5a92000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.cpu | 09edf888e7a92000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.disk | 09edf888e8292000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.diskio | 09edf888e7292000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.docker | 09edf888e6292000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.kernel | 09edf888e7e92000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.mem | 09edf888e6a92000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.net | 09edf888e6692000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.processes | 09edf888e6e92000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.swap | 09edf888e8692000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | inputs.system | 09edf888e5a92000 |</td><td></td><td></td><td></td></tr><tr><td>| telegrafs |</td><td>Docker Monitoring | 09ee20c80fab6000 | outputs.influxdb_v2 | 09edf888e7692000 |</td><td></td><td></td><td></td></tr><tr><td></td><td></td><td></td><td>TOTAL</td><td>13</td></tr><tr><td colspan="5">Stack ID: 09ee20c80d692000</td></tr></table>

如果接下来展示的内容以Stack ID: xxxx结尾，那说明安装成功！

这里的Stack（栈）概念，其实就是模板的实例。

# 12.2.3 查看安装结果

现在，我们打开InfluxDB的 WebUI，看一下我们模板的导入效果。

下图是模板为我们创建的存储桶，名为 docker。

# 12.2.3.1 存储桶

有一个名为docker的存储桶

![image](assets/7fce8681913711f6f834b2bba415a92d8690821e690f5d774f6d77208bfe863a.jpg)


# 12.2.3.2 telegraf 配置

一个名为 Docker Monitor 的 Telegraf 配置文件，这个配置文件可能需要根据你 Docker的配置进行一些修改

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/bb2ef834b725285adc211e958405f46f22456b67f977ce6b8df2d0a2b62e3350.jpg)


# 12.2.3.3 仪表盘

模板还帮我们创建了一个名为 Docker 仪表盘，只不过我们现在的 Bucket 里面还没有数据，所以这列的图表都还没有显示出来。

![image](assets/bbbe369549c6b766f2b2155fc9905f1f98b101d362299cd12d59c6f9b15381ed.jpg)


![image](assets/6d3192c9918fafb23670818d3119d50295d75b66f0dfcfdc9d7f58c04628a23f.jpg)


# 12.2.3.4 报警规则

模板还帮我们设置了 4个报警规则，根据题目的描述，分别是

⚫ 容器CPU使用率持续 15分钟超过80%

⚫ 容器硬盘使用率超过 80%

⚫ 容器的内存使用率持续 15分钟超过80%

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ 容器没有以0状态（正常结束）退出。

![image](assets/0be3ee49fc4840dc329f2f03399cf2467f997938a9d7bdd9a6319b1613258239.jpg)


# 12.2.4 运行 Telegraf 采集数据

现在我们要使用 Teleggraf 跑 docker 模板里的配置文件。但之前我们在~/bin 目录下写过一个 host_tel.sh 的启停脚本，那个文件会保证全局只有一个 telegraf。所以，为了避免混乱，我们现在要先把之前的 telegraf停掉。

```txt
host_tel.sh stop 
```

接下来，我们编写新的脚本。

还是在~/bin目录下，创建docker_tel.sh文件。键入以下内容。

```shell
#!/bin/bash

export INFLUX_TOKEN=h106QMEj47juNUco-6T-
op1Tzz0IeMh5MhBIDT8vUdv1R3BVeAzMvWGq2DtmJIcyuPwvPmHTLbZLTbnKxz3UK
A==

export INFLUX_HOST=http://localhost:8086/

export INFLUX_ORG=atguigu

/opt/module/telegraf-1.23.4/usr/bin/telegraf --config
http://localhost:8086/api/v2/telegrafs/09edf888eeeb6000 
```

按照 docker 模板的要求，在运行 telegraf 之前，我们需要声明 INFLUX_TOKEN、INFLUX_HOST 和 INFLUX_ORG 三个变量。

然后，我们修改一下 docker_tel.sh的执行权限。

```txt
chmod 755 ./docker_tel.sh 
```

最终，启动 docker_tel.sh

```txt
./docker_tel.sh 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/89ae7bccc36f4dd64516fb7551f0ef0c9d7d7e33fab0740dd2649f2c3255294e.jpg)


如上图所示，telegraf成功启动。

# 12.2.5 查看模板效果

首先，可以在 DataExplorer 中查看 docker 存储桶中有没有数据。

![image](assets/97a621178a00d284725423390202c383dc7dca382cf2a5d873a6c5a75153f491.jpg)


如图所示，数据已经成功进入 InfluxDB。

接下来，我们可以看一下仪表盘的状态，如下图，仪表盘也是成功展示数据的。

![image](assets/6f0a8b357b40ef670abb1a7c8c978141ca6fe046a0e5691b40c4d4605dcdbbeb.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 12.2.6 运行一个 docker 容器

使用下面的命令，运行一个 docker 的入门容器。

```batch
docker run -dp 80:80 docker/getting-started 
```

如果你的主机上没有 docker/getting-started 镜像，那么 docker 回去 dockerhub 上拉取镜像，因为这个镜像在国外，速度可能会很慢，如果拉取失败，请自行百度替换源的方法。

另外，容器运行后，还需要一段时间让 telegraf采集数据。

# 12.2.7 再次查看仪表盘

如下图所示，我们的镜像和容器数都从0变成了1。而且系统的内存使用率变高了。

![image](assets/65c6b2c2e252d50f586d2ccd468ae161d535926b17d191ea811f5732669c613b.jpg)


# 12.2.8 删除 stack（模板实例）

⚫ 在 Web UI 上，点击左侧工具栏的 按钮，再点击上方的 TEMPLATES。可以看到已安装的模板列表，每个末班列表的右边都有一个删除按钮，通过这种方式可以快捷删除。

![image](assets/d98f242aa59ff0458b1f712b6f23cccb7a842823e0de7ad9cc254828f03aae25.jpg)


删除后，stack所涉的所有资源会全部消失。

⚫ 你也可以通过 influx-cli 来删除

```batch
influx stacks remove -o atguigu --stack-id=09ee20c80d692000 
```

influx-cli 的功能很全，你也可以用它来给 stack 重命名，或者查看一个组织下的所有stack 等。

# 12.3 InfluxDB 模板的不足

# 12.3.1 FLUX 兼容性

我们之前看到，很多 InfluxDB 模板里面都会内嵌 FLUX 语言脚本。但是不同的InfluxDB 里面编译进去的 FLUX 语言版本是不一样的。最重要的是，FLUX 语言目前还处在较快的变动期，标准库还未确定。尤其是后面版本的 FLUX 可能会废弃之前版本里的函数和API。这就导致FLUX语言向前兼容性不佳。

下图是InfluxDB版本和FLUX语言版本的对应关系。

# InfluxDB Open Source (OSS)

<table><tr><td>InfluxDB OSS version</td><td>Flux version</td></tr><tr><td>InfluxDB nightly</td><td>0.181.0</td></tr><tr><td>InfluxDB 2.4</td><td>0.179.0</td></tr><tr><td>InfluxDB 2.3</td><td>0.171.0</td></tr><tr><td>InfluxDB 2.2</td><td>0.162.0</td></tr><tr><td>InfluxDB 2.1</td><td>0.139.0</td></tr><tr><td>InfluxDB 2.0</td><td>0.131.0</td></tr><tr><td>InfluxDB 1.8</td><td>0.65.1</td></tr><tr><td>InfluxDB 1.7</td><td>0.50.2</td></tr></table>

比如，InfluxDB 2.4 的 FLUX 版本是 0.179，Influx2.0 就是 0.131.0。4 次迭代就能让FLUX相差 40多个版本。最典型的比如smaple-data这个模板。

![image](assets/4dedb49a5e561a58f1ad899c5cd1aaf0f82d3bb45ddbfb188d1588a2d9b7c33a.jpg)


在 InfluxDB 2.3之后就不能用了。如果你加载这份模板，会提示配置文件的第 11行有问题。

![image](assets/e1ea90359865be2143dadf5635f295da3982c030c24183ea968d343b690343f4.jpg)


这其实就是因为这份模板中的，csv.from(url:xxx)已被废弃。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/25b44109d1c59f7945bf5ab93daf17a4738dd87b1ded6730987883d0f57d1502.jpg)


再加上官方维护的这套模板仓库、出于“年久失修”的状态，缺乏维护。所以、你可能还需要自己手动修改一下模板。

# 12.3.2 生态不如 Grafana

Grafana 是一个专门做监控仪表盘的框架、支持设置监控任务而且支持以多种数据库作为数据源。社区活跃度高于 InfluxDB，所以 Grafana 框架下有更加丰富且好用的模板。Grafana 还支持将 InfluxDB 作为数据源，这样在框架选型上可以使用 InfluxDB+Grafana 的方案。这样，InfluxDB 就只负责读写，Grafana 负责数据展示和报警。

下图是 Grafana 社区提供的模板。可以看到过滤后，支持以 InfluxDB 作为数据源的仪表盘就有1117个。

![image](assets/7dbb367ac1fe2ce8d0437f7d086946e46fe762ec9f951b79a98c9058a605e4f0.jpg)


反观 InfluxDB 收录的模板、活跃度和模板更新上都赶不上 granfana。

![image](assets/fbb2fe295a8e5bc3fbd755b758850050bda97ddbdfc98ec408e9980997e3d264.jpg)


# 第13章 定时任务

# 13.1 什么是定时任务

InfluxDB 任务是一个定时执行的 FLUX 脚本，它先查询数据，然后以某种方式进行修改或聚合，然后将数据写回 InfluxDB或执行其他操作。

# 13.2 示例：将数据转成 json 发给别的应用

# 13.2.1 创建任务的途径

有很多方式可以帮你创建任务，比如 DataExplorer、Notebook、HTTP API 和 influx-cli。不过，此处我们为了还愿定时任务的本来面貌，我们会使用 influx-cli去创建定时任务。

# 13.2.2 本任务的需求

我们的定时任务要实现下面的需求。

（1）每 30s调度一次

（2）查询最近30s的第一条数据

（3）将数据转为 json

（4）将 json 通过 HTTP 发送 SimpleHttpPostServer。

SimpleHttpPostServer 是老师自己用 go 语言写的一个最简单的 HTTP POST 服务，它的功能就是接收一个 POST 请求，然后把请求体重的内容转为字符串打印出来。你可以在本次课程的资料（在尚硅谷微信公众号回复“大数据”获取）中获取 simpleHttpServer的源码和编译后程序。或者访问 github 地址 https://github.com/realdengziqi/simpleHttpPostServer ，下载源码自行编译，或者在发行记录中下载我已编译好的 linux-x64可执行程序。

![image](assets/ab477adecafffb9a4d82d7acf3d55868592b39e1bcc6763ab1aa5bef1b1cd0a8.jpg)


# 13.2.3 启动 simpleHttpPostServer

下载 simpleHttpPostServer 后，cd 到其所在目录，执行 simpleHttpPostServer 程序。效果如下图所示，终端会被阻塞。

![image](assets/307ddca9f42ec8411023441a5c3027b7111c6daf502467c464d539f4c1950fd4.jpg)


# 13.2.4 测试 simpleHttpPostServer 是否正常工作

我们可以使用 curl 来验证 simpleHttpPostServer 是否正常工作。使用下面的命令，向simpleHttpPostServer 发送一个 POST 请求。

```shell
curl -POST http://localhost:8080/-d '{"hello":"world"}' 
```

之后，查看 simpleHttpPostServer 所占的终端，如果终端出现了一条新的数据，那么说明 simpleHttpPostServer 工作正常。

![image](assets/91765e1e850ab0c7d5c516a86a45f8d9ce3e136a1897b62f6078037d5a48ee5a.jpg)


# 13.2.5 在 DataExplorer 中编写 FLUX 脚本

我们首先在DataExplorer中把查询到转为json的这段逻辑写好。

我们要查询的是 test_init 存储桶下的 go_goroutines 测量。这个测量反应的是我们当前InfluxDB 程序中的 goroutines（轻量级线程）数量。

打开 DataExplrer，编写如下代码

```txt
import "json" 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```txt
import "http"

from (bucket: "test_init")
    | > range(start:-30s)
    | > filter(fn: (r) => r["measurement"] == "go_goroutines")
    | > first(column: "_value")
    | > map(
    fn: (r) => {
    status_code = http.post(url:"http://localhost:8080/", data:json.encode(v:r))
    return {r with status_code:status_code}
    }
    ) 
```

```python
import "json"
import "http"

from(bucket: "test_init")
    | > range(start:-30s)
    | > filter(fn: (r) => r["measurement"] == "go_goroutines")
    | > first(column: "_value")
    | > map(
    fn: (r) => {
    status_code = http.post(url:"http://localhost:8080/", data: json.encode(v:r))
    return {r with status_code:status_code}
    }
    ) 
```

# 代码解释：

⚫ from -> range -> filter，指定了数据源并取出了我们想要的序列，其中 range 的 start参数我们写死-30s。

⚫ first 函数，Flux 查询 InfluxDB 返回的数据默认是按照时间从先到后排序的，first 函数配合前面的查询相当于只取了最近30s的第一条数据。

⚫ map 函数，我们在 map 函数里完成数据的发送。这里需要注意，因为 map 函数要求必须返回 record，而且输出的 record 不能和输入的 record 一模一样。所以，结尾 return 的时候，我们使用了 with 语法，给 map 输出的 record 增加了一个字段。也就是我们发出 http请求响应的状态码。在 map 中的匿名函数，我们使用了 http.post 向 http://localhost:8080/发送了一条json格式的数据。

# 13.2.6 运行代码并观察效果

现在，我们点击 SUBMIT按钮，执行这个 FLUX查询脚本，观察 DataExplorer返回的数据和 simpleHttpPostServer 的输出。

# 13.2.6.1 观察 DataExplorer

点击 SUBMIT后，点一下 view Raw Data按钮，我们关注 table格式的数据。可以看到，现在数据中多了 status_code 一列，而且它的值是 200。而且，因为 first( )函数的作用，这

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

次查询，我们只产生了一行数据。

![image](assets/7b26cd07fb991e314449fe1efb9285c13d15a635fca9d9af4ce9b52eb30f7a43.jpg)


# 13.2.6.2 观察 SimpleHttpPostServer

可以看到，我们的 httpServer顺利收到了 JSON。

![image](assets/a31538638ec52dcf6c9c1e3c95109ff7c3b838c7d8ce82280f522d4e170d908f.jpg)


截至目前，说明我们的 FLUX 脚本可以实现需求，现在的问题就是如何将这份脚本设为定时任务。

# 13.2.7 配置定时调度

现在，在我们的查询逻辑前面插入一行。

```txt
option task = { name: "example_task", every: 30s, offset: 0m } 
```

表示，我们做了一个设定，指定了一个名为 example_task的任务。这个任务每隔 30秒执行一次。offset 这里暂时设为 0m，后面我们会专门讲这里的 offset 有什么意义。

```python
import "json"
import "http"

option task = { name: "example_task", every: 30s, offset: 0m }

from (bucket: "test_init")
    | > range(start:-30s)
    | > filter(fn: (r) => r["measurement"] == "go_goroutines")
    | > first(column: "_value")
    | > map(
    fn: (r) => {
    status_code = http.post(url:"http://localhost:8080/", data: json.encode(v:r))
    return {r with status_code:status_code}
    }
    ) 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 13.2.8 使用 influx-cli 创建任务

虽然我们在 DataExplorer 里写了一个 option，但是具体到 option 会不会生效，必须要看 FLUX 脚本再跟 InfluxDB 的那个 HTTP API 进行交互。所以这里点击 SUBMIT 按钮只会再执行一次查询，option task 并不会生效。这里，我们会先用 influx-cli 创建一遍任务。

先 把 Flux 脚 本 复 制 出 来 ， 在 /opt/modules/examples 里 创 建 一 个 文 件 ， 就 叫example_task.flux 吧。将之前写的脚本粘贴进去。

```python
import "json"
import "http"

option task = { name: "example_task", every: 30s, offset: 0m }

from (bucket: "test_init")
    | > range(start:-30s)
    | > filter(fn: (r) => r["measurement"] == "go_goroutines")
    | > first(column: "_value")
    | > map(
    fn: (r) => {
    status_code = http.post(url:"http://localhost:8080/", data: json.encode(v:r))
    return {r with status_code:status_code}
    }
    ) 
```

使用下面的命令，创建 FLUX任务。

```batch
./influx task create --org atguigu -f /opt/module/examples/example_task.flux 
```

可以看到，我们的任务已经成功创建了。

<table><tr><td colspan="8">Mon 5 Sep - 23:26 /opt/module/influxdb2-client-2.4.0-linux-amd64</td></tr><tr><td>Datguigu</td><td colspan="7">./influx task create --org atguigu_com -f /opt/module/examples/example_task.flux</td></tr><tr><td>ID</td><td>Name</td><td>Organization ID</td><td>Organization</td><td>Status</td><td>Every</td><td>Cron</td><td>Script</td></tr><tr><td>D</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>9ef90111ea19000</td><td>example_task</td><td>84f8210c906f0282</td><td>atguigu_com</td><td>active</td><td>30s</td><td></td><td></td></tr></table>

# 13.2.9 在 Web UI 上查看定时任务

点击左侧工具栏的 按钮，查看任务列表。可以看到，任务已经成功创建。

![image](assets/2c82d72c212e26863ce72bafeb7e1f1cd6754d34c8097327cb82b70b556a5013.jpg)


点一下任务的名称，可以进去看任务执行详情。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/8d817838bede64c2ae9597489549198b0c8306f64e57113f11cfbafc20d9241e.jpg)


详情页面上，可以看到任务的调度时间、开始时间、任务耗时等信息。在最上方点击EDIT TASK 按钮，可以看到当时的任务定义。在这里，你也可以直接修改任务的定义。

![image](assets/cb55ae86beb270dfb40ddedb93a91a5c96da72a07c164440e1044b4d55188980.jpg)


# 13.2.10 在接收端查看定时任务效果

数据的接收端就是 simpleHttpPostServer。可以看到，我们的接收端目前就是每隔 30s收到一条json数据。

![image](assets/b53c3494a3ebf83a2952b36fcea6ec27d87fff981934ffe9afffc964bb11401e.jpg)


完成！

# 13.2.11 使用 DataExplorer 创建任务

这一次，我们用DataExplorer来创建任务。现在，我们先将已经存在的任务删除。

![image](assets/62e9310aaea566245f4426a9c424cbf7fa557edcd2fc8ceba1f734ea32b97d65.jpg)


打开 DataExplorer，编辑 FLUX 脚本，将我们之前写的查询脚本粘进去。注意，要删除option一行。如下图所示：

<table><tr><td>1</td><td>import &quot;json&quot;</td></tr><tr><td>2</td><td>import &quot;http&quot;</td></tr><tr><td>3</td><td></td></tr><tr><td>4</td><td></td></tr><tr><td>5</td><td>from(bucket: &quot;test_init&quot;)</td></tr><tr><td>6</td><td>| &gt; range(start:-30s)</td></tr><tr><td>7</td><td>| &gt; filter(fn: (r) =&gt; r[&quot;measurement&quot;] == &quot;go_goroutines&quot;)</td></tr><tr><td>8</td><td>| &gt; first(column: &quot;_value&quot;)</td></tr><tr><td>9</td><td>| &gt; map(</td></tr><tr><td>10</td><td>fn: (r) =&gt; {</td></tr><tr><td>11</td><td>status_code = http.post(url:&quot;http://localhost:8080/&quot;, data: json.encode(v:r))</td></tr><tr><td>12</td><td>return {r with status_code:status_code}</td></tr><tr><td>13</td><td>}</td></tr><tr><td>14</td><td>)</td></tr></table>

完成上面的操作后，点击 DataExplorer页面右上方的 SAVE AS按钮，在弹出的对话框中选择TASK选项卡。

![image](assets/1245bea1e5e87cee01271f263868c9dc14fbe354df921e62a5cb718fa213ae0d.jpg)


⚫ Name，填写为 example_task

⚫ Every，填写 30s

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ Offset 可以空着，这样默认就是 0。此处显示的 20m 是前端渲染效果，和具体的任务执行无关。

⚫ 此处的 OutputBucket 的填写需要注意，本来我们自己编写的脚本并没有指定要把数据回写到 InfluxDB，但是如果从 Web UI 创建定时任务的话，Output Bucket 不能不设，这样会造成回写操作。这也是为什么我们之前不用 Web UI创建任务的原因。

配置好后，点击 SAVE AS TASK。

# 13.2.12 再次查看任务详情（注意Web UI的小动作）

现在，我们再次回到任务列表。可以看到，任务已经成功创建，并且已经正常运行。

![image](assets/b29e69efa47b320b0bb4d6f81f261f7c4bee85feff466ffa6862bbc81a17d5c3.jpg)


点击 EDIT TASK 按钮，查看我们的 FLUX 脚本。你会惊奇的发现，我们的 FLUX 代码居然被修改了，原本不回写数据库的操作被强制加上了一个 to 函数。另外，可以看到我们代码的前面也被加上了一个 option task代码，这说明Web UI的页面点按操作只不过是帮我们完成了手敲代码的几个步骤。

![image](assets/a0a5eb6cd877b841435ff54c8e11b28fc85bdb3792a12042f2fe15049c249d33.jpg)


总的来说，看开发者能否接收代码被隐式修改。如果这种行为无法接受，那么强烈建议用influx-cli的方式去创建任务。

# 13.3 数据迟到问题

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

option中的offset是专门用来帮我们处理迟到问题的。

首先，我们来关注一个迟到的场景，如下图所示。我们的定时任务每次查询最近 30秒的数据。同时，调度的间隔设为每 30 秒执行一次。

![image](assets/f00d7e741529ff87ff0c32f9b8d866ce9f29298b0eb4c7e110e112b3b5c718ad.jpg)


这个时候，由于网络的延迟，本该 1分 20秒入库的数据，1分 32秒的时候才来。但在1分30秒的时候，我们的查询已经执行完了。这个时候我们错过了 1分20秒的数据。

这个时候，如果我们将 offset设为5秒。如下图所示：

![image](assets/7efcab7d4bf87789e74ee75bdf0b6f0b24e172533dcdc148867415da7d22fa9b.jpg)


定时任务的执行时间向后延迟了 5 秒，但是查询的还是原来范围的数据。这个时候定时任务执行的时间是1分35秒。原先的迟到数据就能被我们查询到了。

# 13.4 cron 表达式

其实InfluxDB的定时任务还支持 cron表达式。option的写法如下：

```hcl
option task = {
    // ...
    cron: "0 * * * *",
} 
```

本教程就不做详细演示了。

# 13.4.1 参考资料

crontab 是 linux 上一个可以设置定时执行命令的工具。cron 表达式就是最早在就是在crontab 中使用的一种表示时间间隔的表达式。相关资料可以参考菜鸟教程开源的 cron 教程。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

https://www.runoob.com/linux/linux-comm-crontab.html 

# 13.4.2 cron 辅助开发工具

如果是对 cron 非常熟练，一口气能直接写对那当然是很好的。如果开发时一次写不准可以百度一些在线的cron生成工具作为辅助。

不 过 ， 这 里 我 更 推 荐 gitee 上 的 开 源 cron 生 成 器 项 目 。 比 如 ：https://gitee.com/toktok/easy-cron 这一个基于 node.js 的 corn 生成工具。你可以把代码拉下来自己部署，也可以使用它的在线 demo。

http://www.easysb.cn/open/easy-cron/index.html 

这个工具可以同时支持 5、6、7 字段的 cron 表达式。

![image](assets/63c92225f627b634c5e5e0bd9138b3dd7da63642a0cb01f0a552771f845ed4fe.jpg)


# 13.5 补充：InfluxDB 抓取任务的本质

之前我们在 Web UI 里设置的抓取任务，其背后其实就是定时执行的 FLUX 脚本。只不过 InfluxDB在 API上将他们分开了。

```txt
import "experimental/prometheus"
prometheus.scrape(url: "http://localhost:8086/metrics") 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

在当前的 FLUX 版本中，experimental/prometheus 库为我们提供了采集 prometheus 格式数 据 的 能 力 。 详 细 可 以 参 考 https://docs.influxdata.com/flux/v0.x/prometheus/scrape-prometheus/

# 第14章 InfluxDB 仪表盘

# 14.1 什么是 InfluxDB 仪表盘

前面已经随着课程内容给大家介绍过 InfluxDB的仪表盘功能了。

点击左侧的 按钮，可以进入 InfluxDB 的仪表盘管理页面。可以看到仪表盘的管理页面，如下图所示：

![image](assets/ab02078d77f79ef52ad4b02cc7f4d5a0e25da7ce54f18e534f8589c30be69578.jpg)


我这里打开一个 System 仪表盘，注意，这个仪表盘中的内容依赖我们之前做的示例 2。

![image](assets/40c4eb929156d20eb044cbeab75eea964e2021dbeab6c07b4a4d42ecf871751e.jpg)


这是一个监控主机硬件与网络资源的仪表盘。仪表盘中的每个 Cell 其实都是一个FLUX查询语句，通过执行 FLUX获取数据结果，再使用 UI将它展示为各类图表。在你打开仪表盘的一瞬间，InfluxDB就会执行这些查询。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 14.2 仪表盘控件

# 14.2.1 手动刷新

右上方的 按钮，点击一次可以重新执行一轮仪表盘中的查询。因为通常的 FLUX脚本都是查询距当前一段时间的数据，所以刷新的功能还是比较必要的。

# 14.2.2 开启自动刷新

右上方的 ENABLE AUTO REFRESH 按钮，可以开启仪表盘的自动刷新

![image](assets/44d71f2989c458f6335bc89be7b38d354c15439dd592ad309dc7780f3bf28ffe.jpg)


# 14.2.3 切换显示时区

Local按钮，可以选择将当前的日期时间显示为当前时区还是UTC。

# 14.2.4 设定查询范围

指定查询过去多长一段时间的数据。

![image](assets/acbd6b3c5eb4c61cbc9f800e2affe2c424bfdaf0ced97cb686ff1c890dcf3646.jpg)


# 14.2.5 添加一个 Cell

Cell就是仪表盘中多个的图形的一个图形。添加图形对应的是左上角 ADD CELL按钮。

![image](assets/20ff75aed27f6b6a74bcec064e0733386f407aaa055e02a3057f27a0677298cf.jpg)


# 14.2.6 添加一个 Note

一个 Note 也是仪表盘中的一个模块，支持 Markdown 语法。对应左上角 ADD NOTE更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

按钮。

# 三 ADD NOTE

# 14.2.7 显示变量

如果仪表盘中包含涉及到变量的查询，那么在仪表盘的顶部会出现一个下拉菜单，通过下拉菜单这一指定变量的值，从而操作仪表盘展示响应数据。对应左上角的 ShowVariables。

![image](assets/63cb8fc14a3390f78f5b19562cb4e4e213b808ed7e7a7d7232cec0325bb06bf8.jpg)


# 14.2.8 开启注解

你可以按住 shift 和鼠标左键，在仪表盘的图示上添加参考线。打开个关闭注解会影响参考线的可见性。

![image](assets/7de4ee2f5f196d53e66044d72776097afd5871daf90b83e8a3270851ff4251a2.jpg)


# 14.2.9 全屏和黑夜模式

此功能在左上角的… 按钮，如图所示：

![image](assets/3fcc02989df5a174adec7c126ae67da49237e2a6eb6bb7ff941e68d3b1f8d615.jpg)


# 14.3 示例：制作可交互的动态仪表盘

本示例要对 CPU 使用情况的相关指标制作仪表盘，这依赖于示例 2。请在完成示例 2的基础上完成改示例。

# 14.3.1 需求

用户希望我们的仪表盘上能加入一个下拉菜单以选择查看哪个 CPU的使用情况。要监控的指标是 useage_user，仪表盘上要显示每 1 分钟，CPU 使用率的最大值、最小值和中位数。

# 14.3.2 创建变量

这里先不解释为什么创建变量。

鼠标悬停在左侧的 按钮，在弹出栏上选择Variables。如图所示：

![image](assets/70e22e5e760a3d334314345273fab4b6d403b61a3cd15fd4d2110bc76e788d61.jpg)


点击右上角 CREATE VARIABLES 按钮，选择 New Variable，会弹出一个创建变量的对话框。在右上角的 Type 为 Query 的前提下，在脚本编辑区键入以下内容:

```txt
import "influxdata/influxdb/schema"
schema.tagValues(bucket: "example02", tag:"cpu") 
```

解释：

这个脚本可以查询出 example02 存储桶中的 cpu 标签有哪些标签值。

左上角需要给变量指定一个名称，这里老师输入的是 CPU。

# 14.3.3 创建新的仪表盘

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/8795c01d17155b245f76f451bf7eda1b3103a0848664b30eb3ef0a31c8c5b276.jpg)


回到仪表盘管理页面，点击 CREATE DASHBOARD 按钮，创建一个新的仪表盘。点击左上角的 ADD CELL按钮。

# 14.3.4 创建新的 cell

![image](assets/2308d3d5fb69bcde9403e384be333c0eac603db776765dc7eb1c338e5ab1352e.jpg)


可以看到，又出现了我们熟悉的 DataExplorer。进入后直接切换到 SCRIPT EDITOR。

键入以下内容。

```txt
basedata = from(bucket: "example02")
    | > range(start: v.timeRangeStart, stop: v.timeRangeStop)
    | > filter(fn: (r) => r["measurement"] == "cpu")
    | > filter(fn: (r) => r["field"] == "usage_user")
    | > filter(fn: (r) => r["cpu"] == v.CPU)

basedata
    | > aggregateWindow(every: 1m, fn: median, Guamempty: false)
    | > yield(name: "median")

basedata
    | > aggregateWindow(every: 1m, fn: max, Guamempty: false)
    | > yield(name: "max")

basedata
    | > aggregateWindow(every: 1m, fn: min, Guamempty: false)
    | > yield(name: "min") 
```

点击SUBMIT查看效果。

![image](assets/ccd8f79418bbd023c1725cf10bb14d3849729c439c172131d78c19520c6b5d37.jpg)


# 14.3.5 优化展示效果

默认的可视化类型为 Graph，我们现在将它切换为 Band，表示带有边界的折线图。

![image](assets/d83a57cd2470af381bb0fe2c93e4cabdc9f584c7c68587ff1a249998e5a19406.jpg)


切换图形后，点击CUSTOMIZE，进行自定义设置。

有一栏是 Aggregate Functions，在这里分别指定 Upper Column Name 为 max。MainColumn Name 为 median，Lower Column Name 为 min。

![image](assets/f731311ffec3b6da2ff4e39ec605e50500228870684850dd27e62a18d1600922.jpg)


这就是有边界的折线图的效果。

最后，点击右上角的对号保存。

# 14.3.6 查看效果

可以看到，在仪表盘的顶部出现了一个名为 CPU 的下拉菜单，通过这个下拉菜单，我更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

们可以控制整个仪表盘，但前提是 cell 对应的 FLUX 查询语句引用了我们设置的变量v.CPU。

![image](assets/a5b25d92a6b9650eff60e9919b95d89c74eef66092e79f0e052d62d347dde250.jpg)


使用下拉菜单选择不同的CPU，可以显示对应的数据。

![image](assets/19800b95d449040bce3360223b33f45cdd3b4a725f9ed105fa2f1ae151e4c3e4.jpg)


完成！

# 14.4 示例：更加灵活的变量与仪表盘

# 14.4.1 需求

在上一个示例中，我们可以通名为 CPU 的变量对仪表盘中展示的序列进行动态的调整。

但是上一个示例中的仪表盘还有一个缺陷。如图所示，我们每次只能展示一个序列，但是如果我们想对比两个 CPU 的性能差别呢？这个时候上一个示例做出的仪表盘就不够用了。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/2daa95026faab081363aa7411ec45d90a395cbc1d3130e6145f40ce02dac25a0.jpg)


现在，我们希望仪表盘中能够同时显示两个 CPU 到的工作状况，方便我们在视觉上进行对比。

# 14.4.2 创建变量

（1）在左侧的工具栏点击 Settings->Variables按钮。进入到变量的配置页面

![image](assets/bf0427924fef78b1477d94dbfe9198d474002f32279469b3db4d731b7e51ebff.jpg)


（2）点击页面右上角的 CREATE VARIABLE按钮。Web UI上会弹出一个创建变量的对话窗。

![image](assets/cb522c6fc0daafd2e2e747ad742a1050992055a3fdea614f69c31fd07443fef5.jpg)


（3）在右上角的 Type 下拉菜单中选择 CSV。（上一个示例中我们创建的是 Query 类型，Query 类型的变量可以根据数据的状况进行动态的变化。但是另外的 Map 类型和 CSV类型不行，它们是静态的，如果想让其中的值发生改变，除非再次通过 API 或者 Web UI对其值进行手动的调整）

（4）在左上角给变量起好名字，在演示中，我们将变量名设为 cpuxxx。

![image](assets/2368ecadb0bcf9e9423f4e8ce7831c9efcb1f659cd3bf8173cac0d99628086d9.jpg)


（5）中间的主要区域是用来设置变量的值的，这里可以使用 CSV 格式，但却没有必要非要按照行列的方式来组织这些值。这里的 CSV 格式其实只是要求你用,（英文逗号）来分隔值。其实这个地方也能用换行的方式来分隔值，如图所示，老师用的是换行分隔的方式。

此处，我们将值设为 cpu0、cpu1、cpu2、cpu3 和 cpu1|cpu2。注意！此处的 cpu1|cpu2是正则表达式写法，表示 cpu1或者cpu2。

![image](assets/566e3d6fc6d32ea78a9e13feaee6d7098a7e97f84d37a6f0b30ccbb481f46071.jpg)


（6）左下角会实时显示你当前给变量 cpuxxx 设了几种取值。

![image](assets/5779a93e7e33d54ff71c86bf2da3031e6037b50ace6bd5e7359b6b0f637b61d0.jpg)


⚫ 在右下角有一个 Select A Default 下拉菜单，它可以给我们的变量设置一个默认值。此处可以将默认值设为 cpu0。至此，我们的 cpuxxx 就创建好了。

![image](assets/7fb9e88a2c7b00a8324dcc365060997473f4c225af49f4df93b3a80e9e06ed47.jpg)


# 14.4.3 修改FLUX脚本（添加正则过滤）

首先，如图所示，先点击齿轮按钮，再在弹出的菜单中点击 Configure按钮，就可以修改当前的Cell了。

![image](assets/46629bafa3d0bccf2dfe9afa6feb9c720fa0b77f3f2709d90c4d1e1f63b0cb58.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

现在，我们需要对上一个示例中的查询脚本作一些修改。在 filter 中去添加一个蒸锅cpuxxx 的取值进行正则过滤的方法。

下面是我们的最终脚本，此处我们就不展示数据的最大值和最小值了。红色的部分是我们需要额外注意的。

```javascript
basedata = from(bucket: "example02")
| > range(start: v.timeRangeStart, stop: v.timeRangeStop)
| > filter(fn: (r) => r["measurement"] == "cpu")
| > filter(fn: (r) => r["field"] == "usage_user")
| > filter(fn: (r) =>
regexp.matchRegexping(r:regexp.compile(v:v.cpuxxx),v:r["cpu"]) 
```

# 代码解释：

⚫ regexp.compile(v:v.cpuxxx)：需要注意，我们在 InfluxDB 中设置的变量的类型始终都是字符串类型，所以要进行正则匹配的话必须先把字符串转成正则表达式。regexp 包下的compile函数就是专门用来将字符串转为正则表达式的。

⚫ regexp.matchRegexpString：用来判断字符串能否与正则表达式匹配。如果可以匹配上，那么该函数就会返回 true，如果匹配不上，那么就会返回 false。

⚫ 这样的话，当我们将变量 cpuxxx 的值置为 cpu1|cpu2 时，就可以同时展示出我们想要的两个序列了。

最终，点击右上角的√按钮保存修改后的cell。

# 14.4.4 查看最终效果

回到仪表盘后，可以看到，最上方的变量下拉菜单已经从 cpu 变成了 cpuxxx，这说明仪表盘会自动判断内部的 cell 用到了哪些变量并做出相应的调整。如下图所示，这就是修改后的效果。

![image](assets/2f9e2ed3f7e6e7eea4ad6aa27dc68e29796122b04f809ddf46467baa02bc03ef.jpg)


此时，选择cpu1|cpu2，就可以看到之前的cell里面会出现两条序列了。

![image](assets/cf19384d4e24f1617e10498e5d1ed5a209e12872633163173f448ebfa2a104f2.jpg)



完成！


# 第15章 InfluxDB 服务进程参数（influxd 命令的用法）

# 15.1 influxd 命令罗列

我们的 InfluxDB 下载好后，解压目录下的 influxd 就是我们 InfluxDB 服务进程的启动命令。本课程不会介绍 influxd的全部命令，通过下面的命令列表，大家可以窥探 InfluxDB的一些可配置的能力。


详情可以参考：https://docs.influxdata.com/influxdb/v2.4/reference/cli/influx/


<table><tr><td>命令</td><td>直译</td><td>解释</td></tr><tr><td>downgrade</td><td>降级</td><td>将元数据格式降级以匹配旧的发行版</td></tr><tr><td>help</td><td>帮助</td><td>打印 influxd 命令的帮助信息</td></tr><tr><td>inspect</td><td>检查</td><td>检查磁盘上数据库的数据</td></tr><tr><td>print-config</td><td>打印配置</td><td>(此命令 2.4 已被废弃)打印完整的 influxd 在当前环境的配置信息</td></tr><tr><td>recovery</td><td>恢复</td><td>恢复对 InfluxDB 的操作权限,管理 token、组织和用户</td></tr><tr><td>run</td><td>运行</td><td>运行 influxd 服务(默认)</td></tr><tr><td>upgrade</td><td>升级</td><td>将 InfluxDB 从 1.x 升级到 InfluxDB2.4</td></tr><tr><td>version</td><td>版本</td><td>打印 InfluxDB 的当前版本</td></tr></table>

不一定必须通过 influxd 命令来查看 InfluxDB 的当前配置。你还可以使用 influx-cli 的命令：

```txt
influx server-config 
```

# 15.2 influxd 的两个重要命令

在生产条件下最有可能用到的两个命令就是 inspect 和 recovery。下面，我们对这两个命令做一下详细的介绍。

# 15.2.1 inspect 命令

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

你可以使用下面的命令来查看 inspect这个子命令的帮助信息。


./influxd inspect -h


```txt
Available Commands:
build-tsi Rebuilds the TSI index and (where necessary) the Series File.
delete-tsm Deletes a measurement from a raw tsm file.
dump-tsi Dumps low-level details about tsi1 files.
dump-tsm Dumps low-level details about tsm1 files
dump-wal Dumps TSM data from WAL files
export-index Exports TSI index data
export-lp Export TSM data as line protocol
report-tsi Reports the cardinality of TSI files
report-tsm Run TSM report
verify-seriesfile Verifies the integrity of series files.
verify-tombstone Verify the integrity of tombstone files
verify-tsm Verifies the integrity of TSM files
verify-wal Check for WAL corruption 
```

你会发现 inspect 这个子命令下还有很多子命令。

这里出现的 tsi、tsm、wal 都跟 InfluxDB 底层的存储引擎相关，本课程并不涉及这一部分的内容。这里可以稍微点一下，你可以使用下面的命令查看 InfluxDB 中数据存储的大概情况。

./influd inspect report-tsm 

执行结果如下图所示。

```csv
[atguigu@host1 influxdb2_linux_amd64]$ ./influx inspect report-tsm
DB RP Shard File Series New (est) Min Time Max Time Load Time
67c102a3ce1d8f3c autogen 1 000000012-000000002.tsm 1911 1911 2022-09-16T12:55:49.336368751Z 2022-09-18T23:59:54.534475311Z 467.712μs
9e9cce19f22ae746 autogen 3 000000001-000000001.tsm 1 1 2022-09-17T06:47:14Z 2022-09-17T06:55:15Z 44.768μs
67c102a3ce1d8f3c autogen 73 000000043-000000003.tsm 5036 3489 2022-09-19T08:00:04.532267286Z 2022-09-25T12:38:06.576864759Z 2.053834ms
9e9cce19f22ae746 autogen 74 000000001-000000001.tsm 1 1 2022-09-24T18:58:36.576160376Z 2022-09-24T19:36:06.575722533Z 123.197μs
8a56382f3e2bc62a autogen 165 00000005-000000002.tsm 4 4 2022-09-23T05:46:22.996Z 2022-09-23T15:11:17.291Z 118.081μs
5569Bee282b4b2ca autogen 174 00000001-000000081.tsm 1 1 2022-09-23T13:39:05.929112818Z 2022-09-23T13:39:05.929112818Z 115.116μs
2a16bf3d23fbc7d1 autogen 196 00000006-000000082.tsm 138 138 2022-09-24T09:45:08.010857758Z 2022-09-24T21:26:35.08399875Z 95.51μs
b6a24a9ec38131 autogen 264 00000081-00000888/tsm 289 289 2022-09-25T11:17:18Z 2022-09-25T11:59:55Z 135.488μs
b6a24a9ec38131 autogen 265 000D8CCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCTDCT
b6a24a9ec38131 autogen 266 0ECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECECE
b6a24a9ec38131 autogen 267  ENEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEE
b6a24a9ec38131 autogen 268  ENEEIEEIEEIEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEIEEI
Summary: Files: 12
Time Range: 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ - 2G/ -
Duration: 2H4m5.663631249s
Statistics
Series:
- 2a16bf3d23fb7d1 (est): 138 (2%)
- b6a4a4a9ec38131 (est): 289 (4%)
- 67c1Oa3ce1d8f3c (est): 54O(9%)-
- 9e9cece1f9fzaa746 (est): 2 (o%)
- 8a56382f3e2bc6a (est): 4 (o%)
- 55O9BeeBb4bca (est): 1 (o%)
Total (est): S826 -
Completed in 23,18363im$
[atguigu@hosti influxdb2_linux_amd64]$ 
```

展示出来的信息中包含了 InfluxDB 的数据存储情况，比如当前整个 InfluxDB 有多少序列，每个存储桶中又有多少序列等等。

另外，还有一个比较重要的 export-tsm 命令，它可以将某个存储桶中的数据全部导出为InfluxDB行协议。后面我们会在一个示例中详细演示它的使用。

# 15.2.2 recovery 命令

recovery 是恢复的意思。

可以先用下面的命令查看 recovery这一子命令的帮助信息。

```txt
./influd recovery -h 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

如图所示，influxd recovery 命令的作用主要是用来修复或者重新生成对 InfluxDB 进行操作所需的 operator（操作者） 权限的。

```txt
[atguigu@host1 influxdb2_linux_amd64]$ ./influxd recovery -h
Commands used to recover / regenerate operator access to the DB

Usage:
    influxd recovery [flags]
    influxd recovery [command]

Available Commands:
    auth    On-disk authorization management commands, for recovery
    org    On-disk organization management commands, for recovery
    user    On-disk user management commands, for recovery

Flags:
    -h, --help    help for recovery 
```

recovery 下面还有 3 个子命令，分别是 auth、org 和 user。它们分别与 token、组织和用户有关。

下面主要是讲解 auth子命令的用法，使用下面的命令可以进一步查看 auth子命令的帮助信息。

```txt
./influxd recovery auth -h 
```

返回的结果如下图所示：

```yaml
[atguigu@host1 influxdb2_linux_amd64]$ ./influxd recovery auth -h
On-disk authorization management commands, for recovery

Usage:
    influxd recovery auth [flags]
    influxd recovery auth [command]

Available Commands:
    create-operator Create new operator token for a user
    list List authorizations

Flags:
    -h, --help help for auth 
```

可以看到它有两个子命令。

create-operator：为一个用户创建一个新的操作者 token。

⚫ list：列出当前数据库中的全部 token。

使用下面的命令就可以为 tony 用户再次创建一个 operator-token 了。

```txt
./influxd recovery auth create-operator --username tony --org atguigu 
```

命令执行后，终端会显示如下图所示的内容，可以看到这里创建了一个名为 tony'sRecovery Token 的操作者 token。

```txt
0a149c67972e3000 tony 09fd705e55488000 tony's Recovery Token 2iVsstZTAHe o-rI8re_Mw02F0AFicmgJnC3Ms53y57LtZMD0y0YdKz5aFv2J-u-BpZANU8Ld7MG27aIWBP6KUQ== [read:authorizations write:authorizations read:buckets writ e:buckets read:dashboards write:Dashboards read:orgs write:orgs read:sources write:sources read:tasks write:tasks read:telegrafs write:telegrafs read:users write:users read:variables write:variables read:scrapers write:scrapers read:secrets write:secrets read:labels write:labels read:views write:views read:documents write:documents read:notificationRules write:notificationRules read:notificationEndpoints write:notificationEndpoints read:checks write:checks read:dbrp write:dbrp read:notebooks write:notebooks read:annotations write:annotations read:remotes write:remotes read:replications write:replications] 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 15.3 influxd 常用配置项

influxd 的可用配置项超多，本课程不会全部讲解。

详细可以参考：https://docs.influxdata.com/influxdb/v2.4/reference/config-options/#assets-path

以下是一些常用的参数

⚫ bolt-path：BoltDB 文件的路径。

engine-path：InfluxDB 文件的路径

⚫ sqlit-path：sqlite 的路径，InfluxDB 里面还用到了 sqllite，它里面会存放一些关于任务执行的元数据，

⚫ flux-log-enabled：是否开启日志，默认是 false。

⚫ log-level：日志级别，支持 debug、info、error 等。默认是 info。

# 15.4 如何对 influxd 进行配置

有 3 种方式可以对 influxd 的配置。这里以 http-bind-address 进行操作，为大家演示。

# 15.4.1 命令行参数

进行如下操作前，记得关闭当前正在运行的 influxd。你可以使用下面的命令来杀死当然的 influxd进程。否则，原先的 influxd进程会锁住 BoltDB数据库，别的进程不能访问。

当然你也可以修改BlotDB路径，但是那样太过麻烦。

```txt
ps -ef | grep influxd | grep -v grep | awk '{print $2}' | xargs kill 
```

用户influxd命令启动 InfluxDB时，通过命令行参数来传递一个配置项。

比如：

```batch
./influxd --http-bind-address=:8088 
```

可以尝试访问8088端口，看服务有没有挂到端口上

![image](assets/e19e403fabcbbcb1de841b740ed1e10a0f52162b5f2cf185e05d74595e63cbb5.jpg)


# 15.4.2 环境变量

同样，还是先杀死之前的 influxd进程。

运行下面的命令。

```txt
ps -ef | grep influxd | grep -v grep | awk '{print $2}' | xargs kill 
```

用户可以声明一个环境变量，对influxd进行配置

比如：

```typescript
export INFLUXD_HTTP_BIND_ADDRESS=:8089 
```

现在，我们启动一下 influxd看下效果。

![image](assets/693625ce095a8dea7bfe7497dd813f4f9e7c0081af23f0539f9795c2c9ef322e.jpg)


最后，因为我们用的是 export 命令，临时搞了一个环境变量，如果你觉得当前 shell 会话不重要，可以关闭当前 shell会话。否则，你可以使用 unset命令来销毁这个环境变量。

```batch
unset INFLUXD HTTP BIND ADDRESS 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

# 15.4.3 配置文件

你还 nfluxd 所在的目录下放一个 config 文件，它可以是 config.json，config.toml，config.yaml。这 3 种格式 influxd 都能识别，不过文件中的内容一定要合法。influxd 启动时会自动检测这个文件。

在 InfluxDB 的安装目录下创建一个 config.json 文件。

```txt
vim /opt/module/influxdb2_linux_amd64/config.json 
```

编辑如下内容。

```json
{
    "http-bind-address": ":9090"
} 
```

启动之前记得停掉之前的 InfluxDB进程。

```txt
ps -ef | grep influxd | grep -v grep | awk '{print $2}' | xargs kill 
```

现在再启动一下，看看效果。

```txt
./influxd 
```

可以看到端口已经变成 9090。配置同样是生效的。

![image](assets/76fe7589251067e8da1e8ff611af14fcfa5984086918937048ff7750bf9268d0.jpg)


# 15.4.4 小结

最后，如果要做配置的修改，建议一定要参考 InfluxDB 的官方文档，这一部分写的非常清楚，而且官网已经给出了进行配置的各种模板。用好官方文档，可以大大提高开发效率.

![image](assets/48bc25a4b7ce6cd901e2b33b96148bc5f8593527297e0195d39cb4aa9fc64d32.jpg)


# 第16章 时序数据库是怎么存储用户名和密码的

InfluxDB内部自带了一个用 Go语言写的 BlotDB，BlotDB是一个键值数据库，它的功能比较有限，基本上就是专注于存值、读值。同时，因为功能有限，它也可以做的很小很轻量。

InfluxDB 就是把用户名、密码、token什么的信息存在这样的键值数据库里的。默认情况下，BlotDB 的数据会存储在一个单独的文件中，这个文件会在~/.influxdbv2/ 路径下，名称为 influxd.bolt。

这个文件的路径可以在 influxd通过bolt-path配置项来进行修改。

# 第17章 从 InfluxDB OSS 迁移数据

# 17.1 将 InfluxDB 中的数据导出

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

导出 InfluxDB 数据必须使用 influxd 命令（注意，不是 influx 命令）。

在InfluxDB2.x中，数据导出是以存储桶为单位的。

下面是示例命令：

```shell
influxd inspect export-lp \
--bucket-id 12ab34cd56ef \
--engine-path ~/.influxdbv2/engine \
--output-path path/to/export.lp \
--start 2022-01-01T00:00:00Z \
--end 2022-01-31T23:59:59Z \
--compress 
```

# 参数讲解：

⚫ influxd inspect，influxd 是可以操作 InfluxDB 服务进程的命令行工具，inspect 是influxd 命令的子命令，使用 inspect 可以

⚫ export-lp，是 export xxx to line protocol 的缩写，表示将数据导出为行协议。它是inspect 的子命令。

⚫ bucket-id，inspect 的必须参数。存储桶的 id

⚫ engine-path，inspect 的必须参数，不过有默认值~/.influxdbv2/engine。所以如果你的数据目录是~/.influxdbv2/engine 那么不指定这个参数也行。

output-path，inspect 的必须参数，指定输出文件的位置。

⚫ start，非必须，导出数据的开始时间

⚫ end，非必须，导出数据的结束时间。

⚫ compress，建议启用，如果启用了，那么 influxd 会使用 gzip 的方式压缩输出的数据。

# 17.2 示例：将 InfluxDB 中的数据导出

这次，我们尝试导出 test_init 的数据导出，截至目前，这个 bucket 里面的数据应该是当前最多的。

（1）首先，你可以使用 influx-cli也可以使用Web UI 来查看我们想要导出的 bucket对应 的 ID。 这 里 ， 课 程 选 择 使 用 Web UI， 可 以 看 到 test_init 存 储 桶 的 ID 为0a2e821ccd12854a。

```txt
test_init 
```

```txt
Retention: Forever ID: 0a2e821ccd12854a 
```

![image](assets/4fcde65e2efbbe516c7d7b136fc085320de9e6179041ae8eefad6933281d63f2.jpg)


Adda label 

（2）于是，我们运行下面的命令，尝试把数据导出。

```shell
./influxd inspect export-lp \
--bucket-id 0a2e821ccd12854a \
--output-path ./oh.lp 
```

这条命令会把 test_init 存储桶里的数据以 InfluxDB 行协议的格式导出到当前目录下的oh.lp 文件中。

正常情况下，程序会输出一系列读写信息。

```jsonl
二 30 8月 - 18:57 /opt/module/influxdb2_linux_amd64
@atguigu ./influxd inspect export-lp --bucket-id 0a2e821ccd12854a \
--output-path ./oh.lp
{"level":"info","ts":1661857044.301934,"caller":"export_lp/export_lp.go:219","msg":"exporting TSM files","tsm_dir":"/home/dengziqi/.influxdbv2/engine/data/0a2e821ccd12854a","file_count":3}
{"level":"info","ts":1661857047.3002422,"caller":"export_lp/export_lp.go:315","msg":"exporting WAL files","wal_dir":"/home/dengziqi/.influxdbv2/engine/wal/0a2e821ccd12854a","file_count":1}
{"level":"info","ts":1661857047.9432976,"caller":"export_lp/export_lp.go:204","msg":"export complete"}
```

（3）使用下面的命令查看当前路径下的文件及其大小。

```txt
ls -lh 
```

ls的h参数，可以将文件的字节数打印为更容易阅读的 MB、GB单位。

```txt
二 30 8月 - 18:57 /opt/module/influxdb2_linux_amd64
@atguigu ls -lh
总用量 1.6G
-rwxr-xr-x 1 dengziqi dengziqi 144M 8月 19 03:44 influxd
-rw-rw-r-- 1 dengziqi dengziqi 1.1K 8月 19 03:44 LICENSE
-rw-rw-r-- 1 dengziqi dengziqi 1.5G 8月 30 18:57 oh.lp
-rw-rw-r-- 1 dengziqi dengziqi 9.6K 8月 19 03:44 README.md
-rw-rw-r-- 1 dengziqi dengziqi 1.3K 8月 26 12:24 selfsigned.crt
-rw---- 1 dengziqi dengziqi 1.7K 8月 26 12:24 selfsigned.key
```

可以看到，我们导出的数据文件 oh.lp有1.5G大小。

（4）现在，我们使用 tail命令来查看一下文件的内容。

```batch
tail -15 ./oh.lp 
```

命令输出的是文件的最后15行内容，可以看到里面全是InfluxDB行协议的数据。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/957b3d4ee1e7ef2d84e85212cb3d97d86a2ca42748c7cd465c37481526054de5.jpg)


不过，我们要注意到 InfluxDB 行协议的一个特点，其实对于整个文件来说，多条数据的 measurement其实是重复的，tagset的重复率也不低，filed的变化也不会很大。这种高度重复的数据其实是非常适合压缩算法的。

# 17.3 示例：导出数据时压缩

（1）现在，我们重新运行数据导出的命令，这次在命令的最后加上--compress 参数。

```shell
./influxd inspect export-lp \
--bucket-id 0a2e821ccd12854a \
--output-path ./oh.lp
--compress 
```

不必担心目录下已经存在oh.lp文件，程序会直接将其覆盖的。

（2）使用ls命令再次查看文件大小。


ls -lh


![image](assets/edf64fe3dfbb30e22fa5cd18eab714470d6f1fdfd1c04835b0d635f305d68358.jpg)


可以看到文件从之前的 1.5G变成了现在的91M，压缩率非常高。

# 第18章 FLUX查询优化

# 18.1 使用谓词下推的查询

谓词下推常见于SQL查询中，一个SQL中的谓词，通常指的是 where条件。

我们看一个最简单的 SQL 语句。它从一个名为 A 的表中查询数据，并按照 n>10 的条件对数据进行过滤。

```sql
select *
from A 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

where n > 10 

你可以想象一下这条 SQL语句在计算机中的执行流程。大体上有下面两种方式。


没有谓词下推的情况


![image](assets/aff8857a82dd50ab56f1699e10124882cda3140d387250760a766c98bd72a8fb.jpg)



有谓词下推的情况


![image](assets/c6a1ca478d0ef261f07ae789aad73aa65788fb1c0cb9011a3951d457ef8b04e7.jpg)


⚫ 一种是将磁盘里的数据全部读到内存中，再在内存中进行过滤。这种方式我们通常说它没有内存下推

⚫ 另一种是在查询时，就只从磁盘取自己需要的数据到内存中，再进行下一步的操作。通常，我们说这种方式实现了谓词下推。

虽然说 FLUX 语言表面上是一个脚本语言，但在查询这件事上，它并不是老老实实一行行执行的，而是有了优化器的参与。FLUX 语言在执行时，会尽可能实现谓词下推的优化，什么样的查询可以实现谓词下推，可以参考官网文档的优化查询一节https://docs.influxdata.com/influxdb/v2.4/query-data/optimize-queries/

Pushdown functions and function combinations 

Most pushdowns are supported when querying an InfluxDB 2.4 or in the following table,a handful of pushdowns are not supported i 

<table><tr><td>Functions</td><td>InfluxDB 2.4</td><td>InfluxDB Cloud</td></tr><tr><td>count()</td><td>√</td><td>√</td></tr><tr><td>duplicate()</td><td>√</td><td>√</td></tr><tr><td>filter() *</td><td>√</td><td>√</td></tr><tr><td>fill()</td><td>√</td><td>√</td></tr><tr><td>first()</td><td>√</td><td>√</td></tr><tr><td>last()</td><td>√</td><td>√</td></tr><tr><td>max()</td><td>√</td><td>√</td></tr><tr><td>mean()</td><td>√</td><td>√</td></tr><tr><td>min()</td><td>√</td><td>√</td></tr></table>

另外，后面我们会告诉大家如何去查看一个查询的执行计划。

# 18.2 避免将窗口宽度设得过小

窗口（基于时间间隔对数据进行分组）通常用于聚合和降采样数据。将窗口设长一点可以提高性能。窗口过窄会导致需要更多的算力来评估每条数据应该分配到哪个窗口，合理的窗口宽度应该根据查询的总时间宽度来决定。

# 18.3 避免使用“沉重”的功能

下面的这些函数对于 FLUX 来说会比较很重，这些函数会使用更多的内存和 CPU，使用这些函数时要想要是否必要。

⚫ map() 

⚫ reduce() 

⚫ join() 

⚫ union() 

⚫ pivot() 

不过官方又说，InfluxData 一直在优化 FLUX 的性能，所以当前的列表不一定是将来更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

的情况

# 18.4 尽可能使用 set（）而不是 map（）

如果你要给数据查一个静态常量，那么 set 比 map 要有很大的性能优势。map 是我们上一小节说的沉重操作。在后面的示例，我们会比较两种操作的差距。

# 18.5 平衡数据的时间范围和数据精度

想要保证查询的性能良好，应该平衡好查询的时间范围和数据精度。如果，有一个measurement 的数据每秒入库一条，你一次请求 6 个月的数据，那么一个序列就能包含1550 万点数据。如果序列数再多一些，那么数据很可能会变成数十亿点。Flux 必须将这些数据拉到内存再返回给用户。所以一方面做好谓词下推尽量减少对内存的使用。另外，如果必须要查询很长时间范围的数据，那应该创建一个定时任务来对数据进行降采样，然后将查询目标从原始数据改为降采样数据。

# 18.6 使用 FLUX 性能分析工具查看查询性能

执行 FLUX查询时，你可以导入一个名为 profiler的包，然后添加一个 option选项以查看当前FLUX语句的执行计划。比如：

```hcl
option profiler.enabledProfilers = ["query", "operator"] 
```

这里的 query 和 operator 是查询计划的两个选项，query 表示你要查看整个执行脚本的的执行情况，operator表示你要查看一个 FLUX查询各个算子的执行情况。

# 18.6.1 query（查询）

query 提供有关整个 Flux 脚本执行的统计信息。启用后，结果将多出一个表，其中包含以下信息：

⚫ TotalDuration：查询总持续时间（以纳秒为单位）

⚫ CompileDuration：编译查询脚本所花费的时间（以纳秒为单位）

⚫ QueueDuration：排队所花费的时间（以纳秒为单位）

⚫ RequeueDration：重新排队花费的时间（以纳秒为单位）

⚫ PlanDuration：计划查询所花费的时间（以纳秒为单位）

⚫ ExecuteDuration：执行查询所花费的时间（以纳秒为单位）

⚫ Concurrency：并发，分配给处理查询的 goroutines。

⚫ MaxAllocated：查询分配的最大字节数（内存）

⚫ TotalAllocated：查询时分配的总字节数（包括释放然后再次使用的内存）

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ RuntimeErrors：查询执行期间返回的错误消息

⚫ flux/query-plan：flux 查询计划

⚫ influxdb/scanned-values：数据库扫描磁盘的数据条数

⚫ influxdb/scanned-buytes：数据库扫描磁盘的字节数

# 18.6.2 operator（算子）

有关一个查询脚本中每个操作的统计信息。在存储层中执行的操作将作为单个操作返回。启用此配置后，返回的结果将多出一个表，并包含以下内容

⚫ Type：操作类型

⚫ Label：标签

⚫ Count：执行这个操作的总次数

⚫ MinDuration：操作被执行多次中，最快的一次花费的时间（以纳秒为单位）

⚫ MaxDuration：操作被执行多次中，最慢的一次花费的时间（以纳秒为单位）

⚫ DurationSum：当前操作完成的总持续时间（以纳秒为单位）。

⚫ MeanDuration：操作被执行多次的平均持续时间（以纳秒为单位）。

# 18.7 示例：使用 profile 优化查询

# 18.7.1 编写查询

首先，打开DataExplorer。写下如下代码。

```python
from (bucket: "test_init")
| > range(start: -1h)
| > filter(fn: (r) => r["measurement"] == "go_goroutines")
| > map(fn: (r) => ({r with hello:"world"})) 
```

这段代码，会从 test_init 存储桶查询一个名为 go_goroutines 的 measurement，这个测量下没有tag，所以我们只有一个序列。

map函数帮我们在 filter后的数据上加了一个常量列，列名是 hello，值是字符串 world。

# 18.7.2 执行查询

现在，SUBMIT一下上面的代码，然后点击View Raw Data。数据应该如下所示。

![image](assets/95841b2119b5bc5f8bdc13a6477184f5fbcdb81842e40af19507e951cfe86a10.jpg)


可以看到有一个常量列。

# 18.7.3 修改代码查看性能指标和执行计划

现在，我们对代码做出修，以方便我们观察执行计划。

```python
import "profiler"
option profiler.enabledProfilers = ["query", "operator"]
from (bucket: "test_init")
    | > range(start: -1h)
    | > filter(fn: (r) => r["measurement"] == "go_goroutines")
    | > map(fn: (r) => ({r with hello:"world"})) 
```

# 代码解释：

⚫ option profiler.enabledProfilers，这其实是个开关选项，当后面的列表出现“query”时，就显示整个查询的性能和执行计划，当然出现"operator"时，会具体显示每个算子的性能指标。

⚫ import "profiler"，引包，enabledProfilers 是 profiler 中的开关，不引没法用。

现在，再次点击 SUBMIT，同样还是观察原始数据。一上来展示的还是我们查询出来的数据，需要切换到页尾才能看到性能指标和执行计划。

![image](assets/2c5422aef5ab58acf4d4099315d47b6143d1dbb296e96a8d294ffbcc147ef3f2.jpg)


现在，我们看到两张表，有一个_measurement 为 profiler/query，这是我们整个查询的性能指标和执行计划。还有一个_measurement 为 profiler/operator 的，这是我们每个算子的性能指标。里面包括某个算子运行了多长时间等信息。

<table><tr><td>table _PROFILER</td><td>measurement GROUP STRING</td><td>CompileDuration NO. GROUP LONG</td><td>Concurrency NO. GROUP LONG</td><td>ExecuteDuration NO. GROUP LONG</td><td>flux/query-plan NO. GROUP STRING</td><td></td><td></td><td></td></tr><tr><td>0</td><td>profiler/query</td><td>182810</td><td>0</td><td>6644771</td><td colspan="4">digraph \{ &quot;merged_ReadRange4_filter2&quot; &quot;map3&quot; &quot;merged_ReadRange4_filter2&quot; -&gt; &quot;map3&quot; \}</td></tr><tr><td>table _PROFILER</td><td>measurement GROUP STRING</td><td>Count NO GROUP LONG</td><td>DurationSum NO GROUP LONG</td><td>Label NO GROUP STRING</td><td>MaxDuration NO GROUP LONG</td><td>MeanDuration NO GROUP DOUBLE</td><td>MinDuration NO GROUP LONG</td><td>Type NO GROUP STRING</td></tr><tr><td>1</td><td>profiler/operator</td><td>1</td><td>3425879</td><td>merged_ReadRange4_filter2</td><td>3425879</td><td>3425879</td><td>3425879</td><td>*influxdb.readFilterSource</td></tr><tr><td>1</td><td>profiler/operator</td><td>3</td><td>3273240</td><td>map3</td><td>3229716</td><td>1091080</td><td>1331</td><td>*universe.mapTransformation</td></tr></table>

# 18.7.4 如何判断谓词下推

按照官方文档的说法，如果实现了谓词下推，那么多个 operator 会合并成一个。我们可以看到现在操作列表里，有一个叫做 merged_ReadRange4_filter2 的算子操作，后面紧跟的是我们的 map操作。这说明 from -> range -> filter被合并了。它们是一步操作。

![image](assets/9168c51b946bb12180060c12690a1a051da5f06fcc0f553249f30106e4cb513e.jpg)


# 18.7.5 查看查询性能

查询性能有很多指标，但是我们现在只关注两个指标，一个是 MaxAllocated。它表示的是我们为了完成查询，总共使用过的内存（包含释放后又申请的内存）。

![image](assets/aeeadba088a80a5f5c458defb095fb25e7ca0bf2ea1d4a7dd3bd6b93b5e0c845.jpg)


现在这个指标的数值是 52736，也就是说为了完成这个查询，我们前后用了大概 50kb的内存。

另一个是TotalDuration，表示执行这个操作的总时间，现在是17365972纳秒

![image](assets/a217d3c5c198e8723e6dac9c023533166b23bc8f0f4e9fe6d48c6d2acd7a8dbc.jpg)


# 18.7.6 在 map 后增加 AggregateWindow

现在，我们在 map 后面加上一个 AggregateWindow 函数。

整体代码如下：

```python
import "profiler"
option profiler.enabledProfilers = ["query", "operator"]
from (bucket: "test_init")
    | > range(start: -1h)
    | > filter(fn: (r) => r["measurement"] == "go_goroutines")
    | > map(fn: (r) => ({r with hello:"world"}))
    | > aggregateWindow(column: "_value", every: 1h, fn: mean) 
```

红色的部分是我们新增的代码。

# 18.7.7 查看查询性能

首先关注我们的 operator 表，可以看到，操作数从之前的 2 个变成了 3 个。map 的后面，多了一个聚合窗口操作。

<table><tr><td>table _measurement PROFILER</td><td>GROUP STRING</td><td>Count NO GROUP LONG</td><td>DurationSum NO GROUP LONG</td><td>Label NO GROUP STRING</td><td>MaxDuration NO GROUP LONG</td><td>MeanDuration NO GROUP DOUBLE</td></tr><tr><td>1</td><td>profiler/operator</td><td>1</td><td>3979552</td><td>merged_ReadRange10_filter2</td><td>3979552</td><td>3979552</td></tr><tr><td>1</td><td>profiler/operator</td><td>3</td><td>3708401</td><td>map3</td><td>3656369</td><td>1236133.6666666667</td></tr><tr><td>1</td><td>profiler/operator</td><td>3</td><td>47395</td><td>aggregateWindow9</td><td>45632</td><td>15798.333333333334</td></tr></table>

查询的 MaxAllocated 依然是 52736，没有变。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/420471707e42e194339a9717846c89e1b9ac7467339ce9312018c7204e5ea727.jpg)


因为之前需要返回几百条数据，现在开窗聚合，只需要返回两条数据，所以查询的持续时间有所缩短。

![image](assets/600b49f8c79db55d49fcc18216e38d8e79896a97bcc3f2fadf16328336e04877.jpg)


# 18.7.8 将 AggregateWindow 移至 map 前

现在，我们将 AggregateWindow 移到 map 之前 filter 之后。修改后的代码整体如下：

```python
import "profiler"
option profiler.enabledProfilers = ["query", "operator"]
from (bucket: "test_init")
    | > range(start: -1h)
    | > filter(fn: (r) => r["measurement"] == "go_goroutines")
    | > aggregateWindow(column: "_value", every: 1h, fn: mean)
    | > map(fn: (r) => ({r with hello:"world"})) 
```

注意红色是修改的地方。

# 18.7.9 查看查询性能

首先还是关注 operator 表，这张表里之前是 3 个操作，现在变成了两个。map 之前，有一个 ReadWindowAggregateByTime 操作。也就是说，我们的 aggreagteWindow 操作实现了谓词下推。

<table><tr><td>table _measurement GROUP STRING</td><td>Count NO GROUP LONG</td><td>DurationSum NO GROUP LONG</td><td>Label NO GROUP STRING</td><td>MaxDuration NO GROUP LONG</td><td>MeanDurat NO GROUP DOUBLE</td></tr><tr><td>1</td><td>profiler/operator</td><td>1</td><td>605598</td><td>ReadWindowAggregateByTime11</td><td>605598</td></tr><tr><td>1</td><td>profiler/operator</td><td>3</td><td>264038</td><td>map8</td><td>224315</td></tr></table>

当读磁盘的操作完成后，内存中只会存在聚合后的两条数据。

现在我们关注查询性能指标。

可以看到 MaxAllocated 变成了 864，之前这一指标的数值还是 52736。之前要消耗50KB，现在却1KB都不到。

![image](assets/5a9deb34017d156cc84071f5e61c01ea80a830ec6c657cb0e948ed9b1a438780.jpg)


查询的持续时间也有进一步缩短。

![image](assets/6745c91ebcebf867588b8c682244eb20ed03676a6efd043009a69be6bdf51adc.jpg)


# 18.7.10 将 map 改为 set

最后值得说一下，我们的 map 操作数据的原理是对数据集中的数据一行一行处理。此处，我们用它实现了添加常量的功能。其实还有一个同样能完成此类任务的算子叫 set，它操作数据的逻辑是直接操作整个数据集。

数据量越大，这两个算子的性能差距就越明显。

此处，我们将 aggregateWindow 算子去掉，并将 map 改成 set。与第一次查看性能时的代码做比较，当时我们还没有做聚合操作。

改完的代码整体如下：

```hcl
import "profiler"
option profiler.enabledProfilers = ["query", "operator"] 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```python
from (bucket: "test_init")
| > range(start: -1h)
| > filter(fn: (r) => r["measurement"] == "go_goroutines")
| > set(key: "hello", value: "world") 
```

# 18.7.11 查看查询性能

运行之后，查看查询性能。可以看到用 Set 版的 MaxAllocated 是 51712，这跟 map 版的52736几乎没啥区别。

![image](assets/610f0c3f5fe565b3df6f96b0fe395bc87624a53ebc7a6c3792bac1208a380f49.jpg)


但是，我们看一下 TotalDuration 这个指标 4612562。之前的 map 版在这个指标上可是17365972。

![image](assets/45fa5d34f46e156b561c09a92eafdb6cc44237f76890aaed528f42093cbb6034.jpg)


这说明set操作要比 map要快。

# 第19章 使用 InfluxDB 搭建报警系统

# 19.1 什么是监控

监控其实每隔一段时间对数据计算一下。比如，我有一个一氧化碳浓度传感器， 每 1分钟我就算一下这 1 分钟内室内一氧化碳浓度的平均值。将这个结果跟一个写死的标准值做比较，如果超过了就报警。这就是监控的基本逻辑。

所以，InfluxDB 中的监控其实也是一个 FLUX 脚本写的定时任务。只不过，不管是在HTTP API还是在 Web UI上，InfluxDB都把它和定时任务分离区别对待了。

# 19.2 认识检查、报警终端和报警规则

在 Web UI 的左侧工具栏中，点击 Alerts 按钮，会打开一个报警的配置页面。上方的选项 栏 中 显 示 着 CHECKS（ 检 查 ）、NOTIFICATION ENDPOINTS（ 报 警 终 端 ） 和NOTIFICATION RULES（报警规则）分别对应着 InfluxDB 进行报警所需要的 3 个组件。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/d7897db406049bac0e7367ce8493460c8fa2e7e4787e07a341459b7a4e6e0083.jpg)


三个组件的功能分别如下：

⚫ CHECKS（检查）：它其实也是一种定时任务，我们可以称之为检查任务。检查任务会从目标存储桶中读取部分数据然后进行阈值检查，并最终出 4 类信号。CRIT（严重）、WARN（警戒）、INFO（信息）和 OK（良好）。

![image](assets/e1f414221348da2e0125cd66e31fe41973682ae3e493103da416ca0ef608f9bc.jpg)


⚫ NOTIFICATION ENDPOINTS（报警终端）：是一个向指定地址发送报警信号的组件。

⚫ NOTIFICATION RULES（报警规则）：它可以指定哪些 Check 出问题了发送微信报警，哪些Check出问题了可以发邮件通知。它相当于 Check与报警终端之间的路由。

# 19.3 示例：模拟对一氧化碳浓度的报警

# 19.3.1 需求

假设我们现在有一个可以采集一氧化碳浓度的传感器，这个传感器通过物联网网络每隔一段时间就向我们部署在服务器上的InfluxDB插入一条数据，格式如下。

```txt
co,code=01 value=0.001 1664851126000 
```

现在，我们希望使用 InfluxDB能够完成下述的报警功能。

⚫ 当CO浓度大于0.04的时候发出CRIT（严重）级别的通知信号。

⚫ 当CO浓度介于0.04和0.01之间的时候发出WARN（警戒）级别的通知信号。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

⚫ 当 CO 浓度低于 0.01 的时候发出 OK（良好）级别的通知信号。

最终，当 CO 浓度超标时，我们希望相关的工作人员能够收到一通电话，以便对事故做出快速响应。

# 19.3.2 辅助工具

为了方便验证报警终端的效果，老师写了一个很简单的仅支持 POST 请求的 HTTP 服务。在我们的课程资料里面有相关的源码和编译好后的可以在 linux_x86_64 平台上执行的程序。

直接使用下面的命令即可开启一个监听本地 8080 端口的 POST HTTP 服务。

```txt
./simpleHttpPostServer-linux-x64 
```

这个命令执行后会阻塞终端，当它接收到 POST 请求后，会自动将请求体中的内容打印到终端。如果 0.0.0.0 不是你想绑定的 host 或者 8080 端口已经被占用。你可以使用下面的两个参数来修改。

```txt
-h 指定绑定的 host
-p 指定绑定的 port
```

例如：

```txt
./simpleHttpPostServer-linux-x64 -h localhost -p 8080 
```

具体可以参考项目地址： https://github.com/realdengziqi/simpleHttpPostServer

# 19.3.3 创建一个新的存储桶

为了避免我们将本示例的数据同之前的示例搞混，此处我们先创建一个新的名为example_alert 的存储桶，如下图所示：

![image](assets/c0f777c2de8f27558eed51f63fdddadea1f0d0adecf556c06925e874dc19d179.jpg)


# 19.3.4 准备数据模板

在本示例中，我们会自己手动一条条地向 InfluxDB 中插入数据，所以可以打开一个文本编辑器（本教程使用 vs code）先编写一个 InfluxDB 行协议的数据模板，后面可以直接

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

复制数据，稍微改一下数值然后接着插入。

数据模板如下：

```txt
co, code=01 value=0.001 
```

# 19.3.5 事先插入一两条数据

这一步操作是为了后面进行创建检查的操作时，查询构造器中有东西可选。所以为了顺利创建检查，这一步的操作不可以省略。

此处，我们在 Web UI 导入行协议数据的窗口上，分两次各导入一条数据。如图所示：

![image](assets/a42a8060f2404ba59cbd0f470d39c592e20bc2b940ea5abd61f1676f3d4183ee.jpg)


数据如下：

第一次

```txt
co, code=01 value=0.0015 
```

第二次

```txt
co, code=01 value=0.0025 
```

# 19.3.6 创建检查（CHECK）

（1）在左侧的工具栏中点击 Alerts 按钮。默认情况下会进入 CHECKS 的页面，如图所示：

![image](assets/0b2b2d1630e9f90b00e6eca5289b08b0d519fea9e996b32cbf99ebd12cec9131.jpg)


（2）将鼠标悬停在右上角 CREATE按钮，会弹出一个下拉菜单，其中包括两个按钮:

⚫ THRESHOLD CHECK（阈值检查）：这类检查任务主要是去判断数据有没有超出某种阈值限定。

⚫ Deadman Check（死人检查）：这类检查任务是去判断某个序列下多长时间没有写入新的数据了。你也可以设定一个值，比如一旦超过 30s 某个序列还没有数据入库，就发出一个警戒信号。

此处，我们选择Threshold Check，创建一个阈值检查。

![image](assets/4ef97fc090e8a50f4783b26f943b79da07cfd67be3da5457f55ab84aef668fc3.jpg)


（3）后面会弹出一个对话窗口，其布局和 Data Explorer 非常像，但是功能上会有一些差异。

![image](assets/f3259ebccda5b25dc857e66683d2355e0b63b67da656080fa8e1a0aee51e9ee7.jpg)


⚫ 最上方有一个 Name this Check，点击一下可以给当前创建的 Check 命名。

⚫ 左上角有一个选项卡，默认是选中了 DEFINE QUERY（定义查询），也就是上图所示的页面效果。

⚫ 页面的下方是一个查询构造器，需要注意，此处我们无法切换到脚本编辑器，也就是在此处，我们只能使用查询构造器来实现查询。

⚫ 最右边还有一个清单。上面说到为了创建一个阈值检查，你必须选择：

一个字段

一个聚合函数（也就是开窗后的聚合函数）

一个或更多的值域。

（4）现在，我们需要构造查询。如下图所示。

![image](assets/8587a10e27cd569af4edadd9be5127a7cc3cddce0752753f2b959991b1cc1d70.jpg)


a) 在存储桶处选择 example_alert

b) _measurement 处选择 co

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

c) 注意，虽然我们当前 co 这个 measurement 下只有一个序列，但是还是必须将_field=value 添加到过滤条件中，否则右上方检查项 One Field 不会通过。

d) 最后将聚合逻辑从默认的 mean 改为 max。

e) 点击submit，可以预览数据的查询效果。

（5）点击左上方的 CONFIGURE CHECK按钮。这会让我们进入一个新的页面，在这个页面中，我们可以对阈值进行配置。

首先要注意，只有页面的下半部分发生了改变。

![image](assets/7af76a82be4a76d166c049ca787c8a3b9575ef5f38bcf9492cb84a4c8abde044.jpg)


⚫ 最左侧的卡片对应的是查询和调度的进一步配置。这里我们将 Schedule Every 设为15s这样，每隔15秒就会调用一次检查。

⚫ 中间的 STATUS MESSAGE TEMPLATE 是状态消息模板。这里支持使用 Shell 风格的取值语法。${ }。这里 r的含义在后面会进行详细的讲解。此处保持默认的模板不做任何修改。

⚫ 最右侧的 THRESHOLDS 对应的是值域的设定。此处包含 4 种类型的值域，对应着一个检查能够发出的4中状态信息。它们分别是：

CRIT（critical的前4个字母）表示严重紧急。

WARN（warning）表示警告、警戒

Info（Information）表示普通信息，提醒

ok 表示状态良好

此时，点击右下角的 CRIT按钮，会弹出一个小的设置窗口，如下图所示

![image](assets/ee4abaca5dedb3b402f55929e4df1aadf989bac903e93719a131377af5cdae01.jpg)


更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

这里的意思就是，当值大于多少的时候将检查的状态设置为 CRIT。此处 When value右边的 is above 就是大于的意思，可以看到这还是一个下拉菜单。我们可以点击一下。会发现它还有更多可选的选项，包括 is below（小于）、is inside range（在什么范围之内）等。

![image](assets/fb825eb848c011ae527c8f7a42e2f1b647da8659f3c6ec6e4b8641caf7ec8228.jpg)


0.00125 是 Web UI 根据我们当前的查询结果自动帮我们填充的。这里，根据我们的需求，co 的浓度值大于 0.04 的时候才将状态置为 CRIT。效果如下。

![image](assets/868fec7d0c413ae8b933e67b7f115505303066ccb3e9175c1b56a60a91320230.jpg)


同理，设置Warn和ok，Info就不设置了。结果如下。

![image](assets/56658baf6353a73fc0f6cfdbf1cb5783b0776ea6e1c24b6ec6da93626b0c2db7.jpg)


最后点击右上角的对号，保存Check。

现在，我们回到了最初的 CHECKS 页面，可以看到，下方的列表里就有一个我们刚才配置的 Check。

![image](assets/dc400fcfd167154fdba0cfc6ed93a71864efd18b6172b96ec996984c728a37f2.jpg)


# 19.3.7 测试 Check

现在，可以回到上传数据的页面，尝试插入两条数据测试一下检查的运行效果。

![image](assets/e9a09dfa9a5bc93308ff5d841f697e6f6d5f1786f4417bfc4219518fad1d10cf.jpg)


插入的数据如下：

```txt
co, code=01 value=0.025 
```

这时一氧化碳的浓度为 0.025，介于 0.01 到 0.03 之间，这个时候我们刚才创建的CHECK应该发出WARN级别的信号。

现在，我们可以在左侧的工具栏里，点击 Alert History。

![image](assets/db631d5546416390951dee18b3d9c5daf4804ae176f57d1d952c86ff1daf3438.jpg)


可以看到，现在我们的状态记录里面就躺着一条级别为 WARN的通知。这里，右面的MESSAGE 显示 Check:CO_Alert is : warn 就是我们的消息模板生成的消息。

# 19.3.8 修改消息模板

当前，我们的消息模板提示的消息还不够精确，我们希望进行报警的时候能够把当前的一氧化碳浓度的值也输出出来。

这个时候可以看一下官方文档关于模板的说法，可以发现，官方文档指出，我们是可以通过r.字段名的方式去访问到数据具体值的。

![image](assets/231039260e801c71c60448e84e0c1599ca78861c9f6caadfd1a564eedf87a893.jpg)


这样的话我们就可以重新修改消息模板，最终的消息模板如下图所示：

![image](assets/72d183a3f8b8264a605c6cbc04c6a31136b6d9776337b4df855dca143f1e2368.jpg)


注意，模板中的 r.code 和 r.value。通过这个操作，我们可以直接提取数据中的设备编号和当前的一氧化碳浓度值。

# 19.3.9 验证消息模板

接下来，我们再次插入一条数据。

```txt
co, code=01 value=0.0146 
```

0.0146 在 0.01 和 0.03 之间，我们之前创建的 CHECK 应该再发出一个 WARN 级别的信号。

现在，还是在左侧的工具栏点击 Alter History，查看检查汇报的状态记录。我们发现新的状态记录里面的 MESSAGE 有所改变，这次在信息中我们可以看到设备编号和当时的一氧化碳浓度了。

![image](assets/3956cc4c6602f0881ca906e1a8cfaaa5481b82247c33ab2fa9d7e995b1d7ed8e.jpg)


# 19.3.10 创建报警终端（NOTIFICATION ENDPOINT）

仅有状态记录还不够，我们还需要将信息发送到外部系统，比如给开发人员发送邮件，或者拨打电话。那么这个负责向外面发送消息的组件就是报警终端。

（1）首先点击左侧工作栏中的 Alerts 按钮，进入页面后，在上方的工具栏选择NOTIFICATION ENDPOINTS 选项卡。

![image](assets/5a66c22d673bf99ec8ad1eefe0440a97368e00a26c0395150a8567cbc3114ba1.jpg)


（2）点击右上角的CREATE按钮，会弹出一个如下图所示的对话窗。

![image](assets/5f261e2e22ea144f53951fa3eea3a191231d46306acc40cfdec1af298e2cc1c2.jpg)


左上角的 Destination 有个下拉菜单，这个其实是报警终端的类型，可以看到此处为我们提供了 3 种终端，HTTP、Slack 和 Pagerduty。Slack 和 Pagerduty 是海外开发团队常用的通讯软件，此处我们选择HTTP。

![image](assets/6c4820e7a1d6a6d29d8d6d4853ffcb2c4772cca65d59449ab70a118b0efd7442.jpg)


（3）选择 HTTP后，可以看到窗口中的配置项会发生变化。所谓 HTTP终端，其实就是向一个目标地址发送POST请求。

![image](assets/0bba53f171db7dd10eccebb5a72f1daca437e881b9cc64a89ea69fd1ffbc14c3.jpg)


（4）我们暂时不向睿象云对接，而是想办法先去观察一下 HTTP终端发出数据的数据结构。此处，在资料包中找到我们的辅助工具 SimpleHttpPosyServer。

![image](assets/e26471e1b636fc9300d661815ceb2ac6bc199cd4b010112b76a20e63b27148d8.jpg)


将它拷贝的Linux虚拟机后，执行下述命令。

```txt
./simpleHttpPostServer-linux-x64 
```

执行后，程序会监听 0.0.0.0:8080地址。当它收到 POST请求时，会在终端打印收到的数据。

（5）现在，我们可以将 InfluxDB 中的 HTTP 终端地址设为 http://host1:8080。如下图所示：

![image](assets/fe88d82f405080d975c711a07b30d5c24100c8761a0242cc50b06154beaac940.jpg)


（6）最后点击右下角的 CREATE NOTIFICATION ENDPOINT，创建终端。

# 19.3.11 创建报警规则（NOTIFICATION RULES）

报警规则起到报警信息和终端之间的路由作用。报警规则可以指定哪些 Check 的何种级别的信息发送给哪个终端。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

注意！创建报警规则的前提是已经创建了至少一个报警终端，否则 Web UI 上的创建报警规则按钮会变成灰色，也就是无法创建报警规则。

（1）首先在左侧工具栏点击 Alerts 按钮，然后在上方的选项卡点击 NOTIFICATION。接着点击 CREATE 按钮。

![image](assets/f8d9896914fc309fb84019d826d2b5de5075948d56361c2056b1ac426c0d3705.jpg)


（2）现在可以看到一个设置报警规则的弹窗。如下图所示。

![image](assets/ca756c162ff8e9598c6c8eec0c4add9bc0c30286e5383e0eaa04672e30f4fa74.jpg)


最上方可以设置调度时间，看上去就像是一个定时任务。中间 Conditions 的意思是条件。比如现在默认的条件就是当 InfluxDB 中有 CHECK 的状态为 CRIT 时，就使用http_endpoint 发送报警信息。需要注意中间的 Conditions 还有一个按钮叫做 tag Filter，也就是按照标签过滤。

（3）首先，为了能够更快的看到报警效果，我们将调度的时间设为 15 秒。名字可以

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

自己随便起。效果如下图所示。

![image](assets/1bb8eb3fbe9809d8e1c62c043bbe2cd3b7f73de16e2e19e513b8a7870ca60a9e.jpg)


（4）在中间的 Conditions 区域，点击一下 Tag Filter 按钮，然后添加一个标签过滤条件为_check_name == CO_Alert。其中 CO_Alert 是我们之前创建的检查的名称。后面我们会讲到检查和通知规则的工作原理。此处先这样设置，设置后的效果如下图。

![image](assets/fc9a8af0df20cf4ae51e183a63f6a57adb93f23b37ce1e9955bcb5a6d7d8acb2.jpg)


（5）窗口最下方的 Message区域，因为我们目前只有一个名为 http_endpoint的终端，所以这里 UI 自动帮我们选择的 http_enpoint，保持现状就好。

![image](assets/b54118e20cc47fbccb0daff914497a0e0963917a00adb3613d3db81ffac7b259.jpg)


（6）最后点击最下方的 CREATE NOTIFICATION RULE 按钮，创建规则。

![image](assets/7d60726408e19c28fd5475330fd5d683054a7fbeb0f9fd2f502fba8cdd5395a6.jpg)


# 19.3.12 测试报警信号发送效果

（1）现在，我们要测试一下在 InfluxDB 里已经搭建好的报警链路。只需插入一条模拟的一氧化碳浓度的数据，让它的值大于0.04即可。

插入的数据如下：

```txt
co, code=01 value=0.05 
```

如图所示：

![image](assets/e662d513bf94824b3946001b76a0051a0ca83fa053ede85352c725462c684694.jpg)


（2）接下来，在左侧工具栏点击 Alert History，来到报警历史页面，等待大概 15秒。

![image](assets/a61b3c3c9681a0190ff14e968376a41c88c4552d3126cb4979a7890669eb9dd7.jpg)


正常情况下，在检查状态历史记录里应该会出现一个级别为 crit的状态信息。

（3）点击上方的 NOTIFICATIONS 按钮，这时应该出现一个通知记录，这个列表里面是 InfluxDB 向外发送通知的记录。可以看到这条记录的最右边有一个绿色的对号，这说明我们的消息已经成功通过 http_endpoint 发送出去了。

![image](assets/fc54200f447e1949193c39bba8621c0ab36ef55446a40b0144b4f7b22d4e2b66.jpg)


（4）回到之前开启 simpleHttpPostServer的终端，看一下里面的内容。

```jsonl
[atguigu@host1 module]$ ./simpleHttpPostServer-linux-x64
{"check_id":"0a0adf75eddc:000","check_name":"C0_Aler","level":"crit","measurement":"notifications","message":"Check: C0_Aler is: crit\natguigu yyds\n01 is: 0.06","notification_endpoint_id":"0a0b1b37bae0a000","notification_endpoint_name":"http_endpoint","notification_rule_id":"0a0bife01cfb3000","notification_rule_name":"hahaha","source_measurement":"co","source_timestamp":166424499000000000,"start":2022-09-27102:16:45Z,"time":2022-09-27102:16:45Z,"type":"threshold","version":1,"code":"01","value":0.06} 
```

如图所示，我们成功收到一个 POST 请求，并将它请求体中的数据打印在了控制台。将这条json数据格式化后的效果如下：

```json
{
    "_check_id": "0a0adfc75eddc000",
    "_check_name": "CO_Alert",
    "_level": "crit",
    "_measurement": "notifications",
    "_message": "Check: CO_Alert is: crit\natguigu yyds\n01 is: 0.06",
    "_notification_endpoint_id": "0a0b1b37bae0a000",
    "_notification_endpoint_name": "http_endpoint",
    "_notification_rule_id": "0a0b1fe01cfb3000",
    "_notification_rule_name": "hahaha",
    "_source_measurement": "co",
    "_source_timestamp": 1664244990000000000,
    "_start": "2022-09-27T02:16:15Z",
    "_status_timestamp": 1664244990000000000,
    "_stop": "2022-09-27T02:16:45Z",
    "_time": "2022-09-27T02:16:45Z",
    "_type": "threshold",
    "_version": 1,
    "code": "01",
    "value": 0.06
} 
```

可以看到，其中包含了数据的时间，报警的消息，报警的级别，事发时的一氧化碳浓度值等等。如果能够在终端看到最终的 json，说明 InfluxDB 的报警配置已经完成并且能够正常工作。

# 19.3.13 检查和报警规则的工作原理

之前我们设置报警规则的时候发现，配置报警规则是需要设置调度的时间间隔的，感觉上有些奇怪，为什么一个规则还需要每隔一段时间去执行一次呢？这要先从检查的工作原理开始讲起。

InfluxDB 安装好后，会有一个由 InfluxDB 自动创建的名为_monitoring 的存储桶。我们可以用DataExplorer去查询一下其中的内容。

查询结果如下：

![image](assets/f62c832c88565a5a5def164d712f3d74ff35d0485e74c41c55dfbb545a03524f.jpg)


可以看到，查询的结果中，包含了 Check 任务生成的报警信息。比如有一个字段名为_check_name，值为 CO_Alert，这就是我们之前设置的检查名称。

也就是我们定时执行的 Check，其实是定时从 example_alert 里面查询数据出来，然后对其进行阈值检查，最终，将检查后的状态信息写入到_monitoring 存储桶中。效果如下图所示：

![image](assets/15d9eeb37185db0e52950bea56c6d7b1876c353d2ab8dc4cadf48a343d584638.jpg)


其实后面的通知策略也是一个定时任务，它从_monitoring 查询最近一段时间的数据，并根据你设置的条件对数据进行过滤，最终如果有符合要求的数据，就使用我们的http_endpoint将数据转成 json格式发送出去。最终整个流程如下图所示。

![image](assets/27b843692eeab91c267fa4a6e87b12f9341b2876fe61c60dba3c1ab8a94fcf5c.jpg)


这也就是 Check、Notification rule 和 Notification endpoint 在一起工作的原理。

# 19.4 示例：集成睿象云（报警系统的 Saas 方案）

# 19.4.1 什么是睿象云

睿象云是一个告警平台，它提供五花八门的报警方式。你可以充值，选择电话报警，按照说明进行一下配置，之后你会得到一个 API 接口。以后，当你的系统需要报警时，只需在代码里想这个 API 发送一个 http 请求，睿象云就会按照你配置的电话号码，拨打电话，语音提醒程序员该加班了。

# 19.4.2 注册睿象云

官网地址：https://www.aiops.com/

注册过程（略）

# 19.4.3 创建自己的报警 API

# 19.4.4 在睿象云上创建报警 API

（1）首先进入睿象云的首页，点击左侧的智能告警平台按钮，进入智能告警平台的工作页面。

![image](assets/7d378cacf97c84ac0a23a0c7d2a5fdc7db67af7594a33ce84daa4d7af45fb25f.jpg)


（2）如下图所示，点击上方选项卡的集成按钮，进入集成的配置页面

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

![image](assets/eeed4db6283a9fbd5648964016b1fb0dabc01c075aab883227a290a2e5d5567f.jpg)


（3）这个时候，左侧有一个监控工具列表，可以看到睿象云可以和很多监控工具进行集成，但是这个列表里面并没有 InfluxDB，这个时候还有一种万能的集成方案，REST API。

REST API 会向外提供一个 URL，只要你的监控工具能够以 API 要求的数据格式向睿象云发送POST请求，那就可以同睿象云集成。

![image](assets/a00dc2d4cc8275fb3137295ef9ac74d7d41e3b5717d0b8e5573e49f916c262c2.jpg)


（4）这时，我们会进入一个配置页面。首先需要设置一个应用名称。再点击下方的蓝色按钮，保存并获取应用 key。

![image](assets/9c13ba9a2770cdd67e57fda66b1732defc78a5e5fea2818c206e047515caf973.jpg)


（5）这个时候页面上会出现一行红字，这个就是 Appkey。注意不要把这个 key 泄漏出去哦。

![image](assets/0e71026d6d9a6d74aa0632befb83c7cbcafb6fb4219efbc1f0c1568c7218c28d.jpg)


至此，我们的报警API就算是配置完了。

# 19.4.5 创建分派策略

现在外部可以通过接口向睿象云发送报警信息了，但是睿象云平台如何将报警信息发送给具体的个人呢。

将报警信息转发给具体的人，这个过程叫做分派。

（1）回到报警平台的主页，点击上方的配置按钮，然后再点击下方的右边的新建分派按钮。

![image](assets/baca8e3a21830f44c1a0d037e710cf9abcf4ee8a278a39f2c1d955e482251a9e.jpg)


（2）此时会进入一个新的配置页面，可以按照下图的说明进行操作。注意，如果此处设置分派人的时候啥都不显示，说明你的账户目前还没有绑定邮箱，这个时候请自行绑定邮箱，再进行后面的操作。

![image](assets/94e55a18aadd3bb65eae131d2f057ca58e176adf89188f30327a2a3532060c11.jpg)


配置好后，点击保存。

现在，我们的TICK_TEST API一旦接收到报警信息，就会通知到具体的人了。

# 19.4.6 InfluxDB 尝试与睿象云进行对接

现在，我们可以尝试与睿象云进行对接了，现在看来，只要让 InfluxDB 发出的通知数据符合睿象云 API 的要求就好了。我们可以回看一下刚才在睿象云创建的 API 的说明文档（在集成->右侧应用列表->找到你自己创建的REST API应用->点击编辑->页面下方可见）

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

如下图所示，这里说明了我们应该发送什么格式的数据。

<table><tr><td colspan="3">接口</td></tr><tr><td>接口</td><td colspan="2">http://api.aiops.com/alert/api/event</td></tr><tr><td>调用方式</td><td colspan="2">POST</td></tr><tr><td>参数格式(body)</td><td colspan="2">{&quot;app&quot;: &quot;8feac05a2e4a46c495769f198a198a3d&quot;, &quot;eventId&quot;: &quot;12345&quot;, &quot;eventType&quot;: &quot;trigger&quot;, &quot;alarmName&quot;: &quot;FAILURE for production/HTTP o n machine 192.168.0.253&quot;, &quot;entityName&quot;: &quot;host-192.168.0.253&quot;, &quot;entityId&quot;: &quot;host-192.168.0.253&quot;, &quot;priority&quot;: 1, &quot;alarmContent&quot;: {&quot;ping time&quot;: &quot;2 500ms&quot;, &quot;load avg&quot;: 0.75 }, &quot;details&quot;: {&quot;details&quot;:&quot;haha&quot;}, &quot;contexts&quot;: [{&quot;type&quot;: &quot;link&quot;, &quot;text&quot;: &quot;generatorURL&quot;, &quot;href&quot;: &quot;http://www.baidu.com&quot; }, {&quot;type&quot;: &quot;link&quot;, &quot;href&quot;: &quot;http://www.sina.com&quot;, &quot;text&quot;: &quot;CPU Alerting&quot; }, {&quot;type&quot;: &quot;image&quot;, &quot;src&quot;: &quot;http://www.baidu.com/a.png&quot;}]}</td></tr><tr><td>参数格式(URL参数方 式)</td><td colspan="2">?app=8feac05a2e4a46c495769f198a198a3d&amp;eventId=xxx&amp;eventType=trigger&amp;alarmName=xxx&amp;priority=2</td></tr><tr><td colspan="3">参数列表</td></tr><tr><td>参数</td><td>必须</td><td>备注</td></tr><tr><td>app</td><td>必须</td><td>需要告警集成的应用KEY</td></tr><tr><td>eventType</td><td>必须</td><td>触发告警trigger,解决告警resolve</td></tr><tr><td>eventId</td><td>可选</td><td>外部事件id,告警关闭时用到</td></tr><tr><td>alarmName</td><td>可选</td><td>告警标题,alarmName与eventId不能同事为空</td></tr><tr><td>alarmContent</td><td>必须</td><td>告警内容详情</td></tr><tr><td>entityName</td><td>可选</td><td>告警对象名</td></tr><tr><td>entityId</td><td>可选</td><td>告警对象id</td></tr><tr><td>priority</td><td>可选</td><td>提醒1,警告2,严重3,通知4,致命5</td></tr><tr><td>host</td><td>可选</td><td>主机</td></tr><tr><td>service</td><td>可选</td><td>服务</td></tr></table>

# 19.4.7 报警终端的不足

现在我们回到 Web UI 的Alerts页面，并点击到http_endpoint的编辑页面。会发现，没有地方可以修改发送数据的格式。不错，InfluxDB 的报警终端就是没法去设置发送的数据格式的。所以到这一步，我们前功尽弃了。但是老师还有一个方案，下一节我们会直接接触检查与报警的底层。

![image](assets/7de989d50d5195694e1554833b0811d1bb2c7b8d7c3fe3cf6f499d216c72eae1.jpg)


# 19.5 示例：Notebook 与报警的底层

在之前，我们说过使用 Notebook 也可以创建报警任务，但是之后就再也没有接触过Notebook。这一节，我们直接借助Notebook，走进报警的底层。

# 19.5.1 使用 Notebook 创建报警任务

（1）首先，在左侧点击 Notebooks 按钮，来到 Notebooks 的配置页面。然后，点击Set an Alert 模板，创建一个新的 notebook。

![image](assets/4a1ddb1964ecd7b1de2896988aa65fa7e214b936be59b7e3b8ad16bc64f18f5b.jpg)


（2）进入 Notebook 后，会发现第一个 Cell 是一个查询构造器。此处，我们将存储桶设为 example_alert，_measurement 设为 co，_field 设为 value。效果如下图所示：

![image](assets/54442c2118c7e89d66a31b24f87a5d887dfa2d30f6c1f69f84491779b97baaf1.jpg)


（3）点击上方的RUN按钮，查看执行效果。

<table><tr><td>table _measurement</td><td>_field</td><td>_value</td><td>_time</td><td>code</td></tr><tr><td>GROUP 
STRING</td><td>GROUP 
STRING</td><td>NO GROUP 
DOUBLE</td><td>NO GROUP 
DATETIME:RFC3339</td><td>GROUP 
STRING</td></tr><tr><td>0</td><td>co</td><td>value</td><td>0.001</td><td>2022-09-26T20:17:50.312Z</td></tr><tr><td>0</td><td>co</td><td>value</td><td>0.0015</td><td>2022-09-26T20:18:07.156Z</td></tr><tr><td>0</td><td>co</td><td>value</td><td>0.02</td><td>2022-09-26T20:58:08.748Z</td></tr><tr><td>0</td><td>co</td><td>value</td><td>0.025</td><td>2022-09-26T21:03:11.829Z</td></tr><tr><td>0</td><td>co</td><td>value</td><td>0.023</td><td>2022-09-26T21:10:12.294Z</td></tr><tr><td>0</td><td>co</td><td>value</td><td>0.013</td><td>2022-09-26T21:18:16.232Z</td></tr><tr><td>0</td><td>co</td><td>value</td><td>0.0146</td><td>2022-09-26T21:24:37.484Z</td></tr><tr><td>0</td><td>co</td><td>value</td><td>0.024</td><td>2022-09-27T00:41:15.425Z</td></tr></table>


![image](assets/a6774d8621145d8966410c5a7e3e43241224d5f4bd28aeabe987ca1590d7690e.jpg)


可以看到下方的两个 cell，一个 cell 是将查询出来的数据原样展示了出来，另一个是将数据绘制成了折线图。

（4）最后，下面还有一个 cell，左上角有这个 cell的名称，New Alert。也就是说明这个cell是用来配置报警的。

![image](assets/a525c15c80747cc8a165a1124ebbe967e6a1f8cbf18d4f0c32f051393e564974.jpg)


上方有两块，一个是用来设置报警条件的，另一个是用来设置调度间隔的。此处，我们还是按照需求，将报警阈值设为 0.04。细心的同学可能发现，我们此处的报警阈值只能设置一种，少了crit、warn、info和ok，这个问题我们后面会提到。

设置好后的效果如下图所示。

![image](assets/e1415710d29be0aebd6c2acc739d741d7a97487c919ba9ab7ad96718c4c4cba2.jpg)


（5）再看到这个 cell的底部，这是报警终端的配置。此处，我们还是选择 http终端。并将目标 URL 设置为 http://host1:8080。

![image](assets/eb2331975d3c64c90b18dbc96666c869bba801c009de4769b5dbd67966bcc5c3.jpg)


（6）上面的操作都完成后，点击右下方的 EXPORTALERT TASK按钮。

![image](assets/ea187bfa9a598220bb9e92d042693b557028954d8af2e0c372df7a32453e7c02.jpg)


我们会惊喜地发现 Notebook直接为我们生成了一个很长的 FLUX脚本。如下图所示。现在，建议大家先将脚本复制出来，粘贴到 Data Explorer 里。稍后，我们自己研读这段脚本。

![image](assets/27100ce99473b5cec3ff8c8260d57cd232254b460a9a06a1620e87d982b3d29f.jpg)


# 19.5.2 脚本解读

脚本如下：

```python
import "strings"
import "regexp"
import "influxdata/influxdb/monitor"
import "influxdata/influxdb/schema"
import "influxdata/influxdb/secrets"
import "experimental"
import "http"
import "json"

option task = {name: "Notebook Task for local_8dc08939-f5df-447e-8e53-41532537902f", every: 10m, offset: 0s}

option v = {timeRangeStart: -24h, timeRangeStop: now()}

check = {_check_id: "local_8dc08939-f5df-447e-8e53-41532537902f", _check_name: "Notebook Generated Check", _type: "custom", tags: {}}
notification = {_notification_rule_id: "local_8dc08939-f5df-447e-8e53-41532537902f", _notification_rule_name: "Notebook Generated Rule", _notification_endpoint_id: "local_8dc08939-f5df-447e-8e53-41532537902f", _notification_endpoint_name: "Notebook Generated Endpoint"}}

task_data = from(bucket: "example_alert") | > range(start: v.timeRangeStart, stop: v.timeRangeStop)
    | > filter(fn: (r) => r["measurement"] == "co")
    | > filter(fn: (r) => r["field"] == "value") 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

```txt
trigger = (r) => r["undefined"] > 0
messageFn = (r) => "${strings.title(v: r._type)} for
${r._source_measurement} triggered at ${time(v:
r._source_timestamp)}!"
task_data
|> schema["fieldsAsCols"]()
|> set(key: "_notebook_link", value:
"http://host1:8086/orgs/d2377c7832daa87c/notebooks/0a0bc4b03a6ba000")
|> monitor["check"](data: check, messageFn: messageFn, crit: trigger)
|> monitor["notify"]
(data: notification,
endpoint: http["endpoint"](url: "http://host1:8080")(mapFn: (r) => {
body = {r with _version: 1}
return {headers: {"Content-Type":
"application/json"}, data: json["encode"](v: body)}
},
),
) 
```

接下来，我们就按照从前到后的顺序为大家讲解这段代码。

# 19.5.2.1 导包

最上方的 import 代码我们直接跳过，不讲解了。

# 19.5.2.2 option task

```txt
option task = {name: "Notebook Task for local_8dc08939-f5df-447e-8e53-41532537902f", every: 15s, of 
```

option task其实就是对定时任务的设置，这一行代码其实也表名了，notebook帮我们生成的报警脚本本质上是一个 InfluxDB的定时任务。

# 19.5.2.3 option v

第一行代码 option v，声明了一个 record 类型的变量，里面有两个键值对，其实分别表名了查询时间范围的开端和末端。这里显示-24h，其实是因为我们直接在 notebook 里面操作的时候，右上角的时间范围设置了-24h。后面我们会把它改成-15s。

```txt
option v = {timeRangeStart: -24h, timeRangeStop: now()}

check = {_check_id: "local_a7502f79-713a08-ac2e-790443b64521", check_name: "Notebook Generated Check", notification = {_notification_rule_id: "local_a7502f79-7f9f-4a08-ac2e-790443b64521", _notification_rule_name: "Notebook Generated Check", notification = {_notification_rule_id: "local_a7502f79-7f9f-4a08-ac2e-790443b64521", _notification_rule_name: "Notebook Generated Check", notification = {_notification_rule_id: "local_a7502f79-7f9f-4a08-ac2e-790444b64521", _notification_rule_name: "Notebook Generated Check", notification = {_notification_rule_id: "local_a7502f79-7f9f-4a08-ac2e-790444b64521", _notification_rule_name: "Notebook Generated Check", notification = {_notification_rule_id: "local_a7502f79-7f9f-4a09-ac2e-790444b64521", _notification_rule_name: "Notebook Generated Check", notification = {_notification_rule_id: "local_a7502f79-7f9f-4a09-ac2e-790444b64521", _notification_rule_name: "Notebook Generated Check", notification = {_notification_rule_id: "local_a7502f78-7f9f-4a08-ac2e-790444b64521", _notification_rule_name: "Notebook Generated Check", notification = {_notification_rule_id: "local_a7502f78-7f9f-4a08-ac2e-790444b64521", _notification_rule_name: "Notebook Generated Check", notification = {_notification_rule_id: "local_a7502f79-7f9f-4a08-ac2e-790444b64521", _notification_rule_name: "Notebook Generated Check", notification = {_notification_rule_id} 
```

# 19.5.2.4 check 和 notification 两个变量

这两行代码分别声明了一个 record，它其实是用来给后面的 monitor函数做参数用的，

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网因为_moitoring 存储桶中需要_check_id 和_check_name 和_type 等字段，所以 notebook 自动生成的时候自动帮我们安排好了。

```txt
check = { _check_id: "local_a7502f79-7f9f-4a08-ac2e-790443b64521", _check_name: "Notebook Generated Check", _type: "notification = { _notification_rule_id: "local_a7502f79-7f9f-4a08-ac2e-790443b64521", _notification_rule_name: "Note 
```

# 19.5.2.5 查询数据

```txt
task_data = from(bucket: "example_alert") |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
|> filter(fn: (r) => r["measurement"] == "co")
|> filter(fn: (r) => r["field"] == "value") 
```

上图中的代码完成了对 example_alert 存储桶的查询。并且查询的表流被赋值给了一个名为 task_data 的变量。

# 19.5.2.6 声明阈值函数

```txt
trigger = (r) => r["value"] > 0.04 
```

此处，有一个名为 trigger 的函数，可见它的主体逻辑就是一个谓词表达式。在这里声明一个函数其实是因为后面有个 monitor 函数需要传入一个谓词函数。另外，可以看到这个函数的逻辑就是用来判断一氧化碳浓度是否超过0.04的。

# 19.5.2.7 消息模板

```javascript
messageFn = (r) => "${strings.title(v: r._type)} for ${r._source_measurement} triggered at ${time(v: r._source_time 
```

这也是一个函数，不过它直接返回一个字符串，这里面的内容其实是消息模板。

# 19.5.2.8 报警逻辑

接下来这一大段都是报警的逻辑。

```javascript
task_data
|> schema["fieldsAsCols"]()
|> set(key: "_notebook_link", value: "http://host1:8086/orgs/d2377c7832daa87c/notebooks/0a0b8ac396eba000")
|> monitor["check"](data: check, messageFn: messageFn, crit: trigger,)
|> monitor["notify"]
data: notification,
endpoint: http["endpoint"](-2: "http://host1:8086")(mapFn: (r) => {
body = {r with _version: 1}
return {headers: {"Content-Type": "application/json"}, data: json["encode"](v: body)},
}), 
```

（1）首先 schema["fieldAsCols"]函数起到了转换数据结构的作用。效果如下图所示：

```txt
_field    _value
value    0.06
→ value
0.06 
```

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网

（2）set函数为表流添加了一个常量字段

（3）monitor["check"]函数起到了检查状态的作用

```javascript
| > monitor["check"](data: check, messageFn: messageFn, crit: trigger,) 
```

需要注意，data 参数传进来的 check 变量其实就是_check_id，_check_name 这些变量。messageFn 是消息模板，crit 在我们之前的 CHECK 里是一个报警级别，但是这里变成了函数的形参，传进来的函数值是 trigger 是我们之前提到过得谓词函数。

```txt
trigger = (r) => r["value"] > 0.04 
```

另 外 注 意 ，虽 然 notebook 帮 我 们 生成 的 脚 本里 面 值 传 递了 crit 参数，但是monitor["check"]函数其实还有其他可传的参数。如下图所示：

```txt
| > monitor["check"](data: check, messageFn: messageFn, crit: trigger,)
| > monitor["notify"]
    data: notification,
    endpoint: http["endpoint"](url: "http://host1:8086")(mapFn: (r) => {
    body = {r with _version: 1}
    info
    ok
    warn 
```

可以看到，还有 info、ok、warn 参数。所以其实我们还是可以手动修改脚本把这些值域范围给补上的。

（4）monitor["notify"]函数是用来向外发送数据的，可以看到里面声明了一个 http终端。最后，十分需要注意的是有一个名为body的局部变量。

```txt
body = {r with _version: 1} 
```

这个其实就是我们发送 POST请求的请求体。r是我们表流里面的数据，所以说，对接不上睿象云的症结就在这里。

我们只需要将body修改成符合睿象云api要求的格式就可以了。

（5）为什么我们不直接用 if else的逻辑来完成大于小于检查然后直接向外发送请求，而是用两个专门的 monitor 函数来完成功能呢？主要是因为 monitor 函数会在我们的 Alterhistory 里留下痕迹。也就是 monitior["check"]和 monitor["notification"]函数会向_monitoring存储桶里写检查和通知记录，这是非常重要的。

# 19.5.3 修改脚本以集成睿象云

最后，我们把 body这个局部变量做一下修改，让它符合睿象云 API要求的格式就好了。

更多Java–大数据–前端–python人工智能资料下载，可百度访问：尚硅谷官网