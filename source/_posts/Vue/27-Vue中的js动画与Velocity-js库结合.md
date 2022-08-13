---
title: 27.Vue中的js动画与Velocity.js库结合
date: 2019-07-26 22:01:26
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>
引言:

js写动画

再加上Velocity库
</center>

<!--more-->

-------

# js动画

我们可以借助生命钩子来实现动画

```
<body>
	<div id="root">
		<div>
			<transition
			name='fade'
			@before-enter='handleBeforeEnter'
			@enter='handleEnter'
			@after-enter='handleAfterEnter'
			>
				<div v-if="show">你好</div>
			</transition>
			<button @click="handleClick">切换</button>
		</div>
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
				},
				handleBeforeEnter(el){
					el.style.color='red'
				},
				//这里有一个参数el
				handleEnter(el,done){
					setTimeout(() => {
						el.style.color='green'
					},2000);
					setTimeout(() => {
						done()//没有事件也要加上，表示动画已经结束
					},4000)
				},
				//两个参数,el和done，done是一个回调函数
				handleAfterEnter(el){
					el.style.color='#000'
				}
			}
		})
	</script>
</body>
```

# 使用Velocity库

1. 引包

```HTML
<script src="js/velocity.js"></script>	
```

2. 使用

```js
<body>
	<div id="root">
		<div>
			<transition
			name='fade'
			@before-enter='handleBeforeEnter'
			@enter='handleEnter'
			@after-enter='handleAfterEnter'
			>
				<div v-if="show">你好</div>
			</transition>
			<button @click="handleClick">切换</button>
		</div>
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
				},
				handleBeforeEnter(el){
					el.style.opacity = 0;
				},
				handleEnter(el,done){
					Velocity(el,{opacity: 1},{duration:1000,complete:done});//这里要加done函数
				},
				handleAfterEnter(el){
					alert('动画结束')
				}
			}
		})
	</script>
</body>
```