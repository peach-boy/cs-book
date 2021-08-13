# mysql原理(二)锁

## 一，锁的分类
1. 全局锁
2. 表级锁
    1. [MAL:metadata lock](https://dev.mysql.com/doc/refman/5.7/en/metadata-locking.html )
    2. [表锁](https://dev.mysql.com/doc/refman/5.7/en/lock-tables.html)
2. 行级锁:InnoDB特有

## 二、innoDB中有哪些锁

 1. shared (S) locks and exclusive (X) locks：共享锁和独占锁：
> - 当事务T1持有行r的S锁：事务T2同样能持有R的S锁，但是不能获取r行的X锁
> - 当事务T1持有行r的X锁：事务T2无法获取r行的X锁和S锁
 2. 意向锁：Intention Locks
 3. 行锁：Intention Locks
 4. 间隙锁：Gap Locks
 5. Next-Key Locks
 6. 插入意向锁：Insert Intention Locks
 7. AUTO-INC Locks
 8. Predicate Locks for Spatial Indexes


