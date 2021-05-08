---
title: mysql,mongo数据迁移
date: 2021-05-08 16:10:00
categories:
  - mysql
  - mongo
tags:
  - mysql
  - mongo
  - 数据迁移
---

mysql:

使用navicat工具：一条一条导入，性能较差 

详情：https://blog.csdn.net/xuanjiewu/article/details/86152633

使用mysqldump导出导入数据

导出数据

eg: 

导入数据

eg:

详情：https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html#mysqldump-option-examples

使用workbench导入导出数据

详情：https://dev.mysql.com/doc/workbench/en/wb-admin-export-import-management.html

mongo:

mongo tools下载路径：https://docs.mongodb.com/database-tools/installation/installation/

使用mongodump备份数据

eg:

```
mongodump -h <host:port> -d <db> -c <collection> -u <user> -p <password> -o <directory-path>
```

详情：https://docs.mongodb.com/database-tools/mongodump/

使用mongorestore导入数据

eg:

```
mongorestore -h <host:port> -d <db> -c <collection> -u <user> -p <password> --drop <input-path>
```

详情：https://docs.mongodb.com/database-tools/mongorestore/

注意：可能不包含索引（不知道是使用数据源不包含索引原因，还是使用这种方式不包含索引）