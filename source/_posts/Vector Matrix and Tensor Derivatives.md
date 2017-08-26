---
title: Vector Matrix and Tensor Derivatives
date: 2017-08-06 16:19:29
comments: true
categories: [deep learning]
tags: [Tensor, Vector Matrix, 深度学习, deep learn]
---

# 简介
**目的:** 本节主要讲述反向传播和一种采用链式法则计算梯度的方法。理解反向传播的处理过程对于理解、开发、设计、调试神经网络至关重要。

**问题描述:** 本节中主要解决已知函数 $f(x)$，求函数在 $x$ 的梯度，其中 $x$ 为输入向量。

**目的：** 回想一下神经网络这个特定问题中,其中 $f$ 相当于损失函数 $L(x)$ ,输入参数 $x$ 对应训练数据和神经网络中的权重。例如，损失函数是SVM损失函数，输入为训练数据 $\left (x\_{i},y\_{j} \right )$ , $i=1 \cdots N$ 和网络的权重 $W$ 和偏置 $b$ 。请注意通常在机器学习中我们认为训练数据是给定，权重是一个可控的变量。因此，我们采用反向传播可以非常容易求得输入$x$的梯度。在训练过程中，只需要计算参数 $W$, $b$ 的梯度，所以我们可以采用它来执行参数的迭代。然而，在后续课程中$x_i$的梯度在某些情况下同样有用，例如：用于可视化和解释神经网络的执行过程。

如果您在参加本课程前已经了解了链式法则，我们乃鼓励您阅读本文。因为，本文以图解的形式讲解了反向传播的整个过程，对于理解反向传播能有一个直观的理解。

# 梯度
熟悉简单实例后，我们就能处理复杂函数求梯度。以两个变量相乘为例$f(x,y)=xy$，$x$， $y$ 的偏导分表为：

$$f(x,y)=xy\rightarrow\frac{\partial f}{\partial x}=y\qquad\frac{\partial f}{\partial y}=x$$

注解：请记住以下的演算。以下变化率标示函数在值附近无限小的区域内的变化率（如：一元函数的斜率）。

$$\frac{df(x)}{dx} =\lim_{h \rightarrow 0}\frac{f(x+h)-f(x)}{h} $$

采用等式左侧的符号表示函数 $f(x)$ 在 $x$ 处的微分。当 $h$ 比非常小时，上述表达是标示函数在此处的斜率。也就是说每个变量的导数标示整个表达式对于该变量的敏感度。例如：if $x=4$，$y=-3$ then $f(x,y)=-12$,此时函数 $f(x)$ 在 $x$ 处的微分为-3.这个表示，如果变量 $x$ 增加一小点，函数 $f(x)$ 的值将减少3倍。这个可以通过以下表达是看出$f(x,y)=f(x\)+h*\frac{df(x)}{dx}$,类似的$\frac{df(x\)}{dy}=4$,我们对 $h$ 增加一小点，函数的输出将增加 $4h$。
>每个变量的导数标示目标函数在此处的变化快慢。

根据上述，偏导$\Delta f$是一个$\Delta f=[\frac{\partial f}{\partial x},\frac{\partial f}{\partial y}]=[y,x]$的向量。尽管梯度是一个向量，本文中为简述表达我们采用梯度 $x$ 标示偏导向量 $x$。

我们可以求得两个变量相加的偏导：

$$f(x)=x+y\rightarrow\frac{\partial f}{\partial x}=1\qquad\frac{\partial f}{\partial y}=1$$

这个实例中，不管 $x$ 和 $y$ 是取什么值，函数对 $x$ 和 $y$的偏导都是1。也就是说 $x$ 、$y$ 的增长都会导致函数 $f$值的增长，并且这个值的增长比率只于实际的值 $x$ 和 $y$ 有关系（这个与两个变量的乘不一样）。最后一个case我们讨论Max操作：

$$f(x)=max(x,y)\rightarrow\frac{\partial f}{\partial x}=1(x>=y)\qquad\frac{\partial f}{\partial y}=1(y>=x)$$

这个函数中，两个变量较大的值的偏导为1，另外一个则为0。例如：当 $x=4$， $y=2$时，最大值为 $4$，这个函数在此处对于$y$值不敏感，也就是说，当我们在此处增加一个很小的$h$值函数还是输出4，对结果无影响，因此它的梯度为0。当然，当我们将$y$的值增加非常大时，将会对结果函数值产生影响，上述这种情况仅当$\lim_{h\to 0}$时候成立.

# 复合连锁法则
现在开始讲解更复杂的表达式的求解，目标函数由多个函数的组合，例如：$f(x,y,z)=(x+y)x$。这个表达式依然相对简单，我们采用它帮助直观我们理解反向传播。在这个例子中，我们将函数分解为两个表达式：$q=x+y$,$f=q*z$。在前面已经讲解过着两个函数如何求偏导。函数 $f$ 是 $q$ 和 $z$ 变量的乘积，所以，他们的偏导分别为$\frac{\partial f}{\partial q}=z$, $\frac{\partial f}{\partial z}=q$，其中 $q$为 $x$ 和 $y$ 的和，他们的$\frac{\partial q}{\partial x}=1$，$ \frac{\partial q}{\partial y}=1$。然而，在我们的使用中，我们无需关系 $q$ 值的偏导$\frac{\partial f}{\partial q}$的值，这个值在本应用场景没有用。我们只关系函数 $f$在值 $x$， $y$ ， $z$ 处的偏导。链式法则可以求得多个“链式”表达式的梯度。例如：$\frac{\partial f}{\partial x}=\frac{\partial f}{\partial q}\frac{\partial q}{\partial x}$。本例中我们可以通过两个偏导的乘积求得函数 $f$在 $x$的偏导。让我们来看一个python代码写的例子：

```py
# set some inputs
x = -2; y = 5; z = -4

# perform the forward pass
q = x + y # q becomes 3
f = q * z # f becomes -12

# perform the backward pass (backpropagation) in reverse order:
# first backprop through f = q * z
dfdz = q # df/dz = q, so gradient on z becomes 3
dfdq = z # df/dq = z, so gradient on q becomes -4
# now backprop through q = x + y
dfdx = 1.0 * dfdq # dq/dx = 1. And the multiplication here is the chain rule!
dfdy = 1.0 * dfdq # dq/dy = 1
```

最后我们将梯度赋值给变量 $[dfdx, dfdy, dfdz]$, 这些值告诉我们函数 $f$ 对于 $x$， $y$， $z$的敏感度。这个简单的例子阐述了反向传播的过程。今后我们将采用更加简洁的写法，因此我们不使用 $df$。这个例子中使用 $dfdq$ 表示 $dq$梯度的最终输出。

上面的计算过程通过下图图解：

{% img full-image '/images/Vector Matrix and Tensor Derivatives/graph1.jpg' %}

# 直观理解反向传播
反向传播是一个优美的本地处理过程。图中每个计算单元获得输入后，可以立即算出输出值和当前梯度（local gradient）。值得注意的是，计算过程无需关系全图中的其他值，就可以计算出当前梯度。当全部的前向传播计算完以后，逆向传播就可以根据前向传播的值计算梯度。采用链式法则，可以计算出所有输入的偏导。

>神经网络中每个计算单元也可以采用链式法则求得输出值和每个变量的偏导。  

接下来，让我们解释一下以上的例子。加法处理单元的输入 $[-2, 5]$ 计算结果为 $3$。加法处理计算完以后，接着输入每个变量的当前梯度$+1$ 。上图最终的输出结果为 $-12$。在反向传播的过程中运用链式法则求得加法计算单元的梯度为 $-4$。为表述直观，我们将每个单元的计算结果放在横线上，将反向传播的求得的结果放在横线下方。根据上面的计算方法，求得当前单元的梯度 $-4$。继续回退，就可以求得 $x$ 和 $y$ 的梯度（1\*-4=-4）。有上计算可以获得，如果x、y的值降低，他们的和也降低，这个使得他们的乘积增加。

反向传播可以理解为每个计算单元之间的作用使得最终结果是增大或减小（他们的增幅），如此影响最终的输出结果。

# Sigmod函数
上述计算单元可以是任意的处理函数。每个不同的函数都可以当成一个处理单元，然后再将多个处理单元组合为一个大的处理单元（神经网络中就是有一个个小的处理单元组合而成），为了方便运算，我们可以对一个复杂处理进行分解。接下来我们以Sigmod函数为例分析如何拆解：

$$f(W,x)=\frac{1}{1+e^{w0x0+w1x1+w2}}$$

在随后的课程中，Sigmod激活函数是一个2维的核函数。在此先把它当做一个输入为 $w$，$x$的函数。这个函数有多个处理单元组成，这些加法、乘法、max处理单元以及在上面进行了讲解。

$$f(x)=\frac{1}{x}\rightarrow\frac{\partial f}{\partial x}=-1/x^2$$

$$f(x)=e^x\rightarrow\frac{\partial f}{\partial x}=e^x$$

$$f_a(x)=ax\rightarrow\frac{\partial f}{\partial x}=a$$

函数$f_a$分别表示上式中的常数 $c$ 和 $a$。这可以当做已有一个输入的特殊的加法和乘法处理单元。在整个处理过程如下：

{% img full-image '/images/Vector Matrix and Tensor Derivatives/graph2.jpg' %}

由上处理可知，一个长的链式处理函数可以通过 $w$，$x$之间的点乘获得计算结果。这个函数被称作为Sigmoid函数$\partial (x\)$.这个函数的偏导可以采用一下方法求得：

$$\partial (x\)=\frac{1}{1+e^x}$$

$$\rightarrow\frac{d\partial(x)}{dx}=\frac{e^{-x}}{(1+e^{-x})^2}=(\frac{1+e^{-x}-1}{1+e^{-x}})(\frac{1}{1+e^{-x}})=(1-\partial(x)\)(\partial(x))$$


采用以上等式求函数的偏导变得非常简单。例如：sigmod函数输入 $1.0$在正向传播的时候输出 $0.73$。跟进这个表达是可以很快求得偏导 $（1-0.73）\*0.73~=0.2$。接下来看一下核函数求偏导的处理代码：

```py
w = [2,-3,-3] # assume some random weights and data
x = [-1, -2]

# forward pass
dot = w[0]*x[0] + w[1]*x[1] + w[2]
f = 1.0 / (1 + math.exp(-dot)) # sigmoid function

# backward pass through the neuron (backpropagation)
ddot = (1 - f) * f # gradient on dot variable, using the sigmoid gradient derivation
dx = [w[0] * ddot, w[1] * ddot] # backprop into x
dw = [x[0] * ddot, x[1] * ddot, 1.0 * ddot] # backprop into w
# we're done! we have the gradients on the inputs to the circuit
```

注：反向传播阶段。采用dot标示输入 $w$和 $x$的点乘。反向传播的过程中我们计算出每个变量的梯度。

本节我们主要讲一种方便使用的方法计算正向传播与反向传播。在此基础上就可以直观的了解正向传播和反向传播的计算过程。本章还提供了计算代码。

# 反向传播实例：计算过程
接下来我们学习更复杂的实例:

$$f(x,y)=\frac{x+\sigma(y)}{\sigma(x)+(x+y)^2}$$

很显然，函数非常适合与用作方向传播的练习。应为我们采用之前的方法对 $x$ 和 $y$进行分解求偏导将非常复杂。然而，实时事实证明完全没有必要这样做，我们无需一个明确的函数来表示这个演变过程，我们只需要知道如何计算它。以下是正向传播的演变过程：

```py
x = 3 # example values
y = -4

# forward pass
sigy = 1.0 / (1 + math.exp(-y)) # sigmoid in numerator   #(1)
num = x + sigy # numerator                               #(2)
sigx = 1.0 / (1 + math.exp(-x)) # sigmoid in denominator #(3)
xpy = x + y                                              #(4)
xpysqr = xpy**2                                          #(5)
den = sigx + xpysqr # denominator                        #(6)
invden = 1.0 / den                                       #(7)
f = num * invden # done!                                 #(8)
```

以上是正向传播计算的整个过程，我们引进了多个中间变量结构化代码，每个中间表达式都简单可以求偏导。因此，反向传播就比较容易算了。整个反向传播的计算代码如下：

```py
# backprop f = num * invden
dnum = invden # gradient on numerator                             #(8)
dinvden = num                                                     #(8)
# backprop invden = 1.0 / den 
dden = (-1.0 / (den**2)) * dinvden                                #(7)
# backprop den = sigx + xpysqr
dsigx = (1) * dden                                                #(6)
dxpysqr = (1) * dden                                              #(6)
# backprop xpysqr = xpy**2
dxpy = (2 * xpy) * dxpysqr                                        #(5)
# backprop xpy = x + y
dx = (1) * dxpy                                                   #(4)
dy = (1) * dxpy                                                   #(4)
# backprop sigx = 1.0 / (1 + math.exp(-x))
dx += ((1 - sigx) * sigx) * dsigx # Notice += !! See notes below  #(3)
# backprop num = x + sigy
dx += (1) * dnum                                                  #(2)
dsigy = (1) * dnum                                                #(2)
# backprop sigy = 1.0 / (1 + math.exp(-y))
dy += ((1 - sigy) * sigy) * dsigy                                 #(1)
# done! phew
```

注意点：
1. 保留正向传播的计算结果，应为在反向传播过程中会使用到。但是如果太复杂了可以重新计算；  
2. 累加：正向传播过程多次使用 $x$、 $y$ 计算过程中需要注意+=、=等操作符，避免覆盖之前的旧值；

# 反向传播模式
有趣的是反向传播通常可以以一种直观的方式阐述。例如神经网络中常用的三种计算（add、mul、max），在反向传播过程中都可以简单的阐述。让我们看一下一下的例子：

{% img full-image '/images/Vector Matrix and Tensor Derivatives/graph3.jpg' %}

通过上图可知：
**加法操作** 的梯度均匀分配给所有输入变量，不用管正向传播过程中的值。因为加法操作的局部梯度都是 $+1.0$，x\*1.0保持不变，因此输出的梯度完全等于输入时的梯度。本例加法操作的梯度将2.0的局部梯度传递给两个输入变量，均保持不变；
**max操作**与加法操作不一样，max操作将梯度传递给输入值大的。本例中因为 $1.0$大于 $-1$，所以在z出的梯度为 $2$，在w出的值为 $0$；
**乘法操作**解释起来相对复杂一点。局部梯度输入值是根据链式法则来确定梯度的输出。在本例中 $x$的梯度为 $-4.00\*2.00=-8.0$；

不直观的影响以及他们的影响。请注意，如果乘法操作的其中一个输入非常小，而另外一个输入非常大，那么乘法处理就会出现如下现象，它将为小的输入分配一个较大的梯度，并为一个大的数据分配一个较小的梯度。在线下分类器权重和输入点的乘积$W^TX_i$中，意味着数据集的大小对梯度有较大的影响。例如，如果在预处理期间将数据$X_i\*1000$意味着梯度也将扩大1000倍，所以必须通过降低学习率来补偿。

# 梯度计算中的向量运算
之前提的都是单个变量的处理，不过所有这些操作都适合于矩阵和向量计算。然是，在处理过程中需要注意维度。

**矩阵-矩阵的梯度**最复杂的处理是矩阵与矩阵的乘积：

```python
# forward pass
W = np.random.randn(5, 10)
X = np.random.randn(10, 3)
D = W.dot(X)

# now suppose we had the gradient on D from above in the circuit
dD = np.random.randn(*D.shape) # same shape as D
dW = dD.dot(X.T) #.T gives the transpose of the matrix
dX = W.T.dot(dD)
```

tips：使用维度分析！我们无需记住他们的维度，因为可以通过推导获得 $dW$和 $dx$ 的维度。例如，我们知道计算后求得权重的梯度 $dW$ 的维度必须和 $W$计算后的的维度，计算后的维度取决于 $x$ 和 $dD$的乘积。总是存在一种方法可以计算出维度，因此维度分析是有效的。例如，$x$和 $dD$ 分别为 $[10\*3]$, $[5\*3]$的矩阵，根据维度分析，我们可以求得 $dW$ 和 $w$ 的维度为 $[5\*10]$ 可以通过$dD.dot(X.T)$求得。

**使用小的明确的例子**有时很难求得复杂表达式的梯度。我们建议先将复杂表达是分解，然后在纸上推导出他们的求导过程。

Erik Learned-Miller撰写了一个关于矩阵和向量求导的资料，[点击直达](http://cs231n.stanford.edu/vecDerivs.pdf)。 

# 总结
* 我们采用直观的流程图阐述梯度的计算过程，同时每个变量是如何影响函数的输出值。
* 我们讨论了分阶段计算反向传播值。我们可以先将负责函数分解为可以求导的小模块，然后采用链式法则求得梯度。至关重要的是您无需将求导函数在纸上进行推导，您就可以求得每个变量偏导。因此，将表达式进行分解后可以分阶段计算操作值，然后通过反向传播求得梯度。

在下节中，我们将开始学习定义神经网络，通过反向传播我们可以计算神经网络损失函数的梯度。换句话说，我们已经具备训练审定网络的能力，并掌握了本课程中比较难懂的概念！

# 参考文献
[原英文地址：](http://cs231n.github.io/optimization-2/)
[Automatic differentiation in machine learning: a survey](http://arxiv.org/abs/1502.05767)

