# 02_SparkSQL

## 1. Spark SQL概述

### 1.1 Spark SQL入门

<img src="SparkSQL _assets/media/image1.png" style="width:5.90625in;height:1.40625in" />

Spark SQL是基于spark core提供的一个用来**处理结构化数据的**模块（类库）

RDD = 计算逻辑 + 分区信息_数据位置信息 RDD\[Bean\]

DataFrame = RDD + 数据结构

它提供了一个编程抽象叫做DataFrame/Dataset，它可以理解为一个基于RDD数据模型的更高级数据模型，带有结构化元信息（schema），DataFrame其实就是Dataset\[Row\]，Spark SQL可以将针对DataFrame/Dataset的各类SQL运算，翻译成RDD的各类算子执行计划，从而大大简化数据运算编程（请联想Hive）

在idea项目中添加Spark-SQL的依赖

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">XML<br />
&lt;dependency&gt;<br />
&lt;groupId&gt;org.apache.spark&lt;/groupId&gt;<br />
&lt;artifactId&gt;spark-sql_2.12&lt;/artifactId&gt;<br />
&lt;version&gt;3.1.3&lt;/version&gt;<br />
&lt;/dependency&gt;</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
结构化数据<br />
id,name,age,gender<br />
7,kunge,32,F<br />
8,yuge,44,M<br />
9,boss,41,M<br />
10,kunge2,31,F<br />
11,yuge2,44,M<br />
12,boss3,41,M<br />
<br />
-----------------------------------------<br />
<br />
package com.doit.sql<br />
<br />
import com.doit.spark.utils.SparkUtil<br />
import org.apache.spark.SparkContext<br />
import org.apache.spark.sql.{DataFrame, SparkSession}<br />
<br />
/**<br />
* @Date: 23.2.9<br />
* @Author: Hang.Nian.YY<br />
* @微信: 17710299606<br />
* @Tips: 学大数据 ,到多易教育<br />
* @BZ: https://space.bilibili.com/388154396<br />
* @BLOG: https://blog.csdn.net/qq_37933018<br />
* @Description:<br />
*/<br />
object _01_Demo01 {<br />
def main(args: Array[String]): Unit = {<br />
// 需求 求各种性别的平均年龄<br />
// Spark-core<br />
/* val sc: SparkContext = SparkUtil.getSc<br />
sc.textFile("data/users")<br />
.map(line =&gt; {<br />
val arr = line.split(",")<br />
// 样例数据 代码注释<br />
(arr(0).toInt, arr(1), arr(2).toInt, arr(3))<br />
}).groupBy(_._4)<br />
.map(tp =&gt; {<br />
val gender = tp._1<br />
val avgAge = tp._2.map(_._3).sum.toDouble / tp._2.size<br />
(gender, avgAge)<br />
}).foreach(println)*/<br />
<br />
// 0<br />
// Spark-SQL<br />
// 获取Spark的会话环境 sql和core 统一的入口<br />
val session = SparkSession.builder()<br />
.appName("test sql")<br />
.master("local[*]")<br />
.getOrCreate()<br />
// session可以加载数据<br />
// csv jdbc orc table json<br />
//DataFrame = RDD + 结构<br />
// 10,kunge2,31,F<br />
<br />
// 1<br />
val data: DataFrame = session.read<br />
.option("inferSchema", "true")<br />
.option("header", "true").csv("data/users")<br />
// 打印 加载csv文件数据后封装的数据结构<br />
/**<br />
* root<br />
* |-- _c0: string (nullable = true)<br />
* |-- _c1: string (nullable = true)<br />
* |-- _c2: string (nullable = true)<br />
* |-- _c3: string (nullable = true)<br />
*<br />
* .option("header", "true") 第一行为头信息 结构的属性<br />
* root<br />
* |-- id: string (nullable = true)<br />
* |-- name: string (nullable = true)<br />
* |-- age: string (nullable = true)<br />
* |-- gender: string (nullable = true)<br />
*<br />
* option("inferSchema" , "true") 根据字段的值 推导数据结构<br />
* root<br />
* |-- id: integer (nullable = true)<br />
* |-- name: string (nullable = true)<br />
* |-- age: integer (nullable = true)<br />
* |-- gender: string (nullable = true)<br />
*/<br />
// 将data注册成一张表 tb_user<br />
// 2<br />
data.createOrReplaceTempView("tb_user")<br />
<br />
/**<br />
* +------+-------+<br />
* |gender|avg_age|<br />
* +------+-------+<br />
* | F| 31.5|<br />
* | M| 42.5|<br />
* +------+-------+<br />
*/<br />
<br />
//3<br />
<br />
session.sql(<br />
"""<br />
|select<br />
|gender ,<br />
|avg(age) as avg_age<br />
|from<br />
|tb_user<br />
|group by gender<br />
|""".stripMargin).show()<br />
<br />
}<br />
<br />
}</td>
</tr>
</tbody>
</table>

打印DataFrame的schema

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
//打印schema信息<br />
df.printSchema()<br />
show() 展示结果数据</td>
</tr>
</tbody>
</table>

### 1.2 Spark SQL的特性

**1.易整合**

<img src="SparkSQL _assets/media/image2.png" style="width:5.90625in;height:1.61458in" />

Spark SQL使得在spark编程中可以如丝般顺滑地混搭SQL和算子api编程（想想都激动不是！）

**2.统一的数据访问方式**

<img src="SparkSQL _assets/media/image3.png" style="width:5.90625in;height:1.82292in" />

Spark SQL为各类不同数据源提供统一的访问方式，可以跨各类数据源进行愉快的join；所支持的数据源包括但不限于： Hive / Avro / CSV / Parquet / ORC / JSON / JDBC等；（简直太美好了！）

**3.兼容Hive**

<img src="SparkSQL _assets/media/image4.png" style="width:5.90625in;height:1.8125in" />

Spark SQL支持HiveQL语法及Hive的SerDes、UDFs，并允许你访问已经存在的Hive数仓数据；（难以置信的贴心！）

**4.标准的数据连接**

<img src="SparkSQL _assets/media/image5.png" style="width:5.90625in;height:1.54167in" />

Spark SQL的server模式可为各类BI工具提供行业标准的JDBC/ODBC连接，从而可以为支持标准JDBC/ODBC连接的各类工具提供无缝对接；（开发封装平台很有用哦！）

<img src="SparkSQL _assets/media/image6.jpeg" style="width:5.57292in;height:3.1875in" />

SparkSQL可以看做一个转换层，向下对接各种不同的结构化数据源，向上提供不同的数据访问方式。

## 2. SparkSQL快速体验

### 2.1 命令行使用示例

示例需求：查询大于30岁的用户

启动Spark shell

<img src="SparkSQL _assets/media/image7.png" style="width:5.90625in;height:2.32292in" />

创建如下JSON文件，注意JSON的格式，然后上次到HDFS中

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">JSON<br />
{"name":"Michael"}<br />
{"name":"Andy", "age":30}<br />
{"name":"Justin", "age":19}<br />
{"name":"brown", "age":39}<br />
{"name":"jassie", "age":34}</td>
</tr>
</tbody>
</table>

愉快地开始使用：

如果读取本地文件系统，文件的schema为file://

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
scala&gt; val df = spark.read.json("hdfs://node-1.51doit.cn:9000/json/user.json")<br />
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]<br />
<br />
scala&gt; df.printSchema<br />
root<br />
|-- age: long (nullable = true)<br />
|-- name: string (nullable = true)<br />
<br />
<br />
<br />
scala&gt; df.filter($"age" &gt; 21).show<br />
+---+------+<br />
|age| name|<br />
+---+------+<br />
| 30| Andy|<br />
| 39| brown|<br />
| 34|jassie|<br />
+---+------+<br />
<br />
<br />
scala&gt; df.createTempView("v_user")<br />
<br />
scala&gt; spark.sql("select * from v_user where age &gt; 21").show<br />
+---+------+<br />
|age| name|<br />
+---+------+<br />
| 30| Andy|<br />
| 39| brown|<br />
| 34|jassie|<br />
+---+------+</td>
</tr>
</tbody>
</table>

### 2.2 新的编程入口SparkSession

在老的版本中，SparkSQL提供两种SQL查询起始点，一个叫SQLContext，用于Spark自己提供的SQL查询，一个叫HiveContext，用于连接Hive的查询，SparkSession是Spark最新的SQL查询起始点，实质上是SQLContext和SparkContext的组合，所以在SQLContext和HiveContext上可用的API在SparkSession上同样是可以使用的。SparkSession内部封装了sparkContext，所以计算实际上是由sparkContext完成的。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
import org.apache.spark.sql.SparkSession<br />
​<br />
val spark = SparkSession<br />
.builder()<br />
.appName("Spark SQL basic example")<br />
.config("spark.some.config.option", "some-value")<br />
.getOrCreate()<br />
​<br />
// 提供隐式转换支持，如 RDDs to DataFrames<br />
import spark.implicits._</td>
</tr>
</tbody>
</table>

SparkSession.builder 用于创建一个SparkSession。

import spark.implicits.\_的引入是用于将DataFrames隐式转换成RDD，使df能够使用RDD中的方法。

如果需要Hive支持，则需要以下创建语句：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
import org.apache.spark.sql.SparkSession<br />
​<br />
val spark = SparkSession<br />
.builder( )<br />
.appName("Spark SQL basic example")<br />
.config("spark.some.config.option", "some-value")<br />
.enableHiveSupport( ) //开启对hive的支持<br />
.getOrCreate( )<br />
​<br />
// For implicit conversions like converting RDDs to DataFrames<br />
import spark.implicits._</td>
</tr>
</tbody>
</table>

### 2.3 IDEA开发SparkSQL程序

IDEA中SparkSQL程序的开发方式和SparkCore类似。

程序如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
object Demo1_HellWorld {<br />
​<br />
 // 屏蔽掉WARN级别以下的日志（INFO,DEBUG)<br />
 Logger.getLogger("org").setLevel(Level.WARN)<br />
​<br />
 def main(args: Array[String]): Unit = {<br />
​<br />
   // 创建一个sparksql的编程入口(包含sparkcontext，也包含sqlcontext)<br />
   val spark: SparkSession = SparkSession<br />
    .builder()<br />
    .appName(this.getClass.getSimpleName)<br />
    .master("local[*]")<br />
    .getOrCreate()<br />
​<br />
   /*val sc: SparkContext = spark.sparkContext<br />
   val sqlsc: SQLContext = spark.sqlContext*/<br />
​<br />
   // 加载json数据文件为dataframe<br />
   val df: DataFrame = spark.read.json("data/people.dat")<br />
​<br />
   // 打印df中的schema元信息<br />
   df.printSchema()<br />
​<br />
   // 打印df中的数据<br />
   df.show(50, false)<br />
​<br />
   // 在df上，用调api方法的形式实现sql<br />
   df.where("age &gt; 30").show()<br />
​<br />
   // 将df注册成一个“表”（视图），然后写原汁原味的sql<br />
   df.createTempView("people")<br />
   spark.sql("select * from people where age &gt; 30 order by age desc").show()<br />
​<br />
   spark.close()<br />
}<br />
}</td>
</tr>
</tbody>
</table>

## 3. DataFrame编程详解

### 3.1 创建DataFrame

在Spark SQL中SparkSession是创建DataFrames和执行SQL的入口

创建DataFrames有三种方式：

从一个已存在的**RDD**进行转换

从**JSON/Parquet/CSV/ORC**等结构化文件源创建

从**Hive/JDBC**各种外部结构化数据源（服务）创建

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>核心要义：创建DataFrame，需要创建 “RDD + 元信息schema定义”</strong></p>
<p><em>rdd来自于数据</em></p>
<p><em>schema则可以由开发人员定义，或者由框架从数据中推断</em></p></td>
</tr>
</tbody>
</table>

#### 3.1.1 使用RDD创建DataFrame

将RDD关联case class创建DataFrame

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
object DataFrameDemo1 {<br />
<br />
def main(args: Array[String]): Unit = {<br />
<br />
val conf = new SparkConf()<br />
.setAppName(this.getClass.getSimpleName)<br />
.setMaster("local[*]")<br />
//1.SparkSession，是对SparkContext的增强<br />
val session: SparkSession = SparkSession.builder()<br />
.config(conf)<br />
.getOrCreate()<br />
<br />
//2.创建DataFrame<br />
//2.1先创建RDD<br />
val lines: RDD[String] = session.sparkContext.textFile("data/user.txt")<br />
//2.2对数据进行整理并关联Schema<br />
val tfBoy: RDD[Boy] = lines.map(line =&gt; {<br />
val fields = line.split(",")<br />
val name = fields(0)<br />
val age = fields(1).toInt<br />
val fv = fields(2).toDouble<br />
<em>Boy</em>(name, age, fv) //字段名称，字段的类型<br />
})<br />
<br />
//2.3将RDD关联schema，将RDD转成DataFrame<br />
//导入隐式转换<br />
import session.implicits._<br />
val df: DataFrame = tfBoy.toDF<br />
//打印DataFrame的Schema信息<br />
df.printSchema()<br />
//3.将DataFrame注册成视图（虚拟的表）<br />
df.createTempView("v_users")<br />
//4.写sql（Transformation）<br />
val df2: DataFrame = session.sql("select * from v_users order by fv desc, age asc")<br />
//5.触发Action<br />
df2.show()<br />
//6.释放资源<br />
session.stop()<br />
}<br />
}<br />
<br />
case class Boy(name: String, age: Int, fv: Double)</td>
</tr>
</tbody>
</table>

将RDD关联普通class创建DataFrame

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
object C02_DataFrameDemo2 {<br />
<br />
def main(args: Array[String]): Unit = {<br />
<br />
//1.创建SparkSession<br />
val spark = SparkSession.builder()<br />
.appName("DataFrameDemo2")<br />
.master("local[*]")<br />
.getOrCreate()<br />
<br />
//2.创建RDD<br />
val lines: RDD[String] = spark.sparkContext.textFile("data/user.txt")<br />
//2将数据封装到普通的class中<br />
val boyRDD: RDD[Boy2] = lines.map(line =&gt; {<br />
val fields = line.split(",")<br />
val name = fields(0)<br />
val age = fields(1).toInt<br />
val fv = fields(2).toDouble<br />
new Boy2(name, age, fv) //字段名称，字段的类型<br />
})<br />
//3.将RDD和Schema进行关联<br />
val df = spark.createDataFrame(boyRDD, <em>classOf</em>[Boy2])<br />
//df.printSchema()<br />
//4.使用DSL风格的API<br />
import spark.implicits._<br />
df.show()<br />
spark.stop()<br />
}<br />
}<br />
<br />
//参数前面必须有var或val<br />
//必须添加给字段添加对应的getter方法，在scala中，可以@BeanProperty注解<br />
class Boy2(<br />
@BeanProperty<br />
val name: String,<br />
@BeanProperty<br />
val age: Int,<br />
@BeanProperty<br />
val fv: Double) {<br />
<br />
}</td>
</tr>
</tbody>
</table>

|  |
|:---|
| 普通的scala class 必须在成员变量加上@BeanProperty属性，因为sparksql需要通过反射调用getter获取schema信息 |

将RDD关联java class创建DataFrame

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
object SQLDemo4 {<br />
<br />
def main(args: Array[String]): Unit = {<br />
<br />
val spark: SparkSession = SparkSession.<em>builder</em>()<br />
.appName(this.getClass.getSimpleName)<br />
.master("local[*]")<br />
.getOrCreate()<br />
<br />
val lines = spark.sparkContext.textFile("data/boy.txt")<br />
//将RDD关联的数据封装到Java的class中，但是依然是RDD<br />
val jboyRDD: RDD[JBoy] = lines.map(line =&gt; {<br />
val fields = line.split(",")<br />
new JBoy(fields(0), fields(1).toInt, fields(2).toDouble)<br />
})<br />
//强制将关联了schema信息的RDD转成DataFrame<br />
val df: DataFrame = spark.createDataFrame(jboyRDD, <em>classOf</em>[JBoy])<br />
//注册视图<br />
df.createTempView("v_boy")<br />
//写sql<br />
val df2: DataFrame = spark.sql("select name, age, fv from v_boy order by fv desc, age asc")<br />
df2.show()<br />
spark.stop()<br />
}<br />
}</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Java<br />
public class JBoy {<br />
<br />
private String name;<br />
<br />
private Integer age;<br />
<br />
private Double fv;<br />
<br />
public String getName() {<br />
return name;<br />
}<br />
<br />
public Integer getAge() {<br />
return age;<br />
}<br />
<br />
public Double getFv() {<br />
return fv;<br />
}<br />
<br />
public JBoy(String name, Integer age, Double fv) {<br />
this.name = name;<br />
this.age = age;<br />
this.fv = fv;<br />
}<br />
}</td>
</tr>
</tbody>
</table>

将RDD关联Schema创建DataFrame

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
object SQLDemo4 {<br />
<br />
def main(args: Array[String]): Unit = {<br />
<br />
val spark: SparkSession = SparkSession.<em>builder</em>()<br />
.appName(this.getClass.getSimpleName)<br />
.master("local[*]")<br />
.getOrCreate()<br />
<br />
val lines = spark.sparkContext.textFile("data/boy.txt")<br />
//将RDD关联了Schema，但是依然是RDD<br />
val rowRDD: RDD[Row] = lines.map(line =&gt; {<br />
val fields = line.split(",")<br />
<em>Row</em>(fields(0), fields(1).toInt, fields(2).toDouble)<br />
})<br />
<br />
val schema = StructType.<em>apply</em>(<br />
<em>List</em>(<br />
StructField("name", StringType),<br />
StructField("age", IntegerType),<br />
StructField("fv", DoubleType),<br />
)<br />
)<br />
val df: DataFrame = spark.createDataFrame(rowRDD, schema)<br />
//打印schema信息<br />
//df.printSchema()<br />
//注册视图<br />
df.createTempView("v_boy")<br />
//写sql<br />
val df2: DataFrame = spark.sql("select name, age, fv from v_boy order by fv desc, age asc")<br />
df2.show()<br />
spark.stop()<br />
}<br />
}</td>
</tr>
</tbody>
</table>

#### 3.1.2 从结构化文件创建DataFrame

##### (1)从csv文件（不带header）进行创建

csv文件内容：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
1,张飞,21,北京,80.0<br />
2,关羽,23,北京,82.0<br />
3,赵云,20,上海,88.6<br />
4,刘备,26,上海,83.0<br />
5,曹操,30,深圳,90.0</td>
</tr>
</tbody>
</table>

代码示例：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val df = spark.read.csv("data_ware/demodata/stu.csv")<br />
df.printSchema()<br />
df.show()</td>
</tr>
</tbody>
</table>

结果如下：

<img src="SparkSQL _assets/media/image8.png" style="width:5.90625in;height:3.57292in" />

<img src="SparkSQL _assets/media/image9.png" style="width:5.90625in;height:1.875in" />

可以看出，框架对读取进来的csv数据，自动生成的schema中，

字段名为：\_c0,\_c1,.....

字段类型全为String

不一定是符合我们需求的

##### (2)从csv文件（不带header）自定义Schema进行创建

// 创建DataFrame时，传入自定义的schema

// schema在api中用StructType这个类来描述，字段用StructField来描述

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val schema = new StructType()<br />
.add("id", DataTypes.IntegerType)<br />
.add("name", DataTypes.StringType)<br />
.add("age", DataTypes.IntegerType)<br />
.add("city", DataTypes.StringType)<br />
.add("score", DataTypes.DoubleType)<br />
​<br />
val df = spark.read.schema(schema).csv("data_ware/demodata/stu.csv")<br />
df.printSchema()<br />
df.show()</td>
</tr>
</tbody>
</table>

Schema信息：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
root<br />
|-- id: integer (nullable = true)<br />
|-- name: string (nullable = true)<br />
|-- age: integer (nullable = true)<br />
|-- city: string (nullable = true)<br />
|-- score: double (nullable = true)</td>
</tr>
</tbody>
</table>

输出数据信息：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">SQL<br />
+---+------+---+------+-----+<br />
| id|name  |age|city  |score|<br />
+---+------+---+------+-----+<br />
|  1|  张飞| 21|  北京| 80.0|<br />
|  2|  关羽| 23|  北京| 82.0|<br />
|  3|  赵云| 20|  上海| 88.6|<br />
|  4|  刘备| 26|  上海| 83.0|<br />
|  5|  曹操| 30|  深圳| 90.0|<br />
+---+------+---+------+-----+</td>
</tr>
</tbody>
</table>

##### (3)从csv文件（带header）进行创建

csv文件内容：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
id,name,age,city,score<br />
1,张飞,21,北京,80.0<br />
2,关羽,23,北京,82.0<br />
3,赵云,20,上海,88.6<br />
4,刘备,26,上海,83.0<br />
5,曹操,30,深圳,90.0</td>
</tr>
</tbody>
</table>

注意：此文件的第一行是字段描述信息，需要特别处理，否则会被当做rdd中的一行数据

代码示例：关键点设置一个header=true的参数

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val df = spark.read<br />
.option("header",true) //读取表头信息<br />
.csv("data_ware/demodata/stu.csv")<br />
df.printSchema()<br />
df.show()</td>
</tr>
</tbody>
</table>

结果如下：

root

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
|-- id: string (nullable = true)<br />
|-- name: string (nullable = true)<br />
|-- age: string (nullable = true)<br />
|-- city: string (nullable = true)<br />
|-- score: string (nullable = true)<br />
​<br />
+---+----+---+----+-----+<br />
| id|name|age|city|score|<br />
+---+----+---+----+-----+<br />
|  1|  张飞| 21|  北京| 80.0|<br />
|  2|  关羽| 23|  北京| 82.0|<br />
|  3|  赵云| 20|  上海| 88.6|<br />
|  4|  刘备| 26|  上海| 83.0|<br />
|  5|  曹操| 30|  深圳| 90.0|<br />
+---+----+---+----+-----+</td>
</tr>
</tbody>
</table>

问题：虽然字段名正确指定，但是字段类型还是无法确定，默认情况下全视作String对待，当然，可以开启一个参数 inferSchema=true 来让框架对csv中的数据字段进行合理的类型推断

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val df = spark.read<br />
.option("header",true)<br />
.option("inferSchema",true) //推断字段类型<br />
.csv("data_ware/demodata/stu.csv")<br />
df.printSchema()<br />
df.show()</td>
</tr>
</tbody>
</table>

如果推断的结果不如人意，当然可以指定自定义schema

让框架自动推断schema，效率低不建议！

##### (4)从JSON文件进行创建

准备json数据文件

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">JSON<br />
{"name":"Michael"}<br />
{"name":"Andy", "age":30}<br />
{"name":"Justin", "age":19}</td>
</tr>
</tbody>
</table>

代码示例

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">JSON<br />
val df = spark.read.json("data_ware/demodata/people.json")<br />
df.printSchema()<br />
df.show()</td>
</tr>
</tbody>
</table>

##### (5)从Parquet文件进行创建

Parquet文件是一种列式存储文件格式，文件自带schema描述信息

准备测试数据

任意拿一个dataframe，调用write.parquet()方法即可将df保存为一个parquet文件

代码示例：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val df = spark.read.parquet("data/parquet/")</td>
</tr>
</tbody>
</table>

##### (6)从orc文件进行创建

代码示例

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val df = spark.read.orc("data/orcfiles/")</td>
</tr>
</tbody>
</table>

#### 3.2.3外部存储服务创建DF

##### （1）从JDBC连接数据库服务器进行创建

实验准备

在一个mysql服务器中，创建一个数据库demo，创建一个表student，如下：

<img src="SparkSQL _assets/media/image10.png" style="width:4.91667in;height:2.08333in" />

注：要使用jdbc连接读取数据库的数据，需要引入jdbc的驱动jar包依赖

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">XML<br />
&lt;dependency&gt;<br />
&lt;groupId&gt;mysql&lt;/groupId&gt;<br />
&lt;artifactId&gt;mysql-connector-java&lt;/artifactId&gt;<br />
&lt;version&gt;8.0.30&lt;/version&gt;<br />
&lt;/dependency&gt;</td>
</tr>
</tbody>
</table>

代码示例

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val props = new Properties()<br />
props.setProperty("user","root")<br />
props.setProperty("password","root")<br />
val df = spark.read.jdbc("jdbc:mysql://localhost:3306/demo","student",props)<br />
df.show()</td>
</tr>
</tbody>
</table>

结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
+---+------+---+------+-----+<br />
| id|name  |age|city  |score|<br />
+---+------+---+------+-----+<br />
|  1|  张飞| 21|  北京| 80.0|<br />
|  2|  关羽| 23|  北京| 82.0|<br />
|  3|  赵云| 20|  上海| 88.6|<br />
|  4|  刘备| 26|  上海| 83.0|<br />
|  5|  曹操| 30|  深圳| 90.0|<br />
+---+------+---+------+-----+</td>
</tr>
</tbody>
</table>

SparkSql添加了spark-hive的依赖，并在sparkSession构造时开启enableHiveSupport后，就整合了hive的功能（通俗说，就是sparksql具备了hive的功能）；

<img src="SparkSQL _assets/media/image11.png" style="width:5.90625in;height:1.59375in" />

既然具备了hive的功能，那么就可以执行一切hive中能执行的动作：

建表

show 表

建库

show 库

alter表

……

只不过，此时看见的表是spark中集成的hive的本地元数据库中的表！

如果想让spark中集成的hive，看见你外部集群中的hive的表，只要修改配置：把spark端的hive的元数据服务地址，指向外部集群中hive的元数据服务地址；

有两种指定办法：

在spark端加入hive-site.xml ，里面配置 目标元数据库 mysql的连接信息

这会使得spark中集成的hive直接访问mysql元数据库

在spark端加入hive-site.xml ，里面配置 目标hive的元数据服务器地址

这会使得spark中集成的hive通过外部独立的hive元数据服务来访问元数据库

<img src="SparkSQL _assets/media/image12.png" style="width:5.90625in;height:2.9375in" />

##### （2）从Hive创建DataFrame

Sparksql通过spark-hive整合包，来集成hive的功能

Sparksql加载“外部独立hive”的数据，本质上是不需要“外部独立hive”参与的，因为“外部独立hive”的表数据就在hdfs中，元数据信息在mysql中

不管数据还是元数据，sparksql都可以直接去获取！

步骤：

要在工程中添加spark-hive的依赖jar以及mysql的jdbc驱动jar

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">XML<br />
&lt;dependency&gt;<br />
&lt;groupId&gt;mysql&lt;/groupId&gt;<br />
&lt;artifactId&gt;mysql-connector-java&lt;/artifactId&gt;<br />
&lt;version&gt;8.0.30&lt;/version&gt;<br />
&lt;/dependency&gt;<br />
<br />
&lt;!-- spark整合hive的依赖，即可以读取hive的源数据库，使用hive特点的sql --&gt;<br />
&lt;dependency&gt;<br />
&lt;groupId&gt;org.apache.spark&lt;/groupId&gt;<br />
&lt;artifactId&gt;spark-hive_2.12&lt;/artifactId&gt;<br />
&lt;version&gt;3.2.3&lt;/version&gt;<br />
&lt;/dependency&gt;</td>
</tr>
</tbody>
</table>

要在工程中添加hive-site.xml/core-site.xml配置文件

<img src="SparkSQL _assets/media/image13.png" style="width:5.23958in;height:2.65625in" />

创建sparksession时需要调用.enableHiveSupport( )方法

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val spark = SparkSession<br />
.builder()<br />
.appName(this.getClass.getSimpleName)<br />
.master("local[*]")<br />
 // 启用hive支持,需要调用enableHiveSupport，还需要添加一个依赖 spark-hive<br />
 // 默认sparksql内置了自己的hive<br />
 // 如果程序能从classpath中加载到hive-site配置文件，那么它访问的hive元数据库就不是本地内置的了，而是配置中所指定的元数据库了<br />
 // 如果程序能从classpath中加载到core-site配置文件，那么它访问的文件系统也不再是本地文件系统了，而是配置中所指定的hdfs文件系统了<br />
.enableHiveSupport()<br />
.getOrCreate()</td>
</tr>
</tbody>
</table>

加载hive中的表

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val df = spark.sql("select * from t1")</td>
</tr>
</tbody>
</table>

注意点：如果自己也用dataframe注册了一个同名的视图，那么这个视图名会替换掉hive的表

##### （3）从Hbase创建DataFrame

其实，sparksql可以连接任意外部数据源（只要有对应的“连接器”即可）

Sparksql对hbase是有第三方连接器（华为）的，但是久不维护！

建议用hive作为连接器（hive可以访问hbase，而sparksql可以集成hive）

在hbase中建表

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Shell<br />
create 'doitedu_stu','f'</td>
</tr>
</tbody>
</table>

插入数据到hbase表

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Shell<br />
put 'doitedu_stu','001','f:name','zhangsan'<br />
put 'doitedu_stu','001','f:age','26'<br />
put 'doitedu_stu','001','f:gender','m'<br />
put 'doitedu_stu','001','f:salary','28000'<br />
put 'doitedu_stu','002','f:name','lisi'<br />
put 'doitedu_stu','002','f:age','22'<br />
put 'doitedu_stu','002','f:gender','m'<br />
put 'doitedu_stu','002','f:salary','26000'<br />
put 'doitedu_stu','003','f:name','wangwu'<br />
put 'doitedu_stu','003','f:age','21'<br />
put 'doitedu_stu','003','f:gender','f'<br />
put 'doitedu_stu','003','f:salary','24000'<br />
put 'doitedu_stu','004','f:name','zhaoliu'<br />
put 'doitedu_stu','004','f:age','22'<br />
put 'doitedu_stu','004','f:gender','f'<br />
put 'doitedu_stu','004','f:salary','25000'</td>
</tr>
</tbody>
</table>

创建hive外部表映射hbase中的表

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">SQL<br />
CREATE EXTERNAL TABLE doitedu_stu<br />
(<br />
id       string ,<br />
name     string ,<br />
age       int    ,<br />
gender   string ,<br />
salary    double  <br />
)  <br />
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'<br />
WITH SERDEPROPERTIES ( 'hbase.columns.mapping'=':key,f:name,f:age,f:gender,f:salary')<br />
TBLPROPERTIES ( 'hbase.table.name'='default:doitedu_stu')<br />
;</td>
</tr>
</tbody>
</table>

工程中放置hbase-site.xml配置文件

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">XML<br />
&lt;configuration&gt;<br />
   &lt;property&gt;<br />
       &lt;name&gt;hbase.rootdir&lt;/name&gt;<br />
       &lt;value&gt;hdfs://doit01:8020/hbase&lt;/value&gt;<br />
   &lt;/property&gt;<br />
​<br />
   &lt;property&gt;<br />
       &lt;name&gt;hbase.cluster.distributed&lt;/name&gt;<br />
       &lt;value&gt;true&lt;/value&gt;<br />
   &lt;/property&gt;<br />
​<br />
   &lt;property&gt;<br />
       &lt;name&gt;hbase.zookeeper.quorum&lt;/name&gt;<br />
       &lt;value&gt;doit01:2181,doit02:2181,doit03:2181&lt;/value&gt;<br />
   &lt;/property&gt;<br />
​<br />
   &lt;property&gt;<br />
       &lt;name&gt;hbase.unsafe.stream.capability.enforce&lt;/name&gt;<br />
       &lt;value&gt;false&lt;/value&gt;<br />
   &lt;/property&gt;<br />
&lt;/configuration&gt;</td>
</tr>
</tbody>
</table>

​

工程中添加hive-hbase-handler连接器依赖

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">XML<br />
&lt;dependency&gt;<br />
&lt;groupId&gt;org.apache.hive&lt;/groupId&gt;<br />
&lt;artifactId&gt;hive-hbase-handler&lt;/artifactId&gt;<br />
&lt;version&gt;2.3.7&lt;/version&gt;<br />
&lt;exclusions&gt;<br />
&lt;exclusion&gt;<br />
&lt;groupId&gt;org.apache.hadoop&lt;/groupId&gt;<br />
&lt;artifactId&gt;hadoop-common&lt;/artifactId&gt;<br />
&lt;/exclusion&gt;<br />
&lt;/exclusions&gt;<br />
&lt;/dependency&gt;</td>
</tr>
</tbody>
</table>

​

以读取hive表的方式直接读取即可

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
spark.sql("select * from doitedu_stu")</td>
</tr>
</tbody>
</table>

   

### 3.2 输出DF的各种方式

#### 3.2.1 展现在控制台

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
df.show()<br />
df.show(10) //输出10行<br />
df.show(10,false) // 不要截断列</td>
</tr>
</tbody>
</table>

#### 3.2.2 保存为文件

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
object Demo17_SaveDF {<br />
​<br />
def main(args: Array[String]): Unit = {<br />
​<br />
  val spark = SparkUtil.getSpark()<br />
​<br />
  val df = spark.read.option("header",true).csv("data/stu2.csv")<br />
​<br />
  val res = df.where("id&gt;3").select("id","name")<br />
​<br />
  // 展示结果<br />
  res.show(10,false)<br />
}</td>
</tr>
</tbody>
</table>

​

保存结果为文件： parquet,json,csv,orc,textfile

文本文件是自由格式，框架无法判断该输出什么样的形式

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
res.write.parquet("out/parquetfile/")<br />
​<br />
res.write.csv("out/csvfile")<br />
​<br />
res.write.orc("out/orcfile")<br />
​<br />
res.write.json("out/jsonfile")</td>
</tr>
</tbody>
</table>

​

要将df输出为普通文本文件，则需要将df变成一个列

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
res.selectExpr("concat_ws('\001',id,name)")<br />
.write.text("out/textfile")</td>
</tr>
</tbody>
</table>

#### 3.2.3 保存到RDBMS

将dataframe写入mysql的表

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
// 将dataframe通过jdbc写入mysql<br />
val props = new Properties()<br />
props.setProperty("user","root")<br />
props.setProperty("password","root")<br />
<br />
// 可以通过SaveMode来控制写入模式：SaveMode.Append/Ignore/Overwrite/ErrorIfExists(默认)<br />
res.write.mode(SaveMode.Append).jdbc("jdbc:mysql://localhost:3306/demo?characterEncoding=utf8","res",props)</td>
</tr>
</tbody>
</table>

#### 3.2.4 写入hive

开启spark的hive支持

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val spark = SparkSession<br />
 .builder()<br />
 .appName("")<br />
 .master("local[*]")<br />
 .enableHiveSupport()<br />
 .getOrCreate()</td>
</tr>
</tbody>
</table>

放入配置文件

<img src="SparkSQL _assets/media/image14.png" style="width:3.48958in;height:1.33333in" />

写代码：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
// 将dataframe写入hive,saveAsTable就是保存为hive的表<br />
// 前提，spark要开启hiveSupport支持，spark-hive的依赖，hive的配置文件<br />
res.write.saveAsTable("res")</td>
</tr>
</tbody>
</table>

#### 3.2.5 DF输出时的分区操作

Hive中对表数据的存储，可以将数据分为多个子目录！

比如：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Shell<br />
```
create table tx(id int,name string) partitioned by (city string);<br />
load data inpath ‘/data/1.dat’ into table tx partition(city=”beijing”)<br />
load data inpath ‘/data/2.dat’ into table tx partition(city=”shanghai”)</td>
</tr>
</tbody>
```
</table>

Hive的表tx的目录结构如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
/user/hive/warehouse/tx/<br />
                   city=beijing/1.dat<br />
                   city=shanghai/2.dat</td>
</tr>
</tbody>
</table>

查询的时候，分区标识字段，可以看做表的一个字段来用

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">SQL<br />
```
Select * from tx   where city=’shanghai’</td>
</tr>
</tbody>
```
</table>

那么，sparksql既然是跟hive兼容的，必然也有对分区存储支持的机制！

能识别解析分区

有如下数据结构形式：

<img src="SparkSQL _assets/media/image15.png" style="width:5.90625in;height:1.5625in" />

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
* sparksql对分区机制的支持<br />
* 识别已存在分区结构 /aaa/city=a; /aaa/city=b;<br />
* 会将所有子目录都理解为数据内容，会将子目录中的city理解为一个字段<br />
*/<br />
spark.read.csv("/aaa").show()</td>
</tr>
</tbody>
</table>

能将数据按分区机制输出

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
* sparksql对分区机制的支持<br />
* 将dataframe存储为分区结构<br />
*/<br />
val dfp = spark.read.option("header",true).csv("data/stu2.csv")<br />
dfp.write.partitionBy("city").csv("/bbb")<br />
​<br />
​<br />
/**<br />
* 将数据分区写入hive<br />
* 注意：写入hive的默认文件格式是parquet<br />
*/<br />
dfp.write.partitionBy("sex").saveAsTable("res_p")</td>
</tr>
</tbody>
</table>

输入结构如下：

<img src="SparkSQL _assets/media/image16.png" style="width:5.90625in;height:1.69792in" />

### 3.3 DF数据运算操作

#### 3.3.1 纯SQL操作

核心要义：将DataFrame 注册为一个临时视图view，然后就可以针对view直接执行各种sql

临时视图有两种：session级别视图，global级别视图；

session级别视图是Session范围内有效的，Session退出后，表就失效了；

全局视图则在application级别有效；

注意使用全局表时需要全路径访问：global_temp.people

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
// application全局有效<br />
df.createGlobalTempView("stu")<br />
spark.sql(<br />
 """<br />
   |select * from <u>global_temp</u>.stu a order by a.score desc<br />
 """.stripMargin)<br />
  .show()<br />
​</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
// session有效<br />
df.createTempView("s")<br />
spark.sql(<br />
 """<br />
   |select * from s order by score<br />
 """.stripMargin)<br />
.show()</td>
</tr>
</tbody>
</table>

​

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val spark2 = spark.newSession()<br />
// 全局有效的view可以在session2中访问<br />
spark2.sql("select id,name from global_temp.stu").show()<br />
// session有效的view不能在session2中访问<br />
spark2.sql("select id,name from s").show()</td>
</tr>
</tbody>
</table>

以上只是对语法的简单示例，可以扩展到任意复杂的sql

挑战一下 ?

求出每个城市中，分数最高的学生信息；

Go go go !

#### 3.3.2 DSL风格API(TableApi)语法

DSL风格API，就是用编程api的方式，来实现sql语法

DSL：特定领域语言

dataset的tableApi有一个特点：运算后返回值必回到dataframe

因为select后，得到的结果，无法预判返回值的具体类型，只能用通用的Row封装

数据准备

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val df = spark.read<br />
.option("header", true)<br />
.option("inferSchema", true)<br />
.csv("data_ware/demodata/stu.csv")</td>
</tr>
</tbody>
</table>

​

##### （1）基本select及表达式

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
 * 逐行运算<br />
 */<br />
// 使用字符串表达"列"<br />
df.select("id","name").show()<br />
​<br />
// 如果要用字符串形式表达sql表达式，应该使用selectExpr方法<br />
df.selectExpr("id+1","upper(name)").show<br />
// select方法中使用字符串sql表达式，会被视作一个列名从而出错<br />
// df.select("id+1","upper(name)").show()<br />
​<br />
import spark.implicits._<br />
// 使用$符号创建Column对象来表达"列"<br />
df.select($"id",$"name").show()<br />
​<br />
// 使用单边单引号创建Column对象来表达"列"<br />
df.select('id,'name).show()<br />
​<br />
// 使用col函数来创建Column对象来表达"列"<br />
import org.apache.spark.sql.functions._<br />
df.select(col("id"),col("name")).show()<br />
​<br />
// 使用Dataframe的apply方法创建Column对象来表达列<br />
df.select(df("id"),df("name")).show()<br />
​<br />
// 对Column对象直接调用Column的方法，或调用能生成Column对象的functions来实现sql中的运算表达式<br />
df.select('id.plus(2).leq("4").as("id2"),upper('name)).show()<br />
df.select('id+2 &lt;= 4 as "id2",upper('name)).show()</td>
</tr>
</tbody>
</table>

##### （3）字段重命名

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
 * 字段重命名<br />
 */<br />
// 对column对象调用as方法<br />
df.select('id as "id2",$"name".as("n2"),col("age") as "age2").show()<br />
​<br />
// 在selectExpr中直接写sql的重命名语法<br />
df.selectExpr("cast(id as string) as id2","name","city").show()<br />
​<br />
// 对dataframe调用withColumnRenamed方法对指定字段重命名<br />
df.select("id","name","age").withColumnRenamed("id","id2").show()<br />
​<br />
// 对dataframe调用toDF对整个字段名全部重设<br />
df.toDF("id2","name","age","city2","score").show()</td>
</tr>
</tbody>
</table>

##### （2）条件过滤

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
 * 逐行过滤<br />
 */<br />
df.where("id&gt;4 and score&gt;95")<br />
df.where('id &gt; 4 and 'score &gt; 95).select("id","name","age").show()</td>
</tr>
</tbody>
</table>

##### （4）分组聚合

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
 * 分组聚合<br />
 */<br />
df.groupBy("city").count().show()<br />
df.groupBy("city").min("score").show()<br />
df.groupBy("city").max("score").show()<br />
df.groupBy("city").sum("score").show()<br />
df.groupBy("city").avg("score").show()<br />
​<br />
df.groupBy("city").agg(("score","max"),("score","sum")).show()<br />
df.groupBy("city").agg("score"-&gt;"max","score"-&gt;"sum").show()</td>
</tr>
</tbody>
</table>

##### 3.3.2.1 子查询

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
 * 子查询<br />
 * 相当于:<br />
 * select<br />
 * *<br />
 * from<br />
 * (<br />
 *   select<br />
 *   city,sum(score) as score<br />
 *   from stu<br />
 *   group by city<br />
 * ) o<br />
 * where score&gt;165<br />
 */<br />
df.groupBy("city")<br />
.agg(sum("score") as "score")<br />
.where("score &gt; 165")<br />
.select("city", "score")<br />
.show()</td>
</tr>
</tbody>
</table>

##### 3.3.2.2 Join关联查询

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
package cn.doitedu.sparksql<br />
​<br />
import org.apache.spark.sql.{DataFrame, SparkSession}<br />
​<br />
/**<br />
 * 用 DSL风格api来对dataframe进行运算<br />
*/<br />
object Demo14_DML_DSLapi {<br />
 def main(args: Array[String]): Unit = {<br />
​<br />
   val spark = SparkUtil.getSpark()<br />
   import spark.implicits._<br />
​<br />
   val df = spark.read.option("header",true).csv("data/stu2.csv")<br />
   val df2 = spark.read.option("header",true).csv("data/stu22.csv")<br />
​<br />
   /**<br />
     * SQL中都有哪些运算？<br />
     *   1. 查询字段（id)<br />
     *   2. 查询表达式（算术运算，函数 age+10, upper(name)）<br />
     *   3. 过滤<br />
     *   4. 分组聚合<br />
     *   5. 子查询<br />
     *   6. 关联查询<br />
     *   7. union查询<br />
     *   8. 窗口分析<br />
     *   9. 排序<br />
     */<br />
​<br />
   //selectOp(spark,df)<br />
   //whereOp(spark,df)<br />
   //groupbyOp(spark,df)<br />
   joinOp(spark,df,df2)<br />
​<br />
   spark.close()<br />
}<br />
<br />
 /**<br />
   * 关联查询<br />
   * @param spark<br />
   * @param df1<br />
   */<br />
 def joinOp(spark:SparkSession,df1:DataFrame,df2:DataFrame): Unit ={<br />
​<br />
   // 笛卡尔积<br />
   //df1.crossJoin(df2).show()<br />
​<br />
   // 给join传入一个连接条件； 这种方式，要求，你的join条件字段在两个表中都存在且同名<br />
   df1.join(df2,"id").show()<br />
​<br />
   // 传入多个join条件列，要求两表中这多个条件列都存在且同名<br />
   df1.join(df2,Seq("id","sex")).show()<br />
​<br />
​<br />
   // 传入一个自定义的连接条件表达式<br />
   df1.join(df2,df1("id") + 1 === df2("id")).show()<br />
​<br />
​<br />
   // 还可以传入join方式类型： inner(默认)， left ,right, full ,left_semi, left_anti<br />
   df1.join(df2,df1("id")+1 === df2("id"),"left").show()<br />
   df1.join(df2,Seq("id"),"right").show()<br />
​<br />
​<br />
   /**<br />
     * 总结:<br />
     * join方式：   joinType: String<br />
     * join条件：<br />
     * 可以直接传join列名：usingColumn/usingColumns : Seq(String) 注意：右表的join列数据不会出现结果中<br />
     *  可以传join自定义表达式： Column.+(1) === Column     df1("id")+1 === df2("id")<br />
     */<br />
}<br />
}</td>
</tr>
</tbody>
</table>

​

##### 3.3.2.3 Union操作

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
Sparksql中的union，其实是union all<br />
df.union(df).show()</td>
</tr>
</tbody>
</table>

##### 3.3.2.4 窗口分析函数调用

测试数据：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
id,name,age,sex,city,score<br />
1,张飞,21,M,北京,80<br />
2,关羽,23,M,北京,82<br />
7,周瑜,24,M,北京,85<br />
3,赵云,20,F,上海,88<br />
4,刘备,26,M,上海,83<br />
8,孙权,26,M,上海,78<br />
5,曹操,30,F,深圳,90.8<br />
6,孔明,35,F,深圳,77.8<br />
9,吕布,28,M,深圳,98</td>
</tr>
</tbody>
</table>

求每个城市中成绩最高的两个人的详细信息，如果用sql写：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">SQL<br />
select<br />
id,name,age,sex,city,score<br />
from<br />
 (<br />
    select<br />
      id,name,age,sex,city,score,<br />
      row_number() over(partition by city order by score desc) as rn<br />
    from t<br />
 ) o<br />
where rn&lt;=2</td>
</tr>
</tbody>
</table>

DSL风格的API实现：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
package cn.doitedu.sparksql<br />
​<br />
import org.apache.spark.sql.expressions.Window<br />
​<br />
/**<br />
*   用dsl风格api实现sql中的窗口分析函数<br />
*/<br />
object Demo15_DML_DSLAPI_WINDOW {<br />
​<br />
def main(args: Array[String]): Unit = {<br />
​<br />
  val spark = SparkUtil.getSpark()<br />
​<br />
  val df = spark.read.option("header",true).csv("data/stu2.csv")<br />
​<br />
   import spark.implicits._<br />
   import org.apache.spark.sql.functions._<br />
​<br />
​<br />
  val window = Window.partitionBy('city).orderBy('score.desc)<br />
​<br />
  df.select('id,'name,'age,'sex,'city,'score,row_number().over(window) as "rn")<br />
     .where('rn &lt;= 2)<br />
    .drop("rn") // 最后结果中不需要rn列，可以drop掉这个列<br />
    .select('id,'name,'age,'sex,'city,'score) // 或者用select指定你所需要的列<br />
    .show()<br />
<br />
  spark.close()<br />
}<br />
}</td>
</tr>
</tbody>
</table>

Dataset提供与RDD类似的编程算子，即map/flatMap/reduceByKey等等，不过本方式使用略少：

如果方便用sql表达的逻辑，首选sql

如果不方便用sql表达，则可以把Dataset转成RDD后使用 RDD的算子

直接在Dataset上调用类似RDD风格算子的代码示例如下：

 

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
 /**<br />
    * 四、 dataset/dataframe 调 RDD算子<br />
    *<br />
    * dataset调rdd算子，返回的还是dataset[U], 不过需要对应的 Encoder[U]<br />
    *<br />
    */<br />
   val ds4: Dataset[(Int, String, Int)] = ds2.map(p =&gt; (p.id, p.name, p.age + 10))   // 元组有隐式Encoder自动传入<br />
   val ds5: Dataset[JavaPerson] = ds2.map(p =&gt; new JavaPerson(p.id,p.name,p.age*2))(Encoders.bean(classOf[JavaPerson])) // Java类没有自动隐式Encoder，需要手动传<br />
​<br />
   val ds6: Dataset[Map[String, String]] = ds2.map(per =&gt; Map("name" -&gt; per.name, "id" -&gt; (per.id+""), "age" -&gt; (per.age+"")))<br />
   ds6.printSchema()<br />
   /**<br />
    * root<br />
        |-- value: map (nullable = true)<br />
        |   |-- key: string<br />
        |   |-- value: string (valueContainsNull = true)<br />
    */<br />
   // 从ds6中查询每个人的姓名<br />
   ds6.selectExpr("value['name']")<br />
​<br />
​<br />
   // dataframe上调RDD算子，就等价于 dataset[Row]上调rdd算子<br />
   val ds7: Dataset[(Int, String, Int)] = frame.map(row=&gt;{<br />
     val id: Int = row.getInt(0)<br />
     val name: String = row.getAs[String]("name")<br />
     val age: Int = row.getAs[Int]("age")<br />
    (id,name,age)<br />
  })<br />
   <br />
   // 利用模式匹配从row中抽取字段数据<br />
   val ds8: Dataset[Per] = frame.map({<br />
     case Row(id:Int,name:String,age:Int) =&gt; Per(id,name,age*10)<br />
  })</td>
</tr>
</tbody>
</table>

### 3.5 Dataset与RDD混编

#### 3.5.1 DataSet和Dataframe的区别

狭义上，Dataset中装的是用户自定义的类型

那么在抽取数据时，比较方便 stu.id 且类型会得到编译时检查

狭义上，dataframe中装的是Row（框架内置的一个通用类型）

那么在抽取数据时，不太方便，得通过脚标，或者字段名，而且还得强转

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val x:Any =  row.get(1)<br />
val x:Double = row.getDouble(1)<br />
val x:Double = row.getAs[Double](“salary”)</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
* dataset存在的意义？<br />
* 意义要从它的特点说起：<br />
* ds的特点是，可以存储各种自定义类型，自定义类型中，各字段是有类型约束（所以<strong>ds是强类型约束</strong>的）<br />
* df只能存储row类型，而row类型中的字段是没有“类型约束“，全是any（所以<strong>df是弱类型约束</strong>的）<br />
*/<br />
ds.map(bean =&gt; {<br />
// val id:String = bean.id // 提取数据时不会产生类型匹配错误，编译时就会检查<br />
​<br />
})<br />
​<br />
​<br />
val _df: Dataset[Row] = ds.toDF()<br />
_df.map(row =&gt; {<br />
val name: Int = row.getInt(1) // 明明类型不匹配，但是编译时无法检查，运行时才会抛异常<br />
})</td>
</tr>
</tbody>
</table>

#### 3.5.3 DataFrame/dataset转成RDD后取数

要义：有些运算场景下，通过SQL语法实现计算逻辑比较困难，可以将DataFrame转成RDD算子来操作，而DataFrame中的数据是以RDD\[Row\]类型封装的，因此，要对DataFrame进行RDD算子操作，只需要掌握如何从Row中解构出数据即可

示例数据stu.csv

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
id,name,age,city,score<br />
1,张飞,21,北京,80.0<br />
2,关羽,23,北京,82.0<br />
3,赵云,20,上海,88.6<br />
4,刘备,26,上海,83.0<br />
5,曹操,30,深圳,90.0</td>
</tr>
</tbody>
</table>

##### （1）从Row中取数方式1：索引号

<img src="SparkSQL _assets/media/image17.png" style="width:5.90625in;height:5.3125in" />

示例代码

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rdd: RDD[Row] = df.rdd<br />
rdd.map(row=&gt;{<br />
val id = row.get(0).asInstanceOf[Int]<br />
val name = row.getString(1)<br />
 (id,name)<br />
}).take(10).foreach(println)</td>
</tr>
</tbody>
</table>

##### （2）从Row中取数方式2：字段名

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
rdd.map(row=&gt;{<br />
val id = row.getAs[Int]("id")<br />
val name = row.getAs[String]("name")<br />
val age = row.getAs[Int]("age")<br />
val city = row.getAs[String]("city")<br />
val score = row.getAs[Double]("score")<br />
 (id,name,age,city,score)<br />
}).take(10).foreach(println)</td>
</tr>
</tbody>
</table>

##### （3）从Row中取数方式3：模式匹配

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
rdd.map({<br />
 case Row(id: Int, name: String, age: Int, city: String, score: Double)<br />
 =&gt; {<br />
  // do anything<br />
   (id,name,age,city,score)<br />
 }<br />
}).take(10).foreach(println)</td>
</tr>
</tbody>
</table>

##### （4）完整示例

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
package cn.doitedu.sparksql<br />
​<br />
import org.apache.spark.rdd.RDD<br />
import org.apache.spark.sql.{Dataset, Row}<br />
import org.apache.spark.sql.types.{DoubleType, IntegerType, StringType, StructField, StructType}<br />
​<br />
/**<br />
*   有些场景下，逻辑不太方便用sql去实现，可能需要将dataframe退化成RDD来计算<br />
*   示例需求： 求每种性别的成绩总和<br />
*/<br />
object Demo16_DML_RDD {<br />
def main(args: Array[String]): Unit = {<br />
​<br />
  val spark = SparkUtil.getSpark()<br />
  // val schema = new StructType(Array(StructField("id",IntegerType),StructField("name",StringType)))<br />
​<br />
  // val schema = new StructType((StructField("id",IntegerType):: StructField("name",StringType) :: Nil).toArray)<br />
​<br />
  val schema = new StructType()<br />
     .add("id",IntegerType)<br />
     .add("name",StringType)<br />
     .add("age",IntegerType)<br />
     .add("sex",StringType)<br />
     .add("city",StringType)<br />
     .add("score",DoubleType)<br />
​<br />
  val df = spark.read.schema(schema).option("header",true).csv("data/stu2.csv")<br />
​<br />
  // 可以直接在dataframe上用map等rdd算子<br />
  // 框架会把算子返回的结果RDD 再转回dataset，需要一个能对RDD[T]进行解析的Encoder[T]才行<br />
  // 好在大部分T类型都可以有隐式的Encoder来支持<br />
   import spark.implicits._<br />
  val ds2: Dataset[(Int, String)] = df.map(row=&gt;{<br />
    val id = row.getAs[Int]("id")<br />
    val name = row.getAs[String]("name")<br />
     (id,name)<br />
   })<br />
​<br />
​<br />
  // dataframe中取出rdd后，就是一个RDD[Row]<br />
  val rd: RDD[Row] = df.rdd<br />
  // 从Row中取数据，就可以变成任意你想要的类型<br />
  val rdd2: RDD[(Int, String, Int, String, String, Double)] = rd.map(row=&gt;{<br />
​<br />
    // dataframe是一种弱类型结构（在编译时无法检查类型，因为数据被装在了一个array[any]中）<br />
    // val id = row.getDouble(1) // 如果类型取错，编译时是无法检查的，运行时才会报错<br />
​<br />
    // 可以根据字段的脚标去取<br />
    val id: Int = row.getInt(0)<br />
    val name: String = row.getString(1)<br />
    val age: Int = row.getAs[Int](2)<br />
​<br />
    // 可以根据字段名称去取<br />
    val sex: String = row.getAs[String]("sex")<br />
    val city: String = row.getAs[String]("city")<br />
    val score: Double = row.getAs[Double]("score")<br />
​<br />
     (id,name,age,sex,city,score)<br />
   })<br />
​<br />
​<br />
   /**<br />
    * 用模式匹配从Row中抽取数据<br />
    * 效果跟上面的方法是一样的，但是更简洁！<br />
    */<br />
  val rdd22 = rd.map({<br />
     case Row(id:Int,name:String,age:Int,sex:String,city:String,score:Double)=&gt;{<br />
       (id,name,age,sex,city,score)<br />
     }<br />
   })<br />
​<br />
  // 后续就跟dataframe没关系了，跟以前的rdd是一样的了<br />
  val res: RDD[(String, Double)] = rdd22.groupBy(tp=&gt;tp._4).mapValues(iter=&gt;{<br />
    iter.map(_._6).sum<br />
   })<br />
​<br />
  res.foreach(println)<br />
​<br />
  spark.close()<br />
 }<br />
}</td>
</tr>
</tbody>
</table>

#### 3.5.4从RDD创建DataFrame

**准备测试用的数据和RDD**

后续示例都起源于如下RDD

数据文件：doit_stu.txt

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
1,张飞,21,北京,80.0<br />
2,关羽,23,北京,82.0<br />
3,赵云,20,上海,88.6<br />
4,刘备,26,上海,83.0<br />
5,曹操,30,深圳,90.0</td>
</tr>
</tbody>
</table>

创建RDD：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rdd:RDD[String] = spark.sparkContext.textFile("data_ware/demodata/stu.txt")</td>
</tr>
</tbody>
</table>

##### （1）从RDD[Case class类]创建DataFrame

注：定义一个case class来封装数据，如下，Stu是一个case class类

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rdd:RDD[String] = spark.sparkContext.textFile("data_ware/demodata/stu.txt")</td>
</tr>
</tbody>
</table>

示例代码：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rddStu: RDD[Stu]  = rdd<br />
 // 切分字段<br />
.map(_.split(","))<br />
 // 将每一行数据变形成一个多元组tuple<br />
.map(arr =&gt; Stu(arr(0).toInt, arr(1), arr(2).toInt, arr(3), arr(4).toDouble))<br />
// 创建DataFrame<br />
val df = spark.createDataFrame(rddStu)<br />
df.show()</td>
</tr>
</tbody>
</table>

结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
+---+-------+---+-----+-----+<br />
| id|name   |age|city |score|<br />
+---+-------+---+-----+-----+<br />
|  1|  张飞| 21|  北京| 80.0|<br />
|  2|  关羽| 23|  北京| 82.0|<br />
|  3|  赵云| 20|  上海| 88.6|<br />
|  4|  刘备| 26|  上海| 83.0|<br />
|  5|  曹操| 30|  深圳| 90.0|<br />
+---+------+---+------+-----+</td>
</tr>
</tbody>
</table>

**可以发现，框架成功地从case class的类定义中推断出了数据的schema：字段类型和字段名称**

**Schema获取手段：反射**

当然，还有更简洁的方式，利用框架提供的隐式转换

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
// 更简洁办法<br />
import spark.implicits._<br />
val df = rddStu.toDF</td>
</tr>
</tbody>
</table>

##### （2）从RDD[Tuple]创建DataFrame

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rddTuple: RDD[(Int, String, Int, String, Double)] = rdd<br />
 // 切分字段<br />
.map(_.split(","))<br />
 // 将每一行数据变形成一个多元组tuple<br />
.map(arr =&gt; (arr(0).toInt, arr(1), arr(2).toInt, arr(3), arr(4).toDouble))<br />
​<br />
//创建DataFrame<br />
val df = spark.createDataFrame(rddTuple)<br />
df.printSchema() // 打印schema信息<br />
df.show()</td>
</tr>
</tbody>
</table>

结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
root<br />
|-- _1: integer (nullable = false)<br />
|-- _2: string (nullable = true)<br />
|-- _3: integer (nullable = false)<br />
|-- _4: string (nullable = true)<br />
|-- _5: double (nullable = false)<br />
<br />
<br />
+---+-----+---+---+-----+<br />
| _1| _2  | _3| _4|  _5 |<br />
+---+-----+---+---+-----+<br />
|  1| 张飞| 21| 北京|80.0|<br />
|  2| 关羽| 23| 北京|82.0|<br />
|  3| 赵云| 20| 上海|88.6|<br />
|  4| 刘备| 26| 上海|83.0|<br />
|  5| 曹操| 30| 深圳|90.0|<br />
+---+---+---+---+----+</td>
</tr>
</tbody>
</table>

从结果中可以发现一个问题：框架从tuple元组结构中，对schema的推断，也是成功的，只是字段名是tuple中的数据访问索引。

当然，还有更简洁的方式，利用框架提供的隐式转换可以直接调用toDF创建，并指定字段名

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
// 更简洁办法<br />
import spark.implicits._<br />
val df2 = rddTuple.toDF("id","name","age","city","score")</td>
</tr>
</tbody>
</table>

##### （3）从RDD[JavaBean]创建DataFrame

注：此处所说的Bean，指的是用java定义的bean

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Java<br />
public class Stu2 {<br />
   private int id;<br />
   private String name;<br />
   private int age;<br />
   private String city;<br />
   private double score;<br />
​<br />
   public Stu2(int id, String name, int age, String city, double score) {<br />
       this.id = id;<br />
       this.name = name;<br />
       this.age = age;<br />
       this.city = city;<br />
       this.score = score;<br />
  }<br />
​<br />
   public int getId() {<br />
       return id;<br />
  }<br />
​<br />
   public void setId(int id) {}</td>
</tr>
</tbody>
</table>

示例代码：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rddBean: RDD[Stu2] = rdd<br />
 // 切分字段<br />
.map(_.split(","))<br />
 // 将每一行数据变形成一个JavaBean<br />
.map(arr =&gt; new Stu2(arr(0).toInt,arr(1),arr(2).toInt,arr(3),arr(4).toDouble))<br />
val df = spark.createDataFrame(rddBean,classOf[Stu2])<br />
df.show()</td>
</tr>
</tbody>
</table>

结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
+---+-----+---+-----+----+<br />
|age|city | id|name |score|<br />
+---+-----+---+-----+----+<br />
|  1| 张飞| 21| 北京|80.0|<br />
|  2| 关羽| 23| 北京|82.0|<br />
|  3| 赵云| 20| 上海|88.6|<br />
|  4| 刘备| 26| 上海|83.0|<br />
|  5| 曹操| 30| 深圳|90.0|<br />
+---+-----+---+-----+----+</td>
</tr>
</tbody>
</table>

注：RDD\[JavaBean\]在spark.implicits.\_中没有toDF的支持

##### （4）从RDD[普通Scala类]中创建DataFrame

注：此处的普通类指的是scala中定义的非case class的类

框架在底层将其视作java定义的标准bean类型来处理

而scala中定义的普通bean类，不具备字段的java标准getters和setters，因而会处理失败

可以如下处理来解决

普通scala bean类定义：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
class Stu3(<br />
           @BeanProperty<br />
           val id: Int,<br />
           @BeanProperty<br />
           val name: String,<br />
           @BeanProperty<br />
           val age: Int,<br />
           @BeanProperty<br />
           val city: String,<br />
           @BeanProperty<br />
           val score: Double)</td>
</tr>
</tbody>
</table>

示例代码：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rddStu3: RDD[Stu3] = rdd<br />
 // 切分字段<br />
.map(_.split(","))<br />
 // 将每一行数据变形成一个普通Scala对象<br />
.map(arr =&gt; new Stu3(arr(0).toInt, arr(1), arr(2).toInt, arr(3), arr(4).toDouble))<br />
val df = spark.createDataFrame(rddStu3, classOf[Stu3])<br />
df.show()</td>
</tr>
</tbody>
</table>

##### （5）从RDD[Row]中创建DataFrame

注：DataFrame中的数据，本质上还是封装在RDD中，而RDD\[ T \]总有一个T类型，DataFrame内部的RDD中的元素类型T即为框架所定义的Row类型；

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rddRow = rdd<br />
 // 切分字段<br />
.map(_.split(","))<br />
 // 将每一行数据变形成一个Row对象<br />
.map(arr =&gt; Row(arr(0).toInt, arr(1), arr(2).toInt, arr(3), arr(4).toDouble))<br />
​<br />
val schema = new StructType()<br />
.add("id", DataTypes.IntegerType)<br />
.add("name", DataTypes.StringType)<br />
.add("age", DataTypes.IntegerType)<br />
.add("city", DataTypes.StringType)<br />
.add("score", DataTypes.DoubleType)<br />
​<br />
val df = spark.createDataFrame(rddRow,schema)<br />
df.show()</td>
</tr>
</tbody>
</table>

##### （6）从RDD[set/seq/map]中创建DataFrame

版本2.2.0，新增了对SET/SEQ的编解码支持

版本2.3.0，新增了对Map的编解码支持

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
object Demo7_CreateDF_SetSeqMap {<br />
​<br />
 def main(args: Array[String]): Unit = {<br />
​<br />
   val spark = SparkSession.builder().appName("").master("local[*]").getOrCreate()<br />
​<br />
   val seq1 = Seq(1,2,3,4)<br />
   val seq2 = Seq(11,22,33,44)<br />
​<br />
   val rdd: RDD[Seq[Int]] = spark.sparkContext.parallelize(List(seq1,seq2))<br />
​<br />
   import spark.implicits._<br />
   val df = rdd.toDF()<br />
​<br />
   df.printSchema()<br />
   df.show()<br />
​<br />
​<br />
   df.selectExpr("value[0]","size(value)").show()<br />
​<br />
​<br />
   /**<br />
     * set类型数据rdd的编解码<br />
     */<br />
   val set1 = Set("a","b")<br />
   val set2 = Set("c","d","e")<br />
   val rdd2: RDD[Set[String]] = spark.sparkContext.parallelize(List(set1,set2))<br />
​<br />
   val df2 = rdd2.toDF("members")<br />
   df2.printSchema()<br />
   df2.show()<br />
​<br />
​<br />
   /**<br />
     * map类型数据rdd的编解码<br />
     */<br />
​<br />
   val map1 = Map("father"-&gt;"mayun","mother"-&gt;"tangyan")<br />
   val map2 = Map("father"-&gt;"huateng","mother"-&gt;"yifei","brother"-&gt;"sicong")<br />
   val rdd3: RDD[Map[String, String]] = spark.sparkContext.parallelize(List(map1,map2))<br />
​<br />
   val df3 = rdd3.toDF("jiaren")<br />
   df3.printSchema()<br />
   df3.show()<br />
​<br />
   df3.selectExpr("jiaren['mother']","size(jiaren)","map_keys(jiaren)","map_values(jiaren)")<br />
      .show(10,false)<br />
​<br />
​<br />
   spark.close()<br />
}<br />
}</td>
</tr>
</tbody>
</table>

set/seq 结构出来的字段类型为 ： array

Map 数据类型解构出来的字段类型为：map

#### 3.5.5 从RDD创建DataSet

<img src="SparkSQL _assets/media/image18.png" style="width:5.90625in;height:1.10417in" />

##### （1）从RDD[Case class类]创建Dataset

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rdd: RDD[Person] = spark.sparkContext.parallelize(Seq(<br />
Person(1, "zs"),<br />
Person(2, "ls")<br />
))<br />
​<br />
import spark.implicits._<br />
​<br />
// case class 类型的rdd，转dataset<br />
val ds: Dataset[Person] = spark.createDataset(rdd)<br />
val ds2: Dataset[Person] = rdd.toDS()<br />
ds.printSchema()<br />
ds.show()</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
* 创建一个javaBean 的RDD<br />
* 隐式转换中没有支持好对javabean的encoder机制<br />
* 需要自己传入一个encoder<br />
* 可以构造一个简单的encoder，具备序列化功能，但是不具备字段解构的功能<br />
* 但是，至少能够把一个RDD[javabean] 变成一个 dataset[javabean]<br />
* 后续可以通过rdd的map算子将数据从对象中提取出来，组装成tuple元组，然后toDF即可进入sql空间<br />
*/<br />
val rdd2: RDD[JavaStu] = spark.sparkContext.parallelize(Seq(<br />
new JavaStu(1,"a",18,"上海",99.9),<br />
new JavaStu(2,"b",28,"北京",99.9),<br />
new JavaStu(3,"c",38,"西安",99.9)<br />
))<br />
​<br />
​<br />
val encoder = Encoders.kryo(classOf[JavaStu])<br />
val ds2: Dataset[JavaStu] = spark.createDataset(rdd2)(encoder)<br />
​<br />
val df2: Dataset[Row] = ds2.map(stu =&gt; {<br />
 (stu.getId, stu.getName, stu.getAge)<br />
}).toDF("id", "name", "age")<br />
ds2.printSchema()<br />
ds2.show()<br />
df2.show()</td>
</tr>
</tbody>
</table>

> 3.3.2.5 **从RDD\[其他类\]创建Dataset**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
* 将一个RDD[Map] 变成 Dataset[Map]<br />
* 2.3.0版才支持<br />
*<br />
*/<br />
​<br />
val rdd3: RDD[Map[String, String]] = spark.sparkContext.parallelize(Seq(<br />
Map("id"-&gt;"1","name"-&gt;"zs1"),<br />
Map("id"-&gt;"2","name"-&gt;"zs2"),<br />
Map("id"-&gt;"3","name"-&gt;"zs3")<br />
))<br />
​<br />
val ds3: Dataset[Map[String, String]] = rdd3.toDS()<br />
ds3.printSchema()<br />
ds3.show()</td>
</tr>
</tbody>
</table>

### 3.6 RDD/DS/DF互转

RDD、DataFrame、Dataset三者有许多共性，有各自适用的场景常常需要在三者之间转换

*DataFrame/Dataset转RDD：*

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
val rdd1:RDD[Row]=testDF.rdd<br />
val rdd2:RDD[T]=testDS.rdd</td>
</tr>
</tbody>
</table>

RDD转DataFrame：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
import spark.implicits._<br />
val testDF = rdd.map {line=&gt;<br />
    (line._1,line._2)<br />
  }.toDF("col1","col2")</td>
</tr>
</tbody>
</table>

一般用元组把一行的数据写在一起，然后在toDF中指定字段名

*RDD转Dataset：*

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
import spark.implicits._<br />
case class Person(col1:String,col2:Int)extends Serializable //定义字段名和类型<br />
val testDS:Dataset[Person] = rdd.map {line=&gt;<br />
     Person(line._1,line._2)<br />
  }.toDS</td>
</tr>
</tbody>
</table>

可以注意到，定义每一行的类型（case class）时，已经给出了字段名和类型，后面只要往case class里面添加值即可

*Dataset转DataFrame：*

这个也很简单，因为只是把case class封装成Row

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
import spark.implicits._<br />
val testDF:Dataset[Row] = testDS.toDF</td>
</tr>
</tbody>
</table>

DataFrame转Dataset：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
import spark.implicits._<br />
case class Coltest(col1:String,col2:Int)extends Serializable //定义字段名和类型<br />
val testDS = testDF.as[Coltest]</td>
</tr>
</tbody>
</table>

这种方法就是在给出每一列的类型后，使用as方法，转成Dataset，这在数据类型是DataFrame又需要针对各个字段处理时极为方便。

在使用一些特殊的操作时，一定要加上 import spark.implicits.\_ 不然toDF、toDS无法使用

## 4. 用户自定义函数

通过spark.udf功能用户可以自定义函数。

### 4.1用户自定义UDF函数

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Shell<br />
scala&gt; val df = spark.read.json("examples/src/main/resources/people.json")<br />
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]<br />
​<br />
scala&gt; df.show()<br />
+----+-------+<br />
| age|   name|<br />
+----+-------+<br />
|null|Michael|<br />
|  30|   Andy|<br />
|  19| Justin|<br />
+----+-------+<br />
​<br />
​<br />
scala&gt; spark.udf.register("addName", (x:String)=&gt; "Name:"+x)<br />
res5: org.apache.spark.sql.expressions.UserDefinedFunction = UserDefinedFunction(&lt;function1&gt;,StringType,Some(List(StringType)))<br />
​<br />
scala&gt; df.createOrReplaceTempView("people")<br />
​<br />
scala&gt; spark.sql("Select addName(name), age from people").show()<br />
+-----------------+----+<br />
|UDF:addName(name)| age|<br />
+-----------------+----+<br />
|     Name:Michael|null|<br />
|        Name:Andy|  30|<br />
|      Name:Justin|  19|<br />
+-----------------+----+</td>
</tr>
</tbody>
</table>

**UDF案例2**

**需求，有如下数据**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
id,name,age,height,weight,yanzhi,score<br />
1,a,18,172,120,98,68.8<br />
2,b,28,175,120,97,68.8<br />
3,c,30,180,130,94,88.8<br />
4,d,18,168,110,98,68.8<br />
5,e,26,165,120,98,68.8<br />
6,f,27,182,135,95,89.8<br />
7,g,19,171,122,99,68.8</td>
</tr>
</tbody>
</table>

**需要计算每一个人和其他人之间的余弦相似度（特征向量之间的余弦相似度）**

<img src="SparkSQL _assets/media/image19.jpeg" style="width:4.85417in;height:3.63542in" />

**代码实现：**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
package cn.doitedu.sparksql.udf<br />
​<br />
import cn.doitedu.sparksql.dataframe.SparkUtil<br />
import org.apache.spark.sql.Row<br />
import org.apache.spark.sql.expressions.UserDefinedFunction<br />
​<br />
import scala.collection.mutable<br />
​<br />
​<br />
/**<br />
 * UDF 案例2 ： 用一个自定义函数实现两个向量之间的余弦相似度计算<br />
 */<br />
​<br />
case class Human(id: Int, name: String, features: Array[Double])<br />
​<br />
object CosinSimilarity {<br />
​<br />
 def main(args: Array[String]): Unit = {<br />
​<br />
​<br />
   val spark = SparkUtil.getSpark()<br />
   import spark.implicits._<br />
   import spark.sql<br />
   // 加载用户特征数据<br />
   val df = spark.read.option("inferSchema", true).option("header", true).csv("data/features.csv")<br />
   df.show()<br />
​<br />
<br />
​<br />
   // id,name,age,height,weight,yanzhi,score<br />
   // 将用户特征数据组成一个向量(数组)<br />
   // 方式1：<br />
   df.rdd.map(row =&gt; {<br />
     val id = row.getAs[Int]("id")<br />
     val name = row.getAs[String]("name")<br />
     val age = row.getAs[Double]("age")<br />
     val height = row.getAs[Double]("height")<br />
     val weight = row.getAs[Double]("weight")<br />
     val yanzhi = row.getAs[Double]("yanzhi")<br />
     val score = row.getAs[Double]("score")<br />
​<br />
    (id, name, Array(age, height, weight, yanzhi, score))<br />
  }).toDF("id", "name", "features")<br />
​<br />
   // 方式2：<br />
   df.rdd.map({<br />
     case Row(id: Int, name: String, age: Double, height: Double, weight: Double, yanzhi: Double, score: Double)<br />
     =&gt; (id, name, Array(age, height, weight, yanzhi, score))<br />
  })<br />
    .toDF("id", "name", "features")<br />
​<br />
​<br />
   // 方式3: 直接利用sql中的函数array来生成一个数组<br />
   df.selectExpr("id", "name", "array(age,height,weight,yanzhi,score) as features")<br />
   import org.apache.spark.sql.functions._<br />
   df.select('id, 'name, array('age, 'height, 'weight, 'yanzhi, 'score) as "features")<br />
​<br />
   // 方式4：返回case class<br />
   val features = df.rdd.map({<br />
     case Row(id: Int, name: String, age: Double, height: Double, weight: Double, yanzhi: Double, score: Double)<br />
     =&gt; Human(id, name, Array(age, height, weight, yanzhi, score))<br />
  })<br />
    .toDF()<br />
​<br />
   // 将表自己和自己join，得到每个人和其他所有人的连接行<br />
   val joined = features.join(features.toDF("bid","bname","bfeatures"),'id &lt; 'bid)<br />
   joined.show(100,false)<br />
​<br />
   // 定义一个计算余弦相似度的函数<br />
   // val cosinSim = (f1:Array[Double],f2:Array[Double])=&gt;{ /* 余弦相似度 */ }<br />
   // 开根号的api： Math.pow(4.0,0.5)<br />
   val cosinSim = (f1:mutable.WrappedArray[Double], f2:mutable.WrappedArray[Double])=&gt;{<br />
​<br />
     val fenmu1 = Math.pow(f1.map(Math.pow(_,2)).sum,0.5)<br />
     val fenmu2 = Math.pow(f2.map(Math.pow(_,2)).sum,0.5)<br />
​<br />
     val fenzi = f1.zip(f2).map(tp=&gt;tp._1*tp._2).sum<br />
​<br />
     fenzi/(fenmu1*fenmu2)<br />
  }<br />
​<br />
   // 注册到sql引擎： spark.udf.register("cosin_sim",consinSim)<br />
   spark.udf.register("cos_sim",cosinSim)<br />
   joined.createTempView("temp")<br />
​<br />
   // 然后在这个表上计算两人之间的余弦相似度<br />
   sql("select id,bid,cos_sim(features,bfeatures) as cos_similary from temp").show()<br />
​<br />
   // 可以自定义函数简单包装一下，就成为一个能生成column结果的dsl风格函数了<br />
   val cossim2: UserDefinedFunction = udf(cosinSim)<br />
   joined.select('id,'bid,cossim2('features,'bfeatures) as "cos_sim").show()<br />
​<br />
   spark.close()<br />
}<br />
}</td>
</tr>
</tbody>
</table>

​

### 4.2用户自定义聚合函数UDAF

弱类型的DataFrame和强类型的Dataset都提供了相关的聚合函数， 如 count()，countDistinct()，avg()，max()，min()。

除此之外，用户可以设定自己的自定义UDAF聚合函数。

UDAF的编程模板：

/\*\*

\* @date: 2019/10/12

\* @site: www.doitedu.cn

\* @author: hunter.d 涛哥

\* @qq: 657270652

\* @description:

 \*   用户自定义UDAF入门示例：求薪资的平均值

\*/

object MyAvgUDAF extends UserDefinedAggregateFunction{

​

 // 函数输入的字段schema（字段名-字段类型）

 override def inputSchema: StructType = ???

​

 // 聚合过程中，用于存储局部聚合结果的schema

 // 比如求平均薪资，中间缓存(局部数据薪资总和,局部数据人数总和)

 override def bufferSchema: StructType = ???

​

 // 函数的最终返回结果数据类型

 override def dataType: DataType = ???

​

 // 你这个函数是否是稳定一致的？（对一组相同的输入，永远返回相同的结果），只要是确定的，就写true

 override def deterministic: Boolean = true

​

 // 对局部聚合缓存的初始化方法

 override def initialize(buffer: MutableAggregationBuffer): Unit = ???

​

 // 聚合逻辑所在方法，框架会不断地传入一个新的输入row，来更新你的聚合缓存数据

 override def update(buffer: MutableAggregationBuffer, input: Row): Unit = ???

​

 // 全局聚合：将多个局部缓存中的数据，聚合成一个缓存

 // 比如：薪资和薪资累加，人数和人数累加

 override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = ???

​

 // 最终输出

 // 比如：从全局缓存中取薪资总和/人数总和

 override def evaluate(buffer: Row): Any = ???

}

核心要义：

聚合是分步骤进行： 先局部聚合，再全局聚合

局部聚合（update）的结果是保存在一个局部buffer中的

全局聚合(merge)就是将多个局部buffer再聚合成一个buffer

最后通过evaluate将全局聚合的buffer中的数据做一个运算得出你要的结果

如下图所示：

<img src="SparkSQL _assets/media/image20.png" style="width:5.90625in;height:4.63542in" />

#### 4.2.1弱类型用户自定义聚合函数UDAF

##### （1）需求说明

示例数据：

+---+----------------+------+---------+------+----------+

\| id\|    name        \| sales\|discount \|state \|  saleDate\|

+---+----------------+------+---------+------+----------+

\|  1\|       Widget Co\|1000.0\|      0.0\|    AZ\|2014-01-01\|

\|  2\|   Acme Widgets \|2000.0\|    500.0\|    CA\|2014-02-01\|

\|  3\|        Widgetry\|1000.0\|    200.0\|    CA\|2015-01-11\|

\|  4\|   Widgets R Us \|2000.0\|      0.0\|    CA\|2015-02-19\|

\|  5\|Ye Olde Widgete \|3000.0\|      0.0\|    MA\|2015-02-28\|

+---+---------------+------+--------+-----+-------------+

需求：计算x年份的同比上一年份的总销售增长率；比如2015 vs 2014的同比增长

显然，没有任何一个内置聚合函数可以完成上述需求；

可以多写一些sql逻辑来实现，但如果能自定义一个聚合函数，当然更方便高效！

```
Select yearOnyear(saleDate,sales) from t
```
##### （2）自定义UDAF实现销售额同比计算

通过继承UserDefinedAggregateFunction来实现用户自定义聚合函数。

自定义UDAF的代码骨架如下：

class UdfMy extends UserDefinedAggregateFunction{

 override def inputSchema: StructType = ???

​

 override def bufferSchema: StructType = ???

​

 override def dataType: DataType = ???

​

 override def deterministic: Boolean = ???

​

 override def initialize(buffer: MutableAggregationBuffer): Unit = ???

​

 override def update(buffer: MutableAggregationBuffer, input: Row): Unit = ???

​

 override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = ???

​

 override def evaluate(buffer: Row): Any = ???

}

完整实现代码如下：

/\*\*

 \* 工具类

 \* @param startDate

 \* @param endDate

 \*/

case class DateRange(startDate: Timestamp, endDate: Timestamp) {

 def contain(targetDate: Date): Boolean = {

   targetDate.before(endDate) && targetDate.after(startDate)

}

​

}

​

/\*\*

\* @date: 2019/10/10

\* @site: www.doitedu.cn

\* @author: hunter.d 涛哥

\* @qq: 657270652

\* @description: 自定义UDAF实现年份销售额同比增长计算

\*/

class YearOnYearBasis(current: DateRange) extends  UserDefinedAggregateFunction{

​

 // 聚合函数输入参数的数据类型

 override def inputSchema: StructType = {

   StructType(StructField("metric", DoubleType) :: StructField("timeCategory", DateType) :: Nil)

}

​

 // 聚合缓冲区中值得数据类型

 override def bufferSchema: StructType = {

   StructType(StructField("sumOfCurrent", DoubleType) :: StructField("sumOfPrevious", DoubleType) :: Nil)

}

​

 // 返回值的数据类型

 override def dataType: DataType = DoubleType

​

 // 对于相同的输入是否一直返回相同的输出。

 override def deterministic: Boolean = true

​

 // 初始化

 override def initialize(buffer: MutableAggregationBuffer): Unit = {

   buffer.update(0, 0.0)

   buffer.update(1, 0.0)

}

​

 // 相同Execute间的数据合并。

 override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {

   if (current.contain(input.getAs\[Date\](1))) {

     buffer(0) = buffer.getAs\[Double\](0) + input.getAs\[Double\](0)

  }

   val previous = DateRange(subtractOneYear(current.startDate), subtractOneYear(current.endDate))

   if (previous.contain(input.getAs\[Date\](1))) {

     buffer(1) = buffer.getAs\[Double\](0) + input.getAs\[Double\](0)

  }

}

​

 // 不同Execute间的数据合并

 override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {

   buffer1(0) = buffer1.getAs\[Double\](0) + buffer2.getAs\[Double\](0)

   buffer1(1) = buffer1.getAs\[Double\](1) + buffer2.getAs\[Double\](1)

}

​

 // 计算最终结果

 override def evaluate(buffer: Row): Any = {

   if (buffer.getDouble(1) == 0.0)

     0.0

   else

    (buffer.getDouble(0) - buffer.getDouble(1)) / buffer.getDouble(1) \* 100

}

​

​

 def subtractOneYear(d:Timestamp):Timestamp={

   Timestamp.valueOf(d.toLocalDateTime.minusYears(1))

}

}

##### （3）补充示例：自定义UDAF实现平均薪资计算

下面展示一个求平均工资的自定义聚合函数。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
package cn.doitedu.sparksql.udf<br />
​<br />
import org.apache.spark.sql.Row<br />
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}<br />
import org.apache.spark.sql.types.{DataType, DataTypes, StructField, StructType}<br />
​<br />
/**<br />
 * @description:<br />
 * 用户自定义UDAF入门示例：求薪资的平均值<br />
 */<br />
object MyAvgUDAF extends UserDefinedAggregateFunction {<br />
​<br />
 // 函数输入的字段schema（字段名-字段类型）<br />
 override def inputSchema: StructType = StructType(Seq(StructField("salary", DataTypes.DoubleType)))<br />
​<br />
 // 聚合过程中，用于存储局部聚合结果的schema<br />
 // 比如求平均薪资，中间缓存(局部数据薪资总和,局部数据人数总和)<br />
 override def bufferSchema: StructType = StructType(Seq(<br />
   StructField("sum", DataTypes.DoubleType),<br />
   StructField("cnts", DataTypes.LongType)<br />
​<br />
))<br />
​<br />
 // 函数的最终返回结果数据类型<br />
 override def dataType: DataType = DataTypes.DoubleType<br />
​<br />
 // 你这个函数是否是稳定一致的？（对一组相同的输入，永远返回相同的结果），只要是确定的，就写true<br />
 override def deterministic: Boolean = true<br />
​<br />
 // 对局部聚合缓存的初始化方法<br />
 override def initialize(buffer: MutableAggregationBuffer): Unit = {<br />
   buffer.update(0, 0.0)<br />
   buffer.update(1, 0L)<br />
}<br />
​<br />
 // 聚合逻辑所在方法，框架会不断地传入一个新的输入row，来更新你的聚合缓存数据<br />
 override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {<br />
​<br />
   // 从输入中获取那个人的薪资，加到buffer的第一个字段上<br />
   buffer.update(0, buffer.getDouble(0) + input.getDouble(0))<br />
​<br />
   // 给buffer的第2个字段加1<br />
   buffer.update(1, buffer.getLong(1) + 1)<br />
​<br />
}<br />
​<br />
 // 全局聚合：将多个局部缓存中的数据，聚合成一个缓存<br />
 // 比如：薪资和薪资累加，人数和人数累加<br />
 override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {<br />
​<br />
   // 把两个buffer的字段1（薪资和）累加到一起，并更新回buffer1<br />
   buffer1.update(0, buffer1.getDouble(0) + buffer2.getDouble(0))<br />
​<br />
   // 更新人数<br />
   buffer1.update(1, buffer1.getLong(1) + buffer2.getLong(1))<br />
​<br />
}<br />
​<br />
 // 最终输出<br />
 // 比如：从全局缓存中取薪资总和/人数总和<br />
 override def evaluate(buffer: Row): Any = {<br />
​<br />
   if (buffer.getLong(1) != 0)<br />
     buffer.getDouble(0) / buffer.getLong(1)<br />
   else<br />
     0.0<br />
​<br />
}<br />
}</td>
</tr>
</tbody>
</table>

#### 4.2.2强类型用户自定义聚合函数

通过继承Aggregator来实现强类型自定义聚合函数，同样是求平均工资

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
import org.apache.spark.sql.expressions.Aggregator<br />
import org.apache.spark.sql.Encoder<br />
import org.apache.spark.sql.Encoders<br />
import org.apache.spark.sql.SparkSession<br />
<br />
// 既然是强类型，可能有case类<br />
case class Employee(name: String, salary: Long)<br />
case class Average(var sum: Long, var count: Long)<br />
​<br />
object MyAverage extends Aggregator[Employee, Average, Double] {<br />
// 定义一个数据结构，保存工资总数和工资总个数，初始都为0<br />
def zero: Average = Average(0L, 0L)<br />
// Combine two values to produce a new value. For performance, the function may modify `buffer`<br />
// and return it instead of constructing a new object<br />
def reduce(buffer: Average, employee: Employee): Average = {<br />
buffer.sum += employee.salary<br />
buffer.count += 1<br />
buffer<br />
}<br />
// 聚合不同execute的结果<br />
def merge(b1: Average, b2: Average): Average = {<br />
b1.sum += b2.sum<br />
b1.count += b2.count<br />
b1<br />
}<br />
// 计算输出<br />
def finish(reduction: Average): Double = reduction.sum.toDouble / reduction.count<br />
// 设定之间值类型的编码器，要转换成case类<br />
// Encoders.product是进行scala元组和case类转换的编码器<br />
def bufferEncoder: Encoder[Average] = Encoders.product<br />
// 设定最终输出值的编码器<br />
def outputEncoder: Encoder[Double] = Encoders.scalaDouble<br />
}<br />
import spark.implicits._<br />
​<br />
val ds = spark.read.json("examples/src/main/resources/employees.json").as[Employee]<br />
ds.show()<br />
// +-------+------+<br />
// |   name|salary|<br />
// +-------+------+<br />
// |Michael| 3000|<br />
// |   Andy| 4500|<br />
// | Justin| 3500|<br />
// | Berta| 4000|<br />
// +-------+------+<br />
​<br />
// Convert the function to a `TypedColumn` and give it a name<br />
val averageSalary = MyAverage.toColumn.name("average_salary")<br />
val result = ds.select(averageSalary)<br />
result.show()<br />
// +--------------+<br />
// |average_salary|<br />
// +--------------+<br />
// |       3750.0|<br />
// +--------------+<br />
}</td>
</tr>
</tbody>
</table>

## 5. Spark SQL 的运行原理

正常的 SQL 执行先会经过 SQL Parser 解析 SQL，然后经过 Catalyst 优化器处理，最后到 Spark 执行。而 Catalyst 的过程又分为很多个过程，其中包括：

Analysis：主要利用 Catalog 信息将 Unresolved Logical Plan 解析成 Analyzed logical plan；

Logical Optimizations：利用一些 Rule （规则）将 Analyzed logical plan 解析成 Optimized Logical Plan；

Physical Planning：前面的 logical plan 不能被 Spark 执行，而这个过程是把 logical plan 转换成多个 physical plans，然后利用代价模型（cost model）选择最佳的 physical plan；

Code Generation：这个过程会把 SQL逻辑生成Java字节码。

所以整个 SQL 的执行过程可以使用下图表示：

<img src="SparkSQL _assets/media/image21.png" style="width:5.90625in;height:1.10417in" />

其中蓝色部分就是 Catalyst 优化器处理的部分，也是本章主要讲解的内容。

### 5.1 元数据管理SessionCatalog

SessionCatalog 主要用于各种函数资源信息和元数据信息（数据库、数据表、数据视图、数据分区与函数等）的统一管理。

创建临时表或者视图，其实是往SessionCatalog注册；

Analyzer在进行逻辑计划元数据绑定时，也是从catalog中获取元数据；

### 5.2 SQL解析成逻辑执行计划

当调用SparkSession的sql或者SQLContext的sql方法，就会使用<u>SparkSqlParser进行SQL解析。</u>

Spark 2.0.0开始引入了第三方语法解析器工具 ANTLR，对 SQL 进行词法分析并构建语法树。

（Antlr 是一款强大的语法生成器工具，可用于读取、处理、执行和翻译结构化的文本或二进制文件，是当前 Java 语言中使用最为广泛的语法生成器工具，我们常见的大数据 SQL 解析都用到了这个工具，包括 Hive、Cassandra、Phoenix、Pig 以及 presto 等）目前最新版本的 Spark 使用的是 ANTLR4）

它分为2个步骤来生成Unresolved LogicalPlan：

词法分析（SqlBaseLexer）：Lexical Analysis，负责将token分组成符号类

语法分析（SqlBaseParser）：构建一棵分析树(parse tree)或者抽象语法树AST（abstract syntax tree）

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
​/**<br />
* The AstBuilder converts an ANTLR4 ParseTree into a catalyst Expression, LogicalPlan or<br />
* TableIdentifier.<br />
*/<br />
class AstBuilder(conf: SQLConf) extends SqlBaseBaseVisitor[AnyRef] with Logging {<br />
 import ParserUtils._<br />
​<br />
 def this() = this(new SQLConf())<br />
​<br />
 protected def typedVisit[T](ctx: ParseTree): T = {<br />
<br />
...<br />
}<br />
}</td>
</tr>
</tbody>
</table>

具体来说，Spark 基于presto的语法文件定义了Spark SQL语法文件SqlBase.g4

（路径 spark-2.4.3\sql\catalyst\src\main\antlr4\org\apache\spark\sql\catalyst\parser\SqlBase.g4）

这个文件定义了 Spark SQL 支持的 SQL 语法。

<img src="SparkSQL _assets/media/image22.png" style="width:5.90625in;height:3.80208in" />

如果我们需要自定义新的语法，需要在这个文件定义好相关语法。然后使用 ANTLR4 对 SqlBase.g4 文件自动解析生成几个 Java 类，其中就包含重要的词法分析器 SqlBaseLexer.java 和语法分析器SqlBaseParser.java。运行上面的 SQL 会使用 SqlBaseLexer 来解析关键词以及各种标识符等；然后使用 SqlBaseParser 来构建语法树。

下面以一条简单的 SQL 为例进行分析

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">SQL<br />
```
SELECT sum(v)<br />
    FROM (<br />
      SELECT<br />
        t1.id,<br />
    1 + 2 + t1.value AS v<br />
    FROM t1 JOIN t2<br />
      WHERE<br />
    t1.id = t2.id AND<br />
    t1.cid = 1 AND<br />
    t1.did = t1.cid + 1 AND<br />
      t2.id &gt; 5) o</td>
</tr>
</tbody>
```
</table>

整个过程就类似于下图。

<img src="SparkSQL _assets/media/image23.jpeg" style="width:5.90625in;height:1.55208in" />

生成语法树之后，使用 AstBuilder 将语法树转换成 LogicalPlan，这个 LogicalPlan 也被称为 Unresolved LogicalPlan。解析后的逻辑计划如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
== Parsed Logical Plan ==<br />
'Project [unresolvedalias('sum('v), None)]<br />
+- 'SubqueryAlias `doitedu_stu`<br />
   +- 'Project ['t1.id, ((1 + 2) + 't1.value) AS v#16]<br />
      +- 'Filter ((('t1.id = 't2.id) &amp;&amp; ('t1.cid = 1)) &amp;&amp; (('t1.did = ('t1.cid + 1)) &amp;&amp; ('t2.id &gt; 5)))<br />
         +- 'Join Inner<br />
            :- 'UnresolvedRelation `t1`<br />
            +- 'UnresolvedRelation `t2`</td>
</tr>
</tbody>
</table>

图片表示如下：

<img src="SparkSQL _assets/media/image24.png" style="width:5.90625in;height:5.54167in" />

Unresolved LogicalPlan 是从下往上看的，t1 和 t2 两张表被生成了 UnresolvedRelation，过滤的条件、选择的列以及聚合字段都知道了。

Unresolved LogicalPlan 仅仅是一种数据结构，不包含任何数据信息，比如不知道数据源、数据类型，不同的列来自于哪张表等。

### 5.3 Analyzer绑定逻辑计划

Analyzer 阶段会使用事先定义好的 Rule 以及 SessionCatalog 等信息对 Unresolved LogicalPlan 进行元数据绑定。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
* Provides a logical query plan analyzer, which translates [[UnresolvedAttribute]]s and<br />
* [[UnresolvedRelation]]s into fully typed objects using information in a [[SessionCatalog]].<br />
*/<br />
class Analyzer(<br />
   catalog: SessionCatalog,<br />
   conf: SQLConf,<br />
   maxIterations: Int)<br />
 extends RuleExecutor[LogicalPlan] with CheckAnalysis {<br />
<br />
<br />
class SparkSqlParser(conf: SQLConf) extends AbstractSqlParser(conf) {<br />
 val astBuilder = new SparkSqlAstBuilder(conf)<br />
<br />
<br />
 override def parsePlan(sqlText: String): LogicalPlan = parse(sqlText) { parser =&gt;<br />
   astBuilder.visitSingleStatement(parser.singleStatement()) match {<br />
     case plan: LogicalPlan =&gt; plan<br />
     case _ =&gt;<br />
       val position = Origin(None, None)<br />
       throw new ParseException(Option(sqlText), "Unsupported SQL statement", position, position)<br />
  }<br />
}<br />
<br />
Rule 是定义在 Analyzer 里面的，具体如下：<br />
lazy val batches: Seq[Batch] = Seq(<br />
    Batch("Hints", fixedPoint,<br />
      new ResolveHints.ResolveBroadcastHints(conf),<br />
      ResolveHints.ResolveCoalesceHints,<br />
      ResolveHints.RemoveAllHints),<br />
    Batch("Simple Sanity Check", Once,<br />
      LookupFunctions),<br />
    Batch("Substitution", fixedPoint,<br />
      CTESubstitution,<br />
      WindowsSubstitution,<br />
      EliminateUnions,<br />
      new SubstituteUnresolvedOrdinals(conf)),<br />
    Batch("Resolution", fixedPoint,<br />
      ResolveTableValuedFunctions ::                    //解析表的函数<br />
      ResolveRelations ::                               //解析表或视图<br />
      ResolveReferences ::                              //解析列<br />
      ResolveCreateNamedStruct ::<br />
      ResolveDeserializer ::                            //解析反序列化操作类<br />
      ResolveNewInstance ::<br />
      ResolveUpCast ::                                  //解析类型转换<br />
      ResolveGroupingAnalytics ::<br />
      ResolvePivot ::<br />
      ResolveOrdinalInOrderByAndGroupBy ::<br />
      ResolveAggAliasInGroupBy ::<br />
      ResolveMissingReferences ::<br />
      ExtractGenerator ::<br />
      ResolveGenerate ::<br />
      ResolveFunctions ::                               //解析函数<br />
      ResolveAliases ::                                 //解析表别名<br />
      ResolveSubquery ::                                //解析子查询<br />
      ResolveSubqueryColumnAliases ::<br />
      ResolveWindowOrder ::<br />
      ResolveWindowFrame ::<br />
      ResolveNaturalAndUsingJoin ::<br />
      ResolveOutputRelation ::<br />
      ExtractWindowExpressions ::<br />
      GlobalAggregates ::<br />
      ResolveAggregateFunctions ::<br />
      TimeWindowing ::<br />
      ResolveInlineTables(conf) ::<br />
      ResolveHigherOrderFunctions(catalog) ::<br />
      ResolveLambdaVariables(conf) ::<br />
      ResolveTimeZone(conf) ::<br />
      ResolveRandomSeed ::<br />
      TypeCoercion.typeCoercionRules(conf) ++<br />
      extendedResolutionRules : _*),<br />
    Batch("Post-Hoc Resolution", Once, postHocResolutionRules: _*),<br />
    Batch("View", Once,<br />
      AliasViewChild(conf)),<br />
    Batch("Nondeterministic", Once,<br />
      PullOutNondeterministic),<br />
    Batch("UDF", Once,<br />
      HandleNullInputsForUDF),<br />
    Batch("FixNullability", Once,<br />
      FixNullability),<br />
    Batch("Subquery", Once,<br />
      UpdateOuterReferences),<br />
    Batch("Cleanup", fixedPoint,<br />
      CleanupAliases)<br />
)</td>
</tr>
</tbody>
</table>

从上面代码可以看出，多个性质类似的 Rule 组成一个 Batch；而多个 Batch 构成一个 batches。这些 batches 会由 RuleExecutor 执行，先按一个一个 Batch 顺序执行，然后对 Batch 里面的每个 Rule 顺序执行。每个 Batch 会执行一次（Once）或多次（FixedPoint，由spark.sql.optimizer.maxIterations 参数决定），执行过程如下：

<img src="SparkSQL _assets/media/image25.png" style="width:5.90625in;height:2.61458in" />

### 5.4 Optimizer优化逻辑计划

优化器也是会定义一套Rules，利用这些Rule对逻辑计划和Exepression进行迭代处理，从而使得树的节点进行合并和优化

<img src="SparkSQL _assets/media/image26.png" style="width:5.90625in;height:3.09375in" />

在前文的绑定逻辑计划阶段对 Unresolved LogicalPlan 进行相关 transform 操作得到了 Analyzed Logical Plan，这个 Analyzed Logical Plan 是可以直接转换成 Physical Plan 然后在spark中执行。但是如果直接这么弄的话，得到的 Physical Plan 很可能不是最优的，因为在实际应用中，很多低效的写法会带来执行效率的问题，需要进一步对Analyzed Logical Plan 进行处理，得到更优的逻辑算子树。于是，针对SQL 逻辑算子树的优化器 Optimizer 应运而生。

这个阶段的优化器主要是基于规则的（Rule-based Optimizer，简称 RBO），而绝大部分的规则都是启发式规则，也就是基于直观或经验而得出的规则，比如列裁剪（过滤掉查询不需要使用到的列）、谓词下推（将过滤尽可能地下沉到数据源端）、常量累加（比如 1 + 2 这种事先计算好） 以及常量替换（比如 SELECT \* FROM table WHERE i = 5 AND j = i + 3 可以转换成 SELECT \* FROM table WHERE i = 5 AND j = 8）等等。

与绑定逻辑计划阶段类似，这个阶段所有的规则也是实现 Rule 抽象类，多个规则组成一个 Batch，多个 Batch 组成一个 batches，同样也是在 RuleExecutor 中进行执行。

**核心源码骨架如下列截图所示：**

<img src="SparkSQL _assets/media/image27.png" style="width:5.90625in;height:2.48958in" />

<img src="SparkSQL _assets/media/image28.png" style="width:5.90625in;height:3.32292in" />

<img src="SparkSQL _assets/media/image29.png" style="width:5.90625in;height:4.25in" />

<img src="SparkSQL _assets/media/image30.png" style="width:5.90625in;height:3.82292in" />

<img src="SparkSQL _assets/media/image31.png" style="width:5.90625in;height:2.91667in" />

那么针对前文的 SQL 语句，这个过程都会执行哪些优化呢？下文举例说明。

#### 5.4.1谓词下推

谓词下推在 Spark SQL 是由 PushDownPredicate 实现的，这个过程主要**<u>将过滤条件尽可能地下推到底层，最好是数据源</u>**。上面介绍的 SQL，使用谓词下推优化得到的逻辑计划如下：

<img src="SparkSQL _assets/media/image32.png" style="width:5.90625in;height:2.84375in" />

从上图可以看出，谓词下推将 Filter 算子直接下推到 Join 之前了（注意，上图是从下往上看的）。也就是在扫描 t1 表的时候会先使用 ((((isnotnull(cid#2) && isnotnull(did#3)) && (cid#2 = 1)) && (did#3 = 2)) && (id#0 \> 50000)) && isnotnull(id#0) 过滤条件过滤出满足条件的数据；同时在扫描 t2 表的时候会先使用 isnotnull(id#8) && (id#8 \> 50000) 过滤条件过滤出满足条件的数据。经过这样的操作，可以大大减少 Join 算子处理的数据量，从而加快计算速度。

#### 5.4.2列裁剪

列裁剪在 Spark SQL 是由 ColumnPruning 实现的。因为我们查询的表可能有很多个字段，但是每次查询我们很大可能不需要扫描出所有的字段，这个时候利用列裁剪可以把那些查询不需要的字段过滤掉，使得扫描的数据量减少。所以针对我们上面介绍的 SQL，使用列裁剪优化得到的逻辑计划如下：

<img src="SparkSQL _assets/media/image33.png" style="width:5.90625in;height:3.1875in" />

从上图可以看出，经过列裁剪后，t1 表只需要查询 id 和 value 两个字段；t2 表只需要查询 id 字段。这样减少了数据的传输，而且如果底层的文件格式为列存（比如 Parquet），可以大大提高数据的扫描速度的。

#### 5.4.3常量替换

常量替换在 Spark SQL 是由 ConstantPropagation 实现的。也就是将变量替换成常量，比如 SELECT \* FROM table WHERE i = 5 AND j = i + 3 可以转换成 SELECT \* FROM table WHERE i = 5 AND j = 8。这个看起来好像没什么的，但是如果扫描的行数非常多可以减少很多的计算时间的开销的。经过这个优化，得到的逻辑计划如下：

<img src="SparkSQL _assets/media/image34.png" style="width:5.90625in;height:3.22917in" />

我们的查询中有 t1.cid = 1 AND t1.did = t1.cid + 1 查询语句，从里面可以看出 t1.cid 其实已经是确定的值了，所以我们完全可以使用它计算出 t1.did。

#### 5.4.4常量累加

常量累加在 Spark SQL 是由 ConstantFolding 实现的。这个和常量替换类似，也是在这个阶段把一些常量表达式事先计算好。这个看起来改动的不大，但是在数据量非常大的时候可以减少大量的计算，减少 CPU 等资源的使用。经过这个优化，得到的逻辑计划如下：

<img src="SparkSQL _assets/media/image35.png" style="width:5.71875in;height:6.72917in" />

经过上面四个步骤的优化之后，得到的优化之后的逻辑计划为：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
== Optimized Logical Plan ==<br />
Aggregate [sum(cast(v#16 as bigint)) AS sum(v)#22L]<br />
+- Project [(3 + value#1) AS v#16]<br />
   +- Join Inner, (id#0 = id#8)<br />
      :- Project [id#0, value#1]<br />
      :  +- Filter (((((isnotnull(cid#2) &amp;&amp; isnotnull(did#3)) &amp;&amp; (cid#2 = 1)) &amp;&amp; (did#3 = 2)) &amp;&amp; (id#0 &gt; 5)) &amp;&amp; isnotnull(id#0))<br />
      :     +- Relation[id#0,value#1,cid#2,did#3] csv<br />
      +- Project [id#8]<br />
         +- Filter (isnotnull(id#8) &amp;&amp; (id#8 &gt; 5))<br />
            +- Relation[id#8,value#9,cid#10,did#11] csv</td>
</tr>
</tbody>
</table>

对应的图如下：

<img src="SparkSQL _assets/media/image35.png" style="width:5.71875in;height:6.72917in" />

到这里，优化逻辑计划阶段就算完成了。另外，Spark 内置提供了多达70个优化 Rule，详情请参见

https://github.com/apache/spark/blob/master/sql/catalyst/src/main/scala/org/apache/spark/sql/catalyst/optimizer/Optimizer.scala#L59

### 5.5使用SparkPlanner生成物理计划

SparkSpanner使用Planning Strategies，对优化后的逻辑计划进行转换，生成可以执行的物理计划SparkPlan.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
/**<br />
* 将逻辑计划转成物理计划的抽象类.<br />
* 各实现类通过各种GenericStrategy来生成各种可行的待选物理计划.<br />
* 如一个策略无法对逻辑计划树的所有操作转换，则会调用[GenericStrategy#planLater planLater]], 来获得 一个“占位符”对象暂时填充；之后由[[collectPlaceholders collected]]收集并使用其他策略进行转换<br />
<br />
* TODO: 目前为止，永远只生成一个物理计划<br />
*       后续迭代中会对“多计划”予以实现<br />
*/<br />
abstract class QueryPlanner[PhysicalPlan &lt;: TreeNode[PhysicalPlan]] {<br />
 /** A list of execution strategies that can be used by the planner */<br />
 def strategies: Seq[GenericStrategy[PhysicalPlan]]<br />
​<br />
 def plan(plan: LogicalPlan): Iterator[PhysicalPlan] = {<br />
   // 显然，此处还有大量工作需要做，可依然...<br />
​<br />
   // 收集所有可选的物理计划.<br />
   val candidates = strategies.iterator.flatMap(_(plan))<br />
<br />
abstract class SparkStrategies extends QueryPlanner[SparkPlan] {<br />
 self: SparkPlanner =&gt;<br />
​<br />
 /**<br />
  * Plans special cases of limit operators.<br />
  */<br />
 object SpecialLimits extends Strategy {<br />
     <br />
   <br />
class SparkPlanner(<br />
   val sparkContext: SparkContext,<br />
   val conf: SQLConf,<br />
   val experimentalMethods: ExperimentalMethods)<br />
 extends SparkStrategies {      </td>
</tr>
</tbody>
</table>

逻辑计划翻译成物理计划时，使用的是策略（Strategy）；

前面介绍的逻辑计划绑定和优化经过 Transformations 动作之后，树的类型并没有改变，

Logical Plan转化成物理计划后，树的类型改变了，由 Logical Plan 转换成 Physical Plan 了。

一个逻辑计划（Logical Plan）经过一系列的策略处理之后，得到多个物理计划（Physical Plans），物理计划在 Spark 是由 SparkPlan 实现的。

多个物理计划经过代价模型（Cost Model）得到选择后的物理计划（Selected Physical Plan），整个过程如下所示：

<img src="SparkSQL _assets/media/image36.png" style="width:5.90625in;height:1.625in" />

Cost Model 对应的就是基于代价的优化（Cost-based Optimizations，CBO，主要由华为的大佬们实现的，详见 SPARK-16026 ），核心思想是计算每个物理计划的代价，然后得到最优的物理计划。目前，这一部分并没有实现，直接返回多个物理计划列表的第一个作为最优的物理计划，如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
lazy val sparkPlan: SparkPlan = {<br />
    SparkSession.setActiveSession(sparkSession)<br />
    // TODO: We use next(), i.e. take the first plan returned by the planner, here for now,<br />
    //       but we will implement to choose the best plan.<br />
    planner.plan(ReturnAnswer(optimizedPlan)).next()<br />
}</td>
</tr>
</tbody>
</table>

而 SPARK-16026 引入的 CBO 优化主要是在前面介绍的优化逻辑计划阶段 - Optimizer 阶段进行的，对应的 Rule 为 CostBasedJoinReorder，并且默认是关闭的，需要通过 spark.sql.cbo.enabled 或 spark.sql.cbo.joinReorder.enabled 参数开启。

所以到了这个节点，最后得到的物理计划如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
== Physical Plan ==<br />
*(3) HashAggregate(keys=[], functions=[sum(cast(v#16 as bigint))], output=[sum(v)#22L])<br />
+- Exchange SinglePartition<br />
   +- *(2) HashAggregate(keys=[], functions=[partial_sum(cast(v#16 as bigint))], output=[sum#24L])<br />
      +- *(2) Project [(3 + value#1) AS v#16]<br />
         +- *(2) BroadcastHashJoin [id#0], [id#8], Inner, BuildRight<br />
            :- *(2) Project [id#0, value#1]<br />
            :  +- *(2) Filter (((((isnotnull(cid#2) &amp;&amp; isnotnull(did#3)) &amp;&amp; (cid#2 = 1)) &amp;&amp; (did#3 = 2)) &amp;&amp; (id#0 &gt; 5)) &amp;&amp; isnotnull(id#0))<br />
            :     +- *(2) FileScan csv [id#0,value#1,cid#2,did#3] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/iteblog/t1.csv], PartitionFilters: [], PushedFilters: [IsNotNull(cid), IsNotNull(did), EqualTo(cid,1), EqualTo(did,2), GreaterThan(id,5), IsNotNull(id)], ReadSchema: struct&lt;id:int,value:int,cid:int,did:int&gt;<br />
            +- BroadcastExchange HashedRelationBroadcastMode(List(cast(input[0, int, true] as bigint)))<br />
               +- *(1) Project [id#8]<br />
                  +- *(1) Filter (isnotnull(id#8) &amp;&amp; (id#8 &gt; 5))<br />
                     +- *(1) FileScan csv [id#8] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/iteblog/t2.csv], PartitionFilters: [], PushedFilters: [IsNotNull(id), GreaterThan(id,5)], ReadSchema: struct&lt;id:int&gt;</td>
</tr>
</tbody>
</table>

从上面的结果可以看出，物理计划阶段已经知道数据源是从 csv 文件里面读取了，也知道文件的路径，数据类型等。而且在读取文件的时候，直接将过滤条件（PushedFilters）加进去了。

同时，这个 Join 变成了 BroadcastHashJoin，也就是将 t2 表的数据 Broadcast 到 t1 表所在的节点。图表示如下：

<img src="SparkSQL _assets/media/image37.png" style="width:5.90625in;height:3.15625in" />

到这里， Physical Plan 就完全生成了。

### 5.6从物理执行计划获取inputRdd执行

从物理计划上，获取inputRdd

从物理计划上，生成全阶段代码，并编译反射出迭代器newBiIterator的Clazz

\[真名：BufferedRowIterator\]

然后将inputRDD做一个transformation得到最终要执行的rdd

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Scala<br />
inputRdd.mapPartitionsWithIndex((index,iter)=&gt;{<br />
  new newBiIterator(){<br />
    hasNext(){<br />
        iter.hasNext<br />
}<br />
    next(){<br />
        processNext(iter.next())<br />
}<br />
}<br />
})<br />
<br />
然后，对最后返回的rdd，执行你所需要的行动算子<br />
rdd.collect().foreach(println)</td>
</tr>
</tbody>
</table>

## 6. Spark SQL实战

### 6.1 数据说明

数据集是货品交易数据集。

|  |  |
|:---|:---|
| <img src="SparkSQL _assets/media/image38.jpeg" style="width:2.78125in;height:1.05208in" /> |  |

每个订单可能包含多个货品，每个订单可以产生多次交易，不同的货品有不同的单价。

### 6.2 加载数据

**tbStock：**

scala\> case class tbStock(ordernumber:String,locationid:String,dateid:String) extends Serializable

defined class tbStock

​

scala\> val tbStockRdd = spark.sparkContext.textFile("tbStock.txt")

tbStockRdd: org.apache.spark.rdd.RDD\[String\] = tbStock.txt MapPartitionsRDD\[1\] at textFile at \<console\>:23

​

scala\> val tbStockDS = tbStockRdd.map(\_.split(",")).map(attr=\>tbStock(attr(0),attr(1),attr(2))).toDS

tbStockDS: org.apache.spark.sql.Dataset\[tbStock\] = \[ordernumber: string, locationid: string ... 1 more field\]

​

scala\> tbStockDS.show()

+------------+----------+---------+

\| ordernumber\|locationid\|   dataid\|

+------------+----------+---------+

\|BYSL00000893\|      ZHAO\|2007-8-23\|

\|BYSL00000897\|      ZHAO\|2007-8-24\|

\|BYSL00000898\|      ZHAO\|2007-8-25\|

\|BYSL00000899\|      ZHAO\|2007-8-26\|

\|BYSL00000900\|      ZHAO\|2007-8-26\|

\|BYSL00000901\|      ZHAO\|2007-8-27\|

\|BYSL00000902\|      ZHAO\|2007-8-27\|

\|BYSL00000904\|      ZHAO\|2007-8-28\|

\|BYSL00000905\|      ZHAO\|2007-8-28\|

\|BYSL00000906\|      ZHAO\|2007-8-28\|

\|BYSL00000907\|      ZHAO\|2007-8-29\|

\|BYSL00000908\|      ZHAO\|2007-8-30\|

\|BYSL00000909\|      ZHAO\| 2007-9-1\|

\|BYSL00000910\|      ZHAO\| 2007-9-1\|

\|BYSL00000911\|      ZHAO\|2007-8-31\|

\|BYSL00000912\|      ZHAO\| 2007-9-2\|

\|BYSL00000913\|      ZHAO\| 2007-9-3\|

\|BYSL00000914\|      ZHAO\| 2007-9-3\|

\|BYSL00000915\|      ZHAO\| 2007-9-4\|

\|BYSL00000916\|      ZHAO\| 2007-9-4\|

+------------+----------+---------+

only showing top 20 rows

tbStockDetail：

scala\> case class tbStockDetail(ordernumber:String, rownum:Int, itemid:String, number:Int, price:Double, amount:Double) extends Serializable

defined class tbStockDetail

​

scala\> val tbStockDetailRdd = spark.sparkContext.textFile("tbStockDetail.txt")

tbStockDetailRdd: org.apache.spark.rdd.RDD\[String\] = tbStockDetail.txt MapPartitionsRDD\[13\] at textFile at \<console\>:23

​

scala\> val tbStockDetailDS = tbStockDetailRdd.map(\_.split(",")).map(attr=\> tbStockDetail(attr(0),attr(1).trim().toInt,attr(2),attr(3).trim().toInt,attr(4).trim().toDouble, attr(5).trim().toDouble)).toDS

tbStockDetailDS: org.apache.spark.sql.Dataset\[tbStockDetail\] = \[ordernumber: string, rownum: int ... 4 more fields\]

​

scala\> tbStockDetailDS.show()

+------------+------+--------------+------+-----+------+

\| ordernumber\|rownum\|        itemid\|number\|price\|amount\|

+------------+------+--------------+------+-----+------+

\|BYSL00000893\|     0\|FS527258160501\|    -1\|268.0\|-268.0\|

\|BYSL00000893\|     1\|FS527258169701\|     1\|268.0\| 268.0\|

\|BYSL00000893\|     2\|FS527230163001\|     1\|198.0\| 198.0\|

\|BYSL00000893\|     3\|24627209125406\|     1\|298.0\| 298.0\|

\|BYSL00000893\|     4\|K9527220210202\|     1\|120.0\| 120.0\|

\|BYSL00000893\|     5\|01527291670102\|     1\|268.0\| 268.0\|

\|BYSL00000893\|     6\|QY527271800242\|     1\|158.0\| 158.0\|

\|BYSL00000893\|     7\|ST040000010000\|     8\|  0.0\|   0.0\|

\|BYSL00000897\|     0\|04527200711305\|     1\|198.0\| 198.0\|

\|BYSL00000897\|     1\|MY627234650201\|     1\|120.0\| 120.0\|

\|BYSL00000897\|     2\|01227111791001\|     1\|249.0\| 249.0\|

\|BYSL00000897\|     3\|MY627234610402\|     1\|120.0\| 120.0\|

\|BYSL00000897\|     4\|01527282681202\|     1\|268.0\| 268.0\|

\|BYSL00000897\|     5\|84126182820102\|     1\|158.0\| 158.0\|

\|BYSL00000897\|     6\|K9127105010402\|     1\|239.0\| 239.0\|

\|BYSL00000897\|     7\|QY127175210405\|     1\|199.0\| 199.0\|

\|BYSL00000897\|     8\|24127151630206\|     1\|299.0\| 299.0\|

\|BYSL00000897\|     9\|G1126101350002\|     1\|158.0\| 158.0\|

\|BYSL00000897\|    10\|FS527258160501\|     1\|198.0\| 198.0\|

\|BYSL00000897\|    11\|ST040000010000\|    13\|  0.0\|   0.0\|

+------------+------+--------------+------+-----+------+

only showing top 20 rows

tbDate：

scala\> case class tbDate(dateid:String, years:Int, theyear:Int, month:Int, day:Int, weekday:Int, week:Int, quarter:Int, period:Int, halfmonth:Int) extends Serializable

defined class tbDate

​

scala\> val tbDateRdd = spark.sparkContext.textFile("tbDate.txt")

tbDateRdd: org.apache.spark.rdd.RDD\[String\] = tbDate.txt MapPartitionsRDD\[20\] at textFile at \<console\>:23

​

scala\> val tbDateDS = tbDateRdd.map(\_.split(",")).map(attr=\> tbDate(attr(0),attr(1).trim().toInt, attr(2).trim().toInt,attr(3).trim().toInt, attr(4).trim().toInt, attr(5).trim().toInt, attr(6).trim().toInt, attr(7).trim().toInt, attr(8).trim().toInt, attr(9).trim().toInt)).toDS

tbDateDS: org.apache.spark.sql.Dataset\[tbDate\] = \[dateid: string, years: int ... 8 more fields\]

​

scala\> tbDateDS.show()

+---------+------+-------+-----+---+-------+----+-------+------+---------+

\|   dateid\| years\|theyear\|month\|day\|weekday\|week\|quarter\|period\|halfmonth\|

+---------+------+-------+-----+---+-------+----+-------+------+---------+

\| 2003-1-1\|200301\|   2003\|    1\|  1\|      3\|   1\|      1\|     1\|        1\|

\| 2003-1-2\|200301\|   2003\|    1\|  2\|      4\|   1\|      1\|     1\|        1\|

\| 2003-1-3\|200301\|   2003\|    1\|  3\|      5\|   1\|      1\|     1\|        1\|

\| 2003-1-4\|200301\|   2003\|    1\|  4\|      6\|   1\|      1\|     1\|        1\|

\| 2003-1-5\|200301\|   2003\|    1\|  5\|      7\|   1\|      1\|     1\|        1\|

\| 2003-1-6\|200301\|   2003\|    1\|  6\|      1\|   2\|      1\|     1\|        1\|

\| 2003-1-7\|200301\|   2003\|    1\|  7\|      2\|   2\|      1\|     1\|        1\|

\| 2003-1-8\|200301\|   2003\|    1\|  8\|      3\|   2\|      1\|     1\|        1\|

\| 2003-1-9\|200301\|   2003\|    1\|  9\|      4\|   2\|      1\|     1\|        1\|

\|2003-1-10\|200301\|   2003\|    1\| 10\|      5\|   2\|      1\|     1\|        1\|

\|2003-1-11\|200301\|   2003\|    1\| 11\|      6\|   2\|      1\|     2\|        1\|

\|2003-1-12\|200301\|   2003\|    1\| 12\|      7\|   2\|      1\|     2\|        1\|

\|2003-1-13\|200301\|   2003\|    1\| 13\|      1\|   3\|      1\|     2\|        1\|

\|2003-1-14\|200301\|   2003\|    1\| 14\|      2\|   3\|      1\|     2\|        1\|

\|2003-1-15\|200301\|   2003\|    1\| 15\|      3\|   3\|      1\|     2\|        1\|

\|2003-1-16\|200301\|   2003\|    1\| 16\|      4\|   3\|      1\|     2\|        2\|

\|2003-1-17\|200301\|   2003\|    1\| 17\|      5\|   3\|      1\|     2\|        2\|

\|2003-1-18\|200301\|   2003\|    1\| 18\|      6\|   3\|      1\|     2\|        2\|

\|2003-1-19\|200301\|   2003\|    1\| 19\|      7\|   3\|      1\|     2\|        2\|

\|2003-1-20\|200301\|   2003\|    1\| 20\|      1\|   4\|      1\|     2\|        2\|

+---------+------+-------+-----+---+-------+----+-------+------+---------+

only showing top 20 rows

注册表：

scala\> tbStockDS.createOrReplaceTempView("tbStock")

​

scala\> tbDateDS.createOrReplaceTempView("tbDate")

​

scala\> tbStockDetailDS.createOrReplaceTempView("tbStockDetail")

### 6.3 计算所有数据中每年的销售单数、销售总额

统计所有订单中每年的销售单数、销售总额

三个表连接后以count(distinct a.ordernumber)计销售单数，sum(b.amount)计销售总额

|  |  |
|:---|:---|
| <img src="SparkSQL _assets/media/image38.jpeg" style="width:2.78125in;height:1.05208in" /> |  |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>SELECT c.theyear, COUNT(DISTINCT a.ordernumber), SUM(b.amount)</strong></p>
<p><strong>FROM tbStock a</strong></p>
<p><strong>JOIN tbStockDetail b ON a.ordernumber = b.ordernumber</strong></p>
<p><strong>JOIN tbDate c ON a.dateid = c.dateid</strong></p>
<p><strong>GROUP BY c.theyear</strong></p>
<p><strong>ORDER BY c.theyear</strong></p></td>
</tr>
</tbody>
</table>

|  |
|:---|
| **spark.sql("SELECT c.theyear, COUNT(DISTINCT a.ordernumber), SUM(b.amount) FROM tbStock a JOIN tbStockDetail b ON a.ordernumber = b.ordernumber JOIN tbDate c ON a.dateid = c.dateid GROUP BY c.theyear ORDER BY c.theyear").show** |

结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>+-------+---------------------------+--------------------+</strong></p>
<p><strong>|theyear|count(DISTINCT ordernumber)| sum(amount)|</strong></p>
<p><strong>+-------+---------------------------+--------------------+</strong></p>
<p><strong>| 2004| 1094| 3268115.499199999|</strong></p>
<p><strong>| 2005| 3828|1.3257564149999991E7|</strong></p>
<p><strong>| 2006| 3772|1.3680982900000006E7|</strong></p>
<p><strong>| 2007| 4885|1.6719354559999993E7|</strong></p>
<p><strong>| 2008| 4861| 1.467429530000001E7|</strong></p>
<p><strong>| 2009| 2619| 6323697.189999999|</strong></p>
<p><strong>| 2010| 94| 210949.65999999997|</strong></p>
<p><strong>+-------+---------------------------+--------------------+</strong></p></td>
</tr>
</tbody>
</table>

### 6.4 查询每年最大金额的订单及其金额

目标：统计每年最大金额订单的销售额:

|  |  |
|:---|:---|
| <img src="SparkSQL _assets/media/image38.jpeg" style="width:2.78125in;height:1.05208in" /> |  |

1\. 统计每年，每个订单一共有多少销售额

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>SELECT a.dateid, a.ordernumber, SUM(b.amount) AS SumOfAmount</strong></p>
<p><strong>FROM tbStock a</strong></p>
<p><strong>JOIN tbStockDetail b ON a.ordernumber = b.ordernumber</strong></p>
<p><strong>GROUP BY a.dateid, a.ordernumber</strong></p></td>
</tr>
</tbody>
</table>

|  |
|:---|
| **spark.sql("SELECT a.dateid, a.ordernumber, SUM(b.amount) AS SumOfAmount FROM tbStock a JOIN tbStockDetail b ON a.ordernumber = b.ordernumber GROUP BY a.dateid, a.ordernumber").show** |

2\. 结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>+----------+------------+------------------+</strong></p>
<p><strong>| dateid| ordernumber| SumOfAmount|</strong></p>
<p><strong>+----------+------------+------------------+</strong></p>
<p><strong>| 2008-4-9|BYSL00001175| 350.0|</strong></p>
<p><strong>| 2008-5-12|BYSL00001214| 592.0|</strong></p>
<p><strong>| 2008-7-29|BYSL00011545| 2064.0|</strong></p>
<p><strong>| 2008-9-5|DGSL00012056| 1782.0|</strong></p>
<p><strong>| 2008-12-1|DGSL00013189| 318.0|</strong></p>
<p><strong>|2008-12-18|DGSL00013374| 963.0|</strong></p>
<p><strong>| 2009-8-9|DGSL00015223| 4655.0|</strong></p>
<p><strong>| 2009-10-5|DGSL00015585| 3445.0|</strong></p>
<p><strong>| 2010-1-14|DGSL00016374| 2934.0|</strong></p>
<p><strong>| 2006-9-24|GCSL00000673|3556.1000000000004|</strong></p>
<p><strong>| 2007-1-26|GCSL00000826| 9375.199999999999|</strong></p>
<p><strong>| 2007-5-24|GCSL00001020| 6171.300000000002|</strong></p>
<p><strong>| 2008-1-8|GCSL00001217| 7601.6|</strong></p>
<p><strong>| 2008-9-16|GCSL00012204| 2018.0|</strong></p>
<p><strong>| 2006-7-27|GHSL00000603| 2835.6|</strong></p>
<p><strong>|2006-11-15|GHSL00000741| 3951.94|</strong></p>
<p><strong>| 2007-6-6|GHSL00001149| 0.0|</strong></p>
<p><strong>| 2008-4-18|GHSL00001631| 12.0|</strong></p>
<p><strong>| 2008-7-15|GHSL00011367| 578.0|</strong></p>
<p><strong>| 2009-5-8|GHSL00014637| 1797.6|</strong></p>
<p><strong>+----------+------------+------------------+</strong></p></td>
</tr>
</tbody>
</table>

3.以上一步查询结果为基础表，和表tbDate使用dateid join，求出每年最大金额订单的销售额

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>SELECT theyear, MAX(c.SumOfAmount) AS SumOfAmount</strong></p>
<p><strong>FROM (SELECT a.dateid, a.ordernumber, SUM(b.amount) AS SumOfAmount</strong></p>
<p><strong>FROM tbStock a</strong></p>
<p><strong>JOIN tbStockDetail b ON a.ordernumber = b.ordernumber</strong></p>
<p><strong>GROUP BY a.dateid, a.ordernumber</strong></p>
<p><strong>) c</strong></p>
<p><strong>JOIN tbDate d ON c.dateid = d.dateid</strong></p>
<p><strong>GROUP BY theyear</strong></p>
<p><strong>ORDER BY theyear DESC</strong></p></td>
</tr>
</tbody>
</table>

|  |
|:---|
| **spark.sql("SELECT theyear, MAX(c.SumOfAmount) AS SumOfAmount FROM (SELECT a.dateid, a.ordernumber, SUM(b.amount) AS SumOfAmount FROM tbStock a JOIN tbStockDetail b ON a.ordernumber = b.ordernumber GROUP BY a.dateid, a.ordernumber ) c JOIN tbDate d ON c.dateid = d.dateid GROUP BY theyear ORDER BY theyear DESC").show** |

4\. 结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>+-------+------------------+</strong></p>
<p><strong>|theyear| SumOfAmount|</strong></p>
<p><strong>+-------+------------------+</strong></p>
<p><strong>| 2010|13065.280000000002|</strong></p>
<p><strong>| 2009|25813.200000000008|</strong></p>
<p><strong>| 2008| 55828.0|</strong></p>
<p><strong>| 2007| 159126.0|</strong></p>
<p><strong>| 2006| 36124.0|</strong></p>
<p><strong>| 2005|38186.399999999994|</strong></p>
<p><strong>| 2004| 23656.79999999997|</strong></p>
<p><strong>+-------+------------------+</strong></p></td>
</tr>
</tbody>
</table>

### 6.5 计算每年最畅销货品

目标1：统计每年最畅销货品（哪个货品销售额amount在当年最高，哪个就是最畅销货品）

目标2：统计每年最畅销货品（哪个货品销售数量当年最高，哪个就是最畅销货品）

|  |  |
|:---|:---|
| <img src="SparkSQL _assets/media/image38.jpeg" style="width:2.78125in;height:1.05208in" /> |  |

第一步、求出每年每个货品的销售额

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>SELECT c.theyear, b.itemid, SUM(b.amount) AS SumOfAmount</strong></p>
<p><strong>FROM tbStock a</strong></p>
<p><strong>JOIN tbStockDetail b ON a.ordernumber = b.ordernumber</strong></p>
<p><strong>JOIN tbDate c ON a.dateid = c.dateid</strong></p>
<p><strong>GROUP BY c.theyear, b.itemid</strong></p></td>
</tr>
</tbody>
</table>

|  |
|:---|
| **spark.sql("SELECT c.theyear, b.itemid, SUM(b.amount) AS SumOfAmount FROM tbStock a JOIN tbStockDetail b ON a.ordernumber = b.ordernumber JOIN tbDate c ON a.dateid = c.dateid GROUP BY c.theyear, b.itemid").show** |

结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>+-------+--------------+------------------+</strong></p>
<p><strong>|theyear| itemid| SumOfAmount|</strong></p>
<p><strong>+-------+--------------+------------------+</strong></p>
<p><strong>| 2004|43824480810202| 4474.72|</strong></p>
<p><strong>| 2006|YA214325360101| 556.0|</strong></p>
<p><strong>| 2006|BT624202120102| 360.0|</strong></p>
<p><strong>| 2007|AK215371910101|24603.639999999992|</strong></p>
<p><strong>| 2008|AK216169120201|29144.199999999997|</strong></p>
<p><strong>| 2008|YL526228310106|16073.099999999999|</strong></p>
<p><strong>| 2009|KM529221590106| 5124.800000000001|</strong></p>
<p><strong>| 2004|HT224181030201|2898.6000000000004|</strong></p>
<p><strong>| 2004|SG224308320206| 7307.06|</strong></p>
<p><strong>| 2007|04426485470201|14468.800000000001|</strong></p>
<p><strong>| 2007|84326389100102| 9134.11|</strong></p>
<p><strong>| 2007|B4426438020201| 19884.2|</strong></p>
<p><strong>| 2008|YL427437320101|12331.799999999997|</strong></p>
<p><strong>| 2008|MH215303070101| 8827.0|</strong></p>
<p><strong>| 2009|YL629228280106| 12698.4|</strong></p>
<p><strong>| 2009|BL529298020602| 2415.8|</strong></p>
<p><strong>| 2009|F5127363019006| 614.0|</strong></p>
<p><strong>| 2005|24425428180101| 34890.74|</strong></p>
<p><strong>| 2007|YA214127270101| 240.0|</strong></p>
<p><strong>| 2007|MY127134830105| 11099.92|</strong></p>
<p><strong>+-------+--------------+------------------+</strong></p></td>
</tr>
</tbody>
</table>

第二步：在第一步的基础上，统计每年单个货品中的最大金额

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>SELECT d.theyear, MAX(d.SumOfAmount) AS MaxOfAmount</strong></p>
<p><strong>FROM (SELECT c.theyear, b.itemid, SUM(b.amount) AS SumOfAmount</strong></p>
<p><strong>FROM tbStock a</strong></p>
<p><strong>JOIN tbStockDetail b ON a.ordernumber = b.ordernumber</strong></p>
<p><strong>JOIN tbDate c ON a.dateid = c.dateid</strong></p>
<p><strong>GROUP BY c.theyear, b.itemid</strong></p>
<p><strong>) d</strong></p>
<p><strong>GROUP BY d.theyear</strong></p></td>
</tr>
</tbody>
</table>

|  |
|:---|
| **spark.sql("SELECT d.theyear, MAX(d.SumOfAmount) AS MaxOfAmount FROM (SELECT c.theyear, b.itemid, SUM(b.amount) AS SumOfAmount FROM tbStock a JOIN tbStockDetail b ON a.ordernumber = b.ordernumber JOIN tbDate c ON a.dateid = c.dateid GROUP BY c.theyear, b.itemid ) d GROUP BY d.theyear").show** |

结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>+-------+------------------+</strong></p>
<p><strong>|theyear| MaxOfAmount|</strong></p>
<p><strong>+-------+------------------+</strong></p>
<p><strong>| 2007| 70225.1|</strong></p>
<p><strong>| 2006| 113720.6|</strong></p>
<p><strong>| 2004|53401.759999999995|</strong></p>
<p><strong>| 2009| 30029.2|</strong></p>
<p><strong>| 2005|56627.329999999994|</strong></p>
<p><strong>| 2010| 4494.0|</strong></p>
<p><strong>| 2008| 98003.60000000003|</strong></p>
<p><strong>+-------+------------------+</strong></p></td>
</tr>
</tbody>
</table>

第三步：用最大销售额和统计好的每个货品的销售额join，以及用年join，集合得到最畅销货品那一行信息

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>SELECT DISTINCT e.theyear, e.itemid, f.MaxOfAmount</strong></p>
<p><strong>FROM (SELECT c.theyear, b.itemid, SUM(b.amount) AS SumOfAmount</strong></p>
<p><strong>FROM tbStock a</strong></p>
<p><strong>JOIN tbStockDetail b ON a.ordernumber = b.ordernumber</strong></p>
<p><strong>JOIN tbDate c ON a.dateid = c.dateid</strong></p>
<p><strong>GROUP BY c.theyear, b.itemid</strong></p>
<p><strong>) e</strong></p>
<p><strong>JOIN (SELECT d.theyear, MAX(d.SumOfAmount) AS MaxOfAmount</strong></p>
<p><strong>FROM (SELECT c.theyear, b.itemid, SUM(b.amount) AS SumOfAmount</strong></p>
<p><strong>FROM tbStock a</strong></p>
<p><strong>JOIN tbStockDetail b ON a.ordernumber = b.ordernumber</strong></p>
<p><strong>JOIN tbDate c ON a.dateid = c.dateid</strong></p>
<p><strong>GROUP BY c.theyear, b.itemid</strong></p>
<p><strong>) d</strong></p>
<p><strong>GROUP BY d.theyear</strong></p>
<p><strong>) f ON e.theyear = f.theyear</strong></p>
<p><strong>AND e.SumOfAmount = f.MaxOfAmount</strong></p>
<p><strong>ORDER BY e.theyear</strong></p></td>
</tr>
</tbody>
</table>

|  |
|:---|
| **spark.sql("SELECT DISTINCT e.theyear, e.itemid, f.maxofamount FROM (SELECT c.theyear, b.itemid, SUM(b.amount) AS sumofamount FROM tbStock a JOIN tbStockDetail b ON a.ordernumber = b.ordernumber JOIN tbDate c ON a.dateid = c.dateid GROUP BY c.theyear, b.itemid ) e JOIN (SELECT d.theyear, MAX(d.sumofamount) AS maxofamount FROM (SELECT c.theyear, b.itemid, SUM(b.amount) AS sumofamount FROM tbStock a JOIN tbStockDetail b ON a.ordernumber = b.ordernumber JOIN tbDate c ON a.dateid = c.dateid GROUP BY c.theyear, b.itemid ) d GROUP BY d.theyear ) f ON e.theyear = f.theyear AND e.sumofamount = f.maxofamount ORDER BY e.theyear").show** |

结果如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>+-------+--------------+------------------+</strong></p>
<p><strong>|theyear| itemid| maxofamount|</strong></p>
<p><strong>+-------+--------------+------------------+</strong></p>
<p><strong>| 2004|JY424420810101|53401.759999999995|</strong></p>
<p><strong>| 2005|24124118880102|56627.329999999994|</strong></p>
<p><strong>| 2006|JY425468460101| 113720.6|</strong></p>
<p><strong>| 2007|JY425468460101| 70225.1|</strong></p>
<p><strong>| 2008|E2628204040101| 98003.60000000003|</strong></p>
<p><strong>| 2009|YL327439080102| 30029.2|</strong></p>
<p><strong>| 2010|SQ429425090101| 4494.0|</strong></p>
<p><strong>+-------+--------------+------------------+</strong></p></td>
</tr>
</tbody>
</table>

## 7. SparkSQL整合Hive

sparksql可以使用hive的元数据库，如果没有，sparksql也可以自己创建。

在mysql创建一个普通用户（也可以使用root用户）

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">SQL<br />
# 创建一个普通用户，并且授权<br />
CREATE USER 'spark'@'%' IDENTIFIED BY 'DoIt123!@#';<br />
GRANT ALL PRIVILEGES ON hivedb.* TO 'spark'@'%' IDENTIFIED BY 'DoIt123!@#' WITH GRANT OPTION;<br />
FLUSH PRIVILEGES;</td>
</tr>
</tbody>
</table>

添加一个hive-site.xml到spark的conf目录，里面的内容如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">XML<br />
&lt;?xml version="1.0" encoding="UTF-8" standalone="no"?&gt;<br />
&lt;?xml-stylesheet type="text/xsl" href="configuration.xsl"?&gt;<br />
&lt;configuration&gt;<br />
&lt;property&gt;<br />
&lt;name&gt;javax.jdo.option.ConnectionURL&lt;/name&gt;<br />
&lt;value&gt;jdbc:mysql://node-1.51doit.cn:3306/hivedb?createDatabaseIfNotExist=true&lt;/value&gt;<br />
&lt;description&gt;JDBC connect string for a JDBC metastore&lt;/description&gt;<br />
&lt;/property&gt;<br />
<br />
&lt;property&gt;<br />
&lt;name&gt;javax.jdo.option.ConnectionDriverName&lt;/name&gt;<br />
&lt;value&gt;com.mysql.jdbc.Driver&lt;/value&gt;<br />
&lt;description&gt;Driver class name for a JDBC metastore&lt;/description&gt;<br />
&lt;/property&gt;<br />
<br />
&lt;property&gt;<br />
&lt;name&gt;javax.jdo.option.ConnectionUserName&lt;/name&gt;<br />
&lt;value&gt;spark&lt;/value&gt;<br />
&lt;description&gt;username to use against metastore database&lt;/description&gt;<br />
&lt;/property&gt;<br />
<br />
&lt;property&gt;<br />
&lt;name&gt;javax.jdo.option.ConnectionPassword&lt;/name&gt;<br />
&lt;value&gt;DoIt123!@#&lt;/value&gt;<br />
&lt;description&gt;password to use against metastore database&lt;/description&gt;<br />
&lt;/property&gt;<br />
<br />
&lt;property&gt;<br />
&lt;name&gt;hive.metastore.schema.verification&lt;/name&gt;<br />
&lt;value&gt;false&lt;/value&gt;<br />
&lt;/property&gt;<br />
&lt;property&gt;<br />
&lt;name&gt;datanucleus.schema.autoCreateAll&lt;/name&gt;<br />
&lt;value&gt;true&lt;/value&gt;<br />
&lt;/property&gt;<br />
&lt;property&gt;<br />
&lt;name&gt;hive.metastore.warehouse.dir&lt;/name&gt;<br />
&lt;value&gt;hdfs://node-1.51doit.cn:9000/user/hive/warehouse&lt;/value&gt;<br />
&lt;/property&gt;<br />
&lt;/configuration&gt;</td>
</tr>
</tbody>
</table>

上传一个mysql连接驱动,可以将连接驱动放入到spark的安装包的jars或者使用--driver-class-path指定mysql连接驱动的位置

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Shell<br />
bin/spark-sql --master spark://node-4:7077,node-5:7077 --driver-class-path /root/mysql-connector-java-5.1.47.jar</td>
</tr>
</tbody>
</table>

重新启动SparkSQL的命令行

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Shell<br />
bin/spark-sql --master spark://node-1.51doit.cn:7077 --driver-class-path /root/mysql-connector-java-5.1.49.jar</td>
</tr>
</tbody>
</table>

Spark SQL也提供JDBC连接支持，这对于让商业智能(BI)工具连接到Spark集群上以及在多用户间共享一个集群的场景都非常有用。JDBC 服务器作为一个独立的Spark 驱动器程序运行，可以在多用户之间共享。任意一个客户端都可以在内存中缓存数据表，对表进行查询。集群的资源以及缓存数据都在所有用户之间共享。

Spark SQL的JDBC服务器与Hive中的HiveServer2相一致。由于使用了Thrift通信协议，它也被称为“Thrift server”。

服务器可以通过 Spark 目录中的 sbin/start-thriftserver.sh 启动。这个 脚本接受的参数选项大多与 spark-submit 相同。默认情况下，服务器会在 localhost:10000 上进行监听，我们可以通过环境变量(HIVE_SERVER2_THRIFT_PORT 和 HIVE_SERVER2_THRIFT_BIND_HOST)修改这些设置，也可以通过 Hive配置选项(hive. server2.thrift.port 和 hive.server2.thrift.bind.host)来修改。

你也可以通过命令行参数：--hiveconf property=value来设置Hive选项。

在 Beeline 客户端中，你可以使用标准的 HiveQL 命令来创建、列举以及查询数据表。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Shell<br />
# spark-sql 启动HiveServer2<br />
<br />
#stand alone 模式<br />
sbin/start-thriftserver.sh --master spark://node-1.51doit.cn:7077 --executor-memory 1g --total-executor-cores 8 --driver-class-path /root/mysql-connector-java-5.1.49.jar<br />
<br />
# on yarn 模式<br />
sbin/start-thriftserver.sh --master yarn --deploy-mode client --driver-memory 2g --driver-cores 2 --executor-memory 2g --num-executors 3 --driver-class-path /root/mysql-connector-java-5.1.49.jar</td>
</tr>
</tbody>
</table>

Spark的ThriftServer的原理（类似HiveServer2服务）

<img src="SparkSQL _assets/media/image39.png" style="width:5.90625in;height:4.04167in" />

<img src="SparkSQL _assets/media/image40.png" style="width:5.90625in;height:1.47917in" />

启动beeline客户端连接ThriftServer

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Shell<br />
<br />
#使用beline连接HiveServer<br />
<br />
bin/beeline -u jdbc:hive2://node-1.51doit.cn:10000 -n root</td>
</tr>
</tbody>
</table>
