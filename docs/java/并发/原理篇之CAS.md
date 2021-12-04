# 原理篇之 CAS

目录

- [1 什么时 CAS](#1-什么是CAS)
- [2 unsafe 类](#2-unsafe类)
- [3 Atomic 包](#3-Atomic包)
- [4 ABA 问题](#4-ABA问题)

## 1. 什么是 CAS

cas 是一种乐观锁的实现方式，在 juc 包中有大量的应用，如：AtomicInteger

```java
private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();

public final int incrementAndGet() {
        return U.getAndAddInt(this, VALUE, 1) + 1;
}
```

其中我们可以看到最终的 cas 操作是交给 unsafe 来实现的。

## 2. unsafe 类

## 3. Atomic 包

## 4. ABA 问题
