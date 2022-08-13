---
title: 14.Vue基础巩固
date: 2019-07-22 21:52:46
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

这一节复习巩固这几节的内容，加强对经常出现的Vue语法的记忆

加油加油！
</center>

<!-- more -->

--------

# Vue

## 重新认识基本实例

当挂载点写为**类**，时默认会挂载到第一个此类上
```HTML
<div class="bg">
你好{{msg}}
</div>
<div class="bg">
你好{{msg}}
</div>
```
```js
new Vue({
	el:'.bg',
	data:{
		msg:'Vue'
	}
})
```
所以我们要用**唯一的id**来避免这种情况：
```HTML
<div id="app">
	<div class="bg">
		你好{{msg}}
	</div>
	<div class="bg">
		你好{{msg}}
	</div>
</div>
```
```js
new Vue({
	el:'#app',
	data:{
		msg:'Vue'
	}
})
```
## 模板语法

### Vue的基本结构
一个Vue主要由三个部分构成

1. 样式style
2. 模板template
3. 脚本script

例如我们的Demo
1. 样式
```css
.bg{
	color: red
}
```
2. 模板
```HTML
<div id="app">
	<div class="bg">
		你好{{msg}}
	</div>
	<div class="bg">
		你好{{msg}}
	</div>
</div>
```
3. 脚本

```js
new Vue({
	el:'#app',
	data:{
		msg:'Vue'
	}

})
```


我们惯用的插入文本的方法就是用两个大括号
`{{msg}}`

这里面可以是表达式
`{{count+1}}`

还可以包含模板
`{{template}}`

当然如果template是一个标签语法，类似`<div>hello</div>`的话

会直接打印出整个代码的内容， 如果我们只是想要它显示`div`内部的内容

那么我们需要加一个`v-html=`

`v-bind`给页面的元素绑定一个值
`v-on`绑定某一个事件

```HTML
<div id="app">
	{{msg}}

	{{count+1}}
	<!-- 表达式 -->

	{{template}}
	<!-- 这样会原样打出哦 -->

	<a :href='url'>百度</a>
	<!-- 链接到百度 -->

	<button @click='sum()'>加</button>

	<div v-html="template"></div>
</div>
```

```js
new Vue({
	el:'#app',
	data:{
		msg:'Vue',
		count:0,
		template: '<div>你好啊</div>',
		url:'http://www.baidu.com'
	},
	methods:{
		sum(){
			return this.count++
		}
	}
})
```

## 计算属性与侦听器

计算属性`computed`

侦听器  `watch`
```HTML
<div id="app">
	{{msg}}
	<br>
	{{msg1}}
	<br>
	{{another}}
</div>
```
```js
var vm=new Vue({
	el:'#app',
	data:{
		msg:'hello',
		another:'另一个'
	},
	computed:{
		msg1(){
			return 'computed:  '+this.msg+this.another;
			//这两个值任意一个值发生了变化都会影响到msg1的值
		}
	},
	watch:{
		msg:function (newValue,oldValue) {
			console.log('新的值'+newValue);
			console.log('旧的值'+oldValue);
			console.log('另一个值'+this.another)
		}
		// 把监听的量写在这里，函数可以传两个值，一个是新的值一个是旧的值
		// 监听只能监听到本实例当中的变量
	}
})
```
区别：
watch经常用在异步场景中
compute数据联动


## 条件渲染，列表渲染，Class与Style绑定

`v-if，v-else-if,v-else`真值渲染

不再举例

`v-for`

```HTML
<div id="app">
	<div
		v-for='(item,index) in list'
		:key="index"
		:style='msgStyle'
	>
		{{item.name}}
		{{item.age}}
	</div>
</div>
```
```js
var vm=new Vue({
	el:'#app',
	data:{
		msg:'hello',
		list:[
		{
			name:'A',
			age:32
		},{
			name:'B',
			age:20
		}
		],
		msgStyle:{
			color:'red',
			fontSize: 20+'px'
		}
	},

})
```