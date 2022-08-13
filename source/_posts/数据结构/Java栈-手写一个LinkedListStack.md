---
title: Java栈_手写一个LinkedListStack
date: 2020-03-05 20:21:08
tags:
- 数据结构
- 队列
- Java
categories:
- 数据结构
---

<center>
    引言：链表栈
</center>

<!--more-->

# LinkedListStack

之前我们说过，实现栈有两种方式，这次就来通过链表来实现栈


由于底层不需要用户知道，我们还是继续实现栈的
接口：
```java
/**
 * @author BlackKnight
 */
public interface Stack<E> {
    /**
     * 获取大小
     * @return
     */
    int getSize();

    /**
     * 判空
     * @return
     */
    boolean isEmpty();

    /**
     * 入栈
     * @param e
     */
    void push(E e);

    /**
     * 出栈/弹栈
     * @return
     */
    E pop();

    /**
     * 查看栈顶元素
     * @return
     */
    E peak();
}
```

实现的功能:

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/dataStruct/LinkedListStack.png)

代码如下：
```java
package stack;

import linkedList.LinkedList;

/**
 * 使用链表来实现栈
 * @author BlackKnight
 */
public class LinkedListStack<E> implements Stack{
    private LinkedList<E> list;

    public LinkedListStack() {
        list = new LinkedList<E>();
    }

    @Override
    public int getSize() {
        return list.getSize();
    }

    @Override
    public boolean isEmpty() {
        return list.isEmpty();
    }

    @Override
    public void push(Object o) {
        list.addFirst((E)o);
    }

    @Override
    public Object pop() {
        return list.removeFirst();
    }

    @Override
    public Object peak() {
        return list.getFirst();
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("Stack: top");
        sb.append(list);
        return sb.toString();
    }
}
```

# 复杂度分析
```
方法名                  | 时间复杂度 |
-------------------------------------------------
push(E e)               |     O(1)   |
pop()                   |     O(1)   |
peek()                  |     O(1)   |
getSize()               |     O(1)   |
isEmpty()               |     O(1)   |
```
由于`push pop`方法底层是`addFirst removeFirst`，所以他们的时间复杂度其实也是O(1)

这和`ArrayList`是一致的

# ArrayStack 与 LinkedListStack 比较

分别使用两个不同的数据结构实现的栈，他们速度有什么不同呢

```java
public class SpeedTest {
    public static void main(String[] args) {
        int loopTurn = 10_000_000;
        Stack<Integer> arrayStack = new ArrayStack<>();
        Stack<Integer> linkedListStack = new LinkedListStack<>();
        System.out.println("arrayStack运行时间："+testSpeed(loopTurn,arrayStack)+"s");
        System.out.println("linkedListStack运行时间："+testSpeed(loopTurn,linkedListStack)+"s");
    }

    private static double testSpeed(int loopTurn, Stack<Integer> q) {
        long startTime = System.nanoTime();
        for(int i=0;i<loopTurn;i++) {
            Random r = new Random();
            q.push(r.nextInt(Integer.MAX_VALUE));
        }
        for(int i=0;i<loopTurn;i++) {
            q.pop();
        }
        long endTime = System.nanoTime();
        return (endTime - startTime) / 1000000000.0;
    }
}
```
运行的结果如下
```
arrayStack运行时间：2.4849806s
linkedListStack运行时间：4.1543247s
```
看起来`LinkedListStack`好像要比`ArrayStack`慢，

但其实我们分析他们的时间复杂度是一样的

在JVM中，`LinkedListStack`慢的原因是因为，运行如此大的数据，它的事件浪费在了`new`结点上，而不是操作上


# 参考资料
> 参考资料：
[来自慕课liuyubobobo的教程](https://coding.imooc.com/class/207.html)


