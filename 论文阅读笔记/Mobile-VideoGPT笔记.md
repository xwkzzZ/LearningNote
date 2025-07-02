<h1>Mobile-VideoGPT笔记</h1>



[原文](https://arxiv.org/pdf/2503.21782)



<h2><font color=green>Motivation</font></h2>

视频理解一直面临一个计算量大，参数多，推理慢的问题

视频理解分：video caption,visual question answering, visual grounding

虽然GPT-4o，Gemini-1.5-pro以及一些开源的模型Video-LLaVA，Video-ChatGPT很强，但是

部署这些模型都面临问题：吞吐率低，模型太庞大

同时批斗了下2025ICLR的LLaVA-Mini 模型太大(8.4B)，实时表现不行，边缘部署不切实际（我理解的就是在那些小型的设备上）

原话：

For instance, the recently intro duced LLaVa-Mini  proposes an image and video MLLM based on a single one-vision token, yet its total number of parameters is 8.4B and the model size is still around 20GB, making it impractical for real-time per formance and edge deployment.



虽然有：知识蒸馏(knowledge distillation)，模型裁剪(model pruning),高注意力(efficient attetion),参数微调(parameter-efficient fine-tuing)

解决模型在有限资源的复杂性的问题

但是还是一种取舍了一部分的模型性能或者硬件加速

同时如果把某些部件给减小规模（比如视觉或文本编码器），但是原本的投影方式，如何去适应这种改动，保持原有的良好表现，也是个问题



又快又好：
![Mobile-VideoGPT(1)](../论文阅读笔记/img/Mobile-VideoGPT(1).png)





A lighitweight multimodal model designed specifically for real-time video understanding.

First work to propose a lightweight and efficient video understanding framework with a focus on real-time throughput!!!



<h2><font color=red>Mobile-VideoGPT</font></h2>

**结构图：**

![Mobile-VideoGPT(2)](../论文阅读笔记/img/Mobile-VideoGPT(2).png)

视频编码器是一个轻量级的VideoMamba encoder

图像编码器就是CLIP-based Image encoder



提取视觉信息那一块还是老生常谈的的方法，采用均匀采样T帧，然后给两个编码器去处理

proj就是使用多个可学习的权重直接做乘积来实现映射操作

然后再将两种视觉信息得到的token融合为一个单一长度（一维的）token序列



<h3><font color=red>Frame Selection</font></h3>

直接提取T帧然后给视频编码器的方法简单粗暴，但是存在：冗余，低效的问题



卧槽～～！妙啊～～！

图像编码器照常编码，但是得到视觉特征后：$F_{clip}\in R^{T\times H_s \times W_s\times D_s}$

然后将时间维度，空间维度全部堆叠在一起：$S=T\times H_s \times W_s $ ,拉直，这样就成了一张二维的表，一个维度是所有的各个切下来的特征图的小块（不是像素点，因为经过图像编码器之后， $H!=H_s$ 一般应该是大于，所以姑且称之为提取到的特征图的一小块；另一个维度是spatial dimension的特征维度，每个维度个蕴涵了各种视觉信息特征（纹理，线条等等）

这样的展平的二维的特征图，特征表被称为： $F_{sp} \in R^{S\times D_f}$

随后最后self-attention的操作：

$SA=Softmax[\frac{F_{sp}F_{sp}^T}{\sqrt{d_f}}]$

如果仔细去思考这个矩阵的运算：不难理解，前者的每一行代表的是某一个特征图的最小块对应的所有特征维度，去和右边的某一列：所有特征图的小块的各个维度做乘积操作，然后相加——算的是一个特征块所有特征和其他所有特征块某一个特征的相似度（一行乘一竖）。

然后依次再（一行乘以所有竖），得到的是一个特征块所有特征和其他所有特征块所有特征做相似度计算

最后将所有行进行完这些操作，得到一个 $S\times S$ 大小的矩阵，表示的就是所有特征块之间的注意力（相似度，权重，叫法不一样而已）

这个矩阵再除以下面那个数（经典的归一化），得到的就是：一段时间内，通过图像编码器处理得到的所有特征图中的所有最小特征块之间的权重图（它跨越了时间维度），暂且叫这个权重图为Map吧





之后作者的操作是：将Map沿着每一整进行累加（每一行有S个元素)，也就是对某一个每一个特征图最小块对应其他特征图的最小块的权重相加，这样累加得到的结果是每一帧内多个特征块的权重之和。得到 $[S,1]$ 矩阵（张量）

然后再按照 $S/T$ 个行进行合并，按照帧数，各个特征块回并到每一帧中，得到每一帧的重要程度（即包含了自己内部的self-attention,又包含了和其他帧之间的cross-attention），即 $[T,1]$ 矩阵（张量）

矩阵每一行代表每一帧的重要程度！哒哒！

这就是作者所说的： we sum the attention scores each token receives to produce a frame wise importance vector $T_{score}\in R^T$ 

然后再去选取输入给video encoder最重要的几帧： $KF=Top-K(T_score,K)$

选取后再输入给video encoder



<font color=orange>为什么要SA数学表达式下分母有那一坨？</font>

**数值稳定性**
 如果 token 向量各维独立且方差为 1，那么两向量的点积期望为 0、方差为 $d_f$。

- 维度越大，点积数值越容易变得很大（很显然，在之前上面那个一行乘以一竖的时候，维度越大，行和竖都越长，最后相加后就越大）。
- 直接送进 Softmax 会导致指数函数“爆炸”，梯度趋于 0 或 ∞，训练不稳。

**缩放到合适区间**
 将点积除以 $\sqrt{d_f}$ 后，方差约为 1，数值落在一个对 Softmax 友好的范围内，使注意力权重不过度饱和，也不会过于平坦。





<h3><font color=orange>Efficient Toke Projection</font></h3>

![Mobile-VideoGPT(3)](../论文阅读笔记/img/Mobile-VideoGPT(3).png)

**FFN**:基本等同于MLP，但是MLP是多层的，FFN可以一层

**Token Reduction:**使用的AdaptivePool，将H,W缩小，从而达到整体的token数目减少

经典的token减少方式（连知乎都泛滥了）：

$(B,N,D)->(B,D,N)->(B,D,H,W)->(B,D,H/r,W/r)->(B,D,N_r)->(B,N_r,D)$

然后:$N>N_r$

**Convolution-based postional encoder:** 主要是为了在进行后token reduction之后回补一些空间/时间上的信息

（之前做压缩，可能破坏，损失了部分了这方面的信息）

位置编码也有很多种：

- 正弦绝对 PE （直接给token embedding加上一个正余弦的表达式）

- 可学习的 PE (lava-mini有)
- 相对PE（transformer中的用于强调query token和k/v  token 之间隔了几步（数学表达式有点复杂)
- Conv-based PE (本文)





<h2><font color=blue>Experiments</font></h2>

<h3><font color=blue>细节</font></h3>

image encoder:CLIP-B/16 (86M(只算图像编码器）))

video encoder:VideoMamba-M (74M)

LLM:Qwen-2.5 0.5/1.5B

训练：

![Mobile-VideoGPT(4)](../论文阅读笔记/img/Mobile-VideoGPT(4).png)



<h3><font color=blue>真正关心的</font></h3>

![Mobile-VideoGPT(5)](../论文阅读笔记/img/Mobile-VideoGPT(5).png)

**速度很快！**



**而且质量也有保证：**

![Mobile-VideoGPT(6)](../论文阅读笔记/img/Mobile-VideoGPT(6).png)



和LLaVA-OneVision比，其实感觉，5:5开，甚至有点小劣

![Mobile-VideoGPT(7)](../论文阅读笔记/img/Mobile-VideoGPT(7).png)
