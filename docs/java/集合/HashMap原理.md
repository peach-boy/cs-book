# HashMap 原理

## 一、底层数据结构

- java1.7 之前版本：哈希表+链表。
- java1.8 及之后版本：哈希表+链表/红黑树。

首先我们先了解一下上面提到的三种数据结构的特性，为什么 hashmap 会用到这三种数据结构。

- 哈希表：等值查询的时间复杂度为 O(1)，天然适合 kv 结构存储。
- 链表：查询的时间复杂度为 O(n)，查询低效：插入和删除容易。
- 红黑树：查询的时间复杂度为 O(logn)，增删查改介于数组和链表之间，但是维护复杂。

## 二、增删改查如何实现

下面是插入数据的整个流程：

1. 第一步：对 key 做 hash 计算

```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

2. 第二步：确定 hash 槽

```java
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //1.(n - 1) & hash]讲key的hash值与n-1做与运算得到hash槽的位置
        if ((p = tab[i = (n - 1) & hash]) == null) {
            //2.判断如果当前hash槽为null，则直接插入
            tab[i] = newNode(hash, key, value, null);
        }
        else {
            Node<K,V> e; K k;
            //判断key是否已存在，存在则将新值p赋值给结果e
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果hash槽的节点是树节点，则当前数据结构为红黑树，则直接插入
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
            //次时数据结构则为链表 ，遍历链表，讲新节点插入到链表尾部
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //如果链表的长度>=8,将链表转换成红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
}

```

- 第一步：调用 put 方法传入键值对
- 第二步：使用调用 hashCode()计算 hash 值
- 第三步：根据 hash 值确定存放的位置，判断是否和其他键值对位置发生了冲突
- 第四步：若没有发生冲突，直接存放在数组中即可
- 第五步：若发生了冲突，还要判断此时的数据结构是什么？
- 第六步：若此时的数据结构是红黑树，那就直接插入红黑树中
- 第七步：若此时的数据结构是链表，判断插入之后是否大于等于 8
- 第八步：插入之后大于 8 了，就要先调整为红黑树，在插入
- 第九步：插入之后不大于 8，那么就直接插入到链表尾部即可。

## 三、如何扩容

为什么扩容呢？当向容器添加元素的时候，会判断当前容器的元素个数，如果大于等于阈值—即当前数组的长度乘以加载因子的值的时候，就要自动扩容。

这里需要有两个名字介绍一下：

- initialCapacity 初始容量
  > 即上文中的 n ,官方要求我们要输入一个 2 的 N 次幂的值，比如说 2、4、8、16 等等这些，但是我们忽然一个不小心，输入了一个 20 怎么办？没关系，虚拟机会根据你输入的值，找一个离 20 最近的 2 的 N 次幂的值，比如说 16 离他最近，就取 16 为初始容量,取 2 的那次幂主要是为了方便取模
- loadFactor 负载因子：
  > 负载因子，默认值是 0.75。负载因子表示一个散列表的空间的使用程度，有这样一个公式：initailCapacity\*loadFactor=HashMap 的容量。 所以负载因子越大则散列表的装填程度越高，也就是能容纳更多的元素，元素多了，链表大了，所以此时索引效率就会降低。反之，负载因子越小则链表中的数据量就越稀疏，此时会对空间造成烂费，但是此时索引效率高

## 四、为什么非线程安全

- put 的时候导致的多线程数据不一致：这个问题比较好想象，比如有两个线程 A 和 B，首先 A 希望插入一个 key-value 对到 HashMap 中，首先计算记录所要落到的桶的索引坐标，然后获取到该桶里面的链表头结点，此时线程 A 的时间片用完了，而此时线程 B 被调度得以执行，和线程 A 一样执行，只不过线程 B 成功将记录插到了桶里面，假设线程 A 插入的记录计算出来的桶索引和线程 B 要插入的记录计算出来的桶索引是一样的，那么当线程 B 成功插入之后，线程 A 再次被调度运行时，它依然持有过期的链表头但是它对此一无所知，以至于它认为它应该这样做，如此一来就覆盖了线程 B 插入的记录，这样线程 B 插入的记录就凭空消失了，造成了数据不一致的行为
- 多线程环境下有出现死循环的可能，参考[Hashmap 死循环]（https://blog.csdn.net/Kevin_King1992/article/details/79859290）

## 五、常见问题

1. 如何定位哈希槽
   > 通过 hashCode 计算出 key 的 hash 值之后，通过（n-1）&hash 运算得到存放在数组中的位置，n 表示 hashmap 的初始容量，即数组的初始长度，将计算出的 hash 值与 n-1 做与运算得出存在的数组下标。
2. 初始容量为什么是 16，为什么 HashMap 的底层数组长度为何总是 2 的 n 次方
   > 为什么要通过（n-1）&hash 运算确定数组下标：当计算出 key 的 hash 值时，
   > 我们需要将 key 均匀的分布到数组上，也就是需要为每一个 key 值给定一个数组下标，这个下标需要在> 0 到 n-1 之间，首先我们想到的是取模运算，Java 之所有使用位运算(&)来代替取模运算(%)，最主要的考虑就是效率。位运算(&)效率要比代替取模运算(%)高很多，主要原因是位运算直接对内存数据进行操作，不需要转成十进制，因此处理速度非常快。初始容量 n 设置为 2 的 n 次幂，是为了便于做与运算，2 的 n 次幂-1 的到二进制数全为 1,如 7 的二进制表示为 111，任何整数与它做与运算得到的值都会在 0-7 这个区区间，至于为什么是 16,我想应该是经验值，至今没有找到官方解释。
3. 为什么是红黑树而不是 AVL(平衡二叉树)树
   > 见[AVL 树和红黑树的区别](https://blog.csdn.net/u010899985/article/details/80981053)
4. 为什么链表长度达到 8 时转为红黑树：
   > 在使用分布良好的用户的 hashCodes 时，很少使用红黑树结构，因为在理想情况下，链表的节点频率遵循泊松分布（意思就是链表各个长度的命中率依次递减），当命中第 8 次的时候， 链表的长度是 8，此时的概率仅为 0.00000006，小于千万分之一。
5. 为什步开始就直接用红黑树：
   > 因为 TreeNodes 的大小大约是普通 Node 节点的两倍， 所以只有在节点足够多的情况下才会把 Nodes 节点转换成 TreeNodes 节点， 是否足够多又由 TREEIFY_THRESHOLD 决定，而当桶中的节点的数量由于移除或者调整大小变少后， 它们又会被转换回普通的链表结构以节省空间。
   > 通过源码中得知，当链表长度达到 8 就转成红黑树结构，当树节点小于等于 6 时就转换回去，此处体现了时间和空间的平衡思想。
6. 为什么负载因子是 0.75
