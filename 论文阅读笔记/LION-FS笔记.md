<h1>LION-FS笔记</h1>



[原文](https://arxiv.org/abs/2503.03663)



<h2><font color=red>方法</font></h2>



<h4><font color=green>双流视觉编码</font></h4>

采用视频编码器和图像编码器组合进行视觉编码

具体情况是：

用EgoVLPv2（视频编码器），记为 $E_{ego}$

用SigLIP（图像编码器），记为 $E_{gen}$

以此得到关于视觉的两块序列

![LIONE_FS(1)](../论文阅读笔记/img/LION-FS(1).png)

为了将两种序列对齐，作者对比了直接concat和引入一个router来融合（前者会加长序列长度）



将SigLIP的CLS token作为视觉引导，然后通过下面的方式来融合

（这里疑似论文表述有误，他的意思是直接拿图像编码器的CLS token来作为VG，进行后面的运算，个人认为应该是分别拿出视频编码器和图像编码器的VG来进行MLP得到 $G_f([VG])$ 它包含两个元素，这样似乎更合理）

 ![LION-FS(2)](../论文阅读笔记/img/LION-FS(2).png)

大致示意图：

![LION-FS(3)](../论文阅读笔记/img/LION-FS(3).png)



<h4><font color=purple>稀疏编码与token丢弃</font></h4>

**动机：**帧与帧之间差别并不是那么大，有冗余token，所以信息密度低，但是如果直接进行池化操作会有损交互性



![LION-FS(4)](../论文阅读笔记/img/LION-FS(4).png)

直接进行多层上述操作：

如果token数目小于该层的的设定百分比（每一层最多保留 $\beta$ 的token去进行self-attention+FFN操作），那么保持不变，如果大于，那么要丢弃一部分再去做“特征增强的操作”

![LION-FS(5)](../论文阅读笔记/img/LION-FS(5).png)

可以看到公式上其实大于阀值的时候，只是丢弃一部分去做self-attention+FFN然后再residual回来

token数目不变，只是对重要的token进行了提取

![LION-FS(6)](../论文阅读笔记/img/LION-FS(6).png)



<h4><font color=red>Slow Path</font></h4>

![LION-FS(7)](../论文阅读笔记/img/LION-FS(7).png)



通过引入grid tokens，box tokens两方面来实现细粒度的特征提取

然后再prompt:

"Stream: [Frame Tokens] [Grid Tokens]. User:Please focus on [Box Tokens]. Assistant:"



**总体的pipline一览：**

![LION-FS(8)](../论文阅读笔记/img/LION-FS(8).png)









<h2><font color=green>表现与总体评价：</font></h2>

**表现**

动脑子想象也知道，这样做会提高延迟，降低响应上的时间的准确率，但是会提高回答的质量

![LION-FS(9)](../论文阅读笔记/img/LION-FS(9).png)



**总体评价**

感觉这篇论文不大行，创新有限，A+B+C的痕迹比较明显

是否能做到所说的低延迟，保持怀疑，尤其是slow think与fast think是同时进行的(当需要进行响应的时候)



