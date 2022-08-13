---
title: 03.Vue-cli的创建与使用
date: 2019-07-19 17:48:54
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

这一节介绍了Vue-cli如何安装，还有一个Todolist的小实例

加油加油！！
</center>
<!-- more -->

------------

# Vue-cli
安装命令行工具(cli)，他会为我们快速搭建繁杂的脚手架

在终端输入
```
npm install --global vue-cli
```

创建一个基于webpack模板的新项目
```
vue init webpack my-project
```
安装依赖
```
cd my-project
```
进入创建的目录下，run运行
```
npm run dev
```
---------
## 文件目录

build文件：放置一些webpack配置文件,不动

config文件：是针对于开发环境和线上环境的配置文件，不动

node-modules文件：项目的依赖

src文件：源代码放置的目录

static文件：放置的是一些静态的资源

index.html文件：整个网页最外层的html文件，带有一个id为app的挂载点

其他都不动

----
## src下的文件

main.js:
是我们整个项目的入口文件，内部已经注册了一个叫做app的组件

app.js:我们要自行设计的页面，我们在这里设计的组件通过main.js内的挂载点显示在页面上

----
## 单文件组件编码

一个文件代表一个组件

而这个组件内有必要的代码

1. template模板
2. scrpit逻辑
3. style样式

-----------------
# 我们使用vue-cli来开发todolist

## 1. main.js
这里是文件的入口，我们创建的组件在这里要引入
```javascript
import Vue from 'vue'
import Todolist from './TodoList'
//1.import引入
Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  components: { Todolist },//2.注册组件
  template: '<Todolist/>'//3.改完这里就可以使用了
});
<!--我们在这里引入我们自定义的 Todolist 组件-->
```
## 2. 自定义的TodoList组件
组件一般也由三个部分组成，要放在components的目录下
```javascript
<template>
  <li class="item" @click="handleDelete">{{content}}</li>
</template>

<script>
  export default {
    props:['content','index'],
    methods:{
      handleDelete:function () {
        this.$emit('delete',this.index)
        //触发父类的事件
      }
    }
  }
</script>
<!--这里使用了scoped来限制css的作用域，使我们的css只作用于本组件-->
<style scoped>
  .item{
    color: green;
  }
</style>

```
## 3.TodoList页面
TodoList.vue文件就放置在与main.js文件相同的位置即可

```javascript
<template>
  <div>
    <div>
      <input v-model="inputValue"/>
      <button @click="handleSubmit">提交</button>
    </div>
    <ul>
      <todo-item
        v-for="(item,index) of list"
        :key="index"
        :content="item"
        :index="index"
        @delete="handleDelete"
      ></todo-item>
      <!--这里使用了我们自建的组件-->
    </ul>
  </div>
</template>

<script>
  import TodoItem from './components/TodoItem'
  export default {
    components:{
      'todo-item': TodoItem
      <!--这种声明方式可以让我们直接使用todo-item这个标签来使用我们的组件-->
    },
    data : function () {
      return{
        inputValue:'',
        list:[]
      }
    },
    methods: {
      handleSubmit:function(){
        this.list.push(this.inputValue);
        this.inputValue='';
      },
      handleDelete:function () {
        this.list.splice(this.index,1)
      }
    }
  }
</script>

<style>

</style>

```