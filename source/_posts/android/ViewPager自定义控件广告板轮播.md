---
layout: post
title: ViewPager自定义控件广告板轮播
category: Android
tags:
  - Android
  - 开源库
description: ViewPager自定义控件广告板轮播
abbrlink: 149273814
date: 2016-03-16 00:00:00
keywords:
---


## GitHub
- [GitHub地址](https://github.com/chendongMarch/BillBoard)

## Gradle
- `compile 'com.march.billboardview:billboardview:2.0.6-beta4'`



## xml 里面使用

```java
xml 里面使用
<com.march.billboardview.BillBoardView
        android:id="@+id/billboard"
        android:layout_width="match_parent"
        android:layout_height="250dp"
        board:isAutoRun="true"
        board:isLoopIt="true"
        board:intervalTime="2000" />

属性：
isAutoRun:是不是自动播放，不需要滑动，默认true
isLoopIt:是不是无限循环播放，默认是true
intervalTime:播放间隔时间，每隔多长时间走一页
```

## 构建实体
```java
//实体类实现获取url和title的接口
public class Demo implements BoardConfig{}
```


## 定义加载工具
```
//初始化图片加载的工具,你可以自定义使用Picasso还是Glide等图片加载库加载
BillBoard.init(new BillBoard.BillLoadImg() {
            @Override
            public void loadImg(Context context, String title, String url, ImageView imageView) {
                imageView.setImageResource(Integer.parseInt(url));
            }
        });
```


## 使用SimpleBoardAdapter
```java
private BillBoardView billBoardView;
private SimpleBoardAdapter<Demo> mBoardAdapter;
mBoardAdapter = new SimpleBoardAdapter<>(getActivity(), demos);
billBoardView.setAdapter(mBoardAdapter);
```


## 配置BillBoardView
```
billBoardView
      .setAdapter(mBoardAdapter)
      .setSwipeRefreshLayout(sw)
      .click(new OnBoardClickListener() {
              @Override
              public void clickBillBoard(int pos, BoardConfig b) {
                    Log.e("chendong", "click pos " + pos + "   title is " + b.getTitle());
              }
      }).show();
```


## 数据更新
```
mBoardAdapter.notifyDataSetChanged(demos);
```


## 开放停止和开始播放的方法
```
public void startPlay()
public void stopPlay()
```

## SwipeRefreshLayout冲突
- 当与SwipeRefreshLayout嵌套使用时，解决SwipeRefreshLayout冲突

```
public void setSwipeRefreshLayout(SwipeRefreshLayout sw)
```


## 轮播动画和时间
- 下面是可选的插值器,可自定义插值器

```java
//设置动画的方法
public BillBoardView setAnimation(int duration, Interpolator interpolator)
```

|插值器|描述|
|:--|:--|
|new AccelerateInterpolator()  |开始慢后面加速,由于距离较近效果不明显,有点像是匀速|
|new AccelerateDecelerateInterpolator()|  两头速度慢,中间加速,由于距离较近效果不明显,有点像是匀速|
|new DecelerateInterpolator() |开始快后面慢,由于距离较近效果不明显,有点像是匀速|
|new BounceInterpolator() |到达末尾跳跃弹起|
|new AnticipateInterpolator()|先甩一下在移动|
|new AnticipateOvershootInterpolator() |先甩一下到达终点后过界在后退|
|new OvershootInterpolator()| 过界后返回|
|new LinearInterpolator() |常量变速
|new LinearOutSlowInInterpolator() |开始快后面慢|




## 其他API
```java
//BillBoardView
//重新定义ViewPager的将停
public void setOnBoardPageChangeListener(OnPageChangeListener onPageChangeListener)
//获取内部的ViewPager
public ViewPager getViewPager()
```

## BoardAdapter
```java
//一些变量，你可以在子类中访问
protected Context mContext;
protected int mLyRes;
protected int preIndex = -1;//上一个被选中的
protected List<B> datas;
protected boolean isLoop;
protected View mRootView;
protected BillBoardView mBoardView;
```

## 基于SimpleBoardAdapter
```java
//为了方便使用定义了SimpleBoardAdapter

//获取TitleView用于改变字体，颜色，背景，文字大小等
public TextView getTitleView()
//获取底部Bar,用于改变背景，高度等
public ViewGroup getBotLy()
//获取导航条
public LinearLayout getGuideLy()
//设置选中和未选中的资源
public void setSelectRes(int selectRes, int unSelectRes)
//设置标题的位置POS_LEFT = 0, POS_CENTER = 1, POS_RIGHT = 2
public void setTitleGravity(int gravity)
//设置导航栏的位置POS_LEFT = 0, POS_CENTER = 1, POS_RIGHT = 2
public void setGuideLyGravity(int gravity)
```

## 如何自定义Adapter
```java
//Adapter做的工作是，覆盖在BillBoardView上面，随着BillBoardView的变化，修改UI
public class MyAdapter extends BoardAdapter<Demo> {

    public MyAdapter(Context mContext, List<Demo> datas) {
        super(mContext, datas);
    }

    //资源ID.高度建议Match_parent
    @Override
    protected int getLayoutId() {
        return 0;
    }

    //获取控件，findById()或者一些初始化的操作
    @Override
    protected void initAdapterViews() {

    }

    //当划到pos位置。更改界面显示
    @Override
    public void changeItemDisplay(int pos, Demo demo) {

    }

    //当adapter连接到billboardView时，触发
    @Override
    public void onBillBoardViewAttached(BillBoardView billBoardView) {
        super.onBillBoardViewAttached(billBoardView);
    }
}

```

## 优化
- 当你的页面退出时,暂定轮播将是优化的一个很好选择

```java
    @Override
    protected void onResume() {
        super.onResume();
        if(billBoardView!=null)
            billBoardView.startPlay();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        billBoardView.stopPlay();
    }

    @Override
    protected void onPause() {
        super.onPause();
        billBoardView.stopPlay();
    }
```

