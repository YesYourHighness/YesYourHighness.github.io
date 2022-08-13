---
title: Java队列_手写一个LoopQueue
date: 2020-03-04 21:59:59
tags:
- 数据结构
- 队列
- Java
categories:
- 数据结构
---


<center>
    引言：数据结构——循环队列
</center>

<!--more-->


# LoopQueue

在实现完`ArrayQueue`之后，发现`ArrayQueue`有一个致命的缺点，就是出队列需要O(n)的时间复杂度，因为每次出队时，后面的元素都要向前移动一个空间，所以造成了时间复杂度上升

## 怎么解决这个问题呢？

在出队之后，我们可以不移动，使用一个`front`和`tail`来标记队头和队尾，再把队头队尾连起来，实现一个循环队列

这样当出队时，只需要`front++`，入队只需要`tail++`，时间复杂度都是O(1)，解决了上述的问题

## 循环队列需要什么方法

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/dataStruct/loopQueue.png)

----


这里我们会继续实现`Queue`这个接口，`Queue`接口的代码如下，和
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




## 代码如下：
```java
package queue;

import java.util.StringJoiner;

/**
 * @author BlackKnight
 * 实现循环队列
 */
public class LoopQueue<E> implements Queue {
    private E[] data;
    private int front , tail;
    private int size;
    private final static int initialCapacity = 10;

    /**
     * 构造方法
     * @param capacity
     */
    public LoopQueue(int capacity) {
        data = (E[]) new Object[capacity+1];
        //这里为什么要+1呢？
        /*我们在判断循环队列满或空的时候，需要使用一个空间来分辨
          * 假如我们不+1
            空的条件为：front == tail;
            满的条件为：front == tail;
          * 假如我们+1
            空的条件为：front == tail;
            满的条件为：front == tail + 1;
            （考虑到tail指向最后一个空间的时候，
              判断满正确来说应该是：
              (tail + 1) % capacity = front;）
        */
        front = 0;
        tail = 0;
        size = 0;
    }

    public LoopQueue() {
        this(initialCapacity);
    }

    public int getCapacity(){
        //有一个空间浪费掉，所以-1
        return data.length - 1;
    }

    @Override
    public int getSize() {
        return size;
    }

    @Override
    public boolean isEmpty() {
        //判断空的条件
        return front == tail;
    }

    /**
     * 入队
     * @param o
     */
    @Override
    public void enqueue(Object o) {
        //判断是否需要扩容
        if((tail +1 )% data.length == front){
            resize(getCapacity() * 2);
        }
        data[tail] = (E) o;
        tail = (tail +1)% data.length;
        size++;
    }

    /**
     * 扩容
     * @param newCapacity
     */
    private void resize(int newCapacity) {
        E[] newData = (E[]) new Object[newCapacity];
        /*
        我们扩容的时候可以直接往0~size赋值
         */
        for (int i = 0; i < size; i++) {
            //这里data的下标需要注意！！！
            newData[i] = data[(front + i) % data.length];
        }
        data = newData;
        front = 0;
        tail = size;
    }

    /**
     * 出队
     * @return
     */
    @Override
    public E dequeue() {
        if(size == 0){
            throw new IllegalArgumentException("队列为空，不能删除");
        }
        E ret = data[front];
        /*同样为了防止出现游荡对象loitering objects，这里手动的置空*/
        data[front] = null;
        /*循环队列，注意不能简单的++*/
        front = (front+1)%data.length;
        size--;
        /*
            防止复杂度震荡，所以判断size == getCapacity()/4
            且防止当队列capacity为2时，出现2/2==0，重新分配0个内存的情况，所以判断getCapacity()/2!=0
        */
        if(size == getCapacity()/4 && getCapacity()/2!=0){
            resize(getCapacity()/2);
        }
        return ret;
    }

    @Override
    public E getFront() {
        if (isEmpty()){
            throw new IllegalArgumentException("队列为空");
        }
        return data[front];
    }

    @Override
    public String toString() {
        StringJoiner sj = new StringJoiner(",","LoopQueue: front [","] tail ");
        /*
        注意这里循环的方式！！
        循环数组，我们的i也必须循环起来
         */
        for(int i =front; i!=tail ;i= (i+1)%data.length){
            sj.add(data[i]+"");
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
dequeue()               |     O(1)   | （均摊复杂度）
front()                 |     O(1)   |
getSize()               |     O(1)   |
isEmpty()               |     O(1)   |
```

# ArrayQueue 对比 LoopQueue
数组队列和循环队列，他们之间的差距只在`dequeue()`这个方法上，一个是`O(n)`,一个是`O(1)`

这样的差距是天翻地覆的，下面的测试代码
```java
public class Main {
    public static void main(String[] args) {
        int loopTurn = 500_000;
        Queue<Integer> arrayQueue = new ArrayQueue<>();
        Queue<Integer> loopQueue = new LoopQueue<>();
        System.out.println("arrayQueue运行时间："+testQueue(loopTurn,arrayQueue)+"s");
        System.out.println("loopQueue运行时间："+testQueue(loopTurn,loopQueue)+"s");
    }

    private static double testQueue(int loopTurn, Queue<Integer> q) {
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
让他们各自分别执行五十万次，（原本想测试一百万次的，但是觉得太慢了）

在我的电脑上得到如下成绩



```java
arrayQueue运行时间：70.0025624s
loopQueue运行时间：0.0590492s
```
嘶！倒吸一口凉气，要不是我的音乐还在播放，我会以为我电脑卡了的。



# 参考资料
> 参考资料：
[来自慕课liuyubobobo的教程](https://coding.imooc.com/class/207.html)