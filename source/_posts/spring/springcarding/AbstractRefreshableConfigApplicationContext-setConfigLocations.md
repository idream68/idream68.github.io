---
title: ClassPathXmlApplicationContext设置配置文件路径流程
date: 2021-05-11 19:35:35
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
locations=>operation: 检查路径是否为空
resolvePath=>subroutine: 解析路径|:>#解析路径流程
s->locations->resolvePath->e
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

### 解析路径流程

```flow
start=>start: 入口|:>#流程图
end=>end: 返回|:>#流程图
getEnvironment=>subroutine: 获取运行环境|:>#获取运行环境流程 
resolveRequiredPlaceholders=>subroutine: 解析配置路径|:>#解析配置路径流程
tirmSpace=>operation: 去除配置路径的无效空格
start->getEnvironment->resolveRequiredPlaceholders->tirmSpace->end
```

### 流程说明



### 获取运行环境流程

```flow
start=>start: 入口|:>#解析路径流程
end=>end: 返回|:>#解析路径流程
envirmonentExists=>condition: 存在运行环境？
createEnvironment=>subroutine: 创建运行环境 |:> #创建运行环境流程
getEnvironment=>operation: 获取运行环境
start->envirmonentExists
envirmonentExists(yes)->getEnvironment
envirmonentExists(no)->createEnvironment
createEnvironment->getEnvironment
getEnvironment->end
```





### 流程说明



### 创建运行环境流程

```flow
start=>start: 入口|:>#获取运行环境流程
end=>end: 返回|:>#获取运行环境流程
initDefaultProfiles=>operation: 初始化默认配置文件名集合
initPropertySources=>operation: 初始化多配置文件来源
initPropertyResolver=>operation: 初始化普通配置文件解析器
getSystemProperty=>operation: 获取当前系统属性信息
whichGetEnvironment=>condition: 获取当前系统环境信息?
getSystemEnvironment=>operation: 获取当前系统环境信息
start->initDefaultProfiles
initDefaultProfiles->initPropertySources
initPropertySources->initPropertyResolver
initPropertyResolver->getSystemProperty
getSystemProperty->whichGetEnvironment
whichGetEnvironment(yes)->getSystemEnvironment->end
whichGetEnvironment(no)->end
```



### 流程说明

```java
SpringProperties.setProperty("spring.getenv.ignore", "true"); // 控制是否需要获取当前系统环境，默认获取，设置为true:不获取
```



### 解析配置路径流程

```flow
start=>start: 入口|:>#解析路径流程
end=>end: 返回|:>#解析路径流程
helperExists=>condition: 解释器是否存在？
createHepler=>operation: 创建占位符解析器
getHelper=>operation: 获取占位符解析器
resolve=>operation: 解析本地路径
start->helperExists
helperExists(yes)->getHelper
helperExists(no)->createHepler
createHepler->getHelper
getHelper->resolve
resolve->end
```

### 流程说明



