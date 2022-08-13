---
title: Java二叉搜索树_手写一个BST
date: 2020-09-13 21:59:59
tags:
- 数据结构
- 二叉搜索树
- 树
- Java
categories:
- 数据结构
- 树
---


<center>
    引言：数据结构——二叉搜索树
</center>

<!--more-->

# 二叉搜索树

## 二叉树

树的基本结构，不过多叙述：

![image-20200907113521879](http://img.yesmylord.cn//image-20200907113521879.png)

- 具有天然的递归结构

  - 每个结点的左、右子树都是二叉树

- 满二叉树：除了叶子结点外，每个结点都有两个孩子

  例如这一棵就不是满二叉树：

  ![image-20200907113759356](http://img.yesmylord.cn//image-20200907113759356.png)

- 空也是二叉树



## 二分搜索树(Binary Search Tree)

- 首先是一棵二叉树
- **独有的特点**：每一个节点的值
  - 大于左子树所有结点的值
  - 小于右子树所有结点的值
- 每一棵子树也是二分搜索树
- 注意：存储的元素必须具有可比较性，存储一个int类的数据，当然可以比较大小，但是存储一个学生类、车类等实体类，我们必须规定它的比较方式，比如学生可以按年龄比较等等

### 二分搜索树代码

```java
package BinaryTree;

import java.util.LinkedList;
import java.util.Queue;
import java.util.Stack;

/**
 * @author 董文浩
 * @Date 2020/9/7 11:46
 * 二分搜索数
 */
//泛型必须满足可比较性，所以继承了Comparable接口
public class BinarySearchTree<E extends Comparable<E>> {
    /**
     * 内部节点类
     */
    private class Node {
        public E e;
        public Node left, right;

        public Node(E e) {
            this.e = e;
            left = null;
            right = null;
        }
    }

    /**
     * 根
     */
    private Node root;
    /**
     * 存储元素个数
     */
    private int size;

    /**
     * 初始化构造方法
     */
    public BinarySearchTree() {
        root = null;
        size = 0;
    }

    /**
     * 返回当前大小
     *
     * @return
     */
    public int size() {
        return size;
    }

    /**
     * 是否为空
     *
     * @return
     */
    public boolean isEmpty() {
        return size == 0;
    }

    public void add(E e) {
        root = add(root, e);
    }

    /**
     * 递归插入
     *
     * @param node
     * @param e
     */
    private void add1(Node node, E e) {
        //********* 终止条件 *************
        if (e.equals(node.e)) {
            return;
        } else if (e.compareTo(node.e) < 0 && node.left == null) {
            node.left = new Node(e);
            size++;
            return;
        } else if (e.compareTo(node.e) > 0 && node.right == null) {
            node.right = new Node(e);
            size++;
            return;
        }
        //*********  终止条件end ************

        //*********  递归 *************
        if (e.compareTo(node.e) > 0) {
            add1(node.right, e);
        } else {
            // 一定是 < 的 ，因为等于在最一开始就判断了
            add1(node.left, e);
        }
        //*********  递归end *************
    }

    /**
     * 递归插入：优化
     *
     * @param node
     * @param e
     * @return 返回插入新节点后二分搜索数的根节点
     */
    private Node add(Node node, E e) {
        //********* 终止条件 *************
        if (node == null) {
            size++;
            return new Node(e);
        }
        //*********  终止条件end ************

        //*********  递归 *************
        if (e.compareTo(node.e) > 0) {
            node.right = add(node.right, e);
        } else if (e.compareTo(node.e) < 0) {
            node.left = add(node.left, e);
        }
        return node;
        //*********  递归end *************
    }

    /**
     * 二分搜索数的查询操作
     *
     * @param e
     * @return
     */
    public boolean contains(E e) {
        return contains(root, e);
    }

    /**
     * 二分搜索树查询的递归操作
     *
     * @param node
     * @param e
     * @return
     */
    public boolean contains(Node node, E e) {
        if (node == null) {
            return false;
        }
        if (node.e.compareTo(e) == 0) {
            return true;
        } else if (node.e.compareTo(e) > 0) {
            return contains(node.left, e);
        } else {
            return contains(node.right, e);
        }
    }

    /**
     * 前序遍历二叉搜索树：根 -> 左 -> 右
     */
    public void preOrder() {
        preOrder(root);
    }

    private void preOrder(Node node) {
        if (node == null) {
            return;
        }
        System.out.println(node.e);
        preOrder(node.left);
        preOrder(node.right);
    }

    /**
     * 前序遍历二叉搜索树：非递归（深度优先遍历）
     * 使用栈来辅助存储
     * 要点：先将右孩子压入栈，然后将左孩子压入栈，因为出栈时是先进后出，所以前序遍历要先压入右孩子
     */
    public void preOrderNR() {
        Stack<Node> stack = new Stack<Node>();
        stack.push(root);
        while (!stack.isEmpty()){
            Node cur = stack.pop();
            System.out.println(cur.e);
            if(cur.right!=null) {
                // 先压入右孩子
                stack.push(cur.right);
            }
            if(cur.left!=null){
                stack.push(cur.left);
            }
        }
    }

    /**
     * 中序遍历二叉搜索树：左 -> 根 -> 右
     */
    public void inOrder() {
        inOrder(root);
    }

    private void inOrder(Node node) {
        if (node == null) {
            return;
        }
        inOrder(node.left);
        System.out.println(node.e);
        inOrder(node.right);
    }

    /**
     * 二叉搜索树的后序遍历：左 -> 右 -> 根
     */
    public void postOrder() {
        postOrder(root);
    }

    private void postOrder(Node node) {
        if (node == null) {
            return;
        }
        postOrder(node.left);
        postOrder(node.right);
        System.out.println(node.e);
    }

    /**
     * 二叉搜索树的层序遍历（广度优先遍历）
     * 使用队列来辅助记录孩子节点
     */
    public void levelOrder(){
        Queue<Node> queue = new LinkedList<>();
        queue.add(root);
        while (!queue.isEmpty()){
            Node cur = queue.remove();
            System.out.println(cur.e);
            if(cur.left!=null){
                queue.add(cur.left);
            }
            if(cur.right!=null){
                queue.add(cur.right);
            }
        }
    }


    /**
     * 返回以node为根的二分搜索树的最小值所在的节点
     * @return
     */
    private Node minimum(Node node){
        if(node.left == null){
            return node;
        }
        return minimum(node.left);
    }

    /**
     * 返回以node为根的二分搜索树的最大值所在的节点
     * @return
     */
    private Node maximum(Node node){
        if(node.right == null){
            return node;
        }
        return minimum(node.right);
    }

    /**
     * 寻找最小值
     * @return
     */
    public E minimum(){
        if(size==0){
            throw new IllegalArgumentException("BST is empty!");
        }
        return minimum(root).e;
    }

    /**
     * 寻找最大值
     * @return
     */
    public E maximum(){
        if(size==0){
            throw new IllegalArgumentException("BST is empty!");
        }
        return maximum(root).e;
    }

    /**
     * 删除最小值
     * @return
     */
    public E removeMin(){
        E ret = minimum();
        root = removeMin(root);
        return ret;
    }

    /**
     * 删除以node为根的最小结点
     * @param node
     * @return 删除节点后新的二分搜索树的根
     */
    private Node removeMin(Node node) {
        if(node.left ==null){
            Node rightNode = node.right;
            node.right = null;
            size--;
            return rightNode;
        }
        node.left = removeMin(node.left);
        return node;
    }

    /**
     * 删除最大值
     * @return
     */
    public E removeMax(){
        E ret = maximum();
        root = removeMax(root);
        return ret;
    }

    private Node removeMax(Node node) {
        if(node.right ==null){
            Node leftNode = node.left;
            node.left = null;
            size--;
            return leftNode;
        }
        node.right = removeMax(node.right);
        return node;
    }

    /**
     * 删除任意结点：1962年Hibbard提出的-Hibbard Deletion
     * @param e
     * @return
     */
    public void remove(E e){
        root = remove(root, e);
    }

    /**
     * 删除以node为根的二分搜索树中值为 e 的结点
     * @param node
     * @param e
     * @return
     */
    private Node remove(Node node, E e){
        if(node == null){
            return null;
        }
        if(e.compareTo(node.e) < 0){
            node.left = remove(node.left, e);
            return node;
        }else if(e.compareTo(node.e) > 0){
            node.right = remove(node.right, e);
            return node;
        }else{ // e 等于 node.e
            if(node.left == null){
                Node rightNode = node.right;
                node.right = null;
                size --;
                return rightNode;
            }
            if(node.left == null){
                Node leftNode = node.left;
                node.left = null;
                size --;
                return leftNode;
            }
            // 左右结点的左右子树均不为空
            // 找 “最近” 的结点，这里找了被删除结点的后驱结点，其实也可以选择前驱
            Node successor = minimum(node.right);
            successor.right = removeMin(node.right);
            successor.left = node.left;
            // 与不再有关系，将node左右结点置为空
            node.left = node.right = null;
            return successor;
        }
    }



    /**
     * 重写toString方法
     *
     * @return
     */
    @Override
    public String toString() {
        StringBuilder res = new StringBuilder();
        generateBSTString(root, 0, res);
        return res.toString();
    }

    /**
     * 生成以node为根节点，深度为depth的描述二叉树的字符串
     *
     * @param node
     * @param depth
     * @param res
     */
    private void generateBSTString(Node node, int depth, StringBuilder res) {
        if (node == null) {
            res.append(generateDepthString(depth) + "null\n");
            return;
        }
        res.append(generateDepthString(depth) + node.e + "\n");
        generateBSTString(node.left, depth + 1, res);
        generateBSTString(node.right, depth + 1, res);
    }

    /**
     * 打印深度，不同深度用不同的 “-” 来表示
     *
     * @param depth
     * @return
     */
    private String generateDepthString(int depth) {
        StringBuilder res = new StringBuilder();
        for (int i = 0; i < depth; i++) {
            res.append("--");
        }
        return res.toString();
    }

}

```



### 删除操作

> 1962年Hibbard提出的-Hibbard Deletion

删除任意结点操作中，删除一个任意结点的值，关键在于要糅合被删除结点的孩子结点，**关键是要找到被删除结点的“最近”的节点**，如图，绿色的结点就是被删除结点的最近的结点

![image-20200907185257518](http://img.yesmylord.cn//image-20200907185257518.png)

所以我们要做的事情就是用这个绿色的结点**替换**掉蓝色的结点，就可以了

(你可能发现了，其实也可以找53这个结点)

### 实际使用中：

- 前序遍历使用最广，因为前序遍历得到的是一个有序的串
- 后序遍历的思想可以使用在内存的释放过程中
- 前序遍历其实也叫作**深度优先遍历**
- 层序遍历也叫作**广度优先遍历**

### 注意：

当存入的数据是一个**有序**的数据时，我们的二叉搜索树会变成一个链表

为了解决这个问题，我们就要实现**平衡二叉树**（以后我们会说到平衡二叉树）