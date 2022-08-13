---
title: JQuery-01基础
date: 2019-10-27 22:05:53
tags: 
- JQuery
categories: 
- 前端
- JQuery
---

<center>
引言：

JQuery的入门基础篇
</center>

<!--more-->

-----

# JQuery
> JQuery 是javascript的框架，简化js的开发

## 快速入门
1. 下载JQuery
```js
JQuery版本
1.x 兼容IE678，使用最为广泛
2.x 不兼容IE678
3.x 不兼容IE678但支持最新的浏览器 
```
2. 导入JQuery.js文件
3. 使用
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Jquery快速入门</title>
    <script src="jquery-3.3.1.min.js"></script>
    <!--导入JQuery包-->
</head>
<body>
    <div id="div1">div1</div>
    <div id="div2">div2</div>
<script>
    var $div = $("div");
    alert($div);
</script>
</body>
</html>
```

## 使用
- 获取ID对象
```js
//Js源码
var oDiv1 = document.getElementById("div1");
alert(oDiv1.innerHTML);
//js源码使用innerHtml属性来获取

//使用JQuery获取元素对象
var div1 = $("#div1");
alert(div1.html());
//JQ使用html()这个方法来获取
```

- 获取标签对象
```js
//Js源码
var aDiv = document.getElementsByTagName("div");
alert(aDiv);
for (var i=0;i<aDiv.length;i++){
    aDiv[i].innerHTML = "666";
}

//使用JQuery获取元素对象
var $div = $("div");
alert($div);
$div.html("777");
//jq中不需要遍历就直接使用
```
- js对象与jq对象的转换
```js
//Js源码
var aDiv = document.getElementsByTagName("div");
var $div = $("div");
//jq与js对象之间的转换
//1. jq转js--使用方括号加索引[索引]，或者使用get()方法
var div1 = $div.get(0);
var div2 = $div[0];
//2. js转jq--使用 $() 包裹js对象
var $1 = $(aDiv[0]);
```
- 选择器
1. 基本语法
    1. 事件绑定
        ```js
            //js源码
        var oB1 = document.getElementById("b1");
        var oDiv1 = document.getElementById("div1");
        oB1.onclick=function () {
            oDiv1.innerText="点击事件绑定";
        }
    
        //jq实现
        var $b1 = $("#b1");
        var $div = $("div");
        $b1.click(function () {
            $div.get(0).innerHTML="点击事件绑定";
        })
        ```
    2. 入口函数
    ```js
    //js源码入口函数，防止DOM未被载入完成而出错
    window.onload =function () {
        alert(1);
    }
    //jq入口函数
    $(function () {
        alert(1);
    })
    /*
    window.onload与$(function)的区别
    - 前者只能使用一次，
    - 但是后者可以使用多次
    
    */
    ```
    3. 样式控制
    ```
    //js
    window.onload =function () {
        var oDiv1 = document.getElementById("div1");
        oDiv1.style.background ="red";
    }
    //jq
    $(function () {
        var $div2 = $("#div2");
        $div2.css("background","blue");
    })
    ```
2. 选择器的使用
    1. 基本选择器
    ```js
    //1. 元素选择器(见上文)
    //2. id选择器(见上文)
    //3. class选择器
    //js
    window.onload =function () {
        var div1 = document.getElementsByClassName("div1")[0];
        alert(div1);
    }
    //jq
    $(function () { 
        var $div1 = $(".div1");
        alert($div1);
    })
    
    //4. 兄弟选择器
    //js
    window.onload = function () {
        var oDiv1 = document.querySelector(".div1");
        oDiv1.nextElementSibling.style.background = "red";
    }
    //jq
    $(function () {
        var $div1 = $(".div1,.div2");
        $div1.css("background","blue");
    })
    ```
    2. 层级选择器
    ```js
    //选择直接的后代，不会选择后代的后代
    var $div1 = $(".div1>.div3");
    //选择所有的后代，会选择后代的后代
    var $div1 = $(".div1 .div3");
    ```
    3. 属性选择器
    ```js
    //1. 属性名称选择器
    $("div[name]").css("background","lightblue");
    //2. 属性选择器
    $("div[name='third']").css("background","green");
    //3. 复合属性选择器
        //name不为third
        $("div[name!='third']").css("background","blue")
        //name以d结尾
        $("div[name$='d']").css("background","red")
        //name以th开头
        $("div[name^='th']").css("background","orange")
        //name含有d
        $("div[name*='d']").css("background","yellow")
        //选取有属性id且name含有e的
        $("div[id][name*='e']").css("background","purple")
    ```
    4. 过滤选择器
    ```js
    //选择第一个元素
    $("div:first").css("background","red");
    //选择最后一个元素
    $("div:last").css("background","orange");
    //选择class不为div1的所有div元素
    $("div:not(.div1)").css("background","green");
    //选择索引为偶数的元素，从0开始计数
    $("div:even").css("background","blue");
    //选择索引为奇数的元素
    $("div:odd").css("background","pink");
    //选择索引大于3的元素,great than
    $("div:gt(3)").css("background","grey");
    //选择索引小于3的元素,less than
    $("div:lt(3)").css("background","black");
    //选择索引等于3的元素,equals
    $("div:eq(3)").css("background","white");
    //选择标题元素,header
    $(":header").css("background","green");
    ```
    5. 表单过滤选择器
    ```js
    //jQuery对象val()方法改变表单内可用input值,enabled表示可使用
    $("input[type='text']:enabled").val("改变值");
    //改变不可用元素
    $("input[type='text']:disabled").val("改变值");
    //复选框:checked 获取复选框选中的个数
    $("input[type='checkbox']:checked").length;
    //获取下拉框option选中的个数
    $("select>option:checked").length;
    ```
3. DOM操作
    - 内容操作
    1. html():获取/设置元素的标签体内容
    2. text():获取/设置元素的标签体纯文本内容
    3. val():获取/设置元素的value属性值
    ```html
    <a href=""><span>你好</span></a>
    ```
    ```js
    $("a").html();//<span>你好</span>
    $("a").text();//你好
    $("input").val("你好");//所有input的value全为“你好”
    ```
    - 属性操作
    1. 通用属性操作
        1. attr()//获取/设置元素的属性
        2. removeAttr()//删除属性
        3. prop()//获取/设置元素的属性
        4. removeProp()//删除属性
        ```
        //操作固有属性使用prop，操作自定义属性使用attr
        //获取
        console.log($("#bj").attr("name"));
        //设置
        $("#bj").attr("name","更改值");
        $("#bj").attr("new","新增值");
        //删除属性
        $("#bj").removeAttr("name");
        ```
    2. 对class属性操作
        1. addClass():添加class属性值
        2. removeClass():移除class属性值
        3. toggleClass():切换class属性值，有就删除，没有就添加
        ```js
        $("#bj").addClass("first");
        $("#bj").removeClass("first");
        $("#tj").toggleClass("first");
        ```
    3. CRUD操作
        1. `obj1.append(obj2)`:将obj2添加到obj1的内部，并且在末尾
        2. `obj1.prepend(obj2)`将obj2添加到obj1的内部，并且在开头
        3. `obj1.appendTo(obj2)`将obj1添加到obj2的内部
        4. `obj1.pretendTo(obj2)`将obj1添加到obj2的内部
        5. `obj1.after(对象2)`将obj2添加到obj1的后面，兄弟关系
        6. `obj1.before(obj2)`将obj2添加到obj1的前面，兄弟关系
        7. `obj1.insertAfter(obj2)`将obj2添加到obj1的前面，兄弟关系
        8. `obj1.insertBefore(obj2)`将obj2添加到obj1的前面，兄弟关系
        9. `obj.remove()`删除对象,自己删除自己
        10 `obj.empty()`清空元素所有的后代元素，但保留当前对象以及当前节点
        