<h1>StreamChat阅读笔记</h1>



[原文](https://openreview.net/pdf?id=JbPb6RieNC)





<h2><font color=red>方法</font></h2>



**亮点：** training-free

对任意类型、任意长度的视频进行实时推理，并支持多轮交互。

*可实现 32 帧/秒（FPS）的处理速度，比现有方法快六倍*

*文本生成延迟低于 0.9 秒*



`好好好！`



<h3>分层记忆</h3>

长期记忆和对话记忆组成



**帧选择性堆叠**

使用一个算法，计算相邻帧时间的动作向量(motion vector)，应该就是表示的两帧之间的差异

如果差异大于一个定义好的阀值，那么就纳入

形成一个用于暂存视觉特征的“视觉缓冲区”。$B_{vision}$



`我就好奇，这代码咋实现？不同的batch意味有的帧要被舍弃，有的要被编码，这咋批量处理？`



**短期记忆**

从视觉缓冲区 $B_{vision}$ 中选取一定数量 $C$ 的最新视觉特征，根据遗忘概率 $\sigma$ 模型在这些特征中做随机挑选，将挑选出的特征存储到短期记忆中

![StreamChat(1)](../论文阅读笔记/img/StreamChat(1).png)



**长期记忆**

<font color=red>重点来了!!!</font>

![StreamChat(2)](../论文阅读笔记/img/StreamChat(2).png)

看左下角：

它是对每个chunk的记忆片段作两种形式的提取总结：

1.文本标注生成文本总结 t

2.视觉上作k-means运算获得视觉总结（即对整个chunk作降采样操作） v

每个chunk的长度为固定的L，对于第一个chunk就称之为 $K_i$



看中间：

这样得到的视觉总结v和文本总结t，一个chunk一个chunk的形成长期记忆高层次结构（但是注意，它没有舍弃下面的视觉特征信息）而是保存



看右上角：

短期记忆其实就是一种global memory

Global memory的选取就是之前说的，按照视觉记忆的遗忘概率来采样选取



**对话记忆**

将问题和答案编码成一个上下文表征 $d_i$ 然后存储起来



**检索**

 从两方面检索：

长期记忆得到的t，短期记忆的v（它没有对长期记忆提取的视觉进行检索）

同时还会对对话记忆检索

检索算法：

遍历数据结构，不断更新相似度最高的，并最终返回

![StreamChat(3)](../论文阅读笔记/img/StreamChat(3).png)





**系统调度**

![StreamChat(4)](../论文阅读笔记/img/StreamChat(4).png)





<h2><font color=green>表现</font></h2>

对于streamchat有三种配置方式，slow, base,fast

就正如其名（无须多言）



评估方式三方面：

Eco:语义匹配

Coh:多轮对话连贯度,回答质量随轮次变化的波动量

RPD:提出问题到生成回答的等待时间

![StreamChat(6)](../论文阅读笔记/img/StreamChat(6).png)

![StreamChat(7)](../论文阅读笔记/img/StreamChat(7).png)





<h2><font color=blue>STREAMBENCH</font></h2>

主要数据来源包括 **EgoSchema** 和 **YouTube-8M**

306个视频，单个时长平均4.6min

六种任务：

目标搜索（Object Search, OS）

长期记忆搜索（Long-term Memory Search, LM）

短期记忆搜索（Short-term Memory Search, SM）

对话交互（Conversational Interaction, CI）

基于知识的问题回答（Knowledge-based Question Answering, KG）

简单事实（Simple Factual, SF）

![StreamChat(5)](../论文阅读笔记/img/StreamChat(5).png)



速度的来源：

1.Lucas-Kanade Optical Flow算法，不会对所有的视频帧都处理，节省了计算资源，相比于videollm:编码后决定是否给LLM,他这里直接从源头来节省

（阀值设定在0.55，稳32帧）



2.多线程处理

三个线程并行运算，流水线工作

