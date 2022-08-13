---
title: 平衡二叉树-AVL树
date: 2020-10-09 19:37:59
tags:
- 数据结构
- 平衡二叉树
- AVL树
categories:
- 数据结构
- 平衡二叉树
- AVL树
---


<center>
    引言：数据结构——平衡二叉树AVL树
</center>

<!-- more -->


# 平衡二叉树——AVL树

> 平衡二叉树：**任意节点的左子树和右子树的高度差不能超过1**



## 通过什么判断一棵树不再平衡呢？

当然是它的**左右子树差超过1的时候**，我们把这个差值叫做**平衡因子**



如图：

<img src="http://img.yesmylord.cn//image-20201007210019380.png" alt="image-20201007210019380" style="width:70%;" />

树上黑色的数字代表该节点的高度：

​	例如2那个叶子节点，它的高度是1

​	例如8那个节点，它的左子树高度是3，右子树高度是1，那么他的高度就是**较大的那个树的高度再加1**，就是4



树上的蓝色数字代表这个结点的平衡因子：

​	例如2那个叶子结点，它的左右孩子都是null，所以它的平衡因子是`0 - 0 = 0`

​	例如8那个叶子结点，它的左子树高度为3，右子树高度为1，所以它的平衡因子是`3 - 1 = 2`，超过了1，所以该树从这个结点开始不再平衡



## 维护平衡的时机是什么时候呢？

当我们从一个根节点开始不断插入值的时候，这棵树会不断的生长，如果在插入一个值的时候，平衡被破坏了，那么不管如何，**将这颗树改为平衡状态一定是在这个叶子节点的父节点路径上**

![image-20201008194117663](http://img.yesmylord.cn//image-20201008194117663.png)





## AVL树

一种最早的也是最经典的平衡二叉树，由G.M.**A**delson-**V**elsky和E.M.**L**andis两位俄罗斯的科学家找出了 **左旋和右旋** 两种操作来实现树的平衡。



对于平衡二叉树，有两种不平衡的状况（如图）：

![image-20201009173504163](http://img.yesmylord.cn//image-20201009173504163.png)

以上的两种状况是：

- LL 与 RR
- LR 与 RL



### LL与RR

我们先来看**右旋**（同理可以得到左旋的代码）如图：

![image-20201009173434963](http://img.yesmylord.cn//image-20201009173434963.png)

在右旋完成后，我们需要去更新height（平衡因子）两个值，但是留心观察我们会发现，其实**只需要更新x和y**，而且要先更新y再更新x（因为对于T3来说根本没有发生变化，先更新y的原因是因为更新x需要y的值，所以要先更新y）



右旋代码如下：

```java
    // 对节点y进行向右旋转操作，返回旋转后新的根节点x
    //        y                              x
    //       / \                           /   \
    //      x   T4     向右旋转 (y)        z     y
    //     / \       - - - - - - - ->    / \   / \
    //    z   T3                       T1  T2 T3 T4
    //   / \
    // T1   T2
    private Node rightRotate(Node y) {
        Node x = y.left;
        Node T3 = x.right;

        // 向右旋转过程
        x.right = y;
        y.left = T3;

        // 更新height
        y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;
        x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;

        return x;
    }
```

我们可以站在x的角度，理解“右旋”



对于这种情况，只需先转换为LL与RR情况，在做改变即可：

![image-20201009173447206](http://img.yesmylord.cn//image-20201009173447206.png)







## 完整代码

```java
package BinaryTree.AVLTree;

import java.util.ArrayList;

/**
 * @author 董文浩
 * @Date 2020/10/7 21:10
 */
public class AVLTree<K extends Comparable<K>,V> {
    private class Node {
        public K key;
        public V value;
        public Node left, right;
        public int height; // 当前所处高度值

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
            left = null;
            right = null;
            height = 1; //默认高度为1
        }
    }

    private Node root;
    private int size;

    public AVLTree() {
        root = null;
        size = 0;
    }

    // * 获得结点的高度
    private int getHeight(Node node){
        if(node==null) {
            return 0;
        }
        return node.height;
    }
    // * 计算结点的平衡因子

    private int getBalanceFactor(Node node){
        if(node == null) {
            return 0;
        }
        return getHeight(node.left) - getHeight(node.right);
    }

    public void add(K key, V value) {
        root = add(root, key, value);
    }

    // * 判断该二叉树是否是一棵二分搜索树
    public boolean isBST(){
        ArrayList<K> keys = new ArrayList<>();
        inOrder(root,keys);
        // 中序遍历后的二分搜索树应该是一个有序的数列
        for (int i = 1; i < keys.size(); i++) {
            if(keys.get(i-1).compareTo(keys.get(i))>0){
                return false;
            }
        }
        return true;
    }

    // * 中序遍历这棵树
    private void inOrder(Node node, ArrayList<K> keys) {
        if(node == null){
            return;
        }
        inOrder(node.left,keys);
        keys.add(node.key);
        inOrder(node.right,keys);
    }

    // * 判断该二叉树是否是一个平衡二叉树
    public boolean isBalanced(){
        return isBalanced(root);
    }
    // * 递归判断是否是平衡二叉树
    public boolean isBalanced(Node node){
        if(node == null) {
            return true;
        }
        int balanceFactor = getBalanceFactor(node);
        if(Math.abs(balanceFactor) > 1)
            return false;
        return isBalanced(node.left) && isBalanced(node.right);
    }


    // * 每次增加结点改变高度值
    private Node add(Node node, K key, V value) {
        //********* 终止条件 *************
        if (node == null) {
            size++;
            return new Node(key, value);
        }
        //*********  终止条件end ************

        //*********  递归 *************
        if(key.compareTo(node.key) < 0) {
            node.left = add(node.left, key, value);
        } else if(key.compareTo(node.key) > 0) {
            node.right = add(node.right, key, value);
        } else // key.compareTo(node.key) == 0
        {
            node.value = value;
        }

        // * 对当前的node值更新它的height：它的高度是左右子树更高那个加1
        node.height = 1 + Math.max(getHeight(node.left),getHeight(node.right));

        // * 计算结点的平衡因子
        int balanceFactor = getBalanceFactor(node);
        // * 如果平衡因子(有可能为负数)超过了1 ，那么我们就需要进行自平衡了
        // * 如果不平衡，那么判断这个结点的两个孩子：如果左孩子平衡值大于等于0，那么需要右旋；如果右孩子平衡值大于等于0，那么需要左旋
        // LL
        if (balanceFactor > 1 && getBalanceFactor(node.left) >= 0) {
            return rightRotate(node);
        }
        // RR
        if (balanceFactor < -1 && getBalanceFactor(node.right) <= 0) {
            return leftRotate(node);
        }
        // LR
        if (balanceFactor > 1 && getBalanceFactor(node.left) < 0) {
            node.left = leftRotate(node.left);
            return rightRotate(node);
        }
        // RL
        if (balanceFactor < -1 && getBalanceFactor(node.right) > 0) {
            node.right = rightRotate(node.right);
            return leftRotate(node);
        }
        return node;
        //*********  递归end *************
    }

    // 对节点y进行向右旋转操作，返回旋转后新的根节点x
    //        y                              x
    //       / \                           /   \
    //      x   T4     向右旋转 (y)        z     y
    //     / \       - - - - - - - ->    / \   / \
    //    z   T3                       T1  T2 T3 T4
    //   / \
    // T1   T2
    private Node rightRotate(Node y) {
        Node x = y.left;
        Node T3 = x.right;

        // 向右旋转过程
        x.right = y;
        y.left = T3;

        // 更新height
        y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;
        x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;

        return x;
    }

    // 对节点y进行向左旋转操作，返回旋转后新的根节点x
    //    y                             x
    //  /  \                          /   \
    // T1   x      向左旋转 (y)       y     z
    //     / \   - - - - - - - ->   / \   / \
    //   T2  z                     T1 T2 T3 T4
    //      / \
    //     T3 T4
    private Node leftRotate(Node y) {
        Node x = y.right;
        Node T2 = x.left;

        // 向左旋转过程
        x.left = y;
        y.right = T2;

        // 更新height
        y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;
        x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;

        return x;
    }


    private Node removeMin(Node node) {
        if (node.left == null) {
            Node rightNode = node.right;
            node.right = null;
            size--;
            return rightNode;
        }
        node.left = removeMin(node.left);
        return node;
    }

    /**
     * 寻找最小值
     *
     * @return
     */
    public V minimum() {
        if (size == 0) {
            throw new IllegalArgumentException("BST is empty!");
        }
        return minimum(root).value;
    }

    /**
     * 返回以node为根的二分搜索树的最小值所在的节点
     *
     * @return
     */
    private Node minimum(Node node) {
        if (node.left == null) {
            return node;
        }
        return minimum(node.left);
    }

    public V remove(K key) {
        Node node = getNode(root, key);
        if (node != null) {
            root = remove(root, key);
        }

        return null;
    }

    private Node remove(Node node, K key){

        if( node == null ) {
            return null;
        }

        Node retNode;
        if( key.compareTo(node.key) < 0 ){
            node.left = remove(node.left , key);
            // return node;
            retNode = node;
        }
        else if(key.compareTo(node.key) > 0 ){
            node.right = remove(node.right, key);
            // return node;
            retNode = node;
        }
        else{   // key.compareTo(node.key) == 0

            // 待删除节点左子树为空的情况
            if(node.left == null){
                Node rightNode = node.right;
                node.right = null;
                size --;
                // return rightNode;
                retNode = rightNode;
            }

            // 待删除节点右子树为空的情况
            else if(node.right == null){
                Node leftNode = node.left;
                node.left = null;
                size --;
                // return leftNode;
                retNode = leftNode;
            }

            // 待删除节点左右子树均不为空的情况
            else{
                // 找到比待删除节点大的最小节点, 即待删除节点右子树的最小节点
                // 用这个节点顶替待删除节点的位置
                Node successor = minimum(node.right);
                //successor.right = removeMin(node.right);
                successor.right = remove(node.right, successor.key);
                successor.left = node.left;

                node.left = node.right = null;

                // return successor;
                retNode = successor;
            }
        }

        if(retNode == null) {
            return null;
        }

        // 更新height
        retNode.height = 1 + Math.max(getHeight(retNode.left), getHeight(retNode.right));

        // 计算平衡因子
        int balanceFactor = getBalanceFactor(retNode);

        // 平衡维护
        // LL
        if (balanceFactor > 1 && getBalanceFactor(retNode.left) >= 0)
            return rightRotate(retNode);

        // RR
        if (balanceFactor < -1 && getBalanceFactor(retNode.right) <= 0)
            return leftRotate(retNode);

        // LR
        if (balanceFactor > 1 && getBalanceFactor(retNode.left) < 0) {
            retNode.left = leftRotate(retNode.left);
            return rightRotate(retNode);
        }

        // RL
        if (balanceFactor < -1 && getBalanceFactor(retNode.right) > 0) {
            retNode.right = rightRotate(retNode.right);
            return leftRotate(retNode);
        }

        return retNode;
    }

    // 返回以node为根节点的二分搜索树中，key所在的节点
    private Node getNode(Node node, K key){

        if(node == null) {
            return null;
        }

        if(key.equals(node.key)) {
            return node;
        } else if(key.compareTo(node.key) < 0) {
            return getNode(node.left, key);
        } else // if(key.compareTo(node.key) > 0)
        {
            return getNode(node.right, key);
        }
    }

    public boolean contains(K key) {
        return getNode(root, key) != null;
    }

    public V get(K key) {
        Node node = getNode(root, key);
        return node == null ? null : node.value;
    }

    public void set(K key, V newValue){
        Node node = getNode(root, key);
        if(node == null) {
            throw new IllegalArgumentException(key + " doesn't exist!");
        }

        node.value = newValue;
    }

    public int getSize() {
        return size;
    }

    public boolean isEmpty() {
        return size == 0;
    }


}

```

