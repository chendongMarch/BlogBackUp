---
layout: post
title: Android Dialog总结
date: 2015-01-22
category: Android
tags: Android
keywords: 
description: Android Dialog总结
---

## ProgressDialog
- ProgressDialog是AlertDialog的子类


```java
//简单的使用方法，只是用来提示用户等待,
ProgressDialog dialog ＝ ProgressDialog.show(context,"title","message");


//设置更多参数的用法
ProgressDialog mWaitDialog = new ProgressDialog(self);
mWaitDialog.setMessage("请稍候");
mWaitDialog.setTitle("标题");
//设置按钮，DialogInterface.BUTTON_NEGATIVE可以设置按钮的属性，不携带该参数方法已经过时
mWaitDialog.setButton(DialogInterface.BUTTON_NEGATIVE, "返回", new DialogInterface.OnClickListener() {
      @Override
      public void onClick(DialogInterface dialog, int which) {
             dialog.dismiss();
             finish();
});
mWaitDialog.setIndeterminate(false);
mWaitDialog.setCancelable(false);
mWaitDialog.setCanceledOnTouchOutside(false);
mWaitDialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
mWaitDialog.setProgress(0);
mWaitDialog.setMax(100);
mWaitDialog.show();
```