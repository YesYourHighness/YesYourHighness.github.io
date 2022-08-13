---
title: 09.Vue基本语法介绍六
date: 2019-07-20 21:56:50
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

这一节介绍有关条件渲染的基本知识，

加油加油！！
</center>
<!-- more -->

------------
# 条件渲染

## 条件渲染`v-if`
`v-id`用于条件性的渲染一块内容，只有当真值的时候才会被渲染
```HTML
<h1 v-if="awesome">Vue is awesome!</h1>
```
还可以添加一个`v-else`
```HTML
<body>
	
	<div id="demo">
		<h1 v-if='awesome'>我能不能出来</h1>
		<h1 v-else>OOOOOh-------</h1>
		<button @click='isUp'>按钮</button>
	</div>

	<script>
		new Vue({
			el:"#demo",
			data:{
				awesome:true,
			},
			methods:{
				isUp:function(){
					return this.awesome=!this.awesome;
				}
			}
		})
	</script>
</body>
```
## 在`<template>`元素上使用`v-if`条件渲染分组

`v-if`只能添加到一个元素上，
当我们想切换多个元素的时候，可以把一个`template`元素当做不可见的包裹元素，并在上面使用`v-if`

```HTML
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```
## 还有`v-else-if`
```javascript
<body>
	
	<div id="demo">
		<h1 v-if='awesome===2'>我能不能出来</h1>
		<h1 v-else-if='awesome===1'>我有点慌</h1>
		<h1 v-else>OOOOOh-------</h1>
		<button @click='isUp'>按钮</button>
	</div>

	<script>
		new Vue({
			el:"#demo",
			data:{
				awesome:2,
			},
			methods:{
				isUp:function(){
					return this.awesome=this.awesome-1;
				}
			}
		})
	</script>
</body>
```

## 用 key 管理可复用的元素

```HTML
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
```
那么在上面的代码中切换 loginType 将不会清除用户已经输入的内容。因为两个模板使用了相同的元素，`<input> `不会被替换掉——仅仅是替换了它的 placeholder。

但有时我们需要两个元素独立，不要复用他们，只需要加一个key即可
```HTML
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
```
注意，<label> 元素仍然会被高效地复用，因为它们没有添加 key 属性。

## 总被渲染的`v-show`

```
<h1 v-show="ok">Hello!</h1>
```
不同的是`v-show`的元素始终会被渲染并保存在DOM中。v-show只是简单地切换元素的css属性display

**注意，v-show 不支持 <template> 元素，也不支持 v-else。**


## `v-if`和`v-show`的区别
v-if 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

v-if 也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

相比之下，v-show元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

所以：

如果需要非常频繁地切换，则使用 v-show 较好；如果在运行时条件很少改变，则使用 v-if 较好。