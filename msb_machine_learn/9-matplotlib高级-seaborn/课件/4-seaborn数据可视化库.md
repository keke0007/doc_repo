### 1、Seaborn介绍

Seaborn是基于matplotlib的图形可视化python包。它提供了一种高度交互式界面，便于用户能够做出各种有吸引力的统计图表。

Seaborn是在matplotlib的基础上进行了更高级的API封装，从而使得作图更加容易，在大多数情况下使用seaborn能做出很具有吸引力的图，而使用matplotlib就能制作具有更多特色的图。应该把Seaborn视为matplotlib的补充，而不是替代物。

### 2、安装

pip install seaborn -i https://pypi.tuna.tsinghua.edu.cn/simple

[教程](https://blog.csdn.net/Soft_Po/article/details/118605172)

### 3、快速上手

#### 3.1、样式设置

```Python
import seaborn as sns
sns.set(style = 'darkgrid',context = 'talk',font = 'STKaiti')
```

stlyle设置，修改主题风格，属性如下：

| style     | 效果                 |
| --------- | -------------------- |
| darkgrid  | 黑色网格（默认）     |
| whitegrid | 白色网格             |
| dark      | 黑色背景             |
| white     | 白色背景             |
| ticks     | 四周有刻度线的白背景 |

context设置，修改大小，属性如下：

| context          | 效果             |
| ---------------- | ---------------- |
| paper            | 越来越大越来越粗 |
| notebook（默认） | 越来越大越来越粗 |
| talk             | 越来越大越来越粗 |
| poster           | 越来越大越来越粗 |

#### 3.2、线形图

```Python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
sns.set(style = 'dark',context = 'poster',font = 'STKaiti') # 设置样式
plt.figure(figsize=(9,6))

x = np.linspace(0,2*np.pi,20)
y = np.sin(x)

sns.lineplot(x = x,y = y,color = 'green',ls = '--')
sns.lineplot(x = x,y = np.cos(x),color = 'red',ls = '-.')
```

![](./images/1-seaborn-线形图.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1651050856008/c3c3f5a8880640c0919c30115e7b46c2.png)

### 4、各种图形绘制

#### 4.1、调色板

参数palette（调色板），用于调整颜色，系统默认提供了六种选择：`deep, muted, bright, pastel, dark, colorblind`

参数palette调色板，可以有更多的颜色选择，Matplotlib为我们提供了多大178种，这足够绘图用，可以通过代码**print(plt.colormaps())**查看选择

| 178种    |
| -------- |
| Accent   |
| Accent_r |
| Blues    |
| Blues_r  |
| ……     |

#### 4.2、线形图

```Python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
sns.set(style = 'dark',context = 'notebook',font = 'STKaiti') # 设置样式
plt.figure(figsize=(9,6))
fmri = pd.read_csv('./fmri.csv') # fmri这一核磁共振数据

ax = sns.lineplot(x = 'timepoint',y = 'signal',
                  hue = 'event',style = 'event' ,
                  data= fmri,
                  palette='deep',
                  markers=True,
                  markersize = 10)

plt.xlabel('时间节点',fontsize = 30)
plt.savefig('./线形图.png',dpi = 200)
```

lineplot()函数作用是绘制**线型图**。参数x、y，表示**横纵**坐标；参数hue，表示根据属性**分类**绘制**两条线**（"event"属性分两类"stim"、"cue"）；参数style，表示根据属性分类设置**样式**，实线和虚线；参数data，表示**数据**；参数marker、markersize，分别表示画图**标记点**以及尺寸**大小**！

![](./images/2-seaborn-线形图.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1651050856008/d8175123d3ae4b098ac83e688d4479b1.png)

#### 4.3、散点图

```Python
import matplotlib.pyplot as plt
import seaborn as sns
data = pd.read_csv('./tips.csv') # 小费
plt.figure(figsize=(9,6))
sns.set(style = 'darkgrid',context = 'talk')
# 散点图
fig = sns.scatterplot(x = 'total_bill', y = 'tip', 
                      hue = 'time', data = data, 
                      palette = 'autumn', s = 100)
```

![](./images/3-seaborn-散点图.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1651050856008/bb23e7ea4966430a97d6ee7c51588025.png)

#### 4.4、柱状图

```Python
import seaborn as sns
import matplotlib.pyplot as plt
plt.figure(figsize = (9,6))
sns.set(style = 'whitegrid')
tips = pd.read_csv('./tips.csv') # 小费
ax = sns.barplot(x = "day", y = "total_bill", 
                 data = tips,hue = 'sex',
                 palette = 'colorblind',
                 capsize = 0.2)
```

![](./images/4-seaborn-柱状图.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1651050856008/d98bc105698a492cbf0566c270f5b459.png)

#### 4.5、箱式图

```Python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
sns.set(style = 'ticks')
tips = pd.read_csv('./tips.csv')
ax = sns.boxplot(x="day", y="total_bill", data=tips,palette='colorblind')
```

![](./images/5-seaborn-箱式图.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1651050856008/fbae898cb5aa4e29ac2363a0803b63ed.png)

#### 4.6、直方图

```Python
import seaborn as sns
import numpy as np
import matplotlib.pyplot as plt
sns.set(style = 'dark')
x = np.random.randn(5000)
sns.histplot(x,kde = True)
```

![](./images/6-seaborn-直方图.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1651050856008/3377301baf63428e924ce84a5bc5453c.png)

```Python
import seaborn as sns
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
sns.set(style = 'darkgrid')
tips = pd.read_csv('./tips.csv')
sns.histplot(x = 'total_bill', data = tips, kde = True)
```

![](./images/7-seaborn-直方图.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1651050856008/0dc0222737d14366823febd4753ca581.png)

#### 4.7、分类散点图

```Python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
sns.set(style = 'darkgrid')
exercise = pd.read_csv('./exercise.csv')
sns.catplot(x="time", y="pulse", hue="kind", data=exercise)
```

![](./images/8-seaborn-分类散点图.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1651050856008/3fad4f88e96e429d97f4e8c9f60ecf6a.png)

#### 4.8、热力图

```Python
import matplotlib.pyplot as plt
import seaborn as sns
plt.figure(figsize=(12,9))
flights = pd.read_csv('./flights.csv')

flights = flights.pivot("month", "year", "passengers")
sns.heatmap(flights, annot=True,fmt = 'd',cmap = 'RdBu_r',
            linewidths=0.5)
```

![](./images/9-seaborn-热力图.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1651050856008/61d08363631f4626971321fd25d65d1b.png)
