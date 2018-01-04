---
layout: post
title: RecyclerView
categories:
  - Android
  - View
tags: 
  - Android
  - View
keywords:
  - Android
  - RecyclerView
abbrlink: 2021227864
date: 2015-11-12 00:00:00
---

`RecycerView` 是 `Design` 包下的新控件，用来实现列表功能，相比原来的 `ListView` 和  `GridView` 要强大的多，并且有自动回收复用的机制，已经可以完全替代原先的列表控件了。

借助 `LayoutManager` 实现列表的展示方式，支持水平方向和垂直方向，支持列表、网格和瀑布流，支持自定义展示方式。

借助 `ViewHolder` 持有控件，用来回收和复用。

借助 `ItemDecoration` 为列表增加分割线，实际上是控制在每一项周边绘制不同的间隔。

借助  `ItemAnimator` 实现列表项更新动画。

借助 `SnapHelper` 可以实现类似 `ViewPager` 的效果。

借助 `ItemTouchHelper` 可以实现拖动、侧滑。


## LayoutManager

在代码中设置 `LayoutManager`，需要在设置 `adapter` 之前设置 `LayoutManager`

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

继承 `Recycler.ViewHolder` 实现自定义 `ViewHolder`

这里只是实现简单的功能，旨在介绍了解 `RecyclerView` 的使用。
需要注意的一点是，`RecyclerView` 并没有实现 `OnItemClickListener` 事件，也就是我们无法直接获得 `Item` 的点击事件。
采用的方法是在最外层的父布局添加点击事件来模拟 `Item` 点击。
`getAdapterPosition();` 函数获取当前 `Item` 在 `Adapter` 中的位置
`getLayoutPosition());` 函数获得当前 `Item` 在布局中的位置

```java
class CustomViewHolder extends RecyclerView.ViewHolder {
    
    TextView tv;
    
    public CustomViewHolder(View itemView) {
        super(itemView);
        tv = (TextView) itemView.findViewById(R.id.test_tv);
        itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.i("chendong", "点击了item" + getAdapterPosition() + " layout --" + getLayoutPosition());
            }
        });
    }
}
```


## Adapter

继承 `Recycler.Adapter` 实现适配器，下面是一个基本的实现，在实际过程中通常会封装来简化 `Adapter`，这里是我封装的一个类库，扩展了一些常用的功能 [RecyclerView LightAdapter](http://cdevlab.top/article/1632666977/)。

```java
class ContentAdapter extends RecyclerView.Adapter<CustomViewHolder> {
    public ContentAdapter() {
        //在这里获得需要的参数
    }
    @Override
    public int getItemViewType(int position) {
        // 获取数据的类型，用于多类型适配
        return 0;
    }
    @Override
    public CustomViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        // 获取对应类型的 ViewHolder,加载不同的布局，类似 ListView 中 getView() 的代码
        View view = getLayoutInflater().inflate(R.layout.test, parent, false);
        return new CustomViewHolder(view);
    }
    @Override
    public void onBindViewHolder(CustomViewHolder holder, int position) {
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


类似 `ListView` 的数据更新，但是开放了更多方法，支持局部更新，而不用更新整个列表。

```java
// 更新列表
adapter.notifyDataSetChanged();
// 删除某一位置的Item
adapter.notifyItemRemoved(0);
// 修改某一位置的Item
adapter.notifyItemChanged(0);
// 向某一位置掺入Item
adapter.notifyItemInserted(0);
// 将pos1的Item移动到pos2
adapter.notifyItemMoved(0,1);
// 通知部分Item改变
adapter.notifyItemRangeChanged(0,4);
// 向0-4的位置插入数据
adapter.notifyItemRangeInserted(0,4);
// 移除0-4的数据
adapter.notifyItemRangeRemoved(0,4);
```


## 分割线

`RecyclerView` 提供了插入分隔线的方法，但是是个抽象类，需要我们自己重写，并且没有实现好的默认分隔线。

```java
mReplyRv.addItemDecoration(new RecyclerView.ItemDecoration() {
    // onDraw方法先于绘制Item
    @Override
    public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {
        super.onDraw(c, parent, state);
    }
    // onDrawOver在绘制Item之后，一般我们选择复写其中一个即可。
    @Override
    public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
        super.onDrawOver(c, parent, state);
    }
    // getItemOffsets 可以通过outRect.set()为每个Item设置一定的偏移量，主要用于绘制
    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
        super.getItemOffsets(outRect, view, parent, state);
    }
});
```


##  动画效果 

`RecyclerView` 为数据的更新增加了动画效果，并且提供了一个默认的动画，我们可以通过实现 `ItemAnimator` 实现自己的动画效果。

```java
recyclerView.setItemAnimator(new DefaultItemAnimator());
```

## ItemTouchHelper

```java
public class ItemTouchHelperWrap {

    public static <D> ItemTouchHelper init(final Activity activity, 
                                           final List<D> datas, 
                                           final RecyclerView.Adapter adapter, 
                                           RecyclerView recyclerView) {

        ItemTouchHelper itemTouchHelper = new ItemTouchHelper(new ItemTouchHelper.Callback() {

            @Override
            public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
                // getMovementFlags 用于设置是否处理拖拽事件和滑动事件
                // 以及拖拽和滑动操作的方向，比如如果是列表类型的RecyclerView
                // 拖拽只有UP、DOWN两个方向，而如果是网格类型的则有UP、DOWN、LEFT、RIGHT四个方向
                
                int dragFlags;
                int swipeFlags;
                if (recyclerView.getLayoutManager() instanceof GridLayoutManager) {
                    dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN | ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
                    swipeFlags = 0;
                    swipeFlags = ItemTouchHelper.START | ItemTouchHelper.END;

                } else {
                    dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN;
                    swipeFlags = ItemTouchHelper.START | ItemTouchHelper.END;
                }
                return makeMovementFlags(dragFlags, swipeFlags);
            }

            @Override
            public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
                // 如果我们设置了非0的dragFlags
                // 那么当我们长按item的时候就会进入拖拽并在拖拽过程中不断回调onMove()方法
                // 我们就在这个方法里获取当前拖拽的item和已经被拖拽到所处位置的item的ViewHolder
                // 有了这2个ViewHolder，我们就可以交换他们的数据集并调用Adapter的notifyItemMoved方法来刷新item
                int fromPos = viewHolder.getAdapterPosition();
                int toPos = target.getAdapterPosition();
                Log.e("chendong", "moved -> " + fromPos + "   " + toPos);
                if (fromPos < toPos) {
                    for (int i = fromPos; i < toPos; i++) {
                        Collections.swap(datas, i, i + 1);
                    }
                } else {
                    for (int i = fromPos; i > toPos; i--) {
                        Collections.swap(datas, i, i - 1);
                    }
                }
                adapter.notifyItemMoved(fromPos, toPos);
                return true;
            }


            @Override
            public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
                int pos = viewHolder.getAdapterPosition();
                adapter.notifyItemRemoved(pos);
                datas.remove(pos);
            }

            @Override
            public void onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState) {
                //当长按选中item的时候（拖拽开始的时候）调用
                if (actionState != ItemTouchHelper.ACTION_STATE_IDLE && actionState != ItemTouchHelper.ACTION_STATE_SWIPE) {
                    viewHolder.itemView.setBackgroundColor(Color.LTGRAY);
                    Vibrator vib = (Vibrator) activity.getSystemService(Service.VIBRATOR_SERVICE);
                    vib.vibrate(70);
                }
//              if (viewHolder != null) {
//                  getDefaultUIUtil().onSelected(((RvViewHolder) viewHolder).getView(R.id.item_quickadapter_title));
//              }
                super.onSelectedChanged(viewHolder, actionState);
            }

            @Override
            public void clearView(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
                // 当手指松开的时候（拖拽完成的时候）调用
                super.clearView(recyclerView, viewHolder);
                // getDefaultUIUtil().clearView(((RvViewHolder) viewHolder).getView(R.id.item_quickadapter_title));
                viewHolder.itemView.setBackgroundColor(Color.WHITE);
            }

            @Override
            public boolean isLongPressDragEnabled() {
                return false;
            }
        });
        itemTouchHelper.attachToRecyclerView(recyclerView);
        return itemTouchHelper;
    }
}
```



