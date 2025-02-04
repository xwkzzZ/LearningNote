<font size=8>Visual Prompts笔记</font>

[原文](https://arxiv.org/pdf/2407.04681)



**Abstract**

文本在明确传达细粒度或空间密集信息（如掩码）方面存在固有的困难

限制了它们回答需要理解详细或局部视觉元素的问题的能力。



所以：
将细粒度的外部知识（从专门的视觉模型（如实例分割/OCR模型）中获取）整合到MLLMs中。

<img src="../论文阅读笔记/img/Visual Prompts(1).png" alt="Visual Prompts(1)" style="zoom:50%;" />



<font size=5>**3 Proposed Method**</font>

**3.1 Auxiliary Visual Prompt with External Knowledge**

<img src="../论文阅读笔记/img/Visual Prompts(2).png" alt="Visual Prompts(2)" style="zoom:50%;" />

$\{M_{j},C_{j}\}_{j=1}^{N_{s}}=f_{seg}(I)$

$\{B_j,T_j\}_{j=1}^{N_{O}}=f_{ocr}(I)$

通过现成的segmentation model以及OCR model获得mask regions对应类别以及OCR边框及文本

然后将segmentation model生成的class以及OCR model生成的文本输出给文本编码器

$T_{s}=\{t_1,...,t_{N_s}\}=\{f_{text}(C_1),...,f_{text}(C_{N_s})\}$

$T_{o}=\{\hat{t_1},...,\hat{t_{N_s}}\}=\{f_{text}(T_1),...,f_{text}(T_{N_s})\}$

然后为了获得像素级的visual prompt，现创建一个零张量再填充：

<img src="../论文阅读笔记/img/Visual Prompts(3).png" alt="Visual Prompts(3)" style="zoom:50%;" />



**3.2 Visual Prompt Infusion**

<img src="../论文阅读笔记/img/Visual Prompts(4).png" alt="Visual Prompts(4)" style="zoom:50%;" />

<font color=blue>PEN</font>:prompt embedding network

这里的*PEN*是采取三个卷积然后卷积之间加ReLU

这样就有了：
$F_v=f_{MLP}(f_{img}(I))$

$F_p=f_{PEN}(P)$

将image tokens和processed auxiliary visual prompt缝在一起有两种方法：
**feature fusion:** $\hat{F_v}=f(Concat(F_v,F_p))$ f是线性层**feature addition:** $\hat{F_v}=F_v+F_p$



<font size=5>**4 Experiment**</font>

*feature addition* 更有效：

<img src="../论文阅读笔记/img/Visual Prompts(5).png" alt="Visual Prompts(5)" style="zoom:50%;" />

SOTA:

<img src="../论文阅读笔记/img/Visual Prompts(6).png" alt="Visual Prompts(6)" style="zoom:50%;" />

Meme:

<img src="../论文阅读笔记/img/Visual Prompts(7).png" alt="Visual Prompts(7)" style="zoom:50%;" />