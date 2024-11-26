<font size=8>Deep Learning(2)</font>



<font size=5>Tips for Training: Batch and Momentum</font>



<font size=4>**1.为什么要用batch?**</font >



值得注意的是：

a.由于平行运算的能力，更大的batch不一定比小的batch需要更长的时间去完成update

（除非batch-size过于庞大）（60updates与6000updates所需时间相近)

而且小的batch-size实际上跑完一次epoch所需时间实际上比更大的batch-size更多



b.小的batch-size伴随着noisy其实在训练效果上比大的batch-size要好。(optimization fails)

 Reason:给出的可能的原因是在batch-size较大时，如果遇到local minima时都会出现stuck的状况，但是small batch可能就会因为产生不同的loss function而部分规避掉local minima



故batch的大小选择很大程度上会影响我们的训练效果





<font size=5>Loss Function:Classification</font>(short)



我们用one-hot vector来表示class
$$
\begin{gathered}
\begin{bmatrix} 1\\0\\0 \end{bmatrix}
\quad
\begin{bmatrix} 0\\1\\0  \end{bmatrix}
\quad
\begin{bmatrix} 0\\0\\1  \end{bmatrix}
\quad
......
\end{gathered}
$$






<font color=red>Softmax</font>:
$$
y_{i}^{'}=\frac{exp(y_i)}{\sum_{i}exp(y_i)}
$$
y一般称为<font color=red>logit</font>

当有两个class时用sigmoid()







<font color=red>Cross-entropy（最小交叉熵损失）</font>:
$$
e=-\sum_{i}{y_{i}lny_{i}^{'}}
$$


**Minimizing cross-entropy is equivalent to maximizing likelihood**

Cross-entropy比MSE更适合分类，而且在pytorch中，使用cross-entropy会自动调用softmax(),加到原有的network中















