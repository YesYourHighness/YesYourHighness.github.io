---
title: JAVA-集合-6
date: 2019-08-17 16:14:49
tags: 
- Java
- Java集合
categories: 
- 后台
- Java
---

<center>
引言：Collections、Map集合
</center>

<!--more-->

---

# Collections
此类完全由在 `collection` 上进行操作或返回 `collection` 的静态方法组成。

它包含在 `collection` 上操作的多态算法，即“包装器”，包装器返回由指定 `collection` 支持的新 `collection`，以及少数其他内容。

## 常用方法

```java
Collections.addAll()//参数第一个是一个集合，后面是任意个元素

Collections.shuffle//参数：一个集合,打乱集合

//1.
Collections.sort()
//参数只能是一个List集合，不能是set集合！
//将集合的元素按默认排序，默认是升序，从小到大
//自定义类型的sort必须重写Comparable接口中CompareTo这个方法，否则默认报错

//2.
Collections.sort()
//两个参数，第一个是集合，第二个是方法new 
//区别: 
//Comparator相当于找一个第三方的裁判，比较两个
//Comparable自己和参数比较，自己需要实现Comparable接口，重写比较的规则compareTO方法
```
## 示例代码

```java
ArrayList<String> list = new ArrayList<>();
Collections.addAll(list,"a","v","w","6","g");
Collections.shuffle(list);//打乱
System.out.println(list);//[g, 6, v, a, w]
Collections.sort(list);//默认升序排序
System.out.println(list);//[6, a, g, v, w]
```
自定义元素`sort`,需要在自定义类中重写`compareTo`方法

引入`Comparable<>`接口
```java
public class father implements Comparable<father>{
    //这里要引入一个接口Comparable,注意写入泛型
    @Override
    public int compareTo(father f){
        return this.getAge() - f.getAge();
        //return 0;默认返回一个零，认为是相同的
    }
...}
```
```java
father f1 = new father("一号",10);
father f2 = new father("二号",18);
father f3 = new father("三号",40);
ArrayList<father> list = new ArrayList<>();
Collections.addAll(list,f1,f2,f3);
Collections.sort(list);
```

`Comparable<>`接口的排序规则:

this - 参数 -> 升序

参数 - this -> 降序


# Map

Map<K,V>

- K键：此映射所维护的键的类型
- V键：映射值的类型

Map是一个双列集合，与Collection不同

将键映射到值的对象。一个映射不能包含重复的键；每个键最多只能映射到一个值。 

特点：
1. Map集合是一个双列集合，包含一个键一个值
2. 键和值的数据类型不相关
3. key不可以重复，但是value可以重复
4. key和value一一对应

## 包路径
```
java.util
```
## 实现类
- HashMap(底层是一个哈希表)
    1. 查询速度快
    2. 底层是哈希表
    3. 无序集合
- LinkedHashMap
    1. 底层是一个哈希表+链表
    2. 有序集合

## 常用方法

```java
V put(K key, V value)
//将指定的值与此映射中的指定键关联（可选操作）
//如果此映射以前包含一个该键的映射关系，则用指定值替换旧值

//存储键值对的时候
//key不重复，返回值v是null
//key重复，会使用新的v替换map中重复的v，返回被替换的value值

V remove(Object key)
//key存在，返回被删除的值
//key不存在，返回null

V get(Object key)
//返回指定键所映射的值
//如果此映射不包含该键的映射关系，则返回 null

boolean containsKey(Object key)
//如果此映射包含指定键的映射关系，则返回 true。
//更确切地讲，当且仅当此映射包含针对满足 (key==null ? k==null : key.equals(k)) 的键 k 的映射关系时，返回 true。

Set<K> keySet()
//把Map集合中所有的key取出，存入到Set集合当中

```

## 示例代码
```java
Map<String,String> map = new HashMap<>();
map.put("李晨","范围");//不重复返回null
map.put("李晨","范冰冰");//重复返回被替换去掉的值
map.put("杨过","小龙女");
System.out.println(map);//{杨过=小龙女, 李晨=范冰冰}
map.remove("李晨");
System.out.println(map);//{杨过=小龙女}
System.out.println(map.get("杨过"));//小龙女
System.out.println(map.containsKey("李晨"));//false
Set<String> set = map.keySet();//转化为set集合
System.out.println(set);//[杨过]
```

## Map遍历

- 法一：利用keySet方法
```java
Map<String,String> map = new HashMap<>();
map.put("李晨","范围");
map.put("阿q","abc");
map.put("杨过","小龙女");
Set<String> set = map.keySet();
for (String s : set) {
    System.out.println(s + "," + map.get(s));
}
```

- 法二：使用entrySet方法
```java
Map<String,String> map = new HashMap<>();
map.put("李晨","范围");
map.put("阿q","abc");
map.put("杨过","小龙女");
//通过Map名来找到Entry对象
for(Map.Entry<String, String> s : map.entrySet()){
    System.out.println(s.getKey()+","+s.getValue());
}
```
存储自定义类型，记得要重写hashCode方法，去重

## Map.Entry
Map接口中有一个内部接口Entry

作用：当Map集合一旦创建，那么就会在Map集合中创建一个Entry对象，用来记录**键与值**的关系

他有两个方法:
- getKey
- getValue
用来获取V和K

# JDK9优化
jdk9中，list，set，map都存入了一个静态of方法，
**只适用于这三个接口，不适用于这三个接口的实现类**
1. 可以用来  一次性 添加  多个元素
2. of方法的返回值是一个不能改变的集合，该集合不能再用add，put存入元素
3. Set和Map接口在调用of方法时不能有重复的元素
所以要在数量确定的时候再用
```java
List<String> list = List.of("a","b","c");
```