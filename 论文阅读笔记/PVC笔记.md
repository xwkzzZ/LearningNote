<h1>PVC笔记</h1>



[原文](https://arxiv.org/abs/2412.09613)



<h2><font color=red>方法</font></h2>

<font color=green>**3.1 标准化视觉输入以形成为视频**</font>



需求：当前高性能的VLM通常采用不同的方法压缩图像和视频的visual token。由于计算资源有限，为图像分配更多标记有助于理解空间细节，而视频处理则倾向于减少每帧的token以容纳更多帧。





将VLM的视觉输入统一标准化为视频格式



将视频中的每个图像都会重复几次，以此形成所谓的静态视频

这样做的意义是在于：让模型能够进行"revisit"（回顾）(后面的PVC方法)

并且填充了一些帧数较少的片段



<font color=red>**3.2模型架构**</font>

![PVC(1)](../论文阅读笔记/img/PVC(1).png)



**Vision Transformer with Progressive Encoding**

亮点来了！！

它会只编码当前帧在中 在先前帧未存在的信息

向ViT最后$\tilde{L}$层添加时间注意力(temporal attention)



整体的流程：

$x:=x+S-MHA(LayerNorm(x))$

$x:=x+\alpha \odot T-MHA(AdaLN(x;x+TE))$

$x:=x+FFN(LayerNorm(x))$



S-MHA是spatial multi-head attention的缩写

提取空间特征，对每个patch分别作self-attention

输入前shape被整为：[B*T,N,C]，N是序列长度





T-MHA是temporal multi-head attention的缩写

捕捉时间维度上的视觉信息，输入前shape被整为：

[B*N,T,C]



TE是temporal embedding的缩写

$TE=W_2 \cdot SiLU(W_1\tilde{t})$

W2和W1都是可学参数，SiLU是256维度的正弦位置编码



<font color=red>Adapaptive Layer Normalization</font>

自适应归一化



$AdaLN(x,z)=\gamma(z)\odot LayerNorm(x)+\beta(z)$

$\gamma(z)=W_4\cdot SiLU(W_3z)$

$\beta(z)=W_6\cdot SiLU(W_5z)$

z是仿射操作的条件（还是不太懂）

![PVC(2)](../论文阅读笔记/img/PVC(2).png)





<h3><font color=green>Adaptive Compression</font></h3>

先前的使用PixleShuffle加MLP来做压缩的



现在采用PixelShuffle加AdaLN来做压缩：

可以将[B,T,N,C]压缩到[B,T,M,C]

整体的方法就是；

$\tilde{x}=PixelShuffle(x)\in R^{B\times T\times M\times (\frac{N}{M}\cdot C)}$

$v=MLP(AdaLN(\tilde{x},\tilde{x}+TE))$

M默认设置为1/16 N



PixelShuffle的kernel设置为4*4，16个压缩为1个token

![PVC(3)](../论文阅读笔记/img/PVC(3).png)

<font color=blue>训练策略以及数据</font>

两个阶段训练：pre-training,instruction-tuning，但是直接！！！两个阶段训所有参数(LLM,ViT)



Pre-training:

在图像文本对上训练（LAION)

在视频文本对上训练（InternVid）

二者配比1:1



Instruction-tuning;

1. 从InternVL2微调阶段薅来：7.2M图像-文本和视频-文本数据。

2. 从Vript、LLaVA-Video等收集和整理的3.4M视频-文本数据。

   

   图像-文本数据与视频-文本数据的采样比例设置为3:1。



<h2><font color=green>方法</font></h2>

![PVC(4)](../论文阅读笔记/img/PVC(4).png)

直接标准化会导致OCR相关图像任务（DocVQA -3.7，InfoVQA -6.5）和细粒度视频任务（MVBench -3.9）性能下降。这是因为重复图像为普通ViT和视觉压缩模块仅生成四组相同的视觉标记，编码了重复信息，导致空间细节丢失。然而，长视频理解任务因帧数增加而受益，这些任务对视觉细节需求较低。

![PVC(5)](../论文阅读笔记/img/PVC(5).png)



**结论：**

- 对于长视频任务VideoMME，两模型分数均随帧数增加而上升，但PVC增长更快，表明其更擅长消除时间冗余并捕获时间动态。
- 在图像任务上，增加图像重复次数不会提升基线模型性能，因为其重复视觉标记未提供更多信息。对于细节敏感任务InfoVQA，PVC在图像仅重复1次时表现较差（因仅提取部分信息），重复更多次数后细节信息得到补充，结果显著提升。对于MMB（图像空间细节较少），PVC的增益相对较小。



并且这么做引入的计算开销非常边缘化（视觉计算可重复利用）

![PVC(6)](../论文阅读笔记/img/PVC(6).png)



<h2><font color=blue>视觉token压缩</font></h2>

1.**池化（Pooling）、下采样（Down-sampling）或卷积（Convolution）**

2.**基于Q-Former的压缩**

3.**PixelShuffle模块**

4.其他专门针对某些内容，模块做注意力等

