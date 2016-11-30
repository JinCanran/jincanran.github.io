---
layout:     post
title:      "Batch Normalization 学习理解"
subtitle:   "涉及数学不好理解"
date:       2016-11-30
author:     "Jin"
header-img: "img/post-bg-2015.jpg"
tags:
    - Study Notes
    - Paper Reading Notes
---

>   Sergey Ioffe and Christian Szegedy. Batch normalization: Accelerating deep network training by reducing internal covariate shift. ICML, 2015.

## Batch Normalization

### 概述  

- Trick提高网络的训练速度
- 非常有效，提高网络泛化能力
- 应用在激活函数之前，也就是Z（ax+b）上，把Z进行BN之后再传入activation，同样的BN的参数也需update


---------------------------


### Internal Covariate Shift  

- train中,底层网络更新参数将导致后面层输入数据分布的变化
- 只要网络的前面几层发生微小的改变，那么后面几层就会被累积放大下去。
一旦网络某一层的输入数据的分布发生改变，那么这一层网络就需要去适应学习这个新的数据分布，
所以如果训练过程中，训练数据的分布一直在发生变化，那么将会影响网络的训练速度。
- 所以一个简单的想法就是在每一层输入之前增加一个预处理层，那么就可以解决ICS的问题了


#### 原始数据预处理；白化  

- 在UFLDL教程中学过的白化就是一个很强的数据预处理的方式
- 但是白化不适合用在这个地方，原因在于：
	1. 计算量太大
	2. 普通的预处理之后会影响到本层学习所得到的特征的，那不就白干了吗
	
### BN 方法提出

**基于上面的讨论，作者在近似白化预处理之后，又增加了一个变换重构，引入了两个调节参数，这才是算法的关键**

### 简单代码实现

算法分析起来复杂，但其实算法的代码不是很难，需要注意的是对于CNN应用BN的时候卷积层中，一整个特征图是作为一个神经元处理的

- 全连接层：  


			if self.BN == True:                
                self.batch_mean = T.mean(z,axis=0)
                self.batch_var = T.var(z,axis=0)
                
                if can_fit == True:
                    mean = self.batch_mean
                    var = self.batch_var
    
                else:
                    mean = self.mean
                    var = self.var
            
                z = (z - mean)/(T.sqrt(var+self.BN_epsilon))
                z = self.a * z
            
            self.z = z + self.b
            
            # activation function
            y = self.activation(self.z)

		
- 卷积层：  
 

    # batch normalization
    if self.BN == True:
    	
    	# in the convolutional case, there is only a mean per feature map and not per location
    	# http://arxiv.org/pdf/1502.03167v3.pdf   
    	self.batch_mean = T.mean(z,axis=(0,2,3))
    	self.batch_var = T.var(z,axis=(0,2,3))
    			
    	if can_fit == True:
    		mean = self.batch_mean
    		var = self.batch_var
    
    	else:
    		mean = self.mean
    		var = self.var
    
    	z = (z - mean.dimshuffle('x', 0, 'x', 'x'))/(T.sqrt(var.dimshuffle('x', 0, 'x', 'x')+self.BN_epsilon))
    	z = self.a.dimshuffle('x', 0, 'x', 'x') * z
    	
    # bias  
    z = z + self.b.dimshuffle('x', 0, 'x', 'x')
    
    # activation  
    y = self.activation(z)

