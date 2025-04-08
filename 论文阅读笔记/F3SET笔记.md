<h1>F3SET笔记</h1>



[原文](https://openreview.net/pdf?id=vlg5WRKHxh)



<h2><font color=red>F3ED</font></h2>

![F3SET(1)](../论文阅读笔记/img/F3SET(1).png)



<h3><font color=blue>问题定义</font></h3>

给定 $X\in R^{H\times W\times  3\times N}$ 作为输入（N帧RGB图片）

输出为：

M对 $(E_i,t_i)$ ,前者是事件类型，后者是时间戳

E就是one-hot编码，表示是哪一个事件



<font color=orange>视频编码器</font>

视频编码器骨干和双向GRU组成，以此长期视觉时序依赖



<h3><font color=green>事件定位器</font></h3>

**LCL**

利用得到的视觉特征，采用全连接层然后走一个sigmoid激活，实现二元分类

输出的是：

$(\hat{p_1},..,\hat{p_N})$ ,N是视频的帧数，每一个对应事件发生与否的概率（GT:0/1)

${\hat{p_1},...,\hat{p_N}}=Sigmoid(LCL(F_emb))$



<h3><font color=purple>多标签事件分类器</font></h3>

**MLC**

$\hat{E_i}=Sigmoid(MLC(f_i))=[\hat{e_{i,1}},..,\hat{e_{i,K}}]$

K个事件，针对在它第i帧得到的视频特征，去看它是否属于某个事件





<h3><font color=red>上下文模块</font></h3>

**CTX**

需求上是，网球比赛上从开节奏视频提取细微视觉特征，由于有动态模糊等因素

单纯靠前面的方法会导致重要视觉细节的丢失，产生不合理的事件序列

引入上下文模块，同时学习事件序列的知识



CTX使用双向GRU来处理事件序列 $\hat{E}$







<h2><font color=purple>数据集F3SET</font></h2>

![F3SET(2)](../论文阅读笔记/img/F3SET(2).png)

**标注流程**

1.视频切割：用一个工具自动将视频剪切成多块

2.切片筛选：包含感兴趣的内容筛选出来

3.F3事件标注：识别精细时间时刻

并且为标注开发了一个标注工具：

![F3SET(3)](../论文阅读笔记/img/F3SET(3).png)

**F3**:fast,frequent,fine-grained





<h2><font color=purple>消融实验</font></h2>

![F3SET(4)](../论文阅读笔记/img/F3SET(4).png)

若去掉用于建模时间依赖的 GRU,结果明显变差



**Multi-class versus multi-label classification.**

将 1000+ 种事件类型视作单一多类别分类，容易导致数据极度稀疏且训练困难；改为先拆解为多个子类别维度，再以多标签方式进行预测，不仅更自然，也能有效缓解长尾分布问题，性能得到提升

**CTX**

引入 CTX 模块后，通过对整个事件序列进行再次修正

