---
title: Java队列_手写一个ArrayQueue
date: 2020-03-03 23:26:18
tags:
- 数据结构
- 队列
- Java
categories:
- 数据结构
---


<center>
    引言：数据结构——队列
</center>

<!-- more -->

# ArrayQueue


实现队列同样有两种方式，这里我们使用`Array`来实现

用户不需要知道底层的实现方式，所以我们先写一个`Queue`的接口吧
```java
/**
 * @author BlackKnight
 */
public interface Queue<E> {
    int getSize();
    boolean isEmpty();

    /**
     *入队列
     * @param e
     */
    void enqueue(E e);
    /**
     * 出队列
     * @return
     */
    E dequeue();

    /**
     * 得到队头元素
     * @return
     */
    E getFront();
}

```

然后我们想一想，我们需要什么方法？

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/dataStruct/arrayQueue.png)


```java
import array.Array;

import java.util.StringJoiner;

/**
 * @author BlackKnight
 */
public class ArrayQueue<E> implements Queue{
    Array<E> array;

    public ArrayQueue(int capacity) {
        this.array = new Array<>(capacity);
    }

    public ArrayQueue() {
        array = new Array<>();
    }

    @Override
    public int getSize() {
        return array.getSize();
    }

    @Override
    public boolean isEmpty() {
        return array.isEmpty();
    }

    /**
     * 添加从尾添加
     * @param o
     */
    @Override
    public void enqueue(Object o) {
        array.addLast((E)o);
    }
    /**
     * 出从队头出
     * @return
     */
    @Override
    public E dequeue() {
        return array.removeFirst();
    }

    @Override
    public E getFront() {
        return array.get(0);
    }

    public int getCapacity(){
        return array.getCapacity();
    }

    @Override
    public String toString() {
        StringJoiner sj = new StringJoiner(",","Queue: front [","] tail");
        for(int i =0;i<array.getSize();i++){
            sj.add(array.get(i)+"");
        }
        return sj.toString();
    }
}

```




# 复杂度分析
```
方法名                  | 时间复杂度 |
-------------------------------------------------
enqueue(E e)            |     O(1)   | （均摊复杂度）
dequeue()               |     O(n)   |
front()                 |     O(1)   |
getSize()               |     O(1)   |
isEmpty()               |     O(1)   |
```
这里我们可以注意到，在使用Array来实现队列，它的出队时间复杂度需要O(n)
，在数据量很大的情况下，出队一次就需要浪费我们很长时间

那么我们怎么解决这个问题呢？

使用**循环队列**







# 参考资料
> 参考资料：
[来自慕课liuyubobobo的教程](https://coding.imooc.com/class/207.html)