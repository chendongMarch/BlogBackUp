---
layout: post
title: Android V7V4包新控件
date: 2015-03-03
category: Android
tags: [Android,Android5.+]
keywords: 
description: V7新控件
---

## AlertDialog
v7风格的Dialog相对原来的版本去掉了跳跃弹出的动画，下方按钮去掉了边框，聚集在了右侧，显示时去掉了分隔线。

## ToolBar
取代原先的ActionBar实现相对完美的定制
### xml文件
```xml
<android.support.v7.widget.Toolbar
android:id="@+id/md_toolbar"
android:layout_width="match_parent"
android:layout_height="?attr/actionBarSize"
android:minHeight="50dp"
android:background="?attr/colorPrimary"
android:layout_marginBottom="20dp"              
>
```
### 代码
```java
  //隐藏系统默认title,我当时修改了theme用的noactionbar的Theme所以这行代码并没用
getSupportActionBar().setDisplayShowTitleEnabled(false);
Toolbar toolbar = (Toolbar) findViewById(R.id.md_toolbar);
//标题
toolbar.setTitle("测试");
//副标题，显示在标题下方
toolbar.setSubtitle("副标题");
//logo,显示在标题左侧
toolbar.setLogo(R.mipmap.ic_launcher);    
//导航图标，显示在最左侧，可以使用该图标调出菜单
toolbar.setNavigationIcon(android.R.drawable.ic_input_delete);
//这个并没有用，设置菜单时重写onCreateOptionMenu即可
//toolbar.inflateMenu(R.menu.menu_main);
setSupportActionBar(toolbar);
toolbar.setOnMenuItemClickListener(new OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
Toast.makeText(MainActivity.this, "点击了" + item.getItemId(), Toast.LENGTH_SHORT).show();
                return false;
            }
});
```
### 题外话,如何创建菜单项
```
//重写该方法，菜单将会显示在toobar上
@Override
public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        //获取toolbar上的菜单view,也可以在上面的setOnMenuItemClickListener实现更加简单，这里只是一个演示。
        MenuItem item = menu.findItem(R.id.action_share);
        //强转为你需要的view
        View actionView = item.getActionView();
        return super.onCreateOptionsMenu(menu);
}

//菜单文件，为没有接触过的小伙伴准备的额,相关属性请自行查文档
//res/menu/menu_main.xml
<menu 
xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
xmlns:tools="http://schemas.android.com/tools"
tools:context=".MainActivity"
>
<item
android:id="@+id/ab_search"
android:orderInCategory="80"
android:title="action_search"                app:actionViewClass="android.support.v7.widget.SearchView"/>
<item
android:id="@+id/action_share"
android:orderInCategory="90"
android:title="action_share"              app:actionProviderClass="android.support.v7.widget.ShareActionProvider"
app:showAsAction="ifRoom"/>
<item
android:id="@+id/action_settings"
android:orderInCategory="100"
android:title="action_settings"
app:showAsAction="never"/>
</menu>
```
## LinearLayoutCompat
- 在控件中间添加分隔线

```xml
<android.support.v7.widget.LinearLayoutCompat
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:layout_gravity="center|center_horizontal"
android:orientation="vertical"
app:divider="@drawable/line"
app:dividerPadding="4dp"
app:showDividers="middle"
>
.....
</android.support.v7.widget.LinearLayoutCompat>
```

1. `app:divider=”@drawable/line”` 给分隔线设置颜色，这里你需要在drawable在定义shape资源，否则将没有效果。

2. `app:dividerPadding=”25dp”` 给分隔线设置距离左右边距的距离。

3.  `app:showDividers=”middle|beginning|end”` 分隔线显示的位置，有四种参数值：middle 每个item之间，beginning最顶端显示分隔线，end 最底端显示分隔线，none不显示间隔线。


## ListPopupWindow
```java
public void showListPopup(View view) {
    String items[] = {"item1", "item2", "item3", "item4", "item5"};
    final ListPopupWindow listPopupWindow = new ListPopupWindow(this);   
    //设置ListView类型的适配器
    listPopupWindow.setAdapter(new ArrayAdapter<String>(SwipeRefreshActivity.this, android.R.layout.simple_list_item_1, items));    
    //给每个item设置监听事件
    listPopupWindow.setOnItemClickListener(new AdapterView.OnItemClickListener() {
    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
    Toast.makeText(SwipeRefreshActivity.this, "the position is" + position, Toast.LENGTH_SHORT).show();
    //listPopupWindow.dismiss();
    }
    });
    //设置ListPopupWindow的锚点,也就是弹出框的位置是相对当前参数View的位置来显示，
    listPopupWindow.setAnchorView(view);    
    //ListPopupWindow 距锚点的距离，也就是相对锚点View的位置
    listPopupWindow.setHorizontalOffset(100);
    listPopupWindow.setVerticalOffset(100);    
    //设置对话框的宽高
    listPopupWindow.setWidth(300);
    listPopupWindow.setHeight(600);
    listPopupWindow.setModal(false);
    listPopupWindow.show();
    }
```
## PopupMenu
```java
    public void showPopupMenu(View view) {
    //参数View 是设置当前菜单显示的相对于View组件位置，具体位置系统会处理
    PopupMenu popupMenu = new PopupMenu(this, view);
    //加载menu布局
    popupMenu.getMenuInflater().inflate(R.menu.menu_main, popupMenu.getMenu());
    //设置menu中的item点击事件
    popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
    @Override
    public boolean onMenuItemClick(MenuItem item) {
    return false;
    }
    });
    //设置popupWindow消失的点击事件
    popupMenu.setOnDismissListener(this);
    popupMenu.show();
    }
```
## 新风格Spinner
```java
   <Spinner
    android:id="@+id/spinner"
    style="@android:style/Widget.Holo.Light.Spinner"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"></Spinner>
```
## v4 SwipeRefreshLayout
- SwipeRefreshLayout继承自ViewGroup,理论上可以完成任何View的刷新

### 代码
```java
final SwipeRefreshLayout swipeRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.md_SwipeRefreshLayout);
//设置进度条的颜色
swipeRefreshLayout.setColorSchemeColors(Color.RED, Color.BLUE, Color.GREEN);
//设置圆形进度条大小
swipeRefreshLayout.setSize(SwipeRefreshLayout.LARGE);
//设置进度条背景颜色        swipeRefreshLayout.setProgressBackgroundColorSchemeColor(Color.WHITE);
//设置下拉多少距离之后开始刷新数据，不要设置的太大，不然怎么拉都不刷新
swipeRefreshLayout.setDistanceToTriggerSync(5);
```
