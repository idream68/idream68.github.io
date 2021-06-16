---
title: shiro + jwt 基础
date: 2021-06-15 20:20:14
categories:
  - spring
  - springboot
  - shiro
tags:
  - springboot
  - shiro
  - jwt
---

### jwt 简介

> JSON Web Token (JWT)是一个开放标准(RFC 7519)，它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。

JSON Web Token由三部分组成，它们之间用圆点(.)连接。这三部分分别是：

- Header
- Payload
- Signature

因此，一个典型的JWT看起来是这个样子的：

> xxxxx.yyyyy.zzzzz

