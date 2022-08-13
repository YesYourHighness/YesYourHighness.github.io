---
title: Java链表_手写一个LinkedList
date: 2020-03-05 20:12:53
tags:
- 数据结构
- 队列
- Java
categories:
- 数据结构
---

<center>
    引言：数据结构——链表
</center>

<!--more-->

# LinkedList

在线性表家族中，

对于数组、栈、队列来说，他们虽然实现了动态，**但底层却还是静态的**

但是对于链表来说，它是**真正的动态**，但失去了随机读取的功能


方法如下
![image](https://github.com/YesYourHighness/MyPicStore/raw/master/dataStruct/linkedList.png)

代码如下：
```java
/**
 * 手写一个链表
 * @author BlackKnight
 */
public class LinkedList<E>{
    /**
     * 链表的核心——结点
     * 私有内部类
     */
    private class Node{
        public E e;
        public Node next;

        /**
         * 方便我的LinkedList类来访问结点
         * 有多个重载方法
         * @param e
         * @param next
         */
        public Node(E e, Node next) {
            this.e = e;
            this.next = next;
        }

        public Node(E e) {
            this(e,null);
        }

        public Node() {
            this(null,null);
        }

        @Override
        public String toString() {
            return "Node{" +
                    "e=" + e +
                    ", next=" + next +
                    '}';
        }
    }

    private Node head;
    private int size;

    public LinkedList() {
        this.head = null;
        this.size = 0;
    }
    public int getSize(){
        return size;
    }
    public boolean isEmpty(){
        return size==0;
    }

    /**
     * 向链表头添加结点
     * @param e
     */
    public void addFirst(E e){
        /*不同于数组在末尾添加元素，链表更容易在头head添加元素
          将一个元素添加到一个链表的头部，其实只需要两步就能完成
            1. 将要插入的结点的next指向当前链表的head
            2. 再把head重新指向被插入结点
         */
        //Node node = new Node(e);
        //node.next = head;
        //head = node;
        //上述三行代码可以更 优雅 ，一行就可以完成
        head = new Node(e,head);
        size++;
    }

    /**
     * 向链表中间添加结点
     * @param index
     * @param e
     */
    public void add(int index, E e){
        if(index<0||index>size){
            throw new IllegalArgumentException("无效的index");
        }
        //如果要放在第0个位置上，就需要判断一下，因为没有第-1个结点
        //但是我们可以单独设置一个 虚拟头结点 来避免这种判断
        if(index==0){
            addFirst(e);
        }else {
            Node prev = head;
            //插入的核心：找到所要插入位置的前一个结点！
            //这里的循环条件注意要理解，不理解可以画画图，举举例子
            for(int i=0; i < index - 1 ;i++){
                prev = prev.next;
            }
            //Node node = new Node(e);
            //node.next = prev.next;
            //prev.next = node;
            //同样，可以更加的优雅
            prev.next = new Node(e,prev.next);
            size++;
        }
    }

    public void addLast(E e){
        add(size,e);
    }

}

```
上述的代码并没有写完，因为在使用过程中，**带有虚拟头结点**的链表是非常多的，所以我们下面来写使用带有虚拟头结点的链表

真正的代码如下：
```java
package linkedList;

import java.util.StringJoiner;

/**
 * 手写一个链表
 * @author BlackKnight
 */
public class LinkedList<E>{
    /**
     * 链表的核心——结点
     * 私有内部类
     */
    private class Node{
        public E e;
        public Node next;

        /**
         * 方便我的LinkedList类来访问结点
         * 有多个重载方法
         * @param e
         * @param next
         */
        public Node(E e, Node next) {
            this.e = e;
            this.next = next;
        }

        public Node(E e) {
            this(e,null);
        }

        public Node() {
            this(null,null);
        }

        @Override
        public String toString() {
            return "Node{" +
                    "e=" + e +
                    ", next=" + next +
                    '}';
        }
    }

    private Node dummyHead;
    private int size;

    public LinkedList() {
        //虚拟头结点
        this.dummyHead = new Node(null,null);
        this.size = 0;
    }
    public int getSize(){
        return size;
    }
    public boolean isEmpty(){
        return size==0;
    }

    /**
     * 向链表中间添加结点
     * @param index
     * @param e
     */
    public void add(int index, E e){
        if(index<0||index>size){
            throw new IllegalArgumentException("无效的index");
        }
        //设置一个了虚拟头结点 避免判断是不是要插在0的位置上
        Node prev = dummyHead;
        //插入的核心：找到所要插入位置的前一个结点！
        //这里的循环条件注意要理解，不理解可以画画图，举举例子
        for(int i=0; i < index ;i++){
            //因为我们从虚拟头结点开始遍历，条件就不再是 index-1 了
            prev = prev.next;
        }
        //Node node = new Node(e);
        //node.next = prev.next;
        //prev.next = node;
        //同样，可以更加的优雅
        prev.next = new Node(e,prev.next);
        size++;
    }

    /**
     * 向链表头添加结点
     * @param e
     */
    public void addFirst(E e){
        /*
        因为使用了虚拟头结点，所以直接可以复用add方法了
         */
        add(0,e);
        size++;
    }

    /**
     * 向链表位添加结点
     * @param e
     */
    public void addLast(E e){
        add(size,e);
    }

    /**
     * 获得链表第index个元素
     * 并不常用
     * @param index
     * @return
     */
    public E get(int index){
        if(index<0||index>size){
            throw new IllegalArgumentException("无效的index");
        }
        /*
        不同于插入时的遍历，这里的遍历需要遍历到index，而不是index的前一个
         */
        Node cur = dummyHead.next;
        //这里直接赋值到了第0个结点
        for(int i=0 ;i<index;i++){
            cur = cur.next;
        }
        return cur.e;
    }

    public E getFirst(){
        return get(0);
    }
    public E getLast(){
        return get(size);
    }

    /**
     * 修改的方法
     * @param index
     * @param e
     */
    public void set(int index , E e){
        if(index<0||index>size){
            throw new IllegalArgumentException("无效的index");
        }
        Node cur = dummyHead.next;
        for(int i=0 ;i<index;i++){
            cur = cur.next;
        }
        cur.e = e;
    }

    /**
     * 查找
     * @param e
     * @return
     */
    public boolean contains(E e){
        Node cur = dummyHead.next;
        while (cur!=null){
            if(cur.e.equals(e)){
                return true;
            }
            cur = cur.next;
        }
        return false;
    }

    @Override
    public String toString() {
        StringJoiner sj = new StringJoiner("->","LinkedList: dummyHead->[","]->null");
        Node cur = dummyHead.next;
        while (cur!=null){
            sj.add(cur.e+"");
            cur = cur.next;
        }
        return sj.toString();
    }

    /**
     * 删除一个结点
     * @param index
     * @return
     */
    public E remove(int index){
        if(index<0||index>size){
            throw new IllegalArgumentException("无效的index");
        }
        Node prev = dummyHead;
        /*
        删除一个结点，也是要找要删除结点的前一个结点
         */
        for(int i = 0;i < index;i++){
            prev = prev.next;
        }
        Node ret = prev.next;
        prev.next = prev.next.next;

        //将被删除结点从链表中分离出来
        ret.next = null;
        size--;
        return ret.e;
    }

    public E removeFirst(){
        return remove(0);
    }
    public E removeLast(){
        return remove(size-1);
    }

}
```

# 复杂度分析
```
方法名                  | 时间复杂度 |
--------------------------------------
add()                   |     O(n)   |
addFirst()              |     O(1)   | 
addLast()               |     O(n)   |
--------------------------------------      
remove()                |     O(n)   |
removeFirst()           |     O(1)   |
removeLast()            |     O(n)   |
--------------------------------------
get()                   |     O(n)   |
isEmpty()               |     O(1)   |
contains()              |     O(n)   |
--------------------------------------
set()                   |     O(n)   |

注意：O(n/2) = O(n)
```
由此可见，除了少数对头操作的方法外，几乎所有的方法，时间复杂度都是`O(n)`


# 剖析虚拟头结点的作用

`LeetCode`中有一道题

删除链表中等于给定值 val 的所有节点。
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
示例:

输入: 1->2->6->3->4->5->6, val = 6
输出: 1->2->3->4->5
```

这里有两种不同的解决办法

## 不用虚拟头结点
```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        //假如一开始就出现了val
        //注意，当删除完一个结点后，不用再移动结点了
        while(head!=null && head.val==val){
            ListNode del = head;
            head = head.next;
            del.next = null;
        }

        if(head==null){
            return null;
        }

        ListNode prev = head;

        //这里删除中间出现的val
        while(prev.next!=null){
            if(prev.next.val == val){
                ListNode delNode = prev.next;
                prev.next = prev.next.next;
                delNode.next = null;
            }
            else prev = prev.next;
        }
        return head;
    }
}
```
## 使用虚拟头结点
```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        
        ListNode dummyHead = new ListNode(0);
        dummyHead.next = head;
        ListNode prev = dummyHead;
        //这里删除中间出现的val
        while(prev.next!=null){
            if(prev.next.val == val){
                ListNode delNode = prev.next;
                prev.next = prev.next.next;
                delNode.next = null;
            }
            else prev = prev.next;
        }
        return dummyHead.next;
    }
}
```
## 区别
使用虚拟头结点，可以不用再讨论head的特殊性，因为此时所有的结点都有一个前置的结点

返回的时候返回虚拟头结点的next即可

所以我们可以有如下总结
- 如果只是操作头结点，则我们可以不使用虚拟头结点
- 如果要对中间的结点进行操作，使用虚拟头结点可以避免分类讨论
- 两者可以互换，但要注意条件


# 参考资料
> 参考资料：
[来自慕课liuyubobobo的教程](https://coding.imooc.com/class/207.html)


