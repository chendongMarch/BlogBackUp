---
layout: post
title: TabLayout的使用详解
categories:
  - Android
  - View
tags: Android
keywords:
  - Android
  - TabLayout
abbrlink: 1315779230
date: 2015-06-23 00:00:00
---


本文主要介绍 `TabLayout` 的详细使用，持续完善中... 

> 1. `TabLayout` 的基本属性及效果 
> 2. `TabLayout` 与 `ViewPager` 联动
> 3. 自定义 `TabLayout` 的 `tab` 显示


<!--more-->


## 基本属性介绍

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

  
## 与 ViewPager 联动
使用 `TabLayout` 的 `setupWithViewPager()` 方法，可以实现与 `ViewPager` 的联动，但是需要注意的是写 `ViewPager` 时需要实现 `getTitle()` 方法，用来设置 `tab` 的标题。原先给 `Tab` 设置的标题将会被覆盖掉。

```java
tabLy.setupWithViewPager(viewPager);
```




## 自定义 Tab 显示
默认的 `TabLayout` 是可以显示文字和 `Icon` 的，但是定制度不是很高，往往不能实现预期效果，不过好在 `TabLayout` 开放了 `setCustomView(childView)` 这个 `API` 来支持我们自定义 `TabLayout` 的UI，当然自定义显示之后也损失了切换时颜色切换等效果，需要我们自己来处理，下面是我定义了一个辅助类，使用类似 `adapter` 的形式来加载 `TabLayout` 的每个 `tab` 的显示，实现自定义效果，也对选中事件等作了简化操作。

```java
/**
 * CreateAt : 2017/3/29
 * Describe : TabLayout适配器,快速实现TabLayout自定义View数据加载
 *
 * @author chendong
 */
public abstract class BaseTabAdapter<T> {

    public static final String TAG = BaseTabAdapter.class.getSimpleName();

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

    private boolean mIsReady;

    /**
     * Tab选中的接口
     *
     * @param <T>
     */
    public interface OnTabSelectListener<T> {
        void onSelect(int pos, T data);
    }

    
    protected BaseTabAdapter(Context context, List<T> datas, int resId) {
        mDatas = datas;
        mContext = context;
        mResId = resId;
        mTabLayoutViewHolders = new ArrayList<>();
    }

    public void attachTabLayout(TabLayout tabLayout) {
        tabLayout.clearOnTabSelectedListeners();
        attachTabLayout(tabLayout, 0);
    }

    public void attachTabLayout(TabLayout tabLayout, int posSelect) {
        mIsReady = false;
        mTabLayout = tabLayout;
        // 设置导航条高度=0
        tabLayout.setSelectedTabIndicatorHeight(0);
        tabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
            @Override
            public void onTabSelected(TabLayout.Tab tab) {
                L.e(TAG, "onTabSelected " + tab.getPosition());
                int position = tab.getPosition();
                if (mIsReady && mOnTabSelectListener != null) {
                    mOnTabSelectListener.onSelect(position, mDatas.get(position));
                }
                updateStatus(mTabLayoutViewHolders.get(position), mDatas.get(position), true);
            }

            @Override
            public void onTabUnselected(TabLayout.Tab tab) {
                L.e(TAG, "onTabUnselected " + tab.getPosition());
                updateStatus(mTabLayoutViewHolders.get(tab.getPosition()), mDatas.get(tab.getPosition()), false);
            }

            @Override
            public void onTabReselected(TabLayout.Tab tab) {
                L.e(TAG, "onTabReselected " + tab.getPosition());
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
        if (tabAt != null) {
            tabAt.select();
            if (mOnTabSelectListener != null) {
                mOnTabSelectListener.onSelect(posSelect, mDatas.get(posSelect));
            }
        }
        mIsReady = true;
    }
    
    public void notifyItemChanged(int pos) {
        if (pos >= 0 && pos < mDatas.size()) {
            TabLayout.Tab tabAt;
            tabAt = mTabLayout.getTabAt(pos);
            updateStatus(mTabLayoutViewHolders.get(pos), mDatas.get(pos), tabAt != null && tabAt.isSelected());
        }
    }

    public void notityDataSetChanged() {
        TabLayout.Tab tabAt;
        for (int i = 0; i < mDatas.size(); i++) {
            tabAt = mTabLayout.getTabAt(i);
            updateStatus(mTabLayoutViewHolders.get(i), mDatas.get(i), tabAt != null && tabAt.isSelected());
        }
    }

    /**
     * 抽象方法，更新控件状态显示
     *
     * @param holder   view holder
     * @param data     数据
     * @param isSelect 是否选中
     */
    public abstract void updateStatus(TabLayoutViewHolder holder, T data, boolean isSelect);

    public void setOnTabSelectListener(OnTabSelectListener<T> onTabSelectListener) {
        mOnTabSelectListener = onTabSelectListener;
    }

    public interface OnBuildTabListener<D> {
        TabLayout.Tab onBuild(TabLayout.Tab tab, D data);
    }

    public static <D> void buildTabLayout(TabLayout tabLayout, List<D> datas, OnBuildTabListener<D> listener) {
        TabLayout.Tab tab;
        for (D data : datas) {
            tab = tabLayout.newTab();
            tabLayout.addTab(listener.onBuild(tab, data));
        }
    }

    /**
     * 公共holder，用来存储和快速获取控件
     */
    public static class TabLayoutViewHolder {
        private View              parentView;
        private SparseArray<View> mCacheViews;

        public View getItemView() {
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


    public void setUpViewPager(final ViewPager viewPager) {
        if (mTabLayout == null) {
            Log.e("chendong", "TabLayout is null ,invoke attachTabLayout first");
            return;
        }
        mTabLayout.addOnTabSelectedListener(new MyOnTabSelectedListener() {
            @Override
            public void onTabSelected(TabLayout.Tab tab) {
                if (viewPager != null)
                    viewPager.setCurrentItem(tab.getPosition());
            }
        });
        if (viewPager != null)
            viewPager.addOnPageChangeListener(new MyOnPageChangeListener() {
                @Override
                public void onPageSelected(int position) {
                    if (mTabLayout != null) {
                        TabLayout.Tab tabAt = mTabLayout.getTabAt(position);
                        if (tabAt != null)
                            tabAt.select();
                    }
                }

            });
    }
}
```

how to usage?

```java
BaseTabAdapter<Baby> adapter = new BaseTabAdapter<Baby>(mActivty, babies, R.layout.vip_select_family_baby_item)
    @Override
    public void updateStatus(TabLayoutViewHolder holder, Baby data, boolean isSelect) {
        holder.getView(R.id.parent).getLayoutParams().width = (int) (BaoBaoApplication.DISPLAY_WIDTH / 3f);
        TextView nameTv = holder.getView(R.id.tv_name);
        nameTv.setText(data.getBabyName());
        holder.getView(R.id.view_line).setBackgroundColor(isSelect
                ? ContextCompat.getColor(mBaseFragmentActivity, R.color.colorPrimary)
                : ContextCompat.getColor(mBaseFragmentActivity, R.color.color_EFEFEF));
    }
};
adapter.setOnTabSelectListener(new BaseTabAdapter.OnTabSelectListener<Baby>() {
    @Override
    public void onSelect(int pos, Baby data) {
        // 请求数据，展示数据
    }
});
adapter.attachTabLayout(mTabLayout, 1);
```
