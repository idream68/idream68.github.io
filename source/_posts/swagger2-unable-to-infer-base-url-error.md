---
title: SpringBoot使用Swagger2提示Unable to infer base url错误
date: 2021-05-08 11:57:40
categories:
  - springboot
tags:
  - swagger
  - springboot
typora-root-url: ..
---

**错误信息为：**

Unable to infer base url.
This is common when using dynamic servlet registration or when the API is behind an API Gateway.
The base url is the root of where all the swagger resources are served. For e.g. if the api is available at http://example.org/api/v2/api-docs then the base url is http://example.org/api/.
Please enter the location manually:

![](/images/swagger_error.png)

**解决方法：**

- 在application启动类中未定义@EnableSwagger2注解