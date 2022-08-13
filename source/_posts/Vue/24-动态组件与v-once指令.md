---
title: 24.动态组件与v-once指令
date: 2019-07-26 22:00:04
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

动态组件是什么？

v-once指令可以干什么？
</center>

<!--more-->

--------


# 动态组件

我想实现一个切换组件的功能

直接写代码，应该是这个样子的
```js
<body>
	<div id="root">
		<child-one v-if='type==="child-one"'></child-one>
		<child-two v-if='type==="child-two"'></child-two>
		<button @click='handleClick'>change</button>
	</div>

	<script>
		Vue.component('child-one',{
			template: '<div>child-one</div>'
			})
		Vue.component('child-two',{
			template: '<div>child-two</div>'
			})

		let vm = new Vue({
			el:'#root',
			data:{
				type: 'child-one'
			},
			methods:{
				handleClick(){
					this.type = (this.type==='child-one'?'child-two':'child-one')
				}
			}
		})
	</script>
</body>
```

我们可以使用更加方便的方式来完成我们的目的

```js
<body>
	<div id="root">
		<component :is="type"></component>
		<!--component是vue的自带组件，指一个动态组件 ,通过is属性就可以实现这件事情-->
		<button @click='handleClick'>change</button>
	</div>

	<script>
		Vue.component('child-one',{
			template: '<div>child-one</div>'
			})
		Vue.component('child-two',{
			template: '<div>child-two</div>'
			})

		let vm = new Vue({
			el:'#root',
			data:{
				type: 'child-one'
			},
			methods:{
				handleClick(){
					this.type = (this.type==='child-one'?'child-two':'child-one')
				}
			}
		})
	</script>
</body>
```
这样就方便了许多

# v-once指令

```js
Vue.component('child-one',{
			template: '<div>child-one</div>'
			})
		Vue.component('child-two',{
			template: '<div>child-two</div>'
			})
```
当我们使用这样的代码时，底层都会创建一个组件，销毁一个组件，这样十分消耗性能

我们只需加一个`v-once`的命令，这样就可以把此组件放入内存，每次调用不需要再重新创建销毁，节约了性能
```js
Vue.component('child-one',{
			template: '<div v-once>child-one</div>'
			})
		Vue.component('child-two',{
			template: '<div v-once>child-two</div>'
			})
```

