<h1>AdaCM2笔记</h1>





<h2><font color=orange>观察</font></h2>



<h3><font color=green>帧之间cross-attention的稀疏性</font></h3>

**1.在一帧内，只有一部分的视觉token表现出与文本query的高相关性**

![AdaCM2(1)](../论文阅读笔记/img/AdaCM2(1).png)

![AdaCM2(2)](../论文阅读笔记/img/AdaCM2(2).png)

<h3><font color=purple>层级别的cross attention相似度</font></h3>

![AdaCM2(3)](../论文阅读笔记/img/AdaCM2(3).png)

最近五帧之间的余弦相似性超过90%

深层中的相似性比浅层更为显著。

视频中持续存在的冗余以及不同层间冗余的多样性

**2.不同层的相关性多样**



<h2><font color=red>方法</font></h2>



![AdaCM2(4)](../论文阅读笔记/img/AdaCM2(4).png)

###### 

**和MA-LMM的区别**

在原有的基础上，对记忆压缩策略进行改进：

首先它会计算每个视频特征与所有文本之间的注意力

$S_t^c(i)=\sum_{j=1}^{j=N}S_t(j,i)$



然后他将visual memory bank分为两个区域，一个是历史区域，一个是最近区域

然后设置一个超参数 $\alpha$ 决定二者的配比（历史记忆和最近记忆）

每次有新的视觉特征进来的时候，会先对记忆库会按照配比 $\alpha$ 进行划分



每次压缩记忆库，只对历史区域进行操作，操作方式就是只保留 $s_t^c$ 得分前 $\beta$ 的特征（$\beta$ 也是超参数）

之前在第三章说了，不同层的冗余程度具有多样性，所以作者说：为每一层设置了不同的 $\alpha$  $\beta$ 

（也就是说，他至少得设置**22个**或者**24个**超参，原MA-LMM是11层，他的图上看着是12层）



##### ![AdaCM2(5)](../论文阅读笔记/img/AdaCM2(5).png)





<h2><font color=green>表现</font></h2>

![AdaCM2(6)](../论文阅读笔记/img/AdaCM2(6).png)

提升最多的是LVU数据集



COIN提得最少

原本的activity net-qa没有摆出来
