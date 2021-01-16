# 原理篇之CAS

目录
- [1 什么时CAS](#1-什么是CAS)
- [2 unsafe类](#2-unsafe类)
- [3 Atomic包](#3-Atomic包)
- [4 ABA问题](#4-ABA问题)


## 1. 什么是CAS
cas是一种乐观锁的实现方式，在juc包中有大量的应用，如：AtomicInteger

```
private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();

public final int incrementAndGet() {
        return U.getAndAddInt(this, VALUE, 1) + 1;
}
```
其中我们可以看到最终的cas操作是交给unsafe来实现的。

## 2. unsafe类

## 3. Atomic包

## 4. ABA问题

