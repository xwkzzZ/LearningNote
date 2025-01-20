<font size=8>VQA综述笔记</font>



[原文](https://arxiv.org/pdf/2501.07109)



**extractive paradigm**:从给定的上下文中直接抽取一个片段作为答案。

**abstractive paradigm**:通过理解上下文生成新的答案，答案可能是原文中没有直接出现的。



<font size=5>**Early Deep Learning Models and Representation Fusion**</font>



**3.1 CNN-LSTM Models: The First Generation of VQA Models**

<font color=blue>Extractive Paradigm: </font>CNN-LSTM，第一代VQA模型，但是对于模态的融合较为简单，没有捕捉到视觉上的复杂信息之间的关系



<font color=blue>Abstractive Paradigm:</font>将CNN提取的信息给RNN再生成文本，对于fine-grained多模态推理不到位，给的回答还是较为简陋



**3.2 Bilinear Pooling: Improving Multimodal Fusion for VQA**

无论是extractive model还是abstractive model，双线性池化的引入都为之带来了更加细致地理解，效果的提升



**3.3 Attention Mechanisms: Learning to Focus**

提高上下文推理的能力，以及对于extractive model理解空间相对关系的能力



**3.4 Bottom-Up and Top-Down Attention: A Major Breakthrough**

<font color=green>Extractive Paradigm:</font>抽取object-level特征(bottom-up attention)并应用到问题驱动聚焦机制上(top-down attention)



<font color=green>Abstractive Paradigm:</font>能够进行更丰富的关系推理，带来答案的动态，多样性



**3.5 The Shift towards Advanced Models for VQA**

Extractive Paradigm:预训练模型，端到端猛train

Abstractive Paradigm:直接将多模态的embedding融到decoding过程中





<font size=5>**Compositional Reasoning and Modular Networks For VQA**</font>

对于复杂的VQA任务，传统方法力不从心



**4.1** **Neural Module Networks (NMNs): A Modular**

**Approach to Reasoning**

<font color=blue>Extractive Paradigm: </font>引入根据查询结构动态组装的网络。由小型、可重用的模块构成，每个模块负责特定的任务。

但是对任务特定模块的依赖使其计算成本高昂，且难以泛化到多样化的数据集。（过于依赖对于query的解析以及物体检测）



<font color=blue>Abstractive Paradigm:</font>引入了能够根据查询语义，动态调整其结构的模型。



**4.2 Graph-Based Visual Question Answering**

<font color=green>Extractive Paradigm:</font>利用图的方法，将物体作为点，关系作为边，对于每一个查询，从点开始去遍历它的边来获取答案

<font color=green>Abstractive Paradigm: </font>扩展了基于图的方法,通过推理视觉元素并结合先验知识，生成了上下文感知的自由形式答案。



**4.3 Compositional Attention Networks (MAC) for**

**VQA**

<font color=red>Extractive Paradigm: </font>递归注意力机制,逐步推理,提供可解释的推理路径

关注房子，定位房子前面的汽车，识别汽车的颜色,以此回答“What is the color of the car in front of the house?”

<font color=red>Abstractive Paradigm: </font>almost same features



**4.4 The Rise of Pre-trained Transformers and the**

**decline of Modular Networks**

<font color=purple>Extractive Paradigm:</font>预训练的Transformer模型隐式捕捉了关系和组合结构，消除了对显式模块化或基于图的架构的需求。能够端到端处理多模态输入。

<font color=purple>Abstractive Paradigm:</font>集成了外部知识和上下文，生成丰富、自由形式的答案。标志着模块化网络的衰落。



<font size=5>**Transformer Models and Vision Language Pre-training**</font>

**5.1 Cross-Modal Alignment and Feature Integration in VQA**

<font color=blue>Extractive approaches</font>:强调视觉和文本的精确对齐，依赖于输入模态的共享表示,严重依赖于预提取的基于区域的视觉特征，并且在处理更动态和开放式的上下文时往往难以泛化。

<font color=blue>Abstractive approaches</font>:超越了简单的对齐，整合特征以生成全面且上下文丰富的答案.通过抽象视觉和文本输入来合成新信息，提供比直接检索更细致的回答。



**5.2 Unified Understanding and Generalization in VQA**

<font color=green>Extractive approaches</font>:通过对齐视觉输入和文本查询来检索精确答案。将模态映射到共享表示空间，以促进精确推理。

<font color=green>Abstractive approaches</font>:Flamingo,Blip-2都是
