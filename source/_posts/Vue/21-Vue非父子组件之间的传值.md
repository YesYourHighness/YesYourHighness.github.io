---
title: 21.Vue非父子组件之间的传值
date: 2019-07-24 21:39:31
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>
引言：

非父子之间，

该怎么传值呢
</center>

<!--more-->

------

## 非父子之间传值

使用总线机制，来给他们传值

这里用一个兄弟组件的例子


```js
<body>
	<div id="root">
		<child content="Dell"></child>
		<child content="Lee"></child>
		
	</div>

	<script>
		Vue.prototype.bus = new Vue()
		//给Vue的prototype挂一个bus的属性，
		//因为每个组件都是Vue创建的，所以创建的每个组件都有了一个bus的属性

		Vue.component('child',{
			data(){
				return {
					selfContent: this.content
					//拷贝一份传来的数据，单向数据原则！
				}
			},
			props:{
				content: String
			},
			template:'<div @click="handleClick">{{selfContent}}</div>',
			//在子组件内绑定一个事件
			methods:{
				handleClick(){
					this.bus.$emit('change',this.content)
					//this.bus指这个组件的bus属性，
					//因为这个属性又是一个Vue的实例，所以它又有了$emit这个方法
					//所以可以这么使用
				}
			},
			mounted(){
				var this_ = this;
				this.bus.$on('change',function(msg){
					this_.selfContent = msg
					//这里的this是函数内的this，发生了变化,所以我们得在外部保存this指针
				})
				//挂载到mouted这个生命钩子上去监听
				//同样，因为this.bus是一个Vue的实例，所以他也有on方法
			}	
		})

		var vm = new Vue({
			el:'#root'
		})
	</script>
</body>

```