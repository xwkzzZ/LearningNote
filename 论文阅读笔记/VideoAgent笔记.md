<h1>VideoAgent笔记</h1>



[原文](https://arxiv.org/pdf/2403.11481)



<h2><font color=red>方法</font></h2>

![videoagent(1)](../论文阅读笔记/img/videoagent(1).png)



<h4><font color=blue>初始状态</font></h4>

首先均匀采样N帧以此来实现”初步了解“

用VLM将N帧的视觉内容转化为文本描述，然后再将文本描述给LLM来初始化 $s_1$,对视频内容有一个初步了解



<h4><font color=green>决定下一步</font></h4>

action二选一：

1.回答问题

2.搜索新信息



*如何抉择：*

1.通过chain-of-thought让LLM基于当前的state状态以及问题来生成一个初步的回答，预测

2.让后将生成的初步回答，原始的问题让LLM进行自我反思self reflect，生成一个有三层级的置信分数：

（1）信息不充足

（2）仅有部分信息

（3）信息充足

3.然后根据置信分数来抉择action

**值的注意的作者提到的是**

采用这样的三步走的好处是：单步预测更容易取选择搜索更多新信息



<h4><font color=purple>收集新信息</font></h4>

根据已经看过的视频帧片段，将视频分割成多个片段

然后让LLM根据查询文本预测需要什么片段（缩小范围，不是整个视频去CLIP，而是LLM去根据已经看过的内容（将视频切割成多个段），然后缩小范围到某个片段中，然后再去CLIP）



然后再让CLIP在指定的片段，一帧一帧去对比查询文本和视频帧

然后再用这些去更新状态



检索步骤中使用CLIP在计算上是高效的

实验表明，CLIP的计算量不到VLM和LLM的1%。



<h4><font color=orange>更新当前状态</font></h4>

将检索到的新视频帧生成新的注释，然后根据时间顺序拼接上原本的注释，最后让LLM重新生成（更新）



**有趣的分析与解释**

*为什么要多轮分析*

1.一次性给太多帧会引入较多的噪声

2.LLM上下文窗口限制

3.帧太少也不行



<h4><font color=red>总结</font></h4>

![videoagent(2)](../论文阅读笔记/img/videoagent(2).png)

![videoagent(3)](../论文阅读笔记/img/videoagent(3).png)



<h2><font color=green>表现</font></h2>

![videoagent(4)](../论文阅读笔记/img/videoagent(4).png)

![videoagent(5)](../论文阅读笔记/img/videoagent(5).png)