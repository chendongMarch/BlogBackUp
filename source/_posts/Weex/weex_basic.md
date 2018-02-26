---
layout: post
title: Weex 开发 [Weex]
categories:
  - weex
tags: weex
keywords:
  - weex
abbrlink: 41ed2d2f
date: 2017-11-16 11:13:00
location: 杭州上装
photos: http://olx4t2q6z.bkt.clouddn.com/18-2-1/83676814.jpg
---


前几天看知乎，`Evan You` 叫 `Weex` 作 `Vue-Native`😊

本文主要学习如何使用 `Weex` 常用组件进行开发工作，再根据开发过程中遇到的问题，完善补充。	

<!--more-->

## Tips

- 不支持相对单位 `em`，`rem` 等，只能使用 `px`；

- 不支持 `css` 组合写法，只能设置一个属性值，如 <del> `border: 1 solid red;` </del>

- `<image>` 在 `ios` 平台 `border` 不支持设置不同的 `radius`；

- 不支持百分比

- 根布局必须是 `<div>`、`<scroller>`

- 事件最好定义成有意义的形式，不然有时会出问题，例如 `@change="change"` 监听 `change` 事件没问题，但是 `@scroll="scroll"` 就会出现监听不到的问题。

- 标签属性一定要写成标签属性的形式 `porperty=value`，不能用 `css` 属性代替，有些属性在 `Android` 上面支持，但是在 `ios` 那边不支持。

- 当不是根元素的 `div` 嵌套列表时，比如 `list`，列表在 `ios` 上无法显示出来，要显式的指定 `div` 的高度，有点类似两个可滑动的组件套在一起，无法测量高度的情况。

- 使用 `:style` 支持动态设置多个属性时，如果值带有单位，则一定要使用 `''` 包含，中间以逗号分隔，如 `:style="{width:'750px',color:red,height:height+'px'}"`
 

## 盒模型

<image width=200px; src="http://olx4t2q6z.bkt.clouddn.com/17-11-16/13369345.jpg"></br>

`Weex` 盒模型的 `box-sizing` 默认为 `border-box`，即盒子的宽高包含内容、内边距和边框的宽度，不包含外边距的宽度。

width/height - 指的是 `content + padding + border` 部分的尺寸

```CSS
.frame{
  width:100px;
  height:100px;
}
```

padding - 内边距，`content` 和 `border` 之间的距离

```CSS
.frame{
  padding:100px;
  padding-top:100px;
  padding-xxxx:100px;
}
```

margin - 外边距，元素与元素之间的距离

```CSS
.frame{
  margin:100px;
  margin-top:100px;
  margin-xxxx:100px;
}
```

border - 边框

```CSS
.frame{
  /* solid 直线，默认值；dashed 虚线；dotted 点线；*/
  border-style: solid | dashed | dotted;
  border-xxxx-style: solid | dashed | dotted;

  border-color:#0f0f0f;
  border-xxxx-color:#0f0f0f;

  /* 先 top/bottom，后 left/right */
  border-radius:10px;
  border-xxxx-yyyy-radius:10px;
}
```


## FlexBox

`flexbox` 是 `CSS3.0` 新的布局模式，`Weex` 对它进行了部分的支持。

`flexbox` 是唯一的默认的布局模式，不需要指定 `display: flex;`，默认所有的容器类元素都是 `flex` 容器。

布局方向

```css
.frame{
  /* row,左到右排列；column,上到下排列*/
  flex-direction: row | column;
}
```

主轴方向元素的排列

```css
.frame{
  /*
  flex-start : 元素全部靠左
  flex-end : 元素全部靠右
  center : 元素居中
  space-between : 元素之间间距相同，分散排列，两边不留白
  space-arround : 元素之间间距相同，分散排列，两边留间距的一半
  */
  justify-content: flex-start | flex-end | center | space-between | space-arround;
}
```

侧轴方向元素的排列。

```css
.frame{
  /*
  flex-start : 元素靠顶部对齐；
  flex-end : 元素靠底部对齐；
  center : 居中 
  stretch : 拉伸到 flex 容器大小
  */
  align-items : flex-start | flex-end | center | stretch;
}
```

flex 成员项

`flex` 属性定义了 `flex` 成员项可以占用容器中剩余空间的大小。如果所有的成员项设置相同的值 `flex: 1`，它们将平均分配剩余空间. 如果一个成员项设置的值为 `flex: 2`，其它的成员项设置的值为 `flex: 1`，那么这个成员项所占用的剩余空间是其它成员项的 2 倍。

```css
.frame{
  flex:2;
}
```

## pseudo

主要支持 `active`，`focus`，`enabled`，`disabled`。所有组件支持 `active`，只有 `input` 和 `textarea` 支持 `focus`。

在客户端中，`active` 表示组件被按压时的状态，`focus` 表示组件获取到焦点时的状态，`enabled` 表示可用时的状态，`disable` 表示不可用时的状态。

在 `ios` 上，`enable` 没有效果。

优先级：

```
active > focus > enable
active > disabled 

disabled 是一种消极状态，disabled 和 focus、enable 这类积极的状态不会冲突。
```

## div

不可嵌套层级过深，控制在 10 层以内，避免性能问题；

不能在 `div` 中直接写文字，需要使用 `<text>` 组件；

即使高度超出范围 `div` 也不能滑动；

## web

不支持 `click` 和 `longpress` 事件

标签属性 

```javascript
// 加载的地址
src = "https://m.alibaba.com"
```

事件，`event` 里面内容很多

```javascript
// 页面开始加载
@pagestart

// 页面加载完成
@pagefinish

// 加载出现错误
@error

```

## video

内部只能包含 `<text>` 标签

标签属性

```javascript
// 播放地址
src="url"

// 播放状态，控制这个值可以控制播放和暂停
play-status="play|pause"

// 加载完成后自动播放
auto-play = true
```

事件

```javascript
// 每次开始播放，自动播放或者暂停后播放都会触发
@start

// 暂停时触发
@pause

// 播放完成时触发
@finish

// 播放失败时触发
@fail
```

## switch

标签属性

```javascript
checked = true // 是否选中
disabled = false // 是否不可用
```

文档中提到，下面的样式不能使用，测试发现，`width／height`、`margin`、`border` 这些属性都是可以使用的，不过确实有部分属性在 `ios` 上面没有效果。

```css
width
height
min-width
min-height
margin
padding
border
```

事件

```javascript
// 状态切换时触发
@change

event => 
	value {boolean} : 选中否
	timestamp : 时间戳
```

## A

它是一个容器类的标签，看 `Android` 这边的源码，是继承自 `div` 标签，`a` 和 `html` 里面的 a 标签差不多，但是不能包含文本，如果要有文本需要使用 `<text>` 标签。

点击之后会调用一个名为 `event` 的 `module` 的 `openURL` 方法，这个 `module` 需要我们来注入到 `sdk` 中，参数就是 `href`。

```html
<a class="button" href="http://g.tbcdn.cn/ali-wireless-h5/res/0.0.16/hello.js">
  <text class="text">Jump</text>
</a>
```

## image

「`image`」标签用来显示图片，必须指定 `width` 和 `height`，否则无法显示，不能包含子组件，

标签属性

```javascript
// 显示的图片地址
src = "http://www.abc.com/xxx.jpg" 

// 图片下载时的默认图，没有找到方法测试 
placeholder = ".."

// 显示模式
// stretch 是默认值，拉伸图片填充，fitXY
// cover 将会放大图片然后裁剪，centerCrop
// contain 长边填充满，短边按比例，保证图片比例不被破坏，fitCenter
resize = stretch | cover | contain
```

事件

```javascript
// 图片加载成功监听
@load = "loadImageOver"
result =>
	success {boolean}: 是否加载成功
	size => 图片原始宽高
		naturalWidth {number}
		naturalHeight {number}
```

方法

```javascript
// 将图片保存到本地
save(callback(result))

result =>
	success {boolean}: 是否保存成功
	errorDesc {string}: 错误信息，success为false时返回

eg:
var el = this.$refs.image_ref;
el.save(result => this.logt("save image " + result.success));
```


## text + input + textarea

文本样式汇总

```css
.mytext {
  lines: 2; /* 文本行数，默认0不限制 */
  color: red; /* 文本颜色*/
  font-size: 40px; /* 文字大小 ios,h5默认32,android分设备不同*/
  font-style: italic; /* normal正常，italic斜体 */
  font-weight: bold; /* 值必须为整百的数字,normal(400)bold(700),ios支持100-900,android支持normal和bold，也就是400，700，lighter和bolder不支持 */
  text-decoration: underline; /* underline(下划线) line-through(横划线) none(无，默认) */
  text-align: center; /* left center right 默认值为 left，目前暂不支持 justify, justify-all */
  text-overflow: ellipsis; /* 文字超出后截断样式，clip 直接截断，ellipsis 显示省略号 */
  font-family: "iconfont2";
}
```

「`text-decoration`」，已经支持前后带有空格显示，当带有空格时，如果加了 `decoration`，在 `android` 上面空格的地方也会有装饰，`ios` 只会在文字的地方有。

「`font-family`」，在生命周期方法 `beforeCreate` 为页面加载一套字体

```javascript
beforeCreate() {
    var domModule = weex.requireModule("dom");
    // 目前支持ttf、woff文件，不支持svg、eot类型,moreItem at http://www.iconfont.cn/
    domModule.addRule("fontFace", {
    fontFamily: "iconfont2",
        src: "url('http://at.alicdn.com/t/font_1469606063_76593.ttf')"
    });
}
```

### text

组件 `text` 具有 `value` 属性，用来表示显示的值，当 `value` 和内容都指定时，将会显示内容的值。

```html
<text class='mytext' value="test1"> test2 </text>
```


### input

组件 `input` 标签属性，需要注意的是如果标签中间写了文案，会显示在组件下方。

```javascript
maxlength = 10 // 可以输入的最长长度
return-key-type = 'done' // default,done,search,send,go,next 回车键的样式
disabled = false // 是否可以输入，默认false,为true时不可以输入
autofocus = true // 自动获得焦点，弹出键盘
placeholder = '请输入' // 输入提示文案
type = "number" // 输入类型，text,password,url,email,tel,number 约束不是强约束，用户一样可以切换键盘样式。
value = "默认文案" // 默认填充的文字

<input class='myinput'
       maxlength=10
       return-key-type='done'
       disabled=false
       autofocus=true
       placeholder='hint'
       type="number"
       value="11"> 这里的文案会显示在组件下面 </input>
```

伪类

```css
.myinput:active {
  color: purple; /* press时的状态，*/
}
.myinput:focus {
  color: green; /* 获得焦点的状态*/
}
.myinput:enabled {
  color: orange; /* 可用时的状态，ios 无效果*/
}
.myinput:disabled {
  color: sienna; /* 不可用时的状态*/
}
```

支持的 `css` 属性

```css
同样支持 color,font-size,fone-weight,font-style,text-align 属性

.myinput {
  /*提示文案的颜色*/
  placeholder-color: blue;
}
```

事件

```javascript
@input 输入监听，每次输入内容变化，会调
event => 
	value {string} : 文本内容
	timestamp : 时间戳


@focus 获得焦点监听
event =>
	timestamp : 时间戳
	
	
@blur 失去焦点监听
event => 
	timestamp : 时间戳
	
	
@change 输入完成且内容发生改变监听，发生在 blur 之后
event => 
	value {string} : 文本内容
	timestamp : 时间戳


@return 键盘回车键监听
event=>
	returnKeyType {string} : 设置的return-key-type
	value {string} : 文本内容
```

方法，为 `input` 组件设置 `ref="input_ref"`

```javascript
var el = this.$refs["input_ref"];

使 input 获取焦点，弹出键盘
focus() 
eg:
el.focus();


使 input 失去焦点，隐藏键盘
blur() 
eg:
el.blur();


设置选中区域
setSelectionRange(start,end)
eg:
el.setSelectionRange(0,el.attr.value.length);


获取选中区域， ios 不支持
getSelectionRange(event{selectionStart,selectionEnd})
eg:
el.getSelectionRange(event => {
    this.logt(event.selectionStart + " | " + event.selectionEnd);
});
```

### textarea

多行文本输入 `textarea` 和 `input` 基本类似，

他们俩的差别：

事件上，`textarea` 不支持 `@return` 事件

方法上， 均支持

标签属性上，增加了 `rows` 属性，不支持 `return-key-type`，`max-length`，`type`，支持的属性汇总如下：

```javascript
rows = 3 // 指定组件的高度，默认2
disabled = false // 是否可以输入，默认false,为true时不可以输入
autofocus = true // 自动获得焦点，弹出键盘
placeholder = '请输入' // 输入提示文案
value = "默认文案" // 默认填充的文字
```



## slider + indicator

`slider` 默认高度会占满屏幕，所以必须为 `slider` 指定一个高度，而 `indicator` 也会继承这个高度。

```html
<slider class="slider" interval=3000 auto-play=true infinite=true  @change="changePage" @scroll="scrollSlider	">
	<div v-for="(img,index) in imagelist">
		<image class='image' :src="img"></image>
	</div>
	<indicator class="indicator"></indicator>
</slider>
```


slider 标签属性

```js
interval = 3000; // 播放时间间隔 
auto-play = true; // 自动播放
infinite = true; // 无限循环
offset-x-accuracy = 20px; // 控制onscroll事件触发的频率，默认值为10,越小的话 onscroll 触发越频繁，可能降低性能
```

slider css 属性

```css
.slider{
  height:400px; /* 高度，需要指定 */
  scrollable:false; /* 可不可以手势滑动 */
}
```
 
indicator 常用属性

```css
.indicator{
  width:750px;
  height:400px;
  position:absolute;
  top:150px;  /* 绝对定位距离顶部的距离 */
  item-color:#eeeeee;  /* 没有选中的颜色 */
  item-selected-color:#ff0000; /* 选中的颜色 */
  item-size:20; /* 指示标记大小 */
}
```

事件

```js
// change 事件，当页面滑动到下一页时触发
@change = 'changePage'
event =>
	index {number}   滑动到的索引

// scroll 事件，每一页的移动都会触发，触发频率取决于 offset-x-accuracy
@scroll 滑动监听
event =>
	offsetXRatio {number}  偏移量，[-1,1]，负值表示向左滚动，表示当前有每一页的 offsetXRatio 滚动到了页面外
```

## scroller

对客户端来说是不可复用元素的列表

滑动方向 - 可以垂直或者水平滑动，`flex` 和 `scroll` 的方向必须一致。

```html
两者方向必须一致
flex-direction 默认值 row 
scroll-direction 默认值 vertical

如果是垂直滑动，
flex-direction:column;
scroll-direction = vertical

如果水平滑动
flex-direction:row;
scroll-direction = horizontal
```

实现一个 tab 导航

```html
<scroller class="scroller" scroll-direction=horizontal @scroll='scrollerScroll'>
    <div class="tab-group"  v-for="(item,index) in tablist" @click="changeTab(index)" :ref="'tab'+index">
        <text class="tab-item"> {{item}} </text>
        <text> 抢购中 </text>
        <div class="tab-line" v-if="index == current_tab">  </div>
    </div>
</scroller>
```

常用标签属性，同样支持一些列表统一的标签属性 [列表标签属性](#列表标签属性)

```js
// 滑动方向，默认 vertical
scroll-direction = vertical|horizontal
```

支持 `loadmore` 事件，见 [列表 loadmore](#列表事件)

支持 `scroll` 事件，见 [列表 scroll](#列表事件)

支持 `resetLoadmore`，见 [列表 resetLoadmore](#扩展-API)

支持 `scrollToElement`，见 [列表 scrollToElement](#扩展-API)


## list + waterfall + cell + header

 

把他们放在一起是因为他们一起完成了一个列表的显示，其实 `waterfall` 和 `list` 是一样的， 类似 `Android` 上面当年的 `ListView` 和 `GridView`，当然现在都已经被 `RecyclerView` 代替了，他们表现的都是一个 **垂直列表**，而 `waterfall` 支持多列的显示，

使用 `cell` 生成列表的每一项，`header` 其实是 `cell` 的子类，它可以在滑动出屏幕时，固定在屏幕顶端，实现悬挂的效果。
 
列表中只支持 `cell` 、`header`、`refresh`、`loading` 和 使用 `fix` 定位的组件，其他组件不能正确渲染。

实现一个简单的列表显示，需要格外注意的是，`v-for` 的语句要写在 `cell` 上面，这样才能保证生成多个 `cell`，开始的时候我写在了 `div` 上面，其实是在一个 `cell` 里面生成了多个 `div`，这样就无法实现回收和复用。

```html
<list style="height:500px;" @loadmore="loadmorelist">
    <cell v-for="(item,index) in product_list">
        <div  class="product-group" @click="clickProduct(item,index)">
            <text class="origin-price"> ¥ {{item.origin_price}}</text>
        </div>
    </cell>
</list>
```

### header

这个 `header` 用起来坑比较多，汇总一下：


1. 在 `list` 里面用 `header`，默认是一样悬挂效果，就是当 `header` 向上将要被滑出屏幕时会固定在屏幕顶端，但是向下滑出时不会固定在底端。
2. `list` 里面 `header` 上面的 `cell` 不能全部使用 `v-for` 生成，如果是的话，对 `header` 没有挤压效果，`header` 会一直在顶部，就好像 `v-for` 生成的那些 `cell` 没有压在 `header` 上面，但是一旦有一个不是 `v-for` 生成的，就好了，这应该是个bug;
3. `waterfall` 里面使用 `header` 时，默认是没有悬挂效果的，会像普通的 `cell` 一样跟着整个列表一起走，这样可以实现单列和多列并存的效果。
4. 如果希望 `waterfall` 里面有悬挂效果，要对 `header` 设置 `position:sticky;`，使用这个属性，他不会滑出屏幕，不管上面还是下面都会挂住。
5. 在 `list` 中使用 `sticky` 没什么效果，在 `waterfall` 中一方面可以使用 `sticky` 对 `header` 做悬挂效果，另一方面 `cell` 如果设置了 `sticky` 会变成跨越正行，类似 `header`


使用 `header` 标签实现悬挂效果

`<header/>` 标签是 `list` 可以渲染的标签之一，当 `header` 到达顶部时，会吸附在屏幕顶部，`<header/>` 不一定要在顶部，可以在列表的任何位置，当这一项，被滑动到顶端时，会吸附在顶部，不会划出屏幕。

ps：同样的效果，如果在 `cell` 上面使用 `position:fixed;` 在 `Android` 可以正确显示，但是在 `ios` 没有任何效果 

使用方法很简单，只需要将某一项使用  `<header>` 包含即可。

```html
<list>
	<cell></cell>
	<cell></cell>
	<header></header>
	<cell></cell>
</list>
```


### waterfall

👮如果是一个 `list` 套在 `div` 里面显示没问题，但是如果是 `waterfall` 就不行，看到有报错说无法解析 `auto`，当然设置成固定值也不行，解决方案是给  `waterfall` 一个固定的宽度，让他可以计算，就没问题了。

瀑布流 `waterfall` 支持多列显示，因此相比 `list` 多了一些属性，但是 `list` 的属性和事件他也是支持的。

「`waterfall`」 特有属性，`column-width` 和 `column-count` 不能同时指定为 `auto` 无法计算将无法显示。

```javascript
当某个属性指定为 auto 时，会使用下面的约束来计算 auto 对应的值。
totalWidth = column-width * column-count + (column-count-1) * column-gap

column-width: 列宽，可以使用指定宽度，也可以使用 auto 表示根据其他属性来决定

column-count: 列数，可以使用指定数字，也可以使用 auto 表示根据其他属性决定。

column-gap: 间隔，可以使用指定宽度，也可以使用 normal 表示 32。
```

### common

「`waterfall` 和 `list`」 常用属性 - 支持列表统一的标签属性 [列表标签属性](#列表标签属性)

「`waterfall` 和 `list`」 支持 `loadmore` 事件，见 [列表 loadmore](#列表事件)

「`waterfall` 和 `list`」 支持 `scroll` 事件，见 [列表 scroll](#列表事件)

「`waterfall` 和 `list`」 支持 `resetLoadmore`，见 [列表 resetLoadmore](#扩展-API)

「`waterfall` 和 `list`」 支持 `scrollToElement`，见 [列表 scrollToElement](#扩展-API)

## 列表

> 在 weex 指的就是 scroller 和 list 了，这里汇总一些公共的属性和方法，方便管理和对比。

类似 `scroller` 和 `list` 都属于列表，他们有一些通用的属性和方法，提取出来统一介绍，⚠️ 不允许相同方向的列表互相嵌套


### 列表标签属性

```js
// 屏幕底部到页面底部的距离，用来触发 loadmore，默认 0
loadmoreoffset = 20 
// 触发 scroll 的频率，默认 10px
offset-accuracy = 20px
// 显示滚动条，默认true，但是发现在 android 默认是不显示的，但是 ios 显示
show-scrollbar = false
```

### 列表事件


```js
@loadmore 加载更多事件，横向滑动时，`loadmore` 不能触发
没有参数


@scroll  滑动监听
event => 
	contentSize {Object} 列表的尺寸，宽度和高度是总的长度，如 150px * 10 = 1500px
		- width {number}
		- height {number}
	contentOffset {Object} 列表偏移尺寸，整个列表总共被滑动的 px 数，是一个负值，越往右|下滑动，绝对值越大
		- x {number}
		- y {number}
```


### 方法
 
> resetLoadmore(node,options)

重置 `loadmore`，如果进行过一次 `loadmore` 之后，列表内容没有发生变更，则再次滑动到末尾时，不会触发 `loadmore` 事件，需要使用 `resetLoadmore()` 来重置状态。

```js
scroller.resetLoadmore(node,options)

node {node} 指的是节点，暂时不知道做什么用的，传任何节点都可以起作用
options {object}
	- offset {number} 一个到其可见位置的偏移距离，默认 0

eg:
// 设置 scroller 的 ref 为 sv
// 在加载更多的事件中，重置状态，让他可以一直触发 loadmore 事件
loadmoreScroll(){
	const el = this.$refs.sv
	el.resetLoadmore(el,{offset:20})
}
```
 

> scrollToElement(node,options)

使得列表滑动到指定的 node，需要使用 `dom module` 来操作

ps：使用过程中发现 `waterfall` 组件的 `scrollToElement` 在 `andorid` 上是将指定项滚动到屏幕底部，`ios` 是滚动到顶部，`list` 和 `scroller` 正常，都是到顶部。

```js
scrollToElement(node,options)

node {node} 列表的子节点，将会滑动到这个节点
options {object}
	- offset {number} 指定的子节点滑动后距离左边的距离的负值
	- animated {boolean} 动画效果，默认 true
```

示例，对一个横向列表进行操作，使得点击的那个子元素自动滑动到居中显示

```html
<scroller class="scroller" scroll-direction='horizontal'>
    <div class="tab-group"  v-for="(item,index) in tablist" @click="changeTab(index)" :ref="'tab'+index">
        <text class="tab-item"> {{item}} </text>
    </div>
</scroller>
```
在 `click` 事件中，滑动 `scroller`

由于我每一个子项的宽度设置为了 `150px`，而 `weex` 默认屏幕是 `750px`，一次屏幕内正好可以放置 5 个子项，距离左边 `300px` 即可居中。

```js
changeTab(index){
    const dom = weex.requireModule('dom')
    this.current_tab = index;
    const el = this.$refs['tab'+index][0];
    dom.scrollToElement(el,{offset:-300});
},
```

## refresh + loading

使用 `refresh` 和 `loading` 实现加载数据的效果，`refresh` 是下拉刷新数据，`loading` 是上拉加载更多数据，他们只有在被 `list` 和 `scroller` 包含时才能正确渲染，接下来将会以 `list` 为例简单学习一下使用方法，在 `scroller` 是一样的。在用法上，`refresh` 和 `loading` 也基本相同，只是显示的位置不同和一些事件名字不同。

「`diaplay`」属性来决定组件的展示和隐藏，`display` 值为 `show` 或 `hide`。仅隐藏 `<indicator>`，`<loading>` 其他子组件依然可见，`loading` 事件仍会被触发，因此我们要实现刷新的效果需要动态的修改 `display` 的值。

```html
<refresh :display="isRefreshing?'show':'hide'"></refresh>
```

「`<loading-indicator>`」 是内置一个组件，显示是一个带动画效果的圆形进度条，就是系统默认的加载动画，需要注意的是宽高必须指定，不然默认是0无法显示，可以使用 `color` 属性来更改进度条的颜色。

```html
<loading-indicator style='color:red;width:40px;height:40px;'></loading-indicator>
```

事件，`<refresh>` 有 `@refresh` 事件，`<loading>` 有 `@loading` 事件，他们都会在滑动距离超过整个组件的高度时被触发，可以在这里面做加载数据的操作。

另外 `<refresh>` 还有一个自己的事件，`@pullingdown` 事件会返回组件滑动过程中的一些数据，不过这个监听在 `andorid` 和 `ios` 上面的数据差别比较大。

```javascript
event =>
	dy : 每次移动的变化量，表达的是单位时间内滑动的距离，每次开始动值比较大，快停止时值变小。
	
	pullingDistance : 总共滑动的距离；
	当刷新头露出时，android 返回的值为正数，ios 返回的值为负数；
	当刷新头隐藏时，列表滑动时，android 不会触发这个监听，但是 ios 会一直触发，并且 pullingDistance 变为正数，并且一直会变大。
	
	viewHeight : 刷新组件的高度，滑动超过这个高度会触发刷新
	
	event.type : 不变的字符串 pullingdown
```

完整的带有刷新和加载功能的代码

```html
<list>
    <refresh class='load_part' :display="isRefreshing?'show':'hide'" @refresh='refreshList' @pullingdown='pullList'>
        <loading-indicator style='color:red;width:40px;height:40px;'></loading-indicator>
        <text style='text-align:center'> 下拉开始刷新 </text>
    </refresh>

    <cell v-for="(item,index) in product_list">
        ...
    </cell>

    <loading class='load_part' :display="isLoading?'show':'hide'" @loading='loadingList'>
        <loading-indicator style='color:red;width:40px;height:40px;'></loading-indicator>
        <text style='text-align:center'> 上拉开始加载 </text>
    </loading>
</list>
```

默认 `isFreshing` 和 `isLoading` 自然都是 `false`，当刷新时，更改组件的状态为显示，开始加载数据,并且延时 1s 后将组件隐藏。

```javascript
refreshList(){
    modal.toast({message:"refresh"})
    this.isRefreshing=true
    setTimeout(() => {
        this.isRefreshing=false
    }, 1000);
},
loadingList(){
    modal.toast({message:"loading"})
    this.isLoading  =true
    setTimeout(() => {
        this.isLoading=false
    }, 1000);
},
```

## config

文档说明使用 `weex.config` 获取，但是测试发现不行，需要使用 `this.$getConfig()` 获取到，调查发现 `weex.config` 需要 `weex` 版本 `>=0.9.5` 才可以，而我使用官方的 `playground` 版本是 `0.9.4`。

默认以宽度为 `750px` 做适配渲染，要获得 `750px` 下的屏幕高度，可以通过 `height = 750/deviceWidth*deviceHeight` 公式获得，可以使用到 `CSS` 中，用来设置全屏尺寸。

数据结构：

```
config
	bundleUrl {string} :js bundle 的 url
	env {obj}
		weexVersion {string} :Weex sdk 版本
		appName {string} :应用名字
		appVersion {string} :应用版本。
		platform {string} :平台信息，是 iOS、Android 还是 Web
		osName {string} :iOS或者android，表示操作系统的名称
		osVersion {string} :系统版本
		deviceModel {string} :设备型号 (仅原生应用)
		deviceWidth {number} :设备宽度		
		deviceHeight {number} :设备高度
```