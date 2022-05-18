---
title: mongoDB 基础
date: 2022-05-11 14:49:49
categories:
  - database
  - mongodb
tags: 
  - mongodb
---

## 概述

1. mongoDB是一种非关系型数据库
2. 每条记录都是一个json文档。使用json文档的优点：
   - 兼容许多编程语言
   - 减少连接
   - 可动态改变数据结构
3. mongoDB将文档存入到集合中，集合类似于关系型数据库的表
   - 也支持视图（从3.4开始）

## 主要特点

1. 高性能
   mongoDB提供高性能持久化数据操作
   - 支持嵌入文档和数组的索引，可更快查询
   - 嵌入式数据模型减少系统I/O开支
2. 高可用性
   数据多副本集，支持自动故障转移功能。副本集是一组 MongoDB 服务器，它们维护相同的数据集，提供冗余并提高数据可用性。
3. 水平可扩展性
   mongoDB将水平可扩展作为核心功能的一部分，包括：
   - 分片将数据分布在一组机器上
4. 支持多个搜索引擎
   - WiredTiger 存储引擎
   - 内存存储引擎



[官方文档](https://www.mongodb.com/docs/manual/)
