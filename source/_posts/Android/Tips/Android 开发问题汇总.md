---
layout: post
title: Android 开发问题汇总
category: Android
tags:
  - Android
abbrlink: 39cb08bb
keywords:
  - Android
location: 杭州尚妆
date: 2018-02-02 10:50:00
photos: http://olx4t2q6z.bkt.clouddn.com/18-2-22/81734436.jpg
---

本文主要记录 `Android` 开发过程中遇到的比较 **神（cao）奇（dan）**的问题， 一些简单的问题却难以定位，查找问题时耽误很多时间，开发过程中应该从开始就规避这类问题的发生。


<!--more-->

## Bundle 传递问题

为了保证从外面传递过来的数据能够被完整的传递下去，通常我们会直接将原先的 `bundle` 直接传递下去，例如：

```java
public class DetailPagerAdapter extends FragmentPagerAdapter {

    Bundle bundle;

    public DetailPagerAdapter(FragmentManager fm, Bundle bundle) {
        super(fm);
        this.bundle = bundle;
    }

    @Override
    public Fragment getItem(int position) {
        BaseFragment fragment = null;
        switch (position) {
            case 0:
                fragment = new DetailIndexFragment();
                break;
            case 1:
                fragment = new DetailGraphicFragment();
                bundle.putInt("type", 1);
                break;
        }
        if (null != fragment) {
            fragment.setArguments(bundle);
        }
        return fragment;
    }
}
```

这样会造成的问题是多个对象同时持有了 `bundle` 的引用，如上面例子中 <span class="spec"> `adapter` 的宿主，`DetailIndexFragment` 和 `DetailGraphicFragment` </span> 都会持有 `bundle` 的引用，这会让数据变的难以管理，`bundle` 里面的数据会莫名其妙的发生改变，而我们难以定位是谁改变了他，好的做法是，创建一个全新的 `bundle` 并持有当前状态下的数据。

```java
public class DetailPagerAdapter extends FragmentPagerAdapter {
   
    Bundle mTransBundle;
   
    public DetailPagerAdapter(FragmentManager fm, Bundle bundle) {
        super(fm);
        mTransBundle = bundle;
    }
  
    @Override
    public Fragment getItem(int position) {
        BaseFragment fragment = null;
        fragmentBundle = new Bundle(mTransBundle);
        switch (position) {
            case 0:
                fragment = new DetailIndexFragment();
                break;
            case 1:
                fragment = new DetailGraphicFragment();
                fragmentBundle.putInt("type", 1);
                break;
        }
        fragment.setArguments(fragmentBundle);
        return fragment;
    }
}
```


## getter 方法中创建对象

案例：在 `Activity` 的 `onResume()` 方法中调用了 `getViewModel()` 获取到 `ViewModel` 进行一些操作，但是没想到 `getter` 方法中没做判断每次都会返回新的对象，造成了大量的内存占用。

在 `getter` 方法中不应该进行创建对象的操作，如果有要加入仅创建一次的判断，因为当别人和自己在使用该方法可能会直接调用，因为对使用者来说，这只是一个获取操作，就会频繁的使用它，并不知道内部返回了一个全新的对象。


```java
private ViewModel mViewModel;

// 不应该在 get 方法中不加判断的创建对象
public ViewModel getViewModel() {
    return new ViewModel();
}

// 如果仅执行创建新对象的操作应该命名为 newXXX()
public ViewModel newViewModel() {
    return new ViewModel();
}

// 如果一定要使用 get() 方法，需要增加创建一次的判读
public ViewModel getViewModel() {
   if(mViewModel == null){
       mViewModel = new ViewModel();
   }
   return mViewModel;
}
```
 









