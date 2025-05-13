<h1>Flash-VStream笔记</h1>



[原文](https://arxiv.org/pdf/2406.08085)



<h2><font color=red>方法</font></h2>

**总览：**

![Flash-VStream(1)](../论文阅读笔记/img/Flash-VStream(1).png)



**3.2 Spatial-Temporal-Abstract-Retrieved memory**

*memory组成：*

<font color=green>*Spatial memory：* </font>使用队列结构，提取fine-grained空间特征

Shape: $M_{spa}\in R^{N_{spa}\times P^2_{spa}\times D}$



<font color=blue>*Temporal memory：*</font>长度固定，压缩的方式是K-means cluster算法

shape:$M_{tem}\in R^{N_{tem}\times P^2_{tem}\times D}$



<font color=orange>*Abstract memory:*</font>通过semantic attention模块来提取更高层次的语言信息

具体算法流程：

![Flash-VStream(2)](../论文阅读笔记/img/Flash-VStream(2).png)



<font color=purple>*retrieved memory:*</font>

检索检索，检索的是原本被压缩一个群一个群的时间特征被压缩的细粒度特征

在空间特征中检索来补充回去

![Flash-VStream(3)](../论文阅读笔记/img/Flash-VStream(3).png)

**组件总览：**

![Flash-VStream(4)](../论文阅读笔记/img/Flash-VStream(4).png)



**流程总览：**

![Flash-VStream(5)](../论文阅读笔记/img/Flash-VStream(5).png)

**3.3 Real-time LLM decoder**

将所有的memory内容通过embedding和文本embedding一起给LLM实时解码





随后构建了新数据集VStream-QA

每个视频平均时长40分钟，总共21小时



VStream-QA-Ego包含来自Ego4D数据集的10段1小时自我中心视频及1.5K个问答-时间戳三元组；VStream-QA-Movie包含来自MovieNet数据集的22段半小时电影片段及2K个三元组







<h2><font color=green>表现</font></h2>

**模型横向对比:**

![Flash-VStream(6)](../论文阅读笔记/img/Flash-VStream(6).png)

**消融实验：**

并非memory开得越大越好，而是注重配比调和

![Flash-VStream(7)](../论文阅读笔记/img/Flash-VStream(7).png)

![Flash-VStream(8)](../论文阅读笔记/img/Flash-VStream(8).png)