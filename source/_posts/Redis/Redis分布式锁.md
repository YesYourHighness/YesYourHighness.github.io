---
title: Redis分布式锁
date: 2022-02-14 20:16:35
tags:
- Redis
categories:
- Redis
---

<center>
    引言：前天写了人生第一个分布式锁，记录一下。
</center>

<!-- more -->

# Redis分布式锁

需要掌握的点：

- API：`setnx`与`set`
- 加锁、释放锁要保证是原子操作
- 锁要加过期时间
- 锁的value需要不同
- 了解：`redLock`

## 单Redis节点的分布式锁

### Redis 2.6.12之前的`setnx`

> `setnx key value`：  Set if not exists 如果不存在就设置

两个参数：

-  `key` 表示锁 id：锁ID
-  `value`通常设置为：UUID

返回值：

- 为0：表示已经存在锁（可以不断尝试获取）
- 为1：表示设置成功（即获得该锁）

在Redis2.6.12之前，由于没有过期时间，所以需要通过`expire`来实现

但这样存在一个问题：`setnx`与`expire`两个操作，并不是原子操作，所以可能存在`setnx`成功而`expire`失败的情况（肯能会导致死锁，可以写lua脚本实现原子性来实现）

> 为什么要设置过期时间？

如果上锁后，解锁的操作失败，那么此时所有进程都不可能再拿到锁了，因此需要一个过期时间，防止解锁操作失败的情况（至少可以在过期时间后自动解锁）

### Redis 2.6.12之后的`set`

在Redis2.6.12 起，`set`命令完全覆盖了`setnx`，而且还可以设置过期时间

> `set key value [EX seconds] [PX milliseconds] [NX|XX]`
>
> - `EX seconds`：设置失效时长，单位**秒**
> - `PX milliseconds`：设置失效时长，单位**毫秒**
> - `NX`：key**不存在时设置**value，成功返回OK，失败返回(nil)
> - `XX`：key**存在时设置**value，成功返回OK，失败返回(nil)

key值设置为Redis唯一标识即可，value的值设置也不能随便设置

> `value`的值如何设置？为什么要这么设置？

value可以设置为一个随机不重复的串：例如UUID

假如不这样设置：可能会出现这样的情况，服务器A在上锁后，B把A的锁给unlock了

因此value可以**用来区分是谁上的锁**，不同的服务器使用UUID得到的数是不同的，因此可以用来区分到底是哪一台服务器上的锁

```java
// 使用UUID来解锁
if (uuid.toString().equals(Redis.getValue(redisKey))) {
    Redis.delKey(redisKey);
}
```

但是这个demo中，`get`操作与`del`操作也不是原子操作，可能存在几种错误情况：

1. 可能存在`get`成功而`del`失败的情况：这种情况还好，锁还在

2. 可能存在这种情况：锁丢失

   ```java
   // 1、A持有锁
   if (uuid.toString().equals(Redis.getValue(redisKey))) {
       // 2、此时锁过期，B拿到了锁
       Redis.delKey(redisKey); // 3、此时 A del掉了B的锁
       // 4、B：我锁呢？
   }
   ```

> 获取锁与释放锁都需要是原子操作

获取锁Redis已经提供了`set`方法，但是释放锁需要先`get`再`del`，因此释放锁需要原子操作（lua脚本，也可以调Redission的包）

## 多Redis节点的分布式锁

单节点的Redis锁，很有可能随着Redis挂掉，而导致锁失效，因此提出了RedLock

> Redlock：是让客户端和多个独立的 Redis 实例**依次请求加锁**，如果客户端能够和**半数以上的实例成功地完成加锁操作**，那么我们就认为，客户端成功地获得分布 式锁了，否则加锁失败。 

加锁步骤：

1. 客户端获取当前时间
2. 客户端按顺序依次向N个Redis实例执行加锁操作
3. 客户端完成了和所有 Redis 实例的加锁操作，客户端就要计算整个加锁过程的总耗时（为了）

只有满足两个条件，才算真正的加锁成功：

- 有半数以上的Redis节点加锁成功
- 总耗时没有超过锁的有效时间

## 参考资料

- [setnx实现分布式锁](https://zhuanlan.zhihu.com/p/418268774)