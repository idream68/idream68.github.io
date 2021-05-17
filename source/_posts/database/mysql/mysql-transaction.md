---
title: mysql 事务 MVCC
date: 2021-05-13 11:54:00
categories:
  - database
  - mysql
tags:
  - mysql
---

# mysql事务

- 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务
- 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
- 一般来说，事务需要满足4个条件（ACID）：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）
- mysql默认自动提交事务

**原子性**

一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

**一致性**

在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。

**隔离性**

数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

**持久性**

事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

## 事务的隔离级别

### 未提交读（READ UNCOMMITTED）

如果一个事务读到了另一个未提交事务修改过的数据，那么这种隔离级别就称之为未提交读（英文名：READ UNCOMMITTED），示意图如下：

![image](/images/mysql-transaction/mysql-transaction-1.png)

如上图，Session A和Session B各开启了一个事务，Session B中的事务先将id为1的记录的列c更新为'关羽'，然后Session A中的事务再去查询这条id为1的记录，那么在未提交读的隔离级别下，查询结果就是'关羽'，也就是说某个事务读到了另一个未提交事务修改过的记录。但是如果Session B中的事务稍后进行了回滚，那么Session A中的事务相当于读到了一个不存在的数据，这种现象就称之为脏读，就像这个样子：

![image](/images/mysql-transaction/mysql-transaction-2.png)

脏读违背了现实世界的业务含义，所以这种READ UNCOMMITTED算是十分不安全的一种隔离级别。

### 已提交读（Read committed）

如果一个事务只能读到另一个已经提交的事务修改过的数据，并且其他事务每对该数据进行一次修改并提交后，该事务都能查询得到最新值，那么这种隔离级别就称之为已提交读（英文名：READ COMMITTED），如图所示：

![image](/images/mysql-transaction/mysql-transaction-3.png)

从图中可以看到，第4步时，由于Session B中的事务尚未提交，所以Session A中的事务查询得到的结果只是'刘备'，而第6步时，由于Session B中的事务已经提交，所以Session B中的事务查询得到的结果就是'关羽'了。

对于某个处在在已提交读隔离级别下的事务来说，只要其他事务修改了某个数据的值，并且之后提交了，那么该事务就会读到该数据的最新值，比方说：

![image](/images/mysql-transaction/mysql-transaction-4.png)

我们在Session B中提交了几个隐式事务，这些事务都修改了id为1的记录的列c的值，每次事务提交之后，Session A中的事务都可以查看到最新的值。这种现象也被称之为不可重复读。

### 可重复读（Repeatable read）

在一些业务场景中，一个事务只能读到另一个已经提交的事务修改过的数据，但是第一次读过某条记录后，即使其他事务修改了该记录的值并且提交，该事务之后再读该条记录时，读到的仍是第一次读到的值，而不是每次都读到不同的数据。那么这种隔离级别就称之为可重复读（英文名：REPEATABLE READ），如图所示：

![image](/images/mysql-transaction/mysql-transaction-5.png)

从图中可以看出来，Session A中的事务在第一次读取id为1的记录时，列c的值为'刘备'，之后虽然Session B中隐式提交了多个事务，每个事务都修改了这条记录，但是Session A中的事务读到的列c的值仍为'刘备'，与第一次读取的值是相同的。

### 可串行化（Serializable ）

以上3种隔离级别都允许对同一条记录进行读-读、读-写、写-读的并发操作，如果我们不允许读-写、写-读的并发操作，可以使用SERIALIZABLE隔离级别，示意图如下：

![image](/images/mysql-transaction/mysql-transaction-6.png)

如图所示，当Session B中的事务更新了id为1的记录后，之后Session A中的事务再去访问这条记录时就被卡住了，直到Session B中的事务提交之后，Session A中的事务才可以获取到查询结果。


| 隔离级别 | 脏读(Dirty Read) | 不可重复读 (NonRepeatable Read) | 幻读 (Phantom Read) |
| :--: | :--: | :--: | :--: |
| 未提交读（Read uncommitted） | 可能 | 可能 | 可能 |
| 已提交读（Read committed） | 不可能 | 不可能 | 不可能 |
| 可重复读（Repeatable read） | 不可能 | 不可能 | 可能 |
| 可串行化（Serializable ） | 不可能 | 不可能 | 不可能 |

InnoDB默认是可重复读级别的

- 脏读: 脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
- 不可重复读:是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
- 幻读:第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样，幻读是数据行记录变多了或者少了。

简单点总结下他们的区别：脏读是指读取了未修改完的记录，不可重复读指因为被其它事务修改了记录导致某事务两次读取记录不一致，而幻读是指因为其它事务对表做了增删导致某事务两次读取的表记录数不一致问题。

# 链版本(MVCC)

对于使用InnoDB存储引擎的表来说，它的聚簇索引记录中都包含两个必要的隐藏列（row_id并不是必要的，我们创建的表中有主键或者非NULL唯一键时都不会包含row_id列）：

- trx_id：每次对某条聚簇索引记录进行改动时，都会把对应的事务id赋值给trx_id隐藏列。
- roll_pointer：每次对某条聚簇索引记录进行改动时，都会把旧的版本写入到undo日志中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息。

比方说我们的表t现在只包含一条记录：
```
mysql> SELECT * FROM t;
+----+--------+
| id | c      |
+----+--------+
|  1 | 刘备   |
+----+--------+
1 row in set (0.01 sec)
```

假设插入该记录的事务id为80，那么此刻该条记录的示意图如下所示：

![image](/images/mysql-transaction/mysql-transaction-7.png)

假设之后两个id分别为100、200的事务对这条记录进行UPDATE操作，操作流程如下：

![image](/images/mysql-transaction/mysql-transaction-8.png)

> 小贴士： 能不能在两个事务中交叉更新同一条记录呢？哈哈，这是不可以滴，第一个事务更新了某条记录后，就会给这条记录加锁，另一个事务再次更新时就需要等待第一个事务提交了，把锁释放之后才可以继续更新。本篇文章不是讨论锁的，有关锁的更多细节我们之后再说。

每次对记录进行改动，都会记录一条undo日志，每条undo日志也都有一个roll_pointer属性（INSERT操作对应的undo日志没有该属性，因为该记录并没有更早的版本），可以将这些undo日志都连起来，串成一个链表，所以现在的情况就像下图一样：

![image](/images/mysql-transaction/mysql-transaction-9.png)

对该记录每次更新后，都会将旧值放到一条undo日志中，就算是该记录的一个旧版本，随着更新次数的增多，所有的版本都会被roll_pointer属性连接成一个链表，我们把这个链表称之为版本链，版本链的头节点就是当前记录最新的值。另外，每个版本中还包含生成该版本时对应的事务id，这个信息很重要，我们稍后就会用到。

## ReadView

对于使用READ UNCOMMITTED隔离级别的事务来说，直接读取记录的最新版本就好了，对于使用SERIALIZABLE隔离级别的事务来说，使用加锁的方式来访问记录。对于使用READ COMMITTED和REPEATABLE READ隔离级别的事务来说，就需要用到我们上边所说的版本链了，核心问题就是：需要判断一下版本链中的哪个版本是当前事务可见的。所以设计InnoDB的大叔提出了一个ReadView的概念，这个ReadView中主要包含当前系统中还有哪些活跃的读写事务，把它们的事务id放到一个列表中，我们把这个列表命名为为m_ids。这样在访问某条记录时，只需要按照下边的步骤判断记录的某个版本是否可见：

- 如果被访问版本的trx_id属性值小于m_ids列表中最小的事务id，表明生成该版本的事务在生成ReadView前已经提交，所以该版本可以被当前事务访问。
- 如果被访问版本的trx_id属性值大于m_ids列表中最大的事务id，表明生成该版本的事务在生成ReadView后才生成，所以该版本不可以被当前事务访问。
- 如果被访问版本的trx_id属性值在m_ids列表中最大的事务id和最小事务id之间，那就需要判断一下trx_id属性值是不是在m_ids列表中，如果在，说明创建ReadView时生成该版本的事务还是活跃的，该版本不可以被访问；如果不在，说明创建ReadView时生成该版本的事务已经被提交，该版本可以被访问。

如果某个版本的数据对当前事务不可见的话，那就顺着版本链找到下一个版本的数据，继续按照上边的步骤判断可见性，依此类推，直到版本链中的最后一个版本，如果最后一个版本也不可见的话，那么就意味着该条记录对该事务不可见，查询结果就不包含该记录。

在MySQL中，READ COMMITTED和REPEATABLE READ隔离级别的的一个非常大的区别就是它们生成ReadView的时机不同，我们来看一下。

###  COMMITTED --- 每次读取数据前都生成一个ReadView

比方说现在系统里有两个id分别为100、200的事务在执行：

```sql
# Transaction 100
BEGIN;
 
UPDATE t SET c = '关羽' WHERE id = 1;
 
UPDATE t SET c = '张飞' WHERE id = 1;
```

```sql

# Transaction 200
BEGIN;
 
# 更新了一些别的表的记录
...

```

> 小贴士： 事务执行过程中，只有在第一次真正修改记录时（比如使用INSERT、DELETE、UPDATE语句），才会被分配一个单独的事务id，这个事务id是递增的。

此刻，表t中id为1的记录得到的版本链表如下所示：

![image](/images/mysql-transaction/mysql-transaction-10.png)

假设现在有一个使用READ COMMITTED隔离级别的事务开始执行：

```sql

# 使用READ COMMITTED隔离级别的事务
BEGIN;
 
# SELECT1：Transaction 100、200未提交
SELECT * FROM t WHERE id = 1; # 得到的列c的值为'刘备'
```

这个SELECT1的执行过程如下：

1. 在执行SELECT语句时会先生成一个ReadView，ReadView的m_ids列表的内容就是[100, 200]。
2. 然后从版本链中挑选可见的记录，从图中可以看出，最新版本的列c的内容是'张飞'，该版本的trx_id值为100，在m_ids列表内，所以不符合可见性要求，根据roll_pointer跳到下一个版本。
3. 下一个版本的列c的内容是'关羽'，该版本的trx_id值也为100，也在m_ids列表内，所以也不符合要求，继续跳到下一个版本。
4. 下一个版本的列c的内容是'刘备'，该版本的trx_id值为80，小于m_ids列表中最小的事务id100，所以这个版本是符合要求的，最后返回给用户的版本就是这条列c为'刘备'的记录。

之后，我们把事务id为100的事务提交一下，就像这样：

```sql

# Transaction 100
BEGIN;
 
UPDATE t SET c = '关羽' WHERE id = 1;
 
UPDATE t SET c = '张飞' WHERE id = 1;
 
COMMIT;
```

然后再到事务id为200的事务中更新一下表t中id为1的记录：

```sql
# Transaction 200
BEGIN;
 
# 更新了一些别的表的记录
...
 
UPDATE t SET c = '赵云' WHERE id = 1;
 
UPDATE t SET c = '诸葛亮' WHERE id = 1;
```

此刻，表t中id为1的记录的版本链就长这样：

![image](/images/mysql-transaction/mysql-transaction-11.png)

然后再到刚才使用READ COMMITTED隔离级别的事务中继续查找这个id为1的记录，如下：

```sql

# 使用READ COMMITTED隔离级别的事务
BEGIN;
 
# SELECT1：Transaction 100、200均未提交
SELECT * FROM t WHERE id = 1; # 得到的列c的值为'刘备'
 
# SELECT2：Transaction 100提交，Transaction 200未提交
SELECT * FROM t WHERE id = 1; # 得到的列c的值为'张飞'
```

这个SELECT2的执行过程如下：

1. 在执行SELECT语句时会先生成一个ReadView，ReadView的m_ids列表的内容就是[200]（事务id为100的那个事务已经提交了，所以生成快照时就没有它了）。
2. 然后从版本链中挑选可见的记录，从图中可以看出，最新版本的列c的内容是'诸葛亮'，该版本的trx_id值为200，在m_ids列表内，所以不符合可见性要求，根据roll_pointer跳到下一个版本。
3. 下一个版本的列c的内容是'赵云'，该版本的trx_id值为200，也在m_ids列表内，所以也不符合要求，继续跳到下一个版本。
4. 下一个版本的列c的内容是'张飞'，该版本的trx_id值为100，比m_ids列表中最小的事务id200还要小，所以这个版本是符合要求的，最后返回给用户的版本就是这条列c为'张飞'的记录。

### REPEATABLE READ ---在第一次读取数据时生成一个ReadView

对于使用REPEATABLE READ隔离级别的事务来说，只会在第一次执行查询语句时生成一个ReadView，之后的查询就不会重复生成了。我们还是用例子看一下是什么效果。

比方说现在系统里有两个id分别为100、200的事务在执行：

```sql

# Transaction 100
BEGIN;
 
UPDATE t SET c = '关羽' WHERE id = 1;
 
UPDATE t SET c = '张飞' WHERE id = 1;
```

```sql

# Transaction 200
BEGIN;
 
# 更新了一些别的表的记录
...
```

此刻，表t中id为1的记录得到的版本链表如下所示：

![image](/images/mysql-transaction/mysql-transaction-12.png)

假设现在有一个使用REPEATABLE READ隔离级别的事务开始执行：

```sql

# 使用REPEATABLE READ隔离级别的事务
BEGIN;
 
# SELECT1：Transaction 100、200未提交
SELECT * FROM t WHERE id = 1; # 得到的列c的值为'刘备'
```

这个SELECT1的执行过程如下：

1. 在执行SELECT语句时会先生成一个ReadView，ReadView的m_ids列表的内容就是[100, 200]。
2. 然后从版本链中挑选可见的记录，从图中可以看出，最新版本的列c的内容是'张飞'，该版本的trx_id值为100，在m_ids列表内，所以不符合可见性要求，根据roll_pointer跳到下一个版本。
3. 下一个版本的列c的内容是'关羽'，该版本的trx_id值也为100，也在m_ids列表内，所以也不符合要求，继续跳到下一个版本。
4. 下一个版本的列c的内容是'刘备'，该版本的trx_id值为80，小于m_ids列表中最小的事务id100，所以这个版本是符合要求的，最后返回给用户的版本就是这条列c为'刘备'的记录。

之后，我们把事务id为100的事务提交一下，就像这样：

```sql
# Transaction 100
BEGIN;
 
UPDATE t SET c = '关羽' WHERE id = 1;
 
UPDATE t SET c = '张飞' WHERE id = 1;
 
COMMIT;
```

然后再到事务id为200的事务中更新一下表t中id为1的记录：

```sql

# Transaction 200
BEGIN;
 
# 更新了一些别的表的记录
...
 
UPDATE t SET c = '赵云' WHERE id = 1;
 
UPDATE t SET c = '诸葛亮' WHERE id = 1;
```

此刻，表t中id为1的记录的版本链就长这样：

![image](/images/mysql-transaction/mysql-transaction-13.png)

然后再到刚才使用REPEATABLE READ隔离级别的事务中继续查找这个id为1的记录，如下：

```sql
# 使用REPEATABLE READ隔离级别的事务
BEGIN;
 
# SELECT1：Transaction 100、200均未提交
SELECT * FROM t WHERE id = 1; # 得到的列c的值为'刘备'
 
# SELECT2：Transaction 100提交，Transaction 200未提交
SELECT * FROM t WHERE id = 1; # 得到的列c的值仍为'刘备'
```

这个SELECT2的执行过程如下：

1. 因为之前已经生成过ReadView了，所以此时直接复用之前的ReadView，之前的ReadView中的m_ids列表就是[100, 200]。
2. 然后从版本链中挑选可见的记录，从图中可以看出，最新版本的列c的内容是'诸葛亮'，该版本的trx_id值为200，在m_ids列表内，所以不符合可见性要求，根据roll_pointer跳到下一个版本。
3. 下一个版本的列c的内容是'赵云'，该版本的trx_id值为200，也在m_ids列表内，所以也不符合要求，继续跳到下一个版本。
4. 下一个版本的列c的内容是'张飞'，该版本的trx_id值为100，而m_ids列表中是包含值为100的事务id的，所以该版本也不符合要求，同理下一个列c的内容是'关羽'的版本也不符合要求。继续跳到下一个版本。
5. 下一个版本的列c的内容是'刘备'，该版本的trx_id值为80，80小于m_ids列表中最小的事务id100，所以这个版本是符合要求的，最后返回给用户的版本就是这条列c为'刘备'的记录。

也就是说两次SELECT查询得到的结果是重复的，记录的列c值都是'刘备'，这就是可重复读的含义。如果我们之后再把事务id为200的记录提交了，之后再到刚才使用REPEATABLE READ隔离级别的事务中继续查找这个id为1的记录，得到的结果还是'刘备'，具体执行过程大家可以自己分析一下。

### MVCC总结

从上边的描述中我们可以看出来，所谓的MVCC（Multi-Version Concurrency Control ，多版本并发控制）指的就是在使用READ COMMITTD、REPEATABLE READ这两种隔离级别的事务在执行普通的SEELCT操作时访问记录的版本链的过程，这样子可以使不同事务的读-写、写-读操作并发执行，从而提升系统性能。READ COMMITTD、REPEATABLE READ这两个隔离级别的一个很大不同就是生成ReadView的时机不同，READ COMMITTD在每一次进行普通SELECT操作前都会生成一个ReadView，而REPEATABLE READ只在第一次进行普通SELECT操作前生成一个ReadView，之后的查询操作都重复这个ReadView就好了。


参考：
1. https://www.cnblogs.com/personsiglewine/p/11507866.html
2. https://blog.csdn.net/qq_38538733/article/details/88902979