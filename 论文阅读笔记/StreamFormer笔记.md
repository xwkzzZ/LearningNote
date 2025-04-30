<h1>StreamFormer笔记</h1>



[原文](https://arxiv.org/pdf/2504.20041)

(4.28 arXiv)



<h2><font color=red>方法</font></h2>

目标是给出某一时刻的三方面粒度的视觉特征：

$\{v^t,f^t,F^t\}=f_{video}(V^t)$

分别是global, temporal,spatila视频特征



直接将各种任务直接大一统：

重新定义多样时空任务，将他们全部重定义为视觉-语言对齐

给定一系列任务

$Q=\{q_1,q_2,...,q_N\}$

其中某个任务 $q_i$ 的输出视作对齐分数？（应该是笔误吧，应该是说：他们的输出经过后面的 $g_{tasks}$ 来评估这个输出的对齐程度，即视觉-文本对齐分数 $A_i$ )



这样子多个任务被集成为优化 $A_i$



$A_i=g_{tasks}(f_{video}(V^t),f_{text}(y),q_i)$



**首个开发支持在统一框架下处理多时空任务的流式视频骨干网络的工作，使其能够扩展并适用于广泛的视频理解任务。**



<h3><font color=green>视频编码(Video Encoding)</font></h3>

**必要的补充预先知识:patch embedding**



Patch Embedding 是ViT架构中的关键模块。

负责将二维图像转换为一维的序列表示，以便后续的 Transformer 编码器处理。其基本思路是将输入图像分割成若干固定大小的正方形补丁（patch），并将每个补丁展平成一维向量后，通过线性投影或等效的卷积操作映射为指定维度的嵌入向量，从而形成与 NLP 中 token 嵌入类似的序列输入



$e_i=W(flatten(p_i))+b$

$e_i$ 就是patch i的patch embedding



<font size=4>**回到正题：**</font>

输入视频流 $V^T\in R^{T\times H\times W\times 3}$

将它转化成patch embedding $z\in R^{T\times hw\times d}$









$z^{l+1}=f_{FFN}(f_{space}(f_{time}(z^l)))$

$f_{time}$ 是casual temporal attention

$f_{space}$ 是spatial attention

$f_{FFN}$即FFN



<font color=red>Casual Temporal Attention</font>

每一帧只能访问当前与过往的视觉特征



![StreamFormer(1)](../论文阅读笔记/img/StreamFormer(1).png)

进行了因果掩码（上对角线）



<font color=blue>Sptial Attention</font>

直接使用预训练好的ViT来做sptial attention，但是会在此基础上进行LoRA微调



**究极Call Back之必要的补充预先知识:LoRA数学表达式**

$Q^{'}=W^T_Qx^l+W^T_{QB}(W^T_{QA}x^l)$

新的Q=原始Q+LoRA的Q

通过 $W^T_{QA}x_l$将输入$x_l$投影到低秩空间（维度r）。

再通过$W_{QB}^T$ 将低秩特征映射回原始维度d,形成对原始查询的增量调整项。

对于$K^{'}$,$V^{'}$ 也是依葫芦画瓢



这样子就看懂了spatial attention：

![StreamFormer(2)](../论文阅读笔记/img/StreamFormer(2).png)

**Pipline:**

![StreamFormer(3)](../论文阅读笔记/img/StreamFormer(3).png)

Global Video feature是取temoral feature的最后一帧的视觉特征

（处理后最后一帧的视觉特征已经融合了先前的视觉特征）



<h3><font color=blue>语言编码Language Encoding</font></h3>

构建prompt模版，然后将类别标签转化为自然语言描述

将所有的GT转化为自由形式语言y,然后在运算得到它的embedding

$t=f_{text}(y)$



<h3><font color=red>时空多任务学习Spatiotemporal Multitasking</font></h3>



**三个层级：**

1.Global-level training

2.Temporal-level training

3.Spatial-level training



各个层级的训练，通过制定对应的子任务来训练

分别为：

*Global-level*:动作识别action recognition(AR)，视频文本检索video-text retrievla(VTR)



*Temporal-level training*:时序动作定位temporal action localization(TAL),时序视频定位temporal video grounding(TVG)



*Spatila-level training*:视频实例分割video instance segmentation(VIS)(封闭式类别),指代视频实例分割(RVOS)(开放式类别)（类别预定义好的）



然后三个层级训练都有对应的loss，总的StreamFormer的loss即三者之和（无加权）

![StreamFormer(4)](../论文阅读笔记/img/StreamFormer(4).png)

训练时，我们交替从各任务中采样数据进行前向传播，并通过梯度累积在遍历所有任务后统一执行反向传播和参数更新，避免对特定任务的偏向。



<h2><font color=green>表现</font></h2>

各个层级的预训练数据集：

![StreamFormer(5)](../论文阅读笔记/img/StreamFormer(5).png)



其他任务整体看上去是不错的，直接来看我关心的：

![StreamFormer(6)](../论文阅读笔记/img/StreamFormer(6).png)

消融实验：
![StreamFormer(7)](../论文阅读笔记/img/StreamFormer(7).png)