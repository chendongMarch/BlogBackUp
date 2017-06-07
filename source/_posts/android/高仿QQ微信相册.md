---
layout: post
title: 高仿QQ微信相册
date: 2016-11-02
category: Android
tags: [Android]
keywords:  
---


## 1. 前言

> 结合微信和QQ相册的一些特点实现系统相册的选择，这篇博客主介绍了开发过程中遇到的点，记录一下。

> ContentProvider中数据的获取  
微信的浮动相册照片时间显示  
微信的dialog选择相册  
QQ的滑动选中照片  
QQ的图片角标数字显示及更新界面的相关优化  

<!--more-->

## 2. 演示视频和源代码
[仿QQ微信相册选择效果视频演示](http://7xtjec.com1.z0.glb.clouddn.com/qq_wechat_select_image.mp4)

[GitHub源代码](https://github.com/chendongMarch/CommonLib/tree/master/lib-dev/src/main/java/com/march/dev/uikit/selectimg)
 
 
## 3. 推荐阅读
[仿QQ相册RecyclerView滑动选中](http://blog.csdn.net/chendong_/article/details/52454805)

[仿QQ相册RecyclerView滑动选中进阶](http://blog.csdn.net/chendong_/article/details/52473160)

[RecyclerView快速适配Adapter](http://blog.csdn.net/chendong_/article/details/50897581)



## 4. 数据获取

### 4.1 定义照片的数据结构
从ContentProvider中获取结构化数据,实现`Comparable`和`Parcelable`接口，用于数据传输和比较。

```java
public class ImageInfo implements Comparable<ImageInfo>, Parcelable {
    // 设置id为自增长的组件
    private Integer id;
    // 文件地址
    private String path;
    // 照片名字
    private String name;
    // 秒数
    private String date;


    @Override
    public int hashCode() {
        return 1;
    }

    @Override
    public boolean equals(Object o) {
        return super.equals(o) && path.equals(((ImageInfo) o).getPath());
    }

    @Override
    public int compareTo(@NonNull ImageInfo another) {
        long a = Long.parseLong(date);
        long b = Long.parseLong(another.getDate());
        if (b > a) {
            return 1;
        } else if (b < a) {
            return -1;
        } else {
            return 0;
        }
    }


    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeValue(this.id);
        dest.writeString(this.path);
        dest.writeString(this.name);
        dest.writeString(this.date);
    }

    public ImageInfo() {
    }

    protected ImageInfo(Parcel in) {
        this.id = (Integer) in.readValue(Integer.class.getClassLoader());
        this.path = in.readString();
        this.name = in.readString();
        this.date = in.readString();
    }

    public static final Creator<ImageInfo> CREATOR = new Creator<ImageInfo>() {
        @Override
        public ImageInfo createFromParcel(Parcel source) {
            return new ImageInfo(source);
        }

        @Override
        public ImageInfo[] newArray(int size) {
            return new ImageInfo[size];
        }
    };
}

```

---
### 4.2 获取数据
使用ContentProvider获取手机相册中的数据，并按照时间排序

```java
	/**
     * 获取全部照片信息
     *
     * @param context 上下文
     * @return 照片信息列表
     */
    public static List<ImageInfo> getImagesByMediaStore(Context context) {
        List<ImageInfo> mImageInfoList = new ArrayList<>();
        Cursor imageCursor;
        try {
            final String[] projection = {MediaStore.Images.Media.DATA, MediaStore.Images.Media._ID, MediaStore.Images.Media.DATE_MODIFIED};
            final String sortOrder = MediaStore.Images.Media._ID;
            imageCursor = MediaStore.Images.Media.query(context.getContentResolver(), MediaStore.Images.Media.EXTERNAL_CONTENT_URI, projection, null, null, sortOrder);
            if (imageCursor != null && imageCursor.getCount() > 0) {
                while (imageCursor.moveToNext()) {
                    ImageInfo imageInfo = new ImageInfo();
                    // 返回data在第几列，并获取地址
                    int dataColumnIndex = imageCursor.getColumnIndex(MediaStore.Images.Media.DATA);
                    String path = imageCursor.getString(dataColumnIndex);
                    File file = new File(path);
                    // 该路径下的文件存在则添加到集合中
                    if (file.exists()) {
                        // 添加路径到对象中
                        imageInfo.setPath(path);
                        // 插入修改时间
                        int modifiedColumnIndex = imageCursor.getColumnIndex(MediaStore.Images.Media.DATE_MODIFIED);
                        String modifiedDate = imageCursor.getString(modifiedColumnIndex);
                        // 添加修改时间到对象中
                        imageInfo.setDate(modifiedDate);
                        // 添加名字
                        mImageInfoList.add(imageInfo);
                    }
                }
                imageCursor.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        // 按降序排
        Collections.sort(mImageInfoList);
        return mImageInfoList;
    }
```

### 4.3 数据整理
按照照片数据的目录结构整理成map，同目录的照片放在一起

```java
	/**
     * 目录 + 目录下的照片列表
     *
     * @param imageInfoList 全部的照片信息
     * @return (目录 照片信息列表)
     */
    public static Map<String, List<ImageInfo>> formatImages4EachDir(List<ImageInfo> imageInfoList) {
        Map<String, List<ImageInfo>> map = new ArrayMap<>();
        map.clear();
        List<ImageInfo> imageInfoAll = new ArrayList<>();
        imageInfoAll.addAll(imageInfoList);
        map.put(ALL_IMAGE_KEY, imageInfoAll);
        for (ImageInfo imageInfo : imageInfoList) {
            if (imageInfo != null && imageInfo.getPath() != null) {
                File file = new File(imageInfo.getPath());
                if (file.exists()) {
                    if (file.getParentFile() != null) {
                        if (map.containsKey(file.getParentFile().getName())) {
                            // 如果key已经存在
                            map.get(file.getParentFile().getName()).add(imageInfo);
                        } else {
                            // 如果key不存在
                            List<ImageInfo> tempImageInfoList = new ArrayList<>();
                            tempImageInfoList.add(imageInfo);
                            map.put(file.getParentFile().getName(), tempImageInfoList);
                        }
                    }
                }
            }
        }
        return map;
    }
``` 

## 5. 在上方显示当前照片时间
### 5.1 获取需要显示的时间数据
需求是在界面的上方动画显示当前RecyclerView最顶部的一张照片的时间
首先获取当前第一张照片的时间，使用`RecyclerView$OnChildAttachStateChangeListener`和`GridLayoutManager$findFirstVisibleItemPosition()`来获取第一个位置的数据。

```java
mImageRv.addOnChildAttachStateChangeListener(new RecyclerView.OnChildAttachStateChangeListener() {
            @Override
            public void onChildViewAttachedToWindow(View view) {
                // 获取第一个位置
                int firstVisibleItemPosition = mLayoutManager.findFirstVisibleItemPosition();
                if (firstVisibleItemPosition < 0)
                    return;
                String date = mCurrentImages.get(firstVisibleItemPosition).getDate();
                mDateTv.setText(simpleDateFormat.format(new Date(Long.parseLong(date) * 1000)));
            }

            @Override
            public void onChildViewDetachedFromWindow(View view) {

            }
        });
```

### 5.2 根据滑动状态显示隐藏时间
动画显示和隐藏时间，监听RecyclerView的滑动，当RecyclerView滑动时显示，当RecyclerView停止时隐藏。

```java
mAlphaAnimator = ObjectAnimator
                .ofFloat(mDateTv, "alpha", 1.0f, 0f)
                .setDuration(1500);
                
mImageRv.addOnScrollListener(new RecyclerView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);
                if (newState == RecyclerView.SCROLL_STATE_IDLE) {
                    if (!mAlphaAnimator.isRunning())
                        mAlphaAnimator.start();
                } else {
                    if (mAlphaAnimator.isRunning())
                        mAlphaAnimator.cancel();
                    mDateTv.setAlpha(1);
                }
            }
        });
```



## 6. 实现QQ滑动选中
我自定义了滑动选中的控件，相关原理和使用在另一篇博客中单独介绍 
[仿QQ相册RecyclerView滑动选中进阶
](http://chendongmarch.github.io/2016/09/06/Android%E5%BC%80%E5%8F%91/%E4%BB%BFQQ%E7%9B%B8%E5%86%8CRecyclerView%E6%BB%91%E5%8A%A8%E9%80%89%E4%B8%AD/)




## 7. QQ角标数字显示及优化
显示该张图片在被选中的图片中是第几张

```java
tv.setText(String.valueOf(mSelectImages.indexOf(data) + 1));
```

滑动选中的监听事件

```java
mSlidingSelectLy.setOnSlidingSelectListener(new SlidingSelectLayout.OnSlidingSelectListener<ImageInfo>() {
            @Override
            public void onSlidingSelect(int pos, View parentView, ImageInfo data) {
                notifyCheckOrNotCheck((RvViewHolder) mImageRv.findViewHolderForAdapterPosition(pos), data);
            }
        });
```

重点，在更新角标显示的时候，最优化的方法是调用`adapter.notifyItemChanged(int pos)`,但是调用此方法，会导致图片重新加载会闪烁一下，观察了一下QQ，没有这种情况,想到的方法是拿到ViewHolder之后从Holder中拿到角标控件，将它的状态改变(如果当前是被选中的改为不选中，如果当前是不选中的改为选中)，避免重新加载图片。

```java
	/**
     * 修改选中和不选中的状态
     *
     * @param holder
     */
    private void notifyCheckOrNotCheck(RvViewHolder holder, ImageInfo imageInfo) {
        TextView tv = (TextView) holder.getView(R.id.tv_select_image);
        View coverView = holder.getView(R.id.cover_select_image);
        // 是否选中
        if (tv.isSelected()) {
        	  // 选中改为不选中
            mSelectImages.remove(imageInfo);
            tv.setSelected(false);
            coverView.setVisibility(View.GONE);
            tv.setText("");
        } else {
        	  // 不选中改为选中
            if (mSelectImageMaxNum != NO_LIMIT && mSelectImages.size() >= mSelectImageMaxNum) {
                Toaster.get().show(mContext, "最多只能选择" + mSelectImageMaxNum + "张");
                return;
            }
            mSelectImages.add(imageInfo);
            tv.setSelected(true);
            coverView.setVisibility(View.VISIBLE);
            tv.setText(String.valueOf(mSelectImages.indexOf(imageInfo) + 1));
        }
        mEnsureTv.setText("完成  " + mSelectImages.size() + "/" + mSelectImageMaxNum);
        changeNumberDisplayByHolder();
    }
```


经过上面的操作，可以达到不重新加载照片的需求。新需求，当修改其中一张照片的选中状态时，他的角标和其他所有被选中图片的角标都会改变，因为中间出现了数字的改变其他数字也要改变，那就不可避免的要调用`notifyDataSetChanged()`,之前所做的工作就全部浪费了。想到的解决方案是拿到第一个显示和最后一个显示的位置，然后拿到所有的ViewHolder，更新这些正在桌面上显示的Holder的角标。

```java
	// 更新其他的item更新数字显示
    private void changeNumberDisplayByHolder() {
        int findSelectCount = 0;
        ImageInfo temp;
        TextView tv;
        RvViewHolder holder;
        int firstVisibleItemPosition = mLayoutManager.findFirstVisibleItemPosition();
        int lastVisibleItemPosition = mLayoutManager.findLastVisibleItemPosition();
        for (int i = firstVisibleItemPosition; i <= lastVisibleItemPosition; i++) {
            //尽量提前推出，只对第一屏有效
            if (findSelectCount >= mSelectImages.size()) {
                break;
            }
            holder = (RvViewHolder) mImageRv.findViewHolderForAdapterPosition(i);
            tv = (TextView) holder.getView(R.id.tv_select_image);
            if (tv.isSelected()) {
            	   // 如果这个是被选中的，那么更新显示
                temp = (ImageInfo) tv.getTag(R.id.data_image_info);
                if (temp != null && mSelectImages.contains(temp)) {
                    tv.setText(String.valueOf(mSelectImages.indexOf(temp) + 1));
                    findSelectCount++;
                }
            }
        }
    }
```
