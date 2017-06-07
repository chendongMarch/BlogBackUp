---
layout: post
title: 快速实现Fragment切换功能
date: 2016-11-05
category: Android
tags: [Android]
keywords: 
---

## 1. 前言 
>一个app首页通常是使用`Activity + Fragment`的形式展示，控制Fragment的显示和隐藏基本有两种实现方法:   
&emsp;&emsp;1. ViewPager , 比如微信 , 优势是手势操作更加方便，官方提供了FragmentPagerAdapter可以很方便帮助我们实现数据加载（Fragment要使用懒加载的方式，避免浪费资源），劣势就是当你的第一个Fragment中已经使用了ViewPager，两层套一起事件会冲突，而且操作也不友好啦。  
&emsp;&emsp;2. FragmentManager , 比如头条，针对使用ViewPager组合Fragment的问题，使用FragmentManager控制Fragment的显示和隐藏，不需要考虑懒加载的问题，不过不能支持滑动啦。  
&emsp;&emsp;3. 本文主要是封装FragmentManager切换页面的相关操作。

<!--more-->

## 2. 最初的方案
刚开始的方案是定义了一个Activity的基类MultiFragmentActivity，在基类中实现切换Fragment的逻辑，想要实现该功能的Activity继承基类就可以实现切换功能。[MultiFragmentActivity.java源代码](https://github.com/chendongMarch/CommonLib/blob/master/lib-core/src/main/java/com/march/lib/core/activity/MultiFragmentActivity.java)，这种实现方式使用起来要更加简单，但是有个问题，整个项目通常会实现同一个基类来完成类似或者相同的功能，当然我们可以将MultiFragmentActivity继承自项目的基类，然后再继承MultiFragmentActivity，但是这样就不能把这个功能提取出来。

少用继承，多用组合，继承造成的问题就是耦合性太高，不方便分离功能，所以决定使用组合的方法重新设计，定义帮助类FragmentHelper，并且使用一个接口与需要进行Fragment切换的Activity进行交流，Activity可以持有FragmentHelper来完成Fragment的切换，具体的切换逻辑由FragmentHelper来处理。



## 3. 定义接口
首先定义FragmentOperator接口，借助该接口需要进行Fragment切换的Activity可以自定义一些操作

```java
interface FragmentOperator {
    /**
     * 获取放置fragment的控件id
     *
     * @return id
     */
    int getFragmentContainerId();
    /**
     * 构建fragment
     *
     * @param showItem 将要展示的fragment pos
     * @return fragment
     */
    Fragment makeFragment(int showItem);
    /**
     * 进行转换之前做操作,实现该方法可以自定义fragment切换的动画效果等
     *
     * @param transaction transaction
     */
    void beginTransaction(FragmentTransaction transaction);
    /**
     * 切换成功之后会回调这个方法，同步选中之后的显示状态
     * 比如改变按钮的颜色等操作
     *
     * @param selectImage 被选中的item
     */
    void syncSelectState(int showItem);
    /**
     * 当点击显示同一个，当两次切换到了同一个fragment，返回true时会进行此次切换
     *
     * @param showItem 显示的item
     * @return 返回false表示忽略此次点击的切换
     */
    boolean whenShowSameFragment(int showItem);
    /**
     * 当点击显示的不是同一个，用于需要针对切换到某个fragment时的场景
     *
     * @param showItem 显示的item
     * @return 返回false表示忽略此次点击的切换
     */
    boolean whenShowNotSameFragment(int showItem);
}


// 上面的接口中不是所有的方法都是必须的，下面是一个简单的实现
public static abstract class SimpleFragmentOperator implements FragmentOper
    @Override
    public boolean whenShowNotSameFragment(int showItem) {
        return true;
    }
    @Override
    public boolean whenShowSameFragment(int showItem) {
        return false;
    }
    @Override
    public void syncSelectState(int showItem) {
    }
    @Override
    public void beginTransaction(FragmentTransaction transaction) {
    }
}
```



## 4. 核心方法

使用tag作为标记添加fragment,避免重复创建

```java
private static final String FRAGMENT_ATG = "FragmentHelper";
private static final String ITEM_HIDE    = "mHideItem";
private static final String ITEM_SHOW    = "mShowItem";
private FragmentOperator operator;
private Fragment         mCurrentFragment;
private FragmentManager  mFragmentManager;
private int              mShowItem, mHideItem;
private int mExactlyItem = 0;

    
/**
 * 隐藏当前显示的fragment,显示将要显示的fragment
 *
 * @param hideItem   需要隐藏的fragment
 * @param showItem   需要显示的fragment
 * @param isOnCreate 是否是第一次从OnCreate中启动,点击都是false
 */
private void performSelectItem(int hideItem, int showItem, boolean isOnCreate) {
    // 获得将要显示页的tag
    String currentTag = getFragmentTag(hideItem);
    // 隐藏当前的的fragment
    FragmentTransaction transaction = mFragmentManager.beginTransaction();
    operator.beginTransaction(transaction);
    // 第一次创建，一个都没有，不需要隐藏，直接显示
    if (mFragmentManager.getFragments() == null) {
        mShowItem = showItem;
        mExactlyItem = showItem;
        mCurrentFragment = operator.makeFragment(showItem);
        transaction.add(operator.getFragmentContainerId(), mCurrentFragment, getFragmentTag(
                .show(mCurrentFragment);
    } else {
        // 优化，如果被杀后再进来，全部的fragment都会被呈现显示状态，所以都隐藏一遍
        if (isOnCreate && mFragmentManager.getFragments() != null) {
            for (Fragment fragment : mFragmentManager.getFragments()) {
                transaction.hide(fragment);
            }
        } else {
            // 正常按钮点击进入，隐藏上一个即可
            Fragment lastFragment = mFragmentManager.findFragmentByTag(currentTag);
            if (lastFragment != null) {
                transaction.hide(lastFragment);
            }
        }
        // 获得将要显示页的tag
        String toTag = getFragmentTag(showItem);
        // find要显示的Fragment
        mCurrentFragment = mFragmentManager.findFragmentByTag(toTag);
        if (mCurrentFragment != null) {
            // 已经存在则显示
            transaction.show(mCurrentFragment);
        } else {
            // 不存在则添加新的fragment
            mCurrentFragment = operator.makeFragment(showItem);
            if (mCurrentFragment != null) {
                transaction.add(operator.getFragmentContainerId(), mCurrentFragment, toTag);
            }
        }
    }
    // 同步状态
    operator.syncSelectState(showItem);
    // 保存当前显示fragment的item
    mHideItem = hideItem;
    mShowItem = showItem;
    transaction.commitAllowingStateLoss();
}
```




## 5. 在Activty显示某个Fragment
在Activity中使用时只需要调用showFragment方法即可

```java
/**
 * 显示某个fragment
 *
 * @param showItem 显示的item
 */
public void showFragment(int showItem) {
    if (showItem == mShowItem) {
        if (operator.whenShowSameFragment(showItem)) {
            performSelectItem(mExactlyItem, showItem, false);
            mExactlyItem = showItem;
        }
    } else {
        if (operator.whenShowNotSameFragment(showItem)) {
            performSelectItem(mExactlyItem, showItem, false);
            mExactlyItem = showItem;
        }
    }
}
```




## 6. 优化
当Activity被回收时，记录上次的状态

```java
// 保存数据
// FragmentHelper
public void onCreate(Bundle save) {
    if (save != null) {
        mHideItem = save.getInt(ITEM_HIDE, 0);
        mShowItem = save.getInt(ITEM_SHOW, 0);
        performSelectItem(mHideItem, mShowItem, true);
    }
}
// 在Activity销毁时调用
@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    if (mFragmentHelper != null) {
        mFragmentHelper.onSaveInstanceState(outState);
    }
}



// 重新加载
// FragmentHelper
public void onCreate(Bundle save) {
    if (save != null) {
        mHideItem = save.getInt(ITEM_HIDE, 0);
        mShowItem = save.getInt(ITEM_SHOW, 0);
        performSelectItem(mHideItem, mShowItem, true);
    }
}
// 在Activity创建时调用
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mFragmentHelper = new FragmentHelper(getSupportFragmentManager(), mSimpleOperator);
    mFragmentHelper.onCreate(savedInstanceState);
}
```




## 7. 使用
```java
public class TestMultiFragmentActivity2 extends BaseActivity {

    private FragmentHelper mFragmentHelper;
    private FragmentHelper.SimpleFragmentOperator mSimpleOperator;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mSimpleOperator = new FragmentHelper.SimpleFragmentOperator() {
            @Override
            public int getFragmentContainerId() {
                return 0;
            }

            @Override
            public Fragment makeFragment(int showItem) {
                return null;
            }

            @Override
            public void syncSelectState(int showItem) {

            }
        };
        mFragmentHelper = new FragmentHelper(getSupportFragmentManager(), mSimpleOperator);
        // 只有销毁后回来才会调起切换操作否则没反应
        mFragmentHelper.onCreate(savedInstanceState);
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        if (mFragmentHelper != null) {
            mFragmentHelper.onSaveInstanceState(outState);
        }
    }
    
    @Override
    protected int getLayoutId() {
        return 0;
    }
}
```




## 8.  源代码
```java

/**
 * CreateAt : 2016/11/5
 * Describe : 实现Fragment切换
 *
 * @author chendong
 */
public class FragmentHelper {

    private static final String FRAGMENT_ATG = "FragmentHelper";
    private static final String ITEM_HIDE    = "mHideItem";
    private static final String ITEM_SHOW    = "mShowItem";

    private FragmentOperator operator;
    private Fragment         mCurrentFragment;
    private FragmentManager  mFragmentManager;
    private int              mShowItem, mHideItem;
    private int mExactlyItem = 0;


    public void onCreate(Bundle save) {
        if (save != null) {
            mHideItem = save.getInt(ITEM_HIDE, 0);
            mShowItem = save.getInt(ITEM_SHOW, 0);
            performSelectItem(mHideItem, mShowItem, true);
        }
    }


    public void onSaveInstanceState(Bundle outState) {
        outState.putInt(ITEM_HIDE, mHideItem);
        outState.putInt(ITEM_SHOW, mShowItem);
    }


    public FragmentHelper(FragmentManager mFragmentManager, FragmentOperator operator) {
        this.mFragmentManager = mFragmentManager;
        this.mFragmentManager = mFragmentManager;
        this.operator = operator;
    }


    /**
     * 显示某个fragment
     *
     * @param showItem 显示的item
     */
    public void showFragment(int showItem) {
        if (showItem == mShowItem) {
            if (operator.whenShowSameFragment(showItem)) {
                performSelectItem(mExactlyItem, showItem, false);
                mExactlyItem = showItem;
            }
        } else {
            if (operator.whenShowNotSameFragment(showItem)) {
                performSelectItem(mExactlyItem, showItem, false);
                mExactlyItem = showItem;
            }
        }
    }


    /**
     * 获取当前处于活动状态的fragment'
     *
     * @return fragment
     */
    public Fragment getCurrentFragment() {
        return mCurrentFragment;
    }

    /**
     * 隐藏当前显示的fragment,显示将要显示的fragment
     *
     * @param hideItem   需要隐藏的fragment
     * @param showItem   需要显示的fragment
     * @param isOnCreate 是否是第一次从OnCreate中启动,点击都是false
     */
    private void performSelectItem(int hideItem, int showItem, boolean isOnCreate) {
        // 获得将要显示页的tag
        String currentTag = getFragmentTag(hideItem);
        // 隐藏当前的的fragment
        FragmentTransaction transaction = mFragmentManager.beginTransaction();
        operator.beginTransaction(transaction);

        // 第一次创建，一个都没有，不需要隐藏，直接显示
        if (mFragmentManager.getFragments() == null) {
            mShowItem = showItem;
            mExactlyItem = showItem;
            mCurrentFragment = operator.makeFragment(showItem);
            transaction.add(operator.getFragmentContainerId(), mCurrentFragment, getFragmentTag(showItem))
                    .show(mCurrentFragment);
        } else {
            // 优化，如果被杀后再进来，全部的fragment都会被呈现显示状态，所以都隐藏一遍
            if (isOnCreate && mFragmentManager.getFragments() != null) {
                for (Fragment fragment : mFragmentManager.getFragments()) {
                    transaction.hide(fragment);
                }
            } else {
                // 正常按钮点击进入，隐藏上一个即可
                Fragment lastFragment = mFragmentManager.findFragmentByTag(currentTag);
                if (lastFragment != null) {
                    transaction.hide(lastFragment);
                }
            }

            // 获得将要显示页的tag
            String toTag = getFragmentTag(showItem);
            // find要显示的Fragment
            mCurrentFragment = mFragmentManager.findFragmentByTag(toTag);
            if (mCurrentFragment != null) {
                // 已经存在则显示
                transaction.show(mCurrentFragment);
            } else {
                // 不存在则添加新的fragment
                mCurrentFragment = operator.makeFragment(showItem);
                if (mCurrentFragment != null) {
                    transaction.add(operator.getFragmentContainerId(), mCurrentFragment, toTag);
                }
            }
        }
        // 同步状态
        operator.syncSelectState(showItem);
        // 保存当前显示fragment的item
        mHideItem = hideItem;
        mShowItem = showItem;
        transaction.commitAllowingStateLoss();
    }


    private String getFragmentTag(int item) {
        return FRAGMENT_ATG + item;
    }


    interface FragmentOperator {
        /**
         * 获取放置fragment的控件id
         *
         * @return id
         */
        int getFragmentContainerId();

        /**
         * 构建fragment
         *
         * @param showItem 将要展示的fragment pos
         * @return fragment
         */
        Fragment makeFragment(int showItem);

        /**
         * 进行转换之前做操作,实现该方法可以自定义fragment切换的动画效果等
         *
         * @param transaction transaction
         */
        void beginTransaction(FragmentTransaction transaction);

        /**
         * 切换成功之后会回调这个方法，同步选中之后的显示状态
         * 比如改变按钮的颜色等操作
         *
         * @param showItem 被选中的item
         */
        void syncSelectState(int showItem);

        /**
         * 当点击显示同一个，当两次切换到了同一个fragment，返回true时会进行此次切换
         *
         * @param showItem 显示的item
         * @return 返回false表示忽略此次点击的切换
         */
        boolean whenShowSameFragment(int showItem);

        /**
         * 当点击显示的不是同一个，用于需要针对切换到某个fragment时的场景
         *
         * @param showItem 显示的item
         * @return 返回false表示忽略此次点击的切换
         */
        boolean whenShowNotSameFragment(int showItem);
    }

    // 上面的接口中不是所有的方法都是必须的，下面是一个简单的实现
    public static abstract class SimpleFragmentOperator implements FragmentOperator {
        @Override
        public boolean whenShowNotSameFragment(int showItem) {
            return true;
        }

        @Override
        public boolean whenShowSameFragment(int showItem) {
            return false;
        }

        @Override
        public void syncSelectState(int showItem) {

        }

        @Override
        public void beginTransaction(FragmentTransaction transaction) {

        }
    }
}

```