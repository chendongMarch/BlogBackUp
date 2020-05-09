---
layout: post
title: CSS 选择器
categories:
  - 前端
tags:
  - CSS
  - 前端
keywords:
  - CSS
  - 选择器
abbrlink: d318b1f4
date: 2017-07-07 12:48:37
password:
---

使用 `CSS` 可以声明元素的样式规则，而这些规则到底匹配什么类型的元素，则由选择器来决定，选择器就是用来为声明的样式指定一个或者一组匹配的元素的一种规则。

为了能满足匹配的多种要求，选择器的种类也是多种多样的。
 
<!--more-->

本文主要分为两部分，一个是借助选择器实现的一些特殊的效果做一下记录，另一个是系统的介绍各种选择器的使用和特性，做需要时查找使用。
> 效果

- hover A 元素，B 元素作出反应 

> 选择器

- 基本选择器
- 多元素选择器
- 属性选择器 (css2.1)
- 伪类(css2.1)
- 伪元素(css2.1)
- 同级元素通用选择器(css3)
- 属性选择器(css3)
- 与用户界面有关的伪类(css3)
- 结构性伪类(css3)
- 反选伪类(css3)
- :target 伪类(css3)

## 记录一些效果

- hover A 元素，B 元素作出反应 

如果是当前元素的 `hover` 很好实现，使用 `hover` 伪类实现即可，目前的场景是 `hover` A 元素，B 元素作出反应，可以结合伪类选择器和后代选择器来实现，当 父元素 `hover` 时也改变子元素的状态。

```html
<style> 
.child {
	position:absolute;
	height:100px;
	background:blue;
	transition:all 0.5s;
	
	left:-50%;
	opacity: 0;
	width: 300px;
}
.father {
	background-color:red;
	width:500px;
	height: 300px;
}

.father:hover .child {
	left:0;
	opacity: 1;
	width: 100px;
}
</style>
 
<body>
	<div class="father">
		<div class="child"></div>
	</div>
</body>
```

## 基本选择器

标记|描述|含义
:--|:--|:--|
*|通用选择器|匹配所有元素 
E|标签选择器|匹配所有使用E标签的元素 
.info|class选择器|匹配所有class属性中包含info的元素
\#footer|id选择器|匹配所有id属性等于footer的元素

实例

```css
* { margin:0; padding:0; }
p { font-size:2em; }
.info { background:#ff0; }
p.info { background:#ff0; }
p.info.error { color:#900; font-weight:bold; }
#info { background:#ff0; }
p#info { background:#ff0; }
```

## 多元素选择器

标记|描述|含义
:--|:--|:--|
E,F|多元素选择器|同时匹配所有E元素或F元素，E和F之间用逗号分隔
E F|后代元素选择器|匹配所有属于E元素后代的F元素，E和F之间用空格分隔
E > F|子元素选择器|匹配所有E元素的子元素F
E + F|毗邻元素选择器|匹配所有紧随E元素之后的同级元素F

示例

```css
div p { color:#f00; }
#nav li { display:inline; }
#nav a { font-weight:bold; }
div > strong { color:#f00; }
p + p { color:#f00; }
```

## 属性选择器 (css2.1)

标记|描述
:--|:--|
E[attr]|匹配所有具有attr属性的E元素，不考虑它的值。(注意：E在此处可以省略，比如"[title]"，以下同。)
E[attr=val]|匹配所有attr属性等于"val"的E元素
E[attr~=val]|匹配所有attr属性具有多个空格分隔的值、其中一个值等于"val"的E元素
E[attr&vert;=val]|匹配所有att属性具有多个连字号分隔的值、其中一个值以"val"开头的E元素，主要用于lang属性，比如"en"、"en-us"、"en-gb"等等

示例

```css
p[title] { color:#f00; }
div[class=error] { color:#f00; }
td[headers~=col1] { color:#f00; }
p[lang|=en] { color:#f00; }
blockquote[class=quote][cite] { color:#f00; }
```

## 伪类(css2.1)

伪类，表示它具有类的相似性质，但是它并不是类。

可以理解为，伪类更多的表示的元素的一种状态，比如 `:first-child` 表示的第一个字元素，`:visited` 表示元素被访问过，（当然不是所有元素都会有被访问过的这种状态）由浏览器创建一些内置类，来表达这种状态，并根据元素实际的状态，向这些内置类中动态添加或删除。

标记|描述
:--|:--|
E:first-child|匹配父元素的第一个子元素
E:link|匹配所有未被点击的链接
E:visited|匹配所有已被点击的链接
E:active|匹配鼠标已经其上按下、还没有释放的E元素
E:hover|匹配鼠标悬停其上的E元素
E:focus|匹配获得当前焦点的E元素
E:lang(c)|匹配lang属性等于c的E元素

示例

```css
p:first-child { font-style:italic; }
input[type=text]:focus { color:#000; background:#ffe; }
input[type=text]:focus:hover { background:#fff; }
q:lang(sv) { quotes: "\201D" "\201D" "\2019" "\2019"; }
```

## 伪元素(css2.1)

标记|描述
:--|:--|
E:first-line|	匹配E元素的第一行
E:first-letter|匹配E元素的第一个字母
E:before|在E元素之前插入生成的内容
E:after|在E元素之后插入生成的内容

示例

```css
p:first-line { font-weight:bold; color;#600; }
.preamble:first-letter { font-size:1.5em; font-weight:bold; }
.cbb:before { content:""; display:block; height:17px; width:18px; background:url(top.png) no-repeat 0 0; margin:0 0 0 -18px; }
a:link:after { content: " (" attr(href) ") "; }

```
## 同级元素通用选择器(css3)

标记|描述
:--|:--|
E ~ F|匹配任何在E元素之后的同级F元素

示例

```css
p ~ ul { background:#ff0; }
```

## 属性选择器(css3)

标记|描述
:--|:--|
E[attr\^="val"]|属性attr的值以"val"开头的元素
E[attr$="val"]|属性attr的值以"val"结尾的元素
E[attr*="val"]|属性attr的值包含"val"字符串的元素

示例

```css
div[id^="nav"] { background:#ff0; }
```

## 与用户界面有关的伪类(css3)

标记|描述
:--|:--|
E:enabled|匹配表单中激活的元素
E:disabled|匹配表单中禁用的元素
E:checked|匹配表单中被选中的radio 或checkbox 元素	
E:selection|	匹配用户当前选中的元素

示例

```css
input[type="text"]:disabled { background:#ddd; }
```


## 结构性伪类(css3)

标记|描述
:--|:--|
E:root	|匹配文档的根元素，对于HTML文档，就是HTML元素
E:nth-child(n)|匹配其父元素的第n个子元素，第一个编号为1
E:nth-last-child(n)|匹配其父元素的倒数第n个子元素，第一个编号为1
E:nth-of-type(n)|与:nth-child()作用类似，但是仅匹配使用同种标签的元素
E:nth-last-of-type(n)|与:nth-last-child() 作用类似，但是仅匹配使用同种标签的元素
E:last-child|匹配父元素的最后一个子元素，等同于:nth-last-child(1)
E:first-of-type|	匹配父元素下使用同种标签的第一个子元素，等同于:nth-of-type(1)
E:last-of-type|匹配父元素下使用同种标签的最后一个子元素，等同于:nth-last-of-type(1)
E:only-child|匹配父元素下仅有的一个子元素，等同于:first-child:last-child或 :nth-child(1):nth-last-child(1)
E:only-of-type|匹配父元素下使用同种标签的唯一一个子元素，等同于:first-of-type:last-of-type或 :nth-of-type(1):nth-last-of-type(1)
E:empty|匹配一个不包含任何子元素的元素，注意，文本节点也被看作子元素

示例

```css
p:nth-child(3) { color:#f00; }
p:nth-child(odd) { color:#f00; }
p:nth-child(even) { color:#f00; }
p:nth-child(3n+0) { color:#f00; }
p:nth-child(3n) { color:#f00; }
tr:nth-child(2n+11) { background:#ff0; }
tr:nth-last-child(2) { background:#ff0; }
p:last-child { background:#ff0; }
p:only-child { background:#ff0; }
p:empty { background:#ff0; }
```


## 反选伪类(css3)

标记|描述
:--|:--|
E:not(s)|匹配不符合当前选择器的任何元素

示例

```css
:not(p) { border:1px solid #ccc; }
```

## :target 伪类(css3)
标记|描述
:--|:--|
E:target|匹配文档中特定"id"点击后的效果
