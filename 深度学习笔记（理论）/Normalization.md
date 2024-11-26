<font size=8>Normalization</font>



**training tips**

<font color=red>Changing Landscape</font>

由于learning rating是个定值，在遇到不同的dimensions时可能对于loss的变化相差较大（有的方向陡峭些，有的平缓些，但是我们移动的值是定的，导致traing时带来困难）

此时，我们需要让不同的dimension拥有同样的数值范围





<font color=red>Feature Normalization:</font>
$$
\hat{x^{r}}=\frac{x_{i}^{r}-m_{i}}{\sigma}
$$




normalization后，gradient descent的loss会更收敛一点训练会更快一点

而将dataset分为多组batch，对每组batch进行feature normalization也称之为：batch normalization





<font color=red>Batch normalization -Testing</font>

在testing中，可能数据量是慢慢增加的，我们不能等到batch到达一定数量后再进行更新，所以我们必须动态的更新mean与standard

<img src="../深度学习笔记（理论）/imgCollect/Nor(1).png" alt="Nor(1)" style="zoom:40%;" />



