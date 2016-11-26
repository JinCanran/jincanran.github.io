---
layout:     post
title:      "Theano Studying Note"
subtitle:   "theano.scan 使用学习"
date:       2016-11-26
author:     "Jin"
header-img: "img/post-bg-2015.jpg"
tags:
    - CNN
    - Study Notes
    - Theano
---

> StackOverflow上问了蠢问题却没有人解答

## 1. Theano使用初步体会
只看了一遍文档就照着代码开搞，还算不上入门了，主要还是靠问小峰峰解决大多数问题  

- Theano函数一开始先建立声明的变量之间的关系形成一个Graph，这时候还没有具体的值（算是一种符号化的运算关系），调用函数的时候才传入具体的值，所以在计算中途转化Theano的tensor就会特别不合适
- 一开始直接按python的习惯写代码，building的过程中就会各种出错，特别是涉及到非矩阵操作，比如对矩阵里的各个位置具体判断的时候，这样的操作稍微多一点就不行，因为Theano中矩阵是一个整体符号填充的单位，这样才能在GPU下加速整个矩阵计算
- 感觉调试非常不方便，如果是Theano内部逻辑结构写错一点就要build很久以后才知道对不对

-------------------------------

## 2. Scan的使用
 
 Theano中的loop，根据文档描述，比for循环快一点点....
 >- Advantages of using scan over for loops:
    - Number of iterations to be part of the symbolic graph.
    - Minimizes GPU transfers (if GPU is involved).
    - Computes gradients through sequential steps.
    - Slightly faster than using a for loop in Python with a compiled Theano function.
    - Can lower the overall memory usage by detecting the actual amount of memory needed.
    
使用感受：这不只是快一点点，不用这个根本没法跑好不好

 函数声明：

    theano.scan(fn, sequences=None, outputs_info=None, non_sequences=None, n_steps=None, truncate_gradient=-1, go_backwards=False, mode=None, name=None, profile=False, allow_gc=None, strict=False)

 
首先scan的函数参数的定义以及返回值的使用和theano function基本差不多，参照文档的五个示例代码基本都能学会使用
1. fn：传入循环调用的函数的名字，可以为匿名函数lambda（不复杂的操作匿名函数直接写在变量之后：的后面）
2. sequnces： 需要迭代或者遍历的变量，可以传多个，只要顺序和fn函数的最先几个参数能对应上即可
3. outputs_info： fn输出的初始值，也就是可以在这个这个值上面迭代出最后的结果
4. non_sequences： fn函数用到的其他变量，在迭代过程中不会改变的，同样依次对应传入函数的最后几个参数
5. n_steps：迭代次数可以指定

最后把scan compile成theano function 形如：

    final_result = result[-1]
    power = theano.function(inputs=[A,k], outputs=final_result, updates=updates)

*result = outputs [-1]*可以告诉theano只需要取最后一次迭代结果，theano也会足够智能优化不存保存中间几次迭代结果

文档原文链接：[scan – Looping in Theano][1]


  [1]: http://deeplearning.net/software/theano/library/scan.html