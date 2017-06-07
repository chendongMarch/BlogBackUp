---
layout: post
title: Android系统分享的注册和调起
date: 2016-03-24
category: Android
tags: Android
keywords: Android
description: Android系统分享的注册和调起
---


## 前言
> 现在有很多第三方分享平台，比如友盟，shareSdk等，其实我们在相册长按图片点击发送／分享就会调起很多应用，让你选择可以分享到很多平台，比如QQ，微信等，相对使用第三方要简陋一点，但是因为是系统的集成起来也相对简单。

<!--more-->

## 注册系统的分享
需要在manifest文件声明`<intent-filter>`，使得你可以在用户点击分享／发送按钮时调起你的应用，将图片和文字分享到你的App.

```java
<activity
    android:name=".activity.SysShareActivity"
    android:label="我的分享"
    android:theme="@style/AppTheme.NoActionBar">
    //注册分享文字
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="image/*"/>
    </intent-filter>
    //注册分享单张图片
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
    //注册分享多张图片
    <intent-filter>
        <action android:name="android.intent.action.SEND_MULTIPLE"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="image/*"/>
    </intent-filter>
</activity>
```


## 工具类
 
```java
public class ShareSysUtils {
    public static ShareSysUtils sysShareUtils;
    private Activity activity;
    private OnShareDataOkListener listener;
    public static int TYPE_TEXT = 0, TYPE_IMAGE = 1;

    public ShareSysUtils(Activity activity) {
        this.activity = activity;
    }


    public static ShareSysUtils get(Activity activity) {
        if (sysShareUtils == null) {
            synchronized (ShareSysUtils.class) {
                if (sysShareUtils == null) {
                    sysShareUtils = new ShareSysUtils(activity);
                }
            }
        }
        return sysShareUtils;
    }

    /**
     * 作为接受分享的一方,处理分享来的数据
     *
     * @param listener 处理监听,数据处理好之后会返回
     */
    public void handleShare(OnShareDataOkListener listener) {
        this.listener = listener;
        Intent intent = activity.getIntent();
        String action = intent.getAction();
        String type = intent.getType();
        if (null == action || type == null) {
            Log.e("chendong", "没有检索到分享数据");
            return;
        }
        if (Intent.ACTION_SEND.equals(action) && type != null) {
            if ("text/plain".equals(type)) {
                handleSendText(intent);
            } else if (type.startsWith("image/")) {
                handleSendImage(intent);
            }
        } else if (Intent.ACTION_SEND_MULTIPLE.equals(action) && type != null) {
            if (type.startsWith("image/")) {
                handleSendMultipleImages(intent);
            }
        }
    }


    /**
     * 分享文字
     *
     * @param title   文字标题
     * @param content 文字内容
     */
    public void shareText(String title, String content) {
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND);
        shareIntent.putExtra(Intent.EXTRA_TEXT, content);
        shareIntent.putExtra(Intent.EXTRA_TITLE, title);
        shareIntent.setType("text/plain");
        //设置分享列表的标题，并且每次都显示分享列表
        activity.startActivity(Intent.createChooser(shareIntent, "分享到"));
    }

    /**
     * 分享单张图片
     *
     * @param path 图片的路径
     */
    public void shareSingleImage(String path) {
        //由文件得到uri
        Uri imageUri = Uri.fromFile(new File(path));
//        Log.d("share", "uri:" + imageUri);  //输出：file:///storage/emulated/0/test.jpg
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND);
        shareIntent.putExtra(Intent.EXTRA_STREAM, imageUri);
        shareIntent.setType("image/*");
        activity.startActivity(Intent.createChooser(shareIntent, "分享到"));
    }

    /**
     * 分享多张图片
     *
     * @param paths 路径的集合
     */
    public void shareMultipleImage(List<String> paths) {
        ArrayList<Uri> uriList = new ArrayList<>();
        for (String path : paths) {
            uriList.add(Uri.fromFile(new File(path)));
        }
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
        shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, uriList);
        shareIntent.setType("image/*");
        activity.startActivity(Intent.createChooser(shareIntent, "分享到"));
    }


    private void handleListener(int type, List<String> list, String title, String content) {
        if (listener != null) {
            listener.OnHandleOk(type, list, title, content);
        }
    }

    /**
     * 处理分享的文本
     *
     * @param intent
     */
    private void handleSendText(Intent intent) {
        String sharedText = intent.getStringExtra(Intent.EXTRA_TEXT);
        String sharedTitle = intent.getStringExtra(Intent.EXTRA_TITLE);
        handleListener(TYPE_TEXT, null, sharedTitle, sharedText);
    }

    /**
     * 处理分享的单张照片
     *
     * @param intent
     */
    private void handleSendImage(Intent intent) {
        Uri imageUri = intent.getParcelableExtra(Intent.EXTRA_STREAM);
        if (imageUri == null)
            return;
        List<String> list = new ArrayList<>();
        list.add(UriUtils.getRealPathFromURI(activity, imageUri));
        handleListener(TYPE_IMAGE, list, null, null);
    }

    /**
     * 处理分享的多张照片
     *
     * @param intent
     */
    private void handleSendMultipleImages(Intent intent) {

        ArrayList<Uri> imageUris = intent.getParcelableArrayListExtra(Intent.EXTRA_STREAM);
        LUtils.e("chendong", imageUris.toString());
        if (imageUris == null)
            return;
        List<String> list = new ArrayList<>();
        for (Uri uri : imageUris) {
            if (uri != null)
                list.add(UriUtils.getRealPathFromURI(activity, uri));
        }
        handleListener(TYPE_IMAGE, list, null, null);
    }


    public interface OnShareDataOkListener {
        void OnHandleOk(int type, List<String> list, String title, String content);
    }
}
```
```java
/**
 * CdLibsTest     com.march.libs.utils
 * Created by 陈栋 on 16/3/25.
 * 功能:
 */
public class UriUtils {

    /**
     * 从uri获取path
     *
     * @param uri
     * @return
     */
    public static String getRealPathFromURI(Context context, Uri uri) {
        if (null == uri) return null;
        final String scheme = uri.getScheme();
        String data = null;
        if (scheme == null)
            data = uri.getPath();
        else if (ContentResolver.SCHEME_FILE.equals(scheme)) {
            data = uri.getPath();
        } else if (ContentResolver.SCHEME_CONTENT.equals(scheme)) {
            Cursor cursor = context.getContentResolver().query(uri, new String[]{MediaStore.Images.ImageColumns.DATA}, null, null, null);
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

## 如何处理分享得到数据？
```java
//如果你按照前面的要求注册了Activity，你可以在Activity中使用下面的代码处理分享得到的数据
shareSysUtils = ShareSysUtils.get(self);
shareSysUtils.handleShare(new ShareSysUtils.OnShareDataOkListener() {
      @Override
      public void OnHandleOk(int type, List<String> list, String title, String content) {
                if (type == ShareSysUtils.TYPE_TEXT) {
                    Log.e("chendong", "分享文本是 " + title + "   " + content);
                } else if (type == ShareSysUtils.TYPE_IMAGE) {
                    Log.e("chendong", "分享图片是 " + list.toString());
                }
            }
        });
```


## 如何发起分享？
```java
//一行代码发起分享，会调起QQ,微信等App
public void ShareImg(View view) {
      shareSysUtils.shareSingleImage("我是图片路径path");
}

public void ShareTxt(View view) {
      shareSysUtils.shareText("我是title", "我是content");
}
```



