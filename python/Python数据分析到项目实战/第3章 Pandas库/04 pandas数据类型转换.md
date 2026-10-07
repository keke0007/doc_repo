# 第4节　pandas数据类型转换

<!-- 来源：http://www.zlkt.net/book/detail/11/375 -->

> 出自《Python数据分析到项目实战》第3章 Pandas库

---

## Pandas数据类型转换

### 一、Pandas中的数据类型：

不管是`Series`还是`DataFrame`的每一列，都有对应的数据类型。在`Pandas`中存在以下数据类型。

| Pandas dtype | Python 类型 | Numpy类型 | 描述 |
| --- | --- | --- | --- |
| object | str或者mixed（混合类型） | string_, unicode_, mixed类型 | 文本或者是混合的数值或非数值类型 |
| int64 | int | int_, int8, int16, int32, int64, uint8, uint16, uint32, uint64 | 整数类型 |
| float64 | float | float_, float16, float32, float64 | 浮点类型 |
| bool | bool | bool_ | 布尔类型 |
| datetime64 | NA | datetime | 日期和时间类型 |
| timedelta | NA | NA | 时间差 |
| category | NA | NA | 有限的列表文本值（分类） |

### 案例数据文件：

这里我们以一个`sales_data_types.csv`文件为例。来讲解后面的知识点。读取代码如下：

```python
import pandas as pd
import numpy as np

df = pd.read_csv("data/sales_data_types.csv")
df.head()
```

输出结果为：
![sales_data_types.png](../images/sales_data_types.png)

### 数据类型相关操作：

#### 1. 查看DataFrame所有列的类型：

通过`df.dtypes`或者是`df.info`，即可查看`df`对象的类型。输入`df.dtypes`输出结果如下：

```
Customer Number    float64
Customer Name       object
2016                object
2017                object
Percent Growth      object
Jan Units           object
Month                int64
Day                  int64
Year                 int64
Active              object
dtype: object
```

输入`df.info()`输出结果如下：

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 5 entries, 0 to 4
Data columns (total 10 columns):
 #   Column           Non-Null Count  Dtype
---  ------           --------------  -----
 0   Customer Number  5 non-null      float64
 1   Customer Name    5 non-null      object
 2   2016             5 non-null      object
 3   2017             5 non-null      object
 4   Percent Growth   5 non-null      object
 5   Jan Units        5 non-null      object
 6   Month            5 non-null      int64
 7   Day              5 non-null      int64
 8   Year             5 non-null      int64
 9   Active           5 non-null      object
dtypes: float64(1), int64(3), object(6)
memory usage: 528.0+ bytes
```

#### 2. 用astype转换类型：

使用`astype`可以非常方便的转换类型，比如将`Customer Number`转换为整形：

```python
df['Customer Number'].astype('int')
```

以上代码并不会真正改变`df['Customer Number']`的类型，如果想要真正改变，则需要重新进行赋值：

```
df['Customer Number'] = df['Customer Number'].astype('int')
```

此时再查看`df.dtypes`，就可以看到数据类型已经发生改变了：

```
Customer Number     int64
Customer Name      object
2016               object
2017               object
Percent Growth     object
Jan Units          object
Month               int64
Day                 int64
Year                int64
Active             object
dtype: object
```

#### 3. 自定义转换函数：

像`astype`只能转换那些格式正确的数据。比如如果直接将`df['2016']`转换为浮点类型，那么会报错：

```python
df['2016'].astype('float')
```

会报类似以下的错误：

```
ValueError       Traceback (most recent call last)
<ipython-input-45-999869d577b0> in <module>()
----> 1 df['2016'].astype('float')

[lots more code here]

ValueError: could not convert string to float: '$15,000.00'
```

这是因为在`2016`这一列中，有`$`和逗号，直接强制转换会抛出异常。这时候就需要使用自定义转换函数，把`$`去掉，然后再转换。代码如下：

```python
def convert_currency(val):
    """
    转换字符串类型为浮点类型
     - 移除 $符号
     - 移除逗号
     - 转换为浮点类型
    """
    new_val = val.replace(',','').replace('$', '')
    return float(new_val)

df['2016'].apply(convert_currency)
```

以上代码，也可以将`convert_currency`函数使用`lambda`表达式来替换。示例代码如下：

```python
df['2016'].apply(lambda x: x.replace('$', '').replace(',', '')).astype('float')
```

#### 4. 使用`np.where`更换数据类型：

比如`df['Active']`这列，我们可以认为只要值是`Y`，那么就设置为`True`，否则就设置为`False`。代码如下：

```python
np.where(df['Active']=='Y', True, False)
```

#### 5. pandas工具类函数：

##### pd.to\_numeric函数：

`pd.to_numeric`函数是用于将数据转换为数值类型，他的功能更加丰富一些，我们先来看下这个函数定义的参数：

```python
pd.to_numeric(data, errors, downcast)
```

1. `data`：需要进行类型转换的数据。
2. `errors`：在发生转换错误时的处理方式。有`ignore`、`raise`、`coerce`可选，默认类型为`raise`，其中`coerce`代表在发生转换异常的时候，会使用`NAN`来代替。
3. `downcast`：期望转换的类型。有`integer`、`signed`、`unsigned`、`float`可选，默认值为None。如果为None，函数会自动判断需要转换的类型。这个参数设置后，不一定会按照设置的类型来转换，比如在转换的时候出现了NAN值，我们都知道NAN值是float类型，这时候如果你指定为`integer`也没有任何效果。

示例代码如下：

```python
pd.to_numeric(df['Jan Units'], errors='coerce', downcast="integer")
```

输出结果如下：

```
0    500.0
1    700.0
2    125.0
3     75.0
4      NaN
Name: Jan Units, dtype: float64
```

可以看到虽然我们设置了类型为`integer`，但最终还是`float64`，原因是在转换`Jan Units`字段的时候，最后一个数据出现了`NAN`。

如果不想让转换失败的值为`NAN`，比如想用`0`来填充。那么可以使用`fillna`来实现。示例代码如下：

```python
pd.to_numeric(df['Jan Units'], errors='coerce').fillna(0)
```

##### pd.to\_datetime函数：

这个函数功能非常强大，可以将以下类型转换为`datetime`类型：

1. int、floats时间戳类型。
2. 时间格式的字符串类型。
3. np.array一维数组、列表或者元组。
4. Series、DataFrame或者字典类型。

下面分别来进行讲解。

1. int、floats时间戳类型。
   必须指定`unit`参数为`s`，也就是秒。也可以指定为`ms`，代表毫秒，`ns`为纳秒（1毫秒=10^6纳秒）。

```python
# 整形
pd.to_datetime(1642400714, unit="s")
# 浮点类型
pd.to_datetime(1642400714.3847, unit="s")
# 毫秒
pd.to_datetime(1642400714111, unit="s")
```

2. 时间格式的字符串类型。时间格式可以参考：https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior

```python
# 将字符串按照指定格式转换为datetime类型
pd.to_datetime('20220101', format='%Y%m%d')
```

3. np.array一维数组、列表或者元组。

```python
# 根据原始时间转换
pd.to_datetime([1, 2, 3], unit='D',
              origin=pd.Timestamp('2022-01-01'))
```

输出结果为：

```
DatetimeIndex(['2022-01-02', '2022-01-03', '2022-01-04'], dtype='datetime64[ns]', freq=None)
```

或者直接将列表中的字符串转换为时间类型：

```python
pd.to_datetime(['2018-10-26 12:00 -0530', '2018-10-26 12:00 -0500'])
```

输出结果为：

```
Index([2018-10-26 12:00:00-05:30, 2018-10-26 12:00:00-05:00], dtype='object')
```

4. Series或者DataFrame类型。

```python
s = pd.Series(['3/11/2000', '3/12/2000', '3/13/2000'])
pd.to_datetime(s, infer_datetime_format=True)
```

其中`infer_datetime_format`代表自动推测时间格式。
输出结果为：

```
0   2000-03-11
1   2000-03-12
2   2000-03-13
dtype: datetime64[ns]
```

### 综合在一起：

我们可以把转换数据类型的工作，在一开始读取文件的时候就指定好。示例代码如下：

```python
def convert_percent(val):
    """
    转化%的字符串为浮点类型
    - 移除 %
    - 除以100
    """
    new_val = val.replace('%', '')
    return float(new_val) / 100

df_2 = pd.read_csv("data/sales_data_types.csv",
                   dtype={'Customer Number': 'int'},
                   converters={'2016': convert_currency,
                               '2017': convert_currency,
                               'Percent Growth': convert_percent,
                               'Jan Units': lambda x: pd.to_numeric(x, errors='coerce'),
                               'Active': lambda x: np.where(x == "Y", True, False)
                              })
```
