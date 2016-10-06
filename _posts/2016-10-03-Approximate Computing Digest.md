---
layout:     post
title:      "近似计算领域研究框架"
subtitle:   "简单摘要记录"
date:       2016-10-02
author:     "Jin"
header-img: "img/post-bg-2015.jpg"
tags:
    - Appriximate Computing
    - Digest
---

>  主要摘抄一部分维基百科以及概述论文

## Approximate Computing
**论文加粗列出来的都是读过的，在抽屉里**
**斜体是需要接下来去看的**



### 概述
- 大致概念，应用场景，error-tolerent
- 近似计算从门级电路、RTL、逻辑综合到high-level、software-level等等都有研究领域
- Approximate Computing的Metrics（考量指标）：Error rate(ER), Error significance(ES), Error distance(ED), Mean error distance(MED), Normalized error distance(NED)


**[1]** Han J, Orshansky M. Approximate computing: An emerging paradigm for energy-efficient design[C]//2013 18th IEEE European Test Symposium (ETS). IEEE, 2013: 1-6.


### 分方向

#### 1. 逻辑电路level
- 近似计算的门级电路，比如近似全加器（AMAs、AXAs），乘法器，可以允许结果存在几个bit的误差，可以是carry或者是MSB、LSB之类的。
- 这种近似的加法器提供了更小的delay，或者area减少，或low pwer但是结果不精确，但是用于特定的逻辑组合，或者是特定的应用可以起到良好的效果，比如carry的准确度要求不高，但是却可以节约大量的时间这种。
- 郭怡主要研究这一方面的内容
- 上一学期我也看过几篇相关方向的论文，在下面把题目贴上来。大概是关于一个近似的加法器的门级电路设计（MOS管级别的加法器设计），没note记不清楚了。
- 此类论文主要画新电路与旧的近似电路比较有何差异，在某种特定场景有效，还可提出一些额外的方法提高稳定性，比如郭怡发过的有recover调节的[4]

**[1]** Almurib H A F, Kumar T N, Lombardi F. Inexact designs for approximate low power addition by cell replacement[C]//2016 Design, Automation & Test in Europe Conference & Exhibition (DATE). IEEE, 2016: 660-665.  
**[2]** Gupta V, Mohapatra D, Raghunathan A, et al. Low-power digital signal processing using approximate adders[J]. IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems, 2013, 32(1): 124-137.  
[3] Yang Z, Jain A, Liang J, et al. Approximate XOR/XNOR-based adders for inexact computing[C]//Nanotechnology (IEEE-NANO), 2013 13th IEEE Conference on. IEEE, 2013: 690-693.  
[4] Liu C, Han J, Lombardi F. A low-power, high-performance approximate multiplier with configurable partial error recovery[C]//Proceedings of the conference on Design, Automation & Test in Europe. European Design and Automation Association, 2014: 95.  
#### 2. 逻辑综合level

- 用各种各样的方法比如卡诺图之类，想办法化简逻辑表达式之类的方法,**对于给定的function想办法化简出一个合理的approximate的version**
- 减少literals的数量，从而减少实际电路中门级组件的数量
- 我做过一次zeminar关于这个方面，还有PPT(2016/06/15).

**[1]** Shin D, Gupta S K. Approximate logic synthesis for error tolerant applications[C]//Proceedings of the Conference on Design, Automation and Test in Europe. European Design and Automation Association, 2010: 957-960.  
[2] Liu C, Han J, Lombardi F. A low-power, high-performance approximate multiplier with configurable partial error recovery[C]//Proceedings of the conference on Design, Automation & Test in Europe. European Design and Automation Association, 2014: 95.    

#### 3. Approximate storage

- 在数据传输或者存储的过程中可以引入近似计算
- 一部分是数据传输的时候可以进行适当的lower-bits truncating，或者不重要的数据可以存在低能耗但是却不稳定的
- 看过一篇paper是提出一种近似系统有两种存储单元，一种稳定高功耗，一种稳定低功耗，然后切换着使用
 
**[1]** Tagliavini G, Rossi D, Benini L, et al. Synergistic Architecture and Programming Model Support for Approximate Micropower Computing[C]//2015 IEEE Computer Society Annual Symposium on VLSI. IEEE, 2015: 280-285.  

#### 4. Software level

- 这一级比较复杂一些，主要包括近似计算引入编程的框架Programing、支持近似计算的声明以及相应的编译器、近似计算的思想在算法层的使用等方面
    1.  基于现有的语言（比如DSL中的OpenMP）做的部分框架项目[1], 这篇post了reading note
    2.  编程框架：**Green**、Ener等等；完整的编译语言和编译器支持[2][3]
> Green 会在之后再做读书笔记，有需要会看paper[3]

  
  
**[1]** Silvano C, Agosta G, Cherubin S, et al. The ANTAREX approach to autotuning and adaptivity for energy efficient HPC systems[C]//Proceedings of the ACM International Conference on Computing Frontiers. ACM, 2016: 288-293.  
**[2]** *Baek W, Chilimbi T M. Green: a framework for supporting energy-conscious programming using controlled approximation[C]//ACM Sigplan Notices. ACM, 2010, 45(6): 198-209.*  
[3] Ansel J, Wong Y L, Chan C, et al. Language and compiler support for auto-tuning variable-accuracy algorithms[C]//Proceedings of the 9th Annual IEEE/ACM International Symposium on Code Generation and Optimization. IEEE Computer Society, 2011: 85-96.  
**[4]** Tagliavini G, Rossi D, Benini L, et al. Synergistic Architecture and Programming Model Support for Approximate Micropower Computing[C]//2015 IEEE Computer Society Annual Symposium on VLSI. IEEE, 2015: 280-285.  

#### 5. Algorithm level

此部分感觉最难，可能需要深入理解具体算法
- 调节算法准确度和效率，需要一定程度上了解算法[1]
- 类似[2],调节信号位数，autotuning可以动态调节，DCT[3]
- 调节硬件电压什么的算法，不打算看
- 整体的比如在计算用例reuse上引入approximate，这种比较简单,做过一次zeminar（2016/05/27）[4]

*[1] Chippa V K, Mohapatra D, Raghunathan A, et al. Scalable effort hardware design: exploiting algorithmic resilience for energy efficiency[C]//Proceedings of the 47th Design Automation Conference. ACM, 2010: 555-560.*  
**[2]** Silvano C, Agosta G, Cherubin S, et al. The ANTAREX approach to autotuning and adaptivity for energy efficient HPC systems[C]//Proceedings of the ACM International Conference on Computing Frontiers. ACM, 2016: 288-293.  
*[3] Park J, Choi J H, Roy K. Dynamic bit-width adaptation in DCT: an approach to trade off image quality and computation energy[J]. IEEE transactions on very large scale integration (VLSI) systems, 2010, 18(5): 787-793.*   
**[4]** He X, Yan G, Han Y, et al. ACR: Enabling computation reuse for approximate computing[C]//2016 21st Asia and South Pacific Design Automation Conference (ASP-DAC). IEEE, 2016: 643-648.  


