---
layout: post
title: 架构设计三大原则
date: 2023-03-02 10:12:15
categories: 系统设计
tags: 极客时间 技术学习笔记
excerpt: 合适原则是最重要的，它决定了对简单原则和演化原则的判断
---

三大原则：

- 合适原则。 合适优于业界领先。
- 简单原则。简单优于复杂。
- 演化原则。演化优于一步到位。



# 合适原则

再好的梦想，也需要脚踏实地实现！

主要体现如下：

1、将军难打无兵之仗。没那么多人，却想干那么多活，是失败的第一个主要原因。

2、罗马不是一天建成的。没有那么多积累，却想一步登天，是失败的第二个主要原因。

3、冰山下面才是关键。没有那么卓越的业务场景，却幻想灵光一闪成为天才，是失败的第三个主要原因。



真正优秀的架构都是在企业当前人力、条件、业务等各种约束下设计出来的，能够合理地
将资源整合在一起并发挥出最大功效，并且能够快速落地。



# 简单原则

团队的压力有时也会有意无意地促进我们走向复杂的方向，因为大部分人在评价一个方案水平高
低的时候，复杂性是其中一个重要的参考指标。

复杂”在制造领域代表先进，在建筑领域代表领先，但在软件领域，却恰恰相反，代表的
是“问题”。



# 演化原则

软件的演化，其实就是不断进行迭代。

软件架构需要根据业务发展不断变化这个本质特点，软件架构设计其实更加类似于大自然“设计”一个生物，通过演化让生物适应环境，逐步变得更加强大。

软件架构设计的过程：

- 设计出来的架构要满足当时的业务需要。
- 架构要不断地在实际应用过程中迭代，保留优秀的设计，修复有缺陷的设计，改正错误的设计，去掉无用的设计，使得架构逐渐完善。
- 当业务发生变化时，架构要扩展、重构，甚至重写；代码也许会重写，但有价值的经验、教训、逻辑、设计等（类似生物体内的基因）却可以在新架构中延续。



合适原则是最重要的，它决定了对简单原则和演化原则的判断，没有以合适为基础，很难判断简单是否能满足业务的需求，演化的起点在哪里。




----
1、《从零开始学架构》 李运华

