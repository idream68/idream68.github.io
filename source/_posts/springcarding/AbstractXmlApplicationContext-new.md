---
title: ClassPathXmlApplicationContext创建容器流程
date: 2021-05-11 19:27:25
categories:
  - spring
tags:
  - spring流程梳理
  - spring
---

### 流程图
```flow
s=>start: 入口|:>2021/05/11/springcarding/ClassPathXmlApplicationContext-main
e=>end: 返回|:>2021/05/11/springcarding/ClassPathXmlApplicationContext-main
getResourceResolver=>operation: 获取资源解析器(解析本地配置文件等资源)|:>.
setParent=>operation: 设置父容器|:>.
s->getResourceResolver
getResourceResolver->setParent
setParent->e
```


### 流程说明 

1. 获取资源解析器(解析本地配置文件等资源)
   1. 类：org.springframework.context.support.AbstractApplicationContext、

   2. 说明

   3. org.springframework.context.ApplicationContext接口的抽象实现。不要求用于配置的存储类型；
      简单地实现通用上下文功能。使用模板方法设计模式，需要具体的子类来实现抽象方法。
      和普通bean工厂相比较，ApplicationContext检测其内部bean工厂中定义的特殊bean并自动注册一下三个类中定义的bean

      1. org.springframework.beans.factory.config.BeanFactoryPostProcessor BeanFactoryPostProcessors
      2. org.springframework.beans.factory.config.BeanPostProcessor BeanPostProcessors
      3. org.springframework.context.ApplicationListener ApplicationListeners

      消息源类型，多播器类型也会作为bean自动注册

   3. 方法
      AbstractApplicationContext()
   4. 说明
      初始化容器
2. 设置父容器
   1. 类
      org.springframework.context.support.AbstractApplicationContext
   2. 说明
      同上
   3. 方法
      setParent(@Nullable ApplicationContext parent)
   4. 说明
      设置父容器，当父容器存在时，获取父容器的运行环境