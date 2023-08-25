---
layout: post
title: "Rich CNN Features for Water-Body Segmentation from VeryHigh Resolution Aerial and Satellite Imagery"
subtitle: "semantic segmentation"
author: "DYC"
header-img: "img/ss.jpg"
header-mask: 0.3
catalog: true
tags:
  - semantic segmentation

---

#### Rich CNN Features for Water-Body Segmentation from VeryHigh Resolution Aerial and Satellite Imagery

##### motivation

- 遥感图像中水体轮廓不清晰

##### contribution

- 设计了一种新的多特征提取和组合模块，以考虑来自小感受野和大感受野以及通道之间的特征信息。作为编码器的基本单元，该模块完全提取每个尺度的特征信息
- 提出了一个简单有效的多尺度预测优化模块，通过聚合不同尺度的预测结果来实现精细水体分割
- 设计了编码器-解码器语义特征融合模块，以提高编码器和解码器之间特征表示的全局一致性

##### architecture

![image-20230825112440105](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230825112440105.png)

**MEC**

MEC模块由三个子模块组成，即局部特征提取子模块（LFE）、较长感受野特征提取子单元（LRFE）和通道间特征增强（CFE）的提取子单元

LFE和LRFE子模块基于特征图的空间关系（即来自不同的感知场场景），CFE子模块旨在通过建模特征图通道之间的关系来获得额外丰富的特征信息

![image-20230825113034300](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230825113034300.png)

**LFE子模块**如图b所示，旨在根据局部信息学习特征图。具体来说，使用批归一化（BN）和sigmoid函数进行3×3卷积，以学习局部特征的权重图，并将权重图乘以输入。然后，将该结果添加到输入中，作为当前层的最终输出。

**LRFE子模块**使用更大的感受野结构。有两种实现方式：一种是使用密集连接的atrous卷积（DCAC），另一种是通过池化操作或跨步卷积。

**CEF子模块**首先使用全局池运算来获得特征图的全局信息，并采用全连接来学习值之间的关系，以获得通道之间的权重。

**DSFF**

为了解决解码阶段语义不一致影响融合的问题，设计了DSFF模块

![image-20230825142409081](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230825142409081.png)

首先使用1×1卷积带着relu和bn，以将从编码阶段到解码阶段的相同尺度的凹入特征图的通道数减少到一半。然后，通过全局池化从连接的特征生成全局上下文，然后用BN和ReLU进行1×1卷积，用Sigmoid函数进行1×1卷积。全局上下文信息与连接的特征相乘并添加到连接的特征中。最后，将具有BN和ReLU的3×3卷积应用于所获得的特征图

**MPF**

MPF模块在解码阶段对三个尺度的预测结果进行了优化

与BN和ReLU进行1×1卷积以增加通道数量，并分别与BN和S形函数进行3×3、5×5和7×7卷积以学习多尺度权重信息。权重包含来自级联结果的不同感受野的重要信号，这些信号分别乘以权重。最终，我们将这些结果连接起来，并使用3×3卷积核与BN和1×1卷积来获得最终预测结果

![image-20230825141139169](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20230825141139169.png)