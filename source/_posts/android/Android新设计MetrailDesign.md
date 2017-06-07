---
layout: post
title: Android设计MetrailDesign
date: 2015-03-09
category: Android
tags: [Android,Android5.+]
keywords: 
description: Android设计MetrailDesign
---

## Theme
```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
<!--导航栏底色-->
<item name="colorPrimary">@color/accent_material_dark</item>
<!--状态栏底色-->
<item name="colorPrimaryDark">@color/accent_material_light</item>
<!--导航栏上的标题颜色-->
<item name="android:textColorPrimary">@android:color/black</item>
<!--Activity窗口的颜色-->
<item name="android:windowBackground">@color/white</item>
<!--按钮选中或者点击获得焦点后的颜色-->
<item name="colorAccent">@color/accent_material_light</item>
<!--和 colorAccent相反，正常状态下按钮的颜色,所有控价在没有获得焦点时的颜色-->
<item name="colorControlNormal">#ff0000</item>
<!--Button按钮正常状态颜色-->
<item name="colorButtonNormal">@color/accent_material_light</item>
<!--EditText 输入框中字体的颜色-->
<item name="editTextColor">@android:color/white</item>
</style>
```

##TextInputLayout
### XML文件
```xml
<android.support.design.widget.TextInputLayout
    android:id="@+id/md_textinputlayout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    >
    <EditText
    android:id="@+id/md_edittext"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="请输入文本"
    />
    </android.support.design.widget.TextInputLayout>

```
### 代码显示错误信息,文本颜色的需要使用Theme修改
```java
TextInputLayout tily1=(TextInputLayout) findViewById(R.id.tily1);
tily1.setHint("新的Hint");
tily1.getEditText().addTextChangedListener(new TextWatcher()); 
//在监听之中进行如下设置可以显示错误信息
int num = 0;
if (((num = s.toString().length()) < 6) && num != 0) {
    textInputLayout.setErrorEnabled(true);
    textInputLayout.setError("长度不能小于6");
} else {
    textInputLayout.setErrorEnabled(false);
}
```

## SnackBar
## 代码
```java
Snackbar snackbar = Snackbar.make(view, "SnackBar测试", Snackbar.LENGTH_SHORT)
				//点击事件中按钮的颜色
                .setActionTextColor(Color.WHITE)
                //设置点击事件
                .setAction("SnackBar", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Log.i("chendong", "点击了action");
                    }
                });
//修改背景颜色
snackbar.getView().setBackgroundColor(Color.GRAY);
snackbar.show();
}
```
## FloatingActionButton
```xml
<android.support.design.widget.FloatingActionButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@mipmap/ic_launcher"
            android:layout_alignParentRight="true"
            android:layout_alignParentBottom="true"
            android:layout_margin="10dp"
            android:onClick="ClickFloatButton"
            app:fabSize="normal"
            app:backgroundTint="#f00"
            app:elevation="5dp"
            />
```
### 属性
- //上面的文件使用相对布局，可以使用相对锚点的设置定位fab
- app:layout_anchor="@id/md_recyclerview"//锚点，基于哪个控件定位
- app:layout_anchorGravity="bottom|right|end"//相对锚点的位置
- app:fabSize="normal"//控件大小，只支持两种大小，mini,normal
- app:backgroundTint="#f00"//改变背景颜色，不设置好像是黑的还是跟随主题
- app:elevation="5dp"//阴影
```

## AppBarLayout 
- AppBarLayout本身是一个垂直的LinearLayout，被他包裹的控件将作为ActionBar展示


```xml
<android.support.design.widget.AppBarLayout       android:layout_width="match_parent"            android:layout_height="wrap_content"
>
<android.support.v7.widget.Toolbar
      android:id="@+id/md_toolbar"
      android:layout_width="match_parent"
      android:layout_height="?attr/actionBarSize"
      android:minHeight="?attr/actionBarSize"
      android:background="?attr/colorPrimary"
      android:layout_marginBottom="20dp">
</android.support.v7.widget.Toolbar>
<TextView
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:padding="10dp"
      android:text="这是appbarlayout测试"
      android:gravity="center_horizontal"
      android:textSize="20sp"
/>
</android.support.design.widget.AppBarLayout>
```
## CoordinatorLayout
- 这个（Coordinator）单词的意思是协调器，它是协调控件之间动画效果的一个布局

```xml
    <android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.awesome.newfeatures.SecondActivity"
    >
    <android.support.design.widget.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    >
    <android.support.v7.widget.Toolbar
    android:id="@+id/md_toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:minHeight="50dp"
    android:background="?attr/colorPrimary"
    android:layout_marginBottom="20dp"   app:layout_scrollFlags="scroll|enterAlways"
    />
    <android.support.design.widget.TabLayout
    android:id="@+id/md_tablayout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:tabGravity="fill"
    app:tabTextColor="#f00"
    app:tabIndicatorColor="#00f"
    app:tabMode="fixed"
    app:tabIndicatorHeight="2dp"
    />
    </android.support.design.widget.AppBarLayout>
    <android.support.v7.widget.RecyclerView
    android:id="@+id/md_recyclerview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"     app:layout_behavior="@string/appbar_scrolling_view_behavior"
    />
    <android.support.design.widget.FloatingActionButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@mipmap/ic_launcher"
    app:backgroundTint="#00f"
    app:elevation="5dp"
    android:layout_margin="16dp"
    app:layout_anchor="@id/md_recyclerview"
    app:layout_anchorGravity="bottom|right|end"
    android:onClick="Click"
    />
    </android.support.design.widget.CoordinatorLayout>
```

1. app:layout_behavior="@string/appbar_scrolling_view_behavior"//在视图中可滑动的组件使用该属性标示，必须是RecyclerView或NestedScrollView

2. 试验表明，开始使用后面两个属性并没起作用，后来发现需要固定toolbar高度才会起作用。

3. FloatingActionButton被包裹在CoordinatorLayout中时可以防止SnackBar跳出时遮挡Fab

4. app:layout_scrollFlags="scroll|enterAlways"//三种取值

| 属性选择   |     描述 | 
| :-------- | :--------| 
| `scroll`   |  所有需要滑动的视图需要定义该属性，不使用该属性的视图将会固定在屏幕顶端this flag should be set for all views that want to scroll off the screen - for views that do not use this flag, they’ll remain pinned to the top of the screen| 
|`enterAlways`  |  任意向下的操作会导致隐藏视图显示出来this flag ensures that any downward scroll will cause this view to become visible, enabling the ‘quick return’ pattern| 
|`enterAlwaysCollapsed`  |  这个flag定义的是何时进入（已经消失之后何时再次显示）。假设你定义了一个最小高度（minHeight）同时enterAlways也定义了，那么view将在到达这个最小高度的时候开始显示，并且从这个时候开始慢慢展开，当滚动到顶部的时候展开完。When your view has declared a minHeight and you use this flag, your View will only enter at its minimum height (i.e., ‘collapsed’), only re-expanding to its full height when the scrolling view has reached it’s top.| 
|`exitUntilCollapsed`| 这个flag时定义何时退出，当你定义了一个minHeight，这个view将在滚动到达这个最小高度的时候消失。 this flag causes the view to scroll off until it is ‘collapsed’ (its minHeight) before exiting| 


## CollapsingToolbarLayout
- 使用CollapsingToolbarLayout结合CoordinatorLayout实现可缩放的ActionBar,在使用CoordinatorLayout一直实现不了列表滑动到顶端才显示的效果，使用CollapsingToolbarLayout可以实现


```xml
    <android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.awesome.newfeatures.SecondActivity"
    >   
    //需要固定高度
    <android.support.design.widget.AppBarLayout
    android:id="@+id/appbar"
    android:layout_width="match_parent"
    android:layout_height="160dp"
    >
    <android.support.design.widget.CollapsingToolbarLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:contentScrim="?attr/colorPrimary"
    app:layout_scrollFlags="scroll|exitUntilCollapsed"
    >
    <ImageView
    android:id="@+id/backdrop"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scaleType="centerCrop"
    android:src="@mipmap/ic_launcher"
    app:layout_collapseMode="parallax"
    />
    <android.support.v7.widget.Toolbar
    android:id="@+id/md_toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    app:layout_collapseMode="pin"
    />
    </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>    
    <android.support.v7.widget.RecyclerView
    android:id="@+id/md_recyclerview"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    />    
    <android.support.design.widget.FloatingActionButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@mipmap/ic_launcher"
    app:backgroundTint="#00f"
    app:elevation="5dp"
    android:layout_margin="16dp"
    app:layout_anchor="@id/md_recyclerview"
    app:layout_anchorGravity="bottom|right|end"
    android:onClick="Click"
    />
    </android.support.design.widget.CoordinatorLayout> 
```

1. AppBarLayout高度需要固定
2. CollapsingToolbarLayout设置app:layout_scrollFlags

```
 - 23.1.0以后新增一个snap可以在拉动到底部时产生缩放效果
 - app:layout_scrollFlags="scroll|enterAlways"
 - app:layout_scrollFlags="scroll|enterAlwaysCollapsed"
 - 不论ToolBar是否设置pin模式，将会全部滚出屏幕，任意下拉动画会使隐藏的视图显示出来
 - app:layout_scrollFlags="scroll|exitUntilCollapsed"
 - //设置为pin模式的固定在顶端，下拉到顶部才显示出隐藏视图
 - app:layout_collapseMode="parallax"//需要固定的视图使用pin模式，需要滑动的视图使用parallax模式
```


### 一些属性
`setTitle(CharSequence)`//设置固定在顶端时显示的title
`setContentScrim(Drawable) `
`app:contentScrim=”?attr/colorPrimary”`//修改固定在顶端的背景颜色
`setStatusBarScrim(Drawable)`//状态栏背景，5.0以上有效
`app:layout_collapseParallaxMultiplier=”0.6”`//滑动的视觉差，产生的效果是提前将折叠的视图隐藏掉了
`app:layout_collapseMode=”parallax|pin”`//滑动模式，缩放或者固定
`collapsingToolbarLayout.setTitle("title");`//设置标题，将会自动进行缩放
//设置颜色后将会自动进行颜色过渡
`collapsingToolbarLayout.setExpandedTitleColor(Color.WHITE);`
`collapsingToolbarLayout.setCollapsedTitleTextColor(Color.GREEN);`

##  DrawerLayout+NavigationView
- 结合之前学的总结在一个布局中，首先DrawerLayout需要有两个子view，上面的一个代表content,下面的NavigationView代表菜单导航，当然你可以替换成自己的布局，用来做其他的事情。

```xml
    <android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/md_DrawerLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context=".MainActivity"
    >
    <android.support.design.widget.CoordinatorLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    >
    <android.support.design.widget.AppBarLayout
    android:id="@+id/appbar"
    android:layout_width="match_parent"
    android:layout_height="160dp"
    >
    <android.support.design.widget.CollapsingToolbarLayout
    android:id="@+id/md_CollapsingToolbarLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:contentScrim="?attr/colorPrimary"
    app:layout_scrollFlags="scroll|exitUntilCollapsed"
    >
    <ImageView
    android:id="@+id/backdrop"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scaleType="centerCrop"
    android:src="@mipmap/ic_launcher"
    app:layout_collapseMode="parallax"
    app:layout_collapseParallaxMultiplier="0.3"
    />
    <android.support.v7.widget.Toolbar
    android:id="@+id/md_toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    app:layout_collapseMode="pin"
    />
    </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>
    <android.support.v7.widget.RecyclerView
    android:id="@+id/md_recyclerview"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
app:layout_behavior="@string/appbar_scrolling_view_behavior"
    />
    <android.support.design.widget.FloatingActionButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@mipmap/ic_launcher"
    app:backgroundTint="#00f"
    app:elevation="5dp"
    android:layout_margin="16dp"
    app:layout_anchor="@id/md_recyclerview"
    app:layout_anchorGravity="bottom|right|end"
    android:onClick="Click"
    />
    </android.support.design.widget.CoordinatorLayout> 
    <android.support.design.widget.NavigationView
    android:id="@+id/md_NavigationView"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:layout_gravity="start"
    app:headerLayout="@layout/header"
    app:menu="@menu/drawer"
    />
    </android.support.v4.widget.DrawerLayout>
```

### 几个属性
```
关于NavigationView中item的字体颜色和icon选中状态颜色是去当前主题theme中的
<--正常状态下字体颜色和icon颜色-->
<item name="android:textColorPrimary">@android:color/darker_gray</item>
<--选中状态icon的颜色和字体颜色-->
<item name="colorPrimary">@color/accent_material_light</item>
当然你可以通过如下方法或者属性来改变这一状态：
setItemBackgroundResource(int)
app:itemBackground`//给menu设置背景资源
setItemIconTintList(ColorStateList)
app:itemIconTint`//给menu的icon设置颜色，对应的属性
setItemTextColor(ColorStateList)
app:itemTextColor`//给menu的item设置字体颜色，对应的属性
app:headerLayout="@layout/header"//NavigationView的头部布局
app:menu="@menu/drawer"`//NavigationView的菜单文件
```
### 代码
```java
mDrawerLayout = (DrawerLayout) findViewById(R.id.md_DrawerLayout);
//这个组件将会展示一个图标在左上角，它是一个DrawerListener的子类，使用它可以简单地创建一个抽屉的控制图标，v4包下的该组件可以设置导航图标，但是已经过时，这里使用v7包下的组件，但是没有了设置图标的参数，如果想自定义可以使用自定义的图标，在实现DrawerListener时做处理
ActionBarDrawerToggle mDrawerToggle = new ActionBarDrawerToggle(this,  mDrawerLayout, toolbar, R.string.close, R.string.open);
//没有找到合适的方法修改图标，这个方法没有效果
    mDrawerToggle.setHomeAsUpIndicator(R.mipmap.ic_launcher);
    //初始化状态
    mDrawerToggle.syncState();
    mDrawerLayout.setDrawerListener(mDrawerToggle);
    //设置导航栏NavigationView的点击事件
    final NavigationView mNavigationView = (NavigationView) findViewById(R.id.md_NavigationView);
    mNavigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
    @Override
    public boolean onNavigationItemSelected(MenuItem menuItem) {
    //使用该方法可以选中Item，但是出现了一个问题，点击时有时会造成选中无法取消，试过一个解决办法是获得Menu的所有Item设置不选中可以解决这个问题。
    menuItem.setChecked(true);
    mDrawerLayout.closeDrawers();//关闭抽屉
    return true;
    }
    });
```