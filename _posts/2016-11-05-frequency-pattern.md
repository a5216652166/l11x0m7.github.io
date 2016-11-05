--- 
layout: post 
title: 频繁项集和关联规则
date: 2016-11-05 
categories: blog 
tags: [数据挖掘, 算法] 
description: 简要介绍频繁项集合关联规则，支持度、置信度、相关度，以及Apriori和FP-Growth算法
--- 

# 频繁项集和关联规则

很早之前接触过频繁项集和关联规则挖掘的两个算法Apriori和FP-Growth，但是一直没有在实际中应用过，因此当时学的内容基本都忘得差不多了。最近要交数据挖掘课的作业，因此趁此机会，算是复习一下。以后忘记了，也好翻出来看看，看书真的有点头疼……

## 兴趣度量——模式评估方法

除了置信度和支持度之外，还需要考虑一些用户感兴趣的规则，比如相关度量等。

### 相关度量

#### 提升度

对两个频繁项集A和B，提升度公式如下：

$$lift(A,B)=\frac{P(A{\bigcup}B)}{P(A)P(B)}$$

当提升度小于1，则A的出现和B的出现是负相关的；当提升度大于1，则A的出现和B的出现是正相关的。当提升度等于1，则A和B是独立的。

#### $\chi^2$值

$$\chi^2=\sum{\frac{(观测-期望)^2}{期望}}$$


由于上述两个相关度量和零事务相关，容易收到零事务的影响，因此有时候这两个度量的表现会很差，因此我们推荐使用下面的评估模式，对相关性进行度量。

> **零事务**即$\bar{A}\bar{B}$的个数，表明A和B都不出现。这部分是用户不感兴趣的，应该刨除这部分的影响。

### 其他评估模式

#### 全置信度(all_confidence)

$$all\_conf(A,B)=\frac{sup(A{\bigcup}B)}{max\{sup(A),sup(B)\}}=min{P(A|B),P(B|A)}$$


#### 最大置信度(max_confidence)

$$max\_conf(A,B)=min\{P(A|B),P(B|A)\}$$

#### Kulczynski

$$Kulc(A,B)=\frac12(P(A|B)+P(B|A))$$

#### 余弦

$$cosine(A,B)=\sqrt{(P(A|B){\times}P(B|A))}$$

以上四个度量仅仅和两个条件概率有关，并且都在0到1范围。且具有以下性质：

* 大于0.5为正相关
* 小于0.5为负相关
* 等于0.5为中立

![](http://odjt9j2ec.bkt.clouddn.com/frequency-pattern-interest1.png)

![](http://odjt9j2ec.bkt.clouddn.com/frequency-pattern-interest2.png)

### 不平衡比

虽然上述的度量虽然和零事务都没关系（零不变性，上述6种相关度量只有提升度和卡方值不具有零不变性），但是各个度量的效果却不相同，如下面的例子中，D5和D6的Kluc判定m和c为中立的，而余弦度量和全置信度认为负相关，最大置信度认为正相关。

这样，则需要引入不平衡比来衡量两个项集的不平衡程度。


$$IR(A,B)=\frac{|sup(A)-sup(B)|}{sup(A)+sup(B)-sup(A{\bigcup}B)}$$

IR值越大，说明越不平衡。一般结合IR和上述四种度量来衡量项集的相关性。