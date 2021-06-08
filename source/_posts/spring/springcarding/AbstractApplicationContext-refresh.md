---
title: ClassPathXmlApplicationContext刷新容器流程
date: 2021-05-11 17:18:56
categories:
  - spring
  - spring carding
tags:
  - spring流程梳理
  - spring
---

### 流程图
```flow
s=>start: 入口|:>/2021/05/11/springcarding/ClassPathXmlApplicationContext-main
e=>end: 返回|:>/2021/05/11/springcarding/ClassPathXmlApplicationContext-main
prepare=>subroutine: 准备刷新|:>.
obtainFresh=>subroutine: 告知子类刷新内部bean工厂|:>.
prepareBeanFactory=>subroutine: 准备要使用的bean工厂|:>.
postProcessBeanFactory=>subroutine: 允许上下文子类对bean工厂进行后处理|:>.
invokeBeanFactoryPostProcessors=>subroutine: 调用在上下文中注册的bean工厂处理器|:>.
registerBeanPostProcessor=>subroutine: 注册拦截Bean创建的Bean处理器|:>.
initMessageSource=>subroutine: 初始化上下文的消息源|:>.
initApplicationEventMulticaster=>subroutine: 初始化多播器|:>.
onRefresh=>subroutine: 初始化特殊bean|:>.
registerListeners=>subroutine: 检查侦听器并注册|:>.
finishBeanFactory=>subroutine: 实例化所有非懒加载的单例bean|:>.
finishRefresh=>subroutine: 发布相应的事件|:>.
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



### 准备刷新

```flow
s=>start: 入口
startupDate=>operation: 设置启动时间
setClosedFlag=>operation: 设置关闭标志位
setActiveFlag=>operation: 设置活动标志位
writeLog=>operation: 记录日志
initPropertySources=>subroutine: 初始化占位符资源
validateProperties=>subroutine: 验证所有标记属性都是可解析的
earlyListenersExists=>condition: 早期监听器是否存在
setEarlyListeners=>operation: 设置早期监听器
cleanLinteners=>operation: 清空监听器
addListeners=>operation: 将早期监听器添加到监听器中
setEarlyEnents=>operation: 初始化事件收集器
e=>end: 返回
s->startupDate
startupDate->setClosedFlag
setClosedFlag->setActiveFlag
setActiveFlag->writeLog
writeLog->initPropertySources
initPropertySources->validateProperties
validateProperties->earlyListenersExists
earlyListenersExists(no,left)->setEarlyListeners
earlyListenersExists(yes)->cleanLinteners
cleanLinteners->addListeners
setEarlyListeners->setEarlyEnents
addListeners->setEarlyEnents
setEarlyEnents->e
```

### 流程说明



