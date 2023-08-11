---
layout: post
title: "Transformer"
subtitle: "semantic segmentation"
author: "DYC"
header-img: "img/ss.jpg"
header-mask: 0.3
catalog: true
tags:
  - semantic segmentation

---

### Attention is all you need

##### motivation

- 对于RNN而言，递归式进行，但因为它的特点是需要将历史信息传递给当前步骤，以帮助理解和预测后续数据，这种依赖关系意味着在顺序处理数据时，每个时间步骤必须按顺序计算，无法并行化，因此速度较慢
- CNN只能获取局部的感受野，如果想得到全局的需要多尺度融合等方式

##### contribution

- 引入自注意力机制：Transformer模型中使用了自注意力（self-attention）机制，用于建模输入序列中不同位置的依赖关系
- 消除了传统的序列顺序处理：Transformer模型中的自注意力机制使得模型能够同时处理整个输入序列，而不需要按照顺序逐步进行处理
- 使用了位置编码：为了保留输入序列中单词的位置信息，Transformer引入了位置编码，通过将位置信息嵌入到输入向量中来为不同位置的单词提供额外的信息。这样，模型能够感知输入序列中不同位置之间的关系，从而更好地捕捉上下文的语义
- 堆叠多层自注意力和前馈神经网络：Transformer模型由多个相同结构的编码器和解码器层堆叠而成，每个层都包括自注意力机制和前馈神经网络

##### architecture

Transformer model architecture

![image-20230810173457912](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230810173457912.png)

**encoder-decoder结构**

从Transformer结构图也可以看出transformer也是个encoder-decoder结构

![image-20230811105106163](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230811105106163.png)

**encoder**

编码器由6个相同的层组成，每层有两个sub-layer

1. 第一个sub-layer是multi-head self-attention mechanism，用于计算self-attention值
2. 第二个sub-layer是全连接层

并且对于每个sub-layer都使用了残差网络，为了保证每一层能顺利连接，将输出维度Dmodel=512

**decoder**

解码器也是由6个相同的层组成，每层有三个sub-layer

1. 第一个sub-layer是multi-head self-attention mechanism，用于计算self-attention值
2. 第二个sub-layer是全连接层
3. 第三个sub-layer是修改之后的multi-head self-attention mechanism

前两个sub-layer与encoder的sub-layer一样的，对于第三个sub-layer是进行了修改，修改之处在于新增了一个**mask机制**，作用是**将未来位置的权重设置为0用以屏蔽未来的位置**，从而防止当前位置与未来位置之间建立依赖关系（通过防止当前位置对未来位置的注意力，Transformer模型能够更好地处理序列信息。这项技术允许模型在生成输出时只关注已经观察到的输入，避免未来信息对当前预测产生干扰）

唯一需要屏蔽的**多头注意力块是每个解码器块的第一个。**这是因为中间的一层用于组合编码输入和从前一层继承的输出之间的信息。将每个目标标记的表示与任何输入标记的表示组合起来没有问题

**Attention机制**

注意力机制（Attention Mechanism）是一种在神经网络中用于加强模型对输入中不同部分的关注程度的技术。它可以帮助模型更好地理解和处理输入序列中的关系，从而提高模型的表现

其思想是Target中的某个元素Query，通过计算Query和各个Key的相似性或者相关性，得到每个Key对应Value的权重系数，然后对Value进行加权求和，**即得到了最终的Attention数值**

![image-20230811111611592](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230811111611592.png)

**Self-attention**

有时称为intra-attention，是一种将单个序列的不同位置关联起来计算序列表示的注意机制，减少了对外部信息的依赖，更擅长捕获数据或特征的内部相关性

**attention与self-attention的区别**：

对于输入的一句话，attention关注其中哪个单词更重要，self-attention通过建立不同单词之间的关系，来寻找哪个单词与哪个单词的关系更加紧密

![image-20230810220902889](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230810220902889.png)

输入序列X1-X4，通过乘上一个W矩阵得到embedding a1-a4，进入self-attention中，每一个输入乘上三个不同的矩阵，得到向量Q，K，V

![image-20230811153806646](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230811153806646.png)

接着用query与每一个key作attention，得到了每一个词与其他词之间的关系

![image-20230811154329979](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230811154329979.png)

上一步得到的每一个词与其他词之间的关系矩阵与V相乘，得到最后的attention

![image-20230811154757937](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230811154757937.png)

self-attention可以由下面公式表示
$$
Self-Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}})V
$$
其中Q与K的点乘代表了两者的相似度，除以根号d是为了使得内积不至于太大（否则softmax 后非0即1），再进行softmax进行归一化得出来的代表了位置之前的关系权重，再与V相乘就得到了attention向量

**Multi-Head Attention**

但如果只对Q、K、V做一次这样的权重操作是不够的，这里提出了Multi-Head Attention，即通过学习不同的线性投影矩阵，将查询、键和值分别进行多次线性投影，得到多组具有不同维度的查询、键和值，再使用上面的公式得到不同attention值进行拼接

使用多头注意力的好处在于，**不同的head关注的信息可能是不同的，有的head关注的是局部信息，有的关注的是较长距离的信息**，模型能够同时从不同的表示子空间中获取信息，并在不同的位置上共同注意。相比于单一注意力头，多头注意力能够更充分地利用输入序列中的信息，并获得更全面的上下文理解。通过拆分和重组注意力，多头注意力机制在Transformer等模型中取得了很好的效果，提高了模型性能和泛化能力

![image-20230810230141899](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230810230141899.png)

具体做法可以用下面公式表现出来，通过W参数矩阵对Q，K，V进行映射，参数矩阵的维度分别是dmodel×dk，映射之后使用attention公式计算出head值，一共进行h次，最后再进行拼接并通过WO进行投影，得到最终的输出值，WO维度为hdv×dmodel，dmodel表示模型的维度等于dk*h

![image-20230810225702829](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230810225702829.png)

若head为2，可以用下图表示

![image-20230811160847324](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230811160847324.png)

在本文的编码器-解码器注意力层中，查询（queries）来自**前一个解码器层**，而内存键（memory keys）和值（values）来自**编码器的输出**。这样，**解码器中的每个位置都可以关注输入序列中的所有位置**

与此同时也会使用到一些**自我注意力机制**，编码器部分包含自注意力层（self-attention layers），即每个位置的键、值和查询都来自于同一位置，也就是前一层的编码器输出。**编码器中的每个位置可以与前一层编码器中的所有位置进行关联**，类似地，解码器中的自注意力层允许解码器中的每个位置关注到解码器中包括当前位置在内的所有位置，通过使用**mask**将违规连接给屏蔽掉

 **Position-wise Feed-Forward Networks**

编码器和解码器中的每一层还包含一个全连接的前馈神经网络，它对每个位置分别进行相同的处理，该前馈神经网络包括两个线性变换，并在它们之间应用ReLU激活函数

![image-20230810231959630](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230810231959630.png)

虽然线性变换在不同的位置上是相同的，但它们在每一层都使用不同的参数。其实可以将其描述为两个卷积操作，卷积核大小为1

这个前馈神经网络在每个位置上独立地对信息进行非线性变换，增加了模型的表达能力和灵活性。通过使用不同的参数，每层的前馈网络可以学习到不同的特征表示，从而提取更丰富和有用的信息

**Positional Encoding**

上面的模型是不能捕捉序列的顺序，即如果将 K,V 按行打乱顺序（相当于句子中的词序打乱），但Attention 的结果还是一样的

所以为了使模型能够利用序列的顺序信息，需要向编码器和解码器堆栈的底部添加一些关于标记相对或绝对位置的信息，为此引出了positional encodings，位置编码与嵌入具有相同的维度d_model，因此可以将**两者相加**，本文中使用的是Sinusoidal Position Encoding

![image-20230810232535998](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230810232535998.png)

对于位置索引为pos和维度索引为*i*的位置向量，我们分别计算正弦和余弦值，然后将它们分别作为位置向量的两个维度，然后将其组合起来形成一个位置向量，这样的组合方式保留了正弦和余弦的周期性特征，能够利用序列的顺序信息，并更好地理解输入数据

![image-20230811141919935](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230811141919935.png)

**Why Self-Attention**

最后作者将自注意力层与常用于将一个变长序列的符号表示（x1，...，xn）映射到另一个相同长度序列（z1，...，zn）的循环和卷积层进行了比较

![image-20230810233636647](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230810233636647.png)

上表中**Complexity per Laye**r代表了每一层的计算复杂度，**Sequential Operations**代表了能够被并行的计算，用需要的最少的顺序操作的数量来衡量，**Maximum Path Length**代表了影响学习长距离依赖的关键点在于前向/后向信息需要传播的步长，输入和输出序列中路径越短，那么就越容易学习long-range dependencies

[参考](https://zhuanlan.zhihu.com/p/63191028)

最后附上transformer过程图

![img](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/315c679b619c4ef388d91ab026ab12ea-20230811163840589.gif)



