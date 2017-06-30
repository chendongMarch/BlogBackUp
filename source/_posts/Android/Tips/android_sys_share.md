---
layout: post
title: Android系统分享的注册和调起
categories:
  - Android
  - Tips
tags:
  - Android
keywords:
  - Android
  - 系统分享
abbrlink: 3733894393
date: 2016-03-24 00:00:00
---


系统相册照片长按时会弹出分享发送选项，可以选择分享到QQ，微信等，本文主要介绍：

1. 如何将自己的应用注册到系统分享中，使用户可以将照片视频文件等发送到自己的应用中。
2. 如何接受解析系统分享发送过来的数据。
3. 如何唤醒系统分享，分享照片视频文件到别的应用。

<!--more-->

## 注册系统的分享

需要在 `AndroidManifest.xml` 文件声明 `<intent-filter>`，使得你可以在用户点击分享/发送按钮时调起你的应用，将图片和文字等分享到你的App。

关于 `mimeType` 的相关类型，请查看 [MimeUtils.java](http://androidxref.com/4.4.4_r1/xref/libcore/luni/src/main/java/libcore/net/MimeUtils.java) 文件。

```xml
<activity android:name=".activity.HandleShareActivity">
    <intent-filter>
        <!--接受单个文件分享-->
        <action android:name="android.intent.action.SEND"/>
        <!--接受多个文件分享-->
        <action android:name="android.intent.action.SEND_MULTIPLE"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <!--接受图片类文件分享 image/jpeg" "image/bmp" "image/gif" "image/jpg" "image/png"-->
        <data android:mimeType="image/*"/>
        <!--接受文本分享 text/plain-->
        <data android:mimeType="text/*"/>
        <!--接受视频分享 video/wav video/mp4-->
        <data android:mimeType="video/*"/>
        <!--接受声音文件分享-->
        <data android:mimeType="audio/*"/>
        <!--接受压缩文件和其他各种文件分享-->
        <data android:mimeType="application/*"/>
    </intent-filter>
</activity>
```


## 接受分享的数据

当直接长按选中某些文本分享时拿到的数据有些不一样，是存储在 `Intent.EXTRA_TEXT` 中的，因此单独处理一下。

```java
/**
 * 处理分享页面发来的intent
 */
public static void handleIntent(Activity context) {
    Intent intent = context.getIntent();
    String type = intent.getType();
    if (type == null)
        return;
    // 当文件格式无法识别或选择了多种类型的文件时，type会变成 */*
    LogUtils.e("type " + type);
    //type可能为的类型是 image/* | video/* | audio/* | text/* | application/* | */*
    if (type.startsWith("text") && intent.getStringExtra(Intent.EXTRA_TEXT) != null) {
        // 当直接选中文本分享时，会存放在EXTRA_TEXT里面，选择文本文件时，仍然存放在EXTRA_STREAM里面
        LogUtils.e("获取到分享的文本 " + intent.getStringExtra(Intent.EXTRA_TEXT));
    } else {
        List<String> sharePaths = getSharePaths(context, intent);
        LogUtils.e("获取到分享的文件的路径 " + sharePaths.toString());
    }
}
```
区分多个文件和单个文件分享，获取分享过来的路径列表

```java
/**
 * 获取分享过来的路径
 *
 * @param context 上下文
 * @param intent  intent
 * @return 路径列表
 */
private static List<String> getSharePaths(Context context, Intent intent) {
    String action = intent.getAction();
    List<String> paths = new ArrayList<>();
    // 单个文件
    if (Intent.ACTION_SEND.equals(action)) {
        Uri imageUri = intent.getParcelableExtra(Intent.EXTRA_STREAM);
        if (imageUri != null) {
            String realPathFromURI = getRealPathFromURI(context, imageUri);
            if (realPathFromURI != null)
                paths.add(realPathFromURI);
        }
    } 
    // 多个文件
    else if (Intent.ACTION_SEND_MULTIPLE.equals(action)) {
        ArrayList<Uri> imageUris = intent.getParcelableArrayListExtra(Intent.EXTRA_STREAM);
        if (imageUris != null) {
            for (Uri uri : imageUris) {
                paths.add(getRealPathFromURI(context, uri));
            }
        }
    }
    return paths;
}
```

如何从 `uri` 中获取存储路径，将 `uri` 进行转换。

```java
/**
 * 从uri获取path
 *
 * @param uri uri
 * @return path
 */
private static String getRealPathFromURI(Context context, Uri uri) {
    final String scheme = uri.getScheme();
    String data = null;
    if (ContentResolver.SCHEME_FILE.equals(scheme)) {
        data = uri.getPath();
    } else if (ContentResolver.SCHEME_CONTENT.equals(scheme)) {
        Cursor cursor = context.getContentResolver()
                .query(uri, new String[]{MediaStore.Images.ImageColumns.DATA}
                        , null, null, null);
        if (null != cursor) {
            if (cursor.moveToFirst()) {
                int index = cursor.getColumnIndex(MediaStore.Images.ImageColumns.DATA)
                if (index > -1) {
                    data = cursor.getString(index);
                }
            }
            cursor.close();
        }
    }
    return data;
}
```

## 唤醒系统分享
 
```java
/**
 * 分享文字
 *
 * @param context 上下文
 * @param title   文字标题
 * @param content 文字内容
 */
public static void shareText(Context context, String title, String content) {
    Intent shareIntent = new Intent();
    shareIntent.setAction(Intent.ACTION_SEND);
    shareIntent.putExtra(Intent.EXTRA_TEXT, content);
    shareIntent.putExtra(Intent.EXTRA_TITLE, title);
    shareIntent.setType("text/plain");
    //设置分享列表的标题，并且每次都显示分享列表
    context.startActivity(Intent.createChooser(shareIntent, "分享到"));
}


/**
 * 分享单张图片
 *
 * @param context 上下文
 * @param path    图片的路径
 */
public static void shareImage(Context context, String path) {
    //由文件得到uri
    Uri imageUri = Uri.fromFile(new File(path));
    Intent shareIntent = new Intent();
    shareIntent.setAction(Intent.ACTION_SEND);
    shareIntent.putExtra(Intent.EXTRA_STREAM, imageUri);
    shareIntent.setType("image/*");
    context.startActivity(Intent.createChooser(shareIntent, "分享到"));
}


/**
 * 分享多张图片
 *
 * @param context 上下文
 * @param paths   路径的集合
 */
public static void shareImages(Context context, List<String> paths) {
    ArrayList<Uri> uriList = new ArrayList<>();
    for (String path : paths) {
        uriList.add(Uri.fromFile(new File(path)));
    }
    Intent shareIntent = new Intent();
    shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
    shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, uriList);
    shareIntent.setType("image/*");
    context.startActivity(Intent.createChooser(shareIntent, "分享到"));
}
```
 
 
## 附源码

```java
/**
 * CreateAt : 16/8/13
 * Describe : 唤醒系统分享
 *
 * @author chendong
 */
public class ShareUtils {

    /**
     * 分享文字
     *
     * @param context 上下文
     * @param title   文字标题
     * @param content 文字内容
     */
    public static void shareText(Context context, String title, String content) {
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND);
        shareIntent.putExtra(Intent.EXTRA_TEXT, content);
        shareIntent.putExtra(Intent.EXTRA_TITLE, title);
        shareIntent.setType("text/plain");
        //设置分享列表的标题，并且每次都显示分享列表
        context.startActivity(Intent.createChooser(shareIntent, "分享到"));
    }

    /**
     * 分享单张图片
     *
     * @param context 上下文
     * @param path    图片的路径
     */
    public static void shareImage(Context context, String path) {
        //由文件得到uri
        Uri imageUri = Uri.fromFile(new File(path));
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND);
        shareIntent.putExtra(Intent.EXTRA_STREAM, imageUri);
        shareIntent.setType("image/*");
        context.startActivity(Intent.createChooser(shareIntent, "分享到"));
    }

    /**
     * 分享多张图片
     *
     * @param context 上下文
     * @param paths   路径的集合
     */
    public static void shareImages(Context context, List<String> paths) {
        ArrayList<Uri> uriList = new ArrayList<>();
        for (String path : paths) {
            uriList.add(Uri.fromFile(new File(path)));
        }
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
        shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, uriList);
        shareIntent.setType("image/*");
        context.startActivity(Intent.createChooser(shareIntent, "分享到"));
    }


    /**
     * 处理分享页面发来的intent
     */
    public static void handleIntent(Activity context) {
        Intent intent = context.getIntent();
        String type = intent.getType();
        if (type == null)
            return;
        // 当文件格式无法识别或选择了多种类型的文件时，type会变成 */*
        LogUtils.e("type " + type);
        //type可能为的类型是 image/* | video/* | audio/* | text/* | application/*
        if (type.startsWith("text") && intent.getStringExtra(Intent.EXTRA_TEXT) != null) {
            // 当直接选中文本分享时，会存放在EXTRA_TEXT里面，选择文本文件时，仍然存放在EXTRA_STREAM里面
            LogUtils.e("获取到分享的文本 " + intent.getStringExtra(Intent.EXTRA_TEXT));
        } else {
            List<String> sharePaths = getSharePaths(context, intent);
            LogUtils.e("获取到分享的文件的路径 " + sharePaths.toString());
        }
    }


    /**
     * 获取分享过来的路径
     *
     * @param context 上下文
     * @param intent  intent
     * @return 路径列表
     */
    private static List<String> getSharePaths(Context context, Intent intent) {
        String action = intent.getAction();
        List<String> paths = new ArrayList<>();
        // 单个文件
        if (Intent.ACTION_SEND.equals(action)) {
            Uri imageUri = intent.getParcelableExtra(Intent.EXTRA_STREAM);
            if (imageUri != null) {
                String realPathFromURI = getRealPathFromURI(context, imageUri);
                if (realPathFromURI != null)
                    paths.add(realPathFromURI);
            }
        }
        // 多个文件
        else if (Intent.ACTION_SEND_MULTIPLE.equals(action)) {
            ArrayList<Uri> imageUris = intent.getParcelableArrayListExtra(Intent.EXTRA_STREAM);
            if (imageUris != null) {
                for (Uri uri : imageUris) {
                    paths.add(getRealPathFromURI(context, uri));
                }
            }
        }
        return paths;
    }


    /**
     * 从uri获取path
     *
     * @param uri uri
     * @return path
     */
    private static String getRealPathFromURI(Context context, Uri uri) {
        final String scheme = uri.getScheme();
        String data = null;
        if (ContentResolver.SCHEME_FILE.equals(scheme)) {
            data = uri.getPath();
        } else if (ContentResolver.SCHEME_CONTENT.equals(scheme)) {
            Cursor cursor = context.getContentResolver()
                    .query(uri, new String[]{MediaStore.Images.ImageColumns.DATA}
                            , null, null, null);
            if (null != cursor) {
                if (cursor.moveToFirst()) {
                    int index = cursor.getColumnIndex(MediaStore.Images.ImageColumns.DATA);
                    if (index > -1) {
                        data = cursor.getString(index);
                    }
                }
                cursor.close();
            }
        }
        return data;
    }
}

```



