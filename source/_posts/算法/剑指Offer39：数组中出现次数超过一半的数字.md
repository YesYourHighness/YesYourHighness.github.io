---
title: 出现超过数组一半的数
date: 2020-08-16 21:31:01
tags:
- Java
- 算法
categories:
- 算法
---


<center>
引言： 

摩尔投票法

</center>

<!-- more -->


# 剑指Offer39：数组中出现次数超过一半的数字

> 题意：
>
> 
>
> 数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。
>
> 你可以假设数组是非空的，并且给定的数组总是存在多数元素。
>
> 
>
> 示例 1:
>
> 输入: [1, 2, 3, 2, 2, 2, 5, 4, 2]
> 输出: 2

 

## 解法1：HashMap

很容易想到HashMap很容易来解决这种问题，只需要出现一次+1即可

```java
    // 法一：憨憨HashMap法 可以是肯定可以，就是慢
     public static int majorityElement1(int[] nums) {
         HashMap<Integer, Integer> map = new HashMap<>();
         int x = nums[0];
         int turn = 1;
         for (int num : nums) {
             if(map.get(num)==null){
                 map.put(num,1);
             }else {
                 Integer a = map.get(num);
                 map.put(num,++a);
                 if(a > turn){
                     x = num;
                     turn = a;
                 }
             }
         }
         return x;
     }
```



## 解法2：摩尔投票法

这个方法也是为了写这篇blog的原因。



可以结合官网的解析[LeetCode](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/solution/mian-shi-ti-39-shu-zu-zhong-chu-xian-ci-shu-chao-3/)

 

> **摩尔投票法**：
>
> 记投票数： 众数会 +1，非众数会 -1
>
> 1. 最开始，我们记第一个数字就是众数。
>
> 2. 遍历数组，如果投票数为0，则说明之前数中，众数与非众数个数相同，所以在剩下的数字中，众数依然没有变化
> 3. 投票数为0后，记紧接着的下一个数就是众数
>
> 3. 遍历到最后，我们只需要返回最后一个使投票数+1的数即可。

![image-20200816223825187](http://img.yesmylord.cn//image-20200816223825187.png)

```java
    // 法二：摩尔投票法！！！
    public static int majorityElement2(int[] nums) {
         // 初始化投票数和众数
        int votes = 0;
        int res = nums[0];
        // 开始遍历数组
        for (int num : nums) {
            if(votes==0) {
                // 如果票数为0，就记紧挨着的数字为众数
                res= num;
            }
            // 众数就+1 非众数就-1
            votes += res==num ? 1: -1;
        }
        return res;
    }
```
如果题目没有明确说明这个数组内必然有众数，我们可以在求出最后的结果，再遍历一遍数组，记录该结果出现的次数，如果超过数组的一半就是众数。



### 复杂度分析

空间复杂度：O(1)，只使用了一个votes来记录投票个数

时间复杂度：O(N)，遍历一遍数组，需要O(n)的时间


