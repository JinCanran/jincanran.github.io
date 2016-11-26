---
layout:     post
title:      "Paper Reading Note"
subtitle:   "ApproxANN: An Approximate Computing Framework for Artificial Neural Network"
date:       2016-11-02
author:     "Jin"
header-img: "img/post-bg-2015.jpg"
tags:
    - Neural Network
    - Approximate Computing
    - Paper Reading Notes
---

>  Zhang Q, Wang T, Tian Y, et al. ApproxANN: an approximate computing framework for artificial neural network[C]//Proceedings of the 2015 Design, Automation & Test in Europe Conference & Exhibition. EDA Consortium, 2015: 701-706.

## ABSTRACT

- ApproxANN characterizes 神经元对输出质量的影响，用高效的方式
- 然后决定 how to approximate the computation and memory accesses of certain less critical neurons to achieve the maximum energy efficiency gain under a given quality constraint.

-----

## 1. INTRODUCTION

- 主要工作：
1. 在存储访问和计算过程都考虑了approximation，比现存的那些方法更有效[5,8]
2. **Theoretical neuron criticality analysis technique**, which can be efficiently applied to neural networks with any topologies
3. **New optimization procedure** for approximate ANN construction, wherein error-tolerance capability and energy consumption are jointly considered to achieve the optimized energy savings under a given quality constraint.



------

## 2. PRELIMINARIES

- 人工神经网络简介
- 相关工作
    - [5] 第一个用近似乘法器在人工神经网络里
    - [8] 有过一篇AxNN的paper但是有一些可以改进，本文基本基于此
        - 最主要是AxNN的cirticality判断结果可能不那么令人信服，*原因涉及W的矩阵的秩，暂时没看懂*
> 看完了，有空再写Reading Note 

-------

## 3. PROPOSED DESIGN METHODOLOGY  

### 3.1 Neuron Criticality Analysis
- Critical & Resilient ： 神经元微小的改变造成大的输出质量的下降，那就是critical的神经元，反之就是resilient的神经元
    - 去判断每个神经元的criticality，直观的，我们可以引入随机的error在每个神经元然后看对结果的影响，但是不实际单个神经元对于大型网络来说太渺小了
- 本节 Present our efficient theoretical-based criticality analysis for neurons in output layer and hidden layers

- 公式推导出表示神经元的criticality的式子，然后按照criticality来排列criticality ranking vector

### 3.2 Approximate Artificial Neural Network Architecture
- Neurons in the same layer share the same input vector 𝑥, and their different weight vectors 𝑤s form the weight matrix between the current layer and the pervious layer
- 使用图三的widely used 硬件结构，每个processing element（PE）作为一个神经元包括一系列算术逻辑单元和local memory（LM），只读取相应的weight vector从off-chip memory到LM
- 因为streaming vector x 对于每层的神经元都是一样的所以得到这样的结构（图三）
- **得到data transfer DT 和 computation workload CW的公式，推出数据存取耗能和计算耗能的比例公示**主要有以下三种近似计算方法
    1. Memory Access Skipping
    2. Precision Scaling
    3. Approximate Arithmetic Building Blocks

### 3.3 Optimization Procedure

- 具体问题就是在质量下降的限度内，求能量的最小消耗，如果神经元之间互相没有联系的话，那么就是简单的动态规划问题，但是显然神经元之间相互联系紧密，所以采用一种迭代的启发式方法去决定哪些神经元可以近似
- **图四**——现有相关的一个比较大的 budget for quality degradation，意味着比较多的神经元可以降低质量换取能量效率，随着接近算法的收敛，这个budget会越来越小，那么可以近似的神经元也会变少。因此batch size should be adaptive based on the changing quality budget
- **Algo. 1**——看论文



-----

待续