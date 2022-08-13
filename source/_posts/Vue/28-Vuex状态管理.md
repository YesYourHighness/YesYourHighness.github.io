---
title: 28.Vuex状态管理
date: 2019-07-27 21:55:02
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>

引言：

Vuex是什么？

</center>

<!--more-->

-------

# Vuex是什么

专为Vue.js开发的**状态管理模式**

集中式存储管理应用的所有组件状态，

并以对应的规则保证状态以一种可预测的方式发生变化


## 什么是状态管理模式

```js
new Vue({
//state
  data(){
  return {
  	count: 0
  }
  },
  //view
  template:'<div>{{count}}</div>',
  //actions
  methods:{
  increment(){
  	this.count++
  }
}
})
```

- state,驱动应用的数据源
- view,以声明方式将state映射到视图
- actions,响应在view上的用户输入导致的状态变化

vue特性之一就是 "单向数据流"
![image](https://vuex.vuejs.org/flow.png)
但是，当我们需要**多个组件共享状态时**

单向数据流的概念很容易被破坏：
- 多视图依赖于同一个状态
- 不同的行为需要变更同一个状态

一般的方法难以维护

所以，Vue把共享状态抽取出来，以一个全局单例模式管理

不管在树的哪一个位置，任何组件都能获取状态或者触发行为

# Vuex使用

Vuex核心是store(仓库),store内有着应用中大部分的状态state

## Vuex与全局对象的区别

- Vuex的状态存储是响应式的
- 不能直接改变store的状态，**唯一只能**通过显式的提交mutation

## 简单的Store

```js
// 如果在模块化构建系统中，请确保在开头调用了 Vue.use(Vuex)
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```
可以通过`store.state`获取状态对象

通过`store.commit`触发状态变更

```js
store.commit('increment')//加方法的名称

console.log(store.state.count) // -> 1
```

再次强调，

我们通过提交 mutation 的方式，而非直接改变 store.state.count，是因为我们想要更明确地追踪到状态的变化

# 核心概念

## State

### 单一状态树

Vue只用一个对象包含全部的应用层级状态

每个应用仅仅包含一个store实例

单一状态树方便我们管理

### 在Vue组件中获得Vuex状态

从 store 实例中读取状态

最简单的方法就是在计算属性中返回某个状态：

```js
// 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```

另一种更好的办法

Vuex 通过 `store` 选项，提供了一种机制将状态从根组件“注入”到每一个子组件中（需调用 `Vue.use(Vuex)`）：

```js
const app = new Vue({
  el: '#app',
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})
```
把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件

通过在根实例中注册 `store` 选项，
该 `store` 实例会注入到根组件下的所有子组件中，
且子组件能通过 `this.$store` 访问到。

让我们更新下 Counter 的实现：

```js
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

## Getter

有时我们需要从store中的state派生出一些状态

例如对列表进行过滤并计数
```js
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```
如果有多个组件需要用到此属性，

我们要么复制这个函数，

或者抽取到一个共享函数然后在多处导入它——无论哪种方式都不是很理想。

Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。

就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

Getter 接受 state 作为其第一个参数：

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

### 通过属性访问

Getter 会暴露为 `store.getters` 对象，你可以以属性的形式访问这些值：

```js
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
```

Getter 也可以接受其他 getter 作为第二个参数：

```js
getters: {
  // ...
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}
```
```js
store.getters.doneTodosCount // -> 1
```
我们很容易的在任何组件中使用它

```js
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```
注意，getter 在通过属性访问时是作为 Vue 的响应式系统的一部分缓存其中的

### 通过方法访问
通过让 getter 返回一个函数，来实现给 getter 传参。在你对 store 里的数组进行查询时非常有用。
```js
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

```js
store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```
注意，getter 在通过方法访问时，每次都会去进行调用，而不会缓存结果。

## Mutation
更改 Vuex 的 store 中的状态的**唯一方法**是提交 mutation

Vuex 中的 mutation 非常类似于事件：

每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。

这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数：

```
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
```

要唤醒一个 mutation handler，你需要以相应的 type 调用 store.commit 方法：

```js
store.commit('increment')
```

### 提交载荷（Payload）
你可以向 store.commit 传入额外的参数，即 mutation 的 载荷（payload）：

```js
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}
```
```js
store.commit('increment', 10)
```
在大多数情况下，载荷应该是一个对象，这样可以包含多个字段并且记录的 mutation 会更易读：
```js
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```
```js
store.commit('increment', {
  amount: 10
})
```

### 对象风格的提交方式
提交 mutation 的另一种方式是直接使用包含 type 属性的对象：
```js
store.commit({
  type: 'increment',
  amount: 10
})
```
当使用对象风格的提交方式，整个对象都作为载荷传给 mutation 函数，
因此 handler 保持不变：
```js
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

### Mutation需要遵守Vue的响应规则

1. 最好提前在你的store中初始化所有的所需属性
2. 在需要在对象上添加新属性时，应该
    - 使用`Vue.set(obj, 'newProp', 123)`
    - 或者以新对象替换老对象,例如利用 stage-3 的对象展开运算符
    `state.obj = { ...state.obj, newProp: 123 }`

### 在组件中提交 Mutation
你可以在组件中使用 `this.$store.commit('xxx')` 提交 mutation    

### 在 Vuex 中，mutation 都是 同步事务 
```js
store.commit('increment')
// 任何由 "increment" 导致的状态变更都应该在此刻完成。
```

## Action

Action和Mutation的区别:

- Action 提交的是 mutation ,而不是直接变更状态
- Action 可以包含任意异步操作

简单的action例子
```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```
Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，

因此你可以调用 context.commit 提交一个 mutation，

或者通过 context.state 和 context.getters 来获取 state 和 getters。

实践中，我们会经常用到 ES2015 的 参数解构 来简化代码（特别是我们需要调用` commit` 很多次的时候）：
```js
actions: {
  increment ({ commit }) {
    commit('increment')
  }
}
```
### 分发Action

Action 通过 store.dispatch 方法触发：
```js
store.dispatch('increment')
```

乍一眼看上去感觉多此一举，我们直接分发 mutation 岂不更方便？

实际上并非如此，还记得 mutation 必须同步执行这个限制么？

Action 就不受约束！我们可以在 action 内部执行异步操作：
```js
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```
Actions 支持同样的载荷方式和对象方式进行分发：
```
// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
})

// 以对象形式分发
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```
来看一个更加实际的购物车示例，涉及到调用异步 API 和分发多重 mutation：
```js
actions: {
  checkout ({ commit, state }, products) {
    // 把当前购物车的物品备份起来
    const savedCartItems = [...state.cart.added]
    // 发出结账请求，然后乐观地清空购物车
    commit(types.CHECKOUT_REQUEST)
    // 购物 API 接受一个成功回调和一个失败回调
    shop.buyProducts(
      products,
      // 成功操作
      () => commit(types.CHECKOUT_SUCCESS),
      // 失败操作
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```
注意我们正在进行一系列的异步操作，并且通过提交 mutation 来记录 action 产生的副作用（即状态变更）

### 在组件中分发Action

你在组件中使用 `this.$store.dispatch('xxx')` 分发 action

