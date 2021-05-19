---
title: maven常用命令
date: 2021-05-08 16:43:20
categories:
  - maven
tags:
  - maven
---

```
mvn package -DskipTests=true -P prod (跳过测试，并使用prod yml 配置文件)
mvn test-compile 编译测试代码（有时候会跳过，原因暂未找到）
mvn compile 编译代码
mvn install:install-file -Dfile=<path-to-file> -DgroupId=<group-id> \
    -DartifactId=<artifact-id> -Dversion=<version> -Dpackaging=<packaging> 生成本地库
```

