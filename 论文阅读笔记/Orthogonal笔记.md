<h1>Orthogonal笔记</h1>



[原文](https://arxiv.org/pdf/2504.01961)



**本文主要贡献点**

1.量化了三种视频学习方法在顺序学习方法而非乱序学习中的性能下降(DoRA,VideoMAE,future prediction)

2.提出了一个几何原理的优化器，使用orthogonal(直角，矩形)梯度

3.对现在常用的优化器SGC,AdamW使用orthogonal梯度，以提高

4.使用orthogonal优化器用到上面三种视频学习方法，都有提升





<h2><font color=red>方法</font></h2>

![Orthogonal(1)](../论文阅读笔记/img/Orthogonal(1).png)

视频在顺序情况下，相邻的两帧之间的cos相似度是接近1的

这样子，参数就很容易更新不动



<h3><font color=green>必要的补充知识</font></h3>

**什么是正交分量？**

指的是一个向量在另一个向量方向上的“垂直部分”

也就是不相关的部分

$g_t$ 在 $g_{t-1}$ 方向上的投影向量和正交向量





结合具体问题情境：

前后连个梯度他们是非常相似的，那么他们的



正交梯度的目的就是把两帧之间极大相关内容滤掉（投影向量），而是保留正交向量，去通过不一样的地方，去更新



*也就是这样：*

<font color=red>梯度就这样计算:</font>

![Orthogonal(2)](../论文阅读笔记/img/Orthogonal(2).png)



<h3><font color=blue>文绉绉的理论分析</font></h3>

**对于视频理解两个极端的分析**

(1)当训练数据是独立同分布时候

（我的理解就是各个数据相关性很低，应该说是不相关，也就是 $cos(g_t,g_{t-1})$ 接近0）

那么此时得到的 $u_t$ 正交梯度就是原始的梯度（不看原先的内容，每一步独立，没有相同的信息，每一步也是正交的）（公式1直接后面减了个0）



(2)当训练数据高度连续

（直接一张图片重复到死, $cos(g_t,g_{t-1})=1$）

此是的更新梯度是：

$u_t=g_t-\frac{||g_t||}{||g_{t-1}||}g_{t-1}$

虽然是一条公路，但是不会完全按照当前的梯度去更新，避免完全一个方向去更新，并且会根据前面的梯度给定一个类似“偏移”“惯性”

![Orthogonal(3)](../论文阅读笔记/img/Orthogonal(3).png)

在上面两个极端的例子中，都是站得住脚，兼容的，在正常视频例子中，就是：



滤去相同的信息，拿出出现的新信息，然后去更新（就是拿出正交梯度去更新）





<h3><font color=green>LET'S CONTINUE</font></h3>

但是前面的公式都是按照前一步进行更新迭代的，这可能就很不稳定，噪声所带来的扰动就很大



所以会维护一个类似动量 “momentum”的EMA(指数移动平均),记为 $c_t$:

$c_t:=\beta c_{t-1}+(1-\beta)g_t$



每次在计算新的正交梯度时候，滤去的不再是仅仅根据“前一步”的梯度来去滤掉投影向量，获得正交向量，而是根据这个类似动量的东西去更新:

$u_t=g_t-proj_{c_{t-1}}g_t$



**应用到SGD,AdamW:**

![Orthogonal(4)](../论文阅读笔记/img/Orthogonal(4).png)

![Orthogonal(5)](../论文阅读笔记/img/Orthogonal(5).png)



<h2><font color=green>表现</font></h2>

<h3><font color=blue>DoRA基础上跑单视频</font></h3>

![Orthogonal(9)](../论文阅读笔记/img/Orthogonal(9).png)



<h3><font color=purple>VideoMAE基础上跑视频数据集</font></h3>



![Orthogonal(7)](../论文阅读笔记/img/Orthogonal(7).png)

采样策略除开沿着时间维度直接采样，还可以每个时间点，多个视频一起采样，即batching along videos:

![Orthogonal(8)](../论文阅读笔记/img/Orthogonal(8).png)

实验结果是：

采用Orthogonal AdamW更好

采用batching along time axis更好

（SSV2是一个2-6秒的短视频，细致动作分类的数据集）

(K400即：Kinetics-400,大的动作分类数据集，视频大致上10秒)



图标右边的linear-probe和attn-probe指的是：

在原本的视觉编码器（❄️）的基础上，训练一个线性层（linear-probe)，或者训练一个transformer快加MLP层



结果是：

Attn-probe横向对比linear-probe,attn-probe更好（有点废话）





<h3><font color=orange>视频流上未来预测</font></h3>

![Orthogonal(6)](../论文阅读笔记/img/Orthogonal(6).png)

(Pixel MSE,PSNR)数值越低越好





**总结就是都没毛病**
