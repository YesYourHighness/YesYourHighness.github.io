---
title: 26.Vue中使用Animate.css库
date: 2019-07-26 22:01:03
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>
引言：

Animate.css库的使用
</center>

<!--more-->

------

# 如何使用

1. 去官网下载Animate的库文件
[Animate官网](https://daneden.github.io/animate.css/)

2. 引包

```js
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>	</title>
	<script src="./js/vue.js"></script>
	<link rel="stylesheet" href="js/animate.css">
</head>
<body>
	<div id="root">
		<transition 
		name='fade'
		enter-active-class='animated swing'
		leave-active-class='animated shake'
		>
			<div v-if="show">你好</div>
		</transition>

		<button @click="handleClick">切换</button>
	</div>

	<script>
		let vm = new Vue({
			el:'#root',
			data:{
				show: true
			},
			methods:{
				handleClick(){
					this.show=!this.show
				}
			}
		})
	</script>
</body>
</html>
```

# 首次加载即有动画


加两个类名
- appear
- appear-active-class

```HTML
<transition 
name='fade'
appear
enter-active-class='animated swing'
leave-active-class='animated shake'
appear-active-class='animated swing'
>
	<div v-if="show">你好</div>
</transition>
```

# 如何同时使用过渡和动画

```HTML
<transition 
		name='fade'
		appear
		enter-active-class='animated swing fade-enter-active'
		leave-active-class='animated shake fade-leave-active'
		appear-active-class='animated swing'
		>
			<div v-if="show">你好</div>
		</transition>
```
在动画的名后加上对应的过渡动画名即可，过渡动画要自己设置

## 怎么解决过渡与动画时间不统一

加一个`type`或者`duration`即可

type:
```HTML
<transition
		type="transition"
		name='fade'
		appear
		enter-active-class='animated swing fade-enter-active'
		leave-active-class='animated shake fade-leave-active'
		appear-active-class='animated swing'
		>
			<div v-if="show">你好</div>
		</transition>
```

duration:
单位是ms
```HTML
<transition
		:duration='5000'
		name='fade'
		appear
		enter-active-class='animated swing fade-enter-active'
		leave-active-class='animated shake fade-leave-active'
		appear-active-class='animated swing'
		>
			<div v-if="show">你好</div>
		</transition>
```
甚至可以添加起止时间
```HTML
<transition
		:duration='{enter: 5000,leave:5000}'
		name='fade'
		appear
		enter-active-class='animated swing fade-enter-active'
		leave-active-class='animated shake fade-leave-active'
		appear-active-class='animated swing'
		>
			<div v-if="show">你好</div>
		</transition>
```