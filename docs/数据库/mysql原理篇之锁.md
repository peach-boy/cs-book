# mysql 原理篇之锁

## 一，锁的分类

1. 全局锁
2. 表级锁
   1. [MAL:metadata lock](https://dev.mysql.com/doc/refman/5.7/en/metadata-locking.html)
   2. [表锁](https://dev.mysql.com/doc/refman/5.7/en/lock-tables.html)
3. 行级锁:InnoDB 特有

## 二、innoDB 中有哪些锁

1.  shared (S) locks and exclusive (X) locks：共享锁和独占锁：
    > - 当事务 T1 持有行 r 的 S 锁：事务 T2 同样能持有 R 的 S 锁，但是不能获取 r 行的 X 锁
    > - 当事务 T1 持有行 r 的 X 锁：事务 T2 无法获取 r 行的 X 锁和 S 锁
2.  意向锁：Intention Locks
3.  行锁：Intention Locks
4.  间隙锁：Gap Locks
5.  Next-Key Locks
6.  插入意向锁：Insert Intention Locks
7.  AUTO-INC Locks
8.  Predicate Locks for Spatial Indexes

## 三、死锁

死锁检测： innodb_deadlock_detect
