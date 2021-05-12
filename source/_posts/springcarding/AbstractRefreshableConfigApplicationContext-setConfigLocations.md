---
title: ClassPathXmlApplicationContext设置配置文件路径流程
date: 2021-05-11 19:35:35
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
locations=>operation: 将路径加载到内存
s->locations->e
```

### 流程说明
1. 将路径加载到内存

   1. 类

      org.springframework.context.support.AbstractRefreshableConfigApplicationContext

   2. 说明

      在AbstractRefreshableApplicationContext的基础上添加读取本地配置文件方法

   3. 方法

      setConfigLocations(@Nullable String... locations)

   4. 说明

      设置应用容器的本地配置路径，如果为空，则使用适当的默认值