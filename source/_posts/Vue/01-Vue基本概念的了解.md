---
title: 01.Vue的开始认识
date: 2019-07-03 21:25:28
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

Vue是著名的前端框架，从今天开始要慢慢了解它，本节粗略介绍了Vue的基本知识点

先从一个Hello world开始吧

加油加油！！
</center>
<!-- more -->

------------
# Hello world
从Vue官网上下载Vue的源代码，初学我们可以直接用
```javascript
<script src="vue.js"></script>
```
我们可以直接将它放在一个单独的文件当中，然后放在head的位置，用script标签来引入，这样就可以使用vue了


这里是一个Hello world小实例

如果使用纯js的话

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title> </title>
</head>
<body>
    <div id="app"></div>
    <script>
        var iApp = document.getElementById('app');
        iApp.innerHTML='hello world';
        setTimeout(function() {
            iApp.innerHTML = 'bye,world'
        }, 2000)
    </script>
</body>
</html>
```

```javascript
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <script src="vue.js"></script>
    <!--CDN引入vue包-->
</head>
<body>
<div id="root">{{msg}}</div>
<!--省去了DOM操作，可以直接控制这里的数据-->
<script>
    let vm = new Vue({//创建一个Vue的实例
        el: '#root',
        //接管哪一个页面上的element，实现了绑定,el代表掌管页面上的哪一块区域
        data:{
            msg: "hello world"
        }
    })
    
    setTimeout(function() {
        app.$data.msg = 'bye,world'
    }, 2000)
    //$data是Vue实例当中data对象的别名
</script>
</body>
</html>
```
-----

# 挂载点、模板之间的关系
## 挂载点
```javascript
<body>
<!--挂载点、模板、实例的关系-->
<div id="root">{{msg}}</div><!--这里是挂载点-->
<div>{{msg}}</div><!--这里不是挂载点，所以页面内会直接显示出原本的内容-->
<script>
    new Vue({
        el: '#root',
        data: {
            msg: "hello world"
        }
    })
</script>
</body>
```
## 模板
模板就是指挂载点内部的内容
```javascript
<div id="root">
    <h1>hello {{msg}}</h1><!--模板：挂载点内部的内容-->
</div>
```
还有另一种的表现方式
```javascript
<body>
<!--挂载点、模板、实例的关系-->
<div id="root"></div>
<script>
    new Vue({
        el: '#root',
        template:'<h1>hello {{msg}}</h1>',
        //模板可以写在挂载点内部，也可以写在实例的template的属性内部
        data: {
            msg: "hello world"
        }
    })
</script>
</body>
```
------
## 数据的显示
```javascript
<body>
<div id="root">
    <h1>{{msg}}</h1>
    <!--1.我们把数据放在两个大括号的内部，这种语法，叫做插值表达式-->
    <h2 v-text="msg"></h2>
    <!--2.v-text：这个指令意思是，h2当中的内容由这个变量来指定,
    这里的意思就是把msg的内容显示到这里-->
    <h3 v-html="msg"></h3>
    <!--3.v-html：表示也是可以的-->
    <!--那么他们俩的区别是什么呢-->
    <h4 v-text="content"></h4><!--会转义，将标签内容直接打印出来-->
    <h4 v-html="content"></h4><!--不会转义，效果和template一样-->

</div>
<script>
    new Vue({
        el: '#root',
        data: {
            msg: "hello world",
            content:"<h4>会不会转义</h4>"
        }
    })
</script>
</body>
```
## 方法的添加
```javascript
<body>
<div id="root">
    <div v-on:click="handleClick">{{msg}}</div>
    <!--这里可以把 v-on:简写为@-->
    <div @click="handleClick">{{msg}}</div>
    <!--绑定一个事件-->
</div>
<script>
    new Vue({
        el: '#root',
        data: {
            msg: "hello",
        },
        methods: {//把方法添加到这个属性内部
            handleClick: function () {
                this.msg = 'world'
                //直接这样就可以调用数据了，vue会直接改变页面的现实数据
            }
        }
    })
</script>
</body>
```
--------
## 属性绑定
```javascript
<body>
    <div id="root">
        <div v-bind:title="title">hello world</div>
        <!--1.title属性:当鼠标放在上面的时候，会出现一个框，内容就是title的内容-->
        <!--2.当title内部想要使用data的数据的时候，直接写title:"title"是不行的
            必须进行绑定-->
        <!--3.此时后面的内容就是一个表达式了，可以进行js的更改-->
        <!--例如-->
        <div v-bind:title="'你好啊'+title">hello world</div>
        <!--4.v-bind可以缩写为:-->
        <div :title="'你好啊'+title">hello world</div>

    </div>
    <script>
        new Vue({
            el:"#root",
            data:{
                title:"this is hello world"
            }
        })
    </script>
</body>
```
----------
## 双向数据绑定
### 单向数据：数据决定页面的显示，但是页面无法决定数据里的内容
```javascript
<body>
    <div id="root">
        <div :title="title">hello world</div>
        <input :value="content"/>
        <div>{{content}}</div>
        <!--单向绑定：数据决定页面的显示，但是页面无法决定数据里的内容-->
    </div>
    <script>
        new Vue({
            el:"#root",
            data:{
                title:"this is hello world",
                content:"this is content"
            }
        })
    </script>
</body>
```
### 双向数据：数据决定页面的显示，页面也可以决定数据里的内容
```javascript
<body>
    <div id="root">
        <div :title="title">hello world</div>
        <input v-model="content"/>
        <div>{{content}}</div>
        <!--双向绑定：数据决定页面的显示，页面也能决定数据里的内容-->
        <!--v-model双向数据绑定-->
    </div>
    <script>
        new Vue({
            el:"#root",
            data:{
                title:"this is hello world",
                content:"this is content"
            }
        })
    </script>
</body>
```
# Vue中的计算属性和侦听器
## 计算属性
```javascript
<body>
    <div id="root">
        姓：<input v-model="firstName" />
        名：<input v-model="lastName" />
        <div>{{firstName}}{{lastName}}</div>
        <!--最好写成以下的这个样子-->
        <div>{{fullName}}</div>
    </div>
    <script>
        new Vue({
            el:"#root",
            data:{
                firstName:'',
                lastName:''
            },
            computed:{//计算属性computed
                fullName:function () {
                    return this.firstName +""+this.lastName
                }
                //当firstName和lastName没有变的时候，fullName会使用缓存值
                //只有当依赖的数据发生变化的时候，才会重新计算
            }
        })
    </script>
</body>
```
## 监听器
```javascript
<body>
    <div id="root">
        姓：<input v-model="firstName" />
        名：<input v-model="lastName" />
        <div>{{fullName}}</div>
        <div>{{count}}</div>
    </div>
    <script>
        new Vue({
            el:"#root",
            data:{
                firstName:'',
                lastName:'',
                count:0
            },
            computed:{
                fullName:function () {
                    return this.firstName +""+this.lastName
                }
            },
            watch:{//监听某一个数据的变化，一旦这个数据变化，则可以在这里看到他的变化
                fullName:function () {
                    this.count++
                }
            }
        })
    </script>
</body>
```
-----
# 常见命令的使用
## v-if
```javascript
<body>
    <div id="root">
        <div v-if="show">hello world</div>
        <!--v-if指令可以判断条件再来选择是否显示-->
        <!--v-show指令也可以做到这一点，但是他们有区别-->
        <!--1.v-if是直接在DOM树上抹去了这一个标签-->
        <!--2.v-show则是给这个标签添加了一个display:none的属性-->
        <button @click="handleClick">toggle</button>
        <ul>
            <li v-for="item of list">{{item}}</li>
            <!--每次循环都把list的每一个项传递到item这个变量中然后显示出来-->
            <li v-for="item of list" :key="item">{{item}}</li>
            <!--记得要加一个key的属性，会提升每一项渲染的性能-->
            <!--key值要求每一项不可以相同，这里可以直接用item,但当重复的时候我们会这样写-->
            <li v-for="(item,index) of list" :key="index">{{item}}</li>
            <!--我们不需要对排序进行变更及类似的操作，index这样写没有问题，但这样写是会有问题的！-->
        </ul>
    </div>
    <script>
        new Vue({
            el:"#root",
            data:{
                show:true,
                list:[1,2,3]
            },
            methods:{
                handleClick:function () {
                    this.show=!this.show
                }
            },
        })
    </script>
</body>
```
