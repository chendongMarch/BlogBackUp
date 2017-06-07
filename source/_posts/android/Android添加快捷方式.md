---
layout: post
title: Android添加快捷方式
date: 2016-07-29
category: Android
tags: Android
keywords: 
description: Android添加快捷方式
---

## 权限
```java
<uses-permission android:name="com.android.launcher.permission.INSTALL_SHORTCUT"/>
<uses-permission android:name="com.android.launcher.permission.UNINSTALL_SHORTCUT"/>
```

## 配置
- 快捷方式要打开一个界面，需要一个Intent配置这些信息

```java
/**
* 获取Intent
*
* @param thisActivity 当前activity
* @param goActivity 点击打开的activity
* @return 创建的Intent
*/
private static Intent getShortCutIntent(Activity thisActivity, Class goActivity) {
	// 使用MAIN，可以避免部分手机(比如华为、HTC部分机型)删除应用时无法删除快捷方式的问题
	Intent intent = new Intent(Intent.ACTION_MAIN);
	intent.addCategory(Intent.CATEGORY_DEFAULT);
	intent.setClass(thisActivity, goActivity);
	return intent;
}
```

## Action
```java
// Action 添加Shortcut
public static final String ACTION_ADD_SHORTCUT = "com.android.launcher.action.INSTALL_SHORTCUT";
// Action 移除Shortcut
public static final String ACTION_REMOVE_SHORTCUT = "com.android.launcher.action.UNINSTALL_SHORTCUT";

```

## 添加快捷方式
```java
/**
* 添加快捷方式
*
* @param activity    context
* @param name        name
* @param goActivity  要启动的界面
* @param allowRepeat 是否允许重复,建议false
* @param iconBitmap  快捷方式图标
*/
public static void addShortcut(Activity activity, Class goActivity, String name,boolean allowRepeat, Bitmap iconBitmap) {
	Intent addShortcutIntent = new Intent(ACTION_ADD_SHORTCUT);
	// 是否允许重复创建
	addShortcutIntent.putExtra("duplicate", allowRepeat);
	// 快捷方式的标题
	addShortcutIntent.putExtra(Intent.EXTRA_SHORTCUT_NAME, name);
	// 快捷方式的图标
	addShortcutIntent.putExtra(Intent.EXTRA_SHORTCUT_ICON, iconBitmap);
	// 快捷方式的动作
	Intent shortCutIntent = getShortCutIntent(activity, goActivity);
	addShortcutIntent.putExtra(Intent.EXTRA_SHORTCUT_INTENT, shortCutIntent);
	activity.sendBroadcast(addShortcutIntent);
}


public static void addShortcut(Activity activity, Class goActivity, String name,
                                   boolean allowRepeat, int res) {
   BitmapFactory.Options options = new BitmapFactory.Options();
   Bitmap bitmap = BitmapFactory.decodeResource(activity.getResources(), res);
   addShortcut(activity, goActivity, name, allowRepeat, bitmap);
}
```


## 移除快捷方式
```java
/**
 * 移除快捷方式
 *
 * @param activity   context
 * @param goActivity 要启动的Activity
 * @param name       name
 */
public static void removeShortcut(Activity activity, Class goActivity, String name) {
    Intent intent = new Intent(ACTION_REMOVE_SHORTCUT);
    intent.putExtra(Intent.EXTRA_SHORTCUT_NAME, name);
	 //intent.addCategory(Intent.CATEGORY_LAUNCHER);
    intent.putExtra("duplicate", false);
    Intent shortCutIntent = getShortCutIntent(activity, goActivity);
    intent.putExtra(Intent.EXTRA_SHORTCUT_INTENT, shortCutIntent);
    activity.sendBroadcast(intent);
}
```


## 附全部代码，亲测可用
```java
/**
 * com.march.libs.helper
 * CdLibsTest
 * Created by chendong on 16/7/29.
 * Copyright © 2016年 chendong. All rights reserved.
 * Desc :
 */
public class ShortCutHelper {

    // Action 添加Shortcut
    public static final String ACTION_ADD_SHORTCUT = "com.android.launcher.action.INSTALL_SHORTCUT";
    // Action 移除Shortcut
    public static final String ACTION_REMOVE_SHORTCUT = "com.android.launcher.action.UNINSTALL_SHORTCUT";


    /**
     * 获取Intent
     *
     * @param thisActivity 当前activity
     * @param goActivity 点击打开的activity
     * @return 创建的Intent
     */
    private static Intent getShortCutIntent(Activity thisActivity, Class goActivity) {
        // 使用MAIN，可以避免部分手机(比如华为、HTC部分机型)删除应用时无法删除快捷方式的问题
        Intent intent = new Intent(Intent.ACTION_MAIN);
        intent.addCategory(Intent.CATEGORY_DEFAULT);
        intent.setClass(thisActivity, goActivity);
        return intent;
    }

    /**
     * 添加快捷方式
     *
     * @param activity    context
     * @param name        name
     * @param goActivity  要启动的界面
     * @param allowRepeat 是否允许重复
     * @param iconBitmap  快捷方式图标
     */
    public static void addShortcut(Activity activity, Class goActivity, String name,
                                   boolean allowRepeat, Bitmap iconBitmap) {
        Intent addShortcutIntent = new Intent(ACTION_ADD_SHORTCUT);
        // 是否允许重复创建
        addShortcutIntent.putExtra("duplicate", allowRepeat);
        // 快捷方式的标题
        addShortcutIntent.putExtra(Intent.EXTRA_SHORTCUT_NAME, name);
        // 快捷方式的图标
        addShortcutIntent.putExtra(Intent.EXTRA_SHORTCUT_ICON, iconBitmap);
        // 快捷方式的动作
        Intent shortCutIntent = getShortCutIntent(activity, goActivity);
        addShortcutIntent.putExtra(Intent.EXTRA_SHORTCUT_INTENT, shortCutIntent);
        activity.sendBroadcast(addShortcutIntent);
    }

    public static void addShortcut(Activity activity, Class goActivity, String name,
                                   boolean allowRepeat, int res) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        Bitmap bitmap = BitmapFactory.decodeResource(activity.getResources(), res);
        addShortcut(activity, goActivity, name, allowRepeat, bitmap);
    }


    /**
     * 移除快捷方式
     *
     * @param activity   context
     * @param goActivity 要启动的Activity
     * @param name       name
     */
    public static void removeShortcut(Activity activity, Class goActivity, String name) {
        Intent intent = new Intent(ACTION_REMOVE_SHORTCUT);
        intent.putExtra(Intent.EXTRA_SHORTCUT_NAME, name);
		  //intent.addCategory(Intent.CATEGORY_LAUNCHER);
        intent.putExtra("duplicate", false);
        Intent shortCutIntent = getShortCutIntent(activity, goActivity);
        intent.putExtra(Intent.EXTRA_SHORTCUT_INTENT, shortCutIntent);
        activity.sendBroadcast(intent);
    }
}
```
