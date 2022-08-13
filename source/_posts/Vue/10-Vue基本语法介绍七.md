---
title: 10.Vue基本语法介绍七
date: 2019-07-20 21:57:01
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

这一节介绍有关列表，列表渲染的基本知识，

加油加油！！
</center>
<!-- more -->

------------
# 列表渲染

## 用`v-for`把一个数组对应为一组元素

我们可以用 `v-for` 指令基于一个数组来渲染一个列表。`v-for` 指令需要使用 `item in items` 形式的特殊语法，其中 `items` 是源数据数组，而 `item` 则是被迭代的数组元素的别名。

```HTML
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
```
```javascript
var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

`v-for`块中我们可以访问所有父作用域的属性，`v-for`还支持一个可选的第二个参数，即当前项的索引值

```HTML
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```
```javascript
var example2 = new Vue({
  el: '#example-2',
  data: {
    parentMessage: 'Parent',
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```
其中的in我们可以换成of，这两个都可以

当我们需要包裹多个div时，又不想要多用一个div，
就可以使用模板占位符`<template><template>`来替代这个div

## 在`v-for`里使用对象

也可以用`v-for`来遍历一个对象的属性

```HTML
<ul id="v-for-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
```

```javascript
new Vue({
  el: '#v-for-object',
  data: {
    object: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
})
```
也可以提供键名
```HTML
<div v-for="(value, name) in object">
  {{ name }}: {{ value }}
</div>
```
在此之上还可以加上索引
```HTML
<div v-for="(value, name, index) in object">
  {{ index }}. {{ name }}: {{ value }}
</div>
```
在遍历对象时，会按 Object.keys() 的结果遍历，但是不能保证它的结果在不同的 JavaScript 引擎下都一致。

## 维护状态
当 Vue 正在更新使用 `v-for` 渲染的元素列表时，它默认使用“就地更新”的策略。

如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素

只适用于不依赖子组件状态或临时 DOM 状态 (例如：表单输入值) 的列表渲染输出

所以我们需要给每一项提供一个唯一的key属性来确保Vue能正确的跟踪每一个节点

```HTML
<div v-for="item in items" v-bind:key="item.id">
  <!-- 内容 -->
</div>
```
注意：

不要使用对象或数组之类的非基本类型值作为 v-for 的 key。请用字符串或数值类型的值。

## 数组更新检测

#### 变异方法

Vue 将被侦听的数组的变异方法进行了包裹，所以它们也将会**触发视图更新**。这些被包裹过的方法包括：

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

#### 替换数组

我们也可以用不变异的方法，返回一个新的数组代替旧的数组

- `filter()`
- `concat()`
- `slice()`

#### 注意事项
Vue不能检测到如下数组的变动
1. 当使用索引直接设置一个数组项的时候,例如：`vm.items[indexOfItem] = newValue`
2. 当修改数组长度的时候,例如`vm.items.length = newLength`


例如：

```javascript
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // 不是响应性的
vm.items.length = 2 // 不是响应性的
```

为了解决第一类问题，以下两种方式都可以实现和 `vm.items[indexOfItem] = newValue` 相同的效果，同时也将在响应式系统内触发状态更新：
```javascript
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```
**也可以使用 `vm.$set` 实例方法，**该方法是全局方法 Vue.set 的一个别名：
```
vm.$set(vm.items, indexOfItem, newValue)
```
为了解决第二类问题，你可以使用 splice：
```
vm.items.splice(newLength)
```

## 对象变更检测注意事项

Vue不能检测对象属性的添加或删除
```javascript
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` 现在是响应式的

vm.b = 2
// `vm.b` 不是响应式的
```
对于已经创建的实例，Vue 不允许动态添加根级别的响应式属性。

但是，可以使用 `Vue.set(object, propertyName, value)` 方法向嵌套对象添加响应式属性

例如：
```javascript
var vm = new Vue({
  data: {
    userProfile: {
      name: 'Anika'
    }
  }
})
```

可以添加一个新的 age 属性到嵌套的 userProfile 对象：

```javascript
Vue.set(vm.userProfile, 'age', 27)
```

还可以使用 vm.$set 实例方法，它只是全局 Vue.set 的别名：
```javascript
vm.$set(vm.userProfile, 'age', 27)
```

**有时你可能需要为已有对象赋值多个新属性**，比如使用 `Object.assign()` 或` _.extend()`。

在这种情况下，你应该用两个对象的属性创建一个新的对象

**不要**这样做

```javascript
Object.assign(vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```
而要这样做
```javascript
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

## 显示过滤/排序后的结果

我们想要显示一个数组经过过滤或排序后的版本，而不实际改变或重置原始数据

在这种情况下，可以创建一个**计算属性**，来返回过滤或排序后的数组。

```HTML
<li v-for="n in evenNumbers">{{ n }}</li>
```
```javascript
data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
computed: {
  evenNumbers: function () {
    return this.numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
```

在计算属性不适用的情况下 (例如，在嵌套 v-for 循环中) 你可以使用一个方法：
```HTMl
<li v-for="n in even(numbers)">{{ n }}</li>
```
```javascript
data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
methods: {
  even: function (numbers) {
    return numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
```
## 在 `v-for` 里使用值范围

`v-for` 也可以接受整数。在这种情况下，它会把模板重复对应次数
```HTML
<body>
	
	<div id='demo'>
  		<span v-for="n in 10">{{ n }} </span>
	</div>

	<script>
		new Vue({
			el:'#demo',
		})
	</script>
</body>
```

## 在 `<template>` 上使用 `v-for`
类似于 v-if，你也可以利用带有 v-for 的 <template> 来循环渲染一段包含多个元素的内容。比如：

```HTML
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

**v-if的优先级低于v-for**

当你只想为部分项渲染节点时，这种优先级的机制会十分有用，如下：

```HTML
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```
上面的代码将只渲染未完成的 todo

而如果你的目的是有条件地跳过循环的执行，那么可以将 v-if 置于外层元素 (或 <template>)上
```HTML
<ul v-if="todos.length">
  <li v-for="todo in todos">
    {{ todo }}
  </li>
</ul>
<p v-else>No todos left!</p>
```

## 能响应的刷新页面的方法总结

在各类方法中，共有三种方法可以使我们动态的更新页面上的对象和数组

1. 使用八种变异方法（只对数组有效）
2. 改变引用，更换地址
3. 使用全局的`Vue.set`方法(推荐)


## 在组件上使用`v-for`

在自定义组件上，你可以像在任何普通元素上一样使用 v-for 。

```HTML
<my-component v-for="item in items" :key="item.id"></my-component>
```
**key 现在是必须的**

任何数据都不会被自动传递到组件里，因为组件有自己独立的作用域。为了把迭代数据传递到组件里，我们要使用 prop

```HTML
<my-component
  v-for="(item, index) in items"
  v-bind:item="item"
  v-bind:index="index"
  v-bind:key="item.id"
></my-component>
```

不自动将 `item` 注入到组件里的原因是，这会使得组件与 `v-for` 的运作紧密耦合。明确组件数据的来源能够使组件在其他场合重复使用。

下面是一个完整的实例
```HTML
<div id="todo-list-example">
  <form v-on:submit.prevent="addNewTodo">
    <label for="new-todo">Add a todo</label>
    <input
      v-model="newTodoText"
      id="new-todo"
      placeholder="E.g. Feed the cat"
    >
    <button>Add</button>
  </form>
  <ul>
    <li
      is="todo-item"
      v-for="(todo, index) in todos"
      v-bind:key="todo.id"
      v-bind:title="todo.title"
      v-on:remove="todos.splice(index, 1)"
    ></li>
  </ul>
</div>
```
```javascript
Vue.component('todo-item', {
  template: '\
    <li>\
      {{ title }}\
      <button v-on:click="$emit(\'remove\')">Remove</button>\
    </li>\
  ',
  props: ['title']
})

new Vue({
  el: '#todo-list-example',
  data: {
    newTodoText: '',
    todos: [
      {
        id: 1,
        title: 'Do the dishes',
      },
      {
        id: 2,
        title: 'Take out the trash',
      },
      {
        id: 3,
        title: 'Mow the lawn'
      }
    ],
    nextTodoId: 4
  },
  methods: {
    addNewTodo: function () {
      this.todos.push({
        id: this.nextTodoId++,
        title: this.newTodoText
      })
      this.newTodoText = ''
    }
  }
})
```
