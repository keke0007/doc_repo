# 第3节　pandas索引操作

<!-- 来源：http://www.zlkt.net/book/detail/11/374 -->

> 出自《Python数据分析到项目实战》第3章 Pandas库

---

## Pandas索引操作

`Pandas`中的索引操作非常灵活，功能非常强大。学会他的索引操作能帮助我们更好的处理数据。下面来对索引进行讲解。

### 一、索引类型：

不管是`Series`还是`DataFrame`，索引对象的类型都是`Index`或者其子类。我们可以通过以下代码查看：

```python
import pandas as pd
import numpy as np

df = pd.DataFrame(np.random.rand(4,4))

print(type(df.index))
print(type(df.columns))
```

输出结果为：

```
RangeIndex(start=0, stop=4, step=1)
RangeIndex(start=0, stop=4, step=1)
```

可以看到行和列的类型，都是`RangeIndex`类型。`RangeIndex`属于`Index`的子类。当然我们也可以直接通过显示创建`Index`的方式，修改`df`的`index`和`columns`，示例代码如下：

```python
df.index = pd.Index(list("ABCD"))
df.columns = pd.Index(list("abcd"))
print(df)
```

输出结果为：

|  | a | b | c | d |
| --- | --- | --- | --- | --- |
| A | 0.274963 | 0.084407 | 0.157835 | 0.797312 |
| B | 0.090830 | 0.512263 | 0.419373 | 0.466661 |
| C | 0.903084 | 0.367636 | 0.219719 | 0.258690 |
| D | 0.009205 | 0.631668 | 0.495482 | 0.316959 |

常用的`Index`类型还有以下。

#### 1. RangeIndex：

区间索引，用法与Python中的`range`函数类似，可以指定`start`、`stop`、`step`参数。示例代码如下：

```python
df.index = pd.RangeIndex(start=1, stop=9, step=2)
```

#### 1. NumericIndex：

数值类型的索引，包括有浮点类型的`Float64Index`、整形的`Int64Index`、无符号整形的`UInt64Index`、序列类型的`RangeIndex`。他们的用法如下：

```python
# 浮点类型
>>> pd.Float64Index([1,2,3,4])
Float64Index([1.0, 2.0, 3.0, 4.0], dtype="float64")

# 整数
>>> pd.Int64Index([1,2,3,4])
Int64Index([1, 2, 3, 4], dtype="int64")

# 无符号整数
>>> pd.UInt64Index([1,2,3,4])
UInt64Index([1, 2, 3, 4], dtype="uint64")
```

其中`Float64Index`、`Int64Index`、`UInt64Index`在`Pandas 2.0`版本中会被移除，统一使用`NumericIndex`代替。

#### 2. CategoricalIndex：

分类索引，索引的值只能是指定分类的。否则会用NAN来代替。示例代码如下：

```python
>>> df.index = pd.CategoricalIndex(list("ABCD"),categories=list("ABCD"))
```

输出结果为：

```
    a        b        c        d
A    0.274963    0.084407    0.157835    0.797312
B    0.090830    0.512263    0.419373    0.466661
C    0.903084    0.367636    0.219719    0.258690
D    0.009205    0.631668    0.495482    0.316959
```

如果将索引值修改为`list("ABCE")`，因为`E`不在`categories`参数指定的范围内，因此会用NAN来代替。

```python
df.index = pd.CategoricalIndex(list("ABCE"),categories=list("ABCD"))
```

输出结果为：

```
    a        b        c        d
A    0.274963    0.084407    0.157835    0.797312
B    0.090830    0.512263    0.419373    0.466661
C    0.903084    0.367636    0.219719    0.258690
NaN    0.009205    0.631668    0.495482    0.316959
```

关于`CategoricalIndex`的更多用法请参考官方文档：<https://pandas.pydata.org/docs/reference/api/pandas.CategoricalIndex.html>

#### 3. IntervalIndex：

间隔索引，索引的值为一个区间，可以通过`pd.interval_range`函数创建。示例代码如下：

```python
df.index = pd.interval_range(start=0, end=4)
```

输出结果为：

```
    a        b        c        d
(0, 1]    0.274963    0.084407    0.157835    0.797312
(1, 2]    0.090830    0.512263    0.419373    0.466661
(2, 3]    0.903084    0.367636    0.219719    0.258690
(3, 4]    0.009205    0.631668    0.495482    0.316959
```

`interval_range`函数的`start`和`end`参数，也可以为`datetime`类型，并且还可以通过`periods`参数指定区间的个数。示例代码如下：

```python
from datetime import datetime
pd.interval_range(start=datetime(year=2022, month=1, day=1), end=datetime(year=2022, month=1, day=31), periods=4)
```

输出结果如下：

```
IntervalIndex([(2022-01-01, 2022-01-11], (2022-01-11, 2022-01-21], (2022-01-21, 2022-01-31]],
              closed='right',
              dtype='interval[datetime64[ns]]')
```

关于更多`interval_range`和`IntervalIndex`的用法，请参考官方文档：

1. IntervalIndex：<https://pandas.pydata.org/docs/reference/api/pandas.IntervalIndex.html>
2. interval\_range：<https://pandas.pydata.org/docs/reference/api/pandas.interval_range.html>

#### 4. DatetimeIndex：

日期时间索引，可以通过`pd.date_range`函数创建。示例代码如下：

```python
df.index = pd.date_range("2022-01-01", periods=4, freq="Y")
```

输出结果为：

```
        a            b            c            d
2022-12-31    0.274963    0.084407    0.157835    0.797312
2023-12-31    0.090830    0.512263    0.419373    0.466661
2024-12-31    0.903084    0.367636    0.219719    0.258690
2025-12-31    0.009205    0.631668    0.495482    0.316959
```

其中`freq`参数默认是`D`，也就是天，也可以选择日，时分秒等。以下链接可以查看所有的选择：<https://pandas.pydata.org/docs/user_guide/timeseries.html#timeseries-offset-aliases>

关于`date_range`与`DatetimeIndex`的更多用法请参考官方文档：

1. date\_range：<https://pandas.pydata.org/docs/reference/api/pandas.date_range.html>
2. DatetimeIndex：<https://pandas.pydata.org/docs/reference/api/pandas.DatetimeIndex.html>

#### 5. TimedeltaIndex：

时间间隔索引。可以通过`pd.TimedeltaIndex`创建。示例代码如下：

```python
df.index = pd.TimedeltaIndex([12,24,36,48], unit="m")
```

输出结果为：

```
        a            b            c            d
0 days 00:12:00    0.274963    0.084407    0.157835    0.797312
0 days 00:24:00    0.090830    0.512263    0.419373    0.466661
0 days 00:36:00    0.903084    0.367636    0.219719    0.258690
0 days 00:48:00    0.009205    0.631668    0.495482    0.316959
```

以上便是常用的索引类型。索引类型有一个特点，**一旦索引被创建后，将无法进行修改。** 示例代码如下：

```python
df.index[0] = 2
```

执行上述代码，将会抛出类似以下的错误信息：

```
TypeError: Index does not support mutable operations
```

关于`TimedeltaIndex`的更多用法请参考官方文档：<https://pandas.pydata.org/docs/reference/api/pandas.TimedeltaIndex.html>

### 二、Series索引：

在创建`Series`对象的时候，默认的索引值是0-N，我们也可以通过`index`参数单独设置。示例代码如下：

```python
series = pd.Series(range(5), index = ['a', 'b', 'c', 'd', 'e'])
print(series.head())
```

输出结果为：

```
a    0
b    1
c    2
d    3
e    4
dtype: int64
```

#### 1. 行索引：

因为在`Series`中，只有一列，因此不存在列索引。行索引可以通过索引名称获取，也可以通过索引下标获取。示例代码如下：

```python
series = pd.Series(range(0, 10, 2), index = ['a', 'b', 'c', 'd', 'e'])
print(series)
print(series['a'])
print(series[1])
```

输出结果为：

```
a    0
b    2
c    4
d    6
e    8
dtype: int64
0
2
```

如果索引是时间类型，则通过时间字符串即可获取到。示例代码如下：

```python
series = pd.Series(range(2, 12, 2))
series.index = pd.date_range("2022-01-01", periods=5, freq="H")
print(series)
print(series["2022-01-01 00:00:00"])
```

输出结果如下：

```
2022-01-01 00:00:00     2
2022-01-01 01:00:00     4
2022-01-01 02:00:00     6
2022-01-01 03:00:00     8
2022-01-01 04:00:00    10
Freq: H, dtype: int64
2
```

#### 2. 切片索引：

索引也可以类似使用列表的切片方式来提取。切片可以是索引名称，也可以是序号。示例代码如下：

```python
import pandas as pd

persons = ['张三','李四','王五']
series = pd.Series(persons, index=list("ABC"))

# 根据索引名切片
print(series["A":"B"])
# 根据索引序号切片
print(series[0:2])
```

以上两个输出结果都为：

```
A    张三
B    李四
dtype: object
```

可以注意到，如果用索引名称进行切片，那么会包含终止索引的。

#### 3. 不连续索引：

之前通过切片可以一次性获取多个索引的值，也可以直接指定具体几个位置的索引。示例代码如下：

```python
import pandas as pd

persons = ['张三','李四','王五','赵六']
series = pd.Series(persons, index=list("ABCD"))

# 获取索引下标为0和2的元素
print(series[[0,2]])
# 获取索引名称为A和C的元素
print(series[["A","C"]])
```

以上两个print语句的代码执行结果如下：

```
A    张三
C    王五
dtype: object
```

#### 4. 布尔索引：

布尔索引，就是提供条件，选择满足条件的值出来。示例代码如下：

```python
import pandas as pd

persons = [18,20,39,45]
series = pd.Series(persons, index=['张三','李四','王五','赵六'])

# 选择值大于20的所有元素
print(series[series>20])
```

输出结果如下：

```
王五    39
赵六    45
dtype: int64
```

### 三、DataFrame索引：

创建`DataFrame`的时候，可以指定行索引和列索引，示例代码如下：

```python
import numpy as np

df = pd.DataFrame(np.random.randn(5,4), columns = ['a', 'b', 'c', 'd'], index=["11","22","33","44","55"])
print(df)
```

输出结果如下：

```
           a         b         c         d
11  0.963458  1.896413  0.042990 -0.582146
22 -1.764354 -1.529342 -0.430965 -0.215617
33  0.356744 -0.729001 -0.543932  0.852026
44  0.488031  0.459878 -0.577119  0.961865
55 -0.808639  0.925949 -1.333124  0.526995
```

下面我们将使用以上的`df`对象进行讲解。

#### 1. 列索引：

`DataFrame`中包含列索引，可以通过以下方式来获取列索引的数据：

```python
# 只获取一列，返回series类型
print(df["a"])
# 获取多列，返回DataFrame类型
print(df[["a", "b"]])
```

#### 2. loc索引：

`Series`通过`[]`来获取行索引，而`DataFrame`通过`[]`获取的是列索引。如果想要获取行索引，则需要通过`loc`或者`iloc`属性来实现，`loc`与`iloc`的区别是，`loc`是通过名称获取，而`iloc`是通过索引下标获取。

1. 获取一行的值

   ```python
   print(df.loc["11"])
   ```

   输出结果为：

   ```
    a   -0.263100
    b   -0.740024
    c    0.223885
    d    1.015902
    Name: 11, dtype: float64
   ```
2. 获取不连续的多行的值

   ```python
   print(df.loc[["11", "33"]])
   ```

   输出结果为：

   ```
        a            b            c            d
   11    1.886025    0.632712    -0.834526    0.919720
   33    0.058799    0.319352    -0.193082    -1.326811
   ```
3. 获取切片

   ```python
   print(df.loc["11":"33"])
   ```

   输出结果为：

   ```
        a                    b            c            d
   11    0.193039    -0.027850    -1.759310    -0.258800
   22    -0.689990    0.201788    -0.782961    -0.092585
   33    0.275034    1.393938    2.123608    -0.009984
   ```

`loc`的除了能获取行索引外，还可以获取列索引。`.loc[row, col]`的第二个参数即是获取列索引。示例代码如下：

```python
# 获取行索引中11:33，"a"列的数据
df.loc["11":"33", "a"]

# 获取行索引中"11"和"a":"c"列的数据
df.loc["11", "a":"b"]

# 获取行索引中"11","33"，和列索引中"a","c"列的数据
df.loc[["11", "33"], ["a", "b"]]

# 获取"a"列的所有数据
df.loc[:,"a"]

# 获取'11'行中的所有列
df.loc['11', :]
```

#### 3. iloc索引：

作用和`loc`一样，区别是通过索引下标来实现的。示例代码如下：

```python
# 获取第1-2行的所有列
df.iloc[1:3]

# 获取第1，3行的所有列
df.iloc[[1,3]]

# 获取第1行的第1-3列
df.iloc[1, 1:4]
```

### 四、重置索引

在`Pandas`中重置索引有三种方法，分别是`set_index`、`reset_index`以及`reindex`以及直接修改`index`属性。我们使用以下测试数据来作为讲解。

```python
df = pd.DataFrame({'month': [1, 4, 7, 10],
                    'year': [2012, 2014, 2013, 2014],
                    'sale':[55, 40, 84, 31]})
```

输出结果如下：

|  | month | year | sale |
| --- | --- | --- | --- |
| 0 | 1 | 2012 | 55 |
| 1 | 4 | 2014 | 40 |
| 2 | 7 | 2013 | 84 |
| 3 | 10 | 2014 | 31 |

#### 1. set\_index：

如果在想使用某列作为`DataFrame`的索引，那么可以使用`set_index(keys, drop=True)`来实现。其中`keys`是用于设置索引列的名称或者列表，`drop`代表是否要删除作为索引的列。**这个方法不会修改原始DataFrame对象。** 示例代码如下：

```python
df.set_index("month")
```

输出结果如下：

```
	year	sale
month
1	2012	55
4	2014	40
7	2013	84
10	2014	31
```

**应用场景：** 需要将`DataFrame`中某列或多列设置为索引的情况下使用。

#### 2. reset\_index：

重新设置新的下标索引。使用`reset_index(drop=False)`来实现。`drop`代表是否删除原始索引。**这个方法不会修改原始DataFrame对象。** 示例代码如下：

```python
df.reset_index()
```

输出结果如下：

```
	month	year	sale
0	1	2012	55
1	4	2014	40
2	7	2013	84
3	10	2014	31
```

**应用场景：** 重新生成新的下标索引。

#### 3. reindex：

在即不使用原有列作为索引，以及不使用新的下标索引的时候。可以使用`reindex`重新指定新的索引。**这个方法不会修改原始DataFrame对象。** 使用`reindex`有以下特点。

1. 如果新添加的索引在原来索引中不存在，那么只会使用`NAN`来代替。
2. 如果新的索引不包含原来某个索引，那么相当于变相删除了这一行的值。
3. 如果新的索引相对于原来索引顺序发生改变，那么相当于变相修改了行的顺序。

示例代码如下：

```python
# 1. 添加新的索引
df.reindex([0,1,2,3,4])

# 2. 删除某个索引的行
df.reindex([0,2,3])

# 3. 修改索引顺序
df.reindex([2,3,1,0])
```

**应用场景：** 设置新的索引、修改索引顺序、删除某些索引。

#### 4. 修改DataFrame的index属性：

直接修改`index`属性也可以实现修改索引的目的，但是他有一个限制，就是新索引的数量，必须和原索引数量一致，否则会报错。**这个方法会修改原始DataFrame对象。** 示例代码如下：

```python
df.index = ['a', 'b', 'c', 'd' ]
```

还有一个需要注意的是，这种方法会直接修改原始`DataFrame`对象。

**应用场景：** 需要修改原始`DataFrame`对象的索引值。
