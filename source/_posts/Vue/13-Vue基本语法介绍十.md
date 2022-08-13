---
title: 13-Vue基本语法介绍十
date: 2019-07-21 15:33:29
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

这一节介绍Vue组件的基本知识，

加油加油！！
</center>
<!-- more -->

-----------------
# 组件基础

## 基本示例

```javascript
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```
组件是可以重复使用的Vue实例，且带有一个名字，例如`<button-counter>`。

我们可以在一个**通过`new Vue`创建的Vue根实例**中把这个组件作为自定义元素来使用
```HTML
<div id="components-demo">
  <button-counter></button-counter>
</div>
```
```javascript
new Vue({ el: '#components-demo' })
```
## 组件的复用

组件可以任意次复用
```HTML
<div id="components-demo">
  <button-counter></button-counter>
  <button-counter></button-counter>
  <button-counter></button-counter>
</div>
```
注意当点击按钮时，每个组件都会各自独立维护它的 count。

因为你每用一次组件，就会有一个它的新实例被创建。

#### `data`必须是一个函数
当我们定义这个 `<button-counter>` 组件时，你可能会发现它的 data 并不是像这样直接提供一个对象：
```javascript
data: {
  count: 0
}
```
取而代之的是，**一个组件的 data 选项必须是一个函数**，因此每个实例可以维护一份被返回对象的独立的拷贝：
```javascript
data: function () {
  return {
    count: 0
  }
}
```
如果 Vue 没有这条规则，点击一个按钮就可能会影响到其它所有实例

## 组件的组织

通常一个应用会以一棵嵌套的组件树的形式来组织：
![image](https://cn.vuejs.org/images/components.png)
例如，你可能会有页头、侧边栏、内容区等组件，每个组件又包含了其它的像导航链接、博文之类的组件。

为了能在模板中使用，这些组件必须先**注册**以便 Vue 能够识别。

这里有两种组件的注册类型：**全局注册和局部注册**。

至此，我们的组件都只是通过 Vue.component **全局注册**的：
```javascript
Vue.component('my-component-name', {
  // ... options ...
})
```
全局注册的组件可以用在其被注册之后的任何 (通过 new Vue) 新创建的 Vue 根实例，也包括其组件树中的所有子组件的模板中。

## 通过 Prop 向子组件传递数据
Prop 是你可以在组件上注册的一些自定义特性。

当一个值传递给一个 prop 特性的时候，它就变成了那个组件实例的一个属性。

为了给博文组件传递一个标题，我们可以用一个 `props` 选项将其包含在该组件可接受的 prop 列表中：

```javascript
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})
```
一个组件默认可以拥有任意数量的 prop，任何值都可以传递给任何 prop。

在上述模板中，你会发现我们能够在组件实例中访问这个值，就像访问 `data` 中的值一样。

一个 prop 被注册之后，你就可以像这样把数据作为一个自定义特性传递进来：
```HTML
<blog-post title="My journey with Vue"></blog-post>
<blog-post title="Blogging with Vue"></blog-post>
<blog-post title="Why Vue is so fun"></blog-post>
```

然而在一个典型的应用中，你可能在 data 里有一个博文的数组：

```javascript
new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' }
    ]
  }
})
```
```HTML
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
></blog-post>
```

## 单个根元素

当构建一个`<blog-post>` 组件时，你的模板最终会包含的东西远不止一个标题

最最起码，你会包含这篇博文的正文：

```HTML
<h3>{{ title }}</h3>
<div v-html="content"></div>
```

然而如果你在模板中尝试这样写，Vue 会显示一个错误，并解释道 **every component must have a single root element** (每个组件必须只有一个根元素)。

你可以将模板的内容包裹在一个父元素内，来修复这个问题

```HTML
<div class="blog-post">
  <h3>{{ title }}</h3>
  <div v-html="content"></div>
</div>
```

看起来当组件变得越来越复杂的时候，我们的博文不只需要标题和内容，还需要发布日期、评论等等。为每个相关的信息定义一个 prop 会变得很麻烦：

```HTML
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
  v-bind:content="post.content"
  v-bind:publishedAt="post.publishedAt"
  v-bind:comments="post.comments"
></blog-post>
```
所以是时候重构一下这个` <blog-post>` 组件了，让它变成接受一个单独的` post` prop：

```HTML
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:post="post"
></blog-post>
```
```javascript
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <div v-html="post.content"></div>
    </div>
  `
})
```
现在，不论何时为 post 对象添加一个新的属性，它都会自动地在 `<blog-post>` 内可用。

## 监听子组件事件

在我们开发 `<blog-post>` 组件时，它的一些功能可能要求我们和父级组件进行沟通。

例如我们可能会引入一个辅助功能来放大博文的字号，同时让页面的其它部分保持默认的字号。

在其父组件中，我们可以通过添加一个 `postFontSize `数据属性来支持这个功能：

```javascript
new Vue({
  el: '#blog-posts-events-demo',
  data: {
    posts: [/* ... */],
    postFontSize: 1
  }
})
```
它可以在模板中用来控制所有博文的字号：
```HTML
<div id="blog-posts-events-demo">
  <div :style="{ fontSize: postFontSize + 'em' }">
    <blog-post
      v-for="post in posts"
      v-bind:key="post.id"
      v-bind:post="post"
    ></blog-post>
  </div>
</div>
```
现在我们在每篇博文正文之前添加一个按钮来放大字号：
```javascript
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <button>
        Enlarge text
      </button>
      <div v-html="post.content"></div>
    </div>
  `
})
```
问题是这个按钮不会做任何事：
```HTML
<button>
  Enlarge text
</button>
```
当点击这个按钮时，我们需要告诉父级组件放大所有博文的文本。

幸好 Vue 实例提供了一个自定义事件的系统来解决这个问题。

父级组件可以像处理 native DOM 事件一样通过 `v-on `监听子组件实例的任意事件：
```HTML
<blog-post
  ...
  v-on:enlarge-text="postFontSize += 0.1"
></blog-post>
```
同时子组件可以通过调用内建的 `$emit `方法 并传入事件名称来触发一个事件：
```HTML
<button v-on:click="$emit('enlarge-text')">
  Enlarge text
</button>
```
有了这个 `v-on:enlarge-text="postFontSize += 0.1"` 监听器，父级组件就会接收该事件并更新 `postFontSize `的值。

#### 使用事件抛出一个值

有时候我们需要用一个事件来抛出一个特定的值。

例如我们可能想让`<blog-post>`组件决定他的文本要放大多少，这时我们可以用`$emit`的第二个参数来提供这个值：

```HTML
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```
然后当在父级组件监听这个事件的时候，我们可以通过`$event`访问到被抛出的这个值

```HTML
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```
或者,这个事件处理函数是一个方法
```HTML
<blog-post
  ...
  v-on:enlarge-text="onEnlargeText"
></blog-post>
```
那么这个值**会作为第一个参数**传入这个方法
```javascript
methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
```

#### 在组件上使用`v-model`

自定义事件也可以作用于创建支持`v-model`的自定义输入组件

```HTML
<input v-model="searchText">
```
等价于:
```HTML
<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>
```
当作用在组件上时，`v-model`则会这样

```HTML
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
></custom-input>
```

为了让他正常工作这个组件内的`<input>`必须：
- 将其`value`特性绑定到一个名叫`value`的prop上
- 在其`input`事件被触发时，将新的值通过自定义的`input`事件抛出

完成的代码是这样:

```javascript
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
```

现在`v-model`就能在这个组件上完美的工作了

```
<custom-input v-model="searchText"></custom-input>
```

## 通过插槽分发内容

和HTML元素一样，我们经常需要向一个组件传递内容

```HTML
<alert-box>
  Something bad happened.
</alert-box>
```

Vue自定义的`<slot>`元素会非常简单

```javascript
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```
