---
layout:     post
title:      "Paper Reading Note"
subtitle:   "The ANTAREX Approach to Autotuning and Adaptivity"
date:       2016-09-30
author:     "Jin"
header-img: "img/post-bg-2015.jpg"
tags:
    - Paper Reading Notes
    - Paradigms
---

>  DSL:domain Specific Language 
   HPC:High Performance Computing

## ABSTRACT

1. ANTAREX project 通过 DSL 实现runtime manage、自动调节（autotune）应用在green and heterogeneous HPC系统，利用近似计算实现超大规模计算。
2. DSL :
	- energy-efficiency
	- performance
	- 适应性策略
	- resource and power management

--------------

##1. INTRODUCTION

1. HPC现在趋势主要是提高规模
2. 带来的挑战和问题主要是高性能计算规模变大，limiting energy envelop, *大概是能量不够用带来的问题*
- 为解决此问题，欧洲方面主要的研究focus on building supercomputer out of less power-hungry ARM cores and GPGPUs.
	- 但是设计和实现HPC APP需要精通很多特殊的语言和performance tuning tools
	- 因此，这种策略和现在的发展方向不符合——open HPC to wide user
	- 所以现在HPC center staff去负责这个应用实现的这种模式是不合适，长远来看是不可持续的
	- **因此本论文的这个方法即利用 DSL可以让developer work effectively on these platform**
	    - 可以分开不同的职能，自适应和能量效率的设计层面（Aproximate Computing可在此引入）可以和应用的功能设计分开  
	    
####The new DSL  


- AOP(面向切面编程 Spring框架也有相关概念)
- express at compile time the adapttivity/energy/performance strategies
- enforce at runtime application autotuning and resource and power management
- to support the parallelism，scalability and  adaptivity of a dynamic workload by exploiting the full system capabilities

####Two use cases  


1. a biopharmaceutical application for drug discovery deployed on the 1.21 PetaFlops heterogeneous NeXtScale Intel-based IBM system at CINECA
2. a self-adaptive navigation system for smart cities deployed on the server-side on the 1.46 PetaFlops heterogeneous Intel® Xeon Phi™ based system

####ANTEREX approach and related tool flow  


![Figure 1 on paper]

- application functionality is expressed through **C/C++**
- non-functional aspects of application, including parallelisation, mapping, adaptivity through the **DSL** developed in project
- app developer or system integrator leverage DSL template -> encapsulate specific mechanisms  
   - how to generate OpenCL or OpenMP parallelisation
   - how to interact with the runtime resource manager
- The DSL weaver and refactoring tool
    - generete a version of code include: necessary library & partition between the code for general-purpose processors & the code for accelerators.
    - 现成的 X custom的compiler, balancing development effort and optimization level
    - **具体流程涉及大量compiler的知识，e.g. split-compilation、software knobs.**
- application control code contains also runtime monitoring and adaptivity strategies code from DSL.

- Thus, the application is continuously monitored to guarantee the required Service Level Agreement (SLA), while communication with the runtime resource-manager takes place to control the amount of processing resources needed by the application. The application monitoring and autotuning will be supported by a runtime layer implementing an application level collect-analyse-decide-act loop. 

> 意思就是，通过这种编译结构，应用执行时持续runtime监测和调控资源用量来达到SLA（Service Level Agreement）。 和 runtime资源管理器的交互 取代了之前的 控制整个应用所需要的运算处理资源 的模式。
支持应用的监测和autotuning 通过实现在runtime layer的 application level 的 收集-分析-决策-执行的循环。


----------


##2. THE ANTAREX DSL

HPC 需要适应多样的操作和环境的情况
1. In contextual information(workload)
2. In requirement(deadline, energy)
3. In resource available(connectivity, number of processor nodes avalible)

最简单的方法employ hard code of条件表达式和参数，去适应both adaptation specification and implementation.

**这篇paper的方法**：
Runtime adaptability strategies relies on DSL(implementing key cponcept of AOP).
**基于idea that 某些应用或者系统的requirements 需要和功能定义的代码分开**，需求定义在DSL的层面上。
所以有一个weaver 合并常规的编译步骤和DSL AOP的需求。
这种AOP策略可以更干净的程序代码和提高效率比如程序的复用率。

- 开发一个HPC应用需要两个方面的experts（application-domain & HPC system architect），通过这种方式可以分开两部分职能（程序的功能部分&额外的切面）.

####2.1 LARA for Runtime Adaptivity and Compiler Optimization
LARA是一种AOP语言办法用于嵌入式系统，允许开发者封装非功能性的requirements，专注于策略框架之类的。让这部分和功能性的代码解耦和。  
与其他approaches的代码注入方法不同，LARA提供了access to 其他的功能例如代码重构、编译优化、包容添加的信息等。 
通过这些可以让编译器得到更有效率的实现。

######Example Codes

- Code 1 :一种LARA strategy ，**转换所有特定类型的申明 to 目标的类型（e.g. doule to float）** 在某种给定的function或者是calls from this function。
- 递归遍历所有调用符合condition的就apply转换。

```
aspectdef ChangePrecision

    input funcName, oldType, newType end
    
    /* change type of variable declarations found
    * inside the function */
    select func{funcName}.decl end
    apply
        def native_type = newType;
    end
    condition
        $decl.native_type == oldType
    end
    
    /* do the same with the function parameters ... */
    /* do the same with the function return type ... */
    
    /* recursively, do the same on functions that are
    * called inside this function */
    select func{funcName}.fCall end
    apply
        call ChangePrecision($fCall.name, oldType, newType);
    end
end
```
>三步的LARA：
 1. select statement
 2. apply statement
 3. conditon statement 给 apply增加限制条件
这段代码简单好理解,但实际情况要考虑的情况复杂很多。


- Code 2：**clone an existing function and change its type**
- import 并使用了之前code1定义的aspect

```
import ChangePrecision;

aspectdef CreateFloatVersion

    input funcName, suffix end
    
    /* clone the target functions and the child calls */
    call cloned : CloneFunctions(funcName, ’’, suffix);
    
    /* change the precision of the cloned functions */
    for (var newFunc of cloned.newFunctions)
        call ChangePrecision(newFunc, ’double’, ’float’);

end

aspectdef CloneFunctions

    input funcName, callerFunction, suffix end
    output newFunctions = [] end
    
    var newName = funcName + suffix;
    
    /* clone the target function */
    select func{funcName} end
    apply
        exec Clone(newName);
        newFunctions.push(newName);
    end

    /* recursively, do the same on functions that are
    * called inside this function ... */
    
    /* change function calls to the cloned function */
    select func{callerFunction}.fCall{funcName} end
    apply
        def name = newName;
    end
end
```

- Code 3：**Main Aspect**
- 主要功能：根据monitor传来的参数调整应用的源码来决定调用*SumOfInternalDistancies*函数的哪个版本（类似高精度版本和低精度高效率版本，由Code2类似代码产生）
- 涉及自动调节（autotuning）的概念，application被修改可以去测量时间以及和autotuner交流必要信息。然后autotuner 回馈optional信息来决定调用哪个版本的函数。
- 在ANTAREX里提出的LARA-aided source-to-source compiler的一个重要特性就是能够重构应用代码去实现adapitivity的行为和设计，可以被ANTAREX 自动调节组件所接收和利用。

```
import CreateFloatVersion;

aspectdef Main

    input
        funcName = ’SumOfInternalDistancies’,
        suffix = ’_f’
    end

    /* create the new (float) version */
    call CreateFloatVersion(funcName, suffix);
    
    /* ... */
    /* create the monitors */
    call monitorOld : TimerMonitor(timerOld);
    call monitorNew : TimerMonitor(timerNew);
    
    /* ... */
    /* add the code to switch between versions */
    select file.fCall{funcName} end
    apply
        insert before ’/*’;
        insert after ’*/’;
        insert after %{
        /* the new code 插入代码为了记录函数执行时间，然后调用合适的autotuner*/
        switch(version) {
        case 0:
            [[monitorOld.start]]
            internal = [[funcName]](atoms, 1000);
            [[monitorOld.stop]]
            version = call_autotuner("[[funcName]]",
                [[monitorOld.get]]);
            break;
        case 1:
            [[monitorNew.start]]
            internal = [[newName]](atoms_f, 1000);
            [[monitorNew.stop]]
            version = call_autotuner("[[funcName]]",
                [[monitorNew.get]]);
            break;
        }
        }%;
    end
end
```

>这段代码有点抽象，没太懂call-autotuner这一步里头是啥样的，如果有更多部分源代码就好了，不过大致知道修改源码和与autotuner交互是这么个意思。和之后的代码4结合起来看就能看懂。

- Code 4：**Excerpt of the resulting C code**
```
/* the new code */
switch( version ) {
case 0:
    timer_start( tim_old );
    internal = SumOfInternalDistancies(atoms, 1000);
    timer_stop( tim_old );
    version = call_autotuner("SumOfInternalDistancies",
        timer_get_time( tim_old ));
break;
case 1:
    timer_start( tim_new );
    internal = SumOfInternalDistancies_f(atoms_f, 1000);
    timer_stop( tim_new );
    version = call_autotuner("SumOfInternalDistancies",
        timer_get_time( tim_new ));
    break;
}
```

####ANTAREX DSL Concepts

Current LARA 提供了基础去build更复杂的DSL，让我们可以去指定Runtime Adaptability策略。 
  
ANTARX DSL：
1. 分离和表达数据传递和计算并行化
2. 给现存的程序模型增加capabilities by 传递提示和metadata给编译器以得到更好的优化
3. 提高性能*可移植性（？）*同时又尊重（保留）现有的编程框架如OpenCL
4. This is done by 利用DSL的能力去自动探索平行代码的配置空间*（抽象，没理解）*

- Iterative compliation（迭代的编译）技术 具有诱惑力去辨识给定代码的最好的编译优化通过考虑可能的trade-off.
- Given 异质多处理器的多样性和通过runtime information来优化的潜能，runtime optimization 是desirable的
- To combine the two approach（**Iterative compliation & Runtime optimization**），**split compilation** will be used
    - 把编译步骤split到两步里 ： offline 和 online
    - 并且移除尽可能多的复杂性在offline step，传递结果给runtime optimazers
    - **使用这种技术可以得到显著的效果在程序的性能方面以及解放程序员重复的优化测试编译的负担**

-----
##3. SELF-ADAPTIVITY & AUTOTUNING

系统的适应性和自动调节是key issue在HPC system中，HPC system需要即时应对去改变工作量和时间，并且不能影响太多额外的功能单位，像能量或热量特征这种

motivation： 在所有可能的应用部署方式之中得到最大的效率或者性能，考虑到快速增长的计算基础infrastructure（提高计算节点数&利用处理器异质性提高性能）

- 所以有需求application become adaptive -> computing resource 
- 在这个方面另一个有趣的影响是在服务器端和应用端有增长的需求去保证SLA（service level agreement）
- 这种需求和应用的performance有关但同样也和能分配给特定计算的最大能量预算有关
- efforts 主要在两个方面
    1. 开发autotuning framework去配置和适用application-level parameters
    2. 应用精度自动调节的概念给HPC application

####Application Autotuning
White-boxes & Black—boxes
White-boxes：基于autotuning lib并且深入使用领域中特定技术去加速paramter space的方法
Black-boxes：不需要任何底层应用的知识，但是苦于很长的收敛时间和更少的custom（定制）的可能性

The propose framework => grey-box approach
 - 始于不需要领域知识，可**依靠代码注释**去缩小查找空间 —— 自动调节只看确定的subspace  
 - framework 含有一个Application monitoring loop 去触发程序自调节
 - 扩展：
    1. on-line learning => update knowdge from the data collected by the monitors -> autotune the system according to the most recent operating conditions.
    2. Machine learning => decision making engine to support autotuning by predicting the most promising set of parameter
    
####Precison Autotuning

- 改变/指定精度，是有前途的方法，来实现performance和power trade-off，如果一个应用可以承受某种程度上的结果质量损失。  
- 在ANTAREX里这种精度调节可以得到很好的结果with the domain experts
of the two use cases
- 自动动态优化
>In both cases,ANTAREX DSL 都是关键去解耦和 使应用的功能的模块和 软件knobs以及精度调节部分分离开来

-----
##4. APPLICATION SCENARIOS

The ANTAREX project is driven by two industrial HPC applications chosen to address the self-adaptivity and scalability characteristics of two highly relevant scenarios towards the Exascale era.

####User Case 1：Computer Accelerated Drug Discovery

- compute-intensive，而且设计很多化学匹配性以及亲和力对接预测之类的
- massively parallel，since the verification of each
point in the solution space requires a widely varying time，所以计算时间不平衡并且不可预测。
- 不同类型的任务可能在不同类型处理器更有效，特别在复杂的体系
- 动态负载均衡和任务布置可以有效解决这类问题

####User Case 2：Self-Adaptive Navigation System

- 在合理的工作量内找到对于现存的道路网络的最佳利用
- 在服务器和传统移动导航用户之间传递contextual information
- 可以克服现有系统缺陷，利用服务器和客户端之间的计算能力协同作用
- 此类系统高效的操作依赖于平衡数据的收集，大数据分析，极端计算能力

####Target Platforms

HPC系统不太懂，直接把原文贴上：
> The target platforms are the CINECA’s Tier-1 IBM NeXtScale hybrid Linux cluster, based on Intel TrueScale interconnect as well as Xeon Haswell processors and MIC accelerators[3], and IT4Innovations Salomon supercomputer, which is a PetaFlop class system consisting 1008 computational nodes. Each node is equipped with 24 cores (two twelve-core Intel Haswell processors). These computing nodes are interconnected by InfiniBand FDR and Ethernet networks. Salomon also includes of 432 compute nodes with MIC accelerators.

-----
##5. EARLY EVALUATION
- early analysis of the combined impact of parallelisation, precision tuning, compiler optimizations
- miniapp extracted from the Drug Discovery application implements one of the most time consuming computational kernel of the LiGen de-novo drug design software
- 实现了两个功能(SumOfIntervalDistance & MeasureOverlap)，都是计算密集比较有代表性的，没有改源代码，保留类继承之类的，但所有不相关的都剥离了
    1.  4000 atoms per molecule and 400 ligands
    2.  **parallelisation** via a DSL template which instantiate on the outermost loop of each kernel an OpenMP parallel for directive with reduction
    3.  **precision tuning** allowing the computation to be run in any of a range of floating
point types, as well as using integers
    4. applied all highlevel optimization flags (-Ox) exposed by GCC
    5. Table1. Design Space for the combined set of optimizations
    6. Three target machines: a quad-core Intel i5, a 4x quad-core AMD NUMA, and a 2x octa-core Intel Xeon with hyperthreading
    7.The **baseline** the most effective optimization level using maximum precision and 16 OpenMP threads, for each hardware platform and each kernel  
	
	
**Table 1 :**


Parameter | Values|
----|------
OpenMP Threads | none, 2, 4, 8, 16
Precision | __float128, double, float, int 
Optimization | -O0, -O1, -O2, -O3, -O



- 表二记录了每个kernel和硬件配置平台的参数组合提供最佳的加速比over the baseline，并限制误差小于5%
- timing是30次相同输入和配置的计算取平均（**最快的版本最大并行（16）和精度（__float128）**）
- For the MeasureOverlap kernel，在 Xeon sever 上并行的代价超过了它的好处
- Int精度不行，单精度和双精度浮点比较好的匹配数据（原始数据是双精度浮点）在速度和精度上
- 并行减少比顺序版本更加精确，所以__float128提供更好精度给SumOfIntervalDistance的顺序版本，实际上并行的代码从来不需要它
- 由于预期更积极的优化级别经常无法与目标架构相匹配，所以需要更好的控制编译器的转换。


**Table 2 :**


Kernel  |Hardware |OpenMP Threads | Precision | Optimization | Speedup  |Error
----|-------|----|----|----|----|----
MeasureOverlap |4-core Intel i5-2500 3.30GHz |16 |double |-Os |6.1 |0.0%
SumOfIntervalDistance| 4-core Intel i5-2500 3.30GHz |4 |double |-Os |4.0 |0.0%
MeasureOverlap| 4x4-core AMD Opteron 8378 2.40GHz| 16| float| -O3 |2.7 |0.0%
SumOfIntervalDistance |4x4-core AMD Opteron 8378 2.40GHz| 16| double| -Os| 2.7 |0.0%
MeasureOverlap |2x8-core Intel Xeon E5-2630v3 2.40GHz| none| double| -O2 |2.4| 0.0%
SumOfIntervalDistance| 2x8-core Intel Xeon E5-2630v3 2.40GHz| 16v float| -Ofast| 3.6 |0.00065%


----------
##6. CONCLUSIONS

各种好处，主要是分离功能性模块和额外功能模块，解耦合，前面反复提到
本文提出的DSL主要是控制计算精度，并且可以并行通过引入openMP