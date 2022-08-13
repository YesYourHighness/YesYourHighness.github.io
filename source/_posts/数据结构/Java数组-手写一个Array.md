---
title: Java数组_手写一个Array
date: 2020-03-02 21:22:55
tags:
- 数据结构
- 数组
- Java
categories:
- 数据结构
---

<center>
    引言：数据结构——数组
</center>

<!-- more -->


写在前面：
这个系列来重新学习数据结构，并且使用Java来实现他们


# 手写简单的`Array`

一个简单的Array我们需要什么功能？

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/dataStruct/array.png)

看代码不要着急，慢慢看完，重要的地方都有注释

代码如下：

```java
package array;

/**
 * @author BlackKnight
 * @apiNote 自己写一个Array数组
 */
public class Array<E>{
    /*
        如果你是先从容易的int数组来思考，进而转换为泛型时：
        注意要改的地方：
        1. 把数组的类型换为泛型
        2. 把泛型之间比较的方法换为equals
    */

    private E[] data;
    private int size;
    private final static int initialCapacity = 10;

    /**
     * 构造一个函数，初始化一个数组
     *
     * @param capacity 数组大小
     */
    public Array(int capacity) {
        //使用泛型时，我们并不能直接new一个泛型对象，但是可以new一个超类再强转为泛型
        data = (E[])new Object[capacity];
        size = 0;
    }

    /**
     * 用户如果也不知道需要多大，就先创建一个吧
     */
    public Array() {
        this(initialCapacity);
    }

    /**
     * 返回数组已存储的长度
     *
     * @return
     */
    public int getSize() {
        return size;
    }

    /**
     * 返回数组的容量
     *
     * @return
     */
    public int getCapacity() {
        return data.length;
    }

    /**
     * 数组是否为空
     *
     * @return
     */
    public boolean isEmpty() {
        return size == 0;
    }
    //-------------------------------------------------------

    /**
     * 在指定位置插入元素
     *
     * @param index
     * @param e
     */
    public void add(int index, E e) {
        //排除非法index
        if (index < 0 || index > size) {
            throw new IllegalArgumentException("需要index<0 || index > size");
        }

        /*  size表示存储大小，data.length代表数组的大小
        假如我们的存了3个数，数组的大小是10
        那么 size == 3  && data.length ==10
        随着我们不断的添加元素 size慢慢增加，会出现一个问题
        size 大小范围应该在 [0,data.length-1]
        当size == data.length 时， 就会出现数组越界异常
        此时我们需要扩容
        */
        if (size == data.length) {
            /*为了提高效率，如果固定提高第一个常数的话
            当数组越大我们需要扩容的次数越多，这显然不是我们想要的
            所以每次扩容两倍
            */
            resize(2 * data.length);
        }

        /*  主要的难点在于插入的同时要移动元素
                 index：0 1 2 3 4 5 6
            比如我现在有 1 2 3 4 6 7 8
            现在我要插入5到4的位置，我就需要移动 4 ~ size个元素
        */
        for (int i = size - 1; i >= index; i--) {
            data[i + 1] = data[i];
        }
        data[index] = e;
        size ++;
    }

    /**
     * 改变容器大小，这个方法是不需要被用户知道的，所以private
     * @param newCapacity
     */
    private void resize(int newCapacity){
        E[] newData = (E[]) new Object[newCapacity];
        for(int i = 0; i < size ; i++){
            newData[i] = data[i];
        }
        data = newData;
    }
    /**
     * 向数组的末尾添加内添加
     * @param e 元素值
     */
    public void addLast(E e) {
        /*
        if (size == data.length) {
            throw new IllegalArgumentException("数组越界");
        }
        data[size] = e;
        size++;
        //我也可以写成 data[size++] = e;
        */
        //-------------------------------------
        // 但其实，我们完全可以复用add()
        add(size,e);
    }

    /**
     * 向数组头添加
     * @param e
     */
    public void addFirst(E e) {
        add(0,e);
    }
    //-------------------------------------------------------

    @Override
    public String toString(){
        StringBuilder res = new StringBuilder();
        res.append(String.format("Array: size = %d , capacity =%d\n",size,data.length));
        res.append("[");
        for (int i = 0; i < size; i++) {
            res.append(data[i]);
            if(i!=size-1){
                res.append(", ");
            }
        }
        res.append("]");
        return res.toString();
    }

    /**
     * 获取元素
     * @param index
     * @return
     */
    public E get(int index){
        /*
        设计get方法的优点：
        1. 可以判断用户输入的index是否合法
        2. 防止用户可以查询我们未曾使用的数组空间
         */
        if(index<0||index>=size){
            throw new IllegalArgumentException("非法index");
        }
        return data[index];
    }

    /**
     * 更新指定位置的元素
     * @param index
     * @param e
     */
    public void set(int index, E e){
        if(index<0||index>=size){
            throw new IllegalArgumentException("非法index");
        }
        data[index] = e;
    }
    //-------------------------------------------------------

    /**
     * 是否包含某个数
     * @param e
     * @return
     */
    public boolean contains(E e){
        for (int i = 0; i < size; i++) {
            if (data[i].equals(e)){
                return true;
            }
        }
        return false;
    }

    /**
     * 查找某个数并返回下标
     * @param e
     * @return 返回-1代表没有找到这个数
     */
    public int find(E e){
        for (int i = 0; i < size; i++) {
            if (data[i].equals(e)){
                return i;
            }
        }
        return -1;
    }

    /**
     * 删除指定位置的元素
     * @return 返回被删除元素
     */
    public E remove(int index){
        if(index<0||index>=size){
            throw new IllegalArgumentException("非法index");
        }
        E ret = data[index];
        for(int i = index + 1 ; i < size; i++){
            data[i-1]=(data[i]);
        }
        size--;
        /* data[size] = null 说明：
        原本int[]数组的时候，删除一个元素的最后
        size依然指向了data[size]这个单元，但是用户是访问不到的，所以我们并没有处理
        但是使用泛型之后，每个单元都会存储对象的引用
        如果我们不将data[size]清空，虽然用户依然访问不到
        但是因为data[size]中存有引用，这样就不会被自动的垃圾回收掉
        这样的对象专业术语叫 loitering objects （游荡对象）
        所以我们需要自己处理
        */
        data[size] = null;

        /*
        删除元素到一定程度时。缩减容量
         */
            /*
            这里判断 size == data.length / 2也可以
            但是判断 size == data.length / 4会更好一点
            可以先思考一下，末尾解答
            */
        if(size == data.length / 4 && data.length / 2 != 0){
            resize(data.length/2) ;
        }
        return ret;
    }

    /**
     * 删除末尾元素
     * @return
     */
    public E removeLast(){
        return remove(size-1);
    }

    /**
     * 删除第一个元素
     * @return
     */
    public E removeFirst(){
        return remove(0);
    }

    /**
     * 删除与值相同的第一个元素
     * @param e
     */
    public void removeElement(E e){
        int index = find(e);
        if(index!=-1){
            remove(index);
        }
    }

}


```

# 复杂度分析
![image](https://github.com/YesYourHighness/MyPicStore/raw/master/dataStruct/array.png)

回看我们所写的方法，我们来分析他们的时间复杂度

## 增
```
方法名                 |  最坏 | 最好 | 平均   |
------------------------------------------------
addLast(E e)           |  O(1) | O(1) | O(1)   |
addFirst(E e)          |  O(n) | O(n) | O(n)   |
add(int index, E e)    |  O(n) | O(1) | O(n/2) |
resize(int newCapacity)|  O(n) | O(n) | O(n)   |
------------------------------------------------
注意：O(n/2) = O(n)
```

通常我们认为的时间复杂度其实是最坏情况下的复杂度，
所以由上表分析，增方法的全部方法的时间复杂度都是为的O(n)（`addLast()`方法由于有`resize()`方法的存在，所以它的复杂度也是O(n)）

但是这样会有一些问题，因为`resize`方法只有当超出容量了我们才会调用一次
，此时我们可以进行**均摊复杂度**(amortized time complexity)分析
```
假设capacity为n
我们向其中添加的方法是addLast，这个方法时间复杂度为O(1)
当满出时，需要进行resize方法，这个方法时间复杂度为O(n)
当我们执行到第 n+1 次addLast()方法时，我们才会调用resize()方法
所以我们共操作 2n+1 次
用 2n+1 / n 差不多为2
所以这时的时间复杂度为O(1)
```
这样分析，其实`addLast()`方法平均下来的时间复杂度和n没有关系，它一直是O(n)

## 删
```
方法名                  |  最坏 | 最好 | 平均   |
-------------------------------------------------
removeLast()            |  O(1) | O(1) | O(1)   |
removeFirst()           |  O(n) | O(n) | O(n)   |
remove(int index)       |  O(n) | O(1) | O(n/2) |
resize(int newCapacity) |  O(n) | O(n) | O(n)   |
-------------------------------------------------
注意：O(n/2) = O(n)
```
同理，`removeLast()`是O(1)，其他方法依然为O(n)

## 查
```java
方法名                  |  最坏 | 最好 | 平均   |
-------------------------------------------------
get(int index)          |  O(1) | O(1) | O(1)   |
contains(E e)           |  O(1) | O(n) | O(n/2) |
find(E e)               |  O(n) | O(1) | O(n/2) |
-------------------------------------------------
注意：O(n/2) = O(n)
```

## 改
```
方法名                  |  最坏 | 最好 | 平均   |
-------------------------------------------------
set(int index)          |  O(1) | O(1) | O(1)   |
```

# 复杂度震荡
在代码中我们遗留了一个问题
```java
/*
删除元素到一定程度时。缩减容量
 */
    /*
    这里判断 size == data.length / 2也可以
    但是判断 size == data.length / 4会更好一点
    可以先思考一下，末尾解答
    */
if(size == data.length / 4 && data.length / 2 != 0){
    resize(data.length/2) ;
}
return ret;
```
为什么下方的代码会更好一些呢？
我们来思考一个问题

假设我们使用上方的代码`size == data.length / 2`，假设`Capacity`设置为10，现在我们已经有了十个数字，将这个数组已经撑满

现在我们来执行以下操作
1. 调用一次`addLast()`方法，它内部会执行`resize()`方法，`capacity`会变为20
2. 再调用一次`removeLast()`方法，它内部会执行`resize()`方法，`capacity`又会变为10
3. 再调用一次`addLast()`方法，它内部会执行`resize()`方法，`capacity`会变为20
4. 依次下去....

我们发现，我们原先论证的O(1)的复杂度，现在却变成了O(n)，这就叫做**复杂度震荡**

原因在于：判断`size == data.length / 2`过于着急(Eager)

解决：我们需要懒一点(Lazy)，换成下方的代码`size == data.length / 4 ;`，就解决了这个问题。


# 参考资料
> 参考资料：
[来自慕课liuyubobobo的教程](https://coding.imooc.com/class/207.html)