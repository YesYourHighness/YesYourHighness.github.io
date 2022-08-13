---
title: 集合-java实现
date: 2020-09-13 11:02:33
tags:
- 数据结构
- 集合

categories:
- 数据结构
- 集合
---
<center>
    引言：数据结构——集合Set
</center>

<!--more-->
# 集合

## Set

> 集合：
>
> 一个无序的，不会存放相同元素的特殊的数据结构。

需要实现如下方法：

```java
public interface Set<E> {
    void add(E e);
    void remove(E e);
    boolean contains(E e);
    int getSize();
    boolean isEmpty();
}
```

特点：

- 不含重复元素
- 没有顺序

 分别使用**二叉搜索树**和**链表**来实现集合

## LinkedListSet

```java
public class LinkedListSet<E> implements Set<E> {

    private LinkedList<E> linkedList;

    public LinkedListSet() {
        linkedList = new LinkedList<>();
    }

    @Override
    public void add(E e){
        if(!linkedList.contains(e)){
            linkedList.addFirst(e);
        }

    }

    @Override
    public void remove(E e) {
        linkedList.removeElement(e);
    }

    @Override
    public boolean contains(E e) {
        return linkedList.contains(e);
    }

    @Override
    public int getSize() {
        return linkedList.getSize();
    }

    @Override
    public boolean isEmpty() {
        return linkedList.isEmpty();
    }

}

```

## BstSet

二分搜索树来实现集合

```java
public class BstSet<E extends Comparable<E>> implements Set<E> {

    private BinarySearchTree<E> bst;

    public BstSet() {
        bst = new BinarySearchTree<>();
    }

    @Override
    public void add(E e) {
        // 实现的二叉搜索树就是不会重复的
        bst.add(e);
    }

    @Override
    public void remove(E e) {
        bst.remove(e);
    }

    @Override
    public boolean contains(E e) {
        return bst.contains(e);
    }

    @Override
    public int getSize() {
        return bst.size();
    }

    @Override
    public boolean isEmpty() {
        return bst.isEmpty();
    }
}
```

## 复杂度分析

### LinkedListSet

- 增 add：O(n)
- 查 contains：O(n)
- 删 remove: O(n)



### BstSet

最多访问树的高度 h

<img src="http://img.yesmylord.cn//image-20200908174610307.png" alt="image-20200908174610307" style="width:33%;" />

- 增 add ：O(log n) 
- 查 contains：O(log n)
- 删 remove: O(log n)

 

### 结论

二叉搜索树实现的集合要比链表实现的集合性能高的多。

但是当存储的数据为顺序的话，二叉搜索树有可能是这个样子的

<img src="http://img.yesmylord.cn//image-20200908174920746.png" alt="image-20200908174920746" style="width:50%;" />

这个时候二叉搜索树的性能将会和链表一样（AVL树可以解决这种问题）。

