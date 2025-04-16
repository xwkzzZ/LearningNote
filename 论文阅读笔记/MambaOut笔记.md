<h1>MambaOut笔记</h1>



[Man!What can i say,Mambda out](https://arxiv.org/pdf/2405.07992)

2025CVPR Accepted 伟大，无需多言！





<h2><font color=blue>1.Introduction</font></h2>

attention机制 随着序列的增长复杂度而二次增长



为了让复杂度呈线性增长，一系列token mixers：

Linformer,Longformer,Big bird,Performer,动态卷积



RWKV和Mamba等模型已被证明能够有效地作为大型语言模型的主干网络



Mamba 的 token mixer 是结构化状态空间模型（SSM）

然而，实验结果表明，在视觉领域基于 SSM 的模型实际上在性能上不及最先进的卷积



所以讨论 视觉上到底是否需要mamba?



**Mamba 最适合具有两个关键特征的任务：长序列和自回归，因为这得益于 SSM 内在的 RNN 机制**

![MambaOut(1)](../论文阅读笔记/img/MambaOut(1).png)



金言：

*Autoregressive* characteristic,

on the other hand, demands that each token aggregate information solely from preceding and current

tokens, a concept denoted as *causal mode* for token mixing

自回归特征要求每个 token 只能聚合来自之前和当前 token 的信息，这一概念被称为 token mixer 的因果模式



所有视觉识别任务都属于理解范畴而非生成范畴，即模型可以一次性看到整张图片。因此，在视觉识别模型中对 token mixer 施加额外的因果约束可能导致性能下降

![MambaOut(2)](../论文阅读笔记/img/MambaOut(2).png)



mambaout模型堆叠的Gated CNN（左），mamba block（右）

![MambaOut(3)](../论文阅读笔记/img/MambaOut(3).png)





<h2><font color=red>What task is Mamba suitable for?</font></h2>



Mamba的token mixer是选择性SSM

$\bar{A}=exp(\Delta A),\bar{B}=(\Delta A)^{-1}(exp(\Delta A)-I)\cdot \Delta B$



$h_t=\bar{A}h_{t-1}+\bar{B}x_t$

$y_t=Ch_t$

$x_t$ 是输入， $h_t$ 是隐藏状态，$y_t$ 是输出



尽管SSM的递归特性使得Mamba能够高效处理长序列，但它也带来了一个显著限制：hₜ只能获取前一时刻及当前时刻的信息

总之，Mamba非常适用于满足以下两个特征的任务：
 • 特征1：任务涉及处理长序列。
 • 特征2：任务需要因果token混合模式。



<h3><font color=green>判断长序列任务标准?</font></h3>

对于一个输入 $X\in R^{L\times D}$

其中L是序列长度，D时维度

省流：

当

$r_L=\frac{L}{6D}$

大于1是，则视为长序列



<h3><font color=red>总结</font></h3>

**1.对于图像分类，引入SSM不合适**

**2.对于图像检测，图像分割任务，引入SSM有很大潜力**



<h2><font color=purple>实验验证</font></h2>

![MambaOut(4)](../论文阅读笔记/img/MambaOut(4).png)

观察表格，主要是最后几项，去除SSM的MambaOut模型和加上SSM的其他几个模型

反正去掉后准确率更高

（imageNet)



对于目标检测COCO，实例分割ADE20K

加入SSM的mamba则不然

![MambaOut(5)](../论文阅读笔记/img/MambaOut(5).png)

![MambaOut(6)](../论文阅读笔记/img/MambaOut(6).png)



<font size=6>总结省流版</font>

Mamba机制适合长序列，自回归
