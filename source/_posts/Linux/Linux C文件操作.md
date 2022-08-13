---
title: Linux_C文件操作
date: 2020-07-13 21:00:00
tags: 
- Linux
categories: 
- Linux
---
<center>
引言：Linux c 文件操作；文件描述符
</center>

<!-- more -->

# Linux C文件操作

> Linux系统**I/O**：`open() read() write() lseek close`

## 文件描述符

文件描述符是一个数组的下标值。

​		`files`是一个文件指针数组，一般来说，一个进程会从`files[0]`读取输入，将输出写入`files[1]`，将错误信息写入`files[2]`

**每个进程被创建时，`files`的前三位被填入默认值，分别指向标准输入流、标准输出流、标准错误流。我们常说的「文件描述符」就是指这个文件指针数组的索引**，所以程序的文件描述符默认情况下 0 是输入，1 是输出，2 是错误，3及以上就是打开的文件

![img](https://pic2.zhimg.com/v2-cdf5a0fb5fbd7b83f66e842ca394618c_b.jpg)

（图片取自[知乎]([https://www.zhihu.com/search?type=content&q=%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6](https://www.zhihu.com/search?type=content&q=文件描述符))）



## open()函数

头文件：

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
```

原型：

```c
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
```

参数：

- pathname ：返回要打开的文件路径名称

- flags ：

  1. `O_RDONLY`：只读方式打开
  2. `O_WRONLY`：只写方式打开
  3. `O_RDWR`：可读可写
  4. `O_CREAT`：文件不存在则创建
  5. `O_TRUNC`：文件已存在则删除文件原有数据
  6. `O_APPEND`：追加

  （其中1、2、3互相排斥，其余可以使用`|`组合使用）

- mode ：如果文件被新建，指定其权限为mode（八进制表示）

返回值：

- 成功： 大于等于0 的整数（即文件描述符fd）
- 失败：-1,并且errno会被设置

---



示例

```c
#include "stdio.h"
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int main(){
        int fd1 = open("./new.txt",O_RDONLY);             
        printf("%d",fd1);
        getchar();//为了查看程序的状态，在这里卡一下程序
        return 0;
}
```

编译执行程序，打开另一个终端

```shell
[root@localhost local]# ps -aux | grep a.out 
root      3225  0.0  0.0   4220   352 tty1     S+   07:08   0:00 ./a.out
root      3227  0.0  0.0 112808   980 pts/0    S+   07:10   0:00 grep --color=auto a.out
[root@localhost local]# ll /proc/3225/fd
fd/     fdinfo/ 
[root@localhost local]# ll /proc/3225/fd
total 0
lrwx------. 1 root root 64 Jul 13 07:10 0 -> /dev/tty1
lrwx------. 1 root root 64 Jul 13 07:10 1 -> /dev/tty1
lrwx------. 1 root root 64 Jul 13 07:10 2 -> /dev/tty1
lr-x------. 1 root root 64 Jul 13 07:10 3 -> /usr/local/new.txt
```

为什么要查看`/proc/3225/fd`下的内容呢？

每个进程被创建后，在`/proc/[PID]/fd`文件下，会有对应文件标识符，此例的文件标识符

```shell
[root@localhost local]# ls /proc/3225/fd
0  1  2  3
# 其中0、1、2对应着标准输入、标准输出、标准错误，而3就是本文件
```

我们可以看到`lr-x------. 1 root root 64 Jul 13 07:10 3 -> /usr/local/new.txt`，文件的权限是`r-x`可读可执行，这就是`O_RDONLY`，如果改为`O_WRONLY`那这里就是`-wx`

如果我们在文件中`open`打开多个文件，那他们的`fd`就是3,4,5,6...依次往下排序

## close()函数

头文件：

```c
#include <unistd.h>
```

原型：

```c
int close(int fd);
```

参数：

- `fd`：要关闭的文件描述符

备注：重复关闭一个文件或是未打开的文件是安全的

---



示例：

如果一个文件被打开后，又关闭，然后又打开一个文件，那么这个文件的`fd`是3还是4？

```c
#include "stdio.h"
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int main(){
        int fd1 = open("./new.txt",O_RDONLY);
        printf("%d\n",fd1);
        close(fd1);
        int fd2 = open("./new.txt",O_RDONLY);
        printf("%d\n",fd2);
        return 0;
}
```

结果是3

```shell
[root@localhost local]# ./a.out 
3
3
```

说明当我们关闭一个文件后，对应的`files[3]`会被删除，新打开一个文件后，这个文件又被打开。

即使我们打开了三个文件，分别是 3 4 5 ，我们关闭了 3，然后再打开一个文件，这个文件的`fd`依然是3 



## read()函数

头文件：

```c
#include <unistd.h>
```

原型：

```c
ssize_t read(int fd, void *buf, size_t count);
```

参数：

- `fd`：文件描述符
- `buf`：指向存放读到的数据的缓冲区，（就是放数据的内存首地址）
- `count`：想要从文件 fd 中读取的字节数

返回值：

- 成功：实际读到的字节数
- 失败：-1

补充： `ssize_t` ：是类型重定义，为了跨平台兼容。比如说long在32位系统可能是4字节，64位系统可能是8字节，嵌入式开发有的只有16位，那么int只有2个字节。

## write()函数

头文件：

```c
#include <unistd.h>
```

原型：

```
ssize_t write(int fd, const void *buf, size_t count);
```

参数：

- `fd`:将数据写入到文件的fd中
- `buf`:要写入的数据
- `count`:要写入的字节数

备注：实际写入的字节数小于等于count

---

实现从`a.txt`文件中读取数据，写入到`b.txt`中

```c
#include "stdio.h"
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int main(){
        int fd1 = open("./a.txt",O_RDONLY);
        int fd2 = open("./a.txt",O_WRONLY);
        int buf = [100];
        read(fd1,buf,10);
        write(fd2,buf,10);
        return 0;
}
```

## lseek()

待补充

## **mmap()**

待补充