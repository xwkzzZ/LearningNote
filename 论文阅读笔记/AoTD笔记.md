<h1>AoTD笔记</h1>



[原文](https://zhengrongz.github.io/AoTD/)





<h2><font color=red>方法</font></h2>

流程总览：

![AoTD(1)](../论文阅读笔记/img/AoTD(1).png)

区别点：

以往的videoqa是在数据集上指令微调，更端到端。AoTD更强调用CoT来训练



<font color=red>**agent-based系统来构建链式思维**</font>

之前STAR模型通过引入symbolic程序来直接将问题拆分为多个子问题，但是有个问题就是：

![AoTD(2)](../论文阅读笔记/img/AoTD(2).png)

如果需要训练它，所对应的适用的训练数据很少

(上图为AoTD)

所以AoTD会在拆解问题时，产生中间产物的时候，刻意去让它都能够被适用于现有的对口的VideQA数据集



<font color=blue>**agent-based视频问答**</font>

给定一系列候选的视觉模型以及LLM-based agent

agent会将问题拆解成多个子问题，拆解的方式是生成针对这个问题的python代码，然后再调用适用的视觉模型



流程总览：

![AoTD(3)](../论文阅读笔记/img/AoTD(3).png)



*链式思维验证：*

1）滤掉能够正确执行并回答到达正确输出的执行轨迹

对于多选项数据集：输出必须正确匹配

对于开放式数据集，prompt LLM来验证正确性

2）给LLM prompt，评价分析时候的逻辑连贯性以及有用性

直接给出二元回答 "YES"/"NO"

以此来决定高质量的CoTs能够进一步被蒸馏



<font color=red>**最后一步一步的蒸馏**</font>



$L=L_{label}+\lambda L_{rationale}=\sum_{j=1}^{N}l(Phi(V_j,q_j,p_s),\hat{y_j})+\lambda l(\Phi (V_j,q_j,p_s),c_j)$

其中 $c_j$ 是生成的CoT, $\hat{y_j}$ 是答案的GT ，$\lambda$ 是超参



<h2><font color=blue>表现</font></h2>

**指令微调**

利用QA数据以及生成的CoTs来进行微调原本的模型，称之为：
**LLaVA-NeXT-Video-AoTD**

同时，作为baseline，只用QA数据来微调的称之为：

**LLaVA-NeXT-Video-Instruct** 

<font color=green>*数据：*</font>

![AoTD(4)](../论文阅读笔记/img/AoTD(4).png)

值得注意的是，在开放式问答上，它的表现相较于原始模型是倒退的

![AoTD(5)](../论文阅读笔记/img/AoTD(5).png)

作者给出的解释：

1.原始模型在大量的图片上进行预训练，这是可能使它高指标的来源

2.开放式问答，用GPT评价本身就是存在偏差，不好评价





<h2><font color=green>相关工作</font></h2>



*视觉CoTs* 

有用MLLM直接生成CoT的，也有用tool-based依次生成任务的解决方案

局限性在于：

MLLM生成可能出错（结合自己的方法，他的意思应该是出错后不好去过滤，筛选，改进，是固定的）

tool-based方法计算成本相对较高



不过让 MLLM 直接生成 CoT更多停留于图像上



*类似的并行的工作*

同时还有用已有的数据集的标注来生成CoT

但并未引入agent





```
而作者和上面的区别是，CoT是python执行程序经过LLM得来的，它相当于执行python程序的一个传声筒，它不会直接影响答案的生成，因为答案的生成来自于python程序，但是LLM的传声筒，生成的CoT更类似于一个对于python的转化为好算loss的文本，间接训练生成python程序的链式思维，同时之后做的过滤操作也是在间接对生成的python程序做过滤
```



