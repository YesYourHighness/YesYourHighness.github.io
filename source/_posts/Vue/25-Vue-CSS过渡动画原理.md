---
title: 25.Vue-CSS过渡动画原理
date: 2019-07-26 22:00:24
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

介绍Vue中的css过渡动画

各种情况下的过渡

动画封装

</center>

<!--more-->

------------

# 原理

我们可以用`transition`标签来包裹我们想要添加css动画的部分

原理：
```HTML
<body>
	<div id="root">
		<transition name='fade'>
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
```
当一个元素被`transition`标签包裹的时候，Vue会自动的分析css的样式，并且构建一个css流程：

这是出现时的流程
1. 在动画即将被执行的一瞬间，会向内部的标签增加两个class：
- fade-enter
- fade-enter-active

```css
<style>
	.fade-enter{
		opacity: 0;
	}
	.fade-enter-active{
		transition: opacity 1s;
	}
</style>
```
这就是一个淡入的效果

2. 当运行至第二帧执行时，去掉fade-enter，增添一个fade-enter-to的class


3. 执行到结束的时候，会把所有的class全部去掉

都以fade为开头是因为我给`transition`起名为fade，默认的名字以v开头即可

这是离开时的流程:

同理

1. 填入两个clss
- fade-leave
- fade-leave-active

2. 去掉fade-enter，填入fade-enter-to
3. 去掉所有的class

```
.fade-leave-to{
	opacity: 0
  }
.fade-leave-active{
	transition: opacity 1s;
}
```

不论v-if,v-show,还是过度组件，都会有过渡动画效果


# 多元素

```css
.v-enter, .v-leave-to{
	opacity: 0;
}
.v-enter-active,.v-leave-active{
	transition: opacity 1s;
}
```

```js
<body>
	<div id="root">
		<div>
			<transition>
				<div v-if="show" key='hello'>你好</div>
				<div v-else key='bye'>Bye World</div>
        <!--这里要加不同的key值，因为vue会对DOM复用-->
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
				Velocity(el,{opacity: 1},{duration:1000,complete:done});
			},
			handleAfterEnter(el){
				alert('动画结束')
			}
		}
	})
</script>
</body>
```

还可以在`transition`上面加`mode`属性

mode属性有两个值
- mode="in-out" 两个元素先进后出
- mode="out-in" 两个元素先出后进

```HTML
<transition mode='out-in'>
	<div v-if="show" key='hello'>你好</div>
	<div v-else key='bye'>Bye World</div>
</transition>
```

# 多组件

我们需要用到动态组件

其他相同

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>	</title>
	<script src="./js/vue.js"></script>
	<link rel="stylesheet" href="js/animate.css">
	<script src="js/velocity.js"></script>	
	<style>
		.v-enter, .v-leave-to{
			opacity: 0;
		}
		.v-enter-active,.v-leave-active{
			transition: opacity 0.5s;
		}
	</style>
</head>
<body>
	<div id="root">
			<transition mode='out-in'>
				<component :is='type'></component>
			</transition>
			<button @click="handleClick">切换</button>
	</div>


	<script>
		Vue.component("child-one",{
			template: '<div>child-one</div> '
		})
		Vue.component("child-two",{
			template: '<div>child-two</div> '
		})
		let vm = new Vue({
			el:'#root',
			data:{
				type:'child-one'
			},
			methods:{
				handleClick(){
					this.type = this.type === 'child-one'?'child-two':'child-one'
				}
			}

		})
	</script>
</body>
</html>
```

# 列表过渡

使用一个新的标签`<transition-group>`来实现列表的渲染

```HTML
<div id="root">
	<transition-group>
	<div v-for='item of list' :key='item.id'>
		{{item.title}}
	</div>
	</transition-group>
	<button @click='handleClick'>Add</button>
</div>
```

```css
.v-enter, .v-leave-to{
	opacity: 0;
}
.v-enter-active,.v-leave-active{
	transition: opacity 0.5s;
}
```

```js
let count = 0;

let vm = new Vue({
	el:'#root',
	data:{
		list:[]
	},
	methods:{
		handleClick(){
			this.list.push({
				id:count++,
				title:'hello'
			})
		}
	}
})
```

# 封装

```js
Vue.component('fade',{
	props:['show'],
	template:'<transition @before-enter="handleBeforeEnter" @enter="handleEnter"><slot v-if="show"></slot></transition>',
	methods:{
		handleBeforeEnter(el){
			el.style.color = 'red'
		},
		handleEnter(el,done){
			setTimeout(()=>{
				el.style.color ='green'
			},2000)
		}
	}
})
```
这样封装起来，我们就可以直接使用了
```HTML
<fade :show='show'>
	<div>你好</div>
</fade>
```