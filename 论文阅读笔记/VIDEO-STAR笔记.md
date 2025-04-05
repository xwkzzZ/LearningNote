<h1>VIDEO-STAR笔记</h1>



[原文](https://openreview.net/pdf?id=JYV2hrtFSv)



<h2><font color=red>模型</font></h2>



整体流程：

![VIDEO-STAR(1)](../论文阅读笔记/img/VIDEO-STAR(1).png)



在给定数据集（包含视频和标签），模型要为之生成问题以及CoT答案（包含推理以及最终答案）



给定一个模版prompt:

A video is labeled *{*L*}* for the task of *{*T*}*. What questions could you ask

someone about the video that should contain the video labels in the response?



L是label,T是问题描述



模型训话优化，一边作数据增强，一边用增强的数据训练



相当于一个不断提纯 **推理链**的过程

先尝试直接给视频和问题，让它生成推理过程以及最终答案

1.<font color=green>(*answer generation*)</font>如果生成的内容没有正确回答问题那么就要转阶段label generation，如果正确回答了问题，那么它的**推理过程**以及最终**答案**将构成 $a_i$ ，构成训练数据集，在下一轮作指令调优(instruction tuning)，以此迭代模型

2.(<font color=blue>*label generation*</font>)

对于那些回答不出答案的视频，则让它作<font color=blue>*反推*</font>：给他视频，问题，答案，让它推理答案是怎么得到的。同时还有用label verification来筛选验证

3.(<font color=orange>*label verification*</font>)

会有一个解析器parser以及一个验证器verifier

解析器识别主题对象，并用正则表达式匹配字符串

验证器比较提取到的标签和gold labels(a grounding aspect of our datasets and represent some ground-truth knowledge)



比较的指标：IoU,BERT embedding的语句相似度

（由于它所标注的具体细节所指的附录内容以及表格内容在arxive，openreivew贴的原文都找不到，疑似缺失）

![VIDEO-STAR(2)](../论文阅读笔记/img/VIDEO-STAR(2).png)



<h2><font color=orange>数据</font></h2>

![VIDEO-STAR(3)](../论文阅读笔记/img/VIDEO-STAR(3).png)

(Avg dur的单位是秒)

![VIDEO-STAR(4)](../论文阅读笔记/img/VIDEO-STAR(4).png)

由以上三个数据集来生成新的训练数据集





<h2><font color=green>表现</font></h2>

![VIDEO-STAR(5)](../论文阅读笔记/img/VIDEO-STAR(5).png)



**Baseline**

*video-llava*

*video-llava+* ：使用简单的模版，通过视频的label来生成qa对，然后用它来训练video-llava

*$Vid-LLaVA^{Gemini}$* :使用Gemini 1.5 pro-vision来注释一小部分数据集，然后保留50%的videollava+的数据集，混入Gemini 1.5pro-vision生成的数据集，以此微调



（在后面的实验发现：Video-LLaVA+ 的表现未能较 Video-LLaVA 有明显改进，甚至在部分情况下有所下降，这表明不能直接利用带标签数据集进行 LMM 适应。）

![VIDEO-STAR(6)](../论文阅读笔记/img/VIDEO-STAR(6).png)

通过一系列的实验分析会告诉我们一件事：

VIDEO-STAR具有较强的领域适应能力



<h2><font color=blue>相关工作</font></h2>



**Large Vision-Language Models**

Maaz利用activitynet数据，首次扩展了大型视频指令调优数据集VideoInstruct-100K



但是存在问题：*质量不高*

生成的问题通常是视频标注的退化，较少涉及丰富的时空推理能力，这可能会限制 LMM 的整体性能。



**LLM和自我训练**

llm在扩展标注数据集上也遇到挑战，从而有了self-training和self-improvement的探索需求



指令调优数据生成与指令调优之间循环进行，通过不断迭代来提升模型性能。
