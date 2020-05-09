---
layout: post
title: CSS 基础 1
categories:
  - 前端
tags:
  - CSS
  - 前端
keywords:
  - CSS
comments: true
abbrlink: 5b70625a
date: 2017-07-06 17:01:37
password:
---

`CSS` 指层叠样式表 (`Cascading Style Sheets`)

解决内容与表现分离的问题。

<!--more-->



## 开始

```

body{
    background-color:#d0e4fe;
}
h1{
    color:orange;
    text-align:center;
}
p{
    font-family:"Times New Roman";
    font-size:20px;
}

四个值: 第一个值为左上角，第二个值为右上角，第三个值为右下角，第四个值为左下角。
三个值: 第一个值为左上角, 第二个值为右上角和左下角，第三个值为右下角
两个值: 第一个值为左上角与右下角，第二个值为右上角与左下角
一个值: 四个圆角值相同

```

## 插入样式表

向 `html` 文档中插入样式表，样式表中不能包含任何的 `html` 标签。

如果某些属性在不同的样式表中被同样的选择器定义，那么属性值将从更具体的样式表中被继承过来。多重样式层叠优先级如下，同一优先级内同样的样式后面定义的具有更高的优先级

> 内联样式 > 内部样式表 > 外部样式表 > 浏览器缺省设置

### 外部样式表

浏览器会从文件 `mystyle.css` 中读到样式声明，并根据它来格式文档，外部样式表可以在任何文本编辑器中进行编辑，外部样式表应该以 `.css` 扩展名进行保存。

```html
<head>
<link rel="stylesheet" type="text/css" href="mystyle.css">
</head>
```

### 内部样式表

当单个文档需要特殊的样式时，就应该使用内部样式表。

```html
<head>
<style>
	hr {color:red;}
	p {margin-left:20px;}
	body {background-image:url("images/back40.gif");}
</style>
```

### 内联样式

由于要将表现和内容混杂在一起，内联样式会损失掉样式表的许多优势。例如当样式仅需要在一个元素上应用一次时，可以使用内联样式。要使用内联样式，你需要在相关的标签内使用样式`style` 属性。`style` 属性可以包含任何 `CSS` 属性。

```html
<p style="color:red;margin-left:20px">This is a paragraph.</p>
```




## CSS 背景
|Property|描述|
|:--|:--|
|background	|简写属性，作用是将背景属性设置在一个声明中。|
|background-attachment	|背景图像是否固定或者随着页面的其余部分滚动。|
|background-color|	设置元素的背景颜色。|
|background-image|	把图像设置为背景。|
|background-position|	设置背景图像的起始位置。|
|background-repeat|	设置背景图像是否及如何重复。|

```css
1. 背景颜色(background-color)
定义颜色的三种方式
- 十六进制："#123456"
- RGB颜色：rgb(255,0,0)
- 颜色名称：red,green
body {background-color:#b0c4de;}

---
2. 背景图像(background-image)
默认情况下，背景图像进行平铺重复显示，以覆盖整个元素实体.
body {background-image:url('paper.gif');}

---
3. 背景图像平铺方式(background-repeat)
默认情况下，图像会水平和垂直平铺，使用background-repeat指定平铺方式
水平方向平铺 background-repeat:repeat-x; 
垂直方向平铺 background-repeat:repeat-y; 
不平铺 background-repeat:no-repeat; 
body
{
background-image:url('img_tree.png');
background-repeat:no-repeat;
}

---
4. 背景图像滑动方式(background-attachment)
scroll 默认值。背景图像会随着页面其余部分的滚动而移动。
fixed 当页面的其余部分滚动时，背景图像不会移动。
inherit 规定应该从父元素继承 background-attachment 属性的设置
body { 
background-image: url(bgimage.gif);
background-attachment: fixed;
}

---
5. 背景图像的位置(background-position)
指定背景图像的位置
right top left bottom
body
{
background-image:url('img_tree.png');
background-repeat:no-repeat;
background-position:right top;
}

---
6. 简写属性(backgound)
简写属性的顺序为
    1. background-color
    2. background-image
    3. background-repeat
    4. background-attachment
    5. background-position
body {
    background:#ffffff 
    url('img_tree.png') 
    no-repeat 
    right top;
}
```

## CSS 文本
|Property|描述|
|:--|:--|
|color|	设置文本颜色 body {color:blue;}|
|direction|	设置文本方向。|
|letter-spacing|	设置字符间距|
|line-height|	设置行高|
|text-align|	对齐元素中的文本|
|text-decoration|	向文本添加修饰|
|text-indent|	缩进元素中文本的首行|
|text-shadow|	设置文本阴影|
|text-transform|	控制元素中的字母|
|unicode-bidi|	设置或返回文本是否被重写 |
|vertical-align	|设置元素的垂直对齐|
|white-space|	设置元素中空白的处理方式|
|word-spacing|	设置字间距|

```css
1. 文本颜色(color)
对于W3C标准的CSS：如果你定义了颜色属性，你还必须定义背景色属性。
body {color:blue;}
h1 {color:#00ff00;}
h2 {color:rgb(255,0,0);}

---
2. 文本对齐方式(text-align)
四种取值：right left center justify(自适应窗口对齐对)
p.date {text-align:justify;}

---
3. 文本修饰(text-decoration)
三种取值：overline(上划线)  line-through(删除线)  underline(下划线)
h1 {text-decoration:overline;}

---
4. 文本转换(text-transform)
三种取值：uppercase(全部大写)  lowercase(全部小写) capitalize(首字母大写)
p.uppercase {text-transform:uppercase;}

---
5. 文本缩进(text-indent)
首行文本缩进
p {text-indent:50px;}

---
6. 文字间距(letter-spacing)
三个取值
normal	默认。规定字符间没有额外的空间。
length	定义字符间的固定空间（允许使用负值）。
inherit	规定应该从父元素继承 letter-spacing 属性的值。
h2 {letter-spacing:-3px}

---
7. 文字高度(line-height)
normal	默认。设置合理的行间距。
number	设置数字，此数字会与当前的字体尺寸相乘来设置行间距。
length	设置固定的行间距。10px
%	基于当前字体尺寸的百分比行间距。
inherit	规定应该从父元素继承 line-height 属性的值。
p.big {line-height:200%;}
p.test {line-height:10px;}

---
8. 文字阴影(text-shadow)
h-shadow	必需。水平阴影的位置。允许负值。
v-shadow	必需。垂直阴影的位置。允许负值。
blur	可选。模糊的距离。
color	可选。阴影的颜色。参阅 CSS 颜色值。
h1 {
color:white;
text-shadow:2px -2px 3px #000;
}

--- 
9. 文本书写方向多语言支持(unicode-bidi)
unicode-bidi 属性与 direction 属性一起使用，来设置或返回文本是否被重写，
以便在同一文档中支持多种语言。
如阿拉伯语言是从左到右书写的。使用该属性可以改变区域内文字书写方向
normal	默认。不使用附加的嵌入层面。 
embed	创建一个附加的嵌入层面。	 
bidi-override	创建一个附加的嵌入层面。重新排序取决于 direction 属性。 
initial	设置该属性为它的默认值。请参阅 initial
inherit	从父元素继承该属性。请参阅 inherit。
div.ex1
{
	direction:rtl;
	unicode-bidi:bidi-override;
}

---
10. 文字垂直对齐(vertical-align)
baseline	默认。元素放置在父元素的基线上。
sub	垂直对齐文本的下标。
super	垂直对齐文本的上标
top	把元素的顶端与行中最高元素的顶端对齐
text-top	把元素的顶端与父元素字体的顶端对齐
middle	把此元素放置在父元素的中部。
bottom	把元素的顶端与行中最低的元素的顶端对齐。
text-bottom	把元素的底端与父元素字体的底端对齐。
length	 
百分比%	使用 "line-height" 属性的百分比值来排列此元素。允许使用负值。
inherit	规定应该从父元素继承 vertical-align 属性的值。

--- 
11. 文字空白处理(white-space)
normal	默认。空白会被浏览器忽略。
pre	空白会被浏览器保留。其行为方式类似 HTML 中的 <pre> 标签。
nowrap	文本不会换行，文本会在在同一行上继续，直到遇到 <br> 标签为止。
pre-wrap	保留空白符序列，但是正常地进行换行。
pre-line	合并空白符序列，但是保留换行符。
inherit	规定应该从父元素继承 white-space 属性的值

---
12. 单词间距(word-spacing)
单词之间的间距，不同于letter-spacing(字母字符间距)
normal	默认。定义单词间的标准空间。
length	定义单词间的固定空间。
inherit	规定应该从父元素继承 word-spacing 属性的值。
```

## CSS 字体
|Property|描述|
|:--|:--|
|font|	在一个声明中设置所有的字体属性|
|font-family|	指定文本的字体系列|
|font-size|	指定文本的字体大小|
|font-style|	指定文本的字体样式|
|font-variant|	以小型大写字体或者正常字体显示文本。|
|font-weight|	指定字体的粗细。|
```css
1. 字体系列(font-family) 
font-family 属性应该设置几个字体名称作为一种"后备"机制，如果浏览器不支持第一种字体，他将尝试下一种字体。
注意: 如果字体系列的名称超过一个字,多个汉字或多个单词，它必须用引号
p.serif{font-family:"Times New Roman",Times,serif;}

---
2. 字体样式(font-style)
normal italic斜体 oblique斜体
p.oblique {font-style:oblique;}

---
3
```


## 盒模型

盒模型是一个种布局方式，每个元素都被表示一个矩形的盒子，有尺寸大小、属性、颜色、边框和位置（渲染）目标。

盒模型默认的值是content-box，CSS3中新增了一种盒模型计算方式：padding-box，还有常用的 border-box，几种盒模型计算元素宽高的区别如下：

- content-box（默认）

```
width = width + padding-left + padding-right + border-left + border-right

height = height + padding-top + padding-bottom + border-top + border-bottom
```

- padding-box（Css 3）

```
width = width(包含padding-left + padding-right) + border-top + border-bottom

height = height(包含padding-top + padding-bottom) + border-top + border-bottom
```

- border-box

```
width = width(包含padding-left + padding-right + border-left + border-right)

height = height(包含padding-top + padding-bottom + border-top + border-bottom)
```