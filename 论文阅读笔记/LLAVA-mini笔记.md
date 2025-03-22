<h1>LLAVA-mini笔记</h1>

[原文](https://arxiv.org/pdf/2501.03895)





<h2><font color=red>LLAVA-MINI</font></h2>

流程：
![LLAVA-mini(1)](../论文阅读笔记/img/LLAVA-mini(1).png)

**视觉token压缩**

为了减少vision tokens，引入可学习的compression queried $Q^V$

它和vision tokens做cross-attention，选择性地提取视觉特征，得到压缩后的视觉tokens $\hat{H^v}\in R^{C^2\times d_h}$ 

然后为了保存空间信息，引入2D正弦位置编码在可学习的query以及原始的vision tokens上

最后压缩的表达为：

$\hat{H^v}=A\cdot H^v$

$A=Softmax((Q^V+PE(Q^V))\cdot(H^V+PE(H^V))^T)$

得到的 $\hat{H^V}$ 是C*C压缩的视觉tokens

A是相似度

($H^V$ 是经过编码再投影后得到的视觉token) 

**多模态预融合**

先前的模态pre-fusion模型 $f(\cdot)$ 由多个transformer块组成，每个transformer块共享相同的结构以及参数

融合了相关视觉信息的文本tokens $\hat{H}^q \in R^{l_q\times d_h}$ 由此得到：
$$
\hat{H^q}=f(Concat(H^v,H^q))[-l_q:]
$$
 最终($C^2+l_q$)个token给LLM



**高像素图片和视频建模**

1.对于图片，将他们沿着水平和竖直方向拆分成四块。

将 $N^2 \times 4$ 个视觉token压缩至 $C^2$ 。

pre-fusion模型将四个拆分的子图片，原始图片以及文本指令作为输入以此生成长 $l_q$ 的融合tokens

2.相较于lava-v1.5，减少了token数目和VRAM压力

此处它是将几帧和一段文本一起编码，而非一帧一帧编码



**训练**

*第一步：*

用视觉注释数据，学习对齐视觉和文本表征

专注于projection module

*第二步：*

引入compression, modality pre-fusion，然后进行端到端的训练

![LLAVA-mini(6)](../论文阅读笔记/img/LLAVA-mini(6).png)



<h2><font color=green> HOW DOES LLAVA UNDERSTAND VISION TOKENS?</font></h2>

<font size=5>序言分析</font>

得出两个重要结论：

**Vision Tokens are More Important in Early Layers**

直接看图：
![LLAVA-mini(1)](../论文阅读笔记/img/LLAVA-mini(2).png)

![LLAVA-mini(3)](../论文阅读笔记/img/LLAVA-mini(3).png)



**Most Vision Tokens are Focused in Early Layers**

![LLAVA-mini(4)](../论文阅读笔记/img/LLAVA-mini(4).png)



###### **以及一个实验，在不同层移除所有的视觉tokens**

![LLAVA-mini(5)](../论文阅读笔记/img/LLAVA-mini(5).png)



```
这一章是其实是在第三章，也就是related work之后，作者这么安排一节内容显然是为了为后文的压缩视觉token为1个做足铺垫，称之为“足够的研究动机”，也有更信服的数据来表明“视觉上砍掉后面的大量token是没多大问题的“
```



<h2><font color=blue>表现</font></h2>



**省流：**

![LLAVA-mini(7)](../论文阅读笔记/img/LLAVA-mini(7).png)

![LLAVA-mini(9)](../论文阅读笔记/img/LLAVA-mini(9).png)





<font color=red>**值得关注：**</font>

Lava-mini延迟砍到三十多毫秒上下

![LLAVA-mini(8)](../论文阅读笔记/img/LLAVA-mini(8).png)







<h2><font color=red>分析</font></h2>



在同等的FLOPs下，增加pre-fusion层数比增加compression vision tokens的数目带来的效应要大

##### 

实际上LLAVA-mini采用了query-based的方法来压缩视觉Tokens（虽然感觉最上面的图看不出来）

相比较于平均池化，它的效果更好（肯定的）

