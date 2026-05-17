# 1、支持向量机SVM

对两类样本点进行分类，如下图，有a线、b线、c线三条线都可以将两类样本点很好的分开类，我们可以观察到b线将两类样本点分类最好，原因是我们训练出来的分类模型主要应用到未知样本中，虽然a、b、c三条线将训练集都很好的分开类，但是当三个模型应用到新样本中时，b线抗干扰能力最强，也就是泛化能力最好，样本变化一些关系不大，一样能被正确的分类。那么如何确定b线的位置呢？我们可以使用支持向量机SVM来确定b线的位置。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/9f0d04393b2248e98273bd8ff9ab4427.png)

支持向量机（support vector machines,SVM）是一种二分类算法，它的目的是寻找一个**超平面**来对样本进行分割，分割的原则是**间隔最大化**，如果对应的样本特征少，一个普通的SVM就是一条线将样本分隔开，但是要求线到两个类别最近样本点的距离要最大。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/d9bc150073b64d9ea3db5dd86bf73e8d.png)

如上图所示，**b线**就是我们根据支持向量机要找到的分割线，这条线需要到两个类别最近的样本点最远，图上的距离就是d，由于这条线处于两个类别的中间，所以到两个类别的距离都是d。

支持向量机SVM算法就是在寻找一个**最优的决策边界**（上图中的两条虚线）来确定分类线b，所说的支持向量（support vector）就是距离决策边界最近的点（上图中p1、p2、p3点，只不过边界穿过了这些点）。如果没有这些支持向量点，b线的位置就要改变，所以 **SVM就是根据这些支持向量点来最大化margin，来找到了最优的分类线（machine，分类器）** ，这也是SVM分类算法名称的由来。

# 2、构建SVM目标函数

接着上面的分类问题来分析，假设支持向量机最终要找的线是$l_2$，决策边界两条线是$l_1$和$l_3$，那么假设$l_2$的方程为$w^T \cdot x + b =0$，这里$w$表示$(w_1,w_2)^T$，$x$表示$(x_1,x_2)^T$，我们想要确定$l_2$直线，只需要确定w和b即可，此外，由于$l_1$和$l_3$线是$l_2$的决策分类边界线，一定与$l_2$是平行的，平行就意味着斜率不变，b变化即可，所以我们可以假设$l_1$线的方程为$w^T \cdot x +b = c$，$l_3$线的方位为$w^T\cdot x + b = -c$。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/b231bc809f6f41a696c2247d57f9d0e5.png)

【知识补充】：

* 二维空间点$(x_1,y_1)$到直线$Ax + By + C = 0$的距离公式是：

$$
d = \frac{|Ax_1 + By_1 + C|}{\sqrt{A^2 + B^2}}
$$

* 两条平行线$Ax + By + C_1 = 0$与$Ax + By + C_2= 0$之间的距离公式为：

$$
d = \frac{|C_1 - C_2|}{\sqrt{A^2 + B^2}}
$$

我们希望的是决策边界上的样本点到$l_2$直线的距离d越大越好。我们可以直接计算$l_1$或$l_3$上的点到直线$l_2$的距离，假设将空间点扩展到n维空间后，点$x_i = (x_1,x_2,\cdots,x_n)$到直线$w^T \cdot x +b = 0$（严格意义上来说直线变成了超平面）的距离为：

$$
d = \frac{|w^T\cdot x_i +b|}{||w||}
$$

以上$||w|| = \sqrt{w_1^2 + w_2^2 + \cdots + w_n^2}$，读作“w的模”。

由于$l_1$、$l_2$和$l_3$三条直线平行，d也是两条平行线之间的距离，我们可以根据两条平行线之间的距离公式得到d值如下：

$$
d = \frac{|C - 0|}{||w||} = \frac{|-C - 0|}{||w||} = \frac{|C|}{||w||} = \frac{|w^T\cdot x_i +b|}{||w||}
$$

我们可以看到无论计算点到直线的距离还是两个平行线之间的距离，最终结果是一致的。对于$l_1$直线$w^T\cdot x +b = c$，我们可以相对应成比例的缩小各项的系数，例如变成$\frac{w^T}{2}\cdot x + \frac{b}{2}=\frac{c}{2}$，这条直线还是原来本身的直线，现在我们可以把它变成$\frac{w^T}{c}\cdot x + \frac{b}{c}=\frac{c}{c}=1$，即改写后的$l_1$直线$w^T\cdot x +b = 1$还是原来的直线，同理，对于$l_3$直线我们也可以变换成$w^T\cdot x +b = -1$，所以上图可以改变成如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/dcdb28a5791c48c5a6920fbee7e7462c.png)

那么变换之后的d可以根据两条平行线之间的距离得到：

$$
d = \frac{1}{||w||} = \frac{1}{||w||}
$$

我们希望决策边界上的点到超平面$l_2$距离d越大越好。我们将$l_3$上方的点分类标记为“1”类别，将$l_1$下方的点分类标记为“-1”类别，那么我们可以得到如下公式：

$$
\left \{\begin{aligned}&w^T\cdot x_i +b \ge 1 \quad y = 1 \quad (1)式\\\\&w^T\cdot x_i +b \le 1\quad y = -1\quad (2)式 \end{aligned}\right.
$$

两式两侧分别乘以对应的类别可以合并为一个式子：

$$
y\cdot (w^T\cdot x_i +b) \ge 1 \quad (3)式
$$

所以在计算d的最大值时，是有限制条件的，这个限制条件就是(3)式。

所以对于所有样本点我们需要在满足$y\cdot (w^T\cdot x_i +b) \ge 1$条件下，最大化$d = \frac{1}{||w||}$，就是最小化$||w||$，等效于最小化$\frac{1}{2}||w||^2$，这样转换的目的就是为了后期优化过程中求导时方便操作，不会影响最终的结果，所以在支持向量机SVM中要确定一个分类线，我们要做的就是在$y\cdot (w^T\cdot x_i +b) \ge 1$条件下最小化$\frac{1}{2}||w||^2$，记为:

$$
min\frac{1}{2}||w||\quad s.t\quad y\cdot (w^T\cdot x_i +b) \ge 1,\quad i = 1,2,\cdots,n
$$

以上n代表样本总数，缩写$s.t$表示“Subject to”是“服从某某条件”的意思，上述公式描述的是一个典型的不等式约束条件下的二次型函数优化问题，同时也是支持向量机的基本数学模型，这里我们称支持向量机的目标函数。

# 3、拉格朗日乘数法、KKT条件、对偶

## 3.1、拉格朗日乘数法

**拉格朗日乘数法主要是将有等式约束条件优化问题转换为无约束优化问题** ，拉格朗日乘数法如下：

假设$x = [x_1,x_2,\cdots,x_n]$是一个n维向量，$f(x)$和$h(x)$含有x的函数，我们需要找到满足$h(x)=0$条件下$f(x)$最小值，如下：

$$
minf(x)\quad s.t \quad h(x) = 0
$$

可以引入一个自变量$\lambda$，其可以取任意值，则上式严格等价于下式：

$$
L(x,\lambda) = f(x) + \lambda h(x)
$$

$L(x,\lambda)$叫做Lagrange函数（拉格朗日函数），$\lambda$叫做拉格朗日乘子（其实就是系数）。令$L(x,\lambda)$对x的导数为0，对$\lambda$的导数为0，求解出$x,\lambda$ 的值，那么x就是函数$f(x)$在附加条件$h(x)$下极值点。

以上就是拉格朗日乘数法，通俗理解**拉格朗日乘数法就是将含有等式条件约束优化问题转换成了无约束优化问题**构造出拉格朗日函数$L(x,\lambda)$，让$L(x,\lambda)$对未知数x和$\lambda$进行求导，得到一组方程式，可以计算出对应的x和$\lambda$结果，这个x对应的$f(x)$函数值就是在条件$h(x) = 0$下的最小值点。

拉格朗日函数转换成等式过程的证明涉及到导数积分等数学知识，这里不再证明。

参考文章：

[拉格朗日函数转换推导](https://blog.csdn.net/Soft_Po/article/details/118332454)

## 3.2、KKT条件

假设我们面对的是不等式条件约束优化问题，如下：

假设$x = [x_1,x_2,\cdots,x_n]$是一个n维向量，$f(x)$和$h(x)$是含有x的函数，我们需要找到满足$h(x) \le 0$条件下$f(x)$最小值，如下：

$$
minf(x)\quad s.t \quad h(x) \le 0\quad ---式①
$$

针对上式，显然是一个不等式约束最优化问题，不能再使用拉格朗日乘数法，因为拉格朗日乘数法是针对等式约束最优化问题。

那我们可以考虑加入一个“松弛变量”$\alpha^2$让条件$h(x) \le0$来达到等式的效果，即使条件变成$h(x) + \alpha^2 =0$，这里加上$\alpha^2$的原因是保证加的一定是非负数，即：$\alpha^2 \ge 0$，但是目前不知道这个$\alpha^2$值是多少，一定会找到一个合适的$\alpha^2$值使$h(x) +\alpha^2 = 0$成立，所以将①式改写成如下：

$$
minf(x)\quad s.t\quad h(x) + \alpha^2 =0 \quad ---式②
$$

以上②式变成了等式条件约束优化问题，我们就可以使用拉格朗日乘数法来求解满足等式条件下$f(x)$最小值对应的x值。

通过拉格朗日乘数法，将②式转换为如下：

$$
L(x,\lambda,\alpha) = f(x) + \lambda(h(x) + \alpha^2)
$$

**但是上式中**$\lambda$**值必须满足**$\lambda \ge 0$**，由于**$L(x,\lambda,\alpha)$**变成了有条件的拉格朗日函数，这里需要求**$minL(x,\lambda,\alpha)$ **对应下的x值** 。至于为什么$\lambda \ge 0$及为什么计算的是$minL(x,\lambda,\alpha)$下对应的x值，这里证明涉及到很多几何性质，这里也不再证明。参考文章：[KKT不等式约束](https://blog.csdn.net/Soft_Po/article/details/118358564)

所以将上式转换成如下拉格朗日函数：

$$
minL(x,\lambda,\alpha) = f(x) + \lambda(h(x) + \alpha^2) \quad \lambda \ge 0\quad---式③
$$

以上我们可以看到将不等式条件约束优化问题加入“松弛变量”转换成等式条件约束优化问题，使用拉格朗日乘数法进行转换得到$L(x,\lambda,\alpha)$，我们先抛开min，下面我们可以对拉格朗日函数$L(x,\lambda,\alpha)$对各个未知数$x,\lambda,\alpha$进行求导，让对应的导数为零，得到以下联立方程：

$$
\left \{\begin{aligned}&\frac{\partial L}{\partial x} = \frac{\partial f(x)}{\partial x} + \lambda \frac{\partial h(x)}{\partial x} = 0\quad ---式④\\\\&\frac{\partial L}{\partial \lambda} = h(x) + \alpha^2 = 0\quad ---式⑤\\\\&\frac{\partial L}{\partial \alpha} = 2\lambda\alpha = 0\quad---式⑥\\\\&\lambda \ge 0\quad ---式⑦ \end{aligned}\right.
$$

从上式⑥式可知$\lambda\alpha=0$，我们分两种情况讨论：

1) $\lambda =0,\alpha \neq 0$

由于$\lambda = 0$，根据③式可知，**约束不起作用**，根据①式可知$h(x) \le 0$。

2) $\lambda \neq 0,\alpha =0$

由于$\lambda \neq 0$，根据③式可知$\lambda > 0$。由于$\alpha =0$，约束条件起作用，根据⑤式可知，$h(x) = 0$。

综上两个步骤我们可以得到$\lambda \cdot h(x) = 0$，且在**约束条件起作用**时，$\lambda >0,h(x) = 0$；约束条件不起作用时，$\lambda =0,h(x) \le 0$。

上面方程组中的⑥式可以改写成$\lambda\cdot h(x) = 0$。由于$\alpha^2 \ge 0$，所以将⑤式也可以改写成$h(x) \le 0$，所以上面方程组也可以转换成如下：

$$
\left \{\begin{aligned}&\frac{\partial L}{\partial x} = \frac{\partial f(x)}{\partial x} + \lambda \frac{\partial h(x)}{\partial x} = 0\\\\&h(x) \le 0\\\\&\lambda \cdot h(x) =0\\\\&\lambda \ge 0 \end{aligned}\right.
$$

**以上便是不等式约束优化问题的KKT（Karush-Kuhn-Tucker）条件** ，我们回到最开始要处理的问题上，根据③式可知，我们需要找到合适的$x,\lambda,\alpha$值使$L(x,\lambda,\alpha)$最小，但是合适的$x,\lambda,\alpha$必须满足KKT条件。

我们对③式的优化问题可以进行优化如下：

$$
minL(x,\lambda,\alpha) = f(x) + \lambda(h(x) + \alpha^2) \quad \lambda \ge 0\quad---式③
$$

$$
minL(x,\lambda,\alpha) = f(x) +\lambda \cdot h(x) +\lambda \alpha^2
$$

由于$\alpha^2 \ge 0$且$\lambda \ge 0$，$\lambda \alpha^2$一定大于零，所以进一步可以简化为如下：

$$
minL(x,\lambda) = f(x) + \lambda h(x) \quad \quad---式⑧
$$

满足最小化⑧式对应的$x,\lambda$值一定也要满足KKT条件，假设现在我们找到了合适的参数$x$值使$f(x)$取得最小值P【注意：这里根据①式来说，计算$f(x)$**的最小值，这里假设合适参数**$x$**值对应的**$f(x)$ **的值P就是最小值，不存在比这更小的值。**  **】** ，由于⑧式中$\lambda h(x)$一定小于等于零，所以一定有：

$$
L(x,\lambda) = f(x) + \lambda h(x)\le P
$$

也就是说一定有：

$$
L(x,\lambda) \le P
$$

为了找到最优的$\lambda$值，我们一定想要$L(x,\lambda)$接近P，即找到合适的$\lambda$最大化$L(x,\lambda)$，可以写成$\max\limits_{\lambda} L(x,\lambda)$，所以⑧式求解合适的$x,\lambda$值最终可以转换成如下：

$$
\min\limits_{x}\max\limits_{\lambda} L(x,\lambda)\quad s.t\quad \lambda \ge 0\quad ---式⑨
$$

## 3.3、对偶问题

对偶问题是我们定义的一种问题，对于一个不等式约束的原问题：

$$
\min\limits_x\max\limits_{\lambda}L(x,\lambda)\quad s.t\quad\lambda \ge 0
$$

我们定义**对偶问题**为（对上面方程的求解等效求解下面方程）：

$$
\max\limits_{\lambda}\min\limits_{x}L(x,\lambda)\quad s.t\quad\lambda \ge 0
$$

其实就是把min和max对调了一下，当然对应的变量也要变换。

对偶问题有什么好处呢？对于原问题，我们要先求里面的max，再求外面的min。而对于对偶问题，我们可以先求里面的min。有时候，先确定里面关于x的函数最小值，比原问题先求解关于$\lambda$的最大值，要更加容易解。

但是原问题跟对偶问题**并不是等价**的，这里有一个强对偶性、弱对偶性的概念，弱对偶性是对于所有的对偶问题都有的一个性质。

这里给出一个弱对偶性的推导过程:

$$
\max\limits_{\lambda}\min\limits_{x}L(x,\lambda) = \min\limits_xL(x,\lambda^*) \le L(x^*,\lambda^*) \le \max\limits_{\lambda}L(x^*,\lambda)=\min\limits_x\max\limits_{\lambda}L(x,\lambda)
$$

其中 $x^*,\lambda^*$是函数取最大值最小值的时候对应的最优解，也就是说，原问题始终大于等于对偶问题:

$$
\max\limits_{\lambda}\min\limits_{x}L(x,\lambda) \le \min\limits_x\max\limits_{\lambda}L(x,\lambda)
$$

如果两个问题是强对偶的，那么这两个问题其实是等价的问题:

$$
\max\limits_{\lambda}\min\limits_{x}L(x,\lambda) = \min\limits_x\max\limits_{\lambda}L(x,\lambda)
$$

所有的下凸函数都满足强对偶性，也就是说所有下凸函数都满足上式。

【备注】：对弱对偶性推导的理解如下图所示：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/71ae07f2f4fa481ebe94f9f15c6409f5.png)

# 4、目标函数优化【硬间隔】

通过2小节【构建SVM目标函数】我们知道SVM目标函数如下：

$$
min\frac{1}{2}||w||\quad s.t\quad y\cdot (w^T\cdot x_i +b) \ge 1,\quad i = 1,2,\cdots,n
$$

根据拉格朗日乘数法、KKT条件、对偶问题我们可以按照如下步骤来计算SVM目标函数最优的一组w值。

## 4.1、构造拉格朗日函数

将SVM目标函数转换为如下:

$$
min\frac{1}{2}||w||\quad s.t\quad 1-y\cdot (w^T\cdot x_i +b) \ge 0,\quad i = 1,2,\cdots,n
$$

根据3.2【KKT条件】中的⑨式构建拉格朗日函数如下：

$$
\min\limits_{w,b}\max\limits_{\lambda}L(w,b,\lambda) = \frac{1}{2}||w||^2 + \sum\limits_{i=1}^n\lambda_i[1-y_i(w^T\cdot x_i +b)] \quad s.t \quad \lambda_i \ge 0\quad ---式a
$$

以上不等式转换成拉格朗日函数必须满足**KKT条件**，详见3.2【KKT条件】，这里满足的KKT条件如下：

$$
\left\{\begin{aligned}&1-y_i(w^T\cdot x_i + b) \le 0\\\\&\lambda (1-y_i(w^T\cdot x_i + b)) = 0\\\\&\lambda \ge 0\end{aligned}\right.
$$

## 4.2、对偶转换

由于原始目标函数$\frac{1}{2}||w||$是个下凸函数，根据3.3【对偶问题】可知$L(w,b,\lambda)$一定是强对偶问题，所以可以将a式改写成如下：

$$
\max\limits_{\lambda}\min\limits_{w,b}L(w,b,\lambda) = \frac{1}{2}||w||^2 + \sum\limits_{i=1}^n\lambda_i[1-y_i(w^T\cdot x_i +b)] \quad s.t \quad \lambda_i \ge 0\quad ---式b
$$

针对b式，我们假设参数$\lambda$固定，使$L(w,b,\lambda)$对参数w和b进行求导得到如下：

$$
\left\{\begin{aligned}&\frac{\partial L}{\partial w}=w - \sum\limits_{i=1}^n\lambda_i\cdot x_i\cdot y_i = 0\\\\&\frac{\partial L}{\partial b}=\sum\limits_{i=1}^n\lambda_i\cdot y_i = 0\end{aligned}\right.
$$

进一步可以得到：

$$
\left\{\begin{aligned}&\sum\limits_{i=1}^n\lambda_i\cdot x_i\cdot y_i = w\\\\&\sum\limits_{i=1}^n\lambda_i\cdot y_i = 0\end{aligned}\right.\quad --- 式c
$$

按照解方程组的思想，我们现在将以上计算得到的结果代入到b式中得到：

$$
\begin{aligned}&\max\limits_{\lambda}L(w,b,\lambda) = \frac{1}{2}||w||^2 +\sum\limits_{i=1}^n\lambda_i[1 - y_i(w^T\cdot x_i + b)]\\\\&=\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_j y_i y_j(x_i\cdot x_j) +\sum\limits_{i=1}^n\lambda_i -\sum\limits_{i=1}^n\lambda_iy_i(\sum\limits_{j=1}^n\lambda_jx_jy_j\cdot x_i +b)\\\\&=\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_j y_i y_j(x_i\cdot x_j) +\sum\limits_{i=1}^n\lambda_i -\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_iy_i\lambda_jy_j(x_i\cdot x_j)-\sum\limits_{i=1}^n\lambda_i y_i b\\\\&=\sum\limits_{i=1}^n\lambda_i-\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_j y_i y_j(x_i\cdot x_j)\quad s.t \quad \sum\limits_{i=1}^n\lambda_iy_i = 0,\lambda_i \ge 0\end{aligned}
$$

即：

$$
\begin{aligned}&\max\limits_{\lambda}L(w,b,\lambda) = \sum\limits_{i=1}^n\lambda_i-\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_j y_i y_j(x_i\cdot x_j)\quad---式d\\\\& s.t \quad \sum\limits_{i=1}^n\lambda_iy_i = 0,\lambda_i \ge 0\end{aligned}
$$

我们可以发现以上公式经过转换只含有$\lambda$的方程求极值问题，但是$\lambda$含有$\lambda_i,\lambda_j,\cdots$这些未知数（注意不是2个），那么我们要求的一组w如何得到呢？只要针对以上公式得到$\lambda$值后，我们可以根据c式进一步求解到对应的一组w值。针对上式求极值问题，我们常用 SMO(Sequential Minimal Optimization) 算法求解$\lambda_i,\lambda_j,\cdots$。

注意：d式中$(x_i\cdot x_j)$代表两个向量的点积，也叫内积，例如：$x_i = (x_{i1},x_{i2}),x_j = (x_{j1},x_{j2})$那么两个向量点积结果为$x_{i1} * x_{j1} + x_{i2} * x_{j2}$。

## 4.3、SMO算法求解（了解）

SMO(Sequential Minimal Optimization)，序列最小优化算法，其核心思想非常简单：每次只优化一个参数，其他参数先固定住，仅求当前这个优化参数的极值。下面我们只说SVM中如何利用SMO算法进行求解最终结果的，这里不再进行公式推导。

$$
\begin{aligned}&\max\limits_{\lambda}L(w,b,\lambda) = \sum\limits_{i=1}^n\lambda_i-\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_j y_i y_j(x_i\cdot x_j)\quad---式d\\\\& s.t \quad \sum\limits_{i=1}^n\lambda_iy_i = 0,\lambda_i \ge 0\end{aligned}
$$

我们根据d式可以看到有 $\lambda_i,\lambda_j,i = 1,2,\cdots,n,j = 1,2,\cdots,n$ 很多参数，由于优化目标中含有约束条件：$\sum\limits_{i=1}^n\lambda_iy_i = 0$，此约束条件中$i = 1,2,\cdots,n$，$y_i$代表每个样本所属类别-1或者1。

根据SMO算法假设将$\lambda_3,\lambda_4,\cdots,\lambda_n$参数固定，那么$\lambda_1,\lambda_2$之间的关系也确定了，可以将约束条件：$\sum\limits_{i=1}^n\lambda_iy_i = 0$改写成

$$
\lambda_1y_ + \lambda_2y_2 +c = 0 \quad ---式e
$$

我们可以将上式中c看成是$\lambda_3,\lambda_4,\cdots,\lambda_n$乘以对应的类别$y$得到的常数，根据e式我们发现$\lambda_1,\lambda_2$之间有等式关系，即：$\lambda_2 = \frac{-c - \lambda_1 y_1}{y_2}$，也就是说我们可以使用$\lambda_1$的表达式代替$\lambda_2$，这样我们就可以将d式看成只含有$\lambda_1 \ge 0$这一个条件下最优化问题，并且我们可以将$\lambda_2 = \frac{-c - \lambda_1 y_1}{y_2}$代入到d式，d式中只含有$\lambda_1$一个未知数求极值问题，可以对函数进行求导，令导数为零，得到$\lambda_1$的值，然后根据$\lambda_1$的值计算出$\lambda_2$的值。通过以上固定参数方式多次迭代计算得到一组$\lambda$值。

然后对其他的$\lambda$组【$\lambda_3,\lambda_4$】，进行相同操作，多次执行，即可得到全部$\lambda$，for循环迭代优化，即可！

## 4.4、计算分割线w和b的值

根据4.2中的c式，如下：

$$
\left\{\begin{aligned}&\sum\limits_{i=1}^n\lambda_i\cdot x_i\cdot y_i = w\\\\&\sum\limits_{i=1}^n\lambda_i\cdot y_i = 0\end{aligned}\right.\quad --- 式c
$$

我们可以知道获取了一组$\lambda$的值，我们可以得到一组w对应的值。根据4.1中的KKT条件：

$$
\left\{\begin{aligned}&1-y_i(w^T\cdot x_i + b) \le 0\\\\&\lambda (1-y_i(w^T\cdot x_i + b)) = 0\quad ---式f\\\\&\lambda \ge 0\end{aligned}\right.
$$

根据f式中式可知，当$\lambda$时，一定有$ 1-y_i(w^T\cdot x_i + b) = 0$，对于这样的样本就属于支持向量点，就在分类边界上，我们将对应的w代入到下式中

$$
y_i(w^T\cdot x_i + b) = 1 \quad ---式g
$$

一定可以计算出b的值。我们将g式两边乘以$y_i$，得到

$$
y_i^2(w^T\cdot x_i + b) = y_i
$$

由于$y_i^2 = 1$，所以得到最终b的结果如下：

$$
b = y_i - w^T\cdot x_i
$$

假设我们有S个支持向量，以上b的计算可以使用任意一个支持向量代入计算即可，如果数据严格是线性可分的，这些b结果是一致的。对于数据不是严格线性可分的情况，参照后面的软间隔问题，一般我们可以采用一种更健壮的方式，将所有支持向量对应的b都求出，然后将其平均值作为最后的结果，最终b的结果如下：

$$
b = \frac{1}{S}\sum\limits_{i \in S}(y_i - w^T \cdot x_i)
$$

S代表支持向量点的集合，也就是在边界上点的集合。

综上所述，我们可以的到最终的w和b的值如下：

$$
\left\{\begin{aligned}&w = \sum\limits_{i=1}^n\lambda_i\cdot x_i\cdot y_i \\\\&b = \frac{1}{S}\sum\limits_{i \in S}(y_i - w^T \cdot x_i)\end{aligned}\right.
$$

确定w和b之后，我们就能构造出最大分割超平面：$w^T\cdot x + b = 0$，新来样本对应的特征代入后得到的结果如果是大于0那么属于一类，小于0属于另一类。

# 5、软间隔及优化

## 5.1、软间隔问题

以上讨论问题都是基于样本点完全的线性可分，我们称为硬间隔如下图：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/56ae758b4dbb48e1822c9f61df5b5cb7.png)

如果存在部分样本点不能完全线性可分，大部分样本点线性可分的情况，如下图所示：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/a97df4c6260c4a2f8620b212e81838af.png)

那么我们就需要用到软间隔，相比于硬间隔的苛刻条件，我们允许个别样本点出现在间隔带里面，比如：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/aa7255cfc7bd4991a02e78181c61616f.png)

即我们允许部分样本点不满足约束条件：

$$
y\cdot (w^T\cdot x_i +b) \ge 1
$$

为了度量这个间隔软到何种程度，我们为每个样本引入一个“ **松弛变量** ”$\xi_i,\xi_i \ge 0$，（中文：克西）加入松弛变量后，我们的约束条件变成 $ y\cdot (w^T\cdot x_i +b) \ge 1 -\xi_i$，这样不等式就不会那么严格，数据点在不同的约束条件下的情况如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/337c40471a0c4d09a60429590053a79d.png)

内部点：$y\cdot (w^T\cdot x_i +b) \ge 1$,即 $\xi_i = 0$

边界点：$y\cdot (w^T\cdot x_i +b) =1$,即$\xi_i = 0$

正确误差点：$y\cdot (w^T\cdot x_i +b) = 1-\xi_i$,即$ 0 &#x3c; \xi_i &#x3c;1$

错误误差点：$y\cdot (w^T\cdot x_i +b) = 1-\xi_i$,即$\xi_i \ge 1$

## 5.2、优化SVM目标函数（了解）

加入软间隔后我们的目标函数变成了：

$$
min\frac{1}{2}||w||^2 + C\sum\limits_{i=1}^n\xi_i\quad s.t\quad 1-y(w^T\cdot x_i +b) - \xi_i \le 0,\xi_i\ge0,i = 1,2,\cdots,n\quad ---式h
$$

其中C是一个大于0的常数，这里在原有的目标函数中加入$C\sum\limits_{i=1}^n\xi_i$可以理解为错误样本的惩罚程度，我们希望在对应条件下$ min\frac{1}{2}||w||^2 + C\sum\limits_{i=1}^n\xi_i$变小，对应的也是希望$ C\sum\limits_{i=1}^n\xi_i$变小，当C为无穷大，$\xi_i$必然无穷小，这样的话SVM就变成一个完全线性可分的SVM。如果C为有对应的值时，$\xi_i$对应的会有一个大于0的值，这样的SVM就是允许部分样本不遵守约束条件。

接下来我们对新的目标函数h式求解最优化问题，步骤与 4、最小化SVM目标函数【硬间隔】完全线性可分步骤完全一样。

1) **构造拉格朗日函数**

将h式目标函数改写成如下：

$$
min\frac{1}{2}||w||^2 + C\sum\limits_{i=1}^n\xi_i\quad s.t\quad 1-y(w^T\cdot x_i +b) - \xi_i \le 0,\xi_i\ge0,i = 1,2,\cdots,n\quad ---式h
$$

构造拉格朗日函数如下：

$$
\begin{aligned}\min\limits_{w,b,\xi}\max\limits_{\lambda,\mu}L(w,b,\xi,\lambda,\mu) =&min\frac{1}{2}||w||^2 + C\sum\limits_{i=1}^n\xi_i+\sum\limits_{i=1}^n\lambda_i[1-y_i(w^T\cdot x_i +b)-\xi_i] - \sum\limits_{i=1}^n\mu_i\xi_i\\\\&s.t\quad \lambda_i \ge 0,\mu_i \ge 0\quad i=1,2,\cdots,n\quad ---式i \end{aligned}
$$

上式中$\lambda_i,\mu_i$是拉格朗日乘子，$w,b,\xi$是我们要计算的主问题参数。

2) **对偶关系转换**

目标函数是个下凸函数，对应的拉格朗日函数复合强对偶性，所以可以根据强对偶向，将对偶问题转换为：

$$
\min\limits_{w,b,\xi}\max\limits_{\lambda,\mu}L(w,b,\xi,\lambda,\mu) = \max\limits_{\lambda,\mu}\min\limits_{w,b,\xi}L(w,b,\xi,\lambda,\mu)
$$

即：

$$
\begin{aligned}\max\limits_{\lambda,\mu}\min\limits_{w,b,\xi}L(w,b,\xi,\lambda,\mu) =&\frac{1}{2}||w||^2 + C\sum\limits_{i=1}^n\xi_i+\sum\limits_{i=1}^n\lambda_i[1-y_i(w^T\cdot x_i +b)-\xi_i] - \sum\limits_{i=1}^n\mu_i\xi_i\\\\&s.t\quad \lambda_i \ge 0,\mu_i \ge 0\quad i=1,2,\cdots,n\quad ---式j \end{aligned}
$$

使用j式对$w,b,\xi$求导，并令导数为零，得到如下关系：

$$
\left\{\begin{aligned}&\frac{\partial L}{\partial w}=w - \sum\limits_{i=1}^n\lambda_i\cdot x_i\cdot y_i = 0\\\\&\frac{\partial L}{\partial b}=\sum\limits_{i=1}^n\lambda_i\cdot y_i = 0\\\\&\frac{\partial L}{\partial \xi} = C\sum\limits_{i=1}^n1 - \sum\limits_{i=1}^n\lambda_i-\sum\limits_{i=1}^n\mu_i = 0\end{aligned}\right.
$$

整理之后得到：

$$
\left\{\begin{aligned}&w = \sum\limits_{i=1}^n\lambda_i\cdot x_i\cdot y_i\\\\&0=\sum\limits_{i=1}^n\lambda_i\cdot y_i \\\\&C =  \lambda_i+\mu_i\end{aligned}\right.
$$

将以上关系代入到 j 式：

$$
\begin{aligned}&\max\limits_{\lambda,\mu}L(w,b,\xi,\lambda,\mu) =\frac{1}{2}||w||^2 + C\sum\limits_{i=1}^n\xi_i+\sum\limits_{i=1}^n\lambda_i[1-y_i(w^T\cdot x_i +b)-\xi_i] - \sum\limits_{i=1}^n\mu_i\xi_i\\\\&=\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_j y_i y_j(x_i\cdot x_j) +\sum\limits_{i=1}^n\lambda_i\xi_i +\sum\limits_{i=1}^n\mu_i\xi_i +\sum\limits_{i=1}^n\lambda_i-\sum\limits_{i=1}^n\lambda_iy_i(\sum\limits_{j=1}^n\lambda_jx_jy_j\cdot x_i +b)-\sum\limits_{i=1}^n\lambda_i\xi_i-\sum\limits_{i=1}^n\mu_i\xi_i\\\\&=\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_j y_i y_j(x_i\cdot x_j) +\sum\limits_{i=1}^n\lambda_i -\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_iy_i\lambda_jy_j(x_i\cdot x_j)\\\\&=\sum\limits_{i=1}^n\lambda_i -\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_iy_i\lambda_jy_j(x_i\cdot x_j)\\\\&\quad  \quad s.t\quad \lambda_i\ge0,\mu_i \ge 0\quad i = 1,2,\cdots,n\quad C - \lambda_i-\mu_i =0 \quad ---式\ k\end{aligned}
$$

我们发现上式中求$\max\limits_{\lambda,\mu}L(w,b,\xi,\lambda,\mu)$与$\mu$没有关系，但是为什么可以消掉是因为$C = \lambda_i + \mu_i$，所以在约束条件中一定有此条件，所以我们得到最终的目标函数可以写成如下：

$$
\begin{aligned}\max\limits_{\lambda}&L(w,b,\xi,\lambda,\mu) = \max\limits_\lambda[\sum\limits_{i=1}^n\lambda_i -\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_jy_iy_j(x_i\cdot x_j)]\\\\&s.t\quad \lambda_i\ge0,\mu_i \ge 0\quad i = 1,2,\cdots,n\quad C - \lambda_i-\mu_i =0 \quad ---式\ L\end{aligned}
$$

推导到以上之后，我们发现结果和硬间隔结果一样，参照4.3 d式，只是结果中多了约束条件$C = \lambda_i + \mu_i$。

3.**求解结果**

同样我们也可以利用SMO算法对式**L**求解得到一组合适的$\lambda$值和$\mu$值，由于综上所述，我们可以的到最终的w和b的值如下：

$$
\left\{\begin{aligned}&w = \sum\limits_{i=1}^n\lambda_i\cdot x_i\cdot y_i \\\\&b = \frac{1}{S}\sum\limits_{i \in S}(y_i - w^T \cdot x_i)\end{aligned}\right.
$$

这里得到的w和b虽然和硬分割结果一样，但是这是加入松弛变量之后得到的w和b的值，根据**L**式的条件$C = \lambda_i + \mu_i$和 $\lambda_i \ge 0,\mu_i \ge 0$，结合h式，C越大，必然导致松弛越小，如果C无穷大，那么就意味着模型过拟合，在训练SVM时，C是我们需要调节的参数。

那么确定w和b之后，我们就能构造出最大分割超平面：$w^T\cdot x +b = 0$，新来样本对应的特征代入后得到的结果如果是大于0那么属于一类，小于0属于另一类。

## 5.3、SVM代码演练

以乳腺癌为例，使用SVM进行建模训练

```python
from sklearn.svm import SVC
import numpy as np
from sklearn import datasets
from sklearn.model_selection import train_test_split
X,y = datasets.load_breast_cancer(return_X_y=True)

score = 0
for i in range(100):
    X_train,X_test,y_train,y_test = train_test_split(X,y)
    model = SVC(C = 1.0)
    model.fit(X_train,y_train)
    score += model.score(X_test,y_test)
  
print('模型平均得分是：',score)
```

## 5.4、网格搜索最佳参数

![Grid Search Workflow](https://scikit-learn.org/stable/_images/grid_search_workflow.png)

网格搜索就是手动给定模型中想要改动的所有参数，程序自动使用穷举法来将所有的参数或者参数组合都运行一遍，将对应得到的模型做K折交叉验证，将得分最高的参数或参数组合当做最佳结果，并得到模型。

![../_images/grid_search_cross_validation.png](https://scikit-learn.org/stable/_images/grid_search_cross_validation.png)

也就是说网格搜索用来获取最合适的一组参数，不需要我们手动一个个尝试。使用网格搜索由于针对每个参数下的模型需要做K折交叉验证最终导致加大了模型训练时间，所以一般我们可以在小的训练集中使用网格搜索找到对应合适的参数值，再将合适的参数使用到大量数据集中训练模型。例如:逻辑回归中使用正则项时，我们可以先用一小部分训练集确定合适的惩罚系数，然后将惩罚系数应用到大量训练集中训练模型。

```python
from sklearn.svm import SVC
import numpy as np
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.model_selection import GridSearchCV
import pandas as pd
X,y = datasets.load_breast_cancer(return_X_y=True)

score = 0
search_space = {'C': np.logspace(-3, 3, 7)}
best_params = []
for i in range(100):
    X_train,X_test,y_train,y_test = train_test_split(X,y)
    model = SVC()
    gc = GridSearchCV(model, param_grid=search_space,cv=5)
    gc.fit(X_train,y_train)
    best_params.append(gc.best_params_['C'])
    best_model = gc.best_estimator_
    best_model.fit(X_train,y_train)
    score += best_model.score(X_test,y_test)
  
print('模型平均得分是：',score)

# 查看最佳参数
pd.value_counts(best_params)
```

# 6、SVM Hinge Loss（了解）

软间隔情况下，SVM的目标函数如下：

$$
min\frac{1}{2}||w||^2 + C\sum\limits_{i=1}^n\xi_i\quad s.t\quad 1-y(w^T\cdot x_i +b) - \xi_i \le 0,\xi_i\ge0,i = 1,2,\cdots,n\quad ---式h
$$

其实这里我们说SVM的损失主要说的就是上式中的松弛变量$\xi_i$损失，当所有样本总的$\xi_i$为0时那么就是一个完全线性可分的SVM；如果SVM不是线性可分，那么我们希望总体样本的松弛变量$\sum\limits_{i=1}^n\xi_i$越小越好。

根据h式的条件$ 1-y(w^T\cdot x_i +b) - \xi_i \le 0,\xi_i \ge 0$我们可以改写成如下：

$$
\xi_i \ge 1-y(w^T\cdot x_i +b)  ,\xi_i\ge 0
$$

根据下图结合以上两个条件我们可以继续改写成如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/a9fd5ff04b084f359867faa9e334c655.png)

$$
\xi_i = max(0,\ 1-y(w^T\cdot x_i +b)) \quad --- 式m
$$

上式等价于:

$$
\xi_i = \left\{\begin{aligned}&0\quad y (w^T\cdot x_i +b ) \ge 1 \\\\& 1-y (w^T\cdot x_i +b )\quad y (w^T\cdot x_i +b ) < 1\end{aligned}\right.
$$

以上公式就是SVM的损失函数，称为“Hinge Loss”，也被叫做“合页损失函数”。其图像如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/46c1e0dff7e240f9a69d3925f8cfd3b1.png)

特点是当$y(w^T\cdot x_i +b) \ge 1$时损失为0。

我们可以将m式代入到h式得到新的目标函数，暂时先不关注约束条件，新的目标函数如下：

$$
min\frac{1}{2}||w||^2 + C\sum\limits_{i=1}^nmax(0,\ 1-y(w^T\cdot x_i +b))
$$

针对新的目标函数我们同样也是最小化整体目标函数。我们发现以上目标函数与逻辑回归L2正则化的损失函数在形式上非常类似，将上式可以改写成如下：

$$
min\frac{1}{2}||w||^2 + LOSS
$$

逻辑回归L2正则化损失函数如下：

$$
J(\theta) = -\sum\limits_{i = 1}^n[y^{(i)}\ln(h_{\theta}(x^{(i)})) + (1-y^{(i)})\ln(1-h_{\theta}(x^{(i)}))] + \lambda \sum\limits_{i=1}^nw_i^2
$$

也可以改写成如下：

$$
J(\theta) = LOSS + \lambda \sum\limits_{i=1}^nw_i^2
$$

如果将逻辑损失函数图像与“Hinge Loss”图像放在一起图像如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/8f759fbac05648e8a1fac4a17403f9d5.png)

通过上图我们可以看出，在训练逻辑回归时，损失永远不会为0，因此即使分类正确，逻辑回归仍然会不停的训练，以求得更小的损失和更大的概率，但是极有可能出现过拟合，这就是为什么我们需要在梯度下降中提前终止的原因。但是在SVM中，如果分类正确可以直接达到$\sum\limits_{i=1}^n\xi_i =0$，不再进行对分类无益的训练，即学习到一定程度就不再学习。

# 7、核函数

## 7.1、非线性核函数

在前面SVM讨论的硬间隔和软间隔都是指的样本完全可分或者大部分样本点线性可分的情况。那么如果样本完全线性不可分如下图所示：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/6c758152f42a42cd9912d7b3b373390d.png)

我们可以使用升维方式来解决这种线性不可分的问题，例如目前只有两个维度$x_1,x_2$，我们可以基于已有的维度进行升维得到更多维度，例如：$x_1,x_2,x_1x_2$，这样我们可以将以上问题可以转换成高维可分问题，如下图：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/c76e89ac79fa4804b8b04a3c40e6f338.png)

SVM中线性不可分问题就是非线性SVM问题，对这种问题我们可以将样本映射到高维空间，再通过间隔最大化的方式学习得到支持向量机。

## 7.2、核函数（kernel function）

假设原有特征有如下三个$x_1,x_2,x_3$，那么基于三个已有特征进行两两组合（二阶交叉）我们可以得到如下更多特征：

$$
[x_1^2、x_1x_2、x_1x_3、x_2^2、x_2x_1、x_2x_3、x_3^2、x_3x_1、x_3x_2]
$$

我们在训练模型时将以上组合特征可以参与到模型的训练中，其代价是运算量过大，比如原始特征有m个，那么二阶交叉的维度为$C_m^2$个，这种量级在实际应用中很难实现，那么有没有一种方式可以在升维的同时可以有效的降低运算量，在非线性SVM中，核函数就可以实现在升维的同时可以有效降低运算量，那么核函数是什么？

有两个向量$m = [m_1,m_2,m_3]^T,n = [n_1,n_2,n_3]^T$，我们分别对两个向量进行相同升维分别记为$\phi m,\phi n$：

$$
\begin{aligned}&\phi m = [m_1^2、m_1m_2、m_1m_3、m_2^2、m_2m_1、m_2m_3、m_3^2、m_3m_1、m_3m_2]^T\\\\&\phi n = [n_1^2、n_1n_2、n_1n_3、n_2^2、n_2n_1、n_2n_3、n_3^2、n_3n_1、n_3n_2]^T\end{aligned}
$$

之后计算两者内积如下：

$$
\begin{aligned}\phi m\cdot \phi n &= m_1^2n_1^2+m_1m_2n_1n_2+m_1m_3n_1n_3+m_2^2n_2^2+m_2m_1n_2n_1\\\\&+m_2m_3n_2n_3+m_3^2n_3^2+m_3m_1n_3n_1+m_3m_2n_3n_2\\\\&=m_1^2n_1^2 + m_2^2n_2^2+m_3^2n_3^2 + 2m_1m_2n_1n_2 + 2m_1m_3n_1n_3 + 2m_2m_3n_2n_3\quad ---式o\end{aligned}
$$

另外我们计算m和n的内积如下：

$$
m\cdot n = m_1n_1 + m_2n_2 + m_3n_3
$$

对结果进行平方得到：

$$
\begin{aligned}(m\cdot n)^2 &= (m_1n_1 + m_2n_2 + m_3n_3)^2\\\\&=m_1^2n_1^2 + m_2^2n_2^2+m_3^2n_3^2 + 2m_1m_2n_1n_2 + 2m_1m_3n_1n_3 + 2m_2m_3n_2n_3\quad ---式p\end{aligned}
$$

【备注】：$(a + b + c)^2 = a^2 + b^2 + c^2 + 2ab + 2ac + 2bc$

我们发现o式和p式结果一样，也就是

$$
\phi m\cdot \phi n = (m\cdot n)^2 \quad ---式q
$$

以上在m和n向量中只有3个元素时推导成立，那么在m和n中如果有多个元素时也一样成立。

我们发现，如果最终目标只是计算向量升维后的内积$\phi m\cdot \phi n$，根据q式可知，我们可以先计算原始向量的内积$m\cdot n$，再进行平方运算，而不需要真正的进行高运算量的升维操作。这种通过两个低维向量进行数学运算得到它们投影至高维空间时向量的内积方法叫做核方法，相应的算法叫做核函数，上面案例核函数记为：

$$
K(m\cdot n) = \phi m\cdot \phi n = (m\cdot n)^2
$$

以上核函数叫做多项式核函数，常用的核函数有如下几种，不同的核函数的区别是将向量映射到高维空间时采用的升维方法不同，不过高维向量都是不需要计算出来的。

* 线性核函数：

$$
K(x_i\cdot x_j) = x_i^T\cdot x_j
$$

* 多项式核函数：

$$
K(x_i\cdot x_j) = (\alpha(x_i^T\cdot x_j) + c) ^d\quad \alpha,c,d都是参数
$$

* 高斯核函数：

$$
K(x_i\cdot x_j) = e^{-\frac{||x_i - x_j||^2}{2\delta^2}}
$$

## 7.3、核函数在SVM中应用

在非线性SVM中，我们可以对原始特征进行升维，原有的超平面$w^T\cdot x +b = 0$变成了$w^T \cdot \phi x + b = 0$，根据 L 式SVM目标函数：

$$
\begin{aligned}\max\limits_{\lambda}&L(w,b,\xi,\lambda,\mu) = \max\limits_\lambda[\sum\limits_{i=1}^n\lambda_i -\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_jy_iy_j(x_i\cdot x_j)]\\\\&s.t\quad \lambda_i\ge0,\mu_i \ge 0\quad i = 1,2,\cdots,n\quad C - \lambda_i-\mu_i =0 \quad ---式\ L\end{aligned}
$$

可知最终经过对偶处理后的目标函数如下：

$$
\begin{aligned}\max\limits_{\lambda}&L(w,b,\xi,\lambda,\mu) = \max\limits_\lambda[\sum\limits_{i=1}^n\lambda_i -\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_jy_iy_j(\phi x_i\cdot \phi x_j)]\\\\&s.t\quad \lambda_i\ge0,\mu_i \ge 0\quad i = 1,2,\cdots,n\quad C - \lambda_i-\mu_i =0 \end{aligned}
$$

以上公式与l式不同的是只是将$x_i\cdot x_j$转换成了$\phi x_i \cdot \phi x_j$，这样我们可以将上式进而转换成下式：

$$
\begin{aligned}\max\limits_{\lambda}&L(w,b,\xi,\lambda,\mu) = \max\limits_\lambda[\sum\limits_{i=1}^n\lambda_i -\frac{1}{2}\sum\limits_{i=1}^n\sum\limits_{j=1}^n\lambda_i\lambda_jy_iy_j(x_i\cdot  x_j)^2]\\\\&s.t\quad \lambda_i\ge0,\mu_i \ge 0\quad i = 1,2,\cdots,n\quad C - \lambda_i-\mu_i =0 \quad ---式r\end{aligned}
$$

这样我们可以通过核函数将数据映射到高维空间，但是核函数是在低维上进行计算，而将实质上的分类效果表现在了高维空间上，来解决在原始空间中线性不可分的问题。

在SVM中我们可以使用常用的核函数：线性核函数、多项式核函数、高斯核函数来进行升维解决线性不可分问题，应用最广泛的就是高斯核函数，无论是小样本还是大样本、高维或者低维，高斯核函数均适用，它相比于线性核函数来说，线性核函数只是高斯核函数的一个特例，线性核函数适用于数据本身线性可分，通过线性核函数来达到更理想情况。高斯核函数相比于多项式核函数，多项式核函数阶数比较高时，内部计算时会导致一些元素值无穷大或者无穷小，而在高斯核函数会减少数值的计算困难。

综上，我们在非线性SVM中使用核函数时，先使用线性核函数，如果不行尝试换不同的特征，如果还不行那么可以直接使用高斯核函数。

非线性SVM使用核函数代码如下：

```python
from sklearn import svm
from sklearn import datasets
from sklearn.model_selection import train_test_split as ts

X,y = datasets.load_wine(return_X_y=True)

#split the data to  7:3
X_train,X_test,y_train,y_test = ts(X,y,test_size=0.3)
print(y_test)
# select different type of kernel function and compare the score

# kernel = 'rbf'
clf_rbf = svm.SVC(kernel='rbf')
clf_rbf.fit(X_train,y_train)
score_rbf = clf_rbf.score(X_test,y_test)
print("The score of rbf is : %f"%score_rbf)

# kernel = 'linear'
clf_linear = svm.SVC(kernel='linear')
clf_linear.fit(X_train,y_train)
score_linear = clf_linear.score(X_test,y_test)
print("The score of linear is : %f"%score_linear)

# kernel = 'poly'
clf_poly = svm.SVC(kernel='poly')
clf_poly.fit(X_train,y_train)
score_poly = clf_poly.score(X_test,y_test)
print("The score of poly is : %f"%score_poly)
```

# 8、支持向量机SVM特点

## 8.1、抗干扰能力强

根据一些样本点，我们可以通过拉格朗日乘数法、KKT条件、对偶问题、SMO算法计算出支持向量机SVM的分类线$w^T\cdot x + b = 0$,其中

$$
\left\{\begin{aligned}&w = \sum\limits_{i=1}^n\lambda_i\cdot x_i\cdot y_i \\\\&b = \frac{1}{S}\sum\limits_{i \in S}(y_i - w^T \cdot x_i)\end{aligned}\right.
$$

S代表的是边界上的样本点。得到的分割下如下图所示：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/463/1657095974049/487eddf3446647ed838000c90c88dff8.png)

我们可以看出SVM在训练过程中找到的是两类点的分割线，计算样本中有少量异常点，在训练模型时依然能很正确的找到中间分割线，因为训练SVM时考虑了全量数据，确定这条线的b时只与支持向量点（边界上的点）有关。同样在测试集中就算有一些样本点是异常点也不会影响其正常分类，SVM具有抗干扰能力强的特点。

此外，由于训练SVM需要所有样本点参与模型训练，不然不能缺点这条线，所以当数据量大时，SVM训练占用内存非常大。这时SVM模型的缺点。

## 8.2、二分类及多分类

SVM同样支持多分类，我们可以将一个多分类问题拆分成多个二分类问题，例如有A,B,C三类，我们使用SVM训练时可以训练3个模型，第一个模型针对是A类和不是A类进行训练。第二个模型针对是B类和不是B类进行训练。第三个模型针对是C类和不是C类进行训练。这样可以解决多分类问题。
