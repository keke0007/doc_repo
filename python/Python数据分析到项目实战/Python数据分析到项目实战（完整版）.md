# Python数据分析到项目实战

> 来源：http://www.zlkt.net/book/detail/11/307（知了传课）

> 本文档为离线存档，仅用于个人学习。

---


## 第1章　数据分析前奏


### 第1节　什么是数据分析

<!-- 来源：http://www.zlkt.net/book/detail/11/307 -->

#### 数据分析介绍

##### 什么是数据分析：

数据分析是指用适当的统计分析方法对收集来的大量数据进行分析，提取有用信息和形成结论而对数据加以详细研究和概括总结的过程。数据分析的目的有多种，概括起来有三种：现状分析、原因分析、预测分析。现状分析简单来说就是告诉你过去发生了什么。原因分析简单来说就是告诉你某一现状为什么发生。预测分析简单来说就是预测未来会发生什么。

##### 数据分析步骤：

数据分析主要有六个过程：

1. 需求明确：明确做数据分析的目标。为后面的分析过程做好铺垫。
2. 数据收集：通过爬虫、商务合作的方式，获取想要的数据。
3. 数据处理：对获取来的数据进行处理和清洗，把不需要的剔除掉，把需要的加工成我们想要的。方便后面的分析。
4. 数据分析：根据自己的目的，以及现有的数据确定好分析的方法。
5. 数据展现：将数据按照确定好的分析方法进行展示出来。
6. 撰写报告：将分析的结果通过图表和文字的方式形成报告文档。

##### 数据分析的误区：

1. 分析目的不明确，为分析而分析：一定要找准自己分析数据的目标而去分析，比如是要了解现状，还是找出原因，还是预测未来发展等，千万不要为了分析而分析，这样就偏离主题了。
2. 缺乏业务知识，分析结果偏离实际：分析数据的时候，一定要和公司的业务结合起来。如果脱离业务，即使数据分析方法再牛逼，图标再优美，也无济于事。
3. 追求高级分析方法：一些人喜欢用一些高级的分析方法，认为只有这样才能体现专业性。其实高级的数据分析方法不一定是最好的，能够简单有效的解决问题的方法才是最好的。

##### 数据分析的方法和工具：

数据分析可以通过工具，也可以通过代码来实现。以下分别列出这些常用的：

1. 工具：Excel、Tableau、SPSS、百度图说等。
2. 编程：Python语言、R语言、数据库的SQL语言、Excel的VBA语言等。

##### 工具和代码该怎么选：

两者没有好坏之分，只有合适之分。数据分析总体来讲有两个模块，一个是数据处理，一个是可视化。如果数据已经经过处理了，并且手头上的软件可以直接非常方便的做可视化处理，那么我们用软件实现就可以。如果数据没有经过处理，那么最好通过Python或者R对数据进行有一些处理，然后再通过软件可视化。或者软件的可视化无法满足我们的要求，那么可以通过代码来实现。总而言之，工具功能无法100%的满足你的要求，但是效率高。代码做数据处理比较好，最数据可视化比较繁琐，但是DIY属性强！



### 第2节　环境搭建

<!-- 来源：http://www.zlkt.net/book/detail/11/308 -->

#### 环境搭建

##### 一、Python基础：

本课程用到的Python版本都是3.x。要有一定的Python基础，知道列表、字符串、函数等的用法。

---

##### 二、Anaconda：

`Anaconda（水蟒）`是一个捆绑了`Python`、`conda`、其他相关依赖包的一个软件。包含了180多个可学计算包及其依赖。`Anaconda3`是集成了`Python3`的环境，`Anaconda2`是集成了`Python2`的环境。`Anaconda`默认集成的包，是属于内置的`Python`的包。并且支持绝大部分操作系统（比如：Windows、Mac、Linux等）。下载地址如下：`https://www.anaconda.com/distribution/`（如果官网下载太慢，可以在清华大学开源软件站中下载：`https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/`）。根据自己的操作系统，下载相应的版本，因为`Anaconda`内置了许多的包，所以安装过程需要耗费相当长的时间，大家在安装的时候需要耐心等待。在安装完成后，会有以下几个模块：`Anaconda prompt`、`Anaconda Navigator`、`Spyder`、`jupyter notebook`，以下分别做一些介绍。

###### 2.1. Anaconda prompt：

`Anaconda prompt`是专门用来操作`anaconda`的终端。如果你安装完`Anaconda`后没有在环境变量的`PATH`中添加相关的环境变量，那么以后你想在终端使用`anaconda`相关的命令，则必须要在`Anaconda prompt`中完成。
![QQ截图20190215154246.png](images/QQ截图20190215154246.png)

###### 2.2. Anaconda Navigator：

这个相当于是一个导航面板，上面组织了`Anaconda`相关的软件。
![QQ截图20190215155346.png](images/QQ截图20190215155346.png)

###### 2.3. Spyder：

一个专门开发`Python`的软件，熟悉`MATLAB`的同学会比较有亲切感，但在后期的学习过程中，我们将不会使用这个工具写代码，因为还有更好的可替代的工具。

###### 2.4. jupyter notebook：

一个Python编辑环境，可以实时的查看代码的运行效果。
![QQ截图20190215155935.png](images/QQ截图20190215155935.png)

##### 三、使用jupyter notebook的姿势：

1. 先打开`Anaconda Prompt`，然后进入到项目所在的目录。
2. 输入命令`jupyter notebook`打开`jupyter notebook`浏览器。

##### 四、conda基本使用：

`conda`伴随着`Anaconda`安装而自动安装的。`conda`可以跟`virtualenv`一样管理不同的环境，也可以跟`pip`一样管理某个环境下的包。以下来看看两个功能的用法。

###### 4.1. 环境管理：

`conda`能跟`virtualenv`一样管理不同的`Python`环境，不同的环境之间是互相隔离，互不影响的。为什么需要创建不同的环境呢？原因是有时候项目比较多，但是项目依赖的包不一样，比如`A`项目用的是`Python2`开发的，而`B`项目用的是`Python3`开发的，那么我们在同一台电脑上就需要两套不同的环境来支撑他们运行了。创建环境的基本命令如下：

```shell
# conda create --name [环境名称] 比如以下：
conda create --name da-env
```

这样将创建一个叫做`da-env`的环境，这个环境的`python`解释器根据`anaconda`来，如果`anaconda`为`3.7`，那么将默认使用`3.7`的环境，如果`anaconda`内置的是`2.7`，那么将默认使用`2.7`的环境。然后你就可以使用`conda install numpy`的方式来安装包了，并且这样安装进来的包，只会安装在当前环境中。有的同学可能有想问，如果想要装一个`Python2.7`的环境，`anaconda`中没有内置`Python2.7`，那么该怎么实现呢？。实际上，我们只需要在安装的时候指定`python`的版本，如果这个版本现在不存在，那么`anaconda`会自动的给我们下载。所以安装`Python2.7`的环境，使用以下代码即可实现：

```
conda create --name xxx python=2.7
```

以下再列出`conda`管理环境的其他命令：

1. 创建的时候指定需要安装的包：

   ```
    conda create --name xxx numpy pandas
   ```
2. 创建的时候既需要指定包，也需要指定python环境：

   ```
    conda create --name xxx python=3.7 numpy pandas
   ```
3. 进入到某个环境

   ```
    windows: activate xxx
    mac/linux: source activate xxx
   ```
4. 退出环境：

   ```
    deactivate
   ```
5. 列出当前所有的环境：

   ```
    conda env list
   ```
6. 移除某个环境：

   ```
    conda remove --name xxx --all
   ```
7. 环境下的包导出和导入：

   - 导出：`conda env export > environment.yml`。
   - 导入：`conda env create --name xxx -f environment.yml`。

###### 4.2. 包管理：

`conda`也可以用来管理包。比如我们创建完一个新的环境后，想要在这个环境中安装包（比如numpy），那么可以通过以下代码来实现：

```
activate xxx
conda install numpy
```

以下再介绍一些包管理常用的命令：

1. 在不进入某个环境下直接给这个环境安装包：

   ```
   conda install [包名] -n [环境名]
   ```
2. 列出该环境下所有的包：

   ```
    conda list
   ```
3. 卸载某个包：

   ```
    conda remove [包名]
   ```
4. 设置安装包的源：

   ```
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
    conda config --set show_channel_urls yes
   ```



### 第3节　JupyterNotebook使用

<!-- 来源：http://www.zlkt.net/book/detail/11/309 -->

#### Jupyter notebook使用

##### 一、常用快捷键：

###### 1.1. 命令模式（按Esc键）：

1. Enter：转入编辑模式
2. Shift-Enter：运行本单元，选中下个单元
3. Ctrl-Enter：运行本单元
4. Alt-Enter：运行本单元，在其下插入新单元
5. Y：单元转入代码状态
6. M：单元转入markdown状态
7. R：单元转入raw状态
8. 1：设定 1 级标题
9. 2：设定 2 级标题
10. 3：设定 3 级标题
11. 4：设定 4 级标题
12. 5：设定 5 级标题
13. 6：设定 6 级标题
14. Up：选中上方单元
15. K：选中上方单元
16. Down：选中下方单元
17. J：选中下方单元
18. Shift-K：扩大选中上方单元
19. Shift-J：扩大选中下方单元
20. A：在上方插入新单元
21. B：在下方插入新单元
22. X：剪切选中的单元
23. C：复制选中的单元
24. Shift-V：粘贴到上方单元
25. V：粘贴到下方单元
26. Z：恢复删除的最后一个单元
27. D,D：删除选中的单元
28. Shift-M：合并选中的单元
29. Ctrl-S：文件存盘
30. S：文件存盘
31. L：转换行号
32. O：转换输出
33. Shift-O：转换输出滚动
34. Esc：关闭页面
35. Q：关闭页面
36. H：显示快捷键帮助
37. I,I：中断Notebook内核
38. 0,0：重启Notebook内核
39. Shift：忽略
40. Shift-Space：向上滚动
41. Space：向下滚动

###### 1.2. 编辑模式：

1. Tab : 代码补全或缩进
2. Shift-Tab : 提示
3. Ctrl-] : 缩进
4. Ctrl-[ : 解除缩进
5. Ctrl-A : 全选
6. Ctrl-Z : 复原
7. Ctrl-Shift-Z : 再做
8. Ctrl-Y : 再做
9. Ctrl-Home : 跳到单元开头
10. Ctrl-Up : 跳到单元开头
11. Ctrl-End : 跳到单元末尾
12. Ctrl-Down : 跳到单元末尾
13. Ctrl-Left : 跳到左边一个字首
14. Ctrl-Right : 跳到右边一个字首
15. Ctrl-Backspace : 删除前面一个字
16. Ctrl-Delete : 删除后面一个字
17. Esc : 进入命令模式
18. Ctrl-M : 进入命令模式
19. Shift-Enter : 运行本单元，选中下一单元
20. Ctrl-Enter : 运行本单元
21. Alt-Enter : 运行本单元，在下面插入一单元
22. Ctrl-Shift-- : 分割单元
23. Ctrl-Shift-Subtract : 分割单元
24. Ctrl-S : 文件存盘
25. Shift : 忽略
26. Up : 光标上移或转入上一单元
27. Down :光标下移或转入下一单元

##### 二、注意事项：

`jupyter notebook`每一个`cell`运行完后都会把这个`cell`中的变量保存到内存中，如果在一个`cell`中修改了之前的变量，再此运行这个`cell`的时候可能会导致一些问题产生。比如以下代码：

```python
# 第一个cell中的代码
a = 10
b = 20

# 第二个cell中的代码
c = a/b
b = 0
```

因为第二个`cell`修改了`b`变量，此时在整个环境中`b`都是等于0的，所以以后再运行这个`cell`的时候，`a/b`这个就会出问题了。这时候可以使用`Kernel->Rstart&Run All`来重新运行整个项目。



### 第4节　作业

<!-- 来源：http://www.zlkt.net/book/detail/11/310 -->

#### 作业

1. 数据分析分成哪六大步骤？
2. 把`Anaconda`开发环境安装好。（这点特别重要！！！）
3. 如果重复运行一个cell中的代码出现问题了，该怎么解决？



## 第2章　Numpy库


### 第1节　Numpy库介绍

<!-- 来源：http://www.zlkt.net/book/detail/11/311 -->

#### Numpy库介绍

`NumPy`是一个功能强大的`Python`库，主要用于对多维数组执行计算。`NumPy`这个词来源于两个单词-- `Numerical`和`Python`。`NumPy`提供了大量的库函数和操作，可以帮助程序员轻松地进行数值计算。在数据分析和机器学习领域被广泛使用。他有以下几个特点：

1. numpy内置了并行运算功能，当系统有多个核心时，做某种计算时，numpy会自动做并行计算。
2. Numpy底层使用C语言编写，内部解除了GIL（全局解释器锁），其对数组的操作速度不受Python解释器的限制，效率远高于纯Python代码。
3. 有一个强大的N维数组对象Array（一种类似于列表的东西）。
4. 实用的线性代数、傅里叶变换和随机数生成函数。

总而言之，他是一个非常高效的用于处理数值型运算的包。

##### 一、安装：

通过`pip install numpy`即可安装。

##### 二、Numpy数组和Python列表性能对比：

比如我们想要对一个Numpy数组和Python列表中的每个素进行求平方。那么代码如下：

```python
# Python列表的方式
t1 = time.time()
a = []
for x in range(100000):
    a.append(x**2)
t2 = time.time()
t = t2 - t1
print(t)
```

花费的时间大约是`0.07180`左右。而如果使用`numpy`的数组来做，那速度就要快很多了：

```python
t3 = time.time()
b = np.arange(100000)**2
t4 = time.time()
print(t4-t3)
```



### 第2节　数组基本

<!-- 来源：http://www.zlkt.net/book/detail/11/312 -->

#### NumPy数组基本用法

1. `Numpy`是`Python`科学计算库，用于快速处理任意维度的数组。
2. `NumPy`提供一个**N维数组类型ndarray**，它描述了**相同类型**的“items”的集合。
3. `numpy.ndarray`支持向量化运算。
4. `NumPy`使用c语言写的，底部解除了`GIL`，其对数组的操作速度不在受`python`解释器限制。

##### 一、numpy中的数组：

`Numpy`中的数组的使用跟`Python`中的列表非常类似。他们之间的区别如下：

1. 一个列表中可以存储多种数据类型。比如`a = [1,'a']`是允许的，而数组只能存储同种数据类型。
2. 数组可以是多维的，当多维数组中所有的数据都是数值类型的时候，相当于线性代数中的矩阵，是可以进行相互间的运算的。

##### 二、创建数组（np.ndarray对象）：

`Numpy`经常和数组打交道，因此首先第一步是要学会创建数组。在`Numpy`中的数组的数据类型叫做`ndarray`。以下是两种创建的方式：

1. 根据`Python`中的列表生成：

   ```python
   import numpy as np
   a1 = np.array([1,2,3,4])
   print(a1)
   print(type(a1))
   ```
2. 使用`np.arange`生成，`np.arange`的用法类似于`Python`中的`range`：

   ```python
   import numpy as np
   a2 = np.arange(2,21,2)
   print(a2)
   ```
3. 使用`np.random`生成随机数的数组：

   ```python
   a1 = np.random.random((2,2)) # 生成2行2列的随机数的数组
   a2 = np.random.randint(0,10,size=(3,3)) # 元素是从0-10之间随机的3行3列的数组
   ```
4. 使用函数生成特殊的数组：

   ```python
   import numpy as np
   a1 = np.zeros((2,2)) #生成一个所有元素都是0的2行2列的数组
   a2 = np.ones((3,2)) #生成一个所有元素都是1的3行2列的数组
   a3 = np.full((2,2),8) #生成一个所有元素都是8的2行2列的数组
   a4 = np.eye(3) #生成一个在斜方形上元素为1，其他元素都为0的3x3的矩阵
   ```

##### 三、ndarray常用属性：

###### 3.1. `ndarray.dtype`：

因为数组中只能存储同一种数据类型，因此可以通过`dtype`获取数组中的元素的数据类型。以下是`ndarray.dtype`的常用的数据类型：

| 数据类型 | 描述 | 唯一标识符 |
| --- | --- | --- |
| bool | 用一个字节存储的布尔类型（True或False） | ‘b’ |
| int8 | 一个字节大小，-128 至 127 | ‘i1’ |
| int16 | 整数，16 位整数(-32768 ~ 32767) | ‘i2’ |
| int32 | 整数，32 位整数(-2147483648 ~ 2147483647) | ‘i4’ |
| int64 | 整数，64 位整数(-9223372036854775808 ~ 9223372036854775807) | ‘i8’ |
| uint8 | 无符号整数，0 至 255 | ‘u1’ |
| uint16 | 无符号整数，0 至 65535 | ‘u2’ |
| uint32 | 无符号整数，0 至 2 ** 32 - 1 | ‘u4’ |
| uint64 | 无符号整数，0 至 2 ** 64 - 1 | ‘u8’ |
| float16 | 半精度浮点数：16位，正负号1位，指数5位，精度10位 | ‘f2’ |
| float32 | 单精度浮点数：32位，正负号1位，指数8位，精度23位 | ‘f4’ |
| float64 | 双精度浮点数：64位，正负号1位，指数11位，精度52位 | ‘f8’ |
| complex64 | 复数，分别用两个32位浮点数表示实部和虚部 | ‘c8’ |
| complex128 | 复数，分别用两个64位浮点数表示实部和虚部 | ‘c16’ |
| object_ | python对象 | ‘O’ |
| string_ | 字符串 | ‘S’ |
| unicode_ | unicode类型 | ‘U’ |

我们可以看到，`Numpy`中关于数值的类型比`Python`内置的多得多，这是因为`Numpy`为了能高效处理处理海量数据而设计的。举个例子，比如现在想要存储上百亿的数字，并且这些数字都不超过254（一个字节内），我们就可以将`dtype`设置为`int8`，这样就比默认使用`int64`更能节省内存空间了。类型相关的操作如下：

1. 默认的数据类型：

   ```python
   import numpy as np
   a1 = np.array([1,2,3])
   print(a1.dtype)
   # 如果是windows系统，默认是int32
   # 如果是mac或者linux系统，则根据系统来
   ```
2. 指定`dtype`：

   ```python
   import numpy as np
   a1 = np.array([1,2,3],dtype=np.int64)
   # 或者 a1 = np.array([1,2,3],dtype="i8")
   print(a1.dtype)
   ```
3. 修改`dtype`：

   ```python
   import numpy as np
   a1 = np.array([1,2,3])
   print(a1.dtype) # window系统下默认是int32
   # 以下修改dtype
   a2 = a1.astype(np.int64) # astype不会修改数组本身，而是会将修改后的结果返回
   print(a2.dtype)
   ```

###### 3.2. `ndarray.size`：

获取数组中总的元素的个数。比如有个二维数组：

```python
import numpy as np
   a1 = np.array([[1,2,3],[4,5,6]])
   print(a1.size) #打印的是6，因为总共有6个元素
```

###### 3.3. `ndarray.ndim`：

数组的维数。比如：

```python
a1 = np.array([1,2,3])
   print(a1.ndim) # 维度为1
   a2 = np.array([[1,2,3],[4,5,6]])
   print(a2.ndim) # 维度为2
   a3 = np.array([[[1,2,3],[4,5,6]],[[7,8,9],[10,11,12]]])
   print(a3.ndim) # 维度为3
```

###### 3.4. `ndarray.shape`：

数组的维度的元组。比如以下代码：

```python
a1 = np.array([1,2,3])
   print(a1.shape) # 输出(3,)，意思是一维数组，有3个数据

   a2 = np.array([[1,2,3],[4,5,6]])
   print(a2.shape) # 输出(2,3)，意思是二位数组，2行3列

   a3 = np.array([
       [
           [1,2,3],
           [4,5,6]
       ],
       [
           [7,8,9],
           [10,11,12]
       ]
   ])
   print(a3.shape) # 输出(2,2,3)，意思是三维数组，总共有2个元素，每个元素是2行3列的

   a44 = np.array([1,2,3],[4,5])
   print(a4.shape) # 输出(2,)，意思是a4是一个一维数组，总共有2列
   print(a4) # 输出[list([1, 2, 3]) list([4, 5])]，其中最外面层是数组，里面是Python列表
```

另外，我们还可以通过`ndarray.reshape`来重新修改数组的维数。示例代码如下：

```python
a1 = np.arange(12) #生成一个有12个数据的一维数组
   print(a1)

   a2 = a1.reshape((3,4)) #变成一个2维数组，是3行4列的
   print(a2)

   a3 = a1.reshape((2,3,2)) #变成一个3维数组，总共有2块，每一块是2行2列的
   print(a3)

   a4 = a2.reshape((12,)) # 将a2的二维数组重新变成一个12列的1维数组
   print(a4)

   a5 = a2.flatten() # 不管a2是几维数组，都将他变成一个一维数组
   print(a5)
```

注意，`reshape`并不会修改原来数组本身，而是会将修改后的结果返回。如果想要直接修改数组本身，那么可以使用`resize`来替代`reshape`。

###### 3.5. `ndarray.itemsize`：

数组中每个元素占的大小，单位是字节。比如以下代码：

```python
a1 = np.array([1,2,3],dtype=np.int32)
   print(a1.itemsize) # 打印4，因为每个字节是8位，32位/8=4个字节
```



### 第3节　数组操作

<!-- 来源：http://www.zlkt.net/book/detail/11/313 -->

#### Numpy数组操作

##### 一、数组广播机制：

###### 1.1. 数组与数的计算：

在`Python`列表中，想要对列表中所有的元素都加一个数，要么采用`map`函数，要么循环整个列表进行操作。但是`NumPy`中的数组可以直接在数组上进行操作。示例代码如下：

```python
import numpy as np
a1 = np.random.random((3,4))
print(a1)
# 如果想要在a1数组上所有元素都乘以10，那么可以通过以下来实现
a2 = a1*10
print(a2)
# 也可以使用round让所有的元素只保留2位小数
a3 = a2.round(2)
```

以上例子是相乘，其实相加、相减、相除也都是类似的。

###### 1.2. 数组与数组的计算：

1. 结构相同的数组之间的运算：

   ```python
   a1 = np.arange(0,24).reshape((3,8))
   a2 = np.random.randint(1,10,size=(3,8))
   a3 = a1 + a2 #相减/相除/相乘都是可以的
   print(a1)
   print(a2)
   print(a3)
   ```
2. 与行数相同并且只有1列的数组之间的运算：

   ```python
   a1 = np.random.randint(10,20,size=(3,8)) #3行8列
   a2 = np.random.randint(1,10,size=(3,1)) #3行1列
   a3 = a1 - a2 #行数相同，且a2只有1列，能互相运算
   print(a3)
   ```
3. 与列数相同并且只有1行的数组之间的运算：

   ```python
   a1 = np.random.randint(10,20,size=(3,8)) #3行8列
   a2 = np.random.randint(1,10,size=(1,8))
   a3 = a1 - a2
   print(a3)
   ```

###### 二、广播原则：

**如果两个数组的后缘维度（trailing dimension，即从末尾开始算起的维度）的轴长度相符或其中一方的长度为1，则认为他们是广播兼容的。广播会在缺失和（或）长度为1的维度上进行。**。看以下案例分析：

1. `shape`为`(3,8,2)`的数组能和`(8,3)`的数组进行运算吗？
   分析：不能，因为按照广播原则，从后面往前面数，`(3,8,2)`和`(8,3)`中的`2`和`3`不相等，所以不能进行运算。
2. `shape`为`(3,8,2)`的数组能和`(8,1)`的数组进行运算吗？
   分析：能，因为按照广播原则，从后面往前面数，`(3,8,2)`和`(8,1)`中的`2`和`1`虽然不相等，但是因为有一方的长度为`1`，所以能参与运算。
3. `shape`为`(3,1,8)`的数组能和`(8,1)`的数组进行运算吗？
   分析：能，因为按照广播原则，从后面往前面数，`(3,1,4)`和`(8,1)`中的`4`和`1`虽然不相等且`1`和`8`不相等，但是因为这两项中有一方的长度为`1`，所以能参与运算。

---

##### 三、数组形状的操作：

可以通过一些函数，非常方便的操作数组的形状。

###### 3.1. reshape和resize方法：

两个方法都是用来修改数组形状的，但是有一些不同。

1. `reshape`是将数组转换成指定的形状，然后返回转换后的结果，对于原数组的形状是不会发生改变的。调用方式：

   ```python
   a1 = np.random.randint(0,10,size=(3,4))
   a2 = a1.reshape((2,6)) #将修改后的结果返回，不会影响原数组本身
   ```
2. `resize`是将数组转换成指定的形状，会直接修改数组本身。并不会返回任何值。调用方式：

   ```python
   a1 = np.random.randint(0,10,size=(3,4))
   a1.resize((2,6)) #a1本身发生了改变
   ```

###### 3.2. flatten和ravel方法：

两个方法都是将多维数组转换为一维数组，但是有以下不同：

1. `flatten`是将数组转换为一维数组后，然后将这个拷贝返回回去，所以后续对这个返回值进行修改不会影响之前的数组。
2. `ravel`是将数组转换为一维数组后，将这个视图（可以理解为引用）返回回去，所以后续对这个返回值进行修改会影响之前的数组。
   比如以下代码：

```python
x = np.array([[1, 2], [3, 4]])
x.flatten()[1] = 100 #此时的x[0]的位置元素还是1
x.ravel()[1] = 100 #此时x[0]的位置元素是100
```

###### 3.3. 不同数组的组合：

如果有多个数组想要组合在一起，也可以通过其中的一些函数来实现。

1. `vstack`：将数组按垂直方向进行叠加。数组的列数必须相同才能叠加。示例代码如下：

   ```python
   a1 = np.random.randint(0,10,size=(3,5))
   a2 = np.random.randint(0,10,size=(1,5))
   a3 = np.vstack([a1,a2])
   ```
2. `hstack`：将数组按水平方向进行叠加。数组的行必须相同才能叠加。示例代码如下：

   ```python
   a1 = np.random.randint(0,10,size=(3,2))
   a2 = np.random.randint(0,10,size=(3,1))
   a3 = np.hstack([a1,a2])
   ```
3. `concatenate([],axis)`：将两个数组进行叠加，但是具体是按水平方向还是按垂直方向。则要看`axis`的参数，如果`axis=0`，那么代表的是往垂直方向（行）叠加，如果`axis=1`，那么代表的是往水平方向（列）上叠加，如果`axis=None`，那么会将两个数组组合成一个一维数组。需要注意的是，如果往水平方向上叠加，那么行必须相同，如果是往垂直方向叠加，那么列必须相同。示例代码如下：

   ```python
   a = np.array([[1, 2], [3, 4]])
   b = np.array([[5, 6]])
   np.concatenate((a, b), axis=0)
   # 结果：
   array([[1, 2],
       [3, 4],
       [5, 6]])

   np.concatenate((a, b.T), axis=1)
   # 结果：
   array([[1, 2, 5],
       [3, 4, 6]])

   np.concatenate((a, b), axis=None)
   # 结果：
   array([1, 2, 3, 4, 5, 6])
   ```

###### 3.4. 数组的切割：

通过`hsplit`和`vsplit`以及`array_split`可以将一个数组进行切割。

1. `hsplit`：按照水平方向进行切割。用于指定分割成几列，可以使用数字来代表分成几部分，也可以使用数组来代表分割的地方。示例代码如下：

   ```python
   a1 = np.arange(16.0).reshape(4, 4)
   np.hsplit(a1,2) #分割成两部分
   >>> array([[ 0.,  1.],
        [ 4.,  5.],
        [ 8.,  9.],
        [12., 13.]]), array([[ 2.,  3.],
        [ 6.,  7.],
        [10., 11.],
        [14., 15.]])]

   np.hsplit(a1,[1,2]) #代表在下标为1的地方切一刀，下标为2的地方切一刀，分成三部分
   >>> [array([[ 0.],
        [ 4.],
        [ 8.],
        [12.]]), array([[ 1.],
        [ 5.],
        [ 9.],
        [13.]]), array([[ 2.,  3.],
        [ 6.,  7.],
        [10., 11.],
        [14., 15.]])]
   ```
2. `vsplit`：按照垂直方向进行切割。用于指定分割成几行，可以使用数字来代表分成几部分，也可以使用数组来代表分割的地方。示例代码如下：

   ```python
   np.vsplit(x,2) #代表按照行总共分成2个数组
   >>> [array([[0., 1., 2., 3.],
        [4., 5., 6., 7.]]), array([[ 8.,  9., 10., 11.],
        [12., 13., 14., 15.]])]

   np.vsplit(x,(1,2)) #代表按照行进行划分，在下标为1的地方和下标为2的地方分割
   >>> [array([[0., 1., 2., 3.]]),
       array([[4., 5., 6., 7.]]),
       array([[ 8.,  9., 10., 11.],
              [12., 13., 14., 15.]])]
   ```
3. `split/array_split(array,indicate_or_seciont,axis)`：用于指定切割方式，在切割的时候需要指定是按照行还是按照列，`axis=1`代表按照列，`axis=0`代表按照行。示例代码如下：

   ```python
   np.array_split(x,2,axis=0) #按照垂直方向切割成2部分
   >>> [array([[0., 1., 2., 3.],
        [4., 5., 6., 7.]]), array([[ 8.,  9., 10., 11.],
        [12., 13., 14., 15.]])]
   ```

---

##### 四、数组（矩阵）转置和轴对换：

`numpy`中的数组其实就是线性代数中的矩阵。矩阵是可以进行转置的。`ndarray`有一个`T`属性，可以返回这个数组的转置的结果。示例代码如下：

```python
a1 = np.arange(0,24).reshape((4,6))
a2 = a1.T
print(a2)
```

另外还有一个方法叫做`transpose`，这个方法返回的是一个View，也即修改返回值，会影响到原来数组。示例代码如下：

```python
a1 = np.arange(0,24).reshape((4,6))
a2 = a1.transpose()
```

为什么要进行矩阵转置呢，有时候在做一些计算的时候需要用到。比如做矩阵的内积的时候。就必须将矩阵进行转置后再乘以之前的矩阵：

```python
a1 = np.arange(0,24).reshape((4,6))
a2 = a1.T
print(a1.dot(a2))
```



### 第4节　索引和切片

<!-- 来源：http://www.zlkt.net/book/detail/11/314 -->

#### Numpy数组操作（索引和切片）

##### 一、索引和切片：

1. 获取某行的数据：

   ```python
   # 1. 如果是一维数组
    a1 = np.arange(0,29)
    print(a1[1]) #获取下标为1的元素

    a1 = np.arange(0,24).reshape((4,6))
    print(a1[1]) #获取下标为1的行的数据
   ```
2. 连续获取某几行的数据：

   ```python
   # 1. 获取连续的几行的数据
    a1 = np.arange(0,24).reshape((4,6))
    print(a1[0:2]) #获取0行到1行的数据

    # 2. 获取不连续的几行的数据
    print(a1[[0,2,3]])

    # 3. 也可以使用负数进行索引
    print(a1[[-1,-2]])
   ```
3. 获取某行某列的数据：

   ```python
   a1 = np.arange(0,30).reshape((5,6))
    print(a1[1,1]) #获取1行1列的数据

    print(a1[0:2,0:2]) #获取0-1行的0-1列的数据
    print(a1[[1,4],[2,3]]) #获取(1,2)和(4,3)的两个数据，这也叫花式索引
   ```
4. 获取某列的数据：

   ```python
   a1 = np.arange(0,24).reshape((4,6))
    print(a1[:,1]) #获取第1列的数据
   ```

##### 二、布尔索引：

布尔运算也是矢量的，比如以下代码：

```python
a1 = np.arange(0,24).reshape((4,6))
print(a1<10) #会返回一个新的数组，这个数组中的值全部都是bool类型
> [[ True  True  True  True  True  True]
 [ True  True  True  True False False]
 [False False False False False False]
 [False False False False False False]]
```

这样看上去没有什么用，假如我现在要实现一个需求，要将`a1`数组中所有小于10的数据全部都提取出来。那么可以使用以下方式实现：

```python
a1 = np.arange(0,24).reshape((4,6))
a2 = a1 < 10
print(a1[a2]) #这样就会在a1中把a2中为True的元素对应的位置的值提取出来
```

其中布尔运算可以有`!=`、`==`、`>`、`<`、`>=`、`<=`以及`&(与)`和`|(或)`。示例代码如下：

```python
a1 = np.arange(0,24).reshape((4,6))
a2 = a1[(a1 < 5) | (a1 > 10)]
print(a2)
```

##### 三、值的替换：

利用索引，也可以做一些值的替换。把满足条件的位置的值替换成其他的值。比如以下代码：

```python
a1 = np.arange(0,24).reshape((4,6))
a1[3] = 0 #将第三行的所有值都替换成0
print(a1)
```

也可以使用条件索引来实现：

```python
a1 = np.arange(0,24).reshape((4,6))
a1[a1 < 5] = 0 #将小于5的所有值全部都替换成0
print(a1)
```

还可以使用函数来实现：

```python
# where函数：
a1 = np.arange(0,24).reshape((4,6))
a2 = np.where(a1 < 10,1,0) #把a1中所有小于10的数全部变成1，其余的变成0
print(a2)
```



### 第5节　索引和切片作业

<!-- 来源：http://www.zlkt.net/book/detail/11/315 -->

#### Numpy索引和切片作业

1. 将`np.arange(10)`数组中的奇数全部都替换成`-1`。
2. 有一个`4`行`4`列的数组（比如：`np.random.randint(0,10,size=(4,4))`），请将其中对角线的数取出来形成一个一维数组。提示（使用`np.eye`）。
3. 有一个`4`行`4`列的数组，请取出其中`(0,0),(1,2),(3,2)`的点。
4. 有一个`4`行`4`列的数组，请取出其中`2-3`行（包括第3行）的所有数据。
5. 有一个`8`行`9`列的数组，请将其中1-5行（包含第5行）的第8列大于3的数全部都取出来。



### 第6节　深拷贝和浅拷贝

<!-- 来源：http://www.zlkt.net/book/detail/11/317 -->

#### 深拷贝和浅拷贝

在操作数组的时候，它们的数据有时候拷贝进一个新的数组，有时候又不是。这经常是初学者感到困惑。下面有三种情况：

##### 一、不拷贝：

如果只是简单的赋值，那么不会进行拷贝。示例代码如下：

```python
a = np.arange(12)
b = a #这种情况不会进行拷贝
print(b is a) #返回True，说明b和a是相同的
```

##### 二、View或者浅拷贝：

有些情况，会进行变量的拷贝，但是他们所指向的内存空间都是一样的，那么这种情况叫做浅拷贝，或者叫做`View(视图)`。比如以下代码：

```python
a = np.arange(12)
c = a.view()
print(c is a) #返回False，说明c和a是两个不同的变量
c[0] = 100
print(a[0]) #打印100，说明对c上的改变，会影响a上面的值，说明他们指向的内存空间还是一样的，这种叫做浅拷贝，或者说是view
```

##### 三、深拷贝：

将之前数据完完整整的拷贝一份放到另外一块内存空间中，这样就是两个完全不同的值了。示例代码如下：

```python
a = np.arange(12)
d = a.copy()
print(d is a) #返回False，说明d和a是两个不同的变量
d[0] = 100
print(a[0]) #打印0，说明d和a指向的内存空间完全不同了。
```

##### 四、例子：

像之前讲到的`flatten`和`ravel`就是这种情况，`ravel`返回的就是View，而`flatten`返回的就是深拷贝。



### 第7节　文件操作

<!-- 来源：http://www.zlkt.net/book/detail/11/318 -->

#### 文件操作

##### 一、操作CSV文件：

###### 1.1. 文件保存：

有时候我们有了一个数组，需要保存到文件中，那么可以使用`np.savetxt`来实现。相关的函数描述如下：

```python
np.savetxt(frame, array, fmt='%.18e', delimiter=None)
* frame : 文件、字符串或产生器，可以是.gz或.bz2的压缩文件
* array : 存入文件的数组
* fmt : 写入文件的格式，例如：%d %.2f %.18e
* delimiter : 分割字符串，默认是任何空格
```

以下是使用的例子：

```python
a = np.arange(100).reshape(5,20)
np.savetxt("a.csv",a,fmt="%d",delimiter=",")
```

###### 1.2. 读取文件：

有时候我们的数据是需要从文件中读取出来的，那么可以使用`np.loadtxt`来实现。相关的函数描述如下：

```python
np.loadtxt(frame, dtype=np.float, delimiter=None, unpack=False)
* frame：文件、字符串或产生器，可以是.gz或.bz2的压缩文件。
* dtype：数据类型，可选。
* delimiter：分割字符串，默认是任何空格。
* skiprows：跳过前面x行。
* usecols：读取指定的列，用元组组合。
* unpack：如果True，读取出来的数组是转置后的。
```

##### 二、np独有的存储解决方案：

`numpy`中还有一种独有的存储解决方案。文件名是以`.npy`或者`npz`结尾的。以下是存储和加载的函数。

1. 存储：`np.save(fname,array)`或`np.savez(fname,array)`。其中，前者函数的扩展名是`.npy`，后者的扩展名是`.npz`，后者是经过压缩的。
2. 加载：`np.load(fname)`。

---

##### 三、CSV文件操作：

###### 3.1. 读取csv文件：

```python
import csv

with open('stock.csv','r') as fp:
    reader = csv.reader(fp)
    titles = next(reader)
    for x in reader:
        print(x)
```

这样操作，以后获取数据的时候，就要通过下表来获取数据。如果想要在获取数据的时候通过标题来获取。那么可以使用`DictReader`。示例代码如下：

```python
import csv

with open('stock.csv','r') as fp:
    reader = csv.DictReader(fp)
    for x in reader:
        print(x['turnoverVol'])
```

###### 3.2. 写入数据到csv文件：

写入数据到csv文件，需要创建一个`writer`对象，主要用到两个方法。一个是`writerow`，这个是写入一行。一个是`writerows`，这个是写入多行。示例代码如下：

```python
import csv

headers = ['name','age','classroom']
values = [
    ('zhiliao',18,'111'),
    ('wena',20,'222'),
    ('bbc',21,'111')
]
with open('test.csv','w',newline='') as fp:
    writer = csv.writer(fp)
    writer.writerow(headers)
    writer.writerows(values)
```

也可以使用字典的方式把数据写入进去。这时候就需要使用`DictWriter`了。示例代码如下：

```python
import csv

headers = ['name','age','classroom']
values = [
    {"name":'wenn',"age":20,"classroom":'222'},
    {"name":'abc',"age":30,"classroom":'333'}
]
with open('test.csv','w',newline='') as fp:
    writer = csv.DictWriter(fp,headers)
    writer.writerow({'name':'zhiliao',"age":18,"classroom":'111'})
    writer.writerows(values)
```



### 第8节　文件操作作业

<!-- 来源：http://www.zlkt.net/book/detail/11/319 -->

#### 数组操作和文件操作作业

1. 数组`a = np.random.rand(3,2,3)`能和`b = np.random.rand(3,2,2)`进行运算吗？能和`c = np.random.rand(3,1,1)`进行运算吗？请说明结果的原因。
2. 将数组`a = np.random.rand(3,5)`和`b = np.random.rand(6,4)`叠加在一起，其中`a`在`b`的上面，并且在`b`的第2列（下标从0开始）新增一列，用0来填充。
3. 将数组`a = np.random.rand(4,5)`扁平化成一维数组，可以使用`flatten`和`ravel`，对两者的返回值进行操作，哪个会影响到数组`a`？对会影响到`a`数组的那个函数，请说明详细的原因。
4. 使用`numpy`自带的`csv`方法读取出`stock.csv`文件中`preClosePrice`、`openPrice`、`highestPrice`、`lowestPrice`的数据（提示：使用skiprows和usecols参数）。



### 第9节　NAN和INF值处理

<!-- 来源：http://www.zlkt.net/book/detail/11/320 -->

#### NAN和INF值处理

首先我们要知道这两个英文单词代表的什么意思：

1. `NAN`：`Not A number`，不是一个数字的意思，但是他是属于浮点类型的，所以想要进行数据操作的时候需要注意他的类型。
2. `INF`：`Infinity`，代表的是无穷大的意思，也是属于浮点类型。`np.inf`表示正无穷大，`-np.inf`表示负无穷大，一般在出现除数为0的时候为无穷大。比如`2/0`。

##### 一、NAN一些特点：

1. NAN和NAN不相等。比如`np.NAN != np.NAN`这个条件是成立的。
2. NAN和任何值做运算，结果都是NAN。

有些时候，特别是从文件中读取数据的时候，经常会出现一些缺失值。缺失值的出现会影响数据的处理。因此我们在做数据分析之前，先要对缺失值进行一些处理。处理的方式有多种，需要根据实际情况来做。一般有两种处理方式：删除缺失值，用其他值进行填充。

##### 二、删除缺失值：

有时候，我们想要将数组中的`NAN`删掉，那么我们可以换一种思路，就是只提取不为`NAN`的值。示例代码如下：

```python
# 1. 删除所有NAN的值，因为删除了值后数组将不知道该怎么变化，所以会被变成一维数组
data = np.random.randint(0,10,size=(3,5)).astype(np.float)
data[0,1] = np.nan
data = data[~np.isnan(data)] # 此时的data会没有nan，并且变成一个1维数组

# 2. 删除NAN所在的行
data = np.random.randint(0,10,size=(3,5)).astype(np.float)
# 将第(0,1)和(1,2)两个值设置为NAN
data[[0,1],[1,2]] = np.NAN
# 获取哪些行有NAN
lines = np.where(np.isnan(data))[0]
# 使用delete方法删除指定的行,axis=0表示删除行，lines表示删除的行号
data1 = np.delete(data,lines,axis=0)
```

##### 三、用其他值进行替代：

有些时候我们不想直接删掉，比如有一个成绩表，分别是数学和英语，但是因为某个人在某个科目上没有成绩，那么此时就会出现NAN的情况，这时候就不能直接删掉了，就可以使用某些值进行替代。假如有以下表格：

| 数学 | 英语 |
| --- | --- |
| 59 | 89 |
| 90 | 32 |
| 78 | 45 |
| 34 | NAN |
| NAN | 56 |
| 23 | 56 |

如果想要求每门成绩的总分，以及每门成绩的平均分，那么就可以采用某些值替代。比如求总分，那么就可以把NAN替换成0，如果想要求平均分，那么就可以把NAN替换成其他值的平均值。示例代码如下：

```python
scores = np.loadtxt("nan_scores.csv",skiprows=1,delimiter=",",encoding="utf-8",dtype=np.str)
scores[scores == ""] = np.NAN
scores = scores.astype(np.float)
# 1. 求出学生成绩的总分
scores1 = scores.copy()
socres1.sum(axis=1)

# 2. 求出每门成绩的平均分
scores2 = scores.copy()
for x in range(scores2.shape[1]):
    score = scores2[:,x]
    non_nan_score = score[score == score]
    score[score != score] = non_nan_score.mean()
print(scores2.mean(axis=0))
```



### 第10节　random模块

<!-- 来源：http://www.zlkt.net/book/detail/11/321 -->

#### np.random模块

`np.random`为我们提供了许多获取随机数的函数。这里统一来学习一下。

##### 一、np.random.seed：

用于指定随机数生成时所用算法开始的整数值，如果使用相同的`seed()`值，则每次生成的随即数都相同，如果不设置这个值，则系统根据时间来自己选择这个值，此时每次生成的随机数因时间差异而不同。一般没有特殊要求不用设置。以下代码：

```python
np.random.seed(1)
print(np.random.rand()) # 打印0.417022004702574
print(np.random.rand()) # 打印其他的值，因为随机数种子只对下一次随机数的产生会有影响。
```

##### 二、np.random.rand：

生成一个值为`[0,1)`之间的数组，形状由参数指定，如果没有参数，那么将返回一个随机值。示例代码如下：

```python
data1 = np.random.rand(2,3,4) # 生成2块3行4列的数组，值从0-1之间
data2 = np.random.rand() #生成一个0-1之间的随机数
```

##### 三、np.random.randn：

生成均值(μ)为0，标准差（σ）为1的标准正态分布的值。示例代码如下：

```python
data = np.random.randn(2,3) #生成一个2行3列的数组，数组中的值都满足标准正太分布
```

##### 四、np.random.randint：

生成指定范围内的随机数，并且可以通过`size`参数指定维度。示例代码如下：

```python
data1 = np.random.randint(10,size=(3,5)) #生成值在0-10之间，3行5列的数组
data2 = np.random.randint(1,20,size=(3,6)) #生成值在1-20之间，3行6列的数组
```

##### 五、np.random.choice：

从一个列表或者数组中，随机进行采样。或者是从指定的区间中进行采样，采样个数可以通过参数指定：

```python
data = [4,65,6,3,5,73,23,5,6]
result1 = np.random.choice(data,size=(2,3)) #从data中随机采样，生成2行3列的数组
result2 = np.random.choice(data,3) #从data中随机采样3个数据形成一个一维数组
result3 = np.random.choice(10,3) #从0-10之间随机取3个值
```

##### 六、np.random.shuffle：

把原来数组的元素的位置打乱。示例代码如下：

```python
a = np.arange(10)
np.random.shuffle(a) #将a的元素的位置都会进行随机更换
```

##### 七、更多：

更多的random模块的文档，请参考`Numpy`的官方文档：<https://docs.scipy.org/doc/numpy/reference/routines.random.html>



### 第11节　Axis理解

<!-- 来源：http://www.zlkt.net/book/detail/11/322 -->

#### Axis理解

之前的课程中，为了方便大家理解，我们说`axis=0`代表的是行，`axis=1`代表的是列。但其实不是这么简单理解的。这里我们专门用一节来解释一下这个`axis`轴的概念。

简单来说， **最外面的括号代表着 axis=0，依次往里的括号对应的 axis 的计数就依次加 1**。什么意思呢？下面再来解释一下。
![axis1.png](images/axis1.png)

最外面的括号就是`axis=0`，里面两个子括号`axis=1`。
**操作方式：如果指定轴进行相关的操作，那么他会使用轴下的每个直接子元素的第0个，第1个，第2个…分别进行相关的操作。**

现在我们用刚刚理解的方式来做几个操作。比如现在有一个二维的数组：

```python
x = np.array([[0,1],[2,3]])
```

1. 求`x`数组在`axis=0`和`axis=1`两种情况下的和：

   ```python
   >>> x.sum(axis=0)
    array([2, 4])
   ```

   为什么得到的是[2,4]呢，原因是我们按照`axis=0`的方式进行相加，那么就会把**最外面轴下的所有直接子元素中的第0个位置进行相加，第1个位置进行相加…依此类推**，得到的就是`0+2`以及`2+3`，然后进行相加，得到的结果就是`[2,4]`。

   ```python
   >>> x.sum(axis=1)
    array([1, 5])
   ```

   因为我们按照`axis=1`的方式进行相加，那么就会把**轴为1里面的元素拿出来进行求和**，得到的就是`0,1`，进行相加为`1`，以及`2,3`进行相加为`5`，所以最终结果就是`[1,5]`了。
2. 用`np.max`求`axis=0`和`axis=1`两种情况下的最大值：

```python
>>> np.random.seed(100)
>>> x = np.random.randint(0,10,size=(3,5))
>>> x.max(axis=0)
array([8, 8, 3, 7, 8])
```

因为我们是按照`axis=0`进行求最大值，那么就会在最外面轴里面找直接子元素，然后将每个子元素的第0个值放在一起求最大值，将第1个值放在一起求最大值，以此类推。而如果`axis=1`，那么就是拿到每个直接子元素，然后求每个子元素中的最大值：

```python
>>> x.max(axis=1)
array([8, 5, 8])
```

3. 用`np.delete`在`axis=0`和`axis=1`两种情况下删除元素：

   ```python
   >>> np.delete(x,0,axis=0)
    array([[2, 3]])
   ```

   `np.delete`是个例外。我们按照`axis=0`的方式进行删除，那么他会首先找到最外面的括号下的直接子元素中的第0个，然后删掉，剩下最后一行的数据。

   ```python
   >>> np.delete(x,0,axis=1)
    array([[1],
           [3]])
   ```

   同理，如果我们按照`axis=1`进行删除，那么会把第一列的数据删掉。

##### 三维以上数组：

![axis2.png](images/axis2.png)

按照之前的理论，如果以上数组按照`axis=0`的方式进行相加，得到的结果如下：
![axis3.png](images/axis3.png)

如果是按照`axis=1`的方式进行相加，得到的结果如下：
![axis4.png](images/axis4.png)



### 第12节　通用函数

<!-- 来源：http://www.zlkt.net/book/detail/11/323 -->

#### 通用函数

##### 一、一元函数：

| 函数 | 描述 |
| --- | --- |
| np.abs | 绝对值 |
| np.sqrt | 开根 |
| np.square | 平方 |
| np.exp | 计算指数(e^x) |
| np.log，np.log10，np.log2，np.log1p | 求以e为底，以10为低，以2为低，以(1+x)为底的对数 |
| np.sign | 将数组中的值标签化，大于0的变成1，等于0的变成0，小于0的变成-1 |
| np.ceil | 朝着无穷大的方向取整，比如5.1会变成6，-6.3会变成-6 |
| np.floor | 朝着负无穷大方向取证，比如5.1会变成5，-6.3会变成-7 |
| np.rint，np.round | 返回四舍五入后的值 |
| np.modf | 将整数和小数分隔开来形成两个数组 |
| np.isnan | 判断是否是nan |
| np.isinf | 判断是否是inf |
| np.cos，np.cosh，np.sin，np.sinh，np.tan，np.tanh | 三角函数 |
| np.arccos，np.arcsin，np.arctan | 反三角函数 |

##### 二、二元函数：

| 函数 | 描述 |
| --- | --- |
| np.add | 加法运算（即1+1=2），相当于+ |
| np.subtract | 减法运算（即3-2=1），相当于- |
| np.negative | 负数运算（即-2），相当于加个负号 |
| np.multiply | 乘法运算（即2*3=6），相当于* |
| np.divide | 除法运算（即3/2=1.5），相当于/ |
| np.floor_divide | 取整运算，相当于// |
| np.mod | 取余运算，相当于% |
| greater,greater_equal,less,less_equal,equal,not_equal | >,>=,<,<=,=,!=的函数表达式 |
| logical_and | &的函数表达式 |
| logical_or | |的函数表达式 |

##### 三、聚合函数：

| 函数名称 | NAN安全版本 | 描述 |
| --- | --- | --- |
| np.sum | np.nansum | 计算元素的和 |
| np.prod | np.nanprod | 计算元素的积 |
| np.mean | np.nanmean | 计算元素的平均值 |
| np.std | np.nanstd | 计算元素的标准差 |
| np.var | np.nanvar | 计算元素的方差 |
| np.min | np.nanmin | 计算元素的最小值 |
| np.max | np.nanmax | 计算元素的最大值 |
| np.argmin | np.nanargmin | 找出最小值的索引 |
| np.argmax | np.nanargmax | 找出最大值的索引 |
| np.median | np.nanmedian | 计算元素的中位数 |

使用`np.sum`或者是`a.sum`即可实现。并且在使用的时候，可以指定具体哪个轴。同样`Python`中也内置了`sum`函数，但是Python内置的`sum`函数执行效率没有`np.sum`那么高，可以通过以下代码测试了解到：

```python
a = np.random.rand(1000000)
%timeit sum(a) #使用Python内置的sum函数求总和，看下所花费的时间
%timeit np.sum(a) #使用Numpy的sum函数求和，看下所花费的时间
```

##### 四、布尔数组的函数：

| 函数名称 | 描述 |
| --- | --- |
| np.any | 验证任何一个元素是否为真 |
| np.all | 验证所有元素是否为真 |

比如想看下数组中是不是所有元素都为0，那么可以通过以下代码来实现：

```python
np.all(a==0)
# 或者是
(a==0).all()
```

比如我们想要看数组中是否有等于0的数，那么可以通过以下代码来实现：

```python
np.any(a==0)
# 或者是
(a==0).any()
```

##### 五、排序：

1. `np.sort`：指定轴进行排序。默认是使用数组的最后一个轴进行排序。

   ```python
   a = np.random.randint(0,10,size=(3,5))
    b = np.sort(a) #按照行进行排序，因为最后一个轴是1，那么就是将最里面的元素进行排序。
    c = np.sort(a,axis=0) #按照列进行排序，因为指定了axis=0
   ```

   还有`ndarray.sort()`，这个方法会直接影响到原来的数组，而不是返回一个新的排序后的数组。
2. `np.argsort`：返回排序后的下标值。示例代码如下：

   ```python
   np.argsort(a) #默认也是使用最后的一个轴来进行排序。
   ```
3. 降序排序：`np.sort`默认会采用升序排序。如果我们想采用降序排序。那么可以采用以下方案来实现：

   ```python
   # 1. 使用负号
    -np.sort(-a)

    # 2. 使用sort和argsort以及take
    indexes = np.argsort(-a) #排序后的结果就是降序的
    np.take(a,indexes) #从a中根据下标提取相应的元素
   ```

##### 六、其他函数补充：

1. `np.apply_along_axis`：沿着某个轴执行指定的函数。示例代码如下：

   ```python
   # 求数组a按行求均值，并且要去掉最大值和最小值。
    np.apply_along_axis(lambda x:x[(x != x.max()) & (x != x.min())].mean(),axis=1,arr=a)
   ```
2. `np.linspace`：用来将指定区间内的值平均分成多少份。示例代码如下：

   ```python
   # 将0-1分成12分，生成一个数组
    np.linspace(0,1,12)
   ```
3. `np.unique`：返回数组中的唯一值。

   ```python
   # 返回数组a中的唯一值，并且会返回每个唯一值出现的次数。
    np.unique(a,return_counts=True)
   ```

##### 七、更多：

<https://docs.scipy.org/doc/numpy/reference/index.html>



### 第13节　Numpy练习题

<!-- 来源：http://www.zlkt.net/book/detail/11/324 -->

#### Numpy练习题

##### 一、查看Numpy的版本号：

```python
import numpy as np
print(np.__version__)
```

##### 二、如何创建一个所有值都是False的布尔类型的数组：

```python
np.full((3,3),False,dtype=np.bool)
```

##### 三、将一个有10个数的数组的形状进行转换：

```python
arr = np.arange(10)
arr.reshape(2,5) #转换成(2,5)的数组
arr[:,np.newaxis] #转换成(10,1)的数组
```

`np.newaxis`所处的位置，会变成1。比如：

```python
arr = np.random.randint(0,10,size=(10,2))
arr1 = arr[:,np.newaxis,:]
print(arr1.shape)
# 结果是(10,1,2)，因为np.newaxis所在的位置是1
```

##### 四、将数组中所有偶数都替换成0（改变原来数组和不改变原来数组两种方式实现）：

```python
arr = np.random.randint(0,10,size=(3,3))
# 1. 不改变原来数组
arr1 = np.where(arr%2==0,0,arr)
print(arr1)
# 2. 改变原来数组
arr[arr%2==0] = 0
```

##### 五、创建一个一维且有10个数的数组，元素是从`0-1`之间，但是不包含0和1：

```python
arr = np.linspace(0,1,12)[1:-1]
```

其中的`linspace`是在起始值和结束值之间平均的获取指定个数的数。比如以上就是从`0-1`之间获取12个数组。

##### 六、求以下数组大于等于5并且小于等于10的数组：

```python
a = np.arange(15)
# 方法1
index = np.where((a >= 5) & (a <= 10))
a[index]

# 方法2:
index = np.where(np.logical_and(a>=5, a<=10))
a[index]
#> (array([6, 9, 10]),)

# 方法3：
a[(a >= 5) & (a <= 10)]
```

##### 七、将一个二维数组的行和列分别进行逆向：

```python
a = np.arange(15).reshape(3,5)
# 反转行
a1 = a[::-1] #里面传一个数进去（没有出现逗号），代表的是只对行进行操作
# 反转列
a2 = a[:,::-1] #里面传两个数进去，第一个是所有的行，第二个就是针对所有的列，但是取值的方向是从后面到前面。
```

##### 八、如何将科学计数法转换为浮点类型打印：

```python
# set_printoptions用来设置打印的时候的一些配置和选项
# 将suppress设置为True，就不会显示成科学计数法了，并且通过precision来控制小数点后要保留几位
np.set_printoptions(suppress=True,precision=6)
rand_arr = np.random.random([3,3])/1e3
print(rand_arr)
```

##### 九、获取一个数组中唯一的元素：

```python
arr = np.random.randint(0,20,(10,10))
np.unique(arr)
```

##### 十、获取一个数组中唯一的元素个数的排行：

```python
arr = np.random.randint(0,20,(10,10))
np.unique(arr,return_counts=True)
```

##### 十一、如何找到数组中每行的最大值：

```python
# 解决方案1：
np.random.seed(100)
a = np.random.randint(1,10, [5,3])
print(a)
print("="*30)
print(np.amax(a,axis=1))

# 解决方案2：
print(np.apply_along_axis(np.max,arr=a,axis=1))
```

##### 十二、如何按照行求最小值与最大值相除的结果：

```python
np.random.seed(100)
a = np.random.randint(1,10, [5,3])
np.apply_along_axis(lambda x: np.min(x)/np.max(x),arr=a,axis=1)
```

##### 十三、判断两个数组是否完全相等：

```python
a = np.array([0,1,2])
b = np.arange(3)
(a == b).all()
```

##### 十四、设置一个数组不能修改值：

```python
a = np.zeros((2,2))
a.flags.writable = False
a[0] = 1
```

##### 十五、找到数组中离某个元素的最近的值：

```python
np.random.seed(100)
Z = np.random.uniform(0,1,10)
z = 0.5
m = Z[np.abs(Z - z).argmin()]
print(m)
```



## 第3章　Pandas库


### 第1节　pandas库介绍

<!-- 来源：http://www.zlkt.net/book/detail/11/372 -->

##### Pandas库介绍

![pandas.png](images/pandas.png)

`Pandas`的名称来源于面板数据（panel data）和数据分析（data anslysis）的组合，并不是熊猫的意思。

Pandas是一个强大的数据分析库，是基于Numpy构建的，提供了高级数据结构和数据操作工具。Pandas的出现，使得Python成为强大而高效的首选数据分析语言之一。Pandas有以下特点：

- 一个强大的分析和操作大型结构化数据集所需的工具集
- 基于NumPy，提供了高性能矩阵的运算
- 提供了大量能够快速便捷地处理数据的函数和方法
- 应用于数据挖掘，数据分析
- 提供数据清洗功能

##### 官网：

[http://pandas.pydata.org/](http://pandas.pydata.org)



### 第2节　pandas数据结构

<!-- 来源：http://www.zlkt.net/book/detail/11/373 -->

#### Pandas常用数据结构

##### 一、Series类型

在Pandas中，有两个基本的数据类型，使得在处理数据的时候变得非常高效，分别是Series和DataFrame，其中DataFrame是由多个Series组成的，因此要先学好Series，才能对DataFrame的使用更加得心应手。

Series是一种类似于列表的对象，由一列数据以及一列与之对应的索引组成。如下图所示。
![Series.png](images/Series.png)

###### 1. 创建Series对象：

在实际开发中，可能需要经常创建Series对象，而创建Series对象有多重方式，下面进行讲解。

###### 1.1. 通过列表或ndarray数组创建：

```python
import pandas as pd

persons = ['张三','李四','王五']
series = pd.Series(persons)

print(series)
print(type(series))
```

上述代码可以看到输出以上代码输出为：

```
0    张三
1    李四
2    王五
dtype: object

<class 'pandas.core.series.Series'>
```

###### 1.2. 通过字典创建：

```python
persons = {"张三":18, "李四": 25, "王五": 19}
series = pd.Series(persons)

print(series)
print(type(series))
```

上述代码的输出结果为：

```
张三    18
李四    25
王五    19
dtype: int64

<class 'pandas.core.series.Series'>
```

###### 2. Series对象相关操作：

###### 2.1. 获取数据和索引：

通过`series.index`和`series.values`能分别获取到`Series`对象的索引和值。示例代码如下：

```python
persons = ['张三','李四','王五']
series = pd.Series(persons)

print(series.index)
print(series.values)
```

输出结果为：

```
RangeIndex(start=0, stop=3, step=1)
['张三' '李四' '王五']
```

###### 2.2. 通过索引获取对应的值：

我们经常会通过索引来获取对应的值，语法为：`series[索引]`。示例代码如下：

```python
persons = ['张三','李四','王五']
series = pd.Series(persons)

print(series[0])
```

输出结果为：`'张三'`。

##### 二、DataFrame对象：

`DataFrame`由多行多列组成，是一个表格型的数据结构。可以认为一个DataFrame是由多个Series对象组成，这些Series共用同一个行索引，每个Series的数据类型可以不同。结构图如下。
![DataFrame.png](images/DataFrame.png)

###### 1. 创建DataFrame对象：

###### 1.1. 通过ndarray创建DataFrame对象：

二维数组天然具有行和列的概念，因此可以非常方便的使用二维数组创建DataFrame对象。示例代码如下：

```python
import numpy as np

# 通过ndarray创建DataFrame
array = np.random.randn(5,4)
print(array)

df_obj = pd.DataFrame(array)
print(df_obj.head())
```

运行结果如下：

```
[[ 1.70457214  0.53100369  0.37360947  0.93320688]
 [-0.56648031 -0.32116357 -0.3837961  -1.80175888]
 [ 2.12109864  0.83919019 -0.43187152  0.80611653]
 [-0.66275038  0.90984602  0.11596098  0.41519099]
 [-1.62119184  2.40474293  0.96915745 -0.28995527]]
          0         1         2         3
0  1.704572  0.531004  0.373609  0.933207
1 -0.566480 -0.321164 -0.383796 -1.801759
2  2.121099  0.839190 -0.431872  0.806117
3 -0.662750  0.909846  0.115961  0.415191
4 -1.621192  2.404743  0.969157 -0.289955
```

同理，用二维列表，也同样可以达到效果。可以在创建`DataFrame`的时候，指定行索引和列索引。示例代码如下：

```python
df = pd.DataFrame(array, index=[1,2,3,4,5], columns=['星期一','星期二','星期三','星期四']
```

输出结果如下：

```
     星期一     星期二     星期三    星期四
1  1.704572  0.531004  0.373609  0.933207
2 -0.566480 -0.321164 -0.383796 -1.801759
3  2.121099  0.839190 -0.431872  0.806117
4 -0.662750  0.909846  0.115961  0.415191
5 -1.621192  2.404743  0.969157 -0.289955
```

###### 1.2. 通过字典创建DataFrame：

```python
persons = [{"username":"张三","age": 18, "height": 180},{"username":"李四","age": 20, "height": 170}]
df = pd.DataFrame(persons)
print(df)
```

运行结果如下：

```
    username	age	height
0	张三	18	180
1	李四	20	170
```

上述是把多个字典存放到列表中，也可以直接使用字典创建，示例代码如下：

```python
persons = {"username":["张三","李四"],"age": [18, 20], "height": [180, 170]}
df = pd.DataFrame(persons)
print(df)
```

###### 2. DataFrame基本操作：

###### 2.1. 获取列的数据：

通过`df.列名`或者`df['列名']`都可以获取到该列的所有数据。示例代码如下：

```python
# 1. df.列名
print(df.username）
# 2. df['列名']
print(df['username'])
```

以上获取到的数据类型，实际上就是一个Series对象。

###### 2.2. 获取行的数据：

行的数据是通过索引来获取，在`DataFrame`中索引操作功能非常强大，这里我们讲解一个简单的示例。关于索引更多操作读者请参考下一节。

```python
# 获取下标为0的索引的那一行的值
print(df.iloc[0])
```

输出结果如下：

```
username     张三
age          18
height      180
Name: 0, dtype: object
```

###### 2.3. 增加列的数据：

可以通过类似字典操作的方式，给`DataFrame`添加新的列。示例代码如下：

```python
# 添加weight列
df['weight'] = [80, 60]
print(df)
```

输出结果如下：

```
	username    age    height	weight
0	张三	18	180	80
1	李四	20	170	60
```

###### 2.4. 删除列：

通过`del`关键字即可删除某一列的数据。示例代码如下：

```python
del df['weight']
print(df)
```

输出结果如下：

```
    username	age	height
0	张三	18	180
1	李四	20	170
```

###### 3. 查看数据：

###### 3.1. head

如果`DataFrame`数据行数特别多，我们只想看前面部分的数据，那么可以使用`df.head`来查看。示例代码如下：

```python
# 默认查看前面5条数据
df.head()

# 查看前面10条数据
df.head(10)
```

###### 3.2. tail：

与`head`相反，`tail`是用于查看`DataFrame`末尾的数据，默认是5条。示例代码如下：

```python
# 默认查看末尾5调数据
df.tail()

# 查看末尾10条数据
df.tail(10)
```

###### 3.3. describe：

查看`DataFrame`的描述。会把所有数字类型的列进行运算，比如获取总的个数，平均值，中位数等。

```python
import pandas as pd
persons = [{"username":"张三","age": 18, "height": 180},{"username":"李四","age": 20, "height": 170}]
df = pd.DataFrame(persons)
df.describe()
```

输出结果为：

```
        age	        height
count	2.000000	2.000000
mean	19.000000	175.000000
std	1.414214	7.071068
min	18.000000	170.000000
25%	18.500000	172.500000
50%	19.000000	175.000000
75%	19.500000	177.500000
max	20.000000	180.000000
```

###### 3.4. shape：

查看当前`DataFrame`的形状。示例代码如下：

```python
df.shape
```

###### 3.5. index和columns：

`df.index`用于查看当前的行索引。`df.columns`用于查看当前的列索引。示例代码如下：

```python
# 查看行索引
print(df.index)
# 查看列索引
print(df.columns)
```

###### 3.6. T：

`df.T`用于转置数组，可以将行变为列，列变为行。示例代码如下：

```python
print(df.T)
```

###### 3.7. info：

查看数据相关信息，比如列数、每列的类型等。

###### 3.8. dtypes：

查看`DataFrame`的所有列的数据类型。



### 第3节　pandas索引操作

<!-- 来源：http://www.zlkt.net/book/detail/11/374 -->

#### Pandas索引操作

`Pandas`中的索引操作非常灵活，功能非常强大。学会他的索引操作能帮助我们更好的处理数据。下面来对索引进行讲解。

##### 一、索引类型：

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

###### 1. RangeIndex：

区间索引，用法与Python中的`range`函数类似，可以指定`start`、`stop`、`step`参数。示例代码如下：

```python
df.index = pd.RangeIndex(start=1, stop=9, step=2)
```

###### 1. NumericIndex：

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

###### 2. CategoricalIndex：

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

###### 3. IntervalIndex：

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

###### 4. DatetimeIndex：

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

###### 5. TimedeltaIndex：

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

##### 二、Series索引：

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

###### 1. 行索引：

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

###### 2. 切片索引：

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

###### 3. 不连续索引：

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

###### 4. 布尔索引：

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

##### 三、DataFrame索引：

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

###### 1. 列索引：

`DataFrame`中包含列索引，可以通过以下方式来获取列索引的数据：

```python
# 只获取一列，返回series类型
print(df["a"])
# 获取多列，返回DataFrame类型
print(df[["a", "b"]])
```

###### 2. loc索引：

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

###### 3. iloc索引：

作用和`loc`一样，区别是通过索引下标来实现的。示例代码如下：

```python
# 获取第1-2行的所有列
df.iloc[1:3]

# 获取第1，3行的所有列
df.iloc[[1,3]]

# 获取第1行的第1-3列
df.iloc[1, 1:4]
```

##### 四、重置索引

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

###### 1. set\_index：

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

###### 2. reset\_index：

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

###### 3. reindex：

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

###### 4. 修改DataFrame的index属性：

直接修改`index`属性也可以实现修改索引的目的，但是他有一个限制，就是新索引的数量，必须和原索引数量一致，否则会报错。**这个方法会修改原始DataFrame对象。** 示例代码如下：

```python
df.index = ['a', 'b', 'c', 'd' ]
```

还有一个需要注意的是，这种方法会直接修改原始`DataFrame`对象。

**应用场景：** 需要修改原始`DataFrame`对象的索引值。



### 第4节　pandas数据类型转换

<!-- 来源：http://www.zlkt.net/book/detail/11/375 -->

#### Pandas数据类型转换

##### 一、Pandas中的数据类型：

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

##### 案例数据文件：

这里我们以一个`sales_data_types.csv`文件为例。来讲解后面的知识点。读取代码如下：

```python
import pandas as pd
import numpy as np

df = pd.read_csv("data/sales_data_types.csv")
df.head()
```

输出结果为：
![sales_data_types.png](images/sales_data_types.png)

##### 数据类型相关操作：

###### 1. 查看DataFrame所有列的类型：

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

###### 2. 用astype转换类型：

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

###### 3. 自定义转换函数：

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

###### 4. 使用`np.where`更换数据类型：

比如`df['Active']`这列，我们可以认为只要值是`Y`，那么就设置为`True`，否则就设置为`False`。代码如下：

```python
np.where(df['Active']=='Y', True, False)
```

###### 5. pandas工具类函数：

###### pd.to\_numeric函数：

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

###### pd.to\_datetime函数：

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

##### 综合在一起：

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



### 第5节　pandas文件操作

<!-- 来源：http://www.zlkt.net/book/detail/11/376 -->

#### 文件操作：

Pandas中提供了许多的操作文件的函数，包括读取和写入。我们做数据分析用得最多的，就是`CSV`、`Excel`、`SQL`、`JSON`文件。下面来针对这几种文件的操作做一个详细的讲解。

##### CSV文件操作：

读写`CSV`文件分别用的是`pd.read_csv`和`pd.to_csv`方法。普通用法非常简单，但是通过一些参数，可以实现许多高级操作。

###### 1. 读取csv：

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

###### 2. 写入csv：

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

##### Excel文件操作：

###### 1. 读取Excel：

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

###### 2. 写入Excel：

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

##### SQL操作：

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

###### 1. 写入SQL：

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

###### 2. read\_sql\_table读取表数据：

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

###### 3. read\_sql\_query执行SQL查询语句：

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

###### 4. read\_sql读取数据：

`read_sql`是结合了`read_sql_table`和`read_sql_query`两者的功能。既可以从表中读取数据，也可以执行查询SQL语句。示例代码如下：

```python
# 读取表数据
funnel = pd.read_sql("funnel", engine)
# 执行查询SQL语句
funnel = pd.read_sql("select Account, Name from funnel where Product='CPU'", engine)
```

##### 更多：

更多其他文件操作，请参考Pandas官网：<https://pandas.pydata.org/docs/user_guide/io.html>



### 第6节　pandas缺失值处理

<!-- 来源：http://www.zlkt.net/book/detail/11/377 -->

#### pandas缺失值处理

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

##### 一、判断是否为NAN值：

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

##### 二、填充NAN数据：

###### 1. 填充常量值：

如果想要把`DataFrame`或者是`Series`中`NAN`的数据用其他值替换，那么可以使用`fillna`。示例代码如下：

```python
# 把所有`NA`的值填充为0
df.fillna(0)

# 把某一列的值进行填充
df['one'].fillna("missing")
```

###### 2. 使用计算值填充：

我们可以使用某一列的最大值、最小值、平均值等来填充`NAN`值，示例代码如下。

```python
# 对所有列都用平均值填充
df.fillna(df.mean())

# 对one到three列用平均值填充
df.fillna(df.mean()["one": "three"])
```

###### 2. 删除NAN值：

删除`NAN`值可以使用`dropna`来实现。删除`NAN`的操作，会把含有`NAN`值的整行或者整列都删掉，因此要慎用。示例代码如下。

```python
# 按照行删除
df.dropna()
# 或者是
df.dropna(axis=0)

# 按照列删除
df.dropna(axis=1)
```

###### 3. 替换：

针对一些满足条件的字符串，可以使用`replace`方法来替换，这个方法还可以使用正则表达式。示例代码如下。

```python
d = {"a": list(range(4)), "b": list("ab.."), "c": ["a", "b", np.nan, "d"]}
df = pd.DataFrame(d)
df.replace(".", np.nan)
```

##### 三、更多：

更多关于Pandas缺失值的处理方法，请参考官方文档：<https://pandas.pydata.org/docs/user_guide/missing_data.html>



### 第7节　pandas常用操作

<!-- 来源：http://www.zlkt.net/book/detail/11/378 -->

#### Pandas常用操作

Pandas中提供了许多函数，可以帮我们快速处理数据。这里我们来讲一下Pandas的一些常用操作。

##### apply函数：

`apply`函数可以将每行或者每列的数据，放到某个函数中进行处理。比如要获取所有列的最大值，那么通过`apply`函数就非常容易的实现。比如有以下`DataFrame`结构的数据。

```python
import numpy as np
import pandas as pd
df = pd.DataFrame(np.random.randn(5,4) - 1)
print(df)
```

输出结果为：

```
        0	        1	        2	        3
0	-1.392550	-2.303422	-1.842063	-3.295022
1	0.536343	-1.594769	-1.213821	-1.235229
2	0.187501	-1.256436	-0.175250	0.006346
3	-0.678881	-1.280859	-2.835340	-2.012439
4	0.742178	-0.613463	-0.644243	0.927003
```

如果想要求所有列的最大值，那么通过以下代码就可以实现：

```python
df.apply(lambda x: x.max())
```

输出结果为：

```
0    0.742178
1   -0.613463
2   -0.175250
3    0.927003
dtype: float64
```

`apply`默认的`axis`参数（轴）是等于0，也就是代表列。如果想要求行的最大值，那么可以通过设置`axis=1`来实现。示例代码如下：

```python
df.apply(lambda x: x.max(), axis=1)
```

输出结果为：

```
0   -1.392550
1    0.536343
2    0.187501
3   -0.678881
4    0.927003
dtype: float64
```

##### applymap函数：

`apply`可以一次性处理行和列的数据，而如果想要批量处理`DataFrame`的所有数据，比如给每个数据都乘以10，那么可以通过`applymap`来实现。示例代码如下：

```python
df.applymap(lambda x: x*10)
```

输出结果为：

```
        0	        1	        2	        3
0	-13.925498	-23.034218	-18.420631	-32.950220
1	5.363431	-15.947688	-12.138206	-12.352289
2	1.875006	-12.564360	-1.752497	0.063457
3	-6.788815	-12.808594	-28.353398	-20.124392
4	7.421781	-6.134635	-6.442430	9.270026
```

##### 排序：

排序主要是分为两个。一个是按照某个列排序，另外一个是按照索引排序。下面分别来进行讲解。

###### 1. 按照列排序：

按照列进行排序，是使用`sort_values`，可以传递`ascending`参数，用来指定排序方法。如果`ascending=False`，那么将按照降序排序，否则按照升序排序，默认是升序排序。示例代码如下：

```python
df.sort_values("0", ascending=False)
```

###### 2. 按照索引排序：

按照索引排序用的是`sort_index`，用法与`sort_values`类似。但是因为索引只有一列，因此不需要额外指定排序的字段。示例代码如下：

```python
df.sort_index(ascending=False)
```

##### 数据运算：

###### 1. 算术运算:

- add(other):
  比如想要将`Series`中的每一个数据都加1，那么可以使用以下方式来实现：

```python
data['open'].add(1)

2018-02-27    24.53
2018-02-26    23.80
2018-02-23    23.88
2018-02-22    23.25
2018-02-14    22.49
```

- sub(other)：
  相减。比如想要知道每天的涨跌大小，那么可以通过以下代码实现：

```python
# 1、筛选两列数据
close = data['close']
open = data['open']
# 2、收盘价减去开盘价
data['m_price_change'] = close.sub(open)
# 或者是
data['m_price_change'] = close - open
data.head()

            open     high   close   low   volume  price_change  p_change  turnover my_price_change
2018-02-27    23.53    25.88    24.16    23.53    95578.03    0.63    2.68    2.39    0.63
2018-02-26    22.80    23.78    23.53    22.80    60985.11    0.69    3.02    1.53    0.73
2018-02-23    22.88    23.37    22.82    22.71    52914.01    0.54    2.42    1.32    -0.06
2018-02-22    22.25    22.76    22.28    22.02    36105.01    0.36    1.64    0.90    0.03
2018-02-14    21.49    21.99    21.92    21.48    23331.04    0.44    2.05    0.58    0.43
```

###### 2. 逻辑运算：

- 逻辑运算符号<、 >、|、 &
  比如我们想要，筛选p\_change > 2并且open > 15的值。代码如下：

```python
data[(data['p_change'] > 2) & (data['open'] > 15)]

open    high    close    low    volume    price_change    p_change    turnover    my_price_change
2017-11-14    28.00    29.89    29.34    27.68    243773.23    1.10    3.90    6.10    1.34
2017-10-31    32.62    35.22    34.44    32.20    361660.88    2.38    7.42    9.05    1.82
2017-10-27    31.45    33.20    33.11    31.45    333824.31    0.70    2.16    8.35    1.66
2017-10-26    29.30    32.70    32.41    28.92    501915.41    2.68    9.01    12.56    3.11
```

- query函数：
  通过`query`函数，上例中的实现将更加简单。

```python
data.query("p_change > 2 & turnover > 15")
```

- isin函数：
  例如判断’turnover’是否为4.19, 2.39。代码如下：

```python
# 可以指定值进行一个判断，从而进行筛选操作
data[data['turnover'].isin([4.19, 2.39])]

open    high    close    low    volume    price_change    p_change    turnover    my_price_change
2018-02-27    23.53    25.88    24.16    23.53    95578.03    0.63    2.68    2.39    0.63
2017-07-25    23.07    24.20    23.70    22.64    167489.48    0.67    2.91    4.19    0.63
2016-09-28    19.88    20.98    20.86    19.71    95580.75    0.98    4.93    2.39    0.98
2015-04-07    16.54    17.98    17.54    16.50    122471.85    0.88    5.28    4.19    1.00
```

###### 3. 统计函数：

| 函数名 | 描述 |
| --- | --- |
| count | 非空值的个数 |
| sum | 求和 |
| mean | 平均值 |
| median | 中位数 |
| min | 最小值 |
| max | 最大值 |
| abs | 绝对值 |
| prod | 数组元素的乘积 |
| std | 标准差 |
| var | 方差 |
| idxmax | 最大值的位置 |
| idxmin | 最小值的位置 |

###### 4. 累计函数：

| 函数 | 描述 |
| --- | --- |
| cumsum | 计算前1/2/3/…/n个数的和 |
| cummax | 计算前1/2/3/…/n个数的最大值 |
| cummin | 计算前1/2/3/…/n个数的最小值 |
| cumprod | 计算前1/2/3/…/n个数的积 |



### 第8节　pandas数据离散化

<!-- 来源：http://www.zlkt.net/book/detail/11/379 -->

#### pandas数据离散化

数据离散化，是将连续的数据，通过分割，形成离散化的数据。举个例子，比如有一列数据存储人的身高：`165，174，160，180，159，163，192，184`，那么通过离散化可以变为：`150~165, 165~180,180~195`。还有另外一种离散化的数据，就是通过`one-hot`编码，下面会详细讲到。

##### 切割数据离散化：

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

##### One-Hot编码离散化：

`One-Hot`编码是将分类数据的所有项，全部都变成列，然后如果某一行中出现这一列，那么就标记为1，否则就标记为0。比如下图将左边的`Category`变为`One-Hot`编码后，会把`Category`中所有唯一的值都添加为新的列。
![one_hot编码.png](images/one_hot编码.png)

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



### 第9节　pandas合并操作

<!-- 来源：http://www.zlkt.net/book/detail/11/380 -->

#### pandas合并操作

在实际工作中，我们的数据经常存储在多个文件中，这时候就需要挨个读取出来，然后合并成一个`DataFrame`对象。在`pandas`中，可以通过`pd.concat`和`pd.merge`来实现合并的功能。

##### pd.concat：

`pd.concat(datas, axis=1)`，按照行或者列合并多个数据，`axis=0`为列索引，`axis=1`为行索引。比如我们以二手车数据为例，合并广州和北京的二手车数据。示例代码如下：

```python
df_gz = pd.read_csv("data/guazi_gz.csv")
df_bj = pd.read_csv("data/guazi_bj.csv")

df = pd.concat([df_gz, df_bj])
```

其中`df_gz`和`df_bj`的列名都是一样的，上述代码是将多行合并在一起。

如果要将不同列的数据合并在一起，那么则根据行索引名称进行拼接。

##### pd.merge：

`pd.merge(left, right, how="inner", on=None, left_on=None, right_one=None)`类似于`SQL`语句中的连接。都是指定按照共同键值对合并或者左右内连接。参数意义如下：

- `left`和`right`：两个需要合并的`DataFrame`对象。
- `how`：指定合并的方式。有以下可选参数。

  | Merge Method | SQL Join Name | 描述 |
  | --- | --- | --- |
  | left | LEFT OUTER JOIN | 只使用左边的DataFrame的key作为连接字段 |
  | right | RIGHT OUTER JOIN | 只使用右边的DataFrame的key作为连接字段 |
  | outer | FULL OUTER JOIN | 使用左边和右边的key值的并集连接 |
  | inner | INNER JOIN | 使用左边和右边的key值的交集连接 |
- `on`：按照哪个字段进行合并，指定的键必须在两个`DataFrame`中都存在。
- `left_on`：左连接的字段。
- `right_on`：右连接的字段。

###### pd.merge合并：

1. 使用`left_on`和`right_on`参数合并：

```python
df1 = pd.DataFrame({'lkey': ['foo', 'bar', 'baz', 'foo'],
                    'value': [1, 2, 3, 5]})

df2 = pd.DataFrame({'rkey': ['foo', 'bar', 'baz', 'foo'],
                    'value': [5, 6, 7, 8]})

print(df1)
print(df2)
```

输出结果如下：

```
        lkey	value
0	foo	1
1	bar	2
2	baz	3
3	foo	5

	rkey	value
0	foo	5
1	bar	6
2	baz	7
3	foo	8
```

执行`merge`操作代码如下：

```python
pd.merge(df1, df2, left_on="lkey", right_on="rkey")
```

输出结果为：

```
        lkey	value_x	rkey	value_y
0	foo	1	foo	5
1	foo	1	foo	8
2	foo	5	foo	5
3	foo	5	foo	8
4	bar	2	bar	6
5	baz	3	baz	7
```

2. 使用`on`参数合并：
   案例对象如下：

```python
left = pd.DataFrame({'key1': ['K0', 'K0', 'K1', 'K2'],
                        'key2': ['K0', 'K1', 'K0', 'K1'],
                        'A': ['A0', 'A1', 'A2', 'A3'],
                        'B': ['B0', 'B1', 'B2', 'B3']})

right = pd.DataFrame({'key1': ['K0', 'K1', 'K1', 'K2'],
                        'key2': ['K0', 'K0', 'K0', 'K0'],
                        'C': ['C0', 'C1', 'C2', 'C3'],
                        'D': ['D0', 'D1', 'D2', 'D3']})
```

**内连接：**

```python
result = pd.merge(left, right, on=['key1', 'key2'])
```

![内连接.png](images/内连接.png)

**左连接：**

```python
result = pd.merge(left, right, how='left', on=['key1', 'key2'])
```

![左连接.png](images/左连接.png)

**右连接：**

```python
result = pd.merge(left, right, how='right', on=['key1', 'key2'])
```

![右连接.png](images/右连接.png)

**外连接：**

```python
result = pd.merge(left, right, how='outer', on=['key1', 'key2'])
```

![外链接.png](images/外链接.png)



### 第10节　pandas分组与聚合

<!-- 来源：http://www.zlkt.net/book/detail/11/381 -->

#### Pandas分组与聚合

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

##### 1. 根据`fruit`进行分组：

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

##### 2. 根据`fruit`和`color`进行分组：

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

##### 3. 聚合函数：

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



### 第11节　交叉与透视表

<!-- 来源：http://www.zlkt.net/book/detail/11/382 -->

#### pandas交叉与透视表

##### 一、交叉表：

交叉表是将`DataFrame`中两列的数据进行交叉，其中一列的值作为行索引，另一列的值作为列索引，然后求交叉数据出现的次数。也可以通过指定`aggfunc`和`values`，来改变两列交叉后的结果。这里我们先创建一个简单的`DataFrame`测试数据：

```python
df = pd.DataFrame({'A': [1, 2, 2, 2, 2],
                   'B': [3, 3, 4, 4, 4],
                   'C': [1, 1, np.nan, 1, 1]})
df.head()
```

输出结果如下：

```
	A	B	C
0	1	3	1.0
1	2	3	1.0
2	2	4	NaN
3	2	4	1.0
4	2	4	1.0
```

###### 1. 默认使用：

我们求`A`列和`B`列交叉后出现的次数，那么可以通过以下代码实现：

```
pd.crosstab(df['A'], df['B'])
```

输出结果如下：

```
B	3	4
A
1	1	0
2	1	3
```

可以看到，`A`列为1，`B`列为3的数据，总共出现了1次，而`A`列为2，`B`列为4的数据，总共出现了3次。

###### 2. 使用聚合函数：

我们也可以使用聚合函数，来修改聚合后的计算结果，这里我们计算`C`这一列的总和，那么代码如下：

```python
pd.crosstab(df['A'], df['B'], values=df['C'], aggfunc=np.sum)
```

输出结果如下：

```
B	3	4
A
1	1.0	NaN
2	1.0	2.0
```

上述代码中，在`A=1`，`B=3`的前提下，`C`列的所有值的总和为1。而在`A=1`，`B=4`的前提下，`C`列所有值的总和为2。

##### 二、透视表：

透视表有点类似于`groupby`，先通过`index`参数指定分组的列，然后再通过`values`参数指定聚合的列，再通过`aggfunc`参数指定聚合的函数。我们使用以下测试数据集：

```python
df = pd.DataFrame({"A": ["foo", "foo", "foo", "foo", "foo",
                         "bar", "bar", "bar", "bar"],
                   "B": ["one", "one", "one", "two", "two",
                         "one", "one", "two", "two"],
                   "C": ["small", "large", "large", "small",
                         "small", "large", "small", "small",
                         "large"],
                   "D": [1, 2, 2, 3, 3, 4, 5, 6, 7],
                   "E": [2, 4, 5, 5, 6, 6, 8, 9, 9]})
df
```

输出结果如下：

```
	A	B	C		D	E
0	foo	one	small	1	2
1	foo	one	large	2	4
2	foo	one	large	2	5
3	foo	two	small	3	5
4	foo	two	small	3	6
5	bar	one	large	4	6
6	bar	one	small	5	8
7	bar	two	small	6	9
8	bar	two	large	7	9
```

###### 1. 默认使用：

```python
df.pivot_table(index='A')
```

输出结果如下：

```
	D	E
A
bar	5.5	8.0
foo	2.2	4.4
```

默认是根据`index`的列进行分组，然后求类型为数值型的列的平均值。

###### 2. 根据`A`列进行分组，统计每个值出现的次数。示例代码如下：

```python
df.pivot_table(index='A', aggfunc=np.count_nonzero)
```

输出结果如下：

```
	B	C	D	E
A
bar	4	4	4	4
foo	5	5	5	5
```

###### 3. 根据`A`列进行分组，统计`D`列的值的总和。示例代码如下：

```python
df.pivot_table(index='A', values=['D'], aggfunc=np.sum)
```

输出结果如下：

```
	D
A
bar	22
foo	11
```

###### 4. 根据`A`列进行分组，统计`D`列的值的总和，并且根据`B`列进行区分。示例代码如下：

```python
df.pivot_table(index='A', values=['D'], columns=['B'], aggfunc=np.sum)
```

输出结果如下：

```
	D
B	one	two
A
bar	9	13
foo	5	6
```



## 第4章　Matplotlib库


### 第1节　数据分析常见图

<!-- 来源：http://www.zlkt.net/book/detail/11/325 -->

#### 数据分析中常用图

##### 一、折线图：

折线图用于显示数据在一个连续的时间间隔或者时间跨度上的变化，它的特点是反映事物随时间或有序类别而变化的趋势。示例图如下：
![折线图示例.png](images/折线图示例.png)

折线图应用场景：

1. 折线图适合`X`轴是一个连续递增或递减的，对于没有规律的，则不适合使用折线图，建议使用柱状图。
2. 如果折线图条数过多，则不应该都绘制在一个图上。

##### 二、柱状图：

典型的柱状图（又名条形图），使用垂直或水平的柱子显示类别之间的数值比较。其中一个轴表示需要对比的分类，另一个轴代表相应的数值。

柱状图有别于直方图，柱状图无法显示数据在一个区间内的连续变化趋势。柱状图描述的是分类数据，回答的是每一个分类中“有多少？”这个问题。 示例图如下：
![柱状图示例.png](images/柱状图示例.png)

柱状图应用场景：

1. 适用于分类数据对比。
2. 垂直条形图最多不超过12个分类（也就是12个柱形），横向条形图最多不超过30个分类。如果垂直条形图的分类名太长，那么建议换成横向条形图。
   ![城市人口数量垂直柱状图.png](images/城市人口数量垂直柱状图.png)
   ![城市人口数量横向柱状图.png](images/城市人口数量横向柱状图.png)
3. 柱状图不适合表示趋势，如果想要表示趋势，应该使用折线图。

##### 三、直方图：

直方图(Histogram)，又称质量分布图，是一种统计报告图，由一系列高度不等的条纹表示数据分布的情况。一般用横轴表示数据类型，纵轴表示分布情况。
直方图是数值数据分布的精确图形表示。为了构建直方图，第一步是将值的范围分段，即将整个值的范围分成一系列间隔，然后计算每个间隔中有多少值。这些值通常被指定为连续的，不重叠的变量间隔。间隔必须相邻，并且通常是（但不是必须的）相等的大小。
![电影时间直方图.png](images/电影时间直方图.png)

直方图的应用场景：

1. 显示各组数据数量分布的情况。
2. 用于观察异常或孤立数据。
3. 抽取的样本数量过小，将会产生较大误差，可信度低，也就失去了统计的意义。因此，样本数不应少于50个。

##### 四、散点图：

散点图也叫 X-Y 图，它将所有的数据以点的形式展现在直角坐标系上，以显示变量之间的相互影响程度，点的位置由变量的数值决定。

通过观察散点图上数据点的分布情况，我们可以推断出变量间的相关性。如果变量之间不存在相互关系，那么在散点图上就会表现为随机分布的离散的点，如果存在某种相关性，那么大部分的数据点就会相对密集并以某种趋势呈现。数据的相关关系主要分为：正相关（两个变量值同时增长）、负相关（一个变量值增加另一个变量值下降）、不相关、线性相关、指数相关等，表现在散点图上的大致分布如下图所示。那些离点集群较远的点我们称为离群点或者异常点。
![](images/散点图相关性.png)

示例图如下：
![散点图示例.jpg](images/散点图示例.jpg)

散点图的应用场景：

1. 观察数据集的分布情况。
2. 通过分析规律，根据样本数据特征计算出回归方程。

##### 五、饼状图：

饼状图通常用来描述量、频率和百分比之间的关系。在饼图中，每个扇区的弧长大小为其所表示的数量的比例。
![饼状图示例.png](images/饼状图示例.png)

饼状图的应用场景：

1. 展示多个分类的占比情况，分类数量建议不超过9个。
2. 对于一些占比值非常接近的，不建议使用饼状图，可以使用柱状图。

##### 六、箱线图：

箱线图（Box-plot）又称为盒须图、盒式图或箱型图，是一种用作显示一组数据分散情况资料的统计图。因形状如箱子而得名。在各种领域也经常被使用，它主要用于反映原始数据分布的特征，还可以进行多组数据分布特征的比较。箱线图的绘制方法是：先找出一组数据的**上限值、下限值、中位数（Q2）和下四分位数（Q1）以及上四分位数（Q3）**；然后，连接两个四分位数画出箱子；再将最大值和最小值与箱子相连接，中位数在箱子中间。
![箱线图介绍.jpg](images/箱线图介绍.jpg)
![箱线图案例.jpg](images/箱线图案例.jpg)

> 四分位数（Quartile）也称四分位点，是指在统计学中把所有数值由小到大排列并分成四等份，处于三个分割点位置的数值。多应用于统计学中的箱线图绘制。它是一组数据排序后处于25%和75%位置上的值。四分位数是通过3个点将全部数据等分为4部分，其中每部分包含25%的数据。很显然，中间的四分位数就是中位数，因此通常所说的四分位数是指处在25%位置上的数值（称为下四分位数）和处在75%位置上的数值（称为上四分位数）。与中位数的计算方法类似，根据未分组数据计算四分位数时，首先对数据进行排序，然后确定四分位数所在的位置，该位置上的数值就是四分位数。与中位数不同的是，四分位数位置的确定方法有几种，每种方法得到的结果会有一定差异，但差异不会很大。
>
> 上限的计算规则是：
> IQR=Q3-Q1
> 上限=Q3+1.5IQR
> 下限=Q1-1.5IQR

箱线图的应用场景：

1. 直观明了地识别数据中的异常值。
2. 利用箱线图判断数据的偏态。
3. 利用箱线图比较几批数据的形状。
4. 箱线图适合比较多组数据，如果知识要看一组数据的分布情况，建议使用直方图。

##### 七、更多参考：

<https://antvis.github.io/vis/doc/chart/classify/compare.html>



### 第2节　基本使用

<!-- 来源：http://www.zlkt.net/book/detail/11/326 -->

#### Matplotlib库基本使用

`Matplotlib`是一个`Python`的`2D`绘图库，通过`Matplotlib`，开发者可以仅需要几行代码，便可以生成折线图，直方图，条形图，饼状图，散点图等。

##### 一、安装：

如果是用`Anaconda`，可以通过`conda install matplotlib`或者通过`pip install matplotlib`进行安装。

##### 二、基本使用：

首先先看以下例子：

```python
import matplotlib.pyplot as plt
import numpy as np
plt.plot(range(10),[np.random.randint(0,10) for x in range(10)])
```

那么就会出现以下图：
![matplotlib1.png](images/matplotlib1.png)
其中`plot`是一个画图的函数，他的参数为`plot([x],y,[fmt],data=None,**kwargs)`。其中`fmt`可以传一个字符串，用来给这个图做一些样式修改的。默认的绘制样式是`b-`，也就是蓝色实体线条。比如我想将原来的图的线条改成点状，那么可以通过以下代码实现：

```python
import matplotlib.pyplot as plt
plt.plot(range(10),[np.random.randint(0,10) for x in range(10)],":")
```

###### 2.1. 折线图类型：

其中使用`:`代表点线，是`matplotlib`的一个缩写。这些缩写还有以下的：

| 字符 | 类型 | 字符 | 类型 |
| --- | --- | --- | --- |
| ‘-’ | 实线 | ‘–’ | 虚线 |
| ‘-.’ | 虚点线 | ‘:’ | 点线 |
| ‘.’ | 点 | ‘,’ | 像素点 |
| ‘o’ | 圆点 | ‘v’ | 下三角点 |
| ‘^’ | 上三角点 | ‘<’ | 左三角点 |
| ‘>’ | 右三角点 | ‘1’ | 下三叉点 |
| ‘2’ | 上三叉点 | ‘3’ | 左三叉点 |
| ‘4’ | 右三叉点 | ‘s’ | 正方点 |
| ‘p’ | 五角点 | ‘*’ | 星形点 |
| ‘h’ | 六边形点1 | ‘H’ | 六边形点2 |
| ‘+’ | 加号点 | ‘x’ | 乘号点 |
| ‘D’ | 实心菱形点 | ‘d’ | 瘦菱形点 |
| ‘_’ | 横线点 |  |  |

除了设置线条的形状外，我们还可以设置点的颜色。示例代码如下：

```python
plt.plot([1,2,3,4,5],[1,2,3,4,5],'r') #将颜色线条设置成红色
plt.plot([1,2,3,4,5],[1,2,3,4,5],color='red') #将颜色设置成红色
plt.plot([1,2,3,4,5],[1,2,3,4,5],color='#000000') #将颜色设置成纯黑色
plt.plot([1,2,3,4,5],[1,2,3,4,5],color=(0,0,0,0)) #将颜色设置成纯黑色
```

给线条设置颜色总体来说有三种方式，第一种是使用颜色名称（`r`是`red`的缩写）的形式，第二种是使用十六进制的方式，第三种是使用`RGB`或`RGBA`的方式。如果使用的是颜色名称，那么可以和线的形状写在同一个字符串中。比如使用红色的五角点，那么可以使用如下的方式实现：

```python
plt.plot([1,2,3,4,5],[1,2,3,4,5],'rp') #将颜色线条设置成红色
```

###### 2.2. 线条颜色：

其中可以表示颜色的缩写字符有如下：

| 字符 | 颜色 |
| --- | --- |
| ‘b’ | 蓝色，blue |
| ‘g’ | 绿色，green |
| ‘r’ | 红色，red |
| ‘c’ | 青色，cyan |
| ‘m’ | 品红，magenta |
| ‘y’ | 黄色，yellow |
| ‘k’ | 黑色，black |
| ‘w’ | 白色，white |

##### 三、设置图的信息：

现在我们添加图后，没有指定x轴代表什么，y轴代表什么，以及这个图的标题是什么。因此以下我们通过一些属性来设置一下。

###### 3.1. 设置线条样式：

1. 使用`plot`方法：`plot`方法就是用来绘制线条的，因此可以在绘制的时候就把线条相关的样式通过参数传进去。示例代码如下：

   ```python
   plt.plot(x,y,linewidth=2)
   ```
2. 通过`Line2D`对象来设置：`plot`方法会返回一个装有`Line2D`对象的列表，比如`lines=plt.plot(x1,y1,x2,y2)`因为绘制了两根线条，因此`lines`中会有两个`2D`对象。而如果`plot`只绘制一根线条，那么`lines`中就只有一个`Line2D`对象。拿到这个`Line2D`对象后就可以通过`set_属性名`设置线条的样式了：

   ```python
   lines = plt.plot(x,y)
    line = lines[0]
    line.set_aa(False) #关掉反锯齿
    line.set_alpha(0.5) #设置0.5的透明度
   ```
3. 使用`plt.setp`来设置：`setp`的好处是一次性可以设置多根线条的样式。示例代码如下：

   ```python
   lines = plt.plot(x,y)
    plt.setp(lines,linewidth=10,alpha=0.5)
   ```
4. 更多`Line2D`属性：
   ![Line2D属性表.png](images/Line2D属性表.png)

###### 3.2. 设置轴和标题：

1. 设置轴名称：可以通过`plt.xlabel`和`plt.ylabel`来设置`x`轴和`y`轴的的名称。示例代码如下：

   ```python
   plt.plot(x,y,linewidth=10,color='red')
    plt.xlabel("x轴")
    plt.ylabel("y轴")
   ```

   默认情况下是显示不了中文的。需要设置字体。可以通过以下代码来实现：

   ```python
   # 加载字体
    font = font_manager.FontProperties(fname="C:\Windows\Fonts\msyh.ttc")
    plt.plot(x,y,linewidth=10,color='red')
    plt.xlabel("x轴",fontproperties=font)
    plt.ylabel("y轴",fontproperties=font)
   ```

   加载字体的时候，可以到`C:\Windows\Fonts`中找你喜欢的并且可以显示中文的字体。找到字体后，还需要找到字体的真实名称。方法是右键->属性->安全->对象名称：
   ![matplotlib3.png](images/matplotlib3.png)
2. 设置标题：可以通过`plt.title`方法来实现。示例代码如下：

   ```python
   font = font_manager.FontProperties(fname="C:\Windows\Fonts\msyh.ttc")
   plt.title("sin函数",fontproperties=font)
   ```
3. 设置`x`轴和`y`轴的刻度：之前我们画的图，`x`轴和`y`轴的刻度都是`matplotlib`自动生成的。如果想要在生成图的时候手动的指定，那么可以通过`plt.xticks`和`plt.yticks`来实现：

   ```python
   plt.xticks(range(0,20,2)) #在x轴上的刻度是0,2,4,6...20
   ```

   以上会把那个刻度显示在`x`轴上。如果想要显示字符串类型，那么可以再构造一个数组，这个数组的长度必须和`x`轴刻度的长度保持一致。然后传给`xticks`的第二个参数。示例代码如下：

   ```python
   _x = range(0,20,2)
   _xticks = ["%d坐标"%i for i in _x]
   plt.xticks(_x,_xticks,fontproperties=font) #在x轴上的刻度是0坐标,2坐标...20坐标
   ```

   ![matplotlib4.png](images/matplotlib4.png)
   同样`y`轴的刻度设置也是一样的。示例代码如下：

   ```python
   _y = np.arange(-1,1,0.25)
   _yticks = ["%.2f点"%i for i in _y]
   plt.yticks(_y,_yticks,fontproperties=font)
   ```

   效果图如下：
   ![matplotlib5.png](images/matplotlib5.png)

   **复仇者联盟电影票房案例：**

   ```python
   avenger = [17974.4,50918.4,30033.0,40329.1,52330.2,19833.3,11902.0,24322.6,47521.8,32262.0,22841.9,12938.7,4835.1,3118.1,2570.9,2267.9,1902.8,2548.9,5046.6,3600.8]
   plt.figure(figsize=(15,5))
   plt.plot(avenger,marker="o")
   font.set_size(10)
   plt.xticks(range(20),["第%d天"%x for x in range(1,21)],fontproperties=font)
   plt.xlabel("天数",fontproperties=font)
   plt.ylabel("票房数(万)",fontproperties=font)
   plt.grid()
   ```

   ![复仇者联盟票房折线图.png](images/复仇者联盟票房折线图.png)

###### 3.3. 设置marker：

有时候，我们想要在一些关键点上重点标记出来。那么我们可以通过设置`marker`来实现。示例代码如下：

```python
x = np.linspace(0,20)
y = np.sin(x)
plt.plot(x,y,marker="o")
```

![matplotlib2.png](images/matplotlib2.png)
我们设置了`marker`为`o`，这样就是会在`(x,y)`的坐标点上显示出来，并且显示的是圆点。其中`o`跟之前的线条样式的简写是一样的。另外，还可以通过`markerfacecolor`属性和`markersize`来指定标记点的颜色和大小。示例代码如下：

```python
# 以下设置标记点的颜色为黑色，尺寸为10
plt.plot(x,y,marker="o",markerfacecolor='k',markersize=10)
```

###### 3.4. 设置注释文本：

有时候需要在图形中的某个点标记或者注释一下。那么我们可以使用`plt.annotate(text,xy,xytext,arrowprops={})`来实现，其中`text`是注释的文本，`xy`是需要注释的点的坐标，`xytext`是注释文本的坐标，`arrowprops`是箭头的样式属性。示例代码如下：

```python
ax = plt.subplot(111)

x = np.arange(0.0, 5.0, 0.01)
y = np.cos(2*np.pi*t)
line, = plt.plot(x, y,linewidth=2)

plt.annotate('local max', xy=(2, 1), xytext=(3, 1.5),
arrowprops=dict(facecolor='black',shrink=0.05),
)

plt.ylim(-2, 2)
plt.show()
```

###### 3.5. 设置图形样式：

如果想要调整图片的大小和像素，可以通过`plt.figure(num=None, figsize=None, dpi=None, facecolor=None, edgecolor=None, frameon=True)`来实现。
其中`num`是图的编号，`figsize`的单位是英寸，`dpi`是每英寸的像素点，`facecolor`是图片背景颜色，`edgecolor`是边框颜色，`frameon`代表是否绘制画板。
示例代码如下：

```python
plt.figure(figsize=(20,8),dpi=80)
# 其他的绘制图形的代码
```

我们也可以使用`grid`方法，来显示图片的网格：

```python
plt.plot(x,y,color="r")
plt.grid()
```

![matplotlib9.png](images/matplotlib9.png)

###### 3.6. 保存图片：

可以调用`plt.savefig(path)`来保存当前的图片。示例代码如下：

```python
plt.savefig("./abc.png")
```

---

##### 四、绘制多个图：

绘制多个图有两种形式，第一种形式是在一张图中绘制多跟线条，第二种形式是绘制多个子图形。以下分别进行讲解。

###### 4.1. 绘制多根折线：

绘制多根线条，只要准备好坐标，重新使用`plt.plot`绘制即可。示例代码如下：

```python
from matplotlib import font_manager
x = np.linspace(0,20)
y = np.sin(x)
z = np.cos(x)
font = font_manager.FontProperties(fname="C:\Windows\Fonts\msyh.ttc")
plt.xlabel("x轴",fontproperties=font)
plt.ylabel("y轴",fontproperties=font)
_x = range(0,20,2)
_xticks = ["%s点"%i for i in _x]
plt.xticks(range(0,20,2),_xticks,fontproperties=font,rotation=45)
_y = list(np.range(-1,1,0.25))
_yticks = ["%.2f点"%i for i in _y]
plt.yticks(_y,_yticks,fontproperties=font)
plt.plot(x,y)
plt.plot(x,z)
```

示例图如下：
![matplotlib7.png](images/matplotlib7.png)

###### 4.2. 绘制多个子图：

绘制子图的时候，我们可以使用`plt.subplot`或`plt.subplots`来实现。示例代码如下：

```python
plt.subplot(221)
plt.plot(np.arange(10),c='r')
plt.subplot(222)
plt.plot(np.sin(np.arange(10)),c='b')
plt.subplot(223)
plt.plot(np.cos(np.arange(10)),c='y')
plt.subplot(224)
plt.plot(np.tan(np.arange(10)),c='g')
```

效果图如下：
![subplot1.png](images/subplot1.png)
其中`subplot`中的`211`和`212`分别代表的意思是，第一个数表示这个大图中总共有`2`行，第二个数表示总共有`1`列，然后第三个数表示当前绘制第几个图。

也可以使用`fig,axs=plt.subplots(rows,cols,*args,**kwargs)`来绘制多个图形，返回值是一个元组，其中的`fig`参数是`figure`对象，`axs`是`axes`对象的`array`。示例代码如下：

```python
figure,axes = plt.subplots(2,2)
axes[0,0].plot(np.sin(np.arange(10)),c='r')
axes[0,1].plot(np.cos(np.arange(10)),c='b')
axes[1,0].plot(np.tan(np.arange(10)),c='y')
axes[1,1].plot(np.arange(10),c='g')
```

效果图跟之前使用`plt.subplot`一样。另外使用`subplot`和`subplots`都可以传递`sharex/sharey`参数，这两个参数表示是否需要共享X轴和Y轴。示例代码如下：

```python
figure,axes = plt.subplots(2,2,sharex=True,sharey=True)
axes[0,0].plot(np.sin(np.arange(10)),c='r')
axes[0,1].plot(np.cos(np.arange(10)),c='b')
axes[1,0].plot(np.tan(np.arange(10)),c='y')
axes[1,1].plot(np.arange(10),c='g')
```

![subplot2.png](images/subplot2.png)

###### 4.3. 风格设置：

`matplotlib`图片默认内置了几种风格。我们可以通过`plt.style.available`来查看内置的所有风格:

```python
['bmh',
'classic',
'dark_background',
'fast',
'fivethirtyeight',
'ggplot',
'grayscale',
'seaborn-bright',
'seaborn-colorblind',
'seaborn-dark-palette',
'seaborn-dark',
'seaborn-darkgrid',
'seaborn-deep',
'seaborn-muted',
'seaborn-notebook',
'seaborn-paper',
'seaborn-pastel',
'seaborn-poster',
'seaborn-talk',
'seaborn-ticks',
'seaborn-white',
'seaborn-whitegrid',
'seaborn',
'Solarize_Light2',
'tableau-colorblind10',
'_classic_test']
```

在绘制的，可以使用`plt.style.use`方法来使用不同的风格。示例代码如下：

```python
plt.style.use("dark_background")
```

##### 五、官方文档介绍：

1. `plt.plot`使用详解：<https://matplotlib.org/api/_as_gen/matplotlib.pyplot.plot.html#matplotlib.pyplot.plot>
2. `matplotlib.pyplot`使用详解：<https://matplotlib.org/api/pyplot_summary.html>
3. `matplotlib`内置的样式：<https://tonysyu.github.io/raw_content/matplotlib-style-gallery/gallery.html>



### 第3节　条形图

<!-- 来源：http://www.zlkt.net/book/detail/11/327 -->

#### 条形图

条形图的绘制方式跟折线图非常的类似，只不过是换成了`plt.bar`方法。`plt.bar`方法有以下常用参数：

1. `x`：一个数组或者列表，代表需要绘制的条形图的x轴的坐标点。
2. `height`：一个数组或者列表，代表需要绘制的条形图y轴的坐标点。
3. `width`：每一个条形图的宽度，默认是0.8的宽度。
4. `bottom`：`y`轴的基线，默认是0，也就是距离底部为0.
5. `align`：对齐方式，默认是`center`，也就是跟指定的`x`坐标居中对齐，还有为`edge`，靠边对齐，具体靠右边还是靠左边，看`width`的正负。
6. `color`：条形图的颜色。

返回值为`BarContainer`，是一个存储了条形图的容器，而条形图实际上的类型是`matplotlib.patches.Rectangle`对象。

更多参考：<https://matplotlib.org/api/_as_gen/matplotlib.pyplot.bar.html#matplotlib.pyplot.bar>

##### 一、条形图的绘制：

比如现在有`2019`年贺岁片票房的数据（数据来源：[https://piaofang.maoyan.com/dashboard）：](https://piaofang.maoyan.com/dashboard%EF%BC%89%EF%BC%9A)

```python
#票房单位亿元
movies = {
    "流浪地球":40.78,
    "飞驰人生":15.77,
    "疯狂的外星人":20.83,
    "新喜剧之王":6.10,
    "廉政风云":1.10,
    "神探蒲松龄":1.49,
    "小猪佩奇过大年":1.22,
    "熊出没·原始时代":6.71
}
```

用条形图绘制每部电影及其票房的代码如下：

```python
movies = {
    "流浪地球":40.78,
    "飞驰人生":15.77,
    "疯狂的外星人":20.83,
    "新喜剧之王":6.10,
    "廉政风云":1.10,
    "神探蒲松龄":1.49,
    "小猪佩奇过大年":1.22,
    "熊出没·原始时代":6.71
}
plt.bar(np.arange(len(movies)),list(movies.keys()))
plt.xticks(np.arange(len(movies)),list(movies.keys()),fontproperties=font)
plt.grid()
```

效果图如下：
![电影条形图.png](images/电影条形图.png)

其中`xticks`和`yticks`的用法跟之前的折线图一样。这里新出现的方法是`bar`，`bar`常用的有3个参数，分别是`x`（x轴的坐标点）,`y`（y轴的坐标点）以及`width`（条形的宽度）。

##### 二、横向条形图：

横向条形图需要使用`plt.barh`这个方法跟`bar`非常的类似，只不过把方向进行旋转。参数跟`bar`类似，但也有区别。如下：

1. `y`：数组或列表，代表需要绘制的条形图在`y`轴上的坐标点。
2. `width`：数组或列表，代表需要绘制的条形图在`x`轴上的值（也就是长度）。
3. `height`：条形图的高度，默认是0.8。
4. `left`：条形图的基线，也就是距离y轴的距离。
5. 其他参数跟`bar`一样。

返回值也是`BarContainer`容器对象。

还是以以上数据为例，将电影名和票房反转一下。示例代码如下：

```python
movies = {
    "流浪地球":40.78,
    "飞驰人生":15.77,
    "疯狂的外星人":20.83,
    "新喜剧之王":6.10,
    "廉政风云":1.10,
    "神探蒲松龄":1.49,
    "小猪佩奇过大年":1.22,
    "熊出没·原始时代":6.71
}
plt.barh(np.arange(len(movies)),list(movies.values()))
plt.yticks(np.arange(len(movies)),list(movies.keys()),fontproperties=font)
plt.grid()
```

效果图如下：
![电影横向条形图.png](images/电影横向条形图.png)

##### 三、分组条形图：

现在有一组数据，是2019年春节贺岁片前五天的电影票房记录。
示例代码如下：

```python
movies = {
    "流浪地球":[2.01,4.59,7.99,11.83,16],
    "飞驰人生":[3.19,5.08,6.73,8.10,9.35],
    "疯狂的外星人":[4.07,6.92,9.30,11.29,13.03],
    "新喜剧之王":[2.72,3.79,4.45,4.83,5.11],
    "廉政风云":[0.56,0.74,0.83,0.88,0.92],
    "神探蒲松龄":[0.66,0.95,1.10,1.17,1.23],
    "小猪佩奇过大年":[0.58,0.81,0.94,1.01,1.07],
    "熊出没·原始时代":[1.13,1.96,2.73,3.42,4.05]
}
plt.figure(figsize=(20,8))
width = 0.75
bin_width = width/5
movie_pd = pd.DataFrame(movies)
ind = np.arange(0,len(movies))

# 第一种方案
# first_day = movie_pd.iloc[0]
# plt.bar(ind-bin_width*2,first_day,width=bin_width,label='第一天')

# second_day = movie_pd.iloc[1]
# plt.bar(ind-bin_width,second_day,width=bin_width,label='第二天')

# third_day = movie_pd.iloc[2]
# plt.bar(ind,third_day,width=bin_width,label='第三天')

# four_day = movie_pd.iloc[3]
# plt.bar(ind+bin_width,four_day,width=bin_width,label='第四天')

# five_day = movie_pd.iloc[4]
# plt.bar(ind+bin_width*2,five_day,width=bin_width,label='第五天')

# 第二种方案
for index in movie_pd.index:
    day_tickets = movie_pd.iloc[index]
    xs = ind-(bin_width*(2-index))
    plt.bar(xs,day_tickets,width=bin_width,label="第%d天"%(index+1))
    for ticket,x in zip(day_tickets,xs):
        plt.annotate(ticket,xy=(x,ticket),xytext=(x-0.1,ticket+0.1))

# 设置图例
plt.legend(prop=font)
plt.ylabel("单位：亿",fontproperties=font)
plt.title("春节前5天电影票房记录",fontproperties=font)
# 设置x轴的坐标
plt.xticks(ind,movie_pd.columns,fontproperties=font)
plt.xlim
plt.grid(True)
plt.show()
```

示例图如下：
![分组条形图.png](images/分组条形图.png)

##### 四、堆叠条形图：

堆叠条形图，是将一组相关的条形图堆叠在一起进行比较的条形图。比如以下案例：

```python
menMeans = (20, 35, 30, 35, 27)
womenMeans = (25, 32, 34, 20, 25)
groupNames = ('G1','G2','G3','G4','G5')
xs = np.arange(len(menMeans))
plt.bar(xs,menMeans)
plt.bar(xs,womenMeans,bottom=menMeans)
plt.xticks(xs,groupNames)
plt.show()
```

效果图如下：
![堆叠条形图.png](images/堆叠条形图.png)
在绘制女性得分的条形图的时候，因为要堆叠在男性得分的条形图上，所以使用到了一个`bottom`参数，就是距离`x`轴的距离。通过对贴条形图，我们就可以清楚的知道，哪一个队伍的综合排名是最高的，并且在每个队伍中男女的得分情况。

##### 条形图应用场景：

1. 数量统计。
2. 频率统计。



### 第4节　直方图

<!-- 来源：http://www.zlkt.net/book/detail/11/328 -->

#### 直方图

直方图(Histogram)，又称质量分布图，是一种统计报告图，由一系列高度不等的条纹表示数据分布的情况。一般用横轴表示数据类型，纵轴表示分布情况。
直方图是数值数据分布的精确图形表示。为了构建直方图，第一步是将值的范围分段，即将整个值的范围分成一系列间隔，然后计算每个间隔中有多少值。这些值通常被指定为连续的，不重叠的变量间隔。间隔必须相邻，并且通常是（但不是必须的）相等的大小。

##### 一、绘制直方图：

直方图的绘制方法，使用的是`plt.hist`方法来实现，这个方法的参数以及返回值如下：

\*\* 参数： \*\*

1. `x`：数组或者可以循环的序列。直方图将会从这组数据中进行分组。
2. `bins`：数字或者序列（数组/列表等）。如果是数字，代表的是要分成多少组。如果是序列，那么就会按照序列中指定的值进行分组。比如`[1,2,3,4]`，那么分组的时候会按照三个区间分成3组，分别是`[1,2)/[2,3)/[3,4]`。
3. `range`：元组或者None，如果为元组，那么指定`x`划分区间的最大值和最小值。如果`bins`是一个序列，那么`range`没有有没有设置没有任何影响。
4. `density`：默认是`False`，如果等于`True`，那么将会使用频率分布直方图。每个条形表示的不是个数，而是`频率/组距`（落在各组样本数据的个数称为频数，频数除以样本总个数为频率）。
5. `cumulative`：如果这个和`density`都等于`True`，那么返回值的第一个参数会不断的累加，最终等于`1`。
6. 其他参数：请参考：`https://matplotlib.org/api/_as_gen/matplotlib.pyplot.hist.html`。

**返回值：**

1. `n`：数组。每个区间内值出现的个数，如果`density=True`，那么这个将返回的是`频率/组距`。
2. `bins`：数组。区间的值。
3. `patches`：数组。每根条的对象，类型是`matplotlib.patches.Rectangle`。

##### 二、案例：

比如有一组电影票房时长，想要看下这组票房时长的数据，那么可以通过以下代码来实现：

```python
durations = [131,  98, 125, 131, 124, 139, 131, 117, 128, 108, 135, 138, 131, 102, 107, 114, 119, 128, 121, 142, 127, 130, 124, 101, 110, 116, 117, 110, 128, 128, 115,  99, 136, 126, 134,  95, 138, 117, 111,78, 132, 124, 113, 150, 110, 117,  86,  95, 144, 105, 126, 130,126, 130, 126, 116, 123, 106, 112, 138, 123,  86, 101,  99, 136,123, 117, 119, 105, 137, 123, 128, 125, 104, 109, 134, 125, 127,105, 120, 107, 129, 116, 108, 132, 103, 136, 118, 102, 120, 114,105, 115, 132, 145, 119, 121, 112, 139, 125, 138, 109, 132, 134,156, 106, 117, 127, 144, 139, 139, 119, 140,  83, 110, 102,123,107, 143, 115, 136, 118, 139, 123, 112, 118, 125, 109, 119, 133,112, 114, 122, 109, 106, 123, 116, 131, 127, 115, 118, 112, 135,115, 146, 137, 116, 103, 144,  83, 123, 111, 110, 111, 100, 154,136, 100, 118, 119, 133, 134, 106, 129, 126, 110, 111, 109, 141,120, 117, 106, 149, 122, 122, 110, 118, 127, 121, 114, 125, 126,114, 140, 103, 130, 141, 117, 106, 114, 121, 114, 133, 137,  92,121, 112, 146,  97, 137, 105,  98, 117, 112,  81,  97, 139, 113,134, 106, 144, 110, 137, 137, 111, 104, 117, 100, 111, 101, 110,105, 129, 137, 112, 120, 113, 133, 112,  83,  94, 146, 133, 101,131, 116, 111,  84, 137, 115, 122, 106, 144, 109, 123, 116, 111,111, 133, 150]
plt.figure(figsize=(15,5))
nums,bins,patches = plt.hist(durations,bins=20,edgecolor='k')
plt.xticks(bins,bins)
for num,bin in zip(nums,bins):
    plt.annotate(num,xy=(bin,num),xytext=(bin+1.5,num+0.5))
plt.show()
```

效果图如下：
![电影时间直方图.png](images/电影时间直方图-080726.png)

另外，也可以通过`density=True`，来实现频率分布直方图。示例代码如下：

```python
nums,bins,patches = plt.hist(durations,bins=20,edgecolor='k',density=True)
plt.xticks(bins,bins)
for num,bin in zip(nums,bins):
    plt.annotate("%.4f"%num,xy=(bin,num),xytext=(bin+0.2,num+0.0005))
```

![电影频率直方图.png](images/电影频率直方图.png)

而如果想要让`nums`的总和为`1`，那么就需要设置`cumulative=True`参数，示例代码如下：

```python
nums,bins,patches = plt.hist(durations,bins=20,edgecolor='k',density=True,cumulative=True)
plt.xticks(bins,bins)
for num,bin in zip(nums,bins):
    plt.annotate("%.4f"%num,xy=(bin,num),xytext=(bin+0.2,num+0.0005))
```

##### 三、直方图的应用场景：

1. 显示各组数据数量分布的情况。
2. 用于观察异常或孤立数据。
3. 抽取的样本数量过小，将会产生较大误差，可信度低，也就失去了统计的意义。因此，样本数不应少于50个。



### 第5节　散点图

<!-- 来源：http://www.zlkt.net/book/detail/11/329 -->

#### 散点图

散点图也叫 X-Y 图，它将所有的数据以点的形式展现在直角坐标系上，以显示变量之间的相互影响程度，点的位置由变量的数值决定。

通过观察散点图上数据点的分布情况，我们可以推断出变量间的相关性。如果变量之间不存在相互关系，那么在散点图上就会表现为随机分布的离散的点，如果存在某种相关性，那么大部分的数据点就会相对密集并以某种趋势呈现。数据的相关关系主要分为：正相关（两个变量值同时增长）、负相关（一个变量值增加另一个变量值下降）、不相关、线性相关、指数相关等，表现在散点图上的大致分布如下图所示。那些离点集群较远的点我们称为离群点或者异常点。
![散点图相关性.png](images/散点图相关性.png)

示例图如下：
![散点图示例.jpg](images/散点图示例-6a507f.jpg)

##### 一、绘制散点图：

散点图的绘制，使用的是`plt.scatter`方法，这个方法有以下参数：

1. `x,y`：分别是x轴和y轴的数据集。两者的数据长度必须一致。
2. `s`：点的尺寸。如果是一个具体的数字，那么散点图的所有点都是一样大小，如果是一个序列，那么这个序列的长度应该和x轴数据量一致，序列中的每个元素代表每个点的尺寸。
3. `c`：点的颜色。可以为具体的颜色，也可以为一个序列或者是一个`cmap`对象。
4. `marker`：标记点，默认是圆点，也可以换成其他的。
5. 其他参数：`https://matplotlib.org/api/_as_gen/matplotlib.pyplot.scatter.html#matplotlib.pyplot.scatter`。

比如有一组运动员身高和体重以及年龄的数据，那么可以通过以下代码来绘制散点图：

```python
male_athletes = athletes[athletes['Sex'] == 'M']
female_athletes = athletes[athletes['Sex'] == 'F']
male_mean_height = male_athletes['Height'].mean()
female_mean_height = female_athletes['Height'].mean()
male_mean_weight = male_athletes['Weight'].mean()
female_mean_weight = female_athletes['Weight'].mean()

plt.figure(figsize=(10,5))
plt.scatter(male_athletes['Height'],male_athletes['Weight'],s=male_athletes['Age'],marker='^',color='g',label='男性',alpha=0.5)
plt.scatter(female_athletes['Height'],female_athletes['Weight'],color='r',alpha=0.5,s=female_athletes['Age'],label='女性')
plt.axvline(male_mean_height,color="g",linewidth=1)
plt.axhline(male_mean_weight,color="g",linewidth=1)
plt.axvline(female_mean_height,color="r",linewidth=1)
plt.axhline(female_mean_weight,color="r",linewidth=1)
plt.xticks(np.arange(140,220,5))
plt.yticks(np.arange(30,150,10))
plt.legend(prop=font)
plt.xlabel("身高（cm）",fontproperties=font)
plt.ylabel("体重（kg）",fontproperties=font)
plt.title("运动员身高和体重散点图",fontproperties=font)
plt.grid()
plt.show()
```

效果图如下：
![运动员散点图.png](images/运动员散点图.png)

##### 二、绘制回归曲线：

有一组数据后，我们可以对这组数据进行回归分析，回归分析可以帮助我们了解这组数据的大体走向。回归分析按照涉及的变量的多少，分为一元回归和多元回归分析；按照自变量的多少，可分为简单回归分析和多重回归分析；按照自变量和因变量之间的关系类型，可分为线性回归分析和非线性回归分析。如果在回归分析中，只包括一个自变量和一个因变量，且二者的关系可用一条直线近似表示，这种回归分析称为一元线性回归分析。如果回归分析中包括两个或两个以上的自变量，且自变量之间存在线性相关，则称为多重线性回归分析。

| 自变量数量 | 是否线性 | 回归类型 |
| --- | --- | --- |
| 1个 | 是 | 一元线性回归 |
| 多个 | 是 | 多元线性回归 |
| 1个 | 否 | 一元非线性回归 |
| 多个 | 否 | 多元非线性回归 |

通过以上运动员散点图的分析，我们总体上可以看出来是满足线性回归的，因此可以在图上绘制一个线性回归的线条。想要绘制线性回归的线条，需要先按照之前的数据计算出线性方程，假如`x`是自变量，`y`是因变量，那么线性回归的方程可以用以下几个来表示：

```
y = 截距+斜率*x+误差
```

只要把这个方程计算出来了，那么后续我们就可以根据`x`的值，大概的估计出`y`的取值范围，也就是预测。如果我们针对以上运动员的身高和体重的关系，只要有身高，那么就可以大概的估计出体重的值。回归方程的绘制我们需要借助`scikit-learn`库，这个库是专门做机器学习用的，我们需要使用里面的线性回归类`sklearn.liear_regression.LinearRegression`。示例代码如下：

```python
from sklearn.linear_model import LinearRegression
male_athletes = athletes[athletes['Sex'] == 'M'].dropna()
female_athletes = athletes[athletes['Sex'] == 'F'].dropna()
xtrain = male_athletes['Height']
ytrain = male_athletes['Weight']
# 生成线性回归对象
model = LinearRegression()
# 喂训练数据进去，但是需要把因变量转换成1列多行的数据
model.fit(xtrain[:,np.newaxis],ytrain)
# 打印斜率
print(model.coef_)
# 打印截距
print(model.intercept_)
line_xticks = xtrain
# 根据回归方程计算出的y轴坐标
line_yticks = model.predict(xtrain[:,np.newaxis])
```

效果图如下：
![有回归线的运动员散点图.png](images/有回归线的运动员散点图.png)



### 第6节　饼图

<!-- 来源：http://www.zlkt.net/book/detail/11/330 -->

#### 饼图

饼图是一个划分为几个扇形的圆形统计图表，用于描述量、频率或百分比之间的相对关系的。
在`matplotlib`中，可以通过`plt.pie`来实现，其中的参数如下：

1. `x`：饼图的比例序列。
2. `labels`：饼图上每个分块的名称文字。
3. `explode`：设置某几个分块是否要分离饼图。
4. `autopct`：设置比例文字的展示方式。比如保留几个小数等。
5. `shadow`：是否显示阴影。
6. `textprops`：文本的属性（颜色，大小等）。
7. 其他参数：https://matplotlib.org/api/\_as\_gen/matplotlib.pyplot.pie.html#matplotlib.pyplot.pie

**返回值：**

1. `patches`：饼图上每个分块的对象。
2. `texts`：分块的名字文本对象。
3. `autotexts`：分块的比例文字对象。

假如现在我们有一组数据，用来记录各个操作系统的市场份额的。那么用饼状图表示如下：

```python
oses = {
'windows7':60.86,
'windows10': 18.46,
'windows8': 3.61,
'windows xp': 10.3,
'mac os': 6.78,
'其他': 1.12
}
names = oses.keys()
percents = oses.values()
patches,texts,autotexts = plt.pie(percents,labels=names,autopct="%.2f%%",explode=(0,0.05,0,0,0,0))
for text in texts+autotexts:
    plt.setp(text,fontproperties=font)
    text.set_fontsize(10)
for text in autotexts:
    text.set_color("white")
```

效果图如下：
![matplotlib12.png](images/matplotlib12.png)



### 第7节　箱线图

<!-- 来源：http://www.zlkt.net/book/detail/11/331 -->

#### 箱线图

##### 一、箱线图介绍：

箱线图（Box-plot）又称为盒须图、盒式图或箱型图，是一种用作显示一组数据分散情况资料的统计图。因形状如箱子而得名。在各种领域也经常被使用，它主要用于反映原始数据分布的特征，还可以进行多组数据分布特征的比较。箱线图的绘制方法是：先找出一组数据的**上限值、下限值、中位数（Q2）和下四分位数（Q1）以及上四分位数（Q3）**；然后，连接两个四分位数画出箱子；再将最大值和最小值与箱子相连接，中位数在箱子中间。
![箱线图解释图.png](images/箱线图解释图.png)
![箱线图案例.jpg](images/箱线图案例-9c82f4.jpg)

> 中位数：把数据按照从小到大的顺序排序，然后最中间的那个值为中位数，如果数据的个数为偶数，那么就是最中间的两个数的平均数为中位数。
> 上下四分位数：同样把数据排好序后，把数据等分为4份。出现在`25%`位置的叫做下四分位数，出现在`75%`位置上的数叫做上四分位数。但是四分位数位置的确定方法不是固定的，有几种算法，每种方法得到的结果会有一定差异，但差异不会很大。
>
> 上下限的计算规则是：
> IQR=Q3-Q1
> 上限=Q3+1.5IQR
> 下限=Q1-1.5IQR

##### 二、使用matplotlib绘制箱线图：

在`matplotlib`中有`plt.boxplot`来绘制箱线图，这个方法的相关参数如下：

1. `x`：需要绘制的箱线图的数据。
2. `notch`：是否展示置信区间，默认是`False`。如果设置为`True`，那么就会在盒子上展示一个缺口。
3. `sym`：代表异常点的符号表示，默认是小圆点。
4. `vert`：是否是垂直的，默认是`True`，如果设置为`False`那么将水平方向展示。
5. `whis`：上下限的系数，默认是`1.5`，也就是上限是`Q3+1.5IQR`，可以改成其他的。也可以为一个序列，如果是序列，那么序列中的两个值分别代表的就是下限和上限的值，而不是再需要通过`IQR`来计算。
6. `positions`：设置每个盒子的位置。
7. `widths`：设置每个盒子的宽度。
8. `labels`：每个盒子的`label`。
9. `meanline`和`showmeans`：如果这两个都为`True`，那么将会绘制平均值的的线条。

示例代码如下：

```python
data = np.random.rand(100)*100
# 添加两个异常值
data = np.append(data,np.array([-100,100]))
plt.boxplot(data,meanline=True,showmeans=True)
```

效果图如下：
![单一箱型图效果图.png](images/单一箱型图效果图.png)

如果有多组数据绘制箱型图，才能更好的提现出箱型图的优势。
假如我们想要获取奥林匹克运动会上不同国家运动员的身高情况，那么可以把每个国家的运动员身高数据绘制成一个箱线图，然后进行对比。示例代码如下：

```python
athletes = pd.read_csv("athlete_events.csv")
# (中国CHN，日本JPN，韩国KOR)，（埃塞俄比亚ETH，肯尼亚KEN，尼日利亚NIG），(美国USA，加拿大CAN，巴西BRA)，(英国GBR，法国FRA，意大利ITA)
countries = {
    'CHN':'中国',
    'JPN':"日本",
    'KOR':'韩国',
    'USA':"美国",
    'CAN':"加拿大",
    'BRA':"巴西",
    'GBR':"英国",
    'FRA':"法国",
    'ITA':"意大利",
    'ETH':"埃塞俄比亚",
    'KEN':"肯尼亚",
    'NIG':"尼日利亚",
}
dfs = []
for code in countries.keys():
    df = athletes[(athletes['NOC'] == code)&(athletes['Age']>18)]['Height'].dropna()
    dfs.append(df)
font = font_manager.FontProperties(fname=r"C:\\Windows\\Fonts\\msyh.ttc",size=14)
plt.figure(figsize=(20,5))
plt.boxplot(dfs,showmeans=True,meanline=True,labels=countries.values())
plt.xticks(range(1,13),countries.values(),fontproperties=font)
plt.ylabel("身高(cm)",fontproperties=font)
plt.title("奥林匹克运动员身高箱线图",fontproperties=font)
```

效果图如下：
![奥林匹克运动员身高箱线图.png](images/奥林匹克运动员身高箱线图.png)

##### 三、箱线图的应用场景：

1. 直观明了地识别数据中的异常值。
2. 利用箱线图判断数据的偏态。
3. 利用箱线图比较几批数据的形状。
4. 箱线图适合比较多组数据，如果知识要看一组数据的分布情况，建议使用直方图。



### 第8节　雷达图

<!-- 来源：http://www.zlkt.net/book/detail/11/332 -->

#### 雷达图

雷达图（Radar Chart）又被叫做蜘蛛网图，适用于显示三个或更多的维度的变量的强弱情况。比如英雄联盟中某个影响的属性（法术伤害，物理防御等），或者是某个企业在哪些业务方面的投入等，都可以用雷达图方便的表示。

##### 一、使用plt.polar绘制雷达图：

在`matplotlib.pyplot`中，可以通过`plt.polar`来绘制雷达图，这个方法的参数跟`plt.plot`非常的类似，只不过是`x`轴的坐标点应该为弧度（2\*PI=360°）。示例代码如下：

```python
properties = ['输出','KDA','发育','团战','生存']
values = [40,91,44,90,95,40]
theta = np.linspace(0,np.pi*2,6)
plt.polar(theta,values)
plt.xticks(theta,properties,fontproperties=font)
plt.fill(theta,values)
```

效果图如下：
![雷达图效果图.png](images/雷达图效果图.png)

其中有几点需要注意：

1. 因为`polar`并不会完成线条的闭合绘制，所以我们在绘制的时候需要在`theta`中和`values`中在最后多重复添加第0个位置的值，然后在绘制的时候就可以和第1个点进行闭合了。
2. `polar`只是绘制线条，所以如果想要把里面进行颜色填充，那么需要调用`fill`函数来实现。
3. `polar`默认的圆圈的坐标是角度，如果我们想要改成文字显示，那么可以通过`xticks`来设置。

##### 二、使用子图绘制雷达图：

在多子图中，绘图对象不再是`pyplot`而是`Axes`，而`Axes`及其子类绘制雷达图则是通过将直角坐标转换成极坐标，然后再绘制折线图。示例代码如下：

1. 使用`plt.subplot`绘制的子图：

   ```python
   properties = ['输出','KDA','发育','团战','生存']
    values = [40,91,44,90,95,40]
    theta = np.linspace(0,np.pi*2,6)
    # 生成一个子图，并且指定子图的类型为polar
    axes = plt.subplot(111,projection="polar")
    axes.plot(theta,values)
    axes.fill(theta,values)
   ```
2. 使用`plt.subplots`绘制的子图：

   ```python
   properties = ['输出','KDA','发育','团战','生存']
    values = [40,91,44,90,95,40]
    theta = np.linspace(0,np.pi*2,6)
    figure,axes = plt.subplots(1,1,subplot_kw={"projection":"polar"})
    axes.plot(theta,values)
   ```
3. 使用`fig.add_subplot`绘制的子图：

   ```python
   properties = ['输出','KDA','发育','团战','生存']
    values = [40,91,44,90,95,40]
    theta = np.linspace(0,np.pi*2,6)
    fig = plt.figure(figsize=(10,10))
    axes = fig.add_subplot(111,polar=True)
    axes.plot(theta,values)
   ```



### 第9节　matplotlib绘图分析

<!-- 来源：http://www.zlkt.net/book/detail/11/333 -->

#### matplotlib绘图分析

![matplotlib图结构.png](images/matplotlib图结构.png)

解释：

1. `Figure`：图形绘制的画板，他就相当于一个黑板，所有的图都是绘制在`Figure`上面。
2. `Axes`：每个图都是`Axes`对象。一个`Figure`上可以有多个`Axes`对象。
3. `Axis`：`x`轴、`y`轴的对象。
4. `Tick`：`x`轴和`y`轴上的刻度对象。每一个刻度都是一个`Tick`对象。
5. `TickLabel`：每个刻度上都要显示文字，这个文字的显示就是在`TickLabel`上。
6. `AxisLabel`：`x`轴和`y`轴的名称的文字显示。
7. `Legend`：图例对象。
8. `Title`：`Axes`图的标题对象。
9. `Line2D`：绘制在`Axes`上的线条对象，比如折线图等。
10. `Reactangle`：绘制在`Axes`上的矩形对象，比如条形图等。
11. `Marker`：标记点，比如绘制散点图上的每个点就是这个对象。
12. `Artist`：只要是绘制在`Figure`上的元素（包括Figure），都是`Artist`的子类。

##### 一、Figure容器：

`Figure`容器是最顶层的容器，他几乎包含了这个图的所有对象。通过`add_subplot`和`add_axes`方法可以添加`Axes`对象，这两个方法添加的都是`Axes`及其子类的对象。添加完成后是存储在`figure.axes`中。示例代码如下：

```python
In [156]: fig = plt.figure()
In [157]: ax1 = fig.add_subplot(211)
In [158]: ax2 = fig.add_axes([0.1, 0.1, 0.7, 0.3])
In [159]: ax1
Out[159]: <matplotlib.axes.Subplot instance at 0xd54b26c>
In [160]: print(fig.axes)
[<matplotlib.axes.Subplot instance at 0xd54b26c>, <matplotlib.axes.Axes instance at 0xd3f0b2c>]
```

###### 1.1. 添加Axes对象：

`Figure`只是一个黑板，如果想要绘图，需要先添加`Axes`。添加`Axes`可以通过`add_axes`和`add_subplot`来实现。示例代码如下：

```python
# 创建一个figure对象
fig = plt.figure()
# 添加一个Axes
ax1 = fig.add_subplot(211)
# 添加一个Axes，其中参数是left,bottom,width,height
ax2 = fig.add_axes([0.1,0.1,0.8,0.3])
```

###### 1.2. 操作当前Axes对象：

可以通过`figure.gca`以及`figure.sca`来设置和获取当前的`axes`对象。示例代码如下：

```python
fig = plt.figure()
ax1 = fig.add_subplot(211)
ax2 = fig.add_axes([0,0,1,0.3])
print(fig.gca())
print(fig.sca(ax1))

>> Axes(0,0;1x0.3)
>> AxesSubplot(0.125,0.536818;0.775x0.343182)
```

###### 1.3. 删除Axes对象：

`Figure`上的所有`Axes`对象都是保存在`fig.axes`中，但是如果想要删除某个`Axes`对象，那么必须通过`delaxes`来实现：

```python
fig = plt.figure()
ax1 = fig.add_subplot(211)
ax2 = fig.add_axes([0,0,1,0.3])
fig.delaxes(ax1)
print(fig.axes)
```

###### 1.4. 获取所有的axes：

```python
for ax in fig.axes:
    ax.grid(True) # 设置打开网格
```

###### 1.5. `Figure`的属性有如下：

![figure属性.png](images/figure属性.png)

`Figure`**类定义介绍：**<https://matplotlib.org/api/_as_gen/matplotlib.figure.Figure.html#matplotlib.figure.Figure>

##### 二、Axes容器：

`Axes`容器是用来创建具体的图形的。比如画曲线，柱状图，都是画在上面。所以之前我们学的使用`plt.xx`绘制各种图形（比如条形图，直方图，散点图等）都是对`Axes`的封装。比如`plt.plot`对应的是`axes.plot`，比如`plt.hist`对应的是`axes.hist`。针对图的所有操作，都可以在`Axes`上找到对应的`API`。另外后面要讲到的`Axis`容器，是轴的对象，也是绑定在`Axes`上面。
**Axes的类定义介绍：**<https://matplotlib.org/api/axes_api.html#matplotlib.axes.Axes>

###### 2.1. 设置x和y轴的最大值和最小值：

设置完刻度后，我们还可以设置x轴和y轴的最大值和最小值。可以通过`set_xlim/set_ylim`来实现：

```python
fig = plt.figure()
axes = fig.add_subplot(111)
axes.plot(np.random.randn(10))

# 设置x轴的最大值和最小值
axes.set_xlim(-2,12)

# 设置y轴的最大值和最小值
axes.set_ylim(-3,3)
```

###### 2.2. 添加文本：

之前添加文本我们用的是`annotate`，但是如果不是需要做注释，其实还有另外一种更加简单的方式，那就是使用`text`方法：

```python
data = np.random.randn(10)
fig = plt.figure()
axes = fig.add_subplot(111)
axes.plot(data)
# 添加文本，比annotate更加方便
axes.text(0,0,"hello")
```

###### 2.3. 绘制双`Y`轴：

```python
fig = plt.figure()
ax1 = fig.add_subplot(211)
ax1.bar(np.arange(0,10,2),np.random.rand(5))
ax1.set_yticks(np.arange(0,1,0.25))
ax2 = ax1.twinx() #克隆一个共享x轴的axes对象
ax2.plot(np.random.randn(10),c="b")
plt.show()
```

效果图如下：
![](http://www.zlkt.net/assets/chapter04/%E5%8F%8CY%E8%BD%B4%E6%95%88%E6%9E%9C%E5%9B%BE.png)

##### 三、Axis容器：

`Axis`代表的是`x`轴或者`y`轴的对象。包含`Tick`（刻度）对象，`TickLabel`刻度文本对象，以及`AxisLabel`坐标轴文本对象。`axis`对象有一些方法可以操作刻度和文本等。

###### 3.1. 设置x轴和y轴label的位置：

```python
fig = plt.figure()
axes = fig.add_subplot(111)
axes.plot(np.random.randn(10))
axes.set_xlabel("x coordate")
# 设置x轴label的位置为(0.-0.1)
axes.xaxis.set_label_coords(0,-0.1)
```

###### 3.2. 设置刻度上的刻度格式：

```python
import matplotlib.ticker as ticker
fig = plt.figure()
axes = fig.add_subplot(111)
axes.plot(np.random.randn(10))
axes.set_xlabel("x coordate")
# 创建格式化对象
formatter = ticker.FormatStrFormatter('%.2f')
# 设置格式化对象
axes.yaxis.set_major_formatter(formatter)
```

###### 3.3. 设置轴的属性：

```python
fig = plt.figure()

ax1 = fig.add_axes([0.1, 0.3, 0.4, 0.4])
ax1.set_facecolor('lightslategray')

# 设置刻度上文本的属性
for label in ax1.xaxis.get_ticklabels():
    # label是一个Label对象
    label.set_color('red')
    label.set_rotation(45)
    label.set_fontsize(16)

# 设置刻度上线条的属性
for line in ax1.yaxis.get_ticklines():
    # line是一个Line2D对象
    line.set_color('green')
    line.set_markersize(25)
    line.set_markeredgewidth(3)

plt.show()
```

![axis容器.png](images/axis容器.png)

##### 四、Tick容器：

`Tick`是用来做刻度的，包括刻度和网格对象。其中可操作的属性如下：
![Tick容器.png](images/Tick容器.png)

示例代码如下：

```python
import matplotlib.ticker as ticker

# Fixing random state for reproducibility
np.random.seed(19680801)

fig, ax = plt.subplots()
ax.plot(100*np.random.rand(20))

formatter = ticker.FormatStrFormatter('$%.2f')
ax.yaxis.set_major_formatter(formatter)

for tick in ax.yaxis.get_major_ticks():
    tick.label1On = False
    tick.label2On = True
    tick.label2.set_color('green')

plt.show()
```

![Tick案例图.png](images/Tick案例图.png)

更多请参考：
[https://matplotlib.org/api/axis\_api.html#matplotlib.axis.Axis](https://matplotlib.org/api/axis_api.html#matplotlib.axis.Tick)

##### 五、参考：

<https://matplotlib.org/tutorials/intermediate/artists.html#sphx-glr-tutorials-intermediate-artists-py>



### 第10节　多图布局

<!-- 来源：http://www.zlkt.net/book/detail/11/334 -->

#### 多图布局

##### 一、解决元素重叠的问题：

在一个`Figure`上面，可能存在多个`Axes`对象，如果`Figure`比较小，那么有可能会造成一些图形元素重叠，这时候我们就可以通过`fig.tight_layout`或者是`fig.subplots_adjust`方法来帮我们调整。假如现在没有经过调整，那么以下代码的效果图如下：

```python
import matplotlib.pyplot as plt
import numpy as np

def example_plot(ax, fontsize=12):
    ax.plot([1, 2])
    ax.set_xlabel('x-label', fontsize=fontsize)
    ax.set_ylabel('y-label', fontsize=fontsize)
    ax.set_title('Title', fontsize=fontsize)

fig,axes = plt.subplots(2,2)
fig.set_facecolor("y")
example_plot(axes[0,0])
example_plot(axes[0,1])
example_plot(axes[1,0])
example_plot(axes[1,1])
```

效果图如下：
![没有tight_layout效果图.png](images/没有tight_layout效果图.png)

为了避免多个图重叠，可以使用`plt.tight_layout`来实现：

```python
# 之前的代码...
plt.tight_layout()
```

效果图如下：
![有tight_layout效果图.png](images/有tight_layout效果图.png)

其中`tight_layout`还有两个参数可以使用，分别是`w_pad`和`h_pad`，这两个参数分别表示的意思是在水平方向的图之间的间距，以及在垂直方向这些图的间距。

另外也可以通过`fig.subplots_adjust(left=None,bottom=None,right=None,top=None,wspace=None,hspace=None)`来实现，效果如下：

```python
# 之前的代码...
fig.subplots_adjust(0,0,1,1,hspace=0.5,wspace=0.5)
```

效果图如下：
![自定义布局4.png](images/自定义布局4.png)

##### 二、自定义布局方式：

如果布局不是固定的几宫格的方式，而是某个图占据了多行或者多列，那么就需要采用一些手段来实现。如果不是很复杂，那么直接可以通过`subplot`等方法来实现。示例代码如下：

```python
ax1 = plt.subplot(221)
ax2 = plt.subplot(223)
ax3 = plt.subplot(122)
```

效果图如下：
![自定义布局1.png](images/自定义布局1.png)

但是如果实现的布局比较复杂，那么就需要采用`GridSpec`对象来实现。示例代码如下：

```python
fig = plt.figure()
# 创建3行3列的GridSpec对象
gs = fig.add_gridspec(3,3)
ax1 = fig.add_subplot(gs[0,0:3])
ax1.set_title("[0,0:3]")
ax2 = fig.add_subplot(gs[1,0:2])
ax2.set_title("[1,0:2]")
ax3 = fig.add_subplot(gs[1:3,2])
ax3.set_title("[1:3,2]")
ax4 = fig.add_subplot(gs[2,0])
ax4.set_title("[2,0]")
ax5 = fig.add_subplot(gs[2,1])
ax5.set_title("[2,1]")
plt.tight_layout()
```

效果图如下：
![自定义布局2.png](images/自定义布局2.png)

也可以设置宽高比例。示例代码如下：

```python
# 设置宽度比例为1:2:1
widths = (1,2,1)
# 设置高度比例为2:2:1
heights = (2,2,1)
fig = plt.figure()
# 创建GridSpec对象的时候指定宽高的比
gs = fig.add_gridspec(3,3,width_ratios=widths,height_ratios=heights)
for row in range(0,3):
    for col in range(0,3):
        fig.add_subplot(gs[row,col])
plt.tight_layout()
```

效果图如下：
![自定义布局3.png](images/自定义布局3.png)

##### 三、手动设置位置：

通过`fig.add_axes`的方式添加`Axes`对象，可以直接指定位置。也可以在添加完成后，通过`axes.set_position`的方式设置位置。示例代码如下：

```python
# add_axes的方式
fig = plt.figure()
fig.add_subplot(111)
fig.add_axes([0.2,0.2,0.4,0.4])

# 设置position的方式
fig,axes = plt.subplots(1,2)
axes[1].set_position([0.2,0.2,0.4,0.4])
```

##### 四、散点图和直方图合并实战：

```python
fig = plt.figure(figsize=(8,8))
widths = (2,0.5)
heights = (0.5,2)
gs = fig.add_gridspec(2,2,width_ratios=widths,height_ratios=heights)
# 顶部的直方图
ax1 = fig.add_subplot(gs[0,0])
ax1.hist(male_athletes['Height'],bins=20)
for tick in ax1.xaxis.get_major_ticks():
    tick.label1On = False

# 中间的散点图
ax2 = fig.add_subplot(gs[1,0])
ax2.scatter('Height','Weight',data=male_athletes)

# 右边的直方图
ax3 = fig.add_subplot(gs[1,1])
ax3.hist(male_athletes['Weight'],bins=20,orientation='horizontal')
for tick in ax3.yaxis.get_major_ticks():
    tick.label1On = False
fig.tight_layout(h_pad=0,w_pad=0)
```

效果图如下：
![自定义布局案例.png](images/自定义布局案例.png)



### 第11节　matplotlib配置

<!-- 来源：http://www.zlkt.net/book/detail/11/335 -->

#### matplotlib配置

##### 一、修改默认的配置：

修改默认的配置可以通过`matplotlib.rcParams`来设置，比如修改字体，修改线条大小和宽度等。示例代码如下：

```python
import matplotlib.pyplot as plt
# 设置字体为仿宋
plt.rcParams['font.sans-serif'] = ['FangSong']
# 设置字体大小为20
plt.rcParams['font.size'] = 20
# 设置线条宽度
plt.rcParams['lines.linewidth'] = 2
# 设置线条颜色
plt.rcParams['axes.prop_cycle'] = plt.cycler('color', ['r', 'y'])
```

其中`rcParams`中可以设置的属性为如下：

在`Windows`上如果想要显示中文，那么可以通过设置`font.sans-serif`来设置，示例代码如下：

```python
plt.rcParams['font.sans-serif'] = ['FangSong']
```

这个属性可以设置以下字体都可以显示中文：

| 字体名 | 英文名称 |
| --- | --- |
| 黑体 | SimHei |
| 仿宋 | FangSong |
| 楷体 | KaiTi |
| 宋体 | SimSun |
| 隶书 | LiSu |
| 幼圆 | YouYuan |
| 华文细黑 | STXihei |
| 华文楷体 | STKaiti |
| 华文宋体 | STSong |
| 华文中宋 | STZhongsong |
| 华文仿宋 | STFangsong |
| 方正舒体 | FZShuTi |
| 方正姚体 | FZYaoti |
| 华文彩云 | STCaiyun |
| 华文琥珀 | STHupo |
| 华文隶书 | STLiti |
| 华文行楷 | STXingkai |
| 华文新魏 | STXinwei |

`Mac`和`Linux`支持的字体可能会不同，如果不行，可以使用`matplotlib.font_manager`来指定具体的字体。

##### 二、自定义配置文件：

有时候我们可能需要设置一大堆参数，并且这个配置在后面很多项目中可能都会用到，那么这时候我们就可以把这些配置信息放到文件中（可配置项见下），文件的命名规则为`[名称].mplstyle`，然后把这个文件放到`matplotlib.get_configdir()/stylelib`的目录中，在写代码的时候根据名称加载这个配置文件，示例代码如下：

```python
plt.style.use("名称")
```

##### 三、可配置项：

更多可配置项请参考：https://raw.githubusercontent.com/matplotlib/matplotlib/master/matplotlibrc.template



### 第12节　matplotlib作业

<!-- 来源：http://www.zlkt.net/book/detail/11/336 -->

#### matplotlib作业

##### 一、折线图作业要求：

1. 以下是长沙某一个月的天气数据，按照时间的顺序绘制成折线图，其中数据`highest`是最高温度，`lowest`是最低温度。最高温度线条用红色，最低温度线条用蓝色。
2. 具体的坐标点，用圆点marker表示。
3. 把x轴的时间刻度按照`1-31`标记出来，并且标记x轴和y轴的标题。
4. 图的标题是“长沙5月份气温走势”。

**数据：**

```python
highest = [26,21,26,26,22,20,17,19,22,28,30,28,24,28,25,26,25,26,25,23,24,30,32,31,30,27,26,27,29,25,25]
lowest =  [17,13,17,18,18,17,14,15,16,18,19,20,18,18,20,20,20,20,20,16,17,19,21,24,24,23,20,18,19,18,19]
```

**效果图参考：**
![1折线图作业效果图.png](images/1折线图作业效果图.png)

##### 二、条形图作业要求：

1. 以下数据是三类学校（普通本科、中等职业教育、普通高中）在2014-2018（包含2018）的报名人数，用DataFrame构建。
2. 把年份当做x轴，报名人数当做y轴的值。
3. 绘制分组条形图，同一个年份的放在一个组。
4. 图例横向排列（提示：用legend的ncol参数，ncol表示的是把图例分成多少列显示）。
5. 把报名人数在图上绘制出来。

**数据：**

```python
data = {
    "普通本科":[721,738,749,761,791],
    "中等职业教育": [620,601,593,582,557],
    "普通高中": [797,797,803,800,793]
}
```

**效果图参考：**
![2条形图作业效果图.png](images/2条形图作业效果图.png)

##### 三、直方图作业要求：

1. 用pandas从scores.csv读取出来，形成一个DataFrame对象。
2. 绘制化学成绩的直方图（chem列）。
3. 标记x轴的坐标。
4. 标记每个条形上的具体数值。

**数据：**
在`matplotlib代码->作业参考`文件夹的`scores.csv`文件中。

**效果图参考：**
![3直方图作业效果图.png](images/3直方图作业效果图.png)

##### 四、散点图作业要求：

1. 把guazi\_bj（北京）、guazi\_gz（广州）、guazi\_sh（上海）、guazi\_sz（深圳）二手车的数据归类在一个DataFrame中。
2. 新增车辆使用年份（use\_year）与保值率（hedge\_rate）两个字段。其中使用年份的计算是把当前的时间减去购买的时间，然后再转换成年；保值率的计算是将二手车的价格/新车的价格。
3. 把二手车使用年份与保值率（二手车价/新车价格）绘制成散点图，观察他们的分布情况。
4. 把二手车的行驶距离与保值率（二手车价/新车价格）绘制成散点图，观察他们的分布情况。
   ![散点图作业效果1.png](images/散点图作业效果1.png)
   ![散点图作业效果2.png](images/散点图作业效果2.png)

##### 五、饼图作业要求：

1. 把以下数据绘制成饼图。
2. 把Chrome浏览器的模块分割开0.05。
3. 设置阴影。
4. 把百分数的颜色设置成白色，把浏览器的名字颜色设置成黑色。
5. 把Edge和Safari浏览器的比例文字字体大小调成10，其他的12。

效果如下：
![饼图作业效果图.png](images/饼图作业效果图.png)

##### 六、箱线图作业：

1. 读取scores.csv文件。
2. 把所有科目的成绩都在一张图上绘制箱线图。
3. 观察这个图，你能发现什么信息。

效果图：
![箱线图作业效果图.png](images/箱线图作业效果图.png)

##### 七、雷达图作业：

1. 读取scores.csv文件成DataFrame对象。
2. 计算每个科目的平均成绩。
3. 将每个科目的平均成绩绘制成雷达图。

效果图如下：
![雷达图作业效果图.png](images/雷达图作业效果图.png)



## 第5章　Seaborn库


### 第1节　Seaborn库介绍

<!-- 来源：http://www.zlkt.net/book/detail/11/337 -->

#### seaborn库：

`Seaborn`是一种基于`matplotlib`的图形可视化库。他提前已经定义好了一套自己的风格。然后也封装了一系列的方便的绘图函数，之前通过`matplotlib`需要很多代码才能完成的绘图，使用`seaborn`可能就是一行代码的事情。总结一句话：使用`seaborn`绘图比`matplotlib`更好看，更简单！

##### 安装：

1. 通过`pip`：`pip install seaborn`。
2. 通过`anaconda`： `conda install seaborn`。

##### 官方文档：

`https://seaborn.pydata.org/tutorial.html`



### 第2节　关系绘图

<!-- 来源：http://www.zlkt.net/book/detail/11/338 -->

#### seaborn关系绘图

##### 一、`relplot`：

这个函数功能非常强大，可以用来表示多个变量之间的关联关系。默认情况下是绘制散点图，也可以绘制线性图，具体绘制什么图形是通过`kind`参数来决定的。实际上以下两个函数就是`relplot`的特例：

1. `scatterplot`：`relplot(kind='scatter')`。
2. `lineplot`：`relplot(kind='line')`。

###### 1. 基本使用：

```python
import seaborn as sns
tips = sns.load_dataset("tips",cache=True)
sns.relplot(x="total_bill",y="tip",data=tips)
```

效果图如下：
![relplot1.png](images/relplot1.png)

###### 2. 添加hue参数：

`hue`参数是用来控制第三个变量的颜色显示的。比如我们在以上图的基础之上体现出星期几的参数，那么可以通过以下代码来实现：

```python
sns.relplot(x="total_bill",y="tip",hue="day",data=tips)
```

效果图如下：
![relplot2.png](images/relplot2.png)

###### 3. 添加col和row参数：

`col`和`row`，可以将图根据某个属性的值的个数分割成多列或者多行。比如在以上图的基础之上我们想要把`Lunch(午餐)`和`Dinner(晚餐)`分割成两个图来显示，那么可以通过以下代码来实现：

```python
sns.relplot(x="total_bill",y="tip",hue="day",col="time",data=tips)
```

效果图如下：
![relplot3.png](images/relplot3.png)

也可以再在`row`上添加一个新的变量，比如把性别按照行显示出来，代码如下：

```python
sns.relplot(x="total_bill",y="tip",hue="day",col="time",row="sex",data=tips)
```

效果图如下：
![relplot4.png](images/relplot4.png)

###### 4. 指定具体的列：

有时候我们的图有很多，默认情况下会在一行中全部展示出来，那么我们可以通过`col_wrap`来指定具体多少列。示例代码如下：

```python
sns.relplot(x="total_bill",y="tip",col="day",col_wrap=2,data=tips)
```

效果图如下：
![relplot5.png](images/relplot5.png)

###### 5. 绘制折线图：

`relplot`通过设置`kind="line"`可以绘制折线图。并且他的功能比`plt.plot`更加强大。`plot`只能指定具体的`x`和`y`轴的数据（比如x轴是N个数，y轴也必须为N个数）。而`relplot`则可以在自动在两组数据中进行计算绘图。示例代码如下：

```python
fmri = sns.load_dataset("fmri")
sns.relplot(x="timepoint",y="signal",kind="line",data=fmri)
```

效果图如下：
![relplot6.png](images/relplot6.png)

当然也可以添加其他的参数，用来控制整个图的样式和结构。示例代码如下：

```python
# 设置hue为event，就会根据event来绘制不同的颜色
# 设置col为region，就会根据region值的个数来绘制指定个数的图
# 设置style为event，就会根据event来设置线条的样式
sns.relplot(x="timepoint",y="signal",kind="line",hue="event",col="region",style="event",data=fmri)
```

效果图如下：
![relplot7.png](images/relplot7.png)



### 第3节　分类绘图

<!-- 来源：http://www.zlkt.net/book/detail/11/339 -->

#### 分类绘图

分类图的绘制，采用的是`sns.catplot`来实现的。`cat`是`category`的简写。这个方法默认绘制的是`分类散点图`，如果想要绘制其他类型的图，同样也是通过`kind`参数来指定。并且分类绘图中，分成分类散点图，分类分布图，分类统计图。

##### 一、分类散点图：

分类散点图比较适合数据量不是很多的情况，他是用`catplot`来实现，但是也有以下两个特别的方法。

1. `stripplot()`：`catplot(kind="strip")`，默认的。
2. `swarmplot()`：`catplot(kind="swarm")`。

###### 1.1. stripplot：

示例代码如下：

```python
sns.catplot(x="day",y="total_bill",data=tips,hue="sex")
```

示例图如下：
![catplot1.png](images/catplot1.png)

###### 1.2. swarmplot：

以上图展示的是按照星期几的分类散点图，看起来这些点有点重合，如果想要散开来，那么可以使用`catplot(kind="swarm")`。示例代码如下：

```python
sns.catplot(x="day",y="total_bill",kind="swarm",data=tips,hue="sex")
```

![catplot2.png](images/catplot2.png)

`catplot`方法不能使用`size`和`style`参数。

###### 1.3. 横向分类散点图：

想要将垂直的分类散点图变成横向的，只需要把`x`和`y`对应的值进行互换即可。

```python
sns.catplot(y="day",x="total_bill",kind="swarm",data=tips,hue="sex")
```

效果图如下：
![catplot3.png](images/catplot3.png)

##### 二、分类分布图：

分类分布图，主要是根据分类来看，然后在每个分类下数据的分布情况。也是通过`catplot`来实现，以下三个方法分别是不同的`kind`参数：

1. `boxplot()`：`catplot(kind="box")`。
2. `violinplot()`：`catplot(kind="violin")`。

###### 2.1. 箱线图：

示例代码如下：

```python
athletes = pd.read_csv("athlete_events.csv")
countries = {
    'CHN':'中国',
    'JPN':"日本",
    'KOR':'韩国',
    'USA':"美国",
    'CAN':"加拿大",
    'BRA':"巴西",
    'GBR':"英国",
    'FRA':"法国",
    'ITA':"意大利",
    'ETH':"埃塞俄比亚",
    'KEN':"肯尼亚",
    'NIG':"尼日利亚",
}
plt.rcParams['font.sans-serif'] = ['FangSong']
# print(plt.rcParams.keys())
need_athletes = athletes[athletes['NOC'].isin(list(countries.keys()))]
g = sns.catplot(x="NOC",y="Height",data=need_athletes,kind="box",hue="Sex")
g.fig.set_size_inches(20,5)
g.set_xticklabels(list(countries.values()))
```

效果图如下：
![catplot4.png](images/catplot4.png)

###### 2.2. 小提琴图：

小提琴实际上就是两个对称的核密度曲线合并起来，然后中间是一个箱线图（也可以为其他图）组成的。通过小提琴图可以看出数据的分布情况。示例代码如下：

```python
sns.catplot(x="day",y="total_bill",data=tips,kind="violin",hue="sex",split=True)
```

效果图如下：
![catplot6.png](images/catplot6.png)

小提琴的中间默认绘制的是箱线图，也可以修改为其他类型的。可以通过`inner`参数修改，这个参数有以下几个选项：

1. `box`：默认的，箱线图。
2. `quartile`：四分位数。上下四分位数加中位数。
   ![catplot13.png](images/catplot13.png)
3. `point`：散点。
   ![catplot14.png](images/catplot14.png)
4. `stick`：线条。
   ![catplot15.png](images/catplot15.png)

##### 三、分类统计图：

分类统计图，则是根据分类，统计每个分类下的数据的个数或者比例。有以下几种方式：

1. `barplot()`：`catplot(kind="bar")`。
2. `pointplot()`：`catplot(kind="point")`。
3. `countplot()`：`catplot(kind="count")`。

###### 3.1. 条形图：

`seaborn`中的条形图具有统计功能，可以统计出比例，平均数，也可以按照你想要的统计函数来统计。示例代码如下：

1. 统计平均数：

   ```python
   # 统计星期三到星期天的消费总额的平均数
    sns.catplot(x="day",y="total_bill",data=tips,kind="bar")
   ```

   ![catplot7.png](images/catplot7.png)
2. 统计比例：

   ```python
   # 统计男女中获救的比例
    sns.catplot(data=titanic,kind="bar",x="sex",y="survived")
   ```

   ![catplot8.png](images/catplot8.png)
3. 自定义统计函数：

   ```python
   # 自定义统计函数，统计出每个性别下获救的人数
    sns.barplot(x="sex",y="survived",data=titanic,estimator=lambda values:sum(values))
   ```

   ![catplot9.png](images/catplot9.png)

###### 3.2. 柱状图：

柱状图是专门用来统计某个单一变量出现数量的图形。示例代码如下：

```python
sns.catplot(x="sex",data=titanic,kind="count")
```

![catplot10.png](images/catplot10.png)

也可以通过使用`hue`参数来指定分组：

```python
sns.catplot(x="day",kind="count",data=tips,hue="sex")
```

![catplot11.png](images/catplot11.png)

###### 3.3. 点线图：

点线图可以非常方便的看到变量之间的趋势变化。示例代码如下：

```python
sns.catplot(x="sex",y="survived",data=titanic,kind="point",hue="class")
```

效果图如下：
![catplot12.png](images/catplot12.png)



### 第4节　分布绘图

<!-- 来源：http://www.zlkt.net/book/detail/11/340 -->

#### 分布绘图

分布绘图分为单一变量分布，多变量分布，成对绘图。以下进行讲解。

##### 一、单变量分布：

单一变量主要就是通过直方图来绘制。在`seaborn`中直方图的绘制采用的是`distplot`，其中`dist`是`distribution`的简写，不是`histogram`的简写。示例代码如下：

```python
sns.set(color_codes=True)
titanic = sns.load_dataset("titanic")
titanic = titanic[~np.isnan(titanic['age'])]
sns.distplot(titanic['age'])
```

效果图如下：
![distplot1.png](images/distplot1.png)

有以下常用参数：

1. `kde（核密度曲线）`：这个代表是否要显示`kde`曲线，默认是显示的，如果显示`kde`曲线，那么`y`轴表示的就是概率，而不是数量。也可以设置为`False`关掉。示例代码如下：

   ```python
   sns.distplot(titanic['age'],kde=False)
   ```

   ![distplot2.png](images/distplot2.png)
2. `bins`：代表这个直方图显示的数量。也可以通过自己设置。示例代码如下：

   ```python
   sns.distplot(titanic['age'],bins=30)
   ```

   ![distplot3.png](images/distplot3.png)
3. `rug`：代表是否需要显示底部的胡须下线，下面的胡须线越密集的地方，说明数据量越多。示例代码如下：

   ```python
   sns.distplot(titanic['age'],rug=True)
   ```

   ![distplot4.png](images/distplot4.png)

##### 二、二变量分布：

多变量分布图可以看出两个变量之间的分布关系。一般都是采用多个图进行表示。多变量分布图采用的函数是`jointplot`。

###### 2.1. 散点图：

示例代码如下：

```python
tips = sns.load_dataset("tips")
g = sns.jointplot(x="total_bill", y="tip", data=tips)
```

效果图如下：
![jointplot1.png](images/jointplot1.png)

通过设置`kind='reg'`可以设置回归绘图和核密度曲线。示例代码如下：

```python
g = sns.jointplot(x="total_bill", y="tip", data=tips,kind="reg")
```

效果图如下：
![jointplot2.png](images/jointplot2.png)

###### 2.2. 六边形图：

对于一些数据量特别大的数据，用散点图不太利于观察，比如查看奥运会中国运动员的身高和体重分布情况，如果用散点图将会是以下的效果：

```python
athletes = pd.read_csv("athlete_events.csv")
china_athletes = athletes[athletes['NOC']=='CHN']
sns.jointplot(x="Height",y="Weight",data=china_athletes)
```

![jointplot3.png](images/jointplot3.png)

针对这种数据量比较大的情况，可以采用六边形图来绘制，也就是将之前的散点变成六边形，六边形有一个区间大小，之前这些点落在这个六边形中越多颜色越深。示例代码如下：

```python
sns.jointplot(x="Height",y="Weight",data=china_athletes,kind="hex")
```

默认情况，在`x`轴的区间内，可以展示100个六边形，所以默认情况下六边形的尺寸会比较小，如果想要展示得更大一点，那么可以设置减少六边形的个数，通过`gridsize`设置。示例代码如下：

```python
sns.jointplot(x="Height",y="Weight",data=china_athletes,kind="hex",gridsize=20)
```

![jointplot4.png](images/jointplot4.png)

更多请参考：

###### 2.4. jointplot其他常用参数：

1. `x,y,data`：绘制图的数据。
2. `kind`：`scatter`、`reg`、`resid`、`kde`、`hex`。
3. `color`：绘制元素的颜色。
4. `height`：图的大小，图会是一个正方形。
5. `ratio`：主图和副图的比例，只能为一个整形。
6. `space`：主图和副图的间距。
7. `dropna`：是否需要删除`x`或者`y`值中出现了`NAN`的值。
8. `marginal_kws`：副图的一些属性，比如设置`bins`、`rug`等。

##### 三、成对绘图（pairplot）：

`pairplot`可以把某个数据集中某几个字段之间的关系图一次性绘制出来。比如`iris`鸢尾花数据，我们想要看到`petal_width`、`petal_height`、`sepal_width`以及`sepal_height`之间的关系，那么我们就可以通过`pairplot`来绘制。示例代码如下：

```python
sns.pairplot(iris,vars=['sepal_length',"sepal_width",'petal_length','petal_width'])
```

效果图如下：
![pairplot1.png](images/pairplot1.png)

默认情况下，对角线的图（x和y轴的列相同）是直方图，其他地方的图是散点图，如果想要修改这两种图，可以通过`diag_kind`和`kind`来实现。其中这两个参数可取的值为：

1. `diag_kind`：`auto`, `hist`, `kde`。
2. `kind`：`scatter`, `reg`。

示例代码如下：

```python
sns.pairplot(iris,vars=['sepal_length',"sepal_width",'petal_length','petal_width'],diag_kind="kde",kind="reg")
```

![pairplot2.png](images/pairplot2.png)



### 第5节　线性关系绘图

<!-- 来源：http://www.zlkt.net/book/detail/11/341 -->

#### 线性回归绘图

线性回归图可以帮助我们看到数据的关系趋势。在`seaborn`中可以通过`regplot`和`lmplot`两个函数来实现。`regplot`的`x`和`y`可以为`Numpy数组`、`Series`等变量。而`lmplot`的`x`和`y`则必须为字符串，并且`data`的值不能为空：

1. `regplot(x,y,data=None)`。
2. `lmplot(x,y,data)`。

示例代码如下：

```python
sns.lmplot(x="total_bill",y="tip",data=tips)
```

![线性绘图2.png](images/线性绘图2.png)

也可以通过`regplot`来实现。示例代码如下：

```python
sns.regplot(x=tips["total_bill"],y=tips["tip"])
```

![线性绘图1.png](images/线性绘图1.png)

##### 更多请参考文档：

`https://seaborn.pydata.org/tutorial/regression.html`



### 第6节　FacetGrid结构图

<!-- 来源：http://www.zlkt.net/book/detail/11/342 -->

#### FacetGrid结构图

之前我们在绘图的时候，学了`relplot`、`catplot`、`lmplot`等，这些函数可以通过`col`、`row`等在一个`Figure`中绘制多个图。这些函数之所以有这些功能，是因为他们的底层使用了`FacetGrid`来组装这些图形。今天我们就来学习`FacetGrid`的使用。

##### 一、普通的Axes绘图：

在学习`FacetGrid`绘图之前，先来了解一下，实际上`seaborn`的绘图函数中也有大量的直接使用`Axes`进行绘图的，凡是函数名中已经明确显示了这个图的类型，这种图都是使用`Axes`绘图的。比如`sns.scatterplot`、`sns.lineplot`、`sns.barplot`等。`Axes`绘图可以直接使用之前`matplotlib`的一些方式设置图的元素。示例代码如下：

```python
fig,[ax1,ax2] = plt.subplots(2,1,figsize=(10,10))
sns.scatterplot(x="total_bill",y="tip",data=tips,ax=ax1)
sns.barplot(x="day",y="total_bill",data=tips,ax=ax2)
```

![facetgrid9.png](images/facetgrid9.png)

##### 二、FacetGrid基本使用：

先创建一个`FacetGrid`对象，然后再调用这个对象的`map`方法。其中`map`方法的第一个参数是一个函数，后续`map`将调用这个函数来绘制图形。后面的参数就是传给这个函数的参数。示例代码如下：

```python
tips = sns.load_dataset("tips")
g = sns.FacetGrid(tips)
g.map(plt.scatter,"total_bill","tip")
```

效果图如下：
![facetgrid1.png](images/facetgrid1.png)

其中第一个参数是可以绘制`Axes`图，并且可以接收`color`参数的函数。可以取的值如下：

| 参数 | 描述 | 对应使用了`FacetGrid`函数 |
| --- | --- | --- |
| `plt.plot`/`sns.lineplot` | 绘制折线图 | `sns.relplot(kind="line")` |
| `plt.hexbin` | 绘制六边形图形 | `sns.jointplot(kind="hex")` |
| `plt.hist` | 绘制直方图 | `sns.distplot` |
| `plt.scatter`/`sns.scatterplot` | 绘制散点图 | `sns.relplot(kind="scatter")` |
| `sns.stripplot` | 绘制分类散点图 | `sns.catplot(kind="strip")` |
| `sns.swarmplot` | 绘制散开来的分类散点图 | `sns.catplot(kind="swarm")` |
| `sns.boxplot` | 绘制箱线图 | `sns.catplot(kind="box")` |
| `sns.violinplot` | 绘制小提琴图 | `sns.catplot(kind="violin")` |
| `sns.pointplot` | 绘制点线图 | `sns.catplot(kind="point")` |
| `sns.barplot` | 绘制条形图 | `sns.catplot(kind="bar")` |
| `sns.countplot` | 绘制数量柱状图 | `sns.catplot(kind="count")` |
| `sns.regplot` | 绘制带有回归线的散点图 | `sns.lmplot` |

##### 三、绘制多个图形：

`FacetGrid`可以通过`col`和`row`参数，来在一个`Figure`上绘制多个图形，其中`col`和`row`都是数据集中的某个列的名字。只要指定这个名字，那么就会自动的按照指定列的值的个数绘制指定个数的图形。示例代码如下：

```python
g = sns.FacetGrid(tips,col="day",col_wrap=2)
g.map(sns.regplot,"total_bill","tip")
```

效果图如下：
![facetgrid2.png](images/facetgrid2.png)

##### 四、添加颜色观察字段：

可以通过添加`hue`参数来控制每个图中元素的颜色来观察其他的字段。示例代码如下：

```python
g = sns.FacetGrid(tips,col="day",hue="time")
g.map(sns.regplot,"total_bill","tip")
```

![facetgrid4.png](images/facetgrid4.png)

也可以通过`hue_kws`参数来添加`hue`散点的属性，比如设置散点的样式等。

##### 五、设置每个图形的尺寸：

使用`FacetGrid`绘制出图形后，有时候我们想设置每个图形的尺寸或者是宽高比，那么我们可以通过在`FacetGrid`中设置`height`和`aspect`来实现，其中`height`表示的是每个图形的尺寸（默认是宽高一致），`aspect`表示的是`宽度/高度`的比例。示例代码如下：

```python
g = sns.FacetGrid(tips,col="day",row="time",height=10,aspect=2)
g.map(sns.regplot,"total_bill","tip")
```

效果图如下：
![facetgrid3.png](images/facetgrid3.png)

##### 六、设置图例：

默认情况下，不会添加图例，我们可以通过`g.add_legend()`来添加图例。示例代码如下：

```python
g = sns.FacetGrid(tips,col="day",hue="time")
g.map(sns.regplot,"total_bill","tip")
g.add_legend()
```

![facetgrid5.png](images/facetgrid5.png)

另外还可以：

1. 通过`title`来控制图例的标题。
2. 通过`label_order`来控制图例元素的顺序。

示例代码如下：

```python
sns.set(rc={"font.sans-serif":"simhei"})
g3 = sns.FacetGrid(tips,col="day",hue="time")
g3.map(plt.scatter,"total_bill","tip")
new_labels = ['午餐','晚餐']
g3.add_legend(title="时间")
for t,l in zip(g3._legend.texts,new_labels):
    t.set_text(l)
```

![facetgrid6.png](images/facetgrid6.png)

##### 七、设置标题：

设置标题可以通过`g.set_titles(template=None,row_template=None,col_template=None)`来实现，这三个参数分别代表的意义如下：

1. `template`：给图设置标题，其中有`{row_var}：绘制每行图像的名称`，`{row_name}：绘制每行图像的值`，`{col_var}：绘制每列图像的名称`，`{col_name}：绘制每列图像的值`这几个参数可以使用。
2. `col_template`：给图像设置列的标题。其中有`{col_var}`以及`{col_name}`可以使用。
3. `row_template`：给图像设置行的标题。其中有`{row_var}`以及`{row_name}`可以使用。

示例代码如下：

```python
g = sns.FacetGrid(tips,col="day",row="time")
g.map(sns.regplot,"total_bill","tip")
g.set_titles(template="时间{row_name}/星期{col_name}")
```

![facetgrid7.png](images/facetgrid7.png)

##### 八、设置坐标轴：

1. `g.set_axis_labels(x_var,y_var)`：一次性设置`x`和`y`的坐标的标题。
2. `g.set_xlabels(label)`：设置`x`轴的标题。
3. `g.set_ylabels(label)`：设置`y`轴的标题。
4. `g.set(xticks,yticks)`：设置`x`和`y`轴的刻度。
5. `g.set_xticklabels(labels)`：设置`x`轴的刻度文字。
6. `g.set_yticklabels(labels)`：设置`y`轴的刻度文字。

示例代码如下：

```python
g.set(xticks=range(0,60,10),xticklabels=['$0','$10','$20','$30','$40','$50'])
```

效果图如下：
![facetgrid8.png](images/facetgrid8.png)

##### 九、`g.set`方法：

`g.set`方法可以对`FacetGrid`下的每个子图`Axes`设置属性。其中可以设置的参数完全是根据`Axes`的属性来的。比如可以设置每个`Axes`的`facecolor`等。关于`Axes`有哪些属性，请参考`matplotlib.Axes`的官方文档：`https://matplotlib.org/api/axes_api.html?highlight=axes#matplotlib.axes.Axes`。

##### 十、`g.fig`：

通过`g.fig`，可以获取到当前的`Figure`对象。然后通过`Figure`对象再可以设置其他的属性，比如`dip`等。



### 第7节　样式设置

<!-- 来源：http://www.zlkt.net/book/detail/11/343 -->

#### seaborn样式风格设置

用`seaborn`绘图，比直接使用`matplotlib`绘图更加的美观。原因就是因为`seaborn`中已经将一些属性的样式进行了调整。我们可以直接使用，也可以修改他的样式。

##### 一、自带的样式：

`seaborn`中自带了5种样式。分别是：

1. `white`：纯白色的。

   ```python
   sns.set_style("white")
    axes = sns.scatterplot(x="total_bill",y="tip",data=tips)
   ```

   ![](http://www.zlkt.net/assets/chapter05/%E9%A3%8E%E6%A0%BC%E8%AE%BE%E7%BD%AE1.png)
2. `whitegrid`：带有网格的白色的。

   ```python
   sns.set_style("whitegrid")
    axes = sns.scatterplot(x="total_bill",y="tip",data=tips)
   ```

   ![](http://www.zlkt.net/assets/chapter05/%E9%A3%8E%E6%A0%BC%E8%AE%BE%E7%BD%AE2.png)
3. `dark`：灰色的。

   ```python
   sns.set_style("dark")
    axes = sns.scatterplot(x="total_bill",y="tip",data=tips)
   ```

   ![](http://www.zlkt.net/assets/chapter05/%E9%A3%8E%E6%A0%BC%E8%AE%BE%E7%BD%AE3.png)
4. `darkgrid`：带有网格的灰色的（网格线是白色的）。

   ```python
   sns.set_style("darkgrid")
    axes = sns.scatterplot(x="total_bill",y="tip",data=tips)
   ```

   ![](http://www.zlkt.net/assets/chapter05/%E9%A3%8E%E6%A0%BC%E8%AE%BE%E7%BD%AE4.png)
5. `ticks`：白色的，并且在轴上带有刻度条的。

   ```python
   sns.set_style("ticks")
    axes = sns.scatterplot(x="total_bill",y="tip",data=tips)
   ```

   ![](http://www.zlkt.net/assets/chapter05/%E9%A3%8E%E6%A0%BC%E8%AE%BE%E7%BD%AE5.png)

##### 二、风格设置函数：

在`seaborn`中，可以通过三个函数来设置样式。分别是`sns.set_style`、`sns.axes_style`以及`sns.set`方法。以下对着三种方法进行讲解。

###### 1. `sns.axes_style`：

`sns.axes_style(style=None,rc=None)`。
这个函数调用的时候如果不传递任何参数，那么将会返回可以设置的所有属性。有时候我们不知道什么属性可以设置，那么可以打印下这个函数的返回值：

```python
sns.axes_style()
```

输入如下：

```
{'axes.facecolor': 'white',
 'axes.edgecolor': 'black',
 'axes.grid': False,
 'axes.axisbelow': 'line',
 'axes.labelcolor': 'black',
 'figure.facecolor': (1, 1, 1, 0),
 'grid.color': '#b0b0b0',
 'grid.linestyle': '-',
 'text.color': 'black',
 'xtick.color': 'black',
 'ytick.color': 'black',
 'xtick.direction': 'out',
 'ytick.direction': 'out',
 'lines.solid_capstyle': 'projecting',
 'patch.edgecolor': 'black',
 'image.cmap': 'viridis',
 'font.family': ['sans-serif'],
 'font.sans-serif': ['DejaVu Sans',
  'Bitstream Vera Sans',
  'Computer Modern Sans Serif',
  'Lucida Grande',
  'Verdana',
  'Geneva',
  'Lucid',
  'Arial',
  'Helvetica',
  'Avant Garde',
  'sans-serif'],
 'patch.force_edgecolor': False,
 'xtick.bottom': True,
 'xtick.top': False,
 'ytick.left': True,
 'ytick.right': False,
 'axes.spines.left': True,
 'axes.spines.bottom': True,
 'axes.spines.right': True,
 'axes.spines.top': True}
```

这个函数也可以用来设置样式，但是只能通过`with`语句调用。示例代码如下：

```python
with sns.axes_style("dark",{"ytick.left":True}):
    sns.scatterplot(x="total_bill",y="tip",data=tips)
```

##### 2. `sns.set_style()`：

`sns.set_style(style=None,rc=None)`。
这个函数跟`sns.axes_style`一样，也是用来设置绘图风格。但是这个函数的风格设置，不是临时的，而是一旦设置了，那么下面的所有绘图都是用这个风格。示例代码如下：

```python
sns.set_style("darkgrid")
sns.scatterplot(x="total_bill",y="tip",data=tips)
```

##### 3. `sns.set`：

`sns.set(context='notebook', style='darkgrid', palette='deep', font='sans-serif', font_scale=1, color_codes=True, rc=None)`。

`set`方法也是用来设置样式的，他的功能更加强大。除了`style`以外，还可以设置调色板，字体，字体大小，颜色等，也可以设置其他的`matplotlib.rcParams`可以接收的参数。示例代码如下：

```python
sns.set(rc={"lines.linewidth":4})
fmri = sns.load_dataset("fmri")
sns.lineplot(x="timepoint",y="signal",data=fmri)
```

效果图如下：
![风格设置6.png](images/风格设置6.png)



### 第8节　调色盘设置

<!-- 来源：http://www.zlkt.net/book/detail/11/344 -->

#### 调色盘设置

`seaborn`可以非常迅速的做出优美的图形，其中就应该得力于他的调色盘机制。`seaborn`根据应用场景提供了三种不同类型的调色盘：`定性的`、`连续的`、`发散的`。

##### 一、定性调色盘：

定性调色盘。一般在数据不连续，比较离散，想体现分类的情况下使用。在`seaborn`中，使用`sns.color_palette`来创建调色盘。

###### 1. 默认调色盘：

在`seaborn`中，默认情况下就设置了一些颜色供绘图使用。使用`sns.color_palette`即可获取。并且我们可以通过`sns.palplot`来绘制调色盘。示例代码如下：

```python
current_palette = sns.color_palette()
sns.palplot(current_palette)
```

效果图如下：
![调色盘1.png](images/调色盘1.png)

默认的调色盘有10中颜色。这些颜色都有6中风格。分别是：`deep`，`muted`，`pastel`， `bright`，`dark`，`colorblind`。这几种风格的颜色不变，主要调整的是亮度和饱和度。
![调色盘2.png](images/调色盘2.png)

```python
current_palette = sns.color_palette("dark")
sns.palplot(current_palette)
```

![调色盘3.png](images/调色盘3.png)

###### 2. hls圆形颜色系统：

`hls`圆形颜色系统是颜色按照顺序，经过偏移，无缝形成一个圆形。我们在使用这个调色盘的时候，可以指定需要使用多少种颜色。示例代码如下：

```python
# 使用hls圆形颜色系统，取20个颜色
sns.palplot(sns.color_palette("hls",20))
```

![调色盘4.png](images/调色盘4.png)

也可以使用另外一个函数`sns.hls_palette(n_colors=6, h=0.01, l=0.6, s=0.65)`来实现。这个函数可以传递更多的参数。比如我们可以通过更改`hue`来更改开始的颜色，通过更改`l`来调整亮度，通过更改`s`来调整饱和度。示例代码如下：

```python
sns.palplot(sns.hls_palette(10,h=0.4,l=0.4,s=0.5))
```

![调色盘5.png](images/调色盘5.png)

另外也可以通过`sns.husl_palette`来实现色系的调整，这个方法比`sns.hls_palette`亮度和饱和度更加的均匀。

```python
sns.palplot(sns.husl_palette(10))
```

![调色盘6.png](images/调色盘6.png)

###### 3. 分类颜色：

分类颜色是`seaborn`已经提前给你定义了一些颜色，使用这些颜色在做分类分组的时候可以按照自己的需求选择。示例代码如下：

```python
sns.palplot(sns.color_palette("Paired"))
```

![调色盘7.png](images/调色盘7.png)

关于分类的颜色选择，可以通过`sns.choose_colorbrewer_palette("qualitative")`来查看。这个方法只能用在`jupyter notebook`中。可以选择不同的样式，然后还可以调节饱和度等。效果图如下：
![调色盘8.png](images/调色盘8.png)

###### 4. 用xkcd颜色：

`xkcd`是一个漫画名称或者是工作室。`xkcd`开展了一项众包活动，为随机的`RGB`颜色命名。这产生了一组`954`种命名颜色。我们可以从`sns.xkcd_palette`里面提取颜色。提取到后，如果想要用在`palette`参数中，那么还需要放到`sns.xkcd_palette`中。所有的`xkcd`颜色的名称可以参考官网：`https://xkcd.com/color/rgb/`。示例代码如下：

```python
# 获取名字为blue green的颜色
print(sns.xkcd_rgb["blue green"])
# 用xkcd的颜色名称构建一个palette对象
colors = ["windows blue", "amber", "greyish", "faded green", "dusty purple"]
sns.palplot(sns.xkcd_palette(colors))
```

---

##### 二、连续的颜色盘：

有时候我们绘图的时候，想要使用一个同种色系，但是不同深浅，这时候就可以使用连续的颜色盘。示例代码如下：

```python
sns.palplot(sns.color_palette("Blues"))
```

![调色盘10.png](images/调色盘10.png)

默认颜色是从浅入深，如果想要从深变浅，那么可以在色系后加一个`_r`。示例代码如下：

```python
sns.palplot(sns.color_palette("Blues_r"))
```

![调色盘11.png](images/调色盘11.png)

我们也可以通过`sns.choose_colorbrewer_palette("sequential")`查看有哪些色系可供选择。效果图如下：
![调色盘12.png](images/调色盘12.png)

##### 三、离散的色盘：

离散的色盘，是两边的颜色逐渐加深，中间的颜色最淡。或者是中间的颜色最深，两边的颜色最淡。一般离散的色盘可以用于比如温度，零度以上可以用红色表示，零度以下用蓝色表示。越红的地方，表示温度越高，越蓝的地方，表示温度越低。示例代码如下：

```python
values = [12,15,17,18,-5,-10]
with sns.color_palette("RdBu_r"):
    sns.barplot([1,2,3,4,5,6],sorted(values))
```

![调色盘13.png](images/调色盘13.png)

也可以通过`sns.choose_colorbrewer_palette("diverging")`查看离散的色盘有哪些可以选择。
还可以通过`sns.diverging_palette(h_neg, h_pos, s=75, l=50, sep=10, n=6, center='light', as_cmap=False)`来自定义离散色盘。在这里不再做过多讲解。

##### 四、官方文档：

`https://seaborn.pydata.org/tutorial/color_palettes.html`。



### 第9节　seaborn作业

<!-- 来源：http://www.zlkt.net/book/detail/11/345 -->

#### Seaborn作业

##### 一、 有一组温度数据，按照时间和温度绘制折线图。

```python
bj_temps = [29,27,23,22]
bj_hours = ["20时","23时","2时","5时"]
plt.figure(figsize=(5,2))
axes = sns.lineplot(range(0,4),bj_temps,marker="o")
axes.set_xticks(range(0,4))
axes.set_xticklabels(bj_hours)
```

效果图如下：
![作业1.png](images/作业1.png)

##### 二、有以下国家数据，根据时间绘制条形图。

```python
legals = pd.read_csv("../法人人数年度数据.csv",encoding='GB18030')
temp_legals = legals[1:11]

# 清理数据
new_legals = pd.DataFrame()
for index in temp_legals.index:
    row_values =temp_legals.loc[index]
    for x in range(2009,2018):
        year = "%d年"%x
        series = pd.Series({"指标":row_values['指标'],'年份':year,"数量":row_values[year]})
        new_legals = pd.concat([new_legals,series.to_frame().T])
new_legals.reset_index(drop=True,inplace=True)

# 开始绘图
plt.figure(figsize=(20,5))
sns.barplot(x="年份",y="数量",hue="指标",data=new_legals)
plt.legend(ncol=4)
```

![作业2.png](images/作业2.png)

##### 三、有链家网的数据，请按照以下要求实现绘图：

1. `x`轴是`Region（行政区）`，`y`轴是每个区的平均每平米的单价，绘制条形图。`x`轴是`Region（行政区）`，`y`轴是每平米的单价，绘制箱线图。`x`轴是`Region`，`y`轴是每平米的单价，绘制`swarm`图。以上三个图需要绘制在一个`figure`上。

   ```python
   lianjia = pd.read_csv("../lianjia.csv",encoding='utf-8')
   lianjia['UnitPrice'] = lianjia['Price']/lianjia['Size']
   house_mean = lianjia.groupby('Region')['UnitPrice'].mean().sort_values(ascending=False).to_frame().reset_index()
   fig,axes_arr = plt.subplots(3,1,figsize=(20,15))
   sns.barplot(x="Region",y="UnitPrice",data=house_mean,ax=axes_arr[0])
   sns.boxplot(x="Region",y="UnitPrice",data=lianjia,ax=axes_arr[1])
   sns.swarmplot(x="Region",y="UnitPrice",data=lianjia,ax=axes_arr[2])
   ```

   ![作业3.png](images/作业3.png)
2. 使用`FacetGrid`绘制尺寸与单价的关系，并且区分有无电梯。

   ```python
   fg = sns.FacetGrid(lianjia,col="Elevator",height=6,aspect=2)
   fg.map(sns.regplot,"Size","UnitPrice")
   fg.add_legend()
   ```

   ![作业4.png](images/作业4.png)

