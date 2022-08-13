---
title: Tire字典树
date: 2020-09-25 20:35:59
tags:
- 数据结构
- Tire字典树
categories:
- 数据结构
- Tire字典树
---


<center>
    引言：数据结构——Tire字典树
</center>

<!-- more -->

# Trie 字典树

## Trie

> 一个便于搜索的多叉树。
>
> 我们学习了树结构实现映射，它的时间复杂度是O(log n)，如果有两百万个条目，大约会花费20
>
> 但是Tire查询每个条目的时间复杂度和字典中一共有多少条目无关，取决于查询单词的长度O(w)



一棵Trie就像是这样

<img src="http://img.yesmylord.cn//image-20200926093554457.png" alt="image-20200926093554457" style="width:50%;" />

那么这样的一棵树，它的节点是如何定义的？

```java
class Node{
    char c;
    Node next[];

    public Node(char c) {
        this.c = c;
        this.next = new Node[26];
    }
}
```

假如我们的业务是实现单词的存储，那么应该就是这样，每一个结点可以存储26个字母。

但是假如我们的业务是存储网址信息等等，我们会扩展到更多更多，所以我们可以使用一个`Map`集合来充当这里的数据

```java
class Node{
    Map<Character, Node> next;
}
```

但是，如果要存储单词的话，我们会遇到一个问题，就是假设存储`cat`和`category`，两个词前面都是cat，那么我们如何存储呢？

我们可以再给Node加一个字段，就是`isWord`

```java
class Node{
    boolean isWord;
    Map<Character, Node> next;
}
```





全部代码如下：

```java
package Trie;

import java.util.Map;
import java.util.TreeMap;

/**
 * @author 董文浩
 * @Date 2020/9/26 9:37
 * 字典树Trie
 */
public class Trie{
    private class Node{
        boolean isWord;
        Map<Character, Node> next;

        public Node(boolean isWord) {
            this.isWord = isWord;
            this.next = new TreeMap<>();
        }
        public Node(){
            this(false);
        }
    }
    private Node root;
    private int size;

    public Trie(){
        root = new Node();
        size =0;
    }
    public int getSize(){
        return size;
    }
    public void add(String word){
        Node cur = root;
        char[] chars = word.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            if(cur.next.get(chars[i]) == null){
                cur.next.put(chars[i], new Node());
            }
            cur = cur.next.get(chars[i]);
        }
        // 如果之前这不是一个单词
        if(!cur.isWord){
            cur.isWord = true;
            size ++;
        }
    }

    /** 查询是否有一个单词
     * @param word
     * @return
     */
    public boolean contains(String word){
        Node cur = root;
        char[] chars = word.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            if(cur.next.get(chars[i]) == null){
                return false;
            }
            cur = cur.next.get(chars[i]);
        }
        return cur.isWord;
    }

    /** 查询是否在Tire中以prefix为前缀的单词
     * @param prefix
     * @return
     */
    public boolean isPrefix(String prefix){
        Node cur = root;
        char[] chars = prefix.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            if(cur.next.get(chars[i]) == null){
                return false;
            }
            cur = cur.next.get(chars[i]);
        }
        return true;
    }

}

```







