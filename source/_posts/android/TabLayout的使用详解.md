---
layout: post
title: TabLayout的使用详解
date: 2015-06-23
category: Android
tags: [Android,Android5.+]
keywords:  
---
 
## 1. 前言
> TabLayout是用来实现tab导航的控件，放在`android.support.design.widget`包里面，使用它可以很简单有很完美的结合`ViewPager`或者`FragmentManager`实现tab导航，并且实现与ViewPager的联动等功能。 本文主要介绍了：   

> 1. TabLayout的基本特性。  
> 2. 如何与ViewPager联动。  
> 3. 如何自定义TabLayout的显示。  


<!--more-->

---
## 2. 基本使用和常见属性

```xml
<android.support.design.widget.TabLayout
 android:id="@+id/fragment_discover_tably"
 android:layout_width="match_parent"
 android:layout_height="wrap_content"/>
	
1. tabMode
app:tabMode="fixed|scrollable"	
tabMode属性有两个取值:
fixed时，所有的tab会居中显示，是不可滑动的，
scrollable时，所有的tab会靠左显示，当tab数量很多时，就会呈现滑动的效果，这对适配小屏幕手机至关重要。


2. 导航条的高度和颜色
app:tabIndicatorHeight="3dp"
app:tabIndicatorColor="#af00"


3. tab的宽度限制
当文字很少时仍旧不会小于tabMinWidth
当文字过多时，也不会将tab撑的超过tabMaxWidth
app:tabMaxWidth="150dp"
app:tabMinWidth="60dp"


4. tab被选中或没有被选中时的颜色
app:tabTextColor="@color/black"
app:tabSelectedTextColor="@color/colorPrimary"


5. 更改Tab上面的TextView的详细显示
app:tabTextAppearance="@style/TabLayoutStyle"
<style name="TabLayoutStyle" parent="TextAppearance.Design.Tab">
        <item name="android:textSize">16sp</item>
        <item name="android:textStyle">normal</item>
</style>


6. 背景
app:tabBackground="@color/transparent"
设置tab的背景颜色，默认TabLayout点击时是有光晕效果的，使用该属性可以去掉光晕效果。


7. tab间隔
默认tab左右两边是有padding的，使用以下属性可以去掉padding，只使用tabPadding是没有效果的，要具体到设置到哪个padding  
app:tabPadding="0dp"
app:tabPaddingBottom="0dp"
app:tabPaddingEnd="0dp"
app:tabPaddingStart="0dp"
app:tabPaddingTop="0dp"
```

  
## 3. 与ViewPager联动
使用方法`tabLy.setupWithViewPager(viewPager);`，就可以了，但是需要注意的是写ViewPager时需要实现getTitle()方法，用来设置tab的标题。原先给Tab设置的标题将会被覆盖掉。




## 4. 自定义Tab显示
默认的`TabLayout`是可以显示文字和Icon的，定制度不是很高，有时候很难达到想要的效果，不过好在`TabLayout`开放了`setCustomView(childView)`这个API来支持我们自定义`TabLayout`的UI，当然自定义显示之后也损失了切换时颜色切换等效果，需要我们自己来处理，这边我模仿adapter的方式写了一个辅助类，用来更简单的实现自定义Tab显示时的显示和状态切换，源码很简单，主要是使用模板方法的方式，由 **子类决定** 并且 **只关注** 如何加载数据和更改状态显示的操作。

```java
/**
 * Describe : TabLayout适配器,快速实现TabLayout自定义View数据加载
 * @author chendong
 */
public abstract class BaseTabLayoutAdapter<T> {

    private Context                   mContext;
    // 自定义布局layout文件
    private int                       mResId;
    // 数据源
    private List<T>                   mDatas;
    // 全部的控件，用来切换时改变显示
    private List<TabLayoutViewHolder> mTabLayoutViewHolders;
    // tabLayout
    private TabLayout                 mTabLayout;
    // 简化的tab点击事件
    private OnTabSelectListener<T>    mOnTabSelectListener;

    /**
     * Tab选中的接口
     *
     * @param <T>
     */
    public interface OnTabSelectListener<T> {
        void onSelect(int pos, T data);
    }

    protected BaseTabLayoutAdapter(Context context, List<T> datas, int resId) {
        mDatas = datas;
        mContext = context;
        mResId = resId;
        mTabLayoutViewHolders = new ArrayList<>();
    }

    public void attachTabLayout(TabLayout tabLayout) {
        attachTabLayout(tabLayout, 0);
    }

    private void attachTabLayout(TabLayout tabLayout, int posSelect) {
        mTabLayout = tabLayout;
        // 设置导航条高度=0
        tabLayout.setSelectedTabIndicatorHeight(0);
        tabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
            @Override
            public void onTabSelected(TabLayout.Tab tab) {
                int position = tab.getPosition();
                updateStatus(mTabLayoutViewHolders.get(position), mDatas.get(position), true);
                if (mOnTabSelectListener != null) {
                    mOnTabSelectListener.onSelect(position, mDatas.get(position));
                }
            }

            @Override
            public void onTabUnselected(TabLayout.Tab tab) {
                updateStatus(mTabLayoutViewHolders.get(tab.getPosition()), mDatas.get(tab.getPosition()), false);
            }

            @Override
            public void onTabReselected(TabLayout.Tab tab) {
                updateStatus(mTabLayoutViewHolders.get(tab.getPosition()), mDatas.get(tab.getPosition()), true);
            }
        });
        //  初始化view,默认初始化为全部不选中
        View childView;
        TabLayoutViewHolder holder;
        TabLayout.Tab tab;
        for (int i = 0; i < mDatas.size(); i++) {
            childView = LayoutInflater.from(mContext).inflate(mResId, tabLayout, false);
            holder = new TabLayoutViewHolder(childView);
            mTabLayoutViewHolders.add(holder);
            updateStatus(mTabLayoutViewHolders.get(i), mDatas.get(i), false);
            tab = tabLayout.newTab().setCustomView(childView);
            tabLayout.addTab(tab);
        }

        // 选中初始化选中的那个
        TabLayout.Tab tabAt = tabLayout.getTabAt(posSelect);
        if (tabAt != null)
            tabAt.select();
    }

	/**
     * 抽象方法，更新控件状态显示
     * @param holder view holder
     * @param data 数据
     * @param isSelect 是否选中
     */
    public abstract void updateStatus(TabLayoutViewHolder holder, T data, boolean isSelect);

    public void setOnTabSelectListener(OnTabSelectListener<T> onTabSelectListener) {
        mOnTabSelectListener = onTabSelectListener;
    }
    
     /**
     * 公共holder，用来存储和快速获取控件
     */
    public static class TabLayoutViewHolder {
        private View              parentView;
        private SparseArray<View> mCacheViews;

        public View getParentView() {
            return parentView;
        }

        TabLayoutViewHolder(View parentView) {
            this.parentView = parentView;
            mCacheViews = new SparseArray<>();
        }

        public <V extends View> V getView(int resId) {
            V v = (V) mCacheViews.get(resId);
            if (v == null) {
                v = (V) parentView.findViewById(resId);
                if (v != null) {
                    mCacheViews.put(resId, v);
                }
            }
            return v;
        }
    }
}


// 如何使用？
BaseTabLayoutAdapter<BeautifyToolTabData> baseTabLayoutAdapter =
                new BaseTabLayoutAdapter<ModelData>(getContext(), tabDatas, R.layout.kiwi_item_beautify_tably) {
                    @Override
                    public void updateStatus(TabLayoutViewHolder holder, ModelData data, boolean isSelect) {
                        holder.<TextView>getView(R.id.tv_title).setText(data.getTitle());
                        holder.<ImageView>getView(R.id.iv_icon).setImageResource(isSelect ? data.getSelectRes() : data.getUnSelectRes());
                    }
                };
baseTabLayoutAdapter.setOnTabSelectListener(new BaseTabLayoutAdapter.OnTabSelectListener<ModelData>() {
            @Override
            public void onSelect(int pos, ModelData data) {
          	// 选中事件  
            }
        });
baseTabLayoutAdapter.attachTabLayout(mTabLayout);
```

