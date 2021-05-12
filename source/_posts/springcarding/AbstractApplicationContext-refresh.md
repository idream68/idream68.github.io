---
title: ClassPathXmlApplicationContext刷新容器流程
date: 2021-05-11 17:18:56
categories:
  - spring
tags:
  - spring流程梳理
  - spring
---

### 流程图
```flow
s=>start: 入口|:>/2021/05/11/springcarding/ClassPathXmlApplicationContext-main
e=>end: 返回|:>/2021/05/11/springcarding/ClassPathXmlApplicationContext-main
prepare=>operation: 准备刷新|:>.
obtainFresh=>operation: 告知子类刷新内部bean工厂|:>.
prepareBeanFactory=>operation: 准备要使用的bean工厂|:>.
postProcessBeanFactory=>operation: 允许上下文子类对bean工厂进行后处理|:>.
invokeBeanFactoryPostProcessors=>operation: 调用在上下文中注册的bean工厂处理器|:>.
registerBeanPostProcessor=>operation: 注册拦截Bean创建的Bean处理器|:>.
initMessageSource=>operation: 初始化上下文的消息源|:>.
initApplicationEventMulticaster=>operation: 初始化多播器|:>.
onRefresh=>operation: 初始化特殊bean|:>.
registerListeners=>operation: 检查侦听器并注册|:>.
finishBeanFactory=>operation: 实例化所有非懒加载的单例bean|:>.
finishRefresh=>operation: 发布相应的事件|:>.
s->prepare
prepare->obtainFresh
obtainFresh->prepareBeanFactory
prepareBeanFactory->postProcessBeanFactory
postProcessBeanFactory->invokeBeanFactoryPostProcessors
invokeBeanFactoryPostProcessors->registerBeanPostProcessor
registerBeanPostProcessor->initMessageSource
initMessageSource->initApplicationEventMulticaster
initApplicationEventMulticaster->onRefresh
onRefresh->registerListeners
registerListeners->finishBeanFactory
finishBeanFactory->finishRefresh
finishRefresh->e
```

### 流程说明

