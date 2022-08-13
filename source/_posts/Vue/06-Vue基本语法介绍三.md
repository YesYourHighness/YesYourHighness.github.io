---
title: 06.Vue基本语法介绍三
date: 2019-07-20 11:52:57
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

介绍模板语法的基本知识，

加油加油！！
</center>
<!-- more -->

------------

# 插值
## 文本
文本插值法
```javascript
<span>Message: {{ msg }}</span>
```
无论何时只要msg的值发生变化，视图就会更新

一次性的插值,该值不会更新
```
<span v-once>不会改变: {{ msg }}</span>
```
v-text也能实现相同的功能,但是v-text不会转化标签代码
```js
<div id="app">
		<div v-text='name'></div>
	</div>
	<script>
		var vm = new Vue({
			el:'#app',
			data:{
				name:'dell'
			}
		})
	</script>
```
## 原始HTML
为了输出真正的HTML

```HTML
<p>
Using v-html directive: 
<span v-html="rawHtml"></span>
</p>
<!--这里的rawHTML是一个标签表达式，

span标签的内部会被替换成它的属性值-->
```

## 特性
文本插值法不能作用在HTML特性上面，因此需要用到v-bind
```HTML
<button v-bind:disabled="isButton">Button</button>
```
对于布尔特性，如果isButton这个值为null、undefined，false则disabled特性甚至不会被包含在渲染出来的<button>中


## 使用JavaScript表达式

所有的**指令**后都可以使用js的表达式，

插值表达式内也可以使用表达式
```
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```
有个限制就是，每个绑定都只能包含单个表达式，所以下面的例子都不会生效。
```javascript
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```
模板表达式都被放在沙盒中，只能访问全局变量的一个白名单，如 Math 和 Date 。你不应该在模板表达式中试图访问用户定义的全局变量。

## 指令
带有v-前缀的特性
如v-if
```HTML
<p v-if="seen">现在你看到我了</p>
```
## 参数
一些指令可以接收一个参数，在指令名称之后以冒号表示

例如：v-bind可以用于响应式的更新HTML,简写为:
```HTML
<a v-bind:href="url">...</a>
```
这里的href就是参数，告知v-bind指令将该元素的href特性与表达式url的值绑定

又例如：v-on指令，简写为@
```HTML
<a v-on:click="doSomething">...</a>
```
用于监听DOM事件，这里的参数click就是监听的事件名

## 动态参数
可以用方括号括起来的 JavaScript 表达式作为一个指令的参数：
```HTML
<a v-bind:[attributeName]="url"> ... </a>
```
这里的attributeName会被作为一个js的表达式来动态求值，求得的值会作为最终的参数来使用。

例如，如果你的 Vue 实例有一个data属性attributeName，其值为"href"，那么这个绑定将等价于 v-bind:href。

也可以使用动态参数为一个动态的事件名绑定处理函数
```HTML
<a v-on:[eventName]="doSomething"> ... </a>
```
当 eventName 的值为 "focus" 时，v-on:[eventName] 将等价于 v-on:focus

### 动态参数的值的约束
动态参数预期会求出一个**字符串**，异常情况是null

这个**null**值可以被显性的用于移除绑定

### 动态参数表达式的约束

因为某些字符在html的特性名内无效，所以在DOM使用模板要回避大写键名（例如空格和引号）
```HTML
<!-- 这会触发一个编译警告 -->
<a v-bind:['foo' + bar]="value"> ... </a>
```
变通的办法是使用没有空格或引号的表达式，或用计算属性替代这种复杂表达式。

另外，如果你在 DOM 中使用模板 (直接在一个 HTML 文件里撰写模板)，**需要留意浏览器会把特性名全部强制转为小写**
## 修饰符
半角句号 . 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。

