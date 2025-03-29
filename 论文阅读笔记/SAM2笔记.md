<h1>SAM2笔记</h1>



[原文](https://openreview.net/forum?id=Ha6RTeWMd0)



<h2><font color=green>3 TASK: PROMPTABLE VISUAL SEGMENTATION</font></h2>

可以在视频的任意帧提供prompt,并且在随后的整个视频都要能够定位并遮住

就这样子：



如果视频对象跟踪丢失，会在新帧去添加prompt

同时还可以SAM+视频跟踪：补充Prompt完了之后再让重新标注

![SAM2(1)](../论文阅读笔记/img/SAM2(1).png)



<h2><font color=red>模型</font></h2>

不是直接用图片编码，而是结合以往的历史，记忆库来编码

同时，prompt结果可能来自未来，预测结果会存储记忆库中然后供未来所用



![SAM2(2)](../论文阅读笔记/img/SAM2(2).png)

<h3>图像编码(image encoder)</h3>

为了流形编码，将输入进来的帧作一次性地编码。并且用MAE预训练的Hiera图像编码器（分层的），并在解码期间得到多尺度的特征



原文：`allowing us to use multiscale features during decoding.`



<font color=red>*什么是multiscale features during decoding*:</font>

AI回答：

```
这里所说的 “allowing us to use multiscale features during decoding”，指的是由于 Hiera 这类“分层式（层次化）”网络结构会在不同层级产生多种尺度（分辨率）下的特征表示，解码器（例如进行掩码预测的部分）就可以直接访问并组合这些不同分辨率的特征，从而在解码阶段（即生成分割掩码时）得到更丰富且更加精准的上下文信息。

具体而言，Hiera（Ryali et al., 2023; Bolya et al., 2023）属于一种层次化 Vision Transformer，它不像“平铺式”Transformer那样只输出单尺度特征，而是在网络的不同层次（Stage）产生多分辨率特征图。例如，某些层产生高分辨率、局部感知更精细的特征，而更深层次则会产生低分辨率但语义更抽象、更全局的特征。这样一来，在后续解码（比如掩码解码）时，模型既能从高分辨率特征中提取细节（如目标边缘信息），又能从低分辨率特征中获取更全局的上下文。这种结合多尺度特征的方式，有助于提高分割精度和对不同大小目标的适应性。

总结来说，“允许在解码阶段使用多尺度特征”意味着解码器可以并行／交替地访问这些层次化网络中各级特征，从而更好地融合局部细节与全局语义信息，以生成更准确、更鲁棒的分割结果。
```



<h3>记忆注意力(memory attention)</h3>

基于过去视频帧特征，对于新prompt的预测对当前的视频帧特征加入限定条件

将L个transformer块堆叠在一起

每个块先作self-attention，然后再跟memory bank中存储的内容（(prompted/unprompted) frames and object

pointers ）作cross attention，然后再跟上MLP

使用vanilla attention操作来执行上述操作



<h3>prompt编码和mask解码</h3>

Prompt被位置编码然后和学到对应类型的embedding相加

mask被卷积embedded然后和frame embedding相加



Mask decoder沿用SAM:堆叠两个transformer块以更新prompt以及帧embedding

（对于单个点击的prompt可能存在歧义的情况，输出多条掩码候选，如果后续没有提示来消除这种歧义，由模型自动选择当前帧上预测得分最高的掩码并继续传播。）





<h3>记忆编码器</h3>

提取mask,然后再用卷积进行降采样，然后再和image encoder得到的原始frame embedding进行逐元素相加

（作者刻意提出：**参考图这里没有画出来**）



<font color=purple>想法：</font>

卷积降采样，mask是一个块状性质的特征，同时没有细节要求，元素与元素之间的差异不关心，所以降采样获得全局特征，减少无关细节。与原始的frame embedding逐元素相加应该是将处理好的的mask再加回整个图片中



<h3>记忆力库</h3>

保存有过去的预测结果

空间信息上：通过开一个长度为N固定的队列存储帧信息，并且开一个长度为M固定的的队列存储prompt帧信息，这两部分构成空间特征图(spatial feature map)



semantic(语义)上：则是在mask decoder时会为每一帧输出一个object pointers的轻量级的向量来



Memory-attention则利用空间和语义的信息存储起来，做cross-attention



时间信息上：在存储帧的队列添加了时序编码





<h2><font color=blue>相关工作</font></h2>

<h3>VOS</h3>

Video object segmentation

视频第一帧用一个目标掩码作为输入，然后对该目标在视频全时域内进行精确跟踪



<h3>iVOS</h3>

Interactive video object segmentation

通过用户提供的标注（常见形式包括描线、点击或边界框），有效地在视频中获取目标的时空分割掩码（即 “masklet”）



