<h1>videotree笔记</h1>



[原文](http://arxiv.org/pdf/2405.19209)

**training-free**



<h2><font color=red>方法</font></h2>

![videotree(1)](../论文阅读笔记/img/videotree(1).png)



<h3><font color=red>Adaptive Breadth Expansion</font></h3>

**固定长度记忆的局限性：**不同视频的信息密度差异较大——有些视频包含大量场景变化，而有些则基本保持静态



构建树的一层操作：

**visual clustering:** 

使用K-means聚群方法，将视频帧压缩成K个群

$(C_1,...,C_k),(c_1,...,c_k)=K-Means((f_1,...,f_n),k)$

$C_i$ 是第i个群聚集的所有帧,$c_i$ 是该群的中心向量(centroid vector)



**cluster captioning：**  

对于每一个群，找到这个群的质心向量$c_i$,然后找到和它最接近的视频帧特征$F_i$,然后将它给VLM-based captioner Cap(.)，以此将它转化为文本描述

$t_i=Cap(F_i)$



**relevance scoring**

使用LLM，将所有的t和提出的Q给它，然后得到每个t的相关分数$r_i$

给出的r不是分数，而是三个级别

*prompt:*

![videotree(2)](../论文阅读笔记/img/videotree(2).png)

以此自适应提取视频信息



注意，它提取视频帧是一个一个cluster群提取的，所以考虑是否加入迭代，也是对整个cluster进行操作



它设置两个超参 rel_num_thresh，如果得到的r超过这个值，那么加入，否则不要

同时max_breadth,群数k不得超过它



<h3><font color=green>Relevance-Guided Depth Expansion</font></h3>

有的视频信息密度高，对于回答很重要，需要更多细节

所以构建一个分级存储结构



对于得分r为第2级别的"somewhat relevant"的群

将它的第一层级重新聚合为w宽的子群，以此来吸纳更多的帧



对于得分r为第1级别的"highly relevant"聚群，将它结构调整为两层级

分支宽度为w，保留第一层的聚群信息



```
就是说，对于每个层级，如果相关性不够，则拓宽吸纳更多视频特征，若果相关性够，那么向下扩一个层级
```



<h3><font color=purple>LLM Video Reasoning</font></h3>

遍历树，收取关键帧，然后给captioner来获得注释，然后按照时间顺序，排序注释，再拼接在一起，最后将它和问题一起给LLM



<font size=5>**算法总览**</font>

![videotree(3)](../论文阅读笔记/img/videotree(3).png)



```
总结发现：对于内容筛选，压缩信息量的过程只有在第一步：K-means cluster

其他加入的信息都会保存，不做压缩，丢弃
```





<h2><font color=green>表现</font></h2>

**数据集**

*EgoSchema*:长视频问答，5000多选问题，验证集平均时长3分钟

*NExT-QA*：专注推理和时间的视频问答，包含5440视频，平均长度44秒

*Video-MME*：最近的综合性视频评测，在long-term-videos子集上进行测试，平均视频44分钟，范围在30-60之内





对于EgoSchema和NExT-QA基准，1FPS；对于Video-MME，0.125 FPS

![videotree(4)](../论文阅读笔记/img/videotree(4).png)

![videotree(5)](../论文阅读笔记/img/videotree(5).png)





<h2><font color=blue>相关内容等</font></h2>

VLM开始引入对视频帧的结构化理解，以实现对场景上下文的紧凑高效识别。

通过从局部时序信息逐步构建高层知识（即先捕获所有低层细节再聚合的自底向上方式）来处理长视频。