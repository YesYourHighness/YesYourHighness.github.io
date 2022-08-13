---
title: 08.Vue基本语法介绍五
date: 2019-07-20 21:56:38
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

介绍Vue中有关CSS与Style绑定的基本知识，

加油加油！！
</center>
<!-- more -->

------------
# CSS与Style绑定

## 绑定HTML Class

#### 对象语法

我们可以动态的切换class
```HTML
<div v-bind:class="{ active: isActive }"></div>
```
表示：active这个class存在与否将取决于数据属性**isActive**的**真值**

记住形式`:class='{active: isActive}'`

多个值的切换
```HTML
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>
```
```
data: {
  isActive: true,
  hasError: false
}
```
结果就是
```HTML
<div class="static active"></div>
```
当isActive和hasError发生变化的时候，class列表会相应的更新

绑定的数据对象可以写在组件里
```
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```
还可以绑定一个返回对象的计算属性
```javascript
<body>
	<div 
		id="demo" 
		:class='classObject'	
	></div>

	<script>
		data:{
			isActive:true,
			error:null
		},
		computed:{
			classObject:function () {
				return{
					active: this.isActive&&!this.error,
					'text-danger':this.error&&this.error.type==='fatal'
					//text-danger是bootstrap的一个属性，这里理解原理即可
				}
			}
		}
	</script>
</body>
```

#### 数组语法

我们可以把一个数组传给v-bind:class
```HTML
<div v-bind:class="[activeClass, errorClass]"></div>
```
```
data:{
    activeClass:'active',
    errorClass:'text-danger'
}
```
渲染为:
```HTML
<div class='active text-danger'></div>
```

要加入条件可以使用三元表达式
```HTML
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```

这样始终有errorClass但是activeClass会根据真值来判断是否添加

在数组语法中也可以使用对象语法
```HTML
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

#### 用在组件上

当在一个自定义组件上使用class属性的时候，这些类将被添加到该组件的根元素上面，这个元素上已经存在的类不会被覆盖

```javascript
<body>
	<div id="demo">
		<my-component class='baz boo'></my-component>
	</div>
		
	<script>
		Vue.component('my-component', {
  			template: '<p class="foo bar">Hi</p>'
		})
		new Vue({
			el:'#demo'
		})//new Vue一定要放在组件注册的后面，否则会显示组件没有注册
		
	</script>
	
</body>
```
HTMl将被渲染为
```HTML
<p class="foo bar baz boo">Hi</p>
```
## 绑定内嵌样式
#### 对象语法
`v-bind:style` 的对象语法十分直观——看着非常像 CSS，但其实是一个 JavaScript 对象

CSS 属性名可以用驼峰式 或 短横线分隔(括号括起来) 来命名
```HTML
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
```javascript
data: {
  activeColor: 'red',
  fontSize: 30
}
```
直接绑定到一个样式对象上更好
```HTML
<div v-bind:style="styleObject"></div>
```
```javascript
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```
#### 数组语法
`v-bind:style` 的数组语法可以将多个样式对象应用到同一个元素上：

```HTML
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```
#### 多重值
```HTML
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```
这样写只会渲染数组中**最后一个被浏览器支持的值**。在本例中，如果浏览器支持不带浏览器前缀的 flexbox，那么就只会渲染 display: flex。