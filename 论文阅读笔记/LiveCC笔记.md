<h1>LiveCC笔记</h1>



[原文](https://arxiv.org/pdf/2504.16030)



<h2><font color=red>3.Methodology</font></h2>

**数据集生成**

![LiveCC(1)](../论文阅读笔记/img/LiveCC(1).png)

先从四个数据集上为源数据，进行一波过滤，然后得到5.7M

然后分为预训练数据集(Live-CC-5M)和SFT(supervised fine-tune)数据集(Live-WhisperX-526K)，相比预训练数据集洗数据的流程，会更精细



SFT的数据集只包含7种类别：
![LiveCC(2)](../论文阅读笔记/img/LiveCC(2).png)



**建模**

训练时候的序列为：

![LiveCC(3)](../论文阅读笔记/img/LiveCC(3).png)

其中[Con]是video上下文信息(context)(prompt,先前的ASR，视频标题)

<F>是视频帧，<W>是words，字，t就是时间索引，k是时间间隔



在预训练的时候，没引入stream，主要实现单词级对齐（时间，单词，但只是一块时间一块时间的对齐，还没有精确到一个词一个词）

（语义与时间上的对齐）

在SFT阶段，就是使用WhisperX提供的精确单词级时间戳

EOS token是用省略号token

![LiveCC(4)](../论文阅读笔记/img/LiveCC(4).png)

在推理的时候，每240秒丢弃视觉token（SFT最长时间就是240秒），但是会保留文本tokens



<h2><font color=blue>4.LiveSports-3K Benchmark</font></h2>

![LiveCC(5)](../论文阅读笔记/img/LiveCC(5).png)

![LiveCC(6)](../论文阅读笔记/img/LiveCC(6).png)

![LiveCC(7)](../论文阅读笔记/img/LiveCC(7).png)

值的注意的是：

在评估caption的时候，这种开放式问题，是通过对比ChatGPT-4o来评估（同时ChatGPT-4o）又作裁判(看下2图)



<h2><font color=green>5.Experiments</font></h2>

![LiveCC(8)](../论文阅读笔记/img/LiveCC(8).png)

![LiveCC(9)](../论文阅读笔记/img/LiveCC(9).png)



<h2><font color=orange>2.Related Work</font></h2>

**Streaming Video Understanding.** 

*什么叫做offline：*

能够在做预测前一次性访问所有的视频帧





*目前的的局限性：*

很大程度依赖手工标注的数据或者GPT生成的数据，有捷径效应

（以及引言中提到的局限性：

数据集难以收集，要么依赖LLM从视频标注中生成幻觉势流势对话，

要么在小规模的caption数据集上做微调

这些方法都不足以去生成真正有效的streaming video LLMs)

