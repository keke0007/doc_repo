# 第10节　pandas分组与聚合

<!-- 来源：http://www.zlkt.net/book/detail/11/381 -->

> 出自《Python数据分析到项目实战》第3章 Pandas库

---

## Pandas分组与聚合

分组与聚合是做数据分析经常用到的技术。比如我们想要按照班级统计学生英语成绩得`A`的人数，那么要先根据班级进行分组，然后再使用`count`聚合函数计算人数。

这里我们用以下测试数据：

```python
df = pd.DataFrame({'fruit':['apple','banana','orange','apple','banana'],
                    'color':['red','yellow','yellow','cyan','cyan'],
                    'price':[8.5,6.8,5.6,7.8,6.4]})
df
```

输出结果为：

```
	fruit	color	price
0	apple	red		8.5
1	banana	yellow	6.8
2	orange	yellow	5.6
3	apple	cyan	7.8
4	banana	cyan	6.4
```

### 1. 根据`fruit`进行分组：

我们想获取每种水果的平均价格，那么可以先根据`fruit`进行分组，然后求平均值。代码如下：

```python
df.groupby('fruit')['price'].mean()
```

输出结果为：

```
fruit
apple     8.15
banana    6.60
orange    5.60
Name: price, dtype: float64
```

### 2. 根据`fruit`和`color`进行分组：

我们想要获取不同的`fruit`以及`color`的平均价格，那么可以使用以下代码实现：

```python
result = df.groupby(['fruit', 'color'])['price'].mean()
print(type(result.index))
print(result)
```

输出结果为：

```
<class 'pandas.core.indexes.multi.MultiIndex'>

fruit   color
apple   cyan      7.8
        red       8.5
banana  cyan      6.4
        yellow    6.8
orange  yellow    5.6
Name: price, dtype: float64
```

可以看到现在的行索引类型为`MultiIndex`，也就是此时的索引有多列。关于多列索引的用法，读者可以参考：<https://pandas.pydata.org/pandas-docs/stable/user_guide/advanced.html>

### 3. 聚合函数：

除了`mean`以外，还可以使用以下聚合函数：

| 函数名 | 描述 |
| --- | --- |
| count() | 非空值的个数 |
| sum() | 求总和 |
| mean() | 求平均值 |
| median() | 求中位数 |
| min() | 求最小值 |
| max() | 求最大值 |
| mode() | 求众数 |
| std() | 求标准差 |
| var() | 求方差 |
