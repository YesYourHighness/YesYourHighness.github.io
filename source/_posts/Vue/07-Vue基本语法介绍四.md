---
title: 07.Vue基本语法介绍四
date: 2019-07-20 11:53:30
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

介绍有关计算属性的基本知识，

加油加油！！
</center>
<!-- more -->

------------
# 计算属性和监听器

## 计算属性

对于任何的复杂处理都应该使用**计算属性**

例子
```javascript
<body>
	<div id="example">
		<p>Original msg:"{{msg}}"</p>
		<p>Computed reversed msg:"{{revermsg}}"</p>
	</div>

	<script>
		var vm = new Vue({
			el:'#example',
			data:{
				msg:'这条信息会被逆转'
			},
			computed:{
				//计算属性的getter
				revermsg:function () {
					//this指向vm
					return this.msg.split('').reverse().join('');
				}
			}
		})
	</script>
</body>
```
当原属性发生变化的时候，vue会自动重新计算计算属性，他们俩都会变化

## 计算属性缓存vs方法

上述的例子，我们可以使用方法来解决
如：
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>	</title>
	<script src="js/vue.js" ></script>
</head>
<body>
	<div id="example">
		<p>Original msg:"{{msg}}"</p>
		<p>Computed reversed msg:"{{revermsg()}}"</p><!-- 这里要加括号 -->
		
	</div>

	<script>
		var vm = new Vue({
			el:'#example',
			data:{
				msg:'这条信息会被逆转'
			},
			methods:{
				revermsg: function () {
					return this.msg.split('').reverse().join('');
				}
			}

		})
	</script>
</body>
</html>    
```
这两种方法（计算属性与方法）都可以达到效果，区别是：

**计算属性是基于他们的响应式依赖进行缓存的**
只在相关响应式依赖发生改变时才会重新求值

意味着只要**msg**没有发生改变，多次访问**reversemsg**计算属性会立即返回之前的结果，而不必再次执行函数

因为拥有缓存，再次渲染的时候不需要重新计算该值，增加了渲染的速度

## 计算属性vs监听属性

计算属性一般会比监听更加的方便
例如：

监听属性
```javascript
<body>
	<div id="demo">
		{{fullName}}
	</div>

	<script>
		var vm = new Vue({
			el:'#demo',
			data:{
				firstName:'foo',
				lastName:'bar',
				fullName:'foo bar',
			},
			watch:{
				firstName:function(val){
					this.fullName = val + ""+ this.lastName
				},
				lastName:function(val){
                	this.fullName=this.firstName+''+val
				}
			}
		})
	</script>
</body>

```
而使用计算属性就方便了许多
```javascript
<body>
	<div id="demo">
		{{fullName}}
	</div>

	<script>
		var vm = new Vue({
			el:'#demo',
			data:{
				firstName:'foo',
				lastName:'bar',
				fullName:'foo bar',
			},
			computed:{
				fullName:function(){
					return this.firstName+this.lastName
				}
			}
		})
	</script>
</body>
```
当需要在数据变化时执行异步或开销大的操作时，这个方式是最有用的

## 计算属性的getter和setter
例子：

```js
<body>
	<div id="app">
		{{fullName}}
	</div>
	<script>
		var vm = new Vue({
			el:'#app',
			data:{
				firstName:'Dell',
				lastName:'Lee',
			},
			computed:{
				//fullName可以写为一个对象
				fullName: {
					get(){
						return this.firstName+' '+this.lastName
					},
					set(value){
						var arr = value.split(' ');
						//用空格将其打散
						this.firstName = arr[0];//得到firstName
						this.lastName =  arr[1];//得到lastName
					}
				}
			}
		})
	</script>
</body>
```