---
title: 18.Vue组件使用细节
date: 2019-07-24 21:37:20
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>
引言：

描述组建的使用细节

防止运行的错误
</center>

<!--more-->

---------

## table元素

```js
<body>
	<div id="app">
		<table>
			<tbody>
				<row></row>
				<row></row>
				<row></row>
			</tbody>
		</table>
	</div>
	<script>
		Vue.component('row',{
			template:'<tr><td>this is a row</td></tr>'
		})
		var vm = new Vue({
			el:'#app'
		})
	</script>
</body>
```
H5规范tbody内必须有tr，所以浏览器会出错

当这样使用模板的时候，乍一看页面没有问题，但是实际上，自定义的组件到了它的外面

解决办法，使用`is`属性,更改一下HTML即可
```HTML
<table>
	<tbody>
		<tr is='row'></tr>
		<tr is='row'></tr>
		<tr is='row'></tr>
	</tbody>
</table>
```
ol，select，li同理

## data

当在非根组件定义data时，必须要求data是一个函数，否则会报错

## 引用ref

当有复杂的动画特效时，有时只有Vue不能满足

需要我们对DOM进行操控

这时我们就要用到引用ref属性



**当ref使用在原有标签上的时候**，我们用ref标签会得到他的DOM节点：

点击hello world字样的时候会在控制台打印出他的内容

```js
<body>
	<div id="root">
		<div 
			@click='handleClick'
			ref='hello'
		>hello world</div>
	</div>

	<script>
		var vm = new Vue({
			el:'#root',
			methods:{
				handleClick(){
					//$refs指所有的引用
					console.log(this.$refs.hello.innerHTML)
					//这个就是此处的DOM节点
				}
			}
		})
	</script>
</body>
```
**当在自定义组件上使用ref的时候**，获取到的是这个组件的引用
```js
<body>
	<div id="root">
		<counter 
		ref='one'
		@change='handleChange'></counter>
		<counter 
		@change='handleChange'
		ref='two'
		></counter>
		<div>{{total}}</div>
	</div>

	<script>
		Vue.component('counter',{
			template:'<div @click="handleClick">{{number}}</div>',
			data(){
				return {
					number: 0
				}
			},
			methods:{
				handleClick(){
					this.number ++;
					this.$emit('change')
				}
			}
		})

		var vm = new Vue({
			el:'#root',
			data(){
				return{
					total: 0
				}
			},
			methods:{
				handleChange(){
					return this.total = this.$refs.one.number+this.$refs.two.number
					//直接用.即可获取自定义组件内的内容
				}
			}

		})
	</script>
</body>
```

