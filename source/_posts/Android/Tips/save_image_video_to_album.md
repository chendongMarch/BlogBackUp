---
layout: post
title: 保存照片和视频到相册显示
categories:
  - Android
tags:
  - Android
  - AndroidTips
keywords:
  - Android
  - 保存照片视频到相册
abbrlink: 1969447237
date: 2015-08-23 00:00:00
---


照片和视频保存到本地的方法大致都是通过流的方式写入文件里面就可以达到保存到文件夹的目的，但是你保存到文件夹的资源却不一定能够在相册显示出来，只能翻看文件管理。  
怎么能够将保存到本地的照片视频显示在系统相册中，最常用的方式是发送广播扫描的方式来通知系统扫描文件夹，但是这种方式经常不起作用，根本原因在于，向系统发送广播时系统只会去扫描系统资源的相册，如果你保存的文件夹是自己建立的，那么你发广播时系统是不会扫描到的。   
解决这个问题，想要显示在相册需要将数据插入到` ContentProvider `中，因此对于非系统能够扫描到的相册我们使用手动插入的方式。

<!--more-->

## 保存到系统资源相册
系统相册指的的是` Camera `、` DCIM `等等这些目录对应的相册，可能还有其他系统会自动扫描的相册，但是暂时没有去整理这些。  
对于系统相册来说，只需要发送广播进行扫描即可，数据会自动添加到` ContentProvider `中，当然如果不发送广播，在手机重启或者过一段时间之后，扫描操作仍会开启，因此**绝对绝对**不可以自己去进行插入操作，否则相册中会出现两张相同的照片。
 
```java
/**
 * 针对系统文夹只需要扫描,不用插入内容提供者,不然会重复
 *
 * @param context  上下文
 * @param filePath 文件路径
 */
private static void scanFile(Context context, String filePath) {
    if (!checkFile(filePath))
        return;
    Intent intent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
    intent.setData(Uri.fromFile(new File(filePath)));
    context.sendBroadcast(intent);
}
```

## 保存到非系统资源相册
保存到非系统资源相册中时，我们就需要进行` ContentProvider `的插入更新，来达到可以在相册显示的目的。
  
###  初始化 ContentValues 公共字段
照片和视频是有一些公共字段，写一个初始化公共字段的方法，简化` MediaStore `字段的写入操作。

```java
/**
 * 插入时初始化公共字段
 *
 * @param filePath 文件
 * @param time     ms
 * @return ContentValues
 */
private static ContentValues initCommonContentValues(String filePath, long time) {
    ContentValues values = new ContentValues();
    File saveFile = new File(filePath);
    long timeMillis = getTimeWrap(time);
    values.put(MediaStore.MediaColumns.TITLE, saveFile.getName());
    values.put(MediaStore.MediaColumns.DISPLAY_NAME, saveFile.getName());
    values.put(MediaStore.MediaColumns.DATE_MODIFIED, timeMillis);
    values.put(MediaStore.MediaColumns.DATE_ADDED, timeMillis);
    values.put(MediaStore.MediaColumns.DATA, saveFile.getAbsolutePath());
    values.put(MediaStore.MediaColumns.SIZE, saveFile.length());
    return values;
}
```


### 插入照片资源
保存照片到本地，并通知相册显示，需要注意的时间的单位必须是ms

```java
/**
 * 保存到照片到本地，并插入MediaStore以保证相册可以查看到
 * 这是更优化的方法，防止读取的照片获取不到宽高
 *
 * @param context    上下文
 * @param filePath   文件路径
 * @param createTime 创建时间 <=0时为当前时间 ms
 * @param width      宽度
 * @param height     高度
 */
public static void insertImageToMediaStore(Context context, String filePath,
                                           long createTime, int width, int height) {
    if (!checkFile(filePath))
        return;
    createTime = getTimeWrap(createTime);
    ContentValues values = initCommonContentValues(filePath, createTime);
    values.put(MediaStore.Images.ImageColumns.DATE_TAKEN, createTime);
    values.put(MediaStore.Images.ImageColumns.ORIENTATION, 0);
    values.put(MediaStore.Images.ImageColumns.ORIENTATION, 0);
    if (Build.VERSION.SDK_INT > Build.VERSION_CODES.JELLY_BEAN) {
        if (width > 0) values.put(MediaStore.Images.ImageColumns.WIDTH, 0);
        if (height > 0) values.put(MediaStore.Images.ImageColumns.HEIGHT, 0);
    }
    values.put(MediaStore.MediaColumns.MIME_TYPE, getPhotoMimeType(filePath));
    context.getApplicationContext().getContentResolver()
            .insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
}
```


### 插入视频资源
保存视频到本地，并通知相册显示，需要注意的时间的单位必须是ms

```java
/**
 * 保存到视频到本地，并插入MediaStore以保证相册可以查看到
 * 这是更优化的方法，防止读取的视频获取不到宽高
 *
 * @param context    上下文
 * @param filePath   文件路径
 * @param createTime 创建时间 <=0时为当前时间 ms
 * @param duration   视频长度 ms
 * @param width      宽度
 * @param height     高度
 */
public static void insertVideoToMediaStore(Context context, String filePath, long createTime,
                                           int width, int height, long duration) {
    if (!checkFile(filePath))
        return;
    createTime = getTimeWrap(createTime);
    ContentValues values = initCommonContentValues(filePath, createTime);
    values.put(MediaStore.Video.VideoColumns.DATE_TAKEN, createTime);
    if (duration > 0)
        values.put(MediaStore.Video.VideoColumns.DURATION, duration);
    if (Build.VERSION.SDK_INT > Build.VERSION_CODES.JELLY_BEAN) {
        if (width > 0) values.put(MediaStore.Video.VideoColumns.WIDTH, width);
        if (height > 0) values.put(MediaStore.Video.VideoColumns.HEIGHT, height);
    }
    values.put(MediaStore.MediaColumns.MIME_TYPE, getVideoMimeType(filePath));
    context.getContentResolver().insert(MediaStore.Video.Media.EXTERNAL_CONTENT_URI, values);
}
```



## 注意
在应用过程中发现，vivo手机和魅族手机部分机型只支持在文件管理中查看视频，使用本文描述的方法添加后没有效果，用微信保存视频试了一下同样不能在相册中将视频显示出来，应该是手机的原因，特此声明。


## 附源码
贴一下工具类源代码，多了一些辅助方法，比如获取` mime_type `的参数等方法。

```
/**
 * CreateAt : 2017/5/24
 * Describe : 相册更新通知帮助类
 * 创建时间单位ms
 * 视频时长单位ms
 *
 * @author chendong
 */
public class AlbumNotifyHelper {

    public static final String TAG = AlbumNotifyHelper.class.getSimpleName();


    ///////////////////////////////////////////////////////////////////////////
    // 下面是对外公开的重载的方法
    ///////////////////////////////////////////////////////////////////////////

    public static void notifyScanDcim(Context context, String filePath) {
        scanFile(context, filePath);
    }

    public static void insertVideoToMediaStore(Context context, String filePath, long dateTaken, long duration) {
        insertVideoToMediaStore(context, filePath, dateTaken, 0, 0, duration);
    }

    public static void insertVideoToMediaStore(Context context, VideoUtil.VideoInfo videoInfo) {
        insertVideoToMediaStore(context, videoInfo.originalVideoFilePath, videoInfo.dateTaken, videoInfo.width, videoInfo.height, videoInfo.duringTime);
    }

    public static void insertImageToMediaStore(Context context, String filePath, long createTime) {
        insertImageToMediaStore(context, filePath, createTime, 0, 0);
    }


    ///////////////////////////////////////////////////////////////////////////
    // 扫描系统相册核心方法
    ///////////////////////////////////////////////////////////////////////////

    /**
     * 针对系统文夹只需要扫描,不用插入内容提供者,不然会重复
     *
     * @param context  上下文
     * @param filePath 文件路径
     */
    public static void scanFile(Context context, String filePath) {
        if (!checkFile(filePath))
            return;
        Intent intent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
        intent.setData(Uri.fromFile(new File(filePath)));
        context.sendBroadcast(intent);
    }


    ///////////////////////////////////////////////////////////////////////////
    // 非系统相册像MediaContent中插入数据，核心方法
    ///////////////////////////////////////////////////////////////////////////

    /**
     * 针对非系统文件夹下的文件,使用该方法
     * 插入时初始化公共字段
     *
     * @param filePath 文件
     * @param time     ms
     * @return ContentValues
     */
    private static ContentValues initCommonContentValues(String filePath, long time) {
        ContentValues values = new ContentValues();
        File saveFile = new File(filePath);
        long timeMillis = getTimeWrap(time);
        values.put(MediaStore.MediaColumns.TITLE, saveFile.getName());
        values.put(MediaStore.MediaColumns.DISPLAY_NAME, saveFile.getName());
        values.put(MediaStore.MediaColumns.DATE_MODIFIED, timeMillis);
        values.put(MediaStore.MediaColumns.DATE_ADDED, timeMillis);
        values.put(MediaStore.MediaColumns.DATA, saveFile.getAbsolutePath());
        values.put(MediaStore.MediaColumns.SIZE, saveFile.length());
        return values;
    }

    /**
     * 保存到照片到本地，并插入MediaStore以保证相册可以查看到,这是更优化的方法，防止读取的照片获取不到宽高
     *
     * @param context    上下文
     * @param filePath   文件路径
     * @param createTime 创建时间 <=0时为当前时间 ms
     * @param width      宽度
     * @param height     高度
     */
    public static void insertImageToMediaStore(Context context, String filePath, long createTime, int width, int height) {
        if (!checkFile(filePath))
            return;
        createTime = getTimeWrap(createTime);
        ContentValues values = initCommonContentValues(filePath, createTime);
        values.put(MediaStore.Images.ImageColumns.DATE_TAKEN, createTime);
        values.put(MediaStore.Images.ImageColumns.ORIENTATION, 0);
        values.put(MediaStore.Images.ImageColumns.ORIENTATION, 0);
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.JELLY_BEAN) {
            if (width > 0) values.put(MediaStore.Images.ImageColumns.WIDTH, 0);
            if (height > 0) values.put(MediaStore.Images.ImageColumns.HEIGHT, 0);
        }
        values.put(MediaStore.MediaColumns.MIME_TYPE, getPhotoMimeType(filePath));
        context.getApplicationContext().getContentResolver().insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
    }

    /**
     * 保存到视频到本地，并插入MediaStore以保证相册可以查看到,这是更优化的方法，防止读取的视频获取不到宽高
     *
     * @param context    上下文
     * @param filePath   文件路径
     * @param createTime 创建时间 <=0时为当前时间 ms
     * @param duration   视频长度 ms
     * @param width      宽度
     * @param height     高度
     */
    public static void insertVideoToMediaStore(Context context, String filePath, long createTime, int width, int height, long duration) {
        if (!checkFile(filePath))
            return;
        createTime = getTimeWrap(createTime);
        ContentValues values = initCommonContentValues(filePath, createTime);
        values.put(MediaStore.Video.VideoColumns.DATE_TAKEN, createTime);
        if (duration > 0)
            values.put(MediaStore.Video.VideoColumns.DURATION, duration);
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.JELLY_BEAN) {
            if (width > 0) values.put(MediaStore.Video.VideoColumns.WIDTH, width);
            if (height > 0) values.put(MediaStore.Video.VideoColumns.HEIGHT, height);
        }
        values.put(MediaStore.MediaColumns.MIME_TYPE, getVideoMimeType(filePath));
        context.getContentResolver().insert(MediaStore.Video.Media.EXTERNAL_CONTENT_URI, values);
    }


    // 是不是系统相册
    private static boolean isSystemDcim(String path) {
        return path.toLowerCase().contains("dcim") || path.toLowerCase().contains("camera");
    }

    // 获取照片的mine_type
    private static String getPhotoMimeType(String path) {
        String lowerPath = path.toLowerCase();
        if (lowerPath.endsWith("jpg") || lowerPath.endsWith("jpeg")) {
            return "image/jpeg";
        } else if (lowerPath.endsWith("png")) {
            return "image/png";
        } else if (lowerPath.endsWith("gif")) {
            return "image/gif";
        }
        return "image/jpeg";
    }

    // 获取video的mine_type,暂时只支持mp4,3gp
    private static String getVideoMimeType(String path) {
        String lowerPath = path.toLowerCase();
        if (lowerPath.endsWith("mp4") || lowerPath.endsWith("mpeg4")) {
            return "video/mp4";
        } else if (lowerPath.endsWith("3gp")) {
            return "video/3gp";
        }
        return "video/mp4";
    }

    // 获得转化后的时间
    private static long getTimeWrap(long time) {
        if (time <= 0) {
            return System.currentTimeMillis();
        }
        return time;
    }

    // 检测文件存在
    private static boolean checkFile(String filePath) {
        boolean result = FileUtil.fileIsExist(filePath);
        Log.e(TAG, "文件不存在 path = " + filePath);
        return result;
    }
}
```
