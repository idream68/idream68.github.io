---
title: 修改jar包中的配置文件
date: 2021-05-08 15:08:31
categories:
  - java
tags:
  - java
---

有时候经常因为不同开发机器上的一部分配置不同，导致项目中的配置文件有些用户名密码等信息有差异，临时打包的时候经常忘记修改，可以重新打包，但是重新打包如果花费时间过长的时候这样做就太不划算了。因此专门百度了不同的方式，找了一种不需要安装其他工具的方式，综合他们的方法，我详细记录一下我的修改过程（以下过程按照顺序执行，可以跳过某些步骤）：

1. 列出jar文件清单：

   ```
   jar tf abc.jar
   ```

2. 将需要修改的文件解压出来

   ```
   jar xf abc.jar BOOT-INF/classes/application.properties
   ```

此时，会在当前jar包的同级目录下生成一个相对路径文件夹（所要修改的文件就在这里），然后修改文件中的内容

3. 使用修改后的文件替换jar包中对应的文件

   ```
   jar uf abc.jar BOOT-INF/classes/application.properties
   ```