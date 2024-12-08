<font size=8>Diffusion Model原理</font>



引言：

（VAE是一种auto-encoder)

Diffusion颇似VAE，不过不同于encoder与decoder直接让它去学，Diffusion中很多操作已经固定好了，加noise，去noise....





**Train**

<img src="../深度学习笔记（理论）/imgCollect/DM原理(1).png" alt="DM原理(1)" style="zoom:40%;" />



$$\alpha_{t}$$​ 决定保留原图的比例

（ $$\alpha$$ 是hyperparameter）



简洁说就是：train出一个可以预测加了多少noise的机器，输入的是加入了noise的图和step，输出是加入的noise



至于如何生成的noise与step，就是在一个上升数组中取一个，作为保留原始图片程度的参数，然后加入。这是train需要的两个参数



**Inference**

<img src="../深度学习笔记（理论）/imgCollect/DM原理(2).png" alt="DM原理(2)" style="zoom:40%;" />

用之前train出来的noise predictor，将input的图片减去经过缩放后的噪音，然后再加入噪音



<font color=blue>**如何衡量两个分布越接近越好?**</font>



通过数学推导可以证明：

$$\theta^{*}=arg(max \prod_{i=1}^{m}P_{\theta}(x^{i}))=arg min(KL(P_{data}||P_{\theta}))$$

**Maximum Likelihood = Minimize KL Divergence**



我们很难算出 $$P_{\theta}(x^{i})$$



<font color=red>VAE:算P(x)</font>

<img src="../深度学习笔记（理论）/imgCollect/DM原理(3).png" alt="DM原理(3)" style="zoom:40%;" />

视高斯分布的均值为输出的“标答”



在VAE中常常是对logP(x)操作而非直接对P(x)



同时在maximum P(x)的过层中，往往maximum的logP(x)的下界：
<img src="../深度学习笔记（理论）/imgCollect/DM原理(4).png" alt="DM原理(4)" style="zoom:40%;" />





<font color=red>DDPM:计算P(x)</font>

<img src="../深度学习笔记（理论）/imgCollect/DM原理(5).png" alt="DM原理(5)" style="zoom:40%;" />

其实和VAE的处理大差不差

<img src="../深度学习笔记（理论）/imgCollect/DM原理(6).png" alt="DM原理(6)" style="zoom:40%;" />



DDPM的logP(x)的lower bound的计算推导过于变态，略



<font color=blue>**Optimize**</font>

<img src="../深度学习笔记（理论）/imgCollect/DM原理(7).png" alt="DM原理(7)" style="zoom:40%;" />

同时可以通过数学大法将 $$x_{0}$$ 化成 $$x_{t}$$ ，最后的只需要去收敛于一个只带有 $$x_{t}$$ 与 noise的（还有一堆常数）的项

（就是需要预测那个noise)