---
title: Java队列_手写一个LinkedListQueue
date: 2020-03-05 20:21:29
tags:
- 数据结构
- 队列
- Java
categories:
- 数据结构
---


<center>
    引言：数据结构——链表队列
</center>

<!--more-->


# LinkedListQueue

如果我们直接使用之前我们编写的`LinkedList`的话，为了实现队列的FIFO的特性，我们必须选一端入，一端出，但是这就有了一个问题

我们的`LinkedList`有头结点，向头添加或者删除都是O(1)，很方便，但是如果是对尾部的操作的话，就都是O(n)了

这其实和使用`Array`来实现的一样，在数组中，操作尾部是简单的，但是操作头是困难的

所以为了提高链表队列的性能，我们需要改进一下现有的`LinkedList`，即**再增添一个尾节点**

又因为实现队列，只需要FIFO，即只需要对头和尾来进行操作，所以我们不需要设置一个虚拟头结点



![image](https://github.com/YesYourHighness/MyPicStore/raw/master/dataStruct/LinkedListQueue.png)


```java
/**
 * 改进LinkedList实现队列
 * @author BlackKnight
 */
public class LinkedListQueue<E> implements Queue{

    /**
     * 我们依然需要结点
     */
    private class Node{
        public E e;
        public Node next;

        /**
         * 方便我的LinkedListQueue类来访问结点
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
    private Node tail;
    private int size;

    public LinkedListQueue() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }

    @Override
    public int getSize() {
        return size;
    }

    @Override
    public boolean isEmpty() {
        return size==0;
    }

    /**
     * 入队
     * 我们需要对链表的尾部进行操作
     * @param o
     */
    @Override
    public void enqueue(Object o) {
        //如果尾节点是空，说明队列中没有元素
        if(tail == null){
            tail = new Node((E)o);
            head = tail;
        }else {
            tail.next =  new Node((E)o);
            tail = tail.next;
        }
        size++;
    }

    @Override
    public E dequeue() {
        if(isEmpty()){
            throw new IllegalArgumentException("队列为空，出队错误");
        }
        Node delNode = head;
        head = head.next;
        delNode.next = null;
        //假如只有一个元素，我们删掉他之后，head指向空，但是tail还指向那个被删除的元素，所以置空tail
        if(head == null){
            tail = null;
        }
        size--;
        return delNode.e;
    }

    @Override
    public E getFront() {
        if(isEmpty()){
            throw new IllegalArgumentException("队列为空，出队错误");
        }
        return head.e;
    }

    @Override
    public String toString() {
        StringJoiner sj = new StringJoiner("->","Queue: Head [","] tail");
        Node cur = head;
        while (cur!=null){
            sj.add(cur.e+"");
            cur = cur.next;
        }
        return sj.toString();
    }
}
```

# 复杂度分析
```java
方法名                  | 时间复杂度 |
-------------------------------------------------
enqueue()               |     O(1)   |
dequeue()               |     O(1)   |
front()                 |     O(1)   |
getSize()               |     O(1)   |
isEmpty()               |     O(1)   |
```


# LinkedListQueue 对比 LoopQueue

`Queue`现在我们有三种
- ArrayQueue
- LoopQueue
- LinkedListQueue

`ArrayQueue`速度慢于`LoopQueue`很多，那么`LinkedListQueue`和`LoopQueue`相比会怎么样呢

```java
public class SpeedTest {
    public static void main(String[] args) {
        int loopTurn = 10_000_000;
        Queue<Integer> loopQueue = new LoopQueue<>();
        Queue<Integer> linkedListQueue = new LinkedListQueue<>();
        System.out.println("loopQueue运行时间："+testSpeed(loopTurn,loopQueue)+"s");
        System.out.println("linkedListQueue运行时间："+testSpeed(loopTurn,linkedListQueue)+"s");
    }

    private static double testSpeed(int loopTurn, Queue<Integer> q) {
        long startTime = System.nanoTime();
        for(int i=0;i<loopTurn;i++) {
            Random r = new Random();
            q.enqueue(r.nextInt(Integer.MAX_VALUE));
        }
        for(int i=0;i<loopTurn;i++) {
            q.dequeue();
        }
        long endTime = System.nanoTime();
        return (endTime - startTime) / 1000000000.0;
    }
}
```
执行一百万次代码
```
loopQueue运行时间：2.8736317s
linkedListQueue运行时间：4.035901s
```
对比结果，`LinkedListQueue`慢了几秒，这是由于不断的`new`结点造成的速度下降