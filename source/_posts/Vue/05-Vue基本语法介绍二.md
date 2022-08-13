---
title: 05.Vue基本语法介绍二
date: 2019-07-20 10:03:00
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

介绍了数据与方法，

解释了生命周期钩子

加油加油！！
</center>
<!-- more -->

------------
# Vue入门

## 创建一个Vue实例

```javascript
var vm = new Vue({
    //内容
})
```

注意Vue的V要大写，没有完全遵循MVVM模型，但是一般都用**vm**(ViewModel)这个变量名来表示Vue实例

所有的Vue组件都是Vue的实例，并且接受相同的选项对象


## 数据与方法
当Vue实例被创建的时候，它会把**data**内的所有的属性加入到Vue的响应式系统中

当这些属性值发生变化，视图也会跟着变化

```javascript
// 我们的数据对象
var data = { a: 1 }

// 该对象被加入到一个 Vue 实例中
var vm = new Vue({
  data: data
})

// 获得这个实例上的属性
// 返回源数据中对应的字段
vm.a == data.a // => true

// 设置属性也会影响到原始数据
vm.a = 2
data.a // => 2

// ……反之亦然
data.a = 3
vm.a // => 3
```
注意：
1. 改变任意一个值，会影响到原始的数据
2. 只有当实例被创建的时候**data**中的数据才是响应式的

比如这里,新添加一个新的属性，是不会发生响应的
```javascript
vm.b='你好'
```
但是一开始设置为空的话就可以响应式了
，所以我们要对所有需要响应变化的数据在data中设定它的初始值

例如：
```javascript
data: {
  newTodoText: '',
  visitCount: 0,
  hideCompletedTodos: false,
  todos: [],
  error: null
}
```

但是有唯一的例外**Object.freeze方法**会阻止修改现有的属性，意味着响应系统无法再追踪变化

```javascript
<body>
  <div id="app">
    <p>{{foo}}</p>
    <button @click='foo="baz"'>点击刷新</button>
  </div>
  <script>
    var obj= {
      foo:'bar'
    };
    // Object.freeze(obj);
    new Vue({
      el:'#app',
      data:obj
    });
  </script>
</body>
```

```javascript
vm.$data === data
vm.$el === document.getElementById("example")

// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
```

## 实例声明周期钩子

Vue实例在被创建时要经过一系列初始化过程,在这个过程中会运行一些叫做**生命周期钩子的函数**

例如

**created**钩子可以用来在一个实例被创建之后执行代码
```
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
// => "a is: 1"
```
注意：
不要在选项属性或回调上使用箭头函数

## 生命周期图
![image](https://cn.vuejs.org/images/lifecycle.png)

图中我们可以看到：
1. 创建一个Vue实例
2. 初始化事件和生命周期
3. beforeCreate()
4. 处理外部的注入及双向的绑定
5. created()
6. 是否有`el`选项
7. 是否有`template`选项

当有`el`选项没有`template`,会默认把el外层的整个html标签当做template使用：即以下两种是相同的
```js
<div id="app">
    hello world
</div>
  <script>
    var vm= new Vue({
      el:'#app',
    })
  </script>
```
```js
<div id="app"></div>
  <script>
    var vm= new Vue({
      el:'#app',
      template:'<div>hello world</div>'
    })
  </script>
```
8. 有`template`则正常执行 
9. beforeMount()
10. 开始渲染页面
11. mounted()
12. 等待数据更新
13. beforeUpdate()
14. 虚拟DOM重新渲染匹配
15. updated()
16. 当销毁组件的时候`vm.$destroy()`
17. beforeDestroy()
18. 销毁监听，子组件还有监听事件
19. destroyed()

vue共有十一个生命周期函数

还有三个生命周期函数以后说明,
他们是：

activated()

deactivated()

errorCaptured()

代码:

**生命周期函数千万不要放在methods内**
```js
<body>
  <div id="app">{{content}}</div>

  <script>
    //生命周期函数就是在某个时间点自动执行的函数
    var vm= new Vue({
      el:'#app',
      data:{
        content:'hello world'
      },
      //生命周期函数不要写在methods内
      beforeCreate(){
        console.log("第一个");
        console.log(this.$el);
        //undefined
      },
      created(){
        console.log("第二个");
        console.log(this.$el);
        //undefined
      },
      beforeMount(){
        console.log("第三个");
        console.log(this.$el);
        //<div id="app">{{content}}</div>
      },
      mounted(){
        console.log("第四个");
        console.log(this.$el);
        //<div id="app">hello world</div>
      },

      //当数据更新的时候
      //控制台输入vm.content='bye world'
      beforeUpdate(){
        console.log("第五个");
        console.log(this.$el);
        //<div id="app">bye world</div>
      },
      updated(){
        console.log("第六个");
        console.log(this.$el);
        //<div id="app">bye world</div>
      },


      //控制台执行vm.$destroy()函数
      beforeDestroy(){
        console.log("第七个");
        console.log(this.$el);
        //<div id="app">bye world</div>
      },
      destroyed(){
        console.log("第八个");
        console.log(this.$el);
        //<div id="app">bye world</div>
      }
    })
  </script>
</body>

```