<h1>Vista-LLAMA笔记</h1>



[原文](https://arxiv.org/pdf/2312.08870)



<h2><font color=red>方法</font></h2>

总览：

![Vista-LLAMA(1)](../论文阅读笔记/img/Vista-LLAMA(1).png)







**传统注意力机制**

对于第j个query $q_j$ ，query和key之间的相似度，然后根据相似度应用value权重上，它的更新过程可以表示为：
$$
Attention(Q,K,V)_j=\frac{\sum_{i=1}^j sim(q_j,k_i)v_i}{\sum_{i=1}^jsim(q_j,k_i)}
$$
相似度通常这么算：
$$
sim(q_j,k_i)=exp(q_j^Tk_i\sqrt{d})
$$

```Chinese(doge)
难怪我之前的算法一坨屎，给我糖完了
```



**EDVT-Attention**

上面的传统算法没有位置信息，所以RoPE

引入旋转矩阵，然后在每个q和v乘上这个旋转矩阵

$R_j\in R^{d\times d}$


$$
Attention(Q,K,V)_j=\frac{\sum_{i=1}^jsim(R_jq_j,R_i,k_i)v_i}{im(R_jq_j,R_i,k_i)}
$$


所以作者对于幻觉的解释：

因为没有引入位置编码信息，导致一些很远的无瓜的信息被等影响力干扰导致产生幻觉



作者对于文本与文本之间用RoPE，文本和视觉之间不用



<font color=green>*算法：*</font>

![Vista-LLAMA(2)](../论文阅读笔记/img/Vista-LLAMA(2).png)



**连续的视觉投影**



<font color=blue>*原本的Qformer projector*</font>

$x_i=f_Q(o_i,p)\in R^{k\times d}$

o是视觉特征，p是可学习的query embedding



<font color=green>*创新引入的时间Qformer*</font>

$x_t=f_Q(o_i,x_{t-1})$

```
这个好！！！
```

![Vista-LLAMA(3)](../论文阅读笔记/img/Vista-LLAMA(3).png)



<h2><font color=green>表现</font></h2>

![Vista-LLAMA(4)](../论文阅读笔记/img/Vista-LLAMA(4).png)

以及补充一个有趣的数据：

![Vista-LLAMA(5)](../论文阅读笔记/img/Vista-LLAMA(5).png)



虽然整体上是视觉token越多，正确率越高，但是对于时序问题，堆叠token数目并不能真正解决问题





<h2><font color=blue>引入及相关内容</font></h2>

(introdutcion内容和related work内容)



用LLM做多模态任务分为两种：

1.类似昨天的AoTD以及什么ViperGPT之类：

将 LLM 当作“控制器”，多模态模型作为“工具”

2.直接训练大规模的多模态模型





幻觉产生的根本原因：



直接copy:

```
当视觉 token 与当前生成的文本 token 在LLM的上下文中距离过远时，视觉信息的影响力会显著衰减。此外，如果视频较长，视觉 token 数量很多，也会在大语言模型有限的上下文窗口中占用大量空间，导致处理长视频更加困难。
```

