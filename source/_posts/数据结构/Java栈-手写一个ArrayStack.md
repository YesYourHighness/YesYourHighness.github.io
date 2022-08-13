---
title: Java栈_手写一个ArrayStack
date: 2020-03-03 23:26:04
tags:
- 数据结构
- 栈
- Java
categories:
- 数据结构
---
<center>
    引言：数据结构——队列
</center>

<!--more-->

# ArrayStack

实现栈有两种方法，但这次我们使用`Array`来实现栈

由于有两个不同的实现，我们可以把`Stack`作为一个接口，两种方法分别实现它

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

想一下有什么功能?

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/dataStruct/arrayStack.png)

现在来实现`ArrayList`
```java
import array.Array;

import java.util.StringJoiner;

/**
 * @author BlackKnight
 * 这个栈是基于Array来构建的
 */
public class ArrayStack<E> implements Stack {
    Array<E> array;

    /**
     * 使用
     * @param capacity
     */
    public ArrayStack(int capacity) {
        array = new Array<>(capacity);
    }

    /**
     * 用户不知道用多少大小，先创建一个再说
     */
    public ArrayStack() {
        array = new Array<>();
    }

    /**
     * 因为使用Array来实现栈，所以特有了一个getCapacity的方法
     * @return
     */
    public int getCapacity(){
        return array.getCapacity();
    }


    @Override
    public int getSize() {
        return array.getSize();
    }

    @Override
    public boolean isEmpty() {
        return array.isEmpty();
    }

    @Override
    public void push(Object o) {
        array.addLast((E)o);
    }

    @Override
    public E pop() {
        return array.removeLast();
    }

    @Override
    public E peak() {
        return array.get(array.getSize());
    }

    @Override
    public String toString() {
        StringJoiner sj = new StringJoiner(",","Stack: [","] top");
        for(int i = 0 ; i < array.getSize() - 1 ; i++){
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
push(E e)               |     O(1)   | （均摊复杂度）
pop()                   |     O(1)   | （均摊复杂度）
peek()                  |     O(1)   |
getSize()               |     O(1)   |
isEmpty()               |     O(1)   |
```



# 参考资料
> 参考资料：
[来自慕课liuyubobobo的教程](https://coding.imooc.com/class/207.html)