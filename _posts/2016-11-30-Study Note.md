---
layout:     post
title:      "深度神经网络中常见的 Tricks 分析"
subtitle:   "Paper Reading"
date:       2016-09-30
author:     "Jin"
header-img: "img/post-bg-2015.jpg"
tags:
    - Paper Reading Notes
    - Paradigms
    
---


>  [原文：Must Know Tips/Tricks in Deep Neural Networks (by Xiu-Shen Wei)][1]

-------------------------------

## 深度神经网络中常见的 Tricks 分析  

- 逛知乎偶然发现的一遍好文章，读完之后觉得对于初学者来说非常值得学习
- 在此做一个简单的内容摘要，方便以后回来看

--------------------------

### 介绍

Tricks in DNN 主要在以下八个方面：

1.  data augmentation
2.  pre-processing on images
3.  initializations of Networks
4.  some tips during training 
5.  selections of activation functions
6.  diverse regularizations
7.  some insights found from figures and finally 
8.  methods of ensemble multiple deep networks

-----------------------------------

### 1. Data Augmentation

为了得到好的performance，需要在大量的数据集上进行训练，如果初始的训练数据非常有限那么就需要data augmentation.
> Also, data augmentation becomes the thing must to do when training a deep network.  

- There are many ways to do data augmentation, such as the popular **horizontally flipping, random crops and color jittering**. 此类方法可以结合使用
- fancy PCA，调整RGB信道的强度（关于PCA主成分分析：[UFLDL-主成份分析][2]）

--------------------

### 2. Pre-Processing

1. **zero-center** the data, and then **normalize** them
    `X -= np.mean(X, axis = 0) # zero-center`
    `X /= np.std(X, axis = 0) # normalize`

2. PCA Whitening 白化（ufldl：[此处输入链接的描述][3]）  

需要注意，讨论这些预处理只是为了本文的完整性，实践中这些变换不用在CNN中*（为啥？我的代码先白化过..）*
>Please note that, we describe these pre-processing here just for completeness. In practice, these transformations are not used with Convolutional Neural Networks. However, it is also very important to zero-center the data, and it is common to see normalization of every pixel as well.  

---------------

### 3. Initializations

设置初始的网络参数

1. 全零  
    - 不靠谱，虽然统计规律落在零两边的一半一半，但是全部都是0的话，各个神经元都一样没有差异性显然不行
2. 接近0的小随机数
    - 比较合理，和ufldl中说的一样，但是随着输入数目的增长，Variance grows
3. Calibrating the Variances
    - 校准方差：通过神经元的输入数目等调整weight的vector，使得normalize the variance of each neuron's output to 1，但是没有考虑到ReLU激活的神经元（这个目前反而最常用）
4. Current Recommendation
    - 当下比较推荐的做法：上面的没有考虑ReLU，于是 A more recent paper on this topic by He et al. derives an initialization specifically for ReLUs, reaching the conclusion that the variance of neurons in the network should be 2.0/n as:
    `w = np.random.randn(n) * sqrt(2.0/n) # current recommendation`
    
--------------------

### 4. 训练过程中

- Filters and pooling size  
    - 3 X 3 filters with stride 1, could preserve the spatial size of images/feature maps
    - For the pooling layers, the common used pooling size is of 2 X 2  
- Learning rate  
    -  在Ilya Sutskever 的博客中， 建议 divide the gradients by mini batch size，因此，如果经常更改mini batch size，不应该总是修改LR
    - For obtaining an appropriate LR, utilizing the validation set is an effective way. 刚开始学习率设为0.5，然后看验证集的error rate 如果一直不提高的话，就把LR减半或者减为五分之一
- Fine-tune on pre-trained models
    - 许多先进的模型比如VGG之类的已经released， 所以可以应用这些模型在自己的application上， For further improving the classification performance on your data set, a very simple yet effective approach is to fine-tune the pre-trained models on your own data.
    - 重要的因素有两个，一个是新数据集的大小，另一个是和原本网络应用的数据集的相似度
    ![此处输入图片的描述][4]

-----------------------------------

### 5. 激活函数

激活函数是网络的一个关键因素，which brings the non-linearity into networks.    

1. Sigmoid  
    - 压缩到[0，1]之间
    - 过去很常用，但现在用的越来越少了，主要有以下两个缺点：
        - **是饱和函数并且会kill gradients**，当函数饱和在0或者1的时候，梯度就会接近0，在BP算法中，Therefore, if the local gradient is very small, it will effectively “kill” the gradient and almost no signal will flow through the neuron to its weights and recursively to its data
        - **输出不是zero-centered**，这样下一层收到的前层激活值就不是zero-centerd（*可以和之前的数据预处理以及BN之类的处理联系起来*），如果传递来的激活值都是正的，那么BP梯度不是全正就是全负，这样会造成锯齿形的动态梯度更新（显然一点也不好）。However, notice that once these gradients are added up across a batch of data the final update for the weights can have variable signs, somewhat mitigating this issue. Therefore, this is an inconvenience but it has less severe consequences compared to the saturated activation problem above.
2. tanh(x)
    - 和sigmoid类似，是饱和的在[-1,1]之间，但是是zero-centered 所以总是表现的比sigmoid函数要好一点  
3. ReLU  
    - 是最近最popular的，max(0,x)
    - 优缺点如下：
        1. 优点：实现简单并且不饱和
        2. 优点：比起以上几个函数，极大加快了随机梯度收敛的速度，主要得益于局部线性，不饱和的特点
        3. 缺陷：ReLU units can be fragile during training and can “die”，过大的梯度传过ReLU神经元，会让weight的更新in such a way that the neuron will never activate on any datapoint again*（不懂为啥，有时间再去查）*
 4. Leaky ReLU  
    - Leaky ReLUs are one attempt to fix the “dying ReLU” problem，有些人说有效，但是结果不是很确定
 5. Parametric ReLU
    - 复杂，还没来得及看  

In conclusion, three types of ReLU variants all consistently outperform the original ReLU in these three data sets. And PReLU and RReLU seem better choices.

-----------------------------

### 6. Regularizations

主要为了在全局上防止过拟合  

1. L2 regularization
    - 最common的正则化，增加lambda参数，应该就是增加权重惩罚向量（权重衰减），防止过拟合，UFLDL里面说的就是这个
2. L1 regularization
    - 也很common，具体的看文章
3. Max norm constraints
    - 对神经元的权重大小施加绝对上限的约束，*（不是很懂，再说吧）*
4. Dropout
    - 屏蔽一部分上层神经元的输出，读过这篇论文，就不写了

---------------------

### 7.  Insights from Figures

During training time, you can draw some figures to indicate your networks’ training effectiveness.  
这部分还是多看论文结合分析很快就能理解  

1. LR对学习的影响
2. loss 和 epoch的曲线
3. 准确率的曲线
 ![此处输入图片的描述][5]


--------------------

### 8. Ensemble

组合起来，往往能有所成效  
Here we introduce several skills for ensemble in the deep learning scenario.   

还没看完。。待续


 


  [1]: http://lamda.nju.edu.cn/weixs/project/CNNTricks/CNNTricks.html
  [2]: http://ufldl.stanford.edu/wiki/index.php/%E4%B8%BB%E6%88%90%E5%88%86%E5%88%86%E6%9E%90
  [3]: http://ufldl.stanford.edu/wiki/index.php/%E7%99%BD%E5%8C%96
  [4]: http://lamda.nju.edu.cn/weixs/project/CNNTricks/imgs/table.png
  [5]: http://lamda.nju.edu.cn/weixs/project/CNNTricks/imgs/trainfigs.png