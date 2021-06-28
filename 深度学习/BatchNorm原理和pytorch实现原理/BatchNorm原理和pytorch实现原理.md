## 1.Batch Normalization原理

我们在图像预处理过程中通常会对图像进行标准化处理，这样能够加速网络的收敛，如下图所示，对于Conv1来说输入的就是满足某一分布的特征矩阵，但对于Conv2而言输入的feature map就不一定满足某一分布规律了（注意这里所说满足某一分布规律并不是指某一个feature map的数据要满足分布规律，理论上是指整个训练样本集所对应feature map的数据要满足分布规律）。而<font color=red>**我们Batch Normalization的目的就是使我们的feature map满足均值为0，方差为1的分布规律。**</font>
<img src="https://raw.githubusercontent.com/33499/note/main/深度学习/imgs/image-20210628115112142.png" alt="image-20210628115112142.png (1300×631) (raw.githubusercontent.com)" style="zoom:50%;" />

　BN其具体操作流程，如论文中描述的一样：

<img src="https://raw.githubusercontent.com/33499/note/main/深度学习/imgs/image-20210628142731758.png" alt="image-20210628142731758.png (1059×707) (raw.githubusercontent.com)" style="zoom:50%;" />

我们刚刚有说让feature map满足某一分布规律，理论上是指整个训练样本集所对应feature map的数据要满足分布规律，也就是说要计算出整个训练集的feature map然后在进行标准化处理，对于一个大型的数据集明显是不可能的，所以论文中说的是Batch Normalization，也就是我们计算一个Batch数据的feature map然后在进行标准化（batch越大越接近整个数据集的分布，效果越好）。我们根据上图的公式可以知道代表着**我们计算的feature map每个维度（channel）的均值，注意是一个向量不是一个值，向量的每一个元素代表着一个维度（channel）的均值。代表着我们计算的feature map每个维度（channel）的方差，注意是一个向量不是一个值，向量的每一个元素代表着一个维度（channel）的方差**，然后根据和计算标准化处理后得到的值。下图给出了一个计算均值和方差的示例：
![image-20210628143128346.png (2141×1149) (raw.githubusercontent.com)](https://raw.githubusercontent.com/33499/note/main/深度学习/imgs/image-20210628143128346.png)

经过这个<font color=red>**变换后某个神经元的激活x形成了均值为0，方差为1的正态分布，目的是把值往后续要进行的非线性变换的线性区拉动，增大导数值，增强反向传播信息流动性，加快训练收敛速度。**</font><font color=green>**但是这样会导致网络表达能力下降，为了防止这一点，每个神经元增加两个调节参数（scale和shift），这两个参数是通过训练来学习到的，用来对变换后的激活反变换，使得网络表达能力增强，即对变换后的激活进行如下的scale和shift操作，这其实是变换的反操作：**</font>

<img src="https://raw.githubusercontent.com/33499/note/main/深度学习/imgs/image-20210628143454617.png" alt="image-20210628143454617.png (539×126) (raw.githubusercontent.com)" style="zoom:33%;" />

## 2.pytorch实现

### 参数解释

```python
BatchNorm2d(256, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
```

**1.num_features**：一般输入参数为batch_sizenum_featuresheight*width，即为其中特征的数量，即为输入BN层的通道数；
**2.eps**：分母中添加的一个值，目的是为了计算的稳定性，默认为：1e-5,避免分母为0；
**3.momentum**：一个用于运行过程中均值和方差的一个估计参数（我的理解是一个稳定系数，类似于SGD中的momentum的系数）；
**4.affine**：当设为true时，会给定可以学习的系数矩阵gamma和beta
一般来说pytorch中的模型都是继承nn.Module类的，都有一个属性trainning指定是否是训练状态，训练状态与否将会影响到某些层的参数是否是固定的，比如BN层或者Dropout层。通常用model.train()指定当前模型model为训练状态,model.eval()指定当前模型为测试状态。
同时，BN的API中有几个参数需要比较关心的，**一个是affine指定是否需要仿射，还有个是track_running_stats指定是否跟踪当前batch的统计特性。容易出现问题也正好是这三个参数：trainning，affine，track_running_stats。**
**其中的affine指定是否需要仿射，也就是是否需要上面算式的第四个，如果affine=False则γ=1,β=0，并且不能学习被更新。一般都会设置成affine=True**。
**trainning和track_running_stats，track_running_stats=True表示跟踪整个训练过程中的batch的统计特性，得到方差和均值，而不只是仅仅依赖与当前输入的batch的统计特性。**相反的，如果track_running_stats=False那么就只是计算当前输入的batch的统计特性中的均值和方差了。当在推理阶段的时候，如果track_running_stats=False，此时如果batch_size比较小，那么其统计特性就会和全局统计特性有着较大偏差，可能导致糟糕的效果。
如果BatchNorm2d的参数track_running_stats设置False,那么加载预训练后每次模型测试测试集的结果时都不一样；track_running_stats设置为True时，每次得到的结果都一样。
running_mean和running_var参数是根据输入的batch的统计特性计算的，严格来说不算是“学习”到的参数，不过对于整个计算是很重要的。BN层中的running_mean和running_var的更新是在forward操作中进行的，而不是在optimizer.step()中进行的，因此如果处于训练中泰，就算不进行手动step()，BN的统计特性也会变化。

```python
model.train() #处于训练状态
for data , label in self.dataloader:
    pred =model(data)  #在这里会更新model中的BN统计特性参数，running_mean,running_var
    loss=self.loss(pred,label)
    #就算不进行下列三行，BN的统计特性参数也会变化
    opt.zero_grad()
    loss.backward()
    opt.step()
```

这个时候，要用model.eval()转到测试阶段，才能固定住running_mean和running_var，有时候如果是先预训练模型然后加载模型，重新跑测试数据的时候，结果不同，有一点性能上的损失，这个时候基本上是training和track_running_stats设置的不对。
如果使用两个模型进行联合训练，为了收敛更容易控制，先预训练好模型model_A，并且model_A内还有若干BN层，后续需要将model_A作为一个inference推理模型和model_B联合训练，此时希望model_A中的BN的统计特性量running_mean和running_var不会乱变化，因此就需要将model_A.eval()设置到测试模型，否则在trainning模式下，就算是不去更新模型的参数，其BN都会变化，这将导致和预期不同的结果。

### 具体实现

刚刚说了在我们**训练过程中**，均值![\mu _{\ss }](https://private.codecogs.com/gif.latex?%5Cmu%20_%7B%5Css%20%7D)和方差![\sigma_{\ss }^{2}](https://private.codecogs.com/gif.latex?%5Csigma_%7B%5Css%20%7D%5E%7B2%7D)是通过计算当前批次数据得到的记为为![\mu _{now}](https://private.codecogs.com/gif.latex?%5Cmu%20_%7Bnow%7D)和![\sigma _{now}^{2}](https://private.codecogs.com/gif.latex?%5Csigma%20_%7Bnow%7D%5E%7B2%7D)，而我们的**验证以及预测过程中**所使用的均值方差是**一个统计量记为![\mu _{statistic}](https://private.codecogs.com/gif.latex?%5Cmu%20_%7Bstatistic%7D)和![\sigma _{statistic}^{2}](https://private.codecogs.com/gif.latex?%5Csigma%20_%7Bstatistic%7D%5E%7B2%7D)。![\mu _{statistic}](https://private.codecogs.com/gif.latex?%5Cmu%20_%7Bstatistic%7D)和![\sigma _{statistic}^{2}](https://private.codecogs.com/gif.latex?%5Csigma%20_%7Bstatistic%7D%5E%7B2%7D)的具体更新策略如下，其中momentum默认取0.1：**

<img src="https://raw.githubusercontent.com/33499/note/main/深度学习/imgs/image-20210628144200377.png" alt="image-20210628144200377.png (1283×176) (raw.githubusercontent.com)" style="zoom:33%;" />

这里要注意一下，<font color=red>**在pytorch中对当前批次feature进行bn处理时所使用的是总体标准差**</font>，计算公式如下：

<img src="https://raw.githubusercontent.com/33499/note/main/深度学习/imgs/image-20210628144229097.png" alt="image-20210628144229097.png (555×168) (raw.githubusercontent.com)" style="zoom:33%;" />

<font color=red>**在更新统计量时采用的是样本标准差**</font>，计算公式如下：

<img src="https://raw.githubusercontent.com/33499/note/main/深度学习/imgs/image-20210628144239656.png" alt="image-20210628144239656.png (660×197) (raw.githubusercontent.com)" style="zoom:33%;" />

下面是使用pytorch做的测试，代码如下：

（1）bn_process函数是自定义的bn处理方法验证是否和使用官方bn处理方法结果一致。在bn_process中计算输入batch数据的每个维度（这里的维度是channel维度）的均值和标准差（标准差等于方差开平方），然后通过计算得到的均值和总体标准差对feature每个维度进行标准化，然后使用均值和样本标准差更新统计均值和标准差。

（2）初始化统计均值是一个元素为0的向量，元素个数等于channel深度；初始化统计方差是一个元素为1的向量，元素个数等于channel深度，初始化![\gamma](https://private.codecogs.com/gif.latex?%5Cgamma)=1，![\beta](https://private.codecogs.com/gif.latex?%5Cbeta)=0。

```python
import numpy as np
import torch.nn as nn
import torch
 
 
def bn_process(feature, mean, var):
    feature_shape = feature.shape
    for i in range(feature_shape[1]):
        # [batch, channel, height, width]
        feature_t = feature[:, i, :, :]
        mean_t = feature_t.mean()
        # 总体标准差
        std_t1 = feature_t.std()
        # 样本标准差
        std_t2 = feature_t.std(ddof=1)
 
        # bn process
        # 这里记得加上eps和pytorch保持一致
        feature[:, i, :, :] = (feature[:, i, :, :] - mean_t) / np.sqrt(std_t1 ** 2 + 1e-5)
        # update calculating mean and var
        mean[i] = mean[i] * 0.9 + mean_t * 0.1
        var[i] = var[i] * 0.9 + (std_t2 ** 2) * 0.1
    print(feature)
 
 
# 随机生成一个batch为2，channel为2，height=width=2的特征向量
# [batch, channel, height, width]
feature1 = torch.randn(2, 2, 2, 2)
# 初始化统计均值和方差
calculate_mean = [0.0, 0.0]
calculate_var = [1.0, 1.0]
# print(feature1.numpy())
 
# 注意要使用copy()深拷贝
bn_process(feature1.numpy().copy(), calculate_mean, calculate_var)
 
bn = nn.BatchNorm2d(2, eps=1e-5)
output = bn(feature1)
print(output)
```

**然后我们打印出通过自定义bn_process函数得到的输出以及使用官方bn处理得到输出，明显结果是一样的（只是精度不同）**

## 3.BatchNorm的好处

　　**①不仅仅极大提升了训练速度，收敛过程大大加快；②还能增加分类效果，一种解释是这是类似于Dropout的一种防止过拟合的正则化表达方式，所以不用Dropout也能达到相当的效果；③另外调参过程也简单多了，对于初始化要求没那么高，而且可以使用大的学习率等。**总而言之，经过这么简单的变换，带来的好处多得很，这也是为何现在BN这么快流行起来的原因。

