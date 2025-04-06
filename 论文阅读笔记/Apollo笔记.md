<h1>Apollo笔记</h1>



[原文](https://arxiv.org/pdf/2412.10360)

（重点和金句有太多了，有些直接copy翻译或者原文）





<h2><font color=gren>介绍</font></h2>

开头提出灵魂质问：(那我问你，那我问你)

1.视频应该如何采样？

2.哪种视觉编码器产生更优的表征？

3.对于视觉token最优的重采样的策略是什么？



现在的方法：

长上下文窗口(longer context windows)，多模态融合(multi-modality mixing)，agent工作线(agent workflow)，自训练(self-training)



但是上面的方法都难以理解它的表现（可解释性），现在，你meta爷来啦！



<h2><font color=blue>目前benchmark有效性的讨论</font></h2>

对一个 30 亿（3B）参数规模的模型在这些基准上进行完整评估，就需要花费 **184 小时的 A100 GPU 计算**



分析了现有基准的质量及其相互间的冗余性

然后提出ApolloBench



**拷打benchmark的质量**

![Apollo(1)](../论文阅读笔记/img/Apollo(1).png)

很多数据集在纯文本或者单张图片的情况下都能达到一定的准确率（关键关键就在：文本准确率和图片准确率拉开的不大，同时视频准确率额和图片准确率拉开的也不大），说明一个问题就是：视觉考核不够，而且对视觉动态信息的考核更小！



下面的浅蓝色和黄色块代表两种模态之间感知所需要的差距，数值越高表示某一者越重要，越低代表有一个模态的信息更可有可无



而且！**随视频时长增长，模型对视频感知的依赖程度却呈现下降趋势**



**现有benchmark的冗余**

![Apollo(2)](../论文阅读笔记/img/Apollo(2).png)

数值越高，相似度越高，冗余度也就越高（对角线就是自己对自己）

而且同一个数据集的不同类型以及不同任务，（长视频，短视频）（多选题，标注匹配，标注生成），它们的相似度都很高，缺乏多样性



**引入ApolloBench**

主要使用了多选题的形式，以免在评测中依赖 ChatGPT 这样的外部大模型

1.先将那些仅靠纯文本或单帧图像就能被超过 50% 模型正确回答的问题过滤掉

2.基于视频的时序感知，将问题分为五大类：**Temporal OCR**、**Egocentric**、**Spatial**、**Perception** 以及 **Reasoning**

3.筛选出前400的好问题

4.人工核对

5.ApolloBench 能 **将评测时间减少 41 倍**



<h2><font color=orange>Scaling Consistency</font></h2>

需求：在海量数据和大型模型上对多种架构设计和超参数调优进行系统探索，几乎是不现实的



那么？！！

**是否可以先在小规模模型与数据集上，做出设计决策，然后将这些决策直接迁移到大模型与大数据集上？**





选取21种LMM的变体（架构、视频采样、训练策略以及数据混合（data mixtures）等关键要素上存在差异）

又设置了四个小LLM

分别是Qwen系列的0.5B,1.5B,4B和7B



这样得到84个模型和



当 LMM 的规模达到一个临界值（约 2∼4B 参数）之后，对应模型在小规模场景里所做的某些设计决策与更大规模模型（如 7B）呈高度相关

这也被称之为**Scaling Consistency**

![Apollo(3)](../论文阅读笔记/img/Apollo(3).png)



**Scaling Laws AND Scaling Consistency**

scaling laws要求在模型各个大小上训练以此来探究表现上的趋势，但显然，在LMM上计算成本大多扛不住

但是scaling consistency可以从小处着手，就表明小模型的表现可以有参考性地去设计大模型



![Apollo(4)](../论文阅读笔记/img/Apollo(4).png)

红色曲线指的是某个大小的模型和7B模型的相关性，等于1时就是完全一样，蓝色的同理

（我就好奇，如果往7之后画，比如13B和7B，又是咋样呢？）





![Apollo(5)](../论文阅读笔记/img/Apollo(5).png)

对比之下，**4B** 规模的模型在数据量达到 **50 万**（约 50 万样本）后，其与 7B 模型的 $R^2$ 基本达到了平台期；增加更多训练数据也收效甚微

这意味着:**当模型规模足够（约 2∼4B）时，使用 50 万样本规模的训练数据就能够为大模型提供高度相关的设计参考**



作者亲笔划的重点：

<font color=red>**Finding 1:** We discover Scaling Consistency, </font><font color=red>where design decisions can be made on smaller </font><font color=red>models</font><font color=red>and datasets and transfer reliably to larger ones.</font>



<h2><font color=red>探索video-LMM</font></h2>



<h3><font color=green>视频采样</font></h3>

采样方式有两种：

均匀采样（固定一定帧数，不随时间变化）

固定帧率采样（每秒采固定帧数）（也就是说随时间变化，帧数随之变化）



介绍完二者的区别后，直接来第二个结论：

<font color=red>**Finding 2:** fps sampling is preferable over uniform sampling during model training and inference.</font>



同时！！！！

作者提到，the token per second(tps)也是一个重要影响因素

总体上是符合fps越高，表现越好，tps越高表现越好

但是并非那么简单：

![Apollo(6)](../论文阅读笔记/img/Apollo(6).png)

fps所带来的提升很小，但是tps很大

同时作者指出：token per frame(tpf）,fps与tps存在权衡，取8～32整体较优



<font color=red>**Finding 3:** There is a trade-off between tps and fps, with 8-32 tokens per frame being optimal.</font>





作者额外提到：一些方法采用主动帧选择策略，使用初始查询来指导帧采样（需要每轮对话重新采样），没纳入研究



<h3><font color=blue>视频表征</font></h3>

早期思路是使用专门的视频编码器来捕捉时域动态

但也有人指出，图像编码器本身经过大规模图文预训练后，依然能够在时序融合方面带来意外的好效果



反正目前存在争议



目前很多研究表明：输入图像的分辨率影像要大于token的数量

做了一系列实验后得到：

<font color=red>**Finding 4:** SigLIP-SO400M is the best single encoder for video-LMMs.</font>

然后又做了一些列实验，尝试将图片编码器和视频编码器结合，在重采样前将它们在通道维度拼接

<font color=red>**Finding 5:** Combining SigLIP-SO400M with InternVideo2 leads to the best overall performance.</font>



<h3><font color=orange>视频token重采样</font></h3>

视频输出的embedding在维度上是低于LLM的隐藏层的，需要扩大2-4倍，向上映射

早期方法是直接向上映射所有的视觉token到LLM的空间，但是带来了很大的信息的重复冗余

之后有人证明：对图片token进行重采样(resampling)不会削减image-LMM的表现，对于video token而言，更重要

同时锐评了QFormer:虽然可以文本知道视频token重采样，但是在多轮对话上泛化性不足，token根据第一个问题被重采样



1. mlp up-projection + average pooling
2. 2D conv + average pooling
3. perceiver resampling

![Apollo(7)](../论文阅读笔记/img/Apollo(7).png)



什么是perceiver resampling(就是lava-mini的)

AI回答：

```
具体来说，该机制采用一组可学习的潜在向量（latent queries），这些向量并非直接从输入中提取，而是在训练过程中随机初始化并通过反向传播学习得到。它们通过交叉注意力（cross-attention）与输入数据（作为键和值）进行交互，从输入中提取最具代表性的特征。这一过程可以看作是一种“软聚类”：潜在向量充当了“聚类中心”，在对整个输入进行全局信息汇聚时起到了降维和提炼关键信息的作用。

这样一来，即使输入数据非常长（例如高分辨率图像、长视频或高采样率音频），模型主要的自注意力计算仅在较小的潜在空间内进行，从而避免了传统 Transformer 中自注意力操作的二次复杂度问题，实现了计算和内存消耗与输入长度的线性关系。
```



<font color=red>**Finding 6:** Perceiver resampling shows superior performance when reducing the tokens/frame.</font>



<h3><font color=purple>视频token融合</font></h3>

![Apollo(8)](../论文阅读笔记/img/Apollo(8).png)

<font color=red>**Finding 7:** Adding tokens (text, learned, etc.) </font><font color=red>between the video tokens derived from different frames</font><font color=red>or clips is sufficient for efficient token integration.</font>





<h2><font color=red>video-LMMs应该如何训练</font></h2>

**训练规划**

三阶段训练能够达到最优，紧随其后的是双阶段训练。需要注意的是，每个阶段所用的数据组合都不同：当LLM被“冻结”时，模型仅使用视频数据来训练；而当LLM被“解冻”时，模型则使用文本、图像、多图像和视频



<font color=red>**Finding 8:** Progressively unfreezing the </font><font color=red>different components in different stages leads to superior model</font><font color=red>training dynamics.</font>



**训练视频编码器**

大多数实验：当LLM冻住，模型只在视频数据上训练。当LLM解冻，使用文本，图片视频等混合数据。

研究发现：LLM和视频编码器同时解冻，在混合数据上训练，效果明显下降。如果只在视频数据上训练编码器，则产生更大收益



<font color=red>**Finding 9:** Finetuning video encoders on</font> <font color=red>only video data further improves overall performance,</font><font color=red>especially on reasoning and domain-specific tasks.</font>



**数据组成**

![Apollo(9)](../论文阅读笔记/img/Apollo(9).png)

结果表明，训练数据中包含 10%～14% 的文本比例对最终性能至关重要，可能是因为这能防止模型在后续训练中出现“灾难性遗忘”。如果文本占比高于14%（如25%）或低于7%，都不利于性能提升。在保证文本占比适度的情况下，将剩余比例略微向视频数据倾斜会更好，这让模型可以充分借助视频数据掌握丰富的时空信息，同时也能利用规模庞大且高质量的图像训练数据

<font color=red>**Finding 10:** Data mixture matters, and</font> <font color=red>including a moderate amount of text data</font><font color=red> and maintaining a</font><font color=red>slight video-heavy mix leads to optimal performance.</font>



<h2><font color=purple>Apollo</font></h2>

**构成**

采用了 Qwen2.5  系列不同规模LLM作为 Apollo 的主干(1.5B,3B ,7B)

选择 SigLIP-SO400M 编码器与 InternVideo2 视频编码器的组合来处理视频特征

每个编码器的输出先进行插值，再在通道维上拼接，最后通过 Perciver Resampler对其进行 32 tokens/frame 的重采样。



**训练流程**:三阶段训练模式。



**训练数据**

整理了来源多种公共数据的混合训练集

进一步扩充训练语料，由 LLaMA 3.1 70B 驱动的注释工具来生成多轮视频对话，

![Apollo(10)](../论文阅读笔记/img/Apollo(10).png)



<font size=5 color=green>表现</font>

![Apollo(11)](../论文阅读笔记/img/Apollo(11).png)
