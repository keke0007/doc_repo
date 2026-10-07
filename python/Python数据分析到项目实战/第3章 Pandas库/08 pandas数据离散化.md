# 第8节　pandas数据离散化

<!-- 来源：http://www.zlkt.net/book/detail/11/379 -->

> 出自《Python数据分析到项目实战》第3章 Pandas库

---

## pandas数据离散化

数据离散化，是将连续的数据，通过分割，形成离散化的数据。举个例子，比如有一列数据存储人的身高：`165，174，160，180，159，163，192，184`，那么通过离散化可以变为：`150~165, 165~180,180~195`。还有另外一种离散化的数据，就是通过`one-hot`编码，下面会详细讲到。

### 切割数据离散化：

在`pandas`中使用`pd.qcut`或者是`pd.cut`方法实现数据切割。
`pd.qcut(data, q)`的函数意义为：

- `data`：需要被切割的数据。
- `q`：需要切割多少个组。

示例代码如下：

```python
df = pd.read_csv("data/stock_day.csv")
qcut = pd.qcut(df['p_change'], 6)
qcut.value_counts()
```

输出结果如下：

```
(-10.030999999999999, -4.836]    65
(-0.462, 0.26]                   65
(0.26, 0.94]                     65
(5.27, 10.03]                    65
(-4.836, -2.444]                 64
(-2.444, -1.352]                 64
(-1.352, -0.462]                 64
(1.738, 2.938]                   64
(2.938, 5.27]                    64
(0.94, 1.738]                    63
Name: p_change, dtype: int64
```

也可以自己指定切割的区间和数量。这时候可以使用`pd.cut`实现。
`pd.cut(data, bins)`参数意义如下：

- `data`：需要被切割的数据。
- `bins`：切割的区间列表。

示例代码如下：

```python
cut = pd.cut(df["p_change"], bins=[-10, -5, 0, 5, 10, 15])
cut.value_counts()
```

输出结果如下：

```
(0, 5]       272
(-5, 0]      239
(5, 10]       60
(-10, -5]     51
(10, 15]      10
Name: p_change, dtype: int64
```

### One-Hot编码离散化：

`One-Hot`编码是将分类数据的所有项，全部都变成列，然后如果某一行中出现这一列，那么就标记为1，否则就标记为0。比如下图将左边的`Category`变为`One-Hot`编码后，会把`Category`中所有唯一的值都添加为新的列。
![one_hot编码.png](../images/one_hot编码.png)

`One-Hot`编码在机器学习中经常用到，用于预测分类等。在`pandas`中，可以通过`pd.get_dummies(data, prefix=None)`来实现，参数如下：

- `data`：需要被执行`One-Hot`编码的数据。
- `prefix`：分组名称前缀。

示例代码如下：

```python
df = pd.read_excel("data/salesfunnel.xlsx")
pd.get_dummies(df['Product'], prefix="Product")
```

输出结果如下：

```
Product_CPU	Product_Maintenance	Product_Monitor	Product_Software
0	1	0	0	0
1	0	0	0	1
2	0	1	0	0
3	1	0	0	0
4	1	0	0	0
5	1	0	0	0
6	0	0	0	1
7	0	1	0	0
8	1	0	0	0
9	1	0	0	0
10	1	0	0	0
11	0	1	0	0
12	0	0	0	1
13	0	1	0	0
14	1	0	0	0
15	1	0	0	0
16	0	0	1	0
```
