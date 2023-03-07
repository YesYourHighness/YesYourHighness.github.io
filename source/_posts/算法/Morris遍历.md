---
title: Morris中序遍历
date: 2023-01-13 15:37:01
tags:
- Java
- 算法
categories:
- 算法
---

<center>
引言： 新年的第一篇blog，学一下Morris算法吧
</center>

<!-- more -->

# Morris中序遍历

## Morris基本了解

**提出**：Morris原本是用来解决，当树的深度很深的时候，遍历树需要一个很大的栈，Morris可以省去这一部分空间

**原理**：假设`cur`代表当前节点，如果`cur`没有左孩子，那么就直接转到右孩子；如果`cur`有左孩子，那么让`cur`的前驱节点（**前驱节点就是其左孩子的最右孩子**）的`right`指针指向当前的`cur`，这样做的好处是我们无需再挨个出栈，而是一步到位，省去了栈空间

## Morris算法

假设我们有这样一颗树，

```java
/** 
*               6
*             /   \
*            2     8
*           / \    / \
*          1   4   7  9
*             /\
*            3  5
*/
```

Morris基本是中序遍历（左根右顺序），区别在于，遍历到1时，会将1的right指针指向2，然后返回2；同理`2->6`，`3->4`、`5->6`、`7->8`

在第二次遍历到1、3、5、7的位置上时，会将`right`再置为`null`，让树保持原状

下面给出Morris中序遍历的Java实现：

```java
// Morris 中序遍历
public static void morrisInOrderTraverse(TreeNode root) {
    TreeNode cur = root;
    for (; cur != null; ) {
        // 如果该节点没有左孩子，那么就将该节点直接指向右孩子
        if (cur.left == null) {
            System.out.println(cur.val);
            cur = cur.right;
        } else {
            // 寻找前驱节点
            TreeNode pre = cur.left;
            for (; pre.right != null && pre.right != cur; pre = pre.right) ;
            // 判断是否已经设置过线索
            if (pre.right == cur) {
                // 说明设置过线索，开始处理右子树
                pre.right = null; // 将原本的线索擦去
                System.out.println(cur.val + "线索回归");
                cur = cur.right;
            } else {
                // 将前驱节点的right指向cur
                pre.right = cur;
                System.out.println("设置"+ cur.val +"线索，其前驱为"+ pre.val);
                cur = cur.left;
            }
        }
    }
}
```

遍历结果为：

```java
/** 
*               6
*             /   \
*            2     8
*           / \    / \
*          1   4   7  9
*             /\
*            3  5
*/
1
2线索回归
设置4线索，其前驱为3
3
4线索回归
5
6线索回归
设置8线索，其前驱为7
7
8线索回归
9
```

## LeetCode-501. 二叉搜索树中的众数

[501.二叉搜索树中的众数](https://leetcode.cn/problems/find-mode-in-binary-search-tree/description/)

- 首先因为一棵二叉搜索树的中序遍历序列是一个非递减的有序序列，所以**相同的数字一定是连续出现**的；

- 其次为了避免使用Map，保证O1的空间；

以上两个原因保证此题可以是Morris中序遍历来解决

```java
class Solution {
    int base, count, maxCount;
    List<Integer> answer = new ArrayList<Integer>(); 
    // answer列表：众数可能不止一个，存放多个众数

    public int[] findMode(TreeNode root) {
        TreeNode cur = root, pre = null;
        while (cur != null) {
            if (cur.left == null) {
                update(cur.val);
                cur = cur.right;
                continue;
            }
            pre = cur.left;
            while (pre.right != null && pre.right != cur) pre = pre.right;
            
            if (pre.right == null) {
                pre.right = cur;
                cur = cur.left;
            } else {
                pre.right = null;
                update(cur.val);
                cur = cur.right;
            }
        }

        int[] mode = new int[answer.size()];
        for (int i = 0; i < answer.size(); ++i) mode[i] = answer.get(i);
        return mode;
    }

    // eg：1,2,2,2,3,4,5,6,6,7 相同就计数，不同就更新
    public void update(int x) {
        if (x == base) {
            count ++;
        } else {
            count = 1;
            base = x;
        }
        if (count == maxCount) answer.add(base); // 众数可能不止一个
        if (count > maxCount) {
            maxCount = count;
            answer.clear();
            answer.add(base);
        }
    }
}
```



## LeetCode-99. 恢复二叉搜索树

[99. 恢复二叉搜索树](https://leetcode.cn/problems/recover-binary-search-tree/)

多使用一个额外的指针`preNode`

```java
public void recoverTree(TreeNode root) {
    TreeNode cur = root, pre = null, preNode = null;
    TreeNode a = null, b = null;
    while (cur != null) {
        if (cur.left == null) {
            // 新增code开始
            if(preNode != null && cur.val < preNode.val){
                b = cur;
                if(a == null) a = preNode;
            }
            preNode = cur;
            // 新增code结束
            cur = cur.right;
            continue;
        }
        pre = cur.left;
        while (pre.right != null && pre.right != cur) pre = pre.right;

        if (pre.right == null) {
            pre.right = cur;
            cur = cur.left;
        } else {
            // 新增code开始
            if(preNode != null && cur.val < preNode.val){
                b = cur;
                if(a == null) a = preNode;
            }
            preNode = cur;
            // 新增code结束
            pre.right = null;
            cur = cur.right;
        }
    }
    change(a, b);
}
// 交换两个元素
public void change(TreeNode pre,TreeNode cur){
    int temp = pre.val;
    pre.val = cur.val;
    cur.val = temp;
}

```









