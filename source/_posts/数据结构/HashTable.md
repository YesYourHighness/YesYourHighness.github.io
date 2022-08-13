---
title: HashMap哈希表
date: 2021-06-08 15:12:00
tags:
- 数据结构
- HashTable
- Hash
categories:
- 数据结构
- HashTable
---

<center>
    引言：哈希表——一个链表数组
</center>

<!-- more -->

# 哈希表

## 哈希表基础

哈希表数据结构，主要是为了使用空间换取查找时间，如果我们有一个无限大的空间，理论上我们查找任何数据都可在O(1)的时间内完成

我们可以使用哈希来实现集合、映射等等数据结构

### 哈希函数

映射关系就被称作**哈希函数**

比如说将26个小写字母映射到26个数字上，在Java中

```java
public int hashFunc(char chr){
    return chr - 'a';
}
```

这就是一个哈希函数，但是在实际中，我们使用的都是一个一个对象，将对象转换为一个索引是比较复杂的

### 哈希冲突

实际中对象转换成一个索引，通常是不可能做到每一次都转换为不同的索引的，所以当有两个对象经过哈希函数转换后，发现有着相同的索引，此时就发生了**哈希冲突**。

而在基于哈希表实现的数据结构中，如何解决哈希冲突就是一个重中之重！

### 哈希函数的设计

#### 存储整数与浮点数

假如要存储**一堆大的整数**，我们要把它转换为一个索引，最简单的哈希函数，就是让这个整数对一个**精心设计的整数**取模

为什么要说是精心设计的整数呢？

- **保证索引分布均匀**：开辟一个空间，得保证大量的数据存入之后，是均匀的

试想我们使用2、4、6、8这种偶数来作为被取模的数，那么当大量的数据来存放时，会发现他们的倍数都存放到了一起，引起了大量的**哈希冲突**

如果选择3,、5、9这种奇数，效果和偶数其实差不多，许多数学家研究后，**发现选择一个素数**的效果是最好的，可以让索引分布均匀，最大程度减少哈希冲突

而且不同数量级选择的最好的数也不一样，例如在`2^5~2^6`这个数量级之间，选择53是最好的选择，但是在上一个量级`2^6~2^7`选择97是最好的选择，其他量级我们可以去搜索

假如要存储**一堆浮点数**，尽管和整数不一样，但是底层存储还是差不多的，都是存放在32位或是64位的内存中，大小是固定的

#### 存储字符串

假如要存储**一堆字符串**（简单考虑其全由小写字母组成），那么可以将其转换为一个整型处理，如何转换呢？一共有26个小写字母，我们就可以使用**26进制**来实现，例如`dac =d*26^2+ a*26^1 + c*26^0`。如果包括大小写字母，那么用52进制就行，如果有更多更多，我们也可以解决，类似这样

```
# B代表进制，对M求余，防止这个数太大
hash("字符串") = ("字"*B^2 + "符"*B^1 + "串"*B^0) % M

# 上面的式子等同于下面的式子
hash("字符串") = (((("字"*B)+ "符")*B + "串")*B) % M

# 为了防止在运算的过程中出现整型溢出，最终可以写成这个样子
hash("字符串") = ((("字" % M)*B+ "符")%M*B + "串")%M 
```

#### 存储对象

更一般的，假如要存储一堆学生对象（有姓名、学号、年龄、性别等等属性），仍然可以转换为整型，

借鉴字符串的思想，就是这样

```
hash(s1) = (((s1.name % M)*B+ s1.id)%M*B + s1.age)%M 
```

#### 设计原则

总之有着这样几个设计原则：

- 分布均匀
- 一致性：如果a==b那么a的hash值应该和b的哈希值相同
- 高效性：哈希运算可以很快完成



### Java中的哈希函数——hashcode

> hashcode：将对象转换为一个整型值

在Java中，对于**整型**来说，它的hashcode就是**它本身**，而且有正有负

```java
Integer a1 = 123;
System.out.println(a1.hashCode()); // 123
Integer a2 = 1233192123;
System.out.println(a2.hashCode()); // 1233192123
Integer a3 = -1233192123;
System.out.println(a3.hashCode()); // -1233192123
```

对于自定义的类，没重写hashcode方法前，会自动调用继承自Obejct类的hashcode方法，而这个方法是对存储的地址进行一下转换，转换为一个整型值

我们可以重写hashcode方法，例如：

```java
 public class Student {
    private String name;
    private String sex;
    private Integer age;

    public Student(String name, String sex, Integer age) {
        this.name = name;
        this.sex = sex;
        this.age = age;
    }

    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + (sex != null ? sex.hashCode() : 0);
        result = 31 * result + (age != null ? age.hashCode() : 0);
        return result;
    }
}
```



## 哈希冲突的解决

解决哈希冲突，有很多种方法，这里只介绍最出名也是Java8目前所使用的方法——**链地址法**

即如果出现哈希冲突，就把冲突的元素放在一个**链表**里（虽然叫链地址法，但不一定是链表，也可以使用tree等结构）

（Java8之前，每一个位置对应一个链表、Java8开始，当哈希冲突达到一定程度时，会将链表转换成红黑树）



## Java实现哈希表

哈希表的实质：**一个TreeMap数组或者是一个链表数组**

```java
package hash;

import java.util.TreeMap;

/**
 * @author 董文浩
 * @Date 2021/6/8 17:45
 * 哈希表
 */
public class HashTable<K, V> {
    /**
     * 底层使用TreeMap实现（TreeMap是用红黑树实现的）
     */
    private TreeMap<K, V>[] hashtable;
    /**
     * 此M即被模的数
     */
    private int M;
    /**
     * 容量
     */
    private int size;

    /**
     * 指定上下界，当N / M超过upperTol扩容，小于lowerTol时缩容
     * （N是size大小）
     * initCapacity初始化大小
     */
    private static final int upperTol = 10;
    private static final int lowerTol = 2;
    private static final int initCapacity = 7;

    public HashTable() {
        this(97);
    }

    public HashTable(int M) {
        this.M = M;
        this.size = 0;
        hashtable = new TreeMap[M];
        for (int i = 0; i < M; i++) {
            hashtable[i] = new TreeMap<>();
        }
    }

    public int hash(K key) {
        /**
         * 因为hashcode值肯能为负数，例如-84的hash值还是-84
         * 所以与0x7fffffff求与运算就相当于求绝对值
         */
        return (key.hashCode() & 0x7fffffff) % M;
    }

    public int getSize() {
        return size;
    }

    /**
     * 添加元素
     *
     * @param key
     * @param value
     */
    public void add(K key, V value) {
        // 先进行哈希函数转换，然后放在数组指定位置即可
        TreeMap<K, V> map = hashtable[hash(key)];
        if (map.containsKey(key)) {
            // 如果包含了key，那么就更新一下值
            map.put(key, value);
        } else {
            // 添加逻辑
            map.put(key, value);
            size++;
            // N / M > upperTol扩容
            if (size >= upperTol * M) {
                resize(2 * M);
            }
        }
    }

    /**
     * 删除方法
     *
     * @param key
     * @return
     */
    public V remove(K key) {
        TreeMap<K, V> map = hashtable[hash(key)];
        if (map.containsKey(key)) {
            size--;
            // 缩容
            if (size < lowerTol * M && M / 2 > initCapacity) {
                resize(M / 2);
            }
            return map.remove(key);
        }

        return null;
    }

    public void set(K key, V value) {
        TreeMap<K, V> map = hashtable[hash(key)];
        if (!map.containsKey(key)) {
            throw new IllegalArgumentException("键值不存在");
        }
        map.put(key, value);
    }

    public boolean contains(K key) {
        return hashtable[hash(key)].containsKey(key);
    }

    public V get(K key) {
        return hashtable[hash(key)].get(key);
    }

    /**
     * 自动扩容
     * @param newM
     */
    private void resize(int newM) {
        TreeMap<K,V>[] newHashTable = new TreeMap[newM];
        for (int i = 0; i < newHashTable.length; i++) {
            newHashTable[i] = new TreeMap<>();
        }
        int oldM = M;
        this.M = newM;
        for (int i = 0; i < oldM; i++) {
            TreeMap<K, V > map = hashtable[i];
            for (K key:map.keySet()){
                newHashTable[hash(key)].put(key, map.get(key));
            }
        }
    }
}

```

