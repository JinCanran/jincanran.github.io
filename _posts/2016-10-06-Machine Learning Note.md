---
layout:     post
title:      "学习笔记 —— 初识机器学习"
subtitle:   "神经网络基础"
date:       2016-10-06
author:     "Jin"
header-img: "img/post-bg-2015.jpg"
tags:
    - Machine Learning
    - Notes
---

>  UFLDL教程学习笔记
>> 公式直接使用原教程图片，MarkDown自动换行，不适宜人类阅读
>>> 公式太多太长了，等哪天有空再自己写了更新

## 1. 神经网络 Neural Networks
![fig.1](http://ufldl.stanford.edu/wiki/images/thumb/4/40/Network3322.png/500px-Network3322.png)

- 每一个神经元输入得到输出的函数为激活函数 ![formula.1](http://ufldl.stanford.edu/wiki/images/math/8/9/f/89f1f9e549b908834d9fedca36d07bd4.png)
- 激活函数有sigmoid函数![formula][1]和tanh函数(双曲正切)![formula.3][2],本教程采用sigmoid函数
- 关键参数：
    1. ![formula][3]是第 l 层第 j 单元与第 l+1 层第 i 单元之间的联接参数（其实就是连接线上的weight，注意标号顺序）
    2. ![formula][4]是第 l+1 层第  i 单元的偏置项
    3. ![formula][5]第 l 层第 i 单元的激活值（输出值）
  
**之后的描述中扩展为用向量来表示**  
用 ![f][6]表示第 l 层第 i 单元输入加权和（包括偏置单元），比如，![f][7]，则![f][8]

那么**前向传播**的过程可以表示为  
![f][9]  
  
  
![f][10]  

>将参数矩阵化，使用矩阵－向量运算方式，我们就可以利用线性代数的优势对神经网络进行快速求解。


----------


##  2. 反向传播 Back Propagation

代价函数表示输入输出的偏差，对于单个样例*(x,y)*:  
![f][11] =>二分之一方差代价  
则对于整体，代价函数可以定义为：  
![ff][12]  
第二项为权重衰减项，减少权重幅度防止过拟合
- **目标**：求*（W，b）*让代价函数得到最小值

### 梯度下降法

每次迭代都更新参数：  
![f][13]
$\alpha$是学习速率,**关键步骤是计算偏导数**  
**反向传播算法是计算偏导数的有效算法**  

- 计算  
 ![ff][14]和![f][15]
- 先计算单个样例的偏导数:  
 ![ff][16] & ![f][17]
- 则整体代价函数的偏导数:  
 ![ff][18]  
*注:权重衰减只作用于W*  

**BP算法具体细节**：
1. 前馈传导得出各layer的激活值
2. 对于输出层 *nl* 的每个单元 *i* ，根据以下公式计算**残差**：  
![f][19]  
3. 对于隐藏层已经知道后一层的残差，反向传播可以得：  
![f][20]  
4. 算偏导：
![f][21]  

**梯度下降 gradient descent 总结**：
一次迭代  
1. 对所有 *l* 令 ![f][22] ，![f][23]
2. 对于 i=1 到 i=m
    a. 用反向传播计算每个单元的偏导![f][24]和![f][25]  
    b. 计算每个单元的*W*和*b*的更新  
![f][26]  和  ![f][27]
3. 更新总体权重参数
![f][28]
  
>重复梯度下降法的迭代步骤BackPropagation来减少代价函数的值,进而求解


----------
## 3. 梯度检验和高级优化

#### 梯度检验

- 程序中缺位错误之类的会让结果不是最佳
- 可以对求导结果进行梯度检验:
    - 假设最小化目标函数![f][29]  
    - 一次梯度下降之后 ![f][30] 
    - 转化为![f][31]
- 根据导数定义:
![f][32]
*EPSILON为一个很小的常量在$10^{-4}$数量级*
- **计算等式两边是否相等来检验函数是否正确**
- 然后推导到$\theta$ 是向量的情况
  
#### 高级优化
有许多更精妙的算法去最小化![f][33] 

----------

## 4. 稀疏性

-  ![formula][34]表示隐藏神经元的活跃度
-  ![ff][35]表示给定输入为 *x* 的情况下,自编码神经网络隐藏神经元 *j* 的激活度
-  ![d][36]表示隐藏神经元 j 的平均活跃度（在训练集上取平均）
-  可以加入限制 ![f][37]
-  $\rho$ 是**稀疏性参数**，通常是一个接近于0的较小的值,为满足这一条件隐藏神经元 j 的平均活跃度必须接近0
-  为了实现限制,在优化目标函数中额外增加**惩罚因子**,即相对熵
![f][38]
-  **总体代价函数**表示为
![f][39]
- 相对熵的导数计算
将![ff][40]  
换成![ff][41]




  [1]: http://ufldl.stanford.edu/wiki/images/math/c/e/5/ce5df10952ab30aa868f44db2f77486b.png
  [2]: http://ufldl.stanford.edu/wiki/images/math/a/9/0/a9025d0884453bd5898c9681e871b3fb.png
  [3]: http://ufldl.stanford.edu/wiki/images/math/d/f/e/dfe43c64e3c42ea4ff1774fc82b87805.png
  [4]: http://ufldl.stanford.edu/wiki/images/math/4/c/7/4c786c16575b63bbb554254725b6b648.png
  [5]: http://ufldl.stanford.edu/wiki/images/math/c/9/b/c9b144e0a6735fafb01b3615a2a0dc05.png
  [6]: http://ufldl.stanford.edu/wiki/images/math/3/d/d/3dd5c56e0949e76de86690e1b868cdcf.png
  [7]: http://ufldl.stanford.edu/wiki/images/math/a/a/e/aae7340fe1eb75c824b8abc107c3db27.png
  [8]: http://ufldl.stanford.edu/wiki/images/math/c/f/8/cf8cb56750f5aaca7dc59480a53d9676.png
  [9]: http://ufldl.stanford.edu/wiki/images/math/9/6/9/9690acc03c1e5133b0509257b532b4f7.png
  [10]: http://ufldl.stanford.edu/wiki/images/math/5/c/f/5cfcbbe6d55b6c882f56a85a57eafe6e.png
  [11]: http://ufldl.stanford.edu/wiki/images/math/0/2/9/029cdd402b83ee43c7e9a900dccd675a.png
  [12]: http://ufldl.stanford.edu/wiki/images/math/4/5/3/4539f5f00edca977011089b902670513.png
  [13]: http://ufldl.stanford.edu/wiki/images/math/6/f/e/6fe7c74511cd6d49a4c9cb6de2afdc33.png
  [14]: http://ufldl.stanford.edu/wiki/images/math/5/f/b/5fb8e62e296ad365a076617b04d66d03.png
  [15]: http://ufldl.stanford.edu/wiki/images/math/c/a/4/ca49d387f9ead91008f9688b3880e91b.png
  [16]: http://ufldl.stanford.edu/wiki/images/math/5/f/b/5fb8e62e296ad365a076617b04d66d03.png
  [17]: http://ufldl.stanford.edu/wiki/images/math/c/a/4/ca49d387f9ead91008f9688b3880e91b.png
  [18]: http://ufldl.stanford.edu/wiki/images/math/9/3/3/93367cceb154c392aa7f3e0f5684a495.png
  [19]: http://ufldl.stanford.edu/wiki/images/math/5/7/a/57a203683fc9c009c41ff97c1e1f6f54.png
  [20]: http://ufldl.stanford.edu/wiki/images/math/2/0/f/20f9979d6a46e7bca83f217bdfead4f0.png
  [21]: http://ufldl.stanford.edu/wiki/images/math/2/1/d/21db5874b1c1c14bcb675e9961dac9cb.png
  [22]: http://ufldl.stanford.edu/wiki/images/math/3/6/5/3650852a6b08d261b08a5f4f324fe3a0.png
  [23]: http://ufldl.stanford.edu/wiki/images/math/7/5/b/75bf8778e859c31930f7629fe5eab821.png
  [24]: http://ufldl.stanford.edu/wiki/images/math/d/2/1/d21ff7e7308c9fd8c428fd926f671a39.png
  [25]: http://ufldl.stanford.edu/wiki/images/math/f/e/d/fed489077fe3753c894638d131c0b442.png
  [26]: http://ufldl.stanford.edu/wiki/images/math/5/0/b/50bd90d031437ba98debea738afad0a2.png
  [27]: http://ufldl.stanford.edu/wiki/images/math/3/a/b/3abc7162b757ceac7bdb8f0c4555fe8e.png
  [28]: http://ufldl.stanford.edu/wiki/images/math/0/f/7/0f7430e97ec4df1bfc56357d1485405f.png
  [29]: http://ufldl.stanford.edu/wiki/images/math/c/e/0/ce027336c1cb3c0cd461406c81369ebf.png
  [30]: http://ufldl.stanford.edu/wiki/images/math/a/8/c/a8c1af31e58f9f9f2c55c90b33deace1.png
  [31]: http://ufldl.stanford.edu/wiki/images/math/a/0/1/a01cdafbf71127043a4a5d2d097dfd80.png
  [32]: http://ufldl.stanford.edu/wiki/images/math/c/6/d/c6d06b4a25dab5ef468c72900872eda0.png
  [33]: http://ufldl.stanford.edu/wiki/images/math/c/e/0/ce027336c1cb3c0cd461406c81369ebf.png
  [34]: http://ufldl.stanford.edu/wiki/images/math/4/a/2/4a21cfd212bf2151bef47bbbd8d935d4.png
  [35]: http://ufldl.stanford.edu/wiki/images/math/4/b/3/4b3ea0e74395587b75475e7d1a648104.png
  [36]: http://ufldl.stanford.edu/wiki/images/math/8/7/2/8728009d101b17918c7ef40a6b1d34bb.png
  [37]: http://ufldl.stanford.edu/wiki/images/math/1/8/9/189cc2ce8930608381f0aa234c009cb6.png
  [38]: http://ufldl.stanford.edu/wiki/images/math/8/a/7/8a77066279d89dae4688d9bf4508e0a1.png
  [39]: http://ufldl.stanford.edu/wiki/images/math/7/a/4/7a4ac86b3559db835f4357987252b088.png
  [40]: http://ufldl.stanford.edu/wiki/images/math/a/b/2/ab2e3ac6ec9172f9b2d9b8d3542158dc.png
  [41]: http://ufldl.stanford.edu/wiki/images/math/b/e/f/bef8a29947bfb0d746c54a7d922874e8.png