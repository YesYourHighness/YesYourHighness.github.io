---
title: 映射-Java实现
date: 2020-09-25 11:02:33
tags:
- 数据结构
- Map
- 映射

categories:
- 数据结构
- Map
---

<center>
    引言：数据结构——映射Map
</center>

<!--more-->


# 映射

## Map

<img src="http://img.yesmylord.cn//image-20200908175711576.png" alt="image-20200908175711576" style="width:50%;" />

在Java语言中，映射就是一一映射，类似于函数，一个x对应一个y，一个y可以对应多个x

- 使用 **键key** 来快速的找到 **值value**



这里使用二叉搜索树与链表实现映射

```java
/**
 * @Date 2020/9/8 18:00
 * Map映射
 */
public interface Map<K, V> {
    void add(K key, V value);

    V remove(K key);

    boolean contains(K key);
    
    V get(K key);

    void set(K key, V newValue);

    int getSize();

    boolean isEmpty();
}

```



## 链表映射LinkedListMap

链表实现Map

```java

/**
 * @Date 2020/9/8 18:00
 * 链表实现映射map
 */
public class LinkedListMap<K, V> implements Map<K, V> {
    private class Node {
        public K key;
        public V value;
        public Node next;

        public Node(K key, V value, Node next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public Node(K key, V value) {
            this(key, value, null);
        }

        public Node() {
            this(null, null, null);
        }

        @Override
        public String toString() {
            return "Node{" +
                    "key=" + key +
                    ", value=" + value +
                    ", next=" + next +
                    '}';
        }
    }

    private Node dummyHead;
    private int size;

    public LinkedListMap() {
        dummyHead = new Node();
        size = 0;
    }

    /**
     * 辅助方法
     * 通过key获得对应的node
     *
     * @param key
     * @return
     */
    private Node getNode(K key) {
        Node cur = dummyHead.next;
        while (cur != null) {
            if (cur.key.equals(key)) {
                return cur;
            }
            cur = cur.next;
        }
        return null;
    }

    @Override
    public void add(K key, V value) {
        Node node = getNode(key);
        if (node == null) {
            dummyHead.next = new Node(key, value, dummyHead.next);
            size++;
        } else {
            node.value = value;
        }
    }

    @Override
    public V remove(K key) {
        Node prev = dummyHead;
        while (prev.next != null){
            if (prev.next.key.equals(key)){
                break;
            }
            prev = prev.next;
        }
        if(prev.next!=null){
            Node delNode = prev.next;
            prev.next = delNode.next;
            delNode.next = null;
            size --;
            return delNode.value;
        }
        return null;
    }

    @Override
    public boolean contains(K key) {
        return getNode(key) != null;
    }

    @Override
    public V get(K key) {
        Node node = getNode(key);
        if(node == null){
            throw new IllegalArgumentException(key + "doesn't exist!");
        }
        return node.value;
    }

    @Override
    public void set(K key, V newValue) {
        Node node = getNode(key);
        if(node==null){
            throw new IllegalArgumentException(key + "doesn't exist!");
        }
        node.value = newValue;
    }

    @Override
    public int getSize() {
        return size;
    }

    @Override
    public boolean isEmpty() {
        return size == 0;
    }
}

```



## 二叉搜索树映射BstMap

二叉搜索树实现Map

```java
package map;

/**
 * @author 董文浩
 * @Date 2020/9/8 18:00
 */
public class BstMap<K extends Comparable<K>, V> implements Map<K, V> {
    private class Node {
        public K key;
        public V value;
        public Node left, right;

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
            left = null;
            right = null;
        }
    }

    private Node root;
    private int size;

    public BstMap() {
        root = null;
        size = 0;
    }

    @Override
    public void add(K key, V value) {
        root = add(root, key, value);
    }

    private Node add(Node node, K key, V value) {
        //********* 终止条件 *************
        if (node == null) {
            size++;
            return new Node(key, value);
        }
        //*********  终止条件end ************

        //*********  递归 *************
        if (key.compareTo(node.key) > 0) {
            node.right = add(node.right, key, value);
        } else if (key.compareTo(node.key) < 0) {
            node.left = add(node.left, key, value);
        } else {
            node.value = value;
        }
        return node;
        //*********  递归end *************
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

    @Override
    public V remove(K key) {
        Node node = getNode(root, key);
        if (node != null) {
            root = remove(root, key);
        }

        return null;
    }

    private Node remove(Node node, K k) {
        if (node == null) {
            return null;
        }
        if (k.compareTo(node.key) < 0) {
            node.left = remove(node.left, k);
            return node;
        } else if (k.compareTo(node.key) > 0) {
            node.right = remove(node.right, k);
            return node;
        } else { // e 等于 node.e
            if (node.left == null) {
                Node rightNode = node.right;
                node.right = null;
                size--;
                return rightNode;
            }
            if (node.left == null) {
                Node leftNode = node.left;
                node.left = null;
                size--;
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

    // 返回以node为根的二分搜索树中，key所在的节点
    private Node getNode(Node node, K key) {
        if (node == null) {
            return null;
        }
        if (key.compareTo(node.key) == 0) {
            return node;
        } else if (key.compareTo(node.key) < 0) {
            return getNode(node.left, key);
        } else {
            return getNode(node.right, key);
        }

    }

    @Override
    public boolean contains(K key) {
        return getNode(root, key) != null;
    }

    @Override
    public V get(K key) {
        return getNode(root, key) == null ? null : getNode(root, key).value;
    }

    @Override
    public void set(K key, V newValue) {
        Node node = getNode(root, key);
        if (node == null) {
            throw new IllegalArgumentException(key + "doesn't exist");
        }
        node.value = newValue;
    }

    @Override
    public int getSize() {
        return size;
    }

    @Override
    public boolean isEmpty() {
        return size == 0;
    }
}

```

