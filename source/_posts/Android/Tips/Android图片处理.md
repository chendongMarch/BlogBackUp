---
layout: post
title: Android图像处理相关
category: Android
tags: 
	- Android
	- AndroidTips
description: Android图像处理
abbrlink: 2699277879
date: 2016-01-18 00:00:00
keywords:
---


## Bitmap和Drawable转换

```java  
//drawable 转  bitmap
public static Bitmap drawableToBitmap(Drawable drawable) {    
	int width = drawable.getIntrinsicWidth();    
	int height = drawable.getIntrinsicHeight();    
	Bitmap bitmap = Bitmap.createBitmap(width, height, drawable.getOpacity() != PixelFormat.OPAQUE ? Bitmap.Config.ARGB_8888 : Bitmap.Config.RGB_565);    
	Canvas canvas = new Canvas(bitmap);    
	drawable.setBounds(0, 0, width, height);    
	drawable.draw(canvas);    
return bitmap;        
	}    

//bitmap 转 drawable
public static Drawable bitmapToDrawble(Bitmap bitmap,Context mcontext){  
	Drawable drawable = new BitmapDrawable(mcontext.getResources(), bitmap);  
	return drawable;  
} 
```
## 获取图片创建的时间
```java
//文件修改的时间
File file = new File("");
file.lastModified();
//获取照片拍摄的时间
// MediaStore.Images.Media下面有DATE_TAKEN,DATE_ADD,DATE_MODIFIED的参数，分别是照片拍摄的时间，添加到ContentProvider的时间，最后修改的时间，经过测试显示，DATE_TAKEN这个字段下面的时间是最靠谱的，不说他是不是准确，但是系统相册也是使用的这个时间作为照片信息，亲测。另外，有趣的是，只有DATE_TAKEN这个字段下是毫秒级的，另外两个都是秒级的。
public static long getImgCreateTime(Context context, String path) {
        long createTime = -1;
        Uri mImageUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
        ContentResolver mContentResolver = context.getContentResolver();
        Cursor mCursor = mContentResolver.query(mImageUri, new String[]{MediaStore.Images.Media.DATE_TAKEN},
                MediaStore.Images.Media.DATA + "=?",
                new String[]{path}, MediaStore.Images.Media.DATE_TAKEN);
        if (mCursor == null) {
            return -1;
        }
        while (mCursor.moveToNext()) {
            //获取图片的路径
            String str = mCursor.getString(mCursor
                    .getColumnIndex(MediaStore.Images.Media.DATE_TAKEN));
            createTime = Long.parseLong(str);
        }
        return createTime ;
    }
```