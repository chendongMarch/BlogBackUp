---
layout: post
title: Android 性能优化 - UI [进阶]
category: Android
tags:
  - Android
abbrlink: d1ceed0f
keywords:
  - UI
  - 优化
location: 杭州余杭-尚妆
photos: 'http://olx4t2q6z.bkt.clouddn.com/18-3-7/13307025.jpg'
date: 2018-03-07 00:09:47
---

本文主要总结和记录 `Android` 开发过程中对 `UI` 绘制上的优化，优化 `UI` 绘制可以减少绘制的时间，尽可能快速的将界面展示出来，还可以减轻 `CPU` 的压力，避免过度绘制，保证 `UI` 的流畅度。

<!--more-->

## 前言

界面 `UI` 每隔 16 ms 请求绘制一次，相当于 `1000ms/16` => `60fps`，`60fps` 是人能感觉到的最高帧率，也就是说超过 `60fps` 是没有必要的，同时，如果低于 `30fps` 将会无法流畅展示内容。


## 过度绘制

- 调试过度绘制

通过手机的开发者选项可以调试过度绘制，<span class="spec">设置 -> 开发者选项 -> 调试GPU过度绘制 -> 显示GPU过度绘制 </span>


打开调试过度绘制以后，界面会显示不同的颜色，分别代表过度绘制的次数：

|颜色|描述|表示|
|:--|:--|:--|
|蓝紫色| overdraw 1倍|绘制了2次，大片的蓝紫色是可以接受的|  
|绿色| overdraw 2倍|绘制了3次，中等大小的绿色区域是可以接受的但你应该尝试优化、减少它们|
|淡红| overdraw 3倍|绘制了4次，小范围可以接受|
|深红| overdraw 4倍|绘制了5+次，这是错误的，要修复|

查看微信的过度绘制情况，首页基本在2倍以内，其他页面列表有3倍的情况，4倍的绘制一般出现在极小的区域内，比如文字和图标等。

目标是尽量增加蓝紫色的区域，减少红色区域。

- GPU 使用

通过手机的开发者选项可以开启 `GPU` 呈现模式分析， <span class="spec">设置 -> 开发者选项 -> GPU呈现模式分析 </span>，可以查看某个界面的 `GPU` 占用情况。


## UI 层级

如果布局层级嵌套过深，也会导致 `UI` 绘制的问题，尤其是使用 `xml` 布局文件，因为一方面解析 `xml` 就需要耗费大量的 `CPU`，另外布局 `measure` 的时候，子布局需要告知父布局自己的大小和占据的位置，层级过多之后就会占用更多的时间和内存。

`Hierarchy Viewer` 是一个查看 `UI` 布局层级的工具，使用 `AndroidStudio` 在 <span class="spec">Tools -> Android -> Device Monitor</span>

不过真机无法调试，只能使用模拟器，不过大家可以参考这个项目 [GitHub-ViewServer](https://github.com/romainguy/ViewServer)，不过在电脑上面查看会很卡，而且拖动起来很不方便。

如果项目中使用了网络框架，可以使用：

```gradle
compile 'com.facebook.stetho:stetho:1.4.2',
compile 'com.facebook.stetho:stetho-okhttp3:1.4.2',
```
然后在 `chrome://inspect` 查看布局，点击指定布局，手机会同时高亮显示
 


## UI 优化解决办法

针对以上两种情况，解决 `UI` 绘制问题主要是 **减少过度绘制** 和 **减少布局层级** 两个方面。

> - 删除重复无用的背景

如果层级覆盖的情况下，优先设置底层 `View` 的背景，顶层如果具有相同颜色的背景，就不要重复设置啦。

如果底层的 `View` 覆盖了整个屏幕，那么 `Window` 的背景也是不需要的，可以使用：

```java
getWindow().setBackgroundDrawable(null);
```

> - 合理选择布局

布局要遵循 **增加宽度，减小深度** 的原则，尽可能的减少 `UI` 层级

不使用绝对布局 `AbsoluteLayout`。

复杂布局，优先使用 `RelativeLayout`，可以更好的控制子控件的位置。

相同的层级情况下，优先使用 `LinearLayout`，他的布局效率更高。

尽可能少用 `layout_weight` 属性，他会造成多次测量。

使用 `ConstraintLayout`，可以很好的减少布局的层级。
 
> -  Xml Drawable

规则图形，尽量使用 `shape.xml` 代替图片

使用 `selector` 时，将 `normal` 状态下的颜色设置为 `transparent`
 
> -  Include

使用 `<include/>` 标签复用布局，这个其实并不能起到优化 `UI` 绘制的作用，不过讲合适的布局分离处理做成独立的 `xml`，可以更好的将布局组件化。

```xml
<include
    layout="@layout/activity_about_us"
    android:layout_width="match_parent"
    android:layout_height="100dp"
    android:layout_margin="10dp"
    android:orientation="horizontal"
    android:visibility="visible"/>
```

使用 `<include/>` 需要注意几点，在 `<include/>` 标签中可以使用部分属性来从新定义布局，如 `width`、`height`、`visibility`、`margin`、`id` 等，如指定了 `id`，将会覆盖 `<include/>` 里面根布局的 `id`，造成查找不到的情况。

> -  ViewStub

如果一个控件大多数情况下不进行显示，那么使用 `ViewStub` 要比使用 `Visibility` 更好，他不会占据任何位置，解析 `xml` 也不会耗费更多资源。

```xml
<ViewStub
    android:id="@+id/viewstub"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout="@layout/activity_about_us"/>
```

当需要展示内部布局时，使用 `inflate()` 展开布局，布局展开后，`ViewStub` 里面的内容就会代替 `ViewStub` 的位置，同时 `ViewStub` 就查找不到了，而且我们也不希望展开多次，所以要做一下空判断，不为空时，使用 `setVisibility` 展示布局。

```java
private View viewStubView;
private void openViewStub() {
    // 已经展开，直接使用 setVisibility
    if (viewStubView != null) {
        viewStubView.setVisibility(View.VISIBLE);
    } else {
        ViewStub viewStub = findViewById(R.id.viewstub);
        // 没有展开，使用 ViewStub
        if (viewStub != null) {
            viewStubView = viewStub.inflate();
        }
    }
}    
```

> -  Merge

使用 `xml` 布局时，最外层只能有一个父控件，当我们希望把其中部分提取到 `<include/>` 中时，就需要在子布局最外层再加一层控件包含，这样就增加了布局的深度。

`<merge/>` 就是为了解决上述问题，它主要是用来将两个相同的布局合并为一个，主要用于两种情况：

第一种情况是，调用 `setContentView()` 时，其实是将我们的布局设置到了 `id` 为 `R.id.content` 的 `FrameLayout` 中，那么如果我们的顶层布局也是 `FrameLayout`，同时没有 `background` 和 `padding` 等属性，那么就可以使用 `<merge/>`，将二者合为一个，理解起来也简单，本来就是两个一样的布局套在一起，合为一个并不影响。

第二种情况是，使用 `<include/>` 时也是一样的道理，如果 `<include/>` 中的顶层布局和他将要添加到的布局里面的 `layout` 是一样的，那么就可以使用 `<merge/>`，节省一层布局，如下：

```xml
// test_layout.xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <TextView ... />
    <include layout="@layout/test_item_layout"/>
</LinearLayout>

// test_item_layout.xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <TextView ... />
    <TextView ... />
</LinearLayout>
```
由于 `<include/>` 顶层 `View` 也是 `LinearLayout`，因此可以使用 `<merge/>` 代替

```xml
<merge
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <TextView ... />
    <TextView ... />
</merge>
```
  

 