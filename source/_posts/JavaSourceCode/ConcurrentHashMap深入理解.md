---
title: ConcurrentHashMap深入理解
date: 2021-08-14 09:06:41
tags: 
- Java源码
- ConcurrentHashMap
categories: 
- Java源码
- ConcurrentHashMap

---

<center>
    引言：深入学习ConcurrentHashMap
</center>

<!--more-->

# ConcurrentHashMap

有很多东西与HashMap相同，如果不了解HashMap，建议先去研究[HashMap](https://www.yesmylord.cn/2021/06/13/JavaSourceCode/HashMap%E6%BA%90%E7%A0%81%E6%8E%A2%E7%A9%B6/)

## 并发下的Hash存在的问题

本节介绍为什么要引入ConcurrentHashMap

### 1、多线程下的HashMap存在什么问题？

[总结自敖丙](https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453141934&idx=1&sn=49c4f2cef4d93f45c82886b254517bfb&scene=21#wechat_redirect)

HashMap不能再多线程下使用，在JDK1.7版本，HashMap使用**头插法**，可是在**并发扩容**下容易形成**链表环**，如果此时进行遍历，那么将会导致死锁

在JDK1.8，使用**尾插法**，虽然不会形成链表环，但是并发操作下，由于不是原子操作，如果修改的一瞬间，切换了线程，新的线程完成了正常的修改逻辑，切换回旧线程后，又会把值修改回来。

### 2、多线程下怎样使用哈希表？

有三种方法：

- 使用`Collections.synchronizedMap(Map)`创建线程安全的map集合；
- `Hashtable`
- `ConcurrentHashMap`

### 3、为什么不使用HashTable

​	虽然HashTable也是线程安全的，但是它的内部实现是用`synchronized`，效率太低了

#### 4、为什么不使用`Collections.synchronizedMap`

​	其内部是一个普通的Map＋排斥锁，排斥锁可以自己给，内部也有自己的Obejct作为排斥锁

​	它实现线程安全就是使用`synchronized( mutex)`

## ConcurrentHashMap要点汇总

本文将要点总结在此：

- 为什么设置容量为2的幂次：
  1. 方便计算找下标
  2. 加快扩容时转移数据的速度
- CAS的多次使用，避免加锁

## ConcurrentHashMap基本了解

ConcurrentHashMap也是**数组加链表**，在JDK1.7与1.8的实现稍有不同：

### JDK1.7中的ConcurrentHashMap

实现如下：

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {// 继承了ReentrantLock

    private static final long serialVersionUID = 2249069246763182397L;

    // 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
    transient volatile HashEntry<K,V>[] table;

    transient int count;
    // 记得快速失败（fail—fast）么？
    transient int modCount;
    // 大小
    transient int threshold;
    // 负载因子
    final float loadFactor;
}
```

里面提到了我们熟悉的概念负载因子、table数组等等；还有我们不熟悉的modCount

> **快速失败 fail fast 机制**：
>
> ​		是Java集合中的一种机制， 在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出`Concurrent Modification Exception`。

原理就是使用一个 `modCount`，在每次修改的时候改变`modCount`值，集合遍历期间，发现`modCount`值发生改变，就将抛出异常（原理与解决ABA问题的原理相同）

与快速失败机制相关的还有一个 fail safe：

> **安全失败机制（fail-safe）**，这种机制会使你此次读到的数据不一定是最新的数据。

----

扯远了，再回到主角上来

`ConcurrentHashMap`实现并发的机制就是 **table数组`HashEntry`使用了`volatile`关键字**

### 并发度高的原因

使用`Segment`，每个线程分别占用一个`Segment`，互不影响，默认的并发度为16，也即默认初始有16个`Segment`

算是一种空间换时间的策略

### `put`与`get`方法高并发的原因

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    	//这就是为啥他不可以put null值的原因
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask;
    // 定位到对应的segment再进行put操作
    if ((s = (Segment<K,V>)UNSAFE.getObject          
         (segments, (j << SSHIFT) + SBASE)) == null) 
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}
```

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry
    HashEntry<K,V> node = tryLock() ? null :
    scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
                // 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                // 不为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容。
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        //释放锁
        unlock();
    }
    return oldValue;
}
```

put高并发总结：

1. 首先定位到对应的`segment`再进行hash操作（**分段锁**）
2. 判断当前put是否被占用
   - 如果被占用：**自旋锁**，直到获取或达到`MAX_SCAN_RETRIES`，改为使用**阻塞锁**

`get`方法更加简单，直接定位到`segment`再一步定位元素即可（因为有`volatile`保证可见性）

### JDk1.8中的ConcurrentHashMap

以上的讨论，都是在JDK1.7时，这个版本HashMap有的问题它也都有

（比如说链表环导致死锁）

一句话：**JDK1.8抛弃了原有的 Segment 分段锁，而采用了 `CAS + synchronized` 来保证并发安全性。**（数据结构方面也加入了红黑树）

## 各数据的来源依据

```java
private static final int DEFAULT_CAPACITY = 16;
// 初始容量，在HashMap一节也说过，这个容量设置必须是2的幂次
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
// 最大数组大小，为什么要减去8？是因为Array在JVM中还有一些元数据信息要进行存储
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
// 默认的并发级别 是16
private static final float LOAD_FACTOR = 0.75f;
// 负载因子 0.75
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
// 树化和退化的临界阈值
static final int MIN_TREEIFY_CAPACITY = 64;
// 树化的另一个条件，数组的容量得达到64以上
```

**为什么容量必须是2的幂次？**

​		Hash运算大致为 k % m ，显然m如果是一个素数会让整个Hash散列的更加均匀。但是java中选择了幂数，是因为`%`的效率远远低于位运算的效率

假设一个数`1234`，这个数的二进制是`0100 1101 0010`

假设当前容量为`16`，`1234%16=2`

如果我们直接使用这个二进制数的后四位`0010`与16-1的二进制`1111`进行位与运算，结果`0010 & 1111 = 0010`，这个结果也是2

完全避免了使用`%`，在Java中也是如此实现的：

```java
k & (n-1) // n为长度
```

如果容量为2的幂次，那么容量-1，这个数的二进制为全1，这样就可以**加快寻找`index`的速度**

除此之外，**设置为2的幂次还可以加快扩容时转移数据的速度**，下文将进行介绍

**为什么是初始值为16？**

​		统计分析的结果，使用过程中16作为初始容量最为合适，不会太大也不会太小。

**我使用构造方法`new ConcurrentHashMap<>(4)`，它的容量大小会是多少？**

在JDK源码中：会发现我们给了4他也会进行一些计算

```java
public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}
```

`tableSizeFor`方法会返回一个大于输入值的最小的2的幂次数

如果是4的话，传入`tableSizeFor`方法的参数就是7，那么`sizeCtl`的值会是 8

所以要知道，**我们给的容量并不是实际的容量**

## 初始化&sizeCtl变量

和HashMap一样，ConcurrentHashMap的初始化方法并不在构造方法内，而是放在了第一次`put`方法中

```java
if (tab == null || (n = tab.length) == 0)
    tab = initTable();
```

我们主要来研究`initTable`这个方法，再看源码之前，我们必须得有些前置了解：

1、首先注意sizeCtl这个值代表的意思：

```java
private transient volatile int sizeCtl;
//表初始化和调整控制。
// 值为-1 表示进行初始化
// 值为-n 表示正在扩容
// 值为整数 表示当前数组要扩容时的阈值（比如容量为16，此值就为16*0.75=12）
```

2、在内部还会有`SIZECTL`这个变量

```java
// Unsafe mechanics
private static final sun.misc.Unsafe U;
private static final long SIZECTL;
...
static {
        try {
            U = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentHashMap.class;
            SIZECTL = U.objectFieldOffset
                (k.getDeclaredField("sizeCtl"));
            // 直接从内存中获取输入参数的（即获取内存中sizeCtl）地址偏移量
        }
    ...
}
```

再次强调，**`SIZECTL`是`sizeCtl`在内存中的地址偏移量**

3、CAS操作：

3、**CAS（Compare And Swap/Set）：比较并变换**，是一个**原子**操作，**相同则更新**

> CAS(V,E,N)
>
> V 表示要更新的变量（内存值）
>
> E 表示预期值（旧的）
>
> N 表示新值

1. 如果 `V==E` 值时，会将 `V=N`（内存值 == 预期值，**说明没有线程对当前变量进行写操作**）
2. 如果 `V!=E`，则当前线程什么都不做（内存值！= 预期值，说明已经有其他线程做了更新，那么现在就不能更改这个值）
3. 最后，CAS 操作返回当前 V 的真实值

`initTable`源码如下：

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            // 小于0 代表有线程正在进行初始化或是扩容
          	// 如果此时其他并发线程进入，会直接yield
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            // CAS操作！！this与SIZECTL可以确定内存地址，如果此地址的值与sc相同，说明当前没有线程进行写操作，会将-1赋值给SIZECTL
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];// 此处终于创建了数组
                    table = tab = nt;
                    sc = n - (n >>> 2);
                    // 此处相当于 n - (n / 4) = 0.75 * n
                    // 相当于乘了负载因子
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

## `put()`方法

put方法很长，我们将分解的进行对`put()`方法的探究：

```java
public V put(K key, V value) {
    return putVal(key, value, false);
    // put调用了另一个方法
}
```

1、`spread`方法：

​		因为`>>>`无符号右移就是**把低位去掉保留高位**。然后高位和低位进行`^`位运算。这样不管是高位发生变化，还是低位发生变化都会造成其结果的**中低位**发生变化。

为什么我们关注其结果的**中低位**呢，那是因为后面算`index`的时候，用了`h & (length-1)`，它的意思就是把高位去掉。
如果没有进行扰动计算，当`key`仅仅发生高位变动的时候就会发生hash冲突，这对Hashmap来说往往是致命的

​		简而言之，就是**为了增大随机性，减少哈希碰撞的次数**

```java
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
    // HASH_BITS 是各位全为1的值
}
```

2、`tabAt`方法

​		`tabAt(tab, i = (n - 1) & hash)`在源码中如此使用，意思很明确

`(n - 1) & hash`代表下标，此函数的意思就是返回`table[index]`的值



下面来正式看代码

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    // spread 扰动函数
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        // 一个死循环
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();// 上文介绍了initTable方法
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {// 如果此位置为null，说明此处是一个空桶，即没有哈希碰撞过
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                //使用cas给此位置赋值
                break;                   // no lock when adding to empty bin源码的注释也很明确，空桶不用加锁（使用CAS思想）
        }
        else if ((fh = f.hash) == MOVED)
            ...
        else {
            ...
        }
    }
    addCount(1L, binCount);// 扩容方法
    return null;
}
```

## `addCount()`方法

`addCount`中其实并没有进行扩容，而是主要进行了扩容的判断（与扩容阈值`sizeCtl`进行判断）

```java
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
      	// 上面这一堆我们只需关注 s 即可 ，每次都会给s进行增加 s表示当前的size，在下面会对其进行判断
        ...
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        // 注意下面这个判断条件，用s和sizeCtl比较大小，这是扩容的时机
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);// 标识当前数组
            if (sc < 0) {
                ...
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            // transfer方法：正式的开始扩容！
            s = sumCount();
        }
    }
}
```

## `transfer()`方法

我们知道，在Java中没有动态数组，所谓的动态数组，都是在达到一定程度，进行了几步操作，才实现的

那么扩容需要几步？

1. 开辟一块新空间
2. 将旧数组的数据转移到新数组

而其中第二步，转移操作，需要**寻找下标**，原本计算下标的方式是`k&(n-1)`，这里是否还需要这样做呢？

这里Java的操作十分巧妙

```text
假设有数据为：
1110 0011 
//原数据与 16-1 进行与运算，即末尾4位
现在扩容到了32位，就是要与末尾5位进行比较
但是我们不需要再这样操作了
只需要看倒数第五位即可！
如果倒数第五位为0：说明与原数组下标相同
如果为1：只需要将0001 0000加上原数组位置即可
```

其次，为了加快迁移速度，还可以使用多线程，使用多线程不能让数据混乱，因此：

​		在ConcurrentHashMap中，会将原数组进行**分段**，**最少每一个线程完成16个桶的迁移工作**，如果不足16个桶，那么就由一个线程来进行迁移

`transfer`就完成了这两步操作：

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    // 下面的判断是为了使用多线程进行数据迁移；NCPU默认为8，与电脑有关
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range 切分数组进行数据迁移 
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];// 这里，直接扩容了两倍 <<1 代表乘以2
            nextTab = nt;
        }
        ...
    }
    ...
    else {
        synchronized (f) {
            if (tabAt(tab, i) == f) {
                // 此处是正式的转移操作：分为链表的与红黑树两部分
                Node<K,V> ln, hn;
                if (fh >= 0) {
                    ...
                    // 这里的转移方法特别巧妙，就是上面我说明的方法
					// 上面省略的部分会将为0的与为1的分为两个链表然后分别进行操作
                    setTabAt(nextTab, i, ln);//ln代表0对应的链表
                    setTabAt(nextTab, i + n, hn);//hn代表1对应的链表
                    setTabAt(tab, i, fwd);
                    advance = true;
                }
                ...
            }
        }
        ...             
}
```



## 相关链接

1. [HashMap源码探究](https://www.yesmylord.cn/2021/06/13/JavaSourceCode/HashMap%E6%BA%90%E7%A0%81%E6%8E%A2%E7%A9%B6/)