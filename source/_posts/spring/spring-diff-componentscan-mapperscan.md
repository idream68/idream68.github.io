---
title: spring @ComponentScan和@MapperScan区别
date: 2021-05-10 11:47:40
categories:
  - springboot
tags:
  - springboot
---

- @MapperScan:
  - 扫描mapper类的注解
- @ComponentScan:
  - 扫描使用 @Controller @Service @Repository 注解的类，默认扫描使用@ComponentScan的目录和子目录

在同一个项目中可以同时使用

