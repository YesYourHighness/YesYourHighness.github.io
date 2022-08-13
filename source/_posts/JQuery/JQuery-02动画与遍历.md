---
title: JQuery-02动画与遍历
date: 2019-10-27 22:06:06
tags: 
- JQuery
categories: 
- 前端
- JQuery
---

<center>
引言：

JQuery的动画与遍历，事件绑定，案例，插件
</center>

<!--more-->

-----

# 动画
JQuery中定义好了一些方法来实现比较简单的动画

## 默认显示和隐藏方式
折叠收缩

1. show()
2. hide()
3. toggle()
```js
/*
show有三个参数：
1. speed：动画的速度，有三个预定义的值"slow","normal","fast",或者直接给定毫秒值
    //swing是先快后慢，linear是匀速的
2. easing：用来指定切换效果，默认是swing，可用参数linear
3. fn：一个在动画完成时执行的函数，每个元素执行只一次
*/
//展示
$("button:eq(0)").click(function () {
    $(".div1").show("slow","linear",function () {
        alert("展示了");
    });
})
//隐藏
$("button:eq(1)").click(function () {
    $(".div1").hide("slow","linear",function () {
        alert("隐藏了");
    });
})
//自动切换
$("button:eq(2)").click(function () {
$(".div1").toggle("slow","swing",function () {
    alert("切换了");
});
```

## 滑动显示和隐藏方式
从下往上的滑动
```js
//展示
$("button:eq(0)").click(function () {
    $(".div1").slideUp("slow");
})
//隐藏
$("button:eq(1)").click(function () {
    $(".div1").slideDown("slow");
})
//自动切换
$("button:eq(2)").click(function () {
    $(".div1").slideToggle("slow");
});
```

## 淡入淡出显示和隐藏方式
淡入淡出
```js
//展示
$("button:eq(0)").click(function () {
    $(".div1").fadeIn("slow");
})
//隐藏
$("button:eq(1)").click(function () {
    $(".div1").fadeOut("slow");
})
//自动切换
$("button:eq(2)").click(function () {
    $(".div1").fadeToggle("slow");
});
```

# 遍历
js中需要使用for循环遍历对象

但JQuery中的遍历更加简单
1. `obj.each(callback)`方法
    ```
    //obj.each(callback())
    $("ul li").each(function (index,element) {
        //这里可以直接使用this来获取，但是获取不到索引值
        console.log(this.innerHTML);
        //所以我们可以给回调函数设置两个参数，一个是索引，一个是每一个对象
        console.log(index+"："+element.innerHTML);
        //返回false相当于break
        //return false;
        //返回true相当于continue
        //return true;
    });
    ```
2. `$.each(obj,[callback])`方法
    ```js
    $.each($("ul li"),function () {
        console.log(this.innerHTML);
    })
    //这个方法同上
    ```
3. `for..of`方法,JQuery3.0版本之后提供的方法
    ```js
    for (li of $("ul li")){
        console.log(li.innerHTML);
    }
    ```
