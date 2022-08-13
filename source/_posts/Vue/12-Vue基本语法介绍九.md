---
title: 12-Vue基本语法介绍九
date: 2019-07-21 15:33:03
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

这一节介绍有关表单元素，表单输入的基本知识，

加油加油！！
</center>
<!-- more -->

------------


# 表单输入绑定

## 基础用法

用`v-model`在表单`<input><textarea><select>`元素上创建**双向数据绑定**

注意：

v-model 会忽略所有表单元素的 `value、checked、selected` 特性的初始值而总是将 Vue 实例的数据作为数据来源。

你应该通过 JavaScript 在组件的 data 选项中声明初始值。

在文本区域插值 (`<textarea>{{text}}</textarea>`) 并不会生效，应用 v-model 来代替。
#### 文本
```HTML
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```
```HTML
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```

#### 复选框

单个复选框，绑定到布尔值：
```HTML
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
```

多个复选框，绑定到同一个数组：

```HTML
<div id='example-3'>
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>
```

```javascript
new Vue({
  el: '#example-3',
  data: {
    checkedNames: []
  }
})
```

#### 单选按钮
```HTML
<div id="example-4">
  <input type="radio" id="one" value="One" v-model="picked">
  <label for="one">One</label>
  <br>
  <input type="radio" id="two" value="Two" v-model="picked">
  <label for="two">Two</label>
  <br>
  <span>Picked: {{ picked }}</span>
</div>
```
```javascript
new Vue({
  el: '#example-4',
  data: {
    picked: ''
  }
})
```

#### 选择框

+ 单选时
```HTML
<div id="example-5">
  <select v-model="selected">
    <option disabled value="">请选择</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Selected: {{ selected }}</span>
</div>
```
```javascript
new Vue({
  el: '...',
  data: {
    selected: ''
  }
})
```
如果 `v-model` 表达式的初始值未能匹配任何选项，`<select>` 元素将被渲染为“未选中”状态。

在 iOS 中，这会使用户无法选择第一个选项。因为这样的情况下，iOS 不会触发 change 事件。

因此，更推荐像上面这样提供一个值为空的禁用选项。

+ 多选时
```HTML
<div id="example-6">
  <select v-model="selected" multiple style="width: 50px;">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <br>
  <span>Selected: {{ selected }}</span>
</div>
```
```javascript
new Vue({
  el: '#example-6',
  data: {
    selected: []
  }
})
//拖动才能多选
```
也可以用`v-for`来循环
```HTML
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
<span>Selected: {{ selected }}</span>
```
```javascript
new Vue({
  el: '...',
  data: {
    selected: 'A',
    options: [
      { text: 'One', value: 'A' },
      { text: 'Two', value: 'B' },
      { text: 'Three', value: 'C' }
    ]
  }
})
```

## 值绑定

对于单选按钮，复选框及选择框的选项，v-model 绑定的值通常是静态字符串 (对于复选框也可以是布尔值):

```HTML
<!-- 当选中时，`picked` 为字符串 "a" -->
<input type="radio" v-model="picked" value="a">

<!-- `toggle` 为 true 或 false -->
<input type="checkbox" v-model="toggle">

<!-- 当选中第一个选项时，`selected` 为字符串 "abc" -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```
但有时我们可能想把值绑定在Vue实例的一个动态属性，可以用`v-bind`实现

#### 复选框
```HTML
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
```
```javascript
// 当选中时
vm.toggle === 'yes'
// 当没有选中时
vm.toggle === 'no'
```

#### 单选按钮
```HTML
<input type="radio" v-model="pick" v-bind:value="a">
```
```javascript
// 当选中时
vm.pick === vm.a
```

#### 选择框的选项

```HTML
<select v-model="selected">
    <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```
```javascript
// 当选中时
typeof vm.selected // => 'object'
vm.selected.number // => 123
```


## 修饰符

#### `.lazy`
在默认情况下，`v-model` 在**每次** `input` 事件触发后  将输入框的值与数据进行同步 (除了上述输入法组合文字时)。

你可以添加 lazy 修饰符，从而转变为使用 change 事件进行同步：

```HTML
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg" >
```
#### `.number`
如果想自动将用户的输入值转为数值类型，可以给 `v-model` 添加 `number `修饰符：

```HTML
<input v-model.number="age" type="number">
```

#### `.trim`

如果要自动过滤用户输入的首尾空白字符，可以给 `v-model` 添加 trim 修饰符：

```HTML
<input v-model.trim="msg">
```
