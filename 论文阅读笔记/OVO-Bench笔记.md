<h1>OVO-Bench笔记</h1>



[原文](https://arxiv.org/pdf/2501.05510)



<h2><font color=red>OVO-Bench</font></h2>

**3.1. Online Video Understanding Mode Taxonomy**

Online video可以被分为三种模式：

<font color=green>(1)向后追溯(backward tracing)</font>

$R_{t_0}=P(Q_{t_0},Q_{(-\infty,-T]}$

<font color=red>(2)实时感知</font>

$R_{t_0}=P(Q_{t_0},X_{(-T,t_0]})$

<font color=blue>(3)向前激活式响应</font>

$R_{(t_0,+\infty)}=P(Q_{t_0},X_{(t_0,+\infty)})$



其中R是指response

T是指某个时间节点的阀值boundary



*他们直接为上面三个设计了相关任务*



**总览：**

![OVO-Bench(1)](../论文阅读笔记/img/OVO-Bench(1).png)





<h2><font color=green>实验</font></h2>

![OVO-Bench(2)](../论文阅读笔记/img/OVO-Bench(2).png)

**主要结论：**

1.离线video-llms视频理解能力能够有效迁移到实时视频理解上



2.目前的video-llms都欠缺实时回答VQA task的处理时序优先次序的能力



3.video-llms的幻觉普遍存在



4.现有的video-llms需要更高效的推理架构来实现vqa





*个人观点：*

感觉这个数据集设计的很不行，offline模型好的全方面强于online，那要这样子，online所考虑的延迟响应，快速不全放屁？和着大伙就继续作offline就行了呗。