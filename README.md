# 数据库基础
## 1. 数据库基础
### 1.1. 关系型数据库

关系型数据库是指采用了关系模型来组织数据的数据库。简单来说，关系模式就是**二维表格模型，类似依据x,y可以定位一个数据**。主要代表：SQL Server，Oracle,Mysql,PostgreSQL,SQLite。

关系型数据库遵循ACID特性（原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）），支持事务处理能力。

适用场景：需要事务支持；基于sql的结构化查询存储，处理复杂的关系。


### 1.2. 非关系型数据库

即NoSQL数据库（not only sql），一般以简单的key-value模式存储，因此大大增加了数据库的扩展能力，不支持ACID，远超于SQL的性能。

适用场景：对数据高并发的读写（mysql数据库存储在磁盘上，高并发会产生大量IO）；对海量数据的读写（mysql数据库不支持海量数据）；对数据高可扩展性的（例如key-value，redis中支持5种类型的value）

### 1.3. 事务
事务是逻辑上的一组操作，要么都执行，要么都不执行。

```
# 开启一个事务
START TRANSACTION;
# 多条SQL语句
SQL1,SQL2...
# 提交事务
COMMIT;
```
![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/2020-12/640-20201207160554677.png)

关系型数据库事务都有ACID特性：

* 原子性（Atomicity） ： 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
* 一致性（Consistency）： 执行事务前后，数据保持一致，例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；
* 隔离性（Isolation）： 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
* 持久性（Durabilily）： 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

并发事务可能存在的问题：
* 脏读（Dirty read）: 当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。
* 丢失修改（Lost to modify）: 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。 例如：事务 1 读取某表中的数据 A=20，事务 2 也读取 A=20，事务 1 修改 A=A-1，事务 2 也修改 A=A-1，最终结果 A=19，事务 1 的修改被丢失。
* 不可重复读（Unrepeatableread）: 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
* 幻读（Phantom read）: 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

SQL 标准定义了四个隔离级别：

* READ-UNCOMMITTED(读取未提交)： 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
* READ-COMMITTED(读取已提交)： 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
* REPEATABLE-READ(可重复读)： 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
* SERIALIZABLE(可串行化)： 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。
MySQL InnoDB 存储引擎的默认支持的隔离级别是 REPEATABLE-READ（可重读）。我们可以通过SELECT @@tx_isolation;命令来查看，MySQL 8.0 该命令改为SELECT @@transaction_isolation;
**在使用分布式事务时，InnoDB 存储引擎的事务隔离级别必须设置为 SERIALIZABLE。**
## 2. MySQL基础

### 2.1. 存储引擎
MySQL 5.5 之前，MyISAM 引擎是 MySQL 的默认存储引擎。虽然，MyISAM 的性能还行，各种特性也还不错（比如全文索引、压缩、空间函数等）。但是，MyISAM 不支持事务和行级锁，而且最大的缺陷就是崩溃后无法安全恢复。

5.5 版本之后，MySQL 引入了 InnoDB（事务性数据库引擎），MySQL 5.5 版本后默认的存储引擎为 InnoDB。

## 3. 数据库的一些实际经验（阿里巴巴）
模糊查询：