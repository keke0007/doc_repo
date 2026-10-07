# 第6节　pandas缺失值处理

<!-- 来源：http://www.zlkt.net/book/detail/11/377 -->

> 出自《Python数据分析到项目实战》第3章 Pandas库

---

## pandas缺失值处理

为了讲解缺失值处理的知识点，我们先来构造一个有缺失值的`DataFrame`。代码如下：

```python
df = pd.DataFrame(
    np.random.randn(5, 3),
    index=["a", "c", "e", "f", "h"],
    columns=["one", "two", "three"],
)

df['four'] = 'bar'
df['five'] = df['one'] > 0

df = df.reindex(["a", "b", "c", "d", "e", "f", "g", "h"])
print(df)
```

输出结果为：

```
        one	        two	        three	        four	five
a	-0.168907	-1.242016	1.288470	bar	False
b	NaN	NaN	NaN	NaN	NaN
c	-1.146328	0.761175	-0.318050	bar	False
d	NaN	NaN	NaN	NaN	NaN
e	2.495203	-1.069216	-0.384381	bar	True
f	0.314126	-2.085204	-0.579591	bar	True
g	NaN	NaN	NaN	NaN	NaN
h	-0.060276	0.691115	-0.692604	bar	False
```

### 一、判断是否为NAN值：

通过`isna`和`notna`可以判断`DataFrame`或者是`Series`中的数据是否为`NAN`或者不是`NAN`。比如查看`df["one"]`这一列的值是否为`NAN`：

```python
pd.isna(df["one"])
# 或者是
df["one"].isna()
```

输出结果为：

```
a    False
b     True
c    False
d     True
e    False
f    False
g     True
h    False
Name: one, dtype: bool
```

再通过`notna`来查看`df['four']`中有哪些是否不为空：

```python
df['four'].notna()
```

### 二、填充NAN数据：

#### 1. 填充常量值：

如果想要把`DataFrame`或者是`Series`中`NAN`的数据用其他值替换，那么可以使用`fillna`。示例代码如下：

```python
# 把所有`NA`的值填充为0
df.fillna(0)

# 把某一列的值进行填充
df['one'].fillna("missing")
```

#### 2. 使用计算值填充：

我们可以使用某一列的最大值、最小值、平均值等来填充`NAN`值，示例代码如下。

```python
# 对所有列都用平均值填充
df.fillna(df.mean())

# 对one到three列用平均值填充
df.fillna(df.mean()["one": "three"])
```

#### 2. 删除NAN值：

删除`NAN`值可以使用`dropna`来实现。删除`NAN`的操作，会把含有`NAN`值的整行或者整列都删掉，因此要慎用。示例代码如下。

```python
# 按照行删除
df.dropna()
# 或者是
df.dropna(axis=0)

# 按照列删除
df.dropna(axis=1)
```

#### 3. 替换：

针对一些满足条件的字符串，可以使用`replace`方法来替换，这个方法还可以使用正则表达式。示例代码如下。

```python
d = {"a": list(range(4)), "b": list("ab.."), "c": ["a", "b", np.nan, "d"]}
df = pd.DataFrame(d)
df.replace(".", np.nan)
```

### 三、更多：

更多关于Pandas缺失值的处理方法，请参考官方文档：<https://pandas.pydata.org/docs/user_guide/missing_data.html>
