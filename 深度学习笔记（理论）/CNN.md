<font size=8>CNN</font>



<font size=5>Image Classification</font>



**Convolution Layer**

<font color=red>Simplification 1:</font>

对于image classification，我们没有必要作fully connected network，我们可以通过critical part来判断

因此可以将输入切割成一些receptive field，然后将这些去对应不同或者同一个neuron



<font color=red>Simplification 2:</font>

对于某一个critical part可能出现在不同的receptive field，我们通常会对于进行neuron的parameters sharing,但是如果是同一个receptive field,则不共享



Typical setting:

每个receptive field都有一组对应的neurons（常见的为64个）

而每组receptive neurons对应64组neurons

共用同一组参数的neuron用称为filter





每组receptive通过一组filter得到的叫做feature

filter与一个图片的所有channel所产生的多层feature合集叫做feature map

<font color=red>综述：</font>

<img src="../深度学习笔记（理论）/imgCollect/CNN(1).png" alt="CNN(1)" style="zoom:40%;" />





**Pooling**



在convolution后，我们往往进行pooling操作，让图片变小

在实际中，我们也经常将convolution与pooling交替使用

但是pooling会减少样本数目量，subsampling（二次取样）,当我们不考虑算力的时候，也有使用fully convolution的架构





为了得到最后的结果，进行一步flatten



不过CNN无法一样的应付缩放与旋转（原话：CNN is not invariant to scaling and rotation(we need data agumentation)）（Spatial Tranformer Layer)

