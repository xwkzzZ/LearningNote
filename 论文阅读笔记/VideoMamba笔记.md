<h1>VideoMamba笔记</h1>



[原文](https://arxiv.org/pdf/2403.06977)



<h2><font color=green>引言</font></h2>

**主要关注四个问题：**

1.在视觉上的放缩性：纯mamba模型容易过拟合，但是经过self-distilation策略的VideoMamba表现可以（在输入与模型规模都在扩大时，也不需要在大规模数据集上预训练）



2.短期动作识别的感知力：良好



3.长期视频理解：优秀(remarkable)

比TimeSformer快6倍



4.其他模态的兼容性

视频文本检索有提升，但尤其在长视频下，更好



<h2><font color=red>方法</font></h2>

$\bar{A}=exp(\Delta A)$

$\bar{B}=(\Delta A)^{-1}(exp(\Delta A)-I)\cdot \Delta B$

$h_t=\bar{A}h_{t-1}+\bar{B}x_t$

$y_t=Ch_t$



$B\in R^{B\times L\times N}, C\in R^{B\times L\times N}, \Delta \in R^{B\times L\times D}$

其中$\Delta$ 是从输入x变来的

这样就有了上下文感知能力和适应性



**双向SSM** 

视觉上采取双向SSM，也称为 B-Mamba



![VideoMamba(1)](../论文阅读笔记/img/VideoMamba(1).png)



<h3><font color=green>VideoMamba</font></h3>

![VideoMamba(2)](../论文阅读笔记/img/VideoMamba(2).png)

使用3D卷积来将输入视频进行映射

$X^v\in R^{3\times T\times H\times W}$ 映射到 $X^p \in R^{L\times C}$

其中的 $L=t\times h\times w(t=T,h=\frac{H}{16},w=\frac{W}{16})$



VideoMamba编码器：

$X=[X_{cls},X]+p_s+p_t$

其中 $X_{cls}$是一个可学习的分类的token，在序列开始的时候加入

$p_s,p_t$ 分别是空间编码，时间编码（因为牢大对token位置很敏感）



就这样子，多个B-Mamba堆叠，再走一个层归一化，线性层分类



同时为了实现B-Mamba对时间空间的输入感知

在原本B-Mamba的基础上做了其他类的尝试：

（原本的B-Mamba是b）

![VideoMamba(3)](../论文阅读笔记/img/VideoMamba(3).png)

实验发现：

a最好：Spatial-First

牢大的线性复杂度支持它能够去做高分辨率的长视频





与TimeSformer和ViViT相比：

它们将时空注意力拆开，虽然解决了自注意力机制的二次复杂度，但是引入了额外的参数，而且在掩码与训练的场景就不如原来的联合attention了



但是VideoMamba这块就没毛病

![VideoMamba(4)](../论文阅读笔记/img/VideoMamba(4).png)



<h3><font color=blue>架构</font></h3>

采用Mamba默认的超参（state dimension 16,expansion ratio 2)

但调整了embedding dimension

但是mamba模型面临一个问题就是容易过拟合（VMamva-B)（在3/4epoch最好）

为了克服这个问题，采用了知识蒸馏策略,然后能够将较好的收敛





<h2><font color=purple>表现</font></h2>

**重点关心长视频理解**

![VideoMamba(5)](../论文阅读笔记/img/VideoMamba(5).png)

**多模态视频理解**

![VideoMamba(6)](../论文阅读笔记/img/VideoMamba(6).png)
