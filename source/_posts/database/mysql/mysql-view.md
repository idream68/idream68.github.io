---
title: mysql视图
date: 2021-05-13 11:54:54
categories:
  - database
  - mysql
tags:
  - mysql
---

# 视图概述

视图是由数据库中的一个表或多个表导出的虚拟表，是一种虚拟存在的表，方便用户对数据的操作。

## 视图的概念

视图是一个虚拟表，是从数据库中一个或多个表中导出来的表，其内容由查询定义。同真实表一样，视图包含一系列带有名称的列和行数据。但是，数据库中只存放了视图的定义，而并没有存放视图中的数据。这些数据存放在原来的表中。使用视图查询数据时，数据库系统会从原来的表中取出对应的数据。因此，视图中的数据是依赖于原来的表中的数据的。一旦表中的数据发生改变，显示在视图中的数据也会发生改变。

视图是存储在数据库中的查询的SQL语句，它主要出于两种原因：安全原因，视图可以隐藏一些数据，例如，员工信息表，可以用视图只显示姓名、工龄、地址，而不显示社会保险号和工资数等；另一个原因是可使复杂的查询易于理解和使用。

## 视图的作用

对其中所引用的基础表来说，视图的作用类似于筛选。定义视图的筛选可以来自当前或其他数据库的一个或多个表，或者其他视图。通过视图进行查询没有任何限制，通过它们进行数据修改时的限制也很少。视图的作用归纳为如下几点。

### 简单性

看到的就是需要的。视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件。

### 安全性

视图的安全性可以防止未授权用户查看特定的行或列，使有权限用户只能看到表中特定行的方法，如下：

1. 在表中增加一个标志用户名的列。
2. 建立视图，使用户只能看到标有自己用户名的行。
3. 把视图授权给其他用户。

### 逻辑数据独立性

视图可以使应用程序和数据库表在一定程度上独立。如果没有视图，程序一定是建立在表上的。有了视图之后，程序可以建立在视图之上，从而程序与数据库表被视图分割开来。视图可以在以下几个方面使程序与数据独立。

1. 如果应用建立在数据库表上，当数据库表发生变化时，可以在表上建立视图，通过视图屏蔽表的变化，从而使应用程序可以不动。
2. 如果应用建立在数据库表上，当应用发生变化时，可以在表上建立视图，通过视图屏蔽应用的变化，从而使数据库表不动。
3. 如果应用建立在视图上，当数据库表发生变化时，可以在表上修改视图，通过视图屏蔽表的变化，从而使应用程序可以不动。
4. 如果应用建立在视图上，当应用发生变化时，可以在表上修改视图，通过视图屏蔽应用的变化，从而使数据库可以不动。

# 创建视图

创建视图是指在已经存在的数据库表上建立视图。视图可以建立在一张表中，也可以建立在多张表中。

## 查看创建视图的权限

创建视图需要具有CREATE VIEW的权限。同时应该具有查询涉及的列的SELECT权限。可以使用SELECT语句来查询这些权限信息。查询语法如下：

```sql
SELECT Select_priv,Create_view_priv FROM mysql.user WHERE user='用户名';
```

参数说明：

1. Select_priv：属性表示用户是否具有SELECT权限，Y表示拥有SELECT权限，N表示没有。
2. Create_view_priv：属性表示用户是否具有CREATE VIEW权限。
3. mysql.user：表示MySQL数据库下面的user表。
4. 用户名：参数表示要查询是否拥有权限的用户，该参数需要用单引号引起来。

**示例：**查询MySQL中root用户是否具有创建视图的权限。

```sql
SELECT * FROM mysql.user WHERE user='root';
```

## 创建视图

MySQL中，创建视图是通过CREATE VIEW语句实现的。其语法如下：

```sql
CREATE [OR REPLACE] [ALGORITHM={UNDEFINED|MERGE|TEMPTABLE}]
VIEW 视图名[(属性清单)]
AS SELECT语句
[WITH [CASCADED|LOCAL] CHECK OPTION];
```

参数说明：

1. ALGORITHM：可选项，表示视图选择的算法。
2. 视图名：表示要创建的视图名称。
3. 属性清单：可选项，指定视图中各个属性的名词，默认情况下与SELECT语句中的查询的属性相同。
4. SELECT语句：表示一个完整的查询语句，将查询记录导入视图中。
5. WITH CHECK OPTION：可选项，表示更新视图时要保证在该视图的权限范围之内。

**示例：**创建视图。

```sql
CREATE OR REPLACE VIEW view_user
AS
	SELECT id,name FROM tb_user;
```

**示例：**创建视图同时，指定属性清单。

```sql
CREATE OR REPLACE VIEW view_user (a_id,a_name)
AS
	SELECT id,name FROM tb_user;
```

创建视图时需要注意以下几点：

1. 运行创建视图的语句需要用户具有创建视图（create view）的权限，若加了[or replace]时，还需要用户具有删除视图（drop view）的权限；
2. select语句不能包含from子句中的子查询；
3. select语句不能引用系统或用户变量；
4. select语句不能引用预处理语句参数；
5. 在存储子程序内，定义不能引用子程序参数或局部变量；
6. 在定义中引用的表或视图必须存在。但是，创建了视图后，能够舍弃定义引用的表或视图。要想检查视图定义是否存在这类问题，可使用check table语句；
7. 在定义中不能引用temporary表，不能创建temporary视图；
8. 在视图定义中命名的表必须已存在；
9. 不能将触发程序与视图关联在一起；
10. 在视图定义中允许使用order by，但是，如果从特定视图进行了选择，而该视图使用了具有自己order by的语句，它将被忽略。

# 修改视图

修改视图是指修改数据库中已存在的表的定义。当基本表的某些字段发生改变时，可以通过修改视图来保持视图和基本表之间一致。MySQL中通过CREATE OR REPLACE VIEW语句和ALTER VIEW语句来修改视图。

**示例：**修改视图

```sql
ALTER VIEW view_user
AS
	SELECT id,name FROM tb_user where id  in (select id from tb_user);
```

**说明：**ALTER VIEW语句改变了视图的定义，该语句与CREATE OR REPLACE VIEW语句有着同样的限制，如果删除并重新创建一个视图，就必须重新为它分配权限。

# 删除视图

删除视图是指删除数据库中已存在的视图。删除视图时，只能删除视图的定义，不会删除数据。MySQL中，使用DROP VIEW语句来删除视图。但是，用户必须拥有DROP权限。

**示例：**删除视图。

```sql
DROP VIEW IF EXISTS view_user;
```

# MySQL视图中使用IF和CASE语句

在创建视图时，经常需要使用到MySQL的流程控制语句，如：IF语句和CASE语句。

**示例：**创建MySQL视图中使用IF和CASE语句。

1. 创建员工信息表。

   ```sql
   -- 判断数据表是否存在，存在则删除
   DROP TABLE IF EXISTS tb_staff;
    
   -- 创建数据表
   CREATE TABLE IF NOT EXISTS tb_staff
   ( 
   	id INT AUTO_INCREMENT PRIMARY KEY COMMENT '编号',
   	NAME VARCHAR(50) NOT NULL COMMENT '姓名',
   	sex INT COMMENT '性别（1：男；2：女；）',
   	dept_code VARCHAR(10) COMMENT '部门编号',
   	is_post BIT COMMENT '是否在职（0：否；1：是）'
   ) COMMENT = '员工信息表';
   ```

2. 创建员工视图，在视图中使用IF和CASE语句。

   ```sql
   -- 创建视图
   CREATE OR REPLACE VIEW view_staff
   AS
   	SELECT id 
   	,NAME
   	,IF(sex=1,'男','女') AS sex_name
   	,CASE dept_code
   		WHEN '1001' THEN '研发部'
   		WHEN '1002' THEN '人事部'
   		WHEN '1003' THEN '财务部'
   		ELSE '其他'
   	END AS dept_name
   	,IF(is_post,'在职','离职') AS is_post_name
   	FROM tb_staff
   ;
   ```

原文链接：https://blog.csdn.net/pan_junbiao/article/details/86132535
