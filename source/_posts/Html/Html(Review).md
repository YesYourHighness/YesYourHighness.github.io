---
title: HTML5(Review)
date: 2020-12-17 21:21:28
tags: 
- HTML
categories: 
- 前端
- HTML
---

<center>
引言：

马上考试啦、复习一下HTML
</center>
<!-- more -->


# HTML5(Review)

## web标准

> web标准：W3C和其他标准化组织制定的一系列标准的集合。该标准用来创建和解释基于 Web 的内容，其网页部分的标准通过三部分来描述：结构、表现和行为

- 结构标准：用于对网页元素进行整理和分类，主要包括两个部分：XML和XHTML。
- 表现标准：用于设置网页元素的版式、颜色、大小等外观样式，主要指的是CSS。
- 行为标准：指网页模型的定义及交互的编写，主要包括两个部分：DOM和ECMAScript



## Html5

> Html： Hyper Text Markup Language 超文本标记语言

### 语法基础

​		完整的HTML文件包括**头部**和**主体**两大部分。其文档结构由`<html>`、`<head>`和`<body>`这三大元素组成。

- html
  - head
  - body

```html
<!DOCTYPE html>		// 文档声明类型，通知浏览器按照什么方式进行解析
<html>
    <head>
        <meta charset="UTF-8">
        <title>Document</title>
    </head>
    <body>

    </body>
</html>
```

### 标签

单标签有：`<br>`与`<hr>`与`<meta>`与`<link>`与`<input>`，可以加斜杠如`<br />`

大部分都为双标签

### 注释

```html
<!-- 这里是注释-->

<comment>注释信息</comment>
```

### 编写规范

1. 以`<`开始，以`>`结束
2. 可以嵌套、不可以交叉使用
3. 不区分大小写

### head 头部相关标签

1. `<title>`：设置网页标题（标题可以用作默认快捷方式或收藏夹的名称）

   - 一个网页只能有一个标题
   - 标题名称的长度**不超过64**个字符数
   - 标题标记对之间不允许有其它的标签存在

2. `<meta>`：设置页面元信息

   ```html
   <meta charset="UTF-8">	设置编码方式
   <meta http-equiv="" content="">
   <meta name="" content="">
   ```

   http-equiv与content：设置页面相关属性

   ```html
   <meta http-equiv="refresh" content="3">	//3s后刷新页面
   <meta http-equiv="expires" content="Fri,31 Dec 2021 08:00:00 GMT">	//设置过期时间
   <meta http-equiv="Content-Type" content="text/html;Charset=utf-8">	//设置文件内容
   <meta http-equiv =“Set-Cookie” Content="cookievalue = xxx; expires= Thur,26 Jul 2018 08:00:00 GMT; path=/">	//设置cookie
   ```

   name与content：设置搜索引擎相关内容

   ```html
   <meta name="author" content="dwh">				   // 设置作者
   <meta name="description" content="简介">			   //设置内容简介
   <meta name="keywords" content="关键词,时尚,购物">	//设置关键词
   <meta name="generator" content="Dreamweaver">	 //设置编译器类型
   <meta name="revised" content="David,2008/8/8">	     //为搜索引擎提供最新版本
   ```

3. `<link>`：引用外部文件

   ```html
   <link href="" rel="" type="" />
   	href：设置url
   	rel：设置当前文档与引用的外部文档的关系,stylesheet表示定义一个外部样式表
   	type：设置外部文档的类型
   ```

   ```html
   <link rel="stylesheet" type="text/css" href="style.css">
   <link rel="shortcut icon" type="image/ico" href="图标路径"> //可以配置标题旁边的小图标
   ```

4. `<style>`：写内联css样式文件



### 基本标签

#### 标题段落标签

```html
<h1 align="left">1</h1>
<h2 align="center">2</h2>
<h3 align="right">3</h3>
<h4 align="justify">4</h4>
<h5>5</h5>
<h6>6</h6>

<p>段落标签</p>
```

```css
p{
    align: left;
    align: right;
    align: center;
    align: justify;
}// align一般有四个值，左、右、居中、两端对齐
```

这里用word表示一下居中和两端对齐的区别：

<img src="http://img.yesmylord.cn//image-20201202211302082.png" alt="image-20201202211302082" style="width:50%;" />



#### 文本格式标签

```html
<br>换行
<hr size="4" width="100" align="right" noshade="noshade" color="red">水平线
<div>块区域</div>
<span>行内标签</span>
<font size="10" face="宋体" align="left" color="blue">设置字体</font>
<b>加粗</b>
<i>斜体</i>
<big>比周围文本大一个尺寸</big>
<small>比周围文本小一个尺寸</small>
<s>加一条删除线</s>
<u>加一条下划线</u>
<sup>上标</sup>
<sub>下标</sub>
<strong>加粗，语义强调</strong>
<mark>带有记号</mark>
```

span与div的区别：

1. span是行内元素，不会换行；div是块级元素，会换行
2. span不能包div，反之可以



#### 特殊字符标签：

&开始，;结尾

```html
&gt;	小于
&copy;	版权号
```



#### 图片标签

图片格式：

- GIF
  - 支持动画
  - 无损图形格式
  - 支持透明
  - 常用于logo、小图标、色彩单一的图像
- PNG
  - 包括PNG-8和真色彩PNG（PNG-24和PNG-32）
  - 体积比gif小
  - 支持透明
  - 色彩过渡平滑
  - 不支持动画

- JPG
  - 色彩显示最全
  - 有损压缩

小图片考虑GIF或PNG-8，半透明图像考虑PNG-24，类似照片的图像则考虑JPG



```html
<!-- 图片标签 -->
<img 
     src="https://uploadbeta.com/api/pictures/random/?key=BingEverydayWallpaperPicture" 
     alt="图片加载中..." 
     border="1px solid red"
     width="100"
     height="100"
     vspace="100"
     hspace="100"
     title="图片的说明"
     align="center" 
     >
src：设置图片来源
alt：设置加载中的文字
title：设置鼠标提示文字
width、height：设置图像的宽度与高度
vspace、hspace：设置水平、垂直边缘的边距（设置的是margin值）
```



#### 超链接标签

按链接路径不同分为三种：

- 内部链接
- 锚点链接
- 外部链接

链接的组成：

- 链宿：要跳转的目标
- 链源：引起跳转的原因

```html
<!-- 超链接 -->
<a 
   href="http://www.baidu.com"
   name="top"
   title="提示信息"
   target="_blank" 
   >链源</a>
<a href="#top">锚点链接</a>

href：链宿的路径
name：创建文档内的书签（用于标记当前位置）
title：提示信息
target：指定打开的目标窗口，五个取值
	parent 上一级窗口；
	blank	新窗口；
	self 	  同一窗口，默认值；
       top     整个窗口打开；
	framename-框架名
```



#### 表格标签

```html
<table border="1">
    <caption>标题</caption>
    <tr>
        <th>表头1</th>
        <td>数据单元</td>
    </tr>
    <tr>
        <th>表头2</th>
        <td>数据单元</td>
        <td>数据单元</td>
    </tr>
    <tr>
        <td>数据单元</td>
        <td>数据单元</td>
    </tr>
</table>
```

<img src="http://img.yesmylord.cn//image-20201202215214296.png" alt="image-20201202215214296" style="width:50%;" />

表格样式：

```css
table{
    border：设置边框
    cellspacing：设置单元格与单元格边框之间的空白距离，默认为2px
    cellpadding：设置单元格内部padding，默认为1px
    width、height：设置表格宽度
    align：设置对齐样式
    bgcolor：设置表格背景颜色
    background：设置背景图像
    rules: 设置规定内侧边框的哪个部分是可见的。
                none	没有线条。
                groups	位于行组和列组之间的线条。
                rows	位于行之间的线条。
                cols	位于列之间的线条。
                all	位于行和列之间的线条。
     frame: 规定外侧边框的哪个部分是可见的。
        	 void	不显示外侧边框。
                above	显示上部的外侧边框。
                below	显示下部的外侧边框。
                hsides	显示上部和下部的外侧边框。
                vsides	显示左边和右边的外侧边框。
                lhs	显示左边的外侧边框。
                rhs	显示右边的外侧边框。
                box	在所有四个边上显示外侧边框。
                border	在所有四个边上显示外侧边框。
}
tr{
    height：设置行高度
    align：设置一行内容的水平对齐方式	取值left right center
    valign：设置垂直方向上的对齐方式 取值 top middle bottom
    bgcolor：设置表格背景颜色
    background：设置行背景图像
}
th,td{
    width、height：设置单元格宽度、高度
    align：设置一行内容的水平对齐方式	取值left right center
    valign：设置垂直方向上的对齐方式 取值 top middle bottom
    bgcolor：设置表格背景颜色
    background：设置行背景图像
    colspan：水平单元格合并，给数字即可
    rowspan：垂直单元格合并，给数字即可
}

```

设置表格边框的属性：

```html
<table  frame=” ” rules=” ”> </table >
```

![image-20201202215935434](http://img.yesmylord.cn//image-20201202215935434.png)



### HTML5

HTML5 通过一些新标签，新功能为开发更加简单、独立、标准的通用Web应用提供了标准。有了新的结构性的标签的标准，可以让HTML文档更加清晰，可阅读性更强，更利于SEO，也更利于视障人士阅读。

解决了三大问题：

1. **浏览器兼容**问题
2. 解决了**文档结构不明确**的问题（旧版本的html使用`div`+自定义的`id`（例如`<div id="header">`，表示头部，但是语义并不明确），html5提供了新的页面标签）
3. 解决了**Web**应用程序功能受限等问题。

#### 结构元素

```html
<header>标签用于定义文章的页眉信息</header>

<article>
    	用于表示文档、页面或应用程序中独立的、完整的、可以独自被外部引用的内容，该内容可以是一篇文章、一篇短文、一个帖子或一个评论等
</article>

<aside>
    	专门用于定义当前页面或当前文章的附属信息，包括当前页面或当前文章的相关引用、侧边栏、广告以及导航等有别于主体内容的部分
</aside>

<footer>用于定义脚注部分，包括文章的版权信息、作者授权等信息。</footer>

<figure>
    <figcaption>应该被置于 "figure" 元素的第一个或最后一个子元素的位置。</figcaption>
    	表示网站制作页面上一块独立的内容，将其从网页上移除后不会对网页上的其他内容产生影响。figure所表示的内容可以是图片、统计图或代码示例。
</figure>


```

#### 页面节点

```html
<!-- html5 页面节点 -->
<section>
    用于定义文章的节，一般用于成节的内容，会在文档流中开始一个新的节。
    它用来表现普通的文档内容或应用区块，通常由内容及其标题组成。
</section>

<nav>
    代表页面的一个部分，是一个可以作为页面导航的链接组，其中的导航元素链接到其它页面或者当前页面的其它部分，使html代码在语义化方面更加精确，同时对于屏幕阅读器等设备的支持也更好。
</nav>

<address>
    用于一般被作者用来提供该文档的联系人信息，一般放在一个网页的开头或者结尾，最常用的是和其他内容包含在footer元素内.
    如果address元素位于article元素内部，则它表示article元素所包含文章内容的作者的联系信息
    如果直接位于body元素内，那么表示该网页的作者的联系信息。
</address>
```

#### 交互元素

##### progress

表示页面中的某个任务完成的进度

```html
<progress max="50" value="25"></progress>

max： 总数
value：当前数量
```

效果：<progress max="50" value="25"></progress>

##### meter

可用于投票系统中候选人各占比例情况及考试分数统计等

```html
<meter value="0.6">60%</meter>
<meter max="100" min="0" high="80" low="30" value="10"></meter>
<meter max="100" min="0" high="80" low="30" value="40"></meter>
<meter max="100" min="0" high="80" low="30" value="80"></meter>

form：规定meter元素所属的一个或多个表单
high、low：规定视作高、低的值的范围（必需大于或小于此值才会改变颜色，不能正好等于）
max、min：规定最大最小值
optimum：规定度量的优化值
value：必需的值，规定当前值
```

效果：

<meter value="0.6">60%</meter>

<meter max="100" min="0" high="80" low="30" value="10"></meter>
<meter max="100" min="0" high="80" low="30" value="40"></meter>
<meter max="100" min="0" high="80" low="30" value="90"></meter>

##### details

用于说明文档的详细信息。在特定的浏览器下(如Chrome、 Safari ）能够产生像手风琴一样展开和折叠的交互效果。`<summary>`元素包含于 `< details >`元素中，用来说明其的标题。

```html
<details>
    <summary>第二级目录</summary>
    <p>关于HTML5 Summary元素的介绍</p>
</details>
<details draggable="true" open="open">
    <summary>第三级目录</summary>
    <p>关于HTML5 Summary元素的介绍</p>
</details>

details属性：
open：控制details元素是否显示，默认不可见
subject：设置元素对应的项目id号
draggable：true、false是否可拖动元素
```

效果：

<details>
    <summary>第二级目录</summary>
    <p>关于HTML5 Summary元素的介绍</p>
</details>

  	<details draggable="true" open="open">
        <summary>第三级目录</summary>
        <p>关于HTML5 Summary元素的介绍</p>
    </details>



##### menu（了解）

重新启用的一个旧标记。

该元素Menu是一系列菜单命令的集合，在一个容器中，Menu元素用于创建上下文、工具栏和弹出菜单。然而，后面的两个功能还没有浏览器实现，包括FireFox

**目前所有主流浏览器都不支持` <menu> `标签。**

```html
<menu type="toolbar" id="myMenu">
    <menuitem label="单击一下" onclick="alert('您单击了我一下')" icon=""></menuitem>
</menu>
属性：
	label：规定菜单的可见标签
	type：规定要显示哪种菜单类型
		context、toolbar、popup
```

##### command(了解)

定义各种类型的命令按钮。

利用该标记的“url”属性可以添加图片，并且实现图片按钮效果；

另外，改变标记中的“type”属性值，还可以定义复选框或单选框按钮。

只有当command 元素位于menu 元素内时，该元素才是可见的。

**没有浏览器支持 <command> 标签。**

**只有 Internet Explorer 9 （更早或更晚的版本都不支持）支持 <command> 标签。**

![image-20201203172928775](http://img.yesmylord.cn//image-20201203172928775.png)



#### 文本层次语义元素

```html
<!-- HTML5 文本层次语义元素 -->
<em>强调的内容</em>
<strong>语气更强的强调</strong>
<dfn>定义项目</dfn>
<code>int a = 3;</code>//计算机代码文本
<samp>样本文本</samp>
<kbd>CTRL</kbd>//表示键盘内容
<var>a</var>//定义变量
<cite>引用</cite>
<time datetime="2008-02-14">情人节</time> 
	两个属性：
		1. datetime：规定日期/时间
		2. pubdate：指示 time 元素中的日期 / 时间是文档（或 article 元素）的发布日期。
```

<em>强调的内容</em>
<strong>语气更强的强调</strong>
<dfn>定义项目</dfn>
<code>int a = 3;</code>
<samp>样本文本</samp>
<kbd>CTRL</kbd>
<var>a</var>
<cite>引用</cite>

<time datetime="2008-02-14">情人节</time> 



#### 分组元素

```html
<!-- 分组元素 -->
<div>无语义的通用元素</div>
<blockquote>表示摘自另一个源的大段内容</blockquote>
<q>摘自另一个源的小段内容</q>
<pre> 格式 会被   保留</pre>
<ul>无序列表
    <li>你</li>
    <li>好</li>
</ul>
<ol>有序列表
    <li>你</li>
    <li>好</li>
</ol>
<dl>包含一系列术语和定义说明的列表
    <dt>表示术语，充当标题</dt>
    <dd>表示定义，一般是内容</dd>
</dl>
<figure>表示图片
    <figcaption>标题</figcaption>
</figure>
```

<div>无语义的通用元素</div>
<blockquote>表示摘自另一个源的大段内容</blockquote>
<q>摘自另一个源的小段内容</q>

<pre> 格式 会被   保留</pre>

<ul>无序列表
    <li>你</li>
    <li>好</li>
</ul>
<ol>有序列表
    <li>你</li>
    <li>好</li>
</ol>
<dl>包含一系列术语和定义说明的列表
    <dt>表示术语，充当标题</dt>
    <dd>表示定义，一般是内容</dd>
</dl>
<figure>表示图片
    <figcaption>标题</figcaption>
</figure>



##### ul

```html
<!-- ul -->
<ul type="square">
    <li>你</li>
    <li>好</li>
    <li>吗</li>
</ul>

type有三个取值：
	1. disk ：默认，实心小黑点
	2. circle：空心小圆点
	3. square：正方形

ul{ list-style-type:none;} 去掉前面的小圆点
```

<ul type="square">
    <li>你</li>
    <li>好</li>
    <li>吗</li>
</ul>



##### ol

```html
<ol type="i" start="3">
    <li>你</li>
    <li>好</li>
    <li>吗</li>
</ol>
type取值：1、a、A、I、i 分别表示数字、小写字母、大写字母、大写罗马数字、小写罗马数字
start：列表初始值
```

<ol type="i" start="3">
    <li>你</li>
    <li>好</li>
    <li>吗</li>
</ol>

##### dl、dt、dd

```html
<dl>
    <dt>物联网</dt><!--定义术语名词-->
    <dd>物物相连的互联网</dd><!--解释和描述名词-->
    <dd>互联网的应用拓展</dd>
    <dd>物品与物品之间进行信息、交换和通信</dd>
</dl>
```

  	<dl>
      <dt>物联网</dt><!--定义术语名词-->
      <dd>物物相连的互联网</dd><!--解释和描述名词-->
      <dd>互联网的应用拓展</dd>
      <dd>物品与物品之间进行信息、交换和通信</dd>
    </dl>



### 全局属性

**任何一个标签都是可以使用的属性**

在HTML5中新增了一些全局属性，这些属性可以表达非常丰富的语义，也会额外提供很多实用的功能

```css
accesskex：规定激活元素的快捷键。
class：规定元素的一个或多个类名（引用样式表中的类）
contenteditable：规定元素内容是否可编辑。
contextmenu：规定元素的上下文菜单。上下文菜单在用户点击元素时显示。
data-：用于存储页面或应用程序的私有定制数据。
dir：规定元素中内容的文本方向。
draggable：规定元素是否可拖动。
dropzone：规定在拖动被拖动数据时是否进行复制、移动或链接。
hidden：规定元素是否显示
id：元素的唯一id
lang：元素内容的语言
spellcheck：是否对元素进行拼写和语法检查
style：规定行内css样式
tabindex：规定元素tab键的次序
title：有关元素的额外信息
translate：是否翻译元素内容
```



### 表单元素

#### form

结构：

- 提示信息
- 表单控件：包含表单具体功能：如文本输入框、密码输入框、提交按钮

- 表单域：容纳表单控件和提示信息

![image-20201203180223480](http://img.yesmylord.cn//image-20201203180223480.png)

特点：

- 每个表单都以form开始标签开始，以form标签结束。
- 每个表单及其表单控件都有一个 name 属性，用于在提交表单时对表单及数据进行识别
- 访问者通过所提供的提交按钮提交表单——触发提交按钮时，填写的数据就会发送至服务器。form开始标签可以有一些属性，其中最重要的就是action和method。

```html
<form action="url地址" method="post" name="表单名称"></form>
action：用于指定接收并处理表单数据的服务器程序的url地址。
method：get 数据将显示在浏览器的地址栏中，保密性差且有数据量的限制。
		   post方式的保密性好，并且无数据量的限制，使用method="post"可以大量的提交数据。
name：用于指定表单的名称，以区分同一个页面中的多个表单。
```

#### input

常用属性：

```css
input{
    type: 取值为text、password、radio 、 checkbox 、 button、 submit、 reset、 file、image、 hidden等;
    name: 控件的名称;
    value: 默认显示文本;
    size:  设置显示宽度;
    readonly: 设置内容为只读 readonly;
    checked: 默认被选中 checked;
    disabled: 首次加载时禁用该控件;
    maxlength: 允许输入的最多字符数;
    // html5 新增的内容
    type：search（搜索框）、email、url、Tel（电话号码输入框）、Number、range（滑动条）、color（颜色选择）、date（日期选择）、month、week、time、Datetime-local（日期时间）;
    required: 元素必填;
    placeholder: 输入提示;
    pattern: 验证输入字段;
    autofocus: 自动聚焦;
}
```

```html
<!-- 表单元素 -->
<form action="url地址" method="post" name="表单名称">
    <input type="text" placeholder="文本框"> <br>
    <input type="password" placeholder="密码" maxlength="15"> <br>
    <input type="radio"> <br>
    <input type="checkbox"> <br>
    <input type="button" value="按钮"> <br>
    <input type="submit"> <br>
    <input type="reset"> <br>
    <input type="image"> <br>
    <input type="file"> <br>

    <!-- html新type -->
    <input type="range"> <br>
    <input type="color"> <br>
    <input type="search" placeholder="搜索框"> <br>
    <input type="email" placeholder="邮件"> <br>
    <input type="tel" placeholder="电话"> <br>
    <input type="number" placeholder="数字"> <br>
    <input type="date"> <br>
    <input type="month"> <br>
    <input type="week"> <br>
    <input type="time"> <br>
    <input type="Datetime-local"> <br>
    <input type="datetime"> <br>
</form>
```

效果演示：

<!-- 表单元素 -->

<form action="url地址" method="post" name="表单名称">
    <input type="text" placeholder="文本框"> <br>
    <input type="password" placeholder="密码" maxlength="15"> <br>
    <input type="radio"> <br>
    <input type="checkbox"> <br>
    <input type="button" value="按钮"> <br>
    <input type="submit"> <br>
    <input type="reset"> <br>
    <input type="image"> <br>
    <input type="file"> <br>
    <input type="range"> <br>
    <input type="color"> <br>
    <input type="search" placeholder="搜索框"> <br>
    <input type="email" placeholder="邮件"> <br>
    <input type="tel" placeholder="电话"> <br>
    <input type="number" placeholder="数字"> <br>
    <input type="date"> <br>
    <input type="month"> <br>
    <input type="week"> <br>
    <input type="time"> <br>
    <input type="Datetime-local"> <br>
    <input type="datetime"> <br></form>





#### textarea控件

多行文本输入框

```html
<textarea name="" id="" cols="30" rows="10" wrap="soft">
    文本内容
</textarea>
cols与rows表示每行的字符数与显示的行数
wrap有两个值：soft与hard，表示在表单中提交时，前者文本不换行（默认值）、后者文本换行（此时必须规定cols属性）
```

<textarea name="" id="" cols="30" rows="10" wrap="soft">
    文本内容
</textarea>



#### select控件

创建选择框，为用户提供一组选项

```html
<select name="word" size="2">
    <option value="" slected>a</option>
    <option value="">b</option>
    <option value="">c</option>
</select>
// size表示显示的个数
```

<select name="word" size="2">
    <option value="" slected>a</option>
    <option value="">b</option>
    <option value="">c</option>
</select>

### 表单处理

三种处理表单的方式：

- 表单分组：`fieldset`将相关元素组合在一起
- 表单校验：`pattern`验证输入字段的模式
- 添加说明：`label`为表单组件添加说明



#### 表单分组：fieldset元素

使用fieldset元素将相关的元素组合在一起， 使表单更容易理解

> fieldset：表单的一个子容器，将所包含的内容以边框环绕方式显示
>
> legend：添加子容器的标题

```html
<form action="">
    <fieldset>
        <legend>注册表单</legend>
        <input type="text" placeholder="输入用户名"> <br>
        <input type="text" placeholder="输入密码">
    </fieldset>
    <fieldset>
        <legend>下拉框表单</legend>
        <select name="word" size="1">
            <option value="" slected>a</option>
            <option value="">b</option>
            <option value="">c</option>
        </select>
    </fieldset>
    <fieldset>
        <legend>多选表单</legend>
        <input type="checkbox" name="choice" value="cs" checked>计算机 <br>
        <input type="checkbox" name="choice" value="physics">物理 <br>
        <input type="checkbox" name="choice" value="chemical">化学 <br>
    </fieldset>
</form>
```

效果展示：

<form action="">
    <fieldset>
        <legend>注册表单</legend>
        <input type="text" placeholder="输入用户名"> <br>
        <input type="text" placeholder="输入密码">
    </fieldset>
    <fieldset>
        <legend>下拉框表单</legend>
        <select name="word" size="1">
            <option value="" slected>a</option>
            <option value="">b</option>
            <option value="">c</option>
        </select>
    </fieldset>
    <fieldset>
    		<legend>多选表单</legend>
		    <input type="checkbox" name="choice" value="cs" checked>计算机 <br>
		    <input type="checkbox" name="choice" value="physics">物理 <br>
		    <input type="checkbox" name="choice" value="chemical">化学 <br>
    </fieldset>
</form>



#### 表单验证：pattern属性

在用户提交表单之前，验证用户输入的数据是否合法

##### email/url验证：

像html5新增的type`email/url`自带对输入的验证

```html
<form action="">
    <input type="email" value="email">
    <input type="url" value="url">
    <input type="submit">
</form>
```

我们输入不正确的email或是url，点击提交，就会有提示信息：

<form action="">
    <input type="email" value="email">
    <input type="url" value="url">
    <input type="submit">
</form>



##### pattern验证

`pattern`属性具有对表单中输入字段模式进行验证的功能，`pattern`内写正则表达式

```html
<form action="">
    <input type="text" pattern="^[1-9]?[0-9]$" placeholder="请输入一个0-99的数">
    <input type="submit">
</form>
常见的正则表达式元字符的含义:
    . 表示匹配任意字符
    * 匹配0个或多个
    + 匹配一个或多个
    ? 匹配0个或1个
    [...] 匹配方括号内的任意一个字符
    [^..]匹配 非 方括号内的内容
    ^ 开始
    $ 结束
    {n}表示前面的元素连续出现n次
    {n,}表示至少出现n次
    {m,n}表示至少出现m，至多出现n
    | 逻辑或

例如：^[a-zA-Z0-9]{6,} $
表示：只能输入英文或者数字并且至少输入6个字符
```

<form action="">
    <input type="text" pattern="^[1-9]?[0-9]$" placeholder="请输入一个0-99的数">
    <input type="submit">
</form>



##### 添加说明

label标签描述表单字段用途

> 不会向用户呈现任何特殊效果。
>
> 不过，它为鼠标用户改进了可用性。如果您在 `label` 元素内点击文本，就会触发此控件。就是说，当用户选择该标签时，浏览器就会**自动将焦点转到**和标签相关的表单控件上。

相关属性：

```
for： 规定label绑定到哪一个表单元素上，属性值该表单元素的id
form：规定label字段所属的一个或多个表单，取值为formid
```

例如：

```html
<!-- 表单说明 -->
<form action="" method="get"> 
性别：<br/> 
    <input name="sex" id="man" type="radio" value=""/> 
    <label for="man">男</label> 
    <input name="sex" id="woman" type="radio" value=""/>
    女 
</form> 
```

你会发现，当你点击“男”这个字的时候，选项会被打钩，但是“女”不行

<!-- 表单说明 -->

<form action="" method="get"> 
性别：<br/> 
  <input name="sex" id="man" type="radio" value=""/> 
  <label for="man">男</label> 
  <input name="sex" id="woman" type="radio" value=""/>
  女 
</form> 



### 音频与视频

#### 解码器

> 音频解码器：
>
> 定义了音频数据流编码和解码的算法。
>
> 其中，编码器主要是对数据流进行编码操作，用于存储和传输。
>
> 音频播放器主要是对音频文件进行解码，然后进行播放操作。目前，使用较多的音频解码器是Vorbis和ACC

音频编解码器

- **AAC**： 这是一种基于MPEG-2的音频编码技术。

- **Ogg Vorbis**：这是一种新的音频压缩格式，类似于MP3等的音乐格式。开源，支持多声道

> 视频解码器：
>
> 定义了视频数据流编码和解码的算法。
>
> 视频播放器主要是对视频文件进行解码，然后进行播放操作。目前，使用较多的视频解码器是Theora、H.264和VP8 。

视频编解码器

-  **Theora**： 这是一种一款由Xiph.org基金会发布的免费且开放的视频压缩编码技术。

- **H.264**：这是一种由国际电信联盟和国际标准化组织联合推出的高性能的。视频编解码技术。

- **VP8**：由 On2 Technologiesis 开发，随后由 Google 发布的视频压缩格式。



#### 音频标签

```html
<audio src="" controls="controls" loop="loop" autoplay="autoplay">
    <source src="1.mp3">
    <source src="1.wav">
    您的浏览器不支持 audio 标签
</audio>
src： 音频地址
controls：提供一个播放、暂停、音量的控件
autoplay：音频就绪后自动播放
loop：音频循环播放
preload：音频在页面加载时进行加载，并预备播放，如果有autoplay则忽略本属性

source标签可以链接不同的音频文件，浏览器将会使用第一个可以识别的格式
```

支持三种音频格式：

- **Ogg Vorbis**
- **MP3**
- **WAV**

效果演示：

<audio src="" controls="controls"></audio>



#### 视频标签

```html
<video src="" controls="controls">
    <source src="1.ogg">
    <source src="1.mp4">
    <source src="1.webM">
    您的浏览器不支持 video 标签
</video>
标签的属性同audio
```

支持的三种视频格式：

- Ogg
- MPEG4
- WebM

<video src="" controls="controls">
    您的浏览器不支持 video 标签
</video>



#### 音频视频相关的属性与方法（了解即可）

相关属性：

| **属性**            | **描述**                                               |
| ------------------- | ------------------------------------------------------ |
| audioTracks         | 返回表示可用音轨的 AudioTrackList 对象                 |
| autoplay            | 设置或返回是否在加载完成后随即播放音频/视频            |
| buffered            | 返回表示音频/视频已缓冲部分的 TimeRanges 对象          |
| controller          | 返回表示音频/视频当前媒体控制器的 MediaController 对象 |
| controls            | 设置或返回音频/视频是否显示控件（比如播放/暂停等）     |
| crossOrigin         | 设置或返回音频/视频的 CORS 设置                        |
| currentSrc          | 返回当前音频/视频的 URL                                |
| currentTime         | 设置或返回音频/视频中的当前播放位置（以秒计）          |
| defaultMuted        | 设置或返回音频/视频默认是否静音                        |
| defaultPlaybackRate | 设置或返回音频/视频的默认播放速度                      |

| **属性**     | **描述**                                                   |
| ------------ | ---------------------------------------------------------- |
| duration     | 返回当前音频/视频的长度（以秒计）                          |
| ended        | 返回音频/视频的播放是否已结束                              |
| error        | 返回表示音频/视频错误状态的 MediaError 对象                |
| loop         | 设置或返回音频/视频是否应在结束时重新播放                  |
| mediaGroup   | 设置或返回音频/视频所属的组合（用于连接多个音频/视频元素） |
| muted        | 设置或返回音频/视频是否静音                                |
| networkState | 返回音频/视频的当前网络状态                                |
| paused       | 设置或返回音频/视频是否暂停                                |
| playbackRate | 设置或返回音频/视频播放的速度                              |
| played       | 返回表示音频/视频已播放部分的 TimeRanges 对象              |

| **属性**    | **描述**                                        |
| ----------- | ----------------------------------------------- |
| preload     | 设置或返回音频/视频是否应该在页面加载后进行加载 |
| readyState  | 返回音频/视频当前的就绪状态                     |
| seekable    | 返回表示音频/视频可寻址部分的 TimeRanges 对象   |
| seeking     | 返回用户是否正在音频/视频中进行查找             |
| src         | 设置或返回音频/视频元素的当前来源               |
| startDate   | 返回表示当前时间偏移的 Date 对象                |
| textTracks  | 返回表示可用文本轨道的 TextTrackList 对象       |
| videoTracks | 返回表示可用视频轨道的 VideoTrackList 对象      |
| volume      | 设置或返回音频/视频的音量                       |

相关方法：

| **方法**       | **描述**                                |
| -------------- | --------------------------------------- |
| addTextTrack() | 向音频/视频添加新的文本轨道             |
| canPlayType()  | 检测浏览器是否能播放指定的音频/视频类型 |
| load()         | 重新加载音频/视频元素                   |
| play()         | 开始播放音频/视频                       |
| pause()        | 暂停当前播放的音频/视频                 |

| **属性**       | **描述**                                     |
| -------------- | -------------------------------------------- |
| abort          | 当音频/视频的加载已放弃时                    |
| canplay        | 当浏览器可以播放音频/视频时                  |
| canplaythrough | 当浏览器可在不因缓冲而停顿的情况下进行播放时 |
| durationchange | 当音频/视频的时长已更改时                    |
| emptied        | 当目前的播放列表为空时                       |
| ended          | 当目前的播放列表已结束时                     |
| error          | 当在音频/视频加载期间发生错误时              |
| loadeddata     | 当浏览器已加载音频/视频的当前帧时            |
| loadedmetadata | 当浏览器已加载音频/视频的元数据时            |
| loadstart      | 当浏览器开始查找音频/视频时                  |

| **属性**     | **描述**                                    |
| ------------ | ------------------------------------------- |
| play         | 当音频/视频已开始或不再暂停时               |
| playing      | 当音频/视频在已因缓冲而暂停或停止后已就绪时 |
| progress     | 当浏览器正在下载音频/视频时                 |
| ratechange   | 当音频/视频的播放速度已更改时               |
| seeked       | 当用户已移动/跳跃到音频/视频中的新位置时    |
| seeking      | 当用户开始移动/跳跃到音频/视频中的新位置时  |
| stalled      | 当浏览器尝试获取媒体数据，但数据不可用时    |
| suspend      | 当浏览器刻意不获取媒体数据时                |
| timeupdate   | 当目前的播放位置已更改时                    |
| volumechange | 当音量已更改时                              |





