<h1>VideoChat-online笔记</h1>



[原文](https://arxiv.org/pdf/2501.00584)



<h2><font color=blue>1.Introduction</font></h2>

**离线和在线视频阐明：**离线处理是指模型在视频完全捕获后进行分析，而非在接收帧时实时响应。



**对于先前online的不足：**

1.时间视角局限（本应该有past,current,但是有些model就只支持一种）

2.回答不够精细

3.能够应对无限长视频的有效存储结构

（其他其实感觉......)



<h2><font color=green>2.Related work</font></h2>

**对于其他online model的锐评：**

1.VideoLLM-Online:每一帧的视觉输入受限。没有高效的streaming context压缩



VideoLLM-MOD配合mixture of depth解决这个问题，以此能高质量视觉输入



2.Flash-Vstream,VideoStreaming:通过可学习的memory module来实现stream compression



<h2><font color=orange>OVBench</font></h2>

**总览：**

![VideoChat-online(1)](../论文阅读笔记/img/VideoChat-online(1).png)

**详情：**

![VideoChat-online(2)](../论文阅读笔记/img/VideoChat-online(2).png)



<h2><font color=red>4.Modeling</font></h2>

![VideoChat-online(3)](../论文阅读笔记/img/VideoChat-online(3).png)



**4.1 Pyramid Memory Bank**

memory bank有n层，每一层逐渐减少空间细节，强化时间模式(pattern)，通过*sampling rate* 和 *resolution* 两个因素来调控

*sampling rate*  ($r_i$) 指的是采样率，层数越深采样率越高，越去关注时间连续性

*resolution* ($Res_i$) 层数越深，分辨率越低，越去关注时间上的特征，层数越高，分辨率越高，越去关注空间上的信息 $Res_i=\frac{Res_1}{\beta^{i-1}}$  初始化$\beta =2$



Memory bank的操作方式就是所想的：读入，每一层按照对应的 $r_i$,和 $\beta_i$ 来进行写入该层的Memory bank。

如果满了，那么就找相似度最高的两帧，然后根据上一层的 $Res_{i+1}$ 使用AvgPool2d采样到过去。然后再删除这一帧。最后读出，按时间顺序读出



*兼容KVCache*:做了tokeny压缩操作后，会删除对应的KV-Cache（直接删除，可能确实不好到KVcache去更新吧）



<font color=red>注意:</font>每一层的Memory bank是按照所对应层的sample rate去从streaming video input的输入，这个是每一层的memory bank与Input之间的关联（同样的 $Res_i$ )



**4.2 Offline-to-Online Leanring**

离线数据转化为在线数据：

1.将问题按特定时间间隔插入视频对话中。

2.每个样本中的提问均保持相同的任务类别，以便模型在跨轮对话中进行时序推理；

3.为每种标注任务生成 5 种多样化指令，以覆盖更全面的交互场景。



*训练：* 先使用离线视频数据进行基础训练，夯实模型的视频理解能力；随后引入在线数据进行联合优化

###### 

<h2><font color=purple>5.Experiments</font></h2>

**5.1. Implementation Details**

InterViT-300M作为视觉编码器

Phi-3作为大语言模型

视频输入全部1fps

总长控制在64帧



sample rate:{1,2,8}

token number:{256,64,16}





**5.2. Main Results on OVBench**

![VideoChat-online(4)](../论文阅读笔记/img/VideoChat-online(4).png)