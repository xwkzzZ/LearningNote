<h1>HierarQ笔记</h1>



[原文](https://sacrcv.github.io/HierarQ-website/)





<h2><font color=red>方法</font></h2>

![HierarQ(1)](../论文阅读笔记/img/HierarQ(1).png)

![HierarQ(1)](../论文阅读笔记/img/HierarQ(2).png)



<h3><font color=green>和MA-LMM的区别</font></h3>

相当于为了捕捉两部分，一个所谓的细致细节内容，一个长期时序特征

**区别1**：

将原来从memory-bank到Qformer处，直接复制粘贴了一次

粘贴的在做完cross-attention后再整一次对视觉的self-attention，并且与原本的最后隐藏层再1，2，3，4再来一次cross-attention



**区别2**：

对于原版的visual memory bank(short-term memory bank)，记忆压缩策略改变，变为队列，丢掉最早进来的

而复制的(long-term memory bank)，则还是照旧





<h2><font color=purple>我勒个豆</font></h2>

![HierarQ(3)](../论文阅读笔记/img/HierarQ(3).png)

**总结下**

COIN涨了2.8

ActivityNet-QA涨了7.3

LVU平均涨了6.8，最高一项涨了11.2，最低一项涨了4.5





训练细节比对了下，相比于MA-LMM的代码，他文献里面的训练细节，多了LORA以及将学习率从1e-4调低到1e-5





*这个有用！*

![HierarQ(4)](../论文阅读笔记/img/HierarQ(4).png)





哎～确实没啥笔记可记，真的如果吃透了MA-LMM，真就～昂～

