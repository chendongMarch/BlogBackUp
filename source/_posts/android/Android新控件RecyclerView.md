---
layout: post
title: Android新控件RecyclerView
date: 2015-11-12
category: Android
tags: [Android,Android5.+]
keywords: Android,Android5.+
description: Android新控件RecyclerView
---

## 前言
- RecycerView是类似于ListView和GridView的控件，可以使用列表的形式展示数据，同时也很好的支持了瀑布流的模式，但是他根本上是继承自ViewGroup的
`public class RecyclerView extends ViewGroup implements ScrollingView, NestedScrollingChild`。

## LayoutManager
- 在代码中设置LayoutManager，需要在设置adapter之前设置LayoutManager

```java
RecyclerView.LayoutManager layoutManager = null;
//线性布局，第二个参数支持水平垂直两种方向，第三个参数用来设置是否反向显示
layoutManager = new LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false);
//网格布局，第二个参数设置列数，第三个参数支持水平垂直两种方向，第四个参数用来设置是否反向显示
layoutManager = new GridLayoutManager(this,3,GridLayoutManager.HORIZONTAL,false);
//瀑布流布局，第一个参数设置列数，第二个参数支持水平垂直两种方向
 layoutManager = new StaggeredGridLayoutManager(3,StaggeredGridLayoutManager.VERTICAL);
recyclerView.setLayoutManager(layoutManager);
```

## ViewHolder
- 继承Recycler.ViewHolder实现自定义ViewHolder

```java
//这里只是实现简单的功能，旨在介绍了解RecyclerView的使用。
//需要注意的一点是，RecyclerView并没有实现OnItemClickListener事件，也就是我们无法直接获得Item的点击事件。
//采用的方法是在最外层的父布局添加点击事件来模拟Item点击。
//getAdapterPosition();函数获取当前Item在Adapter中的位置//getLayoutPosition());函数获得当前Item在布局中的位置
class CustomViewHolder extends RecyclerView.ViewHolder {
        TextView tv;
        public CustomViewHolder (View itemView) {
            super(itemView);
            tv = (TextView) itemView.findViewById(R.id.test_tv);
            itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.i("chendong", "点击了item" + getAdapterPosition() + "   layout --"+getLayoutPosition());
				}
	        });
        }
    }
```


## Adapter
- 继承Recycler.Adapter实现适配器

```java
class ContentAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {

        public ContentAdapter() {
        //在这里获得需要的参数
        }
        @Override
        public int getItemViewType(int position) {
         //获取数据的类型，用于多类型适配
        }

        @Override
        public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        //获取对应类型的ViewHolder,加载不同的布局，类似ListView中getView（）的代码，示例代码：
        View view = getLayoutInflater().inflate(R.layout.test, parent, false);
        return new CustomViewHolder(view);
        }

        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        //向ViewHolder中的控件绑定数据,这里用的是强转，也可以使用泛型直接生成CustomViewHolder，但是多类型适配时还是需要强转。
        CustomViewHolder temp = (CustomViewHolder)holder;
        holder.tv.setText("a");
        }

        @Override
        public int getItemCount() {
        //获取Item个数
            return 0;
        }
    }
```


## 数据更新
```java
//类似ListView的数据更新，但是开放了更多方法，由于RecyclerView继承自ViewGroup，可以轻松实现对其中每一个控件的操作，而不用更新整个列表。
//舒心列表
adapter.notifyDataSetChanged();
//删除某一位置的Item
adapter.notifyItemRemoved(0);
//修改某一位置的Item
adapter.notifyItemChanged(0);
//向某一位置掺入Item
adapter.notifyItemInserted(0);
//将pos1的Item移动到pos2
adapter.notifyItemMoved(0,1);
//通知部分Item改变
adapter.notifyItemRangeChanged(0,4);
//向0-4的位置插入数据
adapter.notifyItemRangeInserted(0,4);
//移除0-4的数据
adapter.notifyItemRangeRemoved(0,4);
```


## 分割线
- RecyclerView提供了插入分隔线的方法，但是是个抽象类，需要我们自己重写，并且没有实现好的默认分隔线。
```java
recyclerView.addItemDecoration(new RecyclerView.ItemDecoration() {
//onDraw方法先于绘制Item
            @Override
            public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {
                super.onDraw(c, parent, state);
            }
//onDrawOver在绘制Item之后，一般我们选择复写其中一个即可。
            @Override
            public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
                super.onDrawOver(c, parent, state);
            }
//getItemOffsets 可以通过outRect.set()为每个Item设置一定的偏移量，主要用于绘制
            @Override
            public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
                super.getItemOffsets(outRect, view, parent, state);
            }
        });
```
##  动画效果 
- RecyclerView为数据的更新增加了动画效果,并且提供了一个默认的动画，我们可以通过实现ItemAnimator实现自己的动画效果。

```java
recyclerView.setItemAnimator(new DefaultItemAnimator());
```


## 总结
- RecyclerView采用插拔式的数据填充，提供了更加强大的显示效果，可以轻松实现，各个方向的ListView,GridView,瀑布流效果，简单方便。但在进行多类型分类适配时需要创建大量的ViewHolder，需要我们进一步取封装和完善。

