---
title: JQuery-03事件绑定与插件
date: 2019-10-27 22:06:15
tags: 
- JQuery
categories: 
- 前端
- JQuery
---

<center>
引言：

JQuery的事件绑定与插件
</center>

<!--more-->

-----

# 事件绑定

1. JQuery标准的绑定方式

`jqObj.事件方法(回调函数)`
```js
 //鼠标点击,进入,离开，可以链式编程
$("button").click(function () {
    alert(1);
}).mouseover(function () {
    alert("鼠标来了");
}).mouseleave(function () {
    alert("鼠标离开");
})
$("#input1").focus();
//让文本输入框获得焦点
$("#form1").submit();
//会让表单提交
```
2. on绑定事件/off解除绑定

`jqObj.on("事件名称",回调函数)`

`jqObj.off("事件名称")`

```js
$("button:eq(0)").on("click",function () {
    alert("我被点击了");
    //点击后取消他的点击事件
    $("button:eq(0)").off("click");
    //不传递参数会将组件上的所有事件全部解绑
})
```
3. 事件切换：toggle
> **1.9版本后移除**，可以使用插件`migrate`来恢复此功能

`jqObj.toggle(fn1,fn2...)`点击第一下执行fn1，第二下点fn2
```js
$("button").toggle(function () {
    alert(1);
},function () {
    alert(2);
})
//重复点击会来回切换fn1和fn2
```
# 插件

增强JQuery功能
，jQuery提供了两种插件的方式

1. `$.fn.extend(obj)`对象级别插件

    增强通过jQuery获取的对象的功能
2. `$.extend(obj)`全局级别插件

    增强jQuery对象自身的功能

## 小案例
使用插件实现全选的功能
```html
<button class="button1">全选</button>
<button class="button2">取消全选</button>
<input type="checkbox">一个
<input type="checkbox">二个
<input type="checkbox">三个
```

```js
$.fn.extend({
    check:function () {
        //选中所有的复选框
        this.prop("checked",true);
    },
    unCheck:function () {
        //取消全选
        this.prop("checked",false);
    }
});

$(".button1").click(function () {
    $("input[type='checkbox']").check();
});
$(".button2").click(function () {
    $("input[type='checkbox']").unCheck();
})
```

