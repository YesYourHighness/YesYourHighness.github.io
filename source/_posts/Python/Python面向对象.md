---
title: Python面向对象
date: 2021-04-27 21:13:33
tags: 
- Python
categories: 
- Python

---
<center>
引言：

Python面向对象方面的内容
</center>

<!--more-->


# Python面向对象

## python面向对象

本节主要讲述python中的面向对象，先直接阐述要点，再给出代码示例

（可能之后本篇会进行更新）

### 类的定义与使用

要点：

1. 类的组成
   - 类的帮助信息（可以使用`ClassName.__doc__`查看）
   - 类体：
     - 类成员
     - 方法
     - 数据属性
2. 位置参数`self`
   - 仅仅表示一个位置，换成其他名字也可以，只要在第一个位置即可
   - 表示**类的实例本身**，注意是类的实例，即对象
   - 区分类的函数与普通函数的标志
   - 传参时不必传入self的值，通过对象调用公共方法时，会自动将对象隐式绑定到self参数
3. 注意实例化与赋值的区别，实例化才会调用`__init__`方法
4. 在类中定义的为类的变量，使用类名调用

```python
class Fish:
    "鱼的基类"  # 类的帮助信息，可以使用 ClassName.__doc__ 这里就可以使用 Fish.__doc__查看
    totalNum = 0  # 类变量，在类的所有实例之间共享

    def __init__(self, name, size):
        # __init__是特殊的方法，当创建了这个类的实例时，就会被调用
        # self 代表类的实例！！注意是类的实例，所以我们不能使用self.totalNum，应该是用类名进行调用
        print("构造ing...")
        Fish.totalNum += 1
        self.name = name
        self.size = size


if __name__ == '__main__':
    a = Fish  # 这样代表赋值！！意思是a就如同Fish一样，不会进行实例化，不会调用__init__函数
    print(id(a))
    print(id(Fish))
    #   上两行：a的id与Fish的id是一样的，也不会调用__init__函数
    print(a)
    print(Fish)
    #   上两行：打印出<class '__main__.Fish'>

    print("--------------------------------分割线---------------------------------------")
    a = Fish("小黄鱼", 12)  # 加了括号，这才是实例化，会调用__init__构造方法
    print(id(a))
    print(id(Fish))
    #   上两行：a的id与Fish的id不同
    print(a)  # 打印出实例的地址 <__main__.Fish object at 0x000001CC5F3F9190>（没有重写str方法）

    print("--------------------------------分割线---------------------------------------")
    #   为了证明不加括号的确是赋值，我们可以这样尝试一下
    fish = Fish
    b = fish("小蓝鱼", 22)
    print(b)  # 打印出<__main__.Fish object at 0x000001CC5F3F9250>
    #   确实生成了新的实例，fish等同于Fish

    print("--------------------------------分割线---------------------------------------")
    #   查看类的帮助信息
    print(Fish.__doc__)  # 打印出“鱼的基类”

    print("--------------------------------分割线---------------------------------------")
    print(a.totalNum)  # 打印2，因为我们实例化了两条鱼，小黄鱼和小蓝鱼
    print(b.totalNum)  # 打印2
    print(Fish.totalNum)  # 也可以使用类来打印

```



### 类的函数与普通函数、类的内置属性、对象属性

要点：

1. 类的函数与普通函数的最大的区别：类的函数有位置参数
2. 类的内置属性直接使用类名调用即可
3. 对象属性使用全局方法使用

```python
"""
    类的函数与普通函数
"""


class Fish:
    "鱼的基类"
    totalNum = 0

    def __init__(self, name, size):
        print("构造ing...")
        Fish.totalNum += 1
        self.name = name
        self.size = size

    """
    def eat():
        print("我想吃东西...")
    #   不加self会报错！！eat() takes 0 positional arguments but 1 was given
    """

    def hungry(abbccd):  # self只是一个位置参数，没有强求写成self，这里也是正确的
        print("我想吃东西...")


def normalFunction():  # 普通函数不需要位置参数
    print("这是普通的方法")


if __name__ == '__main__':
    normalFunction()  # 普通方法直接调用即可
    c = Fish("小紫鱼", 5)
    c.hungry()  # 使用对象调用
    d = Fish("小红鱼", 9)
    Fish.hungry(d)  # 也可以这样调用对象的方法，我们可以进一步理解位置参数的作用

    print("--------------------------------分割线---------------------------------------")
    # 类的内置属性
    print(Fish.__dict__)  # 类的属性（一个字典），可以看到，我们定义的类的变量totalNum也在这里
    # 打印出{
    #       '__module__': '__main__', 
    #       '__doc__': '鱼的基类',
    #       'totalNum': 2,
    #       '__init__': <function Fish.__init__ at 0x000001CB3117D430>,
    #       'hungry': <function Fish.hungry at 0x000001CB313905E0>,
    #       '__dict__': <attribute '__dict__' of 'Fish' objects>,
    #       '__weakref__': <attribute '__weakref__' of 'Fish' objects>
    #       }
    print(Fish.__name__)  # 类名
    print(Fish.__doc__)  # 类的帮助信息
    print(Fish.__module__)  # 类定义所在的模块
    print(Fish.__bases__)  # 类的所有父类构成元素（包含了一个由所有父类组成的元组）
    print(Fish.__base__)  # 直接的父类

    print("--------------------------------分割线---------------------------------------")
    # 对象的属性访问
    e = Fish("小绿鱼", 18)
    print(hasattr(e, 'name'))  # 判断是否 True
    print(getattr(e, 'size'))  # 18
    print(setattr(e, 'name', '小白鱼'))    # 给值设值，本身没有返回值返回none
    print(e.name)       # 打印出：小白鱼
    print(delattr(e, 'name'))   # 返回none
    # print(e.name)   报错找不到该属性：'Fish' object has no attribute 'name'

```



### python对象销毁（垃圾回收）

要点：

1. Python 使用了**引用计数**这一简单技术来跟踪和回收垃圾。
2. 在 Python 内部记录着所有使用中的对象**各有多少引用**。

3. 这个对象的引用计数变为0 时， 它将被垃圾回收。但是回收不是"立即"的， 由解释器在适当的时机，将垃圾对象占用的内存空间回收。

```python
"""
    python对象销毁（垃圾回收）
"""


class Fish:
    "鱼的基类"
    totalNum = 0

    def __init__(self, name, size):
        Fish.totalNum += 1
        self.name = name
        self.size = size

    def __del__(self):
        print(self.__class__.__name__ + "被销毁了")


if __name__ == '__main__':
    a = Fish("小黑鱼", 1)
    b = a  # 创建Fish的引用
    c = a  # 创建Fish的引用
    del (a)
    print(1)
    del (b)
    print(2)
    del (c)
    print(3)
```

打印的结果是：

```
1
2
Fish被销毁了
3
```

可以看到，我们创建了三个小黑鱼对象的引用，当全部的引用删除完成后，小黑鱼这个对象本身执行`__del__`函数，然后被析构



### 类的继承

要点：

1. 要继承哪个类，写入括号内，有多个父类用逗号隔开
2. 通过`super()`调用父类的方法，**注意：`super()`是带有括号的**（某些教材使用`super(Class, self).xxx`调用，这是python2.x的写法）
3. 子类可以调用父类的方法

```python
"""
    类的继承
"""


class Fish(object):  # 不写括号，默认继承object类
    "鱼的基类"
    totalNum = 0

    def __init__(self, name, size):
        Fish.totalNum += 1
        self.name = name
        self.size = size

    def __str__(self):
        return str("FISHINFO>>>" + self.name + "," + str(self.size))

    def hungry(self):
        print("我饿了！！")


class Shark(Fish):  # 使用括号进行继承，如果有多个父类，就用“,”隔开
    "鲨鱼类"

    def __init__(self, name, size, age):
        super().__init__(name, size)  # 调用父类的构造
        self.age = age  # 鲨鱼类独有的变量，它有一个年龄

    def __str__(self):
        return super().__str__() + "," + str(self.age)

    def eat(self, fish):  # 鲨鱼类自己拥有的方法
        if not isinstance(fish, Fish):  # 如果输入的是Fish类或其子类，那么返回true
            raise Exception("你不能喂鲨鱼鱼以外的其他东西！")
        print("大口吃鱼！！")
        if fish.size < self.size:
            print("我变大了！")
            self.size += 1
            return True
        else:
            print("不行，我吃不掉它，它有" + str(fish.size) + "，我才" + str(self.size))
            return False


if __name__ == '__main__':
    s = Shark("鲨鱼辣椒", 121, 3)  # 名字取自《铁甲小宝》
    a = Fish("小绿鱼", 18)
    b = Fish("大白鱼", 200)
    print(s)
    s.hungry()  # 子类调用父类的方法
    s.eat(a)  # 子类调用自己独有的方法
    print(s)
    s.hungry()
    s.eat(b)
    # s.eat("我是一个字符串")  报错，因为输入的是一个字符串

    print(Shark.__bases__)  # 打印一下鲨鱼类父类的元组

```



### 类属性与方法

要点：

1. 注意：**在python中没有严格意义的访问限制**
2. python中的三种限制级别：
   1. `name` 公共成员。
   2. `_name` 私有成员。本类和子类访问。（**非强制性**，python建议。也就是说你**依然可以在外部访问它**）
   3. `__name` 强制性私有成员。只在本类访问。（但你依然可以通过`_Classname__name`的方式来**蛮横访问**）

3. 保护类型的变量不能用于` from module import *`

```python
"""
    类的私有属性与方法
"""


class Fish:
    "鱼的基类"
    totalNum = 0
    __group = 1  # 使用两个下划线表示类的私有变量，外部不可以访问（但其实可以访问）
    _age = 1  # 一个下划线表示受保护的，外部不建议访问（注意是 “不建议”），但是可以访问

    def __init__(self, name, size):
        Fish.totalNum += 1
        self.name = name
        self.size = size

    def __str__(self):
        # 内部可以调用私有变量和可继承变量
        return str("FISHINFO>>>" + self.name + "," + str(self.size) + "," + str(self.__group) + "," + str(self._age))

    def __hungry(self):  # 私有方法
        print("我想吃东西...")

    def _eat(self):  # 保护的方法
        print("吃东西")


if __name__ == '__main__':
    a = Fish("小白鱼", 12)
    print(a)
    print(a.totalNum)  # 外部可以访问共有的变量
    # print(a.__group)     报错，外部不能访问私有变量
    print(a._age)  # 外部可以访问类的保护成员
    print(a._Fish__group)  # 外部其实也可以调用类的私有变量，只要使用 objectname._Classname__name 来访问即可

    a._eat()
    # a.__hungry() 报错
    a._Fish__hungry()

```

