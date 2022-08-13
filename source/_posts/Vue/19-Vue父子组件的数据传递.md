---
title: 19.Vue父子组件的数据传递
date: 2019-07-24 21:38:28
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>
引言：

早在第二节我们就讲述了父子组件的传值方法

现在我们更加系统的了解一下父子组件之间如何传值

</center>

<!--more-->

--------

## 父组件->子组件

### 父组件通过绑定属性传给子组件

1. 子组件绑定一个想要传递的值
2. 子组件内使用`props`来接收父组件传递的值
3. 这样，子组件就可以在模板内使用了

例子
```js
<body>
	<div id="root">
			<counter :count='0'></counter>
			<counter :count='1'></counter>
	</div>
	<script>
		//子组件,局部组件方式
		var counter = {
			template: '<div>{{count}}</div>',
			props: ['count']
		}
		//根组件
		var vm = new Vue({
			el:'#root',
			components:{
				counter:counter
			}
		})
	</script>
</body>
```
-------
**当我们使用父组件传来的值，要注意：**

在Vue内有一个单向数据流，要求子组件不能改变父组件的值

因为有可能同时还有其他的组件使用着这个数据，一个改变，则全都要改变

直接更改父组件传来的值，Vue会给出警告

所以我们需要copy一份父组件传来的数据

例子：

```js
<body>
	<div id="root">
		
			<counter :count='0'></counter>
			<counter :count='1'></counter>
	</div>

	<script>

		//子组件,局部组件方式
		var counter = {
			template: '<div @click="handleClick">{{number}}</div>',
			props: ['count'],
			data(){
				return{
					number: this.count
				} 
			},
			methods:{
				handleClick(){
					this.number++
				}
			}

		}

		//根组件
		var vm = new Vue({
			el:'#root',
			components:{
				counter:counter
			}
		})
	</script>
</body>

```

## 子组件->父组件

### 子组件传递给父组件通过事件来传值

1. 使用eimt通过事件来传递数据，emit可以添加多个参数，第一个参数是事件名称，其他都可以是传递的数据
2. 在子组件上添加监听事件，监听事件的名称就是emit内填入的名称，类似`@事件名称=父组件要做出反应的事件名称`
3. 父组件定义相对应的反应方法，有接收的参数记得写在函数的参数部位 

```js
<body>
	<div id="root">
		
			<counter 
			@change='handleSum'
			:count='0'></counter>
			<counter
			@change='handleSum' 
			:count='1'></counter>
			<div>{{total}}</div>
	</div>

	<script>

		//子组件,局部组件方式
		var counter = {
			template: '<div @click="handleClick">{{number}}</div>',
			props: ['count'],
			data(){
				return{
					number: this.count,
					step: 1
				} 
			},
			methods:{
				handleClick(){
					this.number = this.number + this.step;
					//子组件传给父组件通过事件来传值
					//emit函数第一个是事件名，还可以往后添加想要传递的数据
					this.$emit('change',this.step)
				}
			}

		}

		//根组件
		var vm = new Vue({
			el:'#root',
			data:{
				total: 1
			},
			components:{
				counter:counter
			},
			methods:{
				handleSum(step){
					//这里记住要写入参数，接收子组件传来的数据
					this.total += step; 
				}
			}
		})
	</script>
</body>
```