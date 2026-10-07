# 第5节　pandas文件操作

<!-- 来源：http://www.zlkt.net/book/detail/11/376 -->

> 出自《Python数据分析到项目实战》第3章 Pandas库

---

## 文件操作：

Pandas中提供了许多的操作文件的函数，包括读取和写入。我们做数据分析用得最多的，就是`CSV`、`Excel`、`SQL`、`JSON`文件。下面来针对这几种文件的操作做一个详细的讲解。

### CSV文件操作：

读写`CSV`文件分别用的是`pd.read_csv`和`pd.to_csv`方法。普通用法非常简单，但是通过一些参数，可以实现许多高级操作。

#### 1. 读取csv：

读取`csv`用的是`pd.read_csv`，主要有以下参数：

1. `filepath_or_buffer`：文件路径，或者是有`read`方法的流对象。
2. `sep`：分隔符，默认是`,`。
3. `header`：指定哪行作为列的名称，如果没有行作为列名，那么应该设置header=None，并且设置names参数。
4. `names`：在csv文件中没有一行来存储列名，可以使用names自己指定，并且设置header=None。
5. `index_col`：使用哪一列作为行索引，可以是列的位置，也可以是列的名称。如果没有指定，那么默认会自动生成一个顺序索引。
6. `usecols`：加载哪几列。比如有时候只想要csv文件中的某几列，那么就可以使用`usecols`。也可以是个函数，这个函数返回True的列会被保留，否则会丢弃。
7. `engine`：csv解析引擎，有C和Python，C速度更快，但是Python功能更完善。
8. `dtype`：指定某些列的类型。
9. `converters`：转换器列表，可以指定每一列在加载的时候就转换为指定的类型。
10. `encoding`：使用指定的编码方式打开文件。
11. `chunksize`：使用迭代器的方式读取，一次返回多少行的数据。

更多参数请查看Pandas官网`read_csv`：https://pandas.pydata.org/docs/user\_guide/io.html#io-read-csv-table

#### 2. 写入csv：

写入`csv`用的是`pd.to_csv`，`Series`和`DataFrame`都可以使用这个方法。主要有以下参数：

1. `path_or_buf`：写入的文件路径、缓存或者是文件对象，如果是文件对象，那么这个文件对象在打开的时候必须指定`newline=''`。
2. `sep`：存储成csv文件格式化的分隔符。
3. `na_rep`：`NAN`值的替代字符串，默认是空的。
4. `float_format`：格式化浮点类型字符串。
5. `columns`：哪些列需要写入到csv文件中。
6. `header`：是否把列的名称也写入进去，默认为True。
7. `index`：是否把行索引名称也写入进去，默认为True。
8. `encoding`：存储的csv文件编码方式。
9. `chunksize`：一次性写入多少行。

更多的参数请查看Pandas官网`to_csv`：https://pandas.pydata.org/docs/user\_guide/io.html#io-store-in-csv

### Excel文件操作：

#### 1. 读取Excel：

读取`Excel`文件用的是`pd.read_excel`方法。因为Excel文件有两种类型，分别是2007年后的`.xlsx`和2003年的`.xls`。其中`.xlsx`需要借助`openpyxl`库，`.xls`需要借助`xlrd`库。如果这两个库没有安装，在运行的时候可能会报错，因此可以提前通过以下命令安装：

```
$ pip install openpyxl
$ pip install xlrd
```

下面来讲解一下`pd.read_excel`的基本用法。先看一个最简单的示例代码：

```python
pd.read_excel("path_to_file.xls", sheet_name="Sheet1")
```

首先指定Excel文件路径，然后通过`sheet_name`参数指定读取Excel文件的哪个Sheet。如果Excel的Sheet比较多，那么我们可以先使用`ExcelFile`类把所有Sheet都读取出来，再使用`pd.read_excel`分别读取。这种方式比直接直接多次使用`pd.read_excel`效率更高。示例代码如下：

```python
xlsx = pd.ExcelFile("path_to_file.xls")
df = pd.read_excel(xlsx, "Sheet1")
```

或者是：

```python
with pd.ExcelFile("path_to_file.xls") as xls:
    df1 = pd.read_excel(xls, "Sheet1")
    df2 = pd.read_excel(xls, "Sheet2")
```

`pd.read_excel`的其他参数，除了没有`chunksize`外，与`pd.read_csv`相同。

更多`pd.read_excel`的用法，请参考Pandas官方文档：<https://pandas.pydata.org/docs/user_guide/io.html#io-excel-reader>

#### 2. 写入Excel：

写入`Excel`用的是`pd.to_excel`方法，使用方法与`pd.to_csv`非常的类似，但是可以指定`sheet_name`参数。并且官方强烈建议使用`openpyxl`库作为引擎，将数据保存保存为`xlsx`文件，如果使用`xlwt`库将数据保存为`xls`文件，出现问题官方是不会负责修复的，因为已经不支持了（具体可以看：https://pandas.pydata.org/docs/user\_guide/io.html#excel-files）。
基本用法如下：

```python
df.to_excel("path_to_file.xlsx", sheet_name="Sheet1")
```

也可以使用`ExcelWriter`类，写入多个`DataFrame`到多个`Sheet`中。示例代码如下：

```python
with pd.ExcelWriter("path_to_file.xlsx") as writer:
    df1.to_excel(writer, sheet_name="Sheet1")
    df2.to_excel(writer, sheet_name="Sheet2")
```

更多`pd.to_excel`的用法，请参考Pandas官方文档：<https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to_excel.html#pandas.DataFrame.to_excel>

### SQL操作：

操作`SQL`主要有以下几个函数：

1. `read_sql_table(table_name, conn, ...)`：用于读取一张表中的数据。
2. `read_sql_query(sql, con, ...)`：执行SQL语句，读取到DataFrame中。
3. `read_sql(sql_or_tablename, con, ...)`：`read_sql_table`和`read_sql_query`两者结合，会自动判断第一个参数是否为表名还是sql语句。
4. `DataFrame.to_sql(name, con, ...)`：将`DataFrame`数据写入到数据库中。`name`指定一个表名。

操作`SQL`需要借助`SQLAlchemy`，如果没有安装，则需要通过`pip install sqlalchemy`安装一下。使用`SQLAlchemy`连接数据库，会根据不同的数据库，使用不同的连接方式。

1. `SQLite`：`"sqlite:///[database name].db"`
2. `MySQL`：`"mysql+pymysql://[username]:[password]@[host]:[port]/[database name]?charset=utf8"`
3. `PostgreSQL`：`"postgresql://[username]:[password]@[host]:[port]/[database name]"`
4. `Oracle`：`"oracle://scott:tiger@127.0.0.1:1521/sidname"`

这里我们以`sqlite`为例来讲解上述方法的使用。

#### 1. 写入SQL：

为了方便演示SQL操作，我们先使用`to_sql`，将一个`DataFrame`数据写入到数据库中。示例代码如下：

```python
# 读取excel中的数据
df = pd.read_excel("data/salesfunnel.xlsx")

# 创建数据库连接
from sqlalchemy import create_engine
engine = create_engine("sqlite:///salessfunnel.db")

# 将df数据，写入到sqlite数据库中，并且表名为funnel
df.to_sql("funnel", engine, index=False)
```

#### 2. read\_sql\_table读取表数据：

使用`read_sql_table`可以将指定表中所有的数据都读取出来（当然也可以使用chunksize来分段读取）。示例代码如下：

```python
funnel = pd.read_sql_table("funnel", engine)
```

如果使用`chunksize`参数，那么将返回一个生成器。可以循环获取里面的数据，示例代码如下：

```python
funnel = pd.read_sql_table("funnel", engine, chunksize=3)
for chunk in funnel:
    print(chunk)
```

在循环`funnel`的时候，每个`chunk`都是含有三条数据的`DataFrame`对象。

#### 3. read\_sql\_query执行SQL查询语句：

在获取数据的时候，如果需要过滤数据，或者是多表查询，那么可以使用`read_sql_query`来执行`sql`语句。示例代码如下：

```python
df2 = pd.read_sql_query("select Account, Name from funnel where Product='CPU'", engine)
print(df2)
```

输出结果如下：

```
        Account	Name
0	714466	Trantow-Barrows
1	737550	Fritsch, Russel and Anderson
2	146832	Kiehn-Spinka
3	218895	Kulas Inc
4	740150	Barton LLC
5	141962	Herman LLC
6	163416	Purdy-Kunde
7	688981	Keeling LLC
8	729833	Koepp Ltd
```

#### 4. read\_sql读取数据：

`read_sql`是结合了`read_sql_table`和`read_sql_query`两者的功能。既可以从表中读取数据，也可以执行查询SQL语句。示例代码如下：

```python
# 读取表数据
funnel = pd.read_sql("funnel", engine)
# 执行查询SQL语句
funnel = pd.read_sql("select Account, Name from funnel where Product='CPU'", engine)
```

### 更多：

更多其他文件操作，请参考Pandas官网：<https://pandas.pydata.org/docs/user_guide/io.html>
