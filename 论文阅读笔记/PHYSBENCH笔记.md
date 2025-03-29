<h1>PHYSBENCH笔记</h1>



[原文](https://openreview.net/forum?id=Q6a9W6kzv5)



<h2><font color=orange>引言</font></h2>



PHYSBENCH包含10002条 视频-图片-文本 交叉的数据集

任务主要针对四大方面：

<font color=orange>*物理物体属性*，*物理关系*，*物理场景理解*，*物理动态*</font>

共19个子任务





实验结果：

当前大多数VLM对物理世界缺乏充分理解，尤其在物理场景和物理动态方面表现较差；闭源模型在这些任务上显著优于开源模型。

这主要是由于VLMs在训练语料和数据中缺乏必要的物理相关信息。当我们为模型提供基于真实物理的训练数据后，其表现会明显提高。





提出了PhysAgent来解决这类问题



<h2><font color=green>相关工作</font></h2>

**物理推理模型**

一类：物理特定模型，只能预测后续状态，无法拓展到其他任务中

另一类：只适用于一部分的任务，而且需要额外的训练模块，且是一个分类模型

**PhysAgent**不局限于固定的逻辑模板，并且在更广泛的问题范围内具备更高的适配性和通用性



**VLM** 一些研究发现大多数VLM在三维空间推理上仍有明显短板，其原因之一在于可供学习的相关数据不够充分。





<h2><font color=red>PHYSBENCH</font></h2>

![PHYSBENCH(1)](../论文阅读笔记/img/PHYSBENCH(1).png)

<h3>VLM的表现</h3>

![PHYSBENCH(2)](../论文阅读笔记/img/PHYSBENCH(2).png)

**总结：**

1.Video vlm表现似乎有点差强人意

2.物理场景 和物理动态的理解不足

3.模型大小与训练数据规模并未带来显著收益。



<h3>为什么</h3>

![PHYSBENCH(3)](../论文阅读笔记/img/PHYSBENCH(3).png)

1.主要为认知错误，其次是缺少物理芝士

![PHYSBENCH(4)](../论文阅读笔记/img/PHYSBENCH(4).png)

2.遍缺少对物理世界相关的先验信息或推理规则，因此当面对更复杂或高度依赖物理原理的问题时容易犯错。

在添加物理芝士例子后能够有效举一反三，表明：原始数据集缺乏物理世界知识导致这个不良表现



<h2><font color=red>PHYSAGENT</font></h2>

<font size=4>**三步走**</font>

1.*Task-specific Prompt Activation*

对问题进行分类，然后激活与该任务相关的提示，将不同任务所需的物理知识嵌入提示中

2.*Foundation Models Integration*

处理视觉基础模型的输出，利用VLMs的推理能力

3.*CoT*

![PHYSBENCH(5)](../论文阅读笔记/img/PHYSBENCH(5).png)



非常非常非常灰常灰常有意思!



<font size=4>**结论**</font>

1.纯文本提示不稳定，有时很灾难

CoT鸡肋

Desp-CoT和PLR帮倒忙

2.ContPhy（2024ICML,一个专门针对物理，视频的模型），3/4的任务表现都不如基础模型gpt4o

模块调用不佳和逻辑模板灵活性不足，难以适应各种场景

3.PHYSAGENT在0样本推理上稳定发挥

