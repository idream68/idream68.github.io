---
title: ClassPathXmlApplicationContext主流程图
date: 2021-05-11 15:18:32
categories:
  - spring
tags:
  - spring流程梳理
  - spring
---

### 流程图：

```flow
st=>start: Start
e=>end
create=>operation: 创建容器|:>x
set_config_path=>operation: 设置配置文件路径|:>y
refresh_content=>operation: 刷新容器，加载bean|:>/2021/05/11/springcarding/AbstractApplicationContext-refresh
st(right)->create(right)->set_config_path(right)->refresh_content(right)->e
```

### 方法说明：

1. 创建容器
   1. 类：org.springframework.context.support.AbstractXmlApplicationContext
   2. 说明：org.springframework.context.ApplicationContext 接口的简单实现，通过org.springframework.beans.factory.xml.XmlBeanDefinitionReader 读取xml文件绘制配置图，子类只需要是实现 getConfigResources或者 getConfigLocations 方法。 此外，可以重写getResourceByPath 解释相对路径 或者使用 getResourcePatternResolver  扩展解析相对路径方法
   3. 方法：AbstractXmlApplicationContext(@Nullable ApplicationContext parent)
   4. 说明：根据给定的父容器生成实例
2. 设置配置文件路径
   1. 类：org.springframework.context.support.AbstractRefreshableConfigApplicationContext

   2. 说明：

      在AbstractRefreshableApplicationContext的基础上添加了公共读取本地配置文件的方法。用作基于XML的应用程序上下文实现的基类

   3. 方法：setConfigLocations(@Nullable String... locations)

   4. 方法说明：设置应用容器的本地配置路径，如果为空，则使用适当的默认值
3. 刷新容器，加载bean
   1. 类：  org.springframework.context.support.AbstractApplicationContext
   2. 说明：
   3. 方法：refresh()
   4. 方法说明：  刷新容器，创建bean等

