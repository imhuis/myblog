---
layout: post
title:  "Mysql Database"
date:   2023-01-01 23:59:59 +0800
categories: database dba
---

## 数据类型

### 整型

```sql
create table test(
	a tinyint unsigned,
    b int(6) unsigned zerofill
)engine=innodb
```

> int(N) 无论N是多少，int永远只占四个字节，N表示宽度，设置zerofill后不足的地方0补位

| **数据类型**  | **字节数** | **带符号最小值**            | **带符号最大值**          | **不带符号最小值** | **不带符号最大值**          |
| --------- | ------- | --------------------- | ------------------- | ----------- | -------------------- |
| TINYINT   | 1       | \-128                 | 127                 | 0           | 255                  |
| SMALLINT  | 2       | \-32768               | 32767               | 0           | 65535                |
| MEDIUMINT | 3       | \-8388608             | 8388607             | 0           | 16777215             |
| INT       | 4       | \-2147483648          | 2147483647          | 0           | 4294967295           |
| BIGINT    | 8       | \-9223372036854775808 | 9223372036854775807 | 0           | 18446744073709551616 |

### 浮点型

| **数据类型** | **字节数** |        |
| -------- | ------- | ------ |
| float    | 4       | 单精度浮点型 |
| double   | 8       | 双精度浮点型 |

> float(M,N) M总位数，N精度

> MySQL的浮点数精度只有53bit，大于53bit会被‘四舍五入’

### 定点型

`decimal(M,N)` 以字符串形式保存

### 日期类型

| **数据类型**  | **字节数** | **格式**              |         |
| --------- | ------- | ------------------- | ------- |
| date      | 3       | yyyy-MM-dd          | 存储日期值   |
| time      | 3       | HH:mm:ss            | 存储时分秒   |
| year      | 1       | yyyy                | 年       |
| datetime  | 8       | yyyy-MM-dd HH:mm:ss | 存储日期+时间 |
| timestamp | 4       | yyyy-MM-dd HH:mm:ss | 时间戳     |

### 字符型

> MySQL要求一个行的定义长度不能超过65535即64K (65535bytes) 1KB = 1024 Bytes

> utf8编码下varchar(M) M=21844，utf8mb4 M=16383，gbk编码下varchar(M)最大的M=32765

varchar(M)，当M范围为0<=M<=255时会专门有一个字节记录varchar型字符串长度，当M>255时会专门有两个字节记录varchar型字符串的长度，把这一点和上一点结合，那么65535个字节实际可用的为65535-3=65532个字节

char是固定长度字符串，其长度范围为`0~255`且与编码方式无关，无论字符实际长度是多少，都会按照指定长度存储，不够的用空格补足；varchar为可变长度字符串，在utf8编码的数据库中其长度范围为`0~21844`

`char` 处理数据时会将结尾的空格删除，`varchar` 不会

当varchar(M)的M大于某些数值时，varchar会自动转为text

- M>255时转为tinytext
- M>500时转为text
- M>20000时转为mediumtext

## 事务

事务就是要保证一组数据库操作，要么全部成功，要么全部失败。在 MySQL 中，事务支持是在引擎层实现的。MySQL 是一个支持多引擎的系统，但并不是所有的引擎都支持事务。**原生的 MyISAM 引擎不支持事务**。

ACID（Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性）

### 隔离性（Isolation）与隔离级别

SQL标准的事务隔离级别（隔离级别越高，效率越低）

| **隔离级别**               |                |                            |                |
| ---------------------- | -------------- | -------------------------- | -------------- |
| 读未提交（read uncommitted） |                | 脏读（dirty read）             | 读取到未提交的事务      |
| 读提交（read committed）    | `Oracle默认隔离级别` | 不可重复读（non-repeatable read） | 数据前后不一致        |
| 可重复读（repeatable read）  | `mySQL默认隔离级别`  | 幻读 （phantom read）          | 多读取到数据，前后数量不一致 |
| 串行化（serializable ）     |                |                            |                |

```sql
show variables like 'transaction_isolation'; // 查看默认的隔离级别
```

### 事务的启动方式

1. 显式启动事务语句， `begin` 或 `start transaction`。配套的提交语句是 `commit`，回滚语句是 `rollback`
2. `set autocommit = 0`，这个命令会将这个线程的自动提交关掉。意味着如果你只执行一个 select 语句，这个事务就启动了，而且不会自动提交。这个事务持续存在直到你主动执行 commit 或 rollback 语句，或者断开连接。（会导致意外的长连接）推荐 `set autocommit = 1`
3. commit work and chain （提交事务并自动启动下一个事务，这样也省去了再次执行 begin 语句的开销）

begin/start transaction 命令不是一个事务的起点，在执行到它们之后的第一个操作 InnoDB 表的语句，事务才真正启动。如果你想要马上启动一个事务，使用 `start transaction with consistent snapshot`

> 为什么不推荐长事务，长事务意味着系统里面会存在很老的事务视图。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间。

> MySQL 5.5 及以前的版本，回滚日志是跟数据字典一起放在 ibdata 文件里的，即使长事务最终提交，回滚段被清理，文件也不会变小。我见过数据只有 20GB，而回滚段有 200GB 的库。最终只好为了清理回滚段，重建整个库。

长事务的影响：长事务占用锁资源，也可能拖垮整个库

### 事务隔离实现

`MVCC` 多版本并发控制，

![882114aaf55861832b4270d44507695e.jpeg](https://res.craft.do/user/full/7ee1bdd7-e066-d2b1-d95d-0d39ee51c2fd/E802BF94-5823-4504-A696-1CD8BEBCAD8A_2/OkVmPJe1zT28mTxNxuoSDxyef7vOesF4Qoytr0SA31Qz/882114aaf55861832b4270d44507695e.jpeg)

一个数据版本，对于一个事务视图来说，除了自己的更新总是可见以外，有三种情况：

- 版本未提交，不可见；
- 版本已提交，但是是在视图创建后提交的，不可见；
- 版本已提交，而且是在视图创建前提交的，可见。

更新数据都是先读后写的，而这个读，只能读当前的值，称为“当前读” (current read)

除了 update 语句外，select 语句如果加锁也是当前读。

```sql
-- 读锁（S 锁，共享锁）
select ... lock in share mode;
-- 写锁（X 锁，排他锁）
select ... for update;
```

InnoDB 的行数据有多个版本，每个数据版本有自己的 row trx_id，每个事务或者语句有自己的一致性视图。普通查询语句是一致性读，一致性读会根据 row trx_id 和一致性视图确定数据版本的可见性。

- 对于可重复读，查询只承认在事务启动前就已经提交完成的数据；
- 对于读提交，查询只承认在语句启动前就已经提交完成的数据；
- 而当前读，总是读取已经提交完成的最新版本。

你也可以想一下，为什么表结构不支持“可重复读”？这是因为表结构没有对应的行数据，也没有 row trx_id，因此只能遵循当前读的逻辑。当然，MySQL 8.0 已经可以把表结构放在 InnoDB 字典里了，也许以后会支持表结构的可重复读。

## 索引

### InnoDB索引模型

索引分为主键索引和非主键索引

**主键索引** 的叶子节点存的是整行数据。在 InnoDB 里，主键索引也被称为聚簇索引（clustered index）

**非主键索引** 的叶子节点内容是主键的值。在 InnoDB 里，非主键索引也被称为二级索引（secondary index）

主键索引和非主键索引的区别在于，基于非主键索引的查询需要多扫描一棵索引树，这个过程称为**回表**

redo log 主要节省的是随机写磁盘的 IO 消耗（转成顺序写），而 change buffer 主要节省的则是随机读磁盘的 IO 消耗

#### 索引维护

> 自增主键不会触发叶子节点的分裂，非主键索引的叶子节点是主键的值，主键占用字节越大，普通索引占用空间也就越大。

> 优先使用自增字段做主键

`NOT NULL PRIMARY KEY AUTO_INCREMENT`

### 操作索引

```sql
alter table T drop index k;
alter table T add index(k);
// 重建主键索引不合理，会导致其他索引失效
alter table T engine=InnoDB;
```

### 索引优化

- 覆盖索引
- 索引下推（MySQL > 5.6）

### 索引失效场景

见**MySQL索引**

[MySQL 索引](craftdocs://open?blockId=8069DA28-C44D-40DB-A059-CA1576C42EA3&spaceId=7ee1bdd7-e066-d2b1-d95d-0d39ee51c2fd)

## MySQL中的锁

### 全局锁

对整个数据库实例加锁，命令 `Flush tables with read lock (FTWRL)` ，DML以及更新语句、更新类事务提交语句会被阻塞。

使用场景：全库逻辑备份（主从架构下，在主库做业务停摆，在从库做会造成主从延迟）

```other
mysqldump –single-transaction
set global readonly=true // 方式二
```

### 表级锁

表锁，元数据锁（meta data lock，MDL)

`lock tables … read/write` unlock tables 主动释放锁，也可以在客户端断开的时候自动释放。需要注意，lock tables 语法除了会限制别的线程的读写外，也限定了本线程接下来的操作对象。

MySQL 5.5 版本中引入了 MDL，当对一个表做增删改查操作的时候，加 MDL 读锁；当要对表做结构变更操作的时候，加 MDL 写锁。

> 如何安全的给表加字段？

- > 查询 `information_schema.innodb_trx` 中的长事务，kill
- > `NOWAIT/WAIT n` 语法

### 行锁

在 InnoDB 事务中，行锁是在需要的时候才加上的，但并不是不需要了就立刻释放，而是要等到事务结束时才释放。这个就是两阶段锁协议

死锁处理策略

- `innodb_lock_wait_timeout`
- 死锁检测，`innodb_deadlock_detect = on` ，发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行

死锁解决方法：调整语句顺序，控制客户端并发量

[MySQL 锁](craftdocs://open?blockId=8427EF96-E00D-4341-9A11-1B91E246CC47&spaceId=7ee1bdd7-e066-d2b1-d95d-0d39ee51c2fd)

## 优化

[SQL分析及数据库优化](craftdocs://open?blockId=37433110-FE31-4BC1-A3E3-4EABE1AC2B18&spaceId=7ee1bdd7-e066-d2b1-d95d-0d39ee51c2fd)

### 附录

#### MySQL各版本新特性

| 5.6之前 | MDL（5.5）        |
| ----- | --------------- |
| 5.6   | Online DDL并行复制 |
| 5.7   |                 |
| 8     |                 |

