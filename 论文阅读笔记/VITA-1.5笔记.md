<h1>VITA-1.5笔记</h1>



[原文](https://arxiv.org/pdf/2501.01957)



<h2><font color=green>Motivation</font></h2>

在原本VITA的基础上，ASR,TTS部分模块不需要额外引入，而是端到端了



总的来讲，这篇paper的贡献点在：VITA的基础上只是让ASR,TTS完全融进VITA中，不依赖引入的工具，可以有效降低延迟的效果



<h2><font color=red>VITA-1.5</font></h2>

**作者原话：**相较于VITA的架构，VITA-1.5在输入端是一样的，但是输出端不用外部的TTS模块

![VITA-1.5(1)](../论文阅读笔记/img/VITA-1.5(1).png)

在原有基础上：

- 视觉编码器加入了dynamic patching的策略，可以捕捉局部的细节
- 以TiCodec作为Speech Decoder，以及对应一个1024的codebook

- 并且设置了两个decoder，所谓的一个Non-Autoregressive Speech Decoder来处理全局的text tokens以及语义特征，还有一个Autoregressive Speech Decoder 来细化，一步一步生成高质量的token 



<h3><font color=red>Why use duplex decoder?Bro</font></h3>

- 首先Non-autoregressive Speech Decoder相较于autoregressive speech decoder它是高度并行的，原本自回归的是一个一个Token进行预测，需要感觉先前的内容来生成当前的token,但是在NAR中，是一次性，一次迭代，并行计算出整个片段所有token
- NAR这样计算的好处当然是速度快，面对online这种任务。而这部分生成的东西就对应上了（这下听懂了）他所说的semantic feature，他就是一个拟稿。
- 然后再让autoregressive speech decoder去精细化的迭代。



不过有一说一，虽然这么做，还是希望有更充足的实验数据去表明这种双deocder的做法是可以降低latency者或提高生成质量的



**至于说后面他提到的训练阶段，比较了下和VITA，基本上八九不离十，只不过某些阶段的任务多加了些训练数据，其他大相径庭**

（主要多了个关于audio的training)

![VITA-1.5(2)](../论文阅读笔记/img/VITA-1.5(2).png)



<h2><font color=blue>Evalution</font></h2>

其实吧，不是很SOTA(video understanding那一块），主要实用性

![VITA-1.5(3)](../论文阅读笔记/img/VITA-1.5(3).png)