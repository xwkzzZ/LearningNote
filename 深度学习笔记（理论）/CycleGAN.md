<font size=8>CycleGAN</font>



<font size=5>Learning from Unpaired Data</font>



当我们无从预先得知y的标准答案时，数据不匹配时，我们通过创建两个转化的架构，从输入到目标生成，并且又将目标生成转化为类原始输出，通过比较原始的输出与两次转化后的类原始输出，我们可以以两个图片的相近程度来训练这种转化，去约束目标输出与原始输出的相关性

见图：

<img src="../深度学习笔记（理论）/imgCollect/CycleGAN(1).png" alt="CycleGAN(1)" style="zoom:40%;" />

Dual GAN,Disco GAN原理上与Cycle GAN一样

<img src="../深度学习笔记（理论）/imgCollect/CycleGAN(2).png" alt="CycleGAN(2)" style="zoom:40%;" />





