---
title: mysql基础
date: 2022-05-24 15:49:29
categories:
  - database
  - mysql
tags:
  - mysql
  - base
---

## 简介

MySQL是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件之一。
MySQL是一种关系型数据库管理系统，关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

## 常用数据类型

- 整数类型

| 类型 | 字节 | 取值范围 |
| -- | -- | -- |
| tinyint | 1 | -128 ~ 127 |
| smallint | 2 | -2 ^ 15 ~ 2 ^ 15 -1 |
| int | 4 | -2 ^ 31 ~ 2 ^ 31 -1 |

- 浮点类型

| 类型 | 字节 |
| -- | -- |
| float | 4 |
| double | 8 |

- 定点数据类型：decimal(M,D)

- 日期时间类型

| 类型 | 名称 | 字节 | 日期格式 | 最小值 | 最大值 |
| -- | -- | -- | -- | -- | -- |
| YEAR | 年 | 1 | YYYY/YY| 1901 | 2155 |
| TIME | 时间 | 3 | HH:MM:SS | -838:59:59 | 838:59:59 |
| DATE | 日期 | 3 | YYYY-MM-DD | 1000-01-01 | 9999-12-31 |
| DATETIME | 日期时间 | 8 | YYYY-MM-DD HH:MM:SS | 1000-01-01 00:00:00 | 9999-12-31 23:59:59 |
| TIMESTAMP | 日期时间 | 4 | YYYY-MM-DD HH:MM:SS | 1970-01-01 00:00:00 UTC | 2038-01-19 03:14:07 UTC |

- 文本字符类型

| 类型 | 长度范围 |
| -- | -- |
| CHAR(M) | 0 - 255 |
| VARCHAR(M) | 0 - 65535 |
| TEXT | 0 - 65535 |
| LONGTEXT | 0 - 4294967295 |

- ENUM 类型
- JSON 类型


[官网](https://dev.mysql.com/)
[配置说明](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html)
