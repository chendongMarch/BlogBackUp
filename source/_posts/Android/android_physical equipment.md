---
layout: post
title: Android物理按键及物理连接
category: Android
tags: Android
abbrlink: 4013192295
date: 2016-02-26 00:00:00
keywords:
---


本文主要介绍 `Android` 手机物理按键触发监听，以及外接蓝牙，耳机等设备的检测。
<!--more-->

## 事件分发
- 拦截按键按下抬起时的事件分发

```java
@Override
    public boolean dispatchKeyEvent(KeyEvent event) {
    //如果是长按事件交给onKeyLongPress处理
        if (event.getRepeatCount() > 0)
            return onKeyLongPress(event.getKeyCode(), event);

        return super.dispatchKeyEvent(event);
    }
```


## 返回按键
```java
//返回键的监听很常见，android提供了简便的方法
@Override
public void onBackPressed() {
        finishThisPage();
}
```


## 按键按下事件处理
- (音量，耳机按键，back键)

```java
//home键和power键没有监听到
//我当时需要的是音量键和耳机按键可以拍照，是可以监听到的
@Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        L.info(event.toString());
        switch (keyCode) {
            case KeyEvent.KEYCODE_VOLUME_DOWN:
                L.info("KEYCODE_VOLUME_DOWN-音量键减小声音");
                return true;
            case KeyEvent.KEYCODE_VOLUME_UP:
                L.info("KEYCODE_VOLUME_UP-音量键增加声音");
                return true;
            case KeyEvent.KEYCODE_VOLUME_MUTE:
                L.info("KEYCODE_VOLUME_MUTE-音量键静音");
                return true;
            case KeyEvent.KEYCODE_HEADSETHOOK:
                L.info("KEYCODE_HEADSETHOOK-耳机线按键");
                return true;
            case KeyEvent.KEYCODE_BACK:
                L.info("KEYCODE_BACK-返回键");
                return true;
        }
        return super.onKeyDown(keyCode, event);
    }
```

## 其他事件
```java
// 使用快捷键可以快速选中菜单中的某一项，在Android没有什么卵用
@Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuItem add = menu.add("");
        add.setShortcut('A','A');
        return super.onCreateOptionsMenu(menu);
    }
@Override
    public boolean onKeyShortcut(int keyCode, KeyEvent event) {
        L.info("onKeyShortcut  " + event.toString());
        return super.onKeyShortcut(keyCode, event);
    }

@Override
    public boolean onKeyLongPress(int keyCode, KeyEvent event) {
        L.info("onKeyLongPress  " + event.toString());
        return super.onKeyLongPress(keyCode, event);
    }

 //暂时没有用到，网上有说在手写和汉字输入时会触发
@Override
    public boolean onKeyMultiple(int keyCode, int repeatCount, KeyEvent event) {
        L.info("onKeyMultiple  " + event.toString());
        return super.onKeyMultiple(keyCode, repeatCount, event);
    }

//按键抬起时会触发
@Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        L.info("onKeyUp   " + event.toString());
        return super.onKeyUp(keyCode, event);
    }
```

## 外设监测
- 检测耳机线，蓝牙耳机是否连接

```java
private boolean checkControlIsConnected() {
        //检测耳机线是否连接
        AudioManager am = (AudioManager) getSystemService(AUDIO_SERVICE);
        //这个方法已经过时，api介绍仅仅用来检测是否连接，可以注册广播接收者接受耳机插拔的广播
        if (am.isWiredHeadsetOn()) {
            //耳机插入
            return true;
        }

        //检测蓝牙设备是否连接
        BluetoothAdapter ba = BluetoothAdapter.getDefaultAdapter();
        //蓝牙适配器是否存在，即是否发生了错误
        if (ba == null || !ba.isEnabled()) {
            //蓝牙不可用
            return false;
        } else if (ba.isEnabled()) {
            Set<BluetoothDevice> bondedDevices = ba.getBondedDevices();
            if (bondedDevices == null || bondedDevices.size() <= 0) {
                //当前没有设备连接
                return false;
            } else {
                for (BluetoothDevice d : bondedDevices) {
                    if (d.getBondState() == BluetoothDevice.BOND_BONDED) {
                        //当前有设备接入并处于连接状态
                        return true;
                    }
                }
            }

        }
        return false;
    }
```
