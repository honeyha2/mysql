## 并发事务带来的问题
- 读读，无问题
- 写写，提交覆盖、回滚覆盖
- 读写，脏读、不可重复读、幻读

## 写写问题
- MVCC无法解决该问题，MySQL存在提交覆盖问题

## 读写问题
- 标准实现方式：基于锁实现隔离级别
- MySQL实现方式：基于MVCC实现隔离级别，无锁化，提升并发性能

## MVCC实现原理
- 版本链
  - 事务ID（DB_TRX_ID）：每当事务对聚簇索引中的记录进行修改时，都会把当前事务的事务id记录到DB_TRX_ID中。
- ReadView
  - 当一个事务执行时，获取当前数据库中正在活跃的事务id（ACTIVE_TRX_ID_RANGE），形成了ReadView
- 可见性比较算法，版本链中的版本 vs ReadView
  - 首先判断版本记录的DB_TRX_ID字段与生成ReadView的事务对应的事务ID是否相等。如果相等，那就说明该版本的记录是在当前事务中生成的，自然也就能够被当前事务读取；否则进行第2步。
  - 如果版本记录的DB_TRX_ID字段小于范围ACTIVE_TRX_ID_RANGE，表明该版本记录是已提交事务修改的记录，即对当前事务可见；否则进行下一步。
  - 如果版本记录的DB_TRX_ID字段位于范围ACTIVE_TRX_ID_RANGE内，表明是未提交的事务，对当前事务不可见。
- 隔离级别不同，是因为ReadView的生成时机不同
  - 在Read Uncommitted隔离级别下，只需要读取版本链上最新版本的记录即可，即无需执行可见性比较算法
  - 在Read Committed隔离级别下，每次读取数据时都会生成ReadView
  - 在Repeatable Read隔离级别下，只会在事务首次读取数据时生成ReadView，之后的读操作都会沿用此ReadView
  - 在Serializable隔离级别下，无论是读取数据时都会生成ReadView，还是事务首次读取数据时生成ReadView，ReadView内容都是一样的。

## 参考文档
https://juejin.cn/post/6844904096378404872
