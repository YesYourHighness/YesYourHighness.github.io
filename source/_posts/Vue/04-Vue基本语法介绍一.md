---
title: 04.Vue基本语法介绍一
date: 2019-07-19 21:24:23
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

这一节介绍有关事件，事件处理的基本知识，

加油加油！！
</center>
<!-- more -->

------------
# Vue入门

## Vue是什么

一套用于构建用户界面的**渐进式框架**

## 1.声明式渲染与指令

Vue.js的核心是:
用一种简单的模板语法来将数据渲染进入DOM


插值表达式
```javascript
<div id="app">{{msg}}</div>

var app = new Vue({
    el:"#app",
    data:{
        msg:'hello'
    }
})
```
---
v-bind:绑定元素，可以简写为一个冒号:
```javascript
<body>
	<div id="app">
		<span v-bind:title='msg'>悬停此处查看信息</span>
		
		<span :title='msg'>这里是简写形式</span>
	</div>

	<script>
		var vm = new Vue({
			el:'#app',
			data:{
				msg:'页面加载于'+new Date().toLocaleString()
			}
		})
	</script>
</body>
```

带有前缀v-的被称为**指令**

他们是Vue提供的特殊特性，他们会在渲染的DOM上应用特殊的响应式行为

v-bind:title='msg' 意思是：  将这个元素节点的 title 特性与Vue实例的 msg 属性保持一致


## 2.条件与循环

条件
```javascript
<body>
	<div id="app">
		<p v-if='seen'>条件</p>
	</div>

	<script>
		var vm = new Vue({
			el:'#app',
			data:{
				seen:true
			}
		})
	</script>
</body>
```
更改seen的布尔值会发现“条件”两字会显示和“消失”（注意这里是消失,直接在DOM上去除）


循环
```javascript
<body>
	<div id="app">
		<ul>
			<li v-for="todo in todos">{{todo.text}}</li>
		</ul>
	</div>
	<script>
		var vm = new Vue({
			el:'#app',
			data:{
				todos:[
				{text: '学习java'},
				{text: '学习js'},
				{text: '学习Vue'}
				]
			}
		})
	</script>
</body>
```
注意这里的**v-for**后面是赋值符号
```javascript
<li v-for="todo in todos">{{todo.text}}</li>
```
in的前面是自定义的一个名字，in的后面是data里面的一个数组

## 3.处理用户输入

**v-on**事件监听器

```javascript
<body>
	<div id="app">
		<p>{{msg}}</p>
		<button v-on:click='reverMsg'>逆序消息</button>
		<button @click='reverMsg'>简写</button>
	</div>
	<script>
		var vm = new Vue({
			el:'#app',
			data:{
				msg:'全面学习Vue'
			},
			methods:{
				reverMsg:function () {
					this.msg=this.msg.split('').reverse().join('')
				}
			}
		})
	</script>
</body>
```
v-on:可以简写为@符号

---
**v-model**来实现表单输入和应用状态的双向绑定
```javascript
<body>
	<div id="app">
		<p>{{msg}}</p>
		<input v-model='msg'>
	</div>
	<script>
		var vm = new Vue({
			el:'#app',
			data:{
				msg:''
			}
		})
	</script>
</body>
```

## 4.组件化应用构建

我们可以用组件来组成一个大型的应用

一个组件在Vue中即是一个Vue的实例

Vue中注册组件
```javascript
<body>
	<div id="app">
		<todo-item>这是一个新的组件</todo-item>
	</div>
	<script>
		Vue.component('todo-item',{
			template:'<li>这是一个自定义的组件</li>'
		})
	</script>
</body>
```
但是这样毫无意义，下面是一个小的实例
```javascript
<body>
	<div id="app">
		<todo-item
			v-for='item in List'
			:todo='item'
			:key='item.id'
		></todo-item>
	</div>
	<script>
		//子类
			Vue.component('todo-item',{
			props:['todo'],//子类接收父类的信息
			template:'<li>{{todo.text}}</li>'
		})
		//父类
		var vm = new Vue({
			el:'#app',
			data:{
				List:[
				{id:0,text:'java'},
				{id:1,text:'js'},
				{id:2,text:'Vue'}
				]
			}
		})
	</script>
</body>
```


