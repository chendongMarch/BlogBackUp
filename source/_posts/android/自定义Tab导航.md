---
layout: post
title: 自定义下方导航Tab
date: 2016-03-09
category: Android
tags: [Android,开源库,自定义控件]
keywords: Android
description: 自定义下方导航Tab
---


## 前言
- 基本大多数应用都会使用底部Tab的导航方式，实现底部Tab导航的方式很多，之前有TabHost,现在有TabLayout,之前一直用的是LinearLayout嵌套的方式或者RadioGroup,但是都相对麻烦，所以自定义了一个控件可以更方便的实现底部tab导航。


## GitHub地址
[GitHub源码地址](https://github.com/chendongMarch/TabHolderSample)


## Gradle
`compile 'com.march.tabholder:tabholder:1.0.2'`



![效果图](http://img.blog.csdn.net/20160329171547640)


## 在xml文件中使用
```xml
<com.march.tabholder.TabHolder
        android:id="@+id/activity_tab_test_tabholder"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:background="#ddd"
        //没有被选中时文字颜色
        app:TabHolderNormalColor="#567"
        //选中时文字颜色
        app:TabHolderSelectColor="#00f"
        //显示的类型，horizontal表示图片在文字左边，vertical表示图片在文字下边
        app:TabHolderType="horizontal"
        //是否显示文字
        app:TabHolderWithText="true"
        //是否显示分割线
        app:TabHolderWithDivider="true"
        />
```


## 初始化
```java
TabHolder mTabHolder = (TabHolder) findViewById(R.id.activity_tab_test_tabholder);

//添加tab
mTabHolder.addTab(R.drawable.camera_filter, R.drawable.camera_filter_press, "滤镜")
                .addTab(R.drawable.camera_record, R.drawable.camera_record_press, "记录")
                .addTab(R.drawable.camera_sticker, R.drawable.camera_sticker_pressed, "贴纸")
                .addTab(R.drawable.camera_sticker, R.drawable.camera_sticker_pressed, "贴纸");

//设置监听事件
mTabHolder.setOnItemSelectListener(new TabHolder.OnItemSelectListener() {
            @Override
            public void OnItemSelect(int preSelect, int currentSelect, TabView preView, TabView currentView) {
                Log.e("chendong", "上一个选择是  " + preSelect + "   当前选择是  " + currentSelect);
            }
});
```

## 与ViewPager联动
```java
//给TabHolder设置ViewPager,是否平滑移动,监听
mTabHolder.setViewPager(vp, false, new ViewPager.OnPageChangeListener() {
            @Override
            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

            }

            @Override
            public void onPageSelected(int position) {

            }

            @Override
            public void onPageScrollStateChanged(int state) {

            }
        });
        
```

## API
```java
//停止自动切换,在使用下面的方法时会造成与自动切换的冲突,建议停用
public void setAutoToogle(boolean autoToogle)
//选中一个
public void select(int pos)
//不选中一个
public void unselect(int pos)
//切换一个
public void toogle(int pos)

//选中一个,上一个被选中的会置为不选中
mTabHolder.singleSelect(0)
```
