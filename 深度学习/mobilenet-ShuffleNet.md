# MobileNetV1



## 网络结构

**Depthwise Separable Convolution**：把标准卷积分解成**深度卷积(depthwise convolution)**和**逐点卷积(pointwise convolution)**。这么做的好处是可以大幅度降低参数量和计算量。分解过程示意图如下：

<img src="C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615102022045.png" alt="image-20210615102022045" style="zoom:50%;" />



## **标准卷积与Depthwise Separable Convolution(DW+PW)比较**：

![image-20210615102443038](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615102443038.png)

## 引入控制模型大小的超参数

**宽度因子α (Width multiplier )**，用于控制输入和输出的通道数，即输入通道从**M**变为**αM**,输出通道从**N**变为**αN**。
**分辨率因子ρ (resolution multiplier )**，用于控制输入和内部层表示。即用分辨率因子控制输入的分辨率。

# MobileNetV2

## Inverted residuals

<img src="C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615104323173.png" alt="image-20210615104323173" style="zoom: 25%;" />

从比较中可以看出**残差块是先降维再升维**，而**倒残差块是先升维后降维**。还有**倒残差块中卷积使用的是Relu6激活函数**

## Linear Bottlenecks

这部分难理解，需要看原论文详解，**v2在倒残差块最后一个1*1卷积层使用的是线性激活函数**

论文实验表示**在低维空间中使用ReLU做激活变换会丢失很多信息**，论文针对这个问题使用linear bottleneck(即不使用ReLU激活，做了线性变换)的来代替原本的非线性激活变换。到此，优化网络架构的思路也出来了：通过在卷积模块中后插入linear bottleneck来捕获兴趣流行。 实验证明，使用linear bottleneck可以防止非线性破坏太多信息。

## 网络结构

v2的整体网络结构如下：

![image-20210615110222091](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615110222091.png)

只有**当stride=1且输入特征矩阵与输出特征矩阵shape相同时才有shortcut连接**。

## 性能表现

分类任务

![image-20210615141954042](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615141954042.png)

目标检测

![image-20210615142711752](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615142711752.png)

截图自原文，可以看到在cpu上检测速度也很快了。

# ShuffleNetV1

## 分组卷积

<font color=red>**Group convolution是将输入层的不同特征图进行分组，然后采用不同的卷积核再对各个组进行卷积**</font>，这样会降低卷积的计算量。因为一般的卷积都是在所有的输入特征图上做卷积，可以说是全通道卷积，这是一种通道密集连接方式（channel dense connection），而group convolution相比则是一种通道稀疏连接方式（channel sparse connection）。

### 1、分组卷积的矛盾——计算量

使用group convolution的网络有很多，如Xception，MobileNet，ResNeXt等。其中<font color=red>**Xception和MobileNet采用了depthwise convolution，这是一种比较特殊的group convolution，此时分组数恰好等于通道数，意味着每个组只有一个特征图。**</font>是这些网络存在一个很大的弊端是采用了**密集的1x1 pointwise convolution**（如下图）。

<img src="https://img2018.cnblogs.com/blog/1161096/201901/1161096-20190125100701953-660808452.png" alt="img" style="zoom:50%;" />

这个问题可以解决：对1x1卷积采用channel sparse connection 即分组操作，那样计算量就可以降下来了，但这就涉及到另外一个问题。



### 2、分组卷积的矛盾——特征通信

group convolution层另一个问题是**不同组之间的特征图需要通信**，否则就好像分了几个互不相干的路，大家各走各的，会降低网络的特征提取能力，这也可以解释为什么Xception，MobileNet等网络采用密集的1x1 pointwise convolution，因为要保证group convolution之后不同组的特征图之间的信息交流。



### 3、channel shuffle

为达到特征通信目的，我们不采用dense pointwise convolution，考虑其他的思路：**channel shuffle**。如图b，其含义就是对group convolution之后的特征图进行“重组”，这样可以保证接下了采用的group convolution其输入来自不同的组，因此信息可以在不同组之间流转。图c进一步的展示了**这一过程并随机**，其实是“均匀地打乱”。

在程序上实现channel shuffle是非常容易的：假定将输入层分为 ![g](https://www.zhihu.com/equation?tex=g) 组，总通道数为 ![g\times n](https://www.zhihu.com/equation?tex=g%5Ctimes+n) ，首先你将通道那个维度拆分为 ![(g,n)](https://www.zhihu.com/equation?tex=%28g%2Cn%29) 两个维度，然后将这两个维度转置变成 ![(n,g)](https://www.zhihu.com/equation?tex=%28n%2Cg%29) ，最后重新reshape成一个维度![g\times n](https://www.zhihu.com/equation?tex=g%5Ctimes+n)。

![image-20210615150510815](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615150510815.png)

## ShuffleNet基本单元

ShuffleNet的核心是采用了两种操作：<font color=red>**pointwise group convolution**</font>和<font color=red>**channel shuffle**</font>，这在保持精度的同时大大降低了模型的计算量。**其基本单元则是在一个残差单元的基础上改进而成。**

下图a展示了基本ResNet轻量级结构，这是一个包含3层的残差单元：首先是1x1卷积，然后是3x3的depthwise convolution（DWConv，主要是为了降低计算量），这里的3x3卷积是瓶颈层（bottleneck），紧接着是1x1卷积，最后是一个短路连接，将输入直接加到输出上。

下图b展示了改进思路：将密集的1x1卷积替换成1x1的group convolution，不过在第一个1x1卷积之后增加了一个channel shuffle操作。值得注意的是3x3卷积后面没有增加channel shuffle，按paper的意思，对于这样一个残差单元，一个channel shuffle操作是足够了。还有就是3x3的depthwise convolution之后没有使用ReLU激活函数。

下图c展示了其他改进，对原输入采用stride=2的3x3 avg pool，在depthwise convolution卷积处取stride=2保证两个通路shape相同，然后将得到特征图与输出进行连接（concat，借鉴了DenseNet？），而不是相加。极致的降低计算量与参数大小。

<img src="C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615160113370.png" alt="image-20210615160113370" style="zoom:50%;" />

## ShuffleNet网络结构

![image-20210615160408438](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615160408438.png)

可以看到开始使用的普通的3x3的卷积和max pool层。然后是三个阶段，每个阶段都是重复堆积了几个ShuffleNet的基本单元。对于每个阶段，第一个基本单元采用的是stride=2，这样特征图width和height各降低一半，而通道数增加一倍。后面的基本单元都是stride=1，特征图和通道数都保持不变。对于基本单元来说，其中瓶颈层，就是3x3卷积层的通道数为输出通道数的1/4，这和残差单元的设计理念是一样的。

# ShuffleNetV2

我们首先来看v1版本和v2版本的基础单元，（a）和（b）是ShuffleNet v1的两种不同block结构，两者的差别在于后者对特征图尺寸做了缩小，这和ResNet中某个stage的两种block功能类似，同理（c）和（d）是ShuffleNet v2的两种不同block结构：

![image-20210615164831008](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615164831008.png)

看点如下：

　　从（a）和（c）的对比可以看出首先（c）在开始处增加了一个**channel split**操作，这个操作将输入特征的通道分成c-c’和c’，c’在文章中采用c/2，这主要是和**第1点**发现对应

　　然后（c）中取消了1*1卷积层中的group操作，这和**第2点**发现对应，同时前面的channel split其实已经算是变相的group操作了

　　channel shuffle的操作移到了concat后面，和**第3点**发现对应，同时也是因为第一个1*1卷积层没有group操作，所以在其后面跟channel shuffle也没有太大必要

　　最后是将element-wise add操作替换成concat，这个和**第4点**发现对应。

多个（c）结构连接在一起的话，channel split、concat和channel shuffle是可以合并在一起的。（b）和（d）的对比也是同理，只不过因为（d）的开始处没有channel split操作，所以最后concat后特征图通道数翻倍，可以结合后面具体网络结构来看：

![image-20210615165255480](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20210615165255480.png)

现在我们查看一个新的概念：**内存访问消耗时间（memory access cost）**，它正比于模型对内存的消耗（特征大小+卷积核大小），在这篇文章中作者使用该指标来衡量模型的速度而非传统的**FLOPs（float-point operations）**，它更多的侧重卷积层的乘法操作，作者认为FLOPs并不能实质的反应模型速度。

现在我们来看一下这四点发现，分别对应了结构中的四点创新：

```
卷积层的输入和输出特征通道数相等时MAC最小，此时模型速度最快

过多的group操作会增大MAC，从而使模型速度变慢

模型中的分支数量越少，模型速度越快

element-wise操作所带来的时间消耗远比在FLOPs上的体现的数值要多，因此要尽可能减少element-wise操作
```

具体性能表现看原论文。