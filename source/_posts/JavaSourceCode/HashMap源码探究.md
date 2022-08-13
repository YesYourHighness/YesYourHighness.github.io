---
title: HashMap源码探究
date: 2021-06-13 09:30:41
tags: 
- Java源码
- HashMap
categories: 
- Java源码
- HashMap
---

<center>
    引言：HashMap源码探究
</center>
<!--more-->

# HashMap源码探究

> 写在前面：
>
> ​		HashMap是一个重要的内容，无论是日常使用还是面试，都会用到；
>
> ​		个人水平不足以完成这个源码探究，但是找到了很多大佬的博客，深有感触，所以决定洗稿开始，加上自己的感受，如果有错误纯属正常操作，请指出。

## 参考来源

1. [猿人谷](https://yuanrengu.com/2020/ba184259.html)：大佬，这篇blog直接路转粉
2. [敖丙](https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453141934&idx=1&sn=49c4f2cef4d93f45c82886b254517bfb&scene=21#wechat_redirect)：并发下的HashMap如何出现问题

## HashMap要点

- HashMap的底层实现：Jdk1.7中使用**数组+链表**；Jdk1.8使用**数组+链表+红黑树**

- 红黑树化条件：（两个缺一不可）

  - 数组的长度大于**64**（如果不大于64，使用扩容策略）
  - 链表的长度大于**8**

- 红黑树节点在**6**以下会恢复为链表

- HashMap的初始化方法仅仅是设置扩容临界值、负载因子、初始容量，真正的初始化Node在`resize`方法中

- resize方法负责两个工作：

  - 初始化`node`数组`table`

  - 扩容

- HashMap是线程不安全的，如果在多线程中，请使用`ConcurrentHashMap`

- 负载因子默认值0.75：

  - 负载因子大：提高内存利用率，但是碰撞几率变大
  - 负载因子小：提高查询速度，浪费内存空间

- 扩容一定是**2的幂次**，成倍的进行扩容

- JDK1.7是**表头插入法**，每次扩容改变存储顺序，易造成列表环

- JDK1.8是**尾部插入法**，扩容不改变存储顺序

## HashMap底层实现

### 基本field

```java
/**
     * 初始化容量大小16
     * << 表示左移位，每移动一位相当于*2
     */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
     * 最大容量：2^30
     */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
     * 负载因子：0.75
     */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
     * 使用树而不是列表的临界点（threshold）！
     * hash表中出现哈希碰撞后，当链表长度达到8的时候，才会考虑转换为红黑树（注意是才会考虑！）
     */
static final int TREEIFY_THRESHOLD = 8;

/**
     * 非树化的临界值
     * 如果红黑树的数量小于6，那么转换回链表
     */
static final int UNTREEIFY_THRESHOLD = 6;

/**
     * 转换为红黑树的另一个条件：数组的长度为64以上
     */
static final int MIN_TREEIFY_CAPACITY = 64;
```

由此我们可以看出，JDk1.8由链表转换红黑树的条件**有两个且必须同时满足**：

1. 数组的长度至少为64
2. 链表的长度至少大于8

由红黑树转换回链表的条件：

1. 红黑树的节点数小于6

### 内部类——Node

在JDK1.7中称为**Entry**，在JDK1.8中叫**Node**

**HashMap本质就是一个Node数组，而Node又有next，可以看做链表**

```java
// 是一个静态类，方便HashMap调用（不需要借助引用）
static class Node<K,V> implements Map.Entry<K,V> {
    // hash值，也不可更改
    final int hash;
    // key 与 value：key是不可以更改的
    final K key;
    V value;
    // 连接到下一个节点
    Node<K,V> next;
    // 构造方法
    Node(int hash, K key, V value, Node<K,V> next) {...}
    // get set toString 相关方法，都有final关键字，说明不可以重写
    public final K getKey()   {...}
    public final V getValue()      {...}
    public final String toString()  {...}
    public final V setValue(V newValue) {...}
    /**
         * hash值计算方法：将key与value的hash值进行位运算：异或运算（相异为1，相同为0）
         * Obejcts类的hashcode方法为：
         * return o != null ? o.hashCode() : 0;
         * 可见，除null是0外，其他都调用Obejct类的hashcode方法（即简单对内存地址进行一下处理）
         * @return
         */
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }
    // equals方法
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            // Map的节点都叫Entry，JDK1.8HashMap将自己的Entry改名为Node
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            // objets的equals方法：return (a == b) || (a != null && a.equals(b));
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

### 构造方法

前置变量：

```java
// transient 表名这个变量不需要被序列化;
// HashMap是一个Node数组
transient Node<K,V>[] table;

// HashMap中元素的数量
transient int size;
```

共有四个重载的构造方法:

- 两个参数：初始容量和负载因子

```java
public HashMap(int initialCapacity, float loadFactor) {
        // 参数：初始容量、负载因子
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        // tableSizeFor 返回一个大于输入参数且是2的整数次幂的数
        // 比如：输入10，返回2的三次方为8、2的四次方为16，大于10，那么输出16
        // tableSizeFor 的代码经过优化调优，使用移位与位异或实现
        //threshold用来判断什么时候进行扩容
        this.threshold = tableSizeFor(initialCapacity);
}
```

- 一个参数

```java
public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

- 无参构造

```java
public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

- 将一个Map转为一个HashMap（不讨论此构造）

```java
public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
}
```

### 核心方法

put方法：

```java
static final int TREEIFY_THRESHOLD = 8;
 
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
 }
  
  
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // 参数说明：hash是key的哈希值、onlyIfAbsent如果是真代表无需改变现有的值、evict如果是假，则表（Node数组）在初始化阶段
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 如果当前map无数据，那么执行resize方法
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 如果要插入的键值对要存放的这个位置刚好没有元素，那么把他封装成Node对象，放在这个位置上即可
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 否则有元素
    else {
        Node<K,V> e; K k;
        // a.如果这个元素的key与要插入的一样，那么就替换一下。
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // b.如果这个node是TreeNode，那么就调用Tree的put方法（代表这个结构现在是红黑树）
        else if (p instanceof TreeNode) {
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        }
        // c.此时这个值是完完全全的新值，而且仍然属于链表结构
        else {
            // 遍历这个链表，把值放在合适的位置上
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 下面就是将链表转换为红黑树的判断
                    // 当链表的长度大于TREEIFY_THRESHOLD时，调用treeifyBin方法
                    // （但不一定会树化，另一个条件在这个方法内，当数组长度小于64时，会进行扩容，而不是树化）
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
    // 判断threshold，决定是否扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

treeifyBin方法：（树化）

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
     int n, index; Node<K,V> e;
     //树形化还有一个要求就是数组长度必须大于等于64，否则继续采用扩容策略
     if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
         resize();
     else if ((e = tab[index = (n - 1) & hash]) != null) {
         ... 树化具体过程
    }
}
```

## HashMap原理

### 为什么JDK1.8要进行树化？

原因1：并发操作（不要并发使用hashmap）易死循环

​		JDK1.7的情况下，**并发扩容时容易形成链表环**，此情况在1.8时就好太多太多了。因为在1.8中当链表长度大于阈值（默认长度为8）时，链表会被改成树形（红黑树）结构。

​	JDK1.7采用**表头插入法**。扩容是改变链表原本的顺序，并发易导致链表形成环。

​	JDK1.8采用**尾部插入法**。扩容时保持原有顺序，不会出现链表环问题。

原因2：**安全问题**

​		因为在元素放置过程中，如果一个对象哈希冲突，都被放置到同一个桶里，则会形成一个链表，我们知道链表查询是线性的，会严重影响存取的性能。

​		而在现实世界，构造哈希冲突的数据并不是非常复杂的事情，恶意代码就可以利用这些数据大量与服务器端交互，导致服务器端CPU大量占用，这就构成了哈希碰撞拒绝服务攻击，国内一线互联网公司就发生过类似攻击事件。

> 用哈希碰撞发起**拒绝服务攻击**(DOS，Denial-Of-Service attack)，常见的场景是攻击者可以事先构造大量相同哈希值的数据，然后以JSON数据的形式发送给服务器，服务器端在将其构建成为Java对象过程中，通常以Hashtable或HashMap等形式存储，哈希碰撞将导致哈希表发生严重退化，算法复杂度可能上升一个数据级，进而耗费大量CPU资源。

### 为什么链表转换红黑树的临界值为8？

链表长度符合`泊松分布`，各个长度的命中概率依次递减，当长度为 8 的时候，是最理想的值。也是出于时间与空间的一种平衡。

- 如果太大：链表的查询速度为`O(n)`会大大降低查询速度

- 如果太小：`TreeNode`占用的资源远大于`Node`

### 负载因子的作用

“负载极限”是一个`0～1`的数值，“负载极限”决定了hash表的最大填满程度。

​	当hash表中的负载因子达到指定的“负载极限”时，hash表会自动成倍地增加容量（桶的数量），并将原有的对象重新分配，放入新的桶内，这称为`rehashing`。

“负载极限”的默认值（0.75）是时间和空间成本上的一种折中：

- `较高`的“负载极限”**可以降低hash表所占用的内存空间**，但会增加查询数据的时间开销，而查询是最频繁的操作（HashMap的get()与put()方法都要用到查询）
- `较低`的“负载极限”会**提高查询数据的性能**，但会增加hash表所占用的内存开销

### 线程不安全

为什么HashMap线程不安全：

HashMap使用时，内部无可避免会进行`resize`与`rehash`。

- `rehash`极有可能形成链表环，导致死循环

- `resize`期间，由于会新建一个新的空数组，并且用旧的项填充到这个新的数组中去。所以，在这个填充的过程中，如果有线程获取值，很可能会取到 null 值，而不是我们所希望的、原来添加的值。

### JDK1.7、1.8计算索引的区别

JDK1.7计算索引使用`indexFor`方法：

```java
static int indexFor(int h, int length) {
     // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
     return h & (length-1);
 }
```

hash方法是这样的：

```java
final int hash(Object k) {
     int h = hashSeed;
     if (0 != h && k instanceof String) {
         return sun.misc.Hashing.stringHash32((String) k);
     }
 
     h ^= k.hashCode();
 
     // This function ensures that hashCodes that differ only by
     // constant multiples at each bit position have a bounded
     // number of collisions (approximately 8 at default load factor).
     h ^= (h >>> 20) ^ (h >>> 12);
     return h ^ (h >>> 7) ^ (h >>> 4);
 }
```

---

JDK1.8具体键值对在哈希表中的位置（数组index）取决于下面的位运算（去除了`indexFor`方法）：

```java
i = (n - 1) & hash
// n是数组的长度；hash是要存储Key的hash值
```

仔细观察哈希值的源头，会发现**它并不是key本身的hashCode**，而是来自于HashMap内部的另一个hash方法。

```java
//（这里的hash值方法来源于）
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    //>>>代表无符号右移，高位统统补0；>>则表示带符号右移，高位补符号位；
}
```

为什么这里需要将高位数据移位到低位进行异或运算呢？

​		这是因为有些数据计算出的**哈希值差异主要在高位**，而HashMap里的哈希寻址是忽略容量以上的高位的，那么这种处理就可以有效避免哈希碰撞。

### 为什么要重写`equals`方法？	

​		`equals`默认是继承自Object的方法，它本身是比较内存地址的；假如有一个HashMap，内部有一个由于哈希碰撞拉出的链表，如果我们不重写`equals`方法，不比较内容的话，就只能找到链表为止，找不到我们要的`node`了

​		而重写`hashcode`方法，是为了让相同内容的对象返回相同的哈希值

## HashMap、HashTable、TreeMap区别

- 可以存储NULL？
  - **HashMap可以存储key为null，value为null的值，但是只能有一个key为null的元素**
  - HashTable、TreeMap都不能为null

> 为什么hashTable的key不能为null，但是Hashmap却可以？

​		因为HashTable对于null的key是直接抛异常的，但是HashMap内部的hash方法进行判断，如果是null，返回0

> 如果深究的话，就是因为**安全失败机制(fail safe)**，这种机制会使你此次**读到的数据不一定是最新的数据。**

如果你使用null值，就会使得其无法判断对应的key是不存在还是为空，因为你无法再调用一次`contain(key)`来对key是否存在进行判断`ConcurrentHashMap`同理。

- 初始容量：
  - hashmap为16
  - HashTable为11（负载因子都是0.75）
  - 扩容机制：
  - Hashmap翻倍
  - HashTable翻倍+1
- 是否有序？
  - HashMap、HashTable无序
  - TreeMap能够对保存的记录根据键进行排序
- 时间复杂度
  - HashTable、HashMap具有`O(1)`级别时间复杂度
  - TreeMap因为有序，所以为`O(logn)`
- 线程安全问题：
  - HashMap线程不安全
  - HashTable线程安全
- 底层实现：
  - HashMap：数组+链表+红黑树
  - TreeMap：数组+红黑树
  - HashTable：数组+链表
- 选择：
  - 首选HashMap
  - 需要排序则选TreeMap



