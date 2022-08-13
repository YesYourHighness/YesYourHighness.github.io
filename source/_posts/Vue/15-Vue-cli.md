---
title: 14.Vue-cli
date: 2019-07-22 21:52:56
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>
引言：

开始认识vue-cli,进行工程化开发把

从安装到设置路由，学习风格...

加油加油!


</center>

<!-- more -->
-----------

# Vue-cli

## 认识Vue-cli

在cmd中输入如下代码

安装
```
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```
创建一个项目
```
vue create my-project
# OR
vue ui
```
进入选项选择

选择Manually select features后我们手动配置

进入选择组件，以下就是常用组件
```
? Check the features needed for your project:
 (*) Babel
 ( ) TypeScript
 ( ) Progressive Web App (PWA) Support
 (*) Router             路由组件
 (*) Vuex               
 (*) CSS Pre-processors css预编译组件
>(*) Linter / Formatter
 ( ) Unit Testing
 ( ) E2E Testing
```
是否使用历史    -yes

是否使用ESlint  -Airbnb

pick css?       -SCSS

是否保存到将来的项目 -no

进入所建目录，运行

```
cd xxx
npm run server
```


## 运行成功我们来看目录
- node_modules (依赖文件)
- src (放置源文件)**关键**
    + main.js
    + views (内部放置各个视图)
   
- public (公共资源)**关键**
    + index.html (入口文件)
- package.json (对整个项目的解释说明)**关键**

---------
**index.html**内有挂载点，默认id为app

**main.js**创建了Vue实例，而且引入了两个东西
- router路由
- store这个是Vuex的引入，管理组件之间的状态


内部代码如下：

```js
new Vue({
  router,
  store,
  render: h => h(App),
}).$mount('#app');//这里就相当于原本index文件代码的el:'#app'
```


## Vue组件化思想

### 为什么要组件化？

实现了模块的**复用**，利于维护

可以高效的执行

降低单页面复杂难度


### 怎么拆分

300行原则

复用原则

业务复杂性原则

### 带来的问题用什么解决

组件状态管理 Vuex

多组件混合使用，业务复杂 vue-router

组件之间传参、消息、事件管理 props,emit/on,bus

## 风格指南

详情点击这里，[Vue官方的风格指南](https://cn.vuejs.org/v2/style-guide/)

## Vue-router 路由

怎么把一个组件拓展到路由内呢？

1. 新建一个组件`Info.vue`
```HTML
<template>
  <div>
    怎么设置路由
  </div>
</template>

<script>
  export default {
    name: ''
  };
</script>

<style scoped>

</style>
```
2. 在src文件下的router.js内的routers里面添加这个组件的信息,并在开头记住引包

```js
import Info from './views/Info.vue';
```

```js
{
  path: '/',
  name: 'home',
  component: Home,
},
```

3. 打开App.vue文件，设置页面上的链接，设置`router-link`


完成


-----


## Vuex

### Vuex是什么

为vue.js开发的**状态管理模式**

组件状态**集中管理**

组件状态改变遵循**统一的规则**


### 怎么使用Vuex
src目录下的`store.js`文件：
```js
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    //组件的状态
  },
  mutations: {
    //唯一可以改变的vuex状态的方法集
  },
  actions: {

  },
});
```

步骤:
1. 在自建的组件Info.vue内添加以下内容

```js
<template>
  <div>
    怎么使用Vuex
    <button @click="InfoAdd()">添加</button>
    <!--1.在这里添加一个事件吧-->
  </div>
</template>

<script>
  import store from '@/store'
  //2.引包，这里的 @ 是src目录
  export default {
    name: 'Info',
    store,//3.还要在这里声明一下，在这里引入
    methods: {
      InfoAdd() {
        console.log('add Event from info')
      }
    }
  };
</script>

<style scoped>

</style>
```

2. 打开`store.js`文件

```js
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    count: 0,//1. 这里添加状态
  },
  mutations: {
    increase() {//2. 这里添加方法
      this.state.count += 1;
    },
  },
  actions: {},
});
```

```
<template>
  <div>
    怎么使用Vuex
    <button @click="InfoAdd()">添加</button>
  </div>
</template>

<script>
  import store from '@/store'
  export default {
    name: 'Info',
    store,
    methods: {
      InfoAdd() {
        console.log('add Event from info')
        store.commit('increase')
        //3. 使用 commit 方法，括号内参数是第二步的方法名称
      }
    }
  };
</script>
<style scoped>
</style>
```
这样我们就完成了Vuex的使用，那么怎么给另一个页面传值呢
```js
<template>
  <div class="about">
    <h1>This is an about page</h1>
    {{msg}}
    <!--1. 在这里插入显示的值-->
  </div>
</template>

<script>
  import store from '@/store'
  //2. 引入vuex
  export default {
    name:'about',
    store,//3. 这里也要声明
    data(){
      return{
        msg: store.state.count
        //4. 直接在这里返回就行了
      }
    }
  }
</script>
```

