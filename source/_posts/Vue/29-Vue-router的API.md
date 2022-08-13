---
title: 29.Vue-router的API
date: 2019-07-27 21:55:48
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>
引言：

vue-router的API参考


</center>

<!--more-->

-----

取自[官方文档](https://router.vuejs.org/zh/api/#router-link)


# `router-link`

功能：支持用户在具有路由功能的应用中 (点击) 导航

`<router-link>` 比起写死的` <a href="...">` 会好一些，理由如下：

- 无论是 HTML5 history 模式还是 hash 模式，它的表现行为一致，所以，当你要切换路由模式，或者在 IE9 降级使用 hash 模式，无须作任何变动。

- 在 HTML5 history 模式下，router-link 会守卫点击事件，让浏览器不再重新加载页面。

- 当你在 HTML5 history 模式下使用 base 选项之后，所有的 to 属性都不需要写 (基路径) 了

## 将激活 class 应用在外层元素
有时候我们要让激活 class 应用在外层元素，而不是 `<a> `标签本身，那么可以用 `<router-link>` 渲染外层元素，包裹着内层的原生 <a> 标签：
```HTML
<router-link tag="li" to="/foo">
  <a>/foo</a>
</router-link>
```
在这种情况下，`<a>` 将作为真实的链接 (它会获得正确的` href` 的)，而 "激活时的 CSS 类名" 则设置到外层的 `<li>`。

## Props

### to
类型: `string | Location`

`required`

表示目标路由的链接。

当被点击后，内部会立刻把 `to` 的值传到 `router.push()`，

所以这个值可以是一个字符串或者是描述目标位置的对象。
```HTML
<!-- 字符串 -->
<router-link to="home">Home</router-link>
<!-- 渲染结果 -->
<a href="home">Home</a>

<!-- 使用 v-bind 的 JS 表达式 -->
<router-link v-bind:to="'home'">Home</router-link>

<!-- 不写 v-bind 也可以，就像绑定别的属性一样 -->
<router-link :to="'home'">Home</router-link>

<!-- 同上 -->
<router-link :to="{ path: 'home' }">Home</router-link>

<!-- 命名的路由 -->
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>

<!-- 带查询参数，下面的结果为 /register?plan=private -->
<router-link :to="{ path: 'register', query: { plan: 'private' }}">Register</router-link>
```

### replace
类型: `boolean`

默认值: `false`

设置` replace `属性的话，当点击时，会调用 `router.replace()` 而不是 `router.push()`，于是导航后不会留下 history 记录。
```HTML
<router-link :to="{ path: '/abc'}" replace></router-link>
```
### append
类型:` boolean`

默认值: `false`

设置 `append `属性后，则在当前 (相对) 路径前添加基路径。例如，我们从` /a` 导航到一个相对路径` b`，如果没有配置 `append`，则路径为 `/b`，如果配了，则为` /a/b`
```HTML
<router-link :to="{ path: 'relative/path'}" append></router-link>
```

### tag
类型: `string`

默认值:` "a"`

有时候想要 `<router-link>` 渲染成某种标签，例如` <li>`。 于是我们使用` tag `prop 类指定何种标签，同样它还是会监听点击，触发导航。
```HTML
<router-link to="/foo" tag="li">foo</router-link>
<!-- 渲染结果 -->
<li>foo</li>
```

### active-class
类型: `string`

默认值: `"router-link-active"`

设置 链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 `linkActiveClass` 来全局配置。

### exact
类型:` boolean`

默认值: `false`

"是否激活" 默认类名的依据是 inclusive match (全包含匹配)。

举个例子，如果当前的路径是 /a 开头的，那么 `<router-link to="/a"> `也会被设置 CSS 类名。

按照这个规则，每个路由都会激活`<router-link to="/">`！想要链接使用 "exact 匹配模式"，则使用 exact 属性：
```HTML
<!-- 这个链接只会在地址为 / 的时候被激活 -->
<router-link to="/" exact>
```


### event
类型: `string | Array<string>`

默认值: `'click'`

声明可以用来触发导航的事件。可以是一个字符串或是一个包含字符串的数组。

### exact-active-class
类型: `string`

默认值: `"router-link-exact-active"`

配置当链接被精确匹配的时候应该激活的 class。注意默认值也是可以通过路由构造函数选项 linkExactActiveClass 进行全局配置的。

# `<router-view>`

`<router-view> `组件是一个 functional 组件，

渲染路径匹配到的视图组件。

`<router-view>` 渲染的组件还可以内嵌自己的 `<router-view>`，根据嵌套路径，渲染嵌套组件。

其他属性 (非 router-view 使用的属性) 都直接传给渲染的组件，

很多时候，每个路由的数据都是包含在路由参数中。

因为它也是个组件，所以可以配合 `<transition>` 和 `<keep-alive>` 使用。如果两个结合一起用，要确保在内层使用 `<keep-alive>`：

```HTML
<transition>
  <keep-alive>
    <router-view></router-view>
  </keep-alive>
</transition>
```

## Props
### name
类型: `string`

默认值: `"default"`

如果 <router-view>设置了名称，

则会渲染对应的路由配置中 components 下的相应组件

# Router构建选项

## routes
类型:` Array<RouteConfig>`

`RouteConfig` 的类型定义：
看看就好
```
declare type RouteConfig = {
  path: string;
  component?: Component;
  name?: string; // 命名路由
  components?: { [name: string]: Component }; // 命名视图组件
  redirect?: string | Location | Function;
  props?: boolean | Object | Function;
  alias?: string | Array<string>;
  children?: Array<RouteConfig>; // 嵌套路由
  beforeEnter?: (to: Route, from: Route, next: Function) => void;
  meta?: any;

  // 2.6.0+
  caseSensitive?: boolean; // 匹配规则是否大小写敏感？(默认值：false)
  pathToRegexpOptions?: Object; // 编译正则的选项
}
```
## mode
类型:` string`

默认值: `"hash" (浏览器环境) | "abstract" (Node.js 环境)`

可选值: `"hash" | "history" | "abstract"`

配置路由模式:

`hash`: 使用 URL hash 值来作路由。支持所有浏览器，包括不支持 HTML5 History Api 的浏览器。

`history`: 依赖 HTML5 History API 和服务器配置

`abstract`: 支持所有 JavaScript 运行环境，如 Node.js 服务器端。**如果发现没有浏览器的 API，路由会自动强制进入这个模式**。

## base
类型: `string`

默认值: `"/"`

应用的基路径。

例如，如果整个单页应用服务在 /app/ 下，然后 base 就应该设为 "/app/"。

## linkActiveClass
类型: `string`

默认值: `"router-link-active"`

全局配置 `<router-link>` 的默认“激活 class 类名”

## linkExactActiveClass
类型: `string`

默认值: `"router-link-exact-active"`

全局配置` <router-link>` 精确激活的默认的 class

## scrollBehavior
类型: `Function`

签名:
```
type PositionDescriptor =
  { x: number, y: number } |
  { selector: string } |
  ?{}

type scrollBehaviorHandler = (
  to: Route,
  from: Route,
  savedPosition?: { x: number, y: number }
) => PositionDescriptor | Promise<PositionDescriptor>
```

## parseQuery / stringifyQuery
类型:` Function`

提供自定义查询字符串的解析/反解析函数。覆盖默认行为。

## fallback
类型: ·boolean·

当浏览器不支持 history.pushState 控制路由是否应该回退到 hash 模式。默认值为 true。

在 IE9 中，设置为 false 会使得每个 router-link 导航都触发整页刷新。它可用于工作在 IE9 下的服务端渲染应用，因为一个 hash 模式的 URL 并不支持服务端渲染。

# Router实例属性

- router.app

类型: `Vue instance`

配置了 `router` 的 Vue 根实例。

- router.mode

类型:` string`

路由使用的模式。

- router.currentRoute

类型: `Route`

当前路由对应的路由信息对象



# Router实例方法

- router.beforeEach
- router.beforeResolve
- router.afterEach

函数签名：
```js
router.beforeEach((to, from, next) => {
  /* must call `next` */
})

router.beforeResolve((to, from, next) => {
  /* must call `next` */
})

router.afterEach((to, from) => {})
```

在 2.5.0+ 这三个方法都返回一个移除已注册的守卫/钩子的函数

- router.push
- router.replace
- router.go
- router.back
- router.forward

函数签名:
```
router.push(location, onComplete?, onAbort?)
router.replace(location, onComplete?, onAbort?)
router.go(n)
router.back()
router.forward()
```
# 路由对象

一个路由对象 (route object) 表示当前激活的路由的状态信息，

包含了当前 URL 解析得到的信息，还有 URL 匹配到的路由记录 (route records)。

路由对象是不可变 (immutable) 的，每次成功的导航后都会产生一个新的对象。

路由对象出现在多个地方:

- 在组件内，即 `this.$route`

- 在 `$route` 观察者回调内

- `router.match(location)` 的返回值

- 导航守卫的参数：
```
router.beforeEach((to, from, next) => {
  // `to` 和 `from` 都是路由对象
})
```
- scrollBehavior 方法的参数:
```
const router = new VueRouter({
  scrollBehavior (to, from, savedPosition) {
    // `to` 和 `from` 都是路由对象
  }
})
```
## 路由对象属性
- $route.path

类型: `string`

字符串，对应当前路由的路径，总是解析为绝对路径，如 `"/foo/bar"`。

- $route.params

类型: `Object`

一个 key/value 对象，包含了动态片段和全匹配片段，如果没有路由参数，就是一个空对象。

- $route.query

类型:` Object`

一个 key/value 对象，表示 URL 查询参数。例如，对于路径 `/foo?user=1`，则有 `$route.query.user == 1`，如果没有查询参数，则是个空对象。

- $route.hash

类型: `string`

当前路由的 hash 值 (带 #) ，如果没有 hash 值，则为空字符串。

- $route.fullPath

类型: string

完成解析后的 URL，包含查询参数和 hash 的完整路径。

- $route.matched

类型: `Array<RouteRecord>`

一个数组，包含当前路由的所有嵌套路径片段的路由记录 。

路由记录就是 routes 配置数组中的对象副本 (还有在 `children` 数组)。
```
const router = new VueRouter({
  routes: [
    // 下面的对象就是路由记录
    { path: '/foo', component: Foo,
      children: [
        // 这也是个路由记录
        { path: 'bar', component: Bar }
      ]
    }
  ]
})
```
当 URL 为 `/foo/bar`，`$route.matched` 将会是一个包含从上到下的所有对象 (副本)。

- $route.name

当前路由的名称，如果有的话。(查看命名路由)

- $route.redirectedFrom

如果存在重定向，即为重定向来源的路由的名字