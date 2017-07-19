---
layout: post
title: Android 6.0 权限动态申请
category: Android
tags:
  - Android
  - AndroidTips
abbrlink: 2939773101
date: 2016-08-16 00:00:00
keywords:
  - 权限
  - Android6.0
---


## minSdkVersion
- minSdkVersion指的是你的应用可以兼容到的最低版本，比如你只使用了2.X的API，那minSdkVersion就可以尽量小，以适配更多机型，小于minSdkVersion的机型将无法安装。

## maxSdkVersion
- maxSdkVersion指的是你的应用可以支持到的最高版本，高于这个版本的机型将不能安装应用，但是这个不需要我们自己去写，Android系统默认向下兼容的。

## targetSdkVersion
- targetSdkVersion如果设置了此属性，那么在程序执行时，如果目标设备的API版本正好等于此数值，他会告诉Android平台：此程序在此版本已经经过充分测，没有问题。不必为此程序开启兼容性检查判断的工作了。


## 介绍
- 由于Android M对隐私权限必须动态申请，当你的targetSdkVersion>=23时就需要进行权限申请了，否则将获得不了权限。不过当targetSdkVersion<23时，由于Android版本向下兼容性，你的应用将运行在<23的环境中，不会出现问题。

- Git上已经有很多开源库供大家使用了，确实方便简单，[PermissionGen](https://github.com/lovedise/PermissionGen)这个库好评率还是蛮高的，使用注解简化了很多操作，网上很多博客都推荐它，不过，我觉得不能遇到啥问题就引个别人的库进来， 一方面如果库的定制性不高，自定义起来也是麻烦，另一方面，像这种比较简单的问题，还是自己写比较好，可以了解一下是如何实现的，对自己的学习也有好处。别人封装的再好也不是自己的，以后只能说我会使用XX,XXX...类库.那就尴尬了。扯远了～



## Android权限
- Android M之后将权限分为了两类，Normal和Dangerous Permission,Dangerous Permission大都是跟用户隐私相关的权限，如下表，更详细见[Android文档－Permissions](https://developer.android.com/guide/topics/security/permissions.html)

| Permission Group | Permissions |
| ------------- |:-------------| 
|CALENDAR	| READ _ CALENDAR |
|			|WRITE _ CALENDAR|
|CAMERA	|CAMERA|
|CONTACTS	|READ _ CONTACTS|
|          |WRITE _ CONTACTS|
|         |GET _ ACCOUNTS
|LOCATION	|ACCESS _ FINE _ LOCATION|
|          |ACCESS _ COARSE _ LOCATION|
|MICROPHONE  |RECORD _ AUDIO|
|PHONE	      |READ _ PHONE _ STATE|
|                 |CALL _ PHONE|
|                |READ _CALL _ LOG|
|               |WRITE _ CALL _ LOG|
|               |ADD _ VOICEMAIL|
|              |USE _ SIP|
|                 |PROCESS _ OUTGOING _ CALLS|
|SENSORS	 |BODY _ SENSORS|
|SMS	    |SEND _ SMS|
|           |RECEIVE _ SMS|
|           |READ _ SMS|
|           |RECEIVE _ WAP _ PUSH|
|           |RECEIVE _ MMS
|STORAGE	 | READ _ EXTERNAL _ STORAGE|
 |          | WRITE _ EXTERNAL _ STORAGE|
 





## 思路和API
- 涉及到的核心方法`context.requestPermissions(String[] permissions,int reqCode);`该方法必须在Android M以上可以使用。大体的思路就是：

1. 当前手机版本是不是大于Android M ？
2. 是否有私有权限需要申请？
3. 该权限是不是已经同意了 
4. 弹个dialog告诉用户我们将要申请哪些权限，现在市面上app大都是这么做的
5. 开始申请 
6. 处理申请结果




## PermissionHelper
- 为了方便操作，我将部分核心方法抽象出来了，调用`checkPermission()`申请权限，返回值会告诉你是不是需要申请，调用`onRequestPermissionsResult()`会格式化处理返回的数据。

```java
/**
 * com.march.baselib.helper
 * CommonLib
 * Created by chendong on 16/8/17.
 * Copyright © 2016年 chendong. All rights reserved.
 * Desc :
 */
public class PermissionHelper {

    public static final int REQ_PERMISSION_CODE = 0x12;

    /**
     * 动态申请权限
     * @param context 上下文
     * @param permissions 需要申请的权限
     * @return 是不是需要申请
     */
    public static boolean checkPermission(Activity context, String[] permissions) {
        //6.0以上
        if (AppHelper.isOverMarshmallow()) {
            //没有权限需要申请时
            if (permissions == null || permissions.length <= 0)
                return true;
            //检查权限是不是已经授予
            List<String> noOkPermissions = new ArrayList<>();
            for (String permission : permissions) {
                if (context.checkSelfPermission(permission) == PackageManager.PERMISSION_DENIED) {
                    noOkPermissions.add(permission);
                }
            }
            //该权限已经授予，不再申请
            if (noOkPermissions.size() <= 0)
                return true;
            //6.0以上需要申请权限
            context.requestPermissions(noOkPermissions.toArray(new String[noOkPermissions.size()]), REQ_PERMISSION_CODE);
            return false;
        }
        //6.0以下下不需要申请
        return true;
    }

    /**
     * 处理权限申请的结果，返回结构化的数据
     * @param requestCode 请求码
     * @param permissions 被请求的权限
     * @param grantResults 请求结果
     * @param listener 监听
     */
    public static void onRequestPermissionsResult(int requestCode,
                                                  @NonNull String[] permissions,
                                                  @NonNull int[] grantResults,
                                                  OnPermissionHandleOverListener listener) {
        if (requestCode != REQ_PERMISSION_CODE)
            return;
        Map<String, Integer> result = new HashMap<>();
        boolean isHavePermissionNotOk = false;
        for (int i = 0; i < Math.min(permissions.length, grantResults.length); i++) {
            result.put(permissions[i], grantResults[i]);
            //有权限没有同意
            if (grantResults[i] == PackageManager.PERMISSION_DENIED) {
                isHavePermissionNotOk = true;
            }
        }
        //如果权限全部同意，继续执行
        if (listener != null)
            listener.onHandleOver(!isHavePermissionNotOk, result);
    }

    public interface OnPermissionHandleOverListener {
        void onHandleOver(boolean isOkExactly, Map<String, Integer> result);
    }
}
```


## BaseActivity
- 一般权限监测是在Activity进行，在基类中监测要简单方便


```java
@Override
public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mSaveBundle = savedInstanceState;
        mActivity = this;
        mContext = getApplicationContext();
        createViewShow();
        //如果不需要监测，会立刻执行invokeCommonMethod()开始Activity操作，如果需要就会发起申请
        if (PermissionHelper.checkPermission(mActivity, getPermission2Check()))
            invokeCommonMethod(mSaveBundle);
    }
    
    
//抽象方法，由子类来决定哪些权限需要申请
protected abstract String[] getPermission2Check();

//子类实现决定处理结果
protected boolean handlePermissionResult(Map<String, Integer> resultNotOk) {
        return true;
}


//处理返回结果
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionHelper.onRequestPermissionsResult(requestCode, permissions, grantResults, new PermissionHelper.OnPermissionHandleOverListener() {
            @Override
            public void onHandleOver(boolean isOkExactly, Map<String, Integer> result) {
                //权限ok或者子类要求直接执行
                if (isOkExactly || handlePermissionResult(result))
                    invokeCommonMethod(mSaveBundle);
            }
        });
    }
```


## 子类

```java
//哪些权限需要申请
@TargetApi(Build.VERSION_CODES.JELLY_BEAN)
@Override
protected String[] getPermission2Check() {
        return new String[]{Manifest.permission.READ_EXTERNAL_STORAGE};
}

//如果权限没有获得如何处理
@Override
protected boolean handlePermissionResult(Map<String, Integer> resultNoOk) {
        if (resultNoOk.get(Manifest.permission.READ_EXTERNAL_STORAGE) == PackageManager.PERMISSION_DENIED) {
            Toaster.get().show("您没有允许读取存储卡，不能继续操作");
            return false;
        }
        return true;
}
```

## 以上
