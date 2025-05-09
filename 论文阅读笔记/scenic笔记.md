<h1>scenic笔记</h1>



[原文](https://openaccess.thecvf.com/content/CVPR2024/papers/Zhou_Streaming_Dense_Video_Captioning_CVPR_2024_paper.pdf)



<h2><font color=red>方法</font></h2>



**视觉编码器**

采用图像编码器，但是并非简单的一张题一张图编码后得到视觉特征然后堆叠，以此构建整个视频的视觉特征

而是会引入一个 记忆机制以因果推理方式处理特征



**文本解码**

采用自回归的方式

给定视觉特征 $f$,文本前缀token(textual prefix tokens)，以及先前的所有生成的word $w_{1:i-1}$

这样生成：（D是文本解码器）

$w_i=D(f,p,w_{1:i-1})$



(作者提到:Note that prefix tokens are typically not used in captioning tasks, but are used in question-answering (QA) to encode the input question)



*额外的芝士引入：*

- **前缀词符（Prefix Tokens）**：是预先拼接到解码器输入序列前端的固定或动态生成的词符（token），作为生成后续文本的“引导条件”。在本文中，前缀可能包含：
  - **历史预测的文本**：例如之前解码点生成的事件描述（如图4中蓝色部分）。
  - **结构化信息**：如时间戳（`[start:1.2s, end:3.5s]`）或任务指令（如“生成烹饪步骤”）。
- **作用**：类似于语言模型中的“prompt”，通过前缀提供上下文，控制生成内容的连贯性和一致性。





**带时间戳的视频标注**

Vid2Seq引入时间token，$w^s$, $w^e$ 分别表示开始，结束时间，这样一个单独的事件可表示为：

$c^{'}=[w^s,w^e,w_1,...,w_n]$



局限性：

1.视觉特征f是整个视频传入解码器，扩展性不行

2.一次性预测所有事件描述，难以生成长而详细的描述



<h4><font color=red>Streaming inputs using memory</font></h4>

直接将视觉特征输入序列太大，一般会去降采样

但是降采样会丢失细粒度特征

于是使用memory mechansim来处理



固定内存大小K，初始化为取前K

使用K-means-like群聚算法，不断更新memory，同时为了防止群聚中心快速偏移到刚刚输入的特征。

将原先的cluster作为一个momentum weight

![scenic(1)](../论文阅读笔记/img/scenic(1).png)

**逐句讲解：**

$pairwise_l2_distance(X,M_t)$:是在计算新的 $K+N_f$ 新的加老的token与原来K个token之间的L2距离，构成一个 $(K+N_f,K)$ 的矩阵，计算其距离

$\delta \leftarrow d.argmin(axis=1)$ ：是在原本的距离矩阵 $(K+N_f,K)$ 每一列的最小值，也就是新的$K+N_f$个token到老的K个token的最小距离



$\delta\leftarrow make\_onehot(\delta,K)$:转化为one-hot编码



$W_t\leftarrow \delta^T W_t$：原本的 $\delta$ shape是 $(K+N_f,K)$, 转置矩阵让映射关系为：K个token根据delta的映射关系映射到 $K+N_f$ 中



$A\leftarrow \delta^T /W_t$ 因为 $\delta$ 是one-hot编码，0或1，取0无效映射，1有效映射关系，所以这么一除，让不对应的映射关系置0，对应的置1



$M_t\leftarrow AX$ ：更新记忆



![scenic(2)](../论文阅读笔记/img/scenic(2).png)





<h4><font color=green>Streaming outputs with decoding points</font></h4>

![scenic(3)](../论文阅读笔记/img/scenic(3).png)

首先进行均匀采样，得到所谓的解码点：也就是生成时间caption的时间节点，默认为这个时间节点到上一个解码点时间节点，这一段时间包含一个事件



整个解码生成的过程，遵循stream方式

$Y_i=\{(w_j^s,w_j^e,c_j|w^e_j<=d_i\}$

其中 $c_j$ 指的是在 $d_i$ 时间点之前的所有时间注释

并且参考图4，**它的做法就是：**

将先前已近在某个解码点生成的内容作为解码前缀decoding prefix（就是每一次生成新的caption，输入包含了先前已近生成的caption以及一段未生成的caption注释)



*另外：*

在训练过程中，会随机从前缀中移除部分已生成的事件描述，将它们重新加入预测目标，以增强模型对早期预测错误的鲁棒性。



**总览：**

![scenic(6)](../论文阅读笔记/img/scenic(6).png)





<h2><font color=green>表现</font></h2>

**消融实验**

memory modules:



*CIDEr:*一种专门为评估图像和视频描述（如字幕生成）任务设计的自动化评价指标(通过对比模型生成的描述与多个人工参考描述的相似性来评估质量)



![scenic(4)](../论文阅读笔记/img/scenic(4).png)

如果No memory最多存16帧给decoder



其中提到的MovieChat是使用的类似MA-LMM的方法

固定K tokens大小来作memory存储（压缩方法是通过合并相邻两个tokens）



其他：

![scenic(5)](../论文阅读笔记/img/scenic(5).png)





