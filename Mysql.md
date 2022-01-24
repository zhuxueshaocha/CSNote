# Mysql存储引擎

## InnoDB和MyISAM对比

InnoDB支持事务，可以进行Commit和Rollback；

MyISAM 只支持表级锁，而 InnoDB 还支持行级锁，提高了并发操作的性能；

InnoDB 支持外键；

MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢；

MyISAM 支持压缩表和空间数据索引，InnoDB需要更多的内存和存储；

InnoDB 支持在线热备份

表结构不同： InnoDB 将表结构存储在 .frm 文件中，数据和索引存储在 .idb 文件中。 每个 MyISAM 表格会保存在磁盘的三个文件中，文件名就是表名：.frm：存储表结构.MYD（MYData）：存储数据.MYI（MYIndex）：存储索引

热备份和冷备份

热备份：在数据库运行的情况下备份的方法。优点：可按表或用户备份，备份时数据库仍可使用，可恢复至任一时间点。但是不能出错

冷备份：数据库正常关闭后，将关键性文件复制到另一位置的备份方式。优点：操作简单快速，恢复简单

应用场景

MyISAM 管理非事务表。它提供高速存储和检索（MyISAM强调的是性能，每次查询具有原子性，其执行速度比InnoDB更快），以及全文搜索能力。如果表比较小，或者是只读数据（有大量的SELECT），还是可以使用MyISAM；

InnoDB 支持事务，并发情况下有很好的性能，基本可以替代MyISAM

## 执行一条sql的过程

<img width="513" alt="截屏2022-01-24 下午6 16 39" src="https://user-images.githubusercontent.com/98211272/150764380-28342de2-596f-43bc-a0ed-048ae05fcf88.png">

mysql8.0版本之后需要了查询缓存的功能，因为数据失效太过频繁。

## mysql的日志系统

### bin log

binlog 用于记录数据库执行的写入性操作(不包括查询)信息，以二进制的形式保存在磁盘中。binlog 是 mysql的逻辑日志，并且由 Server 层进行记录，使用任何存储引擎的 mysql 数据库都会记录 binlog 日志。

逻辑日志：可以简单理解为记录的就是sql语句 。

物理日志：mysql 数据最终是保存在数据页中的，物理日志记录的就是数据页变更 。

binlog 是通过追加的方式进行写入的，可以通过max_binlog_size 参数设置每个 binlog文件的大小，当文件大小达到给定值之后，会生成新的文件来保存日志。

binlog 的主要使用场景有两个，分别是 主从复制 和 数据恢复 。

主从复制 ：在 Master 端开启 binlog ，然后将 binlog发送到各个 Slave 端， Slave 端重放 binlog 从而达到主从数据一致。

数据恢复 ：通过使用 mysqlbinlog 工具来恢复数据。

binlog刷盘时机对于 InnoDB 存储引擎而言，只有在事务提交时才会记录biglog ，此时记录还在内存中，那么 biglog是什么时候刷到磁盘中的呢？

mysql 通过 sync_binlog 参数控制 biglog 的刷盘时机，取值范围是 0-N：

0：不去强制要求，由系统自行判断何时写入磁盘；

1：每次 commit 的时候都要将 binlog 写入磁盘；

N：每N个事务，才会将 binlog 写入磁盘。

从上面可以看出， sync_binlog 最安全的是设置是 1 ，这也是MySQL 5.7.7之后版本的默认值。但是设置一个大一些的值可以提升数据库性能，因此实际情况下也可以将值适当调大，牺牲一定的一致性来获取更好的性能。

### redo log

我们都知道，事务的四大特性里面有一个是 持久性 ，具体来说就是只要事务提交成功，那么对数据库做的修改就被永久保存下来了，不可能因为任何原因再回到原来的状态 。redo log是保证事务的持久性的

redo log 包括两部分：一个是内存中的日志缓冲( redo log buffer )，另一个是磁盘上的日志文件( redo logfile)。mysql 每执行一条 DML 语句，先将记录写入 redo log buffer，后续某个时间点再一次性将多个操作记录写到 redo log file。

这种先写日志，再写磁盘 的技术就是 MySQL里经常说到的 WAL(Write-Ahead Logging) 技术。

redo log 实际上记录数据页的变更，而这种变更记录是没必要全部保存，因此 redo log实现上采用了大小固定，循环写入的方式，当写到结尾时，会回到开头循环写日志。

#### redo log与bin log区别

由 binlog 和 redo log 的区别可知：binlog 日志只用于归档，只依靠 binlog 是没有 crash-safe 能力的。但只有 redo log 也不行，因为 redo log 是 InnoDB特有的，且日志上的记录落盘后会被覆盖掉。因此需要 binlog和 redo log二者同时记录，才能保证当数据库发生宕机重启时，数据不会丢失。

### undo log

用来保证事务的原子性，undo log主要记录了数据的逻辑变化，比如一条 INSERT 语句，对应一条DELETE 的 undo log ，对于每个 UPDATE 语句，对应一条相反的 UPDATE 的 undo log ，这样在发生错误时，就能回滚到事务之前的数据状态。
同时，undo log 也是 MVCC(多版本并发控制)实现的关键。






