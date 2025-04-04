<font size=8>GroundVQA</font>



[原文](https://arxiv.org/pdf/2312.06505)



<h1>方法</h1>



<font size=5>问题定义</font>

给定视频和一连串的问题，能够给出时间窗口以及回答

时间窗口由起始时间节点和截止时间节点来定义

回答可以是openQA和closeQA





<font size=5>多任务架构</font>

![GroundVQA(1)](../论文阅读笔记/img/GroundVQA(1).png)



**Visual-language encoder**

接受视觉特征以及文本嵌入，融合视觉文本信息



**Temporal question localizer**

采用GroundNLQ和ActionFormer相似的模型

由一个分类头和一个回归头组成

分类头输出每个时间戳的概率分布

回归头估计当前时间戳到边界的距离（while the regression head esti

mates the boundary distances from the current timestamp.）（听不懂思密达）



<font size=5>从叙述中生成QA</font>

Ego4D包括了大量第一人称视频的，每个都标注了细节，时间戳以及描述（每分钟平均13.2句话）

用这个来生成一个QA样本



**估计叙述的时间窗口**

时间窗口计算范围，按照EgoVLP的法子来：

文本描述都是由叙述语句以及时间戳组成

$\{(N_j,t_j)\}$

确定时间窗口的方式：

$T_j=(t_j-\frac{\beta_i}{2\alpha},t_j+\frac{\beta_i}{2\alpha})$

$\beta_i$ 是相邻的连续的描述之间的时间间隔

$\alpha$ 是整个视频的所有$\beta$ 的平均， 这两个参数完全由数据集的统计来决定



<font size=5>生成OpenQA数据</font>

单独的描述内容太少了，决定将一些描述拼在一起

按时间顺序，将一个视频的叙述切割成多份，每份至少5句话或者最长持续30秒

然后将这一份份的叙述送给LLM，生成QA对以及时间窗口

![GroundVQA(2)](../论文阅读笔记/img/GroundVQA(2).png)



以此生成了*EGOTIMEQA*数据集：5389第一人称视频 以及303K样本（原文:samples，他的意思应该是对应的QAT标注）



<font size=5>生成CloseQA数据</font>

用同样的方式应用到EGOTIMEQA（刚刚生成的）和QAEGO4D（已有的）来生成CloseQA,但是要经过一个筛选的过程：

将QAEGO4D测试集中能够无视频上下文的情况下回答的问题筛掉：

train一个纯文本处理模型来辨识并且去除可以在10个不同种子下测试能正确回答的QA对

同时还人肉检验，剔除掉样本中包含不正确答案以及时间窗口的

这样生成了：

*QAEGO4D_Close*



<font size=5>多任务训练</font>

模型需要解决三个任务：

CloseQA,QA,Close,VLG(video language grounding)



$L_{VLG}=L_{focal}(T,\hat{T})+L_{DIoU}(T,\hat{T})$

$L=0.5\times L_{VLG}+0.5\times L_{QA}$



```
灵光点+1
L不局限于QA，引入新的loss，针对某个局部，组建，构建针对它的任务，引入对应的新的loss，以此提高整体的表现,先前的GCG如此，这个也是如此，以及很多。但是往往需要生成配套对应的训练数据

创造对应某个局部任务的训练数据，引入主线任务的支线任务，2个loss
```





<h1>介绍</h1>

（introduction+related work)



**局限性：**

先定位在回答问题的局限性在于：误差会逐步累积，因此效果不佳



所以提出同意训练：让问答和定位同步进行

这样减少了误差积累

同时为了将这两个任务串联起来，引入LLM来生成所需的数据集

在QAEGO4D以及Ego4D-NLQ上SOTA



关于数据集：

Ego4D-NLQ 数据集包含配有自然语言查询的长时第一人称视频

NaQ 数据集在Ego4D之上将叙述重新作为查询进行利用，进一步扩展了 NLQ

然而，这些叙述并不能直接应用于问答（QA）任务。（Huh?)(等下去看看NaQ)





<h1>表现</h1>



<font color=red>**QA Baselines**</font>

<font color=blue>*BlindVQA*</font> :纯语言模型（这里是T5-Base)

<font color=blue>*SimpleVQA*</font>:引入了视觉能力，视觉特征被映射到语言空间然后和embedding连接在一起

<font color=blue>*SimpleVQA+*</font>:为语言模型解码器的cross attention加入ranking loss，使用视觉语言定位的监督来突出模型在问题相关视频上的注意力，但是它不能预测时间窗口，效果也不如GroundVQA

<font color=blue>*Rehearsal Memory*</font>:将长视频压缩为一个固定长度的memory。



为了公平比较，视频特征都做统一处理后再来比较



**消融实验**:证明在抛去因提取到特征不同的情况下，作者提出的这种大体框架的有效性：

![GroundVQA(3)](../论文阅读笔记/img/GroundVQA(3).png)

SimpleVQA+在引入VLG任务后，效果反倒会下降。证明视频文本定位不是能靠一个Cross-attention来解决的



<font size=5>训练</font>

将视频文本定位任务和问答任务两个任务一起训练的效果要好于分开训练的效果

（笔记：可能一起训练可以带来模型的协同效应（自己编的术语））

![GroundVQA(4)](../论文阅读笔记/img/GroundVQA(4).png)



**SOTA**

![GroundVQA(5)](../论文阅读笔记/img/GroundVQA(5).png)

![GroundVQA(6)](../论文阅读笔记/img/GroundVQA(6).png)



<h1>局限性与展望</h1>

1.细粒度感知任务中仍面临挑战(复杂环境下的物体识别和计数),采用以对象为中心的特征（如跟踪和计数技术）可能有助于提升这些任务的表现。	



2.处理长时第一人称视频需要大量的计算资源。未来的研究应探索利用记忆网络来压缩视频特征



3.一个查询可能关联多个视频片段，而现有的 Ego4D-NLQ 和 QA-Ego4D 数据集均假设每个问题仅有一个相关片段。
