<h1>VITA阅读笔记</h1>



[原文](https://arxiv.org/pdf/2408.05211)



**VITA:**video,image,text,audio 全模态



<h2><font color=green>Motivation</font></h2>

*开源社区的空缺*：弄出一个和GPT-4o一样的全模态模型，开源的发展怎么强调都不过分

*以及先前的limitation:*

1.传统的音频交互需要一个定义好的唤醒词或者按钮

2.说到点上的一个总结：酶促交互都会被阻塞一旦模型还在生成



所以VITA克服了上面的局限性，并且可以中断模型生成，以人为先

贡献：

1.GPT-4o级别的全模态模型，弥补了开源社区的空缺

2.中英文都行，尤其中文这方面以往比较欠缺

3.开拓了无提示词唤醒，全程对话模型



<h2><font color=red>VITA</font></h2>

**模型总览：**

![VITA(1)](../论文阅读笔记/img/VITA(1).png)

双模型架构



<h3><font color=red>Visual Encoder</font></h3>

使用的InternViT-3ooM-448px来作为视觉编码器

接受 $448\times 448$ 的图片作为输入，如果分辨率更高，那么采 dynamic patching strategy来捕捉局部细节

视频当作一种特殊的很多图片

（每张图片经过2层MLP后提取到256tokens）



如果视频长度小于4秒，那么均匀采样4帧，如果视频长度4-16秒，那么每秒一帧，如果鲳鱼16秒，均匀采样16帧



有一点疑惑，值得注意的一点是：
他做了data concatenation操作，将上下文拼接到固定长度tokens(6K)

只对视频，文本做拼接。不过注意到，他拼接操作意味着将多个不相关的内容凭借在一起（前后问题与内容成独立的）

![VITA(2)](../论文阅读笔记/img/VITA(2).png)



他对此进行数据拼接的解释：

(1)带来更自由的输入形式以及更大更拓展的上下文（我觉得有点牵强，我觉得拼接到固定长度是会有损对于长度感知的灵活性，因为长度全都固定到6K tokens，我觉得如果真的有收益的话，应该来源于一个单个batch中有丰富的问题，这种单样本内的多样性带来收益）

(2)提高计算效率（乍一听好像有道理，长度全都对齐后，这样并行计算效率提高？有点不肯定）

(我猜测，更可能是不好训练，所以搞这么一通)





<h3><font color=red>Audio</font></h3>

编码：先进性 Mel Filter Bank block操作，这个是一个预处理操作，将音频转化为可以被后续4*CNN+24层 transformer操作的一种Mel频带

然后也是用两层的MLP来作为connector

使用ASR的范式去训练，对齐



<h3><font color=orange>个人总结</font></h3>

1.两个模型并行，如果是噪音也会给generation model，只不过是生成的EOS token之类的内容分。如果检测到人的query，会停止generation model的EOS 生成

2.实际上关于音频上还是比较大的去依赖已有的ASR,TTS的工具，如果进行中断，会把先前的history context进行保存

3.模型整体很大

4.应用的情景上是每一次提的问题都是离散的，训练上也是一批也有各色的数据。

5.没有显式的memory bank，无法主动进行响应。属于很适合QA，去思考解决问题的模型，但是实时响应，实时描述，主动生成上是没有这方面的功能的





<h2><font color=blue>Evalution</font></h2>

![VITA(3)](../论文阅读笔记/img/VITA(3).png)