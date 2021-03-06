---
title: QLExpress
date: 2021-07-19 11:21:21
categories:
  - java
tags:
  - java
---

由阿里的电商业务规则、表达式（布尔组合）、特殊数学公式计算（高精度）、语法分析、脚本二次定制等强需求而设计的一门动态脚本引擎解析工具。 在阿里集团有很强的影响力，同时为了自身不断优化、发扬开源贡献精神，于2012年开源。

QLExpress脚本引擎被广泛应用在阿里的电商业务场景，具有以下的一些特性:

- 1、线程安全，引擎运算过程中的产生的临时变量都是threadlocal类型。
- 2、高效执行，比较耗时的脚本编译过程可以缓存在本地机器，运行时的临时变量创建采用了缓冲池的技术，和groovy性能相当。
- 3、弱类型脚本语言，和groovy，javascript语法类似，虽然比强类型脚本语言要慢一些，但是使业务的灵活度大大增强。
- 4、安全控制,可以通过设置相关运行参数，预防死循环、高危系统api调用等情况。
- 5、代码精简，依赖最小，250k的jar包适合所有java的运行环境，在android系统的低端pos机也得到广泛运用。



QLExpressRunner如下图所示，从语法树分析、上下文、执行过程三个方面提供二次定制的功能扩展。
![image](/images/qlexpress.jpg)

git 地址：https://github.com/alibaba/QLExpress



