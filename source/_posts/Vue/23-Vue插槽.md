---
title: 23.Vue插槽
date: 2019-07-26 21:59:43
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言:

Vue的插槽如何使用呢

来看看吧
</center>

<!--more-->

---------

# Vue插槽

当我们在开发中，总会有这样的需求

```js
<div id="root">
		<child content='<p>我想加这个内容</p>'></child>
	</div>

	<script>
		Vue.component('child',{
			props:["content"],
			template:'<div><p>hello</p>{{content}}</div>',
			})

		let vm = new Vue({
			el:'#root'
		})
	</script>
```
但这样传入content会被转义,即直接显示`<p>我想加这个内容</p>`

这显然不是我们想要的

我们想让他直接的显示出来

我们之前学过一个指令`v-html`可以解决这个问题

```js
Vue.component('child',{
	props:["content"],
	template:'<div><p>hello</p><div v-html="content"></div></div>',
	//这里使用template站位符没有作用,会导致渲染不出来
	})
```
但是这种方法，还会给外部增加一个div

Vue提供给我们一个插槽属性来实现这个功能

使用插槽：
```js
<body>
	<div id="root">
		<child>
			<p>使用插槽</p>
		</child>
	</div>

	<script>
		Vue.component('child',{
			template:'<div>
			            <p>hello</p>
			            <slot>这里还可以添加默认值</slot>
		            </div>',
			//当没有插入任何值，会启用这个函数
			})

		let vm = new Vue({
			el:'#root'
		})
	</script>
</body>
```
我们还可以给插槽起名字，像是这个样子:
```js
<body>
	<div id="root">
		<child>
			<p slot='name1'>第一个插槽</p>
			<p>使用插槽</p>
			<p slot='name2'>第二个插槽</p>
		</child>
	</div>

	<script>
		Vue.component('child',{
			template:'<div>
			            <slot name="name1">默认值1</slot>
			            <p>hello</p>
			            <slot name="name2">默认值2</slot>
		            </div>',
			})

		let vm = new Vue({
			el:'#root'
		})
	</script>
</body>
```

# Vue的作用域插槽

当我们想在子组件渲染循环
```js
Vue.component('child',{
	data(){
		return {
			list: [1,2,3,4]
		}
	},
	template:'<div><li v-for="item of list">{{item}}</li></div>',
		})
```
但我们可能不想循环`<li>`标签，我们想让外部决定怎么渲染

这时我们就需要用到作用域插槽
```js
<body>
	<div id="root">
		<child>
			<template slot-scope='props'>
				<h1>{{props.item}}</h1>
				<!--传过来的内容存入了props内-->
			</template>
		</child>
	</div>

	<script>
		Vue.component('child',{
			data(){
				return {
					list: [1,2,3,4]
				}
			},
			template:'<div><slot v-for="item of list" :item=item >{{item}}</slot></div>',
			//这里想插槽内传入想要他显示的内容
			})

		let vm = new Vue({
			el:'#root'
		})
	</script>
</body>
```