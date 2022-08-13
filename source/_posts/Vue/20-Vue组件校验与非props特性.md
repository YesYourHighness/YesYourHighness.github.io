---
title: 20.Vue组件校验与非props特性
date: 2019-07-24 21:39:08
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>
引言：

组件参数校验是怎么一回事？

非props特性是什么？
</center>

<!--more-->

--------

## 组件参数校验

组件之间传值的时候，我们常常会需要特定类型的值，比如说只要String类型等等，

这时我们就需要检验参数

```js
<body>
	<div id="root">
		<child :content='"hello world"'></child>
	</div>

	<script>
		Vue.component('child',{
			template:'<div>{{content}}</div>',
			props:{
				content: String
			}//如果对得到的参数有要求，可以在props内写成这种样子
		})
		var vm = new Vue({
			el:'#root',
		}) 
	</script>
</body>
```

如果想要得到的参数是String或Number类型的任意一个，可以写成:
```js
<body>
	<div id="root">
		<child :content='"hello world"'></child>
	</div>

	<script>
		Vue.component('child',{
			template:'<div>{{content}}</div>',
			props:{
				content: [Number,String]
			}//可以写为这样的数组形式
		})
		var vm = new Vue({
			el:'#root',
		}) 
	</script>
</body>
```
也可以这样, 也表示只能传入Number类型
```js
Vue.component('child',{
			template:'<div>{{content}}</div>',
			props:{
				content: {
					type: Number
				}
			}
		})
```
还可以指定传入的内容是否必须要有某个类型,当没有传入时会警告
```js
Vue.component('child',{
			template:'<div>{{content}}</div>',
			props:{
				content: {
					type: Number,
					required: true
				}
			}
		})
```
还可以设定默认值
```
Vue.component('child',{
			template:'<div>{{content}}</div>',
			props:{
				content: {
					type: String,
					required: false,
					default: 'default value'
				}
			}
		})      
```
还可以写更加复杂的检验逻辑，使用默认的validator函数
```js
Vue.component('child',{
			template:'<div>{{content}}</div>',
			props:{
				content: {
					type: String,
					validator(value){
						return value.length>5;
					}
					//子组件接收一个值，要求其长度必须大于5
				}
			}
		})
```

## 非props特性

props特性：父组件传递的内容在子组件的props内有声明

特点：
1. 不会在DOM属性上显示
2. 子组件可以直接使用插值表达式使用

上述的例子都是props特性

非props：子组件没有用props声明要接受父组件的内容

特点：
1. 没法用
2. 非props特性会显示在DOM的属性内
