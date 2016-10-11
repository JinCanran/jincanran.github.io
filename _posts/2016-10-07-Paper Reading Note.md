---
layout:     post
title:      "Paper Reading Note"
subtitle:   "Approximated Prediction Strategy for Reducing Power Consumption of Convolutional Neural Network Processor"
date:       2016-10-07
author:     "Jin"
header-img: "img/post-bg-2015.jpg"
tags:
    - CNN
    - Approximate Computing
    - Note
---

>  Ujiie T, Hiromoto M, Sato T. Approximated Prediction Strategy for Reducing Power Consumption of Convolutional Neural Network Processor[C]//Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition Workshops. 2016: 52-58.

## ABSTRACT

- CNN在图像识别领域很热门，但是它在嵌入式系统的计算消耗高
- 本Paper提出了一个高效方法 LazyConvPool（LCP）和硬件结构，来降低能耗在CNN-based图像识别
    - LCP利用了CNN的冗余操作，以及只执行必要的卷积通过approximate的预测技术
- 提出了 Sign Connect， 是一个低计算消耗的approximate预测技术
- 在power consumption和energy consumption的方面均有效

----

## 1. Introduction

- A large number of dedicated vision processors have been proposed
    - handcrafted features (Haar-like features, SIFT,HOG, etc.)  
    - learning algorithms (SVM, Boosting, etc.)
    - 以上原文的reference里有
- 尽管是有效的高性能，但是这些设计特殊processor需要大量的开发effort
- In contrast to the traditional recognition algorithms that
utilize the handcrafted features, **representation learning[1]** is becoming increasingly popular in the state-of-the-art recognition methods.
    - 核心思想是让系统除了学习classifier之外，自身学习feature
- CNN是representation learning众多方法中的一个并且取得了显著的成效
    - CNN不断处理2-D卷积过程，最终提取出feature
    - 图像局部特征通过不断调整卷积filter的weight来得到
    - 这个总体的框架让CNN可以识别许多类图片而不需要人工设计提供的feature
    - 为了更准确，CNN网络结构变得deeper and deeper，所以需要大量的算力
    - 如果有一个高效的计算过程来简化大量的卷积过程就会很有用，特别是在能量有限的嵌入式设备中
- Recent research: inherent resilience of CNN
- 模式识别算法普遍包括了冗余的计算去保证噪音下的鲁棒性，换言之，算法的这部分冗余部分可以想办法解决，那么可以减少算力消耗
- 所以，致力于max pooling operation，which is an integral component of CNN that selects
a representative feature in a small region.
- **提出了LazyConvPool (LCP), 在卷积层和pooling层**
    - 只有必要的卷积被执行，通过近似计算预测结果
    - 核心特性：
        1. LCP预测之后会被pooling layer选择的局部区域，to limit the application of the accurate and thus expensive convolutional operations to the predicted region
        2. 预测基于本paper新提出的**Sign Connect**，能够轻量的预测近似的卷积with no multiplication
operation,同时还能保证预测准确率
        3. 还提出了硬件架构，去实现LCP-based CNN with minimal overhead

----
## 2. Related works
### 2.1 Convolutional neural network

- Here use the max pooling
- After the convolution and pooling, non-linearity is used as an activation function，**piecewise continuous functions** are generally used as the activation function
- In fully connected layer， 每层计算都和传统的神经网络一样，the **softmax function** is often used as the activation function for multi-class classification
    - softmax有一个好处： normalizes values in the final stage and facilitates error computation
in the training phase
    - The number of the output units in the final stage is equal to the number of the labels, and the classification result **y** is given by Paper上的公式（5）

### 2.2 Computationally efficient neural networks

- CNN - a huge number of multiplications are executed to propagate signals in the networks
- Recent work：
    -  tackle this problem by replacing the multiplications with simpler operations
    -  Network pruning [14] and network compression [2, 3] reduce the amount of weight parameters by compressing networks
    -  BinaryNet [7] and Ternary Connect [19] are the methods that limit the weight values to {−1,+1} or {−1, 0,+1} during the training phase in order to reduce the memory usage and computation complexity
    -  **Ternary Connect** —— on which **Sign Connect** is based
    -  具体实现看paper
    -  **Ternary Connect**是有效的，可以在训练phase减少计算量，如果简单的应用到识别phase会降低识别性能，此外还需要考虑合适的硬件架构去获得减少乘法的优势
    
### 2.3 CNN processors

- As application-specific processors, some architectures that implement CNN are proposed in [4, 5, 11,13]
- speed up the calculation of convolutions in a highly parallel manner by adopting an array structure and pipelined streaming processing units
    - nn-X [11] —— a salable co-processor for CNN. It consists of an array of possessing elements，dedicated hardware to accelerate the main components of CNN(convolution, pooling, and activation).
    - HWCE [5] —— focusing on 2D convolutions
- **Low-power CNN processors is based on** **approximate computing**
    - ApproxANN [23] and AxNN [21] are examples of using the approximate computing with neural networks
    - 但是近似计算很难应用于CNN，因为CNN的深度结构，近似误差在传播过程中会积累和膨胀，会破坏识别精度  

- 所以在此提出了一个新颖的办法，使用近似计算加强现有的CNN processer 的能量效率，In order
to suppress the degradation of recognition performance,we adopt the approximation only to predict the region to be selected in the max pooling
- 为了减少识别性能的损耗，只用近似计算去预测 max pooling 选择的区域

-----

## 3. Proposed method

**LazyConvPool (LCP) & Sign Connect**

### 3.1 LazyConvPool

- 卷积之后只有一个结果会被传到下一步，其他的都被pooling操作抛弃了，所以说不具代表性的feature 不会 affect feature map，只要pooling的选择是正确的
- 基于此，构建了LCP框架 两个部分：
    1. 近似卷积单元 ： 预测会被pooling选出的特征
    2. 单独的卷积单元 ： 只卷积被预测出来的window
- 基本步骤：
    1. 计算近似卷积
    2. max pooling specify the convolution index of which the convolution result is propagated
    3. 只计算特定index的正常卷积
- 减少卷积很显然，预测需要额外的硬件，为了更轻巧近似预测，设计了Sign Connect

### 3.2 Sign Connect

- Sign Connect作为近似计算单元去降低计算复杂度
- 相关的有BinaryNet和Ternary Connect
- 剧烈的简化乘法，性能增强

-----

## 4. Hardware architecture

### 4.1 Base architecture

- similar to those of nn-X [11] and HWCE [5]
- Each processing element, connected to the shared memory, executes convolution,pooling, and activation
- In this hardware architecture,each element computes individual feature maps on the basis of a common input image (or feature map)
- Line buffer : supply image to each element, enables a continuous and efficient data feed for
convolutions on a sliding windows
- the memory and accumulators are connected to accumulate the value of feature maps. Convolution results are accumulated on the memory that has the size of the feature map

### 4.2 LCP architecture

- processing element 变成CLP的版本，和前面介绍的一样
- 为了减少时间代价把AppConv和Conv做一个并行的版本

---------

## 5. Experiments

### 5.1 Classification accuracy

- Datasets
- Results         

### 5.2 

- Experimental setup
- Results

------

## 6. Conclusion

很强






-----
## References
[1] Y.Bengio, A.Courville, and P.Vincent.Representation
learning: A review and new perspectives.IEEE Transactions
on Pattern Analysis and Machine Intelligence, 35(8):1798–
1828, 2013.1  
[2] W.Chen, J.T.Wilson, S.Tyree, K.Q.Weinberger, and
Y.Chen.Compressing convolutional neural networks.Computing
Research Repository, abs/1506.04449, 2015.2  
[3] W.Chen, J.T.Wilson, S.Tyree, K.Q.Weinberger, and
Y.Chen.Compressing neural networks with the hashing
trick.In Proceedings of International Conference on Machine
Learning, pages 2285–2294, 2015.2, 7  
[4] Y.-H.Chen, T.Krishna, J.Emer, and V.Sze.Eyeriss: An
energy-efficient reconfigurable accelerator for deep convolutional
neural networks.In Proc.IEEE International Solid-
State Circuits Conference, pages 262–263, 2016.3  
[5] F.Conti and L.Benini.A ultra-low-energy convolution engine
for fast brain-inspired vision in multicore clusters.In
Proceedings of the Design, Automation & Test in Europe
Conference & Exhibition, pages 683–688, 2015.3, 4  
[6] C.Cortes and V.Vapnik.Support-vector networks.Machine
learning, 20(3):273–297, 1995.1  
[7] M.Courbariaux and Y.Bengio.Binarynet: Training deep
neural networks with weights and activations constrained to
+1 or −1.Computing Research Repository, abs/1602.02830,2016.2, 4  
[8] N.Dalai, B.Triggs, I.Rhone-Alps, and F.Montbonnot.Histograms
of oriented gradients for human detection.In Proceedings
of the IEEE Computer Society Conference on Computer
Vision and Pattern Recognition, pages 886–893, 2005.
1  
[9] Y.Freund and R.E.Schapire.A decision-theoretic generalization
of on-line learning and an application to boosting.
Journal of computer and system sciences, 55(1):119–139,
1997.1  
[10] R.Girshick.Fast R-CNN.In Proceedings of the IEEE International
Conference on Computer Vision, 2015.1  
[11] V.Gokhale, J.Jin, A.Dundar, B.Martini, and E.Culurciello.
A 240 G-ops/s mobile coprocessor for deep neural networks.
In Proceedings of the IEEE Computer Society Conference on
Computer Vision and Pattern Recognition Workshop, pages
696–701, 2014.3, 4  
[12] I.Goodfellow, D.Warde-farley, M.Mirza, A.Courville, and
Y.Bengio.Maxout networks.In Proceedings of the International
Conference on Machine Learning, pages 1319–1327,
2013.7  
[13] S.Han, X.Liu, H.Mao, J.Pu, A.Pedram, M.A.Horowitz,
and W.J.Dally.EIE: efficient inference engine on compressed
deep neural network.Computing Research Repository,
abs/1602.01528, 2016.3  
[14] S.Han, H.Mao, and W.J.Dally.Deep compression: Compressing
deep neural network with pruning, trained quantization
and huffman coding.Computing Research Repository,
abs/1510.00149, 2015.2, 7  
[15] K.He, X.Zhang, S.Ren, and J.Sun.Deep residual learning
for image recognition.Computing Research Repository,
abs/1512.03385, 2015.1  
[16] A.Krizhevsky.Learning multiple layers of features from
tiny images.Master’s thesis, University of Toronto, 2009.5  
[17] A.Krizhevsky, I.Sutskever, and G.E.Hinton.ImageNet
classification with deep convolutional neural networks.In
Proceedings of the Neural Information Processing Systems,
pages 1097–1105.2012.1  
[18] Y.Lecun, L.Bottou, Y.Bengio, and P.Haffner.Gradientbased
learning applied to document recognition.Proceedings
of the IEEE, 86(11):2278–2324, 1998.5  
[19] Z.Lin, M.Courbariaux, R.Memisevic, and Y.Bengio.Neural
networks with few multiplications.Computing Research
Repository, abs/1510.03009, 2015.2, 4  
[20] D.Lowe.Object recognition from local scale-invariant features.
In Proceedings of the IEEE International Conference
on Computer Vision, volume 2, pages 1150–1157, 1999.1  
[21] S.Venkataramani, A.Ranjan, K.Roy, and A.Raghunathan.
AxNN: Energy-efficient neuromorphic systems using approximate
computing.In Proceedings of the International
Symposium on Low Power Electronics and Design, pages
27–32, 2014.1, 3  
[22] P.Viola and M.Jones.Robust real-time object detection.
International Journal of Computer Vision, 57(2):137–154,
2004.1  
[23] Q.Zhang, T.Wang, Y.Tian, F.Yuan, and Q.Xu.Approx-
ANN: An approximate computing framework for artificial
neural network.In Proceedings of the Design, Automation
& Test in Europe Conference & Exhibition, pages 701–706,
2015.1, 3  
