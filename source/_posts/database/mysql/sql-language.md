---
title: sql语句
date: 2022-05-25 20:40:40
categories:
  - database
  - mysql
  - sql
tags:
  - sql
  - mysql
---

## 简介

结构化查询语言（Structured Query Language）简称SQL；
SQL常用的2个部分
数据操作语言（DML）：其余局包括动词INSERT，UPDATE和DELETE。他们分别用于添加，修改和删除表中的行。也称动作语言；
数据定义语言（DDL）：DDL主要用于操作数据库。

## DDL 简单操作

### 数据库操作

- 连接数据库

```shell
mysql -u root -ppwd
```

- 查看数据库

```sql
show databases;
```

- 创建数据库

```sql
create database dbName;
```

- 删除数据库

```sql
drop database dbName;
```

- 修改数据库

```sql
alter database dbName charset = 'utf8'
```

- 查看数据库中所有表

```sql
show tables;
```

- 使用某数据库

```sql
use dbName;
```

### 表操作

- 创建表

```sql
drop if exists t_user;
create table t_user(
    `id` bigint primary key AUTO_INCREMENT,
    `name` varchar(10) COMMENT '姓名',
    `age` int COMMENT '年龄',
    key indexName (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户';
```

- 删除表

```sql
drop table 表名称;
```

#### 修改表

- 添加表字段

```sql
ALTER TABLE `表名称` 
ADD COLUMN `字段名` VARCHAR(255) NULL COMMENT '字段名称';
```

- 修改字段类型

```SQL
ALTER TABLE `表名称` modify `字段名` ctype
```

- 删除表字段

```sql
ALTER TABLE `表名称` drop `字段名`
```

- 修改字段名

```sql
ALTER TABLE `表名称` change `原字段名` `目标字段名` 目标字段类型;
```

## DML 简单操作

- 插入数据

```sql
insert into 表名 values (v11, v12), (v21, v22),...

insert into 表名(f1, f2...) values (v11, v12...),(v21,v22...)...
```

- 删除数据

```sql
delete from 表名 where 条件;
```

- 更新数据

```sql
update 表名 set f1 = v1, f2 = v2 where 条件
```
  
### 查询数据

- 简单查询

```sql
select * from 表名 [where 条件]
select f1, f2 from 表名 [where 条件]
```

- 排序

```sql
select * from 表名 order by f1 (desc | asc), f2 (desc|asc)
```

- 条数

```sql
select * from t limit 0[偏移量], 10[数量];
```

- 去重

```sql
select distinct f1 from t;
```

- 函数

```sql
select count(*) from 表名;
select f1, sum(f2) s from 表名 group by f1;
...
```

- 多表

```sql
select * from t1 inner join t2 on t1.f1 = t2.f2

select * from t1 left join t2 on t1.f1 = t2.f2

select * from t1 right join t2 on t1.f1 = t2.f2
```

- 事务

```sql
-- 事务 commit
begin;
insert into t value (1);
commit;

-- 事务 rollback
begin;
insert into t value (2);
rollback;
```

### 索引操作

- 创建索引

```sql
alter table tableName add (index | unique | fulltext) indexName(columnName);
```

- 删除索引

```sql
drop index [indexName] on tableName;

```
