---
layout:     post
title:      "Paper Reading Note"
subtitle:   "NEURAL NETWORKS WITH FEW MULTIPLICATIONS"
date:       2016-10-11
author:     "Jin"
header-img: "img/post-bg-2015.jpg"
tags:
    - CNN
    - Approximate Computing
    - Paper Reading Note
---

>  Lin Z, Courbariaux M, Memisevic R, et al. Neural networks with few multiplications[J]. arXiv preprint arXiv:1510.03009, 2015.

## ABSTRACT

- 大多数的神经网络训练时间都花在浮点数的运算上，提出一种方法去减少此过程中的大多数乘法的需要
- 方法主要分两步：
    1. 随机二元化权重去转化计算隐藏状态中涉及的乘法
    2. 反向传播的error derivatives，除了二元化权重，还量化每一层的表示以转化剩余的乘法为二进制shift
- 实验结果：3 popular datasets (MNIST, CIFAR10, SVHN)  Theano

-----

## 1. INTRODUCTION

- Time consuming & huge storage 
    - deep neural networks by resorting to GPU or CPU clusters and to well designed parallelization strategies

- Our method deals with both hidden state computations and backward weight updates.Has 2 components：
    1. In the forward pass, **binary connect** or **ternary connect**：weights are stochastically binarized  
    2. In backpropagation of errors, **quantized back propagation** ：converts multiplications into bit-shifts

------

## 2. RELATED WORK

- 限制weight为Integer，把乘法转换为二进制移位，但是这样性能降低且不能保证训练收敛性
- completely Boolean network, simplifies the test time computation at an acceptable performance hit,benefit does not apply to training
- 各种二进制转换，binary network，减少乘法转成integer shift
- 还有一些不是降低权重精度而是量化状态，学习率，梯度power of 2，可以减少乘法with可忽略的性能降低

-------

## 3. BINARY AND TERNARY CONNECT

> 此节重要，建议直接看论文 

- 对于forward pass

### 3.1 BINARY CONNECT REVISITED

- sampling weights to be -1 or 1
- It is necessary to add some edge constraints to W
- Sampling an integer has to be faster than multiplication for the algorithm to
be worth it

### 3.2 TERNARY CONNECT

- 上一小结的1，-1二元连接换成1，0，-1 的三元连接

-----

## 4. QUANTIZED BACK PROPAGATION

- Backward pass
- 分析公式，把可以转化的输入转化为2的整数次幂，从而可以用shift代替乘法

------

## 5. EXPERIMENTS

- Python Theano跑出准确率，分析可以减少多少乘法